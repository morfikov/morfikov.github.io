---
author: Morfik
categories:
- Linux
date: "2016-01-08T17:53:35Z"
date_gmt: 2016-01-08 16:53:35 +0100
published: true
status: publish
tags:
- xserver
- debian
GHissueID: 530
title: Konfiguracja Xserver'a na debianie (Xorg)
---

Dzięki takiemu wynalazkowi jak Xserver (Xorg) mamy możliwość odpalania aplikacji graficznych. Bez
niego wszystko musielibyśmy robić w czarnej konsoli, przez co funkcjonalność naszej maszyny w dość
znacznym stopniu ucierpiałaby. Oczywiście tryb graficzny ma też swoje wady. Niemniej jednak, Xserver
leży póki co u podstaw każdego środowiska graficznego i jeśli chcemy mieć możliwość odpalania, np.
Firefox'a, czy oglądania filmów w VLC, to nie ma innego wyjścia jak skonfigurować sobie Xserver. W
tym wpisie się zajmiemy tym zagadnieniem, przy czym, chcę uprzedzić, że nie będziemy korzystać z
żadnych automatów, które można znaleźć w tych wszystkich zaawansowanych środowiskach graficznych. A
to z tego względu, że ustawienia tych środowisk zwykle nadpisują ustawienia samego Xserver'a.

<!--more-->
## Instalacja Xserver'a

Xserver jest dość złożoną aplikacją i w jej skład wchodzi szereg pakietów. Generalnie rzecz biorąc,
możemy ograniczyć się jedynie do zainstalowania metapakietu `xorg` , który pociągnie wszystkie
pozostałe paczki w zależnościach. Niemniej jednak, ogromna cześć z tych pakietów jest nam zwyczajnie
nie potrzebna. Mamy tam, np. sterowniki do szeregu kart graficznych. Jeśli mamy jedynie kartę
Intel'a, to logiczne jest, że nie potrzebujemy instalować dodatkowych sterowników do kart Nvidia czy
AMD. Ja zwykle ograniczam się do zainstalowania tych poniższych pakietów (pełna instalacja):

    # aptitude install xorg xserver-xorg xbase-clients xfonts-base xfonts-cyrillic \
    mesa-utils driconf \
    xinput arandr xsel xbacklight scrot numlockx evtest \
    fontconfig ttf-kochi-mincho ttf-mscorefonts-installer fonts-font-awesome

Po zainstalowaniu tych powyższych pakietów, będziemy mieć dostęp do prymitywnego środowiska
graficznego. By je uruchomić, trzeba się zalogować w konsoli TTY i wpisać `startx` .

## Konfiguracja Xserver'a

Xserver można konfigurować przez plik `/etc/X11/xorg.conf` , bądź też przez szereg plików
zlokalizowanych w katalogu `/etc/X11/xorg.conf.d/` . Standardowa instalacja Xorg'a nie dostarcza nam
żadnych z w/w plików i zwykle nie ma potrzeby ich stosowania. Xserver automatycznie jest w stanie
poprawnie rozpoznać i skonfigurować większość sprzętu i zwykle nie będziemy musieli nic zmieniać.
Natomiast w przypadku, gdy monitor, grafika, klawiatura, mysz czy touchpad nie działają tak jak
byśmy tego oczekiwali, to zawsze możemy zmienić ich konfigurację w tych powyższych plikach. Dlatego
też jeśli coś działa prawidłowo, to najlepiej nic nie ruszajmy, bo możemy tylko to popsuć.

### Plik xorg.conf

Jeśli nasz sprzęt wymaga od nas dodatkowych zabiegów konfiguracyjnych, to musimy wiedzieć jak przez
ten proces przebrnąć. Na dobrą sprawę pliki w katalogu `/etc/X11/xorg.conf.d/` są częściami pliku
`/etc/X11/xorg.conf` . Przykładowy plik konfiguracyjny dla Xserver'a można wygenerować zwykle na dwa
sposoby. Pierwszy z nich zakłada posłużenie się narzędziami dostarczanymi przez pakiety Xorg'a.
Drugi zaś jest realizowany przez narzędzia dostarczane do sterowników karty graficznej (np. przez
`nvidia-settings` ). Poniżej przykład generowania pliku `xorg.conf` :

    # Xorg -configure :0

To powyższe polecenie trzeba wydać jako użytkownik root będąc jedynie zalogowanym w trybie
tekstowym. Plik z konfiguracją zostanie utworzony w katalogu `/root/` . Musimy go przenieść do
`/etc/X11/` i zapisać pod nazwą `xorg.conf` .

W przypadku korzystania z zamkniętych sterowników graficznych, trzeba wykomentować wpisy z `dri`
oraz w odpowiednim miejscu zmienić sterownik na właściwy. Poniżej jest przykład dostosowania pliku
`xorg.conf` pod kątem zamkniętych sterowników nvidia:

    Section "Module"
    ...
    #       Load  "dri"
    ...
    #       Load  "dri2"

    Section "Device"
    ...
            Driver      "nvidia"

My jednak nie będziemy się posługiwać plikiem `xorg.conf` i będziemy konfigurować poszczególne
urządzenia w osobnych plikach w katalogu `/etc/X11/xorg.conf.d/` .

### Katalog /etc/X11/xorg.conf.d/

Pliki w `/etc/X11/xorg.conf.d/` trzeba odpowiednio nazywać. Plik konfiguracyjny musi zaczynać się od
dwóch cyfr, po których jest myślnik. Następny ciąg znaków może być dowolny ale nazwa pliku musi
kończyć się na `.conf` . Im mniejszy numer w nazwie pliku, tym konfiguracja w nim zawarta jest
wcześniej aplikowana. Reasumując, nazwa pliku powinna wyglądać następująco: `10-keyboard.conf` .

### Konfiguracja monitora i karty graficznej

Xserver zwykle jest w stanie poprawnie rozpoznać parametry monitora i odpowiednio je ustawić.
Podobnie jest w większości przypadków modułów do kart graficznych. Jeśli jednak mamy problemy z
uruchomieniem sesji graficznej, to być może trzeba [odpowiednio skonfigurować monitor][1].

### Konfiguracja klawiatury

Podobnie spawa ma się w przypadku klawiatury. Z reguły działa ona sprawnie ale też mogą być wymagane
małe poprawki. [Konfiguracja klawiatury][2] została dokładnie opisana w osobnym wpisie.
[Konfiguracja klawiszy multimedialnych][3], które mogą nie być nawet wykrywane przez Xserver,
została opisana także osobno.

### Konfiguracja myszki

Mysz, to jedno z prostszych urządzeń, jakie podłączamy do komputera. Podobnie jak klawiatura,
powinno ono działać bez zarzutu. Możemy jednak [dostosować sobie ręcznie swoją mysz][4] jeśli jej
ustawienia nam nie odpowiadają z jakiegoś powodu.

### Konfiguracja touchpad'a

Jeśli jesteśmy posiadaczami laptopów, to w ich przypadku mamy do dyspozycji touchpad zamiast myszy.
[Konfiguracja touchpad'a][5] została opisana również w osobnym wpisie.

## Logi Xserver'a

W przypadku wystąpienia problemów z załadowaniem się trybu graficznego, pożyteczne informacje
znajdziemy w pliku `/var/log/Xorg.0.log` . Warto przeszukać go pod kątem `WW` albo `EE` . Natomiast
jeśli chodzi o wszelkiego rodzaju błędy generowane przez aplikacje graficzne, to logowane są one do
pliku `~/.xsession-errors` . Warto do niego zajrzeć od czasu do czasu i przejrzeć zalogowane tam
komunikaty. Przez analizę tych logów, można wyeliminować naprawdę sporo błędów.

## Konfiguracja sesji Xserver'a

Konfiguracja sprzętu, na którym ma operować Xserver'a, to jedna sprawa. Osobną kwestią jest
konfiguracja sesji Xserver'a. W debianie mamy kilka poziomów konfiguracyjnych. Pierwszym z nich są
pliki globalne ulokowane w katalogu `/etc/X11/` . Drugą opcją są pliki użytkownika, które nadpisują
konfigurację systemową.

### Konfiguracja globalna Xserver'a

Zajrzymy zatem do katalogu `/etc/X11/` . Mamy tam min. plik `Xsession` , który jest czytany zarówno
przez menadżery logowania jak i w przypadku, gdy podnosimy sesję Xserver'a logując się na TTY i
wydając polecenie `startx` . Ten plik ustawia szereg zmiennych wskazujących na dodatkowe pliki,
rozbudowując tym samym całą konfigurację sesji. Mamy także kilka opcji, które możemy dostosować
sobie w pliku `Xsession.options` .

Analizując dalej plik `Xsession` , mamy tam zmienną `SYSSESSIONDIR` , która wskazuje na katalog
`/etc/X11/Xsession.d/` . W nim są zlokalizowane pliki szeregu usług, np. ssh-agent czy gpg-agent,
tak by te usługi załadowały się na starcie sesji Xserver'a. Zwykle są one wymagane do prawidłowej
pracy systemu. Oczywiście, to jakie pliki mamy w tym katalogu, zależy w dużej mierze od
zainstalowanych pakietów.

W katalogu `/etc/X11/Xresources/` mogą być umieszczone pliki, które odpowiadają za konfigurację
wyglądu poszczególnych aplikacji.

Najważniejszym jednak elementem jest katalog `/etc/X11/xinit/` , który zawiera pliki `xinitrc` oraz
`xserverrc` . W pliku `xserverrc` możemy definiować opcje dla procesu `X` , który jest wywoływany,
za każdym razem, gdy startuje nowa sesja graficzna. Z kolei plik `xinitrc` zajmuje się inicjacją
sesji graficznej i dostosowaniem jej wszystkich elementów, począwszy od ustawiania zmiennych,
skończywszy na uruchamianiu aplikacji. W debianie, domyślny plik `/etc/X11/xinit/xinitrc` wywołuje
plik `/etc/X1/Xsession` .

Lokalna konfiguracja Xserver'a

Szereg z tych powyższych plików ma swoje lokalne odpowiedniki, które zwykle są umieszczane w
katalogu użytkownika. Jeśli one nie istnieją, to czytana jest globalna konfiguracja i to jej
ustawienia są aplikowane. W przypadku, gdy nam ta domyśla sesja Xserver'a nie odpowiada, lub też
chcielibyśmy zmienić opcje procesu `X` , to możemy sobie utworzyć w katalogu domowym pliki
`~/.xinitrc` oraz `~/.xserverrc` i w nich zdefiniować odpowiednie wpisy, posiłkując się oczywiście
globalną konfiguracją.

Wygląd poszczególnych aplikacji dostosowujemy za pomocą plików `~/.Xdefaults` lub `~/.Xresources`.
Ten pierwszy jest już przestarzały i nie powinno się go używać. Więcej informacji na temat tych
plików można znaleźć [pod tym adresem][6].

### Uruchamianie Xserver'a

Nie mając w systemie jeszcze żadnego menadżera logowania, sesję Xserver'a uruchomić możemy za pomocą
polecenia `startx` wpisanego zaraz po zalogowaniu się na konsolę TTY. Nie jest to proces
automatyczny ale możemy to zmienić. W tym celu potrzebna jest nam jedna konsola TTY i odpowiedni
wpis w konfiguracji shell'a. Domyślnym shell'em w debianie jest bash, zatem dodajemy do pliku
`~/.profile` ten poniższy kod:

    if [[ $(tty) = /dev/tty4 ]]; then
          mv ~/.xsession-errors ~/.xsession-errors.old
          exec startx &> ~/.xsession-errors
    fi

W takim przypadku po zalogowaniu się na konsoli nr. 4, automatycznie zostanie zainicjowany
`startx` . W przypadku pozostałych konsol będziemy logowani tak jak przedtem. Domyślnie jednak
system po załadowaniu pozostaje na konsoli nr. 1. Jeśli korzystamy z innej konsoli w celu odpaleniu
sesji graficznej, to musimy poinstruować system, by po zakończeniu startu przeszedł do tej konsoli.
Możemy to zrobić przez plik `/etc/rc.local` dopisując w nim tę poniższą linijkę:

    chvt 4

W taki sposób mamy przygotowaną sesję Xserver'a ale to nie jest koniec. Czeka nas jeszcze
[konfiguracja menadżera logowania LightDM][7], [menadżera okien Openbox][8] i [serwera dźwięku
PulseAudio][9].


[1]: /post/monitor-i-jego-konfiguracja-pod-linuxem/
[2]: /post/klawiatura-i-jej-konfiguracja-pod-debianem/
[3]: /post/klawiatura-multimedialna-i-niedzialajace-klawisze/
[4]: /post/mysz-i-jej-konfiguracja-na-linuxie/
[5]: /post/konfiguracja-touchpada-w-laptopie-pod-linuxem/
[6]: https://wiki.archlinux.org/index.php/X_resources
[7]: /post/menadzer-logowania-lightdm/
[8]: /post/menadzer-okien-openbox/
[9]: /post/konfiguracja-serwera-dzwieku-pulseaudio/
