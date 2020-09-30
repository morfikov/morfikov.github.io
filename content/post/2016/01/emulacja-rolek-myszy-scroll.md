---
author: Morfik
categories:
- Linux
date: "2016-01-17T17:14:57Z"
date_gmt: 2016-01-17 16:14:57 +0100
published: true
status: publish
tags:
- xserver
- mysz
title: Emulacja rolek myszy (scroll)
---

[Na forum DUG](https://forum.dug.net.pl/viewtopic.php?pid=295709) pojawił się ciekawy temat
dotyczący emulacji rolek myszy (scroll) w urządzeniach, które ich nie posiadają. W tym przypadku
chodziło o bliżej nieokreślony model [trackball'a](https://pl.wikipedia.org/wiki/Trackball).
Niemniej jednak, są też myszy, które może i rolki mają, ale użytkownikom tych urządzeń zwyczajnie
nie chce się wysilać, by tymi kółkami kręcić non stop. Jako, że ja się zaliczam do tej grupy osób,
pomyślałem, by zaimplementować sobie ficzer, który sprawi, że moja bardzo wypasiona mysz będzie
miała pod prawym przyciskiem również emulację scroll'a. Oczywiście dalej będzie można klikać prawym
przyciskiem, by uzyskać dostęp do menu kontekstowego i pod tym względem nic się nie zmieni. W tym
wpisie sprawdzimy jak taka emulacja rolek wygląda w praktyce.

<!--more-->
## Emulacja rolki za sprawą Xserver'a

Jako, że to Xserver zajmuje się obsługą urządzeń wskazujących, które podłączamy do naszego
komputera, to w jego konfiguracji możemy dodać stosowne wpisy, które sprawią, że emulacja scroll'a
zostanie aktywowana. Niemniej jednak, musimy ustalić kilka parametrów zanim przejdziemy do pisania
pliku konfiguracyjnego. Wpisujemy zatem `xinput` w terminalu, by uzyskać listę dostępnych urządzeń,
przykładowo:

    $ xinput
    ...
    ⎜   ↳ A4Tech USB Mouse                          id=9    [slave  pointer  (2)]
    ...

To będzie nasza mysz laboratoryjna. Do tego urządzenia możemy odwoływać się po nazwie, tj. `A4Tech
USB Mouse` lub też po ID ( `9` ) . Jeśli odwołujemy się po nazwie, to trzeba ją ująć w `" "` .

Każde wymienione urządzenie ma określoną domyślną konfigurację, na którą składa się szereg
parametrów. Wszystkie z nich możemy podejrzeć za sprawą `xinput list-props` . Nas interesować będą
te poniższe parametry:

    $ xinput list-props 9
    ...
    Evdev Third Button Emulation (286):     0
    Evdev Third Button Emulation Timeout (287):     1000
    Evdev Third Button Emulation Button (288):      3
    Evdev Third Button Emulation Threshold (289):   20
    Evdev Wheel Emulation (290):    0
    Evdev Wheel Emulation Axes (291):       0, 0, 4, 5
    Evdev Wheel Emulation Inertia (292):    10
    Evdev Wheel Emulation Timeout (293):    200
    Evdev Wheel Emulation Button (294):     4
    Evdev Drag Lock Buttons (295):  0

Emulacja trzeciego klawisza jak i scroll'a musi zostać aktywowana oraz trzeba im przypisać
odpowiednie przyciski. Musimy także dostosować sobie czas, po upłynięciu którego emulacja będzie
załączana. Wartość `1000` odpowiada za jedną sekundę. Każdy klik poniżej tego czasu będzie
oznaczać zwyczajne kliknięcie prawym przyciskiem myszki.

Niektóre aplikacje, np. `tmux` mogą nie działać jak należy po zmianie tych ustawień. By to poprawić,
musimy zmienić wartość parametru `Third Button Emulation Threshold` na `0` , zaś parametr `Third
Button Emulation Button` ustawić na `2` .

Podobnie jak w przypadku wyboru urządzenia, przy konfiguracji parametrów również możemy się
odwoływać do pełnych nazw lub ID (wartości w nawiasach). Parametry zmieniamy w poniższy sposób:

    $ xinput set-prop 9 286 1
    $ xinput set-prop 9 287 400
    $ xinput set-prop 9 288 2
    $ xinput set-prop 9 289 0
    $ xinput set-prop 9 290 1
    $ xinput set-prop 9 294 3

    $ xinput get-button-map 9
    1 2 3 4 5 6 7 8 9 10 11 12

Zmiany są natychmiastowe i powinniśmy być w tej chwili w stanie, np. przewijać stronę w przeglądarce
po przyciśnięciu prawego przycisku i wykonaniu ruchu myszą do przodu lub do tyłu. Jeśli nasza mysz
ma rolki, to one również powinny działać bez problemów. Wyżej mamy również podane aktualne mapowanie
przycisków myszy.

Jeśli zadowalają nas te ustawienia, to piszemy plik konfiguracyjny dla Xserver'a. Sprawi to, że nie
będziemy musieli wywoływać tych powyższych poleceń za każdym razem, gdy startujemy graficzną sesję.
Plik nazwijmy sobie `10-mouse.conf` i wrzućmy go do katalogu `/etc/X11/xorg.conf.d/` . Treść, jaką
należy w tym pliku umieścić, jest poniżej:

    Section "InputClass"
          Identifier        "A4Tech USB Mouse"
          Driver            "evdev"
          MatchIsPointer    "yes"
          MatchDevicePath   "/dev/input/event*"
          MatchProduct      "USB"
          MatchVendor       "A4Tech"

          Option "Name" "A4Tech USB Mouse"
          Option "AccelerationNumerator" "2"
          Option "AccelerationDenominator" "1"
          Option "AccelerationThreshold" "4"

          Option "ButtonMapping" "1 2 3 4 5 6 7 8 9 10 11 12"
          Option "YAxisMapping" "4 5

          Option "EmulateThirdButton" "on"
          Option "EmulateThirdButtonButton" "2"
          Option "EmulateThirdButtonMoveThreshold" "0"
          Option "EmulateThirdButtonTimeout" "400"
          Option "EmulateWheel" "1"
          Option "EmulateWheelButton" "3"

Pozostałe opcje użyte w powyższym pliku zostały opisane we wpisie poświęconym [konfiguracji samej
myszy](/post/mysz-i-jej-konfiguracja-na-linuxie/) . Zatem jeśli mamy problem z
dopasowaniem urządzenia lub też nie do końca rozumiemy za co odpowiadają poszczególne parametry, to
zachęcam do zapoznania się z podlinkowanym artykułem. Warto też zajrzeć do [man
evdev](http://manpages.ubuntu.com/manpages/wily/en/man4/evdev.4.html).

## Emulacja roki myszy przy pomocy mouseemu

Emulacja scroll'a myszy może się odbyć także za sprawą narzędzia `mouseemu` . Jest ono dostępne w
debianie w pakiecie o tej samej nazwie, zatem nie powinno być problemów z jego instalacją. Jeśli
chodzi zaś o samo narzędzie, to za jego sprawą jesteśmy w stanie zrealizować mniej więcej to samo
zadanie, co przy pomocy wyżej opisanego pliku konfiguracyjnego Xserver'a. Podstawowa różnica tkwi
jednak w tym, że zamiast korzystać z prawego przycisku myszki, ustawiamy modyfikator w postaci
przycisku na klawiaturze. W taki sposób, po przyciśnięciu i trzymaniu klawisza, przykładowo Alt ,
zachowanie myszy ulegnie zmianie i za pomocą ruchów myszy będziemy w stanie przewijać strony.

Po zainstalowaniu pakietu `mouseemu` zostanie utworzony plik `/etc/default/mouseemu` , w którym jest
trzymana konfiguracja poszczególnych klawiszy. Za scroll odpowiada poniższa linijka:

    SCROLL="-scroll 125"

Numerek `125` określa klawisz Win . Jeśli chcemy przypisać sobie inny modyfikator, to wystarczy
odpalić `evtest` i odczytać kod pożądanego klawisza:

    Event: time 1453044885.934446, type 4 (EV_MSC), code 4 (MSC_SCAN), value 700e3
    Event: time 1453044885.934446, type 1 (EV_KEY), code 125 (KEY_LEFTMETA), value 0
    Event: time 1453044885.934446, -------------- SYN_REPORT ------------

Po dostosowaniu powyższego pliku, usługę trzeba zresetować, by zmiany zaczęły obowiązywać.
