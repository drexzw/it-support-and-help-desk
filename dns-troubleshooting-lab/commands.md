# DNS Troubleshooting Lab — Commands

This document lists the PowerShell commands used during the DNS troubleshooting lab, what each one was used to determine, and whether it's backed by a screenshot in `screenshots/`.

## 1. View Detailed Network Configuration 📸

```powershell
ipconfig /all
```

**Purpose:** Displays full network adapter info — IPv4 address, MAC address, DHCP status, default gateway, DNS servers.

**Screenshot:** `04-working-dns-configuration.png`

**Lab result (working baseline):**

```text
IPv4 Address:    172.31.33.17
Subnet Mask:     255.255.240.0
Default Gateway: 172.31.32.1
DNS Server:      172.31.0.2
```

---

## 2. Test the Default Gateway

```powershell
ping 172.31.32.1
```

**Purpose:** Confirms the workstation can reach its default gateway before testing anything further out.

**Screenshot:** none — performed during baseline setup, not individually captured.

**Result:** 4 packets sent, 4 received, 0% loss.

---

## 3. Test Internet Connectivity Without DNS

```powershell
ping 8.8.8.8
```

**Purpose:** Tests connectivity using a raw IP address, which doesn't depend on DNS. This is what separates "internet is down" from "DNS is down."

**Screenshot:** none — performed during baseline setup, not individually captured.

**Result:** 4 packets sent, 4 received, 0% loss.

---

## 4. Query DNS

```powershell
nslookup google.com
```

**Purpose:** Tests whether the configured DNS server can resolve a hostname.

**Screenshots:**
- Baseline success: not individually captured
- Failure (after DNS was broken): `05-dns-resolution-failure.png`
- Restored success: `06-dns-resolution-restored.png`

**Failure result:** DNS request timed out against `192.0.2.1`.

---

## 5. Test Hostname Connectivity

```powershell
ping google.com
```

**Purpose:** Tests hostname resolution and connectivity together — this is the command that reproduces the user's actual symptom.

**Screenshots:**
- Baseline success: not individually captured
- Failure: `05-dns-resolution-failure.png` → `Ping request could not find host google.com.`
- Restored: `06-dns-resolution-restored.png` → 4 packets sent, 4 received, 0% loss

---

## 6. Identify Network Adapters

```powershell
Get-NetAdapter
```

**Purpose:** Confirms the adapter name and Interface Index needed for the `Set-DnsClientServerAddress` commands below.

**Screenshot:** none — the resulting Interface Index (`8`, `Ethernet 3`) is visible in the outputs of commands 1 and 7 instead.

**Lab adapter:**

```text
Name: Ethernet 3
Interface Description: Amazon Elastic Network Adapter
Interface Index: 8
Status: Up
```

---

## 7. Change the DNS Server (break it, intentionally)

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 192.0.2.1
```

**Purpose:** Intentionally points the adapter at an unreachable DNS server to reproduce a DNS failure on demand. `192.0.2.0/24` is a documentation/testing range (RFC 5737) — it will never respond, so the failure is reliable and repeatable.

**Screenshot:** none directly — but `05-dns-resolution-failure.png` confirms the change took effect, since `nslookup` shows the client querying `192.0.2.1`.

---

## 8. Verify DNS Configuration

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

**Purpose:** Confirms which DNS server is actually configured on the interface — used both to confirm the break and to confirm the restore.

**Screenshot:** `06-dns-resolution-restored.png` (restore verification)

| State     | DNS Server   |
| --------- | ------------ |
| Broken    | `192.0.2.1`  |
| Restored  | `172.31.0.2` |

---

## 9. Restore the Working DNS Server 📸

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 172.31.0.2
```

**Purpose:** Restores the original working DNS configuration.

**Screenshot:** `06-dns-resolution-restored.png`

---

## Troubleshooting Sequence

```text
ipconfig /all
    ↓
ping 172.31.32.1
    ↓
ping 8.8.8.8
    ↓
nslookup google.com
    ↓
ping google.com
    ↓
Get-NetAdapter
    ↓
Break DNS (Set-DnsClientServerAddress → 192.0.2.1)
    ↓
Confirm failure (ping / nslookup)
    ↓
Restore DNS (Set-DnsClientServerAddress → 172.31.0.2)
    ↓
Verify configuration (Get-DnsClientServerAddress)
    ↓
Confirm resolution restored (nslookup / ping)
```

## Command-to-Question Reference

| Question                                  | Command                        |
| ------------------------------------------ | -------------------------------- |
| What is my full network configuration?    | `ipconfig /all`                  |
| Can I reach my gateway?                   | `ping 172.31.32.1`               |
| Can I reach the internet without DNS?     | `ping 8.8.8.8`                   |
| Can DNS resolve a hostname?               | `nslookup google.com`            |
| Can Windows resolve and reach a hostname? | `ping google.com`                |
| What network adapters exist?              | `Get-NetAdapter`                 |
| What DNS server is configured?            | `Get-DnsClientServerAddress`     |
| How do I change the DNS server?           | `Set-DnsClientServerAddress`     |

## Troubleshooting Principle

Commands should answer questions. Instead of "which command should I run," the better question is "what do I need to determine next?" — the command follows from that.
