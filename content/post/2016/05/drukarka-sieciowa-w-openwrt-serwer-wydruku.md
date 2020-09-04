---
author: Morfik
categories:
- OpenWRT
date: "2016-05-11T19:32:50Z"
date_gmt: 2016-05-11 17:32:50 +0200
published: true
status: publish
tags:
- chaos-calmer
- drukarka
- router
- epson
title: Drukarka sieciowa w OpenWRT (serwer wydruku)
---

OpenWRT daje możliwość doinstalowania całej masy aplikacji, które są w stanie realizować pewne dość
wyrafinowane zadania. Jednym z takich zadań jest serwer wydruku, czyli możliwość obsługi różnego
rodzaju drukarek. Jeśli nasz router posiada port USB, to taką drukarkę jesteśmy w stanie do niego
podłączyć. Nawet jeśli router dysponuje tylko jednym portem USB i do tego zajętym już, to nic nie
stoi na przeszkodzie, by dokupić HUB'a i rozgałęzić sobie ten pojedynczy port. Serwer wydruku ma tę
zaletę, że drukarka jest udostępniana przez router w sieci domowej. W efekcie odpada nam
utrzymywanie dedykowanego komputera, który zajmowałby się tylko obsługą takiej drukarki. Możemy
zatem oszczędzić nieco na rachunku za prąd. W tym wpisie skonfigurujemy sobie właśnie taki serwer
wydruku w oparciu o drukarkę EPSON Stylus Color 760 i oprogramowanie `p910nd` .

<!--more-->
## Obsługa drukarki w OpenWRT

Do uruchomienia serwera wydruku pod OpenWRT potrzebne nam będą z grubsza trzy rzeczy. Pierwszą z
nich jest obsługa urządzeń USB na routerze, w szczególności drukarek. Drugą zaś to
sterowniki/firmware do drukarki. No a trzecia to demon realizujący zapytania do drukarki. Poniżej
jest lista pakietów, które musimy doinstalować na routerze:

    opkg update
    # opkg install p910nd kmod-usb-printer kmod-usb-uhci kmod-usb-core kmod-usb2

Podłączamy teraz drukarkę do portu USB routera i podajemy jej zasilanie. Powinna być ona po chwili
rozpoznana przez system:

![]({{< baseurl >}}/img/2016/05/1.p910nd-drukarka-sieciowa-serwer-wydruku-openwrt.png#huge)

Wyżej widzimy frazę `usblp0` . Odpowiada ona za urządzenie `/dev/usb/lp0` . To urządzenie musi
istnieć byśmy mogli korzystać z drukarki.

## Drukarka pod OpenWRT

Mając dostępne w systemie urządzenie `/dev/usb/lp0` , możemy przejść do konfiguracji demona drukarki
`p910nd` . Ma on swój plik w katalogu `/etc/config/p910nd` , w którym to musimy określić min.
urządzenie i kilka pomniejszych parametrów. W przypadku tej drukarki, konfiguracja wygląda mniej
więcej tak:

    config p910nd
        option device        /dev/usb/lp0
        option port          0
        option bidirectional 1
        option enabled       1

W opcji `device` podajemy ścieżkę do urządzenia, które zostało zwrócone w logu po wykryciu drukarki.
Standardowo nie musimy ruszać opcji `port` . Mimo, że ona wskazuje na `0` , to nie jest to port, na
którym będzie operował `p910nd` . Ten demon działa na porcie 9100. W przypadku, gdy ten port
chcielibyśmy zmienić, to w opcji `port` podajemy numer, który zostanie dodany do tego 9100.
Przykładowo, jeśli określimy 10, to usługa będzie aktywna na porcie 9110. Jako, że szereg drukarek
nie działa w trybie `bidirectional` , to czy tę opcję włączymy zależy od posiadanego przez nas
sprzętu. Wyżej w logu widzieliśmy, że ta przykładowa drukarka obsługuje ten tryb. Dlatego też ta
opcja została aktywowana. Upewnijmy się także, że opcja `enabled` również jest włączona. Po
skonfigurowaniu demona, aktywujemy usługę poniższym poleceniem:

    # /etc/init.d/p910nd start

Demon powinien działać w tle. My zaś możemy przejść do konfiguracji maszyn klienckich. W zależności
od systemu operacyjnego na takim komputerze, ten proces będzie inny. [Konfiguracji drukarki na
linux'ie w oparciu o CUPS]({{< baseurl >}}/post/cups-konfiguracja-drukarki-pod-linuxem/) została
opisana w osobnym artykule.
