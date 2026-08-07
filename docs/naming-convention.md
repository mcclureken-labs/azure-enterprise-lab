# Enterprise Azure Lab - Naming Convention

## Purpose

This document defines the naming standards used throughout the Enterprise Azure Lab to maintain consistency, readability, and scalability as the environment grows.

## General Naming Format

Azure resources follow a structured naming convention based on resource type, function, environment, and region.

```text
<prefix>-<function>-<scope>-<environment>-<region>
```

Not every resource requires every component. Resource names are adapted where necessary based on the purpose of the resource and Azure naming requirements.

Example:

```text
vm-mgmt01-corp-prd-eus2
```

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
| Public IP | pip |
| Route Table | rt |
| Storage Account | st |
| Key Vault | kv |
| Recovery Services Vault | rsv |
| Log Analytics Workspace | law |
| Azure Firewall | afw |

## Scope Codes

| Scope | Code |
| --- | --- |
| Corporate | corp |
| Hub / Shared Infrastructure | hub |

Scope identifiers are used where they improve clarity and help distinguish resources with similar functions.

## Environment Codes

| Environment | Code |
| --- | --- |
| Production | prd |
| Development | dev |
| Test | tst |

## Region Codes

| Azure Region | Code |
| --- | --- |
| East US 2 | eus2 |

## Current Naming Examples

### Resource Groups

- `rg-connectivity-prd-eus2`
- `rg-security-prd-eus2`
- `rg-corporate-prd-eus2`

### Virtual Networks

- `vnet-hub-prd-eus2`
- `vnet-corporate-prd-eus2`

### Subnets

Azure-reserved subnet names retain Microsoft's required naming where applicable.

- `AzureFirewallSubnet`
- `AzureBastionSubnet`
- `GatewaySubnet`
- `snet-hub-management-prd-eus2`
- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-private-endpoints-prd-eus2`
- `snet-internal-apps-prd-eus2`

### Network Security Groups

- `nsg-hub-management-prd-eus2`
- `nsg-corp-management-prd-eus2`

### Virtual Machines

- `vm-dc01-corp-prd-eus2`
- `vm-mgmt01-corp-prd-eus2`

### Azure Bastion

- `bas-hub-prd-eus2`

### Public IP Resources

- `pip-bastion-hub-prd-eus2`

The resource name identifies the Azure Public IP resource and does not document the assigned public IP address.

## Server Hostnames

Azure resource names and Windows hostnames serve different purposes.

| Azure VM Resource | Windows Hostname |
| --- | --- |
| `vm-dc01-corp-prd-eus2` | `DC01` |
| `vm-mgmt01-corp-prd-eus2` | `MGMT01` |

Short Windows hostnames are used internally while Azure resource names contain additional context about scope, environment, and region.

## Naming Principles

- Resource names should clearly communicate their purpose.
- Naming should remain consistent across Azure services.
- Environment and region identifiers should be included where practical.
- Scope identifiers should be used when they improve resource identification.
- Shared infrastructure belongs to the Hub.
- Workload-specific resources belong to their respective spokes.
- Azure-required resource names take precedence over the custom naming standard.
- Naming conventions should remain scalable as additional services and workloads are introduced.
