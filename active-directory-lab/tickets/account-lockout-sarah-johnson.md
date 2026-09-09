# Help Desk Ticket — Account Lockout

**Ticket ID:** INC-AD-001
**Status:** Resolved
**Priority:** Medium
**Category:** Account / Authentication
**Environment:** Active Directory Lab
**Domain:** `corp.drexzw.local` (NetBIOS: `DREXZW`)
**Affected User:** Sarah Johnson
**Affected Account:** `sjohnson`
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

Before making changes, the domain's Account Lockout Policy was reviewed in Group Policy Management to determine the conditions under which an account becomes locked.

Reviewed in the Group Policy Management Editor (Default Domain Policy → Computer Configuration → Windows Settings → Security Settings → Account Policies → Account Lockout Policy):

| Setting | Value |
|---|---|
| Account lockout threshold | 5 invalid logon attempts |
| Account lockout duration | 5 minutes |
| Reset account lockout counter after | 1 minute |

Screenshot: `../screenshots/06-account-lockout-policy/02-account-lockout-policy-settings.png`

---

## 3. Troubleshooting

### Step 1 — Verify the User Account (Baseline)

Sarah Johnson's account was opened in Active Directory Users and Computers (Properties → Account tab) to confirm the account existed and to capture its state before reproducing the issue.

Screenshot: `../screenshots/06-account-lockout-policy/01-sjohnson-account-before-lockout.png`

---

### Step 2 — Reproduce the Lockout

Rather than waiting for a real failed-login event, the lockout was intentionally reproduced from a client machine using `runas` with an incorrect password, repeated past the configured threshold:

```powershell
runas /user:DREXZW\sjohnson powershell.exe
```

After 5 failed attempts, the account locked, and Windows returned:

```text
1909: The referenced account is currently locked out and may not be logged on to.
```

Screenshot: `../screenshots/06-account-lockout-policy/03-account-lockout-trigger-test.png`

---

### Step 3 — Confirm the Lockout in Active Directory

Sarah Johnson's account was reopened in Active Directory Users and Computers. The Account tab displayed:

> "This account is currently locked out on this Active Directory Domain Controller."

with the **Unlock account** checkbox now available.

Screenshot: `../screenshots/06-account-lockout-policy/04-account-lockout-confirmation-aduc.png`

---

## 4. Root Cause

**Root Cause:**

The Active Directory user account `sjohnson` was locked after repeated unsuccessful authentication attempts exceeded the configured account lockout threshold (5 invalid attempts).

The lockout was reproduced intentionally as part of the lab exercise, using repeated `runas` attempts with a deliberately incorrect password.

> **Lab note:** In a real production incident, the technician would also investigate the source of the failed authentication attempts before simply unlocking the account.

Potential causes in a production environment could include:

* User entering an incorrect password
* Saved credentials containing an old password
* A mapped network drive using stale credentials
* An application repeatedly attempting authentication
* A Windows service using outdated credentials
* Another device repeatedly attempting to authenticate as the user

---

## 5. Resolution

The account was unlocked directly from Active Directory Users and Computers by checking the **Unlock account** checkbox on the Account tab and applying the change.

Screenshot: `../screenshots/06-account-lockout-policy/04-account-lockout-confirmation-aduc.png`

---

## 6. Validation

After unlocking the account, authentication was tested again using the same `runas` approach:

```powershell
runas /user:DREXZW\sjohnson powershell.exe
whoami
```

The result confirmed successful domain authentication as the affected user:

```text
drexzw\sjohnson
```

Screenshot: `../screenshots/06-account-lockout-policy/05-account-unlock-success-test.png`

The troubleshooting process therefore confirmed:

```text
Account locked (via repeated runas attempts)
      ↓
Verified baseline account state in ADUC
      ↓
Reviewed configured lockout policy (GPMC)
      ↓
Confirmed lockout in ADUC ("currently locked out" message)
      ↓
Unlocked account via ADUC checkbox
      ↓
Re-tested authentication via runas + whoami
      ↓
Access restored — confirmed as drexzw\sjohnson
```

---

## 7. Resolution Notes

**Resolution:** User account unlocked successfully via Active Directory Users and Computers.

**User impact:** User was temporarily unable to authenticate to the domain.

**Final account state:** Unlocked.

**Validation:** Successful `runas` authentication as `DREXZW\sjohnson`, confirmed via `whoami`.

**Follow-up:** If this occurred in production, investigate the source of the failed authentication attempts to prevent the account from becoming locked again.

---

## 8. Technician Notes

This incident was completed as a simulated Help Desk scenario within a personal Active Directory lab.

The purpose of the ticket was to practice a realistic troubleshooting workflow:

1. Identify the user's reported symptom
2. Capture the account's baseline state
3. Review the applicable domain policy
4. Reproduce the behavior intentionally
5. Confirm the resulting account state
6. Apply the appropriate remediation
7. Validate the result
8. Document the resolution

This run of the exercise was performed entirely through the Active Directory GUI (Group Policy Management and Active Directory Users and Computers) plus `runas`/`whoami` on the client for testing. No PowerShell AD cmdlets (`Get-ADUser`, `Unlock-ADAccount`, etc.) were used in this specific run — see the note below for the PowerShell equivalent.

---

## 9. Alternative / Production Approach (Not Pictured in This Run)

In a real environment, or a future run of this lab, the same investigation and remediation could be performed with the Active Directory PowerShell module instead of the GUI:

```powershell
# Check lockout status
Get-ADUser sjohnson -Properties LockedOut

# Review domain lockout policy
Get-ADDefaultDomainPasswordPolicy

# Unlock the account
Unlock-ADAccount -Identity sjohnson

# Confirm it unlocked
Get-ADUser sjohnson -Properties LockedOut
```

These commands are included for reference only — they were not run as part of this ticket and are not shown in any screenshot for this scenario.

---

## 10. Evidence

Supporting screenshots are stored in:

```text
../screenshots/06-account-lockout-policy/
```

| Step | Screenshot |
|---|---|
| Account state before lockout | `01-sjohnson-account-before-lockout.png` |
| Account lockout policy configuration | `02-account-lockout-policy-settings.png` |
| Lockout triggered via repeated `runas` attempts | `03-account-lockout-trigger-test.png` |
| Lockout confirmed in ADUC | `04-account-lockout-confirmation-aduc.png` |
| Successful authentication after unlock | `05-account-unlock-success-test.png` |
