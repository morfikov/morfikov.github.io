---
author: Morfik
categories:
- Linux
date: "2016-01-11T21:49:37Z"
date_gmt: 2016-01-11 20:49:37 +0100
published: true
status: publish
tags:
- dźwięk
- debian
title: Konfiguracja serwera dźwięku PulseAudio
---

Wielu użytkowników linux'a nie przepada zbytnio za PulseAudio, bo ten z jakiegoś powodu sprawia u
nich same kłopoty. U mnie ten serwer dźwięku działa przyzwoicie i zwykle nie ma z nim żadnych
problemów. Obecnie ten projekt jest już na tyle dojrzały, że te większe środowiska graficzne
zwyczajnie polegają na nim w zależnościach. Jeśli jednak [instalowaliśmy debiana za pomocą narzędzia
debootstrap]({{< baseurl >}}/post/instalacja-debiana-z-wykorzystaniem-debootstrap/) i
[konfigurowaliśmy osobno graficzną sesję
Xserver'a]({{< baseurl >}}/post/konfiguracja-xservera-na-debianie-xorg/), [menadżer logowania
LightDM]({{< baseurl >}}/post/menadzer-logowania-lightdm/) czy też [menadżer okien
Openbox]({{< baseurl >}}/post/menadzer-okien-openbox/), to raczej zależy nam na minimalnym
środowisku, które może się zwyczajnie obejść bez PulseAudio. Niemniej jednak, PulseAudio ma kilka
ciekawych bajerów, których ALSA nie posiada. By się nie rozpisywać zbytnio, mogę wspomnieć choćby o
możliwości[przesyłania dźwięku przez
sieć]({{< baseurl >}}/post/pulseaudio-i-przesylanie-dzwieku-przez-siec/), czy też o takim
ficzerze jak [normalizacja głośności]({{< baseurl >}}/post/normalizacja-glosnosci-w-pulseaudio/).
Są to głównie zabawki dla nieco bardziej zaawansowanych użytkowników i gdy ich nie potrzebujemy, to
serwer dźwięku nam się raczej do niczego nie przyda. Warto jednak się zaznajomić z tym nieco
bardziej zaawansowanym kawałkiem oprogramowania i w tym wpisie postaramy się nieco omówić instalację
i konfigurację tego serwera dźwięku.

<!--more-->
## Instalacja PulseAudio

Gdy nie korzystamy z pełnego środowiska graficznego, np. GNOME, musimy sami zatroszczyć się o
zainstalowanie potrzebnych pakietów. Poza samym serwerem dźwięku PulseAudio, doinstalujemy jeszcze
kilka dodatków, tak by dźwięk działał nam bez zarzutu w praktycznie każdej aplikacji, którą odpalimy
w naszym systemie. Tak więc, potrzebujemy te poniższe pakiety:

    # aptitude install pulseaudio pulseaudio-utils pavucontrol rtkit swh-plugins \
    pulseaudio-module-gconf pulseaudio-module-x11 \
    gstreamer1.0-alsa gstreamer1.0-pulseaudio gstreamer1.0-plugins-base gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-libav \
    alsa-utils libao4 \
    phonon phonon-backend-vlc \
    libnotify-bin xfce4-notifyd volumeicon-alsa

W pakiecie `pavucontrol` mamy graficzny mikser, z poziomu którego będziemy w stanie zarządzać
poziomem głośności każdej aplikacji, która się podłączy do PulseAudio. Wygląda to mniej więcej tak:

![]({{< baseurl >}}/img/2016/01/1.pulseaudio-volume-control-pavucontrol.png)

Z kolei pakiet `rtkit` zajmie się ustawieniem priorytetu dla demona dźwięku, tak by te bardziej
żarłoczne aplikacje nie zdusiły nam czasem dźwięku. Pakiety, które mają w nazwie
`pulseaudio-module` odpowiadają za dodatkowe moduły dla PulseAudio, podobnie jak `swh-plugin` ,
który umożliwia zaimplementowanie normalizacji głośności. Przydadzą nam się też podstawowe
narzędzia dla ALSA (np. `alsamixer` ). Dalej mamy jeszcze pakiet `phonon` wraz z backend'em VLC,
który ma za zadanie zbierać dźwięk z aplikacji powiązanych ze środowiskami KDE (interfejs QT). Z
kolei zaś ostatnia linijka zapewni naszemu systemowi notyfikacje dźwięku, np. przy przyciszaniu,
zgłaszaniu czy mutowaniu.

## Kolejność kart dźwiękowych

Czasami może się zdarzyć tak, że po uruchomieniu aplikacji i puszczeniu filmu czy muzyki, nie
słyszymy nic w głośnikach. Niby aplikacja odtwarza dźwięk ale go nie słychać. Nie ma też przy tym
żadnych błędów. Zwykle jest to za sprawą posiadania kilku kart dźwiękowych w systemie, np. jednej
wbudowanej w płytę główną i drugiej zewnętrznej. W takim przypadku system do końca nie wie, z której
karty my chcemy korzystać i ta, która zostanie wykryta jako pierwsza jest używana jako domyślna. W
takiej sytuacji albo trzeba zablokować jedną z kart i używać tylko tej drugiej, albo też ustawić
kolejność kart, tak by nie zamieniały się one nam miejscami podczas następnych startów systemu.

W przypadku chęci posiadania kilku urządzeń dźwiękowych, trzeba pierw ustalić jakie moduły w kernelu
są odpowiedzialne za ich obsługę. Pomóc nam w tym mogą poniższe polecenia:

    $ cat /proc/asound/cards
     0 [CA0106         ]: CA0106 - CA0106
                          Audigy SE [SB0570] at 0xa000 irq 19
     1 [Intel          ]: HDA-Intel - HDA Intel
                          HDA Intel at 0xe3100000 irq 42

    $ cat /proc/asound/modules
     0 snd_ca0106
     1 snd_hda_intel

Jak widzimy wyżej, każda z kart ma przypisany inny numer, odpowiednio `0` i `1` . By każda z tych
kart miała taki sam numerek cały czas, musimy stworzyć sobie plik `/etc/modprobe.d/alsa.conf` i
dodać w nim tę poniższą treść:

    options snd-ca0106 index=0
    options snd-hda-intel index=1

Można także skorzystać z opcji `slots` (nowsza metoda), przykładowo:

    options snd slots=snd-hda-intel,snd-ca0106

W ten sposób rezerwujemy dwa numerki (0 i 1) dla tych poszczególnych kart. Jeśli podłączymy trzecią
kartę do komputera, nie mając przy tym żadnej aktywnej karty dźwiękowej, to zostanie jej przypisany
numer 2.

Niemniej jednak, może się zdarzyć taka sytuacja, że kilka kart będzie obsługiwanych przez ten sam
moduł i jak w takiej sytuacji ustawić kolejność? W tym przypadku trzeba będzie posłużyć się vendor i
product ID, które możemy wyciągnąć z `lspci -nn` czy też `lsusb` w zależności od rodzaju karty.
Mając ID, w tym przypadku `8086:3b56` oraz `0d8c:000c` tworzymy opcję dla modułu w poniższy sposób:

    options snd-hda-intel index=0,1 vid=0x8086,0x0d8c pid=0x3b56,0x000c

I teraz odwołując się do pierwszej karty mam pewność, że dźwięk trafia w pożądane miejsce i nie leci
czasem gdzieś indziej. Więcej informacji na temat kolejności kart dźwiękowych można znaleźć [pod tym
linkiem](http://alsa.opensrc.org/MultipleCards#Reordering_the_driver_for_a_particular_card).

## Konfiguracja PulseAudio

Pliki konfiguracyjne PulseAudio znajdują się w katalogu `/etc/pulse/` . Jest to konfiguracja
systemowa i może zostać nadpisana przez pliki użytkownika, które można umieścić w katalogu
`~/.pulse/` . W ten sposób każdy użytkownik w systemie może mieć skonfigurowanego demona osobno. W
tym przypadku skupimy się na plikach systemowych.

Przede wszystkim, PulseAudio może być uruchamiany w kilku trybach: systemowy i użytkownika. Nie
zaleca się uruchamiania go w trybie systemowym, bo to może [powodować problemy z
bezpieczeństwem](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/WhatIsWrongWithSystemWide/).
Dlatego też szereg dystrybucji nie korzysta z tego trybu. Demon PulseAudio jest odpalany
automatycznie wraz ze startem środowiska graficznego, tuż po zalogowaniu się konkretnego
użytkownika. Taki proces działa na prawach danego użytkownika, a nie jako root. Może to w pewnych
sytuacjach powodować [brak
dźwięku]({{< baseurl >}}/post/brak-dzwieku-przy-nieaktywnej-sesji-logowania/), zwłaszcza, gdy
nie jesteśmy członkami grupy `audio` i posiadamy systemd. Nie jest to jednak jakiś poważny problem i
można go bardzo łatwo rozwiązać (tak jako to zostało zrobione w podlinkowanym wpisie).

### Autostart

Idąc dalej, PulseAudio ma także wbudowany mechanizm autostartu w przypadku, gdy jego proces zostanie
ubity. Czasem nie jest to pożądane, dlatego też przydałoby się na fazy testów, wyłączyć ten ficzer.
Możemy to zrobić za pomocą pliku `/etc/pulse/client.conf` przez usunięcie znaku `;` z początku
poniższej linijki:

    autospawn = no

Od tej chwili możemy swobodnie restartować demona za pomocą `pulseaudio -k` i `pulseaudio -D` .
Pamiętajmy, że te dwa polecenia trzeba wydawać z konta zwykłego użytkownika.

### Konfiguracja demona

Drugim plikiem, do którego musimy zajrzeć przy konfiguracji PulseAudio jest `/etc/pulse/daemon.conf`
. To za jego sprawą konfigurujemy demona, który nasłuchuje żądań od aplikacji dźwiękowych. Poniżej
jest rozpiska kilku opcji, które możemy określić w tym pliku.

  - `daemonize` -- PulseAudio uruchomi się w trybie demona.
  - `fail` -- jeśli jakaś dyrektywa w skrypcie `default.pa` zwróci błąd, to PulseAudio się nie
    uruchomi.
  - `allow-module-loading` -- zezwala na ładowanie modułów w trackie pracy demona.
  - `system-instance ` -- uruchamia PulseAudio w trybie systemowym.
  - `cpu-limit` , `high-priority ` , `realtime-scheduling` , `realtime-priority` , `nice-level` oraz
    `rlimit-*` -- odpowiadają za ustawienie odpowiednich priorytetów dla demona PulseAudio.
  - `default-script-file` oraz `load-default-script-file` -- określają czy oraz jaki skrypt
    załadować na starcie demona.
  - `flat-volumes` -- ustawiony może powodować problemy przy zmianie poziomu głośności w przypadku,
    gdy aktywnych jest kilka aplikacji odtwarzających dźwięk.
  - `default-sample-rate` oraz `alternate-sample-rate` -- określają częstotliwość próbkowania.
  - `default-sample-channels` oraz `default-channel-map` -- odpowiadają za ilość i mapowanie
    kanałów.

Cały plik jest dostępny [na moim
gicie](https://github.com/morfikov/files/tree/master/configs/etc/pulse). Standardowe ustawienia nie
powinny sprawiać problemów. Jeśli jednak coś nam nie gra do końca tak jak byśmy tego oczekiwali, to
zawsze możemy zajrzeć w ten przykładowy plik konfiguracyjny.

### Skrypt dla PulseAudio (default.pa)

Na dobrą sprawę mamy dwa skrypty, które są dostarczane wraz z PulseAudio. Jeden z nich to
`default.pa` i to jest skrypt dla trybu użytkownika. Drugi zaś to `system.pa` dla trybu systemowego.
My będziemy korzystać jedynie z tego pierwszego.

W skrypcie `default.pa` są zawarte informacje, jakie moduły załadować przy starcie demona i jakie
wartości im ustawić. To tutaj mamy między innymi określone zachowanie PulseAudio w przypadku, np.
zapisywania poziomu głośności czy przełączania się z głośników na słuchawki. W całej masie
przypadków nie będziemy musieli ruszać tego pliku.

## Konfiguracja systemu pod PulseAudio

Obecnie system powinien w dużej mierze sam dostosować sobie poszczególne aplikacje, tak by
korzystały z PulseAudio automatycznie. Tak przynajmniej jest w przypadku tych bardziej
rozbudowanych środowisk graficznych. Nie zawsze jednak ma to miejsce, gdy stawiamy sobie bardzo
minimalne środowisko oparte jedynie o prosty menadżer okien. Poniżej jest kilka miejsc, w które
warto zajrzeć, jeśli dźwięk nam nie działa tak jak powinien.

W pliku `/etc/libao.conf` ustawiamy:

    default_driver=pulse

W pliku `/etc/openal/alsoft.conf` zaś dajemy:

    drivers = pulse,alsa

Jeśli chodzi natomiast o wszelkie odtwarzacze audio/video, to w opcjach każdego z nich mamy
możliwość ustawienia wyjścia dźwięku. Z reguły to wyjście jest wykrywane automatycznie i nie
musimy manualnie przestawiać go na `pulse` czy `pulseaudio` . Niemniej jednak, jeśli chcemy
przestawić sobie wyjście dźwięku, to nic nie stoi na przeszkodzie, by to zrobić. Poniżej przykład:

![]({{< baseurl >}}/img/2016/01/2.pulseaudio-smplayer-config.png)

Trzeba jednak wziąć pod uwagę, że nie wszystkie aplikacje mają zaimplementowaną natywną obsługę
PulseAudio i w takim przypadku mogą się pojawić problemy z dźwiękiem. Część z aplikacji może mieć
też dedykowane wtyczki, które musimy doinstalować osobno, by dźwięk w tych aplikacjach działał jak
trzeba. Warto też zajrzeć [pod ten
link](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/PerfectSetup/), który
zawiera konfigurację szeregu aplikacji pod PulseAudio.

## Zajmowanie karty przez PulseAudio

PulseAudio domyślnie skonfigurowany jest w taki sposób, by zajmować urządzenie dźwiękowe, na którym
operuje. W ten sposób aplikacje, które mają ustawione wyjście dźwięku na ALSA, nie mogą zwyczajnie
nic wysłać do karty dźwiękowej, bo PulseAudio ją blokuje. Jako, że nie wszystkie aplikacje mają
zaimplementowaną obsługę PulseAudio, a emulacja ALSA nie zawsze działa poprawnie, to możemy
skonfigurować ten serwer dźwięku, tak by nie zajmował całego urządzenia dla siebie. W tym celu
musimy edytować plik `/etc/pulse/default.pa` i zmienić w nim te poniższe linijki:

    load-module module-alsa-sink device=dmix
    load-module module-alsa-source device=dsnoop
    # load-module module-udev-detect
    # load-module module-detect

## Problemy z PulseAudio

W przypadku, gdy doświadczamy problemów z dźwiękiem, to można zajrzeć w log PulseAudio. Log zaś
możemy sporządzić przy pomocy tego poniższego polecenia:

    $ LANG=C pulseaudio -vvvv --log-time=1 > ~/pulseverbose.log 2>&1

Pamiętajmy jednak, że pierw trzeba zatrzymać demona ( `pulseaudio -k` ) i dopiero wtedy wprowadzić
powyższe polecenie. Log będzie dość spory. Możemy także go zredukować przez odpowiednie ustawienie
parametru `-v` .

Warto też nie zapominać o ALSA. Do dyspozycji mamy `alsamixer`, w którym to mogą być wyciszone
poszczególne potencjometry, a to może być przyczyną braku dźwięku. Nawet jeżeli po odpaleniu tego
miksera, do naszej dyspozycji jest tylko coś na wzór tego poniższego obrazka:

![]({{< baseurl >}}/img/2016/01/3.alsamixer-pulseaudio.png)

To możemy wcisnąć klawisz F6 i wybrać z menu odpowiednią kartę. W ten sposób uzyskamy dostęp do jej
przełączników:

![]({{< baseurl >}}/img/2016/01/4.alsamixer-sound-card.png)

Pomocne w rozwiązywaniu problemów może okazać się również usunięcie katalogu `~/.config/pulse/` przy
wyłączonym środowisku graficznym, a następnie zresetowanie maszyny.

Notyfikacje dźwięku

Nie wszyscy z nas lubią te wyskakujące okienka, które zwykły nas powiadamiać o zbędnych z naszego
punktu widzenia zdarzeniach. Inaczej jednak sprawa ma się w przypadku notyfikacji dźwięku, bo
przecie one dostarczają cennych informacji na temat aktualnego poziomu głośności. Pakiety, które
zainstalowaliśmy na początku dają nam możliwość wysyłania takich powiadomień ale potrzebna nam jest
też jakaś aplikacja, która będzie je w stanie generować. W tym przypadku zdecydowaliśmy się na
pakiet `volumeicon-alsa` . Nie jest to jakieś zaawansowane cudo ale grunt, że realizuje ono to czego
od niego oczekujemy. Tak wygląda konfiguracja tego narzędzia:

![]({{< baseurl >}}/img/2016/01/5.volumeicon-alsa-pulseaudio.png)

Jak widzimy, możemy skonfigurować kanał, który będziemy regulować. Mamy także opcje wywołania
miksera PulseAudio. A w dolnej części mamy zaznaczone trzy klawisze, które będą odpowiedzialne za
regulację dźwięku, po których przyciśnięciu system otrzyma stosowne notyfikacje. By te klawisze
działały, potrzebna jest nam [odpowiednio skonfigurowana klawiatura
multimedialna]({{< baseurl >}}/post/klawiatura-multimedialna-i-niedzialajace-klawisze/), bo te
klawisze nie zawsze są rozpoznawane przez Xserver.
