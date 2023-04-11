
## Настройка STP

`Цель:`
Создание сети и настройка основных параметров устройства.
Выбор корневого моста.
Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.
Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

## Описание/Пошаговая инструкция выполнения домашнего задания:

- `Часть 1.` Настройка топологии сети и основных параметров коммутаторов и маршрутизаторов.
- `Часть 2.` Для каждого экземпляра протокола spanning-tree (коммутируемая сеть LAN или широковещательный домен) определить коммутатор, выделенный в качестве корневого моста (Root Bridge).
- `Часть 3.` Отразить результаты наблюдения за процессом выбора протоколом STP порта, исходя из стоимости портов.
- `Часть 4.` Активация избыточных путей до каждого из коммутаторов, чтобы просмотреть, каким образом протокол STP выбирает порт с учетом приоритета портов.

<br><br>
- ## Часть 1. Топология сети и основные параметры сетевых устройств

![](https://github.com/Samurai1135/otus-network-engeneer/blob/e9c80de0cbc62366fff3cf5a6a07605fa480f33a/Lab-02/NetworkScheme/Topology.png)
<br>

#### Используемое оборудование и конфигурация портов:
<br>

| Наименование       | Обозначение на схеме |  Порт подключения и    IP адресация |
| :----------------- | :------------------- | :---------------------------------- |
|Маршрутизатор/роутер| Router1              |  eth0/0.20 -- IP:10.128.20.254/24  |  
|                    |                      |  eth0/0.30 --  IP:10.128.30.254/24 |
|                    |                      |  eth0/0.254 --  IP:10.128.254.254/24 |
|Коммутатор/Свитч 1. | Switch1              |  eth0/0-2 - trunk, eth1/1 - trunk  |
|Коммутатор/Свитч 2. | Switch2              |  eth0/0-1 - trunk, eth1/0 - access|
|Коммутатор/Свитч 3. | Switch3              |  eth0/0,1,3 - trunk, eth0/2 - access  |
|Коммутатор/Свитч 4. | Switch4              |  eth0/0,1 - trunk, eth0/3 и eth1/0 - access|
|Коммутатор/Свитч 5. | Switch5              |  eth0/0-2 и eht1/1 - trunk, eth0/3 и eth1/0 - access  |
|Пользовательский ПК4| VPC4                 |  eth0 -- IP:10.128.30.5/24         |
|Пользовательский ПК5| PC6                 |  eth0 -- IP:10.128.254.4/24         |
|Пользовательский ПК6| PC7                 |  eth0 -- IP:10.128.20.4/24         | 
|Пользовательский ПК7| PC8                 |  eth0 -- IP:10.128.20.3/24         |
|Пользовательский ПК6| PC9                 |  eth0 -- IP:10.128.30.2/24         | 
|Пользовательский ПК7| PC10                 |  eth0 -- IP:10.128.30.3/24         |

<br>
#### Параметры сетевых устройств 
Пример конфигурации маршрутизатора ![Router-1](https://github.com/Samurai1135/otus-network-engeneer/blob/4e1c5e6eeeb28e85e584e8798aa26700819cb008/Lab-02/Configs/Router-1)
<br>
<br>

~~~
interface Ethernet0/0
 no shutdown
 no ip address
!
interface Ethernet0/0.20
 no shutdown
 encapsulation dot1Q 20
 ip address 10.128.20.254 255.255.255.0
!
interface Ethernet0/0.30
 no shutdown
 encapsulation dot1Q 30
 ip address 10.128.30.254 255.255.255.0
!
interface Ethernet0/0.254
 no shutdown
 encapsulation dot1Q 254
 ip address 10.128.254.254 255.255.255.0
!
interface Ethernet0/1
 no shutdown
 no ip address
 shutdown
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
no ip http server
no ip http secure-server
!
control-plane
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
end
~~~

