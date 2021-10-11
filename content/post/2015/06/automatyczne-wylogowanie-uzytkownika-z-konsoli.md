---
author: Morfik
categories:
- Linux
date: "2015-06-17T22:06:51Z"
date_gmt: 2015-06-17 20:06:51 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- bash
GHissueID: 134
title: Automatyczne wylogowanie użytkownika z konsoli
---

Każdemu z nas zdarzyło się zostawić włączoną konsolę, na której byliśmy zalogowani jako
administrator systemu root. Być może nie zdajemy sobie sprawy jak często potrafimy popełnić tego
typu gafę. Jedną z metod obrony jest oczywiście wyłączenie konta root w systemie i korzystanie z
[sudo](https://pl.wikipedia.org/wiki/Sudo). Ja jednak wolę inne rozwiązanie, które zakłada
ograniczenie czasu bezczynności, po którym to użytkownik zostanie automatycznie wylogowany.

<!--more-->
## Parametr TMOUT

W zależności od wykorzystywanych powłok systemowych, konfiguracja automatycznego wylogowania może
przebiegać nieco inaczej. W przypadku `bash` i `zsh` akurat jest do dyspozycji ten sam parametr.
Mowa o `TMOUT` :

> `TMOUT` If set to a value greater than zero, `TMOUT` is treated as the default timeout for the
> read builtin. The select command terminates if input does not arrive after `TMOUT` seconds when
> input is coming from a terminal. In an interactive shell, the value is interpreted as the number
> of seconds to wait for a line of input after issuing the primary prompt. Bash terminates after
> waiting for that number of seconds if a complete line of input does not arrive.
> 
> [man bash](http://manpages.ubuntu.com/manpages/xenial/en/man1/bash.1.html)

Wyżej możemy przeczytać, że jeśli w czasie określonym przez parametr `TMOUT` (wyrażonym w sekundach)
nie zostanie wydane żadne polecenie w terminalu, nastąpi automatyczne wylogowanie.

By przekonać się jak ten mechanizm działa, odpalmy terminal i wydajmy w nim to poniższe polecenie:

    $ export TMOUT=10

Po upływie 10 sekund od wpisania go, terminal powinien się zamknąć sam z siebie. Jeśli chcielibyśmy
wprowadzić taką politykę w stosunku do wszystkich kont w systemie, to możemy dopisać powyższe
polecenie do pliku `/etc/profile` , choć moim zdaniem lepiej jest dać wolną rękę użytkownikom w
systemie i niech oni sobie ten parametr dostosują via plik `~/.bashrc` . A to z tego względu, że
jeśli mamy dostęp do graficznej sesji to i tak można bez problemu terminal z prawami określonego
użytkownika wywołać, no i oczywiście jeśli korzystamy z jakiegoś multipleksera, np.
[tmux](http://tmux.github.io/) , to ustawienie zmiennej `TMOUT` ubije wszystkie sesje. Podobnie
sprawa ma się w przypadku posiadania konsoli na pulpicie.

## Automatyczne wylogowanie z konta root

Jedyne rozsądne miejsce gdzie należałoby dodać ten parametr, to plik `~/.bashrc` użytkownika root.
To konto nie powinno być pozostawione bez nadzoru przez jakiś dłuższy czas. Ja tutaj zwykle ustawiam
sobie 10 minut:

    export TMOUT=600
