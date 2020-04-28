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

![image](img/PBR.png)

1) Настроите политику маршрутизации для сетей офиса Чокурдах 

Четко что нужно сделать не написано,ок потренируюсь с определенного ip маршрутизировать трафик :

```
ip access-list extended ACL1
 permit ip host 172.16.40.11 any

route-map TEST permit 10
 match ip address ACL1
 set ip next-hop 111.110.35.13
 
interface Ethernet0/2
 ip policy route-map TEST
```

VPC30

```
VPCS> trace 6.6.6.6
trace to 6.6.6.6, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   0.755 ms  0.719 ms  0.677 ms
 2   111.110.35.9   0.932 ms  0.833 ms  0.708 ms
 3   *10.10.30.18   1.084 ms (ICMP type:3, code:3, Destination port unreachable)
```

VPC31

```
VPCS> trace 6.6.6.6
trace to 6.6.6.6, 8 hops max, press Ctrl+C to stop
 1   172.16.40.1   1.488 ms  1.462 ms  1.425 ms
 2   111.110.35.13   1.981 ms  1.662 ms  1.628 ms
 3   *10.10.30.22   2.379 ms (ICMP type:3, code:3, Destination port unreachable)
```

2) Распределите трафик между двумя линками с провайдером в Чокурдах

```
R28(config)#ip route 0.0.0.0 0.0.0.0 111.110.35.9
R28(config)#ip route 0.0.0.0 0.0.0.0 111.110.35.13

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

3) Настроите отслеживание линка через технологию IP SLA в Чокурдах

Так как у нас будет балансировка при падении линка, убираем равнозначные статические маршруты

```
R28(config)#no ip route 0.0.0.0 0.0.0.0 111.110.35.9
R28(config)#ip route 0.0.0.0 0.0.0.0 111.110.35.9 track 1
R28(config)#ip route 0.0.0.0 0.0.0.0 111.110.35.13 10

R28(config)# ip sla 1
R28(config-ip-sla)# icmp-echo 6.6.6.6 source-interface Ethernet0/1
R28(config-ip-sla-echo)#frequency 5

R28(config)#ip sla schedule 1 life forever start-time now

R28(config)#track 1 ip sla 1 reachability
```

Проверяем работу SLA

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

Gateway of last resort is 111.110.35.9 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 111.110.35.9
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

R28(config)#
*Apr 28 19:58:41.797: %TRACK-6-STATE: 1 ip sla 1 reachability Up -> Down
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

S*    0.0.0.0/0 [10/0] via 111.110.35.13
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

Как видим,при падении линка у нас автоматом меняется статический маршрут.

4) Настройте для офиса Лабытнанги маршрут по-умолчанию

```
R27(config)#ip route 0.0.0.0 0.0.0.0 210.110.35.1
```

Между Триадой и Лабытнанги сеть **210.110.35.0 /30**

5) План работы и изменения зафиксированы в документации 

Веду отдельный файл для этого, если требуется ,то покажу.