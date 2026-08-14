# DNS Troubleshooting Lab — Commands

This document contains the PowerShell commands used during the DNS troubleshooting lab and explains what each command was used to determine.

## 1. View IP Configuration

```powershell
ipconfig
```

### Purpose

Displays the basic IPv4 configuration of the Windows machine.

### Lab Configuration

```text
IPv4 Address:    172.31.33.17
Subnet Mask:     255.255.240.0
Default Gateway: 172.31.32.1
```

---

## 2. View Detailed Network Configuration

```powershell
ipconfig /all
```

### Purpose

Displays detailed network adapter information, including:

* IPv4 address
* MAC address
* DHCP status
* Default gateway
* DNS servers
* Adapter information

### Lab DNS Server

```text
172.31.0.2
```

---

## 3. Test the Default Gateway

```powershell
ping 172.31.32.1
```

### Purpose

Tests communication between the Windows workstation and its default gateway.

### Lab Result

```text
4 packets sent
4 packets received
0 packets lost
```

---

## 4. Test Internet Connectivity Without DNS

```powershell
ping 8.8.8.8
```

### Purpose

Tests internet connectivity using an IP address rather than a hostname.

### Why It Matters

This helps separate an internet connectivity problem from a DNS resolution problem.

### Lab Result

```text
4 packets sent
4 packets received
0 packets lost
```

---

## 5. Query DNS

```powershell
nslookup google.com
```

### Purpose

Tests whether the configured DNS server can resolve a hostname.

### Working Result

The initial query successfully returned Google's IP addresses.

### Failure Result

After the DNS configuration was intentionally changed, the DNS request timed out.

---

## 6. Test Hostname Connectivity

```powershell
ping google.com
```

### Purpose

Tests hostname resolution and connectivity.

### Working Result

```text
4 packets sent
4 packets received
0 packets lost
```

### Failure Result

```text
Ping request could not find host google.com.
```

---

## 7. Identify Network Adapters

```powershell
Get-NetAdapter
```

### Purpose

Displays the network interfaces available on the Windows machine.

### Lab Adapter

```text
Name: Ethernet 3
Interface Description: Amazon Elastic Network Adapter
Interface Index: 8
Status: Up
```

The interface index was required when changing the DNS configuration.

---

## 8. Change the DNS Server

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 192.0.2.1
```

### Purpose

Changes the DNS server configured on interface index `8`.

### Lab Use

This intentionally changed the working DNS server to:

```text
192.0.2.1
```

The `192.0.2.0/24` network is reserved for documentation and testing purposes.

---

## 9. Verify DNS Configuration

```powershell
Get-DnsClientServerAddress -InterfaceIndex 8
```

### Failure Configuration

```text
192.0.2.1
```

### Restored Configuration

```text
172.31.0.2
```

---

## 10. Restore the Working DNS Server

```powershell
Set-DnsClientServerAddress -InterfaceIndex 8 -ServerAddresses 172.31.0.2
```

### Purpose

Restores the original working DNS configuration.

---

## Troubleshooting Sequence

```text
ipconfig
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
Change DNS
    ↓
Verify DNS configuration
    ↓
Repeat connectivity tests
    ↓
Restore DNS
    ↓
Verify configuration
    ↓
Test hostname again
```

## Command-to-Question Reference

| Question                                  | Command                      |
| ----------------------------------------- | ---------------------------- |
| What is my IP configuration?              | `ipconfig`                   |
| What DNS server am I using?               | `ipconfig /all`              |
| Can I reach my gateway?                   | `ping 172.31.32.1`           |
| Can I reach the internet without DNS?     | `ping 8.8.8.8`               |
| Can DNS resolve a hostname?               | `nslookup google.com`        |
| Can Windows resolve and reach a hostname? | `ping google.com`            |
| What network adapters exist?              | `Get-NetAdapter`             |
| What DNS server is configured?            | `Get-DnsClientServerAddress` |
| How do I change the DNS server?           | `Set-DnsClientServerAddress` |

## Troubleshooting Principle

Commands should answer questions.

Instead of thinking:

> "Which command should I run?"

Think:

> "What do I need to determine next?"
