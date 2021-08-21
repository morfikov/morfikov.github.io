---
author: Morfik
categories:
- Linux
date: "2015-11-04T18:23:36Z"
date_gmt: 2015-11-04 16:23:36 +0100
published: true
status: publish
tags:
- debian
- apt
- snapshot
- repozytorium
title: Migawka (snapshot) repozytorium debiana
---

Aktualizacje systemu niosą ze sobą nowsze wersje pakietów. Czasami mają one błędy, które wychodzą na
jaw po jakimś czasie korzystania z danej aplikacji. W takiej sytuacji zwykle zachodzi potrzeba
cofnięcia wersji kilku pakietów. Jest jednak wielce prawdopodobne, że akurat tej wersji pakietu,
której potrzebujemy, nie znajdziemy z repozytorium debiana. Pobieranie pojedynczych pakietów z
internetu przez klikanie w pierwszy lepszy link, który zostanie nam zwrócony przez wyszukiwarkę, nie
jest dobrym pomysłem. Na szczęście w przypadku debiana nie musimy się aż tak narażać. A to z tego
względu, że [debian robi migawki (shapshots) swoich repozytoriów][1] 4 razy dziennie (co 6 godzin).
W ten sposób mamy dostęp do różnych stanów repozytoriów, w tym też tych, które zawierają pakiety
aktualnie niedostępne w repozytoriach. W tym wpisie postaramy się pobrać i zainstalować
nieistniejące pakiety z takich snapshot'ów.

<!--more-->
## Snapshot'y repozytorium

W linku, który podałem we wstępnie, znajdują się snapshot'y repozytorium debiana z dość długiego
okresu czasu. Mamy tam wyraźny podział na lata, miesiące, dni i godziny. Jeśli nie pamiętamy [kiedy
dokładnie była przeprowadzana aktualizacja systemu][2], to wystarczy w logu apt czy aptitude
odszukać datę i godzinę. Mając ustalony czas (np. 2015-11-01, godzina 08:12) odszukujemy
[odpowiednią pozycję][3].

Po przejściu do tego konkretnego statu repozytorium, naszym oczom ukazało się drzewo katalogów. W
przypadku gdy potrzebujemy tylko jednego pakietu, to możemy go pobrać bezpośrednio z tej strony.
Wystarczy przejść do odpowiednio folderu i pobrać paczkę `.deb`. Natomiast jeśli chcemy cofnąć wiele
pakietów, to o wiele prostszym rozwiązaniem będzie dodanie tego snapshot'a do pliku
`/etc/apt/sources.list` .

## Dodawanie adresu snapshot'a do pliku sources.list

Dodawanie adresu snapshot'a do listy repozytoriów sprowadza się do wrzucenia odpowiedniego linku do
pliku `sources.list` . Generalnie rzecz biorąc, kopiujemy adres strony snapshot'a, tj.
`https://snapshot.debian.org/archive/debian/20151101T042306Z/` dodając do niego odpowiednie sekcje,
przykładowo:

    deb https://snapshot.debian.org/archive/debian/20151101T042306Z/ testing main contrib non-free

Zapisujemy plik i aktualizujemy listę repozytoriów via `apt-get update` . W logu powinniśmy ujrzeć
wpisy odnoszące się do `https://snapshot.debian.org` . Jeśli tak jest w istocie, to jesteśmy teraz w
stanie zainstalować starsze wersje pakietów. Załóżmy, że chcemy cofnąć wersję pakietu `vlc` . W tym
celu przy pomocy `apt-cache` patrzymy jakie wersje są dostępne:

    # apt-cache policy vlc
    vlc:
      Installed: 2.2.1-5+b1
      Candidate: 2.2.1-5+b1
      Version table:
     *** 2.2.1-5+b1 0
            990 http://ftp.de.debian.org/debian/ sid/main amd64 Packages
            100 /var/lib/dpkg/status
         2.2.1-4+b1 0
            500 http://snapshot.debian.org/archive/debian/20151101T042306Z/ testing/main amd64 Packages

Mamy zatem dostępne dwie wersje: `2.2.1-5+b1` oraz `2.2.1-4+b1` , by teraz cofnąć wersję tego
pakietu do tej ze snapshot'a, wydajemy poniższe polecenie:

    # aptitude install vlc=2.2.1-4+b1
    # aptitude hold vlc

Cofnięte wersje pakietów zostaną zaktualizowane ponownie gdy będziemy przeprowadzać aktualizację
systemu. Dlatego skorzystaliśmy z `aptitude hold`, który sprawi, że pakiet `vlc` nie będzie
aktualizowany. By usunąć pakiet z listy zatrzymanych, korzystamy z opcji `unhold` . To powyższe
rozwiązanie nie jest wygodne i może prowadzić do niestabilności systemu. Jeśli bawimy się wieloma
wersjami pakietów, które pochodzą z różnych repozytoriów, to lepszym rozwiązaniem jest skorzystanie
z apt pinning.

## Stare snapshot'y i problemy z InRelease

Starsze snapshot'y ulegają przedawnieniu i `apt-get`/`aptitude` może nam zwrócić błąd przy
aktualizacji listy pakietów,
    przykładowo

    E: Release file for http://snapshot.debian.org/archive/debian/20140501T105009Z/dists/testing/InRelease is expired (invalid since 545d 6h 50min 10s). Updates for this repository will not be applied.

Mamy tam wyraźną informację, że pakiety z tego snapshot'a nie będą dostępne, bo plik `InRelease`
uległ przedawnieniu prawie 550 dni temu. Gdybyśmy chcieli pozyskać pakiety z tego snapshot'a, to
oczywiście możemy to zrobić. Wystarczy dopisać `Acquire::Check-Valid-Until=0` do `apt-get update` .
Ten parametr wyłącza sprawdzanie pola `Valid-Until` w pliku `InRelease` repozytorium. Zatem linijka
aktualizująca te przestarzałe listy repozytoriów wyglądałaby tak jak poniżej:

    # apt-get -o Acquire::Check-Valid-Until=0 update

W przypadku gdy często bawimy się z różnymi wersjami pakietów, parametr `Acquire::Check-Valid-Until`
możemy dodać do [pliku apt.conf][4].

Warto w tym miejscu dodać, że opcję sprawdzania ważności repozytorium można także dodać dla
pojedynczego repozytorium w pliku `/etc/apt/sources.list` stosując `[check-valid-until=no]` ,
przykładowo:

   deb [check-valid-until=no] https://snapshot.debian.org/archive/debian/20151101T042306Z/ testing main contrib non-free


[1]: https://snapshot.debian.org/archive/debian/
[2]: /post/aktualizacja-systemu-logowanie-komunikatow/
[3]: https://snapshot.debian.org/archive/debian/20151101T042306Z/
[4]: /post/konfiguracja-apt-i-aptitude-w-pliku-apt-conf/
