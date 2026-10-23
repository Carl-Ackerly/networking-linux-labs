# Lab 02: Linux SSH Hardening

**Status:** Completed milestones documented from the lab owner's account; port 53317 firewall allowances remain an unresolved follow-up.  
**Environment:** Omarchy 4.0.6, kernel `7.2.8-9-omarchy`, and a Windows SSH client on a local lab network.  
**Goal:** Establish a system baseline, verify key-based remote access, harden SSH authentication, and restrict SSH reachability to the home subnet.

## Completed outcomes

| Area | Reported outcome |
| --- | --- |
| System baseline | Checked OS, kernel, uptime, and load before changing access settings. |
| OpenSSH | Confirmed systemd `sshd` enabled and running, with port 22 TCP listeners on IPv4 and IPv6. |
| Firewall baseline | Confirmed UFW active, default deny incoming / allow outgoing, and low logging. |
| Authentication | Generated a dedicated Ed25519 key on Windows, installed its public key in the Omarchy user's `authorized_keys`, and verified key-only SSH access. |
| Hardening | Backed up the SSH configuration and applied a drop-in disabling password, root, and keyboard-interactive login while enabling public-key authentication. |
| Validation | Checked syntax, reloaded the service, verified effective settings, opened a fresh key-authenticated session, and confirmed password-only login was denied. |
| SSH source scope | Replaced the single Windows source-IP allowance with the home /24 subnet allowance, removed the redundant old rule, and verified a new SSH connection. |

This record distinguishes reported completed work from illustrative commands. Exact uptime and load values were not retained here; no baseline numbers, terminal output, or reboot-persistence result are fabricated.

## 1. System and service baseline

The baseline identified Omarchy 4.0.6 and kernel `7.2.8-9-omarchy`. Uptime and load were inspected to establish the host's operating state before hardening.

Illustrative Linux checks:

```bash
cat /etc/os-release
uname -r
uptime
systemctl is-enabled sshd
systemctl is-active sshd
sudo ss -ltnp
sudo ufw status verbose
```

The reported service state was enabled and running. SSH listened on TCP port 22 on both IPv4 and IPv6. Listening sockets describe service availability; firewall rules separately control which traffic can reach them.

UFW was active with incoming traffic denied by default, outgoing traffic allowed by default, and logging set to low. Explicit allow rules, including the unresolved rules discussed below, are exceptions to that default incoming policy.

## 2. Dedicated Ed25519 authentication

A dedicated SSH Ed25519 key pair was generated on Windows. Only its public key was added to the intended Omarchy account's `~/.ssh/authorized_keys`. The private key remained on Windows. A key-only connection succeeded before password access was disabled.

Illustrative Windows PowerShell commands; replace placeholders locally:

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\<DEDICATED_KEY>"
ssh -i "$env:USERPROFILE\.ssh\<DEDICATED_KEY>" -o IdentitiesOnly=yes -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no '<LAB_USER>@<LINUX_HOST>'
```

These examples do not reproduce the actual key path, account, host, key material, or key-generation prompts. Server host-key verification remains a separate check from user-key authentication.

## 3. SSH configuration backup and hardening

The SSH configuration was backed up before editing. A drop-in was created at `/etc/ssh/sshd_config.d/00-lab02-hardening.conf` with these settings:

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
KbdInteractiveAuthentication no
```

The effective configuration was verified as:

| Setting | Effective value |
| --- | --- |
| PubkeyAuthentication | yes |
| PasswordAuthentication | no |
| PermitRootLogin | no |
| KbdInteractiveAuthentication | no |

Illustrative validation and reload commands:

```bash
sudo sshd -t
sudo systemctl reload sshd
sudo sshd -T
```

The syntax check passed and the service was reloaded. Effective settings were checked rather than assuming a file's contents alone established the active policy. An existing administrative session should be kept available while validating a fresh connection.

## 4. Fresh-session authentication tests

A fresh Windows SSH session using the dedicated key succeeded after the reload. A separate password-only login attempt was denied.

Illustrative negative test:

```powershell
ssh -o PubkeyAuthentication=no -o PreferredAuthentications=password -o KbdInteractiveAuthentication=no '<LAB_USER>@<LINUX_HOST>'
```

The positive and negative tests checked that the intended key still worked and that password-only access was unavailable. No sensitive authentication logs or identifying failure output are published.

## 5. UFW SSH source-scope update

The original SSH allowance was restricted to a single Windows source IP. It was replaced with an allowance for the home /24 subnet. The new rule was added, the redundant old single-source rule was removed, and a new SSH connection succeeded afterward.

Illustrative rule for an example subnet only:

```bash
sudo ufw allow from 192.0.2.0/24 to any port 22 proto tcp
sudo ufw status numbered
```

`192.0.2.0/24` is a documentation example, not the actual home subnet. Rule numbers must be inspected locally before removing an old rule because numbering can change; the actual rule number and source address are omitted.

**Security tradeoff:** All devices on the allowed local subnet can reach SSH, but a valid authorized key is still required to authenticate. This broadens network reachability compared with a single-client allowance and accommodates client address changes within that subnet.

**Limitation:** If the home subnet changes, the allowance must be reviewed and updated. The /24 example describes IPv4 scope; IPv6 listeners do not establish that an equivalent subnet-based IPv6 SSH allowance exists.

## 6. Unresolved port 53317 firewall allowances

Inspection found UFW allowances for port **53317 TCP and UDP from anywhere on both IPv4 and IPv6**. No corresponding service listener was found at inspection time.

These allowances remain an **unresolved follow-up**. They were not reported removed, and their purpose was not established. The absence of a listener during one inspection does not explain the rules or prove that a service cannot listen later.

Future work is to identify the application or workflow that created the rules, determine whether they are still required, and then review their scope or removal with connectivity checks.

Illustrative follow-up checks:

```bash
sudo ufw status numbered
sudo ss -lntup
```

## Lessons and next steps

The completed workflow separated system baseline, service state, listening sockets, firewall reachability, and authentication policy. It used configuration backup, syntax validation, effective-setting inspection, and fresh-session tests to reduce the risk of losing remote access during hardening.

Next steps include investigating port 53317, testing persistence after reboot, and continuing users, groups, permissions, and least-privilege exercises. These remain future work.

## Privacy

This record omits real private/internal addresses, usernames, device identifiers, SSH keys, passwords, host-key fingerprints, and sensitive logs. Examples use placeholders or a documentation-only subnet. Commands are reference examples rather than terminal transcripts.
