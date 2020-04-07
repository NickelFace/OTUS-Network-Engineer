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

Enable
Configure terminal
hostname R1
Interface loopback 0
ip address 209.165.200.225 255.255.255.252
Interface loopback 1
ip address 192.168.1.1 255.255.255.0
Interface loopback 2
ip address 192.168.2.1 255.255.255.0
exit
interface serial 0/0
ip address 192.168.12.1 255.255.255.252
clock rate 128000
no shutdown
exit
no ip domain-lookup
enable secret class
line vty 0 15
logging synchronous
password cisco
login
exit
line con 0
logging synchronous
password cisco
login
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]

**R2**

Enable
Configure terminal
hostname R2
Interface loopback 6
ip address 192.168.6.1 255.255.255.0
exit
interface serial 0/0
ip address 192.168.12.2 255.255.255.252
no shutdown
exit
interface serial 1/0
ip address 192.168.23.1 255.255.255.252
clock rate 128000
no shutdown
exit
no ip domain-lookup
enable secret class
line vty 0 4
logging synchronous
password cisco
login
exit
line con 0
logging synchronous
password cisco
login
exit
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]

**R3**

enable
configure terminal
hostname R3
interface serial 0/0
ip address 192.168.23.2 255.255.255.252
no shutdown
exit
interface loopback 4 
ip address 192.168.4.1 255.255.255.0
exit
interface loopback 5
ip address 192.168.5.1 255.255.255.0
exit
no ip domain-lookup
enable secret class
line vty 0 4
logging synchronous
password cisco
login
exit
line con 0
logging synchronous
password cisco
login
exit
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]

## Проверьте наличие подключения на уровне 3.

Выполните команду **show ip interface brief**, чтобы убедиться в правильности IP-адресации и активности интерфейсов. Убедитесь, что каждый маршрутизатор может успешно отправлять эхо-запросы соседним маршрутизаторам, подключенным с помощью последовательных интерфейсов.

![21](img\mult\21.png)

![22](img\mult\22.png)

![23](img\mult\23.png)

## Настройка сети OSPFv2 для нескольких областей

В части 2 необходимо настроить сеть OSPFv2 для нескольких областей, используя идентификатор процесса 1. Все интерфейсы loopback локальной сети должны быть пассивными, а для всех последовательных интерфейсов должна быть настроена аутентификация MD5 с ключом **Cisco123**