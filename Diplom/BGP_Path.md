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
Создаем As-path list:
ip as-path access-list 1 deny ^$
------------------------------------------------
Применим As-path list как это указанно в задании:
router bgp 1001
 neighbor 100.100.100.2 filter-list 1 out
------------------------------------------------
R14(config-router)#do clear ip bgp * out
------------------------------------------------
R14(config)#do sh ip bgp
BGP table version is 15, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          1.1.1.15                 0    150      0 301 i
 r                    100.100.100.2                          0 101 i
 *>i 77.77.77.8/30    1.1.1.15                 0    150      0 301 520 i
 *>i 77.77.77.12/30   1.1.1.15                 0    150      0 301 520 i
 r>i 100.100.100.0/30 1.1.1.15                 0    150      0 301 101 i
 *>i 100.100.100.4/30 1.1.1.15                 0    150      0 301 101 i
 *>i 110.110.110.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.110.35.8/30  1.1.1.15                 0    150      0 301 520 i
 *>i 111.110.35.12/30 1.1.1.15                 0    150      0 301 520 i
 *>i 111.111.111.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.111.111.4/30 1.1.1.15                 0    150      0 301 i
 *>i 210.110.35.0/30  1.1.1.15                 0    150      0 301 520 i
 *>  210.210.210.210/32
                       0.0.0.0                  0         32768 i
 *>i 215.215.215.215/32
                       1.1.1.15                 0    150      0 i
 --------------------------------------------------------------------------
 
 210.210.210.210/32 и 215.215.215.215/32 это поднятые Loopback interface на R14/R15 для проверки фильтрации маршрутов                  
```

R15

```
Создаем As-path list:
ip as-path access-list 1 deny ^$
------------------------------------------------
Применим As-path list как это указанно в задании:
router bgp 1001
 neighbor 111.111.111.2 filter-list 1 out
------------------------------------------------
clear ip bgp * out
------------------------------------------------
R15(config-router)#do sh ip bgp
BGP table version is 11, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          111.111.111.2                          0 301 i
 *>  77.77.77.8/30    111.111.111.2                          0 301 520 i
 *>  77.77.77.12/30   111.111.111.2                          0 301 520 i
 *>  100.100.100.0/30 111.111.111.2                          0 301 101 i
 *>  100.100.100.4/30 111.111.111.2                          0 301 101 i
 *>  110.110.110.0/30 111.111.111.2            0             0 301 i
 *>  111.110.35.8/30  111.111.111.2                          0 301 520 i
 *>  111.110.35.12/30 111.111.111.2                          0 301 520 i
 r>  111.111.111.0/30 111.111.111.2            0             0 301 i
 *>  111.111.111.4/30 111.111.111.2            0             0 301 i
 *>  210.110.35.0/30  111.111.111.2                          0 301 520 i
 *>i 210.210.210.210/32
                       1.1.1.14                 0    100      0 i
 *>  215.215.215.215/32
                       0.0.0.0                  0         32768 i


```

**Теперь проверим провайдерские роутеры на наличие маршрутов**

R22

```
R22#sh ip bgp
BGP table version is 40, local router ID is 22.22.22.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i
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
 *   210.110.35.0/30  110.110.110.2                          0 301 520 i
 *>                   100.100.100.6                          0 520 i
------------------------------------------------------------------------------
Как видим у нас отсутствуют маршруты 210.210.210.210/32 и 215.215.215.215/32 
```

R21

```
R21(config-router)#do sh ip bgp
BGP table version is 43, local router ID is 21.21.21.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i
 *   77.77.77.8/30    110.110.110.1                          0 101 520 i
 *>                   111.111.111.6            0             0 520 i
 *   77.77.77.12/30   110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i
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
 *   210.110.35.0/30  110.110.110.1                          0 101 520 i
 *>                   111.111.111.6                          0 520 i
------------------------------------------------------------------------------
Как видим у нас отсутствуют маршруты 210.210.210.210/32 и 215.215.215.215/32 
```

### Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list)

R18

```
Создаем prefix-list :
ip prefix-list DEFAULT seq 5 permit 77.77.77.8/30 le 32
ip prefix-list DEFAULT seq 10 permit 77.77.77.12/30 le 32
ip prefix-list DEFAULT seq 15 deny 0.0.0.0/0 le 32

Прикрепляем его к route-map:
route-map FILTER permit 10
 match ip address prefix-list DEFAULT

------------------------------------------------
router bgp 2042
 neighbor 77.77.77.9 route-map FILTER out
 neighbor 77.77.77.13 route-map FILTER out
------------------------------------------------
```

А теперь проверка:

```
Работу правила я проверил  поднятием Loopback interface 1 на R18
interface Loopback1
 ip address 215.215.215.215 255.255.255.255
 ------------------------------------------------
 R18(config-router)#do sh ip bgp
BGP table version is 12, local router ID is 18.18.18.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r   77.77.77.8/30    77.77.77.13                            0 520 i
 r>                   77.77.77.9               0             0 520 i
 r   77.77.77.12/30   77.77.77.9                             0 520 i
 r>                   77.77.77.13              0             0 520 i
 *   100.100.100.0/30 77.77.77.9                             0 520 101 i
 *>                   77.77.77.13                            0 520 101 i
 *   100.100.100.4/30 77.77.77.9                             0 520 101 i
 *>                   77.77.77.13                            0 520 101 i
 *   110.110.110.0/30 77.77.77.13                            0 520 301 i
 *>                   77.77.77.9                             0 520 301 i
 *   111.110.35.8/30  77.77.77.9                             0 520 i
 *>                   77.77.77.13                            0 520 i
 *   111.110.35.12/30 77.77.77.9                             0 520 i
 *>                   77.77.77.13              0             0 520 i
 *   111.111.111.0/30 77.77.77.13                            0 520 301 i
 *>                   77.77.77.9                             0 520 301 i
 *   111.111.111.4/30 77.77.77.13                            0 520 301 i
 *>                   77.77.77.9                             0 520 301 i
 *   210.110.35.0/30  77.77.77.9                             0 520 i
 *>                   77.77.77.13                            0 520 i
 *>  215.215.215.215/32
                       0.0.0.0                  0         32768 i


 ----------------------------------------------------------------------
Теперь посмотрим,что приходит к провайдеру Триада (R24)
R24#sh ip bgp
BGP table version is 42, local router ID is 24.24.24.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  77.77.77.8/30    0.0.0.0                  0         32768 i
 *>i 77.77.77.12/30   50.0.26.1                0    100      0 i
 *   100.100.100.0/30 111.111.111.5                          0 301 101 i
 *>i                  50.0.23.1                0    100      0 101 i
 *   100.100.100.4/30 111.111.111.5                          0 301 101 i
 *>i                  50.0.23.1                0    100      0 101 i
 *>  110.110.110.0/30 111.111.111.5            0             0 301 i
 *>i 111.110.35.8/30  50.0.25.1                0    100      0 i
 *>i 111.110.35.12/30 50.0.26.1                0    100      0 i
 *>  111.111.111.0/30 111.111.111.5            0             0 301 i
 r>  111.111.111.4/30 111.111.111.5            0             0 301 i
 *>i 210.110.35.0/30  50.0.25.1                0    100      0 i
---------------------------------------------------------------------------
С R18 сеть 215.215.215.215/32 не попадает к провайдеру Триада(R26 тот же результат)
```

### Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по-умолчанию

R22

```
Создаем prefix-list :
ip prefix-list ISP seq 5 permit 0.0.0.0/0
------------------------------------------------
router bgp 101
 neighbor 100.100.100.1 prefix-list ISP out
 neighbor 100.100.100.1 default-originate
------------------------------------------------
```

R14

```
R14(config)#do sh ip bgp
BGP table version is 15, local router ID is 14.14.14.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          1.1.1.15                 0    150      0 301 i
 r                    100.100.100.2                          0 101 i
 *>i 77.77.77.8/30    1.1.1.15                 0    150      0 301 520 i
 *>i 77.77.77.12/30   1.1.1.15                 0    150      0 301 520 i
 r>i 100.100.100.0/30 1.1.1.15                 0    150      0 301 101 i
 *>i 100.100.100.4/30 1.1.1.15                 0    150      0 301 101 i
 *>i 110.110.110.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.110.35.8/30  1.1.1.15                 0    150      0 301 520 i
 *>i 111.110.35.12/30 1.1.1.15                 0    150      0 301 520 i
 *>i 111.111.111.0/30 1.1.1.15                 0    150      0 301 i
 *>i 111.111.111.4/30 1.1.1.15                 0    150      0 301 i
 *>i 210.110.35.0/30  1.1.1.15                 0    150      0 301 520 i
 *>  210.210.210.210/32
                       0.0.0.0                  0         32768 i
 *>i 215.215.215.215/32
                       1.1.1.15                 0    150      0 i

---------------------------------------------------------------------------
Как видно из вывода мы от соседа 100.100.100.2 получаем 0.0.0.0 ,но он не попадает таблицу маршрутизации
```



### Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по-умолчанию и префикс офиса С.-Петербург



R21

```
ip prefix-list ISP seq 5 permit 0.0.0.0/0
------------------------------------------------
router bgp 301
 neighbor 111.111.111.1 prefix-list ISP out
 neighbor 111.111.111.1 default-originate
------------------------------------------------
```

В конце prefix-list есть неявное dany any , поэтому все остальные маршруты до Москвы будут резаться.

R15

```
R15(config-if-range)#do sh ip bgp
BGP table version is 46, local router ID is 15.15.15.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r   0.0.0.0          111.111.111.2                          0 1001 1001 1001 301 i
 r>i                  1.1.1.14                 0    150      0 101 i
```

### Все сети в лабораторной работе должны иметь IP связность

R14/R15

```
R14#ping 77.77.77.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 77.77.77.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 210.110.35.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 111.110.35.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 111.110.35.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### План работы и изменения зафиксированы в документации

**Для доступа к прописанным конфигурациям на роутерах ,то жмём сюда :**

https://e9exu-my.sharepoint.com/:f:/g/personal/nickelface_ermaon_com/Euh_hOXWUWRAr0awcVlpJVYBzobOZWNdcKt4VLkLif40EA?e=GGNmyl