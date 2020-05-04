# Маршрутизация на основе политик (PBR)

## Домашнее задание

### PBR

Цель: Настроить политику маршрутизации в офисе Чокурдах Распределить трафик между 2 линками

  В этой  самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите политику маршрутизации для сетей офиса Чокурдах
2. Распределите трафик между двумя линками с провайдером в Чокурдах
3. Настроите отслеживание линка через технологию IP SLA в Чокурдах
4. Настройте для офиса Лабытнанги маршрут по-умолчанию
5. План работы и изменения зафиксированы в документации 

![PBR](img/PBR.png)

| Устройств | Port | IPv4             | Примечание      | IP-Loopback | Регион   |
| --------- | ---- | ---------------- | --------------- | ----------- | -------- |
| R28       | e0/0 | 111.110.35.14/30 | R26(Triada)     | 1.1.3.28    | Чокурдах |
|           | e0/1 | 111.110.35.10/30 | R25(Triada)     |             |          |
|           | e0/2 | 172.16.40.1/24   | SW29(Chukordah) |             |          |
| SW29      | e0/0 |                  |                 | 1.1.3.29    |          |
|           | e0/1 |                  |                 |             |          |
|           | e0/2 |                  |                 |             |          |
| VPC30     |      | 172.16.40.10     |                 |             |          |
| VPC31     |      | 172.16.40.11     |                 |             |          |

| Устройств | Port | IPv6                 | Примечание      | Link-Local   |  Регион  |
| --------- | ---- | -------------------- | --------------- | ------------ | :------: |
| R28       | e0/0 | 2CAD:1995:B0DA:1::D  | R26(Triada)     |              | Чокурдах |
|           | e0/1 | 2CAD:1995:B0DA:1::E  | R25(Triada)     |              |          |
|           | e0/2 | 2CAD:1995:B0DA:1::F  | SW29(Chukordah) |              |          |
| SW29      | e0/0 | 2CAD:1995:B0DA:1::10 |                 |              |          |
|           | e0/1 |                      |                 |              |          |
|           | e0/2 |                      |                 | fe80:21::28  |          |
| VPC30     |      |                      |                 | fe80:21::130 |          |
| VPC31     |      |                      |                 | fe80:21::131 |          |

Буду выполнять ДЗ не по порядку,а наоборот ,снизу вверх . Так как без SLA это будет работать довольно скверно. 

**Настройте для офиса Лабытнанги маршрут по-умолчанию:**

```
R27(config)# ip route 0.0.0.0 0.0.0.0 210.110.35.1
```

**Настроите отслеживание линка через технологию IP SLA в Чокурдах**

R28

```
ip sla 1
 icmp-echo 10.10.30.18 source-interface Ethernet0/1
 frequency 5
ip sla schedule 1 life forever start-time now

ip sla 2
 icmp-echo 10.10.30.22 source-interface Ethernet0/0
 frequency 5
ip sla schedule 2 life forever start-time now

track 1 ip sla 1 reachability

track 2 ip sla 2 reachability

ip route 0.0.0.0 0.0.0.0 111.110.35.9 track 1
ip route 0.0.0.0 0.0.0.0 111.110.35.13 track 2
```

Так как без прописанных статических маршрутов это работать не будет, то:

```
ip route 10.10.30.16 255.255.255.252 111.110.35.9
ip route 10.10.30.20 255.255.255.252 111.110.35.13
```

(Этот хитрый способ мне пришел недавно в голову)

Проверим работу ,когда оба линка живы:

```
R28(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 111.110.35.13 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 111.110.35.13
                [1/0] via 111.110.35.9
      10.0.0.0/30 is subnetted, 2 subnets
S        10.10.30.16 [1/0] via 111.110.35.9
S        10.10.30.20 [1/0] via 111.110.35.13
      28.0.0.0/32 is subnetted, 1 subnets
C        28.28.28.28 is directly connected, Loopback0
      111.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        111.110.35.8/30 is directly connected, Ethernet0/1
L        111.110.35.10/32 is directly connected, Ethernet0/1
C        111.110.35.12/30 is directly connected, Ethernet0/0
L        111.110.35.14/32 is directly connected, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.40.0/24 is directly connected, Ethernet0/2
L        172.16.40.1/32 is directly connected, Ethernet0/2
```

Когда 1 из линков падает : 

```
*May  4 12:04:53.424: %TRACK-6-STATE: 2 ip sla 2 reachability Up -> Down
R28(config)#do sh ip route

Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 111.110.35.9 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 111.110.35.9
      10.0.0.0/30 is subnetted, 2 subnets
S        10.10.30.16 [1/0] via 111.110.35.9
S        10.10.30.20 [1/0] via 111.110.35.13
      28.0.0.0/32 is subnetted, 1 subnets
C        28.28.28.28 is directly connected, Loopback0
      111.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        111.110.35.8/30 is directly connected, Ethernet0/1
L        111.110.35.10/32 is directly connected, Ethernet0/1
C        111.110.35.12/30 is directly connected, Ethernet0/0
L        111.110.35.14/32 is directly connected, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.40.0/24 is directly connected, Ethernet0/2
L        172.16.40.1/32 is directly connected, Ethernet0/2
```

Работой данного SLA мы как раз поделили трафик ,падает с одной стороны ,удалиться один маршрут,но второй по прежнему работать будет и наоборот. 

**Распределите трафик между двумя линками с провайдером в Чокурдах**

R28

```
interface Ethernet0/2
ip policy route-map TEST

ip access-list extended ACL1
 permit ip host 172.16.40.10 any
 
route-map TEST permit 10
 match ip address ACL1
 set ip default next-hop 111.110.35.13
```

Команда **set ip default next-hop** проверяет, существует ли IP-адрес назначения в таблице маршрутизации, и действует следующим образом:

- если IP-адрес назначения существует, команда не применяет политики для маршрутизации пакета, а пересылает его на основе таблицы маршрутизации;
- если IP-адрес назначения не существует, команда выполняет маршрутизацию пакета на основе политики, отправляя его в указанный следующий узел.

VPC30

```
VPCS> trace 6.6.6.6
trace to 6.6.6.6, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   1.248 ms  1.086 ms  1.332 ms
 2   111.110.35.13   1.995 ms  1.979 ms  1.669 ms
 3   *10.10.30.22   2.123 ms (ICMP type:3, code:3, Destination port unreachable)  *
```

А теперь гасим на R26  и посмотрим что получится:

При этом помним ,что на R28 гаситься track, а значит меняется таблица маршрутизации

```
*May  4 12:22:09.165: %TRACK-6-STATE: 2 ip sla 2 reachability Up -> Down
```

 После падения линка на VPC30

```
VPCS> trace 6.6.6.6
trace to 6.6.6.6, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   0.942 ms  0.797 ms  0.730 ms
 2   111.110.35.9   0.995 ms  0.915 ms  0.893 ms
 3   *10.10.30.18   1.213 ms (ICMP type:3, code:3, Destination port unreachable)  *
```

