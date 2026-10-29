# Help Desk Ticket Scenarios

Two scenario-based tickets showing how I would troubleshoot common Microsoft 365 issues: a sign-in failure and a file access problem.

> **Evidence note:** These are written troubleshooting procedures for reported scenarios. They were not performed in a live tenant, and no screenshots are included (not pictured).

---

## Ticket A: New Employee Cannot Sign In to Microsoft 365

**Reported issue**
"My password works on my personal account, but I can't sign into my Microsoft 365 account."

**Category:** Account access / sign-in failure

### Troubleshooting steps

1. **Confirm which account the user is signing into.**
   Ask whether they are using their personal account or their work (Microsoft 365) account, and whether they are entering the work credentials (work email address and the password issued for that account). A password that works for a personal account will not work for the work account.

2. **Confirm the correct sign-in page.**
   Check that the user is going to the corporate sign-in portal and not a personal Microsoft sign-in page or a saved/old link.

3. **Check the account status.**
   Look for any of the following:
   - Account locked out
   - Sign-in blocked
   - Password expired under the password policy

4. **Confirm the account is active, then reset the password if needed.**
   If the account is active but the corporate password still fails, reset the password and have the user try signing in again.

### Verification
Ask the user to sign in with the new credentials and confirm they can reach Microsoft 365.

### Resolution
Account active and sign-in working after confirming credentials, portal, and account status, or after a password reset.

---

## Ticket B: Access Denied When Opening a Shared Document in Teams

**Reported issue**
"Access denied when trying to open a shared document."

**Category:** File access / permissions

### Troubleshooting steps

1. **Get a screenshot of the error.**
   Ask the user for a screenshot showing the exact error message.

2. **Find out whether anyone else is affected.**
   Check whether other users have the same problem. If many people are affected, it may be a wider issue rather than one person's permissions.

3. **Narrow down the scope.**
   Check whether the problem happens with one specific file or folder, or with other files as well.

4. **Bypass the Teams app.**
   Have the user open the file in a web browser or by signing into SharePoint or Microsoft 365 directly. This helps show whether the problem is the Teams app or the access itself.

5. **Check the underlying SharePoint permissions.**
   Confirm whether the user has the required permissions on the file or folder. If not, add them by following the company's access procedure and approval policy.

### Verification
Ask the user to open the file again and confirm access works.

### Resolution
Access restored by correcting the user's SharePoint permissions, or the cause identified (for example a Teams app issue if the browser works).
