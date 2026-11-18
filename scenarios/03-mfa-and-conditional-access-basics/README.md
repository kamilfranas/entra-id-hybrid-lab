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

Throughout the steps, you will also find additional diagnostic tips that show the complexity of this configuration and how to approach problems when you get stuck. At the end of the scenario, you will also see why a single Conditional Access policy is sometimes not enough and how multiple policies work together.

## Prerequisites

### Licensing and roles

A Microsoft Entra ID P1 or P2 license, required for Conditional Access and dynamic group membership, and a Microsoft Intune Plan 1 license, required for device enrollment and compliance evaluation, both assigned to the test user.

- **Conditional Access Administrator** to create and manage Conditional Access policies.

- **Authentication Policy Administrator** to configure authentication methods.

- **Groups Administrator** to create and manage dynamic security groups.

- **Intune Administrator** to create and assign compliance policies and manage enrolled devices.

- **Hybrid Identity Administrator** to configure Microsoft Entra Connect and device registration.


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


## Step 2: Create Named Locations for Poland (GPS and IP Address)

- Go to Conditional Access, select Named locations, and click Countries location. Enter the name Poland IP, and under Determine location by, select IP address. Select Poland from the country list, and click Create.

<img width="2553" height="1232" alt="image" src="https://github.com/user-attachments/assets/dd197593-5232-4dfe-967f-b40f26674460" />

- Repeat the process to create a second named location. Click Countries location, enter the name Poland GPS, and under Determine location by, select GPS coordinates. Select Poland from the country list, and click Create.

<img width="2558" height="1230" alt="image" src="https://github.com/user-attachments/assets/1df81e45-983d-48e1-be8b-210b33871858" />

- Verify that both named locations, Poland IP and Poland GPS, have been created. The Conditional Access policies column shows that they are not yet used in any policy; this will change after the block policy is created.

<img width="1777" height="819" alt="image" src="https://github.com/user-attachments/assets/866d3734-702c-428c-9976-896f023ddb4a" />


## Step 3: Enroll a Device in Microsoft Intune and Configure Device Compliance

- Open the Microsoft Intune admin center, go to Users, then All users, type the name of the Finance test user in the search box, and open the account. 

<img width="2551" height="481" alt="image" src="https://github.com/user-attachments/assets/1339075d-f2f6-415f-82e6-9b9060471cc2" />

- Verify that the Microsoft Intune Plan 1 license is assigned.

<img width="1100" height="611" alt="image" src="https://github.com/user-attachments/assets/5679a04a-a707-4670-8abc-69aa0ab68bc6" />

- On the Entra Connect server, run the Microsoft Entra Connect wizard, click Configure, select Configure device options, and click Next.
  
<img width="875" height="616" alt="image" src="https://github.com/user-attachments/assets/0206782f-bc54-43d9-b58a-5ed5b0ed7fc9" />

- On the Overview page, review the description of the device options and click Next.

<img width="881" height="620" alt="image" src="https://github.com/user-attachments/assets/1045068f-1345-4375-a5a5-4c36b5a8854d" />

- Sign in to Microsoft Entra ID using your administrator account and click Next.

<img width="880" height="626" alt="image" src="https://github.com/user-attachments/assets/5d361b07-b6b7-4f48-b1cf-83ed803d86c5" />

- On the Device options page, select Configure Hybrid Microsoft Entra ID join and click Next.
  
<img width="879" height="620" alt="image" src="https://github.com/user-attachments/assets/c039c980-0a18-48da-8a79-6d0c4817c514" />

- On the Device operating systems page, select Windows 10 or later domain-joined devices and click Next.

<img width="880" height="619" alt="image" src="https://github.com/user-attachments/assets/06a9562e-2316-4108-ab14-21a4ff2dae51" />

- On the SCP configuration page, select your forest, set the authentication service to Microsoft Entra ID, click Add, and enter the enterprise administrator credentials for your on-premises AD.

<img width="882" height="621" alt="image" src="https://github.com/user-attachments/assets/44088786-6d4a-4741-8ef5-026c86f279e3" />

- On the Ready to configure page, verify that the SCP for device registration will be configured for your forest, and click Configure.

- <img width="881" height="620" alt="image" src="https://github.com/user-attachments/assets/80e7102c-8802-4305-ae0f-1a50cd767225" />

- When the configuration completes and the wizard reports that the hybrid Microsoft Entra ID join tasks completed successfully, click Exit.

<img width="877" height="625" alt="image" src="https://github.com/user-attachments/assets/7239476c-7e05-40f0-870b-606b229ba114" />

- Sign in to the Windows 11 virtual machine with the Finance test user account so that the device can start the hybrid Microsoft Entra ID join process.

<img width="1019" height="769" alt="image" src="https://github.com/user-attachments/assets/e2ede87e-9f50-4b91-a444-2553f202ef7a" />

- The process can take about 30 minutes. On the Entra Connect server, run Delta Sync as shown in the screenshot below.

<img width="973" height="278" alt="image" src="https://github.com/user-attachments/assets/3c512c90-299f-4ed2-a51d-27675432d533" />

- In the Microsoft Entra admin center, go to Devices, then All devices, and verify that the Windows 11 virtual machine appears in the list. The Registered column shows Pending, which means the computer object has been synchronized and the device is waiting to complete its hybrid join. 

<img width="2555" height="525" alt="image" src="https://github.com/user-attachments/assets/ebce92ef-d393-4f8f-94ab-bf4232ed08c3" />

- Manually trigger the Automatic-Device-Join scheduled task and accelerate the Microsoft Entra hybrid device registration process.

<img width="1003" height="639" alt="image" src="https://github.com/user-attachments/assets/07018dea-1d0a-446d-98ad-84f65c8359a4" />

- In the Microsoft Intune admin center, go to Devices, Device Onboarding, Enrollment, Windows, and then Automatic Enrollment.

<img width="2558" height="1230" alt="image" src="https://github.com/user-attachments/assets/e6027aad-4ee6-4f1d-bba4-d95e79c42897" />

- In the Microsoft Intune admin center, select Some under MDM user scope, click Select groups, choose the Finance group we created earlier, and click Select.

<img width="2554" height="1231" alt="image" src="https://github.com/user-attachments/assets/6df7d47a-05e3-4052-91fe-2a08e348039d" />

- Move the Windows 11 computer from the default Computers container to the newly created Workstations OU in on-premises Active Directory.

<img width="1339" height="328" alt="image" src="https://github.com/user-attachments/assets/fdbb8973-c753-4530-9f78-a8f00806f9e9" />

- Open Group Policy Management, right-click the Workstations OU, select Create a GPO in this domain, and Link it here. Name the new GPO "GPO - Automatic Intune Enrollment".

<img width="1129" height="616" alt="image" src="https://github.com/user-attachments/assets/429624c1-a720-459b-93fa-a4886b2cf41b" />

- Right-click the GPO you just created under Workstations and select Edit. Then navigate to Computer Configuration, Policies, Administrative Templates, Windows Components, MDM. Open Enable automatic MDM enrollment using default Microsoft Entra credentials, set it to Enabled, choose User Credential, apply, and click OK.

<img width="1732" height="1012" alt="image" src="https://github.com/user-attachments/assets/2bbaf193-20e7-4e49-98d5-20e9d34cdf01" />

- In the Microsoft Intune admin center, go to Devices, then Windows, and notice that the virtual machine does not appear in the list even though the automatic enrollment GPO has been applied.

<img width="2043" height="799" alt="image" src="https://github.com/user-attachments/assets/8ab10952-3ebd-41cd-afe1-4422be9df44e" />

- **To identify the cause, we will perform several troubleshooting steps on the virtual machine.**

- On the virtual machine, open Event Viewer and go to Applications and Services Logs, Microsoft, Windows, DeviceManagement-Enterprise-Diagnostics-Provider, Admin. Notice that no new enrollment events have been logged, which means the device has not attempted to enroll in Intune.

<img width="1022" height="765" alt="image" src="https://github.com/user-attachments/assets/d9994fd5-5e2b-40ba-9804-aa7cebb171fd" />

- Then open Task Scheduler and check whether an enrollment task exists under Microsoft, Windows, EnterpriseMgmt.

<img width="784" height="562" alt="image" src="https://github.com/user-attachments/assets/26666e54-ae43-44bb-8169-553c320a0d8a" />

- On the Windows 11 VM, run gpresult /scope computer /r in an administrator Command Prompt, and verify that the computer is located in the Workstations OU.
  
<img width="981" height="512" alt="image" src="https://github.com/user-attachments/assets/186204df-08ec-454a-83cc-20b8a51db1a0" />

- Then, scroll down and confirm that the automatic Intune enrollment GPO is listed under Applied Group Policy Objects.

<img width="972" height="510" alt="image" src="https://github.com/user-attachments/assets/fd6a1bd9-bf5b-46d7-9945-49477dfdecd9" />

- Run dsregcmd /status in the Command Prompt and scroll down to the Tenant Details section. Notice that the MDM URL fields are empty, which means the device has not yet received its MDM enrollment settings from Microsoft Entra ID.

<img width="978" height="724" alt="image" src="https://github.com/user-attachments/assets/bf62f0c4-1a62-471f-941c-f20fc60695f1" />

- In Active Directory Users and Computers, open the properties of the Finance test user, go to the Account tab, and change the User logon name suffix to the tenant's onmicrosoft.com domain, then click Apply.

<img width="756" height="530" alt="image" src="https://github.com/user-attachments/assets/1841558f-29bf-41a6-8c84-fc0f077426a9" />

- After changing the UPN suffix and signing in again, run dsregcmd /status and verify that the Tenant Details section now shows the MDM URL, which means the device can enroll in Intune.

<img width="990" height="624" alt="image" src="https://github.com/user-attachments/assets/f396ca59-2c23-4636-8ef0-78d320122958" />

- Scroll down to the SSO State section and verify that AzureAdPrt is set to YES, and check in the Diagnostic Data section that the signed-in account now uses the tenant UPN, which confirms that the user can authenticate against Microsoft Entra ID.

<img width="994" height="626" alt="image" src="https://github.com/user-attachments/assets/62c459ad-ee79-47b5-addc-a7a6b526a2fd" />

- After a few minutes, in the Microsoft Entra admin center, go to Devices, then All devices. Verify that the MDM and Security Settings Management fields are set to Microsoft Intune and that the Finance test user is the owner.

<img width="2556" height="521" alt="image" src="https://github.com/user-attachments/assets/d4a5c279-773c-41d3-867f-17c25750288b" />

- In the Microsoft Intune admin center, go to Devices, then Windows, and verify that the virtual machine now appears in the list.

<img width="2173" height="469" alt="image" src="https://github.com/user-attachments/assets/11c8027d-9ebc-4cc4-ba99-02d00b6fe735" />


## Step 4: Create a Conditional Access Policy to Require MFA and Compliant Devices

- Go to Conditional Access, Policies, and click on New policy.

<img width="2555" height="812" alt="image" src="https://github.com/user-attachments/assets/7fd06e25-f8c4-41cd-aa41-3dad7aed1365" />

- Type the policy name, then exclude the Break Glass and administrator accounts to prevent accidentally blocking administrative access to the Microsoft Entra portal.

<img width="2278" height="1233" alt="image" src="https://github.com/user-attachments/assets/acbdc478-260b-4312-807f-9d73ff082460" />

- Then go to Include, select Users and groups, and choose the SG-CA-Finance group.

<img width="2279" height="1234" alt="image" src="https://github.com/user-attachments/assets/f07eb0fb-8ab6-4ac6-8388-49c62547e1d8" />

- Go to Target resources and select "All resources" in the Include section.

Microsoft recommends excluding emergency access (Break Glass) accounts from Conditional Access policies to prevent accidental tenant-wide lockout and ensure administrative access remains available.

<img width="631" height="747" alt="image" src="https://github.com/user-attachments/assets/0cce5f43-4f0f-4df4-ad74-8bbe2f6abbec" />

- In the Grant section, make sure Grant access is selected, then tick Require multifactor authentication and Require device to be marked as compliant.

<img width="1274" height="1232" alt="image" src="https://github.com/user-attachments/assets/7e302a0b-f905-4a84-97d9-7f932172e07f" />

- Scroll down, make sure Require all the selected controls is selected, and click Select.

<img width="1272" height="1236" alt="image" src="https://github.com/user-attachments/assets/dc91f78a-640a-4c82-8f41-8c7f42fdac8f" />

- No session controls are required for this policy, so leave this section unchanged.
- If a warning appears about compliant devices in report-only mode, click Proceed to continue with the selected configuration. Make sure Enable policy is set to Report-only for now, and click Create.

<img width="1213" height="1229" alt="image" src="https://github.com/user-attachments/assets/9ac430e5-430f-47d6-adc4-567b413928d4" />


## Step 5: Create a Conditional Access Policy to Block Access

- Go to Conditional Access, Policies, and click on New policy. Type the policy name, then exclude the Break Glass and administrator accounts.

<img width="2555" height="1228" alt="image" src="https://github.com/user-attachments/assets/3c506016-556b-44b8-9a38-f9e48f3c6124" />

- In the Include section, choose Users and groups, and select the SG-CA-Finance security group.

<img width="2277" height="1230" alt="image" src="https://github.com/user-attachments/assets/12327e5f-74da-4c22-9245-633cedaf57b2" />

- Next, go to Target resources and select All resources under Include.

<img width="615" height="756" alt="image" src="https://github.com/user-attachments/assets/3f962f82-96ae-4282-aa0b-decdca7477c6" />

- Configure the Network condition to include all locations and exclude the Poland IP and Poland GPS named locations.
  
<img width="2277" height="1229" alt="image" src="https://github.com/user-attachments/assets/fc683252-1a98-43fa-847d-7c8ff45f8970" />

- Under Grant, select Block access to prevent sign-ins from locations outside Poland. Set Enable policy to Report-only, then click Create to save the Conditional Access policy.

<img width="2273" height="1229" alt="image" src="https://github.com/user-attachments/assets/4815e746-d323-4103-b993-04fe61d598b0" />


## Step 6: Tests

- Open the Tor Browser and sign in to the Azure portal.

<img width="1022" height="718" alt="image" src="https://github.com/user-attachments/assets/5bbc4ce0-afe9-4a15-a3c3-728a1546d748" />

- Enter your password

<img width="1017" height="725" alt="image" src="https://github.com/user-attachments/assets/b6c7761a-aa52-4501-9bbb-c90aabc7097e" />

- Open Authenticator and enter the code. Location must be confirmed (if not set by default in your mobile's settings).

<img width="1023" height="722" alt="image" src="https://github.com/user-attachments/assets/4a4ed2a2-d804-43ac-bb79-349932e40140" />

- The warning appears, and access is blocked because Tor does not allow providing a device information (IP addresses are not considered here)

<img width="1021" height="720" alt="image" src="https://github.com/user-attachments/assets/3d30c3e9-5892-48de-9949-99d044436f95" />

- Go to Conditional Access, Monitoring, Sign-in logs, and find the proper record in the list. Open it.

<img width="2503" height="915" alt="image" src="https://github.com/user-attachments/assets/be16c047-33cf-4645-8c97-a322ed839fa5" />

- The Failure reason indicates that the device is not compliant.

<img width="842" height="436" alt="image" src="https://github.com/user-attachments/assets/c6f7f52f-0640-4e95-92a8-432b6a880a14" />

- Download and install any free VPN and set the country to Canada. Sign in to the Azure Portal. Why is access not blocked based on location?

<img width="1023" height="720" alt="image" src="https://github.com/user-attachments/assets/7d49615e-368e-47eb-83a1-bdd6cc4bd64c" />

- Sign-in log shows success; MFA requirement satisfied by claim in the token.

<img width="842" height="553" alt="image" src="https://github.com/user-attachments/assets/d9146deb-a3cd-4c7d-9846-f25f81594b1c" />

- The Conditional Access tab shows that the block policy was not applied

<img width="849" height="314" alt="image" src="https://github.com/user-attachments/assets/3f7686c8-6f1a-459a-a0b0-ae005a878b72" />

- The details show that the location does not match

<img width="726" height="856" alt="image" src="https://github.com/user-attachments/assets/8d91eaca-553f-44ba-9e4b-4c410e0e1a9a" />

- Edit the block policy. Go to Network, Exclude, and select only Poland_IP. Then, create a copy of the policy, but this time in the exclude choose Poland_GPS.

<img width="623" height="1230" alt="image" src="https://github.com/user-attachments/assets/cd22dcb1-46c3-42e1-9eef-683da6b7a34e" />

- Once again, sign in to the Azure Portal; access is blocked.

<img width="984" height="690" alt="image" src="https://github.com/user-attachments/assets/61b9bab5-3837-456c-9a21-62a832abc5e9" />

- Troubleshooting details show that the device is compliant, and the error code indicates that the access is blocked by the Conditional Access Policy.

<img width="985" height="692" alt="image" src="https://github.com/user-attachments/assets/d6f492de-f4d8-4d1e-9296-7ab1bdb2565a" />

- Disable the block policy that uses IP, and make sure the VPN is not set to Poland. In the Sign-ins, block-GPS is not applied.

<img width="855" height="310" alt="image" src="https://github.com/user-attachments/assets/3cd463db-e74c-4203-9263-fb39329a12e7" />

- The location tab indicates the Authenticator app location	is Poland, which is excluded from the block list.
  
<img width="854" height="356" alt="image" src="https://github.com/user-attachments/assets/3ce669ae-2c3c-4b04-b20d-3cbfab89085f" />

- Disabled policy does not provide any details in the request. Change it to read-only mode and try another country via VPN.
- “Report-only: Failure means that the policy would have blocked access if it had been enabled.

<img width="852" height="940" alt="image" src="https://github.com/user-attachments/assets/e46d2ae5-cbd5-4d2c-ab56-ef708845d615" />

