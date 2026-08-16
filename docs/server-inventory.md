# Server Inventory

**Version:** 1.3  
**Last Updated:** August 16, 2026  
**Author:** Kendrick McClure

---

## Overview

This document provides a high-level inventory of server resources deployed within the Azure Enterprise Lab.

The current environment includes Windows Server infrastructure supporting Active Directory and centralized administration, along with a redundant Linux-based internal application tier.

The current server placement and network relationships are shown in the architecture topology below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [../diagrams/azure-enterprise-lab-topology.drawio](../diagrams/azure-enterprise-lab-topology.drawio).

---

## Active Servers

| Server | Operating System | Role | Private IP | IP Allocation | Network Zone |
| --- | --- | --- | --- | --- | --- |
| DC01 | Windows Server 2025 Datacenter: Azure Edition | Active Directory Domain Services / DNS | 10.1.0.4 | Static | Identity |
| MGMT01 | Windows Server 2025 Datacenter: Azure Edition | Domain Management / Administration | 10.1.1.4 | Dynamic | Management |
| WEB01 | Ubuntu Server 24.04 LTS | Nginx Web Server | 10.1.3.4 | Dynamic | Internal Apps |
| WEB02 | Ubuntu Server 24.04 LTS | Nginx Web Server | 10.1.3.5 | Dynamic | Internal Apps |

---

## DC01

**Azure Resource:** `vm-dc01-corp-prd-eus2`

**Role:** Active Directory Domain Services and DNS

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Domain:** `corp.mccluretech.com`

### Purpose

DC01 provides the core identity and DNS services for the Windows environment.

Current services include:

- Active Directory Domain Services
- Active Directory-integrated DNS
- Domain authentication
- Active Directory service discovery
- Global Catalog services

### Network Configuration

- Subnet: `snet-identity-prd-eus2`
- Address Space: 10.1.0.0/24
- Private IP: 10.1.0.4
- IP Allocation: Static
- No direct public IP assigned
- Outbound Connectivity: Azure NAT Gateway

DC01 uses static private addressing because domain-connected systems depend on a consistent endpoint for Active Directory-integrated DNS and directory services.

---

## MGMT01

**Azure Resource:** `vm-mgmt01-corp-prd-eus2`

**Role:** Domain Management and Administration

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Domain:** `corp.mccluretech.com`

### Purpose

MGMT01 provides a dedicated domain-joined platform for Windows infrastructure administration.

Current functions include:

- Active Directory administration
- DNS administration
- Group Policy administration
- Windows infrastructure management
- File-services and authorization testing

### Active Directory Configuration

- Joined to the `corp.mccluretech.com` domain
- Computer object located within the custom `Servers` OU
- Uses Active Directory-integrated DNS hosted on DC01
- Domain authentication validated

### Administrative Tooling

Remote Server Administration Tools installed on MGMT01 include:

- Active Directory Administrative Center
- Active Directory Domains and Trusts
- Active Directory Sites and Services
- Active Directory Users and Computers
- Active Directory PowerShell module
- DNS Manager
- Group Policy Management

### File Services

MGMT01 hosts the lab's initial `ITShare` resource used for Active Directory group-based authorization and NTFS permission testing.

Detailed authorization design is maintained in the [Active Directory Design](active-directory.md) documentation.

### Network Configuration

- Subnet: `snet-corp-management-prd-eus2`
- Address Space: 10.1.1.0/24
- Private IP: 10.1.1.4
- IP Allocation: Dynamic
- No direct public IP assigned
- Network Security Group: `nsg-corp-management-prd-eus2`
- Outbound Connectivity: Azure NAT Gateway

---

## WEB01

**Azure Resource:** `vm-web01-corp-prd-eus2`

**Role:** Internal Nginx Web Server

**Operating System:** Ubuntu Server 24.04 LTS

### Purpose

WEB01 provides one backend instance for the internal web application tier.

Current functions include:

- Hosting Nginx
- Serving internal HTTP traffic on TCP port 80
- Participating in the internal Azure Load Balancer backend pool

### Network Configuration

- Subnet: `snet-corporate-internal-apps-prd-eus2`
- Address Space: 10.1.3.0/24
- Private IP: 10.1.3.4
- IP Allocation: Dynamic
- No direct public IP assigned
- Network Security Group: `nsg-corporate-internal-apps-prd-eus2`
- Outbound Connectivity: Azure NAT Gateway

### Load Balancing

WEB01 is a backend member of `lb-corporate-internal-apps-prd-eus2`.

Nginx listens on TCP port 80 and participates in the Load Balancer health-monitoring configuration.

---

## WEB02

**Azure Resource:** `vm-web02-corp-prd-eus2`

**Role:** Internal Nginx Web Server

**Operating System:** Ubuntu Server 24.04 LTS

### Purpose

WEB02 provides the second backend instance for the internal web application tier.

Current functions include:

- Hosting Nginx
- Serving internal HTTP traffic on TCP port 80
- Participating in the internal Azure Load Balancer backend pool

### Network Configuration

- Subnet: `snet-corporate-internal-apps-prd-eus2`
- Address Space: 10.1.3.0/24
- Private IP: 10.1.3.5
- IP Allocation: Dynamic
- No direct public IP assigned
- Network Security Group: `nsg-corporate-internal-apps-prd-eus2`
- Outbound Connectivity: Azure NAT Gateway

### Load Balancing

WEB02 is a backend member of `lb-corporate-internal-apps-prd-eus2`.

Nginx listens on TCP port 80 and participates in the Load Balancer health-monitoring configuration.

---

## Internal Application Tier

WEB01 and WEB02 form the current redundant internal web application tier.

Both servers:

- Run Ubuntu Server 24.04 LTS
- Host Nginx
- Serve HTTP traffic on TCP port 80
- Reside within `snet-corporate-internal-apps-prd-eus2`
- Remain privately addressed
- Participate in the backend pool for `lb-corporate-internal-apps-prd-eus2`

The internal Azure Load Balancer uses the static private frontend address 10.1.3.10.

The current Load Balancer configuration includes:

- Frontend Protocol: TCP
- Frontend Port: 80
- Backend Port: 80
- Backend Servers: WEB01 and WEB02
- Health Probe: TCP/80

Detailed Load Balancer architecture is maintained in the [Network Design](network-design.md) documentation, while functional testing and failover validation are maintained in [Troubleshooting](troubleshooting.md).

---

## Server Relationships

DC01 provides Active Directory Domain Services and Active Directory-integrated DNS for the Windows environment. MGMT01 is joined to the `corp.mccluretech.com` domain and serves as the primary administrative platform for managing those services.

WEB01 and WEB02 form the internal Nginx application tier and participate in the backend pool of the internal Azure Load Balancer.

All four server workloads remain privately addressed within their respective Corporate network segments.

---

## Future Improvements

Planned server infrastructure improvements include:

- Additional Windows Server workloads
- Additional Linux application workloads
- Expanded remote administration capabilities
- Additional Group Policy configuration
- Additional file-services scenarios
- Windows Server auditing and logging
- Centralized monitoring
- Additional application availability scenarios
- PowerShell administration and automation
- Infrastructure as Code deployment

---

## Related Documentation

Additional implementation details are maintained in:

- [Network Design](network-design.md)
- [Security Design](security-design.md)
- [Active Directory Design](active-directory.md)
- [Naming Conventions](naming-convention.md)
- [Troubleshooting](troubleshooting.md)

---

## Summary

The Azure Enterprise Lab currently contains four server workloads distributed across dedicated Identity, Management, and Internal Application network segments.

DC01 provides Active Directory Domain Services and DNS, MGMT01 provides centralized Windows infrastructure administration, and WEB01 and WEB02 form a redundant Nginx application tier behind an internal Azure Load Balancer.

The server inventory provides a concise reference for the roles, placement, addressing, and key infrastructure relationships of the currently deployed compute resources.
