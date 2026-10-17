# Junos and network diagnostics

Use these commands selectively in an authorized environment. Capture the device, software version, timestamp, routing instance, source address and purpose with each output. Names and addresses below are examples; `192.0.2.0/24`, `198.51.100.0/24` and `.test` domains are reserved for documentation.

## Junos operational mode

| Command | What to inspect |
| --- | --- |
| `show version` | Platform and release; confirm command support |
| `show system uptime` | Time and reboot context |
| `show interfaces terse` | Administrative/link state and logical addresses |
| `show interfaces ge-0/0/0 extensive` | Last flap, traffic and errors; compare counters over time |
| `show chassis alarms` | Active hardware/chassis alarms |
| `show system alarms` | Active system alarms |
| `show route` | Active route selection in the displayed tables |
| `show route 192.0.2.10` | Selected route covering the example destination |
| `show route table BRANCH.inet.0` | Routes in an example routing instance |
| `show arp no-resolve` | IPv4 neighbor resolution; scope to the relevant interface |
| `show bgp summary` | Peer state, uptime and route counts |
| `show bgp neighbor 192.0.2.1` | Specific peer detail and last error |
| `show ospf neighbor` | Adjacency state relative to the intended topology |
| `show ospf interface` | OSPF-enabled interface details |

BGP `Active` is not established. A route can be present while forwarding fails because of return-path, policy or next-hop problems. Do not interpret every OSPF 2Way neighbor as a fault: broadcast DROther relationships can remain 2Way normally.

## SRX firewall and VPN checks

These security commands apply to supported SRX flow-mode platforms and features, not every Junos switch or router. Check CLI `?` help for the running release and assigned privileges.

```text
show security flow session
show security flow session source-prefix 192.0.2.10/32
show security ike security-associations
show security ipsec security-associations
```

Prefer a source-filtered session lookup over dumping a busy session table. Match source/destination, ports, policy and traffic direction. An existing session does not establish that an application completed successfully. For VPNs, distinguish IKE establishment from IPsec SAs and actual bidirectional traffic.

Do not enable broad flow tracing, clear sessions, restart routing or alter policy as a first diagnostic step. Escalate with the relevant flow details and bounded observations.

## Linux host and path checks

Use an approved diagnostic host. `ip` and `ss` are generally supplied by iproute2; `mtr`, `traceroute`, `tcpdump` and `nslookup` may require separate packages. Install through the environment's normal package/change process.

```sh
# Local addressing, neighbors and selected route
ip -br address
ip route
ip route get 192.0.2.10
ip neigh
ss -tan

# Bounded probes to an authorized example target
ping -c 5 192.0.2.10
traceroute -n -m 15 -w 1 192.0.2.10
mtr -n -r -c 10 192.0.2.10

# Ask a specific example resolver
nslookup app.example.test 192.0.2.53
```

Loss at one intermediate hop may reflect ICMP rate limiting. Look for loss continuing to the destination, compare the return path where possible and confirm the application symptom. Traceroute probes may use a different protocol/path from the failing service. DNS success proves resolution, not service reachability; compare the expected answer and resolver.

## Bounded packet capture

Identify the real interface with `ip -br address`. The example `eth0` is not universal. Obtain capture authorization and select the narrowest useful host/port filter.

```sh
# Linux example: stop after 30 seconds or 200 packets
umask 077
sudo timeout 30 tcpdump -i eth0 -nn -s 128 -c 200 \
  -w /tmp/triage-example.pcap 'host 192.0.2.10 and tcp port 443'

# Read a saved capture without resolving names
sudo tcpdump -nn -r /tmp/triage-example.pcap
```

The snap length limits captured bytes per packet; it does not guarantee removal of sensitive content. Store captures in approved restricted storage and follow retention rules. The example path is temporary, not an evidence repository.

For TCP, correlate SYN, SYN-ACK, ACK, retransmissions and resets with capture location. A missing reply at one observation point is not enough to locate a drop. TLS payload is encrypted; a capture can show transport behavior without establishing application-level success. Bounded capture commands still consume resources, so avoid unfiltered captures on busy production devices.

## Handover checklist

Record the tested source/destination and ports, routing context, exact command, timestamps, output summary, current hypothesis and next owner. Include an evidence reference rather than pasting unrestricted full configuration or packet data into a public ticket.

## References

- [Juniper: BGP summary](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/command/show-bgp-summary.html)
- [Juniper: flow sessions by source prefix](https://www.juniper.net/documentation/us/en/software/junos/cli-reference/topics/ref/command/show-security-flow-session-source-prefix.html)
- [tcpdump manual](https://www.tcpdump.org/manpages/tcpdump.1.html)
