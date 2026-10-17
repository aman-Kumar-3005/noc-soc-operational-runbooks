# WAN branch outage: L1/L2 triage and RCA

## Entry conditions

Use for a branch-down alarm, loss of a WAN circuit, repeated flaps or failure to reach a business application. Confirm the branch identifier, affected service, circuit IDs, primary/backup design and escalation matrix from the inventory. Do not infer a carrier fault from one failed ping.

## 1. Establish impact and a timeline

1. Acknowledge the alarm and open or link the incident. Record first observed impact, detection time, acknowledgement time and timezone; mark estimates explicitly.
2. In HP NNMi, inspect the node/interface event and topology dependencies. In Zabbix, compare trigger history with the underlying item timestamps and latest data. In Prognosis, check the relevant service or performance view configured for this estate. View names and available metrics depend on the deployment.
3. Check poller/collector health and maintenance windows. Multiple unrelated sites losing telemetry simultaneously may indicate a monitoring-path problem.
4. Confirm business impact with branch staff: all users or one device, all services or one application, primary and backup status, power and CPE LEDs. Request observations, not an unsupervised reboot.
5. Apply the service priority matrix. For a P1/P2 bridge, assign incident commander, technical lead, scribe and communications owner. Agree the next update time.

## 2. Isolate the fault domain

| Observation | Next check | Evidence to retain |
| --- | --- | --- |
| All local equipment unavailable | Branch power, UPS, local access and out-of-band reachability | Staff confirmation and last-seen times |
| WAN port down | Administrative state, carrier, CPE/ONT alarms, optics if supported | Interface state, last flap and counter deltas |
| Port up, gateway unreachable | VLAN/subinterface, ARP, addressing and provider handoff | Correct source/VRF, neighbor entry and bounded probe |
| Gateway reachable, remote prefix absent | BGP/OSPF peer state and route selection | Neighbor state, uptime and route table |
| Route present, application fails | Return path, VPN, firewall policy, DNS and service endpoint | Five-tuple, NAT mapping and application symptom |
| Backup restored service | Verify actual traffic path and capacity | Route/flow evidence, utilization and user confirmation |

Compare counters over a recorded interval; a historical error count is not proof of a current fault. Correlate interface changes with routing events and service impact before escalating.

## 3. Junos routing checks

Run only on the relevant device and routing instance. Replace the example interface and documentation address.

```text
show interfaces terse
show interfaces ge-0/0/0 extensive
show chassis alarms
show route
show route 192.0.2.10
show bgp summary
show bgp neighbor
show ospf neighbor
show ospf interface
```

For a VRF, use the appropriate table, for example `show route table BRANCH.inet.0`. The table name is an example, not a default.

- BGP: check the expected peer, uptime, prefix counts and last error. `Active` means a connection attempt, not a healthy established session. A stable session with missing routes can still indicate policy or advertisement trouble.
- OSPF: compare state with the intended topology. Full is expected for required adjacencies; 2Way between DROther routers on a broadcast segment can be normal. Persistent ExStart/Exchange needs L2 review of MTU and adjacency parameters. Do not alter timers or authentication during L1 triage.
- Look for correlated changes and route preference. Confirm that backup traffic uses the expected path, rather than assuming a reachable management address proves failover.

Do not clear peers, bounce ports or reboot devices simply to make an alarm disappear. Preserve pre-change evidence and use the approved change or incident procedure.

## 4. ISP escalation template

```text
Subject: [Priority] [Branch ID] [Circuit ID] - connectivity loss
Internal incident / carrier ticket:
Impact and affected service:
First observed impact / timezone:
Primary / backup circuit state:
Handoff device and interface:
Power / local equipment checks:
Interface state, last flap, recent error deltas:
Gateway / routing observations and probe source:
Relevant timestamps and sanitized evidence attachments:
Recent approved changes:
Requested action: check circuit/handoff and provide fault location and ETA
Branch access contact / access window:
Bridge contact / next update due:
```

Send only the information needed by the carrier. Keep customer traffic, credentials, internal topology and unrelated logs in the internal case. Record the carrier ticket, owner and promised update time; an ETA is not a confirmed restoration time.

## 5. Restore and validate

Verify interface stability, expected neighbors and routes, and application transactions through the authorized service owner. Check both directions where possible. Watch for recurring flaps over the environment's agreed observation window and confirm monitoring has received fresh samples. Close the bridge only when the incident owner accepts the restoration evidence.

If service runs on backup, record the degraded resilience and keep the primary repair task open. Do not close corrective work merely because users can reconnect.

## 6. Duration, SLA and RCA

Track impact start, detection, acknowledgement, escalation, restoration and final resolution separately. For an individual incident, restoration duration is restoration time minus impact start; acknowledgement delay is acknowledgement minus detection. MTTR is an aggregate mean over a defined incident set, and the team must state whether it means restore, repair or resolve.

Apply the contracted SLA calendar and exclusions. Keep elapsed time and SLA-counted time separate; record approved pauses and their evidence. Do not invent a pause because a carrier is investigating.

RCA record:

- Impact: affected services, branches, users and measured duration.
- Timeline: timestamp, observation/action, owner and evidence link.
- Confirmed cause: mechanism supported by evidence; mark unresolved hypotheses.
- Contributing conditions: backup capacity, alerting, configuration or escalation gaps.
- Restoration: action taken and validation result.
- Prevention: named owner, due date and a measurable completion check for each action.

## References

- [Juniper: show bgp summary](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/command/show-bgp-summary.html)
- [Juniper: show bgp neighbor](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/command/show-bgp-neighbor.html)
- [Juniper: show OSPF neighbor](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/command/show-ospf-ospf3-neighbor.html)
