# Active Directory Lab — Windows Server 2022

## Overview

This project is a hands-on Active Directory lab built to simulate a small business Windows domain environment and practice common IT Support / Help Desk administration tasks.

The lab uses a Windows Server 2022 instance as the Domain Controller and a Windows client joined to the domain. The environment was used to configure Active Directory, organizational units, users and groups, Group Policy, domain membership, account lockout behavior, department-based NTFS permissions, SMB file sharing, and Group Policy troubleshooting.

The lab also includes troubleshooting scenarios to practice diagnosing issues from both the Domain Controller and client side.

> **Environment:** Personal lab / simulated business environment
> **Domain:** `corp.drexzw.local` (NetBIOS: `DREXZW`)

---

## Objectives

* Deploy and configure a Windows Server 2022 Domain Controller
* Install Active Directory Domain Services (AD DS)
* Create and organize Active Directory OUs across multiple departments
* Create domain users and security groups
* Join a Windows client to the domain
* Configure and verify Group Policy
* Configure password and account lockout policies
* Create department folders and apply group-based NTFS permissions
* Review and correct inherited permissions using `icacls`
* Publish the department folders as an SMB share and troubleshoot a share-versus-NTFS permission conflict
* Diagnose a Group Policy application failure caused by a disabled GPO link, using `gpresult` and `gpupdate`
* Test domain authentication
* Troubleshoot Active Directory and domain-join issues using both the GUI and PowerShell
* Simulate a Help Desk account-lockout ticket
* Document troubleshooting procedures and commands

---

## Lab Environment

| Component              | Configuration                         |
| ---------------------- | ------------------------------------- |
| Domain Controller      | Windows Server 2022                   |
| Client                 | Windows workstation                   |
| Domain                 | `corp.drexzw.local`                   |
| NetBIOS Name           | `DREXZW`                              |
| Directory Service      | Active Directory Domain Services      |
| DNS                    | Windows Server / Active Directory DNS |
| Virtualization / Cloud | AWS EC2                               |
| Domain Controller Role | DC + DNS                              |
| Client Role            | Domain-joined workstation             |

---

## Lab Architecture

```text
                    AWS Environment
                          |
                          |
              +-----------------------+
              | Windows Server 2022   |
              | Domain Controller     |
              |                       |
              | AD DS                 |
              | DNS                   |
              | Group Policy          |
              +-----------+-----------+
                          |
                    corp.drexzw.local
                          |
             +------------+------------+
             |                         |
       Active Directory           Windows Client
             |                         |
   +----+----+----+----+----+     Domain Joined
   |    |    |    |    |    |
  IT   HR  Finance Sales Mgmt Workstations
   |    |    |    |    |          |
 GG-IT GG-HR GG-Fin GG-Sales GG-Mgmt   Client Computer
   |
 jsmith
```

Each department OU (IT, HR, Finance, Sales, Management) has a corresponding `GG-<Dept>-Users` global security group. The Workstations OU holds domain-joined client computers and has the account lockout / password GPO linked to it. An additional `GG-IT-Admins` group is used for administrative access to the IT department folder (see section 7).

---

# Implementation

## 1. Domain Controller Deployment

A Windows Server 2022 EC2 instance was deployed and configured as the Domain Controller.

The server was configured with the appropriate network settings before installing Active Directory Domain Services. The Domain Controller was verified both through the GUI/PowerShell locally and via PowerShell from a remote administrative session.

Evidence:

* Static/network configuration
* AD DS installation
* Domain Controller verification (`Get-ADDomain` / `Get-ADDomainController`)
* Domain Controller discovery verification (`Get-ADDomainController -Discover`)

Screenshots:

```text
screenshots/01-domain-controller/01-dc-ip-configuration.png
screenshots/01-domain-controller/02-ad-ds-installation.png
screenshots/01-domain-controller/03-domain-controller-verification.png
screenshots/01-domain-controller/04-domain-controller-powershell-verification.png
```

---

## 2. Active Directory Structure

After installing AD DS, the Active Directory Users and Computers console was used to create a full organizational structure spanning multiple departments: **IT, HR, Finance, Sales, Management, Service Accounts,** and **Workstations**.

The Workstations OU was verified empty prior to the client domain join, and its properties/attribute editor were reviewed as part of confirming the OU's configuration (including the linked GPO). After the client joined the domain, the same OU was reviewed again to confirm the computer object was correctly placed inside it.

Evidence:

```text
screenshots/02-active-directory-structure/01-aduc-ou-structure.png
screenshots/02-active-directory-structure/02-workstations-ou-empty-before-join.png
screenshots/02-active-directory-structure/03-workstations-ou-attribute-editor.png
screenshots/02-active-directory-structure/04-workstations-ou-computer-confirmed.png
```

This structure provides a foundation for applying different administrative policies to users and computers on a per-department basis, rather than managing the domain as one flat group.

---

## 3. Users and Groups

A domain user was created for testing authentication and Group Policy behavior, along with a department-based security group.

The lab included:

* User: `jsmith` (IT OU)
* Group: `GG-IT-Users`
* Group: `GG-IT-Admins` (administrative access to the IT folder)
* Additional department groups visible in the same structure: `GG-HR-Users`, `GG-Finance-Users`, `GG-Sales-Users`, `GG-Management-Users`

Evidence:

```text
screenshots/03-users-and-groups/01-gg-it-users-group.png
screenshots/03-users-and-groups/02-jsmith-it-user.png
```

Using security groups provides a scalable way to assign permissions and policies rather than managing users individually.

---

## 4. Client Domain Join

A Windows client was configured to communicate with the Domain Controller and joined to the `corp.drexzw.local` domain.

DNS resolution and domain membership were verified from the client:

```text
screenshots/04-client-domain-join/01-client-dns-resolution.png
screenshots/04-client-domain-join/02-client-domain-membership.png
```

Domain authentication was then tested using the domain user:

```text
screenshots/04-client-domain-join/03-domain-user-login-attempt.png
screenshots/04-client-domain-join/04-domain-user-login-verification.png
screenshots/04-client-domain-join/05-domain-user-gpo-session-verification.png
```

Finally, the join was independently re-verified from the Domain Controller side using PowerShell — confirming the client's computer object, its OU placement, and its linked GPO:

```text
screenshots/04-client-domain-join/06-adcomputer-workstations-dn-check.png
screenshots/04-client-domain-join/07-gpo-link-verification-powershell.png
screenshots/04-client-domain-join/08-adcomputer-searchbase-confirmation.png
```

---

# 5. Group Policy

A Group Policy Object was created and linked to the Workstations organizational unit.

The policy was used to configure password/account security requirements, and the policy application was verified on the client.

Evidence:

```text
screenshots/05-group-policy/01-gpo-linked-to-workstations.png
screenshots/05-group-policy/02-password-policy-settings.png
screenshots/05-group-policy/03-gpo-application-verification.png
```

The client was refreshed and the resulting policy application was verified.

This GPO is also used later in the lab as the subject of a troubleshooting ticket (section 9).

---

# 6. Account Lockout Help Desk Scenario

A simulated Help Desk incident was created involving the user **Sarah Johnson** (`sjohnson`), whose account became locked after repeated unsuccessful authentication attempts.

The scenario was used to practice:

* Identifying an account lockout
* Understanding the configured lockout policy
* Reproducing the behavior in the lab
* Verifying the account state
* Restoring account access
* Validating successful authentication after remediation

The complete ticket is documented in:

```text
tickets/account-lockout-sarah-johnson.md
```

Supporting evidence is stored in:

```text
screenshots/06-account-lockout-policy/
```

---

# 7. Department Permissions (NTFS)

This stage moves the lab from "users belong to groups" to "groups control access to data." Department folders were created under `C:\CompanyData` on the Domain Controller, and each department's security group was granted access to its own folder.

> **Scope:** These are NTFS permissions on folders hosted on the Domain Controller (a lab simplification; a production environment would normally use a dedicated file server). This stage was verified with `icacls` and PowerShell on the Domain Controller. The folders were shared over SMB afterwards, in section 8.

## Folder and Group Layout

| Folder | Group | Permission |
| --- | --- | --- |
| `C:\CompanyData\IT` | `DREXZW\GG-IT-Users` | Modify |
| `C:\CompanyData\IT` | `DREXZW\GG-IT-Admins` | Full Control |
| `C:\CompanyData\HR` | `DREXZW\GG-HR-Users` | Modify |
| `C:\CompanyData\Finance` | `DREXZW\GG-Finance-Users` | Modify |
| `C:\CompanyData\Sales` | `DREXZW\GG-Sales-Users` | Modify |
| `C:\CompanyData\Management` | `DREXZW\GG-Management-Users` | Modify |

Permissions were granted with `(OI)(CI)` so they apply to files and subfolders created inside each department folder. Permissions are assigned to security groups rather than to individual users, so access changes become group-membership changes.

## Group Membership

Membership of each group was confirmed with `Get-ADGroupMember`:

| Group | Member |
| --- | --- |
| `GG-IT-Users` | John Smith (`jsmith`) |
| `GG-IT-Admins` | John Smith (`jsmith`) |
| `GG-HR-Users` | Sarah Johnson (`sjohnson`) |
| `GG-Finance-Users` | Michael Brown (`mbrown`) |
| `GG-Sales-Users` | David Wilson (`dwilson`) |
| `GG-Management-Users` | Robert Davis (`rdavis`) |

## Steps Performed

Evidence is in `screenshots/07-department-permissions/`.

1. **Create the department folders** with `New-Item` (`01-companydata-folders-created.png`).
2. **Confirm the groups and folders**, filtering for `GG` instead of scrolling through the 50+ built-in groups (`02-security-groups-and-folders-verified.png`).
3. **Capture the baseline ACL** of `C:\CompanyData` and `C:\CompanyData\IT` with `Get-Acl` (`03-baseline-acl-before-group-permissions.png`).
4. **Grant department permissions** with `icacls`: Modify for each department group, Full Control for `GG-IT-Admins` on the IT folder (`04-icacls-department-grants.png`).
5. **Verify group membership** with `Get-ADGroupMember` (`05-group-membership-verification.png`).
6. **Review the resulting ACLs** with `icacls` (`06-icacls-before-inheritance-cleanup.png`).
7. **Disable inheritance and remove `BUILTIN\Users`** on the five department folders (`07-inheritance-disabled-builtin-users-removed.png`).
8. **Verify the final ACLs** (`08-icacls-after-inheritance-cleanup.png`).

## Finding: Inherited Access Was Still Present

After the group grants were applied, the ACL output (step 6) showed that every department folder still carried inherited `BUILTIN\Users` entries from `C:\CompanyData`: read and execute, plus rights to add files and subfolders. On a Domain Controller, `BUILTIN\Users` is expected to cover ordinary domain users, so those entries would likely have weakened the separation between departments, even though each group's own grant had succeeded.

This was identified from the ACL output. It was not reproduced from a client session.

The fix was to disable inheritance on each department folder (which converts the inherited entries to explicit ones) and then remove `BUILTIN\Users`. The order matters: inherited entries cannot be removed from a child folder while it is still inheriting.

The final ACLs (step 8) contain only:

* The department group (`GG-<Dept>-Users`, plus `GG-IT-Admins` on the IT folder)
* `NT AUTHORITY\SYSTEM` (Full Control)
* `BUILTIN\Administrators` (Full Control)
* `CREATOR OWNER` (Full Control, inherit-only, applying to items users create)

No entry is marked as inherited `(I)`, and `BUILTIN\Users` no longer appears on any department folder. The IT folder has five entries; the other four folders have four each.

## Not Covered in This Stage

* The first set of `icacls` grant commands failed because the wrong domain prefix was used (`CORP\` instead of `DREXZW\`). The failed attempt was not captured in a screenshot (**not pictured**). The domain's NetBIOS name is shown in `screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png`, and the corrected commands are in `07-department-permissions/04-icacls-department-grants.png`. See `troubleshooting.md`.

---

# 8. SMB File Share and Shared-Folder Access Ticket (INC-AD-002)

This stage publishes the department folders over SMB and uses the share to practice a realistic Help Desk problem: a user whose NTFS permissions are correct but who still cannot write to a shared folder.

> **Scope:** The share is hosted on the Domain Controller (a lab simplification). It was tested from the Windows client as `DREXZW\sjohnson` (HR). Access for the other departments' users was not tested from the client.

## The Share

| Setting | Value |
| --- | --- |
| Share name | `CompanyData` |
| Local path | `C:\CompanyData` |
| Client path used in testing | `\\10.0.1.70\CompanyData` |
| Share permissions (after the fix) | `Everyone`: Read; `DREXZW\GG-HR-Users`: Change |

Two permission layers now control access, and the more restrictive of the two wins:

```text
User -> Share permissions (SMB) -> NTFS permissions -> Result
```

## The Incident

`INC-AD-002` is a simulated ticket. The fault was introduced deliberately: `sjohnson`'s NTFS permissions on the HR folder were correct (Modify through `GG-HR-Users`), but `GG-HR-Users` had only **Read** at the share level, so she could open the folder but not create files.

The troubleshooting followed this order:

1. Confirm who the user is on the client (`whoami`): `drexzw\sjohnson`.
2. Confirm the account and domain details (`Get-ADUser`, `NetBIOSName`).
3. Confirm group membership (`Get-ADGroupMember GG-HR-Users`).
4. Check NTFS permissions on the HR folder (`icacls`): `GG-HR-Users` has Modify.
5. Check share permissions (`Get-SmbShareAccess`): `GG-HR-Users` has Read.
6. Reproduce the failure from the client: creating a file returned "Access to the path ... is denied."
7. Fix at the share layer: `GG-HR-Users` changed from Read to Change.
8. Retest from the client: the file was created successfully.

**Root cause:** the share-level permission for `GG-HR-Users` was Read. NTFS allowed Modify, but the effective access over SMB is the more restrictive of the two.

The complete ticket is in:

```text
tickets/shared-folder-access-sarah-johnson.md
```

Evidence is in `screenshots/08-smb-share-access/`:

| # | What it shows |
| --- | --- |
| 01 | Domain details (NetBIOS name `DREXZW`) and `sjohnson`'s account in the HR OU |
| 02 | `GG-HR-Users` membership and the HR folder's NTFS ACL |
| 03 | The `CompanyData` SMB share created |
| 04 | Share permissions with `GG-HR-Users` set to Read |
| 05 | `sjohnson` denied when creating a file in HR |
| 06 | `GG-HR-Users` changed from Read to Change |
| 07 | `sjohnson` retest: file created successfully |

## Limitations

* Only `GG-HR-Users` was given Change on the share. The other department groups only have the `Everyone` Read entry at the share layer, so their write access over SMB has not been configured or tested.
* From `sjohnson`'s session, the share root listed all five department folder names (screenshot 07). Nothing in this stage tests whether she can open another department's folder.
* Screenshot 05 was captured after the fix and placed in incident order, so the failing condition was recreated for the capture. See the ticket's evidence notes.

---

# 9. Group Policy Not Applying Ticket (INC-AD-003)

This stage uses the existing Workstation Security Policy GPO to practice a common Group Policy problem: a GPO that exists and is configured correctly, but is not applying to the computer because its link is disabled.

> **Scope:** This is a controlled lab simulation, not a report from a real end user. The fault was introduced deliberately in Group Policy Management. Checks were run on the domain-joined computer `AD-CLIENT` from an elevated PowerShell session as the local `Administrator`, so the results are **computer-scope** only. Individual policy settings were not re-tested; the check is whether the GPO appears in the computer's applied-policy list.

## Environment

| Item | Value |
| --- | --- |
| Domain | `corp.drexzw.local` (NetBIOS: `DREXZW`) |
| Domain Controller | `AD-DC01` |
| Client computer | `AD-CLIENT` (in the Workstations OU) |
| GPO | Workstation Security Policy |
| Linked to | Workstations OU (not enforced) |

> **Note:** `gpresult` on `AD-CLIENT` reports `OS Configuration: Member Server` and `OS Version: 10.0.20348`.

## What Was Done

1. **Initial state:** `gpresult /r` on the client showed Workstation Security Policy and Default Domain Policy applied.
2. **Scope and link review:** In Group Policy Management, the GPO's Scope tab and the Workstations OU's Linked Group Policy Objects tab both showed the link enabled.
3. **Baseline before the fault:** `gpupdate /force` followed by `gpresult /r /scope computer` still showed both GPOs applied.
4. **Fault:** The link between Workstation Security Policy and the Workstations OU was disabled. The Linked Group Policy Objects tab then showed `Link Enabled: No`, while `GPO Status` stayed `Enabled`.
5. **Diagnosis:** After `gpupdate /force`, `gpresult /r /scope computer` listed only Default Domain Policy as applied. Workstation Security Policy appeared under the "filtered out" list with `Filtering: Disabled (Link)`.
6. **Fix:** The link was re-enabled (`Link Enabled: Yes`).
7. **Verification:** After `gpupdate /force`, `gpresult /r /scope computer` showed Workstation Security Policy applied again, alongside Default Domain Policy.

## Root Cause

The link between Workstation Security Policy and the Workstations OU had been disabled. The GPO itself was still enabled and unchanged, but a disabled link means the GPO does not apply to the OU.

## Key Takeaway

`gpupdate /force` refreshes policy, and `gpresult /r` shows the result of that refresh. They answer different questions. Here the refresh worked correctly; the `gpresult` output is what exposed the cause, because it names the reason a GPO was filtered out (`Disabled (Link)`).

## Evidence

The complete ticket is in:

```text
tickets/group-policy-not-applying.md
```

Screenshots are in `screenshots/09-group-policy-troubleshooting/`:

| # | What it shows |
| --- | --- |
| 01 | `gpresult /r`: initial applied policies |
| 02 | GPO Scope tab: link to Workstations enabled |
| 03 | Workstations OU: link enabled, GPO status Enabled |
| 04 | `gpupdate /force` and `gpresult`: both GPOs applied just before the fault |
| 05 | Link disabled (`Link Enabled: No`) |
| 06 | `gpresult`: Workstation Security Policy filtered out, `Disabled (Link)` |
| 07 | Link re-enabled (`Link Enabled: Yes`) |
| 08 | `gpresult`: Workstation Security Policy applied again |

Screenshot 05 was captured about a minute after screenshot 06, while the link was still disabled, and placed here in incident order. See the ticket's evidence notes.

## Not Covered in This Stage

* Only one computer was checked, and only computer-scope results. User-scope results were not part of this ticket.
* The individual settings inside Workstation Security Policy were not re-tested.
* The exact menu steps used to disable and re-enable the link were not captured (**not pictured**); the GUI state before and after is.

---

# Troubleshooting Experience

One of the main purposes of this lab was to practice troubleshooting rather than simply following installation steps.

Issues investigated during the lab included:

* Client DNS configuration and domain resolution
* Domain membership verification
* Active Directory computer/user visibility
* Domain authentication behavior
* Group Policy application and verification
* Account lockout behavior
* Differences between local and domain credentials
* Verifying changes from both the client and Domain Controller
* Wrong domain prefix in permission commands (`CORP\` vs `DREXZW\`)
* Inherited permissions weakening department folder separation
* Share-level (SMB) permission blocking a user whose NTFS permissions were correct
* A correctly configured GPO not applying because its link was disabled

Detailed troubleshooting notes are available in:

```text
troubleshooting.md
```

---

# Key Skills Demonstrated

### Active Directory

* Active Directory Domain Services
* Organizational Units (multi-department structure)
* Users and security groups
* Domain Controllers
* Active Directory Users and Computers (GUI)
* Domain authentication

### Windows Administration

* Windows Server 2022
* Windows client administration
* DNS configuration and troubleshooting
* Domain joining
* Group Policy
* Password policies
* Account lockout policies
* Group Policy troubleshooting (GPO links and OU scope, `gpresult`, `gpupdate`)

### File System and Share Permissions

* NTFS permissions (Modify, Full Control, inheritance flags)
* Group-based access control (permissions assigned to security groups, not individual users)
* Reviewing and correcting inherited permissions
* Creating SMB shares and managing share-level permissions
* Diagnosing share-versus-NTFS permission conflicts
* ACL and share-permission review with `icacls`, `Get-Acl`, and `Get-SmbShareAccess`

### Help Desk / IT Support

* Ticket-based troubleshooting
* User authentication troubleshooting
* Account lockout investigation
* Client-side diagnostics
* Server-side verification
* Root-cause analysis
* Reproducing a fault, correcting the cause, and verifying the result
* Documentation of troubleshooting steps

### PowerShell / Command Line

* `ipconfig`, `ipconfig /flushdns`
* `nslookup`
* `whoami`
* `systeminfo`
* `runas`
* `gpupdate /force`, `gpresult /r`, `gpresult /r /scope computer`
* Active Directory PowerShell cmdlets (`Get-ADDomain`, `Get-ADDomainController`, `Get-ADComputer`, `Get-ADOrganizationalUnit`, `Get-ADGroup`, `Get-ADGroupMember`)
* `icacls`, `Get-Acl`, `New-Item`, `Get-ChildItem`, `Test-Path`
* `New-SmbShare`, `Get-SmbShare`, `Get-SmbShareAccess`, `Grant-SmbShareAccess`

---

# Evidence Structure

Screenshots are organized according to the major stages of the lab, and numbered within each folder to reflect the order the steps were actually performed:

```text
screenshots/
├── 01-domain-controller/
├── 02-active-directory-structure/
├── 03-users-and-groups/
├── 04-client-domain-join/
├── 05-group-policy/
├── 06-account-lockout-policy/
├── 07-department-permissions/
├── 08-smb-share-access/
└── 09-group-policy-troubleshooting/
```

This organization makes it possible to follow the lab chronologically instead of presenting a flat collection of screenshots.

---

# Future Improvements

The lab now covers the core Active Directory environment, department-based NTFS permissions, an SMB share with a documented access ticket, and a documented Group Policy troubleshooting ticket.

Planned next steps:

* Configure share-level permissions for the remaining department groups and test them from the client
* Test cross-department access from the client (for example, confirming a user is denied another department's folder)
* More detailed Group Policy configurations (for example, mapping department drives with Group Policy Preferences)
* Additional Help Desk tickets

These additions will build on the existing domain rather than replacing the current environment.

---

# Conclusion

This lab demonstrates the process of building and troubleshooting a basic Windows Active Directory environment from the Domain Controller level through the client and end-user authentication level.

The main goal was not only to configure Active Directory, but to develop the troubleshooting workflow required when supporting Windows domain users.

The account-lockout scenario provides a practical Help Desk example where a user-facing authentication problem can be investigated through Group Policy, Active Directory Users and Computers, and client-side testing.

The permissions stages extend the lab from authentication to authorization: they show how group membership maps to folder access, why an ACL should be reviewed in full instead of assuming the grants alone produce the intended result, and how share-level and NTFS permissions combine when a user cannot write to a shared folder.

The Group Policy ticket adds a different kind of problem: a correctly configured GPO that is not applying. It shows the difference between refreshing policy (`gpupdate`) and reading the resulting state (`gpresult`), and that a GPO's link and scope are part of troubleshooting, not only its settings.
