## Лабораторная работа. Настройка EtherChannel

### Топология

![img](C:\Users\Admin\Documents\GitHub\OTUS_Network\EtherChannel\img/1.png)

### Таблица адресации

| Устройство | Интерфейс | IP-адрес      | Маска подсети |
| ---------- | --------- | ------------- | ------------- |
| S1         | VLAN 99   | 192.168.99.11 | 255.255.255.0 |
| S2         | VLAN 99   | 192.168.99.12 | 255.255.255.0 |
| S3         | VLAN 99   | 192.168.99.13 | 255.255.255.0 |
| PC-A       | NIC       | 192.168.10.1  | 255.255.255.0 |
| PC-B       | NIC       | 192.168.10.2  | 255.255.255.0 |
| PC-C       | NIC       | 192.168.10.3  | 255.255.255.0 |

### Цели

##### Часть 1. Настройка базовых параметров коммутатора

##### Часть 2. Настройка PAgP

##### Часть 3. Настройка LACP



### Настройка базовых параметров коммутатора

Создание сети из примера лабораторной работы , а также настройка базовых параметров на коммутаторах: Имя хоста, создание inter vlan 99 , назначение ip-address , назначение пароля class на привилегированный режим , пароль на VTY ,  настроим предотвращение прерывание команд , отключение портов на коммутаторе , создание VLAN 99 , 10 именем **Management** и **Staff** соответственно, сохранение конфигурации.

**S1:**
enable
conf t
hos S1
int vla 99
ip addr 192.168.99.11 255.255.255.0
no shut
exit
no ip domain-l
service password-encryption
Banner motd "This is a secure system. Authorized Access Only!"
enable secret class
line vty 0 4
logging synchronous
password cisco
login
exit

int ran e0/1-3
shut
int ran e1/0-3
shut

vlan 99
name Management
vlan 10
name Staff

do copy run start
[Enter]

