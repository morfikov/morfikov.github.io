---
author: Morfik
categories:
- Linux
date: "2016-07-01T14:27:57Z"
date_gmt: 2016-07-01 12:27:57 +0200
published: true
status: publish
tags:
- system-plików
- ext4
GHissueID: 387
title: 'Unexpected Inconsistency: Inode has corrupt extent header'
---

Dzisiaj system plików jednej z partycji mojego głównego dysku uległ awarii z niewiadomych przyczyn.
Dałbym sobie nawet głowę uciąć, że wszystkie partycje zostały poprawnie odmontowane podczas
wyłączania maszyny. Niemniej jednak, z jakiegoś powodu podczas startu systemu, ten wyrzuca szereg
komunikatów dotyczących głównego systemu plików, tj. `/` . Sam komunikat brzmi mniej więcej tak:
`UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY` . Oznacza to, że błędy w systemie plików nie są łatwe
do naprawy i wymagana jest nasza ingerencja w ten proces. Przy sprawdzaniu systemu plików w
poszukiwaniu błędów przy pomocy `fsck.ext4` można było dostrzec min. taką wiadomość: `Inode 556975
has corrupt extent header` . Co ona tak naprawdę oznacza i czy damy radę wybrnąć z tej sytuacji bez
szwanku?

<!--more-->
## System live

Jako, że system plików zawiera błędy, to trzeba go odmontować w ewentualnym procesie naprawy.
Problem w tym, że jest to główny system plików. Nie możemy go zwyczajnie odmontować i zapuścić na
nim `fsck.ext4` . Tzn. możemy ale wtedy nie będziemy mieli żadnej kontroli nad procesem naprawczym.
Dlatego też potrzebny nam jest system live w formie CD/DVD/pendrive. Jeśli nie dysponujemy żadnym z
w/w, to trzeba się w taki system live zaopatrzyć pobierając go ze [strony
debiana](https://www.debian.org/CD/live/). Mając system live, możemy uruchomić maszynę i załadować
ten system do pamięci RAM bez wprowadzania zmian na głównym dysku twardym i o to nam głównie chodzi.

## FSCK: Inode 556975 has corrupt extent header

W tym przypadku system plików jest szyfrowany ale ten fakt na dobrą sprawę na nic nie wpływa.
Jedynie co, to trzeba otworzyć pierw kontener LUKS ( `cryptsetup luksOpen` ) przed skanowaniem
dysku, czyli standardowa procedura. Mając już dostęp do konkretnej partycji możemy przy pomocy
`fsck.ext4` zeskanować ją w poniższy sposób:

    # fsck.ext4 -vf /dev/mapper/debian_laptop-root

Poniżej zaś są wszystkie komunikaty, które zostały zwrócone w procesie skanowania:

    e2fsck 1.43 (17-May-2016)
    Pass 1: Checking inodes, blocks, and sizes
    Inode 556975 has corrupt extent header.  Clear inode<y>? yes
    Inode 556975, i_blocks is 56, should be 0.  Fix<y>? yes
    Pass 2: Checking directory structure
    Entry 'gconv-modules.cache' in /usr/lib/x86_64-linux-gnu/gconv (525709) has deleted/unused inode 556975.  Clear<y>? yes
    Pass 3: Checking directory connectivity
    Pass 4: Checking reference counts
    Pass 5: Checking group summary information
    Block bitmap differences:  -(2132608--2132614)
    Fix<y>? yes
    Free blocks count wrong for group #65 (1208, counted=1215).
    Fix<y>? yes
    Free blocks count wrong (1271388, counted=1271395).
    Fix<y>? yes
    Inode bitmap differences:  -556975
    Fix<y>? yes
    Free inodes count wrong for group #67 (9, counted=10).
    Fix<y>? yes
    Free inodes count wrong (566692, counted=566693).
    Fix<y>? yes

    root: ***** FILE SYSTEM WAS MODIFIED *****

          219739 inodes used (27.94%, out of 786432)
             297 non-contiguous files (0.1%)
             317 non-contiguous directories (0.1%)
                 # of inodes with ind/dind/tind blocks: 0/0/0
                 Extent depth histogram: 185938/64
         1874333 blocks used (59.58%, out of 3145728)
               0 bad blocks
               1 large file

          164603 regular files
           21014 directories
              12 character device files
              25 block device files
               0 fifos
              28 links
           34073 symbolic links (33687 fast symbolic links)
               4 sockets
    ------------
          219758 files

Na samym początku mamy informację: `Inode 556975 has corrupt extent header` , czyli uszkodzony
i-węzeł. Trzeba go zwyczajnie wyczyścić. Niżej z kolei mamy komunikat `Entry 'gconv-modules.cache'
in /usr/lib/x86_64-linux-gnu/gconv (525709) has deleted/unused inode 556975` , czyli plik
`gconv-modules.cache` był podpięty pod ten i-węzeł, który właśnie usunęliśmy. Pliki w systemie
plików nie mogą istnieć bez i-węzłów, dlatego też zwyczajnie uwaliliśmy ten plik. I tu właśnie jest
sedno całego problemu.

## Apt-file i chroot

Dokładnie nie wiem za co ten plik odpowiada ale załóżmy na chwilę, że za coś ważnego. Za coś, bez
czego system się nie będzie chciał nam uruchomić albo będzie działał bardzo niestabilnie. Powyższy
log dostarczył nam praktycznie wszystkich informacji potrzebnych do przywrócenia systemu do stanu
sprzed awarii. Musimy tylko przy pomocy środowiska `chroot` zainstalować pakiet, który za ten plik
jest odpowiedzialny. [Dokładna procedura tworzenia środowiska chroot jest opisana
tutaj](/post/przygotowanie-srodowiska-chroot-do-pracy/).

By odnaleźć uszkodzony plik, musimy [przeszukać zawartość pakietów przy pomocy
apt-file](/post/przeszukiwanie-zawartosci-pakietow-apt-file/). To narzędzie możemy
zainstalować poza środowiskiem `chroot` . Następnie aktualizujemy bazę danych plików i szukamy pliku
`/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache` :

    # apt-file update
    # apt-file search /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
    libc6: /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache

Wiemy zatem, że uszkodzony plik jest dostarczany z pakietem `libc6` . Ten pakiet musimy ponownie
zainstalować w naszym głównym systemie. Wchodzimy zatem do niego przy pomocy `chroot` i
reinstalujemy pakiet:

    # chroot /mnt/ /bin/bash
    # aptitude reinstall libc6

I to w zasadzie cały proces odzyskiwania kontroli nad systemem, którego system plików uległ awarii.
W tej chwili możemy zresetować maszynę i powinniśmy być w stanie uruchomić główny system bez żadnego
problemu.
