[🇷🇺 Русская версия](BGP.ru.md)


## BGP Basics

### Assignment

Goal: Configure BGP between autonomous systems and ensure connectivity between the Moscow and St. Petersburg offices.

1. Configure eBGP between the Moscow office and two providers — Kitorn and Lamas
2. Configure eBGP between providers Kitorn and Lamas
3. Configure eBGP between Lamas and Triada
4. Configure eBGP between the St. Petersburg office and provider Triada
5. Enable IP connectivity between the Moscow and St. Petersburg offices
6. Document the plan and changes

![EVE Topology](/img/neteng/11/1.png)

A link between R14 and R15 was added — to simplify connecting them in area 0 and to provide redundancy for internet-bound traffic.


## eBGP — Moscow ↔ Kitorn and Lamas

<details>
<summary>R14 (AS 1001) — config</summary>
<pre><code>
enable
configure terminal
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 10.10.10.26 remote-as 1001
 neighbor 100.100.100.2 remote-as 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R14 — show bgp summary</summary>
<pre><code>
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 84, main routing table version 84

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.26     4         1001     379     383       84    0    0 05:34:13        9
100.100.100.2   4          101     381     378       84    0    0 05:33:33       10
</code></pre>
</details>

<details>
<summary>R15 (AS 1001) — config</summary>
<pre><code>
enable
configure terminal
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 10.10.10.25 remote-as 1001
 neighbor 111.111.111.2 remote-as 301
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15 — show bgp summary</summary>
<pre><code>
BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 50, main routing table version 50

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.25     4         1001     385     380       50    0    0 05:35:41        9
111.111.111.2   4          301     378     373       50    0    0 05:34:10       10
</code></pre>
</details>

<details>
<summary>R21 — Lamas (AS 301) — config</summary>
<pre><code>
enable
configure terminal
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 network 110.110.110.0 mask 255.255.255.252
 network 111.111.111.0 mask 255.255.255.252
 network 111.111.111.4 mask 255.255.255.252
 neighbor 110.110.110.1 remote-as 101
 neighbor 111.111.111.1 remote-as 1001
 neighbor 111.111.111.6 remote-as 520
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R21 — show bgp summary</summary>
<pre><code>
BGP router identifier 21.21.21.21, local AS number 301
BGP table version is 30, main routing table version 30

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
110.110.110.1   4          101     385     380       30    0    0 05:35:38        7
111.111.111.1   4         1001     375     379       30    0    0 05:35:38        1
111.111.111.6   4          520     361     364       30    0    0 05:20:03        6
</code></pre>
</details>

<details>
<summary>R22 — Kitorn (AS 101) — config</summary>
<pre><code>
enable
configure terminal
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 network 100.100.100.4 mask 255.255.255.252
 neighbor 100.100.100.1 remote-as 1001
 neighbor 100.100.100.6 remote-as 520
 neighbor 110.110.110.2 remote-as 301
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R22 — show bgp summary</summary>
<pre><code>
BGP router identifier 22.22.22.22, local AS number 101
BGP table version is 41, main routing table version 41

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.100.100.1   4         1001     383     386       41    0    0 05:37:31        1
100.100.100.6   4          520     363     369       41    0    0 05:23:07        6
110.110.110.2   4          301     382     386       41    0    0 05:36:40        8
</code></pre>
</details>


## eBGP — Kitorn/Lamas ↔ Triada

<details>
<summary>R23 — Triada (AS 520) — config</summary>
<pre><code>
enable
configure terminal
router bgp 520
 bgp router-id 23.23.23.23
 bgp log-neighbor-changes
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 100.100.100.5 remote-as 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R23 — show bgp summary</summary>
<pre><code>
BGP router identifier 23.23.23.23, local AS number 520
BGP table version is 32, main routing table version 32

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.24.1       4          520     364     364       32    0    0 05:23:00        5
50.0.25.1       4          520     358     363       32    0    0 05:22:01        2
50.0.26.1       4          520     357     362       32    0    0 05:20:51        2
100.100.100.5   4          101     371     366       32    0    0 05:25:02        5
</code></pre>
</details>

<details>
<summary>R24 — Triada (AS 520) — config</summary>
<pre><code>
enable
configure terminal
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 network 77.77.77.8 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 77.77.77.10 remote-as 2042
 neighbor 111.111.111.5 remote-as 301
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R24 — show bgp summary</summary>
<pre><code>
BGP router identifier 24.24.24.24, local AS number 520
BGP table version is 22, main routing table version 22

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520     367     367       22    0    0 05:25:27        4
50.0.25.1       4          520     359     365       22    0    0 05:24:28        2
50.0.26.1       4          520     361     362       22    0    0 05:23:18        2
77.77.77.10     4         2042     370     368       22    0    0 05:25:27        0
111.111.111.5   4          301     371     367       22    0    0 05:25:27        5
</code></pre>
</details>


## eBGP — St. Petersburg ↔ Triada

<details>
<summary>R26 — Triada (AS 520) — config</summary>
<pre><code>
enable
configure terminal
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 network 77.77.77.12 mask 255.255.255.252
 network 111.110.35.12 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 77.77.77.14 remote-as 2042
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R26 — show bgp summary</summary>
<pre><code>
BGP router identifier 26.26.26.26, local AS number 520
BGP table version is 16, main routing table version 16

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520     366     361       16    0    0 05:24:08        4
50.0.24.1       4          520     363     362       16    0    0 05:24:08        5
50.0.25.1       4          520     363     359       16    0    0 05:24:08        2
77.77.77.14     4         2042     370     363       16    0    0 05:24:08        0
</code></pre>
</details>

<details>
<summary>R18 — St. Petersburg (AS 2042) — config</summary>
<pre><code>
enable
configure terminal
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R18 — show bgp summary</summary>
<pre><code>
BGP router identifier 18.18.18.18, local AS number 2042
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
77.77.77.9      4          520      10       2        1    0    0 00:00:24       10
77.77.77.13     4          520       6       2        1    0    0 00:00:24        5
</code></pre>
</details>


## Verify connectivity — Moscow ↔ St. Petersburg

<details>
<summary>R15 ping R18</summary>
<pre><code>
R15#ping 77.77.77.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R15#ping 77.77.77.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
</code></pre>
</details>

<details>
<summary>R14 ping R18</summary>
<pre><code>
R14#ping 77.77.77.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R14#ping 77.77.77.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
</code></pre>
</details>


## Provider networks

| Triada           | Lamas            | Kitorn           |
| ---------------- | ---------------- | ---------------- |
| 77.77.77.8/30    | 111.111.111.0/30 | 100.100.100.0/30 |
| 77.77.77.12/30   | 110.110.110.0/30 | 100.100.100.4/30 |
| 111.110.35.8/30  | 111.111.111.4/30 |                  |
| 111.110.35.12/30 |                  |                  |
| 210.110.35.0/30  |                  |                  |


## Full router configs

<details>
<summary>R14 (AS 1001)</summary>
<pre><code>
enable
configure terminal
hostname R14
ipv6 unicast-routing
no ip domain-lookup
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
 network 10.10.10.12 0.0.0.3 area 101
 network 10.10.10.16 0.0.0.3 area 0
 network 10.10.10.20 0.0.0.3 area 0
 default-information originate
!
ipv6 router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
!
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 10.10.10.26 remote-as 1001
 neighbor 100.100.100.2 remote-as 101
!
ip route 0.0.0.0 0.0.0.0 100.100.100.2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15 (AS 1001)</summary>
<pre><code>
enable
configure terminal
hostname R15
ipv6 unicast-routing
no ip domain-lookup
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
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 10.10.10.25 remote-as 1001
 neighbor 111.111.111.2 remote-as 301
!
ip route 0.0.0.0 0.0.0.0 111.111.111.2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R21 — Lamas (AS 301)</summary>
<pre><code>
enable
configure terminal
hostname R21
!
interface Ethernet0/0
 ip address 111.111.111.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 110.110.110.2 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 111.111.111.5 255.255.255.252
 no shutdown
!
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 network 110.110.110.0 mask 255.255.255.252
 network 111.111.111.0 mask 255.255.255.252
 network 111.111.111.4 mask 255.255.255.252
 neighbor 110.110.110.1 remote-as 101
 neighbor 111.111.111.1 remote-as 1001
 neighbor 111.111.111.6 remote-as 520
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R22 — Kitorn (AS 101)</summary>
<pre><code>
enable
configure terminal
hostname R22
!
interface Ethernet0/0
 ip address 100.100.100.2 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 110.110.110.1 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 100.100.100.5 255.255.255.252
 no shutdown
!
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 network 100.100.100.4 mask 255.255.255.252
 neighbor 100.100.100.1 remote-as 1001
 neighbor 100.100.100.6 remote-as 520
 neighbor 110.110.110.2 remote-as 301
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R23 — Triada (AS 520)</summary>
<pre><code>
enable
configure terminal
hostname R23
!
interface Loopback0
 ip address 50.0.23.1 255.255.255.255
!
interface Ethernet0/0
 ip address 100.100.100.6 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.30.5 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.10.30.1 255.255.255.252
 no shutdown
!
router bgp 520
 bgp router-id 23.23.23.23
 bgp log-neighbor-changes
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 100.100.100.5 remote-as 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R24 — Triada (AS 520)</summary>
<pre><code>
enable
configure terminal
hostname R24
!
interface Loopback0
 ip address 50.0.24.1 255.255.255.255
!
interface Ethernet0/0
 ip address 111.111.111.6 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 10.10.30.13 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.10.30.2 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 77.77.77.9 255.255.255.252
 no shutdown
!
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 network 77.77.77.8 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 77.77.77.10 remote-as 2042
 neighbor 111.111.111.5 remote-as 301
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R25 — Triada (AS 520)</summary>
<pre><code>
enable
configure terminal
hostname R25
!
interface Loopback0
 ip address 50.0.25.1 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.30.6 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 210.110.35.1 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.10.30.9 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 111.110.35.9 255.255.255.252
 no shutdown
!
router bgp 520
 bgp router-id 25.25.25.25
 bgp log-neighbor-changes
 network 111.110.35.8 mask 255.255.255.252
 network 210.110.35.0 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R26 — Triada (AS 520)</summary>
<pre><code>
enable
configure terminal
hostname R26
!
interface Loopback0
 ip address 50.0.26.1 255.255.255.255
!
interface Ethernet0/0
 ip address 10.10.30.14 255.255.255.252
 no shutdown
!
interface Ethernet0/1
 ip address 111.110.35.13 255.255.255.252
 no shutdown
!
interface Ethernet0/2
 ip address 10.10.30.10 255.255.255.252
 no shutdown
!
interface Ethernet0/3
 ip address 77.77.77.13 255.255.255.252
 no shutdown
!
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 network 77.77.77.12 mask 255.255.255.252
 network 111.110.35.12 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 77.77.77.14 remote-as 2042
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R18 — St. Petersburg (AS 2042)</summary>
<pre><code>
enable
configure terminal
hostname R18
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
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520
end
copy running-config startup-config
</code></pre>
</details>


*Network Engineer Course | Lab 11*

---

[← 10. OSPFv3](OSPFv3.md) | [🏠 Home](../README.md) | [12. iBGP →](IBGP.md)
