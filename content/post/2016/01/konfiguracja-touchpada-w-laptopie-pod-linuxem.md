---
author: Morfik
categories:
- Linux
date: "2016-01-03T20:48:40Z"
date_gmt: 2016-01-03 19:48:40 +0100
published: true
status: publish
tags:
- xserver
- touchpad
GHissueID: 526
title: Konfiguracja touchpad'a w laptopie pod linux'em
---

Laptopy zwykle mają niewielkie rozmiary i są możliwie okrojone, tak by zapewnić jak największą
mobilność. Oczywiście nikt tych maszyn nie pozbawił klawiatury, bo to czyniłoby je mało użytecznymi.
Możemy oczywiście spotkać całą masę modeli, które nie mają wydzielonej klawiatury numerycznej, czy
też brak określonych przycisków. Niemniej jednak, nie jest to aż taka niedogodność jak brak myszy. W
laptopach zamiast myszki mamy zaimplementowany [touchpad](https://pl.wikipedia.org/wiki/Touchpad).
Korzystanie z niego wymaga znacznej wprawy ale i tak wątpię, że ktoś byłby w stanie używać go do gry
w CS'a. Tak czy inaczej, każde urządzenie w linux'ie idzie w mniejszym czy większym stopniu sobie
skonfigurować. Taki laptopowy touchpad nie jest żadnym wyjątkiem i w tym wpisie postaramy się go
nieco ogarnąć.

<!--more-->
## Touchpad i Xserver

Jakiś czas temu byłem zmuszony pracować w minimalnym środowisku właśnie na laptopie i się okazało,
że korzystanie z touchpad'a ma linux'ie przy standardowych ustawieniach nie jest zbyt wygodne.
Jako, że touchpad ma robić za taką niezbyt sprawną mysz, to jest on konfigurowany za pośrednictwem
plików Xserver'a. Mamy całą gamę parametrów, które możemy sobie zmienić wedle uznania poprawiając
tym samym dość znacznie komfort pracy na laptopie. Stwórzmy sobie zatem plik
`/etc/X11/xorg.conf.d/50-synaptics.conf` i dodajmy w nim tę poniższą zawartość:

    Section "InputClass"
    Identifier "touchpad"
    Driver "synaptics"
    MatchIsTouchpad "on"
          Option "TapButton1" "1"
          Option "TapButton2" "3"
          Option "TapButton3" "2"
          Option "VertEdgeScroll" "off"
          Option "VertTwoFingerScroll" "on"
          Option "HorizEdgeScroll" "off"
          Option "HorizTwoFingerScroll" "on"
          Option "PalmDetect" "1"
          Option "PalmMinWidth" "8"
          Option "PalmMinZ" "200"
          Option "RBCornerButton" "3"
          Option "EmulateTwoFingerMinZ" "40"
          Option "EmulateTwoFingerMinW" "8"
          Option "FingerLow" "35"
          Option "FingerHigh" "40"
          Option "MaxSpeed" "3"
          Option "MinSpeed" "1"
    EndSection

Opcji, które jesteśmy w stanie skonfigurować jest dość sporo. Listę wszystkich opcji wraz z
wartościami możemy uzyskać przez wpisanie w terminalu `synclient -l` . Wyżej mamy jedynie te
częściej używane. Poniżej zaś jest rozpiska użytych opcji:

  - `TapButton1` -- lewy przycisk myszy. Wartość `1` odpowiada za jeden punkt styczny z touchpad'em
    (jeden palec).
  - `TapButton2` -- środkowy przycisk myszy. Wartość `3` odpowiada za trzy punktu styczne z
    touchpad'em (trzy palce).
  - `TapButton3` -- prawy przycisk myszy. Wartość `2` odpowiada za dwa punkty styczne z touchpad'em
    (dwa palce).
  - `VertEdgeScroll` -- przewijanie stron w pionie przy pomocy prawej krawędzi touchpad'a.
  - `VertTwoFingerScroll` -- przewijanie stron w pionie przy pomocy dwóch punktów stycznych (dwóch
    palców).
  - `HorizEdgeScroll` -- przewijanie stron w poziomie przy pomocy dolnej krawędzi touchpad'a
  - `HorizTwoFingerScroll` -- przewijanie stron w poziomie przy pomocy dwóch punktów stycznych
    (dwóch palców).
  - `PalmDetect` -- wykrywanie dłoni. Gdy dłoń zostanie wykryta, kursor myszy zostanie zablokowany i
    nie będzie można klikać. Przydatne podczas pisania.
  - `PalmMinWidth` -- minimalny obszar nacisku (szerokość dłoni).
  - `PalmMinZ` -- minimalna siła nacisku.
  - `RBCornerButton` -- konfiguracja prawego górnego rogu touchpad'a. Wartość `3` odpowiada za prawy
    przycisk myszki.
  - `EmulateTwoFingerMinZ` -- siła nacisku przy przewijaniu stron za pomocą dwóch palców.
  - `EmulateTwoFingerMinW` -- szerokość obszaru nacisku przy przewijaniu stron za pomocą dwóch
    palców.
  - `FingerLow` -- siła nacisku, poniżej której system uważa, że przycisk został zwolniony.
    Niekoniecznie trzeba odrywać palce od powierzchni touchpad'a.
  - `FingerHigh` -- siła nacisku, powyżej, której system uważa, że przycisk został wciśnięty.
  - `MaxSpeed` -- maksymalna szybkość przemieszczania się kursora po ekranie.
  - `MinSpeed` -- minimalna szybkość przemieszczania się kursora po ekranie.

Dzięki tej powyższej konfiguracji będziemy mogli używać dowolnego miejsca na powierzchni touchpad'a
jako lewego przycisku myszy. Będziemy też w stanie symulować prawy i środkowy przycisk myszy
stukając jednocześnie dwoma lub trzema palcami w jego powierzchnię. Będziemy mieć możliwość
przewijania stron w pionie i poziomie za pomocą dwóch palców przeciąganych po powierzchni
touchpad'a. Mamy także włączoną detekcję dłoni, choć każdy z nas musi dobrać parametry pod siebie
drogą eksperymentów. Jeśli chodzi zaś o szybkość przemieszczania się kursora, to im bardziej
gwałtownie przesuniemy palce po powierzchni, tym szybciej kursor się przemieści. Do kalibracji
touchpad'a może posłużyć nam narzędzie `evtest`. Można w nim podejrzeć wartości nacisku na
powierzchnie, jak i również aktualną pozycję palca.

## Konfiguracja touchpad'a w czasie rzeczywistym

Przy pomocy narzędzia `synclient` jesteśmy w stanie nie tylko podejrzeć aktualną konfigurację
touchpad'a ale również mamy możliwość dokonywania zmian we wszystkich parametrach. Zmiany mają efekt
natychmiastowy i nie musimy za każdym razem resetować środowiska graficznego, tak jak to miałoby
miejsce w przypadku dostosowania pliku konfiguracyjnego Xserver'a. By ustawić przykładową opcję,
wpisujemy do terminala jej nazwę oraz pożądaną wartość, przykładowo:

    $ synclient PalmDetect=1

Jeśli interesują nas opcje, których nie ma określonych wyżej w pliku, to nic nie stoi na
przeszkodzie, by je tam dodać. Trzeba jednak pamiętać, że ten plik ma określoną strukturę. Najpierw
precyzujemy `Option` . Następnie między `" "` podajemy nazwę opcji, którą możemy odczytać z
`synclient` , przykładowo `"PalmDetect"` . Na końcu zaś, również w `" "` , umieszczamy wartość tej
opcji, przykładowo `"1"` . Wszystko jest oddzielone od siebie za pomocą spacji albo tabulacji.
