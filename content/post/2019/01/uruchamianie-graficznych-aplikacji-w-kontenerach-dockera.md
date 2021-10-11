---
author: Morfik
categories:
- Linux
date: "2019-01-27T11:32:18Z"
published: true
status: publish
tags:
- debian
- firefox
- namespaces
- docker
GHissueID: 314
title: Uruchamianie graficznych aplikacji w kontenerach Docker'a
---

[Bawiąc się ostatnio na Debianie przestrzeniami nazw sieciowych](/post/jak-uruchomic-firefoxa-w-osobnej-przestrzeni-nazw-sieciowych/),
wpadł mi do głowy pomysł na nieco bardziej zautomatyzowaną formę separacji procesów użytkownika od
pozostałej części systemu. Co by nie mówić, opisany w podlinkowanym artykule sposób uruchomienia
Firefox'a niezbyt mi przypadł do gustu. Nowy sposób separacji zakłada za to wykorzystanie
kontenerów Docker'a, w których to będzie uruchamiany dowolny proces, np. Firefox, a całym
przedsięwzięciem związanym z procesem konteneryzacji będzie zajmował się już Docker. W ten sposób
uruchomienie dowolnej aplikacji, w tym też tych graficznych (GUI), będzie sprowadzać się do wydania
w terminalu tylko jednego polecenia. Zatem do dzieła.

<!--more-->
## Konfiguracja Docker'a

Nie będę w tym artykule poruszał tematu konfiguracji samej usługi Docker'a. Generalnie, to Docker
jest w pełni zautomatyzowany i po jego instalacji w systemie można go używać OOTB. Problemem może
okazać się wdrożenie Docker'a w systemie, który ma dość niestandardową konfigurację, np. korzysta z
`nftables` zamiast `iptables` lub też wykorzystywane są statyczne interfejsy mostka. W takim
przypadku trzeba sobie obadać [dokumentację Docker'a](https://docs.docker.com/) i ręcznie
dostosować pewne rzeczy.

Technicznie rzecz ujmując, to konfiguracja kontenerów Docker'a może odbywać się na dwa sposoby.
Możemy albo wpisywać wszystkie polecenia ręcznie w terminal, co nie jest wygodne, albo też możemy
posłużyć się plikami `dockerfile` oraz `docker-compose.yml` i w tym artykule to właśnie one zostaną
wykorzystane.

Plik `dockerfile` znajduje zastosowanie przy budowaniu obrazów, natomiast konfiguracja kontenera
jest ogarniana przez plik `docker-compose.yml` . Dodatkowo, oba te pliki można połączyć tworząc tym
samym niezły automat, który przy pomocy jednego polecenia zbuduje, skonfiguruje i odpali nam cały
kontener z Firefox'em.

Obrazami Docker'a można zarządzać z poziomu zwykłego użytkownika. Nie trzeba zatem do tego celu
zaprzęgać administratora root. Wystarczy dodać zwykłego użytkownika do grupy `docker` .

Dalsza część artykułu zakłada, że mamy już w pełni przygotowanego Docker'a do pracy.

## Oficjalne obrazy Debiana dla Docker'a

Jako, że będziemy pracować z kontenerami Docker'a, to musimy sobie zbudować obraz, z którego
uczynimy swego rodzaju punkt wyjścia. [Tutaj znajdują się obrazy](https://hub.docker.com/), z
których możemy skorzystać. Jako, że ja używam na co dzień dystrybucji Debian, to mnie najbardziej
interesują [oficjalne obrazy Debiana](https://hub.docker.com/_/debian/). Oczywiście można
skorzystać z obrazów innych dystrybucji.

Wyżej w linku jest zawarta informacja na temat wspieranych tagów, które można wykorzystać w pliku
`dockerfile` . Plik `dockerfile` jest instrukcją dla Docker'a, według której będzie on próbował
stworzyć dla nas kontener, który my będziemy mogli później wykorzystać do własnych już celów.

## Przygotowanie kontenera Docker'a

Stwórzmy sobie zatem katalog roboczy o nazwie `docker-firefox/` , a w nim dwa puste póki co pliki:
`dockerfile` oraz `docker-compose.yml` :

    $ mkdir docker-firefox/
    $ cd docker-firefox/
    $ touch dockerfile docker-compose.yml

### Konfiguracja kontenera (docker-compose.yml)

Zacznijmy może od uzupełnienia pliku `docker-compose.yml` . Poniżej znajduje się już gotowy plik,
natomiast pod nim jest krótkie wyjaśnienie użytych w nim opcji:

    version: '3.6'

    services:
      browser:
        build:
          context: .
          dockerfile: dockerfile
        image: firefox:morfitest
        container_name: firefox
        hostname: firefox
        domainname: local
        restart: "no"
        volumes:
          - user_profile:/home/morfitest/.mozilla
          - /tmp/.X11-unix:/tmp/.X11-unix
        environment:
          - DISPLAY=$DISPLAY
        networks:
          default:
            ipv4_address: 10.10.2.10

    volumes:
      user_profile:

    networks:
      default:
        name: firefox
        driver: bridge
        ipam:
          driver: default
          config:
            - subnet: 10.10.2.0/24

Przede wszystkim, wcięcia mają znaczenie i nie wolno o nich zapominać. Ten powyższy plik
`docker-compose.yml` z grubsza dzieli się na trzy sekcje. Konfiguracja sieci jest opcjonalna i w
przypadku nieokreślenia jej, Docker użyje domyślnego mostka i sam skonfiguruje sieć dla kontenera.

Opcji, które możemy wykorzystać w pliku `docker-compose.yml`
jest [cała masa](https://docs.docker.com/compose/compose-file/). Dlatego też wypadałoby sobie
przeczytać ten podlinkowany dokument, by ocenić czy coś jest nam jeszcze potrzebne. Poniżej zaś
znajduje się krótka rozpiska użytych opcji w stworzonym wyżęj pliku `docker-compose.yml` .

#### Sekcja services:

Każdy plik `docker-compose.yml` musi mieć określoną wersję. Dlatego też na samej górze pliku
podajemy `version: '3.6'` . Następnie rozpoczynamy blok usług, tj. `services:` i w niej
określamy konfigurację poszczególnych usług. Nazwy są dowolne. W tym przypadku będziemy
konfigurować usługę przeglądarki, którą nazwiemy sobie roboczo `browser:` . W podsekcji `browser:`
mamy szereg kluczy:

- `build:` -- odnosi się do etapu budowania obrazu na potrzeby kontenera. Mamy tutaj dwa dodatkowe
parametry, które określają nazwę pliku, tj. `dockerfile` oraz jego położenie względem pliku
`docker-compose.yml` . W tym przypadku `.` oznacza, że oba pliki znajdują się w tym samym katalogu.
- `image:` -- jako że budujemy własny obraz, to Docker nazwie go nam `firefox:morfitest` i w ten
sposób będziemy mogli się do tego obrazu odwoływać w strukturze Docker'a.
- `container_name:` -- to nazwa kontenera.
- `hostname:` -- to nazwa hosta wykorzystywana wewnątrz kontenera.
- `domainname:` -- to nazwa domeny wykorzystywana wewnątrz kontenera.
- `restart:` -- określa co zrobić z kontenerem w przypadku wystąpienia błędów. Jeśli nie chcemy, by
kontener się ponownie uruchomił samoczynnie lub niepożądane jest, by startował wraz z usługą
Docker'a, to określamy tutaj `"no"` .
- `volumes:` -- przy pomocy tego klucza określamy zasoby (katalogi) maszyny hosta, które mają być
widoczne w kontenerze (montowanie via `bind` ). Jesteśmy też w stanie poinformować Docker'a, by
plików w katalogach wewnątrz kontenera nie zmieniał przy różnych czynnościach związanymi z
zarządzaniem kontenerami/obrazami. W tym przypadku, jako że będziemy korzystać z aplikacji
mającej interfejs graficzny, musimy w kontenerze udostępnić katalog `/tmp/.X11-unix/` . Jeśli
zaś chodzi o pliki, których Docker nie powinien ruszać, to oczywiście określamy profil przeglądarki.
- `environment:` -- jest w stanie ustawić pewne zmienne środowiskowe wewnątrz kontenera. Potrzebna
nam jest zmienna `$DISPLAY` , która musi pasować do tej, którą mamy aktualnie wyeksportowaną w
naszym systemie.
- `networks:` -- określa sieć, do której zostanie przypisany kontener oraz jego adres IP. Nazwa
`default` musi pasować do tej, którą określiliśmy w globalnej sekcji `networks:` .

#### Sekcja volumes:

Wszystkie zasoby z wnętrza kontenera, które mają przetrwać wszelkie zmiany dotyczące samego
kontenera, trzeba globalnie określić w `volumes:` . Standardowo nazwa użyta tutaj, tj.
`user_profile` musi pasować do tego zasobu, który był określony w sekcji `services:` .

Warto tutaj nadmienić, że Docker ma pewne problemy z montowaniem zasobów (z użyciem `bind` ) z
prawami użytkownika innego niż
root. [Generalnie to Docker tego nie potrafi](https://github.com/moby/moby/issues/3124) i nie można
przy użyciu globalnej sekcji `volumes:` skonfigurować punktów montowania dla plików, które mają
zostać wyprowadzone poza kontener, gdy chcemy odpalać przeglądarkę jako inny użytkownik niż root.
Dlatego też w pliku `dockerfile` trzeba będzie przygotować volume pod profil przeglądarki. Nie
usuwamy też opcji od volume z pliku `docker-compose.yml` -- muszą one być zdefiniowane, bo inaczej
Docker będzie tworzył nam losowe voluminy za każdym razem jak tylko będziemy podnosić kontener,
przez co odtworzenie plików profilu Firefox'a nie byłoby zbytnio możliwe.

#### Sekcja networks:

Konfiguracja sieci jest opcjonalna. Jeśli chcemy jednak mieć nieco większą kontrolę nad tym
aspektem pracy kontenera, to możemy zdefiniować szereg parametrów sieciowych:

- `default:` -- to nazwa sieci użytkownika. Może być ona dowolna ale musi pasować do tego co
zostało określone w sekcji `services:` .
- `name:` -- to nazwa sieci, która będzie widoczna w strukturze Docker'a, do której będziemy mogli
się odwołać.
- `driver:` -- to domyślny sterownik użyty na potrzeby sieci. Standardowo `bridge` jest
wykorzystywany.
- `ipam:` -- określa dodatkowe parametry adresacji IP i w zasadzie wypadałoby tutaj zdefiniować
podsieć.

### Konfiguracja fazy budowy obrazu (dockerfile)

W zasadzie to połowa pracy już za nami. Został nam jeszcze jednak do skonfigurowania sam obraz, a
do tego celu trzeba posłużyć się plikiem `dockerfile` . Do pliku `dockerfile` będziemy dodawać
kolejno polecenia, które powiedzą Docker'owi co ma robić w fazie budowania obrazu z Firefox'em.

Na początek musimy określić, z którego obrazu (bazowy upsteram) zamierzamy korzystać. Ja na co
dzień korzystam z Debian Sid ale dla lepszej anonimizacji można skorzystać z innej wersji systemu,
czy nawet innej dystrybucji linux'a. Na potrzeby tego artykułu wystarczy nam `debian:sid` .
Dodajemy zatem poniższą linijkę do pliku `dockerfile` :

    FROM debian:sid

Następnie musimy zainstalować potrzebne pakiety. W sumie to potrzebny nam jest jedynie pakiet
`firefox` , jego zależności oraz te rzeczy, które uważamy za niezbędne, np. dodatki do Firefox'a:

    RUN apt-get update && apt-get install -y \
        firefox \
    && rm -rf /var/lib/apt/lists/*

Należy tutaj zaznaczyć, by nie wywoływać `apt-get update` i `apt-get install` w osobnych linijkach
z `RUN` . Doprowadzi to do problemów z cache podczas przebudowy/aktualizacji kontenera.

Następnie definiujemy polecenie, które doda nam zwykłego użytkownika wewnątrz kontenera:

    RUN useradd \
          --create-home \
          --home-dir /home/morfitest/ \
          --shell /bin/bash \
          --uid 1000 \
          --user-group \
          morfitest

Przełączamy teraz użytkownika w skrypcie, na tego cośmy go sobie stworzyli przez dodanie poniższej
linijki:

    USER morfitest

Od tej pory każde następne polecenie będzie wykonywane z uprawnieniami tego użytkownika, a nie
użytkownika root. I tu właśnie przychodzi rozwiązanie problemu z wyprowadzeniem profilu
przeglądarki Firefox poza obręb kontenera Docker'a. Dodajemy te dwa poniższe polecenia:

    RUN mkdir /home/morfitest/.mozilla
    VOLUME ["/home/morfitest/.mozilla"]

W ten sposób katalog `/home/morfitest/.mozilla` będzie miał uprawnienia zwykłego użytkownika, przez
co nie będziemy mieli problemów z jego zapisem, gdy odpalimy Firefox'a jako zwykły user.

Eksportujemy również zmienną `$HOME` , która jest potrzebna części aplikacji do poprawnego
działania oraz określamy katalog roboczy, czyli ten, do którego zostaniemy zrzuceni, np. po
zalogowaniu się na shell:

    ENV HOME /home/morfitest
    WORKDIR /home/morfitest

W pliku `dockerfile` może się znajdować tylko i wyłącznie jedno wywołanie `CMD` . Służy ono jako
domyślne polecenie, które ma zostać uruchomione po podniesieniu kontenera. Jeśli nie chce nam się
zbytnio ręcznie podnosić Firefox'a po uruchomieniu kontenera, to możemy ustawić w `CDM` ścieżkę do
binarki FF:

    CMD /usr/bin/firefox

To polecenie zostanie wywołane z uprawnieniami użytkownika, który został określony wyżej w pliku
za sprawą `USER` .

## Budowa i aktywacja kontenera Docker'a z Firefox'em

Będąc w katalogu, w którym mamy pliki `dockerfile` oraz `docker-compose.yml` , wydajemy poniższe
polecenie, by zbudować kontener (można również `docker-compose up` ):

    $ docker-compose build
    Building browser
    Step 1/9 : FROM debian:sid
    sid: Pulling from library/debian
    6d3c280df34a: Pull complete
    Digest: sha256:4897384eea515b05134d87a7799b2233c38e0a8029955e69820b83788e56e1e0
    Status: Downloaded newer image for debian:sid
     ---> 9be61b3e4c11
    Step 2/9 : RUN apt-get update && apt-get install -y     firefox && rm -rf /var/lib/apt/lists/*
     ---> Running in f99d397de4b1
    Get:1 http://cdn-fastly.deb.debian.org/debian sid InRelease [243 kB]
    Get:2 http://cdn-fastly.deb.debian.org/debian sid/main amd64 Packages [8378 kB]
    Fetched 8621 kB in 6s (1511 kB/s)
    Reading package lists...
    Reading package lists...
    Building dependency tree...
    Reading state information...
    The following additional packages will be installed:
    ...

Widać, że proces budowy obrazu odbywa się w etapach i w zależności od rozbudowania pliku
`dockerfile` , tych etapów może być więcej lub mniej. Tak czy inaczej po chwili powinien nam się
zbudować obraz:

    ...
    Removing intermediate container f99d397de4b1
     ---> acd9681c498f
    Step 3/9 : RUN useradd       --create-home       --home-dir /home/morfitest/       --shell /bin/false       --uid 1000       --user-group       morfitest
     ---> Running in b95ac564729b
    Removing intermediate container b95ac564729b
     ---> df8b08ec2d2d
    Step 4/9 : USER morfitest
     ---> Running in eba238535846
    Removing intermediate container eba238535846
     ---> ebd78268960e
    Step 5/9 : RUN mkdir /home/morfitest/.mozilla
     ---> Running in 430fff3f5c32
    Removing intermediate container 430fff3f5c32
     ---> e0d61a9d8b53
    Step 6/9 : VOLUME ["/home/morfitest/.mozilla"]
     ---> Running in a68181ee8aad
    Removing intermediate container a68181ee8aad
     ---> 6eba20c13aed
    Step 7/9 : WORKDIR /home/morfitest
     ---> Running in c0c39975a8b0
    Removing intermediate container c0c39975a8b0
     ---> 2223398de088
    Step 8/9 : ENV HOME /home/morfitest
     ---> Running in f105aa0e67f0
    Removing intermediate container f105aa0e67f0
     ---> 3ba1573a543c
    Step 9/9 : CMD /usr/bin/firefox
     ---> Running in 1d950faa6202
    Removing intermediate container 1d950faa6202
     ---> ecbaa2eb4f45

    Successfully built ecbaa2eb4f45
    Successfully tagged firefox:morfitest

Obraz powinien być już widoczny przez Docker'a:

    $ docker image ls
    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    firefox                 morfitest           ecbaa2eb4f45        2 hours ago         633MB
    debian                  sid                 9be61b3e4c11        3 weeks ago         113MB

Są tutaj dwa obrazy. Ten `debian:sid` pobraliśmy z internetu i w oparciu o niego stworzyliśmy
własny kontener `firefox:morfitest` , który posiada dokładnie to samo co `debian:sid` plus szereg
dodatkowych pakietów i ich konfigurację.

Ten kontener można bez problemu odpalić przy pomocy poniższego polecenia:

    $ docker-compose up

Powinien nam on bez większego problemu wystartować, a naszym oczom powinno ukazać się okno
przeglądarki Firefox:

![](/img/2019/01/001-firefox-docker-container-debian-linux.png#huge)

### Aktualizacja kontenera Docker'a z Firefox'em

Z czasem pakiety w utworzonym przez nas obrazie przestaną być aktualne i przydałoby się przebudować
cały kontener, tak by zawierał najnowszą wersję Firefox'a, czy też ogólnie nowszą wersję systemu
Debian. Taki proces przebudowy kontenera jest bardzo prosty i w zasadzie sprowadza się do wydania w
terminalu tego poniższego polecenia:

    $ docker-compose build --pull

Na liście obrazów powinny pojawić się nowe pozycje:

    $ docker image ls
    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    firefox                 morfitest           8825e8a74132        7 minutes ago       645MB
    debian                  sid                 4bbbed41ab08        2 weeks ago         114MB
    <none>                  <none>              ecbaa2eb4f45        4 weeks ago         633MB
    debian                  <none>              9be61b3e4c11        8 weeks ago         113MB

Jeśli nowe obrazy działają bez zarzutu (automatycznie zostaną przepięte), to możemy się pozbyć
przy okazji tych starych:

    $ docker image prune
    $ docker image ls
    REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
    firefox                 morfitest           8825e8a74132        20 minutes ago      645MB
    debian                  sid                 4bbbed41ab08        2 weeks ago         114MB

## Problemy natury bezpieczeństwa

Powyżej opisany sposób na odpalenie Firefox'a (czy ogólnie rzecz ujmując aplikacji graficznych) w
kontenerze Docker'a jest najprostszą opcją mającą zrealizować to zadanie. Nie jest jednak
najbezpieczniejszą, bo proces wewnątrz kontenera jest w stanie komunikować się z procesem `X`
odpalonym na hoście (przez unix socket, który podmontowaliśmy wewnątrz kontenera). W ten sposób
Firefox działa mniej więcej na podobnej zasadzie, co jego lokalna wersja, czyli przy braku
odseparowania od systemu hosta i wszystkie aspekty bezpieczeństwa, które trzeba byłoby uwzględnić
przy pracy z lokalnym FF, również trzeba wziąć pod uwagę przy pracy z FF zamkniętym w kontenerze
Docker'a.

Po co zatem się bawić w zamykanie przeglądarki internetowej wewnątrz kontenera Docker'a skoro
niewielki z tego pożytek? Cały ten proces był jedynie odpowiedzią na zagadnienie ogarnięcia w jakiś
przyzwoity sposób uruchamiania aplikacji w osobnych przestrzeniach nazw sieciowych (network
namespaces). Ta konteneryzacja FF, która została opisana wyżej, realizuje to zadanie. Drugorzędne
znaczenie ma tutaj bezpieczeństwo samej aplikacji. Chodziło generalnie o manipulację szeregiem
zmiennych, które zobaczy zdalny serwer WWW, np. lokalny adres IP, fingerprint przeglądarki czy
systemu operacyjnego, nazwa tego systemu, jak i również zainstalowane czcionki. Wszystkie te
wymienione informacje (oraz cała masa innych rzeczy) są zwracane do serwera przy odwiedzaniu strony
WWW i można na ich podstawie śledzić naszą aktywność w sieci. Stosując kontenery Docker'a możemy
stworzyć sobie kompletnie odrębne tożsamości internetowe i korzystać z nich, gdy tego potrzebujemy.
Dodatkowo, pakiety wysyłane w świat można poddać obróbce w filtrze pakietów `nftables`/`iptables` i
to zadanie nie stanowi praktycznie żadnego problemu, gdy korzystamy z kontenerów Docker'a. No i
oczywiście mamy praktycznie taką samą kontrolę nad dostępem do plików procesów z wnętrza kontenera,
co w przypadku uzbrojenia aplikacji lokalnej w profil AppArmor'a. Wszystkie te rzeczy jednocześnie
sprawiają, że możliwość zapakowania przeglądarki internetowej do kontenera Docker'a jest całkiem
przyzwoitym rozwiązaniem, mimo, że nie daje nam ono dodatkowego bezpieczeństwa w stosunku do sesji
X działającej lokalnie.

Oczywiście, jako że to był najprostszy sposób na zamknięcie aplikacji GUI w kontenerze Docker'a, to
zarazem znaczy, że nie była to jedyna opcja. Na dobrą sprawę,
to [tych sposobów jest jeszcze kilka](http://wiki.ros.org/docker/Tutorials/GUI) i wymagają one
osobnego przyjrzenia się im, w
szczególności [przesyłanie sesji X z kontenera Docker'a przez SSH](https://blog.docker.com/2013/07/docker-desktop-your-desktop-over-ssh-running-inside-of-a-docker-container/)
oraz
postawienie [osobnej sesji X wewnątrz kontenera przy wykorzystaniu VNC](https://github.com/fcwu/docker-ubuntu-vnc-desktop)
ale na to rzucę okiem później.
