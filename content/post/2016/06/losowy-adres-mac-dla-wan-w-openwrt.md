---
author: Morfik
categories:
- OpenWRT
date: "2016-06-05T18:28:12Z"
date_gmt: 2016-06-05 16:28:12 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
- mac
title: Losowy adres MAC dla WAN w OpenWRT
---

Na dużych dystrybucjach linux'a adres MAC można zmienić bez problemu. Podobnie sprawa ma się w
przypadku [automatycznego generowania takiego adresu
MAC]({{< baseurl >}}/post/jak-przypisac-losowy-adres-mac-interfejsu/) za każdym razem, gdy chcemy
nawiązać połączenie z internetem. W OpenWRT rozwiązanie tego zadania nie jest tak oczywiste jak, np.
na debianie, ale też znowu nie jest niemożliwe. W repozytorium OpenWRT mamy dostępny pakiet
`macchanger` . Niemniej jednak, w przypadku routerów o małych pamięciach flash, instalowanie
dodatkowych pakietów może nie być dobrym pomysłem. Przydałoby się zatem zaprojektować mechanizm
generowania i zmiany adresu MAC interfejsu WAN za każdym razem, gdy będziemy resetować router i to
zadanie postaramy się zrealizować w tym artykule.

<!--more-->
## Skrypt startowy generujący losowy adres MAC

Przede wszystkim, potrzebny nam jest [skrypt startowy
init]({{< baseurl >}}/post/skrypty-startowe-init-w-openwrt/). Ten skrypt będzie zawierał dwie
sekcje: `boot() {}` oraz `start() {}` . W pierwszej z nich będą zawarte instrukcje wywoływane na
starcie routera. W drugiej zaś akcje, które będzie można wywołać w dowolnym innym czasie. Tworzymy
zatem plik `/etc/init.d/mac-random` i wrzucamy do niego tę poniższą zawartość:

    #!/bin/sh /etc/rc.common

    START=17

    boot() {
        macaddr=$(hexdump -e '5/1 "%02X:" "%02X"' /dev/urandom -n 6 | head -c 17)
        lastfive=$( echo "$macaddr" | cut -d: -f 2-6 )
        firstbyte=$( echo "$macaddr" | cut -d: -f 1 )

        firstbyte=$( printf '%02x' $(( 0x$firstbyte & 254 | 2)) )

        newmac="$firstbyte:$lastfive"

        logger "Setting NEW MAC address for WAN: $newmac ..."
        uci set network.wan.macaddr=$newmac
        uci commit network
    }

    start() {
        macaddr=$(hexdump -e '5/1 "%02X:" "%02X"' /dev/urandom -n 6 | head -c 17)
        lastfive=$( echo "$macaddr" | cut -d: -f 2-6 )
        firstbyte=$( echo "$macaddr" | cut -d: -f 1 )

        firstbyte=$( printf '%02x' $(( 0x$firstbyte & 254 | 2)) )

        newmac="$firstbyte:$lastfive"

        ifdown wan
        logger "Setting NEW MAC address for WAN: $newmac ..."
        uci set network.wan.macaddr=$newmac
        uci commit network
        ifup wan
    }

Jako, że to jest skrypt startowy, to musimy mu jeszcze nadać prawa wykonywania i dodać go do
autostartu:

    # chmod +x /etc/init.d/mac-random
    # /etc/init.d/mac-random enable

W powyższym skrypcie można oczywiście skorzystać z narzędzia `macchanger` . Niemniej jednak, w tym
przypadku korzystamy z [UCI](https://wiki.openwrt.org/doc/uci). Warto zaznaczyć, że korzystanie z
`uci` czyści komentarze w plikach konfiguracyjnych. Jeśli w pliku `/etc/config/network` posiadamy
jakieś linijki zaczynające się od `#` , to lepiej zróbmy sobie backup tego pliku.

## Cykliczna zmiana adresu MAC via cron

Routery mają to do siebie, że zwykle pracują bez przerwy. Dlatego też co 6h, 12h czy 24h przydałoby
się ustawić na nowo adres MAC. Ten adres może zostać zmieniony jedynie w momencie dezaktywacji
interfejsu sieciowego. Dlatego wymagane jest rozłączenie połączenia. Jeżeli możemy pozwolić sobie na
chwilową utratę łączności ze światem, to przy pomocy cron'a możemy cyklicznie ten adres zmieniać. By
to uczynić, w terminalu wpisujemy `crontab -e` . Tam z kolei umieszczamy ten poniższy wpis:

    # min   hour    day     mon     dow     command
    10     */6    *       *       *       /etc/init.d/mac-random start

Liczba `10` oznacza wywołanie skryptu w dziesiątej minucie. Natomiast `*/6` określa konkretne
godziny. W tym przypadku są to 0, 6, 12, 18, czyli co 6h.

Pozostaje nam już tylko uruchomić router ponownie i sprawdzić czy adres MAC interfejsu WAN uległ
zmianie:

    # logread | grep MAC
    Sun Jun  5 17:48:34 2016 user.notice root: Setting NEW MAC address for WAN: 32:E0:84:06:C8:C8 ...

    # ip link show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc htb state UP mode DEFAULT group default qlen 1000
        link/ether 32:e0:84:06:c8:c8 brd ff:ff:ff:ff:ff:ff
