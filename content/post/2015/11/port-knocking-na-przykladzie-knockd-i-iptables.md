---
author: Morfik
categories:
- Linux
date: "2015-11-05T00:26:27Z"
date_gmt: 2015-11-04 22:26:27 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
GHissueID: 249
title: Port knocking na przykładzie knockd i iptables
---

[Port knocking](https://pl.wikipedia.org/wiki/Port_knocking) zezwala na zdalny dostęp do usług,
które są chronione za pomocą zapory sieciowej. Generalnie rzecz biorąc, iptables ma blokować
jakikolwiek ruch na porcie, na którym nasłuchuje jakaś demon. Oczywiście tym sposobem żaden klient
nie mógłby nawiązać połączenia z serwerem i tutaj właśnie znajduje zastosowanie `knockd` , który
jest w stanie dodawać dynamicznie odpowiednie reguły do filtra iptables. Nawiązywanie połączenia
trwa z reguły bardzo szybko. Po tym jak klient uzyskał dostęp do serwera, te dodane wcześniej reguły
są usuwane blokując tym samym wszelkie nowe próby połączenia ale nie odcinając jednocześnie
ustanowionych już połączeń. Ten wpis ma jedynie na celu zaprezentowanie narzędzia `knockd` .
Niemniej jednak, jest ono już przestarzałe i powinno się od niego odchodzić na rzecz [Single Packet
Authorization, czyli alternatywnego port
knocking'u](/post/port-knocking-i-single-packet-authorization/).

<!--more-->
## Konfiguracja demona knockd

Załóżmy, że mamy zdalny dostęp do serwera za pomocą shell'a. Port, który zostałby tradycyjnie
odblokowany na zaporze to 22. Musimy ten wyjątek usunąć ze [skryptu
firewall'a](/post/firewall-na-linuxowe-maszyny-klienckie/). Następnie, by móc
korzystać z dobrodziejstw jakie oferuje port knocking, na serwerze instalujemy pakiet `knockd` .
Mając już ten pakiet w systemie, przechodzimy do edycji pliku `/etc/knockd.conf` . Definiujemy w
nim te poniższe bloki:

    [options]
          UseSyslog

    [OpenSSH]
          sequence    = 2222:tcp,3333:tcp,4444:tcp
          seq_timeout = 5
          command     = /sbin/iptables -A approved -s %IP% -j ACCEPT
          tcpflags    = syn

    [CloseSSH]
          sequence    = 4444:tcp,3333:tcp,2222:tcp
          seq_timeout = 5
          command     = /sbin/iptables -D approved -s %IP% -j ACCEPT
          tcpflags    = syn

Mamy tutaj dwie sekwencje portów. Pierwsza z nich doda adres źródłowy klienta do wyjątków w
iptables. W ten sposób klient będzie w stanie zainicjować połączenie z demonem SSH. Natomiast druga
sekwencja portów usunie ten adres z filtra iptables. Oczywiście możemy tworzyć bardziej
skomplikowane reguły i nie musimy się ograniczać jedynie do adresu źródłowego. W tym przypadku
wymagane jest ręczne wpisanie obu sekwencji, czyli po tym jak podamy prawidłową sekwencję portów,
dostęp do serwera będzie możliwy tak długo aż wprowadzimy drugą sekwencję portów. Możemy ten
mechanizm nieco zautomatyzować, tak by ustawić tylko określony przedział czasu, przez który okno
będzie otwarte. W takim przypadku musimy przepisać ten powyższy kod do poniższej postaci:

    [options]
          UseSyslog

    [OpenCloseSSH]
          sequence      = 2222:tcp,3333:tcp,4444:tcp
          seq_timeout   = 5
          tcpflags      = syn
          start_command = /sbin/iptables -A approved -s %IP% -j ACCEPT
          cmd_timeout   = 15
          stop_command  = /sbin/iptables -D approved -s %IP% -j ACCEPT

Parametr `seq_timeout` odpowiada za czas pukania w porty. W tym przypadku mamy 5 sekund na
wprowadzenie poprawnej sekwencji. Z kolei `cmd_timeout` jest to czas, po przekroczeniu którego
zostanie wykonane polecenie określone w `stop_command` . W `start_command` i `stop_command` możemy
także wywoływać skrypty.

## Zdalne otwieranie portów

Mając skonfigurowanego demona `knockd` uruchamiamy go na serwerze przy pomocy poniższego polecenia:

    # /usr/sbin/knockd -i eth0 -v -D --config /etc/knockd.conf

Możemy teraz spróbować zapukać w porty i przetestować ten cały port knocking. Przechodzimy zatem na
maszynę kliencką i wpisujemy w terminal to poniższe polecenie:

    # knock -v 192.168.1.150 2222:tcp 3333:tcp 4444:tcp

    hitting tcp 192.168.1.150:2222
    hitting tcp 192.168.1.150:3333
    hitting tcp 192.168.1.150:4444

Jeśli pukanie zakończyło się powodzeniem, do łańcucha `approved` powinna zostać dodana reguła z
adresem IP maszyny, która przesłała poprawną sekwencję portów. W przypadku automatycznego usuwania
reguł, po chwili ta reguła powinna zostać usunięta.

## Zagrożenie bezpieczeństwa jakie niesie knockd

Korzystanie z port knocking'u w wykonaniu `knockd` ma jedną ale za to ogromną wadę. Mianowicie,
można podsłuchać sekwencję portów. A jeśli ktoś będzie ją znał, to jego adres IP zostanie dopisany
jako zaufany do filtra iptables. Możemy się na tę okoliczność zabezpieczyć wykorzystując mechanizm
ochrony jaki oferują [pliki hosts.allow i
hosts.deny](/post/pliki-hosts-allow-i-hosts-deny/) . W takim przypadku, jeśli
osobie próbującej uzyskać dostęp do serwera udałoby się obejść firewall i złamać hasło do konta
shell'owego, to podczas łączenia do demona SSH zobaczy jedynie ten poniższy komunikat:

    $ ssh morfik@192.168.1.150
    ssh: Exited: Error connecting: Connection refused
