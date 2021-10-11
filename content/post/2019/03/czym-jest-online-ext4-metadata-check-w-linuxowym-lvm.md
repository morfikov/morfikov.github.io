---
author: Morfik
categories:
- Linux
date: "2019-03-17T19:10:30Z"
published: true
status: publish
tags:
- debian
- system-plików
- ext4
- lvm
GHissueID: 302
title: Czym jest Online ext4 Metadata Check w linux'owym LVM
---

Przeglądając dzisiaj rano logi systemowe wpadł mi w oczy komunikat, którego treść brzmiała mniej
więcej tak: `e2scrub Volume group "wd_blue_label" has insufficient free space (0 extents): 64
required` , po którym z kolei można zanotować `e2scrub snapshot FAILED, will not check!` oraz
`Failed to start Online ext4 Metadata Check for /media/Debian` . Oczywiście ten punkt montowania to
nazwa partycji odnosząca się do jednego z dysków logicznych struktury LVM. Skąd się te błędy
wzięły? Przecież jeszcze do niedawna (przez ostatnich parę lat) wszystko z moim linux'em
rezydującym na dysku (LUKS+LVM) było w porządku, a teraz nagle takie bardzo niepokojące błędy.
Czym jest w ogóle ten `e2scrub` i czym jest ten cały `Online ext4 Metadata Check` , który
najwyraźniej ma coś wspólnego ze sprawdzaniem systemu plików voluminów logicznych w locie? No i
najważniejsze chyba pytanie -- czemu to nie działa jak należy?

<!--more-->
## Skąd się wziął e2scrub i Online ext4 Metadata Check

Analizując komunikaty w logu systemowym można znaleźć parę ciekawych informacji odnośnie tych
wiadomości, które zostały wymienione we wstępie artykułu. Poniżej znajduje się dokładny wycinek
logu i wypadałoby mu się nieco bardziej przyjrzeć:

    systemd[1]: Starting Online ext4 Metadata Check for /media/Debian...
    e2scrub@-media-Debian[117266]:   Volume group "wd_blue_label" has insufficient free space (0 extents): 64 required.
    e2scrub@-media-Debian[117266]: /media/Debian: e2scrub snapshot FAILED, will not check!
    systemd[1]: e2scrub@-media-Debian.service: Main process exited, code=exited, status=1/FAILURE
    systemd[1]: e2scrub@-media-Debian.service: Failed with result 'exit-code'.
    systemd[1]: Failed to start Online ext4 Metadata Check for /media/Debian.
    systemd[1]: e2scrub@-media-Debian.service: Triggering OnFailure= dependencies.
    systemctl[117263]: Job for e2scrub@-media-Debian.service failed because the control process exited with error code.
    systemctl[117263]: See "systemctl status e2scrub@-media-Debian.service" and "journalctl -xe" for details.
    systemd[1]: Created slice system-e2scrub_fail.slice.
    systemd[1]: Starting Online ext4 Metadata Check Failure Reporting for /media/Debian...
    systemd[1]: e2scrub_fail@-media-Debian.service: Succeeded.
    systemd[1]: Started Online ext4 Metadata Check Failure Reporting for /media/Debian.

To co się rzuca w oczy od razu to `insufficient free space (0 extents): 64 required` . Wymagane są
64 wolne zakresy w grupie voluminów `wd_blue_label` . Te 64 zakresy, to 256 MiB, bo każdy zakres w
LVM ma 4 MiB. I to jest w zasadzie przyczyna problemów. Widoczna jest również usługa dla systemd:
`e2scrub@-media-Debian.service` i wypadałoby sprawdzić, czy jest może ona wywoływana przez `cron`
albo też przez samego `systemd` za sprawą jego mechanizmu czasówek (timers):

    $ systemctl list-timers --all
    NEXT                         LEFT         LAST                         PASSED    UNIT                         ACTIVATES
    ...
    Sun 2019-03-24 03:10:50 CET  6 days left  Sun 2019-03-17 09:56:29 CET  7h ago    e2scrub_all.timer            e2scrub_all.service

    6 timers listed.

Wcześniej tego zegara nie było, zatem coś musiało się zmienić w konfiguracji systemowej. Pytanie
tylko dlaczego ja nic o ty mnie wiem. :D Mając jednak punkt zaczepienia, konkretnie chodzi o tę
cyklicznie wywoływaną usługę `e2scrub_all.service` , można ustalić jaki pakiet to ustrojstwo
zainstalował:

    $ apt-file search e2scrub_all.service
    e2fsprogs: /lib/systemd/system/e2scrub_all.service

    $ dpkg -L e2fsprogs | grep scrub
    /etc/cron.d/e2scrub_all
    /etc/e2scrub.conf
    /lib/systemd/system/e2scrub@.service
    /lib/systemd/system/e2scrub_all.service
    /lib/systemd/system/e2scrub_all.timer
    /lib/systemd/system/e2scrub_fail@.service
    /lib/systemd/system/e2scrub_reap.service
    /lib/udev/rules.d/96-e2scrub.rules
    /sbin/e2scrub
    /sbin/e2scrub_all
    /usr/lib/x86_64-linux-gnu/e2fsprogs/e2scrub_all_cron
    /usr/lib/x86_64-linux-gnu/e2fsprogs/e2scrub_fail
    /usr/share/man/man8/e2scrub.8.gz
    /usr/share/man/man8/e2scrub_all.8.gz

Wiemy zatem, że odpowiedzialnym za te błędy w logu systemowym jest pakiet `e2fsprogs` . Wypadałoby
teraz zajrzeć w `changelog` tego pakietu w poszukiwaniu jakichś sensownych informacji odnośnie
zmian, które w tym pakiecie zostały w ostatnim casie wprowadzone:

    $ apt-get changelog e2fsprogs

    e2fsprogs (1.45.0-1) unstable; urgency=medium

      ...
      * There is now an e2scrub script which will allow e2fsck to be run
        on mounted file systems using an LVM device.  There will be a systemd
        script to automatically run e2scrub on all ext4* file systems where it
        can be supported.
      ...
     -- Theodore Y. Ts'o <tytso@mit.edu>  Wed, 06 Mar 2019 12:55:18 -0500

Z komunikatu wyżej wynika jasno, że została wprowadzona nowa funkcjonalność za sprawą skryptu
`e2scrub`. Zajrzymy zatem do manuala `e2scrub` . Zgodnie z tym co można w nim wyczytać, to
`e2scrub` próbuje sprawdzić (bez naprawiania) metadane podmontowanych systemów plików z rodziny
ext2, ext3 i ext4, które rezydują w obrębie urządzenia LVM. `e2scrub` jest w stanie taki zabieg
przeprowadzić wykonując migawkę systemu plików i puszczając na niej `e2fsck` . Urządzenie LVM musi
jednak dysponować minimum 256 MiB wolnego miejsca (tych, których akurat zabrakło). To miejsce
zostanie przeznaczone na snapshot. W przypadku, gdy system plików jest dość ekstensywnie
wykorzystywany, to trzeba się liczyć z faktem, że te 256 MiB może być niewystarczające. Po
zakończonym procesie skanowania, migawka zostanie automatycznie usunięta. W przypadku, gdy nie
zostaną stwierdzone żadne błędy, licznik montowań danego systemu plików ulegnie wyzerowaniu. W taki
sposób możliwe jest przeprowadzenie reakcyjnego sprawdzania konsystencji systemu plików zamiast
cyklicznego. Przy większych partycjach, takie podejście jest w stanie zaoszczędzić nam sporo czasu
podczas startu systemu. Gdyby jednak podczas skanowania migawki jakieś błędy systemu plików się
pojawiły, to dany system plików zostanie oznaczony jako błędny i przy następnym uruchomieniu
komputera zostanie zainicjowany `e2fsck` na tej partycji, no chyba, że wcześniej sami oto się
postaramy.

No to już wiem dlaczego ten cały proces sprawdzania voluminów logicznych LVM się nie powiódł na
jednym z dysków w moim laptopie, bo krojąc go na partycje nie zmierzałem robić jakichkolwiek
migawek żadnego z jego dysków logicznych. Wygląda na to, że powinienem nieco zmienić układ dysków
logicznych na tym urządzeniu LVM i wykroić tak ze 4 GiB wolnego miejsca, by ten proces sprawdzania
systemu plików był w stanie działać jak należy.

## Jak przygotować nośnik LVM pod e2scrub

Jeśli jesteśmy sytuacji, gdzie cała wolna przestrzeń grupy voluminów została przeznaczona na
logiczne dyski, to niestety trzeba jeden z tych dysków pomniejszyć, a odzyskaną w ten sposób
przestrzeń zostawić w formie nieużywanej na potrzebny snapshot'a. Poniżej znajdują się statystyki
grup voluminów w moim laptopie:

    # pvscan
      PV /dev/mapper/sdb1_crypt   VG wd_blue_label    lvm2 [820.30 GiB / 0    free]
      PV /dev/mapper/sda2_crypt   VG wd_black_label   lvm2 [<230.88 GiB / 4.00 GiB free]
      Total: 2 [<1.03 TiB] / in use: 2 [<1.03 TiB] / in no VG: 0 [0   ]

Grupa voluminów `wd_black_label` ma jak widać już 4 GiB wolnego miejsca, z racji wykorzystywania go
przy procesie aktualizacji systemu. Te bardziej ryzykowne aktualizacje mojego Debiana są
przeprowadzane właśnie z wykorzystaniem migawki LVM, bo jeśli coś pójdzie nie tak, to taką migawkę
można bardzo szybko usunąć cofając przy tym wszelkie zmiany, które zostały wprowadzone za sprawą
aktualizacji systemu. Natomiast jak widać w przypadku grupy voluminów `wd_blue_label` , tu już
brakuje nam wolnego miejsca i trzeba je jakoś odzyskać. Spójrzmy zatem za rozkład partycji na tym
nośniku:

    # lsblk /dev/sdb
    NAME                            SIZE FSTYPE      TYPE  LABEL         MOUNTPOINT     UUID
    sdb                           931.5G             disk
    ├─sdb1                        820.3G crypto_LUKS part  wd_blue_label                66861f93-9fc7-46f9-b969-1ade25dcb898
    │ └─sdb1_crypt                820.3G LVM2_member crypt                              HTmcBl-es16-2tT3-TK2n-uOyE-NaTL-ZRAT1c
    │   ├─wd_blue_label-Debian      360G ext4        lvm   Debian        /media/Debian  141f5fee-3080-4852-9511-d9fb929028f7
    │   ├─wd_blue_label-android     320G ext4        lvm   android       /media/Android 91f0c6ba-a318-4f7e-94ad-5ccdef94dca4
    │   ├─wd_blue_label-grafi      99.3G ext4        lvm   grafi         /media/Grafi   abc7c0af-76b3-423e-8dd2-e89008614036
    │   └─wd_blue_label-ccache       41G ext4        lvm   ccache        /media/ccache  d340d1a1-3a02-449c-98e4-603679fba072
    ├─sdb2                         62.5G ntfs        part  windows                      78E28017E27FD838
    └─sdb3                         46.7G ntfs        part  win_data                     5E046FAF1D2A959B

Jako, że mam trochę wolnego miejsca jeszcze na partycji `ccache` , która w zasadzie i tak służy
jedynie w celach przyśpieszania kompilacji programów, to właśnie ta partycja zostanie pomniejszona
o 4 GiB. Wpisujemy zatem w terminal to poniższe polecenia:

    # systemctl stop media-ccache.mount
    # lvreduce -L -4G -r /dev/mapper/wd_blue_label-ccache
    fsck from util-linux 2.33.1
    ccache: clean, 75288/2686976 files, 1033158/10747904 blocks
    resize2fs 1.45.0 (6-Mar-2019)
    Resizing the filesystem on /dev/mapper/wd_blue_label-ccache to 9699328 (4k) blocks.
    The filesystem on /dev/mapper/wd_blue_label-ccache is now 9699328 (4k) blocks long.

      Size of logical volume wd_blue_label/ccache changed from 41.00 GiB (10496 extents) to 37.00 GiB (9472 extents).
      Logical volume wd_blue_label/ccache successfully resized.

Powinniśmy już mieć wolne 4 GiB przestrzeni w grupie voluminów `wd_blue_label` :

    # pvscan
      /dev/sdc: open failed: No medium found
      PV /dev/mapper/sdb1_crypt   VG wd_blue_label    lvm2 [820.30 GiB / 4.00 GiB free]
      PV /dev/mapper/sda2_crypt   VG wd_black_label   lvm2 [<230.88 GiB / 4.00 GiB free]
      Total: 2 [<1.03 TiB] / in use: 2 [<1.03 TiB] / in no VG: 0 [0   ]

Gdy zegar wybije następny moment, w który `e2scrub` będzie chciał przeprowadzić skanowanie systemu
plików któregoś z dysku logicznego tej grupy voluminów, to powinno mu się to już udać.

## Ręczne wymuszenie sprawdzenia systemu plików

Jeśli nie chce nam się czekać parę dni na przetestowanie zmian, które wprowadziliśmy, możemy
zainicjować ręczne skanowanie systemu plików. Wystarczy, że w terminalu uruchomimy usługę
`e2scrub_all.service` :

    # systemctl start  e2scrub_all.service

Zanim jednak tę usługę uruchomimy, warto zainteresować się plikiem `/etc/e2scrub.conf` i zmienić w
nim parametr `snap_size_mb=` tak, by pasował do ilości wolnego miejsca w grupach voluminów:

    snap_size_mb=4096

Po puszczeniu usługi, `e2scrub` stworzy kolejno migawkę dla każdego dysku logicznego LVM i
przeskanuje jej system plików:

    # lsblk /dev/sdb
    NAME                                   SIZE FSTYPE      TYPE  LABEL          MOUNTPOINT     UUID
    sdb                                  931.5G             disk
    ├─sdb1                               820.3G crypto_LUKS part  wd_blue_label                 66861f93-9fc7-46f9-b969-1ade25dcb898
    │ └─sdb1_crypt                       820.3G LVM2_member crypt                               HTmcBl-es16-2tT3-TK2n-uOyE-NaTL-ZRAT1c
    │   ├─wd_blue_label-Debian-real        360G             lvm
    │   │ ├─wd_blue_label-Debian           360G ext4        lvm   Debian         /media/Debian  141f5fee-3080-4852-9511-d9fb929028f7
    │   │ └─wd_blue_label-Debian.e2scrub   360G ext4        lvm   Debian                        141f5fee-3080-4852-9511-d9fb929028f7
    │   ├─wd_blue_label-android            320G ext4        lvm   android        /media/Android 91f0c6ba-a318-4f7e-94ad-5ccdef94dca4
    │   ├─wd_blue_label-grafi             99.3G ext4        lvm   grafi          /media/Grafi   abc7c0af-76b3-423e-8dd2-e89008614036
    │   ├─wd_blue_label-ccache              37G ext4        lvm   ccache                        d340d1a1-3a02-449c-98e4-603679fba072
    │   └─wd_blue_label-Debian.e2scrub-cow   4G             lvm
    │     └─wd_blue_label-Debian.e2scrub   360G ext4        lvm   Debian                        141f5fee-3080-4852-9511-d9fb929028f7
    ├─sdb2                                62.5G ntfs        part  windows                       78E28017E27FD838
    └─sdb3                                46.7G ntfs        part  win_data                      5E046FAF1D2A959B

Jak widać volumin "Debian" jest ciągle aktywny, zaś w logu mamy takie oto informacje:

    systemd[1]: Starting Online ext4 Metadata Check for /media/Debian...
    lvm[121988]: Monitoring snapshot wd_blue_label-Debian.e2scrub.
    e2scrub@-media-Debian[128007]:   Logical volume "Debian.e2scrub" created.
    e2scrub@-media-Debian[128007]: e2fsck 1.45.0 (6-Mar-2019)
    e2scrub@-media-Debian[128007]: Pass 1: Checking inodes, blocks, and sizes
    e2scrub@-media-Debian[128007]: Pass 2: Checking directory structure
    e2scrub@-media-Debian[128007]: Pass 3: Checking directory connectivity
    e2scrub@-media-Debian[128007]: Pass 4: Checking reference counts
    e2scrub@-media-Debian[128007]: Pass 5: Checking group summary information
    e2scrub@-media-Debian[128007]: Debian: 307831/23592960 files (0.2% non-contiguous), 91434555/94371840 blocks
    e2scrub@-media-Debian[128007]: /media/Debian: Scrub succeeded.
    e2scrub@-media-Debian[128007]: tune2fs 1.45.0 (6-Mar-2019)
    e2scrub@-media-Debian[128007]: Setting current mount count to 0
    e2scrub@-media-Debian[128007]: Setting time filesystem last checked to Sun Mar 17 18:42:35 2019
    dmeventd[121988]: No longer monitoring snapshot wd_blue_label-Debian.e2scrub.
    e2scrub@-media-Debian[128007]:   Logical volume "Debian.e2scrub" successfully removed
    e2scrub@-media-Debian[128007]: /media/Debian: Trimming free space.
    e2scrub@-media-Debian[128007]: fstrim: /media/Debian: the discard operation is not supported
    systemd[1]: e2scrub@-media-Debian.service: Succeeded.
    systemd[1]: Started Online ext4 Metadata Check for /media/Debian.

Wyżej widzimy `Setting current mount count to 0` , który resetuje licznik aktualnych montowań do 0,
bo nie stwierdzono żadnych błędów systemu plików.

Nie zawsze jednak będzie tak miło. Podczas sprawdzania partycji `/tmp/` log wyglądał już nieco
inaczej:

    systemd[1]: Starting Online ext4 Metadata Check for /tmp...
    lvm[121988]: Monitoring snapshot wd_black_label-tmp.e2scrub.
    e2scrub@-tmp[128288]:   Logical volume "tmp.e2scrub" created.
    e2scrub@-tmp[128288]: e2fsck 1.45.0 (6-Mar-2019)
    e2scrub@-tmp[128288]: Pass 1: Checking inodes, blocks, and sizes
    e2scrub@-tmp[128288]: Deleted inode 40 has zero dtime.  Fix? yes
    e2scrub@-tmp[128288]: Pass 2: Checking directory structure
    e2scrub@-tmp[128288]: Pass 3: Checking directory connectivity
    e2scrub@-tmp[128288]: /lost+found not found.  Create? yes
    e2scrub@-tmp[128288]: Pass 4: Checking reference counts
    e2scrub@-tmp[128288]: Pass 5: Checking group summary information
    e2scrub@-tmp[128288]: Free inodes count wrong for group #0 (8149, counted=8150).
    e2scrub@-tmp[128288]: Fix? yes
    e2scrub@-tmp[128288]: Free inodes count wrong (262069, counted=262070).
    e2scrub@-tmp[128288]: Fix? yes
    e2scrub@-tmp[128288]: tmp: ***** FILE SYSTEM WAS MODIFIED *****
    e2scrub@-tmp[128288]: tmp: 74/262144 files (2.7% non-contiguous), 36994/1048576 blocks
    e2scrub@-tmp[128288]: /tmp: Scrub FAILED due to corruption!  Unmount and run e2fsck -y.
    e2scrub@-tmp[128288]: tune2fs 1.45.0 (6-Mar-2019)
    e2scrub@-tmp[128288]: Setting filesystem error flag to force fsck.
    dmeventd[121988]: No longer monitoring snapshot wd_black_label-tmp.e2scrub.
    e2scrub@-tmp[128288]:   Logical volume "tmp.e2scrub" successfully removed
    systemd-journald[472]: Missed 2 kernel messages
    systemd[1]: e2scrub@-tmp.service: Main process exited, code=exited, status=1/FAILURE
    systemd[1]: e2scrub@-tmp.service: Failed with result 'exit-code'.
    systemd[1]: Failed to start Online ext4 Metadata Check for /tmp.
    systemd[1]: e2scrub@-tmp.service: Triggering OnFailure= dependencies.
    systemctl[128287]: Job for e2scrub@-tmp.service failed because the control process exited with error code.
    systemctl[128287]: See "systemctl status e2scrub@-tmp.service" and "journalctl -xe" for details.
    systemd[1]: Starting Online ext4 Metadata Check Failure Reporting for /tmp...
    systemd[1]: Starting Online ext4 Metadata Check for /var/tmp...
    systemd[1]: e2scrub_fail@-tmp.service: Succeeded.
    systemd[1]: Started Online ext4 Metadata Check Failure Reporting for /tmp.

No i tu już mamy błędy systemu plików, co oznajmia nam komunikat: `Scrub FAILED due to
corruption!` . Wyżej w logu zaś mamy również informację `Setting filesystem error flag to force
fsck` , która mówi nam, że system plików został oznaczony jako błędy. Przy następnej próbie
montowania tej partycji w systemie zostanie wymuszone skanowanie systemu plików.

## Wyłączenie e2scrub

Oczywiście ten mechanizm sprawdzania systemu plików na podmontowanych voluminach LVM za sprawą
`e2scrub` można wyłączyć jeśli go naturalnie nie potrzebujemy. Choć jak widać jest on bardzo
użyteczny. Jeśli nie chcemy z niego korzystać to wystarczy wpisać w terminalu te dwa poniższe
polecenia:

    # systemctl disable e2scrub_all.timer
    # systemctl stop e2scrub_all.timer

Dla pewności można sprawdzić czasówki:

    # systemctl list-timers -all

Jeśli nie widać tam `e2scrub_all.timer` , to znaczy, że już się nam automatycznie on nie uruchomi.
