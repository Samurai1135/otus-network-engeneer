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

В офисе Чокурдах расположено два объекта в двух разных VLAN'ах: VPC30 и VPC31
Маршрутизатор  R28 имеет два линка от провайдера, поэтому целесообразно разделить трафик VLAN2(VPC30) и VLAN3(VPС31) между ними.
Для этого создадим на R28 два Access-List'а с разрешением доступа для наших машин VPC:
 ~~~
ip access-list extended VPC30 //в нашем случае можно использовать и standart access-list (только для IPv4)
 permit ip host 192.168.2.1 any
ip access-list extended VPC31
 permit ip host 192.168.3.1 any
 ~~~
Для распределения трафика между двумя линками настроим карту маршрута:
~~~
!
route-map BALLANCE permit 10 
 match ip address VPC30
 set ip next-hop 10.0.0.1 //в сторону R26
!
route-map BALLANCE permit 20
 match ip address VPC31
 set ip next-hop 10.0.1.1 //в сторону R25
!
~~~
Таким образом мы получили "пути" для трафика из VLAN2 на маршрутизатор R26, а из VLAN3 на маршрутизатор R25

#### !!! Важно отметить, что для вступления в силу route-map на нужных нам интерфейсах  (в нашем случае eth0/2.2 и eth0/2.3) необходимо выполнить команду:
~~~
ip policy route-map BALLANCE
~~~

- ## Часть 3. Настройка отслеживания линка через технологию IP SLA.(только для IPv4)

Самым оптимальным решением является проверка состяния линков от провайдера командой PING. 
Создадим задание для маршрутизатора R28, которое будет "заставлять" маршрутизатор проверять доступность каналов каждые 5 секунд:

~~~
ip sla 1 //обозначаем правило SLA
 icmp-echo 10.0.0.1 source-ip 10.0.0.2 //указываем выполняемую команду  ping  и порт, с которого команда должна выполняться
 frequency 5 //задаем периодичность проверки доступности
ip sla schedule 1 life forever start-time now //выставляем тайминги работы сервиса (работать всегда, запуститься немедленно)
ip sla 2
 icmp-echo 10.0.1.1 source-ip 10.0.1.2
 frequency 5
ip sla schedule 2 life forever start-time now
!
~~~
Проверим работу нашего задания: <br>
![](https://github.com/Samurai1135/otus-network-engeneer/blob/4de5fb53ce74a350d14e038d26c05e56d9d30a0c/Lab-05/Screenshots/stat.png)

Как видим - сервис работает.

## Настроим переключение трафика между линками в случае недоступности одного из них:
~~~
route-map BALLANCE permit 10
 match ip address VPC30
 set ip next-hop verify-availability 10.0.0.1 10 track 1 //в случае доступности исользуем этот next-hop. 10 - приоритет
 set ip next-hop verify-availability 10.0.1.1 20 track 2 //в случае недоступности первого next-hop используем этот
 set ip next-hop 10.0.0.1
!
route-map BALLANCE permit 20
 match ip address VPC31
 set ip next-hop verify-availability 10.0.1.1 10 track 2
 set ip next-hop verify-availability 10.0.0.1 20 track 1
 set ip next-hop 10.0.1.1
~~~
~~~
!
track 1 ip sla 1 reachability //сопоставляем track 1 выполняющемуся правилу sla 1
!
track 2 ip sla 2 reachability
~~~

- ## Часть 4. Настройка для офиса Лабытнанги маршрута по-умолчанию.

Ближайший "сосед" офиса Лабытнанги - R25 (eth 0/1)
Зададим маршрут для R27:
~~~
ip route 0.0.0.0 0.0.0.0 10.0.3.1
~~~
## Теперь сделаем проверку переключения трафика на R28.
Мы априори завели на каждом активном сетевом устройстве адрес Loopback-интерфейса. Поскольку подсети Лабытнанги не оговариваются - попробуем пропинговать его адрес Loopback 100.0.1.11
Для работы сети нам нужно задать несколько маршрутов на роутерах R28,R25 и R26.

R28:
~~~
ip route 100.0.1.11 255.255.255.255 10.0.0.1 name 2Labyt2 //маршрут до Лабытнанги через "нижнего" провайдера (R28-R26-R25-R27)
ip route 100.0.1.11 255.255.255.255 10.0.1.1 name 2Labyt1 //маршрут до Лабытнанги через "верхнего" провайдера (R28-R25-R27)
~~~
R26:
~~~
ip route 100.0.1.11 255.255.255.255 10.4.0.2 name 2Labyt //маршрут до Лабытнанги
ip route 192.168.2.0 255.255.254.0 10.0.0.2
~~~
R25:
~~~
ip route 100.0.1.11 255.255.255.255 10.0.3.2 name 2Labyt
ip route 192.168.2.0 255.255.254.0 10.0.1.2 name 2Chok
ip route 192.168.2.0 255.255.254.0 10.4.0.1 200 name 2Chok-Triada
~~~
Проверим трассировку от VPC30 и VPC31 до Loopback-адреса офиса Лабытнанги:
<br>
VPC30 - Лабытнанги:<br>
![](https://github.com/Samurai1135/otus-network-engeneer/blob/5dd284f103f2c40a526c2d3b5afc7fef8de54268/Lab-05/Screenshots/VPC30%20trace1.png)<br>
VPC31 - Лабытнанги:<br>
![](https://github.com/Samurai1135/otus-network-engeneer/blob/5dd284f103f2c40a526c2d3b5afc7fef8de54268/Lab-05/Screenshots/VPC31%20trace1.png)
<br>
Мы видим, что трафик VLAN2 проходит по одному маршруту через R26, а трафик VLAN3 - через R25.
Пробуем "погасить" линк eth0/1 на R26, ждем 5 секунд и повторно произведем трассировку до Лабытнанги с наших VPC. <br>
VPC30 - Лабытнанги:<br>
![](https://github.com/Samurai1135/otus-network-engeneer/blob/ddb1446049702c0afd19497e5eb5c15b7c34866e/Lab-05/Screenshots/VPC30%20trace2.png) <br>
VPC31 - Лабытнанги:<br>
![](https://github.com/Samurai1135/otus-network-engeneer/blob/ddb1446049702c0afd19497e5eb5c15b7c34866e/Lab-05/Screenshots/VPC31trace2.png)<br>
Из скриншотов прекрасно видно, что трафик VLAN2 изменил маршрут согласно ROUTE-MAP и идет напрямую к R25.
