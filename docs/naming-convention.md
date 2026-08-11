# Enterprise Azure Lab - Naming Convention

## Purpose

This document defines the naming standards used throughout the Enterprise Azure Lab to maintain consistency, readability, and scalability as the environment grows.

The naming convention is intended to make Azure resources understandable without requiring the resource to be opened individually in the Azure portal.

---

## General Naming Format

Azure resources generally follow a structured naming convention based on resource type, function, scope, environment, and region.

```text
<prefix>-<function>-<scope>-<environment>-<region>
```

Not every resource requires every component. Resource names are adapted where necessary based on the purpose of the resource, architectural context, and Azure naming requirements.

Example:

```text
vm-mgmt01-corp-prd-eus2
```

This identifies the resource as:

- `vm` - Virtual Machine
- `mgmt01` - Management server 01
- `corp` - Corporate scope
- `prd` - Production environment
- `eus2` - East US 2 region

---

## Resource Prefixes

| Resource Type           | Prefix |
| ----------------------- | ------ |
| Resource Group          | rg     |
| Virtual Network         | vnet   |
| Subnet                  | snet   |
| Network Security Group  | nsg    |
| Virtual Machine         | vm     |
| Network Interface       | nic    |
| Azure Bastion           | bas    |
| NAT Gateway             | nat    |
| Public IP               | pip    |
| Route Table             | rt     |
| Storage Account         | st     |
| Key Vault               | kv     |
| Recovery Services Vault | rsv    |
| Log Analytics Workspace | law    |
| Azure Firewall          | afw    |

Some prefixes listed above are reserved for services planned for future phases of the environment and may not yet correspond to deployed resources.

---

## Scope Codes

| Scope                       | Code |
| --------------------------- | ---- |
| Corporate                   | corp |
| Hub / Shared Infrastructure | hub  |

Scope identifiers are used where they improve clarity and help distinguish resources with similar functions.

A resource's scope identifies its architectural purpose or the workload it serves and does not necessarily identify the Resource Group in which the resource is stored.

For example, `nat-corp-prd-eus2` provides outbound connectivity for Corporate workloads but is maintained within the shared connectivity Resource Group.

---

## Environment Codes

| Environment | Code |
| ----------- | ---- |
| Production  | prd  |
| Development | dev  |
| Test        | tst  |

The current lab primarily uses the `prd` designation to represent the production-style enterprise environment being modeled.

Development and test identifiers are reserved for future environments if separate lifecycle stages are introduced.

---

## Region Codes

| Azure Region | Code |
| ------------ | ---- |
| East US 2    | eus2 |

Additional region codes will be introduced if the environment expands into multiple Azure regions.

---

## Current Naming Examples

### Resource Groups

Resource Groups use a functional naming pattern because they organize resources according to broad architectural responsibilities.

```text
rg-<function>-<environment>-<region>
```

Current examples:

- `rg-connectivity-prd-eus2`
- `rg-security-prd-eus2`
- `rg-corporate-prd-eus2`

---

### Virtual Networks

Virtual Networks use a scope-oriented naming pattern.

```text
vnet-<scope>-<environment>-<region>
```

Current examples:

- `vnet-hub-prd-eus2`
- `vnet-corporate-prd-eus2`

---

### Subnets

Custom subnets generally follow:

```text
snet-<function>-<scope>-<environment>-<region>
```

The exact components are adjusted when including both function and scope would create unnecessary repetition.

Current custom subnet names include:

- `snet-hub-management-prd-eus2`
- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-private-endpoints-prd-eus2`
- `snet-internal-apps-prd-eus2`

Azure-reserved subnet names retain Microsoft's required naming conventions where applicable:

- `AzureFirewallSubnet`
- `AzureBastionSubnet`
- `GatewaySubnet`

Azure-required naming takes precedence over the custom naming convention.

---

### Network Security Groups

Network Security Groups are named according to the network segment they protect.

Current examples:

- `nsg-hub-management-prd-eus2`
- `nsg-corp-management-prd-eus2`

This creates a direct naming relationship between the NSG and its associated subnet.

For example:

```text
nsg-corp-management-prd-eus2
        ↓
snet-corp-management-prd-eus2
```

---

### Virtual Machines

Azure Virtual Machines use:

```text
vm-<role><number>-<scope>-<environment>-<region>
```

Current examples:

- `vm-dc01-corp-prd-eus2`
- `vm-mgmt01-corp-prd-eus2`

The Azure resource name provides infrastructure context while the Windows hostname remains intentionally shorter.

---

### Azure Bastion

Current Azure Bastion resource:

```text
bas-hub-prd-eus2
```

The `hub` designation reflects that Bastion is deployed as shared administrative infrastructure within the Hub Virtual Network.

---

### NAT Gateway

Current NAT Gateway resource:

```text
nat-corp-prd-eus2
```

The `corp` scope identifies the workload network served by the NAT Gateway.

Although the NAT Gateway is stored within `rg-connectivity-prd-eus2`, its name reflects the Corporate workloads for which it provides outbound connectivity.

This distinction allows resource names to describe architectural purpose independently from Resource Group organization.

---

### Public IP Resources

Current Public IP resources include:

- `pip-bastion-hub-prd-eus2`
- `pip-nat-corp-prd-eus2`

The resource name identifies both the Azure resource type and the service with which the Public IP is associated.

For example:

```text
pip-bastion-hub-prd-eus2
```

identifies the Public IP resource associated with Azure Bastion in the Hub.

```text
pip-nat-corp-prd-eus2
```

identifies the Public IP resource associated with the Corporate NAT Gateway.

Assigned public IP addresses themselves are intentionally not documented in the public repository.

---

## Server Hostnames

Azure resource names and Windows hostnames serve different purposes.

| Azure VM Resource           | Windows Hostname |
| --------------------------- | ---------------- |
| `vm-dc01-corp-prd-eus2`     | `DC01`           |
| `vm-mgmt01-corp-prd-eus2`   | `MGMT01`         |

Short Windows hostnames are used internally for Active Directory and operating system administration, while Azure resource names contain additional information about role, scope, environment, and region.

This prevents infrastructure metadata from unnecessarily complicating Windows hostnames while maintaining descriptive Azure resource names.

---

## Naming Design Decisions

### Function Before Location

Resource names describe what a resource does or the workload it supports rather than simply repeating the Resource Group in which it is stored.

This allows resources to remain understandable even when viewed outside their Resource Group context.

### Consistency Over Absolute Uniformity

The naming convention is intentionally structured but not every Azure resource uses an identical number of naming components.

For example, Resource Groups are organized primarily by function, while Virtual Machines require additional role and scope information.

The goal is consistent meaning rather than forcing every Azure resource into an unnecessarily rigid format.

### Azure Requirements Take Precedence

Some Azure services require specific names or impose service-specific naming restrictions.

Microsoft-required resource names, such as `AzureBastionSubnet`, `AzureFirewallSubnet`, and `GatewaySubnet`, take precedence over the custom naming standard.

### Scalability

The naming convention is designed to support additional workloads, environments, regions, and Azure services without requiring the existing naming strategy to be redesigned.

---

## Naming Principles

- Resource names should clearly communicate their purpose.
- Naming should remain consistent across Azure services.
- Environment and region identifiers should be included where practical.
- Scope identifiers should be used when they improve resource identification.
- Resource names should reflect architectural purpose rather than only Resource Group placement.
- Shared connectivity and security resources should be organized according to their architectural function.
- Workload-specific resources should use scope identifiers that reflect the environments they support.
- Azure-required resource names take precedence over the custom naming standard.
- Consistency of meaning is prioritized over forcing every resource into an identical naming structure.
- Naming conventions should remain scalable as additional services, workloads, environments, and regions are introduced.
