# Help Desk Ticket — Group Policy Not Applying

**Ticket ID:** INC-AD-003
**Status:** Resolved
**Priority:** Medium
**Category:** Group Policy / Workstation Configuration
**Environment:** Active Directory Lab
**Domain:** `corp.drexzw.local` (NetBIOS: `DREXZW`)
**Domain Controller:** `AD-DC01`
**Affected Computer:** `AD-CLIENT` (Workstations OU)
**Group Policy Object:** Workstation Security Policy
**Technician:** IT Support Lab
**Date:** October 2026

---

## 1. Reported Issue

The workstation was reported to be missing its expected security policy.

> **Lab note:** This was a controlled lab simulation, not a report from an actual end user. The fault was introduced deliberately so the troubleshooting could be practiced.

---

## 2. Initial Assessment

A missing policy on a domain-joined computer can come from several places: the computer is in the wrong OU, the GPO is not linked, the link is disabled, the GPO is filtered by security or WMI, or the client has not refreshed policy. The first step was to establish what the computer was actually applying, then check the GPO's scope and link.

---

## 3. Troubleshooting

### Step 1 — Check the Applied Policies

On the client, the applied policies were checked:

```powershell
gpresult /r
```

The computer settings listed two applied GPOs: **Workstation Security Policy** and **Default Domain Policy**. The computer is in `OU=Workstations,DC=corp,DC=drexzw,DC=local`, and policy was applied from `AD-DC01.corp.drexzw.local`.

Screenshot: `../screenshots/09-group-policy-troubleshooting/01-gpresult-initial-applied-policies.png`

---

### Step 2 — Check the GPO's Scope and Link

In Group Policy Management, the Workstation Security Policy's **Scope** tab showed one link, to the Workstations OU, with `Enforced: No` and `Link Enabled: Yes`.

Screenshot: `../screenshots/09-group-policy-troubleshooting/02-gpo-scope-link-enabled.png`

The Workstations OU's **Linked Group Policy Objects** tab showed the same link with `Link Enabled: Yes` and `GPO Status: Enabled`.

Screenshot: `../screenshots/09-group-policy-troubleshooting/03-workstations-ou-link-enabled.png`

---

### Step 3 — Record the State Before the Fault

Policy was refreshed and the computer-scope results were captured:

```powershell
gpupdate /force
gpresult /r /scope computer
```

Both Workstation Security Policy and Default Domain Policy were still applied.

Screenshot: `../screenshots/09-group-policy-troubleshooting/04-gpupdate-gpresult-before-fault.png`

---

### Step 4 — Simulate the Failure

In Group Policy Management, the link between Workstation Security Policy and the Workstations OU was disabled. The Workstations OU's Linked Group Policy Objects tab then showed `Link Enabled: No`, while `GPO Status` remained `Enabled`.

Screenshot: `../screenshots/09-group-policy-troubleshooting/05-gpo-link-disabled.png`

---

### Step 5 — Test and Diagnose from the Client

```powershell
gpupdate /force
gpresult /r /scope computer
```

`gpupdate /force` completed successfully for both computer and user policy. The `gpresult` output then showed:

* **Applied Group Policy Objects:** Default Domain Policy only
* **Filtered out:** Workstation Security Policy, with `Filtering: Disabled (Link)`
* **Filtered out:** Local Group Policy, with `Filtering: Not Applied (Empty)`

Because the refresh succeeded and the GPO was reported as filtered out because of its link, the cause was the link and not a failure to download policy.

Screenshot: `../screenshots/09-group-policy-troubleshooting/06-gpresult-gpo-filtered-disabled-link.png`

---

## 4. Root Cause

**Root Cause:**

The link between **Workstation Security Policy** and the **Workstations OU** had been disabled during the controlled troubleshooting exercise. The GPO itself still existed and was enabled, but a GPO whose link is disabled does not apply to the OU.

> **Lab note:** In a real environment, the technician would also find out who or what disabled the link (for example, a recent change in the change log) before simply re-enabling it.

---

## 5. Resolution

The link was re-enabled in Group Policy Management. The Workstations OU's Linked Group Policy Objects tab again showed `Link Enabled: Yes`.

Screenshot: `../screenshots/09-group-policy-troubleshooting/07-gpo-link-restored.png`

---

## 6. Validation

Policy was refreshed again on the client and the computer-scope results were re-checked:

```powershell
gpupdate /force
gpresult /r /scope computer
```

Workstation Security Policy and Default Domain Policy were both listed under Applied Group Policy Objects.

Screenshot: `../screenshots/09-group-policy-troubleshooting/08-gpresult-gpo-applied-after-restore.png`

The troubleshooting process therefore confirmed:

```text
Expected policy reported missing
      ↓
Checked applied policies (gpresult /r)
      ↓
Checked GPO scope and link (Group Policy Management)
      ↓
Recorded the baseline (both GPOs applied)
      ↓
Disabled the link (controlled fault)
      ↓
gpupdate /force + gpresult /r /scope computer
      ↓
Workstation Security Policy filtered out: Disabled (Link)
      ↓
Re-enabled the link
      ↓
gpupdate /force + gpresult /r /scope computer
      ↓
Workstation Security Policy applied again
```

---

## 7. Resolution Notes

**Resolution:** GPO link re-enabled on the Workstations OU.

**User impact (simulated):** The workstation was not receiving the Workstation Security Policy while the link was disabled.

**Final state:** `Link Enabled: Yes`; Workstation Security Policy and Default Domain Policy applied on `AD-CLIENT`.

**Validation:** `gpresult /r /scope computer` after `gpupdate /force`.

**Follow-up:** In production, review who changed the link and why, and consider whether GPO link changes should be tracked or alerted on.

---

## 8. Technician Notes

This incident was completed as a simulated Help Desk scenario within a personal Active Directory lab.

The purpose of the ticket was to practice a realistic workflow for "the expected policy is not applying":

1. Establish what the computer is actually applying
2. Check the GPO's scope and link
3. Record the baseline state
4. Reproduce the fault in a controlled way
5. Refresh policy and read the resulting state
6. Identify the cause from the reported filtering reason
7. Correct the cause
8. Verify the resulting state and document it

`gpupdate` and `gpresult` answer different questions: `gpupdate /force` refreshes policy, and `gpresult` shows what applied and, for filtered GPOs, why not.

**Scope limits of this ticket:**

* Checks were run on one computer (`AD-CLIENT`), from an elevated PowerShell session as the local `Administrator`, so the results are computer-scope. User-scope results were not part of this ticket.
* `gpresult` on `AD-CLIENT` reports `OS Configuration: Member Server` and `OS Version: 10.0.20348`.
* The individual settings inside Workstation Security Policy were not re-tested. The check was whether the GPO appears in the computer's applied-policy list.
* The exact menu steps used to disable and re-enable the link were not captured (**not pictured**). The Group Policy Management state before and after is pictured.

---

## 9. Evidence Notes

* Screenshot `05` (link disabled in Group Policy Management) was captured about a minute after screenshot `06` (the failing `gpresult`), while the link was still disabled. It is placed in incident order. The `Disabled (Link)` filtering reason in screenshot `06` independently shows the link was disabled at that point. Screenshot `07` (link restored) followed shortly after.
* The screenshots contain redacted (blacked-out) areas in the Group Policy Management window.

---

## 10. Alternative / Production Approach (Not Pictured in This Run)

The link state could also be checked and corrected with the Group Policy PowerShell module (on a Domain Controller or with RSAT):

```powershell
# Show GPO links and inheritance on the Workstations OU
Get-GPInheritance -Target "OU=Workstations,DC=corp,DC=drexzw,DC=local"

# Re-enable the link
Set-GPLink -Name "Workstation Security Policy" -Target "OU=Workstations,DC=corp,DC=drexzw,DC=local" -LinkEnabled Yes
```

These commands are included for reference only. They were not run as part of this ticket and are not shown in any screenshot.

---

## 11. Skills Demonstrated

* Group Policy troubleshooting
* GPO links and OU scope
* `gpupdate` and `gpresult`
* Root-cause identification
* Configuration recovery and verification
* Technical documentation

---

## 12. Lessons Learned

A Group Policy Object can exist and be correctly configured but fail to apply when its link is disabled. Troubleshooting requires checking policy scope and links, reproducing the fault, correcting the cause, and verifying the resulting state.

---

## 13. Evidence

Supporting screenshots are stored in:

```text
../screenshots/09-group-policy-troubleshooting/
```

| Step | Screenshot |
|---|---|
| Initial applied policies (`gpresult /r`) | `01-gpresult-initial-applied-policies.png` |
| GPO scope: link enabled | `02-gpo-scope-link-enabled.png` |
| Workstations OU: link enabled | `03-workstations-ou-link-enabled.png` |
| Baseline before the fault | `04-gpupdate-gpresult-before-fault.png` |
| Link disabled | `05-gpo-link-disabled.png` |
| `gpresult`: filtered out, `Disabled (Link)` | `06-gpresult-gpo-filtered-disabled-link.png` |
| Link restored | `07-gpo-link-restored.png` |
| `gpresult`: GPO applied again | `08-gpresult-gpo-applied-after-restore.png` |
