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
(Get-CimInstance Win32_ComputerSystem).Domain
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

During the lab, the workstation's computer object was not immediately visible in the expected Active Directory Users and Computers location.

This created a useful troubleshooting scenario because the client could not simply be assumed to be missing from Active Directory.

## Investigation

The client-side domain membership was checked first.

The Active Directory Users and Computers console was then reviewed for the computer object.

PowerShell could also be used to search computer accounts:

```powershell
Get-ADComputer -Filter *
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
  +-- Active Directory Users and Computers
  +-- Computer object
```

This prevented the troubleshooting process from focusing on only one system.

## Lesson

When an AD object appears to be missing, verify the client state before making additional changes to Active Directory.

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

First, the policy configuration and OU linkage were checked in Group Policy Management.

The client was then forced to refresh Group Policy:

```powershell
gpupdate /force
```

The resulting policies were checked with:

```powershell
gpresult /r
```

A detailed report can also be generated with:

```powershell
gpresult /h gpresult.html
```

## Resolution

The client was refreshed and the resulting Group Policy application was verified.

## Lesson

Creating or linking a GPO does not mean the desired result should immediately be assumed to be present.

A better troubleshooting approach is:

```text
GPO exists
   ↓
GPO linked correctly
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

A simulated Help Desk ticket was created for a user whose Active Directory account had become locked after repeated unsuccessful authentication attempts.

The scenario was based on a common Windows support problem.

## Investigation

The configured account lockout policy was checked using:

```powershell
Get-ADDefaultDomainPasswordPolicy
```

The relevant lockout settings included:

* Lockout threshold
* Lockout duration
* Lockout observation window

The user's account state could then be checked with:

```powershell
Get-ADUser <username> -Properties LockedOut
```

## Testing

The lockout behavior was reproduced in the lab by generating unsuccessful authentication attempts.

This demonstrated that the configured policy could cause an account to become locked after the required number of failed attempts.

## Resolution

After confirming that the account was locked, the account could be unlocked by an administrator using:

```powershell
Unlock-ADAccount -Identity <username>
```

The account state was then verified again:

```powershell
Get-ADUser <username> -Properties LockedOut
```

The expected state after remediation was:

```text
LockedOut : False
```

The user was then able to authenticate successfully again.

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

In this lab, the lockout was intentionally reproduced to understand and verify the configured policy.

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
gpresult /r
```

## Layer 3 — Active Directory

Check:

* User exists
* User is enabled
* Account is locked
* Group membership
* Computer object
* Domain policy

Useful commands:

```powershell
Get-ADUser <username> -Properties LockedOut
Get-ADGroupMember "GG-IT-Users"
Get-ADComputer -Filter *
Get-ADDefaultDomainPasswordPolicy
```

## Layer 4 — Remediation

Make the smallest appropriate change.

Examples:

```powershell
Unlock-ADAccount -Identity <username>
```

or refresh policy:

```powershell
gpupdate /force
```

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
tickets/account-lockout-sarah-johnson.md
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
