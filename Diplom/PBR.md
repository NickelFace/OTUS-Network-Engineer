[🇷🇺 Русская версия](PBR.ru.md)


## Policy-Based Routing (PBR)

### Assignment

Goal: Configure routing policy for the Chokurdakh office and distribute traffic between 2 uplinks.

In this lab you are expected to independently:

1. Configure routing policy for the Chokurdakh office networks
2. Distribute traffic between the two provider uplinks in Chokurdakh
3. Configure link tracking via IP SLA in Chokurdakh
4. Configure a default route for the Labytnangi office
5. Document the plan and changes

> **Note:** `verify-availability` in route-map may not work for IPv6.

![PBR](/img/neteng/08/1.png)

| Device | Port | IPv4             | IPv6                 | Note            | Region     |
| ------ | ---- | ---------------- | -------------------- | --------------- | ---------- |
| R27    | e0/0 | 210.110.35.2/30  | 2CAD:1995:B0DA:A::27 | R25(Triada)     | Labytnangi |
| R28    | e0/0 | 111.110.35.14/30 | 2CAD:1995:B0DA:7::28 | R26(Triada)     | Chokurdakh |
|        | e0/1 | 111.110.35.10/30 | 2CAD:1995:B0DA:8::28 | R25(Triada)     |            |
|        | e0/2 | 172.16.40.1/24   | 2CAD:1995:B0DA:9::28 | SW29(Chokurdah) |            |
| VPC30  |      | 172.16.40.10     | 2cad:1995:b0da:9::10 |                 |            |
| VPC31  |      | 172.16.40.11     | 2cad:1995:b0da:9::11 |                 |            |


## Configure routing policy for the Chokurdakh office networks

<details>
<summary>R28 — interface addresses</summary>
<pre><code>
R28#show ipv6 int brief
Ethernet0/0            [up/up]
    FE80::A8BB:CCFF:FE01:C000
    2CAD:1995:B0DA:7::28
Ethernet0/1            [up/up]
    FE80::A8BB:CCFF:FE01:C010
    2CAD:1995:B0DA:8::28
Ethernet0/2            [up/up]
    FE80:21::28
    2CAD:1995:B0DA:9::28

R28#show ip int bri
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                111.110.35.14   YES manual up                    up
Ethernet0/1                111.110.35.10   YES manual up                    up
Ethernet0/2                172.16.40.1     YES manual up                    up
</code></pre>
</details>


## Distribute traffic between the two provider uplinks in Chokurdakh

ACL `ACL1` matches traffic from VPC30 (172.16.40.10) and the route-map steers it via R26 (e0/0). A static route to R25's loopback (215.215.215.215) is added to verify the alternate path — the provider side runs OSPF with no additional configuration required.

<details>
<summary>R28</summary>
<pre><code>
enable
configure terminal

ip route 215.215.215.215 255.255.255.255 111.110.35.9

ip access-list extended ACL1
 permit ip host 172.16.40.10 any

route-map TEST permit 10
 match ip address ACL1
 set ip next-hop 111.110.35.13
 set ipv6 next-hop 2CAD:1995:B0DA:7::26

interface Ethernet0/2
 ip policy route-map TEST
end
copy running-config startup-config
</code></pre>
</details>

VPC30 trace before and after applying the route-map:

<details>
<summary>VPC30</summary>
<pre><code>
enable
configure terminal
! Before route-map
VPCS> trace 215.215.215.215
trace to 215.215.215.215, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   0.551 ms  0.369 ms  0.357 ms
 2   *111.110.35.9   1.014 ms (ICMP type:3, code:3, Destination port unreachable)  *

! After route-map
VPCS> trace 215.215.215.215
trace to 215.215.215.215, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   0.629 ms  0.560 ms  0.463 ms
 2   111.110.35.13   0.902 ms  0.655 ms  0.590 ms
 3   *10.10.30.9   0.882 ms (ICMP type:3, code:3, Destination port unreachable)  *
end
copy running-config startup-config
</code></pre>
</details>


## Configure link tracking via IP SLA in Chokurdakh

Two IP SLA probes monitor reachability to each provider next-hop. Tracked objects are referenced in both the default routes and the route-map:

<details>
<summary>R28</summary>
<pre><code>
enable
configure terminal
ip sla 1
 icmp-echo 111.110.35.9 source-interface Ethernet0/1
 frequency 5
ip sla schedule 1 life forever start-time now
!
ip sla 2
 icmp-echo 111.110.35.13 source-interface Ethernet0/0
 frequency 5
ip sla schedule 2 life forever start-time now
!
track 1 ip sla 1 reachability
track 2 ip sla 2 reachability
!
ip route 0.0.0.0 0.0.0.0 111.110.35.9 track 1
ip route 0.0.0.0 0.0.0.0 111.110.35.13 track 2

! Updated route-map with availability verification
route-map TEST permit 10
 match ip address ACL1
 set ip next-hop verify-availability 111.110.35.9 10 track 1
 set ip next-hop verify-availability 111.110.35.13 15 track 2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R28 — show track / show route-map</summary>
<pre><code>
R28#show track 1
Track 1
  IP SLA 1 reachability
  Reachability is Up
    13 changes, last change 00:40:47
  Latest operation return code: OK
  Latest RTT (millisecs) 1
  Tracked by:
    ROUTE-MAP 0
    STATIC-IP-ROUTING 0

R28#show track 2
Track 2
  IP SLA 2 reachability
  Reachability is Up
    7 changes, last change 01:09:14
  Latest operation return code: OK
  Latest RTT (millisecs) 1
  Tracked by:
    ROUTE-MAP 0
    STATIC-IP-ROUTING 0

R28#show route-map
route-map TEST, permit, sequence 10
  Match clauses:
    ip address (access-lists): ACL1
  Set clauses:
    ip next-hop verify-availability 111.110.35.9 10 track 1  [up]
    ip next-hop verify-availability 111.110.35.13 15 track 2  [up]
     ipv6 next-hop 2CAD:1995:B0DA:7::26
  Policy routing matches: 234 packets, 24284 bytes
</code></pre>
</details>


## Configure a default route for the Labytnangi office

<details>
<summary>R27</summary>
<pre><code>
enable
configure terminal
R27(config)#do sh run | s route
ip route 0.0.0.0 0.0.0.0 210.110.35.1
ipv6 route ::/0 2CAD:1995:B0DA:A::25
end
copy running-config startup-config
</code></pre>
</details>


*Network Engineer Course | Lab 08*

---

[← 07. IPv4/IPv6](IPv4-IPv6.md) | [🏠 Home](../README.md) | [09. OSPF Filter →](OSPF-Filter.md)
