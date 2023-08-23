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



~~~
ip nat pool NAT-MOSCOW1 89.20.0.1 89.20.0.1 netmask 255.255.255.252
ip nat inside source list 1 pool NAT-MOSCOW1 overload
~~~
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
~~~
address-family ipv4
  network 10.128.254.15 mask 255.255.255.255
  network 89.20.0.0 mask 255.255.255.0
~~~