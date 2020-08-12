---
author: Morfik
categories:
- OpenWRT
date: "2016-11-01T18:14:53Z"
date_gmt: 2016-11-01 17:14:53 +0100
published: true
status: publish
tags:
- wifi
- chaos-calmer
- lte
- router
- tethering
title: Udostępnianie LTE/3G ze smartfona przez router OpenWRT (tethering)
---

Przeglądając [forum eko.one.pl natrafiłem ciekawy
problem](http://eko.one.pl/forum/viewtopic.php?pid=175547), nad którym też się zastanawiałem jakiś
czas temu. Chodzi o udostępnienie internetu komórkowego (LTE/3G) komputerom w domowej sieci za
pomocą smartfona (tzw. [tethering](https://pl.wikipedia.org/wiki/Tethering)). W takiej sytuacji, w
przypadku problemów z lokalnym dostawcą internetu moglibyśmy przepiąć wszystkie komputery na
internet świadczony przez operatora GSM, z którego korzystamy. Z reguły standardowy firmware
routerów WiFi nie pozwala na tego typu rozwiązania. Niemniej jednak, mając do dyspozycji router z
OpenWRT można spróbować połączyć go z naszym smartfonem udostępniając sieci lokalnej internet
LTE/3G. W tym artykule zostanie przedstawione tego typu rozwiązanie przy wykorzystaniu [routera
Archer C7](http://www.tp-link.com.pl/en/products/details/cat-9_Archer-C7.html) v2 od TP-LINK oraz
[smartfona Neffos C5](http://www.neffos.com/en/product/details/C5), również od TP-LINK. Na routerze
zaś jest wgrana najnowsza stabilna wersja OpenWRT (Chaos Calmer). Sprawdzimy sobie jak takie
rozwiązanie wygląda oraz sprawuje się w praktyce i czy jest ono w ogóle godne jakiegoś większego
zainteresowania.

<!--more-->
## Problemy z tethering'iem

Domowe sieci komputerowe cechuje różnorodność implementacji i wykorzystywanych rozwiązań. Nie
wszystkie z tych sieci wykorzystują router WiFi. Być może też taki router jest na wyposażeniu ale z
jakiegoś powodu nie działa jego funkcjonalność odpowiedzialna za połączenie bezprzewodowe. Nawet
jeśli mamy w pełni sprawny router WiFi, to i tak w dalszym ciągu szereg komputerów w naszym domu
może się łączyć przewodowo, a to ze względu na zbyt słaby sygnał odbierany z routera, lub też taka
stacja zwyczajnie nie ma bezprzewodowej karty sieciowej.

Komputery, które mamy w domu, możemy zatem podzielić z grubsza na dwie grupy: te posiadające
możliwość łączenia się bezprzewodowego oraz te, które takiej możliwości nie posiadają. W przypadku
tej pierwszej klasy urządzeń, implementacja tethering'u jest w miarę prostym zdaniem, bo smartfony
są w stanie stworzyć punkt dostępowy sieci WiFi, do którego można bezprzewodowo podpiąć komputery.
Problem jednak pojawia się w przypadku tej drugiej grupy, gdzie urządzeń zwykle nie ma jak podłączyć
do smartfona. Tu właśnie może znaleźć zastosowanie router WiFi, który byłby w stanie skomunikować
się ze smartfonem za pomocą sieci WiFi czy też przez port USB i udostępnić jego połączenie LTE/3G
wszystkim hostom w sieci lokalnej.

Tethering wymaga specyficznego sprzętu. Przede wszystkim, potrzebny nam jest smartfon, który ma
możliwość łączenia się po WiFi. Obecnie większość modeli na rynku oferuje tego typu funkcjonalność.
Taki smartfon musi mieć także modem LTE/3G, by można było mówić o jakimś rozsądnym połączeniu ze
światem, no i też by uniknąć zbyt wysokich rachunków za wykorzystane pakiety danych. Jeśli chodzi
zaś o routery WiFi, to część z nich wspiera tryb WISP (Wireless ISP). Nie są to jednak wszystkie
modele. Porty USB przy zastosowaniu stock'owego firmware są dla nas bezużyteczne. Dlatego też
przydałoby się by router miał zainstalowane alternatywne oprogramowanie na bazie OpenWRT. Mając
spełnione te warunki, mamy punkt zaczepienia i możemy zacząć konfigurować połączenie na bazie
tethering'u.

Będąc zaopatrzonym w smartfon, który wspiera tethering, trzeba mieć na uwadze kilka istotnych
rzeczy. Większość modeli tych urządzeń dostępnych na rynku oferuje tylko jeden strumień transferu
danych i to zwykle w paśmie 2,4 GHz. Podłączenie wielu komputerów po WiFi do takiego smartfona jest
co prawda możliwe ale jakość połączenia może nie być najlepsza zwłaszcza, gdy w sąsiedztwie mamy
bardzo zaszumiony eter. Oczywiście w dalszym ciągu taki tethering jest lepszym wyjściem niż
kompletny brak internetu, z tym, że nie spodziewajmy się cudów po tym rozwiązaniu.

Kolejna sprawa, to spora utylizacja baterii smartfona przy włączonym WiFi i LTE, zwłaszcza w
przypadku aktywnego transferu danych. Dlatego też tego typu rozwiązanie znajduje zastosowanie raczej
jedynie jako backup, a nie jako główne połączenie z internetem mające za zadanie zastąpić łącze
stacjonarne.

## Jak włączyć tethering na smartfonie Neffos C5

Jako, że ja dysponuję smartfonem Neffos C5, który ma na pokładzie system Android 5.1 (Lollipop), to
w oparciu o to urządzenie opiszę jak włączyć i skonfigurować tethering. Potrzebne nam opcje są w
menu pod Ustawienia =\> Więcej =\> Tethering i punkt
dostępu.

[![001.tethering-smartfon-router-openwrt-lte-wlaczenie]({{< baseurl >}}/img/2016/11/001.tethering-smartfon-router-openwrt-lte-wlaczenie-660x361.png)]({{< baseurl >}}/img/2016/11/001.tethering-smartfon-router-openwrt-lte-wlaczenie.png)

Wyżej na fotce widzimy kilka opcji, z których nas interesują głownie dwie: Hotspot WLAN i Tethering
przez USB. W zależności od możliwości naszej sieci domowej będziemy korzystać z jednego lub drugiego
rozwiązania. Poniżej zostanie opisana konfiguracja obu tych połączeń.

### Tethering po WiFi

Jako, że tethering najprościej jest skonfigurować za pomocą sieci WiFi, to od tego rozwiązania
zaczniemy. W tym przypadku niekoniecznie potrzebujemy router WiFi z OpenWRT. Na dobrą sprawę, to
bezprzewodowy router w ogóle nie jest nam potrzebny, no chyba, że część komputerów w naszej sieci
domowej nie jest w stanie połączyć się bezprzewodowo. Zacznijmy zatem od skonfigurowania tethering'u
w smartfonie. By to zrobić, włączamy bezprzewodowy punkt dostępowy
WiFi:

[![002.tethering-smartfon-router-openwrt-lte-konfiguracja]({{< baseurl >}}/img/2016/11/002.tethering-smartfon-router-openwrt-lte-konfiguracja-660x361.png)]({{< baseurl >}}/img/2016/11/002.tethering-smartfon-router-openwrt-lte-konfiguracja.png)

W celu oszczędzania baterii możemy ustawić automatyczne wyłączanie WiFi po 5 lub 10 minutach
bezczynności. Nas jednak bardziej interesuje konfiguracja zabezpieczeń. Opcji nie ma zbyt wiele i
jedyne co musimy ustawić to hasło do sieci WiFi (WPA2-PSK). Jeśli nie odpowiada nam domyślny ESSID,
to również możemy go zmienić. Jak widać wyżej, mamy też możliwość określenia liczby urządzeń, które
będą w stanie nawiązać połączenie ze smartfonem w tym samym czasie (maksymalnie 8). Mając
skonfigurowany hotspot WiFi możemy go
uruchomić:

[![003.tethering-smartfon-router-openwrt-lte-hotspot]({{< baseurl >}}/img/2016/11/003.tethering-smartfon-router-openwrt-lte-hotspot-401x660.png)]({{< baseurl >}}/img/2016/11/003.tethering-smartfon-router-openwrt-lte-hotspot.png)

Poniżej znajdują się sytuacje, w których człowiek może się znaleźć przy udostępnianiu połączenia
internetowego komputerom za pomocą smartfona.

#### Podłączanie komputera

Pierwszy scenariusz zastosowania tethering'u zakłada wykorzystanie pojedynczego komputera, który
jest w stanie łączyć się bezprzewodowo (posiada kartę WiFi). Może to być laptop, desktop albo też
inny smartfon. Podłączenie takiej maszyny jest realizowane w taki sam sposób, co w przypadku
standardowej sieci WiFi. Wystarczy wpisać dane logowania do AP udostępnianego przez smartfon (ESSID
i hasło) i możemy już korzystać z połączenia internetowego oferowanego przez operatora GSM na
komputerze. Nie musimy nic dodatkowo konfigurować, bo proces jest automatyczny.

#### Podłączanie routera WiFi z oficjalnym firmware TP-LINK

W przypadku, gdy nasz router TP-LINK wspiera tryb WISP na stock'owym firmware, to w zasadzie,
konfiguracja połączenia jest mniej więcej taka sama jak w przypadku każdego innego WISP. Jedynym z
moich routerów, które są w stanie skonfigurować połączenie WISP na oficjalnym firmware jest
[TL-MR3020](http://www.tp-link.com.pl/products/details/cat-14_TL-MR3020.html) v1. Nie jest to może
pełnowymiarowy router ale posiada jeden port RJ-45, przez co jesteśmy w stanie podłączyć do tego
routera przewodowo co najmniej jeden komputer. Co najmniej, bo zawsze można podpiąć zwykły switch i
rozdzielić sygnał na kilka maszyn. Konfiguracja klienta WiFi sprowadza się do uzupełnienia
poniższego formularza w panelu
administracyjnym:

[![004.tethering-smartfon-router-openwrt-lte-tp-link-stock]({{< baseurl >}}/img/2016/11/004.tethering-smartfon-router-openwrt-lte-tp-link-stock-657x660.png)]({{< baseurl >}}/img/2016/11/004.tethering-smartfon-router-openwrt-lte-tp-link-stock.png)

Zatem nie jest to jakiś skomplikowany proces. Ważne jest tylko, by router wspierał tryb WISP, bo bez
niego nie damy rady podłączyć routera do smartfona, no chyba, że mamy na nim wgrany alternatywny
firmware OpenWRT.

#### Podłączanie routera WiFi z firmware OpenWRT

Praktycznie każdemu routerowi WiFi mającemu firmware OpenWRT można przełączyć czip bezprzewodowy w
tryb klienta przy jednoczesnym zachowaniu funkcjonalności płynącej z trybu AP. W takim przypadku
karta działa w dwóch trybach jednocześnie. W ten sposób tryb AP nasłuchuje zapytań od klientów sieci
WiFi, a tryb STA łączy router do smartfona, mniej więcej w taki sam sposób jak każdy inny komputer.
Informacje na temat [konfiguracji trybu STA na OpenWRT są
tutaj]({{< baseurl >}}/post/konfiguracja-wisp-openwrt-tryb-sta-ap/). Musimy tylko podać
odpowiednie dane logowania do AP na smartfonie, tj. ESSID i hasło.

Przy trybie WISP routerów trzeba liczyć się ze spadkiem wydajności o połowę przy przesyle danych
przez sieć bezprzewodową. Oczywiście obecnie routery WiFi potrafią dysponować dość znacznymi
prędkościami wielokrotnie przewyższającymi pełne możliwości smartfonów. Niemniej jednak, w
przypadku tych mniejszych routerów trzeba brać poprawkę na tę kwestię. Chodzi o to, że router będzie
musiał odbierać dane od smartfona po WiFi i transferować taką samą porcję danych do swoich klientów
również po WiFi. Dlatego właśnie sumaryczny transfer danych na routerze po WiFi dwukrotnie
przekroczy transfer po LTE na smartfonie. Ten problem nie występuje, gdy korzystamy z tethering'u po
USB.

### Tethering po USB w OpenWRT

Innym podejściem do udostępniania połączenia LTE/3G za pomocą smartfona w sieci domowej jest
podłączenie [telefonu do portu USB routera
WiFi](https://wiki.openwrt.org/doc/howto/usb.tethering). W przypadku oficjalnego firmware routera,
takie rozwiązanie raczej odpada i pozostaje nam jedynie sprzęt wspierany przez OpenWRT. Mając taki
router, trzeba na nim zainstalować odpowiednie pakiety. W sumie powinien wystarczy
`kmod-usb-net-rndis` oraz jego zależności:

    # opkg update
    # opkg install kmod-usb-net-rndis

Teraz już wystarczy tylko podłączyć smartfon do portu USB routera i aktywować w nim tethering. W
logu systemowym powinien się nam pojawić poniższy komunikat:

    kern.info kernel: [  614.470000] usb 2-1: new high-speed USB device number 2 using ehci-platform
    kern.info kernel: [  621.220000] usb 2-1: USB disconnect, device number 2
    kern.info kernel: [  621.630000] usb 2-1: new high-speed USB device number 3 using ehci-platform
    kern.info kernel: [  621.780000] rndis_host 2-1:1.0 usb0: register 'rndis_host' at usb-ehci-platform.1-1, RNDIS device, 0a:4b:e9:55:cb:19

W systemie routera mamy dostępny nowy interfejs sieciowy `usb0` . Musimy go skonfigurować w pliku
`/etc/config/network` . Można tylko przepisać istniejące bloki od konfiguracji WAN, poniżej
przykład:

    config interface 'wan'
    #   option ifname 'eth0'
        option ifname 'usb0'
        option proto 'dhcp'

    config interface 'wan6'
    #   option ifname 'eth0'
        option ifname 'usb0'
        option proto 'dhcpv6'

Zapisujemy plik i restartujemy router. Połączenie na linii komputer \<=\> router WiFi \<=\> smartfon
\<=\> LTE/3G powinno zostać zestawione. Można puścić `ping` dla pewności ale nie powinno być
problemów.

![]({{< baseurl >}}/img/2016/11/005.tethering-smartfon-router-openwrt-lte-test.png)

Tam na wiki OpenWRT można przeczytać, że mogą się pojawić problemy przy podłączaniu smartfona do
portu USB czy wyłączaniu w opcjach Androida tethering'u. Rzekomo OpenWRT ma mieć problem z ponownym
nawiązaniem połączenia po aktywacji tethering'u. Mój Neffos C5 i router Archer C7 v2 nie wykazują
takich dziwnych zachowań. Poniżej jest log z rozłączenia
tethering'u:

[![006.tethering-smartfon-router-openwrt-lte-log]({{< baseurl >}}/img/2016/11/006.tethering-smartfon-router-openwrt-lte-log-660x287.png)]({{< baseurl >}}/img/2016/11/006.tethering-smartfon-router-openwrt-lte-log.png)

Podobnie sprawa wygląda po wyciągnięciu wtyczki z portu USB routera.
