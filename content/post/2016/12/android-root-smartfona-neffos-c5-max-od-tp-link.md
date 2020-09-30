---
author: Morfik
categories:
- Android
date: "2016-12-13T17:27:37Z"
date_gmt: 2016-12-13 16:27:37 +0100
published: true
status: publish
tags:
- tp-link
- smartfon
- lollipop
- root
- twrp
- neffos
title: 'Android: Root smartfona Neffos C5 MAX od TP-LINK'
---

Bawiąc się ostatnio [smartfonem Neffos C5 MAX][1] od TP-LINK, obiecałem sobie, że jak tylko będę
miał chwilę czasu, to postaram się ukorzenić Androida, który w tym telefonie siedzi (Lollipop).
Generalnie rzecz biorąc, sposób root'owania tego urządzenia jest bardzo podobny do tego, który
miałem już możliwość [przeprowadzić na innym modelu TP-LINK'a, tj. Neffos C5][2]. Dlatego też
poniższy artykuł jest bardzo zbliżony treścią, choć lekko zaktualizowany pod kątem Neffos'a C5 MAX.
Grunt, że nie było żadnych problemów z przeprowadzeniem backup'u flash'a telefonu jak i samego
procesu root.

<!--more-->

> Prostszy sposób na przeprowadzanie procesu root w smartfonach Neffos od TP-LINK z wykorzystaniem
> natywnych obrazów TWRP [został opisany w nowym wątku][3].

## Narzędzia ADB i fastboot

Przede wszystkim, by zabrać się za proces root'owania smartfona Neffos C5 MAX, musimy przygotować
sobie odpowiednie narzędzia. Zapewnią one nam możliwość rozmawiania z telefonem. Będziemy
potrzebować `adb` ([Android Debug Bridge][4]) oraz `fastboot` . [Proces instalacji tych narzędzi na
linux][5], a konkretnie w dystrybucji Debian, został opisany osobno.

## Narzędzie SP Flash Tool

Kolejnym narzędziem, które będzie nam niezbędne jest [SP Flash Tool][6]. Niestety nie jest ono
włączone do dystrybucji Debian i musimy posiłkować się paczką, którą można znaleźć w podanym wyżej
linku. Tutaj ważna uwaga. SP FLash Tool jest przeznaczony tylko dla smartfonów mających SoC od
Mediatek.

Pobieramy paczkę `.zip` dla linux'a i wypakowujemy ją. Jako, że SP Flash Tool wykorzystuje do
komunikacji interfejs `/dev/ttyACM0` , to do poprawnej pracy wymaga on operowania na tym
interfejsie. Standardowo tylko administrator systemu oraz członkowie grupy `dialout` są w stanie
korzystać z tego interfejsu. Musimy zatem dodać naszego użytkownika do tej grupy w poniższy sposób:

    # gpasswd -a morfik dialout

## Kompletny backup flash'a Neffos C5 MAX

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
    userdata         0x0000000124000000     0x0000000286780000
    flashinfo        0x00000003aa780000     0x0000000001000000
    sgpt             0x00000003ab780000     0x0000000000080000

Ten zwrócony wyżej wynik jest dla smartfona Neffos C5 MAX. W przypadku innych telefonów, ta tabelka
może mieć inną postać i nie możemy kopiować z niej wartości jeśli mamy inne urządzenie. Generalnie
rzecz biorąc, to te dane potrzebne nam są do zbudowania pliku `scatter.txt` , w oparciu o który
działa SP Flash Tool. Ten plik to zwyczajna mapa przestrzeni flash'a telefonu, który podzielony
jest na szereg widocznych wyżej partycji.

### Plik scatter.txt dla Neffos C5 MAX

[Tutaj znajduje się plik scatter.txt][7], który ja wykorzystałem do pracy z Neffos C5 MAX. Kluczowa
sprawa, to opisanie każdej partycji. W sumie to musimy odpowiednio dostosować pole
`partition_index` , które jest zwyczajnie kolejnym numerkiem. Z kolei w `partition_name` podajemy
nazwę partycji, którą uzyskaliśmy przez `adb` . Dalej w `linear_start_addr` oraz `physical_start_
addr` wpisujemy tę wartość, która została wypisana przez `adb` w kolumnie `Start` . Na podobnej
zasadzie uzupełniamy `partition_size` , podając wartość, którą widzieliśmy w `adb` w kolumnie
`Size` . I to w zasadzie wszystkie zmiany, które musimy wprowadzić do pliku `scatter.txt` . Póki co
nie mam informacji co do pozostałych opcji w tym pliku, wiem tylko, że część z nich jest uzupełniana
przez SP Flash Tool podczas przeprowadzania działań w tym programie.

### Tworzenie backupu

Mając plik `scatter.txt` możemy go wskazać w SP Flash Tool. Przechodzimy zatem do katalogu z
wypakowaną zawartością pobranej paczki i uruchamiamy SP Flash Tool wpisując w terminalu
`./flash_tool` . Powinniśmy zobaczyć okienko, z kilkoma zakładkami. Na jednej z nich widnie napis
`Download` . W niej z kolei znajduje się pozycja `Scatter-loading file` . To tutaj musimy wskazać
ścieżkę do pliku `scatter.txt` , który utworzyliśmy wcześniej:

![](/img/2016/12/001.neffos-c5-max-smartfon-root-android-tp-link-backup-sp-flash-tool.png#huge)

Teraz przechodzimy na zakładkę `Readback` i tam dodajemy nową pozycję w tabelce. To tutaj określamy
przestrzeń flash'a w telefonie, która zostanie skopiowana na dysk komputera. Nas interesuje cały
flash. Dlatego też początek ustawiamy na `0x0` , a koniec musimy obliczyć z danych dostarczanych
przez `adb` . Interesuje nas ostatnia partycja. Ma ona początek na `0x3ab780000` , a jej rozmiar to
`0x80000` . Te dwie wartości musimy do siebie dodać, w wyniku czego otrzymujemy `0x3ab800000` i to
tę wartość wpisujemy w SP Flash Tool. Region określamy jako `EMC_USER` :

![](/img/2016/12/002.neffos-c5-max-smartfon-root-android-tp-link-backup-sp-flash-tool.png#big)

Dodajemy również drugą pozycję, która zrobi nam backup preloader'a. Z tym, że tutaj wybieramy region
`EMMC_BOOT_1` i określamy początek jako `0x0` , a koniec jako `0x40000` :

![](/img/2016/12/003.neffos-c5-max-smartfon-root-android-tp-link-backup-sp-flash-tool.png#big)

Teraz wyłączamy telefon. Następnie w SP Flash Tool aktywujemy proces backup'u Neffos'a C5 MAX
przyciskając `Read Back` . Podłączamy telefon do portu USB komputera. Po chwili smartfon powinien
nam zawibrować ale nie uruchomi się. Rozpocznie się za to kopiowanie danych z telefonu na dysk.
Proces backup'u potrwa dłuższą chwilę. W moim przypadku trwało prawie dwie godziny (transfer na
poziomie 3 MiB/s).

![](/img/2016/12/004.neffos-c5-max-smartfon-root-android-tp-link-backup-sp-flash-tool.png#huge)

Ten backup jest nas w stanie zabezpieczyć na wypadek popełnionych błędów przy flash'owaniu telefonu.
Podejrzymy jeszcze ten obraz w `fdisk`/`gdisk` , by mieć absolutną pewność, że jest w nim faktyczna
kopia flash'a Neffos'a C5 MAX:

![](/img/2016/12/005.neffos-c5-max-smartfon-root-android-tp-link-obraz-flash-gdisk.png#huge)

## Jak odblokować bootloader w Neffos C5 MAX

Mając zrobiony kompletny backup flash'a, możemy przejść do odblokowania bootloader'a. Chodzi o to,
że na smartfonach zwykle jest ulokowana partycja `/recovery/` . Na niej znajduje się oprogramowanie
umożliwiające przeprowadzanie operacji na poziomie systemowym, np. backup lub też flash'owanie
ROM'u. Problem w tym, że to oprogramowanie w standardzie zwykle za wiele nie potrafi i by
przeprowadzić proces root'owania Androida, musimy pozyskać bardziej zaawansowany soft, np.
[ClockworkMod][8] czy [TWRP][9], i wgrać go na partycję `/recovery/` . By to zrobić musimy pierw
odblokować bootloader.

Proces odblokowania bootloader'a usuwa wszystkie dane, które wgraliśmy na flash telefonu, tj.
podczas odblokowywania jest inicjowany [factory reset][10]. Dane na karcie SD pozostają nietknięte.
By ten proces zainicjować zaczynamy od przestawienia jednej opcji w telefonie. W tym celu musimy
udać się w Ustawienia => Opcje Programistyczne i tam przełączyć `Zdjęcie blokady OEM` :

![](/img/2016/12/006.neffos-c5-max-smartfon-root-android-tp-link-unlock-bootloader.png#huge)

Następnie wyłączamy telefon i włączamy go trzymając Volume Up + Power. Z menu wybieramy tryb
fastboot. Następnie w terminalu wpisujemy poniższe polecenia:

    # fastboot devices
    8HCMMZFI89ROBI9H        fastboot

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
Opcje Programistyczne, a w nich tryb debugowania portu USB.

## Wyodrębnianie obrazu partycji /recovery/ z obrazu Neffos'a C5 MAX

Mając zrobiony backup flash'a telefonu, możemy z niego wyciągnąć obraz partycji `/recovery/` .
Musimy tylko zamontować ten obraz w systemie za pomocą `losetup` . Pamiętajmy, że ten obraz ma wiele
partycji. Trzeba zatem nieco dostosować moduł kernela, by wszystkie z tych partycji zostały
zamontowane za pomocą jednego polecenia. Informacje na temat tego [jak dostosować moduł loop][11]
znajdują się tutaj. Przechodzimy zatem w miejsce, w którym zapisaliśmy obraz i montujemy go w
poniższy sposób:

    # cd /path/to/image/
    # losetup /dev/loop0 ROM_0

Teraz tworzymy obraz partycji `/recovery/` . W tym przypadku `/dev/loop0p8` jest urządzeniem, które
musimy podać `dd` :

    # dd if=/dev/loop0p8 of=./recovery.img

    # file recovery.img
    recovery.img: Android bootimg, kernel (0x40080000), ramdisk (0x44000000), page size: 2048, cmdline (bootopt=64S3,32N2,64N2)

## Pozyskanie obrazu recovery.img z TWRP

Musimy także pozyskać obraz `recovery.img` zawierający TWRP. Niestety, póki co [nie ma obrazów dla
Neffos'a C5 MAX][12]. Dlatego też musimy sobie taki obraz `recovery.img` stworzyć sami przerabiając
inny obraz, który jest przeznaczony na telefon zbliżony parametrami do naszego urządzenia (ten sam
SoC, wielkość flash i rozdzielczość ekranu). [Pod tym linkiem znajduje się gotowy obraz
recovery.img][13].

Warto tutaj zaznaczyć, że nie zawsze taki obraz będzie nam działać bez problemu. W takim przypadku
trzeba próbować innych obrazów, aż któryś zadziała. Nie musimy się tez obawiać wgrania złego obrazu
na partycję `/recovery/` . Jeśli zdarzy nam się wgrać niedziałający obraz `recovery.img` , to nie
uszkodzimy smartfona. Zamiast tego telefon się zrestartuje i przywróci sobie starą partycję
`/recovery/` , a nas zaloguje do systemu jak gdyby nigdy nic.

## Przepakowanie obrazu recovery.img z innego smartfona

By przepakować obraz przeznaczony na inny smartfon, który jest zbliżony parametrami do naszego
Neffos'a C5 MAX, musimy pierw pozyskać odpowiednie narzędzia. Na linux'ie możemy skorzystać tego
celu z [abootimg][14] lub też ze [skryptów Android Image Kitchen][15]. Ja będę korzystał z tego
drugiego rozwiązania.

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

Zgodnie z [informacją zawartą w tym HOWTO (windows)][16], musimy przekopiować kilka plików z
oryginalnego obrazu naszego Neffos'a C5 MAX do obrazu TWRP:

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

## Wgrywanie przepakowanego obrazu recovery.img na Neffos'a C5 MAX

Wyłączamy telefon i podpinamy go do portu USB. Następnie włączamy go trzymając VolumeUP + Power. Z
menu, które nam się pokaże na ekranie telefonu wybieramy tryb fastboot. Wracamy na komputer i przy
pomocy narzędzia `fasboot` wgrywamy wyżej wygenerowany obraz na telefon.

    # fastboot flash recovery image-new.img
    target reported max download size of 134217728 bytes
    sending 'recovery' (12868 KB)...
    OKAY [  1.125s]
    writing 'recovery'...
    OKAY [  0.360s]
    finished. total time: 1.485s

Restartujemy telefon ( `fastboot reboot` ) i wchodzimy w tryb recovery (VolumeUp + Power), by
potwierdzić, że nasz nowy obraz recovery się wgrał pomyślnie. W przypadku wgrania poprawnego obrazu,
powinniśmy zobaczyć logo TWRP. Jeśli wgrany obraz był nieprawidłowy, to zobaczymy jedynie czarny
ekran i nasz Neffos C5 MAX się po chwili uruchomi ponownie w normalnym trybie przywracając
oryginalną partycję `/recovery/` .

## SuperSU, BusyBOX, RootCheck i emulator terminala

Ostatnią rzeczą na drodze do zrobienia root na Neffos C5 MAX jest wgranie aplikacji umożliwiającej
korzystanie różnym programom z praw administratora systemu w telefonie. My skorzystamy z SuperSU.
Dodatkowo, musimy wgrać sobie BusyBOX, który zawiera minimalistyczne odpowiedniki linux'owych
narzędzi, co pozwoli nam swobodnie operować w Androidzie, tak jakbyśmy to robili pod
pełnowymiarowym pingwinem. Potrzebny nam także będzie jakiś emulator terminala, w którym to
będziemy wpisywać wszystkie nasze polecenia.

### Instalacja SuperSU

Zacznijmy od pobrania stosownej paczki z
[SuperSU][17]. Jako, że my nie mamy jeszcze zrobionego root'a, to musimy pobrać
`TWRP / FlashFire installable ZIP` . Tej paczki nie wypakowujemy, tylko wrzucamy ją w pobranej
formie na kartę SD w telefonie. Odpalamy teraz tryb recovery w smartfonie (VolumeUp + Power) i
przechodzimy do Instaluj (TWRP jest również w języku polskim):

![](/img/2016/12/007.neffos-c5-max-smartfon-root-android-tp-link-supersu-tryb-recovery-twrp.png#huge)

Następnie wskazujemy paczkę `.zip` , którą umieściliśmy na karcie SD. Tam z kolei zaznaczamy
`Weryfikuj sygnatury pliku zip` i przeciągamy trzy strzałki na prawą stronę.

![](/img/2016/12/008.neffos-c5-max-smartfon-root-android-tp-link-supersu-tryb-recovery-twrp.png#huge)

Teraz możemy uruchomić ponownie Neffos'a C5 MAX i zainstalować jakąś aplikację, która pokaże nam czy
nasz smartfon ma root'a.

### Sprawdzenie czy Neffos C5 MAX ma root'a

Po uruchomieniu się systemu na smartfonie, instalujemy aplikację [RootCheck][18], po czym
uruchamiamy ją. Powinien się pojawić monit informujący, że ta aplikacja żąda praw administracyjnych,
na co zezwalamy. Jeśli nasz telefon ma root'a, to powinien się pojawić stosowny komunikat:

![](/img/2016/12/009.neffos-c5-max-smartfon-root-android-tp-link-check.png#huge)

### Instalacja BusyBOX

Kolejnym krokiem jest instalacja [BusyBOX'a][19]. Po wgraniu tej aplikacji na smartfona, musimy ją
uruchomić i wcisnąć w niej przycisk `install` . BusyBOX również nas poprosi o dostęp do praw
administracyjnych:

![](/img/2016/12/010.neffos-c5-max-smartfon-root-android-tp-link-busybox.png#huge)

Po zainstalowaniu, weryfikujemy jeszcze, czy aby wszystko zostało pomyślne wgrane. Możemy to zrobić
zarówno w samej aplikacji BusyBOX, jak i w CheckRoot:

![](/img/2016/12/011.neffos-c5-max-smartfon-root-android-tp-link-busybox-check.png#huge)

### Instalacja terminala

Z BusyBOX'em zostały dostarczone niskopoziomowe narzędzia, które mogą korzystać z uprawnień
administratora systemu za sprawą SuperSU. Niemniej jednak, by móc odpalać te programiki, musimy w
czymś je uruchomić. Do tego celu potrzebny jest nam shell oraz emulator terminala. Domyślny shell
`ash` jest dostarczany razem z BusyBOX. Terminal musimy doinstalować osobno.

Generalnie to znalazłem dwa terminale, które są OpenSource i bez reklam/opłat. Są to
[Android-Terminal-Emulator][20] oraz [Termux][21]. Wybieramy sobie jeden z nich i instalujemy w
systemie. Tutaj warto jeszcze zaznaczyć, że ten drugi terminal instaluje sobie również shell `dash`
(domyślny shell w Debianie) . Również w jego przypadku możemy łatwo doinstalować sobie aplikacje za
pomocą `apt` , podobnie jak w Debianie (do tego celu nie jest wymagany root). Jako, że ja korzystam
na co dzień z Debiana, to instaluje Termux'a.

![](/img/2016/12/012.neffos-c5-max-smartfon-root-android-tp-link-htop.png#huge)

### Aplikacje i prawa administracyjne

Teraz już pozostało nam tylko odpalenie terminala i zalogowanie się na użytkownika root. Do tego
celu służy polecenie `su` . Wpiszmy je zatem w okienku Termux'a:

![](/img/2016/12/013.neffos-c5-max-smartfon-root-android-tp-link-termux.png#huge)

I teraz możemy uruchamiać aplikacje z prawami admina, tak jak to zwykliśmy robić w każdym innym
linux'ie. Pamiętajmy tylko, że standardowo system plików jest zamontowany w trybie tylko do odczytu
(RO) i by móc zmieniać pliki systemowe z poziomu tego terminala, musimy przemontować system plików w
tryb do zapisu (RW). Robimy to w poniższy sposób:

    $ su
    # mount -o remount,rw /system

Gdy skończymy się bawić, to montujemy ten system plików ponownie w tryb RO:

    # mount -o remount,ro /system


[1]: /post/recenzja-smartfon-neffos-c5-max-od-tp-link/
[2]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[3]: /post/root-w-smartfonach-neffos-od-tp-link-x1-c5-c5-max-y5-y5l/
[4]: https://developer.android.com/studio/command-line/adb.html
[5]: /post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/
[6]: http://spflashtool.com/
[7]: /img/manual/mt6753-neffos-c5-max-tp-link-scatter.txt
[8]: https://www.clockworkmod.com/
[9]: https://twrp.me/
[10]: /post/android-reset-ustawien-do-fabrycznych-factory-defaults/
[11]: /post/obsluga-wielu-partycji-w-module-loop/
[12]: https://twrp.me/Devices/
[13]: /img/manual/recovery-neffos-c5-tp-link-twrp.img
[14]: https://github.com/codeworkx/abootimg
[15]: https://github.com/ndrancs/AIK-Linux-x32-x64/
[16]: http://www.chinaphonearena.com/forum/Thread-Tutorial-HOW-TO-PORT-TWRP-MT6735-MT6752-MT6753-MT6795-MT6797-TWRP-MT67xx-tutorial
[17]: https://forum.xda-developers.com/apps/supersu/stable-2016-09-01supersu-v2-78-release-t3452703
[18]: https://play.google.com/store/apps/details?id=com.jrummyapps.rootchecker
[19]: https://play.google.com/store/apps/details?id=stericson.busybox
[20]: https://play.google.com/store/apps/details?id=jackpal.androidterm
[21]: https://play.google.com/store/apps/details?id=com.termux
