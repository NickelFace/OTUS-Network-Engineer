[🇬🇧 English version](README.md)

# OTUS Сетевой Инженер

> Лабораторные работы и дипломный проект курса OTUS «Сетевой Инженер» — коммутация, протоколы маршрутизации, VPN-туннели и проектирование корпоративной сети на Cisco IOS / EVE-NG.

## Лабораторные работы

| # | Тема | Описание |
|---|------|----------|
| 01 | [VLAN](VLAN/) | Настройка расширенных сетей VLAN, VTP и DTP. Trunk-каналы, динамические соединения. |
| 02 | [STP](STP/) | Коммутируемая сеть с резервными каналами. Выбор Root-моста, настройка стоимости и приоритета портов. |
| 03 | [EtherChannel](EtherChannel/) | EtherChannel на LACP и PAgP. Port-channel, VLAN trunking через агрегированные каналы. |
| 03 | [HSRP](EtherChannel/HSRP.ru.md) | Резервирование шлюза с HSRP. Active/standby маршрутизаторы, виртуальный IP, приоритет и preemption. |
| 04 | [OSPF. Базовый](OSPF/) | OSPFv2 в единой зоне. Router-id, пассивные интерфейсы, соседи, таблица маршрутизации, метрика. |
| 05 | [OSPF. Несколько зон](OSPF/Multiarea.ru.md) | OSPFv2 multi-area. ABR, ASBR, суммаризация между зонами, MD5-аутентификация. |
| 06 | [EIGRP. Базовый](EIGRP/) | EIGRP для IPv4 — соседи, таблица топологии, настройка полосы пропускания, пассивные интерфейсы. |
| 06 | [EIGRP. Расширенный](EIGRP/Advanced.ru.md) | Расширенная настройка EIGRP — auto-summary, редистрибуция статического default route, таймеры. |
| 07 | [Адресация IPv4/IPv6](Diplom/IPv4-IPv6.ru.md) | Дипломный проект: проектирование адресного пространства IPv4/IPv6, VLAN, management-сети. |
| 08 | [PBR](Diplom/PBR.ru.md) | Policy-Based Routing для офиса Чокурдах — распределение трафика между двумя uplink с IP SLA. |
| 09 | [Фильтрация OSPF](Diplom/OSPF-Filter.ru.md) | OSPF с разбивкой на зоны и фильтрацией маршрутов между зонами в офисе Москва. |
| 10 | [OSPFv3](Diplom/OSPFv3.ru.md) | OSPFv3 для IPv6 — метрики, таймеры и фильтры по аналогии с OSPFv2. |
| 11 | [BGP. Основы](Diplom/BGP.ru.md) | eBGP между автономными системами для связи между офисами Москва и Санкт-Петербург. |
| 12 | [iBGP](Diplom/IBGP.ru.md) | iBGP в офисе Москва и провайдере Триада. Выбор предпочтительного провайдера, балансировка трафика. |
| 13 | [Фильтрация BGP](Diplom/BGP-Path.ru.md) | Фильтрация маршрутов BGP для офисов — ограничение анонсов провайдера до default route. |
| 14 | [PAT, DHCP, NTP](Diplom/PAT-DHCP-NTP.ru.md) | NAT/PAT, DHCP-сервер и синхронизация времени NTP для всех офисов. |
| 15 | [DMVPN](Diplom/DMVPN.ru.md) | GRE-туннель Москва — Санкт-Петербург. DMVPN hub-and-spoke между Москвой, Чокурдахом и Лабытнанги. |
| 16 | [IPSec over DMVPN](Diplom/DMVPN-IPSec.ru.md) | GRE over IPSec и DMVPN over IPSec — шифрование туннелей. |
| 17 | [Проект](PROJECT/) | Корпоративная сеть: трёхуровневая иерархическая модель Cisco с OSPF, HSRP, STP, DHCP, PAT, L2 Security. |
| — | [Самообучение](SelfLearn/) | Дополнительные материалы: BGP, GRE-туннели, IS-IS, подключение к провайдеру. |

## Среда

- Симулятор: [EVE-NG](https://www.eve-ng.net/)
- Платформа: Cisco IOS
- Курс: [OTUS Сетевой Инженер](https://otus.ru/lessons/network-engineer/)
