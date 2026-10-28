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

## 2. Sanitized addressing

Actual IPv4 values have been replaced with placeholders for this public copy.

| Device / interface | IPv4 placeholder | Notes |
|---|---|---|
| pfSense OPT1 | `<OPT1_GATEWAY>/<PREFIX>` | Static IPv4; IPv6 set to None |
| Kali `eth0` | `<KALI_IP>/<PREFIX>` | Configured manually for the lab |
| Ubuntu | `<UBUNTU_IP>` | Ubuntu-side LAN |
| pfSense LAN | `<LAN_GATEWAY>/<PREFIX>` | Verify in current UI before relying on it |

For the original lab, the prefix length was configured as `/24` on OPT1. In your own notes, record the subnet mask and address plan privately. Replace placeholders with your verified local values before running any command shown below.

## 3. pfSense OPT1 configuration

1. Open **Interfaces → Assignments**.
2. Add the available port as an interface; pfSense then labels it OPT1.
3. Open **Interfaces → OPT1**.
4. Enable the interface.
5. Set IPv4 Configuration Type to **Static IPv4**.
6. Set the IPv4 address to your private `<OPT1_GATEWAY>/<PREFIX>` value.
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

These display interface state and IPv4 addresses. Review the output locally; redact actual addresses before sharing it publicly.

A lab address was added to `eth0` with this command pattern:

```bash
sudo ip addr add <KALI_IP>/<PREFIX> dev eth0
```

The old IPv4 address was then removed using its actual address (replace both placeholders with the exact old address and prefix):

```bash
sudo ip addr del <OLD_IP>/<PREFIX> dev eth0
```

The route table initially showed an old default gateway. The default route was changed using this pattern:

```bash
sudo ip route replace default via <OPT1_GATEWAY> dev eth0
ip route
```

The old default route was removed using its actual gateway:

```bash
sudo ip route del default via <OLD_GATEWAY>
```

**Important:** These commands change runtime network settings and may not survive reboot. Removing the old address or route can interrupt existing connectivity. Confirm the actual address and route before running a delete command. Do not type the angle-bracket placeholders literally; substitute your verified lab values.

## 5. Connectivity tests and findings

| Test | Result reported | What it tells us |
|---|---|---|
| Kali → pfSense OPT1 before rule | 100% packet loss | Gateway reachability failed from Kali initially |
| pfSense Diagnostics → Ping, source OPT1, target Kali | 3 pings succeeded | pfSense could reach Kali in the reverse direction; this alone did not prove Kali-initiated traffic was allowed |
| Kali → pfSense OPT1 after OPT1 pass rule | Successful | The rule change resolved the reported gateway ping issue |
| Kali → Ubuntu LAN host | Successful | Inter-subnet connectivity worked after routing/rules were in place |
| Ubuntu → Kali | Successful | Return-direction test also worked |
| Windows → Kali | Not working / not yet diagnosed | Requires separate host/adapter/routing investigation |

## 6. Firewall rule troubleshooting

The **Firewall → Rules → OPT1** tab was empty. A rule was added with these reported settings:

- Action: **Pass**
- Source: **Single host or alias** → `<KALI_IP>`
- Destination: **Any** (used for initial troubleshooting)

After applying the rule, Kali could ping pfSense OPT1. Kali-to-Ubuntu and Ubuntu-to-Kali ping tests were then reported successful.

### Security note

The destination **Any** rule is broad. It was used as a troubleshooting step, not a final least-privilege design. For a hardened lab, replace it with rules that allow only required destinations/protocols (for example, ICMP to the gateway for a specific diagnostic test), enable logging while troubleshooting, and verify the resulting traffic in **Status → System Logs → Firewall**. Avoid leaving unnecessary broad rules enabled.

## 7. Troubleshooting commands quick reference

Replace placeholders with the verified values from your own lab before running commands.

| Command | Purpose |
|---|---|
| `ip -br addr` | Brief view of interfaces and addresses |
| `ip -4 addr show dev eth0` | Inspect IPv4 addresses on Kali's eth0 |
| `ip route` | Inspect routing table and default gateway |
| `ping -c 4 <OPT1_GATEWAY>` | Test Kali → pfSense OPT1 |
| `ping -c 4 <UBUNTU_IP>` | Test Kali → Ubuntu |
| `sudo ip addr add <KALI_IP>/<PREFIX> dev eth0` | Add Kali's lab IPv4 address |
| `sudo ip addr del <OLD_IP>/<PREFIX> dev eth0` | Remove a specific old address (substitute exact values) |
| `sudo ip route replace default via <OPT1_GATEWAY> dev eth0` | Set Kali's default route via pfSense OPT1 |
| `sudo ip route del default via <OLD_GATEWAY>` | Remove the old default route, if it exists |

## 8. Day 1 outcome

**Completed:** OPT1 assignment and static IPv4 configuration; Kali static IP and default route adjustment; an OPT1 pass rule; successful Kali ↔ pfSense, Kali ↔ Ubuntu, and Ubuntu ↔ Kali ping tests (as reported).

**Pending:** Windows-to-Kali connectivity; reviewing and tightening the temporary broad pass rule; collecting screenshots and firewall log evidence.

## 9. Evidence to add

Save sanitized screenshots under `docs/day-01/evidence/` (create the folder when adding images):

- [ ] VirtualBox network adapters for pfSense and Kali
- [ ] pfSense Interfaces → OPT1 configuration (redact addresses)
- [ ] Kali interface and route output (redact addresses)
- [ ] Firewall → Rules → OPT1
- [ ] Successful ping output in both directions
- [ ] Windows `ipconfig` and the exact failed ping result (hide unrelated/private details)

Do not claim a screenshot or log exists until it has been captured.

## Public repository privacy

Private lab IPs have been replaced with placeholders in this document to reduce unnecessary exposure of internal topology. Do not publish public/WAN IP addresses, credentials, API keys, or screenshots containing identifying details. This edit does not remove previous values from Git history, forks, caches, or existing clones; if an actual public IP or secret was exposed, take appropriate additional remediation.
