# DHCP / IP Configuration Lab — Commands

## `ipconfig`

Displays basic IP configuration information.

```cmd
ipconfig
```

Useful for quickly checking the computer's IPv4 address, subnet mask, and default gateway.

---

## `ipconfig /all`

Displays detailed network configuration.

```cmd
ipconfig /all
```

Useful information includes:

* IPv4 address
* Subnet mask
* Default gateway
* DHCP status
* DHCP server
* DNS servers
* MAC address
* DHCP lease information

---

## `ipconfig /release`

Releases the current DHCP lease.

```cmd
ipconfig /release
```

This removes the current DHCP-provided IPv4 configuration from the adapter.

---

## `ipconfig /renew`

Requests a new DHCP lease.

```cmd
ipconfig /renew
```

This asks the DHCP service for updated network configuration.

---

## `ping`

Tests network connectivity.

```cmd
ping 127.0.0.1
```

Tests the local TCP/IP stack.

```cmd
ping [DEFAULT-GATEWAY]
```

Tests connectivity to the local gateway.

```cmd
ping 8.8.8.8
```

Tests connectivity to an external IP address.

```cmd
ping google.com
```

Tests both hostname resolution and connectivity.

---

## `nslookup`

Tests DNS name resolution.

```cmd
nslookup google.com
```

Useful when a hostname does not resolve correctly.

---

## `ncpa.cpl`

Opens Windows Network Connections.

```text
ncpa.cpl
```

This can be used to access network adapter properties and configure IPv4 settings.

---

## Troubleshooting Reference

| Command/Test          | What it helps determine                 |
| --------------------- | --------------------------------------- |
| `ipconfig`            | Current IP configuration                |
| `ipconfig /all`       | Detailed DHCP/DNS/network configuration |
| `ipconfig /release`   | Releases DHCP lease                     |
| `ipconfig /renew`     | Requests a new DHCP lease               |
| `ping 127.0.0.1`      | Local TCP/IP functionality              |
| `ping [gateway]`      | Local network/gateway connectivity      |
| `ping 8.8.8.8`        | External IP connectivity                |
| `ping google.com`     | DNS + connectivity                      |
| `nslookup google.com` | DNS resolution                          |
| `ncpa.cpl`            | Windows network adapter configuration   |
