# Day 3 — Protocol and port filtering with HTTP

**Status: HTTP/TCP validation in pfSense is pending.** Basic Kali/Ubuntu ping connectivity has been reported in the existing lab notes, but that does not prove HTTP is reachable.

## Scenario
Ubuntu hosts a simple HTTP service. Kali is used as an authorized client. The goal is to test one service using TCP destination port 80 without blocking unrelated traffic.

## Lab roles

- pfSense: routes/filters traffic between the intended lab networks.
- Kali: test client, represented as `<KALI_IP>`.
- Ubuntu: HTTP server, represented as `<UBUNTU_IP>`.
- Service: HTTP over TCP destination port 80.

Use actual current lab addresses locally; do not publish private addresses.

## Step 1 — Start a safe test server on Ubuntu

Create/use a dedicated directory containing no personal files, then run:

```bash
mkdir -p ~/http-lab
cd ~/http-lab
echo "pfSense HTTP lab test" > index.html
sudo python3 -m http.server 80
```

Keep the server terminal open during testing.

## Step 2 — Check the baseline from Kali

```bash
curl -v --connect-timeout 5 http://<UBUNTU_IP>:80/
```

Record the full output. If the request fails, do not add a Block rule yet. First check the server listener, Kali route, Ubuntu route, host firewall, VirtualBox networks and whether the packets actually traverse pfSense.

## Step 3 — Configure a targeted pfSense rule

If the client and server are on different networks and the traffic crosses pfSense:

1. Open **Firewall → Rules → OPT1** (or the actual ingress interface for Kali-originated traffic).
2. Add a rule with Action **Block** for the restrictive test.
3. Set Address Family to IPv4 and Protocol to TCP.
4. Set Source to the Kali test host.
5. Set Destination to the Ubuntu HTTP server.
6. Set Destination port to HTTP (80).
7. Enable logging and add a descriptive label.
8. Save and Apply Changes.
9. Ensure the specific Block rule is above any broader Pass rule that could match the same traffic.

For the allow test, use a specific Pass rule with the same source, destination, protocol and port. Do not leave conflicting broad rules enabled.

## Step 4 — Retest and inspect evidence

Repeat the curl command from Kali. Then open **Status → System Logs → Firewall** and look for a matching TCP event with the expected source, destination and destination port. Check **Diagnostics → States** for an established connection when testing the Pass case.

## Expected vs actual

| Stage | Expected | pfSense lab record |
|---|---|---|
| Ubuntu HTTP listener | TCP/80 listening | Not yet recorded |
| Baseline curl | HTTP response | Pending |
| Specific TCP/80 Block | New HTTP connection denied | Pending |
| Matching pfSense log | TCP/80 Block event | Pending |
| Pass test/state | HTTP response and active state | Pending |

A ping success does not guarantee TCP/80 success. A timeout alone does not prove that the firewall caused the failure. Also, traffic between VMs on the same Layer-2 network may bypass pfSense.

## Evidence checklist

- [ ] Ubuntu terminal showing the HTTP server running
- [ ] Kali baseline curl output
- [ ] pfSense rule screenshot showing interface, action, source, destination, TCP and port 80
- [ ] Kali curl output after the rule change
- [ ] Matching firewall log entry
- [ ] State-table evidence for the successful Pass case

## Key lesson
Protocol and destination port narrow a policy to a particular service. Confirm the packet path and a successful baseline before interpreting a failed connection as a firewall block.
