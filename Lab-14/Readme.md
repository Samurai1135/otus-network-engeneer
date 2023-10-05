IPSec over DmVPN

Цель:
Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги


Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
Все узлы в офисах в лабораторной работе должны иметь IP связность.
План работы и изменения зафиксированы в документации.
Дополнительно: Для IPSec использовать CA и сертификаты.

GRE не обеспечивает шифрование. Если хотим делать шифрование, то делаем GRE поверх IPSec или DMVPN поверх IPSec.

IPsec (сокращение от IP Security) — набор протоколов для обеспечения защиты данных, передаваемых по межсетевому протоколу IP. Позволяет осуществлять подтверждение подлинности (аутентификацию), проверку целостности и/или шифрование IP-пакетов. IPsec также включает в себя протоколы для защищённого обмена ключами в сети Интернет. В основном, применяется для организации VPN-соединений.Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.

На R18 подняты Tunnel1 и Tunnel12. Будем настраивать IPSec на них.

R15:
~~~
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
~~~
Делаем аналогичную настройку на R18 с указанием точки подключения R15.
Так же настраиваем шифрование для Tunnel2
~~~
R15#sh crypto isakmp peers
Peer: 10.0.0.2 Port: 500 Local: 89.20.0.15
 Phase1 id: 10.0.0.2
Peer: 10.0.3.2 Port: 500 Local: 89.20.0.15
 Phase1 id: 10.0.3.2
Peer: 89.30.0.18 Port: 500 Local: 89.20.0.15
 Phase1 id: 89.30.0.18
~~~
~~~
Crypto session current status

Interface: Tunnel1
Session status: UP-ACTIVE
Peer: 89.30.0.18 port 500
  Session ID: 0
  IKEv1 SA: local 89.20.0.15/500 remote 89.30.0.18/500 Active
  Session ID: 0
  IKEv1 SA: local 89.20.0.15/500 remote 89.30.0.18/500 Active
  IPSEC FLOW: permit 47 host 89.20.0.15 host 89.30.0.18
        Active SAs: 4, origin: crypto map

Interface: Tunnel10
Session status: UP-ACTIVE
Peer: 10.0.0.2 port 500
  Session ID: 0
  IKEv1 SA: local 89.20.0.15/500 remote 10.0.0.2/500 Active
  IPSEC FLOW: permit 47 host 89.20.0.15 host 10.0.0.2
        Active SAs: 2, origin: crypto map

Interface: Tunnel10
Session status: UP-ACTIVE
Peer: 10.0.3.2 port 500
  Session ID: 0
  IKEv1 SA: local 89.20.0.15/500 remote 10.0.3.2/500 Active
  IPSEC FLOW: permit 47 host 89.20.0.15 host 10.0.3.2
        Active SAs: 2, origin: crypto map
~~~
~~~
R15#sh crypto isakmp policy

Global IKE policy
Protection suite of priority 10
	encryption algorithm:	AES - Advanced Encryption Standard (128 bit keys).
	hash algorithm:		Secure Hash Standard
	authentication method:	Pre-Shared Key
	Diffie-Hellman group:	#2 (1024 bit)
	lifetime:		86400 seconds, no volume limit
~~~

Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.  
В целом настраиваем аналогично R14, R15, R28, R27, с применением того же профиля (см. коммиты).
R27:  
~~~
R27#sh cry isakmp policy

Global IKE policy
Protection suite of priority 10
	encryption algorithm:	AES - Advanced Encryption Standard (128 bit keys).
	hash algorithm:		Secure Hash Standard
	authentication method:	Pre-Shared Key
	Diffie-Hellman group:	#2 (1024 bit)
	lifetime:		86400 seconds, no volume limit
~~~
R28:  
~~~
R28#sh cry isakmp policy

Global IKE policy
Protection suite of priority 10
	encryption algorithm:	AES - Advanced Encryption Standard (128 bit keys).
	hash algorithm:		Secure Hash Standard
	authentication method:	Pre-Shared Key
	Diffie-Hellman group:	#2 (1024 bit)
	lifetime:		86400 seconds, no volume limit
~~~

Видно, что было применено шифрование.  
Настройка сертификатов  

Главный минус предыдущего подхода – необходимость предварительного распространения пароля. В случае большого и динамичеси меняющегося количество участников DMVPN сети (SPOKE-ов), ручное распространение паролей становится сложной задачей.

Поэтому поднимем свой центр сертификации на HUB-роутере, при помощи которого будем выписывать сертификаты и выдавать их участникам сети.

Далее, по выданным сертификатам, будет производиться авторизация участников сети.

План:

Настроить центр сертификации (CA)
Послать запросы на выдачу сертификатов и выписать сертификаты
Настроить на роутерах работу IPSec через сертификаты
Проверить, что все работает
Параметры сертификатов:
~~~
subject-name CN=R15,OU=Moscow,O=moscow.net,C=RU
subject-name CN=R27,OU=Labintagi,O=moscow.net,C=RU
subject-name CN=R28,OU=Chokhurdakh,O=moscow.net,C=RU
~~~

Настройка CA  
Роль CA будет выполнять роутер R15. Выполним:  
~~~
ip domain-lookup
ip domain-name moscow.local

ip http server
crypto key generate rsa general-keys label CA exportable modulus 2048
crypto pki server CA
no shutdown
password:MOSCOWNET
~~~

Настройка клиента и запрос сертификата
На всех маршрутизаторах, участвующих в VPN, выполнить (отличие в subject-name):
~~~
R28(config)#ip domain-lookup
R28(config)#ip domain-name moscow.net

R28(config)#crypto key generate rsa label VPN modulus 2048
~~~

Настройка trust point:
~~~
R28(config)#crypto pki trustpoint VPN
R28(ca-trustpoint)#enrollment url http://89.20.0.15
R28(ca-trustpoint)#subject-name CN=R28,OU=Chokhurdakh,O=moscow.net,C=RU
R28(ca-trustpoint)#rsakeypair VPN
R28(ca-trustpoint)#revocation-check none
R28(ca-trustpoint)#exit
~~~
Запросить сертификат CA и проверить его, запросить сертификат для себя:
~~~
R28(config)#
*Oct  5 09:55:11.213: CRYPTO_PKI:  Certificate Request Fingerprint MD5: 542D9F74 8737A93E 6F52B6D3 790322F3
*Oct  5 09:55:11.213: CRYPTO_PKI:  Certificate Request Fingerprint SHA1: 12A94DF1 A495B4BE 7C3C3D53 CD4D27BF 562355AF

% Do you accept this certificate? [yes/no]: yes
Trustpoint CA certificate accepted.
~~~
~~~
R28(config)#crypto pki enroll VPN
~~~
После отправки запрос можно посмотреть на CA:
~~~
R15#show crypto pki server CA request
***
Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
1      pending    8EF30F1B6264C558C44D58B768DAAEC6 hostname=R28.moscow.net,cn=R28,ou=Chokhurdakh,o=moscow.net,c=RU```
~~~
Подтвердить выпуск:
~~~
R15#crypto pki server CA grant 1
~~~
или
~~~
crypto pki server CA grant all
~~~
После подтверждения запроса статус pending меняется на granted.
~~~
R15#show crypto pki server CA request
Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
1      granted    8EF30F1B6264C558C44D58B768DAAEC6 hostname=R28.moscow.net,cn=R28,ou=Chokhurdakh,o=moscow.net,c=RU```
~~~
C запрашивающего маршрутизатора можно посмотреть полученный сертификат:
~~~
R28#sh cry pki certificates
Certificate
  Status: Available
  Certificate Serial Number (hex): 02
  Certificate Usage: General Purpose
  Issuer:
    cn=CA
  Subject:
    Name: R28.moscow.net
    Serial Number: 67109312
    serialNumber=67109312+hostname=R28.moscow.net
    cn=R28
    ou=Chokurdakh
    o=moscow.local
    c=RU
  Validity Date:
    start date: 16:56:09 UTC Oct 3 2023
    end   date: 16:56:09 UTC Oct 2 2024
  Associated Trustpoints: VPN
  Storage: nvram:CA#2.cer

CA Certificate
  Status: Available
  Certificate Serial Number (hex): 01
  Certificate Usage: Signature
  Issuer:
    cn=CA
  Subject:
    cn=CA
  Validity Date:
    start date: 16:49:09 UTC Oct 3 2023
    end   date: 16:49:09 UTC Oct 2 2026
  Associated Trustpoints: VPN
  Storage: nvram:CA#1CA.cer
~~~
Настройка IPSec
В целом далее настройки IPSec и применение профиля аналогичны описанным выше, но тип аутентификации меняется с authentication pre-share на authentication rsa-sig.

После применения настроек и выписывания сертификатов между (для примера R28 и R15) видно, что пиры теперь выглядят по другому:  
