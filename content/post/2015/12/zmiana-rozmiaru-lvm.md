---
author: Morfik
categories:
- Linux
date: "2015-12-17T20:15:26Z"
date_gmt: 2015-12-17 19:15:26 +0100
published: true
status: publish
tags:
- hdd/ssd
- lvm
title: Zmiana rozmiaru dysków w strukturze LVM
---

[Logical Volume Manager (LVM)](https://pl.wikipedia.org/wiki/LVM), czyli menadżer voluminów
logicznych w systemach linux, to mechanizm, który jest w stanie podzielić jedną partycję fizyczną na
szereg dysków logicznych. Każdy z tych dysków może być programowo dostosowany bez potrzeby edycji
czy zmiany tablicy partycji fizycznego dysku. Dzięki LVM jesteśmy w stanie obejść kilka ograniczeń
płynących z wykorzystywania tablicy partycji MS-DOS. Głównie chodzi tutaj o 4 partycje podstawowe,
które mieszczą się w [MBR](https://pl.wikipedia.org/wiki/Master_Boot_Record). Jeśli zdecydowaliśmy
się na wykorzystywanie LVM, to w pewnym momencie może okazać się, że niektóre voluminy mają zbyt
duże lub też zbyt małe rozmiary. Trzeba będzie je zatem dostosować i w tym wpisie postaramy się ten
proces zasymulować.

<!--more-->
## Pozyskiwanie informacji o LVM

Poniżej mamy przykładowy dysk, który zawiera jedną partycję podstawową, na której to znajduje się
kilka dysków logicznych. Tak wygląda ten dysk w `fdisk` :

    # fdisk -l /dev/sdb
    
    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    81922047    40960000   83  Linux

Nie widzimy tutaj żadnych dodatkowych dysków, tylko podstawową partycję `sdb1` . By być w stanie
zobaczyć voluminy logiczne, które są ulokowane na tej partycji, musimy podejrzeć dysk w `lsblk` :

    # lsblk /dev/sdb
    NAME                       MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb                          8:16   0  74,5G  0 disk
    └─sdb1                       8:17   0  39,1G  0 part
      ├─grupa1-volumin1 (dm-1) 254:1    0     5G  0 lvm  /media/1
      ├─grupa1-volumin2 (dm-2) 254:2    0     7G  0 lvm  /media/2
      ├─grupa1-volumin3 (dm-3) 254:3    0     9G  0 lvm  /media/3
      ├─grupa1-volumin4 (dm-4) 254:4    0    11G  0 lvm  /media/4
      └─grupa1-volumin5 (dm-5) 254:5    0   7,1G  0 lvm  /media/5

By zacząć cokolwiek robić z kontenerem LVM, musimy pierw uzyskać stosowne informacje. Musimy min.
wiedzieć jaki jest rozmiar fizycznego voluminu, z którego LVM może zrobić użytek. Uzyskamy go z
`pvscan` :

    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [39,06 GiB / 0    free]
      Total: 1 [39,06 GiB] / in use: 1 [39,06 GiB] / in no VG: 0 [0   ]

Musimy także poznać strukturę podziału tej przestrzeni na dyski logiczne. Możemy to wyciągnąć z
`pvs` i `lvs` , przykładowo:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  39,06g    0      0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  39,06g    0   1280  1792 volumin2     0 linear /dev/sdb1:1280-3071
      /dev/sdb1  grupa1 lvm2 a--  39,06g    0   3072  2304 volumin3     0 linear /dev/sdb1:3072-5375
      /dev/sdb1  grupa1 lvm2 a--  39,06g    0   5376  2816 volumin4     0 linear /dev/sdb1:5376-8191
      /dev/sdb1  grupa1 lvm2 a--  39,06g    0   8192  1807 volumin5     0 linear /dev/sdb1:8192-9998
    
    # lvs -v --segments
        Finding all logical volumes
      LV       VG     Attr      Start SSize  #Str Type   Stripe Chunk
      volumin1 grupa1 -wc-ao---    0   5,00g    1 linear     0     0
      volumin2 grupa1 -wc-ao---    0   7,00g    1 linear     0     0
      volumin3 grupa1 -wc-ao---    0   9,00g    1 linear     0     0
      volumin4 grupa1 -wc-ao---    0  11,00g    1 linear     0     0
      volumin5 grupa1 -wc-ao---    0   7,06g    1 linear     0     0

Jeśli ktoś preferuje sektory nad bajtami, wystarczy sprecyzować jednostkę przy pomocy `--units s` :

    # pvs -v --units s
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize     PFree DevSize   PV UUID
      /dev/sdb1  grupa1 lvm2 a--  81911808S    0S 81920000S xMddCU-5h1X-ZZL4-GFfO-ea4L-PqW8-YCGLte
    
    # lvs -v --segments --units s
        Finding all logical volumes
      LV       VG     Attr      Start SSize     #Str Type   Stripe Chunk
      volumin1 grupa1 -wc-ao---    0S 10485760S    1 linear     0S    0S
      volumin2 grupa1 -wc-ao---    0S 14680064S    1 linear     0S    0S
      volumin3 grupa1 -wc-ao---    0S 18874368S    1 linear     0S    0S
      volumin4 grupa1 -wc-ao---    0S 23068672S    1 linear     0S    0S
      volumin5 grupa1 -wc-ao---    0S 14802944S    1 linear     0S    0S

Jak widać z powyższych informacji, mamy 1 kontener o rozmiarze nieco ponad 39 GiB, na którym
znajduje się 5 voluminów logicznych. Każdy z nich ma strukturę ciągłą, tzn. że zakresy bloków
występują jeden po drugim. Wiąże się z tym problem zmiany rozmiaru dysków. Chodzi o to, że w takiej
konfiguracji zachowują się one jak zwykłe partycje i limitowane są tym co leży obok nich. W tym
przypadku można, by rozciągnąć i skurczyć bez problemu tylko ostatni dysk. W przypadku, gdy dyski
nie mają struktury ciągłej, można bez problemu obracać nimi i nie ma znaczenia, gdzie one są, byleby
tylko miejsca w kontenerze wystarczyło.

Ogólne info na temat voluminów LVM można uzyskać też przez `dmsetup` :

    # dmsetup info --columns
    Name             Maj Min Stat Open Targ Event  UUID
    grupa1-volumin5  254   5 L--w    1    1      0 LVM-Yas25yA36PmWfMQNJ17UWf48Uat1Wd4MAkxiRDJ1HSjRhwAdfvQsvfaSKD3dqJ3J
    grupa1-volumin4  254   4 L--w    1    1      0 LVM-Yas25yA36PmWfMQNJ17UWf48Uat1Wd4MVLPNyLV6edfQ5F1Whn2cekmQzuHEiU2a
    grupa1-volumin3  254   3 L--w    1    1      0 LVM-Yas25yA36PmWfMQNJ17UWf48Uat1Wd4MXOe2Txojibc6XiGCLk5RPKxjEOJ9gRxk
    grupa1-volumin2  254   2 L--w    1    1      0 LVM-Yas25yA36PmWfMQNJ17UWf48Uat1Wd4MTUAwlgfkbmodBL5MSAb9Xt2yoNpNj9o8
    grupa1-volumin1  254   1 L--w    1    1      0 LVM-Yas25yA36PmWfMQNJ17UWf48Uat1Wd4MYaU6fBv6UC4XBJPOmeCJ1lrlqk0FA5cU

Atrybuty: (L)ive, (I)nactive, (s)uspended, (r)ead-only, read-(w)rite

Jeśli potrzebujemy bardziej szczegółowych informacji na temat któregoś z voluminów, to możemy
posłużyć się `lvdisplay` , przykładowo:

    # lvdisplay /dev/mapper/grupa1-volumin1
      --- Logical volume ---
      LV Path                /dev/grupa1/volumin1
      LV Name                volumin1
      VG Name                grupa1
      LV UUID                YaU6fB-v6UC-4XBJ-POme-CJ1l-rlqk-0FA5cU
      LV Write Access        read/write
      LV Creation host, time livemor, 2014-01-15 09:46:44 +0100
      LV Status              available
      # open                 1
      LV Size                5,00 GiB
      Current LE             1280
      Segments               1
      Allocation             contiguous
      Read ahead sectors     auto
      - currently set to     256
      Block device           254:1

## Powiększanie LVM

Spróbujmy teraz rozszerzyć partycję LVM i 2 voluminy w niej zawarte. Zgodnie z informacjami, które
uzyskaliśmy w `pvs` , rozmiar fizycznego voluminu (pv) to 81911808 sektorów. Za to rozmiar
urządzenia, na którym znajduje się LVM to 81920000 sektorów. Jak widać te dwie pozycje różnią się o
8192 sektory (4 MiB). W fdisk'u rozmiar wyjściowy, to rozmiar partycji LVM, czyli 81920000. Do tej
wartości trzeba dodać 10 GiB, co daje nam (10\*1024\*1024\*1024)/512=20971520 sektorów. Rozmiar
nowej partycji wynosić będzie zatem 81920000+20971520=102891520 sektorów (pamiętajmy by w `fdisk`
odjąć 1). Wchodzimy do fdisk'a i dodajemy +10 GiB miejsca:

    # fdisk /dev/sdb
    
    Command (m for help): p
    
    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    81922047    40960000   83  Linux
    
    Command (m for help): d
    Selected partition 1
    
    Command (m for help): n
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p):
    Using default response p
    Partition number (1-4, default 1):
    Using default value 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-156299374, default 156299374): +102891519
    
    Command (m for help): t
    Selected partition 1
    Hex code (type L to list codes): 8e
    Changed system type of partition 1 to 8e (Linux LVM)
    
    Command (m for help): p
    
    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    255 heads, 63 sectors/track, 9729 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048   102893567    51445760   8e  Linux LVM
    
    Command (m for help): w
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.

Sprawdzamy czy rozmiar partycji pod LVM się zmienił:

    # lsblk /dev/sdb
    NAME                       MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb                          8:16   0  74,5G  0 disk
    └─sdb1                       8:17   0  49,1G  0 part
      ├─grupa1-volumin1 (dm-1) 254:1    0     5G  0 lvm  /media/1
      ├─grupa1-volumin2 (dm-2) 254:2    0     7G  0 lvm  /media/2
      ├─grupa1-volumin3 (dm-3) 254:3    0     9G  0 lvm  /media/3
      ├─grupa1-volumin4 (dm-4) 254:4    0    11G  0 lvm  /media/4
      └─grupa1-volumin5 (dm-5) 254:5    0   7,1G  0 lvm  /media/5

Jest +10 GiB. Teraz trzeba jeszcze rozszerzyć kontener LVM:

    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [39,06 GiB / 0    free]
      Total: 1 [39,06 GiB] / in use: 1 [39,06 GiB] / in no VG: 0 [0   ]
    
    # pvresize /dev/sdb1
      Physical volume "/dev/sdb1" changed
      1 physical volume(s) resized / 0 physical volume(s) not resized
    
    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [49,06 GiB / 10,00 GiB free]
      Total: 1 [49,06 GiB] / in use: 1 [49,06 GiB] / in no VG: 0 [0   ]

Wolne miejsce wskazuje, że 10 GiB dodanych wewnątrz LVM zostało uwzględnione. Rozszerzmy zatem 2
dyski logiczne. Powiedzmy, że będą to 2 i 5. Dodajmy im po 5 GiB. Z piątym, jako, że jest ostatni,
nie będzie problemu:

    # lvresize -L +5G /dev/mapper/grupa1-volumin5
      Extending logical volume volumin5 to 12,06 GiB
      Logical volume volumin5 successfully resized

Ale gdy spróbujemy dodać +5 GiB do drugiego logicznego dysku, dostaniemy poniższy komunikat:

    # lvresize -L +5G /dev/mapper/grupa1-volumin2
      Extending logical volume volumin2 to 12,00 GiB
      Insufficient suitable contiguous allocatable extents for logical volume volumin2: 1280 more required

Co w takim przypadku zrobić? Wyjścia są dwa. Możemy usunąć volumin i stworzyć nowy, tym razem bez
opcji `-C y` , Możemy także zmienić strukturę tego voluminu, tak by przestała być ciągła. Jeśli nie
uśmiecha nam się kasowanie danych, możemy skorzystać z drugiego rozwiązania:

    # lvchange -C n /dev/mapper/grupa1-volumin2
      Logical volume "volumin2" changed

I sprawdzamy czy volumin stracił ciągłość:

    # lvscan
      ACTIVE            '/dev/grupa1/volumin1' [5,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin2' [7,00 GiB] inherit
      ACTIVE            '/dev/grupa1/volumin3' [9,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin4' [11,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin5' [12,06 GiB] contiguous

Rozszerzamy go o teraz o 5 GiB:

    # lvresize -L +5G /dev/mapper/grupa1-volumin2
      Extending logical volume volumin2 to 12,00 GiB
      Logical volume volumin2 successfully resized

Rozmiar został zmieniony. Sprawdźmy jeszcze jak wygląda struktura LVM:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0      0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0   1280  1792 volumin2     0 linear /dev/sdb1:1280-3071
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0   3072  2304 volumin3     0 linear /dev/sdb1:3072-5375
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0   5376  2816 volumin4     0 linear /dev/sdb1:5376-8191
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0   8192  3087 volumin5     0 linear /dev/sdb1:8192-11278
      /dev/sdb1  grupa1 lvm2 a--  49,06g    0  11279  1280 volumin2  1792 linear /dev/sdb1:11279-12558

Wszystko wygląda dobrze. Taka uwaga. Wszystkie powyższe operacje były przeprowadzane przy
podmontowanych voluminach. Do tej pory nie było potrzeby, by te voluminy były odmontowane. Niemniej
jednak, teraz trzeba rozciągnąć ich system plików, tak by wypełniał całą przestrzeń dostępną na tych
logicznych dyskach. Dlatego też musimy odmontować voluminy, których system plików będzie zmieniany.
W tym przypadku wszystkie voluminy mają system plików EXT4. Przed zmianą rozmiaru trzeba
przeskanować voluminy przy pomocy `fsck` . Rozmiar zostanie zmieniony w przypadku dwóch, także
skanujemy tylko te dwa:

    # umount /media/2
    # umount /media/5
    # e2fsck -f /dev/mapper/grupa1-volumin2
    # e2fsck -f /dev/mapper/grupa1-volumin5

I rozszerzamy ich system plików przy pomocy `resize2fs` . W przypadku niepodania argumentu rozmiaru,
system plików zostanie rozciągnięty na całą dostępną przestrzeń w voluminie:

    # resize2fs -p /dev/mapper/grupa1-volumin2
    resize2fs 1.42.8 (20-Jun-2013)
    Resizing the filesystem on /dev/mapper/grupa1-volumin2 to 3145728 (4k) blocks.
    The filesystem on /dev/mapper/grupa1-volumin2 is now 3145728 blocks long.
    
    # resize2fs -p /dev/mapper/grupa1-volumin5
    resize2fs 1.42.8 (20-Jun-2013)
    Resizing the filesystem on /dev/mapper/grupa1-volumin5 to 3161088 (4k) blocks.
    The filesystem on /dev/mapper/grupa1-volumin5 is now 3161088 blocks long.

By sprawdzić czy wszystko działa jak należy, montujemy dyski logiczne:

    # mount /dev/mapper/grupa1-volumin2 /media/2
    # mount /dev/mapper/grupa1-volumin5 /media/5

Nie ma żadnych problemów, co oznacza, że dyski logiczne, jak i sam kontener LVM, zostały z
powodzeniem rozciągnięte.

## Zmniejszanie LVM

Wiemy zatem jak przebrnąć przez proces zwiększania kontenera LVM. Przydałoby się jeszcze odwrócić
cały schemat i zmniejszyć przykładowy kontener. Statystyki tego voluminu wyglądają mniej więcej
tak:

    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [49,06 GiB / 0    free]
      Total: 1 [49,06 GiB] / in use: 1 [49,06 GiB] / in no VG: 0 [0   ]
    
    # lvscan
      ACTIVE            '/dev/grupa1/volumin1' [5,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin2' [12,00 GiB] inherit
      ACTIVE            '/dev/grupa1/volumin3' [9,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin4' [11,00 GiB] contiguous
      ACTIVE            '/dev/grupa1/volumin5' [12,06 GiB] contiguous

Zmniejszmy zatem partycję pod LVM o 30 GiB. Załóżmy, że mamy dwa zbędne voluminy (numery 2 i 4). Da
nam to tylko 23 GiB. Dodatkowo, volumin3 ma sporo wolnego miejsca i nadaje się do skurczenia.
Odmontujmy wszystkie voluminy oraz usuńmy volumin2 oraz volumin4:

    # umount /media/1
    # umount /media/2
    # umount /media/3
    # umount /media/4
    # umount /media/5
    
    # lvremove /dev/mapper/grupa1-volumin2
    Do you really want to remove active logical volume volumin2? [y/n]: y
      Logical volume "volumin2" successfully removed
    
    # lvremove /dev/mapper/grupa1-volumin4
    Do you really want to remove active logical volume volumin4? [y/n]: y
      Logical volume "volumin4" successfully removed

Zmieniamy także rozmiar logicznego dysku numer 3. Musimy pierw skurczyć system plików przy pomocy
`resize2fs` :

    # e2fsck -f /dev/mapper/grupa1-volumin3
    # resize2fs -p /dev/mapper/grupa1-volumin3 2G
    resize2fs 1.42.8 (20-Jun-2013)
    Resizing the filesystem on /dev/mapper/grupa1-volumin3 to 524288 (4k) block.
    Begin pass 3 (max = 24)
    Scanning inode table XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/mapper/grupa1-volumin3 is now 524288 blocks long.

Zmieniamy teraz rozmiar voluminu, tak by pasował do rozmiaru systemu plików. Ci którzy nie ufają
rozmiarom definiowanym w bajtach (K,M,G), mogą oczywiście przeliczyć rozmiar na sektory. W tym celu
bierzemy bloki wypisane przez `resize2fs` i mnożymy je przez 8, co daje 524288\*8=4194304 sektorów
512-bajtowych. Jeśli chcemy definiować rozmiar po sektorach, trzeba odjąć 1 od tej wartości:
4194304-1=4194303.

    # lvresize -t -L 4194303s /dev/mapper/grupa1-volumin3
    TEST MODE: Metadata will NOT be updated and volumes will not be (de)activated.
    Rounding size to boundary between physical extents: 2,00 GiB
    WARNING: Reducing active and open logical volume to 2,00 GiB

Jeśli nie chce nam się bawić w przeliczanie, możemy po prostu ustawić 2G:

    # lvresize -t -L 2G /dev/mapper/grupa1-volumin3
    TEST MODE: Metadata will NOT be updated and volumes will not be (de)activated.
    WARNING: Reducing active and open logical volume to 2,00 GiB
    THIS MAY DESTROY YOUR DATA (filesystem etc.)

Jak widać na powyższym teście, w obu przypadkach nowy rozmiar to 2 GiB. Wybieramy jeden z powyższych
i zmieniamy wielkość voluminu na 2 GiB:

    # lvresize -L 2G /dev/mapper/grupa1-volumin3
      WARNING: Reducing active and open logical volume to 2,00 GiB
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce volumin3? [y/n]: y
      Reducing logical volume volumin3 to 2,00 GiB
      Logical volume volumin3 successfully resized

Sprawdzamy czy udało się odzyskać 30 GiB:

    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [49,06 GiB / 30,00 GiB free]
      Total: 1 [49,06 GiB] / in use: 1 [49,06 GiB] / in no VG: 0 [0   ]

Mamy 30 GiB wolnego miejsca, więc wszystko zgodnie z planem. Sprawdźmy jeszcze czy pozostałe 3
voluminy montują się bez problemu:

    # mount /dev/mapper/grupa1-volumin1 /media/1
    # mount /dev/mapper/grupa1-volumin3 /media/3
    # mount /dev/mapper/grupa1-volumin5 /media/5
    
    # lsblk /dev/sdb
    NAME                       MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb                          8:16   0  74,5G  0 disk
    └─sdb1                       8:17   0  49,1G  0 part
      ├─grupa1-volumin1 (dm-1) 254:1    0     5G  0 lvm  /media/1
      ├─grupa1-volumin3 (dm-2) 254:2    0     2G  0 lvm  /media/3
      └─grupa1-volumin5 (dm-3) 254:3    0  12,1G  0 lvm  /media/5

Nie ma żadnych błędów, co oznacza, że wszystko jest w porządku. Ostatni krok to zmniejszenie
partycji pod LVM ale tu już nie będzie tak łatwo. Jeśli spojrzymy na to jak wygląda obecnie
struktura kontenera LVM, zobaczymy dziury:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree  Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g     0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  1280  1792              0 free
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  3072   512 volumin3     0 linear /dev/sdb1:3072-3583
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  3584  4608              0 free
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  8192  3087 volumin5     0 linear /dev/sdb1:8192-11278
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g 11279  1280              0 free

Jeśli próbowalibyśmy zmniejszyć rozmiar partycji o 30 GiB, dostalibyśmy poniższy komunikat:

    # pvresize --setphysicalvolumesize 29,06G /dev/sdb1
      /dev/sdb1: cannot resize to 7439 extents as later ones are allocated.
      0 physical volume(s) resized / 1 physical volume(s) not resized

Dzieje się tak dlatego, że za ostatnim voluminem jest dostępnych 1280\*4MiB=5 GiB wolnego miejsca. W
tej chwili możemy zmniejszyć kontener LVM, jak i samą partycję, jedynie o 5 GiB z pożądanych 30 GiB.
Jeśli policzymy sobie pozostałe przestrzenie, tj. 4608\*4MiB=18 GiB oraz 1792\*4MiB=7 GiB, to da nam
to 5+18+7=30 GiB. Czemu mnożymy przez 4 MiB? To jest blok LVM, można go odczytać z:

    # vgdisplay
      --- Volume group ---
      VG Name               grupa1
      System ID
      Format                lvm2
      Metadata Areas        1
      Metadata Sequence No  13
      VG Access             read/write
      VG Status             resizable
      MAX LV                0
      Cur LV                3
      Open LV               3
      Max PV                0
      Cur PV                1
      Act PV                1
      VG Size               49,06 GiB
      PE Size               4,00 MiB
      Total PE              12559
      Alloc PE / Size       4879 / 19,06 GiB
      Free  PE / Size       7680 / 30,00 GiB
      VG UUID               Yas25y-A36P-mWfM-QNJ1-7UWf-48Ua-t1Wd4M

Co zrobić by usunąć wolne miejsce z kontenera? Trzeba poprzenosić logiczne voluminy i upakować je
blisko siebie ale trzeba zwracać uwagę na wolne miejsce, bo nie da rady wpakować większego voluminu
do mniejszej dziury. Podobnie jest z przesuwaniem voluminu. Jeśli dziura przed voluminem ma 2 GiB, a
sam volumin 3 GiB, to nie damy rady przesunąć voluminu. Trzeba będzie go skopiować gdzieś, by
uwolnić pierw miejsce, tak by dziura miała minimum 3 GiB.

W tym przypadku akurat sprawa wygląda prosto: volumin3 ma rozmiar 512 bloków i się zmieści w dziurę
przed nim, a potem tylko dokleimy volumin5 za voluminem3. Zatem do dzieła. Każdy logiczny volumin
jest reprezentowany przez dwie wartości: początek dysku i jego rozmiar. Rozpatrując przeniesienie
voluminu3, jego rozmiar to 512 bloków i musi zostać przeniesiony na pozycję 1280. Zatem nowe
parametry będą wynosić 1280-1791. Trzeba pamiętać o odjęciu 1 od końcowego bloku. Przenosimy volumin
na tym samym fizycznym nośniku, zatem polecenie będzie miało poniższą postać:

    # pvmove --alloc anywhere -i 30 /dev/sdb1:3072-3583  /dev/sdb1:1280-1791
      /dev/sdb1: Moved: 0,0%
      /dev/sdb1: Moved: 15,0%
      /dev/sdb1: Moved: 30,3%

Zobaczmy co się dzieje wewnątrz kontenera LVM podczas przenoszenia voluminów:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree  Start SSize LV        Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g     0  1280 volumin1      0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g  1280   512 [pvmove0]     0 mirror /dev/sdb1:3072-3583 /dev/sdb1:1280-1791
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g  1792  1280               0 free
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g  3072   512 [pvmove0]     0 mirror /dev/sdb1:3072-3583 /dev/sdb1:1280-1791
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g  3584  4608               0 free
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g  8192  3087 volumin5      0 linear /dev/sdb1:8192-11278
      /dev/sdb1  grupa1 lvm2 a--  49,06g 28,00g 11279  1280               0 free

Mamy tam nowy volumin: `pvmove0` . Jest on tworzony tylko na czas przenoszenia danych. Po pomyślnym
zakończeniu, stary volumin zostanie usunięty. Gdy proces dobiegnie końca, przenosimy volumin5.
Sprawdzamy pierw jak prezentuje się układ LVM:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree  Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g     0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  1280   512 volumin3     0 linear /dev/sdb1:1280-1791
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  1792  6400              0 free
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  8192  3087 volumin5     0 linear /dev/sdb1:8192-11278
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g 11279  1280              0 free 

Volumin przenosimy przy pomocy tego poniższego polecenia:

    # pvmove --alloc anywhere -i 30 /dev/sdb1:8129-11278  /dev/sdb1:1792-4878

Jeśli się pomyliliśmy lub z jakiegoś powodu chcemy przerwać proces przenoszenia voluminu, to nic nam
nie da wciśnięcie Ctrl-C . Proces dalej będzie działał. Trzeba w konsoli wpisać `pvmove --abort` .
Wtedy proces przenoszenia voluminu zostanie przerwany. Żadne zmiany nie zostaną poczynione w
stosunku do starego voluminu.

Po skończonej pracy, powinniśmy uzyskać wynik podobny do tego poniżej:

    # pvs -v --segments
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize  PFree  Start SSize LV       Start Type   PE Ranges
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g     0  1280 volumin1     0 linear /dev/sdb1:0-1279
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  1280   512 volumin3     0 linear /dev/sdb1:1280-1791
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  1792  3087 volumin5     0 linear /dev/sdb1:1792-4878
      /dev/sdb1  grupa1 lvm2 a--  49,06g 30,00g  4879  7680              0 free
    
    # lvs -v --segments
        Finding all logical volumes
      LV       VG     Attr      Start SSize  #Str Type   Stripe Chunk
      volumin1 grupa1 -wc-ao---    0   5,00g    1 linear     0     0
      volumin3 grupa1 -wc-ao---    0   2,00g    1 linear     0     0
      volumin5 grupa1 -wc-ao---    0  12,06g    1 linear     0     0

Czas zmniejszyć kontener LVM. Jeśli spróbujemy skurczyć kontener za bardzo, dostaniemy błąd:

    # pvresize -v /dev/sdb1 --setphysicalvolumesize 19G
        Using physical volume(s) on command line
        Archiving volume group "grupa1" metadata (seqno 21).
        /dev/sdb1: Pretending size is 39845888 not 102884229 sectors.
        Resizing volume "/dev/sdb1" to 102884229 sectors.
        Resizing physical volume /dev/sdb1 from 0 to 4863 extents.
      /dev/sdb1: cannot resize to 4863 extents as 4879 are allocated.
      0 physical volume(s) resized / 1 physical volume(s) not resized

Powyżej mamy wypisaną wartość jakiej powinniśmy użyć: 4879. Jest to (4879\*4MiB)+1=19517M

    # pvresize -v /dev/sdb1 --setphysicalvolumesize 19517M
        Using physical volume(s) on command line
        Archiving volume group "grupa1" metadata (seqno 21).
        /dev/sdb1: Pretending size is 39970816 not 102884229 sectors.
        Resizing volume "/dev/sdb1" to 102884229 sectors.
        Resizing physical volume /dev/sdb1 from 0 to 4879 extents.
        Updating physical volume "/dev/sdb1"
        Creating volume group backup "/etc/lvm/backup/grupa1" (seqno 22).
      Physical volume "/dev/sdb1" changed
      1 physical volume(s) resized / 0 physical volume(s) not resized

Sprawdzamy czy rzeczywiście rozmiar kontenera uległ zmianie:

    # pvscan
      PV /dev/sdb1   VG grupa1   lvm2 [19,06 GiB / 0    free]
      Total: 1 [19,06 GiB] / in use: 1 [19,06 GiB] / in no VG: 0 [0   ]

Rozmiar 19,06 GiB i 0 wolnego miejsca. Ostatni krok to zmniejszenie partycji via `fdisk` . Jakiego
rozmiaru potrzebujemy? Sprawdzamy rozmiar kontenera w sektorach:

    # pvs -v --unit s
        Scanning for physical volume names
      PV         VG     Fmt  Attr PSize     PFree DevSize    PV UUID
      /dev/sdb1  grupa1 lvm2 a--  39968768S    0S 102891520S xMddCU-5h1X-ZZL4-GFfO-ea4L-PqW8-YCGLte

Rozmiar partycji powinien być większy o 8192 sektory w stosunku do wielkości kontenera LVM. W tym
przypadku kontener ma 39968768 sektorów. Zatem rozmiar partycji wyniesie: 39968768+8192=39976960
sektorów (pamiętajmy, by odjąć 1 w `fdisk`). Nie należy brać pod uwagę wpisu w `DevSize` , bo on
ciągle ma starą wartość.

    # fdisk /dev/sdb
    
    Command (m for help): d
    Selected partition 1
    
    Command (m for help): n
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p):
    Using default response p
    Partition number (1-4, default 1):
    Using default value 1
    First sector (2048-156299374, default 2048):
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-156299374, default 156299374): +39976959
    
    Command (m for help): t
    Selected partition 1
    Hex code (type L to list codes): 8e
    Changed system type of partition 1 to 8e (Linux LVM)
    
    Command (m for help): p
    
    Disk /dev/sdb: 80.0 GB, 80025280000 bytes
    100 heads, 3 sectors/track, 520997 cylinders, total 156299375 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000c741d
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    39979007    19988480   8e  Linux LVM
    
    Command (m for help): w
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    Syncing disks.

Być może też czasem dostaniemy poniższy komunikat:

    WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)

Może się on pojawić nawet przy odmontowanych voluminach. Jest to zasługą aktywnej grupy. Można ją
albo deaktywować ( `vgchane -a n` ) na czas zmian tablicy partycji, albo zaktualizować wpisy przy
pomocy `partprobe` . Nie ma potrzeby resetowania systemu, by nowe wartości zaczęły obowiązywać.

Sprawdzamy, czy wszystko się montuje bez problemu:

    # vgchange -a n grupa1
      0 logical volume(s) in volume group "grupa1" now active
    # lsblk /dev/sdb
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb      8:16   0  74,5G  0 disk
    └─sdb1   8:17   0  19,1G  0 part
    
    # vgchange -a y grupa1
      3 logical volume(s) in volume group "grupa1" now active
    # lsblk /dev/sdb
    NAME                       MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb                          8:16   0  74,5G  0 disk
    └─sdb1                       8:17   0  19,1G  0 part
      ├─grupa1-volumin1 (dm-1) 254:1    0     5G  0 lvm
      ├─grupa1-volumin3 (dm-2) 254:2    0     2G  0 lvm
      └─grupa1-volumin5 (dm-3) 254:3    0  12,1G  0 lvm
    
    # mount /dev/mapper/grupa1-volumin1 /media/1
    # mount /dev/mapper/grupa1-volumin3 /media/3
    # mount /dev/mapper/grupa1-volumin5 /media/5

Wychodzi na to, że wszystko gra.
