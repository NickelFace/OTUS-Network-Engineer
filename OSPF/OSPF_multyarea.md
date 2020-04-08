# Лабораторная работа. Настройка OSPFv2 для нескольких областей

## Топология

![image-20200407151059761](img/mult/1.png)

## Таблица адресации

| Устройство | Интерфейс    | IP-адрес        | Маска подсети   |
| ---------- | ------------ | --------------- | --------------- |
| R1         | Lo0          | 209.165.200.225 | 255.255.255.252 |
|            | Lo1          | 192.168.1.1     | 255.255.255.0   |
|            | Lo2          | 192.168.2.1     | 255.255.255.0   |
|            | S0/0/0 (DCE) | 192.168.12.1    | 255.255.255.252 |
| R2         | Lo6          | 192.168.6.1     | 255.255.255.0   |
|            | S0/0/0       | 192.168.12.2    | 255.255.255.252 |
|            | S0/0/1 (DCE) | 192.168.23.1    | 255.255.255.252 |
| R3         | Lo4          | 192.168.4.1     | 255.255.255.0   |
|            | Lo5          | 192.168.5.1     | 255.255.255.0   |
|            | S0/0/1       | 192.168.23.2    | 255.255.255.252 |

## Задачи

**Часть 1. Создание сети и настройка основных параметров устройства**

**Часть 2. Настройка сети OSPFv2 для нескольких областей**

**Часть 3. Настройка межобластных суммарных маршрутов**



## Создание сети и настройка основных параметров устройства

**R1**

Enable<br/>
Configure terminal<br/>
hostname R1<br/>
Interface loopback 0<br/>
ip address 209.165.200.225 255.255.255.252<br/>
Interface loopback 1<br/>
ip address 192.168.1.1 255.255.255.0<br/>
Interface loopback 2<br/>
ip address 192.168.2.1 255.255.255.0<br/>
exit<br/>
interface serial 0/0<br/>
ip address 192.168.12.1 255.255.255.252<br/>
clock rate 128000<br/>
no shutdown<br/>
exit<br/>
no ip domain-lookup<br/>
enable secret class<br/>
line vty 0 15<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>
line con 0<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
Banner motd "This is a secure system. Authorized Access Only!"<br/>
do copy run start<br/>
[Enter]<br/>

**R2**

Enable<br/>
Configure terminal<br/>
hostname R2<br/>
Interface loopback 6<br/>
ip address 192.168.6.1 255.255.255.0<br/>
exit<br/>
interface serial 0/0<br/>
ip address 192.168.12.2 255.255.255.252<br/>
no shutdown<br/>
exit<br/>
interface serial 1/0<br/>
ip address 192.168.23.1 255.255.255.252<br/>
clock rate 128000<br/>
no shutdown<br/>
exit<br/>
no ip domain-lookup<br/>
enable secret class<br/>
line vty 0 4<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>
line con 0<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>
Banner motd "This is a secure system. Authorized Access Only!"<br/>
do copy run start<br/>
[Enter]<br/>

**R3**

enable<br/>
configure terminal<br/>
hostname R3<br/>
interface serial 0/0<br/>
ip address 192.168.23.2 255.255.255.252<br/>
no shutdown<br/>
exit<br/>
interface loopback 4 <br/>
ip address 192.168.4.1 255.255.255.0<br/>
exit<br/>
interface loopback 5<br/>
ip address 192.168.5.1 255.255.255.0<br/>
exit<br/>
no ip domain-lookup<br/>
enable secret class<br/>
line vty 0 4<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>
line con 0<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>
Banner motd "This is a secure system. Authorized Access Only!"<br/>
do copy run start<br/>
[Enter]<br/>

**Проверьте наличие подключения на уровне 3.**

Выполните команду **show ip interface brief**, чтобы убедиться в правильности IP-адресации и активности интерфейсов. Убедитесь, что каждый маршрутизатор может успешно отправлять эхо-запросы соседним маршрутизаторам, подключенным с помощью последовательных интерфейсов.

![21](img\mult\21.png)

![22](img\mult\22.png)

![23](img\mult\23.png)

## Настройка сети OSPFv2 для нескольких областей

В части 2 необходимо настроить сеть OSPFv2 для нескольких областей, используя идентификатор процесса 1. Все интерфейсы loopback локальной сети должны быть пассивными, а для всех последовательных интерфейсов должна быть настроена аутентификация MD5 с ключом **Cisco123**

###  Определите типы маршрутизаторов OSPF в топологии.

Определите магистральные маршрутизаторы:  R1, R2, R3

Определите граничные маршрутизаторы автономной системы (ASBR):  R1

Определите граничные маршрутизаторы области (ABR):  R1 , R2

Определите внутренние маршрутизаторы:  R3

### Настройте протокол OSPF на маршрутизаторе R1.

Настроим сети OSPF на R1 , укажем пассивные интерфейсы , а также будем рассказывать по OSPF о дефолтном маршруте выхода в интернет.

router ospf 1
router-id 1.1.1.1
network192.168.1.0 0.0.0.255 area 1
network 192.168.2.0 0.0.0.255 area 1
 network 192.168.12.0 0.0.0.3 area 0
 passive-interface Loopback1
 passive-interface Loopback2
default-information originate
exit 
ip route 0.0.0.0 0.0.0.0 loopback 0
do clear ip ospf process
[yes]

### Настройте протокол OSPF на маршрутизаторе R2.

router ospf 1
router-id 2.2.2.2
network 192.168.6.0 0.0.0.255 area 3
network 192.168.12.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 3
passive-interface Loopback6
do clear ip ospf process
[yes]

### Настройте протокол OSPF на маршрутизаторе R3.

router ospf 1
router-id 3.3.3.3
network 192.168.23.0 0.0.0.3 area 3
passive-interface Loopback4
passive-interface Loopback5
network 192.168.4.0 0.0.0.255 area 3
network 192.168.5.0 0.0.0.255 area 3
do clear ip ospf process
[yes]

###  Убедитесь в правильности настройки протокола OSPF и в установлении отношений смежности между маршрутизаторами.

![image-20200408170643422](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\31.png)

![image-20200408170758777](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\32.png)

Интересно ,то ,что настройку выполнил по схеме , а строки  `It is an area border router`не оказалось. Возможно глюк Packet Tracer.

![image-20200408171138159](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\33.png)

К какому типу маршрутизаторов OSPF относится каждый маршрутизатор?

R1: ASBR , ABR, Backbone router

R2: ABR, Backbone router

R3: Internal router

Убедимся в установлении отношений смежности OSPF между маршрутизаторами.

![image-20200408173119139](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\41.png)

![image-20200408173156400](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\42.png)

![image-20200408173238592](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\43.png)

Команда **show ip ospf interface brief** не реализована в Packet Tracer ,которая отображает   сводку стоимости маршрутов интерфейсов.
Зато show ip ospf interface реализован , с более подробной информацией 
![image-20200408173754239](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\50.png)

На остальных роутерах аналогичная картина, Serial интерфейсы стоимостью 64 , Loopback стоимостью равны 1.

### Настройте аутентификацию MD5 для всех последовательных интерфейсов

**R1** 

interface Serial0/0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 Cisco123

**R2**

interface Serial0/0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 Cisco123

interface Serial1/0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 Cisco123

**R3**

interface Serial0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Cisco123

Почему перед настройкой аутентификации OSPF полезно проверить правильность работы OSPF?

Очевидно же, чтобы проблем себе не прибавлять ,после того, как пропишем процесс OSPF на каждом роутере.

### Проверьте восстановление отношений смежности OSPF.

**повтор**

![image-20200408173119139](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\60.png)

![image-20200408173156400](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\61.png)

![image-20200408173238592](C:\Users\lopunov\AppData\Roaming\Typora\typora-user-images\image-20200408173238592.png)

##  Настройка межобластных суммарных маршрутов

![image-20200408175301332](C:\Users\lopunov\AppData\Roaming\Typora\typora-user-images\image-20200408175301332.png)

![image-20200408175358695](C:\Users\lopunov\AppData\Roaming\Typora\typora-user-images\image-20200408175358695.png)

![image-20200408175435853](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\72.png)

Все маршруты помеченные как (`O IA`) являются межобластным маршрутом.

### Просмотрите базы данных LSDB на всех маршрутизаторах

R1

![image-20200408175737324](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\80.png)

R2

![image-20200408175820321](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\81.png)

R3

![image-20200408175848856](C:\Users\lopunov\Documents\GitHub\OTUS_Network\OSPF\img\mult\82.png)

### Настройте межобластные суммарные маршруты

