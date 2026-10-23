# Linux & Networking Labs

Hands-on Linux administration and networking practice, documented with a focus on troubleshooting, secure remote access, and clear technical communication.

## Completed labs

| Lab | Scope | Status |
| --- | --- | --- |
| [Omarchy Linux: installation and remote administration](labs/01-omarchy-remote-access.md) | Installation, OpenSSH, SSH troubleshooting, UFW, host key verification, and Windows-to-Linux access | Completed |
| [Linux SSH hardening](labs/02-linux-ssh-hardening.md) | System baseline, Ed25519 key authentication, SSH hardening, configuration validation, and subnet-scoped UFW access | Completed |

The first lab established a working Linux environment and successful remote shell access from Windows. It demonstrates practical familiarity with service management, network connectivity checks, firewall rules, and SSH server identity verification.

Lab 02 built on that access with a system baseline, dedicated Ed25519 authentication, verified SSH hardening, and a revised firewall source scope. An unexplained port 53317 firewall allowance remains an unresolved follow-up documented in the lab.

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
- Privacy-conscious technical documentation

## Planned future labs

These are learning objectives, not completed projects or claims of experience.

- Investigate the unresolved port 53317 TCP/UDP firewall allowances recorded in Lab 02
- Validate SSH and firewall persistence after reboot
- Linux users, groups, permissions, and least privilege
- DNS, routing, and network diagnostics
- Service monitoring and log analysis
- Backup and recovery exercises
- Repeatable administration with shell scripts

## Documentation approach

Completed milestones reflect the lab owner's reported work. The lab separates those outcomes from illustrative commands and future work. Raw terminal transcripts, screenshots, and exact configuration exports are not included; the commands are reference examples, not execution evidence.

Environment identifiers and credentials are omitted. Examples use placeholders that must be replaced locally and must never be committed with real values.
