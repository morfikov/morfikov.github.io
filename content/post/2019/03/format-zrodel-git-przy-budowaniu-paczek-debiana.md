---
author: Morfik
categories:
- Linux
date: "2019-03-27T05:23:42Z"
published: true
status: publish
tags:
- debian
- git
title: Format źródeł 3.0 (git) przy budowaniu paczek Debiana
---

Pisząc jakiś czas
temu [poradnik na temat budowania paczek .deb](/post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/)
dla dystrybucji linux'a Debian, poruszyłem w nim kwestię związaną z aktualizacją paczki, która
zawierała projekt utrzymywany w systemie kontroli wersji (CVS), np. git. Rozchodzi się tutaj o
format źródeł. Do niedawna myślałem, że są w zasadzie dwa formaty (tych, z których się zwykle
korzysta): `3.0 (native)` oraz `3.0 (quilt)` . Wszystkie moje pakiety budowane do tej pory miały
ten drugi format. Podglądając ostatnio kilka paczek `.deb` , natrafiłem w jednej z nich na format
źródeł `3.0 (git)` . Okazuje się, że ten format jest w stanie bardzo łatwo ogarnąć projekty
hostowane w takich serwisach jak GitHub, czy GitLab i można za jego sprawą nieco ułatwić sobie
życie. Wypadałoby zatem rzucić na niego okiem i ocenić go pod kątem przydatności przy budowaniu
pakietów dla Debiana.

<!--more-->
## Formaty źródeł native, quilt i git

Po zajrzeniu do manuala `dpkg-source` , można odnaleźć z nim dział `SOURCE PACKAGE FORMATS` . Jest
tam opisanych szereg formatów źródeł, które mechanizm budowy pakietów Debiana jest w stanie
obsłużyć. Jest też tam informacja, że jeśli format źródeł budowanego projektu jest nieznany, to
powinno się skorzystać z `3.0 (native)` lub `3.0 (quilt)` . Czym się różnią te dwa formaty od
siebie? W zasadzie to łatkami, które w przypadku tego drugiego formatu są nakładane na źródła po
ich wypakowaniu, a całym tym przedsięwzięciem kieruje `quilt` . Gdy w grę wchodzi teraz
repozytorium git, to wszelkie zmiany w nim są rejestrowane przez CVS i zamiast nakładania patch'a
na źródła, to tworzy się zwykły commit podebrany ze zdalnej gałęzi/repo, przez co odpada nam
bawienie się w ręczne nakładanie łat -- wystarczy zsynchronizować repozytorium, by pobrać
wszystkie nowe zmiany i można budować nowszą wersję aplikacji.

## Przygotowanie źródeł

Przygotowanie źródeł do budowy pakietu `.deb` w przypadku formatu `3.0 (git)` nie różni się zbytnio
w stosunku do formatu `3.0 (quilt)` czy `3.0 (native)` . Dlatego też wypadałoby się pierw
zaznajomić z podlinkowanym we wstępie artykułem, bo w nim są zawarte wszystkie niezbędne informacje
na temat tego jak zdebianizować źródła, by nadawały się pod mechanizm budowy pakietów Debiana.

## Problemy związane z formatem źródeł git

Jako, że format źródeł `3.0 (git)` odnosi się do budowania paczki z repozytorium git, to nie może
ono podlegać jakimkolwiek zmianom z naszej strony. Chodzi generalnie o to, że nawet poprawienie
najdrobniejszej literówki skutkować będzie tym, że VCS będzie widział zmianę (w końcu od tego jest
ten system). Niemniej jednak, mając zmiany w lokalnym repozytorium, `dpkg-source` nie będzie chciał
zezwolić na budowę takiego pakietu i zwróci nam poniższy błąd:

    $ debuild -S -sa -d -i
    ...
    dpkg-source: info: using source format '3.0 (git)'
    dpkg-source: error: uncommitted, not-ignored changes in working directory:
    ...
    dpkg-buildpackage: error: dpkg-source -i -b . subprocess returned exit status 255
    debuild: fatal error at line 1182:
    dpkg-buildpackage -us -uc -ui -S -sa -d -i failed

Jeśli źródła są odpowiednio przygotowane przez upstream, to wszystkie zbędne pliki powinny zostać
uwzględnione w `.gitignore` .

Kolejny problem dotyczy katalogu `debian/` . Jeśli upstream go nie stworzył i nie utrzymuje, to w
zasadzie jest pozamiatane, bo stworzenie takiego katalogu będzie widziane jako zewnętrzna zmiana w
źródłach. Co prawda, można by stworzyć stworzyć plik `.gitignore` wewnątrz katalogu `debian/` i
umieścić w nim `*` , przez co CVS przestanie widzieć zmiany w tym katalogu, jak i sam katalog.
Niemniej jednak, gdybyśmy chcieli zbudować taki pakiet przy pomocy `pbuilder` , to on nam z kolei
zwróci taki błąd:

    $ sudo pbuilder --build nftables_0.9.0-3\~git20190312.1.dsc
    ...
    I: Extracting source
    ...
    dpkg-source: info: extracting nftables in nftables-0.9.0
    dpkg-source: info: cloning nftables_0.9.0-3~git20190312.1.git
    warning: unable to access '/root/.config/git/attributes': Permission denied
    dpkg-source: error: cannot write nftables-0.9.0/debian/source/format: No such file or directory
    E: pbuilder: Failed extracting the source
    ...

Ten błąd uniemożliwia dalszy proces budowy pakietu. W zasadzie to się on nawet nie zdążył rozpocząć.
Wszystko przez ten zignorowany katalog `debian/` , którego teraz nie ma wewnątrz środowiska chroot.

Oczywiście te powyższe problemy znikną jak tylko zostaną stworzone lokalne commit'y ale one z kolei
sprawią, że nasze zmiany (jeśli ich nie pchniemy do upstream'u) będą tracone za każdym jak będziemy
chcieli zsynchronizować zdalne repozytorium (albo będziemy musieli się bawić w merge). Podobnie
sprawa wygląda z nakładaniem zewnętrznych łat. Tego też nie damy rady zrobić bez ingerencji w
historię zmian VCS.

Widać już zatem, że ten format źródeł `3.0 (git)` nie jest taki miły jakby się mogło na pierwszy
rzut oka wydawać. No chyba, że źródła zostaną odpowiednio przygotowane w upstream'ie. Tak czy
inaczej niektóre repozytoria mają już przygotowane odpowiednie drzewo katalogów,
np. [hw-probe](https://github.com/linuxhw/hw-probe/tree/master/debian). Postanowiłem zatem na jego
przykładzie sprawdzić jak się operuje na tym eksperymentalnym formacie źródeł `3.0 (git)` .

## Gdy pojawiają się nowe commit'y w repo git

Deweloperzy rozwijając dany projekt niekoniecznie chcą co każdy kolejny commit wypuszczać nową
wersję programu. Sporo problemów i/lub błędów jest naprawianych chwilę po ich zgłoszeniu, co
owocuje kolejnymi commit'ami. Gdy tych commit'ów zbierze się jakaś większa ilość (lub będą łatać
krytyczne dziury), to wtedy dopiero autor projektu decyduje się na wydanie nowej wersji appki.

Przez cały ten czas od jednego wydania do drugiego korzystamy ze starszej wersji programu do
momentu, aż zostanie wypuszczony nowy release, mimo, że pewne zmiany w projekcie już się dokonały.
Jeśli nie chce nam się czekać na moment wypuszczenia kolejnego release, to możemy zsynchronizować
sobie repozytorium git i z tego świeżego snapshot'a źródeł aplikacji zbudować pakiet `.deb` .
Pamiętajmy przy tym, że tak zbudowana aplikacja może być bardzo niestabilna -- w końcu deweloper
nie odważył się wypuścić stabilnego release, więc można przypuszczać, że appka nie jest jeszcze
gotowa. Jeśli jednak ten fakt nas nie odstrasza, to trzeba pobrać repozytorium projektu, z którego
zamierzamy zbudować paczkę `.deb` :

    $ git clone https://github.com/linuxhw/hw-probe/

Możemy teraz zbudować źródła przy pomocy `debuild -S -sa -d -i` :

    $ debuild -S -sa -d -i
     dpkg-buildpackage -us -uc -ui -S -sa -d -i
    dpkg-buildpackage: info: source package hw-probe
    dpkg-buildpackage: info: source version 1.4-6-git20190103
    dpkg-buildpackage: info: source distribution unstable
    dpkg-buildpackage: info: source changed by Mikhail Novosyolov <mikhailnov@dumalogiya.ru>
     dpkg-source -i --before-build .
     fakeroot debian/rules clean
    dh clean
       dh_auto_clean
            make -j1 clean
    make[1]: Entering directory '/media/Kabi/debian_sources/git-hw-probe/hw-probe'
    echo "Nothing to clean up."
    Nothing to clean up.
    make[1]: Leaving directory '/media/Kabi/debian_sources/git-hw-probe/hw-probe'
       dh_clean
     dpkg-source -i -b .
    dpkg-source: info: using source format '3.0 (git)'
    dpkg-source: info: bundling: --all
    dpkg-source: info: building hw-probe in hw-probe_1.4-6-git20190103.dsc
     dpkg-genbuildinfo --build=source
     dpkg-genchanges -sa --build=source >../hw-probe_1.4-6-git20190103_source.changes
    dpkg-genchanges: info: including full source code in upload
     dpkg-source -i --after-build .
    dpkg-buildpackage: info: source-only upload: Debian-native package
    Now running lintian hw-probe_1.4-6-git20190103_source.changes ...
    N: Using profile debian/main.
    N: Starting on group hw-probe/1.4-6-git20190103
    N: Unpacking packages in group hw-probe/1.4-6-git20190103
    ...
    N: ----
    N: Processing changes file hw-probe (version 1.4-6-git20190103, arch source) ...
    N: ----
    N: Processing buildinfo package hw-probe (version 1.4-6-git20190103, arch source) ...
    N: Finished processing group hw-probe/1.4-6-git20190103
    Finished running lintian.
    Now signing changes and any dsc files...
    ...

    Successfully signed dsc, buildinfo, changes files

Przy budowaniu źródeł w formacie `3.0 (git)` wyrzucany jest błąd `internal error: could not find
the source tarball` (bo nie mamy oryginalnej paczki `.tar` ze źródłami), a przez to `lintian` nie
jest w stanie poddać źródeł analizie `warning: skipping check of source package hw-probe` . Te
błędy jednak nie wpływają na sam proces budowania pakietu `.deb` ale mogą się odbić na jego jakości
przez niespełnienie pewnych wytycznych narzuconych przez Debiana.

Mając zbudowane źródła, możemy zbudować paczkę:

    $ cd ..
    $ sudo pbuilder --build hw-probe_1.4-6-git20190103.dsc

W procesie budowania paczki `.deb` , `pbuilder` wyrzucił również ostrzeżenie `warning: unable to
access '/root/.config/git/attributes': Permission denied` ale nie ma się co nim przejmować. Paczka
powinna się zbudować po dłuższej chwili.

### Synchronizacja repozytorium git

Być może za jakiś czas zostaną dodane nowe commit'y do repozytorium `hw-probe` i będziemy chcieli
zbudować nowszą jego wersję. W takiej sytuacji wystarczy przejść do katalogu, gdzie mamy
repozytorium tego projektu i wydać to poniższe polecenie:

    $ git remote update && git status
    Fetching origin
    On branch master
    Your branch is behind 'origin/master' by 3 commits, and can be fast-forwarded.
      (use "git pull" to update your local branch)

    nothing to commit, working tree clean

Nasze lokalne repozytorium jest 3 commit'y z tyłu w stosunku do upstream'u. Wypadałoby zatem się
zsynchronizować:

    $ git pull
    Updating cba9de1..6352a26
    Fast-forward
     README.md   |  11 ++++++
     hw-probe.pl | 234 +++++++++++++++++++++++++++++++++--------------------
     2 files changed, 168 insertions(+), 77 deletions(-)

I już można budować pakiet `.deb` sposobem opisanym wyżej. Niemniej jednak, trzeba w tym miejscu
zaznaczyć, że jeśli upstream nie podbije wersji z `1.4-6-git20190103` , to paczka będzie co prawda
zawierała wszystkie zmiany wprowadzone w nowych commit'ach ale zastąpi nam te wcześniej zbudowaną.
Ma to spore znaczenie, gdy w grę wchodzi wrzucanie takiego pakietu do repozytorium pakietów, np.
gdy
mamy [lokalnie wdrożone reprepro](/post/tworzenie-repozytorium-przy-pomocy-reprepro/).

## Podsumowanie

Mnie jakoś specjalnie ten format źródeł `3.0 (git)` nie przekonuje. W zasadzie upstream bardzo
rzadko dostosowuje źródła pod kątem wytycznych konkretnej dystrybucji, a nawet jeżeli się to już
zdarzy, to i tak zwykle nie dba się o to, by te pliki były aktualne.

Z kolei problematyczność lokalnego poprawiania błędów przez nakładanie łatek, tak jak to można
robić w przypadku formatu `3.0 (quilt)` , sprawia, że ja jednak będę korzystał z tego rozwiązania
co do tej pory, tj.

    $ git clone git://git.app.org/app
    $ tar --exclude='.git*' --exclude='.pc' -cf - app | xz -9 -c - > app_1.2.3~git20190312.orig.tar.xz

I mamy można się bawić.
