# Day 1 — pfSense setup and connectivity tests

## Objective
Set up pfSense interfaces in VirtualBox, connect Kali and Ubuntu to their intended networks, and understand the packet path before writing restrictive rules.

## Lab setup

| Device | Adapter / network | Role |
|---|---|---|
| pfSense | Adapter 1 — NAT | WAN |
| pfSense | Adapter 2 — Internal Network: `int network` | LAN |
| pfSense | Adapter 3 — Internal Network: `attacker` | OPT1 |
| Kali | Adapter 1 — Internal Network: `attacker` | OPT1 test client |
| Ubuntu | LAN-side adapter | Internal test machine |

Recheck adapter assignments in VirtualBox and **Interfaces → Assignments** in pfSense if the topology changes.

## Addressing

Real IPs are omitted from this public copy. Replace placeholders with the current values from your own lab.

| Device | Address |
|---|---|
| pfSense OPT1 | `<OPT1_GATEWAY>/<PREFIX>` |
| Kali interface | `<KALI_IP>/<PREFIX>` |
| Ubuntu | `<UBUNTU_IP>/<PREFIX>` |
| pfSense LAN | `<LAN_GATEWAY>/<PREFIX>` |

## Commands used on Kali

```bash
ip -br addr
ip -4 addr show dev eth0
ip route
```

Example command patterns (substitute the actual values):

```bash
sudo ip addr add <KALI_IP>/<PREFIX> dev eth0
sudo ip route replace default via <OPT1_GATEWAY> dev eth0
ip route
```

These commands change the current interface configuration and may not persist after reboot. Inspect existing addresses and routes before replacing anything.

## Connectivity troubleshooting

During initial setup, Kali could not ping the pfSense OPT1 gateway while the OPT1 rules page had no applicable allow rule. A temporary pass rule was added with the Kali host as source and Any as destination. After applying it, Kali could ping the gateway. Kali and Ubuntu ping tests were also reported as successful. Windows-to-Kali connectivity remains unresolved.

The temporary destination-Any rule is broad and must not be treated as a final least-privilege policy. Replace it with a narrowly scoped rule after confirming the traffic path.

## Test record

| Test | Reported result |
|---|---|
| Kali → pfSense OPT1 before temporary pass rule | Failed |
| Kali → pfSense OPT1 after temporary pass rule | Worked |
| Kali ↔ Ubuntu ping | Worked |
| Windows → Kali | Still pending |

These are user-reported lab observations. Screenshots, exact timestamps and corresponding firewall logs have not yet been attached.

## Useful pfSense dashboard locations

- Interface configuration: **Interfaces → Assignments**, then select the interface.
- Rules: **Firewall → Rules → OPT1** (or the actual ingress interface).
- Firewall logs: **Status → System Logs → Firewall**.

## Evidence checklist

- [ ] VirtualBox adapter settings
- [ ] pfSense interface assignments and interface addresses (redacted)
- [ ] Kali `ip -br addr` and `ip route` output
- [ ] OPT1 rule screenshot
- [ ] Ping outputs and matching firewall log entries
- [ ] Windows-to-Kali troubleshooting evidence

## Key lesson
A rule is evaluated on the interface where traffic enters pfSense. Confirm the route and ingress interface first. A broad Any-to-Any-style pass rule is useful only as a temporary diagnostic measure and should be replaced with least-privilege policy.
