# pfSense Firewall Homelab

**Hands-on pfSense network security lab** | VirtualBox · pfSense CE/Plus · Kali Linux · Ubuntu · Windows host

## Project overview
This repository documents a controlled VirtualBox homelab for learning firewall administration, packet flow, access-control rules, state tracking, aliases, NAT, logging and network segmentation. Each challenge separates what is configured, what was tested, what was actually observed and what still needs evidence.

## Lab architecture
```text
                 VirtualBox NAT
                       |
                    pfSense
                  /         \
                LAN         OPT1
                 |            |
              Ubuntu         Kali
```
The diagram is a logical overview. Verify the actual VirtualBox adapters, pfSense interface assignments, routes and gateways before each test. Two VMs on the same Layer-2 network may communicate directly without their packets crossing pfSense.

## Lab devices

| Device | Role |
|---|---|
| pfSense WAN | Outside connection through VirtualBox NAT |
| pfSense LAN | Internal Ubuntu-side network |
| pfSense OPT1 | Kali test network |
| Ubuntu | Internal test server |
| Kali Linux | Authorized test client |
| Windows host | Additional connectivity test source; Windows-to-Kali test remains pending |

Public documentation uses placeholders such as `<KALI_IP>`, `<UBUNTU_IP>` and `<OPT1_GATEWAY>`. Replace them locally with current lab values before running commands.

## Learning path

| Day | Topic | pfSense lab status |
|---|---|---|
| 01 | Interface setup and basic connectivity | Interfaces and basic Kali/Ubuntu connectivity tested; evidence screenshots pending |
| 02 | PASS/BLOCK rules, matching and rule order | Learning notes added; targeted rule-order validation pending |
| 03 | TCP/UDP ports, HTTP testing and state tracking | Ping connectivity is reported; HTTP/TCP test and state evidence pending |
| 04 | IP aliases and reusable firewall policies | Documentation added; alias configuration and validation in pfSense pending |
| 05 | NAT — WAN port forwarding to an Ubuntu test service | Initial test documented; external/target/listener port mapping needs confirmation |\n| 06–15 | Segmentation, outbound NAT, DHCP/DNS, logging, VPN, HA, IDS/IPS and capstone | Planned |

## Documentation

- [Day 1 — Setup and connectivity](docs/day-01/README.md)
- [Day 1 — Network architecture](docs/day-01/network-architecture.md)
- [Day 2 — Rule processing and order](docs/day-02/README.md)
- [Day 3 — Protocol, port and stateful testing](docs/day-03/README.md)
- [Day 4 — IP aliases](docs/day-04/README.md)
- [Configuration register](docs/configuration/README.md)

## pfSense dashboard paths used in this lab

- Rules: **Firewall → Rules → [interface tab]**
- Aliases: **Firewall → Aliases**
- Firewall logs: **Status → System Logs → Firewall**
- Active connections: **Diagnostics → States**

## Evidence standard
A rule being saved does not prove that it worked. For each test, record the exact client command/output, the matching pfSense log entry and (where useful) the active state. A timeout alone does not prove a firewall block. Do not use an ICMP log entry as proof that a TCP/80 request was blocked.

## Safety and privacy
Keep test services on isolated lab networks. Do not expose them to the public internet. Do not publish passwords, API keys, raw configuration exports or screenshots containing private details. Redact addresses and host identifiers before committing evidence.

## Current next step
Complete the Day 2 targeted rule-order test, then establish a successful HTTP baseline for Day 3 before applying a TCP/80 policy. Only mark a result verified after collecting the client output and corresponding pfSense log.
