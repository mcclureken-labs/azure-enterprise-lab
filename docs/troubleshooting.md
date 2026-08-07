# Enterprise Azure Lab - Troubleshooting

## Overview

This document records technical issues encountered while building the Enterprise Azure Lab, including investigation steps, root causes, resolutions, and lessons learned.

The goal is to document real-world troubleshooting scenarios encountered during the deployment and continued development of the environment.

---

# Issue #1 - VM Had No Outbound Internet Connectivity

## Symptoms

The virtual machine was accessible through Azure Bastion but could not establish outbound HTTPS connectivity.

Observed behavior included:

- Azure Bastion connectivity was successful.
- DNS resolution functioned correctly.
- Windows Server displayed an activation warning.
- Outbound HTTPS connectivity failed.

The following PowerShell test was used:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : False
```

---

## Investigation

The troubleshooting process validated each major component of the network path.

### Private IP Configuration

The virtual machine received a valid private IP address within its assigned subnet.

### DNS Resolution

DNS resolution was tested using:

```powershell
nslookup www.microsoft.com
```

Name resolution completed successfully, indicating that DNS was functioning.

### Effective Routes

Azure effective routes were reviewed and contained an active default route:

```text
0.0.0.0/0
Next Hop: Internet
State: Active
```

### Network Security Group

The applicable Network Security Group configuration permitted outbound Internet traffic.

### Route Table

No User Defined Route (UDR) was associated with the affected subnet.

These checks indicated that DNS, routing, and Network Security Group configuration were not preventing the connection.

---

## Root Cause

The affected subnet was configured as a private subnet with default outbound access disabled.

At that stage of the deployment, no explicit outbound connectivity method had been configured for the subnet.

As a result, the virtual machine had no mechanism for establishing outbound Internet connections despite functioning private networking, DNS resolution, and Azure Bastion connectivity.

---

## Temporary Resolution

During the initial deployment, default outbound connectivity was temporarily enabled to allow required Internet access while the environment was being configured.

After changing the subnet configuration, the virtual machine was stopped and deallocated and then started again.

Steps performed:

1. Temporarily permit default outbound connectivity for the affected subnet.
2. Stop and deallocate the virtual machine.
3. Start the virtual machine.
4. Reconnect through Azure Bastion.
5. Test outbound HTTPS connectivity.

Validation command:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Successful result:

```text
TcpTestSucceeded : True
```

This restored Internet connectivity and allowed deployment activities to continue.

However, reliance on default outbound access was treated as a temporary configuration rather than the intended long-term network design.

---

## Permanent Architectural Resolution

An Azure NAT Gateway was later deployed to provide an explicit outbound connectivity path for private workloads.

The NAT Gateway was associated with the Identity and Corporate Management subnets.

The implementation followed this sequence:

1. Deploy an Azure NAT Gateway.
2. Assign a dedicated public IP resource to the NAT Gateway.
3. Associate the NAT Gateway with the required workload subnets.
4. Validate outbound HTTPS connectivity.
5. Confirm that Internet-bound traffic used the NAT Gateway public egress address.
6. Restore private subnet configuration.
7. Disable reliance on default outbound access.
8. Revalidate outbound HTTPS connectivity.

After private subnet configuration was restored, the following test continued to succeed:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : True
```

The observed Internet-facing source address was also verified to match the NAT Gateway public IP resource.

The actual public IP address is intentionally excluded from repository documentation.

---

## Final Traffic Flow

```text
Private Virtual Machine
        |
        v
Private Subnet
        |
        v
Azure NAT Gateway
        |
        v
Dedicated Public Egress IP
        |
        v
Internet
```

Server virtual machines remain privately addressed and do not require direct public IP assignments for outbound Internet connectivity.

---

## Lessons Learned

### DNS Resolution Does Not Prove Internet Connectivity

Successful DNS resolution only confirms that a hostname can be resolved to an IP address.

Testing the destination TCP port separately helped identify that name resolution was functioning while outbound HTTPS connectivity was not.

### Troubleshoot Network Connectivity Layer by Layer

Validating private addressing, DNS, effective routes, Network Security Groups, route tables, and application-layer connectivity independently helped isolate the actual problem.

### Temporary Fixes Should Be Revisited

Enabling default outbound connectivity allowed deployment work to continue, but it was intentionally treated as a temporary workaround.

Once the underlying requirement was understood, the environment was redesigned with an explicit outbound connectivity method.

### Prefer Explicit Outbound Architecture

The final design uses Azure NAT Gateway to provide predictable outbound connectivity for private workload subnets without assigning direct public IP addresses to the server virtual machines.

NAT Gateway provides outbound address translation but does not replace a firewall or provide application-layer traffic inspection.

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
