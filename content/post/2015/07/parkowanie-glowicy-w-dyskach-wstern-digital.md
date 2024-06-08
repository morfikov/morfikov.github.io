---
author: Morfik
categories:
- Linux
date:    2015-07-08 20:28:44 +0200
lastmod: 2024-06-08 13:55:00 +0200
published: true
status: publish
tags:
- smart
- hdd
- western-digital
- smart
- intellipark
- intellipower
- udev
- hdparm
GHissueID: 152
title: Parkowanie głowicy w dyskach HDD Wstern Digital (Load_Cycle_Count)
---

Dyski zużywają się z różnych powodów. Jednak najczęstszą przyczyną są nowe wynalazki, które
producent w nich implementuje, bo te niezbyt dobrze działają w określonych warunkach, czy też pod
kontrolą pewnych systemów operacyjnych. Tak właśnie jest w przypadku dysków HDD firmy Western
Digital (WD). Maja one wprowadzony ficzer parkowania głowicy w przypadku, gdy dysk [jest
nieużywany][1] (tzw. IntelliPark/IntelliPower). Ma to na celu zmniejszyć pobór prądu i, co za tym
idzie, temperaturę urządzenia oraz też uodpornić nieco taki nośnik na różnego rodzaju
wstrząsy/przeciążenia. Parkowanie głowicy w dyskach HDD nie działa poprawnie pod linux'em
(dystrybucja Debian), tj. parametr SMART `Load_Cycle_Count` po krótkim czasie użytkowania takiego
dysku może osiągać wartości idące w setki tysięcy. Nasuwa się więc pytanie, czy jesteśmy jakoś w
stanie wyłączyć to parkowanie głowicy, by wydłużyć żywotność dysku twardego?

<!--more-->
## Parkowanie głowicy zabija dyski pod linux'em

Wszystko by było w porządku gdyby nie fakt, że dysk ma skończoną żywotność, którą nawet podaje
producent, np. 500K parkowań głowicy. Po przekroczeniu tego pułapu, dysk prawdopodobnie się rozleci.
Mój najstarszy dysk, który ma przepracowane już ponad 65K godzin, nie obsługuje parkowania głowicy i
pewnie dlatego wciąż działa. Za to jeden z moich nowszych dysków WDC WD15EARS-00MVWB0 w nieco ponad
rok zaparkował już 350K razy, co jest bardzo przygnębiające i gdyby ten dysk pochodził tak jeszcze z
rok, to prawdopodobnie by zdechł.

Jak to się dzieje, że producent WD implementuje takie coś w dyskach? Odpowiedź pewnie jest
oczywista, by skrócić żywotność dysku. Można wnioskować to choćby po braku wypuszczenia
odpowiedniego oprogramowania, które by ułatwiło dostosowanie czasu po jakim głowica może zaparkować.
Obecnie ten czas jest ustawiony na minimum, a biorąc pod uwagę specyfikację linux'a oraz jego logi,
to ciągle coś jest zapisywane na dysku. Dlatego też, głowica może nawet parokrotnie zaparkować w
ciągu jednej minuty.

## Aktualna wartość parametru Load_Cycle_Count

Aktualną wartość parametru odpowiadającego za ilość parkowań głowicy
( `Load/Unload Cycle` / `Load_Cycle_Count` ) możemy odczytać z raportu [SMART][2]. Poniżej fotka
tabeli parametrów zwróconych przez aplikację `gsmartcontrol` dla jednego z moich dysków HDD:

![parkowanie-glowicy-smart](/img/2015/07/1.parkowanie-glowicy-smart.png#huge)

Możemy także skorzystać z tekstowego odpowiednika `smartctl` :

    # smartctl --attributes /dev/sda | grep -i load
    193 Load_Cycle_Count        0x0032   196   196   000    Old_age   Always       -       14932

## Narzędzie idle3ctl

Za pomocą inżynierii wstecznej [powstał program idle3ctl][3], za pomocą którego jesteśmy w stanie
nawet całkowicie wyłączyć parkowanie głowicy. Twórca tego narzędzia jednak wyraźnie podkreśla, że
nie zaleca zupełnego wyłączania parkowania. Poza tym, zawsze trzeba liczyć się z ryzykiem
uszkodzenia sprzętu, bo jakby nie patrzeć, to dokonuje się zmian w firmware dysku. Ten program jest
dostępny w repozytorium dystrybucji Debian w pakiecie `idle3-tools` , z tym, że należy pamiętać, że
to oprogramowanie operuje jedynie na dyskach z rodziny WD.

## Jak wyłączyć parkowanie głowicy

Ja u siebie wyłączyłem kompletnie parkowanie głowicy. Dysk działa bez zarzutu. Jedyne co się rzuca w
oczy to wzrost temperatury o parę stopni, oczywiście nie jest to żadna krytyczna wartość, bo jedyne
36-37 stopni. W każdym razie zamontowałem w sekcji dysków w mojej obudowie wiatraczek, który lekko
mieli powietrze i temperatura dysku została obniżona do 27-29 stopni. Także jest to nawet mniej niż
przed wyłączeniem parkowania, bo wtedy było około 32 stopni.

Dysk widoczny wyżej na fotce ma przepracowane prawie 4K godzin i od jakiegoś czasu ma wyłączone
parkowanie głowicy. Jak możemy zauważyć, ilość parkowań wynosi zaledwie 15K. Jeśli nasz dysk ma w
tym polu wartość wskazującą na paręset tysięcy, to jak najszybciej wyłączmy to parkowanie głowicy.
Możemy to zrobić przez wydanie poniższego polecenia:

    # idle3ctl -d /dev/sda

Zmiany nie wejdą w życie od razu. Trzeba pamiętać, że w końcu dokonywaliśmy zmian w firmware
dysku HDD i po zapisaniu ustawień trzeba taki nośnik odłączyć od zasilania całkowicie. W przeciwnym
wypadku, zmiany nie zostaną zastosowane (restart systemu nie jest wystarczający).

## Jak sprawdzić czy parkowanie głowicy jest wyłączone

Po dokonaniu zmian i wykonaniu cyklu zasilania naszego dysku HDD, uruchamiamy system i przy pomocy
`idle3ctl` lub `hdparm` sprawdzamy czy zegar odliczający sekundy do zaparkowania głowicy został
wyłączony:

    # hdparm -J /dev/sdb

    /dev/sdb:
     wdidle3      = disabled

    # idle3ctl -g /dev/sdb
    Idle3 timer is disabled

Jak widać w obu przypadkach mamy informację `disabled` , zatem głowica dysku nie powinna już
parkować.

## Parkowanie głowicy w nowszych HDD od WD oraz dyskach innych producentów

Rozglądając się za kupnem nowego dysku HDD o rozmiarze 2TB+, chciałem ponownie zdecydować się na
dysk od producenta Western Digital. Problem jednak w tym, że [jak można przeczytać choćby
tutaj][4], WD wprowadził najwyraźniej zmiany w firmware tych nowszych modeli swoich dysków HDD i
nie damy rady już przy pomocy `idle3ctl` wyłączyć (czy też zdefiniować czasu) tego zegara
odliczającego czas do zaparkowania głowicy. Zatem trzeba się liczyć z faktem, że opisany w tym
artykule sposób na poprawę żywotności dysku już w zasadzie nie działa. Próba odczytania wartości
tego zegara skutkuje błędami:

    # hdparm -J /dev/sdb

    /dev/sdb:
    SG_IO: bad/missing sense data, sb[]:  70 00 0b 00 00 00 00 0a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

     wdidle3      = disabled

    # idle3ctl -g /dev/sdb
    sg16(VSC_SENDKEY) failed: Invalid exchange

Podobnie sprawa wygląda w przypadku dysków HDD innych producentów, którzy również tego typu
mechanizm parkowania głowicy zaimplementowali w swoich nośnika pamięci masowej.

## Istnieje jakiś sposób na wyłączenie parkowania głowicy?

Próbowałem w zasadzie szeregu metod, które miały na celu ujarzmić mechanizmy oszczędzania energii w
dyskach HDD ale w zasadzie wszystkie z tych rozwiązań nie przyniosły oczekiwanego efektu, jakim
było zaprzestanie zwiększania wartości parametru SMART `193 Load_Cycle_Count` przez dysk HDD.

### UDEV i hdparm

Jednym z popularniejszych rozwiązań, o którym można poczytać na internetach, jest próba zmuszenia
dysku HDD przy pomocy `hdparm` , by odpuścił on sobie zarządzanie energią. Możemy to zrobić w
następujący sposób:

    # /hdparm -B 254 -S 0 /dev/sdb

    /dev/sdb:
     setting Advanced Power Management level to 0xfe (254)
     setting standby to 0 (off)
     APM_level      = 254

To powyższe polecenie nie daje trwałego efektu i po restarcie maszyny wartości zostaną zresetowane
do domyślnych. Lepszym rozwiązaniem jest napisać prostą regułę dla UDEV'a. Tworzymy zatem plik
`/etc/udev/rules.d/90-hdparm.rules` i dodajemy do niego poniższą zawartość:

    ACTION=="add", SUBSYSTEM=="block", KERNEL=="[sh]d[a-z]", ATTRS{queue/rotational}=="1", \
            ENV{ID_MODEL}=="TOSHIBA_HDWL120", \
            RUN+="/sbin/hdparm -B 254 -S 0 /dev/%k"

Ta powyższa reguła ma na celu uruchomić polecenie `hdparm` z opcjami konfigurującymi Advanced Power
Management ( `-B 254` ) oraz idle/low-power mode ( `-S 0` ) na konkretnym modelu dysku HDD. Mamy
określone w tej regule, że interesuje nas dysk talerzowy `ATTRS{queue/rotational}=="1"` oraz model
dysku `TOSHIBA_HDWL120` . Model naszego dysku HDD możemy odczytać w poniższy sposób:

    # udevadm info /dev/sdb
    ...
    E: ID_MODEL=TOSHIBA_HDWL120
    ...

W poleceniu `hdparm` zostały określone dwa parametry `-B` oraz `-S` . W przypadku parametru `-B` ,
im niższą wartość ustawimy, tym bardziej agresywniejsze będzie zarządzanie energią, tj. dysk
częściej będzie się wyłączał (zatrzymywał talerze i parkował głowice). Można tutaj ustawić wartości
`0-255` , z czego `255` odpowiada za całkowite wyłączenie zarządzania energią, co może nie być
wspierane przez wszystkie dyski, dlatego też lepszym rozwiązaniem jest skorzystanie z bardziej
uniwersalnej wartości `254` , co powinno zapewnić najwyższą wydajność kosztem większego poboru
prądu. Jeśli zaś chodzi o parametr  `-S` , to wartość `0` oznacza, że dysk nie powinien przechodzić
w tryb idle/low-power.

Po zapisaniu pliku przeładowujemy politykę UDEV'a i testujemy regułę:

    # udevadm control --reload-rules

    # udevadm test /dev/sdb
    ...
    run: '/sbin/hdparm -B 254 -S 0 /dev/sdb'

Restartujemy system (lub korzystamy z `udevadm trigger` ) i sprawdzamy, czy `APM_level` został
ustawiony na `254` :

    # udevadm trigger --type=devices --action=change

    # hdparm -B /dev/sdb

    /dev/sdb:
     APM_level      = 254

Może i wartości zostały ustawione poprawnie ale nijak to wpływa na parkowanie głowicy, przynajmniej
w przypadku mojego dysku HDD.

### Odpytywanie parametrów SMART

Szukając dalej natrafiłem na sposób z odpytywaniem co kilka sekund parametrów SMART. Chodzi o to,
że jeśli będziemy co jakiś czas wysyłać żądania do dysku, to ten zegar od parkowania głowicy będzie
się resetował, przez co głowica już nie powinna parkować. Najlepiej jest sobie napisać prosty
skrypt i dorobić do niego prostą usługę systemd.

Poniżej przykład skryptu:

    # cat stop-hdd-sleep.sh

    #!/bin/sh

    while true; do sleep 10 && smartctl -g apm /dev/disk/by-id/ata-TOSHIBA_HDWL120_209ZTE8DT > /dev/null ; done

Poniżej przykładowa usługa systemd:

    # cat stop-hdd-sleep.service
    [Unit]
    Description=Stop HDD Sleep Service
    After=multi-user.target
    [Service]
    Type=simple
    Restart=always
    ExecStart=/bin/sh -c "exec /usr/local/bin/stop-hdd-sleep.sh"
    [Install]
    WantedBy=multi-user.target

Zamiast literek `sda` czy `sdb` , lepszym rozwiązaniem jest odwołanie się do dysku po jego ID
(katalog `/dev/disk/by-id/` ).

#### Dobór czasu

Ten powyższy sposób zdaje się działać. Niemniej jednak, trzeba sobie dobrać czas odpytywania
parametrów SMART. W przypadku jednych dysków HDD będzie on tak krótki jak kilka sekund, w przypadku
innych może być dłuższy, np. kilkadziesiąt sekund albo nawet i parę minut. Trzeba metodą prób i
błędów ustalić po jakim czasie głowica parkuje, gdy dysk jest bezczynny. Najprościej jest co jakiś
czas wydać w terminalu polecenie `smartctl` i weryfikować czy wartość parametru
`193 Load_Cycle_Count` ulega zwiększeniu:

    # smartctl -x /dev/sdb | egrep 193
    193 Load_Cycle_Count        -O--CK   096   096   000    -    40332

W pewnym momencie wartość `193 Load_Cycle_Count` nie powinna się zmieniać i wtedy dostosowujemy w
tym skrypcie wartość `sleep 10` , gdzie 10 oznacza ilość sekund bezczynności.


[1]: http://wdc.custhelp.com/app/answers/detail/a_id/5357
[2]: https://pl.wikipedia.org/wiki/S.M.A.R.T._%28informatyka%29
[3]: http://idle3-tools.sourceforge.net/
[4]: https://superuser.com/questions/1755007/is-the-idle3-timeout-mess-still-relevant-on-western-digital-wd-blue-hard-drive
