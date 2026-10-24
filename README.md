# Linux & Networking Labs

Hands-on Linux administration and networking practice, documented with a focus on troubleshooting, secure remote access, and clear technical communication.

## Completed labs

| Lab | Scope | Status |
| --- | --- | --- |
| [Omarchy Linux: installation and remote administration](labs/01-omarchy-remote-access.md) | Installation, OpenSSH, SSH troubleshooting, UFW, host key verification, and Windows-to-Linux access | Completed |
| [Linux SSH hardening](labs/02-linux-ssh-hardening.md) | System baseline, Ed25519 key authentication, SSH hardening, configuration validation, and subnet-scoped UFW access | Completed |
| [Network discovery & diagnostics](labs/03-network-discovery-diagnostics.md) | Addressing, routing, ping, ARP, Nmap host discovery, reverse lookup, sockets, and UFW review | Completed; follow-ups open |

The first lab established a working Linux environment and successful remote shell access from Windows. It demonstrates practical familiarity with service management, network connectivity checks, firewall rules, and SSH server identity verification.

Lab 02 built on that access with a system baseline, dedicated Ed25519 authentication, verified SSH hardening, and a revised firewall source scope. An unexplained port 53317 firewall allowance remains an unresolved follow-up documented in the lab.

Lab 03 verified local and Internet connectivity, discovered 8 responding LAN hosts, explored reverse lookup, and related application connections and SSH listeners to UFW policy. Unknown host identification and the port 53317 security review remain open.

## Skills demonstrated

- Linux installation and initial system setup
- OpenSSH service administration
- Layered troubleshooting across connectivity, services, firewall policy, and authentication
- System baseline and listening-socket inspection
- Ed25519 key authentication and SSH access hardening
- SSH configuration backups, drop-ins, syntax checks, and reload validation
- Positive key-login and negative password-only authentication tests
- UFW firewall configuration and subnet-scoped SSH access
- SSH host key verification and cross-platform remote access
- IPv4 addressing, DHCP routes, and layered ping diagnostics
- ARP neighbor-state inspection, Nmap host discovery, and reverse lookup
- Active connection and process inspection with ss
- Privacy-conscious technical documentation

## Planned future labs

These are learning objectives, not completed projects or claims of experience.

- Investigate the unresolved port 53317 TCP/UDP firewall allowances recorded in Labs 02 and 03
- Validate SSH and firewall persistence after reboot
- Linux users, groups, permissions, and least privilege
- Identify remaining unknown hosts discovered in Lab 03 and continue DNS/routing diagnostics
- Service monitoring and log analysis
- Backup and recovery exercises
- Repeatable administration with shell scripts

## Documentation approach

Completed milestones reflect the lab owner's reported work. The lab separates those outcomes from illustrative commands and future work. Raw terminal transcripts, screenshots, and exact configuration exports are not included; the commands are reference examples, not execution evidence.

Credentials, keys, MAC addresses, and identifying device names are omitted. Labs 01 and 02 use sanitized placeholders and documentation addresses; Lab 03 retains the private IPv4 addresses and firewall scopes requested for its diagnostic record. Replace reference values as appropriate when repeating commands on another network.
