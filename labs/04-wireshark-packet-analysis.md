# Lab 04: Wireshark Packet Analysis

**Status:** Completed milestones documented from the lab owner's account.  
**Environment:** Omarchy / Arch Linux, Wireshark Qt, and wireless interface `wlp1s0`.  
**Goal:** Capture and identify ICMP, DNS, TCP, and TLS traffic, then explain what encryption protects and what metadata remains visible.

## Completed outcomes

| Area | Reported outcome |
| --- | --- |
| Setup | Updated Omarchy, installed Wireshark Qt, and configured capture access through the `wireshark` group. |
| Capture | Captured traffic on `wlp1s0` while generating ping, DNS, and HTTPS requests. |
| ICMP | Identified Echo Request type 8 and Echo Reply type 0. |
| DNS | Observed a router DNS response containing two A records (IPv4 addresses). |
| TCP | Identified the SYN, SYN-ACK, ACK connection handshake and an HTTPS server port of 443. |
| TLS | Observed TLS 1.3 Client Hello, Server Hello, and encrypted Application Data records. |
| Privacy | Distinguished encrypted application content from visible connection metadata. |

These findings summarize the owner's reported work. Commands are reproducible references, not raw transcripts or claims of a new capture performed for this documentation. Raw captures and exact private endpoint addresses are not included.

## 1. Omarchy / Arch Linux setup

Update Omarchy before installing the package, avoiding a partial Arch upgrade:

```bash
omarchy-update
sudo pacman -S wireshark-qt
sudo usermod -aG wireshark "$USER"
```

Log out completely and log back in so the new group membership takes effect. Verify membership and launch the application as the normal user:

```bash
groups
wireshark
```

The Arch package uses the `wireshark` group to permit packet capture through `dumpcap`; the graphical application does not need to run as root. Group membership grants capture privileges and should be limited to trusted users. See the [ArchWiki Wireshark setup](https://wiki.archlinux.org/title/Wireshark) and [Wireshark capture privileges](https://wiki.wireshark.org/CaptureSetup/CapturePrivileges).

## 2. Capture and generate traffic

Select **wlp1s0** in Wireshark and start capturing before running the commands below. This was the lab's active wireless interface; another host may use a different name. Capture only on networks and devices you are authorized to inspect.

```bash
ping -c 4 1.1.1.1
resolvectl query example.com
curl -I https://example.com
```

Stop the capture after generating the traffic. `ping` sends four ICMP Echo Requests; `resolvectl` requests name resolution through systemd-resolved; `curl -I` requests HTTP response headers over HTTPS. Background applications can contribute unrelated packets, so correlate endpoints, timing, and streams with each command.

A repeat DNS query may be answered from cache without a new DNS exchange on `wlp1s0`. Resolver configuration, returned addresses, TLS negotiation, and packet ordering can differ on a later run.

## 3. Display filters

Enter these expressions in Wireshark's **display filter** bar after capture. They select already captured packets; they do not restrict what the capture collects or sanitize a saved capture.

| Filter | Purpose |
| --- | --- |
| `icmp` | Show IPv4 ICMP, including ping requests and replies. |
| `dns` | Show decoded DNS queries and responses. |
| `tcp` | Show TCP traffic. |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Show initial SYN packets without ACK set. |
| `tcp.flags.syn == 1 && tcp.flags.ack == 1` | Show SYN-ACK responses. |
| `tcp.stream == 3` | Isolate both directions of TCP stream 3 in this capture. |

The stream index is assigned within a capture: `3` is not a port number or a universal HTTPS identifier. Select a relevant packet and use **Follow > TCP Stream**, or inspect its TCP stream index, to find the correct value in another capture. Following an encrypted TCP stream does not decrypt TLS.

## 4. ICMP: request and reply

With `icmp` applied, expand **Internet Control Message Protocol** in the packet details:

- **Type 8, code 0:** Echo Request sent toward `1.1.1.1`.
- **Type 0, code 0:** Echo Reply returned to the client.

Use the identifier and sequence number to relate requests to replies. The type values here apply to ICMP for IPv4; ICMPv6 uses different values. Successful replies demonstrate ICMP reachability at the time of the test, not that every application service is available.

## 5. DNS: a response from the router

With `dns` applied, inspect the query name, transaction ID, response flag, and Answers section. The reported response came from the router acting as the client's DNS resolver and contained **two A records for `example.com`**, each holding an IPv4 address.

Multiple A records allow a name to map to multiple IPv4 endpoints; the answer does not mean the client connected to both. The router's response also does not establish that it is authoritative for the domain: it may cache or forward queries. The router address and exact returned addresses are omitted, and later DNS answers may differ.

## 6. TCP: establishing the HTTPS connection

Use `tcp` and the SYN filters to locate the connection, then isolate its stream:

1. **SYN:** The client initiates a connection to the server.
2. **SYN-ACK:** The server acknowledges the client's SYN and sends its own SYN.
3. **ACK:** The client acknowledges the server's SYN, completing the three-way handshake.

The HTTPS connection used server TCP port **443**, with an ephemeral port on the client. TCP establishes an ordered, reliable byte stream; it does not encrypt that stream. Port 443 is conventionally used for HTTPS, but the port alone is not proof of TLS or server identity.

## 7. TLS 1.3: protecting application content

Within the HTTPS TCP stream, the reported capture showed **Client Hello**, **Server Hello**, and **Application Data** records for a TLS 1.3 connection. The hello exchange negotiates security parameters and key establishment; TLS then protects application traffic with encryption and integrity checks. Certificate validation by the client authenticates the server in a normal HTTPS connection.

The terminal can show the response headers from `curl -I` because curl is an endpoint that decrypts its own session. A passive capture without the session secrets does not expose those HTTP headers or other protected HTTP content as plaintext.

In TLS 1.3, encrypted handshake messages after Server Hello also use the outer Application Data record type. That label alone does not prove a record contains HTTP data. Use the negotiated supported-version information rather than a legacy record-version field to identify TLS 1.3. See the [Wireshark TLS guide](https://wiki.wireshark.org/TLS/) and [TLS 1.3 specification](https://www.rfc-editor.org/info/rfc8446/).

## Lessons and next steps

Encryption protects content while leaving useful traffic metadata visible at the capture point:

- Source and destination IP addresses and TCP ports.
- Packet sizes, direction, timing, and connection duration.
- TCP flags and the connection handshake.
- DNS query names and answers when DNS is sent in plaintext.
- Visible TLS hello information; hostname visibility depends on whether Encrypted Client Hello is used.

Seeing source and destination IP addresses does not mean TLS encryption failed. TCP provides transport, while TLS protects the application exchange carried over it. Encryption does not make traffic anonymous, and observable metadata can still reveal communication patterns.

Future practice: repeat the capture, find the new stream index, compare DNS answers, and explain the same protocol sequence without relying on fixed packet numbers. These are next steps, not additional completed results.

## Privacy

No raw `.pcapng` or `.pcap` files are included. This writeup omits private host/router addresses, MAC addresses, identifying device names, credentials, and TLS session secrets. `wlp1s0`, the public test target `1.1.1.1`, and `example.com` are retained to make the workflow clear. Display filtering does not remove unrelated or sensitive packets from a capture file; keep captures local and review any future shared material separately.

[Back to lab index](../README.md)
