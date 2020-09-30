---
author: Morfik
categories:
- Linux
date: "2016-01-08T20:12:46Z"
date_gmt: 2016-01-08 19:12:46 +0100
published: true
status: publish
tags:
- xserver
- debian
title: Menadżer logowania LightDM
---

Istnieje kilka sposobów na to, by uruchomić sesję graficzną Xserver'a. W przypadku, gdy nie
wykorzystujemy zaawansowanych środowisk graficznych, zwykle korzystamy z polecenia `startx`
wydawanego zaraz po zalogowaniu się na konsolę TTY. Niemniej jednak, nawet w tych najprostszych
środowiskach graficznych możemy pokusić się o instalację lekkiego menadżera okien, tak by nie
musieć konfigurować samodzielnie szeregu opcji sesji. Menadżery logowania (display managers) mają
na celu zrobienie dokładnie tego samego co pliki `~/.xinitrc` i `~/.xserverrc` , z tym, że w nieco
bardziej przyjaznej dla użytkownika formie. W tym wpisie przyjrzymy się nieco dokładniej
[menadżerowi LightDM](https://www.freedesktop.org/wiki/Software/LightDM/). Zostanie także omówiony
sposób blokowania aktywnej sesji Xserver'a bez potrzeby jej zamykania przy pomocy narzędzia
[light-locker](https://github.com/the-cavalry/light-locker).

<!--more-->
## Wiele menadżerów logowania w jednym systemie

W systemie możemy posiadać wiele różnych menadżerów logowania, choć zalecana jest instalacja jedynie
tego, z którego korzystamy. Jeśli chodzi zaś o pakiety potrzebne do instalacji menadżera LightDM i
narzędzia `light-locker` , to musimy doinstalować te poniższe:

    # aptitude install lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings light-locker \
    desktop-base accountsservice

Każde większe środowisko graficzne zwykle dostarcza swój własny menadżer logowania. Jest więc zatem
bardzo prawdopodobne, że w systemie mamy więcej niż jeden taki menadżer. Jeśli tak jest w istocie i
chcemy korzystać z LightDM, to możemy ustawić go jako domyślny za pomocą tego poniższego polecenia:

    # update-alternatives --config lightdm

W tych lżejszych środowiskach nie będziemy mieli takiego problemu.

## Konfiguracja LightDM

Konfiguracja LightDM sprowadza się do edycji kilku plików, które są zlokalizowane w katalogu
`/etc/lightdm/` . Oczywiście, domyślna konfiguracja jest w miarę optymalna i nie musimy nic
dostosowywać. Jeśli jednak chcemy dokonać kilku zmian, to musimy wiedzieć, że osobno się konfiguruje
wygląd tego menadżera logowania (tzw. greeter) i osobno sam menadżer logowania.

Jest wiele greeter'ów, które możemy sobie wgrać i każdy z nich ma innych plik konfiguracyjny. W tym
przypadku zainstalowaliśmy pakiet `lightdm-gtk-greeter` i konfiguracja tego greeter'a jest ulokowana
w pliku `lightdm-gtk-greeter.conf` , poniżej przykład:

    [greeter]
    background=/usr/share/images/desktop-base/login-background.svg
    theme-name=Adwaita
    icon-theme-name=matefaenzadark
    font-name=Sans 9
    xft-antialias=true
    xft-dpi=96
    xft-hintstyle=hintfull
    xft-rgba=none
    indicators=~host;~spacer;~clock;~spacer;~language;~session;~power
    clock-format=%Y-%m-%d %T
    #keyboard=
    #reader=
    position=100 100
    screensaver-timeout=10

Jak widzimy, mamy całkiem niezłe pole manewru. Możemy ustawić tapetę przy pomocy parametru
`background` . Możemy także dostosować motyw jak i style ikonek przy pomocy `theme-name` i
`icon-theme-name` . Mamy także możliwość określenia kroju i wielkości czcionki w `font-name` . Z
kolei opcje `xft-*` odpowiadają za poprawę jakości czcionek. Jeśli są one niejasne, to w [tym wpisie
została opisana konfiguracja
czcionek](/post/fontconfig-i-konfiguracja-czcionek-w-debianie/) i wyżej użyte
terminy zostały dokładnie wyjaśnione. Dalej mamy format zegara ( `clock-format` ), który możemy
sobie dostosować wedle własnego upodobania. Potrzebne zaś parametry można odczytać z `date --help` .
W tym przypadku zegar ma postać `2016-01-08 16:45:55` . Parametr `position` odpowiada za
umieszczenie okienka logowania w konkretnym miejscu na ekranie. Natomiast `screensaver-timeout`
wyłączy ekran po 10 sekundach nieaktywności.

Oczywiście nie musimy ręcznie konfigurować greeter'a. Możemy zdać się na graficzne narzędzie
`lightdm-gtk-greeter-settings` , które jest w stanie ustawić praktycznie wszystkie wyszczególnione
wyżej opcje:

![](/img/2016/01/1.lightdm-greeter-ustawienia-gui.png#medium)

Jeśli chodzi zaś o konfigurację samego menadżera logowania, to jest ona bardziej skomplikowana.
Dlatego też plik `lightdm.conf` jest nieco rozbudowany. Poniżej przykład konfiguracji:

    [LightDM]
    greeter-user=lightdm
    minimum-display-number=0
    minimum-vt=7
    logind-check-graphical=true
    log-directory=/var/log/lightdm
    run-directory=/var/run/lightdm
    cache-directory=/var/cache/lightdm

    [Seat:*]
    greeter-session=lightdm-gtk-greeter
    greeter-hide-users=false
    greeter-allow-guest=false
    greeter-show-manual-login=true
    greeter-show-remote-login=true
    user-session=openbox
    allow-user-switching=true
    allow-guest=false
    autologin-guest=false
    autologin-user-timeout=0
    autologin-in-background=false
    #xserver-command=X -listen tcp -auth "$HOME/.Xauthority"
    #xserver-allow-tcp=true

    [XDMCPServer]

    [VNCServer]

To w tym pliku jesteśmy w stanie dostosować szereg aspektów pracy samego Xserver'a, wliczając w to
opcje procesu `X` . Wyżej mamy wyłączone autologowanie użytkowników przy starcie systemu oraz nie
zezwalamy na zalogowanie się gościom, czyli by skorzystać z tego systemu, trzeba się świadomie
zalogować podając login i hasło. W parametrze `greeter-session` podajemy nazwę wykorzystywanego
greeter'a. Jeśli chodzi zaś o `minimum-vt` , to określa on, na której konsoli TTY zaczynać podnosić
sesje Xserver'a. Z reguły 1-6 są tekstowe, a graficzne zaczynają się od 7. Mamy także opcję
`minimum-display-number` , która odpowiada za minimalny numer jaki może zostać przypisany sesji
graficznej (widoczny w zmiennej `$DISPLAY` ). Dalej mamy już tylko `greeter-user` , który odpowiada
za uruchomienie greeter'a na prawach innego użytkownika niż root.

W konfiguracji LightDM mamy także szereg innych wykomentowanych wpisów i w takim wypadku są
dobierane wartości domyślne. Wyżej przytoczyłem dwa z nich, które są w stanie skonfigurować jedną z
bardziej przydanych cech Xserver'a. Chodzi oczywiście o edycję parametrów procesu `X` i umożliwienie
nawiązywania połączeń sieciowych.

Mamy także możliwość skonfigurowania samych użytkowników. Chodzi o to, że nie koniecznie chcemy aby
wszyscy użytkownicy w systemie mogli logować się i korzystać z sesji graficznej. W tym celu
korzystamy z pliku `users.conf` , poniżej przykład:

    [UserList]
    minimum-uid=1000
    hidden-users=nobody nobody4 noaccess
    hidden-shells=/bin/false /usr/sbin/nologin

Ta powyższa konfiguracja zezwala na wyświetlenie na liście użytkowników jedynie tych loginów, które
mają ID większe lub równe `1000`. Dodatkowo, użytkownicy, którzy mają nazwy `nobody` , `nobody4` i
`noaccess` również się na niej nie pojawią. Odfiltrowane zostaną również wszystkie konta, które mają
ustawionego nieprawidłowego shell'a. W taki oto sposób jesteśmy w stanie wyświetlić tylko te konta,
które mają prawo się logować do sesji graficznej.

## AccountsService

Pewne opcje menadżera LightDM można dostosować za pomocą pakietu `accountsservice` . W tym przypadku
postaramy się go wykorzystać do ustawienia avatara dla konkretnego konta. Teoretycznie powinna być
możliwość dokonania tego za pomocą pliku `~/.face` lub `~/.face.icon` . Nie zawsze jednak ten sposób
działa. Natomiast jeśli chodzi o `accountsservice` , to nie ma tu żadnych problemów.

Tworzymy zatem plik `/var/lib/AccountsService/users/morfik` , gdzie `morfik` oznacza nazwę konta,
dla którego chcemy ustawić avatar. Do tego pliku dodajemy poniższą treść:

    [User]
    Language=en_US.UTF-8
    XSession=lightdm-xsession
    Icon=/var/lib/AccountsService/icons/morfik.icon
    SystemAccount=false

Opcje `Language` określamy na podstawie [ustawionych w systemie
locale](/post/jezyk-polski-w-srodowisku-graficznym/). W `XSession` definiujemy
sesje graficzną, która ma być odpalana dla tego użytkownika. Jeśli nie wiemy jakie sesje mamy do
wyboru, to możemy zawsze zajrzeć do katalogu `/usr/share/xsessions/` . Dalej mamy `Icon` , która
właśnie odpowiada za avatar. Ostatnia pozycja, tj. `SystemAccount` określa dane konto jako
systemowe, czyli takie, które nie ma możliwości logowania się do sesji graficznej.

Następnie wgrywamy do katalogu `/var/lib/AccountsService/icons/` plik o nazwie zdefiniowanej wyżej.
Obrazki powinny mieć 96x96 pikseli być w formacie `.png` .

Większość środowisk graficznych jest w stanie ten proces zautomatyzować w swoich panelach
sterowania. Ta powyższa instrukcja jest głównie do środowisk, które są oparte o menadżery okien, np.
openbox.

## Narzędzie light-locker

Narzędzie `light-locker` jest w stanie zablokować sesję graficzną, gdy ta jest nieużywana przez
jakiś dłuższy czas. Może także zostać zainicjowane przy pomocą skrótu klawiszowego. Domyślny czas,
po którym `light-locker` zostanie uruchomiony jest określona przez DPMS, które są konfigurowane
bezpośrednio w opcjach Xserver'a. Ta kwestia była lekko zarysowana we wpisie na temat [jak
skonfigurować sobie monitor pod
linux'em](/post/monitor-i-jego-konfiguracja-pod-linuxem/). W dużym skrócie, musimy
utworzyć plik `20-monitor.conf` w katalogu `/etc/X11/xorg.conf.d/` i dodać w nim tę poniższą treść:

    Section "ServerLayout"
          Identifier "Main"
          Screen      0 "Screen0"
    EndSection

    Section "ServerFlags"
          Option "DefaultServerLayout" "Main"
          Option "BlankTime" "10"
          Option "StandbyTime" "10"
          Option "SuspendTime" "10"
          Option "OffTime" "10"
    EndSection

Czasy są w minutach i całą konfigurację można podejrzeć wydając w terminalu polecenie `set -q` .
Czyli po 10 minutach nieużywania (brak ruchów myszy i wciskania przycisków na klawiaturze), ekran
zostanie wyłączony przez Xserver i do tego zablokowany przez `light-locker` .

W sporej części przypadków, tego typu zachowanie jest jak najbardziej pożądane ale co jeśli oglądamy
jakiś film w VLC? Na dobrą sprawę, szereg player'ów video jest w stanie deaktywować tę blokadę na
czas odtwarzania filmu i nie musimy się o nic martwić. Problem pojawia się w momencie odtwarzania
filmów w przeglądarkach internetowych, np. w serwisie youtube. Oczywiście nic nie stoi na
przeszkodzie, by ten materiał przesłać do playera video i w nim go oglądać, ewentualnie korzystać z
takich linux'owych wynalazków jak [mps-youtube](https://github.com/mps-youtube/mps-youtube) .

Pamiętajmy też, by odpowiednio sobie [skonfigurować klawisz
SysRq](/post/aktywacja-i-konfiguracja-klawisza-sysrq/), tak by czasem [OOM-Killer
nie zdjął przez przypadek blokady sesji
Xserver'a](/post/bezpieczenstwo-xservera-pod-linuxem/).
