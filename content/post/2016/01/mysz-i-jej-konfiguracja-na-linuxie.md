---
author: Morfik
categories:
- Linux
date: "2016-01-08T15:09:28Z"
date_gmt: 2016-01-08 14:09:28 +0100
published: true
status: publish
tags:
- xserver
- mysz
- debian
- xserver
- openbox
title: Mysz i jej konfiguracja na linux'ie
---

Mysz, zaraz obok klawiatury, to jedno z tych narzędzi, z których korzystamy praktycznie bez przerwy
ilekroć tylko siadamy przed komputerem. Z reguły działa ona prawidłowo, nawet pod linux'em, z tym,
że dla jednych użytkowników standardowe ustawienia nie są zbyt zadowalające. Nawet jeśli wszystkie
przyciski zostaną rozpoznane poprawnie, to zawsze pozostaje, np. kwestia dostosowania szybkości
przemieszczania się kursora po ekranie. Oczywiście, istnieje także szereg innych aspektów, które
możemy sobie dostosować i tym właśnie się zajmiemy w niniejszym wpisie.

<!--more-->
## Mysz widziana oczami Xserver'a

W zaawansowanych środowiskach graficznych nie ma zbytnio potrzeby zawracać sobie głowy szczegółową
konfiguracją szeregu urządzeń, bo mamy do tego celu oddelegowany panel sterowania, gdzie
poszczególne opcje możemy sobie w prosty sposób dostosować. Niemniej jednak, te osoby, które cenią
sobie lekkość środowiska i korzystają jedynie z menadżerów okien, np. openbox, muszą nauczyć się
konfigurować sprzęt ręcznie i mysz nie jest tu żadnym wyjątkiem.

Xserver jest w stanie rozpoznać i poprawnie skonfigurować prawie każdą mysz, którą podłączymy do
komputera i to można mu zaliczyć na plus. Domyślne ustawienia, jako że są domyślne, mają
satysfakcjonować większość osób, tak by nie wymagać od nich poświęcania czasu przy konfiguracji
sprzętu. Jeśli nasza mysz nie działa prawidłowo lub też nie zadowalają nas te standardowe
ustawienia, to naturalnie możemy je sobie dostosować za pomocą pliku `xorg.conf` . My generalnie nie
będziemy wykorzystywać tego pliku w całości, dlatego też sekcję poświęconą myszy upchniemy w pliku,
którzy wrzucimy do katalogu `/etc/X11/xorg.conf.d/` . Stwórzmy zatem tam plik `10-mouse.conf` i
dodajmy do niego tę poniższą treść:

    Section "InputClass"
          Identifier        "A4Tech USB Mouse"
          Driver            "evdev"
          MatchIsPointer    "yes"
          MatchDevicePath   "/dev/input/event*"
          MatchProduct      "USB"
          MatchVendor       "A4Tech|Logitech"

          Option      "Name" "A4Tech USB Mouse"
          Option      "AccelerationNumerator" "2"
          Option      "AccelerationDenominator" "1"
          Option      "AccelerationThreshold" "4"

          Option      "ButtonMapping"   "1 2 3 4 5 6 7 8 9 10 11 12"
    #     Option      "XAxisMapping"    ""
          Option      "YAxisMapping"    "4 5"
    #     Option      "SwapAxes"        "on"
    #     Option      "InvertX"         "on"
    #     Option      "InvertY"         "on"
    EndSection

Więcej opcji naturalnie można znaleźć w [man
evdev](https://www.x.org/archive/X11R7.5/doc/man/man4/evdev.4.html). Poniżej zaś znajduje się
wyjaśnienie użytych wyżej opcji:

  - `Identifier` odpowiada za identyfikator urządzenia. Można wpisać dowolną frazę.
  - `Driver` to sterownik odpowiedzialny za konfigurację urządzenia, obecnie korzysta się z `evdev`
    .
  - `MatchIsPointer` odpowiada za dopasowanie myszy.
  - `MatchDevicePath` odpowiada za dopasowanie urządzenia na podstawie ścieżki.
  - `MatchProduct` oraz `MatchVendor` odpowiadają za dopasowanie urządzenia na postawie jego nazwy i
    nazwy jego producenta.
  - `Option` precyzuje dodatkowe opcje, opisane poniżej.

W celu odpowiedniego uzupełnienia wszystkich opcji dopasowania (te mające w nazwie `Match` ), trzeba
ustalić parametry urządzenia, które podpięliśmy do komputera. W tym celu możemy skorzystać z
narzędzia `evtest` . W debianie znajduje się ono w pakiecie o tej samej nazwie. Możemy także
zajrzeć do pliku `/proc/bus/input/devices` . W tym przypadku mamy do czynienia z następującym
modelem myszy:

    # evtest
    ...
    /dev/input/event13:     A4Tech USB Mouse

Nazwa `A4Tech USB Mouse` składa się z części `Vendor` (A4Tech) oraz `Product` (USB Mouse). Jeśli
chcemy skonfigurować jedynie określony model myszy, to te wartości mogą zostać podane w opcjach
`MatchProduct` oraz `MatchVendor` . Jeśli parametry mają dotyczyć wszystkich myszy, to możemy
ograniczyć się do opcji `MatchIsPointer` , ewentualnie podać także ścieżkę do urządzenia w
`MatchDevicePath` . Naturalnie, można stosować dowolne kombinacje.

### Logi Xserver'a

Logi Xserver'a są zawsze przydatne, gdy nasz sprzęt nie został rozpoznany jak należy. W takim
przypadku możemy odszukać określony log i ustalić, gdzie tkwi błąd. Weźmy sobie przykładowy log
dotyczący rozpoznanej wyżej myszy:

    [ 32408.914] (II) config/udev: Adding input device A4Tech USB Mouse (/dev/input/event5)
    [ 32408.914] (**) A4Tech USB Mouse: Applying InputClass "evdev pointer catchall"
    [ 32408.914] (II) Using input driver 'evdev' for 'A4Tech USB Mouse'
    [ 32408.914] (**) A4Tech USB Mouse: always reports core events
    [ 32408.914] (**) evdev: A4Tech USB Mouse: Device: "/dev/input/event5"
    [ 32408.968] (--) evdev: A4Tech USB Mouse: Vendor 0x9da Product 0xa
    [ 32408.968] (--) evdev: A4Tech USB Mouse: Found 12 mouse buttons
    [ 32408.968] (--) evdev: A4Tech USB Mouse: Found scroll wheel(s)
    [ 32408.968] (--) evdev: A4Tech USB Mouse: Found relative axes
    [ 32408.968] (--) evdev: A4Tech USB Mouse: Found x and y relative axes
    [ 32408.968] (II) evdev: A4Tech USB Mouse: Configuring as mouse
    [ 32408.968] (II) evdev: A4Tech USB Mouse: Adding scrollwheel support
    [ 32408.968] (**) evdev: A4Tech USB Mouse: YAxisMapping: buttons 4 and 5
    [ 32408.968] (**) evdev: A4Tech USB Mouse: EmulateWheelButton: 4, EmulateWheelInertia: 10, EmulateWheelTimeout: 200
    [ 32408.968] (**) Option "config_info" "udev:/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1/2-1.1.1/2-1.1.1:1.0/0003:09DA:000A.000A/input/input39/event5"
    [ 32408.968] (II) XINPUT: Adding extended input device "A4Tech USB Mouse" (type: MOUSE, id 9)
    [ 32408.968] (II) evdev: A4Tech USB Mouse: initialized for relative axes.
    [ 32408.968] (**) A4Tech USB Mouse: (accel) keeping acceleration scheme 1
    [ 32408.968] (**) A4Tech USB Mouse: (accel) acceleration profile 0
    [ 32408.968] (**) A4Tech USB Mouse: (accel) acceleration factor: 2.000
    [ 32408.968] (**) A4Tech USB Mouse: (accel) acceleration threshold: 4
    [ 32408.969] (II) config/udev: Adding input device A4Tech USB Mouse (/dev/input/mouse0)

Mamy tutaj szczegółową rozpiskę na temat konfiguracji myszy. W przypadku, gdy część parametrów nie
została dobrana poprawnie, możemy je sobie przestawić w oparciu o te powyższe informacje. Jeśli
jednak czegoś nam brakuje, to zawsze możemy skorzystać z `xinput` :

    # xinput
    ...
          ↳ A4Tech USB Mouse                          id=9    [slave  pointer  (2)]
    ...
    # xinput list-props 9

### Akceleracja myszy

Jeśli kursor myszy porusza się na ekranie zbyt wolno albo zbyt szybko i mamy problemy z jego
ogarnięciem, możemy nieco zmienić akcelerację myszy. W tym celu korzystamy z opcji
`AccelerationNumerator` , `AccelerationDenominator` oraz `AccelerationThreshold` . [Istnieje ich
jeszcze kilka](https://www.x.org/archive/X11R7.7/doc/man/man5/xorg.conf.5.xhtml#heading8) ale na
nasze potrzeby, te trzy nam wystarczą.

Jeśli chcemy przyśpieszyć mysz, to współczynnik `AccelerationNumerator`/`AccelerationDenominator`
musi być większy od 1. Jeśli chcemy zatem dwukrotnie przyśpieszyć kursor myszy, to określamy
odpowiednio 2 i 1 dla tych parametrów. Możemy także mysz spowolnić. Opcja `AccelerationThreshold`
odpowiada zaś za efektywność akceleracji. Domyślne wartości są poniżej:

    Option      "AccelerationNumerator" "2"
    Option      "AccelerationDenominator" "1"
    Option      "AccelerationThreshold" "4"

Te powyższe opcje możemy dostosować sobie także za sprawą narzędzia `xset` , przykładowo:

    $ xset m 2/1 4

Można je dodać do autostartu środowiska graficznego, tak by te ustawienia były aplikowane za każdym
razem. By z kolei przywrócić ustawienia domyślne, wpisujemy:

    $ xset m default

Możemy także podejrzeć aktualną konfigurację myszy za pomocą `xset -q` .

### Konfiguracja osi myszy

Opcje `InvertX` oraz `InvertY` odwracają osie myszy. Jeśli ustawione, kierunek poruszania kursora
ulegnie zamianie na przeciwny. Jeśli chcemy skorzystać z obu tych opcji naraz, to możemy posłużyć
się także parametrem `SwapAxes` .

### Konfiguracja rolek

Parametr `ButtonMapping` odpowiada za odpowiednie zmapowanie przycisków myszy. Każda mysz ma ich
kilka i cześć z nich może być rozpoznana błędnie. Jeśli jakiś przycisk chcemy wyłączyć, to musimy
podać wartość `0` . W logu mieliśmy podane, że mysz ma 12 przycisków, oraz, że 4 i 5 zostały
przypisane rolce Y. Gdyby wystąpił problem z rolką, to wtedy trzeba by skorzystać z parametrów
`XAxisMapping` oraz `YAxisMapping` mapując odpowiednie przyciski. Ta mysz ma tylko jedną rolkę (Y).

## Wygląd kursora myszy

Mysz, a właściwie jej kursor, możemy sobie dostosować w wielu miejscach. Należy jednak pamiętać, że
te ustawienia mogą zostać nadpisane przez rozbudowane środowiska graficzne, takie jak GNOME czy KDE.
Nawet jeśli korzystamy jedynie z prostego menadżera okien, np. openbox, to również musimy uważać, by
czasem nie przepisać sobie tej konfiguracji. Z grubsza mamy trzy opcje, z których możemy skorzystać.

### Konfiguracja kursora za pomocą update-alternatives

Jeśli interesuje nas globalna konfiguracja (systemowa), to możemy pokusić się o skorzystanie z
narzędzia `update-alternatives` . Przy jego pomocy możemy nie tylko konfigurować domyślny wygląd
kursora myszy ale także całą masę innych rzeczy. Niemniej jednak, tutaj ograniczymy się tylko do
kursorów. By zmienić wygląd kursora, wpisujemy w terminal to poniższe polecenie (działa uzupełnianie
za pomocą klawisza Tab ):

    # update-alternatives --config x-cursor-theme
    There are 3 choices for the alternative x-cursor-theme (providing /usr/share/icons/default/index.theme).

      Selection    Path                                     Priority   Status
    ------------------------------------------------------------
      0            /usr/share/icons/Adwaita/cursor.theme     90        auto mode
      1            /usr/share/icons/Adwaita/cursor.theme     90        manual mode
    * 2            /usr/share/icons/DMZ-Black/cursor.theme   30        manual mode
      3            /usr/share/icons/DMZ-White/cursor.theme   50        manual mode

    Press Enter to keep the current choice[*], or type selection number:

Gwiazda ( `*` ) oznacza, że ten styl kursora jest aktualnie ustawiony. Niby są 4 pozycje ale na
dobrą sprawę mamy tylko 3 motywy. Pozycja `0` odpowiada za tryb automatyczny. Jeśli to ona jest
wybrana, to wtedy decyduje priorytet. Możemy oczywiście przestawić się na tryb manualny i wybrać
sobie jedną z pozostałych opcji podając jej numer. To, jakie pozycje są do naszej dyspozycji, zależy
od zainstalowanych w systemie pakietów.

### Pliki w katalogu /etc/X11/Xresources/ oraz plik ~/.Xresources

Wygląd kursora myszy można konfigurować również za pomocą plików w katalogu `/etc/X11/Xresources/`
lub też ich lokalnego odpowiednika, tj. pliku `~/.Xresources` . W debianie, ich lokalizacje można
sobie dostosować w pliku `/etc/X11/Xsession` . Standardowo mamy tam min. te dwie poniższe linijki:

    SYSRESOURCES=/etc/X11/Xresources
    USRRESOURCES=$HOME/.Xresources

Przykładowy wpis w pliku `~/.Xresources` odpowiadający za mysz, może wyglądać mniej więcej tak:

    Xcursor.theme:     DMZ-Black
    Xcursor.size:      16

Rozmiar kursora określamy za pomocą `Xcursor.size` . Natomiast nazwę motywu precyzujemy w
`Xcursor.theme` . To jakie motywy mamy do wyboru, możemy odszukać za pomocą tego poniższego
polecenia:

    $ find /usr/share/icons ~/.icons -type d -name "cursors"
    /usr/share/icons/DMZ-White/cursors
    /usr/share/icons/mate/cursors
    /usr/share/icons/Adwaita/cursors
    /usr/share/icons/DMZ-Black/cursors

### Narzędzie lxappearance

Kursor myszy można także skonfigurować, np. za pomocą narzędzia `lxappearance` , które jest dostępne
standardowo w debianie w pakiecie o tej samej nazwie. Ten pakiet zwykle jest instalowany w
środowiskach LXDE ale też jest bardzo często wykorzystywany w minimalnych instalacjach, np. gdy
mamy do czynienia jedynie z menadżerami okiem. Tak wygląda w nim konfiguracja kursora myszy:

![](/img/2016/01/1.konfiguracja-mysz-lxappearance.png#big)

W `lxappearance` możemy dostosować szereg innych rzeczy, tak jak to widać na fotce powyżej. Po
zmianie ustawień, zostaną wygenerowane dwa pliki konfiguracyjne: `~/.gtkrc-2.0` oraz
`~/.config/gtk-3.0/settings.ini` . Jak nazwy wskazują, są to pliki interfejsu GTK. Możemy je
oczywiście ręcznie poddać edycji ale pamiętać należy, że wszystkie zmiany jakie wprowadzimy zostaną
nadpisane po ponownym skorzystaniu z `lxappearance` .
