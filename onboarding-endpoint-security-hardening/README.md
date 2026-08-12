# New Employee Onboarding & Endpoint Security Hardening

## Project Overview

## Business Scenario

BarberPro Studios LLC is a growing business that requires a structured IT onboarding process for new employees.

As a Junior IT Support Technician, my responsibility was to prepare a new employee environment by:

- Creating user accounts
- Configuring access permissions
- Securing the endpoint environment
- Verifying system readiness
- Documenting the onboarding process

This project simulates a real-world Help Desk ticket involving employee provisioning and endpoint security. The new hire, **Jason Will** (`jwill`), was being onboarded into the **front-desk-staff** group on a Linux endpoint.

---

## Objectives

The goals of this project were:

- Create and manage Linux user accounts
- Assign users to appropriate groups
- Enforce a password aging policy
- Harden SSH access (key-based auth, non-default port, restricted users)
- Configure a host firewall (UFW)
- Deploy brute-force protection (Fail2ban)
- Verify user access and system configuration end-to-end
- Document the onboarding process using IT support procedures — including the troubleshooting encountered along the way

---

## Environment

**Operating System:** Ubuntu 26.04 LTS (running under WSL2 on Windows)
**System Role:** Linux workstation/server environment used to simulate an employee endpoint
**Users involved:**
- `victor_zw` — admin/sudo account performing the onboarding
- `jwill` (Jason Will) — new employee account, front-desk-staff

---

## Tools Used

| Category | Tools |
|---|---|
| User & group management | `adduser`, `usermod`, `id`, `groups` |
| Password policy | `chage`, `/etc/login.defs` |
| SSH hardening | `/etc/ssh/sshd_config`, `sshd -t`, `systemctl edit ssh.socket` |
| Firewall | `ufw` |
| Brute-force protection | `fail2ban`, `/etc/fail2ban/jail.local` |
| Verification | `ss -tulpn`, `systemctl status`, `fail2ban-client status` |

---

## Implementation Walkthrough

### 1. Create the User & Assign Groups
`screenshots/01-create-and-add-user.png`

Created the `jwill` account with `sudo adduser jwill`, set full name (Jason Will), and added the account to the `front-desk-staff` group with `usermod -aG front-desk-staff jwill`. Verified group membership with `id jwill` and `groups jwill`.

### 2. Confirm Least Privilege
`screenshots/02-change-password-policy.png`

Confirmed `jwill` does **not** have sudo rights (`sudo -U jwill -l` → "not allowed to run sudo") before moving into password policy configuration.

### 3. Set Password Aging Policy
`screenshots/02-change-password-policy.png`, `screenshots/03-password-aging-policy.png`

Applied a per-user aging policy with:
```
sudo chage -m 7 -M 90 -W 14 -I 7 jwill
```
Then set the same values as the system-wide default in `/etc/login.defs`:
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```
This enforces a 90-day max password age, a 7-day minimum between changes (to prevent password cycling), and a 14-day expiry warning.

### 4. Harden SSH — Move Off the Default Port
`screenshots/04-ssh-port-2222-troubleshoot_socket-override-fix.png`

Edited `/etc/ssh/sshd_config` and set the listening port to `2222`. Confirmed the change was staged with:
```
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|port"
```
**Ran into a real issue here:** connecting on port 2222 kept failing with "Connection refused" even after the config edit and a service restart — across `127.0.0.1`, the private IP, and the public interface IP. Root cause: on this Ubuntu setup, **`ssh.socket` was overriding the port defined in `sshd_config`**, since socket activation controls the listening port independently of the daemon config. Fixed it with a socket override:
```
sudo systemctl edit ssh.socket
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```
After that, `ssh -p 2222 jwill@<server-ip>` connected and prompted for host key verification as expected.

### 5. Install and Configure the Firewall
`screenshots/05-installing-ufw.png`, `screenshots/06-installing-firewall-and-checking-status.png`

UFW wasn't installed by default (`sudo: 'ufw': command not found`), so it was installed via `apt install ufw` along with its dependencies (`iptables`, `nftables`, etc.). Configured a default-deny posture:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw enable
```
Verified with `sudo ufw status verbose` — confirmed active, default deny incoming, and only 2222/tcp (v4 + v6) allowed in.

### 6. Set Up SSH Key-Based Authentication
`screenshots/07-ssh-hardening.png.png`

Backed up the working config first (`cp sshd_config sshd_config.bak`), then built out the `.ssh` directory structure for key-based login:
```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Validated the edited config with `sudo sshd -t` before restarting the service (`systemctl restart ssh`) — catches syntax errors before they lock you out of the session.

### 7. Restrict SSH Access by User
`screenshots/14-chnagng-fail2ban-policy.png`

Locked SSH down further in `sshd_config`:
```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
AllowUsers victor_zw jwill

Match User jwill
    MaxAuthTries 3
```
Root login is disabled, only key-based auth is allowed by default, and only two named accounts can connect at all. `jwill` specifically gets a tighter `MaxAuthTries 3` to feed into the Fail2ban jail below.

### 8. Configure Fail2ban
`screenshots/08-hardening-policy.png`

Set up `/etc/fail2ban/jail.local` to monitor SSH:
```
[sshd]
enabled  = true
port     = ssh
maxretry = 3
findtime = 10m
bantime  = 1h
```
Three failed attempts within a 10-minute window triggers a 1-hour IP ban.

### 9. Verify SSH Is Listening Correctly
`screenshots/09-checking-status.png.png`, `screenshots/10-looking-for-port.png.png`

Confirmed via `systemctl status ssh` that the service was active and the log line explicitly showed `Server listening on 0.0.0.0 port 2222`. Double-checked with `ss -tulpn | grep ssh`, confirming `sshd` bound to `0.0.0.0:2222`.

### 10. Test the SSH Hardening End-to-End
`screenshots/11-testing-ssh-changes.png.png`

Ran negative tests before positive ones:
- `ssh jwill@<ip>` (no port) → refused — port 22 is closed, as expected.
- `ssh victor_zw@<ip>` (no port) → refused — same reason.
- `ssh -p 2222 victor_zw@<ip>` → **succeeds**, authenticated via SSH key.
- `ssh -p 2222 jwill@<ip>` → **denied (publickey)** — correct, since `jwill` had no key loaded yet at this point.

This confirmed the hardening was actually enforcing what it was supposed to: unauthorized users and the default port are both blocked, and key-based auth for the authorized account works.

### 11. Troubleshoot Fail2ban Startup Failure
`screenshots/12-starting-fail2ban.png`, `screenshots/13-fail2ban-fails.png`

`systemctl start fail2ban` came up as `failed (Result: exit-code)`. Diagnosed with `fail2ban-client -t`, which returned:
```
ERROR - Failed during configuration: While reading from '/etc/fail2ban/jail.conf' [line 282]: section 'sshd' already exists
```
**Root cause:** a duplicate `[sshd]` section between the packaged `jail.conf` and the custom `jail.local`, left over from earlier edits/reinstalls. Attempted a package reinstall first (`apt-get install --reinstall fail2ban`), which didn't fully resolve it since the local override files persisted.

### 12. Resolve the Fail2ban Conflict
`screenshots/15-fixing-fail2ban.png`

Fixed it by removing the stale local override files and rebuilding the jail config cleanly:
```
sudo rm -f /etc/fail2ban/jail.local
sudo rm -rf /etc/fail2ban/jail.d/*
sudo nano /etc/fail2ban/jail.local   # re-added the sshd jail settings only
sudo fail2ban-client -t              # OK: configuration test is successful
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd     # jail active, 0 currently banned
```
**Lesson learned:** Fail2ban merges `jail.conf`, `jail.d/*.conf`, and `jail.local` — if the same section is defined in more than one of those, the parser fails outright rather than merging silently. Keeping all custom jail definitions in a single `jail.local` file (and clearing `jail.d`) avoids the conflict.

### 13. Final Firewall & Port Verification
`screenshots/16-final-endpoint-readiness-check.png`

Re-ran `ss -tulpn` and `ufw status verbose` after the Fail2ban fix to confirm nothing had regressed: `sshd` still listening only on `2222`, UFW still active with default-deny incoming and only `2222/tcp` allowed.

### 14. Final Endpoint Readiness Check — jwill Account
`screenshots/17-final-endpoint-readiness.png.png`

Completed the onboarding by provisioning `jwill`'s own SSH key access and proving the security controls hold under real conditions:
```
sudo mkdir -p /home/jwill/.ssh
sudo cp ~/.ssh/authorized_keys /home/jwill/.ssh/authorized_keys
sudo chown -R jwill:jwill /home/jwill/.ssh
sudo chmod 700 /home/jwill/.ssh
sudo chmod 600 /home/jwill/.ssh/authorized_keys
```
Then validated the full policy stack in one pass:
- Forced password change on first login (administrator-enforced) — accepted.
- Re-entering the same password → **rejected** ("password is the same as the old one").
- A password containing the username → **rejected** ("BAD PASSWORD: contains the username").
- Enough consecutive failed login attempts → account temporarily locked out under the `MaxAuthTries 3` / Fail2ban policy.

Every control configured earlier in the lab — password aging, weak-password rejection, SSH key requirement, per-user auth limits, and Fail2ban banning — was independently verified working against the actual new-hire account.

---

## Troubleshooting & Lessons Learned

Two real issues came up during this build, and both are worth calling out because they reflect actual on-the-job debugging rather than a clean, scripted walkthrough:

1. **SSH port change not taking effect** — `sshd_config` correctly specified port 2222, but connections kept refusing. The cause was `ssh.socket` overriding the daemon's listening port via socket activation. Fixed with a `systemctl edit ssh.socket` override, not a `sshd_config` change.
2. **Fail2ban failing to start** — a duplicate `[sshd]` jail section across `jail.conf` and `jail.local` caused the config parser to fail outright. Fixed by consolidating all custom jail rules into a single `jail.local` and clearing `jail.d`.

---

## Outcome

By the end of this lab, the endpoint met the following state:

- New employee account created, grouped, and permission-scoped correctly
- Password aging policy enforced system-wide and per-user
- SSH restricted to key-based auth, non-default port (2222), and a named allowlist of two users
- Root login disabled entirely
- UFW active with default-deny incoming, allowing only the SSH port
- Fail2ban actively monitoring SSH and banning after 3 failed attempts within 10 minutes
- All controls verified against the live `jwill` account, not just assumed from config files
