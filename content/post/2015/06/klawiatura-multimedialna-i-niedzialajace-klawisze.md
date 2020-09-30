---
author: Morfik
categories:
- Linux
date: "2015-06-08T12:51:05Z"
date_gmt: 2015-06-08 11:51:05 +0200
published: true
status: publish
tags:
- udev
- xserver
- klawiatura
title: Klawiatura multimedialna i niedziałające klawisze
---

Bardzo ciężko spotkać wypasioną klawiaturę, po której podłączeniu wszystkie przyciski będą
funkcjonować jak należy, a to z tego względu, że nie do końca są one wykrywane przez nasz system.
Zwykle są to klawisze multimedialne lub inne niestandardowe przyciski, które nie pasują do układu
104 klawiszy. W większości przypadków [odpowiednie skonfigurowanie modelu
klawiatury](/post/klawiatura-i-jej-konfiguracja-pod-debianem/) powinno rozwiązać
nasze problemy. Nie zawsze jednak posiadany przez nas model jest do wyboru. Czasami nawet i jego
wskazanie nie pomaga i najzwyczajniej w świecie niektóre klawisze po prostu nie zostaną wykryte
przez system.

<!--more-->
## SCANCODE, KEYCODE oraz KEYSYM

Ja posiadam klawiaturę Logitech Media Keyboard Elite i w jej przypadku połowa klawiszy dodatkowych
nie działa. By móc ustalić przyczynę, trzeba wiedzieć co się tak naprawdę dzieje po wciśnięciu
klawisza na klawiaturze. Są generalnie trzy terminy, z którymi można się spotkać: SCANCODE, KEYCODE
oraz KEYSYM.

SCANCODE jest to kod klawisza, który klawiatura przesyła do kernela po przyciśnięciu jakiegoś
przycisku na niej. KEYCODE jest to przemapowany przez kernel SCANCODE i każdy kernel posiada tablicę
z parami KEYCODE-SCANCODE. Jeśli kernel jest w stanie odczytać SCANCODE, powinien zwrócić KEYCODE. Z
kolei KEYSYM jest tym, co my rozpoznajemy jako literki i cyferki lub też inne znaki jakie mamy
zwykle nadrukowane na klawiszach naszej klawiatury.

## Klawiatura i jej błędne kody

Jest szereg narzędzi, które potrafią odczytać SCANCODE lub/i KEYCODE. Zwykle ludzie używają do tego
celu `showkey` . Jednak jest bardzo duże prawdopodobieństwo, że znajdziemy się w takiej sytuacji,
gdzie klawisz będzie miał KEYCODE, a nie będzie miał przy tym SCANCODE, czyli wygląda to tak jakby
klawiatura nie wysłała żadnego sygnału do kernela, a ten jakimś cudem potrafił go zidentyfikować.
Tak było z klawiszami na mojej klawiaturze.

Innym narzędziem szeroko rozpowszechnionym i też przy tym dość często używanym jest `xev` . Podobnie
jak poprzednik, ma dość sporo wad. Przede wszystkim, podaje złe kody. Dla przykładu, jeśli wklepiemy
poniższą linijkę do
    terminala:

    $ xev | grep -A2 --line-buffered '^KeyRelease' | sed -n '/keycode /s/^.*keycode \([0-9]*\).* (.*, \(.*\)).*$/ /p'

to po wciśnięciu klawisza `a` zostanie zwrócony wynik `38 a` . Co w tym dziwnego? Mamy KEYCODE oraz
odpowiadający mu znak, z tym, że nie do końca. Jeśli byśmy skorzystali z `showkey` , to on z kolei
pokaże `akeycode 30 press` . Zatem mamy inny KEYCODE. Komu wierzyć?

Xserver potrafi obsłużyć niezbyt wygórowaną liczbę KEYCODE, która wynosi 256. Z jakichś powodów
pierwsze 8 kodów (0-7) jest zarezerwowanych. Tak więc do dyspozycji mamy znaki od 8 do 255 włącznie
i to co `xev` rozumie pod znakiem 38, w rzeczywistości nie ma kodu 38, a 30. Dodatkowo kody 8 i 255
również są zarezerwowane, czyniąc tym samym kod 9 pierwszym, który możemy wykorzystać. Domyślnie
jest on przeznaczony dla klawisza Esc. I tak w `showkey` ten klawisz jest widziany jako 1, a w `xev`
zaś jako 9. Jeśli ktoś chce korzystać z `xev`, musi pamiętać, by odjąć 8 od wartości KEYCODE
zwracanej przez ten program.

Problem z podbijaniem numerków przez `xev` to nie jedyna z jego wad. Inną jest to, że czasami nie
podaje on zupełnie żadnego KEYCODE i tak też było w moim przypadku. Nawet po przemapowaniu pewnych
klawiszy, `xev` nie zwraca żadnego KEYCODE po ich przyciśnięciu.

### Złe wartości SCANCODE w kernelu

Szukając czegoś na temat SCANCODE, natknąłem się na informację, że [czasami kernel może zwracać złe
wartości SCANCODE](https://bbs.archlinux.org/viewtopic.php?id=43662) oraz iż można dopisać do
linijki kernela w extlinux/grub odpowiedni parametr, by to przekłamanie naprawić:

    atkbd.softraw=0

Prawdopodobnie nie będziemy musieli z tego rozwiązania korzystać.

## Zmiana układu klawiatury przy pomocy UDEV'a

Jak nie `showkey` i nie `xev` , to jaka pozostaje nam alternatywa? Na szczęście nie jest tak źle
jakby mogło się wydawać. Mamy jeszcze w zanadrzu pakiet `evtest` . Zainstalujmy i odpalmy go:

    # evtest
    No device specified, trying to scan all of /dev/input/event*
    Available devices:
    /dev/input/event0:      AT Translated Set 2 keyboard
    /dev/input/event1:      Lid Switch
    /dev/input/event2:      Power Button
    /dev/input/event3:      Power Button
    /dev/input/event4:      Video Bus
    /dev/input/event5:      A4Tech USB Mouse
    /dev/input/event6:      Logitech Logitech USB Keyboard
    /dev/input/event7:      PC Speaker
    /dev/input/event8:      HDA Digital PCBeep
    /dev/input/event9:      HDA Intel MID Mic
    /dev/input/event10:     HDA Intel MID Headphone
    /dev/input/event11:     HDA Intel MID HDMI/DP,pcm=3
    /dev/input/event12:     HP WMI hotkeys
    /dev/input/event13:     SynPS/2 Synaptics TouchPad
    /dev/input/event14:     HP Webcam-101
    /dev/input/event15:     Logitech Logitech USB Keyboard
    Select the device event number [0-15]:

Jak widzimy, zostały przeskanowane wszystkie urządzenia w katalogu `/dev/input` i jak możemy również
zauważyć `event6` oraz `event15` są od mojej zewnętrznej klawiatury, z którą mam problemy. Czemu są
dwa urządzenia, a nie jedno? Klawiatura jest tylko jedna przecież. Do końca nie wiem jak to wygląda
na innych klawiaturach ale w przypadku tej, pierwsza pozycja odpowiada za zwykłe klawisze, druga zaś
za klawisze multimedialne. Jako, że mi nie działa tylko połowa klawiszy multimedialnych, to będę
korzystał z urządzenia `event15`. Jeśli poddamy je testowi, zobaczymy poniższą mapę klawiszy:

    # evtest /dev/input/event15
    ...
        Event code 113 (KEY_MUTE)
        Event code 114 (KEY_VOLUMEDOWN)
        Event code 115 (KEY_VOLUMEUP)
        Event code 116 (KEY_POWER)
        Event code 131 (KEY_UNDO)
        Event code 138 (KEY_HELP)
        Event code 140 (KEY_CALC)
        Event code 142 (KEY_SLEEP)
        Event code 143 (KEY_WAKEUP)
        Event code 155 (KEY_MAIL)
        Event code 156 (KEY_BOOKMARKS)
        Event code 158 (KEY_BACK)
        Event code 159 (KEY_FORWARD)
        Event code 163 (KEY_NEXTSONG)
        Event code 164 (KEY_PLAYPAUSE)
        Event code 165 (KEY_PREVIOUSSONG)
        Event code 166 (KEY_STOPCD)
        Event code 168 (KEY_REWIND)
        Event code 171 (KEY_CONFIG)
        Event code 172 (KEY_HOMEPAGE)
        Event code 182 (KEY_REDO)
        Event code 208 (KEY_FASTFORWARD)
        Event code 210 (KEY_PRINT)
        Event code 217 (KEY_SEARCH)
        Event code 234 (KEY_SAVE)
        Event code 319 (?)
        Event code 328 (BTN_TOOL_QUINTTAP)
        Event code 329 (?)
        Event code 330 (BTN_TOUCH)
        Event code 331 (BTN_STYLUS)
        Event code 418 (KEY_ZOOMIN)
        Event code 419 (KEY_ZOOMOUT)
        Event code 420 (KEY_ZOOMRESET)
        Event code 421 (KEY_WORDPROCESSOR)
        Event code 423 (KEY_SPREADSHEET)
        Event code 425 (KEY_PRESENTATION)
        Event code 430 (KEY_MESSENGER)
    ...

Jak widzimy, część z klawiszy ma kod \> 255 i dlatego one nie działają. Problem w tym, że by
przemapować KEYCODE, czyli zmienić ich numerki na niższe (te \< 255), musimy znać SCANCODE klawisza
na klawiaturze.

W starszych wersjach UDEV'a (przed 206) dostępne były narzędzia `keymap` oraz `findkeyboards` . Od
tej wersji mapa klawiatury jest trzymana w bazie danych UDEV'a pod `/etc/udev/hwdb.bin` . Jest ona
kompilowana z plików `.hwdb` zlokalizowanych min. w katalogu `/etc/udev/hwdb.d/` . Nie mamy też już
do dyspozycji tych dwóch w/w narzędzi.

SCANCODE uzyskujemy za pomocą `evtest` . Po przyciśnięciu klawisza, powinien zostać nam zwrócony
poniższy log:

    Event: time 1396640830.231616, type 4 (EV_MSC), code 4 (MSC_SCAN), value c022e
    Event: time 1396640830.231616, type 1 (EV_KEY), code 419 (KEY_ZOOMOUT), value 1
    Event: time 1396640830.231616, -------------- SYN_REPORT ------------
    Event: time 1396640830.359616, type 4 (EV_MSC), code 4 (MSC_SCAN), value c022e
    Event: time 1396640830.359616, type 1 (EV_KEY), code 419 (KEY_ZOOMOUT), value 0
    Event: time 1396640830.359616, -------------- SYN_REPORT ------------

Pozycja `value c022e` , to właśnie jest SCANCODE.

Natomiast jeśli chodzi o KEYCODE, to musimy wybrać jeden z wolnych, które wyrzucił `evtest` . Jeśli
spojrzymy na powyższy log z mapą klawiszy, zobaczymy, że min. KEYCODE `202` oraz `203` są wolne.
Musimy zatem dowiedzieć się pod jaką nazwą one występują, a tę z kolei odczytujemy z pliku
`/usr/include/linux/input.h` . Dla przykładu:

    ...
    #define KEY_PROG3       202
    #define KEY_PROG4       203
    ...

W moim przypadku, kernel przypisał KEYCODE do paru nieistniejących klawiszy, także jeśli nie
jesteśmy pewni jakie numerki wybrać, to najlepiej powciskać wszystkie klawisze na klawiaturze i
pospisywać każdy KEYCODE.

Mając ustalone wolne KEYCODE, informacje o SCANCODE oraz nazwy klawiszy, stwórzmy plik
`/etc/udev/hwdb.d/65-keyboard-logitech.hwdb`, którego zawartość składać się będzie z dwóch kolumn. W
jednej będzie SCANCODE, a w drugiej nowy KEYCODE. W moim przypadku wygląda to tak (wcięcia mają
znaczenie):

    # /usr/include/linux/input.h

    evdev:input:b0003v046DpC30Fe0110-e0,1,4,k71*
     KEYBOARD_KEY_c01bc=chat
     KEYBOARD_KEY_c022d=prog3
     KEYBOARD_KEY_c022e=prog4
     KEYBOARD_KEY_90040=finance
     KEYBOARD_KEY_c0184=sport
     KEYBOARD_KEY_c0186=shop
     KEYBOARD_KEY_c0188=f14
     KEYBOARD_KEY_90049=f15
     KEYBOARD_KEY_9004a=f16
     KEYBOARD_KEY_9004b=f17
     KEYBOARD_KEY_9004c=f18

Linijkę zaczynającą się od `evdev` trzeba sobie odpowiednio zbudować tak, by dopasować urządzenie,
do którego będzie się odwoływać nowe mapowanie klawiszy. Najprościej jest to zrobić przez
sprawdzenie co zwróci nam to poniższe polecenie:

    # find /sys -name modalias -print0 | xargs -0 cat | sort -u | grep -i 046d
    hid:b0003g0001v0000046Dp0000C30F
    input:b0003v046DpC30Fe0110-e0,1,4,11,14,k71,72,73,74,75,77,79,7A,7B,7C,7D,7E,7F,80,81,82,83,84,85,86,87,88,89,8A,B7,B8,B9,BA,BB,BC,BD,BE,BF,C0,C1,C2,F0,ram4,l0,1,2,3,4,sfw
    input:b0003v046DpC30Fe0110-e0,1,4,k71,72,73,74,83,8A,8C,8E,8F,9B,9C,9E,9F,A3,A4,A5,A6,A8,AB,AC,B6,B8,B9,BA,BB,BC,CA,CB,D0,D2,D8,D9,DB,DC,DD,EA,1A4,ram4,lsfw
    usb:v046DpC30Fd2300dc00dsc00dp00ic03isc00ip00in01
    usb:v046DpC30Fd2300dc00dsc00dp00ic03isc01ip01in00

We wcześniejszych wersjach UDEV'a (\<220) korzystałem z wpisów zaczynających się od `usb` ale one
przestały działać. Wobec tego przeszedłem na te zaczynające się od `input` . Numerki zaczynające się
od `v` oraz `p` czyli `v046d` i `pc30f` to idVendor=046d oraz idProduct=c30f , które można
wyciągnąć, np. z `lsusb` :

    # lsusb | grep Logitech
    Bus 002 Device 009: ID 046d:c30f Logitech, Inc. Logicool HID-Compliant Keyboard (106 key)

Przepisanie klawisza sprowadza się do utworzenia wpisów w postaci `KEYBOARD_KEY_scancode=keycode` ,
gdzie keycode, to nazwa wyciągnięta z pliku `/usr/include/linux/input.h` ale bez `KEY_` i pisana z
małych liter. Po zdefiniowaniu klawiszy, trzeba przeładować bazę danych UDEV'a przy pomocy:

    # udevadm hwdb --update

Nowy układ klawiatury powinien być widoczny po ponownym podłączeniu jej do portu USB komputera.
