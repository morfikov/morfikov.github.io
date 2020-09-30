---
author: Morfik
categories:
- Linux
date: "2015-06-10T10:45:05Z"
date_gmt: 2015-06-10 09:45:05 +0200
published: true
status: publish
tags:
- udev
title: Jedna reguła udev'a realizująca dwa zadania
---

Jakiś czas temu opisywałem jak podejść do [pisania
reguł](/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/) dla
[udev'a](https://pl.wikipedia.org/wiki/Udev) . W tamtym przypadku wykorzystywana reguła składała się
tak naprawdę z dwóch mniejszych, z których jedna miała ustawioną zmienną `ACTION` na `add` , z kolei
zaś druga na `remove` i w ten oto sposób pierwsza z nich była aplikowana podczas podłączania
określonego urządzenia do komputera, a druga przy jego odłączaniu. Okazuje się jednak, że dwie
reguły nie są konieczne w przypadku gdy mają one dotyczyć tego samego sprzętu i możemy zamiast nich
stworzyć jedną regułę, która będzie stosowana zarówno przy podłączaniu jak i odłączeniu danego
urządzenia.

<!--more-->
## Reguła 2 w 1

Weźmy poniższy przykład reguł udev'a, które mają za zadanie włączać/wyłączać touchpad w zależności
od tego czy zewnętrzna mysz USB zostanie podpięta do laptopa:

    ACTION=="add", SUBSYSTEM=="input", KERNEL=="mouse[0-9]*", SUBSYSTEMS=="usb", \
          ENV{DISPLAY}=":0", \
          ENV{XAUTHORITY}="/home/morfik/.Xauthority", \
          RUN+="/usr/bin/synclient TouchpadOff=1"

    ACTION=="remove", SUBSYSTEM=="input", KERNEL=="mouse[0-9]*", SUBSYSTEMS=="usb", \
          ENV{DISPLAY}=":0", \
          ENV{XAUTHORITY}="/home/morfik/.Xauthority", \
          RUN+="/usr/bin/synclient TouchpadOff=0"

Jeśli akcja pod kątem danego urządzenia ma być przeprowadzana zarówno przy podłączaniu jak i
odłączaniu go, to lepszym wyjściem jest skorzystanie ze zmiennej `ENV{REMOVE_CMD}` . Dzięki
takiemu rozwiązaniu ilość kodu zmniejszy się prawie dwukrotnie przy zachowaniu dokładnie takiej
samej funkcjonalności. Jak zatem powinien wyglądać odpowiednik tych dwóch powyższych reguł?
Odpowiedź jest poniżęj:

    ACTION=="add", SUBSYSTEM=="input", ENV{ID_INPUT_MOUSE}=="1", SUBSYSTEMS=="usb", \
        ENV{XAUTHORITY}="/home/morfik/.Xauthority", \
        ENV{DISPLAY}=":0", \
        ENV{REMOVE_CMD}="/usr/bin/synclient TouchpadOff=0", \
        RUN+="/usr/bin/synclient TouchpadOff=1"

I to wszystko. Jedna reguła realizująca dwa zadania.
