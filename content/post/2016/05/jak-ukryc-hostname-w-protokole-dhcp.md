---
author: Morfik
categories:
- Linux
date: "2016-05-24T20:07:14Z"
date_gmt: 2016-05-24 18:07:14 +0200
published: true
status: publish
tags:
- prywatność
- wifi
- sieć
- dhcp
title: Jak ukryć hostname w protokole DHCP
---

Darmowe hotspoty sieci WiFi są dostępne w każdym mieście. Dzięki nim możemy uzyskać połączenie z
internetem praktycznie za free. Niemniej jednak, takie połączenie nie jest do końca bezpieczne i
może zagrażać naszej prywatności. Wiele osób stara się temu przeciwdziałać [generując losowy adres
MAC]({{< baseurl >}}/post/jak-przypisac-losowy-adres-mac-interfejsu/). No i to jest jakieś
wyjście, o ile ten adres jest generowany z głową. Niemniej jednak, w takich sieciach WiFi, host ma
przydzielaną adresację za pomocą [protokołu
DHCP](https://pl.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol). Ci, którzy wiedza, jak
odbywa się konfiguracja za pomocą tego protokołu, wiedzą też, że nasz komputer przesyła pewne dane
do serwera DHCP. Jakie dane? To zwykle zależy od konfiguracji klienta DHCP. Na linux'ach domyślnym
klientem DHCP jest `dhclinet` i on standardowo przesyła nazwę hosta (hostname) w zapytaniu o
adresację. Co nam zatem po losowym adresie MAC, gdy można nas zidentyfikować po nazwie hosta? W tym
artykule postaramy się ukryć lub też losowo wygenerować hostname danej maszyny w sieci.

<!--more-->
## Jak wygląda zapytanie do serwera DHCP

By zrozumieć na czym polega problem, musimy sięgnąć do analizy pakietów, które są przesyłane przez
sieć. Potrzebny nam zatem będzie jakiś sniffer sieciowy. Ja lubię korzystać z
[wireshark'a](https://www.wireshark.org/). Pakiet `wireshark` jest dostępny w debianie i nie powinno
być problemów z jego zainstalowaniem. Po procesie instalacyjnym, odpalamy tę aplikację i
przechodzimy co "Capture Options" (4 ikonka na lewo w menu). Wybieramy interfejs sieciowy i
ustawiamy mu filtr tak, by łapał jedynie pakiety z protokołu DHCP: `port 67 or port 68` :

![]({{< baseurl >}}/img/2016/05/1.wireshark-filtr-dhcp.png)

Po zapuszczeniu sniffer'a, podnosimy interfejs sieciowy, np. za pomocą `ifup bond0` . Powinny zostać
odnotowane cztery pakiety. Klikamy pierwszy z nich i rozwijamy opcje protokołu. Tam z kolei widzimy,
że klient DHCP przesłał do serwera DHCP hostname naszego
komputera:

[![2.wireshark-dhcp-hostname]({{< baseurl >}}/img/2016/05/2.wireshark-dhcp-hostname-1024x569.png)]({{< baseurl >}}/img/2016/05/2.wireshark-dhcp-hostname.png)

W ten sposób sami się identyfikujemy w darmowych sieciach i to nie tylko muszą być sieci WiFi.

## Jak ukryć hostname w pakietach protokołu DHCP

Jak wspomniałem we wstępie, dynamiczna konfiguracja hosta wymaga do poprawnego działania serwera i
klienta DHCP. W linux'ach standardowo klientem DHCP jest `dhclient` . Konfiguracja tego narzędzia
znajduje się w pliku `/etc/dhcp/dhclient.conf` . Zajrzyjmy zatem tam:

    ...
    send host-name = gethostname();

    request subnet-mask, broadcast-address, time-offset, routers,
        host-name, domain-name-servers, domain-name, domain-search,
        dhcp6.name-servers, dhcp6.domain-search,
        dhcp6.fqdn, dhcp6.sntp-servers,
        netbios-name-servers, netbios-scope, interface-mtu,
        rfc3442-classless-static-routes, ntp-servers;
    ...

Mamy tutaj dwie sekcje: `send` oraz `request` . Wszystkie opcje, które widnieją w `request` są
przesyłane do serwera DHCP i jeśli taki serwer posiada informacje na którąkolwiek z nich, to ją
załącza w pakiecie zwrotnym. Natomiast w `send` są zawarte informacje, które nasz klient DHCP
przesyła do serwera w zapytaniu o konfigurację adresacji. Widzimy wyraźnie, że widnieje tutaj
`host-name` . W przypadku `request` , ta opcja nie ma praktycznie żadnego znaczenia. Natomiast jeśli
chcemy zataić nasz hostname, to musimy oduczyć klienta DHCP wysyłania tej powyższej informacji.
Jako, że tylko jedna opcja jest przesyłana do serwera DHCP, to wystarczy wykomentować tę poniższą
linijkę:

    #send host-name = gethostname();

Zapiszmy konfigurację i jeszcze raz zapuśćmy
wiresharka:

[![3.wireshark-dhcp-ukrycie-hostname]({{< baseurl >}}/img/2016/05/3.wireshark-dhcp-ukrycie-hostname-1024x569.png)]({{< baseurl >}}/img/2016/05/3.wireshark-dhcp-ukrycie-hostname.png)

Jak widzimy na powyższym obrazku, `dhclient` nie przesłał tym razem naszego hostname do serwera
DHCP.

## Czy ustawienie losowego hostname ma sens

W powyżej opisanej sytuacji nie przesyłamy żadnego hostname do serwera DHCP. Niektórzy jednak
chcieliby wygenerowania losowego hostname w celach "lepszej anonimowości". Czasami losowość jest tym
co nas może zidentyfikować. Nawet jeśli ustawimy sobie losowy hostname na `XHDYSIE` , a później
`JDKSIWS` , to w gąszczu nielosowych nazw te powyższe dwie bardzo rzucają się w oczy. Dlatego też
unikałbym tego rozwiązania. Oczywiście, jako, że klient DHCP wysyła standardowo hostname, to
istnieje też szansa, że rzucą się w oczy komuś maszyny bez zdefiniowanej nazwy i będzie on w stanie
z mniejszym lub większym prawdopodobieństwem nas zidentyfikować.
