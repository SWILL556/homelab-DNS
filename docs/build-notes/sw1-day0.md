# sw1 — Cisco Catalyst 3560-CX Day 0 Build Notes

| Field | Value |
|---|---|
| Device | Cisco Catalyst WS-C3560CX-8PC-S (used) |
| IOS | 15.2(7)E10, license ipbase |
| Hostname | sw1 (sw1.home.arpa) |
| Mgmt IP | 192.168.10.2/24 on Vlan1, default gateway 192.168.10.1 |
| Access | Console + SSH v2 only (Telnet and HTTP/HTTPS disabled) |
| Dates | Wipe 2026-11-25, baseline 2026-11-27 |

## 1. Initial inspection (2026-11-25)
- Console: USB-to-RJ45, PuTTY Serial, 9600 8-N-1, no flow control.
- No startup config (setup dialog shown). Only default VLANs; no `vlan.dat`.
- Leftovers from previous owner: `private-config.text`, old IOS 15.2(7)E2 image, 15.2(4)E6 directory.
- Password recovery: enabled. All POST tests passed.

## 2. Wipe
- Verified `show boot` → BOOT path = E10 image.
- `write erase` → removed `private-config.text`.
- Deleted old E2 image and E6 directory (kept running E10).
- `reload` → booted clean to the setup dialog.

## 3. Baseline config
- Identity: hostname, domain `home.arpa`, `no ip domain-lookup`.
- Auth: enable secret + local admin user (scrypt / type 9), `service password-encryption`. Secrets stored in password manager.
- Remote access: RSA 2048 key, SSH v2, VTY `transport input ssh`, `login local`; HTTP servers disabled.
- Lines: `logging synchronous`, 15-min `exec-timeout`.
- Mgmt: `interface vlan 1` 192.168.10.2/24, `ip default-gateway 192.168.10.1`, MOTD banner.
- Saved: `copy running-config startup-config`.

## 4. Verification
- `show ip interface brief`: Vlan1 192.168.10.2 up/down (expected — no uplink yet).
- `show ip ssh`: SSH v2.0, 2048-bit key.
- Console login with admin → lands at `sw1#`.

## 5. Open items
- [ ] Connect to BE400 and test SSH from laptop.
- [ ] NTP (clock currently wrong).
- [ ] Remove legacy SSH algorithms (Project 7).
- [ ] VLANs (Project 5).
