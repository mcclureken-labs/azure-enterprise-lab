# Enterprise Azure Lab - Network Design

**Version:** 1.5  
**Last Updated:** August 20, 2026  
**Author:** Kendrick McClure

---

## Overview

The Azure Enterprise Lab uses a Hub-and-Spoke network architecture to separate shared connectivity and administrative infrastructure from Corporate workload resources.

The Hub Virtual Network provides centralized connectivity, security, and administrative services, while the Corporate Virtual Network contains Identity, Management, and Internal Application workloads.

The current network design incorporates:

- Hub-and-Spoke network segmentation
- Bidirectional VNet peering
- Azure Bastion for private administrative connectivity
- Azure Firewall for centralized outbound traffic control
- User Defined Routes for centralized outbound routing
- Azure Monitor telemetry routed through centralized firewall egress
- Active Directory-integrated DNS
- Subnet-level Network Security Groups
- Dedicated Identity and Management networks
- A dedicated Internal Apps subnet
- Two redundant Nginx web servers
- An internal Azure Load Balancer
- Reserved address space for future gateway and Private Endpoint services

The current network topology is shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [../diagrams/azure-enterprise-lab-topology.drawio](../diagrams/azure-enterprise-lab-topology.drawio).

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
| AzureFirewallSubnet | 10.0.0.0/26 | Azure Firewall data traffic | Active |
| GatewaySubnet | 10.0.0.64/27 | Reserved for VPN / ExpressRoute Gateway | Reserved |
| AzureBastionSubnet | 10.0.0.128/26 | Azure Bastion | Active |
| AzureFirewallManagementSubnet | 10.0.0.192/26 | Azure Firewall Basic management traffic | Active |
| snet-hub-management-prd-eus2 | 10.0.2.0/24 | Shared management infrastructure | Active |

The Azure Firewall subnets provide dedicated network segments for Azure Firewall data and management traffic.

The Gateway subnet remains reserved for future hybrid-connectivity infrastructure.

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

Explicit outbound Internet connectivity for private Corporate workloads is routed through Azure Firewall in the Hub Virtual Network.

| Property | Value |
| --- | --- |
| Name | fw-hub-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Private IP | 10.0.0.4 |
| Firewall Tier | Basic |
| Data Subnet | AzureFirewallSubnet |
| Management Subnet | AzureFirewallManagementSubnet |

A User Defined Route is used to direct Internet-bound traffic from the active Corporate workload subnets to Azure Firewall.

| Property | Value |
| --- | --- |
| Route Table | rt-corporate-egress-prd-eus2 |
| Address Prefix | 0.0.0.0/0 |
| Next Hop Type | Virtual Appliance |
| Next Hop Address | 10.0.0.4 |

The route table is associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

These subnets are configured as private subnets with default outbound access disabled.

Internet-bound connections originating from workloads in these subnets are therefore directed to Azure Firewall for centralized policy enforcement.

Azure Firewall controls permitted outbound connectivity while the server workloads remain privately addressed and without direct public IP assignments.

## Outbound Connectivity Validation

Outbound routing and firewall policy enforcement were validated from MGMT01 after the User Defined Route and Azure Firewall configuration were implemented.

Testing confirmed that explicitly permitted Azure KMS activation traffic over TCP port 1688 successfully traversed the configured outbound path.

Unmatched outbound web traffic was also tested and denied by Azure Firewall because no firewall rule matched the request.

This validated both permitted outbound connectivity and default-deny behavior for traffic not explicitly allowed by firewall policy.

Azure Monitor connectivity was subsequently validated after adding narrowly scoped HTTPS/443 firewall policy for the required monitoring endpoints, with successful telemetry ingestion confirmed from both Windows and Linux workloads.

## Azure Monitor Egress

Azure Monitor Agent telemetry from the Corporate server workloads uses the existing centralized outbound path through Azure Firewall.

Firewall policy permits the required Azure Monitor communication over HTTPS/443 while maintaining default-deny behavior for outbound traffic that is not explicitly permitted.

This allows DC01, MGMT01, WEB01, and WEB02 to send monitoring telemetry to Azure Monitor and the configured Log Analytics workspace without requiring direct public IP addresses or unrestricted outbound Internet access.

---

# DNS Architecture

Active Directory-integrated DNS is hosted on DC01 at 10.1.0.4.

| Property | Value |
| --- | --- |
| DNS Server | DC01 |
| Private IP | 10.1.0.4 |
| AD DNS Domain | corp.mccluretech.com |

The Corporate Virtual Network is configured with 10.1.0.4 as its custom DNS server.

This VNet-level configuration allows workloads using the VNet-provided DNS configuration to receive DC01 as their DNS server.

MGMT01 therefore uses DC01 for internal DNS resolution and Active Directory service discovery.

The DNS configuration supports resolution of the corp.mccluretech.com namespace and the Active Directory service records required for domain authentication and directory operations.

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
| nsg-corporate-identity-prd-eus2 | snet-identity-prd-eus2 |
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
| Outbound Connectivity | Azure Firewall |
| Application Protocol | HTTP |
| Application Port | TCP/80 |

WEB01 and WEB02 run Nginx and remain privately addressed within the Corporate VNet.

Neither web server requires a direct public IP address.

The Internal Apps subnet uses Azure Firewall for controlled outbound Internet connectivity and an internal Azure Load Balancer for application delivery.

These two services perform different functions:

- Azure Load Balancer distributes internal application traffic to healthy backend servers.
- Azure Firewall controls outbound Internet connectivity originating from the subnet.

Separating these functions allows the application tier to remain private while still supporting controlled outbound connectivity.

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

The Load Balancer frontend uses the static private IP address 10.1.3.10.

Clients within the connected private network access the web service through this frontend rather than connecting directly to WEB01 or WEB02.

The Load Balancer backend pool contains the network interfaces associated with WEB01 and WEB02.

A TCP health probe on port 80 monitors backend availability.

The load-balancing rule maps frontend TCP port 80 to backend TCP port 80 and distributes application traffic only to backend instances considered healthy.

Because the Load Balancer is internal, it does not expose the Nginx application directly to the public Internet.

---

# Load Balancer Validation

The internal application tier was validated from MGMT01 through the Load Balancer frontend at 10.1.3.10.

An initial request to `http://10.1.3.10` successfully reached the Nginx application. WEB01 was then intentionally shut down to simulate a backend failure.

After the health mechanism detected that WEB01 was unavailable, another request to the same frontend was successfully served by WEB02.

This validated private frontend connectivity, backend health monitoring, failure detection, and continued application availability through the remaining healthy backend while clients continued using the same frontend address.

Detailed troubleshooting and validation scenarios are maintained in the [Troubleshooting](troubleshooting.md) documentation.

---

# Network Segmentation

The environment separates infrastructure according to function rather than placing all systems into a single network segment.

### Hub

- **Azure Bastion** - Centralized private administrative connectivity
- **Hub Management** - Shared management infrastructure
- **Azure Firewall** - Centralized outbound traffic filtering and policy enforcement
- **Azure Firewall Management** - Dedicated Azure Firewall Basic management traffic
- **Gateway** - Reserved subnet for future VPN or ExpressRoute connectivity

### Corporate

- **Identity** - Active Directory Domain Services and DNS
- **Management** - Administrative systems and management tooling
- **Private Endpoints** - Reserved for private connectivity to supported Azure services
- **Internal Applications** - Nginx application tier and internal Azure Load Balancer

This segmentation allows network controls, routing decisions, security policies, and connectivity services to be applied according to workload function.

It also provides a scalable structure for introducing additional workloads without placing unrelated infrastructure into the same network segment.

---

# Design Decisions

## Hub-and-Spoke Architecture

Shared connectivity and administrative services are separated from Corporate workload infrastructure.

This centralizes shared services within the Hub while maintaining a logical boundary between common infrastructure and workload resources. The design also provides a scalable foundation for adding additional spokes without duplicating shared connectivity services.

## Private Server Addressing

DC01, MGMT01, WEB01, and WEB02 do not have direct public IP addresses.

Keeping server workloads privately addressed reduces direct Internet exposure and allows administrative access, internal application delivery, and outbound connectivity to be provided through services designed specifically for those functions.

## Explicit Outbound Connectivity

Default outbound access is disabled on the active private Corporate subnets.

User Defined Routes direct Internet-bound traffic from the Corporate workload subnets to Azure Firewall in the Hub. Azure Firewall provides a centralized enforcement point where outbound connectivity can be explicitly permitted or denied according to firewall policy while the workloads remain privately addressed.

## Dedicated Identity Network

The Domain Controller is placed within a dedicated Identity subnet rather than sharing a subnet with routine administrative or application workloads.

Separating identity infrastructure creates a clearer security boundary around services that support authentication and DNS. It also provides a foundation for applying identity-specific network controls as the environment develops.

## Dedicated Management Network

MGMT01 is placed within a dedicated Corporate Management subnet and serves as the administrative platform for managing domain resources.

This keeps routine administrative activity separate from the Domain Controller and provides a dedicated network segment for management systems and tooling as the environment grows.

## Dedicated Internal Application Network

WEB01 and WEB02 are deployed within a dedicated Internal Apps subnet rather than sharing network space with identity or administrative infrastructure.

This isolates the application tier from other workload types and allows application-specific NSG rules, outbound connectivity, and load-balancing configuration to be managed independently.

## Internal Load Balancing

The Nginx workload is accessed through an internal Azure Load Balancer rather than by clients targeting individual backend servers.

The Load Balancer provides a consistent private application endpoint while allowing multiple backend servers to participate in the application tier. Health probing also allows unavailable backends to be removed from active service while healthy systems continue receiving traffic.

## Reserved Network Capacity

Dedicated address space remains reserved for gateway connectivity and Private Endpoints.

Reserving these network segments allows planned hybrid connectivity and private service-access capabilities to be introduced without redesigning the existing address plan or disrupting currently deployed workloads.

---

# Future Improvements

Planned network improvements include:

- Expanded network security rules
- Additional internal application workloads
- Private Endpoint implementation
- VPN or other hybrid connectivity using the reserved `GatewaySubnet`
- Network security integration with Microsoft Defender for Cloud
- Additional load-balancing and application availability scenarios
- Infrastructure automation using PowerShell, Bicep, and Terraform

---

# Summary

The Azure Enterprise Lab network currently uses a Hub-and-Spoke architecture that separates shared connectivity and security services from Corporate Identity, Management, and Internal Application workloads.

Azure Bastion provides private administrative connectivity across VNet peering, Active Directory-integrated DNS provides internal name resolution and domain service discovery, and Azure Firewall provides centralized outbound traffic control for private Corporate workloads through User Defined Routes. Azure Monitor Agent telemetry from Corporate server workloads also traverses the centralized Azure Firewall egress path over HTTPS/443.

The Internal Apps subnet hosts two Nginx backend servers behind an internal Azure Load Balancer with a static private frontend, TCP/80 health monitoring, and validated backend failover.

The current design demonstrates network segmentation, private workload addressing, centralized administrative access, controlled outbound connectivity, centralized firewall policy enforcement, internal application load balancing, backend health monitoring, and reserved capacity for future hybrid-connectivity services.
