# DHCP / IP Configuration & Troubleshooting Lab

## Scenario

**Ticket:** Secondary network adapter on server `EC2AMAZ-VQU1ICD` is not obtaining a valid IP address via DHCP and cannot reach the network.

**User Report:** *"The secondary NIC on the Windows server isn't getting an IP from DHCP. It looks like it might have been manually configured at some point. Can you check it and restore normal connectivity?"*

Full details in [support-ticket.md](./support-ticket.md).

## Documentation

- [support-ticket.md](./support-ticket.md) — the simulated help-desk ticket: report, root cause, resolution, closure
- [deployment-notes.md](./deployment-notes.md) — full technical log, command-by-command, with recorded output

## Overview

This lab demonstrates diagnosing and resolving a DHCP/IP misconfiguration on a Windows Server EC2 instance. Rather than observing a healthy DHCP renewal, this version of the lab deliberately introduces a real fault — a static IP misconfigured outside the valid subnet range on a secondary network adapter — then walks through diagnosis, resolution, and verification, the way an actual help-desk ticket would be worked.

A second Elastic Network Interface (ENI) was attached to the instance specifically so the fault could be introduced and fixed without risking the adapter carrying the active Remote Desktop session.

## Environment

* Platform: AWS EC2 (Windows Server), region `us-east-2`
* Hostname: `EC2AMAZ-VQU1ICD`
* Network adapters:
  * **Ethernet 3** — primary adapter, carries the RDP session, left untouched throughout
  * **Test-NIC** (originally "Ethernet 2") — secondary adapter, used for the fault/fix
* Primary Tools:
  * PowerShell / Command Prompt
  * `ipconfig`
  * `ping`
  * Windows Network Connections (`ncpa.cpl`)

## Objectives

* Attach and configure a secondary network interface on an EC2 instance
* Identify a DHCP/IP misconfiguration using `ipconfig /all`
* Reproduce and confirm a connectivity failure caused by an incorrect static IP
* Diagnose the root cause via the adapter's IPv4 properties
* Resolve the issue by restoring automatic DHCP configuration
* Verify the fix and document the incident in help-desk ticket format

## Key Concepts

### DHCP

Dynamic Host Configuration Protocol (DHCP) automatically provides network configuration information to clients, including IPv4 address, subnet mask, default gateway, and DNS servers.

### Static IP Misconfiguration

When an adapter is manually assigned an IP address outside its actual subnet, it can no longer reach the correct gateway or any other host on the real network, even though the adapter itself shows as "up."

### Elastic Network Interface (ENI)

An ENI is a virtual network interface that can be attached to an EC2 instance. A single instance can have multiple ENIs, each with its own IP configuration — used here to safely isolate the test adapter from the one carrying the management (RDP) connection.

## Troubleshooting Method

1. Record the healthy baseline configuration and connectivity on both adapters.
2. Introduce a static IP misconfiguration on the secondary adapter only.
3. Reproduce the resulting connectivity failure.
4. Diagnose the root cause via the adapter's IPv4 properties.
5. Restore automatic DHCP configuration.
6. Verify the adapter receives a valid lease and document the outcome.

## Screenshots

| # | Screenshot | Description |
|---|---|---|
| 01 | `01-baseline-config.png` | Both adapters healthy, DHCP-assigned (`ipconfig /all`) |
| 02 | `02-baseline-connectivity.png` | Loopback, gateway, internet, and DNS tests all passing |
| 03 | `03-misconfigured-static-ip.png` | Test-NIC manually set to a static IP outside the valid subnet |
| 04 | `04-failed-gateway-ping.png` | Ping to the expected gateway fails — 100% loss |
| 05 | `05-diagnosis-static-ip-found.png` | IPv4 Properties dialog confirming the misconfigured static IP |
| 06 | `06-dhcp-fix-applied.png` | Release/renew on Test-NIC — valid DHCP-assigned IP restored |

*A final post-fix connectivity re-test (ping to the restored gateway) was not captured as a separate screenshot; screenshot 06 confirms the adapter received a valid DHCP lease and gateway, but a dedicated ping-based verification screenshot is not pictured.*

## Skills Demonstrated

* AWS EC2 network interface configuration
* Windows network troubleshooting
* DHCP/IP fault diagnosis and resolution
* Connectivity testing
* Command-line and GUI-based network administration
* Help-desk incident documentation

## Result

The misconfigured static IP on the secondary adapter was identified and corrected by restoring automatic DHCP configuration. The adapter received a valid IP address, subnet mask, and gateway from the DHCP server, matching its original baseline configuration.

## Lessons Learned

Manually reviewing an adapter's IPv4 properties directly confirmed the root cause faster than relying on `ipconfig` output alone. Isolating the test scenario onto a secondary network interface allowed the fault to be safely introduced and resolved without risking the active management connection to the instance — a practical example of why production troubleshooting is often done on non-critical interfaces first.
