# Day 4 — pfSense IP aliases and reusable policies

**Status: Documentation and test plan only. The alias previously entered in the separate OPNsense lab has not been confirmed as configured in this pfSense repository's lab.**

## Scenario
Several approved client hosts may need the same access policy. An IP alias groups host addresses so that one firewall rule can reference the group.

## What is an alias?
An alias is a named object containing IP addresses, networks, ports or URLs (depending on its type). The firewall rule still defines the action; the alias supplies reusable match criteria.

Example: an alias named `IT_TEAM_HOSTS` could contain the IP addresses of approved IT workstations. A rule can then use that alias as its source. Adding or removing a workstation from the alias updates the hosts covered by the rule.

## pfSense configuration workflow

1. Open **Firewall → Aliases**.
2. Select **IP** and click **Add**.
3. Enter a clear name, such as `IT_TEAM_HOSTS`.
4. Choose the appropriate type (Host(s) for individual addresses, or Network(s) for subnets).
5. Add only the intended lab host addresses and a useful description.
6. Save and Apply Changes.
7. Open **Firewall → Rules → OPT1** (or the actual ingress interface).
8. Create a narrowly scoped Pass rule.
9. Select the alias as Source, the Ubuntu test server as Destination, Protocol TCP and Destination port HTTP (80).
10. Enable logging, save and apply.

Do not assume that an alias permits traffic by itself. It only supplies the addresses used by a firewall rule.

## Intended policy

| Setting | Example |
|---|---|
| Interface | OPT1, if that is the ingress interface |
| Action | Pass |
| Address family | IPv4 |
| Protocol | TCP |
| Source | `IT_TEAM_HOSTS` alias |
| Destination | `<UBUNTU_IP>` |
| Destination port | HTTP / TCP 80 |
| Logging | Enabled for the test |
| Description | Allow approved IT hosts to Ubuntu HTTP |

Replace example names and addresses with the values actually configured in pfSense.

## Validation plan

From an approved Kali test client, run:

```bash
curl -v --connect-timeout 5 http://<UBUNTU_IP>:80/
```

Then verify all of the following:

- [ ] Alias exists under **Firewall → Aliases** with the intended members.
- [ ] OPT1 rule uses that alias as its source.
- [ ] HTTP request succeeds when the Pass rule and network path are correct.
- [ ] **Status → System Logs → Firewall** shows the matching TCP/80 event.
- [ ] **Diagnostics → States** shows the connection state.
- [ ] If a second authorized host is available, add it to the alias and verify that the same rule applies.

A timeout alone does not prove that the alias rule blocked traffic. Check routing, ingress interface, server listener, host firewall and the matching log entry.

## Evidence checklist

- [ ] Alias configuration screenshot with addresses redacted
- [ ] Rule screenshot showing alias source and TCP/80 destination
- [ ] Client curl output
- [ ] Matching pfSense firewall log
- [ ] State-table entry
- [ ] Optional second-host test

## Key lesson
Aliases make policies easier to maintain by separating reusable address groups from rule logic. The alias defines who matches; the firewall rule defines what happens to their traffic.
