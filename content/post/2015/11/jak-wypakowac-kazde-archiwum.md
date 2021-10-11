---
author: Morfik
categories:
- Linux
date: "2015-11-20T15:15:30Z"
date_gmt: 2015-11-20 14:15:30 +0100
published: true
status: publish
tags:
- bash
- archiwa
GHissueID: 253
title: Jak wypakować każde archiwum
---

Archiwum `.tar.gz` czy też każde inne, np. `.zip` lub `.rar` , jest bardzo użyteczne przy
przesyłaniu przez sieć plików między wieloma maszynami. Nie dość, że zaoszczędzają nam sporo
transferu, to jeszcze do tego operujemy na jednym pliku. Problem z tymi wszystkimi rodzajami
archiwów jest taki, że do każdego z nich mamy inne narzędzia. Zwykle też każde z tych narzędzi ma
inne opcje, które taką paczkę potrafią wypakować. Przydałby się zatem sposób, który ogarnąłby
wypakowanie różnych archiwów i sprowadzałby się do wydania w terminalu tylko jednego polecenia, bez
potrzeby pamiętania nazw narzędzi i ich opcji. W tym wpisie postaramy się coś takiego
zaimplementować na swoich linux'ach.

<!--more-->
## Typ archiwum i programy archiwizujące

Na początek zróbmy sobie krótki przegląd wszystkich narzędzi archiwizujących (tych tekstowych),
które są dostępne w debianie. Poniżej znajduje się krótka lista:

  - pakiet `tar` -- pliki `.tar.gz` , `.tar.bz2` , `.tar` , `.tbz2` , `.tgz` .
  - pakiet `bzip2` -- pliki `.bz2` .
  - pakiet `unrar` -- pliki `.rar` .
  - pakiet `gzip` -- pliki `.gz` , `.Z` .
  - pakiet `unzip` -- pliki `.zip` .
  - pakiet `p7zip-full` -- pliki `.7z` .

Nie wiem czy brakuje jeszcze jakichś archiwów, w każdym razie, na te powyższe się ciągle natykam
podczas pracy w swoim systemie.

## Wypakowanie archiwum

Mając rozszerzenia plików oraz odpowiadające im narzędzia, które znajdują się w tych powyższych
pakietach, możemy napisać sobie prosty alias polecenia i umieścić go w pliku `~/.bashrc` . Jak ktoś
woli, to zawsze może z tego poniższego kodu zrobić sobie skrypt:

    extract() {
        if [ -f $1 ] ; then
            case $1 in
                *.tar.bz2)   tar xvjf $1     ;;
                *.tar.gz)    tar xvzf $1     ;;
                *.bz2)       bunzip2 $1      ;;
                *.rar)       unrar x $1      ;;
                *.gz)        gunzip $1       ;;
                *.tar)       tar xvf $1      ;;
                *.tbz2)      tar xvjf $1     ;;
                *.tgz)       tar xvzf $1     ;;
                *.zip)       unzip $1        ;;
                *.Z)         uncompress $1   ;;
                *.7z)        7z x $1         ;;
                *)           echo "'$1' cannot be extracted via >extract<" ;;
            esac
        else
            echo "'$1' is not a valid file!"
        fi
    }

Po przeładowaniu konfiguracji shella via `source ~/.bashrc` , polecenie `extract` zostanie oddane
nam do użytku. By wypakować jakieś archiwum za jego pomocą, wystarczy jako argument podać ścieżkę do
pliku.

Istnieje także narzędzie zwane `patool` (w debianie dostępne w pakiecie pod tą samą nazwą), które w
dużej mierze robi to co powyższy alias. Obsługuje też więcej typów plików, no i oczywiście
automatycznie pociąga odpowiednie zależności. Więcej informacji o tym narzędziu można znaleźć w [man
patool](http://manpages.ubuntu.com/manpages/wily/man1/patool.1.html).
