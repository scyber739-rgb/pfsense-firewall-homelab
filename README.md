# pfSense Firewall Homelab

A hands-on firewall and network-routing lab built with **pfSense CE**, **Kali Linux**, **Ubuntu**, and **VirtualBox**. The goal is to learn interface assignment, IPv4 addressing, routing between subnets, firewall rule evaluation, and systematic troubleshooting.

> **Lab status:** Day 1 basic topology and inter-subnet connectivity were configured and tested. Windows-to-lab connectivity is still pending investigation. This repository documents only the work and observations reported during the lab; add screenshots/logs as evidence when available.

## Lab goals

- Build separate LAN and attacker subnets.
- Assign and verify pfSense interfaces.
- Configure Kali's lab IP and default route.
- Use firewall rules to permit specific lab traffic.
- Troubleshoot failed ICMP connectivity with route, interface, and firewall checks.
- Keep the lab isolated and avoid exposing test systems to untrusted networks.

## Topology (logical)

```text
                         VirtualBox
                    +------------------+
                    |     pfSense      |
Internet / NAT ---->| WAN              |
                    |                  |
Ubuntu LAN <------->| LAN               |
                    |                  |
Kali attacker <---->| OPT1              |
                    +------------------+
```

The diagram is logical, not a physical wiring diagram. Confirm the exact adapter-to-interface mapping in pfSense before making changes.

## Address plan used in the lab

| Node / interface | Address / network | Purpose |
|---|---|---|
| pfSense WAN | VirtualBox NAT; observed earlier as DHCP-assigned | Upstream connectivity |
| pfSense LAN | Ubuntu-side LAN; reported as `192.168.1.0/24` | Internal LAN |
| Ubuntu | Reported as `192.168.1.2` | LAN test host |
| pfSense OPT1 | `192.168.20.1/24` | Attacker-side gateway |
| Kali `eth0` | `192.168.20.10/24` | Attacker-side test host |
| Kali default gateway | `192.168.20.1` | Route to other networks through pfSense |

Use these addresses as the current lab notes, then verify them against screenshots/command output before treating them as final evidence. Do not publish real public IPs, credentials, API keys, or sensitive host information.

## Day 1

See [Day 1 — Interface Setup, Routing & Troubleshooting](docs/day-01/README.md).

## Safety and scope

- Run tests only on your own isolated VirtualBox lab.
- Do not use broad `Any → Any` allow rules as a permanent policy.
- The Day 1 OPT1 pass rule was added to troubleshoot connectivity. Review and narrow it before treating the configuration as hardened.
- Linux `ip` commands change runtime network state and may be lost after reboot. Verify the current route before changing it.
- Capture screenshots of pfSense interface settings, firewall rules, successful/failed pings, and route output as evidence.

## Planned roadmap

- [x] Day 1 — Interface assignment, IP configuration, routing and initial connectivity troubleshooting
- [ ] Day 2 — Rule ordering, least privilege, logging and controlled allow/deny tests
- [ ] Day 3 — NAT concepts and controlled traffic-flow validation
- [ ] Day 4 — Windows/host connectivity and packet-capture troubleshooting
- [ ] Later — Documentation cleanup, test matrix, and resume-ready results

## Evidence checklist

- [ ] VirtualBox adapter settings (with sensitive host details hidden)
- [ ] pfSense interface assignment and OPT1 IPv4 settings
- [ ] Kali `ip -br addr` and `ip route`
- [ ] Firewall → Rules → OPT1 rule
- [ ] Successful Kali ↔ pfSense ping
- [ ] Successful Kali ↔ Ubuntu ping
- [ ] Windows connectivity test results
