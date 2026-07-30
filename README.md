# Azure Home Lab: Identity, Access & Cloud Administration

A self-directed Azure lab simulating a small company's IT environment, covering identity management, role-based access control, cloud storage, network security, and monitoring, built to demonstrate practical Help Desk / IT Support readiness.

## Project Overview

I built a small Azure environment simulating the day-to-day responsibilities of an entry-level IT/Help Desk technician:

- **Identity management**: provisioned users in Microsoft Entra ID and managed access through security groups
- **Access control**: applied role-based access control (RBAC) at both the resource and data level, following least-privilege principles
- **Cloud storage**: deployed and configured a storage account with access controls
- **Network security**: configured Network Security Group (NSG) rules and diagnosed rule-priority behavior
- **Monitoring**: set up alerting for administrative changes to the environment

Every task was logged as a support ticket, the same format used to document real Help Desk and Sysadmin work, rather than just following a tutorial checklist.

## Business Scenario

Most entry-level IT roles involve the same recurring responsibilities: onboarding new employees, managing who has access to what, keeping shared resources available and secure, and knowing when something in the environment changes. This lab simulates a small company (`rg-homelab-01`) onboarding two employees onto a shared IT environment, provisioning their access through groups rather than individual permissions, and setting up the storage, network security, and monitoring a real Help Desk tech would be responsible for maintaining.

## Skills Demonstrated

- Microsoft Entra ID user provisioning and least-privilege account creation
- Group-based access management (security groups instead of direct permission assignment)
- Azure RBAC at both the resource level (Reader) and data level (Storage Blob Data Reader)
- Azure Storage account deployment and configuration
- Network Security Group (NSG) rule creation and priority-based rule evaluation
- Troubleshooting real Azure behaviors: NSG rule priority logic and RBAC propagation delay
- Azure Monitor alert configuration for administrative change tracking
- Cost governance (budget alerts configured before provisioning any resources)
- Ticket-style technical documentation

## Architecture

```
Azure Subscription (Pay As You Go)
│
└── Resource Group: rg-homelab-01
    │
    ├── Microsoft Entra ID
    │   ├── User: John Doe (jdoe@edawk10gmail.onmicrosoft.com)
    │   ├── User: Jane Smith (jsmith)
    │   └── Security Group: Help-Desk-Team
    │       ├── Members: John Doe, Jane Smith
    │       ├── RBAC: Reader (scope: rg-homelab-01)
    │       └── RBAC: Storage Blob Data Reader (scope: homelabstorage01)
    │
    ├── Storage Account: homelabstorage01
    │   ├── Performance: Standard, Replication: LRS
    │   ├── Blob anonymous access: Disabled
    │   └── Container: test-files
    │
    ├── Network Security Group: nsg-homelab-01
    │   ├── Deny-RDP-Inbound (priority 100, TCP 3389)
    │   └── Deny-All-Inbound-Baseline (priority 4096, all ports)
    │
    └── Monitor Alert: Resource-Change-Alert
        └── Action Group: homelab-alerts (email notifications)
```

**Design decisions:**
- **Group-based access, not direct permissions.** Both users gain access exclusively through `Help-Desk-Team` membership, so onboarding a new employee (Ticket #005) took one step: add to the existing group.
- **Two-tier RBAC.** Reader is scoped at the resource group level (can see resources exist), while Storage Blob Data Reader is scoped separately at the storage account level (can actually read blob data). Azure treats management-plane and data-plane access as distinct permission boundaries, and this lab replicates that separation deliberately.
- **LRS over geo-redundant storage.** Locally-redundant storage is sufficient for lab data; geo-replication is a cost/reliability tradeoff appropriate for production data, not test data.

## Implementation Summary

### 1. Environment Setup
Created Azure subscription and budget alert ($5/mo at 80% threshold), and resource group `rg-homelab-01` before provisioning any resources.

<img width="550" alt="Resource group created in Azure" src="https://github.com/user-attachments/assets/febfb664-2378-4b9c-9f91-e1d0c83ec99d" />

<br>

### 2. User Provisioning
Created user John Doe in Microsoft Entra ID with no default group or role membership, following least-privilege principles: access is granted deliberately, not automatically at account creation.

<img width="600" alt="User created in Microsoft Entra ID" src="https://github.com/user-attachments/assets/a203cea1-ba3d-4db1-968f-c8865aa0bdf0" />

<br>

### 3. Group-Based Access Management
Created security group `Help-Desk-Team` and added John Doe as a member, rather than assigning permissions to his account directly.

<img width="600" alt="Help-Desk-Team group membership" src="https://github.com/user-attachments/assets/291b7001-9f97-4913-9a35-a1ab09af2d7d" />

<br>

### 4. RBAC Role Assignment
Assigned the **Reader** role to `Help-Desk-Team`, scoped to the resource group (not the subscription), and verified John Doe inherits access through group membership via Access Control (IAM) → Check Access.

<img width="450" alt="Reader role assigned to Help-Desk-Team" src="https://github.com/user-attachments/assets/50c8fd40-3a5f-4e23-9ccb-a4f7f3fd70f7" />
<img width="450" alt="Check Access verification for inherited role" src="https://github.com/user-attachments/assets/7161d91c-39bd-48a5-acf0-045a7407a673" />

<br>

### 5. Scaling the Access Model
Onboarded a second user, Jane Smith, and added her to the existing `Help-Desk-Team` group, confirming the group-based model scales: new hires get consistent access in one step.

<img width="600" alt="Second user added to existing group" src="https://github.com/user-attachments/assets/e90a7c0a-b477-405d-9368-d8392aeaed52" />

<br>

### 6. Storage Account Deployment
Deployed storage account `homelabstorage01` (Standard performance, LRS replication, anonymous blob access disabled, secure transfer enabled) and confirmed file upload to a private blob container.

<img width="650" alt="File uploaded to private blob container" src="https://github.com/user-attachments/assets/43418c1a-9f0f-48fb-88e4-d7b720237566" />

<br>

### 7. Data-Level Access Control
Assigned **Storage Blob Data Reader** to `Help-Desk-Team`, scoped to the storage account, demonstrating the distinction between resource-level visibility and data-level access.

<img width="650" alt="Storage Blob Data Reader role assignment" src="https://github.com/user-attachments/assets/f78bb9e4-0613-453c-9419-3212176293e5" />

<br>

### 8. Network Security Configuration
Created NSG `nsg-homelab-01` with an inbound rule denying RDP (port 3389) from any source, a baseline defense against the constant internet-wide scanning for open RDP ports.
> See [Troubleshooting Simulations](#troubleshooting-simulations) for the rule-priority scenario this setup enabled.

<img width="600" alt="NSG deny-RDP rule with scope warnings" src="https://github.com/user-attachments/assets/260c4cc6-f4d1-4851-a8a9-bd7cd2d1e14c" />
<img width="280" alt="NSG warning detail" src="https://github.com/user-attachments/assets/558bee57-8f8a-4f29-84c6-e8070cc4d71e" />

<br>

### 9. Monitoring & Alerting
Configured an alert rule to notify by email whenever an administrative change occurs in `rg-homelab-01`, rather than manually checking activity logs.

<img width="650" alt="Active alert rule configured" src="https://github.com/user-attachments/assets/03885957-f8e3-4587-bd90-d86fe980abeb" />


## Troubleshooting Simulations

Rather than only configuring resources, I deliberately constructed two realistic failure scenarios to practice diagnosis, the same way a Help Desk tech has to reason through a ticket without knowing the cause in advance.

### Simulation 1: NSG rule priority misconception

- **Scenario:** Simulated a "VM completely unreachable, no RDP, no ping" ticket to test diagnosis against the NSG rules already in place.
- **Investigation:** Reviewed the NSG's inbound rules: `Deny-RDP-Inbound` (priority 100, TCP port 3389 only) and `Deny-All-Inbound-Baseline` (priority 4096, all ports/protocols). Initially assumed the more restrictive rule would take precedence.
- **Root cause correction:** Azure evaluates NSG rules by priority **number**, lowest number first, not by how restrictive the rule is. Since ping uses ICMP rather than port 3389, the RDP-specific rule couldn't be responsible for blocking ping. The all-ports `Deny-All-Inbound-Baseline` rule was the actual cause of total connectivity loss.
- **Result:** Correctly diagnosed the root cause by reasoning through rule logic, and corrected a real misconception I had about how rule priority works.

### Simulation 2: RBAC "Authorization Permission Mismatch" traced to propagation delay

- **Scenario:** Simulated a second user (Jane Smith) reporting an "Authorization Permission Mismatch" error accessing a storage container, despite having identical group membership and role assignment to a user (John Doe) who could access it fine.
- **Investigation:** Confirmed Jane's group membership and role assignment matched John's exactly. Considered that Azure RBAC role assignments are not always instant, propagation can take several minutes even though the Azure Portal shows the change as saved immediately.
- **Root cause:** RBAC propagation delay, not a misconfiguration.
- **Fix:** No configuration change made. Waited several minutes and retested access.
- **Result:** Learned to rule out propagation delay before assuming misconfiguration, avoiding unnecessary troubleshooting or unneeded permission changes on a working configuration.
## Lessons Learned

- Group-based access control isn't just cleaner, it's the difference between onboarding a new employee in one step versus reconfiguring permissions from scratch every time.
- Azure separates management-plane access from data-plane access deliberately. Being able to see that a resource exists and being able to read the data inside it are two different permission boundaries, and real environments should treat them that way.
- NSG rule evaluation is priority-number-based, not restrictiveness-based. This is a common misconception, and getting it wrong during a real incident would mean troubleshooting the wrong rule entirely.
- Not every access issue is a misconfiguration. Azure RBAC changes can take several minutes to propagate, and checking timing first prevents unnecessary changes to a configuration that was already correct.
- Cost governance (budget alerts) belongs at the start of a cloud environment's setup, not as an afterthought.

## Tech Stack

Microsoft Azure · Microsoft Entra ID · Azure RBAC · Azure Storage · Virtual Networks & NSGs · Azure Monitor


---

*Lab built independently in a personal Azure subscription. All ticket numbers referenced above correspond to the detailed ticket-style documentation completed during the build.*
