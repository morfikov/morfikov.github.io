---
author: Morfik
categories:
- Linux
date: "2015-06-11T14:43:35Z"
date_gmt: 2015-06-11 12:43:35 +0200
published: true
status: publish
tags:
- pendrive
- live
- karta-sd
- flash
GHissueID: 84
title: Jak wgrać system live na uszkodzony pendrive
---

Każdy z nas spotkał się już chyba w swoim życiu z systemami live. To taki wynalazek zawierający
domyślne oprogramowanie, tak skonfigurowane, by system zdołał się odpalić na większość maszyn w ich
pamięci operacyjnej, oczywiście zakładając, że wymagania sprzętowe zostaną zaspokojone, głównie
chodzi o pamięć RAM. Takie systemy live są dostarczane w postaci kilku rodzajów obrazów: iso, hdd
oraz hybryda. Pierwszy z nich zwykle wypala się na płytkach, drugi na pendrive, z kolei hybrydowy
obraz można wgrać na każdy rodzaj urządzenia i nawet wykorzystać w przypadku maszyn wirtualnych i
zwykle spotkamy się z tym ostatnim typem, nawet jeśli plik ma rozszerzenie `.iso` .

<!--more-->
## Problematyczna utrata danych

W przypadku płyt cd/dvd raczej nie razi nas to, że na taki obraz live trzeba przeznaczyć całą
płytkę. Natomiast jeśli chodzi o pendrive, to te maja zwykle więcej miejsca niż standardowy dysk
cd/dvd i przy wgrywaniu obrazu tracimy wszystkie dane znajdujące się na pendrive. Dzieje się tak
dlatego, że przepisywany jest MBR i co za tym idzie również tablica partycji.

Możemy wyeliminować tę niedogodność, z tym, że trzeba będzie zrezygnować z wgrywania obrazu live
bezpośrednio na pendrive. Zamiast tego trzeba będzie cały proces wgrywania obrazu przeprowadzić
ręcznie. Nie jest on skomplikowany i zwykle nie zajmie nam więcej niż 5 min. Dodatkową korzyścią
płynąca z tego rozwiązania jest fakt, że możemy taki system live ulokować w dowolnym miejscu na
pendrive i nie musi być to obszar początkowy, który jest zwykle najbardziej utylizowany, co
przekłada się na [uszkodzone sektory](/post/kiedy-zywot-pendrive-dobiega-konca/) i
w takim przypadku jeśli nasz pendrive ma choć jeden bad blok gdzieś na początku, to nie damy rady
wgrać obrazu live przy pomocy `dd`.

## Partycjonowanie uszkodzonego pendrive

Posługując się przykładem uszkodzonego pendrive w linku wyżej, mamy do dyspozycji 16G przestrzeni, z
czego uszkodzone sektory są rozsiane na pierwszych 5GiB. Zwykle obraz live ma nieco ponad 1GiB,
zatem najlepiej odpuścić sobie ten obszar początkowy i utworzyć minimum dwie partycje, z których
jedna będzie mieć 5GiB, a druga będzie rozciągać się na pozostałe miejsce na pendrive, przykładowo:

![gparted-uszkodzony-pendrive-live](/img/2015/06/1.gparted-uszkodzony-pendrive-live.png#big)

Naturalnie, możemy stworzyć więcej partycji i przeznaczyć jedynie 2GiB na system live. W dalszym
ciągu także możemy korzystać z pierwszej partycji, choć nie jest to zalecane, bo istnieje spore
ryzyko utraty danych. Można co prawda oznaczyć padnięte sektory ale to jest pamięć flash, a nie dysk
twardy i odczyt/zapis danych w komórkach czasem może być możliwy, a czasem niekoniecznie i liczba
padniętych bloków będzie się zmieniać z każdym skanowaniem, przynajmniej tak było w tym przypadku.

## Przenoszenie zawartości obrazu live na pendrive

Mając odpowiednio sformatowany pendrive, pobieramy [obraz live](https://www.debian.org/CD/live/) ,
po czym sprawdzamy jego zawartość przy pomocy narzędzia `parted` :

    # parted  /home/morfik/Desktop/debian-live-8.1.0-amd64-mate-desktop.iso
    (parted) unit s

    (parted) print
    Model:  (file)
    Disk /home/morfik/Desktop/debian-live-8.1.0-amd64-mate-desktop.iso: 2015232s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start  End       Size      Type     File system  Flags
     1      64s    2015231s  2015168s  primary               boot, hidden

Plik można traktować jak zwykły dysk twardy, bo zawiera partycje i wszelkie operacje, które można
przeprowadzić na dyskach, można również przeprowadzić na obrazach live. Jak widzimy wyżej, plik
zawiera jedną partycję, która zaczyna się na 64 sektorze. Jest to 64*512=32768 bajtów. Możemy tę
partycję zamontować w systemie uwzględniając ten offset:

    # mount -o loop,offset=32768 /home/morfik/Desktop/debian-live-8.1.0-amd64-mate-desktop.iso /mnt
         mount: /dev/loop0 is write-protected, mounting read-only

Powinniśmy teraz mieć dostęp do systemu plików partycji:

    # ls -al /mnt
    total 593K
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:09:57 ./
    drwxr-xr-x 24 root root 4.0K 2015-06-08 20:54:43 ../
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:08:34 .disk/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 15:59:10 dists/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:09:41 install/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:08:29 isolinux/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:08:29 live/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 15:59:00 pool/
    dr-xr-xr-x  1 root root 2.0K 2015-06-06 16:09:37 tools/
    -r--r--r--  1 root root  133 2015-06-06 16:09:44 autorun.inf
    lr-xr-xr-x  1 root root    1 2015-06-06 15:59:10 debian -> ./
    -r--r--r--  1 root root 177K 2015-06-06 16:09:44 g2ldr
    -r--r--r--  1 root root 8.0K 2015-06-06 16:09:44 g2ldr.mbr
    -r--r--r--  1 root root  28K 2015-06-06 16:09:57 md5sum.txt
    -r--r--r--  1 root root 360K 2015-06-06 16:09:44 setup.exe
    -r--r--r--  1 root root  228 2015-06-06 16:09:44 win32-loader.ini

Te wszystkie powyższe pliki musimy przekopiować na drugą partycję pendrive. Montujemy ją zatem
również w systemie, u mnie jest ona widoczna pod `/media/morfik/good/` :

    # cp -a /mnt/* /media/morfik/good/

Teraz musimy jeszcze dograć jakiś bootloader na pendrive, z tym, że takowy jest już dołączony z
obrazem live i jest to `isolinux`, wnioskując po katalogu o tej samej nazwie. problem w tym, że
partycja na pendrive ma system plików ext4 i nie możemy wykorzystać isolinuxa, bo ten obsługuje
system plików ISO 9660. Do partycji posiadających system plików z rodziny ext wykorzystywany jest
`extlinux` . Po przekopiowaniu się plików, przechodzimy na partycję pendrive i wydajemy poniższe
polecenia:

    # cd /media/morfik/good/isolinux
    # mv isolinux.cfg extlinux.conf
    # cp /usr/lib/syslinux/modules/bios/* ./
    # cd ..
    # mv isolinux extlinux

Powyższe kroki dostosują odpowiednio nazwy plików i katalogów oraz przekopiują kilka dodatkowych
modułów.

Musimy jeszcze wgrać MBR w pierwszy sektor pendrive, bo bez tego system live się nie
    odpali:

    # printf '\x2' | cat /usr/lib/SYSLINUX/altmbr.bin - | dd bs=440 count=1 iflag=fullblock conv=notrunc of=/dev/sdb
    1+0 records in
    1+0 records out
    440 bytes (440 B) copied, 0.000160464 s, 2.7 MB/s

Prawdopodobnie możemy wykorzystać zwykły kod bootloadera i ustawić jedynie flagę `boot` w opcjach
partycji, ja jednak wolę wdrukować w MBR na sztywno numerek partycji, z której ma startować system.
Ten numerek wskazujemy przy pomocy `\x2` .

Ostatnim krokiem, jaki nam pozostał, jest zainstalowanie pozostałego kodu bootloadera na drugiej
partycji pendrive:

    # extlinux -i /media/morfik/good/extlinux
    /media/morfik/good/extlinux is device /dev/sdb2

Wszystkie kroki, za wyjątkiem kopiowania plików przeprowadzamy tylko raz. Gdyby z jakichś powodów
zaszła potrzeba wgrania nowego obrazu, montujemy go i kopiujemy jego pliki z wyłączeniem katalogu
`isolinux` . Dobrze jest też usunąć wszystkie poprzednie pliki z tej partycji pendrive, oczywiście
zostawiając jedynie katalog `extlinux` .
