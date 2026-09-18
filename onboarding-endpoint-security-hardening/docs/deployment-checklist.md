# Onboarding & Endpoint Readiness Checklist

Ticket #HD-1147 — `jwill` (Jason Will, Front Desk). Used to confirm the endpoint was ready before closing the ticket.

## Account Provisioning
- [x] User account created (`jwill`)
- [x] Added to correct access group (`front-desk-staff`)
- [x] Confirmed no sudo / admin rights
- [x] Group membership verified (`id`, `groups`)

## Password Policy
- [x] Per-user password aging set (`chage`)
- [x] System-wide defaults set (`/etc/login.defs`)
- [x] Weak-password rejection confirmed (reused password, username-in-password)
- [x] Forced password change on first login confirmed

## SSH Access
- [x] Default port (22) closed
- [x] Non-default port (2222) confirmed listening
- [x] Root login disabled
- [x] Password authentication disabled (key-based only)
- [x] Connection allowlist restricted to named users (`victor_zw`, `jwill`)
- [x] Per-user `MaxAuthTries` set for `jwill`
- [x] `jwill`'s SSH key provisioned and access confirmed
- [x] Negative tests passed (unauthorized user + default port both refused)

## Firewall
- [x] UFW installed and enabled
- [x] Default-deny incoming
- [x] Only the SSH port (2222/tcp) allowed in
- [x] Status re-verified after Fail2ban fix (no regression)

## Brute-Force Protection
- [x] Fail2ban installed and running
- [x] `sshd` jail active (3 attempts / 10 min → 1 hr ban)
- [x] Lockout behavior confirmed against a real failed-login sequence

## Sign-off
- [x] All controls verified against config files
- [x] All controls re-verified against the live `jwill` account
- [x] Documentation complete (README, commands reference, troubleshooting log, security report)

**Endpoint ready. Ticket closed.**
