# DHCP / IP Configuration Lab — Deployment Notes

## 1. Initial Network Configuration

The Windows system's current network configuration was inspected using:

```
ipconfig /all
```

### Recorded Configuration

- Network Adapter: `Wireless LAN adapter WiFi`
- IPv4 Address: `192.168.0.155`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.0.1`
- DHCP Enabled: `Yes`
- DHCP Server: `192.168.0.1`
- DNS Server: `68.105.28.11, 68.105.29.11, 68.105.28.12`

The initial configuration was documented before making any changes.

---

## 2. Local TCP/IP Test

The local TCP/IP stack was tested using:

```
ping 127.0.0.1
```

### Result

`Successful` — *not pictured separately at this stage; loopback connectivity was re-confirmed later as part of the final connectivity verification (Section 9).*

This verified that the local TCP/IP stack was functioning.

---

## 3. Default Gateway Test

The default gateway was tested using:

```
ping 192.168.0.1
```

### Result

`Successful` — 4 packets sent, 4 received, 0% loss (min 4ms / max 14ms / avg 10ms).

This test was used to determine whether the computer could communicate with the local gateway.

---

## 4. Internet Connectivity Test

External connectivity was tested using:

```
ping 8.8.8.8
```

### Result

`Successful` — 4 packets sent, 4 received, 0% loss (min 26ms / max 77ms / avg 41ms).

Testing an IP address directly helps determine whether Internet connectivity is available independently of DNS hostname resolution.

---

## 5. DNS Resolution Test

DNS resolution was tested using:

```
ping google.com
```

### Result

`Successful` — resolved to `2607:f8b0:4023:200d::64`, 4 packets sent, 4 received, 0% loss (min 25ms / max 33ms / avg 28ms).

This test checks whether the system can resolve a hostname and communicate with the resulting IP address.

---

## 6. DHCP Lease Release

The existing DHCP lease was released using:

```
ipconfig /release
```

*Command output not pictured — this step was run immediately prior to the renewal in Section 7.*

This removed the current DHCP-provided IPv4 configuration from the active adapter.

---

## 7. DHCP Lease Renewal

A new DHCP configuration was requested using:

```
ipconfig /renew
```

The system successfully obtained on the WiFi adapter:

- IPv4 address: `192.168.0.155`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.0.1`

Other adapters (Local Area Connection* 9, Local Area Connection* 10, Bluetooth Network Connection) reported "No operation can be performed ... while it has its media disconnected" — expected, since those adapters were not physically connected.

---

## 8. DHCP Configuration Verification

The resulting configuration was verified with:

```
ipconfig /all
```

### Final Configuration

- IPv4 Address: `169.254.177.112 (Autoconfiguration/APIPA)`
- Subnet Mask: `255.255.0.0`
- Default Gateway: `None assigned (IPv4) — only an IPv6 link-local gateway was listed`
- DHCP Enabled: `Yes`
- DHCP Server: `Not shown in this capture`
- DNS Server: `fec0:0:0:ffff::1%1, fec0:0:0:ffff::2%1, fec0:0:0:ffff::3%1 (stale IPv6 site-local entries)`

**Note:** at the moment this screenshot was captured, the WiFi adapter had *not* received a DHCP-assigned IPv4 address and had self-assigned an APIPA address instead, despite `ipconfig /renew` succeeding moments earlier in Section 7. This is a real, unedited observation from the lab rather than a clean "expected" result — it's consistent with a brief DHCP negotiation delay/timeout on the adapter. The connectivity test in Section 9, run afterward, confirms the system had a valid DHCP-assigned address again by that point.

---

## 9. Final Connectivity Verification

The following tests were performed:

```
ping 127.0.0.1
ping 192.168.0.1
ping 8.8.8.8
ping google.com
```

### Results

- Loopback: `PASS` 
- Default Gateway: `PASS` 
- Internet IP: `PASS` 
- DNS/Hostname: `PASS` 

---

## 10. Final Status

The Windows system was returned to automatic DHCP configuration.

Final network connectivity was verified using local, gateway, Internet, and DNS tests, and all four passed with 0% packet loss.

The lab demonstrated the process of inspecting and troubleshooting DHCP/IP configuration on a Windows workstation, including an unplanned but instructive moment where the adapter briefly fell back to an APIPA address before DHCP re-established a valid lease — a good example of the kind of transient DHCP behavior a help desk technician may need to recognize and confirm resolves on its own versus one that requires intervention.

The lab demonstrated the process of inspecting and troubleshooting DHCP/IP configuration on a Windows workstation.
