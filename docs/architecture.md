# Lab Architecture

## Network Topology
```
INTERNET
    │
    ▼
Huawei B636 (ISP Router)
192.168.1.1
    │
    ▼
HP EliteBook 850 G3 (KVM Host)
Ubuntu — 192.168.1.x (physical)
virbr0 — 192.168.122.1 (virtual bridge)
    │
    ▼
OpenWRT VM (KVM/QEMU)
  WAN: eth1 = 192.168.122.2
  LAN: br-lan = 192.168.2.1
  VPN: wg0 = 10.0.0.1/24
```

## WireGuard Tunnel
```
Linux Host                    OpenWRT VM
wg-lab: 10.0.0.2    ←───►    wg0: 10.0.0.1
         └── Encrypted UDP ──── port 51820
```

All management traffic (SSH, LuCI) flows through the encrypted tunnel.

## Interface Roles

| Interface | Device | IP | Role |
|-----------|--------|----|------|
| virbr0 | KVM host | 192.168.122.1 | Virtual bridge — connects host to VMs |
| eth1 | OpenWRT | 192.168.122.2 | WAN — uplink to host/internet |
| br-lan | OpenWRT | 192.168.2.1 | LAN — internal network |
| wg0 | OpenWRT | 10.0.0.1 | WireGuard VPN server endpoint |
| wg-lab | Linux host | 10.0.0.2 | WireGuard VPN client endpoint |

## Design Decisions

### Why KVM instead of physical flashing
The TP-Link TD-W8961N Ver:3.0 uses a proprietary Broadcom ADSL chipset not supported by OpenWRT. Virtualization provides identical learning outcomes with no hardware risk and adds hypervisor management skills to the portfolio.

### Why WireGuard over OpenVPN
WireGuard has ~4,000 lines of code vs ~600,000 for OpenVPN — smaller attack surface, faster performance, and is now the industry standard for new VPN deployments.

### Why split tunneling
AllowedIPs=10.0.0.0/24 routes only lab traffic through the VPN. Internet traffic takes the direct path — this is intentional lab design, not a security gap.

### Capture interface selection
- eth1: sees raw WAN traffic including encrypted WireGuard packets
- wg0: sees decrypted traffic inside the VPN tunnel
- br-lan: would see LAN-side traffic (not used in this lab)
