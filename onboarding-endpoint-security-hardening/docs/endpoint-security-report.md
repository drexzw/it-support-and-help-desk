# Endpoint Security Report

**Ticket:** #HD-1147
**System:** BarberPro Studios LLC — front-desk endpoint (Ubuntu 26.04 LTS)
**New account provisioned:** `jwill` (Jason Will, Front Desk)
**Prepared by:** Victor Z., Junior IT Support Technician
**Status:** Closed — all controls verified

## Summary

A new front-desk employee required a Linux endpoint account. This report covers the account provisioning, the security controls applied to the endpoint as part of that onboarding, and the verification performed before the ticket was closed. Two configuration issues were hit during the build (SSH port not taking effect, Fail2ban failing to start) — both are root-caused and resolved; full detail in [`troubleshooting.md`](troubleshooting.md).

## Controls Implemented

| Control | Configuration | Verification Method | Status |
|---|---|---|---|
| User provisioning | `jwill` created, added to `front-desk-staff` group | `id`, `groups` | ✅ Verified |
| Least privilege | No sudo rights for `jwill` | `sudo -U jwill -l` → denied | ✅ Verified |
| Password aging | Max 90 days, min 7 days, 14-day warning (per-user + system default) | `chage -l`, `/etc/login.defs` review | ✅ Verified |
| Password strength | Weak/reused/username-containing passwords rejected | Live test against `jwill` account | ✅ Verified |
| SSH hardening | Non-default port (2222), key-based auth only, root login disabled, `AllowUsers` allowlist | `sshd -T`, live connection tests (positive + negative) | ✅ Verified |
| Per-user auth limits | `MaxAuthTries 3` for `jwill` | `sshd_config` review, live lockout test | ✅ Verified |
| Host firewall | UFW default-deny incoming, only 2222/tcp allowed | `ufw status verbose` | ✅ Verified |
| Brute-force protection | Fail2ban `sshd` jail — 3 attempts / 10 min → 1 hr ban | `fail2ban-client status sshd`, live lockout test | ✅ Verified |

## Issues Encountered

Two issues came up during implementation; both were diagnosed to root cause rather than worked around:

1. **SSH port change had no effect** — `ssh.socket` was overriding the port set in `sshd_config` via socket activation. Resolved with a `systemctl edit ssh.socket` override.
2. **Fail2ban would not start** — a duplicate `[sshd]` section across `jail.conf` and `jail.local` caused the config parser to reject the whole file. Resolved by consolidating all custom jail rules into a single `jail.local` and clearing `jail.d/`.

Full diagnostic trail, including the approaches that didn't work, is in [`troubleshooting.md`](troubleshooting.md).

## Final Verification

All controls above were validated twice: once against the configuration files directly, and once end-to-end against the live `jwill` account — including a forced first-login password change, a rejected password reuse attempt, a rejected username-in-password attempt, and an account lockout triggered by repeated failed logins. Screenshots for every step are in `screenshots/` and referenced inline in [`README.md`](../README.md).

## Residual Risk / Recommendations

Scoped to this ticket, the endpoint met the required baseline. Outside that scope, worth flagging for a follow-up ticket rather than blocking this one:

- No centralized log shipping is configured — Fail2ban and SSH logs currently live only on the endpoint itself.
- No SSH key rotation or expiry policy exists yet for provisioned accounts.
- This lab covers one endpoint; a fleet of front-desk machines would benefit from configuration management (e.g. Ansible) rather than manual per-box hardening.

## Sign-off

Endpoint meets BarberPro's onboarding security baseline. Ticket closed.
