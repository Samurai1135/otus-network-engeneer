# Основные протоколы сети интернет. NAT. DHCP. NTP
`Цель:`  
Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах  
Настроить синхронизацию времени в офисе Москва  
Настроить DHCP в офисе Москва  

# Задание:  
- `Часть1.`Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
- `Часть2.`Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
- `Часть3.`Настроить статический NAT для R20.
- `Часть4.`Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
- `Часть5.`Настроить статический NAT(PAT) для офиса Чокурдах.
- `Часть6.`Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
- `Часть7.`Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.
- `Часть8.`Все офисы в лабораторной работе должны иметь IP связность.

![](https://github.com/Samurai1135/otus-network-engeneer/blob/e740525ec174aa37cd294218ebd497b0dc22b089/Lab-12/Pics/Scheem.png)  
## Настройка NAT в офисе Москва на R14 и R15.
Внутренние сети будем скрывать под адресом 89.20.0.1 . Условно, адрес, выданный нам провайдером.  
~~~
ip nat pool NAT-MOSCOW1 89.20.0.1 89.20.0.1 netmask 255.255.255.252
ip nat inside source list 1 pool NAT-MOSCOW1 overload
~~~
В данном случае `overload` говорит об использовании PAT
~~~
access-list 1 permit 192.168.0.0 0.0.255.255
~~~

~~~
interface Ethernet0/0
 ip address 10.128.23.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.128.22.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 89.20.0.1 255.255.255.0 secondary
 ip address 10.8.0.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/3
 ip address 10.128.31.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 102
!
~~~
Необходимо анонсировать "новую" сеть в BGP, удалив при этом старые network (локальные сети 192.168.6.0 и 192.168.7.0 ) - они будут попросту не нужны
~~~
address-family ipv4
  network 89.20.0.0 mask 255.255.255.0
~~~
Аналогичную настройку делаем и на R14
## Проверка связи Москвы с Петербургом
Пинг VPC7->VPC
~~~
VPCS> ping 192.168.5.1

84 bytes from 192.168.5.1 icmp_seq=1 ttl=55 time=2.610 ms
84 bytes from 192.168.5.1 icmp_seq=2 ttl=55 time=1.587 ms
84 bytes from 192.168.5.1 icmp_seq=3 ttl=55 time=1.624 ms
84 bytes from 192.168.5.1 icmp_seq=4 ttl=55 time=1.594 ms
84 bytes from 192.168.5.1 icmp_seq=5 ttl=55 time=1.533 ms
~~~
## Настроим PAT на R18, используя для трансяции пул из 5 адресов
Поскольку мы ранее создавали правила трансляции только внутренних подсетей - необходимо еще добавить правило для трансляции в BGP выбранного пула адресов (поскольку предполагается использование NAT - транслирование локальных сетей теряет смысл)
~~~
ip nat pool NAT-SPB 89.30.0.1 89.30.0.5 netmask 255.255.255.248
ip nat inside source list 1 pool NAT-SPB overload
ip route 89.30.0.0 255.255.255.248 Null0
!
!
ip prefix-list NO-TRANSIT seq 5 permit 192.168.4.0/24
ip prefix-list NO-TRANSIT seq 10 permit 192.168.5.0/24
ip prefix-list NO-TRANSIT seq 20 permit 89.30.0.0/29
!
!
access-list 1 permit 192.168.0.0 0.0.255.255
~~~
## Настроим статический NAT для R20
~~~

R15(config)#ip nat inside source static 10.128.31.2 89.20.0.5
~~~
Проверим результаты вывода команды show ip nat translations (показать все преобразования адресов (NAT))
~~~
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 89.20.0.5:1       10.128.31.2:1      89.30.0.1:1        89.30.0.1:1
icmp 89.20.0.5:3       10.128.31.2:3      10.128.254.23:3    10.128.254.23:3
icmp 89.20.0.5:4       10.128.31.2:4      10.128.254.21:4    10.128.254.21:4
--- 89.20.0.5          10.128.31.2        ---                ---
~~~
## Настроим NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
Сперва настроим на R19 telnet:
~~~
configure terminal
line vty 0 4
transport input telnet 
password otus
login
~~~
Защитим подключение к R19 с помощью "подмены" 23 порта на 2023:
~~~
R15(config)#ip nat source static tcp 10.128.30.2 23 89.20.0.10 2023
~~~
~~~
VPCS> ping 89.20.0.10 -p 2023 -P 6

Connect 2023@89.20.0.10 seq=1 ttl=248 time=2.122 ms
SendData 2023@89.20.0.10 seq=1 ttl=248 time=1.069 ms
Close 2023@89.20.0.10 timeout(9.559ms)
Connect 2023@89.20.0.10 seq=2 ttl=248 time=4.300 ms
SendData 2023@89.20.0.10 seq=2 ttl=248 time=1.077 ms
Close 2023@89.20.0.10 timeout(9.557ms)
Connect 2023@89.20.0.10 seq=3 ttl=248 time=4.289 ms
SendData 2023@89.20.0.10 seq=3 ttl=248 time=1.075 ms
Close 2023@89.20.0.10 timeout(10.618ms)
Connect 2023@89.20.0.10 seq=4 ttl=248 time=4.287 ms
SendData 2023@89.20.0.10 seq=4 ttl=248 time=1.069 ms
Close 2023@89.20.0.10 timeout(10.599ms)
Connect 2023@89.20.0.10 seq=5 ttl=248 time=4.301 ms
SendData 2023@89.20.0.10 seq=5 ttl=248 time=1.070 ms
Close 2023@89.20.0.10 timeout(6.366ms)
~~~
~~~
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
tcp 89.20.0.10:2023    10.128.30.2:23     ---                ---
--- 89.20.0.5          10.128.31.2        ---                ---
~~~
Доступ работает.  
## Настроим для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
Сперва на R12 создадим исключения для уже использующихся сетевыми устройствами адресов и добавим два DHCP-пула для каждой из VLAN,  
Ограничим пулы для R12 маршрутизатора с 128 по 254 адрес, исключив адреса 192.168.6(7).1-192.168.6(7).127 для избежания конфликта адресов с базой DHCP-сервера на R13:
~~~
R12(config)#ip dhcp excluded-address 192.168.6.254
R12(config)#ip dhcp excluded-address 192.168.6.1 192.168.6.127
R12(config)#ip dhcp excluded-address 192.168.7.1 192.168.7.127
R12(config)#ip dhcp excluded-address 192.168.7.254
R12(config)#ip dhcp pool VLAN-6
R12(dhcp-config)#network 192.168.6.0 255.255.255.0
R12(dhcp-config)#default-router 192.168.6.254
R12(dhcp-config)#domain-name moscow.net
R12(dhcp-config)#ex
R12(config)#ip dhcp pool VLAN-7
R12(dhcp-config)#network 192.168.7.0 255.255.255.0
R12(dhcp-config)#default-router 192.168.7.254
R12(dhcp-config)#domain-name moscow.net
~~~
Подобным образом настроим R13, исключив адреса с 128 по 254 (используются на R12):
~~~
R13(config)#ip dhcp excluded-address 192.168.6.128 192.168.6.254
R13(config)#ip dhcp excluded-address 192.168.7.128 192.168.7.254
R13(config)#ip dhcp pool VLAN-6
R13(dhcp-config)#network 192.168.6.0 255.255.255.0
R13(dhcp-config)#default-router 192.168.6.254
R13(dhcp-config)#domain-name moscow.net
R13(config)#ip dhcp pool VLAN-7
R13(dhcp-config)#network 192.168.7.0 255.255.255.0
R13(dhcp-config)#default-router 192.168.7.254
R13(dhcp-config)#domain-name moscow.net
~~~
Необходимо добавить, что наши свичи SW4 и SW5 - L3 коммутаторы. Для того, чтобы ответ от DHCP на широковещательный запрос доходил до адресата, необходимо на интерфейсах коммутаторов, смотрящих в локальные сети, прописать адрес `ip helper-address` (адрес смотрящего на локальные сети интерфейса DHCP-сервера).
Выполнив требуемые настройки запросим IP-адрес для VPC от DHCP-сервер(а/ов):
~~~
VPCS> ip dhcp -r
DDORA IP 192.168.7.2/24 GW 192.168.7.254
~~~
~~~
VPCS> ip dhcp
DORA IP 192.168.6.2/24 GW 192.168.6.253
~~~
## Настроим NTP сервера на R12 и R13
Поскольку доступа к Интернет нет и не с чем синхронизироваться, сделаем R12 и R13 NTP-мастерами. Другие сетевые устройства будут синхронизировать свои часы ориентируясь на эти серверы.
На R12 и R13 сделаем настройку:
~~~
conf t
!
ntp master 2
exit
!
~~~
А на сетевых устройствах нашей АС:
~~~
conf t
!
ntp server 192.168.25.1
ntp server 192.168.26.1
!
ntp update-calendar
!
~~~
Проверим синхронизацию времени на R15:
~~~
R15#sh ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.128.25.1     .LOCL.           1     35    128     7  0.000   0.000  2.657
+~10.128.26.1     127.127.1.1      2     26    128     7  0.000   0.000  2.325
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
~~~
... и на SW4
~~~
SW4#sh ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
*~10.128.25.1     .LOCL.           1     15    128    17  0.000   0.000  2.812
+~10.128.26.1     127.127.1.1      2     12    128    17  0.000   0.000  2.555
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
~~~
