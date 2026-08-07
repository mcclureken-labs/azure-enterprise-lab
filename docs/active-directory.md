# Active Directory Design

**Version:** 1.1  
**Last Updated:** August 7, 2026  
**Author:** Kendrick McClure

---

## Overview

This document outlines the Active Directory architecture, configuration, and administrative design for the Enterprise Azure Lab.

Active Directory Domain Services (AD DS) provides centralized identity, authentication, and DNS services for the Windows infrastructure within the Corporate network.

---

## Environment Information

| Setting                 | Value                                         |
| ----------------------- | --------------------------------------------- |
| Forest Name             | corp.mccluretech.com                          |
| Domain Name             | corp.mccluretech.com                          |
| NetBIOS Name            | CORP                                          |
| Forest Functional Level | Windows Server 2025                           |
| Domain Functional Level | Windows Server 2025                           |
| Domain Controller       | DC01                                          |
| Azure Virtual Machine   | vm-dc01-corp-prd-eus2                         |
| Private IP              | 10.1.0.4                                      |
| IP Allocation           | Static                                        |
| Operating System        | Windows Server 2025 Datacenter: Azure Edition |

---

## Domain Controller

| Server | Role                                   | Status |
| ------ | -------------------------------------- | ------ |
| DC01   | Active Directory Domain Services / DNS | Active |

DC01 is located within the dedicated Identity subnet of the Corporate Virtual Network.

The server uses static private addressing to provide a consistent network endpoint for directory and DNS services.

DC01 does not have a direct public IP address and is administratively accessed through Azure Bastion.

---

## DNS Configuration

DNS was installed alongside Active Directory Domain Services and provides name resolution and service discovery for the Active Directory environment.

Current configuration includes:

- Active Directory-integrated DNS
- DNS hosted on DC01
- Internal DNS namespace: `corp.mccluretech.com`
- DC01 private IP: `10.1.0.4`
- Static private addressing for the DNS server
- Corporate VNet configured to use DC01 for DNS
- DNS delegation not configured for the standalone lab environment

Domain-joined workloads within the Corporate VNet use DC01 as their DNS server, allowing systems to resolve the internal Active Directory namespace and locate domain services.

External DNS queries can be resolved through the DNS server's recursive resolution and configured forwarding behavior.

---

## Organizational Unit Structure

The following Organizational Units were created to provide logical separation for administrative objects and future Group Policy implementation.

```text
corp.mccluretech.com
│
├── Admins
├── Groups
├── Servers
├── User Accounts
├── Workstations
├── Service Accounts
└── Disabled Objects
```

Default Active Directory containers and Organizational Units remain in place, including:

- Builtin
- Computers
- Domain Controllers
- ForeignSecurityPrincipals
- Managed Service Accounts
- Users

The default **Domain Controllers** Organizational Unit is retained for domain controller computer objects.

MGMT01 was moved into the custom **Servers** OU after being joined to the domain.

---

## Management Architecture

A dedicated domain-joined management server has been deployed:

```text
MGMT01
```

Azure resource:

```text
vm-mgmt01-corp-prd-eus2
```

MGMT01 is located within the dedicated Corporate Management subnet and provides a separate administrative platform for managing the Windows environment.

MGMT01 has been successfully joined to:

```text
corp.mccluretech.com
```

Following the domain join:

- Domain membership was validated successfully.
- Domain authentication was tested from MGMT01.
- The MGMT01 computer object was moved into the custom **Servers** OU.
- MGMT01 uses the Active Directory DNS service hosted on DC01.
- Administrative access continues to occur through Azure Bastion without a direct public IP assignment.

This design separates routine administrative activity from the Domain Controller and provides a dedicated system for future remote administration tooling.

---

## Domain Authentication

A named administrative account was created for privileged Active Directory administration.

Domain authentication was validated from MGMT01 using the administrative identity, confirming successful communication between the management server and Active Directory.

The authenticated security context was verified from MGMT01 using:

```powershell
whoami
```

The test confirmed that the active session was authenticated using the `CORP` domain rather than a local machine account.

Specific privileged account names and authentication information are intentionally excluded from public documentation.

---

## Administrative Strategy

The environment is designed around separation between standard user activity, administrative activity, and domain infrastructure.

Administrative design principles include:

- Routine administration performed from dedicated management infrastructure
- Named administrative accounts used for privileged administrative tasks
- Built-in administrative accounts reserved for recovery or emergency scenarios where appropriate
- Administrative workloads separated from the Domain Controller
- Domain-joined management infrastructure used for centralized administration
- Least-privilege access used as the environment matures

Specific privileged account names and authentication information are intentionally excluded from public documentation.

---

## Naming Standards

### Domain Controllers

```text
DC##
```

Example:

```text
DC01
```

### Management Servers

```text
MGMT##
```

Example:

```text
MGMT01
```

### Azure Virtual Machines

```text
vm-<role><number>-<scope>-<environment>-<region>
```

Examples:

```text
vm-dc01-corp-prd-eus2
vm-mgmt01-corp-prd-eus2
```

---

## Current Architecture

```text
Corporate VNet
│
│  DNS: DC01
│
├── Identity Subnet
│   │
│   └── DC01
│       ├── Active Directory Domain Services
│       ├── Active Directory-integrated DNS
│       └── Static Private IP
│
└── Management Subnet
    │
    └── MGMT01
        ├── Domain Joined
        ├── Servers OU
        └── Dedicated Management Server
```

Administrative connectivity to both servers is provided through Azure Bastion without assigning direct public IP addresses to the virtual machines.

The Identity and Management subnets use private subnet configuration with explicit outbound Internet connectivity provided through Azure NAT Gateway.

---

## Current Implementation Status

The following Active Directory components are currently implemented:

- Active Directory forest and domain
- Windows Server 2025 forest and domain functional levels
- Active Directory-integrated DNS
- Dedicated Identity subnet
- Static private addressing for DC01
- Corporate VNet DNS configured to use DC01
- Custom Organizational Unit structure
- Dedicated management server
- MGMT01 joined to the Active Directory domain
- MGMT01 computer object placed in the Servers OU
- Named administrative identity
- Validated domain authentication from MGMT01
- Private administrative connectivity through Azure Bastion

---

## Future Expansion

Planned Active Directory improvements include:

- Install remote administration tools on MGMT01
- Perform routine Active Directory administration remotely from MGMT01
- Create standard user accounts
- Create security groups
- Create service accounts
- Implement Group Policy Objects (GPOs)
- Add domain-joined member servers and workstations
- Evaluate an additional Domain Controller for redundancy
- Explore hybrid integration with Microsoft Entra ID
- Implement more granular administrative privilege delegation

---

## Design Decisions

- Active Directory was deployed as a new forest.
- AD DS and DNS are hosted on DC01.
- DC01 uses a static private IP address.
- The Corporate VNet uses DC01 for Active Directory DNS resolution.
- Identity and management infrastructure are separated into dedicated subnets.
- Domain Controller and management workloads are hosted on separate virtual machines.
- MGMT01 is joined to the Active Directory domain.
- The MGMT01 computer object is organized within the custom Servers OU.
- Named administrative identities are used instead of relying exclusively on built-in administrative accounts.
- Server virtual machines do not have direct public IP addresses.
- Azure Bastion provides administrative connectivity.
- Private workload subnets use explicit outbound connectivity through Azure NAT Gateway.
- The default Domain Controllers OU is retained for Domain Controller computer objects.
- The OU structure was designed to support future administrative delegation and Group Policy implementation.
