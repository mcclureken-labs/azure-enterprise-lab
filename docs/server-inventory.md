# Server Inventory

This document tracks all Windows Server virtual machines deployed within the Enterprise Azure Lab.

---

# MGMT01

## Purpose

Enterprise management server used to administer the Azure environment. This server provides a secure administrative workstation inside the Corporate VNet and is accessed exclusively through Azure Bastion.

---

## Azure Information

| Property | Value |
|----------|-------|
| Azure Resource Name | vm-mgmt01-prd-eus2 |
| Resource Group | rg-corporate-prd-eus2 |
| Region | East US 2 |
| Virtual Network | vnet-corporate-prd-eus2 |
| Subnet | snet-corp-management-prd-eus2 |
| Private IP | 10.1.1.4 |
| Public IP | None |

---

## Operating System

| Property | Value |
|----------|-------|
| Hostname | MGMT01 |
| Operating System | Windows Server 2025 Datacenter Azure Edition |
| Deployment | Azure Virtual Machine |

---

## Security

| Setting | Value |
|---------|-------|
| Management Method | Azure Bastion |
| Network Security Group | nsg-corp-management-prd-eus2 |
| Public RDP | Disabled |
| Public SSH | Disabled |

---

## Design Decisions

- Server deployed within the Corporate Management subnet.
- No Public IP assigned.
- Administrative access is provided exclusively through Azure Bastion.
- Network security is enforced at the subnet level using an NSG.
- Naming convention follows enterprise standards.

---

## Troubleshooting Notes

### Outbound Connectivity

Issue:

After changing the subnet from Private (no default outbound access) to allowing default outbound access, the VM still could not reach the Internet.

Resolution:

- Stopped (Deallocated) the VM.
- Started the VM.
- Verified outbound connectivity.

Command used:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```
TcpTestSucceeded : True
```

Lesson Learned:

Changing the subnet's outbound access configuration required the VM to be deallocated before Azure applied the updated networking behavior.

---
