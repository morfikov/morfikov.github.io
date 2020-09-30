---
author: Morfik
categories:
- Linux
date: "2015-11-02T21:14:35Z"
date_gmt: 2015-11-02 19:14:35 +0100
published: true
status: publish
tags:
- tty
- framebuffer
title: Zrzut ekranu konsoli TTY (fbcat)
---

W przypadku, gdy dzieją się dziwne rzeczy z naszym linux'em i ten zaczyna sypać niezrozumiałymi dla
nas błędami, to zawsze możemy takie zachowanie uwiecznić na zrzucie ekranu. Co jednak w przypadku,
gdy nawali nam środowisko graficzne i nie będziemy mieli jak zrobić zrzutu? Przecie takie aplikacje
jak `scrot` czy też `shutter` działają w oparciu o Xserver i bez niego nie zrobią fotki naszego
desktop'a. Zwykle gdy zmuszeni jesteśmy ratować nasz system, robimy to w konsoli TTY. No chyba, że
sprawa jest poważniejsza, wtedy sięgamy po [system
live](/post/wlasny-system-live-i-tworzenie-go-od-podstaw/) . Logi systemowe czy też
aplikacji mogą okazać się pomocne ale przecie "fotka jest warta więcej niż tysiąc słów". Jak zatem
zrobić zrzut ekranu mając do dyspozycji jedynie tekstową konsolę TTY? Czy jest to w ogóle możliwe?

<!--more-->
## Robienie zrzutu przy pomocy fbcat

Jest kilka narzędzi, które mogą nam pomóc zrobić zrzut zawartości konsoli TTY. Mi najbardziej do
gustu przypadł [fbcat](http://jwilk.net/software/fbcat) . Jest dostępny w repozytoriach debiana,
także nie powinno być żadnych problemów z jego instalacją. Po tym jak go wgramy do systemu,
przechodzimy na jedną z konsol TTY ( Ctrl-Alt-F2 ).

By zrobić fotkę przy pomocy `fbcat` musimy albo posiadać uprawnienia administratora, albo też być
dodani do grupy `video` . A to z tego względu, że musimy posiadać dostęp do urządzenia `/dev/fb0` .
Bez tego, `fbcat` jest kompletnie bezużyteczny. Dlatego też, zakładam, że mamy możliwość odczytu w/w
urządzenia.

Zrzut tego co aktualnie jesteśmy w stanie zobaczyć na TTY robimy przez wpisanie poniższego
polecenia:

    # fbcat > fotka.png

I to w sumie cała filozofia. Poniżej fotka, która została zrobiona na potrzeby tego wpisu:

![](/img/2015/11/1.tty-fbcat-zrzut-ekranu.png#huge)
