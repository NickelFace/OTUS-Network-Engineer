# BGP. Основы

## Цель: Настроить BGP между автономными системами Организовать доступность между офисами Москва и С.-Петербург

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
2. Настроите eBGP между провайдерами Киторн и Ламас
3.  Настроите eBGP между Ламас и Триада
4. eBGP между офисом С.-Петербург и провайдером Триада
5. Организуете IP доступность между офисами Москва и С.-Петербург
6. Настроите отслеживание линка через технологию IP SLA
7. План работы и изменения зафиксированы в документации

![EVE_Topology](img/EVE_Topology.png)

Решил добавить линк между  R14 и R15 ,чтобы  проще соединить их в area 0  ,во вторых ,чтобы обеспечить отказоустойчивость при прохождении пакета через интернет.

### eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас

R14 

```
router bgp 1001
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 neighbor 10.10.10.26 remote-as 1001
 neighbor 100.100.100.2 remote-as 101
```

```
R14(config-router)#do sh ip bgp sum
BGP router identifier 100.100.100.1, local AS number 1001
BGP table version is 34, main routing table version 34
10 network entries using 1440 bytes of memory
17 path entries using 1360 bytes of memory
8/6 BGP path/bestpath attribute entries using 1216 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4160 total bytes of memory
BGP activity 12/2 prefixes, 23/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.26     4         1001     185     191       34    0    0 02:37:13        6
100.100.100.2   4          101     258     259       34    0    0 03:41:46       10
```

R15 

```
router bgp 1001
 bgp log-neighbor-changes
 network 111.111.111.0 mask 255.255.255.252
 neighbor 10.10.10.25 remote-as 1001
 neighbor 111.111.111.2 remote-as 301
```

```
BGP router identifier 111.111.111.1, local AS number 1001
BGP table version is 26, main routing table version 26
10 network entries using 1440 bytes of memory
17 path entries using 1360 bytes of memory
8/6 BGP path/bestpath attribute entries using 1216 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4160 total bytes of memory
BGP activity 12/2 prefixes, 26/9 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.25     4         1001     192     186       26    0    0 02:38:07        6
111.111.111.2   4          301     184     185       26    0    0 02:37:22       10
```

R21

```
router bgp 301
 bgp log-neighbor-changes
 network 110.110.110.0 mask 255.255.255.252
 network 111.111.111.0 mask 255.255.255.252
 network 111.111.111.4 mask 255.255.255.252
 neighbor 110.110.110.1 remote-as 101
 neighbor 111.111.111.1 remote-as 1001
 neighbor 111.111.111.6 remote-as 520
```

```
BGP router identifier 111.111.111.5, local AS number 301
BGP table version is 17, main routing table version 17
10 network entries using 1440 bytes of memory
20 path entries using 1600 bytes of memory
10/5 BGP path/bestpath attribute entries using 1520 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4728 total bytes of memory
BGP activity 11/1 prefixes, 26/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
110.110.110.1   4          101     266     267       17    0    0 03:52:06        6
111.111.111.1   4         1001     186     185       17    0    0 02:38:09        5
111.111.111.6   4          520     176     183       17    0    0 02:33:28        6
```

R22

```
router bgp 101
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 network 100.100.100.4 mask 255.255.255.252
 network 110.110.110.0 mask 255.255.255.252
 neighbor 100.100.100.1 remote-as 1001
 neighbor 100.100.100.6 remote-as 520
 neighbor 110.110.110.2 remote-as 301
```

```
BGP router identifier 110.110.110.1, local AS number 101
BGP table version is 22, main routing table version 22
10 network entries using 1440 bytes of memory
20 path entries using 1600 bytes of memory
10/4 BGP path/bestpath attribute entries using 1520 bytes of memory
7 BGP AS-PATH entries using 168 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4728 total bytes of memory
BGP activity 12/2 prefixes, 29/9 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.100.100.1   4         1001     262     261       22    0    0 03:44:28        5
100.100.100.6   4          520     157     162       22    0    0 02:16:57        6
110.110.110.2   4          301     268     267       22    0    0 03:53:07        6
```

### Настроите eBGP между Ламас (заодно и Киторн) и Триада

R23

```
router bgp 520
 bgp log-neighbor-changes
 network 100.100.100.4 mask 255.255.255.252
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 100.100.100.5 remote-as 101
```

```
BGP router identifier 100.100.100.6, local AS number 520
BGP table version is 35, main routing table version 35
10 network entries using 1440 bytes of memory
14 path entries using 1120 bytes of memory
5/4 BGP path/bestpath attribute entries using 760 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3392 total bytes of memory
BGP activity 15/5 prefixes, 27/13 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.24.1       4          520      12      12       35    0    0 00:01:49        4
50.0.25.1       4          520       8       9       35    0    0 00:01:43        2
50.0.26.1       4          520      15      17       35    0    0 00:08:07        2
100.100.100.5   4          101     223     220       35    0    0 03:08:37        5
```

R24

```
router bgp 520
 bgp log-neighbor-changes
 network 111.111.111.4 mask 255.255.255.252
 network 111.111.111.8 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 50.0.26.1 remote-as 520
 neighbor 50.0.26.1 update-source Loopback0
 neighbor 111.111.111.5 remote-as 301
 neighbor 111.111.111.10 remote-as 2042
 ----------------------------------------------
 BGP router identifier 111.111.111.9, local AS number 520
BGP table version is 38, main routing table version 38
10 network entries using 1440 bytes of memory
16 path entries using 1280 bytes of memory
6/4 BGP path/bestpath attribute entries using 912 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3728 total bytes of memory
BGP activity 12/2 prefixes, 29/13 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      15      15       38    0    0 00:04:22        3
50.0.25.1       4          520       7       8       38    0    0 00:00:23        2
50.0.26.1       4          520      20      21       38    0    0 00:10:24        2
111.111.111.5   4          301     248     242       36    0    0 03:28:42        5
111.111.111.10  4         2042     205     213       36    0    0 02:58:31        2
```

### eBGP между офисом С.-Петербург и провайдером Триада

R26

```
router bgp 520
 bgp log-neighbor-changes
 network 111.110.35.12 mask 255.255.255.252
 network 111.111.111.12 mask 255.255.255.252
 neighbor 50.0.23.1 remote-as 520
 neighbor 50.0.23.1 update-source Loopback0
 neighbor 50.0.24.1 remote-as 520
 neighbor 50.0.24.1 update-source Loopback0
 neighbor 50.0.25.1 remote-as 520
 neighbor 50.0.25.1 update-source Loopback0
 neighbor 111.111.111.14 remote-as 2042

 
 ------------------------------------------------------------
 
BGP router identifier 111.111.111.13, local AS number 520
BGP table version is 38, main routing table version 38
10 network entries using 1440 bytes of memory
17 path entries using 1360 bytes of memory
8/4 BGP path/bestpath attribute entries using 1216 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 4136 total bytes of memory
BGP activity 18/8 prefixes, 28/11 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520       7       7       38    0    0 00:00:33        5
50.0.24.1       4          520       9       9       38    0    0 00:00:17        6
50.0.25.1       4          520       5       7       38    0    0 00:00:08        2
111.111.111.14  4         2042     196     195       38    0    0 02:42:28        2
```

R18

```
router bgp 2042
 bgp log-neighbor-changes
 network 111.111.111.8 mask 255.255.255.252
 network 111.111.111.12 mask 255.255.255.252
 neighbor 111.111.111.9 remote-as 520
 neighbor 111.111.111.13 remote-as 520
 
----------------------------------------------------------------

BGP router identifier 111.111.111.14, local AS number 2042
BGP table version is 14, main routing table version 14
10 network entries using 1440 bytes of memory
18 path entries using 1440 bytes of memory
5/5 BGP path/bestpath attribute entries using 760 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3712 total bytes of memory
BGP activity 10/0 prefixes, 19/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
111.111.111.9   4          520     159     154       14    0    0 02:15:18        8
111.111.111.13  4          520     151     156       14    0    0 02:09:22        8

```

R24 смотри выше ,чтобы не повторяться,но  там картина аналогичная. Соседство присутствует с Ламас и с СПб.

### Организуете IP доступность между офисами Москва и С.-Петербург

R15 ping R18

```
R15(config-router)#do ping 111.111.111.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.111.111.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
------------------------------------------------------------------------------------
R15(config-router)#do ping 111.111.111.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.111.111.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

R14 ping R18

```
R14(config-router)#do ping 111.111.111.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.111.111.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
-----------------------------------------------------------------------------------
R14(config-router)#do ping 111.111.111.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.111.111.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

### Настроите отслеживание линка через технологию IP SLA

R18

```
ip sla 1
 icmp-echo 111.111.111.13 source-interface Ethernet0/3
 frequency 5
ip sla schedule 1 life forever start-time now

ip sla 2
 icmp-echo 111.111.111.9 source-interface Ethernet0/2
 frequency 5
ip sla schedule 2 life forever start-time now

track 1 ip sla 1 reachability
track 2 ip sla 2 reachability

ip route 0.0.0.0 0.0.0.0 111.111.111.13 track 1
ip route 0.0.0.0 0.0.0.0 111.111.111.9 track 2

----------------------------------------------------------------------------

R18(config)#do sh ip route static
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 111.111.111.13 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 111.111.111.13
                [1/0] via 111.111.111.9
```

Упадет 1 из линков , уберется из таблицы маршрутизации и маршрут по умолчанию.

R15

```
ip sla 1
 icmp-echo 111.111.111.2 source-interface Ethernet0/2
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

ip route 0.0.0.0 0.0.0.0 111.111.111.2 track 1
ip route 0.0.0.0 0.0.0.0 10.10.10.25 10

------------------------------------------------------------------------------
R15(config)#do sh ip route static
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 111.111.111.2 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 111.111.111.2

```

R14

```
ip sla 1
 icmp-echo 100.100.100.2 source-interface Ethernet0/2
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

ip route 0.0.0.0 0.0.0.0 100.100.100.2 track 1
ip route 0.0.0.0 0.0.0.0 10.10.10.26 10

-----------------------------------------------------------------------------

R14(config)#do sh ip route static
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 100.100.100.2 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 100.100.100.2
```

