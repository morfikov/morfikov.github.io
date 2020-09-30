---
author: Morfik
categories:
- Linux
date: "2016-01-19T16:59:36Z"
date_gmt: 2016-01-19 15:59:36 +0100
published: true
status: publish
tags:
- gry
- wine
- quake
title: Quake II pod linux'em bez Wine
---

Linux nie nadaje się za bardzo na konsolę do gier i do tego stwierdzenia raczej nikogo nie trzeba
przekonywać. Wszyscy znamy projekt WineHQ, który umożliwia odpalanie szeregu aplikacji z windowsa, w
tym też i gier, ale zwykle też trzeba się nieco napracować, by daną grę uruchomić pod Wine. Nawet
jeśli się nam to uda, to i tak zawsze będziemy mieć problemy czy to z wydajnością, czy też jakimiś
mniej lub bardziej dającymi się we znaki błędami. Bardzo rzadko zdarza mi się grać w cokolwiek ale
jest kilka kultowych gierek z lat '90, które można odpalić na linux'ie bez zaciągania do tego Wine.
Jedyne czego nam potrzeba to posiadać nośnik z plikami do danej gry. W tym wpisie postaramy się
odpalić Quake II na 64 bitowym debianie.

<!--more-->
## Klient Yamagi Quake II

[Yamagi Quake II](http://www.yamagi.org/quake2/) jest w stanie przerobić windosowskiego Quake II na
natywnego klienta linux'owego. Wymagane jest jednak, by posiadać pliki z grą. Jeśli nie mamy starej
płytki instalacyjnej, to grę możemy pozyskać za sprawą serwisu [gog.com](https://www.gog.com/) lub
też pożyczyć ją od kumpla. Tę grę będziemy musieli zainstalować na windowsie (lub też za pomocą
Wine), by uzyskać dostęp do jej plików, z których zbudujemy klienta na linux'a.

Mając już zainstalowanego klienta windosowskiego, [pobieramy oficjalnego
patch'a 3.20](ftp://ftp.idsoftware.com/idstuff/quake2/q2-3.20-x86-full-ctf.exe) i zapisujemy go
sobie w katalogu roboczym, powiedzmy `quake2/` . Może i jest to plik `.exe` ale możemy go wypakować
za pomocą `uzip` :

    $ mkdir /media/quake2/
    $ cd /media/quake2/
    $ wget ftp://ftp.idsoftware.com/idstuff/quake2/q2-3.20-x86-full-ctf.exe
    $ unzip q2-3.20-x86-full-ctf.exe

Po wypakowaniu, usuwamy te poniższe pliki:

    $ rm 3.20_Changes.txt \
    quake2.exe \
    ref_gl.dll \
    ref_soft.dll \
    baseq2/gamex86.dll \
    baseq2/maps.lst \
    ctf/ctf2.ico \
    ctf/gamex86.dll \
    ctf/readme.txt \
    ctf/server.cfg \
    xatrix/gamex86.dll \
    rogue/gamex86.dll

Z zainstalowanego wcześniej windosowskiego klienta (ewentualnie z płytki CD) kopiujemy poszczególne
rzeczy do odpowiednich katalogów:

    $ cp "/mnt/GOG Games/Quake II/baseq2/pak0.pak" ./baseq2/
    $ cp -R "/mnt/GOG Games/Quake II/baseq2/video" ./baseq2/
    $ cp -R "/mnt/GOG Games/Quake II/music" ./baseq2/

    $ cp "/mnt/GOG Games/Quake II/rogue/pak0.pak" ./rogue/
    $ cp -R "/mnt/GOG Games/Quake II/rogue/video" ./rogue/
    $ cp -R "/mnt/GOG Games/Quake II/music" ./rogue/

    $ cp "/mnt/GOG Games/Quake II/xatrix/pak0.pak" ./xatrix/
    $ cp -R "/mnt/GOG Games/Quake II/xatrix/video" ./xatrix/
    $ cp -R "/mnt/GOG Games/Quake II/music" ./xatrix/

    $ cp "/mnt/GOG Games/Quake II/ctf/pak0.pak" ./ctf/
    $ cp -R "/mnt/GOG Games/Quake II/music" ./ctf/

Pliki z muzyką wymagają dodatkowego zabiegu. Chodzi o to, że muszą być one w formie `02.ogg` ,
`03.ogg` , itd, a są w postaci `Track02.ogg` i trzeba te nazwy przepisać.

Tworzymy osobny katalog roboczy na potrzeby kompilacji, powiedzmy `kompilacja/` . Następnie
pobieramy i wypakowujemy te poniższe pliki (narzędzie `patool` jest dostępne w repo debiana):

    $ mkdir /media/kompilacja/
    $ cd /media/kompilacja/
    $ wget http://deponie.yamagi.org/quake2/quake2-5.32.tar.xz
    $ wget http://deponie.yamagi.org/quake2/quake2-ctf-1.03.tar.xz
    $ wget http://deponie.yamagi.org/quake2/quake2-xatrix-2.03.tar.xz
    $ wget http://deponie.yamagi.org/quake2/quake2-rogue-2.02.tar.xz
    $ patool extract quake2-*

Przechodzimy teraz do kolejnych folderów i kompilujemy poszczególne rzeczy. Potrzebne nam jednak
będą dodatkowe pakiety: `libopenal-dev` , `libsdl2-dev` oraz `libvorbis-dev` . Te pakiety po
kompilacji wraz z ich zależnościami będzie można bez problemu usunąć z systemu.

    # aptitude install libopenal-dev libsdl2-dev libvorbis-dev
    $ cd /media/kompilacja/quake2-5.32/
    $ make
    $ cd /media/kompilacja/quake2-ctf-1.03/
    $ make
    $ cd /media/kompilacja/quake2-rogue-2.02/
    $ make
    $ cd /media/kompilacja/quake2-xatrix-2.03/
    $ make
    # aptitude purge --purge libopenal-dev libsdl2-dev libvorbis-dev

W każdym z powyższych katalogów znajduje się podkatalog `release/` . Jego zawartość musimy
przekopiować do odpowiednich folderów w `/media/quake2/` :

    $ cp -R /media/kompilacja/quake2-5.32/release/* /media/quake2/
    $ cp /media/kompilacja/quake2-ctf-1.03/release/game.so /media/quake2/ctf/
    $ cp /media/kompilacja/quake2-rogue-2.02/release/game.so /media/quake2/rogue/
    $ cp /media/kompilacja/quake2-xatrix-2.03/release/game.so /media/quake2/xatrix/

W ten sposób mamy gotowego klienta Quake II z kilkoma jego dodatkami. Grę możemy uruchomić wpisując
w terminalu jedno z tych poniższych poleceń:

    $ ./quake2
    $ ./quake2 +set game xatrix
    $ ./quake2 +set game rogue
    $ ./quake2 +set game ctf

## Problemy z grą

Prawdopodobnie będziemy musieli dostosować sobie nieco ustawień. W moim przypadku doświadczyłem
nieco dziwnego efektu wizualnego, który objawiał się zamazaniem ekranu. Ta sytuacja została pokazana
dokładnie na poniższej fotce:

![](/img/2016/01/1.quake-ii-bug-wizualny.png#huge)

By wyeliminować ten bug, musimy nieco zmienić konfigurację Quake II, która przechowywana w katalogu
`~/.yq2/` . Edytujemy zatem plik `~/.yq2/baseq2/config.cfg` i ustawiamy tam poniższy parametr:

    set gl_ext_pointparameters "0"

Pozostałe opcje zdają się nie sprawiać żadnych problemów. Niemniej jednak, jeśli z naszym klientem
gry jest coś nie tak, to dobrze jest przejrzeć sobie [ten link](https://github.com/yquake2/yquake2).
W punkcie 5. znajdują się informacje na temat popularnych błędów, które były zgłaszane przez
użytkowników.

## Lepszej jakości tekstury

Istnieje także możliwość wymiany standardowych tekstur na tekstury o większej rozdzielczości. Paczkę
można [pobrać stąd](http://www-personal.umich.edu/~jimw/q2/Quake2/). przechodzimy zatem do katalogu
roboczego z grą i pobieramy stosowne pliki:

    $ cd /media/quake2/
    $ cd baseq2/
    $ wget http://www-personal.umich.edu/\~jimw/q2/Quake2/baseq2/textures_for_q2.zip
    $ unzip textures_for_q2.zip
    $ rm textures_for_q2.zip
    $ cd ..

    $ cd rogue/
    $ wget http://www-personal.umich.edu/\~jimw/q2/Quake2/rogue/textures.zip
    $ unzip textures.zip
    $ rm textures.zip
    $ cd ..

    $ cd xatrix/
    $ wget http://www-personal.umich.edu/\~jimw/q2/Quake2/xatrix/textures.zip
    $ unzip textures.zip
    $ rm textures.zip
    $ cd ..

Włączenie tych tekstur odbywa się automatycznie i znacząco poprawia jakość grafiki w Quake II.
Poniżej porównanie:

![](/img/2016/01/2.quake-ii-tekstury.png#huge)

Można także dociągnąć sobie tekstury modeli, które są dostępne
[tutaj](http://deponie.yamagi.org/quake2/texturepack/), choć zgodnie z tym co ludzie piszą, to nie
wszystkie pasują i nie za dobrze to wygląda. Tak stworzony klient zajmuje trochę ponad 2GiB i można
bez problemu pograć sobie w Quake II na linux'ie.
