# Active Directory Design

**Version:** 1.2  
**Last Updated:** August 10, 2026  
**Author:** Kendrick McClure

---

## Overview

This document outlines the Active Directory architecture, configuration, and administrative design for the Enterprise Azure Lab.

Active Directory Domain Services (AD DS) provides centralized identity, authentication, DNS, and service discovery for the Windows infrastructure within the Corporate network.

The Active Directory infrastructure is integrated with the broader Azure network architecture shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

## Environment Information

| Setting                 | Value                                         |
| ----------------------- | --------------------------------------------- |
| Forest Name             | corp.mccluretech.com                          |
| Domain Name             | corp.mccluretech.com                          |
| NetBIOS Name            | CORP                                          |
| Forest Functional Level | Windows Server 2025                           |
| Domain Functional Level | Windows Server 2025                           |
| Domain Controller       | DC01                                          |
| Azure Virtual Machine   | vm-dc01-corp-prd-eus2                         |
| Private IP              | 10.1.0.4                                      |
| IP Allocation           | Static                                        |
| Operating System        | Windows Server 2025 Datacenter: Azure Edition |

---

## Domain Controller

| Server | Role                                   | Status |
| ------ | -------------------------------------- | ------ |
| DC01   | Active Directory Domain Services / DNS | Active |

DC01 is located within the dedicated Identity subnet of the Corporate Virtual Network.

The server uses static private addressing to provide a consistent network endpoint for directory and DNS services.

DC01 does not have a direct public IP address and is administratively accessed through Azure Bastion.

Separating the Domain Controller from routine administrative workloads provides a foundation for applying more specific network and administrative controls as the environment develops.

---

## DNS Configuration

DNS was installed alongside Active Directory Domain Services and provides name resolution and service discovery for the Active Directory environment.

Current configuration includes:

- Active Directory-integrated DNS
- DNS hosted on DC01
- Internal DNS namespace: `corp.mccluretech.com`
- DC01 private IP: `10.1.0.4`
- Static private addressing for the DNS server
- Corporate VNet configured to use `10.1.0.4` as its custom DNS server
- DNS delegation not configured for the standalone lab environment

The Corporate VNet-level DNS configuration directs workloads using the VNet-provided DNS settings to DC01.

MGMT01 therefore uses the Active Directory DNS service hosted on DC01, allowing the domain-joined management server to resolve the internal namespace and locate Active Directory services.

External DNS queries can be resolved through the DNS server's recursive resolution and configured forwarding behavior.

---

## Organizational Unit Structure

The following Organizational Units were created to provide logical separation for administrative objects and future Group Policy implementation.

```text
corp.mccluretech.com
│
├── Admins
├── Groups
├── Servers
├── User Accounts
├── Workstations
├── Service Accounts
└── Disabled Objects
