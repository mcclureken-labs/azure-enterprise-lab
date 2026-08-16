# Enterprise Azure Lab - Network Design

**Version:** 1.3  
**Last Updated:** August 15, 2026  
**Author:** Kendrick McClure

---

## Overview

The Azure Enterprise Lab uses a Hub-and-Spoke network architecture to separate shared connectivity and administrative infrastructure from Corporate workload resources.

The Hub Virtual Network provides centralized connectivity and administrative services, while the Corporate Virtual Network contains Identity, Management, and Internal Application workloads.

The current network design incorporates:

- Hub-and-Spoke network segmentation
- Bidirectional VNet peering
- Azure Bastion for private administrative connectivity
- Azure NAT Gateway for explicit outbound Internet access
- Active Directory-integrated DNS
- Subnet-level Network Security Groups
- Dedicated Identity and Management networks
- A dedicated Internal Apps subnet
- Two redundant Nginx web servers
- An internal Azure Load Balancer
- Reserved address space for future Azure Firewall, gateway, and Private Endpoint services

The current network topology is shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

# Hub Virtual Network

| Property | Value |
| --- | --- |
| Name | vnet-hub-prd-eus2 |
| Address Space | 10.0.0.0/20 |
| Resource Group | rg-connectivity-prd-eus2 |

## Hub Subnets

| Subnet | Address Space | Purpose | Status |
| --- | --- | --- | --- |
| AzureFirewallSubnet | 10.0.0.0/26 | Reserved for Azure Firewall | Reserved |
| GatewaySubnet | 10.0.0.64/27 | Reserved for VPN / ExpressRoute Gateway | Reserved |
| AzureBastionSubnet | 10.0.0.128/26 | Azure Bastion | Active |
| snet-hub-management-prd-eus2 | 10.0.2.0/24 | Shared management infrastructure | Active |

The Azure Firewall and Gateway subnets are reserved for future infrastructure and do not currently host those services.

Reserving these network segments in advance provides space for future centralized security and hybrid-connectivity services without requiring the existing address plan to be redesigned.

---

# Corporate Virtual Network

| Property | Value |
| --- | --- |
| Name | vnet-corporate-prd-eus2 |
| Address Space | 10.1.0.0/20 |
| Resource Group | rg-corporate-prd-eus2 |

## Corporate Subnets

| Subnet | Address Space | Purpose | Status |
| --- | --- | --- | --- |
| snet-identity-prd-eus2 | 10.1.0.0/24 | Active Directory and DNS infrastructure | Active |
| snet-corp-management-prd-eus2 | 10.1.1.0/24 | Administrative management infrastructure | Active |
| snet-private-endpoints-prd-eus2 | 10.1.2.0/24 | Private Endpoints | Reserved |
| snet-corporate-internal-apps-prd-eus2 | 10.1.3.0/24 | Internal application workloads | Active |

The Identity, Management, and Internal Apps subnets host the currently deployed Corporate infrastructure.

The Private Endpoints subnet remains reserved for future private connectivity to supported Azure platform services.

## Current Server Placement

| Server | Role | Subnet | Private IP | Allocation |
| --- | --- | --- | --- | --- |
| DC01 | Active Directory Domain Services / DNS | snet-identity-prd-eus2 | 10.1.0.4 | Static |
| MGMT01 | Domain Management / Administration | snet-corp-management-prd-eus2 | 10.1.1.4 | Dynamic |
| WEB01 | Nginx Web Server | snet-corporate-internal-apps-prd-eus2 | 10.1.3.4 | Dynamic |
| WEB02 | Nginx Web Server | snet-corporate-internal-apps-prd-eus2 | 10.1.3.5 | Dynamic |

DC01 uses static private addressing because domain-connected systems depend on a consistent endpoint for DNS and directory services.

MGMT01 uses dynamic private addressing because other infrastructure does not currently depend on it maintaining a fixed address.

WEB01 and WEB02 also use dynamic private addressing because application clients access the service through the internal Load Balancer frontend rather than directly targeting individual backend server addresses.

---

# VNet Peering

The Hub and Corporate Virtual Networks are connected using bidirectional VNet peering.

- Hub → Corporate
- Corporate → Hub

Peering enables private IP communication between resources in the Hub and Corporate networks across the Microsoft Azure backbone.

This allows shared services hosted in the Hub, such as Azure Bastion, to reach privately addressed resources within the Corporate VNet without assigning public IP addresses directly to those workloads.

The peering relationship provides the connectivity foundation for centralized Hub services while maintaining logical separation between shared infrastructure and Corporate workloads.

---

# Azure Bastion

Azure Bastion is deployed within the `AzureBastionSubnet` in the Hub Virtual Network when administrative access to private virtual machines is required.

| Property | Value |
| --- | --- |
| Name | bas-hub-prd-eus2 |
| Virtual Network | vnet-hub-prd-eus2 |
| Subnet | AzureBastionSubnet |
| Public IP Resource | pip-bastion-hub-prd-eus2 |

Azure Bastion provides administrative connectivity to privately addressed virtual machines without assigning direct public IP addresses to the servers.

Administrative traffic reaches Corporate resources across the Hub-to-Corporate VNet peering connection.

This provides a centralized administrative access path while avoiding direct RDP or SSH exposure from the workload virtual machines to the public Internet.

Because Azure Bastion is a billable lab resource, it can be deployed or removed as required while the underlying private server architecture remains unchanged.

---

# Outbound Internet Connectivity

Explicit outbound Internet connectivity for private Corporate workloads is provided through Azure NAT Gateway.

| Property | Value |
| --- | --- |
| Name | nat-corp-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2 |

The NAT Gateway is associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

These subnets are configured as private subnets with default outbound access disabled.

Internet-bound connections originating from workloads in these subnets use the NAT Gateway as their explicit outbound path.

Azure NAT Gateway performs Source Network Address Translation (SNAT) using the dedicated static Public IP resource `pip-nat-corp-prd-eus2`.

This allows DC01, MGMT01, WEB01, and WEB02 to initiate outbound Internet connections when required while remaining privately addressed and without receiving direct public IP assignments.

The NAT Gateway does not provide unsolicited inbound Internet connectivity to these virtual machines.

## Outbound Connectivity Validation

Outbound connectivity was validated from private workloads after the NAT Gateway and subnet associations were configured.

Testing included successful Internet connectivity required for package and operating-system updates while the workloads remained privately addressed.

The NAT Gateway was also used by the Internal Apps subnet to provide WEB01 and WEB02 with explicit outbound connectivity required for Linux package installation and Nginx deployment.

The assigned NAT Gateway public IP address itself is intentionally excluded from repository documentation.

---

# DNS Architecture

Active Directory-integrated DNS is hosted on DC01 at `10.1.0.4`.

| Property | Value |
| --- | --- |
| DNS Server | DC01 |
| Private IP | 10.1.0.4 |
| AD DNS Domain | corp.mccluretech.com |

The Corporate Virtual Network is configured with `10.1.0.4` as its custom DNS server.

This VNet-level configuration allows workloads using the VNet-provided DNS configuration to receive DC01 as their DNS server.

MGMT01 therefore uses DC01 for internal DNS resolution and Active Directory service discovery.

The DNS configuration supports resolution of the `corp.mccluretech.com` namespace and the Active Directory service records required for domain authentication and directory operations.

The DNS relationship is represented separately in the architecture topology because the Corporate VNet DNS configuration and the DNS service running on DC01 represent two distinct layers of the design:

- The Corporate VNet defines which DNS server workloads should use.
- DC01 hosts the Active Directory-integrated DNS service at that configured address.

---

# Network Security Groups

Network Security Groups are used to apply subnet-level network controls throughout the environment.

Current NSG assignments include:

| Network Security Group | Associated Subnet |
| --- | --- |
| nsg-hub-management-prd-eus2 | snet-hub-management-prd-eus2 |
| nsg-corp-management-prd-eus2 | snet-corp-management-prd-eus2 |
| nsg-corporate-internal-apps-prd-eus2 | snet-corporate-internal-apps-prd-eus2 |

NSGs provide network-layer traffic filtering based on source, destination, protocol, and port.

The Internal Apps NSG permits the traffic required for the internal web workload while maintaining the subnet as a privately addressed application segment.

HTTP traffic on TCP port 80 is used by the current Nginx application tier and Azure Load Balancer configuration.

HTTPS on TCP port 443 is reserved within the network design for future TLS-enabled application traffic but is not currently implemented by the Nginx workload.

Additional network security controls will be introduced as more workloads and centralized security services are deployed.

---

# Internal Application Network

The `snet-corporate-internal-apps-prd-eus2` subnet provides a dedicated network segment for internal application workloads.

| Property | Value |
| --- | --- |
| Subnet | snet-corporate-internal-apps-prd-eus2 |
| Address Space | 10.1.3.0/24 |
| WEB01 | 10.1.3.4 |
| WEB02 | 10.1.3.5 |
| Load Balancer Frontend | 10.1.3.10 |
| Outbound Connectivity | Azure NAT Gateway |
| Application Protocol | HTTP |
| Application Port | TCP/80 |

WEB01 and WEB02 run Nginx and remain privately addressed within the Corporate VNet.

Neither web server requires a direct public IP address.

The Internal Apps subnet uses Azure NAT Gateway for explicit outbound Internet connectivity and an internal Azure Load Balancer for application delivery.

These two services perform different functions:

- Azure Load Balancer distributes internal application traffic to healthy backend servers.
- Azure NAT Gateway provides outbound Internet connectivity originating from the subnet.

Separating these functions allows the application tier to remain private while still supporting outbound package installation, updates, and other required Internet-bound connections.

---

# Internal Azure Load Balancer

An internal Azure Load Balancer provides a stable private frontend for the Nginx application tier.

| Property | Value |
| --- | --- |
| Name | lb-corporate-internal-apps-prd-eus2 |
| Type | Internal |
| Frontend Private IP | 10.1.3.10 |
| Frontend Allocation | Static |
| Backend Servers | WEB01, WEB02 |
| Application Protocol | TCP |
| Application Port | 80 |
| Health Probe | TCP/80 |

The Load Balancer frontend uses the static private IP address `10.1.3.10`.

Clients within the connected private network access the web service through this frontend rather than connecting directly to WEB01 or WEB02.

The Load Balancer backend pool contains the network interfaces associated with WEB01 and WEB02.

A TCP health probe on port 80 monitors backend availability.

The load-balancing rule maps frontend TCP port 80 to backend TCP port 80 and distributes application traffic only to backend instances considered healthy.

Because the Load Balancer is internal, it does not expose the Nginx application directly to the public Internet.

---

# Load Balancer Validation

The internal application tier was tested from MGMT01 using the Load Balancer frontend address.

An HTTP request to:

`http://10.1.3.10`

successfully returned the Nginx response hosted by WEB01.

WEB01 was then intentionally shut down to simulate backend failure.

After the health mechanism detected that WEB01 was unavailable, another request to the same Load Balancer frontend successfully returned the response hosted by WEB02.

This validated:

- Private connectivity to the Load Balancer frontend
- Backend pool configuration
- TCP/80 application connectivity
- Health probe operation
- Backend availability detection
- Continued service through the remaining healthy backend
- Use of a consistent frontend address during backend failure

The test demonstrates basic internal application resiliency without exposing the backend servers directly to the Internet.

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
- **Internal Applications** - Nginx application tier and internal Azure Load Balancer

This segmentation allows network controls, routing decisions, security policies, and connectivity services to be applied according to workload function.

It also provides a scalable structure for introducing additional workloads without placing unrelated infrastructure into the same network segment.

---

# Traffic Flow

The environment currently uses several distinct traffic paths depending on the purpose of the connection.

## Administrative Traffic

Administrative access originates through Azure Bastion in the Hub VNet and reaches privately addressed Corporate workloads across VNet peering.

Direct administrative ports are not exposed from the server virtual machines to the public Internet.

## Active Directory and DNS Traffic

Domain-connected systems use DC01 at `10.1.0.4` for internal DNS resolution and Active Directory service discovery.

The Corporate VNet custom DNS configuration distributes this DNS server setting to workloads using VNet-provided DNS configuration.

## Internal Application Traffic

Internal clients access the Nginx application through the internal Load Balancer frontend at `10.1.3.10`.

The Load Balancer distributes TCP/80 traffic to healthy WEB01 and WEB02 backend instances.

## Outbound Internet Traffic

Internet-bound connections originating from the Identity, Management, and Internal Apps subnets use Azure NAT Gateway for explicit outbound connectivity.

The NAT Gateway performs SNAT using its dedicated static Public IP resource.

These traffic paths intentionally separate administrative access, internal application delivery, directory services, and outbound Internet connectivity.

---

# Design Decisions

## Hub-and-Spoke Architecture

Shared connectivity and administrative services are separated from Corporate workload infrastructure.

This provides a foundation that can scale to additional spokes while allowing shared services to remain centralized.

## Private Server Addressing

DC01, MGMT01, WEB01, and WEB02 do not have direct public IP addresses.

Administrative connectivity is provided through Azure Bastion when required, internal application delivery is provided through an internal Azure Load Balancer, and required outbound Internet connectivity is provided separately through Azure NAT Gateway.

## Explicit Outbound Connectivity

Default outbound access is disabled on the active private Corporate subnets.

Azure NAT Gateway provides an explicitly configured outbound path and predictable public egress resource for Internet-bound connections originating from those subnets.

## Dedicated Identity Network

The Domain Controller is placed within a dedicated Identity subnet rather than sharing a subnet with routine administrative or application workloads.

This creates clearer infrastructure boundaries and provides a foundation for applying more specific security controls as the environment develops.

## Dedicated Management Network

MGMT01 is placed within a dedicated Corporate Management subnet and serves as the administrative platform for managing domain resources.

This separates routine administrative activity from the Domain Controller itself.

## Dedicated Internal Application Network

WEB01 and WEB02 are deployed within a dedicated Internal Apps subnet rather than sharing network space with identity or administrative infrastructure.

This allows application-specific NSG rules, outbound connectivity, and load-balancing configuration to be managed independently from other workload types.

## Internal Load Balancing

The Nginx workload is accessed through an internal Azure Load Balancer rather than by clients targeting individual backend servers.

The Load Balancer provides a stable private frontend while allowing multiple backend systems to participate in the application tier.

Health probing allows unavailable backend instances to be removed from active service while healthy instances continue receiving traffic.

## Separation of Inbound and Outbound Connectivity

The internal Load Balancer and NAT Gateway serve separate networking functions.

The Load Balancer handles internal application delivery to WEB01 and WEB02.

The NAT Gateway handles Internet-bound connections originating from the Internal Apps subnet.

This separation avoids using public addressing for the application workload while still providing the outbound connectivity required by the servers.

## Reserved Network Capacity

Dedicated address space remains reserved for Azure Firewall, gateway connectivity, and Private Endpoints.

These network segments are represented in the topology but clearly identified as reserved where the associated workloads or services have not yet been deployed.

---

# Design Goals

- Centralized networking using a Hub-and-Spoke architecture
- Secure administrative access through Azure Bastion
- No direct public IP addresses assigned to server virtual machines
- Explicit outbound Internet connectivity through Azure NAT Gateway
- Private Corporate workload subnets with default outbound access disabled
- Predictable public egress for Internet-bound workloads
- Logical separation of infrastructure roles
- Dedicated network segments for identity, management, and internal application systems
- Internal application delivery through a private Azure Load Balancer
- Backend health monitoring and application failover capability
- Subnet-level traffic control using Network Security Groups
- Active Directory-integrated DNS for Windows infrastructure
- Consistent Azure resource naming
- Reserved network capacity for future centralized security and connectivity services
- Scalable network design for future services and workloads

---

# Future Improvements

Planned network improvements include:

- Azure Firewall deployment within the reserved `AzureFirewallSubnet`
- User Defined Routes for centralized traffic inspection
- Expanded network security rules
- Additional internal application workloads
- Private Endpoint implementation
- VPN or other hybrid connectivity using the reserved `GatewaySubnet`
- Centralized network monitoring and logging
- Network security integration with Microsoft Defender for Cloud
- Additional load-balancing and application availability scenarios
- Infrastructure automation using PowerShell, Bicep, and Terraform

---

# Summary

The Azure Enterprise Lab network currently uses a Hub-and-Spoke architecture that separates shared connectivity services from Corporate Identity, Management, and Internal Application workloads.

Azure Bastion provides private administrative connectivity across VNet peering, Active Directory-integrated DNS provides internal name resolution and domain service discovery, and Azure NAT Gateway provides explicit outbound Internet connectivity for private Corporate workloads.

The Internal Apps subnet hosts two Nginx backend servers behind an internal Azure Load Balancer with a static private frontend, TCP/80 health monitoring, and validated backend failover.

The current design demonstrates network segmentation, private workload addressing, centralized administrative access, explicit outbound connectivity, internal application load balancing, backend health monitoring, and reserved capacity for future centralized security and hybrid-connectivity services.
