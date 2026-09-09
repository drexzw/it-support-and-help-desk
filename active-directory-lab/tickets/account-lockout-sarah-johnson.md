# Help Desk Ticket — Account Lockout

**Ticket ID:** INC-AD-001
**Status:** Resolved
**Priority:** Medium
**Category:** Account / Authentication
**Environment:** Active Directory Lab
**Domain:** `corp.drexzw.local`
**Affected User:** Sarah Johnson
**Affected Account:** `sarah`
**Technician:** IT Support Lab
**Date:** September 2026

---

## 1. User Report

**User:** Sarah Johnson

**Issue:**

Sarah reported that she was unable to sign in to her Windows workstation because her domain account had been locked.

**User-facing symptom:**

> "I can't sign in. My account says it has been locked."

---

## 2. Initial Assessment

The issue appeared to be an Active Directory account lockout.

Before making changes, the account lockout policy was reviewed to determine the conditions under which an account becomes locked.

The Domain Controller was checked to verify the configured account lockout settings.

Command used:

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Relevant settings included:

* Lockout threshold
* Lockout duration
* Lockout observation window

---

## 3. Troubleshooting

### Step 1 — Verify the User Account

The user's Active Directory account was checked to confirm that the account existed.

```powershell
Get-ADUser sarah
```

The account was present in Active Directory.

---

### Step 2 — Check Account Lockout Status

The `LockedOut` property was checked:

```powershell
Get-ADUser sarah -Properties LockedOut
```

The account was confirmed to be locked.

Example result:

```text
LockedOut : True
```

---

### Step 3 — Review Lockout Policy

The domain's account lockout configuration was reviewed:

```powershell
Get-ADDefaultDomainPasswordPolicy
```

The configured lockout threshold explained why repeated unsuccessful authentication attempts could result in the account becoming locked.

---

### Step 4 — Reproduce the Issue

The account-lockout behavior was intentionally reproduced in the lab by generating unsuccessful authentication attempts.

This confirmed that the configured Active Directory policy was functioning as expected.

The test demonstrated that repeated failed authentication attempts could transition the account into a locked state.

---

## 4. Root Cause

**Root Cause:**

The Active Directory user account was locked after repeated unsuccessful authentication attempts exceeded the configured account lockout threshold.

The lockout was reproduced intentionally as part of the lab exercise.

> **Lab note:** In a real production incident, the technician would also investigate the source of the failed authentication attempts before simply unlocking the account.

Potential causes in a production environment could include:

* User entering an incorrect password
* Saved credentials containing an old password
* A mapped network drive
* An application repeatedly attempting authentication
* A Windows service using outdated credentials
* Another device repeatedly attempting to authenticate

---

## 5. Resolution

The locked account was unlocked from the Domain Controller using:

```powershell
Unlock-ADAccount -Identity sarah
```

The account status was then checked again:

```powershell
Get-ADUser sarah -Properties LockedOut
```

Expected result:

```text
LockedOut : False
```

---

## 6. Validation

After unlocking the account, authentication was tested again.

The account was no longer locked and the user was able to authenticate successfully.

The troubleshooting process therefore confirmed:

```text
Account locked
      ↓
Verified account exists
      ↓
Confirmed LockedOut = True
      ↓
Reviewed lockout policy
      ↓
Unlocked account
      ↓
Verified LockedOut = False
      ↓
Tested authentication
      ↓
Access restored
```

---

## 7. Resolution Notes

**Resolution:** User account unlocked successfully.

**User impact:** User was temporarily unable to authenticate to the domain.

**Final account state:** Unlocked.

**Validation:** Successful authentication confirmed after remediation.

**Follow-up:** If this occurred in production, investigate the source of the failed authentication attempts to prevent the account from becoming locked again.

---

## 8. Technician Notes

This incident was completed as a simulated Help Desk scenario within a personal Active Directory lab.

The purpose of the ticket was to practice a realistic troubleshooting workflow:

1. Identify the user's reported symptom
2. Verify the account
3. Check the account state
4. Review the applicable domain policy
5. Reproduce the behavior when appropriate
6. Apply the appropriate remediation
7. Validate the result
8. Document the resolution

The exercise demonstrates how a Help Desk technician can use both the Active Directory GUI and PowerShell to investigate and resolve a common Windows authentication issue.

---

## 9. Evidence

Supporting screenshots are stored in:

```text
../screenshots/06-account-lockout/
```

Evidence includes the account lockout configuration, testing, account state verification, remediation, and successful validation.
