# Enterprise Azure Lab - Troubleshooting

## Overview

This document records technical issues encountered while building the Enterprise Azure Lab, including investigation steps, root causes, resolutions, validation, and lessons learned.

The goal is to document real-world troubleshooting scenarios encountered during the deployment and continued development of the environment.

The current network architecture is shown below for reference.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

# Issue #1 - VM Had No Outbound Internet Connectivity

## Symptoms

During the initial deployment of server infrastructure, a virtual machine was accessible through Azure Bastion but could not establish outbound HTTPS connectivity.

Observed behavior included:

- Azure Bastion connectivity was successful.
- The virtual machine received a valid private IP address.
- DNS resolution functioned correctly.
- Windows Server displayed an activation warning.
- Outbound HTTPS connectivity failed.

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

No User Defined Route (UDR) was associated with the affected subnet.

This ruled out a custom route table as the source of the failed outbound connection.

### Investigation Summary

At this point, the following had been established:

- Private IP configuration was valid.
- DNS resolution was functioning.
- An active default route existed.
- Network Security Group configuration permitted outbound traffic.
- No User Defined Route was redirecting or blocking the traffic.
- Azure Bastion connectivity to the virtual machine was functioning.
- Outbound TCP/443 connectivity was still failing.

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

However, reliance on default outbound access was intentionally treated as a **temporary configuration** rather than the desired long-term network architecture.

---

## Permanent Architectural Resolution

After the initial deployment work was completed, the outbound architecture was revisited to eliminate reliance on Azure default outbound access.

An Azure NAT Gateway was deployed to provide an explicit outbound connectivity path for the private workload subnets.

### NAT Gateway Resources

| Property           | Value                    |
| ------------------ | ------------------------ |
| NAT Gateway        | nat-corp-prd-eus2        |
| Resource Group     | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2    |

The NAT Gateway was associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

A dedicated static Azure Public IP resource was associated with the NAT Gateway to provide a predictable outbound egress address.

The assigned public IP address itself is intentionally excluded from the public repository.

---

## NAT Gateway Implementation

The permanent outbound connectivity implementation followed this sequence:

1. Deploy Azure NAT Gateway.
2. Create and associate the dedicated static Public IP resource `pip-nat-corp-prd-eus2`.
3. Associate the NAT Gateway with `snet-identity-prd-eus2`.
4. Associate the NAT Gateway with `snet-corp-management-prd-eus2`.
5. Validate outbound HTTPS connectivity.
6. Confirm that Internet-bound traffic used the NAT Gateway Public IP resource for egress.
7. Restore private subnet configuration.
8. Confirm that default outbound access remained disabled.
9. Revalidate outbound HTTPS connectivity from the private workload.

Azure NAT Gateway performs **Source Network Address Translation (SNAT)** for outbound connections originating from the associated private subnets.

The private source address of an outbound workload connection is translated to the static Public IP resource associated with the NAT Gateway.

This provides the workload subnets with an explicitly configured outbound Internet path without assigning public IP addresses directly to the server virtual machines.

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

- Azure Bastion administrative connectivity remained functional.
- Private IP addressing remained functional.
- DNS resolution remained functional.
- Outbound HTTPS connectivity succeeded.
- The Identity subnet remained configured as a private subnet.
- The Corporate Management subnet remained configured as a private subnet.
- Default outbound access was disabled for the private workload subnets.
- Azure NAT Gateway provided explicit outbound connectivity.
- Internet-bound traffic used the NAT Gateway Public IP resource for outbound SNAT.
- The observed public egress address matched the expected NAT Gateway Public IP resource.
- DC01 remained without a direct public IP assignment.
- MGMT01 remained without a direct public IP assignment.

The final network architecture and outbound connectivity path are represented in the topology diagram at the beginning of this document.

---

## Architectural Outcome

The troubleshooting process resulted in more than restoration of Internet connectivity.

The initial problem exposed an architectural requirement for explicitly defined outbound connectivity from private workload subnets.

Rather than leaving the temporary default outbound configuration in place, the environment was updated to use Azure NAT Gateway as the intended outbound egress mechanism.

The resulting design separates three different connectivity functions:

- **Administrative access** is provided through Azure Bastion.
- **Private workload communication** occurs through Azure Virtual Networks and VNet peering.
- **Outbound Internet connectivity** is provided through Azure NAT Gateway.

Server virtual machines remain privately addressed and do not require direct public IP assignments for administrative or outbound connectivity.

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

### Prefer Explicit Outbound Architecture

The final design uses Azure NAT Gateway to provide predictable outbound connectivity for private workload subnets without assigning direct public IP addresses to the server virtual machines.

The dedicated Public IP resource also provides a known egress resource for outbound connections.

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

# Future Troubleshooting Entries

Additional troubleshooting scenarios will be documented as the environment grows.

Potential areas include:

- Active Directory Domain Services
- DNS
- Domain joins
- Group Policy
- Azure Bastion
- Azure Firewall
- Azure Key Vault
- Azure Monitor
- Virtual Network Peering
- Microsoft Entra ID integration
