---
author: Morfik
categories:
- Linux
date: "2015-11-09T14:57:01Z"
date_gmt: 2015-11-09 13:57:01 +0100
published: true
status: publish
tags:
- tty
- bash
GHissueID: 258
title: Kolorowanie wyjścia terminala
---

Każdy terminal jest w stanie wyświetlić tekst w kilku kolorach. Zwykle mamy ich do dyspozycji 8
lub 16. Niektóre terminale potrafią rozróżniać nawet 256 kolorów. Niemniej jednak, kolor całego
tekstu jaki jest wyświetlany w terminalu jest zwykle jednolity i nie ma w nim praktycznie żadnych
urozmaiceń. W taki sposób mamy czarny tekst i białe tło. Jako, że te terminale są w stanie
wyświetlić więcej kolorów, to dla większej czytelności przydałoby się skonfigurować kolorowanie
wyjścia takich narzędzi jak `ls` , `grep` czy `man` . Jesteśmy w stanie pokolorować także szereg
innych rzeczy i o tym będzie ten poniższy wpis.

<!--more-->
## Kolory i formatowanie

Kolorowanie wyjścia w terminalu możemy przeprowadzić przy pomocy [aliasów
poleceń](https://pl.wikipedia.org/wiki/Alias_%28Unix%29), które zawierają odpowiednie opcje. W
linux'ie są też takie komendy, które korzystają z wartości ustawionych w zmiennych środowiskowych.
Tak czy inaczej, kolory zależą w dużej mierze od możliwości i konfiguracji samego terminala i
niekoniecznie muszą być one takie same na kilku stacjach roboczych. Jeśli jesteśmy ciekawi jakie
kolory ma ustawiony nasz terminal, to możemy się posłużyć się tym poniższym skryptem lub też
skorzystać z tego dostepnego [tutaj](http://misc.flogisoft.com/bash/tip_colors_and_formatting):

    #!/bin/bash
    esc="33\["
    echo -n " \_ \_ \_ \_ \_40 \_ \_ \_ 41\_ \_ \_ \_42 \_ \_ \_ 43"
    echo "\_ \_ \_ 44\_ \_ \_\_45 \_ \_ \_ 46\_ \_ \_ \_47 \_"
    for fore in 30 31 32 33 34 35 36 37; do
        line1="$fore "
        line2=" "
        for back in 40 41 42 43 44 45 46 47; do
            line1="${line1}${esc}${back};${fore}m Normal ${esc}0m"
            line2="${line2}${esc}${back};${fore};1m Bold ${esc}0m"
        done
        echo -e "$line1\n$line2"
    done

Po uruchomieniu tego skryptu, powinniśmy zobaczyć konfigurację terminala odpowiedzialną za
wykorzystywane kolory:

![kolorowanie-wyjscia-terminal-kolory](/img/2015/11/1.kolorowanie-wyjscia-terminal-kolory.png#big)

## Kolorowanie wyjścia ls

Wyjście narzędzia `ls` jest dużo bardziej czytelne, gdy posiada kolory. W taki sposób jesteśmy w
stanie odróżniać od siebie szereg pozycji, np. pliki od katalogów, czy też poszczególne pliki od
siebie. Wygląda to mniej więcej tak:

![kolorowanie-wyjscia-ls](/img/2015/11/2.kolorowanie-wyjscia-ls.png#huge)

By pokolorować wyjście `ls` , musimy stworzyć dla niego alias zawierający parametr `--color=auto` i
umieścić go w pliku `.bashrc` , przykładowo:

    if [ -x /usr/bin/dircolors ]; then
        test -r ~/.dir_colors && eval "$(dircolors -b ~/.dir_colors)" || eval "$(dircolors -b)"
        alias ls='ls -Fh --group-directories-first --color=auto --time-style="+%Y-%m-%d %H:%M:%S"'
    fi

### Kilka słów o dircolors

Wyżej mieliśmy linijkę zawierającą `dircolors` . Zgodnie z tym co możemy wyczytać w [man
dir_colors](http://manpages.ubuntu.com/manpages/wily/en/man5/dir_colors.5.html), ma ona na celu
ustawienie zmiennej `LS_COLORS` . To w oparciu o tę zmienną będą kolorowane rozszerzenia konkretnych
plików. By móc z tego ficzera skorzystać, musimy mieć zainstalowany w systemie pakiet `coreutils` ,
który zawiera narzędzie `dircolors` . Konfigurację zmiennej `LS_COLORS` możemy ustawić w pliku
`/etc/DIR_COLORS` lub też `~/.dir_colors` .

Możemy także zaimplementować sobie
[dircolors-solarized](https://github.com/seebi/dircolors-solarized). W zależności od tego czy mamy
terminal 16 czy 256 kolorowy, instalujemy odpowiedni plik. W tym przypadku jest to
`dircolors.256dark` i jego treść wrzucamy do pliku `.dir_colors` .

## Kolorowanie wyjścia grep

Innym narzędziem, którego wyjście staje się nieco bardziej przejrzyste, jest `grep` . W jego
przypadku również musimy stworzyć alias i dodać go do pliku `.bashrc` :

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'

Jesteśmy także w stanie zdefiniować kolor, jakim `grep` będzie wyróżniał pasujące frazy. Standardowo
jest to kolor czerwony. Jeśli chcemy by był to, np. zielony, to dopisujemy tę poniższą zmienną do
pliku `.bashrc` :

    GREP_COLOR='1;32'

## Kolorowanie wyjścia man

Można również pokolorować manuale. Chodzi o te dostępne po wpisaniu w terminalu polecenia `man` .
Nie będzie tego, co prawda, dużo ale zawsze łatwiej odnaleźć pewne rzeczy gdy się wyróżniają z
tłumu. Efekt będzie mniej więcej taki:

![kolorowanie-wyjscia-man](/img/2015/11/3.kolorowanie-wyjscia-man.png#huge)

Jeśli nam odpowiada taki format dokumentacji (kolory można dostosować), to dopiszmy ten poniższy
blok kodu do pliku `.bashrc` :

    LESS_TERMCAP_mb=$'\E[01;31m' # begin blinking
    LESS_TERMCAP_md=$'\E[01;31m' # begin bold
    LESS_TERMCAP_me=$'\E[0m' # end mode
    LESS_TERMCAP_se=$'\E[0m' # end standout-mode
    LESS_TERMCAP_so=$'\E[01;44;33m' # begin standout-mode - info box
    LESS_TERMCAP_ue=$'\E[0m' # end underline
    LESS_TERMCAP_us=$'\E[01;32m' # begin underline
    #MANPAGER="/usr/bin/most -s" # color using most

## Kolorowanie wyjścia kompilatora GCC

W przypadku, gdy zdarza nam się kompilować programy, to na terminalu jest wyrzucana bardzo duża
ilość informacji. Część z tych komunikatów może zawierać ostrzeżenia, a nawet i błędy. Jeśli ten
tekst jest jednokolorowy, to możemy je zwyczajnie przegapić. Przydałoby się zatem [pokolorować
wyjście kompilatora GCC](https://insanecoding.blogspot.fr/2014/04/gcc-49-diagnostics.html). Możemy
to zrobić przez wyeksportowanie zmiennej `GCC_COLORS` w pliku `.bashrc` . Poniżej przykład:

    export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

## Kolorowanie wyjścia konsoli TTY

Konsola TTY dysponuje jedynie 16 kolorami. Jeśli w środowisku graficznym mamy terminal 256 kolorowy
i korzystamy przy tym z dircolors-solarized, to jest niemal pewne, że kolory będą strasznie
kontrastować na tym czarny tyle. Kolorowanie wyjścia konsoli TTY możemy przeprowadzić osobno, co nie
będzie mieć żadnego wpływu na graficzne terminale. W tym celu do pliku `.bashrc` dodajemy ten
poniższy blok kodu:

    if [ "$TERM" = "linux" ]; then
        echo -en "\e]P0232323"
        echo -en "\e]P1a40000"
        echo -en "\e]P24E9A06"
        echo -en "\e]P3C4A000"
        echo -en "\e]P43465A4"
        echo -en "\e]P575507B"
        echo -en "\e]P6ce5c00"
        echo -en "\e]P7babdb9"
        echo -en "\e]P7d75f00"
        echo -en "\e]P8555753"
        echo -en "\e]P9EF2929"
        echo -en "\e]PA8AE234"
        echo -en "\e]PBffcc00"
        echo -en "\e]PC729FCF"
        echo -en "\e]PDAD7FA8"
        echo -en "\e]PEfcaf3e"
        echo -en "\e]PFEEEEEC"
        echo -en "\e]PFd75f00"
        clear #for background artifacting
    fi

Ten numerek co się kryje za `P0232323` składa się tak naprawdę z dwóch części: `P0` oraz `232323` .
Pierwsza część określa kolory od 0 do 15, razem 16. Z z kolei druga część to kolor RGB w zapisie
hexalnym: `23` , `23` i `23` . Wartości mogą się zawierać w przedziale od 00 do FF.
