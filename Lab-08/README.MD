# EIGRP
`Цель:`
Настроить EIGRP в С.-Петербург; Использовать named EIGRP

# Задания:
- `Часть1.` В офисе С.-Петербург настроить EIGRP, R32 получает только маршрут по умолчанию.
- `Часть2.` R16-17 анонсируют только суммарные префиксы.
- `Часть3.` Использовать EIGRP named-mode для настройки сети.

#### Схема

![](https://github.com/Samurai1135/otus-network-engeneer/blob/2558d1ae6945ecc2dea16e128783420969eb5ebf/Lab-08/Screenshots/scheem.png)  

## В офисе С.-Петербург настроить EIGRP, R32 получает только маршрут по умолчанию.  
Произведем настройку EIGRP named-mode на маршрутизаторе R16 (используется семейство адресов address-family для унификации настройки сети по  IPv4 и IPv6):  
~~~
!
router eigrp SPB                                         //Включаем протокол EIGRP  и задаем ему имя SPB
 !
 address-family ipv4 unicast autonomous-system 2042      //Указываем семейство адресов в автономной системе 2042
  !
  af-interface Ethernet0/3                               //Указываем интерфес, смотрящий на R32
   summary-address 10.128.0.0 255.255.248.0              //Производим суммаризацию всех подсетей офиса на этом интерфейсе
  exit-af-interface
  !
  topology base                                          //Выбираем топологию по-умолчанию
  exit-af-topology
  network 10.128.1.0 0.0.0.3                             //Указываем подсети маршрутизатора
  network 10.128.1.8 0.0.0.3
  network 10.128.2.0 0.0.0.3
  network 10.128.4.0 0.0.0.3
  eigrp router-id 10.128.254.16                          //Присваиваем маршрутизатору ID, равный его Loopback-адресу
 exit-address-family
~~~

Настройка на R32:
~~~
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  topology base
  exit-af-topology
  network 10.128.4.0 0.0.0.3
  network 10.128.254.32 0.0.0.0
  eigrp router-id 10.128.254.32
 exit-address-family
~~~
#### Аналогичную настройку необходимо произвести на всех маршрутизаторах зоны AS-2042  
Обратимся к информации по протоколу EIGRP на R32:
~~~
Routing for Networks:
    10.128.4.0/30
    10.128.254.32/32
~~~
Сети маршрутизируются по EIGRP, проверим из доступность c R16:  
~~~
R16#sh ip eigrp neighbors
EIGRP-IPv4 VR(SPB) Address-Family Neighbors for AS(2042)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
3   10.128.1.2              Et0/2                    13 02:05:33    1   100  0  18
2   10.128.1.9              Et0/0                    12 02:05:34    3   100  0  10
1   10.128.4.2              Et0/3                    11 02:05:36    1   100  0  5
0   10.128.2.1              Et0/1                    13 02:05:37    2   100  0  5
~~~
Так-же на R16 можем видеть полученные маршруты:
~~~
      10.0.0.0/8 is variably subnetted, 16 subnets, 3 masks
D        10.128.0.0/21 is a summary, 02:06:49, Null0
D        10.128.0.0/30 [90/2048000] via 10.128.1.9, 02:06:49, Ethernet0/0
                       [90/2048000] via 10.128.1.2, 02:06:49, Ethernet0/2
D        10.128.0.8/30 [90/1536000] via 10.128.1.9, 02:06:49, Ethernet0/0
C        10.128.1.0/30 is directly connected, Ethernet0/2
L        10.128.1.1/32 is directly connected, Ethernet0/2
C        10.128.1.8/30 is directly connected, Ethernet0/0
L        10.128.1.10/32 is directly connected, Ethernet0/0
C        10.128.2.0/30 is directly connected, Ethernet0/1
L        10.128.2.2/32 is directly connected, Ethernet0/1
D        10.128.2.8/30 [90/1536000] via 10.128.1.2, 02:06:51, Ethernet0/2
D        10.128.3.0/30 [90/1536000] via 10.128.1.9, 02:06:54, Ethernet0/0
                       [90/1536000] via 10.128.1.2, 02:06:54, Ethernet0/2
D        10.128.3.8/30 [90/1536000] via 10.128.1.9, 02:06:54, Ethernet0/0
                       [90/1536000] via 10.128.1.2, 02:06:54, Ethernet0/2
C        10.128.4.0/30 is directly connected, Ethernet0/3
L        10.128.4.1/32 is directly connected, Ethernet0/3
C        10.128.254.16/32 is directly connected, Loopback0
D        10.128.254.32/32 [90/1024640] via 10.128.4.2, 02:06:58, Ethernet0/3
D     192.168.4.0/24 [90/1536000] via 10.128.1.2, 02:06:51, Ethernet0/2
D     192.168.5.0/24 [90/1536000] via 10.128.1.9, 02:06:49, Ethernet0/0
~~~
## R32 получает только маршрут по умолчанию
Маршрут по-умолчанию можно задать через редистрибьюцию:
~~~
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  topology base
   redistribute static
  exit-af-topology
~~~
После выполнения команды на всех нижних маршрутизаторах появятся дефолтные маршруты:
~~~
R17#sh ip route
Gateway of last resort is 10.128.0.1 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/1536000] via 10.128.0.1, 03:01:09, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 15 subnets, 3 masks
~~~
~~~
R16#sh ip route
Gateway of last resort is 10.128.2.1 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/1536000] via 10.128.2.1, 03:01:56, Ethernet0/1
      10.0.0.0/8 is variably subnetted, 16 subnets, 3 masks
~~~
Либо через суммаризацию всех маршрутов на интерфейсе eth0/3 маршрутизатора R16:
~~~
R16(config)#router eigrp SPB
R16(config-router)#address-family ipv4 unicast autonomous-system 2042
R16(config-router-af)#af-interface Ethernet 0/3
R16(config-router-af-interface)#summary-address 0.0.0.0 0.0.0.0
R16(config-router-af-interface)#exit
R16(config-router-af)#exit
~~~
На R32 появится маршрут по-умолчанию:
~~~
R32#sh ip route
Gateway of last resort is 10.128.4.1 to network 0.0.0.0
D*  0.0.0.0/0 [170/2048000] via 10.128.4.1, 03:04:05, Ethernet0/0
~~~
## R16-17 анонсируют только суммарные префиксы.
По-умолчанию автосуммаризация выключена.  
Произведем суммаризацию нескольких подсетей с R16 и R17 смаской /30 в единую подсеть с маской /21 на R18.
Для этого на интерфейсах, смотрящих к провайдеру, выполним команду:
~~~
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  af-interface Ethernet0/1
   summary-address 10.128.0.0 255.255.248.0
  exit-af-interface
  !
~~~
И проверим результат суммаризации на R18:  
~~~
R18#sh ip route eigrp
(Информационные сообщения опущены)
     10.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
D        10.128.0.0/21 is a summary, 00:00:32, Null0
D        10.128.254.32/32 [90/1536640] via 10.128.2.2, 02:49:27, Ethernet0/0
D     192.168.4.0/24 [90/2048000] via 10.128.2.2, 02:49:27, Ethernet0/0
                     [90/2048000] via 10.128.0.2, 02:49:27, Ethernet0/1
D     192.168.5.0/24 [90/2048000] via 10.128.2.2, 02:49:29, Ethernet0/0
                     [90/2048000] via 10.128.0.2, 02:49:29, Ethernet0/1
~~~
Все адреса подсетей 10.128.х.х./30 схлопнулись в 10.128.0.0/21  
