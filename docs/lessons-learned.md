# Lessons Learned

## 1. Pre-flight Compatibility Research Saves Hardware
Checking the OpenWRT Table of Hardware before attempting a flash prevented bricking the TP-Link device. The Ver:3.0 variant uses a different chipset than earlier versions — version number matters, not just model name.

## 2. Virtualisation Permissions Are Layered
KVM requires: CPU flags → /dev/kvm access → group membership → bridge helper setuid → bridge.conf permissions. Missing any single layer produces cryptic errors. Methodical layer-by-layer verification is faster than guessing.

## 3. Capture Position Determines What You See
Capturing on eth1 shows encrypted WireGuard blobs. Capturing on wg0 shows decrypted tunnel contents. Capturing on br-lan shows LAN traffic. Where you place your tap is as important as what you're looking for.

## 4. UCI Index Shifting
Deleting OpenWRT firewall rules with `uci delete firewall.@rule[N]` renumbers all subsequent rules immediately. Always verify indices with `uci show firewall | grep name` after each deletion before proceeding.

## 5. Dropbear vs OpenSSH
OpenWRT uses Dropbear instead of OpenSSH. Dropbear lacks the sftp-server subsystem, so modern SCP fails with "sftp-server not found". Workaround: `ssh host 'cat /path/file' > local_file` pipes file content over the SSH connection directly.

## 6. Suricata Rule File Location
Suricata's config references rules relative to the data directory (/var/lib/suricata/rules/), not the config directory (/etc/suricata/rules/). Custom rules must be placed in or copied to the data directory to be loaded.

## 7. WireGuard Port Appears open|filtered
Nmap reports WireGuard's UDP port as open|filtered because WireGuard intentionally ignores packets without a valid cryptographic handshake. This is a security feature, not a scanning artefact.

## 8. rfc1918_filter Blocks LuCI Access from Virtual Networks
OpenWRT's uhttpd has rfc1918_filter enabled by default, which blocks access to the web UI from private IP ranges on the WAN interface. In a virtualised lab where the host connects via a private bridge IP, this must be disabled.
