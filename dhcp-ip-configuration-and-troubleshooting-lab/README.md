# DHCP / IP Configuration & Troubleshooting Lab

## Overview

This lab demonstrates basic Windows IPv4 configuration and DHCP troubleshooting techniques commonly used in IT support and help-desk environments.

The lab involved inspecting the system's current network configuration, testing network connectivity, releasing and renewing the DHCP lease, understanding static versus dynamic IPv4 configuration, and verifying that DHCP was restored successfully.

## Before / After Summary

| | Before Troubleshooting | After Troubleshooting |
|---|---|---|
| IPv4 Address | `192.168.0.155` | `192.168.0.155` (DHCP-assigned) |
| Default Gateway | `192.168.0.1` | `192.168.0.1` |
| DHCP Status | Enabled | Enabled — lease renewed |
| DNS Servers | `68.105.28.11, 68.105.29.11, 68.105.28.12` | `68.105.28.11, 68.105.29.11, 68.105.28.12` |
| Connectivity (loopback / gateway / internet / DNS) | Not yet tested | All 4 tests **PASS**, 0% packet loss |

> Note: mid-process, the adapter briefly self-assigned an APIPA address (`169.254.177.112`) after `ipconfig /renew` — see [deployment-notes.md](./deployment-notes.md#8-dhcp-configuration-verification) for the full detail on that transient state.

## Documentation

- [commands.md](./commands.md) — full command reference (`ipconfig`, `ping`, `nslookup`, etc.) with explanations
- [deployment-notes.md](./deployment-notes.md) — step-by-step log of the lab, in order, with recorded output at each stage

## Objectives

* Inspect a Windows computer's IPv4 configuration
* Identify the IPv4 address, subnet mask, default gateway, DHCP server, and DNS servers
* Test local and network connectivity using `ping`
* Release and renew a DHCP lease
* Understand the difference between DHCP and static IP addressing
* Identify an APIPA address and understand what it can indicate
* Restore the system to automatic DHCP configuration
* Document troubleshooting steps in a help-desk format

## Environment

* Operating System: Windows
* Network Configuration: IPv4
* Address Assignment: DHCP
* Primary Tools:

  * Command Prompt
  * `ipconfig`
  * `ping`
  * Windows Network Connections
  * IPv4 Properties

## Key Concepts

### DHCP

Dynamic Host Configuration Protocol (DHCP) automatically provides network configuration information to clients.

This can include:

* IPv4 address
* Subnet mask
* Default gateway
* DNS server information
* DHCP lease information

### IPv4 Address

The IPv4 address identifies a device on an IP network.

### Subnet Mask

The subnet mask determines which portion of an IPv4 address represents the network and which portion represents the host.

### Default Gateway

The default gateway is normally the router used by the computer to communicate with devices outside its local network.

### DNS

Domain Name System (DNS) translates human-readable hostnames such as `google.com` into IP addresses.

### APIPA

Windows can automatically assign an address in the `169.254.x.x` range when it cannot obtain a normal IPv4 configuration through DHCP.

A 169.254.x.x address can therefore be an important troubleshooting clue when investigating DHCP problems.

## Troubleshooting Method

The lab used a layered troubleshooting approach:

1. Inspect the IP configuration.
2. Test the local TCP/IP stack.
3. Test the default gateway.
4. Test Internet connectivity using an IP address.
5. Test DNS resolution using a hostname.
6. Release the existing DHCP lease.
7. Renew the DHCP lease.
8. Verify the resulting configuration.
9. Restore automatic DHCP settings.
10. Perform final connectivity tests.

## Screenshots

| #  | Screenshot                       | Description                         |
| -- | --------------------------------- | ------------------------------------ |
| 01 | `01-ipconfig-all.png`            | Initial IPv4 and DHCP configuration |
| 02 | `02-gateway-ping.png`            | Connectivity to the default gateway |
| 03 | `03-internet-dns-test.png`       | Internet and DNS connectivity tests |
| 04 | `04-dhcp-renew.png`              | DHCP lease release and renewal      |
| 05 | `05-apipa-fallback.png`           | DHCP configuration restored         |
| 06 | `06-final-connectivity.png`      | Final connectivity verification     |

## Skills Demonstrated

* Windows network troubleshooting
* IPv4 configuration
* DHCP troubleshooting
* DNS troubleshooting fundamentals
* Connectivity testing
* Command-line troubleshooting
* Help-desk diagnostic reasoning
* Technical documentation

## Result

The system was returned to automatic DHCP configuration and final connectivity testing was performed to verify network functionality.

## Lessons Learned

This lab demonstrated that network troubleshooting should be performed systematically rather than by randomly changing settings.

Testing the loopback address, default gateway, public IP address, and hostname provides useful information about where a connectivity failure may be occurring.
