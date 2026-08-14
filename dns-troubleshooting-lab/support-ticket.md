# Support Ticket — DNS Resolution Failure

## Ticket Information

| Field       | Information                |
| ----------- | -------------------------- |
| Ticket ID   | `DNS-001`                  |
| Priority    | Medium                     |
| Category    | Network Connectivity / DNS |
| Environment | Windows Server 2022 EC2    |
| Status      | Resolved                   |

## User Report

> "My computer is connected to the internet, but I can't access websites by their names."

## Initial Assessment

The reported symptom appeared to indicate an internet connectivity problem.

Rather than immediately changing the network configuration, connectivity was tested progressively to determine which layer was failing.

## Troubleshooting Performed

### 1. Checked IP Configuration

```powershell
ipconfig
```

The workstation had a valid IPv4 configuration:

```text
IPv4 Address:     172.31.33.17
Subnet Mask:      255.255.240.0
Default Gateway:  172.31.32.1
```

**Finding:** IP configuration appeared valid.

### 2. Tested Default Gateway

```powershell
ping 172.31.32.1
```

**Result:**

```text
4 packets sent
4 packets received
0 packets lost
```

**Finding:** The workstation could communicate with its default gateway.

### 3. Tested Internet Connectivity Without DNS

```powershell
ping 8.8.8.8
```

**Result:**

```text
4 packets sent
4 packets received
0 packets lost
```

**Finding:** Internet connectivity was working.

Because the test used an IP address rather than a hostname, it did not depend on DNS resolution.

### 4. Tested DNS Resolution

```powershell
nslookup google.com
```

The initial query successfully returned Google's IP addresses.

**Finding:** DNS was functioning during the baseline test.

### 5. Tested Hostname Connectivity

```powershell
ping google.com
```

**Result:**

```text
4 packets sent
4 packets received
0 packets lost
```

**Finding:** Hostname resolution and connectivity were working normally.

## Controlled Failure Simulation

A DNS failure was intentionally introduced to simulate a real troubleshooting incident.

Original DNS server:

```text
172.31.0.2
```

Testing DNS server:

```text
192.0.2.1
```

Command used:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 192.0.2.1
```

The change was verified with:

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

## Failure Reproduction

### Internet Connectivity

```powershell
ping 8.8.8.8
```

**Result:**

```text
4 packets sent
4 packets received
0 packets lost
```

**Finding:** Internet connectivity remained functional.

### Hostname Resolution

```powershell
ping google.com
```

**Result:**

```text
Ping request could not find host google.com.
```

**Finding:** Hostname resolution failed.

### DNS Query

```powershell
nslookup google.com
```

**Result:** The DNS request timed out.

**Finding:** The configured DNS server was not successfully responding to DNS queries.

## Root Cause

The workstation was configured to use an invalid/unreachable DNS server:

```text
192.0.2.1
```

The underlying network connection remained operational, but DNS resolution failed.

| Test                        | Result |
| --------------------------- | ------ |
| IP configuration            | PASS   |
| Gateway connectivity        | PASS   |
| Internet connectivity by IP | PASS   |
| DNS resolution              | FAIL   |
| Hostname connectivity       | FAIL   |

The issue was isolated to DNS resolution.

## Resolution

The original DNS server was restored:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 172.31.0.2
```

The configuration was verified:

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

DNS resolution was tested:

```powershell
nslookup google.com
```

The original user-facing symptom was then tested:

```powershell
ping google.com
```

**Final Result:**

```text
4 packets sent
4 packets received
0 packets lost
```

## Resolution Summary

**Root Cause:** Incorrect DNS server configuration.

**Resolution:** Restored the original working DNS server.

**Verification:** Successful DNS resolution and hostname connectivity.

**Final Status:** Resolved

## Technician Notes

The issue was isolated by comparing IP-based connectivity with hostname-based connectivity.

The workstation retained internet connectivity after the DNS configuration was changed.

This demonstrated that:

```text
Network connectivity ≠ DNS connectivity
```

A DNS failure can prevent users from accessing resources by hostname even when the underlying network and internet connection are functioning normally.

## Recommended Follow-Up

In a production environment, the technician should determine why the incorrect DNS server was configured before closing the incident.

Potential areas for investigation include:

* DHCP configuration
* Network adapter configuration
* VPN configuration
* Organizational DNS policies
* Active Directory DNS
* DNS server availability
* Recent network configuration changes

## Ticket Closure

**Resolution:** Restored the workstation's original DNS server configuration.

**Verification:** Confirmed successful DNS resolution and hostname connectivity.

**Final Status:** Resolved
