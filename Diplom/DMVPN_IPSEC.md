# IPsec

![](img/EVE_Topology.png)

## Домашнее задание

IPSec over DmVPN

Цель: Настроить GRE поверх IPSec между офисами Москва и С.-Петербург Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите GRE поверх IPSec между офисами Москва и С.-Петербург
2. Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги
3. Все узлы в офисах в лабораторной работе должны иметь IP связность
4. План работы и изменения зафиксированы в документации

### Настроите GRE поверх IPSec между офисами Москва и С.-Петербург

R15

```
interface Tunnel0
 ip address 10.0.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 111.111.111.1
 tunnel destination 77.77.77.14

 crypto isakmp policy 1
 encr 3des
 hash md5
 authentication pre-share
 group 2
 lifetime 86400
 
crypto isakmp key cisco address 77.77.77.14

crypto ipsec transform-set TS esp-3des esp-md5-hmac
 mode transport

crypto ipsec profile protect-gre
 set security-association lifetime seconds 86400
 set transform-set TS

interface Tunnel0
 tunnel protection ipsec profile protect-gre
```

R18

```
interface Tunnel0
 ip address 10.0.0.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 77.77.77.14
 tunnel destination 111.111.111.1

crypto isakmp policy 1
 encr 3des
 hash md5
 authentication pre-share
 group 2
 lifetime 86400
 
crypto isakmp key cisco address 111.111.111.1

crypto ipsec transform-set TS esp-3des esp-md5-hmac
 mode transport

crypto ipsec profile protect-gre
 set security-association lifetime seconds 86400
 set transform-set TS

interface Tunnel0
 tunnel protection ipsec profile protect-gre
```

Проверка

```
R18(config-if)#do ping 10.0.0.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/6/6 ms

R18(config-if)#do ping 10.0.0.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/5 ms

R18(config-if)#do show crypto session
Crypto session current status

Interface: Tunnel0
Session status: UP-ACTIVE     
Peer: 111.111.111.1 port 500 
  Session ID: 0  
  IKEv1 SA: local 77.77.77.14/500 remote 111.111.111.1/500 Active 
  IPSEC FLOW: permit 47 host 77.77.77.14 host 111.111.111.1 
        Active SAs: 2, origin: crypto map
```



### Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги

R14

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
 
 
crypto isakmp policy 1
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key isakmp1234 address 0.0.0.0        
         
         
crypto ipsec transform-set TS esp-3des esp-md5-hmac 
 mode tunnel
         
         
crypto ipsec profile protect-gre
 set security-association lifetime seconds 86400
 set transform-set TS 

interface Tunnel0
 tunnel protection ipsec profile protect-gre
```

R27

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
 
 crypto isakmp policy 1
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key isakmp1234 address 0.0.0.0        
         
         
crypto ipsec transform-set TS esp-3des esp-md5-hmac 
 mode tunnel
         
         
crypto ipsec profile protect-gre
 set security-association lifetime seconds 86400
 set transform-set TS 
 
interface Tunnel0
 tunnel protection ipsec profile protect-gre
```

R28

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
 tunnel protection ipsec profile protect-gre
 
crypto isakmp policy 1
 encr 3des
 hash md5 
 authentication pre-share
 group 2  
crypto isakmp key isakmp1234 address 0.0.0.0        
         
         
crypto ipsec transform-set TS esp-3des esp-md5-hmac 
 mode tunnel
         
         
crypto ipsec profile protect-gre
 set security-association lifetime seconds 86400
 set transform-set TS 
 
interface Tunnel0
 tunnel protection ipsec profile protect-gre 
```

Проверка

```
R27#show dmvpn            
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:3, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 100.100.100.1          10.1.0.1    UP 00:55:19     S
                              10.1.0.4    UP 00:00:06     D
     1 77.77.77.10            10.1.0.2    UP 00:19:13     D
     1 111.110.35.14          10.1.0.3    UP 00:19:11     D
--------------------------------------------------------------------------------------------------------------------
R27#show crypto isakmp sa 
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
210.110.35.2    111.110.35.14   QM_IDLE           1003 ACTIVE
111.110.35.14   210.110.35.2    QM_IDLE           1001 ACTIVE
100.100.100.1   210.110.35.2    QM_IDLE           1002 ACTIVE

IPv6 Crypto ISAKMP SA

--------------------------------------------------------------------------------------------------------------------
R27#ping 10.1.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/6/6 ms
R27#ping 10.1.0.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/6 ms
R27#ping 10.1.0.4
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.4, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/7/8 ms 
```

### План работы и изменения зафиксированы в документации

https://1drv.ms/u/s!AiW_chHQt5JCg64XQ0nBMdjjoQ3bLw?e=B1LOJE