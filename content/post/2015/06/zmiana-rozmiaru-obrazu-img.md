---
author: Morfik
categories:
- Linux
date: "2015-06-14T13:05:13Z"
date_gmt: 2015-06-14 11:05:13 +0200
published: true
status: publish
tags:
- img
- loop
title: Zmiana rozmiaru obrazu .img
---

Struktura pliku `.img` czy `.iso` niczym nie odbiega od struktury przeciętnego dysku twardego. W obu
przypadkach mamy dokładnie taki sam schemat budowy, tj. mamy obecny
[MBR]({{< baseurl >}}/post/mbr-ebr-i-tablica-partycji-dysku-twardego/) i partycje, z których
pierwsza zwykle jest wyrównana do 1MiB, zostawiając tym samym 2047 sektorów wolnego miejsca z
początku pliku, tuż za MBR, tzw. MBR-GAP. Obraz `.img` można poddać edycji, np. utworzyć wewnątrz
niego inne partycje, zmienić rozmiar ich systemu plików, a nawet można manipulować rozmiarem samego
obrazu `.img` . Taki plik możemy również zamontować w systemie przy pomocy narzędzia `mount` ale też
trzeba odpowiednio podejść do tego zadania, bo przez fakt, że mamy z początku MBR i trochę wolnego
miejsca, to linux zwyczajnie nie potrafi rozpoznać systemu plików, który znajduje się w obrazie
`.img`.

<!--more-->
## Tworzenie obrazu img

By wykazać, że taki zwykły plik binarny niczym nie różni się od dysku twardego, stworzymy na
potrzeby testu przy pomocy `dd` obraz składający się z samych zer. W tym celu wydajemy poniższe
polecenie:

    $ dd if=/dev/zero bs=2M count=200 > binary.img

Mamy zatem plik o rozmiarze 400MiB. Jako, że on zawiera same zera, to nie ma także jeszcze
utworzonej struktury dysku twardego. Zobaczmy zatem jak widzi go `fdisk` :

![]({{< baseurl >}}/img/2015/06/1.podglad-obrazu-img.png)

Jak widać wyżej, z automatu została utworzona tablica partycji ms-dos, bo bez niej nie dalibyśmy
rady utworzyć żadnej partycji. Stwórzmy zatem dwie partycje. Całość powinna wyglądać mniej więcej
tak:

    Command (m for help): p
    Disk binary.img: 400 MiB, 419430400 bytes, 819200 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x46f1aaca

    Device      Boot  Start    End Sectors   Size Id Type
    binary.img1        2048 400000  397953 194.3M 83 Linux
    binary.img2      400001 819199  419199 204.7M 83 Linux

Partycje może i zostały utworzone, natomiast nie posiadają one jeszcze systemu plików. Tworzenie
systemu plików w takim obrazie z wieloma partycjami jest trochę uporczywe, niemniej jednak przy
pomocy [odpowiedniej konfiguracji urządzeń
loop]({{< baseurl >}}/post/obsluga-wielu-partycji-w-module-loop/) możemy sobie bardzo uprościć
życie. W każdym razie zakładam, że stworzyliśmy system plików na tych partycjach oraz, że montują
się one w systemie bez problemów.

## Powiększanie obrazu img

Załóżmy na chwilę, że z jakichś powodów utworzyliśmy za mały obraz `.img` . Nie musimy go tworzyć od
nowa. Wystarczy go nieco rozciągnąć. W tym celu musimy dograć do niego pożądaną ilość zer:

    # dd if=/dev/zero bs=2M count=200 >> binary.img

Przy pomocy narzędzia `parted` sprawdzamy czy nowa przestrzeń została dodana z powodzeniem:

    (parted) print free
    Model:  (file)
    Disk /home/morfik/Desktop/binary.img: 839MB
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start    End       Size     Type     File system  Flags
            63s      2047s     1985s             Free Space
     1      2048s    400000s   397953s  primary  ext4
     2      400001s  819199s   419199s  primary  ext4
            819200s  1638399s  819200s           Free Space

Dodajmy teraz do drugiej partycji to wolne miejsce.

    (parted) resizepart 2
    End?  [819199s]? 1638399s

    (parted) print free
    Model:  (file)
    Disk /home/morfik/Desktop/binary.img: 1638400s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start    End       Size      Type     File system  Flags
            63s      2047s     1985s              Free Space
     1      2048s    400000s   397953s   primary  ext4
     2      400001s  1638399s  1238399s  primary  ext4

Teraz już tylko montujemy obie partycje i jak można zobaczyć poniżej, wszystko jest w należytym
porządku:

    # lsblk /dev/loop0
    NAME        SIZE FSTYPE TYPE LABEL MOUNTPOINT UUID
    loop0       800M        loop
    ├─loop0p1 194.3M ext4   loop       /mnt       bb42bde7-cab9-4ff5-8b23-0c4ef33af294
    └─loop0p2 604.7M ext4   loop       /mnt2      104f6b41-dfa6-4d3d-a37a-e7c39779746f

## Zmniejszanie obrazu img

Podobnie możemy się bawić w drugą stronę, z tym, że najpierw kurczymy jedną partycję:

    (parted) resizepart 2
    End?  [700000s]? 700000s

    (parted) print free
    Model:  (file)
    Disk /home/morfik/Desktop/binary.img: 1638400s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start    End       Size     Type     File system  Flags
            63s      2047s     1985s             Free Space
     1      2048s    400000s   397953s  primary  ext4
     2      400001s  700000s   300000s  primary  ext4
            700001s  1638399s  938399s           Free Space

A dopiero potem obcinamy obraz przy pomocy `dd` podając parametry `bs=512` oraz `count` o wartości
sektora tuż za końcem ostatniej partycji:

    # dd if=./binary.img of=binary-cut.img bs=512 count=700001
    700001+0 records in
    700001+0 records out
    358400512 bytes (358 MB) copied, 4.33031 s, 82.8 MB/s

I sprawdzamy czy dobrze ucięło:

    (parted) print free
    Model:  (file)
    Disk /home/morfik/Desktop/binary-cut.img: 700001s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start    End      Size     Type     File system  Flags
            63s      2047s    1985s             Free Space
     1      2048s    400000s  397953s  primary  ext4
     2      400001s  700000s  300000s  primary  ext4
