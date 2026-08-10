# Azure Enterprise Lab

## Overview

This repository documents the design, deployment, administration, and continued development of a fictional enterprise environment built in Microsoft Azure.

The project is designed to develop hands-on Azure infrastructure and systems administration skills through the implementation of enterprise-style networking, security, identity, compute, and management services.

The environment is being built incrementally alongside preparation for the Microsoft Azure Administrator (AZ-104) certification, with an emphasis on understanding not only how Azure services are configured, but why specific architectural decisions are made.

---

## Project Objectives

- Design a scalable enterprise-style Azure environment
- Implement Hub-and-Spoke network architecture
- Apply network segmentation and security controls
- Provide private administrative access to infrastructure
- Implement explicit outbound connectivity for private workloads
- Deploy and administer Windows Server infrastructure
- Implement Active Directory Domain Services and DNS
- Develop identity and access management experience
- Practice real-world infrastructure troubleshooting
- Implement monitoring and security services
- Develop PowerShell and infrastructure automation skills
- Explore Infrastructure as Code
- Maintain professional technical documentation throughout the project

---

## Current Architecture

The environment currently uses a Hub-and-Spoke architecture consisting of a centralized Hub Virtual Network and a Corporate workload Virtual Network.

Azure Bastion provides private administrative connectivity to server infrastructure without assigning direct public IP addresses to the virtual machines.

The Corporate Identity and Management subnets use Azure NAT Gateway for explicit outbound Internet connectivity. NAT Gateway performs Source Network Address Translation (SNAT) using a dedicated static Public IP resource.

Active Directory Domain Services and Active Directory-integrated DNS are hosted on DC01. The Corporate Virtual Network is configured to use DC01 as its custom DNS server, allowing domain-joined systems such as MGMT01 to locate and communicate with Active Directory services.

![Azure Enterprise Lab Topology](images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`diagrams/azure-enterprise-lab-topology.drawio`](diagrams/azure-enterprise-lab-topology.drawio).

### Architecture Highlights

- Hub-and-Spoke network topology
- Bidirectional VNet peering
- Dedicated Identity and Management subnets
- Azure Bastion for private administrative access
- No direct public IP addresses assigned to server virtual machines
- Azure NAT Gateway for explicit outbound Internet connectivity
- Dedicated static Public IP resource for NAT Gateway egress
- Active Directory Domain Services and DNS hosted on DC01
- Corporate VNet configured to use Active Directory DNS
- Domain-joined MGMT01 management server
- Reserved network segments for future Azure Firewall, gateway, Private Endpoint, and internal application workloads

---

## Current Infrastructure

### Networking

- Hub Virtual Network
- Corporate Virtual Network
- Hub-and-Spoke topology
- Bidirectional VNet peering
- Dedicated infrastructure subnets
- Enterprise IP addressing plan
- Network Security Groups
- Segmented Identity and Management networks
- Private workload subnets
- Azure NAT Gateway
- Dedicated static Public IP resource for outbound egress
- Explicit outbound Internet connectivity

### Secure Administration

- Azure Bastion
- Private server administration
- No direct public IP addresses assigned to server virtual machines
- RDP not directly exposed to the public Internet
- Dedicated domain management infrastructure
- Named administrative identity
- Separation of Domain Controller and routine management workloads

### Identity and DNS

- Windows Server 2025 Domain Controller
- Active Directory Domain Services
- Active Directory-integrated DNS
- Active Directory forest and domain
- Corporate VNet configured to use Active Directory DNS
- Dedicated Identity subnet
- Organizational Unit structure
- Static private addressing for directory and DNS services
- Domain-joined management server
- Validated domain authentication

### Compute

**DC01**

- Active Directory Domain Services
- Active Directory-integrated DNS
- Windows Server 2025 Datacenter: Azure Edition
- Static private addressing
- Dedicated Identity subnet

**MGMT01**

- Domain-joined management server
- Member of the `corp.mccluretech.com` domain
- Computer object organized within the custom Servers OU
- Windows Server 2025 Datacenter: Azure Edition
- Located within a dedicated Management subnet
- Uses Active Directory DNS hosted on DC01
- Dedicated platform for infrastructure administration

---

## Current Progress

### Phase 1 - Enterprise Networking

- [x] Resource organization
- [x] Enterprise IP addressing plan
- [x] Hub Virtual Network
- [x] Corporate Virtual Network
- [x] Infrastructure subnets
- [x] Hub-and-Spoke topology
- [x] VNet peering
- [x] Network Security Groups
- [x] Private workload subnet configuration
- [x] Azure NAT Gateway
- [x] Dedicated static Public IP for NAT Gateway
- [x] Explicit outbound connectivity
- [x] Outbound connectivity validation

### Phase 2 - Secure Administration

- [x] Azure Bastion deployment
- [x] Private server connectivity
- [x] Management subnet
- [x] Management server deployment
- [x] No direct public IP assignments to server virtual machines
- [x] Named administrative identity
- [x] Domain authentication from management infrastructure

### Phase 3 - Identity Infrastructure

- [x] Windows Server Domain Controller deployment
- [x] Active Directory Domain Services
- [x] Active Directory-integrated DNS
- [x] Active Directory forest and domain
- [x] Organizational Unit structure
- [x] Corporate VNet DNS configuration
- [x] Domain-join MGMT01
- [x] Move MGMT01 computer object to Servers OU
- [x] Validate domain authentication
- [ ] Configure remote administration tools
- [ ] Create standard user accounts
- [ ] Create security groups
- [ ] Create service accounts
- [ ] Implement Group Policy

### Future Phases

- [ ] Additional Windows Server workloads
- [ ] Azure Firewall
- [ ] Microsoft Defender for Cloud
- [ ] Azure Policy
- [ ] Azure Key Vault
- [ ] Azure Monitor and centralized logging
- [ ] Microsoft Entra ID integration
- [ ] Additional identity security controls
- [ ] PowerShell automation
- [ ] Bicep
- [ ] Terraform

---

## Documentation

Detailed technical documentation is maintained in the `docs` directory.

Current documentation includes:

- [Network Design](docs/network-design.md)
- [Security Design](docs/security-design.md)
- [Active Directory Design](docs/active-directory.md)
- [Server Inventory](docs/server-inventory.md)
- [Naming Conventions](docs/naming-convention.md)
- [Troubleshooting](docs/troubleshooting.md)

Documentation is updated as the environment evolves and includes architecture decisions, implementation details, troubleshooting scenarios, and lessons learned.

---

## Repository Structure

```text
azure-enterprise-lab/
│
├── README.md
│
├── docs/
│   ├── active-directory.md
│   ├── naming-convention.md
│   ├── network-design.md
│   ├── security-design.md
│   ├── server-inventory.md
│   └── troubleshooting.md
│
├── diagrams/
│   ├── README.md
│   └── azure-enterprise-lab-topology.drawio
│
├── images/
│   ├── README.md
│   └── azure-enterprise-lab-topology.png
│
├── powershell/
├── bicep/
└── terraform/ (future)
