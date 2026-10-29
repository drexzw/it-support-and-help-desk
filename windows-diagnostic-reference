# Windows Diagnostics Reference

A quick reference for common Windows network and system diagnostic commands. Each entry covers what the command is for, how to use it, what healthy output looks like, what unusual output might mean, and what to run next.

Suggested troubleshooting order for a "can't get online" ticket:

`ipconfig /all` → `ping <gateway-ip>` → `tracert 8.8.8.8` → `nslookup google.com` → `Test-NetConnection google.com`

`sfc /scannow` is separate: use it when system files may be corrupted.

---

## 1. `ipconfig /all`

**Purpose**
Displays the full network configuration of every physical and virtual network adapter on the computer, including IPv4 and IPv6 addresses, subnet mask, default gateway, DNS servers, DHCP details, and MAC address.

**Example usage**
```
ipconfig /all
```

**What normal results look like**
- The active adapter has a valid IPv4 address in the expected range for the network (for example, `192.168.1.x`)
- Subnet mask and Default Gateway are filled in
- DHCP Enabled shows `Yes` (on most networks), with a lease obtained and lease expiry listed
- DNS Servers are listed (router IP or a public DNS server)

**What an unusual result might mean**
- IPv4 address starting with `169.254.x.x` (APIPA): the computer could not get an address from DHCP
- Empty Default Gateway: no route out of the local network
- Media disconnected: cable unplugged, Wi-Fi off, or adapter problem
- Wrong or missing DNS servers: websites may fail to load by name even though the connection is up
- Address in an unexpected range: possible wrong network, VPN adapter, or static IP misconfiguration

**Next diagnostic step**
If the address and gateway look correct, run `ping <gateway-ip>` using the Default Gateway value. If the address is `169.254.x.x` or blank, check the cable/Wi-Fi, then try `ipconfig /release` followed by `ipconfig /renew`.

---

## 2. `ping <gateway-ip>`

**Purpose**
Tests whether the computer can reach another device over the network. Pinging the default gateway (usually the router) confirms the local connection works before looking further out.

**Example usage**
```
ping 192.168.1.1
```

**What normal results look like**
- Four `Reply from ...` lines
- `Lost = 0 (0% loss)`
- Low response times (typically a few milliseconds on a wired or good Wi-Fi connection)

**What an unusual result might mean**
- `Request timed out`: gateway is down, unreachable, or blocking ping, or there is a cable/Wi-Fi problem
- `Destination host unreachable`: the computer has no valid path to that address (wrong IP, wrong subnet, or no connection)
- Some packets lost or high/erratic times: congestion, weak Wi-Fi signal, interference, or a faulty cable

**Next diagnostic step**
- If the ping fails: recheck the adapter settings in `ipconfig /all`, reseat the cable or reconnect Wi-Fi, and check that the router is powered on
- If the ping succeeds: the local network is fine, so move on to `tracert 8.8.8.8` to test beyond the router

---

## 3. `tracert 8.8.8.8`

**Purpose**
Shows the path packets take from the computer to a destination, listing each router (hop) along the way and the response time at each. Using `8.8.8.8` (Google's public DNS) tests internet reachability by IP address, without involving DNS.

**Example usage**
```
tracert 8.8.8.8
```

**What normal results look like**
- Hop 1 is the local gateway
- Following hops are the ISP's routers, with response times that generally rise gradually
- The trace completes at `8.8.8.8` with `Trace complete.`

**What an unusual result might mean**
- Stops after hop 1 (timeouts after that): the problem is between the router and the ISP, or the router has no internet
- `* * *` at a single middle hop with later hops responding normally: usually just a router that doesn't answer trace requests, not a real problem
- A big latency jump at one hop that stays high for all later hops: congestion or a problem at that point in the path
- The same hops repeating: possible routing loop

**Next diagnostic step**
- If the trace reaches `8.8.8.8`: internet access by IP works, so if websites still fail, test DNS with `nslookup google.com`
- If it fails at hop 1: focus on the router and its connection
- If it fails at a later hop: the issue is likely with the ISP, so note the last responding hop for the escalation ticket

---

## 4. `nslookup google.com`

**Purpose**
Queries a DNS server to translate a domain name into an IP address. Used to check whether name resolution is working.

**Example usage**
```
nslookup google.com
```

**What normal results look like**
- Shows the DNS server being used (name and address)
- Returns one or more IP addresses for `google.com`, often under `Non-authoritative answer`

**What an unusual result might mean**
- `DNS request timed out`: the configured DNS server is unreachable or not responding
- `Non-existent domain`: the name could not be resolved (typo, or a DNS problem)
- Server shows `UnKnown`: the DNS server has no reverse-lookup name. This alone is usually harmless if an address is still returned
- Returned IP looks wrong or unexpected: possible DNS misconfiguration or poisoned/cached bad record

**Next diagnostic step**
Try a different DNS server directly:
```
nslookup google.com 8.8.8.8
```
If that works but the default lookup fails, the problem is the DNS server the computer is configured to use. Check the DNS settings in `ipconfig /all`, and try `ipconfig /flushdns`.

---

## 5. `Test-NetConnection google.com`

**Purpose**
A PowerShell cmdlet that tests connectivity to a host. By default it resolves the name and sends a ping; with the `-Port` parameter it also tests whether a specific TCP port is open, which makes it useful for checking services like web (443) or RDP (3389).

**Example usage**
```powershell
Test-NetConnection google.com
Test-NetConnection google.com -Port 443
Test-NetConnection google.com -TraceRoute
```

**What normal results look like**
- `RemoteAddress` shows the resolved IP address
- `PingSucceeded : True`
- With `-Port`, `TcpTestSucceeded : True`

**What an unusual result might mean**
- Name resolution warning or no `RemoteAddress`: DNS problem
- `PingSucceeded : False`: the host may be down, unreachable, or blocking ICMP (some hosts do by design)
- `TcpTestSucceeded : False`: the port is blocked by a firewall, the service is not running, or there is no route to the host

**Next diagnostic step**
- If name resolution failed: go back to `nslookup google.com`
- If ping failed but the port test works: ICMP is likely blocked, which may be fine
- If the port test fails: check the local firewall and network firewall rules, and confirm the service is running on the target. Use `-TraceRoute` to see where the path breaks.

---

## 6. `sfc /scannow`

**Purpose**
System File Checker. Scans protected Windows system files and replaces corrupted or modified ones with a cached good copy. Use it when Windows shows odd behavior, crashes, or missing/damaged system components. It is not a network command.

**Example usage**
Run from an elevated (Administrator) Command Prompt or PowerShell:
```
sfc /scannow
```

**What normal results look like**
- `Windows Resource Protection did not find any integrity violations.`

**What an unusual result might mean**
- `Windows Resource Protection found corrupt files and successfully repaired them.`: the problem was fixed. Restart and retest
- `Windows Resource Protection found corrupt files but was unable to fix some of them.`: the cached copy may be damaged too
- `Windows Resource Protection could not perform the requested operation.`: not run as administrator, or a system issue is blocking the scan

**Next diagnostic step**
- If repairs were made: restart and check whether the original issue is gone
- If some files could not be fixed: run `DISM /Online /Cleanup-Image /RestoreHealth`, then run `sfc /scannow` again. Details are in `C:\Windows\Logs\CBS\CBS.log`
- If the scan could not run: reopen the terminal as Administrator, or try again in Safe Mode
