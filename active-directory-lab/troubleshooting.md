# Active Directory Lab — Troubleshooting

Every problem investigated during the lab, with the cause and what it taught. The four tickets (three resolved, one unresolved) are written up in full in [`tickets/`](tickets/); the others are short notes here.

## Summary

| # | Problem | Cause | Detail |
| --- | --- | --- | --- |
| 1 | Client not reliably talking to the domain | Client DNS was not pointing at the Domain Controller | [Below](#1-client-could-not-be-verified-against-the-domain) |
| 2 | Domain join needed verifying | Joining finishing is not proof of membership | [Below](#2-verifying-domain-membership) |
| 3 | Computer object location needed confirming | Checked with two tools, not assumed | [Below](#3-confirming-a-computer-objects-location) |
| 4 | Domain sign-in needed testing | Local and domain accounts had to be told apart | [Below](#4-domain-user-authentication) |
| 5 | GPO did not appear to apply at once | Policy had to be refreshed and checked | [Below](#5-group-policy-did-not-immediately-appear-to-apply) |
| 6 | Account locked out (INC-AD-001) | Failed sign-ins passed the threshold of 5 | [Ticket](tickets/account-lockout-sarah-johnson.md) |
| 7 | `icacls` grants failed | Wrong domain prefix: `CORP\` instead of `DREXZW\` | [Below](#7-permission-command-failed-wrong-domain-prefix) |
| 8 | Departments not fully separated | Inherited `BUILTIN\Users` entries on every folder | [Below](#8-inherited-permissions-weakened-folder-separation) |
| 9 | User can open a share but not save (INC-AD-002) | Share permission Read, NTFS Modify | [Ticket](tickets/shared-folder-access-sarah-johnson.md) |
| 10 | Policy missing on the workstation (INC-AD-003) | GPO link disabled | [Ticket](tickets/group-policy-not-applying.md) |
| 11 | Department drive maps not appearing (INC-AD-004, unresolved) | Not determined: user policy refresh fails with access denied (`0x5`) | [Below](#11-department-drive-maps-not-applying-unresolved-inc-ad-004) |

## Method

Work from the symptom outward, and always finish by testing again.

1. **Symptom:** what exactly is the user seeing?
2. **Client:** `ipconfig /all`, `nslookup corp.drexzw.local`, `whoami`, `systeminfo | findstr /B /C:"Domain"`, `gpresult /r`.
3. **Active Directory:** does the user exist, is the account enabled or locked, what groups is it in, which OU is the computer in?
4. **Fix:** make the smallest change that addresses the cause.
5. **Verify:** reproduce the original failure condition and confirm it is gone.

---

## 1. Client could not be verified against the domain

**Symptom:** the client did not give enough evidence that it was talking to the domain.
**Checked:** `ipconfig /all` on the client compared with the Domain Controller, then `nslookup corp.drexzw.local`, then connectivity to the Domain Controller.
**Fix:** the client was set to use the Active Directory DNS, after which the domain resolved.
**Lesson:** Active Directory depends on DNS. When a client cannot join or reach a domain, check DNS first.

## 2. Verifying domain membership

**Symptom:** the client needed to be confirmed as a domain member, not a workgroup machine.
**Checked:** `systeminfo | findstr /B /C:"Domain"` returned `corp.drexzw.local` ([screenshot](screenshots/04-client-domain-join/02-client-domain-membership.png)).
**Lesson:** do not assume a join worked because the wizard finished. Verify the end state from the client.

## 3. Confirming a computer object's location

**Symptom:** the client's computer object had to be confirmed in the Workstations OU.
**Checked:** ADUC with the filter set to Groups only (no computer visible), then with Computers included (`AD-CLIENT` visible). Cross-checked on the Domain Controller:

```powershell
Get-ADComputer -Filter 'Name -eq "AD-CLIENT"' -Properties DistinguishedName | Select-Object Name, DistinguishedName
Get-ADComputer -SearchBase "OU=Workstations,DC=corp,DC=drexzw,DC=local" -Filter * | Select-Object Name, DistinguishedName
```

**Lesson:** check from both the client and the server, and use more than one tool, before deciding an object is missing.

## 4. Domain user authentication

**Symptom:** a domain user had to be tested on the client, and told apart from a local account.
**Checked:** `whoami`, the domain membership, and the account in Active Directory.
**Lesson:** establish which account is in use, whether it is local or domain, which domain the computer is in, whether the Domain Controller is reachable, and whether the account exists and is enabled.

## 5. Group Policy did not immediately appear to apply

**Symptom:** a GPO was linked to the Workstations OU, but its effect was not visible straight away.
**Checked:** the link in Group Policy Management and the `LinkedGroupPolicyObjects` attribute with `Get-ADOrganizationalUnit`, then `gpupdate /force` on the client.
**Lesson:** creating a GPO is not the same as it applying. Check that it exists, is linked, the computer is in the right OU, refresh, and verify the result. A later, fuller case of this is INC-AD-003.

## 6. Account lockout (INC-AD-001)

Reproduced with repeated `runas` attempts using a wrong password, confirmed in ADUC, unlocked with the checkbox, and verified with `runas` and `whoami`. **Lesson:** unlocking is not the whole job. Ask why it locked (stale saved credentials, mapped drives, services using an old password). Full write-up: [ticket](tickets/account-lockout-sarah-johnson.md).

## 7. Permission command failed: wrong domain prefix

**Symptom:** the first `icacls` grants failed using `CORP\GG-IT-Users`. The failed attempt was not captured (**not pictured**).
**Cause:** `DOMAIN\name` needs the domain's **NetBIOS name**, `DREXZW`, not the first part of the DNS name (`corp`). The NetBIOS name is visible in [`08-smb-share-access/01`](screenshots/08-smb-share-access/01-domain-and-user-identity-verified.png).
**Fix:**

```powershell
icacls "C:\CompanyData\IT" /grant "DREXZW\GG-IT-Users:(OI)(CI)M"
```

The working commands are in [`07-department-permissions/04`](screenshots/07-department-permissions/04-icacls-department-grants.png).
**Lesson:** use the NetBIOS name. A wrong prefix produces errors that look like the group does not exist.

## 8. Inherited permissions weakened folder separation

**Symptom:** none reported. Found while reviewing the ACLs after the grants had succeeded.
**Found:** every department folder showed inherited `BUILTIN\Users` entries ([before](screenshots/07-department-permissions/06-icacls-before-inheritance-cleanup.png)). On a Domain Controller, `BUILTIN\Users` is expected to include ordinary domain users, so those entries would likely have let users outside a department read into its folder. This was concluded from the ACLs, not reproduced from a client.
**Fix:** disable inheritance on each folder, then remove `BUILTIN\Users`:

```powershell
icacls "C:\CompanyData\IT" /inheritance:d
icacls "C:\CompanyData\IT" /remove "BUILTIN\Users"
```

The order matters: inherited entries cannot be removed from a child folder while it is still inheriting. SYSTEM and Administrators keep Full Control, so nobody is locked out. [After](screenshots/07-department-permissions/08-icacls-after-inheritance-cleanup.png): only the department group, SYSTEM, Administrators and CREATOR OWNER remain.
**Lesson:** read the whole ACL, not just the entry you added. A grant can succeed while inherited entries still give others access.

## 9. Shared folder: can reach the share, cannot write (INC-AD-002)

The user's identity, group membership and NTFS permission were all correct. The share permission was Read, and over SMB the stricter of share and NTFS applies. Changed to Change and retested from the client. **Lesson:** check both permission layers. Full write-up: [ticket](tickets/shared-folder-access-sarah-johnson.md).

## 10. Policy not applying: GPO link disabled (INC-AD-003)

`gpupdate /force` worked, but `gpresult /r /scope computer` listed the GPO as filtered out with `Disabled (Link)`. Re-enabling the link fixed it. **Lesson:** `gpupdate` refreshes policy; `gpresult` shows what applied and why not. Check a GPO's link and scope, not only its settings. Full write-up: [ticket](tickets/group-policy-not-applying.md).

## 11. Department drive maps not applying: unresolved (INC-AD-004)

**Symptom:** the drives that the Department Drive Maps GPO should map did not appear, and `gpupdate /target:user /force` for `sjohnson` failed with `User Policy Update Failed.`
**Checked:** DNS and Domain Controller discovery (both worked), the HR share by its network path (reachable), `gpresult` for the user (no RSoP data), and the Group Policy event log (event 7320, error code `0x5`, access denied), repeated over several days. Evidence: [`10-gpo-drive-mapping-troubleshooting/`](screenshots/10-gpo-drive-mapping-troubleshooting/).
**Not established:** the cause. A GPO permission lead (Read without Apply Group Policy) was seen but not captured in a screenshot and not confirmed by a fix and retest. SYSVOL access by domain name was not confirmed either: the one pictured test returned `False`, possibly because of the path typed.
**Lesson:** a reachable share does not mean Group Policy will map it, so test the two layers separately. When a problem is not solved, stop repeating the same test and write down what was ruled in, what was ruled out, and what was never captured. Full write-up: [ticket](tickets/gpo-drive-mapping-not-applying.md).

---

## Lessons in one list

1. Start with evidence before changing anything.
2. DNS is fundamental to Active Directory.
3. Verify from both the client and the Domain Controller.
4. A GPO existing is not the same as it applying.
5. Reproduce the problem when you can, so the policy is observed, not assumed.
6. Always test again after the fix.
7. Use the NetBIOS name in `DOMAIN\principal` commands.
8. Read the full ACL, not just your own change.
9. Over SMB, check both the share and NTFS layers.
10. Check the GPO link, not only its settings.
11. A reachable share does not mean Group Policy will apply. Test the two separately.
12. Document an unresolved problem as unresolved: what was checked, what was found, and what is still unknown.
