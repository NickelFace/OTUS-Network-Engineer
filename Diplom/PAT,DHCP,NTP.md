# Основные протоколы сети интернет 

## Домашнее задание

**Основные протоколы сети интернет**

Цель: Настроить DHCP в офисе Москва Настроить синхронизацию времени в офисе Москва Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001
2. Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042
3. Настроите статический NAT для R20
4. Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления
5. Настроите статический NAT(PAT) для офиса Чокурдах
6. Настроите DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP
7. Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13
8. Все офисы в лабораторной работе должны иметь IP связность
9. План работы и изменения зафиксированы в документации

![](img/EVE_Topology.png)



### Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001

R14

```
interface Ethernet0/0
 ip nat inside

interface Ethernet0/1
 ip nat inside

interface Ethernet0/2
 ip nat outside
 
interface Ethernet0/3
 ip nat inside

interface Ethernet1/0
 ip nat inside
 
 // указываем на какой ip будем транслировать внутренние ip адреса локальной сети:
ip nat pool OVRLD 100.100.100.1 100.100.100.1 netmask 255.255.255.252 

// включаем PAT:
ip nat inside source list 10 pool OVRLD overload 

// указываем пул внутренних ip адресов, которые будем транслировать:
access-list 10 permit 10.10.10.0 0.0.0.31 

```

R15

```
interface Ethernet0/0
 ip nat inside

interface Ethernet0/1
 ip nat inside

interface Ethernet0/2
 ip nat outside
 
interface Ethernet0/3
 ip nat inside

interface Ethernet1/0
 ip nat inside
 
 // указываем на какой ip будем транслировать внутренние ip адреса локальной сети:
ip nat pool OVRLD 111.111.111.1 111.111.111.1 netmask 255.255.255.252

// включаем PAT:
ip nat inside source list 10 pool OVRLD overload 

// указываем пул внутренних ip адресов, которые будем транслировать:
access-list 10 permit 10.10.10.0 0.0.0.31 

```

### Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042

R18

```
interface Ethernet0/2
 ip nat outside
 
 interface Ethernet0/3
 ip nat outside
 
interface Ethernet0/1
 ip nat inside
 
 interface Ethernet0/0
 ip nat inside
 
 // указываем на какой ip будем транслировать внутренние ip адреса локальной сети:
ip nat pool OVRLD1 77.77.77.10 77.77.77.10 netmask 255.255.255.252
ip nat pool OVRLD1 77.77.77.14 77.77.77.14 netmask 255.255.255.252

ip nat inside source list 10 pool OVRLD overload 
ip nat inside source list 11 pool OVRLD1 overload 

// указываем пул внутренних ip адресов, которые будем транслировать:
access-list 10 permit 10.10.20.0 0.0.0.3
access-list 10 permit 10.10.20.4 0.0.0.3
access-list 10 permit 10.10.20.8 0.0.0.3
access-list 10 permit 172.16.10.0 0.0.0.255
access-list 10 permit 172.16.11.0 0.0.0.255

access-list 11 permit 10.10.20.0 0.0.0.3
access-list 11 permit 10.10.20.4 0.0.0.3
access-list 11 permit 10.10.20.8 0.0.0.3
access-list 11 permit 172.16.10.0 0.0.0.255
access-list 11 permit 172.16.11.0 0.0.0.255
```

