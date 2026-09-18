# Troubleshooting Log

Two real issues came up during this build. Both are documented here the way a help desk ticket note would capture them: what was observed, what was checked, the actual root cause, the fix, and how it was verified. A third, minor issue is included for completeness since it shows up in the screenshots.

---

## Issue 1: SSH Port Change Not Taking Effect

**Screenshot:** `screenshots/04-ssh-port-change-socket-override-fix.png`

**Symptom**
After editing `/etc/ssh/sshd_config` to set `Port 2222` and restarting the service, every connection attempt on port 2222 failed with `Connection refused` — tried against `127.0.0.1`, the internal private IP, and the box's public interface IP in turn.

**What I checked**
- Confirmed the staged config with `sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|port"` — it correctly reported the new port.
- Attempted the connection against three different addresses (`ssh -p 2222 jwill@127.0.0.1`, the `10.255.x.x` interface, and the `172.29.x.x` interface) via `ip a` to rule out a wrong-interface issue. All three refused.
- Also tried port 22 directly on one of the interfaces to sanity-check whether the daemon was even listening anywhere — still refused.

**Root cause**
`ssh.socket` was overriding the port defined in `sshd_config`. On systems where SSH is socket-activated, the listening port is controlled by the socket unit, independent of what's in the daemon's own config file — so editing `sshd_config` alone doesn't move the port.

**Fix**
```bash
sudo systemctl edit ssh.socket
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```

**Verification**
`ssh -p 2222 jwill@<server-ip>` connected and prompted for host-key verification as expected — confirmed again later in `screenshots/10-ssh-service-status.png` (`Server listening on 0.0.0.0 port 2222`) and `screenshots/11-ssh-port-listening-check.png` (`ss -tulpn` showing `sshd` bound to `0.0.0.0:2222`).

---

## Issue 2: Fail2ban Failing to Start — Duplicate `[sshd]` Section

**Screenshots:** `screenshots/13-fail2ban-start-failure.png`, `screenshots/14-fail2ban-troubleshooting.png`, `screenshots/15-fail2ban-fix-verification.png`

**Symptom**
`sudo systemctl start fail2ban` came back `failed (Result: exit-code)`. `systemctl status fail2ban` showed the process exiting immediately (`status=255/EXCEPTION`).

**Diagnosis**
`sudo fail2ban-client -t` gave the actual reason:
```
ERROR - Failed during configuration: While reading from '/etc/fail2ban/jail.conf' [line 282]: section 'sshd' already exists
```

**Dead ends tried first**
- Reinstalling the package (`sudo apt-get install --reinstall fail2ban`) — didn't resolve it, since the local override files that caused the conflict weren't touched by the reinstall.
- A second reinstall with `-o Dpkg::Options::="--force-confask"` to force a prompt on the packaged config file — this correctly offered to reinstall the maintainer's version of `jail.conf`, but the conflict was actually coming from the custom `jail.local`/`jail.d` files sitting alongside it, so the error persisted.

**Root cause**
Fail2ban merges `jail.conf`, every file in `jail.d/`, and `jail.local` at startup. A `[sshd]` section had ended up defined in more than one of those files (left over from earlier edits), and the parser fails outright on a duplicate section rather than merging it silently.

**Fix**
```bash
sudo rm -f /etc/fail2ban/jail.local
sudo rm -rf /etc/fail2ban/jail.d/*
sudo nano /etc/fail2ban/jail.local     # re-added only the sshd jail settings
sudo fail2ban-client -t                # OK: configuration test is successful
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

**Verification**
`fail2ban-client status sshd` returned a clean jail status — filter active, 0 currently failed, 0 currently banned. Re-checked after the fix (`screenshots/16-final-firewall-port-check.png`) to confirm nothing else had regressed.

**Lesson learned**
Keep all custom jail definitions in a single `jail.local` and leave `jail.d/` empty. Splitting jail definitions across multiple files is what let the same section get defined twice in the first place.

---

## Issue 3 (minor): Typo in the SSH Config Backup Filename

**Screenshot:** `screenshots/07-ssh-key-auth-setup.png`

**What happened**
While backing up the working SSH config before editing it, the destination filename was typed as `/etc/ssh/ssshd_config.bak` instead of `sshd_config.bak`.

**Impact**
None functionally — the backup file was still created and never referenced again by name, and the live `sshd_config` was untouched. Worth documenting anyway: it's a good reminder to double-check filenames on backup commands specifically, since a typo there is silent (the command still succeeds) and would only bite if the backup were ever actually needed for a restore.
