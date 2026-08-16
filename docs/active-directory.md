# Active Directory Design

**Version:** 1.4  
**Last Updated:** August 16, 2026  
**Author:** Kendrick McClure

---

## Overview

This document outlines the Active Directory architecture, configuration, and administrative design for the Azure Enterprise Lab.

Active Directory Domain Services (AD DS) provides centralized identity, authentication, DNS, Group Policy, and resource authorization services for the Windows infrastructure within the Corporate network.

The Active Directory environment is designed around a dedicated domain controller, a separate domain-joined management server, Active Directory-integrated DNS, structured Organizational Units, group-based access control, and centralized Group Policy administration.

The Active Directory infrastructure is integrated with the broader Azure network architecture shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [../diagrams/azure-enterprise-lab-topology.drawio](../diagrams/azure-enterprise-lab-topology.drawio).

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

DC01 provides the core directory and DNS services for the `corp.mccluretech.com` domain.

The server uses the static private IP address 10.1.0.4 to provide a consistent network endpoint for directory and DNS services.

DC01 does not have a direct public IP address. Administrative connectivity is performed through the private Azure network architecture using Azure Bastion when required.

Separating the Domain Controller from routine administrative workloads reduces the need to perform day-to-day management directly from DC01 and provides a foundation for applying more specific network and administrative controls as the environment develops.

DC01 also operates as a Global Catalog server for the domain.

---

## DNS Configuration

DNS was installed alongside Active Directory Domain Services and provides internal name resolution and Active Directory service discovery.

Current configuration includes:

- Active Directory-integrated DNS
- DNS hosted on DC01
- Internal DNS namespace: `corp.mccluretech.com`
- DC01 private IP: 10.1.0.4
- Static private addressing for the DNS server
- Corporate VNet configured to use 10.1.0.4 as its custom DNS server
- Active Directory-integrated forward lookup zones
- Automatic DNS registration for domain-joined systems
- Active Directory service discovery records
- DNS delegation not configured for the standalone lab environment

The Corporate VNet-level DNS configuration directs workloads using the VNet-provided DNS settings to DC01.

MGMT01 therefore uses the Active Directory DNS service hosted on DC01, allowing the domain-joined management server to resolve the internal namespace and locate Active Directory services required for authentication and administration.

The `corp.mccluretech.com` forward lookup zone contains registered host records for the domain infrastructure, including:

| Host | FQDN | Private IP |
| --- | --- | --- |
| DC01 | dc01.corp.mccluretech.com | 10.1.0.4 |
| MGMT01 | mgmt01.corp.mccluretech.com | 10.1.1.4 |

The DNS environment also contains the Active Directory-generated `_msdcs`, `_sites`, `_tcp`, and `_udp` structures used for domain controller and service discovery.

DNS and Active Directory service discovery were validated from the domain-joined management infrastructure.

---

## Organizational Unit Structure

Custom Organizational Units were created to provide logical separation between administrative identities, users, groups, servers, workstations, service accounts, and disabled objects.

```text
corp.mccluretech.com
│
├── Admins
├── Disabled Objects
├── Groups
├── Servers
├── Service Accounts
├── User Accounts
└── Workstations
```

The OU structure provides logical administrative boundaries and creates a foundation for targeted Group Policy application, delegated administration, and future identity-management requirements.

The `Servers` OU is used for domain-joined member servers. MGMT01 was moved from the default Computers container into this OU after joining the domain.

The Domain Controller remains within the built-in `Domain Controllers` OU.

---

## Domain-Joined Management Server

MGMT01 serves as the dedicated Windows administration server for the Corporate environment.

| Setting | Value |
| --- | --- |
| Server | MGMT01 |
| Azure Virtual Machine | vm-mgmt01-corp-prd-eus2 |
| Domain | corp.mccluretech.com |
| Private IP | 10.1.1.4 |
| IP Allocation | Dynamic |
| Organizational Unit | Servers |
| Operating System | Windows Server 2025 Datacenter: Azure Edition |

MGMT01 was successfully joined to the `corp.mccluretech.com` domain and uses the Active Directory-integrated DNS service hosted on DC01.

Domain authentication and internal DNS resolution were validated after the server was joined.

MGMT01 provides a dedicated administrative platform rather than requiring routine Active Directory administration to be performed directly from the Domain Controller.

Remote Server Administration Tools available from MGMT01 include:

- Active Directory Administrative Center
- Active Directory Domains and Trusts
- Active Directory Sites and Services
- Active Directory Users and Computers
- Active Directory PowerShell module
- DNS Manager
- Group Policy Management

This design separates routine administrative activity from the Domain Controller while providing centralized access to Windows infrastructure management tools.

---

## User Account Structure

Test user accounts were created within the dedicated `User Accounts` OU to provide identities for authentication, group membership, and access-control testing.

The lab uses fictional test identities rather than real production users.

Separating user objects into a dedicated OU provides a foundation for future user-specific Group Policy, administrative delegation, and identity lifecycle management.

---

## Security Group Design

Active Directory security groups are used to assign resource access through group membership rather than applying permissions directly to individual user accounts.

The initial access-control implementation uses an AGDLP-style group nesting model:

```text
Accounts
   │
   ▼
Global Group
GG-IT-Users
   │
   ▼
Domain Local Group
DL-ITShare-RW
   │
   ▼
Resource Permission
ITShare - Modify
```

### Global Group

`GG-IT-Users` is a Global security group representing users requiring access to IT resources.

Test user accounts requiring IT resource access are members of this group.

### Domain Local Group

`DL-ITShare-RW` is a Domain Local security group representing the permission required on the ITShare resource.

`GG-IT-Users` is nested within `DL-ITShare-RW`.

This separates user-role membership from resource authorization and allows permissions to be managed through group membership rather than directly modifying the resource ACL for individual users.

---

## File Share Authorization

A test file resource named `ITShare` was created on MGMT01 to validate Active Directory group-based authorization.

The final access model is:

```text
User Account
   │
   ▼
GG-IT-Users
   │
   ▼
DL-ITShare-RW
   │
   ▼
ITShare
NTFS Modify
```

`DL-ITShare-RW` is explicitly assigned NTFS `Modify` permissions to the resource.

The final NTFS ACL retains:

- `DL-ITShare-RW` - Modify
- `SYSTEM` - Full Control
- Local Administrators - Full Control

The resulting ACL provides standard resource access through the intended Active Directory security-group structure while preserving required SYSTEM and local administrative permissions.

Detailed permission troubleshooting and validation are maintained in the [Troubleshooting](troubleshooting.md) documentation.

---

## Group Policy Design

Group Policy Management is administered from MGMT01.

Rather than placing custom server configuration directly into Microsoft's default domain policies, a dedicated server baseline Group Policy Object was created:

`GPO-Servers-Baseline`

The GPO is linked to the `Servers` OU.

Current configuration includes:

| Configuration | Setting |
| --- | --- |
| Policy Scope | Servers OU |
| Link Enabled | Yes |
| Enforced | No |
| Security Filtering | Authenticated Users |
| Computer Configuration | Enabled |
| User Configuration | No settings configured |

The initial security baseline disables the local Guest account through:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── Security Options
                    └── Accounts: Guest account status
                        └── Disabled
```

Creating a dedicated server baseline allows additional Windows Server security controls to be introduced incrementally without modifying the Default Domain Policy.

This also provides a scalable structure for applying different configuration baselines to servers, workstations, users, and other Organizational Units as the environment expands.

---

## Validation

The Active Directory implementation was validated through hands-on administrative and functional testing.

Major validation performed within the environment includes:

- **Directory services:** Validated deployment of Active Directory Domain Services and creation of the `corp.mccluretech.com` forest and domain.
- **DNS and service discovery:** Validated Active Directory-integrated DNS, internal host registration, and directory service discovery.
- **Domain integration and administration:** Validated MGMT01 domain join, domain authentication, placement within the `Servers` OU, and centralized administration through management tools installed on MGMT01.
- **Directory organization:** Validated creation and organization of test user accounts, Global security groups, and Domain Local security groups.
- **Resource authorization:** Validated AGDLP-style nested group membership and group-based NTFS access to the ITShare resource.
- **Group Policy:** Validated creation and linkage of `GPO-Servers-Baseline` and implementation of the initial server security configuration.

Detailed troubleshooting, remediation, and validation scenarios are maintained in the [Troubleshooting](troubleshooting.md) documentation.

---

## Future Improvements

Planned Active Directory improvements include:

- Additional server baseline Group Policy settings
- Workstation-specific Group Policy
- User-specific Group Policy
- Additional role-based security groups
- Service account implementation
- Administrative account separation
- Additional password and account security controls
- Expanded PowerShell administration and automation
- Additional file-service permission models
- Additional Windows Server workloads
- Centralized auditing and logging
- Integration with additional Microsoft security services

---

## Related Documentation

Additional implementation details are maintained in:

- [Network Design](network-design.md)
- [Security Design](security-design.md)
- [Server Inventory](server-inventory.md)
- [Naming Conventions](naming-convention.md)
- [Troubleshooting](troubleshooting.md)

---

## Summary

The Active Directory environment provides centralized identity, authentication, DNS, Group Policy, and resource authorization services for the Corporate Azure network.

The current implementation includes a Windows Server 2025 Domain Controller, Active Directory-integrated DNS, a structured Organizational Unit hierarchy, a dedicated domain-joined management server, centralized administrative tooling, security-group-based authorization, an initial server security baseline GPO, and group-controlled NTFS resource access.

The design provides a practical foundation for continued development of enterprise Windows administration, identity management, access control, Group Policy, automation, and security capabilities within the Azure Enterprise Lab.
