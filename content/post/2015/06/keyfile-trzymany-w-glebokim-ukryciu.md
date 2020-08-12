---
author: Morfik
categories:
- Linux
date: "2015-06-15T19:53:56Z"
date_gmt: 2015-06-15 17:53:56 +0200
published: true
status: publish
tags:
- pendrive
- keyfile
- luks
title: Keyfile trzymany w głębokim ukryciu
---

Pisząc ostatni [artykuł na temat
udeva]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/) i montowania przy jego
pomocy zaszyfrowanego kontenera, wpadł mi do głowy ciekawy pomysł na trzymanie pliku klucza
(keyfile) w czymś co się potocznie nazywa "głębokim ukryciem". Z reguły ludzie nie chcą używać haseł
do odblokowywania swoich systemów czy partycji i zamiast nich wolą stosować keyfile, czyli małe
pliki, zwykle o rozmiarze paru KiB, które, jakby nie patrzeć, są dość unikatowe i odporne na ataki
słownikowe czy inne formy przemocy. Jedyny problem z jakim człowiek musi się zmierzyć, to z
zabezpieczeniem takiego keyfile i tutaj sprawa nie wygląda wcale dobrze. Takie pliki klucze są
trzymane zwykle na tym samym urządzeniu, do których mają zapewniać bezpieczny dostęp, a nawet jeśli
nie na tym samym, to w pobliżu takich urządzeń, dając nam tym samym jedynie fałszywe poczucie
bezpieczeństwa.

<!--more-->
## Szukanie dogodnego miejsca na keyfile

A może istnieje jakiś sposób, który by ukrył taki keyfile by nikt, kto nie jest jego świadom, nie
mógł go użyć do odszyfrowania zabezpieczonego urządzenia? Jeśli weźmiemy pod lupę, np. jakiś
pendrive, czy dysk twardy, to możemy zobaczyć poniższy schemat partycji (fotka za
wikipedią):

[![1.linux-mbr-keyfile]({{< baseurl >}}/img/2015/06/1.linux-mbr-keyfile-1024x576.png)]({{< baseurl >}}/img/2015/06/1.linux-mbr-keyfile.png)

Mamy tutaj dwie różne tablice partycji: MS-DOS oraz GPT. Nas głównie interesuje
[MS-DOS](https://pl.wikipedia.org/wiki/Master_Boot_Record), bo na nim mamy większe pole manewru. W
przypadku bootloader'a grub, część jego kodu wędruje do przestrzeni między MBR oraz pierwszą
partycję, tak jak pokazane wyżej w pierwszym przykładzie. Dokładnie nie wiem ile to jest KiB ale
inne bootloader'y mogą nie korzystać z tej przestrzeni zupełnie i tak właśnie sprawa wygląda w
przypadku extlinux, gdzie cały MBR-GAP jest do naszej dyspozycji. Oczywiście to tyczy się tylko
ewentualnych obrazów live i systemów operacyjnych zainstalowanych na HDD czy SSD. W przypadku
wszystkich pozostałych nośników wykorzystujących tablicę partycji MS-DOS, możemy dowolnie zarządzać
sobie przestrzenią za MBR. Zwykle pierwsza partycja jest tworzona na 2048 sektorze, w wyniku
równania do jednego 1 MiB. Możemy naturalnie utworzyć ją sobie i dalej, choć 1 MiB powinien nam
wystarczy w zupełności, by ktoś czasem nie zaczął zastanawiać się, po co zostawiliśmy tam tyle
"wolnego" miejsca. Nie chcemy też zapisywać MBR, który rezyduje w pierwszym sektorze na pozycji 0.
Zostaje nam zatem 2047 sektorów, co daje 1048064 bajtów wolnego miejsca, a my potrzebujemy jakieś 2
KiB . Czyli mamy dość miejsca by ulokować keyfile tak by nie można było zgadnąć gdzie on jest. Ten
keyfile nie będzie miał nazwy i nawet nie będzie to plik, przynajmniej w ludzkim rozumieniu tego
słowa, bo jest to tylko zbitka bitów na dysku, do której nie ma żadnego odnośnika, nie wspominając
już nawet o tym, że za MBR i przed pierwszą partycją nie ma nawet stworzonego systemu plików.
Dlatego właśnie to jest najdogodniejsze miejsce, bo ulokowany tam keyfile nie pozostawia praktycznie
żadnych wskazówek co do swojego istnienia.

## Czyszczenie MBR-GAP

Naturalnie można by stworzyć mały pliczek i wgrać go w któreś miejsce między 0 a 2048 sektorem ale
może to zagrozić poufności pliku klucza, bo w przypadku gdyby tam były same 0 i wgralibyśmy tam
jakieś dane o długości 2048 bajtów, to będą one widoczne jak na dłoni. Dlatego też zapiszemy pierw
losowymi danymi całą wolną przestrzeń, po czym określimy, które sektory będą robić za keyfile.

To do dzieła. Zapisujemy 2047 sektorów 512 bajtowych pomijając pierwszy z nich, w którym siedzi MBR.
Nie chcemy sobie przecież wyczyścić przy okazji tablicy partycji.

    # dd if=/dev/urandom of=/dev/sdb bs=512 count=2047 seek=1
    2047+0 records in
    2047+0 records out
    1048064 bytes (1.0 MB) copied, 0.346335 s, 3.0 MB/s

## Tworzenie keyfile

W ten sposób mamy losowe dane w MBR-GAP. Pozostało nam jeszcze określenie, który kawałek będzie
robił za keyfile. Niech będzie to 2 KiB zaczynając od 101 sektora nośnika. Oczywiście rozmiar może
być dowolny, tzn. nie większy niż 8192KiB.

Pierwsze z poniższych poleceń czyta 4\*512 bajtów, czyli 2 KiB danych, pomijając 100 sektorów. To
tam właśnie będzie znajdował się nasz keyfile. Drugie polecenie zaś dodaje ten klucz do nagłówka
LUKS:

    # dd if=/dev/sdb bs=512 count=4 skip=100 > ./keyfile
    4+0 records in
    4+0 records out
    2048 bytes (2.0 kB) copied, 0.00102532 s, 2.0 MB/s

    # cryptsetup luksAddKey /dev/sdb2 ./keyfile
    Enter any passphrase:

Sprawdzamy czy keyfile został dodany do nagłówka:

    # cryptsetup luksDump /dev/sdb2
    ...
    Key Slot 0: ENABLED
            Iterations:             37558
            Salt:                   c1 1a 6f 49 5f f4 49 94 e2 4d 56 3a 49 39 87 14
                                    95 38 2f d3 cd 99 43 73 50 9b 44 09 cb c5 ad 9f
            Key material offset:    8
            AF stripes:             4000

    Key Slot 1: ENABLED
            Iterations:             15685
            Salt:                   95 f7 1b 27 04 c6 87 c8 3c 14 23 5b c7 99 f8 c2
                                    cf ab 1c ed 8c 70 69 b4 da a3 a7 67 97 5e f6 a4
            Key material offset:    512
            AF stripes:             4000
    ...

`Slot 0` to hasło do zaszyfrowanej partycji, natomiast na pozycji `slot 1` został wgrany keyfile.

By teraz otworzyć partycję na pendrive z wykorzystaniem tego keyfile, wpisujemy poniższą linijkę do
terminala:

    # dd if=/dev/sdb bs=512 count=4 skip=100 | cryptsetup luksOpen /dev/sdb2 sdb2 --key-file=-

I jeszcze możemy sprawdzić czy aby na pewno kontener się otwiera, gdy podamy koordynaty `512` `4`
`100` dla dd. Podmieńmy zatem 100 na 101:

    # dd if=/dev/sdb bs=512 count=4 skip=101 | cryptsetup luksOpen /dev/sdb2 sdb2 --key-file=-
    No key available with this passphrase.

Nasz keyfile, co prawda, jest dostarczany razem z zaszyfrowanym urządzeniem ale pozostaje w głębokim
ukryciu i nikt kto nie zna tych 3 parametrów, nie uzyska dostępu do zaszyfrowanej partycji. Poza
tym, kto by chował keyfile w MBR-GAP?
