# Commands Used

Every command below appears in a screenshot in this lab. Passwords are replaced with placeholders. Run all commands from **Command Prompt as Administrator**.

---

## List Local User Accounts

```cmd
net user
```

Lists every local user account on the computer.
**Screenshots:** [01](screenshots/01-existing-users.png), [02](screenshots/02-user-created.png), [08](screenshots/08-user-deleted.png)

---

## Create a Local User

```cmd
net user jsmith <InitialPassword> /add
```

Creates the local account `jsmith`.
**Screenshot:** [02](screenshots/02-user-created.png)

---

## View Account Details

```cmd
net user jsmith
```

Shows account status (Account active), Password last set, Password expires, Last logon, and group memberships.
**Screenshots:** [03](screenshots/03-user-information.png), [05](screenshots/05-password-last-set.png), [06](screenshots/06-account-disabled.png), [07](screenshots/07-account-enabled.png)

---

## Reset a Password

```cmd
net user jsmith <NewTemporaryPassword>
```

Sets a new password for the account. Confirm it worked by checking that **Password last set** changed.
**Screenshot:** [04](screenshots/04-password-reset.png)

---

## Disable an Account

```cmd
net user jsmith /active:no
```

Disables the account so it cannot sign in. `net user jsmith` then shows Account active = No.
**Screenshot:** [06](screenshots/06-account-disabled.png)

---

## Enable an Account

```cmd
net user jsmith /active:yes
```

Re-enables the account. `net user jsmith` then shows Account active = Yes.
**Screenshot:** [07](screenshots/07-account-enabled.png)

---

## Delete the Test Account

```cmd
net user jsmith /delete
```

Removes the local account. Used for cleanup after the lab.
**Screenshot:** [08](screenshots/08-user-deleted.png)

---

## Note on Typing Passwords

In this lab the passwords were typed directly into the command, so they appear on screen and in the console history. This was acceptable for a throwaway test account. For real accounts, running `net user <username> *` prompts for the password without displaying it. That method was not used or pictured in this lab.
