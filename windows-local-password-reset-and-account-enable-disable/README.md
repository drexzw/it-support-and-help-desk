# Windows Local Password Reset & Account Enable/Disable

## Project Overview

This lab simulates a Help Desk request in which a user reports being unable to sign in after forgetting their Windows password. Using an elevated Command Prompt, I created a test account, checked its status, reset its password, confirmed the reset, disabled and re-enabled the account, and then removed the test account.

Everything in this lab was done on a **local Windows 11 machine** using the built-in `net user` command. No Active Directory or Windows Server was involved. For the domain-based version of account handling (Group Policy lockout settings and a documented lockout ticket), see the [Active Directory lab](../active-directory-lab/README.md).

---

## Scenario

A simulated user (John Smith, account `jsmith`) reports that he forgot his password after returning from vacation. As the Help Desk technician, I verified that the account exists and is active, reset the password to a temporary one, and confirmed the reset by checking the account's **Password last set** value.

---

## Environment

| Item | Detail |
| ---- | ------ |
| Operating system | Windows 11 (build 10.0.26200.8875, shown in screenshot 00) |
| Account type | Local user account (`jsmith`), member of the local `Users` group |
| Tool | Command Prompt, run as Administrator |
| Command | `net user` |

---

## Evidence

| # | Screenshot | What it shows |
| - | ---------- | ------------- |
| 00 | [Admin Command Prompt](screenshots/00-command-prompt-admin.png) | Elevated Command Prompt opened |
| 01 | [Existing users](screenshots/01-existing-users.png) | `net user` listing local accounts before `jsmith` exists |
| 02 | [User created](screenshots/02-user-created.png) | `jsmith` created with `/add` |
| 03 | [User information](screenshots/03-user-information.png) | `net user jsmith`: Account active = Yes, Password last set = 03/08/2026 14:02:35, Last logon = Never |
| 04 | [Password reset](screenshots/04-password-reset.png) | Password reset command completed successfully |
| 05 | [Password last set](screenshots/05-password-last-set.png) | Password last set changed to 03/08/2026 14:08:45, confirming the reset |
| 06 | [Account disabled](screenshots/06-account-disabled.png) | `/active:no` applied, Account active = No |
| 07 | [Account enabled](screenshots/07-account-enabled.png) | `/active:yes` applied, Account active = Yes |
| 08 | [User deleted](screenshots/08-user-deleted.png) | Test account removed, no longer listed by `net user` |

Dates are shown as displayed by the system (DD/MM/YYYY).

---

## Not Pictured

To keep this documentation accurate, the following are **not** shown in the screenshots and are not claimed as completed:

* A lockout caused by failed sign-in attempts (the user report mentions failed attempts, but no lockout state was reproduced)
* Unlocking a locked account
* Identity verification of the caller
* The user signing in with the temporary password (Last logon still shows "Never" in screenshots 05 and 07)
* Forcing a password change at next sign-in

---

## Skills Demonstrated

* Creating and removing local user accounts with `net user`
* Reading account status: active state, password last set, password expiry, group membership
* Resetting a local account password from an elevated Command Prompt
* Verifying a change using evidence from the system (Password last set timestamp)
* Disabling and re-enabling an account
* Writing a Help Desk ticket that separates what was done from what was not verified

---

## Repository Contents

| File | Purpose |
| ---- | ------- |
| **README.md** | Overview, evidence map, and limitations |
| **commands.md** | Commands used, with the screenshot each one appears in |
| **support-ticket.md** | Simulated ticket HD-001 with investigation, resolution, and work notes |
| **screenshots/** | Numbered evidence for each step |

---

## What I Learned

* `net user <username>` shows the fields a technician needs first: whether the account is active, when the password was last set, when it expires, and whether a sign-in has ever happened.
* The **Password last set** timestamp gives objective proof that a reset took place. It changed from 14:02:35 to 14:08:45 between screenshots 03 and 05.
* A disabled account and a locked-out account are different problems. Disabling (`/active:no`) is an administrator action, while a lockout is triggered by failed sign-ins under a lockout policy. This lab covers the first; the Active Directory lab covers the second.
* Typing a password directly into a command puts it on screen and in the console history. In a real environment I would avoid that.

---

## Future Improvements

* Reproduce a real local lockout by setting a lockout threshold, failing sign-ins on purpose, and documenting the unlock
* Add a screenshot of the user signing in with the temporary password
* Force a password change at next sign-in
* Document an identity-verification step before any reset
* Handle the same request in Active Directory (see the [AD lab](../active-directory-lab/README.md))
