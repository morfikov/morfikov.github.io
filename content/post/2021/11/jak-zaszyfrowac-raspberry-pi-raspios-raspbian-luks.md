---
author: Morfik
categories:
- RaspberryPi
date:    2021-11-07 22:17:00 +0100
lastmod: 2021-11-07 22:17:00 +0100
published: true
status: publish
tags:
- luks
- raspberry-pi-4b
- raspios
- raspbian
- szyfrowanie
- initrd
- initramfs
GHissueID: 579
title: Jak zaszyfrować Raspberry Pi (RasPiOS/Raspbian, LUKS)
---

Ostatnio napisał do mnie pewien osobnik, który miał dość spory problem z zaszyfrowaniem swojego
Raspberry Pi. Po wymianie kilku maili zacząłem tracić rozeznanie w całej tej sytuacji i doszedłem
do wniosku, że najwyraźniej zaszyfrowanie systemu maliny nie jest rzeczą trywialną. Postanowiłem
zatem rzucić okiem na swojego RPI i sprawdzić czy są tutaj faktycznie jakieś odstępstwa od
tradycyjnego szyfrowania na bazie LUKS w stosunku do szyfrowania spotykanego w pierwszej lepszej
dystrybucji linux'a, takiej jak Debian czy Ubuntu. Pobrałem więc ze strony Raspberry Pi najnowszy
obraz systemu RasPiOS/Raspbian, wrzuciłem go na kartę SD przy pomocy `dd` i okazało się, że w
zasadzie większych problemów z zaszyfrowaniem mojej maliny nie było. Przyznać trzeba, że informacje
na temat szyfrowania RPI, które można spotkać na internetach, są w przeważającej większości bardzo
słabej jakości i mam wrażenie, że pisane przez osoby, które nie do końca się zainteresowały
poruszanym przez siebie zagadnieniem. Postanowiłem zatem na przykładzie mojego Raspberry Pi 4B
opisać w miarę dokładny sposób od początku do końca jak przeprowadzić cały ten proces szyfrowania
danych na karcie SD, którą podpinamy do naszego minikomputera.

<!--more-->
## RasPiOS/Raspbian, czyli system operacyjny dla Raspberry Pi

By przystąpić do zaszyfrowania systemu w Raspberry Pi, będzie nam potrzebna karta SD, na którą
wgramy sobie system RasPiOS (dawniej zwany Raspbian). Będziemy także potrzebować jakiegoś nośnika
danych, na który zrzucimy sobie pliki rezydujące na partycji `/` tego OS'a. Takim nośnikiem może
być inna karta SD czy pendrive, lub też jakiś katalog na dysku naszego laptopa lub desktopa.

Zacznijmy może od przejścia na [oficjalną stronę RaspberryPi][1] i pobrania z niej systemu
RasPiOS/Raspbian. Pobrany obraz jest spakowany i trzeba będzie go wypakować, po czym tak otrzymany
plik wrzucimy sobie na kartę SD via `dd` .

Zanim jednak przejdziemy do wrzucania samego obrazu, zweryfikujmy sumę kontrolną przy pomocy
`sha256sum` , by się upewnić, że obraz w procesie pobierania nie został w żaden sposób zmieniony.
Szkoda tylko, że brakuje podpisów cyfrowych i nie ma jak zweryfikować czy czasem tych obrazów nikt
nie zmienił w sposób nieuprawniony:

    $ echo "84a711d9ff4c295711a40af43fea38893a20b3b087a48370d8fb926fb541faf5  2021-05-07-raspios-buster-armhf-full.zip" | sha256sum -c
    2021-05-07-raspios-buster-armhf-full.zip: OK

Suma kontrolna się zgadza, zatem wypakowujemy ten plik:

    $ patool extract  '2021-05-07-raspios-buster-armhf-full.zip'

    $  ls -hal 2021-05-07-raspios-buster-armhf-full.*
    -rw-r--r--+ 1 morfik p2p 8.1G 2021-05-07 17:23:51 2021-05-07-raspios-buster-armhf-full.img
    -rw-rw----+ 1 morfik p2p 2.8G 2021-11-06 19:51:04 2021-05-07-raspios-buster-armhf-full.zip

No jak widać, rozmiarowo bardzo się różnią te dwa pliki.

Wrzucamy obraz na kartę SD via `dd` :

    # dd if=2021-05-07-raspios-buster-armhf-full.img of=/dev/mmcblk0 status=progress

Jak tylko ten proces dobiegnie końca, podłączamy kartę SD do naszej maliny i uruchamiamy system,
tak by wstępnie go skonfigurować za sprawą PiWizard.

![raspberry-pi-rpi-luks-encrypt-fresh-install](/img/2021/11/001.raspberry-pi-rpi-luks-encrypt-fresh-install.png#huge)

## Instalacja potrzebnego oprogramowania

By móc z powodzeniem przeprowadzić proces szyfrowania/deszyfrowania plików zgromadzonych na
partycji `/` na karcie SD w Raspberry Pi, potrzebujemy odpowiedniego oprogramowania. Najlepiej
zainstalować sobie standardowy zestaw pakietów, w skład którego wchodzą `cryptsetup` , `lvm2` ,
`busybox` oraz `initramfs-tools` .

    root@raspberrypi:/home/pi# apt-get install cryptsetup lvm2 busybox initramfs-tools

Po instalacji potrzebnego oprogramowania dobrze jest zrestartować RPI, tak by wszystkie nowe usługi
się poprawnie zainicjowały, tj. uruchomiły się i utworzyły wszystkie potrzebne do swojego
prawidłowego działania pliki/katalogi, etc.

## Raspberry Pi i obraz initramfs/initrd

Domyślnie Raspberry Pi nie korzysta z obrazu initramfs/initrd, bo w zasadzie nie ma takiej potrzeby.
Niemniej jednak, jeśli chcemy zaszyfrować główny systemu plików naszego RPI, to musimy zdawać sobie
sprawę, że kernel nie będzie potrafił tak zaszyfrowanego systemu plików zamontować. Dlatego właśnie
będzie nam potrzebny obraz initramfs/initrd i bez niego się zwyczajnie nie obejdziemy. Musimy zatem
włączyć obsługę initramfs/initrd w Raspberry Pi.

Na sam początek edytujemy plik `/etc/default/raspberrypi-kernel` i usuwamy komentarz z linijki
zawierającej `INITRD=` , który ma na celu włączenie generowania obrazu initramfs/initrd ilekroć
tylko kernel będzie instalowany/aktualizowany w systemie:

    INITRD=Yes
    #RPI_INITRD=Yes

Parametr `RPI_INITRD=` zostawiamy w spokoju, bo w repozytorium nie ma póki co pakietu
`rpi-initramfs-tools` . Być może kiedyś w przyszłości taki [pakiet się pojawi][3].

### Wstępne generowanie obrazu

Możemy teraz wstępnie sobie wygenerować obraz initramfs/initrd korzystając z `update-initramfs` . W
tym celu wystarczy w terminalu wpisać to poniższe polecenie:

    root@raspberrypi:/home/pi# update-initramfs -c -k $(uname -r)
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    cryptsetup: ERROR: Couldn't resolve device /dev/root
    cryptsetup: WARNING: Couldn't determine root device
    cryptsetup: WARNING: The initramfs image may not contain cryptsetup binaries
        nor crypto modules. If that's on purpose, you may want to uninstall the
        'cryptsetup-initramfs' package in order to disable the cryptsetup initramfs
        integration and avoid this warning.

Mamy tutaj parę błędów generowanych przez `cryptsetup` ale na razie nie ma co się nimi przejmować,
bo jeszcze nie utworzyliśmy zaszyfrowanego kontenera LUKS, nie wspominając nawet o jego
konfiguracji. Jeśli jednak zajrzymy do katalogu `/boot/` , to zobaczymy, że obraz initrd/initramfs
został wygenerowany z powodzeniem:

    root@raspberrypi:/home/pi# ls -hal /boot | grep -i initrd
    -rwxr-xr-x  1 root root  17M Nov  6 23:24 initrd.img-5.10.63-v7l+

#### Problematyczne nazwy obrazów initramfs/initrd

Początkowo myślałem, że da rade utworzyć dowiązania symboliczne w `/boot/` do obrazów
initramfs/initrd, coś na wzór przeciętnej dystrybucji linux'a. Problem jednak w tym, że Raspberry
Pi ma partycję `/boot/` sformatowaną systemem plików `vfat` , a nie `ext4` . System plików FAT
niestety nie wspiera linków i można zapomnieć, np. o rozwiązaniu, które [zostało przedstawione w
tym artykule][4]. Nie da rady też sformatować partycji `/boot/` systemem plików ext4, bo
najwyraźniej [bootloader Raspberry Pi][5] wymaga, by ta partycja miała system plików FAT.

Wygląda na to, że domyślnie przy aktualizacji kernela trzeba będzie zwracać uwagę na numerek obecny
w pliku `initrd.img` i za każdym razem, gdy ulegnie on zmianie będziemy musieli odpowiednio
dostosować konfigurację RPI, by system był w stanie odszyfrować dane z karty SD. Przyznam, że
niezbyt mi się taka opcja spodobała i postanowiłem poszukać jakiegoś bardziej cywilizowanego
rozwiązania. [Trafiłem na taki oto post][15], który w zasadzie idealnie rozwiązuje cały zaistniały
problem z nazwami obrazów initramfs/initrd.

#### Przyjazne nazwy

By obrazy initramfs/initrd miały nieco bardziej dające przewidzieć się nazwy i do tego niezmienne w
czasie, musimy zaprzęgnąć do pracy [skrypty initramfs][16] (nie mylić ze skryptami
`initramfs-tools` ). Skrypty `initramfs` rezydują w katalogu `/etc/initramfs/` i oferują one bardzo
ciekawą funkcjonalność, z której będziemy robić użytek. Niemniej jednak, w domyślnej instalacji
systemu RasPiOS/Raspbian (czy nawet na Debianie) próżno takiego folderu szukać. Trzeba zatem
stworzyć katalog `/etc/initramfs/` , a w nim podkatalog `post-update.d/` :

    # mkdir -p /etc/initramfs/post-update.d/

Następnie tworzymy skrypt `rpi-initrd` oraz nadajemy mu prawa wykonywania:

    #  touch /etc/initramfs/post-update.d/rpi-initrd
    #  chmod +x /etc/initramfs/post-update.d/rpi-initrd

Do pliku `rpi-initrd` wrzucamy poniższą zawartość:

    #!/bin/bash

    ABI="$1"
    INITRD="$2"
    BOOTDIR="$(dirname "$INITRD")"

    # Note: the match below _must_ be synced with the boot kernel
    if [[ "$ABI" == *-v7l+ ]]; then
            echo "Building v7l+ image, updating top initrd"
            cp -p "$INITRD" "$BOOTDIR/initrd.img"
    fi

W tym przypadku mamy do czynienia z 32-bitowym OS, zatem ABI kernela trzeba ustawić na `-v7l+` .
Gdybyśmy mieli 64 bitowy system w RPI, to wtedy ABI trzeba by ustawić na `-v8+` . Nie trzeba się
tych wartości uczyć na pamięć, bo można je sobie wyprowadzić z nazwy obrazu initramfs/initrd
generowanego domyślnie przez system Raspberry Pi. Przykładowo, jeśli mamy plik
`initrd.img-5.10.63-v7l+` , to ABI tutaj znajduje się po ostatnim myślniku, czyli `-v7l+` .

Po ponownym wygenerowaniu obrazu initramfs/initrd, w terminalu powinniśmy zauważyć dodatkowy
komunikat `Building v7l+ image, updating top initrd` :

    root@raspberrypi:/home/pi# update-initramfs -u -k $(uname -r)
    ln: failed to create hard link '/boot/initrd.img-5.10.63-v7l+.dpkg-bak' => '/boot/initrd.img-5.10.63-v7l+': Operation not permitted
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    Building v7l+ image, updating top initrd

Pojawiła się również informacja, że nie udało się stworzyć twardego dowiązania
`/boot/initrd.img-5.10.63-v7l+.dpkg-bak` . Ten komunikat dotyczy poruszanej wcześniej kwestii
braku wsparcia dla linków w systemie plików FAT. System tutaj próbował zrobić backup obrazu
starego initramfs/initrd ale bez tych linków najwyraźniej ten krok się nie powiedzie. W niczym to
nam jednak nie przeszkadza.

Gdy teraz zajrzymy do katalogu `/boot/` , to znajdziemy tam dwa obrazy initramfs/initrd:

    root@raspberrypi:/home/pi# ls -hal /boot | grep -i initrd
    -rwxr-xr-x  1 root root  17M Nov  7 21:10 initrd.img-5.10.63-v7l+
    -rwxr-xr-x  1 root root  17M Nov  7 21:10 initrd.img

Plik z nazwą `initrd.img` przyda nam się później.

### Plik /etc/initramfs-tools/initramfs.conf

Kolejnym krokiem jest edycja pliku `/etc/initramfs-tools/initramfs.conf` . Upewnijmy się, że w
wygenerowanym obrazie initramfs/initrd znajdą się niezbędne moduły do obsługi systemu plików i
dysków twardych. Dodatkowo potrzebny nam będzie `busybox` oraz odpowiedni układ klawiatury (dla
wpisywania hasła).

Reasumując, w tym powyższym pliku powinniśmy mieć ustawione te poniższe opcje:

    MODULES=most
    BUSYBOX=y
    KEYMAP=y

### Plik /boot/config.txt

Kolejnym plik, który trzeba poddać edycji to `/boot/config.txt` . Trzeba w nim dodać parametr
`initramfs` oraz podać mu dwie wartości. Pierwszą z nich jest nazwa obrazu initramfs/initrd (w
katalogu `/boot/` ) , drugą zaś adres pamięci RAM, w który zostanie wgrany obraz, przykładowo:

    initramfs initrd.img followkernel

W tym przypadku skorzystaliśmy z `followkernel` jako adres pamięci operacyjnej, co jest
równoznaczne z adresem `0x0` , czyli obraz initramfs/initrd zostanie wgrany do pamięci RAM
bezpośrednio za obrazem kernela. Więcej informacji na temat opcji w pliku `/boot/config.txt` [można
znaleźć tutaj][6].

### Plik /boot/cmdline.txt

Idąc dalej, musimy nieco przerobić linijkę kernela, która w przypadku Raspberry Pi znajduje się w
pliku `/boot/cmdline.txt` . Domyślnie mamy tutaj te poniższe parametry:

    console=serial0,115200 console=tty1 root=PARTUUID=0d03b705-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles

Usuwamy z tej linijki parametry `quiet` , `splash` oraz `plymouth.ignore-serial-consoles` . Te
parametry nam nie są w ogóle potrzebne i tylko przeszkadzają w diagnozie ewentualnych problemów.
Potrzebny nam za to będzie parametr `loglevel=4` oraz `break=premount` . Zatem przerobiony kernel
cmdline powinien wyglądać mniej więcej tak:

    console=serial0,115200 console=tty1 root=PARTUUID=0d03b705-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait loglevel=4 break=premount

Parametr `break=premount` przyda nam się jedynie do celów testowych, tj. do sprawdzenia czy te
wszystkie powyższe ustawienia odnośnie obrazu initramfs/initrd działają prawidłowo.

### Regeneracja obrazu initramfs/initrd

Gdy skończymy przerabiać pliki konfiguracyjne, musimy na nowo wygenerować obraz initramfs/initrd.
Robimy to przy pomocy znanego nam już polecenia `update-initramfs` i podobnie jak wcześniej, nie
przejmujemy się żadnym z błędów, które mogliśmy wcześniej zaobserwować.

### Testowanie wsparcia dla initramfs/initrd w RPI

Restartujemy teraz naszą maszynkę z Raspberry Pi i jeśli wszystkie powyższe kroki przeprowadziliśmy
prawidłowo, to podczas startu systemu powinniśmy zostać zrzuceni do obrazu initrd/initramfs.
Poznamy to po prompt, w którym widnieje fraza `initramfs` , tak jak to zostało zobrazowane na
poniższej fotce:

![raspberry-pi-rpi-luks-encrypt-initramfs-initrd-test](/img/2021/11/002.raspberry-pi-rpi-luks-encrypt-initramfs-initrd-test.jpg#medium)

Jeśli taki prompt udało się nam uzyskać, to znaczy, że możemy przejść do dalszej części artykułu,
tj. do zaszyfrowania partycji `/` na karcie SD.

Procedura startu systemu, którą zainicjowaliśmy może być kontynuowana. Trzeba tylko w terminalu
wydać polecenie `exit` . Po uruchomieniu systemu, parametr `break=premount` z kernel cmdline w
pliku `/boot/cmdline.txt` możemy usunąć, bo nie będzie on nam już w zasadzie do niczego potrzebny.
Można naturalnie go zostawić aż do momentu zaszyfrowania karty SD, bo w obrazie initramfs/initrd
znajduje się binarka `busybox` i szereg użytecznych skryptów oraz innych narzędzi, przy pomocy
których to ręcznie możemy próbować odratować system Raspberry Pi bez potrzeby przekładania karty SD
do innego urządzenia (taki dość bardzo prymitywny system live).

## Szyfrowanie partycji głównej na karcie SD

W tym przypadku mamy możliwość wyjęcia karty SD z Raspberry Pi i podłączenia jej do komputera. Nie
zawsze jednak taki krok będzie możliwy i w takiej sytuacji trzeba będzie zaprzęgnąć dodatkowy
pendrive podłączony do portu USB w Raspberry Pi, z którego to będziemy mogli uruchomić system --
coś mniej więcej jak system live, z tą różnicą, że tutaj na pendrive będziemy mieli pełnowymiarowy
system RasPiOS/Raspbian.

Nie musimy klonować systemu z flash'a urządzenia i takie praktyki nie są zalecane. Chodzi o to, że
po sklonowaniu partycje będą mieć takie same numerki identyfikacyjne i próba uruchomienia systemu z
dwoma takimi nośnikami naraz zawsze będzie prowadzić do poważnych problemów. Najlepiej wgrać sobie
czysty system RasPiOS/Raspbian na pendrive i z niego uruchomić Raspberry Pi w celu przeprowadzenia
dalszych prac pod katem zaszyfrowania danych obecnych na wbudowanym flash'u RPI.

Na karcie SD z wgranym systemem RasPiOS/Raspbian znajdują się dwie partycje: `/` oraz `/boot/` .
Partycję `/` chcemy poddać szyfrowaniu z wykorzystaniem linux'owego mechanizmu LUKS (dm-crypt). Nie
możemy jednak tego zrobić, gdy system RasPiOS/Raspbian jest uruchomiony z karty SD włożonej do
Raspberry Pi (czy też tego wbudowanego flash'a). Ta karta SD nie może być używana przez system
podczas przeprowadzania procesu szyfrowania. Dlatego właśnie trzeba będzie tę kartę z RPI wyciągnąć,
a gdy nie mamy takiej możliwości, to trzeba się posiłkować dodatkowym pendrive.

W tym przypadku karta SD została wyciągnięta z Raspberry Pi i podłączona do mojego laptopa z
Debianem na pokładzie. Zatem cały proces szyfrowania partycji `/` będzie przeprowadzany na maszynie
z linux'em, w którym pakiet `cryptsetup` również trzeba będzie doinstalować. Niemniej jednak,
wszystkie opisane poniżej kroki bez większego problemu można będzie zaaplikować do wersji z
wbudowanym flash'em, choć zapewne będzie trzeba je ździebko dostosować.

Jeśli zaś chodzi o partycję `/boot/` , to w zasadzie nie będziemy jej ruszać.

### Zgranie danych z karty SD przy pomocy rsync

Pierwszym krokiem na drodze do zaszyfrowania plików na karcie SD jest zgranie danych, które
rezydują w obrębie partycji `/` . Podpinamy zatem kartę SD do komputera, montujemy system plików
tej partycji (np. w katalogu `/media/rpi/` ) i zgrywamy pliki z karty SD via `rsync` :

    # mkdir /media/{rpi,rpi-back}/
    # mount /dev/mmcblk0p2 /media/rpi
    # rsync -axHAWXS --numeric-ids --info=progress2 /media/rpi/ /media/rpi-back/

Po zakończonym procesie synchronizacji trzeba odmontować system plików karty SD:

    # umount /media/rpi

## Tworzenie zaszyfrowanego kontenera

Przyszła pora na utworzenie zaszyfrowanego kontenera LUKS na partycji `/` . Nie ma potrzeby
wcześniejszego usuwania tej partycji i tworzenia jej na nowo, bo `cryptsetup` sobie wszystko sam
przygotuje. Jedyne co nam jest potrzebne do szczęścia, to poniższe polecenie:

    # cryptsetup luksFormat /dev/mmcblk0p2 \
      --type luks2 \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --pbkdf argon2i \
      --pbkdf-force-iterations 4 \
      --pbkdf-memory 524288 \
      --pbkdf-parallel 2 \
      --label rpi \
      --subsystem "" \
      --use-random \
      --verify-passphrase \
      --verbose

    WARNING: Device /dev/mmcblk0p2 already contains a 'ext4' superblock signature.

    WARNING!
    ========
    This will overwrite data on /dev/mmcblk0p2 irrevocably.

    Are you sure? (Type 'yes' in capital letters): YES
    Enter passphrase for /dev/mmcblk0p2:
    Verify passphrase:
    Existing 'ext4' superblock signature on device /dev/mmcblk0p2 will be wiped.
    Key slot 0 created.
    Command successful.

Mamy tutaj kilka ostrzeżeń, że na wskazanej wyżej partycji mamy sygnaturę superbloku systemu plików
ext4 oraz, że zostanie ona wyczyszczona. Dlatego też sprawdźmy ścieżkę do urządzenia i upewnijmy
się dwukrotnie, że to właśnie to urządzenie ma zostać sformatowane.

Następnie otwieramy kontener pod dowolną nazwą ale ta nazwa będzie miała kluczowe znaczenie w
późniejszej części artykułu. W tym przypadku kontener został otworzony pod nazwą `rpi_crypt` :

    # cryptsetup luksOpen /dev/mmcblk0p2 rpi_crypt
    Enter passphrase for /dev/mmcblk0p2:

    # ls -al /dev/mapper/rpi_crypt
    lrwxrwxrwx 1 root root 8 2021-11-07 02:24:19 /dev/mapper/rpi_crypt -> ../dm-11

### AES vs. Google Adiantum

Procesor w Raspberry Pi nie posiada wsparcia dla instrukcji AES i szyfrowanie z wykorzystaniem tego
algorytmu nie należy do najbardziej wydajnych. [Niektórzy zalecają][9] korzystanie z szyfru [Google
Adiantum][10], który wykorzystuje algorytm ChaCha. Z informacji zawartych na wiki można wyczytać,
że szyfr Adiantum został opracowany z myślą o urządzeniach niskiej mocy, tj. głównie te działające
pod kontrolą Androida. Adiantum z powodzeniem nada się zatem również i w przypadku Raspberry Pi.

Niemniej jednak, w obrębie partycji `/` na Raspberry Pi raczej za dużo operacji zapisu (bo to one
najbardziej cierpią przy szyfrowaniu) nie będziemy przeprowadzać. Osobną sprawa jest  prędkość
samego flash'a RPI. Jeśli mamy pamięć flash z górnej półki, to faktycznie w takim przypadku
korzystanie z AES mijałoby się z celem. Niemniej jednak, większość systemów RPI działa na flash'u,
który prędkość zapisu ma na poziomie 10M/s i niższej, przez co raczej nie zauważymy większej
różnicy między AES czy Adiantum.

Jeśli jednak zaliczamy się do tej niewielkiej grupy ludzi, którzy mają lepszej jakości flash w
swoim Raspberry Pi, to lepszym rozwiązaniem będzie skorzystanie z tego poniższego polecenia przy
tworzeniu zaszyfrowanego kontenera LUKS, które robi użytek z Adiantum zamiast AES:

    # cryptsetup luksFormat /dev/mmcblk0p2 \
      --type luks2 \
      --cipher xchacha20,aes-adiantum-plain64 \
      --key-size 256 \
      --hash sha512 \
      --pbkdf argon2i \
      --pbkdf-force-iterations 4 \
      --pbkdf-memory 524288 \
      --pbkdf-parallel 2 \
      --label rpi \
      --subsystem "" \
      --use-random \
      --verify-passphrase \
      --verbose

Jeśli mamy wyjątkowo słabą maszynę, wartym rozważenia jest skorzystanie z szyfru
`xchacha12,aes-adiantum-plain64` . Ten szyfr nie jest jednak tak bezpieczny jak
`xchacha20,aes-adiantum-plain64` ale wciąż raczej nie do złamania, przynajmniej jeśli chodzi o
perspektywę kilku (może kilkunastu) następnych lat.

Poniżej znajduje się benchmark dla tych trzech szyfrów przeprowadzony z poziomu Raspberry Pi 4B:

    pi@raspberrypi:~ $ cryptsetup benchmark -c aes-xts-plain64 -s 512
    # Tests are approximate using memory only (no storage IO).
    # Algorithm |       Key |      Encryption |      Decryption
        aes-xts        512b        60.4 MiB/s        55.8 MiB/s

    pi@raspberrypi:~ $ cryptsetup benchmark -c xchacha20,aes-adiantum-plain64
    # Tests are approximate using memory only (no storage IO).
    #            Algorithm |       Key |      Encryption |      Decryption
    xchacha20,aes-adiantum        256b        84.0 MiB/s        97.1 MiB/s

    pi@raspberrypi:~ $ cryptsetup benchmark -c xchacha12,aes-adiantum-plain64
    # Tests are approximate using memory only (no storage IO).
    #            Algorithm |       Key |      Encryption |      Decryption
    xchacha12,aes-adiantum        256b       104.5 MiB/s       110.4 MiB/s

Powyższe wyniki są dla takiego samego rozmiaru klucza szyfrującego, tj. 256 bitów ([AES w trybie
XTS dzieli klucz na połowę][14] i efektywnie mamy w jego przypadku też klucz o długości 256 bitów,
a nie 512). Jak widać z powyższych testów, przy `xchacha12,aes-adiantum-plain64` mamy tak około
dwukrotnie wyższą wydajność w stosunku do AES. Ten nieco bezpieczniejszy szyfr
`xchacha20,aes-adiantum-plain64` jest trochę wolniejszy ale te wartości i tak są grubo powyżej
10M/s i jeśli mamy wolną kartę SD, to wszystko jedno, który z tych trzech szyfrów sobie wybierzemy.

### System plików ext4

Po stworzeniu kontenera i jego otworzeniu, w systemie pojawi się nowe urządzenie
`/dev/mapper/rpi_crypt` . To urządzenie formatujemy systemem plików ext4:

    # mke2fs \
        -t ext4 \
        -m 0 \
        -L rootfs \
        -J size=128 \
        -O 64bit,has_journal,extents,huge_file,flex_bg,metadata_csum,dir_nlink,extra_isize,^resize_inode,^uninit_bg \
        -E lazy_itable_init=0,lazy_journal_init=0 \
        /dev/mapper/rpi_crypt

    mke2fs 1.46.4 (18-Aug-2021)
    Creating filesystem with 7486464 4k blocks and 1872304 inodes
    Filesystem UUID: 0ca2062b-142b-4826-bb74-d465ca89b554
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
            4096000

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done

### Przenoszenie danych karty SD do kontenera LUKS

Mając utworzony system plików ext4 w zaszyfrowanym kontenerze, możemy przenieść do niego uprzednio
zgrane z karty SD pliki. W tym przypadku również korzystamy z `rsync` :

    # mount /dev/mapper/rpi_crypt /media/rpi
    # rsync -axHAWXS --numeric-ids --info=progress2 /media/rpi-back/ /media/rpi/

## Dostosowanie konfiguracji Raspberry Pi

Po przeniesieniu plików na ich pierwotne miejsce, musimy jeszcze dostosować kilka rzeczy jeśli
chodzi o konfigurację samego systemu Raspberry Pi. Gdybyśmy tę kartę SD w tym momencie włożyli do
RPI, to system zwyczajnie by nam się nie uruchomił, bo nie widziałby jak ma czytać zaszyfrowaną
partycję `/` . Dlatego też zanim zamkniemy kontener i odłączymy kartę SD od komputera, trzeba
dokonać edycji kilku kluczowych plików.

### Plik /etc/fstab

Pierwszym plikiem na który musimy rzucić okiem to `/etc/fstab` . Domyślnie u mnie miał on poniższą
postać:

    proc                  /proc           proc    defaults          0       0
    PARTUUID=0d03b705-01  /boot           vfat    defaults          0       2
    PARTUUID=0d03b705-02  /               ext4    defaults,noatime  0       1
    # a swapfile is not a swap partition, no line here
    #   use  dphys-swapfile swap[on|off]  for that

Interesuje nas póki co wpis z `/` . Trzeba tutaj dostosować numerek identyfikacyjny partycji oraz
też po części opcje dla systemu plików. Jak widać wyżej, w Raspberry Pi partycje są domyślnie
dopasowywane po `PARTUUID` . Po tych zabiegach z utworzeniem zaszyfrowanego kontenera LUKS, te
numerki się w żaden sposób nie zmienią, co możemy zaobserwować na poniższym listingu:

    # lsblk -o "NAME,SIZE,FSTYPE,TYPE,LABEL,MOUNTPOINT,UUID,PARTUUID" /dev/mmcblk0
    NAME           SIZE FSTYPE      TYPE  LABEL  MOUNTPOINT UUID                                 PARTUUID
    mmcblk0       28.8G             disk
    ├─mmcblk0p1    256M vfat        part  boot              B05C-D0C4                            0d03b705-01
    └─mmcblk0p2   28.6G crypto_LUKS part  rpi               0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be 0d03b705-02
      └─rpi_crypt 28.6G ext4        crypt rootfs /media/rpi 0ca2062b-142b-4826-bb74-d465ca89b554

Niemniej jednak, w pliku `/etc/fstab` trzeba by umieścić PARTUUID partycji, na której znajduje się
główny system plików. PARTUUID odnosi się jedynie do fizycznych partycji (tych obecnych w tablicy
partycji danego nośnika) i dlatego też przy pozycji `rpi_crypt` w `lsblk` nie mamy żadnego numerka
w ostatniej kolumnie. Nie możemy zatem zidentyfikować tego systemu plików po PARTUUID. Trzeba wiec
skorzystać z `UUID` .

Reasumując, plik `/etc/fstab` musimy przepisać do poniższej postaci:

    proc                                       /proc           proc    defaults          0       0
    UUID=0ca2062b-142b-4826-bb74-d465ca89b554  /               ext4    defaults,lazytime,errors=remount-ro  0       1
    PARTUUID=0d03b705-01                       /boot           vfat    defaults          0       2
    # a swapfile is not a swap partition, no line here
    #   use  dphys-swapfile swap[on|off]  for that

Jeśli zaś chodzi o opcje dla tego systemu plików, które zostały określone wyżej, to na uwagę w
zasadzie zasługuje jedynie [lazytime][8]. W mojej ocenie, w przypadku pamięci flash, `lazytime`
nadaje się o wiele lepiej niż ten domyślny `noatime` , bo drastycznie redukuje zapisy do tablicy
i-węzłów, tj. aktualizacja czasów `atime` , `mtime` oraz `ctime` odbywa się tylko na obecnej w
pamięci RAM wersji i-węzła danego pliku. Co jakiś czas te czasy są naturalnie
synchronizowane. [Więcej o lazytime można poczytać tutaj][7].

### Plik /etc/crypttab

W pliku `/etc/crypttab` trzeba utworzyć wpis wskazujący lokalizację kontenera. Kluczowe znaczenie
ma nazwa kontenera, pod którą wcześniej został on otworzony. W tym przypadku kontener LUKS został
otworzony jako `rpi_crypt` . Dalej, musimy też określić UUID kontenera, który możemy uzyskać z
wyjścia `lsblk` . Dodatkowo w opcjach trzeba podać `initramfs` , który wskaże skryptom z pakietu
`initramfs-tools` , by to urządzenie było przetwarzane w fazie initramfs/initrd, co z kolei wymusza
dodanie do obrazu initramfs/initrd wszystkich niezbędnych plików i narzędzi.

Reasumując, w pliku `/etc/crypttab` potrzebujemy linijki na wzór tej poniższej:

    # <target name>  <source device>  <key file>  <options>
    rpi_crypt  UUID=0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be   none  luks,initramfs

#### Kwestia pliku /etc/cryptsetup-initramfs/conf-hook

W niektórych tutorialach poświęconych szyfrowaniu Raspberry Pi można wyczytać, by poddać edycji
plik `/etc/cryptsetup-initramfs/conf-hook` i odkomentować w nim tę poniższą linijkę:

    #CRYPTSETUP=y

To rozwiązanie jest już obecnie przestarzałe i zostało zastąpione tym opisanym wyżej, tj. przez
opcję `initramfs` w pliku `/etc/crypttab` . Nie ma zatem już potrzeby edytowania pliku
`/etc/cryptsetup-initramfs/conf-hook` , bo gdy system wykryje, że ma do czynienia z zaszyfrowanym
nośnikiem danych, to stosowne narzędzia automatycznie załaduje do obrazu initramfs/initrd, co
możemy zweryfikować ręcznie podglądając zawartość tego obrazu:

    # lsinitramfs /media/rpi/boot/initrd.img-5.10.63-v7l+ | grep -i cryptsetup
    usr/lib/arm-linux-gnueabihf/libcryptsetup.so.12
    usr/lib/arm-linux-gnueabihf/libcryptsetup.so.12.4.0
    usr/lib/cryptsetup
    usr/lib/cryptsetup/askpass
    usr/lib/cryptsetup/functions
    usr/sbin/cryptsetup

### Plik /boot/cmdline.txt

Musimy także poddać edycji plik `/boot/cmdline.txt` , w którym to trzeba zmienić parametr `root=` ,
tak by wskazywał na UUID (czy też ścieżkę w katalogu `/dev/mapper/` ) voluminu `rpi_crypt` ,
przykładowo:

    console=serial0,115200 console=tty1 root=/dev/mapper/rpi_crypt rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait loglevel=4

#### Parametr cryptdevice

Gdzieniegdzie można spotkać się z dodawaniem do kernel cmdline również i parametru `cryptdevice` ,
tj. coś na wzór `cryptdevice=/dev/mmcblk0p2:rpi_crypt` . Ten parametr nie jest jednak opisany na
[liście znanych parametrów kernela][12] i wygląda raczej na specyficzny dla niektórych dystrybucji
linux'a, np. [Archlinux][13]. Dystrybucja Debian i jej pochodne, w tym też RasPiOS/Raspbian, bez
problemu potrafią się bez tego parametru obejść i nie ma potrzeby go dodawać do linijki kernela.

### Regeneracja obrazu initramfs/initrd

Po dostosowaniu tych powyższych plików, trzeba jeszcze raz wygenerować obraz initramfs/initrd. Nie
możemy jednak tego zrobić z poziomu działającego Raspberry Pi, bo nam on się przecież nie uruchomi
jeszcze. Trzeba zaprzęgnąć do pracy mechanizm `chroot` .

Musimy pierw przygotować środowisko pod chroot, tj. zamontować w strukturze docelowego systemu
kilka dodatkowych katalogów. Najprościej do tego celu skorzystać z tego poniższego polecenia:

    # for f in dev dev/pts sys proc run ; do mount -o bind /$f /media/rpi/$f ; done

Jako, że obraz initramfs/initrd rezyduje w obrębie partycji `/boot/` , to tę partycję również
musimy udostępnić w środowisku chroot:

    # mount /dev/mmcblk0p1 /media/rpi/boot/

Ostatnim krokiem jest przełączenie się do docelowego systemu przez wydanie polecenia `chroot` :

    # chroot /media/rpi/ /bin/bash

Powinniśmy być teraz wewnątrz systemu Raspberry Pi, co możemy poznać po zmianie prompt'a w
terminalu.

Generujemy obraz initramfs/initrd:

    # update-initramfs -u -k 5.10.63-v7l+

Po wygenerowaniu obrazu możemy opuścić środowisko chroot:

    # exit

Trzeba pamiętać, by odmontować wszystkie podmontowane wcześniej na potrzeby chroot katalogi:

    # for f in dev/pts dev sys proc run ; do umount /media/rpi/$f ; done

Odmontujmy także partycję `/boot/` oraz `/` i zamknijmy kontener LUKS:

    # umount /media/rpi/boot
    # umount /media/rpi/
    # cryptsetup luksClose rpi_crypt

#### chroot: failed to run command ‘/bin/bash’: Exec format error

Być może zdarzy się nam taka sytuacja, że podczas wydawania polecenia `chroot` zostanie nam
zwrócony poniższy błąd:

    # chroot /media/rpi /bin/bash
    chroot: failed to run command ‘/bin/bash’: Exec format error

Ten komunikat bierze się z faktu, że Raspberry Pi ma procesor oparty o architekturę ARM, podczas
gdy przeciętny desktop czy laptop ma procesor od Intel/AMD i by polecenie `chroot` nam zadziałało
[musimy skorzystać z emulatora QEMU][11].

## Test zaszyfrowanego kontenera LUKS

Wyciągamy zatem już kartę SD z portu USB komputera, wkładamy ją do naszego Raspberry Pi i
uruchamiamy urządzenie. System powinien nam się uruchomić do momentu, w którym na ekranie
zostaniemy poproszeni o wpisanie hasła do zaszyfrowanego kontenera LUKS, co wygląda mniej więcej
tak:

![raspberry-pi-rpi-luks-encrypt-unlock-luks-password](/img/2021/11/003.raspberry-pi-rpi-luks-encrypt-unlock-luks-password.jpg#huge)

Po wpisaniu hasła, start systemu będzie przebiegał w standardowy sposób:

![raspberry-pi-rpi-luks-encrypt-unlock-luks-system-start](/img/2021/11/004.raspberry-pi-rpi-luks-encrypt-unlock-luks-system-start.jpg#huge)

## Podsumowanie

No jak widać, system Raspberry Pi bez większego problemu można zamknąć w zaszyfrowanym kontenerze
LUKS. Pytanie jakie się nasuwa, to czy takie rozwiązanie ma jakikolwiek sens? Komputery osobiste
mamy zwykle pod ręką i w razie czego możemy je wyłączyć, bo tylko wtedy szyfrowanie może spełniać
swoje zadanie. Takie systemy są też zwykle stale przez nas monitorowane, np. mamy je w plecaku czy
innej torbie. A co w przypadku takich malinek? Mogą one chodzić wiele dni czy nawet tygodni bez
restartu i pozostawiamy je w zasadzie bez jakiegokolwiek nadzoru przez większość czasu. W takich
sytuacjach szyfrowanie może nam dać jedynie złudne poczucie bezpieczeństwa i sprawić, że będziemy
na RPI przechowywać dane, których tam nie powinniśmy trzymać. Z racji wdrożenia szyfrowania trzeba
też będzie liczyć się ze spowolnieniem pracy głównego procesora, choć raczej nie powinniśmy tego
jakoś szczególnie odczuć, zwłaszcza gdy do dyspozycji mamy kartę SD o przeciętnej wydajności.
Problematyczne może też być wpisywanie hasła, by kartę SD odszyfrować, wszak większość
minikomputerów Raspberry Pi działa bez podpiętej klawiatury/myszy, a nierzadko też i przy braku
monitora. Tę kwestię wpisywania hasła można rozwiązać ale jest to zagadnienie na inny artykuł, bo
ten już i tak wyszedł dość długi.


[1]: https://www.raspberrypi.com/software/operating-systems/
[2]: https://raspberrypi.stackexchange.com/questions/92557/how-can-i-use-an-init-ramdisk-initramfs-on-boot-up-raspberry-pi
[3]: https://forums.raspberrypi.com/viewtopic.php?t=241313
[4]: /post/jak-przepisac-linki-initrd-img-old-i-vmlinuz-old-do-boot/
[5]: https://forums.raspberrypi.com/viewtopic.php?t=58502
[6]: https://www.raspberrypi.com/documentation/computers/config_txt.html
[7]: https://lwn.net/Articles/621046/
[8]: https://man7.org/linux/man-pages/man8/mount.8.html
[9]: https://forums.raspberrypi.com/viewtopic.php?f=63&t=252980&p=1543723#p1543753
[10]: https://en.wikipedia.org/wiki/Adiantum_(cipher)
[11]: /post/chroot-do-32-bit-systemu-arm-z-poziomu-64-bit-linuxowego-hosta/
[12]: https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html
[13]: https://wiki.archlinux.org/title/Dm-crypt/System_configuration#cryptdevice
[14]: https://en.wikipedia.org/wiki/Disk_encryption_theory#XEX-based_tweaked-codebook_mode_with_ciphertext_stealing_.28XTS.29
[15]: https://k1024.org/posts/2021/2021-01-25-raspbian-with-initrd/
[16]: https://kernel-team.pages.debian.net/kernel-handbook/ch-update-hooks.html#s-initramfs-hooks
