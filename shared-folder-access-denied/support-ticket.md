# Support Ticket HD-2026-004

*Simulated help desk ticket for a lab environment.*

| Field | Details |
|---|---|
| Ticket ID | HD-2026-004 |
| Category | File Access / Permissions |
| Priority | Medium |
| Status | Closed: Resolved |
| Affected user | `employee2` |
| Affected resource | `C:\Company-Data\Finance` |
| Environment | Windows 11, standalone machine, local accounts, NTFS permissions |

---

## Summary

`employee2` cannot open the Finance folder and receives a permission error. Root cause: an explicit DENY entry on the folder for `employee2`. Resolved by removing the DENY entry.

---

## User Report

The user reported being unable to open the Finance folder. Windows displays a message saying they don't currently have permission to access it.

> Scenario premise: other users can open the folder. This was part of the ticket story and was not demonstrated in the lab screenshots.

## Business Impact

The user could not reach the company files stored in the Finance folder, which blocked any work depending on them.

---

## Troubleshooting Performed

### Step 1: Reproduce the issue

Opening the folder produced the message "You don't currently have permission to access this folder," with a Continue button that requires administrator elevation. *(Screenshot 04. The logged-in account is not visible in this screenshot.)*

### Step 2: Identify the affected account

```powershell
whoami
```

Result: `victor\employee2`. *(Screenshot 05)*

### Step 3: Inspect folder permissions

```powershell
icacls C:\Company-Data\Finance
```

Findings *(Screenshot 06)*:

- `Victor\employee2:(OI)(CI)(DENY)(Rc,RD,REA,X,RA)`: an explicit DENY for read-related rights, applied to the folder and its contents
- No explicit ALLOW entry for `employee2`
- `BUILTIN\Users:(I)(OI)(CI)(RX)`: an inherited Read & execute entry that the DENY overrides

### Step 4: Add an explicit allow

```powershell
icacls C:\Company-Data\Finance /grant "employee2:(RX)"
```

The command succeeded. The re-checked ACL showed **both** the DENY and the new `(RX)` entry for `employee2`. *(Screenshot 07, upper section)*

### Step 5: Remove the DENY entry

```powershell
icacls C:\Company-Data\Finance /remove:d "employee2"
```

The DENY entry was removed and the ACL re-checked. `employee2:(RX)` remained with no DENY entry. *(Screenshot 07, lower section)*

---

## Root Cause

`employee2` had an explicit DENY entry on the Finance folder for read-related rights. An explicit DENY takes precedence over ALLOW entries, including inherited ones, so the account was blocked despite the inherited `BUILTIN\Users` Read & execute permission.

## Resolution

Removed the DENY entry for `employee2` with `icacls /remove:d`.

An explicit `(RX)` entry was also added first. It is likely redundant given the inherited `Users` entry (not tested in isolation), so the DENY removal was the change that mattered.

## Verification

- The final ACL shows no DENY entry for `employee2`. *(Screenshot 07)*
- The Finance folder opens and lists `Payroll.xlsx`. *(Screenshot 08. The logged-in account is not visible in this screenshot.)*

---

## Evidence Limitations

Not captured in the screenshots:

- The step that created the DENY entry
- The logged-in account in screenshots 04 and 08
- An access re-test after the grant and before the DENY removal
- The first, unquoted grant attempt that failed with a PowerShell parsing error
- Another user opening the folder

Screenshots 01 to 03 show the folder on the Desktop, and screenshots 04 to 08 show it at `C:\Company-Data\Finance`. The move between locations was not captured.

---

## Technician Notes

Recommendations, not implemented in this lab:

- Assign permissions through security groups rather than individual accounts
- Avoid explicit DENY entries unless clearly needed
- Periodically review folder ACLs for leftover DENY entries
- Follow least-privilege access

**Related:** [Full write-up](./README.md) | [Commands and ACL notation](./commands.md)
