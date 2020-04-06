

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

**Часть 2. Настройка обеспечения избыточности на первом хопе с помощью HSRP** /



## Построение сети и проверка соединения

Мы уже соединили все узлы согласно нашей лабораторной работе , а теперь займемся настройкой Роутеров и Коммутаторов. Так как настройка устройств типична для каждого вида ,то распишу лишь настройку **R1** и **S1**.

**Router:**

Enable<br/>
Configure terminal<br/>

no ip domain-lookup <br/>
hos R1<br/>
interface Serial5/0<br/>
ip address 10.1.1.1 255.255.255.252<br/>
clock rate 128000<br/>
interface GigabitEthernet6/0<br/>
ip address 192.168.1.1 255.255.255.0<br/>

enable secret class <br/>
line vty 0 4 <br/>
logging synchronous <br/>
password cisco <br/>
login <br/>
end<br/>
copy running-config startup-config<br/>
[Enter]<br/>

**Switch:**<br/>

Enable<br/>
Configure terminal<br/>

no ip domain-lookup <br/>
hostname S1<br/>
enable secret class <br/>

interface Vlan1<br/>
ip address 192.168.1.11 255.255.255.0<br/>

ip default-gateway 192.168.1.1<br/>

line vty 0 4 <br/>
logging synchronous <br/>
password cisco <br/>
login<br/>
end<br/>
copy running-config startup-config<br/>
[Enter]<br/>

Проверка подключения :<br/>

1. ПК А и ПК С ping 192.168.1.33<br/>

   Pinging 192.168.1.33 with 32 bytes of data:<br/>

   Reply from 192.168.1.33: bytes=32 time=1ms TTL=128<br/>
   Reply from 192.168.1.33: bytes=32 time=1ms TTL=128<br/>
   Reply from 192.168.1.33: bytes=32 time<1ms TTL=128<br/>
   Reply from 192.168.1.33: bytes=32 time<1ms TTL=128<br/>

2. Запустите сеанс эхо-тестирования на PC-A и разорвите соединение между S1 и R1<br/>

C: />ping 209.165.200.225<br/>

Pinging 209.165.200.225 with 32 bytes of data:<br/>

Request timed out.<br/>
Request timed out.<br/>
Request timed out.<br/>
Request timed out.<br/>

Ping statistics for 209.165.200.225:<br/>

​    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)<br/>

Эхо трафик перестал ходит по причине обрыва шлюза по умолчанию 192.168.1.1 ,  ранее пинги проходили .

С ПК-С проходят ping , так как доступен шлюз по умолчанию<br/>

C: />ping 209.165.200.225<br/>

Pinging 209.165.200.225 with 32 bytes of data:<br/>

Reply from 209.165.200.225: bytes=32 time=2ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>

Ping statistics for 209.165.200.225:<br/>
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),<br/>
Approximate round trip times in milli-seconds:<br/>
    Minimum = 1ms, Maximum = 2ms, Average = 1ms<br/>

### Настройте HSRP на R1 и R3.

В этом шаге вам предстоит настроить HSRP и изменить адрес шлюза по умолчанию на компьютерах PC-A, PC-C, S1 и коммутаторе S2 на виртуальный IP-адрес для HSRP. R1 назначается активным маршрутизатором с помощью команды приоритета HSRP.

 Настройте протокол HSRP на маршрутизаторе R1.

R1(config)# **interface g0/1**<br/>
R1(config-if)# **standby version 2**<br/>
R1(config-if)# **standby 1 ip 192.168.1.254**<br/>
R1(config-if)# **standby 1 priority 150**<br/>
R1(config-if)# **standby 1 preempt**<br/>

Настройте протокол HSRP на маршрутизаторе R3.

R3(config)# **interface g0/1**<br/>
R3(config-if)# **standby version 2**<br/>
R3(config-if)# **standby 1 ip 192.168.1.254**<br/>

Проверьте HSRP, выполнив команду **show standby** на R1 и R3.

![image-20200407010715086]( img/sh_st_R1.png)

![image-20200407010752893]( img/sh_st_R3.png)

Какой маршрутизатор является активным? 

R1 является активным роутером

Какой MAC-адрес используется для виртуального IP-адреса?

MAC-адрес **0000.0C9F.F001** используется для виртуального IP-адреса

Какой IP-адрес и приоритет используются для резервного маршрутизатора?

Приоритет 100

Используйте команду **show standby brief** на R1 и R3, чтобы просмотреть сводку состояния HSRP. 

![image-20200407021028047](img/image-20200407021028047.png)

![image-20200407020753734](img/image-20200407020258461.png)

 Измените адрес шлюза по умолчанию для PC-A, PC-C, S1 и S3. Какой адрес следует использовать?

192.168.1.254

Проверьте новые настройки. Отправьте эхо-запрос с PC-A и с PC-C на loopback-адрес маршрутизатора R2. Успешно ли выполнены эхо-запросы? 

![image-20200407011608516](img/pingPC_AC.png)

###  Запустите сеанс эхо-тестирования на PC-A и разорвите соединение с коммутатором, подключенным к активному маршрутизатору HSRP (R1).

C: />ping -t 209.165.200.225

Pinging 209.165.200.225 with 32 bytes of data:

Reply from 209.165.200.225: bytes=32 time=8ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=8ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>
Request timed out.<br/>
Request timed out.<br/>
Reply from 209.165.200.225: bytes=32 time=8ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=8ms TTL=254<br/>
Reply from 209.165.200.225: bytes=32 time=1ms TTL=254<br/>

Ping statistics for 209.165.200.225:<br/>
       Packets: Sent = 10, Received = 8, Lost = 2 (20% loss),<br/>
Approximate round trip times in milli-seconds:<br/>
      Minimum = 1ms, Maximum = 8ms, Average = 1ms<br/>

Как видим, 2 потерянных эхо запроса ушло на восстановление 

### Проверьте настройки HSRP на маршрутизаторах R1 и R3.

a.   Выполните команду **show standby brief** на маршрутизаторах R1 и R3.

Какой маршрутизатор является активным?    R1 

Повторно подключите кабель, соединяющий коммутатор и маршрутизатор, или включите интерфейс F0/5. Какой маршрутизатор теперь является активным? Поясните ответ.   

R1  так как приоритет выше

###  Изменение приоритетов HSRP.

a.   Измените приоритет HSRP на 200 на маршрутизаторе R3. Какой маршрутизатор является активным?     **R1**

b.  Выполните команду, чтобы сделать активным маршрутизатор R3 без изменения приоритета. Какую команду вы использовали?

R3(config-if)#standby 1 preempt

c.   Используйте команду **show**, чтобы убедиться, что R3 является активным маршрутизатором.

![image-20200407015702388](img\sh_st_br_R3.png)

Вопросы для повторения

Для чего в локальной сети может потребоваться избыточность?

Обеспечение безотказного доступа к ресурсам