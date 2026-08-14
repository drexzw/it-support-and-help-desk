# Support Ticket — DNS Resolution Failure

## Ticket Information

| Field       | Information                 |
| ----------- | ---------------------------- |
| Ticket ID   | `DNS-001`                    |
| Priority    | Medium                        |
| Category    | Network Connectivity / DNS   |
| Environment | Windows Server 2022 EC2      |
| Status      | Resolved                      |

## User Report

> "My computer is connected to the internet, but I can't access websites by their names."

## Initial Assessment

The reported symptom sounded like a general internet connectivity problem. Rather than changing any network settings immediately, connectivity was tested layer by layer to find out exactly where it was failing.

## Troubleshooting Performed

### 1. Checked Network Configuration

```powershell
ipconfig /all
```

![ipconfig /all showing a valid, working configuration](screenshots/04-working-dns-configuration.png)

**Result:**

```text
IPv4 Address:     172.31.33.17
Subnet Mask:      255.255.240.0
Default Gateway:  172.31.32.1
DNS Server:       172.31.0.2
```

**Finding:** IP configuration and DNS server assignment were both valid.

### 2. Tested Default Gateway

```powershell
ping 172.31.32.1
```

**Result:** 4 packets sent, 4 received, 0% loss. *(Performed as part of the baseline check; not separately screenshotted.)*

**Finding:** The workstation could reach its default gateway.

### 3. Tested Internet Connectivity Without DNS

```powershell
ping 8.8.8.8
```

**Result:** 4 packets sent, 4 received, 0% loss. *(Performed as part of the baseline check; not separately screenshotted.)*

**Finding:** Internet connectivity was working independent of DNS, since this test used a raw IP rather than a hostname.

### 4. Tested DNS Resolution and Hostname Connectivity

```powershell
nslookup google.com
ping google.com
```

**Result:** Both succeeded at baseline. *(Not separately screenshotted — confirmed working before the failure was intentionally introduced below.)*

**Finding:** DNS and hostname connectivity were both functioning normally before the incident was simulated.

## Controlled Failure Simulation

To reproduce a DNS incident under controlled conditions, the DNS server was intentionally pointed at an unreachable address:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 192.0.2.1
```

`192.0.2.0/24` is reserved for documentation/testing (RFC 5737) and will never respond, making the failure reliable to reproduce.

## Failure Reproduction

```powershell
ping google.com
nslookup google.com
```

![Hostname ping and DNS query both failing against the unreachable server](screenshots/05-dns-resolution-failure.png)

**Results:**

* `ping google.com` → `Ping request could not find host google.com.`
* `nslookup google.com` → DNS request timed out, querying `192.0.2.1`

**Finding:** The configured DNS server was not responding, and hostname resolution failed as a direct result.

## Root Cause

The workstation was configured to use an invalid/unreachable DNS server (`192.0.2.1`). The underlying network connection remained fully operational — the failure was isolated entirely to DNS resolution.

| Test                          | Result       | Evidence                 |
| ------------------------------ | ------------ | -------------------------- |
| IP configuration                | PASS         | Screenshot 04              |
| Gateway connectivity            | PASS         | Performed, not pictured    |
| Internet connectivity by IP     | PASS         | Performed, not pictured    |
| DNS resolution                  | FAIL         | Screenshot 05              |
| Hostname connectivity           | FAIL         | Screenshot 05              |

## Resolution

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 172.31.0.2
Get-DnsClientServerAddress -InterfaceIndex 8
nslookup google.com
ping google.com
```

![DNS restored, verified, and resolution confirmed working](screenshots/06-dns-resolution-restored.png)

**Final Result:** `Get-DnsClientServerAddress` confirms `172.31.0.2` restored on interface 8; `nslookup` resolves successfully; `ping google.com` returns 4 packets sent, 4 received, 0% loss.

## Resolution Summary

**Root Cause:** Incorrect DNS server configuration on the network adapter.

**Resolution:** Restored the original working DNS server (`172.31.0.2`) on interface 8.

**Verification:** Confirmed via `Get-DnsClientServerAddress`, a successful `nslookup`, and a successful `ping google.com`.

**Final Status:** Resolved

## Technician Notes

The issue was isolated by comparing IP-based connectivity against hostname-based connectivity. The workstation kept working internet access after DNS was broken, which confirmed:

```text
Network connectivity ≠ DNS connectivity
```

A DNS failure can block hostname-based access to everything — internal and external — even while the underlying network connection is completely healthy. That distinction is what keeps a technician from making unnecessary changes to the wrong layer.

## Recommended Follow-Up

In a production environment, before closing this ticket I'd want to know *why* the DNS server got misconfigured in the first place — this lab only simulated the symptom, not a real root cause. Areas worth checking:

* DHCP configuration (was this pushed by DHCP or set manually?)
* Network adapter configuration drift
* VPN client overriding DNS settings
* Organizational DNS policy or Group Policy changes
* Active Directory DNS health
* Recent network configuration changes in change management logs

## Ticket Closure

**Resolution:** Restored the workstation's original DNS server configuration.

**Verification:** Confirmed successful DNS resolution and hostname connectivity.

**Final Status:** Resolved
