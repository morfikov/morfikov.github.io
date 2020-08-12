---
author: Morfik
categories:
- Linux
date: "2015-12-16T19:06:57Z"
date_gmt: 2015-12-16 18:06:57 +0100
published: true
status: publish
tags:
- system-plików
- hdd/ssd
- ext4
title: Zmiana rozmiaru partycji EXT4
---

Jeśli jeszcze nie dokonaliśmy [migracji systemu plików z EXT2/3 na
EXT4]({{< baseurl >}}/post/migracja-systemu-plikow-ext2-i-ext3-na-ext4/), to powinniśmy rozważyć
tę kwestię z przyczyn czysto praktycznych. W tym wpisie nie będziemy sobie głowy zawracać migracją
między poszczególnymi wersjami systemu plików z rodziny EXT, a raczej skupimy się na tym jak zmienić
rozmiar partycji, której systemem plików jest właśnie EXT4. Bawienie się rozmiarem partycji w tym
przypadku niczym zbytnio się nie różni w stosunku do omawianego wcześniej [systemu plików
NTFS]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ntfs-pod-linuxem/). Będziemy wykorzystywać
tylko nieco inne narzędzia.

<!--more-->
## Przygotowanie systemu plików EXT4

Procedura zmiany rozmiaru każdego systemu plików jest mniej więcej taka sama. Musimy posiadać
partycję, a na niej system plików. Przed przystąpieniem do pracy, system plików należy odmontować i
przeskanować go w poszukiwaniu ewentualnych błędów. Nie musimy instalować dodatkowych pakietów, bo
wszystkie niezbędne narzędzia powinniśmy już mieć w systemie. System plików EXT4 skanujemy w
poniższy sposób:

    # fsck.ext4 -f -v /dev/sdb2

## Zmniejszanie rozmiaru partycji EXT4

Jeśli chcemy skurczyć system plików do możliwe małego, to możemy użyć opcji `-M` . Z kolei, by
ustalić najmniejszy możliwy rozmiar systemu plików na danej partycji, korzystamy z opcji `-P` . My
jednak ustawimy określony rozmiar:

    # resize2fs -p /dev/sdb2 10G
    resize2fs 1.42.9 (28-Dec-2013)
    Resizing the filesystem on /dev/sdb2 to 2621440 (4k) blocks.
    Begin pass 2 (max = 85252)
    Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 3 (max = 274)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 4 (max = 430)
    Updating inode references     XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/sdb2 is now 2621440 blocks long.

Cóż to jest te 2621440 bloków? To jest właściwy rozmiar systemu plików:
2621440\*4096/1024/1024/1024=10GiB . Przy czym, rozmiar bloku (4096) może być inny, w zależności od
wielkości partycji. Rozmiar bloku możemy ustalić wydając poniższe polecenie:

    # tune2fs -l /dev/sdb2 | grep -i "block size"
    Block size:               4096

Wartość tego bloku oraz wartość ta, która widnieje w `fdisk` na pozycji `block` nie są tożsame. Dla
fdisk'a rozmiar bloku jest równy 1024 (przynajmniej u mnie). Także nie możemy kierować się powyższą
wartością 2621440 i trzeba przyjąć liczbę czterokrotnie wyższą, czyli 10485760. Tylko, że to są
bloki. Trzeba je przeliczyć na sektory: (10485760\*2)-1=20971519, albo sprecyzować wielkość partycji
przez +10G. W obu przypadkach ( `resize2fs` i `fdisk` ) używana jest 2 jako podstawa potęgi.
Odpalamy fdisk'a, kasujemy partycję i robimy nową. Z tym, że trzeba uważać. U mnie partycja druga
nie zaczyna się zaraz po pierwszej. Zawsze dobrze jest sprawdzić i spisać sobie początek zmienianej
partycji:

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    41945087    20971520    7  HPFS/NTFS/exFAT
    /dev/sdb2        84713472   156299263    35792896   83  Linux

Liczbę 84713472 trzeba będzie wpisać jako pierwszy sektor przy tworzeniu nowej partycji. Kasujemy
partycję `sdb2` i tworzymy nową podając albo +10G, albo +20971519 jako ostatni sektor:

    Command (m for help): d
    Partition number (1-4): 2

    Command (m for help): n
    Partition type:
       p   primary (1 primary, 0 extended, 3 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 2): 2
    First sector (41945088-156299374, default 41945088): 84713472
    Last sector, +sectors or +size{K,M,G} (84713472-156299374, default 156299374): +20971519

    Command (m for help): t
    Partition number (1-4): 2
    Hex code (type L to list codes): 83

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

I to wszystko jeśli chodzi o kurczenie rozmiaru partycji EXT4. Wyżej zmieniliśmy także typ partycji
ustawiając odpowiedni kod (83). Ten krok jest opcjonalny, bo wszystkie tworzone za sprawą fdisk'a
partycje mają domyślnie ten typ.

## Zwiększanie rozmiaru partycji EXT4

Po sprawdzeniu systemu plików w poszukiwaniu błędów, odpalamy fdisk'a i kasujemy odpowiednią
partycję:

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    41945087    20971520    7  HPFS/NTFS/exFAT
    /dev/sdb2        84713472   105684991    10485760   83  Linux

    Command (m for help): d
    Partition number (1-4): 2

Na jej miejscu tworzymy nową uwzględniając większy rozmiar. W tym przypadku chcemy rozciągnąć starą
partycję z 10G do 15G. Zatem w `fdisk` wpisujemy `n` , ustawiamy odpowiednio pierwszy sektor i
precyzujemy rozmiar partycji na `+15G` :

    Command (m for help): n
    Partition type:
       p   primary (1 primary, 0 extended, 3 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 2): 2
    First sector (41945088-156299374, default 41945088): 84713472
    Last sector, +sectors or +size{K,M,G} (84713472-156299374, default 156299374): +15G

Precyzujemy także typ partycji 83 (EXT4) i zapisujemy nową tablice partycji:

    Command (m for help): t
    Partition number (1-4): 2
    Hex code (type L to list codes): 83

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

Teraz powiększamy system plików przy pomocy `resize2fs` :

    # resize2fs -p /dev/sdb2
    resize2fs 1.42.9 (28-Dec-2013)
    Resizing the filesystem on /dev/sdb2 to 3932160 (4k) blocks.
    The filesystem on /dev/sdb2 is now 3932160 blocks long.

Można sprecyzować oczywiście i rozmiar ale, gdy w grę wchodzi powiększanie partycji, tak by
wypełniła całą dostępną przestrzeń, to można ominąć ten parametr. W ten sposób system będzie
wiedział, że ma rozciągnąć system plików, tak by wypełnił całą partycję.
