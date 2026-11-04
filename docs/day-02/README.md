# Day 2 — pfSense PASS/BLOCK rules and rule order

**Status: Learning notes documented. Capture pfSense-specific rule screenshots and test results before marking the exercise complete.**

## Scenario
A company needs to allow approved traffic while denying unwanted connections. Overlapping policies can produce unexpected results if a broad pass rule is placed before a specific restriction.

## Rule fields

| Field | Meaning |
|---|---|
| Action | Pass, Block or Reject |
| Interface | Interface tab where the rule is evaluated |
| Address family | IPv4 or IPv6 |
| Protocol | TCP, UDP, ICMP or another supported protocol |
| Source | Originating host, network or alias |
| Destination | Target host, network or alias |
| Destination port | Service port on the destination |
| Log | Records matching traffic in the firewall log |
| Description | Human-readable purpose of the rule |

## Practical workflow

1. Open **Firewall → Rules**.
2. Select the interface on which the test traffic enters pfSense.
3. Add a rule and select Pass or Block.
4. Select the correct address family and protocol.
5. Specify the source and destination as narrowly as possible.
6. Set a destination port when testing a particular TCP/UDP service.
7. Enable logging for the test rule and add a clear description.
8. Save and apply changes.
9. Review the rule order and check for broader rules that could match first.
10. Test from the client and inspect **Status → System Logs → Firewall**.

## Rule-order exercise

Example: block one test host from reaching an HTTP server, while allowing other approved traffic. Place the specific Block rule above a broader Pass rule that would otherwise match the same packet.

| Test | Expected result | Evidence required |
|---|---|---|
| Packet matches specific Block | Denied | Failed client test plus matching Block log |
| Packet matches specific Pass | Allowed | Successful client test plus matching Pass log/state |
| No rule permits the packet | Default deny | Matching deny log |

The table describes the intended exercise, not a claim that each pfSense test has already been completed.

## Important concepts

- pfSense evaluates interface rules for traffic entering that interface.
- Rule order matters; an earlier matching rule can determine the result.
- Source and destination are different: source is the sender, destination is the target.
- Use the narrowest protocol, host/network and port needed.
- A timeout by itself does not prove that a particular rule blocked traffic.
- Existing states can affect retests. Inspect **Diagnostics → States** and clear only the relevant test state when appropriate.

## Evidence checklist

- [ ] OPT1 rules screenshot before and after reordering
- [ ] Specific Block rule and broader Pass rule
- [ ] Client command/output for each test
- [ ] Matching firewall log entries
- [ ] Explanation of which rule matched and why

## Key lesson
The correct action, interface, match criteria and rule order must all agree with the real packet path. Do not leave broad troubleshooting rules enabled as permanent policy.
