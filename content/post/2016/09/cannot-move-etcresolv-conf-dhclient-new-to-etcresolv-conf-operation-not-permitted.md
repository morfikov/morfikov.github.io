---
author: Morfik
categories:
- Linux
date: "2016-09-10T20:54:57Z"
date_gmt: 2016-09-10 18:54:57 +0200
published: true
status: publish
tags:
- dns
- dhcp
- dhclient
title: 'Cannot move /etc/resolv.conf.dhclient-new to /etc/resolv.conf: Operation not permitted'
---

Użytkownicy linux'a przykładają nieco większą wagę do konfiguracji swojego systemu. Te nieco
bardziej świadome jednostki zdają sobie sprawę, że różnego rodzaju automaty są w stanie przepisywać
konfigurację systemową bez naszej wiedzy. Weźmy sobie resolver DNS. To bardzo krytyczna usługa, nie
tylko z punktu widzenia bezpieczeństwa ale też i prywatności. Zakładając, że chcemy korzystać z
pewnych określonych serwerów DNS lub też mamy [skonfigurowaną usługę
dnscrypt-proxy](/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/), trzeba zadbać
o to, by adresy w pliku `/etc/resolv.conf` nie zostały z jakiegoś powodu przepisane. Użytkownicy
zwykle nadają temu plikowi atrybut odporności ( `chattr +i` ) . Niemniej jednak, przy pobieraniu
konfiguracji sieciowej za sprawą protokołu DHCP, w logu można zaobserwować komunikat: `mv: cannot
move '/etc/resolv.conf.dhclient-new' to '/etc/resolv.conf': Operation not permitted` . Niby w niczym
on nie przeszkadza ale możemy tak skonfigurować demona `dhclient` , by tę wiadomość wyeliminować.

<!--more-->
## Konfiguracja serwerów DNS przez plik /etc/network/interfaces

W pliku `/etc/network/interfaces` zwykliśmy umieszczać konfigurację interfejsów sieciowych. W tym
pliku można zawrzeć całą masę parametrów i przypisać je do konkretnego interfejsu. Jest też opcja
konfigurująca serwery DNS. Konkretnie rozchodzi się o `dns-nameservers` . Przy pomocy tej opcji
jesteśmy w stanie nadpisać konfigurację serwerów DNS pobieraną z serwera DHCP. Poniżej przykład:

    auto eth0
    iface eth0 inet dhcp
       dns-nameservers 8.8.8.8 8.8.4.4

Niemniej jednak, w tym powyższym przypadku, plik `/etc/resolv.conf` również jest przepisywany.
Poniżej przykład logu systemowego:

![](/img/2016/09/1.resolv-conf-dns-resolver-dhclinet.png#huge)

Może i nasze serwery DNS zostaną umieszczone w tym pliku ale nadal nie jest to konfiguracja, która
by zaspokoiła te bardziej wyrafinowane umysły informatyczne.

## Skrypty dhclient'a

Jakiś czas temu opisywałem [skrypty
dhclient'a](/post/skrypt-dhclienta-dhclient-script/). To bardzo ciekawy ficzer tego
narzędzia, bo dzięki niemu możemy niezależnie skonfigurować całą masę opcji protokołu DHCP.
Potrzebny jest nam tylko odpowiedni skrypt, który wyłączy na określonych interfejsach sieciowych
konfigurację serwerów DNS. Poniżej znajduje się przykład takiego skryptu:

    #/bin/sh

    # see /sbin/dhclient-script

    RUN="yes"

    if [ "$RUN" = "yes" ]; then
        if [ "${interface}" = "eth0" ] || [ "${interface}" = "wlan0" ]; then
            case $reason in
                BOUND|RENEW|REBIND|REBOOT)

                    make_resolv_conf() {
                        return 0
                    }

                    ;;
            esac
        fi
    fi

Sam skrypt nie jest jakoś szczególnie skomplikowany. Przy konfiguracji interfejsów, zmienna
`${interface}` przyjmuje wartość, np. `eth0` lub `wlan0` . Generalnie na podstawie tej zmiennej
jesteśmy w stanie rozróżnić interfejsy sieciowe. Mając tę informację, możemy stworzyć warunki i w
nich dokładnie skonfigurować serwery DNS.

Tak się składa, że `dhclient` ma oddelegowaną osobną funkcję do wyciągania adresów serwerów DNS:
`make_resolv_conf()` . W przypadku, gdy zwróci ona `0` , serwery DNS nie będą konfigurowane. Jako,
że proces konfigurowania interfejsów sieciowych za pomocą protokołu DHCP ma szereg faz, to musimy
je również uwzględnić. W tym przypadku są to `BOUND` , `RENEW` , `REBIND` oraz `REBOOT` .

## Test przepisywania pliku /etc/resolv.conf

Zapisujemy skrypt i sprawdzamy, czy tym razem `dhclinet` zostawi plik `/etc/resolv.conf` w spokoju i
czy błąd z przepisaniem pliku zostanie wyeliminowany. Odpalamy zatem terminal i wpisujemy w nim
`ifup eth0` :

![](/img/2016/09/2.resolv-conf-dns-resolver-dhclinet.png#huge)

Jak widzimy, nie ma już błędu, bo system nie chce nam już przepisywać pliku `/etc/resolv.conf` ,
który w dalszym ciągu ma ustawiony atrybut odporności ( `i` widoczne w ostatniej linijce na
powyższej fotce).
