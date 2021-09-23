---
author: Morfik
categories:
- Linux
date: "2015-06-11T17:06:16Z"
date_gmt: 2015-06-11 15:06:16 +0200
published: true
status: publish
tags:
- live
- system-plików
title: Squashfs jako system plików obrazów live
---

System live to nic innego jak spakowany system plików jakieś zainstalowanej dystrybucji, który
podczas startu maszyny jest montowany w trybie tylko do odczytu, a potrzebne pliki są wypakowywane w
czasie rzeczywistym w pamięci operacyjnej RAM. Do implementacji takiego rozwiązania stosuje się
[squashfs](https://pl.wikipedia.org/wiki/SquashFS) i jako że jest to system plików tylko do
odczytu, nie można go jako tako edytować. Nie jesteśmy też zupełnie na straconej pozycji, bo możemy
skorzystać z dwóch rozwiązań:
[persistence](/post/persistence-czyli-zachowanie-zmian-w-systemie-live/) lub możemy
pokusić się o przepakowanie systemu plików.

<!--more-->
## Operowanie na systemie plików squashfs

Najsampierw zlokalizujmy plik `.squashfs` , bo to na nim będziemy operować. Przechodzimy zatem do
głównego katalogu pendrive, na którym mamy wgrany system live i listujemy jego zawartość:

    # cd /media/morfik/good/

    $ ls -al
    total 632K
    drwxr-xr-x 9 morfik root   4.0K 2015-06-10 22:21:27 ./
    drwxr-xr-x 3 morfik morfik 4.0K 2015-06-11 16:08:34 ../
    dr-xr-xr-x 3 morfik root   4.0K 2015-06-06 15:59:10 dists/
    dr-xr-xr-x 2 morfik root   4.0K 2015-06-10 22:21:16 extlinux/
    dr-xr-xr-x 3 morfik root   4.0K 2015-06-06 16:09:41 install/
    dr-xr-xr-x 2 morfik root   4.0K 2015-06-06 16:08:29 live/
    drwx------ 2 morfik root    16K 2015-06-09 21:18:53 lost+found/
    dr-xr-xr-x 3 morfik root   4.0K 2015-06-06 15:59:00 pool/
    dr-xr-xr-x 2 morfik root   4.0K 2015-06-06 16:09:37 tools/
    -r--r--r-- 1 morfik root    133 2015-06-06 16:09:44 autorun.inf
    lrwxrwxrwx 1 morfik root      1 2015-06-06 15:59:10 debian -> ./
    -r--r--r-- 1 morfik root   177K 2015-06-06 16:09:44 g2ldr
    -r--r--r-- 1 morfik root   8.0K 2015-06-06 16:09:44 g2ldr.mbr
    -r--r--r-- 1 morfik root    28K 2015-06-06 16:09:57 md5sum.txt
    -r--r--r-- 1 morfik root   360K 2015-06-06 16:09:44 setup.exe
    -r--r--r-- 1 morfik root    228 2015-06-06 16:09:44 win32-loader.ini

Plik, o który nam chodzi, znajduje się w katalogu `live/` pod niezbyt zaskakującą nazwą
`filesystem.squashfs` . Kopiujemy go teraz na komputer, po czym wypakowujemy. Z tym, że do tego celu
potrzebujemy odpowiednich narzędzi, a te są dostępne w pakiecie `squashfs-tools` . Jeśli nie mamy
tego pakietu w systemie, to go musimy doinstalować. System plików z kolei wypakowujemy w poniższy
sposób:

    # cp live/filesystem.squashfs /media/live/
    # cd /media/live/
    # unsquashfs filesystem.squashfs
    Parallel unsquashfs: Using 2 processors
    134474 inodes (142880 blocks) to write
    [=========/               ]  12010/142880   8%

W wyniku tej operacji, zostanie utworzony katalog `squashfs-root/` i to własnie jest rozpakowany
system plików. By wprowadzić zmiany, musimy skorzystać ze [środowiska
chroot](/post/przygotowanie-srodowiska-chroot-do-pracy/), a w nim działania
przeprowadzamy tak samo jak w normalnym systemie, tylko za pomocą trybu tekstowego. Po zakończeniu
prac, system plików trzeba ponownie spakować:

    # mksquashfs squashfs-root filesystem.squashfs -b 1024k -comp xz -Xbcj x86

Ta operacja zajmie chwilę, a gdy dobiegnie końca, to plik `filesystem.squashfs` wgrywamy, w tym
przypadku, do katalogu `/media/morfik/good/live/` . Domyślnie jednak ilość wolnego miejsca po
wgraniu obrazu na pendrive nie zezwala na zbytnie modyfikacje i jeśli dokonujemy większych zmian w
systemie plików, to potrzebujemy alternatywnego sposobu [przygotowania nośnika pod system
live](/post/jak-wgrac-system-live-na-uszkodzony-pendrive/), tak by mieć do
dyspozycji nieco więcej miejsca.
