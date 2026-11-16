# Scenario 1: Sync AD Users to Entra ID with Entra Connect

## Goal

Install Microsoft Entra Connect, sync on-premises AD users to Entra ID, and exclude selected users using extensionAttribute15 (these accounts should not exist in the cloud, and privileged accounts such as admins should stay on-prem: synced accounts can be enumerated through Microsoft Graph and become targets for password spray attacks. Service and break-glass accounts often have weak or never-expiring passwords and no MFA, so they should not be reachable from the internet. Following least privilege, only people who need cloud access should have a cloud identity), and prove that password hash sync works by signing in to Entra with an AD password.

## Step 1: Prepare AD users

- Set extensionAttribute15 on the accounts to exclude them from the sync to the cloud. Below is the list of all excluded accounts (one account is excluded only for testing purposes).

<img width="831" height="443" alt="Zrzut ekranu 2026-11-15 170115" src="https://github.com/user-attachments/assets/ca8846fb-e1d1-459b-92c2-68b50ab528e3" />

## Step 2: Download Entra Connect

- In the Entra admin center, go to Entra ID, then Entra Connect, then Connect Sync, and download the Microsoft Entra Connect installer.
<img width="1260" height="1253" alt="Zrzut ekranu 2026-11-13 152554" src="https://github.com/user-attachments/assets/9dc741cb-1592-484a-9ecc-9db1634ee780" />

<img width="1275" height="1166" alt="Zrzut ekranu 2026-11-13 155119" src="https://github.com/user-attachments/assets/ca04d42f-70e3-4b3e-9c78-6cc2b3539126" />

<img width="581" height="478" alt="Zrzut ekranu 2026-11-13 155225" src="https://github.com/user-attachments/assets/57b12908-4842-488f-89a1-cd6ed9f97c40" />

## Step 3: Install Entra Connect on the Windows 2022 server

- Open the downloaded installer on the Windows 2022 server, run it as administrator, then agree to the license terms and privacy notice, and click Continue.
  
<img width="883" height="618" alt="Zrzut ekranu 2026-11-13 161029" src="https://github.com/user-attachments/assets/1e767763-1b42-4778-b28e-be46ef314a52" />

- On the next screen, click Customize instead of Use express settings.
  
<img width="706" height="492" alt="Zrzut ekranu 2026-11-14 151719" src="https://github.com/user-attachments/assets/1060a8ff-7560-427b-b6d2-5e92960f9fe8" />

- On the user sign-in screen, select Password Hash Synchronization, tick Enable single sign-on, and click Next.
  
<img width="702" height="493" alt="Zrzut ekranu 2026-11-14 152727" src="https://github.com/user-attachments/assets/6e65d34b-d5ff-4de2-824a-4fe4fa645894" />

- On the Connect to Microsoft Entra ID screen, sign in with an account that has the Hybrid Identity Administrator role.

<img width="702" height="497" alt="Zrzut ekranu 2026-11-14 155135" src="https://github.com/user-attachments/assets/c2855c58-1b4a-4820-b45e-af780d37d305" />

- On the Connect your directories screen, select your AD forest, and click Add Directory. In the AD forest account window, sign in with an on-premises account that has the required permissions in your domain, and click OK.

<img width="710" height="496" alt="Zrzut ekranu 2026-11-14 154539" src="https://github.com/user-attachments/assets/40922d8b-268d-4410-8260-2f8cf8289716" />

<img width="480" height="381" alt="Zrzut ekranu 2026-11-14 154701" src="https://github.com/user-attachments/assets/e5deb99d-4fde-4a17-bcd9-2ae3453c9351" />

- On the Microsoft Entra sign-in screen, which warns that users can’t sign in with on-premises credentials if the UPN suffix doesn’t match a verified domain, tick “Continue without matching all UPN suffixes to verified domains” and click Next.

<img width="707" height="497" alt="Zrzut ekranu 2026-11-14 155521" src="https://github.com/user-attachments/assets/b6a7af7b-1632-4d68-a633-8503b483f3af" />

- On the Domain and OU filtering screen, select “Sync all domains and OUs” and click Next.

<img width="702" height="497" alt="Zrzut ekranu 2026-11-14 160927" src="https://github.com/user-attachments/assets/0763c3a9-b1fa-46b2-8b80-a6a1273925e1" />

- On the Uniquely identifying your users screen, leave the default settings and click Next.

<img width="706" height="492" alt="Zrzut ekranu 2026-11-14 160156" src="https://github.com/user-attachments/assets/433bbe72-a7b6-4fcd-9d04-555700d7cbf7" />

- On the Optional features screen, tick Password writeback so password changes made in the cloud are written back to on-premises AD, and click Next.

<img width="702" height="496" alt="Zrzut ekranu 2026-11-14 160449" src="https://github.com/user-attachments/assets/35a27b32-85d8-4fdd-8928-b1752c728a3f" />

- On the Enable single sign-on screen, check that the credentials for the cam.iam forest, entered on an earlier screen, are accepted, which shows as a green circle with a tick, then click Next.

<img width="700" height="493" alt="Zrzut ekranu 2026-11-14 162655" src="https://github.com/user-attachments/assets/d9146ef3-087c-4ba4-82b1-9ffce51f12b2" />

- On the Ready to configure screen, untick "Start the synchronization process when configuration completes" because we need to make some other configuration changes before the first sync, and click Install.

<img width="701" height="495" alt="Zrzut ekranu 2026-11-14 164211" src="https://github.com/user-attachments/assets/c2b81a9d-b5f7-4e11-9f7c-9541fa62e0c4" />

- Once the installation finishes, the Configuration complete screen confirms that Microsoft Entra Connect was configured successfully, so click Exit. Since you unticked the automatic sync, nothing has been synchronized yet.

<img width="707" height="491" alt="Zrzut ekranu 2026-11-14 164720" src="https://github.com/user-attachments/assets/75d53fbd-db2d-4a80-b05b-3a16da1a18e9" />

## Step 4: Edit synchronization rules

- Open the Synchronization Rules Editor from the Start menu on the Windows 2022 server.
  
<img width="517" height="533" alt="Zrzut ekranu 2026-11-14 164846" src="https://github.com/user-attachments/assets/7c2d046d-1621-494e-8d99-dd33c15d23a9" />

- In the Synchronization Rules Editor, set the direction to Inbound, select the existing rule "In from AD - User Join", click Edit, and when prompted that the default rule will be disabled and an editable copy created, click Yes, since this is a lab.

<img width="756" height="583" alt="Zrzut ekranu 2026-11-15 165546" src="https://github.com/user-attachments/assets/38eab3e7-ef56-426a-8e9d-58ca54b9e96c" />

- On the Description tab of the editable copy, change the precedence value to 80, so it takes priority over the other inbound rules.

<img width="922" height="610" alt="Zrzut ekranu 2026-11-15 142155" src="https://github.com/user-attachments/assets/8bdec023-b07a-4f3a-9118-9a32f0fd4ba4" />

- On the Scoping filter tab, add a clause where the attribute extensionAttribute15 is NOTEQUAL to “DoNotSyncToCloud”, so users with that value are filtered out, then save the rule.

<img width="924" height="613" alt="Zrzut ekranu 2026-11-15 153131" src="https://github.com/user-attachments/assets/3ddffa20-f58c-47aa-8483-56ab2bf93273" />

## Step 5: Run the sync and verify the results

- Open PowerShell as administrator on the Windows 2022 server and run the sync as shown below.

<img width="782" height="406" alt="Zrzut ekranu 2026-11-15 170430" src="https://github.com/user-attachments/assets/d71eabd7-d06e-4d20-80cf-d3724a72e427" />

- In the Synchronization Service Manager, go to the Connectors tab, right-click the kam.iam connector, choose Run, select the Full Synchronization run profile, and click OK.

<img width="1015" height="705" alt="image" src="https://github.com/user-attachments/assets/910a3d16-8c93-4765-aa2e-a78f1126914f" />

- Right-click the Microsoft Entra ID connector, choose Run, select the Export run profile, and click OK to push the synchronized users to the cloud. Then go to the Operations tab and confirm that the export finished with the status “success”.

<img width="1014" height="709" alt="image" src="https://github.com/user-attachments/assets/f12e8da9-32f7-467d-969c-a9cf9615f67d" />

- In the Synchronization Service Manager, open the Metaverse Search tab and search for one of the excluded users, to confirm that the user was not synchronized.

<img width="810" height="302" alt="Zrzut ekranu 2026-11-15 173214" src="https://github.com/user-attachments/assets/212df735-70f3-48d1-b10f-a9bb907f38a8" />

- In the Entra admin center, go to Entra ID, then Users, then All users, and confirm that the users from AD now appear, with On-premises sync enabled set to Yes.

<img width="1409" height="991" alt="image" src="https://github.com/user-attachments/assets/68d7139f-ed34-431d-885f-4fe8190777b7" />

- In the Entra admin center, go to Users, then All users, and search for another one of the excluded users to confirm that the account does not exist in the cloud.

<img width="1761" height="459" alt="image" src="https://github.com/user-attachments/assets/a68634b3-160e-4828-89d1-b69b1cae2105" />

- Click on one of the synced users and open the Properties tab to see the values synchronized from on-premises AD.

<img width="1445" height="1235" alt="image" src="https://github.com/user-attachments/assets/dbf3cd9f-cfdf-42c1-89a4-4ff81905dbef" />

## Step 6: Verify password hash sync

- Open a private browser window, go to the Entra portal, and sign in as one of the synced users with their on-premises AD password.

<img width="986" height="697" alt="Zrzut ekranu 2026-11-15 182234" src="https://github.com/user-attachments/assets/9f1e967b-7dcb-4d5f-9af4-593b53c84e44" />
<img width="982" height="695" alt="Zrzut ekranu 2026-11-15 182308" src="https://github.com/user-attachments/assets/26ad6b08-70ed-4190-b23a-82d9aaa7734d" />

- After signing in, Microsoft shows a "Keep your account secure" prompt, which confirms the AD password was accepted, so click Next to start setting up the Microsoft Authenticator app for this user.

<img width="987" height="698" alt="Zrzut ekranu 2026-11-15 182348" src="https://github.com/user-attachments/assets/633bea50-762c-4928-ac3d-7b75661180aa" />

- On your mobile phone, install the Microsoft Authenticator app from the App Store or Google Play, then click Next in the browser.

<img width="990" height="704" alt="Zrzut ekranu 2026-11-15 182439" src="https://github.com/user-attachments/assets/13ef9304-f238-434c-a975-1d8a631c8a2d" />

- Open the Authenticator app on your phone, add a work or school account, and scan the QR code shown in the browser, as in the two screenshots below, then click Next.

<img width="986" height="697" alt="Zrzut ekranu 2026-11-15 182502" src="https://github.com/user-attachments/assets/ce545186-efff-41a3-83d6-50b0096d32a6" />
<img width="989" height="699" alt="Zrzut ekranu 2026-11-15 182552" src="https://github.com/user-attachments/assets/9434d1b9-0f96-497f-987f-e7ccbf9f1c2a" />

- Enter the six-digit code from the Authenticator app into the browser to confirm that the setup works, then click Next.
  
<img width="988" height="699" alt="Zrzut ekranu 2026-11-15 182631" src="https://github.com/user-attachments/assets/c4b48f98-2abb-4648-8264-0353f08c9d2c" />

- The next screen confirms that the Authenticator app was registered successfully, so click Done.
  
<img width="989" height="700" alt="Zrzut ekranu 2026-11-15 182706" src="https://github.com/user-attachments/assets/9088100e-f9e5-41fd-8fd0-bcc86d3bcb85" />

- Finally, the Microsoft Entra admin center opens, and the user's name in the top right corner confirms the sign-in worked, which proves that password hash sync is working and the AD password can be used to sign in to Entra ID.

<img width="2548" height="441" alt="Zrzut ekranu 2026-11-15 182832" src="https://github.com/user-attachments/assets/75f16a13-343a-47e0-bd58-c2b93fa2a107" />
