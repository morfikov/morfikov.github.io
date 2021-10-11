---
author: Morfik
categories:
- Linux
date: "2016-07-10T19:14:31Z"
date_gmt: 2016-07-10 17:14:31 +0200
published: true
status: publish
tags:
- bash
- zsh
GHissueID: 386
title: Jak zweryfikować status poleceń w pipe
---

Ja zbytnio się nie nadaję na programistę ale czasem jakieś trzeciorzędne skrypty nawet potrafię
napisać. Problem ze skryptami jest taki, że mogą one nie do końca działać jak należy. Zwykle w
takich przypadkach, skrypt zwraca jakiś kod wyjścia. Z reguły też jest on inny od 0, który z kolei
oznacza, że skrypt został wykonany prawidłowo. Załóżmy teraz, że wywołujemy szereg poleceń. Każde z
nich przekierowuje swoje wyjście na wejście innego polecenia przy pomocy znaku `|` (pipe) . Jak w
takim przypadku ustalić czy wszystkie z tych poleceń w łańcuchu wykonały się poprawnie? W tym wpisie
postaramy się odpowiedzieć na to pytanie.

<!--more-->
## Sprawdzenie statusu przykładowego polecenia

Każde wykonane polecenie zwraca jakiś kod wyjścia. Ten kod można odczytać przy pomocy `$?` . Odpalmy
sobie zatem terminal i wpiszmy w nim jakieś polecenie, które się wykona z powodzeniem:

    $ echo "test" > /dev/null
    
    $ echo $?
    0

Dla porównania wpiszmy także polecenie, które zwróci jakiś błąd:

    $ echo "test" > /etc/hosts
    bash: permission denied: /etc/hosts
    
    $ echo $?
    1

W ten sposób można łatwo określić, czy dane polecenie wykonało się poprawnie. Niemniej jednak, nie
możemy z tego sposobu skorzystać bezpośrednio w przypadku pipe.

## Sprawdzenie kodu wyjścia poleceń w pipe

Weźmy sobie przykład skryptu, w którym mamy szereg poleceń wywoływanych w opisany we wstępie sposób.
Dla przykładu niech to będzie ten poniższy kawałek kodu:

    #!/bin/bash
    ...
    cat /etc/peerblock/$set.gz |
    gunzip |
    cut -d: -f2 |
    grep -E "^[-0-9.]+$" |
    gawk -v my_set=$set '{print "add " my_set " " $1}' |
    $ips --restore -exist;

Sam skrypt został nieco skrócony dla czytelności. Widzimy wyżej 5 znaków `|` . To są właśnie nasze
pipe, przy pomocy których kierujemy wyjście jednego polecenia na wejście kolejnego polecenia.
Wszystkie polecenia w tym łańcuchu są traktowane indywidualnie, tj. każde z nich ma własny kod
wyjścia. Gdybyśmy teraz chcieli sprawdzić przy pomocy `$?` status tego całego łańcucha, to zostanie
nam zwrócony tylko kod ostatniego polecenia. W ten sposób nie dowiemy się czy, np. drugie polecenie
zakończyło się sukcesem i nie zwróciło żadnych błędów. Oczywiście można posiłkować się komunikatami
zwracanymi na konsoli ale w przypadku skryptów, gdzie jedne polecenia są oparte o inne, nie zawsze
nam takie wiadomości mogą pomóc.

### Opcja pipefail

W przypadku shell'ów `bash` i `zsh` mamy dość ułatwione zadanie [sprawdzenia kodu wyjścia wszystkich
poleceń w
pipe](https://unix.stackexchange.com/questions/14270/get-exit-status-of-process-thats-piped-to-another).
Możemy w tych shell'ach ustawić [opcję
pipefail](http://www.gnu.org/software/bash/manual/html_node/Pipelines.html), która zwróci status 0 w
przypadku, gdy żadne z poleceń w pipe nie zwróciło błędów. Uzupełnijmy zatem powyższy skrypt o w/w
opcję i postarajmy się zweryfikować status wyjścia całego polecenia:

    #!/bin/bash
    ...
    set -o pipefail
    
    cat /etc/peerblock/$set.gz |
    gunzip |
    cut -d: -f2 |
    grep -E "^[-0-9.]+$" |
    gawk -v my_set=$set '{print "add " my_set " " $1}' |
    $ips --restore -exist;
    
    if [ "$?" -ne "0" ]; then
        exit 1
    fi

### Zmienne $PIPESTATUS oraz $pipestatus

Istnieje jeszcze inna metoda na sprawdzenie status poleceń w pipe. Shell `bash` i `zsh` zachowują
status wyjścia ostatniego pipe w zmiennych, odpowiednio `$PIPESTATUS` oraz `$pipestatus` . W ten
sposób można ustalić, które z poleceń zawiodło. Poniżej przykład:

    $ false | true | true | true | false; echo "${PIPESTATUS[@]}"
    1 0 0 0 1

Dla jeszcze większego uproszenia i czytelności korzystamy z poleceń `true` i `false` , które
zwracają odpowiedni status wyjścia. Widzimy, że tam, gdzie jest `false` , to mamy 1. Jeśli teraz
interesowałoby nas trzecie polecenie, to możemy sprawdzić jego status w poniższy sposób:

    $ false | true | true | true |false; echo "${PIPESTATUS[2]}"
    0

Polecenia są numerowane od 0, dlatego w `${PIPESTATUS[2]}` wstawiliśmy 2.
