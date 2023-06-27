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
#### Аналогичную натсройку необходимо произвести на всех маршрутизаторах зоны AS-2042  
Обратимся к информации по протоколу EIGRP на R32:
~~~
Routing for Networks:
    10.128.4.0/30
    10.128.254.32/32
~~~
Сети маршрутизируются по EIGRP, проверим из доступность c R16:  
~~~

~~~
