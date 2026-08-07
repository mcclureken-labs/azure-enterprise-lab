# Enterprise Azure Lab - Security Design

## Overview

This document outlines the security architecture and design decisions implemented throughout the Enterprise Azure Lab.

The environment uses layered security controls, network segmentation, private administrative access, and explicitly defined outbound connectivity to reduce unnecessary exposure of infrastructure resources.

---

# Security Principles

The environment is designed around the following principles:

- Least privilege
- Defense in depth
- Network segmentation
- Private administrative access
- Explicit outbound connectivity
- Separation of infrastructure roles
- Reduced public exposure
- Consistent resource organization

---

# Network Security Groups (NSGs)

Network Security Groups are used to control traffic at the subnet level and provide network-layer access controls between infrastructure segments.

## Hub Management

| Property          | Value                        |
| ----------------- | ---------------------------- |
| Name              | nsg-hub-management-prd-eus2  |
| Resource Group    | rg-security-prd-eus2         |
| Associated Subnet | snet-hub-management-prd-eus2 |

### Purpose

Protects shared management infrastructure located within the Hub Virtual Network.

---

## Corporate Management

| Property          | Value                         |
| ----------------- | ----------------------------- |
| Name              | nsg-corp-management-prd-eus2  |
| Resource Group    | rg-security-prd-eus2          |
| Associated Subnet | snet-corp-management-prd-eus2 |

### Purpose

Protects administrative systems deployed within the Corporate Virtual Network, including MGMT01.

---

# Administrative Access

Administrative access to server virtual machines is centralized through Azure Bastion.

Neither DC01 nor MGMT01 is assigned a direct public IP address.

## Azure Bastion

| Property           | Value                    |
| ------------------ | ------------------------ |
| Name               | bas-hub-prd-eus2         |
| Resource Group     | rg-connectivity-prd-eus2 |
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
- Domain membership for centralized authentication
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

# Private Subnets

The Identity and Corporate Management subnets are configured as private subnets with default outbound access disabled.

Current private workload subnets include:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

Workloads within these subnets are not assigned direct public IP addresses and do not rely on Azure default outbound access for Internet connectivity.

---

# Outbound Connectivity

Explicit outbound Internet connectivity for the Identity and Corporate Management subnets is provided through Azure NAT Gateway.

| Property       | Value                    |
| -------------- | ------------------------ |
| Name           | nat-corp-prd-eus2        |
| Resource Group | rg-connectivity-prd-eus2 |
| SKU            | StandardV2               |
| Public IP      | pip-nat-corp-prd-eus2    |

The assigned public IP address is intentionally excluded from the public repository.

The NAT Gateway provides a defined outbound path for workloads that require Internet connectivity while allowing server virtual machines to remain privately addressed.

## Outbound Traffic Flow

```text
Private Workload
      |
      v
Private Subnet
      |
      v
Azure NAT Gateway
      |
      v
Dedicated Public Egress IP
      |
      v
Internet
```

NAT Gateway provides outbound address translation and does not expose server virtual machines for unsolicited inbound Internet connectivity.

NAT Gateway is not used as a replacement for network filtering or traffic inspection. Additional centralized traffic inspection may be introduced later through Azure Firewall.

---

# Current Security Decisions

- Server virtual machines do not receive direct public IP addresses.
- Administrative access is centralized through Azure Bastion.
- RDP is not exposed directly to the public Internet.
- Network Security Groups are applied at the subnet level where appropriate.
- Identity and management workloads are separated into dedicated subnets.
- Identity and Corporate Management subnets use private subnet configuration.
- Default outbound access is disabled for current server workload subnets.
- Explicit outbound Internet connectivity is provided through Azure NAT Gateway.
- Domain Controller and management functions are hosted on separate virtual machines.
- Hub and Corporate resources are separated using a Hub-and-Spoke architecture.
- Infrastructure resources follow a consistent naming and organizational strategy.

---

# Future Security Enhancements

Planned security improvements include:

- Azure Firewall
- Microsoft Defender for Cloud
- Azure Policy
- Azure Key Vault
- Just-In-Time VM Access
- Network monitoring and diagnostics
- Centralized logging and monitoring
- Additional identity security controls
- More granular administrative privilege delegation

---

# Security Documentation Approach

This repository documents the architecture and security principles of the lab while intentionally excluding sensitive operational information such as credentials, secrets, authentication tokens, privileged account details, externally reachable IP addresses, and unique Azure subscription or tenant identifiers.

---

# Lessons Learned

## Private Subnets and Explicit Outbound Connectivity

The Identity and Corporate Management subnets were initially configured without default outbound Internet access.

During early deployment, Internet connectivity was temporarily enabled to support operating system updates, role installation, and configuration tasks.

Rather than retaining implicit default outbound connectivity, an Azure NAT Gateway was later deployed to provide an explicitly defined outbound path.

After the NAT Gateway was associated with the required workload subnets, private subnet configuration was restored and default outbound access was disabled.

Outbound HTTPS connectivity was then validated from a private workload, and the observed public egress address was confirmed to match the NAT Gateway public IP resource.

This demonstrated that private Azure workloads can maintain required outbound Internet connectivity without direct public IP assignments or reliance on Azure default outbound access.

## Security Role of NAT Gateway

Implementing NAT Gateway improved the outbound network architecture by providing a predictable and explicitly configured egress path.

However, NAT Gateway does not perform application-layer traffic inspection or replace a network firewall.

Future iterations of the environment may introduce Azure Firewall to provide centralized traffic filtering, inspection, and policy enforcement.
