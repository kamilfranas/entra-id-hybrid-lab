# Hybrid Identity Lab – Microsoft Entra ID

## Overview

This repository documents a hands-on hybrid identity lab focused on integrating an existing on-premises Active Directory environment with Microsoft Entra ID.

The project explores hybrid identity architecture, directory synchronization, authentication methods, identity security, and cloud identity administration.

The goal is to demonstrate practical knowledge of Microsoft Entra ID and hybrid identity management through documented scenarios, configuration examples, scripts, and technical observations.

## Architecture

The lab is based on an existing on-premises Active Directory environment connected to Microsoft Entra ID.

### On-premises environment

- **Operating system:** Windows Server 2019
- **Directory service:** Active Directory Domain Services (AD DS)
- **Domain:** `kam.iam`
- **Role:** Source directory for hybrid identity synchronization

### Cloud environment

- **Identity platform:** Microsoft Entra ID
- **Synchronization:** Microsoft Entra Connect Sync
- **Initial authentication method:** Password Hash Synchronization (PHS)

The authentication configuration may be extended to explore alternative hybrid authentication and single sign-on approaches.

## Project Scope

The topics listed below represent the overall scope of the project and do not imply a fixed implementation sequence.

### 1. Hybrid Identity and Directory Synchronization

- Integrating an existing on-premises Active Directory with Microsoft Entra ID.
- Configuring and validating directory synchronization.
- Understanding object synchronization, attributes, and identity matching.
- Exploring synchronization behavior and troubleshooting.

### 2. Authentication and Single Sign-On

- Password Hash Synchronization (PHS).
- Pass-through Authentication (PTA).
- Federation with Active Directory Federation Services (AD FS).
- Seamless Single Sign-On.
- Understanding Kerberos-related authentication scenarios.

### 3. Microsoft Entra ID Administration

- Managing users, groups, and administrative roles.
- Managing enterprise applications and app registrations.
- Exploring Microsoft Graph API and Microsoft Graph PowerShell.
- Reviewing directory objects and their properties.

### 4. Identity Security

- Multi-factor authentication (MFA).
- Conditional Access policies.
- Microsoft Entra ID Protection.
- Privileged Identity Management (PIM).
- Emergency access accounts.
- Sign-in logs and identity-related monitoring.

### 5. Automation and Administration

- PowerShell automation.
- Microsoft Graph API integration.
- Administrative scripts and reporting.
- Identity management and configuration validation.

## Technologies

- Microsoft Active Directory Domain Services
- Microsoft Entra ID
- Microsoft Entra Connect Sync
- Microsoft Graph API
- Microsoft Graph PowerShell SDK
- PowerShell
- Windows Server 2019
- Git and GitHub

## Repository Structure

```text
entra-id-hybrid-lab/
├── README.md
├── .gitignore
├── scenarios/
├── scripts/
└── documentation/
    └── architecture/
```

## Project Status

The on-premises Active Directory environment is already available.

The hybrid identity integration and subsequent Entra ID scenarios are being developed and documented incrementally.

Only implemented and validated configurations will be marked as completed.

## Disclaimer

This repository is an independent technical learning project.

All examples are intended for a lab environment. No production credentials, secrets, or confidential organizational information should be committed to this repository.