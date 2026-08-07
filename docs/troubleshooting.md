# Enterprise Azure Lab - Troubleshooting

## Overview

This document records technical issues encountered while building the Enterprise Azure Lab, including investigation steps, root causes, resolutions, and lessons learned.

The goal is to document real-world troubleshooting scenarios encountered during the deployment and continued development of the environment.

---

# Issue #1 - VM Had No Outbound Internet Connectivity

## Symptoms

The virtual machine was accessible through Azure Bastion but could not establish outbound HTTPS connectivity.

Observed behavior included:

- Azure Bastion connectivity was successful.
- DNS resolution functioned correctly.
- Windows Server displayed an activation warning.
- Outbound HTTPS connectivity failed.

The following PowerShell test was used:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Result:

```text
TcpTestSucceeded : False
```

---

## Investigation

The troubleshooting process validated each major component of the network path.

### Private IP Configuration

The virtual machine received a valid private IP address within its assigned subnet.

### DNS Resolution

DNS resolution was tested using:

```powershell
nslookup www.microsoft.com
```

Name resolution completed successfully, indicating that DNS was functioning.

### Effective Routes

Azure effective routes were reviewed and contained an active default route:

```text
0.0.0.0/0
Next Hop: Internet
State: Active
```

### Network Security Group

The applicable Network Security Group configuration permitted outbound Internet traffic.

### Route Table

No User Defined Route (UDR) was associated with the affected subnet.

These checks indicated that DNS, routing, and Network Security Group configuration were not preventing the connection.

---

## Root Cause

The affected subnet was originally configured as a **private subnet with default outbound access disabled**.

The subnet configuration was later changed to permit default outbound connectivity. However, the existing virtual machine had not yet picked up the updated network behavior.

---

## Resolution

The virtual machine was stopped and deallocated through Azure and then started again.

Resolution steps:

1. Stop and deallocate the virtual machine.
2. Start the virtual machine.
3. Reconnect through Azure Bastion.
4. Test outbound HTTPS connectivity.

Validation command:

```powershell
Test-NetConnection www.microsoft.com -Port 443
```

Successful result:

```text
TcpTestSucceeded : True
```

Outbound connectivity functioned after the VM was deallocated and restarted.

---

## Lesson Learned

Azure networking changes can require more than a guest operating system restart before the updated configuration is reflected by an existing virtual machine.

In this scenario, stopping and deallocating the VM and then starting it again resolved the connectivity issue after the subnet's outbound access configuration had been changed.

The troubleshooting process also demonstrated the value of testing network connectivity layer by layer rather than assuming that successful DNS resolution indicates full Internet connectivity.

---

# Future Troubleshooting Entries

Additional troubleshooting scenarios will be documented as the environment grows.

Potential areas include:

- Active Directory Domain Services
- DNS
- Domain joins
- Group Policy
- Azure Bastion
- Azure Firewall
- Azure Key Vault
- Azure Monitor
- Virtual Network Peering
- Microsoft Entra ID integration
