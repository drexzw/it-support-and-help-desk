# Support Ticket — Developer Group Access Issue

| Field | Detail |
|---|---|
| **Ticket #** | IT-2026-0091 |
| **Reported by** | John Smith (Developers) |
| **Assigned to** | IT Support |
| **Priority** | Medium |
| **Status** | Closed — Resolved |

---

## Summary

User `john.smith` reported that he could no longer access the shared `developers` directory at `/shared/company/developers`, a resource he needs regularly for his work. He confirmed he had not changed anything on his own account.

## Reported Symptoms

* Running `cd developers` from `/shared/company` returns `Permission denied`.
* User states he previously had access to this directory.
* Access to `/shared/company/public` is unaffected.

## Investigation

1. Logged in as `john.smith` and reproduced the issue: `cd developers` returned `Permission denied`.
2. Checked group membership with `id john.smith` — output showed only `groups=1006(john.smith)`, with no `developers` group listed.
3. Checked directory permissions with `ls -ld developers` — the folder is owned `root:developers` with mode `770`, meaning access is restricted to members of the `developers` group.
4. User attempted a self-service fix (`sudo usermod -aG developers john.smith`) but was correctly blocked, since standard users are not sudoers on this system.

**Evidence:** `screenshots/06-troubleshooting/01-repro-and-diagnosis-group-membership-missing.png`

## Root Cause

`john.smith`'s account was not (or was no longer) a member of the `developers` secondary group. Since access to `/shared/company/developers` is controlled entirely by group membership (`770`, owned by `root:developers`), the missing group membership caused the permission denial.

## Resolution

1. As an administrator, restored group membership:
   ```bash
   sudo usermod -aG developers john.smith
   ```
2. Verified the change:
   ```bash
   groups john.smith
   ```
3. Had the user start a fresh session (`su - john.smith`) so the updated group membership would take effect.
4. Confirmed the user could access the directory and read its contents:
   ```bash
   cd /shared/company/developers && cat project-notes.txt
   ```

**Evidence:** `screenshots/06-troubleshooting/02-fix-applied-and-verified.png`

## Verification

* User confirmed restored access to `developers`.
* Re-tested the `finance` directory to confirm it remained correctly inaccessible to `john.smith`, preserving least-privilege boundaries.

## Notes

This ticket reflects a common real-world scenario: a user's secondary group membership being dropped or never applied correctly, producing symptoms that look like a permissions bug but are actually an identity/group-membership issue. Checking `id`/`groups` before touching `chmod`/`chown` is the fastest way to tell the two apart.
