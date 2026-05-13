[🇷🇺 Русская версия](OSPF-Filter.ru.md)


## OSPF Route Filtering

### Assignment

Goal: Configure OSPF in the Moscow office, divide the network into areas, and configure inter-area filtering.

1. R14–R15 are in area 0 — backbone
2. **R12–R13 are in area 10. They must also receive the default route**
3. R19 is in area 101 and receives only the default route
4. R20 is in area 102 and receives all routes except routes to area 101 networks
5. Document the plan and changes

![OSPF](/img/neteng/09/1.png)


## R14–R15 — area 0 (backbone)

The topology crashed and EVE-NG froze, so reconfiguring everything from scratch.

<details>
<summary>R14</summary>
<pre><code>
enable
conf t
hos R14
ipv6 unic
int e0/0
 ipv6 enable
 ipv6 addr 2002:acad:db8:4::14/64
 ip addr 10.10.10.17 255.255.255.252
 no shut
int e0/1
 ipv6 enable
 ipv6 addr 2002:acad:db8:5::14/64
 ip addr 10.10.10.21 255.255.255.252
 no shut
int e0/2
 ipv6 enable
 ipv6 addr 2002:acad:db8:6::14/64
 ip addr 100.100.100.1 255.255.255.252
 no shut
int e0/3
 ipv6 enable
 ipv6 addr 2002:acad:db8:7::14/64
 ip addr 10.10.10.13 255.255.255.252
 no shut
no ip domain-lookup
line con 0
 exec-t 0 0
exit

router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 network 10.10.10.12 0.0.0.3 area 101
 network 10.10.10.16 0.0.0.3 area 0
 network 10.10.10.20 0.0.0.3 area 0
 default-information originate

ip route 0.0.0.0 0.0.0.0 100.100.100.2
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R15</summary>
<pre><code>
enable
conf t
hos R15
ipv6 unic
int e0/0
 ipv6 enable
 ipv6 addr 2002:acad:db8:0::15/64
 ip addr 10.10.10.5 255.255.255.252
 no shut
int e0/1
 ipv6 enable
 ipv6 addr 2002:acad:db8:1::15/64
 ip addr 10.10.10.9 255.255.255.252
 no shut
int e0/2
 ipv6 enable
 ipv6 addr 2002:acad:db8:2::15/64
 ip addr 111.111.111.1 255.255.255.252
 no shut
int e0/3
 ipv6 enable
 ipv6 addr 2002:acad:db8:3::15/64
 ip addr 10.10.10.1 255.255.255.252
 no shut

router ospf 1
 router-id 15.15.15.15
 network 10.10.10.0 0.0.0.3 area 102
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.8 0.0.0.3 area 0
 distribute-list prefix PL1 in
 default-information originate

ip route 0.0.0.0 0.0.0.0 111.111.111.2

ip prefix-list PL1 seq 5 deny 10.10.10.12/30
ip prefix-list PL1 seq 10 permit 0.0.0.0/0 le 32
end
copy running-config startup-config
</code></pre>
</details>


## R12–R13 — area 10 (default route required)

<details>
<summary>R12</summary>
<pre><code>
enable
conf t
hos R12
ipv6 unic
int e0/0
 ip addr 172.16.0.1 255.255.255.0
 no shut
 ipv6 enable
 ipv6 addr 2002:acad:db8:8::12/64
int e0/3
 ip addr 10.10.10.10 255.255.255.252
 no shut
 ipv6 enable
 ipv6 addr 2002:acad:db8:1::12/64
int e0/2
 ip addr 10.10.10.18 255.255.255.252
 no shut
 ipv6 enable
 ipv6 addr 2002:acad:db8:4::12/64

router ospf 1
 router-id 12.12.12.12
 network 10.10.10.8 0.0.0.3 area 0
 network 10.10.10.16 0.0.0.3 area 0
 network 172.16.0.0 0.0.0.255 area 10
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R13</summary>
<pre><code>
enable
conf t
hos R13
ipv6 unic
int e0/0
 ipv6 enable
 ipv6 addr 2002:acad:db8:9::13/64
 ip addr 172.16.1.1 255.255.255.0
 no shut
int e0/2
 ipv6 enable
 ipv6 addr 2002:acad:db8:0::13/64
 ip addr 10.10.10.6 255.255.255.252
 no shut
int e0/3
 ipv6 enable
 ipv6 addr 2002:acad:db8:5::13/64
 ip addr 10.10.10.22 255.255.255.252
 no shut

router ospf 1
 router-id 13.13.13.13
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.20 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.255 area 10
end
copy running-config startup-config
</code></pre>
</details>

Both routers receive a default route — confirmed in routing table:

<details>
<summary>R12 — show ip route ospf</summary>
<pre><code>
R12#show ip route ospf
Gateway of last resort is 10.10.10.17 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.10.10.17, 02:22:36, Ethernet0/2
                [110/1] via 10.10.10.9, 02:21:41, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O IA     10.10.10.0/30 [110/20] via 10.10.10.9, 05:30:07, Ethernet0/3
O        10.10.10.4/30 [110/20] via 10.10.10.9, 05:30:07, Ethernet0/3
O IA     10.10.10.12/30 [110/20] via 10.10.10.17, 05:30:07, Ethernet0/2
O        10.10.10.20/30 [110/20] via 10.10.10.17, 05:30:07, Ethernet0/2
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
O IA     172.16.1.0/24 [110/30] via 10.10.10.17, 05:29:40, Ethernet0/2
                       [110/30] via 10.10.10.9, 05:29:40, Ethernet0/3
</code></pre>
</details>

<details>
<summary>R13 — show ip route ospf</summary>
<pre><code>
R13#show ip route ospf
Gateway of last resort is 10.10.10.21 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.10.10.21, 02:24:20, Ethernet0/3
                [110/1] via 10.10.10.5, 02:23:25, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O IA     10.10.10.0/30 [110/20] via 10.10.10.5, 05:31:25, Ethernet0/2
O        10.10.10.8/30 [110/20] via 10.10.10.5, 05:31:25, Ethernet0/2
O IA     10.10.10.12/30 [110/20] via 10.10.10.21, 05:31:25, Ethernet0/3
O        10.10.10.16/30 [110/20] via 10.10.10.21, 05:31:25, Ethernet0/3
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
O IA     172.16.0.0/24 [110/30] via 10.10.10.21, 05:31:25, Ethernet0/3
                       [110/30] via 10.10.10.5, 05:31:25, Ethernet0/2
</code></pre>
</details>


## R19 — area 101 (default route only)

Area 101 is configured as `stub no-summary` on R14, which suppresses all LSA type 3 advertisements — R19 receives only the default route injected by the ABR.

<details>
<summary>R19</summary>
<pre><code>
enable
conf t
hos R19
ipv6 unic
int e0/0
 ip addr 10.10.10.14 255.255.255.252
 ipv6 enable
 ipv6 addr 2002:acad:db8:7::19/64
 no shut

router ospf 1
 router-id 19.19.19.19
 area 101 stub
 network 10.10.10.12 0.0.0.3 area 101
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R19 — show ip route ospf</summary>
<pre><code>
R19#show ip route ospf
Gateway of last resort is 10.10.10.13 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.10.10.13, 18:31:11, Ethernet0/0
</code></pre>
</details>


## R20 — area 102 (all routes except area 101)

R15 uses a prefix-list `PL1` with `distribute-list in` to block `10.10.10.12/30` (area 101) from being installed in its own routing table, and thus it is not redistributed further into area 102.

<details>
<summary>R20</summary>
<pre><code>
enable
conf t
hos R20
ipv6 unic
int e0/0
 ip addr 10.10.10.2 255.255.255.252
 ipv6 enable
 ipv6 addr 2002:acad:db8:3::20/64
 no shut
no ip domain-loo
line con 0
 exec-t 0 0

router ospf 1
 router-id 20.20.20.20
 network 10.10.10.0 0.0.0.3 area 102
end
copy running-config startup-config
</code></pre>
</details>

<details>
<summary>R20 — show ip route ospf</summary>
<pre><code>
R20#show ip route ospf
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.10.10.4/30 [110/20] via 10.10.10.1, 00:31:46, Ethernet0/0
O IA     10.10.10.8/30 [110/20] via 10.10.10.1, 00:31:46, Ethernet0/0
O IA     10.10.10.16/30 [110/30] via 10.10.10.1, 00:31:46, Ethernet0/0
O IA     10.10.10.20/30 [110/30] via 10.10.10.1, 00:31:46, Ethernet0/0
      172.16.0.0/24 is subnetted, 2 subnets
O IA     172.16.0.0 [110/30] via 10.10.10.1, 00:31:46, Ethernet0/0
O IA     172.16.1.0 [110/30] via 10.10.10.1, 00:31:46, Ethernet0/0
</code></pre>
</details>

The 10.10.10.12/30 network (area 101) is absent from the table — objective achieved.


*Network Engineer Course | Lab 09*

---

[← 08. PBR](PBR.md) | [🏠 Home](../README.md) | [10. OSPFv3 →](OSPFv3.md)
