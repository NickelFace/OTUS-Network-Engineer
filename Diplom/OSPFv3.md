Работа протокола OSPF для IPv6

## Домашнее задание

### OSPF для IPv6

Цель: Настроить OSPF для IPv6, сохранив ту же логику работы, что у OSPF для IPv4

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите OSPF для IPv6. сохранив ту же логику работы(метрики, таймеры, фильтры), что OSPF для IPv4
2. План работы и изменения зафиксированы в документации

![OSPF](img/OSPF.png)



В прошлый раз я забыл прописать Link-Local  адресацию ,поэтому исправляюсь .

**R14**

```
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

```



**R15**

```
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

```



**R12**

```
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
```

**R13**

```
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
```



**R19**

```
ipv6 router ospf 1
 router-id 19.19.19.19
 area 101 stub

interface Ethernet0/0
 ipv6 address FE80:6::19 link-local
 ipv6 ospf 1 area 101
 
```

Проверим какие получает маршруты R19

```
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
```

**R20**

```
ipv6 router ospf 1
 router-id 20.20.20.20

interface Ethernet0/0
 ipv6 address FE80:3::20 link-local
 ipv6 ospf 1 area 102
```

Проверим какие получает маршруты R20

```
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

```

Как видно ,внешнюю сеть **2002:ACAD:DB8:7::/64** мы не получаем





Решил проверить разницу в таймерах (и метриках) метриках и не нашел отличий,поэтому я просто ничего не стал менять 

```
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
  Graceful restart helper support enabled
  Index 1/2/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 3, maximum is 5
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 13.13.13.13  (Designated Router)
  Suppress hello for 0 neighbor(s)
  
R15#show ip ospf interface e0/0
Ethernet0/0 is up, line protocol is up
  Internet Address 10.10.10.5/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 15.15.15.15, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 15.15.15.15, Interface address 10.10.10.5
  Backup Designated router (ID) 13.13.13.13, Interface address 10.10.10.6
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:02
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/2, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 2, maximum is 4
  Last flood scan time is 0 msec, maximum is 1 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 13.13.13.13  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)

```

