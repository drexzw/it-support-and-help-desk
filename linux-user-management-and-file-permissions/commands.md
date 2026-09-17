# Commands Reference

Commands are listed in the order they were run, grouped to match the `screenshots/` evidence folders. Commands not directly visible in a screenshot are marked **(not pictured)**.

---

## 01 — User Management

Account and group creation were performed **(not pictured)** — only verification is captured on screen.

```bash
id john.smith
id sarah.johnson
id michael.brown
```

**Evidence:** `screenshots/01-user-management/01-group-membership-verification.png`

---

## 02 — Directory Structure

```bash
cd it-support
sudo mkdir -p /shared/company/public
sudo mkdir -p /shared/company/developers
sudo mkdir -p /shared/company/finance
ls -l /shared/company
ls -l /shared/company/*
```

**Evidence:** `screenshots/02-directory-structure/01-mkdir-shared-company-structure.png`

---

## 03 — Ownership

```bash
ls -la
sudo chown root:developers developers
sudo chown root:finance finance
ls -la
ls -ld /shared/company/*
```

**Evidence:** `screenshots/03-ownership/01-chown-before-after.png`

---

## 04 — Permissions

```bash
sudo chmod 777 public
sudo chmod 770 developers
sudo chmod 770 finance
ls -ld /shared/company/*

# public was over-permissioned at 777; corrected:
sudo chmod 755 public
ls -ld /shared/company/*
```

**Evidence:** `screenshots/04-permissions/01-chmod-770-777-before-after.png`

Test files were created inside each directory to support later access testing **(not pictured in the organized evidence trail)**:

```bash
echo "Ubuntu Tech solves your IT problems" | sudo tee company-info.txt
echo "Developers are up and working" | sudo tee developers/project-notes.txt
echo "We have no money so far" | sudo tee finance/budget.txt
```

File-level ownership was then verified:

```bash
sudo ls -la public developers finance
```

**Evidence:** `screenshots/04-permissions/02-file-ownership-inside-directories.png`

---

## 05 — Access Testing

**john.smith** (developers group):

```bash
su - john.smith
cd /shared/company
cd developers && cat project-notes.txt
cd ../finance          # Permission denied
cd ../public && cat company-info.txt
```

**Evidence:** `screenshots/05-access-testing/01-john-smith-access-test.png`

**sarah.johnson** (developers group):

```bash
su - sarah.johnson
cd /shared/company
cd developers && cat project-notes.txt
cd ../finance          # Permission denied
cd public && cat company-info.txt
```

**Evidence:** `screenshots/05-access-testing/02-sarah-johnson-access-test.png`

**michael.brown** (finance group):

```bash
su - michael.brown
cd /shared/company
cd finance && cat budget.txt
cd ../developers       # Permission denied
cd public && cat company-info.txt
```

**Evidence:** `screenshots/05-access-testing/03-michael-brown-access-test.png`

---

## 06 — Troubleshooting

Reproduction and diagnosis:

```bash
su - john.smith
cd /shared/company
cd developers           # Permission denied (unexpected)
cd public
cd ..
id john.smith            # developers group missing
ls -ld developers
sudo usermod -aG developers john.smith   # blocked — not a sudoer
```

**Evidence:** `screenshots/06-troubleshooting/01-repro-and-diagnosis-group-membership-missing.png`

Fix and verification (performed as an administrator):

```bash
sudo usermod -aG developers john.smith
groups john.smith
su - john.smith
cd /shared/company
cd developers && cat project-notes.txt
```

**Evidence:** `screenshots/06-troubleshooting/02-fix-applied-and-verified.png`
