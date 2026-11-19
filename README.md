# Microsoft Entra ID Labs

Hands-on identity labs built around a hybrid environment: on-premises Active Directory synchronized to Microsoft Entra ID. Each scenario starts from a concrete problem, shows the implementation step by step, and ends with a test that proves the result.

> All examples come from a personal lab environment. No production data, credentials or secrets are included. Tenant identifiers and other sensitive values in screenshots are redacted.

## Scenarios

1. [Hybrid sync with Entra Connect and filtering](scenarios/hybrid-identity-entra-connect/)
2. [SSPR with password writeback and password protection](scenarios/02-sspr-writeback/)
3. [Conditional Access: MFA and geo-blocking](scenarios/03-conditional-access-geo-blocking/)

Each scenario folder contains its own README with the goal, prerequisites, steps, and the final result.

## Lab architecture

**On-premises**

| Component | Details |
|-----------|---------|
| Domain controller | Windows Server 2019, AD DS and DNS, domain `kam.iam` |
| Entra Connect server | Windows Server 2022, domain-joined, dedicated to Microsoft Entra Connect Sync |
| Client | Windows 11, domain-joined |

**Cloud**

| Component | Details |
|-----------|---------|
| Identity platform | Microsoft Entra ID tenant |
| Synchronization | Microsoft Entra Connect Sync |
| Authentication method | Password Hash Synchronization (PHS) |

Entra Connect runs on a separate member server rather than on the domain controller.

## Technologies

Active Directory Domain Services, Microsoft Entra ID, Microsoft Entra Connect Sync, Conditional Access, Self-Service Password Reset, Windows Server 2019 and 2022, Windows 11, PowerShell, Git and GitHub.

## Repository structure

```
entra-id-labs/
├── README.md
├── scenarios/
│   ├── 01-entra-connect-install-and-basic-sync/
│   ├── 02-sspr-writeback/
│   └── 03-conditional-access-geo-blocking/
└── scripts/
```
