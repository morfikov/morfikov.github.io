---
author: Morfik
categories:
- OpenWRT
date: "2016-04-22T19:53:08Z"
date_gmt: 2016-04-22 17:53:08 +0200
published: true
status: publish
tags:
- chaos-calmer
- lte
- rbm
- play
- router
- modem
- huawei
- e3372
GHissueID: 413
title: Monitor połączenia 3G/LTE W OpenWRT (3ginfo)
---

W pewnych sytuacjach lub też ze zwykłej ciekawości możemy chcieć sprawdzić jak sprawuje się
połączenie LTE, którego obsługę zaimplementowaliśmy na naszym routerze mającym na pokładzie
firmware OpenWRT. Nie będę tutaj opisywał samej konfiguracji takiego połączenia, bo to zostało
zrobione we wpisie poświęconym [konfiguracji modemu Huawei E3372 pod
OpenWRT](/post/modem-lte-pod-openwrt/), jak i przy okazji [konfiguracji połączenia
Aero2](/post/konfiguracja-polaczenia-aero2-na-openwrt/). W tym wpisie zaś skupimy
się na monitorowaniu za pomocą `3ginfo` już działającego połączenia, które jest realizowane za
pomocą modemu LTE podłączonego do portu USB routera.

<!--more-->
## Projekt 3ginfo

[Projekt 3ginfo](http://eko.one.pl/?p=openwrt-3ginfo) ma na celu dostarczenia informacji na temat
statusu połączenia internetowego, które udostępniane jest nam za sprawą modemu 3G/LTE. Te dane
zostaną wyświetlone na stronie www. Przejdźmy zatem to instalacji potrzebnych pakietów. Logujemy się
na router i wpisujemy w terminalu te poniższe polecenia:

    # opkg update
    # opkg install 3ginfo

Jak możemy wyczytać w podlinkowanej wyżej stronie, `3ginfo` to w zasadzie skrypt, który w graficznej
formie za pomocą interfejsu webowego zwraca przetworzone wyniki. Nie zjada zatem zbyt dużo zasobów,
zwłaszcza miejsca na flash'u routera. Niemniej jednak, wymagana jest instalacja serwera www
`uhttpd` , który zostanie pociągnięty automatycznie w zależnościach.

## Konfiguracja 3ginfo

Konfiguracja sprowadza się do edycji pliku `/etc/config/3ginfo` . Poniżej przykład:

    config '3ginfo'
            option 'device' '/dev/ttyUSB1'
            option 'pincode' ''
            option 'http_port' '81'
            option 'language' 'en'
            option 'network' 'wan'
            option 'connect_button' '1'
            option 'clf' '/tmp/26001-v3.0h.clf'

Opcja `device` ma wskazywać na interfejs diagnostyczny modemu LTE. Zwykle jest to ostatni interfejs
udostępniany przez modem. Modem Huawei E3372 ma dwa interfejsy `/dev/ttyUSB0` oraz `/dev/ttyUSB1` .
W przypadku, gdy karta SIM jest zabezpieczona za pomocą kodu PIN, to musimy odpowiednio uzupełnić
wartość na pozycji `pincode` . Dalej mamy port, na którym ma nasłuchiwać demon `uhttpd`
udostępniający interfejs webowy. Możemy także przy pomocy `language` zmienić język tego
interfejsu. Aktualnie wspierane są polski ( `pl` ), angielski ( `en` ) oraz włoski ( `it` ). Dalej
mamy `network` , gdzie podajemy sekcję, w której został skonfigurowany modem LTE (w oparciu o plik
`/etc/config/network` ). Mamy także możliwość ukrycia przycisku Connect/Disconnect za pomocą
`connect_button` . Ostatnia opcja, tj. `clf` , odpowiada za link do pliku zawierającego listę stacji
BTS.

### Stacje BTS

Jeśli interesują nas informacje na temat BTS, do którego podłączył się nasz modem, to w opcji `clf`
podajemy link do pliku zawierającego taką listę. Samą listę możemy uzyskać przechodząc [pod ten
adres](http://beta.btsearch.pl/bts/export). Niemniej jednak, ten plik musi być w formacie CLF 3.0,
HEX. Może być także spakowany (archiwum `.gz` ) lub w formie zwykłego tekstu. Ten plik zwykle jest
dość obszerny. W tym przypadku wersja tekstowa ma około 1,5 MiB. spakowana 140 KiB.

Poniżej przykład formularza:

![](/img/2016/04/1.formularz-bts-3ginfo.png#big)

## Monitorowanie połączenia

Po dostosowaniu powyższych opcji, możemy przetestować jak działa `3ginfo` . W tym celu wystarczy
wydać w terminalu to poniższe polecenie:

    # 3ginfo
    Status: Connected
    Connection time: 0d, 01:29:20
    Received / Transmitted data: 29.4 MiB / 34.2 MiB
    Operator: Play
    Operating mode: LTE
    Signal strenght: 77%
    Device: huawei E3372
    MCC MNC: 260 06
    LAC: 0073 (115)
    LCID: 00189B56 (1612630)
    RNC: - (-)
    eNB: 00189B (6299)
    CID: 0056 (86)
    CSQ: 24
    RSSI: -65 dBm
    RSCP: -145 dBm
    Ec/IO: -32 dB
    RSRP: -90 dBm
    RSRQ: -9 dB

Zostanie także utworzony nowy proces:

    # ps | grep uhttpd | grep 3g
     2955 root      1380 S    uhttpd -c /usr/share/3ginfo/uhttpd.conf -h /usr/share/3ginfo/ -p 0.0.0.0:81

Czyli wiemy, że demon `uhttpd` nasłuchuje, a interfejs webowy aplikacji `3ginfo` oczekuje na nas pod
adresem `http://192.168.1.1:81` . Przejdźmy tam i sprawdźmy co tam zastaniemy:

![](/img/2016/04/2.3ginfo-rozlaczony.png#big)

By się połączyć, wciskamy Connect:

![](/img/2016/04/3.3ginfo-polaczony.png#big)

Możemy także podejrzeć szczegóły połączenia rozwijając Show details:

![](/img/2016/04/4.3ginfo-dodatkowe-informacje.png#big)

W oparciu o wygenerowaną listę BTS, `3ginfo` podał nam informacje na temat lokalizacji BTS'a, do
którego zostaliśmy podłączeni. Nie zawsze jednak te informacje będą dostępne, nawet po
wygenerowaniu i zaaplikowaniu listy. Po prostu niektóre BTS nie zostały jeszcze na liście
uwzględnione lub też zawierają błędne informacje.

Co do samego wykresu siły sygnału, który widzimy wyżej, to pod adresem
`http://192.168.1.1:81/signal.html` ten pasek jest aktualizowany automatycznie co 3 sekundy.

W przypadku wystąpienia błędów, dobrze jest rzucić okiem na polecenie `3ginfo-test` .
