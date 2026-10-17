# SOC L1 SIEM alert triage

## 1. Validate the alert

Record case ID, rule/version, alert time, event time, timezone, detection window, affected asset/user and raw-event references. Check ingestion delay, parser health and duplicate alerts before treating a count as distinct activity. Identify asset criticality and whether the activity is ongoing.

Start with the rule window plus a small documented margin, then widen if the evidence warrants it. Do not use a fixed event threshold as universal proof of compromise. Compare with the account, host and service baseline.

## 2. Correlate identity and traffic

Use the asset inventory, DHCP/VPN records and authentication logs to map an IP address to a host or user at the event time. Shared NAT, proxies and reassigned addresses make IP-only attribution unreliable.

| Source | Useful fields | Interpretation |
| --- | --- | --- |
| Palo Alto traffic logs | Device/vsys, generated time, source/destination and NAT addresses/ports, rule, zones, application, session ID, action, bytes and session-end reason | Join related traffic/threat records using device, time and session context; session ID alone is insufficient |
| FortiGate forward traffic logs | Device/VDOM, eventtime, srcip/srcport, dstip/dstport, sessionid, policyid, action, translation fields, sentbyte/rcvdbyte | Preserve raw fields alongside normalized SIEM values; field names vary by release and log subtype |
| Identity/VPN/EDR | User, host, result, MFA outcome, source, session and process lineage where available | Establish which account and process were involved |

An allow/accept action means the policy permitted traffic. It does not establish that the destination or process was benign. A deny event does not prove every related attempt was blocked. Avoid double-counting session start, periodic and end records; inspect the collector's normalization and aggregation behavior.

## 3. Suspicious outbound traffic

1. Identify the originating host and user at the relevant time. Resolve NAT mappings and distinguish endpoint activity from a proxy or gateway.
2. Review destination, port, protocol/application, frequency, duration and byte direction. Compare against the host's role and normal destinations.
3. Correlate DNS/proxy records and available EDR process evidence. A domain lookup alone does not prove a connection; a connection alone does not prove data theft.
4. Check approved change records, scheduled jobs and known scanners. Reputation is supporting evidence, not a verdict; shared hosting and stale intelligence can mislead.
5. Escalate repeated unexplained callbacks, suspicious process lineage or material outbound transfer with a clear timeline and evidence. Apply blocking/isolation only under the authorized response procedure.

## 4. Brute-force and password-spray triage

Group failures by account, source and service. Inspect a sequence of failures followed by success, a single source targeting many accounts, and distributed sources targeting one account. Include VPN/IdP/MFA outcomes and lockout events where available.

Check for expired service credentials, scheduled tasks, known user retries and authorized testing. Determine whether a successful session occurred, then inspect its activity and device context. Geolocation or a high count alone is insufficient.

Escalate suspected successful compromise, privileged-account impact or active lateral activity promptly. Preserve relevant session and authentication evidence; credential reset, token revocation and disabling accounts belong to the approved response owner.

## 5. Phishing report

- Preserve the original message and headers in the approved evidence store. Record message ID, recipient, delivery time, subject and reported user actions.
- Compare display name, From, Reply-To, Return-Path and domains. Read Received headers with the organization's trusted mail boundary in mind; sender-supplied headers can be forged.
- Review Authentication-Results from a trusted mail system. SPF, DKIM and DMARC results need context; passing authentication does not make a message harmless.
- Inspect URL text versus the actual target, redirects, lookalike domains and attachment metadata. Defang indicators in tickets. Do not browse suspicious links on the analyst workstation or enable attachment macros.
- Use the approved sandbox and intelligence services. Do not upload sensitive customer emails, documents or token-bearing URLs to a public scanner.
- Ask whether the user clicked, downloaded, executed or entered credentials. Correlate mail, proxy/DNS, identity and endpoint records. Escalate credential entry or execution immediately under the response SOP.

## 6. Disposition and handover

| Disposition | Minimum rationale |
| --- | --- |
| True positive | Evidence supports the detected malicious or prohibited activity |
| Benign positive | The rule correctly matched activity that is authorized in this context |
| False positive | The rule/parser or matching logic incorrectly characterized the event |
| Inconclusive | Required evidence is missing or conflicting; state what is needed next |

Use the organization's definitions if its taxonomy differs. Do not close solely because the IP has no reputation hits or the user says the activity was expected.

```text
Case / rule / priority:
Impact and affected assets/users:
Event range and timezone / ingestion delay:
Observed sequence and raw-event references:
Identity/NAT correlation:
Checks completed and results:
Hypothesis / confidence / alternatives:
Disposition or reason for escalation:
Actions taken under SOP / approver:
Missing evidence / next action / owner / due time:
```

## Synthetic example

A demonstration alert shows login failures against several accounts from `198.51.100.25`, followed by a success for `lab-user` on `vpn.example.test`. This is a lead to investigate, not proof of compromise. Correlate the successful session, MFA result and endpoint identity; confirm whether an approved test explains it. Record unknowns instead of inventing missing logs.

## References

- [Palo Alto Networks: traffic log fields](https://docs.paloaltonetworks.com/ngfw/administration/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/traffic-log-fields)
- [Fortinet: log message fields, FortiOS 7.4.4](https://docs.fortinet.com/document/fortigate/7.4.4/fortios-log-message-reference/357866/log-message-fields)
- [CISA: phishing guidance](https://www.cisa.gov/resources-tools/resources/phishing-guidance-stopping-attack-cycle-phase-one)
