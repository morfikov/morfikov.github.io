---
author: Morfik
categories:
- Linux
date: "2015-10-31T22:53:43Z"
date_gmt: 2015-10-31 20:53:43 +0100
published: true
status: publish
tags:
- pulseaudio
- dźwięk
- sieć
- debian
title: PulseAudio i przesyłanie dźwięku przez sieć
---

Bardzo wielu ludzi nie uświadamia sobie faktu, że w przypadku komputerów praktycznie wszystkie dane,
z którymi mamy styczność, są zapisane za pomocą dwóch znaków, tj. 0 i 1. Mając to na względzie, nie
ma chyba informacji, której by nie można było przesłać przez sieć. Serwer dźwięku, jak sama nazwa
wskazuje, jest w stanie odbierać dane zawierają informacje dźwiękowe. Dlatego też jeśli jakiś
komputer nie posiada karty muzycznej lub/i nie jest fizycznie podłączony do głośników, to nie
stanowi to większego problemu by był on w stanie odtwarzać dźwięk, przynajmniej w tym sensie jakim
my to rozumiemy. W tym wpisie sþróbujemy zrealizować przesyłanie dźwięku przez sieć wykorzystując do
tego [PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/) i zobaczymy czy sprawi nam
to jakieś problemy.

<!--more-->
## Konfiguracja serwera dźwięku

Jak zatem odtworzyć dźwięk na maszynie, która fizycznie nie ma takiej możliwości? Przede wszystkim,
aplikacje, które są uruchamiane w danym systemie, komunikują się między sobą. Jeśli jakaś maszyna
jest wyposażona w lokalny serwer dźwięku PulseAudio, to zainstalowane na niej programy będą
przesyłać dźwięk do tego serwera. Ten z kolei prześle otrzymany sygnał do karty muzycznej, która
pośle go na głośniki. Widzimy zatem, że wystarczy zmienić lokalizację serwera dźwięku, by sterować
tym jaki hardware go odtworzy. Generalnie rzecz biorąc, tam gdzie są głośniki jest i karta dźwiękowa
i to ta maszyna ma z zamiarze odtwarzać dźwięk. Nie zawsze jednak będziemy dysponować komputerem,
który będzie wyposażony w kartę dźwiękowa lub/i głośniki. Być może i jakość podzespołów na danym
komputerze będzie o wiele lepsza niż w przypadku tych, którymi dysponujemy na co dzień. Podobnie
sprawa ma się w sytuacji, gdy nie che nam się przełączać i konfigurować poszczególnych elementów
domowego studia. W każdym z powyższych, serwer oferujący przesyłanie dźwięku przez sieć zdaje się
być dość przyzwoitą opcją, która może nam znacznie ułatwić życie.

Jeśli mamy do dyspozycji maszynę, która ma niewielkie zasoby systemowe, tj. mało pamięci
operacyjnej, czy też słaby procesor, ale przy tym dobrą kartę dźwiękową i głośniki, to możemy na
niej zainstalować serwer dźwięku PulseAudio. Dźwięk na takim systemie konfigurujemy w taki sposób
jak byśmy chcieli go używać lokalnie, z tą różnicą, że musimy załadować dodatkowy moduł:
`module-native-protocol-tcp` . To właśnie on jest odpowiedzialny za realizację zdalnych żądań. By
aktywować ten moduł dopisujemy do pliku `/etc/pulse/default.pa` tę poniższą linijkę:

    load-module module-native-protocol-tcp auth-ip-acl=192.168.10.0/24

Powyżej mamy określony parametr `auth-ip-acl` , który definiuje adresy IP, z których połączenia mają
być akceptowane. Wszystkie inne zostaną odrzucone. Po edycji tego pliku konfiguracyjnego, resetujemy
serwer:

    $ pulseaudio -k
    $ pulseaudio -D

## Konfiguracja klienta

Obecnie chyba każda dystrybucja linux'a jest zależna od PulseAudio. Może nie wykorzystuje go jako
domyślnego systemu dźwięku ale nie wpływa to na obecność pakietu `libpulse0` . To za jego sprawą
sprawdzane jest środowisko w poszukiwaniu zmiennej `$PULSE_SERVER` . Jeśli zostanie ona odnaleziona,
to dźwięk zostanie przesłany pod adres w niej wskazany. W przypadku gdy nie można jej odnaleźć w
środowisku, to wykorzystywany jest lokalny serwer PulseAudio, no chyba, że nie jest on
zainstalowany. Wtedy dźwięk jest odtwarzany za pomocą ALSA. By być w stanie zrealizować przesyłanie
dźwięku przez sieć, to na stacji klienckiej musimy wyeksportować zmienną `$PULSE_SERVER` , podając
przy tym adres serwera dźwięku, przykładowo:

    $ export PULSE_SERVER=192.168.1.150

I to w zasadzie cała nasza praca. Po wyeksportowaniu tej zmiennej, przesyłanie dźwięku przez sieć
powinno działać OOTB i nie powinniśmy mieć z nim żadnych problemów. Poniżej jest fotka obrazująca
odtwarzanie pliku `.mp3` w `mpv` na maszynie, która nie ma zainstalowanej karty dźwiękowej.

![](/img/2015/10/1.przesylanie-dzwieku-przez-siec.png#big)

Może i karty dźwiękowej nie ma zainstalowanej w tym systemie ale widać to nie przeszkadza w żaden
sposób by `mpv` odtwarzał dźwięk bez większych przeszkód.
