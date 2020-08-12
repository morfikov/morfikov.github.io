---
author: Morfik
categories:
- Linux
date: "2019-04-14T00:10:11Z"
published: true
status: publish
tags:
- debian
- apt/aptitude
- apparmor
title: Mechanizm trigger'ów dla apt/aptitude w Debianie
---

Czasami pewna niestandardowa konfiguracja naszego linux'a może sprawiać pewne problemy podczas
aktualizacji zainstalowanych w nim pakietów. Dla przykładu, wykorzystując mechanizm AppArmor do
okrojenia profilów Firefox'a, muszę tworzyć osobne twarde dowiązania do binarki tej przeglądarki.
Te dowiązania mają taki problem, że jak usunie się plik, na który wskazywały, np. podczas
aktualizacji paczki, to utworzenie pliku w tym samym miejscu przez menadżer pakietów
`apt`/`aptitude` nie sprawi, że te dowiązania zaczną ponownie funkcjonować poprawnie (tak jak to
jest w przypadku dowiązań symbolicznych). Z początku usuwałem te stare dowiązania i tworzyłem nowe
ale postanowiłem w końcu [poszukać rozwiązania](https://forum.dug.net.pl/viewtopic.php?id=30382),
które by zautomatyzowało cały ten proces i uczyniło go transparentnym dla użytkownika końcowego.
Tak natrafiłem na mechanizm Debianowych
trigger'ów ([deb-trigger](https://manpages.debian.org/unstable/dpkg-dev/deb-triggers.5.en.html)),
które aktywują się za każdym razem ilekroć pliki w konkretnych ścieżkach są ruszane w jakiś sposób
przez menadżer pakietów. W tym artykule spróbujemy sobie zaprojektować taki trigger i obadać czy
może on nam się w ogóle do czegoś przydać

<!--more-->
## Jak stworzyć paczkę zawierającą trigger

By stworzyć trigger, który będzie wywoływał podczas instalacji czy aktualizacji pakietów określone
polecenia,
musimy [stworzyć paczkę .deb]({{< baseurl >}}/post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/).
W tej paczce `.deb` główną rolę będą grać dwa pliki. Jednym z nich będzie skrypt shell'owy z
pożądanymi przez nas poleceniami, a drugi będzie określał ścieżki, których zmiana będzie pociągała
wykonanie tego skryptu.

Stwórzmy sobie zatem katalog roboczy i przejdźmy do niego:

    $ mkdir firefox-trigger-1.0/
    $ cd firefox-trigger-1.0/

W tym katalogu stwórzmy folder `debian/` , a w nim szereg plików:

    $ mkdir debian/
    $ mkdir debian/source/
    $ touch debian/changelog
    $ touch debian/compat
    $ touch debian/control
    $ touch debian/copyright
    $ touch debian/files
    $ touch debian/firefox-trigger.postinst
    $ touch debian/firefox-trigger.triggers
    $ touch debian/rules
    $ touch debian/source/format

Oczywiście szkielety wszystkich tych pliki można wygenerować za pomocą `dh_make --native` ale
akurat w tym przypadku czyszczenie śmieci powstałych za sprawą tego narzędzia zajmie więcej czasu
niż stworzenie tych plików od podstaw. Poniżej znajduje się zawartość każdego z tych plików.

### Plik debian/changelog

W pliku `debian/changelog` dodajemy poniższy wpis, który umożliwi podpięcie pakietu pod mechanizm
budowania pakietów przy pomocy `pbuilder`:

    firefox-trigger (1.0) unstable; urgency=medium

      * Initial Release.

     -- Mikhail Morfikov <morfik@nsa.com>  Wed, 28 Mar 2018 20:47:50 +0200

### Plik debian/compat

W pliku `debian/compat` określamy poziom kompatybilności `debhelper`'a i standardowo jest to `11` :

    11

### Plik debian/control

W pliku `debian/control` nie musimy zbytnio definiować żadnych zależności. Przydałoby się jednak
uzależnić ten trigger od pakietu dostarczającego pliki, których zmiany nas interesują. W tym
przypadku chodzi o paczkę `firefox` . Trzeba także dobrać nazwy dla źródeł oraz wynikowej paczki.

    Source: firefox-trigger
    Section: misc
    Priority: optional
    Maintainer: Mikhail Morfikov <morfik@nsa.com>
    Build-Depends: debhelper (>= 11)
    Standards-Version: 4.2.1
    #Vcs-Git: https://anonscm.debian.org/git/collab-maint/firefox-trigger.git
    #Vcs-Browser: https://anonscm.debian.org/cgit/collab-maint/firefox-trigger.git

    Package: firefox-trigger
    Architecture: all
    Depends: ${shlibs:Depends}, ${misc:Depends},
     firefox
    Description: Simple trigger for Firefox to create hard links
     Simple trigger for firefox to create hard links

### Plik debian/copyright

W pliku `debian/copyright` umieszczamy standardowe informacje na temat licencji plików:

    Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
    Upstream-Name: firefox-trigger
    Source: https://firefox-trigger.lh

    Files: *
    Copyright: 2018 Mikhail Morfikov <morfik@nsa.com>
    License: GPL-3.0+

    Files: debian/*
    Copyright: 2018 Mikhail Morfikov <morfik@nsa.com>
    License: GPL-3.0+

    License: GPL-3.0+
     This program is free software: you can redistribute it and/or modify
     it under the terms of the GNU General Public License as published by
     the Free Software Foundation, either version 3 of the License, or
     (at your option) any later version.
     .
     This package is distributed in the hope that it will be useful,
     but WITHOUT ANY WARRANTY; without even the implied warranty of
     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
     GNU General Public License for more details.
     .
     You should have received a copy of the GNU General Public License
     along with this program. If not, see <https://www.gnu.org/licenses/>.
     .
     On Debian systems, the complete text of the GNU General
     Public License version 3 can be found in "/usr/share/common-licenses/GPL-3".


### Plik debian/firefox-trigger.prerm

W pliku `debian/firefox-trigger.prerm` znajdują się instrukcje dla menadżera pakietów odnośnie tego
co on ma zrobić przy usuwaniu paczki z trigger'em. Wypadałoby pousuwać wszystkie pliki, które
ręcznie zamierzamy stworzyć. W przeciwnym razie po usunięciu paczki z trigger'em te pliki nam
zostaną i będą jedynie zaśmiecać system.

    #!/bin/sh

    set -e

    case "$1" in
        remove|upgrade|deconfigure)
            rm -f /usr/lib/firefox/firefox-p
            rm -f /usr/lib/firefox/firefox-b
        ;;

        failed-upgrade)
        ;;

        *)
            echo "prerm called with unknown argument \`$1'" >&2
            exit 1
        ;;
    esac

    #DEBHELPER#

    exit 0

### Plik debian/firefox-trigger.postinst

W pliku `debian/firefox-trigger.postinst` jest zawarta instrukcja na temat akcji, które menadżer
pakietów ma przeprowadzić, gdy paczka z trigger'em zostanie zainstalowana oraz, gdy nasz trigger
zostanie aktywowany ( `triggered)` ):

    #!/bin/sh

    set -e

    case "$1" in
        configure)
            ln /usr/lib/firefox/firefox /usr/lib/firefox/firefox-p
            ln /usr/lib/firefox/firefox /usr/lib/firefox/firefox-b
        ;;

        triggered)
            rm /usr/lib/firefox/firefox-p
            rm /usr/lib/firefox/firefox-b
            ln /usr/lib/firefox/firefox /usr/lib/firefox/firefox-p
            ln /usr/lib/firefox/firefox /usr/lib/firefox/firefox-b
        ;;

        abort-upgrade|abort-remove|abort-deconfigure)
        ;;

        *)
            echo "postinst called with unknown argument \`$1'" >&2
            exit 1
        ;;
    esac

    #DEBHELPER#

    exit 0

### Plik debian/firefox-trigger.triggers

W pliku `debian/firefox-trigger.triggers` określamy ścieżki do plików, których zmiana będzie
aktywować trigger. Ścieżek może być tyle ile chcemy. Definiujemy także dyrektywę trigger'a jako
`interest-noawait` , z której powinno się korzystać w przypadku, gdy funkcjonalność trigger'a nie
zalicza się do tych najistotniejszych.

    interest-noawait /usr/lib/firefox/firefox

### Plik debian/rules

Pliku `debian/rules` w zasadzie nie ma co ruszać, bo nie będziemy budować żadnego kodu, a jedynie
wyprodukujemy sobie paczkę `.deb` :

    #!/usr/bin/make -f

    export DH_VERBOSE = 1

    %:
        dh $@

### Plik debian/source/format

Jako, że budujemy paczkę natywną dla Debiana, to ustawiamy jej format w pliku
`debian/source/format` na `3.0 (native)` :

    3.0 (native)

## Budowanie paczki z trigger'em

Mając przygotowane pliki możemy zbudować sobie paczkę przy pomocy narzędzia `pbuilder` :

    $ debuild -S -sa -d -i
    $ sudo pbuilder --build ../firefox-trigger_1.0.dsc

Choć akurat w tym przypadku zaprzęganie `pbuilder`'a mija się ździebko z celem i ten pakiet
trigger'a można zbudować prościej korzystając z `dpkg-buildpackage` :

    $ dpkg-buildpackage -b -uc -us

## Testowanie trigger'a

Mając paczkę `.deb` z trigger'em możemy ją zainstalować w systemie. Następnie spróbujmy ponownie
zainstalować aplikację, która manipuluje w jakiś sposób ścieżkami określonymi w pliku
`debian/firefox-trigger.triggers` . W tym przypadku przy reinstalacji/aktualizacji przeglądarki
Firefox, log wygląda mniej więcej tak:

    # apt-get dist-upgrade
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    Calculating upgrade... Done
    The following packages will be upgraded:
      firefox
    1 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    Need to get 0 B/41.6 MB of archives.
    After this operation, 3,072 B disk space will be freed.
    Do you want to continue? [Y/n]
    Retrieving bug reports... Done
    Parsing Found/Fixed information... Done
    (Reading database ... 248269 files and directories currently installed.)
    Preparing to unpack .../firefox_59.0.2-1_amd64.deb ...
    Unpacking firefox (59.0.2-1) over (59.0.1-1) ...
    Processing triggers for mime-support (3.60) ...
    Processing triggers for desktop-file-utils (0.23-2) ...
    Setting up firefox (59.0.2-1) ...
    Processing triggers for man-db (2.8.2-1) ...
    Processing triggers for hicolor-icon-theme (0.17-2) ...
    Processing triggers for firefox-trigger (1.0) ...

I tu w ostatniej linijce jest już widoczne przetwarzanie naszego trigger'a.

Jak widać na przytoczonym wyżej przykładzie jesteśmy w stanie podpiąć dowolne akcje czy polecenia
pod menadżer pakietów w chwili, gdy interesujące nas pliki są zmieniane przez system w drodze
instalacji/aktualizacji pakietów. Oczywiście w tym przypadku dorobiliśmy jedynie dwa proste
dowiązania do binarki Firefox'a ale możliwości jakie daje posiadanie swoich własnych trigger'ów są
w zasadzie ograniczone jedynie naszą wyobraźnią. Tak czy inaczej, umiejętność korzystania z
trigger'ów jest wielce nieoceniona przy automatyzacji zadań.
