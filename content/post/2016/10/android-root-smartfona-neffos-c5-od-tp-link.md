---
author: Morfik
categories:
- Android
date: "2016-10-09T18:08:36Z"
date_gmt: 2016-10-09 16:08:36 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- lollipop
- root
- twrp
- neffos
title: 'Android: Root smartfona Neffos C5 od TP-LINK'
---

Smartfony mają to do siebie, że ogromna większość z nich pracuje pod kontrolą systemu linux, a
konkretnie jest to jakiś Android. Tak też jest w przypadku [Neffos'a
C5](http://www.neffos.pl/product/details/C5) od TP-LINK, gdzie mamy zainstalowaną wersję 5.1
(Lollipop). My linux'iarze chcemy mieć pełny dostęp do systemu operacyjnego, by bez większych
przeszkód móc zarządzać urządzeniem, które pod jego kontrolą pracuje. Problem w tym, że ten Neffos
C5 nie ma w standardzie root'a i nie mamy administracyjnego dostępu do całego systemu plików
telefonu. Jest kilka metod root'owania smartfona, np. za pomocą Kingoroot/Kingroot ale nie działają
one w przypadku tego telefonu (i całe szczęście). W tym artykule zostanie pokazany sposób na root
systemu Neffos'a C5 przy zachowaniu wszelkich norm bezpieczeństwa, które w sytuacjach podbramkowych
pomogą nam odzyskać kontrolę nad telefonem.

Prostszy sposób na przeprowadzanie procesu root w smartfonach Neffos od TP-LINK z wykorzystaniem
natywnych obrazów TWRP [został opisany w nowym wątku]({{< baseurl >}}/post/root-w-smartfonach-neffos-od-tp-link-x1-c5-c5-max-y5-y5l/).

<!--more-->
## Narzędzia ADB i fastboot

Przede wszystkim, by zabrać się za proces root'owania smartfona Neffos C5, musimy przygotować sobie
odpowiednie narzędzia. Zapewnią one nam możliwość rozmawiania z telefonem. Będziemy potrzebować
`adb` ([Android Debug Bridge](https://developer.android.com/studio/command-line/adb.html)) oraz
`fastboot` . [Proces instalacji tych narzędzi na
linux]({{< baseurl >}}/post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/), a konkretnie w
dystrybucji Debian, został opisany osobno.

## Narzędzie SP Flash Tool

Kolejnym narzędziem, które będzie nam niezbędne jest [SP Flash Tool](http://spflashtool.com/).
Niestety nie jest ono włączone do dystrybucji Debian i musimy posiłkować się paczką, którą można
znaleźć w podanym wyżej linku. Tutaj ważna uwaga. SP FLash Tool jest przeznaczony tylko dla
smartfonów mających SoC od Mediatek.

Pobieramy paczkę `.zip` dla linux'a i wypakowujemy ją. Jako, że SP Flash Tool wykorzystuje do
komunikacji interfejs `/dev/ttyACM0` , to do poprawnej pracy wymaga on operowania na tym
interfejsie. Standardowo tylko administrator systemu oraz członkowie grupy `dialout` są w stanie
korzystać z tego interfejsu. Musimy zatem dodać naszego użytkownika do tej grupy w poniższy sposób:

    # gpasswd -a morfik dialout

## Kompletny backup flash'a Neffos C5

Mając zainstalowane te powyższe narzędzia, możemy przejść do zrobienia backup'u całego flash'a,
który jest w naszym smartfonie. Proces backup'u najprościej przeprowadzić za pomocą SP Flash Tool.
Niemniej jednak, potrzebne nam są pewne informacje, które możemy uzyskać przy pomocy `adb` .
Podpinamy zatem telefon do portu USB komputera i w terminalu wpisujemy poniższe polecenie:

    # adb shell cat /proc/partinfo
    Name             Start                  Size
    pgpt             0x0000000000000000     0x0000000000080000
    proinfo          0x0000000000080000     0x0000000000300000
    nvram            0x0000000000380000     0x0000000000500000
    protect1         0x0000000000880000     0x0000000000a00000
    protect2         0x0000000001280000     0x0000000000a00000
    lk               0x0000000001c80000     0x0000000000080000
    para             0x0000000001d00000     0x0000000000080000
    boot             0x0000000001d80000     0x0000000001000000
    recovery         0x0000000002d80000     0x0000000001000000
    logo             0x0000000003d80000     0x0000000000800000
    expdb            0x0000000004580000     0x0000000000a00000
    seccfg           0x0000000004f80000     0x0000000000080000
    oemkeystore      0x0000000005000000     0x0000000000200000
    secro            0x0000000005200000     0x0000000000600000
    keystore         0x0000000005800000     0x0000000000800000
    tee1             0x0000000006000000     0x0000000000500000
    tee2             0x0000000006500000     0x0000000000500000
    frp              0x0000000006a00000     0x0000000000100000
    nvdata           0x0000000006b00000     0x0000000002000000
    metadata         0x0000000008b00000     0x0000000002500000
    system           0x000000000b000000     0x0000000100000000
    cache            0x000000010b000000     0x0000000019000000
    userdata         0x0000000124000000     0x000000027ed80000
    flashinfo        0x00000003a2d80000     0x0000000001000000
    sgpt             0x00000003a3d80000     0x0000000000080000

Ten zwrócony wyżej wynik jest dla smartfona Neffos C5. W przypadku innych telefonów, ta tabelka może
mieć inną postać i nie możemy kopiować z niej wartości jeśli mamy inne urządzenie. Generalnie rzecz
biorąc, to te dane potrzebne nam są do zbudowania pliku `scatter.txt` , w oparciu o który działa SP
Flash Tool. Ten plik to zwyczajna mapa przestrzeni flash'a telefonu, który podzielony jest na szereg
widocznych wyżej partycji.

### Plik scatter.txt dla Neffos C5

[Tutaj znajduje się plik
scatter.txt]({{< baseurl >}}/img/manual/mt6735-neffos-c5-tp-link-scatter.txt),
który ja wykorzystałem do pracy z Neffos C5. Kluczowa sprawa, to opisanie każdej partycji. W sumie
to musimy odpowiednio dostosować pole `partition_index` , które jest zwyczajnie kolejnym numerkiem.
Z kolei w `partition_name` podajemy nazwę partycji, którą uzyskaliśmy przez `adb` . Dalej w
`linear_start_addr` oraz `physical_start_addr` wpisujemy tę wartość, która została wypisana przez
`adb` w kolumnie `Start` . Na podobnej zasadzie uzupełniamy `partition_size` , podając wartość,
którą widzieliśmy w `adb` w kolumnie `Size` . I to w zasadzie wszystkie zmiany, które musimy
wprowadzić do pliku `scatter.txt` . Póki co nie mam informacji co do pozostałych opcji w tym pliku,
wiem tylko, że część z nich jest uzupełniana przez SP Flash Tool podczas przeprowadzania działań w
tym programie.

### Tworzenie backupu

Mając plik `scatter.txt` możemy go wskazać w SP Flash Tool. Przechodzimy zatem do katalogu z
wypakowaną zawartością pobranej paczki i uruchamiamy SP Flash Tool wpisując w terminalu
`./flash_tool` . Powinniśmy zobaczyć okienko, z kilkoma zakładkami. Na jednej z nich widnie napis
`Download` . W niej z kolei znajduje się pozycja `Scatter-loading file` . To tutaj musimy wskazać
ścieżkę do pliku `scatter.txt` , który utworzyliśmy
wcześniej:

[![1.neffos-c5-smartfon-android-root-backup-sp-flash-tool]({{< baseurl >}}/img/2016/10/1.neffos-c5-smartfon-android-root-backup-sp-flash-tool-660x444.png)]({{< baseurl >}}/img/2016/10/1.neffos-c5-smartfon-android-root-backup-sp-flash-tool.png)

Teraz przechodzimy na zakładkę `Readback` i tam dodajemy nową pozycję w tabelce. To tutaj określamy
przestrzeń flash'a w telefonie, która zostanie skopiowana na dysk komputera. Nas interesuje cały
flash. Dlatego też początek ustawiamy na `0x0` , a koniec musimy obliczyć z danych dostarczanych
przez `adb` . Interesuje nas ostatnia partycja. Ma ona początek na `0x3a3d80000` , a jej rozmiar to
`0x80000` . Te dwie wartości musimy do siebie dodać, w wyniku czego otrzymujemy `0x3a3e00000` i to
tę wartość wpisujemy w SP Flash Tool. Region określamy jako `EMC_USER`
:

![]({{< baseurl >}}/img/2016/10/2.neffos-c5-smartfon-android-root-backup-sp-flash-tool.png)

Dodajemy również drugą pozycję, która zrobi nam backup preloader'a. Z tym, że tutaj wybieramy region
`EMMC_BOOT_1` i określamy początek jako `0x0` , a koniec jako `0x40000`
:

![]({{< baseurl >}}/img/2016/10/2.1.neffos-c5-smartfon-android-root-backup-sp-flash-tool.png)

Teraz wyłączamy telefon i podłączamy go do portu USB komputera. Następnie w SP Flash Tool aktywujemy
proces backup'u Neffos'a C5 przyciskając `Read Back` . Włączamy teraz telefon przyciskając i
trzymając przycisk Volume Up + Power do momentu aż nam zawibruje. Smartfon się nie włączy ale za to
rozpocznie się kopiowanie danych z telefonu na dysk. Proces backup'u potrwa dłuższą chwilę. W moim
przypadku trwało prawie dwie godziny (transfer na poziomie 3
MiB/s).

[![3.1.neffos-c5-smartfon-android-root-backup-sp-flash-tool]({{< baseurl >}}/img/2016/10/3.1.neffos-c5-smartfon-android-root-backup-sp-flash-tool-660x427.png)]({{< baseurl >}}/img/2016/10/3.1.neffos-c5-smartfon-android-root-backup-sp-flash-tool.png)

Ten backup jest nas w stanie zabezpieczyć na wypadek popełnionych błędów przy flash'owaniu telefonu.
Podejrzymy jeszcze ten obraz w `fdisk`/`gdisk` , by mieć absolutną pewność, że jest w nim faktyczna
kopia flash'a Neffos'a
C5:

[![4.neffos-c5-smartfon-android-root-backup-obraz-gdisk]({{< baseurl >}}/img/2016/10/4.neffos-c5-smartfon-android-root-backup-obraz-gdisk-586x660.png)]({{< baseurl >}}/img/2016/10/4.neffos-c5-smartfon-android-root-backup-obraz-gdisk.png)

## Jak odblokować bootloader w Neffos C5

Mając zrobiony kompletny backup flash'a, możemy przejść do odblokowania bootloader'a. Chodzi o to,
że na smartfonach zwykle jest ulokowana partycja `/recovery/` . Na niej znajduje się oprogramowanie
umożliwiające przeprowadzanie operacji na poziomie systemowym, np. backup lub też flash'owanie
ROM'u. Problem w tym, że to oprogramowanie w standardzie zwykle za wiele nie potrafi i by
przeprowadzić proces root'owania Androida, musimy pozyskać bardziej zaawansowany soft, np.
[ClockworkMod](https://www.clockworkmod.com/) czy [TWRP](https://twrp.me/), i wgrać go na partycję
`/recovery/` . By to zrobić musimy pierw odblokować bootloader.

Proces odblokowania bootloader'a usuwa wszystkie dane, które wgraliśmy na flash telefonu, tj.
podczas odblokowywania jest inicjowany [factory
reset]({{< baseurl >}}/post/android-reset-ustawien-do-fabrycznych-factory-defaults/). Dane na
karcie SD pozostają nietknięte. By ten proces zainicjować zaczynamy od przestawienia jednej opcji w
telefonie. W tym celu musimy udać się w Ustawienia =\> Opcje Programistyczne i tam przełączyć
`Zdjęcie blokady OEM`
:

[![5.neffos-c5-smartfon-android-root-unlock-bootloader]({{< baseurl >}}/img/2016/10/5.neffos-c5-smartfon-android-root-unlock-bootloader-660x361.png)]({{< baseurl >}}/img/2016/10/5.neffos-c5-smartfon-android-root-unlock-bootloader.png)

Następnie wyłączamy telefon i włączamy go trzymając Volume Up + Power. Z menu wybieramy tryb
fastboot. Następnie w terminalu wpisujemy poniższe polecenia:

    # fastboot devices
    TSL7DA69OBSO49PJ        fastboot

    # fastboot oem unlock

Na ekranie smartfona powinno nam pokazać się poniższe ostrzeżenie:

    Unlock bootloader?

    If you unlock the bootloader, you will be able to install custom operating system software on this phone.

    A custom OS is not subject to the same testing as the original OS, and can cause your phone and installed application to stop working properly.

    To prevent unauthorized access to your personal data, unlocking the bootloader will also delete all personal data from your phone "factory reset".

    Press the Volume UP/Down buttons to select Yes/No.

Wciskamy Volume Up, by potwierdzić chęć odblokowania bootloader'a, po czym restartujemy smartfon:

    # fastboot reboot

Jako, że proces odblokowania bootloader'a usunął wszelkie ustawienia, to jeszcze raz musimy włączyć
Opcje programistyczne, a w nich tryb debugowania portu USB.

## Wyodrębnianie obrazu partycji /recovery/ z obrazu Neffos'a C5

Mając zrobiony backup flash'a telefonu, możemy z niego wyciągnąć obraz partycji `/recovery/` .
Musimy tylko zamontować ten obraz w systemie za pomocą `losetup` . Pamiętajmy, że ten obraz ma wiele
partycji. Trzeba zatem nieco dostosować moduł kernela, by wszystkie z tych partycji zostały
zamontowane za pomocą jednego polecenia. Informacje na temat tego [jak dostosować moduł
loop]({{< baseurl >}}/post/obsluga-wielu-partycji-w-module-loop/) znajdują się tutaj. Przechodzimy
zatem w miejsce, w którym zapisaliśmy obraz i montujemy go w poniższy sposób:

    # cd /path/to/image/
    # losetup /dev/loop0 ROM_0

Teraz tworzymy obraz partycji `/recovery/` . W tym przypadku `/dev/loop0p8` jest urządzeniem, które
musimy podać `dd` :

    # dd if=/dev/loop0p8 of=./recovery.img

    # file recovery.img
    recovery.img: Android bootimg, kernel (0x40080000), ramdisk (0x44000000), page size: 2048, cmdline (bootopt=64S3,32N2,64N2)

## Pozyskanie obrazu recovery.img z TWRP

Musimy także pozyskać obraz `recovery.img` zawierający TWRP. Niestety, póki co [nie ma obrazów dla
Neffos'a C5](https://twrp.me/Devices/). Dlatego też musimy sobie taki obraz `recovery.img` stworzyć
sami przerabiając inny obraz, który jest przeznaczony na telefon zbliżony parametrami do naszego
urządzenia (ten sam SoC, wielkość flash i rozdzielczość ekranu).

Warto tutaj zaznaczyć, że nie zawsze taki obraz będzie nam działać bez problemu. W takim przypadku
trzeba próbować innych obrazów, aż któryś zadziała. Nie musimy się tez obawiać wgrania złego obrazu
na partycję `/recovery/` . Jeśli zdarzy nam się wgrać niedziałający obraz `recovery.img` , to nie
uszkodzimy smartfona. Zamiast tego telefon się zrestartuje i przywróci sobie starą partycję
`/recovery/` , a nas zaloguje do systemu jak gdyby nigdy nic.

Moje pierwsze podejście do wgrania obrazu `recovery.img` na Neffos'a C5 się nie powiodło.
Prawdopodobnie obraz nie był odpowiednio przygotowany i w efekcie trzeba było wgrać inny obraz. Ja
skorzystałem z [obrazu podrzuconego przez użytkownika
@GWJ](http://tplink-forum.pl/index.php?/topic/5287-jak-przeprowadzi%C4%87-root-androida-na-neffos-c5/#comment-45313).
Musiałem go tylko dostosować pod Neffos'a C5 ([link do
pobrania]({{< baseurl >}}/img/manual/recovery-neffos-c5-tp-link-twrp.img)).

## Przepakowanie obrazu recovery.img z innego smartfona

By przepakować obraz przeznaczony na inny smartfon, który jest zbliżony parametrami do naszego
Neffos'a C5, musimy pierw pozyskać odpowiednie narzędzia. Na linux'ie możemy skorzystać tego celu z
[abootimg](https://github.com/codeworkx/abootimg) lub też ze [skryptów Android Image
Kitchen](https://github.com/ndrancs/AIK-Linux-x32-x64/). Ja będę korzystał z tego drugiego
rozwiązania.

Tworzymy sobie jakiś katalog roboczy i kopiujemy do niego zarówno oryginalny obraz partycji
`/recovery/` jak i ten z innego smartfona. Następnie znajdując się w tym katalogu roboczym,
pobieramy skrypty z github'a (wymagane zainstalowane narzędzie `git` w systemie) i przechodzimy do
utworzonego w ten sposób katalogu. W nim zaś tworzymy dwa podkatalogi `stock/` oraz `port/` :

    $ git clone https://github.com/ndrancs/AIK-Linux-x32-x64/
    $ chmod +x ./AIK-Linux-x32-x64/*
    $ chmod +x ./AIK-Linux-x32-x64/bin/*
    $ cd ./AIK-Linux-x32-x64/
    $ mkdir stock/ port/

Kopiujemy oryginalny obraz partycji `/recovery/` z katalogu nadrzędnego i wypakowujemy go za pomocą
skryptu `unpackimg.sh` . Następnie przenosimy tak wyodrębnioną zawartość do katalogu `stock/` :

    $ cp ../recovery_orig.img ./recovery.img
    $ ./unpackimg.sh recovery.img
    $ mv split_img/ ramdisk/ stock/
    $ rm recovery.img

Kopiujemy teraz obraz partycji `/recovery/` mający TWRP i wypakowujemy go. Przenosimy jego zawartość
do katalogu `port/` :

    $ cp ../recovery_twrp.img ./recovery.img
    $ ./unpackimg.sh recovery.img
    $ mv split_img/ ramdisk/ port/
    $ rm recovery.img

Zgodnie z [informacją zawartą w tym HOWTO
(windows)](http://www.chinaphonearena.com/forum/Thread-Tutorial-HOW-TO-PORT-TWRP-MT6735-MT6752-MT6753-MT6795-MT6797-TWRP-MT67xx-tutorial),
musimy przekopiować kilka plików z oryginalnego obrazu naszego Neffos'a C5 do obrazu TWRP:

    $ cp ./stock/split_img/recovery.img-kerneloff ./port/split_img/
    $ cp ./stock/split_img/recovery.img-zImage ./port/split_img/
    $ cp ./stock/ramdisk/fstab.mt6735 ./port/ramdisk/
    $ cp ./stock/ramdisk/meta_init.modem.rc ./port/ramdisk/
    $ cp ./stock/ramdisk/meta_init.project.rc ./port/ramdisk/
    $ cp ./stock/ramdisk/meta_init.rc ./port/ramdisk/
    $ cp ./stock/ramdisk/ueventd.rc ./port/ramdisk/

Z tak przygotowanych plików w katalogu `port/` trzeba zrobić nowy obraz `recovery.img` przy pomocy
skryptu `repackimg_x64.sh` :

    $ rm -R stock/
    $ mv port/ramdisk ./
    $ mv port/split_img ./
    $ rmdir port/
    $ ./repackimg_x64.sh

W katalogu roboczym powinien zostać utworzony nowy plik o nazwie `image-new.img` . To jest właśnie
nasz nowy obraz partycji `/recovery/` , który musimy wgrać na telefon w trybie fastboot.

## Wgrywanie przepakowanego obrazu recovery.img na Neffos'a C5

Wyłączamy telefon i podpinamy go do portu USB. Następnie włączamy go trzymając VolumeUP + Power. Z
menu, które nam się pokaże na ekranie telefonu wybieramy tryb fastboot. Wracamy na komputer i przy
pomocy narzędzia `fasboot` wgrywamy wyżej wygenerowany obraz na telefon.

    # fastboot flash recovery image-new.img
    target reported max download size of 134217728 bytes
    sending 'recovery' (12642 KB)...
    OKAY [  1.301s]
    writing 'recovery'...
    OKAY [  0.279s]
    finished. total time: 1.580s

Restartujemy telefon ( `fastboot reboot` ) i wchodzimy w tryb recovery (VolumeUp + Power), by
potwierdzić, że nasz nowy obraz recovery się wgrał pomyślnie. W przypadku wgrania poprawnego obrazu,
powinniśmy zobaczyć logo TWRP. Jeśli wgrany obraz był nieprawidłowy, to zobaczymy jedynie czarny
ekran i nasz Neffos C5 się po chwili uruchomi ponownie w normalnym trybie przywracając oryginalną
partycję `/recovery/` .

## SuperSU, BusyBOX, RootCheck i emulator terminala

Ostatnią rzeczą na drodze do zrobienia root na Neffos C5 jest wgranie aplikacji umożliwiającej
korzystanie różnym programom z praw administratora systemu w telefonie. My skorzystamy z SuperSU.
Dodatkowo, musimy wgrać sobie BusyBOX, który zawiera minimalistyczne odpowiedniki linux'owych
narzędzi, co pozwoli nam swobodnie operować w Androidzie, tak jakbyśmy to robili pod
pełnowymiarowym pingwinem. Potrzebny nam także będzie jakiś emulator terminala, w którym to
będziemy wpisywać wszystkie nasze polecenia.

### Instalacja SuperSU

Zacznijmy od pobrania stosownej paczki z
[SuperSU](https://forum.xda-developers.com/apps/supersu/stable-2016-09-01supersu-v2-78-release-t3452703).
Jako, że my nie mamy jeszcze zrobionego root'a, to musimy pobrać `TWRP / FlashFire installable ZIP`
. Tej paczki nie wypakowujemy, tylko wrzucamy ją w pobranej formie na kartę SD w telefonie. Odpalamy
teraz tryb recovery w smartfonie (VolumeUp + Power) i przechodzimy kolejno do Instaluj (TWRP jest
również w języku
polskim):

[![1.twrp-instalacja-supersu-tryb-recovery]({{< baseurl >}}/img/2016/10/1.twrp-instalacja-supersu-tryb-recovery-660x390.png)]({{< baseurl >}}/img/2016/10/1.twrp-instalacja-supersu-tryb-recovery.png)

Następnie wskazujemy paczkę `.zip` , którą umieściliśmy na karcie SD. Tam z kolei zaznaczamy
`Weryfikuj sygnatury pliku zip` i przeciągamy trzy strzałki na prawą
stronę.

[![2.twrp-instalacja-supersu-tryb-recovery]({{< baseurl >}}/img/2016/10/2.twrp-instalacja-supersu-tryb-recovery-660x390.png)]({{< baseurl >}}/img/2016/10/2.twrp-instalacja-supersu-tryb-recovery.png)

Teraz możemy uruchomić ponownie Neffos'a C5 i zainstalować jakąś aplikację, która pokaże nam czy
nasz smartfon ma root'a.

### Sprawdzenie czy Neffos C5 ma root'a

Po uruchomieniu się systemu na smartfonie, instalujemy aplikację
[RootCheck](https://play.google.com/store/apps/details?id=com.jrummyapps.rootchecker), po czym
uruchamiamy ją. Powinien się pojawić monit informujący, że ta aplikacja żąda praw administracyjnych,
na co zezwalamy. Jeśli nasz telefon ma root'a, to powinien się pojawić stosowny
komunikat:

[![6.neffos-c5-smartfon-android-root-success-root-check]({{< baseurl >}}/img/2016/10/6.neffos-c5-smartfon-android-root-success-root-check-1-660x543.png)]({{< baseurl >}}/img/2016/10/6.neffos-c5-smartfon-android-root-success-root-check-1.png)

### Instalacja BusyBOX

Kolejnym krokiem jest instalacja
[BusyBOX'a](https://play.google.com/store/apps/details?id=stericson.busybox). Po wgraniu tej
aplikacji na smartfona, musimy ją uruchomić i wcisnąć w niej przycisk `install` . BusyBOX również
nas poprosi o dostęp do praw
administracyjnych:

[![7.neffos-c5-smartfon-android-root-busybox]({{< baseurl >}}/img/2016/10/7.neffos-c5-smartfon-android-root-busybox-660x362.png)]({{< baseurl >}}/img/2016/10/7.neffos-c5-smartfon-android-root-busybox.png)

Po zainstalowaniu, weryfikujemy jeszcze, czy aby wszystko zostało pomyślne wgrane. Możemy to zrobić
zarówno w samej aplikacji BusyBOX, jak w w
CheckRoot:

[![8.neffos-c5-smartfon-android-root-busybox-success-check]({{< baseurl >}}/img/2016/10/8.neffos-c5-smartfon-android-root-busybox-success-check-660x543.png)]({{< baseurl >}}/img/2016/10/8.neffos-c5-smartfon-android-root-busybox-success-check.png)

### Instalacja terminala

Z BusyBOX'em zostały dostarczone niskopoziomowe narzędzia, które mogą korzystać z uprawnień
administratora systemu za sprawą SuperSU. Niemniej jednak, by móc odpalać te programiki, musimy w
czymś je uruchomić. Do tego celu potrzebny jest nam shell oraz emulator terminala. Domyślny shell
`ash` jest dostarczany razem z BusyBOX. Terminal musimy doinstalować osobno.

Generalnie to znalazłem dwa terminale, które są OpenSource i bez reklam/opłat. Są to
[Android-Terminal-Emulator](https://play.google.com/store/apps/details?id=jackpal.androidterm) oraz
[Termux](https://play.google.com/store/apps/details?id=com.termux). Wybieramy sobie jeden z nich i
instalujemy w systemie. Tutaj warto jeszcze zaznaczyć, że ten drugi terminal instaluje sobie również
shell `dash` (domyślny shell w Debianie) . Również w jego przypadku możemy łatwo doinstalować sobie
aplikacje za pomocą `apt` , podobnie jak w Debianie (do tego celu nie jest wymagany root). Jako, że
ja korzystam na co dzień z Debiana, to instaluje
Termux'a.

[![9.neffos-c5-smartfon-android-root-termux-htop]({{< baseurl >}}/img/2016/10/9.neffos-c5-smartfon-android-root-termux-htop-660x371.png)]({{< baseurl >}}/img/2016/10/9.neffos-c5-smartfon-android-root-termux-htop.png)

### Aplikacje i prawa administracyjne

Teraz już pozostało nam tylko odpalenie terminala i zalogowanie się na użytkownika root. Do tego
celu służy polecenie `su` . Wpiszmy je zatem w okienku
Termux'a:

[![10.neffos-c5-smartfon-android-root-termux-su]({{< baseurl >}}/img/2016/10/10.neffos-c5-smartfon-android-root-termux-su-660x542.png)]({{< baseurl >}}/img/2016/10/10.neffos-c5-smartfon-android-root-termux-su.png)

I teraz możemy uruchamiać aplikacjie z prawami admina, tak jak to zwykliśmy robić w każdym innym
linux'ie. Pamiętajmy tylko, że standardowo system plików jest zamontowany w trybie tylko do odczytu
(RO) i by móc zmieniać pliki systemowe z poziomu tego terminala, musimy przemontować system plików w
tryb do zapisu (RW). Robimy to w poniższy sposób:

    $ su
    # mount -o remount,rw /system

Gdy skończymy się bawić, to montujemy ten system plików ponownie w tryb RO:

    # mount -o remount,ro /system
