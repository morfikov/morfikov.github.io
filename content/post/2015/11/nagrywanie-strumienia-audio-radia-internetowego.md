---
author: Morfik
categories:
- Linux
date: "2015-11-19T17:49:32Z"
date_gmt: 2015-11-19 16:49:32 +0100
published: true
status: publish
tags:
- dźwięk
GHissueID: 268
title: Nagrywanie strumienia audio radia internetowego
---

Człowiek raczej nigdy nie przywyknie do absolutnej ciszy. Dlatego nawet jeśli pracujemy nad czymś
ważnym, to chcemy mieć w tle jakiś dźwięk, który zagłuszy ciszę. Zwykle jest to nasza ulubiona
muzyka odtwarzana na jednym z player'ów audio, który mamy zainstalowany w swoim linux'ie. Muzyka
może nam się szybko znudzić, zwłaszcza gdy w kółko odsłuchujemy te same kawałki. Te bardziej
wymagające osobniki preferują informacje zamiast muzyki, a to wiąże się w dużej mierze z wszelkiej
maści podcastami czy też radiami internetowymi. W sporej części przypadków, musimy być obecni przy
kompie podczas nadawania audycji. Może nie koniecznie jest to wymagane przy podcastach, bo pliki
`.mp3` możemy sobie ściągnąć w późniejszym czasie ale sporo stacji radiowych nie dostarczy
słuchaczom audycji w trybie offline. Poza tym, w przypadku podcastów, mamy także część audycji,
która jest nienagrywana i nie jest uwzględniona w pliku `.mp3` . Tak czy inaczej, przydałoby się
mieć możliwość nagrania strumienia audio, który jest przesyłany przez sieć. Czy istnieje jakiś
prosty sposób, który nam to umożliwi?

<!--more-->
## Źródła strumieni audio

Najtrudniejszym krokiem w całym procesie nagrywania ścieżki audio jest problematyczność z
namierzeniem samego strumienia. Większość stacji radiowych udostępnia na swojej stronie jedynie
player'a (i to często we flash), za pomocą którego serwują taki strumień. Obecnie flash już wypadł z
łask i rzadko kto zainstalowałby go w celu odsłuchania jakiejś audycji, która pojawia się raz na
tydzień. Trzeba zatem poszukać alternatywnego źródła, z którego można wyciągnąć adres IP maszyny
nadającej strumień audio. Szereg polskich stacji możemy znaleźć [tutaj](http://radiomap.eu/pl/) .
Weźmy sobie dla przykładu RadioZET. Jego strumień jest dostępny pod tym poniższym linkiem:

    http://radiozetmp3-01.eurozet.pl:8400/listen.pls

Plik `listen.pls` to zwykły plik tekstowy, gdzie znajduje się link do faktycznego strumienia. Tak
czy inaczej, większość player'ów audio jest w stanie ten link wydobyć i odtworzyć go bez problemów.
Przynajmniej [mpv](https://mpv.io/installation/) jest w stanie. Zatem mamy możliwość usłyszenia tego
co goście gadają w tym radiu. Połowa pracy za nami, teraz przechodzimy do lżejszej części, czyli do
tego jak zripować taki strumień.

## Nagrywanie strumienia

Strumień audio możemy nagrać, np. za pomocą `mpv` . Z grubsza mamy do dyspozycji dwa parametry:
`--stream-capture` oraz `--stream-dump` . Jak możemy wyczytać w [manualu
mpv](https://mpv.io/manual/stable/) , różnica między tymi dwoma opcjami polega na tym, że ten
pierwszy równocześnie odtwarza dany strumień i zrzuca go do pliku. Zatem jeśli chcemy słuchać i
nagrywać jednocześnie, to korzystamy z `--stream-capture` , w przeciwnym wypadku używamy drugiej
opcji. Nagrywanie strumienia przy pomocy `mpv` możemy przeprowadzić w poniższy sposób:

    $ mpv http://radiozetmp3-01.eurozet.pl:8400/listen.pls --stream-capture=radio.mp3

Link oczywiście możemy podać bezpośrednio do strumienia zamiast do pliku `listen.pls` . Po wydaniu
tego powyższego polecenia, strumień audio będzie zapisywany w pliku `radio.mp3` . Jeśli w
przyszłości wywołalibyśmy jeszcze raz to polecenie, to dane ze strumienia zostaną dopisane do
końca pliku. Można zatem w bardzo prosty sposób wycinać reklamy.

## Automatyczne nagrywanie

Jeśli z jakiegoś powodu nie możemy nagrać swoich ulubionych audycji, np. nie ma nas w tym czasie w
domu, ale znamy godzinę rozpoczęcia programu, np. w piątki o 20:30, to nic nie stoi na przeszkodzie
by zaprogramować sobie swój system tak by przy pomocy cron'a wykonał odpowiednią akcję o ustalonej
porze. Odpalamy zatem terminal i wpisujemy w nim `crontab -e` i dopisujemy tę poniższą
    linijkę:

    30    20 *  *  5  timeout 2h mpv http://radiozetmp3-01.eurozet.pl:8400/listen.pls --stream-dump=/home/morfik/radio-$(date +'\%Y-\%m-\%d-\%H-\%M-\%S').mp3

Wyżej mamy `timeout` , który po czasie `2h` zabije proces i tym samym zakończy nagrywanie.
