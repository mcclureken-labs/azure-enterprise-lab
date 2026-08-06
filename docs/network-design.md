# Enterprise Network Design

## Overview

This document describes the network architecture for the Azure Enterprise Lab.

The objective of this project is to simulate a production-style Azure environment using enterprise design principles while developing hands-on Microsoft Azure administration skills.

---

# Design Goals

- Build a scalable Azure network
- Follow consistent naming conventions
- Separate shared infrastructure from workloads
- Design with future growth in mind
- Simulate an enterprise cloud environment

---

# Current Network

## Resource Group

| Name | Purpose |
|------|---------|
| `rg-connectivity-prd-eus2` | Shared networking infrastructure |

---

## Virtual Network

| Property | Value |
|---------|-------|
| Name | `vnet-hub-prd-eus2` |
| Address Space | `10.0.0.0/20` |

### Why a /20?

The hub virtual network serves as the central networking environment for the organization. A `/20` provides 4,096 IP addresses, leaving sufficient room for future shared networking services and expansion.

---

# Current Subnets

| Subnet | CIDR | Purpose |
|---------|------|---------|
| `AzureFirewallSubnet` | `10.0.0.0/26` | Reserved for Azure Firewall |
| `GatewaySubnet` | `10.0.0.64/27` | Reserved for VPN or ExpressRoute Gateway |
| `AzureBastionSubnet` | `10.0.0.128/26` | Reserved for Azure Bastion |
| `snet-management-prd-eus2` | `10.0.2.0/24` | Management resources |

---

# Design Decisions

## Separate Resource Group

Networking resources are stored in a dedicated Resource Group because they represent shared infrastructure that will support multiple workloads throughout the environment.

## Reserved Infrastructure Subnets

Dedicated subnets have been reserved for Azure Firewall, Virtual Network Gateway, and Azure Bastion. Although these services have not yet been deployed, reserving their address space now avoids future network redesign.

## Address Planning

The network intentionally leaves unused address space within the VNet. This allows future subnets to be added without renumbering existing resources.

---

# Future Expansion

The following components are planned for future implementation:

- Network Security Groups (NSGs)
- Route Tables
- Hub-and-Spoke Network Topology
- Azure Firewall
- Azure Bastion
- VPN Gateway
- Corporate and Application Spoke VNets

---

# Lessons Learned

- A Virtual Network defines the overall private address space.
- Subnets divide the VNet into logical network segments.
- Special Azure services require dedicated subnet names.
- Good IP planning considers future growth, not just current requirements.
