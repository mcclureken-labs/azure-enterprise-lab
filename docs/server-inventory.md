# Server Inventory

This document provides a high-level inventory of server resources deployed within the Enterprise Azure Lab.

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
- Supports Active Directory domain authentication and service discovery

**Network Configuration:**

- Located within the dedicated Identity subnet
- Static private IP addressing
- Private IP: `10.1.0.4`
- No direct public IP assigned
- Identity subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway

**Administrative Access:**

- Administrative access is performed through Azure Bastion
- Direct RDP exposure to the public Internet is not permitted
- Routine administrative workloads are being separated from the Domain Controller through dedicated management infrastructure

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

- Located within the dedicated Corporate Management subnet
- Dynamic private IP addressing
- Current private IP: `10.1.1.4`
- No direct public IP assigned
- Management subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway

**Administrative Access:**

- Administrative access is performed through Azure Bastion
- Domain authentication is used for privileged administrative sessions
- Direct RDP exposure to the public Internet is not permitted

---

## Server Communication

Current server communication follows the domain architecture below:

```text
DC01
├── Active Directory Domain Services
├── Active Directory-integrated DNS
└── Static Private Address
          |
          | Domain Services / DNS
          v
MGMT01
├── Domain Joined
├── Servers OU
└── Dedicated Management Platform
```

Both servers remain privately addressed within the Corporate Virtual Network.

Outbound Internet connectivity is provided through Azure NAT Gateway rather than relying on Azure default outbound access or direct public IP assignments.

---

## Design Considerations

Server roles are separated across dedicated network segments to support network segmentation and reduce unnecessary exposure.

The Domain Controller uses static private addressing because other systems rely on it for Active Directory-integrated DNS and domain services.

The Management Server uses dynamic private addressing because other infrastructure does not currently depend on it being reachable at a fixed IP address.

Domain services and management functions are hosted on separate virtual machines rather than combining routine administrative workloads with the Domain Controller.

MGMT01 is domain joined and provides a dedicated administrative platform for future remote management tooling and centralized infrastructure administration.

The Identity and Corporate Management subnets are configured as private subnets with explicit outbound connectivity provided through Azure NAT Gateway.

Azure Bastion provides administrative access without assigning public IP addresses directly to the server virtual machines.
