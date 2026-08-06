# Enterprise Azure Lab - Naming Convention

## Purpose

This document defines the naming standards used throughout the Enterprise Azure Lab to ensure consistency, readability, and maintainability.

---

# Naming Format

All Azure resources follow the format:

<resource-type>-<purpose>-<scope>-<environment>-<region>

Example:

vm-mgmt01-prd-eus2

---

# Resource Prefixes

| Resource Type | Prefix |
|---------------|--------|
| Resource Group | rg |
| Virtual Network | vnet |
| Subnet | snet |
| Network Security Group | nsg |
| Virtual Machine | vm |
| Network Interface | nic |
| Azure Bastion | bas |
| Public IP | pip |
| Route Table | rt |
| Network Security Group | nsg |
| Storage Account | st |
| Key Vault | kv |
| Recovery Services Vault | rsv |
| Log Analytics Workspace | law |
| Azure Firewall | afw |

---

# Environment Codes

| Environment | Code |
|-------------|------|
| Production | prd |
| Development | dev |
| Test | tst |

---

# Region Codes

| Azure Region | Code |
|--------------|------|
| East US 2 | eus2 |

---

# Current Naming Examples

## Resource Groups

- rg-connectivity-prd-eus2
- rg-security-prd-eus2
- rg-corporate-prd-eus2

---

## Virtual Networks

- vnet-hub-prd-eus2
- vnet-corporate-prd-eus2

---

## Subnets

- AzureFirewallSubnet
- AzureBastionSubnet
- GatewaySubnet
- snet-hub-management-prd-eus2
- snet-identity-prd-eus2
- snet-corp-management-prd-eus2
- snet-private-endpoints-prd-eus2
- snet-internal-apps-prd-eus2

---

## Network Security Groups

- nsg-hub-management-prd-eus2
- nsg-corp-management-prd-eus2

---

## Virtual Machines

- vm-mgmt01-prd-eus2
- vm-dc01-prd-eus2 *(planned)*

---

## Azure Bastion

- bas-hub-prd-eus2

---

## Public IP Addresses

- pip-bastion-hub-prd-eus2

---

# Naming Principles

- Resource names should clearly communicate their purpose.
- Naming should remain consistent across all Azure services.
- Environment and region identifiers should always be included where practical.
- Shared infrastructure belongs to the Hub.
- Workload-specific resources belong to their respective spokes.
