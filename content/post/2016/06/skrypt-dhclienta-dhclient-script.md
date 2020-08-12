---
author: Morfik
categories:
- Linux
date: "2016-06-14T16:26:15Z"
date_gmt: 2016-06-14 14:26:15 +0200
published: true
status: publish
tags:
- dhcp
- dhclient
title: Skrypt dhclient'a (dhclient script)
---

Jak wiele osób zapewne wie, szereg dystrybucji linux'a wykorzystuje `dhclient` w celu pobrania
sieciowej konfiguracji hosta za sprawą protokółu DHCP. W zasadzie cała konfiguracja tego narzędzia
sprowadza się do określenia szeregu opcji w pliku `/etc/dhcp/dhclient.conf` . W debianie nawet nie
musimy dotykać tego pliku, by wszystko działało nam jak trzeba. Niemniej jednak, czasem konfiguracja
interfejsów sieciowych może wymagać od nas dodatkowych zabiegów. W celu ułatwienia życia adminom
dodano obsługę skryptów shell'owych
([dhclient-script](http://manpages.ubuntu.com/manpages/xenial/en/man8/dhclient-script.8.html)). Taki
skrypt jesteśmy w stanie wywołać w zależności od zaistniałych zdarzeń protokołu DHCP. W tym wpisie
zostanie pokazane w jaki sposób te skrypty możemy utworzyć i wykorzystać.

<!--more-->
## Tryb debugowania w dhclient

Przede wszystkim, musimy zaznajomić się ze zdarzeniami, które mają miejsce w protokole DHCP. Ta
wiedza będzie nam potrzebna do budowania samego skryptu. Dlatego też na samym początku zaleca się
włączenie trybu debugowania w `dhclient` , tak by wszystkie zdarzenia protokołu DHCP były
rejestrowane. W taki sposób łatwiej zorientujemy się co tak naprawdę zachodzi podczas konfiguracji
hosta w sieci przy pomocy tego protokołu. By taki tryb debugowania aktywować, musimy edytować plik
`/etc/dhcp/debug` i dostosować w nim tę poniższą linijkę:

    RUN="yes"

W ten sposób za każdym razem, gdy dojdzie do jakiegoś zdarzenia przy konfiguracji hosta, np.
pobranie lease DHCP, stosowne informacje zostaną zalogowane w pliku `/tmp/dhclient-script.debug` .
Poniżej jest wersja skrócona tego pliku:

    Mon Jun 13 16:05:27 CEST 2016: entering /etc/dhcp/dhclient-enter-hooks.d, dumping variables.
    reason='PREINIT'
    interface='bond0'
    --------------------------
    Mon Jun 13 16:05:27 CEST 2016: entering /etc/dhcp/dhclient-exit-hooks.d, dumping variables.
    reason='PREINIT'
    interface='bond0'
    --------------------------
    Mon Jun 13 16:05:27 CEST 2016: entering /etc/dhcp/dhclient-enter-hooks.d, dumping variables.
    reason='BOUND'
    interface='bond0'
    new_ip_address='192.168.1.150'
    new_network_number='192.168.1.0'
    ...
    old_ip_address='192.168.1.150'
    old_network_number='192.168.1.0'
    ...
    --------------------------
    Mon Jun 13 16:05:28 CEST 2016: entering /etc/dhcp/dhclient-exit-hooks.d, dumping variables.
    reason='BOUND'
    interface='bond0'
    new_ip_address='192.168.1.150'
    new_network_number='192.168.1.0'
    ...
    old_ip_address='192.168.1.150'
    old_network_number='192.168.1.0'
    ...
    --------------------------

Mamy tutaj szereg bardzo przydatnych informacji. Cały ten powyższy log jest z pojedynczego zdarzenia
uzyskania lease DHCP. Mamy tutaj dwie akcje, które następują po sobie. Każda z nich ma uwzględniony
czas oraz, co ważniejsze, wpis z `entering` . Widzimy, że mamy tam dwa katalogi
`/etc/dhcp/dhclient-enter-hooks.d/` oraz `/etc/dhcp/dhclient-exit-hooks.d/` , które są przeszukiwane
przez `dhclient` . To w tych folderach znajdować się będą skrypty ale o tym będzie w dalszej części
wpisu.

W tym powyższym logu mamy jeszcze szereg informacji, które przydadzą nam się w pisaniu skryptów.
Najważniejsza rzecz to `reason` . To na podstawie wartości w tej zmiennej, `dhclient` wie czy dany
skrypt ma zostać wywołany, np. przy pozyskiwaniu lease, czy też przy odnowieniu lease. Jest szereg
wartości, które mogą zostać ustawione w zmiennej `reason` . Zaliczają się do nich te następujące:
`MEDIUM` , `PREINIT` , `BOUND` , `RENEW` , `REBIND` , `REBOOT` , `EXPIRE` , `FAIL` , `STOP` ,
`RELEASE` , `NBI` i `TIMEOUT` . Wyżej widzieliśmy dwa z tych powodów. Pierwszym z nich był `PREINIT`
, który jest zawsze wywoływany przy pozyskiwaniu lease po raz pierwszy z serwera DHCP. Drugim z
kolei był `BOUND` i tu `dhclient` odczytał adresację lease w celu zaaplikowania jej w konfiguracji
hosta.

W przypadku `BOUND` widzimy też uwzględnionych szereg zmiennych. Wszystkie te z nich mające prefiks
`old_*` zawierają w sobie starą konfigurację hosta. Z kolei wpisy z prefiksem `new_*` mają wartości
uzyskane od serwera DHCP. Poniżej jest przykład tych zmiennych:

    new_ip_address='192.168.1.150'
    new_host_name='morfikownia'
    new_network_number='192.168.1.0'
    new_subnet_mask='255.255.255.0'
    new_broadcast_address='192.168.1.255'
    new_routers='192.168.1.1'
    new_domain_name='mhouse'
    new_domain_name_servers='192.168.1.1'

Nie wszystkie z tych powyższych zmiennych zostaną uwzględnione w pliku `/tmp/dhclient-script.debug`
. Zależy to głównie od konfiguracji samego dhclient'a jak i serwera DHCP.

## Skrypty w dhclient-enter-hooks.d/ oraz dhclient-exit-hooks.d/

Mamy zatem w grubsza omówioną strukturę pliku `/tmp/dhclient-script.debug` . Dlaczego jednak mamy
dwa osobne katalogi `/etc/dhcp/dhclient-enter-hooks.d/` oraz `/etc/dhcp/dhclient-exit-hooks.d/` ?
Otóż każda akcja, która zostanie zainicjowana przez `dhclient` , może wywołać skrypt. Taki skrypt z
kolei może zostać uruchomiony przed lub po konkretnej akcji. Weźmy sobie przykład z `BOUND` widoczny
w powyższym logu. Ta akcja dotyczy aplikowania konfiguracji uzyskanej od serwera DHCP. Wszystkie
skrypty znajdujące się w katalogu `/etc/dhcp/dhclient-enter-hooks.d/` zostaną wykonane tuż przed
konfiguracją interfejsu. Podobnie sprawa ma się w przypadku katalogu
`/etc/dhcp/dhclient-exit-hooks.d/` , gdzie skrypty zostaną wywołane tuż po skonfigurowaniu
interfejsu.

Wiemy jakie zdarzenia mogą zaistnieć przy odpytywaniu serwera DHCP o lease z konfiguracją adresacji.
Wiemy też jak wywoływane są skrypty. Dodatkowo, wszystkie widoczne wyżej zmienne są przekazywane do
skryptów. W ten sposób możemy bardzo prosto stworzyć skrypty, które będą działać w oparciu o te
zmienne. Wystarczy stworzyć szereg warunków i przypisać im odpowiednie polecenia, które zwykle
chcielibyśmy wpisać w terminal po konfiguracji maszyny za sprawą protokołu DHCP. Przykładem
sytuacji, która wymagałaby dodatkowych poleceń, może być zwykły [failover łącza czy też równoważenie
obciążenia (load
balancing)]({{< baseurl >}}/post/rownowazenie-ruchu-lacz-kilku-isp-load-balancing/), gdzie musimy
skonfigurować trasy routingu. Przy pomocy skryptów dhclient'a, możemy cały proces konfiguracji hosta
w pełni zautomatyzować.

## Jak stworzyć skrypt dla dhclient

Do stworzenia skryptu potrzebnych nam jest kilka informacji. Przede wszystkim, musimy znać nazwę
interfejsu sieciowego, w przypadku którego skrypt zostanie wykonany. Musimy znać także powód (
`reason` ). Wszystkie te informacje są do odczytania z logu dostępnego w pliku
`/tmp/dhclient-script.debug` . Poniżej jest prosty szkielet skryptu, który można wykorzystać:

    #/bin/sh

    # see /sbin/dhclient-script

    RUN="no"

    if [ "$RUN" = "yes" ]; then
        if [ "${interface}" = "eth0" ]; then
            case $reason in
                BOUND|RENEW|REBIND|REBOOT)

                    ;;
                EXPIRE|FAIL|RELEASE|STOP)

                    ;;
            esac
        fi
    fi

Oczywiście, `reason` trzeba sobie odpowiednio dostosować do własnych potrzeb. Ten powyższy szkielet
trzeba także uzupełnić o odpowiednie polecenia, które trzeba wpisać przed `;;` . Tak stworzony
skrypt musimy zapisać w katalogu `/etc/dhcp/dhclient-enter-hooks.d/` lub
`/etc/dhcp/dhclient-exit-hooks.d/` , w zależności od tego czy skrypt ma się wykonać przed czy po
akcji dhclient'a. W sytuacji, gdy tworzymy skrypt, który ma się wykonać w obu przypadkach, możemy go
umieścić w folderze `/etc/dhcp/` i podlinkować go do w/w katalogów.
