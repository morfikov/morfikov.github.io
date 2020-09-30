---
author: Morfik
categories:
- Linux
date: "2015-11-07T16:56:11Z"
date_gmt: 2015-11-07 15:56:11 +0100
published: true
status: publish
tags:
- bash
title: Uzupełnianie poleceń w bash (bash completion)
---

Bash nie nadaje się dla nieco bardziej zaawansowanych użytkowników linux'a. Najbardziej odczuwalnym
elementem bash'a jest brak uzupełniania poleceń za pomocą klawisza Tab . Nie mówimy tutaj o
przeszukiwaniu zmiennej `$PATH` pod kątem dopasowań pliku wykonywalnego do tego co wpisujemy
aktualnie w terminalu. Nie chodzi też o uzupełnianiu ścieżek podawanych do `cd` ale o opcje, które
jakiś program może przyjąć jako argument. Zwykle musimy się uczyć ich na pamięć lub zaglądać do
help'a czy manuala. Możliwe jest jednak skorzystanie z dodatku zwanego [bash
completion](https://en.wikipedia.org/wiki/Command-line_completion), który w sporej części przypadków
potrafi dostarczyć dość zaawansowane uzupełnianie poleceń, którymi się posługujemy na co dzień. Ten
wpis ma na celu pokazanie jak włączyć ten cały mechanizm i uprościć sobie nieco życie podczas pracy
w terminalu.

<!--more-->
## Parę słów o bash completion

Obecnie chyba niemal każdy shell ma wbudowane uzupełnienie poleceń, a to z tego względu, że ta
funkcjonalność drastycznie poprawia szybkość i płynność operacji przeprowadzanych w trybie
tekstowym. Niemniej jednak, te zdolności są dość mocno ograniczone w przypadku domyślnego shell'a w
debianie, jakim jest bash. Dlatego też powstał [projekt bash
completion](http://bash-completion.alioth.debian.org/#links) , który ma na celu znaczne rozbudowanie
możliwości tego shell'a jeśli chodzi o uzupełnianie poleceń.

Generalnie rzecz biorąc, mechanizm uzupełniania sprowadza się do przyciśnięcia klawisza `Tab`
dwukrotnie w momencie wpisywania jakiegoś polecenia do terminala. Jeśli wpiszemy kilka znaków, po
czym wciśniemy `Tab` , to zostanie nam zwróconych szereg pozycji pasujących do tej wpisanej frazy.
Jeśli chodzi o nieco bardziej zaawansowane uzupełnianie poleceń, to po wpisaniu nazwy polecenia i
przyciśnięciu klawisza `Tab` , zostanie nam zwrócony szereg opcji, jakie możemy dopisać w argumencie
dla tego polecenia. Wygląda to mniej więcej tak:

![](/img/2015/11/1.bash-completion-uzupelnianie-nazw-polecen.png#huge)

Mało tego, po określeniu parametru, możemy znów przycisnąć klawisz `Tab` dwa razy i w ten sposób
będziemy wiedzieć jakiego formatu danych oczekuje konkretna opcja tego polecenia. W powyższym
przypadku, po wybraniu opcji `luksFormat` i przyciśnięciu `Tab` , automatycznie pojawił się katalog
`/dev/` sugerujący by wybrać jakiś urządzenie, co wydaje się logiczne.

## Jak włączyć uzupełnianie poleceń

By móc skorzystać z tego wynalazku uzupełniającego polecenia przy pomocy klawisza Tab , musimy
posiadać zainstalowany w systemie pakiet `bash-completion` . Musimy także edytować plik `.bashrc` ,
który znajduje się w katalogu domowym użytkownika. Otwieramy zatem wspomniany plik i dodajemy do
niego ten poniższy blok kodu:

    if ! shopt -oq posix; then
          if [ -f /usr/share/bash-completion/bash_completion ]; then
                . /usr/share/bash-completion/bash_completion
          elif [ -f /etc/bash_completion ]; then
                . /etc/bash_completion
          fi
    fi

I to w zasadzie cała robota. Zmiany nie są natychmiastowe i wymagają przeładowania konfiguracji przy
pomocy tego poniższego polecenia:

    $ source ~/.bashrc
