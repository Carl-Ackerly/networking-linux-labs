# Lab 03: Network Discovery & Diagnostics

**Status:** Completed milestones documented from the lab owner's account; unknown host identification and the port 53317 security review remain unresolved.  
**Environment:** Omarchy Linux and a Windows desktop on the home network `192.168.1.0/24`.  
**Goal:** Inspect addressing and routing, test connectivity and DNS, discover local hosts, and relate active connections and listening sockets to firewall policy.

## Completed outcomes

| Area | Reported outcome |
| --- | --- |
| Addressing | Confirmed the Linux IPv4 address `192.168.1.12/24`. |
| Routing | Confirmed a DHCP-provided default route through `192.168.1.1`, with source `192.168.1.12`. |
| Connectivity | Successfully pinged the router, `1.1.1.1`, and `google.com`. |
| Neighbor discovery | Observed the Windows desktop at `192.168.1.91` change from ARP neighbor state `STALE` to `REACHABLE`. |
| Host discovery | Installed Nmap and discovered 8 hosts on the home /24 subnet. |
| Name lookup | Practiced reverse lookup while investigating discovered addresses; not all host identities were established. |
| Connections | Identified an established Chromium connection from local TCP port `51914` to peer port `443`. |
| Listeners and firewall | Observed IPv4/IPv6 SSH listeners and reviewed active UFW policy, Docker DNS exceptions, and unexplained port 53317 allowances. |

Results below summarize the owner's completed work. Command blocks are reference commands, not raw terminal transcripts. Exact interface names, MAC addresses, external peer addresses, DNS names returned by reverse lookup, latency values, and the full host list are not reproduced.

## 1. Interface addressing and default route

```bash
ip -br addr
ip route
ip -4 route show default
```

The brief address view showed `192.168.1.12/24` on the Linux host. The route inspection identified the default gateway as `192.168.1.1`, with DHCP route provenance (`proto dhcp`) and source address `192.168.1.12` (`src`). The /24 places the host on `192.168.1.0/24`; destinations outside directly connected routes use the default gateway.

## 2. Layered connectivity checks

Reference commands; stop each continuous ping with Ctrl+C:

```bash
ping 192.168.1.1
ping 1.1.1.1
ping google.com
```

All three checks succeeded. The router response confirmed local gateway reachability. The public IP response confirmed connectivity beyond the LAN without requiring name resolution. The hostname test demonstrated successful name resolution and ICMP reachability to the resolved destination at that time. These checks do not establish that every Internet service or protocol works.

## 3. ARP neighbor state

```bash
ip neigh
ping 192.168.1.91
ip neigh
```

The Windows desktop at `192.168.1.91` was observed first as `STALE` and later as `REACHABLE`. A stale neighbor entry retains a link-layer mapping whose reachability needs reconfirmation; it does not by itself mean the desktop is offline. The later state reflected recent reachability confirmation.

## 4. Nmap host discovery and reverse lookup

Nmap was installed using the Arch package manager:

```bash
sudo pacman -S nmap
sudo nmap -sn 192.168.1.0/24
```

The scan reported **8 hosts up** on the home network. The `-sn` option performs host discovery without a port scan. This is a snapshot of hosts that responded to the discovery methods used, not proof that every device on the subnet was found.

Reverse lookup was also explored to seek names associated with discovered IP addresses. A reference lookup for the known Windows address is:

```bash
getent hosts 192.168.1.91
```

This reference command is not a claim that this exact lookup command or a particular returned name was retained in the session. A missing name does not mean a host is absent, and a returned name alone does not verify device ownership. Some discovered hosts remained unidentified; no complete device inventory is claimed.

## 5. Active connections and process ownership

```bash
ss -tun
sudo ss -tnp state established
```

The connection inspection associated an established TCP connection with Chromium: local port `51914` connected to a peer on port `443`, conventionally used for HTTPS. The local port was an ephemeral client port, not evidence of an inbound server listener. Process attribution helped connect network activity to the application; port 443 alone does not prove the application protocol or the peer's identity.

## 6. Listening sockets and UFW policy

```bash
sudo ss -tulnp
sudo ufw status verbose
```

The socket inspection showed `sshd` listening on TCP port 22 for both IPv4 and IPv6. Listener availability and firewall allowances are separate observations.

UFW was **active**, with logging on at low level, **default deny incoming**, **allow outgoing**, and routed traffic disabled. The reported incoming exceptions were:

| Destination | Protocol | Allowed source | Note |
| --- | --- | --- | --- |
| Port 22 | TCP | `192.168.1.0/24` | SSH home LAN; the SSH allow rule was limited to this IPv4 subnet. |
| `172.17.0.1`, port 53 | UDP | `172.16.0.0/12` | `allow-docker-dns` |
| `172.17.0.1`, port 53 | UDP | `192.168.0.0/16` | `allow-docker-dns` |
| Port 53317 | TCP and UDP | Anywhere, IPv4 and IPv6 | Purpose unresolved. |

The IPv6 SSH listener did not imply that an IPv6 SSH allow rule existed. The Docker DNS rules were observed exceptions; their presence alone does not establish Docker DNS functionality or justify their scope.

## 7. Unresolved port 53317 security review

```bash
sudo ss -tulnp | grep 53317
```

The command produced **no matching output** at inspection time. No corresponding listening/bound socket was observed in that check, while UFW still allowed port 53317 TCP and UDP from anywhere on IPv4 and IPv6.

This remains an **unresolved security follow-up**, continuing the finding from [Lab 02](02-linux-ssh-hardening.md). The application or workflow responsible for the rules was not established, and no removal or restriction is claimed. A snapshot without a matching socket does not establish that an application cannot bind the port later or that the allowances are unnecessary.

Future work is to identify the rules' owner and purpose, determine whether they are needed, and review their scope or removal with connectivity checks.

## Lessons and next steps

The lab connected interface configuration, routing, DNS, ARP, host discovery, process-owned connections, listeners, and firewall policy into a layered diagnostic workflow.

Completed diagnostics do not close the remaining work: identify unknown discovered hosts, review port 53317, and validate SSH/firewall persistence after reboot. Repeat discovery results should be treated as time-specific observations.

## Privacy

This lab retains the private IPv4 addresses and firewall scopes requested for the diagnostic record. It omits MAC addresses, device-specific names, usernames, keys, credentials, and external peer addresses. Earlier labs retain their sanitized examples.

[Back to lab index](../README.md)
