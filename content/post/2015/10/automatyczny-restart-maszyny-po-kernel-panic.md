---
author: Morfik
categories:
- Linux
date: "2015-10-28T23:56:51Z"
date_gmt: 2015-10-28 21:56:51 +0100
published: true
status: publish
tags:
- sysctl
- kernel
title: Automatyczny restart maszyny po kernel panic
---

Gdy nasz linux napotka z jakiegoś powodu błąd wewnątrz swojej struktury, to istnieją sytuacje, w
których obsługa tego błędu czasem nie jest możliwa. Wobec czego, zostaje wyrzucony komunikat
systemowy oznajmiający nam, że [kernel spanikował (kernel
panic)](https://pl.wikipedia.org/wiki/Kernel_panic), bo nie wie co w takim przypadku zrobić. Gdy
tego typu sytuacja się nam przytrafia, nie ma innego wyjścia jak tylko uruchomić system ponownie. Co
jednak w przypadku gdy pracujemy zdalnie i nie jesteśmy w stanie zresetować takiej maszyny
fizycznie? Na szczęście kernel ma kilka opcji, które mogą zainicjować automatyczny restart w
przypadku wystąpienia kernel panic.

<!--more-->
## Parametr kernel.panic

Pierwszym parametrem, na który warto rzucić okiem, jest `kernel.panic` . Określa on ilość czasu (w
sekundach), po którym system zostanie uruchomiony ponownie w przypadku wystąpienia kernel panic.
Jeśli ten parametr będzie miał przypisaną wartość `0`, restart zostanie wyłączony całkowicie. W
takim przypadku maszynę trzeba będzie zresetować ręcznie. Trzeba jednak pamiętać o jednej istotnej
rzeczy. W przypadku błędów, które zdarzają się raz lub bardzo rzadko, pożądane jest ustawienie w tym
parametrze innej wartości niż 0. Natomiast jeśli błąd jest powodowany przez jakiś czynnik, którego
nie potrafimy wyeliminować, to system może się resetować w kółko. Tak czy inaczej, te błędy zdarzają
się niezmiernie rzadko i dobrze jest wskazać kernelowi, by ten zresetował system, np. po 60
sekundach od wystąpienia kernel panic. By to uczynić, musimy dodać do pliku `/etc/sysctl.conf` tę
poniższą linijkę:

    kernel.panic = 60

## Parametr kernel.panic\_on\_oops

Istnieje także parametr `kernel.panic_on_oops` , który odpowiada za wywołanie kernel panic w
przypadku innych błędów niż te krytyczne. Mniej poważne błędy kernela są nazywane "oops". Nie
doprowadzają one bezpośrednio do paniki, gdyż kernel może się obronić ubijając złowrogi proces. W
sporej części przypadków "oops", system z mniejszym lub większym powodzeniem może pracować dalej i
nie jest wymagany jego restart. Jeśli w tym parametrze ustawimy `0` , sprawi to, że kernel będzie
próbował funkcjonować dalej w przypadku gdy napotka taki niezbyt krytyczny błąd. Natomiast jeśli
ustawimy `1`, to kernel spanikuje natychmiast. Dopisujemy zatem i ten poniższy parametr do pliku
`/etc/sysctl.conf` :

    kernel.panic_on_oops = 0

Warto dodać, że w przypadku gdy parametr `kernel.panic` jest ustawiony na niezerową wartość, system
zostanie zresetowany w czasie określonym w tym parametrze.
