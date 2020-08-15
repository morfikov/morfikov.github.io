---
author: Morfik
categories:
- Linux
- Android
date:    2020-08-15 02:43:00 +0200
lastmod: 2020-08-15 02:43:00 +0200
published: true
status: publish
tags:
- debian
- szyfrowanie
- luks
- smartfon
title: Jak odszyfrować linux'a przy pomocy telefonu z Androidem
---

Zaszyfrowane systemy (desktopy/laptopy) mają jeden poważny problem, gdy chodzi o zapewnianie
bezpieczeństwa chronionym plikom przechowywanym na dyskach twardych. Gdy siedzimy obok naszej
maszyny, możemy czuć się bezpiecznie, bo przecież nikt nie może się włamać do jej systemu bez
naszej wiedzy. Nawet jeśli ktoś będzie próbował się dostać do naszego PC, to istnieje spora szansa,
że takie działanie zostałoby natychmiast przez nas wykryte, przez co moglibyśmy w odpowiedni sposób
zareagować na zaistniałe zagrożenie. Co jednak w przypadku, gdy zostawiamy przykładowo naszego
laptopa samego? Nawet jeśli zablokujemy mu ekran, wyłączymy go albo zahibernujemy, to ta maszyna
wciąż nie jest odpowiednio zabezpieczona, by uniemożliwić osobom postronnym dostęp do naszych
wrażliwych danych. Problem leży w fizycznym dostępie do sprzętu, który ludzie mogą uzyskać, gdy nas
nie ma w pobliżu naszego komputera. W taki sposób osoby trzecie mogą wykorzystać fakt, że tracimy
maszynę z oczu i być w stanie zastawić na nas różne pułapki. By uniknąć zagrożenia związanego z
zostawieniem laptopa/desktopa bez nadzoru, nie możemy w zasadzie pozostawiać tego urządzenia samego,
co jest zadaniem praktycznie nie do wykonania. Komputery stacjonarne czy nawet laptopy nie są
urządzeniami o małych gabarytach i zwykle nie możemy ich wszędzie zabrać ze sobą, w przeciwieństwie
do smartfonów. Postanowiłem zatem tak skonfigurować swojego linux'a, by jego zaszyfrowany dysk
(LUKS + LVM) można było odszyfrować jedynie przy pomocy mojego telefonu z Androidem, z którym w
zasadzie się nie rozstaję.

<!--more-->
## USB Mountr i obraz partycji /boot/

W zasadzie, to każdy z nas obecnie wykorzystuje smartfon do całej masy codziennych czynności. Jeśli
ten telefon ma na pokładzie Androida, to jest też spora szansa, że to urządzenie może wspierać
niestandardowe ROM'y na bazie [AOSP][2], np.[LineageOS][3]. W taki sposób możemy mieć naprawdę
przyjazne pod kątem prywatności i bezpieczeństwa urządzenie, które może zostać użyte jako swojego
rodzaju token przy dostępie do zaszyfrowanego komputera. Wszystko dzięki komponentowi USB MSD,
który jest obecny w kernelu Androida. Sprawia on, że możliwe jest, np. uruchomienie obrazu systemu
live bezpośrednio z telefonu po podłączeniu go do portu USB (przy wykorzystaniu aplikacji
[USB Mountr][4]). Nie jest to jednak jedyne zastosowanie tej aplikacji, które może nam się do
czegoś przydać. Możemy też przykładowo stworzyć boot'owalny obraz partycji `/boot/` naszego linux'a
i umieścić w nim nagłówki zaszyfrowanych kontenerów LUKS. W ten sposób możemy uniknąć ataków na MBR,
ataków na bootloader oraz ataków na initramfs/initrd, czyniąc tym samym praktycznie niemożliwym
dostęp do naszego desktopa/laptopa bez przejęcia jednocześnie kontroli nad naszym smartfonem. Co
więcej, po przeniesieniu nagłówków LUKS do obrazu partycji `/boot/` , klucz szyfrujący dane na
dysku twardym komputera zostanie z niego usunięty, przez co nikt nie będzie w stanie
wydobyć/odszyfrować tego klucza nawet w przypadku, gdyby ten ktoś zdołał przechwycić w jakiś sposób
hasło wpisywane podczas startu maszyny.

Spróbujmy zatem utworzyć obraz partycji `/boot/` , który będzie zawierał bootloader extlinux (może
być także GRUB/GRUB2), konfigurację dla tego bootloader'a, obraz initramfs/initrd oraz nagłówki
LUKS. Stworzony przez nas obraz będzie także zawierał MBR i tablicę partycji MS-DOS. Zatem do
dzieła.

## Tworzenie obrazu partycji /boot/

Zaczynamy od stworzenia obrazu na pliki partycji `/boot/` . To zadanie możemy zrealizować na dwa
sposoby. Pierwszym rozwiązaniem jest kopia całej partycji `/boot/` do pliku (przy pomocy `dd` ),
Natomiast drugi ze sposobów zakłada stworzenie nowego obrazu (via `dd` ), utworzenie w tym obrazie
systemu plików (via `mkfs` ) i skopiowanie do niego zawartości (plików) partycji `/boot/` . Oba te
rozwiązania mają swoje wady i zalety ale w tym artykule skorzystamy z pierwszego z nich:

    # mkdir /media/boot/
    # cd /media/boot/
    # dd if=/dev/sda1 bs=1M | pv -s 2G > ./boot-2g.img

Następnie konfigurujemy urządzenie `loop` pod nasz obraz:

    # losetup /dev/loop4 boot-2g.img
    # losetup -a
    /dev/loop4: [65031]:6815746 (/media/boot/boot-2g.img)
    # mount -o ro /dev/loop4 /mnt
    # df -h /mnt
    Filesystem     Type  Size  Used Avail Use% Mounted on
    /dev/loop4     ext4  2.0G   51M  1.8G   3% /mnt

W tym przypadku, partycja `/boot/` waży 2 GiB i zwykle przeznaczanie na nią tyle miejsca jest
marnowaniem przestrzeni. Jak możemy zauważyć wyżej, pliki w obrębie tego systemu plików
wykorzystują 51 MiB. Możemy skurczyć rozmiar obrazu, np. do 256 MiB. Jeśli chcemy to zrobić, to
wcześniej trzeba skurczyć system plików obrazu.

### Zmiana rozmiaru obrazu partycji /boot/

Do zmiany rozmiaru systemu plików w obrazie partycji `/boot/` będziemy korzystać z narzędzia
`resize2fs` . Robimy to w poniższy sposób:

    # umount /mnt
    # resize2fs -p /dev/loop4 256M
    resize2fs 1.44.0 (7-Mar-2018)
    Resizing the filesystem on /dev/loop4 to 65536 (4k) blocks.
    Begin pass 2 (max = 16475)
    Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 3 (max = 16)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/loop4 is now 65536 (4k) blocks long.

System plików obrazu ma obecnie 65536 bloków, czyli 65536*4096=268435456 bajtów albo 256 MiB.
Pamiętajmy jednak, że nie tylko regularne pliki znajdują się w tym obrazie. Jest też w nim zawarta
struktura systemu plików EXT4, tj. metadane opisujące pliki, w tym też dziennik (ext4 journal),
który w tym przypadku ma 32 MiB:

    # dumpe2fs /dev/loop4 | grep "Journal size"
    Journal size:             32M

Dlatego właśnie ustawiliśmy rozmiar obrazu na 256 MiB, zostawiając tym samym trochę wolnego miejsca
na przyszłe wykorzystanie.

Po zmniejszeniu systemu plików, trzeba obciąć resztę obrazu, bo nie jest nam ona już do niczego
potrzebna:

    # losetup -d /dev/loop4
    # dd if=./boot-2g.img of=./boot-256m bs=4K count=65536
    65536+0 records in
    65536+0 records out
    268435456 bytes (268 MB, 256 MiB) copied, 2.25373 s, 119 MB/s

Możemy przetestować czy system plików w obrazie jest w dobrym stanie i nie został w żaden sposób
uszkodzony. Trzeba go tylko zamontować:

    # losetup /dev/loop4 boot-256m
    # mount -o ro /dev/loop4 /mnt
    # df -h /mnt
    Filesystem     Type  Size  Used Avail Use% Mounted on
    /dev/loop4     ext4  188M   49M  122M  29% /mnt

Jeśli nie ma żadnych błędów, to znaczy, że wszystko jest w porządku. Odmontujmy teraz ten obraz:

    # umount /mnt
    # losetup -d /dev/loop4

Gdy sprawdzimy obraz w `parted` , to zobaczymy, że na początku tego obrazu nie ma ani jednego
bitu wolnego miejsca:

    # parted boot-256m
    GNU Parted 3.2
    Using /media/boot/boot-256m
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) unit s
    (parted) print free
    Model:  (file)
    Disk /media/boot/boot-256m: 524288s
    Sector size (logical/physical): 512B/512B
    Partition Table: loop
    Disk Flags:

    Number  Start  End      Size     File system  Flags
    1      0s     524287s  524288s  ext4

Bez wolnego miejsca na początku, nie da rady utworzyć MBR ani tablicy partycji. Trzeba zatem dodać
trochę wolnego miejsca na początku obrazu.

### Tworzenie MBR i tablicy partycji w obrazie

Kolejnym krokiem jest stworzenie MBR oraz tablicy partycji wewnątrz obrazu. W tym przypadku będzie
wykorzystywana tablica partycji MS-DOS. Zanim jednak stworzymy tablicę partycji, trzeba utworzyć
kolejny obraz przy pomocy `dd` :

    # dd if=/dev/zero of=./free-space.img bs=512 count=2048
    2048+0 records in
    2048+0 records out
    1048576 bytes (1.0 MB, 1.0 MiB) copied, 0.0180124 s, 58.2 MB/s

Ten obraz ma rozmiar jedynie 1 MiB ale powinien nam w zupełności wystarczyć. Chodzi o to, że
wszystkie narzędzia partycjonujące dyski twarde równają partycje do 1 MiB przy tworzeniu pierwszej
partycji.

Łączymy teraz te dwa obrazy w jeden:

    # pv boot-256m >> free-space.img
    256MiB 0:00:02 [ 116MiB/s] [===========================>] 100%
    # mv free-space.img new-boot.img
    # ls -al new-boot.img
    -rw-r--r-- 1 root root 257M 2018-03-15 17:57:37 new-boot.img

Tworzymy teraz tablicę partycji w obrazie przy wykorzystaniu narzędzia `fdisk` :

    # fdisk new-boot.img

    Welcome to fdisk (util-linux 2.31.1).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0x105e4249.

    Command (m for help): n
    Partition type
     p   primary (0 primary, 0 extended, 4 free)
     e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-526335, default 2048): 2048
    Last sector, +sectors or +size{K,M,G,T,P} (2048-526335, default 526335):

    Created a new partition 1 of type 'Linux' and of size 256 MiB.
    Partition #1 contains a ext4 signature.

    Do you want to remove the signature? [Y]es/[N]o: n

Ustawiamy także flagę `boot` :

    Command (m for help): a
    Selected partition 1
    The bootable flag on partition 1 is enabled now.

Sprawdzamy teraz czy wszystko jest w porządku:

    Command (m for help): p
    Disk new-boot.img: 257 MiB, 269484032 bytes, 526336 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x105e4249

    Device        Boot Start    End Sectors  Size Id Type
    new-boot.img1 *     2048 526335  524288  256M 83 Linux

I jeśli tak, to zapisujemy tablicę partycji w obrazie:

    Command (m for help): w
    The partition table has been altered.
    Syncing disks.

Gdy teraz będziemy tworzyć urządzenie `loop` dla tego nowego obrazu, powinniśmy zaobserwować pewne
zmiany w stosunku do tworzonych urządzeń w katalogu `/dev/`:

    # losetup /dev/loop4 new-boot.img
    # losetup -a
    /dev/loop4: [65031]:6815748 (/media/boot/new-boot.img)
    # ls -al /dev/loop4*
    brw-rw---- 1 root disk 7, 128 2018-03-15 18:03:33 /dev/loop4
    brw-rw---- 1 root disk 7, 129 2018-03-15 18:03:33 /dev/loop4p1

Mamy dodatkowe urządzenie w postaci `/dev/loop4p1` , a to z tego powodu, że w obrazie jest zawarta
partycja i tak ten obraz jest widziany przez naszego linux'a:

    # file new-boot.img
    new-boot.img: DOS/MBR boot sector; partition 1 :
    ID=0x83,
    start-CHS (0x0,32,33),
    end-CHS (0x20,194,34),
    startsector 2048,
    524288 sectors,
    extended partition table (last)

W taki sposób, partycja `/boot/` jest podpięta pod urządzenie `/dev/loop4p1` , a plik obrazu
pod `/dev/loop4` . Gdyby w obrazie były 4 partycje, to mielibyśmy urządzenia `loop4p1` , `loop4p2` ,
 `loop4p3` i `loop4p4` , każde dla osobnej partycji w obrazie.

Musimy teraz zamontować partycję `/boot/` i jeśli wszystko będzie w porządku, to pliki wciąż
powinny w tym obrazie być widoczne:

    # mount -o ro /dev/loop4p1 /mnt
    # ls -al /mnt
    total 45M
    drwxr-xr-x  4 root root 4.0K 2018-03-14 19:15:28 ./
    drwxr-xr-x 25 root root 4.0K 2018-03-01 18:54:11 ../
    drwxr-xr-x  2 root root 4.0K 2018-02-26 18:00:35 extlinux/
    drwx------  2 root root  16K 2018-02-17 16:34:18 lost+found/
    -rw-r--r--  1 root root 3.0M 2018-02-18 09:36:49 System.map-4.15.0-1-amd64
    -rw-r--r--  1 root root 194K 2018-02-18 09:36:49 config-4.15.0-1-amd64
    -rw-r--r--  1 root root  37M 2018-03-14 19:15:26 initrd.img-4.15.0-1-amd64
    -rw-r--r--  1 root root 179K 2015-06-25 19:16:52 memtest86+.bin
    -rw-r--r--  1 root root 181K 2015-06-25 19:16:52 memtest86+_multiboot.bin
    -rw-r--r--  1 root root 4.7M 2018-02-18 09:36:49 vmlinuz-4.15.0-1-amd64

Jeśli nie ma błędów w syslog'u, to proces przygotowania obrazu zakończył się powodzeniem. Nie jest
to jednak koniec pracy. Musimy jeszcze zainstalować jakiś bootloader w MBR tego obrazu. W tym
przypadku będzie to syslinux/extlinux.

### Instalacja bootloader'a w obrazie

By zainstalować bootloader syslinux/extlinux w obrazie, musimy zrobić dwie rzeczy. Po pierwsze
trzeba zainstalować kod MBR bootloader'a w pierwszym sektorze obrazu:

    # dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/loop4

Po drugie, musimy zainstalować VBR (Volume Boot Record) extlinux'a  na partycji `/boot/` :

    # extlinux --install /mnt/extlinux
    /mnt/extlinux is device /dev/loop4p1
    Warning: unable to obtain device geometry (defaulting to 64 heads, 32 sectors)
            (on hard disks, this is usually harmless.)

Powyższe ostrzeżenie może zostać zignorowane.

## Testowanie obrazu

Te powyżej przeprowadzone kroki powinny uczynić obraz boot'owalnym. Możemy przesłać teraz plik
obrazu na kartę SD czy pamięć smartfona i przetestować go przy pomocy aplikacji USB Mountr na
Androida. W tym przypadku udało się uruchomić system mojego laptopa bez żadnego problemu.

## Usuwanie partycji /boot/ z dysku

Możemy teraz usunąć zawartość katalogu `/boot/` dysku twardego czy też skasować całą partycję
`/boot/` , bo nie jest ona nam już do niczego potrzeba -- kopia tej partycji została przeniesiona
do smartfona. Nie musimy też zmieniać żadnych wpisów w pliku `/etc/fstab` , bo numerek UUID
partycji w obrazie jest dokładnie taki sam jak numerek partycji /boot/ na dysku komputera. Jeśli
jednak chcielibyśmy zachować starą partycję `/boot/` , to musimy się upewnić, że zmieniliśmy UUID
systemu plików w stworzonym obrazie:

    # umount /mnt
    # uuidgen
    6f3b0020-0491-4a12-98ca-c97a7a80f5b7
    # tune2fs /dev/loop4p1 -U 6f3b0020-0491-4a12-98ca-c97a7a80f5b7
    tune2fs 1.44.0 (7-Mar-2018)
    Setting UUID on a checksummed filesystem could take some time.
    Proceed anyway (or wait 5 seconds to proceed) ? (y,N) <proceeding>
    # lsblk /dev/loop4
    NAME       SIZE FSTYPE TYPE LABEL MOUNTPOINT UUID
    loop4      257M        loop
    └─loop4p1  256M ext4   loop boot             6f3b0020-0491-4a12-98ca-c97a7a80f5b7


I teraz jeszcze zmieniamy UUID w `/etc/fstab` , tak by pasował do tego wygenerowanego wyżej:

    # <file system>					<mount point>	<type>	<options>			<dump>  <pass>
    ....
    UUID=6f3b0020-0491-4a12-98ca-c97a7a80f5b7	/boot           ext4    defaults,errors=remount-ro,noauto,nofail,commit=10 0 2

### Flagi noauto i nofail w /etc/fstab

Jeszcze taka informacja odnośnie flag `noauto` oraz `nofail` . Jak możemy wyczytać w manualu
`mount` :

>  nofail -- Do not report errors for this device if it does not exist.
>
>  noauto -- Can only be mounted explicitly (i.e., the -a option will not cause the filesystem to
>  be mounted).

Zatem opcja `noauto` jest dla nas użyteczna, bo nie chcemy by partycja `/boot/` była zamontowana
cały czas w systemie (może być to niebezpieczne dla jej systemu plików). W zasadzie, to
potrzebujemy tej partycji jedynie w momencie startu systemu oraz przy ewentualnej aktualizacji
systemu lub też, gdy majstrujemy przy obrazie initramfs/initrd albo zmieniamy konfigurację
bootloader'a. Poza tymi wyżej wymienionymi rzeczami, partycja `/boot/` niekoniecznie musi być
obecna w systemie, co daje nam możliwość odłączenia i podłączenia smartfona do portu USB komputera,
gdy będziemy tego potrzebować. Niemniej jednak, gdy odłączymy telefon, nasz linux będzie miał z
tym faktem pewne problemy i wypisze w logu poniższe komunikaty:

    systemd[1]: dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.device: Job dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.device/start timed out.
    systemd[1]: Timed out waiting for device dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.device.
    systemd[1]: Dependency failed for /boot.
    systemd[1]: boot.mount: Job boot.mount/start failed with result 'dependency'.
    systemd[1]: Dependency failed for File System Check on /dev/disk/by-uuid/6f3b0020-0491-4a12-98ca-c97a7a80f5b7.
    systemd[1]: systemd-fsck@dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.service: Job systemd-fsck@dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.service/start failed with result 'dependency'.
    systemd[1]: dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.device: Job dev-disk-by\x2duuid-6f3b0020\x2d0491\x2d4a12\x2d98ca\x2dc97a7a80f5b7.device/start failed with result 'timeout'.

Ta powyższa wiadomość jest nieszkodliwa ale cześć usług (takich jak `mount` i `fsck` ) ma
zależności w postaci urządzenia partycji `/boot/` i dlatego te powyższe błędy będą wypisywane w
logu po odłączeniu telefonu. Możemy jednak nakazać linux'owi, by ignorował fakt braku obecności
urządzenia partycji `/boot/` przez dodanie tej drugiej flagi, tj. `nofail` .

## Nagłówki LUKS

Przyszedł czas by zatroszczyć się o nagłówki LUKS. Skopiujemy nagłówki LUKS do obrazu (na partycję
`/boot/` ) i zarazem usuniemy te nagłówki z dysku twardego, zostawiając na tym nośniku jedynie
zaszyfrowane dane. W ten sposób nawet jeśli ktoś by nam zabrał dysk twardy (albo i komputer), to
nie będzie on w stanie zdeszyfrować informacji w nim zawartych, bo na dysku nie będzie klucza
głównego. Bez naszego smartfona, nikt nie będzie w stanie nawet powiedzieć czy dysk jest
zaszyfrowany czy też nie. Zatem jeśli chcemy dołączyć nagłówki LUKS do partycji `/boot/` w
wygenerowanym powyższym sposobem obrazie, możemy to zrobić w poniższy sposób.

Najpierw tworzymy backup nagłówka LUKS:

    # mount /dev/loop4p1 /mnt
    # mkdir /mnt/headers
    # cryptsetup luksHeaderBackup /dev/sda2 --header-backup-file /mnt/headers/head.img
    # ls -al /mnt/headers/head.img
    -r-------- 1 root root 4.0M 2018-03-15 18:58:39 /mnt/headers/head.img

Rozmiar tego nagłówka w tym przypadku ma 4 MiB, dlatego też trzeba będzie nadpisać te 4 MiB na
dysku zaczynając od początku zaszyfrowanej partycji:

    # dd if=/dev/urandom of=/dev/sda2 bs=1M count=4

Następnie aktualizujemy plik `/etc/crypttab` :

    #sda2_crypt	UUID=e017ac1c-c46f-4b3f-a319-e1f5ed15144a  none  luks
    sda2_crypt	UUID=e017ac1c-c46f-4b3f-a319-e1f5ed15144a  none  luks,header=/boot/headers/head.img

### Regeneracja obrazu initramfs/initrd

Ostatnim krokiem jest regeneracja obrazu initramfs/initrd ale by to zrobić, musimy pierw
odmontować starą partycję `/boot/` i podmontować w jej miejsce tę nową:

    # umount /boot
    # mount /boot
    # lsblk | grep boot
    NAME                              SIZE FSTYPE      TYPE  LABEL          MOUNTPOINT     UUID
    ├─sda1                              2G ext4        part  boot                          37a2b033-c47c-4c41-9287-b2f2a44e1bcd
    └─sdc1                            256M ext4        part  boot           /boot          6f3b0020-0491-4a12-98ca-c97a7a80f5b7

### Skrypty dla initramfs-tools

Kolejnym krokiem jest dodanie dwóch skryptów do obrazu initramfs/initrd. Pierwszy skrypt będzie
miał za zadanie zamontować partycję `/boot/` w fazie initramfs/initrd, drugi skrypt zaś odmontuje
tę partycję zanim główny system plików zostanie zamontowany.

Poniżej jest skrypt `/etc/initramfs-tools/scripts/local-top/mount-boot` :


	#!/bin/sh
	PREREQ=""
	prereqs()
	{
		echo "$PREREQ"
	}

	case $1 in
	prereqs)
		prereqs
		exit 0
		;;
	esac

	. /scripts/functions

	export PATH=/sbin:/usr/sbin:/bin:/usr/bin

	[ -d /boot ] || mkdir -m 0755 /boot

	DEVICE=/dev/disk/by-uuid/6f3b0020-0491-4a12-98ca-c97a7a80f5b7
	if [ ! -b "$DEVICE" ]; then
		echo "Waiting for device..." >&2
		until [ -b "$DEVICE" ]; do
			sleep 1
		done
	fi

	mount -t ext4 -o ro "$DEVICE" /boot

	exit 0

Niżej zaś jest skrypt `/etc/initramfs-tools/scripts/local-bottom/umount-boot` :


	#!/bin/sh
	PREREQ=""
	prereqs()
	{
		echo "$PREREQ"
	}

	case $1 in
	prereqs)
		prereqs
		exit 0
		;;
	esac

	. /scripts/functions

	export PATH=/sbin:/usr/sbin:/bin:/usr/bin

	#umount /boot
	mount -o move /boot ${rootmnt}/boot

	exit 0

Te dwa plik muszą mieć być wykonywalne:

    # chmod +x /etc/initramfs-tools/scripts/local-top/mount-boot
    # chmod +x /etc/initramfs-tools/scripts/local-bottom/umount-boot

Teraz regenerujemy obraz initramfs/initrd przy pomocy poniższego polecenia:

    # update-initramfs -u -k all
    update-initramfs: Generating /boot/initrd.img-4.15.0-1-amd64

I to w zasadzie tyle. Możemy jeszcze sprawdzić czy nasz linux uruchamia się w pełni z obrazu, który
mamy w smartfonie -- mój nie ma z tym problemów.

## Jak aktualizować system

Przy aktualizacji systemu musimy pamiętać, by telefon uprzednio podłączyć do portu USB komputera,
tak by partycja `/boot/` została poprawnie zamontowana na czas procesu instalacji pakietów. Zwykle
partycja `/boot/` jest montowana w trybie zapisu, przez co mamy możliwość zmiany jej systemu plików.
Niemniej jednak, w takiej konfiguracji, jaką sobie zaprojektowaliśmy tutaj, nie powinniśmy montować
tej partycji domyślnie z prawami zapisu (w naszym przypadku nie montujemy jej w ogóle). Chodzi
generalnie o to, że możemy uszkodzić jej system plików zapominając go odmontować przed odłączeniem
telefonu od komputera. W ten sposób istnieje ryzyko utraty nagłówków LUKS, a jeśli je stracimy, to
nawet my nie będziemy w stanie uruchomić naszego linux'a, no chyba, że zrobiliśmy backup tych
nagłówków.

Co jednak począć w przypadku aktualizacji systemu? Mało praktycznym jest manualne montowanie
partycji `/boot/` w tryb do odczytu potem do zapisu i znowu do odczytu. W Debianie możemy
skorzystać z pliku `/etc/apt/apt.conf` dodając do niego poniższą konfigurację:

    DPkg
    {
      Pre-Invoke
      {
        "if grep -q boot /proc/mounts; then mount -o remount,rw /boot; else mount -o rw /boot; fi";
      };

      Post-Invoke
      {
        "test ${NO_APT_REMOUNT:-no} = yes || umount /boot || true";
      };
    };

W ten sposób za każdym razem jak będziemy chcieli aktualizować system (instalować pakiety),
`apt-get`/`aptitude` zamontuje partycję `/boot/` za nas automatycznie z prawami do zapisu, a gdy
proces aktualizacji się zakończy, to ta partycja zostanie również w sposób automatyczny odmontowana.

> Ten artykuł został pierwotnie napisany po angielsku i [opublikowany w GitHub Gist][1] ponad 2
> lata temu. Za sprawą pewnego osobnika postanowiłem ten tekst przepisać na język polski, tak by
> osoby niewładające angielskim zbyt dobrze, mogły bez większego problemu opisane tutaj rozwiązanie
> wdrożyć u siebie i w nieco większym stopniu zabezpieczyć swoją maszynę przed nieautoryzowanym
> dostępem.


[1]: https://gist.github.com/morfikov/0bd574817143d0239c5a0e1259613b7d
[2]: https://source.android.com/
[3]: https://www.lineageos.org/
[4]: https://github.com/Streetwalrus/android_usb_msd
