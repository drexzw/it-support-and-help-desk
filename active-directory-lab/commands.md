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

# 7. General Troubleshooting Workflow

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
