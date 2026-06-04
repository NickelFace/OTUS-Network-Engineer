[🇷🇺 Русская версия](README.ru.md)

# Enterprise Network Design — Cisco IOS / EVE-NG

> Design and implementation of a multi-site corporate WAN connecting four offices — **Moscow (HQ), St. Petersburg, Chokurdakh, and Labytnangi** — with dynamic routing (OSPF, EIGRP, BGP), DMVPN/IPSec overlays, first-hop redundancy, and a full three-tier campus network. Built and validated in EVE-NG on Cisco IOS.

## Skills demonstrated

| Domain | Technologies |
|---|---|
| Routing | OSPFv2/v3 multi-area, EIGRP, eBGP & iBGP, route filtering, Policy-Based Routing |
| Switching | VLAN / VTP / DTP, STP, EtherChannel (LACP & PAgP) |
| High availability | HSRP, dual-uplink failover with IP SLA tracking |
| VPN / Overlay | DMVPN hub-and-spoke, GRE, IPSec encryption |
| Services | NAT/PAT, DHCP, NTP |
| Addressing | IPv4/IPv6 address-space planning and summarization |

## Diploma project — multi-site WAN

```
                     ┌──────────────┐         ┌────────────────────┐
   AS peering  ◄────►│   Provider   │◄───────►│   St. Petersburg   │
                     │   (Triada)   │  eBGP   │   eBGP / iBGP edge │
                     └──────┬───────┘         └────────────────────┘
                            │ eBGP
                     ┌──────┴────────┐
                     │   Moscow HQ   │  OSPF multi-area · HSRP · STP
                     │  (DMVPN hub)  │  PAT · DHCP · NTP
                     └──────┬────────┘
               DMVPN/IPSec  │  hub-and-spoke
             ┌──────────────┴───────────────┐
   ┌─────────┴──────────┐          ┌─────────┴──────────┐
   │     Chokurdakh     │          │     Labytnangi     │
   │  PBR, dual uplinks │          │       spoke        │
   └────────────────────┘          └────────────────────┘
```

| Decision | What was solved |
|---|---|
| IPv4/IPv6 address plan | Allocated /24–/30 ranges per site, management networks, and link-local addressing for the whole WAN |
| OSPF multi-area + filtering | Segmented the Moscow office into areas and filtered inter-area routes to keep routing tables lean |
| eBGP / iBGP between AS | Connected Moscow and St. Petersburg across autonomous systems; selected the preferred provider and load-balanced traffic |
| BGP path filtering | Restricted provider advertisements to a default route only — prevented the network becoming a transit AS |
| DMVPN over IPSec | Built an encrypted hub-and-spoke overlay (Moscow hub → Chokurdakh, Labytnangi spokes) over the public WAN |
| PBR + IP SLA | Distributed traffic across two uplinks at the Chokurdakh office with tracked failover |

## Labs

| # | Topic | Focus |
|---|-------|-------|
| 01 | [VLAN](VLAN/) | Extended VLAN, VTP/DTP, trunk links, extended-range VLANs |
| 02 | [STP](STP/) | Redundant switched topology, root bridge election, port cost/priority tuning |
| 03 | [EtherChannel](EtherChannel/) | EtherChannel with LACP and PAgP, VLAN trunking over aggregated links |
| 03 | [HSRP](EtherChannel/HSRP.md) | First-hop redundancy — active/standby, virtual IP, priority and preemption |
| 04 | [OSPF Basics](OSPF/) | OSPFv2 single-area — router IDs, passive interfaces, neighbor adjacencies, metric |
| 05 | [OSPF Multiarea](OSPF/Multiarea.md) | OSPFv2 multi-area — ABR, ASBR, inter-area summarization, MD5 authentication |
| 06 | [EIGRP Basic](EIGRP/) | EIGRP for IPv4 — neighbors, topology table, bandwidth tuning |
| 06 | [EIGRP Advanced](EIGRP/Advanced.md) | Auto-summary, redistribute static default, hello/hold timers, bandwidth percent |
| 07 | [IPv4 / IPv6 Addressing](Diplom/IPv4-IPv6.md) | WAN-wide address-space planning, VLANs, management networks |
| 08 | [PBR](Diplom/PBR.md) | Policy-Based Routing with IP SLA tracking across two uplinks (Chokurdakh) |
| 09 | [OSPF Filtering](Diplom/OSPF-Filter.md) | Area segmentation and inter-area route filtering (Moscow) |
| 10 | [OSPFv3](Diplom/OSPFv3.md) | OSPFv3 for IPv6 — metrics, timers and filters preserved from OSPFv2 |
| 11 | [BGP Basics](Diplom/BGP.md) | eBGP between autonomous systems (Moscow ↔ St. Petersburg) |
| 12 | [iBGP](Diplom/IBGP.md) | iBGP in Moscow and the Triada provider — preferred provider, load balancing |
| 13 | [BGP Filtering](Diplom/BGP-Path.md) | Restricting provider advertisements to default route only |
| 14 | [PAT, DHCP, NTP](Diplom/PAT-DHCP-NTP.md) | NAT/PAT, DHCP server, NTP synchronization across all offices |
| 15 | [DMVPN](Diplom/DMVPN.md) | GRE tunnels and DMVPN hub-and-spoke (Moscow, Chokurdakh, Labytnangi) |
| 16 | [IPSec over DMVPN](Diplom/DMVPN-IPSec.md) | GRE over IPSec — encrypted DMVPN overlay |
| 17 | [Project](PROJECT/) | Three-tier campus network — OSPF, HSRP, STP, DHCP, PAT, L2 security |
| — | [Self-Study](SelfLearn/) | BGP, GRE tunnels, IS-IS, provider connectivity scenarios |

## Background

I work as a network engineer with production experience on **MikroTik RouterOS**, and this project extends that into enterprise **Cisco IOS** design — a combination that maps directly onto real ISP and corporate edge environments. I'm currently preparing for **CCNA 200-301**.

I also codify infrastructure as code — see [homelab-infrastructure](https://github.com/NickelFace/homelab-infrastructure) for Terraform/Ansible/Docker managing a live multi-service lab.

## Environment

- Simulator: [EVE-NG](https://www.eve-ng.net/)
- Platform: Cisco IOS
- Origin: built during the [OTUS Network Engineer](https://otus.ru/lessons/network-engineer/) program
