# Shared Folder Access Denied - Windows Help Desk Lab

## Overview

This project simulates a real-world Help Desk ticket where a user is unable to access a shared folder due to incorrect Windows file permissions.

The purpose of this lab was to practice troubleshooting user access issues, investigating permissions, identifying the root cause, applying a solution, and documenting the resolution.

---

# Scenario

A user reports that they cannot access the Finance shared folder on their workstation.

When attempting to open the folder, Windows displays an "Access Denied" message.

Other users are able to access the folder successfully, suggesting that the issue is related to user permissions.

As the Help Desk technician, the objective was to:

- Verify the affected user
- Investigate folder permissions
- Identify the cause of the access issue
- Restore the correct permissions
- Confirm the user can access the folder

---

# Environment

| Component | Details |
|---|---|
| Operating System | Windows 11 Home |
| Tools Used | PowerShell, File Explorer |
| User Accounts | employee1, employee2 |
| Issue Category | File Access / Permissions |

---

# Skills Demonstrated

- Windows local user management
- NTFS permissions troubleshooting
- File system troubleshooting
- PowerShell usage
- Access control concepts
- Root cause analysis
- Help Desk documentation

---

# Lab Workflow

The troubleshooting process followed a standard Help Desk workflow:

1. Create a simulated user environment
2. Configure folder permissions
3. Reproduce the access issue
4. Investigate the cause
5. Apply the required permission changes
6. Verify access was restored
7. Document the resolution

---

# Root Cause

The affected user account did not have the required NTFS permissions to access the Finance folder.

---

# Resolution

The correct Read and Execute permissions were assigned to the user account, allowing access to the shared folder.

---

# Documentation

Additional documentation for this project:

- Screenshots:
  
  [View Screenshots](./screenshots)

- Commands Used:

  [View Commands.md](./Commands.md)

- Support Ticket Documentation:

  https://github.com/drexzw/it-support-and-help-desk/blob/main/shared-folder-access-denied/support-ticket.md

---

# Lessons Learned

This lab improved understanding of:

- How Windows permissions affect user access
- How to troubleshoot access denied errors
- How Help Desk technicians investigate user problems
- How to document technical resolutions clearly
