# Support Ticket

**Ticket ID:** HD-2026-0142
**Date Opened:** September 24, 2026
**Priority:** Medium
**Category:** Network / DHCP
**Reported By:** End User
**Assigned To:** IT Support Technician

## User Report

> "The secondary network adapter on the Windows server isn't getting an IP address from DHCP. It looks like it might have been manually configured with a static address at some point. Can you check it and get it back on DHCP?"

## Environment

* Server: `EC2AMAZ-VQU1ICD` (AWS EC2, Windows Server, `us-east-2`)
* Affected adapter: secondary network interface (`Test-NIC`)
* Primary adapter (RDP/management connection): unaffected, left untouched throughout

## Diagnostic Steps

1. Reviewed baseline configuration on both adapters using `ipconfig /all` — both were healthy and DHCP-assigned. *(Screenshot 01)*
2. Confirmed baseline connectivity (loopback, gateway, internet, DNS) on the working configuration. *(Screenshot 02)*
3. Re-ran `ipconfig /all` on the affected adapter and found `DHCP Enabled: No`, with a static IP (`10.0.0.30`) outside the instance's actual subnet range. *(Screenshot 03)*
4. Attempted to ping the address the misconfigured adapter believed to be its gateway (`10.0.0.1`) — request timed out, 100% packet loss, confirming the connectivity failure. *(Screenshot 04)*
5. Opened the adapter's IPv4 Properties and confirmed "Use the following IP address" was selected with the incorrect static IP and gateway manually entered. *(Screenshot 05)*

## Root Cause

The secondary network adapter had been manually configured with a static IPv4 address and gateway that did not belong to the instance's actual VPC subnet, preventing it from communicating with the real gateway or any other host on the network.

## Resolution

1. In the adapter's IPv4 Properties, changed the setting back to "Obtain an IP address automatically."
2. Ran `ipconfig /release "Test-NIC"` followed by `ipconfig /renew "Test-NIC"`.
3. The adapter obtained a valid DHCP-assigned IPv4 address, subnet mask, and gateway, matching its original baseline configuration. *(Screenshot 06)*

## Verification

`ipconfig /all` after the fix confirms the adapter is back on DHCP with a valid lease and gateway. *(Screenshot 06)*

> Note: a dedicated post-fix ping test to the restored gateway was not captured as a separate screenshot. The DHCP lease and gateway shown in Screenshot 06 confirm the configuration was restored, but connectivity was not re-verified with a fresh ping screenshot at time of writing.

## Technician Notes

The primary network adapter carrying the active Remote Desktop session was never modified. The fault was isolated to a secondary ENI attached to the instance specifically for this purpose, avoiding any risk of losing remote access while reproducing and resolving the issue.

## Recommended Follow-Up

* Capture a final ping-based connectivity screenshot to fully close out verification.
* Periodically audit adapters for manually-set static configurations that may have been left over from prior troubleshooting or testing.

## Ticket Status

**Resolved** — pending final connectivity screenshot for complete verification.
