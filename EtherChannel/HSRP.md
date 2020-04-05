

# Лабораторная работа. Настройка HSRP

## Топология

![img](img/HSRP_Scheme.png)

## Таблица адресации

| Устройство | Интерфейс     | IP-адрес        | Маска подсети   | Шлюз по умолчанию |
| ---------- | ------------- | --------------- | --------------- | ----------------- |
| R1         | G0/1          | 192.168.1.1     | 255.255.255.0   | —                 |
|            | S0/0/0  (DCE) | 10.1.1.1        | 255.255.255.252 | —                 |
| R2         | S0/0/0        | 10.1.1.2        | 255.255.255.252 | —                 |
|            | S0/0/1  (DCE) | 10.2.2.2        | 255.255.255.252 | —                 |
|            | Lo1           | 209.165.200.225 | 255.255.255.224 | —                 |
| R3         | G0/1          | 192.168.1.3     | 255.255.255.0   | —                 |
|            | S0/0/1        | 10.2.2.1        | 255.255.255.252 | —                 |
| S1         | VLAN 1        | 192.168.1.11    | 255.255.255.0   | 192.168.1.1       |
| S3         | VLAN 1        | 192.168.1.13    | 255.255.255.0   | 192.168.1.3       |
| PC-A       | NIC           | 192.168.1.31    | 255.255.255.0   | 192.168.1.1       |
| PC-C       | NIC           | 192.168.1.33    | 255.255.255.0   | 192.168.1.3       |

## Задачи

**Часть 1. Построение сети и проверка соединения**

**Часть 2. Настройка обеспечения избыточности на первом хопе с помощью HSRP**\



## Построение сети и проверка соединения

Мы уже соединили все узлы согласно нашей лабораторной работе , а теперь займемся настройкой Роутеров и Коммутаторов. Так как настройка устройств типична для каждого вида ,то распишу лишь настройку **R1** и **S1**.

**Router:**

Enable
Configure terminal

no ip domain-lookup 
hos R1
interface Serial5/0
ip address 10.1.1.1 255.255.255.252
clock rate 128000
interface GigabitEthernet6/0
ip address 192.168.1.1 255.255.255.0

enable secret class 
line vty 0 4 
logging synchronous 
password cisco 
login 
end
copy running-config startup-config
[Enter]

**Switch:**

Enable
Configure terminal

no ip domain-lookup 
hostname S1
enable secret class 

interface Vlan1
ip address 192.168.1.11 255.255.255.0

ip default-gateway 192.168.1.1

line vty 0 4 
logging synchronous 
password cisco 
login
end
copy running-config startup-config
[Enter]

Проверка подключения :

1. ПК А и ПК С ping 192.168.1.33

   Pinging 192.168.1.33 with 32 bytes of data:

   Reply from 192.168.1.33: bytes=32 time=1ms TTL=128
   Reply from 192.168.1.33: bytes=32 time=1ms TTL=128
   Reply from 192.168.1.33: bytes=32 time<1ms TTL=128
   Reply from 192.168.1.33: bytes=32 time<1ms TTL=128

2. Запустите сеанс эхо-тестирования на PC-A и разорвите соединение между S1 и R1

