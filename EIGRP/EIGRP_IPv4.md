# Лабораторная работа. Базовая настройка протокола EIGRP для IPv4

## Топология

![image](img/1.png)

## Таблица адресации

| Устройство | Интерфейс    | IP-адрес    | Маска подсети   | Шлюз по умолчанию |
| ---------- | ------------ | ----------- | --------------- | ----------------- |
| R1         | G0/0         | 192.168.1.1 | 255.255.255.0   | —                 |
|            | S0/0/0 (DCE) | 10.1.1.1    | 255.255.255.252 | —                 |
|            | S0/0/1       | 10.3.3.1    | 255.255.255.252 | —                 |
| R2         | G0/0         | 192.168.2.1 | 255.255.255.0   | —                 |
|            | S0/0/0       | 10.1.1.2    | 255.255.255.252 | —                 |
|            | S0/0/1 (DCE) | 10.2.2.2    | 255.255.255.252 | —                 |
| R3         | G0/0         | 192.168.3.1 | 255.255.255.0   | —                 |
|            | S0/0/0 (DCE) | 10.3.3.2    | 255.255.255.252 | —                 |
|            | S0/0/1       | 10.2.2.1    | 255.255.255.252 | —                 |
| PC-A       | NIC          | 192.168.1.3 | 255.255.255.0   | 192.168.1.1       |
| PC-B       | NIC          | 192.168.2.3 | 255.255.255.0   | 192.168.2.1       |
| PC-C       | NIC          | 192.168.3.3 | 255.255.255.0   | 192.168.3.1       |

## Задачи

**Часть 1. Построение сети и проверка соединения**

**Часть 2. Настройка маршрутизации EIGRP**

**Часть 3. Проверка маршрутизации EIGRP**

**Часть 4. Настройка пропускной способности и пассивных интерфейсов**



##  Построение сети и проверка связи

Шаг 1:   Создайте сеть согласно топологии.

R1

```
enable
configure terminal
hostname R1
interface Serial0/0
ip address 10.1.1.1 255.255.255.252
interface Serial1/0
ip address 10.3.3.1 255.255.255.252
interface GigabitEthernet2/0
ip address 192.168.1.1 255.255.255.0
line console 0
exec-timeout 0 0
exit
no ip domain-lookup
enable secret class
line vty 0 15
logging synchronous
password cisco
login
exit
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]
```

R2

```
enable
configure terminal
hostname R2
interface Serial0/0
ip address 10.1.1.2 255.255.255.252
interface Serial1/0
ip address 10.2.2.2 255.255.255.252
interface GigabitEthernet2/0
ip address 192.168.2.1 255.255.255.0
line console 0
exec-timeout 0 0
exit
no ip domain-lookup
enable secret class
line vty 0 15
logging synchronous
password cisco
login
exit
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]
```

R3

```
enable
configure terminal
hostname R2
interface Serial0/0
ip address 10.3.3.2 255.255.255.252
interface Serial1/0
ip address 10.2.2.1 255.255.255.252
interface GigabitEthernet2/0
ip address 192.168.3.1 255.255.255.0
line console 0
exec-timeout 0 0
exit
no ip domain-lookup
enable secret class
line vty 0 15
logging synchronous
password cisco
login
exit
Banner motd "This is a secure system. Authorized Access Only!"
do copy run start
[Enter]
```

