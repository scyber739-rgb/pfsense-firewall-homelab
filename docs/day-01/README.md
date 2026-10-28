# Day 1 — pfSense setup and connectivity tests

## What I was trying to do

I wanted to set up the pfSense interfaces and get Kali and Ubuntu talking to each other through the firewall.

## My setup

- VirtualBox
- pfSense
- Kali Linux
- Ubuntu
- Windows host (Windows-to-Kali test is still pending)

### VirtualBox adapters I used

| VM | Adapter | Network | Why |
|---|---|---|---|
| pfSense | Adapter 1 | NAT | WAN |
| pfSense | Adapter 2 | Internal Network: `int network` | LAN |
| pfSense | Adapter 3 | Internal Network: `attacker` | OPT1 |
| Kali | Adapter 1 | Internal Network: `attacker` | Connect to OPT1 |

I should check these again if I change the VM settings.

## IP address setup

I removed the real IP addresses from this public version. The values below are placeholders, so I need to replace them with my actual lab values before running a command.

| Device | Address |
|---|---|
| pfSense OPT1 | `<OPT1_GATEWAY>/<PREFIX>` |
| Kali eth0 | `<KALI_IP>/<PREFIX>` |
| Ubuntu | `<UBUNTU_IP>` |
| pfSense LAN | `<LAN_GATEWAY>/<PREFIX>` |

OPT1 was configured with a static IPv4 address. IPv6 was set to None for this interface.

## What I did on Kali

First I checked the interface and route:

```bash
ip -br addr
ip -4 addr show dev eth0
ip route
```

I added the lab address to eth0 using this pattern:

```bash
sudo ip addr add <KALI_IP>/<PREFIX> dev eth0
```

I also changed the default route to use the pfSense OPT1 gateway:

```bash
sudo ip route replace default via <OPT1_GATEWAY> dev eth0
ip route
```

These commands change the current network settings and might not stay after reboot. Before removing an old address or route, I need to check the exact value first.

## The problem and what fixed it

At first, Kali could not ping the pfSense OPT1 gateway. I checked the OPT1 rules page and it was empty.

For troubleshooting, I added a pass rule with:
- Action: Pass
- Source: Kali host
- Destination: Any

After applying the rule, Kali could ping the OPT1 gateway. I then tested Kali and Ubuntu in both directions, and those ping tests worked too.

## Test results

| Test | Result |
|---|---|
| Kali to pfSense OPT1 before the rule | Failed |
| pfSense ping to Kali | Worked |
| Kali to pfSense OPT1 after the rule | Worked |
| Kali to Ubuntu | Worked |
| Ubuntu to Kali | Worked |
| Windows to Kali | Still not working |

## Important thing to fix

The rule with destination set to `Any` is broad. I only used it to help troubleshoot. I should change it later so it allows only the traffic needed for a test, and check the firewall logs under **Status → System Logs → Firewall**.

## Commands I used

| Command | What it shows/does |
|---|---|
| `ip -br addr` | Shows network interfaces and addresses |
| `ip route` | Shows routes and default gateway |
| `ping -c 4 <OPT1_GATEWAY>` | Tests Kali to the OPT1 gateway |
| `ping -c 4 <UBUNTU_IP>` | Tests Kali to Ubuntu |

Use the real lab values in place of placeholders before running these commands.

## Day 1 status

The basic interface setup and Kali/Ubuntu ping tests are working based on the tests I ran. Windows-to-Kali still needs checking. I also need to make the temporary firewall rule more specific and add screenshots/logs once I capture them.

## Screenshots to add later

- [ ] VirtualBox adapter settings
- [ ] pfSense OPT1 settings
- [ ] Kali IP and route output
- [ ] OPT1 firewall rule
- [ ] Ping test results
- [ ] Windows-to-Kali test

I have not added these screenshots yet. Before uploading, I need to hide IP addresses and other private details.

## Privacy

IP addresses are hidden in this public copy. This does not remove older values from Git history, forks or existing clones, so I should not upload any passwords, keys or private screenshots.
