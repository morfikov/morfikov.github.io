---
author: Morfik
categories:
- Linux
date: "2016-01-10T21:31:47Z"
date_gmt: 2016-01-10 20:31:47 +0100
published: true
status: publish
tags:
- xserver
- debian
- openbox
title: Menadżer okien Openbox
---

Spora większość środowisk graficznych rozrosła się obecnie nieco i ich instalacja na maszynach
wyposażonych w niewiele pamięci RAM może nie wchodzić w grę. Nie jesteśmy też skazani na życie w
konsoli, ani na kupno nowego sprzętu. Możemy nieco odchudzić instalację pozbywając się zbędnych
usług, których tak naprawdę nie potrzebujemy. Poza tym, praktycznie każde z graficznych narzędzi,
przy pomocy których chcemy konfigurować linux'a, ma tekstowe zamienniki, lub też o wiele lżejsze
alternatywy. Jednak w przypadku, gdy korzystamy z takich rozbudowanych środowisk jak GNOME, to nie
koniecznie da się usunąć szereg tych zasobożernych komponentów. Pozostaje nam zwykle jedna opcja,
którą jest usunięcie całego środowiska graficznego i zainstalowanie potrzebnych nam komponentów
osobno. Jako, że mamy już opisany[proces instalacji debiana przy pomocy
debootstrap](/post/instalacja-debiana-z-wykorzystaniem-debootstrap/), [instalację i
konfigurację Xserver'a](/post/konfiguracja-xservera-na-debianie-xorg/), jak i
również [menadżera okien LightDM](/post/menadzer-logowania-lightdm/), to przyszedł
czas na omówienie niezbędnego w sesji graficznej [menadżera
okien](https://pl.wikipedia.org/wiki/Mened%C5%BCer_okien). W tym wpisie skupimy się głównie na
instalacji i konfiguracji Openbox'a.

<!--more-->
## Instalacja Openbox'a w debianie

Na instalację Openbox'a nie składa się wiele pakietów ale, by nas on nie odstraszał swoim wyglądem,
musimy doinstalować szereg dodatkowych pakietów, np. style GTK czy menadżer kompozycji. Poniżej
znajduje się lista pakietów, które zwykle przydają się, gdy chcemy korzystać ze środowiska
graficznego opartego jedynie o sam menadżer okien:

    # aptitude install openbox obconf obmenu obkey lxappearance \
    xdg-utils python-xdg python3-xdg \
    upower \
    dmz-cursor-theme \
    compton conky-all \
    rxvt-unicode-256color tmux gksu \
    tint2 \
    spacefm udevil \
    geany geany-plugin-spellcheck \
    qt4-qtconfig \
    gtk2-engines gtk2-engines-murrine gtk2-engines-pixbuf murrine-themes libgtk2.0-bin \
    gtk3-engines-unico \
    gnome-icon-theme gnome-icon-theme-symbolic gnome-themes-standard at-spi2-core \
    mate-icon-theme-faenza mate-themes \
    dconf-editor dconf-cli

Ten powyższy zestaw zainstaluje nie tylko sam menadżer okien Openbox ale również zostaną
doinstalowane podstawowe komponenty środowiska graficznego, tak by to wszystko miało ręce i nogi.
Pakiety mające w nazwie `xdg` odpowiadają za [implementację autostartu
aplikacji](https://standards.freedesktop.org/autostart-spec/autostart-spec-latest.html), czy też
inne aspekty związane ze specyfikacją samego środowiska graficznego. Z kolei [upower odpowiada za
zarządzanie energią](https://upower.freedesktop.org/) i przełączanie systemu w stan uśpienia czy
hibernacji w zależności od poziomu rozładowania akumulatora. Mamy tam też zaawansowany w swojej
prostocie [menadżer kompozycji compton](https://github.com/chjj/compton), który nada Openbox'owi
przyzwoite efekty bez zbytniego obciążania sprzętu. Przyda się nam także [monitor systemu
conky](https://github.com/brndnmtthws/conky), dzięki któremu będziemy w stanie monitorować procesy
naszego systemu na bieżąco. W środowisku graficznym nie może zabraknąć wirtualnego terminala, choć
nie wszystkim może się podobać [rvxt-unicode](http://software.schmorp.de/pkg/rxvt-unicode.html). Za
to wyśmienicie integruje się z [multiplekserem tmux](https://tmux.github.io/) dając nam możliwość
dzielenia jednego okna terminala na szereg mniejszych. Przydatne też jest posiadanie [panelu
tint2](https://gitlab.com/o9000/tint2/). No i też nie może zabraknąć [menadżera plików
SpaceFM](https://ignorantguru.github.io/spacefm/). A jak zamierzamy korzystać ze SpaceFM, to
koniecznie potrzebujemy [narzędzia udevil](https://ignorantguru.github.io/udevil/), które umożliwi
nam montowanie zasobów bez potrzeby zaciągania do tego użytkownika root. No i mamy tam także bardzo
[zaawansowany edytor tekstu geany](http://www.geany.org/), choć na dobrą sprawę jest to uszczuplone
IDE. Pozostałe pakiety mają zaś zapewnić odpowiednią konfigurację wyglądu okienek aplikacji GTK i QT
oraz dodać nieco urozmaicenia w postaci ikonek, kursorów myszki czy też samego motywu.

Sporo rzeczy z tej listy jest opcjonalnych i nie koniecznie wszystkie z powyższych pakietów musimy
wgrywać sobie do systemu. Niemniej jednak, ten powyższy zestaw jest naprawdę bardzo podstawowy.

## Konfiguracja Openbox'a

Przejdźmy zatem do konfiguracji menadżera okien. Standardowo po zalogowaniu się w graficznej sesji
zobaczymy jedynie szary pulpit i to właśnie jest Openbox. Nie zrażajmy się jednak, bo są to jedynie
pozory, a jak wiadomo mogą one mylić. Poniżej jest fotka obrazująca jak można dostosować sobie
wygląd tego ogołoconego środowiska.

![](/img/2016/01/1.srodowisko-graficzne-openbox.png#huge)

Także jak widać, ogranicza nas jedynie nasza wyobraźnia i potrzeby. Tak czy inaczej, by przekuć tego
standardowego szaraka w nieco bardziej wyrafinowany pod względem wyglądu desktop, musimy zając się
konfiguracją Openbox'a.

Na konfigurację tego menadżera okien składają się z grubsza cztery pliki ulokowane w katalogu
`~/.config/openbox/` . Standardowo nie ma tam jeszcze żadnych plików i musimy całą konfigurację
stworzyć sobie od zera. No prawie od zera, bo mamy do dyspozycji pliki szkieletowe, które możemy
przekopiować z katalogu `/etc/xdg/openbox/` i tylko je sobie dostosować. Przekopiujmy je zatem:

    $ mkdir -p ~/.config/openbox/
    $ cp /etc/xdg/openbox/{rc.xml,menu.xml,autostart,environment} ~/.config/openbox/

W debianie nie musimy korzystać z plików `~/.xinitrc` czy `~/.xprofile` w celu odpalenia Openbox'a.
Domyślna sesja w plikach systemowych jest już tak skonfigurowana, że Openbox startuje bez
jakiejkolwiek ingerencji z naszej strony. Domyślnego menadżera okien, jak i sesję Xserver'a, możemy
określić za pomocą `update-alternatives` w poniższy sposób:

    # update-alternatives --config x-session-manager
    # update-alternatives --config x-window-manager

Zmienne środowiskowe, które mają być ustawione przed startem Openbox'a ustawia się w
`/etc/xdg/openbox/environment` albo `~/.config/openbox/environment` . Za to całe zamieszanie jest
odpowiedzialny skrypt w `/usr/bin/openbox-session` , z którego wynika, że plik `environment` jest
czytany przed startem sesji Openbox'a. Oczywiście trzeba mieć na uwadze, że zmienne eksportowane
przy pomocy tego pliku są widoczne tylko i wyłącznie w sesji Openbox'a. W przypadku zalogowania się
na TTY albo korzystania z innego środowiska graficznego, te zmienne nie zostaną ustawione.

### Autostart aplikacji (autostart)

Autostart Openbox'a jest zarządzany przez plik `~/.config/openbox/autostart` . To w nim umieszczamy
listę aplikacji, które mają być odpalane automatycznie. Możemy także umieszczać tutaj polecenia, te
same, które zwykliśmy wpisywać w terminalu. Zatem jeśli nasz ekran wymaga dostosowania pod kątem
jasności, to nic nie stoi na przeszkodzie, by odpowiednią komendę tutaj wywołać. Pamiętajmy tylko,
że ten plik jest przetwarzany tak jak zwykły skrypt i wszystkie polecenia trzeba odpalać w tle, tak
by nie zablokować procedury autostartu. Poniżej przykładowy wpis:

    (sleep 0.5s && LC_TIME=en_DK.utf8 /opt/thunderbird/thunderbird) &

Ponad to, w katalogu `/etc/xdg/autostart/` znajdują się pliki `.desktop` tworzone automatycznie
przez szereg aplikacji, które chcą startować wraz z podnoszeniem się sesji graficznej. Mogą one być
również brane pod uwagę przez Openbox'a. Jest to bardzo użyteczna właściwość, dlatego wyżej
zainstalowaliśmy pakiet `python-xdg` . Oczywiście każdy użytkownik ma swój własny autostart
zlokalizowany w `~/.config/autostart/` i w przypadku, gdy nie podobają nam się włączone domyślnie
usługi, to możemy je wyłączyć poprzez umieszczenie kopi pliku `.desktop` w katalogu
`~/.config/autostart/` odpowiednio ją przy tym modyfikując. Przykładowy plik `.desktop` wygląda tak:

    [Desktop Entry]
    Name=Dropbox
    GenericName=File Synchronizer
    Comment=Sync your files across computers and to the web
    Exec=dropbox start -i
    Terminal=false
    Type=Application
    Icon=dropbox
    Categories=Network;FileTransfer;
    StartupNotify=false
    OnlyShowIn=GNOME;XFCE;

Opcja `OnlySHowIn` odpowiada za ukrywanie pliku w środowiskach, które nie są tam sprecyzowane. W
przypadku braku tego pola, usługa zostanie odpalona na każdym z nich. Gdy chcemy wyłączyć jakąś
usługę, dopisujemy tam `none` . Powyższy przykład ma tylko kilka pól, które można wykorzystać przy
konfiguracji. Pełna ich lista znajduje się
[tutaj](https://standards.freedesktop.org/desktop-entry-spec/latest/ar01s05.html).

Jeśli chcemy sprawdzić, które aplikacje są automatycznie uruchamiane przez mechanizm XDG w naszym
środowisku graficznym, wydajemy to poniższe polecenie:

    $ /usr/lib/x86_64-linux-gnu/openbox-xdg-autostart --list OPENBOX

### Zmienne środowiskowe (environment)

W pliku `~/.config/openbox/environment` definiujemy zmienne środowiskowe, które można podejrzeć, np.
wpisując w terminalu polecenie `printenv`/`env` . Zmienne dopisane do tego pliku będą ustawione
globalnie dla całej sesji graficznej i wywoływanych w niej terminalach. Poniżej znajduje się
przykładowy wpis:

    export BROWSER="/opt/firefox/firefox"

Jeśli chcemy się pozbyć jakichś zmiennych, które zostały ustawione w sesji graficznej, to możemy to
zrobić za pomocą `unset` , przykładowo:

    unset BROWSER

Trzeba także pamiętać, że `unset BROWSER` nie oznacza tego samego co `export BROWSER=""` .

### Menu kontekstowe (menu.xml)

W pliku `~/.config/openbox/menu.xml` definiujemy wpisy dla menu kontekstowego, dostępnego pod prawym
klawiszem myszki. Można również stworzyć menu dla lewego jak i środkowego przycisku. Takie menu
możemy sobie zrobić ręcznie lub też zaprzęgnąć do roboty jakiś automat. Niemniej jednak, taki
generator spowolni trochę działanie samego menu, no i oczywiście doda w nim sporo śmieci. Dlatego
też plik `menu.xml` najlepiej sobie dostosować ręcznie albo też korzystając z narzędzia `obmenu` :

![](/img/2016/01/2.openbox-obmenu.png#big)

Po każdej edycji menu trzeba przeładować konfigurację Openbox'a przy pomocy tego poniższego
polecenia:

    $ openbox --reconfigure

### Konfiguracja motywu

Wygląd Openbox'a, co prawda, nie powala na pierwszy rzut oka ale można go bez problemu zmienić.
Trzeba tylko ustawić odpowiedni motyw, np. używając do tego celu narzędzia `obconf` :

![](/img/2016/01/3.openbox-obconf.png#big)

Jak widzimy wyżej, mamy dość sporo rzeczy, które możemy sobie dostosować w tym okienku. Wszelkie
wprowadzone zmiany są zapisywane w pliku `~/.config/openbox/rc.xml` i na dobrą sprawę, korzystanie z
tego powyższego narzędzia nie jest wymagane, by dokonać zmian w ustawieniach. Zalecane jest ono
jednak, by wyeliminować możliwość popełnienia błędów.

Motyw ustawiony za pomocą `obconf` dotyczy jedynie samej belki tytułowej i po części też obramowania
okna. Cała zawartość, która znajduje się w takim okienku jest już rysowana w oparciu o motywy GTK
lub QT, w zależności od tego z jaką aplikacją mamy do czynienia. Motyw dla GTK możemy wybrać w
`lxappearance` . Możemy w nim także dostosować ikonki, style myszki czy też wygląd czcionek:

![](/img/2016/01/4.openbox-konfiguracja-lxappearance.png#big)

Po zapisaniu ustawień, zostaną wygenerowane pliki `~/.gtk2rc-2.0` oraz
`~/.config/gtk-3.0/settings.ini` zawierające wskazane wyżej ustawienia.

Motyw QT jest konfigurowany za pomocą narzędzia `qtconfig` :

![](/img/2016/01/5.openbox-qtconfig.png#big)

To narzędzie z kolei wygeneruje plik `~/.config/Trolltech.conf` .

By aplikacje QT i GTK wyglądały z grubsza tak samo, trzeba ustawić `style=GTK+` . Jednak to nie
zadziała jeśli nie posiadamy w środowisku zmiennej `GTK2_RC_FILES="$HOME/.gtkrc-2.0"` . Inną
uciążliwością może być niezgodność ikon. W aplikacjach GTK mogą być ikony znane nam z GNOME, a w
aplikacjach QT są ikonki, które zwykliśmy kojarzyć z KDE. Można to ujednolicić przez ustawienie
zmiennej `GNOME_DESKTOP_SESSION_ID=1` . To jednak pociąga za sobą pewne komplikacje, gdyż ta zmienna
wysyła do programów błędne sygnały o obecności środowiska GNOME na pokładzie, a my go przecież nie
posiadamy. Mamy jedynie menadżer okien Openbox. Prawdopodobnie trzeba będzie doinstalować pewne
komponenty środowiska GNOME, by zachować kompatybilność. Być może część aplikacji, których będziemy
używać już zawiera odpowiednie pakiety w zależnościach. Jeśli jednak nam różne ikonki nie
przeszkadzają, to możemy darować sobie ustawianie tej zmiennej. Tak czy inaczej, potrzebne nam
zmienne dopisujemy do pliku `~/.config/openbox/environment` :

    export GTK2_RC_FILES="$HOME/.gtkrc-2.0"
    export GNOME_DESKTOP_SESSION_ID=1

Jeśli cierpimy na zbyt mały wybór motywów, możemy doinstalować pakiet `murrine-themes` , który
zawiera ich dość sporo. Openbox jest w stanie także przeszukiwać katalog użytkownika `~/.themes/`
pod kątem odnalezienia w nim jakichś motywów. Także możemy bez problemu wgrać sobie również taki
motyw, którego nie ma w repozytorium debiana. Gdy interfejs graficznych aplikacji odpalanych jako
root nie wygląda tak samo jak ten w przypadku zwykłego użytkownika, to te powyższe pliki trzeba
będzie również skopiować w odpowiednie miejsca w katalogu `/root/` .

### Skróty klawiszowe

W pliku `~/.config/openbox/rc.xml` jest także trzymana konfiguracja skrótów klawiszowych. Domyślny
wpis dla skrótu wygląda mniej więcej tak:

    <keybind key="W-Left">
          <action name="DesktopLeft"/>
    </keybind>

Ręczna edycja tego pliku jest mało wygodna ale można skorzystać z graficznego narzędzia `obkey` ,
które znacznie ułatwia definiowanie skrótów:

![](/img/2016/01/6.openbox-obkey.png#big)

### Rozmieszczanie okien

W pliku `~/.config/openbox/rc.xml` można również określać konfigurację dotyczącą rozmieszczania
okien w momencie, gdy zostają one wyświetlone na ekranie, czyli tuż po starcie aplikacji. Spotkałem
się na necie z [narzędziem obapps](http://sourceforge.net/projects/obapps/) ale nie jest ono póki co
dostępne w debianie i trzeba ten plik dostosować sobie manualnie. By to zrobić, musimy określić
szereg wartości identyfikujących okno. Potrzebne nam informacje mogą zostać wydobyte poprzez takie
narzędzia jak `obxprop` czy też `xprop` . Ten pierwszy jest dostarczany bezpośrednio w pakiecie z
Openbox'em, ten drugi w pakietach z Xserver'em. Oba te narzędzia zwracają praktycznie te same
informacje, więc nie ma zbytnio znaczenia, z którego będziemy korzystać. By uzyskać zaś informacje o
przykładowym oknie, wpisujemy jedno z tych poleceń w terminalu, po czym klikamy myszką w jakieś
okno, poniżej przykład:

    $ obxprop | grep OB_APP
    _OB_APP_TYPE(UTF8_STRING) = "normal"
    _OB_APP_TITLE(UTF8_STRING) = "Mozilla Firefox"
    _OB_APP_GROUP_CLASS(UTF8_STRING) = "Firefox"
    _OB_APP_GROUP_NAME(UTF8_STRING) = "firefox"
    _OB_APP_CLASS(UTF8_STRING) = "Firefox"
    _OB_APP_NAME(UTF8_STRING) = "Navigator"
    _OB_APP_ROLE(UTF8_STRING) = "browser"

Te powyższe parametry jednoznacznie identyfikują okno danej aplikacji. W tym przypadku mamy do
czynienia z oknem przeglądarki Firefox. W oparciu o te powyższe informacje, jesteśmy w stanie
skonfigurować sobie wygląd tego okna w pliku `~/.config/openbox/rc.xml` za pomocą tej poniższej
zwrotki:

    <application class="Firefox" name="Navigator" type="normal">
    ...
    </application>

Opcji dotyczących położenia i wyglądu okna jest sporo i z tych ważniejszych możemy wymienić te
poniższe:

  - `focus` sprawi, że okno po tym jak się pojawi nie przechwyci automatycznie myszy i klawiatury.
  - `decor` odpowiada za obramowanie okna.
  - `shade` ustawia cienie okna.
  - `layer` precyzuje gdzie umieścić dane okno.
  - `skip_pager` odpowiada za konfigurację pager'a dla danego okna.
  - `skip_taskbar` odpowiada za konfigurację paska zadań dla danego okna.
  - `size` definiuje rozmiar okna za pomocą parametrów `width` i `height` .
  - `position` określa pozycję okna przy pomocy parametrów `x` i `y` .

Kompletny przykład użycia Openbox'a do zarządzania oknami został pokazany na przykładzie [osadzenia
konsoli urxvt na
pulpicie](/post/osadzanie-urxvt-na-pulpicie-przy-pomocy-openboxa/). Zachęcam zatem
do zajrzenia i obadania możliwości jakie daje Openbox w starciu z pojawiającymi się na ekranie
monitora okienkami.

## Tapeta pulpitu

Tapetę można ustawić albo korzystając z pakietu `nitrogen` , albo posługując się narzędziem `feh` .
My tutaj skorzystamy z tego drugiego sposobu. Poniżej mamy potrzebne pakiety:

    # aptitude install feh libjpeg-turbo-progs

Tworzymy teraz katalogu `~/wallpapers/` i wrzucamy do niego jakieś obrazki. Następnie wydajemy to
poniższe polecenie:

    $ feh --bg-scale '/home/morfik/wallpapers/tapeta.png'

Jeśli po wydaniu powyższego polecenia, tło pulpitu uległo zmianie, to możemy je dodać do autostartu
Openbox'a.

## Menadżer kompozycji compton

Prostota środowiska graficznego opartego jedynie o Openbox'a sprawia, że nawet po dostosowaniu
motywów pojawiających się na monitorze okienek czujemy pewien swojego rodzaju niedosyt związany z
efektami wizualnymi. Nie chodzi o coś co w znacznym stopniu obciąży nam procesor graficzny i przy
tym spowolni pracę całego systemu ale o te podstawowe funkcje jak, np. cienie okien czy lekkie
opóźnienie w ich pojawianiu się.

Istnieje szereg menadżerów kompozycji, które są bardzo zasobożerne i zwykle potrafią zjeść tyle
pamięci RAM, co całe środowisko z Openbox'em na pokładzie. Compton jest inny i znakomicie pasuje do
Openbox'em dodając mu nieco polotu i finezji.

Nie będę się tutaj rozpisywał na temat konfiguracji compton'a, a jedynie pokażę jak go uruchomić.
Wszystko czego nam potrzeba, to dopisanie do pliku autostartu Openbox'a tej poniższej linijki:

    compton --config /home/morfik/.config/compton.conf -b

Powyżej mamy oczywiście określony plik konfiguracyjny. Niemniej jednak, nie jest on wymagany i w
takim przypadku compton zaaplikuje ustawienia domyślne. Jeśli ktoś chciałby sobie ten plik utworzyć,
to [na moim gicie jest konfiguracja dla
compton'a](https://github.com/morfikov/files/blob/master/configs/home/morfik/.config/compton.conf),
z której ja obecnie korzystam. Posiadanie menadżera kompozycji eliminuje też problem z gotowaniem
się wręcz procesora podczas przeciągania okien po ekranie.
