---
author: Morfik
categories:
- Linux
date: "2018-12-27T22:14:00Z"
lastmod: 2020-06-09 21:20:00 +0200
published: true
status: publish
tags:
- debian
- kernel
title: Budowanie kernela linux dla konkretnej maszyny z Debianem
---

Każda maszyna działająca pod kontrolą dystrybucji linux ma na swoim pokładzie kernel, czyli jądro
operacyjne, które zarządza praktycznie każdym aspektem pracy takiego komputera. W dystrybucji
Debian, kernel jest dostarczany w pakietach mających nazwy zaczynające się od `linux-image-*`.
Te pakiety są budowane przez odpowiednie osoby z zespołu Debiana i udostępniane do łatwej
instalacji użytkownikowi końcowemu. Niemniej jednak, taki kernel ma za zadanie działać na jak
największej ilości komputerów, a przez to posiada całą masę modułów, które na naszej maszynie nigdy
nie będą wykorzystane. Ten fakt nie wpływa w jakimś ogromnym stopniu na pracę maszyny, ale gdy
później zachodzi potrzeba skonfigurowania kernela w nieco inny sposób, np. włączenie jednej czy
dwóch opcji czy też nałożenie patch'a, który nie został zaaplikowany przez dev'ów Debiana, to
trzeba taki kernel na nowo skompilować już samodzielnie, a to zajmie nam bardzo dużo czasu. Zamiast
tego można pokusić się o przygotowanie kernela pod konkretny hardware wyłączając przy tym całą masę
zbędnych rzeczy i ograniczając przy tym czas jaki jest potrzebny na zbudowanie całego jądra
operacyjnego. Czy istnieje jakiś prosty sposób, by taki kernel zbudować sobie samemu mając przy tym
minimalną wiedzę co do opcji kernela, które mogą nas przysporzyć o ból... głowy? Okazuje się, że
tak i w tym artykule prześledzimy sobie cały ten proces.

<!--more-->
## Źródła kernela oraz zależności

Przede wszystkim, by móc bawić się w kompilację kernela musimy sobie załatwić jego źródła, a te z
kolei możemy albo pozyskać z dystrybucji Debiana, albo też bezpośrednio z [kernel.org][1]. Bez
znaczenia, na które źródła się zdecydujemy -- one są prawie identyczne, zakładając, że numerki się
zgadzają. Prawie, a to dlatego, że Debianowy kernel zawiera szereg łat bezpieczeństwa, które
domyślnie będą zaaplikowane już. Jeśli chcemy goły kernel, to musimy pobrać go z oficjalnej strony
kernela. Jeśli wybierzemy źródła kernela Debiana, to w terminalu wydajemy poniższe polecenie:

    # aptitude install linux-source

Po pobraniu paczki, źródła zostaną zainstalowane w katalogu `/usr/src/linux-*/`, gdzie `*` oznacza
numerek kernela.

Musimy doinstalować także szereg zależności niezbędnych w procesie budowania źródeł kernela:

    # aptitude install \
     build-essential \
     bc \
     kmod \
     cpio \
     flex \
     libncurses5-dev \
     libssl-dev \
     libelf-dev \
     qt5-default \

## Wstępna konfiguracja systemu i zmienne środowiskowe

Okazuje się, że w przypadku budowania paczki z kernelem dla Debiana trzeba nieco inaczej podejść
do sprawy eksportu zmiennych środowiskowych, min. przy ustawianiu flag dla kompilatora i linkera.

Z reguły ludzie też starają się eksportować zmienną `$ARCH` w konfiguracji shell'a tak, by proces
kompilacji mógł sobie ją podebrać. Zmienna `$ARCH` precyzuje architekturę naszego systemu i będzie
ona widoczna w procesie kompilacji ale nie powinniśmy jej w ten sposób eksportować. Chodzi o to, że
niektóre narzędzia Debiana mogą zostać wprowadzone w błąd jeśli ta zmienna zostanie w konfiguracji
shell'a ustawiona. Poniżej jest przykład aktualizacji systemu, a konkretnie jednej z paczek DKMS:

    # cat /var/lib/dkms/xtables-addons/3.2/build/make.log
    DKMS make.log for xtables-addons-3.2 for kernel 4.19.13-amd64-morficzny (x86_64)
    Thu  3 Jan 10:57:49 CET 2019
    make: Entering directory '/usr/src/linux-headers-4.19.13-amd64-morficzny'
    Makefile:618: arch//Makefile: No such file or directory
    make: *** No rule to make target 'arch//Makefile'.  Stop.
    make: Leaving directory '/usr/src/linux-headers-4.19.13-amd64-morficzny'

I jak widać z powyższego błędu, proces kompilacji tego modułu się nie powiódł bo plik
`arch//Makefile` nie został odnaleziony. Żeby ten błąd fix'nąć, trzeba zmienną `$ARCH` usunąć ze
środowiska przez `unset` albo zwyczajnie jej nie eksportować.

Zamiast eksportować zmienną `$ARCH` globalnie, lepiej jest wrzucić ją do polecenia z `make` ,
przykładowo:

    make ARCH="x86_64" -j2 bindeb-pkg

Natomiast jeśli chodzi o flagi kompilatora i linkera, to ten temat jest nieco skompilowany.

### Skrypt scripts/package/mkdebian oraz plik debian/rules

Przede wszystkim, przy budowaniu targetu `bindeb-pkg` , tworzona jest bardzo podstawowa
konfiguracja dla Debianowych narzędzi w podkatalogu `debian/` . Chwilę po wywołaniu polecenia
`make bindeb-pkg` wołany jest skrypt `scripts/package/mkdebian` , który to ma za zadanie uzupełnić
szereg plików w oparciu o pewne zmienne (min. o tę ustawioną wcześniej zmienną `$ARCH` ). Te pliki
są przepisywane za każdym razem ilekroć wołany jest `make bindeb-pkg` i generalnie to tych plików
nie powinniśmy edytować ręcznie. Skryptu w zasadzie też nie ma po co ruszać.

### Zmienne $CFLAGS, $CPPFLAGS, $CXXFLAGS i $LDFLAGS

Technicznie rzecz biorąc, to proces budowania paczki z kernelem jest niemal taki sam jak w
przypadku [budowania każdej innej paczki .deb][2] dla dystrybucji Debiana. Jedyną różnicą jest fakt,
że kernel nie jest standardową [binarką ELF][3].

Gdy buduje się zwykłe paczki różnych aplikacji, to w pliku `debian/rules` ustawia się kilka
zmiennych m.in. zmienną `$DEB_BUILD_MAINT_OPTIONS` , która to konfiguruje wstępnie zmienne dla
kompilatora i linkera. W zasadzie wszystkie standardowe flagi używane przy kompilacji mogą zostać
skonfigurowane przez tę zmienną `$DEB_BUILD_MAINT_OPTIONS` , przykładowo:

    export DEB_BUILD_MAINT_OPTIONS = hardening=+all
    DPKG_EXPORT_BUILDFLAGS = 1
    include /usr/share/dpkg/buildflags.mk

Przynajmniej tak jest w przypadku zwykłych aplikacji. Jeśli chodzi zaś o proces kompilowania samego
kernela, to ustawianie zmiennej `$DEB_BUILD_MAINT_OPTIONS` nic nam nie da. Podobnie jak i ręczne
ustawianie zmiennych `$CFLAGS` , `$CPPFLAGS` , `$CXXFLAGS` oraz `$LDFLAGS` w konfiguracji shell'a,
bo nie będą brane w ogóle pod uwagę.

Jeśli już ktoś chciałby te zmienne ustawić, to musiałbym to zrobić przez plik `debian/rules` , tj.
za sprawą skryptu `scripts/package/mkdebian` dodając zaraz pod:

    ...
    cat <<EOF > debian/rules
    #!$(command -v $MAKE) -f
    ...

poniższe linijki kodu:

    CFLAGS   += -march=native -O2 -pipe -Wall -fstack-clash-protection -fpic
    CXXFLAGS += -march=native -O2 -pipe -Wall -fstack-clash-protection -fpic
    CPPFLAGS += -march=native -O2 -pipe -Wall -fstack-clash-protection -fpic
    LDFLAGS  += -Wl,-O2 -Wl,--as-needed -Wl,-z,defs -Wl,-shared

To jakie flagi faktycznie zostaną ustawione można podejrzeć przez dodanie do pliku `debian/rules`
(za sprawą skryptu) poniższego polecenia w targecie `build` :

    ...
    build:
	    dpkg-buildflags --status
    ...

Narzędzie `dpkg-buildflags` jest w stanie ustawić te poniższe flagi:

- `$CFLAGS` określa flagi dla kompilatora C. Domyślne flagi uwzględniają `-g` oraz `-O2` , chyba,
że `DEB_BUILD_MAINT_OPTIONS` ma zdefiniowane `noopt` i wtedy jest używany `-O0` .
- `$CXXFLAGS` określa flagi dla kompilatora C++. Takie same jak `$CFLAGS` .
- `$CPPFLAGS` określa flagi preprocesora C (`C` `P`re`P`rocessor). Domyślnie puste.
- `$FCFLAGS` określa flagi kompilatora Fortran 9x . Takie same jak `$CFLAGS`
- `$FFLAGS` określa flagi kompilatora Fortran 77. Podzbiór `$CFLAGS` .
- `$GCJFLAGS` określa flagi kompilatora GNU Java (gcj). Podzbiór `$CFLAGS` .
- `$LDFLAGS` określa flagi linkera. Domyślnie puste.
- `$OBJCFLAGS` określa flagi kompilatora obiektowego C. Takie same jak `$CFLAGS` .
- `$OBJCXXFLAGS` określa flagi kompilatora obiektowego C++. Takie same jak `$CXXFLAGS` .

Jeśli teraz odpalilibyśmy proces kompilacji, to `dpkg-buildflags --status` zwróci nam coś na wzór
poniższego kodu:

    ...
    dpkg-buildflags --status
    dpkg-buildflags: status: environment variable DEB_BUILD_OPTIONS=parallel=2
    dpkg-buildflags: status: environment variable DEB_HOST_ARCH=amd64
    dpkg-buildflags: status: vendor is Debian
    dpkg-buildflags: status: future features: lfs=no
    dpkg-buildflags: status: hardening features: bindnow=no format=yes fortify=yes pie=yes relro=yes stackprotector=yes stackprotectorstrong=yes
    dpkg-buildflags: status: qa features: bug=no canary=no
    dpkg-buildflags: status: reproducible features: fixdebugpath=yes fixfilepath=no timeless=yes
    dpkg-buildflags: status: sanitize features: address=no leak=no thread=no undefined=no
    dpkg-buildflags: status: CFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: CPPFLAGS [vendor]: -Wdate-time -D_FORTIFY_SOURCE=2
    dpkg-buildflags: status: CXXFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: FCFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong
    dpkg-buildflags: status: FFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong
    dpkg-buildflags: status: GCJFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong
    dpkg-buildflags: status: LDFLAGS [vendor]: -Wl,-z,relro
    dpkg-buildflags: status: OBJCFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: OBJCXXFLAGS [vendor]: -g -O2 -fdebug-prefix-map=/usr/src/linux-source-4.19=. -fstack-protector-strong -Wformat -Werror=format-security
    ...

Przez konfigurację obszarów cech (feature areas) można wpływać ma to jakie flagi zostaną ustawione.
To jakie flagi te obszary cech definiują zostało dokładnie opisane
w [manualu dpkg-buildflags][4].

#### Flagi kompilatora

Poniżej jest krótka rozpiska [flag kompilatora][5], które są wykorzystywane przy budowaniu kodu
źródłowego:

- flaga `-march=` określa dla jakiego procesora jest budowany kod. W przypadku, gdy kompilujemy
kernel (czy w ogóle jakiś kod) tylko i wyłącznie dla naszej maszyny, to nie musimy sobie zawracać
głowy właściwościami jej procesora. W takim przypadku można tutaj określić `native` i kompilator
już sobie obada ten procesor i odpowiednie flagi dobierze sam.
- flaga `-O` odpowiada za stopień optymalizacji kodu. Stopni jest kilka: `0` , `1` , `2` , `3` ,
`s` , `g` oraz `fast` . [Każdy poziom jest nacechowany innymi flagami][20] ale tym zalecanym jest
poziom `2`.
- flaga `-pipe` nie ma jako takiego przełożenia na generowany kod ale może uczynić proces
kompilacji szybszym przez wykorzystanie potoków (pipe) zamiast fizycznych plików tymczasowych.
Takie rozwiązanie wiąże się jednak z większym apetytem procesu kompilacji na pamięć operacyjną i
nie powinno się tej flagi stosować, gdy nasz komputer nie ma dostatecznie dużo pamięci, by sobie z
procesem kompilacji poradzić, bo kompilator może zostać ubity przez system.
- flaga `-g` tworzy informacje wykorzystywane przy debugowaniu w natywnym dla systemu operacyjnego
formacie (stabs, COFF, XCOFF lub DWARF).
- flaga `-Wall` sprawa, że wszystkie ostrzeżenia w procesie kompilacji będą widoczne.
- flagi `-Werror=*` odpowiadają za podniesienie pewnych grup ostrzeżeń do rangi błędu, co
efektywnie kończy proces kompilacji w przypadku, gdy takie ostrzeżenie zostanie odnotowane.
- flaga `-Wformat` sprawdza czy wywołania `printf` i `scanf`, itp. mają określony odpowiedni format.
- flaga `-fstack-protector-strong` ma za zadanie chronić stos przed jego celowym uszkodzeniem
([stack smashing/stack buffer overflow][6]).
- flaga `-fstack-clash-protection` ma za zadanie chronić stos przed atakami [stack clash][7].
- flaga `-D_FORTIFY_SOURCE=2` odpowiada za zastąpienie niebezpiecznych wywołań funkcji mających
nieograniczony rozmiar buforu tymi, które taki limit posiadają.
- flagi `-pie` i `-fPIE` sprawiają, że pozycyjnie niezależne pliki wykonywalne (Position Independent
Executable) mogą korzystać z ALSR ([Address Space Layout Randomization][8]).
- flaga `-fpic` generuje kod niezależny od położenia
([position independent code, PIC][9]) dla bibliotek współdzielonych.

#### Flagi linkera

Jeśli chodzi zaś o flagi linkera, to nie są one podawane bezpośrednio do niego ( `ld` ) ale do
kompilatora ( `gcc` ). Dlatego trzeba każdą z flag linkera poprzedzić flagą `-Wl` :

- flaga `-O` odpowiada za stopień optymalizacji kodu.
- flaga `--as-needed` nakazuje linkerowi, by linkował w zbudowanym pliku binarnym tylko te
biblioteki, które zawierają symbole używane przez ten konkretny plik binarny. Więcej info
[tutaj][10].
- flaga `-z,relro` sprawia, że w chwili ładowania programu pewne
sekcje [pamięci ELF][11], które muszą być zapisane przez linker, będą zmienione tylko do odczytu
przed przekazaniem kontroli temu programowi. Chroni to przed częścią [ataków GOT][12] (Global
Offset Table).
- flaga `-z,now` sprawia, że podczas ładowania programu, wszystkie dynamiczne symbole są
rozwiązywane pozwalając tym samym na oznaczenie GOT jako tylko do odczytu. Może to jednak nieść ze
sobą pewne problemy z wydajnością przy ładowaniu większych aplikacji.

Więcej o flagach można poczytać [tutaj][13] oraz [tutaj][14].

### Hardening kernela
W zasadzie to nie musimy ustawiać żadnych z tych opisanych wyżej flag samodzielnie. Wszystko co
powinno zostać ustawione, zostanie ustawione automatycznie i żadnych dodatkowych czynności nie
musimy przeprowadzać. Niemniej jednak, na necie można się spotkać z różnymi formami hardening'u
kernela przez ustawianie flag. Wychodzi jednak na to, że manipulacja flagami nie wpływa w żaden
sposób na to co zostanie zbudowane ze źródeł kernela, przynajmniej w przypadku wywoływania
`make bindeb-pkg` . Zwykle każda zmiana flag powinna skutkować przebudową części albo całego kodu
od początku w przypadku korzystania z `ccache` ale tak się nie dzieje. Po zmianie flag, Cache HIT
rośnie, a nie powinien. Dlatego też tę całą zabawę w hardening kernela za sprawą flag można sobie
darować.

Jeśli już chodzi o prawdziwy hardening kernela, to trzeba podejść do tego zadania od strony
konfiguracyjnej, a konkretnie chodzi o [ustawienie stosownych opcji][15] w samym kernelu.

Warto tutaj dodać, że `hardening-check` na wypakowanej binarce kernela zwróci nam błędny wynik
niezależnie od tego z jakimi flagami będziemy źródła kernela kompilować. Dzieje się tak dlatego, że
aplikacje typu `hardening-check` są przeznaczone do pracy z regularnymi binarkami ELF, a binarka
kernela się do takowych nie zalicza i ciężko jest oczekiwać od tych narzędzi, by zwracały poprawny
wynik.

#### GCC plugins
Jeśli będziemy się bawić w hardening kernela przez jego odpowiednią konfigurację, to przydałoby się
też włączyć opcje od `GCC plugins` . Standardowo są one jednak nieaktywne lub wyłączone i nie da
się ich przestawić bez wcześniejszej instalacji w systemie paczki `gcc-8-plugin-dev` :

    # aptitude install gcc-8-plugin-dev

Teraz te opcje powinny być już aktywne.

### Problemy z xconfig

Generalnie rzecz biorąc, kernel możemy
konfigurować [przez narzędzie  GUI][16] wydając w terminalu polecenie `make xconfig`. To narzędzie
wykorzystuje interfejs QT, a do jego skonfigurowania potrzebny nam będzie `qt5ct`. Musimy zatem
doinstalować sobie poniższy pakiet:

    # aptitude install qt5ct

W przypadku problemów przy odpalaniu `qt5ct`, trzeba będzie wyeksportować zmienną
`$QT_QPA_PLATFORMTHEME`:

    # export QT_QPA_PLATFORMTHEME="qt5c"

### ccache

By dodatkowo przyśpieszyć proces ponownej kompilacji kodu, możemy doinstalować w systemie `ccache`:

    # aptitude install ccache

Trzeba także porobić kilka dowiązań, by `ccache` był wykorzystywany podczas kompilacji:

    # ln -s /usr/bin/ccache /usr/local/bin/gcc
    # ln -s /usr/bin/ccache /usr/local/bin/g++
    # ln -s /usr/bin/ccache /usr/local/bin/cc
    # ln -s /usr/bin/ccache /usr/local/bin/c++

Musimy również wyeksportować kilka zmiennych:

    # export USE_CCACHE="1"
    # export CCACHE_DIR="/media/kernel/ccache"

Wielkością cache sterujemy w poniższy sposób:

    # ccache -M 10G
    Set cache size limit to 10.0 GB

    # ccache -s
    cache directory                     /media/kernel/ccache/
    primary config                      /media/kernel/ccache/ccache.conf
    ...
    max cache size                      10.0 GB

### Wersja gcc/g++/cpp

W Debianie dostępnych jest kilka wersji kompilatorów gcc/g++ i preprocesora C. Aktualnie domyślną
wersją jest wersja 9. Niemniej jednak, w systemie można zainstalować także wersję 10 i to raczej
nie powinno stanowić problemu. Natomiast problemem może być wybór, którą wersję tych narzędzi
chcemy wykorzystywać przy budowaniu kernela czy też innych projektów. W takich sytuacjach z pomocą
przychodzi system alternatyw, za pomocą którego to musimy określić preferowaną wersję gcc/g++/cpp ,
a linki do odpowiednich binarek zostaną już automatycznie utworzone za nas. Mając już zainstalowane
stosowne pakiety, odpalamy terminal i wpisujemy w nim te poniższe polecenia.

Usuwamy stare alternatywy dla gcc/g++/cpp:

    # update-alternatives --remove-all gcc
    # update-alternatives --remove-all g++
    # update-alternatives --remove-all cpp

Dodajemy teraz nowe alternatywy:

    # update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 10
    # update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 100

    # update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 10
    # update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 100

    # update-alternatives --install /usr/bin/cpp cpp-bin /usr/bin/cpp-10 10
    # update-alternatives --install /usr/bin/cpp cpp-bin /usr/bin/cpp-9 100

Ustawiamy również by linki cc/c++ wskazywały na gcc/g++ :

    # update-alternatives --install /usr/bin/cc cc /usr/bin/gcc 30
    # update-alternatives --set cc /usr/bin/gcc

    # update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++ 30
    # update-alternatives --set c++ /usr/bin/g++

    # update-alternatives --set cpp-bin /usr/bin/cpp-10

I na koniec konfigurujemy sobie preferowaną wersję gcc/g++/cpp:

    # update-alternatives --config gcc
    # update-alternatives --config g++
    # update-alternatives --config cpp

## Pozyskiwanie informacji o sprzęcie

To co jest niezbędne do zbudowania kernela, który będzie w stanie obsłużyć nasz sprzęt w pełni, to
informacje o modułach, których potrzebują poszczególne podzespoły. Każdy kawałek hardware działa w
oparciu o jakiś moduł. Listę aktualnie załadowanych w systemie modułów możemy ustalić wydając
polecenie `lsmod`. Trzeba tutaj jednak pamiętać, że jest to lista modułów dla urządzeń, które
aktualnie działają w systemie, ewentualnie też dla tych, które zostały od niego odłączone ale
system nie został zresetowany, a moduły nie zostały wyładowane ręcznie. Dlatego też przydałoby się
w tym momencie podłączyć wszystkie te urządzenia, z których zamierzamy korzystać w przyszłości, np.
pendrive, modemy LTE, karty WiFi, adaptery bluetooth, myszy, klawiatury USB, itp. System sobie
dobierze odpowiednie moduły i je nam załaduje. Lista tych modułów zostanie następnie podana podczas
wstępnej konfiguracji kernela.

## make localyesconfig

Gdy już mamy załadowane wszystkie potrzebne moduły, przechodzimy do katalogu ze źródłami i wydajemy
poniższe polecenie:

    # make localyesconfig

W katalogu roboczym powinien zostać wygenerowany plik `.config` zawierający konfigurację kernela.
Być może zostaniemy poproszeni o udzielenie odpowiedzi na szereg pytań dotyczących nowych opcji,
czyli tych, które nie występowały we wcześniejszej konfiguracji:

![](/img/2018/12/002.debian-kernel-localyesconfig.png#huge)

Można bez problemu odpowiedzieć negatywnie na wszystkie pytania wciskając `N`, chyba, że coś jest
nam potrzebne.

## make xconfig

Mając konfigurację w pliku `.config` możemy wpisać w terminal poniższe polecenie:

    # make xconfig

Powinno zostać nam wyświetlone okienko, w którym będziemy mogli już dowolnie włączać lub wyłączać
określone opcje kernela:

![](/img/2018/12/003.debian-kernel-xconfig.png#huge)

Mój laptop posiada procesor Intel ale szereg opcji od procesorów AMD było również pozaznaczanych.
Dlatego też warto przejrzeć te już zaznaczone opcje i je sobie dostosować.

## Budowanie kernela linux

Gdy już skończymy się bawić w konfiguracji, zapisujemy ją i wracamy na terminal, gdzie wydajemy
poniższe polecenie:

    # make ARCH="x86_64" -j2 bindeb-pkg

Target `bindeb-pkg` zbuduje nam trzy pliki: `linux-image-*.deb` , `linux-headers-*.deb` oraz
`linux-libc-dev_*.deb` . Te dwa pierwsze można zainstalować już w systemie, natomiast ten ostatni
zwykle będzie kolidował z wersją, która została zainstalowana w systemie z racji posiadania
dystrybucyjnego kernela. Ja tego trzeciego pakietu nie instalowałem póki co, a system zdaje się
działać jak należy.

Jeśli jest potrzeba zbudowania innych rzeczy, to dobrze jest zajrzeć do pomocy:

    # make help

## Funkcjonalność kernela wkompilowana na stałe czy jako moduł

Kernel linux'a posiada całą masę opcji przekładających się na jego funkcjonalność podczas normalnej
pracy komputera. Gdyby wszystkie te rzeczy wkompilować na stałe w kernel, to miałby on spore
rozmiary i zjadałby nam cenne zasoby pamięci operacyjnej. Dlatego też kernele różnych dystrybucji
linux'a mają całą masę modułów, które są lub mogą być ładowane w późniejszym czasie, gdy zachodzi
taka potrzeba, np. podłączamy nowe urządzenie USB do komputera.

Moduły są w porządku, gdy mamy do czynienia z kernelem, który ma działać na sporej masie sprzętów,
lub też gdy pewne urządzenia podłączamy jedynie sporadycznie. Niemniej jednak, cała ta
funkcjonalność kernela, która jest niezbędna naszej konkretnej maszynie do prawidłowego
funkcjonowania powinna być wkompilowana na stałe. To czy wkompilować na stałe obsługę modemu LTE,
z którego korzystamy w chwili, gdy nam padnie główne połączenie internetu, to już każdy powinien
sobie odpowiedzieć sam. Podobnie sprawa ma się z pozostałym sprzętem, którego nie używamy na co
dzień.

Jeśli chodzi o wydajność, to nie ma praktycznie żadnej różnicy w przypadku kompilacji części kodu
na stałe w stosunku do odpowiednika modułowego, przynajmniej jeśli chodzi o działanie, bo przecie w
przypadku modułu trzeba doliczyć trochę czasu na jego załadowanie.

### Rozmiar kernela i obrazu initramfs/initrd

Poniżej porównany został rozmiar samego kernela oraz obrazu initramfs, które są ładowane do pamięci
komputera zaraz po jego uruchomieniu.

    # ls -alh /boot/ | egrep "initrd|vmlinuz"
    -rw-r--r--  1 root root  32M 2018-12-31 02:52:17 initrd.img-4.19.0-1-amd64
    -rw-r--r--  1 root root 9.9M 2019-01-01 16:58:10 initrd.img-4.19.13-amd64-morficzny
    -rw-r--r--  1 root root 5.0M 2018-12-30 10:04:03 vmlinuz-4.19.0-1-amd64
    -rw-r--r--  1 root root 8.4M 2019-01-01 16:46:34 vmlinuz-4.19.13-amd64-morficzny

Powyższe pliki dotyczą tego samego kernela, z tym, że ten mający sufiks `-morficzny` został
zbudowany przeze mnie, a nie przez ludzi z dystrybucji Debiana. Ja wbudowałem całą funkcjonalność
kernela odpowiedzialną za prawidłowe działanie mojego laptopa na stałe. Tak samo postąpiłem w
przypadku pozostałych urządzeń, które od czasu do czasu podłączam do niego.

Oczywiście powyższe pliki są skompresowane i by porównać ich faktyczne rozmiary, to trzeba te pliki
pierw wypakować:

    # /usr/src/linux-source-4.19/scripts/extract-vmlinux /boot/vmlinuz-4.19.13-amd64-morficzny > kernel-morficzny
    # /usr/src/linux-source-4.19/scripts/extract-vmlinux /boot/vmlinuz-4.19.0-1-amd64  > kernel-debiana

    # ls -alh /tmp/kernel-*
    -rw-r--r-- 1 root root 27M 2019-01-01 18:45:01 /tmp/kernel-debiana
    -rw-r--r-- 1 root root 38M 2019-01-01 18:44:49 /tmp/kernel-morficzny

Widać wyraźnie, że mój kernel jest o ponad 40% większy od tego dystrybucyjnego . Nie oznacza to, że
będzie on zjadał więcej pamięci RAM -- bo przecie ten dystrybucyjny kernel musi sobie jeszcze
załadować moduły, które zostały wgrane na dysk.

Jeśli natomiast chodzi o obraz initramfs, to sprawa prezentuje się następująco:

    # unmkinitramfs /boot/initrd.img-4.19.13-amd64-morficzny  /tmp/initramfs-morficzny
    # unmkinitramfs /boot/initrd.img-4.19.0-1-amd64  /tmp/initramfs-debiana

    # du -hm /tmp/initramfs-*
    104     /tmp/initramfs-debiana/main
    2       /tmp/initramfs-debiana/early
    105     /tmp/initramfs-debiana
    21      /tmp/initramfs-morficzny/main
    2       /tmp/initramfs-morficzny/early
    22      /tmp/initramfs-morficzny

Wartości w pierwszej kolumnie są w megabajtach. Widać zatem, że mój obraz initramfs już jest
znacznie mniejszy od tego dystrybucyjnego, bo w przeciwieństwie do niego nie zawiera całej masy
zbędnych modułów. Oczywiście różnica na poziomie parudziesięciu megabajtów raczej w dzisiejszych
czasach nikomu snu z powiek nie spędzi. Podobnie jak i ładowanie modułów z wolnego HDD czy nawet
szybszego SSD, bo to jest kwestia co najwyżej kilku czy kilkunastu sekund.

### Jaka funkcjonalność kernela została wkompilowana na stałe

Po zresetowaniu maszyny, ta powinna się nam uruchomić bez większego problemu, chyba, że za dużo
rzeczy ręcznie wyłączyliśmy. Mi póki co udało się pozbawić mojego laptopa klawiatury, dźwięku,
filtra pakietów (iptables), no i też przez złe dobranie modułów kryptograficznych nie mogłem
odszyfrować swojego systemu ale wszystkie te rzeczy zostały już poprawione. To co jednak może się
popsuć na samym początku, to usługa systemd odpowiedzialna za ładowanie modułów podczas startu
systemu. Warto jest się zatem zainteresować tym, co mamy w katalogu `/etc/modules-load.d/`. Błędy
będą powodowane jedynie przez brak konkretnego modułu. Jeśli taka funkcjonalność kernela zostanie
wkompilowana na stałe, to system zwróci nam jedynie stosowną informacje:

    systemd-modules-load[582]: Module 'loop' is builtin

Trzeba też pamiętać o fakcie, że wszystkie wbudowane moduły na stałe nie będą już widoczne w
`lsmod` lub w pliku `/proc/modules` . Jeśli chcemy sprawdzić jaka funkcjonalność została wbudowana
kernel i czy są te moduły, których się spodziewamy, to możemy podejrzeć konfigurację kernela:

    # cat /boot/config-$(uname -r)
    # zcat /proc/config.gz

Możemy także zajrzeć do pliku `modules.builtin` w katalogu z modułami kernela:

    # cat /lib/modules/$(uname -r)/modules.builtin

W przypadku, gdy kompilujemy wszystkie opcje kernela na stałe, to dobrze jest jeszcze sprawdzić czy
nie zostały nam jakieś niedobitki:

    # egrep \=m /boot/config-4.19.13-amd64-morficzny

Jeśli to powyższe polecenie nam nie zwróci żadnego wyniku, to cała funkcjonalność kernela została
wkompilowana w niego na stałe.

### Parametry modułów wkompilowanych na stałe

Jeśli zdarzyło nam się konfigurować parametry modułów, to raczej powinniśmy być zaznajomieni z
katalogiem `/etc/modprobe.d/` . W przypadku, gdy moduły mamy wkompilowane na stałe, to konfiguracja
z tego katalogu nam zwyczajnie nie zadziała. Niemniej jednak, w dalszym ciągu możemy parametry
modułów zmieniać. Trzeba to tylko robić przez "kernel command line", którą możemy uzupełnić na dwa
sposoby.

Pierwszym sposobem jest edycja konfiguracji bootloader'a, gdzie mamy wpis podobny do tego poniżej:

    ...
    APPEND root=/dev/mapper/wd_black_label-root ... loop.max_part=63 ro
    ...

Drugim sposobem jest określenie stosownej opcji w konfiguracji źródeł kernela:

![](/img/2018/12/004.debian-kernel-xconfig.png#huge)

Oba te sposoby są w zasadzie podobne, bo wynikowa linijka kernela widziana pod `/proc/cmdline`
będzie dokładnie taka sama, tj. zawierać te same parametry i te same
wartości. [Sposób określania parametrów][17] w linijce kernela jest następujący. Najpierw określamy
nazwę modułu, w tym przypadku jest to `loop` . Później dajemy `.` , która oddziela nazwę modułu od
jego parametru podawanego po kropce, tj. `max_part`. Temu parametrowi można przypisać wartość i
dlatego po nim występuje `=` i pożądana wartość `63` .

Wiemy zatem jak skonfigurować moduł ale skąd wziąć parametry modułów? Można odczytać je z `modinfo`:

    # modinfo loop
    ...
    parm:           max_loop:Maximum number of loop devices (int)
    parm:           max_part:Maximum number of partitions per loop device (int)

Można także użyć `systool`:

    # systool -v -m loop
    ...
      Parameters:
        max_loop            = "0"
        max_part            = "63"

W przypadku `systool` widać także aktualną konfigurację modułu.

Narzędzia `modinfo` jest w stanie nam pomóc jedynie w przypadku, gdy mamy załadowany kernel mający
moduły, czyli jeszcze nie przelogowaliśmy się na nasze świeżo ugotowane jajo. Gdy to zrobimy, to
`modinfo` zwróci nam jedynie błąd, że moduł nie został znaleziony:

    #  modinfo loop
    modinfo: ERROR: Module loop not found.

Więc albo posiłkujmy się narzędziem `systool` , albo trzeba zajrzeć do katalogu
`/sys/module/nazwa_modułu/parameters/` .

### Całkowite wyłączenie obsługi modułów

Gdy zamierzamy całkowicie zrezygnować z modułów i wkompilować całą potrzebną nam funkcjonalność
kernela na stałe, to można by się pokusić o całkowite wyłączenie obsługi modułów w kernelu.
Przemawiać może za tym poprawa bezpieczeństwa (nikt żadnych modułów już nie będzie w stanie
załadować) oraz prostszy w budowie kernel, co z kolei przyśpieszy jego pracę. Niemniej jednak,
odhaczenie `CONFIG_MODULES` sprawia, że niektóre aplikacje mogą mieć problemy z ustaleniem czy
pewna funkcjonalność jest oferowana przez kernel. Jedną z takich aplikacji jest `containerd` ,
który zwraca poniższy błąd:

    modprobe: ERROR: ../libkmod/libkmod.c:514 lookup_builtin_file() could not open builtin file '/lib/modules/4.19.13-amd64-morficzny/modules.builtin.bin'
    modprobe: FATAL: Module overlay not found in directory /lib/modules/4.19.13-amd64-morficzny

Niby wszystkie potrzebne moduły zostały wbudowane w kernel ale z racji braku pliku
`modules.builtin.bin` usługa się nie chce podnieść. Zwykle wystarczy poszukać w usługach wywołań z
`modprobe` i je wykomentować:

    ...
    [Service]
    #ExecStartPre=/sbin/modprobe overlay
    ExecStart=/usr/bin/containerd
    ...

Podobnie sprawa ma się z usługą `systemd-modules-load.service` , która również się nam nie odpali i
zwróci błąd. I tę usługę wypadałoby także wyłączyć.

    # systemctl mask systemd-modules-load.service

Warto też zajrzeć do plików w katalogu `/etc/initramfs-tools/` i przejrzeć je pod kątem
ewentualnych prób ładowania modułów.

O ile usługi oraz konfigurację systemową można bez problemu poprawić, to gdy zamierzamy instalować
paczki mające w nazwie `*-dkms` , to niestety nie możemy wyłączyć całkowicie obsługi modułów,
bo [mechanizm DKMS][18] w końcu buduje moduły, które będą ładowane, gdy zajdzie taka potrzeba.

#### Wyłączenie możliwości ładowania modułów podczas pracy systemu

Jeśli całkowite wyłączenie modułów w kernelu generuje u nas sporo problemów, to zawsze można
skorzystać z mniej inwazyjnej opcji jaką jest wyłączenie możliwości ładowania modułów podczas pracy
systemu. Tutaj już wystarczy ustawić tylko tę poniższą opcję w pliku `/etc/sysctl.conf` :

    kernel.modules_disabled = 1

Trzeba jednak pamiętać, że `kernel.modules_disabled` chroni nas jedynie od momentu ustawienia tej
opcji -- nie da się jej przestawić z `1` na `0` bez restartu systemu. W dalszym ciągu jednak trefne
moduły mogą zostać załadowane na starcie systemu.

## Nakładanie łatek na kernel (patch)

Niekiedy pojawiają się ciekawe łatki na kernel, które z jakiegoś powodu (czasami zasadnego, np.
wywołują BUG'i w kernelu) nie są aplikowane na kernel dystrybucyjny. Przykładem takiej łatki może
być [TPE (Trusted Path Execution)][19]. Mając własny kernel, można taką łatę sobie pobrać i
zaaplikować. Proces nakładania łatek może być automatyczny lub manualny, w zależności od tego czy
potrafimy korzystać z `quilt`'a.

Standardowo, taki patch pobiera się do katalogu, w którym mamy również katalog kernela, czyli w
tym przypadku jest to `/usr/src/` po czym przechodzi się do katalogu kernela i aplikuje patch w
poniższy sposób:

    # cd /usr/src/linux-4.19.12/
    # patch --dry-run -p1 < ../v2-1-1-Add-Trusted-Path-Execution-as-a-stackable-LSM.diff
    # patch -p1 < ../v2-1-1-Add-Trusted-Path-Execution-as-a-stackable-LSM.diff

A gdy zachodzi potrzeba ściągnąć łatkę, to robimy to przy pomocy parametru `-R`:

    # patch -R --dry-run -p1 < ../v2-1-1-Add-Trusted-Path-Execution-as-a-stackable-LSM.diff
    # patch -R -p1 < ../v2-1-1-Add-Trusted-Path-Execution-as-a-stackable-LSM.diff

Jeśli patch składa się z kilku plików, to nakładamy te pliki w kolejności numerycznej od
najmniejszego. W przypadku ściągania takiego patch'a robimy to w odwrotnej kolejności, czyli
ostatni zaaplikowany plik ściągamy w pierwszej kolejności.

Problem z takim zakładaniem i ściganiem patch'y jest taki, że nie mamy żadnego mechanizmu
śledzącego zmiany, których dokonujemy na źródłach kernela. Gdy mamy do dyspozycji tylko kilka
łatek, to taka sytuacja raczej nam wielkiego problemu nie przysporzy ale jeśli w grę wchodzi wiele
patch'y, to już łatwo można się pogubić. Dlatego zamiast ogarniać sprawę łat ręcznie, dobrze jest
zaopatrzyć się w `quilt` . To narzędzie może nie być domyślnie zainstalowane. Także jak coś to
trzeba je doinstalować:

    # aptitude install quilt

Tworzymy sobie plik konfiguracyjny dla `quilt` , tj. `/root/.quiltrc` o poniższej zawartości:

    # cat /root/.quiltrc
    export QUILT_PATCHES="debian/patches"
    export QUILT_PUSH_ARGS="--color=auto"
    export QUILT_DIFF_ARGS="--no-timestamps --no-index -p ab --color=auto"
    export QUILT_REFRESH_ARGS="--no-timestamps --no-index -p ab"
    export QUILT_DIFF_OPTS='-p'

Z powyższego pliku najbardziej interesuje nas zmienna `$QUILT_PATCHES` , bo to ona definiuje w
jakim katalogu będą się znajdować łaty. Ścieżka do katalogu jest względna w stosunku do katalogu
roboczego, tj. źródeł kernela. Podczas budowania paczki z kernelem dla Debiana zostanie stworzony
katalog `debian/` (i szereg plików w nim) w głównym katalogu ze źródłami kernela. Jeśli nie chcemy
umieszczać łatek w `debian/patches/` , to zawsze możemy ten katalog usunąć i stworzyć na jego
miejsce dowiązanie symboliczne, np. do katalogu `/usr/src/kernel-patches/` :

    # rmdir debian/patches
    # ln -s /usr/src/kernel-patches/ ./debian/patches

W ten sposób wszystkie łatki zawsze będą rezydować poza źródłami kernela i sobie ich przez
przypadek nie usuniemy, gdy będziemy usuwać stare źródła.

Mając przygotowanego `quilt`'a, możemy zaimportować łatę i nałożyć ją w poniższy sposób:

    # quilt import /sciezka/do/pliku
    # quilt push

![](/img/2018/12/005.debian-kernel-patch.png#huge)

Łatkę ściągamy za to tak:

    # quilt pop

Jak chcemy ściągnąć lub nałożyć wszystkie łaty naraz, to dajemy:

    # quilt push -a
    # quilt pop -a

Po nałożeniu łątek, zwykle w `xconfig` będziemy mieli do zaznaczenia kilka dodatkowych opcji. Jeśli
nie chce nam się ich szukać (przez tą zbugowaną szukajkę), to możemy zamiast `make xconfig` dać
`make oldconfig` , który wypisze nam wszystkie nowe opcje w stosunku do tych określonych już w
konfiguracji.

![](/img/2018/12/006.debian-kernel-oldconfig.png#huge)

Dla pewności możemy jeszcze zajrzeć do `xconfig`:

![](/img/2018/12/007.debian-kernel-xconfig-v.png#huge)

Po nałożeniu łaty, kernel trzeba skompilować jeszcze raz. W zależności od dokonanych zmian w
konfiguracji, rekompilacja/kompilacja może dotyczyć tylko części kodu kernela. Więc nie trzeba
kompilować kernela od początku mając już wcześniej skompilowane źródła, no chyba, że wyda się
`make clean`. Dlatego też jeśli zmieniamy tylko nieznacznie konfigurację kernela, to paczki `.deb`
z gotowym do zainstalowania kernelem możemy zrobić sobie bardzo szybko, no chyba, że wyjdzie nowsza
wersja kernela, to wtedy niestety kompilacja będzie przebiegać od początku, a to może potrwać
chwilę.


[1]: https://www.kernel.org/
[2]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
[3]: https://pl.wikipedia.org/wiki/Executable_and_Linkable_Format
[4]: https://manpages.debian.org/unstable/dpkg-dev/dpkg-buildflags.1.en.html
[5]: http://man7.org/linux/man-pages/man1/gcc.1.html
[6]: https://en.wikipedia.org/wiki/Stack_buffer_overflow
[7]: https://blog.qualys.com/securitylabs/2017/06/19/the-stack-clash
[8]: https://en.wikipedia.org/wiki/Address_space_layout_randomization
[9]: https://en.wikipedia.org/wiki/Position-independent_code
[10]: https://wiki.gentoo.org/wiki/Project:Quality_Assurance/As-needed
[11]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
[12]: https://en.wikipedia.org/wiki/Global_Offset_Table
[13]: https://wiki.gentoo.org/wiki/GCC_optimization
[14]: https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/
[15]: https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project/Recommended_Settings
[16]: https://en.wikipedia.org/wiki/Xconfig
[17]: https://people.freedesktop.org/~narmstrong/meson_drm_doc/admin-guide/kernel-parameters.html
[18]: /post/dkms-czyli-automatycznie-budowane-moduly/
[19]: https://patchwork.kernel.org/patch/9773791/
[20]: https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html
