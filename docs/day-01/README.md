# Day 1 — Interface Setup, Routing & Troubleshooting

## Objective

Create two separate IPv4 networks behind pfSense and verify that Kali and Ubuntu can communicate through the firewall. Record the troubleshooting process rather than only the final working state.

## 1. Environment

- Hypervisor: Oracle VirtualBox
- Firewall: pfSense
- Test hosts: Kali Linux and Ubuntu
- Windows host connectivity: not yet working; investigation remains open

### VirtualBox adapters reported

| VM | Adapter | Mode / network label | Role |
|---|---|---|---|
| pfSense | Adapter 1 | NAT | WAN/upstream |
| pfSense | Adapter 2 | Internal Network — `int network` | LAN side |
| pfSense | Adapter 3 | Internal Network — `attacker` | OPT1 / attacker side |
| Kali | Adapter 1 | Internal Network — `attacker` | Connects to pfSense OPT1 |

Confirm adapter mappings in the VM settings and pfSense console if the topology is changed.

## 2. Addressing configured

| Device / interface | IPv4 | Notes |
|---|---|---|
| pfSense OPT1 | `192.168.20.1/24` | Static IPv4; IPv6 set to None |
| Kali `eth0` | `192.168.20.10/24` | Configured manually for the lab |
| Ubuntu | `192.168.1.2` (reported) | Ubuntu-side LAN |
| pfSense LAN | `192.168.1.1/24` (reported earlier) | Verify in current UI before relying on it |

For OPT1, the `/24` subnet mask is `255.255.255.0`. The subnet is `192.168.20.0/24`, with usable host addresses `192.168.20.1` through `192.168.20.254`; `.0` is the network address and `.255` is the broadcast address.

## 3. pfSense OPT1 configuration

1. Open **Interfaces → Assignments**.
2. Add the available `em2` port as an interface; pfSense then labels it OPT1.
3. Open **Interfaces → OPT1**.
4. Enable the interface.
5. Set IPv4 Configuration Type to **Static IPv4**.
6. Set IPv4 address to `192.168.20.1/24`.
7. Set IPv6 Configuration Type to **None**.
8. Leave the upstream gateway as **None**; OPT1 is an internal interface, not the WAN.
9. Leave **Block private networks** and **Block bogon networks** unchecked for this internal lab interface.
10. Save and apply changes.

A DHCP warning appeared during apply. The changes were applied successfully. DHCP was not documented as configured for OPT1; Kali was assigned a static address.

## 4. Kali IPv4 and route configuration

Commands used during troubleshooting:

```bash
ip -br addr
ip -4 addr show dev eth0
```

These display interface state and IPv4 addresses.

A new address was added to `eth0`:

```bash
sudo ip addr add 192.168.20.10/24 dev eth0
```

The old IPv4 address was then removed using its actual address (replace the example with the exact old address):

```bash
sudo ip addr del OLD_IP/24 dev eth0
```

The route table initially showed the old default gateway `192.168.1.1`. The default route was changed:

```bash
sudo ip route replace default via 192.168.20.1 dev eth0
ip route
```

The old default route was removed:

```bash
sudo ip route del default via 192.168.1.1
```

**Important:** These commands change runtime network settings and may not survive reboot. Removing the old address or route can interrupt existing connectivity. Confirm the actual address and route before running a delete command.

## 5. Connectivity tests and findings

| Test | Result reported | What it tells us |
|---|---|---|
| Kali → pfSense OPT1 (`192.168.20.1`) before rule | 100% packet loss | Gateway reachability failed from Kali initially |
| pfSense Diagnostics → Ping, source OPT1, target Kali (`192.168.20.10`) | 3 pings succeeded | pfSense could reach Kali in the reverse direction; this alone did not prove Kali-initiated traffic was allowed |
| Kali → pfSense OPT1 after OPT1 pass rule | Successful | The rule change resolved the reported gateway ping issue |
| Kali → Ubuntu LAN host | Successful | Inter-subnet connectivity worked after routing/rules were in place |
| Ubuntu → Kali | Successful | Return-direction test also worked |
| Windows → Kali | Not working / not yet diagnosed | Requires separate host/adapter/routing investigation |

## 6. Firewall rule troubleshooting

The **Firewall → Rules → OPT1** tab was empty. A rule was added with these reported settings:

- Action: **Pass**
- Source: **Single host or alias** → `192.168.20.10`
- Destination: **Any** (used for initial troubleshooting)

After applying the rule, Kali could ping pfSense OPT1. Kali-to-Ubuntu and Ubuntu-to-Kali ping tests were then reported successful.

### Security note

The destination **Any** rule is broad. It was used as a troubleshooting step, not a final least-privilege design. For a hardened lab, replace it with rules that allow only required destinations/protocols (for example, ICMP to the gateway for a specific diagnostic test), enable logging while troubleshooting, and verify the resulting traffic in **Status → System Logs → Firewall**. Avoid leaving unnecessary broad rules enabled.

## 7. Troubleshooting commands quick reference

| Command | Purpose |
|---|---|
| `ip -br addr` | Brief view of interfaces and addresses |
| `ip -4 addr show dev eth0` | Inspect IPv4 addresses on Kali's eth0 |
| `ip route` | Inspect routing table and default gateway |
| `ping -c 4 192.168.20.1` | Test Kali → pfSense OPT1 |
| `ping -c 4 192.168.1.2` | Test Kali → Ubuntu (replace with Ubuntu's verified IP) |
| `sudo ip addr add 192.168.20.10/24 dev eth0` | Add Kali's lab IPv4 address |
| `sudo ip addr del OLD_IP/24 dev eth0` | Remove a specific old address (substitute the exact address) |
| `sudo ip route replace default via 192.168.20.1 dev eth0` | Set Kali's default route via pfSense OPT1 |
| `sudo ip route del default via 192.168.1.1` | Remove the old default route, if it exists |

## 8. Day 1 outcome

**Completed:** OPT1 assignment and static IPv4 configuration; Kali static IP and default route adjustment; an OPT1 pass rule; successful Kali ↔ pfSense, Kali ↔ Ubuntu, and Ubuntu ↔ Kali ping tests (as reported).

**Pending:** Windows-to-Kali connectivity; reviewing and tightening the temporary broad pass rule; collecting screenshots and firewall log evidence.

## 9. Evidence to add

Save sanitized screenshots under `docs/day-01/evidence/` (create the folder when adding images):

- [ ] VirtualBox network adapters for pfSense and Kali
- [ ] pfSense Interfaces → OPT1 configuration
- [ ] Kali `ip -br addr` and `ip route`
- [ ] Firewall → Rules → OPT1
- [ ] Successful ping output in both directions
- [ ] Windows `ipconfig` and the exact failed ping result (hide unrelated/private details)

Do not claim a screenshot or log exists until it has been captured.
