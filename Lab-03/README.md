# Настройка DHCP-сервера на маршрутизаторе CISCO

`Цель:`
Настроить DHCPv4
Настроить DHCPv6

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой лабораторной работе настраивается автоматическое получение IP адресов в сети.

- `Часть 1.` Настройка топологии IPv4 сети и основных параметров коммутаторов и маршрутизаторов
- `Часть 2.` Настройка DHCPv4-сервера на роутере R1  
- `Часть 3.` Настройка DHCP-реле на роутере R2
- `Часть 4.` Настройка топологии IPv6 сети и основных параметров коммутаторов и маршрутизаторов
- `Часть 5.` Настройка DHCPv6-сервера на роутере R1. Натройка пулов адресов.  
- `Часть 6.` Настройка маршрута между роутерами R1 и R2

<br><br>
- ## Часть 1. Топология сети и основные параметры сетевых устройств

![](https://github.com/Samurai1135/otus-network-engeneer/blob/b1eceb41981722dd08eed58294725ac951e37342/Lab-03/NetworkScheme/IPv4.png)
<br>

#### Используемое оборудование и конфигурация портов:
<br>

| Наименование       | Обозначение на схеме |  Порт подключения и    IP адресация |
| :----------------- | :------------------- | :---------------------------------- |
|Маршрутизатор/роутер| R1              |  eth0/0 -- IP:10.0.0.1/30  |  
|                    |                      |  eth0/1 --  IP:192.168.1.254/24 |
|Маршрутизатор/роутер| R2              |  eth0/0 -- IP:10.0.0.2/30  |  
|                    |                      |  eth0/1 --  IP:192.168.220.2/24 |
|Коммутатор/Свитч 1. | S1              |  eth0/1 - trunk, eth0/0 - access  |
|Коммутатор/Свитч 2. | S2              |  eth0/1 - trunk, eth0/0 - access|
|Пользовательский ПК5| PC5                 |  eth0 -- IP:dhcp         |
|Пользовательский ПК6| PC6                 |  eth0 -- IP:dhcp        |

<br>

#### Параметры сетевых устройств 
> Пример конфигурации маршрутизатора [R1](https://github.com/Samurai1135/otus-network-engeneer/blob/e53521d24920c8734b7d5570d00f27144669b4b7/Lab-03/Configs/IPv4/R1):
~~~

!
! Last configuration change at 07:37:33 UTC Wed Apr 19 2023
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$dM9x$134d3t3KbFSnNZLWXRLVh1
enable password cisco
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
ip dhcp excluded-address 192.168.1.254
!
ip dhcp pool Pool-1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.254 
!
ip dhcp pool Pool-220
 network 192.168.220.0 255.255.255.0
 default-router 192.168.220.2 
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
redundancy
!
interface Ethernet0/0
 no shutdown
 ip address 10.0.0.1 255.255.255.252
!
interface Ethernet0/1
 no shutdown
 ip address 192.168.1.254 255.255.255.0
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
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.0.0.2
!
control-plane
!
banner motd Secure Zone! Password protection!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 password cisco
 logging synchronous
 login
 transport input none
!
!
end

~~~

> Пример конфигурации коммутатора [S1](https://github.com/Samurai1135/otus-network-engeneer/blob/e53521d24920c8734b7d5570d00f27144669b4b7/Lab-03/Configs/IPv4/S1):

~~~

!
! Last configuration change at 07:43:28 UTC Wed Apr 19 2023
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname S1
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$v2F.$wEdUNnUSpT/8C/eDOhLyp/
enable password cisco
!
no aaa new-model
!
no ip domain-lookup
ip cef
no ipv6 cef
!
!
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
interface Ethernet0/0
 no shutdown
 switchport mode access
!
interface Ethernet0/1
 no shutdown
 switchport trunk allowed vlan 1,200
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 no shutdown
 switchport access vlan 200
 switchport mode access
!
interface Ethernet0/3
 no shutdown
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
control-plane
!
banner motd Secure Zone! Password protection!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 password cisco
 logging synchronous
 login
!
!
end
~~~

- ## Часть 2. Настройка DHCPv4-сервера на роутере R1

~~~

ip dhcp excluded-address 192.168.1.254 
ip dhcp pool Pool-1 
 network 192.168.1.0 255.255.255.0 
 default-router 192.168.1.254 
ip dhcp pool Pool-220
 network 192.168.220.0 255.255.255.0
 default-router 192.168.220.2 
~~~
`ip dhcp excluded-address {address}` //Исключаем адрес из списка DHCP <br>
`ip dhcp pool {Pool-Name}`           //Выбираем название пула адресов <br>
`network`                            //Указываем подсеть, из которой будет раздача адресов IPv4 <br>
`default-router`                     //Шлюз по-умолчанию (интерфейс, смотрящий в подсеть) <br>

Для связи подсетей 192.168.1.0/24 и 192.168.220.0/24 прокинем маршрут на роутере R1 (аналогичную процедуру необходимо выполнить на роутере R2)
~~~
ip route 0.0.0.0 0.0.0.0 10.0.0.2
~~~

- ## Часть 3. Настройка DHCP-реле на роутере R2

Ниже приведены результаты команды "show spanning-tree active" на каждом из коммутаторов:

### SW1
~~~
S1#sh spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et1/2               Desg LRN 100       128.7    Shr
Et1/3               Desg LRN 100       128.8    Shr


 --More--
*Apr 17 07:05:14.044: %LINK-3-UPDOWN: Interface Vlan20, changed state to up
*Apr 17 07:05:15.051: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan20, changed state to up

VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
~~~

### SW2
~~~
S2#sh spanning-tree

VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/1               Desg FWD 100       128.6    Shr



VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    32798
             Address     aabb.cc00.2000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
~~~
### SW3
~~~
S3#sh spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.3000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Et1/2               Desg FWD 100       128.7    Shr



VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr



VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    32798
             Address     aabb.cc00.2000
             Cost        200
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
~~~

### SW4
~~~
S4#sh sp

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.4000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et1/2               Desg FWD 100       128.7    Shr
Et1/3               Desg FWD 100       128.8    Shr



VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             Cost        200
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr



VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    32798
             Address     aabb.cc00.2000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     aabb.cc00.4000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/1               Desg FWD 100       128.6    Shr
~~~

### SW5

~~~
S5#sh sp

VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
Et1/1               Desg FWD 100       128.6    Shr



VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    32798
             Address     aabb.cc00.2000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
~~~
Как видно из результатов команды "show spanning-tree active" наименьший BridgeID Address (aabb:cc00:1000) имеет коммутатор Switch1(SW1), что и делает его корневым мостом.
Стоимость пути Root Path Cost на портах коммутаторов равна 100, что говорит о прямом подключении к RouteBridge. Исключение составляет коммутатор Switch4(SW4), чья стоимость пути Root Path Cost = 200, т.к. на маршрутах между ним и RootBridge есть еще по одному сетевому устройству - коммутатор Switch2(SW2) (по пути слева на схеме) и Switch3(SW3) (по пути справа на схеме). Все порты корневого коммутатора находятся в режиме Designated. 

#### Логика выбора Alternate-портов (неактивных портов):
На транковом соединении между SW4(e0/0) и SW5(e0/1) порт SW5(e0/1) является Designated-портом, т.к. имеет меньший RootPathCost стоимость пути, а порт SW4(e0/0) становится Alternate-портом. Но транк SW5-SW1 - прямое подключение SW5 к Root Bridge и стоимость пути RootPathCost = 100.

В случае порта SW4(e0/1) - порт переходит в Alternate, т.к. маршрут через SW2 является более приоритетным, чем через SW3, т.к. BridgeID Address SW2 меньше, чем у SW3, хотя стоимость пути у магистральных портов одинаковая (Cost = 200)

`Следует принять во внимание факт: при одинаковых RootPathCost Designated портом будет тот, чей коммутатор имеет меньший BridgeID.
А в случае совпадения BID выбор падает на порт с меньшим PortID.`

Аналогичная ситуация с Alternate-портом SW5(e0/0)

- ## Часть 4. Активация избыточных путей до каждого из коммутаторов

  Добавим транковое соединение SW5 с SW1 через интерфейсы eth1/1 обоих коммутаторов
  ![](https://github.com/Samurai1135/otus-network-engeneer/blob/5ab7618a4014591d182a436f780733e5d6b7e719/Lab-02/NetworkScheme/STP-SW5-SW1.png)
 ~~~
  S5#sh sp
  VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Desg LRN 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Desg LRN 100       128.4    Shr
Et1/1               Altn BLK 100       128.6    Shr



VLAN0030
  Spanning tree enabled protocol rstp
  Root ID    Priority    32798
             Address     aabb.cc00.2000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     aabb.cc00.5000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg LRN 100       128.3    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg LRN 100       128.6    Shr
~~~
<br>
Как видно из результатов команды "show spanning-tree active" на коммутаторе Switch5(SW5) интерфейс eth1/1 перешел в режим Alternate-port, т.к. PortID отправителя SW1(e0/2) меньше, чем у SW1(eth1/1), хотя оба транка связывают коммутатор с RootBridge и имеют одинаковую стоимость пути до корневого коммутатора RootPathCost.
В случае отключения транка на портах e0/2 коммутаторов SW5 и SW1 порт eth1/1 коммутатора SW5 станет Root-портом
