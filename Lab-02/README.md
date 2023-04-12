
## Настройка STP

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

![](https://github.com/Samurai1135/otus-network-engeneer/blob/e9c80de0cbc62366fff3cf5a6a07605fa480f33a/Lab-02/NetworkScheme/Topology.png)
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

> Пример конфигурации коммутатора [Switch-4](https://github.com/Samurai1135/otus-network-engeneer/blob/5e792fcf66594cdf07a56e5f768891060c2ee396/Lab-02/Configs/SW4):

~~~
Last configuration change at 12:58:52 UTC Tue Apr 11 2023
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname Switch-4
!
boot-start-marker
boot-end-marker
!
no aaa new-model
!
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
 switchport trunk allowed vlan 20,30,254
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 no shutdown
 switchport trunk allowed vlan 20,30,254
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 no shutdown
 switchport trunk allowed vlan 20,30,254
 switchport trunk encapsulation dot1q
 switchport mode trunk
 shutdown
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
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
!
end
}
~~~

- ## Часть 2. Определение коммутатора в качестве корневого моста (Root Bridge)

![](https://github.com/Samurai1135/otus-network-engeneer/blob/d84951344bef66936550fe3a89f464a0818b0991/Lab-02/NetworkScheme/STP%20Vlan2.png)

В качестве корневого моста (Root Bridge) был добавлен коммутатор Switch-1, т.к. при одинаковых значениях (в лабораторной работе) BridgeID приоритет выбора падает на коммутатор, чей MainMAC-адрес меньше остальных.

- ## Часть 3. Результаты наблюдения процесса выбора протоколом STP порта, исходя из стоимости портов.

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
Стоимость портов Cost на коммутаторах равна 100, что говорит о прямом подключении к RouteBridge. Исключение составляет коммутатор Switch4(SW4), чья стоимость порта Cost = 200, т.к. на маршрутах между ним и RootBridge есть еще по одному сетевому устройству - коммутатор Switch2(SW2) (по пути слева на схеме) и Switch3(SW3) (по пути справа на схеме)
