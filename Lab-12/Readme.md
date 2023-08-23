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

Пинг VPC7->VPC
~~~
VPCS> ping 192.168.5.1

84 bytes from 192.168.5.1 icmp_seq=1 ttl=55 time=2.610 ms
84 bytes from 192.168.5.1 icmp_seq=2 ttl=55 time=1.587 ms
84 bytes from 192.168.5.1 icmp_seq=3 ttl=55 time=1.624 ms
84 bytes from 192.168.5.1 icmp_seq=4 ttl=55 time=1.594 ms
84 bytes from 192.168.5.1 icmp_seq=5 ttl=55 time=1.533 ms
~~~
~~~
ip nat pool NAT-SPB 89.30.0.1 89.30.0.5 netmask 255.255.255.248
ip nat inside source list 1 pool NAT-SPB overload
ip route 0.0.0.0 0.0.0.0 10.0.2.1
ip route 0.0.0.0 0.0.0.0 10.0.4.1
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
~~~
R15(config)#ip nat inside source static 10.128.31.2 89.20.0.5
~~~
~~~
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 89.20.0.5:1       10.128.31.2:1      89.30.0.1:1        89.30.0.1:1
icmp 89.20.0.5:3       10.128.31.2:3      10.128.254.23:3    10.128.254.23:3
icmp 89.20.0.5:4       10.128.31.2:4      10.128.254.21:4    10.128.254.21:4
--- 89.20.0.5          10.128.31.2        ---                ---
~~~
~~~
R15(config)#ip nat source static tcp 10.128.30.2 23 89.20.0.10 2023 extendable
~~~
~~~
VPCS> ping 89.20.0.10 -p 2023

84 bytes from 89.20.0.10 icmp_seq=1 ttl=253 time=1.002 ms
84 bytes from 89.20.0.10 icmp_seq=2 ttl=253 time=0.952 ms
84 bytes from 89.20.0.10 icmp_seq=3 ttl=253 time=0.947 ms
84 bytes from 89.20.0.10 icmp_seq=4 ttl=253 time=0.921 ms
84 bytes from 89.20.0.10 icmp_seq=5 ttl=253 time=0.923 ms
~~~
~~~
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 89.20.0.5:12      10.128.31.2:12     10.8.0.1:12        10.8.0.1:12
--- 89.20.0.5          10.128.31.2        ---                ---
icmp 89.20.0.1:9753    192.168.6.1:9753   89.20.0.5:9753     89.20.0.5:9753
icmp 89.20.0.1:10009   192.168.6.1:10009  89.20.0.5:10009    89.20.0.5:10009
icmp 89.20.0.1:10265   192.168.6.1:10265  89.20.0.5:10265    89.20.0.5:10265
icmp 89.20.0.1:10521   192.168.6.1:10521  89.20.0.5:10521    89.20.0.5:10521
icmp 89.20.0.1:10777   192.168.6.1:10777  89.20.0.5:10777    89.20.0.5:10777
icmp 89.20.0.1:13337   192.168.6.1:13337  89.20.0.5:13337    89.20.0.5:13337
icmp 89.20.0.1:13593   192.168.6.1:13593  89.20.0.5:13593    89.20.0.5:13593
icmp 89.20.0.1:13849   192.168.6.1:13849  89.20.0.5:13849    89.20.0.5:13849
icmp 89.20.0.1:14105   192.168.6.1:14105  89.20.0.5:14105    89.20.0.5:14105
icmp 89.20.0.1:14361   192.168.6.1:14361  89.20.0.5:14361    89.20.0.5:14361
~~~

~~~
R12(config)#ip dhcp excluded-address 192.168.6.245
R12(config)#ip dhcp excluded-address 192.168.6.5 192.168.6.127
R12(config)#ip dhcp excluded-address 192.168.7.5 192.168.7.127
R12(config)#ip dhcp excluded-address 192.168.7.254
R12(config)#ip dhcp pool VLAN-6
R12(dhcp-config)#network 192.168.6.0 255.255.255.0
R12(dhcp-config)#default-router 10.128.25.1
R12(dhcp-config)#domain-name moscow.net
R12(dhcp-config)#ex
R12(config)#ip dhcp pool VLAN-7
R12(dhcp-config)#network 192.168.7.0 255.255.255.0
R12(dhcp-config)#default-router 10.128.26.1
R12(dhcp-config)#domain-name moscow.net

~~~
~~~
R13(config)#ip dhcp excluded-address 192.168.6.128 192.168.6.254
R13(config)#ip dhcp excluded-address 192.168.7.128 192.168.7.254
R13(config)#ip dhcp pool VLAN-6
R13(dhcp-config)#network 192.168.6.0 255.255.255.0
R13(dhcp-config)#default-router 10.128.25.1
R13(dhcp-config)#domain-name moscow.net
R13(config)#ip dhcp pool VLAN-7
R13(dhcp-config)#network 192.168.7.0 255.255.255.0
R13(dhcp-config)#default-router 10.128.26.1
R13(dhcp-config)#domain-name moscow.net
~~~
~~~
VPCS> ip dhcp -r
DDORA IP 192.168.7.2/24 GW 192.168.7.254
~~~
~~~
VPCS> ip dhcp
DORA IP 192.168.6.2/24 GW 192.168.6.253
~~~
