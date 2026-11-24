# IP Plan: House LAN

**Version:** 1.0 · **Date:** 2026-11-23 · **Status:** Approved

## Subnet
- **Network:** 192.168.10.0/24 (future VLAN 10)
- **Gateway:** 192.168.10.1 (Archer BE400)
- **Domain:** home.arpa (RFC 8375)

Why: distinct from the Xfinity side (10.0.0.0/24), avoids the common
192.168.0.x/1.x defaults that clash over VPN, and the third octet
matches future VLAN IDs (10 house, 20 lab, 30 IoT, 99 mgmt).

## Address Layout

| Range | Purpose | Assigned |
|-------|---------|----------|
| .1 | Default gateway | BE400 |
| .2–.9 | Network gear | .2 Cisco 3560-CX mgmt (future) |
| .10–.19 | GMKtec ("lab") | .10 pve1 (Proxmox), .11 pihole1 (LXC) |
| .20–.29 | OptiPlex ("prod") | .20 pve2 (Proxmox), .21 pihole2 (LXC), .22 jellyfin (LXC) |
| .50–.99 | Lab VMs | .50 lab-ubuntu01 |
| .100–.199 | DHCP pool | Clients |
| .200–.254 | Spare / future static | |

## Rules
- Static addresses are always outside the DHCP pool.
- Addresses are grouped by host, so the IP tells you which box it lives on.
- All statics use /24 with gateway 192.168.10.1.

## Planned Changes

| Device | Current | New |
|--------|---------|-----|
| Proxmox (GMKtec) | 10.0.0.151 | 192.168.10.10 |
| lab-ubuntu01 | 10.0.0.152 | 192.168.10.50 |
