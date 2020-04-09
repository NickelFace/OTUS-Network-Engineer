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

![21](img/mult/21.png)

![22](img/mult/22.png)

![23](img/mult/23.png)

## Настройка сети OSPFv2 для нескольких областей

В части 2 необходимо настроить сеть OSPFv2 для нескольких областей, используя идентификатор процесса 1. Все интерфейсы loopback локальной сети должны быть пассивными, а для всех последовательных интерфейсов должна быть настроена аутентификация MD5 с ключом **Cisco123**

###  Определите типы маршрутизаторов OSPF в топологии.

Определите магистральные маршрутизаторы:  R1, R2, R3

Определите граничные маршрутизаторы автономной системы (ASBR):  R1

Определите граничные маршрутизаторы области (ABR):  R1 , R2

Определите внутренние маршрутизаторы:  R3

### Настройте протокол OSPF на маршрутизаторе R1.

Настроим сети OSPF на R1 , укажем пассивные интерфейсы , а также будем рассказывать по OSPF о дефолтном маршруте выхода в интернет.

router ospf 1<br/>
router-id 1.1.1.1<br/>
network192.168.1.0 0.0.0.255 area 1<br/>
network 192.168.2.0 0.0.0.255 area 1<br/>
 network 192.168.12.0 0.0.0.3 area 0<br/>
 passive-interface Loopback1<br/>
 passive-interface Loopback2<br/>
default-information originate<br/>
exit <br/>
ip route 0.0.0.0 0.0.0.0 loopback 0<br/>
do clear ip ospf process<br/>
[yes]<br/>

### Настройте протокол OSPF на маршрутизаторе R2.

router ospf 1<br/>
router-id 2.2.2.2<br/>
network 192.168.6.0 0.0.0.255 area 3<br/>
network 192.168.12.0 0.0.0.3 area 0<br/>
network 192.168.23.0 0.0.0.3 area 3<br/>
passive-interface Loopback6<br/>
do clear ip ospf process<br/>
[yes]<br/>

### Настройте протокол OSPF на маршрутизаторе R3.

router ospf 1<br/>
router-id 3.3.3.3<br/>
network 192.168.23.0 0.0.0.3 area 3<br/>
passive-interface Loopback4<br/>
passive-interface Loopback5<br/>
network 192.168.4.0 0.0.0.255 area 3<br/>
network 192.168.5.0 0.0.0.255 area 3<br/>
do clear ip ospf process<br/>
[yes]<br/>

###  Убедитесь в правильности настройки протокола OSPF и в установлении отношений смежности между маршрутизаторами.

![image]( img/mult/31.png)

![image]( img/mult/32.png)

Интересно ,то ,что настройку выполнил по схеме , а строки  `It is an area border router`не оказалось. Возможно глюк Packet Tracer.

![image]( img/mult/33.png)

К какому типу маршрутизаторов OSPF относится каждый маршрутизатор?

R1: ASBR , ABR, Backbone router

R2: ABR, Backbone router

R3: Internal router

Убедимся в установлении отношений смежности OSPF между маршрутизаторами.

![image-20200408173119139]( img/mult/41.png)

![image-20200408173156400]( img/mult/42.png)

![image-20200408173238592]( img/mult/43.png)

Команда **show ip ospf interface brief** не реализована в Packet Tracer ,которая отображает   сводку стоимости маршрутов интерфейсов.
Зато show ip ospf interface реализован , с более подробной информацией 
![image-20200408173754239]( img/mult/50.png)

На остальных роутерах аналогичная картина, Serial интерфейсы стоимостью 64 , Loopback стоимостью равны 1.

### Настройте аутентификацию MD5 для всех последовательных интерфейсов

**R1** 

interface Serial0/0<br/>
ip ospf authentication message-digest<br/>
ip ospf message-digest-key 1 md5 Cisco123<br/>

**R2**

interface Serial0/0<br/>
ip ospf authentication message-digest<br/>
ip ospf message-digest-key 1 md5 Cisco123<br/>

interface Serial1/0<br/>
ip ospf authentication message-digest<br/>
ip ospf message-digest-key 1 md5 Cisco123<br/>

**R3**

interface Serial0/0<br/>
 ip ospf authentication message-digest<br/>
 ip ospf message-digest-key 1 md5 Cisco123<br/>

Почему перед настройкой аутентификации OSPF полезно проверить правильность работы OSPF?

Очевидно же, чтобы проблем себе не прибавлять ,после того, как пропишем процесс OSPF на каждом роутере.

### Проверьте восстановление отношений смежности OSPF.

![image-20200408173119139]( img/mult/60.png)

![image-20200408173156400]( img/mult/61.png)

![image]( img/mult/62.png)

##  Настройка межобластных суммарных маршрутов

![image]( img/mult/70.png)

![image]( img/mult/71.png)

![image]( img/mult/72.png)

Все маршруты помеченные как (`O IA`) являются межобластным маршрутом.

### Просмотрите базы данных LSDB на всех маршрутизаторах

R1

![image]( img/mult/80.png)

R2

![image]( img/mult/81.png)

R3

![image]( img/mult/82.png)

### Настройте межобластные суммарные маршруты

Настроим суммарный маршрут для сетей в области 1 на R1.

R1(config)# **router ospf 1**<br/>
R1(config-router)# **area 1 range 192.168.0.0 255.255.252.0**<br/>

![image](img/mult/90.png)

Настроим суммарный маршрут для сетей в области 3 на R2.

R2(config)# **router ospf 1**<br/>
R2(config-router)#**area 3 range 192.168.4.0 255.255.254.0**<br/>

![image](img/mult/91.png)

![image](img/mult/92.png)

На скриншотах выше показано использование команды и её результаты работы ,которые повлияли на таблицу маршрутизации. Таблица стала меньше ,более компактной.

### Повторно отобразите таблицы маршрутизации OSPF для всех маршрутизаторов.

Выполним команду **show ip route ospf** на каждом маршрутизаторе. Запишим результаты для суммарных и межобластных маршрутов.

![image]( img/mult/100.png)

![image]( img/mult/101.png)

![image]( img/mult/102.png)

### Просмотрите базы данных LSDB на всех маршрутизаторах.

R1

![image]( img/mult/110.png)

R2

![image]( img/mult/111.png)

R3

![image]( img/mult/112.png)



Пакет LSA какого типа передается в магистраль маршрутизатором ABR, когда включено объединение межобластных маршрутов?

Summary Net Link States , то есть LSA 3 типа

###  Проверьте наличие сквозного соединения.

Проверил  доступность каждой сети . Как и в начале , до каждой сети можно добраться с каждого роутера. Не вижу смысла выводить скриншоты.

Вопросы для повторения

Какие три преимущества при проектировании сети предоставляет OSPF для нескольких областей?

- **Снижение накладных расходов на обновление состояний каналов.**
- **Таблицы маршрутизации меньшего размера**. 
- **Снижение частоты расчётов кратчайшего пути SPF**.

