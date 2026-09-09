# Active Directory Lab — Windows Server 2022

## Overview

This project is a hands-on Active Directory lab built to simulate a small business Windows domain environment and practice common IT Support / Help Desk administration tasks.

The lab uses a Windows Server 2022 instance as the Domain Controller and a Windows client joined to the domain. The environment was used to configure Active Directory, organizational units, users and groups, Group Policy, domain membership, and account lockout behavior.

The lab also includes troubleshooting scenarios to practice diagnosing issues from both the Domain Controller and client side.

> **Environment:** Personal lab / simulated business environment
> **Domain:** `corp.drexzw.local`

---

## Objectives

* Deploy and configure a Windows Server 2022 Domain Controller
* Install Active Directory Domain Services (AD DS)
* Create and organize Active Directory OUs
* Create domain users and security groups
* Join a Windows client to the domain
* Configure and verify Group Policy
* Configure password and account lockout policies
* Test domain authentication
* Troubleshoot Active Directory and domain-join issues
* Simulate a Help Desk account-lockout ticket
* Document troubleshooting procedures and commands

---

## Lab Environment

| Component              | Configuration                         |
| ---------------------- | ------------------------------------- |
| Domain Controller      | Windows Server 2022                   |
| Client                 | Windows workstation                   |
| Domain                 | `corp.drexzw.local`                   |
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
      +------+-------+           Domain Joined
      |              |
   IT OU        Workstations OU
      |              |
   jsmith       Client Computer
      |
 GG-IT-Users
```

---

# Implementation

## 1. Domain Controller Deployment

A Windows Server 2022 EC2 instance was deployed and configured as the Domain Controller.

The server was configured with the appropriate network settings before installing Active Directory Domain Services.

Evidence:

* Static/network configuration
* AD DS installation
* Domain Controller verification

Screenshots:

```text
screenshots/01-domain-controller/dc-ip-configuration.png
screenshots/01-domain-controller/ad-ds-installation.png
screenshots/01-domain-controller/domain-controller-verification.png
```

---

## 2. Active Directory Structure

After installing AD DS, the Active Directory Users and Computers console was used to create a basic organizational structure.

The environment included separate organizational units for areas such as IT users and workstations.

Evidence:

```text
screenshots/02-active-directory-structure/aduc-ou-structure.png
```

This structure provides a foundation for applying different administrative policies to users and computers.

---

## 3. Users and Groups

A domain user was created for testing authentication and Group Policy behavior.

The lab included:

* User: `jsmith`
* Group: `GG-IT-Users`
* IT organizational structure

Evidence:

```text
screenshots/03-users-and-groups/jsmith-it-user.png
screenshots/03-users-and-groups/gg-it-users-group.png
```

Using security groups provides a scalable way to assign permissions and policies rather than managing users individually.

---

## 4. Client Domain Join

A Windows client was configured to communicate with the Domain Controller and joined to the `corp.drexzw.local` domain.

DNS resolution and domain membership were verified from the client.

Evidence:

```text
screenshots/04-client-domain-join/client-dns-resolution.png
screenshots/04-client-domain-join/client-domain-membership.png
```

Domain authentication was then tested using the domain user.

Evidence:

```text
screenshots/04-client-domain-join/domain-user-login-attempt.png
screenshots/04-client-domain-join/domain-user-login-verification.png
screenshots/04-client-domain-join/domain-user-gpo-session-verification.png
```

---

# 5. Group Policy

A Group Policy Object was created and linked to the appropriate workstation organizational unit.

The policy was used to configure password/account security requirements and verify that the policy was being applied to the client.

Evidence:

```text
screenshots/05-group-policy/gpo-linked-to-workstations.png
screenshots/05-group-policy/password-policy-settings.png
screenshots/05-group-policy/gpo-application-verification.png
```

The client was refreshed and the resulting policy application was verified.

---

# 6. Account Lockout Help Desk Scenario

A simulated Help Desk incident was created involving a user account becoming locked out after repeated unsuccessful authentication attempts.

The scenario was used to practice:

* Identifying an account lockout
* Understanding the configured lockout policy
* Reproducing the behavior in the lab
* Verifying the account state
* Troubleshooting authentication failures
* Restoring account access
* Validating successful authentication after remediation

The complete ticket is documented in:

```text
tickets/account-lockout-sarah-johnson.md
```

Supporting evidence is stored in:

```text
screenshots/06-account-lockout/
```

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

Detailed troubleshooting notes are available in:

```text
troubleshooting.md
```

---

# Key Skills Demonstrated

### Active Directory

* Active Directory Domain Services
* Organizational Units
* Users and security groups
* Domain Controllers
* Active Directory Users and Computers
* Domain authentication

### Windows Administration

* Windows Server 2022
* Windows client administration
* DNS configuration and troubleshooting
* Domain joining
* Group Policy
* Password policies
* Account lockout policies

### Help Desk / IT Support

* Ticket-based troubleshooting
* User authentication troubleshooting
* Account lockout investigation
* Client-side diagnostics
* Server-side verification
* Root-cause analysis
* Documentation of troubleshooting steps

### PowerShell / Command Line

* `ipconfig`
* `nslookup`
* `whoami`
* `gpupdate`
* `gpresult`
* Active Directory PowerShell cmdlets
* Windows account-management commands

---

# Evidence Structure

Screenshots are organized according to the major stages of the lab:

```text
screenshots/
├── 01-domain-controller/
├── 02-active-directory-structure/
├── 03-users-and-groups/
├── 04-client-domain-join/
├── 05-group-policy/
└── 06-account-lockout/
```

This organization makes it possible to follow the lab chronologically instead of presenting a flat collection of screenshots.

---

# Future Improvements

The current lab establishes the core Active Directory environment.

Future improvements will include:

* NTFS permissions
* File-share permissions
* Department-specific access control
* Additional user roles
* More detailed Group Policy configurations
* Permission troubleshooting scenarios
* Additional Help Desk tickets

These additions will build on the existing domain rather than replacing the current environment.

---

# Conclusion

This lab demonstrates the process of building and troubleshooting a basic Windows Active Directory environment from the Domain Controller level through the client and end-user authentication level.

The main goal was not only to configure Active Directory, but to develop the troubleshooting workflow required when supporting Windows domain users.

The account-lockout scenario provides a practical Help Desk example where a user-facing authentication problem can be investigated through Active Directory, Group Policy, client-side testing, and account-state verification.
