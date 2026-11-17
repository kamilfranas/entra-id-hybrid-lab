# Scenario 3: MFA and Conditional Access Basics

## Goal

Secure the Finance Department in Microsoft Entra ID by implementing multi-factor authentication (MFA) using Microsoft Authenticator, requiring compliant devices, and restricting access to trusted locations (Poland).

## Scenario description

In this scenario, you will implement basic identity and access security controls for the Finance Department in a hybrid identity environment.

First, you will create a dynamic security group in Microsoft Entra ID based on the synchronized department attribute from on-premises Active Directory. This will allow users to be automatically added to the group based on their department assignment.

Next, you will configure Microsoft Authenticator as the required MFA method and create Conditional Access policies to enforce additional access restrictions.

The policies will require users from the Finance Department to:

- Complete MFA using Microsoft Authenticator.
- Access corporate resources from compliant devices.
- Sign in only from trusted locations, with Poland configured as an allowed location.

Finally, you will test the configuration using a Finance Department user account and review the sign-in logs to verify that the Conditional Access policies are applied correctly.


## Prerequisites

### Licensing and roles
Licensing and roles

A Microsoft Entra ID P1 or P2 license assigned to the test user, required for Conditional Access and dynamic group membership.

**Conditional Access Administrator** to create and manage Conditional Access policies.

**Authentication Policy Administrator**, to configure authentication methods.

**Groups Administrator** to create and manage dynamic security groups.

## Step 1: Create a Dynamic Security Group for the Finance Department

- Open the Entra admin center, go to Groups, and click New group.

<img width="1378" height="852" alt="image" src="https://github.com/user-attachments/assets/29f6ea3e-c724-42ba-bd54-b9280e9d96e5" />

- Fill in the required fields as shown in the screenshot below, and click Add dynamic query.

<img width="1051" height="723" alt="image" src="https://github.com/user-attachments/assets/7a2a6749-2a9b-4fd9-b4f9-54e1bab7b762" />

- Write the dynamic membership rule to include users in the Finance OU in the on-premises AD and click Save.

<img width="2555" height="560" alt="image" src="https://github.com/user-attachments/assets/f3387679-a007-4d64-b65e-12d703b2e9e3" />

- Click **No owners selected**, select the proper account, and create the role.

<img width="2176" height="1231" alt="image" src="https://github.com/user-attachments/assets/c00dee43-f9c7-4a05-ae95-df83109be61d" />

- Go to Groups and find the group that has been created. Click on the name to open it.

<img width="2556" height="642" alt="image" src="https://github.com/user-attachments/assets/b544033c-0743-4244-80fb-a76b3d98de90" />

- Go to Members and verify the list.

<img width="2556" height="1230" alt="image" src="https://github.com/user-attachments/assets/54f6f0dc-8a1f-4896-b789-8c946ae4b613" />

- Compare it with the on-premises list retrieved using the Get-ADUser command.

<img width="1339" height="546" alt="image" src="https://github.com/user-attachments/assets/fab4c862-7b99-47e8-b22b-3f8e94662331" />

## Step 2: Create a Conditional Access Policy based on the underlying assumptions.

- Go to Conditional Access, Policies, and click on New policy.

<img width="2555" height="812" alt="image" src="https://github.com/user-attachments/assets/7fd06e25-f8c4-41cd-aa41-3dad7aed1365" />

- Type the policy name, then exclude the Break Glass and administrator accounts to prevent accidentally blocking administrative access to the Microsoft Entra portal.

<img width="2278" height="1233" alt="image" src="https://github.com/user-attachments/assets/acbdc478-260b-4312-807f-9d73ff082460" />

- Then go to Include, select Users and groups, and choose the SG-CA-Finance group.

<img width="2279" height="1234" alt="image" src="https://github.com/user-attachments/assets/f07eb0fb-8ab6-4ac6-8388-49c62547e1d8" />

- Go to Target resources and select "All resources" in the Include section.

Microsoft recommends excluding emergency access (Break Glass) accounts from Conditional Access policies to prevent accidental tenant-wide lockout and ensure administrative access remains available.

<img width="631" height="747" alt="image" src="https://github.com/user-attachments/assets/0cce5f43-4f0f-4df4-ad74-8bbe2f6abbec" />

















