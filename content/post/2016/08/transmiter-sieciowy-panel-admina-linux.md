---
author: Morfik
categories:
- Linux
date: "2016-08-20T21:54:57Z"
date_gmt: 2016-08-20 19:54:57 +0200
published: true
status: publish
tags:
- wifi
- sieć
- tp-link
- ekstender
GHissueID: 558
title: Transmiter sieciowy i jego panel admina pod linux'em
---

Bawię się ostatnio trochę transmiterem sieciowym (powerline ekstender). Konkretnie jest to zestaw
[TL-WPA4226T KIT (AV500)][1] od TP-LINK. Same urządzenia działają przyzwoicie i realizują powierzane
im funkcje w sposób bardzo zadowalający ale był jeden problem, który mi nie dawał spokoju. Do tych
ekstenderów jest dołączona płytka. Na płytce są aplikacje, które umożliwiają konfigurację tych
transmiterów sieciowych. Te programiki nie mają wersji dla linux'a. Nasunęło mi się zatem pytanie:
to jak mam niby te transmitery skonfigurować pod tym systemem operacyjny? Niby one działają OOTB ale
w przypadku bezprzewodowego routera WiFi z alternatywnym firmware OpenWRT/LEDE na pokładzie
występuje kolizja adresów IP. Zarówno ekstendery jak i router roszczą sobie prawo do adresu
192.168.1.1 . Panel admina takich transmiterów umożliwia zmianę tego adresu, tylko nie mamy jak się
do niego dobrać z poziomu linux'a. W tym artykule postaramy się rozwiązać problem kolizji adresów IP
i skonfigurujemy nasz transmiter tak, by miał inny adres.

<!--more-->
## Czy każdy transmiter sieciowy ma adres IP

W zestawie TL-WPA4226T KIT były tylko dwa rodzaje transmiterów: TL-WPA4220 oraz TL-PA4020P.
TL-PA4020P nie dysponuje adresem IP i nie posiada przy tym punktu dostępowego sieci WiFi. Natomiast
TL-WPA4220 ma adres IP i ma również AP. Zatem można stąd wysnuć wniosek, że adres IP mają te
ekstendery, które dysponują AP. Wydaje się to oczywiste, bo jak inaczej mielibyśmy się podłączyć do
tego punktu dostępowego za pomocą sieci WiFi?

Problematyczne jednak może okazać się znalezienie adresu IP tego punktu, a zarazem samego
transmitera sieciowego. Nie znając adresu, nie możemy się połączyć z urządzeniem. Jeśli go poznamy,
to możemy być pewni, że TP-LINK ulokował na nim jakiś panel administracyjny, mniej więcej taki jak w
swoich routerach. Przy routerach TP-LINK'a, adres IP jest ustawiony na 192.168.0.1, a w przypadku
ekstenderów powerline? To jest dobre pytanie, a `wireshark` nam z chęcią na nie odpowie.

## Jak ustalić adres IP transmitera sieciowego

W tym przypadku rozbudowa sieci za pomocą transmiterów sieciowych przebiegła w standardowy sposób.
Dwa transmitery zostały ze sobą powiązane (sparowane) i została skopiowana konfiguracja sieci
bezprzewodowej (WiFi CLONE) udostępnianej na routerze wyposażonym w firmware OpenWRT/LEDE. Cały
proces przebiegł szybko i sprawnie, a sieć działa tak jak powinna.

Jeśli teraz podłączymy sobie maszynę kliencką do naszej sieci i odpalimy na niej sniffer
`wireshark` , to zobaczymy coś w poniższym stylu:

![](/img/2016/08/1.wireshark-duplikad-adres-ip-transmiter-sieciowy.png#huge)

Dwie maszyny w naszej sieci mają przypisany ten sam adres IP: 192.168.1.1 . Taka sytuacja będzie
powodować problemy z połączeniem. W sumie to już możemy na starcie to odczuć, choćby przez wpisanie
w przeglądarce adresu `http://192.168.1.1/` . Powinien nam się wyświetlić panel administracyjny
ekstendera ale przeglądarka zwraca błąd, bo zapytanie leci bezpośrednio do routera WiFi.

Adres 192.168.1.1 jest domyślnie wykorzystywany przez firmware OpenWRT i moim zdaniem powinniśmy go
zostawić w spokoju. Zaistniały problem możemy rozwiązać przez zmianę adresu IP transmitera, tylko
pierw musimy się do niego jakoś podłączyć. Zanim to jednak uczynimy, musimy zresetować to urządzenie
do ustawień fabrycznych. Na jego obudowie jest przycisk RESET, który wciskamy i trzymamy, aż
transmiter zgasi diody. Po restarcie transmitera, nie parujemy go jeszcze. Ten ekstender WiFi nie
może być częścią naszej sieci, przynajmniej do momentu zmiany jego adresu IP.

Ten transmiter sieciowy może i posiada punkt dostępowy z określonym adresem IP ale nie posiada
serwera DHCP. W efekcie po powiązaniu klienta z AP, nie możemy liczyć na przesłanie dynamicznej
konfiguracji sieci. To samo tyczy się połączenia przewodowego z ekstenderem.

Wiemy za to, że ten transmiter jest przypisany do sieci 192.168.1.0/24 . Musimy zatem ręcznie
przypisać adres z tej klasy na interfejsie sieciowym naszego linux'a. Połączenie może być przewodowe
lub bezprzewodowe, to bez znaczenia. Dla uproszczenia, lepiej jest wybrać połączenie przewodowe, bo
nie musimy zawracać sobie głowy parametrami sieci bezprzewodowej. Poniżej przykładowe polecenia:

    # ifdown eth0
    # ip set dev eth0 up
    # ip addr flush dev eth0
    # ip addr add 192.168.1.150/24 brd + dev eth0

## Testy panela administracyjnego transmitera

W tej chwili powinniśmy mieć połączenie z ekstenderem WiFi, które możemy przetestować za pomocą
polecenia `ping 192.168.1.1` . Jeśli `ping` lata w obie strony, to odpalamy przeglądarkę internetową
i w pasku adresu wpisujemy `http://192.168.1.1/` . Powinien nam pojawić się prompt z zapytaniem o
login i hasło. W obu przypadkach wpisujemy `admin` :

![](/img/2016/08/2.transmiter-sieciowy-panel-admina.png#medium)

Po chwili powinien nam się już załadować znajomy panel administracyjny TP-LINK'a:

![](/img/2016/08/3.transmiter-sieciowy-panel-admina.png#huge)

Jak widzimy wyżej, ten ekstender WiFi ma adres 192.168.1.1 i to ten adres musimy zmienić na coś
innego, z tym, że z klasy 192.168.1.0/24. Zmieńmy adres tego urządzenia na 192.168.1.10 .
Pamiętajmy, że standardowa pula DHCP w OpenWRT/LEDE jest z zakresu 192.168.1.100-192.168.1.250 . W
celu zmiany adresu przechodzimy w menu pod Network i wpisujemy w formularzu nasz nowy IP:

![](/img/2016/08/4.transmiter-sieciowy-panel-admina-samiana-adresu.png#huge)

Po chwili nastąpi restart urządzenia:

![](/img/2016/08/5.transmiter-sieciowy-panel-admina-restart.png#huge)

Teraz możemy kontynuować ze standardową procedurą implementacji transmitera sieciowego w naszym
domu. Po sparowaniu urządzeń, nie powinno być już w sieci zduplikowanego adresu IP 192.168.1.1
przypisanego do dwóch różnych adresów MAC. Natomiast panel administracyjny naszego transmitera
powinien być dostępny pod adresem 192.168.1.10 . Pamiętajmy, że każdy taki ekstender WiFi na
linux'ach trzeba będzie w ten sposób potraktować jeśli chcemy skonfigurować szereg opcji, które
udostępnia nam panel admina.


[1]: http://www.tp-link.com.pl/products/details/TL-WPA4226T-KIT.html
