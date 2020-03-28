## Лабораторная работа. Настройка EtherChannel

### Топология

![img](img/1.png)

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

Создание сети из примера лабораторной работы , а также настройка базовых параметров на коммутаторах: Имя хоста, создание inter vlan 99 , назначение ip-address ,отключение поиск DNS, шифрование незашифрованных паролей, Создание баннерного сообщения дня MOTD, назначение пароля class на привилегированный режим , пароль на VTY ,  настроим предотвращение прерывание команд , сохранение конфигурации.

**S1:**
enable<br/>
conf t<br/>
hos S1<br/>
int vla 99<br/>
S1: ip addr 192.168.99.11 255.255.255.0<br/>
S2: ip addr 192.168.99.12 255.255.255.0 <br/>
S3: ip addr 192.168.99.13 255.255.255.0 <br/>
no shut<br/>
exit<br/>
no ip domain-loo<br/>
service password-encryption<br/>
Banner motd "This is a secure system. Authorized Access Only!"<br/>
enable secret class<br/>
line vty 0 4<br/>
logging synchronous<br/>
password cisco<br/>
login<br/>
exit<br/>

Конечно на других устройствах будет другой номер имени Устройства и IP-адрес.
Далее отключение портов на коммутаторах:

int ran e0/1-3<br/>
shut<br/>
int ran e1/0-3<br/>
shut<br/>

 создание VLAN 99 , 10 именем **Management** и **Staff** соответственно

vlan 99<br/>
name Management<br/>
vlan 10<br/>
name Staff<br/>

Присвоим на портах коммутатора в сторону ПК режим доступа: 

int e0/0<br/>
sw m ac<br/>
sw ac v 10<br/>

Сохраним конфигурацию:

do copy run start<br/>
[Enter]<br/>

Пропишем на ПК

**A:** 
ip 192.168.10.1/24<br/>
save<br/>
**B:** 
ip 192.168.10.2/24<br/>
save<br/>
**C:** 
ip 192.168.10.3/24<br/>
save<br/>

На всех остальных коммутаторах в сети конфигурация повторяется,поэтому не стал дублировать текст для каждого устройства.

### Настройка протокола PAgP

Настроим для S1 и S3 PAgP :

S1(config): 
int ran e1/2-3<br/>
channel-group 1 mode desirable<br/>
no shut<br/>

S3(config): 

int ran e1/2-3<br/>
channel-group 1 mode auto<br/>
no shut<br/>

**do show run interface e1/2**

![img](img/2.png)



**S1# show interfaces e1/2 switchport**

![img](img/3.png)

S3# **show etherchannel summary**

![image-20200328134313895](C:\Users\Admin\Documents\GitHub\OTUS_Network\EtherChannel\img/4.png)

