# Lab 01: Omarchy Linux Installation & Remote Access

**Status:** Completed milestones documented from the lab owner's account.  
**Environment:** Omarchy Linux host and Windows SSH client on a local lab network.  
**Goal:** Establish remote administration from Windows and understand the services, firewall policy, and identity checks that support SSH access.

## Completed outcomes

| Area | Reported outcome |
| --- | --- |
| Installation | Installed Omarchy Linux and reached a usable Linux environment. |
| OpenSSH | Set up the SSH server for remote access. |
| Troubleshooting | Worked through SSH connection issues and achieved connectivity. |
| Firewall | Configured UFW in support of SSH access. |
| Server identity | Completed SSH host key verification. |
| Remote access | Successfully connected from Windows to a Linux shell over SSH. |

Exact installer selections, error messages, firewall rule scope, authentication method, and command history were not retained in this public record. No particular root cause, hardening setting, or reboot-persistence test is claimed.

## 1. Installation and initial setup

The completed installation provided the Linux environment for the remaining exercises. This portfolio entry focuses on administration and connectivity rather than reconstructing undocumented installer steps.

## 2. OpenSSH setup

OpenSSH provides the remote shell service. Service state, configuration validity, and listening sockets are separate checks: an installed package alone does not establish that a remote client can connect.

Illustrative Linux checks:

```bash
sudo systemctl status sshd
sudo sshd -t
sudo ss -ltnp
```

If the server needs to be started and enabled, the reference command is:

```bash
sudo systemctl enable --now sshd
```

These commands illustrate how to inspect and manage the service; their output is intentionally omitted. See the [ArchWiki OpenSSH documentation](https://wiki.archlinux.org/title/OpenSSH).

## 3. SSH troubleshooting

The lab reached successful SSH access after troubleshooting. The table below is a reusable diagnostic guide, not a list of errors observed in this lab.

| Symptom | Next check | Interpretation |
| --- | --- | --- |
| Connection times out | Destination, route, firewall policy, and host availability | The connection may be blocked or the host may be unreachable. |
| Connection refused | SSH service and listening port | A rejection can indicate no listener or an explicit reject rule. |
| Permission denied | Intended account and authentication configuration | Transport has progressed to authentication; check permitted credentials and access policy. |
| Host identification warning | Server identity through a trusted independent channel | Stop and investigate before changing saved trust information. |

Illustrative Windows PowerShell diagnostics; replace the placeholders locally:

```powershell
Test-NetConnection -ComputerName '<LINUX_HOST>' -Port 22
ssh -v '<LAB_USER>@<LINUX_HOST>'
```

Port 22 is an example default, not a claim about the recorded configuration. Verbose SSH output can contain identifying details and should be reviewed before sharing.

## 4. UFW firewall configuration

UFW configuration was completed as part of enabling remote access. The precise rules are not reproduced here. For repeatable setup, confirm the actual SSH port and review existing rules before applying a policy. Keep local console access available while changing firewall rules.

Reference sequence for a fresh lab using the default SSH port, not a transcript of the completed configuration:

```bash
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```

Add the SSH allowance before enabling the firewall. A port-only allowance permits any source that can reach the host; restricting access to an approved management source is a separate design choice. Confirm IPv4 and IPv6 policy as applicable, then test a new SSH session. See the [ArchWiki UFW documentation](https://wiki.archlinux.org/title/Uncomplicated_Firewall).

## 5. SSH host key verification

Host key verification was completed before trusting the remote endpoint. A host key identifies the server; it is distinct from a user's authentication key.

For a repeatable verification process, obtain the server's public host key fingerprint directly at its console or through another trusted independent channel, then compare both the key type and fingerprint with the Windows SSH prompt. Accept the prompt only after they match.

Illustrative Linux command for an Ed25519 host key, if present:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

The real fingerprint and prompt are omitted. If a saved key changes, investigate whether the host was rebuilt or its keys legitimately rotated before updating the saved entry. Do not disable host key checking to bypass a warning. See the [OpenSSH ssh-keygen manual](https://man.openbsd.org/ssh-keygen).

## 6. Windows-to-Linux remote access

The final reported result was an authenticated Linux shell reached from Windows using SSH. This demonstrated a working path through client connectivity, the Linux SSH service, firewall policy, server identity checks, and authentication.

Illustrative connection command:

```powershell
ssh '<LAB_USER>@<LINUX_HOST>'
```

For a future repeatable validation, run a harmless command such as `pwd` in the remote shell and use `exit` to disconnect. No fabricated terminal output is included as evidence of the completed session.

## Lessons and next steps

The workflow separates network reachability, service availability, firewall policy, server identity, and authentication so each can be investigated independently.

Future work includes key-based authentication, least-privilege access, persistence checks after reboot, and sanitized validation records. These remain planned and are not part of the completed claims above.

## Privacy

This record omits device identifiers, real addresses, account names, host fingerprints, credentials, and secrets. Placeholder examples describe the process without publishing the lab's environment details.
