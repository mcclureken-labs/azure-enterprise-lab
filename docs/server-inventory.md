# Server Inventory

This document provides a high-level inventory of server resources deployed within the Enterprise Azure Lab.

The current server placement and network relationships are shown in the architecture topology below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

## Active Servers

| Server | Azure Resource           | Role                                   | Private IP | IP Allocation | Network Zone |
| ------ | ------------------------ | -------------------------------------- | ---------- | ------------- | ------------ |
| DC01   | vm-dc01-corp-prd-eus2   | Active Directory Domain Services / DNS | 10.1.0.4   | Static        | Identity     |
| MGMT01 | vm-mgmt01-corp-prd-eus2 | Domain Management Server               | 10.1.1.4   | Dynamic       | Management   |

---

## DC01

**Role:** Active Directory Domain Services and DNS

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Purpose:**

- Provides centralized identity and authentication services for the lab environment
- Hosts Active Directory Domain Services
- Hosts Active Directory-integrated DNS
- Provides DNS services for domain-joined systems
- Supports Active Directory authentication and service discovery

**Network Configuration:**

- Located within `snet-identity-prd-eus2`
- Subnet: `10.1.0.0/24`
- Static private IP addressing
- Private IP: `10.1.0.4`
- No direct public IP assigned
- Identity subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway

**Administrative Access:**

- Administrative access is performed through Azure Bastion
- Direct RDP exposure to the public Internet is not permitted
- Routine administrative workloads are separated from the Domain Controller through dedicated management infrastructure

---

## MGMT01

**Role:** Domain Management Server

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Domain:** `corp.mccluretech.com`

**Purpose:**

- Provides a dedicated system for infrastructure administration
- Provides a domain-joined administrative platform
- Separates routine management activities from the Domain Controller
- Will host remote administration tools for managing domain resources
- Provides the foundation for centralized Windows infrastructure administration

**Active Directory Configuration:**

- Joined to the `corp.mccluretech.com` domain
- Computer object located within the custom **Servers** OU
- Domain authentication successfully validated
- Uses Active Directory DNS hosted on DC01

**Network Configuration:**

- Located within `snet-corp-management-prd-eus2`
- Subnet: `10.1.1.0/24`
- Dynamic private IP addressing
- Current private IP: `10.1.1.4`
- No direct public IP assigned
- Corporate Management subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway
- Subnet protected by `nsg-corp-management-prd-eus2`

**Administrative Access:**

- Administrative access is performed through Azure Bastion
- Domain authentication is used for privileged administrative sessions
- Direct RDP exposure to the public Internet is not permitted

---

## Server Relationships

DC01 and MGMT01 are deployed within separate Corporate network segments to separate identity services from routine administrative activity.

DC01 provides Active Directory Domain Services and Active Directory-integrated DNS for the environment.

MGMT01 is joined to the `corp.mccluretech.com` domain and uses DC01 at `10.1.0.4` for DNS resolution and Active Directory service discovery.

Both servers remain privately addressed within the Corporate Virtual Network.

Administrative access is provided through Azure Bastion across VNet peering, while required outbound Internet connectivity is provided through Azure NAT Gateway rather than Azure default outbound access or direct public IP assignments.

---

## Design Considerations

Server roles are separated across dedicated network segments to support network segmentation and reduce unnecessary exposure.

The Domain Controller uses static private addressing because other systems rely on it for Active Directory-integrated DNS and domain services.

The Management Server uses dynamic private addressing because other infrastructure does not currently depend on it being reachable at a fixed IP address.

Domain services and management functions are hosted on separate virtual machines rather than combining routine administrative workloads with the Domain Controller.

MGMT01 is domain joined and provides a dedicated administrative platform for future remote management tooling and centralized infrastructure administration.

The Identity and Corporate Management subnets are configured as private subnets with explicit outbound connectivity provided through Azure NAT Gateway.

Azure Bastion provides administrative access without assigning public IP addresses directly to the server virtual machines.
