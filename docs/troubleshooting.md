# Enterprise Azure Lab - Troubleshooting

## Overview

This document records issues encountered while building the Enterprise Azure Lab, along with their root causes and resolutions.

The goal is to document real-world troubleshooting scenarios and lessons learned throughout the project.

---

# Issue #1 - VM Had No Outbound Internet Connectivity

## Symptoms

- Azure Bastion successfully connected to the VM.
- DNS resolution worked correctly.
- `Test-NetConnection www.microsoft.com -Port 443` returned:

```text
TcpTestSucceeded : False
```

- Windows Azure Edition displayed an activation warning.

---

## Investigation

The following components were verified:

✅ VM received a valid private IP address.

✅ Default gateway was configured correctly.

✅ DNS resolution worked using:

```powershell
nslookup www.microsoft.com
```

✅ Effective routes contained:

```text
0.0.0.0/0
Next Hop: Internet
State: Active
```

✅ Network Security Group allowed outbound Internet traffic.

✅ No User Defined Route (UDR) was associated with the subnet.

---

## Root Cause

The subnet was originally configured as a **Private Subnet (No Default Outbound Access)**.

Although the subnet configuration was later changed to allow default outbound Internet access, the VM networking configuration was not refreshed.

---

## Resolution

1. Stopped (Deallocated) the virtual machine.
2. Started the virtual machine.
3. Reconnected using Azure Bastion.
4. Verified outbound connectivity.

Validation command:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Successful result:

```text
TcpTestSucceeded : True
```

---

## Lesson Learned

Changing a subnet from private (no default outbound access) to allowing default outbound Internet access may require the virtual machine to be **Stopped (Deallocated)** before Azure applies the updated networking configuration.

A normal guest operating system restart is not sufficient.

---

# Future Troubleshooting Entries

Additional issues and resolutions will be documented here as the environment grows.

Examples include:

- Azure Bastion deployment
- Active Directory installation
- DNS configuration
- Group Policy
- Azure Firewall
- Azure Key Vault
- Azure Monitor
- Virtual Network Peering
- Microsoft Entra ID integration
