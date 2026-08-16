# Enterprise Azure Lab - Troubleshooting

**Version:** 1.2  
**Last Updated:** August 15, 2026  
**Author:** Kendrick McClure

---

## Overview

This document records technical issues encountered while building the Azure Enterprise Lab, including investigation steps, root causes, resolutions, validation, and lessons learned.

The goal is to document real-world troubleshooting scenarios encountered during the deployment and continued development of the environment.

Entries focus on situations that required meaningful investigation, validation, or remediation rather than documenting every routine configuration task as a troubleshooting event.

The current network architecture is shown below for reference.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

# Issue #1 - VM Had No Outbound Internet Connectivity

## Symptoms

During the initial deployment of server infrastructure, a virtual machine was accessible through Azure Bastion but could not establish outbound HTTPS connectivity.

Observed behavior included:

- Azure Bastion connectivity was successful
- The virtual machine received a valid private IP address
- DNS resolution functioned correctly
- Windows Server displayed an activation warning
- Outbound HTTPS connectivity failed

The following PowerShell test was used to validate TCP connectivity to an external HTTPS endpoint:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : False
```

This indicated that the virtual machine could resolve the destination but could not successfully establish the outbound TCP connection.

---

## Investigation

The troubleshooting process validated each major component of the network path individually rather than assuming that successful DNS resolution indicated full Internet connectivity.

### Private IP Configuration

The virtual machine received a valid private IP address within its assigned subnet.

This confirmed that the Azure network interface and subnet configuration were providing the expected private addressing.

### DNS Resolution

DNS resolution was tested using:

```powershell
nslookup www.microsoft.com
```

Name resolution completed successfully.

This demonstrated that the virtual machine could resolve the external hostname to an IP address, but it did not confirm that the system could establish an outbound connection to the resolved destination.

### HTTPS Connectivity

Outbound TCP connectivity was tested separately using:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : False
```

The difference between successful DNS resolution and failed TCP connectivity helped narrow the issue to outbound network connectivity rather than name resolution.

### Effective Routes

Azure effective routes were reviewed to determine whether the virtual machine had a valid route for Internet-bound traffic.

An active default route was present:

```text
0.0.0.0/0
Next Hop: Internet
State: Active
```

The presence of the route indicated that the issue was not caused by the absence of a default route.

### Network Security Group

The applicable Network Security Group configuration was reviewed.

Outbound Internet traffic was permitted by the effective NSG configuration, indicating that an NSG rule was not blocking the HTTPS connection.

### Route Table

The subnet configuration was reviewed for custom routing.

No User Defined Route was associated with the affected subnet.

This ruled out a custom route table as the source of the failed outbound connection.

### Investigation Summary

At this point, the following had been established:

- Private IP configuration was valid
- DNS resolution was functioning
- An active default route existed
- Network Security Group configuration permitted outbound traffic
- No User Defined Route was redirecting or blocking the traffic
- Azure Bastion connectivity to the virtual machine was functioning
- Outbound TCP/443 connectivity was still failing

These findings shifted the investigation toward the subnet's outbound connectivity configuration.

---

## Root Cause

The affected workload subnet was configured as a **private subnet with default outbound access disabled**.

At that stage of the deployment, no explicit outbound connectivity method had been configured for the subnet.

The virtual machine therefore had functioning private networking and DNS resolution but did not have an explicitly configured mechanism for establishing outbound Internet connections.

This explained why Azure Bastion connectivity and DNS resolution could function while outbound HTTPS connectivity still failed.

---

## Temporary Resolution

During the initial deployment, default outbound connectivity was temporarily enabled to allow required Internet access while the environment was being configured.

After changing the subnet configuration, the existing virtual machine did not immediately exhibit the expected outbound behavior.

The virtual machine was therefore stopped and deallocated through Azure and then started again.

### Steps Performed

1. Temporarily permit default outbound connectivity for the affected subnet.
2. Stop and deallocate the virtual machine.
3. Start the virtual machine.
4. Reconnect through Azure Bastion.
5. Revalidate DNS resolution.
6. Test outbound HTTPS connectivity.

Validation command:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Successful result:

```text
TcpTestSucceeded : True
```

Outbound connectivity functioned after the subnet configuration was changed and the virtual machine was deallocated and restarted.

This restored the connectivity required to continue deployment activities.

However, reliance on default outbound access was intentionally treated as a temporary configuration rather than the desired long-term network architecture.

---

## Permanent Architectural Resolution

After the initial deployment work was completed, the outbound architecture was revisited to eliminate reliance on Azure default outbound access.

An Azure NAT Gateway was deployed to provide an explicit outbound connectivity path for the private Corporate workload subnets.

### NAT Gateway Resources

| Property | Value |
| --- | --- |
| NAT Gateway | nat-corp-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2 |

The NAT Gateway is currently associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

A dedicated static Azure Public IP resource is associated with the NAT Gateway to provide a predictable outbound egress resource.

The assigned public IP address itself is intentionally excluded from the public repository.

---

## NAT Gateway Implementation

The permanent outbound connectivity implementation followed this sequence:

1. Deploy Azure NAT Gateway.
2. Create and associate the dedicated static Public IP resource `pip-nat-corp-prd-eus2`.
3. Associate the NAT Gateway with the required Corporate workload subnets.
4. Validate outbound HTTPS connectivity.
5. Confirm that Internet-bound traffic used the NAT Gateway Public IP resource for egress.
6. Restore private subnet configuration.
7. Confirm that default outbound access remained disabled.
8. Revalidate outbound HTTPS connectivity from the private workload.

Azure NAT Gateway performs **Source Network Address Translation (SNAT)** for outbound connections originating from the associated private subnets.

The private source address of an outbound workload connection is translated to the static Public IP resource associated with the NAT Gateway.

This provides the workload subnets with an explicitly configured outbound Internet path without assigning public IP addresses directly to the server virtual machines.

The Internal Apps subnet was later associated with the same NAT Gateway to provide explicit outbound connectivity for WEB01 and WEB02.

---

## Permanent Resolution Validation

After the NAT Gateway was deployed and private subnet configuration was restored, outbound HTTPS connectivity was tested again.

Validation command:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : True
```

The observed Internet-facing source address was also verified to match the Public IP resource associated with the NAT Gateway.

This confirmed that outbound traffic was using the intended NAT Gateway egress path rather than relying on Azure default outbound access.

### Final State

The final configuration was validated by confirming:

- Azure Bastion administrative connectivity remained functional
- Private IP addressing remained functional
- DNS resolution remained functional
- Outbound HTTPS connectivity succeeded
- Active Corporate workload subnets remained configured as private subnets
- Default outbound access remained disabled
- Azure NAT Gateway provided explicit outbound connectivity
- Internet-bound traffic used the NAT Gateway Public IP resource for outbound SNAT
- The observed public egress address matched the expected NAT Gateway Public IP resource
- DC01 remained without a direct public IP assignment
- MGMT01 remained without a direct public IP assignment
- WEB01 remained without a direct public IP assignment
- WEB02 remained without a direct public IP assignment

---

## Architectural Outcome

The troubleshooting process resulted in more than restoration of Internet connectivity.

The initial problem exposed an architectural requirement for explicitly defined outbound connectivity from private workload subnets.

Rather than leaving the temporary default outbound configuration in place, the environment was updated to use Azure NAT Gateway as the intended outbound egress mechanism.

The resulting design separates several connectivity functions:

- **Administrative access** is provided through Azure Bastion
- **Private workload communication** occurs through Azure Virtual Networks and VNet peering
- **Internal application delivery** is provided through an internal Azure Load Balancer
- **Outbound Internet connectivity** is provided through Azure NAT Gateway

Server virtual machines remain privately addressed and do not require direct public IP assignments for administrative, application, or outbound connectivity.

---

## Lessons Learned

### DNS Resolution Does Not Prove Internet Connectivity

Successful DNS resolution confirms that a hostname can be resolved to an IP address.

It does not prove that a TCP connection can be established to the resolved destination.

Testing the destination TCP port separately helped demonstrate that name resolution was functioning while outbound HTTPS connectivity was not.

### Troubleshoot Network Connectivity Layer by Layer

The issue reinforced the value of testing network connectivity one layer at a time.

The investigation separately validated:

- Private IP configuration
- DNS resolution
- TCP connectivity
- Effective routes
- Network Security Groups
- Route tables
- Subnet outbound configuration

This approach reduced guesswork and helped isolate the actual cause of the failed connection.

### Azure Bastion Connectivity Is Independent of Workload Internet Egress

Successful Azure Bastion connectivity demonstrated that the virtual machine was reachable through the private Azure network architecture.

However, successful administrative connectivity through Bastion did not mean that the workload itself had outbound Internet connectivity.

The two connectivity paths serve different purposes and should be evaluated independently.

### Temporary Fixes Should Be Revisited

Enabling default outbound connectivity allowed deployment work to continue, but it was intentionally treated as a temporary workaround.

Once the underlying requirement was understood, the environment was redesigned with an explicitly configured outbound connectivity method.

This prevented a temporary troubleshooting change from becoming the permanent architecture by accident.

### Deallocation Can Matter After Network Configuration Changes

After the initial subnet outbound configuration was changed, stopping and deallocating the virtual machine and then starting it again was required before the expected connectivity behavior was observed.

This reinforced the importance of considering Azure resource state when troubleshooting infrastructure changes rather than relying exclusively on guest operating system restarts.

### NAT Gateway Performs Translation, Not Traffic Inspection

Azure NAT Gateway performs outbound SNAT but is not a firewall.

It does not replace Network Security Groups, centralized network filtering, or application-layer traffic inspection.

Future implementation of Azure Firewall will address a different architectural requirement than the NAT Gateway.

### Validate the Final Architecture, Not Just the Original Symptom

Restoring Internet connectivity was not treated as sufficient evidence that the final architecture was correct.

After the permanent NAT Gateway configuration was implemented, outbound connectivity was tested again with private subnet configuration restored and default outbound access disabled.

The observed public egress address was also verified against the NAT Gateway Public IP resource.

This confirmed that the intended architecture was actually being used.

---

# Issue #2 - Active Directory Domain Join and DNS Dependency

## Scenario

After deploying DC01 and promoting it to a Domain Controller, MGMT01 needed to be joined to the new `corp.mccluretech.com` Active Directory domain.

The implementation required more than network reachability between the two servers.

Active Directory domain joining depends heavily on DNS because clients use DNS records to discover Domain Controllers and Active Directory services.

---

## Investigation

DC01 was configured with the static private IP address:

```text
10.1.0.4
```

Active Directory Domain Services and Active Directory-integrated DNS were hosted on DC01.

The Corporate VNet DNS configuration was then configured to use:

```text
10.1.0.4
```

as its custom DNS server.

This ensured that Corporate workloads using the VNet-provided DNS configuration would query DC01 rather than relying on Azure-provided DNS for the internal Active Directory namespace.

MGMT01 was then configured to use the updated network DNS configuration.

---

## Resolution

After the Corporate VNet was configured to use DC01 as its custom DNS server, MGMT01 was successfully joined to:

```text
corp.mccluretech.com
```

The MGMT01 computer object was subsequently moved into the custom `Servers` Organizational Unit.

Domain authentication from MGMT01 was also validated.

---

## Validation

The completed configuration confirmed:

- DC01 provided Active Directory Domain Services
- DC01 provided Active Directory-integrated DNS
- DC01 maintained a static private IP address
- The Corporate VNet used DC01 as its custom DNS server
- MGMT01 could locate the Active Directory domain
- MGMT01 successfully joined `corp.mccluretech.com`
- Domain authentication functioned from MGMT01
- MGMT01 was organized within the custom `Servers` OU

---

## Lessons Learned

### Active Directory Depends on DNS

Successful IP connectivity to a Domain Controller is not sufficient for Active Directory functionality.

Domain clients must be able to resolve the Active Directory DNS namespace and locate the DNS service records used for domain discovery.

### Infrastructure DNS Requires a Stable Endpoint

Because DC01 provides DNS for the environment, its private address must remain predictable.

A static private IP assignment was therefore appropriate for DC01.

### VNet DNS Configuration Connects Azure Networking to Active Directory

Configuring custom DNS at the Corporate VNet level allows Azure workloads using the VNet-provided configuration to receive the Active Directory DNS server automatically.

This creates an important relationship between Azure network configuration and Windows domain infrastructure.

---

# Issue #3 - Overly Broad NTFS Permissions on ITShare

## Symptoms

During implementation and validation of the `ITShare` file resource on MGMT01, Active Directory security groups were created to provide group-based access to the resource.

The intended authorization structure followed an AGDLP-style model:

```text
User Account
    ↓
GG-IT-Users
    ↓
DL-ITShare-RW
    ↓
ITShare
```

`DL-ITShare-RW` was assigned Modify access to the ITShare folder.

During review of the folder's effective NTFS permissions, an additional access path was discovered:

```text
Authenticated Users - Modify
```

This permission was inherited from the parent filesystem structure.

As a result, the intended Active Directory group model was not the only mechanism capable of granting normal user access to the folder.

---

## Investigation

The folder's NTFS security configuration was reviewed after the Active Directory groups and resource permissions were configured.

The investigation identified that `Authenticated Users` had inherited Modify permissions.

This was broader than the intended authorization design.

The goal of the resource was for normal user access to be controlled through:

```text
GG-IT-Users
    ↓
DL-ITShare-RW
    ↓
NTFS Modify
```

Leaving the inherited `Authenticated Users` permission in place would undermine that design because authenticated domain users could potentially receive access independently of membership in the intended security groups.

---

## Remediation

The issue was remediated specifically on the ITShare folder rather than changing permissions on the root operating-system volume.

Inheritance was disabled on the ITShare folder.

Existing inherited permissions were first converted to explicit permissions.

This prevented required administrative entries from being unintentionally removed when inheritance was disabled.

The unnecessary broad permission entries were then removed.

The final intended NTFS ACL retained:

| Principal | Permission |
| --- | --- |
| DL-ITShare-RW | Modify |
| SYSTEM | Full Control |
| Local Administrators | Full Control |

This preserved administrative access while ensuring that normal user access was controlled through the intended Active Directory authorization model.

---

## Validation

The final permissions were reviewed after remediation.

Validation confirmed that:

- `DL-ITShare-RW` had Modify permission
- `SYSTEM` retained Full Control
- Local Administrators retained Full Control
- The unnecessary `Authenticated Users` Modify permission was removed
- Folder inheritance no longer introduced the unintended broad permission
- Resource authorization aligned with the intended Active Directory group design

---

## Root Cause

The issue was caused by inherited NTFS permissions from the parent filesystem structure.

Creating the intended security groups and assigning the Domain Local group to the resource did not automatically remove permissions inherited from the parent folder.

The authorization model therefore needed to be evaluated as a complete ACL rather than validating only that the desired group had been added.

---

## Lessons Learned

### Adding the Correct Permission Does Not Mean the ACL Is Correct

The presence of the intended security group on a resource does not prove that access is appropriately restricted.

Other ACL entries may provide broader access than intended.

### Inheritance Must Be Evaluated

NTFS inheritance can introduce permissions that conflict with the desired authorization model.

Inherited permissions should be reviewed when implementing restricted resources.

### Avoid Changing Root Filesystem Permissions Unnecessarily

The issue was corrected at the resource folder rather than changing permissions on the root operating-system volume.

This reduced the risk of affecting unrelated Windows components or resources.

### Preserve Administrative Access During ACL Changes

Converting inherited permissions to explicit permissions before removing unnecessary entries provided a controlled way to modify the ACL while retaining required SYSTEM and administrative access.

### Validate Effective Authorization

Security configuration should be validated based on who can actually access a resource, not merely on whether the intended security group appears in the permissions list.

---

# Issue #4 - Internal Load Balancer Failover Validation

## Scenario

WEB01 and WEB02 were deployed as two Nginx backend servers within the Internal Apps subnet.

An internal Azure Load Balancer was configured with the private frontend address:

```text
10.1.3.10
```

Both web servers were added to the backend pool, and TCP port 80 was used for application traffic and backend health monitoring.

The application tier then needed to be validated to confirm that the Load Balancer could continue serving the application if one backend became unavailable.

---

## Initial Validation

Testing was performed from MGMT01 against the Load Balancer frontend rather than by targeting the individual backend server addresses.

An HTTP request was sent to:

```text
http://10.1.3.10
```

The request successfully returned the Nginx response hosted by WEB01.

The backend web pages were intentionally configured to identify which server generated the response, making backend selection observable during testing.

This confirmed that:

- MGMT01 could reach the internal Load Balancer frontend
- The Load Balancer could forward TCP/80 traffic to a backend
- WEB01 was healthy and responding
- The application was accessible through the Load Balancer rather than requiring direct client access to the backend server

---

## Failure Simulation

WEB01 was intentionally shut down to simulate loss of one application backend.

The Load Balancer frontend address remained unchanged:

```text
10.1.3.10
```

Another HTTP request was then sent to the same frontend address.

The application response was successfully returned by WEB02.

---

## Validation

The test confirmed:

- The Load Balancer frontend remained reachable
- WEB01 could become unavailable without changing the client endpoint
- The health mechanism detected backend availability
- WEB02 remained available to serve application traffic
- Traffic could be delivered to the remaining healthy backend
- Application availability was maintained after one backend was intentionally removed from service

---

## Architectural Outcome

The test demonstrated the value of separating the application endpoint from individual backend server addresses.

Clients use:

```text
10.1.3.10
```

rather than depending directly on:

```text
WEB01
WEB02
```

The Load Balancer therefore provides a stable application endpoint while backend membership and availability can change independently.

The test also reinforced that the following components perform different functions:

- **Azure Load Balancer** - distributes application traffic
- **Health probe** - evaluates backend availability
- **Network Security Group** - controls permitted network traffic
- **Azure NAT Gateway** - provides outbound Internet connectivity
- **Nginx** - provides the application service

---

## Lessons Learned

### Test the Service Endpoint, Not Just Individual Servers

Successfully connecting directly to WEB01 or WEB02 would prove that the individual server was reachable.

It would not prove that the Load Balancer configuration worked.

Testing through `10.1.3.10` validated the actual client-facing application path.

### High Availability Should Be Tested by Creating Failure

A backend pool containing two healthy servers does not by itself prove that service remains available during failure.

Intentionally shutting down WEB01 provided a controlled way to validate behavior when a backend became unavailable.

### The Client Endpoint Should Remain Stable

The client continued using the same Load Balancer frontend address before and after WEB01 was shut down.

This demonstrates one of the primary benefits of placing backend workloads behind a load-balancing layer.

### Load Balancing and Outbound Connectivity Are Separate

The Internal Apps subnet uses Azure NAT Gateway for outbound Internet access, while the internal Azure Load Balancer handles application delivery.

The successful failover test reinforced that these services solve separate network requirements.

---

# Troubleshooting Methodology

The issues encountered during development of the lab reinforced a repeatable troubleshooting process.

When investigating infrastructure problems:

1. Identify the exact symptom.
2. Separate assumptions from verified behavior.
3. Test the lowest relevant layer first.
4. Validate addressing and name resolution independently.
5. Test the required protocol and port directly.
6. Review Azure network configuration and effective controls.
7. Verify operating-system or application configuration where appropriate.
8. Identify the root cause before making permanent architectural changes.
9. Apply the smallest appropriate remediation.
10. Re-test the original failure condition.
11. Validate that the final architecture behaves as intended.
12. Document the root cause, resolution, and lessons learned.

This approach helps avoid making unrelated configuration changes while troubleshooting and provides a repeatable method for isolating failures across cloud, network, identity, operating-system, and application layers.

---

# Future Troubleshooting Entries

Additional troubleshooting scenarios will be documented as meaningful issues are encountered during continued development of the environment.

Potential areas include:

- Additional Active Directory scenarios
- DNS
- Group Policy
- Azure Bastion
- Azure Firewall
- Azure Key Vault
- Azure Monitor
- Virtual Network routing
- Private Endpoints
- Microsoft Entra ID integration
- Application security
- Infrastructure automation

Future entries will document issues actually encountered during implementation rather than presenting planned services as completed troubleshooting scenarios.

---

# Summary

Troubleshooting within the Azure Enterprise Lab has included network connectivity, Active Directory and DNS integration, Windows resource authorization, and application availability validation.

The outbound-connectivity issue demonstrated how DNS, routing, Network Security Groups, subnet configuration, and explicit Internet egress must be evaluated independently.

The Active Directory implementation reinforced the dependency between domain services and DNS configuration.

The ITShare permission review identified and remediated an overly broad inherited NTFS permission that conflicted with the intended AGDLP-style authorization model.

The internal Load Balancer test demonstrated application availability by intentionally removing WEB01 from service and validating continued access through WEB02 using the same private frontend address.

These scenarios demonstrate a troubleshooting approach based on isolating layers, validating assumptions, identifying root causes, applying controlled remediation, and testing the final architecture rather than simply confirming that the original symptom disappeared.
