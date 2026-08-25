# Support Ticket — NTFS Permissions Issue

## Ticket Information

**Ticket ID:** INC-NTFS-001
**Category:** File & Folder Access
**Subcategory:** NTFS Permissions
**Priority:** Medium
**Status:** Resolved
**Affected System:** Windows Server
**Environment:** AWS EC2

---

## User Issue

**User Report:**

> User reported that they were unable to access or perform the required operation on a Windows folder.

The user stated that the folder was available on the system but that they did not have the expected level of access.

---

## Initial Assessment

The issue was identified as a potential **NTFS permissions problem**.

The following areas were investigated:

* Current user account
* Folder existence
* NTFS permissions
* User/group membership
* Permission inheritance
* Actual file-system access

---

## Troubleshooting Performed

### 1. Identified the Current User

The currently logged-in account was identified using:

```powershell
whoami
```

This established which security principal was being used during the access test.

### 2. Verified the Folder

The test directory was checked to confirm that it existed and was accessible from the Windows system.

### 3. Inspected NTFS Permissions

The folder's ACL was inspected using:

```powershell
Get-Acl "C:\NTFS-Lab"
```

The individual access-control entries were then reviewed to determine which accounts and groups had permissions.

### 4. Checked Group Membership

The user's group memberships were reviewed:

```powershell
whoami /groups
```

This was important because NTFS permissions may be granted through group membership rather than directly to an individual user.

### 5. Tested File Access

A test file operation was performed to determine whether the configured permissions actually allowed the required action.

For example:

```powershell
Add-Content "C:\NTFS-Lab\test.txt" "NTFS permission test"
```

The result of the operation was used to verify the user's effective access.

### 6. Verified the Final Configuration

The ACL was reviewed again after the permissions configuration:

```powershell
(Get-Acl "C:\NTFS-Lab").Access
```

This confirmed the final permission configuration.

---

## Root Cause

The issue was related to **NTFS access control configuration**.

The user's ability to interact with the folder depended on the permissions assigned to the account and/or groups to which the account belonged.

---

## Resolution

The NTFS permissions were reviewed and configured according to the access requirements of the lab scenario.

Access was then retested to confirm that the intended user could perform the required operation.

The final configuration was verified through both the Windows security interface and PowerShell ACL inspection.

---

## Verification

The resolution was considered successful after confirming:

* The correct user account was being tested.
* The target directory existed.
* The expected NTFS permissions were present.
* The ACL configuration matched the intended access.
* The required file operation could be performed.
* Unauthorized access remained restricted where applicable.

---

## Evidence

Screenshots associated with this ticket document the troubleshooting and verification process.

Evidence includes:

1. Windows environment
2. Test directory
3. Security/permission configuration
4. Permission verification
5. File-access testing
6. Final configuration

See the `screenshots/` directory for the corresponding evidence.

---

## Technician Notes

NTFS permission issues should be investigated systematically rather than immediately granting Full Control.

A proper troubleshooting process should determine:

1. **Who** is attempting the operation?
2. **What** resource are they attempting to access?
3. **What permissions** does the user have?
4. **Which groups** provide additional permissions?
5. **Are permissions inherited?**
6. **What does the user actually have access to?**
7. **Can the issue be resolved using the minimum required permissions?**

The principle of least privilege should be followed whenever possible.

---

## Resolution Summary

**Issue:** User unable to perform the required operation on a Windows folder.

**Cause:** NTFS permission configuration.

**Action:** Inspected and configured NTFS permissions and verified effective access.

**Result:** Required access was successfully verified.

**Status:** **Resolved**
