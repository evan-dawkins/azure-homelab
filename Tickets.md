
# Ticket #001 — Azure Environment Setup

## What I did
Created Azure account (Pay As You Go), GitHub account, budget alert 
($5/mo, 80% actual spend), and resource group `rg-homelab-01`.

## Why it matters
Every Azure resource needs a container (resource group) and cost 
controls before real work starts. This mirrors how a company sets 
up a new cloud environment.

## Screenshot
![[01-resource-group.png]]

## Problem encountered
Budget alert form rejected email initially — "unique email required" 
error. Fixed by re-entering email in the Alert recipients field 
directly. Nothing too serious

## What I'd tell an interviewer
"I set up my lab environment with cost governance from day one — 
budget alerts before touching any resources — same as I'd expect 
in a real company subscription."




# Ticket #002 — Create New User (Onboarding Simulation)

## Ticket Summary
Simulate onboarding a new employee into Microsoft Entra ID.

## User/Business Impact
New hires need an account before they can access any company
resources. This is typically a Day 1 Help Desk task.

## Investigation
N/A — straightforward provisioning task.

## Solution
Created new user in Microsoft Entra ID:
- UPN: jdoe@edawk10gmail.onmicrosoft.com
- Display name: John Doe
- User type: Member
- Account enabled: Yes
- No groups or roles assigned at creation (least privilege — access is granted separately, not by default).

## Verification
![[02-user-created.png]]

## Documentation
**Temporary password (for lab purposes only — never commit real credentials to GitHub):**
`Vuxa755539`

## Interview Explanation
"I created a new user with no default group or role membership, following least-privilege principles. Access should be granted deliberately through groups or assigned roles rather than automatically at account creation."

```
```
# Ticket #003 — Create Security Group & Assign User

## Ticket Summary
Create a security group and add the new user instead of granting permissions directly to the individual account.

## User/Business Impact
Managing access through groups is far more scalable. When multiple employees require the same permissions, administrators simply add or remove users from a group rather than managing permissions on each account individually.

## Investigation
N/A — standard provisioning task.

## Solution
Created the security group `Help-Desk-Team` in Microsoft Entra ID.
Added John Doe as a member of the group.

## Verification
![[03-group-membership.png]]

## Documentation
Used a Microsoft Entra security group to demonstrate role-based administration and group-based access management.

## Interview Explanation
"I used group-based access instead of assigning permissions directly to the user. This is how real organizations manage permissions because it's easier to scale, audit, and maintain. When someone joins or leaves a team, you only update their group membership instead of modifying permissions on every individual account."


# Ticket #004 — Assign RBAC Role to Group

## Ticket Summary
Grant the Help-Desk-Team group Reader access on the resource group,
instead of granting permissions to John Doe directly.

## User/Business Impact
New employees need read access to see the environment before they're
trusted with change/delete permissions. Reader is the safe default
starting point.

## Investigation
N/A — standard provisioning task.

## Solution
Assigned "Reader" role to `Help-Desk-Team` group, scoped to 
`rg-homelab-01`, via Access control (IAM).

## Verification
![[04a-rbac-role-assignment.png]]
![[04b-check-access-verification.png]]
Confirmed John Doe inherits Reader role via Help-Desk-Team membership — access control chain works end to end.
## Documentation
Role: Reader
Scope: rg-homelab-01 (resource group level, not subscription-wide)

## Interview Explanation
"I scoped the Reader role to the resource group, not the whole 
subscription — least privilege means limiting both what someone 
can do AND where they can do it." 
# Ticket #005 — Add Second User to Existing Group

## Ticket Summary
Onboard a second employee (Jane Smith) and add to Help-Desk-Team, 
reusing the existing group instead of reconfiguring permissions.

## User/Business Impact
Proves the group-based model scales — new hires get instant, 
consistent access by joining an existing group.

## Solution
Created user Jane Smith (jsmith). Added to Help-Desk-Team group.

## Verification
![[05-group-two-members.png]]

# Ticket #006 — Deploy Storage Account

## Ticket Summary
Create a storage account to hold blob data, simulating a company 
file/data storage need.

## User/Business Impact
Companies use storage accounts for backups, app data, and shared 
files. Common Help Desk tickets: "can't access shared storage," 
"backup failed," "storage running out."

## Investigation
N/A — provisioning task.

## Solution
Created storage account `homelabstorage01` in `rg-homelab-01`.
- Performance: Standard
- Replication: LRS (locally-redundant — sufficient for lab data, 
  avoids unnecessary cost of geo-replication)
- Blob anonymous access: Disabled
- Secure transfer: Enabled
- Access tier: Hot

## Verification
Confirmed file upload to private blob container "test-files" — storage account functioning as expected.

![[06-blob-upload.png|670]]

## Documentation
Primary service: Blob storage
Replication choice reasoning: LRS vs RA-GRS — geo-redundancy is for 
production data that can't be lost; unnecessary cost for a lab.

## Interview Explanation
"I chose LRS over geo-redundant storage because this is a lab 
environment — geo-replication is a cost/reliability tradeoff you 
make for production data, not test data. Matching redundancy level 
to actual business need is part of cost-conscious cloud admin."

## Documentation
Temp password: Duku183387


## Interview Explanation
"Onboarding a second employee took one step — add to existing 
group — instead of reconfiguring permissions from scratch. That's 
the payoff of setting up groups correctly the first time."

# Ticket #007 — Data-Level Access Control (Storage RBAC)

## Ticket Summary
Assign a data-specific role (Storage Blob Data Reader) to 
Help-Desk-Team, distinct from the resource-level Reader role 
assigned earlier.

## User/Business Impact
Seeing that a storage account exists is not the same as being 
allowed to read the data inside it. Companies separate these two 
permission levels deliberately.

## Solution
Assigned "Storage Blob Data Reader" role to Help-Desk-Team, scoped 
to homelabstorage01 storage account.

## Verification
![[07-storage-rbac.png]]

## Documentation
Two separate roles now govern this resource:
- Reader (resource group level) — can see the storage account exists
- Storage Blob Data Reader (storage account level) — can read data inside

## Interview Explanation
"Azure separates management-plane access from data-plane access. 
Someone can see a storage account exists without being able to 
read what's inside it — that's a deliberate security boundary I 
replicated in my lab."

# Ticket #008 — NSG Inbound Rule (Deny RDP)

## Ticket Summary
Create a Network Security Group with a rule denying inbound RDP 
(port 3389) traffic from any source.

## User/Business Impact
RDP left open to the internet is a common attack vector — bots 
constantly scan for open port 3389. Explicitly denying it is a 
baseline security practice.

## Investigation
Rule creation triggered two Azure warnings: the "Any" source is 
broad enough to also affect AzureLoadBalancer traffic and internal 
VirtualNetwork-to-VirtualNetwork traffic, not just external RDP 
attempts.

## Solution
Created NSG `nsg-homelab-01` with inbound rule "Deny-RDP-Inbound":
Deny, TCP, port 3389, source Any, priority 100.

## Verification![[08-nsg-rule-warnings.png]]
![[08b-nsg-warnings-detail.png|402]]

NSG not yet attached to a subnet/NIC — rule is configured but not 
actively enforced (expected, no VM/subnet association yet).

## Documentation
Warnings understood as scope-broadness notices, not errors. In a 
live environment, this rule would need to be paired with a higher-
priority Allow rule for VirtualNetwork-to-VirtualNetwork traffic 
to avoid unintended internal disruption.

## Interview Explanation
"I created a deny rule for RDP and Azure flagged that my 'Any 
source' scope was broad enough to also affect internal load 
balancer and VNet traffic. That's the kind of warning you need to 
read carefully — a rule that's technically correct for its 
intended purpose can still have side effects if the scope is too 
wide."

# Ticket #009 — Troubleshoot Total Connectivity Loss (NSG Priority)

## Ticket Summary
Simulated user report: new VM unreachable — no RDP, no ping. 
Diagnose using NSG rule priority, no changes made yet.

## User/Business Impact
Total connectivity loss blocks all work on a VM — high-urgency 
ticket in a real environment. Fast, correct diagnosis matters.

## Investigation
Reviewed Inbound security rules on nsg-homelab-01:
- Deny-RDP-Inbound — priority 100, TCP port 3389 only
- Deny-All-Inbound-Baseline — priority 4096, all ports/protocols

Initially assumed "more restrictive rule wins" — corrected: Azure 
evaluates by priority NUMBER, lowest number = evaluated first, not 
by how restrictive the rule is.

Ping uses ICMP, not port 3389 — so Deny-RDP-Inbound rule can't be 
the cause of ping failing. Deny-All-Inbound-Baseline (all ports/
protocols) is the rule responsible for blocking all traffic types.

## Solution
Root cause identified: Deny-All-Inbound-Baseline rule blocks all 
inbound traffic, including protocols/ports unrelated to RDP. This 
is expected behavior for a default-deny baseline rule — not a 
misconfiguration, but something that needs a matching Allow rule 
for any traffic that should actually get through.

## Verification
![[09-nsg-priority-troubleshoot.png]]

## Documentation
Priority number = evaluation order (lower number = evaluated 
first), NOT restrictiveness. Common misconception worth 
remembering.

## Interview Explanation
"I diagnosed a simulated total-connectivity-loss ticket by reading 
NSG rule priorities instead of guessing. I initially assumed the 
more restrictive rule would win, but Azure actually evaluates by 
priority number, lowest first. Ping failing pointed me to the 
all-ports deny rule specifically, since RDP-only rules can't affect 
ICMP traffic."

# Ticket #010 — Troubleshoot Delayed RBAC Propagation

## Ticket Summary
Simulated user report: Jane Smith added to Help-Desk-Team, same as 
John Doe, but gets "Authorization Permission Mismatch" error 
accessing the same storage container John can access fine.

## User/Business Impact
Common real-world Help Desk call — "I just got access and it's not 
working" is often mistaken for misconfiguration when it's actually 
normal propagation delay.

## Investigation
Group membership and role assignment for Jane confirmed identical 
to John's on paper. Considered: RBAC role assignments in Azure are 
not instant — propagation can take several minutes after a group 
membership change, even though the portal shows it as saved right 
away.

## Solution
No configuration change needed. Root cause: RBAC propagation delay, 
not a misconfiguration. Correct action: wait several minutes, then 
retest access.

## Verification
[if retested: screenshot access working after delay]

## Documentation
RBAC changes are not instant. Before assuming misconfiguration, 
rule out propagation delay first — avoids unnecessary 
troubleshooting or unnecessary permission changes.

## Interview Explanation
"Not every access issue is a misconfiguration. I've learned to 
check timing first — Azure RBAC changes can take a few minutes to 
propagate, so a 'just added, not working yet' report often 
resolves itself. Jumping straight to reconfiguring access without 
ruling out delay first can create unnecessary changes or confusion."

# Ticket #011 — Configure Resource Change Alert

## Ticket Summary
Set up an alert rule to email notification when administrative 
changes occur in rg-homelab-01.

## User/Business Impact
Change awareness is a core admin responsibility — knowing when 
something in the environment changes, without manually checking 
logs constantly.

## Solution
Created alert rule "Resource-Change-Alert" scoped to rg-homelab-01, 
condition: All Administrative operations, action group 
"homelab-alerts" (email notification).

## Verification
![[11-alert-rule-active.png]]

## Documentation
Also noted: Entra ID sign-in logs only show actual login events, 
not just user existence — John Doe shows no sign-in history since 
the account was never logged into, separate from being provisioned.

## Interview Explanation
"I set up an alert so I'm notified by email anytime something 
changes in my resource group, instead of manually checking logs. 
I also learned sign-in logs and user provisioning are separate — 
an account existing doesn't mean it's been used."