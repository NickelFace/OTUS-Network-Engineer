[🇷🇺 Русская версия](IBGP.ru.md)


## iBGP

### Assignment

Goal: Configure iBGP in the Moscow office, configure iBGP in the Triada provider network, and establish full IP connectivity across all networks.

1. Configure iBGP in the Moscow office between R14 and R15
2. Configure iBGP in the Triada provider network
3. Configure the Moscow office to prefer Lamas as the primary provider
4. Enable iBGP in the St. Petersburg office (without using OSPF)
5. Configure the St. Petersburg office to load-balance traffic to any office across both uplinks simultaneously
6. All networks in the lab must have IP connectivity
7. Document the plan and changes

![EVE Topology](/img/neteng/11/1.png)


## iBGP — Moscow (R14 ↔ R15)

iBGP session uses Loopback0 addresses reachable via OSPF area 0. `next-hop-self` ensures the iBGP peer can reach external next-hops.

<details>
<summary>R14 — Loopback + OSPF + bgp summary</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.1.14 255.255.255.255

router ospf 1
 network 1.1.1.14 0.0.0.0 area 0

R14#show ip bgp summary
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 11, main routing table version 11

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.15        4         1001      24      27       11    0    0 00:14:09        0
100.100.100.2   4          101      28      23       11    0    0 00:18:46       10
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15 — Loopback + OSPF + bgp summary</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.1.15 255.255.255.255

router ospf 1
 network 1.1.1.15 0.0.0.0 area 0

R15#show ip bgp summary
BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 21, main routing table version 21

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.14        4         1001      31      28       21    0    0 00:18:04       10
111.111.111.2   4          301      31      32       21    0    0 00:20:19        0
end
copy running-config startup-config
</code></pre>
</details>


## iBGP — Triada (R23, R24, R25, R26)

<details>
<summary>R23 — show ip bgp summary</summary>
<pre><code>
R23>show ip bgp summary
BGP router identifier 23.23.23.23, local AS number 520
BGP table version is 16, main routing table version 16

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.24.1       4          520      87      88       16    0    0 01:13:00        5
50.0.25.1       4          520      84      88       16    0    0 01:13:01        2
50.0.26.1       4          520      85      88       16    0    0 01:13:02        2
100.100.100.5   4          101      87      88       16    0    0 01:13:48        5
</code></pre>
</details>

<details>
<summary>R24 — show ip bgp summary</summary>
<pre><code>
R24#show ip bgp summary
BGP router identifier 24.24.24.24, local AS number 520
BGP table version is 18, main routing table version 18

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      89      88       18    0    0 01:13:45        4
50.0.25.1       4          520      84      89       18    0    0 01:13:57        2
50.0.26.1       4          520      85      88       18    0    0 01:13:54        2
77.77.77.10     4         2042      89      90       18    0    0 01:14:30        0
111.111.111.5   4          301      89      89       18    0    0 01:14:33        5
</code></pre>
</details>

<details>
<summary>R25 — show ip bgp summary</summary>
<pre><code>
R25#show ip bgp summary
BGP router identifier 25.25.25.25, local AS number 520
BGP table version is 16, main routing table version 16

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      89      86       16    0    0 01:14:28        4
50.0.24.1       4          520      90      85       16    0    0 01:14:40        5
50.0.26.1       4          520      85      87       16    0    0 01:14:36        2
</code></pre>
</details>

<details>
<summary>R26 — show ip bgp summary</summary>
<pre><code>
R26#show ip bgp summary
BGP router identifier 26.26.26.26, local AS number 520
BGP table version is 6, main routing table version 6

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      91      88        6    0    0 01:15:35        4
50.0.24.1       4          520      90      87        6    0    0 01:15:43        5
50.0.25.1       4          520      88      86        6    0    0 01:15:41        2
77.77.77.14     4         2042      91      89        6    0    0 01:16:24        0
</code></pre>
</details>


## Moscow — Lamas as preferred provider

R15 marks all updates from R21 (Lamas) with `local-preference 150` at ingress via route-map `LP`. This preference is then propagated to all iBGP peers — R14 sees LP=150 for Lamas-learned routes and prefers them over Kitorn paths. A filter-list restricts outbound announcements to only locally originated prefixes (`^$`).

<details>
<summary>R15 — BGP config</summary>
<pre><code>
enable
configure terminal
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 215.215.215.215 mask 255.255.255.255
 network 219.219.219.219 mask 255.255.255.255
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 next-hop-self
 neighbor 111.111.111.2 remote-as 301
 neighbor 111.111.111.2 route-map LP in
 neighbor 111.111.111.2 filter-list 1 out

route-map LP permit 10
 set local-preference 150

ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R14 — BGP config</summary>
<pre><code>
enable
configure terminal
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 214.214.214.214 mask 255.255.255.255
 network 215.215.215.215 mask 255.255.255.255
 network 219.219.219.219 mask 255.255.255.255
 neighbor MSK peer-group
 neighbor MSK remote-as 1001
 neighbor MSK update-source Loopback0
 neighbor MSK next-hop-self
 neighbor 1.1.1.15 peer-group MSK
 neighbor 100.100.100.2 remote-as 101
 neighbor 100.100.100.2 filter-list 1 out
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R14 — show ip bgp (Lamas preferred, LP=150)</summary>
<pre><code>
R14#show ip bgp
     Network          Next Hop            Metric LocPrf Weight Path
 *   77.77.77.8/30    100.100.100.2                          0 101 520 i
 *>i                  1.1.1.15                 0    150      0 301 520 i
 *   77.77.77.12/30   100.100.100.2                          0 101 520 i
 *>i                  1.1.1.15                 0    150      0 301 520 i
 r>i 100.100.100.0/30 1.1.1.15                 0    150      0 301 101 i
 r                    100.100.100.2            0             0 101 i
 *>i 100.100.100.4/30 1.1.1.15                 0    150      0 301 101 i
 *                    100.100.100.2            0             0 101 i
 *>i 110.110.110.0/30 1.1.1.15                 0    150      0 301 i
 *                    100.100.100.2                          0 101 301 i
 *   111.110.35.8/30  100.100.100.2                          0 101 520 i
 *>i                  1.1.1.15                 0    150      0 301 520 i
 *   111.110.35.12/30 100.100.100.2                          0 101 520 i
 *>i                  1.1.1.15                 0    150      0 301 520 i
 *>i 111.111.111.0/30 1.1.1.15                 0    150      0 301 i
 *                    100.100.100.2                          0 101 301 i
 *>i 111.111.111.4/30 1.1.1.15                 0    150      0 301 i
 *                    100.100.100.2                          0 101 301 i
 *   115.115.115.115/32
                       100.100.100.2                          0 101 520 2042 i
 *>i                  1.1.1.15                 0    150      0 301 520 2042 i
 *   210.110.35.0/30  100.100.100.2                          0 101 520 i
 *>i                  1.1.1.15                 0    150      0 301 520 i
 *>  214.214.214.214/32
                       0.0.0.0                  0         32768 i
 *>i 215.215.215.215/32
                       1.1.1.15                 0    100      0 i
 *>i 219.219.219.219/32
                       1.1.1.15                 0    100      0 i
</code></pre>
</details>

<details>
<summary>R14 — traceroute via Lamas</summary>
<pre><code>
R14#traceroute 77.77.77.10 source ethernet 0/2
  1 10.10.10.26 1 msec 0 msec 1 msec
  2 111.111.111.2 [AS 301] 1 msec 1 msec 0 msec
  3 111.111.111.6 [AS 301] 1 msec 1 msec 1 msec
  4 77.77.77.10 [AS 520] 2 msec

R14#traceroute 111.110.35.13
  1 10.10.10.26 0 msec 0 msec 0 msec
  2 111.111.111.2 [AS 301] 1 msec 0 msec 1 msec
  3 111.111.111.6 [AS 301] 1 msec 1 msec 1 msec
  4 10.10.30.14 1 msec *  2 msec

R14#traceroute 115.115.115.115
  1 10.10.10.26 2 msec 1 msec 1 msec
  2 111.111.111.2 [AS 301] 3 msec 1 msec 1 msec
  3 111.111.111.6 [AS 301] 2 msec 2 msec 1 msec
  4 77.77.77.10 [AS 520] 2 msec *  1 msec
</code></pre>
</details>

> **Note:** Setting local-preference at R15 ingress (instead of outbound from R14 toward R15) is the scalable approach — R15 propagates LP=150 to all iBGP peers automatically. *(Thanks to Alexei for the consultation.)*


## iBGP — St. Petersburg (AS 2042, no OSPF)

EIGRP is used for loopback reachability within the office. iBGP sessions run between all four routers using loopback addresses.

<details>
<summary>R16 — config</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.2.16 255.255.255.255

router bgp 2042
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0

router eigrp 1
 network 1.1.2.16 0.0.0.0
 network 10.10.20.4 0.0.0.3
 network 10.10.20.8 0.0.0.3
 network 172.16.11.0 0.0.0.255
 passive-interface Ethernet0/0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R16 — show ip bgp summary</summary>
<pre><code>
BGP router identifier 1.1.2.16, local AS number 2042
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.17        4         2042      13      13        1    0    0 00:10:01        0
1.1.2.18        4         2042      21      15        1    0    0 00:10:26       10
1.1.2.32        4         2042      12      12        1    0    0 00:09:27        0
</code></pre>
</details>

<details>
<summary>R17 — config</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.2.17 255.255.255.255

router bgp 2042
 bgp log-neighbor-changes
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0

router eigrp 1
 network 1.1.2.17 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 172.16.10.0 0.0.0.255
 passive-interface Ethernet0/0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R17 — show ip bgp summary</summary>
<pre><code>
BGP router identifier 1.1.2.17, local AS number 2042
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      15      15        1    0    0 00:12:08        0
1.1.2.18        4         2042      24      17        1    0    0 00:12:11       10
1.1.2.32        4         2042      14      14        1    0    0 00:11:34        0
</code></pre>
</details>

<details>
<summary>R18 — config</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.2.18 255.255.255.255

router bgp 2042
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0

router eigrp 1
 network 1.1.2.18 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 10.10.20.4 0.0.0.3
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R18 — show ip bgp summary</summary>
<pre><code>
BGP router identifier 18.18.18.18, local AS number 2042
BGP table version is 12, main routing table version 12

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      19      25       12    0    0 00:14:16        0
1.1.2.17        4         2042      19      26       12    0    0 00:13:55        0
1.1.2.32        4         2042      18      25       12    0    0 00:13:19        0
77.77.77.9      4          520     121     122       12    0    0 01:45:22       10
77.77.77.13     4          520     119     122       12    0    0 01:45:24        5
</code></pre>
</details>

<details>
<summary>R32 — config</summary>
<pre><code>
enable
configure terminal
interface Loopback0
 ip address 1.1.2.32 255.255.255.255

router bgp 2042
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0

router eigrp 1
 network 1.1.2.32 0.0.0.0
 network 10.10.20.8 0.0.0.3
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R32 — show ip bgp summary</summary>
<pre><code>
BGP router identifier 1.1.2.32, local AS number 2042
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      10      10        1    0    0 00:07:34        0
1.1.2.17        4         2042      10      10        1    0    0 00:07:34        0
1.1.2.18        4         2042      19      12        1    0    0 00:07:36       10
</code></pre>
</details>


## Verify full IP connectivity

<details>
<summary>R15 / R14 ping R18 (St. Petersburg)</summary>
<pre><code>
R15#ping 77.77.77.10
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R15#ping 77.77.77.14
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R14#ping 77.77.77.10
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R14#ping 77.77.77.14
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
</code></pre>
</details>

<details>
<summary>R18 ping Labytnangi / Chokurdakh</summary>
<pre><code>
R18#ping 210.110.35.2
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R18#ping 111.110.35.14
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R18#ping 111.110.35.10
!!!!!  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
</code></pre>
</details>


## Documentation

IP address plan: [Google Sheets](https://docs.google.com/spreadsheets/d/1KDH3y6QoNxf8ykn9vnS4ybjlyAnmF-XwGE42wKX74wE/edit?usp=sharing)


## Full router configs

> R21, R22, R23, R24, R25, R26 configs unchanged from lab 11.

<details>
<summary>R14 (AS 1001) — updated</summary>
<pre><code>
enable
configure terminal
hostname R14
ipv6 unicast-routing
no ip domain-lookup
!
interface Loopback0
 ip address 1.1.1.14 255.255.255.255
!
interface Loopback1
 ip address 214.214.214.214 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.10.17 255.255.255.252
 ipv6 enable
 ipv6 address FE80:4::14 link-local
 ipv6 address 2002:ACAD:DB8:4::14/64
 ipv6 ospf 1 area 0
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.10.21 255.255.255.252
 ipv6 enable
 ipv6 address FE80:5::14 link-local
 ipv6 address 2002:ACAD:DB8:5::14/64
 ipv6 ospf 1 area 0
 no shutdown
!
interface Ethernet0/2
 ip address 100.100.100.1 255.255.255.252
 ipv6 enable
 ipv6 address 2002:ACAD:DB8:6::14/64
 no shutdown
!
interface Ethernet0/3
 ip address 10.10.10.13 255.255.255.252
 ipv6 enable
 ipv6 address FE80:6::14 link-local
 ipv6 address 2002:ACAD:DB8:7::14/64
 ipv6 ospf 1 area 101
 no shutdown
!
interface Ethernet1/0
 ip address 10.10.10.25 255.255.255.252
 ipv6 enable
 ipv6 address FE80:A::14 link-local
 ipv6 address 2002:ACAD:DB8:A::14/64
 no shutdown
!
router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 network 1.1.1.14 0.0.0.0 area 0
 network 10.10.10.12 0.0.0.3 area 101
 network 10.10.10.16 0.0.0.3 area 0
 network 10.10.10.20 0.0.0.3 area 0
 default-information originate
!
ipv6 router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 214.214.214.214 mask 255.255.255.255
 network 215.215.215.215 mask 255.255.255.255
 network 219.219.219.219 mask 255.255.255.255
 neighbor MSK peer-group
 neighbor MSK remote-as 1001
 neighbor MSK update-source Loopback0
 neighbor MSK next-hop-self
 neighbor 1.1.1.15 peer-group MSK
 neighbor 100.100.100.2 remote-as 101
 neighbor 100.100.100.2 filter-list 1 out
!
ip route 0.0.0.0 0.0.0.0 100.100.100.2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15 (AS 1001) — updated</summary>
<pre><code>
enable
configure terminal
hostname R15
ipv6 unicast-routing
no ip domain-lookup
!
interface Loopback0
 ip address 1.1.1.15 255.255.255.255
!
interface Loopback1
 ip address 215.215.215.215 255.255.255.255
!
interface Loopback2
 ip address 219.219.219.219 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.10.5 255.255.255.252
 ipv6 enable
 ipv6 address FE80:1::15 link-local
 ipv6 address 2002:ACAD:DB8:0::15/64
 ipv6 ospf 1 area 0
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.10.9 255.255.255.252
 ipv6 enable
 ipv6 address FE80:2::15 link-local
 ipv6 address 2002:ACAD:DB8:1::15/64
 ipv6 ospf 1 area 0
 no shutdown
!
interface Ethernet0/2
 ip address 111.111.111.1 255.255.255.252
 ipv6 enable
 ipv6 address 2002:ACAD:DB8:2::15/64
 no shutdown
!
interface Ethernet0/3
 ip address 10.10.10.1 255.255.255.252
 ipv6 enable
 ipv6 address FE80:3::15 link-local
 ipv6 address 2002:ACAD:DB8:3::15/64
 ipv6 ospf 1 area 102
 no shutdown
!
interface Ethernet1/0
 ip address 10.10.10.26 255.255.255.252
 ipv6 enable
 ipv6 address FE80:A::15 link-local
 ipv6 address 2002:ACAD:DB8:A::15/64
 no shutdown
!
router ospf 1
 router-id 15.15.15.15
 network 1.1.1.15 0.0.0.0 area 0
 network 10.10.10.0 0.0.0.3 area 102
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.8 0.0.0.3 area 0
 distribute-list prefix PL1 in
 default-information originate
!
ipv6 router ospf 1
 router-id 15.15.15.15
 distribute-list prefix-list PL6 in
!
ip prefix-list PL1 seq 5 deny 10.10.10.12/30
ip prefix-list PL1 seq 10 permit 0.0.0.0/0 le 32
!
ipv6 prefix-list PL6 seq 5 deny 2002:ACAD:DB8:7::/64 le 128
ipv6 prefix-list PL6 seq 10 permit ::/0 le 128
!
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*
!
route-map LP permit 10
 set local-preference 150
!
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 215.215.215.215 mask 255.255.255.255
 network 219.219.219.219 mask 255.255.255.255
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 update-source Loopback0
 neighbor 1.1.1.14 next-hop-self
 neighbor 111.111.111.2 remote-as 301
 neighbor 111.111.111.2 route-map LP in
 neighbor 111.111.111.2 filter-list 1 out
!
ip route 0.0.0.0 0.0.0.0 111.111.111.2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R16 — St. Petersburg (AS 2042)</summary>
<pre><code>
enable
configure terminal
hostname R16
!
interface Loopback0
 ip address 1.1.2.16 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.11.1 255.255.255.0
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:5::16/64
 ipv6 address FE80:15::16 link-local
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.20.6 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:0::16/64
 ipv6 address FE80:10::16 link-local
 no shutdown
!
interface Ethernet0/3
 ip address 10.10.20.9 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:4::16/64
 ipv6 address FE80:14::16 link-local
 no shutdown
!
router eigrp 1
 network 1.1.2.16 0.0.0.0
 network 10.10.20.4 0.0.0.3
 network 10.10.20.8 0.0.0.3
 network 172.16.11.0 0.0.0.255
 passive-interface Ethernet0/0
!
router bgp 2042
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R17 — St. Petersburg (AS 2042)</summary>
<pre><code>
enable
configure terminal
hostname R17
!
interface Loopback0
 ip address 1.1.2.17 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.10.1 255.255.255.0
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:6::17/64
 ipv6 address FE80:16::17 link-local
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.20.2 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:1::17/64
 ipv6 address FE80:11::17 link-local
 no shutdown
!
router eigrp 1
 network 1.1.2.17 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 172.16.10.0 0.0.0.255
 passive-interface Ethernet0/0
!
router bgp 2042
 bgp log-neighbor-changes
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R18 — St. Petersburg (AS 2042) — updated</summary>
<pre><code>
enable
configure terminal
hostname R18
!
interface Loopback0
 ip address 1.1.2.18 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.20.5 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:0::18/64
 ipv6 address FE80:10::18 link-local
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.20.1 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:1::18/64
 ipv6 address FE80:11::18 link-local
 no shutdown
!
interface Ethernet0/2
 ip address 77.77.77.10 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:2::18/64
 no shutdown
!
interface Ethernet0/3
 ip address 77.77.77.14 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:3::18/64
 no shutdown
!
router eigrp 1
 network 1.1.2.18 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 10.10.20.4 0.0.0.3
!
router bgp 2042
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R32 — St. Petersburg (AS 2042)</summary>
<pre><code>
enable
configure terminal
hostname R32
!
interface Loopback0
 ip address 1.1.2.32 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.20.10 255.255.255.252
 ipv6 enable
 ipv6 address 2CAD:1995:B0DA:4::32/64
 ipv6 address FE80:14::32 link-local
 no shutdown
!
router eigrp 1
 network 1.1.2.32 0.0.0.0
 network 10.10.20.8 0.0.0.3
!
router bgp 2042
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 update-source Loopback0
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
end
copy running-config startup-config
</code></pre>
</details>


*Network Engineer Course | Lab 12*

---

[← 11. BGP Basics](BGP.md) | [🏠 Home](../README.md) | [13. BGP Filtering →](BGP-Path.md)
