[🇷🇺 Русская версия](OSPFv3.ru.md)


## OSPFv3 for IPv6

### Assignment

Goal: Configure OSPF for IPv6, preserving the same logic (metrics, timers, filters) as OSPF for IPv4.

1. Configure OSPFv3 for IPv6, preserving the same area structure as OSPFv2
2. Document the plan and changes

![OSPF](/img/neteng/09/1.png)

Link-local addresses were missing from the previous lab — adding them now alongside OSPFv3 config.


## Configuration

<details>
<summary>R14</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary

interface Ethernet0/0
 ipv6 address FE80:4::14 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/1
 ipv6 address FE80:5::14 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 address FE80:6::14 link-local
 ipv6 ospf 1 area 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 15.15.15.15
 distribute-list prefix-list PL6 in

interface Ethernet0/0
 ipv6 address FE80:1::15 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/1
 ipv6 address FE80:2::15 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 address FE80:3::15 link-local
 ipv6 ospf 1 area 102

ipv6 prefix-list PL6 seq 5 deny 2002:ACAD:DB8:7::/64 le 128
ipv6 prefix-list PL6 seq 10 permit ::/0 le 128
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R12</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 12.12.12.12

interface Ethernet0/0
 ipv6 address FE80:7::12 link-local
 ipv6 ospf 1 area 10

interface Ethernet0/2
 ipv6 address FE80:4::12 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 address FE80:2::12 link-local
 ipv6 ospf 1 area 0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R13</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 13.13.13.13

interface Ethernet0/0
 ipv6 address FE80:8::13 link-local
 ipv6 ospf 1 area 10

interface Ethernet0/2
 ipv6 address FE80:1::13 link-local
 ipv6 ospf 1 area 0

interface Ethernet0/3
 ipv6 address FE80:5::13 link-local
 ipv6 ospf 1 area 0
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R19</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 19.19.19.19
 area 101 stub

interface Ethernet0/0
 ipv6 address FE80:6::19 link-local
 ipv6 ospf 1 area 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R20</summary>
<pre><code>
enable
configure terminal
ipv6 router ospf 1
 router-id 20.20.20.20

interface Ethernet0/0
 ipv6 address FE80:3::20 link-local
 ipv6 ospf 1 area 102
end
copy running-config startup-config
</code></pre>
</details>


## Verification

R19 (area 101 — stub no-summary) receives only the default route:

<details>
<summary>R19 — show ipv6 route ospf</summary>
<pre><code>
R19#sh ipv6 route ospf
IPv6 Routing Table - default - 4 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
OI  ::/0 [110/11]
     via FE80:6::14, Ethernet0/0
</code></pre>
</details>

R20 (area 102) receives all routes except the area 101 network **2002:ACAD:DB8:7::/64** — filtered by `PL6` on R15:

<details>
<summary>R20 — show ipv6 route ospf</summary>
<pre><code>
R20#show ipv6 route ospf
IPv6 Routing Table - default - 9 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
OI  2002:ACAD:DB8::/64 [110/20]
     via FE80:3::15, Ethernet0/0
OI  2002:ACAD:DB8:1::/64 [110/20]
     via FE80:3::15, Ethernet0/0
OI  2002:ACAD:DB8:4::/64 [110/30]
     via FE80:3::15, Ethernet0/0
OI  2002:ACAD:DB8:5::/64 [110/30]
     via FE80:3::15, Ethernet0/0
OI  2002:ACAD:DB8:8::/64 [110/30]
     via FE80:3::15, Ethernet0/0
OI  2002:ACAD:DB8:9::/64 [110/30]
     via FE80:3::15, Ethernet0/0
</code></pre>
</details>

Timers and metrics are identical to OSPFv2 — no changes needed:

<details>
<summary>R15 — show ipv6 ospf interface e0/0 vs show ip ospf interface e0/0</summary>
<pre><code>
R15#show ipv6 ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Link Local Address FE80:1::15, Interface ID 3
  Area 0, Process ID 1, Instance ID 0, Router ID 15.15.15.15
  Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State BDR, Priority 1
  Designated Router (ID) 13.13.13.13, local address FE80:1::13
  Backup Designated router (ID) 15.15.15.15, local address FE80:1::15
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:01
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 13.13.13.13  (Designated Router)

R15#show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 10.10.10.5/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 15.15.15.15, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 15.15.15.15, Interface address 10.10.10.5
  Backup Designated router (ID) 13.13.13.13, Interface address 10.10.10.6
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 13.13.13.13  (Backup Designated Router)
</code></pre>
</details>


*Network Engineer Course | Lab 10*

---

[← 09. OSPF Filter](OSPF-Filter.md) | [🏠 Home](../README.md) | [11. BGP Basics →](BGP.md)
