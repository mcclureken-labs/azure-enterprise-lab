# Active Directory Design

**Version:** 1.0  
**Last Updated:** August 6, 2026  
**Author:** Kendrick McClure

---

## Overview

This document outlines the Active Directory architecture, configuration, and administrative design for the Enterprise Azure Lab.

Active Directory Domain Services (AD DS) provides centralized identity, authentication, and DNS services for the Windows infrastructure within the Corporate network.

---

## Environment Information

| Setting | Value |
| --- | --- |
| Forest Name | corp.mccluretech.com |
| Domain Name | corp.mccluretech.com |
| NetBIOS Name | CORP |
| Forest Functional Level | Windows Server 2025 |
| Domain Functional Level | Windows Server 2025 |
| Domain Controller | DC01 |
| Azure Virtual Machine | vm-dc01-corp-prd-eus2 |
| Private IP | 10.1.0.4 |
| IP Allocation | Static |
| Operating System | Windows Server 2025 Datacenter: Azure Edition |

---

## Domain Controller

| Server | Role | Status |
| --- | --- | --- |
| DC01 | Active Directory Domain Services / DNS | Active |

DC01 is located within the dedicated Identity subnet of the Corporate Virtual Network.

The server uses static private addressing to provide a consistent network endpoint for directory and DNS services.

DC01 does not have a direct public IP address and is administratively accessed through Azure Bastion.

---

## DNS Configuration

DNS was installed alongside Active Directory Domain Services and provides name resolution for the Active Directory environment.

Current configuration includes:

- Active Directory-integrated DNS
- DNS hosted on DC01
- Internal DNS namespace: `corp.mccluretech.com`
- Static private addressing for the DNS server
- DNS delegation not configured for the standalone lab environment

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

---

## Management Architecture

A dedicated management server has been deployed:

```text
MGMT01
```

Azure resource:

```text
vm-mgmt01-corp-prd-eus2
```

MGMT01 is located within the dedicated Corporate Management subnet and will provide a separate administrative platform for managing the Windows environment.

Planned configuration includes:

- Joining MGMT01 to the Active Directory domain
- Installing Windows remote administration tools
- Performing routine Active Directory administration from MGMT01
- Separating routine administrative activity from the Domain Controller

---

## Administrative Strategy

The environment is being designed around separation between standard user activity, administrative activity, and domain infrastructure.

Administrative design principles include:

- Routine administration performed from dedicated management infrastructure
- Named administrative accounts used for privileged administrative tasks
- Built-in administrative accounts reserved for recovery or emergency scenarios where appropriate
- Administrative workloads separated from the Domain Controller
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
├── Identity Subnet
│   └── DC01
│       ├── Active Directory Domain Services
│       └── DNS
│
└── Management Subnet
    └── MGMT01
        └── Management Server
```

Administrative connectivity to both servers is provided through Azure Bastion without assigning direct public IP addresses to the virtual machines.

---

## Future Expansion

Planned Active Directory improvements include:

- Join MGMT01 to the domain
- Install remote administration tools on MGMT01
- Create administrative and standard user accounts
- Create security groups
- Create service accounts
- Implement Group Policy Objects (GPOs)
- Add domain-joined member servers and workstations
- Evaluate an additional Domain Controller for redundancy
- Explore hybrid integration with Microsoft Entra ID

---

## Design Decisions

- Active Directory was deployed as a new forest.
- AD DS and DNS are hosted on DC01.
- DC01 uses a static private IP address.
- Identity and management infrastructure are separated into dedicated subnets.
- Domain Controller and management workloads are hosted on separate virtual machines.
- Server virtual machines do not have direct public IP addresses.
- Azure Bastion provides administrative connectivity.
- The default Domain Controllers OU is retained for Domain Controller computer objects.
- The OU structure was designed to support future administrative delegation and Group Policy implementation.
