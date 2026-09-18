# Command Reference — Onboarding & Endpoint Hardening

Every command actually run during this lab, grouped by phase and in the order performed. This is a copy/paste reference; for the narrative (what each step was for, and what broke) see [`README.md`](README.md) and [`docs/troubleshooting.md`](docs/troubleshooting.md).

`<server-ip>` stands in for the endpoint's address — a placeholder, not a literal value to paste.

## 1. Create User & Assign Groups
```bash
sudo adduser jwill
sudo usermod -aG front-desk-staff jwill
id jwill
groups jwill
```

## 2. Confirm Least Privilege
```bash
sudo -U jwill -l
```

## 3. Password Aging Policy
```bash
sudo chage -m 7 -M 90 -W 14 -I 7 jwill
sudo nano /etc/login.defs
```
Values set in `/etc/login.defs`:
```
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```

## 4. SSH — Move Off the Default Port
```bash
sudo nano /etc/ssh/sshd_config          # Port 2222
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication|port"
ssh -p 2222 jwill@<server-ip>
ip a
```
Fix for the port not taking effect (`ssh.socket` overriding `sshd_config`):
```bash
sudo systemctl edit ssh.socket
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
```

## 5. Install & Configure the Firewall (UFW)
```bash
sudo apt update && sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw enable
sudo ufw status verbose
```

## 6. SSH Key-Based Authentication
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys
sudo nano /etc/ssh/sshd_config
sudo sshd -t
sudo systemctl restart ssh
```

## 7. Restrict SSH Access by User
Added to `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
AllowUsers victor_zw jwill

Match User jwill
    MaxAuthTries 3
```

## 8. Fail2ban — Initial Jail Configuration
```bash
sudo nano /etc/fail2ban/jail.local
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
sudo fail2ban-client status sshd
```
Jail definition:
```
[sshd]
enabled  = true
port     = ssh
maxretry = 3
findtime = 10m
bantime  = 1h
```

## 9. Verify SSH Is Listening Correctly
```bash
sudo systemctl status ssh
sudo ss -tulpn | grep ssh
```

## 10. End-to-End SSH Access Test
```bash
ssh jwill@<server-ip>              # expected: refused (port 22 closed)
ssh victor_zw@<server-ip>          # expected: refused (port 22 closed)
ssh -p 2222 victor_zw@<server-ip>  # expected: succeeds (SSH key)
ssh -p 2222 jwill@<server-ip>      # expected: denied (publickey) — jwill has no key loaded yet
```

## 11. Fail2ban — Diagnosing the Startup Failure
```bash
sudo fail2ban-client -t
sudo apt-get install --reinstall fail2ban
sudo apt-get install --reinstall -o Dpkg::Options::="--force-confask" fail2ban
```

## 12. Fail2ban — Resolving the Duplicate Jail Conflict
```bash
sudo rm -f /etc/fail2ban/jail.local
sudo rm -rf /etc/fail2ban/jail.d/*
sudo nano /etc/fail2ban/jail.local     # re-add the sshd jail settings only
sudo fail2ban-client -t                # OK: configuration test is successful
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd       # jail active, 0 currently banned
```

## 13. Final Firewall & Port Verification
```bash
sudo ss -tulpn
sudo ufw status verbose
```

## 14. Final Endpoint Readiness — jwill Account
```bash
sudo mkdir -p /home/jwill/.ssh
sudo cp ~/.ssh/authorized_keys /home/jwill/.ssh/authorized_keys
sudo chown -R jwill:jwill /home/jwill/.ssh
sudo chmod 700 /home/jwill/.ssh
sudo chmod 600 /home/jwill/.ssh/authorized_keys
```
Final validation pass performed against the live `jwill` account (forced password change, password-reuse rejection, username-in-password rejection, `MaxAuthTries`/Fail2ban lockout) — see [`README.md`](README.md#14-final-endpoint-readiness-check--jwill-account) for the pass/fail detail on each.
