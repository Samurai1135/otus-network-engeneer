# IS-IS
`Цель:`
Настроить IS-IS офисе Триада

# Задание:
- `Часть1:` Настроить IS-IS в ISP Триада.
- `Часть2:` R23 и R25 обозначить в зоне 2222.
- `Часть3:` R24 обозначить в зоне 24.
- `Часть4:` R26 обозначить в зоне 26
<br><br>
#### Схема зон Триады
![](https://github.com/Samurai1135/otus-network-engeneer/blob/fb59101fb26dbe6089464be9e75813a90a372cff/Lab-07/screenshots/zones.png)

- ## Настройка IS-IS в ISP Триада.

На каждом из маршрутизаторов провайдера Триада необходимо настроить протокол ISIS из режима конфигурации:
 ~~~
router isis                     //Включаем протокол ISIS
net 49.2222.0000.0000.0023.00   //49-приватный адрес, 2222 - зона, 0000.0000.0023 - id роутера, 00 - селектор
 ~~~
Далее на каждом из интерфейсов, смотрящих в зоны 2222,24 и 26 включить протокол ISIS:

~~~
R25(config-if)#ip router isis
~~~ 
После проведенных настроек нужно убедиться в установлении соседства роутеров.
Приведем результат команды show isis neighbors:
~~~
R23#show isis neighbors

System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.2.0.2        UP    7        R24.02
R25            L1   Et0/1       10.3.0.1        UP    8        R25.01
R25            L2   Et0/1       10.3.0.1        UP    8        R25.01
~~~
~~~
R26#show isis neighbors

System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/0       10.1.0.2        UP    20       R26.01
R25            L2   Et0/2       10.4.0.2        UP    24       R26.02
~~~

Убедившись в установлении соседства необходимо проверить появление путей (маршрутов):
~~~
R26#show ip route isis
          (опущены информационные строки)
      10.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
i L2     10.0.3.0/30 [115/20] via 10.4.0.2, 01:19:23, Ethernet0/2
i L2     10.2.0.0/30 [115/20] via 10.1.0.2, 01:19:23, Ethernet0/0
i L2     10.3.0.0/30 [115/20] via 10.4.0.2, 01:19:23, Ethernet0/2
~~~
~~~
R23#show ip route isis
        (опущены информационные строки)
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks
i L1     10.0.3.0/30 [115/20] via 10.3.0.1, 01:21:32, Ethernet0/1
i L2     10.1.0.0/30 [115/20] via 10.2.0.2, 01:20:40, Ethernet0/2
i L1     10.4.0.0/30 [115/20] via 10.3.0.1, 01:21:32, Ethernet0/1
~~~

Если взглянуть на топологию ISIS на примере R23 - можно обратить внимание на разделение LSDB для L1(соседи внутри зоны) и L2(соседи из других зон) связей:
~~~
R23#show isis topology

IS-IS TID 0 paths to level-1 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R25                  10         R25                  Et0/1       aabb.cc01.9000

IS-IS TID 0 paths to level-2 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R24                  10         R24                  Et0/2       aabb.cc01.8020
R25                  10         R25                  Et0/1       aabb.cc01.9000
R26                  20         R24                  Et0/2       aabb.cc01.8020
                                R25                  Et0/1       aabb.cc01.9000~~~
