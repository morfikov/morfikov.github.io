---
author: Morfik
categories:
- Linux
date: "2015-07-08T20:28:44Z"
date_gmt: 2015-07-08 18:28:44 +0200
published: true
status: publish
tags:
- smart
- hdd/ssd
- western-digital
title: Parkowanie głowicy w dyskach Wstern Digital
---

Dyski zużywają się z różnych powodów. Jednak najczęstszą przyczyną są nowe wynalazki, które
producent w nich implementuje, bo te niezbyt dobrze działają w określonych warunkach, czy też pod
kontrolą pewnych systemów operacyjnych. Tak właśnie jest w przypadku nowszych dysków firmy Western
Digital (WD). Maja one wprowadzony ficzer parkowania głowicy w przypadku, gdy dysk [jest
nieużywany](http://wdc.custhelp.com/app/answers/detail/a_id/5357). Ma to na celu zmniejszyć pobór
prądu i, co za tym idzie, temperaturę urządzenia. Jako, że parkowanie głowicy w dyskach WD nie
działa poprawnie pod moim linux'em (dystrybucja Debian), to nasuwa się pytanie: jak wyłączyć
parkowanie głowicy by wydłużyć żywotność dysku twardego?

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
to ciągle coś jest zapisywane na dysku. Dlatego też, głowica parkuje wiele razy w ciągu jednej
minuty.

## Wartość parametru Load/Unload Cycle

Aktualną wartość parametru odpowiadającego za ilość parkowań głowicy ( `Load/Unload Cycle` ) możemy
odczytać z raportu [S.M.A.R.T](https://pl.wikipedia.org/wiki/S.M.A.R.T._%28informatyka%29) . Poniżej
fotka tabeli parametrów zwróconych przez aplikację `gsmartcontrol` :

![]({{< baseurl >}}/img/2015/07/1.parkowanie-głowicy-smart.png)

Możemy także skorzystać z tekstowego odpowiednika `smartctl` :

    # smartctl --attributes /dev/sda | grep -i load
    193 Load_Cycle_Count        0x0032   196   196   000    Old_age   Always       -       14932

## Narzędzie idle3ctl

Za pomocą inżynierii wstecznej [powstał program idle3ctl](http://idle3-tools.sourceforge.net/), za
pomocą którego jesteśmy w stanie nawet całkowicie wyłączyć parkowanie głowicy. Twórca tego narzędzia
jednak wyraźnie podkreśla, że nie zaleca zupełnego wyłączania parkowania. Poza tym, zawsze trzeba
liczyć się z ryzykiem uszkodzenia sprzętu, bo jakby nie patrzeć, to dokonuje się zmian w firmware
dysku. Ten program jest dostępny w repozytorium dystrybucji Debian w pakiecie `idle3-tools` , z tym,
że należy pamiętać, że to oprogramowanie operuje jedynie na dyskach z rodziny WD.

## Jak wyłączyć parkowanie głowicy

Ja u siebie wyłączyłem kompletnie parkowanie głowicy. Dysk działa bez zarzutu. Jedyne co się rzuca w
oczy to wzrost temperatury o parę stopni, oczywiście nie jest to żadna krytyczna wartość, bo jedyne
36-37 stopni. W każdym razie zamontowałem w sekcji dysków w mojej obudowie wiatraczek, który lekko
mieli powietrze i temperatura dysku została obniżona do 27-29 stopni. Także jest to nawet mniej niż
przed wyłączeniem parkowania, bo wtedy było koło 32.

Dysk widoczny wyżej na fotce ma przepracowane prawie 4K godzin i od jakiegoś czasu ma wyłączone
parkowanie głowicy. Jak możemy zauważyć, ilość parkowań wynosi zaledwie 15K. Jeśli nasz dysk ma w
tym polu wartość wskazującą na paręset tysięcy, to jak najszybciej wyłączmy to parkowanie głowicy.
Możemy to zrobić przez wydanie poniższego polecenie:

    # idle3ctl -d /dev/sda
    # idle3ctl -g /dev/sda
    Idle3 timer is disabled

Zmiany nie wejdą w życie od razu. Trzeba pamiętać, że w końcu dokonywaliśmy zmian w firmwarze
urządzenia i po zapisaniu ustawień trzeba ten dysk odłączyć od zasilania całkowicie. W przeciwnym
wypadku, zmiany nie zostaną zastosowane.
