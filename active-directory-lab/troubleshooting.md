# Active Directory Lab — Windows Server 2022

## Overview

This project is a hands-on Active Directory lab built to simulate a small business Windows domain environment and practice common IT Support / Help Desk administration tasks.

The lab uses a Windows Server 2022 instance as the Domain Controller and a Windows client joined to the domain. The environment was used to configure Active Directory, organizational units, users and groups, Group Policy, domain membership, account lockout behavior, and department-based NTFS permissions.

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

> **Scope:** These are local NTFS permissions on folders hosted on the Domain Controller (a lab simplification; a production environment would normally use a dedicated file server). The folders have **not** been shared over SMB, and access has **not yet been tested from the domain-joined client**. Everything below was verified with `icacls` and PowerShell on the Domain Controller.

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

### 1. Create the department folders

Five folders (IT, HR, Finance, Sales, Management) were created under `C:\CompanyData` using `New-Item`.

```text
screenshots/07-department-permissions/01-companydata-folders-created.png
```

### 2. Confirm the groups and folders

The department security groups were listed by filtering for `GG` (rather than scrolling through the 50+ built-in groups), and the folder list was checked.

```text
screenshots/07-department-permissions/02-security-groups-and-folders-verified.png
```

### 3. Capture the baseline ACL

Before changing anything, the ACLs on `C:\CompanyData` and `C:\CompanyData\IT` were captured with `Get-Acl`. Both showed the default entries, including `BUILTIN\Users`.

```text
screenshots/07-department-permissions/03-baseline-acl-before-group-permissions.png
```

### 4. Grant department permissions

Each department group was granted Modify on its own folder with `icacls`, and `GG-IT-Admins` was granted Full Control on the IT folder. The principals use the NetBIOS domain name (`DREXZW\`).

```text
screenshots/07-department-permissions/04-icacls-department-grants.png
```

### 5. Verify group membership

`Get-ADGroupMember` was run against all six groups to confirm each contains a user.

```text
screenshots/07-department-permissions/05-group-membership-verification.png
```

### 6. Review the resulting ACLs

`icacls` was run against each department folder. The new group entries were present, but so were inherited `BUILTIN\Users` entries (see the finding below).

```text
screenshots/07-department-permissions/06-icacls-before-inheritance-cleanup.png
```

### 7. Disable inheritance and remove `BUILTIN\Users`

On each of the five department folders, inheritance was disabled and the `BUILTIN\Users` entries were removed.

```text
screenshots/07-department-permissions/07-inheritance-disabled-builtin-users-removed.png
```

### 8. Final verification

`icacls` was run again on each folder to confirm the final ACLs.

```text
screenshots/07-department-permissions/08-icacls-after-inheritance-cleanup.png
```

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

* The folders are not shared over SMB, so share-level permissions are not part of this stage.
* Access was not tested by signing in as a department user on the client.
* The first set of `icacls` grant commands failed because the wrong domain prefix was used (`CORP\` instead of `DREXZW\`). The failed attempt was not captured in a screenshot (**not pictured**); the corrected commands are in `screenshots/07-department-permissions/04-icacls-department-grants.png`. See `troubleshooting.md`.

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

### File System Permissions

* NTFS permissions (Modify, Full Control, inheritance flags)
* Group-based access control (permissions assigned to security groups, not individual users)
* Reviewing and correcting inherited permissions
* ACL review and verification with `icacls` and `Get-Acl`

### Help Desk / IT Support

* Ticket-based troubleshooting
* User authentication troubleshooting
* Account lockout investigation
* Client-side diagnostics
* Server-side verification
* Root-cause analysis
* Documentation of troubleshooting steps

### PowerShell / Command Line

* `ipconfig`, `ipconfig /flushdns`
* `nslookup`
* `whoami`
* `systeminfo`
* `runas`
* Active Directory PowerShell cmdlets (`Get-ADDomain`, `Get-ADDomainController`, `Get-ADComputer`, `Get-ADOrganizationalUnit`, `Get-ADGroup`, `Get-ADGroupMember`)
* `icacls`, `Get-Acl`, `New-Item`, `Get-ChildItem`

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
└── 07-department-permissions/
```

This organization makes it possible to follow the lab chronologically instead of presenting a flat collection of screenshots.

---

# Future Improvements

The lab now covers the core Active Directory environment and department-based NTFS permissions on the Domain Controller.

Planned next steps:

* An access-related Help Desk ticket (access denied), diagnosed and verified from both the client and the server
* Client-side testing of the department folders (including SMB share access, if shares are added)
* More detailed Group Policy configurations

These additions will build on the existing domain rather than replacing the current environment.

---

# Conclusion

This lab demonstrates the process of building and troubleshooting a basic Windows Active Directory environment from the Domain Controller level through the client and end-user authentication level.

The main goal was not only to configure Active Directory, but to develop the troubleshooting workflow required when supporting Windows domain users.

The account-lockout scenario provides a practical Help Desk example where a user-facing authentication problem can be investigated through Group Policy, Active Directory Users and Computers, and client-side testing.

The department-permissions stage extends the lab from authentication to authorization: it shows how group membership maps to folder access, and why an ACL should be reviewed in full instead of assuming the grants alone produce the intended result.
