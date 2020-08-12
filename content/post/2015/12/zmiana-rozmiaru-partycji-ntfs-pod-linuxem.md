---
author: Morfik
categories:
- Linux
date: "2015-12-16T18:03:54Z"
date_gmt: 2015-12-16 17:03:54 +0100
published: true
status: publish
tags:
- system-plików
- hdd/ssd
- ntfs
title: Zmiana rozmiaru partycji NTFS pod linux'em
---

Zmiana rozmiaru partycji, nie tylko tej zawierającej system plików NTFS, nie sprawia w dzisiejszych
czasach praktycznie żadnych problemów. Mamy przecie do dyspozycji takie narzędzia jak
[gparted](http://gparted.org/), które w dużej mierze automatyzują cały proces tworzenia i usuwania
partycji, czy też zmiany ich rozmiaru. W tym wpisie przyjrzymy się temu procesowi z bliska, z tym,
że nie będziemy korzystać z żadnych graficznych nakładek. Wszystkie kroki postaramy się
zreprodukować ręcznie z poziomu konsoli przy pomocy takich narzędzi jak `fdisk` czy `ntfsresize` .

<!--more-->
## System plików NTFS

Wszystkie poniższe kroki najlepiej jest przeprowadzać w środowisku testowym, tak by sobie przez
przypadek nie usunąć ważnych danych. Niżej mamy przykładowy dysk. Znajdują się na nim dwie partycje
i w tym też jest partycja NTFS. Wygląda to mniej więcej tak:

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
     1      2048s       61442047s   61440000s  primary  ntfs
     2      61442048s   156299263s  94857216s  primary  ext4
            156299264s  156299374s  111s                Free Space

Obsługa systemu plików NTFS w `parted` nie jest zaimplementowana i przy próbie zmiany rozmiaru
takiej partycji, dostaniemy poniższy komunikat:

    No Implementation: Support for opening ntfs file systems is not implemented yet.

W przypadku partycji NTFS odpadają nam zatem `gparted` oraz `parted` . Za to w pakiecie `ntfs-3g`
mamy narzędzie `ntfsresize` . Jak sama nazwa wskazuje, potrafi ono zmienić rozmiar partycji NTFS i
to nim będziemy się posługiwać w dalszej części wpisu.

## Zmniejszanie rozmiaru partycji NTFS

Przed przystąpieniem do zmiany rozmiaru jakiejkolwiek partycji, nie tylko NTFS, trzeba ją pierw
przeskanować w poszukiwaniu ewentualnych błędów w systemie plików. Poniżej przykładowa linijka:

    # ntfsresize -i -f -v /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 31457276416 bytes (31458 MB)
    Current device size: 31457280000 bytes (31458 MB)
    Checking for bad sectors ...
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (1.7%)
    Collecting resizing constraints ...
    Estimating smallest shrunken size supported ...
    File feature         Last used at      By inode
    $MFTMirr           :      7204 MB             1
    Ordinary           :     14403 MB          3380
    You might resize at 537702400 bytes or 538 MB (freeing 30920 MB).
    Please make a test run using both the -n and -s options before real resizing!

Jak widać powyżej, jest nawet informacja na temat tego jak bardzo można zjechać z rozmiarem
partycji. Jako, że jest tam na niej około 500 MB danych, minimalny rozmiar partycji nie może
przekroczyć 538 MB. Jest też również informacja, by przed rzeczywistą zmianą rozmiaru przeprowadzić
test używając opcji `-n` i `-s` . Sprawdźmy zatem co taki test nam powie:

    # ntfsresize -f -s 10G -n /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 31457276416 bytes (31458 MB)
    Current device size: 31457280000 bytes (31458 MB)
    New volume size    : 9999995392 bytes (10000 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (1.7%)
    Collecting resizing constraints ...
    Needed relocations : 23717 (98 MB)
    Schedule chkdsk for NTFS consistency check at Windows boot time ...
    Resetting $LogFile ... (this might take a while)
    Relocating needed data ...
    100.00 percent completed
    Updating $BadClust file ...
    Updating $Bitmap file ...
    Updating Boot record ...
    The read-only test run ended successfully.

Narzędzie `ntfsresize` operuje na podstawie potęgi 10 zamiast 2. W takim przypadku 10 GB i 10 GiB
nie oznacza tych samych wartości. Można również zauważyć, że będzie potrzeba przenieść 98 MB zanim
system plików zostanie zmniejszony. W przypadku większych partycji, naturalnie będzie to sporo
więcej danych. Jeśli nie będzie żadnych błędów, wydajemy polecenie bez opcji `-n` :

    # ntfsresize -f -s 10G /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 31457276416 bytes (31458 MB)
    Current device size: 31457280000 bytes (31458 MB)
    New volume size    : 9999995392 bytes (10000 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (1.7%)
    Collecting resizing constraints ...
    Needed relocations : 23717 (98 MB)
    WARNING: Every sanity check passed and only the dangerous operations left.
    Make sure that important data has been backed up! Power outage or computer
    crash may result major data loss!
    Are you sure you want to proceed (y/[n])? y
    Schedule chkdsk for NTFS consistency check at Windows boot time ...
    Resetting $LogFile ... (this might take a while)
    Relocating needed data ...
    100.00 percent completed
    Updating $BadClust file ...
    Updating $Bitmap file ...
    Updating Boot record ...
    Syncing device ...
    Successfully resized NTFS on device '/dev/sdb1'.
    You can go on to shrink the device for example with Linux fdisk.
    IMPORTANT: When recreating the partition, make sure that you
      1)  create it at the same disk sector (use sector as the unit!)
      2)  create it with the same partition type (usually 7, HPFS/NTFS)
      3)  do not make it smaller than the new NTFS filesystem size
      4)  set the bootable flag for the partition if it existed before
    Otherwise you won't be able to access NTFS or can't boot from the disk!
    If you make a mistake and don't have a partition table backup then you
    can recover the partition table by TestDisk or Parted's rescue mode.

Zmniejszyliśmy właśnie system plików. By się przekonać czy faktycznie jego rozmiar uległ zmianie,
wydajemy poniższe polecenie:

    # ntfsresize -i -f /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 9999995392 bytes (10000 MB)
    Current device size: 31457280000 bytes (31458 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (5.4%)
    Collecting resizing constraints ...
    You might resize at 537047040 bytes or 538 MB (freeing 9462 MB).
    Please make a test run using both the -n and -s options before real resizing!

Wartości parametrów `Current volume size` i `Current device size` różnią się. Teraz jeszcze trzeba
zmniejszyć rozmiar partycji. By tego dokonać potrzebny nam `fdisk` . W fdisk'u możemy precyzować
rozmiar używając bajtów lub sektorów, z tym, że `fdisk` używa podstawy potęgi 2. Jak widać wyżej,
niby wpisaliśmy 10G ale wyszła liczba 9999995392, która nijak się dzieli przez 1024. Za to dzieli
się ona przez 512. Liczba 512 odpowiada też za rozmiar sektora i możemy sobie wyliczyć rozmiar
nowej partycji w sektorach. Można też bez problemu podać rozmiar w bajtach. Jeśli jednak ktoś
preferuje sektory, to będzie to 9999995392/512=19531241 sektorów. Odpalamy `fdisk`, kasujemy
zmienianą partycję i na jej miejsce tworzymy nową. Musi ona się zaczynać w miejscu starej. Jedynie
co, to zmieni się położenie jej końca:

    # fdisk /dev/sdb
    Command (m for help): d
    Partition number (1-4): 1
    
    Command (m for help): n
    Partition type:
       p   primary (1 primary, 0 extended, 3 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1):
    Using default value 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-61442047, default 61442047): +19531241

Domyślnie w fdisk'u zostanie stworzona partycja o kodzie 83 (linux), musimy to zmienić na kod 7
(NTFS). Możemy sobie wylistować typy partycji w fdisk'u przy pomocy opcji `L` . Wybieramy zatem
partycję i ustawiamy jej pożądany typ:

    Command (m for help): t
    Partition number (1-4): 1
    
    Hex code (type L to list codes): 7
    Changed system type of partition 1 to 7 (HPFS/NTFS/exFAT)

Sprawdzamy czy wszystko się zgadza i jeśli jesteśmy zadowoleni z wyniku, zapisujemy układ partycji:

    Command (m for help): p
    
    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    19533289     9765621    7  HPFS/NTFS/exFAT
    /dev/sdb2        61442048   156299263    47428608   83  Linux
    
    Command (m for help): w
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    Syncing disks.

Jeszcze by mieć absolutną pewność, że nic nie popsuliśmy, rzućmy okiem na to, co nam powie sam
`ntfsresize` :

    # ntfsresize -i -f /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 9999995392 bytes (10000 MB)
    Current device size: 9999995904 bytes (10000 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (5.4%)
    Collecting resizing constraints ...
    You might resize at 537047040 bytes or 538 MB (freeing 9462 MB).
    Please make a test run using both the -n and -s options before real resizing!

Ja u siebie jeszcze na wszelki wypadek sprawdziłem czy nowa partycja sprawia jakiś kłopot w
`gparted` ale nawet on nie zwraca żadnych problemów. Dane na dysku również są i można je odczytać.
Także wszystko przebiegło bez większych problemów.

## Zwiększanie rozmiaru partycji NTFS

Skoro istnieje potrzeba, by skurczyć czasem partycję, to na pewno ktoś będzie potrzebował jakąś
partycję rozciągnąć. W przypadku powiększania partycji, postępujemy odwrotnie jak w przypadku jej
zmniejszania. Najpierw rozciągamy partycję, a następnie jej system plików. Odpalamy `fdisk`,
kasujemy starą partycję i na jej miejsce tworzymy nową:

    Command (m for help): d
    Partition number (1-4): 1
    
    Command (m for help): n
    Partition type:
       p   primary (1 primary, 0 extended, 3 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-61442047, default 61442047): +20G
    
    Command (m for help): t
    Partition number (1-4): 1
    Hex code (type L to list codes): 7
    Changed system type of partition 1 to 7 (HPFS/NTFS/exFAT)
    
    Command (m for help): w
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    Syncing disks.

W przypadku, gdy spróbujemy rozciągnąć system plików bez uprzedniego zwiększenia rozmiaru partycji,
dostalibyśmy poniższy komunikat:

    ERROR: New size can't be bigger than the device size.

Mając już powiększoną partycję, zmieniamy rozmiar jej systemu plików. Pierw oczywiście test w trybie
tylko do odczytu:

    # ntfsresize -s 20G -f -n  /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 9999995392 bytes (10000 MB)
    Current device size: 21474836480 bytes (21475 MB)
    New volume size    : 19999994368 bytes (20000 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (5.4%)
    Collecting resizing constraints ...
    Schedule chkdsk for NTFS consistency check at Windows boot time ...
    Resetting $LogFile ... (this might take a while)
    Updating $BadClust file ...
    Updating $Bitmap file ...
    Updating Boot record ...
    The read-only test run ended successfully.

`fdisk` bez problemu policzył 20\*1024\*1024\*1024=21474836480 , a ten `ntfsresize` próbował
20\*1000\*1000\*1000. Jako, że komputery działają w oparciu o podstawę potęgi 2, a nie 10, to w
życiu nie uzyskamy okrągłego wyniku. Dlatego mamy wartość 19999994368 zamiast okrągłego
20000000000. W każdym razie zamiast `-s 20G` można sprecyzować `-s 21474836480` :

    # ntfsresize -s 21474836480 -f -n  /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 9999995392 bytes (10000 MB)
    Current device size: 21474836480 bytes (21475 MB)
    New volume size    : 21474832896 bytes (21475 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (5.4%)
    Collecting resizing constraints ...
    Schedule chkdsk for NTFS consistency check at Windows boot time ...
    Resetting $LogFile ... (this might take a while)
    Updating $BadClust file ...
    Updating $Bitmap file ...
    Updating Boot record ...
    The read-only test run ended successfully.

Jak już jesteśmy pewni co do swoich czynów i ustawiliśmy wszystko jak trzeba, usuwamy opcję `-n` :

    # ntfsresize -s 21474836480 -f /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 9999995392 bytes (10000 MB)
    Current device size: 21474836480 bytes (21475 MB)
    New volume size    : 21474832896 bytes (21475 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (5.4%)
    Collecting resizing constraints ...
    WARNING: Every sanity check passed and only the dangerous operations left.
    Make sure that important data has been backed up! Power outage or computer
    crash may result major data loss!
    Are you sure you want to proceed (y/[n])? y
    Schedule chkdsk for NTFS consistency check at Windows boot time ...
    Resetting $LogFile ... (this might take a while)
    Updating $BadClust file ...
    Updating $Bitmap file ...
    Updating Boot record ...
    Syncing device ...
    Successfully resized NTFS on device '/dev/sdb1'.

I jeszcze dla pewności sprawdzamy, czy aby wszystko jest w porządku:

    # ntfsresize -i -f /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Device name        : /dev/sdb1
    NTFS volume version: 3.1
    Cluster size       : 4096 bytes
    Current volume size: 21474832896 bytes (21475 MB)
    Current device size: 21474836480 bytes (21475 MB)
    Checking filesystem consistency ...
    100.00 percent completed
    Accounting clusters ...
    Space in use       : 538 MB (2.5%)
    Collecting resizing constraints ...
    You might resize at 537395200 bytes or 538 MB (freeing 20937 MB).
    Please make a test run using both the -n and -s options before real resizing!

Rozmiary partycji i systemu plików są z grubsza takie same no i żaden program nie zwraca błędów,
zatem wszystko musi być w porządku.
