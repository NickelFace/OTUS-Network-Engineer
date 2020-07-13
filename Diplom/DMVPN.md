# Виртуальная частные сети - VPN

![](img/EVE_Topology.png)

## Домашнее задание

Цель: Настроить GRE между офисами Москва и С.-Петербург Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите GRE между офисами Москва и С.-Петербург
2.  Настроите DMVMN между Москва и Чокурдах, Лабытнанги
3.  Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации

### Настроите GRE между офисами Москва и С.-Петербург

R15

```
interface Tunnel0
 ip address 10.0.0.1 255.255.255.252
 tunnel source 111.111.111.1
 tunnel destination 77.77.77.14
```

R18

```
interface Tunnel0
 ip address 10.0.0.2 255.255.255.252
 tunnel source 77.77.77.14
 tunnel destination 111.111.111.1
```

### Настроите DMVMN между Москва и Чокурдах, Лабытнанги

R14 (Hub)

```
interface Tunnel0
 description DMVPN Tunnel
 ip address 10.1.0.1 255.255.255.0
 no ip redirects
 ip mtu 1440
 ip nhrp authentication nhrp1234
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 load-interval 30
 keepalive 5 10
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
```

R18 (Spoke)

```
interface Tunnel1
 ip address 10.1.0.2 255.255.255.0
 no ip redirects
 ip mtu 1440
 ip nhrp authentication nhrp1234
 ip nhrp map multicast dynamic
 ip nhrp map 10.1.0.1 100.100.100.1
 ip nhrp map multicast 100.100.100.1
 ip nhrp network-id 1
 ip nhrp nhs 10.1.0.1
 ip nhrp registration no-unique
 load-interval 30
 keepalive 5 10
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
```

R28 (Spoke)

```
interface Tunnel0
 ip address 10.1.0.3 255.255.255.0
 no ip redirects
 ip mtu 1440
 ip nhrp authentication nhrp1234
 ip nhrp map multicast dynamic
 ip nhrp map 10.1.0.1 100.100.100.1
 ip nhrp map multicast 100.100.100.1
 ip nhrp network-id 1
 ip nhrp nhs 10.1.0.1
 ip nhrp registration no-unique
 load-interval 30
 keepalive 5 10
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

R27 (Spoke)

```
interface Tunnel0
 ip address 10.1.0.4 255.255.255.0
 no ip redirects
 ip mtu 1440
 ip nhrp authentication nhrp1234
 ip nhrp map multicast dynamic
 ip nhrp map 10.1.0.1 100.100.100.1
 ip nhrp map multicast 100.100.100.1
 ip nhrp network-id 1
 ip nhrp nhs 10.1.0.1
 ip nhrp registration no-unique
 load-interval 30
 keepalive 5 10
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

### Все узлы в офисах в лабораторной работе должны иметь IP связность

R14

```
DMVPN

R14#ping 10.1.0.4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 10.1.0.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 10.1.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R14#ping 10.1.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/5 ms
```

R15 

```
GRE Tunnel

R15#ping  10.0.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15#ping  10.0.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/5 ms
```

### План работы и изменения зафиксированы в документации

https://1drv.ms/u/s!AiW_chHQt5JCg64XQ0nBMdjjoQ3bLw?e=B1LOJE