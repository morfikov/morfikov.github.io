---
author: Morfik
categories:
- Linux
date: "2016-01-25T21:56:20Z"
date_gmt: 2016-01-25 20:56:20 +0100
published: true
status: publish
tags:
- moduły-kernela
- xserver
- dźwięk
title: Jak wyłączyć systemowy "beep"
---

Zgodnie z tym co można wyczytać na [wiki
Archlinux'a](https://wiki.archlinux.org/index.php/Disable_PC_speaker_beep), mamy kilka źródeł
generowania dźwięków, które trafiają do wbudowanego głośnika naszego komputera (case speaker). Te
dźwięki określane mianem "beep" mogą powstać za sprawą BIOS'u płyty głównej, systemu operacyjnego,
środowiska graficznego lub też różnych programów użytkowych. Najbardziej uporczywe są dźwięki
generowane przez BIOS. Na dobrą sprawę, jeśli w BIOS'ie nie ma żadnych opcji dotyczących
konfiguracji tego głośnika, to raczej niewiele jesteśmy w stanie zrobić w tej kwestii. Możemy zawsze
ten głośnik odłączyć fizycznie. Choć nie jest to zalecane, bo na podstawie wydawanych przez niego
dźwięków, jesteśmy w stanie określić czy z naszym komputerem jest wszystko w porządku. Niemniej
jednak, w tych pozostałych trzech w/w punkach mamy większe pole manewru, gdzie możemy dostosować
sobie szereg parametrów i o tym właśnie będzie ten wpis.

<!--more-->
## Jak sprawić, by głośnik zrobił "beep"

Mając do czynienia z linux'ami, często możemy natrafić na sytuacje, w których zarówno sam system
operacyjny jak i poszczególne aplikacje potrafią zainicjować wysoki i do tego bardzo głośny dźwięk.
Przykładem mogą być systemy live z debianem, czy też same manuale konkretnych poleceń. Te dźwięki są
za sprawą [konkretnych
znaków](https://unix.stackexchange.com/questions/1974/how-do-i-make-my-pc-speaker-beep), które
możemy ręcznie wpisać w terminalu. Każdy taki znak powoduje, że głośnik robi "beep". Dla przykładu,
weźmy sobie to poniższe polecenie:

    $ echo -e "\a"

Można także skorzystać [z siódmego znaku ASCII lub
Unicode](https://en.wikipedia.org/wiki/Bell_character) (bell code).

Jeśli nasz głośnik piszczy przy wydawaniu tych poleceń i nie podoba nam się generowany przez niego
dźwięk, to możemy to w bardzo prosty sposób zmienić.

## Moduł pcspkr

W linux'ach za obsługę tego wbudowanego głośnika odpowiada moduł kernela `pcspkr` , który
standardowo jest ładowany przy starcie systemu. Jeśli chcemy całkowicie wyeliminować ten "beep" to
wystarczy zablokować automatyczne ładowanie tego modułu przez system. Możemy to zrobić przez
dopisanie tej poniższej linijki do pliku `/etc/modprobe.d/modules-blacklist.conf` :

    blacklist pcspkr

Oczywiście to rozwiązanie dotyczy wszystkich użytkowników w danym systemie, co nie zawsze może być
pożądane. Być może chcemy wyłączyć ten głośnik tylko dla określonego użytkownika albo nawet nie tyle
wyłączyć co jedynie go przyciszyć nieco, tak by wydobywające się z niego sygnały nie budziły przy
okazji całej okolicy? Zostawmy zatem ten moduł włączony i zobaczmy co jeszcze możemy w tej sprawie
uczynić.

## Jak przyciszyć głośnik

Czasem mamy możliwość sterowania tym głośnikiem za pomocą `alsamixer` , który jest dostępny w
debianie w pakiecie `alsa-utils` . Jeśli korzystamy z PulseAudio, to musimy przejść do suwaków karty
dźwiękowej wciskając klawisz F6 i wybierając odpowiednią pozycję. W przypadku mojej karty, suwaki
wyglądają mniej więcej tak:

![](/img/2016/01/1.glosnik-beep-alsamixer.png#huge)

Suwak na pozycji `Beep` odpowiada za nasz głośnik. Jak widzimy jest ustawiony na full. Jeśli chcemy
nieco przyciszyć dźwięki generowane za jego sprawą, to wystarczy tutaj dostosować sobie poziom
głośności. PulseAudio powinien zapisać ustawienia automatycznie. Jeśli nie korzystamy z niego, to
zawsze możemy zapisać je ręcznie, przez wydanie w terminalu tego poniższego polecenia:

    # alsactl store

## Jak wyłączyć głośnik dla konkretnego użytkownika

Jeśli nie mamy dostępu do konta administratora lub też zwyczajnie nie chcemy zmieniać ustawień
systemowych, to zawsze jesteśmy w stanie napisać lokalną konfigurację, która będzie aplikowana tylko
w stosunku do konkretnych użytkowników.

Głośnik możemy wyłączyć za pomocą narzędzia `xset` . Niemniej jednak, wymagany jest działający
Xserver. Aktualne ustawienia głośnika możemy sprawdzić przez wydanie w terminalu tego poniższego
polecenia:

    $ xset q | grep bell
      bell percent:  50    bell pitch:  400    bell duration:  100

Te trzy wartości możemy sobie zmieniać podając je kolejno w argumencie w `xset` , przykładowo:

    $ xset b 50 750 100

Jeśli jednak chcemy ten "beep" wyłączyć całkowicie, wydajemy to poniższe polecenie:

    $ xset b off

Nie jest to jednak permanentne ustawienie i musimy tę komendę dodać do autostartu środowiska
graficznego, choć raczej GNOME, czy KDE powinny mieć odpowiednie opcje, które pozwalają
skonfigurować ten cały "beep".

## Visual bell

Te piskliwe dźwięki wydobywające się z głośnika mają na celu zaznaczenie jakiegoś zdarzenia. Weźmy
na przykład podręcznik jakiegoś linux'owego polecenia. Spróbujemy w nim przejść do końca pliku. Jak
tylko dotrzemy na koniec pliku, to odezwie się głośnik. Podobnie sprawa ma się z powrotem do
początku pliku. Oba te zdarzenia są wyraźne zaznaczone przez system przy pomocy "beep". Możemy
jednak wyłączyć ten mechanizm i przejść na notyfikację wizualną. W takim przypadku nie usłyszymy już
dźwięku, a zamiast niego na ekranie pojawi się nam tzw. "visual bell" (zwany też "visible bell") i
to on będzie nam sygnalizował różne zdarzenia.

"Visual bell" można włączyć w opcjach używanego terminala. W przypadku terminala urxvt, dodajemy tę
poniższą linijkę do pliku `~/.Xresources` :

    URxvt*visualBell: true

Musimy jeszcze przeładować konfigurację przy pomocy tego poniższego polecenia:

    $ xrdb ~/.Xresources

W ten sposób przy każdy zdarzeniu, w terminalu pojawi się przebłysk o kolorze tekstu trwający ułamek
sekundy.

## Pakiet beep

Jeśli komuś się podoba ten cały ficzer z piszczeniem głośnika, to może go zainteresować pakiet
`beep` , który zawiera narzędzia do generowania dźwięków o różnych częstotliwościach. W taki sposób
można oskrypcić sobie szereg zdarzeń i rozpoznawać je po tonie jaki wydobywa się z głośnika. Można
także skonfigurować sobie ilość sygnałów oraz czas ich trwania. Także całkiem użyteczne narzędzie.
Więcej informacji na temat tego programiku można znaleźć w [man
beep](http://manpages.ubuntu.com/manpages/wily/en/man1/beep.1.html).
