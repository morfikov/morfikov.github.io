---
author: Morfik
categories:
- Linux
date: "2015-06-19T17:17:11Z"
date_gmt: 2015-06-19 15:17:11 +0200
published: true
status: publish
tags:
- pendrive
- bezpieczeństwo
- cd
- dvd
- hdd
- ssd
- autostart
title: Autostart i automatyczne montowanie nośników
---

Menadżery plików potrafią automatycznie montować nośniki wymienne w oparciu o pakiet
[udisks2](https://www.freedesktop.org/wiki/Software/udisks/). Potrafią także uruchamiać odpowiednie
aplikacje zlokalizowane na tych urządzeniach. Może to prowadzić do oczywistych zagrożeń i jeśli nasz
system ma być bezpieczny, to musimy wyłączyć obie te opcje. W różnych menadżerach plików, ten proces
przebiega inaczej. Ja korzystam ze [spacefm](https://ignorantguru.github.io/spacefm/) i w jego
przypadku mamy dość rozbudowany mechanizm polityki dotyczącej tych dwóch powyższych kwestii.
Niemniej jednak, w przypadku każdego z menadżera plików, proces postępowania powinien przebiegać
mniej więcej w ten sam sposób.

<!--more-->
## Automatyczne montowanie zasobów

Jeśli korzystamy z jakichś środowisk graficznych, to prawdopodobnie spotkaliśmy się z sytuacją taką,
że po wsadzeniu pendrive, wyskakuje nam okienko z zawartością kilku jego partycji. Jeśli mamy ich 2
czy więcej, może to być troszkę uporczywe.

W przypadku menadżera `spacefm` , mamy możliwość określenia tego jakie nośniki mają być
automatycznie montowane. Każde urządzenie podłączane do komputera ma szereg opcji i jeśli chodzi o
napędy cd/dvd czy pendrive, to one mają przypisane flagę `removable` , te pierwsze dodatkowo mają
jeszcze `optical` . To na ich podstawie możemy zdefiniować, które urządzenia mają być montowane
automatycznie.

Niektóre menadżery plików (w tym i spacefm) dają nam możliwość określenia pewnych urządzeń, które
mają być automatycznie montowane przy podłączaniu nośnika. Dzięki takiej polityce, możemy dodać
jedynie jakiś zaufany pendrive (lub jedną z jego partycji), a całą resztę montować ręcznie:

![](/img/2015/06/1.automatyczne-montowanie-spacefm.png#huge)

Ja uważam jednak, że skoro zamontowanie urządzenia sprowadza się do kliknięcia w odpowiednią pozycję
na liście, to możemy odpuścić sobie definiowanie zaufanych urządzeń. Jakby nie patrzeć, to stwarza
zagrożenie bezpieczeństwa. Zamiast tego najlepiej wyłączyć to automatyczne montowanie nośników
całkowicie:

![](/img/2015/06/2.automatyczne-montowanie-spacefm-opcje.png#huge)

## Autostart aplikacji

Innym zagrożeniem bezpieczeństwa są pliki [Autorun.inf](https://pl.wikipedia.org/wiki/Autorun.inf) ,
które mają na celu start określonego programu znajdującego się np. na płycie cd/dvd. Jeśli
korzystamy przy tym z automatycznego montowania nośników, to po podłączeniu takiego urządzenia czy
wsadzeniu płytki do napędu, zostanie uruchomiony program i praktycznie nie mamy wpływu na to jaki.
Tego typu scenariusz to doskonała droga dla różnej maści wirusów. Dlatego ja zalecam wyłączenie
również i tej opcji:

![](/img/2015/06/3.autostart-spacefm.png#huge)
