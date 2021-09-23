---
author: Morfik
categories:
- Linux
date: "2015-12-18T17:01:12Z"
date_gmt: 2015-12-18 16:01:12 +0100
published: true
status: publish
tags:
- luks
- system-plików
- hdd
- ssd
- lvm
title: Kopia struktury dysku twardego
---

Wszyscy wiedzą, że zabawa z dyskiem zwykle kończy się tragicznie dla zawartych na nim danych. W tym
wpisie spróbujemy się przyjrzeć sytuacjom, które nie jednego użytkownika linux'a potrafią przyprawić
o zawał serca. Chodzi generalnie o uszkodzenie struktury dysku, oczywiście tylko tej programowej. Na
błędy fizyczne nie możemy zbytnio nic poradzić. Natomiast jeśli chodzi o sferę logiczną, to mamy
tutaj dość duże pole do popisu i jesteśmy w stanie się zabezpieczyć przed wieloma krytycznymi
sytuacjami. Postaramy się tutaj omówić to jak wykonać kopię dysku. Taka kopia będzie miała tylko
kilka (ew. kilkanaście) MiB, w skład której wchodzić będzie superblok systemu plików, nagłówki
zaszyfrowanych partycji LUKS, struktura LVM i tablica partycji. Uszkodzenie każdego z tych
powyższych uniemożliwia nam dostęp do danych zgromadzonych na dysku.

<!--more-->
## Objawy przy złym przycięciu partycji

Gdy bawimy się naszym dyskiem i zmieniamy w nim układ partycji, to jest niemal pewne, że prędzej czy
później popełnimy błąd i podamy złe parametry w `fdisk` . Jeśli zmniejszymy za bardzo partycję i
obetniemy jej trochę systemu plików, to w zależności od posiadanego systemu plików dostaniemy różne
informacje z błędami. Poniżej jest kilka takich komunikatów.

W przypadku, gdy [zmienialiśmy rozmiar systemu plików
NTFS](/post/zmiana-rozmiaru-partycji-ntfs-pod-linuxem/) i zrobiliśmy to błędnie,
`ntfsresize` zwróci nam poniższy komunikat:

    # ntfsresize -i -f /dev/sdb1
    ntfsresize v2013.1.13AR.1 (libntfs-3g)
    Failed to read last sector (19531240): Invalid argument
    HINTS: Either the volume is a RAID/LDM but it wasn't setup yet,
       or it was not setup correctly (e.g. by not using mdadm --build ...),
       or a wrong device is tried to be mounted,
       or the partition table is corrupt (partition is smaller than NTFS),
       or the NTFS boot sector is corrupt (NTFS size is not valid).
    ERROR(22): Opening '/dev/sdb1' as NTFS failed: Invalid argument
    The device '/dev/sdb1' doesn't have a valid NTFS.
    Maybe you selected the wrong partition? Or the whole disk instead of a
    partition (e.g. /dev/hda, not /dev/hda1)? This error might also occur
    if the disk was incorrectly repartitioned (see the ntfsresize FAQ).

Gdy próbowaliśmy zaś [zmienić rozmiar systemu plików
EXT4](/post/zmiana-rozmiaru-partycji-ext4/) i również coś pochrzaniliśmy, to przy
sprawdzaniu błędów w `fsck` , ten zwróci nam poniższe zapytanie:

    # fsck.ext4 -fv /dev/sdb1
    e2fsck 1.42.9 (28-Dec-2013)
    The filesystem size (according to the superblock) is 1280000 blocks
    The physical size of the device is 786432 blocks
    Either the superblock or the partition table is likely to be corrupt!
    Abort? yes

Natomiast jeśli chodzi o problemy przy [zmianie rozmiaru systemu plików
FAT32](/post/zmiana-rozmiaru-partycji-fat32/), to `dosfsck` będzie miał poniższy
problem:

    # dosfsck -v /dev/sdb1
    dosfsck 3.0.16 (01 Mar 2013)
    dosfsck 3.0.16, 01 Mar 2013, FAT32, LFN
    Checking we can access the last sector of the filesystem
    Seek to 5242879488:Invalid argument

W przypadku, gdy [zmienialiśmy rozmiar partycji LVM](/post/zmiana-rozmiaru-lvm/) i
przycięliśmy ją za bardzo w `fdisk` , to przy skanowaniu voluminów w `pvscan` dostaniemy taki
komunikat:

    # pvscan
      /dev/grupa1/volumin5: read failed after 0 of 4096 at 12947750912: Błąd wejścia/wyjścia
      /dev/grupa1/volumin5: read failed after 0 of 4096 at 12947808256: Błąd wejścia/wyjścia
      PV /dev/sdb1   VG grupa1   lvm2 [19,06 GiB / 0    free]
      Total: 1 [19,06 GiB] / in use: 1 [19,06 GiB] / in no VG: 0 [0   ]

Oczywiście nie da rady w takich przypadkach zamontować żadnego przyciętego systemu plików. Jeśli
spróbujemy to `mount` przy próbie zamontowania takiego zasobu wyrzuci poniższy komunikat:

    mount: wrong fs type, bad option, bad superblock on /dev/sdb1,
           missing codepage or helper program, or other error
           In some cases useful info is found in syslog - try
           dmesg | tail  or so

Bez obaw. Trzeba po prostu znów odpalić fdisk'a i zrobić partycję nieco większą. Żadnych danych w
ten sposób nie stracimy, gdyż operujemy jedynie na tablicy partycji, a ta ma w swojej strukturze
parę pozycji, z których tylko 3 są brane pod uwagę, przynajmniej w linux'ie. Są to sektor
początkowy partycji, rozmiar w sektorach, oraz typ partycji. Możemy dowolnie się tymi parametrami
bawić bez utraty danych, tylko lepiej pamiętać pozycję wyjściową.

## Ryzyko utraty danych

Ryzyko utraty danych zawsze istnieje, zwłaszcza gdy obchodzimy się z systemem plików bardzo
nieumiejętnie ale wystarczy odrobina rozsądku i kalkulator, aby to ryzyko całkowicie wyeliminować.
Poniżej kilka wskazówek na temat tego jak ma wyglądać kopia struktury dysku twardego i jak ją
wykonać.

### Kopia wpisów tablicy partycji MS-DOS

Zanim się zaczniemy bawić z tablicą partycji, która jest zlokalizowana w
[MBR](https://pl.wikipedia.org/wiki/Master_Boot_Record) (przynajmniej jej część), najlepiej jest
wykonać kopię zapasową tego sektora i zachować ją w bezpiecznym miejscu. Jeśli posiadamy partycję
rozszerzoną, przydałoby się również zrobić kopię każdego
[EBR](https://en.wikipedia.org/wiki/Extended_boot_record), a ich lokalizacja zależy od faktycznego
rozkładu dysków logicznych, przykładowo:

    Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

Są tutaj 3 EBR, a lokalizacja każdego z nich znajduje się zawsze w pierwszym sektorze logicznego
dysku. W przypadku pierwszego dysku logicznego jest to parametr "start" partycji rozszerzonej. W
przypadku pozostałych dysków sprawa ma się nieco inaczej, bo jest to sektor zaraz za końcem
poprzedniej partycji. Między tymi dyskami logicznymi jest trochę wolnego miejsca. W przypadku
równania do 1 MiB jest tam 2047 sektorów + 1 sektor na EBR. Jeśli byśmy wykonali kopię tej pozycji
co jest w parametrze "start", skopiujemy pierwszy sektor systemu plików, a nie EBR, i przywrócenie
tego sektora nic nam nie da.

[Kopię MBR za to możemy wykonać](/post/mbr-ebr-i-tablica-partycji-dysku-twardego/)
bardzo łatwo, bo MBR zawsze znajduje się na tej samej pozycji i jest to pierwszy sektor dysku. Kopię
możemy zrobić przy pomocy narzędzia `dd` , przykładowo:

    # dd if=/dev/sdb of=./mbr bs=512 count=1

W razie problemów, by przywrócić podstawową tablicę partycji możemy wgrać 64 bajty do MBR z
uprzednio zrobionej kopi zapasowej:

    # dd if=./mbr of=/dev/sdb bs=1 count=64 skip=446 seek=446

Kopie EBR robimy w ten sam sposób podając tylko odpowiednie adresy sektorów.

Zamiast jednak się bawić w robienie kopi MBR i EBR, możemy zrobić kopię wpisów w tablicy partycji
przy pomocy `sfdisk` :

    # sfdisk -d /dev/sdb > sdb_table
    # sfdisk /dev/sda < sdb_table

Tak wygląda przykładowa tablica partycji:

    # partition table of /dev/sdb
    unit: sectors

    /dev/sdb1 : start=     2048, size= 45639680, Id=83
    /dev/sdb2 : start= 45641728, size= 34525184, Id=83
    /dev/sdb3 : start= 80166912, size= 24057856, Id=83
    /dev/sdb4 : start=104224768, size= 52074496, Id= 5
    /dev/sdb5 : start=104226816, size= 15101952, Id=83
    /dev/sdb6 : start=119330816, size=  9390080, Id=83
    /dev/sdb7 : start=128722944, size= 27576320, Id=83

Jak coś nawali, to zawsze możemy ręcznie wpisywać wartości z tej powyższej tabelki do fdisk'a.

### Kopia wpisów tablicy partycji GPT

Jeśli ktoś używa tablicy partycji GPT, to nie może kierować się powyższymi wskazówkami co do kopi
struktury tablicy partycji, bo ta w przypadku GPT jest inna. Bez problemu możemy zrobić jej kopię i
zachować ją jako plik przy pomocy narzędzi `gdisk` lub `sgdisk` , przykładowo:

    # gdisk /dev/sdb
    GPT fdisk (gdisk) version 0.8.8

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.

    Command (? for help): b
    Enter backup filename to save: backup_gpt_file
    The operation has completed successfully.

Problem ze stworzonym w ten sposób plikiem jest taki, że nie jest on w postaci czytelnej dla
człowieka i trzeba przywracać całą kopię partycji, zamiast jedynie pojedynczych wpisów.

### Kopia nagłówków zaszyfrowanych partycji LUKS

Jeśli chodzi o zaszyfrowane kontenery LUKS, to trzeba pamiętać, że by się dostać do tego co jest w
środku, musimy podać odpowiedni klucz/hasło, a te są przechowywane w nagłówku partycji. Taki
nagłówek to zwykle 2 MiB danych. Kopię takiego nagłówka możemy zrobić ręcznie przy pomocy `dd`
albo też posłużyć się w tym celu natywnymi narzędziami LUKS. Jeśli zdecydujemy, aby tak kopia
nagłówka LUKS była zrobiona ręcznie, to musimy pierw uzyskać informacje na temat tego ile taki
nagłówek faktycznie zajmuje:

    # cryptsetup status sdb1
    /dev/mapper/sdb1 is active.
      type:    LUKS1
      cipher:  aes-xts-plain64
      keysize: 512 bits
      device:  /dev/sdb1
      offset:  4096 sectors
      size:    78229504 sectors
      mode:    read/write

W tym przypadku jest to 4096 sektorów, z których każdy ma 512 bajtów. Zatem wklepujemy te parametry
do `dd` :

    # dd if=/dev/sdb1 of=./backup_header_sdb1 bs=512 count=4096

W ten sposób utworzona kopia nagłówka może zostać przywrócona ręcznie w poniższy sposób:

    # dd if=./backup_header_sdb1 of=/dev/sdb1 bs=512 count=4096

Kopię nagłówka LUKS możemy także wykonać przy pomocy `luksHeaderBackup` , przykładowo:

    # cryptsetup luksHeaderBackup /dev/sdb1 --header-backup-file ./backup_header_sdb1

Przywracanie kopi zapasowej nagłówka LUKS odbywa się za sprawą `luksHeaderRestore` :

    # cryptsetup luksHeaderRestore /dev/sdb1 --header-backup-file ./backup_header_sdb1

### Kopia struktury LVM

LVM posiada narzędzia umożliwiające dokonanie kopi struktury kontenera i zapisanie jej w czytelnej
dla człowieka formie w pliku tekstowym. By zrobić sobie taką kopię, wpisujemy do terminala poniższą
komendę:

    # vgcfgbackup
      Volume group "grupa1" successfully backed up.

Zostanie w ten sposób utworzony plik `/etc/lvm/backup/grupa1` , który może nam posłużyć do
odtworzenia całej struktury, w przypadku, gdy ta zostanie w jakiś sposób uszkodzona.

### Kopia superbloku systemu plików

Superblok to krytyczny sektor każdego systemu plików. Zawiera on informacje, według których kernel
jest w stanie operować na danej partycji. W przypadku, gdy superblok zostaje uszkodzony, kernel nie
wie jak zamontować taki system plików, w efekcie czego nie możemy uzyskać dostępu do danych. Systemy
plików są w stanie chronić swój superblok kopiując go (kilka razy) i zapisując w innych
lokalizacjach w obrębie granic partycji. W taki sposób, jeśli uszkodzeniu ulegnie główny superblok,
to możemy przywrócić jego kopię z któregoś z tych zapasowych sektorów.

Lokalizację superbloku jak i jego kopi w systemie plików EXT4 możemy poznać za pomocą `dumpe2fs` :

    # dumpe2fs /dev/sdb1 | grep -i super
    ...
      Primary superblock at 0, Group descriptors at 1-1
      Backup superblock at 32768, Group descriptors at 32769-32769
      Backup superblock at 98304, Group descriptors at 98305-98305
      Backup superblock at 163840, Group descriptors at 163841-163841
    ...

W tym przypadku główny superblok jest ulokowany na pozycji `0` . Standardowo jest to pierwszy blok
systemu plików. Ilość zapasowych kopi zależy od rozmiaru systemu plików. W tym przypadku mamy do
czynienia z partycją 9 GiB. Zatem jest ona niewielka ale i tak mamy kilka kopi zapasowych
ulokowanych, min. w bloku `32768` , `98304` , `163840` . Na dobrą sprawę, to nie musimy uczyć się
tych pozycji na pamięć. W każdym przypadku są one takie same. Nie musimy także ręcznie dokonywać
kopi superbloku, bo system plików robi to za nas.

## Uszkodzenie struktury dysku

Może i zmiana rozmiaru LVM, LUKS i normalnych partycji potrafi przebiegać zwykle bez najmniejszych
problemów ale w życiu czasem różnie bywa i niekiedy struktura danych zawartych na dysku może zostać
rozbita. Zadajmy sobie pytanie, co się stanie jak badsector uderzy w MBR, czyli w miejsce gdzie
siedzą wpisy tablicy partycji albo też co się stanie, gdy uszkodzeniu ulegnie nagłówek zaszyfrowanej
partycji? Wyżej został opisany, co prawda, sposób dokonania kopi danych i wgrywania ich w przypadku
problemów ale to trochę takie bezpłciowe. Poniżej sprawdzimy czy faktyczne uszkodzenie struktury
dysku spowoduje utratę rzeczywistych danych.

### Uszkodzenie tablicy partycji

W przypadku tablicy partycji MS-DOS może ulec awarii MBR. W MBR poza kodem bootloader'a mamy
4\*16=64 bajty na tablicę partycji. Dobrze jest zrobić kopię całego MBR, a potem ewentualnie
przywracać jego części. MBR może przechować maksymalnie 4 wpisy odpowiadające 4 partycjom. Nie
przechowuje jednak wpisów, które są na partycji rozszerzonej.

Partycję rozszerzoną można traktować jako dysk zagnieżdżony w innym dysku. Posiada swój MBR z tym,
że nieco inny. EBR ma taką samą strukturę co MBR ale jest pozbawiony kodu bootloader'a i
wykorzystuje dwa z czterech wpisów na partycje. Jeden z nich opisuje partycję (start partycji, bez
offsetu, oraz długość partycji), a drugi linkuje do kolejnego EBR (start w miejscu nowego EBR,
rozmiar całej partycji począwszy od EBR). W przypadku ostatniej partycji w łańcuchu EBR, ten drugi
wpis ma 0 bajtów.

Co się zatem mogłoby stać? Mógłby zostać uszkodzony wpis podstawowej partycji w MBR, w takim
przypadku stracilibyśmy jedną partycję. Mógłby zostać też trafiony wpis rozszerzony. W wyniku
takiego zdarzenia stracilibyśmy wszystkie partycje w łańcuchu EBR. Jeśli mamy 4 dyski logiczne na
rozszerzonej partycji i uszkodzeniu uległby drugi EBR, stracilibyśmy nie tylko 2 dysk logiczny ale
również 3 i 4.

Tak wygląda przykładowy dysk, ma 3 partycje podstawowe i 3 rozszerzone:

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

Skopiujmy teraz całą tablicę partycji:

    # sfdisk -d /dev/sdb > ./sdb_table
    Warning: extended partition does not start at a cylinder boundary.
    DOS and Linux will interpret the contents differently.

    # cat sdb_table
    # partition table of /dev/sdb
    unit: sectors

    /dev/sdb1 : start=     2048, size= 45639680, Id=83
    /dev/sdb2 : start= 45641728, size= 34525184, Id=83
    /dev/sdb3 : start= 80166912, size= 24057856, Id=83
    /dev/sdb4 : start=104224768, size= 52074496, Id= 5
    /dev/sdb5 : start=104226816, size= 15101952, Id=83
    /dev/sdb6 : start=119330816, size=  9390080, Id=83
    /dev/sdb7 : start=128722944, size= 27576320, Id=83

W przypadku, gdy zniknie nam jeden wpis, możemy odtworzyć ręcznie tylko ten, którego brakuje. Czas
nabroić trochę. Skoro tablica partycji to 64 bajty po 446 bajtach, to uszkodzimy drugą partycję
przez zapisanie 5 bajtów z tych 16 opisujących tą partycję. Zatem 446+16=462 , czyli na 462 zaczyna
się drugi wpis, jako że numerowanie zaczyna się od 0, a nie od 1:

    # dd if=/dev/zero of=/dev/sdb bs=1 seek=462 count=5
    5+0 records in
    5+0 records out
    5 bytes (5 B) copied, 0.00636016 s, 0.8 kB/s
    # partprobe

I patrzymy cośmy uczynili:

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592    0  Empty
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

No ładnie druga partycja zniszczona i oznaczona jako "empty". Nie jest to, co prawda, wolna
przestrzeń ale dane są pozornie utracone. Wpis partycji ma szereg bajtów opisujących jej strukturę i
w tym przypadku nadpisaliśmy tylko typ partycji ale w realiach może się zdarzyć, że nieczytelne będą
inne bajty opisujące, np. początek partycji. Jak zatem naprawić szkody, które spowodowaliśmy? W tym
przypadku wystarczy na nowo ustawić typ partycji:

    # fdisk /dev/sdb

    Command (m for help): t
    Partition number (1-7): 2
    Hex code (type L to list codes): 83
    Changed system type of partition 2 to 83 (Linux)

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

A w innych przypadkach? Można oczywiście usunąć cały wpis w tablicy partycji i stworzyć go na nowo.

    Command (m for help): d
    Partition number (1-7): 2
    Warning: partition 2 has empty type
    Command (m for help): n
    Partition type:
       p   primary (2 primary, 1 extended, 1 free)
       l   logical (numbered from 5)
    Select (default p): p
    Selected partition 2
    First sector (45641728-156299374, default 45641728):
    Using default value 45641728
    Last sector, +sectors or +size{K,M,G} (45641728-80166911, default 80166911):
    Using default value 80166911

    Command (m for help): t
    Partition number (1-7): 2
    Hex code (type L to list codes): 83
    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

### Uszkodzenie partycji rozszerzonej

Scenariusz drugi, czyli uszkodzenie partycji rozszerzonej.

    # dd if=/dev/zero of=/dev/sdb bs=1 seek=494 count=5
    5+0 records in
    5+0 records out
    5 bytes (5 B) copied, 0.00958093 s, 0.5 kB/s
    # partprobe

W tym przypadku partycja rozszerzona będzie pokazywana jako partycja podstawowa i nie da rady
zmienić jej typu na 5 (partycja rozszerzona). Spróbujemy przywrócić cały wpis. Jeśli nie wiemy jaki
powinien być rozmiar partycji rozszerzonej, możemy nam pomóc kopia zrobiona przez `sfdisk` :

    /dev/sdb4 : start=104224768, size= 52074496, Id= 5

Początkowy sektor to 104224768. Rozmiar zaś to 52074496 sektorów. Pamiętajmy tylko, by od 52074496
odjąć 1 przy wpisywaniu rozmiaru w `fdisk` :

    root:~# fdisk /dev/sdb
    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    0  Empty

    Command (m for help): d
    Partition number (1-5): 4

    Command (m for help): n
    Partition type:
       p   primary (3 primary, 0 extended, 1 free)
       e   extended
    Select (default e): e
    Selected partition 4
    First sector (104224768-156299374, default 104224768): 104224768
    Last sector, +sectors or +size{K,M,G} (104224768-156299374, default 156299374): +52074495

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

Jak widać stworzenie nowej partycji rozszerzonej nie przywróciło dysków logicznych. Jak je
przywrócić? Trzeba ręcznie przepisać wpisy z kopi zrobionej przez `sfdisk` :

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended

    Command (m for help): n
    All primary partitions are in use
    Adding logical partition 5
    First sector (104226816-156299263, default 104226816): 104226816
    Last sector, +sectors or +size{K,M,G} (104226816-156299263, default 156299263): +15101951

    Command (m for help): n
    All primary partitions are in use
    Adding logical partition 6
    First sector (119330816-156299263, default 119330816): 119330816
    Last sector, +sectors or +size{K,M,G} (119330816-156299263, default 156299263): +9390079

    Command (m for help): n
    All primary partitions are in use
    Adding logical partition 7
    First sector (128722944-156299263, default 128722944): 128722944
    Last sector, +sectors or +size{K,M,G} (128722944-156299263, default 156299263): +27576319

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

Jeszcze tylko sprawdźmy czy damy radę zamontować partycję numer 6:

    # mount /dev/sdb6 /mnt

Można również wgrać pierwszy EBR, jego pozycja znajduje się zawsze na początku partycji
rozszerzonej, a przy tworzeniu nowej rozszerzonej partycji ten sektor jest nadpisywany. To dlatego
nie ma dysków logicznych. Jego kopię w tym przypadku można by sporządzić przy pomocy `dd` w poniższy
sposób:

    # dd if=/dev/sdb bs=512 skip=104224768 count=1 of=./ebr

I jeśli stracimy rozszerzony wpis w MBR, tworzymy go na nowo w fdisk'u:

    # fdisk /dev/sdb

    Command (m for help): d
    Partition number (1-7): 4
    Command (m for help): n
    Partition type:
       p   primary (3 primary, 0 extended, 1 free)
       e   extended
    Select (default e): e
    Selected partition 4
    First sector (104224768-156299374, default 104224768): 104224768
    Last sector, +sectors or +size{K,M,G} (104224768-156299374, default 156299374): +52074495

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.

I wgrywamy kopię EBR:

    # dd if=./ebr bs=512 seek=104224768 count=1 of=/dev/sdb
    1+0 records in
    1+0 records out
    512 bytes (512 B) copied, 0.000119726 s, 4.3 MB/s
    # partprobe

I co widzimy w fdisk'u?

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

Logiczne dyski wróciły na swoje miejsce. Sprawdźmy jeszcze czy da radę zamontować partycję numer 6
bez błędów:

    # mount /dev/sdb6 /mnt

Ostatnie uszkodzenie będzie polegać na nadpisaniu EBR poprzedzającego partycję `sdb6` . Tylko tutaj
taka uwaga, nie możemy nadpisywać sektora 119330816 (start sda6), bo tutaj jest uwzględniony offset.
EBR tej partycji znajduje się zaraz za końcem partycji poprzedzającej, czyli: 119328767+1=119328768
i to ten sektor należy uszkodzić.

    # dd if=/dev/zero bs=512 seek=119328768 of=/dev/sdb count=1
    1+0 records in
    1+0 records out
    512 bytes (512 B) copied, 0.00091464 s, 560 kB/s
    # partprobe

I sprawdźmy:

    # fdisk /dev/sdb
    omitting empty partition (6)

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux

Tak jak można było się spodziewać, zniknęły dwie partycje. Ta, której EBR nadpisaliśmy oraz następne
w łańcuchu EBR, a że była tylko jedna, to w sumie zniknęły dwie. Oczywiście w tym przypadku sprawa
wygląda podobnie, trzeba przywrócić EBR. A co jeśli nie mamy EBR? Można, by skorzystać z kopi
sfdisk'a. W sumie to sfdisk rozwiązuje wszystkie problemy. Zatem wchodzimy do fdisk'a i przywracamy
dwa zagubione wpisy:

    # fdisk /dev/sdb
    omitting empty partition (6)

    Command (m for help): n
    All primary partitions are in use
    Adding logical partition 6
    First sector (119330816-156299263, default 119330816): 119330816
    Last sector, +sectors or +size{K,M,G} (119330816-156299263, default 156299263): +9390079

    Command (m for help): n
    All primary partitions are in use
    Adding logical partition 7
    First sector (128722944-156299263, default 128722944): 128722944
    Last sector, +sectors or +size{K,M,G} (128722944-156299263, default 156299263): +27576319

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

Montujemy i sprawdzamy czy są jakieś błędy:

    # mount /dev/sdb6 /mnt

Także kopia pełnej tablicy partycji uchroni nas przed utratą danych wynikającą z jej uszkodzenia.

### Uszkodzenie struktury LVM

Przetestujmy zatem czy kopia zapasowa struktury LVM, która jest trzymana w katalogu
`/etc/lvm/backup/` uchroni nas przed czymś. Popsujmy celowo cały kontener LVM:

    # vgremove grupa1
    Do you really want to remove volume group "grupa1" containing 3 logical volumes? [y/n]: y
    Do you really want to remove active logical volume volumin1? [y/n]: y
      Logical volume "volumin1" successfully removed
    Do you really want to remove active logical volume volumin3? [y/n]: y
      Logical volume "volumin3" successfully removed
    Do you really want to remove active logical volume volumin5? [y/n]: y
      Logical volume "volumin5" successfully removed
      Volume group "grupa1" successfully removed

    # pvremove /dev/sdb1
      Labels on physical volume "/dev/sdb1" successfully wiped

    # lsblk /dev/sdb
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb      8:16   0  74,5G  0 disk
    └─sdb1   8:17   0  19,1G  0 part

No to ładnie namieszaliśmy. Spróbujmy to teraz przywrócić przy pomocy `vgcfgrestore` :

    # vgcfgrestore -v grupa1 -f /etc/lvm/backup/grupa1
      Couldn't find device with uuid xMddCU-5h1X-ZZL4-GFfO-ea4L-PqW8-YCGLte.
      Cannot restore Volume Group grupa1 with 1 PVs marked as missing.
      Restore failed.

    # pvcreate --uuid xMddCU-5h1X-ZZL4-GFfO-ea4L-PqW8-YCGLte --restorefile /etc/lvm/backup/grupa1 /dev/sdb1
      Couldn't find device with uuid xMddCU-5h1X-ZZL4-GFfO-ea4L-PqW8-YCGLte.
      Physical volume "/dev/sdb1" successfully created

    # vgcfgrestore -vv -f /home/user/grupa1 grupa1
          Setting activation/monitoring to 0
          Setting global/locking_type to 1
          Setting global/wait_for_locks to 1
          File-based locking selected.
          Setting global/locking_dir to /run/lock/lvm
          Setting global/prioritise_write_locks to 1
          Locking /run/lock/lvm/V_grupa1 WB
          Locking /run/lock/lvm/P_orphans WB
          /dev/sdb: size is 156299375 sectors
          /dev/sdb1: size is 39976960 sectors
          /dev/sdb1: size is 39976960 sectors
          /dev/sdb1: lvm2 label detected at sector 1
      Restored volume group grupa1
          Unlocking /run/lock/lvm/P_orphans
          Unlocking /run/lock/lvm/V_grupa1

Sprawdzamy czy coś to dało:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  19,06g    0      0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  19,06g    0   1280   512 volumin3     0 linear /dev/sdb1:1280-1791
      /dev/sdb1  grupa1 lvm2 a--  19,06g    0   1792  3087 volumin5     0 linear /dev/sdb1:1792-4878

    # lvs -v --segments
        Finding all logical volumes
      LV       VG     Attr      Start SSize  #Str Type   Stripe Chunk
      volumin1 grupa1 -wc------    0   5,00g    1 linear     0     0
      volumin3 grupa1 -wc------    0   2,00g    1 linear     0     0
      volumin5 grupa1 -wc------    0  12,06g    1 linear     0     0

Podmontujmy jeszcze te voluminy:

    # vgchange -a y grupa1
      3 logical volume(s) in volume group "grupa1" now active
    # mount /dev/mapper/grupa1-volumin1 /media/1
    # mount /dev/mapper/grupa1-volumin3 /media/3
    # mount /dev/mapper/grupa1-volumin5 /media/5

    # lsblk /dev/sdb
    NAME                       MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb                          8:16   0  74,5G  0 disk
    └─sdb1                       8:17   0  19,1G  0 part
      ├─grupa1-volumin1 (dm-1) 254:1    0     5G  0 lvm  /media/1
      ├─grupa1-volumin3 (dm-2) 254:2    0     2G  0 lvm  /media/3
      └─grupa1-volumin5 (dm-3) 254:3    0  12,1G  0 lvm  /media/5

Wszystko działa jak trzeba. Także, jeśli kontener LVM się spieprzy i gdzieś leży kopia metadanych w
postaci pliku (na osobnym dysku), to możemy być spokojni. Co ciekawe, wszelkie zmiany struktury
kontenera LVM są odnotowywane i zapisywane w `/etc/lvm/archive/` . Są tam po prostu wszystkie
przeszłe konfiguracje kontenerów LVM. Jeśli manipulujemy rozmiarem kontenera LVM najlepiej zrobić
kopię całego foldera `/etc/lvm/` i trzymać ją na osobnym dysku.

### Uszkodzenie zaszyfrowanego kontenera LUKS

W przypadku zaszyfrowanych partycji LUKS, uszkodzeniu mogą ulec nagłówki uniemożliwiając nam dostęp
do zaszyfrowanych danych. Robimy zatem backup nagłówków:

    # cryptsetup luksHeaderBackup /dev/sdb1 --header-backup-file ./backup_header_sdb1
    # ls -al backup_header_sdb1
    -r-------- 1 root root 2.0M Jan 16 16:51 backup_header_sdb1

Nadpiszmy teraz nagłówki przy pomocy 20 sektorów 512 bajtowych:

    # dd if=/dev/zero of=/dev/sdb1 bs=512 count=20 seek=100
    20+0 records in
    20+0 records out
    10240 bytes (10 kB) copied, 0.00268263 s, 3.8 MB/s

Spróbujmy otworzyć kontener:

    # cryptsetup luksOpen /dev/sdb1 sdb1
    Enter passphrase for /dev/sdb1:
    No key available with this passphrase.
    Enter passphrase for /dev/sdb1:
    No key available with this passphrase.
    Enter passphrase for /dev/sdb1:
    No key available with this passphrase.

Kontener nie przyjmuje hasła, mimo iż jest ono prawidłowe. Jeśli nie otworzymy kontenera, to nie da
rady dostać się do danych. Jeśli sprawdzimy nagłówek, zobaczymy, że jakieś hasło tam jest
przechowywane:

    # cryptsetup luksDump /dev/sdb1
    LUKS header information for /dev/sdb1

    Version:        1
    Cipher name:    aes
    Cipher mode:    xts-plain64
    Hash spec:      sha512
    Payload offset: 4096
    MK bits:        512
    MK digest:      1e 6b c3 90 8f 4d ae 35 0c 08 38 16 5f 81 97 e2 a5 34 d0 ec
    MK salt:        da 47 d9 88 8f ef 97 79 99 96 cb 9d ca cd 4d 56
                    73 a1 5f 76 5b 44 a8 76 3d a4 a9 93 35 e1 7c fd
    MK iterations:  46875
    UUID:           88ddc490-8904-4fad-b072-39686730977b

    Key Slot 0: ENABLED
            Iterations:             185180
            Salt:                   01 88 98 93 3d 96 8c 31 5e ea d2 f4 d9 d8 52 9b
                                    66 a2 40 19 72 cb 46 c7 7f a4 1c 97 c2 d8 40 45
            Key material offset:    8
            AF stripes:             4000
    Key Slot 1: DISABLED
    Key Slot 2: DISABLED
    Key Slot 3: DISABLED
    Key Slot 4: DISABLED
    Key Slot 5: DISABLED
    Key Slot 6: DISABLED
    Key Slot 7: DISABLED

Ale jakież to hasło? Tego raczej nie zgadniemy. Przywróćmy zatem kopię nagłówka:

    # cryptsetup luksHeaderRestore /dev/sdb1 --header-backup-file ./backup_header_sdb1

    WARNING!
    ========
    Device /dev/sdb1 already contains LUKS header. Replacing header will destroy existing keyslots.

    Are you sure? (Type uppercase yes): YES

No to jeszcze sprawdźmy czy jesteśmy w stanie zamontować bez problemów ten zaszyfrowany system
plików:

    # cryptsetup luksOpen /dev/sdb1 sdb1
    Enter passphrase for /dev/sdb1:
    # mount /dev/mapper/sdb1 /mnt

Także posiadanie kopi zapasowej nagłówków zaszyfrowanych partycji jest iście obowiązkowe.

### Uszkodzenie superbloku systemu plików

Sprawdźmy co się stanie, gdy zostanie uszkodzony superblok sytemu plików. Tak wygląda przykładowy
dysk w `fsidk` :

    # fdisk -l /dev/sdb
    Disk /dev/sdb: 74.5 GiB, 80025280000 bytes, 156299375 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x000642e6

    Device     Boot    Start       End   Sectors  Size Id Type
    /dev/sdb1  *        2048  19531775  19529728  9.3G 83 Linux
    /dev/sdb2       19533822 156297215 136763394 65.2G  5 Extended
    /dev/sdb5       19533824  25100287   5566464  2.7G 82 Linux swap / Solaris
    /dev/sdb6       25102336 156297215 131194880 62.6G 83 Linux

By uszkodzić superblok partycji `sdb1` , musimy zapisać jej pierwszy blok. Niekoniecznie musi to być
pełny 4096-bajtowy blok. Zwróćmy uwagę, że zapisujemy urządzenie `/dev/sdb1` , a nie `/dev/sdb` .

    # dd if=/dev/urandom bs=4096 count=1 of=/dev/sdb1

W tej chwili, przy próbie zamontowania tego systemu plików via `mount` , kernel zwróci nam ten
poniższy komunikat:

    mount.nilfs2: Error while mounting /dev/sdb1 on /mnt: Invalid argument

Przy skanowaniu tego systemu plików w `fsck` zostanie zwrócona masa błędów, których nie powinniśmy w
żaden sposób naprawiać. Dlatego też jeśli już skanujemy taki dysk to zawsze róbmy to z opcją `-n` ,
która zapobiegnie wprowadzaniu zmian do tego uszkodzonego systemu plików.

Spróbujmy zatem przywrócić kopię superbloku, która jest przechowywana w jednym z zapasowych bloków,
dajmy na to `32768` . By przywrócić superblok, również posługujemy się narzędziem `fsck` , tylko
wskazujemy mu inną lokalizację superbloku, przykładowo:

    # fsck -y -f -b 32768 /dev/sdb1

System plików powinien zostać zmodyfikowany, a my powinniśmy być w stanie go bez problemu
zamontować. Nie powinniśmy też utracić żadnych danych.

## Wnioski

Jak widać w pokazanym przeze mnie doświadczeniu, mając w posiadaniu kalkulator i kopię określonej
struktury dysku w zależności od jego konfiguracji, dane przechowywane na takim nośniku są
bezpieczne. Zatem nie ma się co stresować przy zmianie poszczególnych parametrów, tylko ważne jest,
by była kopia zapasowa, z której to możemy wrócić do pozycji wyjściowej.
