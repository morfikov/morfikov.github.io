---
author: Morfik
categories:
- Linux
date: "2020-03-06T02:45:00Z"
lastmod: 2020-03-12 16:30:00 +0100
published: true
status: publish
tags:
- debian
- firmware
- wifi
- kernel
GHissueID: 24
title: Jak załadować firmware karty WiFi przed initrd/initramfs
---

Każdy kto ma laptopa wyposażonego w kartę WiFi, czy też ogólnie komputer posiadający bezprzewodową
sieciówkę, ten prawdopodobnie spotkał się z błędem podobnym do tego: `Direct firmware load for
iwlwifi-6000g2a-6.ucode failed with error -2` . W tym przypadku sprawa dotyczyła karty `Intel
Corporation Centrino Advanced-N 6205 [Taylor Peak]` działającej w oparciu o moduł kernela
`iwlwifi` . W takich przypadkach zwykle wystarczy zainstalować firmware od określonego modułu i po
kłopocie. No i faktycznie w Debianie jest dostępny pakiet `firmware-iwlwifi` , który zawiera ten
potrzebny plik `iwlwifi-6000g2a-6.ucode` . Problem jednak w tym, że instalacja paczki z firmware
niekoniecznie może nam pomóc. Ten powyższy przykład nie jest odosobniony i czasami pliki z firmware
muszą być dostępne w chwili ładowania kernela do pamięci RAM czy też na etapie initramfs/initrd. W
takim przypadku zainstalowanie paczki z firmware w naszym linux'ie nic nam nie da, bo pliki
rezydują na niezamontowanym jeszcze dysku. Jak zatem wybrnąć z tej wydawać by się było patowej
sytuacji?

<!--more-->
## Kernel, /lib/firmware/ i zaszyfrowany linux

Technicznie rzecz biorąc ilekroć instalujemy w Debianie paczki mające w nazwie `firmware-*` , to
pliki z firmware są umieszczane w katalogu `/lib/firmware/` na dysku. Te pliki są czytane przez
kernel w momencie ładowania określonego modułu, np. gdy wydamy polecenie `modprobe iwlwifi` .
Niemniej jednak, bywają sytuacje, w których ta lokalizacja nie zawsze jest dostępna. Przykładem
może być zaszyfrowany system, który wymaga obecności obrazu initrd. Po tym jak kernel zostanie
załadowany do pamięci RAM podczas fazy boot, to ładowany jest też obraz initrd w celu odszyfrowania
dysku i zamontowania głównego systemu plików. Problem natomiast pojawia się w momencie, gdy taki
moduł wymagający do poprawnego działania dodatkowych plików firmware wkompilujemy na stałe w
kernel. W takim przypadku, moduł będzie obecny w kernelu od razu po załadowaniu go do pamięci
operacyjnej ale pliki z firmware ciągle będą rezydować w obrębie zaszyfrowanej przestrzeni dysku.
Taki stan rzeczy powoduje, że kernel nie widząc firmware zwraca poniższe błędy:

    kernel: iwlwifi 0000:03:00.0: firmware: failed to load iwlwifi-6000g2a-6.ucode (-2)
    kernel: iwlwifi 0000:03:00.0: firmware: failed to load iwlwifi-6000g2a-6.ucode (-2)
    kernel: iwlwifi 0000:03:00.0: Direct firmware load for iwlwifi-6000g2a-6.ucode failed with error -2
    kernel: iwlwifi 0000:03:00.0: firmware: failed to load iwlwifi-6000g2a-5.ucode (-2)
    kernel: iwlwifi 0000:03:00.0: firmware: failed to load iwlwifi-6000g2a-5.ucode (-2)
    kernel: iwlwifi 0000:03:00.0: Direct firmware load for iwlwifi-6000g2a-5.ucode failed with error -2
    kernel: iwlwifi 0000:03:00.0: minimum version required: iwlwifi-6000g2a-5
    kernel: iwlwifi 0000:03:00.0: maximum version supported: iwlwifi-6000g2a-6
    kernel: iwlwifi 0000:03:00.0: check git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git

Brak firmware z kolei skutkuje problemami w obsłudze sprzętu lub też całkowicie uniemożliwia mu
funkcjonowanie. W tym przypadku nawet nie pojawił się interfejs sieciowy `wlan0` sprawiając
wrażenie, że karta sieciowa jest być może uszkodzona i nie ruszy pod linux.

# EXTRA_FIRMWARE_DIR i EXTRA_FIRMWARE

Niemniej jednak, jeśli [kompilujemy własny kernel dla określonej maszyny][1], to musimy nieco
inaczej go skonfigurować, by ten zaistniały problem wyeliminować. W kernelu mamy dostępne dwie
opcje `CONFIG_EXTRA_FIRMWARE_DIR` oraz `CONFIG_EXTRA_FIRMWARE` , które mogą nam pomóc uporać się z
tym zadaniem. Ta pierwsza z nich przyjmuje w argumencie katalog, z którego kernel ma czytać pliki
firmware podczas procesu kompilacji. Druga opcja zaś określa nazwy plików firmware (oddzielone
spacjami), które chcemy załączyć. W tym przypadku te parametry trzeba ustawić w poniższy sposób:

    CONFIG_EXTRA_FIRMWARE_DIR="/lib/firmware"
    CONFIG_EXTRA_FIRMWARE="iwlwifi-6000g2a-6.ucode"

Taka konfiguracja sprawi, że w kernel bezpośrednio zostanie wbudowany plik firmware i nie będzie
już potrzeby ładować go z zewnętrznego źródła, o czym możemy przekonać się obserwując dokładnie
proces kompilacji jądra:

    ...
    make KERNELRELEASE=5.5.7-amd64 ARCH=x86_64      KBUILD_BUILD_VERSION=18 -f ./Makefile
      DESCEND  objtool
      CALL    scripts/atomic/check-atomics.sh
      CALL    scripts/checksyscalls.sh
      CHK     include/generated/compile.h
      UPD     include/generated/compile.h
      CC      init/version.o
      AR      init/built-in.a
      AR      crypto/built-in.a
      GZIP    kernel/config_data.gz
      CC      kernel/configs.o
      AR      kernel/built-in.a
      CC      drivers/base/firmware_loader/fallback_table.o
      CC      drivers/base/firmware_loader/main.o
      CC      drivers/base/firmware_loader/fallback.o
      UPD     drivers/base/firmware_loader/builtin/iwlwifi-6000g2a-6.ucode.gen.S
      AS      drivers/base/firmware_loader/builtin/iwlwifi-6000g2a-6.ucode.gen.o
      AR      drivers/base/firmware_loader/builtin/built-in.a
      AR      drivers/base/firmware_loader/built-in.a
      AR      drivers/base/built-in.a
      AR      drivers/built-in.a
      GEN     .version
      CHK     include/generated/compile.h
      LD      vmlinux.o
    ...

Po załadowaniu się kerenla, karta WiFi powinna być już sprawna:

    kernel: iwlwifi 0000:03:00.0: loaded firmware version 18.168.6.1 op_mode iwldvm
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEBUG disabled
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEBUGFS disabled
    kernel: iwlwifi 0000:03:00.0: CONFIG_IWLWIFI_DEVICE_TRACING enabled
    kernel: iwlwifi 0000:03:00.0: Detected Intel(R) Centrino(R) Advanced-N 6205 AGN, REV=0xB0
    kernel: ieee80211 phy0: Selected rate control algorithm 'iwl-agn-rs'
    Mar 05 23:02:20 morfikownia kernel: iwlwifi 0000:03:00.0: Radio type=0x1-0x2-0x0

## Kopiowanie plików firmware do initrd/initramfs

Jeśli z jakichś względów to powyższe rozwiązanie nie wchodzi w grę, np. nie budujemy kernela
samodzielnie, to zawsze możemy przekopiować potrzebne pliki firmware do obrazu initrd/initramfs. W
tym celu potrzebny nam będzie skrypt, który trzeba umieścić w katalogu
`/etc/initramfs-tools/hooks/` . Stwórzmy sobie zatem plik `fix_missing_firmware` w tej lokalizacji
(pamiętajmy o nadaniu mu praw wykonywania) oraz dodajmy do niego tę poniższą treść:

    #!/bin/sh -e

    # Copy missing firmware files

    PREREQ=""

    prereqs () { echo "${PREREQ}"; }

    case "${1}" in prereqs) prereqs; exit 0 ;; esac ;

    . /usr/share/initramfs-tools/hook-functions

    echo -n "Copying missing firmware files... "

    [ ! -d "${DESTDIR}/lib/firmware/" ] && mkdir -p ${DESTDIR}/lib/firmware/

    cp /lib/firmware/iwlwifi-6000g2a-6.ucode ${DESTDIR}/lib/firmware/

    echo "done."

    exit 0

Zapisujemy plik i regenerujemy obraz initrd/initramfs:

    # update-initramfs -u

    update-initramfs: Generating /boot/initrd.img-5.5.8-amd64
    Copying missing firmware files... done.

Dla pewności możemy jeszcze podejrzeć czy pliki firmware zostały uwzględnione w obrazie:

    # lsinitramfs /boot/initrd.img-5.5.8-amd64 | grep firmware

    usr/lib/firmware
    usr/lib/firmware/iwlwifi-6000g2a-6.ucode
    ...

No i jak widać, są. Po restarcie maszyny, kernel powinien te pliki z obrazu initrd/initramfs
podebrać automatycznie i załadować. W przypadku gdyby się tak nie stało to pozostaje nam jedynie
wkompilowanie firmware bezpośrednio w kernel.


[1]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
