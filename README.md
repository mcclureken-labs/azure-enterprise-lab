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

```text
                    Azure
                      |
              +----------------+
              |    Hub VNet    |
              +----------------+
                      |
              +----------------+
              | Azure Bastion  |
              +----------------+
                      |
                VNet Peering
                      |
              +----------------+
              | Corporate VNet |
              +----------------+
                 |          |
                 |          |
          Identity        Management
           Subnet           Subnet
              |               |
            DC01            MGMT01
              |
        +-----------+
        | AD DS/DNS |
        +-----------+
```

Server virtual machines are not assigned direct public IP addresses. Administrative connectivity is provided through Azure Bastion.

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

### Secure Administration

- Azure Bastion
- Private server administration
- No direct public IP addresses assigned to server virtual machines
- Dedicated management infrastructure

### Identity

- Windows Server 2025 Domain Controller
- Active Directory Domain Services
- Active Directory-integrated DNS
- Dedicated Identity subnet
- Organizational Unit structure
- Static private addressing for directory and DNS services

### Compute

**DC01**
- Active Directory Domain Services
- DNS
- Windows Server 2025 Datacenter: Azure Edition

**MGMT01**
- Dedicated management server
- Windows Server 2025 Datacenter: Azure Edition
- Located within a dedicated Management subnet

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

### Phase 2 - Secure Administration

- [x] Azure Bastion deployment
- [x] Private server connectivity
- [x] Management subnet
- [x] Management server deployment

### Phase 3 - Identity Infrastructure

- [x] Windows Server Domain Controller deployment
- [x] Active Directory Domain Services
- [x] DNS
- [x] Active Directory forest and domain
- [x] Organizational Unit structure
- [ ] Domain-join MGMT01
- [ ] Configure remote administration tools
- [ ] Create administrative and standard user accounts
- [ ] Create security groups
- [ ] Implement Group Policy

### Future Phases

- [ ] Additional Windows Server workloads
- [ ] Azure Firewall
- [ ] Controlled outbound connectivity
- [ ] Microsoft Defender for Cloud
- [ ] Azure Policy
- [ ] Azure Key Vault
- [ ] Azure Monitor and centralized logging
- [ ] Microsoft Entra ID integration
- [ ] PowerShell automation
- [ ] Bicep
- [ ] Terraform
- [ ] Additional identity security controls

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
- Azure Bastion
- Network Security Groups
- Azure Resource Manager
- Windows Server 2025
- Active Directory Domain Services
- DNS
- Microsoft Entra ID
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

The environment is designed around defense in depth, network segmentation, private administrative access, and separation of infrastructure roles.

Public project documentation focuses on architecture, implementation decisions, and technical learning while intentionally excluding credentials, secrets, authentication tokens, privileged account information, and externally reachable IP addresses.

---

## Learning and Troubleshooting

This project is intentionally built through hands-on implementation rather than architecture documentation alone.

Issues encountered during deployment are investigated and documented to demonstrate the troubleshooting process, including:

- Connectivity testing
- DNS validation
- Effective route analysis
- Network Security Group validation
- Subnet configuration
- Windows Server configuration
- Azure networking behavior

The `docs/troubleshooting.md` document will continue to grow as additional services and infrastructure are implemented.

---

## Purpose

This repository serves as both a technical portfolio and a record of my Azure learning journey.

The objective is to demonstrate practical experience designing, deploying, securing, administering, troubleshooting, and documenting an enterprise-style Microsoft Azure environment while continuously expanding the environment as new Azure administration concepts are learned.
