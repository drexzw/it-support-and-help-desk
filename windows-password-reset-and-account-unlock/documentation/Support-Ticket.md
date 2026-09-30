# Help Desk Support Ticket

*This is a simulated ticket created for a home lab. The user and the report are fictional.*

## Ticket Information

| Field | Value |
| ----- | ----- |
| **Ticket ID** | HD-001 |
| **Category** | User Account Management |
| **Priority** | Medium |
| **Affected account** | `jsmith` (local account) |
| **Status** | Resolved: password reset applied and confirmed. User sign-in not pictured. |

---

## User Report

**Submitted by:** John Smith

> Hi IT,
>
> I forgot my Windows password after returning from vacation. After several failed login attempts, I can no longer access my account. I have an important report due today.
>
> Thank you.

---

## Investigation

| Check | Result | Evidence |
| ----- | ------ | -------- |
| Account exists | `jsmith` present in the local user list | [Screenshot 03](screenshots/03-user-information.png) |
| Account active | Account active = Yes | [Screenshot 03](screenshots/03-user-information.png) |
| Password state | Password last set 03/08/2026 14:02:35, expires 14/09/2026 | [Screenshot 03](screenshots/03-user-information.png) |
| Previous sign-in | Last logon = Never | [Screenshot 03](screenshots/03-user-information.png) |
| Group membership | Local group `Users` only | [Screenshot 03](screenshots/03-user-information.png) |

**Finding:** The account exists, is active, and its password has not expired, so the account is not disabled and the password is not expired. A password reset was chosen as the resolution.

**Note on the lockout:** The user mentions failed login attempts. No lockout state was reproduced or pictured in this lab, so a lockout was not confirmed. Lockout handling is documented in the [Active Directory lockout ticket](../active-directory-lab/tickets/account-lockout-sarah-johnson.md).

---

## Resolution

1. Opened Command Prompt as Administrator ([Screenshot 00](screenshots/00-command-prompt-admin.png)).
2. Reset the account password to a temporary password with `net user` ([Screenshot 04](screenshots/04-password-reset.png)). The command returned "The command completed successfully."
3. Re-ran `net user jsmith` and confirmed **Password last set** changed from 14:02:35 to 14:08:45 ([Screenshot 05](screenshots/05-password-last-set.png)).
4. Confirmed the account remained active after the reset ([Screenshot 05](screenshots/05-password-last-set.png)).

---

## Not Pictured

* Verifying the caller's identity before the reset
* Delivering the temporary password to the user
* The user signing in with the temporary password (Last logon = Never in screenshots 05 and 07)
* Forcing a password change at next sign-in

---

## Work Notes

Only two timestamps appear in the screenshots, so other entries are listed in order without times.

| Time | Action | Evidence |
| ---- | ------ | -------- |
| n/a | Opened an elevated Command Prompt | [00](screenshots/00-command-prompt-admin.png) |
| n/a | Listed local users to see the starting state | [01](screenshots/01-existing-users.png) |
| n/a | Created the test account `jsmith` for the scenario | [02](screenshots/02-user-created.png) |
| 14:02:35 | Reviewed `jsmith` account details (password last set at this time) | [03](screenshots/03-user-information.png) |
| n/a | Reset the password | [04](screenshots/04-password-reset.png) |
| 14:08:45 | Confirmed the new Password last set value | [05](screenshots/05-password-last-set.png) |

---

## Additional Exercise: Disable and Re-enable

After the reset, I practiced the account status controls on the same account. This is separate from the user's reported issue.

* Disabled the account with `/active:no`; `net user jsmith` then showed Account active = No ([Screenshot 06](screenshots/06-account-disabled.png)).
* Re-enabled it with `/active:yes`; the account showed Account active = Yes again ([Screenshot 07](screenshots/07-account-enabled.png)).

---

## Cleanup

The test account `jsmith` was deleted after the lab and no longer appears in the user list ([Screenshot 08](screenshots/08-user-deleted.png)).

---

## Resolution Summary

The password for `jsmith` was reset and the change was confirmed by the Password last set timestamp. The account stayed active throughout the reset. The user's sign-in with the temporary password and the identity-verification step were not captured in this lab.

**Final Status:** Resolved (reset confirmed). Sign-in verification not pictured.
