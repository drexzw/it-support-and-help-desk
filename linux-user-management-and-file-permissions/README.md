# Linux User Management & File Permissions

## Overview

This lab simulates a user and file-permission management task performed by the IT team at **Ubuntu Tech Cloud Solutions**.

The objective was to configure Linux users and groups, establish appropriate file and directory ownership, apply permissions based on job responsibilities, and verify that users could access only the resources they were authorized to use.

The lab demonstrates fundamental Linux administration and access-control practices commonly used in IT support, system administration, and cloud environments.

---

## Business Scenario

Ubuntu Tech Cloud Solutions is onboarding employees who require access to company resources based on their departments.

The IT team is responsible for:

* Creating and managing employee accounts
* Assigning users to appropriate security groups
* Creating department-specific resources
* Configuring ownership and permissions
* Testing authorized and unauthorized access
* Troubleshooting permission-related issues
* Applying the principle of least privilege

The goal is to ensure employees have the access required to perform their jobs without unnecessarily exposing company resources.

---

## Environment

| Component        | Details                          |
| ---------------- | -------------------------------- |
| Operating System | Ubuntu Linux                     |
| Environment      | Linux virtual/lab environment    |
| Shell            | Bash                             |
| User Management  | `useradd` / `usermod` / `passwd` |
| Group Management | Linux groups                     |
| Permissions      | `chmod`                          |
| Ownership        | `chown`                          |
| Verification     | `id`, `groups`, `ls -l`          |

---

## Users and Groups

The lab environment contains the following employees and department groups:

| User            | Department Group |
| --------------- | ---------------- |
| `john.smith`    | `developers`     |
| `sarah.johnson` | `developers`     |
| `michael.brown` | `finance`        |

These group memberships are used to control access to department-specific resources.

---

## Objectives

The following objectives were completed as part of the lab:

1. Create and manage Linux user accounts.
2. Create department-based Linux groups.
3. Assign users to the appropriate groups.
4. Verify user and group membership.
5. Create directories and files for company resources.
6. Configure file and directory ownership.
7. Apply Linux read, write, and execute permissions.
8. Test access using different users.
9. Identify and troubleshoot permission-related issues.
10. Apply the principle of least privilege.

---

## Tasks Completed

### 1. User Management

Linux user accounts were created for employees at Ubuntu Tech Cloud Solutions.

User accounts were verified using commands such as:

```bash
id username
```

and:

```bash
groups username
```

---

### 2. Group Management

Department-based groups were created to simplify access management.

The following groups were configured:

* `developers`
* `finance`

Users were then assigned to their respective department groups.

---

### 3. File and Directory Management

Company resources were organized into directories and files that could be assigned to specific departments.

Ownership was configured using:

```bash
chown
```

This allowed resources to be associated with the appropriate users and groups.

---

### 4. Linux Permissions

Linux file permissions were configured using:

```bash
chmod
```

Permissions were applied using the standard Linux permission categories:

* **User/Owner**
* **Group**
* **Others**

The following permission types were used:

* `r` — Read
* `w` — Write
* `x` — Execute

Numeric permissions such as `755`, `770`, and `660` were also used where appropriate.

---

### 5. Access Testing

Access was tested using different employee accounts to verify that permissions were working as intended.

Testing included:

* Authorized access
* Unauthorized access
* Reading files
* Modifying files
* Directory access
* Permission verification

The results were compared against the intended access requirements.

---

## Skills Demonstrated

This lab demonstrates practical experience with:

* Linux user administration
* Linux group administration
* File and directory management
* File ownership
* Linux permissions
* `chmod`
* `chown`
* User/group verification
* Access-control troubleshooting
* Least-privilege access
* Bash command-line administration
* Basic IT support troubleshooting

---

## Key Linux Concepts

### Permission Structure

Linux permissions are represented in three categories:

```text
Owner | Group | Others
```

For example:

```text
rwxr-xr-x
```

can be interpreted as:

```text
rwx | r-x | r-x
```

Where:

* Owner: read, write, execute
* Group: read, execute
* Others: read, execute

---

## Documentation

Additional documentation for this lab is available below:

* [Commands](./commands.md)
* [Troubleshooting](./troubleshooting.md)
* [Screenshots](./screenshots/)

---

## Outcome

The lab successfully demonstrated how Linux user accounts, groups, ownership, and permissions can be combined to control access to company resources.

The exercise also provided practical troubleshooting experience by requiring access permissions to be verified and corrected when users did not have the expected level of access.

This represents a foundational Linux administration skill set applicable to IT support, system administration, and future cloud/DevOps environments.

