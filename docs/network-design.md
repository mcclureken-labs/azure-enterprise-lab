# Enterprise Azure Lab - Network Design

## Overview

This environment is designed using a Hub-and-Spoke architecture to separate shared infrastructure from application workloads while maintaining centralized management and security.

---

# Hub Virtual Network

| Property | Value |
|----------|-------|
| Name | vnet-hub-prd-eus2 |
| Address Space | 10.0.0.0/20 |
| Resource Group | rg-connectivity-prd-eus2 |

## Hub Subnets

| Subnet | Address Space | Purpose |
|---------|---------------|---------|
| AzureFirewallSubnet | 10.0.0.0/26 | Azure Firewall |
| GatewaySubnet | 10.0.0.64/27 | VPN / ExpressRoute Gateway |
| AzureBastionSubnet | 10.0.0.128/26 | Azure Bastion |
| snet-hub-management-prd-eus2 | 10.0.2.0/24 | Shared management infrastructure |

---

# Corporate Virtual Network

| Property | Value |
|----------|-------|
| Name | vnet-corporate-prd-eus2 |
| Address Space | 10.1.0.0/20 |
| Resource Group | rg-corporate-prd-eus2 |

## Corporate Subnets

| Subnet | Address Space | Purpose |
|---------|---------------|---------|
| snet-identity-prd-eus2 | 10.1.0.0/24 | Active Directory Domain Controllers |
| snet-corp-management-prd-eus2 | 10.1.1.0/24 | Management Servers |
| snet-private-endpoints-prd-eus2 | 10.1.2.0/24 | Private Endpoints |
| snet-internal-apps-prd-eus2 | 10.1.3.0/24 | Internal Applications |

---

# VNet Peering

The Hub and Corporate virtual networks are connected using bidirectional VNet Peering.

- Hub → Corporate
- Corporate → Hub

This allows resources to communicate over Microsoft's private backbone without traversing the public Internet.

---

# Azure Bastion

Azure Bastion is deployed in the Hub Virtual Network.

### Purpose

- Secure browser-based RDP
- No Public IPs assigned to servers
- Centralized management access

---

# Design Goals

- Centralized networking
- Secure administrative access
- Enterprise naming convention
- Logical workload segmentation
- Private management architecture
