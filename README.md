# IT Support & Help Desk Portfolio

Hands-on labs that practice what an entry-level IT Support / Help Desk technician does every day: fix access problems, troubleshoot Windows and Linux systems, and write it all up like a real ticket.

Every lab follows the same flow: **reported issue → investigation → root cause → fix → verification**, with screenshots as evidence.

---

## ⭐ Start here: Active Directory & Help Desk Administration

A small-business Windows domain (`corp.drexzw.local`) built from scratch on AWS EC2: a Windows Server 2022 Domain Controller and a domain-joined client. Most of what a Help Desk technician touches is in it: users, groups, file shares, permissions, Group Policy, and four tickets.

**What I did in this lab:**

* Built the domain: Active Directory, DNS, 7 OUs, users, and a security group for each department
* Joined a Windows client to the domain and verified the join from both the client and the server
* Created and linked Group Policy for password and account lockout settings, and verified it with `gpresult`
* Set up department folders and permissions with `icacls`, and found and fixed an inheritance problem during review
* Worked four Help Desk tickets: reproduced the problem, found the cause, fixed it, and proved the fix
* Wrote up one problem that is still unsolved, with what was proven, what is only a lead, and what was not captured

**Short on time?** Read the [lab README](active-directory-lab/README.md), then [INC-AD-002](active-directory-lab/tickets/shared-folder-access-sarah-johnson.md) (a solved ticket) and [INC-AD-004](active-directory-lab/tickets/gpo-drive-mapping-not-applying.md) (an unsolved one).

| Ticket | Problem | Outcome |
| --- | --- | --- |
| [INC-AD-001](active-directory-lab/tickets/account-lockout-sarah-johnson.md) | "My account is locked." | Resolved: lockout threshold exceeded, account unlocked and verified |
| [INC-AD-002](active-directory-lab/tickets/shared-folder-access-sarah-johnson.md) | "I can open the HR folder but can't save." | Resolved: share permission was Read, and the stricter permission wins |
| [INC-AD-003](active-directory-lab/tickets/group-policy-not-applying.md) | "My workstation is missing its policy." | Resolved: the GPO link was disabled |
| [INC-AD-004](active-directory-lab/tickets/gpo-drive-mapping-not-applying.md) | "My department drive didn't map." | **Unresolved.** Investigated and documented honestly: what was proven, what is a lead, what was not captured |

All of it is backed by 56 numbered screenshots, grouped by stage.

**[Open the Active Directory lab →](active-directory-lab/)**

---

## More labs

| Lab | Scenario | Skills |
| --- | --- | --- |
| [Shared Folder Access Denied](shared-folder-access-denied/) | A user cannot reach a shared business resource | User and group investigation, shared folder troubleshooting |
| [NTFS Permissions](ntfs-permissions-lab/) | Users need different access based on their role | NTFS permissions, inheritance, Windows Server, Remote Desktop |
| [Windows Local Password Reset & Account Enable/Disable](windows-local-password-reset-and-account-enable-disable/) | A user cannot sign in after forgetting their password | `net user`, password reset, verification with "Password last set" |
| [DNS Troubleshooting](dns-troubleshooting-lab/) | A server is online but cannot resolve names | Telling a connectivity problem from a DNS problem, `nslookup`, `ping` |
| [DHCP / IP Configuration](dhcp-ip-configuration-and-troubleshooting-lab/) | A workstation has no working network configuration | Layered troubleshooting, `ipconfig`, DHCP release/renew, gateway and DNS checks |
| [Linux User Management & File Permissions](linux-user-management-and-file-permissions/) | A new employee needs department access, and others must not have it | Users, groups, ownership, permissions, least privilege |
| [Linux Onboarding & Endpoint Hardening](onboarding-endpoint-security-hardening/) | Prepare a Linux endpoint for a new hire with baseline security | SSH keys, UFW, Fail2ban, systemd; includes real fixes for an SSH socket issue and a Fail2ban conflict |

---

## How the work is documented

* **Ticket style:** each write-up gives the symptom, the checks, the root cause, the fix and the proof, the way a Help Desk ticket is written.
* **Evidence first:** the documentation is matched to the screenshots. Where something was not captured or not solved, it says so instead of claiming it (see INC-AD-004).
* **Commands explained:** each lab lists the commands used and what each one is for, so the steps can be repeated.
* **Troubleshooting method:** identify the problem, gather information, isolate it, find the root cause, fix it, verify the fix, and document it.

---

## Skills at a glance

| Area | What the labs cover |
| --- | --- |
| Windows & Active Directory | Domain setup, OUs, users and groups, Group Policy, NTFS and share permissions, account lockout, PowerShell, Remote Desktop |
| Linux | Users and groups, file permissions, SSH, UFW, Fail2ban, systemd |
| Networking | IPv4, DHCP, DNS, gateways, connectivity testing |
| Cloud | Windows and Linux lab environments hosted on AWS EC2 |
| Support practice | Ticket-based troubleshooting, root-cause analysis, verification, clear documentation |

---

## Status

All eight labs are complete as documented. The Active Directory lab is ongoing and grows through separate, documented stages and tickets; department drive mapping is recorded as it stands: investigated, not yet fixed.

The direction is **IT Support / Help Desk → Cloud Support → DevOps**, built on practical, documented work.

---

## About

I'm building practical IT skills by diagnosing problems, fixing them, and writing up the work clearly, rather than relying on certificates alone.

**GitHub:** [@drexzw](https://github.com/drexzw)

> **Don't just document what worked. Document how you figured out why it worked.**
