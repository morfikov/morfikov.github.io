---
author: Morfik
categories:
- OpenWRT
date: "2016-09-09T17:59:24Z"
date_gmt: 2016-09-09 15:59:24 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- antena
GHissueID: 365
title: 'OpenWRT: Konfiguracja anten via txantenna/rxantenna'
---

Przeglądając forum eko.one.pl wpadł mi w oko [taki oto
temat](http://eko.one.pl/forum/viewtopic.php?pid=171898#p171898). Problem, który został w nim
poruszony dotyczył wykorzystywania pewnej określonej anteny routera. Zakładając, że przeciętny
router ma trzy anteny, powiedzmy, że chcemy wykorzystywać tylko jedną z nich. Dlaczego mielibyśmy
rozważać w ogóle taką sytuację? Przy trzech antenach, teoretyczny transfer w paśmie 2,4 GHz to, w
zależności od routera, 450-600 mbit/s. Przy jednej antenie będziemy mieli max 150-200 mbit/s. Z tego
co czytałem wcześniej w różnych źródłach, uszkodzenie w jakiś sposób toru antenowego może
drastycznie pogorszyć lub wręcz uniemożliwić routerowi transmisję sygnału. Opisana w podlinkowanym
wyżej wątku sytuacja dotyczyła właśnie tego typu zdarzenia, gdzie jedno z gniazd antenowych routera
zostało uszkodzone. Firmware OpenWRT/LEDE jest nam w stanie umożliwić wybór określonych anten przy
pomocy parametrów `diversity` , `txantenna` oraz `rxantenna` . W tym wpisie zobaczymy jak
skonfigurować sobie anteny na przykładzie [routera TL-WR1043ND
V2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) od TP-LINK.

<!--more-->
## Standardowa konfiguracja anten

Firmware producenta bezprzewodowych routerów zwykle nie umożliwia konfiguracji anten. Mamy co prawda
możliwość regulacji mocy transmisyjnej nadajnika ale w przypadku uszkodzenia gniazd anteny, WiFi w
takim urządzeniu zwykle przestaje nam działać. Jeśli konfiguracja naszej sieci domowej opiera się na
WiFi, to taka sytuacja wiąże się zwykle z zakupem nowego routera. W przypadku, gdy nie wszystkie
gniazda antenowe zostały uszkodzone, możemy pokusić się o wgranie firmware OpenWRT/LEDE na router i
odpowiednio skonfigurować anteny do pracy.

Domyślnie w OpenWRT/LEDE są wykorzystywane wszystkie anteny, tak jak w przypadku oryginalnego
firmware. Mamy jednak kilka parametrów, które możemy sobie dostosować i wypadałoby się im przyjrzeć
nieco bliżej. Przede wszystkim, mamy parametr `diversity` , który odpowiada za automatyzację procesu
doboru anten przez sterownik czipu bezprzewodowego. Ta opcja jest domyślnie ustawiona na `1` , zatem
dobór anten jest automatyczny.

Pozostałe dwie opcje to `rxantenna` oraz `txantenna` . Określają one, które anteny mają być używane
do odbierania lub transmitowania sygnału. Te opcje nie są ustawione standardowo, bo nie mają żadnego
efektu, gdy został określony parametr `diversity` .

Standardową konfigurację anten dla routera TL-WR1043ND możemy odczytać z wyjścia polecenia `iw` ,
które trzeba wpisać w terminalu będąc zalogowanym po SSH lub telnet na router. Poniżej przykład:

    # iw phy phy0 info | grep -i ant
        Available Antennas: TX 0x7 RX 0x7
        Configured Antennas: TX 0x7 RX 0x7

Pierwsza zwrócona pozycja określa dostępne anteny, natomiast druga informuje nas o konfiguracji
anten. Oznaczenia `TX`/`RX` dotyczą odpowiednio anten nadawczych i odbiorczych. Zastanawiającą
pozycją może być `0x7` (akurat w przypadku tego routera). Jest to zapis HEX, który po
przetłumaczeniu na system binarny daje 111. Mamy trzy anteny, a 1 oznacza, że dana antena jest
włączona w konfiguracji. Jeśli chcemy wyłączyć środkową antenę w routerze, to zapis binarny będzie
101, co przekłada się na 0x5 w systemie HEX. Problematyczne może być tylko ustalenie, która antena w
naszym routerze jest na jakiej pozycji w tym ciągu binarnym.

## Jak ustalić kolejność anten w routerze

Mój router jest w pełni sprawny. Niemniej jednak, to w niczym nie przeszkadza. Poniżej jest fotka
obrazująca parametry połączenia bezprzewodowego w `wavemon` odbieranego na stacji klienckiej z
linux'em na pokładzie:

![txantenna-rxantenna-anteny-router-openwrt-konfiguracja](/img/2016/09/1.txantenna-rxantenna-anteny-router-openwrt-konfiguracja.png#huge)

Mamy tutaj całkiem przyzwoitą jakość sygnału. Odkręćmy teraz wszystkie anteny, które są doczepione
do tego routera WiFi. Sprawdźmy co się stało z
sygnałem:

![txantenna-rxantenna-anteny-router-openwrt-konfiguracja](/img/2016/09/2.txantenna-rxantenna-anteny-router-openwrt-konfiguracja.png#huge)

Widzimy, że jakość sygnału uległa bardzo mocnej degradacji. Teraz musimy ustalić, która antena
znajduje się jako pierwsza w szeregu binarnym. Robimy to przez dodanie do pliku
`/etc/config/wireless` parametrów `diversity` , `rxantenna` oraz `txantenna` w bloku `config
'wifi-device'` , przykładowo:

    ...
    config 'wifi-device'
        ...
        option diversity '0'
        option rxantenna '1'
        option txantenna '1'
        ...

Teraz wystarczy dokręcić jedną antenę we wszystkie gniazda antenowe i zobaczyć kiedy sygnał ulegnie
poprawie. Będzie to wskazanie, że to jest antena oznaczona 001. W tym przypadku sygnał uległ
poprawie po podłączeniu anteny do gniazda z lewej strony patrząc od przodu routera:

![txantenna-rxantenna-anteny-router-openwrt-konfiguracja](/img/2016/09/3.txantenna-rxantenna-anteny-router-openwrt-konfiguracja.png#huge)

Jak widzimy sygnał jest nieco gorszej jakości w stosunku do bazowego ale to właśnie dlatego, że
tylko jedna z trzech anten jest w użyciu.

## Dobranie wartości rxantenna/txantenna

Wiemy zatem, że antena z lewej strony routera jest oznaczona 001. Środkowa antena ma numer 010 i
prawa antena ma numer 100. Poniżej jest rozpiska wartości, które trzeba wpisać do pliku
`/etc/config/wireless` w parametrach `rxantenna` oraz `txantenna` w zależności od pożądanej przez
nas konfiguracji anten:

    bi     hex      anteny
    ================================================
    001     1       lewa                        *
    010     2       środkowa                    *
    011     3       lewa  + środkowa            *
    100     4       prawa                       *
    101     5       lewa  + prawa
    110     6       prawa + środkowa
    111     7       lewa  + środkowa + prawa    *

Zatem jeśli padło nam prawe gniazdo anteny, to w pliku `/etc/config/wireless` dajemy poniższą
konfigurację:

    ...
    config 'wifi-device'
        ...
        option diversity '0'
        option rxantenna '3'
        option txantenna '3'
        ...

Po każdej zmianie pliku `/etc/config/wireless` restartujemy połączenie bezprzewodowe na routerze
wpisując w terminalu polecenie `wifi` . Jeśli następnie wydamy polecenie `iw` , to powinien zostać
nam zwrócony nieco inny wynik w porównaniu do domyślnych ustawień. Dla anteny lewej i środkowej
pracujących jednocześnie będzie coś w poniższym stylu:

    # wifi
    # iw phy phy0 info | grep -i ant
        Available Antennas: TX 0x7 RX 0x7
        Configured Antennas: TX 0x3 RX 0x3

## Problem z doborem anten

Trzeba jednak uważać, bo nie wszystkie konfiguracje anten są możliwe. W przypadku routera
TL-WR1043ND możliwe są kombinacje, które w powyższej rozpisce są oznaczone za pomocą `*` . Nie wiem
dlaczego dwie skrajne anteny oraz prawa + środkowa antena nie mogą być wykorzystywane jednocześnie.
Niby można ustawić taki setup ale komputery tracą połączenie przez WiFi.
