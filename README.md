# Hybrid Identity Lab – Microsoft Entra ID

## Overview

This repository documents a hands-on hybrid identity lab focused on integrating an existing on-premises Active Directory environment with Microsoft Entra ID.

The project explores hybrid identity architecture, directory synchronization, authentication methods, identity security, and cloud identity administration.

The objective is to develop and demonstrate practical skills in Microsoft Entra ID and hybrid identity management through documented implementation scenarios, configuration examples, automation scripts, and technical observations.

## Architecture

The lab consists of an existing on-premises Active Directory environment, a dedicated synchronization server, and a Microsoft Entra ID tenant.

### On-premises environment

**Active Directory Domain Controller**

- **Operating system:** Windows Server 2019
- **Directory service:** Active Directory Domain Services (AD DS)
- **Domain:** `kam.iam`
- **Role:** Source directory for hybrid identity synchronization
- **Services:** Active Directory and DNS

**Microsoft Entra Connect Server**

- **Operating system:** Windows Server 2022
- **Domain membership:** Joined to `kam.iam`
- **Role:** Dedicated directory synchronization server
- **Synchronization software:** Microsoft Entra Connect Sync

Microsoft Entra Connect Sync is installed on a separate domain-joined Windows Server 2022 machine rather than directly on the domain controller.

### Cloud environment

- **Identity platform:** Microsoft Entra ID
- **Synchronization:** Microsoft Entra Connect Sync
- **Initial authentication method:** Password Hash Synchronization (PHS)

The authentication configuration may be extended to explore alternative hybrid authentication and single sign-on approaches.

## Project Scope

The topics listed below represent the overall scope of the project and do not imply a fixed implementation sequence or completed configuration.

### 1. Hybrid Identity and Directory Synchronization

- Integrating an existing on-premises Active Directory with Microsoft Entra ID.
- Installing and configuring Microsoft Entra Connect Sync.
- Configuring and validating directory synchronization.
- Understanding object synchronization, attributes, and identity matching.
- Exploring synchronization rules, transformations, and attribute flows.
- Implementing synchronization filtering and exclusion scenarios.
- Troubleshooting synchronization and connector-related issues.
- Understanding the synchronization engine and its components.

### 2. Authentication and Single Sign-On

- Password Hash Synchronization (PHS).
- Pass-through Authentication (PTA).
- Federation with Active Directory Federation Services (AD FS).
- Microsoft Entra Seamless Single Sign-On.
- Understanding Kerberos-based authentication scenarios.
- Exploring hybrid authentication behavior and troubleshooting.

### 3. Microsoft Entra ID Administration

- Managing users, groups, and administrative roles.
- Managing enterprise applications and app registrations.
- Configuring application authentication and permissions.
- Exploring Microsoft Graph API and Microsoft Graph PowerShell.
- Reviewing directory objects and their properties.
- Understanding cloud and synchronized identity objects.

### 4. Identity Security

- Multi-factor authentication (MFA).
- Conditional Access policies.
- Microsoft Entra ID Protection.
- Privileged Identity Management (PIM).
- Emergency access accounts.
- Sign-in logs and identity-related monitoring.
- Authentication methods and security controls.
- Risk-based identity protection scenarios.

### 5. Automation and Administration

- PowerShell automation.
- Microsoft Graph API integration.
- User and group lifecycle management.
- Administrative scripts and reporting.
- Identity management and configuration validation.
- Automating repetitive identity administration tasks.

## Technologies

- Microsoft Active Directory Domain Services
- Microsoft Entra ID
- Microsoft Entra Connect Sync
- Microsoft Graph API
- Microsoft Graph PowerShell SDK
- PowerShell
- Windows Server 2019
- Windows Server 2022
- Git and GitHub

## Repository Structure

```text
entra-id-hybrid-lab/
├── README.md
├── .gitignore
├── scenarios/
│   └── hybrid-identity-entra-connect/
│       └── README.md
├── scripts/
└── documentation/
    └── architecture/
```

Additional scenarios, scripts, and technical documentation will be added as the lab evolves.

## Project Status

The on-premises Active Directory environment is already available.

A dedicated Windows Server 2022 machine is domain-joined and hosts Microsoft Entra Connect Sync.

The hybrid identity integration and subsequent Entra ID scenarios are being developed and documented incrementally.

Only implemented and validated configurations will be marked as completed.

## Disclaimer

This repository is an independent technical learning project.

All examples are intended for a lab environment. No production credentials, secrets, or confidential organizational information should be committed to this repository.
