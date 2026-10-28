# pfSense Firewall Homelab

This is my pfSense firewall lab that I am building in VirtualBox. I am using pfSense, Kali Linux and Ubuntu to learn firewall rules, routing and basic network troubleshooting.

## What I want to learn

- Set up WAN, LAN and OPT1 in pfSense
- Connect Kali and Ubuntu to different networks
- Understand how traffic moves through the firewall
- Test ping and check why traffic is blocked
- Learn to write safer firewall rules

## My lab setup

```text
                 VirtualBox NAT
                       |
                    pfSense
                  /         \
                LAN         OPT1
                 |            |
              Ubuntu         Kali
```

This is a simple view of the setup. The actual adapter names should be checked in VirtualBox and pfSense if I change anything.

## IP address notes

I have hidden the lab IP addresses in this public repo. Placeholders like `<KALI_IP>` and `<OPT1_GATEWAY>` are examples only. I need to replace them with the actual values from my own lab before running commands.

| Device | Role |
|---|---|
| pfSense WAN | Connects to VirtualBox NAT |
| pfSense LAN | Ubuntu side |
| pfSense OPT1 | Kali side |
| Ubuntu | Internal test machine |
| Kali Linux | Test machine on OPT1 |

## Day 1

- [Day 1 notes](docs/day-01/README.md)
- [Network diagram](docs/day-01/network-architecture.md)

## What worked so far

- Kali could ping the pfSense OPT1 gateway after I added a firewall rule.
- Kali and Ubuntu could ping each other.
- Windows-to-Kali connectivity is still not working, so I need to troubleshoot it separately.

These results are based on the tests I ran/reported. I will add screenshots after I capture and clean them up.

## Things I still need to do

- [x] Set up the basic interfaces
- [x] Test basic connectivity between Kali and Ubuntu
- [ ] Check why Windows cannot reach Kali
- [ ] Review the temporary OPT1 rule and make it more specific
- [ ] Add screenshots and firewall log evidence
- [ ] Continue with rule order, logging and NAT

## Safety notes

I am using this setup for my own lab only. The OPT1 rule used during troubleshooting had destination set to `Any`, which is too broad to leave as a final rule. I need to limit it to only the traffic required for each test.

I have replaced IPs in these files, but that does not remove older values from Git history or existing clones. I should also avoid uploading passwords, API keys or screenshots with private network details.
