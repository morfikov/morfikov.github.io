---
author: Morfik
categories:
- Linux
date: "2016-01-30T20:37:08Z"
date_gmt: 2016-01-30 19:37:08 +0100
published: true
status: publish
tags:
- video
- sieć
title: Streaming obrazu za sprawą ffmpeg i netcat
---

[Na forum DUG'a pojawił się ciekawy post](https://forum.dug.net.pl/viewtopic.php?id=28188), w którym
autor wątku chciał wykonać coś co określił jako "display mirroring". Poszukałem trochę informacji na
temat tego zagadnienia i okazało się, że to nic innego jak tylko wyświetlenie tej samej zawartości,
np. na dwóch monitorach. Nie jest to nic zaawansowanego, bo przecie Xserver jest w stanie tego typu
zadanie zrealizować. Niemniej jednak, oba monitory muszą być podłączone do tego samego komputera. W
tym przypadku mamy dwie maszyny i dwa osobne monitory. Celem jest przesłanie obrazu z jednej maszyny
na drugą za pomocą sieci. W tym podlinkowanym wątku została poruszona kwestia przechwycenia obrazu
przy pomocy `ffmpeg` i przesłania go przez sieć za pomocą `nc` (netcat). Tak bardzo zainteresowało
mnie to rozwiązanie, że postanowiłem zobaczyć jak wygląda ono w praktyce.

<!--more-->
## Przechwycenie obrazu w ffmpeg

W debianie mamy dostępny pakiet `ffmpeg` , który będziemy musieli zainstalować w systemie, który ma
przesyłać obraz. Problem z tą paczką jest taki, że prawdopodobnie ze względów licencyjnych część
kodeków nie została wkompilowana i zwyczajnie ich brakuje. Jeśli poniższe polecenia nie działają jak
należy, to warto zainteresować się pakietami dostępnymi w repozytorium [debian
multimedia](https://www.deb-multimedia.org/). Dodatkowo, na każdej ze stacji, musimy zainstalować
pakiet `netcat-traditional` .

Poniżej znajduje się przykładowe polecenie, które przechwyci obraz wyświetlany na monitorze:

    ffmpeg \
          -f alsa -ac 2 -i pulse -async 1 \
          -f x11grab -r 30 -s 1366x768 -i :0.0 \
          -vcodec libx264 -preset veryfast -pix_fmt yuv444p -crf 15 \
          -acodec libmp3lame -ab 256k \
          -threads 0 \
          -f mpegts - | nc -l -p 9000

Nie trzeba definiować wszystkich powyższych opcji. Należy jednak dostosować sobie rozdzielczość
ekranu ( `-s` ), ilość klatek na sekundę ( `-r` ), no i oczywiście kodeki. W zależności od potrzeb,
można by także dostosować sobie opcje kodeka video. [Pod tym
linkiem](http://fomori.org/blog/?p=1213) jest ciekawy post dotyczący dostosowania tych powyższych
parametrów.

## Streaming danych do netcat'a

W tej powyższej zwrotce, nas najbardziej interesuje końcówka, tj. `-f mpegts - | nc -l -p 9000` ,
gdzie `-f mpegts` odpowiada za [MPEG transport
stream](https://en.wikipedia.org/wiki/MPEG_transport_stream), który jest konieczny, by przesłać
strumień danych do `nc` . Netcat będzie nasłuchiwał na wszystkich adresach na porcie 9000.

Po wpisaniu tego powyższego polecenia, zacznie się oczekiwanie na podłączenie klienta. Przechodzimy
zatem na drugą stację roboczą i odpalamy dowolny player video. Poniżej przykłady dla `mpv` i `vlc` :

    $ nc 192.168.1.150 9000 | mpv -
    $ nc 192.168.1.150 9000 | vlc -

Po chwili połączenie powinno zostać zestawione. W okienku odtwarzacza video powinniśmy zaś zobaczyć
to co jest aktualnie wyświetlane na monitorze na drugiej stacji roboczej.
