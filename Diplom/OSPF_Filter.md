# Различные виды фильтрации в протоколе OSPF

## Домашнее задание

### OSPF

Цель: Настроить OSPF офисе Москва Разделить сеть на зоны Настроить фильтрацию между зонами

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 1015
5. План работы и изменения зафиксированы в документации

![OSPF](img/OSPF.png)

R14

```
enable
configure terminal
hostname R14
line con 0
exec-t 0 0
exit
router ospf 1
router-id 14.14.14.14
network 10.10.10.16 0.0.0.3 area 0
network 10.10.10.20 0.0.0.3 area 0
network 10.10.10.12 0.0.0.3 area 101
default-information originate
area 101 stub no-summary
```

R15

```
enable
configure terminal
hostname R15
line con 0
exec-t 0 0
exit
router ospf 1
router-id 15.15.15.15
network 10.10.10.8 0.0.0.3 area 0
network 10.10.10.4 0.0.0.3 area 0
network 10.10.10.0 0.0.0.3 area 102
default-information originate
area 102 range 10.10.10.0 255.255.255.252 not-advertise
```

R13

```
enable
configure terminal
hostname R13
line con 0
exec-t 0 0
exit
router ospf 1
 router-id 13.13.13.13
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.20 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.255 area 10
```

R12

```
enable
configure terminal
hostname R12
line con 0
exec-t 0 0
exit
router ospf 1
router-id 12.12.12.12
network 10.10.10.8 0.0.0.3 area 0
network 10.10.10.16 0.0.0.3 area 0
network 172.16.0.0 0.0.0.255 area 10
```

 Остается вопрос по поводу  

```
area 102 range 10.10.10.0 255.255.255.252 not-advertise
```

Правильно ли я ограничил передачу маршрутов за area 102 ?



В 4 вопросе опечатка **1015** ... такого area не было в задании.