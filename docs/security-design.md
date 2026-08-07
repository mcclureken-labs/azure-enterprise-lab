# Enterprise Azure Lab - Security Design

## Overview

This document outlines the security architecture and design decisions implemented throughout the Enterprise Azure Lab.

The environment uses layered security controls, network segmentation, and private administrative access to reduce unnecessary exposure of infrastructure resources.

---

# Security Principles

The environment is designed around the following principles:

- Least privilege
- Defense in depth
- Network segmentation
- Private administrative access
- Separation of infrastructure roles
- Reduced public exposure
- Consistent resource organization

---

# Network Security Groups (NSGs)

Network Security Groups are used to control traffic at the subnet level and provide network-layer access controls between infrastructure segments.

## Hub Management

| Property | Value |
| --- | --- |
| Name | nsg-hub-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-hub-management-prd-eus2 |

### Purpose

Protects shared management infrastructure located within the Hub Virtual Network.

---

## Corporate Management

| Property | Value |
| --- | --- |
| Name | nsg-corp-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-corp-management-prd-eus2 |

### Purpose

Protects administrative systems deployed within the Corporate Virtual Network, including MGMT01.

---

# Administrative Access

Administrative access to server virtual machines is centralized through Azure Bastion.

Neither DC01 nor MGMT01 is assigned a direct public IP address.

## Azure Bastion

| Property | Value |
| --- | --- |
| Name | bas-hub-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-bastion-hub-prd-eus2 |

The Azure resource name of the Bastion Public IP is documented for architectural reference. The assigned public IP address is intentionally not included in the public project documentation.

### Benefits

- No direct public IP addresses assigned to Windows servers
- Browser-based administrative connectivity
- RDP access without exposing RDP directly to the public Internet
- Private IP connectivity between Azure Bastion and managed virtual machines
- Centralized administrative access
- Reduced server attack surface

---

# Server Security

## DC01

DC01 provides Active Directory Domain Services and DNS for the environment.

Security considerations include:

- No direct public IP address
- Administrative access through Azure Bastion
- Dedicated Identity subnet
- Static private IP addressing for reliable directory and DNS services
- Domain Controller role separated from routine management workloads

## MGMT01

MGMT01 provides a dedicated platform for infrastructure administration.

Security considerations include:

- No direct public IP address
- Administrative access through Azure Bastion
- Dedicated Management subnet
- Routine administrative workloads separated from the Domain Controller

---

# Virtual Network Segmentation

## Hub Network

The Hub Virtual Network contains shared connectivity and management infrastructure.

Current and planned services include:

- Azure Bastion
- Shared management infrastructure
- Azure Firewall (future)
- VPN Gateway (future)

## Corporate Network

The Corporate Virtual Network contains workload and identity infrastructure.

Network segments include:

- Identity
- Corporate Management
- Private Endpoints
- Internal Applications

Separating infrastructure by function allows security controls to be applied according to workload requirements.

---

# Current Security Decisions

- Server virtual machines do not receive direct public IP addresses.
- Administrative access is centralized through Azure Bastion.
- RDP is not exposed directly to the public Internet.
- Network Security Groups are applied at the subnet level where appropriate.
- Identity and management workloads are separated into dedicated subnets.
- Domain Controller and management functions are hosted on separate virtual machines.
- Hub and Corporate resources are separated using a Hub-and-Spoke architecture.
- Infrastructure resources follow a consistent naming and organizational strategy.

---

# Outbound Connectivity

The initial network design restricted default outbound Internet connectivity for workload subnets.

During deployment, outbound Internet connectivity was temporarily enabled where required to support operating system updates, role installation, and configuration tasks.

This is a transitional lab configuration rather than the intended long-term outbound connectivity design.

Future iterations of the environment will implement controlled outbound connectivity using dedicated Azure networking services.

---

# Future Security Enhancements

Planned security improvements include:

- Azure Firewall
- Controlled outbound connectivity
- Microsoft Defender for Cloud
- Azure Policy
- Azure Key Vault
- Just-In-Time VM Access
- Network monitoring and diagnostics
- Centralized logging and monitoring
- Additional identity security controls

---

# Security Documentation Approach

This repository documents the architecture and security principles of the lab while intentionally excluding sensitive operational information such as credentials, secrets, authentication tokens, privileged account information, and externally reachable IP addresses.

---

# Lessons Learned

## Private Subnets and Outbound Connectivity

Restricting default outbound Internet connectivity reduces unnecessary external network access but requires deliberate planning for services that need outbound connectivity.

During the Active Directory deployment, outbound connectivity was temporarily enabled to support deployment and configuration requirements.

A production-oriented design would provide controlled outbound connectivity through an appropriate centralized egress solution rather than relying on unrestricted default outbound access.

This configuration will be revisited as the network security architecture continues to mature.
