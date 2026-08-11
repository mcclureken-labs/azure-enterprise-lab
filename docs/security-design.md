# Enterprise Azure Lab - Security Design

## Overview

This document outlines the security architecture and design decisions implemented throughout the Enterprise Azure Lab.

The environment uses layered security controls, network segmentation, private administrative access, separation of infrastructure roles, and explicitly defined outbound connectivity to reduce unnecessary exposure of infrastructure resources.

The current architecture is shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

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

These principles guide the current implementation as well as future security enhancements planned for the environment.

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

Azure Bastion is deployed within the Hub Virtual Network and reaches privately addressed server resources in the Corporate Virtual Network across VNet peering.

### Security Benefits

- No direct public IP addresses assigned to Windows servers
- Browser-based administrative connectivity
- RDP access without exposing RDP directly to the public Internet
- Private IP connectivity between Azure Bastion and managed virtual machines
- Centralized administrative access
- Reduced direct exposure of server infrastructure

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
- Dedicated Corporate Management subnet
- Domain membership for centralized authentication
- Dedicated administrative identity
- Routine administrative workloads separated from the Domain Controller

Separating management activity from the Domain Controller reduces the need to perform routine administrative work directly on DC01.

---

# Virtual Network Segmentation

Network segmentation is used to separate infrastructure according to function.

## Hub Network

The Hub Virtual Network contains shared connectivity and management infrastructure.

Current and reserved infrastructure includes:

- Azure Bastion
- Shared management infrastructure
- Azure Firewall subnet (reserved)
- Gateway subnet (reserved)

Azure Firewall and gateway services have not yet been deployed.

## Corporate Network

The Corporate Virtual Network contains workload and identity infrastructure.

Network segments include:

- Identity
- Corporate Management
- Private Endpoints
- Internal Applications

The Identity and Corporate Management subnets currently host deployed infrastructure, while the Private Endpoint and Internal Application subnets are reserved for future workloads.

Separating infrastructure by function allows security controls and routing decisions to be applied according to workload requirements.

---

# Private Subnets

The Identity and Corporate Management subnets are configured as private subnets with default outbound access disabled.

Current private workload subnets include:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

Server virtual machines within these subnets are not assigned direct public IP addresses and do not rely on Azure default outbound access for Internet connectivity.

Required outbound connectivity is instead provided through an explicitly configured Azure NAT Gateway.

---

# Outbound Connectivity

Explicit outbound Internet connectivity for the Identity and Corporate Management subnets is provided through Azure NAT Gateway.

| Property           | Value                    |
| ------------------ | ------------------------ |
| Name               | nat-corp-prd-eus2        |
| Resource Group     | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2    |

The NAT Gateway is associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`

Internet-bound connections originating from these subnets use the NAT Gateway as their explicit outbound path.

Azure NAT Gateway performs Source Network Address Translation (SNAT), translating the private source address of outbound connections to the dedicated static Public IP resource associated with the NAT Gateway.

This provides a predictable outbound egress resource while allowing DC01 and MGMT01 to remain privately addressed.

The assigned public IP address itself is intentionally excluded from the public repository.

## NAT Gateway Security Boundary

Azure NAT Gateway provides outbound connectivity and address translation but is not treated as a firewall or traffic inspection service.

It does not provide application-layer traffic inspection or replace security controls such as Network Security Groups or Azure Firewall.

The NAT Gateway also does not provide unsolicited inbound Internet connectivity to DC01 or MGMT01.

Future iterations of the environment may introduce Azure Firewall to provide centralized network filtering, inspection, and policy enforcement.

---

# Identity and Administrative Separation

The environment separates domain infrastructure from routine administrative workloads.

DC01 hosts Active Directory Domain Services and DNS, while MGMT01 provides a dedicated domain-joined system for administrative activity.

Current design decisions include:

- Domain Controller and management workloads hosted on separate virtual machines
- Dedicated Identity and Management network segments
- Named administrative identity for privileged tasks
- MGMT01 joined to the Active Directory domain
- MGMT01 computer object organized within the custom Servers OU
- Administrative connectivity provided through Azure Bastion
- Privileged account details intentionally excluded from public documentation

Additional privilege separation and least-privilege controls will be introduced as the identity architecture develops.

---

# Current Security Controls

The following controls and architectural decisions are currently implemented:

- Server virtual machines do not receive direct public IP addresses.
- Administrative access is centralized through Azure Bastion.
- RDP is not exposed directly to the public Internet.
- Network Security Groups are applied at the subnet level where appropriate.
- Identity and management workloads are separated into dedicated subnets.
- Identity and Corporate Management subnets use private subnet configuration.
- Default outbound access is disabled for current server workload subnets.
- Explicit outbound Internet connectivity is provided through Azure NAT Gateway.
- NAT Gateway uses a dedicated static Public IP resource for outbound SNAT.
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

These services and controls are planned enhancements and should not be interpreted as currently deployed infrastructure.

---

# Security Documentation Approach

This repository documents the architecture and security principles of the lab while intentionally excluding sensitive operational information such as credentials, secrets, authentication tokens, privileged account details, externally reachable IP addresses, and unique Azure subscription or tenant identifiers.

Resource names, private addressing, network architecture, and other non-secret implementation details are documented where they provide useful architectural context.

---

# Lessons Learned

## Private Subnets and Explicit Outbound Connectivity

The Identity and Corporate Management subnets were initially configured without default outbound Internet access.

During early deployment, Internet connectivity was temporarily enabled to support operating system updates, role installation, and configuration tasks.

Rather than retaining implicit default outbound connectivity, an Azure NAT Gateway was later deployed to provide an explicitly defined outbound path.

After the NAT Gateway was associated with the required workload subnets, private subnet configuration was restored and default outbound access was disabled.

Outbound HTTPS connectivity was then validated from a private workload, and the observed public egress address was confirmed to match the NAT Gateway Public IP resource.

This demonstrated that private Azure workloads can maintain required outbound Internet connectivity without direct public IP assignments or reliance on Azure default outbound access.

## Security Role of NAT Gateway

Implementing NAT Gateway improved the outbound network architecture by providing a predictable and explicitly configured egress path.

The implementation also reinforced the distinction between address translation and network security enforcement.

NAT Gateway performs outbound SNAT but does not provide application-layer traffic inspection or replace a network firewall.

Future iterations of the environment may introduce Azure Firewall to provide centralized traffic filtering, inspection, and policy enforcement.
