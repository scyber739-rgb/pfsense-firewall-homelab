# Day 5 — NAT: WAN Port Forwarding to an Internal Server

**Status: Initial port-forward test was attempted. The HTTP response and Ubuntu request log are useful evidence, but the external and internal port values must be checked against the saved pfSense rule before calling the mapping fully verified.**

## Goal

Today I worked on Destination NAT (DNAT), also called port forwarding. The goal was to send a connection arriving at the pfSense WAN address on a selected TCP port to a web service running on the internal Ubuntu test machine. This is a controlled VirtualBox homelab exercise, not a recommendation to expose a lab server to the public internet.

## What port forwarding does

```text
Test client
    | TCP to <PFSENSE_WAN_IP>:<EXTERNAL_PORT>
    v
pfSense WAN
    | Destination NAT / port translation
    v
Ubuntu server: <UBUNTU_IP>:<INTERNAL_SERVICE_PORT>
```

pfSense changes the packet destination from its WAN address and external port to the Ubuntu server address and internal service port. The firewall filter policy must also allow the forwarded connection.

- **NAT** decides how an address and/or port is translated.
- **Firewall rules** decide whether traffic is allowed.
- A port forward does not mean every inbound connection is allowed.

## Configuration workflow

Open **Firewall → NAT → Port Forward**. The test rule was configured with these values:

| Field | Test setting |
|---|---|
| Interface | WAN |
| Protocol | TCP |
| Source | Any (temporary lab test scope; restrict this in a real deployment) |
| Destination | WAN address |
| Destination port | External port 18 was the value reported during setup |
| Redirect target IP | Ubuntu test server (`<UBUNTU_IP>`) |
| Redirect target port | Verify the saved rule; do not assume it from the external port |
| Description | Day 5 — WAN to Ubuntu test service |

Save the rule and apply changes. Check that the intended associated filter rule is present on the WAN rules tab. Do not add a broad WAN allow rule.

## Test and observation

The test was initiated from the authorized Kali lab client. The command reported during the test was:

```bash
curl -v --connect-timeout 5 http://<PFSENSE_WAN_IP>:18/
```

The captured output showed an HTTP connection and a **301 Moved Permanently** response. The Ubuntu terminal also showed a request arriving from the Kali test client.

A 301 response means an HTTP server answered and asked the client to use a different URL. It is not the same as receiving a normal 200 OK page, but it does show that an HTTP response was returned. The Ubuntu request log is additional evidence that the request reached the server.

### Port-number check before marking this complete

The captured test details contain a mismatch: the command was described as using external port 18, while the pasted curl output reported a connection to port 80. The Ubuntu service was also discussed using port 8080 earlier. These values cannot safely be treated as interchangeable.

Before marking the mapping fully verified, check all three values:

1. External/WAN destination port in the port-forward rule.
2. Redirect target port in the NAT rule.
3. Actual Ubuntu listening port.

On Ubuntu, run:

```bash
sudo ss -lntp
```

Compare the output with **Firewall → NAT → Port Forward**. Use the exact external port in the Kali curl URL. Do not claim a specific internal port until these values match.

## Evidence checklist

- [ ] Screenshot of the pfSense Port Forward rule, with private addresses redacted.
- [ ] Screenshot of the associated WAN filter rule.
- [ ] Kali terminal showing the exact curl command and complete output.
- [ ] Ubuntu terminal showing the test request in the server log.
- [ ] Ubuntu `ss -lntp` output confirming the listening port.
- [ ] Optional matching event from **Status → System Logs → Firewall**.
- [ ] Confirmed mapping: external port → Ubuntu IP → internal service port.

Do not publish raw configuration exports, credentials, public IPs or unredacted private lab details.

## Troubleshooting

1. Confirm the test client is actually reaching the pfSense WAN interface. Check VirtualBox network mode and routing.
2. Confirm the Port Forward rule uses the correct external port and redirect target.
3. Confirm the associated WAN firewall rule is present and enabled.
4. Confirm Ubuntu is listening on the redirect target port.
5. Confirm Ubuntu's host firewall allows that port.
6. Check whether the Ubuntu log shows the incoming request.

A timeout alone does not prove pfSense blocked the traffic. Confirm the matching firewall log and server-side evidence.

## Security notes

- Source **Any** was used only as a temporary lab setting. Restrict the source to known trusted addresses whenever practical.
- Forward only the required TCP port to the required server.
- Do not expose pfSense administration or unrelated internal services through WAN port forwards.
- Keep the test environment isolated and remove the temporary rule when the exercise is finished.
- A successful port forward is not a substitute for patching, authentication, TLS and application-level security.

## What I learned

Port forwarding lets an outside client reach one selected internal service through the firewall's WAN address. pfSense translates the destination, while the firewall policy controls whether the connection is allowed. The server-side log helps confirm whether the request actually arrived at Ubuntu.

**Next action:** verify the external port, redirect target port and Ubuntu listening port against one another, then capture the remaining evidence before treating Day 5 as fully verified.