# NTFS Permissions Lab — Commands

## Overview

This document records the commands used during the NTFS Permissions Lab to inspect the Windows environment, create the test directory, configure permissions, and verify access.

---

## 1. Verify the Current User

```powershell
whoami
```

**Purpose:**
Displays the currently logged-in Windows account.

**Why it was used:**
The account must be identified before troubleshooting file and folder permissions.

---

## 2. Check the Current Directory

```powershell
pwd
```

**Purpose:**
Displays the current working directory.

**Why it was used:**
Helps confirm the location where file-system operations are being performed.

---

## 3. Create the Test Directory

```powershell
New-Item -ItemType Directory -Path "C:\NTFS-Lab"
```

**Purpose:**
Creates a dedicated directory for testing NTFS permissions.

---

## 4. View the Directory

```powershell
Get-ChildItem "C:\NTFS-Lab"
```

**Purpose:**
Lists the contents of the test directory.

**Why it was used:**
Confirms that the directory exists and allows its contents to be inspected.

---

## 5. View NTFS Access Control Entries

```powershell
Get-Acl "C:\NTFS-Lab"
```

**Purpose:**
Displays the Access Control List (ACL) associated with the directory.

The ACL contains the users and groups that have permissions on the folder and the type of access they have been granted.

---

## 6. Display the ACL in a More Readable Format

```powershell
(Get-Acl "C:\NTFS-Lab").Access
```

**Purpose:**
Displays individual access-control entries associated with the folder.

This makes it easier to identify:

* Which account or group has access
* Whether access is allowed or denied
* The permission level
* Whether permissions are inherited

---

## 7. Check the Current User's Group Membership

```powershell
whoami /groups
```

**Purpose:**
Displays the security groups associated with the current user.

**Why it was used:**
NTFS permissions can be assigned directly to users or through groups, so identifying group membership is important when troubleshooting access.

---

## 8. Verify File-System Access

```powershell
Test-Path "C:\NTFS-Lab"
```

**Purpose:**
Tests whether the specified path exists and can be accessed.

---

## 9. Create a Test File

```powershell
New-Item -ItemType File -Path "C:\NTFS-Lab\test.txt"
```

**Purpose:**
Creates a test file inside the directory.

**Why it was used:**
Creating a file provides a practical way to verify whether the user has write access to the directory.

---

## 10. Verify the Test File

```powershell
Get-Item "C:\NTFS-Lab\test.txt"
```

**Purpose:**
Confirms that the test file exists and allows its properties to be inspected.

---

## 11. Review Permissions After Configuration

```powershell
(Get-Acl "C:\NTFS-Lab").Access | Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited
```

**Purpose:**
Provides a concise view of the configured NTFS permissions.

The output helps verify:

* Identity receiving the permission
* File-system rights
* Allow/Deny status
* Whether the permission was inherited

---

## 12. Verify Access by Testing File Operations

Example:

```powershell
Add-Content "C:\NTFS-Lab\test.txt" "NTFS permission test"
```

**Purpose:**
Attempts to write data to the file.

**Why it was used:**
A successful operation indicates that the current account has sufficient write permission.

If the operation fails with an access-denied error, the NTFS permissions should be investigated.

---

## Command Summary

| Command          | Purpose                           |
| ---------------- | --------------------------------- |
| `whoami`         | Identify the current user         |
| `pwd`            | Display the current directory     |
| `New-Item`       | Create the test directory/file    |
| `Get-ChildItem`  | List directory contents           |
| `Get-Acl`        | Retrieve NTFS ACL information     |
| `whoami /groups` | Display user group membership     |
| `Test-Path`      | Test whether a path is accessible |
| `Get-Item`       | Inspect a file or directory       |
| `Add-Content`    | Test write access                 |

## Key Takeaway

The most important commands for NTFS troubleshooting are `Get-Acl` and the commands used to test the actual file operation.

Checking the ACL shows **what permissions are configured**, while performing an actual file operation helps confirm **what the user can really do**.
