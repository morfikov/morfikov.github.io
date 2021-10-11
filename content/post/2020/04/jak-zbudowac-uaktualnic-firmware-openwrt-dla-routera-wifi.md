---
author: Morfik
categories:
- OpenWRT
date: "2020-04-13T21:03:00Z"
published: true
status: publish
tags:
- router
- kompilacja
- tp-link
GHissueID: 13
title: Jak zbudować/uaktualnić firmware OpenWRT dla routera WiFi
---

Od dłuższego już czasu na swoich routerach WiFi wykorzystuję firmware OpenWRT. W przypadku mojego
domowego routera TP-Link Archer C7 v2 zarządzanie jego oprogramowaniem sprowadza się w zasadzie do
przeprowadzania aktualizacji raz na kilka tygodni czy miesięcy. Z reguły nie jest to jakoś
czasochłonne zadanie, bo wystarczy pobrać stosowny obraz z [serwera eko.one.pl][2] i wrzucić go na
router czy to przez interfejs LuCI, czy też przez `sysupgrade` . No tak tylko, że po wgraniu
OpenWRT na flash routera trzeba zwykle też dograć szereg pakietów, których nie ma w standardzie,
przynajmniej jeśli chodzi akurat o ten mój router bezprzewodowy. Podobnie sprawa ma się z
odtwarzaniem konfiguracji, której pewne elementy pozostają niezmienne nawet po aktualizacji ze
starszego wydania OpenWRT do nowszego. Postanowiłem zatem zgłębić nieco proces kompilacji źródeł i
budowy obrazu z firmware OpenWRT, tak by nieco zautomatyzować sobie (czy też wręcz wyeliminować)
chociaż część z kroków, które zwykle przeprowadzam chwilę po wgraniu obrazu na router. Cały ten
proces budowy obrazu zostanie opisany przy wykorzystaniu dystrybucji Debian linux.

<!--more-->
## Instalowanie potrzebnych zależności

Jako, że w grę wchodzić będzie kompilacja, to musimy odpowiednio przygotować sobie system, by ten
proces przebiegł nam bez większych problemów. Przede wszystkim, w naszym Debianie musimy
[doinstalować szereg pakietów][3]. Ta poniższa lista może być niekompletna. Czasami mogą być
potrzebne również dodatkowe rzeczy, a wszystko zależy od tego co tak naprawdę będziemy chcieli
zbudować. W tym przypadku zainstalowanie tych poniższych zależności zdaje się być optymalnym
rozwiązaniem:

    # aptitude install \
                git subversion build-essential \
                python3 python3-distutils \
                zlib1g-dev libncurses5-dev \
                gawk gettext unzip file \
                libssl-dev wget libelf-dev \
                ecj fastjar binutils \
                bzip2 patch flex

## Pobieranie źródeł OpenWRT

Źródła OpenWRT są utrzymywane w systemie kontroli wersji git i są dostępne pod adresem
[https://git.openwrt.org/openwrt/openwrt.git][4]. Te źródła musimy pobrać na dysk za pomocą
polecenia `git clone` . Same źródła nie ważą sporo, bo jest to 153.67 MiB dla pełnego klonu i
11.67 MiB dla płytkiego. Niemniej jednak, to jest rozmiar danych, które trzeba pobrać przez sieć.
Te dane są skompresowane i `git` po pobraniu nam je automatycznie wypakuje i troszkę ich objętość
wzrośnie: 252 MiB vs. 96 MiB. To w dalszym ciągu nie jest pełny rozmiar źródeł OpenWRT, bo w skład
tego projektu wchodzi cała masa innych pakietów, które będą dociągane w późniejszym czasie. W taki
oto sposób całe źródła OpenWRT mogą ważyć nawet kilkanaście-kilkadziesiąt GiB, w zależności od tego
co będziemy budować. Poniżej przykład:

![](/img/2020/04/000-kompilacja-firmware-openwrt-rozmiar-dysk.png#big)

Dlatego też dobrze jest sobie zawczasu wygospodarować wolne miejsce, by uniknąć problemów później.

Jeśli chcemy pobrać pełne źródła OpenWRT, to korzystamy z poniższego polecenia:

    $ git clone https://git.openwrt.org/openwrt/openwrt.git
    Cloning into 'openwrt'...
    remote: Enumerating objects: 508113, done.
    remote: Counting objects: 100% (508113/508113), done.
    remote: Compressing objects: 100% (137368/137368), done.
    remote: Total 508113 (delta 353835), reused 502462 (delta 349504)
    Receiving objects: 100% (508113/508113), 153.67 MiB | 295.00 KiB/s, done.
    Resolving deltas: 100% (353835/353835), done.
    Updating files: 100% (8924/8924), done.

Jeśli zaś interesuje nas jedynie płytki klon repozytorium konkretnego wydania, np v19.07, to
wpisujemy w terminal poniższą linijkę:

    $ git clone --depth=1 https://git.openwrt.org/openwrt/openwrt.git -b openwrt-19.07 openwrt-v19.07

    Cloning into 'openwrt-v19.07'...
    remote: Enumerating objects: 9975, done.
    remote: Counting objects: 100% (9975/9975), done.
    remote: Compressing objects: 100% (8633/8633), done.
    remote: Total 9975 (delta 2255), reused 4467 (delta 630)
    Receiving objects: 100% (9975/9975), 11.67 MiB | 763.00 KiB/s, done.
    Resolving deltas: 100% (2255/2255), done.

W tym drugim poleceniu w przełączniku `-b` trzeba określić tag/gałąź. [Tutaj znajduje się lista
tagów][5], a tutaj [lista gałęzi][21], które możemy sobie określić przy pobieraniu źródeł OpenWRT.

Jeśli nie zamierzamy przyczyniać się do rozwoju projektu OpenWRT, to naturalnie możemy skorzystać z
tego drugiego rozwiązania. Nic też nie stoi na przeszkodzie by pobrać źródła przy pomocy tego
pierwszego polecenia i przełączyć się do nowej gałęzi w poniższy sposób:

    $ cd openwrt/
    $ git checkout openwrt-19.07
    Updating files: 100% (7970/7970), done.
    Branch 'openwrt-19.07' set up to track remote branch 'openwrt-19.07' from 'origin'.
    Switched to a new branch 'openwrt-19.07'

## WARNING: Makefile 'package/.../Makefile' has a dependency on '...', which does not exist

Przy wydawaniu poleceń, które pojawią się w późniejszej części tego artykułu, można zaobserwować
pewne ostrzeżenia dotyczące problemów z zależnościami. By uniknąć błędów podobnych do tych poniżej,
dobrze jest skonfigurować sobie [OpenWRT Feeds][10]:

    WARNING: Makefile 'package/utils/busybox/Makefile' has a dependency on 'libpam', which does not exist
    WARNING: Makefile 'package/utils/busybox/Makefile' has a build dependency on 'libpam', which does not exist
    WARNING: Makefile 'package/network/utils/curl/Makefile' has a dependency on 'libgnutls', which does not exist
    WARNING: Makefile 'package/network/utils/curl/Makefile' has a dependency on 'libopenldap', which does not exist
    WARNING: Makefile 'package/network/utils/curl/Makefile' has a dependency on 'libidn2', which does not exist
    WARNING: Makefile 'package/network/utils/curl/Makefile' has a dependency on 'libssh2', which does not exist
    WARNING: Makefile 'package/boot/kexec-tools/Makefile' has a dependency on 'liblzma', which does not exist
    WARNING: Makefile 'package/network/services/lldpd/Makefile' has a dependency on 'libnetsnmp', which does not exist
    ...

Feeds'y figurują w zasadzie w pliku `feeds.conf` (ewentualnie też w `feeds.conf.default` , jeśli
`feeds.conf` nie istnieje). Zwykle nie ma potrzeby ruszać tego pliku ale po pobraniu źródeł OpenWRT
trzeba wydać te dwa poniższe polecenia, by te feeds'y pobrać:

    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a

Gdzieniegdzie można się spotkać także z alternatywnym poleceniem wykorzystującym `make` i target
`package/symlinks` , przykładowo:

    $ make package/symlinks

Niemniej jednak, to polecenie robi dokładnie to samo co te dwa wyżej, o czym można się przekonać
zaglądając do pliku `include/toplevel.mk` .

## Wstępna konfiguracja OpenWRT

Mając już źródła na dysku, możemy przejść do ich konfiguracji. Do wyboru mamy konfigurację w trybie
tekstowym za pomocą polecenia `make menuconfig` , jak i też konfigurację w trybie graficznym za
sprawą polecenia `make xconfig` . W tym artykule zostanie opisany ten drugi sposób ale nic nie stoi
na przeszkodzie by korzystać z trybu tekstowego. Wpisujemy zatem w oknie terminala to poniższe
polecenie:

    $ make xconfig

Naszym oczom powinno ukazać się okienko z całą masą opcji, w którym musimy określić minimum trzy
rzeczy.

Pierwszą z nich jest `Target System` :

![](/img/2020/04/001-kompilacja-firmware-openwrt-config-target.png#huge)

Jeśli nie wiemy jaki target wybrać, to dobrze jest poszukać naszego routera WiFi na
[wiki OpenWRT][6] lub też na [wikidevi][7]. Tam powinien być podany target:

![](/img/2020/04/002-kompilacja-firmware-openwrt-config-target.png#huge)

W tym przypadku jest `ar71xx-ath79` , co oznacza, że akurat ten router występuje zarówno w targecie
`ar71xx` jak i `ath79` i można określić jeden albo drugi.

Drugą opcją, którą musimy określić, jest `Subtarget` :

![](/img/2020/04/003-kompilacja-firmware-openwrt-config-target.png#huge)

W zasadzie jeśli nasz router nie dysponujemy małym flash lub flash'em opartym o technologię NAND,
to wybieramy `TARGET_ath79_generic` .

Jeśli poprawnie wybraliśmy te dwie poprzednie opcje, to w trzeciej opcji powinien być widoczny
model naszego routera:

![](/img/2020/04/004-kompilacja-firmware-openwrt-config-target.png#huge)

I jak widać jest do wyboru `TARGET_ath79_generic_DEVICE_tplink_archer-c7-v2` .

Te trzy opcje są w stanie wybrać cały niezbędny zestaw pakietów, które nasz router potrzebuje do
poprawnego działania. Niemniej jednak, jak się przyjrzymy tej liście, to jest tam bardzo dużo opcji.
Jeśli potrzebujemy w późniejszym czasie pracy routera instalować jakieś aplikacje czy pakiety z
modułami kernela, to przydałoby się te rzeczy uwzględnić zaznaczając stosowne pozycje.

Każda z opcji na liście może zostać włączona lub wyłączona. Niektóre pozycje zależą od innych i nie
da się ich włączyć lub wyłączyć bez uprzedniego rozwiązania problemów z zależnościami. Weźmy dla
przykładu ten poniższy `PACKAGE_luci` :

![](/img/2020/04/005-kompilacja-firmware-openwrt-config-zaleznosci.png#huge)

Nie możemy go ani włączyć ani wyłączyć, zatem jest on włączony czy wyłączony? Pozycja `Symbol:
PACKAGE_luci [=y]` wskazuje, że jest włączony. Co zatem aktywuje tę pozycję? Odpowiedź jest w
ostatniej linijce `Selected by: PACKAGE_luci-ssl [=n] || PACKAGE_luci-ssl-openssl [=y] ||
PACKAGE_luci-app-noddos [=n]` . Czyli jesli jedna z tych trzech opcji będzie włączona ( `||`
oznacza lub), to `PACKAGE_luci` zostanie automatycznie włączony. I jak możemy wywnioskować z
powyższego obrazka, opcja `PACKAGE_luci-ssl-openssl` jest włączona. Jeśli chcielibyśmy pozbyć się
interfejsu LuCI całkowicie z firmware routera, to trzeba by pierw odznaczyć
`PACKAGE_luci-ssl-openssl` , a dopiero później `PACKAGE_luci` .

Zgodnie z tym co można [wyczytać tutaj][8], to nie powinno się dołączać do obrazu pakietów, które
często podlegają aktualizacji. Chodzi o to, że jeśli taki pakiet dołączymy do obrazu, to późniejsza
jego aktualizacja wgra na flash routera ([za sprawą mechanizmu overlayfs][9]) drugą, nowszą wersję
aplikacji. W efekcie na flash'u będziemy mieć dwie wersje z tym, że będzie wykorzystywana ta nowsza,
a ta starsza wersja będzie jedynie marnować miejsce. W przypadku gdybyśmy ręcznie zainstalowali ten
pakiet po wgraniu firmware OpenWRT, to późniejsza jego aktualizacja bez problemu by zastąpiła pliki.
Oczywiście jeśli rzadko kiedy aktualizujemy oprogramowanie routera, tj. raz na parę tygodni czy
miesięcy wgrywamy jedynie nowy obraz, to naturalnie możemy sobie wszystkie niezbędne nam pakiety
wbudować bezpośrednio w obraz.

## Pobieranie źródeł projektów zależnych

Mając wstępnie skonfigurowane źródła OpenWRT, możemy przejść do ich kompilacji. Niemniej jednak,
zanim przejdziemy do samego procesu kompilacji źródeł, to dobrze jest pobrać źródła wszystkich
projektów, które wybraliśmy wcześniej. Chodzi generalnie o to, że te źródła będą dociągane podczas
procesu kompilacji. Jeśli chcemy przeprowadzić kompilację w trybie offline, to możemy wszystkie
niezbędne źródła pobrać przed procesem kompilacji. W tym celu wpisujemy w terminal polecenie
`make download` :

![](/img/2020/04/006-kompilacja-firmware-openwrt-make-download.png#huge)

## Kompilacja źródeł OpenWRT

Pozostało nam już w zasadzie uruchomienie procesu kompilacji przy pomocy `make` . Zgodnie z tym co
[jest napisane tutaj][11], to powinno się uważać przy korzystaniu z przełącznika `-j` , który
określa ilość jednoczesnych wątków kompilacji mających na celu przyśpieszenie całego procesu, ze
względu na fakt, że może to prowadzić do różnych błędów. Część z tych błędów można wyeliminować
uprzednio wydając polecenie `make download` , tak by wszystkie niezbędne źródła były dostępne
lokalnie przed rozpoczęciem procesu kompilacji. Ja budowałem źródła z `-j4` i nic strasznego się
nie stało z tego powodu. Niemniej jednak, jeśli już napotkamy błędy podczas procesu budowania
obrazu z firmware, to dobrze jest określić jeden wątek `-j1` oraz włączyć tryb rozmowny za sprawą
`V=s` ([stdout+stderr][13]). W taki sposób zostanie zwrócona masa informacji, które mogą nam pomóc
w ustaleniu przyczyny problemu. Dobrze jest także [zajrzeć tutaj][14]. Odpalamy zatem już
kompilację:

![](/img/2020/04/007-kompilacja-firmware-openwrt-make.png#huge)

Proces kompilacji potrwa dłuższą chwilę (parę godzin). Jeśli z jakiegoś powodu nie będziemy mogli
go ukończyć, np. nie będziemy mieć czasu, to zawsze ten proces można przerwać i później kontynuować
od miejsca, w którym skończyliśmy bez potrzeby rekompilacji wszystkiego od początku.

Po udanym procesie kompilacji, pliki z obrazem firmware OpenWRT będą na nas czekać w katalogu
`bin/targets/ath79/generic/` . Będzie tam kilka plików mających sufiks `.bin` :

    $  ls -alh bin/targets/ath79/generic | grep bin
    -rw-r--r-- 1 morfik morfik 9.7M 2020-04-04 20:50:50 openwrt-ath79-generic-tplink_archer-c7-v2-initramfs-kernel.bin
    -rw-r--r-- 1 morfik morfik  16M 2020-04-04 20:50:36 openwrt-ath79-generic-tplink_archer-c7-v2-squashfs-factory-eu.bin
    -rw-r--r-- 1 morfik morfik  16M 2020-04-04 20:50:35 openwrt-ath79-generic-tplink_archer-c7-v2-squashfs-factory-us.bin
    -rw-r--r-- 1 morfik morfik  16M 2020-04-04 20:50:35 openwrt-ath79-generic-tplink_archer-c7-v2-squashfs-factory.bin
    -rw-r--r-- 1 morfik morfik  11M 2020-04-04 20:50:36 openwrt-ath79-generic-tplink_archer-c7-v2-squashfs-sysupgrade.bin

Zgodnie z tym co można [wyczytać tutaj][12], to obraz mający w nazwie `factory` służy do przejścia
z oryginalnego oprogramowania producenta routera, na firmware OpenWRT. Tam wyżej mamy dodatkowo
obrazy przeznaczone do aktualizacji routerów sprzedawanych na terenie Stanów Zjednoczonych
( `-us` ) i Unii Europejskiej ( `-eu` ). Jeśli zaś chodzi o obraz mający w nazwie `initramfs` , to
niektóre urządzenia wymagają takich obrazów, by w ogóle móc na nich zainstalować OpenWRT. W
przypadku routera Archer C7 v2, ten obraz `initramfs` jest zupełnie zbędny i można wyłączyć jego
budowę odznaczając opcję `TARGET_ROOTFS_INITRAMFS` w konfiguracji. Obraz mający w nazwie
`sysupgrade` służy do aktualizacji w obrębie firmware OpenWRT (za sprawą `sysupgrade` lub panelu
webowego LuCI czy Gargoyle) i to ten obraz nas najbardziej będzie interesować.

### Wgrywanie obrazu na router

Mając gotowy obraz, musimy go jeszcze przesłać na router. W tym przypadku akurat wykorzystywany
jest panel LuCI ale nic nie stoi na przeszkodzie by skorzystać z `sysupgrade` .

![](/img/2020/04/008-kompilacja-firmware-openwrt-flash-image-luci.png#huge)

Możemy naturalnie sprawdzić sumę kontrolną via `sha256sum` ale nie jest to konieczne z racji, że
obraz był budowany lokalnie.

![](/img/2020/04/010-kompilacja-firmware-openwrt-flash-image-luci.png#huge)

Proces powinien zakończyć się powodzeniem, a router zrestartować.

## Importowanie starej konfiguracji

Jeśli w przeszłości przygotowywaliśmy sobie obraz z firmware OpenWRT dla naszego routera WiFi, to
zapewne mamy na dysku plik `.config` , który zawiera konfigurację takiego obrazu. By zaoszczędzić
sobie trochę czasu, możemy ten plik zaimportować do nowych źródeł. Problem w tym, że z czasem opcje
konfiguracji ulegają zmianie i ten nasz stary plik `.config` może zawierać nieistniejące pozycje,
lub też nie posiadać tych, które zostały utworzone przy wprowadzaniu zmian w OpenWRT. Trzeba zatem
taki plik konfiguracyjny zaktualizować i mamy do wyboru dwie opcje: `make defconfig` oraz
`make oldconfig` .

W przypadku korzystania z `make defconfig` , ze starego pliku `.config` zostaną podebrane tylko te
istniejące opcje, zaś wartości nowych opcji zostaną ustawione na domyślne. Jeśli chodzi o
`make oldconfig` , to sprawa wygląda podobnie ale w przypadku nowych opcji będziemy ręcznie musieli
określić ich wartości. Zatem jeśli importujemy konfigurację, to mamy do wyboru:

    $ cp /old/openwrt/.config ./.config
    $ make defconfig

lub:

    $ cp /old/openwrt/.config ./.config
    $ make oldconfig

## Dostosowanie konfiguracji kernela

W przypadku gdybyśmy chcieli nieco inaczej skonfigurować sam kernel, to standardowe
`make menuconfig` czy `make xconfig` nam tego nie umożliwi. Mamy jednak do dyspozycji jeszcze
target `make kernel_menuconfig` (brak odpowiednika z xconfig) i przy pomocy tego targetu jesteśmy
w stanie zmienić opcje kernela tak jak uważamy za stosowne.

![](/img/2020/04/014-kompilacja-firmware-openwrt-config-kernel.png#huge)

Po dokonaniu zmian w konfiguracji kernela, trzeba jeszcze raz zbudować cały obraz z firmware
OpenWRT.

## Czyszczenie środowiska kompilacji

Od czasu do czasu będziemy musieli przeczyścić środowisko kompilacji. Zwykle będziemy to robić przy
napotkaniu różnych błędów lub też profilaktycznie raz na jakiś czas. Warto zatem wiedzieć, że
istnieją targety dla `make` , które w czyszczeniu źródeł mogą nam pomóc.

### make clean

Przede wszystkim jest `make clean` , który w zasadzie czyści zawartość katalogów `bin/` oraz
`build_dir/` . Czyści on także tylko tę architekturę/targety, które są określone w pliku `.config` .
Dobrą praktyką jest wydawanie `make clean` za każdym razem przed kompilacją, by mieć pewność, że
nie ma żadnych pozostałości po poprzedniej kompilacji.

### make dirclean

Dalej mamy `make dirclean` , który czyści zawartość katalogów `bin/` , `build_dir/` ,
`staging_dir/` , `toolchain/` , `tmp/` oraz `logs/` (jeśli istnieje) .

### make distclean

Jest jeszcze `make distclean` , który to czyści wszystko co zostało skompilowane lub skonfigurowane
(w tym plik `.config` ). Ten target także usuwa wszystkie pobrane feeds'y i źródła pakietów.
Korzystanie z `make distclean` powinno mieć w zasadzie miejsce jedynie wtedy gdy zamierzamy
całkowicie zresetować środowisko kompilacji przywracając je do stanu chwilę po pobraniu źródeł
OpenWRT.

### make target/linux/clean

Te trzy powyższe targety czyszczą dość sporo rzeczy. Jeśli istnieje potrzeba by wyczyścić tylko
obiekty kernela, to możemy skorzystać z `make target/linux/clean` .

### make package/*/clean

Podobnie sprawa wygląda w przypadku czyszczenia pojedynczych pakietów. Wystarczy odszukać
odpowiednią ścieżkę ze źródłami pakietu (w katalogu `package/`). Dla przykładu, jeśli chcemy
wyczyścić kompilację interfejsu LuCI, to musimy określić `make package/feeds/luci/luci/clean` .

## Pakiety spoza repozytorium OpenWRT

W OpenWRT jest skonfigurowanych kilka dodatkowych źródeł pakietów zwanych feeds. Te standardowe
źródła są uwzględnione w pliku `feeds.conf.default` . Jeśli będziemy chcieli wbudować w obraz
pakiety, których nie ma w tych domyślnych feeds'ach, ale są przy tym w jakichś innych, to te
dodatkowe feeds'y trzeba będzie dodać do pliku `feeds.conf` .

Dla przykładu weźmy sobie dwa użyteczne narzędzia z `eko.one.pl` . Pierwszym z nich jest
`sysinfo.sh` zwracającym dodatkowe informacje przy logowaniu się na router przez SSH. Drugim zaś
jest `stat.sh` , który [zgłasza statystyki routera][16]. Te skrypty siedzą odpowiednio w pakietach
`ekooneplstat` , `luci-app-ekooneplstat` oraz `sysinfo` z tym, że są one dostępne w [repozytorium
git obsy/packages][17]. By być w stanie zbudować i dołączyć do wynikowego obrazu te pakiety, to
musimy pierw dodać to powyższe repozytorium git do pliku `feeds.conf` .

Kopiujemy zatem domyślny plik z feeds'ami i zmieniamy mu nazwę na `feeds.conf` :

    $ cp feeds.conf.default feeds.conf

Na jego końcu dopisujemy jeszcze tę  poniższą linijkę:

    src-git obsy_packages https://github.com/obsy/packages.git

Następnie aktualizujemy feeds'y:

    $ make package/symlinks

Mając już zaktualizowane feeds'y, powinniśmy być w stanie dostrzec każdy z tych pakietów na liście
w `make menuconfig` lub `make xconfig` , przykładowo:

![](/img/2020/04/011-kompilacja-firmware-openwrt-add-feeds.png#huge)

Wystarczy teraz je już tylko zaznaczyć tak jak to zwykle czynimy w przypadku każdej innej pozycji
na liście i przebudować obraz z firmware.

## Budowanie obrazów dla kilku routerów

O ile budowanie obrazu OpenWRT na pojedynczy router WiFi nie powinno stanowić problemów, to w
przypadku wielu urządzeń możemy napotkać problem natury konfiguracyjnej. Chodzi o to, że np. plik
`.config` trzeba przepisywać ilekroć tylko zmieniany jest target. Takie ciągłe przepisywanie
targetu i wszystkich zależnych od niego opcji mija się trochę z celem, podobnie też pozbawione jest
sensu kopiowanie konfiguracji i odtwarzanie jej ilekroć tylko będziemy chcieli zbudować obraz z
firmware na inny router. Dlatego też deweloperzy OpenWRT postanowili rozwiązać ten problem poprzez
zaimplementowanie [osobnych środowisk kompilacji][15] (Build Environments), w których to zarówno
plik `.config` jak i katalog `files/` (oraz pliki w nim) są rozdzielone dla każdego z tych
środowisk. Technicznie rzecz biorąc, to w dalszym ciągu mamy jeden plik `.config` i jeden katalog
`files/` ale z racji zaprzęgnięcia git'a i jego gałęzi możemy stworzyć całą masę targetów
przełączając się między tymi gałęziami git'a. Cały ten mechanizm jest zarządzany przez skrypt
`scripts/env` , a sterowanie nim odbywa się w poniższy sposób.

Na samym początku przy pomocy `./scripts/env new` tworzymy osobne środowiska kompilacji dla każdego
z routerów, na które chcemy zbudować firmware OpenWRT. W tym przypadku będą dwa -- Archer C7 v2 i
Archer C2600:

    $ ./scripts/env new archer_c7_v2
    $ ./scripts/env new archer_c2600

Mając więcej niż jedno środowisko kompilacji trzeba się miedzy nimi jakoś przełączać. Do tego celu
służy `./scripts/env switch`

    $ ./scripts/env switch archer_c7_v2
    Switched to branch 'archer_c7_v2'

    $ ./scripts/env switch archer_c2600
    Switched to branch 'archer_c2600'

Będąc już w konkretnym środowisku kompilacji, konfigurujemy target za pomocą `make menuconfig` lub
`make xconfig` . Wszystkie zmiany, które poczynimy w konfiguracji możemy podejrzeć przez
`./scripts/env diff` :

![](/img/2020/04/012-kompilacja-firmware-openwrt-diff.png#huge)

I jeśli uważamy, że wszystko jest w porządku, to zapisujemy konfigurację obrazu za sprawą
poniższego polecenia:

    $ ./scripts/env save

By teraz zbudować obraz dla któregoś z tych dwóch routerów, to trzeba najpierw wybrać stosowne
środowisko kompilacji. Po tym jak już je wybierzemy, wystarczy wydać polecenie `make` .

Obraz zostanie zbudowany jedynie dla tego routera, w którego środowisku kompilacji się znajdujemy.
By zbudować obraz dla pozostałych routerów, trzeba się przełączyć do ich środowisk kompilacji i
również wydać polecenie `make` . Póki co nie ma żadnego automatu, który by budował wiele obrazów
OOTB i trzeba się ratować własnym skryptem.

Warto tutaj wspomnieć, że to powyższe rozwiązanie dotyczy jedynie sytuacji, w której kilka różnych
routerów ma inny target. Jeśli zarówno `Target System` oraz `Subtarget` są takie same dla
posiadanych przez nas urządzeń, to możemy budować obrazy dla tych routerów w jednym podejściu.
Wystarczy wybrać `Multiple devices` (opcja `TARGET_MULTI_PROFILE` ) na liście `Target Profile` . W
taki sposób zostanie mam udostępniona lista wielokrotnego wyboru, w której to będziemy mogli
zaznaczyć interesujące nas modele routerów:

![](/img/2020/04/016-kompilacja-firmware-openwrt-image-config-multiple-routers.png#huge)

## Aktualizacja źródeł OpenWRT

W późniejszym czasie zapewne zajdzie potrzeba zaktualizowania firmware routera do nowszej wersji.
Gdy ten moment nastąpi, to trzeba będzie zaktualizować źródła OpenWRT wraz ze źródłami wszystkich
projektów, które zamierzamy budować. Ten proces na szczęście nie jest jakoś skomplikowany, bo
sprowadza się do wydania kilku prostych poleceń.

Najpierw trzeba pobrać informacje ze zdalnego repozytorium git, by ustalić czy pojawiły się nowe
commit'y:

    $ git remote update && git status
    ...
    Your branch is behind 'origin/master' by 105 commits, and can be fast-forwarded.
      (use "git pull" to update your local branch)

Jak widać, od czasu ostatniej aktualizacji pojawiło się 105 nowych commit'ów. Jeśli interesuje nas
jedynie co się zmieniło, to możemy na szybko przejrzeć komentarze w commit'ach:

    $ git log -105 --pretty=oneline

Jeśli zaś chcemy zsynchronizować nasze lokalne repozytorium z tym zdalnym i pobrać te 105
brakujących commit'ów, to wydajemy to poniższe polecenie:

    $ git pull

Dodatkowo trzeba także zaktualizować feeds'y (przy pomocy `make package/symlinks` ), bo lista
pakietów w repozytoriach ulega zmianie i jedne pakiety wypadają, a inne są z kolei dodawane.
Poniżej przykład:

    $ make package/symlinks
    Updating feed 'packages' from 'https://git.openwrt.org/feed/packages.git^99efce0cd27adfcc53384fba93f37e5ee2e517de' ...
    Create index file './feeds/packages.index'
    Updating feed 'luci' from 'https://git.openwrt.org/project/luci.git^13dd17fca148965d38f0d4e578b19679a7c4daa2' ...
    Create index file './feeds/luci.index'
    Updating feed 'routing' from 'https://git.openwrt.org/feed/routing.git^efa6e5445adda9c6545f551808829ec927cbade8' ...
    Create index file './feeds/routing.index'
    Updating feed 'telephony' from 'https://git.openwrt.org/feed/telephony.git^6f95d6ab3f359ee2ce81c20522700937424d1591' ...
    Create index file './feeds/telephony.index'
    Updating feed 'obsy_packages' from 'https://github.com/obsy/packages.git' ...
    remote: Enumerating objects: 17, done.
    remote: Counting objects: 100% (17/17), done.
    remote: Compressing objects: 100% (7/7), done.
    remote: Total 16 (delta 1), reused 16 (delta 1), pack-reused 0
    Unpacking objects: 100% (16/16), 2.65 KiB | 159.00 KiB/s, done.
    From https://github.com/obsy/packages
       5f5a4b5..e5a3d14  master     -> origin/master
    Updating 5f5a4b5..e5a3d14
    Fast-forward
     luci-app-ekooneplstat/Makefile                                           | 44 ++++++++++++++++++++++++++++++++++++++++++++
     luci-app-ekooneplstat/files/usr/lib/lua/luci/controller/ekooneplstat.lua | 13 +++++++++++++
     luci-app-ekooneplstat/files/usr/lib/lua/luci/i18n/ekooneplstat.po        | 33 +++++++++++++++++++++++++++++++++
     luci-app-ekooneplstat/files/usr/lib/lua/luci/model/cbi/ekooneplstat.lua  | 43 +++++++++++++++++++++++++++++++++++++++++++
     4 files changed, 133 insertions(+)
     create mode 100644 luci-app-ekooneplstat/Makefile
     create mode 100644 luci-app-ekooneplstat/files/usr/lib/lua/luci/controller/ekooneplstat.lua
     create mode 100644 luci-app-ekooneplstat/files/usr/lib/lua/luci/i18n/ekooneplstat.po
     create mode 100644 luci-app-ekooneplstat/files/usr/lib/lua/luci/model/cbi/ekooneplstat.lua
    Create index file './feeds/obsy_packages.index'
    Collecting package info: done
    /bin/sh: 1: 8: Bad file descriptor
    Collecting package info: done
    Installing all packages from feed packages.
    Installing all packages from feed luci.
    Installing all packages from feed routing.
    Installing all packages from feed telephony.
    Installing all packages from feed obsy_packages.
    Installing package 'luci-app-ekooneplstat' from obsy_packages

Jak widać wyżej, do repo `obsy_packages` trafiła nowa paczka `luci-app-ekooneplstat` i bez
aktualizacji tego feeds'a nie bylibyśmy w stanie jej dostrzec, zbudować i dołączyć do wynikowego
obrazu.

Po zaktualizowaniu źródeł, trzeba także zaktualizować konfigurację w pliku `.config` przy pomocy
`make defconfig` lub `make oldconfig` . W tym przypadku został użyty ten drugi z racji, że ja
zawsze wolę sprawdzić co uległo zmianie w konfiguracji:

![](/img/2020/04/013-kompilacja-firmware-openwrt-config-update.png#huge)

Jak widać wyżej, mamy informację o nowym pakiecie i musimy zdecydować czy chcemy go zbudować czy
też nie. Domyślna akcja jest oznaczona dużą literą, w tym przypadku `N` . Gdybyśmy skorzystali z
`make defconfig` , to tego powyższego zapytania by nie było i `N` by zostało ustawione domyślnie.

W przypadku posiadania wielu środowisk kompilacji (utworzonych za sprawą `./scripts/env new` ),
aktualizację pliku `.config` trzeba niestety przeprowadzać osobno dla każdego środowiska oddzielnie.

## Edycja plików w obrazie

Wynikowy obraz nadaje się do wgrania na router praktycznie chwilę po zbudowaniu. Niemniej jednak,
taki obraz, który ma dołączone nasze pakiety, trzeba będzie po wgraniu na router skonfigurować.
Możemy sobie nieco ułatwić to zadanie prekonfigurując obraz. Dla przykładu, ja korzystam z
internetu LTE i muszę utworzyć i skonfigurować to połączenie. Idąc dalej, muszę także skonfigurować
domową sieć WiFi, której parametry zostają w zasadzie niezmienne. Podobnie sprawa wygląda z
kluczami SSH, które ciągle trzeba kopiować na router chwilę po wgraniu firmware. Te wszystkie
rzeczy (i też całą masę innych) można bez problemu ogarnąć przed zbudowaniem obrazu, tak by nie
trzeba było tych czynności przeprowadzać po wgraniu obrazu na router.

W przypadku chęci zmiany konfiguracji obrazu lepiej jest [zaprzęgnąć UCI (Unified Configuration
Interface)][20] zamiast wgrywać własne pliki. Chodzi generalnie o fakt, że te pliki mogą ulec
zmianie wraz z rozwojem OpenWRT, co może być przyczyną różnych błędów w działaniu routera. W
zasadzie to będzie nam potrzebny jeden skrypt, w którym te wszystkie polecenia zostaną zebrane. Z
początku operowanie na UCI może być trudne ale zawsze można podejrzeć aktualną konfigurację
któregoś podsystemu przez `uci show` (np. `uci show dhcp` ). By dodać wpisy trzeba korzystać z
`uci add` , by je zmienić `uci set` , a by skasować `uci del` . Można także w panelu webowym LuCI
routera zapisać konfigurację bez wprowadzania zmian i podejrzeć jakie polecenia UCI system ma
zamiar wydać (w prawym górnym rogu interfejsu).


Stwórzmy sobie zatem strukturę plików i katalogów, która będzie nam potrzebna:

    $ mkdir -p files/etc/uci-defaults/
    $ touch files/etc/uci-defaults/01-config.sh
    $ chmod +x files/etc/uci-defaults/01-config.sh
    $ echo "#!/bin/sh" > files/etc/uci-defaults/01-config.sh

Do pliku `01-config.sh` dodajemy teraz polecenia UCI. Poniżej jest kilka przykładów.

Konfiguracja interfejsu LTE:

    uci set network.lte=interface
    uci set network.lte.proto='ncm'
    uci set network.lte.device='/dev/cdc-wdm0'
    uci set network.lte.service='preferlte'
    uci set network.lte.pdptype='IP'
    uci set network.lte.apn='internet'
    uci set network.lte.pincode='internet'
    uci set network.lte.ipv6='auto'

    uci del firewall.@zone[1].network
    uci set firewall.@zone[1].network='wan wan6 lte'

Konfiguracja WiFi

    uci set wireless.default_radio0.ssid='Ever_Vigilant_5G'
    uci set wireless.default_radio0.encryption='psk2+ccmp'
    uci set wireless.default_radio0.key='pass'
    uci set wireless.default_radio0.wpa_disable_eapol_key_retries='1'
    uci set wireless.default_radio0.ifname='wifi5g'
    uci del wireless.default_radio0.disabled
    uci set wireless.radio0.country='PL'
    uci del wireless.radio0.disabled

    uci set wireless.default_radio1.ssid='Ever_Vigilant'
    uci set wireless.default_radio1.encryption='psk2+ccmp'
    uci set wireless.default_radio1.key='pass'
    uci set wireless.default_radio1.wpa_disable_eapol_key_retries='1'
    uci set wireless.default_radio1.ifname='wifi2g'
    uci del wireless.default_radio1.disabled
    uci set wireless.radio1.country='PL'
    uci del wireless.radio1.disabled

Taki zestaw poleceń jest w stanie skonfigurować nasz router w taki sposób, że my nie będziemy
musieli w zasadzie w ogóle ruszać routera po wgraniu firmware. Na końcu skryptu `01-config.sh`
dodajemy jeszcze poniższe polecenia:

    uci commit
    reload_config

Oczywiście nie każdy aspekt konfiguracyjny da radę ogarnąć przez zastosowanie poleceń UCI. W takich
sytuacjach będziemy potrzebować fizycznych plików, które zostaną podmienione lub dodane (jeśli nie
istnieją) w wynikowym obrazie. Przykładem mogą być wspomniane wcześniej klucze SSH, które rezydują
w pliku `/etc/dropbear/authorized_keys` na routerze. Możemy ten plik skopiować z routera i umieścić
w katalogu `files/` , przez co ten plik zostanie dodany do obrazu, a my nie będziemy musieli dodać
kluczy SSH do routera.

    $ mkdir -p files/etc/dropbear/
    $ cp /router/authorized_keys files/etc/dropbear/authorized_keys

### Podejrzenie wynikowego obrazu

Jeśli nie mamy pewności czy nasze pliki zostały dodane w odpowiednie miejsce w wynikowym obrazie,
to zawsze możemy ten fakt zweryfikować przez podejrzenie zawartości obrazu z firmware. Podejrzeć
pliki w takim obrazie możemy wypakowując gotowy obraz przy pomocy [binwalk][18], choć w przypadku
tego narzędzia potrzebne są dodatkowe zależności, których w Debianie brakuje (np. [sasquatch][19],
czyli łatki na `squashfs-tools` ). Alternatywnym podejściem jest sprawdzenie co do obrazu firmware
trafi przed jego zbudowaniem. Wszystkie pliki, które zostaną załączone w wynikowym obrazie znajdują
się  w katalogu `build_dir/target-mips_24kc_musl/root-ath79/` . Warto tutaj zaznaczyć, że chodzi o
katalog `root-ath79/` , a nie o `root.orig-ath79/` . Ten `root.orig-ath79/` zawiera jedynie
podstawowe pliki OpenWRT, zaś w `root-ath79/` znajdują się pliki wymagane przez konkretny model
routera, w tym też pliki z katalogu `files/` :

![](/img/2020/04/015-kompilacja-firmware-openwrt-image-files-root-orig.png#huge)

Jeśli pliki znajdują się w `root-ath79/` w odpowiednim miejscu w strukturze drzewa katalogów, to
możemy być pewni, że w obrazie wynikowym również będą w tym konkretnym miejscu.

## Podsumowanie

Jak widać budowanie obrazu z firmware OpenWRT na swój domowy router WiFi nie jest jakoś
skomplikowane, choć pewne umiejętności trzeba posiadać by płynnie przez ten proces przebrnąć.
Niewątpliwym plusem tego rozwiązania jest fakt, że przyzwoicie skonfigurowany obraz jest nam w
stanie zaoszczędzić sporo pracy przy aktualizacji firmware routera, bo nie trzeba w kółko
przeprowadzać tych samych czynności.


[1]: https://dev.archive.openwrt.org/ticket/8418.html
[2]: http://dl.eko.one.pl/openwrt-19.07/targets/ath79/generic/
[3]: https://openwrt.org/docs/guide-developer/quickstart-build-images
[4]: https://git.openwrt.org/openwrt/openwrt.git
[5]: https://git.openwrt.org/?p=openwrt/openwrt.git;a=tags
[6]: https://openwrt.org/toh/hwdata/tp-link/tp-link_archer_c7_ac1750_v2.0
[7]: https://wikidevi.wi-cat.ru/TP-LINK_Archer_C7_v2.x
[8]: https://eko.one.pl/?p=openwrt-kompilacja
[9]: https://en.wikipedia.org/wiki/OverlayFS
[10]: https://openwrt.org/docs/guide-developer/feeds
[11]: https://openwrt.org/docs/guide-user/additional-software/beginners-build-guide
[12]: https://eko.one.pl/?p=openwrt-19.07
[13]: https://oldwiki.archive.openwrt.org/doc/techref/buildroot#warnings_errors_and_tracing
[14]: https://oldwiki.archive.openwrt.org/doc/howto/build#troubleshooting
[15]: https://openwrt.org/docs/guide-developer/env
[16]: http://dl.eko.one.pl/stat.html
[17]: https://github.com/obsy/packages
[18]: https://github.com/ReFirmLabs/binwalk
[19]: https://github.com/devttys0/sasquatch
[20]: https://openwrt.org/docs/guide-user/base-system/uci
[21]: https://git.openwrt.org/?p=openwrt/openwrt.git;a=heads
