---
author: Morfik
categories:
- Linux
date: "2015-06-19T13:24:34Z"
date_gmt: 2015-06-19 11:24:34 +0200
published: true
status: publish
tags:
- pliki
- foldery
title: Dzielenie pliku i łączenie jego części w całość
---

Obecnie technika podziału jednego pliku na szereg mniejszych nie jest już tak szeroko stosowana jak
parę lat temu gdy internet dopiero zaczynał pojawiać się w naszych domach. Głównym powodem jest
postęp technologii i dziś już mało kto posiada łącza 128Kbit/s, wobec czego przesłanie pliku, który
waży nawet kilka GiB nie jest niczym niezwykłym i nie ma potrzeby go dzielić na części. Niemniej
jednak, z jakichś powodów ludzie chcą mieć taką możliwość i zastanawiają się [jak podzielić plik na
kilka
części](https://unix.stackexchange.com/questions/24630/whats-the-best-way-to-join-files-again-after-splitting-them)
na jednej maszynie i połączyć je na innym komputerze.

<!--more-->
## Dzielenie pliku przy pomocy split

Na internecie można spotkać się z szeregiem aplikacji specjalnie dedykowanych zadaniu podziału
plików. Moim zdaniem jest to wynajdywanie koła na nowo, bo przecie już w linxuie mamy dostępne
niskopoziomowe narzędzia, które potrafią realizować tę kwestię w sposób doskonały i nie musimy
tworzyć nowych aplikacji by ten cel osiągnąć.

Załóżmy, że mamy jakiś film, który zajmuje 1.9 GiB i chcemy go pociąć na części 100 MiB. Oczywiście,
to może być dowolny plik, np. obraz .iso . Jak zatem podzielić plik na kilka części? Do tego celu
służy narzędzie o jakże wyszukanej nazwie `split` . Jeśli chcemy by wynikowe pliki miały stały i do
tego ustalony rozmiar, to dzielenie pliku sprowadza się do wpisania poniższego polecenia:

    $ split -b 100M -d -a 3 --verbose film.mkv "film.mkv."
    creating file 'film.mkv.000'
    creating file 'film.mkv.001'
    ...
    creating file 'film.mkv.018'
    creating file 'film.mkv.019'

Użyte opcje:

  - `-b 100M` odpowiada za rozmiar, po którym ma nastąpić podział pliku
  - `-d` zmienia domyślny sufix na cyferki
  - `-a 3` definiuje ile pozycji ma być zarezerwowanych na sufix
  - `--verbose` aktywuje tryb rozmowny
  - `film.mkv` określa plik wejściowy, który zostanie poddany cięciu
  - `"film.mkv."` odpowiada za prefix, zwykle taki sam jak nazwa ciętego pliku

Zajrzyjmy zatem do katalogu, w którym dokonało się dzielenie pliku:

    $ ls -al film.*
    -rw-r--r-- 1 morfik morfik 100M 2015-06-19 11:43:01 film.mkv.000
    -rw-r--r-- 1 morfik morfik 100M 2015-06-19 11:43:05 film.mkv.001
    ...
    -rw-r--r-- 1 morfik morfik 100M 2015-06-19 11:44:14 film.mkv.018
    -rw-r--r-- 1 morfik morfik  54M 2015-06-19 11:44:17 film.mkv.019

Jako, że ostatnia część nie osiągnęła rozmiaru 100M, oznacza to koniec pliku.

## Łączenie części pliku w całość

Mając plik w częściach, wszystko jedno, czy pocięty przy pomocy `split` , czy pobrany z internetu,
raczej chcielibyśmy połączyć poszczególne kawałki tak by otrzymać cały plik. I w tym przypadku
również nie musimy odwoływać się do jakichś zaawansowanych technik i wystarczy nam do tego celu
narzędzie `cat` , które, podobnie jak poprzednik, znajduje się w pakiecie `coreutils` . Także jak
widać, nawet nie musimy instalować dodatkowych pakietów, bo wszystkie potrzebne narzędzia mamy już
obecne w systemie.

Pliki łączymy zaś w poniższy sposób:

    $ cat film.mkv.0* > film.mkv

Po całym procesie dobrze jest także sprawdzić jak wygląda [suma
kontrolna]({{< baseurl >}}/post/suma-kontrolna-nagranego-obrazu-iso/) pliku wynikowego, oczywiście
w przypadku gdybyśmy dysponowali sumą oryginalnego pliku. A to z tego względu, że narzędzie `cat`
nie weryfikuje wejścia i zwyczajnie kopiuje poszczególne pliki i łączy je ze sobą. Jeśli któryś z
nich jest uszkodzony albo zwyczajnie go brakuje, wtedy wyjściowy plik również ulegnie uszkodzeniu i
nawet nie zostaniemy o tym powiadomieni.
