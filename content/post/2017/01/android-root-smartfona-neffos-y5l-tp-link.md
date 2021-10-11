---
author: Morfik
categories:
- Android
date:    2017-01-11 19:30:26 +0100
lastmod: 2017-01-11 19:30:26 +0100
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
GHissueID: 60
title: 'Android: Root smartfona Neffos Y5L od TP-LINK'
---

Może i ten najtańszy smartfon w ofercie TP-LINK nie może popisać się najmocniejszymi podzespołami
ale w zasadzie ten fakt nie przeszkadza nam, by przeprowadzić na Neffos Y5L (TP801A) proces root.
Ten smartfon ma zbliżony SoC do Neffos Y5, a konkretnie mamy tutaj do czynienia z Snapdragon 210
(MSM8209) od Qualcomm'a. Ten fakt sprawia, że w przypadku Neffos Y5L cały proces uzyskiwania
uprawnień administratora systemu przebiega bardzo podobnie do tego [opisywanego wcześniej dla Neffos
Y5][1]. Dlatego też poniższy artykuł za bardzo się nie różni i w zasadzie został jedynie lekko
przerobiony pod kątem zgodności ze smartfonem Neffos Y5L.

Prostszy sposób na przeprowadzanie procesu root w smartfonach Neffos od TP-LINK z wykorzystaniem
natywnych obrazów TWRP [został opisany w nowym wątku][2].

<!--more-->
## Narzędzia ADB i fastboot

Przede wszystkim, by zabrać się za proces root'owania smartfona Neffos Y5L, musimy przygotować sobie
odpowiednie narzędzia. Zapewnią one nam możliwość rozmawiania z telefonem. Będziemy potrzebować
`adb` ([Android Debug Bridge][3]) oraz `fastboot` . [Proces instalacji tych narzędzi na linux][4],
a konkretnie w dystrybucji Debian, został opisany osobno.

## Problematyczny backup flash'a smartfona Neffos Y5L

W przypadku [Neffos C5][5] i [Neffos C5 MAX][6], do zrobienia backup'u całego flash'a można było
wykorzystać narzędzie SP Flash Tool. Niemniej jednak, to oprogramowanie jest przeznaczone jedynie
dla smartfonów mających SoC od MediaTek, a jak już zostało wspomniane we wstępie, Neffos Y5L ma SoC
Snapdragon 210 (MSM8209) od Qualcomm'a. Jak zatem zrobić backup flash'a tego smartfona przed
wprowadzaniem w nim jakichkolwiek zmian?

Generalnie trzeba się zmierzyć z problemem jajka i kury, czyli by dokonać backup'u flash'a trzeba
skorzystać z niestandardowego obrazu partycji `/recovery/` , np. TWRP, a nie możemy go przecież
wgrać na telefon, bo wprowadzimy w ten sposób zmiany. Możemy jednak wgrać taki obraz bezpośrednio
do pamięci RAM telefonu i z niej go uruchomić. W takim przypadku będziemy w stanie zrobić backup
flash'a telefonu bez wprowadzania żadnych zmian. Niemniej jednak, w dalszym ciągu obraz partycji
`/recovery/` musimy jakoś pozyskać.

## Pozyskanie obrazu recovery.img z TWRP

Niestety, póki co [nie ma obrazów dla Neffos'a Y5L][7]. Dlatego też musimy sobie taki obraz
`recovery.img` stworzyć sami przerabiając inny obraz, który jest przeznaczony na telefon zbliżony
parametrami do naszego urządzenia (ten sam SoC, wielkość flash i rozdzielczość ekranu). Ja
posłużyłem się [obrazem dla Neffos'a C5L][8], którego SoC (MSM8909) jest bardzo podobny do tego
zastosowanego w Neffos Y5L. W zasadzie rozdziałka ekranu i wielkość flash się zgadzają ale układ
partycji jest nieco inny i trzeba będzie ten obraz trochę przerobić, a do tego celu potrzebny nam
będzie stock'owy obraz `boot.img` lub `recovery.img` .

Gotowy [obraz recovery.img dla smartfona Neffos Y5L][9] znajduje się tutaj i jedyne co, to musimy
go wgrać na telefon. Jeśli jednak ktoś jest ciekaw jak proces przepakowania tego obrazu przebiega,
to jest on opisany poniżej.

## Pozyskanie stock'owego obrazu boot/recovery

Proces dostosowania obrazu `recovery.img` z TWRP nieco się różni w przypadku Neffos Y5L w stosunku
do poprzednio opisywanych przez mnie modeli Neffos C5 i Neffos C5 MAX ale jest mniej więcej taki sam
co w przypadku Neffos Y5. Chodzi o to, że w zasadzie nie mamy jak wydobyć obrazu partycji
`/recovery/` z telefonu, no bo przecież nie możemy skorzystać z SP Flash Tool, a póki co nie jestem
świadom alternatywnego oprogramowania, które by nam z tym zadaniem pomogło w podobny sposób.
Niemniej jednak, obraz `recovery.img` w dalszym ciągu możemy zbudować ale potrzebny nam jest
firmware Neffos'a Y5L, który na szczęście możemy pobrać ze [strony producenta tego smartfona][10].
Pamiętajmy by pobrać plik przeznaczony na ten konkretny model telefonu, który posiadamy (w tym
przypadku TP801A). Poniżej jest pełna specyfikacja wgranego oprogramowania oraz dokładne numery
mojego smartfona:

![](/img/2017/01/001.neffos-y5l-smartfon-tp-link-root-informacje-model.png#small)

W paczce `.zip` z firmware, którą pobraliśmy, znajduje się plik `boot.img` . Musimy go wydobyć w
celu wyodrębnienia pewnych plików i wgrania ich na portowany obraz `recovery.img` .

![](/img/2017/01/002.neffos-y5l-smartfon-tp-link-root-obraz-boot.png#huge)

## Przepakowanie obrazu recovery.img

By przepakować obraz przeznaczony na inny smartfon, który jest zbliżony parametrami do naszego
Neffos'a Y5L, musimy pierw pozyskać odpowiednie narzędzia. Na linux'ie możemy skorzystać do tego
celu z [abootimg][11] lub też ze [skryptów Android Image Kitchen][12]. Ja będę korzystał z tego
drugiego rozwiązania.

Tworzymy sobie jakiś katalog roboczy i kopiujemy do niego zarówno oryginalny obraz `boot.img` , jak
i obraz `recovery.img` z innego smartfona. Następnie znajdując się w tym katalogu roboczym,
pobieramy skrypty z github'a (wymagane zainstalowane narzędzie `git` w systemie) i przechodzimy do
utworzonego w ten sposób katalogu. W nim zaś tworzymy dwa podkatalogi `stock/` oraz `port/` :

    $ git clone https://github.com/ndrancs/AIK-Linux-x32-x64/
    $ chmod +x ./AIK-Linux-x32-x64/*
    $ chmod +x ./AIK-Linux-x32-x64/bin/*
    $ cd ./AIK-Linux-x32-x64/
    $ mkdir stock/ port/

Kopiujemy oryginalny obraz `boot.img` z katalogu nadrzędnego i wypakowujemy go za pomocą skryptu
`unpackimg.sh` . Następnie przenosimy tak wyodrębnioną zawartość do katalogu `stock/` :

    $ cp ../orig_boot.img ./recovery.img
    $ ./unpackimg.sh recovery.img
    $ mv split_img/ ramdisk/ stock/
    $ rm recovery.img

Kopiujemy teraz obraz partycji `/recovery/` mający TWRP i wypakowujemy go. Przenosimy jego zawartość
do katalogu `port/` :

    $ cp ../recovery_twrp.img ./recovery.img
    $ ./unpackimg.sh recovery.img
    $ mv split_img/ ramdisk/ port/
    $ rm recovery.img

### Kernel

W zasadzie to musimy tylko przekopiować plik `recovery.img-zImage` z oryginalnego obrazu naszego
Neffos'a Y5L do obrazu TWRP:

    $ cp ./stock/split_img/recovery.img-zImage ./port/split_img/

### Fstab

Musimy także dostosować nieco plik `port/ramdisk/etc/recovery.fstab` , bo flash telefonu, z którego
wzięliśmy obraz `recovery.img` z TWRP ma inny nieco inny układ partycji.

W oparciu o informacje uzyskane z [aplikacji DiskInfo][13] oraz z pliku `/proc/partitions` w
telefonie, układ flash'a w przypadku Neffos Y5L prezentuje się następująco (kolumna najbardziej na
prawo została dodana przeze mnie):

    # adb shell
    shell@Y5L:/ $ cat /proc/partitions
    major minor  #blocks  name

     253        0     524288 zram0
     179        0    7634944 mmcblk0
     179        1      65536 mmcblk0p1      modem (/firmware/ , vfat)
     179        2        512 mmcblk0p2      sbl1
     179        3        512 mmcblk0p3      sbl1bak
     179        4       1024 mmcblk0p4      aboot
     179        5       1024 mmcblk0p5      abootbak
     179        6        512 mmcblk0p6      rpm
     179        7        512 mmcblk0p7      rpmbak
     179        8        768 mmcblk0p8      tz
     179        9        768 mmcblk0p9      tzbak
     179       10       1024 mmcblk0p10     pad
     179       11       1536 mmcblk0p11     modemst1
     179       12       1536 mmcblk0p12     modemst2
     179       13       1024 mmcblk0p13     misc
     179       14          1 mmcblk0p14     fsc
     179       15          8 mmcblk0p15     ssd
     179       16      10240 mmcblk0p16     splash
     179       17         32 mmcblk0p17     DDR
     179       18       1536 mmcblk0p18     fsg
     179       19         16 mmcblk0p19     sec
     179       20      32768 mmcblk0p20     boot
     179       21    1913652 mmcblk0p21     System (/system/ , ext4)
     179       22      32768 mmcblk0p22     persist (/persist/ , ext4)
     179       23     262144 mmcblk0p23     Cache (/cache/ , ext4)
     179       24      32768 mmcblk0p24     recovery
     179       25       1024 mmcblk0p25     devinfo
     179       26        512 mmcblk0p26     keystore
     179       27      65536 mmcblk0p27     oem
     179       28        512 mmcblk0p28     config
     179       29    5077999 mmcblk0p29     Data (/data/ , ext4)
     179       32        512 mmcblk0rpmb    mmcblk0rpmb

Rozmiary poszczególnych partycji są w blokach, a każdy z nich ma 1024 bajty. Partycja `mmcblk0`
odpowiada za cały obszar flash'a. Będziemy zatem w stanie zrobić backup całego flash'a albo też
poszczególnych jego partycji. Tak czy inaczej potrzebne nam są odpowiednie wpisy w pliku
`port/ramdisk/etc/recovery.fstab` . Poniżej jest zawartość mojego pliku:

    # mmcblk0p1 (modem)
    /firmware       vfat      /dev/block/bootdevice/by-name/modem                flags=display="Firmware";backup=1;mounttodecrypt
    # mmcblk0p2 (sbl1)
    /sbl1           emmc      /dev/block/bootdevice/by-name/sbl1                 flags=display="sbl1";backup=1
    # mmcblk0p3 (sbl1bak)
    /sbl1bak        emmc      /dev/block/bootdevice/by-name/sbl1bak              flags=display="sbl1bak";backup=1
    # mmcblk0p4 (aboot)
    /aboot          emmc      /dev/block/bootdevice/by-name/aboot                flags=display="aboot";backup=1
    # mmcblk0p5 (abootbak)
    /abootbak       emmc      /dev/block/bootdevice/by-name/abootbak             flags=display="abootbak";backup=1
    # mmcblk0p6 (rpm)
    /rpm            emmc      /dev/block/bootdevice/by-name/rpm                  flags=display="rpm";backup=1
    # mmcblk0p7 (rpmbak)
    /rpmbak         emmc      /dev/block/bootdevice/by-name/rpmbak               flags=display="rpmbak";backup=1
    # mmcblk0p8 (tz)
    /tz             emmc      /dev/block/bootdevice/by-name/tz                   flags=display="tz";backup=1
    # mmcblk0p9 (tzbak)
    /tzbak          emmc      /dev/block/bootdevice/by-name/tzbak                flags=display="tzbak";backup=1
    # mmcblk0p10 (tzbak)
    /pad            emmc      /dev/block/bootdevice/by-name/pad                  flags=display="pad";backup=1
    # mmcblk0p11 (modemst1)
    /efs1           emmc      /dev/block/bootdevice/by-name/modemst1             flags=display="Modemst1";backup=1
    # mmcblk0p12 (modemst2)
    /efs2           emmc      /dev/block/bootdevice/by-name/modemst2             flags=display="Modemst2";backup=1
    # mmcblk0p13 (misc)
    /misc           emmc      /dev/block/bootdevice/by-name/misc                 flags=display="Misc";backup=1
    # mmcblk0p14 (fsc)
    /fsc            emmc      /dev/block/bootdevice/by-name/fsc                  flags=display="fsc";backup=1
    # mmcblk0p15 (ssd)
    /ssd            emmc      /dev/block/bootdevice/by-name/ssd                  flags=display="ssd";backup=1
    # mmcblk0p16 (splash)
    /splash         emmc      /dev/block/bootdevice/by-name/splash               flags=display="splash";backup=1
    # mmcblk0p17 (DDR)
    /ddr            emmc      /dev/block/bootdevice/by-name/DDR                  flags=display="DDR";backup=1
    # mmcblk0p18 (fsg)
    /fsg            emmc      /dev/block/bootdevice/by-name/fsg                  flags=display="fsg";backup=1
    # mmcblk0p19 (sec)
    /sec            emmc      /dev/block/bootdevice/by-name/sec                  flags=display="sec";backup=1
    # mmcblk0p20 (boot)
    /boot           emmc      /dev/block/bootdevice/by-name/boot                 flags=display="Boot";backup=1
    # mmcblk0p21 (System)
    /system         ext4      /dev/block/bootdevice/by-name/system               flags=display="System";backup=1;wipeingui;mounttodecrypt
    # mmcblk0p22 (persist)
    /persist        ext4      /dev/block/bootdevice/by-name/persist              flags=display="Persist";backup=1
    # mmcblk0p23 (Cache)
    /cache          ext4      /dev/block/bootdevice/by-name/cache                flags=display="Cache";backup=1;wipeingui;wipeduringfactoryreset
    # mmcblk0p24 (recovery)
    /recovery       emmc      /dev/block/bootdevice/by-name/recovery             flags=display="Recovery";backup=1
    # mmcblk0p25 (devinfo)
    /devinfo       emmc       /dev/block/bootdevice/by-name/devinfo              flags=display="devinfo";backup=1
    # mmcblk0p26 (keystore)
    /keystore       emmc      /dev/block/bootdevice/by-name/keystore             flags=display="keystore";backup=1
    # mmcblk0p27 (oem)
    /oem            emmc      /dev/block/bootdevice/by-name/oem                  flags=display="oem";backup=1
    # mmcblk0p28 (config)
    /config         emmc      /dev/block/bootdevice/by-name/config               flags=display="config";backup=1
    # mmcblk0p29 (Data)
    /data           ext4      /dev/block/bootdevice/by-name/userdata             flags=display="Data";backup=1;wipeingui;wipeduringfactoryreset;encryptable=footer;length=-16384

    #
    #/mmcblk0rpmb    emmc      /dev/block/bootdevice/mmcblk0rpmb                 flags=display="mmcblk0rpmb";backup=1

    # External
    /sdcard1        auto      /dev/block/mmcblk1p1                               flags=display="MicroSD";storage;wipeingui;removable
    #/usb-otg        auto      /dev/block/sda1                                   flags=display="USBOTG";storage;wipeingui;removable

    # Full partition images
    /firmware_image emmc      /dev/block/bootdevice/by-name/modem                flags=display="Firmware-Image";backup=1
    /system_image   emmc      /dev/block/bootdevice/by-name/system               flags=display="System-Image";backup=1
    /persist_image  emmc      /dev/block/bootdevice/by-name/persist              flags=display="Persist-Image";backup=1
    /cache_image    emmc      /dev/block/bootdevice/by-name/cache                flags=display="Cache-Image";backup=1
    /data_image     emmc      /dev/block/bootdevice/by-name/userdata             flags=display="Data-Image";backup=1
    /full_flash     emmc      /dev/block/mmcblk0                                 flags=display="Full-Flash-Image";backup=1

Jeśli ktoś jest ciekaw użytych tutaj opcji, to są one wyjaśnione [w tym wątku na forum XDA][14].

## Tworzenie obrazu recovery z TWRP dla Neffos Y5L

Z tak przygotowanych plików w katalogu `stock/` trzeba zrobić nowy obraz `recovery.img` przy pomocy
skryptu `repackimg_x64.sh` :

    $ rm -R stock/
    $ mv port/ramdisk ./
    $ mv port/split_img ./
    $ rmdir port/
    $ ./repackimg_x64.sh

W katalogu roboczym powinien zostać utworzony nowy plik o nazwie `image-new.img` . To jest właśnie
nasz nowy obraz partycji `/recovery/` , który musimy wgrać na telefon w trybie bootloader'a przez
fastboot. Niemniej jednak, zanim będziemy w stanie to zrobić, musimy odblokować bootloader.

## Jak odblokować bootloader w Neffos Y5L

Może nie mamy możliwości zrobić backup'u całego flash'a telefonu przed podjęciem jakichkolwiek prac
ale też raczej nie powinniśmy znowu nic namieszać. Jedyna rzecz jaką musimy zrobić, to odblokować
bootloader. Chodzi o to, że na smartfonach zwykle jest ulokowana partycja `/recovery/` . Na niej
znajduje się oprogramowanie umożliwiające przeprowadzanie niskopoziomowych operacji, np. backup lub
też flash'owanie ROM'u. Problem w tym, że to oprogramowanie w standardzie zwykle za wiele nie
potrafi i by przeprowadzić proces root'owania Androida, musimy pozyskać bardziej zaawansowany soft,
np. [ClockworkMod][15] czy [TWRP][16], i wgrać go na partycję `/recovery/` . By to zrobić musimy
pierw odblokować bootloader.

Proces odblokowania bootloader'a usuwa wszystkie dane, które wgraliśmy na flash telefonu, tj.
podczas odblokowywania jest inicjowany [factory reset][17]. Dane na karcie SD pozostają nietknięte.
By ten proces zainicjować zaczynamy od przestawienia jednej opcji w telefonie. W tym celu musimy
udać się w Ustawienia => Opcje Programistyczne i tam przełączyć `Zdjęcie blokady OEM` :

![](/img/2017/01/003.neffos-y5l-smartfon-tp-link-root-blokada-booloader.png#huge)

Następnie w terminalu wpisujemy poniższe polecenia:

    # adb reboot bootloader
    # fastboot devices
    8a8f289 fastboot

    # fastboot oem unlock-go

Na ekranie smartfona powinien nam się pokazać zielony robocik informujący o przeprowadzaniu Factory
Reset. Po chwili ten proces powinien dobiec końca, a smartfon uruchomi się ponownie na ustawieniach
domyślnych. Wyłączamy urządzenie i włączamy je via przyciski VolumeDown + Power i sprawdzamy status
blokady bootloader'a:

    # fastboot oem device-info
    ...
    (bootloader)    Device tampered: false
    (bootloader)    Device unlocked: true
    (bootloader)    Charger screen enabled: true
    (bootloader)    Display panel:
    OKAY [  0.004s]
    finished. total time: 0.004s

Jeśli przy `Device unlocked:` widnieje wartość `true` , to blokada bootloader'a została pomyślnie
zdjęta. Jako, że proces odblokowania bootloader'a usunął wszelkie ustawienia, to jeszcze raz musimy
włączyć Opcje programistyczne, a w nich tryb debugowania portu USB.

## Testowanie przepakowanego obrazu recovery.img

Zanim jednak wgramy nowo stworzony obraz `recovery.img` , przydałoby się sprawdzić pierw, czy aby na
pewno ten obraz działa jak należy. Podpinamy telefon do portu USB komputera i przy pomocy narzędzia
`fasboot` przetestujmy wyżej wygenerowany obraz próbując uruchomić go z pamięci telefonu:

    # fastboot boot image-new.img

W przypadku, gdyby pojawiła nam się informacja `FAILED (remote: unlock device to use this
command)` , to prawdopodobnie zapomnieliśmy odblokować bootloader. Jeśli blokada została zdjęta, to
wydanie tego powyższego polecenia powinno załadować do pamięci RAM telefonu zmieniony obraz partycji
`/recovery/` , oczywiście o ile obraz jest poprawny. Jeśli zamiast tego smartfon uruchomi się
ponownie, to coś z takim obrazem jest nie tak i lepiej nie wgrywać go na telefon.

## Jak przeprowadzić backup flash'a Neffos Y5L

Mając załadowany obraz `recovery.img` z TWRP do pamięci smartfona, możemy przejść do zrobienia
backup'u całego flash'a telefonu. Opcje wyboru partycji, które będziemy mieć do uwzględnienia w
backup'ie, zależą od pliku `recovery.fstab` , który edytowaliśmy sobie wcześniej. W tym przypadku
mamy możliwość zrobienia backup'u całego flash'a, jak i jego poszczególnych partycji. Nie musimy
jednak robić backup'u wszystkich partycji i możemy zdecydować się jedynie na niektóre z nich.

Przede wszystkim, potrzebny nam będzie backup partycji `/system/` , `/boot/` i `/recovery/` , bo to
je zwykle będziemy poddawać edycji i wprowadzać w nich zmiany. Ja jednak wolę zrobić backup
pozostałej części flash'a, tak na wszelki wypadek. No i skoro mam do zrobienia praktycznie backup
całej pamięci flash, to można przecież upchnąć go w jednym pliku zrzucając zawartość urządzenia
`/dev/block/mmcblk0` . Można oczywiście zapisać sobie każdą partycję do osobnego pliku ale przecie z
obrazu całego flash'a również można te poszczególne partycje wydobyć.

W zasadzie cały backup zajmie około 2 GiB, chyba, że zrobiliśmy sobie pełną kopię. W przypadku tego
drugiego rozwiązania potrzeba nam będzie karta SD o pojemności większej lub równej pojemności
flash'a w telefonie. Dodatkowo, jako że z reguły flash w smartfonach ma pojemność większą niż 4 GiB
(zwykle 16-32 GiB), to w takim przypadku karta musi zostać sformatowana innym systemem plików niż
FAT, bo ten ma ograniczenia wielkości pliku do 4 GiB, a obraz będzie przecie zajmował tyle ile
zajmuje pamięć flash. TWRP obsługuje bez większego problemu karty SD sformatowane jako EXT4 i z tego
systemu plików możemy skorzystać. Pamiętajmy jednak, że takiej karty Android nam nie będzie czytał
standardowo.

Niepełny backup z kolei można przeprowadzić zapisując go na flash'u smartfona, choć nie zaleca się
tego robić, a to z tego względu, że kopia pamięci danego urządzenia powinna być zapisywana na
zewnętrznym nośniku. Dlatego lepiej zakupić sobie kartę SD rozmiarem przypominającą flash telefonu.

Różnica między robieniem obrazów partycji EXT4 i EMMC polega na tym, że w przypadku standardowych
partycji EMMC, ich obraz można zamontować przez `mount` na dowolnym linux'ie. Natomiast obrazy EXT4,
są w zasadzie zwykłymi archiwami plików, które można wypakować jak zwykłego ZIP'a. Druga różnica
jest taka, że te spakowane paczki są dzielone na kawałki o rozmiarze 1,5 GiB, przez co można je bez
problemu zapisywać na karcie SD, która ma system plików FAT.

Warto w tym miejscu jeszcze dodać, że można pominąć backup partycji `/cache/` i `/data/` , bo one są
i tak czyszczone podczas procesu Factory Reset. Jeśli zaś chcemy dokonać backup'u danych
użytkownika, tj. partycji `/data/` , to w jej przypadku lepiej jest spakować pliki zamiast robić
backup całej partycji, bo wtedy robimy backup tylko danych i nie wchodzi w to wolne miejsce.

Jak już ustalimy jakie partycje uwzględnimy w backup'ie, to przechodzimy do pozycji Backup i
wybieramy kartę SD oraz zaznaczamy odpowiednie obszary pamięci flash, tak jak to widać na poniższej
fotce:

![](/img/2017/01/004.neffos-y5l-smartfon-tp-link-root-backup-flash.png#huge)

W przypadku robienia pełnego backup'u, cały proces może zająć dłuższą chwilę. Po jego ukończeniu, na
karcie SD pojawi się obraz flash'a, który możemy sprawdzić w `gdisk` lub `parted` :

![](/img/2017/01/005.neffos-y5l-smartfon-tp-link-root-flash-gdisk.png#huge)

![](/img/2017/01/006.neffos-y5l-smartfon-tp-link-root-flash-parted.png#huge)

## Wgranie obrazu recovery z TWRP na Neffos Y5L

Po sprawdzeniu czy obraz się boot'uje poprawnie i dokonaniu backup'u określonych obszarów pamięci
flash, możemy ten obraz wgrać na telefon lub też możemy zainstalować jedynie samo SuperSU. Ja
postanowiłem wgrać TWRP recovery na mojego Neffos'a Y5L. W sumie ta procedura się za wiele nie różni
od testowania samego obrazu w pamięci telefonu. Jedyne co trzeba zrobić to zrestartować telefon do
trybu bootloader'a i wgrać obraz recovery przy pomocy `fastboot` w poniższy sposób:

    # adb reboot bootloader
    # fastboot flash recovery image-new.img
    # fastboot reboot

## SuperSU, BusyBOX, RootCheck i emulator terminala

Ostatnią rzeczą na drodze do zrobienia root na Neffos Y5L jest wgranie aplikacji umożliwiającej
korzystanie różnym programom z praw administratora systemu w telefonie. My skorzystamy z SuperSU.
Dodatkowo, musimy wgrać sobie BusyBOX, który zawiera minimalistyczne odpowiedniki linux'owych
narzędzi, co pozwoli nam swobodnie operować w Androidzie, tak jakbyśmy to robili pod
pełnowymiarowym pingwinem. Potrzebny nam także będzie jakiś emulator terminala, w którym to
będziemy wpisywać wszystkie nasze polecenia.

### Instalacja SuperSU

Zacznijmy od pobrania stosownej paczki z [SuperSU][18]. Jako, że my nie mamy jeszcze zrobionego
root'a, to musimy pobrać `TWRP / FlashFire installable ZIP` . Tej paczki nie wypakowujemy, tylko
wrzucamy ją w pobranej formie na kartę SD w telefonie. Odpalamy teraz tryb recovery w smartfonie
(VolumeUp + Power) i przechodzimy do Install i wskazujemy paczkę `.zip` , którą umieściliśmy na
karcie SD. Tam z kolei zaznaczamy `ZIP signature verification` i przeciągamy trzy strzałki na prawą
stronę.

![](/img/2017/01/007.neffos-y5l-smartfon-tp-link-root-supersu-instalacja.png#huge)

Teraz możemy uruchomić ponownie Neffos'a Y5L i zainstalować jakąś aplikację, która pokaże nam czy
nasz smartfon ma root'a.

### Sprawdzenie czy Neffos Y5L ma root'a

Po uruchomieniu się systemu na smartfonie, instalujemy aplikację [RootCheck][19], po czym
uruchamiamy ją. Powinien się pojawić monit informujący, że ta aplikacja żąda praw administracyjnych,
na co zezwalamy. Jeśli nasz telefon ma root'a, to powinien się pojawić stosowny komunikat:

![](/img/2017/01/008.neffos-y5l-smartfon-tp-link-root-checkroot.png#big)

### Instalacja BusyBOX

Kolejnym krokiem jest instalacja [BusyBOX'a][20]. Po wgraniu tej aplikacji na smartfona, musimy ją
uruchomić i wcisnąć w niej przycisk `install` . BusyBOX również nas poprosi o dostęp do praw
administracyjnych. Po zainstalowaniu, weryfikujemy jeszcze, czy aby wszystko zostało pomyślne
wgrane. Możemy to zrobić zarówno w samej aplikacji BusyBOX, jak w CheckRoot:

![](/img/2017/01/009.neffos-y5l-smartfon-tp-link-root-busybox.png#huge)

### Instalacja terminala

Generalnie rzecz biorąc, terminal jako taki nie jest obowiązkowy, bo SuperSU jak i BusyBOX są
wymagane przez konkretne aplikacje do poprawnego ich działania. Niemniej jednak, jeśli zamierzamy
korzystać z tych niskopoziomowych narzędzi dostarczonych przez BusyBOX, czy też innych narzędzi
obecnych standardowo w Androidzie na uprawnieniach root, to terminal jak najbardziej się nam przyda.

Znalazłem dwa terminale, które są OpenSource i bez reklam/opłat. Są to
[Android-Terminal-Emulator][21] oraz [Termux][22]. Wybieramy sobie jeden z nich i instalujemy w
systemie. Jako, że ja korzystam na co dzień z Debiana, to instaluję Termux'a.

### Aplikacje i prawa administracyjne

Teraz już pozostało nam tylko odpalenie terminala i zalogowanie się na użytkownika root. Do tego
celu służy polecenie `su` . Wpiszmy je zatem w okienku Termux'a:

![](/img/2017/01/010.neffos-y5l-smartfon-tp-link-root-termux-su.png#huge)

I teraz możemy uruchamiać aplikacje z prawami admina, tak jak to zwykliśmy robić w każdym innym
linux'ie. Pamiętajmy tylko, że standardowo system plików jest zamontowany w trybie tylko do odczytu
(RO) i by móc zmieniać pliki systemowe z poziomu tego terminala, musimy przemontować system plików w
tryb do zapisu (RW). Robimy to w poniższy sposób:

    $ su
    # mount -o remount,rw /system

Gdy skończymy się bawić, to montujemy ten system plików ponownie w tryb RO:

    # mount -o remount,ro /system


[1]: /post/android-root-smartfona-neffos-y5-od-tp-link/
[2]: /post/root-w-smartfonach-neffos-od-tp-link-x1-c5-c5-max-y5-y5l/
[3]: https://developer.android.com/studio/command-line/adb.html
[4]: /post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/
[5]: /post/android-root-smartfona-neffos-c5-max-od-tp-link/
[6]: /post/android-root-smartfona-neffos-c5-max-od-tp-link/
[7]: https://twrp.me/Devices/
[8]: http://4pda.ru/forum/index.php?showtopic=738058&st=80#entry52106796
[9]: /img/manual/recovery-neffos-y5l-tp-link-twrp.img
[10]: http://www.neffos.com/en/support/download/Y5L
[11]: https://github.com/codeworkx/abootimg
[12]: https://github.com/ndrancs/AIK-Linux-x32-x64/
[13]: https://play.google.com/store/apps/details?id=me.kuder.diskinfo&hl=pl
[14]: https://forum.xda-developers.com/showthread.php?t=1943625
[15]: https://www.clockworkmod.com/
[16]: https://twrp.me/
[17]: /post/android-reset-ustawien-do-fabrycznych-factory-defaults/
[18]: https://forum.xda-developers.com/apps/supersu/stable-2016-09-01supersu-v2-78-release-t3452703
[19]: https://play.google.com/store/apps/details?id=com.jrummyapps.rootchecker
[20]: https://play.google.com/store/apps/details?id=stericson.busybox
[21]: https://play.google.com/store/apps/details?id=jackpal.androidterm
[22]: https://play.google.com/store/apps/details?id=com.termux
