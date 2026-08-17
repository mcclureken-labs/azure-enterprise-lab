# Enterprise Azure Lab - Naming Convention

**Version:** 1.4  
**Last Updated:** August 17, 2026  
**Author:** Kendrick McClure

---

## Purpose

This document defines the naming standards used throughout the Azure Enterprise Lab to maintain consistency, readability, and scalability as the environment grows.

The convention applies primarily to Azure resources. Operating system hostnames, Active Directory objects, and child configuration objects use shorter naming patterns appropriate to their administrative context.

---

## General Naming Format

Azure resources generally follow a structured naming convention based on resource type, function, scope, environment, and region.

```text
<prefix>-<function>-<scope>-<environment>-<region>
```

Not every resource requires every component. Resource names are adapted where necessary based on purpose, architectural context, Azure naming requirements, and whether the object exists independently or within another Azure resource.

Example:

```text
vm-mgmt01-corp-prd-eus2
```

This identifies the resource as:

- `vm` - Virtual Machine
- `mgmt01` - Management server 01
- `corp` - Corporate scope
- `prd` - Production-style environment
- `eus2` - East US 2 region

The goal is consistent meaning rather than forcing every object into an identical naming structure.

---

# Naming Codes

## Resource Prefixes

| Resource Type | Prefix |
| --- | --- |
| Resource Group | rg |
| Virtual Network | vnet |
| Subnet | snet |
| Network Security Group | nsg |
| Virtual Machine | vm |
| Network Interface | nic |
| Azure Bastion | bas |
| Azure Firewall | fw |
| Firewall Policy | fwpol |
| Route Table | rt |
| Public IP | pip |
| Load Balancer | lb |

Additional prefixes will be introduced as new Azure services are implemented.

---

## Scope Codes

| Scope | Code |
| --- | --- |
| Corporate | corp |
| Hub / Shared Infrastructure | hub |

Scope identifiers are used where they improve clarity and help distinguish resources with similar functions.

A resource's scope identifies its architectural purpose or the workload it serves and does not necessarily identify the Resource Group in which the resource is stored.

For example:

```text
fw-hub-prd-eus2
```

The `hub` designation identifies Azure Firewall as shared network infrastructure deployed within the Hub Virtual Network.

---

## Environment Codes

| Environment | Code |
| --- | --- |
| Production | prd |
| Development | dev |
| Test | tst |

The current lab primarily uses the `prd` designation to represent the production-style enterprise environment being modeled.

The lab is not a production environment. Development and test identifiers are reserved for future environments if separate lifecycle stages are introduced.

---

## Region Codes

| Azure Region | Code |
| --- | --- |
| East US 2 | eus2 |

Additional region codes can be introduced if the environment expands into multiple Azure regions.

---

# Azure Resource Naming

## Resource Groups

Resource Groups use a functional naming pattern:

```text
rg-<function>-<environment>-<region>
```

Current examples:

- `rg-connectivity-prd-eus2`
- `rg-security-prd-eus2`
- `rg-corporate-prd-eus2`

The Resource Group name identifies the broad administrative or architectural function of the resources contained within it.

---

## Virtual Networks

Virtual Networks use a scope-oriented naming pattern:

```text
vnet-<scope>-<environment>-<region>
```

Current examples:

- `vnet-hub-prd-eus2`
- `vnet-corporate-prd-eus2`

---

## Subnets

Custom subnets generally follow:

```text
snet-<function>-<scope>-<environment>-<region>
```

Components are adjusted where including both function and scope would create unnecessary repetition or reduce readability.

Current custom subnet names include:

- `snet-hub-management-prd-eus2`
- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-private-endpoints-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

For example, `snet-identity-prd-eus2` uses a shorter pattern because its architectural purpose is already clear without adding an additional Corporate scope identifier.

The naming standard prioritizes clear meaning and consistency over requiring every subnet to contain an identical number of components.

### Azure-Reserved Subnet Names

Azure-reserved subnet names retain Microsoft's required naming conventions where applicable:

- `AzureFirewallSubnet`
- `AzureFirewallManagementSubnet`
- `AzureBastionSubnet`
- `GatewaySubnet`

Azure-required naming takes precedence over the custom naming convention.

---

## Network Security Groups

Network Security Groups are named according to the network segment they protect.

General pattern:

```text
nsg-<protected-segment>-<environment>-<region>
```

Current examples:

- `nsg-hub-management-prd-eus2`
- `nsg-corp-management-prd-eus2`
- `nsg-corporate-identity-prd-eus2`
- `nsg-corporate-internal-apps-prd-eus2`

The naming relationship between an NSG and its associated subnet makes the intended network-security association recognizable from the resource names.

For example:

```text
nsg-corp-management-prd-eus2
        ↓
snet-corp-management-prd-eus2
```

---

## Virtual Machines

Azure Virtual Machines use:

```text
vm-<role><number>-<scope>-<environment>-<region>
```

Current examples:

- `vm-dc01-corp-prd-eus2`
- `vm-mgmt01-corp-prd-eus2`
- `vm-web01-corp-prd-eus2`
- `vm-web02-corp-prd-eus2`

Role identifiers include:

| Role | Meaning |
| --- | --- |
| dc | Domain Controller |
| mgmt | Management Server |
| web | Web Server |

Sequential numbering allows additional systems with the same role to be introduced without changing the naming structure.

---

## Network Interfaces

Network Interface resources use the `nic` prefix and identify the virtual machine they support.

General pattern:

```text
nic-<server>-<scope>-<environment>-<region>
```

NIC names should maintain a clear relationship with their corresponding virtual machine.

---

## Azure Bastion

Azure Bastion uses:

```text
bas-<scope>-<environment>-<region>
```

Current resource:

```text
bas-hub-prd-eus2
```

The `hub` designation reflects Bastion's role as shared infrastructure within the Hub Virtual Network.

---

## Azure Firewall

Azure Firewall uses:

```text
fw-<scope>-<environment>-<region>
```

Current resource:

```text
fw-hub-prd-eus2
```

The `hub` designation reflects Azure Firewall's role as centralized network-security infrastructure within the Hub Virtual Network.

---

## Firewall Policy

Azure Firewall Policy resources use:

```text
fwpol-<scope>-<environment>-<region>
```

Current resource:

```text
fwpol-hub-prd-eus2
```

The Firewall Policy name maintains a clear relationship with the Azure Firewall resource it governs.

---

## Route Tables

Route Tables use:

```text
rt-<scope>-<function>-<environment>-<region>
```

Current resource:

```text
rt-corporate-egress-prd-eus2
```

The resource name identifies both the workload scope and routing purpose of the Route Table.

---

## Public IP Resources

Public IP resources use:

```text
pip-<associated-service>-<scope>-<environment>-<region>
```

Current Public IP resources include:

- `pip-bastion-hub-prd-eus2`
- `pip-fw-hub-prd-eus2`
- `pip-fw-mgmt-hub-prd-eus2`

The resource name identifies the service with which the Public IP resource is associated.

Assigned public IP addresses themselves are intentionally excluded from the public repository.

---

## Internal Load Balancer

Azure Load Balancer resources use:

```text
lb-<function>-<scope>-<environment>-<region>
```

Current resource:

```text
lb-corporate-internal-apps-prd-eus2
```

The name identifies the internal application tier served by the Load Balancer rather than an individual backend server, allowing backend membership to change without requiring the Load Balancer resource to be renamed.

---

## Load Balancer Child Objects

Configuration objects contained within an Azure Load Balancer use concise functional names because the parent resource already provides application, environment, and regional context.

This convention applies to:

- Frontend IP configurations
- Backend pools
- Health probes
- Load-balancing rules

Child-object names should describe their function without unnecessarily repeating information already provided by the parent Load Balancer resource.

---

# Server Hostnames

Azure VM resource names and operating system hostnames serve different purposes.

| Azure VM Resource | Hostname |
| --- | --- |
| `vm-dc01-corp-prd-eus2` | `DC01` |
| `vm-mgmt01-corp-prd-eus2` | `MGMT01` |
| `vm-web01-corp-prd-eus2` | `WEB01` |
| `vm-web02-corp-prd-eus2` | `WEB02` |

Short hostnames are used internally for operating-system administration, Active Directory, DNS, application identification, and troubleshooting.

Azure resource names contain additional information about role, scope, environment, and region without unnecessarily complicating operating-system hostnames.

---

# Active Directory Naming

Active Directory objects use naming patterns appropriate to identity and Windows administration rather than the Azure resource naming convention.

## Domain

Current Active Directory domain:

```text
corp.mccluretech.com
```

NetBIOS domain:

```text
CORP
```

---

## Organizational Units

Organizational Units use descriptive names based on the object type or administrative purpose they contain.

Current custom OUs include:

- `Admins`
- `Disabled Objects`
- `Groups`
- `Servers`
- `Service Accounts`
- `User Accounts`
- `Workstations`

Human-readable names are used because these objects are managed within the context of the Active Directory domain rather than as standalone Azure resources.

---

## Active Directory Security Groups

Security-group names communicate both group scope and purpose.

Current examples:

- `GG-IT-Users`
- `DL-ITShare-RW`

Group-scope prefixes include:

| Prefix | Meaning |
| --- | --- |
| GG | Global Group |
| DL | Domain Local Group |

Permission identifiers may be appended where they improve clarity.

For example, `RW` identifies read/write access in `DL-ITShare-RW`.

This convention supports the AGDLP-style authorization structure implemented within the lab.

---

## Group Policy Objects

Group Policy Objects use:

```text
GPO-<scope-or-purpose>-<function>
```

Current example:

```text
GPO-Servers-Baseline
```

The name identifies both the target workload category and the purpose of the policy, allowing additional policies to be introduced without relying on generic names.

---

## File Resources

Windows file resources use concise descriptive names based on their administrative purpose.

Current example:

```text
ITShare
```

The corresponding Domain Local security group incorporates the resource name and access level:

```text
DL-ITShare-RW
```

This creates a recognizable relationship between the resource and the group used to authorize access.

---

# Naming Design Decisions

### Function Before Location

Resource names describe what a resource does or the workload it supports rather than simply repeating the Resource Group in which it is stored. This keeps resources understandable even when viewed outside their Resource Group context.

### Consistency Over Absolute Uniformity

Not every Azure resource requires the same number of naming components. Resource Groups emphasize function, Virtual Machines include workload role and sequence, NSGs identify protected network segments, and Active Directory objects follow Windows administration conventions.

The goal is consistent meaning rather than rigid structural uniformity.

### Parent Context Reduces Repetition

Configuration objects contained within another Azure resource do not repeat information already communicated by the parent resource.

This keeps child-object names concise while the parent resource retains the broader infrastructure context.

### Azure Requirements Take Precedence

Microsoft-required names and service-specific naming restrictions take precedence over the custom naming standard.

Examples include:

- `AzureBastionSubnet`
- `AzureFirewallSubnet`
- `AzureFirewallManagementSubnet`
- `GatewaySubnet`

### Operating System Names Remain Concise

Operating system hostnames remain shorter than Azure resource names.

For example:

```text
vm-web01-corp-prd-eus2
```

maps to:

```text
WEB01
```

This preserves descriptive Azure resource naming without unnecessarily complicating Windows and Linux administration.

### Identity Objects Use Administrative Context

Active Directory groups, Organizational Units, Group Policy Objects, and file resources use conventions designed for Windows identity administration rather than Azure infrastructure naming.

Examples include:

- `GG-IT-Users`
- `DL-ITShare-RW`
- `GPO-Servers-Baseline`

### Scalability

The naming convention is designed to support additional workloads, servers, network segments, environments, Azure regions, identity objects, and Azure services without requiring the existing naming strategy to be redesigned.

Sequential numbering and functional naming allow the environment to expand while preserving recognizable relationships between resources.

---

# Future Naming Extensions

The naming convention will be extended as additional services such as Private Endpoints, Key Vault, centralized monitoring, and Infrastructure as Code resources are implemented.

New naming patterns will be documented when those resources are deployed rather than documenting hypothetical resource names as current infrastructure.

---

# Summary

The Azure Enterprise Lab uses a structured naming convention designed to communicate resource type, function, architectural scope, environment, and region while remaining readable and scalable.

Azure resources use descriptive infrastructure-oriented names, operating system hostnames remain concise, Active Directory objects use identity-administration conventions, and configuration objects contained within parent Azure resources avoid unnecessary repetition.

The naming strategy prioritizes clear meaning, recognizable resource relationships, Azure platform requirements, and the ability to expand the environment without redesigning the existing convention.
