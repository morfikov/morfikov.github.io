---
author: Morfik
categories:
- Linux
date: "2015-12-17T18:38:25Z"
date_gmt: 2015-12-17 17:38:25 +0100
published: true
status: publish
tags:
- luks
- hdd/ssd
title: Zmiana rozmiaru kontenera LUKS
---

Ci z nas, którzy zabezpieczają swoje systemy przy pomocy technik kryptograficznych wiedzą, że taki
system trzeba traktować nieco inaczej niż ten, który nie jest w żaden sposób zaszyfrowany. Gdy mamy
na swoim dysku kilka [kontenerów LUKS](https://pl.wikipedia.org/wiki/Linux_Unified_Key_Setup) (czy
też TrueCrypt), problematyczna może się okazać zmiana rozmiaru tego typu partycji. Praktycznie żadne
narzędzia graficzne, ewentualnie inne automaty, nie są w stanie nas przez ten proces bezstresowo
przeprowadzić. Musimy zatem skorzystać z niskopoziomowych aplikacji, takich jak `fdisk` czy
`cryptsetup` , by ten rozmiar sobie dostosować. Problem w tym, że nieumiejętne operowanie na tych
narzędziach może skończyć się tragicznie dla zgromadzonych na dysku danych. W tym wpisie postaram
się opisać cały proces zmiany rozmiaru zaszyfrowanego kontenera LUKS wliczając w to również
dostosowanie partycji i jej systemu plików.

<!--more-->
## Przygotowanie kontenera LUKS

W przypadku zwykłych, niezaszyfrowanych partycji, zmiana rozmiaru nie jest niczym trudnym.
Praktycznie do każdego typu systemu plików (w tym też do
[EXT4]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ext4/),
[FAT32]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-fat32/),
[NTFS]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ntfs-pod-linuxem/)) posiadamy dedykowane
narzędzia, które potrafią zmienić ich rozmiar. Wszystkie te narzędzia operują głównie na systemach
plików. Z kolei do zmiany rozmiaru partycji mamy `fdisk` . Widzimy zatem, że struktura jest
warstwowa. Najpierw jest partycja, a na niej system plików. W przypadku zaszyfrowanych partycji,
dodawana jest nowa warstwa między te dwie powyższe.

W debianie, do obsługi zaszyfrowanych kontenerów LUKS wykorzystuje się narzędzia z pakietu
`cryptsetup-bin` . Zakładam, że mamy już do dyspozycji jakiś kontener, którego rozmiar chcemy
zmienić. Nie będę tutaj opisywał procesu tworzenia kontenera, bo to jest zagadnienie na osoby
artykuł. Tak czy inaczej, kontener musi zostać otwarty:

    # cryptsetup luksOpen /dev/sdb1 sdb1
    Enter passphrase for /dev/sdb1:

    # lsblk /dev/sdb
    NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sdb                8:16   0  74.5G  0 disk
    └─sdb1             8:17   0  37.3G  0 part
      └─sdb1 (dm-10) 254:10   0  37.3G  0 crypt

W zależności od tego jaki system plików posiadamy w kontenerze, trzeba skorzystać z odpowiedniego
narzędzia do sprawdzenia integralności danych. W tym przypadku, w kontenerze znajduje się system
plików EXT4. Zatem korzystamy z `fsck.ext4` , podając mu ścieżkę do odszyfrowanego systemu plików
zlokalizowanego w `/dev/mapper/` , przykładowo:

    # fsck.ext4 -Dvf /dev/mapper/sdb1

## Zmniejszanie kontenera LUKS

Jeśli stworzyliśmy kiedyś zaszyfrowaną partycję, z której rozmiaru obecnie nie jesteśmy zadowoleni i
chcielibyśmy ją zmniejszyć, to nic nie stoi na przeszkodzie, by to zrobić. Kolejność działań jest
mniej więcej taka sama jak w przypadku zmiany rozmiaru zwykłej partycji. Najpierw zmieniamy rozmiar
systemu plików, a potem... no właśnie, co potem? Trzeba będzie zmniejszyć rozmiar kontenera, a
dopiero później przyciąć samą partycję. Zatem do dzieła, zmieniamy rozmiar systemu plików EXT4 z
obecnych 37 GiB na 10 GiB przy pomocy `resize2fs` :

    # resize2fs -p /dev/mapper/sdb1 10G
    resize2fs 1.42.9 (28-Dec-2013)
    Resizing the filesystem on /dev/mapper/sdb1 to 2621440 (4k) blocks.
    Begin pass 2 (max = 78514)
    Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 3 (max = 299)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 4 (max = 915)
    Updating inode references     XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/mapper/sdb1 is now 2621440 blocks long.

Mając ciągle otwarty kontener ale niepodmontowany zasób, zmieniamy jego rozmiar przez `cryptsetup
resize` . Przy czym, ten parametr przyjmuje wartości w sektorach, a wyżej mamy ilość bloków
(2621440), z których każdy ma 4096 bajtów. Trzeba to przeliczyć na sektory 512 bajtowe, czyli
wartość 2621440 pomnożyć przez 8 (4096/512), co daje 20971520-1=20971519 sektorów. Tylko tutaj
ważna uwaga. Ta liczba bloków nie uwzględnia nagłówka zaszyfrowanej partycji. A więc jest to tylko
rozmiar systemu plików. Jeśli nie wiemy ile zajmuje nagłówek naszej partycji, możemy to sprawdzić
przez:

    # cryptsetup status sdb1
    /dev/mapper/sdb1 is active.
      type:    LUKS1
      cipher:  aes-xts-plain64
      keysize: 512 bits
      device:  /dev/sdb1
      offset:  4096 sectors
      size:    78229504 sectors
      mode:    read/write

Jest to 4096 sektorów 512-bajtowych, co daje 2097152 bajtów, 2097152/1024/1024=2 MiB . Jeśli teraz
byśmy nie uwzględnili w wyliczeniach tego nagłówka i próbowali dostosować partycję obcinając te
dodatkowe bloki, to przy próbie montowania dostaniemy błąd:

    # mount /dev/mapper/sdb1 /mnt
    mount: wrong fs type, bad option, bad superblock on /dev/mapper/sdb1,
           missing codepage or helper program, or other error
           In some cases useful info is found in syslog - try
           dmesg | tail  or so

A w syslogu poniższy
    komunikat:

    kernel: [40192.322415] EXT4-fs (dm-10): bad geometry: block count 2621440 exceeds size of device (2620928 blocks)

Sama partycja otworzy się bez problemu, bo obcięliśmy kawałek z tyłu partycji. Jeśli przyjrzymy się
bliżej temu powyższemu komunikatowi, to 2621440-2620928=512. Jest to 512 bloków, a każdy z nich ma
4096 bajtów, co daje w sumie wartość 2097152 (512\*4096). A to z kolei jest 2097152/1024/1024=2 MiB,
czyli tyle samo, co w przypadku ustalania wielkości nagłówka zaszyfrowanego dysku. Trzeba pamiętać,
by do faktycznego rozmiaru bloków wskazanego przez `resize2fs` (2621440) dodać 512 bloków i dopiero
w oparciu o te bloki, tj. (2621440+512)\*8-1=20975615, zmienić rozmiar partycji w `fdisk` . Nie ma
się co stresować i jeśli źle coś policzymy, to po prostu trzeba poprawić wpis w `fdisk` w oparciu o
nowe obliczenia.

W przypadku kontenera LUKS, stosujemy sektory bez offsetu, tj. 2621440\*8-1=20971519 :

    # cryptsetup resize sdb1 --size 20971519

    # lsblk /dev/sdb
    NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sdb                8:16   0  74.5G  0 disk
    └─sdb1             8:17   0  37.3G  0 part
      └─sdb1 (dm-10) 254:10   0    10G  0 crypt

Rozmiar kontenera `sdb1` uległ zmianie ale partycja nadal ma 37 GiB. By zmienić rozmiar partycji,
zamykamy kontener i odpalamy `fdisk` :

    # cryptsetup luksClose sdb1
    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    78235647    39116800   83  Linux

    Command (m for help): d
    Selected partition 1

    Command (m for help): n
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-156299374, default 156299374): +20975615

    Command (m for help): t
    Selected partition 1
    Hex code (type L to list codes): 83
    Changed system type of partition 1 to 83 (Linux)

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    20977663    10487808   83  Linux

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

No to teraz by sprawdzić czy wszystko działa, otwieramy kontener:

    # cryptsetup luksOpen /dev/sdb1 sdb1
    Enter passphrase for /dev/sdb1:

    # lsblk /dev/sdb
    NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sdb                8:16   0  74.5G  0 disk
    └─sdb1             8:17   0    10G  0 part
      └─sdb1 (dm-10) 254:10   0    10G  0 crypt

Rozmiar partycji `sdb1` uległ zmniejszeniu. Montujemy system plików i sprawdzamy czy zawarte w nim
dane są możliwe do odczytania:

    # mount /dev/mapper/sdb1 /mnt
    # ls -al /mnt

Jeśli jesteśmy w stanie odczytać zawartość głównego katalogu, znaczy to nic innego jak tylko to, że
proces zmniejszenia rozmiaru zaszyfrowanej partycji zakończył się sukcesem.

## Powiększanie partycji LUKS

No to tak jeszcze, by w pełni wyczerpać temat zmiany rozmiaru zaszyfrowanej partycji, trzeba
rozpatrzyć drugi wariant, czyli zwiększenie rozmiaru kontenera LUKS. Tak obecnie wygląda ułożenie
partycji na dysku:

    # lsblk /dev/sdb
    NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sdb                8:16   0  74.5G  0 disk
    └─sdb1             8:17   0    10G  0 part
      └─sdb1 (dm-10) 254:10   0    10G  0 crypt

Zamykamy kontener:

    # cryptsetup luksClose sdb1

W fdisk'u zaś kasujemy starą partycję i tworzymy nową, powiedzmy +15G. Czyli nowy rozmiar kontenera
wyniesie 25 GiB:

    # fdisk /dev/sdb

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    139 heads, 49 sectors/track, 22948 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    20973567    10485760   83  Linux

    Command (m for help): d
    Selected partition 1

    Command (m for help): n
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1):
    Using default value 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-156299374, default 156299374): +25G

    Command (m for help): t
    Selected partition 1
    Hex code (type L to list codes): 83

    Command (m for help): p

    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    139 heads, 49 sectors/track, 22948 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    52430847    26214400   83  Linux

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

Otwieramy kontener:

    # cryptsetup luksOpen /dev/sdb1 sdb1
    Enter passphrase for /dev/sdb1:

    # lsblk /dev/sdb
    NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sdb                8:16   0  74.5G  0 disk
    └─sdb1             8:17   0    25G  0 part
      └─sdb1 (dm-10) 254:10   0    25G  0 crypt

Rozmiar partycji i kontenera LUKS uległ zmianie. Trzeba jeszcze rozszerzyć system plików. Tylko
tutaj jest problem związany z nagłówkami. W fdisk'u ustawiliśmy 25G, czyli jest to
25\*1024\*1024\*1024=26843545600 bajtów. Z kolei 26843545600/512 daje 52428800 sektorów. Co
przekłada się na 52428800/2=26214400 bloków (tych w fdisku) lub 52428800/8=6553600 bloków w
`resize2fs` . Jeśli byśmy spróbowali ustawić 25G jako rozmiar, dostaniemy błąd:

    # resize2fs -p /dev/mapper/sdb1 25G
    resize2fs 1.42.9 (28-Dec-2013)
    The containing partition (or device) is only 6553088 (4k) blocks.
    You requested a new size of 6553600 blocks.

Oczywiście można, by pójść na łatwiznę i wpisać jeszcze raz rozmiar, tym razem używając liczby
6553088, którą wypisał powyżej `resize2fs` . Ale my to policzymy. By system plików miał odpowiedni
rozmiar, trzeba odjąć rozmiar nagłówków, a to jest 512 bloków 4096-bajtowych (512\*4096=2097152).
Czyli rozmiar nowego systemu plików wynieść powinien 26843545600-2097152=26841448448 bajtów.
Narzędzie `resize2fs` nie przyjmie rozmiaru w bajtach (tylko w przypadku K,M,G). Za to przyjmuje
rozmiar w sektorach. Zatem nowy rozmiar powinien wynieść 26841448448/512=52424704 sektorów. Jeśli
chcielibyśmy operować na blokach: 6553600−512=6553088 bloków, 6553088\*4096=26841448448 bajtów,
26841448448/512=52424704 sektorów, 52424704/8=6553088 bloków 4096-bajtowych (4096=8\*512). Dokładnie
tyle samo ile zasugerował `resize2fs` . Sprawdźmy zatem czy `resize2fs` przyjmie rozmiar 52424704s :

    # resize2fs -p /dev/mapper/sdb1 52424704s
    resize2fs 1.42.9 (28-Dec-2013)
    Resizing the filesystem on /dev/mapper/sdb1 to 6553088 (4k) blocks.
    The filesystem on /dev/mapper/sdb1 is now 6553088 blocks long.

Weszło. W przypadku, gdyby pojawił się poniższy komunikat:

    # resize2fs -p /dev/mapper/sdb1 52424704s
    resize2fs 1.42.9 (28-Dec-2013)
    Please run 'e2fsck -f /dev/mapper/sdb1' first.

Trzeba przeskanować system plików i ponowić żądanie zmiany rozmiaru. Montujemy jeszcze system plików
i sprawdzamy czy są w nim wcześniej stworzone pliki:

    # mount /dev/mapper/sdb1 /mnt
    # ls -al /mnt/

I to wszystko co się tyczy zmiany rozmiaru zaszyfrowanego kontenera.
