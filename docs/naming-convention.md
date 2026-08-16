# Enterprise Azure Lab - Naming Convention

**Version:** 1.2  
**Last Updated:** August 15, 2026  
**Author:** Kendrick McClure

---

## Purpose

This document defines the naming standards used throughout the Azure Enterprise Lab to maintain consistency, readability, and scalability as the environment grows.

The naming convention is intended to make Azure resources understandable without requiring each resource to be opened individually in the Azure portal.

The convention applies primarily to Azure resources. Operating system hostnames, Active Directory objects, and child configuration objects use shorter naming patterns appropriate to their administrative context.

---

## General Naming Format

Azure resources generally follow a structured naming convention based on resource type, function, scope, environment, and region.

    <prefix>-<function>-<scope>-<environment>-<region>

Not every resource requires every component. Resource names are adapted where necessary based on the purpose of the resource, architectural context, Azure naming requirements, and whether the object is a standalone Azure resource or a configuration object contained within another service.

Example:

    vm-mgmt01-corp-prd-eus2

This identifies the resource as:

- `vm` - Virtual Machine
- `mgmt01` - Management server 01
- `corp` - Corporate scope
- `prd` - Production-style environment
- `eus2` - East US 2 region

The goal is consistent meaning rather than forcing every object into an identical naming structure.

---

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
| NAT Gateway | nat |
| Public IP | pip |
| Load Balancer | lb |
| Route Table | rt |
| Storage Account | st |
| Key Vault | kv |
| Recovery Services Vault | rsv |
| Log Analytics Workspace | law |
| Azure Firewall | afw |

Some prefixes listed above are reserved for services planned for future phases of the environment and may not yet correspond to deployed resources.

---

## Scope Codes

| Scope | Code |
| --- | --- |
| Corporate | corp |
| Hub / Shared Infrastructure | hub |

Scope identifiers are used where they improve clarity and help distinguish resources with similar functions.

A resource's scope identifies its architectural purpose or the workload it serves and does not necessarily identify the Resource Group in which the resource is stored.

For example:

    nat-corp-prd-eus2

provides outbound connectivity for Corporate workloads but is maintained within the shared connectivity Resource Group.

This allows resource names to communicate architectural purpose independently from Resource Group placement.

---

## Environment Codes

| Environment | Code |
| --- | --- |
| Production | prd |
| Development | dev |
| Test | tst |

The current lab primarily uses the `prd` designation to represent the production-style enterprise environment being modeled.

The lab is not a real production environment. The designation is used to practice enterprise naming and lifecycle conventions.

Development and test identifiers are reserved for future environments if separate lifecycle stages are introduced.

---

## Region Codes

| Azure Region | Code |
| --- | --- |
| East US 2 | eus2 |

Additional region codes can be introduced if the environment expands into multiple Azure regions.

---

# Azure Resource Naming

## Resource Groups

Resource Groups use a functional naming pattern because they organize resources according to broad architectural responsibilities.

    rg-<function>-<environment>-<region>

Current examples:

- `rg-connectivity-prd-eus2`
- `rg-security-prd-eus2`
- `rg-corporate-prd-eus2`

The Resource Group name identifies the broad administrative or architectural function of the resources contained within it.

---

## Virtual Networks

Virtual Networks use a scope-oriented naming pattern.

    vnet-<scope>-<environment>-<region>

Current examples:

- `vnet-hub-prd-eus2`
- `vnet-corporate-prd-eus2`

The VNet name identifies the architectural network boundary represented by the resource.

---

## Subnets

Custom subnets generally follow:

    snet-<function>-<scope>-<environment>-<region>

The exact components are adjusted when including both function and scope would create unnecessary repetition or reduce readability.

Current custom subnet names include:

- `snet-hub-management-prd-eus2`
- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-private-endpoints-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

Some earlier resources use shorter naming patterns where the architectural context is already clear, such as:

    snet-identity-prd-eus2

rather than:

    snet-corp-identity-prd-eus2

The naming standard prioritizes clear meaning and consistency without unnecessarily repeating information.

### Azure-Reserved Subnet Names

Azure-reserved subnet names retain Microsoft's required naming conventions where applicable:

- `AzureFirewallSubnet`
- `AzureBastionSubnet`
- `GatewaySubnet`

Azure-required naming takes precedence over the custom naming convention.

---

## Network Security Groups

Network Security Groups are named according to the network segment they protect.

General pattern:

    nsg-<protected-segment>-<environment>-<region>

Current examples:

- `nsg-hub-management-prd-eus2`
- `nsg-corp-management-prd-eus2`
- `nsg-corporate-internal-apps-prd-eus2`

This creates a direct naming relationship between an NSG and its associated subnet.

For example:

    nsg-corp-management-prd-eus2
            ↓
    snet-corp-management-prd-eus2

and:

    nsg-corporate-internal-apps-prd-eus2
            ↓
    snet-corporate-internal-apps-prd-eus2

This makes the intended network-security association recognizable from the resource names alone.

---

## Virtual Machines

Azure Virtual Machines use:

    vm-<role><number>-<scope>-<environment>-<region>

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

For example:

- `web01`
- `web02`
- `web03`

The Azure resource name provides infrastructure context while the operating system hostname remains intentionally shorter.

---

## Network Interfaces

Network Interface resources use the `nic` prefix and generally identify the virtual machine to which they belong.

General pattern:

    nic-<server>-<scope>-<environment>-<region>

The NIC naming convention allows network interfaces to be associated with their corresponding virtual machines when viewed independently in Azure.

NIC names should maintain a clear relationship with the VM they support.

---

## Azure Bastion

Azure Bastion uses:

    bas-<scope>-<environment>-<region>

Current resource:

    bas-hub-prd-eus2

The `hub` designation reflects that Bastion operates as shared administrative infrastructure within the Hub Virtual Network.

---

## NAT Gateway

NAT Gateway uses:

    nat-<served-scope>-<environment>-<region>

Current resource:

    nat-corp-prd-eus2

The `corp` scope identifies the workload network served by the NAT Gateway.

Although the NAT Gateway is stored within `rg-connectivity-prd-eus2`, its name reflects the Corporate workloads for which it provides outbound connectivity.

This distinction allows resource names to describe architectural purpose independently from Resource Group organization.

---

## Public IP Resources

Public IP resources use:

    pip-<associated-service>-<scope>-<environment>-<region>

Current Public IP resources include:

- `pip-bastion-hub-prd-eus2`
- `pip-nat-corp-prd-eus2`

For example:

    pip-bastion-hub-prd-eus2

identifies the Public IP resource associated with Azure Bastion in the Hub.

    pip-nat-corp-prd-eus2

identifies the Public IP resource associated with the Corporate NAT Gateway.

The resource names are documented for architectural reference.

Assigned public IP addresses themselves are intentionally excluded from the public repository.

---

## Internal Load Balancer

Azure Load Balancer resources use:

    lb-<function>-<scope>-<environment>-<region>

Current resource:

    lb-corporate-internal-apps-prd-eus2

The name identifies the resource as:

- `lb` - Azure Load Balancer
- `corporate-internal-apps` - Internal Corporate application function
- `prd` - Production-style environment
- `eus2` - East US 2 region

The Load Balancer name intentionally describes the application tier it serves rather than the individual backend servers.

This allows additional backend systems to be added without requiring the Load Balancer resource to be renamed.

---

# Load Balancer Configuration Object Naming

Objects contained within the Azure Load Balancer use shorter functional names because the parent Load Balancer already provides the broader scope, environment, and regional context.

These objects do not need to repeat the complete Azure resource naming convention.

## Frontend IP Configuration

Frontend IP configurations use a functional identifier describing their purpose.

The frontend configuration represents the private application endpoint presented by the internal Load Balancer.

The naming pattern should remain concise because the object exists within:

    lb-corporate-internal-apps-prd-eus2

The static frontend private IP currently used by the application tier is:

    10.1.3.10

The IP address itself is configuration data rather than part of the naming convention.

---

## Backend Pool

Backend pool names describe the collection of workloads receiving application traffic.

The backend pool associated with the internal application tier contains:

- WEB01
- WEB02

Backend pool names should describe the workload or application tier rather than an individual server because membership may change over time.

---

## Health Probe

Health probe names should describe the service or protocol being validated.

The current health probe validates:

- Protocol: TCP
- Port: 80

Health probe names should remain concise and service-oriented because the parent Load Balancer already provides the application and environment context.

---

## Load-Balancing Rules

Load-balancing rule names should describe the application protocol or service being distributed.

The current application rule distributes:

- Protocol: TCP
- Frontend port: 80
- Backend port: 80

Rule names should identify the service being delivered without unnecessarily repeating the full Load Balancer resource name.

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

Azure resource names contain additional information about role, scope, environment, and region.

This prevents cloud infrastructure metadata from unnecessarily complicating operating-system hostnames while maintaining descriptive Azure resource names.

---

# Active Directory Naming

Active Directory objects use naming patterns appropriate to identity and Windows administration rather than the Azure resource naming convention.

## Domain

Current Active Directory domain:

    corp.mccluretech.com

NetBIOS domain:

    CORP

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

Current examples include:

- `GG-IT-Users`
- `DL-ITShare-RW`

Prefixes identify Active Directory group scope:

| Prefix | Meaning |
| --- | --- |
| GG | Global Group |
| DL | Domain Local Group |

For example:

    GG-IT-Users

identifies a Global security group representing IT users.

    DL-ITShare-RW

identifies a Domain Local security group representing read/write access to the ITShare resource.

Permission identifiers may be appended where they improve clarity.

Current example:

- `RW` - Read / Write access

This convention supports the AGDLP-style authorization structure implemented within the lab.

---

## Group Policy Objects

Group Policy Objects use:

    GPO-<scope-or-purpose>-<function>

Current example:

    GPO-Servers-Baseline

This identifies the object as:

- `GPO` - Group Policy Object
- `Servers` - Target workload category
- `Baseline` - Baseline configuration purpose

Group Policy names should clearly identify both the target and the purpose of the policy.

This allows additional policies to be introduced without relying on generic names that provide little administrative context.

---

## File Resources

Windows file resources use concise descriptive names based on their business or administrative purpose.

Current example:

    ITShare

The corresponding Domain Local security group includes the resource name and access level:

    DL-ITShare-RW

This creates a recognizable relationship between the resource and the group used to authorize access.

---

# Naming Design Decisions

## Function Before Location

Resource names describe what a resource does or the workload it supports rather than simply repeating the Resource Group in which it is stored.

This allows resources to remain understandable even when viewed outside their Resource Group context.

---

## Consistency Over Absolute Uniformity

The naming convention is intentionally structured, but not every Azure resource uses an identical number of naming components.

For example:

- Resource Groups are organized primarily by function.
- Virtual Machines include role, scope, environment, and region.
- Network Security Groups describe the segment they protect.
- Load Balancer child objects use shorter functional names.
- Active Directory objects follow Windows administration conventions rather than Azure resource naming conventions.

The goal is consistent meaning rather than forcing every object into an unnecessarily rigid format.

---

## Parent Context Reduces Repetition

Configuration objects contained within another Azure resource do not need to repeat all information already communicated by the parent resource.

For example, a backend pool contained within:

    lb-corporate-internal-apps-prd-eus2

does not need to repeat:

    corporate-internal-apps-prd-eus2

within the backend pool name.

Shorter child-object names improve readability while the parent resource retains the complete infrastructure context.

---

## Azure Requirements Take Precedence

Some Azure services require specific names or impose service-specific naming restrictions.

Microsoft-required resource names, such as:

- `AzureBastionSubnet`
- `AzureFirewallSubnet`
- `GatewaySubnet`

take precedence over the custom naming standard.

---

## Operating System Names Remain Concise

Operating system hostnames remain shorter than Azure resource names.

For example:

    vm-web01-corp-prd-eus2

maps to:

    WEB01

This keeps Windows and Linux administration readable while Azure retains the additional cloud infrastructure context.

---

## Identity Objects Use Administrative Context

Active Directory groups, Organizational Units, Group Policy Objects, and file resources use conventions designed for Windows identity administration.

For example:

    GG-IT-Users
    DL-ITShare-RW
    GPO-Servers-Baseline

These names communicate administrative purpose more effectively than applying the Azure region and environment naming pattern to domain objects.

---

## Scalability

The naming convention is designed to support additional:

- Workloads
- Servers
- Network segments
- Environments
- Azure regions
- Security groups
- Group Policy Objects
- Load Balancer backends
- Azure services

without requiring the existing naming strategy to be redesigned.

Sequential server numbering and functional resource naming allow the environment to expand while preserving recognizable relationships between resources.

---

# Naming Principles

- Resource names should clearly communicate their purpose.
- Naming should remain consistent across Azure services.
- Environment and region identifiers should be included where practical.
- Scope identifiers should be used when they improve resource identification.
- Resource names should reflect architectural purpose rather than only Resource Group placement.
- Shared connectivity and security resources should be organized according to their architectural function.
- Workload-specific resources should use scope identifiers that reflect the environments they support.
- Network Security Group names should clearly relate to the network segment they protect.
- Virtual Machine names should identify workload role and sequence.
- Operating system hostnames should remain concise.
- Active Directory object names should reflect administrative purpose and group scope.
- Group Policy names should identify both target and function.
- Child configuration objects should avoid unnecessarily repeating information already provided by the parent resource.
- Azure-required resource names take precedence over the custom naming standard.
- Consistency of meaning is prioritized over forcing every resource into an identical naming structure.
- Naming conventions should remain scalable as additional services, workloads, environments, and regions are introduced.

---

# Current Naming Summary

| Resource / Object | Current Name |
| --- | --- |
| Connectivity Resource Group | `rg-connectivity-prd-eus2` |
| Security Resource Group | `rg-security-prd-eus2` |
| Corporate Resource Group | `rg-corporate-prd-eus2` |
| Hub VNet | `vnet-hub-prd-eus2` |
| Corporate VNet | `vnet-corporate-prd-eus2` |
| Hub Management Subnet | `snet-hub-management-prd-eus2` |
| Identity Subnet | `snet-identity-prd-eus2` |
| Corporate Management Subnet | `snet-corp-management-prd-eus2` |
| Private Endpoints Subnet | `snet-private-endpoints-prd-eus2` |
| Internal Apps Subnet | `snet-corporate-internal-apps-prd-eus2` |
| Hub Management NSG | `nsg-hub-management-prd-eus2` |
| Corporate Management NSG | `nsg-corp-management-prd-eus2` |
| Internal Apps NSG | `nsg-corporate-internal-apps-prd-eus2` |
| Domain Controller VM | `vm-dc01-corp-prd-eus2` |
| Management VM | `vm-mgmt01-corp-prd-eus2` |
| Web Server 01 VM | `vm-web01-corp-prd-eus2` |
| Web Server 02 VM | `vm-web02-corp-prd-eus2` |
| Azure Bastion | `bas-hub-prd-eus2` |
| Bastion Public IP | `pip-bastion-hub-prd-eus2` |
| NAT Gateway | `nat-corp-prd-eus2` |
| NAT Gateway Public IP | `pip-nat-corp-prd-eus2` |
| Internal Load Balancer | `lb-corporate-internal-apps-prd-eus2` |
| Active Directory Domain | `corp.mccluretech.com` |
| Global IT Users Group | `GG-IT-Users` |
| ITShare Domain Local Group | `DL-ITShare-RW` |
| Server Baseline GPO | `GPO-Servers-Baseline` |
| File Resource | `ITShare` |

---

# Future Naming Extensions

As additional services are deployed, the same naming principles will be extended to resources such as:

- Azure Firewall
- Route Tables
- Private Endpoints
- Key Vault
- Storage Accounts
- Log Analytics Workspaces
- Recovery Services Vaults
- Additional Load Balancers
- Additional application workloads
- Additional Group Policy Objects
- Additional Active Directory security groups
- Infrastructure as Code deployments

New naming patterns will be documented as those resources are implemented rather than documenting hypothetical resource names as if they already exist.

---

# Summary

The Azure Enterprise Lab uses a structured naming convention designed to communicate resource type, function, architectural scope, environment, and region while remaining readable and scalable.

Azure resources use descriptive infrastructure-oriented names, operating system hostnames remain concise, Active Directory objects use identity-administration conventions, and configuration objects contained within parent Azure resources avoid unnecessary repetition.

The naming strategy prioritizes clear meaning, recognizable resource relationships, Azure platform requirements, and the ability to expand the environment without redesigning the existing convention.
