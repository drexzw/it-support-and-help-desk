# DHCP / IP Configuration Lab — Deployment Notes

## 1. Initial Network Configuration

The Windows system's current network configuration was inspected using:

```cmd
ipconfig /all
```

### Recorded Configuration

* Network Adapter: `[Wi-Fi / Ethernet]`
* IPv4 Address: `[YOUR IPv4 ADDRESS]`
* Subnet Mask: `[YOUR SUBNET MASK]`
* Default Gateway: `[YOUR DEFAULT GATEWAY]`
* DHCP Enabled: `[Yes/No]`
* DHCP Server: `[YOUR DHCP SERVER]`
* DNS Server: `[YOUR DNS SERVER]`

The initial configuration was documented before making any changes.

---

## 2. Local TCP/IP Test

The local TCP/IP stack was tested using:

```cmd
ping 127.0.0.1
```

### Result

`127.0.0.1` responded successfully.

This verified that the local TCP/IP stack was functioning.

---

## 3. Default Gateway Test

The default gateway was tested using:

```cmd
ping [YOUR DEFAULT GATEWAY]
```

### Result

`[Successful / Failed]`

This test was used to determine whether the computer could communicate with the local gateway.

---

## 4. Internet Connectivity Test

External connectivity was tested using:

```cmd
ping 8.8.8.8
```

### Result

`[Successful / Failed]`

Testing an IP address directly helps determine whether Internet connectivity is available independently of DNS hostname resolution.

---

## 5. DNS Resolution Test

DNS resolution was tested using:

```cmd
ping google.com
```

### Result

`[Successful / Failed]`

This test checks whether the system can resolve a hostname and communicate with the resulting IP address.

---

## 6. DHCP Lease Release

The existing DHCP lease was released using:

```cmd
ipconfig /release
```

This removed the current DHCP-provided IPv4 configuration from the active adapter.

---

## 7. DHCP Lease Renewal

A new DHCP configuration was requested using:

```cmd
ipconfig /renew
```

The system successfully obtained:

* IPv4 address
* Subnet mask
* Default gateway
* DNS configuration

`[Modify this section if your actual result differed.]`

---

## 8. DHCP Configuration Verification

The resulting configuration was verified with:

```cmd
ipconfig /all
```

DHCP was confirmed as enabled.

### Final Configuration

* IPv4 Address: `[YOUR FINAL IPv4 ADDRESS]`
* Subnet Mask: `[YOUR FINAL SUBNET MASK]`
* Default Gateway: `[YOUR FINAL GATEWAY]`
* DHCP Enabled: `[Yes/No]`
* DHCP Server: `[YOUR DHCP SERVER]`
* DNS Server: `[YOUR DNS SERVER]`

---

## 9. Final Connectivity Verification

The following tests were performed:

```cmd
ping 127.0.0.1
ping [YOUR DEFAULT GATEWAY]
ping 8.8.8.8
ping google.com
```

### Results

* Loopback: `[PASS/FAIL]`
* Default Gateway: `[PASS/FAIL]`
* Internet IP: `[PASS/FAIL]`
* DNS/Hostname: `[PASS/FAIL]`

---

## 10. Final Status

The Windows system was returned to automatic DHCP configuration.

Final network connectivity was verified using local, gateway, Internet, and DNS tests.

The lab demonstrated the process of inspecting and troubleshooting DHCP/IP configuration on a Windows workstation.
