---
author: Morfik
categories:
- Linux
date: "2015-06-08T16:02:13Z"
date_gmt: 2015-06-08 15:02:13 +0200
published: true
status: publish
tags:
- xserver
- klawiatura
title: Przecinek vs. kropka na klawiaturze numerycznej
---

Jak możemy [przeczytać pod tym linkiem](https://en.wikipedia.org/wiki/Decimal_mark) , zróżnicowanie
pod względem zapisu większych liczb jak i części dziesiętnych różni się w zależności od kraju. Jedne
do oddzielania od siebie tysięcy wykorzystują kropki, inne zaś przecinki i vice versa w przypadku
części niecałkowitej. Nie będę tutaj się rozpisywał na temat wyższości kropki nad przecinkiem, bo
problem dotyczy zupełnie czegoś innego, mianowicie, tego jak system interpretuje znaki pochodzące z
klawiatury numerycznej.

<!--more-->
## Przecinek czy kropka?

W zależności od układu klawiatury, klawisz del na klawiaturze numerycznej powinien być automatycznie
dostosowany do danego języka. Czyli w przypadku polskiego, powinniśmy mieć, chyba przecinek, a może
kropkę? Dokładnie nie wiem, bo ja i tak korzystam z kropek do oddzielania części niecałkowitej i
spacji w przypadku liczb idących w tysiące czy miliony. Wobec czego mam pewien problem, tj. nie
udało mi się ustawić kropki pod klawiszem del . Zawsze tam jest przecinek, choć nawet nadruk
bezpośrednio na klawiszu pokazuje kropkę. Można by przejść nad tym do porządku dziennego i szukać
klawisza \> na zwykłej klawiaturze, bo pod nim też jest kropka ale tak się nie stanie w tym
przypadku.

Istnieje jedna opcja dla Xservera, która potrafi wymusić korzystanie z odpowiedniego separatora. By
z niej skorzystać musimy mieć już [wstępnie skonfigurowaną
klawiaturę]({{< baseurl >}}/post/klawiatura-i-jej-konfiguracja-pod-debianem/) w systmie. W
zależności od tego czy korzystamy z natywnych debianowych narzędzi czy też konfiguracja odbywa się
przez Xserver, poniższy parametr trzeba dopisać do odpowiedniego pliku, tj. `/etc/default/keyboard`
lub `/etc/X11/xorg.conf.d/10-keyboard.conf` odpowiednio.

Zatem, jeśli życzymy sobie by pod klawiszem del na klawiaturze numerycznej była kropka, dopisujemy
do parametrów Xservera `kpdl:dot` . Jeśli zaś chcemy by to był przecinek, dodajemy `kpdl:comma` .
Teraz już tylko wystarczy zresetować środowisko graficzne i zmiany powinny zacząć obowiązywać.
