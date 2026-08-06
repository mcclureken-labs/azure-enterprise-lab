# Enterprise Network Security Design

## Overview

This document defines the initial network security approach for the Azure Enterprise Lab.

The security design follows least-privilege principles. Network access will be granted only when a documented technical or business requirement exists.

## Security Ownership

Security policies and Network Security Groups are maintained separately from the shared connectivity resources.

| Resource Group | Purpose |
|---|---|
| `rg-connectivity-prd-eus2` | Shared networking infrastructure |
| `rg-security-prd-eus2` | Network security policies and related security resources |

The Network Security Group can reside in the security resource group while protecting a subnet in the connectivity resource group.

## Management Subnet

### Protected Subnet

- **Virtual Network:** `vnet-hub-prd-eus2`
- **Subnet:** `snet-management-prd-eus2`
- **Address Range:** `10.0.2.0/24`
- **Network Security Group:** `nsg-management-prd-eus2`

## Security Requirements

The management subnet will follow these requirements:

1. Direct RDP access from the public internet is prohibited.
2. Direct SSH access from the public internet is prohibited.
3. Administrative access will eventually be provided through Azure Bastion.
4. Custom access rules must use the narrowest practical source, destination, protocol, and port.
5. Broad `Any` source or destination rules should be avoided.
6. New rules must have a documented purpose.
7. Unmatched inbound traffic remains denied.
8. Outbound access will be reviewed and restricted as the environment develops.

## Current NSG Configuration

No custom security rules are currently required.

The default NSG rules provide the initial baseline:

### Inbound

- Allow traffic originating from the virtual network.
- Allow Azure Load Balancer health probe traffic.
- Deny all other unmatched inbound traffic.

### Outbound

- Allow traffic within the virtual network.
- Allow outbound internet traffic.
- Deny all other unmatched outbound traffic.

The outbound internet rule will be reviewed when management workloads are deployed.

## Planned Administrative Access

Azure Bastion will eventually provide browser-based RDP and SSH access to management resources without assigning public IP addresses to the target virtual machines.

When Bastion is deployed, the management NSG will allow:

- TCP 3389 from `AzureBastionSubnet` to approved Windows management systems.
- TCP 22 from `AzureBastionSubnet` to approved Linux management systems.

No direct public RDP or SSH rule will be created.

## Future Security Controls

Planned controls include:

- Azure Bastion
- Workload-specific Network Security Groups
- Application Security Groups
- Route tables
- Centralized traffic inspection
- Azure Policy
- Microsoft Defender for Cloud
- Network monitoring and flow analysis
