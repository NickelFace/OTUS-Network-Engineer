# BGP. Выбор пути 

![](img/EVE_Topology.png)

## Домашнее задание

Цель: Настроить фильтрацию для офисе Москва Настроить фильтрацию для офисе С.-Петербург

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург
5. Все сети в лабораторной работе должны иметь IP связность
6. План работы и изменения зафиксированы в документации

### Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path)

R14

```
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 200.20.20.0 mask 255.255.252.0
 neighbor MSK peer-group
 neighbor MSK remote-as 1001
 neighbor MSK update-source Loopback0
 neighbor MSK next-hop-self
 neighbor 1.1.1.15 peer-group MSK
 neighbor 100.100.100.2 remote-as 101
 neighbor 100.100.100.2 filter-list 1 out


ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*

------------------------------------------------

interface Loopback0
 ip address 1.1.1.14 255.255.255.255
//Для соседа iBGP

interface Loopback14
 ip address 200.20.20.14 255.255.252.0
//Сеть которую мы анонсируем в интернет
```

R15

```
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 200.20.20.0 mask 255.255.252.0
 neighbor 1.1.1.14 remote-as 1001
 neighbor 1.1.1.14 next-hop-self
 neighbor 111.111.111.2 remote-as 301
 neighbor 111.111.111.2 route-map LP in
 neighbor 111.111.111.2 filter-list 1 out

ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*

------------------------------------------------

interface Loopback0
 ip address 1.1.1.15 255.255.255.255
 //Для соседа iBGP
 
interface Loopback15
 ip address 200.20.20.15 255.255.252.0
//Сеть которую мы анонсируем в интернет
```

**Теперь проверим провайдерские роутеры на наличие маршрутов**

R22

```
R22#show ip bgp
BGP table version is 47, local router ID is 110.110.110.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   77.77.77.8/30    110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i
 *   77.77.77.12/30   110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i
 *>  100.100.100.0/30 0.0.0.0                  0         32768 i
 *>  100.100.100.4/30 0.0.0.0                  0         32768 i
 r   110.110.110.0/30 100.100.100.6                          0 520 301 i
 r>                   110.110.110.2            0             0 301 i
 *   111.110.35.8/30  110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i
 *   111.110.35.12/30 110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i
 *   111.111.111.0/30 100.100.100.6                          0 520 301 i
 *>                   110.110.110.2            0             0 301 i
 *   111.111.111.4/30 100.100.100.6                          0 520 301 i
 *>                   110.110.110.2            0             0 301 i
 *   200.20.20.0/22   110.110.110.2                          0 301 1001 i 
 *>                   100.100.100.1            0             0 1001 i
 *   210.110.35.0/30  110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i

------------------------------------------------------------------------------
Как видно из вывода АС 1001 анонсирует только свои сети и не является транзитом для других
```

R21

```
R21#show ip bgp
BGP table version is 13, local router ID is 111.111.111.5
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   77.77.77.8/30    110.110.110.1                          0 101 520 i
 *>                   111.111.111.6            0             0 520 i
 *   77.77.77.12/30   110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i
 *   100.10.8.0/22    110.110.110.1                          0 101 520 2042 i
 *>                   111.111.111.6                          0 520 2042 i
 *   100.100.100.0/30 111.111.111.6                          0 520 101 i
 *>                   110.110.110.1            0             0 101 i
 *   100.100.100.4/30 111.111.111.6                          0 520 101 i
 *>                   110.110.110.1            0             0 101 i
 *>  110.110.110.0/30 0.0.0.0                  0         32768 i
 *   111.110.35.8/30  110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i
 *   111.110.35.12/30 110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i
 *>  111.111.111.0/30 0.0.0.0                  0         32768 i
 *>  111.111.111.4/30 0.0.0.0                  0         32768 i
 *   200.20.20.0/22   110.110.110.1                          0 101 1001 i
 *>                   111.111.111.1            0             0 1001 i
 *   210.110.35.0/30  110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i

------------------------------------------------------------------------------
Картина аналогичная предыдущему выводу
```

### Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)

R18

```
interface Loopback18
 ip address 100.10.8.18 255.255.252.0
 //Сеть которую мы анонсируем в интернет
-------------------------------------------------

ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny .*

Создаем prefix-list :
ip prefix-list DEFAULT seq 15 permit 100.10.8.0/22 le 32 
ip prefix-list DEFAULT seq 20 deny 0.0.0.0/0 le 32

Прикрепляем его к route-map:
route-map FILTER permit 10
 match ip address prefix-list DEFAULT
 
------------------------------------------------

router bgp 2042
 template peer-policy TRIADA_POLICY
  route-map FILTER out
  filter-list 1 out
 exit-peer-policy
 !
 template peer-session TRIADA
  remote-as 520
 exit-peer-session
 !
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 network 100.10.8.0 mask 255.255.252.0
 neighbor SPB peer-group
 neighbor SPB remote-as 2042
 neighbor SPB update-source Loopback0
 neighbor SPB next-hop-self
 neighbor 1.1.2.16 peer-group SPB
 neighbor 1.1.2.17 peer-group SPB
 neighbor 1.1.2.32 peer-group SPB
 neighbor 77.77.77.9 inherit peer-session TRIADA
 neighbor 77.77.77.9 inherit peer-policy TRIADA_POLICY
 neighbor 77.77.77.13 inherit peer-session TRIADA
 neighbor 77.77.77.13 inherit peer-policy TRIADA_POLICY

------------------------------------------------
```

Теперь посмотрим,что приходит у провайдера Триада (R24/R26)

```
R24#show ip bgp
BGP table version is 19, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  77.77.77.8/30    0.0.0.0                  0         32768 i
 *>i 77.77.77.12/30   50.0.26.1                0    100      0 i
 * i 100.10.8.0/22    50.0.26.1                0    100      0 2042 i
 *>                   77.77.77.10              0             0 2042 i
 *>i 100.100.100.0/30 50.0.23.1                0    100      0 101 i
 *                    111.111.111.5                          0 301 101 i
 *>i 100.100.100.4/30 50.0.23.1                0    100      0 101 i
 *                    111.111.111.5                          0 301 101 i
 *>  110.110.110.0/30 111.111.111.5            0             0 301 i
 *>i 111.110.35.8/30  50.0.25.1                0    100      0 i
 *>i 111.110.35.12/30 50.0.26.1                0    100      0 i
 *>  111.111.111.0/30 111.111.111.5            0             0 301 i
 r>  111.111.111.4/30 111.111.111.5            0             0 301 i
 * i 200.20.20.0/22   50.0.23.1                0    100      0 101 1001 i
 *>                   111.111.111.5                          0 301 1001 i
 *>i 210.110.35.0/30  50.0.25.1                0    100      0 i

Нас лишь интересует маршрут 100.10.8.0/22 за АС 2042  
---------------------------------------------------------------------------
R26#show ip bgp
BGP table version is 26, local router ID is 26.26.26.26
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 77.77.77.8/30    50.0.24.1                0    100      0 i
 *>  77.77.77.12/30   0.0.0.0                  0         32768 i
 * i 100.10.8.0/22    50.0.24.1                0    100      0 2042 i
 *>                   77.77.77.14              0             0 2042 i
 *>i 100.100.100.0/30 50.0.23.1                0    100      0 101 i
 *>i 100.100.100.4/30 50.0.23.1                0    100      0 101 i
 *>i 110.110.110.0/30 50.0.24.1                0    100      0 301 i
 *>i 111.110.35.8/30  50.0.25.1                0    100      0 i
 *>  111.110.35.12/30 0.0.0.0                  0         32768 i
 *>i 111.111.111.0/30 50.0.24.1                0    100      0 301 i
 *>i 111.111.111.4/30 50.0.24.1                0    100      0 301 i
 * i 200.20.20.0/22   50.0.23.1                0    100      0 101 1001 i
 *>i                  50.0.24.1                0    100      0 301 1001 i
 *>i 210.110.35.0/30  50.0.25.1                0    100      0 i

-----------------------------------------------------------------------
Нас лишь интересует маршрут 100.10.8.0/22 за АС 2042

Как видно из вывода мы смогли выполнить фильтрацию с помощью Prefix-list
Воспользоваться as-path access-list задании не запрещалось.
```

### Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию

R22

```
Создаем prefix-list :
ip prefix-list ISP seq 5 permit 0.0.0.0/0
ip prefix-list ISP seq 10 permit 100.10.8.0/22
ip prefix-list ISP seq 20 deny 0.0.0.0/0 le 32

Прикрепляем его к route-map :
route-map DEFAULT permit 10
 match ip address prefix-list ISP
------------------------------------------------
router bgp 101
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.252
 network 100.100.100.4 mask 255.255.255.252
 neighbor 100.100.100.1 remote-as 1001
 neighbor 100.100.100.1 default-originate
 neighbor 100.100.100.1 route-map DEFAULT out //применяем rout-map для нашего "соседа"
 neighbor 100.100.100.6 remote-as 520
 neighbor 110.110.110.2 remote-as 301

------------------------------------------------
```

R14

```
R14(config-if)#do sh ip bgp
BGP table version is 25, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          100.100.100.2                          0 101 i
 *>i 77.77.77.8/30    1.1.1.15                 0    150      0 301 520 i
 *>i 77.77.77.12/30   1.1.1.15                 0    150      0 301 520 i
 *>i 100.10.8.0/22    1.1.1.15                 0    150      0 301 520 2042 i
 *>i 110.110.110.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.110.35.8/30  1.1.1.15                 0    150      0 301 520 i
 *>i 111.110.35.12/30 1.1.1.15                 0    150      0 301 520 i
 *>i 111.111.111.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.111.111.4/30 1.1.1.15                 0    150      0 301 i
 * i 200.20.20.0/22   1.1.1.15                 0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 210.110.35.0/30  1.1.1.15                 0    150      0 301 520 i

---------------------------------------------------------------------------
Маршрут по умолчанию мы смогли передать,значит задача выполнена
```



### Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург



R21

```
Создаем prefix-list :
ip prefix-list ISP seq 5 permit 0.0.0.0/0      // маршрут по-умолчанию
ip prefix-list ISP seq 10 permit 100.10.8.0/22 //Сеть Спб
ip prefix-list ISP seq 15 deny 0.0.0.0/0 le 32  

route-map DEFAULT permit 10
 match ip address prefix-list ISP
------------------------------------------------

router bgp 301
 bgp log-neighbor-changes
 network 110.110.110.0 mask 255.255.255.252
 network 111.111.111.0 mask 255.255.255.252
 network 111.111.111.4 mask 255.255.255.252
 neighbor 110.110.110.1 remote-as 101
 neighbor 111.111.111.1 remote-as 1001
 neighbor 111.111.111.1 route-map DEFAULT out
 neighbor 111.111.111.6 remote-as 520

------------------------------------------------
```

R15 / R14

```
R15(config-if)#do show ip bgp
BGP table version is 48, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          1.1.1.14                 0    100      0 101 i
 *>  100.10.8.0/22    111.111.111.2                 150      0 301 520 2042 i
 * i 200.20.20.0/22   1.1.1.14                 0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 
-------------------------------------------------------------------------------
R14#show ip bgp
BGP table version is 53, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          100.100.100.2                          0 101 i
 *   100.10.8.0/22    100.100.100.2                          0 101 520 2042 i
 *>i                  1.1.1.15                 0    150      0 301 520 2042 i
 * i 200.20.20.0/22   1.1.1.15                 0    100      0 i
 *>                   0.0.0.0                  0         32768 i

```

### Все сети в лабораторной работе должны иметь IP связность

R14/R15

```
R14#ping 77.77.77.10 source e0/2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 77.77.77.14 source e0/2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 210.110.35.2 source e0/2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 111.110.35.10 source e0/2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 111.110.35.14 source e0/2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### План работы и изменения зафиксированы в документации

**Для доступа к прописанным конфигурациям на роутерах ,то жмём сюда :**

https://e9exu-my.sharepoint.com/:f:/g/personal/nickelface_ermaon_com/Euh_hOXWUWRAr0awcVlpJVYBzobOZWNdcKt4VLkLif40EA?e=GGNmylЦель: 

