---
author: Morfik
categories:
- Linux
date: "2015-12-17T16:05:23Z"
date_gmt: 2015-12-17 15:05:23 +0100
published: true
status: publish
tags:
- system-plików
- fat32
title: Zmiana rozmiaru partycji FAT32 pod linux'em
---

Partycje mające system plików FAT32 są głównie wykorzystywane w przypadku pendrive i innych pamięci
flash, gdzie nie mamy do czynienia ze sporą ilością danych czy też dużymi plikami. Czasem się zdarza
tak, że układ partycji, który chcieliśmy zrobić, nie wyszedł nam tak jak powinien i musimy tego
pendrive jeszcze raz przeformatować. Pół biedy, gdy na nim nie ma żadnych danych lub mamy możliwość
zgrania ich na osobny dysk. Natomiast w przypadku, gdy nie możemy z jakiegoś powodu skorzystać z w/w
opcji, to musimy liczyć się z utratą danych. Oczywiście, możemy także spróbować zmienić rozmiar
takiej partycji i nic nie powinno się stać znajdującym się na niej danym. W tym wpisie postaramy się
przejść przez proces zmiany rozmiaru partycji mającej system plików FAT32 i zobaczymy czy nasz linux
poradzi sobie z tym zdaniem bez większego problemu.

<!--more-->
## Przygotowanie systemu plików FAT32

Zakładam, że mamy już jakąś partycję, na której znajduje się system plików FAT32. Przed
przystąpieniem do zmiany rozmiaru, ten system plików musimy odmontować i sprawdzić go w
poszukiwaniu ewentualnych błędów. Wszystkie potrzebne narzędzia znajdują się w pakiecie `dosfstools`
. Poniżej przykład skanowania:

    # dosfsck -v -a -w /dev/sdb3
    dosfsck 3.0.16 (01 Mar 2013)
    dosfsck 3.0.16, 01 Mar 2013, FAT32, LFN
    Checking we can access the last sector of the filesystem
    Boot sector contents:
    System ID "mkdosfs"
    Media byte 0xf8 (hard disk)
           512 bytes per logical sector
         16384 bytes per cluster
            32 reserved sectors
    First FAT starts at byte 16384 (sector 32)
             2 FATs, 32 bit entries
       5029888 bytes per FAT (= 9824 sectors)
    Root directory start at cluster 2 (arbitrary size)
    Data area starts at byte 10076160 (sector 19680)
       1253401 data clusters (20535721984 bytes)
    63 sectors/track, 255 heads
             0 hidden sectors
      40128512 sectors total

    Reclaiming unconnected clusters.
    Checking free cluster summary.
    /dev/sdb3: 20757 files, 72885/1253401 clusters

Jeśli chodzi o zmianę rozmiaru partycji FAT32, to nie znalazłem narzędzia, które zmieniałoby tylko
rozmiar systemu plików. Za to w debianie jest dostępny pakiet `fatresize` , który jak nazwa
wskazuje, umożliwia zmianę rozmiaru systemu plików FAT. Można do tego celu wykorzystać także
`parted` . Nie będziemy tutaj korzystać z jego graficznej nakładki [gparted](http://gparted.org/),
bo tam wszystko można sobie wyklikać bez większych problemów.

## Wykorzystanie fatresize do zmiany rozmiaru partycji FAT32

Narzędzie `fatresize` , to typowy automat. Grunt, że dostępny jest spod konsoli. Możemy przy jego
pomocy zbadać urządzenie i ustalić jaki nowy rozmiar może mieć zmieniana partycja. By to sprawdzić,
wydajemy poniższe polecenie:

    # fatresize -v -i /dev/sdb3
    fatresize 1.0.2 (04/23/10)
    FAT: fat32
    Size: 20545798144
    Min size: 1320769536
    Max size: 80025280000

Powyższe wartości są w bajtach, a sam program obsługuje zarówno podstawę potęgi 2 jak i 10. Zmieńmy
zatem rozmiar z 20 GiB na 10 GiB:

    # fatresize -v -p -s 10Gi /dev/sdb3
    fatresize 1.0.2 (04/23/10)
    Resizing file system.
    .........................................Done.
    Committing changes.

Wszystko sprowadza się do ustawienia nowego rozmiaru. Nie da się jednak skorzystać z tego narzędzia,
gdy w grę wchodzi zaszyfrowany kontener LUKS posiadający system plików FAT32.

## Wykorzystanie parted do zmiany rozmiaru partycji FAT32

`parted` umożliwia nieco bardziej swobodny obrót tymi typami partycji, które są przez niego
obsługiwane. Jeśli chodzi zaś o zmianę rozmiaru partycji przy pomocy tego narzędzia, to w
przyszłych wersjach zostanie on tej możliwości pozbawiony. Podejrzymy zatem jak w nim prezentuje
się nasz dysk na chwilę obecną:

    # parted /dev/sdb
    GNU Parted 2.3
    Using /dev/sdb
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) unit
    Unit?  [compact]? s
    (parted) print free
    Model: WDC WD80 0JB-00JJA0 (scsi)
    Disk /dev/sdb: 156299375s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start       End         Size       Type     File system  Flags
            63s         2047s       1985s               Free Space
     1      2048s       41945087s   41943040s  primary  ntfs
            41945088s   84713471s   42768384s           Free Space
     2      84713472s   116170751s  31457280s  primary  ext4
     3      116170752s  137109375s  20938624s  primary  fat32
            137109376s  156299374s  19189999s           Free Space

Jeśli komuś przeszkadzają sektory jako jednostki, może to z powodzeniem zmienić przez wpisanie
`unit` . By teraz zmienić rozmiar partycji numer 3, wskazujemy ją w `resize` i dostosowujemy
początkowy oraz końcowy sektor partycji. Wygląda to mniej więcej tak:

    (parted) resize 3
    Start?  [116170752s]?
    End?  [137109375s]? 156290000s
    moving data... 46%      (time left 00:29)

Sprawdzamy czy partycja ma nowy rozmiar:

    (parted) print free
    Model: WDC WD80 0JB-00JJA0 (scsi)
    Disk /dev/sdb: 156299375s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos

    Number  Start       End         Size       Type     File system  Flags
            63s         2047s       1985s               Free Space
     1      2048s       41945087s   41943040s  primary  ntfs
            41945088s   84713471s   42768384s           Free Space
     2      84713472s   116170751s  31457280s  primary  ext4
     3      116170752s  156290000s  40119249s  primary  fat32
            156290001s  156299374s  9374s               Free Space

Lepiej zawsze zostawić trochę miejsca na końcu dysku. Być może kiedyś będziemy potrzebować
[przekonwertować tablicę partycji z MS-DOS na
GPT]({{< baseurl >}}/post/konwersja-tablicy-partycji-ms-dos-na-gpt/).

W przypadku [zmiany rozmiaru systemu plików
EXT4]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ext4/) również możemy korzystać z `parted` w
celu zautomatyzowania i przyśpieszenia całej pracy. Nie ma on jednak zaimplementowanej obsługi NTFS
i ewentualną [zmianę rozmiaru systemy plików NTFS musimy przeprowadzać
ręcznie]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ntfs-pod-linuxem/).

Jeśli system nie widzi nowych partycji, być może pomocne okaże się wydanie w terminalu polecenia
`partprobe` jako użytkownik root.
