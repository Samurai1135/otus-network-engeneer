# OSPF. Фильтрация
`Цель:`
Настроить OSPF офисе Москва
Разделить сеть на зоны
Настроить фильтрацию между зонами

# Задание:
- `Часть1:` Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.
- `Часть2:` Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
- `Часть3:` Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.
- `Часть4:` Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.
<br><br>
#### Схема офиса Москва
![](https://github.com/Samurai1135/otus-network-engeneer/blob/85d1767c530c643491340105688876f95122c6ef/Lab-06/ScreenShots/stand.png)

- ## Часть 1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.

Area 0 - core level
Area 10 - distribution level

Для уменьшения служебного трафика на R14 и R15 переведем неиспользуемые интерфейсы в пассивный режим (hello-пакеты не будут пересылаться)

 ~~~
router ospf 1
 router-id 2.2.2.2               //R14 router-id (R15 - 1.1.1.1)
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 no passive-interface Ethernet1/0
!         
 ~~~
Настроим линк между роутерами в нулевой зоне (Area 0) на R14 и R15
~~~
!         
interface Ethernet1/0
 ip address 10.128.150.1 255.255.255.0  //R14
 ip ospf 1 area 0
!         
~~~

- ## Часть 2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
Поскольку в зоне 10 у нас находятся 3-х уровневые свичи, настройке ниже так-же должны быть применены к ним:
(сперва присоединим к зоне 10 R14 и R15)
~~~
interface Ethernet0/1          //R14
 ip address 10.128.21.1 255.255.255.252
 ip ospf 1 area 10
! 
~~~
~~~
!         
interface Ethernet0/1         //R15
 ip address 10.128.22.1 255.255.255.252
 ip ospf 1 area 10
~~~
Произведем аналогичные настройки для R12, R13, SW4 и SW5 и проверим установление соседства:
~~~
R15#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:37    10.128.150.1    Ethernet1
/0
4.4.4.4           1   FULL/BDR        00:00:35    10.128.22.2     Ethernet0
/1
~~~
~~~
R14#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:34    10.128.150.2    Ethernet1
/0
6.6.6.6           1   FULL/BDR        00:00:34    10.128.21.2     Ethernet0
/1
~~~
~~~
R12#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
5.5.5.5           1   FULL/BDR        00:00:36    10.128.25.2     Ethernet0
/0
3.3.3.3           1   FULL/DR         00:00:38    10.128.25.9     Ethernet0
/1
1.1.1.1           1   FULL/DR         00:00:37    10.128.22.1     Ethernet0
/3
~~~
Как видим, соседство установлено. <br>
# Анонс маршрута по-умолчанию

Нв маршрутизаторе R15 задаем маршрут по-умолчанию:
~~~
R15(config)#ip route 0.0.0.0 0.0.0.0 10.8.0.1
~~~
И в настройках OSPF вводим команду:
~~~
R15(config-router)#default-information originate 
~~~
Проверим на R12 появление нового маршрута по-умолчанию:
~~~
R12#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-
2
       ia - IS-IS inter area, * - candidate default, U - per-user static ro
ute
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.128.22.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.128.22.1, 00:01:54, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 15 subnets, 3 masks
O        10.128.21.0/30 [110/30] via 10.128.25.9, 00:26:13, Ethernet0/1
                        [110/30] via 10.128.25.2, 00:26:13, Ethernet0/0
O        10.128.23.0/30 [110/30] via 10.128.25.9, 00:26:23, Ethernet0/1
                        [110/30] via 10.128.25.2, 00:26:23, Ethernet0/0
O        10.128.26.0/30 [110/20] via 10.128.25.9, 00:28:13, Ethernet0/1
O        10.128.26.8/30 [110/20] via 10.128.25.2, 00:27:09, Ethernet0/0
O IA     10.128.150.0/24 [110/20] via 10.128.22.1, 00:22:07, Ethernet0/3
O        10.128.200.0/24 [110/20] via 10.128.25.9, 00:28:13, Ethernet0/1
                         [110/20] via 10.128.25.2, 00:26:59, Ethernet0/0
O        10.128.201.0/24 [110/20] via 10.128.25.9, 00:28:13, Ethernet0/1
                         [110/20] via 10.128.25.2, 00:26:59, Ethernet0/0
~~~
O*E2 - это и есть маршрут по-умолчанию

- ## Часть 3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.

Для начала на R19(eth0/0) и R14(eth0/3) "погрузим" в зону 101 (Area101)
~~~
R14(config)#int e0/3
R14(config-if)#ip os
R14(config-if)#ip ospf 1 ar
R14(config-if)#ip ospf 1 area 101
~~~
На R19 должны подтянуться все известные маршруты соседних сетей:
~~~
R19#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-
2
       ia - IS-IS inter area, * - candidate default, U - per-user static ro
ute
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.128.30.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.128.30.1, 00:00:02, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 13 subnets, 3 masks
O IA     10.128.20.0/30 [110/50] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.21.0/30 [110/20] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.22.0/30 [110/50] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.23.0/30 [110/30] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.25.0/30 [110/40] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.25.8/30 [110/40] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.26.0/30 [110/30] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.26.8/30 [110/30] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.150.0/24 [110/20] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.200.0/24 [110/40] via 10.128.30.1, 00:00:02, Ethernet0/0
O IA     10.128.201.0/24 [110/40] via 10.128.30.1, 00:00:02, Ethernet0/0
~~~
Как видим, подгрузились все маршруты. Но нам нужен только маршрут по-умолчанию.
Сделаем фильтрацию.
Сперва заблокируем маршруты из других зон:
~~~
R19(config-router)#area 101 stub no-summary 
~~~
Потом выведем все доступные на R19 маршруты:

~~~
R19#sh ip route     
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-
2
       ia - IS-IS inter area, * - candidate default, U - per-user static ro
ute
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.128.30.1 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.128.30.1, 00:00:44, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.128.30.0/30 is directly connected, Ethernet0/0
L        10.128.30.2/32 is directly connected, Ethernet0/0
~~~
Все маршруты пропали, кроме маршрута по-умолчанию.


















