# Scenario 2: Password Writeback, SSPR and Password Protection

## Goal

The help desk is receiving more tickets for password resets. In this scenario, you let users manage their own passwords without help desk involvement, while keeping on-premises Active Directory as the source of truth.
To perform the tasks, the Helpdesk Administrator and Authentication Policy Administrator roles must be active on your admin account. 
The user must have a Microsoft Entra ID P1 or P2 license assigned to use SSPR with password writeback.
First, you reset a synced user's password in the Entra admin center and confirm the new password works on a domain-joined Windows 11 machine, which proves that password writeback works. 
Next, you enable self-service password reset (SSPR) so users can reset their own passwords, and verify the login again.
Finally, you configure password protection to block weak and easily guessed passwords.


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

- Enter the new password, confirm, and click Finish.

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








