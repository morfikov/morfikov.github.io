---
author: Morfik
categories:
- Linux
date: "2015-07-15T18:16:18Z"
date_gmt: 2015-07-15 16:16:18 +0200
published: true
status: publish
tags:
- tty
- locale
- czcionki
GHissueID: 159
title: Polskie znaki pod TTY
---

Jeśli w środowisku graficznym mamy ustawiony [polski
język](/post/jezyk-polski-w-srodowisku-graficznym/), nie mamy przy tym problemów z
kodowaniem znaków w tekście i nasza klawiatura ma ustawiony odpowiedni [układ
klawiszy](/post/klawiatura-i-jej-konfiguracja-pod-debianem/) ale jednocześnie
doświadczamy problemów jeśli chodzi o polskie znaki pod TTY, oznacza to prawdopodobnie źle
skonfigurowany wirtualny terminal. Generalnie rzecz biorąc środowisko graficzne i konsola TTY, to
tak jakby dwa różne światy i trzeba je konfigurować w pewnych aspektach osobno.

<!--more-->
## Polskie znaki czy locale?

Mamy z grubsza dwa przypadki do analizy jeśli chodzi o polskie znaki. Pierwszy dotyczy sytuacji
gdzie znak nie jest w ogóle pokazywany np. po wciśnięciu alt + L i tu mamy do poprawienia
lokalizację. Natomiast jeśli po przyciśnięciu powyższej kombinacji klawiszy zostanie coś napisane
na ekranie, z tym, że jesteśmy tego w stanie odczytać, to problem dotyczy złego kodowania lub
nieodpowiedniej czcionki, która nie jest w stanie wydrukować polskich znaków, bo zwyczajnie ich nie
posiada. Rozwiązaniem zatem jest przestawienie kodowania lub/i zmiana czcionki.

### Konfiguracja wirtualnego terminalu

Za tego tupu manewry pod TTY odpowiada pakiet `console-setup` . Jeśli nie mamy go w systemie, to
doinstalujmy. Podobnie jak w większości niskopoziomowych narzędzi konfiguracyjnych w debianie,
możemy skorzystać z `dpkg-reconfigure` do konfiguracji pakietu `console-setup` . Odpalamy zatem
terminal, logujemy się na konto użytkownika root i wpisujemy poniższe polecenie:

    # dpkg-reconfigure console-setup

Po chwili powinno nam się pokazać okienko z wyborem kodowania znaków. Oczywiście zaznaczamy
`UTF-8` :

![](/img/2015/06/1.linux-polskie-znaki-tty.png#big)

Następnie wybieramy zestaw znaków, który powinien być wspierany przez czcionkę konsoli TTY i tu
musimy wybrać `Latin2`:

![](/img/2015/06/2.linux-polskie-znaki-tty.png#big)

Następnie wybieramy czcionkę i ja tutaj zawsze ustawiam sobie `Terminus` , bo nie męczy tak
strasznie oczu:

![](/img/2015/06/3.linux-polskie-znaki-tty.png#big)

Ostatnią opcją jest dostosowanie wielkości czcionek. Jeśli odpowiada nam wielkość tych, które
widzimy przy starcie systemu, to zaznaczamy `8x16` , jeśli natomiast są zbyt małe, to zawsze możemy
sobie zwiększyć:

![](/img/2015/06/4.linux-polskie-znaki-tty.png#big)

Po tym jak konfiguracja pakietu dobiegnie końca, zostanie wygenerowany plik
`/etc/default/console-setup` :

    ACTIVE_CONSOLES="/dev/tty[1-6]"

    CHARMAP="UTF-8"

    CODESET="Lat2"
    FONTFACE="Terminus"
    FONTSIZE="8x16"

    VIDEOMODE=

I to w sumie cała robota z naszej strony. Po zresetowaniu systemu, polskie znaki na konsoli TTY
powinny być już rozpoznawane bez problemu.
