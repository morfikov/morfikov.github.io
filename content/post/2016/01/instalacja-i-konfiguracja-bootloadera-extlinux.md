---
author: Morfik
categories:
- Linux
date: "2016-01-02T01:47:30Z"
date_gmt: 2016-01-02 00:47:30 +0100
published: true
status: publish
tags:
- mbr
- bootloader
title: Instalacja i konfiguracja bootloader'a extlinux
---

W debianie mamy do dyspozycji kilka
[bootloader'ów](https://pl.wikipedia.org/wiki/Program_rozruchowy). Z tych częściej używanych to
będą syslinux, extlinux oraz grub2. Jeśli potrzebujemy automatyzacji oraz szeregu zaawansowanych
ficzerów, to dobrym wyjściem jest grub2. Jeśli natomiast korzystamy ze standardowej konfiguracji,
którą można by określić mianem BIOS-MBR i do tego chcemy mieć pełną kontrolę na bootloader'em, to
najlepiej wybrać extlinux'a lub syslinux'a. Syslinux jest wykorzystywany głównie w przypadku
partycji FAT, która znajduje zastosowanie w różnego rodzaju systemach live. Natomiast jeśli w grę
wchodzi system plików EXT4, który jest domyślny na sporej części linux'ów, to pozostaje nam do
wyboru jedynie extlinux. W tym wpisie postaramy się przebrnąć przez proces instalacji i konfiguracji
tego bootloader'a. Nie będziemy przy tym korzystać z żadnych automatów i wszystko postaramy się
dostosować ręcznie.

<!--more-->
## Extlinux i partycja /boot/

Plik każdego bootloader'a, w tym też i extlinux'a, znajdują się w katalogu `/boot/` . Obecnie w
większości przypadków nie ma sensu stosowania oddzielnej partycji `/boot/` , no chyba, że korzysta
się z zaszyfrowanych kontenerów LUKS. Jeśli posiadamy nowszej klasy komputer, który wykorzystuje
[EFI](https://pl.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface), to nawet nie potrzebny
jest nam bootloader, by uruchomić nasz system operacyjny. Jest jednak wiele kontrowersji wokół tego
następcy BIOS'u i raczej nie prędko ludzie mu zaufają. Zwłaszcza po tych doniesieniach, że na EFI co
raz częściej linux'y nie chcą się odpalać. Tak czy inaczej, zawsze pozostaje nam to starsze
rozwiązanie, które robi użytek z bootloader'a.

## Instalacja potrzebnych pakietów

Na potrzeby tego artykułu, wydzieliłem na dysku osobną partycję, która nie jest jakichś wielkich
rozmiarów, bo ma 1 GiB. Jest to dość sporo jeśli rozważyć fakt, że na tej partycji mają być
przechowywane jedynie obrazy z modułami kernela i kilka plików bootloader'a. Niemniej jednak,
potrzebujemy jeszcze odpowiednich narzędzi i w debianie są one w tych poniższych pakietach:

    # aptitude install extlinux syslinux syslinux-common memtest86+

## Kopia MBR

Na wypadek gdyby coś poszło nie tak przy wgrywaniu kodu bootloader'a, dobrze jest sobie zrobić kopię
całego MBR, a najlepiej to przeczytać sobie wpis dotyczący [MBR, EBR i tablicy
partycji](/post/mbr-ebr-i-tablica-partycji-dysku-twardego/) oraz artykuł opisujący
[jak powinna wyglądać kopia struktury dysku
twardego](/post/kopia-struktury-dysku-twardego/), tak by w przyszłości uniknąć
wszelkich problemów wynikających z jej uszkodzenia. W ogromnym skrócie, sprowadza się to do wydania
tego poniższego polecenia:

    # dd if=/dev/sda of=/path/to/backup/sda_mbr_backup bs=512 count=1

Zawsze też możemy skorzystać z [mbrback](https://github.com/IgnorantGuru/mbrback).

Instalacja extlinux'a w MBR

Po instalacji pakietów i zrobieniu kopi MBR, wgrywamy program ładujący do MBR. Potrzebne nam pliki
znajdują się w katalogu `/usr/lib/EXTLINUX/` . Mamy tam z grubsza trzy pliki. W zależności od
wykorzystywanej tablicy partycji, wybieramy `gptmbr.bin` lub `mbr.bin` . Plik `altmbr.bin` jest
wykorzystywany w przypadku tablicy partycji MS-DOS, z tym, że umożliwia ustawienie na sztywno
partycji, z której ma wystartować system. W tym przypadku korzystamy z `mbr.bin` . Możemy go wgrać
na 2 sposoby:

    # dd if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/sda bs=440 count=1
    # cat /usr/lib/syslinux/mbr/mbr.bin > /dev/sda

Katalog /boot/extlinux/

W katalogu `/boot/extlinux/` będzie trzymana konfiguracja extlinux'a. Musimy ją przekopiować sobie z
katalogu `/usr/lib/syslinux/modules/` . Mamy tam do dyspozycji trzy inne podkatalogi: `bios/` ,
`efi32/` oraz `efi64/` . W tym przypadku interesują nas pliki w katalogu `bios/` . Kopiujemy zatem
jego zawartość do `/boot/extlinux/` :

    # cp /usr/lib/syslinux/modules/bios/* /boot/extlinux/

Ja dodatkowo zrobiłem sobie tapetę pod mojego extlinux'a i stworzyłem pusty plik konfiguracyjny
`extlinux.conf` .

Z reguły, obrazki muszą być w takiej samej rozdzielczości co okno bootloader'a. Jednak niektóre
ekrany nie zmienią rozdzielczości na taką, którą wpiszemy do pliku konfiguracyjnego. W takim
przypadku trzeba użyć obrazków w rozdzielczości 640x480, inaczej wystąpią dziwne efekty wizualne.
Problem w tym, że, np. laptopy, zwykle nie mają matryc w proporcjach 4:3 i wskazanie extlinux'owi
obrazka o takich proporcjach nieco go rozciągnie.

By obejść ten problem, trzeba odpowiednio przeskalować daną grafikę. Trzeba stworzyć pierw obrazek w
natywnej rozdzielczości monitora, przykładowo 1366x768, potem ładujemy go do gimp'a i skalujemy go
proporcjonalnie do rozdzielczości 854x480. Ostatnim krokiem jest zmiana bez proporcji pierwszego
parametru z 854 do 640 pixeli. To może i zniekształci nieco obrazek ale podczas ładowania go przez
extlinux, zostanie on rozciągnięty do prawidłowych proporcji.

Plik /boot/extlinux/ldlinux.sys

Część pliku `ldlinux.sys` musi zostać wgrana w pierwszy sektor partycji `/boot/`, zwany
[VBR](https://en.wikipedia.org/wiki/Volume_boot_record). Trzeba o tym pamiętać przy ewentualnym
dotykaniu tego pliku, gdyż może to zmienić jego położenie i w efekcie uniemożliwi to programowi
ładującemu, który siedzi w MBR zlokalizowanie tego pliku. Poniższe polecenie powinno utworzyć dwa
pliki `/boot/extlinux/ldlinux.c32` oraz `/boot/extlinux/ldlinux.sys` :

    # mkdir -p /boot/extlinux/
    # extlinux --install /boot/extlinux
    /boot/extlinux is device /dev/sda2

Plik /boot/extlinux/extlinux.conf

Bootloader jest już na swoim miejscu. Potrzebujemy jeszcze pliku `/boot/extlinux/extlinux.conf` ,
który nam ustawi szereg opcji i umożliwi wczytanie określonych plików z modułami kernela. Poniżej
jest przykład konfiguracji:

    DEFAULT debian-debian-normal
    PROMPT 0
    TIMEOUT 200

    UI vesamenu.c32

    #MENU RESOLUSION 1366 768

    MENU MARGIN 10
    MENU WIDTH 80
    MENU ROWS 10
    MENU HELPMSGROW 25
    MENU CMDLINEROW 25
    MENU TABMSGROW 25
    MENU HSHIFT 0
    MENU VSHIFT 0
    MENU TITLE Everybody lies xD
    MENU BACKGROUND background_640x480.png
    MENU COLOR border       30;44   #40ffffff #a0000000 std
    MENU COLOR title        1;36;44 #9033ccff #a0000000 std
    MENU COLOR sel          7;37;40 #e0ffffff #20ffffff all
    MENU COLOR unsel        37;44   #50ffffff #a0000000 std
    MENU COLOR help         37;40   #c0ffffff #a0000000 std
    MENU COLOR timeout_msg  37;40   #80ffffff #00000000 std
    MENU COLOR timeout      1;37;40 #c0ffffff #00000000 std
    MENU COLOR msg07        37;40   #90ffffff #a0000000 std
    MENU COLOR tabmsg       31;40   #30ffffff #00000000 std

    ALLOWOPTIONS 1
    #MENU MASTER PASSWD

    LABEL debian-debian-normal
    #   MENU PASSWD
        MENU LABEL Debian 4.3.0-1-amd64 (powersave)
        KERNEL ../vmlinuz-4.3.0-1-amd64
        APPEND root=/dev/mapper/debian_laptop-root cgroup_enable=memory acpi_osi="!Windows 2012" acpi=force acpi_enforce_resources=lax net.ifnames=1 apparmor=1 security=apparmor udev.children-max=64 plymouth.enable=1 quiet splash ro
        INITRD ../initrd.img-4.3.0-1-amd64
        TEXT HELP
        Zaszyfrowany debian (/dev/sda1).
        ENDTEXT

    LABEL Windows
          MENU LABEL Windows
          COM32 chain.c32
          APPEND hd0 4
          TEXT HELP
          Windows 10 (/dev/sda4).
          ENDTEXT

    LABEL memtest
          MENU LABEL Memtest
          LINUX ../memtest86+.bin

    LABEL hdt
          MENU LABEL HDT (Hardware Detection Tool)
          COM32 hdt.c32

    LABEL reboot
          MENU LABEL Reboot
          COM32 reboot.c32

    LABEL poweroff
          MENU LABEL Power Off
          COM32 poweroff.c32

    MENU CLEAR

Wyjaśnienie użytych opcji:

  - `MENU` odpowiada za wygląd extlinux'a.
  - `DEFAULT` określa, która pozycja na liście bootloader'a ma być domyślna. Trzeba pamiętać, że
    nazwa musi pasować do linijki z `LABEL` .
  - `TIMEOUT` czas, po upłynięciu którego bootloader załaduje domyślny wpis. Wartość `200` odpowiada
    za 20s.
  - `PROMPT` jeśli ustawiony na `1` wyświetli wiersz poleceń (boot:).
  - `UI` odpowiada za rodzaj wykorzystywanego interfejsu bootloader'a i zależy od załadowanego
    modułu.
  - `ALLOWOPTIONS` daje możliwość precyzowania dodatkowych parametrów z pozycji bootloader'a przez
    edycję wpisu (klawisz Tab ).
  - `MASTER PASSWD` zakłada hasło na wszystkie wpisy w extlinux'ie. Hasło jest zahashowane, a do
    wygenerowania hasha można wykorzystać `md5pass` lub `sha1pass` (dostępne w pakiecie
    `syslinux-utils` ).
  - `MENU CLEAR` czyści ekran po wybraniu pozycji.

Pozostała część pliki to pozycje na liście bootloader'a. Przyjrzyjmy się bliżej wpisowi od linux'a:

    LABEL debian-debian-normal
    #   MENU PASSWD
        MENU LABEL Debian 4.3.0-1-amd64 (powersave)
        KERNEL ../vmlinuz-4.3.0-1-amd64
        APPEND root=/dev/mapper/debian_laptop-root cgroup_enable=memory acpi_osi="!Windows 2012" acpi=force acpi_enforce_resources=lax net.ifnames=1 apparmor=1 security=apparmor udev.children-max=64 plymouth.enable=1 quiet splash ro
        INITRD ../initrd.img-4.3.0-1-amd64
        TEXT HELP
        Zaszyfrowany debian (/dev/sda1).
        ENDTEXT

Wyjaśnienie opcji:

  - `LABEL` definiuje kolejną pozycję wyboru na liście bootloader'a.
  - `MENU PASSWD` zakłada hasło na edycję parametrów tego wpisu (klawisz Tab w oknie bootloader'a).
    Hasło może być określone w przypadku tylko jednego wpisu, lub też globalnie dla wszystkich (
    `MENU MASTER PASSWD` ).
  - `MENU LABEL` to nazwa pod jaką wpis się pojawi w okienku bootloader'a.
  - `KERNEL` próbuje wykryć typ pliku, który ma zostać wczytany.
  - `LINUX` zawsze spodziewa się kernela linux'a.
  - `APPEND` określa parametry, które są przekazywane kernelowi.
  - `INITRD` odpowiada za lokalizację obrazu z modułami.
  - `TEXT HELP` i `ENDTEXT` precyzują tekst, który pojawi się po zaznaczeniu wpisu w menu
    extlinux'a.

Przy instalacji kenrela, w katalogu `/boot/` jest tworzonych kilka plików. W przypadku debiana są
to: `config-*` , `initrd.img-*` , `System.map-*` oraz `vmlinuz-*` . By skonfigurować wpis w
extlinux, interesują nas dwa z nich, tj. `vmlinuz-*` oraz `initrd.img-*` . Ścieżki do plików podaje
się względem pliku konfiguracyjnego, stąd `../` wskazujący na folder wyżej.

Mamy tam jeszcze wpis od windowsa:

    LABEL Windows
          MENU LABEL Windows
          COM32 chain.c32
          APPEND hd0 4
          TEXT HELP
          Windows 10 (/dev/sda4).
          ENDTEXT

Wyjaśnienie opcji:

  - `COM32` odpowiada za moduł, który odpali windowsa.
  - `APPEND` określa partycję, na której siedzi windows. Numerowanie jest trochę dziwne, bo dyski
    mają numery od 0, a partycje od 1.

Zamiast kopiować pliki extlinux'a, można potworzyć dowiązania symboliczne. Problem jest w sytuacji,
gdy szyfrujemy partycje. W takim przypadku partycja z plikami w katalogu `/usr/` podczas
wyświetlania okna wyboru systemu jest jeszcze zaszyfrowana i system nie ma dostępu do tych plików,
dlatego trzeba je przekopiować na partycję `/boot/` . Pamiętajmy, by te pliki regularnie
aktualizować.
