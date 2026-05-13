[🇷🇺 Русская версия](README.ru.md)

# Laboratory work. Configuring the basic OSPFv2 protocol for one area

## Topology

![OSPF](img/OSPFv2.png)

## Addressing table

| Device | Interface    | IP address   | Subnet Mask     | Gateway     |
| ------ | ------------ | ------------ | --------------- | ----------- |
| R1     | G0/0         | 192.168.1.1  | 255.255.255.0   | —           |
|        | S0/0/0 (DCE) | 192.168.12.1 | 255.255.255.252 | —           |
|        | S0/0/1       | 192.168.13.1 | 255.255.255.252 | —           |
| R2     | G0/0         | 192.168.2.1  | 255.255.255.0   | —           |
|        | S0/0/0       | 192.168.12.2 | 255.255.255.252 | —           |
|        | S0/0/1 (DCE) | 192.168.23.1 | 255.255.255.252 | —           |
| R3     | G0/0         | 192.168.3.1  | 255.255.255.0   | —           |
|        | S0/0/0 (DCE) | 192.168.13.2 | 255.255.255.252 | —           |
|        | S0/0/1       | 192.168.23.2 | 255.255.255.252 | —           |
| PC-A   | NIC          | 192.168.1.3  | 255.255.255.0   | 192.168.1.1 |
| PC-B   | NIC          | 192.168.2.3  | 255.255.255.0   | 192.168.2.1 |
| PC-C   | NIC          | 192.168.3.3  | 255.255.255.0   | 192.168.3.1 |

## Tasks

Part 1. Creating a network and configuring the basic parameters of the device

Part 2. Configuring and verifying OSPF Routing

Part 3. Changing the purpose of router IDs

Part 4. Configuring Passive OSPF Interfaces

Part 5. Changing the OSPF metric



### Creating a network and configuring basic device parameters

<details>
<summary>R1</summary>
<pre><code>
Enable
Configure terminal
!
no ip domain-lookup
hos R1
!
enable secret class 
line vty 0 4 
logging synchronous 
password cisco 
login 
exit
!
line con 0
exec-t 0 0
logging synchronous 
password cisco 
login 
exit 
!
Banner motd "This is a secure system. Authorized Access Only!"
!
int e0/0
ip addr 192.168.1.1 255.255.255.0
no shut
exit
!
int se1/0
 clock rate 128000 
ip addr 192.168.12.1 255.255.255.252
no shut
exit
!
int se1/1
ip addr 192.168.13.1 255.255.255.252
no shut
exit
!
end
wr
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
Enable
Configure terminal
!
hos R2
!
no ip domain-lookup
enable secret class 
!
line vty 0 4 
logging synchronous 
password cisco 
login 
exit
! 
line con 0 
exec-t 0 0
logging synchronous 
password cisco 
login 
exit 
!
Banner motd "This is a secure system. Authorized Access Only!"
!
int se1/0
ip addr 192.168.12.2 255.255.255.252
no shut
exit
!
int se1/1
 clock rate 128000 
ip addr 192.168.23.1 255.255.255.252
no shut
exit
!
int e 0/0
ip addr 192.168.2.1 255.255.255.0
no shut
exit
!
end
write
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
Enable
Configure terminal
!
no ip domain-lookup
hos R3
!
enable secret class 
line vty 0 4 
logging synchronous 
password cisco 
login 
exit 
!
line con 0 
exec-t 0 0
logging synchronous 
password cisco 
login 
exit 
Banner motd "This is a secure system. Authorized Access Only!"
!
int se1/0
 clock rate 128000 
ip addr 192.168.13.2 255.255.255.252
no shut
exit
!
int se 1/1
ip addr 192.168.23.2 255.255.255.252
no shut
exit
!
int e0/0
ip addr 192.168.3.1 255.255.255.0
no shut
exit
!
do write
</code></pre>
</details>
<details>
<summary>PC-A</summary>
<pre><code>
set pcname PC-A
ip 192.168.1.3 24 192.168.1.1
</code></pre>
</details>
<details>
<summary>PC-B</summary>
<pre><code>
set pcname PC-B
ip 192.168.2.3 24 192.168.2.1
</code></pre>
</details>
<details>
<summary>PC-C</summary>
<pre><code>
set pcname PC-C
ip 192.168.3.3 24 192.168.3.1
</code></pre>
</details>


### Configuring and verifying OSPF routing

Configure the OSPF protocol on routers:

<details>
<summary>R1</summary>
<pre><code>
router ospf 1
network 192.168.1.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.3 area 0
network 192.168.13.0 0.0.0.3 area 0
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
router ospf 1
network 192.168.2.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
router ospf 1
network 192.168.3.0 0.0.0.255 area 0
network 192.168.13.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
</code></pre>
</details>

Let's check the information about neighboring devices and OSPF routing

```
show ip ospf neighbor
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.23.2      0   FULL/  -        00:00:34    192.168.13.2    Serial1/1
192.168.23.1      0   FULL/  -        00:00:39    192.168.12.2    Serial1/0
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
R2(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.23.2      0   FULL/  -        00:00:39    192.168.23.2    Serial1/1
192.168.13.1      0   FULL/  -        00:00:35    192.168.12.1    Serial1/0
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
R3(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.23.1      0   FULL/  -        00:00:35    192.168.23.1    Serial1/1
192.168.13.1      0   FULL/  -        00:00:32    192.168.13.1    Serial1/0
</code></pre>
</details>


**show ip route**

<details>
<summary>R1</summary>
<pre><code>
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override
!
Gateway of last resort is not set
!
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:05:51, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:05:41, Serial1/1
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/30 is directly connected, Serial1/0
L        192.168.12.1/32 is directly connected, Serial1/0
      192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/30 is directly connected, Serial1/1
L        192.168.13.1/32 is directly connected, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:05:31, Serial1/1
                      [110/128] via 192.168.12.2, 00:05:41, Serial1/0
</code></pre>
</details>

**show ip protocols** 

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip protocols
*** IP Routing is NSF aware ***
!
Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)
!
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 192.168.13.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.23.2         110      00:02:20
    192.168.23.1         110      00:02:30
  Distance: (default is 110)
</code></pre>
</details>

**show ip ospf**

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf
 Routing Process "ospf 1" with ID 192.168.13.1
 Start time: 00:42:09.100, Time elapsed: 00:03:58.496
 Supports only single TOS(TOS0) routes
 Supports opaque LSA
 Supports Link-local Signaling (LLS)
 Supports area transit capability
 Supports NSSA (compatible with RFC 3101)
 Supports Database Exchange Summary List Optimization (RFC 5243)
 Event-log enabled, Maximum number of events: 1000, Mode: cyclic
 Router is not originating router-LSAs with maximum metric
 Initial SPF schedule delay 5000 msecs
 Minimum hold time between two consecutive SPFs 10000 msecs
 Maximum wait time between two consecutive SPFs 10000 msecs
 Incremental-SPF disabled
 Minimum LSA interval 5 secs
 Minimum LSA arrival 1000 msecs
 LSA group pacing timer 240 secs
 Interface flood pacing timer 33 msecs
 Retransmission pacing timer 66 msecs
 Number of external LSA 0. Checksum Sum 0x000000
 Number of opaque AS LSA 0. Checksum Sum 0x000000
 Number of DCbitless external and opaque AS LSA 0
 Number of DoNotAge external and opaque AS LSA 0
 Number of areas in this router is 1. 1 normal 0 stub 0 nssa
 Number of areas transit capable is 0
 External flood list length 0
 IETF NSF helper support enabled
 Cisco NSF helper support enabled
 Reference bandwidth unit is 100 mbps
    Area BACKBONE(0)
        Number of interfaces in this area is 3
        Area has no authentication
        SPF algorithm last executed 00:03:09.979 ago
        SPF algorithm executed 4 times
        Area ranges are
        Number of LSA 3. Checksum Sum 0x019D46
        Number of opaque link LSA 0. Checksum Sum 0x000000
        Number of DCbitless LSA 0
        Number of indication LSA 0
        Number of DoNotAge LSA 0
        Flood list length 0
</code></pre>
</details>

**show ip ospf interface brief** 

<details>
<summary>R1</summary>
<pre><code>
R1#show ip ospf interface brief
!
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/1           1     0                   192.168.13.1/30      64      P2P   1/1
Se1/0           1     0                   192.168.12.1/30      64      P2P   1/1
Et0/0            1     0                   192.168.1.1/24        10      DR    0/0
</code></pre>
</details>

**Show ip ospf interface**

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do Show ip ospf interface
Serial1/1 is up, line protocol is up
  Internet Address 192.168.13.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 192.168.13.1, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:08
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/3/3, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 192.168.23.2
  Suppress hello for 0 neighbor(s)
Serial1/0 is up, line protocol is up
  Internet Address 192.168.12.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 192.168.13.1, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:05
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/2/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 192.168.23.1
  Suppress hello for 0 neighbor(s)
Ethernet0/0 is up, line protocol is up
  Internet Address 192.168.1.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 192.168.13.1, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 192.168.13.1, Interface address 192.168.1.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:04
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
</code></pre>
</details>

We check the operation of the OSPF protocol using ping

<details>
<summary>PC-A</summary>
<pre><code>
PC-A> ping 192.168.2.3
!
84 bytes from 192.168.2.3 icmp_seq=1 ttl=62 time=10.127 ms
84 bytes from 192.168.2.3 icmp_seq=2 ttl=62 time=7.261 ms
84 bytes from 192.168.2.3 icmp_seq=3 ttl=62 time=10.823 ms
84 bytes from 192.168.2.3 icmp_seq=4 ttl=62 time=10.724 ms
84 bytes from 192.168.2.3 icmp_seq=5 ttl=62 time=11.083 ms
!
PC-A> ping 192.168.3.3
!
84 bytes from 192.168.3.3 icmp_seq=1 ttl=62 time=11.863 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=62 time=10.602 ms
84 bytes from 192.168.3.3 icmp_seq=3 ttl=62 time=10.919 ms
84 bytes from 192.168.3.3 icmp_seq=4 ttl=62 time=10.393 ms
84 bytes from 192.168.3.3 icmp_seq=5 ttl=62 time=10.881 ms
</code></pre>
</details>

###   Changing assigned Router IDs

If there is no RouterID, then it will take it from the loopback interface. Let's check it out! 

<details>
<summary>R1</summary>
<pre><code>
interface lo0
ip address 1.1.1.1 255.255.255.255
do wr
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
interface lo0
ip address 2.2.2.2 255.255.255.255
do wr
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
interface lo0
ip address 3.3.3.3 255.255.255.255
do wr
</code></pre>
</details>

```
clear ip ospf process
```

Let's see what has changed

```
show ip protocols
```

<details>
<summary>R1</summary>
<pre><code>
R1(config)#do show ip protocols
*** IP Routing is NSF aware ***
!
Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)
!
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    3.3.3.3              110      00:07:22
    2.2.2.2              110      00:07:36
    192.168.23.2         110      00:07:56
    192.168.23.1         110      00:07:56
  Distance: (default is 110)
</code></pre>
</details>

```
show ip ospf neighbor
```

<details>
<summary>R1</summary>
<pre><code>
R1(config)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:37    192.168.13.2    Serial1/1
2.2.2.2           0   FULL/  -        00:00:37    192.168.12.2    Serial1/0
</code></pre>
</details>

Change the ID using Router id now to:

| R1          | R2          | R3          |
| ----------- | ----------- | ----------- |
| 11.11.11.11 | 22.22.22.22 | 33.33.33.33 |

<details>
<summary>R1</summary>
<pre><code>
router ospf 1
router-id 11.11.11.11
do clear ip ospf process
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
router ospf 1
router-id 22.22.22.22
do clear ip ospf process
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
router ospf 1
router-id 33.33.33.33
do clear ip ospf process
</code></pre>
</details>

```
show ip protocols
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip protocols
*** IP Routing is NSF aware ***
!
Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)
!
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 11.11.11.11
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    33.33.33.33          110      00:01:55
    22.22.22.22          110      00:02:34
    3.3.3.3              110      00:03:25
    2.2.2.2              110      00:03:25
    192.168.23.2         110      00:22:32
    192.168.23.1         110      00:22:32
  Distance: (default is 110)
</code></pre>
</details>

```
show ip ospf neighbor
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:32    192.168.13.2    Serial1/1
22.22.22.22       0   FULL/  -        00:00:31    192.168.12.2    Serial1/0
</code></pre>
</details>

###   Configuring Passive OSPF Interfaces

Let's check the interface that goes towards the PC

```
show ip ospf interface e0/0
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 192.168.1.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 11.11.11.11, Interface address 192.168.1.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:07
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
</code></pre>
</details>

```
R1(config)# router ospf 1
R1(config-router)# passive-interface e0/0
```

<details>
<summary>R1</summary>
<pre><code>
  R1(config-router)#router ospf 1
  R1(config-router)#passive-interface e0/0
</code></pre>
</details>

Run the **show ip ospf interface e0/0** command again to make sure that the E0/0 interface has become passive.

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 192.168.1.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State WAITING, Priority 1
  No designated router on this network
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface)
    Wait time before Designated router selection 00:00:30
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
</code></pre>
</details>
Enter the command **show ip route** on routers R2 and R3 to make sure that the route to the 192.168.1.0/24 network remains available.

<details>
<summary>R2</summary>
<pre><code>
R2(config-router)#do show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override
!
Gateway of last resort is not set
!
      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback0
O     192.168.1.0/24 [110/74] via 192.168.12.1, 00:14:23, Serial1/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, Ethernet0/0
L        192.168.2.1/32 is directly connected, Ethernet0/0
O     192.168.3.0/24 [110/74] via 192.168.23.2, 00:13:44, Serial1/1
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/30 is directly connected, Serial1/0
L        192.168.12.2/32 is directly connected, Serial1/0
      192.168.13.0/30 is subnetted, 1 subnets
O        192.168.13.0 [110/128] via 192.168.23.2, 00:13:44, Serial1/1
                      [110/128] via 192.168.12.1, 00:14:23, Serial1/0
      192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.23.0/30 is directly connected, Serial1/1
L        192.168.23.1/32 is directly connected, Serial1/1
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
R3(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override
!
Gateway of last resort is not set
!
      3.0.0.0/32 is subnetted, 1 subnets
C        3.3.3.3 is directly connected, Loopback0
O     192.168.1.0/24 [110/74] via 192.168.13.1, 00:14:23, Serial1/0
O     192.168.2.0/24 [110/74] via 192.168.23.1, 00:14:23, Serial1/1
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Ethernet0/0
L        192.168.3.1/32 is directly connected, Ethernet0/0
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/128] via 192.168.23.1, 00:14:23, Serial1/1
                      [110/128] via 192.168.13.1, 00:14:23, Serial1/0
      192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/30 is directly connected, Serial1/0
L        192.168.13.2/32 is directly connected, Serial1/0
      192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.23.0/30 is directly connected, Serial1/1
L        192.168.23.2/32 is directly connected, Serial1/1
</code></pre>
</details>

###  Configure the passive interface on the router as the default interface.

command **show ip ospf neighbor** on router R1 to make sure that R2 is listed as an OSPF neighbor device

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:35    192.168.13.2    Serial1/1
22.22.22.22       0   FULL/  -        00:00:38    192.168.12.2    Serial1/0
</code></pre>
</details>
<details>
<summary>R2</summary>
<pre><code>
R2(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:33    192.168.23.2    Serial1/1
11.11.11.11       0   FULL/  -        00:00:37    192.168.12.1    Serial1/0
</code></pre>
</details>
<details>
<summary>R3</summary>
<pre><code>
R3(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
22.22.22.22       0   FULL/  -        00:00:33    192.168.23.1    Serial1/1
11.11.11.11       0   FULL/  -        00:00:32    192.168.13.1    Serial1/0
</code></pre>
</details>

Configure all interfaces as passive by default

```
router ospf 1
passive-interface default
exit
```
<details>
<summary>R1,R2,R3</summary>
<pre><code>
router ospf 1
passive-interface default
exit
</code></pre>
</details>


```
do show ospf neighbor
do show ip ospf interface s1/0
do show ip ospf interface s1/1
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#
*Nov  1 22:16:08.724: %OSPF-5-ADJCHG: Process 1, Nbr 22.22.22.22 on Serial1/0 from FULL to DOWN, Neighbor Down: Interface down or detached
*Nov  1 22:16:08.724: %OSPF-5-ADJCHG: Process 1, Nbr 33.33.33.33 on Serial1/1 from FULL to DOWN, Neighbor Down: Interface down or detached
!
R1(config-router)#do show ospf neighbor
!
R1(config-router)#do show ip ospf interface s1/0
!
Serial1/0 is up, line protocol is up
  Internet Address 192.168.12.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface)
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/2/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
!
R1(config-router)#do show ip ospf interface s1/1
!
Serial1/1 is up, line protocol is up
  Internet Address 192.168.13.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 64
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           64        no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface)
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/3/3, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0
  Suppress hello for 0 neighbor(s)
</code></pre>
</details>

Turn on the interfaces connected to the neighbors

```
router ospf 1
no passive-interface serial 1/0
no passive-interface serial 1/1
exit
```

<details>
<summary>R1,R2,R3</summary>
<pre><code>
router ospf 1
no passive-interface serial 1/0
no passive-interface serial 1/1
exit
</code></pre>
</details>

Run the commands again **show ip route** and **show ip ospf neighbor** 

<details>
<summary>R1</summary>
<pre><code>
!
R1(config-router)#do show ip ospf neighbor
!
Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:36    192.168.13.2    Serial1/1
22.22.22.22       0   FULL/  -        00:00:36    192.168.12.2    Serial1/0
!
!
!
R1(config-router)#do show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override
!
Gateway of last resort is not set
!
      1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:00:53, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:00:43, Serial1/1
      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/30 is directly connected, Serial1/0
L        192.168.12.1/32 is directly connected, Serial1/0
      192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/30 is directly connected, Serial1/1
L        192.168.13.1/32 is directly connected, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:00:43, Serial1/1
                      [110/128] via 192.168.12.2, 00:00:53, Serial1/0
</code></pre>
</details>

The network has been restored, the full neighborhood has been preserved.

### Changing the OSPF metric

The default reference bandwidth for OSPF is 100 Mbps (Fast Ethernet speed). But the channel speed in most modern network infrastructure devices exceeds 100 Mbit/s. Since the OSPF cost metric must be an integer, the cost for all channels with a transfer rate of 100 Mbit/s and higher is 1. Therefore, the Fast Ethernet, Gigabit Ethernet and 10G Ethernet interfaces have the same cost. Therefore, to account for networks with channels whose speed exceeds 100 Mbit/s, a higher value of the reference bandwidth is necessary.

Run the **show interface** command on the R1 router to view the default bandwidth value for the interface `Serial1/0`

```
R1#show interface Serial1/0
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-router)#do show inter s1/0
Serial1/0 is up, line protocol is up
  Hardware is M4T
  Internet address is 192.168.12.1/30
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
....
</code></pre>
</details>

Enter the command **show ip route ospf** on the router R1 to determine the route to the network 192.168.3.0/24

<details>
<summary>R1</summary>
<pre><code>
R1#show ip route ospf 
!
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:12:40, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:12:30, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:12:30, Serial1/1
                      [110/128] via 192.168.12.2, 00:12:40, Serial1/0
</code></pre>
</details>

The cost is 74

Let's check the path to 192.168.3.1/24

<details>
<summary>R1</summary>
<pre><code>
R1#traceroute 192.168.3.1
Type escape sequence to abort.
Tracing the route to 192.168.3.1
VRF info: (vrf in name/id, vrf out name/id)
  1 192.168.13.2 10 msec *  9 msec
</code></pre>
</details>

```
R1 --> 192.168.13.2(R3) --> 192.168.3.1(R3)
```

Run the **show ip ospf interface Serial1/1** command on the R1 router to see the routing cost for the Serial1/1 interface.

<details>
<summary>R1</summary>
<pre><code>
R1#show ip ospf interface Serial1/1
Serial1/1 is up, line protocol is up
  Internet Address 192.168.13.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 64
...
</code></pre>
</details>

Run the **show ip ospf interface e0/0** command on the R3 router to determine the routing cost for the e0/0 interface.

<details>
<summary>R3</summary>
<pre><code>
R3(config-router)#do show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 192.168.3.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 33.33.33.33, Network Type BROADCAST, Cost: 10
...
</code></pre>
</details>
R3 is the owner of the network 192.168.3.0/24 and the cost is 10

As can be seen from the results of the **show ip route** command, the sum of the cost metrics of these two interfaces is the total cost of the route to the 192.168.3.0/24 network for the R3 router, calculated by the formula 10 + 64 = 74.


To change the default reference bandwidth parameter, run the command **auto-cost reference-bandwidth 10000** on router R1. With this parameter, the cost of 10 Gbit/s interfaces will be 1, the cost of 1 Gbit/s interfaces will be 10, and the cost of 100 Mbit/s interfaces will be 100

<details>
<summary>R1,R2,R3</summary>
<pre><code>
router ospf 1
 auto-cost reference-bandwidth 10000
 exit
...
</code></pre>
</details>

Let's see how the cost has changed:

<details>
<summary>R1,R2,R3</summary>
<pre><code>
R1(config-router)#do sh ip route ospf
!
O     192.168.2.0/24 [110/7476] via 192.168.12.2, 00:01:52, Serial1/0
O     192.168.3.0/24 [110/7476] via 192.168.13.2, 00:01:42, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/12952] via 192.168.13.2, 00:01:42, Serial1/1
                      [110/12952] via 192.168.12.2, 00:01:42, Serial1/0
!
!
R1(config-router)#do show ip ospf interface Serial1/1
Serial1/1 is up, line protocol is up
  Internet Address 192.168.13.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 6476
!
</code></pre>
</details>

Of course, the cost has changed and now it has not become clearer. It is recommended to change the reference cost only if there are interfaces with a bandwidth of more than 100 Mbit/s.

To restore the default value for the reference bandwidth, run the command **auto-cost reference-bandwidth 100** on all three routers.

```
router ospf 1
auto-cost reference-bandwidth 100
exit
```

<details>
<summary>R1</summary>
<pre><code>
R1#show ip route ospf 
!
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:12:40, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:12:30, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:12:30, Serial1/1
                      [110/128] via 192.168.12.2, 00:12:40, Serial1/0
</code></pre>
</details>

### Change the bandwidth for the interface.

```
show ip route ospf
```

<details>
<summary>R1</summary>
<pre><code>
show ip route ospf
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:06:15, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:05:58, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:05:58, Serial1/1
                      [110/128] via 192.168.12.2, 00:06:15, Serial1/0
</code></pre>
</details>

```
show interface Serial1/1
```

<details>
<summary>R1</summary>
<pre><code>
R1#show interface Serial1/1
Serial1/1 is up, line protocol is up
  Hardware is M4T
  Internet address is 192.168.13.1/30
  MTU 1500 bytes, BW 1544 Kbit/sec, DLY 20000 usec
</code></pre>
</details>

100Mbit / 1.544 Mbit = 64

The bandwidth changes on the interfaces, so we'll change the cost of 1544 to 2 on the Serial 1/1 interface and see how the route metric changes.

<details>
<summary>R1</summary>
<pre><code>
interface serial 1/1
bandwidth 2
exit
</code></pre>
</details>

```
R1(config)#do show ip ospf interface Serial 1/1
Serial1/1 is up, line protocol is up
  Internet Address 192.168.13.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type POINT_TO_POINT, Cost: 50000
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           50000     no          no            Base
```

The cost has become very expensive and the route will become unprofitable, let's look at the routing table to make sure of this.

```
show ip route ospf
```

<details>
<summary>R1</summary>
<pre><code>
show ip route ospf
!
O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:30:20, Serial1/0
O     192.168.3.0/24 [110/138] via 192.168.12.2, 00:02:17, Serial1/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.12.2, 00:02:17, Serial1/0
</code></pre>
</details>


As you can see, the route through 192.168.13.0/30 has become unprofitable, At the moment Se 1/0 is a more optimal route.

```
R1(config)#do show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/1        1     0               192.168.13.1/30    50000 P2P   1/1
Se1/0        1     0               192.168.12.1/30    64    P2P   1/1
Et0/0        1     0               192.168.1.1/24     10    DR    0/0
```

Let's put everything back in its place

```
R1(config)#int se1/1
R1(config-if)#no bandwidth
R1(config-if)#do show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/1        1     0               192.168.13.1/30    64    P2P   1/1
Se1/0        1     0               192.168.12.1/30    64    P2P   1/1
Et0/0        1     0               192.168.1.1/24     10    DR    0/0
```

###  Change the cost of the route

OSPF uses the bandwidth value to calculate the channel cost by default. But this calculation can be changed by manually setting the cost of the channel using the **ip ospf cost** command. 

<details>
<summary>R1</summary>
<pre><code>
show ip route ospf
O     192.168.2.0/24 [110/74] via 192.168.12.2, 01:19:47, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:41:10, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:41:10, Serial1/1
                      [110/128] via 192.168.12.2, 00:51:44, Serial1/0
</code></pre>
</details>

Execute the command **ip ospf cost 1565** for the Serial 1/1 interface of the router R1. The cost of 1565 is higher than the total cost of the route passing through router R2 (1562).

```
R1(config)# interface Serial1/1
R1(config-if)# ip ospf cost 1565
```

<details>
<summary>R1</summary>
<pre><code>
R1(config-if)#do show ip route ospf
!
O     192.168.2.0/24 [110/74] via 192.168.12.2, 01:24:13, Serial1/0
O     192.168.3.0/24 [110/138] via 192.168.12.2, 00:02:08, Serial1/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.12.2, 00:56:10, Serial1/0
</code></pre>
</details>
As you can see, all routes with Serial1/1 are gone 

This happened because the router recalculated the routes and decided that it was more cost-effective to drive traffic through Serial1/0 than through Serial1/1. The route has not disappeared anywhere, he has memorized it and keeps it in case of Serial1/0 failure.

Questions to repeat

1. Why is it so important to manage the assignment of the router ID when using the OSPF protocol?

   Because on the basis of the Router-ID, DR and BDR elections take place, and also gives the System Administrator a simplified understanding of the network. (no need to remember which Loopback is registered there and which networks it knows and is registered on the interfaces)

2. Why is the DR/BDR selection process not considered in this laboratory work?

Because point-to-point connections, LSA type 2 cannot run in such a network, and therefore DR and BDR can be selected.

3. Why is it recommended to configure the OSPF interface as passive?

Reducing network load , as well as security . So that from the side where routers should not be, the attacker could not change the topology and implement his devices.
---

[← 03. HSRP](../EtherChannel/HSRP.md) | [🏠 Home](../README.md) | [05. OSPF Multiarea →](Multiarea.md)
