# Enterprise Azure Lab - Security Design

**Version:** 1.3  
**Last Updated:** August 15, 2026  
**Author:** Kendrick McClure

---

## Overview

This document outlines the security architecture, implemented controls, and security-focused design decisions within the Azure Enterprise Lab.

The environment uses layered security controls, network segmentation, private administrative access, centralized identity, group-based authorization, workload separation, and explicitly defined outbound connectivity to reduce unnecessary exposure of infrastructure resources.

Security controls are implemented across multiple layers of the environment, including:

- Azure network architecture
- Network Security Groups
- Private workload addressing
- Administrative access
- Active Directory Domain Services
- Group Policy
- Security group-based authorization
- Windows NTFS permissions
- Internal application delivery
- Explicit outbound Internet connectivity

The current architecture is shown below.

![Azure Enterprise Lab Topology](../images/azure-enterprise-lab-topology.png)

The editable source for the architecture diagram is maintained in [`../diagrams/azure-enterprise-lab-topology.drawio`](../diagrams/azure-enterprise-lab-topology.drawio).

---

# Security Principles

The environment is designed around the following principles:

- Least privilege
- Defense in depth
- Network segmentation
- Private administrative access
- Explicit outbound connectivity
- Separation of infrastructure roles
- Group-based authorization
- Reduced public exposure
- Centralized identity and policy management
- Consistent resource organization

These principles guide both the current implementation and future security enhancements planned for the environment.

---

# Defense-in-Depth Architecture

Security is implemented through multiple complementary controls rather than relying on a single security mechanism.

Current security layers include:

1. **Network segmentation** separates infrastructure according to function.
2. **Network Security Groups** provide subnet-level traffic filtering.
3. **Private addressing** prevents server workloads from requiring direct public IP addresses.
4. **Azure Bastion** provides a centralized administrative access path.
5. **Azure NAT Gateway** provides explicit outbound connectivity without creating unsolicited inbound access.
6. **Active Directory Domain Services** provides centralized authentication and identity management.
7. **Group Policy** provides centralized Windows security configuration.
8. **Active Directory security groups** provide role-based resource authorization.
9. **NTFS permissions** enforce access controls on Windows file resources.
10. **Internal Azure Load Balancer** provides private application delivery without publicly exposing the backend web servers.

These controls operate at different layers and provide a foundation for introducing additional security services as the environment develops.

---

# Network Security Groups

Network Security Groups are used to control traffic at the subnet level and provide network-layer access controls between infrastructure segments.

## Hub Management

| Property | Value |
| --- | --- |
| Name | nsg-hub-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-hub-management-prd-eus2 |

### Purpose

Protects shared management infrastructure located within the Hub Virtual Network.

---

## Corporate Management

| Property | Value |
| --- | --- |
| Name | nsg-corp-management-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-corp-management-prd-eus2 |

### Purpose

Protects administrative systems deployed within the Corporate Virtual Network, including MGMT01.

---

## Internal Applications

| Property | Value |
| --- | --- |
| Name | nsg-corporate-internal-apps-prd-eus2 |
| Resource Group | rg-security-prd-eus2 |
| Associated Subnet | snet-corporate-internal-apps-prd-eus2 |

### Purpose

Protects the Internal Apps subnet containing WEB01, WEB02, and the private application tier.

The current Nginx workload uses HTTP on TCP port 80.

HTTPS on TCP port 443 is reserved for future TLS-enabled application traffic but is not currently implemented by the Nginx workload.

Network Security Groups provide traffic filtering but are not treated as replacements for centralized firewalling, application-layer inspection, endpoint protection, or identity-based access controls.

---

# Administrative Access

Administrative access to server virtual machines is centralized through Azure Bastion when interactive server access is required.

DC01, MGMT01, WEB01, and WEB02 do not have direct public IP addresses.

## Azure Bastion

| Property | Value |
| --- | --- |
| Name | bas-hub-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-bastion-hub-prd-eus2 |

The Azure resource name of the Bastion Public IP is documented for architectural reference. The assigned public IP address itself is intentionally excluded from the public project documentation.

Azure Bastion is deployed within the Hub Virtual Network when required and reaches privately addressed server resources in the Corporate Virtual Network across VNet peering.

### Security Benefits

- No direct public IP addresses assigned to server workloads
- Browser-based administrative connectivity
- RDP access without directly exposing RDP to the public Internet
- SSH access without directly exposing SSH to the public Internet
- Private IP connectivity between Azure Bastion and managed virtual machines
- Centralized administrative access
- Reduced direct exposure of server infrastructure

Because Azure Bastion is a billable lab resource, it may be deployed or removed as required while the underlying private server architecture remains unchanged.

---

# Server Security

## DC01

DC01 provides Active Directory Domain Services and Active Directory-integrated DNS for the environment.

Security considerations include:

- No direct public IP address
- Administrative access through Azure Bastion when interactive access is required
- Dedicated Identity subnet
- Static private IP addressing for reliable directory and DNS services
- Domain Controller role separated from routine management workloads
- Routine administration performed from MGMT01 where practical
- Active Directory-integrated DNS
- Dedicated Domain Controllers Organizational Unit

Separating the Domain Controller from routine administrative workloads reduces the amount of day-to-day activity performed directly on critical identity infrastructure.

---

## MGMT01

MGMT01 provides a dedicated domain-joined platform for Windows and Active Directory administration.

Security considerations include:

- No direct public IP address
- Administrative access through Azure Bastion
- Dedicated Corporate Management subnet
- Domain membership for centralized authentication
- Dedicated administrative identity
- Computer object located within the custom `Servers` OU
- Centralized Active Directory, DNS, and Group Policy administration
- Routine administrative workloads separated from the Domain Controller
- Server-specific Group Policy applied through the `Servers` OU

MGMT01 reduces the need to perform routine administrative work directly on DC01.

---

## WEB01 and WEB02

WEB01 and WEB02 provide the redundant internal Nginx application tier.

Security considerations include:

- No direct public IP addresses
- Dedicated Internal Apps subnet
- Subnet-level NSG protection
- Private application delivery through an internal Azure Load Balancer
- Explicit outbound connectivity through Azure NAT Gateway
- Application traffic limited to the required current HTTP service
- Backend health monitoring through the Load Balancer
- Clients access the application through the Load Balancer frontend rather than directly targeting individual backend systems

The web tier remains internal to the Azure network and is not directly exposed to the public Internet.

---

# Virtual Network Segmentation

Network segmentation separates infrastructure according to function.

## Hub Network

The Hub Virtual Network contains shared connectivity and management infrastructure.

Current and reserved infrastructure includes:

- Azure Bastion
- Shared management infrastructure
- Azure Firewall subnet - reserved
- Gateway subnet - reserved

Azure Firewall and gateway services have not yet been deployed.

## Corporate Network

The Corporate Virtual Network contains identity, management, and application workloads.

Network segments include:

- Identity
- Corporate Management
- Private Endpoints
- Internal Applications

The Identity, Corporate Management, and Internal Applications subnets currently host deployed infrastructure.

The Private Endpoints subnet remains reserved for future workloads.

Separating infrastructure by function allows network controls, routing decisions, and future security policies to be applied according to workload requirements.

It also limits the need to place unrelated infrastructure into the same network segment.

---

# Private Subnets

The active Corporate workload subnets are configured as private subnets with default outbound access disabled.

Current private workload subnets include:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

Server virtual machines within these subnets are not assigned direct public IP addresses and do not rely on Azure default outbound access for Internet connectivity.

Required outbound connectivity is instead provided through an explicitly configured Azure NAT Gateway.

This provides a clearer network architecture by separating private workload addressing from outbound Internet connectivity.

---

# Outbound Connectivity

Explicit outbound Internet connectivity for active Corporate workload subnets is provided through Azure NAT Gateway.

| Property | Value |
| --- | --- |
| Name | nat-corp-prd-eus2 |
| Resource Group | rg-connectivity-prd-eus2 |
| Public IP Resource | pip-nat-corp-prd-eus2 |

The NAT Gateway is associated with:

- `snet-identity-prd-eus2`
- `snet-corp-management-prd-eus2`
- `snet-corporate-internal-apps-prd-eus2`

Internet-bound connections originating from these subnets use the NAT Gateway as their explicit outbound path.

Azure NAT Gateway performs Source Network Address Translation (SNAT), translating private source addresses for outbound connections to the dedicated static Public IP resource associated with the NAT Gateway.

This provides a predictable outbound egress resource while allowing DC01, MGMT01, WEB01, and WEB02 to remain privately addressed.

The assigned public IP address itself is intentionally excluded from the public repository.

## NAT Gateway Security Boundary

Azure NAT Gateway provides outbound connectivity and address translation but is not treated as a firewall or traffic-inspection service.

It does not provide application-layer traffic inspection and does not replace controls such as:

- Network Security Groups
- Azure Firewall
- Host-based security controls
- Identity-based authorization
- Application security controls

The NAT Gateway also does not provide unsolicited inbound Internet connectivity to the private server workloads.

Future iterations of the environment may introduce Azure Firewall to provide centralized network filtering, inspection, and policy enforcement.

---

# Internal Application Security

The internal application tier consists of WEB01 and WEB02 behind an internal Azure Load Balancer.

The Load Balancer uses the private frontend address:

`10.1.3.10`

Application traffic currently uses TCP port 80.

The internal Load Balancer provides a stable private application endpoint while keeping the backend servers privately addressed.

Security characteristics of the design include:

- Internal-only Load Balancer frontend
- No public frontend IP for the application
- No direct public IP addresses on WEB01 or WEB02
- Dedicated Internal Apps subnet
- Subnet-level Network Security Group
- Health probing of backend systems
- Clients use the Load Balancer frontend rather than directly targeting backend servers
- Outbound connectivity handled separately through Azure NAT Gateway

The Load Balancer is not treated as a firewall.

Its purpose is application traffic distribution and backend availability monitoring rather than application-layer security inspection.

---

# Active Directory Security

Active Directory Domain Services provides centralized authentication and identity management for the Windows environment.

The domain is:

`corp.mccluretech.com`

Current identity-security design includes:

- Dedicated Domain Controller
- Dedicated Identity subnet
- Active Directory-integrated DNS
- Dedicated domain-joined management server
- Organizational Unit separation
- Security group-based resource authorization
- Group Policy-based server configuration
- Separation of user, administrative, service, server, workstation, group, and disabled objects
- Privileged account details excluded from public documentation

The current OU structure includes:

- Admins
- Disabled Objects
- Groups
- Servers
- Service Accounts
- User Accounts
- Workstations

This structure provides a foundation for applying security policy and administrative controls according to object type and workload function.

---

# Identity and Administrative Separation

The environment separates domain infrastructure from routine administrative workloads.

DC01 hosts Active Directory Domain Services and DNS, while MGMT01 provides a dedicated domain-joined system for administrative activity.

Current design decisions include:

- Domain Controller and management workloads hosted on separate virtual machines
- Dedicated Identity and Management network segments
- Named administrative identity for privileged tasks
- MGMT01 joined to the Active Directory domain
- MGMT01 computer object organized within the custom `Servers` OU
- Active Directory administration tools installed on MGMT01
- DNS administration available from MGMT01
- Group Policy Management available from MGMT01
- Administrative connectivity provided through Azure Bastion
- Privileged account details intentionally excluded from public documentation

Additional privilege separation and delegated administration controls can be introduced as the identity architecture develops.

---

# Group Policy Security

Group Policy is used to centrally apply Windows configuration and security settings to domain-joined systems.

A dedicated Group Policy Object was created:

`GPO-Servers-Baseline`

The GPO is linked to the custom `Servers` OU.

The initial security baseline configures:

`Accounts: Guest account status` → `Disabled`

The policy is configured under:

`Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options`

Creating a dedicated server baseline GPO keeps custom server security settings separate from the Default Domain Policy.

This provides a scalable foundation for adding additional Windows Server security controls without placing workload-specific configuration into Microsoft's default domain policies.

Future baseline improvements can include additional account policies, auditing, security options, Windows Defender settings, firewall configuration, and other server-hardening controls.

---

# Group-Based Authorization

Resource access is assigned through Active Directory security groups rather than directly to individual user accounts.

The initial file-services implementation follows an AGDLP-style authorization model:

**Accounts → Global Group → Domain Local Group → Permission**

Current implementation:

- Test users are assigned to `GG-IT-Users`
- `GG-IT-Users` is nested within `DL-ITShare-RW`
- `DL-ITShare-RW` represents read/write access to the ITShare resource
- `DL-ITShare-RW` receives NTFS Modify permissions on the resource

This separates user-role membership from resource authorization.

If access requirements change, administrators can modify security-group membership rather than repeatedly modifying permissions directly on the resource.

This design also scales more effectively than assigning NTFS permissions directly to individual user accounts.

---

# NTFS Permission Security

The `ITShare` resource hosted on MGMT01 is used to validate group-based Windows resource authorization.

The final NTFS ACL includes:

| Principal | Permission |
| --- | --- |
| DL-ITShare-RW | Modify |
| SYSTEM | Full Control |
| Local Administrators | Full Control |

During validation, the resource was found to inherit an overly broad `Authenticated Users` Modify permission from the parent filesystem structure.

Rather than modifying permissions at the root of the operating-system volume, inheritance was disabled specifically on the ITShare folder.

Existing inherited permissions were first converted to explicit permissions to prevent unintended access loss.

The unnecessary broad access entries were then removed while preserving:

- SYSTEM administrative access
- Local Administrators access
- Explicit access through `DL-ITShare-RW`

This resulted in an intentional resource ACL where normal user access is controlled through the Active Directory security-group model.

The exercise also demonstrated the importance of evaluating both share-level and NTFS permissions when determining effective Windows file access.

---

# Least-Privilege Design

The environment incorporates least-privilege concepts across several layers.

Examples include:

- Server workloads do not receive direct public IP addresses
- Administrative connectivity is centralized rather than exposing management ports publicly
- Identity infrastructure is separated from routine administration
- Application infrastructure is separated from identity and management networks
- Resource access is assigned through security groups rather than directly to users
- Broad inherited NTFS permissions were removed from the ITShare resource
- Custom server security policy is scoped to the `Servers` OU
- Workloads receive only the network services currently required for their function
- Privileged account details and credentials are excluded from public documentation

The lab will continue to expand these controls as additional services are implemented.

---

# Current Security Controls

The following controls and architectural decisions are currently implemented:

- Server virtual machines do not receive direct public IP addresses
- Administrative access is centralized through Azure Bastion when required
- RDP is not exposed directly to the public Internet
- SSH is not exposed directly to the public Internet
- Network Security Groups are applied at the subnet level where appropriate
- Identity, Management, and Internal Application workloads are separated into dedicated subnets
- Active Corporate workload subnets use private subnet configuration
- Default outbound access is disabled for current private server workload subnets
- Explicit outbound Internet connectivity is provided through Azure NAT Gateway
- NAT Gateway uses a dedicated static Public IP resource for outbound SNAT
- Domain Controller and management functions are hosted on separate virtual machines
- Hub and Corporate resources are separated using a Hub-and-Spoke architecture
- Active Directory provides centralized authentication
- Active Directory-integrated DNS provides internal domain name resolution
- Organizational Units separate infrastructure and identity object types
- Dedicated Group Policy is used for server security configuration
- The local Guest account is disabled through the server baseline GPO
- Resource access is assigned through Active Directory security groups
- AGDLP-style group nesting is used for the initial ITShare authorization model
- NTFS permissions are explicitly controlled on the ITShare resource
- Overly broad inherited NTFS permissions were identified and remediated
- WEB01 and WEB02 remain privately addressed
- Internal application traffic is delivered through an internal Azure Load Balancer
- Load Balancer health probing is used to detect unavailable application backends
- Infrastructure resources follow a consistent naming and organizational strategy

---

# Security Validation

Security controls were validated through functional testing rather than documentation alone.

Validation performed within the environment includes:

- Private administrative access to Windows and Linux workloads
- Successful domain authentication from MGMT01
- Active Directory DNS resolution and service discovery
- Domain-joined server placement within the appropriate OU
- Group Policy creation and linkage to the `Servers` OU
- Server baseline security configuration
- Active Directory security-group creation and nesting
- NTFS resource permission assignment through a Domain Local security group
- Review and remediation of inherited filesystem permissions
- Explicit outbound Internet connectivity through Azure NAT Gateway
- NAT Gateway egress validation
- Internal Load Balancer frontend connectivity
- Backend health monitoring
- Application traffic distribution to Nginx backend systems
- Application availability after intentionally shutting down one backend server

The validation process is used to confirm that implemented controls behave as intended rather than relying solely on configuration state.

---

# Future Security Enhancements

Planned security improvements include:

- Azure Firewall
- Microsoft Defender for Cloud
- Azure Policy
- Azure Key Vault
- Just-In-Time VM Access
- Network monitoring and diagnostics
- Centralized logging and monitoring
- Additional identity security controls
- Additional Group Policy server-hardening settings
- More granular administrative privilege delegation
- Expanded role-based security groups
- Windows auditing configuration
- Additional application security controls
- TLS for internal web workloads
- Private Endpoint implementation
- Infrastructure security automation

These services and controls are planned enhancements and should not be interpreted as currently deployed infrastructure.

---

# Security Documentation Approach

This repository documents the architecture, implementation decisions, and security principles of the lab while intentionally excluding sensitive operational information such as:

- Credentials
- Passwords
- Authentication tokens
- Recovery information
- Privileged account details
- Externally reachable IP addresses
- Azure subscription identifiers
- Azure tenant identifiers
- Other secret or uniquely sensitive operational information

Resource names, RFC1918 private addressing, network architecture, fictional domain information, security-group names, and other non-secret implementation details are documented where they provide useful architectural context.

This approach provides enough technical detail to demonstrate the implementation without publishing credentials or sensitive cloud-account information.

---

# Lessons Learned

## Private Subnets and Explicit Outbound Connectivity

The Identity and Corporate Management subnets were initially configured without default outbound Internet access.

During early deployment, Internet connectivity was temporarily enabled to support operating-system updates, role installation, and configuration tasks.

Rather than retaining implicit default outbound connectivity, an Azure NAT Gateway was later deployed to provide an explicitly defined outbound path.

After the NAT Gateway was associated with the required workload subnets, private subnet configuration was restored and default outbound access was disabled.

The Internal Apps subnet was subsequently associated with the same NAT Gateway to provide WEB01 and WEB02 with explicit outbound connectivity while preserving private workload addressing.

Outbound connectivity was then validated from private workloads.

This demonstrated that private Azure workloads can maintain required outbound Internet connectivity without direct public IP assignments or reliance on Azure default outbound access.

---

## Security Role of NAT Gateway

Implementing NAT Gateway improved the outbound network architecture by providing a predictable and explicitly configured egress path.

The implementation also reinforced the distinction between address translation and network security enforcement.

NAT Gateway performs outbound SNAT but does not provide application-layer traffic inspection or replace a network firewall.

Future iterations of the environment may introduce Azure Firewall to provide centralized traffic filtering, inspection, and policy enforcement.

---

## Share Permissions vs. NTFS Permissions

Implementing the ITShare resource reinforced the distinction between Windows share permissions and NTFS filesystem permissions.

Share-level access and NTFS access are evaluated together when a resource is accessed across the network.

During validation, the share configuration was reviewed separately from the underlying NTFS ACL.

The NTFS ACL revealed that `Authenticated Users` inherited Modify access from the parent filesystem structure, which would have allowed broader access than intended by the Active Directory group design.

This highlighted the importance of validating effective authorization rather than assuming that creating a security group and assigning it to a resource automatically establishes least-privilege access.

The inherited access was remediated at the ITShare folder rather than modifying permissions on the root operating-system volume.

---

## Group-Based Resource Authorization

The ITShare implementation demonstrated the operational benefit of separating identities from resource permissions.

Rather than assigning NTFS access directly to individual users, the environment uses:

**User → Global Group → Domain Local Group → Resource Permission**

This provides a more scalable authorization model because user-role membership and resource permissions can be managed independently.

The implementation also provided practical experience with Active Directory group scope, nested group membership, and Windows filesystem authorization.

---

## Group Policy Separation

Creating `GPO-Servers-Baseline` reinforced the value of keeping workload-specific configuration separate from default domain policies.

The server baseline is linked specifically to the `Servers` OU, allowing server security configuration to evolve independently from other Active Directory objects.

The initial configuration is intentionally limited, providing a controlled foundation for introducing additional server-hardening settings over time.

---

## Internal Application Resiliency

Deploying WEB01 and WEB02 behind an internal Azure Load Balancer demonstrated how application availability can be improved without publicly exposing backend systems.

Health probing allows the Load Balancer to determine whether backend instances are available to receive traffic.

Failover testing was performed by intentionally shutting down WEB01 and confirming that the same Load Balancer frontend continued serving the application through WEB02.

The exercise reinforced that high availability, network security, and outbound connectivity are separate architectural concerns:

- The Load Balancer distributes application traffic.
- The health probe evaluates backend availability.
- The Network Security Group controls network traffic.
- The NAT Gateway provides outbound Internet connectivity.
- Private addressing prevents direct public exposure of the backend servers.

---

# Summary

The Azure Enterprise Lab currently implements layered security controls across networking, identity, administration, Windows policy, resource authorization, and application delivery.

The environment uses Hub-and-Spoke network segmentation, private server addressing, Azure Bastion, subnet-level Network Security Groups, Azure NAT Gateway, Active Directory Domain Services, Active Directory-integrated DNS, dedicated Organizational Units, Group Policy, AGDLP-style security-group nesting, controlled NTFS permissions, and an internal Azure Load Balancer.

Security controls are validated through hands-on testing, including domain authentication, DNS resolution, Group Policy configuration, NTFS permission review and remediation, NAT Gateway egress validation, and internal application failover testing.

The current implementation provides a foundation for future Azure Firewall, Microsoft Defender for Cloud, Azure Policy, Key Vault, centralized monitoring, expanded Windows hardening, and additional identity-security controls.
