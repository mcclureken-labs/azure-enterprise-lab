# Server Inventory

This document provides a high-level inventory of server resources deployed within the Enterprise Azure Lab.

---

## Active Servers

| Server | Azure Resource | Role | Private IP | IP Allocation | Network Zone |
| --- | --- | --- | --- | --- | --- |
| DC01 | vm-dc01-corp-prd-eus2 | Active Directory Domain Services / DNS | 10.1.0.4 | Static | Identity |
| MGMT01 | vm-mgmt01-corp-prd-eus2 | Management Server | 10.1.1.4 | Dynamic | Management |

---

## DC01

**Role:** Active Directory Domain Services and DNS

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Purpose:**
- Provides centralized identity services for the lab environment
- Hosts Active Directory Domain Services
- Provides internal DNS services
- Supports authentication for domain-joined systems

**Network Configuration:**
- Located within the dedicated Identity subnet
- Static private IP addressing
- No direct public IP assigned

**Administrative Access:**
- Administrative access is performed through Azure Bastion
- Direct administrative exposure to the public Internet is not permitted

---

## MGMT01

**Role:** Management Server

**Operating System:** Windows Server 2025 Datacenter: Azure Edition

**Purpose:**
- Provides a dedicated system for infrastructure administration
- Separates routine management activities from the Domain Controller
- Will support remote administration tools for managing domain resources

**Network Configuration:**
- Located within the dedicated Management subnet
- Dynamic private IP addressing
- No direct public IP assigned

**Administrative Access:**
- Administrative access is performed through Azure Bastion
- Direct administrative exposure to the public Internet is not permitted

---

## Design Considerations

Server roles are separated across dedicated network segments to support network segmentation and reduce unnecessary exposure.

The Domain Controller uses static private addressing because other systems rely on it for Active Directory-integrated DNS and domain services.

The Management Server uses dynamic private addressing because other infrastructure does not depend on it being reachable at a fixed IP address.

Domain services and management functions are hosted on separate virtual machines rather than combining administrative workloads with the Domain Controller.

Azure Bastion provides administrative access without assigning public IP addresses directly to the server virtual machines.
