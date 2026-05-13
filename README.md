[🇷🇺 Русская версия](README.ru.md)

# OTUS Network Engineer

> Labs and diploma project for the OTUS Network Engineer course — switching, routing protocols, VPN tunneling, and enterprise network design on Cisco IOS / EVE-NG.

## Labs

| # | Topic | Description |
|---|-------|-------------|
| 01 | [VLAN](VLAN/) | Configuring Extended VLAN, VTP and DTP. Trunk links, dynamic trunk protocol, extended-range VLANs. |
| 02 | [STP](STP/) | Deploying a switched network with redundant links. Root bridge election, port cost and priority tuning. |
| 03 | [EtherChannel](EtherChannel/) | Configuring EtherChannel with LACP and PAgP. Port-channel setup, VLAN trunking over aggregated links. |
| 03 | [HSRP](EtherChannel/HSRP.md) | First-hop redundancy with HSRP. Active/standby routers, virtual IP, priority and preemption. |
| 04 | [OSPF Basics](OSPF/) | OSPFv2 single-area. Router IDs, passive interfaces, neighbor relationships, routing table, OSPF metric. |
| 05 | [OSPF Multiarea](OSPF/Multiarea.md) | OSPFv2 multi-area. ABR, ASBR, inter-area summarization, MD5 authentication. |
| 06 | [EIGRP Basic](EIGRP/) | EIGRP for IPv4 — neighbors, topology table, bandwidth tuning, passive interfaces. |
| 06 | [EIGRP Advanced](EIGRP/Advanced.md) | Advanced EIGRP — auto-summary, redistribute static default route, hello/hold timers, bandwidth percent. |
| 07 | [IPv4 / IPv6 Addressing](Diplom/IPv4-IPv6.md) | Diploma project: IPv4/IPv6 address space planning, VLANs, management networks, link-local addressing. |
| 08 | [PBR](Diplom/PBR.md) | Policy-Based Routing for the Chokurdakh office — traffic distribution between two uplinks with IP SLA tracking. |
| 09 | [OSPF Filtering](Diplom/OSPF-Filter.md) | OSPF with area segmentation and inter-area route filtering for the Moscow office. |
| 10 | [OSPFv3](Diplom/OSPFv3.md) | OSPFv3 for IPv6 — same logic as OSPFv2 with metrics, timers, and filters preserved. |
| 11 | [BGP Basics](Diplom/BGP.md) | eBGP between autonomous systems to enable connectivity between Moscow and St. Petersburg offices. |
| 12 | [iBGP](Diplom/IBGP.md) | iBGP in the Moscow office and Triada provider. Preferred provider selection and traffic load balancing. |
| 13 | [BGP Filtering](Diplom/BGP-Path.md) | BGP route filtering for Moscow and St. Petersburg — restricting provider advertisements to default route only. |
| 14 | [PAT, DHCP, NTP](Diplom/PAT-DHCP-NTP.md) | NAT/PAT, DHCP server, and NTP time synchronization for all offices. |
| 15 | [DMVPN](Diplom/DMVPN.md) | GRE tunnel between Moscow and St. Petersburg. DMVPN hub-and-spoke between Moscow, Chokurdakh, and Labytnangi. |
| 16 | [IPSec over DMVPN](Diplom/DMVPN-IPSec.md) | GRE over IPSec between Moscow and St. Petersburg. DMVPN over IPSec with encryption. |
| 17 | [Project](PROJECT/) | Corporate office network: Cisco three-tier hierarchical model with OSPF, HSRP, STP, DHCP, PAT, and L2 Security. |
| — | [Self-Study](SelfLearn/) | Additional materials: BGP, GRE tunnels, IS-IS, provider connectivity scenarios. |

## Environment

- Simulator: [EVE-NG](https://www.eve-ng.net/)
- Platform: Cisco IOS
- Course: [OTUS Network Engineer](https://otus.ru/lessons/network-engineer/)
