---
author: Morfik
categories:
- Linux
date: "2015-06-30T12:03:20Z"
date_gmt: 2015-06-30 10:03:20 +0200
published: true
status: publish
tags:
- debian
- apt/aptitude
title: Przeszukiwanie zawartości pakietów (apt-file)
---

Podczas procesu kompilacji pakietów często zdarza się tak, że brakuje jakichś zależności, bez
których dany pakietów nie chce się nam zbudować. W większości przypadków, system powinien nam
podpowiedzieć jaki pakiet powinniśmy doinstalować. Nie zawsze jednak będzie to takie oczywiste i
jedyne co nam zostanie zwrócone, to ścieżka danego pliku lub tylko jego nazwa. Nawet jeśli nie
kompilujemy programów, to podczas zwykłego użytkowania komputera możemy potrzebować odnaleźć pakiet,
który zawiera pewien określony plik binarny czy konfiguracyjny. Jak zatem odnaleźć się w gąszczu
plików i katalogów by efektywnie ustalić pakiet, który zawiera interesujące nas pliki?

<!--more-->
## Menadżer pakietów dpkg

W debianie mamy do dyspozycji szereg menadżerów pakietów. Jednym z nich jest `dpkg` , za którego to
sprawą jesteśmy w stanie przeszukać zawartość pakietów pod kątem dopasowań. Przykładowo, załóżmy, że
musimy ustalić, w jakim pakiecie znajduje się często używane przez nas polecenie `less` . Możemy to
zrobić w poniższy sposób:

    $ dpkg -S less

W powyższym przypadku zostanie zwróconych bardzo dużo wyników, a to z tego względu, że `dpkg`
inaczej niż my identyfikuje ciąg `less` . Dla niego to tylko cztery literki w określonej kolejności
i każda nazwa, która je zawiera, zostanie wypisana po wydaniu tego powyższego polecenia. Jeśli mamy
unikatowe nazwy jakichś programów, czy plików konfiguracyjnych, to raczej możemy je podawać bez
większych obaw.

Co w przypadku tego `less` ? Jak odnaleźć pakiet, który go zawiera? Tutaj raczej nie obejdzie się
bez wykorzystania `which` , który wskazuje pełną ścieżkę do danego programu. W `dpkg` możemy
określić również ścieżkę w miejscu szukanej nazwy, przykładowo:

    $ which less
    /usr/bin/less

    $ dpkg -S /usr/bin/less
    less: /usr/bin/less

Zatem wiemy, że polecenie `less` znajduje się w pakiecie o tej samej nazwie, bo zawiera ono
określoną przez nas ścieżkę do pliku.

Menadżer pakietów `dpkg` potrafi również zwrócić listę plików, które zostały wgrane do systemu za
sprawą jakiegoś pakietu. Przykładowo:

    $ dpkg -L less
    /.
    /usr
    /usr/lib
    /usr/lib/mime
    /usr/lib/mime/packages
    /usr/lib/mime/packages/less
    /usr/share
    /usr/share/doc
    /usr/share/doc/less
    /usr/share/doc/less/changelog.Debian.gz
    ...

Bardzo przydatna cecha gdy potrzebujemy odnaleźć, np. wszystkie pliki konfiguracyjne danego
programu.

## Pakiet apt-file

Problem z `dpkg` jest taki, że potrafi on operować jedynie na zainstalowanych w systemie pakietach.
Jeśli poszukujemy nazwy w pakiecie, który nie jest zainstalowany, np. chcielibyśmy pozyskać taki
pakiet w oparciu o szukaną frazę, to `dpkg` nam w tym nie pomoże i musimy skorzystać z innego
rozwiązania, tj. z narzędzia `apt-file` , które jest dostępne w pakiecie pod tą samą nazwą.
`apt-file` jest często używany przy budowaniu pakietów, gdzie korzystając z minimalnego [środowiska
chroot]({{< baseurl >}}/post/przygotowanie-srodowiska-chroot-do-pracy/) nie mamy w systemie
praktycznie żadnych pakietów i w przypadku wystąpienia problemów z zależnościami, jedyne informacje
jakie mamy, to właśnie nazwy brakujących plików lub ścieżek do nich.

### Generowanie indeksu plików

Pakiet `apt-file` nie posiada konfiguracji jako takiej ale trzeba wygenerować indeks plików i
regularnie go aktualizować. To w tym indeksie trzymana jest lista wszystkich plików we wszystkich
pakietach w całej dystrybucji jak i również wszelkich pozostałych repozytoriach, które mamy
aktywowane w pliku `/etc/apt/sources.list` . By utworzyć/zaktualizować indeks, wydajemy poniższe
polecenie:

    # apt-file update

Utworzony w ten sposób indeks jest systemowy i wszyscy użytkownicy będą z niego korzystać. Istnieje
możliwość generowanie indeksu jako zwykły użytkownik, na wypadek gdyby administrator się opieprzał i
nie aktualizował tego systemowego regularnie. W przypadku gdy to my jesteśmy administratorem systemu
i zapominamy o aktualizacji indeksu, to możemy dopisać sobie tę poniższą linijkę do cron'a:

    12    20    *     *     6 /usr/bin/nice -n 19 /usr/bin/ionice -c 3 /usr/bin/apt-file update

### Przeszukiwanie indeksu

Poniżej jest przedstawiony przykład z życia wzięty. Podczas kompilacji został wyrzucony błąd, który
był wynikiem niespełnionej zależności, czyli brakowało jakiegoś pakietu w systemie. Poniżej jest
wycinek tego
    błędu:

    CMake Error at /usr/lib/x86_64-linux-gnu/cmake/Qt5LinguistTools/Qt5LinguistToolsConfig.cmake:22 (message):
      The package "Qt5LinguistTools" references the file

         "/usr/lib/x86_64-linux-gnu/qt5/bin/lrelease"

      but this file does not exist.  Possible reasons include:

Widzimy wyraźnie, że została zwrócona informacja, że plik `lrelease` nie został odnaleziony. Mając
nazwę pliku (ewentualnie ścieżkę) możemy przy pomocy `apt-file` przeszukać indeks plików:

    $ apt-file search /usr/lib/x86_64-linux-gnu/qt5/bin/lrelease
    qttools5-dev-tools: /usr/lib/x86_64-linux-gnu/qt5/bin/lrelease

Został nam zwrócony pakiet `qttools5-dev-tools` i to jego instalacja jest wymagana by ukończyć z
powodzeniem budowę tego przykładowego pakietu.
