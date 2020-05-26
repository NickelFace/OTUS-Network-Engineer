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