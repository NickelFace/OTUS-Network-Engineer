[🇷🇺 Русская версия](README.ru.md)

# Enterprise Network Design — Cisco IOS / EVE-NG

> End-to-end network infrastructure design for a multi-site corporate environment:  
> BGP inter-AS routing, OSPF multi-area, DMVPN + IPSec overlays, and L2 redundancy —  
> implemented and verified on Cisco IOS in EVE-NG.

This repository documents hands-on network engineering work covering the full stack from  
L2 switching to BGP policy. The diploma project models a real-world corporate WAN  
connecting Moscow HQ, St. Petersburg, Chokurdakh, and Labytnangi offices.

## Skills Demonstrated

| Domain | Technologies |
|---|---|
| Routing protocols | OSPFv2, OSPFv3, EIGRP, BGP (iBGP/eBGP), IS-IS |
| L2 switching | VLANs, STP, EtherChannel (LACP/PAgP), HSRP |
| WAN / VPN | DMVPN hub-and-spoke, GRE over IPSec, PBR with IP SLA |
| Network services | NAT/PAT, DHCP, NTP, MD5 authentication |
| Design patterns | Cisco three-tier hierarchy, area segmentation, route filtering |
| Addressing | IPv4/IPv6 dual-stack, address planning, summarization |

## Diploma Project — Multi-Site Corporate WAN

The capstone project connects four geographically distributed offices into a single  
routed domain. Key design decisions and their rationale are documented in each lab.

```
Moscow HQ ──eBGP── Triada ISP ──eBGP── St. Petersburg
    │                                        │
    └──────── DMVPN hub-and-spoke ───────────┘
                    │           │
               Chokurdakh   Labytnangi
```

| Lab | Topic | What was solved |
|---|---|---|
| [IPv4/IPv6 Addressing](Diplom/IPv4-IPv6.md) | Address planning | Designed /24–/30 allocation across all sites with IPv6 dual-stack |
| [OSPF Filtering](Diplom/OSPF-Filter.md) | Intra-domain routing | Area segmentation for Moscow; inter-area route filtering to limit LSA flooding |
| [OSPFv3](Diplom/OSPFv3.md) | IPv6 routing | OSPFv3 deployed with identical metrics and filters as OSPFv2 |
| [BGP Basics](Diplom/BGP.md) | Inter-AS connectivity | eBGP sessions between autonomous systems; Moscow ↔ St. Petersburg reachability |
| [iBGP](Diplom/IBGP.md) | Traffic engineering | iBGP mesh in Moscow and Triada; preferred-provider selection and load balancing |
| [BGP Filtering](Diplom/BGP-Path.md) | Route policy | Restricted provider advertisements to default route; prevented transit AS |
| [PBR](Diplom/PBR.md) | Policy routing | Dual-uplink traffic distribution at Chokurdakh with IP SLA failover tracking |
| [PAT / DHCP / NTP](Diplom/PAT-DHCP-NTP.md) | Network services | NAT/PAT for internet access; centralised DHCP and NTP for all offices |
| [DMVPN](Diplom/DMVPN.md) | Overlay WAN | GRE tunnel Moscow–SPb; DMVPN phase 2 hub-and-spoke for remote offices |
| [IPSec over DMVPN](Diplom/DMVPN-IPSec.md) | Encryption | AES IPSec applied to both GRE and DMVPN tunnels |

## Switching and Routing Foundation

| Lab | Topic | Key outcome |
|---|---|---|
| [VLAN](VLAN) | L2 segmentation | Extended VLANs, VTP domain, 802.1Q trunk links |
| [STP](STP) | Loop prevention | Root bridge election, port cost/priority tuning, topology change analysis |
| [EtherChannel](EtherChannel) | Link aggregation | LACP and PAgP port-channels; VLAN trunking over aggregated links |
| [HSRP](EtherChannel/HSRP.md) | Gateway redundancy | Active/standby pair, virtual IP, preemption and priority |
| [OSPF Single-Area](OSPF) | L3 routing | Neighbor relationships, passive interfaces, metric tuning |
| [OSPF Multi-Area](OSPF/Multiarea.md) | Scalable OSPF | ABR/ASBR roles, inter-area summarisation, MD5 authentication |
| [EIGRP](EIGRP) | Cisco routing | Topology table, bandwidth tuning, feasible successor |
| [EIGRP Advanced](EIGRP/Advanced.md) | Traffic tuning | Auto-summary, default route redistribution, hello/hold timers |
| [Corporate Project](PROJECT) | Full design | Three-tier Cisco hierarchy: core/distribution/access with OSPF, HSRP, DHCP, PAT |
| [Self-Study](SelfLearn) | Extended topics | BGP, GRE, IS-IS, provider connectivity scenarios |

## Background

This work builds on production network experience with MikroTik (MTCNA, MTCWE, MTCIPv6E)  
and translates that knowledge into Cisco IOS syntax and enterprise design patterns.

**Certifications:**  
✅ LPIC-2 (202-450) &nbsp; ✅ MTCNA / MTCWE / MTCIPv6E &nbsp; 🔄 Cisco CCNA 200-301 (in progress)

**Related repositories:**  
[homelab-infrastructure](https://github.com/NickelFace/homelab-infrastructure) — Terraform + Ansible + Docker for a self-hosted Linux lab

## Environment

- Simulator: [EVE-NG Community](https://www.eve-ng.net/)
- Platform: Cisco IOS (routers and L3 switches)
- Topology files: included in each lab folder
