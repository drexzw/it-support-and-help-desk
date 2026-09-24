# DHCP / IP Configuration Lab — Deployment Notes

## Environment Setup

A second Elastic Network Interface (ENI) was created and attached to EC2 instance `EC2AMAZ-VQU1ICD` (Windows Server, `us-east-2`) via the EC2 console, without stopping the instance. In Windows, this appeared as a new adapter alongside the existing one. The new adapter was renamed to `Test-NIC` to clearly distinguish it from the primary adapter carrying the active RDP session.

---

## 1. Baseline Configuration

The healthy starting configuration was recorded on both adapters using:

```
ipconfig /all
```

### Recorded Configuration — Test-NIC (originally "Ethernet 2")

- Description: `Amazon Elastic Network Adapter #2`
- DHCP Enabled: `Yes`
- IPv4 Address: `172.31.36.32`
- Subnet Mask: `255.255.240.0`
- Default Gateway: `172.31.32.1`
- DHCP Server: `172.31.32.1`
- DNS Servers: `172.31.0.2`
- Lease Obtained: `Thursday, September 24, 2026 5:30:57 AM`
- Lease Expires: `Thursday, September 24, 2026 6:30:57 AM`

### Recorded Configuration — Ethernet 3 (primary, untouched throughout)

- DHCP Enabled: `Yes`
- IPv4 Address: `172.31.44.53`
- Subnet Mask: `255.255.240.0`
- Default Gateway: `172.31.32.1`
- DHCP Server: `172.31.32.1`
- DNS Servers: `172.31.0.2`

*(Screenshot 01: `01-baseline-config.png`)*

---

## 2. Baseline Connectivity Test

The following tests were run to confirm the healthy starting state:

```
ping 127.0.0.1
ping 172.31.32.1
ping 8.8.8.8
ping google.com
```

### Results

- Loopback (`127.0.0.1`): `PASS` — 0% loss
- Default Gateway (`172.31.32.1`): `PASS` — 0% loss
- Internet IP (`8.8.8.8`): `PASS` — 0% loss, TTL=116
- DNS/Hostname (`google.com` → `142.251.210.110`): `PASS` — 0% loss

*(Screenshot 02: `02-baseline-connectivity.png`)*

---

## 3. Fault Introduced — Static IP Misconfiguration

On the `Test-NIC` adapter only, the IPv4 configuration was manually changed from automatic (DHCP) to a static address outside the instance's actual subnet:

- IP Address (manually set): `10.0.0.30`
- Subnet Mask (manually set): `255.255.255.0`
- Default Gateway (manually set): `10.0.0.1`
- DHCP Enabled: `No`

The primary adapter (`Ethernet 3`), carrying the active RDP session, was left completely unchanged and remained on its original DHCP-assigned configuration throughout.

*(Screenshot 03: `03-misconfigured-static-ip.png`)*

---

## 4. Symptom Reproduction

With the static misconfiguration active, connectivity to the (incorrect) configured gateway was tested:

```
ping 10.0.0.1
```

### Result

`Failed` — 4 packets sent, 0 received, 100% loss ("Request timed out" x4).

This confirmed the adapter could not reach any gateway under its current configuration.

*(Screenshot 04: `04-failed-gateway-ping.png`)*

---

## 5. Diagnosis

The `Test-NIC` adapter's IPv4 Properties were opened via Network Connections. The dialog confirmed "Use the following IP address" was selected, with the incorrect static IP (`10.0.0.30`), subnet mask (`255.255.255.0`), and gateway (`10.0.0.1`) manually entered — none of which matched the instance's actual VPC subnet.

This identified the root cause: a manually-configured static IP address outside the valid subnet range, rather than a DHCP server or network-side failure.

*(Screenshot 05: `05-diagnosis-static-ip-found.png`)*

---

## 6. Resolution — Restore Automatic DHCP Configuration

In the same IPv4 Properties dialog, the adapter was switched back to "Obtain an IP address automatically." The lease was then explicitly released and renewed:

```
ipconfig /release "Test-NIC"
ipconfig /renew "Test-NIC"
```

### Result

The `Test-NIC` adapter obtained a new valid DHCP lease:

- IPv4 Address: `172.31.36.32`
- Subnet Mask: `255.255.240.0`
- Default Gateway: `172.31.32.1`
- DHCP Enabled: `Yes`
- Lease Obtained: `Thursday, September 24, 2026 6:00:53 AM`
- Lease Expires: `Thursday, September 24, 2026 7:00:53 AM`

This matches the adapter's original baseline configuration from Section 1.

*(Screenshot 06: `06-dhcp-fix-applied.png`)*

---

## 7. Final Verification

`ipconfig /all`, captured in Screenshot 06, confirms `Test-NIC` returned to a valid DHCP-assigned IP address and gateway matching the original baseline.

**Not pictured:** a dedicated post-fix ping test (e.g., `ping 172.31.32.1`) re-confirming active connectivity was not captured as a separate screenshot. The restored DHCP configuration strongly implies connectivity was restored, but this was not independently verified with a ping screenshot at the time of writing.

---

## 8. Final Status

The static IP misconfiguration on the secondary adapter (`Test-NIC`) was identified and corrected by restoring automatic DHCP configuration. The adapter successfully obtained a valid IP address, subnet mask, and gateway from the DHCP server. The primary adapter carrying the active RDP session was never modified, demonstrating a safe approach to fault isolation and testing on a live server.
