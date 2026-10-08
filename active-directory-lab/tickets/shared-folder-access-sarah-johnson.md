# Help Desk Ticket — Shared Folder Access (Read-Only Over SMB)

**Ticket ID:** INC-AD-002
**Status:** Resolved
**Priority:** Medium
**Category:** File Access / Permissions
**Environment:** Active Directory Lab
**Domain:** `corp.drexzw.local` (NetBIOS: `DREXZW`)
**Affected User:** Sarah Johnson
**Affected Account:** `sjohnson`
**Technician:** IT Support Lab
**Date:** October 2026

---

## 1. User Report

**User:** Sarah Johnson (HR)

**Issue (simulated):**

Sarah can open the HR shared folder but cannot create or save files in it.

**User-facing symptom:**

> "I can see the HR folder, but when I try to create a file, I get access denied."

> **Lab note:** This is a simulated ticket. The fault was introduced deliberately by setting the share-level permission for `GG-HR-Users` to Read, so the troubleshooting could be practiced.

---

## 2. Initial Assessment

The user could reach the share, so the problem was unlikely to be network connectivity, DNS, or domain authentication. The failure was limited to writing, which pointed to a permission issue at one of two layers:

* NTFS permissions on the HR folder
* Share permissions on the `CompanyData` SMB share

Both layers needed to be checked, along with the user's identity and group membership.

---

## 3. Troubleshooting

### Step 1 — Verify the Domain and the Account

The domain details and the user's account were checked on the Domain Controller. The NetBIOS name is `DREXZW`, and `sjohnson` is in the HR OU.

```powershell
Get-ADDomain
Get-ADUser sjohnson
```

Screenshot: `../screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png`

---

### Step 2 — Verify Group Membership and NTFS Permissions

`sjohnson` is a member of `GG-HR-Users`, and the HR folder grants that group Modify:

```powershell
Get-ADGroupMember GG-HR-Users
icacls C:\CompanyData\HR
```

```text
C:\CompanyData\HR DREXZW\GG-HR-Users:(OI)(CI)(M)
```

The NTFS layer was correct.

Screenshot: `../screenshots/08-smb-share-access/02-hr-group-membership-and-ntfs-verified.png`

---

### Step 3 — Check the SMB Share

The `CompanyData` share was created on the Domain Controller:

```powershell
New-SmbShare -Name "CompanyData" -Path "C:\CompanyData" -Description "Company Department File Share"
```

Screenshot: `../screenshots/08-smb-share-access/03-smb-share-created.png`

Share permissions were then reviewed. `GG-HR-Users` was set to **Read** at the share level:

```powershell
Get-SmbShareAccess -Name CompanyData
```

| Account | Access |
|---|---|
| `Everyone` | Read |
| `DREXZW\GG-HR-Users` | Read |

Screenshot: `../screenshots/08-smb-share-access/04-smb-share-permissions-hr-read-only.png`

---

### Step 4 — Reproduce the Problem from the Client

From the client, in a PowerShell session running as `sjohnson`:

```powershell
whoami
Test-Path "\\10.0.1.70\CompanyData"
New-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt" -ItemType File
```

Results:

* `whoami` returned `drexzw\sjohnson`
* `Test-Path` returned `True` (the share is reachable)
* `New-Item` failed:

```text
New-Item : Access to the path '\\10.0.1.70\CompanyData\HR\sjohnson-test.txt' is denied.
```

Screenshot: `../screenshots/08-smb-share-access/05-sjohnson-access-denied-creating-file.png`

---

## 4. Root Cause

**Root Cause:**

The SMB share permission for `DREXZW\GG-HR-Users` was set to **Read**. The NTFS permission on the HR folder was Modify, but over the network a user gets the more restrictive of the share and NTFS permissions, so `sjohnson` could read but not create files.

---

## 5. Resolution

The share permission for `GG-HR-Users` was changed from Read to Change:

```powershell
Grant-SmbShareAccess -Name "CompanyData" -AccountName "DREXZW\GG-HR-Users" -AccessRight Change -Force
```

The share's permissions after the change:

| Account | Access |
|---|---|
| `Everyone` | Read |
| `DREXZW\GG-HR-Users` | Change |

Screenshot: `../screenshots/08-smb-share-access/06-smb-share-permission-changed-to-change.png`

---

## 6. Validation

The test was repeated from the client as `sjohnson`:

```powershell
whoami
Test-Path "\\10.0.1.70\CompanyData"
Get-ChildItem "\\10.0.1.70\CompanyData"
New-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt" -ItemType File
Remove-Item "\\10.0.1.70\CompanyData\HR\sjohnson-test.txt"
```

The file was created successfully (`sjohnson-test.txt`, 0 bytes), and the test file was then removed.

Screenshot: `../screenshots/08-smb-share-access/07-sjohnson-retest-file-creation-succeeds.png`

> **Note on the red error in the screenshot:** the first `Remove-Item` attempt included `-ItemType File`, which is not a `Remove-Item` parameter, so PowerShell returned a parameter error. This was a command-syntax mistake and unrelated to permissions. Running `Remove-Item` without it returned no error. The deletion was not listed afterwards (**not pictured**).

The troubleshooting process therefore confirmed:

```text
User can reach the share but cannot create a file
      ↓
Verified identity and domain details (whoami, Get-ADDomain, Get-ADUser)
      ↓
Verified group membership (Get-ADGroupMember)
      ↓
Checked NTFS permissions on the HR folder (Modify, correct)
      ↓
Checked share permissions (GG-HR-Users: Read, too restrictive)
      ↓
Reproduced the denial from the client
      ↓
Changed GG-HR-Users to Change at the share layer
      ↓
Re-tested from the client as sjohnson
      ↓
File created successfully
```

---

## 7. Resolution Notes

**Resolution:** Share permission for `GG-HR-Users` changed from Read to Change.

**User impact:** User could open the HR folder but could not create or save files.

**Final share state:** `Everyone`: Read; `DREXZW\GG-HR-Users`: Change.

**Validation:** From the client as `drexzw\sjohnson`, a file was created in `\\10.0.1.70\CompanyData\HR`.

**Follow-up:** In a production environment, document the intended share permissions per department so the share layer and NTFS layer are set consistently, and avoid relying on `Everyone` at the share level without a reason.

---

## 8. Technician Notes

This incident was completed as a simulated Help Desk scenario within a personal Active Directory lab.

The purpose of the ticket was to practice a realistic troubleshooting workflow for "user can open a folder but cannot save":

1. Establish what works (the share is reachable) and what fails (writing)
2. Confirm the user's identity and group membership
3. Compare the permission layers: NTFS and share
4. Reproduce the failure from the client
5. Fix the layer that is wrong, not the one that is already correct
6. Re-test from the client as the same user
7. Document the result

**Scope limits of this ticket:**

* The share is hosted on the Domain Controller, a lab simplification.
* Only `GG-HR-Users` was given Change on the share. The other department groups only have the `Everyone` Read entry at the share layer, so their write access over SMB was not configured or tested.
* From `sjohnson`'s session, the share root listed all five department folder names (screenshot 07). This ticket does not test whether she can open another department's folder.

---

## 9. Evidence Notes

* Screenshot `05` (access denied) was captured after the fix, and is placed in incident order. The failing condition was therefore recreated for the capture rather than recorded live before the fix. The earlier steps (`02`, `04`) show the state that caused the failure.
* The client path in the screenshots is the IP address `\\10.0.1.70\CompanyData`.

---

## 10. Evidence

Supporting screenshots are stored in:

```text
../screenshots/08-smb-share-access/
```

| Step | Screenshot |
|---|---|
| Domain details and `sjohnson` account | `01-domain-and-user-identity-verified.png` |
| Group membership and NTFS permissions on HR | `02-hr-group-membership-and-ntfs-verified.png` |
| SMB share created | `03-smb-share-created.png` |
| Share permissions with `GG-HR-Users` at Read | `04-smb-share-permissions-hr-read-only.png` |
| Access denied creating a file | `05-sjohnson-access-denied-creating-file.png` |
| Share permission changed to Change | `06-smb-share-permission-changed-to-change.png` |
| Retest: file created successfully | `07-sjohnson-retest-file-creation-succeeds.png` |
