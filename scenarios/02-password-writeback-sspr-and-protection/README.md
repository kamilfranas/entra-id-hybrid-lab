# Scenario 2: Password Writeback, SSPR and Password Protection

## Goal

The help desk is receiving more tickets for password resets. In this scenario, you let users manage their own passwords without help desk involvement, while keeping on-premises Active Directory as the source of truth.
To perform the tasks, the Helpdesk Administrator and Authentication Policy Administrator roles must be active on your admin account. 
The user must have a Microsoft Entra ID P1 or P2 license assigned to use SSPR with password writeback.
First, you reset a synced user's password in the Entra admin center and confirm the new password works on a domain-joined Windows 11 machine, which proves that password writeback works. 
Next, you enable self-service password reset (SSPR) so users can reset their own passwords, and verify the login again.
Finally, you configure password protection to block weak and easily guessed passwords.

## Prerequisites

### Licensing and roles

- A Microsoft Entra ID P1 or P2 license assigned to the test user, required for SSPR with password writeback.
- **Helpdesk Administrator**, to reset a synced user's password.
- **Authentication Policy Administrator**, to configure SSPR and password protection settings.
- **Global Administrator**, to register the first Password Protection proxy in the tenant.
- **Global Administrator** or **Security Administrator**, to register the Active Directory forest.
- **Enterprise Administrator** in on-premises Active Directory, also needed for the forest registration.

### PowerShell commands used

Loads the Password Protection module so its cmdlets are available:

```powershell
Import-Module AzureADPasswordProtection
```

Checks that the proxy service is installed and running:

```powershell
Get-Service AzureADPasswordProtectionProxy | fl
```

Registers the proxy server with Microsoft Entra ID, using device code sign-in:

```powershell
Register-AzureADPasswordProtectionProxy -AccountUpn 'admin@contoso.onmicrosoft.com' -AuthenticateUsingDeviceCode
```

Registers the Active Directory forest with Microsoft Entra ID, using device code sign-in:

```powershell
Register-AzureADPasswordProtectionForest -AccountUpn 'admin@contoso.onmicrosoft.com' -AuthenticateUsingDeviceCode
```

Tests that the proxy, TLS and Azure connectivity are healthy:

```powershell
Test-AzureADPasswordProtectionProxyHealth -TestAll
```

## Step 1: Reset a synced user's password in Entra

- In the Entra admin center, reset the password of a synced test user. 

<img width="2556" height="728" alt="image" src="https://github.com/user-attachments/assets/18970b21-5b2a-429f-8ead-971c23c6a286" />

<img width="301" height="256" alt="image" src="https://github.com/user-attachments/assets/8069364f-d53c-4909-a19e-5dcecb6600a3" />

- If the reset fails with the message that your on-premises policy does not allow it, check the domain password policy in Active Directory, in particular the minimum password age, which blocks changing a password that was set recently.

<img width="977" height="509" alt="image" src="https://github.com/user-attachments/assets/5d5dbf8d-dc47-418f-88d8-07b7792c223d" />

- Sign in using the new password (set in Entra ID) and then confirm the change.
<img width="1030" height="770" alt="image" src="https://github.com/user-attachments/assets/c6205688-cfba-465f-8784-7d6383daef75" />
<img width="1022" height="768" alt="image" src="https://github.com/user-attachments/assets/c4c602e4-b99c-47dd-be65-dc27686d94c9" />

- Type the new password and click on the arrow.
 
<img width="1025" height="764" alt="image" src="https://github.com/user-attachments/assets/96bdca30-be76-4ac8-9a08-1d5c6ddb3d44" />

- The password has been updated.
  
<img width="1022" height="763" alt="image" src="https://github.com/user-attachments/assets/972c3eeb-c6b4-452f-9053-489760b3b10e" />

- The currently signed-in domain user is displayed using the whoami command in CMD.

<img width="1026" height="768" alt="image" src="https://github.com/user-attachments/assets/31ce7149-1255-4624-8523-0e6db9f4cd8a" />

## Step 2: Configure SSPR

- First, create a new group.

<img width="1380" height="1232" alt="image" src="https://github.com/user-attachments/assets/daf00b37-9aff-4efc-b828-8cc3ea1262c8" />

- Select the owner(s).

<img width="2168" height="1226" alt="image" src="https://github.com/user-attachments/assets/ae3e8dd8-be54-4f5c-aa92-5ef70b8a6d3a" />

- For this scenario, only a few accounts will be added. Select them and click Create.

<img width="2173" height="1226" alt="image" src="https://github.com/user-attachments/assets/8c391da2-4c14-43cd-a945-91c57113bb2c" />

- Once the group is created, navigate to Password reset → Properties, select Selected (Self service password reset enabled), choose the newly created group, and click Save.

<img width="2168" height="1228" alt="image" src="https://github.com/user-attachments/assets/07732a65-38c3-4c8f-9664-a84b89690023" />

- Navigate to the SSPR portal (https://aka.ms/sspr) and enter the user's sign-in name (UPN). Click Next.

<img width="988" height="698" alt="image" src="https://github.com/user-attachments/assets/111f1eb5-e825-422d-bd02-cd4384f28abc" />

- Open the Microsoft Authenticator app on your mobile device and enter the 6-digit verification code in the browser.

<img width="988" height="694" alt="image" src="https://github.com/user-attachments/assets/d7d85afe-47db-4a79-b826-64eb74e04e14" />

- Enter the new password, confirm it, and click Finish.

<img width="987" height="695" alt="image" src="https://github.com/user-attachments/assets/6efdfccf-7822-4b94-8c23-1ebc5e7099fa" />

- A confirmation message indicates that the password has been successfully reset.

<img width="987" height="691" alt="image" src="https://github.com/user-attachments/assets/ae42a8e2-e141-42eb-b1ac-8337cb0c1f73" />

- Verify the password change by signing in to the Microsoft Azure portal using the new password.
  
<img width="988" height="693" alt="image" src="https://github.com/user-attachments/assets/c15f96c0-366c-4eb3-bcd9-1a47defa09fa" />

<img width="990" height="693" alt="image" src="https://github.com/user-attachments/assets/a97bd7a8-2886-4748-8438-45f242a2714b" />

<img width="844" height="696" alt="image" src="https://github.com/user-attachments/assets/df4d0d9d-21bc-4428-ba60-0167a140e98a" />

- Verify the password change by signing in to the Windows 11 VM joined to the domain.

<img width="1025" height="769" alt="image" src="https://github.com/user-attachments/assets/333dd744-1c76-469f-a811-a91a1b35de8a" />

<img width="1027" height="769" alt="image" src="https://github.com/user-attachments/assets/36228e2d-e9f4-446f-810b-ac9af506849d" />

## Step 3: Configure Microsoft Entra Password Protection

*The Global banned password list is a Microsoft-maintained list of commonly used, weak, and easily guessed passwords. Microsoft automatically checks passwords against this list to prevent users from choosing vulnerable passwords. It is always enabled and does not require administrator configuration.

- Open the Microsoft Download Center page for Microsoft Entra Password Protection for Windows Server Active Directory, which lists the two installers: the proxy setup file and the DC agent setup file.

<img width="1621" height="1058" alt="image" src="https://github.com/user-attachments/assets/97019207-ac6b-4523-aa81-aa2eea52ad73" />

- Select both files in the download dialog, the proxy installer and the DC agent installer, then click Download.

<img width="1163" height="505" alt="image" src="https://github.com/user-attachments/assets/05a103f9-4bba-419c-8be3-b59a21b49a10" />

- On the Windows Server 2022 member server, run the Microsoft Entra Password Protection proxy installer, accept the license terms and conditions, and click Install.

<img width="478" height="297" alt="image" src="https://github.com/user-attachments/assets/5c3007ac-7453-4c8a-89bc-3d70b9e97410" />

- In an elevated 64-bit PowerShell window, import the AzureADPasswordProtection module and run Get-Service AzureADPasswordProtectionProxy to confirm the service status is Running.
  Then register the proxy by running Register-AzureADPasswordProtectionProxy with the AccountUpn and AuthenticateUsingDeviceCode parameters. PowerShell displays a sign-in link and a device code.

<img width="1327" height="516" alt="image" src="https://github.com/user-attachments/assets/bbcb46a6-e587-4816-a3ab-a5dae8f312b2" />

- Open the Microsoft device login page, enter the code shown in PowerShell to allow access, and click Next.

<img width="839" height="689" alt="image" src="https://github.com/user-attachments/assets/46bef69a-52ac-49ab-8c3e-61a9ee7d1851" />

- When prompted to approve the sign-in on another device, confirm the request by selecting Approve.

<img width="845" height="862" alt="image" src="https://github.com/user-attachments/assets/efb3a687-ea54-4983-bb02-baf48075057d" />

- The browser confirms the sign-in with the message “You have signed in to the Microsoft Azure PowerShell application on your device,” and the window can be closed.

<img width="845" height="860" alt="image" src="https://github.com/user-attachments/assets/faddbe2e-df2c-4ffd-a83b-b9e6dd8e4ac2" />

- In the same PowerShell window, register the Active Directory forest by running Register-AzureADPasswordProtectionForest with the AccountUpn and AuthenticateUsingDeviceCode parameters.
  Complete the sign-in the same way as for the proxy registration: open the Microsoft device login page, enter the code, and approve the request.
  Verify the deployment by running Test-AzureADPasswordProtectionProxyHealth with the TestAll parameter. All three tests, covering the proxy, TLS, and Azure connectivity, return Passed.

<img width="1325" height="513" alt="image" src="https://github.com/user-attachments/assets/60655184-4e05-47f8-907c-f99b98149c5a" />

- On the Windows Server 2019 domain controller, run the Microsoft Entra Password Protection DC agent installer, accept the license terms, and click Install.

<img width="494" height="385" alt="image" src="https://github.com/user-attachments/assets/95af14b3-3d1c-49ac-b07a-791a614a9eac" />

- When the installer says the system must be restarted to complete the configuration, click Yes. The password filter only loads after the restart, and no further configuration is needed on the server.

<img width="361" height="166" alt="image" src="https://github.com/user-attachments/assets/dc29237f-9170-4bfb-b1f9-b117eed56ced" />

- Navigate to Entra ID → Authentication methods → Password protection, set Enforce custom banned password list to Yes, and add an example password to the custom banned password list.
  
<img width="1278" height="845" alt="Zrzut ekranu 2026-11-17 094834" src="https://github.com/user-attachments/assets/1c8f9dcc-7e45-4b20-a0da-6b38c665a797" />

- In the Microsoft Entra admin center, attempt to reset the password of a synced user to one containing a banned term.
  The reset is rejected with the message that the user’s password cannot be reset because the on-premises password policy does not allow it, which shows that on-premises Password Protection is enabled and enforcing the custom banned password list.

<img width="313" height="351" alt="image" src="https://github.com/user-attachments/assets/d7266ea4-486a-418f-bb84-5a2358ebdafd" />

- On the Windows Server 2019 domain controller, open Event Viewer and go to Applications and Services Logs, Microsoft, AzureADPasswordProtection, DCAgent, Admin.
  The most recent event shows that the service is now enforcing the current Azure password policy, and its tenant policy date is from today, a few minutes earlier, which confirms the DC agent downloaded the latest policy.

<img width="1481" height="766" alt="Zrzut ekranu 2026-11-17 100023" src="https://github.com/user-attachments/assets/de305991-31a3-468d-8cb9-341ba041c54c" />

- Another event in the same Admin log shows that the password reset for the specified user was rejected because it matched at least one token in the tenant’s custom banned password list in the current Azure password policy, which confirms that on-premises enforcement works.

<img width="1483" height="751" alt="Zrzut ekranu 2026-11-17 100136" src="https://github.com/user-attachments/assets/5b61c971-40d5-4628-a1a0-c9971f3c73f9" />









