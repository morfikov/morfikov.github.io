---
author: Morfik
categories:
- Linux
date: "2015-10-15T20:52:50Z"
date_gmt: 2015-10-15 18:52:50 +0200
published: true
status: publish
tags:
- pulseaudio
- systemd
- dźwięk
- sudo
title: Brak dźwięku przy nieaktywnej sesji logowania
---

Jeśli korzystamy z [serwera dźwięku
PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/) na swoim linux'ie, prawdopodobnie
zdarzyła nam się już sytuacja, w której chcieliśmy zablokować lub wygasić ekran monitora mając
jednocześnie odpalony jakiś odtwarzacz muzyki. Gdy tylko ekran zostanie zablokowany, możemy
odnotować brak dźwięku, bo ten zwyczajnie natychmiast zamiera. Za to po odblokowaniu ekranu, dźwięk
wraca. Podobnie sprawa ma się przy przejściu z trybu graficznego (Xorg) na jedną z konsol TTY.

<!--more-->
## Brak dźwięku za sprawą uprawnień do urządzeń

Jak możemy wyczytać, np.
[tutaj](https://enotty.pipebreaker.pl/2012/05/23/linux-automatic-user-acl-management/) , `logind` ,
który jest częścią projektu systemd, jest w stanie zarządzać uprawnieniami do pewnych urządzeń w
obrębie sesji użytkownika. W wyżej podlinkowanym wpisie, jest link do materiału video, który
idealnie obrazuje cały problem. W skrócie, jeśli odpalimy amaroka i puścimy w nim jakąś piosenkę,
zapewne usłyszymy dźwięk wydobywający się z głośników podpiętych do naszego komputera. Po tym jak
tylko przyciśniemy Ctrl-Alt-F1 i nastąpi przejście do konsoli TTY, gdzie zwykle się logujemy do
systemu w trybie tekstowym, dźwięk zdycha.

Brak dźwięku w tym przypadku jest wynikiem nieaktywnej sesji. Dzieje się tak ponieważ sesja, która
wcześniej była aktywna, po przejściu do konsoli TTY, gdzie nie jesteśmy w żaden sposób zalogowani,
przestaje być aktywna. Podobnie sprawa ma się w kwestii blokowania ekranu, gdzie następuje
wylogowanie użytkownika. Żaden użytkownik, którego sesja nie jest aktywna, nie może mieć dostępu do
pewnych urządzeń i jest to dyktowane polityką bezpieczeństwa. Wyobraźmy sobie co by się stało, gdyby
w tle działał inny użytkownik i nasłuchiwał zdarzeń dźwiękowych, tych które możemy albo usłyszeć na
głośnikach, albo też zarejestrować przez mikrofon. Co w przypadku gdyby to byłą potajemna
konferencja na skrype? Taki nieaktywny użytkownik byłby w stanie zarejestrować całą konwersację. W
przypadku gdy taki użytkownik ma zablokowaną sesję, to nie da rady tego typu podsłuchu
przeprowadzić. Oczywiście, dźwięk to tylko jeden z przykładów. Innym mogą być zdarzenia uzyskane z
klawiatury czy myszy, ewentualnie obraz wyświetlany na monitorze.

Skupmy się jednak na dźwięku i PulseAudio. Nie zawsze potrzebujemy tego typu restrykcji, zwłaszcza
gdy chcemy puścić sobie mp3 i wyłączyć monitor w celu oszczędzania energii. Jak zatem poprawić tego
typu niedogodność? Wyjść są co najmniej trzy.

## Uruchamianie PulseAudio w trybie systemowym

Domyślnie PulseAudio jest uruchamiane wraz ze startem sesji graficznej danego użytkownika, a po jego
wylogowaniu jest ubijane. Jest to `user mode` . Innym trybem jest `system mode` , który polega na
uruchomieniu PulseAudio w taki sposób, by jego [demon był
współdzielony](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/SystemWide/)
przez wszystkich użytkowników w systemie. Niesie to za sobą opisane wyżej problemy z
bezpieczeństwem, bo każdy użytkownik w takim systemie jest w stanie podsłuchiwać wszystkich
pozostałych.

W przypadku jednak gdy mamy jednoosobową stację roboczą i sami korzystamy z komputera, raczej nie
musimy się obawiać tego typu sytuacji. Dlatego też jeśli ktoś chciałby uruchomić PulseAudio w
`system mode` może to zrobić edytując plik `/etc/pulse/daemon.conf` i dopisując tam poniższą
linijkę:

    system-instance = yes

## Dodanie użytkownika do grupy audio

Innym, nieco bezpieczniejszym, sposobem jest dodanie konkretnego użytkownika do grupy `audio` .
Różni się to od tego sposobu opisanego powyżej tym, że tylko wybrani użytkownicy będą mogli
podsłuchiwać, a cała reszta nie będzie miała ku temu okazji. I podobnie jak wyżej, jeśli mamy tylko
jedną osobę, która korzysta z komputera, to możemy dodać takiego użytkownika do grupy `audio` i
problem zostanie rozwiązany. By to zrobić wydajemy poniższe polecenie:

    # adduser morfik audio

Po tej czynności trzeba się ponownie zalogować w systemie.

## Uruchamianie procesu PulseAudio z grupą audio

Innym wyjściem i chyba do tego najbardziej bezpiecznym jest uruchomienie samego procesu `pulseaudio`
[z grupą](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/PerfectSetup/)
`audio` . Problem w tym, że PulseAudio startuje wraz z sesją użytkownika i to nie my go wywołujemy.
Robi to autostart XDG, którego pliki są zlokalizowane w katalogu `/etc/xdg/autostart/` . Oczywiście,
nie będziemy edytować plików systemowych, bo pliki autostartu XDG mogą także być przechowywane w
katalogu domowym konkretnego użytkownika pod `~/.config/autostart/` . Musimy zatem przekopiować plik
`pulseaudio.desktop` i odpowiednio zmienić linijkę, która uruchamia PulseAudio.

Plik autostartu odpowiedzialny za wywoływanie procesu `pulseaudio` wygląda mniej więcej tak:

    [Desktop Entry]
    Version=1.0
    Name=PulseAudio Sound System
    Name[pl]=System dźwięku PulseAudio
    Comment=Start the PulseAudio Sound System
    Comment[pl]=Uruchomienie systemu dźwięku PulseAudio
    Exec=sudo -u morfik -g audio pulseaudio -D
    #Exec=start-pulseaudio-x11
    Terminal=false
    Type=Application
    Categories=
    GenericName=
    X-GNOME-Autostart-Phase=Initialization
    X-KDE-autostart-phase=1

Interesuje nas pole `Exec` , które musimy przerobić, tak jak to widać powyżej. Wykorzystaliśmy tutaj
[sudo](https://pl.wikipedia.org/wiki/Sudo), przy pomocy którego możemy dokonać zmiany grupy nie
będąc jednocześnie jej członkiem i to bez podawania żadnego hasła. Wymagana jest jednak dodatkowa
konfiguracja samego narzędzia sudo. Poniżej znajdują się wpisy, które trzeba umieścić w pliku
`/etc/sudoers` (via `visudo` wpisane z root'a):

    Defaults!/usr/bin/pulseaudio !authenticate, !requiretty, !env_reset
    
    Host_Alias HOSTY = localhost,morfikownia
    
    morfik      HOSTY = (morfik:audio) /usr/bin/pulseaudio

W pierwszej linijce mamy ustawione opcje dla polecenia `/usr/bin/pulseaudio` . Pierwsza z nich, tj.
`!authenticate` sprawi, że nie będziemy pytani o hasło przy wywoływaniu tego polecenia, odpowiednik
`NOPASSWD:` . Dalej mamy `!requiretty` , który umożliwi wykonywanie polecenia poza terminalem, np.
przez autostart środowiska graficznego ( `TTY=unknown` ). Z kolei opcja `!env_reset` ma na celu
zachować zmienne środowiskowe, bo bez tego aplikacje nie będą mogły się połączyć z serwerem dźwięku
PulseAudio. Druga linijka to alias na hosty, na których można wykonywać polecenia. W trzeciej
linijce zezwalamy użytkownikowi `morfik` na wykonanie określonego polecenia, któremu zostanie
zmieniona grupa na `audio` .
