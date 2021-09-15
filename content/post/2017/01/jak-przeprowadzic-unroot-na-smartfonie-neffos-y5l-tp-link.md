---
author: Morfik
categories:
- Android
date:    2017-01-12 20:07:33 +0100
lastmod: 2017-01-12 20:07:33 +0100
published: true
status: publish
tags:
- tp-link
- smartfon
- root
- neffos
- neffos-y5l
- adb
- fastboot
- twrp
title: Jak przeprowadzić unroot na smartfonie Neffos Y5L od TP-LINK
---

Przeprowadzenie [procesu root na smartfonie Neffos Y5L][1] od TP-LINK nie było tak łatwe jak w
przypadku innych modeli telefonów tego producenta. Niemniej jednak, trzeba zdawać sobie sprawę, że
ukorzenianie Androida niesie za sobą pewne zagrożenia. Nie chodzi tutaj tylko o niezaufane
aplikacje ale też trzeba brać pod uwagę możliwość przypadkowego (przypadki nie istnieją) skasowania
czy zmienienia plików systemowych, przez co nasz telefon może przestać nam działać poprawnie lub
też przestanie się w ogóle uruchamiać. Jeśli natomiast wgraliśmy SuperSU i praktycznie w ogóle z
niego nie korzystamy, to moim zdaniem lepiej jest przeprowadzić proces unroot i korzystać z
Neffos'a Y5L, tak jak ze zwykłego urządzenia z Androidem na pokładzie. Proces cofania zmian w
systemie nie jest jakoś specjalnie trudny ale trzeba uważać, by w jego trakcie nie uszkodzić
smartfona. Ten artykuł ma na celu pokazanie jak cofnąć wszelkie zmiany wprowadzone w telefonie za
sprawą dostępu do praw administracyjnych w Neffos Y5L.

<!--more-->
## Odinstalowanie SuperSU (unroot)

Zmiany wprowadzane w systemie za sprawą ukorzenionego Androida mogą być niewielkie lub też mogą dość
znacznie ingerować w jego struktury. W zasadzie unroot przeprowadzany z poziomu SuperSU działa OOTB.
Trzeba tutaj jednak wyraźnie zaznaczyć, że SuperSU nie usunie nam zmian wprowadzonych przez inne
aplikacje wymagające praw administratora root. SuperSU jest w zasadzie zdolny odinstalować sam
siebie oraz (opcjonalnie) przywrócić partycję `/recovery/` do stanu fabrycznego.

W przypadku, gdy nie chcemy zbytnio powracać do standardowego ROM'u, a jedynie nieco zabezpieczyć
nasz smartfon przez uniemożliwienie logowania się aplikacjom na użytkownika root, to możemy w
zasadzie odinstalować samo SuperSU. Chodzi generalnie o to, że wprowadzone przez nas zmiany na
partycji `/system/` , do przeprowadzenia których potrzebny nam był SuperSU, i tak przetrwają
odinstalowanie tego programiku. Po skonfigurowaniu Androida, SuperSU jest nam zwyczajnie zbędny i
stwarza on tylko niepotrzebne zagrożenie dla bezpieczeństwa systemu.

SuperSU w Neffos Y5L możemy odinstalować z menu tejże aplikacji przechodząc w Ustawienia => Pełny
Unroot.

![](/img/2017/01/001.unroot-neffos-y5l-tp-link-smartfon-supersu.png#huge)

Jeśli nie chcemy przywracać partycji `/recovery/` , to w ostatnim kroku wybieramy opcję NIE. Jeśli
smartfon nie uruchomi się ponownie automatycznie, to naturalnie po całym procesie telefon
restartujemy ręcznie. W przypadku, gdy proces usuwania SuperSU się zawiesi nam, to trzeba
zrestartować smartfon i ponowić proces unroot bezpośrednio po włączeniu telefonu.

Możemy naturalnie sprawdzić czy cały proces przebiegł zgodnie z planem i czy nasz Neffos Y5L w
dalszym ciągu posiada root:

![](/img/2017/01/002.unroot-neffos-y5l-tp-link-smartfon-root-check.png#small)

Niemniej jednak, jeśli w Neffos Y5L chcemy przywrócić całą partycję `/system/` usuwając tym samym
wszelkie zmiany wprowadzone w telefonie, to trzeba do tej kwestii podejść nieco inaczej.

## Wydobywanie partycji /system/ , /recovery/ i /boot/ z obrazu flash'a

W zasadzie mając zrobiony pełny obraz flash'a smartfona Neffos Y5L możemy wydobyć z niego określone
partycje via `dd` i wgrać je w stosowne miejsca przez bootloader za pomocą fastboot. Zamontujmy
zatem ten obraz backup'u w systemie (za pomocą `losetup` ) i sprawdźmy jak wygląda jego layout, np.
w `gdisk` :

![](/img/2017/01/003.unroot-neffos-y5l-tp-link-smartfon-uklad-flash.png#huge)

Interesują nas partycje 21 ( `/system/` ), 24 ( `/recovery/` ) oraz 20 ( `/boot/` ). Robimy ich
zrzut do osobnych plików via `dd` :

    # losetup /dev/loop0 '/smartfon/Neffos Y5L/full_flash.emmc.win'
    # dd if=/dev/loop0p21 of=./neffos_y5l_orig_system.img
    # dd if=/dev/loop0p24 of=./neffos_y5l_orig_recovery.img
    # dd if=/dev/loop0p20 of=./neffos_y5l_orig_boot.img

## Przywracanie partycji /system/ via fastboot

Przywrócenie partycji `/system/` w trybie bootloader'a za pomocą narzędzia `fastboot` nie odtworzy
automatycznie nam partycji `/recovery/` . Musimy ją wgrać osobno. Na początek wgrajmy obraz partycji
`/system/` . Wyłączamy telefon i włączamy go ponownie trzymając przyciski VolumeDown + Power
(powinno pojawić się logo TP-LINK'a i Android'a). Następnie podpinamy telefon do portu USB komputera
i sprawdzamy, czy narzędzie `fastboot` jest w stanie wykryć nasz smartfon:

    # fastboot devices
    8a8f289 fastboot

Wgrywamy teraz wydobyty wcześniej obraz `neffos_y5l_orig_system.img` na smartfon przy pomocy
poniższego polecenia:

    # fastboot flash system neffos_y5l_orig_system.img
    target reported max download size of 262144000 bytes
    Invalid sparse file format at header magi
    erasing 'system'...
    OKAY [  1.532s]
    sending sparse 'system' (241786 KB)...
    OKAY [ 10.892s]
    writing 'system'...
    OKAY [ 24.863s]
    sending sparse 'system' (232980 KB)...
    OKAY [ 10.580s]
    writing 'system'...
    OKAY [ 27.701s]
    ...
    sending sparse 'system' (185348 KB)...
    OKAY [  8.556s]
    writing 'system'...
    OKAY [ 22.117s]
    finished. total time: 308.857s

Partycja `/system/` przed wgraniem nowego obrazu została pierw wyczyszczona. Następnie obraz tejże
partycji został podzielony na kawałki i przesłany na telefon, no i oczywiście wszystkie części
obrazu zostały pomyślnie wgrane na flash smartfona.

## Czyszczenie partycji /data/ i /cache/

Jako, że wgraliśmy świeży obraz partycji `/system/` , to przydałoby się także wyczyścić dane
użytkownika znajdujące się na partycji `/data/` :

    # fastboot format userdata
    Creating filesystem with parameters:
        Size: 5199867904
        Block size: 4096
        Blocks per group: 32768
        Inodes per group: 8144
        Inode size: 256
        Journal blocks: 19835
        Label:
        Blocks: 1269499
        Block groups: 39
        Reserved block group size: 311
    Created filesystem with 11/317616 inodes and 42271/1269499 blocks
    target reported max download size of 262144000 bytes
    erasing 'userdata'...
    OKAY [  3.013s]
    sending 'userdata' (83009 KB)...
    OKAY [  3.543s]
    writing 'userdata'...
    OKAY [  6.071s]
    finished. total time: 12.627s

Dobrze jest także wyczyścić dane znajdujące się w cache:

    # fastboot format cache
    Creating filesystem with parameters:
        Size: 268435456
        Block size: 4096
        Blocks per group: 32768
        Inodes per group: 8192
        Inode size: 256
        Journal blocks: 1024
        Label:
        Blocks: 65536
        Block groups: 2
        Reserved block group size: 15
    Created filesystem with 11/16384 inodes and 2089/65536 blocks
    target reported max download size of 262144000 bytes
    erasing 'cache'...
    OKAY [  0.190s]
    sending 'cache' (6248 KB)...
    OKAY [  0.278s]
    writing 'cache'...
    OKAY [  0.936s]
    finished. total time: 1.405s

Warto tutaj dodać, by nie korzystać z opcji `erase` w miejscu `format` . W przypadku `erase` system
podczas ponownego startu złapie nam bootloop'a i trzeba będzie ponawiać czyszczenie partycji
`/data/` i `/cache/` z wykorzystaniem `format` . Opcja `erase` przydaje się jedynie przed ponownym
flash'owaniem.

## Przywracanie partycji /recovery/ i /boot/ via fastboot

Podobnie jak w przypadku partycji `/system/` , partycje `/recovery/` i `/boot/` również przywracamy
z poziomu bootloader'a za pomocą narzędzia `fastboot` . Ten proces się zbytnio wcale nie różni,
tylko wymaga wskazania pozostałych obrazów i wgrania ich na odpowiednie partycje:

    # fastboot flash recovery neffos_y5l_orig_recovery.img
    # fastboot flash boot neffos_y5l_orig_boot.img

## Ponowny restart Neffos'a Y5L

Po wgraniu świeżego obrazu na partycję `/system/` , `/recovery/` i `/boot/` oraz wyczyszczeniu
partycji `/data/` i `/cache/` , możemy zresetować naszego Neffos'a Y5L i sprawdzić czy się uruchomi
on ponownie. Wpisujemy zatem w terminal poniższe polecenie:

    # fastboot reboot

Neffos Y5L powinien się uruchomić bez większych problemów, o ile przeprowadziliśmy powyższe kroki
tak jak trzeba. Proces pierwszego startu zajmie dłuższą chwilę ale ostatecznie powinniśmy zobaczyć
znajomy nam wszystkim ekran pierwszego logowania z wyborem języka systemu.

## Zablokowanie bootloader'a w Neffos Y5L

To jednak nie jest koniec i w zasadzie możemy sobie darować konfigurację telefonu w tej fazie. A to
z tego względu, że bootloader w dalszym ciągu jest odblokowany. Jeśli teraz byśmy skonfigurowali
wstępnie system, to po zablokowaniu bootloader'a ponownie będziemy musieli wszystko ustawiać.
Dlatego też wyłączamy telefon i uruchamiamy go w trybie bootloader'a za pomocą przycisków
VolumeDown + Power. Następnie w terminalu wpisujemy poniższe polecenie:

    # fastboot oem lock

Nasz Neffos Y5L powinien nam się uruchomić ponownie, a na jego ekranie powinniśmy zobaczyć zielonego
robocika przeprowadzającego proces Factory Reset. Po chwili smartfon uruchomi się ponownie, a po
jeszcze dłuższej chwili system powinien się załadować już na fabrycznych
ustawieniach:

![](/img/2017/01/001.unroot-neffos-y5l-tp-link-smartfon-defaults.png#small)


[1]: /post/android-root-smartfona-neffos-y5l-tp-link/
