## Лабораторная работа. Поиск и устранение неполадок в работе EtherChannel



### Топология

![img](img/21.png)



### Таблица адресации

| **Устройство** | **Интерфейс** | **IP-адрес** | **Маска подсети** |
| -------------- | ------------- | ------------ | ----------------- |
| S1             | VLAN 99       | 192.168.1.11 | 255.255.255.0     |
| S2             | VLAN 99       | 192.168.1.12 | 255.255.255.0     |
| S3             | VLAN 99       | 192.168.1.13 | 255.255.255.0     |
| PC-A           | NIC           | 192.168.0.2  | 255.255.255.0     |
| PC-C           | NIC           | 192.168.0.3  | 255.255.255.0     |

### Назначения сети VLAN

| VLAN | Имя        |
| ---- | ---------- |
| 10   | User       |
| 99   | Управление |

### Задачи

Часть 1. Построение сети и загрузка настроек устройств

Часть 2. Отладка EtherChannel



###  Построение сети и загрузка настроек устройств

**Конфигурация коммутатора S1:**

hostname S1<br/>
interface range f0/1-24, g0/1-2<br/>
shutdown<br/>
exit<br/>
enable secret class<br/>
no ip domain lookup<br/>
line vty 0 15<br/>
password cisco<br/>
login<br/>
line con 0<br/>
password cisco<br/>
logging synchronous<br/>
login<br/>
exit<br/>
vlan 10<br/>
name User<br/>
vlan 99<br/>
Name Management<br/>
interface range f0/1-2<br/>
switchport mode trunk<br/>
channel-group 1 mode active<br/>
switchport trunk native vlan 99<br/>
no shutdown<br/>
interface range f0/3-4<br/>
channel-group 2 mode desirable<br/>
switchport trunk native vlan 99<br/>
no shutdown<br/>
interface f0/6<br/>
switchport mode access<br/>
switchport access vlan 10<br/>
no shutdown<br/>
interface vlan 99<br/>
ip address 192.168.1.11 255.255.255.0<br/>
interface port-channel 1<br/>
switchport trunk native vlan 99<br/>
switchport mode trunk<br/>
interface port-channel 2<br/>
switchport trunk native vlan 99<br/>
switchport mode access<br/>

**Конфигурация коммутатора S2:**

hostname S2<br/>
interface range f0/1-24, g0/1-2<br/>
shutdown<br/>
exit<br/>
enable secret class<br/>
no ip domain lookup<br/>
line vty 0 15<br/>
password cisco<br/>
login<br/>
line con 0<br/>
password cisco<br/>
logging synchronous<br/>
login<br/>
exit<br/>
vlan 10<br/>
name User<br/>
vlan 99<br/>
name Management<br/>
spanning-tree vlan 1,10,99 root primary<br/>
interface range f0/1-2<br/>
switchport mode trunk<br/>
channel-group 1 mode desirable<br/>
switchport trunk native vlan 99<br/>
no shutdown<br/>
interface range f0/3-4<br/>
switchport mode trunk<br/>
channel-group 3 mode desirable<br/>
switchport trunk native vlan 99<br/>
interface vlan 99<br/>
ip address 192.168.1.12 255.255.255.0<br/>
interface port-channel 1<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,99<br/>
interface port-channel 3<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,10,99,99<br/>
switchport mode trunk<br/>

**Конфигурация коммутатора S3:**

hostname S3<br/>
interface range f0/1-24, g0/1-2<br/>
shutdown<br/>
exit<br/>
enable secret class<br/>
no ip domain lookup<br/>
line vty 0 15<br/>
password cisco<br/>
login<br/>
line con 0<br/>
password cisco<br/>
logging synchronous<br/>
login<br/>
exit<br/>
vlan 10<br/>
name User<br/>
vlan 99<br/>
name Management<br/>
interface range f0/1-2<br/>
interface range f0/3-4<br/>
switchport mode trunk<br/>
channel-group 3 mode desirable<br/>
switchport trunk native vlan 99<br/>
no shutdown<br/>
interface f0/18<br/>
switchport mode access<br/>
switchport access vlan 10<br/>
no shutdown<br/>
interface vlan 99<br/>
ip address 192.168.1.13 255.255.255.0<br/>
interface port-channel 3<br/>
no switchport trunk native vlan 99<br/>
no switchport mode trunk<br/>

#### Шаг 1:   Выполните поиск и устранение неполадок в работе маршрутизатора S1.

**show interfaces trunk** - команда пустая ,а это значит не в одну сторону не настроены trunk порты

**show etherchannel summary** - указывает настроенный LACP, что противоречит условию

![img](img/24.png)

S1(config-if)# **do show run | begin interface Port-channel**  - на 2 группе каналов стоит режим доступа

Исправим это :

S1(config)# 
<<<<<<< HEAD

interface port-channel 2<br/>
switchport mode trunk<br/>
switchport trunk allowed vlan 1,10,99<br/>

no interface port-channel 1<br/>
interface port-channel 1<br/>
switchport mode trunk<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,10,99<br/>

interface range fastEthernet 0/1-2<br/>
channel-group 1 mode auto <br/>

#### Шаг 2:   Выполните поиск и устранение неполадок в работе маршрутизатора S2.

![img](img/23.png)

S2(config)#interface port-channel 1<br/>
no switchport trunk allowed vlan 1,99<br/>
switchport trunk allowed vlan 1,10,99<br/>

interface range fastEthernet 0/3-4<br/>
no shutdown <br/>

#### Шаг 3:   Выполните поиск и устранение неполадок в работе маршрутизатора S3.

На 3 перепутаны процессы PAgP и отсутствуют настройки на interface range fastEthernet 0/1-2<br/> 
Так что переделаем все заново настройки .

interface range fastEthernet 0/3-4<br/>
no channel-group 3 mode desirable<br/>
no switchport trunk native vlan 99<br/>
no switchport mode trunk<br/>

interface range fastEthernet 0/1-2<br/>
channel-group 3 mode auto <br/>

interface port-channel 3<br/>
switchport mode trunk<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,10,99<br/>

interface range fastEthernet 0/3-4<br/>
 channel-group 2 mode auto<br/>

interface port-channel 2<br/>
switchport mode trunk<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,10,99<br/>


interface port-channel 2<br/>
switchport mode trunk<br/>

interface range f0/1-2<br/>
no channel-group 1 mode active<br/>

no interface port-channel 1<br/>

int r f0/1-2<br/>
channel-group 1 mode auto<br/>

interface port-channel 1<br/>
switchport mode trunk<br/>
switchport trunk native vlan 99<br/>
switchport trunk allowed vlan 1,10<br/>



#### Шаг 4: Проверка связности коммутаторов S1, S2, S3 и  ПК А , ПК С.

При проверке ,оказалось, что ping не проходят , тогда я выключил  interface vlan 1 и поднял 99.
После проделанных манипуляций, всё заработало. Чтобы не городить огород из скриншотов ,покажу логи за S3.

![img](img/22.png)s

Ping с ПК А на С тоже идут ,  а это значит, что с задачей справился.

![img](img/ping.png)













