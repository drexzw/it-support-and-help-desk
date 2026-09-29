# Commands and Troubleshooting Log

Commands used in this lab, in the order they appear in the screenshots, with the reasoning behind each. Output shown is copied from the screenshots. Anything not captured is listed separately at the end.

---

## 1. Account setup (screenshot 01)

Run in an elevated PowerShell session ("Administrator: Windows PowerShell").

### Create the test accounts

```powershell
net user employee1 <password> /add
net user employee2 <password> /add
```

Creates two local accounts to simulate employees. Each returned `The command completed successfully.`

> Passwords are redacted in the screenshot and replaced with `<password>` here. Typing a password directly on the command line leaves it in the console and shell history. `net user employee1 * /add` prompts for the password instead.

### List local accounts

```powershell
net user
```

Confirms both accounts exist. The output lists `Administrator`, `DefaultAccount`, `employee1`, `employee2`, `Guest`, `WDAGUtilityAccount`, and the technician's own account (redacted).

---

## 2. Identify the affected account (screenshot 05)

Run in a standard (non-elevated) PowerShell session.

```powershell
whoami
```

Output:

```
victor\employee2
```

Confirms which account was experiencing the problem.

---

## 3. Inspect permissions (screenshot 06)

Run in an elevated PowerShell session.

```powershell
icacls C:\Company-Data\Finance
```

Output:

```
C:\Company-Data\Finance Victor\employee2:(OI)(CI)(DENY)(Rc,RD,REA,X,RA)
                        Victor\employee1:(OI)(CI)(RX)
                        BUILTIN\Administrators:(I)(OI)(CI)(F)
                        NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
                        BUILTIN\Users:(I)(OI)(CI)(RX)
                        NT AUTHORITY\Authenticated Users:(I)(M)
                        NT AUTHORITY\Authenticated Users:(I)(OI)(CI)(IO)(M)
```

The first line is the problem: `employee2` has an explicit DENY, and no explicit ALLOW entry.

### Reading the `icacls` output

| Notation | Meaning |
|---|---|
| `(OI)` | Object inherit: applies to files inside the folder |
| `(CI)` | Container inherit: applies to subfolders |
| `(IO)` | Inherit only: applies to children, not the folder itself |
| `(I)` | Permission is inherited from a parent folder |
| `(F)` | Full control |
| `(M)` | Modify |
| `(RX)` | Read and execute |
| `(DENY)` | The entry blocks the listed rights |
| `Rc` | Read permissions |
| `RD` | Read data / list folder |
| `REA` | Read extended attributes |
| `X` | Execute / traverse |
| `RA` | Read attributes |

---

## 4. Add an explicit allow, then remove the DENY (screenshot 07)

### Grant Read and Execute

```powershell
icacls C:\Company-Data\Finance /grant "employee2:(RX)"
```

Output: `processed file: C:\Company-Data\Finance` and `Successfully processed 1 files; Failed processing 0 files`.

The permission string is wrapped in quotes because PowerShell treats unquoted parentheses as its own syntax.

Re-running `icacls` afterwards showed **both** entries for `employee2` in the ACL:

```
Victor\employee2:(OI)(CI)(DENY)(Rc,RD,REA,X,RA)
Victor\employee2:(RX)
```

Note that the new entry is `(RX)` with **no `(OI)(CI)`**, so it applies to the folder itself only, not to its contents. The DENY entry does have `(OI)(CI)`.

### Remove the DENY entry

```powershell
icacls C:\Company-Data\Finance /remove:d "employee2"
```

`/remove:d` removes only the DENY entries for the named user and leaves ALLOW entries in place.

### Verify

```powershell
icacls C:\Company-Data\Finance
```

Output (first lines):

```
C:\Company-Data\Finance Victor\employee2:(RX)
                        Victor\employee1:(OI)(CI)(RX)
                        BUILTIN\Administrators:(I)(OI)(CI)(F)
                        ...
```

The DENY entry is gone.

---

## Troubleshooting Sequence Summary

1. `whoami` to confirm the affected account (`employee2`)
2. `icacls` to read the full ACL and find the explicit DENY
3. `icacls /grant "employee2:(RX)"` to add an explicit allow; the ACL then shows both entries
4. `icacls /remove:d "employee2"` to remove the DENY
5. `icacls` again to confirm the DENY is gone; the folder then opens (screenshot 08)

## Key Takeaway

An explicit DENY takes precedence over ALLOW entries, including inherited ones such as `BUILTIN\Users:(I)(OI)(CI)(RX)`. A successful `/grant` does not fix a DENY. Read the whole ACL before changing anything.

---

## Not Pictured

These parts of the lab were **not captured** in screenshots:

- **The command that created the DENY entry for `employee2`.** Screenshot 03 shows the Security list before `employee2` appears in the ACL, and screenshot 06 shows the DENY in place. The step between them was not recorded.
- **The first grant attempt without quotes**, which failed with a PowerShell parsing error (`CommandNotFoundException`) before the quoted version was used:
  ```powershell
  icacls C:\Company-Data\Finance /grant employee2:(RX)
  ```
- **An access re-test after the grant and before the DENY removal.**
- **Which account was logged in for screenshots 04 and 08.**
