# Support Ticket — DHCP / IP Configuration Issue

## Ticket Information

**Ticket ID:** DHCP-001
**Category:** Network Connectivity
**Priority:** Medium
**Status:** Resolved
**Affected Device:** Windows workstation

---

## User Report

**Issue:**

User reported a loss of network connectivity and requested assistance troubleshooting the workstation's IP configuration.

---

## Initial Investigation

The technician began by checking the workstation's network configuration:

```cmd
ipconfig /all
```

The following information was reviewed:

* IPv4 address
* Subnet mask
* Default gateway
* DHCP status
* DHCP server
* DNS servers

---

## Troubleshooting Performed

### 1. Local TCP/IP Test

```cmd
ping 127.0.0.1
```

The loopback test was performed to verify the local TCP/IP stack.

**Result:** `[PASS]`

### 2. Default Gateway Test

```cmd
ping [DEFAULT GATEWAY]
```

This tested communication between the workstation and its local gateway.

**Result:** `[PASS]`

### 3. External IP Test

```cmd
ping 8.8.8.8
```

This tested external connectivity without relying on DNS hostname resolution.

**Result:** `[PASS]`

### 4. DNS Test

```cmd
ping google.com
```

This tested hostname resolution and external connectivity.

**Result:** `[PASS]`

### 5. DHCP Lease Renewal

The existing DHCP configuration was released:

```cmd
ipconfig /release
```

A new DHCP lease was then requested:

```cmd
ipconfig /renew
```

The resulting IP configuration was verified with:

```cmd
ipconfig /all
```

---

## Resolution

The workstation was returned to automatic DHCP configuration.

The final configuration was verified and connectivity was tested again.

Final tests included:

```cmd
ping 127.0.0.1
ping [DEFAULT GATEWAY]
ping 8.8.8.8
ping google.com
```

**Final Result:** `[Resolved ]`

---



## Technician Notes

The troubleshooting process demonstrated the importance of isolating network problems by testing connectivity at different layers.

Testing the gateway, an external IP address, and a hostname helps distinguish between local network, Internet connectivity, and DNS-related problems.

A `169.254.x.x` APIPA address would be an important indicator of a possible DHCP configuration or connectivity problem.

---

## Skills Demonstrated

* Windows network troubleshooting
* DHCP lease management
* IPv4 configuration
* DNS troubleshooting
* Connectivity testing
* Command Prompt
* Technical documentation
* Help-desk troubleshooting methodology
