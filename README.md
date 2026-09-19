# IT Support & Help Desk Portfolio

A hands-on IT Support and Help Desk portfolio built around realistic small-business support scenarios.

This repository documents my practical experience troubleshooting Windows and Linux systems, supporting users, resolving network and access issues, managing permissions, configuring endpoints, and working with cloud-hosted environments.

The labs are designed around a simulated business environment, **BarberPro Studios LLC**, and are documented using a ticket-based troubleshooting approach similar to what an entry-level IT Support or Help Desk technician may encounter in a real workplace.

---

## 🎯 Portfolio Objective

The goal of this repository is to demonstrate practical IT Support skills through hands-on troubleshooting rather than purely theoretical coursework.

Each lab focuses on a specific support scenario and documents:

* The reported issue or business requirement
* The technical environment
* Troubleshooting steps
* Commands and tools used
* Root-cause identification
* Resolution
* Verification
* Supporting screenshots and evidence
* Lessons learned

The overall objective is to develop the skills required for an **IT Support / Help Desk → Cloud Support → DevOps** career path.

---

## 🖥️ Technical Environment

The labs use a combination of local and cloud-based environments depending on the scenario.

### Operating Systems

* Windows 11
* Windows Server 2022
* Linux

### Cloud

* Amazon Web Services (AWS)
* Amazon EC2
* Amazon EBS
* Amazon CloudWatch
* AWS networking and security services

### Networking

* IPv4
* DHCP
* DNS
* Default gateways
* IP troubleshooting
* Connectivity testing
* `ping`
* `ipconfig`
* `nslookup`

### Windows Administration

* Local user administration
* Active Directory
* NTFS permissions
* Shared folders
* Password resets
* Account management
* Remote Desktop
* PowerShell

### Linux Administration

* User and group management
* File ownership
* File permissions
* SSH
* UFW
* Fail2ban
* systemd
* Linux command line

### Documentation & Support

* Support tickets
* Troubleshooting documentation
* Commands reference
* Deployment notes
* Screenshots and technical evidence
* Root-cause analysis
* Verification testing

---

# 📂 IT Support Labs

## 01 — Linux User Management & File Permissions

**Scenario:**
A new employee needs access to department resources while unauthorized users must be prevented from accessing them.

### Skills Demonstrated

* Linux user creation
* Group management
* File ownership
* Linux permissions
* Department resource access
* Least-privilege principles
* Access verification

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/linux-user-management-and-file-permissions)

---

## 02 — New Employee Onboarding & Endpoint Security Hardening

**Scenario:**
Prepare a Linux endpoint for a new employee while applying baseline security controls.

### Skills Demonstrated

* User onboarding
* Group configuration
* Password policies
* SSH configuration
* SSH key authentication
* UFW firewall
* Fail2ban
* systemd troubleshooting
* Security verification
* Endpoint hardening

### Troubleshooting Highlights

This lab includes real troubleshooting scenarios encountered during implementation, including SSH socket configuration and Fail2ban configuration issues.

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/onboarding-endpoint-security-hardening)

---

## 03 — DHCP / IP Configuration & Troubleshooting

**Scenario:**
A workstation is experiencing network connectivity problems caused by incorrect or missing IP configuration.

### Skills Demonstrated

* IPv4 troubleshooting
* DHCP
* `ipconfig`
* DHCP release/renew
* Default gateway verification
* DNS configuration
* Connectivity testing
* Layered troubleshooting

### Troubleshooting Method

The investigation follows a structured approach:

1. Inspect current configuration
2. Test local connectivity
3. Test the default gateway
4. Test external connectivity
5. Test DNS resolution
6. Renew DHCP configuration
7. Verify restored configuration
8. Perform final connectivity testing

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/dhcp-ip-configuration-and-troubleshooting-lab)

---

## 04 — DNS Troubleshooting

**Scenario:**
A Windows Server can communicate over the network but cannot resolve external hostnames.

### Skills Demonstrated

* DNS troubleshooting
* `nslookup`
* `ping`
* IP vs hostname troubleshooting
* DNS configuration
* Layered network troubleshooting
* Root-cause analysis

### Key Concept Demonstrated

The lab demonstrates how to distinguish between:

> **Network connectivity problems** and **DNS resolution problems**

The environment intentionally introduces an incorrect DNS configuration and then uses testing to identify and resolve the failure.

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/dns-troubleshooting-lab)

---

## 05 — NTFS Permissions

**Scenario:**
Users require different levels of access to files and folders based on their responsibilities.

### Skills Demonstrated

* Windows user management
* NTFS permissions
* Permission inheritance
* Access control
* Permission verification
* Windows Server
* Remote Desktop
* Cloud-hosted Windows environments

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/nfts-permissions-lab)

---

## 06 — Shared Folder Access Denied

**Scenario:**
A user reports that they cannot access a shared business resource that they are expected to use.

### Skills Demonstrated

* Access troubleshooting
* Windows permissions
* User/group investigation
* Shared folder troubleshooting
* Access verification
* Help Desk ticket resolution

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/shared-folder-access-denied)

---

## 07 — Windows Password Reset & Account Unlock

**Scenario:**
A user is unable to sign in and requires assistance with their account.

### Skills Demonstrated

* Windows account administration
* Password resets
* Account status investigation
* Command-line administration
* User verification
* Ticket documentation
* Resolution verification

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/windows-password-reset-and-account-unlock)

> **Future expansion:** Active Directory-based password reset and account lockout troubleshooting is covered in the Active Directory lab below.

---

## 08 — Active Directory & Help Desk Administration

**Scenario:**
Simulate a small-business Windows domain environment (**BarberPro Studios LLC**) and practice the Active Directory administration and troubleshooting tasks a Help Desk technician would encounter day to day.

### Environment

* Domain: `corp.drexzw.local` (NetBIOS: `DREXZW`)
* Windows Server 2022 Domain Controller + domain-joined Windows client
* Hosted on AWS EC2

### Skills Demonstrated

* Active Directory Domain Services deployment
* Organizational Unit design across multiple departments (IT, HR, Finance, Sales, Management, Service Accounts, Workstations)
* Security groups and user administration
* Client domain join, verified from both the client and Domain Controller
* Group Policy creation, linking, and refresh verification
* Password and account lockout policy configuration
* Domain authentication testing
* Active Directory PowerShell (`Get-ADUser`, `Get-ADComputer`, `Get-ADOrganizationalUnit`, `Unlock-ADAccount`, and related cmdlets)

### Troubleshooting Highlights

Includes a full Help Desk-style account-lockout ticket: an account lockout was intentionally reproduced, investigated through Group Policy and Active Directory Users and Computers, resolved, and re-verified — plus five additional documented troubleshooting scenarios covering DNS, domain membership, computer-object visibility, authentication, and Group Policy application.

[View Lab →](https://github.com/drexzw/it-support-and-help-desk/tree/main/active-directory-lab)

> **Planned additions:** NTFS and file-share permissions, department-specific access control, additional Group Policy configurations, and further Help Desk tickets will extend this environment.

---

# 🛠️ Troubleshooting Philosophy

The labs follow a structured troubleshooting methodology rather than relying on trial-and-error fixes.

### 1. Identify

Understand the user's reported problem and reproduce the issue where possible.

### 2. Gather Information

Collect configuration details, error messages, logs, command output, and other relevant evidence.

### 3. Isolate

Determine which layer or component is responsible for the problem.

### 4. Identify Root Cause

Use the available evidence to determine why the problem occurred.

### 5. Resolve

Apply the appropriate fix while minimizing unnecessary changes.

### 6. Verify

Confirm that the original problem has been resolved.

### 7. Document

Record the symptoms, investigation, root cause, resolution, and verification results.

This approach is intended to reflect how troubleshooting should be performed in a real IT Support environment.

---

# 📋 Documentation Structure

Where applicable, individual labs use a consistent documentation structure:

```text
lab/
├── README.md
├── commands.md
├── support-ticket.md
├── deployment-notes.md
└── screenshots/
```

### README.md

Explains the scenario, environment, objectives, implementation, troubleshooting, and results.

### commands.md

Contains the important commands used during the lab and explains their purpose.

### support-ticket.md

Documents the scenario from a Help Desk perspective, including:

* User issue
* Symptoms
* Investigation
* Root cause
* Resolution
* Verification
* Ticket closure

### deployment-notes.md

Documents infrastructure and environment configuration where applicable.

### screenshots/

Contains chronological technical evidence supporting the troubleshooting process.

---

# 🎓 Skills Demonstrated

Through these projects, I am developing practical experience in:

### IT Support

* Ticket-based troubleshooting
* User support
* Account administration
* Password resets
* Access troubleshooting
* Endpoint troubleshooting
* Documentation
* Root-cause analysis

### Windows

* Windows 11
* Windows Server
* PowerShell
* NTFS permissions
* Shared folders
* User administration
* Remote Desktop
* Active Directory

### Linux

* User and group management
* Permissions
* SSH
* Firewall configuration
* Fail2ban
* systemd
* Command-line administration

### Networking

* IPv4
* DHCP
* DNS
* Gateways
* Connectivity testing
* Network troubleshooting

### Cloud

* AWS EC2
* EBS
* CloudWatch
* Cloud-hosted Windows environments
* Cloud networking fundamentals

---

# ☁️ Career Progression

This portfolio is being developed as part of a broader technical progression:

```text
IT Support / Help Desk
        │
        ├── Windows
        ├── Linux
        ├── Networking
        ├── Active Directory
        └── Troubleshooting
                │
                ▼
          Cloud Support
                │
        ├── AWS
        ├── EC2
        ├── IAM
        ├── CloudWatch
        ├── Networking
        └── Cloud Troubleshooting
                │
                ▼
             DevOps
                │
        ├── Linux
        ├── Git
        ├── Python
        ├── Bash
        ├── Docker
        ├── CI/CD
        ├── Terraform
        └── AWS Automation
```

The immediate focus of this repository is **IT Support and Help Desk**, with cloud-based projects gradually being incorporated as the portfolio progresses.

---

# 📌 Current Status

### Completed

* [x] Linux User Management & File Permissions
* [x] New Employee Onboarding & Endpoint Security Hardening
* [x] DHCP / IP Configuration & Troubleshooting
* [x] DNS Troubleshooting
* [x] NTFS Permissions
* [x] Shared Folder Access Denied
* [x] Windows Password Reset & Account Unlock
* [x] Active Directory & Help Desk Administration

### In Progress / Planned

* [ ] Extend Active Directory lab with NTFS/file-share permissions and additional tickets
* [ ] Standardize documentation across existing labs
* [ ] Improve screenshot organization and naming
* [ ] Refine support-ticket documentation
* [ ] Connect related labs into a unified simulated business environment
* [ ] Expand AWS / Cloud Support projects

---

# 📈 Future Development

Once the core IT Support portfolio is complete, future projects will increasingly focus on:

* AWS Cloud Support
* Identity and Access Management
* Cloud networking
* Monitoring and troubleshooting
* Infrastructure automation
* Linux administration
* Git and GitHub
* Python
* Bash
* Docker
* CI/CD
* Infrastructure as Code
* DevOps

The objective is to build progressively from **IT Support → Cloud Support → DevOps** while maintaining practical, documented, hands-on experience.

---

## 👤 About

This repository represents my hands-on learning journey into IT Support, Cloud Computing, and eventually DevOps.

Rather than focusing solely on certifications or theoretical coursework, I use these labs to practice diagnosing problems, implementing solutions, documenting technical work, and thinking through issues from the perspective of an IT Support technician.

**GitHub:** [@drexzw](https://github.com/drexzw)

---

## ⭐ Repository Philosophy

> **Don't just document what worked. Document how you figured out why it worked.**

The goal of this portfolio is not to demonstrate that every lab was completed without problems.

The goal is to demonstrate the ability to **investigate problems, understand their root causes, implement appropriate solutions, verify the results, and communicate the work clearly.**
