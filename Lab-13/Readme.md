# VPN. GRE. DmVPN

`Цель:`
Настроить GRE между офисами Москва и С.-Петербург
Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги


# Задание:

- `Часть1.`Настроить GRE между офисами Москва и С.-Петербург.
- `Часть2.`Настроить DMVMN между Москва и Чокурдах, Лабытнанги.
- `Часть3.`Все узлы в офисах в лабораторной работе должны иметь IP связность.

## Настроим GRE-тоннель между Москвой и Питером:
Проверяем IP-связность
~~~
R18#ping 89.20.0.15 source 89.30.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 89.20.0.15, timeout is 2 seconds:
Packet sent with a source address of 89.30.0.18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

//R18#ping 89.20.0.14 source 89.30.0.18
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 89.20.0.14, timeout is 2 seconds:
Packet sent with a source address of 89.30.0.18
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~
Строим GRE-тоннель на бордерах Москвы
R14:
~~~
interface Tunnel2
 ip address 10.100.1.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 89.20.0.14
 tunnel destination 89.30.0.18
!
~~~
R15:
~~~
interface Tunnel1
 ip address 10.100.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 89.20.0.15
 tunnel destination 89.30.0.18
~~~
Далее настраиваем бордер Питера аналогичным образом:
 ~~~
interface Tunnel1
 ip address 10.100.0.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 89.30.0.18
 tunnel destination 89.20.0.15
!
interface Tunnel2
 ip address 10.100.1.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 89.30.0.18
 tunnel destination 89.20.0.14
!

~~~
Проверяем, поднялись ли тоннели:
~~~
R18#sh ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.128.2.1      YES NVRAM  up                    up
Ethernet0/1                10.128.0.1      YES NVRAM  up                    up
Ethernet0/2                10.0.4.2        YES NVRAM  up                    up
Ethernet0/3                10.0.2.2        YES NVRAM  up                    up
Loopback0                  10.128.254.18   YES NVRAM  up                    up
NVI0                       10.128.2.1      YES unset  up                    up
Tunnel1                    10.100.0.2      YES NVRAM  up                    up
Tunnel2                    10.100.1.2      YES NVRAM  up                    up
~~~
Проверим сетевую связность бордеров через тоннели:
~~~
R18#ping 10.100.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R18#ping 10.100.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~

## Настроим DMVPN между офисами Москва и Чокурдах, Лабытнанги
Проверяем связность:
~~~
R15#ping 10.0.3.2 source 89.20.0.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.3.2, timeout is 2 seconds:
Packet sent with a source address of 89.20.0.15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

//R14#ping 10.0.3.2 source 89.20.0.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.3.2, timeout is 2 seconds:
Packet sent with a source address of 89.20.0.14
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~
Дописываем недостающие маршруты на R28:
~~~
ip route 89.20.0.14 255.255.255.255 10.0.1.1
ip route 89.20.0.15 255.255.255.255 10.0.0.1
~~~
~~~
R15#ping 10.0.1.1 source 89.20.0.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.1, timeout is 2 seconds:
Packet sent with a source address of 89.20.0.15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 10.0.0.1 source 89.20.0.15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
Packet sent with a source address of 89.20.0.15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~

Объединять в виртуальную сеть будем следующим образом (для роутеров указаны connected-интерфейсы и роли):

Офис в Москве - HUB:
~~~
R14: 89.20.0.14
R15: 89.20.0.15
~~~
Лабынтаги и Чокурдах - SPOKE:
~~~
R27: 10.0.3.2
R28: 10.0.0.1, 10.0.1.1
~~~

Настроим R15 с ролью HUB:
~~~
interface Tunnel10
 ip address 10.10.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 10
 ip tcp adjust-mss 1360
 tunnel source 89.20.0.15
 tunnel mode gre multipoint
 tunnel key 200
~~~
 tunnel key необходимо задать так как в Лабытнангах оба тунеля приземляются в один интерфейс.

R14 настраиваем аналогично.
~~~
interface Tunnel11
 ip address 10.11.0.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 10
 ip tcp adjust-mss 1360
 tunnel source 89.20.0.14
 tunnel mode gre multipoint
 tunnel key 200
!
~~~
Настроим R27 с ролью SPOKE:
~~~
interface Tunnel10
 ip address 10.10.0.5 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 89.20.0.15
 ip nhrp map 10.10.0.1 89.20.0.15
 ip nhrp network-id 10
 ip nhrp nhs 10.10.0.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 200
!
interface Tunnel11
 ip address 10.11.0.5 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 89.20.0.14
 ip nhrp map 10.11.0.1 89.20.0.14
 ip nhrp network-id 10
 ip nhrp nhs 10.11.0.1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 200
!
~~~
На обоих интерфейсах указывается один и тот же адрес источника.

Проверяем:
~~~
R27#ping 10.10.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~
~~~
interface Tunnel10
 ip address 10.10.0.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 89.20.0.15
 ip nhrp map 10.10.0.1 89.20.0.15
 ip nhrp network-id 200
 ip nhrp holdtime 600
 ip nhrp nhs 10.10.0.1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 200
!
interface Tunnel11
 ip address 10.11.0.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 89.20.0.14
 ip nhrp map 10.11.0.1 89.20.0.14
 ip nhrp network-id 10
 ip nhrp holdtime 600
 ip nhrp nhs 10.11.0.1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 200
!
~~~
~~~
R15#sh ip nhrp
10.10.0.3/32 via 10.10.0.3
   Tunnel10 created 04:41:23, expire 00:08:58
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 10.0.0.2
10.10.0.5/32 via 10.10.0.5
   Tunnel10 created 04:41:37, expire 01:59:38
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 10.0.3.2
~~~
~~~
R15#ping 10.10.0.5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R15#ping 10.10.0.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~
~~~
R28#ping 10.11.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.11.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R28#ping 10.10.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R28#ping 10.10.0.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.0.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
~~~
Аналогичный тоннель строится и на бордере R14
