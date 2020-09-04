---
author: Morfik
categories:
- Android
date: "2017-02-19T21:06:04Z"
date_gmt: 2017-02-19 20:06:04 +0100
published: true
status: publish
tags:
- smartfon
- twrp
- marshmallow
- omni-rom
title: Budowanie obrazu TWRP recovery ze źródeł OMNI ROM
---

Gdy zamierzamy zbudować sobie własny ROM na smartfon z Androidem, np.
[LineageOS](https://lineageos.org/) (CyanogenMod nie jest już rozwijany) czy nawet jedynie obraz
recovery ([TWRP](https://twrp.me/) albo [CWM](https://www.clockworkmod.com/)), to potrzebne nam jest
stosowne urządzenie oraz odpowiedni kod źródłowy. Skoro chcemy budować te ww. rzeczy, to
prawdopodobnie nasz telefon nie jest przez to oprogramowanie jeszcze wspierany lub też sam soft nie
jest regularnie aktualizowany przez dewelopera. W zasadzie zarówno pełne ROM'y jak i obrazy recovery
są budowane ze źródeł Androida. Niemniej jednak, oficjalny kod dostarczany przez Google budzi czasem
wiele kontrowersji i ci nieco bardziej zaawansowani użytkownicy zmieniają go, np. czyniąc go w pełni
OpenSource czy też implementując w nim pewną niestandardową funkcjonalność. Tak powstają Custom
ROM'y, które w późniejszym czasie z racji swojej popularności przestają być "Custom" i zaczynają żyć
swoim własnym życiem obok tego Góglowskiego Androida. W przypadku budowania obrazu recovery nie są
nam potrzebne całe źródła konkretnego ROM'u. Jakby nie patrzeć, potrafią one zajmować trochę
miejsca, a poza tym proces ich budowania jest stosunkowo czasochłonny. Tak czy inaczej, jakieś
źródła trzeba pozyskać i przygotować je do dalszej pracy. W tym artykule nie będziemy sobie
jeszcze budować całego ROM'u i skupimy się na zbudowaniu od podstaw jedynie obrazu TWRP recovery ze
źródeł [OMNI ROM](https://omnirom.org/). Ten proces zostanie pokazany na przykładzie smartfona
Neffos Y5 od TP-LINK przy wykorzystaniu systemu linux, a konkretnie dystrybucji Debian.

<!--more-->
## Środowisko 64-bit

Zgodnie z [informacjami jakie można znaleźć tutaj](https://source.android.com/source/requirements),
budowanie kodu Androida począwszy od wersji 2.3.x (Gingerbread) wymaga 64 bitowego systemu. Warto o
tym pamiętać, by uniknąć problemów przy przygotowywaniu środowiska pod kompilację. Dobrze jest też
posiadać maszynę z 8 GiB pamięci RAM.

## Android SDK

Operowanie na oprogramowaniu, które mamy w naszych smartfonach, wymaga zainstalowania na komputerze
pakietu Android SDK. W nim zawarte są narzędzia deweloperskie min. `fastboot` i `adb` , przy pomocy
których będziemy w stanie przeprowadzić szereg akcji na smartfonie. Te narzędzia można zainstalować
w systemie na kilka sposobów.

Standardowo `fastboot` i `adb` są dostępne w repozytoriach Debiana w pakietach `android-tools-adb`
oraz `android-tools-fastboot` . [Proces instalacji i konfiguracji tych narzędzi na
Debianie]({{< baseurl >}}/post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/) został opisany
w osobnym wątku. Niemniej jednak, nie są to wszystkie narzędzia, które Android SDK dostarcza, a
biorąc pod uwagę fakt, że obecnie w Debianie panuje spory nieporządek w pakietach, to lepiej
[zainstalować Android SDK lub Android
Studio]({{< baseurl >}}/post/android-studio-i-android-sdk-pod-linux/) bezpośrednio ze strony
Google.

## Narzędzia repo i git

Do pobrania źródeł Androida jest wykorzystywane dedykowane narzędzie `repo` . W dystrybucji Debiana
mamy taki pakiet i możemy go bez większego problemu zainstalować. Problem w tym, że wersja tego
narzędzia może być nieaktualna. Obecnie jest to 1.23 . [Najnowsza wersja repo jest dostępna zawsze
pod tym linkiem](https://storage.googleapis.com/git-repo-downloads/repo). By ją zainstalować ręcznie
w systemie, w terminalu wpisujemy poniższe polecenia:

    # curl https://storage.googleapis.com/git-repo-downloads/repo > /usr/local/bin/repo
    # chmod a+x /usr/local/bin/repo
    # chown root:staff /usr/local/bin/repo

Narzędzie `repo` operuje na `git` i ten pakiet również musimy w systemie sobie zainstalować.
Podstawy operowania na repozytorium GIT musimy znać. Warto zatem rzucić okiem na [dokumentację
git'a](https://git-scm.com/) i ją sobie chociaż powierzchownie przejrzeć.

## Przygotowanie katalogu roboczego pod źródła

Źródła Androida trzeba gdzieś pobrać. Stwórzmy sobie zatem dedykowany katalog roboczy i przejdźmy do
niego:

    $ mkdir /Android/android-src/
    $ cd /Android/android-src/

Od tej chwili wszystkie polecenia będą wydawane w tym właśnie katalogu.

## Inicjowanie repozytorium GIT

Na samym początku trzeba zainicjować lokalne repozytorium. Różne smartfony działają pod kontrolą
innych wersji Androida (Lollipop, Marshmallow, Nougat). Jeśli zamierzamy kompilować jedynie część
modułów, a nie cały ROM, i to głównie dla siebie, to warto zadbać o to, by wersja źródeł Androida
pasowała do wersji Androida, którą mamy w telefonie. Niemniej jednak, w przypadku obrazu TWRP
recovery możemy korzystać z najnowszych źródeł, bo w zasadzie nie będziemy wgrywać żadnych plików
bezpośrednio na partycję `/system/` , która zawiera stock'owy ROM.

Musimy zatem określić stosowną gałąź repozytorium GIT, którą zamierzamy sobie sklonować na dysk.
[Lista wszystkich gałęzi jest dostępna
tutaj](https://source.android.com/source/build-numbers#source-code-tags-and-builds) i z reguły
Custom ROM'y przestrzegają tego nazewnictwa. Warto też określić parametr `--depth=1` , który uchroni
nas przed pobraniem wszystkich rewizji. Zamiast tego, zostanie pobrany jedynie ostatni snapshot
wskazanej przez nas gałęzi, a to z kolei znacznie zmniejszy ilość danych, które trzeba będzie
przetransferować przez sieć. Bez tej opcji zostałoby pobranych jakieś 30-40 GiB, a może nawet i
więcej).

[Źródła TWRP recovery znajdują się tutaj](https://github.com/omnirom/android_bootable_recovery).
Niektóre ROM'y nie mają zawartego w sobie tego repozytorium i trzeba je ręcznie określić w pliku
`.repo/manifest.xml` w katalogu ze źródłami Androida. My jednak będziemy korzystać ze [źródeł OMNI
ROM](https://github.com/omnirom/android), które już to repozytorium zawiera. Zatem żadnych
dodatkowych kroków nie będziemy musieli przeprowadzać.

Lokalne repozytorium inicjujemy w poniższy sposób:

    $ repo init -u https://github.com/omnirom/android -b android-7.1 --depth=1

Po zainicjowaniu lokalnego repozytorium, w katalogu roboczym powinien pojawić się folder `.repo/` .
W nim z kolei znajduje się plik (właściwie link) `manifest.xml` . Zajrzyjmy do niego i upewnijmy
się, że w `default revision` widnieje numerek wskazanej wyżej wersji Androida ( `android-7.1.x` ).

Warto wspomnieć, że większość repozytoriów zdefiniowanych w pliku `manifest.xml` jest zbędna przy
budowaniu samego obrazu TWRP recovery. Jeśli komuś nie zależy na oszczędzaniu transferu danych oraz
ma dostatecznie dużo miejsca na dysku, to może pobrać całe źródła OMNI ROM. Dla tych, którzy skąpią
miejsca na dysku i transferu jest [okrojona wersja](https://github.com/minimal-manifest-twrp), którą
można z powodzeniem wykorzystać.

Poniżej jest polecenie inicjujące lokalne repozytorium z wykorzystaniem okrojonego pliku
`manifest.xml` .

    $ repo init -u git://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-6.0 --depth=1

![]({{< baseurl >}}/img/2017/02/001.twrp-recovery-budowanie-zrodla-omni-smartfon-inicjacja-repo.png#huge)

W tym przypadku, lokalne repozytorium zostało zainicjowane w oparciu o ten okrojony plik
`manifest.xml` . Trzeba tutaj jednak wyraźnie zaznaczyć, że niektóre konfiguracje sprzętowe (czy też
opcje trybu recovery) będą wymagać dodatkowych repozytoriów, które trzeba będzie ręcznie dodać do
pliku `.repo/local_manifests/local_manifest.xml` . Warto zatem [poznać budowę tego
pliku](https://forum.xda-developers.com/showthread.php?t=2329228) i nauczyć się na nim operować.

Gdy już uporządkujemy sprawy związane z repozytoriami, to przy pomocy `repo sync` synchronizujemy
źródła każdego repozytorium, które mamy zdefiniowane w pliku `manifests.xml` . Naturalnie im
więcej ich mamy, tym więcej miejsca będą one zajmować i dłużej będzie trwał proces synchronizacji.
W przypadku, gdy dysponujemy szybkim łączem internetowym, to możemy też zwiększyć liczbę
jednoczesnych połączeń przy pomocy `--jobs` (domyślnie 4):

    $ repo sync --current-branch --jobs=4

![]({{< baseurl >}}/img/2017/02/002.twrp-recovery-budowanie-zrodla-omni-smartfon-synchronizacja-repo.png#huge)

## Zależności potrzebne do zbudowania Androida ze źródeł

Pobranie źródeł to jedna sprawa, a ich zbudowanie to całkiem inna kwestia. By uniknąć ewentualnych
problemów będziemy musieli zainstalować w systemie kilka dodatkowych zależności. Poniżej jest [lista
potrzebnych rzeczy](https://source.android.com/source/initializing):

    # aptitude install git-core gnupg flex bison gperf build-essential \
      zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
      lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
      libgl1-mesa-dev libxml2-utils xsltproc unzip openjdk-7-jdk

Bez tych pakietów, proces budowy obrazu będzie napotkał błędy podobne do tego [opisanego
tutaj](https://stackoverflow.com/questions/30430251/failed-to-build-android-kernel).

## Przyśpieszenie procesu kompilacji z ccache

Kompilacja to proces czasochłonny i dość zasobożerny. W zasadzie budując ten sam kod kilka razy, nie
ma potrzeby ponawiania całego procesu kompilacji, bo w sporej części przypadków można odwołać się do
już zbudowanych obiektów. Niemniej jednak, potrzebny jest nam jakiś cache dla kompilatora, którego
standardowo w linux'ie nie ma. Dlatego właśnie powstało narzędzie `ccache` , które jest nam w stanie
dość znacznie umilić życie, przy ciągłym budowaniu tych samych programów czy projektów.

Narzędzie `ccache` musimy sobie odpowiednio skonfigurować. Na dobrą sprawę, to jedyne co musimy
zrobić, to dodać te dwie poniższe zmienne do pliku `~/.bashrc` lub `~/.zshrc` :

    export USE_CCACHE=1
    export CCACHE_DIR=/mnt/.ccache

Rozmiar całego cache definiujemy w poniższy sposób (polecenie wywołane z głównego katalogu ze
źródłami Androida):

    $ prebuilts/misc/linux-x86/ccache/ccache -M 50G

Jeśli chcemy się upewnić, czy `ccache` działa prawidłowo, to odpalamy osobny terminal podczas
procesu kompilacji, przechodzimy do katalogu ze źródłami i wydajemy poniższe polecenie:

    $ watch -n1 -d prebuilts/misc/linux-x86/ccache/ccache -s

## Tworzenie konfiguracji dla TWRP recovery pod konkretny model smartfona

W zasadzie źródła już mamy przygotowane do budowania ale potrzebna nam jest jeszcze konfiguracja dla
naszego smartfona. Musimy zatem stworzyć kilka plików i katalogów. Poniższe informacje dotyczą
smartfona Neffos Y5 ale ten krok w zasadzie nie różni się w przypadku innych modeli telefonów.
Jedyne co, to trzeba pozyskać pewne informacje na temat podzespołów urządzenia (min. SoC), dla
którego zamierzamy zbudować obraz TWRP i wpisać te dane w stosowne miejsca.

Najprościej jest zacząć od pliku `/system/build.prop` , który został zawarty w stock'owym firmware
(można też wyciągać zawartość zmiennych przez `getprop` ). Podłączmy zatem na moment smartfon do
komputera i przy pomocy `adb` patrzymy co siedzi w zmiennych `ro.product.manufacturer` oraz
`ro.product.name` :

    # adb shell "getprop ro.product.manufacturer"
    TP-LINK

    # adb shell "getprop ro.product.name"
    TP802A

Przechodzimy teraz do głównego folderu ze źródłami Androida i pod `device/` tworzymy strukturę
katalogów na zasadzie `manufacturer/name` . Wszystkie nazwy mają być pisane z małych liter:

    $ mkdir device/tp-link/tp802a/

W tak powstałym katalogu tworzymy konfigurację dla naszego smartfona, [w tym przypadku Neffos
Y5](https://github.com/morfikov/android_device_tp-link_tp802a). Poniżej znajduje się krótkie
objaśnienie co do struktury plików.

### Plik device.mk

Zaczynamy od utworzenia pliku `device.mk` , gdzie będziemy definiować wszystkie niezbędne moduły i
pliki wymagane do zbudowania obrazu TWRP recovery dla naszego smartfona.

### Plik omni_tp802a.mk

Następnie tworzymy plik `omni_tp802a.mk` , w którym będą deklarowane informacje takie jak nazwa,
model i producent smartfona.

### Plik AndroidProducts.mk

Kolejny plik, który musimy utworzyć, to `AndroidProducts.mk` . W nim zamieszczamy wpis wskazujący na
ten powyższy plik `omni_tp802a.mk` .

### Plik BoardConfig.mk

Plik `BoardConfig.mk` jest w zasadzie sercem całej konfiguracji. To w tym pliku określamy min. to co
tak naprawdę siedzi w naszym smartfonie. Znajdują się tutaj ustawienia platformy sprzętowej,
architektury systemowej, konfiguracja kernela, rozmiary poszczególnych partycji oraz oczywiście
opcje TWRP recovery.

Wszystkie te parametry mogą się różnic w zależności od podzespołów smartfona, z którym mamy do
czynienia. W zasadzie obraz TWRP recovery powinien działać bez zarzutu nawet przy minimalnej
konfiguracji ale w przypadku budowania pełnego ROM'u, to już niestety będziemy musieli nieco
bardziej się postarać, by nasz smartfon działał bez problemu na nowym oprogramowaniu.

Część opcji, które trzeba określić w `BoardConfig.mk` , można wyciągnąć ze smartfona zaglądając do
pliku `/system/build.prop` . Poniżej znajduje się też lekka rozpiska ułatwiająca pozyskanie
informacji dla poszczególnych sekcji.

#### Sekcja Platform

Sekcję `Platform` może nam sprawić najwięcej problemów, zwłaszcza gdy nie mamy za bardzo pojęcia
jakie podzespoły siedzą w naszym smartfonie. W tym przypadku mamy do czynienia z Neffos Y5 i on
posiada SoC MSM8909 od Qualcomm. Konfiguracja konkretnego SoC jest w zasadzie stała i można ją
przepisać ze źródeł innego smartfona, który ma ten sam SoC, zakładając oczywiście, że deweloper nie
popełnił błędów w konfiguracji tamtego urządzenia.

#### Sekcja Kernel

Bawiąc się w ukorzenianie swoich Neffos'ów, jednym z wymaganych etapów było rozmontowanie obrazu
`boot.img` za pomocą [tych skryptów](https://github.com/ndrancs/AIK-Linux-x32-x64/). Podczas tego
procesu, w terminalu można było zanotować poniższe
wyjście:

![]({{< baseurl >}}/img/2017/02/003.twrp-recovery-budowanie-zrodla-omni-smartfon-informacje-kernel.png#huge)

Jak widzimy, praktycznie cała sekcja jest nam podana jak na dłoni i wystarczy uzupełnić stosowne
parametry dodając przed ich wartościami `0x` .

Trzeba tutaj zaznaczyć, że my nie budujemy kernela ze źródeł Androida i załączamy tutaj plik
kernela, który można wyciągnąć po rozmontowaniu stock'owego obrazu `boot.img` .

#### Sekcja Qualcomm

Sekcja `Qualcomm` definiuje szereg opcji specyficznych dla SoC tego producenta. W zasadzie to trzeba
dostosować ścieżkę, która musi wskazywać na bazę platformy naszego smartfona. Ścieżka w tym
przypadku wskazuje na katalog `/sys/devices/soc.0/` ale z pominięciem `/sys/` .

#### Sekcja Encryption

Nowsze wersje TWRP recovery wspierają szyfrowanie/deszyfrowanie partycji `/data/` . Jeśli z poziomu
Androida zaszyfrowaliśmy dane użytkownika, to TWRP standardowo nie będzie w stanie zamontować tej
partycji, co będzie przyczyną całej masy błędów. Dlatego też musimy włączyć moduł szyfrowania.

W zasadzie mamy dwa rodzaje szyfrowania: programowe i sprzętowe. To, które w naszym smartfonie
zostało zaimplementowane możemy odczytać przechodząc w Ustawienia Androida => Zabezpieczenia =>
Typ Pamięci.

![]({{< baseurl >}}/img/2017/02/004.twrp-recovery-budowanie-zrodla-omni-smartfon-informacje-szyfrowanie.png#medium)

W przypadku smartfona Neffos Y5, widzimy, że mamy do czynienia z szyfrowaniem sprzętowym, bo
widnieje zapis "Wspomagana sprzętowo". Gdyby tam widniał zapis "Tylko programowa", to szyfrowanie
byłoby jedynie programowe. Naturalnie szyfrowanie sprzętowe wymaga od nas dodatkowych nakładów
pracy, by je skonfigurować. W zasadzie to trzeba będzie pozyskać szereg plików ze stock'owego
firmware i uwzględnić je w obrazie TWRP. Bez tych plików, nie da rady odszyfrować danych na partycji
`/data/` nawet podając prawidłowe hasło. System za każdym razem zwróci nam taki oto błąd.

![]({{< baseurl >}}/img/2017/02/005.twrp-recovery-budowanie-zrodla-omni-smartfon-blad-deszyfrowanie.png#medium)

To jakie pliki trzeba będzie uwzględnić w obrazie zależy od modelu smartfona. Nie da rady tego
prosto ustalić i w zasadzie pozostaje nam dochodzenie do rozwiązania metodą prób i błędów. Możemy
naturalnie podpierać się logami z trybu recovery, które mogą zawierać nazwy wymaganych bibliotek, a
to już może nam wskazać dobrą drogę.

#### Sekcja Partitions

W oparciu o dane z pliku `/proc/partitions` lub `/proc/emmc` trzeba odpowiednio opisać kilka
partycji smartfona. Chodzi generalnie o partycje `/boot/` , `/recovery/` , `/system/` , `/data/`
oraz `/cache/` . Do odczytania rozmiaru można też zaprzęgnąć różne aplikacje na Androida, np.
diskinfo. Wartości podajemy w HEX.

Możemy także skonfigurować sobie wsparcie dla szeregu dodatkowych systemów plików. Trzeba jednak
pamiętać, że każde dodatkowe ficzery zajmują miejsce i w pewnych konfiguracjach może nam tej
przestrzeni zwyczajnie zabraknąć. Tutaj mamy do dyspozycji 32 MiB i w zasadzie jest to dość sporo.

#### Sekcja Recovery

W sekcji `Recovery` musimy wskazać lokalizację do pliku `fstab` . To przy jego pomocy TWRP będzie w
stanie operować na partycjach smartfona. Ten plik trzeba sobie zbudować samemu podpierając się
wpisami w `/proc/partitions` lub `/proc/emmc` oraz aplikacjami na smartfona typu diskinfo.

#### Sekcja TWRP

W sekcji `TWRP` ustawia się opcje charakterystyczne dla TWRP recovery. Wszystkich opcji jest dość
sporo i są one wyszczególnione [pod tym
linkiem](https://forum.xda-developers.com/showthread.php?t=1943625). W zasadzie to musimy poprawnie
ustawić zmienną `TW_THEME` , która odpowiada za rozdzielczość wyświetlacza smartfona. Do wyboru
mamy:

    # portrait_mdpi  = 320x480 480x800 480x854 540x960
    # portrait_hdpi  = 720x1280 800x1280 1080x1920 1200x1920 1440x2560 1600x2560
    # watch_mdpi     = 240x240 280x280 320x320
    # landscape_mdpi = 800x480 1024x600 1024x768
    # landscape_hdpi = 1280x800 1920x1200 2560x1600

Neffos Y5 ma rozdzielczość 720x1280, dlatego został wybrany `portrait_hdpi` . Może i mamy tutaj do
wyboru dwa tryby wyświetlania (portrait i landscape) ale TWRP nie potrafi się przełączać między nimi
dynamicznie. Jeśli chcemy mieć układ poziomy, to nie da rady przełączyć go w pionowy i vice versa.
Pozostałe opcje mają raczej samo opisujące się nazwy.

### Plik vendorsetup.sh

Za sprawą pliku `vendorsetup.sh` będziemy w stanie dodać nasze urządzenie do budowania. Ten plik
będzie miał jedną linijkę, w której musimy określić dwie rzeczy: `product_name` oraz `variant` .

Jeśli chodzi o `product_name` , to odczytujemy go z pliku `omni_tp802a.mk` ze zmiennej
`PRODUCT_NAME` . Natomiast w `variant` możemy ustawić min. `eng` , `user` albo `userdebug` . Nas
interesuje ta ostatnia opcja oferująca dostęp root i tryb debugowania.

## Budowanie obrazu TWRP recovery

By przygotować środowisko pod budowę źródeł TWRP recovery, musimy wyeksportować szereg zmiennych.
Nie robimy tego jednak ręcznie, a za pomocą pliku `build/envsetup.sh` w poniższy sposób:

    $ make clobber
    $ . ./build/envsetup.sh
    including device/tp-link/tp802a/vendorsetup.sh

Jak widać wyżej, konfiguracja dla naszego smartfona została załączona.

W przypadku korzystania z innego shell'a niż `bash` (w moim przypadku `zsh` ) wydanie tego
powyższego polecenia zwraca takie oto ostrzeżenie:

    $ . ./build/envsetup.sh
    build/envsetup.sh:565: command not found: complete
    WARNING: Only bash is supported, use of other shell would lead to erroneous results

Wygląda na to, że na czas budowy czegoś związanego z Androidem, trzeba korzystać z bash'a.

Teraz możemy wpisać `lunch` i wybrać wcześniej stworzoną kombinację:

    $ lunch

    You're building on Linux

    Lunch menu... pick a combo:
         1. aosp_arm-eng
         2. aosp_arm64-eng
         3. aosp_mips-eng
         4. aosp_mips64-eng
         5. aosp_x86-eng
         6. aosp_x86_64-eng
         7. omni_tp802a-userdebug

    Which would you like?

Mamy na liście `omni_tp802a-userdebug` i to ją musimy wskazać. Możemy także od razu podać tę
pozycję w `lunch` :

    $ lunch omni_tp802a-userdebug

![]({{< baseurl >}}/img/2017/02/006.twrp-recovery-budowanie-zrodla-omni-smartfon-lunch.png#huge)

No i teraz pozostało nam już zbudowanie źródeł. Pamiętajmy tylko, że budujemy jedynie obraz TWRP
recovery, a do tego celu służy poniższe polecenie:

    $ make clean && make -j2 recoveryimage

Po kilku czy kilkunastu minutach, obraz recovery powinien nam się
zbudować:

![]({{< baseurl >}}/img/2017/02/007.twrp-recovery-budowanie-zrodla-omni-smartfon-finish-test.png#huge)

## Repozytorium git na Github'ie

Po zbudowaniu obrazu TWRP sprawdzamy naturalnie czy działa on prawidłowo ładując plik
`out/target/product/tp802a/recovery.img` na smartfon przy pomocy `fastboot boot` . Jeśli nie mamy
zastrzeżeń co do działania trybu recovery, to możemy stworzoną w powyższy sposób konfigurację TWRP
opublikować na GitHub'ie. Oczywiście musimy posiadać stosowne konto i stworzyć odpowiednie
repozytorium. Nazwa tego repozytorium ma wskazywać na ścieżkę drzewa katalogów konfiguracji
urządzenia. Przykładowo, mamy tutaj ścieżkę `source/device/tp-link/tp802a/` , zatem nazwa
repozytorium to `android_device_tp-link_tp802a` :

![]({{< baseurl >}}/img/2017/02/008.twrp-recovery-budowanie-zrodla-omni-smartfon-github-repo.png#huge)

Następnie przechodzimy do katalogu z plikami konfiguracyjnymi, z których zbudowaliśmy obraz TWRP
recovery i inicjujemy w nim lokalne repozytorium:

    $ cd /Android/omni-twrp-6.0/device/tp-link/tp802a/
    $ git init
    $ git remote add origin git@github.com:morfikov/android_device_tp-link_tp802a.git

W pliku `.git/config` dopisujemy sobie te poniższe parametry:

    [user]
        name = Mikhail Morfikov
        email = morfik@nsa.com
        signingkey = 0xCD046810771B6520
    [gpg]
        program = gpg

Następnie dodajemy wszystkie plik, tworzymy commit i nową gałąź, którą nazywamy sobie, np.
`twrp-6.0` :

    $ git add --all
    $ git commit -S -m "first commit"
    $ git branch twrp-6.0
    $ git checkout twrp-6.0
    $ git add --all

Teraz już wystarczy przesłać zmiany do zdalnego repozytorium na GitHub'ie:

    $ git push origin master
    $ git push origin twrp-6.0

I to w zasadzie cała robota. Wszelkie zmiany w repozytorium od tej pory będą rejestrowane przez
system kontroli wersji. [Możemy także przesłać całą
konfigurację](https://twrp.me/faq/OfficialMaintainer.html), tak by TWRP oficjalnie wspierało
naszego smartfona.
