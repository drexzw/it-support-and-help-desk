# DNS Troubleshooting Lab

## Overview

This lab simulates a Help Desk incident involving a Windows Server 2022 EC2 instance that is unable to access websites using domain names.

The lab demonstrates how to troubleshoot a DNS-related connectivity issue by separating network connectivity into different layers and using evidence to identify the root cause.

The troubleshooting process followed this sequence:

1. Verify IP configuration
2. Test default gateway connectivity
3. Test internet connectivity using an IP address
4. Test DNS resolution
5. Test hostname connectivity
6. Intentionally introduce a DNS failure
7. Identify the failed layer
8. Restore the correct DNS configuration
9. Verify that connectivity has been restored

## Scenario

### Ticket ID

`DNS-001`

### Priority

Medium

### Category

Network Connectivity / DNS

### User Report

> "My computer is connected to the internet, but I can't access websites by their names."

The objective was to determine whether the issue was caused by:

* Incorrect IP configuration
* Gateway connectivity
* General internet connectivity
* DNS resolution
* Incorrect DNS server configuration

## Environment

### AWS

* Amazon EC2
* Windows Server 2022 Base
* Instance type: `t3.micro`
* Default VPC
* Public IPv4 address enabled
* Security Group configured for RDP
* EBS-backed EC2 instance

### Windows

* Windows Server 2022
* PowerShell
* Amazon Elastic Network Adapter (ENA)
* Network interface: `Ethernet 3`
* Interface Index: `8`

## Initial Network Configuration

| Configuration   | Value           |
| --------------- | --------------- |
| IPv4 Address    | `172.31.33.17`  |
| Subnet Mask     | `255.255.240.0` |
| Default Gateway | `172.31.32.1`   |
| DNS Server      | `172.31.0.2`    |

## Troubleshooting Process

### 1. Verify IP Configuration

```powershell
ipconfig
```

The system had a valid IPv4 configuration, subnet mask, and default gateway.

### 2. Test Default Gateway Connectivity

```powershell
ping 172.31.32.1
```

Result:

* 4 packets sent
* 4 packets received
* 0% packet loss

The workstation could communicate with its default gateway.

### 3. Test Internet Connectivity Without DNS

```powershell
ping 8.8.8.8
```

Result:

* 4 packets sent
* 4 packets received
* 0% packet loss

This demonstrated that internet connectivity was working independently of DNS.

### 4. Test DNS Resolution

```powershell
nslookup google.com
```

The query successfully returned Google's IP addresses.

### 5. Test Hostname Connectivity

```powershell
ping google.com
```

Result:

* 4 packets sent
* 4 packets received
* 0% packet loss

At this point, the network was functioning normally.

## Simulating the DNS Failure

The original DNS server was:

```text
172.31.0.2
```

The DNS server was intentionally changed to:

```text
192.0.2.1
```

using:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 192.0.2.1
```

The configuration was verified with:

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

The `192.0.2.0/24` network is reserved for documentation and testing purposes.

## Failure Testing

### Internet Connectivity by IP

```powershell
ping 8.8.8.8
```

Result:

* 4 packets sent
* 4 packets received
* 0% packet loss

Internet connectivity remained functional.

### Hostname Connectivity

```powershell
ping google.com
```

Result:

```text
Ping request could not find host google.com.
```

### DNS Query

```powershell
nslookup google.com
```

The DNS request timed out.

## Root Cause

The Windows network adapter had been configured to use an invalid/unreachable DNS server:

```text
192.0.2.1
```

The underlying network connection was still operational, but the workstation could no longer resolve domain names.

| Test                        | Result |
| --------------------------- | ------ |
| IP configuration            | PASS   |
| Gateway connectivity        | PASS   |
| Internet connectivity by IP | PASS   |
| DNS resolution              | FAIL   |
| Hostname connectivity       | FAIL   |

This isolated the problem to DNS rather than general network connectivity.

## Resolution

The original DNS server was restored:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 172.31.0.2
```

The configuration was verified:

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

DNS resolution was tested again:

```powershell
nslookup google.com
```

Finally:

```powershell
ping google.com
```

Result:

* 4 packets sent
* 4 packets received
* 0% packet loss

DNS resolution and hostname connectivity were successfully restored.

## Troubleshooting Methodology

The most important lesson from this lab was not memorizing commands.

The goal was to understand:

> What question am I trying to answer, and which tool can provide the evidence?

| Question                                     | Tool                         |
| -------------------------------------------- | ---------------------------- |
| What is the machine's network configuration? | `ipconfig`                   |
| Can the machine reach its gateway?           | `ping <gateway>`             |
| Can the machine reach the internet by IP?    | `ping 8.8.8.8`               |
| Can DNS resolve a hostname?                  | `nslookup <hostname>`        |
| Can Windows communicate using a hostname?    | `ping <hostname>`            |
| Which network adapters exist?                | `Get-NetAdapter`             |
| Which DNS server is configured?              | `Get-DnsClientServerAddress` |
| How can the DNS configuration be changed?    | `Set-DnsClientServerAddress` |

## Real-World Help Desk Application

A user may report:

> "The internet isn't working."

A Help Desk technician should not immediately assume that the entire network connection is down.

Instead, the technician should determine what is actually failing.

```text
Can the computer reach the gateway?
            ↓
Can it reach the internet by IP?
            ↓
Can DNS resolve a hostname?
            ↓
Can the computer connect using that hostname?
```

This approach helps isolate problems and prevents unnecessary configuration changes.

## Important DNS Consideration

DNS can be working correctly while a specific internal hostname is not.

For example:

```text
google.com                  → works
youtube.com                 → works
fileserver.company.local    → fails
```

Possible causes include:

* Missing DNS record
* Incorrect DNS record
* Internal DNS zone problem
* Active Directory DNS
* Split-DNS configuration
* VPN configuration
* DNS cache
* Internal server availability

In a corporate environment, replacing an organization's DNS server with a public DNS server may restore public internet access while breaking internal resources.

## Screenshots

```text
01-ec2-security-group-rdp.png
02-rdp-client.png
03-windows-ec2-desktop.png
04-working-dns-configuration.png
05-dns-resolution-failure.png
06-dns-resolution-restored.png
```

## Skills Demonstrated

* AWS EC2
* Windows Server
* RDP
* AWS Security Groups
* IPv4 networking
* DNS troubleshooting
* PowerShell
* `ipconfig`
* `ping`
* `nslookup`
* `Get-NetAdapter`
* `Get-DnsClientServerAddress`
* `Set-DnsClientServerAddress`
* Layered troubleshooting
* Root cause identification
* Incident documentation
* Verification and remediation

## Final Outcome

The simulated DNS incident was successfully diagnosed and resolved.

The final troubleshooting model was:

```text
Identify the symptom
        ↓
Collect evidence
        ↓
Test each network layer
        ↓
Isolate the failure
        ↓
Apply the appropriate fix
        ↓
Verify the fix
        ↓
Document the incident
```
