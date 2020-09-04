---
author: Morfik
categories:
- OpenWRT
date: "2016-06-15T20:33:34Z"
date_gmt: 2016-06-15 18:33:34 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- router
title: Karta WiFi w trybie MONITOR w OpenWRT
---

Routery WiFi, zwłaszcza te na podzespołach firmy Qualcomm, mają zwykle bardzo dobre wsparcie w
alternatywnym firmware OpenWRT. Te bezprzewodowe routery posiadają na pokładzie zwykle jedną lub
dwie karty WiFi, w zależności od obsługiwanego pasma (2,4 GHz i/lub 5 GHz). Karty tych routerów
działają standardowo w trybie AP (Access Point), czyli [punktu dostępowego][1]. W taki sposób
jesteśmy w stanie bezprzewodowo połączyć szereg urządzeń w sieci domowej z internetem. Niemniej
jednak, karta WiFi może pracować w kilku innych trybach. W tym wpisie zobaczymy jak przełączyć
kartę WiFi routera w tryb MONITOR.

<!--more-->
## Różnica między trybem AP i MONITOR

Standardowo w trybie AP, karta routera akceptuje jedynie te pakiety sieciowe, które są dla niej
przeznaczone. Wszystkie pozostałe pakiety są ignorowane. Tryb MONITOR z kolei, to taki tryb, który
umożliwia monitorowanie pasma sieci bezprzewodowej. W tym trybie, karta sieciowa jest w stanie
wyłapać wszystkie pakiety, które latają w eterze na określonej częstotliwości. Nawet jeśli sieć
WiFi jest zabezpieczona protokołem WPA2-PSK, to i tak w dalszym ciągu jesteśmy w stanie wyłapać
pewne użyteczne informacje, np. adresy MAC klientów takiej sieci.

Routery pracują w trybie AP ze zrozumiałego chyba nam wszystkim powodu. Zwykle też w oryginalnym
firmware producenta routera nie znajdziemy opcji od monitorowania pasma bezprzewodowego. Po co nam
zatem tego typu umiejętność przełączania karty w tryb MONITOR? W sumie to jest dobre pytanie ale
skoro OpenWRT daje nam taką możliwość, to grzechem by było przejść obok tego ficzera obojętnie.

## Włączanie trybu MONITOR w konfiguracji OpenWRT

Konfiguracja sieci WiFi w OpenWRT jest zlokalizowana w pliku `/etc/config/wireless` . Mamy tam
zwykle dwa bloki. Jeden z nich jest odpowiedzialny za konfigurację urządzenia ( `config wifi-device`
). Drugi zaś za konfigurację parametrów sieci WiFi ( `config wifi-iface` ). Nas będzie interesował
ten drugi blok.

Trzeba tutaj wyraźnie zaznaczyć, że karta routera może pracować w kilku trybach jednocześnie. Nie
musimy zatem wyłączać aktualnie skonfigurowanego profilu sieci w pliku `/etc/config/wireless` , by
włączyć tryb MONITOR.

Edytujemy zatem plik `/etc/config/wireless` i na jego końcu dodajemy ten oto poniższy kod:

    config wifi-iface
        option device 'radio0'
        option network 'monitor'
        option mode 'monitor'
        option disabled '0'

Teraz przy pomocy polecenia `iwinfo` sprawdzamy czy karta działa w pożądanym przez nas trybie. W tym
przypadku, karta działa zarówno w trybie AP, jak i MONITOR. Wygląda to mniej więcej tak:

![]({{< baseurl >}}/img/2016/06/1.openwrt-router-wifi-monitor-mode-tryb.png#big)

Mając kartę w trybie MONITOR, możemy na routerze doinstalować szereg pakietów, min. `reaver` lub
`aircrack-ng` , których przeznaczeniem są testy penetracyjne sieci WiFi. W ten sposób możemy
przeskanować naszą domową sieć bezprzewodową i sprawdzić czy jest ona odporna na szereg popularnych
ataków.

[1]: https://pl.wikipedia.org/wiki/Punkt_dost%C4%99pu
