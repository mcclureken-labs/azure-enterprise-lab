# Enterprise Azure Lab - Troubleshooting

## Overview

This document records technical issues encountered while building the Enterprise Azure Lab, including investigation steps, root causes, resolutions, and lessons learned.

The goal is to document real-world troubleshooting scenarios encountered during the deployment and continued development of the environment.

The current network architecture is shown below for reference.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

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
