# Traffic Capture Scenarios

Three scenarios were captured and analysed in Wireshark during Phase 6.
Pcap files are not included in this repository.

## Scenario 1 — Normal Traffic (eth1)
Captured on the WAN interface during idle and light activity.
Protocols observed: WireGuard (encrypted transport), ARP, NTP, SSDP, STP (virtual bridge noise).
Key insight: All SSH management traffic appears as opaque WireGuard encrypted packets on eth1.

## Scenario 2 — Port Scan (eth1)
Captured during an Nmap SYN scan of ports 1-1000 against OpenWRT.
Filter used: `tcp.flags.syn == 1 && tcp.flags.ack == 1`
Result: 500+ packets reduced to 2 SYN-ACK responses (ports 22 and 443).
Key insight: SYN scan pattern — rapid SYN probes, RST responses from closed ports, no full handshake completed.

## Scenario 3 — Plaintext HTTP (eth1)
Captured during an HTTP request from OpenWRT to a Python HTTP server on the host.
Filter used: `http`
Result: Full GET request and HTTP 200 response visible including headers, server software version, and complete HTML body.
Key insight: TCP Stream Follow reconstructed the entire conversation — URL, User-Agent, server version, and content all readable in plaintext.
