# VLAN

## Цель:
Настройка DTP.
Добавление сетей VLAN и назначение портов.

> В этой лабораторной работе настраиваются магистральные каналы между коммутаторами.


#### Схема сети, построенная на эмуляторе EVE-NG
![](https://github.com/Samurai1135/otus-network-engeneer/blob/73da4c02996a5d358a01fa937864a16ec8272ce5/Lab-01/NetworkScheme/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%B2%20EVE-ng.png)



#### Используемое оборудование:

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



### Пример конфигурации маршрутизатора/роутера [Router1](https://github.com/Samurai1135/otus-network-engeneer/blob/5d99e9ae3b454713c38ee46384ad0a144e1dccd8/Lab-01/Configs/R1)

~~~
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Router
!
boot-start-marker
boot-end-marker
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
redundancy
!
interface Ethernet0/0
 no shutdown
 no ip address
 shutdown
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
interface Ethernet0/2
 no shutdown
 no ip address
 shutdown
!
interface Ethernet0/3
 no shutdown
 no ip address
 shutdown
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
control-plane
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
end
~~~
