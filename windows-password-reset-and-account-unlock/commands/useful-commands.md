# Commands Used

This document contains the Windows commands used during the Password Reset & Account Unlock project.

---

## Create a Local User

```cmd
net user jsmith Password123! /add
```

Creates a new local user account named **jsmith** with the specified password.

---

## Display All Local Users

```cmd
net user
```

Lists every local user account on the computer.

---

## View User Details

```cmd
net user jsmith
```

Displays information about the user account, including:

* Account status
* Password last set
* Local group memberships
* Login permissions

---

## Reset a User Password

```cmd
net user jsmith Welcome2026!
```

Changes the user's password to a temporary password.

---

## Disable an Account

```cmd
net user jsmith /active:no
```

Disables the user account, preventing sign-in.

---

## Enable an Account

```cmd
net user jsmith /active:yes
```

Re-enables the user account.

---

## Delete a Test User

```cmd
net user jsmith /delete
```

Deletes the local user account from the computer.

---

## Key Takeaways

The `net user` command is one of the most commonly used Windows command-line tools for managing local user accounts. Understanding these commands is a valuable skill for entry-level Help Desk technicians, particularly when troubleshooting login and account-related issues.
