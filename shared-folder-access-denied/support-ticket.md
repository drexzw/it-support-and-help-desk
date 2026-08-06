# Support Ticket

## Ticket ID

HD-2026-004

---

## Issue Category

File Access / Permissions

---

## Priority

Medium

---

## User Report

The user reported that they were unable to access the Finance shared folder.

When attempting to open the folder, Windows displayed an "Access Denied" message.

Other users were able to access the folder successfully.

---

## Business Impact

The user was unable to access required company files stored within the shared folder.

This prevented them from completing tasks requiring access to Finance documents.

---

## Troubleshooting Performed

### Step 1 - Identify the User

Verified the currently logged-in account.

```powershell
whoami
```

Finding: the affected account was `employee2`.

### Step 2 - Reproduce the Issue

Attempted to access the folder directly:

```
C:\Company-Data\Finance
```

Result: an "Access Denied" message was displayed.

### Step 3 - Investigate Permissions

Checked the folder's NTFS permissions:

```powershell
icacls C:\Company-Data\Finance
```

This revealed that `employee2` had an explicit **DENY** entry on the folder:

```
Victor\employee2:(OI)(CI)(DENY)(Rc,RD,REA,X,RA)
```

---

## Root Cause

The `employee2` account had an explicit DENY entry on the Finance folder's ACL. Because DENY permissions always override ALLOW permissions in NTFS regardless of order, this blocked access even though the account was otherwise a valid user of the shared folder.

---

## Resolution

Removed the conflicting DENY entry:

```powershell
icacls C:\Company-Data\Finance /remove:d "employee2"
```

Then granted the required Read and Execute permissions:

```powershell
icacls C:\Company-Data\Finance /grant "employee2:(RX)"
```

The user was then able to access the folder successfully.

---

## Verification

Confirmed permissions were applied correctly:

```powershell
icacls C:\Company-Data\Finance
```

Confirmed that:

- The DENY entry for `employee2` was removed
- The Finance folder opened successfully
- The user could access the files inside the folder
- The issue was resolved

---

## Ticket Status

Closed - Resolved

---

## Technician Notes

Recommended improvements:

- Use security groups instead of individual user permissions in larger environments
- Regularly review folder permissions for conflicting or leftover DENY entries
- Follow least privilege access principles
