# Active Directory Lab — Troubleshooting

This document records the troubleshooting performed during the Active Directory lab.

The purpose was to practice diagnosing Windows domain problems from the perspective of an IT Support / Help Desk technician rather than simply following configuration instructions.

---

# 1. Client Could Not Be Reliably Verified Against the Domain

## Symptoms

The Windows client did not initially provide enough evidence to confirm that it was communicating correctly with the Active Directory environment.

The issue required checking the client configuration rather than immediately changing Active Directory settings.

## Investigation

The client configuration was checked using:

```powershell
ipconfig /all
```

The Domain Controller's network information and DNS configuration were then compared against the client's configuration.

DNS resolution was tested with:

```powershell
nslookup corp.drexzw.local
```

Connectivity to the Domain Controller was also tested.

## Resolution

The client was configured to communicate with the Active Directory DNS environment and was subsequently able to resolve the domain correctly.

## Lesson

Active Directory relies heavily on DNS.

When a domain client is having problems joining or communicating with a domain, DNS should be one of the first areas checked.

---

# 2. Verifying Domain Membership

## Symptoms

The client needed to be verified as a member of the Active Directory domain rather than a local workgroup.

## Investigation

The client's domain information was checked using:

```powershell
systeminfo | findstr /B /C:"Domain"
```

The system information was also reviewed.

## Resolution

The client was successfully joined to:

```text
corp.drexzw.local
```

The result was then verified from the client.

## Lesson

Do not assume that a domain join succeeded simply because the join process completed.

Verify the final state from the client.

---

# 3. Active Directory Computer Object Visibility

## Symptoms

During the lab, the workstation's computer object needed to be confirmed in the expected Active Directory Users and Computers location (the Workstations OU) rather than simply assumed.

## Investigation

The client-side domain membership was checked first.

The Active Directory Users and Computers console was then reviewed for the computer object, first with the object filter limited to Groups only (showing no computer), then again with Computers included in the filter (showing `AD-CLIENT` present in the Workstations OU).

This was cross-checked from the Domain Controller using PowerShell:

```powershell
Get-ADComputer -Filter 'Name -eq "AD-CLIENT"' -Properties DistinguishedName | Select-Object Name, DistinguishedName
Get-ADComputer -SearchBase "OU=Workstations,DC=corp,DC=drexzw,DC=local" -Filter * | Select-Object Name, DistinguishedName
```

## Resolution

The issue was investigated from both sides:

```text
Client
  |
  +-- Domain membership
  +-- DNS configuration
  +-- Computer identity
  |
  v
Domain Controller
  |
  +-- Active Directory Users and Computers (GUI filter)
  +-- Get-ADComputer (PowerShell)
```

This prevented the troubleshooting process from focusing on only one system.

## Lesson

When confirming an AD object's location, verify the client state and cross-check with more than one tool (GUI filter and PowerShell) before assuming an object is missing or misplaced.

---

# 4. Domain User Authentication

## Symptoms

A domain user needed to be tested on the Windows client.

The authentication process required distinguishing between local credentials and domain credentials.

## Investigation

The current authentication context was checked with:

```powershell
whoami
```

The domain membership was also verified.

The user account was confirmed in Active Directory.

## Resolution

The domain account was successfully used for authentication and the resulting session was verified.

## Lesson

When troubleshooting Windows authentication, always establish:

1. Which account is being used
2. Whether the account is local or domain-based
3. Which domain the computer belongs to
4. Whether the Domain Controller is reachable
5. Whether the user account exists and is enabled

---

# 5. Group Policy Did Not Immediately Appear to Apply

## Symptoms

A Group Policy configuration was created and linked to the workstation organizational unit, but the expected behavior was not immediately visible on the client.

## Investigation

First, the policy configuration and OU linkage were checked in Group Policy Management, and cross-checked from PowerShell with `Get-ADOrganizationalUnit` (confirming the `LinkedGroupPolicyObjects` attribute on the Workstations OU).

The client was then forced to refresh Group Policy:

```powershell
gpupdate /force
```

## Resolution

The client was refreshed and the resulting Group Policy application was verified.

## Lesson

Creating or linking a GPO does not mean the desired result should immediately be assumed to be present.

A better troubleshooting approach is:

```text
GPO exists
   ↓
GPO linked correctly (verified via GUI and Get-ADOrganizationalUnit)
   ↓
Client belongs to correct OU
   ↓
Group Policy refreshed
   ↓
Policy application verified
   ↓
Expected behavior tested
```

---

# 6. Account Lockout Investigation

## Scenario

A simulated Help Desk ticket was created for the user Sarah Johnson (`sjohnson`), whose Active Directory account became locked after repeated unsuccessful authentication attempts.

The scenario was based on a common Windows support problem.

## Investigation

The configured account lockout policy was checked in the Group Policy Management Editor (Default Domain Policy → Account Lockout Policy):

* Lockout threshold: 5 invalid logon attempts
* Lockout duration: 5 minutes
* Reset lockout counter after: 1 minute

The user's baseline account state was reviewed in Active Directory Users and Computers (Properties → Account tab) before reproducing the issue.

## Testing

The lockout was reproduced by repeatedly running the following from the client with an intentionally incorrect password:

```powershell
runas /user:DREXZW\sjohnson powershell.exe
```

After the configured threshold was exceeded, Windows returned:

```text
1909: The referenced account is currently locked out and may not be logged on to.
```

Reopening the account in Active Directory Users and Computers confirmed the lockout, showing:

> "This account is currently locked out on this Active Directory Domain Controller."

## Resolution

The account was unlocked by checking the **Unlock account** checkbox on the Account tab in Active Directory Users and Computers.

Authentication was then re-tested:

```powershell
runas /user:DREXZW\sjohnson powershell.exe
whoami
```

which returned:

```text
drexzw\sjohnson
```

confirming the account was unlocked and functional.

## Lesson

An account lockout should not automatically be treated as a simple "unlock the account" task.

A support technician should also consider why the account became locked.

Possible causes include:

* Incorrect password attempts
* Saved credentials
* Applications using an old password
* Services running under the user's credentials
* Mapped drives
* Repeated authentication attempts from another device

In this lab, the lockout was intentionally reproduced using repeated `runas` attempts to understand and verify the configured policy.

> **Note:** This run of the scenario was performed entirely through the GUI (Group Policy Management and Active Directory Users and Computers) plus `runas`/`whoami` for testing. The equivalent PowerShell cmdlets (`Get-ADUser -Properties LockedOut`, `Unlock-ADAccount`) are valid alternatives and are documented in `commands.md` for reference, but were not used in this specific run.

---

# 7. Troubleshooting Method Used

The overall troubleshooting process followed a layered approach.

## Layer 1 — User / Symptom

Determine exactly what the user is experiencing.

Example:

```text
"My account is locked and I cannot sign in."
```

## Layer 2 — Client

Check:

* Network configuration
* DNS
* Domain membership
* Current user
* Group Policy

Useful commands:

```powershell
ipconfig /all
nslookup corp.drexzw.local
whoami
systeminfo | findstr /B /C:"Domain"
```

## Layer 3 — Active Directory

Check:

* User exists
* User is enabled
* Account is locked
* Group membership
* Computer object
* Domain policy

This can be checked via the GUI (Active Directory Users and Computers, Group Policy Management) or PowerShell:

```powershell
Get-ADComputer -Filter *
Get-ADOrganizationalUnit -Identity "OU=Workstations,DC=corp,DC=drexzw,DC=local"
Get-ADUser <username> -Properties LockedOut
Get-ADDefaultDomainPasswordPolicy
```

## Layer 4 — Remediation

Make the smallest appropriate change.

Examples:

* Check the **Unlock account** box in Active Directory Users and Computers, or run `Unlock-ADAccount -Identity <username>`
* Refresh policy with `gpupdate /force`

## Layer 5 — Validation

Never stop immediately after making the change.

Test again and confirm the original problem has actually been resolved.

---

# Key Troubleshooting Lessons

### 1. Start with evidence

Do not immediately change settings.

First determine what is actually happening.

### 2. DNS is fundamental to Active Directory

If DNS is incorrect, domain joining and authentication can fail even when the Domain Controller itself is functioning.

### 3. Verify from both sides

When troubleshooting domain problems, check both:

```text
Windows Client
        ↕
Domain Controller
```

### 4. Group Policy requires verification

A GPO being created is not the same thing as the GPO being successfully applied.

### 5. Reproduce problems when appropriate

The account-lockout scenario was intentionally reproduced in the lab so the configured policy could be observed rather than simply assumed.

### 6. Always validate the fix

A successful command does not necessarily mean the user's problem is fixed.

The final step should always be to reproduce the original failure condition and confirm that it has been resolved.

---

# Related Documentation

Account-lockout incident:

```text
tickets/account-lockout-sjohnson.md
```

Command reference:

```text
commands.md
```

Evidence:

```text
screenshots/
```

---

# Future Troubleshooting Scenarios

The next planned extension of this lab is NTFS and file-share permissions.

Future scenarios can include:

* User can access a share but cannot open a file
* User has incorrect NTFS permissions
* Share permissions conflict with NTFS permissions
* Department users cannot access the correct folder
* User receives "Access Denied"
* Group membership changes do not immediately appear in the user's session

These scenarios will extend the current Active Directory environment into more realistic Windows Help Desk permission troubleshooting.
