---
author: Morfik
categories:
- Linux
date: "2016-09-02T17:50:11Z"
date_gmt: 2016-09-02 15:50:11 +0200
published: true
status: publish
tags:
- wifi
- debian
- openbox
- sieć
title: Metryki tras interfejsów eth0 i wlan0 w laptopie (metric)
---

W obecnych czasach posiadanie komputera, który dysponuje kilkoma interfejsami sieciowymi nie jest
niczym niezwykłym. Praktycznie każdy laptop posiada już na pokładzie co najmniej jedną kartę WiFi i
minimum jeden port ethernet. W efekcie czego jesteśmy w stanie podłączyć się do sieci zarówno
przewodowo jak i bezprzewodowo. Problem jednak pojawia się w momencie, gdy chcemy wykorzystywać oba
te interfejsy, z tym, że dysponujemy jedynie niezbyt zaawansowanym menadżerem okien Openbox. Takie
środowiska zwykle nie mają na pokładzie automatów pokroju Network Manager, przez co bardziej
zaawansowana konfiguracja sieci może być dość skomplikowana. Do tej pory wykorzystywałem [interfejs
bond0](/post/konfiguracja-interfejsow-bond-bonding/), by mieć możliwość łatwego
przełączania się miedzy sieciami. Istnieje inny sposób konfiguracji interfejsów `eth0` i `wlan0` w
pliku `/etc/network/interfaces` tak, by działały one nam równolegle i nie powodowały problemów z
połączeniem, a wszystko za sprawą opcji `metric` .

<!--more-->
## Standardowa konfiguracja interfejsów eth0 i wlan0

Konfiguracja sieci w Debianie opartym o menadżer okiem Openbox odbywa się zwykle przez plik
`/etc/network/interfaces` . W przypadku posiadania interfejsu przewodowego, dodajemy tam coś na wzór
poniższego bloku:

    auto eth0
    iface eth0 inet dhcp

Gdy w grę wchodzi interfejs bezprzewodowy, to dodajemy nieco więcej parametrów:

    auto wlan0
    iface wlan0 inet dhcp
        wpa-driver nl80211
        wpa-debug-level -1
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Oczywiście w przypadku tego interfejsu WiFi, wszystkie parametry potrzebne do podłączenia się do
sieci bezprzewodowej są określane w pliku `/etc/wpa_supplicant/wpa_supplicant.conf` i to
`wpasupplicant` zajmuje się tą częścią połączenia. Więcej informacji na ten temat znajduje się w
[artykule o konfiguracji sieci WiFi pod
Debianem](/post/konfiguracja-polaczenia-wifi-pod-debianem/).

## Problem trasy domyślnej (default gateway)

Mając tak skonfigurowane interfejsy sieciowe będziemy mieć problem z połączeniem. Chodzi o to, że
trasa domyślna (default gateway) zostanie przyporządkowana tylko jednemu z interfejsów `eth0` lub
`wlan0` , poniżej przykład:

    # ip route show
    default via 192.168.1.1 dev eth0
    192.168.1.0/24 dev eth0  proto kernel  scope link  src 192.168.1.150
    192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.113

W tym przypadku interfejs `eth0` został skonfigurowany jako pierwszy i to przez niego biegnie trasa
domyślna. W przypadku odłączenia przewodu od tej karty sieciowej, stracimy połączenie i to pomimo
faktu, że karta WiFi jest w dalszym czasie aktywna:

    # ip route show
    default via 192.168.1.1 dev eth0 linkdown
    192.168.1.0/24 dev eth0  proto kernel  scope link  src 192.168.1.150 linkdown
    192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.113

Połączenie możemy przepiąć na interfejs `wlan0` ale musimy usunąć tę powyżej określoną trasę
domyślną i na jej miejsce dodać nową trasę, która będzie wskazywać na interfejs `wlan0` . No tak,
tylko nie chce nam się tego robić ręcznie, a poza tym, możemy posłużyć się metryką (parametr
`metric` ), która skonfiguruje nam trasy domyślne mniej więcej na takiej samej zasadzie co w
przypadku [konfiguracji łącz różnych
ISP](/post/rownowazenie-ruchu-lacz-kilku-isp-load-balancing/).

## Określanie metryki dla interfejsów (metric)

W pliku `/etc/network/interfaces` w każdej zwrotce z konfiguracją interfejsu sieciowego możemy
określić parametr `metric` . Przyjmuje on argument w postaci liczby. Domyślnie jest ona równa `0`
dla każdego interfejsu i to powoduje problemy. W tym przypadku mamy dwa interfejsy i musimy im
określić dwie różne metryki. Im niższy numer podamy w parametrze `metric` , tym wyższy priorytet
nadamy trasie. Przepiszmy zatem nasze dwa interfejsy tak, by zawierały metrykę:

    auto eth0
    iface eth0 inet dhcp
        metric 5

    auto wlan0
    iface wlan0 inet dhcp
        metric 10
        wpa-driver nl80211
        wpa-debug-level -1
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Restartujemy interfejsy za pomocą `ifdown` oraz `ifup` i patrzymy w listing tras:

    # ip route show
    default via 192.168.1.1 dev eth0  metric 5
    default via 192.168.1.1 dev wlan0  metric 10
    192.168.1.0/24 dev eth0  proto kernel  scope link  src 192.168.1.150
    192.168.1.0/24 dev wlan0  proto kernel  scope link  src 192.168.1.113

Jak widzimy wyżej, mamy dwie trasy domyślne. Pakiety będą lecieć przez interfejs `eth0` do momentu
odłączenia przewodu od karty sieciowej. Gdy takie zdarzenie nastąpi, połączenie będzie realizowane
bezprzewodowo do momentu podłączenia przewodu.

Jest to podobny mechanizm co w przypadku interfejsu `bond0` , z tym, że mamy kilka różnić. Przede
wszystkim, tutaj mamy dwa różne adresy IP, a w przypadku `bond0` jest tylko jeden adres. Kolejna
sprawa to problemy przy przełączaniu się między połączeniem przewodowym i bezprzewodowym. Chodzi o
to, że z racji dwóch różnych adresów IP na dwóch interfejsach sieciowych, pakiety będą inaczej
adresowane, tj. będą miały inne źródło. W efekcie czego stracimy nawiązane sesje i będziemy musieli
połączyć się jeszcze raz, np. z serwerem SSH.

Powyżej mieliśmy dwa różne adresy z tej samej sieci 192.168.1.0/24 ale to przez zalogowanie się do
sieci domowej zarówno po kablu jak i bezprzewodowo. Niemniej jednak, oba połączenia są niezależne i
mogą pochodzić nawet od dwóch różnych ISP, z tym, że interfejs `eth0` jest preferowany. Jeśli nam to
nie opowiada, to zawsze możemy pobawić się parametrem `metric` .

## Problemy z połączeniem

Jeśli powyższe ustawienia powodują problemy przy przełączaniu się między łączem przewodowym i
bezprzewodowym, to musimy także dostosować sobie odpowiednio kilka parametrów kernela. W tym celu
edytujemy plik `/etc/sysctl.conf` i dopisujemy w nim te poniższe linijki:

    net.ipv4.conf.all.ignore_routes_with_linkdown = 1
    net.ipv4.conf.default.ignore_routes_with_linkdown = 1
    net.ipv6.conf.all.ignore_routes_with_linkdown = 1
    net.ipv6.conf.default.ignore_routes_with_linkdown = 1

Za ich sprawą, kernel będzie ignorował trasy interfejsów, przez które nie można nawiązać połączenia.
Zapisujemy plik i restartujemy komputer.
