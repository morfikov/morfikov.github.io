---
author: Morfik
categories:
- Linux
date: "2015-10-07T15:45:35Z"
date_gmt: 2015-10-07 13:45:35 +0200
published: true
status: publish
tags:
- udev
- hdd/ssd
title: Parametr readahead w dyskach twardych
---

W celu optymalizacji swojej pracy i poprawy wydajności przy transferze danych, dyski twarde często
odczytują więcej sektorów niż było to określone w żądaniu. Chodzi o to, że odczytywany przez nas z
dysku plik jest podzielony na sektory i gdy dysk odczytuje pierwszy sektor tego pliku, to wczytuje
także kilka kolejnych sektorów zlokalizowanych za tym, którego żądanie odczytania zostało właśnie
zrealizowane. Te dodatkowe sektory trafiają do wewnętrznego cache dysku twardego, z którego mogą
zostać odczytane w późniejszym czasie, jeśli zajdzie taka potrzeba. Dostęp do danych w cache jest o
wiele szybszy w porównaniu do repozycjonownia głowicy i odczytywania fizycznych sektorów na dysku. W
efekcie czego mamy zwykle dość znaczny wzrost wydajności przy transferze danych. Ten mechanizm nosi
nazwę **readahead** i w tym wpisie przyjrzymy mu się nieco bliżej.

<!--more-->
## Bufor dysku twardego

Chyba wszystkie produkowane obecnie [dyski twarde posiadają wbudowany
bufor](https://en.wikipedia.org/wiki/Disk_buffer), którego rozmiar waha się w przedziale od 16MiB do
128MiB. Nawet całkiem sporo patrząc po rozmiarze sektora, który ma 512 bajtów. Gdyby dysk odczytywał
jedynie ten sektor, o który jest proszony przez system operacyjny, transfer danych na takim dysku by
znacznie ucierpiał.

Jeśli chcemy wiedzieć jak duży jest bufor naszego dysku, możemy to ustalić posługując się narzędziem
`hdparm` . Jest ono dostępne w debianie w pakiecie pod tą samą nazwą. Rozmiar cache odczytujemy zaś
w poniższy sposób:

    # hdparm -i /dev/sda | grep -i buff
     BuffType=unknown, BuffSize=16384kB, MaxMultSect=16, MultSect=16

Zatem w tym przypadku mamy do czynienia z dyskiem, którego bufor ma 16MiB.

## Ustalenie optymalnej wartości readahead

Optymalna wartość **readahead** zależy od kilku czynników. Przede wszystkim od rozmiaru bufora dysku
twardego. Im jest on większy, tym większą wartość **readahead** powinniśmy ustawić. Trzeba też
pamiętać, że zbyt wysoka wartość będzie powodować pojawianie się śmieci w cache dysku, tj. danych,
które będą odczytywane z kolejnych sektorów ale nigdy nie będą wykorzystywane, czyli będą jedynie
zajmować miejsce w cache, a ten znowu nie jest aż tak wielki. Trzeba się zastanowić jaki rodzaj
plików mamy na dysku. Czy są do duże pliki, np. filmy, czy też jest to raczej dysk systemowy, gdzie
jest pełno małych plików. Im większe są pliki, tym lepiej sprawuje się ustawienie większej wartości
w **readahead**.

Niestety nie da się tak łatwo powiedzieć jaka wartość **readahead** jest odpowiednia dla naszego
dysku i raczej trzeba to ustalić metodą prób i błędów, która zakłada eksperymentowanie z wartościami
tego parametru. Na sam początek ustalmy to jaka jest domyślna wartość **readahead** dla naszego
dysku. Odpalamy zatem `hdparm` :

    # hdparm -a /dev/sda

    /dev/sda:
     readahead     = 256 (on)

W przypadku mojego dysku, jest to `256` dodatkowych sektorów, czyli jakieś 128KiB danych. Sprawdźmy
więc jaki transfer dysk jest w stanie uzyskać przy takim **readahead**:

    # hdparm -Tt /dev/sda

    /dev/sda:
     Timing cached reads:   5194 MB in  2.00 seconds = 2597.34 MB/sec
     Timing buffered disk reads: 272 MB in  3.02 seconds =  90.00 MB/sec

Odczyt danych z tego dysku przy standardowych ustawieniach, to około 90MB/s. Co się zatem stanie gdy
ustawimy **readahead** na `0` ? Sprawdźmy:

    # hdparm -a 0 /dev/sda

    /dev/sda:
     setting fs readahead to 0
     readahead     =  0 (off)

    # hdparm -Tt /dev/sda

    /dev/sda:
     Timing cached reads:   5040 MB in  2.00 seconds = 2520.06 MB/sec
     Timing buffered disk reads:  76 MB in  3.02 seconds =  25.20 MB/sec

Widzimy drastyczny spadek prędkości odczytu danych z dysku. Poprzednio mieliśmy `90MB/s` , zaś teraz
nieco ponad `25MB/s` . Zależność jest w miarę prosta, im większą wartość tutaj ustawimy, tym większą
prędkość odczytu uzyskamy. Jesteśmy jedynie limitowani przez tryb w jakim działa dysk i w tym
przypadku jest to `udma5` :

    # hdparm -i /dev/sda |grep -i udma
     UDMA modes: udma0 udma1 udma2 udma3 udma4 *udma5

[Teoretyczna przepustowość](https://pl.wikipedia.org/wiki/Ultra-DMA) tego dysku wynosi około
100MB/s. Zatem wartość **readahead** powinniśmy zwiększać aż osiągniemy 90-95MB/s przy odczycie
danych. W przypadku tego dysku wartość `256` wydaje się być odpowiednia, choć można by zjechać z nią
do wartości `128` czy nawet `64` , bo nie zanotowałem przy nich jakiejś większej różnicy w prędkości
odczytu. Jedyne co może rzutować na wybór konkretnej wartości, to wielkość plików, które są
zlokalizowane na dysku. Jeśli są to duże porcje danych, np. filmy, to skłaniałbym się do ustawienia
tutaj raczej `256` . Więcej i tak przecie nie ma sensu.

## Reguła dla udev'a

Ustawienia dokonywane za pomocą `hdparm` mają efekt jedynie tymczasowy, a konkretnie do ponownego
uruchomienia danej maszyny. Jeśli chcemy by były one aplikowane na starcie komputera, to musimy
pokusić się o [napisanie reguły dla
udev'a]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/).

Trzeba pamiętać, że określanie dysków po nazwach typu `sda` czy `sdb` może zakończyć się tragicznie
gdy mamy kilka dysków w systemie i z jakiegoś powodu zamienią się one tymi ostatnimi literkami.
Dlatego też lepiej wykorzystać linki znajdujące się w katalogu `/dev/disk/by-id/` . Sprawi to, że
będziemy mieć do czynienia z numerami seryjnymi urządzeń, co czyni je unikalnymi wartościami, w
oparciu o które możemy pisać reguły udev'a i mieć przy tym pewność, że zostaną one zaaplikowane
jedynie do konkretnego dysku twardego.

Tworzymy zatem zatem plik `90-hdparm.rules` w katalogu `/etc/udev/rules.d/` o poniższej treści:

    ACTION=="add", SUBSYSTEM=="block", \
          ENV{ID_SERIAL}=="WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928", \
          RUN+="/sbin/hdparm -a 256 /dev/disk/by-id/ata-WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928"

Jak widać, dopasowujemy urządzenie po numerze seryjnym, ten zaś możemy uzyskać wydając poniższe
polecenie:

    # udevadm info --name /dev/sda | grep -i serial
    E: ID_SERIAL=WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928
    E: ID_SERIAL_SHORT=WD-WX41A90E5928
