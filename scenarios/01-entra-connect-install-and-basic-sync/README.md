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

