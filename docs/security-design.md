# Enterprise Azure Lab - Security Design

## Overview

This document outlines the security architecture and design decisions implemented throughout the Enterprise Azure Lab.

---

# Security Principles

The environment is designed around the following principles:

- Least privilege
- Defense in depth
- Network segmentation
- Private administrative access
- Enterprise resource organization

---

# Network Security Groups (NSGs)

## Hub Management

| Property | Value |
|----------|-------|
| Name | nsg-hub-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-hub-management-prd-eus2 |

### Purpose

Protects shared management infrastructure located within the Hub Virtual Network.

---

## Corporate Management

| Property | Value |
|----------|-------|
| Name | nsg-corp-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-corp-management-prd-eus2 |

### Purpose

Protects management servers deployed within the Corporate Virtual Network.

---

# Administrative Access

Administrative access is provided exclusively through Azure Bastion.

## Azure Bastion

| Property | Value |
|----------|-------|
| Name | bas-hub-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP | pip-bastion-hub-prd-eus2 |

### Benefits

- No Public IP addresses assigned to Windows servers
- Browser-based RDP over HTTPS
- Administrative traffic remains on Microsoft's private backbone
- Reduced attack surface

---

# Virtual Network Segmentation

## Hub Network

Shared infrastructure services.

Examples:

- Azure Bastion
- Azure Firewall (future)
- VPN Gateway (future)
- Shared Management

---

## Corporate Network

Business workloads.

Examples:

- Active Directory
- Management Servers
- Internal Applications
- Private Endpoints

---

# Current Security Decisions

- Windows servers do not receive Public IP addresses.
- Administrative access is centralized through Azure Bastion.
- Network Security Groups are applied at the subnet level.
- Hub and Corporate resources are separated using a Hub-and-Spoke architecture.
- Management workloads are isolated from shared networking infrastructure.

---

# Future Security Enhancements

The following services will be added later in the project:

- Azure Firewall
- NAT Gateway
- Microsoft Defender for Cloud
- Azure Policy
- Azure Key Vault
- Just-In-Time VM Access
- Network Watcher

---

# Lessons Learned

## Private Subnets

Private subnets provide improved security by removing default outbound Internet access.

During this project, default outbound Internet access was temporarily enabled to simplify the Active Directory deployment.

In a production environment, outbound connectivity would instead be provided through an Azure NAT Gateway or Azure Firewall.

This temporary change will be revisited later as part of the networking architecture.
