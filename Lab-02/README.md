
# Настройка STP

`Цель:`
Создание сети и настройка основных параметров устройства.
Выбор корневого моста.
Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.
Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

## Описание/Пошаговая инструкция выполнения домашнего задания:

- `Часть 1.` Настройка топологии сети и основных параметров коммутаторов и маршрутизаторов.
- `Часть 2.` Для каждого экземпляра протокола spanning-tree (коммутируемая сеть LAN или широковещательный домен) определить коммутатор, выделенный в качестве корневого моста (Root Bridge).
- `Часть 3.` Отразить результаты наблюдения за процессом выбора протоколом STP порта, исходя из стоимости портов.
- `Часть 4.` Активация избыточных путей до каждого из коммутаторов, чтобы просмотреть, каким образом протокол STP выбирает порт с учетом приоритета портов.

<br><br>
- ## Часть 1. Топология сети и основные параметры сетевых устройств

![](https://github.com/Samurai1135/otus-network-engeneer/blob/8b6e6785283c7d86172ae0cff723feca60682384/Lab-02/NetworkScheme/NetworkScheme.png)
<br>

#### Используемое оборудование и конфигурация портов:
<br>

| Наименование       | Обозначение на схеме |  Порт подключения и    IP адресация |
| :----------------- | :------------------- | :---------------------------------- |
|Маршрутизатор/роутер| Router1              |  eth0/0.20 -- IP:10.128.20.254/24  |  
|                    |                      |  eth0/0.30 --  IP:10.128.30.254/24 |
|Коммутатор/Свитч 1. | Switch1              |  eth0/0-2 - trunk, eth1/1 - trunk  |
|Коммутатор/Свитч 2. | Switch2              |  eth0/0-1 - trunk, eth1/0 - access|
|Коммутатор/Свитч 3. | Switch3              |  eth0/0,1,3 - trunk, eth0/2 - access  |
|Коммутатор/Свитч 4. | Switch4              |  eth0/0,1 - trunk, eth0/3 и eth1/0 - access|
|Коммутатор/Свитч 5. | Switch5              |  eth0/0-2 и eht1/1 - trunk, eth0/3 и eth1/0 - access  |
|Пользовательский ПК4| VPC4                 |  eth0 -- IP:10.128.30.5/24         |
|Пользовательский ПК6| PC6                 |  eth0 -- IP:10.128.30.6/24         |
|Пользовательский ПК7| PC7                 |  eth0 -- IP:10.128.20.4/24         | 
|Пользовательский ПК8| PC8                 |  eth0 -- IP:10.128.20.3/24         |
|Пользовательский ПК9| PC9                 |  eth0 -- IP:10.128.30.2/24         | 
|Пользовательский ПК10| PC10                 |  eth0 -- IP:10.128.30.3/24         |
|Пользовательский ПК13| PC10                 |  eth0 -- IP:10.128.20.7/24         |
|Пользовательский ПК14| PC10                 |  eth0 -- IP:10.128.20.8/24         |

<br>

#### Параметры сетевых устройств 
> Пример конфигурации маршрутизатора [Router-1](https://github.com/Samurai1135/otus-network-engeneer/blob/4e1c5e6eeeb28e85e584e8798aa26700819cb008/Lab-02/Configs/Router-1):
~~~

!
! Last configuration change at 09:37:33 UTC Wed Apr 12 2023
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname Router1
!
boot-start-marker
boot-end-marker
!
enable password class
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
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
 no ip address
!
interface Ethernet0/0.20
 no shutdown
 encapsulation dot1Q 20
 ip address 10.128.20.254 255.255.255.0
!
interface Ethernet0/0.30
 no shutdown
 encapsulation dot1Q 30
 ip address 10.128.30.254 255.255.255.0
!
interface Ethernet0/0.254
 no shutdown
 encapsulation dot1Q 254
 ip address 10.128.254.254 255.255.255.0
!
interface Ethernet0/1
 no shutdown
 no ip address
 shutdown
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
!
control-plane
!
banner motd Attemtion! Need Password!
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
~~~

> Пример конфигурации коммутатора [Switch-4](https://github.com/Samurai1135/otus-network-engeneer/blob/5e792fcf66594cdf07a56e5f768891060c2ee396/Lab-02/Configs/SW4):

~~~
!
! Last configuration change at 09:39:21 UTC Wed Apr 12 2023
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname S4
!
boot-start-marker
boot-end-marker
!
enable password class
!
no aaa new-model
!
no ip domain-lookup
ip cef
no ipv6 cef
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
interface Ethernet0/0
 no shutdown
 switchport trunk allowed vlan 20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 no shutdown
 switchport trunk allowed vlan 20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 no shutdown
 switchport trunk allowed vlan 20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/3
 no shutdown
 switchport access vlan 30
 switchport mode access
!
interface Ethernet1/0
 no shutdown
 switchport access vlan 20
 switchport mode access
!
interface Ethernet1/1
 no shutdown
!
interface Ethernet1/2
 no shutdown
!
interface Ethernet1/3
 no shutdown
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
control-plane
!
banner motd ATTENTION!!! Password needed!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 password cisco
 login
!
!
end
~~~

- ## Часть 2. Определение коммутатора в качестве корневого моста (Root Bridge)

![](https://github.com/Samurai1135/otus-network-engeneer/blob/d84951344bef66936550fe3a89f464a0818b0991/Lab-02/NetworkScheme/STP%20Vlan2.png)

В качестве корневого моста (Root Bridge) был добавлен коммутатор Switch-1, т.к. при одинаковых значениях (в лабораторной работе) PriorityID приоритет выбора падает на коммутатор, чей BridgeID-адрес меньше остальных.

- ## Часть 3. Результаты наблюдения процесса выбора протоколом STP порта, исходя из стоимости пути.

Ниже приведены результаты команды "show spanning-tree active" на каждом из коммутаторов:

### SW1
![](https://github.com/Samurai1135/otus-network-engeneer/blob/031e2b010c3fd1fb508cc59eb9eda3876dd8562a/Lab-02/NetworkScheme/RootBridge-SW1-VLAN20.png)
### SW2
![](https://github.com/Samurai1135/otus-network-engeneer/blob/031e2b010c3fd1fb508cc59eb9eda3876dd8562a/Lab-02/NetworkScheme/SW2-STP-VLAN20.png)
### SW3
![](https://github.com/Samurai1135/otus-network-engeneer/blob/031e2b010c3fd1fb508cc59eb9eda3876dd8562a/Lab-02/NetworkScheme/SW3-%20STP-VLAN20.png)
### SW4
![](https://github.com/Samurai1135/otus-network-engeneer/blob/031e2b010c3fd1fb508cc59eb9eda3876dd8562a/Lab-02/NetworkScheme/SW4-STP-VLAN20.png)
### SW5
![](https://github.com/Samurai1135/otus-network-engeneer/blob/031e2b010c3fd1fb508cc59eb9eda3876dd8562a/Lab-02/NetworkScheme/SW5-STP-VLAN20.png)

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
  ![](https://github.com/Samurai1135/otus-network-engeneer/blob/1c072f0c2af0372aeeadf2401aa17595c0de59ed/Lab-02/NetworkScheme/SW5-STP-e1:1.png)
<br>
Как видно из результатов команды "show spanning-tree active" на коммутаторе Switch5(SW5) интерфейс eth1/1 перешел в режим Alternate-port, т.к. PortID отправителя SW1(e0/2) меньше, чем у SW1(eth1/1), хотя оба транка связывают коммутатор с RootBridge и имеют одинаковую стоимость Cost.
В случае отключения транка на портах e0/2 коммутаторов SW5 и SW1 порт eth1/1 коммутатора SW5 станет Root-портом
