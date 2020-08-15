---
author: Morfik
categories:
- Linux
date: "2015-11-16T12:34:52Z"
date_gmt: 2015-11-16 11:34:52 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- pliki
- foldery
title: Uprawnienia do plików systemowych w linux'ie
---

Każdy z nas popełnia błędy. Niektóre z nich są błahe i w sporej części łatwe do poprawienia. Zwykle
też nie niosą one ze sobą większych konsekwencji. Natomiast błędy, które popełniamy podczas pracy w
systemie operacyjnym wykonując różne prace administracyjne mogą nas czasem słono kosztować. W
linux'ach ogromną rolę odgrywają prawa do plików. O ile root ma dostęp do wszystkich plików, to w
przypadku zwykłych użytkowników (czy tez usług systemowych) już tak nie jest. Przypadkowa zmiana
tych uprawnień może zaowocować problemami związanymi z bezpieczeństwem takiego systemu, a w
niektórych przypadkach może nawet uniemożliwić jego start. Na necie kilka razy obił mi się o oczy
temat, gdzie ludzie przez przypadek (lub też [całkiem
świadomie](https://forum.dug.net.pl/viewtopic.php?id=27876)) zmienili masowo uprawnienia w takich
katalogach jak `/usr/` czy `/etc/` . Powiem nawet więcej, mi się raz taka sytuacja kiedyś
przytrafiła. Co w takim przypadku można zrobić? Czy jedyną opcją, jaka nam pozostaje, to ponowna
instalacja sytemu? Na szczęście nie, bo uprawnienia do plików możemy sobie zwyczajnie spisać i
odtworzyć je w późniejszym czasie.

<!--more-->
## Problematyczne uprawnienia do plików

Każdy linux ma w swojej strukturze szereg plików i katalogów. Wszystkie z nich posiadają
uprawnienia, które możemy wyciągnąć, np. wydając w terminalu polecenie `ls -al` . Weźmy zatem
przykładowo plik `/etc/shadow` . Właścicielem tego pliku jest `root` . Grupa z kolei wskazuje na
`shadow` . Dodatkowo uprawnienia tego pliku to `-rw-r-----` , czyli `640` . Zatem wszyscy
użytkownicy, który należą do grupy `shadow` mogą odczytać ten plik, a użytkownik `root` jest w
stanie go także zapisać.

Problem z grupami jest taki, że sporo z nich nie posiada stałych identyfikatorów. Numerki wszystkich
aktualnych numerów grup, można sobie odczytać z pliku `/etc/group` . W przypadku wgrywania nowych
pakietów, lub tez zmiany kolejności wgrywania tych, które zwykliśmy mieć w systemie, te
identyfikatory grup mogą ulec zmianie. Trzeba się z tym jak najbardziej liczyć gdy odtwarzamy pliki
z archiwów `.tar.gz` , gdzie zwykliśmy korzystać z opcji `-p` .

Pojedyncze pliki wgrywane z backupu nie są znowu aż tak groźne. Zawsze można nadać właściwe
uprawnienia do plików przy pomocy `chown` i mieć problem z głowy. Większym wyzwaniem jest sytuacja,
gdzie zmianie uległy uprawnienia do szeregu plików. Tak jak widzieliśmy na przykładzie pliku
`shadow` , nie ma on standardowych uprawnień, tj. `root:root` i `644` . Zatem w tego typu
przypadkach, gdy te uprawnienia do plików ulegną zamianie, to zbytnio nie mamy możliwości ich
odtworzyć. Zwykle jedyną opcją jest ponowna instalacja systemu. To jednak nie rozwiązuje problemu.
No może i rozwiązuje ale zastanówmy się przez chwilę nad innym przykładem. Co w przypadku gdyby
jakiś robal nam te uprawnienia poprzestawiał? Skąd wiedzielibyśmy, że te uprawnienia są takie jak
być powinny? Czy kiedykolwiek weryfikowaliśmy te uprawnienia do plików w swoich linux'ach?

Mi osobiście przytrafiła się bardzo dziwna sytuacja, gdzie podczas wielomiesięcznej pracy w jednym z
moich systemów, zauważyłem, że kilka pojedynczych plików w niewyjaśnionych okolicznościach zmieniło
sobie uprawnienia. Nie były to krytyczne zmiany pod względem bezpieczeństwa i wyglądały raczej na
przypadkowe. Chodziło generalnie o zamianę kilku mniej istotnych grup. Jako, że ten problem dotyczył
kilku plików, to można było przyjąć, że pewnie są w systemie też i inne pliki, których uprawnienia
są nie takie jak być powinny. Dlatego postanowiłem opracować sobie mechanizm, który by zabezpieczył
mój system przed taką niepożądaną zmianą uprawnień. Może niekoniecznie przed samą zmianą ale by był
w stanie wykryć pewne anomalie, tak by było wiadomo, że coś jest nie tak.

## Jak zapisać uprawnienia do plików

Poniżej znajduje się prosty skrypt, który został zaczerpnięty z [forum
dug'a](https://dug.net.pl/tekst/117/) . Ma on za zadanie przeskanowanie określonych lokalizacji przy
pomocy `find` i wyciągnięcie uprawnień do wszystkich plików i katalogów w nich się znajdujących.

    #!/bin/sh

    GDZIE=$PWD

    #FOLDER /bin
    touch /$GDZIE/bin.sh
    echo '/bin/sh'>/$GDZIE/bin.sh
    find /bin -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/bin.sh

    #FOLDER /sbin
    touch /$GDZIE/sbin.sh
    echo '/bin/sh'>/$GDZIE/sbin.sh
    find /sbin -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/sbin.sh

    #FOLDER /lib
    touch /$GDZIE/lib.sh
    echo '/bin/sh'>/$GDZIE/lib.sh
    find /lib -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/lib.sh

    #FOLDER /usr
    touch /$GDZIE/usr.sh
    echo '/bin/sh'>/$GDZIE/usr.sh
    find /usr -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/usr.sh

    #FOLDER /etc
    touch /$GDZIE/etc.sh
    echo '/bin/sh'>/$GDZIE/etc.sh
    find /etc -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/etc.sh

    #FOLDER /opt
    touch /$GDZIE/opt.sh
    echo '/bin/sh'>/$GDZIE/opt.sh
    find /opt -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/opt.sh

    #Folder /var
    touch /$GDZIE/var.sh
    echo '/bin/sh'>/$GDZIE/var.sh
    find /var -printf "chown %u:%g %p && chmod  %m %p\n">>/$GDZIE/var.sh

    chmod +x /$GDZIE/*.sh

Ten skrypt powinien być uruchamiany jedynie na dopiero co postawionym systemie, bo tylko w takim
przypadku, uprawnienia do plików są wiarygodne. Oczywiście nic nie stoi na przeszkodzie, by ten
skrypt wywoływać sobie cyklicznie i porównywać tak uzyskiwane uprawnienia z tymi wzorcowymi.

## Jak odtworzyć uprawnienia do plików

Gdy uruchomimy powyższy skrypt, to w katalogu roboczym zostanie utworzonych szereg plików
zawierających linijki podobne do tej poniżej:

    chown root:shadow /etc/shadow && chmod  640 /etc/shadow

Jest to zwykłe wykonanie `chown` oraz `chmod` z odpowiednimi opcjami na każdym przeskanowanym pliku.
Zatem, jeśli uprawnienia do plików się nie zgadzają z jakiegoś powodu i mamy podejrzenie, że problem
może dotyczyć całego systemu, to wystarczy uruchomić te wynikowe skrypty. W przypadku gdy chcemy
profilaktycznie sprawdzić, czy pliki w systemie mają odpowiednie uprawnienia, to wystarczy porównać
dwa wyjścia powyższego skryptu, np. korzystając z `meld` . Wszelkie różnice będą widoczne jak na
dłoni. Trzeba jednak pamiętać o aktualizacji wyników. Jakby nie patrzeć, nawet jeśli zweryfikujemy
uprawnienia starych plików, to wraz z rozwojem systemu mogą w nim pojawić się inne pliki, których
uprawnienia mogą w późniejszym czasie ulec zmianie.
