# Active Directory Design

**Version:** 1.0  
**Last Updated:** August 6, 2026  
**Author:** Kendrick McClure

---

# Overview

This document outlines the Active Directory design, configuration, and administrative standards for the Enterprise Azure Lab environment.

---

# Environment Information

| Setting | Value |
|---------|-------|
| Forest Name | corp.mccluretech.com |
| Domain Name | corp.mccluretech.com |
| NetBIOS Name | CORP |
| Forest Functional Level | Windows Server 2025 |
| Domain Functional Level | Windows Server 2025 |
| Primary Domain Controller | DC01 |
| Azure Virtual Machine | vm-dc01-corp-prd-eus2 |
| Operating System | Windows Server 2025 Datacenter: Azure Edition |

---

# Infrastructure

## Domain Controllers

| Server | Role | Status |
|---------|------|--------|
| DC01 | Primary Domain Controller & DNS Server | Active |

---

# DNS Configuration

- Active Directory Integrated DNS
- DNS Server installed on DC01
- Internal DNS namespace: `corp.mccluretech.com`
- DNS delegation not configured (not required for this standalone lab)

---

# Organizational Unit Structure

```text
corp.mccluretech.com
│
├── Admins
├── Groups
├── Servers
├── User Accounts
├── Workstations
├── Service Accounts
├── Disabled Objects
│
├── Builtin
├── Computers
├── Domain Controllers
├── ForeignSecurityPrincipals
├── Managed Service Accounts
└── Users
```

---

# Administrative Strategy

- Daily administration will be performed from MGMT01.
- The built-in **Administrator** account is reserved for emergency access.
- Named administrative accounts will be used for routine administration.

---

# Naming Standards

## Domain Controllers

```text
DC##
```

Example:

```text
DC01
```

## Azure Virtual Machines

```text
vm-<role><number>-<scope>-<environment>-<region>
```

Examples:

```text
vm-dc01-corp-prd-eus2
vm-mgmt01-corp-prd-eus2
```

---

# Future Expansion

Planned additions include:

- Management Server (MGMT01)
- Group Policy Objects (GPOs)
- Administrative user accounts
- Security groups
- Service accounts
- File Server
- Application Server
- Hybrid Microsoft Entra ID integration
- Additional Domain Controller for redundancy

---

# Design Notes

- Active Directory deployed as a new forest.
- Windows Server 2025 functional level selected.
- Azure Static Private IP configured through the Azure NIC.
- Azure Bastion used for secure administrative access.
- Domain Controllers remain in the default **Domain Controllers** Organizational Unit.
