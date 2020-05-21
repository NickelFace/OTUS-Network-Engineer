# Router reflector

![image-20200521142853726](C:\Users\lopunov\Documents\GitHub\OTUS_Network\BGP\img\1.png)

## Конфигурация роутеров

### R1

```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 1 area 0
!
interface Loopback1
 ip address 100.100.100.100 255.255.255.255
!
interface Ethernet0/0
 ip address 10.1.2.1 255.255.255.0
!
interface Ethernet0/1
 ip address 10.1.3.1 255.255.255.0
 ip ospf 1 area 0
!
router ospf 1
 router-id 1.1.1.1
!
router bgp 65000
 bgp log-neighbor-changes
 network 100.100.100.100 mask 255.255.255.255
 neighbor 3.3.3.3 remote-as 65000
 neighbor 3.3.3.3 update-source Loopback0
 neighbor 3.3.3.3 route-reflector-client
 neighbor 4.4.4.4 remote-as 65000
 neighbor 4.4.4.4 update-source Loopback0
 neighbor 4.4.4.4 route-reflector-client

```

### R3

```
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
 ip ospf 1 area 0
!
interface Loopback1
 ip address 200.200.200.200 255.255.255.255
!
interface Ethernet0/0
 ip address 10.3.4.3 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.1.3.3 255.255.255.0
 ip ospf 1 area 0

router ospf 1
 router-id 3.3.3.3
!
router bgp 65000
 bgp log-neighbor-changes
 network 200.200.200.200 mask 255.255.255.255
 neighbor 1.1.1.1 remote-as 65000
 neighbor 1.1.1.1 update-source Loopback0

```

### R4

```
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 ip address 10.3.4.4 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.2.4.4 255.255.255.0
!
router ospf 1
 router-id 4.4.4.4
!
router bgp 65000
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 65000
 neighbor 1.1.1.1 update-source Loopback0

```

