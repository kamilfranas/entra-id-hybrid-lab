# Privileged Accounts and Break-Glass Setup with Microsoft Graph

## Goal

Set up a secure privileged access foundation in a Microsoft Entra ID tenant: two cloud-only emergency access (break-glass) accounts protected with phishing-resistant, passwordless authentication, and a separate named administrator account with only the roles it needs. Every step is performed with Microsoft Graph PowerShell and verified with Graph queries.

## Prerequisites

- Microsoft Entra ID tenant with Global Administrator access
- Microsoft Graph PowerShell SDK installed
- One FIDO2 security key (for the first break-glass account)
- A certificate authority and a certificate for certificate-based authentication (for the second break-glass account)
- Microsoft Authenticator on a mobile device (for the named administrator account)
- Existing Conditional Access policies to exclude the break-glass accounts from

## Scenario Overview

This scenario describes the account design, the protection of each account, and the Conditional Access exclusions that keep the emergency accounts from being locked out.

| Account | Type | Authentication method |
|---------|------|-----------------------|
| Break-glass-account-1 | Cloud-only, Global Administrator | FIDO2 security key |
| Break-glass-account-2 | Cloud-only, Global Administrator | Certificate-based authentication |
| adm-kamil | Cloud-only, limited admin roles | Microsoft Authenticator |

## Step 1: Install and connect to Microsoft Graph

- Install the Microsoft Graph PowerShell module, then connect to Microsoft Graph with the scopes shown in the screenshot below, and sign in with your work or school account that has the Global Administrator role.

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","RoleManagement.ReadWrite.Directory","Policy.ReadWrite.ConditionalAccess","Policy.ReadWrite.AuthenticationMethod","UserAuthenticationMethod.ReadWrite.All"
```

<img width="1119" height="662" alt="image" src="https://github.com/user-attachments/assets/2694ec74-d07d-4231-b495-2ca5752d92a9" />

- After signing in, PowerShell confirms the connection to Microsoft Graph.

<img width="1160" height="420" alt="image" src="https://github.com/user-attachments/assets/e4be5f36-4ecb-4e39-98e1-c98e62d1beba" />

- To check which scopes were granted for the current session, run Get-MgContext and expand the Scopes property.

```powershell
(Get-MgContext).Scopes
Get-MgContext | select Scopes -ExpandProperty Scopes
```

<img width="1164" height="813" alt="image" src="https://github.com/user-attachments/assets/ce4e5f86-071a-47ff-b612-d1b3aa0b57bd" />

## Step 2: Create the break-glass accounts

- Prompt for the initial password as a secure string so it never appears on screen, convert it for the password profile, and create the first break-glass account with New-MgUser.

```powershell
$secure = Read-Host "Enter initial password" -AsSecureString

$plain = [System.Net.NetworkCredential]::new("", $secure).Password
New-MgUser `
  -DisplayName "Break-glass-account-1" `
  -UserPrincipalName "Break-glass-account-1@yourtenant.onmicrosoft.com" `
  -MailNickname "Break-glass-account-1" `
  -AccountEnabled `
  -PasswordProfile @{ Password = $plain; ForceChangePasswordNextSignIn = $false }
```

<img width="1165" height="376" alt="image" src="https://github.com/user-attachments/assets/748ab87c-7884-4657-a9ed-ee5a85b73b3b" />

- Verify that both break-glass accounts were created by listing them with Get-MgUser.

```powershell
Get-MgUser -Filter "startswith(displayName, 'Break-glass')" -Property *
``` 

<img width="1163" height="238" alt="image" src="https://github.com/user-attachments/assets/6b91735e-7ff2-4aac-9447-7d18dd764afe" />


## Step 3: Create the break-glass security group and add the accounts.

- Create a new security group for the break-glass accounts with New-MgGroup, and store the result in a variable so its ID can be reused in the next steps.

```powershell
$group = New-MgGroup `
  -DisplayName "Break-glass-accounts" `
  -MailEnabled:$false `
  -SecurityEnabled:$true `
  -MailNickname "break-glass-accounts" `
  -Description "Emergency access accounts excluded from Conditional Access"

$group
```

<img width="1169" height="317" alt="image" src="https://github.com/user-attachments/assets/7d52882d-00d7-405f-a14b-622da5153b14" />

```powershell
Get-MgUser -Filter "startsWith(displayName, 'Break-glass')"

$users

$users | ForEach-Object { New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $_.Id }

Get-MgGroupMember -GroupId $group.Id
```

- Add both break-glass accounts to the group with New-MgGroupMember, then confirm the membership with Get-MgGroupMember.

<img width="1173" height="412" alt="image" src="https://github.com/user-attachments/assets/ae91597d-4124-47c7-ab89-d8ebdf087427" />

## Step 4: Assign the Global Administrator role

- Look up the Global Administrator role definition with Get-MgRoleManagementDirectoryRoleDefinition, assign it to both break-glass accounts with New-MgRoleManagementDirectoryRoleAssignment, and confirm the assignments.

```powershell
$role = Get-MgDirectoryRole | Where-Object {$_.DisplayName -eq "Global Administrator"}

$users | ForEach-Object { New-MgRoleManagementDirectoryRoleAssignment `
  -PrincipalId $_.Id `
  -RoleDefinitionId $role.Id `
  -DirectoryScopeId "/" }

Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id | ForEach-Object {Get-MgUser -UserId $_.Id }

```

<img width="1638" height="438" alt="image" src="https://github.com/user-attachments/assets/ee763fb8-12dc-4d33-b6dc-3f0efae92940" />


## Step 5: Register the FIDO2 key and configure certificate-based authentication

### Register the FIDO2 key

- Enable the FIDO2 authentication method for the break-glass group by updating the Fido2 configuration in the authentication methods policy, so the accounts in that group can register a security key.

```powershell
$params = @{
    "@odata.type" = "#microsoft.graph.fido2AuthenticationMethodConfiguration"
    state = "enabled"
    includeTargets = @( @{ targetType = "group"; id = $group.Id } )
}

Update-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "Fido2" -BodyParameter $params
```

<img width="986" height="211" alt="image" src="https://github.com/user-attachments/assets/e47012d8-bbba-4436-8f8e-e36079d77b94" />

- Check that the Temporary Access Pass method is enabled for the break-glass group, then create a one-time pass for Break-glass-account-1, which it will use to sign in and register the FIDO2 key.

```powershell
Get-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "TemporaryAccessPass" |
    Select-Object Id, State

$tapParams = @{
    "@odata.type" = "#microsoft.graph.temporaryAccessPassAuthenticationMethodConfiguration"
    state = "enabled"
    isUsableOnce = $true
    defaultLifetimeInMinutes = 60
    includeTargets = @(
        @{
            targetType = "group"
            id = $group.Id
            isRegistrationRequired = $false
        }
    )
}

Update-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "TemporaryAccessPass" `
    -BodyParameter $tapParams

$user1 = Get-MgUser -Filter "displayName eq 'Break-glass-account-1'"

$tap = New-MgUserAuthenticationTemporaryAccessPassMethod `
    -UserId $user1.Id `
    -BodyParameter @{ isUsableOnce = $true; lifetimeInMinutes = 60 }

$tap.TemporaryAccessPass
```

<img width="988" height="680" alt="image" src="https://github.com/user-attachments/assets/fcf52195-416d-495e-8fe1-c12967eaaf6e" />

- Open a private browser window, go to the My Security Info page, and sign in as Break-glass-account-1 using the Temporary Access Pass.

<img width="844" height="861" alt="image" src="https://github.com/user-attachments/assets/8e2c96a8-dd1b-4daa-b899-a59c966f1e96" />

- After signing in, select Add sign-in method and choose Passkey to start registering the security key.

<img width="848" height="862" alt="image" src="https://github.com/user-attachments/assets/2216e223-cac6-413a-b153-0d5c94f05d22" />

- When the Windows Security pop-up for the security key setup appears, follow its prompts to continue the registration.
  
<img width="435" height="482" alt="image" src="https://github.com/user-attachments/assets/d9b0d3b9-45d2-4f06-b4f8-a460d04eefbe" />

- When the Windows Security pop-up appears, choose Security key and follow the prompts to set up the key.
  
<img width="847" height="860" alt="image" src="https://github.com/user-attachments/assets/1ccf8d60-528e-433f-8e59-cb0b1e2326a5" />

- Continue the setup, then insert the security key into a USB port when Windows prompts for it.

<img width="456" height="322" alt="image" src="https://github.com/user-attachments/assets/59d65644-226a-4a35-a702-d261a64ca4f4" />

<img width="453" height="222" alt="image" src="https://github.com/user-attachments/assets/dd05d86c-7c4e-4d3b-a910-ec2b7bc058ca" />

- Create a PIN for the security key when prompted, then touch the key to confirm you are physically present, so the key cannot be used silently by malware.

<img width="453" height="322" alt="image" src="https://github.com/user-attachments/assets/05b412c3-b503-4c40-93de-abb8e6997981" />

<img width="453" height="197" alt="image" src="https://github.com/user-attachments/assets/c3865322-2a40-4911-b582-f78faff1e1e3" />

- Skip the prompt to add another verification method, since the break-glass account relies on the FIDO2 key alone.
  
<img width="929" height="945" alt="image" src="https://github.com/user-attachments/assets/8906d928-800e-432e-b477-52ec5957ae9b" />

- Name the passkey and confirm that it appears in the list of sign-in methods. 

<img width="845" height="858" alt="image" src="https://github.com/user-attachments/assets/9bcf7733-192b-467d-b8d0-5d381ead2e61" />

- A message confirms that the passkey was created.

<img width="437" height="409" alt="image" src="https://github.com/user-attachments/assets/25bdcdf2-9e86-4483-90d9-7363773482fb" />

- Run Get-MgUserAuthenticationFido2Method for the break-glass account to confirm that the security key is registered.

```powershell
Get-MgUserAuthenticationFido2Method -UserId $user1.Id
```

<img width="987" height="206" alt="image" src="https://github.com/user-attachments/assets/3f6bf170-ecf4-4df6-96f9-8cd175d76884" />


### Configure certificate-based authentication
- **The certificate request and issuance process will not be demonstrated, as the PKI configuration and certificate enrollment steps are outside the scope of this lab.**
- 
- Create a PKI container in the Entra trust store with Microsoft Graph.

```powershell
$body = @{ displayName = "KAM-PKI" } | ConvertTo-Json
$pki = Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/directory/publicKeyInfrastructure/certificateBasedAuthConfigurations" -Body $body -ContentType "application/json"
$pki
```

<img width="1997" height="358" alt="image" src="https://github.com/user-attachments/assets/dc7eb17d-158b-4a4e-a153-b92ada7f76d3" />

- Add the root CA certificate to the PKI container with Microsoft Graph.
  
```powershell
$cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("Path")
$body = @{
certificateAuthorityType = "root"
certificate = [System.Convert]::ToBase64String($cer.RawData)
expirationDateTime = $cer.NotAfter.ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
} | ConvertTo-Json
try {
Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/directory/publicKeyInfrastructure/certificateBasedAuthConfigurations/$($pki.id)/certificateAuthorities" -Body $body -ContentType "application/json"
} catch {
$_.ErrorDetails.Message
}
```

<img width="2207" height="585" alt="image" src="https://github.com/user-attachments/assets/f2ff0a09-cba6-4d0e-a44a-388c48f61ba4" />

- Add the issuing CA certificate to the PKI container as an intermediate with Microsoft Graph.

```powershell
$cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("Path")
$body = @{
certificateAuthorityType = "intermediate"
certificate = [System.Convert]::ToBase64String($cer.RawData)

} | ConvertTo-Json
try {
Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/directory/publicKeyInfrastructure/certificateBasedAuthConfigurations/$($pki.id)/certificateAuthorities" -Body $body -ContentType "application/json"
} catch {
$_.ErrorDetails.Message
}
```
<img width="2211" height="587" alt="image" src="https://github.com/user-attachments/assets/0a71f781-9746-44bd-ba8a-11b4df66e2c0" />

- Open Public key infrastructure in the Entra admin center and confirm the KAM-PKI container exists.

<img width="2552" height="1007" alt="image" src="https://github.com/user-attachments/assets/65655f72-77c2-4cc1-a4f1-9108e4c5ff58" />

- Open the KAM-PKI container and confirm it holds the root CA and the issuing CA.

<img width="2277" height="387" alt="image" src="https://github.com/user-attachments/assets/d856e485-e058-42c9-a50c-cb688763b398" />

- Create the Break-glass-CBA group and add Break-glass-account-2 to it.

```powershell
$group = New-MgGroup -DisplayName "Break-glass-CBA" -MailEnabled:$false -MailNickname "break-glass-cba" -SecurityEnabled:$true
$userId = (Get-MgUser -Filter "DisplayName eq 'Break-glass-account-2'").Id
New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $userId
Get-MgGroupMember -GroupId $group.Id
```

<img width="1368" height="272" alt="image" src="https://github.com/user-attachments/assets/382794f2-e33f-445e-a500-154b4d8d6c12" />

- Enable the certificate-based authentication method for the Break-glass-CBA group with Microsoft Graph.

```powershell
$groupName = "Break-glass-CBA"
$groupId = (Get-MgGroup -Filter "displayName eq '$groupName'").Id
$body = @{
    "@odata.type" = "#microsoft.graph.x509CertificateAuthenticationMethodConfiguration"
    id    = "X509Certificate"
    state = "enabled"
    certificateUserBindings = @(
        @{
            x509CertificateField = "PrincipalName"
            userProperty         = "userPrincipalName"
            priority             = 1
        }
    )
    authenticationModeConfiguration = @{
        x509CertificateAuthenticationDefaultMode = "x509CertificateMultiFactor"
    }
    includeTargets = @(
        @{
            targetType             = "group"
            id                     = $groupId
            isRegistrationRequired = $false
        }
    )
} | ConvertTo-Json -Depth 5
try {
    Invoke-MgGraphRequest -Method PATCH -Uri "https://graph.microsoft.com/v1.0/policies/authenticationMethodsPolicy/authenticationMethodConfigurations/x509Certificate" -Body $body -ContentType "application/json"
} catch {
    $_.ErrorDetails.Message
}
```
<img width="1948" height="626" alt="image" src="https://github.com/user-attachments/assets/7de6a9ad-5c54-4df2-9a35-4243b8e90d8a" />

- Open myapps.microsoft.com in a private browser window and enter the UPN of Break-glass-account-2.

<img width="1267" height="810" alt="image" src="https://github.com/user-attachments/assets/9a78d655-660d-44df-b99d-b39d11845975" />

- Select Use a certificate or smart card on the sign-in page.

<img width="1270" height="802" alt="image" src="https://github.com/user-attachments/assets/61c235c7-a706-423e-8d34-b9017d595b91" />

- Choose the Break-glass-account-2 certificate in the certificate picker.

<img width="1271" height="807" alt="image" src="https://github.com/user-attachments/assets/93f5edbf-ffe6-4515-bf40-ed8ce89162ed" />

- Successfully signed in to the My Apps portal using the certificate issued to Break-glass-account-2, confirming that certificate-based authentication (CBA) is working correctly.

<img width="1271" height="808" alt="image" src="https://github.com/user-attachments/assets/d2631f21-e519-4884-a35b-0b1023b221cb" />


## Step 6: Create the named administrator account and assign roles

- Create the adm-kamil admin account, create the administrators group, and add the account to it with Microsoft Graph.

<img width="843" height="362" alt="image" src="https://github.com/user-attachments/assets/a634a5e5-2dfd-4027-a286-5e52d0bf1623" />

- Assign the Global Reader role to adm-kamil as an active assignment.

```powershell
$gr = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayname eq 'Global Reader'"
New-MgRoleManagementDirectoryRoleAssignment -PrincipalId $admin.Id -RoleDefinitionId $gr.Id -DirectoryScopeId "/"
```

<img width="1692" height="326" alt="image" src="https://github.com/user-attachments/assets/316d5f40-66e6-4f57-bbbf-72663b121227" />

- Check the available subscriptions, set the usage location, and assign the AAD_PREMIUM_P2 license to adm-kamil

```powershell
Get-MgSubscribedSku | Format-List
$sku = Get-MgSubscribedSku | Where {$_.SkuPartNumber -eq "AAD_PREMIUM_P2"}
Update-MgUser -UserId $admin.Id -UsageLocation "PL"
Set-MgUserLicense -UserId $admin.Id -AddLicenses @{SkuId = $sku.SkuId} -RemoveLicenses @()
```

<img width="1067" height="785" alt="image" src="https://github.com/user-attachments/assets/cd35ad64-89c0-4719-bbea-9471a413d55b" />

- Assign eligible directory roles to the user. The roles selected are illustrative; apply the principle of least privilege and assign administrators only the roles necessary for their responsibilities.

```powershell
$eligible = "User Administrator","Groups Administrator","Authentication Administrator","Authentication Policy Administrator","Conditional Access Administrator"
$eligible | ForEach-Object { Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayname eq '$_'" | select id }
$eligible | ForEach-Object {
     $roleName = $_

     $role = Get-MgRoleManagementDirectoryRoleDefinition `
         -Filter "displayName eq '$roleName'"

     New-MgRoleManagementDirectoryRoleEligibilityScheduleRequest `
         -Action "adminAssign" `
         -RoleDefinitionId $role.Id `
         -DirectoryScopeId "/" `
         -PrincipalId $admin.Id `
         -Justification "Eligible assignment for '$($admin.DisplayName)'" `
         -ScheduleInfo @{
             StartDateTime = (Get-Date).ToUniversalTime()
             Expiration = @{
                 Type = "afterDateTime"
                 EndDateTime = (Get-Date).ToUniversalTime().AddDays(365)
             }
         }
 }
```

<img width="2232" height="848" alt="image" src="https://github.com/user-attachments/assets/94e47b93-5b16-4005-b926-1eacafb38c71" />

- To verify it, sign in to the Entra portal, navigate to ID Governance, Privileged Identity Management, and My roles.

<img width="2551" height="803" alt="image" src="https://github.com/user-attachments/assets/ce252666-4e09-4c4d-a196-3153d4095a7f" />

- Activate access to the first role.
  
<img width="586" height="498" alt="image" src="https://github.com/user-attachments/assets/73d2c97e-d2d1-4841-bf66-a62f5473e99a" />

- Role activation settings should follow the Zero Trust principle, with appropriate security controls such as approval, MFA, and justification based on the role's privilege level and associated risk.
- Sign in to the Microsoft Entra admin center as Global Administrator, navigate to Roles & admins, select the desired role, then open Role settings, Edit to configure its activation requirements.

<img width="2557" height="491" alt="image" src="https://github.com/user-attachments/assets/8b0d2d3d-7fb1-436b-a8fd-fcc4ffc667d9" />

- Under Activation, enable Require justification and Require approval, select the designated approver or approver group, then click Next. As a common practice, require administrators (and users) to provide a change or service ticket number explaining why the role needs to be activated, ensuring traceability and accountability.

<img width="652" height="1215" alt="image" src="https://github.com/user-attachments/assets/7ba76ba7-17e0-488c-b214-ab8b2f5eec05" />

- Leave the default configuration.

<img width="690" height="1221" alt="image" src="https://github.com/user-attachments/assets/99bd2223-1733-4a17-8546-15a043ac8d5b" />

- Under Notification, you can additionally configure recipients who should be notified when someone activates a role or when an eligible role assignment is granted to a user. This provides additional visibility and helps ensure that relevant security or IAM teams are aware of privileged access changes.

<img width="1622" height="1227" alt="image" src="https://github.com/user-attachments/assets/479e09d5-c6ea-4157-89ef-4eb223c9afcf" />

- Activate the role. You will now be required to provide a ticket number for the activation request.

<img width="587" height="498" alt="image" src="https://github.com/user-attachments/assets/4b1917c4-ef50-4981-85dc-4863619d5b9e" />

- The approver can use the link in the email or go to PIM in the Entra portal to approve the request.

<img width="2556" height="791" alt="image" src="https://github.com/user-attachments/assets/a8aeecd2-d2d9-4094-af11-4e2a28b363c1" />

- A short justification is also required before approving the request.

<img width="367" height="687" alt="image" src="https://github.com/user-attachments/assets/f89d97cf-54f1-4019-8aba-3247b45dc234" />

## Step 7: Exclude the break-glass group from Conditional Access policies

### The Entra admin center does not support excluding accounts or groups from multiple Conditional Access policies simultaneously, so we will use Microsoft Graph PowerShell to automate this process.
  
