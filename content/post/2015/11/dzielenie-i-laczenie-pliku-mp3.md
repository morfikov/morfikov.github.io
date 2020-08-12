---
author: Morfik
categories:
- Linux
date: "2015-11-13T13:03:57Z"
date_gmt: 2015-11-13 12:03:57 +0100
published: true
status: publish
tags:
- pliki/foldery
- dźwięk
title: Dzielenie i łączenie pliku mp3
---

Każdy z nas odsłuchuje czasem pliki `.mp3` . Niekoniecznie musi to być muzyka, np. mogą być to
audycje radiowe, czy też inne materiały audio. W przypadku dłuższych nagrań problematyczne może być
odnalezienie w nich tego kawałka, który akurat chcielibyśmy ponownie odsłuchać. Podobnie sprawa ma
się w przypadku, gdy dany materiał chcemy komuś przesłać. Jeśli ślemy całe nagranie, to musimy dodać
informację od którego momentu zaczyna się coś dziać w tym utworze. Nie prościej wyciąć ten
interesujący nas kawałek i zapisać go w osobnym pliku?

<!--more-->
## Dzielenie pliku mp3

Do dzielenia plików `.mp3` można wykorzystać [narzędzie
mp3splt](http://mp3splt.sourceforge.net/mp3splt_page/home.php), które w debianie jest dostępne w
pakiecie pod tą samą nazwą. Załóżmy na chwilę, że jesteśmy w posiadaniu dłuższego nagrania audio i
chcielibyśmy wyciąć z niego kawałek, który zawiera się w przedziale czasowym od 10:23 do 15:34.
Odpalamy zatem terminal i wpisujemy w nim to poniższe polecenie:

    $ mp3splt 10.23 15.34 plik.mp3

Możemy także nakazać by `mp3splt` wyszukiwał ciche miejsca w ścieżce audio i na podstawie takich
miejsc ciął plik. Jesteśmy także w stanie określić stały czas wynikowego pliku `.mp3` i w ten sposób
podzielić plik, którego ścieżka trwa godzinę, na 6 części, z których każda mieści się w 10 minutach.
Tych opcji jest naprawdę sporo, dlatego też lepiej jest zaprzęgnąć do pracy `mp3splt-gtk` . Jest to
graficzna nakładka na `mp3splt` , z tym, że tego narzędzia nie ma w repozytoriach debiana. Jest za
to w repozytorium [deb-multimedia](http://www.deb-multimedia.org/). Tak wygląda ten programik:

![]({{< baseurl >}}/img/2015/11/1.dzielenie-pliku-mp3-na-czesci.png)

## Łączenie plików mp3

Jako, że wyżej omówiliśmy sobie dzielenie pliku `.mp3` na części, to przydałoby się poruszyć także
temat łączenia plików `.mp3` . Do tego celu służy nieco inne narzędzie i jest to
[mp3wrap](http://mp3wrap.sourceforge.net/). Nie posiada ono, tak jak poprzednik, interfejsu GUI,
dlatego też będziemy musieli się nauczyć operować nim z poziomu wiersza poleceń. Choć w sumie,
scalanie plików `.mp3` sprowadza się do zapamiętania tylko jednego polecenia:

    $ mp3wrap plik.wynikowy plik*
    ...
      50 %  --> Wrapping plik1.mp3 ... OK
      100 % --> Wrapping plik2.mp3 ... OK

      Calculating CRC, please wait... OK

I to w zasadzie cała filozofia łączenia kilku plików `.mp3` w jeden. Oczywiście można podawać pliki
jeden po drugim ale w przypadku gdy mamy ich dużo to można skorzystać z `*` .
