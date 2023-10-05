IPSec over DmVPN

Цель:
Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
Все узлы в офисах в лабораторной работе должны иметь IP связность.
План работы и изменения зафиксированы в документации.
Дополнительно: Для IPSec использовать CA и сертификаты.


Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.

На R18 подняты Tunnel1 и Tunnel12. Будем настраивать IPSec на них.

R15:

crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 2
crypto isakmp key MOSCOWNET address 89.30.0.18
crypto isakmp key MOSCOWNET address 10.0.3.2
crypto isakmp key MOSCOWNET address 10.0.0.2
crypto isakmp key MOSCOWNET address 10.0.1.2
!
!
crypto ipsec transform-set GRE-IPSEC esp-3des esp-sha-hmac 
 mode transport
!
crypto ipsec profile PROTECT-GRE
 set transform-set GRE-IPSEC 
!

interface Tunnel 1
 tunnel protection ipsec profile PROTECT-GRE
 exit
exit

Делаем аналогичную настройку на R18 с указанием точки подключения R15.
Так же настраиваем шифрование для Tunnel2


