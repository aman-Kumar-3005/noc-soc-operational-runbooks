# NOC and SOC operational runbooks

Practical notes for branch connectivity incidents, SIEM alert triage and Junos diagnostics. The focus is on collecting evidence, narrowing the fault domain and handing an actionable case to the next team.

These are reference procedures and synthetic examples. They do not contain customer configurations, production logs or claims of incidents resolved. Adapt interface names, routing instances, severity rules and escalation contacts to the environment's approved SOP.

## Runbook index

| Reference | Use it for | Expected output |
| --- | --- | --- |
| [WAN branch outage and RCA](runbooks/wan-branch-outage-rca.md) | Branch unreachable, unstable circuit or partial application loss | Fault-domain assessment, ISP handover, incident timeline and RCA actions |
| [SOC L1 SIEM triage](runbooks/soc-l1-siem-triage.md) | Suspicious traffic, authentication alerts and phishing reports | Evidence-backed disposition or L2 escalation |
| [Junos and network diagnostics](cli-notes/junos-and-network-diagnostics.md) | Interface, route, neighbor, session and packet checks | Bounded diagnostic outputs with an interpretation |

## Working rules

- Record event time, collection time and timezone. Preserve raw evidence in the approved case system; redact credentials and personal data from shared summaries.
- Separate observation, hypothesis and confirmed cause. A green monitoring icon or an allowed firewall session does not establish application health.
- Start with read-only checks. Route changes, session resets, failover, host isolation and account disabling require the appropriate incident authority or pre-approved SOP.
- Use the service impact matrix for priority. A single circuit failure with healthy backup is different from loss of both paths serving a critical branch.
- Assign an owner and next-update time. Keep restoration and permanent corrective work as separate tracked outcomes.

## Repository use

```sh
git clone https://github.com/shreyasingh3005/noc-soc-operational-runbooks.git
cd noc-soc-operational-runbooks
```

Open the Markdown files locally or on GitHub. There are no scripts to install and no commands that should be run as a batch. CLI examples use documentation addresses and must be adapted to authorized targets.

## Evidence standard

A useful case includes the affected service and scope, timestamps, source of each observation, checks already completed, current hypothesis, missing evidence, owner and next action. Vendor references are linked in each document; command availability varies by platform and software release.
