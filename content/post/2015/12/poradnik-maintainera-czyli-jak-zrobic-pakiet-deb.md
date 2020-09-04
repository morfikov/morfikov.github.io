---
author: Morfik
categories:
- Linux
date: "2015-12-07T20:26:45Z"
date_gmt: 2015-12-07 19:26:45 +0100
published: true
status: publish
tags:
- debian
title: Poradnik maintainer'a, czyli jak zrobić pakiet deb
---

Debian posiada bardzo rozbudowany system robienia pakietów. Generalnie rzecz biorąc, to wszystkie z
nich musiały przejść przez ten proces zanim trafiły do głównego repozytorium dystrybucji. Dzięki
takiemu stanu rzeczy, nie musimy ręcznie powielać pracy szeregu innych osób i odpada nam
własnoręczna kompilacja pakietów, a wszyscy wiemy, że zajmuje ona cenny czas i zasoby. Paczki
`.deb` są tworzone ze źródeł i instalowane przy pomocy menadżera pakietów `aptitude`/`apt`/`dpkg` .
Nic jednak nie stoi na przeszkodzie by daną aplikację skompilować sobie ręcznie i zainstalować ją
przy pomocy `make install` . Problem w tym, że w taki sposób robi się śmietnik w naszym systemie i
śledzenie wszystkich zainstalowanych w ten sposób pakietów w pewnym momencie stanie się wręcz
niemożliwe. Dlatego też przydałby nam się mechanizm, który ułatwiłby nam nieco to zadanie. Debian
udostępnia szereg narzędzi, które są w stanie w pełni zautomatyzować cały ten proces budowy
pakietów. Ten poradnik zaś ma na celu zebranie wszystkich istotniejszych informacji związanych z
obsługą narzędzi takich jak `dh_make` , `dpkg-buildpackage` , `pbuilder` , `quilt` czy `lintian` ,
tak by tworzyć pakiety w prosty sposób i przy tym równając do najwyższych standardów debiana.

<!--more-->
## Potrzebne pakiety

Przeciętny użytkownik debiana raczej nie będzie w stanie zbudować własnej paczki `.deb` ,
przynajmniej nie od razu. Nie musimy też rzucać się na głęboką wodę i możemy zwyczajnie zacząć od
prostszej czynności jaką jest aktualizacja konkretnego pakietu, który już jest dostępny w
repozytorium debiana (zobacz rozdział 9). Wszystkie paczki `.deb` wymagają tego by nimi się
opiekować, np. dostosowywać zależności, zgłaszać błędy upstream'owi i tego typu rzeczy. Nie jest to
specjalnie trudne ale największym wyzwaniem jest "debianizacja źródeł". Ten proces polega na
odpowiednim dostosowaniu katalogu `debian/` , który zostanie utworzony w głównym folderze ze
źródłami aplikacji i nasze zadanie sprowadzać się będzie do odpowiedniego uzupełniania
określonych plików w tym katalogu. W przypadku pakietów już dostępnych w dystrybucji, debianizacja
źródeł już się odbywała i przeprowadził ją za nas ktoś inny. Dlatego aktualizacja tych pakietów
zwykle nie powinna stwarzać problemów, a nawet jeśli już, to jest ich sporo mniej niż w przypadku,
gdy byśmy budowali tę paczkę od podstaw.

Poniżej znajduje się lista narzędzi wraz z ich krótkim opisem. To przy ich pomocy będziemy
odpowiednio [dostosowywać katalog debian/](https://www.debian.org/doc/manuals/maint-guide/). Spora
część z nich jest w pełni zautomatyzowana i od nas wymagać będzie się jedynie ustawienia określonych
przełączników przy wydawaniu konkretnych poleceń.

  - `packaging-dev` -- meta pakiet, który zainstaluje min:
      - `build-essential` -- bez tego nie ma nawet co podchodzić do budowania pakietów.
      - `debhelper` -- szereg programów zajmujących się obrabianiem pliku `debian/rules` .
      - `dh-make` -- debianizuje źródła (tworzy katalog `debian/`).
      - `devscripts` -- ten pakiet zawiera szereg skryptów ułatwiających zarządzanie plikami w
        katalogu `debian/` , więcej info o tym jakie skrypty ten pakiet zawiera (oraz ich
        zależności) można odczytać z opisu samego pakietu, np. w `aptitude` .
      - `lintian` -- weryfikuje poprawność zbudowanych pakietów.
      - `pbuilder` -- buduje paczki `.deb` .
      - `quilt` -- tworzy łaty (i zarządza nimi) w oparciu o zmiany dokonywane na źródłach.
  - `gnupg` -- obsługa kluczy GPG, wymagane do podpisywania plików `.dsc` i `.changes` .
  - `desktop-file-utils` -- weryfikuje składnię plików `.desktop` .
  - `ccache` -- przyśpiesza znacznie rekompilację pakietów.
  - `dh-*` -- różne moduły dla debhelper'a, potrzebne podczas budowania pewnych pakietów, np. te
    posiadające pliki `.service` dla systemd wymagać będą `dh-systemd` , etc.
  - `gmanedit` -- ułatwia tworzenie manuali.
  - `help2man` -- generuje manualne dla plików wykonywalnych w oparciu o parametr `--help` .
  - `piuparts` -- testuje instalację, usuwanie i aktualizację zbudowanych pakietów.
  - `apt-file` -- przeszukuje pakiety w oparciu o wzorzec nazwy pliku.
  - `patchutils`, `wdiff` -- dodatkowe narzędzia do pracy z łatami.
  - `blhc` -- testuje log z budowania pakietu pod kątem braku określonych flag.
  - `hardening-includes` -- zawiera narzędzie `hardening-check` pomagające ustalić poziom
    bezpieczeństwa plików wykonywalnych.
  - `python-minimal` -- zawiera narzędzie `pyversions` pomagające ustalić wersję python'a.
  - `binutils`, `strace` -- `binutils` zawiera narzędzie `dpkg-depcheck` , które przy pomocy
    `strace` może pomóc w ustalaniu zależności.
  - `fakeroot` -- symuluje uprawnienia super użytkownika.
  - `sudo` -- przekazuje część uprawnień administratora root zwykłemu użytkownikowi.

Wato też nadmienić, że skoro mamy do czynienia z budową pakietów, to przydałby się gdzieś te pakiety
umieszczać. Jest raczej mało prawdopodobne, że dostaną się do głównego repozytorium debiana, dlatego
też dobrze jest się pierw pokusić o [stworzenie własnego repozytorium przy pomocy
reprepro]({{< baseurl >}}/post/tworzenie-repozytorium-przy-pomocy-reprepro/) i to w nim umieszczać
pakiety, które będziemy instalować przy pomocy `aptitude` czy też `apt` .

Jak widać jest tego dość sporo, no i oczywiście będzie tego więcej, gdy każdy z powyższych pakietów
dociągnie swoje zależności. Poza tymi powyższymi, dojdą nam jeszcze pewnie inne pakiety, w
zależności od tego co tak naprawdę będziemy budować.

Dla ułatwienia, poniżej jest linijka instalująca wszystkie powyższe narzędzia:

    # aptitude install packaging-dev gnupg desktop-file-utils ccache \
    gmanedit help2man piuparts apt-file patchutils wdiff blhc hardening-includes \
    python-minimal binutils strace fakeroot

Powyższe pakiety (za wyjątkiem pewnych dodatków) są jedynymi pakietami, które będziemy instalować w
swoim głównym systemie. Wszelkie zależności budowanych pakietów będą już pakowane do [środowiska
chroot]({{< baseurl >}}/post/przygotowanie-srodowiska-chroot-do-pracy/), które zostanie utworzone
automatycznie przez `pbuilder` . Takie rozwiązanie pomoże nam utrzymać porządek w systemie.

Upychanie plików w paczkach ma na celu jedynie ich organizację i niczym zbytnio nie różni się od
zwykłego przekopiowania ich do odpowiednich katalogów (czy zainstalowania via `make install` ), no
może poza faktem, że w przypadku paczki, wszystkie pliki są śledzone przez menadżer pakietów. Chodzi
generalnie o to, że w przypadku, gdyby się pojawiła nowsza wersja jakiegoś programu, lub zwyczajnie
chcielibyśmy się pozbyć go z systemu, to może być to dość problematyczne. W przypadku jednego czy
dwóch programów, które nie mają za wiele plików, usunięcie ich raczej nie powinno sprawić problemu,
a co w przypadku bardziej rozbudowanych projektów? Część z nich potrafi się pomyślnie odinstalować
sama jeśli skrypt `makefile` obsługuje taki ficzer ale nawet w takim przypadku trzeba, po pierwsze,
trzymać źródła, a po drugie, musimy zainstalować wszystkie zależności potrzebne do zbudowania
takiego pakietu w swoim systemie. Natomiast w przypadku budowania paczek, te problemy nas nie
dotyczą.

## Konfiguracja narzędzi

Zanim jednak przejdziemy do omawiania poszczególnych plików, które mogą (ale nie muszą) znajdować
się w katalogu `debian/` , musimy poświęcić chwilę czasu na dostosowanie konfiguracji tych
wszystkich powyższych narzędzi.

### dh-make

Standardowo przy wywoływaniu polecenia `dh_make` podczas debianizowania źródeł, musimy podawać kilka
dodatkowych parametrów, tak by określić kto buduje paczkę. Zamiast tego, możemy wyeksportować
odpowiednie zmienne i uprościć nieco cały ten proces. W tym celu dodajemy poniższe linijki do [pliku
konfiguracyjnego shella, .bashrc]({{< baseurl >}}/post/plik-bashrc-czyli-konfiguracja-basha/):

    DEBEMAIL="morfik@nsa.com"
    DEBFULLNAME="Mikhail Morfikov"
    export DEBEMAIL DEBFULLNAME

W oparciu o te zmienne, szereg narzędzi będzie uzupełniać odpowiednie pola, przykładowo plik
`changelog` .

### gnupg

Jeśli planujemy robić z paczkami coś większego, np. przesyłać je do repozytorium debiana, to
będziemy potrzebować kluczy GPG, by taką paczkę podpisać. Nie będę tutaj opisywał całego procesu
generowania i konfigurowania kluczy GPG, bo zostało to już zrobione w odpowiednich wpisach na tym
blogu. Zachęcam zatem do zapoznania się z tymi artykułami: [Bezpieczny klucz
GPG]({{< baseurl >}}/post/bezpieczny-klucz-gpg/), [Konfiguracja GPG w pliku
gpg.conf]({{< baseurl >}}/post/konfiguracja-gpg-w-pliku-gpg-conf/) oraz [Serwer kluczy GPG i
kwestia prywatności]({{< baseurl >}}/post/serwer-kluczy-gpg-i-kwestia-prywatnosci/).

### devscripts

Jak wspomniałem na początku, w tym pakiecie znajduje się szereg użytecznych skryptów, które będziemy
wykorzystywać w swojej pracy. Jednym z częściej używanych narzędzi będzie `debsign` , które to
będzie nam podpisywało pliki `.changes` i `.dsc` zawierające między innymi sumy kontrolne. I tu,
podobnie jak w przypadku narzędzia `dh-make` , możemy określić kila parametrów, tak by nie musieć
ich wpisywać za każdym razem, gdy chcemy podpisać jakąś paczkę.

Konfiguracja pakietu `devscripts` jest trzymana w pliku `/etc/devscripts.conf` . Dobrze jest
przejrzeć ten plik, choć objętościowo może nieco przytłoczyć. W każdym razie interesujące nas opcje
to:

    ..
    DEBSIGN_PROGRAM=gpg
    ...
    DEBSIGN_SIGNLIKE=gpg
    ...
    DEBSIGN_KEYID=B820057A
    ...

Możemy także zweryfikować wprowadzone dane wydając poniższe polecenie:

    $ debsign --help
    ...
    Default settings modified by devscripts configuration files:
      DEBSIGN_PROGRAM=gpg
      DEBSIGN_KEYID=B820057A

### lintian

Konfiguracja [lintian'a](https://lintian.debian.org/manual/index.html) trzymana jest w pliku
`/etc/lintianrc` i w dużej mierze odpowiada za to jaki rodzaj błędów będzie nam pokazywany. Poniżej
mój plik:

    # /etc/lintianrc -- Lintian configuration file
    #
    # Note, that Lintian has reasonable default values for all variables
    # specified below. Thus, you don't have to change this file unless you
    # want something special.
    #
    # Also note, that this file uses a special syntax:
    # Empty lines are allowed, comments are introduced by a hash sign (#).
    # All other lines must have the format
    #    VAR=text
    # or
    #    VAR="text"
    # It is allowed to use '~' and '$HOME' in the variables, but not other
    # shell/environment variables.

    # Enable info tags by default (--display info)
    display-info = yes

    # Enable pedantic tags by default (--pedantic)
    pedantic = yes

    # Enable experimental tags by default (--display-experimental)
    display-experimental = no

    # Enable colored output for terminal output (--color)
    color = always

    # Show overridden tags (--show-overrides)
    #show-overrides = yes

    # Ignore all overrides (--no-override)
    #override = no

    # Verbose output by default (--verbose)
    verbose = no

    info = no

    # Quiet by default (--quiet)
    #quiet = yes

    # Specify a laboratory--a directory where Lintian should store some info
    # about packages being checked.
    LINTIAN_LAB="/media/Kabi/pbuilder/lintian"

    # Use a different directory for temporary files - useful if /tmp is a
    # tmpfs with "limited" capacity.
    #TMPDIR="/var/tmp"

Jeśli jesteśmy początkującymi amatorami, możemy się nieco pogubić w tym co będzie próbował nam
powiedzieć `lintian` . Dobrze jest przestawić z początku opcję `verbose` oraz `info` na `yes` .
Spowoduje to wyświetlenie dość obszernej informacji na temat każdego z błędu. Zawsze można wpisać
kod błędu w google i pierwszy wynik odeśle nas na stronę lintian'a i tam na pewno znajdziemy
wyjaśnienie.

### apt-file

To narzędzie nie jest bezpośrednio powiązane z budowaniem paczek, niemniej jednak bardzo się
przydaje, a to z tego względu, że często będziemy postawieni w sytuacji, gdzie pakiet nie będzie
mógł się poprawnie zbudować i to przez brak jakiegoś pliku. Przy pomocy `apt-file` będziemy w
stanie ten plik namierzyć. Nie będę tutaj opisywał tego jak korzystać z tego narzędzia, bo to
zostało dokładnie wyjaśnione we wpisie dotyczącym
[apt-file]({{< baseurl >}}/post/przeszukiwanie-zawartosci-pakietow-apt-file/). Zachęcam zatem do
zapoznania się również i z tym tekstem.

### pbuilder

[Pbuilder](http://pbuilder.alioth.debian.org/) to narzędzie, które zrobi praktycznie cała robotę za
nas, przynajmniej jeśli chodzi o zbudowanie pakietu `.deb` . Trzeba mu tylko skonfigurować szereg
parametrów. Przykładowy plik konfiguracyjny dla pbuilder'a znajduje się w
`/usr/share/doc/pbuilder/examples/pbuilderrc` . Trzeba go skopiować do katalogu `/etc/` i
odpowiednio przerobić. Można także zrobić sobie lokalną wersję tego pliku i umieścić go w katalogu
domowym pod nazwą `.pbuilderrc` . Poniżej zaś znajduje się mój aktualny plik konfiguracyjny:

    # pbuilder defaults; edit /etc/pbuilderrc to override these and see
    # pbuilderrc.5 for documentation

    BASETGZ=/media/Kabi/pbuilder/base.tgz
    EXTRAPACKAGES="apt-utils debconf-utils ccache eatmydata libfile-fcntllock-perl"
    #export DEBIAN_BUILDARCH=athlon
    BUILDPLACE=/media/Kabi/pbuilder/build/
    MIRRORSITE=http://ftp.de.debian.org/debian/
    #OTHERMIRROR="deb http://www.home.com/updates/ ./"
    #export http_proxy=http://your-proxy:8080/
    USEPROC=yes
    USEDEVPTS=yes
    USENETWORK=no
    USERUNSHM=yes
    USEDEVFS=no
    BUILDRESULT=/media/Kabi/pbuilder/result/

    # specifying the distribution forces the distribution on "pbuilder update"
    DISTRIBUTION=sid
    # specifying the architecture passes --arch= to debootstrap; the default is
    # to use the architecture of the host
    ARCHITECTURE='dpkg --print-architecture'
    # specifying the components of the distribution, for instance to enable all
    # components on Debian use "main contrib non-free" and on Ubuntu "main
    # restricted universe multiverse"
    COMPONENTS="main"
    #specify the cache for APT
    APTCACHE="/media/Kabi/pbuilder/pbuilder_apt_cache/"
    APTCACHEHARDLINK="no"
    REMOVEPACKAGES="no"
    #HOOKDIR="/usr/lib/pbuilder/hooks"
    HOOKDIR="/media/Kabi/pbuilder/hooks"
    # NB: this var is private to pbuilder; ccache uses "CCACHE_DIR" instead
    # CCACHEDIR="/var/cache/pbuilder/ccache" ccache -M 20G
    CCACHEDIR="/media/Kabi/pbuilder/ccache"
    export CCACHE_DIR="/media/Kabi/pbuilder/ccache"

    # make debconf not interact with user
    export DEBIAN_FRONTEND="noninteractive"

    #for pbuilder debuild
    BUILDSOURCEROOTCMD="fakeroot"
    PBUILDERROOTCMD="sudo -E"
    # use cowbuilder for pdebuild
    #PDEBUILD_PBUILDER="cowbuilder"

    # additional build results to copy out of the package build area
    #ADDITIONAL_BUILDRESULTS=(xunit.xml .coverage)

    # command to satisfy build-dependencies; the default is an internal shell
    # implementation which is relatively slow; there are two alternate
    # implementations, the "experimental" implementation,
    # "pbuilder-satisfydepends-experimental", which might be useful to pull
    # packages from experimental or from repositories with a low APT Pin Priority,
    # and the "aptitude" implementation, which will resolve build-dependencies and
    # build-conflicts with aptitude which helps dealing with complex cases but does
    # not support unsigned APT repositories
    #PBUILDERSATISFYDEPENDSCMD="/usr/lib/pbuilder/pbuilder-satisfydepends"
    PBUILDERSATISFYDEPENDSCMD="/usr/lib/pbuilder/pbuilder-satisfydepends-experimental"

    # Arguments for $PBUILDERSATISFYDEPENDSCMD.
    # PBUILDERSATISFYDEPENDSOPT=()

    # You can optionally make pbuilder accept untrusted repositories by setting
    # this option to yes, but this may allow remote attackers to compromise the
    # system. Better set a valid key for the signed (local) repository with
    # $APTKEYRINGS (see below).
    ALLOWUNTRUSTED=no

    # Option to pass to apt-get always.
    export APTGETOPT=()
    # Option to pass to aptitude always.
    export APTITUDEOPT=()

    #Command-line option passed on to dpkg-buildpackage.
    #DEBBUILDOPTS="-IXXX -iXXX"
    DEBBUILDOPTS="-j1 -pgpg -kB820057A -sa"

    #APT configuration files directory
    APTCONFDIR="/media/Kabi/pbuilder/pbuilder_apt_conf/"

    # the username and ID used by pbuilder, inside chroot. Needs fakeroot, really
    BUILDUSERID=1234
    BUILDUSERNAME=morfik

    # BINDMOUNTS is a space separated list of things to mount
    # inside the chroot.
    BINDMOUNTS="${CCACHE_DIR}"

    # Set the debootstrap variant to 'buildd' type.
    DEBOOTSTRAPOPTS=(
        '--variant=buildd'
        '--keyring' '/usr/share/keyrings/debian-archive-keyring.gpg'
        )
    # or unset it to make it not a buildd type.
    # unset DEBOOTSTRAPOPTS

    # Keyrings to use for package verification with apt, not used for debootstrap
    # (use DEBOOTSTRAPOPTS). By default the debian-archive-keyring package inside
    # the chroot is used.
    APTKEYRINGS=()

    # Set the PATH I am going to use inside pbuilder: default is "/usr/sbin:/usr/bin:/sbin:/bin"
    #export PATH="/usr/sbin:/usr/bin:/sbin:/bin"
    export PATH="/usr/lib/ccache:${PATH}"

    # SHELL variable is used inside pbuilder by commands like 'su'; and they need sane values
    export SHELL=/bin/bash

    # The name of debootstrap command, you might want "cdebootstrap".
    DEBOOTSTRAP="debootstrap"

    # default file extension for pkgname-logfile
    PKGNAME_LOGFILE_EXTENSION="_$(dpkg --print-architecture).build"

    # default PKGNAME_LOGFILE
    PKGNAME_LOGFILE=""

    # default AUTOCLEANAPTCACHE
    AUTOCLEANAPTCACHE="no"

    #default COMPRESSPROG
    COMPRESSPROG="gzip"

W `EXTRAPACKAGES` umieściłem kilka dodatkowych pakietów, które będą instalowane po wypakowaniu
środowiska chroot, a to ze względu na szereg ostrzeżeń jakie wyrzucał mi `pbuilder` przy budowaniu
pakietów. Te pozycje tutaj nie są obowiązkowe i raczej nic się paczkom nie stanie z powodu ich
braku. Jeśli ktoś nie rozumie konkretnych opcji, to odsyłam do
[manuala](http://manpages.ubuntu.com/manpages/wily/en/man5/pbuilderrc.5.html), gdzie wszystkie z
nich są przystępnie opisane.

### ccache

W przypadku `ccache` nie musimy zbytnio nic robić, bo wszystko zostało już określone w pliku
konfiguracyjnym pbuilder'a. Konkretnie chodzi o te wpisy:

    ...
    EXTRAPACKAGES="... ccache ..."
    ...
    CCACHEDIR="/media/Kabi/pbuilder/ccache"
    export CCACHE_DIR="/media/Kabi/pbuilder/ccache"
    ...

Tylko jest jeden problem. Ten `ccache` zadziała jedynie przy wywoływaniu polecenia `pbuilder
--build` . W przypadku, gdy napotkamy błąd podczas budowy pakietu, to mamy opcję by sprawdzić co się
stało. Wtedy zostaniemy zalogowani wewnątrz środowiska chroot. Jeśli zmienimy pliki konfiguracyjne,
to wewnątrz tego chroot'a będziemy w stanie zbudować pakiet via `dpkg-buildpackage` . Niemniej
jednak, nie będziemy mieli dostępu do cache `ccache` i pakiet się będzie budował tak jakbyśmy z
niego w ogóle nie korzystali. Nie mam pojęcia czemu się tak dzieje, być może to bug albo coś nie tak
z powyższą konfiguracją, ewntualnie `pbuilder` tak po prostu już ma.

### sudo

Dostęp do narzędzia `pbuilder` jest standardowo zarezerwowany jedynie dla użytkownika root. Nie jest
to zbytnio wygodne i przydałoby się umożliwić korzystanie z niego zwykłemu użytkownikowi. W tym celu
edytujemy konfigurację `sudo` wpisując w terminalu `visudo` . Dopisujemy tam te poniższe linijki:

    Host_Alias HOSTY = localhost,morfikownia

    morfik     HOSTY = (root) NOPASSWD: /usr/sbin/pbuilder

Więcej informacji na temat samego `sudo` jak i pliku konfiguracyjnego można znaleźć
[tutaj](https://dug.net.pl/tekst/63/przewodnik_po_sudo/).

## Przygotowanie środowiska chroot

Narzędzie `puilder` musi na czymś operować, inaczej odmówi współpracy. Musimy mu stworzyć podstawowe
środowisko pracy. Chodzi o zrobienia minimalnego i do tego spakowanego chroot'a, który będzie
używany w każdym procesie budowania pakietów. Takie minimalistyczne środowisko ma na celu
zapewnienie, że pakiet będzie się budował poprawnie (chodzi o zależności) na 99% maszyn, pod
warunkiem, że się zbuduje bez problemu w tym środowisku chroot. Musimy stworzyć także szereg
katalogów, do których ścieżki podaliśmy w pliku konfiguracyjnym pbuilder'a. Dodatkowo musimy także
dostosować konfigurację dla `apt` . Jeśli chodzi zaś o same repozytoria, to raczej poniżej sida nie
ma co schodzić. Jeśli jakichś pakietów nie ma w repo testowym czy stabilnym, to albo przez licencję,
albo przez niespełnione zależności, które zwykle muszą być w najnowszych wersjach, a te są z reguły
w sidzie i/lub experimental, bo powodują problemy. Tworzymy zatem potrzebne nam katalogi:

    $ mkdir /media/Kabi/pbuilder/

    $ mkdir /media/Kabi/pbuilder/{hooks,build,result,pbuilder_apt_cache,ccache,pbuilder_apt_conf,lintian}

    $ cp /etc/apt/sources.list /media/Kabi/pbuilder/pbuilder_apt_conf/
    $ cp /etc/apt/trusted.gpg /media/Kabi/pbuilder/pbuilder_apt_conf/

W pliku `sources.list` zostawiamy jedynie wpisy od sida:

    deb     http://ftp.de.debian.org/debian/ sid main non-free contrib
    deb-src http://ftp.de.debian.org/debian/ sid main non-free contrib

Pewnie zdarzy się nam taki przypadek, że pakiet, który chcemy zbudować, ma w zależnościach inny
pakiet, który dopiero co zbudowaliśmy i jak z takiej sytuacji wybrnąć? Potrzebne nam będzie własne
repozytorium, to, o którym była mowa we wstępie. Wpisy do takiego repozytorium również dodajemy do
konfiguracji `apt` dla pbuilder'a:

    deb http://deb.morfikownia.lh/debian/ sid main contrib non-free
    deb-src http://deb.morfikownia.lh/debian/ sid main contrib non-free

Wszystkie zewnętrzne repozytoria będą wymagać publicznych kluczy GPG. Trzeba je również dostarczyć
pbuilder'owi. Kopiujemy zatem systemowy keyring:

    $ cp /etc/apt/trusted.gpg /media/Kabi/pbuilder/pbuilder_apt_conf/

Tworzymy teraz spakowane środowisko chroot. Po wydaniu poniższego polecenia, zostanie zainicjowany
`debootstrap` , który pobierze minimalnego debiana (sid), który to zostanie wstępnie skonfigurowany
i upchnięty w paczce `.tgz` :

    morfik:~$ sudo pbuilder create
    W: /root/.pbuilderrc does not exist
    I: Distribution is sid.
    I: Current time: Thu Feb 19 12:36:23 CET 2015
    I: pbuilder-time-stamp: 1424345783
    I: Building the build environment
    I: running debootstrap
    /usr/sbin/debootstrap
    I: Retrieving Release
    I: Retrieving Release.gpg
    I: Checking Release signature
    I: Valid Release signature (key id A1BD8E9D78F7FE5C3E65D8AF8B48AD6246925553)
    I: Retrieving Packages
    I: Validating Packages
    ...
    I: creating base tarball [/media/Kabi/pbuilder/base.tgz]
    I: cleaning the build env
    I: removing directory /media/Kabi/pbuilder/build//46518 and its subdirectories

W ten sposób utworzona paczka będzie wypakowywana za każdym razem, gdy będziemy budować jakiś pakiet
przy pomocy `sudo pbuilder --build` . Niemniej jednak, całe wypakowane środowisko, niezależnie od
powodzenia akcji budowania, zostanie na zakończenie usunięte. Jeśli będziemy budować wiele razy i to
tylko jeden pakiet, można edytować tego spakowanego chroot'a i doinstalować tam szereg zależności,
tak by podczas budowania nie były w kółko pobierane i instalowane. By edytować środowisko, wpisujemy
poniższe polecenie:

    $ sudo pbuilder --login --save-after-login

Każdy system, nawet ten minimalny chroot zrobiony przy pomocy `debootstrap`, wymaga by go
aktualizować w miarę regularnie, a konkretnie, przed budowaniem pakietu. By zaktualizować takie
środowisko, wpisujemy w terminalu to poniższe polecenie:

    $ sudo pbuilder --update

## Źródła i informacje o nich

W internecie jest wiele rozmaitych miejsc, gdzie można znaleźć źródła pakietu, który chcemy
zbudować. Zwykle będzie to strona projektu lub też i jakiś git, np. github. Niemniej jednak, w tych
lokalizacjach bardzo rzadko spotkamy się ze źródłami, które pozwolą nam na zbudowanie pakietu tuż po
ściągnięciu ich na dysk.

W sporej części przypadków, większość roboty jaką będziemy musieli odwalić, to zbieranie informacji
na temat samego projektu. Chodzi generalnie o takie informacje jak licencja, zależności czy autorzy.
Czasem możemy pójść nieco na skróty i skorzystać z czyjejś pracy, tak by nieco przyśpieszyć
zbieranie tych danych.

Jeśli strona projektu nie jest w najlepszym stanie i brakuje kluczowych dla nas informacji, możemy
oczywiście wysłać maila do twórcy i poprosić go o stosowne info ale to zajmuje czas. Możemy także
zajrzeć na [git debiana](https://anonscm.debian.org/cgit/webwml/packages.git/) i tam spróbować
odnaleźć interesujący nas projekt. Jeśli doszukamy się go, możemy być pewni, że część pracy
przeprowadził już ktoś za nas.

Innym miejscem jest repozytorium [AUR dystrybucji Archlinux](https://aur.archlinux.org/) . Tam
zawsze idzie coś znaleźć, tylko paczki nie są zbytnio kompatybilne z debianem i trzeba wiedzieć
gdzie i czego szukać. Przykładowo, załóżmy, że interesuje nas pakiet `monitorix` :

![]({{< baseurl >}}/img/2015/12/1.pakiet-aur-deb.png#big)

Jak widzimy pakiet został odnaleziony i mamy tam info o licencji i o zależnościach. Niemniej jednak,
to nie wszystko co możemy wyciągnąć z AUR. Tam w prawym górnym rogu jest `PKGBUILD` . Jest to plik,
na podstawie którego buduje się pakiet dla Archlinux'a. Jak on nam może pomóc? Wystarczy do niego
zajrzeć. Są tam zależności (choć nazwy pakietów mogą być inne) , patch'e (do wydobycia przez link
`Download tarball` ), a nawet wywoływane konkretne polecenia (trzeba uważać na ścieżki). Wszystkie
te informacje możemy wykorzystać przy budowaniu paczki `.deb` . W tym przypadku strona projektu jest
w miarę rozsądna i można się doszukać na niej wszystkich informacji.

## Katalog debian/

Katalog `debian/` zawsze będziemy tworzyć po wypakowaniu źródeł i to w nim będziemy dokonywać
większości zmian, dlatego też musimy poznać nieco jego strukturę. Poniżej postaram się opisać
wszystkie pliki z tego katalogu, z którymi się spotkałem budując pakiety.

Generalnie rzecz biorąc, w oparciu o dane określone w tych plikach, narzędzie
[install](http://manpages.ubuntu.com/manpages/wily/en/man1/install.1.html) będzie kopiować
odpowiednie pliki/foldery i budować z tego drzewo katalogów, które następnie zostanie przeniesione
do pakietu wynikowego. Dlatego też dobrze jest sobie przyswoić poszczególne parametry tego
polecenia, bo to ułatwi ewentualne szukanie przyczyn problemów, np z błędnymi uprawnieniami plików.

### Tworzenie szkieletu

Zakładając, że nie udało nam się znaleźć zdebianizowanych źródeł, musimy przejść przez proces ich
zmiany. Na sam początek pobieramy źródła takie jak widzimy na stronie projektu i wypakowujemy je:

    $ mkdir debian_build

    $ cd debian_build/

    $ wget http://www.monitorix.org/monitorix-3.6.0.tar.gz

    $ tar xpf monitorix-3.6.0.tar.gz

    $ cd monitorix-3.6.0/

Tworzymy teraz katalog `debian/` przy pomocy `dh_make` :

    $ dh_make -s -c gpl2 -f ../monitorix-3.6.0.tar.gz
    Maintainer name  : Mikhail Morfikov
    Email-Address    : morfik@nsa.com
    Date             : Thu, 07 Dec 2015 14:56:01 +0100
    Package Name     : monitorix
    Version          : 3.6.0
    License          : gpl2
    Type of Package  : Single
    Hit <enter> to confirm:
    Done. Please edit the files in the debian/ subdirectory now. You should also
    check that the monitorix Makefiles install into $DESTDIR and not in / .

Nazwa maintainera oraz adres email został automatycznie uzupełnione w oparciu o zmienne `DEBEMAIL`
oraz `DEBFULLNAME` , które podaliśmy wcześniej w pliku `~/.bashrc` . W zależności od tego jaki
pakiet będziemy budować, możemy skorzystać ze wzorców plików. Inaczej wyglądają one w przypadku
bibliotek, a inaczej w przypadku modułów kernela, etc. W tym przypadku budujemy pojedynczą binarkę i
dlatego skorzystaliśmy z opcji `-s`. Opcja `-c gpl2` odpowiada za uzupełnienie pliku `copyright` (o
nim później) o odpowiedni tekst licencji. Ostatnia opcja `-f` odpowiada za podanie pliku z
upstream'owymi źródłami. To w oparciu o ten plik będą robione ewentualne patche (o tym też później),
bo źródeł upstream'owych jako takich nie wolno nam zmieniać i nie należy mylić ich z plikami, które
mamy w wypakowanym katalogu.

Powinniśmy mieć już dostępny katalog `debian/` . Z kolei w katalogu nadrzędnym powinna pojawić się
dodatkowa paczka: `monitorix_3.6.0.orig.tar.gz` . Nazwa nowego pliku ze źródłami ma swój
zdefiniowany format, który wygląda tak: `pakiet_wersja.orig.tar.gz` .

Przechodzimy do katalogu `debian/` :

    $ cd debian/

Mamy w nim szereg plików, które zostały automatycznie wygenerowane ale nie wszystkie z nich zawsze
będą nam potrzebne. Dobrze jest też wiedzieć, że w przypadku, gdy skasujemy (naszym zdaniem)
niepotrzebne pliki, to możemy nakazać `dh_make` aby przywrócił ich szablony. By odzyskać skasowane
pliki, będziemy musieli wydać to poniższe polecenie:

    $ dh_make --addmissing
    ...
    File source/format exists, skipping.
    File changelog exists, skipping.
    File compat exists, skipping.
    ...

Nowe pliki powinny być już dostępne w katalogu `debian/` i jak widać wyżej, system nie ruszył tych,
które już istniały, a jedynie dodał te brakujące.

### debian/control

Jednym z ważniejszych plików jest `debian/control` . To w nim są definiowane min. zależności
potrzebne do zbudowania/instalacji pakietu i to w oparciu o te zależności właśnie menadżery
pakietów, takie jak `apt` czy `aptitude` , będą wiedzieć jak zainstalować konkretny pakiet. W tym
przypadku, ten plik wygląda następująco:

    Source: monitorix
    Section: net
    Priority: optional
    Maintainer: Mikhail Morfikov <morfik@nsa.com>
    Build-Depends: debhelper (>= 9), dh-systemd (>= 1.5)
    Standards-Version: 3.9.6
    Homepage: http://www.monitorix.org/
    Vcs-Git: git://github.com/mikaku/Monitorix.git
    Vcs-Browser: https://github.com/mikaku/Monitorix

    Package: monitorix
    Architecture: any
    Depends: ${shlibs:Depends}, ${misc:Depends}, ${perl:Depends},
     rrdtool,
     ...
    Suggests: hddtemp,
     ...
    Description: Web system monitoring tool
     ...

Trochę tego jest. Jedziemy zatem od góry. Linijka `Source` określa nazwę źródła, czyli to co
zostanie pobrane po wydaniu polecenia `apt-get source` , z tym, że bez wersji i sufix'u `.tar.gz` .

Dalej mamy `Section` i odpowiada to za przypisanie pakietu do konkretnej sekcji, a tych jest dość
sporo. Wszystkie można znaleźć [tutaj](https://packages.debian.org/unstable/). Tylko taka mała
uwaga, zwykle tam w linku spotkamy się z nazwami typu "Administration Utilities" lub "Network" i to
nie są te nazwy, które musimy wpisać do pliku `debian/control` . Jeśli klikniemy w daną sekcję, na
samej górze dopiero dostaniemy prawidłową nazwę, przykładowo: Software Packages in "sid", Subsection
`net` i to właśnie to musimy wpisać.

Kolejna linijka to `Priority` i określa ona jak ważna jest paczka z punktu widzenia prawidłowego
działania systemu i z reguły tutaj będziemy wpisywać `optional` albo `extra` . Różnica między tymi
dwoma polega na tym, że ten pierwszy nie koliduje z żadnymi pakietami, które mają ustawiony
priorytet `required` , `important` lub `standard` , czyli pakiety wchodzące w skład bardzo
podstawowej instalacji systemu.

Następnie mamy `Maintainer` , czyli osobę, która budowała i będzie się opiekować danym pakietem.

Kolejna pozycja to `Build-Depends` i jest to nic innego jak zależności, które muszą zostać
spełnione, aby podjąć próbę budowania pakietu. Jeśli jakaś zależność nie zostanie spełniona,
`pbuilder` nawet nie podejmie się próby stworzenia takiego pakietu. Nie zawsze też pakiet zbuduje
się nam poprawnie w przypadku, gdy wszystkie zależności zostaną zaspokojone. Wtedy będzie trzeba
poszukać tych brakujących pakietów, których zapomnieliśmy dodać lub zwyczajnie je przeoczyliśmy i
dlatego wymagane jest budowanie paczek w minimalnym środowisku chroot. Taka mała uwaga odnośnie
"bardzo podstawowych" zależności, np. `gcc` . Jeśli zdefiniujemy `gcc` w tutaj zależnościach, system
upomni się o sprecyzowanie konkretnej wersji takiego pakietu. Jeśli korzystamy z dodatkowych modułów
dla debhelper'a, tak jak w tym przypadku wykorzystywany jest `dh-systemd` , trzeba ten pakiet
również uwzględnić w zależnościach.

Zależności będą podane na stronie projektu ale czasem (albo i często) nazwy będą bardzo
nieprecyzyjne, np. `qt5` i ciężko będzie nam ustalić, o który pakiet tak naprawdę może chodzić.
Zwykle też do źródeł będzie dołączony plik `README` lub `INSTALL` z instrukcjami na temat budowy
programu. Przykład:

    REQUIREMENTS
    ===============================================================================
    Monitorix requires some others packages to be installed that your GNU/Linux
    distribution may or may not have:

    - Perl
    - Perl-CGI
    - Perl-libwww
    - Perl-MailTools
    - Perl-MIME-Lite
    - Perl-DBI
    - Perl-XML-Simple
    - Perl-Config-General
    - Perl-HTTP-Server-Simple
    - perl-IO-Socket-SSL
    - RRDtool and its Perl bindings (perl-rrdtool or rrdtool-perl)
    - (Optional) a CGI capable Web server (Apache, Nginx, lighttpd, etc.)

Z tym, że trzeba rozróżnić zależności potrzebne do budowania pakietu i zależności potrzebne do
poprawnego działania programu.

Jeśli dany pakiet posiada plik `configure` , możemy poszukać na podstawie tego pliku potrzebnych
zależności przy pomocy `dpkg-depcheck` . Z tym, że trzeba mieć na uwadze, że ten `dpkg-depcheck`
wyrzuca zależności potrzebne do wykonania się polecenia i niekoniecznie wszystkie pakiety, które nam
zwróci musimy wpisać jako zależności w budowanej paczce. Poza tym informacje o zależnościach są
czasem niezbyt precyzyjne, np. nie wiemy nic wersji danej zależności. Poniżej przykład:

    $ dpkg-depcheck -d ./configure
    checking build system type... x86_64-unknown-linux-gnu
    checking host system type... x86_64-unknown-linux-gnu
    ...
    ----------------------------------------------------------------------
    Packages needed:
      libc6-i386
      cpio
      perl
      ...

Zwykle będą nas interesować pakiety zaczynające się od `lib` , do których to będzie trzeba dodać
końcówkę `-dev` i taki pakiet dać w zależnościach. Trzeba także pamiętać, że część z powyższych
zależności może być pociągnięta przez jakiś metapakiet np. `build-essential` i możemy sobie darować
ich wpisywanie do pliku `debian/control` . Na dobrą sprawę, powyższe wyjście dobrze jest traktować
orientacyjne, a nie dosłownie.

Innym sposobem na ustalenie zależności może być posłużenie się `objdump` , z tym, że ten skanuje
pliki wykonywalne i by móc z niego skorzystać, musimy mieć uprzednio skompilowany program. W każdym
razie, jeśli już mamy w systemie jakiś pakiet, to zależności ustalamy tak:

    $ objdump -p /usr/sbin/dnscrypt-proxy | grep NEEDED
      NEEDED               libsodium.so.13
      NEEDED               libdl.so.2
      NEEDED               libm.so.6
      NEEDED               libc.so.6

Podobnie jak w przypadku `dpkg-depcheck` , obcinamy `.so.*` i dołączamy sufix `-dev` .

Następnie mamy `Standards-Version` i jest to wersja standardu polityki debiana jaka ma być
spełniona, aby zbudować tę paczkę i zawsze trzeba ustawiać tutaj najnowszą wersję, obecnie jest to
3.9.6 . Jeśli w późniejszym czasie wersja ulegnie zmianie, pakiet trzeba będzie przebudować i
prawdopodobnie będą potrzebne też jakieś mniejsze zmiany, by spełnić wymogi nowszej wersji
standardu.

Opcje `Homepage` , `Vcs-Git` oraz `Vcs-Browser` określają położenie projektu w sieci i czy korzysta
z jakiegoś systemu kontroli wersji (VCS). Jeśli projekt nie korzysta z VCS, możemy usunąć te linijki
z pliku.

Część opisową źródeł mamy z głowy. Druga część pliku dotyczy tego co zostanie zbudowane, czyli jaki
pakiet opuści plac budowy.

Linijka z `Package` określa nazwę pakietu, domyślnie taka sama jak nazwa źródła.

Kolejna linijka to `Architecture` i tu możemy wpisać `all` albo `any` . Różnica między tymi dwoma
tkwi w rodzaju budowanych plików. Jeśli budujemy skrypty, wtedy tak naprawdę nie budujemy ich tylko
kopiujemy pliki do paczki, a taki skrypt może być odpalony na każdej architekturze i w takim
przypadku wybieramy `all` . Z kolei pliki wymagające kompilacji trzeba budować dla każdej
architektury osobno i tutaj dajemy zwykle `any` , chyba, że pakiet ma wyraźnie zaznaczone na stronie
projektu, że przeznaczony jest tylko dla np. amd64 czy i386 . Wtedy podajemy wedle opisu. Taka mała
uwaga jeszcze. Przyjdzie nam budować pakiety, które będą się składać ze skryptów, obrazków, mp3 i
całego mnóstwa innych rzeczy, którym normalnie byśmy nadali arch `all` ale dodatkowo będą zawierać
jeden lub kilka plików binarnych i z tego powodu arch `all` odpada ale też nie możemy zbudować
takiego pakietu z arch `any` , bo `lintian` będzie się rzucał, że marnujemy miejsce na serwerze, bo
przecie z jego perspektywy, jeśli jakiś pakiet zawiera znaczne ilości plików niezależnych od
architektury, to nie możemy dawać arch `any` i musimy dać `all` i jak z takiej sytuacji wybrnąć?
Trzeba pakiet podzielić na dwie części. W jednej z nich dać pliki wymagające kompilacji, a w drugiej
(zwykle paczka-common) dać wszystkie pliki niezależne od architektury i dodać odpowiednie zależności
do paczki trzymającej plik binarny -- o tym będzie więcej przy okazji omawiania pliku `debian/rules`
.

Następne linijki to `Depends` , `Suggests` , `Recommends` , `Conflicts` , `Breaks` , `Provides` i
`Replaces` i są to zależności, które muszą być spełnione przy instalacji pakietu. Nie wszystkie z
wyżej wymienionych opcji będziemy zawsze używać. Zwykle ograniczymy się do pierwszych trzech.
Różnice między nimi są następujące. Jeśli pakiet wymaga do działania innego pakietu, dajemy go w
`Depends` . Jeśli nasz pakiet może się obyć bez innego pakietu ale traci na tym trochę ze swojej
funkcjonalności, to dajemy tamten pakiet w `Recommends` . Jeśli jakiś pakiet przydaje się ale można
bez niego żyć, to dopisujemy go do `Suggests` . Jeśli nasz pakiet dostarcza pliki dostępne w innych
pakietach, dajemy ten drugi pakiet w `Conflics`. Z kolei `Breaks` będziemy używać przy przenoszeniu
plików z jednego pakietu do innego, np. przy podziale pakietu na kilka mniejszych. Jeśli nasz pakiet
jest alternatywą dla innego pakietu, np. firefox jest alternatywą dla chrome (i vice versa), możemy
wpisać w polu `Provides` www-browser i wtedy każdy inny pakiet, który ma w swoich zależnościach
www-browser , przy instalacji będzie prosił o zainstalowanie firefoxa albo chrome (ew. obu). Jeśli
chodzi o `Replaces` , to zwykle jest używany on (w połączeniu z `Conflicts`) przy zastępowaniu
starych i nieaktualnych już pakietów, np. jeśli mamy do czynienia z forkami jakichś projektów i
projekt główny nie jest już rozwijany ale w repozytorium wciąż jest dostępna paczka z nim.
Generalnie nie możemy mieć zainstalowanych obu tych pakietów jednocześnie.

Każde z tych pól może zawierać określoną wersję danego pakietu. Wykorzystujemy do tego następujące
operatory: `<<` , `<=` , `=` , `>=` oraz `>>` . Jeśli pakiet ma być w określonej wersji, to dajemy
`=` . Jeśli wersja ma być mniejsza lub równa, to wtedy używamy `<=` . Z kolei jeśli większa lub
równa to `>=` . Jeśli zaś wersja ma być mniejsza, to korzystamy z `<<` , a jeśli większa, to `>>` .

Jeśli chodzi zaś o same zależności wymagane do prawidłowego działania pakietu, to
`${shlibs:Depends}` i `${misc:Depends}` automatycznie ustalą czego paczka potrzebuje w oparciu o
zależności potrzebne do zbudowania tego pakietu. Niemniej jednak, czasem (jak w tym przypadku)
trzeba sprecyzować kilka dodatkowych pakietów. Ponadto, jeśli budujemy paczkę ze skryptami perl'a,
trzeba dopisać `${perl:Depends}` , jeśli jest to python'owy skrypt, to dajemy `${python:Depends}`
lub `${python3:Depends}` w zależności od wersji python'a. Wszystkie te dodatkowe listy z
generowanymi pakietami są wpisywane do pliku `debian/control` w paczce wynikowej i możemy te
zależności dokładnie obejrzeć i ewentualnie oszacować czy czegoś brakuje. Więcej informacji na
temat określania zależności między pakietami, można znaleźć
[tutaj](https://www.debian.org/doc/debian-policy/ch-relationships.html). Wspomnieć też należy, że to
nie są jedyne opcje, które możemy wykorzystać przy podstawianiu zmiennych, więcej o pozostałych
można przeczytać w manie
[deb-substvars](http://manpages.ubuntu.com/manpages/wily/man5/deb-substvars.5.html) .

I ostatnia pozycja w tym pliku to `Description` , czyli opis pakietu. Ten po dwukropku zwykle jest
krótki i nie powinien przekraczać 65 znaków -- więcej zostanie obciętych. Jeśli zaś chodzi o dłuższy
opis, to wpisujemy go bezpośrednio pod tą linijką, z tym, że wcinamy tekst o jedną spację i długość
linijki nie może przekraczać 80 znaków. Każda nowa linia musi być wcięta. Jeśli chcemy zrobić odstęp
między paragrafami (pusta linia), to dajemy kropkę (również wciętą) i dalej nowy paragraf. Jeśli
chcemy skorzystać z list, czyli wypunktować różne rzeczy, wcinamy linijkę przy pomocy dwóch spacji i
dajemy myślnik albo gwiazdkę. Ja zwykle daję gwiazdki.

### debian/rules

Jak sama nazwa wskazuje, ten plik będzie zawierał szereg reguł, w oparciu o które pakiet zostanie
zbudowany. Generalnie rzecz biorac jest to plik `makefile` tylko nieco inny.

Budowanie pakietu odbywa się w etapach. Wywoływany jest pierwszy etap i dokonywane są zdefiniowane w
nim reguły, po czym następuje przejście do kolejnego etapu i aplikowane są reguły określone tutaj i
tak dalej, aż do samego końca.

Poniżej znajduje się minimalny plik `debian/rules` :

    #!/usr/bin/make -f

    export DH_VERBOSE = 1
    export DH_OPTIONS = -v

    %:
       dh $@

Dobrze jest wyeksportować te dwie dodatkowe zmienne `DH_VERBOSE` oraz `DH_OPTIONS` . Uwidocznią one
poszczególne etapy budowania pakietu i będziemy widzieć każde polecenie jakie będzie wykonywane,
dzięki czemu znacznie przyśpieszymy proces przyswajania sobie całej tej potrzebnej nam wiedzy z
zakresu budowania pakietów.

Dalej w pliku mamy `%:` i jest to wildcard oznaczający każdy target (etap) przy budowaniu pakietu, a
tych jest sporo. Poniżej jest zobrazowany proces budowania paczki `dnscrypt-proxy` :

     fakeroot debian/rules clean
    dh clean --with systemd,autotools-dev
       dh_testdir
       dh_auto_clean
       dh_clean

     debian/rules build
    dh build --with systemd,autotools-dev
       dh_testdir
       dh_auto_configure
       dh_auto_build
       dh_auto_test

     fakeroot debian/rules binary
    dh binary --with systemd,autotools-dev
       dh_testroot
       dh_prep
       dh_installdirs
       dh_auto_install
       dh_install
       dh_installdocs
       dh_installchangelogs
       dh_installexamples
       dh_installman
       dh_installcatalogs
       dh_installcron
       dh_installdebconf
       dh_installemacsen
       dh_installifupdown
       dh_installinfo
       dh_systemd_enable
       dh_installinit
       dh_systemd_start
       dh_installmenu
       dh_installmime
       dh_installmodules
       dh_installlogcheck
       dh_installlogrotate
       dh_installpam
       dh_installppp
       dh_installudev
       dh_installwm
       dh_installgsettings
       dh_bugfiles
       dh_ucf
       dh_lintian
       dh_gconf
       dh_icons
       dh_perl
       dh_usrlocal
       dh_link
       dh_installxfonts
       dh_compress
       dh_fixperms
       dh_strip
       dh_makeshlibs
       dh_shlibdeps
       dh_installdeb
       dh_gencontrol
       dh_md5sums
       dh_builddeb

W każdym z powyższych targetów może znajdować się szereg akcji, np:

    dh_installman
       man --recode UTF-8 ./dnscrypt\-proxy\.8 > dnscrypt\-proxy\.8\.new
       chmod 644 dnscrypt-proxy.8.new
       mv -f dnscrypt-proxy.8.new dnscrypt-proxy.8
       man --recode UTF-8 ./hostip\.8 > hostip\.8\.new
       chmod 644 hostip.8.new
       mv -f hostip.8.new hostip.8

Każdy z etapów wywołuje określone narzędzie, które ma na celu coś zrobić. Z reguły są to skrypty
zaczynające się od `dh_` i każdy z nich ma swój własny manual. Warto poczytać jeśli potrzebujemy
informacji na temat pewnych opcji pojawiających się w logu podczas budowania. Nie zawsze będziemy
korzystać ze wszystkich wyżej wylistowanych targetów. To zależeć będzie od tego co tak naprawdę
będziemy budować. Jeśli budujemy pakiet mający pliki unitów dla systemd, to logiczne jest, że
będziemy przechodzić przez etap `dh_systemd_enable` i `dh_systemd_start` . Warto też wiedzieć, że
szereg z powyższych targetów ma swoje pliki konfiguracyjne w katalogu `debian/` , np. targetowi
`dh_install` przypisany jest plik `*.install` .

Wszelkie moduły, z których chcemy skorzystać przy budowie paczki, precyzujemy po `dh $@` . Poniżej
przykład skorzystania z modułu `dh_systemd` :

    ...
    %:
       dh $@ --with dh_systemd
    ...

Jeśli modułów było by więcej, dodajemy je kolejno oddzielając je od siebie za pomocą przecinka.
Musimy także pamiętać o dodaniu odpowiednich zależności w pliku `debian/control` .

Jeśli chodzi o dalszą część pliku `debian/rules`, to nasuwa się pytanie -- skąd ja mam wiedzieć co
tutaj wpisać i jakie reguły zaaplikować? Zwykle `debhelper` wykryje upstream'owy plik `makefile` i
powinien sobie z procesem budowania poradzić bez naszej ingerencji ale czasem szereg plików trzeba
będzie usunąć, a niektóre stworzyć lub przenieść w inne miejsce, tak by paczka miała ręce i nogi i
nie zawierała przy tym zbędnych śmieci.

Jeśli kiedyś zdarzyło nam się instalować pakiet ręcznie, tj. via `./configure`
(dh_auto_configure) , `make` (dh_auto_build) i `make install` (dh_auto_install), to może nam to
podsunąć kilka pomysłów, bo tak na dobrą sprawę, to jak sama nazwa wskazuje `makefile` tworzy pliki.
Zatem jest to prosta instrukcja gdzie wgrać jakie pliki, by program działał jak trza. Zatem możemy
podejrzeć plik `makefile` dołączony do źródeł i prześledzić co tak naprawdę w nim się odbywa. Z
reguły pliki `makefile` są długie i nie będę tutaj przytaczał całości. Rzucimy jedynie okiem na
najważniejsze jego fragmenty:

    PN = monitorix

    PREFIX ?= /usr
    CONFDIR = /etc
    BASEDIR = /var/lib/monitorix/www
    LIBDIR = /var/lib/monitorix
    INITDIR_SYSTEMD = $(PREFIX)/lib/systemd/system
    INITDIR_OTHER = $(CONFDIR)/init.d
    BINDIR = $(PREFIX)/bin
    DOCDIR = $(PREFIX)/share/doc/$(PN)
    MAN5DIR = $(PREFIX)/share/man/man5
    MAN8DIR = $(PREFIX)/share/man/man8

    RM = rm -f
    RMD = rmdir
    SED = sed
    INSTALL = install -p
    INSTALL_PROGRAM = $(INSTALL) -m755
    INSTALL_SCRIPT = $(INSTALL) -m755
    INSTALL_DATA = $(INSTALL) -m644
    INSTALL_DIR = $(INSTALL) -d

Powyżej mamy nagłówek pliku `makefile` monitorix'a. Jak widać jest ustawianych szereg zmiennych.
Głównie precyzowane są katalogi, do których powędrują pliki, jak i same polecenia instalujące
pliki w tych katalogach.

Dalej mamy już zwykle operacje na plikach, np:

    install-bin:
        $(Q)echo -e '\033[1;32mInstalling script and modules...\033[0m'
        $(INSTALL_DIR) "$(DESTDIR)$(BINDIR)"
        $(INSTALL_PROGRAM) $(PN) "$(DESTDIR)$(BINDIR)/$(PN)"

        $(INSTALL_DIR) "$(DESTDIR)$(BASEDIR)/cgi"
        $(INSTALL_DIR) "$(DESTDIR)$(BASEDIR)/imgs"
        $(INSTALL_PROGRAM) $(PN).cgi "$(DESTDIR)$(BASEDIR)/cgi/$(PN).cgi"
        $(INSTALL_DATA) logo_bot.png "$(DESTDIR)$(BASEDIR)/logo_bot.png"
        $(INSTALL_DATA) logo_top.png "$(DESTDIR)$(BASEDIR)/logo_top.png"
        $(INSTALL_DATA) monitorixico.png "$(DESTDIR)$(BASEDIR)/monitorixico.png"

        $(INSTALL_DIR) "$(DESTDIR)$(CONFDIR)/$(PN)"
        $(INSTALL_DATA) $(PN).conf "$(DESTDIR)$(CONFDIR)/$(PN)/$(PN).conf"
    ...

I jak widać niczym się to zbytnio nie różni o zwykłego Ctrl-C z jednego miejsca i Ctrl-V w inne
miejsce. Zatem mamy już mniej więcej obraz tego co tak naprawdę się będzie działo podczas budowania
pakietu. `debhelper` przetworzy ten plik `makefile` i zainstaluje poszczególne pliki w
zdefiniowanych lokalizacjach, a potem z tego zrobi paczkę. Z tym, że trzeba wziąć pod uwagę jedną
rzecz. W tym przypadku mamy do czynienia ze skryptem perl'a i tutaj nie odbywa się żadna kompilacja.
Niemniej jednak, po kompilacji są tworzone pliki wynikowe i to w dużej mierze właśnie te pliki będą
upychane w odpowiednich katalogach.

Jeśli nie chce nam się zbytnio brodzić w pliku `makefile` , to z reguły informacje na temat
budowania pakietu będą podane w pliku `README` albo `INSTALL` , które są dołączone do paczki ze
źródłami. Przykładowo:

    INSTALLATION
    ===============================================================================
    Running a 'make install-xxx' as root will distribute the files to the file
    system. Most users will want to select from three options depending on target
    init system (do not run all four)!

       # make install-systemd-all
       # make install-upstart-all
       # make install-debian-all
       # make install-redhat-all

    Alternatively, users may override any of the in make targets.  For example:

       $ make DESTDIR=~/pub BASEDIR=/srv/http/monitorix install-systemd-all
    ...

I tu już wiemy dokładnie czego szukać w pliku `makefile` i jak zainstalować ten programik. Nam
potrzebne są 2 (max 3) z tych 4 wyżej wymienionych rzeczy. W końcu budujemy paczkę dla debiana,
zatem plik `debian/rules` powinien przybrać poniższą postać:

    #!/usr/bin/make -f

    export DH_VERBOSE = 1

    %:
       dh $@

    override_dh_auto_install:
       dh_auto_install -- \
          install-systemd-all \
          install-upstart-all \
          install-debian-all

Normalnie proces budowy przechodząc przez poszczególne etapy dotrze do targetu `dh_auto_install` ,
który wykona instrukcje zawarte w pliku `makefile` , czyli zainstaluje wszystko co tam jest podane,
a tego nie zawsze chcemy. Dlatego też, mamy możliwość nadpisywania szeregu etapów budowania przez
dopisanie `override_*` . Powyżej nadpisaliśmy etap automatycznej instalacji przekazując do polecenia
`make` trzy parametry, uzyskując w ten sposób dokładnie taką samą sekwencję poleceń (za wyjątkiem
jednego), która była określona w pliku `README` .

Kluczowe w tej zwrotce z `dh_auto_install` jest zrozumienie po co na końcu tego polecenia dodawane
są `--` , bo można także podać parametry bez tych dwóch myślników. Jaka jest zatem różnica? Jeśli
wywołujemy jakiś target z `--` , to wszystkie parametry znajdujące się za tymi myślnikami są
dołączane do wynikowego polecenia. W przypadku, gdy nie podamy tych myślników, system zresetuje
wszystkie poprzednie opcje i zamiast nich użyje te, które zostaną podane bezpośrednio za wywoływanym
targetem.

Weźmy sobie przykład skryptu `configure` . Domyślnie ma on określone, dajmy na to, takie parametry
`--prefix=/usr --localstatedir=/var` . Jeśli chcielibyśmy dodać kolejny parametr, korzystamy z
myślników:

    dh_auto_configure -- --sysconfdir=/etc

Jeśli chcielibyśmy użyć kompletnie innych wartości dla skryptu configure, pomijamy `--` ,
przykładowo:

    dh_auto_configure --prefix=/usr/local --localstatedir=/usr/local/var --sysconfdir=/usr/local/etc

Mamy jeszcze kilka innych opcji tyczących się nadpisywania targetów. Jeśli, przykładowo,
odpowiadałby nam etap instalacji ale chcielibyśmy coś dodać do niego (przed albo po), wtedy musimy
wywołać jego target i dodać swoje polecenia przed lub po nim, przykładowo:

    ...
    override_dh_auto_install:
       dh_auto_install -- install install-gui
       rm debian/tmp/usr/share/doc/ansifilter/ChangeLog
       rm debian/tmp/usr/share/doc/ansifilter/COPYING
       rm debian/tmp/usr/share/doc/ansifilter/INSTALL
       rm debian/tmp/usr/share/man/man1/ansifilter.1.gz
    ...

Powyżej został wywołany `make` z dwoma parametrami `install` oraz `install-gui` , poza tym, część
plików automatycznie zainstalowanych, zostanie usunięta, tak by nie trafiła do wynikowego pakietu.

Jeśli natomiast chcielibyśmy nadpisać jakiś etap całkowicie, tak by nie wykonała się w nim żadna
akcja, to możemy to zrobić precyzując jedynie sam target + override, przykładowo:

    override_dh_auto_install:

A jeśli pasowałoby nam jedynie nadpisać standardowe akcje, wtedy nie wywołujemy `dh_auto_install` ,
tylko od razu precyzujemy własne linijki, przykładowo:

    ...
    override_dh_auto_install:
       install -d -m 755 debian/napi/usr/share/napi/
       install -d -m 755 debian/napi/usr/bin/
       install -m755 ./napi.sh debian/napi/usr/bin/
       install -m755 ./subotage.sh debian/napi/usr/bin/
       install -m755 ./napi_common.sh debian/napi/usr/share/napi/
    ...

Jeśli instalujemy jakieś pliki, ścieżki do nich podajemy względem głównego katalogu ze źródłami. Z
kolei jeśli chcemy jakiś plik usunąć, to zwykle podajemy ścieżki debian/tmp/ lub
debian/nazwa_pakietu/ + odpowiednia ścieżka w drzewie katalogu, np.
`debian/tmp/usr/share/doc/ansifilter/INSTALL` . Jeśli nie usunęlibyśmy tego pliku, w paczce
znalazłby się on pod `/usr/share/doc/ansifilter/INSTALL` .

### debian/changelog i debian/upstream.changelog

Każdy projekt musi mieć listę wprowadzanych zmian, a te z kolei dzielimy na takie, które sami
wprowadzamy lub też zostały uwzględnione w upstream'ie. Generalnie rzecz biorąc, większość projektów
utrzymuje swój plik changelog (z reguły zlokalizowany w głównym katalogu źródeł) i nie musimy się o
niego martwić. Niemniej jednak, spotkamy się z takimi projektami, które nie mają pliku changeloga,
lub w ogóle nie posiadają żadnej historii zmian za wyjątkiem kolejnego numerku wersji.

Jeśli jest to przypadek pierwszy, czyli projekt ma changelog, np. na swojej stronie, ale nie
dostarcza pliku, w którym zmiany by były uwzględnione, możemy taki plik zrobić i do niego skopiować
zawartość strony www. Plik taki nazywamy `upstream.changelog` . Format tego pliku jest chyba
dowolny, np. może wyglądać tak:

    Version 2.2
    -------------
     * Fix problems with small displays like 80×25
     * Fix ATA compliance recognition (see bug #57)
     * Replace “Press any key” with “Press ‘m’ for menu”
     * Rework signal handling during procedure for stability
     * Improve build system

    Version 2.1
    -------------
     * Enhanced smart, smart_noreverse copying strategies;
     * Introduced new copying strategies skipfail, skipfail_noreverse;
     * Introduce journaling of copying and automatic resuming of interrupted copying;
     * Support procedure parameters overriding by ~/.whddrc config file;
     * Several bug fixes.
    ...

Trzeba tylko poinformować debhelper'a by przepisał nazwę pliku, pod którą ten changelog będzie
widoczny w wynikowej paczce, przykładowo:

    override_dh_installchangelogs:
       dh_installchangelogs -k debian/upstream.changelog

Powyższy kod wpisujemy oczywiście do pliku `debian/rules` i stworzy on linka `changelog` do pliku
`upstream.changelog` . Więcej informacji na temat `dh_installchangelogs` można znaleźć w
[manie](http://manpages.ubuntu.com/manpages/wily/en/man1/dh_installchangelogs.1.html).

Jeśli chodzi zaś o te zmiany, które sami wprowadzamy, to dopisujemy je do pliku `debian/changelog` ,
który na początku ma poniższą postać:

    monitorix (3.6.0-1) unstable; urgency=low

      * Initial release (Closes: #nnnn)

     -- Mikhail Morfikov <morfik@nsa.com>  Thu, 19 Feb 2015 14:00:01 +0100

Oczywiście nie edytujemy tego pliku ręcznie (choć można). Do jego obsługi mamy narzędzie `dch` , do
którego doklejamy tylko odpowiedni parametr i zwykle to będzie `-a` (dodanie nowej pozycji w
istniejącym już wpisie), `-i` (utworzenie nowego wpisu), `-r` (zaktualizowanie sygnatury czasowej),
oraz `-v` (zmiana wersji).

Przy zmianie wersji trzeba jednak uważać, bo zostanie zmieniona nazwa katalogu roboczego,
przykładowo:

    morfik:~/debian_build/monitorix-3.5.0$ dch -v 3.6.0
    dch warning: New package version is Debian native whilst previous version was not
    dch warning: your current directory has been renamed to:
    ../monitorix-3.6.0

    morfik:~/debian_build/monitorix-3.5.0$ cd ..

    morfik:~/debian_build$ cd monitorix-3.6.0/

    morfik:~/debian_build/monitorix-3.6.0$

W przypadku, gdy będziemy chcieli włączyć pakiet do repozytorium debiana, trzeba będzie założyć
stosowny wątek na bugtrackerze, któremu zostanie przypisany odpowiedni numer identyfikacyjny i to
ten numer będziemy musieli podać w pliku `debian/changelog` , np. `(Closes: #123456` . Ma to na celu
zautomatyzowanie procesu śledzenia i zamykania błędów, bo plik `debian/changelog` jest skanowany w
poszukiwaniu pewnych wyrażeń i np. BTS po dodaniu powyższego pakietu automatycznie zamknie zawarte w
nim kody błędów, oczywiście o ile wpiszemy poprawne numerki. To samo tyczy się zgłaszania kolejnych
błędów. Jeśli uda nam się je poprawić albo zostaną poprawione w upstream'ie, to wystarczy dopisać w
changelogu numerki zgłoszonych przez kogoś błędów i po przesłaniu paczki, system zamknie je
automatycznie. Bez tego automatu, sami musielibyśmy to robić.

Jeśli chcielibyśmy się bawić w maintainera z prawdziwego zdarzenia, trzeba będzie nam przyswoić
umiejętność operowania na listach mailingowych debiana. To tam będziemy mieć dostęp do bugtrackera,
gdzie będą zgłaszane i omawiane błędy w pakietach, którymi będziemy musieli się zająć. Debian ma
także kilka użytecznych narzędzi, które pomogą nam w tym zadaniu i jednym z nich jest
[reportbug](https://www.debian.org/Bugs/Reporting). Przydaje się ono nie tylko do zgłaszania błędów.
Zbiór list mailingowych debiana można znaleźć [tutaj](https://lists.debian.org/), z kolei [pod tym
linkiem](https://www.debian.org/MailingLists/) można znaleźć garść użytecznych informacji na temat
tego jak używać samych list.

Skrypt `dch` jest dość rozbudowany i ma sporo opcji. Jeśli ktoś chciałby w pełni zgłębić możliwości
tego narzędzia, to informacje znajdzie w
[manie](http://manpages.ubuntu.com/manpages/wily/en/man1/dch.1.html). Więcej informacji na temat
formatu samego pliku `debian/changelog` można znaleźć
[tutaj](https://www.debian.org/doc/debian-policy/ch-source.html#s-dpkgchangelog).

#### debian/NEWS

Wszystkie ważne informacje na temat zmian w pakiecie możemy wrzucić również do tego pliku. Podczas
instalacji nowszej wersji takiego pakietu, narzędzie takie jak `apt-listchanges` wyrzuci monit z
tekstem, który tam umieścimy. Więcej informacji na temat dokładnej zasady działania tego mechanizmu
można znaleźć w [manualu](http://manpages.ubuntu.com/manpages/wily/man1/apt-listchanges.1.html).

Format tego pliku jest podobny do formatu pliku `debian/changelog` , z tym, że nie zawiera gwiazdek.
Przy pomocy narzędzia `dpkg-parsechangelog` możemy sprawdzić poprawność tego pliku:

    $ dpkg-parsechangelog -ldebian/NEWS
    Source: ansifilter
    Version: 1.11-1
    Distribution: unstable
    Urgency: low
    Maintainer: Mikhail Morfikov <morfik@nsa.com>
    Date: Tue, 03 Feb 2015 09:19:10 +0100
    Changes:
     ansifilter (1.11-1) unstable; urgency=low
     .
       Some important news
       Some important news
       Some important news
       Some important news

### debian/copyright

Licencja jest chyba jedną z tych rzeczy na jaką debian kładzie największy nacisk. Samych licencji
mamy całe mnóstwo ale cześć z nich jest odrzucana przez politykę debiana i jeśli będziemy pakować
jakąś aplikację na jednej z takich licencji, ten pakiet nigdy nie trafi do głównego repozytorium
debiana.

Licencje akceptowalne przed debiana można odczytać z `dh_make` przy tworzeniu pakietu:

    $ dh_make --help
      -c, --copyright <type>    use <type> of license in copyright file
                                (apache|artistic|bsd|gpl|gpl2|gpl3|lgpl|lgpl2|
                                 lgpl3|mit)

Pliki licencji są zlokalizowane w katalogu `/usr/share/common-licenses/` i mamy tam do dyspozycji
licencje takie jak Apache-2.0, GPL, LGPL, czy BSD.

Nie musimy całej treści tych plików kopiować do `debian/copyright` . Możemy jedynie dać nagłówek i
zawrzeć odnośnik do jednego z powyższych plików. Tak jak to robi standardowo `dh_make` . Są też
licencje, które nie widnieją w powyższym katalogu i równocześnie są akceptowane przez debiana, np.
MIT, i w takim przypadku trzeba podać treść całej licencji w pliku `debian/copyright` .

Zwykle każdy plik ma w nagłówku u siebie licencje (przynajmniej powinien mieć) ale przeglądanie
dziesiątek czy setek plików w celu ustalenia, który jest na jakiej licencji może być lekko męczące.
Na szczęście mamy do dyspozycji narzędzie `licensecheck` , które wywołane z opcją `-r` w głównym
katalogu, zwróci nam listę plików wraz z ich licencją, przykładowo:

    $ licensecheck -r *
    docs/monitorix-alert.sh: *No copyright* UNKNOWN
    docs/htpasswd.pl: *No copyright* UNKNOWN
    lib/fail2ban.pm: GPL (v2 or later)
    lib/HTTPServer.pm: GPL (v2 or later)
    ...

Czasem nie wszystkie pliki zostaną uwzględnione w powyższym listingu, domyślnie będą to te pasujące
do następującego
    wzorca:

    '\.(c(c|pp|xx)?|h(h|pp|xx)?|f(77|90)?|go|p(l|m)|xs|sh|php|py(|x)|rb|java|js|vala|el|sc(i|e)|cs|pas|inc|dtd|xsl|mod|m|tex|mli?|(c|l)?hs)$'

Jeśli któryś z plików w naszym pakiecie się nie łapie pod ten wzorzec, możemy ręcznie określić
własny przy pomocy opcji `-c` .

O te pliki, które nie mają licencji, trzeba będzie się dopytać, chyba, że na stronie projektu jest
informacja, że "pliki są na GPL-2", wtedy wszystko jest już jasne.

Poniżej przykład pliku `debian/copyright` wygenerowanego przez dh_make :

    Format: http://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
    Upstream-Name: monitorix
    Source: http://www.monitorix.org

    Files: *
    Copyright: 2005-2014 Jordi Sanfeliu <jordi@fibranet.cat.>
    License: GPL-2.0+

    Files: debian/*
    Copyright: 2015 Mikhail Morfikov <morfik@nsa.com>
    License: GPL-2.0+

    License: GPL-2.0+
     This package is free software; you can redistribute it and/or modify
     it under the terms of the GNU General Public License as published by
     the Free Software Foundation; either version 2 of the License, or
     (at your option) any later version.
     .
     This package is distributed in the hope that it will be useful,
     but WITHOUT ANY WARRANTY; without even the implied warranty of
     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
     GNU General Public License for more details.
     .
     You should have received a copy of the GNU General Public License
     along with this program. If not, see <http://www.gnu.org/licenses/>
     .
     On Debian systems, the complete text of the GNU General
     Public License version 2 can be found in "/usr/share/common-licenses/GPL-2".

Jeśli chodzi o samą strukturę tego pliku, to w pierwszej linijce mamy `Format` i zwykle jest taki
jak wyżej. Uzupełniamy `Upstream-Name` i `Source` , odpowiednio, nazwę pakietu oraz skąd zostały
pobrane źródła.

Dalej mamy określanie praw do plików. Jeśli wszystkie pliki w źródłach pochodzą od jednej osoby,
wtedy w linijce z `Files` dajemy `*` . Jeśli autorów jest więcej i każdy z nich rości sobie prawa do
innego pliku lub plików w danym katalogu, musimy te pliki wypunktować oddzielając je spacją,
przykład:

    Files: src/libsodium/crypto_auth/hmacsha256/cp/hmac_hmacsha256.c
     src/libsodium/crypto_auth/hmacsha512256/cp/hmac_hmacsha512256.c
     src/libsodium/crypto_hash/sha256/cp/hash_sha256.c
     src/libsodium/crypto_hash/sha512/cp/hash_sha512.c
     src/libsodium/crypto_pwhash/scryptsalsa208sha256/pbkdf2-sha256.h
     src/libsodium/crypto_pwhash/scryptsalsa208sha256/pbkdf2-sha256.c
    Copyright: 2005,2007,2009 Colin Percival
    License: BSD-2-clause

Sekcji z `Files` może być dowolna ilość, tak by każdy plik miał określoną licencję. Jeśli któryś z
nich zostanie pominięty, `lintian` będzie miał z tym problem i nam to oświadczy. Z reguły to my
będziemy mieć prawa do plików w katalogu `debian/` , bo je tworzymy.

W linijce z `Copyright` podajemy informacje na temat tego kto jest właścicielem praw autorskich do
plików w formie "data nazwisko email" (email nie jest obowiązkowy). Jeśli autorów by było więcej,
wtedy definiujemy ich jeden pod drugim np:

    Files: src/libsodium/crypto_pwhash/scryptsalsa208sha256/crypto_scrypt.h
     src/libsodium/crypto_pwhash/scryptsalsa208sha256/nosse/pwhash_scryptsalsa208sha256_nosse.c
     src/libsodium/crypto_pwhash/scryptsalsa208sha256/sse/pwhash_scryptsalsa208sha256_sse.c
    Copyright: 2009 Colin Percival
     2012, 2013 Alexander Peslyak
    License: BSD-2-clause

Ostatnia rzecz, o której trzeba wspomnieć to `License` i bynajmniej nie chodzi tutaj o rodzaj
licencji tylko o sposób jej definiowania. Generalnie można to robić na dwa sposoby. Jeden z nich
zakłada dołączania pod linijką z `License` treści licencji ale to troszeczkę psuje całą estetykę i
lepiej jest tego nie robić. Alternatywne rozwiązanie zakłada zdefiniowanie poszczególnych bloków
plików jeden pod drugim i dopiero na końcu podanie treść licencji, przykładowo:

    Files: *
    Copyright: © 2015 William Jon McCann
               © 2015 Peter de Ridder
               © 2015 Simon Steinbeiß
    License: GPL-2

    Files: debian/*
    Copyright: © 2013 Yves-Alexis Perez
    License: GPL-2
    ...
    Files: data/*.1
    Copyright: 2007 Sven Arvidsson
      2014 Peter de Ridder
    License: GPL-2+

    License: GPL-2
     On Debian systems, the complete text of the GNU General Public License version
     2 can be found in '/usr/share/common-licenses/GPL-2'.

    License: GPL-2+
     On Debian systems, the complete text of the GNU General Public License can be
     found in '/usr/share/common-licenses/GPL-2'.

Formatowanie pliku odbywa się dokładnie na takich samych zasadach co w przypadku pliku
`debian/control` , czyli paragrafy (np. w licencjach) są oddzielone kropkami, a nowe linie są wcięte
minimum o jedną spację.

### debian/watch i debian/upstream/signing-key.asc

W oparciu o te pliki, narzędzie `uscan` potrafi zweryfikować czy pojawiła się nowsza wersja jakiejś
aplikacji. Plik `debian/watch` będzie wyglądał mniej więcej tak:

    version=3
    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/Monitorix-$1\.tar\.gz/ \
      https://github.com/mikaku/Monitorix/tags .*/v?(\d\S*)\.tar\.gz

Linijka z `version` obecnie przyjmuje wartość `3` . Natomiast kolejna linijka jest nieco
skomplikowana. Mamy tam wyrażenia regularne perl'a. Nie będziemy zajmować się tutaj samymi
wyrażeniami i skupimy się na wzorcach, które jedynie będziemy dostosowywać. Więcej informacji o
samych wyrażeniach regularnych można znaleźć [tutaj](http://perldoc.perl.org/perlre.html).

Większość projektów jest hostowana na githubie czy sourceforge i w ich przypadku możemy skorzystać z
czyjejś pracy, czyli skopiować sobie tę powyższą linijkę i odpowiednio dostosować. Powyżej mamy
przykład projektu hostowanego na githubie i raczej przerobienie linku nie powinno przysporzyć
problemów.

Wszystkie inne większe strony z projektami opensource mają swoje regułki zebrane [pod tym
linkiem](https://wiki.debian.org/debian/watch/). Jeśli tam nie znajdziemy strony, z której
pobraliśmy źródła, będziemy musieli sami naskrobać odpowiednią regułkę. Wprawdzie nie jest to
obowiązkowe ale znacząco ułatwia późniejsze aktualizowanie źródeł, bo wystarczy, że w katalogu ze
źródłami wydamy polecenie `uscan` i ten już sprawdzi czy pojawiła się nowsza wersja w upstream'ie.
Jeśli tak, pobierze ją i automatycznie dostosuje nazwy plików, co przyśpieszy migrację pakietu do
nowszej wersji.

Po uzupełnieniu pliku, aktualizację przeprowadzamy w poniższy sposób:

    $ uscan --verbose
    -- Scanning for watchfiles in .
    -- Found watchfile in ./debian
    -- In debian/watch, processing watchfile line:
       opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/Monitorix-$1\.tar\.gz/   https://github.com/mikaku/Monitorix/tags .*/v?(\d\S*)\.tar\.gz
    -- Found the following matching hrefs:
         /mikaku/Monitorix/archive/v3.6.0.tar.gz (3.6.0)
         /mikaku/Monitorix/archive/v3.5.1.tar.gz (3.5.1)
         /mikaku/Monitorix/archive/v3.5.0.tar.gz (3.5.0)
    Newest version on remote site is 3.6.0, local version is 3.6.0
     => Package is up to date
    -- Scan finished

Jak widzimy wyżej, plik `debian/watch` został odnaleziony i przetworzony, zwracając 3 pliki o
różnych wersjach. Jako, że posiadam najnowszą wersję źródeł, żadna akcja mająca na celu dokonanie
aktualizacji nie została podjęta.

#### Podpisy cyfrowe źródeł

Trzeba jeszcze wspomnieć o jeden ważnej rzeczy, mianowicie o podpisach cyfrowych. Jeśli budujemy
projekt, który ma podpisane źródła, koniecznie musimy weryfikować ich podpis, z tym, że będziemy
potrzebować klucza publicznego osoby składającej sygnaturę. Poniżej jest przedstawiony proces
pozyskiwania klucza publicznego.

Na sam początek odwiedzamy stronę projektu i patrzymy co za pliki są do pobrania. W tym przypadku
mamy min. `dnscrypt-proxy-1.4.3.tar.gz` oraz `dnscrypt-proxy-1.4.3.tar.gz.sig` . Pobieramy oba pliki
i przechodzimy do katalogu gdzie zostały pobrane. Próbujemy póki co ręcznie zweryfikować podpis:

    $ gpg --verify dnscrypt-proxy-1.4.3.tar.gz.sig
    gpg: assuming signed data in 'dnscrypt-proxy-1.4.3.tar.gz'
    gpg: Signature made Tue 10 Feb 2015 12:00:24 PM CET
    gpg:                using RSA key 0x62F25B592B6F76DA
    gpg: Can't check signature: public key not found

Jak widzimy, nie można zweryfikować sygnatury, bo nie posiadamy klucza publicznego. Szukamy go zatem
i importujemy:

    $ gpg --search-keys 0x62F25B592B6F76DA

Klucz został zaimportowany do naszego lokalnego keyring'a. Jeśli teraz byśmy spróbowali sprawdzić
sygnaturę, podpis powinien zostać zweryfikowany. Musimy teraz wydobyć ten klucz z keyring'a i
zapisać go do pliku w formacie ascii:

    $ mkdir debian/upstream/

    $ gpg --armor --export 0x62F25B592B6F76DA > debian/upstream/signing-key.asc

Zatem klucz publiczny jest już na swoim miejscu. Musimy jeszcze nieco dostosować linijkę w pliku
`debian/watch` , tak by `uscan` automatycznie weryfikował podpis bez naszej ingerencji:

    version=3
    opts=pgpsigurlmangle=s/$/.sig/ \
      http://download.dnscrypt.org/dnscrypt-proxy/dnscrypt-proxy-([0-9].+).tar.gz

Od tej pory, gdy się pojawi nowsza wersja pakietu, zostanie ona pobrana po wywołaniu polecenia
`uscan` , natomiast sygnatura zostanie zweryfikowana automatycznie.

### debian/manpage.* i debian/*.manpages (dh_installman)

Dokumentacja projektu to bardzo ważna rzecz i bez niej ani rusz. To właśnie na jej podstawie wiemy
jakie zadania ma realizować dany program. Zwykle jest nieco bardziej obszerna niż parametr `--help`
doczepiony do wywołanego programu. Jeśli dany projekt się szanuje, powinien wypuścić przyzwoitą
dokumentację i dołączyć ją paczki ze źródłami. Nie zawsze jednak taki manual ma odpowiednią formę.
[Pod tym linkiem](http://liw.fi/manpages/) znajduje się krótki tutorial na temat składni stosowanej
w plikach manuala wykorzystywanych w debianie. Będziemy musieli się tego nauczyć, w przeciwnym
wypadku, może się zdarzyć tak, że nasze paczki zostaną bez dokumentacji, a to niedobrze.

Z grubsza będziemy mieli styczność z trzema rodzajami przypadków. W pierwszym z nich nie będzie
żadnej dokumentacji. W drugim będzie dokumentacja ale trzeba będzie ją przeformatować. Zaś w
trzecim będziemy musieli poprawić błędy w formatowaniu, które wypunktuje nam `lintian` . Pomijam
oczywiście ten rzadki przypadek, w którym wszystko będzie jak trza i nie będziemy musieli nic robić.

Manuale tyczą się głównie plików wykonywalnych i według polityki debiana, każdy taki plik musi mieć
manual, nawet jeśli jest to jakaś aplikacja GUI z dwoma przyciskami na krzyż, z których jeden to OK,
a drugi to ANULUJ. Nazwy plików manuali mają również swój format np. `napi.1` . Numerki są od 1-9 ,
w zależności od tego o czym jest konkretna strona manuala. Najczęściej jednak będziemy się spotykać
z numerkami `1` (info o opcjach, które program może przyjąć) i `5` (pliki konfiguracyjne). Jeśli
kogoś interesują pozostałe numerki, może o nich poczytać w `man man` .

Bardzo rzadko będziemy tworzyć plik manuala od podstaw. Jeśli się jednak na to zdecydujemy, to
istnieją graficzne narzędzia, które mogą nam ułatwić to zadanie. Jednym z nich jest `gmanedit` . W
pozostałych przypadkach będziemy szli na skróty. Samo poprawianie poszczególnych linijek w pliku nie
powinno sprawić problemów, zwłaszcza jeśli się opanuje składnię pliku manuala. Możemy także
korzystać z polecenia `man -l plik-mana.1` aby podejrzeć jak wyglądać będzie ten manual w systemie.

#### Automatyczne generowanie manuali

Watro też wspomnieć iż istnieje możliwość wygenerowania pliku manuala w oparciu o polecenie
`--help` , z tym, że nie zawsze wszystkie aplikacje posiadają w swoim wyposażeniu ten parametr.
Poza tym, jeśli chodzi o aplikacje GUI, muszą one mieć dostęp do sesji graficznej, by taki manual
wygenerować. I takim automatycznym generowaniem plików manuali zajmuje się `help2man` . Można go
wywołać bezpośrednio przy budowaniu paczki, z tym, że to pociąga za sobą dodatkowe zależności oraz
kilka linijek w pliku `debian/rules` , a konkretnie trzeba będzie nadpisać target `dh_installman`
oraz uwzględnić ten nowy plik przy czyszczeniu w targecie `dh_clean`. Poniżej przykład:

    override_dh_installman:
        help2man -N --no-discard-stderr --version-string='0.3.3' -n 'feature-rich screen recorder that supports X11 and OpenGL' debian/simplescreenrecorder/usr/bin/simplescreenrecorder > simplescreenrecorder.1
        help2man -N --no-discard-stderr --version-string='0.3.3' -n 'inject the GLInject library into a given command' debian/simplescreenrecorder/usr/bin/ssr-glinject > ssr-glinject.1
        dh_installman simplescreenrecorder.1 ssr-glinject.1

    override_dh_clean:
        dh_clean -- simplescreenrecorder.1 ssr-glinject.1

Możemy także wygenerować plik manuala po zbudowaniu paczki i wrzucić go później do katalogu
`debian/` .

Jeśli zdecydujemy się na to drugie wyjście, to manual generujemy w poniższy sposób:

    $ which napi
    /usr/bin/napi

    $ help2man /usr/sbin/napi > napi.1

Tak wygenerowany plik dobrze jest przejrzeć i poprawić ewentualne błędy.

Mając już pliki manuali, musimy uwzględnić je w `debian/*.manpages` , gdzie gwiazdka zwykle
odpowiada nazwie pakietu, do którego ten manual ma trafić. Generalnie chodzi o to, że w przypadku,
gdy budujemy większy projekt, który ma kilka paczek, to raczej chcielibyśmy aby manual od jednego
pliku trafił do konkretnej paczki, podobnie z pozostałymi. Poniżej przykład pliku `napi.manpages` :

    debian/napi.1
    debian/subotage.1

Format tego pliku jest prosty. Każda linijka to jeden plik manuala i określamy w niej położenie
samego pliku w katalogu ze źródłami. Jako, że stworzyliśmy dopiero co nowe pliki manuala, to
umieściliśmy je w podkatalogu `debian/` . Nie używamy tutaj początkowego `/` , bo ścieżki są
względne.

Istnieje także możliwość zmiany położenia i wtedy po spacji dodajemy kolejną ścieżkę, która określa
lokalizację pliku w wynikowej paczce i też nie używamy początkowego `/` , przykładowo:

    debian/napi.1 usr/share/man/man1/

Z reguły nie ma takiej potrzeby i wystarczy określić tylko pierwszą ścieżkę, druga zostanie dobrana
automatycznie na podstawie numerka.

### debian/*.docs (dh_installdocs)

Ten plik będzie zawierał ścieżki do plików, które niosą ze sobą jakąś przydatną z punktu widzenia
projektu informację ale nie są to manuale. Zwykle ich nazwy będą pisane dużymi lietrami, np.
`README` . Ze wszystkich takich plików robimy listę. Trzeba tylko się pilnować, by nie umieszczać
śmieci, np. nie potrzebujemy pliku `INSTALL`, czy plików związanych z innymi dystrybucjami linux'a.
Pakujemy tu tylko co się tyczy debiana. Poniżej przykład pliku:

    AUTHORS
    BUGS
    COLABORATION
    README.md

Z reguły system powinien wykryć cześć plików ale dobrze jest przejrzeć główny katalog źródeł i
posprawdzać czy coś nie zostało pominięte.

### Pliki demonów (dh_installinit)

Nie będę tutaj przytaczał żadnych skryptów -- chodzi o skrypty sysvinit, unity systemd i podobne
pliki związane z odpalaniem usług systemowych, bo to jest poza zakresem tego poradnika. Niemniej
jednak, zostanie załączony link do dokumentacji poszczególnych plików, tak by mieć jakiś punkt
zaczepienia.

#### debian/*.init

Niby `debhelper` tworzy plik `*.init.d` ale wszędzie w paczkach spotkałem się tylko z samym
`*.init` . W każdym razie, nie ma to większego znaczenia, z której nazwy skorzystamy. Oba pliki
odpowiadają za instalowanie skryptów startowych sysvinit, tych w katalogu `/etc/init.d/` . By
sprawnie tworzyć takie skrypty, trzeba zrozumieć sam nagłówek
[LSB](https://wiki.debian.org/LSBInitScripts) oraz ogarnąć narzędzie
[start-stop-daemon](http://manpages.ubuntu.com/manpages/wily/man8/start-stop-daemon.8.html).
Dodatkowo trzeba również opanować obsługę `sh` ( `dash` ), tak by
[uniknąć](http://mywiki.wooledge.org/Bashism) [bashismów](https://wiki.ubuntu.com/DashAsBinSh).

Skrypty init dobrze jest także potraktować poleceniem `sh -n` , które zwróci ewentualne problemy ze
składnią, przykładowo:

    $ sh -n dnscrypt-proxy

Jeśli jakieś błędy się pojawią, trzeba będzie je poprawić.

#### debian/*default

Plik `*.default` odpowiada za konfigurację skryptu init trzymaną w katalogu `/etc/default/`.
Generalnie chodzi o możliwość zachowania części z ustawionych opcji, które zwykle powinny być stałe
i nie ulegać resetowaniu podczas aktualizacji pakietu. Sam skrypt init może ulec zmianie ale w
stopniu, w którym nie powinien ingerować w opcje określone tutaj w tym pliku, przynajmniej taka jest
teoria.

Ten plik zawiera jedynie szereg zmiennych i trochę komentarzy, a do tego raczej nie potrzebujemy
manuala.

#### debian/*service (dh_systemd_enable, dh_systemd_start)

W tym pliku jest trzymana konfiguracja unitów dla systemd. By z nich skorzystać musimy w pliku
`debian/rules` uwzględnić moduł `dh_systemd` . Dokładne informacje na temat opcji, które możemy użyć
w plikach unitów, możemy znaleźć w dokumentacji systemd dostępnej
[tutaj](https://www.freedesktop.org/software/systemd/man/systemd.index.html). Jest tego dość sporo i
dla ułatwienia podpowiem tylko, że interesować nas będą głównie strony od
[systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) oraz
[systemd.unit](https://www.freedesktop.org/software/systemd/man/systemd.unit.html).

#### debian/*tmpfile

W tym pliku jest trzymana konfiguracja plików tymczasowych, które są niezbędne do działania danego
programu. Wszystkie pliki i katalogi określone tutaj będą tworzone i konfigurowane automatycznie na
starcie systemu. Dokładne informacje na temat formatu samego pliku można znaleźć
[tutaj](https://www.freedesktop.org/software/systemd/man/tmpfiles.d.html).

### debian/*.symbols

Z tym plikiem spotkamy się jedynie przy budowaniu bibliotek, co nie jest prostym zadaniem.
Zakładając, że udało nam się zbudować bibliotekę, będziemy musieli wygenerować dla niej plik
`*.symbols` , a tego możemy dokonać jedynie mając do dyspozycji paczkę wynikową. Także zbudowanie
biblioteki wymaga przepakowania jej co najmniej jeden dodatkowy raz.

Do generowania symboli posłuży nam narzędzie `dpkg-gensymbols` , zaś do wypakowania paczki
`dpkg-deb` . Sam proces przebiega mniej więcej w poniższy sposób:

    $ dpkg-deb -x /media/Kabi/pbuilder/result/libssr-glinject_0.3.3-1_amd64.deb /tmp/libssr-glinject
    $ dpkg-gensymbols -v0.3.3 -plibssr-glinject -P/tmp/libssr-glinject -Olibssr-glinject.symbols
    $ sed -i s/0\.3\.3$/0\.3\.3\~/g libssr-glinject.symbols

Pierwsza linijka wypakowuje paczkę z biblioteką, druga generuje symbole, a trzecia dopisuje do
wersji znaczek `~` , o który domagał się `lintian` . Tak stworzony plik przenosimy do katalogu
`debian/` i budujemy paczkę jeszcze raz.

Czasem też się zdarzy tak, że nazwa, którą nadaliśmy paczce z biblioteką, jest niewłaściwa. Jak
zatem ustalić odpowiednią nazwę? Do tego celu posłuży nam narzędzie `readelf` , poniżej przykład:

    $ readelf -d  /usr/lib/x86_64-linux-gnu/libssr-glinject.so | grep SONAME
     0x000000000000000e (SONAME)             Library soname: [libssr-glinject.so]

I już wiemy, że paczka ma się nazywać `libssr-glinject` . Więcej informacji na temat symboli można
znaleźć [tutaj](https://www.debian.org/doc/manuals/maint-guide/advanced.en.html#librarysymbols) i
[tutaj](https://www.debian.org/doc/debian-policy/ch-sharedlibs.html#s-sharedlibs-symbols).

### debian/*.dirs (dh_installdirs)

Jeśli do zbudowania pakietu potrzebne nam są dodatkowe katalogi, które standardowo nie są tworzone,
oznaczać to może najprawdopodobniej błąd w pliku `makefile` . Możemy te katalogi wpisać do tego
pliku i poprawić tym samym ten błąd. Jeśli dodajemy pliki do `debian/*.install` , to nie musimy
tutaj uwzględniać tych katalogów.

### debian/*.install (dh_install)

Jeśli z jakiegoś powodu pewne pliki nie są brane pod uwagę przez `makefile` i zwyczajnie nie zostaną
utworzone w paczce wynikowej, musimy je tutaj określić.

Ten plik będziemy także wykorzystywać do rozdzielania plików na mniejsze paczki ale tutaj taka mała
uwaga. Pierw musimy zbudować pakiet bez jakiegokolwiek rozdzielania na mniejsze i zobaczyć jakie
pliki uzyskamy w paczce wynikowej. Jeśli na samym początku zaczniemy bawić wpisami w `*.install` ,
jest wysokie prawdopodobieństwo, że pogubimy część potrzebnych plików, bo nie zostaną one w tych
plikach `*.install` uwzględnione.

Jeśli z jednego źródła chcemy zbudować dwie paczki, potrzebne będą nam dwa pliki `.install` . Każdy
z nich będzie miał nazwę wynikowego pakietu. Poniżej jest przykład budowania pakietu
`simplescreenrecorder` i biblioteki `libssr-glinject` . Oba są budowane z tych samych źródeł. By
odseparować pewne pliki od siebie i upchnąć je w osobnych paczkach, tworzymy poniższe pliki:

Plik debian/simplescreenrecorder.install :

    usr/share/
    usr/bin/

Plik debian/libssr-glinject.install :

    usr/lib/*/libssr-glinject.so

W powyższym przykładzie jest podana tylko jedna ścieżka. Druga może być podana po spacji. Pierwsza z
nich określa skąd kopiować plik, druga gdzie go wrzucić (w drzewie katalogu pakietu). Jeśli nie
podamy tej drugiej ścieżki, zostanie przyjęta domyślnie i będzie taka sama jak pierwsza. Co do
samych plików, to możemy precyzować każdy z osobna uwzględniając przy tym nazwę pliku w pierwszej
ścieżce albo też możemy podawać ścieżki do całych katalogów, tak jak to widać wyżej. Trzeba
pamiętać o tym, że przy pomocy `*.install` nie damy rady przepisać nazwy pliku, a druga ścieżka
zawsze ma postać katalogu.

Jeśli chodzi o same biblioteki, to przy instalowaniu plików trzeba uważać, bo w systemie są różne
ścieżki w zależności od jego architektury. Zamiast podawać dokładną ścieżkę, można użyć wildcard'a,
tak by dopasować pewną jej części i sprawić tym samym, że pakiet zbuduje się na innej architekturze
niż był pierwotnie budowany. Zwykle będzie to wyglądać tak: `usr/lib/*/` , kluczowa jest tutaj ta
gwiazdka w trzecim członie i to ona upchnie plik biblioteki w odpowiednim folderze (normalnie byśmy
tam podali np. x86_64-linux-gnu).

Po rozdzieleniu paczek, trzeba będzie także odpowiednio uzupełnić plik `debian/control` tak by dodać
opisy i zależności, przykładowo:

    ...
    Package: simplescreenrecorder
    Architecture: i386 amd64
    Pre-Depends: ${misc:Pre-Depends}
    Depends: ${shlibs:Depends}, ${misc:Depends}
    Recommends: libssr-glinject
    Description: Feature-rich screen recorder (main program)
     SimpleScreenRecorder is a feature-rich screen recorder that supports X11 and
     OpenGL. It has a Qt-based graphical user interface. It can record the entire
     creen or part of it, or record OpenGL applications directly. The recording
     can be paused and resumed at any time. Many different file formats and
     codecs are supported
     .
     This package contains the main program.

    Package: libssr-glinject
    Architecture: i386 amd64
    Multi-Arch: same
    Pre-Depends: ${misc:Pre-Depends}
    Depends: ${shlibs:Depends}, ${misc:Depends}, simplescreenrecorder
    Description: Feature-rich screen recorder (GLInject library)
     SimpleScreenRecorder is a feature-rich screen recorder that supports X11 and
     OpenGL. It has a Qt-based graphical user interface. It can record the entire
     screen or part of it, or record OpenGL applications directly. The recording
     can be paused and resumed at any time. Many different file formats and
     codecs are supported
     .
     This package contains the GLInject library.

### debian/compat

Ten plik określa poziom kompatybilności debhelper'a i obecnie ma zawierać cyfrę `9` . Za co
odpowiada ten poziom kompatybilności, to chyba nikt nie wie

### debian/*.links (dh_link)

Ten plik ma na celu tworzenie linków do plików, których nie chcemy bezpośrednio kopiować, bo, jakby
nie patrzeć, to zajmuje cenne zasoby dyskowe, poza tym mniej danych trzeba przesłać przez sieć.
Poniżej jest przykład takiego linku:

    usr/share/man/man1/ansifilter.1.gz usr/share/man/man1/ansifilter-gui.1.gz

Ta linijka stworzy link `ansifilter-gui.1.gz` do pliku `/usr/share/man/man1/ansifilter.1.gz` w
wynikowym pakiecie.

### debian/*.lintian-overrides i debian/source/lintian-overrides

Te dwa pliki są wykorzystywane przez narzędzie `lintian` , które po przeskanowaniu paczek/źródeł
wyrzuca komunikaty w przypadku, gdy coś odbiega od ustalonego standardu. Niemniej jednak, czasem się
zdarza tak, że `lintian` będzie miał buga, albo też jego błąd tak naprawdę nie będzie żadnym błędem
i będzie można cały komunikat zakwalifikować jako false-positive.

W obu przypadkach mamy możliwość ukrycia tych komunikatów, tak by nas już więcej nie niepokoiły.
Jeśli problem dotyczy źródeł, będziemy wykorzystywać plik `debian/source/lintian-overrides` .
Natomiast jeśli `lintian` ma problem z jakąś paczką wynikową, wtedy trzeba będzie stworzyć osobny
plik z nazwą paczki i sufixem `.lintian-overrides` .

Źródła powinny zostać zweryfikowane, a to zwykle odbywa się przez sprawdzenie sygnatury GPG. Nie
każdy serwis daje nam taką możliwość i nie mamy tak naprawdę żadnego pola manewru w sytuacji, gdy
twórca źródeł nie podpisał swojego dzieła cyfrowo. Jeśli napotkamy podobny problem, to jako, że
komunikat dotyczy źródeł pakietu, wrzucamy poniższą linijkę do pliku
`debian/source/lintian-overrides` :

    light-locker source: debian-watch-may-check-gpg-signature

Spora część komunikatów będzie jednak dotyczyć paczek wynikowych. Poniżej jest przykład ukrycia
błędu w paczce `libssr-glinject` . W katalogu `debian/` tworzymy plik
`libssr-glinject.lintian-overrides` i wrzucamy do niego np. coś takiego:

    libssr-glinject: shlib-without-versioned-soname usr/lib/*/libssr-glinject.so libssr-glinject.so
    libssr-glinject: pkg-has-shlibs-control-file-but-no-actual-shared-libs

By stworzyć takie pliki musimy znać komunikat błędu, nazwę paczki oraz nazwę pliku, którego błąd
dotyczy. Wszystkie te informacje wypisze nam `lintian` po przeskanowaniu paczek. Poniżej jest
przykład:

    W: libssr-glinject: shlib-without-versioned-soname usr/lib/x86_64-linux-gnu/libssr-glinject.so libssr-glinject.so
    E: libssr-glinject: pkg-has-shlibs-control-file-but-no-actual-shared-libs

Jak widać, praktycznie niczym się te linijki z błędami nie różnią od tego co wpisaliśmy wyżej w
pliku, no za wyjątkiem początkowych `W:` i `E:` . Struktura obu powyższych plików jest taka sama i
dokładnie opisana jest [tutaj](https://lintian.debian.org/manual/section-2.4.html) .

### source/format

Ten plik określa format źródeł pakietu i z reguły wpisujemy w nim `3.0 (native)` dla natywnych
debianowych pakietów, oraz `3.0 (quilt)` dla wszystkiego innego i zwykle będziemy budować właśnie
ten drugi rodzaj, gdzie wszystkie zmiany dokonywane przez nas na źródłach będą rejestrowane i
zapisywane w plikach w katalogu `debian/patches/` (o nim później). Takie zmiany po wypakowaniu
źródeł będą automatycznie nakładane przez `dpkg-source` .

### source/local-options i source/options

W tych plikach możemy zdefiniować opcje dla narzędzia `dpkg-source` wywoływanego z parametrami `-b`
lub `--print-format` . Użyteczne mogą się okazać `--compression` czy `--compression-level` , z tym,
że dodajemy je do pliku bez tych dwóch myślników na początku, przykładowo:

    compression = "bzip2"
    compression-level = 9

Jedyną różnicą między tymi dwoma plikami jest to, że `source/local-options` nie będzie dołączany w
zbudowanych źródłach. Wszystkie dostępne opcje dla `dpkg-source` są przystępnie opisane w
[manualu](http://manpages.ubuntu.com/manpages/wily/en/man1/dpkg-source.1.html).

### debian/patches/ i debian/patches/series

Jako, że nie możemy bezpośrednio zmieniać upstream'owych źródeł, musimy robić łatki, by przy ich
pomocy nanieść pożądane zmiany. Jeśli zmienimy pewnie pliki w źródłach, po czym spróbujemy zbudować
taki pakiet, zostanie nam wypisany poniższy komunikat:

    ...
    dpkg-source -b monitorix-3.6.0
    dpkg-source: info: using source format '3.0 (quilt)'
    dpkg-source: info: building monitorix using existing ./monitorix_3.6.0.orig.tar.gz
    dpkg-source: info: local changes detected, the modified files are:
     monitorix-3.6.0/monitorix
    dpkg-source: error: aborting due to unexpected upstream changes, see /tmp/monitorix_3.6.0-1.diff.6zxg54
    dpkg-source: info: you can integrate the local changes with dpkg-source --commit
    dpkg-buildpackage: error: dpkg-source -b monitorix-3.6.0 gave error exit status 2
    debuild: fatal error at line 1376:
    dpkg-buildpackage -rfakeroot -d -us -uc -S -sa failed

By zbudować taki pakiet musimy stworzyć łatę przy pomocy `dpkg-source --commit` .

Jest też jeszcze inne narzędzie, które ułatwia zarządzanie uprzednio stworzonymi patch'ami i jest to
`quilt` . Pomoże nam on nie tylko w zakładaniu/ściąganiu łat ale również w ich edycji.

Musimy wiedzieć jak działa `quilt` , by sprawnie operować na łatach. Przede wszystkim, mamy do
dyspozycji plik `debian/patches/series` i to w nim są zawarte informacje na temat tego jakie patch'e
i w jakiej kolejności mają być założone na źródła. Poniżej przykładowy plik:

    disable-update-check
    #proper-tempfiles
    assure-quit-keybinding
    fix_desktop_file.patch

W katalogu `debian/patches/` mamy zaś:

    morfik:~/debian_build/minitube-2.3.1$ ls -al debian/patches/
    total 28K
    drwxr-xr-x 2 morfik morfik 4.0K 2015-02-12 20:13:12 ./
    drwxr-xr-x 4 morfik morfik 4.0K 2015-02-15 04:55:45 ../
    -rw-r--r-- 1 morfik morfik  968 2014-09-05 23:02:33 assure-quit-keybinding
    -rw-r--r-- 1 morfik morfik  427 2014-09-05 23:02:33 disable-update-check
    -rw-r--r-- 1 morfik morfik  396 2015-02-12 20:13:12 fix_desktop_file.patch
    -rw-r--r-- 1 morfik morfik 2.5K 2014-09-05 23:02:33 proper-tempfiles
    -rw-r--r-- 1 morfik morfik   85 2015-02-12 19:34:19 series

Patch'e są zakładane w kolejności od góry do dołu z perspektywy pliku `debian/patches/series` .
Zatem pierw zostanie założony patch `disable-update-check` , potem by został założony
`proper-tempfiles` ale, że jest wykomentowany, to zostanie pominięty, następnie jest zakładany
`assure-quit-keybinding` i tak dalej, aż do końca pliku.

By założyć kolejny patch, korzystamy z `quilt push`, by ściągnąć obecny patch, dajemy `quilt pop` .
Parametr `-a` dopisany do obu z powyższych poleceń, odpowiednio założy i ściągnie wszystkie łaty.
Jeśli przez przypadek usunęliśmy pliki z patch'ami ale przy tym uprzednio ich nie ściągnęliśmy,
będziemy musieli posłużyć się opcją `-f` . By sprawdzić, jaki patch jest aktualnie założony,
wydajemy `quilt applied` . Możemy także przeprowadzać różne operacje na samych łatach, np. zmieniać
ich nazwy. Dobrze jest rzucić okiem na [manual
quilt'a](http://manpages.ubuntu.com/manpages/wily/man1/quilt.1.html) , gdzie znajdziemy opis
wszystkich przydatnych funkcji. Można również zajrzeć pod [ten
adres](https://raphaelhertzog.com/2012/08/08/how-to-use-quilt-to-manage-patches-in-debian-packages/),
gdzie znajdziemy opis praktycznego zastosowania tego narzędzia.

#### Nagłówek łaty

Z bardziej przydatnych rzeczy musimy jeszcze wiedzieć co nieco o edycji nagłówków plików łat. Każdy
patch musi bowiem spełniać [standard DEP-3](http://dep.debian.net/deps/dep3/), który definiuje
szereg pół wykorzystywanych w nagłówkach. Nagłówek wygenerowany przez `dpkg-source --commit` jest
dość rozbudowany. Jeśli nie chce nam się wypełniać wszystkich linijek albo zwyczajnie nie jesteśmy
w posiadaniu informacji, które moglibyśmy tam umieścić, to możemy ograniczyć się do dwóch
obowiązkowych pól, czyli `Description:` oraz `Author` (opis i autor). Resztę natomiast można bez
problemu skasować.

Spotkamy się także z sytuacjami, gdzie ktoś stworzy lub podeśle patch ale nie poda w nim żadnego
nagłówka lub informacje w nim będą błędne. Być może nawet sami zapomnimy zmienić powyższy nagłówek,
lub zwyczajnie go przeoczymy i nie uwzględnimy w łacie. W takich przypadkach możemy oczywiście
dokonać edycji nagłówka, z tym, że dobrze jest to zrobić via `quilt header --dep3 -e` . Spowoduje to
utworzenie domyślnego nagłówka, co ułatwi nam nieco robotę.

### debian/*.examples (dh_installexamples)

Pozycje w tym pliku będą instalowane pod `/usr/share/docs/nazwa_paczki/examples/` i generalnie
będziemy tutaj umieszczać pliki zawierające przykłady, np. konfiguracji.

### debian/*.templates

Ten plik jest używany do interakcji z użytkownikiem przy instalacji pakietu i odbywa się to przez
zadawanie pytań, na które trzeba udzielić odpowiedzi i w oparciu o nie można skonfigurować pakiet.
Generalnie rzecz biorąc nie spotkałem się z tym plikiem i za bardzo nic na jego temat nie napiszę.
Za to, [pod tym linkiem](https://www.leaseweb.com/labs/2013/06/creating-custom-debian-packages/)
jest przykład jego wykorzystania.

### debian/conffiles (dh_installdeb)

Debian automatycznie oznacza wszystkie pliki w katalogu `/etc/` jako pliki konfiguracyjne i ich
domyślnie ani nie usuwa przy pozbywaniu się pakietu z systemu ani też nie nadpisuje ich przy
instalacji/aktualizacji pakietu. Jeśli się przyjrzymy procesowi budowania paczki, dostrzeżemy tam
coś takiego:

    ...
       dh_installdeb
    ...
            find debian/monitorix/etc -type f -printf '/etc/%P' | LC_ALL=C sort >> debian/monitorix/DEBIAN/conffiles
            chmod 644 debian/monitorix/DEBIAN/conffiles
    ...

Dlatego też, nie musimy tworzyć pliku `debian/conffiles` , by powpisywać tam wszystkie pliki
konfiguracyjne z katalogu `/etc/` . Natomiast, jeśli zajrzymy do paczki wynikowej, zobaczymy tam, że
został utworzy plik `debian/connfiles` i jest tam lista plików, np:

    /etc/init.d/monitorix
    /etc/logrotate.d/monitorix
    /etc/monitorix/conf.d/debian.conf
    /etc/monitorix/monitorix.conf
    /etc/sysconfig/monitorix

By usunąć te pliki trzeba korzystać z opcji `purge` (np. przy `aptitude`). Z kolei jeśli chodzi o
nowsze wersje tych plików, to sprawdzane są sumy kontrolne i jeśli się różnią, wtedy dostajemy
stosowne powiadomienie i podejmujemy akcję dotyczącą ewentualnego nadpisania tego pliku, choć i tak
stary (ew. nowy) plik będzie backup'owany. Jeśli sumy są takie same, pliki nadpisywane są bez
jakiegokolwiek powiadamiania nas o tym, no bo w sumie i tak nie dokonywaliśmy żadnych zmian w tych
plikach.

Plik `debian/conffiles` znajduje jedynie zastosowanie w przypadku, gdy konfiguracja budowanego
pakietu znajduje się poza katalogiem `/etc/` , co zwykle nie powinno mieć miejsca.

### debian/*.cron.* (dh_installcron)

Czasami zdarzy się tak, że pewna funkcjonalność dostarczana wraz z budowanym pakietem będzie wymagać
okresowego wywoływania jakichś operacji. Możemy to osiągnąć przez stworzenie jednego z pięciu (ew.
kilku) plików: `debian/*.cron.hourly` , `debian/*.cron.daily` , `debian/*.cron.weekly` ,
`debian/*.cron.monthly` , `debian/*.cron.d` . Każdy z tych plików powędruje w określone miejsce w
systemie, odpowiednio będzie to katalog `/etc/cron.hourly/` , `/etc/cron.daily/` ,
`/etc/cron.monthly/` , `/etc/cron.weekly/` oraz `/etc/cron.d/` . Do pierwszych czterech wrzuca się
skrypty, zaś do ostatniego plik, który musi ma być w formacie określonym przez
[crontab](http://manpages.ubuntu.com/manpages/wily/en/man5/crontab.5.html) .

### debian/*.logrotate (dh_installlogrotate)

Jeśli nasza paczka będzie zawierać demony, które będą logować zdarzenia, dobrze jest także zadbać o
rotację logów, tak by jeden plik nie rozrastał się w nieskończoność. Wszystkie pliki określone w
`debian/*.logrotate` powędrują do `/etc/logrotate.d/` , zaś sam format tych plików określony jest w
[manie](http://manpages.ubuntu.com/manpages/wily/en/man8/logrotate.8.html).

### debian/*.postinst, debian/*.postrm, debian/*.preinst, debian/*.prerm

Te cztery pliki to skrypty maintainera i nie biorą one udziału przy budowaniu pakietu. Za to,
potrafią robić pewne rzeczy przy instalowaniu, usuwaniu i aktualizowaniu paczki. To właśnie tutaj
dodajemy, np. nowego użytkownika, czy nadajemy uprawnienia pewnym plikom. Zwykle nie będziemy
musieli tworzyć tych skryptów. Jeśli jednak zdecydujemy się na nie, to musimy bardzo uważać, by
czasem nie skasować sobie połowy systemu.

Skrypt `preinst` jest wywoływany przed rozpakowaniem paczki, z kolei skrypt `postinst` po jej
rozpakowaniu. Podobnie z dwoma pozostałymi skryptami. `prerm` jest wywoływany przed usunięciem
pakietu, a `postrm` po usunięciu.

Dokładny proces instalowania/deinstalowania/aktualizowania pakietu jest opisany
[tutaj](https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html). Są tam wyszczególnione
wszystkie akcje podejmowane podczas powyższych czynności. Z kolei zaś, [pod tym
linkiem](https://wiki.debian.org/ConfigPackages) można znaleźć przykłady samych skryptów. Dobrze
jest też zajrzeć w katalog `debian/` innych pakietów, by podejrzeć jak inni maintainerzy tworzą te
skrypty.

### *.desktop

Ten plik nie jest bezpośrednio związany z katalogiem `debian/` ale jest bardzo użyteczny w przypadku
aplikacji graficznych, bo na jego podstawie mogą zostać utworzone skróty w menu, w które można
kliknąć myszą. Format tego pliku jest opisany
[tutaj](https://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html#entries).
Warto wiedzieć, że istnieje także narzędzie, które może pomóc nam w weryfikacji składni tego pliku.
Jest to `desktop-file-validate` (pakiet `desktop-file-utils` ).

## Hardening pakietów

Jeśli nasz pakiet zawierać będzie pliki wykonywalne, trzeba będzie je skompilować i trzeba będzie to
zrobić z pewnymi określonymi flagami. Większość projektów ma już ustawione pewne flagi domyślne.
Czasem może się zdarzyć tak, że akurat to są te flagi, których my potrzebujemy. Niemniej jednak, mi
się przytrafił taki projekt, którego nie szło z początku niczym ruszyć. W przypadku jednej z paczek
trzeba było edytować plik `makefile` , z kolei w przypadku drugiej trzeba było odprawić większe
czary ale o tym za moment.

Na sam początek prześledźmy część pliku `makefile`:

    ...
    ###### Compiler, tools and options

    CC            = gcc
    CXX           = g++
    DEFINES       = -DO2 -DQT_NO_DEBUG -DQT_GUI_LIB -DQT_CORE_LIB -DQT_SHARED
    CFLAGS        = -pipe -march=x86-64 -mtune=generic -O2 -pipe -fstack-protector --param=ssp-buffer-size=4 -Wall -W -D_REENTRANT $(DEFINES)
    CXXFLAGS      = -pipe -march=x86-64 -mtune=generic -O2 -pipe -fstack-protector --param=ssp-buffer-size=4 -Wall -W -D_REENTRANT $(DEFINES)
    INCPATH       = -I/usr/share/qt4/mkspecs/linux-g++ -I. -I/usr/include/qt4/QtCore -I/usr/include/qt4/QtGui -I/usr/include/qt4 -I. -I.. -I. -I.
    LINK          = g++
    LFLAGS        = -Wl,-O1,--sort-common,--as-needed,-z,relro -Wl,-O1
    LIBS          = $(SUBLIBS)  -L/usr/lib -lQtGui -lQtCore -lpthread
    AR            = ar cqs
    RANLIB        =
    QMAKE         = /usr/bin/qmake-qt4
    TAR           = tar -cf
    COMPRESS      = gzip -9f
    COPY          = cp -f
    SED           = sed
    COPY_FILE     = $(COPY)
    COPY_DIR      = $(COPY) -r
    STRIP         = strip
    INSTALL_FILE  = install -m 644 -p
    INSTALL_DIR   = $(COPY_DIR)
    INSTALL_PROGRAM = install -m 755 -p
    DEL_FILE      = rm -f
    SYMLINK       = ln -f -s
    DEL_DIR       = rmdir
    MOVE          = mv -f
    CHK_DIR_EXISTS= test -d
    MKDIR         = mkdir -p
    ...

Mamy tam takie linijki jak `CFLAGS` , `CXXFLAGS` oraz `LFLAGS` . To właśnie te pozycje musimy
zmienić. To jakie flagi musimy ustawić, możemy odnaleźć na wiki debiana, do poczytania
[tutaj](https://wiki.debian.org/HardeningWalkthrough),
[tutaj](https://wiki.debian.org/ReleaseGoals/SecurityHardeningBuildFlags) i
[tutaj](https://wiki.debian.org/Hardening) . Nie zagłębiając się w szczegóły, zwykle wystarczy
dopisać na początku pliku `debian/rules` , te poniższe linijki:

    export DEB_BUILD_MAINT_OPTIONS = hardening=+all
    DPKG_EXPORT_BUILDFLAGS = 1
    include /usr/share/dpkg/buildflags.mk

Po zbudowaniu paczki, możemy sprawdzić czy powyższe rozwiązanie zadziałało i wykorzystujemy do tego
celu polecenie `hardening-check` (pakiet `hardening-includes` ), przykładowo:

    $ hardening-check  /usr/bin/ansifilter-gui
    /usr/bin/ansifilter-gui:
     Position Independent Executable: no, normal executable!
     Stack protected: no, not found!
     Fortify Source functions: no, only unprotected functions found!
     Read-only relocations: no, not found!
     Immediate binding: no, not found!

Jeśli widzimy log jak powyżej, oznacza to, że flagi nie zostały poprawnie ustawione i trzeba
kombinować jak je poprawić. Najłatwiejszym sposobem jest oczywiście edycja flag w plikach
źródłowych, z tym, że nie zawsze to działa, np. projekty, które są budowane przy pomocy `qmake`
generują sobie plik makefile i co z tego, że przepiszemy flagi, jak podczas budowania, ten plik
zostanie nadpisany. Na [wiki debiana](https://wiki.debian.org/Hardening#dpkg-buildflags) jest kilka
przykładów opisujących min. `qmake` czy `cmake` i możemy z nich skorzystać. W tym przypadku (
`qmake` ) trzeba było edytować plik `.pro` i ustawić w
    nim:

    QMAKE_CPPFLAGS *= -g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro -Wl,-z,now
    QMAKE_CFLAGS   *= -g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro -Wl,-z,now
    QMAKE_CXXFLAGS *= -g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro -Wl,-z,now
    QMAKE_LFLAGS   *= -g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -fPIE -pie -Wl,-z,relro -Wl,-z,now

W każdej linijce zostały zawarte flagi, które zwracał `dpkg-buildflags` przy ustawieniu
`hardening=+all`, przykładowo:

    dpkg-buildflags --status
    dpkg-buildflags: status: environment variable DEB_BUILD_OPTIONS=parallel=2
    dpkg-buildflags: status: environment variable DEB_HOST_ARCH=amd64
    dpkg-buildflags: status: vendor is Debian
    dpkg-buildflags: status: hardening features: bindnow=no format=yes fortify=yes pie=no relro=yes stackprotector=yes stackprotectorstrong=yes
    dpkg-buildflags: status: qa features: bug=no canary=no
    dpkg-buildflags: status: reproducible features: timeless=no
    dpkg-buildflags: status: CFLAGS [vendor]: -g -O2 -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: CPPFLAGS [vendor]: -D_FORTIFY_SOURCE=2
    dpkg-buildflags: status: CXXFLAGS [vendor]: -g -O2 -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: FCFLAGS [vendor]: -g -O2 -fstack-protector-strong
    dpkg-buildflags: status: FFLAGS [vendor]: -g -O2 -fstack-protector-strong
    dpkg-buildflags: status: GCJFLAGS [vendor]: -g -O2 -fstack-protector-strong
    dpkg-buildflags: status: LDFLAGS [vendor]: -Wl,-z,relro
    dpkg-buildflags: status: OBJCFLAGS [vendor]: -g -O2 -fstack-protector-strong -Wformat -Werror=format-security
    dpkg-buildflags: status: OBJCXXFLAGS [vendor]: -g -O2 -fstack-protector-strong -Wformat -Werror=format-security

Sprawdzając plik wykonywalny przy pomocy `hardening-check` , log zmienił się nieco:

    $ hardening-check  /usr/bin/ansifilter-gui
    /usr/bin/ansifilter-gui:
     Position Independent Executable: yes
     Stack protected: yes
     Fortify Source functions: yes (some protected functions found)
     Read-only relocations: yes
     Immediate binding: yes

I dokładnie o takie coś nam chodzi.

Jeśli mamy problem z ustaleniem jakich flag brakuje, pomocne może okazać się narzędzie `blhc` .
Wszystko czego nam potrzeba by z niego skorzystać, to log z budowania pakietu. Jeśli dysponujemy
takowym, brakujące flagi sprawdzamy w poniższy sposób:

    $ blhc --color --build build_log
    W-dpkg-buildflags-missing|CPPFLAGS 16 (of 32) missing|

    $ blhc --color build_log
    CXXFLAGS missing (-g): g++ -c -O2 -fstack-protector-strong -Wformat -Werror=format-security arg_parser.cpp -o arg_parser.o
    CPPFLAGS missing (-D_FORTIFY_SOURCE=2): g++ -c -O2 -fstack-protector-strong -Wformat -Werror=format-security arg_parser.cpp -o arg_parser.o
    CXXFLAGS missing (-g): g++ -c -O2 -fstack-protector-strong -Wformat -Werror=format-security stringtools.cpp -o stringtools.o
    ...

Jak widzimy wyżej, kilku obiektom brakuje flagi `-g` oraz `-D_FORTIFY_SOURCE=2` . Trzeba mieć też na
uwadze fakt, że przez taki hardening można nieco popsuć funkcjonalność pakietu.

## Budowanie źródeł

Jeśli już dostosowaliśmy wszystkie pliki w katalogu `debian/` , przyszedł czas na zbudowanie źródeł.
W tym celu przechodzimy do głównego katalogu ze źródłami i wydajemy poniższe polecenie:

    morfik:~/debian_build/monitorix-3.6.0$ debuild -S -sa -d

Skrypt `dbuild` , w zależności od konfiguracji, zainicjuje `dpkg-buildpackage` oraz kilka
dodatkowych narzędzi Podczas całego procesu, wszystkie łaty, które założyliśmy, zostaną
automatycznie ściągnięte ( `quilt` ), a sam katalog `debian/` zostanie oddzielony od źródeł i
upchnięty w osobnej paczce ( `dpkg-source` ). Takie rozwiązanie ma na celu zmniejszenie ruchu
sieciowego repozytorium, no bo jeśli aktualizujemy paczkę, to wystarczy pobrać tylko katalog
`debian/` i nie ma potrzeby przy tym pobierać samych źródeł z repozytorium, które czasami mogą ważyć
nawet i dziesiątki czy setki MiB. Same źródła zostaną sprawdzone pod kątem ewentualnych błędów (
`lintian` ). Zostaną także wygenerowane trzy dodatkowe pliki: `.build` , `.dsc` oraz `changes` . Po
całym procesie pliki `.dsc` i `.changes` zostaną podpisane cyfrowo ( `gpg` ).

W pliku `.build` będzie umieszczony log z operacji budowania źródeł, czyli dokładnie to samo co
zostanie wydrukowane w terminalu. Plik `.dsc` zawiera min. sumy kontrole spakowanych źródeł i
katalogu `debian` . Na tym etapie, plik `.changes` zbytnio się nie różni od pliku `.dsc` ale to w
nim będą zawarte sumy kontrole wszystkich wyprodukowanych przez nas paczek `.deb` . Jest tam również
kilka dodatkowych informacji, takich jak np. numery błędów, które ten pakiet ma zamykać.

## Budowanie pakietu

Po zbudowaniu źródeł, przyszedł czas na zbudowanie pakietu. W tym celu przechodzimy do katalogu
nadrzędnego (tam gdzie jest plik `.dsc` ) i zaprzęgamy pbuilder'a do roboty:

    $ sudo pbuilder --build ./monitorix_3.6.0-1.dsc

W tej chwili `pbuilder` wypakowuje przygotowanego wcześniej chroot'a. Po wypakowaniu są sprawdzane i
instalowane zależności. Po tym etapie, pliki źródłowe są kopiowane do środowiska chroot, po czym są
one wypakowywane. Jeśli robiliśmy jakieś łaty, to zostaną one następnie automatycznie założone na
źródła. Po tym procesie rozpoczyna się właściwa faza budowania pakietu. Dalej już tylko są
wywoływane poszczególne targety debhelper'a (linijki z `dh_` ) i jeśli proces zakończy się bez
błędów, zbudowane w ten sposób paczki wędrują do katalogu docelowego. Jest także generowany plik
`.changes` . Po zakończeniu, środowisko chroot jest usuwane. Paczki powinny się znaleźć w tym
katalogu, który określiliśmy w konfiguracji pbuilder'a.

W pliku `.changes` powinny być teraz uwzględnione paczki `.deb` :

    ...
    Checksums-Sha1:
    ...
     aed14e43b9604dc0bdfdc54ea05df91edb33d50b 156398 monitorix_3.6.0-1_amd64.deb
    Checksums-Sha256:
    ...
     951ddd2ace33cac27cd1c02ec695ffce9cd5e6e2d6758676851003e72969ae83 156398 monitorix_3.6.0-1_amd64.deb
    Files:
    ...
     adede69324183ba2248c093263e06934 156398 net optional monitorix_3.6.0-1_amd64.deb

Niemniej jednak, ten plik `.changes` nie został jeszcze podpisany. Musimy go podpisać ręcznie przy
pomocy tego poniższego polecenia:

    $ debsign /media/Kabi/pbuilder/result/monitorix_3.6.0-1_amd64.changes

Została nam jeszcze ostania rzecz, czyli zweryfikowanie czy stworzona przez nas paczka zawiera
jakieś błędy. Wołamy zatem lintian'a by sprawdził wszystkie paczki, które znajdzie w wygenerowanym
przed chwilą pliku `.changes` :

    $ rm -R /media/Kabi/pbuilder/lintian/*

    $ lintian --setup-lab

    $ lintian /media/Kabi/pbuilder/result/monitorix_3.6.0-1_amd64.changes

Jeśli chodzi o samego lintian'a jeszcze, to dobrze jest usunąć poprzednie katalogi robocze, bo
śmieci tam zgromadzone, mogą wpłynąć na wyniki skanowania.

W tym przypadku nie ma żadnych błędów ani ostrzeżeń, zatem paczka została zbudowana pomyślnie i
pewnie spełnia wszystkie standardy debiana, co otwiera drogę do umieszczenia jej w oficjalnym
repozytorium dystrybucji.

### Manualne budowanie pakietów

Prawdopodobnie zdarzy się tak, że z jakiegoś powodu utkniemy i trzeba będzie przetestować szereg
rzeczy, tak by doprowadzić proces budowania pakietu do końca. Jeśli będziemy z każdą drobną poprawką
wywoływać pbuilder'a, to nas szlag trafi w oczekiwaniu na zainstalowanie wszystkich tych zależności
w środowisku chroot. Nie chcemy też sobie zaśmiecać systemu tymi zbędnymi pakietami używanymi
jedynie przy budowaniu paczki. Mamy do wyboru dwie opcje, z których jedna została opisana już na
początku. Druga opcja to skorzystanie z hook'a, który jest dostępny w katalogu
`/usr/share/doc/pbuilder/examples/` . Mowa o `C10shell` . Umożliwia on przerwanie operacji przy
ewentualnych błędach podczas budowania paczki i zrzucenie nas do shell'a wewnątrz środowiska chroot.
Standardowo `pbuilder` by przerwał akcję i posprzątał po sobie. Linkujemy ten hook:

    $ ln -s /usr/share/doc/pbuilder/examples/C10shell /media/Kabi/pbuilder/hooks/

Od tego momentu, jeśli `pbuilder` napotka błąd, doinstaluje w chroot min. edytor `vim` i da nam
szansę sprawdzić czemu pakiet się nie zbudował.

Musimy umieć się poruszać wewnątrz środowiska chroot. Przede wszystkim, znajdziemy się w katalogu ze
źródłami (tam gdzie jest katalog `debian/` ), z tym, że nasz katalog roboczy zostanie zmieniony
nieco:

    root@morfikownia:~/monitorix-3.6.0# pwd
    /tmp/buildd/monitorix-3.6.0

Sama edycja plików nie powinna przysporzyć problemów, jedynie co, to trzeba opanować vim'a, choć
można też i to obejść ale o tym za moment. Jeśli problemy będą tkwić w ścieżkach plików, zawsze
możemy porównać je z tym co wypisał nam `debhelper` . Przykładowo:

    ...
    dh_install --
       cp -a ./docs/debian.conf debian/monitorix/etc/monitorix/conf.d/
    ...

Powyższy wycinek z logu mówi, że plik `debian.conf` z folderu `./docs/` (wszystko odbywa się
względem naszego katalogu roboczego), czyli w sumie jest to
`/tmp/buildd/monitorix-3.6.0/docs/debian.conf` ma zostać przekopiowany do
`debian/monitorix/etc/monitorix/conf.d/` . Zajrzyjmy zatem do tego tajemniczego katalogu
`debian/monitorix/` :

    root@morfikownia:~/monitorix-3.6.0# ls -al debian/monitorix
    total 24
    drwxr-xr-x 6 morfik morfik 4096 Feb 22 12:50 .
    drwxr-xr-x 5 morfik morfik 4096 Feb 22 12:50 ..
    drwxr-xr-x 6 morfik morfik 4096 Feb 22 12:50 etc
    drwxr-xr-x 3 morfik morfik 4096 Feb 22 12:50 lib
    drwxr-xr-x 5 morfik morfik 4096 Feb 22 12:50 usr
    drwxr-xr-x 3 morfik morfik 4096 Feb 22 12:50 var

Widzimy tutaj, że jest to część struktury folderów, która przypomina tę z naszego systemu
operacyjnego. To tutaj właśnie się buduje cały projekt, czyli w odpowiednie miejsca są kopiowane
dostarczane z projektem pliki. Jeśli z jakiegoś powodu coś nie gra, to tu zaczynamy poszukiwania,
oczywiście podpierając się logiem debhelper'a.

Gdy już ustalimy w czym tkwił problem, budujemy testowo paczkę `.deb` , z tym, że korzystamy już
bezpośrednio z narzędzia `dpkg-buildpackage` :

    root@morfikownia:~/monitorix-3.6.0# dpkg-buildpackage -uc -us -b
    ...

Jeśli paczka się zbuduje, znaczy, że możemy opuścić ten chroot.

Powyższe środowisko to chroot, zatem mamy dostęp do jego plików z maszyny hosta, wystarczy znaleźć
miejsce gdzie się ulokował ten chroot. Miejsce gdzie zaczynamy poszukiwania jest określone w pliku
konfiguracyjnym pbuilder'a. Jako, że środowiska są wypakowywane za każdym razem, tworzone są foldery
z numerkami pid'ów procesu pbuilder'a, przykładowo `/media/Kabi/pbuilder/build/69865/` . Jeśli
chcielibyśmy w trybie graficznym edytować, np. plik `debian/rules` , to używamy ścieżki
`/media/Kabi/pbuilder/build/69865/tmp/buildd/monitorix-3.6.0/debian/rules` .

### Zgubione pliki (dh_missing)

Budując pojedynczą paczkę nie ma w zasadzie obaw, że jakieś pliki nam się gdzieś zawieruszą, bo
wszystkie pliki wynikowe są pakowane do tej jednej paczki. Niemniej jednak, w przypadku bardziej
zaawansowanych projektów, z jednego źródła będziemy w stanie wyprodukować kilka paczek. W takim
przypadku trzeba będzie rozdzielić pliki. Może się zatem zdarzyć tak, że przez naszą nieuwagę pewne
pliki nie trafią na swoje miejsce lub zostaną pominięte i zwyczajnie nieuwzględnione w żadnym
pakiecie, który zostanie zbudowany. By tego typu sytuacjom przeciwdziałać dobrze jest w pliku reguł
`debian/rules` zawrzeć poniższe przepisanie targetu `dh_missing` :

    override_dh_missing:
	    dh_missing --list-missing --fail-missing

W tym powyższym nadpisaniu mamy dwa dodatkowe parametry, z których `--list-missing` wypisuje
wszystkie pliki, które nie zostaną uwzględnione w żadnym wynikowym pakiecie. Jeśli dodatkowo
zostanie podany `--fail-missing` , to proces budowy pakietu zostanie przerwany jeśli taki plik
zostanie odnotowany.

### Instalowanie zbudowanego pakietu

Jeśli mamy obawy co do tego jak zachowa się paczka podczas jej instalacji w systemie, możemy
skorzystać z narzędzia `piuparts` , które w środowisku chroot przetestuje wszystkie kombinacje
instalacji/deinstalacji i aktualizacji pakietu i zwróci obszerny log na temat modyfikowanych plików.
Na końcu zostanie także przedstawione podsumowanie. Poniżej przykładowe sprawdzenie paczki
`ansifilter-gui_1.11-1_amd64.deb` w środowisku chroot upchniętym w `base.tgz`
    .

    # piuparts /media/Kabi/pbuilder/result/ansifilter-gui_1.11-1_amd64.deb -b /media/Kabi/pbuilder/base.tgz

## Aktualizowanie pakietu

Aktualizacja pakietu `.deb`, który jest dostępny w debianie i ma zdebianizowane źródła, sprowadza
się do pobrania tych źródeł przy pomocy `apt` . Trzeba pamiętać by wpis z `deb-src` był dodany do
pliku `/etc/apt/sources.list` . Źródła pobieramy w poniższy sposób:

    $ apt-get source wpasupplicant
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    Picking 'wpa' as source package instead of 'wpasupplicant'
    Selected version '2.2-1' (testing) for wpa
    NOTICE: 'wpa' packaging is maintained in the 'Svn' version control system at:
    svn://anonscm.debian.org/pkg-wpa/wpa/trunk/
    Need to get 1,801 kB of source archives.
    Get:1 http://ftp.pl.debian.org/debian/ sid/main wpa 2.2-1 (dsc) [2,483 B]
    Get:2 http://ftp.pl.debian.org/debian/ sid/main wpa 2.2-1 (tar) [1,725 kB]
    Get:3 http://ftp.pl.debian.org/debian/ sid/main wpa 2.2-1 (diff) [74.3 kB]
    Fetched 1,801 kB in 1s (1,458 kB/s)
    gpgv: Signature made Wed 17 Sep 2014 09:53:09 PM CEST using RSA key ID C2B35520
    gpgv: Can't check signature: public key not found
    dpkg-source: warning: failed to verify signature on ./wpa_2.2-1.dsc
    dpkg-source: info: extracting wpa in wpa-2.2
    dpkg-source: info: unpacking wpa_2.2.orig.tar.xz
    dpkg-source: info: unpacking wpa_2.2-1.debian.tar.xz
    dpkg-source: info: applying 01_use_pkg-config_for_pcsc-lite_module.patch
    dpkg-source: info: applying 02_dbus_group_policy.patch
    dpkg-source: info: applying 06_wpa_gui_menu_exec_path.patch
    dpkg-source: info: applying 07_dbus_service_syslog.patch
    dpkg-source: info: applying 12_wpa_gui_knotify_support.patch
    dpkg-source: info: applying wpa_gui_desktop_add-keywords-entry.patch
    dpkg-source: info: applying wpa_supplicant-MACsec-fix-build-failure-for-IEEE8021.patch
    dpkg-source: info: applying ap_config_c_fix-typo-for-capabilities.patch

Paczki budowane ze źródeł nie zawsze mają nazwy takie same jak ich źródła ale to nie stanowi
problemu dla `apt` i ten sobie odszuka odpowiednie pakiety, tak jak nam to wyżej oznajmił: `Picking
'wpa' as source package instead of 'wpasupplicant'` . Dalej w logu widzimy, że z repozytorium
zostały pobrane trzy pliki: `.dsc` , `.tar` oraz `.diff` .

Plik `.dsc` zawiera szereg informacji opisujących pliki, źródła i ludzi, którzy opiekują się daną
paczką. Są tam też zawarte linki, pod którymi można znaleźć dany projekt. Dodatkowo, znajdują się
tam też informacje na temat sum kontrolnych źródeł oraz katalogu `debian/` . Całość jest podpisana
cyfrowo, co gwarantuje integralność danych, które właśnie pobraliśmy. Jeśli uważnie przejrzeliśmy
log pobierania źródeł, możemy tam zauważyć, że `apt` nie może zweryfikować sygnatury. By
zweryfikować podpis, potrzebny nam jest klucz publiczny osoby, która podpisała powyższy plik.
Pobranie klucza i weryfikację podpisu możemy przeprowadzić ręcznie przy pomocy poniższych linijek:

    $ gpg --verify wpa_2.2-1.dsc
    $ gpg --search-keys 0xFF914AF0C2B35520
    $ gpg --verify wpa_2.2-1.dsc

Zwykle plik zostanie poprawnie zweryfikowany ale jako, że nie mamy określonego zaufania do osoby,
której klucz publiczny posiadamy, to zostanie nam wyświetlone ostrzeżenie: `WARNING: This key is
not certified with a trusted signature!`. Samo ostrzeżenie nie wpływa jednak na weryfikację podpisu,
a ten jest w porządku, zatem możemy przejść dalej.

Poza plikiem `.dsc` , pobrał się także plik `.tar` . Zawiera on spakowane źródła aplikacji, zwykle
te udostępnianie przez jej dewelopera. W pliku `.dif` zaś są umieszczone wszelkie zmiany jakie muszą
być poczynione, by zdebianizować oryginalne źródła. Po pobraniu tych plików, źródła są wypakowywane
i dostosowywane, wliczając w to też nakładanie łat, jakie opiekun pakietu przygotował. Po tym
zabiegu mamy przygotowany katalog `wpa/` .

W katalogu z wypakowanymi źródłami znajduje się podkatalog `debian/` i to w nim będziemy dokonywać
praktycznie wszelkich zmian. Sam katalog, jak możemy wywnioskować z tych wszystkich powyższych
informacji, jest odrębną całością i jest pakowany oddzielnie. Można go też przenosić między
kolejnymi wersjami źródeł. Jeśli zajrzymy do katalogu gdzie pobraliśmy źródła, możemy dostrzec
również spakowany plik `wpa_2.2-1.debian.tar.xz` . To tam właśnie jest ulokowany katalog
`debian/` .

### Aktualizacja długo nieaktualizowanego pakietu

Jakiś czas temu, próbowałem sobie zaktualizować pakiet `minitube` , bo w repozytorium była dostępna
wersja sprzed chyba 2 lat. Co w takim przypadku należałoby zrobić, by zaktualizować ten pakiet? Na
początek pobieramy stare źródła przy pomocy `apt-get source` . Po ich pobraniu, będziemy mieli
dostęp do pliku `minitube_2.0-1.debian.tar.xz` . Pobieramy ze strony projektu najnowsze źródła i
zapisujemy je w tym samym katalogu. Zmieniamy nazwę pliku z nowymi źródłami, na taką według wzoru:
`program_versja.orig.tar.{gz,bz2,xz}` , po czym wypakowujemy te dwa pliki. Następnie kopiujemy
katalog `debian/` do folderu z nowszą wersją źródeł i przechodzimy do tego katalogu. Poniżej
praktyczny przykład:

    $ apt-get source minitube
    $ wget http://flavio.tordini.org/files/minitube/minitube.tar.gz
    $ mv minitube.tar.gz minitube_2.2.orig.tar.gz
    $ tar xpf minitube_2.2.orig.tar.gz
    $ tar xpf minitube_2.0-1.debian.tar.xz
    $ cp -a debian/ minitube

Teraz pozostaje nam edycja poszczególnych plików. Pierwszym, na który musimy rzucić okiem, jest plik
`debian/control` . Na stronie projektu danej aplikacji, zwykle przy nowszych wersjach, będą
publikowane informacje na temat zależności, które muszą być spełnione, by dana aplikacja działała w
naszym systemie bez zarzutu. Także jeśli podczas budowania nowszej wersji pakietu, system narzeka na
brak pewnych bibliotek, znaczy to, że musimy uaktualnić pole `Build-Depends:` . Nie trzeba też
zwykle definiować zależności, które muszą być spełnione przy instalacji pakietu w systemie, bo one
powinny zostać wygenerowane automatycznie w oparciu o `${shlibs:Depends}, ${misc:Depends},` .

Kolejnym miejscem gdzie prawdopodobnie będziemy musieli zajrzeć przy aktualizacji pakietu jest
katalog `debian/patches/` , który zawiera wszystkie modyfikacje źródeł wprowadzone przez opiekuna
pakietu. Zwykle przy nowszej wersji, część z łatek zwyczajnie staje się zbędna lub też zbudowanie
pakietu za ich sprawą może okazać się niemożliwe. W takim przypadku, w logu będzie informacja
odnośnie łaty, która stwarza problemy i należy ją wyłączyć przez zakomentowanie odpowiedniej
linijki w pliku `debian/patches/series` .

Przy aktualizacji paczki, dobrze jest też za sprawą `dch` uzupełnić wpisy w changelog'u odnośnie
tego co zostało zmienione. Zwykle będziemy korzystać z opcji `-n` , `-i` oraz `-r` . Jeśli byśmy
dokonywali aktualizacji czyjejś paczki, wtedy posługujemy się parametrami `-n` oraz `-r` . W
przypadku aktualizacji swojej, wykorzystujemy `-i` oraz `-r` . Opcja `-n` uzupełnia wpis w
changelog'u o `Non-maintainer upload` :


     minitube (2.2-1.1) UNRELEASED; urgency=medium

      * Non-maintainer upload.
      *

      -- Mikhail Morfikov <morfik@nsa.com>  Fri, 26 Sep 2014 17:30:33 +0200


Tam gdzie są gwiazdki, definiujemy zmiany. Natomiast opcja `-r` zmienia `UNRELEASED` , widoczny
wyżej, na `unstable` , przykład:


     minitube (2.2-1.1) unstable; urgency=medium

       * Non-maintainer upload.
       * New upsteram version.

      -- Mikhail Morfikov <morfik@nsa.com>  Fri, 26 Sep 2014 17:33:19 +0200


I to jest w zasadzie wszystko czego od nas wymaga proces przygotowywania źródeł, przynajmniej w
przypadku aktualizowanych paczek. Musimy teraz zbudować osobno źródła i paczkę `.deb` , tak jak to
zostało opisane wyżej w rozdziale `8` . Dobrze jest także zajrzeć sobie do [tego
poradnika](https://debian-handbook.info/browse/stable/sect.building-first-package.html) i przejrzeć
poszczególne podrozdziały. Więcej bardziej zaawansowanych informacji można znaleźć w [podręczniku
dla deweloperów](https://www.debian.org/doc/manuals/developers-reference/index.html) oraz w
[dokumencie poświęconym polityce debiana](https://www.debian.org/doc/debian-policy/index.html). Na
wiki debiana jest też kilka obszerniejszych wpisów min. ten dotyczący [wprowadzenia do
pakietowania](https://wiki.debian.org/Packaging/Intro), który można obrać jako punkt wyjścia.

### Aktualizacja pakietu z repozytorium git

Jako, że spora ilość projektów jest utrzymywana w jakimś systemie kontroli wersji (CVS), to
przydałoby się poruszyć temat ich aktualizacji. Oczywiście, gdy zostaje wypuszczona nowa wersja
danej aplikacji, to nie ma problemu z aktualizacją takiego oprogramowania. Niemniej jednak,
deweloperzy rozwijając dany projekt niekoniecznie chcą co każdy kolejny commit wypuszczać nową
wersję programu. Sporo problemów i/lub błędów jest naprawianych chwilę po ich zgłoszeniu, co
owocuje kolejnymi commit'ami. Gdy tych commit'ów zbierze się jakaś większa ilość (lub będą łatać
krytyczne dziury), to wtedy dopiero autor projektu decyduje się na wydanie nowej wersji appki.
Przez ten cały czas od jednego wydania do drugiego zwykle korzystamy ze starszej wersji programu do
momentu, aż zostanie wypuszczony nowy release, mimo, że pewne zmiany w projekcie już się dokonały.
Jeśli nie chce nam się czekać na moment wypuszczenia kolejnego release, to możemy pobrać wersję git
danej aplikacji i to z niej zbudować pakiet `.deb` .

Najnowsze źródła zawsze pobieramy za pomocą `git clone` (lub `git pull` , jeśli repozytorium mamy
już na dysku). Następnie tak uzyskany katalog trzeba spakować. Poniżej przykład:

    $ git clone https://github.com/mhogomchungu/ussd-gui/
    $ tar --exclude='.git*' --exclude='.pc' -cf - ussd-gui | xz -9 -c - > ussd-gui_1.2.0~git20160426.orig.tar.xz

W ten sposób uzyskujemy paczkę z oryginalnymi źródłami niezbędnymi w dalszym procesie budowania
pakietu `.deb` . Nazwa tego archiwum zawiera w sobie `~git20160426` . Standardowo po numerze wersji
ostatniego release jest wskazanie, że mamy do czynienia z wersją git ( `~git` ). Następnie jest
określona data najświeższego commit'a, który został przepchnięty do repozytorium, w formie YYYYMMDD.

W changelog'u ( `dch -i` ) uwzględniamy tą wersję oraz dorzucamy informacje na temat tego commit'a.
Można ją wyciągnąć z logu git'a w poniższy sposób:

    $ git log --graph --decorate --oneline
    * 84b2c13 (HEAD -> master, origin/master, origin/HEAD) minor code improvements
    ...

Poniżej zaś przykładowy wpis w pliku changelog'a:

    ussd-gui (1.2.0~git20160426-2) unstable; urgency=medium

      * New upstream snapshot (84b2c13)

     -- Mikhail Morfikov <morfik@nsa.com>  Tue, 26 Apr 2016 21:00:59 +0200

Tak przygotowane źródła można zbudować standardową metodą.
