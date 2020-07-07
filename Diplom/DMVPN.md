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

Проверка

```
R15#ping 10.0.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

### Настроите DMVMN между Москва и Чокурдах, Лабытнанги