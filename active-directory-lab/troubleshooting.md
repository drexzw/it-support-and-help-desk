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

# 7. Permission Command Failed: Wrong Domain Prefix

## Symptoms

While granting NTFS permissions on the department folders, the first `icacls` commands failed. The principal had been written as:

```text
CORP\GG-IT-Users
```

The failed attempt was not captured in a screenshot (**not pictured**).

## Investigation

The domain is named `corp.drexzw.local`, so `CORP` looked like the domain name. But the `DOMAIN\name` format uses the domain's **NetBIOS name**, which in this lab is `DREXZW`, not the first label of the DNS name. The `NetBIOSName` value is visible in the domain details in `screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png`.

## Resolution

The commands were re-run with the correct prefix:

```powershell
icacls "C:\CompanyData\IT" /grant "DREXZW\GG-IT-Users:(OI)(CI)M"
```

The successful commands are captured in `screenshots/07-department-permissions/04-icacls-department-grants.png`.

## Lesson

In `DOMAIN\principal` format, use the NetBIOS name. The DNS name and the NetBIOS name of a domain can differ, and guessing from the DNS name produces errors that look like the group does not exist.

---

# 8. Inherited Permissions Undermined Department Folder Separation

## Symptoms

No user reported a problem. The issue was found while verifying the permissions after the group grants succeeded.

## Investigation

The baseline ACL (`03-baseline-acl-before-group-permissions.png`) showed default entries on `C:\CompanyData` and the IT folder, including `BUILTIN\Users`.

After the department groups were granted access, `icacls` was run against each folder (`06-icacls-before-inheritance-cleanup.png`). Every department folder showed both the new group entry and inherited `BUILTIN\Users` entries:

```text
BUILTIN\Users:(I)(OI)(CI)(RX)
BUILTIN\Users:(I)(CI)(AD)
BUILTIN\Users:(I)(CI)(WD)
```

Each `icacls /grant` command had reported success, but a successful grant only means the entry was added. It does not show what else is in the ACL. On a Domain Controller, `BUILTIN\Users` is expected to include ordinary domain users, so those inherited entries would likely have let users outside a department read into that department's folder.

This conclusion comes from reading the ACLs. It was not reproduced from a client session.

## Resolution

Inheritance was disabled on each department folder, then `BUILTIN\Users` was removed:

```powershell
icacls "C:\CompanyData\IT" /inheritance:d
icacls "C:\CompanyData\IT" /remove "BUILTIN\Users"
```

(Repeated for HR, Finance, Sales, and Management; evidence in `07-inheritance-disabled-builtin-users-removed.png`.)

The order matters. Inherited entries belong to the parent, so they cannot be removed from the child while it is still inheriting. `/inheritance:d` converts the inherited entries into explicit ones, which can then be removed. SYSTEM and Administrators keep Full Control, so no one is locked out.

## Verification

`icacls` was run again on all five folders (`08-icacls-after-inheritance-cleanup.png`). Each now shows only its department group (plus `GG-IT-Admins` on the IT folder), SYSTEM, Administrators, and CREATOR OWNER. No `BUILTIN\Users` entry and no `(I)` markers remain.

## Lesson

Verify the whole ACL, not just that your own change succeeded. Permissions that arrive through inheritance can quietly widen access, and the way to see them is to read the full ACL before and after.

---

# 9. Shared Folder: User Can Reach the Share but Cannot Write (INC-AD-002)

## Symptoms

`sjohnson` (HR) could reach the `CompanyData` share but could not create a file in the HR folder. The failure on the client was:

```text
New-Item : Access to the path '\\10.0.1.70\CompanyData\HR\sjohnson-test.txt' is denied.
```

This is a simulated incident: the fault was introduced deliberately for the lab.

## Investigation

The reachable share and the write-only failure pointed to a permission problem rather than a network or authentication problem. Checked in order:

1. **Identity:** `whoami` on the client returned `drexzw\sjohnson`, and `Test-Path` on the share returned `True`. Authentication and connectivity were working.
2. **Group membership:** `Get-ADGroupMember GG-HR-Users` listed `sjohnson`.
3. **NTFS permissions:** `icacls C:\CompanyData\HR` showed `GG-HR-Users` with Modify. The file-system layer was correct.
4. **Share permissions:** `Get-SmbShareAccess -Name CompanyData` showed `Everyone` at Read and `GG-HR-Users` at Read.

The share layer was the only one that did not match what the user needed.

## Root Cause

`GG-HR-Users` had only **Read** at the SMB share level. NTFS allowed Modify, but over the network the effective access is the more restrictive of the share and NTFS permissions, so the user could read but not write.

## Resolution

The share permission for the group was changed from Read to Change:

```powershell
Grant-SmbShareAccess -Name "CompanyData" -AccountName "DREXZW\GG-HR-Users" -AccessRight Change -Force
```

## Verification

From the client as `sjohnson`, `New-Item` created `sjohnson-test.txt` in the HR folder, and the file was then removed.

The first `Remove-Item` attempt in that screenshot returned a red parameter error because `-ItemType` is not a `Remove-Item` parameter. That was a syntax mistake, unrelated to permissions; the retry without it returned no error.

## Evidence

All in `screenshots/08-smb-share-access/`: `02` (group and NTFS), `03` (share created), `04` (share permissions at Read), `05` (access denied), `06` (changed to Change), `07` (retest succeeds). Screenshot `05` was captured after the fix and placed in incident order. See `tickets/shared-folder-access-sarah-johnson.md`.

## Lesson

When a user can reach a share but cannot write, check **both** permission layers. A correct NTFS permission does not help if the share permission is more restrictive, and the reverse is also true. Verify identity and group membership first, then compare share and NTFS permissions side by side.

---

# 10. Workstation Security Policy Not Applying: GPO Link Disabled (INC-AD-003)

## Symptoms

The workstation was reported to be missing its expected security policy. This was a controlled lab simulation, not a report from a real end user.

## Investigation

1. **Baseline:** `gpresult /r` on `AD-CLIENT` showed Workstation Security Policy and Default Domain Policy applied.
2. **GPO scope and link:** Group Policy Management showed the GPO linked to the Workstations OU with `Link Enabled: Yes`, `Enforced: No`, and `GPO Status: Enabled`.
3. **Fault introduced:** The link to the Workstations OU was disabled (`Link Enabled: No`). `GPO Status` stayed `Enabled`.
4. **Client check:** `gpupdate /force` completed successfully, and `gpresult /r /scope computer` listed only Default Domain Policy as applied. Workstation Security Policy appeared under "filtered out":

```text
Workstation Security Policy
    Filtering:  Disabled (Link)
```

The refresh worked, so the policy was not failing to download. The filtering reason named the link.

## Root Cause

The link between Workstation Security Policy and the Workstations OU was disabled. The GPO itself still existed, was enabled, and was unchanged, but a GPO with a disabled link does not apply.

## Resolution

The link was re-enabled in Group Policy Management (`Link Enabled: Yes`).

## Verification

`gpupdate /force` was run again on the client, and `gpresult /r /scope computer` listed Workstation Security Policy and Default Domain Policy as applied.

## Evidence

All in `screenshots/09-group-policy-troubleshooting/`: `01` to `04` (initial state and scope), `05` (link disabled), `06` (client shows `Disabled (Link)`), `07` (link restored), `08` (policy applied again). Screenshot `05` was captured about a minute after `06`, while the link was still disabled, and placed in incident order. See `tickets/group-policy-not-applying.md`.

## Limits

Results are computer-scope, from one computer (`AD-CLIENT`), run as the local `Administrator`. The settings inside the GPO were not re-tested; the check was whether the GPO appears in the applied list.

## Lesson

A GPO can exist, be correctly configured, and still not apply because its **link** is disabled. `gpupdate` refreshes policy; `gpresult` shows what actually applied and, for a filtered GPO, why not. Check the GPO's scope and link as well as its settings.

```text
Expected policy missing
   ↓
gpresult: is the GPO applied? (no)
   ↓
Filtered-out reason: Disabled (Link)
   ↓
Check the link in Group Policy Management
   ↓
Re-enable the link
   ↓
gpupdate /force, then gpresult again
   ↓
GPO applied
```

---

# 11. Troubleshooting Method Used

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

### 7. Use the NetBIOS name in DOMAIN\principal commands

The domain's DNS name (`corp.drexzw.local`) and NetBIOS name (`DREXZW`) are different. Permission commands need the NetBIOS name.

### 8. Read the full ACL, not just the entries you added

A permission grant can succeed while inherited entries still give other users access. Review the complete ACL before and after changes.

### 9. Check both the share and NTFS layers

Over SMB, a user gets the more restrictive of the share permission and the NTFS permission. When someone can open a shared folder but cannot save, compare both layers.

### 10. Check the GPO link, not only its settings

A correctly configured GPO does not apply if its link is disabled. `gpresult` names the reason a GPO was filtered out, which is faster than guessing.

---

# Related Documentation

Account-lockout incident:

```text
tickets/account-lockout-sarah-johnson.md
```

Shared-folder access incident:

```text
tickets/shared-folder-access-sarah-johnson.md
```

Group Policy not applying incident:

```text
tickets/group-policy-not-applying.md
```

Command reference:

```text
commands.md
```

Evidence:

```text
screenshots/
```

Department permissions evidence (scenarios 7 and 8):

```text
screenshots/07-department-permissions/
```

SMB share and access ticket evidence (scenario 9):

```text
screenshots/08-smb-share-access/
```

Group Policy ticket evidence (scenario 10):

```text
screenshots/09-group-policy-troubleshooting/
```

---

# Future Troubleshooting Scenarios

NTFS permissions for the department folders (scenario 8) an SMB share with a share-versus-NTFS access ticket (scenario 9), and a disabled GPO link (scenario 10) are now documented. Further scenarios can build on them.

Future scenarios can include:

* User can access a share but cannot open a file
* User has incorrect NTFS permissions
* Share permissions conflict with NTFS permissions
* Department users cannot access the correct folder
* User receives "Access Denied"
* Group membership changes do not immediately appear in the user's session

These scenarios will extend the current Active Directory environment into more realistic Windows Help Desk permission troubleshooting.
