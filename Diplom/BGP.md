# BGP. Основы

## Цель: Настроить BGP между автономными системами Организовать доступность между офисами Москва и С.-Петербург

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1.  eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
2. Настроите eBGP между провайдерами Киторн и Ламас
3. Настроите eBGP между Ламас и Триада
4. eBGP между офисом С.-Петербург и провайдером Триада
5. Организуете IP доступность между офисами Москва и С.-Петербург
6. План работы и изменения зафиксированы в документации

![EVE_Topology](img/EVE_Topology.png)

Решил добавить линк между  R14 и R15 ,чтобы  проще соединить их в area 0  ,во вторых ,чтобы обеспечить отказоустойчивость при прохождении пакета через интернет.

### eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас

R14 

```
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 neighbor 10.10.10.26 remote-as 1001
 neighbor 100.100.100.2 remote-as 101
-------------------------------------------------------------------------------------
BGP router identifier 14.14.14.14, local AS number 1001
BGP table version is 84, main routing table version 84
10 network entries using 1480 bytes of memory
19 path entries using 1216 bytes of memory
6/4 BGP path/bestpath attribute entries using 816 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3656 total bytes of memory
BGP activity 36/26 prefixes, 102/83 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.26     4         1001     379     383       84    0    0 05:34:13        9
100.100.100.2   4          101     381     378       84    0    0 05:33:33       10
```



R15 

```
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 neighbor 10.10.10.25 remote-as 1001
 neighbor 111.111.111.2 remote-as 301
 
 ---------------------------------------------------------------------------

BGP router identifier 15.15.15.15, local AS number 1001
BGP table version is 50, main routing table version 50
10 network entries using 1480 bytes of memory
19 path entries using 1216 bytes of memory
6/4 BGP path/bestpath attribute entries using 816 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3656 total bytes of memory
BGP activity 39/29 prefixes, 110/91 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.25     4         1001     385     380       50    0    0 05:35:41        9
111.111.111.2   4          301     378     373       50    0    0 05:34:10       10
```



R21

```
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 network 110.110.110.0 mask 255.255.255.252
 network 111.111.111.0 mask 255.255.255.252
 network 111.111.111.4 mask 255.255.255.252
 neighbor 110.110.110.1 remote-as 101
 neighbor 111.111.111.1 remote-as 1001
 neighbor 111.111.111.6 remote-as 520

-------------------------------------------------------------------------------

BGP router identifier 21.21.21.21, local AS number 301
BGP table version is 30, main routing table version 30
10 network entries using 1480 bytes of memory
17 path entries using 1088 bytes of memory
7/4 BGP path/bestpath attribute entries using 952 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3640 total bytes of memory
BGP activity 31/21 prefixes, 82/65 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
110.110.110.1   4          101     385     380       30    0    0 05:35:38        7
111.111.111.1   4         1001     375     379       30    0    0 05:35:38        1
111.111.111.6   4          520     361     364       30    0    0 05:20:03        6
```



R22

```
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 network 100.100.100.4 mask 255.255.255.252
 neighbor 100.100.100.1 remote-as 1001
 neighbor 100.100.100.6 remote-as 520
 neighbor 110.110.110.2 remote-as 301
 
 ------------------------------------------------------------------------------------
 
BGP router identifier 22.22.22.22, local AS number 101
BGP table version is 41, main routing table version 41
10 network entries using 1480 bytes of memory
17 path entries using 1088 bytes of memory
6/3 BGP path/bestpath attribute entries using 816 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3504 total bytes of memory
BGP activity 32/22 prefixes, 97/80 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.100.100.1   4         1001     383     386       41    0    0 05:37:31        1
100.100.100.6   4          520     363     369       41    0    0 05:23:07        6
110.110.110.2   4          301     382     386       41    0    0 05:36:40        8
```



### Настроите eBGP между Ламас (заодно и Киторн) и Триада

R23

```
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

----------------------------------------------------------------------------------

BGP router identifier 23.23.23.23, local AS number 520
BGP table version is 32, main routing table version 32
10 network entries using 1480 bytes of memory
14 path entries using 896 bytes of memory
5/4 BGP path/bestpath attribute entries using 680 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3152 total bytes of memory
BGP activity 39/29 prefixes, 72/58 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.24.1       4          520     364     364       32    0    0 05:23:00        5
50.0.25.1       4          520     358     363       32    0    0 05:22:01        2
50.0.26.1       4          520     357     362       32    0    0 05:20:51        2
100.100.100.5   4          101     371     366       32    0    0 05:25:02        5

```



R24

```
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

 --------------------------------------------------------------------------------
 
BGP router identifier 24.24.24.24, local AS number 520
BGP table version is 22, main routing table version 22
10 network entries using 1480 bytes of memory
14 path entries using 896 bytes of memory
6/5 BGP path/bestpath attribute entries using 816 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3288 total bytes of memory
BGP activity 31/21 prefixes, 74/60 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520     367     367       22    0    0 05:25:27        4
50.0.25.1       4          520     359     365       22    0    0 05:24:28        2
50.0.26.1       4          520     361     362       22    0    0 05:23:18        2
77.77.77.10     4         2042     370     368       22    0    0 05:25:27        0
111.111.111.5   4          301     371     367       22    0    0 05:25:27        5

```

### eBGP между офисом С.-Петербург и провайдером Триада

R26

```
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


 
 ------------------------------------------------------------
 
BGP router identifier 26.26.26.26, local AS number 520
BGP table version is 16, main routing table version 16
10 network entries using 1480 bytes of memory
13 path entries using 832 bytes of memory
6/2 BGP path/bestpath attribute entries using 816 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3224 total bytes of memory
BGP activity 35/25 prefixes, 74/61 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
50.0.23.1       4          520     366     361       16    0    0 05:24:08        4
50.0.24.1       4          520     363     362       16    0    0 05:24:08        5
50.0.25.1       4          520     363     359       16    0    0 05:24:08        2
77.77.77.14     4         2042     370     363       16    0    0 05:24:08        0

```

R18

```
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 neighbor 77.77.77.9 remote-as 520
 neighbor 77.77.77.13 remote-as 520

 
----------------------------------------------------------------

BGP router identifier 18.18.18.18, local AS number 2042
BGP table version is 1, main routing table version 1
10 network entries using 1480 bytes of memory
15 path entries using 960 bytes of memory
5/0 BGP path/bestpath attribute entries using 680 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3216 total bytes of memory
BGP activity 45/35 prefixes, 70/55 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
77.77.77.9      4          520      10       2        1    0    0 00:00:24       10
77.77.77.13     4          520       6       2        1    0    0 00:00:24        5

```



### Организуете IP доступность между офисами Москва и С.-Петербург

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



### Сети провайдера 



| Triada           | LAMAS            | Kitorn           |
| ---------------- | ---------------- | ---------------- |
| 77.77.77.8/30    | 111.111.111.0/30 | 100.100.100.0/30 |
| 77.77.77.12/30   | 110.110.110.0/30 | 100.100.100.4/30 |
| 111.110.35.8/30  | 111.111.111.4/30 |                  |
| 111.110.35.12/30 |                  |                  |
| 210.110.35.0/30  |                  |                  |

**Для доступа к прописанным конфигурациям на роутерах ,то жмём сюда :**

https://e9exu-my.sharepoint.com/:f:/g/personal/nickelface_ermaon_com/Euh_hOXWUWRAr0awcVlpJVYBzobOZWNdcKt4VLkLif40EA?e=GGNmyl

1111111111111111111111111111