# Shared Folder Access Denied - Troubleshooting Log

This document contains the PowerShell commands used during the troubleshooting process, along with the reasoning behind each step.

---

## User Account Management

### Create Test Users

Created two local user accounts to simulate employees.

```powershell
net user employee1 Password123! /add
net user employee2 Password123! /add
```

### View Local User Accounts

Used to verify that the accounts were created successfully.

```powershell
net user
```

### Check Current Logged-In User

Used to confirm which account was experiencing the issue.

```powershell
whoami
```

Example output:

```
DESKTOP-PC\employee2
```

---

## Permission Investigation

### View Folder Permissions

Used to check the current NTFS permissions assigned to the folder.

```powershell
icacls C:\Company-Data\Finance
```

Example output:

```
Victor\employee1:(OI)(CI)(RX)
Victor\employee2:(OI)(CI)(DENY)(Rc,RD,REA,X,RA)
```

This revealed the real problem: `employee2` had an existing **DENY** entry blocking read/execute access — not just a missing grant.

---

## Permission Resolution

### First attempt (failed): unquoted parentheses

```powershell
icacls C:\Company-Data\Finance /grant employee2:(RX)
```

PowerShell parses parentheses as its own syntax, so it tried to run `RX` as a separate command and threw `CommandNotFoundException`.

### Fix: wrap the permission string in quotes

```powershell
icacls C:\Company-Data\Finance /grant "employee2:(RX)"
```

This succeeded, but access was **still denied** — because the existing DENY entry from earlier in the lab overrides any ALLOW entry, regardless of order.

### Remove the conflicting DENY entry

```powershell
icacls C:\Company-Data\Finance /remove:d "employee2"
```

`/remove:d` removes only DENY entries for the specified user, leaving the ALLOW entry intact.

### Verify Permission Changes

```powershell
icacls C:\Company-Data\Finance
```

Expected result:

```
Victor\employee2:(RX)
```

---

## Troubleshooting Process Summary

1. **Identify affected user**
   ```powershell
   whoami
   ```
2. **Check existing permissions**
   ```powershell
   icacls C:\Company-Data\Finance
   ```
3. **Attempt to grant permissions** (hit a PowerShell parsing error on unquoted parentheses)
   ```powershell
   icacls C:\Company-Data\Finance /grant "employee2:(RX)"
   ```
4. **Discover a conflicting DENY entry** was still blocking access despite the successful grant
5. **Remove the DENY entry**
   ```powershell
   icacls C:\Company-Data\Finance /remove:d "employee2"
   ```
6. **Verify the fix**
   ```powershell
   icacls C:\Company-Data\Finance
   ```

## Key Takeaway

In NTFS, **DENY always overrides ALLOW**, regardless of the order permissions were applied in. A successful `/grant` command can still leave a user locked out if a conflicting DENY entry exists elsewhere on the ACL — always check `icacls` output in full, not just confirm the grant command ran without error.
