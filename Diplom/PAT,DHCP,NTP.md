# ntpОсновные протоколы сети интернет 

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
ip nat pool OVRLD 200.20.20.14 200.20.20.14 netmask 255.255.252.0

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
ip nat pool OVRLD 200.20.20.15 200.20.20.15 netmask 255.255.252.0

// включаем PAT:
ip nat inside source list 10 pool OVRLD overload 

// указываем пул внутренних ip адресов, которые будем транслировать:
access-list 10 permit 10.10.10.0 0.0.0.31 

```

### Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042

R18

```
//Смотрят в сторону ТРИАДА
interface Ethernet0/2
 ip nat outside
 
 interface Ethernet0/3
 ip nat outside
 
//Смотрят внутрь Локальной сети 
interface Ethernet0/1
 ip nat inside
 
 interface Ethernet0/0
 ip nat inside
 
// указываем на какой ip будем транслировать внутренние ip адреса локальной сети:
ip nat pool OVRLD 77.77.77.10 77.77.77.10 netmask 255.255.255.252
ip nat pool OVRLD1 77.77.77.14 77.77.77.14 netmask 255.255.255.252

//Вкл самого NAT
ip nat inside source list 10 pool OVRLD overload 
ip nat inside source list 11 pool OVRLD1 overload 

// указываем пул внутренних ip адресов, которые будем транслировать:
access-list 10 permit 10.10.20.0 0.0.0.3
access-list 10 permit 172.16.10.0 0.0.0.255

access-list 11 permit 10.10.20.4 0.0.0.3
access-list 11 permit 10.10.20.8 0.0.0.3
access-list 11 permit 172.16.11.0 0.0.0.255
```

Первая половина будет NAT за подсеть **77.77.77.8/30** другая за **77.77.77.12/30**

### Настроите статический NAT для R20

R15

```
interface Loopback15
 ip address 215.215.215.215 255.255.255.255

ip nat inside source static 1.1.1.20 215.215.215.215

router bgp 1001
 network 215.215.215.215 mask 255.255.255.255
 
```

R14

```
 ip nat inside source static 1.1.1.20 215.215.215.215

router bgp 1001
 network 215.215.215.215 mask 255.255.255.255
```

Проверку в программе WireShark выполнил ,трафик sniffer смотрел на R20 на interface e0/0

### Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления

 R15

```
interface Loopback19
 ip address 219.219.219.219 255.255.255.255

 router bgp 1001
 	network 219.219.219.219 mask 255.255.255.255

ip nat inside source static tcp 1.1.1.19 22 219.219.219.219 22
```

R14

```
router bgp 1001
 	network 219.219.219.219 mask 255.255.255.255

ip nat inside source static tcp 1.1.1.19 22 219.219.219.219 22
```

R19

```
Hostname R19
ip domain-name Test
crypto key generate rsa
[Enter]
1024
line vty 0 4
transport input ssh
login local
username admin secret cisco
ip ssh version 2
enable secret cisco

interface Loopback19
 ip address 1.1.1.19 255.255.255.255
 
router ospf 1
 network 1.1.1.19 0.0.0.0 area 101

```

### Настроите статический NAT(PAT) для офиса Чокурдах

R28

```
interface ethernet 0/1
  ip nat outside

interface ethernet 0/0
  ip nat outside

interface ethernet 0/2
  ip nat inside

access-list 28 permit 172.16.40.0 /24
ip nat pool OVRLD 111.110.35.10 111.110.35.10 netmask 255.255.255.252
ip nat inside source list 28 pool OVRLD overload

Заодно и DHCP настроил :

ip dhcp excluded-address 172.16.40.1
ip dhcp pool DHCP28
 network 172.16.40.0 255.255.255.0
 default-router 172.16.40.1
```

### Настроите DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP

R12

```
ip dhcp excluded-address 172.16.0.1
ip dhcp pool DHCP12
 network 172.16.0.0 255.255.255.0
 default-router 172.16.0.1
```

R13

```
ip dhcp excluded-address 172.16.1.1
ip dhcp pool DHCP13
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
```

Так как предполагается ,что эти ПК будут в разных отделах ,то заодно покажу настройку  Switch 

SW2

```
ena
conf t
host SW2 

line con 0
exec-t 0 0
no ip domain loo

vlan 7
name LEFT

vlan 8
name RIGHT

vlan 5
nam NATIVE

vlan 100
nam SECURITY

inter loo 2
ip addr 1.1.1.2 255.255.255.255


vlan dot1q tag native // тэгируем родной vlan 

// В сторону SW5
interface Ethernet0/0
 switchport trunk allowed vlan 8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk

// В сторону SW4
interface Ethernet0/1
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk

// В сторону LAN
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode access
 switchport access vlan 8

-----------------------------------
//Отключим остальные

interface ethernet 0/3
switchport mode access	
switchport access vlan 100
shutdown

interface range ethernet 1/0-3
switchport mode access	
switchport access vlan 100
shutdown	
```

SW3

```
ena
conf t
host SW3

line con 0
exec-t 0 0
no ip domain loo

vlan 7
name LEFT

vlan 8
name RIGHT

vlan 5
nam NATIVE

vlan 100
name SECURITY

inter loo 3
ip addr 1.1.1.3 255.255.255.255

vlan dot1q tag native // тэгируем родной vlan 

// В сторону SW4
interface Ethernet0/0
 switchport trunk allowed vlan 7
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
// В сторону SW5
interface Ethernet0/1
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
// В сторону LAN
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode access
 switchport access vlan 7

-----------------------------------
//Отключим остальные

interface ethernet 0/3
switchport mode access	
switchport access vlan 100
shutdown

interface range ethernet 1/0-3
switchport mode access	
switchport access vlan 100
shutdown

```

SW4

```
ena
conf t
host SW4

line con 0
exec-t 0 0
no ip domain loo

vlan 7
name LEFT

vlan 8
name RIGHT

vlan 5
nam NATIVE

inter loo 4
ip addr 1.1.1.4 255.255.255.255

vlan dot1q tag native

interface Port-channel9
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
------------------------------------------------
// В сторону роутера R12

interface Ethernet1/0
 switchport trunk allowed vlan 7
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
-------------------------------------------------
// В сторону SW5

interface Ethernet0/2
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
 channel-protocol pagp
 channel-group 9 mode desirable
!
interface Ethernet0/3
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
 channel-protocol pagp
 channel-group 9 mode desirable

---------------------------------------------------
// В сторону SW2

interface Ethernet0/1
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
---------------------------------------------------
// В сторону SW3

interface Ethernet0/0
 switchport trunk allowed vlan 7
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
----------------------------------------------------
```

SW5

```
ena
conf t
host SW5

line con 0
exec-t 0 0
no ip domain loo

vlan 7
name LEFT

vlan 8
name RIGHT

vlan 5
name NATIVE

inter loo 5
ip addr 1.1.1.5 255.255.255.255


vlan dot1q tag native

interface Port-channel9
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
-----------------------------------------------------

interface Ethernet1/0
 switchport trunk allowed vlan 8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
----------------------------------------------------
interface Ethernet0/2
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
 channel-protocol pagp
 channel-group 9 mode desirable
!
interface Ethernet0/3
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
 channel-protocol pagp
 channel-group 9 mode desirable

--------------------------------------------------
interface Ethernet0/1
 switchport trunk allowed vlan 7,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk

interface Ethernet0/0
 switchport trunk allowed vlan 8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 5
 switchport mode trunk
```

**Результат:**

VPC1

```
VPCS> dhcp -r
DDORA IP 172.16.0.2/24 GW 172.16.0.1
```

VPC7

```
VPCS> dhcp -r
DDORA IP 172.16.1.2/24 GW 172.16.1.1
```

### Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13

Сначала настроим NTP сервера:

R13

```
interface Ethernet0/2
 ntp broadcast

interface Ethernet0/3
 ntp broadcast

ntp source Loopback13
ntp master 5
ntp update-calendar
```

R12

```
interface Ethernet0/2
 ntp broadcast

interface Ethernet0/3
 ntp broadcast

ntp source Loopback12
ntp master 5
ntp update-calendar
```

Потом настроим NTP клиентов:
R14

```
interface Ethernet0/0
 ntp broadcast client
!
interface Ethernet0/1
 ntp broadcast client

ntp server 1.1.1.12
ntp server 1.1.1.13
```

R15

```
interface Ethernet0/0
 ntp broadcast client
!
interface Ethernet0/1
 ntp broadcast client

ntp server 1.1.1.12
ntp server 1.1.1.13
```

R20

```
interface Ethernet0/0
 ntp broadcast client

ntp server 1.1.1.12
ntp server 1.1.1.13
```

R19

```
interface Ethernet0/0
 ntp broadcast client

ntp server 1.1.1.12
ntp server 1.1.1.13
```

Свичи я настраивать не стал(но попробовал всеми способами) ,так как они L2 и они не пингуются с роутерами ,а значит и настроить NTP не получиться.


### Все офисы в лабораторной работе должны иметь IP связность

Проверяем с R15

```
R15>ping 210.110.35.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms
R15>ping 111.110.35.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15>ping 111.110.35.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 111.110.35.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15>ping 77.77.77.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R15>ping 77.77.77.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 77.77.77.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```


**Столкнулся  проблемой ,устройства которые за NAT в МСК не могут пропинговать только сеть Лабытнанги(другие могут)**

```
R20>ping 210.110.35.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R20>trace 210.110.35.2
Type escape sequence to abort.
Tracing the route to 210.110.35.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.10.10.1 1 msec 1 msec 0 msec
  2 111.111.111.2 1 msec 1 msec 0 msec
  3 111.111.111.6 1 msec 1 msec 1 msec
  4  *  *  *
  5  *  *  *
  6  *  *

```

Сеть 111.111.111.6 принадлежит R24 ,пробуем пропинговать от него туда и обратно(учитываем NAT)

```
R24>ping 200.20.20.20 // Статический NAT для R20
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.20.20.20, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R24>traceroute 200.20.20.20
Type escape sequence to abort.
Tracing the route to 200.20.20.20
VRF info: (vrf in name/id, vrf out name/id)
  1 111.111.111.5 [AS 301] 1 msec 0 msec 1 msec
  2 111.111.111.1 [AS 301] 1 msec 1 msec 1 msec
  3 200.20.20.20 [AS 1001] 1 msec *  1 msec


R24>ping 210.110.35.2 // сеть Лабытнанги
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 210.110.35.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R24>traceroute 210.110.35.2
Type escape sequence to abort.
Tracing the route to 210.110.35.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.10.30.14 1 msec
    10.10.30.1 0 msec
    10.10.30.14 1 msec
  2 10.10.30.6 0 msec
    10.10.30.9 1 msec
    10.10.30.6 1 msec
  3 210.110.35.2 1 msec *  2 msec

```

Я так и не смог понять,в чем собственно причина такого поведения, буду рад ,если поможете разобраться.

### План работы и изменения зафиксированы в документации

**Для доступа к прописанным конфигурациям на роутерах ,то жмём сюда :**

https://e9exu-my.sharepoint.com/:f:/g/personal/nickelface_ermaon_com/Euh_hOXWUWRAr0awcVlpJVYBzobOZWNdcKt4VLkLif40EA?e=GGNmyl

