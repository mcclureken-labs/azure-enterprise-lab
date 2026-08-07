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

Azure Bastion provides administrative connectivity to privately addressed server infrastructure, while Azure NAT Gateway provides explicit outbound Internet connectivity for private workload subnets.

```text
                         Internet
                            ^
                            |
                       NAT Gateway
                            ^
                            |
                    Corporate VNet
                     /           \
                    /             \
           Identity Subnet    Management Subnet
                 |                  |
               DC01               MGMT01
            AD DS / DNS         Domain Joined
                 |                  |
                 +--------+---------+
                          |
                     VNet Peering
                          |
                       Hub VNet
                          |
                    Azure Bastion
                          ^
                          |
                    Administrator
```

Server virtual machines are not assigned direct public IP addresses.

Administrative connectivity is provided through Azure Bastion, while outbound Internet connectivity for the Identity and Management subnets is explicitly provided through Azure NAT Gateway.

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
- Dedicated public egress resource
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

- Network Design
- Security Design
- Active Directory Design
- Server Inventory
- Naming Conventions
- Troubleshooting

Documentation is updated as the environment evolves and includes architecture decisions, implementation details, troubleshooting scenarios, and lessons learned.

---

## Repository Structure

```text
azure-enterprise-lab/
│
├── docs/
│   ├── active-directory.md
│   ├── naming-convention.md
│   ├── network-design.md
│   ├── security-design.md
│   ├── server-inventory.md
│   └── troubleshooting.md
│
├── images/
├── powershell/
├── bicep/
└── terraform/ (future)
```

---

## Technologies

Technologies currently implemented or planned as part of the lab include:

- Microsoft Azure
- Azure Virtual Network
- VNet Peering
- Azure Bastion
- Azure NAT Gateway
- Network Security Groups
- Azure Resource Manager
- Windows Server 2025
- Active Directory Domain Services
- Active Directory-integrated DNS
- Microsoft Entra ID
- Azure Firewall
- Azure Monitor
- Microsoft Defender for Cloud
- Azure Policy
- Azure Key Vault
- PowerShell
- Azure CLI
- Bicep
- Terraform

---

## Security Approach

The environment is designed around defense in depth, network segmentation, private administrative access, explicit outbound connectivity, and separation of infrastructure roles.

Server virtual machines remain privately addressed without direct public IP assignments. Azure Bastion provides administrative connectivity, while private workload subnets use Azure NAT Gateway for explicitly defined outbound Internet connectivity rather than relying on Azure default outbound access.

NAT Gateway provides outbound address translation and predictable egress but is not treated as a replacement for network filtering or traffic inspection. Additional centralized security controls, including Azure Firewall, are planned as the environment matures.

Public project documentation focuses on architecture, implementation decisions, and technical learning while intentionally excluding credentials, secrets, authentication tokens, privileged account details, externally reachable IP addresses, and unique Azure subscription or tenant identifiers.

---

## Learning and Troubleshooting

This project is intentionally built through hands-on implementation rather than architecture documentation alone.

Issues encountered during deployment are investigated and documented to demonstrate the troubleshooting process, including:

- Connectivity testing
- DNS validation
- Effective route analysis
- Network Security Group validation
- Private subnet configuration
- Outbound connectivity troubleshooting
- NAT Gateway implementation and validation
- Windows Server configuration
- Active Directory and domain authentication
- Azure networking behavior

One documented scenario follows the progression from a private workload without Internet connectivity, through temporary use of default outbound access, to implementation and validation of an explicit Azure NAT Gateway egress architecture.

The `docs/troubleshooting.md` document will continue to grow as additional services and infrastructure are implemented.

---

## Purpose

This repository serves as both a technical portfolio and a record of my Azure learning journey.

The objective is to demonstrate practical experience designing, deploying, securing, administering, troubleshooting, and documenting an enterprise-style Microsoft Azure environment while continuously expanding the environment as new Azure administration concepts are learned.
