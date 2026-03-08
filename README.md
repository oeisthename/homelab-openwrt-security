# OpenWRT Network Security Homelab

A virtualized network security lab built on KVM/QEMU, featuring an OpenWRT router VM with WireGuard VPN, firewall hardening, traffic analysis, and Suricata IDS deployment.

## Overview

This project documents the design, deployment, and hardening of a virtualized network security environment. Rather than flashing physical hardware (the target device was assessed as incompatible with OpenWRT), the lab uses KVM virtualization to achieve identical learning outcomes with zero hardware risk.

## Architecture
```
INTERNET → Huawei B636 (ISP router, 192.168.1.1)
         → HP EliteBook 850 G3 (KVM host, Ubuntu)
           virbr0: 192.168.122.1
           └── OpenWRT VM (192.168.122.2)
                 WAN: eth1 = 192.168.122.2
                 LAN: br-lan = 192.168.2.1
                 VPN: wg0 = 10.0.0.1/24

VPN Tunnel (WireGuard):
  Linux Host (10.0.0.2) ←→ OpenWRT (10.0.0.1)
  Encrypted UDP on port 51820
```

## Skills Demonstrated

| Domain | Skills |
|--------|--------|
| Virtualization | KVM/QEMU setup, VM lifecycle, snapshots, persistent definitions |
| Networking | Static IP config, bridge networking, NAT, IPv4/IPv6 |
| VPN | WireGuard deployment, key management, tunnel testing |
| Firewall | Zone-based firewall, rule auditing, attack surface reduction |
| Hardening | SSH key auth, password auth disabled, service restriction by IP |
| Traffic Analysis | tcpdump, Wireshark, pcap analysis, protocol identification |
| Intrusion Detection | Suricata IDS, custom rule writing, alert analysis |
| Linux | UCI, OpenWRT, bash scripting, service management |

## What Was Built

### Phase 1 — Compatibility Assessment
Assessed the TP-Link TD-W8961N Ver:3.0 against the OpenWRT Table of Hardware. Determined the device was incompatible due to a proprietary ADSL chipset. Pivoted to a virtualized architecture — same curriculum, zero brick risk.

### Phase 2 — KVM Lab Deployment
- Verified CPU virtualization support (8 threads, KVM acceleration confirmed)
- Installed and configured KVM/QEMU/libvirt stack
- Downloaded and SHA256-verified OpenWRT 23.05.3 x86-64 image
- Converted raw image to qcow2, resized, deployed as persistent KVM VM
- Resolved 5 distinct deployment issues (bridge permissions, setuid, IP conflicts)

### Phase 3 — Network Configuration
- Configured static WAN IP (192.168.122.2) on virbr0
- Resolved LAN IP conflict with ISP router (changed from 192.168.1.1 to 192.168.2.1)
- Enabled and accessed LuCI web interface over HTTPS

### Phase 4 — Hardening
- Generated ED25519 SSH key pair, deployed to OpenWRT dropbear
- Disabled SSH password authentication and root password login
- Restricted SSH and LuCI access to host IP only (192.168.122.1)
- Disabled HTTP (port 80), HTTPS only
- Audited full firewall ruleset — removed 4 unnecessary rules:
  - Allow-Ping (reveals router to network scanners)
  - Allow-IGMP (multicast management, unused in lab)
  - Allow-IPSec-ESP (replaced by WireGuard)
  - Allow-ISAKMP (IPSec key exchange, unused)
- Final attack surface: 2 open ports from 65,535

### Phase 5 — WireGuard VPN
- Installed WireGuard kernel module and tools on OpenWRT
- Generated Curve25519 key pairs on both endpoints
- Configured wg0 interface (10.0.0.1) on OpenWRT server
- Configured wg-lab interface (10.0.0.2) on Linux client
- Established encrypted tunnel — handshake confirmed, SSH over VPN working
- Added WireGuard firewall zone and UDP 51820 rule

### Phase 6 — Traffic Analysis & IDS
- Installed tcpdump on OpenWRT, captured traffic on multiple interfaces
- Analysed 3 scenarios in Wireshark:
  - Normal traffic: WireGuard, ARP, NTP, SSDP identification
  - Port scan: SYN flood pattern, RST responses, SYN-ACK filter to isolate open ports
  - Plaintext HTTP: full request/response reconstruction via TCP stream follow
- Deployed Suricata 7.0.3 IDS on host, updated Emerging Threats ruleset (48,867 rules)
- Wrote 3 custom detection rules:
  - ICMP ping detection
  - SSH connection monitoring
  - Port scan threshold detection (20 SYN packets / 3 seconds)
- Confirmed alerts firing correctly in fast.log

## Security Controls Implemented

| Control | Implementation |
|---------|---------------|
| SSH authentication | ED25519 keys only, password auth disabled |
| Access restriction | SSH and LuCI restricted to 192.168.122.1 |
| Encrypted management | All admin traffic through WireGuard tunnel |
| HTTP disabled | HTTPS only on port 443 |
| Firewall minimised | 7 rules from original 11, 4 unnecessary rules removed |
| IDS monitoring | Suricata watching virbr0 with custom + ET Open rules |

## Key Troubleshooting Resolved

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| VM creation failed | qemu-bridge-helper missing setuid bit | chmod u+s on helper binary |
| LuCI inaccessible | rfc1918_filter blocking private IPs on WAN | Disabled rfc1918_filter in uhttpd |
| SCP file transfer failed | Dropbear lacks sftp-server subsystem | Used ssh + cat pipe instead |
| Suricata rules not loading | Rules written to wrong directory | Copied to /var/lib/suricata/rules/ |

## Lab Environment

| Component | Details |
|-----------|---------|
| Host | HP EliteBook 850 G3, Ubuntu, 8 CPU threads |
| Hypervisor | KVM/QEMU with libvirt |
| Router OS | OpenWRT 23.05.3 (x86-64) |
| VPN | WireGuard (kernel 5.6+) |
| IDS | Suricata 7.0.3, Emerging Threats Open ruleset |
| Capture tools | tcpdump 4.99.4, Wireshark |

## Repository Structure
```
├── configs/
│   ├── firewall/     # OpenWRT firewall ruleset export
│   └── wireguard/    # Sanitised WireGuard config (no private keys)
├── docs/
│   ├── architecture.md    # Detailed network diagrams
│   ├── phase-log.md       # Step-by-step phase documentation
│   └── lessons-learned.md # Issues encountered and resolved
├── rules/
│   └── local.rules   # Custom Suricata detection rules
└── captures/
    └── README.md     # Traffic capture scenario descriptions
```

## Snapshots

| Name | Description |
|------|-------------|
| fresh-install | Clean OpenWRT boot |
| base-config | Root password, LAN IP configured |
| luci-accessible | Static WAN, LuCI working |
| hardened-phase4 | SSH keys, firewall restricted |
| wireguard-vpn | VPN tunnel operational |
| phase6-monitoring | tcpdump, Wireshark, Suricata deployed |
