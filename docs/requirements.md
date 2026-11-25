# Project 3: Home DNS Server and Ad Blocker
## Requirements (v1.1)

**Author:** <your name>
**Date:** 2026-11-14
**Status:** Approved for design

---

## 1. Problem Statement

Ads and trackers appear on nearly every website and app on every device in the
house, including phones, laptops, and TVs, where a browser extension cannot be
installed. Besides being disruptive, ads are a security risk: malicious ads
(malvertising) can deliver malware, and trackers build profiles of our activity
across many sites.

## 2. Functional Requirements (what the system must do)

| ID | Requirement |
|----|-------------|
| FR-1 | The system must block known ad, tracker, and malicious domains for every device that uses it for DNS. |
| FR-2 | The system must allow an admin to allowlist a domain that was blocked by mistake. |
| FR-3 | The system must cover, at minimum, phones, computers/laptops, and TVs. |
| FR-4 | If a domain is not on the blocklist, the system must forward the lookup to an upstream resolver and return the real IP address. |
| FR-5 | The system must keep a query log showing which device asked for which domain and whether it was blocked or allowed. |

## 3. Non-Functional Requirements (how well it must do it)

| ID | Category | Requirement |
|----|----------|-------------|
| NFR-1 | Availability | If either DNS server is down, every device must still be able to resolve names. |
| NFR-2 | Performance | Measured from the laptop: cached lookups answer in under 40 ms; uncached lookups answer in under 110 ms. |
| NFR-3 | Security | The admin page must be password-protected, and only the two household admins may change settings. |

## 4. Constraints

- Hardware: two hosts only: the GMKtec M6 Ultra running Proxmox (lab) and the
  Dell OptiPlex 7060 Micro ("production").
- Budget: $0 beyond hardware already owned.

## 5. Out of Scope (v1)

- Building a custom central dashboard to manage the homelab (later project).
- Access from outside the home network (a VPN is a later project).
- Taking over DHCP from the Xfinity gateway.

## 6. Success Criteria (acceptance tests)

| Test | How | Pass if |
|------|-----|---------|
| T1: Blocking | `nslookup doubleclick.net <DNS server IP>` | Returns 0.0.0.0 or "Non-existent domain" |
| T2: Normal lookups | `nslookup www.google.com <DNS server IP>` | Returns a real IP |
| T3: Speed | `Measure-Command { Resolve-DnsName google.com -Server <DNS server IP> }`, run twice | Meets NFR-2 |
| T4: Failover | Reboot the primary server, then browse; repeat with the secondary | Names resolve the whole time |
| T5: Devices | Open an ad-heavy page on a laptop, a phone, and in a TV app | Banner ads gone (see Known Limitations) |
| T6: Logging | Check the query log after T1 | T1's lookup appears with the device and "blocked" |
| T7: Allowlist | Allowlist a blocked domain, then repeat T1 for it | Returns a real IP |

## 7. Known Limitations and Risks

- Ads served from the same domain as the content (e.g., YouTube, Netflix)
  cannot be blocked by DNS without blocking the service itself.
- Some devices may use hard-coded DNS servers and skip ours (investigate in design).
- If all DNS servers fail, the internet will appear "down" to every device even
  though the connection works. NFR-1 exists to prevent this.
## Change Log

| Version | Date | Change | Reason |
|---------|------|--------|--------|
| 1.0 | 2026-11-14 | Initial requirements | — |
| 1.1 | 2026-11-24 | DHCP for the house moves to the Archer BE400. Added test T8 (no IPv6 bypass) | Xfinity gateway can't hand out custom DNS or disable DHCP (evidence 2026-11-15) |
