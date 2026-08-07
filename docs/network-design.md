# Enterprise Azure Lab - Network Design

## Overview

The Enterprise Azure Lab uses a Hub-and-Spoke network architecture to separate shared infrastructure from workload resources while providing centralized connectivity, management, and security.

---

# Hub Virtual Network

| Property       | Value                    |
| -------------- | ------------------------ |
| Name           | vnet-hub-prd-eus2        |
| Address Space  | 10.0.0.0/20              |
| Resource Group | rg-connectivity-prd-eus2 |

## Hub Subnets

| Subnet                       | Address Space | Purpose                                 |
| ---------------------------- | ------------- | --------------------------------------- |
| AzureFirewallSubnet          | 10.0.0.0/26   | Reserved for Azure Firewall             |
| GatewaySubnet                | 10.0.0.64/27  | Reserved for VPN / ExpressRoute Gateway |
| AzureBastionSubnet           | 10.0.0.128/26 | Azure Bastion                           |
| snet-hub-management-prd-eus2 | 10.0.2.0/24   | Shared management infrastructure        |

---

# Corporate Virtual Network

| Property       | Value                   |
| -------------- | ----------------------- |
| Name           | vnet-corporate-prd-eus2 |
| Address Space  | 10.1.0.0/20             |
| Resource Group | rg-corporate-prd-eus2   |

## Corporate Subnets

| Subnet                          | Address Space | Purpose                                 |
| ------------------------------- | ------------- | --------------------------------------- |
| snet-identity-prd-eus2          | 10.1.0.0/24   | Active Directory and DNS infrastructure |
| snet-corp-management-prd-eus2   | 10.1.1.0/24   | Administrative management servers       |
| snet-private-endpoints-prd-eus2 | 10.1.2.0/24   | Private Endpoints                       |
| snet-internal-apps-prd-eus2     | 10.1.3.0/24   | Internal application workloads          |

## Current Server Placement

| Server | Role                                   | Subnet               | Private IP | Allocation |
| ------ | -------------------------------------- | -------------------- | ---------- | ---------- |
| DC01   | Active Directory Domain Services / DNS | Identity             | 10.1.0.4   | Static     |
| MGMT01 | Management Server                      | Corporate Management | 10.1.1.4   | Dynamic    |

DC01 uses static private addressing because domain-joined systems depend on reliable access to DNS and directory services.

MGMT01 uses dynamic private addressing because other infrastructure does not depend on it maintaining a fixed address.

---

# VNet Peering

The Hub and Corporate virtual networks are connected using bidirectional VNet Peering.

- Hub → Corporate
- Corporate → Hub

Peering enables private communication between resources in the Hub and Corporate networks using the Azure backbone.

---

# Azure Bastion

Azure Bastion is deployed within the Hub Virtual Network and provides administrative connectivity to server resources without assigning public IP addresses directly to the virtual machines.

## Management Path

```text
Administrator
     |
     v
Azure Bastion
     |
     v
Hub VNet
     |
     v
VNet Peering
     |
     v
Corporate VNet
     |
     +---- Identity Subnet ------ DC01
     |
     +---- Management Subnet ---- MGMT01
```

---

# Outbound Internet Connectivity

Explicit outbound Internet connectivity for the Identity and Corporate Management subnets is provided through an Azure NAT Gateway.

| Property       | Value                    |
| -------------- | ------------------------ |
| Name           | nat-corp-prd-eus2        |
| Resource Group | rg-connectivity-prd-eus2 |
| SKU            | StandardV2               |
| Public IP      | pip-nat-corp-prd-eus2    |

The NAT Gateway is currently associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

Both subnets are configured as private subnets with default outbound access disabled.

This provides an explicitly defined outbound path for workloads that require Internet connectivity while allowing server virtual machines to remain privately addressed without direct public IP assignments.

## Outbound Traffic Path

```text
DC01 / MGMT01
      |
      v
Corporate Private Subnet
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

Outbound connectivity was validated by confirming successful HTTPS connectivity from MGMT01 and verifying that Internet-bound traffic was translated through the NAT Gateway public IP.

The public IP address itself is intentionally excluded from repository documentation.

---

# DNS Architecture

The Corporate Virtual Network uses the Active Directory DNS service hosted on DC01 for DNS resolution.

| Property   | Value                    |
| ---------- | ------------------------ |
| DNS Server | DC01                     |
| Private IP | 10.1.0.4                 |
| DNS Zone   | corp.mccluretech.com      |

Corporate workloads receive DC01 as their DNS server through the Virtual Network DNS configuration.

This enables domain-joined systems to locate Active Directory services and resolve internal domain records. External DNS requests can be resolved through the DNS server's configured forwarding and recursive resolution behavior.

---

# Network Segmentation

The Corporate Virtual Network separates infrastructure according to function:

- **Identity** - Active Directory and DNS infrastructure
- **Management** - Administrative systems and management tooling
- **Private Endpoints** - Private connectivity to supported Azure services
- **Internal Applications** - Internal application workloads

This design allows security controls and routing decisions to be applied according to workload function rather than placing all infrastructure within a single subnet.

---

# Current Network Architecture

```text
                         Internet
                            ^
                            |
                     NAT Gateway
                            ^
                            |
                  Corporate Private Subnets
                            |
          +-----------------+-----------------+
          |                                   |
     Identity Subnet                    Management Subnet
          |                                   |
        DC01                                MGMT01
          |                                   |
          +---------- Corporate VNet ----------+
                         |
                    VNet Peering
                         |
                       Hub VNet
                         |
                   Azure Bastion
                         ^
                         |
                   Administrator
```

---

# Design Goals

- Centralized networking using a Hub-and-Spoke architecture
- Secure administrative access through Azure Bastion
- No direct public IP addresses assigned to server virtual machines
- Explicit outbound Internet connectivity through Azure NAT Gateway
- Private Identity and Management subnets with default outbound access disabled
- Predictable public egress for Internet-bound workloads
- Logical separation of infrastructure roles
- Dedicated network segments for identity and management systems
- Consistent Azure resource naming
- Scalable network design for future services and workloads
