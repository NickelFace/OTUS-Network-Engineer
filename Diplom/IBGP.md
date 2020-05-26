# IBGP

## Домашнее задание

Цель: Настроить iBGP в офисе Москва Настроить iBGP в сети провайдера Триада Организовать полную IP связанность всех сетей

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. iBGP в офисом Москва между маршрутизаторами R14 и R15
2. Настроите iBGP в провайдере Триада
3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.
4. В офисе С.-Петербург работает протокол iBGP. (Не использовать протокол OSPF)
5. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
6. Все сети в лабораторной работе должны иметь IP связность
7. План работы и изменения зафиксированы в документации



![EVE_Topology](img/EVE_Topology.png)

### iBGP в офисом Москва между маршрутизаторами R14 и R15

R14

```
R14#show ip bgp summary
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 14, main routing table version 14
10 network entries using 1480 bytes of memory
19 path entries using 1216 bytes of memory
6/4 BGP path/bestpath attribute entries using 816 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3656 total bytes of memory
BGP activity 10/0 prefixes, 19/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.26     4         1001      77      78       14    0    0 01:04:49        9
100.100.100.2   4          101      77      76       14    0    0 01:04:45       10
```

R15

```
R15#show ip bgp summary
BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 13, main routing table version 13
10 network entries using 1480 bytes of memory
19 path entries using 1216 bytes of memory
6/4 BGP path/bestpath attribute entries using 816 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3656 total bytes of memory
BGP activity 10/0 prefixes, 19/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.25     4         1001      81      80       13    0    0 01:07:15        9
111.111.111.2   4          301      80      76       13    0    0 01:07:15       10
```

### Настроите iBGP в провайдере Триада

R23

```
R23>show ip bgp summary
BGP router identifier 23.23.23.23, local AS number 520
BGP table version is 16, main routing table version 16
10 network entries using 1480 bytes of memory
14 path entries using 896 bytes of memory
5/4 BGP path/bestpath attribute entries using 680 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3152 total bytes of memory
BGP activity 10/0 prefixes, 16/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.24.1       4          520      87      88       16    0    0 01:13:00        5
50.0.25.1       4          520      84      88       16    0    0 01:13:01        2
50.0.26.1       4          520      85      88       16    0    0 01:13:02        2
100.100.100.5   4          101      87      88       16    0    0 01:13:48        5
```

R24

```
R24#show ip bgp summary
BGP router identifier 24.24.24.24, local AS number 520
BGP table version is 18, main routing table version 18
10 network entries using 1480 bytes of memory
14 path entries using 896 bytes of memory
6/5 BGP path/bestpath attribute entries using 816 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3288 total bytes of memory
BGP activity 10/0 prefixes, 17/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      89      88       18    0    0 01:13:45        4
50.0.25.1       4          520      84      89       18    0    0 01:13:57        2
50.0.26.1       4          520      85      88       18    0    0 01:13:54        2
77.77.77.10     4         2042      89      90       18    0    0 01:14:30        0
111.111.111.5   4          301      89      89       18    0    0 01:14:33        5
```

R25

```
R25#show ip bgp summary
BGP router identifier 25.25.25.25, local AS number 520
BGP table version is 16, main routing table version 16
10 network entries using 1480 bytes of memory
13 path entries using 832 bytes of memory
6/2 BGP path/bestpath attribute entries using 816 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3224 total bytes of memory
BGP activity 10/0 prefixes, 18/5 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      89      86       16    0    0 01:14:28        4
50.0.24.1       4          520      90      85       16    0    0 01:14:40        5
50.0.26.1       4          520      85      87       16    0    0 01:14:36        2
```

R26

```
R26#show ip bgp summary
BGP router identifier 26.26.26.26, local AS number 520
BGP table version is 6, main routing table version 6
10 network entries using 1480 bytes of memory
13 path entries using 832 bytes of memory
6/2 BGP path/bestpath attribute entries using 816 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3224 total bytes of memory
BGP activity 10/0 prefixes, 18/5 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520      91      88        6    0    0 01:15:35        4
50.0.24.1       4          520      90      87        6    0    0 01:15:43        5
50.0.25.1       4          520      88      86        6    0    0 01:15:41        2
77.77.77.14     4         2042      91      89        6    0    0 01:16:24        0
```

### Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.

В данном случае я предпринял несколько действий 

1. для ухудшения пути для R15 я поставил route-map в сторону провайдера увеличивающий путь
2. Установил R14 роутером по умолчанию для BGP 
3. R14 раздает маршруты от себя 

R14

```
R14(config)#do sh run | s bgp
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 10.10.10.26 remote-as 1001
 neighbor 10.10.10.26 next-hop-self
 neighbor 100.100.100.2 remote-as 101
 default-information originate

```



R15

```
R15(config-router)#do sh run | s bgp
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 10.10.10.25 remote-as 1001
 neighbor 111.111.111.2 remote-as 301
 neighbor 111.111.111.2 route-map SET-ASPATH in

 ---------------------------------------------------------------------------

route-map SET-ASPATH permit 10
 set as-path prepend 1001 1001 1001

```

### В офисе С.-Петербург работает протокол iBGP. (Не использовать протокол OSPF)

Раз OSPF нельзя, значит EIGRP можно,логика проста)

R16

```
int lo0 
ip addr 1.1.2.16 255.255.255.255
------------------------------------------------
router bgp 2042
neighbor 1.1.2.18 remote-as 2042
neighbor 1.1.2.18 up lo0
neighbor 1.1.2.17 remote-as 2042
neighbor 1.1.2.17 up lo0
neighbor 1.1.2.32 remote-as 2042
neighbor 1.1.2.32 up lo0
------------------------------------------------
router eigrp 1
 network 1.1.2.16 0.0.0.0
 network 10.10.20.4 0.0.0.3
 network 10.10.20.8 0.0.0.3
 network 172.16.11.0 0.0.0.255
 passive-interface Ethernet0/0
 ------------------------------------------------
 R16(config-router)#do sh ip bgp summ
BGP router identifier 1.1.2.16, local AS number 2042
BGP table version is 1, main routing table version 1
10 network entries using 1480 bytes of memory
10 path entries using 640 bytes of memory
4/0 BGP path/bestpath attribute entries using 544 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2760 total bytes of memory
BGP activity 10/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.17        4         2042      13      13        1    0    0 00:10:01        0
1.1.2.18        4         2042      21      15        1    0    0 00:10:26       10
1.1.2.32        4         2042      12      12        1    0    0 00:09:27        0
```

R17

```
int lo0 
ip addr 1.1.2.17 255.255.255.255
------------------------------------------------
router bgp 2042
 bgp log-neighbor-changes
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 update-source Loopback0
 neighbor 1.1.2.18 remote-as 2042
 neighbor 1.1.2.18 update-source Loopback0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 update-source Loopback0
------------------------------------------------
router eigrp 1
 network 1.1.2.17 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 172.16.10.0 0.0.0.255
 passive-interface Ethernet0/0
------------------------------------------------
R17(config-router)#do sh ip bgp summ
BGP router identifier 1.1.2.17, local AS number 2042
BGP table version is 1, main routing table version 1
10 network entries using 1480 bytes of memory
10 path entries using 640 bytes of memory
4/0 BGP path/bestpath attribute entries using 544 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2760 total bytes of memory
BGP activity 10/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      15      15        1    0    0 00:12:08        0
1.1.2.18        4         2042      24      17        1    0    0 00:12:11       10
1.1.2.32        4         2042      14      14        1    0    0 00:11:34        0

```

R18

```
nt lo0
 ip addr 1.1.2.18 255.255.255.255
------------------------------------------------
router bgp 2042
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520
 neighbor 1.1.2.16 remote-as 2042
 neighbor 1.1.2.16 up lo0
 neighbor 1.1.2.17 remote-as 2042
 neighbor 1.1.2.17 up lo0
 neighbor 1.1.2.32 remote-as 2042
 neighbor 1.1.2.32 up lo0
------------------------------------------------
router eigrp 1
 network 1.1.2.18 0.0.0.0
 network 10.10.20.0 0.0.0.3
 network 10.10.20.4 0.0.0.3
 ------------------------------------------------
 R18(config-router)#do sh ip bgp summ
BGP router identifier 18.18.18.18, local AS number 2042
BGP table version is 12, main routing table version 12
10 network entries using 1480 bytes of memory
15 path entries using 960 bytes of memory
5/5 BGP path/bestpath attribute entries using 680 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3216 total bytes of memory
BGP activity 10/0 prefixes, 15/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      19      25       12    0    0 00:14:16        0
1.1.2.17        4         2042      19      26       12    0    0 00:13:55        0
1.1.2.32        4         2042      18      25       12    0    0 00:13:19        0
77.77.77.9      4          520     121     122       12    0    0 01:45:22       10
77.77.77.13     4          520     119     122       12    0    0 01:45:24        5

```

R32

```
int lo0 
ip addr 1.1.2.32 255.255.255.255
------------------------------------------------
router bgp 2042
neighbor 1.1.2.18 remote-as 2042
neighbor 1.1.2.18 up lo0
neighbor 1.1.2.17 remote-as 2042
neighbor 1.1.2.17 up lo0
neighbor 1.1.2.16 remote-as 2042
neighbor 1.1.2.16 up lo0
------------------------------------------------
router eigrp 1
 network 1.1.2.32 0.0.0.0
 network 10.10.20.8 0.0.0.3
------------------------------------------------
 R32(config-router)#do sh ip bgp summ
BGP router identifier 1.1.2.32, local AS number 2042
BGP table version is 1, main routing table version 1
10 network entries using 1480 bytes of memory
10 path entries using 640 bytes of memory
4/0 BGP path/bestpath attribute entries using 544 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2760 total bytes of memory
BGP activity 10/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.2.16        4         2042      10      10        1    0    0 00:07:34        0
1.1.2.17        4         2042      10      10        1    0    0 00:07:34        0
1.1.2.18        4         2042      19      12        1    0    0 00:07:36       10
```

### Все сети в лабораторной работе должны иметь IP связность

R15 ping R18

```
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
```

R14 ping R18

```
R14# ping 77.77.77.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms


R14# ping 77.77.77.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

R18 -> R28 / R27

```
До Лабытнанги
R18(config-if-range)#do ping 210.110.35.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

До Чукордах
R18(config-if-range)#do ping 111.110.35.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R18(config-if-range)#do ping 111.110.35.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

```

### План работы и изменения зафиксированы в документации

**Для доступа к прописанным конфигурациям на роутерах ,то жмём сюда :**

https://e9exu-my.sharepoint.com/:f:/g/personal/nickelface_ermaon_com/Euh_hOXWUWRAr0awcVlpJVYBzobOZWNdcKt4VLkLif40EA?e=GGNmyl