# Active Directory Lab — Commands Reference

This document contains the primary Windows and PowerShell commands used to configure, verify, and troubleshoot the Active Directory lab.

> **Environment:** Windows Server 2022 Domain Controller + Windows client
> **Domain:** `corp.drexzw.local`

---

# 1. Network and DNS Troubleshooting

## Display IP Configuration

```powershell
ipconfig
```

Displays the current IP configuration.

For more detailed information:

```powershell
ipconfig /all
```

Useful for checking:

* IP address
* Subnet mask
* Default gateway
* DNS servers
* DHCP configuration

---

## Test Network Connectivity

```powershell
ping <IP-address>
```

Used to determine whether the client can communicate with another system.

Example:

```powershell
ping <domain-controller-ip>
```

---

## Test DNS Resolution

```powershell
nslookup corp.drexzw.local
```

Used to verify that the client can resolve the Active Directory domain through DNS.

A specific host can also be queried:

```powershell
nslookup <domain-controller-hostname>
```

---

# 2. Domain and Authentication Verification

## Display Current User

```powershell
whoami
```

Shows the account currently being used.

For a domain account, the result should identify the domain and username.

Example:

```text
corp\jsmith
```

---

## Display Domain Information

```powershell
systeminfo
```

Useful information includes the computer name, Windows version, and domain/workgroup information.

---

## Display Computer Domain Membership

```powershell
(Get-CimInstance Win32_ComputerSystem).Domain
```

This can be used to verify which domain the workstation belongs to.

---

## Display Computer System Information

```powershell
Get-CimInstance Win32_ComputerSystem
```

Provides information about:

* Computer name
* Domain
* Logged-on user
* Manufacturer
* Model
* System configuration

---

# 3. Group Policy

## Refresh Group Policy

```powershell
gpupdate /force
```

Forces the client to refresh both computer and user Group Policy settings.

---

## Display Applied Group Policy

```powershell
gpresult /r
```

Displays a summary of Group Policy settings applied to the current user and computer.

---

## Generate Detailed Group Policy Report

```powershell
gpresult /h gpresult.html
```

Creates an HTML report containing detailed Group Policy information.

---

# 4. Active Directory PowerShell

The Active Directory PowerShell module can be used to query and manage objects from the Domain Controller.

## View Active Directory Users

```powershell
Get-ADUser -Filter *
```

Lists Active Directory user accounts.

---

## View a Specific User

```powershell
Get-ADUser jsmith
```

Displays information about the specified user.

For additional properties:

```powershell
Get-ADUser jsmith -Properties *
```

---

## View Active Directory Groups

```powershell
Get-ADGroup -Filter *
```

Lists Active Directory groups.

---

## View Group Membership

```powershell
Get-ADGroupMember "GG-IT-Users"
```

Displays the members of the IT security group.

---

## Find a Computer Account

```powershell
Get-ADComputer -Filter *
```

Lists computer accounts stored in Active Directory.

A specific computer can be queried with:

```powershell
Get-ADComputer "<computer-name>"
```

---

# 5. Account Lockout Investigation

## Check a User's Account Status

```powershell
Get-ADUser <username> -Properties LockedOut
```

The `LockedOut` property indicates whether the account is currently locked.

Example:

```powershell
Get-ADUser sarah -Properties LockedOut
```

---

## Unlock a User Account

```powershell
Unlock-ADAccount -Identity <username>
```

Example:

```powershell
Unlock-ADAccount -Identity sarah
```

This unlocks the specified Active Directory account.

---

## Verify the Account Is Unlocked

```powershell
Get-ADUser <username> -Properties LockedOut
```

The expected result after unlocking should show:

```text
LockedOut : False
```

---

# 6. Password and Account Policy

## View Domain Password Policy

```powershell
Get-ADDefaultDomainPasswordPolicy
```

This displays the domain's default password and account lockout settings.

Important properties include:

* `LockoutThreshold`
* `LockoutDuration`
* `LockoutObservationWindow`
* `MaxPasswordAge`
* `MinPasswordLength`
* `PasswordHistoryCount`

---

## Check Lockout Threshold

```powershell
Get-ADDefaultDomainPasswordPolicy |
Select-Object LockoutThreshold
```

Useful for confirming how many failed authentication attempts are required before an account is locked.

---

# 7. Department Folders and NTFS Permissions

Commands used to create department folders and apply group-based NTFS permissions on the Domain Controller. Evidence for each step is in `screenshots/07-department-permissions/`.

> **Note:** These are NTFS permissions on the Domain Controller, verified with `icacls` and PowerShell on the server. The folders are shared over SMB in section 8.

## Create the Department Folders

```powershell
New-Item -Path "C:\CompanyData\IT" -ItemType Directory
New-Item -Path "C:\CompanyData\HR" -ItemType Directory
New-Item -Path "C:\CompanyData\Finance" -ItemType Directory
New-Item -Path "C:\CompanyData\Sales" -ItemType Directory
New-Item -Path "C:\CompanyData\Management" -ItemType Directory
```

Creates one folder per department under `C:\CompanyData`.

**Evidence:** `screenshots/07-department-permissions/01-companydata-folders-created.png`

---

## List the Department Security Groups and Folders

```powershell
Get-ADGroup -Filter 'Name -like "*GG*"' | Select-Object Name
Get-ChildItem "C:\CompanyData" -Directory | Select-Object Name
```

Filtering on `GG` returns only the lab's six security groups instead of the 50+ built-in groups. The second command lists the department folders.

**Evidence:** `screenshots/07-department-permissions/02-security-groups-and-folders-verified.png`

---

## Capture the Baseline ACL

```powershell
Get-Acl "C:\CompanyData" | Format-List
Get-Acl "C:\CompanyData\IT" | Format-List
```

Shows the owner and access entries before any group permissions are applied. Run this before changing permissions so there is a record of the starting state.

**Evidence:** `screenshots/07-department-permissions/03-baseline-acl-before-group-permissions.png`

---

## Grant Department Permissions

```powershell
icacls "C:\CompanyData\IT" /grant "DREXZW\GG-IT-Users:(OI)(CI)M"
icacls "C:\CompanyData\IT" /grant "DREXZW\GG-IT-Admins:(OI)(CI)F"
icacls "C:\CompanyData\HR" /grant "DREXZW\GG-HR-Users:(OI)(CI)M"
icacls "C:\CompanyData\Finance" /grant "DREXZW\GG-Finance-Users:(OI)(CI)M"
icacls "C:\CompanyData\Sales" /grant "DREXZW\GG-Sales-Users:(OI)(CI)M"
icacls "C:\CompanyData\Management" /grant "DREXZW\GG-Management-Users:(OI)(CI)M"
```

Grants each department group Modify on its own folder, and `GG-IT-Admins` Full Control on the IT folder. `(OI)(CI)` makes the permission apply to files and subfolders.

> **Important:** Use the NetBIOS domain name (`DREXZW\`) in the principal, not `CORP\`. The first attempt used `CORP\GG-IT-Users` and failed (**not pictured**). The NetBIOS name is shown in `screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png`. See `troubleshooting.md`.

**Evidence:** `screenshots/07-department-permissions/04-icacls-department-grants.png`

---

## Verify Group Membership

```powershell
Get-ADGroupMember "GG-IT-Users" | Select-Object Name, SamAccountName
Get-ADGroupMember "GG-IT-Admins" | Select-Object Name, SamAccountName
Get-ADGroupMember "GG-HR-Users" | Select-Object Name, SamAccountName
Get-ADGroupMember "GG-Finance-Users" | Select-Object Name, SamAccountName
Get-ADGroupMember "GG-Sales-Users" | Select-Object Name, SamAccountName
Get-ADGroupMember "GG-Management-Users" | Select-Object Name, SamAccountName
```

Confirms which user is in each group. Each group contained one user in this lab.

**Evidence:** `screenshots/07-department-permissions/05-group-membership-verification.png`

---

## Review the Resulting ACLs

```powershell
icacls "C:\CompanyData\IT"
icacls "C:\CompanyData\HR"
icacls "C:\CompanyData\Finance"
icacls "C:\CompanyData\Sales"
icacls "C:\CompanyData\Management"
```

Displays the full ACL for each folder. After the grants, the output still showed inherited `BUILTIN\Users` entries (marked `(I)`) alongside the new group entries.

**Evidence:** `screenshots/07-department-permissions/06-icacls-before-inheritance-cleanup.png`

---

## Disable Inheritance and Remove BUILTIN\Users

Run for each of the five department folders. Disable inheritance first, then remove the entry.

```powershell
icacls "C:\CompanyData\IT" /inheritance:d
icacls "C:\CompanyData\HR" /inheritance:d
icacls "C:\CompanyData\Finance" /inheritance:d
icacls "C:\CompanyData\Sales" /inheritance:d
icacls "C:\CompanyData\Management" /inheritance:d

icacls "C:\CompanyData\IT" /remove "BUILTIN\Users"
icacls "C:\CompanyData\HR" /remove "BUILTIN\Users"
icacls "C:\CompanyData\Finance" /remove "BUILTIN\Users"
icacls "C:\CompanyData\Sales" /remove "BUILTIN\Users"
icacls "C:\CompanyData\Management" /remove "BUILTIN\Users"
```

`/inheritance:d` stops the folder from inheriting from `C:\CompanyData` and converts the existing inherited entries into explicit ones, so nothing is lost. `/remove` then deletes every `BUILTIN\Users` entry from the folder's ACL. Inherited entries cannot be removed while the folder is still inheriting, so the order matters.

Only the five department folders were changed. The parent `C:\CompanyData` was left as it was.

**Evidence:** `screenshots/07-department-permissions/07-inheritance-disabled-builtin-users-removed.png`

---

## Verify the Final ACLs

```powershell
icacls "C:\CompanyData\IT"
icacls "C:\CompanyData\HR"
icacls "C:\CompanyData\Finance"
icacls "C:\CompanyData\Sales"
icacls "C:\CompanyData\Management"
```

Expected result for each folder:

* No `BUILTIN\Users` entry
* No `(I)` (inherited) markers
* The department group with `(OI)(CI)(M)`; `GG-IT-Admins` with `(OI)(CI)(F)` on the IT folder
* `NT AUTHORITY\SYSTEM` and `BUILTIN\Administrators` with `(OI)(CI)(F)`
* `CREATOR OWNER` with `(OI)(CI)(IO)(F)`

**Evidence:** `screenshots/07-department-permissions/08-icacls-after-inheritance-cleanup.png`

---

## Reading icacls Output

| Code | Meaning |
| --- | --- |
| `F` | Full control |
| `M` | Modify |
| `RX` | Read and execute |
| `AD` | Add subdirectory (append data) |
| `WD` | Add file (write data) |
| `OI` | Object inherit: applies to files in the folder |
| `CI` | Container inherit: applies to subfolders |
| `IO` | Inherit only: applies to child items, not the folder itself |
| `I` | Permission is inherited from a parent folder |

---

# 8. SMB Share and Shared-Folder Access (INC-AD-002)

Commands used to publish `C:\CompanyData` as an SMB share and to troubleshoot a share-versus-NTFS permission problem for `sjohnson`. Evidence for each step is in `screenshots/08-smb-share-access/`.

> **Note:** The server-side commands were run on the Domain Controller. The client-side commands were run in a PowerShell session running as `sjohnson@corp.drexzw.local` on the Windows client. The client reached the share at `\\10.0.1.70\CompanyData`.

## Confirm the Domain and the User

```powershell
Get-ADDomain
Get-ADUser sjohnson
```

`Get-ADDomain` returns the domain details, including `NetBIOSName` (`DREXZW`), the name to use in `DOMAIN\principal` commands. `Get-ADUser` confirms `sjohnson` exists and is in the HR OU.

The `Get-ADDomain` command line itself had scrolled out of view in the screenshot; the domain properties are visible.

**Evidence:** `screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png`

---

## Confirm Group Membership and NTFS Permissions

```powershell
Get-ADGroup GG-HR-Users
Get-ADGroupMember GG-HR-Users
Get-ChildItem "C:\CompanyData"
icacls C:\CompanyData\HR
```

Confirms `GG-HR-Users` is a global security group containing `sjohnson`, that the department folders exist, and that the HR folder grants `GG-HR-Users` Modify. Run before testing the share so the NTFS layer is known to be correct.

**Evidence:** `screenshots/08-smb-share-access/02-hr-group-membership-and-ntfs-verified.png`

---

## Create the SMB Share

```powershell
New-SmbShare -Name "CompanyData" -Path "C:\CompanyData" -Description "Company Department File Share"
Get-SmbShare -Name CompanyData
```

Publishes `C:\CompanyData` as the `CompanyData` share and confirms it exists.

**Evidence:** `screenshots/08-smb-share-access/03-smb-share-created.png`

---

## Review and Set Share Permissions (the Fault)

```powershell
Get-SmbShareAccess -Name CompanyData
Grant-SmbShareAccess -Name "CompanyData" -AccountName "DREXZW\GG-HR-Users" -AccessRight Read -Force
Get-SmbShareAccess -Name CompanyData
```

The new share started with `Everyone` at Read. `GG-HR-Users` was then deliberately given **Read** at the share level to create the problem for this ticket, while its NTFS permission remained Modify.

**Evidence:** `screenshots/08-smb-share-access/04-smb-share-permissions-hr-read-only.png`

---

## Reproduce the Problem from the Client

Run on the client as `sjohnson`:

```powershell
whoami
Test-Path "\\10.0.1.70\CompanyData"
New-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt" -ItemType File
```

`whoami` confirms the identity (`drexzw\sjohnson`). `Test-Path` returning `True` shows the share is reachable, so connectivity and authentication are working. `New-Item` returned "Access to the path ... is denied," which narrows the problem to a permission on the write.

**Evidence:** `screenshots/08-smb-share-access/05-sjohnson-access-denied-creating-file.png`

> **Capture note:** this screenshot was taken after the fix and placed here in incident order. See `tickets/shared-folder-access-sarah-johnson.md`.

---

## Fix the Share Permission

```powershell
Grant-SmbShareAccess -Name "CompanyData" -AccountName "DREXZW\GG-HR-Users" -AccessRight Change -Force
Get-SmbShare -Name CompanyData
Get-SmbShareAccess -Name CompanyData
```

Changes `GG-HR-Users` from Read to Change at the share layer. `Everyone` stays at Read. `Get-SmbShareAccess` confirms the result.

**Evidence:** `screenshots/08-smb-share-access/06-smb-share-permission-changed-to-change.png`

---

## Retest from the Client

Run on the client as `sjohnson`:

```powershell
whoami
Test-Path "\\10.0.1.70\CompanyData"
Get-ChildItem "\\10.0.1.70\CompanyData"
Get-ChildItem "\\10.0.1.70\CompanyData\HR"
New-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt" -ItemType File
Remove-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt"
```

The test file was created successfully (a 0-byte `sjohnson-test.txt`), confirming the fix. The test file was then removed.

> **Note on the red error in the screenshot:** the first `Remove-Item` attempt included `-ItemType File`. `Remove-Item` does not have that parameter, so PowerShell returned a parameter error. That is a command-syntax mistake and unrelated to permissions. Re-running `Remove-Item` without it returned no error. The deletion was not listed afterwards (**not pictured**).

**Evidence:** `screenshots/08-smb-share-access/07-sjohnson-retest-file-creation-succeeds.png`

---

## Share Permissions Versus NTFS Permissions

| Layer | Where it is set | Checked with |
| --- | --- | --- |
| Share permissions | On the share itself (SMB) | `Get-SmbShareAccess` |
| NTFS permissions | On the folder (file system) | `icacls`, `Get-Acl` |

Over the network, a user gets the **more restrictive** of the two. A user with Modify on NTFS but Read on the share can read but not write. When a user can reach a share but cannot write, check both layers.

---

# 9. General Troubleshooting Workflow

A useful troubleshooting sequence for this lab is:

```text
1. Identify the reported problem
        |
        v
2. Determine whether the issue is client-side or server-side
        |
        v
3. Check network connectivity
        |
        v
4. Check DNS resolution
        |
        v
5. Verify domain membership
        |
        v
6. Verify user/account state
        |
        v
7. Check Group Policy if relevant
        |
        v
8. Reproduce the issue when safe
        |
        v
9. Apply the appropriate fix
        |
        v
10. Test again
        |
        v
11. Document the resolution
```

This workflow was particularly useful when troubleshooting domain authentication and account lockout behavior.

---

# Important Notes

Commands should be run with the appropriate permissions.

Some Active Directory commands require the Active Directory PowerShell module and administrative privileges.

For production environments, destructive or account-management commands should be verified against the organization's procedures before execution.
