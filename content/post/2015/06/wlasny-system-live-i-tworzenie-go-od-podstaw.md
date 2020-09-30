---
author: Morfik
categories:
- Linux
date: "2015-06-12T15:48:19Z"
date_gmt: 2015-06-12 13:48:19 +0200
published: true
status: publish
tags:
- pendrive
- live
title: Własny system live i tworzenie go od podstaw
---

Jeśli mamy nieco odbiegające od standardowych oczekiwania dotyczące systemów live, np. wynikają one
z braku obecności pewnych pakietów w obrazie wygenerowanym przez developerów jakiejś konkretnej
dystrybucji, to możemy sobie stworzyć własny obraz live, gdzie mamy możliwość dostosowania całej
konfiguracji takiego systemu wliczając w to również i instalację brakujących pakietów. Obrazy, które
są [dostępne na stronie debiana](https://www.debian.org/CD/live/), zawierają wydanie stabilne i jak
wiadomo, nie jest ono zbyt aktualne pod względem oprogramowania. Natomiast jeśli chodzi o tworzenie
własnych obrazów live, to możemy zdefiniować sobie gałąź, z której mają być pobierane pakiety użyte
w procesie budowania, jak i również dograć te pakiety, które nie są w żaden sposób powiązane z daną
dystrybucją.

<!--more-->
## Narzędzia potrzebne do zbudowania systemu live

Na stronie projektu mamy dostępną [obszerną
dokumentację](https://debian-live.alioth.debian.org/live-manual/stable/manual/html/live-manual.en.html)
na temat własnoręcznego budowania obrazów live, jak i także manuale opisujące funkcje i opcje
poszczególnych narzędzi wykorzystywanych w tym procesie (zawarte w pakiecie `live-manual` ). Jako,
że jest to temat obszerny, polecam choćby przejrzenie tych dokumentów, bo ułatwią one nam pracę ze
skryptami `lb build` czy `lb config` .

W tym wpisie skupimy się na stworzeniu stabilnego obrazu live, bo z nim są najmniejsze problemy. By
nieco ułatwić sobie zadanie, nie będzie wpisywać długich linijek w terminalu i zamiast nich
skorzystamy z plików konfiguracyjnych, w których możemy zdefiniować wszystkie wykorzystywane
parametry. Zaletą tego rozwiązania jest możliwość szybkiej reprodukcji obrazu live, bo przecie całe
środowisko produkcyjne będziemy mieć już przygotowane. Zanim jednak je sobie stworzymy, musimy
wejść w posiadanie odpowiednich narzędzi, a te są dostarczane wraz z pakietem `live-build` .

## Tworzenie struktury katalogów

Po zaopatrzeniu się w odpowiednie narzędzia, możemy przejść do tworzenia środowiska, w którym
będziemy tworzyć obrazy live. Podczas tego procesu będą nam potrzebne uprawnienia root, także
logujemy się na konto administratora systemu i tworzymy odpowiednią strukturę katalogów:

    # mkdir /media/Kabi/debian-live
    # cd /media/Kabi/debian-live
    # lb config

Musimy również stworzyć kilka plików konfiguracyjnych w katalogu `auto/` , z tym, że nie nie musimy
tego robić ręcznie, bo mamy przygotowane pliki szkieletowe, które możemy wykorzystać:

    # cp /usr/share/doc/live-build/examples/auto/* auto/

To co właśnie przygotowaliśmy, to jest minimalna konfiguracja systemu live o domyślnych parametrach.
Teraz jeszcze musimy odpowiednio dostosować sobie konfigurację obrazu. W tym celu edytujemy plik
`auto/config` i dodajemy tam poniższą zwrotkę:

    #!/bin/sh

    set -e

    lb config noauto \
          --apt aptitude \
          --apt-indices true \
          --apt-recommends true \
          --apt-secure true \
          --apt-source-archives false \
          --architectures amd64 \
          --binary-image hdd \
          --binary-filesystem ext4 \
          --bootloader syslinux \
          --debian-installer netinst \
          --debian-installer-distribution stable \
          --debian-installer-gui true \
          --distribution stable \
          --keyring-packages "debian-archive-keyring" \
          --linux-flavours "amd64" \
          --linux-packages "linux-image linux-headers" \
          --memtest memtest86+ \
          --mirror-bootstrap http://ftp.pl.debian.org/debian/ \
          --mirror-binary http://ftp.pl.debian.org/debian/ \
          --mode debian \
          --system live \
          --archive-areas "main contrib non-free" \
          --firmware-binary true \
          --firmware-chroot true \
          --bootappend-live "\
    boot=live \
    config \
    noeject \
    swap=true \
    persistence \
    persistence-encryption=luks \
    persistence-media=removable \
    persistence-label=data \
    live-config.hostname=jaqen-hghar \
    live-config.username=morfik \
    live-config.locales=pl_PL.UTF-8,en_US.UTF-8 \
    live-config.timezone=Europe/Warsaw \
    live-config.keyboard-layouts=pl \
    live-config.utc=yes \
    live-config.noautologin \
    " \
          "${@}"

Trochę tych opcji jest ale nie zawsze wszystkich z nich będziemy potrzebować. Poza tym, szereg z
tych parametrów ma domyślne wartości, zatem można je pominąć. Ja jednak wolę wyraźnie wskazać co i w
jaki sposób ma być budowane. Wyżej mamy podział na dwie grupy parametrów konfiguracyjnych, z których
jedna odpowiada za przygotowanie samego obrazu wynikowego, natomiast druga za parametry startowe
systemu:

Parametry odpowiadające za konfigurację obrazu live:

  - `--apt` określa z jakiego menadżera pakietów korzystać, do wyboru `aptitude|apt-get`
  - `--apt-*` to opcje konfiguracyjne dla powyżej zdefiniowanego menadżera
  - `--architectures` określa architekturę budowanego systemu live
  - `--binary-image` to typ obrazu, do wyboru jeden z `iso|iso-hybrid|netboot|tar|hdd`
  - `--binary-filesystem` definiuje system plików jaki zostanie użyty w obrazie, możliwe
    `fat16|fat32|ext2|ext3|ext4`
  - `--bootloader` określa wykorzystywany bootloader, do wyboru `grub|syslinux`
  - `--debian-installer` daje możliwość załączenia instalatora w wynikowym obrazie live, do wyboru
    `true|cdrom|netinst|netboot|businesscard|live|false`
  - `--debian-installer-distribution` określa jaka dystrybucja ma być instalowana, do wyboru jeden z
    `daily|stable|testing|sid`
  - `--debian-installer-gui` precyzuje czy załączyć także graficzny interfejs instalatora
  - `--distribution` określa dystrybucję wynikowego systemu live
  - `--keyring-packages` daje możliwość określenia dodatkowych pakietów z kluczami gpg
  - `--linux-flavours` określa jakie kernele powinny zostać dołączone
  - `--linux-packages` definiuje jakie dodatkowe pakiety kernela zainstalować
  - `--memtest` dołącza narzędzia testujące pamięć operacyjną RAM, do wyboru
    `memtest86+|memtest86|none`
  - `--mirror-*` odpowiada za serwery lustrzane dla pobieranych pakietów
  - `--mode` określa globalny tryb budowania obrazu, do wyboru `debian|progress|ubuntu`
  - `--system` definiuje jaki rodzaj systemu ma być budowany, do wyboru `live|normal`
  - `--archive-areas` daje możliwość instalowania pakietów z niewolnych obszarów debiana, można
    określić jedną lub więcej opcji z `main|contrib|non-free`
  - `--firmware-*` określa czy dołączyć pakiety mające w nazwie firmware
  - `--bootappend-live` definiuje parametry startowe systemu live

Parametry startowe systemu:

  - `boot` aktywuje narzędzie live-config
  - `noeject` sprawia, że przy restarcie czy zamykaniu systemu nie będą wysuwane żadne nośniki
  - `swap` włącza wykorzystanie wykrytych partycji SWAP
  - `persistence` daje możliwość zachowywania zmian
    ([persistence](/post/persistence-czyli-zachowanie-zmian-w-systemie-live/))
    wprowadzonych w systemie live
  - `persistence-*` definiuje opcje dla persistence
  - `live-config.hostname` ustawia nazwę hosta systemu
  - `live-config.username` definiuje nazwę użytkownika, na którego zostaniemy zalogowani po
    załadowaniu systemu
  - `live-config.locales` określa lokalizacje dla systemu live
  - `live-config.timezone` ustawia strefę czasową
  - `live-config.keyboard-layouts` ustawia układ klawiatury
  - `live-config.utc` przyjmuje, że czas sprzętowy jest ustawiony na UTC
  - `live-config.noautologin` wyłącza automatyczne logowanie na konsole TTY i do środowiska
    graficznego po załadowaniu się systemu

Standardowe parametry logowania to użytkownik `live` oraz hasło `live` . Jeśli zmieniliśmy parametr
`live-config.username` , to nazwą tego użytkownika się posługujemy przy logowaniu w systemie live.

Jeśli korzystamy z szyfrowanego persistence, to również musimy zdefiniować kilka dodatkowych
pakietów, które muszą być uwzględnione w zbudowanym obrazie live. W tym celu tworzymy plik
`config/package-lists/apps.list.chroot` i dodajemy pożądane pakiety:

    cryptsetup
    lvm2

Naturalnie, można tutaj sobie dodać dowolne pakiety. W każdym razie, określamy co nam potrzebne i
dopisujemy do powyższego pliku. By teraz zbudować obraz live, wydajemy kolejno te trzy polecenia:

    # lb clean
    # lb config
    # lb build

Zaowocuje to uruchomieniem narzędzia `debootstrap` , które pobierze minimalnego debiana i z niego
zbuduje nam obraz. Jeśli chcielibyśmy dołączyć jakieś środowisko graficzne, wtedy musimy odpowiednie
pakiety uwzględnić w pliku `live/config/package-lists/desktop.list.chroot` . Oczywiście, nie musimy
definiować setek paczek by zainstalować jakieś określone środowisko graficzne, możemy posłużyć się
metapakietami:

    # apt-cache search --names-only ^task-
    ...
    task-gnome-desktop - GNOME
    ...

Jeśli chcemy zainstalować sobie całe środowisko graficzne GNOME, to wystarczy, że dodamy do listy
ten powyższy pakiet. Natomiast nazwy mniejszych metapakietów można sprawdzić posługując się
narzędziem `debtags` (dostępne w pakiecie pod tą samą nazwą) :

    # debtags search role::metapackage

Po tym jak proces budowania obrazu dobiegnie końca, zostanie utworzony plik `  ` zawierający
spakowany system plików systemu live. Jeśli jednak z jakichś powodów budowa obrazu nie powiedzie
się, warto zajrzeć do pliku `build.log` w katalogu roboczym.
