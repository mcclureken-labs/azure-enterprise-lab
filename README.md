# Azure Enterprise Lab

## Overview

This repository documents the design, deployment, administration, troubleshooting, and continued development of a fictional enterprise environment built in Microsoft Azure.

The project is designed to develop hands-on Azure infrastructure, networking, systems administration, identity, and security skills through the implementation of enterprise-style services and architectural patterns.

The environment is being built incrementally alongside preparation for the Microsoft Azure Administrator (AZ-104) certification, with an emphasis on understanding not only how Azure services are configured, but why specific architectural decisions are made and how those decisions affect security, availability, and manageability.

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
- Implement Group Policy and group-based resource authorization
- Deploy and validate an internal load-balanced application tier
- Practice real-world infrastructure troubleshooting
- Implement monitoring and security services
- Develop PowerShell and infrastructure automation skills
- Explore Infrastructure as Code
- Maintain professional technical documentation throughout the project

---

## Current Architecture

The environment uses a Hub-and-Spoke architecture consisting of a centralized Hub Virtual Network and a Corporate workload Virtual Network.

The Hub provides shared connectivity and administrative infrastructure. Azure Bastion provides private administrative connectivity to server workloads across VNet peering without assigning direct public IP addresses to the virtual machines.

The Corporate VNet separates Identity, Management, and Internal Application workloads into dedicated network segments.

Azure NAT Gateway currently provides explicit outbound Internet connectivity for the active private Corporate workload subnets. NAT Gateway performs Source Network Address Translation (SNAT) using a dedicated static Public IP resource while allowing the server virtual machines to remain privately addressed.

Active Directory Domain Services and Active Directory-integrated DNS are hosted on DC01. The Corporate VNet uses DC01 as its custom DNS server, allowing domain-connected systems such as MGMT01 to locate Active Directory services and resolve the internal namespace.

MGMT01 provides a dedicated domain-joined administrative platform for Active Directory, DNS, Group Policy, and Windows infrastructure administration.

The Internal Apps subnet hosts two Ubuntu Nginx web servers, WEB01 and WEB02, behind an internal Azure Load Balancer. The Load Balancer provides a static private application frontend at 10.1.3.10, distributes TCP/80 traffic across healthy backend systems, and uses health probing to detect backend availability.

![Azure Enterprise Lab Topology](images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [diagrams/azure-enterprise-lab-topology.drawio](diagrams/azure-enterprise-lab-topology.drawio).

### Architecture Highlights

- Hub-and-Spoke network topology
- Bidirectional VNet peering
- Dedicated Identity, Management, and Internal Apps subnets
- Subnet-level Network Security Groups
- Azure Bastion for private administrative access
- No direct public IP addresses assigned to server virtual machines
- Azure NAT Gateway for explicit outbound Internet connectivity
- Dedicated static Public IP resource for NAT Gateway egress
- Active Directory Domain Services and DNS hosted on DC01
- Corporate VNet configured to use Active Directory DNS
- Domain-joined MGMT01 administrative server
- Group Policy-based Windows Server configuration
- AGDLP-style group-based resource authorization
- Internal Azure Load Balancer
- Redundant WEB01 and WEB02 Nginx backends
- TCP/80 backend health monitoring
- Validated application failover between backend servers
- Reserved network segments for future Azure Firewall, gateway, and Private Endpoint services

---

## Server Infrastructure

### DC01

**Azure Resource:** `vm-dc01-corp-prd-eus2`

- Windows Server 2025 Datacenter: Azure Edition
- Active Directory Domain Services
- Active Directory-integrated DNS
- Global Catalog
- Static private IP 10.1.0.4
- Dedicated Identity subnet
- No direct public IP
- Explicit outbound connectivity through Azure NAT Gateway

### MGMT01

**Azure Resource:** `vm-mgmt01-corp-prd-eus2`

- Windows Server 2025 Datacenter: Azure Edition
- Member of the corp.mccluretech.com domain
- Computer object located within the custom Servers OU
- Private IP 10.1.1.4
- Dedicated Management subnet
- Uses Active Directory DNS hosted on DC01
- Dedicated platform for Windows infrastructure administration
- Active Directory, DNS, and Group Policy administrative tooling
- Hosts the initial ITShare file resource
- No direct public IP
- Explicit outbound connectivity through Azure NAT Gateway

### WEB01

**Azure Resource:** `vm-web01-corp-prd-eus2`

- Ubuntu Server
- Nginx web server
- Private IP 10.1.3.4
- Dedicated Internal Apps subnet
- Internal Load Balancer backend
- No direct public IP
- Explicit outbound connectivity through Azure NAT Gateway

### WEB02

**Azure Resource:** `vm-web02-corp-prd-eus2`

- Ubuntu Server
- Nginx web server
- Private IP 10.1.3.5
- Dedicated Internal Apps subnet
- Internal Load Balancer backend
- No direct public IP
- Explicit outbound connectivity through Azure NAT Gateway

---

## Implemented Identity and Access Model

The Windows environment uses Active Directory security groups to separate user-role membership from resource permissions.

The initial ITShare implementation follows an AGDLP-style authorization model:

```text
User Account
    ↓
GG-IT-Users
    ↓
DL-ITShare-RW
    ↓
ITShare
    ↓
NTFS Modify
```

This allows users to be managed through role-based Global group membership while resource permissions are assigned to a Domain Local group.

---

## Implemented Group Policy

A dedicated server baseline Group Policy Object is used for server-specific Windows security configuration.

Current GPO:

```text
GPO-Servers-Baseline
```

The GPO is linked to the custom Servers OU.

The initial baseline configures:

```text
Accounts: Guest account status → Disabled
```

The server baseline is maintained separately from the Default Domain Policy to provide a scalable location for future Windows Server security settings.

---

## Current Progress

### Phase 1 - Enterprise Networking

- [x] Resource organization
- [x] Enterprise IP addressing plan
- [x] Hub Virtual Network
- [x] Corporate Virtual Network
- [x] Infrastructure and workload subnets
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
- [x] Remote Active Directory administration tooling
- [x] DNS administration tooling
- [x] Group Policy administration tooling

### Phase 3 - Identity and Windows Administration

- [x] Windows Server Domain Controller deployment
- [x] Active Directory Domain Services
- [x] Active Directory-integrated DNS
- [x] Active Directory forest and domain
- [x] Organizational Unit structure
- [x] Corporate VNet DNS configuration
- [x] Domain-join MGMT01
- [x] Move MGMT01 computer object to Servers OU
- [x] Validate domain authentication
- [x] Configure remote administration tools
- [x] Create standard test user accounts
- [x] Create security groups
- [x] Implement AGDLP-style group nesting
- [x] Implement Group Policy
- [x] Create server baseline GPO
- [x] Implement Windows file-share authorization
- [x] Configure NTFS permissions
- [x] Review and remediate inherited permissions
- [ ] Create dedicated service accounts
- [ ] Expand Group Policy security baseline

### Phase 4 - Internal Application Tier

- [x] Internal Apps subnet
- [x] Internal Apps Network Security Group
- [x] WEB01 deployment
- [x] WEB02 deployment
- [x] Nginx installation and configuration
- [x] Internal Azure Load Balancer
- [x] Static private frontend configuration
- [x] Backend pool configuration
- [x] TCP/80 health probe
- [x] TCP/80 load-balancing rule
- [x] NAT Gateway association
- [x] Internal application connectivity validation
- [x] Backend failure simulation
- [x] Application failover validation

### Future Phases

- [ ] Azure Firewall
- [ ] User Defined Routes
- [ ] Microsoft Defender for Cloud
- [ ] Azure Policy
- [ ] Azure Key Vault
- [ ] Azure Monitor and centralized logging
- [ ] Microsoft Entra ID integration
- [ ] Private Endpoints
- [ ] Additional identity security controls
- [ ] Additional Group Policy hardening
- [ ] Additional Windows Server workloads
- [ ] TLS for internal application workloads
- [ ] PowerShell automation
- [ ] Bicep
- [ ] Terraform

---

## Troubleshooting and Validation

The project documents troubleshooting and validation performed throughout implementation rather than only recording final configuration state.

Examples include:

- Diagnosed private-subnet outbound connectivity and implemented Azure NAT Gateway as the current explicit egress solution
- Validated NAT Gateway SNAT and outbound connectivity while maintaining private workload addressing
- Configured and validated Active Directory DNS dependencies and domain integration
- Identified and remediated overly broad inherited NTFS permissions
- Validated internal application availability through backend failure testing

Detailed investigation, remediation, and lessons learned are maintained in the [Troubleshooting](docs/troubleshooting.md) documentation.

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

The documentation covers architecture decisions, implementation details, security controls, troubleshooting scenarios, validation, and lessons learned as the environment evolves.

---

## Key Skills Demonstrated

This project currently demonstrates hands-on experience with:

- Microsoft Azure administration
- Azure Virtual Networks, subnetting, VNet peering, and Hub-and-Spoke architecture
- Network Security Groups and private workload networking
- Azure Bastion
- Azure NAT Gateway and SNAT
- Azure Load Balancer, health probing, and failover validation
- Windows Server 2025 administration
- Active Directory Domain Services and Active Directory-integrated DNS
- Organizational Unit design, domain joining, and authentication
- Active Directory security groups and AGDLP-style authorization
- Group Policy and Windows security configuration
- Windows NTFS permissions and access-control troubleshooting
- Linux server administration and Nginx
- Network and infrastructure troubleshooting
- Enterprise naming conventions and technical architecture documentation

---

## Project Status

**Active Development**

The environment will continue to expand as additional Azure administration, security, monitoring, automation, and Infrastructure as Code concepts are implemented.

Future work will build on the existing architecture rather than replacing it, allowing the repository to document the evolution of the environment from foundational networking and identity services into a broader enterprise-style Azure platform.
