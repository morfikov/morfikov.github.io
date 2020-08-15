---
author: Morfik
categories:
- Linux
date: "2020-03-08T19:00:00Z"
published: true
status: publish
tags:
- debian
- monitor
- dpi
- openbox
- xserver
- qt
- gtk2
- gtk3
- czcionki
title: Zmiana DPI w Openbox/Xorg dla monitora HiDPI
---

Jeśli mieliśmy do czynienia z monitorami wysokiej rozdzielczości, to za pewne natrafiliśmy na
problem zbyt małych czcionek, które czyniły interfejs aplikacji w naszym linux'ie mało czytelnym. W
przypadku środowisk graficznych takich jak GNOME czy KDE5/Plasma5 skalowanie interfejsu i czcionek
powinno odbywać się automatycznie ([jeśli nasz ekran ma 192+ DPI i rozdzielczość 1200+ pikseli][5])
lub też za sprawą drobnej zmiany w konfiguracji, tak by użytkownik mógł w miarę komfortowo
korzystać z systemu. O ile w przypadku tych pełnowymiarowych środowisk graficznych można w zasadzie
przełączyć tylko jedną opcję i wszystkie jego aplikacje powinny zostać z powodzeniem odpowiednio
zeskalowane, o tyle problem zaczyna się w momencie, gdy mamy mieszane aplikacje lub też zwyczajnie
używamy jedynie prostego menadżera okien dla Xserver'a, np. Openbox i do tego jeszcze nasz
wyświetlacz ma mniejsze DPI niż 192. W takiej sytuacji konfiguracja interfejsu użytkownika i
czcionek dla ekranów wysokiej rozdzielczości może być nie lada wyzwaniem.

<!--more-->
## Skalowanie interfejsu w Openbox

Środowisko, w którym użytkownik linux'a pracuje, można podzielić na graficzne oraz tekstowe (TTY),
do którego można się przełączyć za pomocą klawiszy `ctrl+alt+F1` . Spora część użytkowników tego
alternatywnego systemu korzysta z gotowych rozwiązań pokroju KDE5/Plasma5 czy też GNOME ale są też
wśród nas tacy użytkownicy, który lubią konfigurować sobie środowisko pracy samodzielnie i używają
jedynie prostego menadżera okien typu Openbox.

Niezależnie jednak od wykorzystywanego środowiska mamy możliwość korzystania z przeróżnych
aplikacji, np. tych mających interfejs GTK3 lub QT. W ich przypadku przy pomocy pewnych zmiennych
systemowych można wpłynąć na wygląd appki wykorzystującej określony interfejs graficzny. Niemniej
jednak, niektóre programy zdają się ignorować ustawienia systemowe i trzeba do nich podejść
indywidualnie, np. aplikacje GTK2.

Widać zatem, że w przypadku Openbox konfiguracja interfejsu aplikacji oraz czcionek na ekranach
wysokiej rozdzielczości może być ździebko problematyczna. Dlatego też poniżej postanowiłem zebrać
do kupy wszystkie te informacje, które mogą pomóc w konfiguracji DPI w takich monitorach.

## Jak wyliczyć optymalny DPI

Po przesiadce ze startego laptopa na nowy okazało się, że zmianie uległa nie tylko przekątna ekranu
(17" -> 14"), ale również i rozdzielczość (1360x768 -> 1600x900). Zatem z jednej strony
zmniejszeniu uległy wymiary wyświetlacza, a jednocześnie wzrosła ilość pikseli. Tego typu sytuacja
efektywnie zwiększyła DPI (Dots Per Inch) monitora, przez co te same elementy, które miałem na
pulpicie starego laptopa, teraz były sporo mniejsze i komfortowo na nie się patrzeć już nie daje.
Jak bardzo zatem zmieniło się DPI?

Standardowo w linux wyświetlacz ma 96x96 DPI. Znając rozdzielczość ekranu oraz jego wymiary, można
to DPI wyliczyć. Wszystkie potrzebne nam wartości można zwykle wydobyć z `xdpyinfo` :

    $ xdpyinfo | grep -B2 resolution
    screen #0:
      dimensions:    1600x900 pixels (423x238 millimeters)
      resolution:    96x96 dots per inch

Trzeba jednak tutaj uważać, bo wymiary podane wyżej w `dimensions` są określane na podstawie tej
wartości co siedzi w `resolution` . Zatem nie zawsze te fizyczne wymiary wyświetlacza mogą być
prawidłowe i w tym przypadku nie są.

Jeśli mamy wątpliwości, to rzućmy okiem również na wyjście `xrandr` :

    $ xrandr | grep -w connected
    LVDS-1 connected primary 1600x900+0+0 (normal left inverted right x axis y axis) 309mm x 174mm

Tu z kolei mamy 309x174 mm.

Możemy też spróbować [wyliczyć te wymiary][1] w oparciu o twierdzenie Pitagorasa i natywną
rozdzielczość ekranu, choć nie zawsze dla każdego monitora się to da zrobić:

    $ echo 'scale=5;sqrt(1600^2+900^2)' | bc
    1835.75597

    $ echo 'scale=5;(14/1836)*1600*25.4' | bc
    309.67680

    $ echo 'scale=5;(14/1836)*900*25.4' | bc
    174.19320

Jak widać, wynik jest bardzo zbliżony do tego, który został określony przez `xrandr` . A jakie
wymiary ma wyświetlacz tego laptopa w rzeczywistości? Okazuje się, że po zmierzeniu go metodą
manualną ma jakieś 310x174 mm. Można zatem przyjąć, że wartości, które zwrócić `xrandr` są
prawidłowe, a ewentualne różnice zwalić na błędy pomiarowe albo błędy przyrządów pomiarowych. Tak
czy inaczej, dysponując wiarygodnymi wymiarami ekranu możemy wyliczyć jego DPI w poniższy sposób:

    3096/254 = 12.19
    1742/254 = 6.86

Wartości 3096 i 1742 to pomnożone przez 10 wymiary monitora w mm (i do tego zaokrąglone), a 254 to
2.54 (1 cal) pomnożone przez 100. Teraz wystarczy podzielić rozdzielczość w pikselach przez te
wyżej otrzymane wymiary w calach:

    1600/12.19 ≈ 131.25
    900/6.86 = 131.19

Można zatem przyjąć, że DPI dla tego monitora to 132x132, a nie 96x96. Zatem interfejs aplikacji i
tekst powinny zostać zeskalowane do 137,5%, by były mniej więcej takiej samej wielkości co na
ekranie starego laptopa.

Zgodnie z tym co można wyczytać na [wiki Archlinux][1], to możemy sobie ustawić dowolną wartość DPI.
Powinniśmy jednak się trzymać wartości 96 (+0%), 120 (+25%), 144 (+50%), 168 (+75%), 192 (+100%) i
podobnych. Gdy zamierzamy korzystać z innych niestandardowych DPI, takich jak nasz 132, to mogą
wystąpić problemy w przypadku interfejsów aplikacji wykorzystujących bitmapy. Zatem jeśli chodzi o
monitor w tym laptopie, to najlepiej by było się posłużyć wartością DPI 144 albo 120. Niemniej
jednak, zeskalowany interfejs w oparciu o te 120 DPI w przypadku tego monitora, to trochę za dużo i
postanowiłem ustawić DPI na 108. Póki co nie zaobserwowałem żadnych problemów natury wizualnej.

### DPI dla Xorg i SDDM

Konfigurację DPI najlepiej rozpocząć od skonfigurowania Xorg'a, a ta sprowadza się do podania
argumentu `-dpi` w poleceniu uruchamiającym Xserver. Pytanie tylko jak uruchamiany jest Xserver?
Zwykle wykorzystywany jest do tego celu jakieś menadżer logowania, np. SDDM. W takiej sytuacji
trzeba wyedytować konfigurację SDDM i zmienić w pliku `/etc/sddm.conf.d/sddm.conf` poniższą
linijkę:

    ServerArguments=-nolisten tcp -dpi 108

Warto w tym miejscu wspomnieć, że SDDM niby jest w stanie włączyć automatyczne skalowanie
interfejsu aplikacji QT dla ekranów wysokiej rozdzielczości za sprawą opcji `EnableHiDPI` w
`[Wayland]`/`[X11]` ale w moim przypadku to automatyczne skalowanie nie działa za dobrze. A poza
tym, to i tak ono dotyczy jedynie aplikacji QT. Dlatego też odradzałbym ustawianie `EnableHiDPI` w
konfiguracji SDDM.

Po określeniu `-dpi` w `ServerArguments` trzeba zrestartować środowisko graficzne, by zmiany
zaczęły obowiązywać. Jeśli interfejs zdaje się być za bardzo powiększony, to możemy nieco obniżyć
wartość parametru `-dpi` . W razie problemów lub wątpliwości zajrzymy do pliku
`/var/log/Xorg.0.log` , bo tam powinna znajdować się informacja o DPI ustawionym przez Xserver:

    # egrep DPI /var/log/Xorg.0.log
    [    83.172] (++) modeset(0): DPI set to (108, 108)

#### DPI dla startx

Jeśli zaś korzystamy ze `startx` , to musimy edytować plik `/etc/X11/xinit/xserverrc` i w nim
dopisać parametr `-dpi` :

    exec /usr/bin/X -nolisten tcp -dpi 108 "$@"

### DPI w Openbox

Jeśli chodzi zaś o samego Openbox'a, to tutaj zbytnio nie konfiguruje się DPI, bo wszystkie
ustawienia powinny być dziedziczone od Xserver'a, jeśli tylko oczywiście określiliśmy mu parametr
`-dpi` . Możemy za to, określić DPI dla konkretnego monitora dopisując w pliku autostartu
`~/.config/openbox/autostart` tę poniższą linijkę:

    xrandr --dpi 108/LVDS

Jeśli zaś chodzi o `LVDS` , to wyciągamy tę wartość z wyjścia `xrandr` :

    $ xrandr | grep connected
    LVDS-1 connected primary 1600x900+0+0 (normal left inverted right x axis y axis) 309mm x 174mm
    VGA-1 disconnected (normal left inverted right x axis y axis)
    HDMI-1 disconnected (normal left inverted right x axis y axis)
    DP-1 disconnected (normal left inverted right x axis y axis)
    HDMI-2 disconnected (normal left inverted right x axis y axis)
    HDMI-3 disconnected (normal left inverted right x axis y axis)
    DP-2 disconnected (normal left inverted right x axis y axis)
    DP-3 disconnected (normal left inverted right x axis y axis)

### Aplikacje QT i GTK2/GTK3

Jeśli mamy problemy z czcionkami w przypadku aplikacji wykorzystujących interfejsy QT i GTK3, to
musimy w nich osobno skonfigurować DPI i do tego celu będzie nam potrzebny zestaw zmiennych
środowiskowych. Te zmienne trzeba będzie wyeksportować w dwóch miejscach. Pierwszym z nich jest
konfiguracja shell'a (zwykle `~/.bashrc` lub `~/.zshrc` ), a drugim jest plik
`~/.config/openbox/environment` . Poniżej są potrzebne zmienne:

    export GDK_SCALE="1"
    export GDK_DPI_SCALE="1.125"

    #export QT_AUTO_SCREEN_SCALE_FACTOR="0"
    #export QT_SCALE_FACTOR="1.125"
    export QT_SCREEN_SCALE_FACTORS="LVDS-1=1.125;"
    export QT_FONT_DPI="108"

[Skalowanie aplikacji GTK3][3] jest określane przez zmienne `GDK_SCALE` oraz `GDK_DPI_SCALE` . Ta
pierwsza zmienna przyjmuje jedynie całkowite wartości i w moim przypadku ustawienie tutaj `2`
powiększało interfejs dość znacznie, dlatego tutaj ustawiłem `1` . Z kolei `GDK_DPI_SCALE` ma
wartość wskazywaną przez DPI, które ustawiliśmy wcześniej (108/96=1.125).

Problemem natomiast mogą być aplikacje GTK2, w których nie ma wsparcia dla skalowania interfejsu.
Jedyną opcją w ich przypadku jest zmigrowanie na odpowiednik GTK3, bo wiele aplikacji takowym
dysponuje.

Jeśli zaś chodzi o zmienne dotyczące interfejsu QT, to mamy tam w zasadzie
`QT_AUTO_SCREEN_SCALE_FACTOR` , który odpowiada za automatyczne skalowanie interfejsu (to samo co
było w konfiguracji SDDM) i w tym przypadku zostało wyraźnie określone, że nie chcemy
autoskalowania interfejsu, bo mamy do czynienia z monitorem, który ma DPI mniejsze niż 192. Dlatego
też trzeba ustawić albo `QT_SCREEN_SCALE_FACTORS` , albo `QT_SCALE_FACTOR` . [Różnic między tymi
dwiema zmiennymi][2] jest kilka. Jeśli ustawimy `QT_SCALE_FACTOR` , to zarówno interfejs jak i
czcionki zostaną przeskalowane i to na wszystkich podłączonych monitorach. Określenie samego
`QT_SCREEN_SCALE_FACTORS` wpłynie jedynie na skalowanie interfejsu dla konkretnego monitora (pary
argument=wartości trzeba oddzielić `;` ). Jeśli dodatkowo czcionki mają ulec skalowaniu, to trzeba
określić również `QT_FONT_DPI` w stosunku do `QT_SCREEN_SCALE_FACTORS` . Mając dwie osobne zmienne
można inaczej zeskalować interfejs aplikacji, jak i tekst jeśli tego typu funkcjonalność nas
interesuje.

### Pozostałe aplikacje GUI

Te powyższe ustawienia powinny w zasadzie zeskalować interfejs sporej części aplikacji, z których
korzystamy na co dzień. Niemniej jednak, nie wszystkie aplikacje będą nam szły na rękę i rysować
się zgodnie z podanymi im instrukcjami. Pomijając aplikacje GTK2, które nie mają
zaimplementowanego wsparcia dla skalowania, to niektóre appki mogą albo ignorować wszystkie te
powyższe ustawienia,  albo też nie do końca ustawiać je poprawnie. Poniżej jest lista tych
aplikacji, które u mnie sprawiały problemy i wymagały dodatkowego dostosowania.

#### Firefox i Thunderbird

W zasadzie te aplikacje wykorzystują interfejs GTK3 i skalują się bez problemu ale trochę za
bardzo -- ich interfejs jest ździebko za duży w stosunku do pozostałych aplikacji. Jeśli nam nie
odpowiada nowy wygląd Firefox'a czy Thunderbird'a, to w ich zaawansowanych ustawieniach
( `about:config` ) można określić ten poniższy klucz:

    layout.css.devPixelsPerPx = 1.125

Domyślnie `layout.css.devPixelsPerPx` przyjmuje wartość `-1` , która nakazuje Firefox'owi i
Thundebird'owi korzystać z ustawień systemowych. Każda inna wartość sprawi, że te aplikacje
przeskalują swój interfejs niezależnie.

#### Tint2

Tint2, to aplikacja GTK2. Na szczęście deweloperzy umożliwili skalowanie elementów panelu przy
pomocy poniższego parametru, który trzeba określić w pliku `~/.config/tint2/tint2rc` :

    scale_relative_to_dpi = 108

Jeśli dodatkowo ikonki zdają się być za małe, to możemy albo im zwiększyć rozmiar, albo też włączyć
im autoskalowanie:

    launcher_icon_size = 0
    systray_icon_size = 0

#### Xresources i URXVT oraz conky

Jeśli mamy w systemie aplikacje, które nie używają interfejsów GTK3 i QT, to można jeszcze
spróbować dla nich ustawić konfigurację czcionek w pliku `~/.Xresources` , a konkretnie chodzi o
dodanie w nim poniższej linijki:

    Xft.dpi: 108

Jeśli dokonujemy zmian w pliku `~/.Xresources` , to trzeba przeładować bazę ustawień poniższym
poleceniem:

    $ xrdb  ~/.Xresources

Dla pewności dobrze jest zweryfikować, czy DPI zostało poprawnie ustawione:

    $ xrdb -query | grep dpi
    Xft.dpi:        108

#### Telegram

Żadne z tych powyżej skonfigurowanych ustawień nie wpłynęło w żaden sposób na aplikację Telegram. W
jej przypadku pozostaje w zasadzie konfiguracja czcionek w ustawieniach tej appki. By to zrobić,
wchodzimy w `Settings` i przewijamy do pozycji `Default interface scale` . Jeśli zaznaczenie tej
opcji (i zrestartowanie Telegrama) nic nam nie da, to pozostaje nam ustawienie skalowania na
sztywno w opcjach tejże appki.

## Konfiguracja DPI pod TTY

Osobnym aspektem pracy na linux jest tryb tekstowy w postaci konsoli TTY. Tam te powyższe
ustawienia nie zadziałają i trzeba sobie tę konsolę skonfigurować za pomocą narzędzi dostarczanych
na Debianie w pakiecie `console-setup` . Można to zrobić przy pomocy poniższego polecenia:

    # dpkg-reconfigure console-setup

lub też przez edycję pliku `/etc/default/console-setup` dodając lub zmieniając te poniższe
wartości:

    ACTIVE_CONSOLES="/dev/tty[1-6]"

    CHARMAP="UTF-8"

    CODESET="Lat2"
    FONTFACE="Terminus"
    FONTSIZE="16x32"

    VIDEOMODE=

Bez znaczenia jest sposób, który sobie wybierzemy, grunt by ustawić odpowiednio zmienną `FONTSIZE` .
Domyślnie jest `Terminus 8x16` , no i jak łatwo można się domyśleć, dla ekranów mających DPI 192
można tutaj ustawić `Terminus 16x32` .

### Konsola podczas startu maszyny

Jeśli mamy do czynienia z DPI większymi lub równymi 192, to można też pokusić się o ustawienie w
kernelu opcji `CONFIG_FONTS` oraz `CONFIG_FONT_TER16x32` , tak by komunikaty podczas startu systemu
(jak i też konsola odzyskiwania) miały nieco bardziej przyjazny dla oka rozmiar. Mając te parametry
ustawione, określamy jeszcze w kernel cmd opcję `fbcon=font:TER16x32` .

## Problemy z aplikacjami po zmianie DPI

Prawdopodobnie niektóre aplikacje będą miały jakieś większe lub mniejsze problemy z wyświetlaniem
czcionek po zmianie DPI. Poniżej znajduje się lista aplikacji, które miały jakieś problemy z
wyświetlaniem czcionek. Warto też rzucić okiem na [artykuł poświęcony konfiguracji czcionek na
Debianie][4], gdyby te poniższe rady nie przyniosły żadnego rezultatu.

### Poszarpane czcionki w qpdfview

W przypadku `qpdfview` wystąpił bardzo dziwny problem, w którym jedynie tekst dokumentów PDF był
strasznie poszarpany, rozmyty i bardzo nieczytelny. Ta aplikacja wykorzystuje interfejs QT, zatem
sprawdziłem czy usunięcie zmiennych `QT_` ze środowiska poprawi sytuację i poprawiło. Jednocześnie
inne aplikacje QT, w tym przeglądarka PDF `okular` , nie doświadczały tego typu problemów z
czcionkami.

Problem tkwił w automatycznym współczynniku piksela (pixel ratio), który domyślnie jest dobierany w
`qpdfview` . Wystarczy zaznaczyć opcję `Use device pixel ratio` przechodząc kolejno w menu
Edit -> Settings -> Graphics -> General. Efekt powinien być zauważalny natychmiast po zastosowaniu
ustawień.

### Aplikacje uruchamiane jako root

Jeśli uruchamiamy aplikacje jako root, np. graficzny edytor tekstu, to może on nie stosować się do
tych ustawień skalowania, które sobie ustawiliśmy. W takim przypadku trzeba w konfiguracji shell'a
dla użytkownika root wyeksportować te zmienne, które eksportowaliśmy dla zwykłego użytkownika.


[1]: https://wiki.archlinux.org/index.php/Xorg#Display_size_and_DPI
[2]: https://doc.qt.io/qt-5/highdpi.html
[3]: https://developer.gnome.org/gtk3/stable/gtk-x11.html
[4]: {{< baseurl >}}/post/fontconfig-i-konfiguracja-czcionek-w-debianie/
[5]: https://wiki.gnome.org/HowDoI/HiDpi
