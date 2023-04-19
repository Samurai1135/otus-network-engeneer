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
- `Часть 6.` Настройка роутера R2. Настройка маршрута между роутерами R1 и R2

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

~~~
ip dhcp relay information trust-all
~~~
Реле - свойство маршрутизатора ретранслировать работу служб с роутера, на котором поднят сервер
~~~
ip helper-address 10.0.0.1
~~~
`ip helper-address {address}` //команда, с помощью которой указываем интерфейс, с которого будем получать параметры DHCP (в нашем случае) 


## Проверим работу нашей сети и правильность назначения IPv4-адресов
#### PC5:
~~~
VPCS> ip dhcp
DDORA IP 192.168.1.1/24 GW 192.168.1.254
~~~
#### PC6:
~~~
VPCS> ip dhcp
DDORA IP 192.168.220.1/24 GW 192.168.220.2
~~~
#### Пропингуем PC6 с PC5:
~~~
VPCS> ping 192.168.220.1

84 bytes from 192.168.220.1 icmp_seq=1 ttl=62 time=1.763 ms
84 bytes from 192.168.220.1 icmp_seq=2 ttl=62 time=0.858 ms
84 bytes from 192.168.220.1 icmp_seq=3 ttl=62 time=0.816 ms
84 bytes from 192.168.220.1 icmp_seq=4 ttl=62 time=0.702 ms
84 bytes from 192.168.220.1 icmp_seq=5 ttl=62 time=0.801 ms
~~~
Как видим, сеть работает!


- ## Часть 4. Настройка топологии IPv6 сети и основных параметров коммутаторов и маршрутизаторов
Будем использовать схему сети из предыдущего примера:
  ![](https://github.com/Samurai1135/otus-network-engeneer/blob/45c34561b083ef0c8ed43fcb8247d5be92a706f6/Lab-03/NetworkScheme/IPv6.png)

#### Используемое оборудование и конфигурация портов:
<br>
Префикс подсети со шлюзом на R1: FE80::1
Префикс подсети со шлюзом на R2: FE80::2

| Наименование       | Обозначение на схеме |  Порт подключения и    IP адресация |
| :----------------- | :------------------- | :---------------------------------- |
|Маршрутизатор/роутер| R1              |  eth0/0 -- IPv6: 2001:DB8:ACAD:2::1/64  |  
|                    |                      |  eth0/1 --  IPv6: 2001:DB8:ACAD:1::1/64 |
|Маршрутизатор/роутер| R2              |  eth0/0 -- IPv6: 2001:DB8:ACAD:2::2/64  |  
|                    |                      |  eth0/1 --  IPv6: 2001:DB8:ACAD:3::1/64 |
|Коммутатор/Свитч 1. | S1              |  eth0/1 - trunk, eth0/0 - access  |
|Коммутатор/Свитч 2. | S2              |  eth0/1 - trunk, eth0/0 - access|
|Пользовательский ПК1| PC1                 |  eth0 -- IPv6: DHCPv6         |
|Пользовательский ПК2| PC2                 |  eth0 -- IPv6: DHCPv6        |

<br>

- ## Часть 5. Настройка DHCPv6-сервера на роутере R1. Натройка пулов адресов и портов. 

### Настройка пулов адресов:
~~~
ipv6 unicast-routing
ipv6 dhcp pool R1-STATELESS
 dns-server 2001:DB8:ACAD::254
 domain-name STATELESS.com
!
ipv6 dhcp pool R2-STATEFUL
 address prefix 2001:DB8:ACAD:3:AAA::/80
 dns-server 2001:DB8:ACAD::254
 domain-name STATEFUL.com
~~~
`ipv6 unicast-routing` // включаем маршрутизацию по протоколу IPv6 <br>
`ipv6 dhcp pool {PoolName}` // задаем имя пула адресов IPv6 <br>
`dns-server {address}` // указываем адрес DNS-сервера <br>
`domain-name`// указываем имя домена <br>
`address prefix {IPv6 prefix}` // назначаемый префикс адреса IPv6, который DHCPv6-сервер будет добавлять к адресам сетевых устройств <br>

### Настройка интерфейсов:
~~~
interface Ethernet0/0
 no ip address
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
 ipv6 dhcp server R2-STATEFUL
!
interface Ethernet0/1
 no ip address
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server R1-STATELESS
~~~
`no ip address` // отсутствует IPv4-адрес (мы используем протокол IPv6) <br>
`ipv6 address {address} link-local` // адрес IPv6-подсети <br>
`ipv6 nd other-config-flag` // префикс сети предоставляется по протоколу SLAAC (без помощи DHCPv6) <br>

- ## Часть 6. Настройка маршрута между роутерами R1 и R2
~~~
ipv6 route ::/0 2001:DB8:ACAD:2::2
~~~
