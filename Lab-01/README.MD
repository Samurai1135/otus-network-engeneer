# VLAN

## Цель:
Настройка DTP.
Добавление сетей VLAN и назначение портов.

> В этой лабораторной работе настраиваются магистральные каналы между коммутаторами.


- #### Схема сети, построенная на эмуляторе EVE-NG
![](https://github.com/Samurai1135/otus-network-engeneer/blob/73da4c02996a5d358a01fa937864a16ec8272ce5/Lab-01/NetworkScheme/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%B2%20EVE-ng.png)



- #### Используемое оборудование:

| Наименование       | Обозначение на схеме |  Порт подключения и    IP адресация |
| :----------------- | :------------------- | :---------------------------------- |
|Маршрутизатор/роутер| Router1              |  eth0/1.20 -- IP:192.168.20.254/24  |  
|                    |                      |  eth0/1.30 --  IP:192.168.30.254/24 |
|Коммутатор/Свитч 1. | Switch1              |  eth0/0 - trunk, eth0/1,2 - access  |
|Коммутатор/Свитч 2. | Switch2              |  eth0/0,2 - trunk, eth0/1,3 - access|
|Пользовательский ПК2| VPC2                 |  eth0 -- IP:192.168.20.2/24         |
|Пользовательский ПК5| VPC5                 |  eth0 -- IP:192.168.30.2/24         |
|Пользовательский ПК6| VPC6                 |  eth0 -- IP:192.168.20.3/24         | 
|Пользовательский ПК7| VPC7                 |  eth0 -- IP:192.168.30.3/24         |



- #### Пример конфигурации маршрутизатора/роутера [Router1](https://github.com/Samurai1135/otus-network-engeneer/blob/5d99e9ae3b454713c38ee46384ad0a144e1dccd8/Lab-01/Configs/R1):

~~~
conf t
!
hostname Router1
!
interface Ethernet0/1
 no shutdown
 no ip address
!
interface Ethernet0/1.20
 no shutdown
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
!
interface Ethernet0/1.30
 no shutdown
 encapsulation dot1Q 30
 ip address 192.168.30.254 255.255.255.0
!
~~~
- #### Пример конфигурации коммутатора/свича [Switch2](https://github.com/Samurai1135/otus-network-engeneer/blob/0e32d26be229c73e5e031ce8d3db0a6f552e44bc/Lab-01/Configs/SW2):
~~~

conf t
!
hostname Switch2
!
interface Ethernet0/0
 no shutdown
 switchport trunk allowed vlan 20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 no shutdown
 switchport access vlan 30
 switchport mode access
!
interface Ethernet0/2
 no shutdown
 switchport trunk allowed vlan 20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/3
 no shutdown
 switchport access vlan 20
 switchport mode access
!
~~~
