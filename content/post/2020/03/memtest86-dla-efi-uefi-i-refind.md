---
author: Morfik
categories:
- Linux
date: "2020-03-03T04:05:00Z"
lastmod: 2020-06-13 14:52:00 +0200
published: true
status: publish
tags:
- debian
- efi/uefi
- memtest86
- refind
title: Memtest86 dla EFI/UEFI i rEFInd
---

Zapewne każdy z nas słyszał o narzędziu do testowania pamięci operacyjnej RAM zwanym memtest86. W
Debianie są dostępne dwa pakiety [memtest86][1] ([strona projektu][2]) oraz [memtest86+][3]
([strona projektu][4]) , które można sobie zainstalować w systemie. Niemniej jednak, jak się
popatrzy na daty ostatnich wersji obu tych aplikacji (rok 2014), to mamy do czynienia z dość starym
oprogramowaniem. Tak czy inaczej, jeśli dany soft działa, to bez znaczenia powinno być jak stary on
jest. Problem w przypadku memtest86 dostarczanego w tych dwóch pakietach jest taki, że działa on w
zasadzie jedynie w konfiguracji BIOS, a nie EFI/UEFI. Dodatkowo, oryginalny memtest86 został
sprzedany PassMark'owi, który [od wersji 5.0 uczynił go własnościowym softem][5]. To dlatego w
Debianie nie będzie już nowszej wersji memtest86. W dalszym ciągu memtest86 może działać w
konfiguracji EFI/UEFI ale potrzebna nam jest wersja, która to EFI/UEFI wspiera. Memtest86 zaczął
wspierać EFI/UEFI od wersji 5.0. Jeśli nam nie przeszkadza licencja własnościowa, to możemy sobie
przygotować memtest86, tak by można go było bez problemu odpalić z menadżera rozruchu [rEFInd][6].

<!--more-->
## Przygotowanie memtest86 dla rEFInd

Po przejściu na [stronę projektu memtest86][2] naszym oczom powinien ukazać się przycisk pobierania,
którego to z kolei kliknięcie zainicjuje proces pobierania pliku `memtest86-usb.zip` . Sama paczka
ZIP nie waży dużo ale skrywa ona obraz `.img` , którego przeznaczeniem jest dostanie się na nośnik
USB. Nas to w zasadzie w ogóle nie interesuje i musimy z tego obrazu `.img` wydobyć interesujący
nas plik aplikacji memtest86, który będziemy ładować via rEFInd. Wypakujmy zatem pobraną paczkę ZIP:

    $ cd memtest
    $ wget https://www.memtest86.com/downloads/memtest86-usb.zip
    $ unzip  memtest86-usb.zip
    Archive:  memtest86-usb.zip
      inflating: memtest86-usb.img
      inflating: readme.txt
      inflating: imageUSB.exe
      inflating: ReadMe_imageUSB.txt
      inflating: MemTest86_User_Guide_UEFI.pdf
       creating: Help/
       creating: Help/HTML/
      inflating: Help/HTML/imageusb-banner.jpg
      inflating: Help/HTML/hmftsearch.htm
      inflating: Help/HTML/cicon_loadindex_ani.gif
      inflating: Help/HTML/imageusb_content_dyn.html
      inflating: Help/HTML/zoom_search.js
      inflating: Help/HTML/jquery.js
      inflating: Help/HTML/index.html
      inflating: Help/HTML/imageusb_kwindex_static.html
      inflating: Help/HTML/Thumbs.db
      inflating: Help/HTML/helpman_navigation.js
      inflating: Help/HTML/contacting_passmark_software.htm
     extracting: Help/HTML/cicon9.png
      inflating: Help/HTML/imageusb_popup_html.js
      inflating: Help/HTML/system_requirements.htm
      inflating: Help/HTML/imageusb_navigation.js
      inflating: Help/HTML/highlight.js
      inflating: Help/HTML/hmkwindex.htm
      inflating: Help/HTML/hmcontextids.js
      inflating: Help/HTML/introduction_and_overview.htm
      inflating: Help/HTML/zoom_index.js
      inflating: Help/HTML/imageusb_ftsearch.html
      inflating: Help/HTML/helpman_settings.js
      inflating: Help/HTML/zoom_pageinfo.js
      inflating: Help/HTML/helpman_topicinit.js
      inflating: Help/HTML/imageusb_content_static.html
      inflating: Help/HTML/settings.js
      inflating: Help/HTML/cicon9.gif
      inflating: Help/HTML/usage.htm
      inflating: Help/HTML/imageusb_kwindex_dyn.html
      inflating: Help/HTML/hmcontent.htm
      inflating: Help/HTML/default.css
      inflating: Help/HTML/gui.jpg
      inflating: Help/HTML/purchasing_information.htm

No i jak możemy zauważyć, po zakończonym procesie wypakowania, na dysku zostanie stworzony plik o
rozmiarze 500 MiB:

	$  ls -alh memtest86-usb.img
	-rw-r--r-- 1 morfik morfik 500M 2019-11-22 08:09:23 memtest86-usb.img

Musimy ten obraz teraz wypakować przy pomocy `7z` :

    $ 7z x -y memtest86-usb.img

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs Intel(R) Pentium(R) CPU        P6100  @ 2.00GHz (20655),ASM)

    Scanning the drive for archives:
    1 file, 524288000 bytes (500 MiB)

    Extracting archive: memtest86-usb.img
    --
    Path = memtest86-usb.img
    Type = GPT
    Physical Size = 524288000
    ID = 68264C0F-858A-49F0-B692-195B64BE4DD7

    Everything is Ok

    Files: 2
    Size:       522174464
    Compressed: 524288000

W wyniku tego powyższego polecenia zostanie stworzony plik `EFI System Partition.img` . Ten plik
również trzeba wypakować:

    $ 7z x -y "EFI System Partition.img" -oc:"memtest86"

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs Intel(R) Pentium(R) CPU        P6100  @ 2.00GHz (20655),ASM)

    Scanning the drive for archives:
    1 file, 261078528 bytes (249 MiB)

    Extracting archive: EFI System Partition.img
    --
    Path = EFI System Partition.img
    Type = FAT
    Physical Size = 261078528
    File System = FAT16
    Cluster Size = 4096
    Free Space = 254345216
    Headers Size = 295424
    Sector Size = 512
    ID = 2584546620

    Everything is Ok

    Folders: 4
    Files: 7
    Size:       6423833
    Compressed: 261078528

Zawartość obrazu `EFI System Partition.img` została wrzucona do podkatalogu `memtest86/` . Nas
najbardziej interesować będzie plik `memtest86/EFI/BOOT/BOOTX64.efi` -- to ten plik trzeba będzie
umieścić na [partycji ESP][8]. Niemniej jednak, [rEFInd wymaga][7] określonej nazwy dla tego pliku.
Można posłużyć się jedną z następujących nazw: `memtest86.efi` , `memtest86_x64.efi` ,
`memtest86x64.efi` albo `bootx64.efi` . Ja przepisałem nazwę do `memtest86_x64.efi` . Tak
przygotowany plik trzeba teraz umieścić w katalogu `EFI/tools/` na partycji ESP.

## Sprawdzenie instalacji memtest86

Jeśli wszystko zrobiliśmy prawidłowo, to rEFInd podczas startu powinien zaprezentować nam opcję
wejścia w memtest86 (to ta kostka RAM'u pod menu wyboru systemu).

![]({{< baseurl >}}/img/2020/03/001-memtest86-refind-selection-menu.jpg#huge)

Po uruchomieniu powinien nas przywitać poniższy ekran:

![]({{< baseurl >}}/img/2020/03/002-memtest86-config-menu.jpg#huge)

Naturalnie wybieramy ikonkę `config` :

![]({{< baseurl >}}/img/2020/03/003-memtest86-config-menu.jpg#huge)

I w zasadzie tyle roboty. Teraz już wystarczy tylko postępować według instrukcji na ekranie by
sobie przetestować pamięć operacyjną RAM.

## Problem z memtest86 w trybie Secure Boot

Binarka `memtest86_x64.efi` domyślnie jest podpisana kluczami Microsoft'u i będzie ona działać OOTB
w środowiskach EFI/UEFI posiadających zaszyte odpowiednie certyfikaty (czyli praktycznie na każdym
desktopie czy laptopie):

    $ sbverify --list memtest86/EFI/BOOT/BOOTX64.efi
    signature 1
    image signature issuers:
     - /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
    image signature certificates:
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Windows UEFI Driver Publisher
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation Third Party Marketplace Root

W przypadku, gdy [zaoraliśmy firmware EFI/UEFI i wgraliśmy swoje własne certyfikaty][9], to tę
binarkę trzeba podpisać swoim kluczem prywatnym `db` :

    # sbsign --cert db.crt --key db.key  memtest86/EFI/BOOT/BOOTX64.efi
    Image was already signed; adding additional signature

Problem w tym, że jak można zauważyć po komunikacie wyżej, w takim przypadku będą dwie sygnatury:

    # sbverify --list memtest86/EFI/BOOT/BOOTX64.efi.signed
    signature 1
    image signature issuers:
     - /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
    image signature certificates:
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Windows UEFI Driver Publisher
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
     - subject: /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation UEFI CA 2011
       issuer:  /C=US/ST=Washington/L=Redmond/O=Microsoft Corporation/CN=Microsoft Corporation Third Party Marketplace Root
    signature 2
    image signature issuers:
     - /CN=morfikov's kernel-signing key
    image signature certificates:
     - subject: /CN=morfikov's kernel-signing key
       issuer:  /CN=morfikov's kernel-signing key

Czemu niby miałoby to stanowić jakiś problem, przecie w zmiennej `db` mamy swój własny certyfikat,
którym firmware EFI/UEFI powinien bez problemu zweryfikować złożony pod tą binarką podpis? Wygląda
na to, że gdy binarka ma więcej niż jedną sygnaturę, to nie idzie zweryfikować jej żadnym
certyfikatem kluczy, które zostały użyte do podpisania takiego pliku. Trzeba zatem przy pomocy
`sbattach` (dostępnym w pakiecie `sbsigntool` ) usunąć tę poprzednią sygnaturę z pliku i dopiero
wtedy podpisać taką binarkę swoim własnym kluczem prywatnym:

    # sbattach --remove memtest86/EFI/BOOT/BOOTX64.efi

    # sbverify --list memtest86/EFI/BOOT/BOOTX64.efi
    No signature table present

    # sbsign --cert db.crt  --key db.key memtest86/EFI/BOOT/BOOTX64.efi
    Signing Unsigned original image

    # sbverify --list memtest86/EFI/BOOT/BOOTX64.efi.signed
    signature 1
    image signature issuers:
     - /CN=morfikov's kernel-signing key
    image signature certificates:
     - subject: /CN=morfikov's kernel-signing key
       issuer:  /CN=morfikov's kernel-signing key

I dopiero tak podpisanego memtest'a można skopiować do `/efi/EFI/tools/memtest86_x64.efi` , by
uniknąć problemów przy weryfikacji podpisu przez firmware EFI/UEFI.


[1]: https://tracker.debian.org/pkg/memtest86
[2]: https://www.memtest86.com/
[3]: https://tracker.debian.org/pkg/memtest86+
[4]: http://www.memtest.org/
[5]: https://en.wikipedia.org/wiki/Memtest86
[6]: https://www.rodsbooks.com/refind/
[7]: https://www.rodsbooks.com/refind/installing.html
[8]: https://en.wikipedia.org/wiki/EFI_system_partition
[9]: {{< baseurl >}}/post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/
