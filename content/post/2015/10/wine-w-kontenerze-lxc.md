---
author: Morfik
categories:
- Linux
date: "2015-10-30T17:13:58Z"
date_gmt: 2015-10-30 15:13:58 +0100
published: true
status: publish
tags:
- pulseaudio
- xserver
- lxc
- wine
GHissueID: 195
title: Wine w kontenerze LXC
---

Każdy kto zmienił architekturę systemu z i386 na amd64 raczej nie dostrzegł większej różnicy w
operowaniu na którejś z nich. Czasem jedynie pakiety w nazwie mają 64 zamiast 32. Jest natomiast
jedna rzecz, która drażni chyba każdego. Mowa tutaj o [projekcie Wine](https://www.winehq.org/).
Wine nie umie obsługiwać natywnie serwera dźwięku PulseAudio. Do tego dochodzi jeszcze problem,
który związany jest z tymi wszystkimi pakietami 32 bitowymi, które trzeba zainstalować. I w ten
sposób nasz system staje się bardziej multiarch niż amd64. W tym wpisie postaramy się przy pomocy
kontenera LXC odizolować Wine od całej reszty systemu operacyjnego, tak by nie musieć wgrywać do
niego całej masy zbędnych bibliotek.

<!--more-->
## Wine 32-bit, 64-bit i kontener LXC

Na stronie projektu Wine jest informacja, że [Wine dorobił się
wersji 64-bitowej](https://wiki.winehq.org/FAQ#Is_there_a_64_bit_Wine.3F). Z której wersji zatem
powinniśmy korzystać? Generalnie rzecz biorąc, wszystkie wersje aplikacji, tj. 16, 32 i 64-bitowe,
będą działać na Wine 64-bit.

W kontenerze LXC możemy zainstalować inny system niż ten macierzysty. Dlatego też możemy posiadać
system w wersji 64-bit, a w kontenerze zainstalować Wine 32-bitowego. To jaka wersja Wine zostanie
zainstalowana zależy głównie od tego jakie aplikacje chcemy odpalać na nim. Jeśli są to tylko
32-bitowe aplikacje, to możemy się ograniczyć jedynie do 32-bitowej wersji. W każdym innym
przypadku, lepiej jest zainstalować Wine w wersji 64-bit.

Z kontenerów LXC korzystamy z powodu oszczędności zasobów systemowych. W przypadku mechanizmów
wirtualizacji oferowanych, np. przez VirtualBox, występują dwa problemy. Po pierwsze trzeba stworzyć
maszynę wirtualną i przydzielić jej określone zasoby. Po drugie, procesor musi wspierać
wirtualizacje. Do tego dochodzi jeszcze marnowanie zasobów na system operacyjny, który w takiej
maszynie rezyduje. W przypadku kontenerów LXC nie musimy się o te wszystkie rzeczy martwić.

## Konfiguracja kontenerów LXC

Dalsza cześć tego wpisu zakłada, że mamy w systemie zainstalowane odpowiednie pakiety oraz, że
[utworzyliśmy już i wstępne skonfigurowaliśmy kontener
LXC](/post/konfiguracja-kontenerow-lxc/).

Mając przygotowany kontener, możemy przejść do jego konfiguracji. Edytujemy zatem plik
konfiguracyjny kontenera. W moim przypadku jest to `/media/Kabi/lxc_machines/wine/config` . Musimy w
nim określić odpowiednie urządzenia, do których kontener musi mieć dostęp. Wine i uruchamiane za
jego pomocą aplikacje mają w zamierzeniu uzyskać dostęp do dźwięku oraz do Xserver'a. Ten ostatni
zaś implikuje dostęp do grafiki, myszy oraz klawiatury.

W przypadku kart graficznych, kluczową rolę odgrywają sterowniki oraz producent karty. Ja korzystam
z grafiki Intel'a ale może się zdarzyć tak, że będziemy posiadać kartę nvidii. W tym drugim
przypadku będziemy mieli do wyboru albo zamknięte sterowniki, albo te otwarte z modułem `nouveau` .
Na zamkniętych sterownikach nvidii, mamy urządzenia `/dev/nvidia*` , na otwartych jest katalog
`/dev/dri/` i w nim kilka urządzeń.

Poniżej są wpisy, które zezwalają na dostęp do tych wszystkich urządzeń, z których kontener powinien
swobodnie korzystać w przypadku Wine:

    # nvidia
    #lxc.cgroup.devices.allow = c 195:0 rwm
    #lxc.cgroup.devices.allow = c 195:255 rwm

    # dri -- card0 controlD64 renderD128
    lxc.cgroup.devices.allow = c 226:0 rwm
    lxc.cgroup.devices.allow = c 226:64 rwm
    lxc.cgroup.devices.allow = c 226:128 rwm

    # fb0
    lxc.cgroup.devices.allow = c 29:0 rwm

    # mouse and keyboard
    lxc.cgroup.devices.allow = c 13:* rwm

    # sound
    lxc.cgroup.devices.allow = c 116:* rwm

Możemy teraz podnieść kontener przy pomocy tego poniższego polecenia:

    # lxc-start -n winehq -P /media/Kabi/lxc_machines/

Logujemy się na konto administratora i tworzymy zwykłego użytkownika w kontenerze. Dodajemy go także
do grup `audio` oraz `video` :

    root@winehq:~# adduser morfik
    root@winehq:~# adduser morfik video
    root@winehq:~# adduser morfik audio

### Tworzenie plików urządzeń

`udev` nie działa z kontenerami LXC, wobec czego nie mamy dostępu do wymaganych urządzeń, tych
których określiliśmy wyżej w pliku konfiguracyjnym kontenera. Skoro `udev` nie może ich stworzyć,
sami musimy to zrobić. Urządzenia, które nas interesują znajdują się w: `/dev/input/*` (klawiatura
oraz mysza), `/dev/snd/*` (dźwięk) oraz `/dev/nvidia*`/`/dev/dri/*` (grafika, w zależności od
sterowników).

Potrzebne urządzenia tworzymy przy pomocy `mknod` . Zaglądamy zatem do katalogu `/dev/` na hoście i
ustalamy ich numery przy pomocy `ls -al` . Następnie będąc w kontenerze, wydajemy kolejno poniższe
polecenia:

    root@winehq:/dev# mkdir  /dev/input
    root@winehq:/dev# mknod -m 666 /dev/input/mice c 13 63
    root@winehq:/dev# mknod -m 666 /dev/input/mouse0 c 13 32
    root@winehq:/dev# mknod -m 666 /dev/input/event0 c 13 64
    root@winehq:/dev# mknod -m 666 /dev/input/event1 c 13 65
    root@winehq:/dev# mknod -m 666 /dev/input/event2 c 13 66
    root@winehq:/dev# mknod -m 666 /dev/input/event3 c 13 67
    root@winehq:/dev# mknod -m 666 /dev/input/event4 c 13 68
    root@winehq:/dev# mknod -m 666 /dev/input/event5 c 13 69
    root@winehq:/dev# mknod -m 666 /dev/input/event6 c 13 70
    root@winehq:/dev# chmod 600 /dev/input/*

    root@winehq:/dev# mkdir /dev/snd
    root@winehq:/dev# mknod -m 666 /dev/snd/controlC0 c 116 11
    root@winehq:/dev# mknod -m 666 /dev/snd/midiC0D0 c 116 2
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D0c c 116 10
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D0p c 116 9
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D1c c 116 8
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D1p c 116 7
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D2c c 116 6
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D2p c 116 5
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D3c c 116 4
    root@winehq:/dev# mknod -m 666 /dev/snd/pcmC0D3p c 116 3
    root@winehq:/dev# mknod -m 666 /dev/snd/seq c 116 1
    root@winehq:/dev# mknod -m 666 /dev/snd/timer c 116 33
    root@winehq:/dev# chown root:audio /dev/snd/*

    root@winehq:/dev# mkdir /dev/dri
    root@winehq:/dev# mknod -m 666 /dev/dri/card0 c 226 0
    root@winehq:/dev# mknod -m 666 /dev/dri/controlD64 c 226 64
    root@winehq:/dev# mknod -m 666 /dev/dri/renderD128 c 226 128
    root@winehq:/dev# chown root:video /dev/dri/*

    root@winehq:/dev# mknod -m 666 /dev/fb0 c 29 0
    root@winehq:/dev# chown root:video /dev/fb0

## Połączenie z Xserver'em

Podstawową konfigurację mamy z głowy. Teraz trzeba przygotować Xserver do pracy. Na debianie Xserver
startuje z opcją `-nolisten tcp` , co powoduje, że zdalne połączenia do tego Xserver'a są niemożliwe
i trzeba usunąć ten parametr. Jeśli korzystamy z DM, czyli graficznego menadżera logowania, to
trzeba poszukać w opcjach tegoż programu jak ustawić odpowiednie parametry dla procesu `X` . W
przypadku LightDM trzeba edytować plik `/etc/lightdm/lightdm.conf` i dostosować w nim te dwa
poniższe parametry:

    xserver-command=X -auth "$HOME/.Xauthority" -listen tcp
    xserver-allow-tcp=true

W przypadku odpalania Xserver'a przy pomocy `startx` , trzeba edytować plik
`/etc/X11/xinit/xserverrc` i przerobić go do poniższej postaci:

    exec /usr/bin/X -auth "$HOME/.Xauthority" -listen tcp "$@"

Po zresetowaniu sesji graficznej, Xserver powinien zacząć nasłuchiwać na porcie `6000` :

    # netstat -an | grep -F 6000
    tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN
    tcp6       0      0 :::6000                 :::*                    LISTEN

### Parę słów o xhost i xauth

To, że zezwalamy Xserver'owi na akceptowanie połączeń przez sieć, nie znaczy automatycznie, że on
zacznie nawiązywać połączenia z każdym kto tego będzie chciał. Jest wiele mechanizmów ochrony przed
nieautoryzowanym dostępem ale tutaj skupimy się na tych oferowanych przez Xserver, tj. `xhost` oraz
`xauth` .

`xhost` jest podatny na szereg ataków, np. nie rozróżnia on użytkowników zdalnych, czyli jeśli mamy
adres IP i na nim wielu użytkowników, to możemy zezwolić albo im wszystkim na połączenie, albo
żadnemu. `xhost` nie sprawdza też czy ten kto się łączy jest tym za kogo się podaje. Jeśli jednak
mamy zaufaną sieć, a tak jest w tym przypadku, możemy zezwolić adresowi IP kontenera LXC
(192.168.10.20) na podłączenie się do sesji Xserver'a na hoście. W tym celu na głównej maszynie
wydajemy poniższe polecenie:

    $ xhost +192.168.10.20

Można również sprecyzować nazwę ustawioną przez plik `/etc/hosts` . Jeśli teraz sprawdzimy jakie
hosty mogą nawiązywać połączenia z naszym Xserver'em, powinniśmy ujrzeć coś podobnego do tego
poniżej:

    $ xhost
    access control enabled, only authorized clients can connect
    INET:winehq.mhouse.lh
    SI:localuser:morfik

Ustawienia nie są trwałe i znikają po zamknięciu sesji. Dlatego dobrze jest dodać powyższą linijkę
do autostartu na maszynie hosta.

Innym rozwiązaniem, o wiele bezpieczniejszym są ciasteczka Xserver'a, trzymane z reguły w katalogu
użytkownika w pliku `~/.Xauthority` . Nie będę tutaj opisywał tego mechanizmu, bo w tym przypadku
jest on nam do niczego nie potrzebny. Jeśli ktoś jest zainteresowany tym tematem, to może rzucić
okiem na osobny [wpis poświęcony
xauth](/post/xauth-i-xhost-na-strazy-bezpieczenstwa-xservera/). W każdym razie,
`xhost` nam w zupełności wystarczy.

### Zmienna $DISPLAY

Dostęp do kontenera mamy zagwarantowany, jednak by moc przesłać dane do Xserver'a musimy wskazać
systemowi gdzie ten Xserver się znajduje. Robimy to przez wyeksportowanie zmiennej `$DISPLAY`
wewnątrz kontenera. Musi ona zawierać adres IP lub nazwę hosta, na którym nasłuchuje Xserver. W tym
przypadku jest to `192.168.1.150` lub `morfikownia.mhouse.lh` :

    morfik@winehq:~$ export DISPLAY="192.168.1.150:0"

Dźwięk i PulseAudio

U mnie dźwięk na Wine zawsze stwarzał problemy i nigdy mi się nie udało go skonfigurować tak by
współpracował z PulseAudio. Tak samo jest i w tym przypadku. Jeśli odpalę grę w kontenerze, to
Wine zajmuje urządzenie dźwiękowe i nie da rady nic odtwarzać w tym czasie na maszynie hosta. Jeśli
jednak odpalę Wine na standardowych ustawieniach i nie będę nic odtwarzał na maszynie hosta w tym
czasie, to dźwięk jak najbardziej będzie działał.

Jeśli chcemy korzystać z dźwięku na Wine w kontenerze, to na hoście musimy załadować dodatkowy moduł
dla PulseAudio i oznaczyć adresy IP jako zaufane. Chodzi oczywiście o te, które będą mogły się
łączyć z naszym serwerem dźwięku. Moduły można dodawać i usuwać przy pomocy `pactl` bez
konieczności restartowania PulseAudio:

    $ pactl load-module module-native-protocol-tcp auth-ip-acl=192.168.10.0/24
    $ pactl unload-module module-native-protocol-tcp

Tak załadowany moduł nie jest trwały i trzeba by go załadować za każdym razem gdy restartujemy
PulseAudio. Możemy za to dodać konfigurację dla tego modułu do pliku `/etc/pulse/default.pa` :

    load-module module-native-protocol-tcp auth-ip-acl=192.168.10.0/24

To cała praca jeśli chodzi o serwer PulseAudio.

W kontenerze LXC nie trzeba nawet tam instalować PulseAudio. Po instalacji Wine i sterowników do
grafiki, jedyny pakiet związany z PulseAudio jaki będziemy mieć w kontenerze, to `libpulse0` . Szuka
on zmiennej `$PULSE_SERVER` w środowisku i jeśli znajdzie, kieruje tam dźwięk. Musimy ją zatem
wyeksportować:

    morfik@winehq:~$ export PULSE_SERVER=192.168.1.150

## Instalacja Wine i sterowników do grafiki

Praktycznie całą konfigurację kontenera LXC pod Wine mamy już z głowy. Przyszła pora by zainstalować
Wine oraz sterowniki do grafiki w kontenerze:

    root@wine:~# apt-get install wine xserver-xorg-video-intel

Jeśli wszystkie kroki przeprowadziliśmy prawidłowo, po wydaniu polecenia `winecfg` w kontenerze,
powinniśmy zobaczyć okno z opcjami konfiguracyjnymi Wine:

![](/img/2015/10/1.kontenerl-lxc-wine.png#huge)
