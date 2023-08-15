# BGP. Фильтрация
`Цель:`
Настроить фильтрацию для офисе Москва
Настроить фильтрацию для офисе С.-Петербург

# Задание:
- `Часть1.` Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
- `Часть2.` Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
- `Часть3.` Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
- `Часть4.` Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
- `Часть5.` Все сети в лабораторной работе должны иметь IP связность.
![](https://github.com/Samurai1135/otus-network-engeneer/blob/95263ab0946111740c927f2bc6a498d6676490a6/Lab-11/Pics/Map.png)
## Устранение транзитного трафика
Существует два основных способа фильтрации транзитного трафика:
  * когда мы не являемся транзитной зоной АС (фильтрация по as-path)
  * когда мы являемся транзитной зоной АС (фильтрация по набору префиксов)

## Устраняем транзитный трафик в Москве

По сути у нас нет транзитного трафика, т.к. офис Москва является конечной АС. Тем не менее настроим фильтрацию транзитного трафика на случай появления нового офиса, подключенного через офис Москва:
~~~
R15(config)#ip as-path access-list 1 permit ^$
R15(config)#ip as-path access-list 1 deny .*
R15(config)#router bgp 1001
R15(config-router)#neighbor 10.8.0.1 filter-list 1 out
~~~
## Устраним транзитный трафик в Санкт-Петербурге
Тут так же нет транзитного трафика. Сделаем разрешение на трансляцию только клиентских сетей:
~~~
R18(config)#ip prefix-list NO-TRANSIT seq 5 permit 192.168.4.0/24
R18(config)#ip prefix-list NO-TRANSIT seq 10 permit 192.168.5.0/24
R18(config)#router bgp 2042
R18(config-router)#neighbor 10.0.4.1 prefix-list NO-TRANSIT out
R18(config-router)#neighbor 10.0.2.1 prefix-list NO-TRANSIT out
~~~
## Передадим маршрут по-умолчанию от Киторн по направлению к  Москве
До фильтрации:
~~~
R14#sh ip bgp
BGP table version is 16, local router ID is 10.128.254.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.128.254.14/32 0.0.0.0                  0         32768 i
 r>i 10.128.254.15/32 10.128.254.15            0    200      0 i
 *>i 10.128.254.21/32 10.128.254.15            0    200      0 301 i
 *                    10.7.0.1                               0 101 301 i
 *>i 10.128.254.22/32 10.128.254.15            0    200      0 301 101 i
 *                    10.7.0.1                 0             0 101 i
 * i 192.168.6.0/23   10.128.254.15            0    200      0 i
 *>                   10.128.20.2              0         32768 i

~~~

Прокинем только дефолтный маршрут, а остальные запретим:
~~~
R22(config)#ip prefix-list PL-DEFAULT deny 0.0.0.0/0
R22(config)#router bgp 101
R22(config-router)#neighbor 10.7.0.2 default-originate
R22(config-router)#neighbor 10.7.0.2 prefix-list PL-DEFAULT out
R22(config-router)#neighbor 10.128.254.14 default-originate
R22(config-router)#neighbor 10.128.254.14 prefix-list PL-DEFAULT out
~~~
После фильтрации и передачи дефолта видим только один маршрут 0.0.0.0. из зоны AS101
~~~
R14#sh ip bgp
BGP table version is 17, local router ID is 10.128.254.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          10.7.0.1                               0 101 i
 *>  10.128.254.14/32 0.0.0.0                  0         32768 i
 r>i 10.128.254.15/32 10.128.254.15            0    200      0 i
 *>i 10.128.254.21/32 10.128.254.15            0    200      0 301 i
 *>i 10.128.254.22/32 10.128.254.15            0    200      0 301 101 i
 * i 192.168.6.0/23   10.128.254.15            0    200      0 i
 *>                   10.128.20.2              0         32768 i
R14#sh ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.7.0.1 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 10.7.0.1, 00:01:34
      10.0.0.0/8 is variably subnetted, 24 subnets, 3 masks
B        10.128.254.21/32 [200/0] via 10.128.254.15, 00:38:03
B        10.128.254.22/32 [200/0] via 10.128.254.15, 00:37:40
~~~
## Теперь передадим дефолт от Ламас в сторону Москвы

Так же передаем маршрут от R21 в сторону R15
В маршрутах на R15 есть сети Киторн'а:
~~~
R15#sh ip bgp
BGP table version is 18, local router ID is 10.128.254.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          10.128.254.14            0    100      0 101 i
 r>i 10.128.254.14/32 10.128.254.14            0    100      0 i
 *>  10.128.254.15/32 0.0.0.0                  0         32768 i
 *>  10.128.254.21/32 10.8.0.1                 0             0 301 i
 *>  10.128.254.22/32 10.8.0.1                               0 301 101 i
 *>  192.168.4.0      10.8.0.1                               0 301 520 2042 i
 *>  192.168.5.0      10.8.0.1                               0 301 520 2042 i
 * i 192.168.6.0/23   10.128.254.14            0    100      0 i
 *>                   10.128.22.2              0         32768 i
~~~
~~~
R15#sh ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.8.0.1 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 23 subnets, 3 masks
B        10.128.254.21/32 [20/0] via 10.8.0.1, 01:05:53
B        10.128.254.22/32 [20/0] via 10.8.0.1, 01:05:29
B     192.168.4.0/24 [20/0] via 10.8.0.1, 00:04:50
B     192.168.5.0/24 [20/0] via 10.8.0.1, 00:04:19
~~~
Теперь сделаем так, чтобы R21 отдавал дефолт на R15 и фильтровал все префиксы, отличные от AS2024 (Питер)
~~~
R21(config)#ip as-path access-list 1 permit _2042$
R21(config)#ip as-path access-list 1 deny .*
R21(config)#router bgp 301
R21(config-router)#neighbor 10.8.0.2 default-originate
R21(config-router)#neighbor 10.8.0.2 filter-list 1 out
R21(config-router)#neighbor 10.128.254.15 default-originate
R21(config-router)#neighbor 10.128.254.15 filter-list 1 out
~~~
Теперь мы видим, что на R15 поступают только маршруты до пользовательских сетей:
~~~
R15#sh ip route bgp

Gateway of last resort is 10.8.0.1 to network 0.0.0.0

B     192.168.4.0/24 [20/0] via 10.8.0.1, 00:10:05
B     192.168.5.0/24 [20/0] via 10.8.0.1, 00:09:34
~~~
~~~
R15#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          10.8.0.1                               0 301 i
 r>i 10.128.254.14/32 10.128.254.14            0    100      0 i
 *>  10.128.254.15/32 0.0.0.0                  0         32768 i
 *>  192.168.4.0      10.8.0.1                               0 301 520 2042 i
 *>  192.168.5.0      10.8.0.1                               0 301 520 2042 i
 * i 192.168.6.0/23   10.128.254.14            0    100      0 i
 *>                   10.128.22.2              0         32768 i
~~~
## Настроим фильтры на входящие анонсы в Москве
Для начала нужно снять фильтры с Киторна и Ламаса чтоб пустить тестовый префикс Киторна.
~~~
R21(config)#router bgp 301
R21(config-router)#no neighbor 10.8.0.2 filter-list 1 out
R21(config-router)#no neighbor 10.128.254.15 filter-list 1 out
~~~
Проверим, что префикс Киторона появился (10.128.254.22)
~~~
R15#sh ip bgp


     Network          Next Hop            Metric LocPrf Weight Path
 r>  0.0.0.0          10.8.0.1                               0 301 i
 r>i 10.128.254.14/32 10.128.254.14            0    100      0 i
 *>  10.128.254.15/32 0.0.0.0                  0         32768 i
 *>  10.128.254.21/32 10.8.0.1                 0             0 301 i
 *>  10.128.254.22/32 10.8.0.1                               0 301 101 i
 *>  192.168.4.0      10.8.0.1                               0 301 520 2042 i
 *>  192.168.5.0      10.8.0.1                               0 301 520 2042 i
 * i 192.168.6.0/23   10.128.254.14            0    100      0 i
 *>                   10.128.22.2              0         32768 i
~~~
Теперь на R15 настрим дефолтный маршрут + маршруты из Питера
~~~
R15(config)#ip as-path access-list 2 permit _2042$
R15(config)#ip as-path access-list 2 deny .*
R15(config)#router bgp 1001
R15(config-router)#neighbor 10.128.254.21 filter-list 2 in
~~~
Делаем проверку:
~~~
R15#sh ip bgp

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          10.128.254.14            0    100      0 101 i
 r>i 10.128.254.14/32 10.128.254.14            0    100      0 i
 *>  10.128.254.15/32 0.0.0.0                  0         32768 i
 *>  192.168.4.0      10.8.0.1                               0 301 520 2042 i
 *>  192.168.5.0      10.8.0.1                               0 301 520 2042 i
 *>i 192.168.6.0/23   10.128.254.14            0    100      0 i
~~~
Маршрут до Киторна пропал! Проведем аналогичную настройку для R14 (будем получать только дефолт)
~~~
R14(config)#ip prefix-list NO-FULL s
R14(config)#ip prefix-list NO-FULL seq 5 p
R14(config)#ip prefix-list NO-FULL seq 5 p
R14(config)#ip prefix-list NO-FULL seq 5 permit 0.0.0.0/0
R14(config)#ip prefix-list NO-FULL seq 10 de
R14(config)#ip prefix-list NO-FULL seq 10 deny 0.0.0.0/0 ge 1
R14(config)#router bg
R14(config)#router bgp 1001
R14(config-router)#nei
R14(config-router)#neighbor 10.7.0.1 pr
R14(config-router)#neighbor 10.7.0.1 prefix-list NO-FULL in
~~~
~~~
R14#sh ip bgp
BGP table version is 7, local router ID is 10.128.254.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          10.7.0.1                               0 101 i
 *>  10.128.254.14/32 0.0.0.0                  0         32768 i
 r>i 10.128.254.15/32 10.128.254.15            0    200      0 i
 *>i 192.168.4.0      10.128.254.15            0    200      0 301 520 2042 i
 *>i 192.168.5.0      10.128.254.15            0    200      0 301 520 2042 i
 *>  192.168.6.0/23   10.128.20.2              0         32768 i
~~~

## Проверим работу наших сетей
Пинг VPC8->VPC1
~~~
VPCS> ping 192.168.6.1

84 bytes from 192.168.6.1 icmp_seq=1 ttl=56 time=1.811 ms
84 bytes from 192.168.6.1 icmp_seq=2 ttl=56 time=1.693 ms
84 bytes from 192.168.6.1 icmp_seq=3 ttl=56 time=1.566 ms
84 bytes from 192.168.6.1 icmp_seq=4 ttl=56 time=1.582 ms
84 bytes from 192.168.6.1 icmp_seq=5 ttl=56 time=1.570 ms
~~~
Пинг VPC8->VPC7
~~~
VPCS> ping 192.168.7.1

84 bytes from 192.168.7.1 icmp_seq=1 ttl=56 time=3.131 ms
84 bytes from 192.168.7.1 icmp_seq=2 ttl=56 time=1.624 ms
84 bytes from 192.168.7.1 icmp_seq=3 ttl=56 time=1.668 ms
84 bytes from 192.168.7.1 icmp_seq=4 ttl=56 time=1.643 ms
84 bytes from 192.168.7.1 icmp_seq=5 ttl=56 time=1.570 ms
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
