---
author: Morfik
categories:
- Linux
date: "2016-02-02T16:21:08Z"
date_gmt: 2016-02-02 15:21:08 +0100
published: true
status: publish
tags:
- hdd
- ssd
- lvm
title: Jak dodać nowy dysk do LVM
---

Rozmiary obecnych dysków twardych zwykliśmy już liczyć w TiB. Jest to dość sporo ale ciągle zdarzają
się sytuacje, gdzie zaczyna nam brakować miejsca na pliki. W takich przypadkach myślimy raczej o
zmianie rozmiaru istniejących już partycji czy też dokupieniu nowego dysku. Pierwsza z powyższych
opcji nie zawsze może wchodzić w grę, no chyba, że zaimplementowaliśmy sobie [LVM][1]. Jeśli tak,
to możemy bez większego problemu [zmieniać rozmiar każdego z tych voluminów LVM][2]. Niemniej
jednak, nawet jeśli już dostosujemy sobie te wirtualne dyski, to miejsce wciąż może nam się
skończyć i raczej można przyjąć za pewne, że tak się stanie w bliżej nieokreślonej przyszłości. Gdy
to nastąpi, czeka nas druga opcja wymieniona wyżej, tj. zaopatrzenie się w dodatkowy nośnik danych.
Przy pomocy LVM jesteśmy w stanie ten nowy dysk dodać do istniejącej grupy voluminów i zwiększyć tym
wirtualnym partycjom rozmiar bez potrzeby przerywania pracy systemu, czy też wykonywania dodatkowych
czynności związanych z formatowaniem i instalowaniem systemu na nowo. W tym wpisie postaramy się
przebrnąć przez ten proces.

<!--more-->
## Nowy dysk, a LVM

Jak wspomnieliśmy we wstępie, dodawanie nowego dysku HDD czy SDD do LVM nie jest żadnym problemem.
Natomiast w drugą stronę, to już tak miło nie jest. Załóżmy na chwilę, że mamy do dyspozycji dysk
`sda` i dokupiliśmy dysk `sdb` . Rozmiary nie mają w tej chwili znaczenia. Jeśli w przyszłości
będziemy chcieli odłączyć ten nośnik `sdb` , to nie będziemy w stanie tego zrobić podczas pracy
systemu. Naturalnie w dalszym ciągu będziemy mogli to zrobić z poziomu systemu live. Można by
pomyśleć, że ilość zgromadzonych danych będzie miała jakiekolwiek znaczenie, tj. usuwając nowy
dysk, dane musiałyby się zmieścić na dysku `sda` . Niestety to nie działa w ten sposób. Inaczej
sprawa ma się, gdy taki nośnik chcielibyśmy wymienić, czyli dokupiliśmy dysk `sdc` i wymieniamy
`sda` lub `sdb` . Tutaj LVM umożliwia przeniesienie struktury każdego z dysków (lub nawet obu) na
ten dodatkowy twardziel. Trzeba jedynie pilnować, by ten kolejny dysk był odpowiednio większy. Mając
na uwadze to powyższe ograniczenie, naprawdę zastanówmy się, czy chcemy dodać nowy dysk.

## Dodawanie nowego nośnika do istniejącej struktury LVM

W przypadku, gdy zdecydowaliśmy się na dodanie nowego dysku, to przydałoby się wiedzieć jak tego
dokonać. Zakładam, że dysponujemy już systemem, który operuje na jakichś voluminach LVM. Tutaj mamy
do dyspozycji dysk 80 GB, na którym praktycznie cała przestrzeń jest przeznaczona pod LVM:

    # pvscan
      PV /dev/sda2   VG debian          lvm2 [73.59 GiB / 0    free]
      Total: 1 [73.59 GiB] / in use: 1 [73.59 GiB] / in no VG: 0 [0   ]

W obszarze tego LVM znajduje się linux. Oczywiście niekoniecznie musi to być dysk systemowy i lepiej
nie rozkładać systemu na kilka kilka nośników. Naturalnie takie katalogi jak `/home/` czy `/var/`
można bez problemu umieścić na tylu dyskach na ilu będzie to konieczne. Grunt, to trzymać `/` na
jednym z nich. Tak wygląda zaś testowy LVM:

    # pvs -v --segments
      PV         VG     Fmt  Attr PSize  PFree Start SSize LV   Start Type   PE Ranges
      /dev/sda2  debian lvm2 a--  73.59g    0      0  4768 root     0 linear /dev/sda2:0-4767
      /dev/sda2  debian lvm2 a--  73.59g    0   4768  2384 home     0 linear /dev/sda2:4768-7151
      /dev/sda2  debian lvm2 a--  73.59g    0   7152   476 swap     0 linear /dev/sda2:7152-7627
      /dev/sda2  debian lvm2 a--  73.59g    0   7628 11212 ext      0 linear /dev/sda2:7628-18839

    # lvs -v --segments
      LV   VG     Attr       Start SSize  #Str Type   Stripe Chunk
      ext  debian -wi-ao----    0  43.80g    1 linear     0     0
      home debian -wi-ao----    0   9.31g    1 linear     0     0
      root debian -wi-ao----    0  18.62g    1 linear     0     0
      swap debian -wi-ao----    0   1.86g    1 linear     0     0

Grupa voluminów to `debian`. Mamy 4 wirtualne dyski o widocznych wyżej rozmiarach. Do takiej
konfiguracji dodajemy nowy dysk. Zanim jednak to zrobimy, to na tym nowym nośniku musimy utworzyć
fizyczny volumin LVM. Robimy to przy pomocy tego poniższego polecenia:

    # pvcreate /dev/sdd1
      Physical volume "/dev/sdd1" successfully created

Sprawdźmy co się w tej chwili stało:

    # pvscan
      PV /dev/sda2   VG debian          lvm2 [73.59 GiB / 0    free]
      PV /dev/sdd1                      lvm2 [74.53 GiB]
      Total: 2 [148.12 GiB] / in use: 1 [73.59 GiB] / in no VG: 1 [74.53 GiB]

    # pvs -v --segments
      PV         VG     Fmt  Attr PSize  PFree  Start SSize LV   Start Type   PE Ranges
      /dev/sda2  debian lvm2 a--  73.59g     0      0  4768 root     0 linear /dev/sda2:0-4767
      /dev/sda2  debian lvm2 a--  73.59g     0   4768  2384 home     0 linear /dev/sda2:4768-7151
      /dev/sda2  debian lvm2 a--  73.59g     0   7152   476 swap     0 linear /dev/sda2:7152-7627
      /dev/sda2  debian lvm2 a--  73.59g     0   7628 11212 ext      0 linear /dev/sda2:7628-18839
      /dev/sdd1         lvm2 ---  74.53g 74.53g     0     0          0 free

LVM zaczął widzieć nowy dysk ale nie jest on jeszcze przydzielony do żadnej grupy voluminów. Musimy
zatem go dodać do grupy `debian` przy pomocy `vgextend` , przykładowo:

    # vgextend debian /dev/sdd1
      Volume group "debian" successfully extended

W tej chwili ten nowy dysk powinien już być uwzględniony w grupie `debian` , a jego wolna przestrzeń
oddana nam do dyspozycji. Sprawdźmy czy tak jest w istocie:

    # pvscan -v
      PV /dev/sda2   VG debian          lvm2 [73.59 GiB / 0    free]
      PV /dev/sdd1   VG debian          lvm2 [74.53 GiB / 74.53 GiB free]
      Total: 2 [148.12 GiB] / in use: 2 [148.12 GiB] / in no VG: 0 [0   ]

Wyżej widzimy, że dysk ma przypisaną grupę `VG debian` oraz, że mamy do rozdysponowania
`74.53 GiB` . Cała grupa voluminów, jako, że składa się z dwóch dysków 80G, ma łącznie rozmiar
148 GiB. Na dobrą sprawę, to jest maksymalny rozmiar partycji, jaką moglibyśmy tutaj utworzyć i to
mimo faktu, że każdy z fizycznych nośników ma do dyspozycji jedynie 80G.

Przy założeniu, że na poszczególnych voluminach logicznych kończy nam się miejsce, przydałoby się
przestrzeń nowego dysku odpowiednio przydzielić. Robimy to przy pomocy `lvextend` w poniższy sposób:

    # lvextend -r -L +10G debian/root /dev/sdd1
    # lvextend -L +20G debian/home /dev/sdd1
    # lvextend -r -l +100%FREE debian/ext /dev/sdd1

Opcja `-r` odpowiada za automatyczne rozszerzenie systemu plików danego voluminu. Z kolei parametry
`-L` oraz `-l` odpowiadają za rozmiar nowych dysków. Volumin, który ma ulec rozciągnięciu
precyzujemy w formie `grupa/volumin` . Ostatni argument wskazuje na nowy dysk, tj. tam gdzie
dodatkowe zakresy bloków zostaną umieszczone.

Po wydaniu tych powyższych poleceń, struktura LVM powinna się nieco zmienić:

    # pvscan
      PV /dev/sda2   VG debian          lvm2 [73.59 GiB / 0    free]
      PV /dev/sdd1   VG debian          lvm2 [74.53 GiB / 0    free]
      Total: 2 [148.12 GiB] / in use: 2 [148.12 GiB] / in no VG: 0 [0   ]

    # pvs -v --segments
      PV         VG     Fmt  Attr PSize  PFree Start SSize LV   Start Type   PE Ranges
      /dev/sda2  debian lvm2 a--  73.59g    0      0  4768 root     0 linear /dev/sda2:0-4767
      /dev/sda2  debian lvm2 a--  73.59g    0   4768  2384 home     0 linear /dev/sda2:4768-7151
      /dev/sda2  debian lvm2 a--  73.59g    0   7152   476 swap     0 linear /dev/sda2:7152-7627
      /dev/sda2  debian lvm2 a--  73.59g    0   7628 11212 ext      0 linear /dev/sda2:7628-18839
      /dev/sdd1  debian lvm2 a--  74.53g    0      0  2560 root  4768 linear /dev/sdd1:0-2559
      /dev/sdd1  debian lvm2 a--  74.53g    0   2560  5120 home  2384 linear /dev/sdd1:2560-7679
      /dev/sdd1  debian lvm2 a--  74.53g    0   7680 11399 ext  11212 linear /dev/sdd1:7680-19078

    # lvs -v --segments
      LV   VG     Attr       Start  SSize  #Str Type   Stripe Chunk
      ext  debian -wi-ao----     0  43.80g    1 linear     0     0
      ext  debian -wi-ao---- 43.80g 44.53g    1 linear     0     0
      home debian -wi-ao----     0   9.31g    1 linear     0     0
      home debian -wi-ao----  9.31g 20.00g    1 linear     0     0
      root debian -wi-ao----     0  18.62g    1 linear     0     0
      root debian -wi-ao---- 18.62g 10.00g    1 linear     0     0
      swap debian -wi-ao----     0   1.86g    1 linear     0     0

Widzimy zatem, że zakresy poszczególnych voluminów uległy zmianie. Część z tych zakresów jest
ulokowana na nowym dysku. Na rozmiar całkowity, np. voluminu `ext` , składa się suma dwóch jego
części, tj. 43.80 GiB i 44.53 GiB. Podobnie z pozostałymi dyskami logicznymi.


[1]: https://pl.wikipedia.org/wiki/LVM
[2]: {{< baseurl >}}/post/zmiana-rozmiaru-lvm/
