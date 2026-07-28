# Azure Home Lab — Capstone Overview

## Summary
I built a small Azure lab simulating a company's IT environment — 
covering user onboarding, access control, basic networking and 
security, and monitoring — so I could practice the kind of 
practical decision-making that comes up in a Help Desk role.

## Environment Built
- Resource group organizing all lab resources
- 2 simulated users (Microsoft Entra ID)
- 1 security group with scoped RBAC (resource + data-level roles)
- 1 storage account with blob container
- 1 virtual network + NSG with inbound security rules
- 1 monitoring alert rule (email notifications on resource changes)

## Skills Demonstrated
- Identity & access management (least privilege, group-based RBAC)
- Cloud resource provisioning (storage, networking)
- Network security fundamentals (NSG rules, priority logic)
- Troubleshooting (rule priority misdiagnosis, RBAC propagation delay)
- Monitoring & alerting
- Technical documentation (ticket-style, ongoing throughout)

## Ticket Index
- #001 — Environment setup
- #002 — User creation
- #003 — Security group + assignment
- #004 — RBAC role assignment + verification
- #005 — Second user onboarding (scaling test)
- #006 — Storage account deployment
- #007 — Data-level access control (storage RBAC)
- #008 — NSG inbound rule (deny RDP)
- #009 — Troubleshooting: NSG priority logic
- #010 — Troubleshooting: RBAC propagation delay
- #011 — Monitoring & alert configuration

## What I'd Improve With More Time
I'd deploy an actual Azure VM to test NSG rule enforcement directly, rather than only configuring rules and reading Azure's warnings. My local machine has RAM constraints for heavy local virtualization, but Azure VMs run cloud-side. This is a next step I'd pursue with a slightly larger compute budget.