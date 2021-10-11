---
author: Morfik
categories:
- Linux
date: "2015-11-07T15:23:38Z"
date_gmt: 2015-11-07 14:23:38 +0100
published: true
status: publish
tags:
- bash
GHissueID: 256
title: Plik .bashrc, czyli konfiguracja bash'a
---

Jakiś czas temu opisywałem [konfigurację historii bash'a w pliku .bash_history][1] ale możliwości
konfiguracyjne bash'a nie ograniczają się jedynie do zmiany kilku parametrów czy zmiennych
dotyczących historii wpisywanych w terminalu poleceń. Ten wpis ma na celu zebranie tych bardziej
użytecznych funkcjonalności bash'a, które często są wykorzystywane przez użytkowników linux'a i
dopisywane w pliku `.bashrc` .

<!--more-->
## Pliki .bash_history, .bash_aliases, .bashrc, .bash_profile i .bash_logout

W katalogu użytkownika mamy kilka plików, które składają się na konfigurację shell'a bash. W pliku
`.bash_history` jest trzymana historia wykonywanych poleceń. Wszystkie parametry dotyczące historii
poleceń zostały opisane w osobnym artykule (link we wstępie) i nie będę ich tutaj przytaczał. Dalej
mamy do dyspozycji plik `.bash_aliases` , w którym dla wygody można umieszczać aliasy. By móc
korzystać z tego pliku, musimy umieścić ten poniższy kod w pliku konfiguracyjnym bash'a:

    if [ -f ~/.bash_aliases ]; then
        . ~/.bash_aliases
    fi

Z kolei plik `.bashrc` zawiera główną konfigurację. Wszystkie te poniższe opcje będziemy umieszczać
w tym właśnie pliku. Dodatkowo, mamy też plik `.bash_profile` . W nim są tylko te dwa poniższe
warunki:

    [[ -f ~/.bashrc ]] && . ~/.bashrc
    [[ -f ~/.profile ]] && . ~/.profile

Jak widzimy wyżej, najpierw jest ładowana główna konfiguracja bash'a, a po niej jeszcze plik
`.profle` . Jaki jest zatem cel pliku `.bash_profile` i czy nie możemy korzystać jedynie z
`.bashrc` ? Bash może zostać wywołany [interaktywnie lub nieinteraktywnie][2]. Różnica między tymi
dwoma sposobami polega na tym, że ten drugi dotyczy głównie sesji logowania, np. na konsoli TTY. Z
kolei sesje interaktywne są powiązane z pseudoterminalami, gdzie nie ma potrzeby się logować, np. w
środowisku graficznym. Zatem w każdym z tych powyższych przypadków zostanie załadowany plik
`.bashrc` , natomiast w trybie tekstowym (TTY) zostanie także załadowana konfiguracja z pliku
`.profile` . To rozgraniczenie jest bardzo użyteczne ze względu na fakt, że część konfiguracji
bash'a może działać znakomicie w środowisku graficznym ale powodować błędy w trybie tekstowym.

Ostatnim plikiem jest `.bash_logout` , w którym są określone polecenia jakie mają zostać wykonane,
gdy w konsoli zostanie wpisane `exit` lub `logout` . Ten plik jest ładowany tylko w trybie
nieinteraktywnym, [głównie na konsolach TTY][3].

Jako, że wszystkie poniższe opcje będą ustawiane za pomocą `shopt` , to aktualną konfigurację możemy
podejrzeć wpisując w terminalu to właśnie polecenie. Te opcje, które zostaną wpisane do pliku
`.bashrc` , będą mieć efekt permanentny. Dlatego też dobrze jest je pierw przetestować. Wszystkie
możliwe opcje są opisane w [manualu bash'a][4].

### Sprawdzanie parametrów okna przy wykonywaniu polecenia (checkwinsize)

Często zmieniamy rozmiar okna terminala, a to powoduje zmianę wartości aktualnych wierszy i kolumn,
odpowiednio zmiennych `$LINES` i `$COLUMNS` . Szereg aplikacji, np. `dpkg` polegają na tych
parametrach. Jeśli zmienimy rozmiar okna, to te dwie powyższe zmienne nie są aktualizowane, wobec
czego wyjście takich programów może być lekko nieczytelne. Przy pomocy parametru `checkwinsize`
możemy nakazać bash'owi by aktualizował zmienne `$LINES` i `$COLUMNS` przed wywołaniem polecenia.
Dopisujemy zatem do pliku `.bashrc` tę linijkę:

    shopt -s checkwinsize

### Korekta ścieżek w cd (cdspell)

Bash jest w stanie również poprawiać błędy w nazwach ścieżek przy wywoływaniu polecenia `cd` , przez
co może nam zaoszczędzić nieco czasu. Jak taki mechanizm ma działać? Załóżmy, że chcemy przejść do
katalogu `/etc/` ale przez przypadek zrobiliśmy literówkę i wpisaliśmy w terminalu `cd /ect` . Bez
korekty, dostalibyśmy błąd, natomiast po aktywowaniu tej opcji, bash nas powiadomi o fakcie
dokonania poprawki i przejdzie do katalogu `/etc/` . My zaś nie będziemy musieli ponownie wprowadzać
polecenia `cd` i poprawiać błędu. By aktywować tę funkcję bash'a, dodajemy do pliku `.bashrc` tę
poniższą linijkę:

    shopt -s cdspell

### Przejściowe zmienne (cdable_vars)

Ścieżki do katalogów mogą być naprawdę długie, a przechodzenie do nich (nawet z uzupełnianiem przy
pomocy klawisza TAB ) powolne. Każdą z tych długich ścieżek możemy zamknąć w zmiennej. Problem w
tym, że gdy wydamy polecenie `cd $zmienna` , to nie znajdziemy się w katalogu, który pod tą zmienną
się znajduje. Możemy jednak zmienić to zachowanie bash'a i dopisać w pliku `.bashrc` parametr
`cdable_vars` :

    shopt -s cdable_vars

Trzeba mieć jednak na uwadze, że znak `~` , który symbolizuje katalog domowy użytkowników, nie
zadziała z tą opcją i trzeba precyzować pełne ścieżki, tj. zaczynając od `/home/morfik/` .


[1]: /post/plik-bash_history-czyli-historia-polecen-basha/
[2]: https://unix.stackexchange.com/questions/38175/difference-between-login-shell-and-non-login-shell
[3]: https://superuser.com/questions/410525/explain-why-bash-logout-wont-run-commands
[4]: http://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html
