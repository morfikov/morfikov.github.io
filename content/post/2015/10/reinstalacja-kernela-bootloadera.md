---
author: Morfik
categories:
- Linux
date: "2015-10-22T18:41:06Z"
date_gmt: 2015-10-22 16:41:06 +0200
published: true
status: publish
tags:
- bootloader
- szyfrowanie
- kernel
title: Reinstalacja kernela i bootloader'a
---

Wykorzystywanie pełnego szyfrowania dysku twardego ma jedną zasadniczą wadę. O ile nasze dane są
należycie zabezpieczone, o tyle trzeba zwracać uwagę na to komu zezwalamy na dostęp do naszego
komputera. Nie chodzi tutaj o to, kto będzie używał samego systemu operacyjnego, choć to też jest
ważne, ale przede wszystkim chodzi o te osoby, które mają dostęp fizyczny do naszej maszyny. Czasem
możemy nabrać podejrzenia, że ktoś mógł nam jakąś pluskwę podłożyć. Wykrycie takiego robala, np. w
postaci sprzętowego keylogger'a, nie powinno sprawić problemów. Z kolei już manipulacja boot
sektorem dysku twardego, lub też zmiany w initramfs, który znajduje się na niezaszyfrowanej partycji
`/boot/` mogą przejść niezauważone. Jak zatem odratować system, co do którego mamy jakieś
zastrzeżenia?

<!--more-->
## Dostęp do zaszyfrowanego dysku z poziomu systemu live

Przede wszystkim, jeśli już mamy jakieś wątpliwości co do wiarygodności naszego sprzętu i jest on
wyłączony, to nie powinniśmy go uruchamiać, a tym bardziej odblokowywać zaszyfrowanych partycji.
Jest bowiem duże prawdopodobieństwo, że znaki z klawiatury, które zostaną przekazane systemowi,
trafią także w niepowołane ręce. Jeśli po rozkręceniu maszyny nie dostrzegliśmy żadnych zmian w
sprzęcie, możemy przejść do drugiej fazy, tj. tej programowej, gdzie będzie nam potrzebny inny
system operacyjny, najlepiej ten z rodziny live. Chyba wszystkie dystrybucje linux'a mają już swój
zestaw płytek live. [Debian swoje obrazy live trzyma tutaj](https://www.debian.org/CD/live/) i jeśli
nie mamy akurat żadnej pod ręką, to powinniśmy jakąś sobie szybko dociągnąć.

## Boot sektor (MBR)

[Boot sektor](https://pl.wikipedia.org/wiki/Sektor_rozruchowy), to pierwszy sektor dysku twardego, w
którym znajdują się informacje o położeniu kodu programu ładującego system. Ten sektor jest
niezabezpieczony w żaden sposób i jest wielce podatny na zmiany. Został nawet opracowany dedykowany
[atak evilmaid](https://niebezpiecznik.pl/post/implementacja-ataku-evil-maid-falszywy-chkdsk/) pod
kątem tego sektora rozruchowego.

Mając na uwadze powyższe, musimy wyczyścić MBR ale też nie cały. Znajdują się tam też informacje o
strukturze partycji i ich nie można nam ruszyć. Obszar który musimy wyczyścić to jedynie pierwsze
`446` bajtów. Odpalamy zatem system live i wpisujemy w terminalu te poniższe polecenia:

    $ sudo su
    # dd if=/dev/urandom of /dev/sda bs=446 count=1

## Wolna przestrzeń za MBR (MBR gap)

Większość narzędzi do partycjonowania dysków, tworzy partycje z uwzględnieniem wyrównania do 1MiB.
Jest to za sprawą [technologi Advanced
Format](https://wiki.archlinux.org/index.php/Advanced_Format), którą stosują obecnie wszystkie
nowsze dyski twarde. Równanie do 1MiB powoduje przesunięcie pierwszej partycji o 2047 sektorów
względem MBR. Jako, że MBR znajduje się na pozycji 0, to pierwsza partycja będzie się zaczynać od
sektora 2048. By dokładnie zweryfikować ustawienia partycji, trzeba zajrzeć do `fdisk` :

    # fdisk -l
    ...
    Device     Boot     Start       End   Sectors   Size Id Type
    /dev/sda1            2048  83888127  83886080    40G  7 HPFS/NTFS/exFAT
    ...

Te wolne 2047 sektorów służyło kiedyś do przechowywania części kodu bootloader'a. Obecnie są one w
dużej mierze nieużywane. Te wirusy, które potrafią modyfikować MBR, potrafią ulokować cześć swojego
kodu właśnie w tej wolnej przestrzeni za sektorem rozruchowym. Wobec czego, przydałoby się tę
przestrzeń również wyczyścić:

    dd if=/dev/urandom of /dev/sda bs=512 count=2047 seek=1

## Partycja /boot/

Na partycji `/boot/` znajdują się obrazy z modułami kernela oraz szereg plików konfiguracyjnych i
skryptów, które umożliwiają zamontowanie głównego systemu plików przy starcie systemu. Pliki na tej
partycji, podobnie jak MBR, nie są w żaden sposób zabezpieczone. Dlatego też musimy kompletnie
wyczyścić tę partycję:

    # dd if=/dev/urandom of /dev/sda3

## Wykorzystanie chroot

Musimy teraz przywrócić [kopię
MBR](/post/mbr-ebr-i-tablica-partycji-dysku-twardego/) i stworzyć nową partycję
`/boot/` , np. przy użyciu `gparted`. Jeśli nie posiadamy kopi MBR, to jako że potrzebny nam jest
tylko kod bootloader'a, będziemy musieli go przeinstalować ale po kolei. Na sam początek otwieramy
szyfrowany system:

    # cryptsetup luksOpen /dev/sda4 sda4_crypt
    # vgscan
    # vgchange nazwa_grupy -a y
    # lvscan

Pamiętajmy by [odpowiednio zmodyfikować](https://forum.dug.net.pl/viewtopic.php?id=23053) parametr
`sda4_crypt` i by ta nazwa zgadzała się z tą zdefiniowaną w pliku `/etc/crypttab` . Po wydaniu
ostatniego polecenia powinniśmy uzyskać listę aktywnych dysków LVM. Musimy teraz zamontować
wszystkie niezbędne partycje i przy pomocy
[chroot](/post/przygotowanie-srodowiska-chroot-do-pracy/) uzyskać dostęp do tego
systemu. Pamiętajmy też by podmontować partycję `/boot/` w odpowiednim miejscu.

    # mount /dev/mapper/sda4_crypt-root /mnt
    # mount /dev/sda3 /mnt/boot
    # mount -o bind /proc /mnt/proc
    # mount -o bind /sys /mnt/sys
    # mount -o bind /dev /mnt/dev
    # mount -o bind /dev/pts /mnt/dev/pts
    # cp /etc/resolv.conf /mnt/etc/resolv.conf

Teraz już pozostało nam zrootować środowisko i zalogować się na szyfrowany system:

    # chroot /mnt/ /bin/bash

### Reinstalacja bootloader'a

Za pomocą menadżerów pakietów `apt-get` lub `aptitude` reinstalujemy bootloader. Ja korzystam z
`extlinux` , także jego reinstalacja sprowadza się do wydania tych dwóch poniższych poleceń:

    # dd if=/usr/lib/SYSLINUX/mbr.bin bs=440 count=1 iflag=fullblock conv=notrunc of=/dev/sda
    # extlinux -i /mnt/boot/extlinux

Jako, że usunęliśmy wszystkie pliki z katalogu `/boot/` , sama reinstalacja bootloader'a nam nie
odpali systemu. Ptrzebujemy jeszcze plików konfiguracyjnych bootloader'a. Jeśli korzystamy z
automatów, to wystarczy przeinstalować określone pakiety:

    # aptitude reinstall extlinux syslinux-common syslinux-utils

Musimy także zaktualizować numerk UUID partycji `/boot/` w pliku `/etc/fstab` , a ten z kolei możemy
uzyskać przy pomocy `lsblk` :

    # lsblk -o "NAME,SIZE,FSTYPE,TYPE,LABEL,MOUNTPOINT,UUID"
    NAME                       SIZE FSTYPE      TYPE  LABEL       MOUNTPOINT   UUID
    ...
    ├─sda3                       1G ext4        part  boot        /boot        4c67dff5-3d8e-4b3f-9cf1-49b88d5f67a9
    ...

### Reinstalacja kernela

Nieuchronna będzie także reinstalacja kernela. Sprowadza się ona do przeinstalowania pakietów, które
mają w nazwie `linux-` . Możemy to zrobić przy pomocy poniższego polecenia:

    # aptitude reinstall linux-~i

Czasem reinstalacja kernela może powodować problemy. Jeśli nie dałoby rady przeinstalować tych
powyższych pakietów, trzeba będzie je usunąć (opcja `purge` ) i ponownie zainstalować.
