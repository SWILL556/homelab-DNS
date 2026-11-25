# Project 3: Home DNS Server and Ad Blocker
## Design (v1.0)

**Author:** Seldon Williams · **Date:** 2026-11-24 · **Status:** Approved
**Related:** requirements.md (v1.1) · ip-plan.md (v1.0) · network-diagram.png (v1.0)

---

## 1. Summary

Two Pi-hole DNS servers, one on each physical host, provide ad/tracker
blocking for every device in the house. An Archer BE400 router, placed
behind the Xfinity gateway, becomes the house's router and DHCP server so it
can hand out both Pi-holes as DNS servers. The Xfinity gateway can't.

## 2. Key Decisions

| # | Decision | Why | Alternatives considered |
|---|----------|-----|-------------------------|
| D1 | Use the Archer BE400 as the house router (double NAT behind Xfinity) | Xfinity gateway has no DHCP DNS field and can't disable DHCP (evidence: gateway screenshots, 2026-11-15). BE400 has Primary/Secondary DNS fields in DHCP | Manual DNS per device (doesn't meet FR-3); second DHCP server on the Xfinity LAN (rogue DHCP); GMKtec as router (it's the lab box) |
| D2 | Double NAT first, gateway bridge mode later only if needed | Easy rollback: unplug the BE400 and the house is back to normal | Bridge mode now (harder to undo) |
| D3 | Pi-hole v6 | Largest community/docs, built-in web admin, local DNS records | AdGuard Home, Technitium |
| D4 | Two Pi-holes on separate hardware (GMKtec + OptiPlex) | Removes the single point of failure; a lab reboot doesn't take down the house's DNS (NFR-1) | Two Pi-holes on one Proxmox host (shared failure) |
| D5 | Pi-holes run as LXC containers on Proxmox; OptiPlex also runs Proxmox (standalone, not clustered) | Lightweight, snapshots before changes, Jellyfin isolated from DNS | VMs; bare-metal; Docker. Clustering rejected (two-node quorum issue) |
| D6 | House LAN 192.168.10.0/24, IPv6 off on BE400 | See ip-plan.md. IPv6 off closes the Comcast IPv6 DNS bypass | Keeping IPv6 (needs extra work to stop bypass) |
| D7 | Cisco 3560-CX as the LAN switch, Day 0 baseline only (VLAN 1) | Owned, going in the rack. VLANs deferred to Project 5 | Plug hosts straight into BE400 |
| D8 | Upstream DNS: Quad9 (9.9.9.9) primary, Cloudflare (1.1.1.1) secondary | Quad9 also blocks known-malicious domains (supports FR-1); two different providers so one outage doesn't break lookups | Comcast DNS; Google DNS |
| D9 | After cutover, turn off the Xfinity gateway's Wi-Fi (or change its password so only admins know it) | Any device on Xfinity Wi-Fi skips Pi-hole, which breaks FR-3. Keeping it during the pilot gives the family a fallback | Leave it on (family devices may bypass Pi-hole) |

## 3. Components

| Name | Type | Host | IP |
|------|------|------|----|
| BE400 | Router / DHCP / Wi-Fi | — | 192.168.10.1 |
| C3560-CX | Switch (VLAN 1) | — | 192.168.10.2 (mgmt) |
| pve1 | Proxmox host | GMKtec | 192.168.10.10 |
| pihole1 | Pi-hole LXC | pve1 | 192.168.10.11 |
| pve2 | Proxmox host | OptiPlex | 192.168.10.20 |
| pihole2 | Pi-hole LXC | pve2 | 192.168.10.21 |

## 4. How a Lookup Flows

Device joins BE400 Wi-Fi → DHCP gives it IP + DNS 192.168.10.11, .21 →
device asks a Pi-hole → blocked domain = 0.0.0.0 · allowed domain =
forwarded to Quad9/Cloudflare and real IP returned → logged per device.

## 5. Failure Modes

| If this fails… | What happens | Mitigation |
|----------------|--------------|------------|
| One Pi-hole | Devices use the other one | D4; keep both blocklists in sync |
| Both Pi-holes | Names don't resolve; "internet down" | Separate hardware; rollback below |
| BE400 | House loses internet (single point of failure, accepted for v1) | Rollback below; UPS planned |
| GMKtec (lab) | pihole1 down only | pihole2 on OptiPlex |
| Power outage | Everything down | Future UPS |

**Sync:** the two Pi-holes don't sync automatically. For v1, export
settings from pihole1 with Pi-hole's Teleporter and import them on pihole2
after every change. Automating this is a future improvement.

## 6. Build Order (parallel network)

The new 192.168.10.0/24 network is built **beside** the existing Xfinity
network. Family devices stay on Xfinity Wi-Fi until cutover.

| Phase | Work | Family affected? |
|-------|------|------------------|
| 1 | BE400: LAN → 192.168.10.1/24, pool .100–.199. Switch Day 0 baseline (console wipe, hostname, mgmt .2, SSH). Rack and cable | No |
| 2 | Move lab: GMKtec → .10, lab-ubuntu01 → .50 | No |
| 3 | Install Proxmox on OptiPlex (.20); build pihole1 (.11) and pihole2 (.21); tests T1–T3, T6, T7 | No |
| 4 | **Pilot:** BE400 DNS fields → .11/.21; only admin laptop + phone on BE400 Wi-Fi for a few days; tests T4, T5, T8; measure NFR-2 | No |
| 5 | **Cutover:** family devices join BE400 Wi-Fi; apply D9 | Yes (planned window) |

**Rollback (any phase):** devices rejoin the Xfinity Wi-Fi, which is
unchanged until phase 5. After phase 5: turn the Xfinity Wi-Fi back on.

## 7. Security

- Pi-hole admin pages: strong password, two admins only (NFR-3).
- Pi-hole, Proxmox, and switch management reachable from the LAN only;
  no port forwarding from the internet.
- Switch: SSH only (no Telnet), enable secret set.
- No public IPs or passwords in this repository.

## 8. Open Items

- Automate Pi-hole sync (future).
- Gateway bridge mode if double NAT causes problems (future).
- UPS for BE400, switch, and both hosts (future).
