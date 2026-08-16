# Server Inventory

**Version:** 1.2  
**Last Updated:** August 15, 2026  
**Author:** Kendrick McClure

---

## Overview

This document provides a high-level inventory of server resources deployed within the Azure Enterprise Lab.

The current environment includes Windows Server infrastructure supporting Active Directory and centralized administration, along with a redundant Linux web application tier hosted behind an internal Azure Load Balancer.

The current server placement and network relationships are shown in the architecture topology below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

## Active Servers

| Server | Azure Resource | Operating System | Role | Private IP | IP Allocation | Network Zone |
| --- | --- | --- | --- | --- | --- | --- |
| DC01 | vm-dc01-corp-prd-eus2 | Windows Server 2025 Datacenter: Azure Edition | Active Directory Domain Services / DNS | 10.1.0.4 | Static | Identity |
| MGMT01 | vm-mgmt01-corp-prd-eus2 | Windows Server 2025 Datacenter: Azure Edition | Domain Management / Administration | 10.1.1.4 | Dynamic | Management |
| WEB01 | vm-web01-corp-prd-eus2 | Ubuntu Server | Nginx Web Server | 10.1.3.4 | Dynamic | Internal Apps |
| WEB02 | vm-web02-corp-prd-eus2 | Ubuntu Server | Nginx Web Server | 10.1.3.5 | Dynamic | Internal Apps |

---

## DC01

**Azure Resource:** `vm-dc01-corp-prd-eus2`

**Role:** Active Directory Domain Services and DNS

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Domain:** `corp.mccluretech.com`

### Purpose

DC01 provides the core identity and name-resolution services for the Corporate environment.

Primary responsibilities include:

- Hosting Active Directory Domain Services
- Hosting Active Directory-integrated DNS
- Providing centralized identity and authentication services
- Providing DNS resolution for domain-connected systems
- Supporting Active Directory service discovery
- Operating as a Global Catalog server

### Network Configuration

- Located within `snet-identity-prd-eus2`
- Subnet: `10.1.0.0/24`
- Private IP: `10.1.0.4`
- IP allocation: Static
- No direct public IP assigned
- Identity subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway

Static private addressing is used because domain-connected systems depend on DC01 as the internal DNS server and Active Directory infrastructure requires a consistent DNS endpoint.

### Administrative Access

Administrative connectivity is performed through Azure Bastion when direct server access is required.

Direct RDP exposure to the public Internet is not permitted.

Routine Active Directory administration is performed from the dedicated MGMT01 management server where practical, reducing the need to perform day-to-day administrative work directly from the Domain Controller.

---

## MGMT01

**Azure Resource:** `vm-mgmt01-corp-prd-eus2`

**Role:** Domain Management and Administration Server

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Domain:** `corp.mccluretech.com`

### Purpose

MGMT01 provides a dedicated administrative platform for the Windows and Active Directory environment.

Primary responsibilities include:

- Providing a domain-joined administrative system
- Separating routine administrative activity from the Domain Controller
- Hosting Windows and Active Directory administration tools
- Providing centralized access to Active Directory, DNS, and Group Policy management consoles
- Supporting Active Directory and access-control testing
- Hosting the lab's initial IT file-share resource

### Active Directory Configuration

- Joined to the `corp.mccluretech.com` domain
- Computer object located within the custom `Servers` OU
- Domain authentication successfully validated
- Uses Active Directory-integrated DNS hosted on DC01
- Registered within the `corp.mccluretech.com` DNS namespace

### Administrative Tooling

MGMT01 has Windows remote administration tooling available for managing domain infrastructure, including:

- Active Directory Administrative Center
- Active Directory Domains and Trusts
- Active Directory Sites and Services
- Active Directory Users and Computers
- Active Directory PowerShell module
- DNS Manager
- Group Policy Management

MGMT01 is used as the primary administrative platform for these services rather than performing routine management directly from DC01.

DC01 has not currently been added to the MGMT01 Server Manager server pool. Centralized Server Manager-based administration remains a future extension of the management design.

### File Services

MGMT01 hosts the lab's initial `ITShare` resource used to implement and validate Active Directory group-based authorization.

Access is structured through:

- `GG-IT-Users` as the Global security group representing authorized IT users
- `DL-ITShare-RW` as the Domain Local security group representing access to the resource
- `GG-IT-Users` nested within `DL-ITShare-RW`
- `DL-ITShare-RW` assigned NTFS Modify permissions to `ITShare`

This provides an AGDLP-style authorization model and avoids assigning resource permissions directly to individual user accounts.

### Network Configuration

- Located within `snet-corp-management-prd-eus2`
- Subnet: `10.1.1.0/24`
- Private IP: `10.1.1.4`
- IP allocation: Dynamic
- No direct public IP assigned
- Corporate Management subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway
- Subnet protected by `nsg-corp-management-prd-eus2`

### Administrative Access

Administrative connectivity is performed through Azure Bastion.

Domain authentication is used for Active Directory administration.

Direct RDP exposure to the public Internet is not permitted.

---

## WEB01

**Azure Resource:** `vm-web01-corp-prd-eus2`

**Role:** Internal Nginx Web Server

**Operating System:** Ubuntu Server

### Purpose

WEB01 provides one of two backend web servers used to implement and validate the internal application tier.

Primary responsibilities include:

- Hosting the Nginx web service
- Serving HTTP traffic on TCP port 80
- Participating in the Azure Load Balancer backend pool
- Providing a distinguishable web response for load-balancing and failover validation

The Nginx landing page identifies the responding server as WEB01, allowing backend selection to be observed during testing.

### Network Configuration

- Located within `snet-corporate-internal-apps-prd-eus2`
- Subnet: `10.1.3.0/24`
- Private IP: `10.1.3.4`
- IP allocation: Dynamic
- No direct public IP assigned
- Internal Apps subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway
- Subnet protected by `nsg-corporate-internal-apps-prd-eus2`

### Load Balancing

WEB01 is a member of the backend pool associated with:

`lb-corporate-internal-apps-prd-eus2`

The application tier uses:

- Internal Load Balancer frontend: `10.1.3.10`
- Frontend protocol and port: TCP/80
- Backend port: TCP/80
- Health probe: TCP/80

WEB01 receives application traffic through the Load Balancer while it is considered healthy by the configured health probe.

---

## WEB02

**Azure Resource:** `vm-web02-corp-prd-eus2`

**Role:** Internal Nginx Web Server

**Operating System:** Ubuntu Server

### Purpose

WEB02 provides the second backend web server within the internal application tier.

Primary responsibilities include:

- Hosting the Nginx web service
- Serving HTTP traffic on TCP port 80
- Participating in the Azure Load Balancer backend pool
- Providing service availability if another backend becomes unavailable
- Supporting load-balancing and failover validation

The Nginx landing page identifies the responding server as WEB02, allowing backend selection to be observed during testing.

### Network Configuration

- Located within `snet-corporate-internal-apps-prd-eus2`
- Subnet: `10.1.3.0/24`
- Private IP: `10.1.3.5`
- IP allocation: Dynamic
- No direct public IP assigned
- Internal Apps subnet configured as a private subnet
- Explicit outbound Internet connectivity provided through Azure NAT Gateway
- Subnet protected by `nsg-corporate-internal-apps-prd-eus2`

### Load Balancing

WEB02 is a member of the backend pool associated with:

`lb-corporate-internal-apps-prd-eus2`

The application tier uses:

- Internal Load Balancer frontend: `10.1.3.10`
- Frontend protocol and port: TCP/80
- Backend port: TCP/80
- Health probe: TCP/80

WEB02 provides continued application availability when another backend server is unavailable.

---

## Internal Application Tier

WEB01 and WEB02 form a redundant internal web application tier within the Corporate Virtual Network.

Both servers run Nginx and are members of the backend pool associated with the internal Azure Load Balancer.

The Load Balancer provides a single private frontend address:

`10.1.3.10`

Client traffic directed to TCP port 80 on the frontend is distributed across healthy backend servers listening on TCP port 80.

The Load Balancer health probe monitors TCP port 80 to determine backend availability and prevents unhealthy instances from continuing to receive new application traffic.

The Load Balancer is used for internal application delivery and is not responsible for outbound Internet connectivity.

Outbound connectivity for WEB01 and WEB02 is provided separately through the Azure NAT Gateway associated with the Internal Apps subnet.

This separates application delivery from outbound Internet egress and allows the backend servers to remain privately addressed.

---

## Load Balancer Failover Validation

The internal application tier was functionally tested from MGMT01 using the Load Balancer frontend address.

An HTTP request to:

`http://10.1.3.10`

successfully returned content from WEB01.

WEB01 was then intentionally shut down to simulate a backend server failure.

After the unavailable backend was detected by the Load Balancer health mechanism, another request to the same frontend address successfully returned content from WEB02.

The validation demonstrated that:

- The internal Load Balancer frontend was reachable from the Corporate network
- Backend pool membership was functioning
- Nginx was responding successfully on both backend servers
- The health probe could identify backend availability
- Application traffic could be served by multiple backend systems
- Service remained available when one backend server became unavailable
- Clients continued using the same Load Balancer frontend address during backend failure

This test validated the basic high-availability and failover behavior of the internal application tier.

---

## Server Relationships

DC01, MGMT01, WEB01, and WEB02 are separated according to infrastructure function and network zone.

DC01 provides Active Directory Domain Services and Active Directory-integrated DNS for the Corporate environment.

MGMT01 is joined to the `corp.mccluretech.com` domain and uses DC01 at `10.1.0.4` for DNS resolution and Active Directory service discovery. MGMT01 also provides the primary administrative platform for Active Directory, DNS, Group Policy, and the lab's initial Windows file-service implementation.

WEB01 and WEB02 provide the redundant internal web application tier. Clients access the service through the internal Azure Load Balancer at `10.1.3.10` rather than targeting the individual backend servers directly.

All four server virtual machines remain privately addressed.

Administrative connectivity is provided through Azure Bastion when required, while explicit outbound Internet connectivity is provided through Azure NAT Gateway rather than direct public IP assignments or implicit default outbound access.

The complete relationship between these workloads and their respective network segments is represented in the topology diagram at the beginning of this document.

---

## Design Considerations

Server workloads are separated according to their infrastructure function to support network segmentation and reduce unnecessary exposure.

The Domain Controller uses static private addressing because other systems rely on it for Active Directory-integrated DNS and domain services.

MGMT01 uses dynamic private addressing because infrastructure services do not currently depend on it being reachable through a fixed private address.

WEB01 and WEB02 also use dynamic private addressing because the application is consumed through the Load Balancer frontend rather than by clients directly targeting individual backend addresses.

Domain services and management functions are hosted on separate virtual machines rather than combining routine administrative workloads with the Domain Controller.

The web application workload is separated into a dedicated Internal Apps subnet and distributed across two backend servers.

The Identity, Corporate Management, and Internal Apps subnets use explicit outbound connectivity through Azure NAT Gateway.

Azure Bastion provides administrative access without requiring direct public IP addresses on the virtual machines.

The internal Azure Load Balancer provides a stable private application endpoint while allowing individual backend systems to be maintained or become unavailable without requiring clients to target a different service address.

---

## Current Server Summary

| Server | Primary Function | Key Design Characteristic |
| --- | --- | --- |
| DC01 | Identity and DNS | Static infrastructure endpoint |
| MGMT01 | Windows and AD administration | Dedicated domain management platform |
| WEB01 | Internal web backend | Load-balanced Nginx instance |
| WEB02 | Internal web backend | Load-balanced Nginx instance |

---

## Future Improvements

Planned server infrastructure improvements include:

- Expanded centralized remote server administration
- Additional Group Policy security controls
- Additional Windows Server workloads
- Additional internal application workloads
- Application health and availability monitoring
- Centralized logging and monitoring
- Microsoft Defender for Cloud integration
- PowerShell-based server administration and automation
- Infrastructure as Code deployment using Bicep and Terraform

---

## Related Documentation

Additional implementation details are maintained in:

- [Active Directory Design](active-directory.md)
- [Network Design](network-design.md)
- [Security Design](security-design.md)
- [Naming Conventions](naming-convention.md)
- [Troubleshooting](troubleshooting.md)

---

## Summary

The Azure Enterprise Lab currently contains four server workloads distributed across dedicated Identity, Management, and Internal Apps network segments.

DC01 provides Active Directory Domain Services and DNS, MGMT01 provides the primary Windows administrative platform and initial file-services workload, and WEB01 and WEB02 provide a redundant Nginx-based internal application tier.

The server architecture demonstrates role separation, network segmentation, private administration, explicit outbound connectivity, centralized identity services, domain-based administration, group-based resource authorization, and internal application load balancing with backend health validation and failover testing.
