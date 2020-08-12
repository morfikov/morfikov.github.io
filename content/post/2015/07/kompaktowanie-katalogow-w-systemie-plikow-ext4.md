---
author: Morfik
categories:
- Linux
date: "2015-07-11T13:39:06Z"
date_gmt: 2015-07-11 11:39:06 +0200
published: true
status: publish
tags:
- system-plików
- ext4
title: Kompaktowanie katalogów w systemie plików ext4
---

Jakiś czas temu pewien człowiek [miał dziwaczny
problem](https://forum.dug.net.pl/viewtopic.php?id=27485). Jak możemy wyczytać w przytoczonym linku,
system tego użytkownika lekko mówiąc nie zachowywał się tak jak powinien. Objawiało się to przez
dość ekstensywne wykorzystywanie pamięci operacyjnej RAM przy zwykłym listowaniu plików via `ls` w
pewnych określonych katalogach. Struktura systemu plików zdaje się być porządku, bo program `fsck`
nie zwraca żadnych błędów. Zatem w czym problem?

<!--more-->
## Magiczna liczba 4096

Jeśli rzucimy okiem na listing katalogów w naszym systemie, okaże się, że większość z nich ma
rozmiar `4096` bajtów. Katalogi są jak pliki i przechowują dane, z tym, że jeśli chodzi o foldery to
te przechowują dane o znajdujących się w nich plikach, w tym też ich nazwy. Domyślna wartość 4KiB
odpowiada rozmiarowi bloku w systemie plików, a ten w przypadku ext4 na przeciętnej instalacji
linuxa wynosi 4KiB. Rzućmy zatem okiem jak wygląda kawałek struktury drzewa katalogów w naszym
systemie:

    $ ls -al /
    total 92K
    drwxr-xr-x  23 root root 4.0K 2015-07-10 12:37:03 ./
    drwxr-xr-x  23 root root 4.0K 2015-07-10 12:37:03 ../
    drwxr-xr-x   2 root root 4.0K 2015-07-09 06:15:38 bin/
    drwxr-xr-x   5 root root 4.0K 2015-07-10 12:56:52 boot/
    drwxr-xr-x  22 root root 4.0K 2015-07-11 12:45:46 dev/
    drwxr-xr-x 158 root root  12K 2015-07-10 21:50:04 etc/
    drwxr-xr-x   6 root root 4.0K 2015-06-24 11:07:17 home/
    drwxr-xr-x  21 root root 4.0K 2015-07-10 21:47:00 lib/
    drwxr-xr-x   2 root root 4.0K 2015-07-10 21:47:00 lib32/
    drwxr-xr-x   2 root root 4.0K 2015-07-10 21:47:13 lib64/
    drwx------   2 root root 4.0K 2014-12-31 18:24:27 lost+found/
    drwxr-xr-x  16 root root 4.0K 2015-04-06 15:39:49 media/
    drwxr-xr-x   2 root root 4.0K 2015-06-20 18:37:50 mnt/
    drwxr-xr-x  11 root root 4.0K 2015-07-10 12:36:34 opt/
    dr-xr-xr-x 224 root root    0 2015-07-11 12:16:52 proc/
    drwxr-xr-x  37 root root 4.0K 2015-07-11 12:45:45 root/
    drwxr-xr-x  26 root root  860 2015-07-11 12:18:42 run/
    drwxr-xr-x   2 root root  12K 2015-07-10 21:47:07 sbin/
    drwxr-xr-x   2 root root 4.0K 2015-01-17 13:10:39 srv/
    dr-xr-xr-x  13 root root    0 2015-07-11 12:16:53 sys/
    drwxrwxrwt  18 root root 4.0K 2015-07-11 12:47:40 tmp/
    drwxr-xr-x  11 root root 4.0K 2015-05-29 08:15:35 usr/
    drwxr-xr-x  13 root root 4.0K 2015-06-15 10:57:40 var/
    lrwxrwxrwx   1 root root   11 2014-11-27 11:14:41 cdrom -> media/cdrom/

Jak widzimy wyżej, część z katalogów ma większy rozmiar niż 4KiB, np. katalog `/etc/` . Naturalne
jest, że im więcej plików dany katalog zawiera, tym jego rozmiar będzie większy. Problem w tym, że
to miejsce nie jest zwalnianie nawet w przypadku usuwania plików z katalogu. Zatem jeśli wgramy
mnóstwo plików do katalogu i po chwili je usuniemy, rozmiar folderu nie ulegnie zmianie. Natomiast
zwiększy się w przypadku gdy zabraknie mu okupowanej przestrzeni. Ma to na celu zredukować
fragmentację katalogów.

Możemy przeprowadzić mały test. Stwórzmy zatem jakiś katalog i wgrajmy do niego przy pomocy
narzędzia `cat` kilka plików:

    $ mkdir test
    $ cd test
    $ touch {0..100}
    $ ls -al ../ | grep test
    drwxr-xr-x   2 morfik morfik 4.0K 2015-07-11 13:08:58 test/

Jak widzimy rozmiar katalogu w dalszym ciągu ma 4KiB mimo, że wgraliśmy do niego 100 plików o
nazwach z przedziału 0 do 100. Wgrajmy trochę więcej plików:

    $ touch {0..4000}
    $ ls -al ../ | grep test
    drwxr-xr-x   2 morfik morfik  68K 2015-07-11 13:10:50 test/

I tu już widzimy, że rozmiar katalogu zwiększył się do 68KiB. Usuńmy zatem wszystkie stworzone
wcześniej pliki:

    $ rm *
    $ ls -al ../ | grep test
    drwxr-xr-x   2 morfik morfik  68K 2015-07-11 13:11:50 test/

Po mimo faktu, że katalog jest pusty, folder waży 68KiB ale jeśli wgramy do niego znów kilka plików:

    $ touch {0..3000}
    $ ls -al ../ | grep test
    drwxr-xr-x   2 morfik morfik  68K 2015-07-11 13:13:12 test/

To możemy zauważyć, że rozmiar nie uległ zmianie.

## Kompaktowanie katalogów

Jeśli operowaliśmy na jakichś folderach dość intensywnie, cześć z nich może okupować zbędne miejsce,
co może lekko odbić się na wydajności systemu. Przy pomocy `fsck` możemy przeprowadzić
[kompaktowanie
katalogów](https://unix.stackexchange.com/questions/38639/how-to-compact-a-directory), tak by nieco
je zoptymalizować.

Kompaktowanie katalogów można przeprowadzić jedynie przy odmontowanym systemie plików i robimy to
korzystając z opcji `-f` oraz `-D` . Poniżej przykład:

    # fsck.ext4 -Dvf /dev/mapper/debian_laptop-root
    ...
    Pass 3A: Optimizing directories
    ...
    root: ***** FILE SYSTEM WAS MODIFIED *****

Opcja `-D` wymusza dodatkowy przebieg przy pomocy, którego przeprowadzane jest kompaktowanie
katalogów. Widzimy także, że system plików został zmodyfikowany.
