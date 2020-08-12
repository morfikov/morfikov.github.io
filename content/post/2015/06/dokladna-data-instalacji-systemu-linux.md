---
author: Morfik
categories:
- Linux
date: "2015-06-08T20:21:27Z"
date_gmt: 2015-06-08 19:21:27 +0200
published: true
status: publish
tags:
- instalacja
title: Dokładna data instalacji systemu linux
---

Czy zadawaliście sobie kiedyś pytanie na temat tego jaka jest dokładna data instalacji aktualnie
używanego przez was systemu operacyjnego? To chyba powinna być jedna z podstawowych danych, do
których użytkownik powinien mieć swobodny dostęp, by wiedział kiedy instalował swój system. Jednak
w przypadku linux'a z jakiegoś powodu ciężko jest ten fakt ustalić. Postaramy się zatem ocenić
możliwie dokładnie to kiedy proces instalacji linux'a mógł mieć miejsce.

<!--more-->
## Nieprecyzyjna data instalacji

Użytkownicy, którzy wykorzystują do instalacji swojego systemu płytki instalacyjne Debiana, raczej
nie powinni mieć problemów z ustaleniem wieku systemu. Przybliżona data instalacji może nam zostać
pokazana przez linux'a na kilka sposobów.

Pierwszym z nich jest sprawdzenie dat plików w katalogu `/var/log/installer/` przy pomocy `ls -al` .
Problem w tym, że ten katalog nie zostanie stworzony w przypadku instalowania systemu w sposób inny
niż za sprawą płytek instalacyjnych. Możemy jeszcze przeszukać kilka innych katalogów, by ustalić
datę najstarszego pliku przy pomocy narzędzia `find` :

    # find /var/log -type f -printf '%T+ %p\n' | sort | head -n 1
    2014-11-27+12:09:32.4999270240 /var/log/cups/page_log

Choć moim zdaniem poleganie na datach plików może być zwodnicze. Niby katalog `/var/log/` wydaje
się być najrozsądniejszym miejscem na analizę dat ale też nie dałbym sobie głowy uciąć za ten
powyższy wynik.

Innym sposobem może być sprawdzenie daty utworzenia systemu plików partycji `/` (tej, na której
zainstalowaliśmy linux'a):

    # tune2fs -l /dev/mapper/debian_laptop-root | grep 'Filesystem created:'
    Filesystem created:       Wed Nov 26 21:11:02 2014

No już lepiej ale też nie do końca, bo co w przypadku, gdy będziemy instalować system na tym samym
systemie plików, a poprzedniego Debiana pozbędziemy się via `rm -Rf /*` ?

## Czy data instalacji systemu linux jest do ustalenia?

To wręcz nieprawdopodobne, że nie ma prostego rekordu, w którym można by zarejestrować datę
instalacji systemu. Ostatnim wysiłkiem sprawdźmy jeszcze jedną możliwość, mianowicie datę ostatniej
zmiany hasła użytkownika `sys` . Raczej nikt nie zmieniał hasła temu użytkownikowi, prawda? A, że
jest on tworzony wraz z instalacją systemu, to być może dostaniemy dokładną datę instalacji:

    # passwd -S sys | tail -1 | awk '{print $3}'
    11/27/2014

No może nie mamy tutaj godziny ale przy założeniu, że interesuje nas jedynie konkretny dzień
instalacji linux'a, to możemy się raczej zgodzić, że ten system został postawiony 27 listopada 2014
roku.
