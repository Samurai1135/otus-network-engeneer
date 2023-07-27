# iBGP
`Цель:`
Настроить iBGP в офисе Москва
Настроить iBGP в сети провайдера Триада
Организовать полную IP связанность всех сетей

# Задание
- `Часть1.` Настроить iBGP в офисом Москва между маршрутизаторами R14 и R15.
- `Часть2.` Настроить iBGP в провайдере Триада.
- `Часть3.` Настройть офис Москва так, чтобы приоритетным провайдером стал Ламас.
- `Часть4.` Настройть офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
- `Часть5.` Все сети в лабораторной работе должны иметь IP связность.


Для удобства будем настривать iBGP на Loopback-адресах роутеров, т.к. это избавит нас от привязки к физическому линку. В случае потери линка проблема будет устранена переключением магистрали на другой порт.
#### Важные моменты:
- `next-hop` и `AS_PATH` внутри iBGP не изменяется. В iBGP `next-hop` указывает адрес выхода из АС.  
- всегда `full mesh` и маршруты, полученные от соседа, не анонсируются дальше  
- выставлен `multihop != 1`, так как часто используются Loopback-адреса.  

#### Настройка iBGP (На примере R14):
~~~
router bgp 1001
 bgp router-id 10.128.254.14
 neighbor 10.128.254.15 remote-as 1001
 neighbor 10.128.254.15 update-source Loopback0
~~~
## Настройка iBGP в офисе Москвы между маршрутизаторами R14 и R15
Настроим Loopbak-адреса на R14 и R15
~~~
R14(config)#interface Loopback0
R14(config-if)#ip address 10.128.254.14 255.255.255.255
R14(config-if)#ip ospf 1 area 0
R14(config-if)#end
~~~
~~~
R15(config)#int loopback0
R15(config-if)#ip address 10.128.254.15 255.255.255.255
R15(config-if)#ip ospf 1 area 0
R15(config-if)#exit
~~~
Установим BGP-соседство указав update-source Loopback0 для установки соединения через Loopbaсk'и:
~~~
R14(config)#router bgp 1001
R14(config-router)#neighbor 10.128.254.15 update-source Loopback0
~~~
~~~
R15(config)#router bgp 1001
R15(config-router)#neighbor 10.128.254.14 update-source Loopback0
~~~

Проверим установку BGP-соединения:
~~~
R15#sh ip bgp summary
BGP router identifier 10.128.254.15, local AS number 1001
BGP table version is 4, main routing table version 4
3 network entries using 420 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1116 total bytes of memory
BGP activity 3/0 prefixes, 4/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.8.0.1        4          301      38      37        4    0    0 00:29:59        1
10.128.254.14   4         1001      10       9        4    0    0 00:03:16        1
~~~
## Настройка iBGP в Триаде
Произведем полную настройку BGP аналогично офису в Москве и проверим результаты установки BGP-соединения между роутерами:
~~~
Router#sh ip bgp summary

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.128.254.24   4          520       0       0        1    0    0 never    Idle
10.128.254.25   4          520       0       0        1    0    0 never    Idle
10.128.254.26   4          520       0       0        1    0    0 never    Idle
~~~
~~~
R24#sh ip bgp summary

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.4.2        4         2042      47      47        5    0    0 00:38:10        1
10.6.0.2        4          301      48      47        5    0    0 00:38:10        2
10.128.254.23   4          520       0       0        1    0    0 never    Idle
10.128.254.25   4          520       0       0        1    0    0 never    Idle
10.128.254.26   4          520       0       0        1    0    0 never    Idle
~~~
~~~
Router#sh ip bgp summary

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.128.254.23   4          520       0       0        1    0    0 never    Idle
10.128.254.24   4          520       0       0        1    0    0 never    Idle
10.128.254.26   4          520       0       0        1    0    0 never    Idle
~~~
~~~
R26#sh ip bgp summary

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.2.2        4         2042      51      49        2    0    0 00:42:00        1
10.128.254.23   4          520       0       0        1    0    0 never    Idle
10.128.254.24   4          520       0       0        1    0    0 never    Idle
10.128.254.25   4          520       0       0        1    0    0 never    Idle
~~~
Чтобы не клонировать настройки на новые пиры - можно создать peer-group с набором настроек и тиражировать ее при добавлении нового пира в AS:
~~~
R23(config-router)#neighbor triada peer-group
R23(config-router)#neighbor triada remote-as 520
R23(config-router)#neighbor triada update-source Loopback0
~~~
При подключении пиров:
~~~
R23(config-router)#neighbor 10.128.254.24 peer-group triada
R23(config-router)#neighbor 10.128.254.25 peer-group triada
R23(config-router)#neighbor 10.128.254.26 peer-group triada
~~~
## Настройка Route-Reflector
RouteReflector - маршрутизатор, который забирает в пределах зоны на себя весь трафик, рефлектирует обновленияи подставляет свой IP.
В качестве такого маршрутизатора выберем R23 и произведем настройку:
~~~
!
router bgp 520
 bgp log-neighbor-changes
 neighbor 10.128.254.24 remote-as 520
 neighbor 10.128.254.24 update-source Loopback0
 neighbor 10.128.254.24 route-reflector-client
 neighbor 10.128.254.25 remote-as 520
 neighbor 10.128.254.25 update-source Loopback0
 neighbor 10.128.254.25 route-reflector-client
 neighbor 10.128.254.26 remote-as 520
 neighbor 10.128.254.26 update-source Loopback0
 neighbor 10.128.254.26 route-reflector-client
!
~~~
Как видно, каждому соседу добавили параметр `route-reflector-client`
На соседних роутерах укажем R23 (Lo0:10.128.254.23) как `next-hop-self`:
~~~
router bgp 520
 bgp router-id 10.128.254.26
 bgp log-neighbor-changes
 neighbor 10.128.254.23 remote-as 520
 neighbor 10.128.254.23 update-source Loopback0
 neighbor 10.128.254.23 next-hop-self
!
~~~
