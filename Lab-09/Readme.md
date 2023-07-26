# BGP. Основы

`Цель:`
Настроить BGP между автономными системами
Организовать доступность между офисами Москва и С.-Петербург

# Задание:

- `Часть1.` Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
- `Часть2.` Настроить eBGP между провайдерами Киторн и Ламас.
- `Часть3.` Настроить eBGP между Ламас и Триада.
- `Часть4.` Настроить eBGP между офисом С.-Петербург и провайдером Триада.
- `Часть5.` Организовать IP доступность между пограничным роутерами офисов Москва и С.-Петербург.

## Минимальная настройка eBGP между офисами и провайдерами  
В пятой части задания требуется организовать доступность между пограничными роутерами офисов Москва и С.-Петербург 
Проверку доступности будем осуществлять посредствам "пинга" Loopback-адресов пограничных маршрутизаторов.
Настраивая eBGP анонсируем Loopback'и в address-family ipv4

#### Пример настройки R15:
~~~
!
router bgp 1001
 bgp router-id 10.128.254.15
 bgp log-neighbor-changes
 neighbor 10.8.0.1 remote-as 301
 !
 address-family ipv4
  network 10.128.254.15 mask 255.255.255.255
  neighbor 10.8.0.1 activate
 exit-address-family
!
~~~
#### Пример настройки R18:
~~~
!
router bgp 2042
 bgp router-id 10.128.254.18
 bgp log-neighbor-changes
 neighbor 10.0.2.1 remote-as 520
 neighbor 10.0.4.1 remote-as 520
 !
 address-family ipv4
  network 10.128.254.18 mask 255.255.255.255
  neighbor 10.0.2.1 activate
  neighbor 10.0.4.1 activate
 exit-address-family
!
~~~
#### Пример настройки R21 (Ламас)
~~~
!
router bgp 301
 bgp router-id 10.128.254.21
 bgp log-neighbor-changes
 neighbor 10.6.0.1 remote-as 520
 neighbor 10.8.0.2 remote-as 1001
 neighbor 10.9.0.2 remote-as 101
 !
 address-family ipv4
  neighbor 10.6.0.1 activate
  neighbor 10.8.0.2 activate
  neighbor 10.9.0.2 activate
 exit-address-family
!
~~~
## Проверка GBP-связности между пограничными маршрутизаторами R15-R18

Проверим марштруты со стороны Москвы:
~~~
R14#sh ip bgp
BGP table version is 4, local router ID is 10.128.254.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.128.254.14/32 0.0.0.0                  0         32768 i
 *>  10.128.254.18/32 10.7.0.1                               0 101 301 520 2042 i
~~~
~~~
R15#sh ip bgp
BGP table version is 5, local router ID is 10.128.254.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.128.254.15/32 0.0.0.0                  0         32768 i
 *>  10.128.254.18/32 10.8.0.1                               0 301 520 2042 i
~~~
Проверим марштруты со стороны Питера:
~~~
R18#sh ip bgp
BGP table version is 6, local router ID is 10.128.254.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.128.254.14/32 10.0.4.1                               0 520 301 101 1001 i
 *>  10.128.254.15/32 10.0.4.1                               0 520 301 1001 i
 *>  10.128.254.18/32 0.0.0.0                  0         32768 i
~~~
