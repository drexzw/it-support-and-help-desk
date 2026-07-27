# Linux User Management & File Permissions Lab

## Overview

This lab simulates a common IT Support and Linux system administration task: onboarding a new employee, managing user accounts, assigning group memberships, and configuring secure file permissions. The objective is to demonstrate fundamental Linux administration skills used in enterprise environments to manage user access and protect shared resources.

---

## Objectives

* Create and manage Linux user accounts
* Create and manage user groups
* Assign users to department groups
* Configure user passwords
* Create shared departmental directories
* Manage file and directory ownership
* Configure Linux file permissions using the principle of least privilege
* Verify user access through permission testing
* Remove temporary user accounts

---

## Environment

* **Operating System:** Ubuntu Linux
* **Shell:** Bash
* **Tools Used:**

  * `useradd`
  * `userdel`
  * `groupadd`
  * `usermod`
  * `passwd`
  * `mkdir`
  * `chmod`
  * `chgrp`
  * `ls`
  * `id`
  * `groups`
  * `su`

---

## Scenario

A new employee named **Sarah** joins the Marketing department. As the IT Support technician, the task is to create her account, assign her to the appropriate department group, configure secure access to a shared marketing directory, and verify that permissions allow only authorized users to access departmental resources.

To validate the configuration, a temporary contractor account is created to confirm that unauthorized users are denied access before being removed from the system.

---

## Tasks Completed

* Created a new Linux group for the Marketing department
* Created a new user account with a home directory and Bash shell
* Assigned the user to the Marketing group
* Configured a secure password for the new account
* Created a shared Marketing directory
* Assigned group ownership to the Marketing group
* Configured directory permissions using `chmod`
* Created a shared file for departmental collaboration
* Configured file permissions for group access
* Verified authorized user access by logging in as the new employee
* Verified unauthorized access restrictions using a contractor account
* Removed the temporary contractor account after testing

---

## Skills Demonstrated

* Linux User Administration
* Group Management
* File and Directory Permissions
* Access Control
* Linux Command Line
* Identity and Access Management (IAM) Concepts
* Principle of Least Privilege
* System Administration Fundamentals

---

## Key Linux Commands

```bash
useradd
userdel
groupadd
usermod
passwd
chmod
chgrp
mkdir
ls
id
groups
su
```

---

## Outcome

Successfully configured a Linux environment that allows authorized users to collaborate within a shared department while preventing unauthorized users from accessing protected resources. This lab demonstrates practical Linux administration skills commonly performed by IT Support Specialists, Help Desk Technicians, and Junior Linux Administrators.
