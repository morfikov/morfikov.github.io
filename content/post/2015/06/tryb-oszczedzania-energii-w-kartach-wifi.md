---
author: Morfik
categories:
- Linux
date: "2015-06-23T10:25:32Z"
date_gmt: 2015-06-23 08:25:32 +0200
published: true
status: publish
tags:
- wifi
- usb
- udev
title: Tryb oszczędzania energii w kartach WiFi na linux
---

Niektóre karty wifi znane są z tego, że niezbyt chcą one działać pod systemem operacyjnym linux. Nie
jest to wina samego sprzętu, ani tym bardziej linuxa, tylko raczej faktu, że producent nie potrafi
napisać pod ten OS odpowiednich sterowników. Czasem jednak pod względem programowym wszystko wydaje
się być w porządku, tj. sterowniki zostały zainstalowane, są one dobrej jakości i karta działa
praktycznie bez zarzutu. Niemniej jednak, strony www wydają się ładować jakoś tak ociężale, z pewnym
opóźnieniem. Jeśli doświadczyliśmy tego typu zachowania ze strony naszej karty wifi, prawdopodobnie
oznacza to, że ma ona włączony tryb oszczędzania energii (powersave).

<!--more-->
## Objawy

Objawy towarzyszące złemu zarządzaniu energią w kartach wifi mogą być podobne do tych, które
występują przy zbyt słabym zasięgu. Możemy jednak rozgraniczyć te dwie kwestie i ustalić czy
problem tkwi w zasięgu czy też leży po stronie zarządzania energią. Najprościej sprawdzić moc
odbieranego sygnału przy pomocy [jednego z narzędzi
linuxowych](https://www.cyberciti.biz/tips/linux-find-out-wireless-network-speed-signal-strength.html).
Poniżej fotka z `wavemon` :

![]({{< baseurl >}}/img/2015/06/1.tryb-oszczedzania-energii-karta-wifi.png#huge)

Jeśli zasięg zdaje się być w porządku, to możemy przeprowadzić inny test polegający na sprawdzeniu
przepustowości łącza internetowego. Jeśli nie mamy wgranego na router alternatywnego firmware
OpenWRT i akurat w tej chwili dysponujemy jedną maszyną kliencką, to pozostaje nam skorzystać, np. z
serwisu [speedtest.net](http://beta.speedtest.net/) :

![]({{< baseurl >}}/img/2015/06/2.tryb-oszczedzania-energii-speedtest.png#small)

Prawdopodobnie, tak jak i w tym przypadku, oba testy nic nie wykażą. Niby wszystko jest w porządku
ale coś jest nie tak. Ostatni test jaki przychodzi mi do głowy to `ping` . Odpytajmy zatem jakąś
domenę:

    $ ping wp.pl
    PING wp.pl (212.77.100.101) 56(84) bytes of data.
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=1 ttl=247 time=19.2 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=2 ttl=247 time=18.3 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=3 ttl=247 time=170 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=4 ttl=247 time=192 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=5 ttl=247 time=181 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=6 ttl=247 time=204 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=7 ttl=247 time=170 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=8 ttl=247 time=240 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=9 ttl=247 time=15.2 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=11 ttl=247 time=16.3 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=12 ttl=247 time=181 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=13 ttl=247 time=204 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=14 ttl=247 time=124 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=15 ttl=247 time=147 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=16 ttl=247 time=170 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=17 ttl=247 time=24.3 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=18 ttl=247 time=17.6 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=19 ttl=247 time=170 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=20 ttl=247 time=192 ms
    ...

Tu już widać istotę rzeczy -- ping skacze, z tym, że parabolicznie. Sprawdźmy zatem czy karta wifi
ma włączony tryb zarządzania energią. Podobnie jak w przypadku sprawdzania siły sygnału, status
zarządzania energią możemy odczytać z wielu miejsc w systemie, w tym przypadku posłużymy się
narzędziem `iw` :

    # iw dev wlan0 get power_save
    Power save: on

## Jak wyłączyć tryb oszczędzania energii

W przypadku gdy mamy jeszcze wątpliwości co do przyczyny problemów z połączeniem bezprzewodowym,
możemy tymczasowo sprawdzić czy wyłączenie trybu oszczędzania energii na coś wpłynie. W tym celu
wpisujemy w terminal te dwa poniższe polecenia:

    # iw dev wlan2 set power_save off
    # iw dev wlan2 get power_save
    Power save: off

W tej chwili ping powinien wrócić do normy i utrzymywać się na niskim poziomie. Jeśli tak się w
istocie stało, to znaczy, że winny był tryb oszczędzania energii.

To powyższe rozwiązanie, jak już wspomniałem, ma efekt tylko tymczasowy, a konkretnie albo do
restartu systemu, albo do wyciągnięcia karty z portu usb. By wyłączyć tryb oszczędzania energii w
kartach wifi całkowicie i przy tym niezależnie od podłączanej karty, musimy [napisać regułkę dla
udev'a]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/). Tworzymy zatem plik
`/etc/udev/rules.d/70-wifi-powersave.rules` i dodajemy w nim poniższą zawartość:

    ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlan*" RUN+="/sbin/iw dev %k set power_save off"

I to w zasadzie rozwiązuje nasz problem.
