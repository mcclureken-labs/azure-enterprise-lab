# Enterprise Azure Lab - Network Design

## Overview

The Enterprise Azure Lab uses a Hub-and-Spoke network architecture to separate shared infrastructure from workload resources while providing centralized connectivity, management, and security.

The Hub Virtual Network contains shared connectivity and administrative services, while the Corporate Virtual Network contains workload-specific Identity and Management infrastructure.

The current network topology is shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

# Hub Virtual Network

| Property       | Value                    |
| -------------- | ------------------------ |
| Name           | vnet-hub-prd-eus2        |
| Address Space  | 10.0.0.0/20              |
| Resource Group | rg-connectivity-prd-eus2 |

## Hub Subnets

| Subnet                       | Address Space | Purpose                                 | Status   |
| ---------------------------- | ------------- | --------------------------------------- | -------- |
| AzureFirewallSubnet          | 10.0.0.0/26   | Reserved for Azure Firewall             | Reserved |
| GatewaySubnet                | 10.0.0.64/27  | Reserved for VPN / ExpressRoute Gateway | Reserved |
| AzureBastionSubnet           | 10.0.0.128/26 | Azure Bastion                           | Active   |
| snet-hub-management-prd-eus2 | 10.0.2.0/24   | Shared management infrastructure        | Active   |

The Azure Firewall and gateway subnets are reserved for future infrastructure and do not currently host those services.

---

# Corporate Virtual Network

| Property       | Value                   |
| -------------- | ----------------------- |
| Name           | vnet-corporate-prd-eus2 |
| Address Space  | 10.1.0.0/20             |
| Resource Group | rg-corporate-prd-eus2   |

## Corporate Subnets

| Subnet                          | Address Space | Purpose                                 | Status   |
| ------------------------------- | ------------- | --------------------------------------- | -------- |
| snet-identity-prd-eus2          | 10.1.0.0/24   | Active Directory and DNS infrastructure | Active   |
| snet-corp-management-prd-eus2   | 10.1.1.0/24   | Administrative management servers       | Active   |
| snet-private-endpoints-prd-eus2 | 10.1.2.0/24   | Private Endpoints                       | Reserved |
| snet-internal-apps-prd-eus2     | 10.1.3.0/24   | Internal application workloads          | Reserved |

The Private Endpoint and Internal Application subnets are currently reserved for future workloads.

## Current Server Placement

| Server | Role                                   | Subnet                       | Private IP | Allocation |
| ------ | -------------------------------------- | ---------------------------- | ---------- | ---------- |
| DC01   | Active Directory Domain Services / DNS | snet-identity-prd-eus2       | 10.1.0.4   | Static     |
| MGMT01 | Management Server                      | snet-corp-management-prd-eus2 | 10.1.1.4  | Dynamic    |

DC01 uses static private addressing because domain-joined systems depend on reliable access to DNS and directory services.

MGMT01 uses dynamic private addressing because other infrastructure does not depend on it maintaining a fixed address.

---

# VNet Peering

The Hub and Corporate Virtual Networks are connected using bidirectional VNet Peering.

- Hub → Corporate
- Corporate → Hub

Peering enables private IP communication between resources in the Hub and Corporate networks across the Microsoft Azure backbone.

This allows shared services hosted in the Hub, such as Azure Bastion, to reach privately addressed resources in the Corporate VNet without exposing those virtual machines directly to the public Internet.

---

# Azure Bastion

Azure Bastion is deployed within the `AzureBastionSubnet` in the Hub Virtual Network.

| Property           | Value                        |
| ------------------ | ---------------------------- |
| Name               | bas-hub-prd-eus2             |
| Virtual Network    | vnet-hub-prd-eus2            |
| Subnet             | AzureBastionSubnet           |
| Public IP Resource | pip-bastion-hub-prd-eus2     |

Azure Bastion provides administrative connectivity to privately addressed server resources without assigning public IP addresses directly to DC01 or MGMT01.

Administrative traffic reaches Corporate resources across the Hub-to-Corporate VNet peering connection.

This creates a centralized administrative access path while avoiding direct RDP exposure from the server virtual machines to the public Internet.

---

# Outbound Internet Connectivity

Explicit outbound Internet connectivity for the Identity and Corporate Management subnets is provided through Azure NAT Gateway.

| Property           | Value                    |
| ------------------ | ------------------------ |
| Name               | nat-corp-prd-eus2        |
| Resource Group     | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2    |

The NAT Gateway is associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

Both subnets are configured as private subnets with default outbound access disabled.

Internet-bound connections originating from these subnets use the NAT Gateway as their explicit outbound path. Azure NAT Gateway performs Source Network Address Translation (SNAT) using the dedicated static Public IP resource `pip-nat-corp-prd-eus2`.

This allows DC01 and MGMT01 to initiate outbound Internet connections when required while remaining privately addressed and without receiving direct public IP assignments.

Inbound Internet connections are not provided to the server virtual machines through the NAT Gateway.

## Validation

Outbound connectivity was validated by confirming successful HTTPS connectivity from MGMT01 and verifying that Internet-bound traffic used the NAT Gateway public IP for egress.

The assigned public IP address itself is intentionally excluded from repository documentation.

---

# DNS Architecture

Active Directory-integrated DNS is hosted on DC01 at `10.1.0.4`.

| Property       | Value                |
| -------------- | -------------------- |
| DNS Server     | DC01                 |
| Private IP     | 10.1.0.4             |
| AD DNS Domain  | corp.mccluretech.com |

The Corporate Virtual Network is configured with `10.1.0.4` as its custom DNS server.

This VNet-level configuration allows workloads using the VNet-provided DNS configuration to receive DC01 as their DNS server. MGMT01 therefore uses DC01 for DNS resolution and Active Directory service discovery.

This enables domain-joined systems to locate Active Directory services and resolve records within the `corp.mccluretech.com` domain.

The DNS configuration relationship is represented separately in the architecture topology because the Corporate VNet DNS setting and the DNS service running on DC01 represent two different layers of the design:

- The Corporate VNet defines which DNS server workloads should use.
- DC01 hosts the Active Directory-integrated DNS service at that configured address.

---

# Network Security Groups

Network Security Groups are used to apply subnet-level network controls within the environment.

Current documented NSG assignments include:

| Network Security Group         | Associated Subnet                |
| ------------------------------ | -------------------------------- |
| nsg-hub-management-prd-eus2    | snet-hub-management-prd-eus2     |
| nsg-corp-management-prd-eus2   | snet-corp-management-prd-eus2    |

NSGs provide network-layer traffic filtering based on source, destination, protocol, and port.

Additional security controls will be introduced as more workloads and centralized security services are deployed.

---

# Network Segmentation

The environment separates infrastructure according to function rather than placing all systems into a single network segment.

### Hub

- **Azure Bastion** - Centralized private administrative connectivity
- **Hub Management** - Shared management infrastructure
- **Azure Firewall** - Reserved subnet for future centralized traffic filtering
- **Gateway** - Reserved subnet for future VPN or ExpressRoute connectivity

### Corporate

- **Identity** - Active Directory Domain Services and DNS
- **Management** - Administrative systems and management tooling
- **Private Endpoints** - Reserved for private connectivity to supported Azure services
- **Internal Applications** - Reserved for internal application workloads

This segmentation allows network controls, routing decisions, and future security policies to be applied according to workload function.

---

# Design Decisions

## Hub-and-Spoke Architecture

Shared connectivity and administrative services are separated from Corporate workload infrastructure.

This provides a foundation that can scale to additional spokes while allowing shared services to remain centralized.

## Private Server Addressing

DC01 and MGMT01 do not have direct public IP addresses.

Administrative connectivity is provided through Azure Bastion, and required outbound Internet connectivity is provided separately through Azure NAT Gateway.

## Explicit Outbound Connectivity

Default outbound access is disabled on the Identity and Corporate Management subnets.

Azure NAT Gateway provides an explicitly configured outbound path and a predictable public egress resource for Internet-bound connections originating from those subnets.

## Dedicated Identity Network

The Domain Controller is placed within a dedicated Identity subnet rather than sharing a subnet with routine administrative workloads.

This creates clearer infrastructure boundaries and provides a foundation for applying more specific security controls as the environment develops.

## Dedicated Management Network

MGMT01 is placed within a dedicated Corporate Management subnet and serves as the administrative platform for managing domain resources.

This separates routine administrative activity from the Domain Controller itself.

## Reserved Network Capacity

Dedicated address space has been reserved for Azure Firewall, gateway connectivity, Private Endpoints, and internal applications.

These network segments are represented in the topology but clearly identified as reserved where the associated workloads or services have not yet been deployed.

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
