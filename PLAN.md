# vBNG Router — Master Plan (VPP-Based, No Dynamic Routing)

> Goal: Build a robust, production-ready vBNG focused on L2/L3, IPoE/PPPoE subscriber termination, AAA, CGNAT, QoS, Security, VRRP HA, and Accounting — **without** dynamic routing (use static/VRF + ECMP/UCMP only).

---

## 1) Architecture at a Glance

**Data Plane (VPP):**

* L2: 802.1Q / QinQ sub-interfaces, Bridge Domains/XConnect, LACP via bonding.
* L3: VRFs, IP unnumbered, ARP/Proxy-ARP, IPv6 NDP/SLAAC, DHCPv4/6 Relay, ICMP/ICMPv6.
* PBR: policy-based routing per VRF / per-sub.
* Multipath: ECMP/UCMP.
* CGNAT: NAT44 (Deterministic, EIM, Hairpin), ALGs (ICMP, PPTP), NAT event logs (IPFIX/Netflow v9).
* QoS: policers/shapers, WRR; hook points via feature arcs.
* Security: ACL (stateless), conn-track assisted stateful where practical.
* HA: VRRP (native plugin if available) else keepalived integration.
* Accounting: IPFIX flow export + NAT event export.

**Control Plane (Microservices):**

* Subscriber Manager (SM): session lifecycle, per-sub VRF/QoS/ACL, leases, counters.
* AAA Agent: RADIUS auth+acct (PAP/CHAP), CoA handler; policy translation.
* IPoE Agent: DHCP snooping/relay correlation → SM.
* PPPoE Agent: Discovery/LCP/IPCP/PAP/CHAP; integrate **Accel-PPPoE** initially via memif/tap; roadmap to native PPP FSM plugin.
* NAT Log Exporter: translate VPP NAT events → collectors.
* Configd: typed config (YAML/JSON), validation, transactional apply (idempotent), audit.
* Telemetry: VPP Stats API + Prometheus exporter; IPFIX collector optional for self-tests.

**Management Plane:**

* gRPC/REST API; RBAC; audit log; backup/restore; schema versioning.
* CLI/UX: your preferred shell style (Cisco-like) backed by the API.

---

## 2) What VPP Gives vs What We Build

| Area                                      | VPP Built-in     | Our Work (Glue or New)                                       |
| ----------------------------------------- | ---------------- | ------------------------------------------------------------ |
| VLAN/QinQ, Bridge/XConnect                | ✅                | Profiles, auto-provisioning, tests                           |
| LACP (bond)                               | ✅ (bond driver)  | Orchestrate, monitoring, failover policy                     |
| VRF, IP unnumbered                        | ✅                | Per-sub templates; API; validation                           |
| ARP/Proxy-ARP, ICMP                       | ✅                | Scale tuning, DoS guards                                     |
| IPv6 NDP/SLAAC/RA                         | ✅                | NDP proxy where needed; policy                               |
| DHCPv4/6 Relay                            | ✅ plugins        | Lease/SM correlation; Option82; logging                      |
| PBR                                       | ✅ (pbr/classify) | Per-sub rules & compiler                                     |
| ECMP/UCMP                                 | ✅                | Health-driven weights; probes                                |
| NAT44 (EIM, Deterministic, hairpin, ALGs) | ✅ nat44 plugins  | Policy engine; pools; IPFIX/NFV9 export                      |
| ACL (stateless)                           | ✅                | Large IP sets management                                     |
| Stateful filtering                        | ⚠️ partial/ct    | Build simple conntrack-based policy using NAT/session tables |
| QoS (policer/shaper hooks)                | ✅ primitives     | H-QoS policy compiler, WRR profiles, attach points           |
| VRRP                                      | ⚠️ plugin varies | Fallback: keepalived + netns/memif integration               |
| IPFIX exporter                            | ✅                | Schemas, rate control, collectors                            |
| PPPoE termination                         | ⚠️ basic I/O     | Integrate **Accel-PPPoE** first; plan native PPP FSM         |

---

## 3) Development Ground Rules

* **Reproducible builds:** pin VPP/DPDK; containerized dev toolchain; Makefile targets.
* **Idempotent config:** apply same config twice → no change.
* **Crash-safe:** any service restart must not drop dataplane unless intended.
* **Observability-first:** every feature lands with counters, logs, and a quick probe.
* **Testable artifacts:** each feature ships with a minimal lab recipe.

**Repo Layout (suggested):**

```
/infra        # Ansible/Terraform, lab wiring
/vpp          # VPP plugins, API shims, feature arcs
/ctrl         # Go/Rust microservices (SM, AAA, IPoE, PPPoE, NAT-logger)
/api          # Protobuf/OpenAPI specs, codegen
/cli          # Text CLI → API bridge
/ui           # (optional) Web admin
/etc          # Reference configs, YAML schemas
/tests        # TRex, dhclient, rp-pppoe, pcap scripts
/tools        # helpers: pkt-gen, tap/memif wiring
```

---

## 4) Milestones (dependency-aware; not “phases”)

* **M0: Bootstrap** — build/run VPP, configd skeleton, stats exporter.
* **M1: L2 Core** — VLAN/QinQ, LACP, bridge/xconnect, IP unnumbered.
* **M2: IPv4/IPv6 Core** — ARP/NDP, DHCPv4/6 relay, SLAAC, RA policies.
* **M3: VRF+PBR+ECMP** — multi-table, policy compiler, health weighting.
* **M4: IPoE BNG** — session model, RADIUS auth/acct, CoA, per-sub VRF.
* **M5: PPPoE BNG (Accel-PPPoE)** — memif/tap I/O, session sync, RADIUS.
* **M6: CGNAT** — NAT44 det/eim, pools, logging (IPFIX/NFV9), hairpin.
* **M7: QoS** — policer/shaper compiler, WRR attach, per-sub profiles.
* **M8: Security** — ACL compiler, basic stateful using conn-track.
* **M9: HA** — VRRP (native or keepalived), state sync minimal.
* **M10: Accounting/Obs** — IPFIX flows, dashboards, alarms.

Each milestone should be shippable and demoable.

---

## 5) Interfaces & Schemas (essentials)

**Config Schema (YAML, excerpt):**

```yaml
system:
  workers: 4
  hugepages: 4096
l2:
  bonds:
    - name: bond0
      mode: lacp
      members: [TenGig0/0/0, TenGig0/0/1]
  subifs:
    - if: bond0
      type: qinq
      svid: 200
      cvid: 300
l3:
  vrfs:
    - id: 10
      name: subs
  svi:
    - if: bond0.200.300
      vrf: 10
      unnumbered: loop0
ip4:
  dhcp_relay:
    servers: [192.0.2.10]
  proxy_arp: true
ip6:
  ra:
    mode: managed
  dhcpv6_relay:
    servers: [2001:db8::10]
subs:
  ipoe:
    option82: true
  pppoe:
    pado_delay_ms: 100
    padi_rate_limit: 2000
aaa:
  radius:
    servers:
      - host: 198.51.100.5
        secret: mysecret
nat44:
  mode: deterministic
  pools:
    - 203.0.113.0/24
qos:
  profiles:
    - name: basic50
      cir: 50mbit
      pir: 60mbit
security:
  acls:
    - name: allow-dns
      rules:
        - src: any; dst: any; proto: udp; dport: 53; action: permit
ha:
  vrrp:
    groups:
      - vrid: 10
        iface: bond0.200
        vip: 198.51.100.1/24
```

**Core Northbound API (excerpt):**

* `POST /config/apply` — typed YAML, returns diff + result.
* `POST /subs/{id}/coa` — push CoA attrs; returns applied policy.
* `GET /vpp/stats` — selected counters.
* `GET /nat/events` — stream (IPFIX/NFV9) → gRPC.

---

## 6) Test & Validation Playbook

* **Traffic:** TRex profile for sub-count & pps; iperf for throughput.
* **IPoE:** dhclient behind VLAN; verify lease → SM.
* **PPPoE:** rp-pppoe/accel-ppp client; LCP/IPCP, auth via RADIUS.
* **NAT:** outbound many-to-one; hairpin; deterministic mapping.
* **QoS:** meter/shaper conformance; WRR fairness; latency under load.
* **Security:** ACL hit-counters; basic stateful (SYN/FIN/RST cases).
* **HA:** VRRP failover < 1s; session continuity policy.

---

## 7) Risks & Mitigations

* **PPPoE native FSM is heavy:** start with Accel-PPPoE; plan native later.
* **Stateful firewall complexity:** begin with NAT conn-track signals; expand.
* **QoS H-QoS limits:** implement practical 2-level hierarchy first.
* **VRRP plugin availability:** fallback to keepalived (netns+memif) if absent.

---
