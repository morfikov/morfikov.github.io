---
author: Morfik
categories:
- Linux
date: "2016-06-07T18:56:45Z"
date_gmt: 2016-06-07 16:56:45 +0200
published: true
status: publish
tags:
- cron
GHissueID: 353
title: Uruchamianie zadań cron co 15 minut lub 6 godzin
---

[cron](https://pl.wikipedia.org/wiki/Cron) to takie linux'owe narzędzie, które cyklicznie, co pewien
ustalony interwał czasu, wykonuje jakieś zaplanowane zadania. Minimalna wartość tego interwału to
jedna minuta. Niemniej jednak, bardzo wielu użytkowników linux'a zastanawia się w jaki sposób
wywołać określone zadanie, np. co 5, 10, 15, 30 minut, czy nawet co 2, 6 albo 12 godzin. Dlatego
właśnie powstał ten wpis, by nieco przybliżyć mechanikę samego cron'a.

<!--more-->
## Tablica zadań cron'a (crontab)

Zadania, które `cron` przetwarza, mogą zostać umieszczone w kilku miejscach. Jednym z nich są
katalogi `/etc/cron.hourly/` , `/etc/cron.daily/` , `/etc/cron.weekly/` oraz `/etc/cron.monthly/` .
Nazwy tych folderów jasno wskazują kiedy umieszczone w nich zadania (skrypty) mają być wywoływane,
tj. co godzinę, co dzień, co tydzień i co miesiąc. Zawartość tych katalogów jest obsługiwana przez
plik `/etc/crontab` . To jest systemowa [tablica cron'a](https://pl.wikipedia.org/wiki/Crontab).
Poniżej jest przykład takiej systemowej tablicy:

    # min hour day mon dow user command
    17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
    25 17   * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
    47 17   * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
    52 17   1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

Każdy z tych wpisów odnosi się do określonego katalogu. Na początku takiego wpisu mamy coś na wzór
`25 17 * * *` . To jest instrukcja, którą `cron` się posiłkuje przy wykonywaniu zadań. W tym
przypadku `25` i `17` oznacza godzinę 17:25 . To w tym czasie zostanie wywołane to drugie zadanie.
Taka instrukcja się różni dla każdego z tych w/w wpisów, w zależności od tego, czy akcja ma być
wywoływana co godzinę, co dzień, co tydzień czy co miesiąc. Wszystkie te wpisy dotyczą wywoływania
skryptów obecnych w katalogach, o których wspomnieliśmy wyżej. Do obsługi zaplanowanych zadań
użytkowników są przeznaczone inne tablice.

Każdy użytkownik w systemie może posiadać własną tablicę, które w debianie są umieszczone w katalogu
`/var/spool/cron/crontabs/` . Te tablice wyglądają podobnie do tej systemowej, z tą różnicą, że
użytkownik definiuje w nich zwykle konkretne polecenia, a nie skrypty. Nic jednak nie stoi na
przeszkodzie, by za pomocą tablicy użytkownika skrypty wywołać. Poniżej jest przykład tablicy
użytkownika:

    # min   hour    day     mon     dow     command
    05    17 *  *  *  /usr/bin/gpg --refresh-keys > /dev/null 2>&1

Jedyna widoczna różnica to brak kolumny `user` . Powyżej mamy przykład zadania, które wykona się o
godzinie 17:05 w każdym dniu, czyli co 24h. Jeśli zastąpimy `17` przy pomocy `*` , to wtedy to
zadanie się będzie wykonywać co godzinę, za każdym razem w piątej minucie. Pozornie nie ma opcji, by
wywołać to zadanie co 15 minut czy 6 godzin ale to tylko pozory. `cron` dzieli każdą godzinę na 60
minut, a każdy dzień na 24 godziny. Biorąc pod uwagę fakt, że minimalny interwał to jedna minuta,
możemy nieco inaczej sprecyzować czas w instrukcji dla cron'a.

## Wywoływanie zadania cron co 5, 10, 15 i 30 minut

Generalnie, to nie ma znaczenia dla nas czy `cron` ma wywołać jakieś zadanie co 5 minut czy co 30
minut. W każdym przypadku postępuje się w podobny sposób. Jako, że godzina ma 60 minut, to ten czas
dla interwałów 5, 10, 15, 30 minut trzeba opisać w tablicy cron'a następująco:

    */5     *   *     *     *
    */10    *   *     *     *
    */15    *   *     *     *
    */30    *   *     *     *

W tym przypadku rozpatrujemy jedną godzinę, czyli 60 minut, zatem `*/5` oznacza interwał 5-minutowy
począwszy od minuty 0, czyli 0, 5, 10, 15, etc. Tak samo wygląda to w każdej z pozostałych
instrukcji.

## Wywoływanie zadania cron co 2, 6 i 12 godzin

Podobnie sprawa wygląda w przypadku wywoływania zadać co kilka godzin. Z tym, że tutaj dysponujemy
24 godzinami i to tę liczbę trzeba wziąć pod uwagę. Zatem, instrukcja wygląda mniej więcej tak:

    05    */2    *     *     *
    05    */6    *     *     *
    05    */12   *     *     *

W ten sposób `*/2` wywoła zadanie w godzinie 0, 2, 4, etc, czyli co dwie godziny. Trzeba przy tym
zaznaczyć, że nic nie stoi na przeszkodzie, by korzystać z przecinka do precyzowania konkretnych
godzin czy minut, przykładowo:

    05    0,6,12,18   *     *     *

Ta powyższa instrukcja jest równoważna z zapisem `*/6` .

## Wywoływanie skryptów

Możemy także utworzyć kolejny katalog dla skryptów w `/etc/`, dajmy na to `/etc/cron.half-hour/` i
umieszczać w nim takie skrypty, które mają być wywoływane co pół godziny. Niemniej jednak, trzeba
dopisać w systemowej tablicy cron'a jedno zadanie, które będzie miało ustawione czas w formie
opisanej w powyższych przykładach. Edytujemy zatem plik `/etc/crontab` i dopisujemy do niego tę
poniższą
    linijkę:

    */30 *    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.half-hour )
