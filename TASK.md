
# task.md — Execution Backlog (Qoder-Ready)

> Rule: Every task has a "Qoder Prompt" you can paste into Qoder. Keep focus narrow.

## M0 Bootstrap

1. **VPP Build & Run Skeleton**

   * DoD: `vppctl show ver` OK; workers pinned; hugepages on; stats export reachable.
   * Qoder Prompt:

```
Setup a reproducible VPP build/run on Debian 12 with DPDK. Pin versions, enable hugepages (1G/2M), set 4 workers pinned to NUMA0. Output: Makefile targets (build, run, clean), a sample startup.conf enabling stats API and cli socket, and a bash script that binds a 25G NIC to vfio-pci safely.
```

2. **Configd Skeleton**

   * DoD: `POST /config/apply` loads YAML, writes VPP config via binary API; idempotent.
   * Qoder Prompt:

```
Create a Go microservice called configd with one endpoint POST /config/apply. It takes typed YAML (provide schema) and applies VLAN subifs + VRF assignments to VPP via govpp. Include validation, dry-run diff, and rollback on failure.
```

## M1 L2 Core

3. **VLAN/QinQ Provisioner**

   * DoD: create/delete 1000s of subifs; persists across restart.
   * Qoder Prompt:

```
Implement a VLAN/QinQ provisioner in configd: create sub-interfaces (802.1Q and 802.1ad), attach to bridge domains or VRFs, and bring up state. Provide govpp code and VPP CLI parity examples. Include idempotent reconciliation.
```

4. **LACP Bonding Orchestrator**

   * DoD: bond0 LACP across 2 ports, health monitoring, fast failover.
   * Qoder Prompt:

```
Add LACP bonding support using VPP bond interface API. Implement member add/remove, LACP mode, and monitoring loop. Expose /bonds in the API. Provide a sample config and a health probe.
```

## M2 IPv4/IPv6 Core

5. **IPv4 ARP/Proxy-ARP + DHCP Relay**

   * DoD: DHCPv4 relay to servers; Option82 insertion; per-SVI proxy-ARP.
   * Qoder Prompt:

```
Configure VPP DHCPv4 relay with Option82 and per-interface server lists. Enable proxy-ARP per SVI and provide flood controls. Expose metrics for leases/sec and ARP cache. Provide tests with dhclient.
```

6. **IPv6 NDP/SLAAC + DHCPv6 Relay**

   * DoD: Clients get GUA/PD; NDP cache controlled; RA policies.
   * Qoder Prompt:

```
Enable IPv6 RA/SLAAC and DHCPv6 relay in VPP. Implement NDP guard and configurable RA lifetimes. Add metrics and a test using a Linux client that obtains GUA+PD.
```

## M3 VRF/PBR/ECMP

7. **VRFs + Policy Compiler**

   * DoD: multiple tables, per-sub routing domains.
   * Qoder Prompt:

```
Implement VRF creation and per-interface VRF assignment. Build a tiny policy compiler that translates YAML PBR rules into VPP classify tables. Include unit tests.
```

8. **ECMP/UCMP Manager**

   * DoD: add multipath static routes with weights; health-based weight.
   * Qoder Prompt:

```
Create an ECMP manager that programs multipath routes with weights in VPP. Provide a small health probe (ICMP) to adjust weights dynamically within limits. No BGP/OSPF.
```

## M4 IPoE BNG

9. **Subscriber Manager (SM)**

   * DoD: in-memory sub DB; CRUD; counters; per-sub VRF/QoS/ACL hooks.
   * Qoder Prompt:

```
Build a Go service 'subs-manager' that tracks IPoE subscribers (MAC,VLAN,IP,VRF). Expose gRPC to set per-sub VRF, QoS profile, ACL. Store counters. Provide a watcher that listens to DHCP relay events and auto-creates sessions.
```

10. **RADIUS AAA (Auth/Acct, CoA)**

* DoD: PAP/CHAP; start/stop/interim; CoA → live policy update.
* Qoder Prompt:

```
Implement a RADIUS client library with auth+acct and CoA support. Map attributes (rate-limit, ACLs, VRF) to internal policy. Provide a CoA server and apply changes to SM and VPP live.
```

## M5 PPPoE BNG (Accel-PPPoE first)

11. **PPPoE Integration via memif/tap**

* DoD: working PPPoE termination; session sync to SM.
* Qoder Prompt:

```
Integrate accel-ppp with VPP via memif. Provide systemd units, memif wiring, and a translator that syncs sessions (LCP/IPCP, IPs, DNS) into subs-manager. RADIUS auth/acct path shared.
```

12. **Native PPP FSM (Roadmap)**

* DoD: prototype LCP/IPCP FSM; not in critical path.
* Qoder Prompt:

```
Design a VPP plugin skeleton for PPPoE with LCP/IPCP FSM hooks and a binary API. No full implementation; just compile, load, and demo state transitions with unit tests.
```

## M6 CGNAT

13. **NAT44 Policy Engine**

* DoD: deterministic/dynamic pools; per-VRF NAT instances; hairpin.
* Qoder Prompt:

```
Program NAT44 (deterministic and dynamic) in VPP with per-VRF instances. Implement pool management, EIM, and hairpin. Provide YAML to define pools and which interfaces are inside/outside.
```

14. **NAT Event Export (IPFIX/NFV9)**

* DoD: exporter with rate control; schema docs.
* Qoder Prompt:

```
Consume VPP NAT event stream and export IPFIX/NetFlow v9 to a collector. Implement buffering and rate limiting. Provide a small collector for tests.
```

## M7 QoS

15. **QoS Profile Compiler**

* DoD: translate profiles (CIR/PIR/burst) → policers/shapers.
* Qoder Prompt:

```
Implement a QoS compiler that maps YAML profiles (CIR,PIR,burst) into VPP policers/shapers and attaches them per sub-interface or session. Include WRR queueing where available.
```

## M8 Security

16. **ACL Orchestrator (Stateless)**

* DoD: large IP sets; idempotent; counters.
* Qoder Prompt:

```
Build an ACL manager that compiles large stateless rulesets into VPP ACLs, manages lifecycles, and exposes hit counters. Provide delta-apply to avoid churn.
```

17. **Basic Stateful (Conn-Track Assist)**

* DoD: TCP state tracking via conn/NAT tables for allow-related.
* Qoder Prompt:

```
Implement a simple stateful filter layer that leverages VPP conn/NAT tables to allow ESTABLISHED/RELATED and block INVALID. Expose metrics for states.
```

## M9 HA

18. **VRRP Gateway**

* DoD: VRRP VIPs for SVIs; failover < 1s.
* Qoder Prompt:

```
Implement VRRP for SVI gateways. Prefer the VPP VRRP plugin; if unavailable, integrate keepalived in a netns with a veth/memif to VPP. Provide failover test script.
```

19. **Minimal State Sync**

* DoD: replicate sub state & NAT pool cursor between peers.
* Qoder Prompt:

```
Build a lightweight state sync that mirrors subscriber session metadata and NAT pool cursors between two nodes over gRPC. On failover, restore policies quickly.
```

## M10 Accounting & Observability

20. **IPFIX Flow Accounting**

* DoD: per-sub flow export; tested with collector.
* Qoder Prompt:

```
Enable IPFIX flow export per VRF and tag records with subscriber ID. Provide a sample dashboard and a replayable pcap test.
```

21. **Prometheus Exporter**

* DoD: VPP stats, SM counters, AAA, NAT, QoS metrics.
* Qoder Prompt:

```
Implement a Prometheus exporter that scrapes VPP stats API and your services (SM, AAA, NAT logger). Provide example alert rules.
```

---

## Definitions of Done (Project)

* Cold start → full config applied in < 60s.
* 10k subscribers attach/detach without crash.
* NAT throughput ≥ NIC line rate (per target), with logging enabled.
* QoS conforms to CIR/PIR within 5% error under load.
* VRRP failover < 1s; session policies re-applied.
* Telemetry coverage: 90% of modules with counters and alerts.

---

## Quick Lab Topology (Reference)

* **Access**: switch with VLAN per subscriber (QinQ optional).
* **BNG**: VPP node with bond uplink to core, subif for each VLAN.
* **Core**: static routes with ECMP to upstream firewall/Internet.
* **AAA**: FreeRADIUS; **IPAM/DHCP**: ISC Kea/ISC DHCP; **Collector**: IPFIX/NFV9.

> This plan keeps dynamic routing out for now; you can safely add FRR later by swapping the static+ECMP manager.
