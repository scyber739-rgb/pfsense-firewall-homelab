# pfSense configuration register

Use this register to record only settings that have actually been entered into the pfSense lab. Keep real IP values in private notes and use role labels in this public repository.

## Firewall policy register

| Rule ID | Interface/direction | Action | Protocol | Source | Destination | Port | Business reason | Verification |
|---|---|---|---|---|---|---|---|---|

## Alias register

| Alias name | Type | Members | Used by rule | Verification |
|---|---|---|---|---|
| Pending | IP | Not configured/confirmed in pfSense yet | Planned TCP/80 policy | Pending |

## Lab roles

| Device | Public identifier | Purpose |
|---|---|---|
| Kali Linux | `<KALI_IP>` | Authorized test client |
| Ubuntu Linux | `<UBUNTU_IP>` | Internal test server |
| pfSense OPT1 | `<OPT1_GATEWAY>` | Kali-side gateway / ingress interface |
| pfSense LAN | `<LAN_GATEWAY>` | Ubuntu-side gateway |

## HTTP test record

| Test | Command / service | Result | Evidence |
|---|---|---|---|
| Baseline HTTP | `curl http://<UBUNTU_IP>:80/` | Pending | Terminal output required |
| TCP/80 Block | Kali → Ubuntu | Pending | Client output + matching firewall log |
| TCP/80 Pass | Kali → Ubuntu | Pending | HTTP response + state entry |

## NAT change record

| NAT ID | External interface/address:port | Internal destination:port | Associated filter rule | Test client | Observed result |
|---|---|---|---|---|---|

## Change-control notes
For each change, record the original rule, the modification, expected result, actual result, rollback action and evidence reference. Do not publish raw pfSense configuration exports containing credentials, private keys or sensitive identifiers.
