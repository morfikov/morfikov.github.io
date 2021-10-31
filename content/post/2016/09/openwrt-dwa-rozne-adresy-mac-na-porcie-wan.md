---
author: Morfik
categories:
- OpenWRT
date: "2016-09-11T18:42:58Z"
date_gmt: 2016-09-11 16:42:58 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- mac
GHissueID: 362
title: 'OpenWRT: Dwa różne adresy MAC na porcie WAN'
---

Na forum eko.one.pl pojawił się ciekawy temat dotyczący [problemów z adresami MAC w
OpenWRT.](http://eko.one.pl/forum/viewtopic.php?id=14224) Chodzi o to, że by uzyskać połączenie u
pewnych ISP, trzeba im podać adres MAC tego urządzenia, które będzie wpięte bezpośrednio w strukturę
sieci ISP. Niby normalna sprawa ale w pewnych przypadkach, ISP potrafi uwalić połączenie, gdy inne
urządzenie zostanie podpięte do sieci w miejscu starego. Zwykle wystarczy telefon do ISP z prośbą o
aktualizację adresu MAC ale w przypadku firmware OpenWRT może to być ździebko problematyczna
kwestia. Wychodzi na to, że OpenWRT identyfikuje się dwoma adresami MAC na porcie WAN. Jeden z nich
to ten standardowy MAC, który powinien być wykorzystywany i podany ISP. Drugim zaś jest MAC, który
pojawia się przy rozgłaszaniu trybu failsafe podczas fazy startu routera WiFi. Ja nigdy nie
zaobserwowałem problemów z tego powodu. Niemniej jednak, postanowiłem sprawdzić, jak ta sytuacja
dokładnie wygląda i jak sobie z nią poradzić już teraz, na wypadek, gdyby w przyszłości trafił mi
się jeden z takich dziwnych ISP.

<!--more-->
## Dlaczego na porcie WAN są dwa adresy MAC

Z reguły w ofertach producentów routerów możemy wyczytać, że dany model ma jeden port WAN i 4 porty
LAN. Niemniej jednak, szereg takich urządzeń ma tylko jeden fizyczny switch z wydzielonymi portami
WAN i LAN. Zatem jest to jeden układ i ma on przypisany tylko jeden adres MAC. Później w fazie
konfiguracji tych portów, część przeznaczona na port WAN otrzymuje swój własny adres MAC, podobnie
jak i cześć LAN.

Te dwa adresy różnią się nieznacznie, zwykle o 1. Dla przykładu mam tutaj [router
TL-WR1043ND](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) v2 od TP-LINK. Po
zalogowaniu się na to urządzenie via SSH/telnet, jesteśmy w stanie odczytać adresy MAC jego
interfejsów sieciowych za pomocą narzędzi `ip`/`ifconfig` :

    E8:94:F6:68:79:F0   eth1
    E8:94:F6:68:79:F1   eth0

W tym przypadku port WAN to interfejs `eth0` , a porty LAN to interfejs `eth1` . Na obudowie routera
z kolei można się doszukać tylko jednego adresu MAC: E8:94:F6:68:79:F0 . Zatem możemy przypuszczać,
że switch w routerze TL-WR1043ND ma standardowo ustawiony ten adres MAC, a dla portu WAN jest on
dobierany przez podbicie numeru o 1. I tu właśnie zaczynają się problemy z trybem failsafe.

## Tryb failsafe i adres MAC na porcie WAN

[Failsafe to tryb ratunkowy wykorzystywany w firmware
OpenWRT](/post/tryb-ratunkowy-failsafe-w-openwrt/) w przypadkach błędów natury
programowej (błędna konfiguracja systemu routera). Gdy router jest uruchamiany lub restartowany,
zgłasza on możliwość wejścia w tryb failsafe w pewnym określonym momencie. Ta gotowość jest
potwierdzana pakietem UDP wysyłanym z adresu routera (192.168.1.1) na adres rozgłoszeniowy sieci
(192.168.1.255). Ten pakiet zawiera także docelowy adres MAC FF:FF:FF:FF:FF:FF, czyli każda maszyna
w danym segmencie sieci odbierze ten pakiet.

Niemniej jednak, jako że nasz router we wczesnej fazie startu nie ma jeszcze skonfigurowanego
switch'a, to ten pakiet zostaje podpisany adresem MAC, który jest nadrukowany na routerze. W tym
przypadku jest to E8:94:F6:68:79:F0, który po konfiguracji zostanie przypisany portom LAN, a dla WAN
zostanie wygenerowany nowy MAC. W efekcie ISP najpierw widzi pakiet failsafe z jednym adresem MAC, a
po skonfigurowaniu switch'a, router przesyła prośbę o konfigurację sieci przez protokół DHCP już
portem WAN, który ma inny adres MAC. Wygląda to mniej więcej tak:

![rozny-adres-mac-isp-openwrt-wireshark](/img/2016/09/1.rozny-adres-mac-isp-openwrt-wireshark.png#huge)

Tutaj mój laptop był wpięty do portu WAN routera, by z perspektywy ISP ustalić jakie pakiety do
niego trafiają. Widzimy, że pierwszy pakiet jest zgłoszeniem gotowości trybu failsafe. Zwróćmy uwagę
na kolumnę Src MAC, czyli na źródłowy adres MAC. Ten pierwszy pakiet jest jedynym, który ma inny MAC
niż wszystkie pozostałe i stąd się bierze problem.

## Jak rozwiązać problem z różnymi adresami MAC

Przede wszystkim, ten problem z podwójnym MAC na porcie WAN zdaje się dotyczyć tylko firmware
OpenWRT. Na oryginalnym firmware producenta taka sytuacja nie występuje. By jakoś wybrnąć z tej
sytuacji, [część użytkowników](http://eko.one.pl/forum/viewtopic.php?id=10180) skłania się do
edycji/rekompilacji obrazu firmware. Można, np. [wyłączyć generowanie pakietu
failsafe](https://gist.github.com/zhangxc/944446).

Ja zbytnio jeszcze nigdy nie kompilowałem własnego obrazu i raczej nie prędko sobie jakiś zrobię.
Niemniej jednak, dla mnie rekompilacja obrazu, by zmienić adres MAC na porcie WAN, czy wyłączyć
pakiet failsafe, to lekka przesada. Musi istnieć jakieś prostsze rozwiązanie tego całego problemu.

W pliku `/etc/config/network` mamy bloki konfigurujące interfejsy sieciowe. Mamy tam zarówno
interfejs `eth0` jak i `eth1` . Można je tak skonfigurować, by nadać im odpowiednie adresy MAC. W
przypadku routera TL-WR1043ND konfiguracja wygląda mniej więcej tak:

    config interface 'lan'
        option ifname 'eth1'
    ...
        option macaddr 'E8:94:F6:68:79:F1'

    config interface 'wan'
        option ifname 'eth0'
    ...
        option macaddr 'E8:94:F6:68:79:F0'

W opcji `macaddr` podajemy dla interfejsu WAN ten adres MAC, który jest zawarty w pakiecie failsafe.
Przepisujemy również MAC dla portów LAN. Po restarcie routera, interfejsy sieciowe powinny mieć
poniższe adresy MAC:

    # ifconfig
    br-lan    Link encap:Ethernet  HWaddr E8:94:F6:68:79:F1
    ...
    eth0      Link encap:Ethernet  HWaddr E8:94:F6:68:79:F0
    ...
    eth1      Link encap:Ethernet  HWaddr E8:94:F6:68:79:F0
    ...

Może i interfejs `eth0` i `eth1` mają te same MAC ale wirtualny interfejs mostka ma już ten MAC,
który ustawiliśmy w pliku `/etc/config/network` w sekcji `LAN` . Nie wiem czy takie ustawienie w
czymś przeszkadza ale jeśli teraz podepniemy się do portu WAN routera i ponownie sprawdzimy jak
wygląda komunikacja routera z ISP, to zobaczymy coś takiego:

![rozny-adres-mac-isp-openwrt-wireshark](/img/2016/09/2.rozny-adres-mac-isp-openwrt-wireshark.png#huge)

Jak widzimy, teraz już wszystkie pakiety mają jeden i ten sam adres MAC, co powinno rozwiązać
kwestię problemów z podwójnym adresem MAC po stronie ISP.
