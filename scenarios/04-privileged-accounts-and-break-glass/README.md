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

