---
author: Morfik
categories:
- Linux
date: "2016-01-23T19:13:22Z"
date_gmt: 2016-01-23 18:13:22 +0100
published: true
status: publish
tags:
- terminal
- xserver
- openbox
GHissueID: 534
title: Konfiguracja terminala urxvt
---

Na rynku oprogramowania linux'owego mamy całą gamę różnego rodzaju pseudo terminali, które na dobrą
sprawę robią za konsolę w środowiskach graficznych. Jako, że takie środowiska rozrosły się dość
mocno ostatnimi czasy, to instalacja niektórych terminali może pociągać za sobą wiele zależności. To
z kolei przyczynia się do wgrania zbędnego oprogramowania. Inną kwestią są zasoby systemowe, bo
niektóre z terminali potrafią zjeść naprawdę sporo pamięci operacyjnej. Są oczywiście lżejsze
alternatywy i w tym wpisie omówimy sobie konfigurację terminala urxvt.

<!--more-->
## Różnica między rxvt, urxvt, rxvt-unicode i rxvt-unicode-256color

Jeśli chodzi o zagadnienie terminala urxvt, to można się spotkać z szeregiem nazw, które mogą nieco
skomplikować nam życie. Przede wszystkim, wszystko zaczęło się od
[rxvt](https://en.wikipedia.org/wiki/Rxvt). Na jego podstawie powstał w późniejszym czasie fork,
który został nazwany [rxvt-unicode](http://software.schmorp.de/pkg/rxvt-unicode.html), w skrócie
urxvt. Jego główne różnice w stosunku do poprzednika, to obsługa wielu języków dzięki wsparciu dla
kodowania UTF-8 oraz możliwość rozbudowania funkcjonalności terminala przy pomocy skryptów perl'a. Z
kolei `256color` w nazwie oznacza, że ten terminal jest w stanie obsługiwać 256 kolorów. Standardowe
terminale potrafią wyświetlać jedynie 16 kolorów. W debianie mamy dostępnych kilka pakietów, które
mają w nazwie rxvt. Nas interesować będzie `rxvt-unicode-256color` .

Domyślne ustawienia urxvt mogą jednak przerazić każdego. Sam terminal z kolei sprawia wrażenie
niegodnego uwagi i zainteresowania. Pozory jednak mylą, trzeba tylko ten terminal doprowadzić do
użytku.

## Konfiguracja urxvt w ~/.Xresources

Gdy nie korzystamy z pełnych środowisk graficznych i nasz system [opieramy jedynie na menadżerze
okien, np. Openbox](/post/menadzer-okien-openbox/), to jest wielce prawdopodobne,
że spotkaliśmy się już z plikiem `~/.Xresources` . Na dobrą sprawę, to w tym pliku mogą zostać
zdefiniowane ustawienia wyglądu praktycznie każdej aplikacji, którą odpalamy za pośrednictwem
Xserver'a. Zwykle jednak nie korzystamy z tego pliku, bo przecie od zarządzania okienkami mamy
zewnętrzny menadżer okien, a ich wygląd konfigurujemy za pomocą stylów GTK i QT. Niemniej jednak,
konfiguracja zawarta w pliku `~/.Xresources` jest aplikowana w niższej warstwie i jeśli style GTK i
QT nie są w stanie dostosować wizualnie jakiejś aplikacji, to nie ma innego wyjścia jak zrobić to za
pomocą tego pliku.

Na sam początek stwórzmy domyślną konfigurację dla terminala urxvt. Wpisy oczywiście można tworzyć
ręcznie ale też można wygenerować je przy pomocy tego poniższego polecenia:

    $ urxvt --help 2>&1| sed -n '/:  /s/^ */! URxvt*/gp' > ~/.Xresources

Jeśli teraz zajrzymy do pliku `~/.Xresources` , to okaże się, że mamy dość sporo opcji, które możemy
sobie dostosować. Nie są to oczywiście wszystkie możliwe parametry.

### Czcionki w terminalu

By nasz terminal był w miarę czytelny i nie męczył nam zbytnio oczu, musimy skonfigurować mu wygląd
czcionek. Nie chodzi tutaj tylko o krój i wielkość czcionki ale także hinting, antyaliasing i
wygładzanie podpikselowe. W naszym przypadku potrzebujemy tych poniższych linijek::

    Xft.autohint:      false
    Xft.antialias:     true
    Xft.rgba:          none
    Xft.hinting:       true
    Xft.hintstyle:     hintslight
    Xft.lcdfilter:     lcddefault
    Xft.dpi:           96

DPI czcionek, który zostanie tutaj ustawiony będzie miał wpływ na to jak duże będą czcionki.

Krój i rozmiar czcionek definiujemy zaś w poniższy sposób:

    URxvt*font:           xft:monospace:size=9
    URxvt*boldFont:       xft:monospace:bold:size=9
    URxvt*italicFont:     xft:monospace:italic:size=9
    URxvt*boldItalicFont: xft:monospace:bold:italic:size=9

Trzeba jednak pamiętać, że ostateczny wygląd czcionek zależy od [globalnej konfiguracji czcionek w
systemie (fontconfig)](/post/fontconfig-i-konfiguracja-czcionek-w-debianie/). Zatem
jeśli efekt jest inny od oczekiwanego, to musimy zajrzeć do katalogu `/etc/fonts/` , czy też innych
miejsc zdefiniowanych w [specyfikacji
fontconfig'a](https://www.freedesktop.org/software/fontconfig/fontconfig-user.html).

### Ikonka dla terminala

Terminal urxvt domyślnie chyba nie posiada żadnej ikonki i przydałoby mu się jakąś ustawić. Jeśli
nie mamy pomysłu na żaden wzór, to zawsze możemy zakosić ikonkę innemu terminalowi i odpowiednią ją
sobie dostosować, np. zmienić kolorystykę, itp. Mając już ikonkę, dodajemy ten poniższy parametr do
pliku `~/.Xresources` :

    URxvt*iconFile: /sciezka/do/pliku.png

### Pasek przewijania (scrollbar)

Obecnie rzadko się wykorzystuje scrollbar do przewijania treści w terminalu, bo jakby nie patrzeć w
myszach mamy rolki. Nie powiedziałbym, że eliminują one całkowitą potrzebę istnienia pasków
przewijania ale niektóre osoby, w tym też i ja, nie korzystają z nich praktycznie wcale, zwłaszcza
gdy w grę wchodzi okno terminala. Jeśli nie potrzebujemy scrollbar'a, to można go zwyczajnie
wyłączyć poprzez dodanie tej poniższej linijki do pliku `~/.Xresources` :

    URxvt*scrollBar: false

Jeśli natomiast chcielibyśmy aby pasek przewijania był obecny w okienku ale chcielibyśmy dostosować
jego pozycję, to przestawiamy wartość tego powyższego parametru na `true` i dodajemy również tę
poniższą opcję (po jej ustawieniu, scrollbar pojawi się po prawej stronie):

    URxvt*scrollBar_right: true

Możemy także dostosować sobie styl, kolor i grubość samego paska przewijania przy pomocy tych
poniższych parametrów:

    URxvt.scrollstyle: plain
    URxvt*scrollColor: #D75F00
    URxvt*thickness: 5

### Przewijanie treści w okienku terminala

Niektóre z poleceń mogą produkować naprawdę sporo informacji. Przykładem może być aktualizacja
systemu za pomocą `aptitude` czy `apt-get` w debianie, gdzie przez szereg minut na ekranie monitora
jest nam zwracanych całe mnóstwo komunikatów. Ta wyświetlana treść jest przewijania jeśli nie jest w
stanie się zmieścić na ekranie. W ten sposób nie tracimy bezpowrotnie tego co pojawiło się w
terminalu. Niemniej jednak, jeśli przeglądamy historię wiadomości i pojawi się nowy komunikat, to
automatycznie zostaniemy do niego przywołani. Alternatywą jest określenie, by terminal buforował
sobie nowe komunikaty i wyświetlał nam je wtedy, gdy nie przeglądamy aktualnie historii komunikatów.
W tym celu do pliku `~/.Xresources` dodajemy te poniższe parametry:

    URxvt*scrollTtyOutput: false
    URxvt*scrollWithBuffer: true

Możemy także nakazać urxvt, by powrócił do znaku zachęty po rozpoczęciu pisania. Na dobrą sprawę
chodzi jedynie o przyciśnięcie jakiegoś przycisku, a skoro to zwykle oznacza wydawanie polecenia, to
przydaje się, by prompt był widoczny w tym czasie.

    URxvt*scrollTtyKeypress: true

Czasem w terminalu pojawia się bardzo dużo nowych informacji, które mogą powodować problemy z
przewijaniem się treści. Możemy nieco złagodzić ten proces przez niewyświetlanie wszystkich
komunikatów, które system chce nam przekazać. Te dwie poniższe opcje mówią, że w przypadku
pojawiania się dużej ilości linii, okno będzie odświeżane około 60 razy na sekundę lub, gdy zostanie
zapełnione w całości nowymi komunikatami.

    URxvt*jumpScroll: true
    URxvt*skipScroll: true

Jest także opcja, która sprawi, że przy kręceniu rolką myszki, cała strona komunikatów zostanie
przewinięta naraz:

    URxvt*mouseWheelScrollPage: false

### Obramowanie i położenie okna

Możemy także sprecyzować czy terminal ma mieć obramowanie. Jeśli tak, to również jego kolor. No i
oczywiście nic nie stoi na przeszkodzie, by okienko terminala umieścić w określonym miejscu.
Niemniej jednak, te rzeczy mogą również być dostosowane za sprawą, np. menadżera okien Openbox, i
radziłbym byśmy zdecydowali się na jeden z tych dwóch mechanizmów i nie mieszali ich ze sobą.
Niemniej jednak, jeśli chcemy wypozycjonować okno terminala za sprawą pliku `~/.Xresources` , to
musimy dodać te poniższe parametry:

    URxvt*borderLess:false
    URxvt*borderColor: #E77320
    URxvt*geometry: 147x39+100+207

Dwie pierwsze wartości w parametrze `geometry` odpowiadają za liczbę kolumn i wierszy, tj szerokość
i wysokość okienka terminala. Z kolei dwie następne są w pikselach i określają położenie okna na
pulpicie.

### Przezroczystość terminala

Mamy także możliwość ustawienia przezroczystości okna terminala urxvt. Możemy też zdefiniować kolor
tła oraz kolor tekstu. Jeśli chcemy zmienić domyślne wartości, to dopisujemy do pliku
`~/.Xresources` te trzy poniższe parametry:

    URxvt*foreground: #D75F00
    URxvt*depth: 32
    URxvt*background: [50]#000000

Cyfra w kwadratowym nawiasie `[50]` oznacza procent prześwitu koloru czarnego.

W przypadku ustawienia przezroczystości, musimy się liczyć z dość nieprzyjemnym efektem wizualnym.
Chodzi generalnie o to, że im dane okno jest bardziej przezroczyste, tym bardziej będą nam znikać
napisy przy zaznaczaniu tekstu. Dzieje się tak za sprawą odwrócenia kolorów. Jeśli mamy tekst biały
na czarnym tle, to po zaznaczeniu go, te kolory ulegną zamianie. Wobec czego, teraz tekst będzie
czarny, a tło białe. Jeśli jednak w grę wchodzi przezroczystość, to kolor tła zanika, czyniąc
podświetlony tekst praktycznie nieczytelnym. By jakoś sobie z tym problemem radzić, możemy na
sztywno zdefiniować kolory tekstu i tła przy podświetleniu przy pomocy tych poniższych parametrów:

    URxvt*highlightColor: #000000
    URxvt*highlightTextColor: #D75F00

### Kursor i wskaźnik myszy

W pliku `~/.Xresources` możemy także skonfigurować sobie wygląd kursora i wskaźnika myszy. Możemy,
np. określić czy kursor ma przybrać formę kreski czy też prostokącika:

    URxvt*cursorUnderline: true

Jest też opcja włączenia lub wyłączenia migania kursora:

    URxvt*cursorBlink: true

Z kolei jeśli chodzi o wskaźnik myszy, to możemy ustawić czas nieaktywności, po którym on zwyczajnie
zniknie:

    URxvt*pointerBlank: true
    URxvt*pointerBlankDelay: 2

Dodatkowo można pokolorować zarówno kursor jak i sam wskaźnik:

    URxvt*cursorColor: #E77320
    URxvt*cursorColor2: #E77320
    URxvt*pointerColor: #E77320
    URxvt*pointerColor2: #E77320

### Kolory terminala

Każdy terminal ma możliwość zdefiniowania 16 podstawowych kolorów, które jest w stanie wyświetlić.
Kolory możemy określać po nazwach albo po wartościach HEX (są też i inne sposoby). Jeśli nie
zadowalają nas te podstawowe kolory, do dopiszmy sobie do pliku `~/.Xresources` taki blok kodu:

    ! Tango colors
    ! black
    *color0:  #2E3436
    *color8:  #555753
    ! red
    *color1:  #a40000
    *color9:  #EF2929
    ! green
    *color2:  #4E9A06
    *color10: #8AE234
    ! yellow
    *color3:  #C4A000
    *color11: #ffcc00
    ! blue
    *color4:  #3465A4
    *color12: #729FCF
    ! purple
    *color5:  #75507B
    *color13: #AD7FA8
    ! orange (replaces cyan)
    *color6:  #ce5c00
    *color14: #fcaf3e
    ! white
    *color7:  #babdb9
    *color15: #EEEEEC

Są to jedynie podstawowe kolory i nie należy ich mylić z tymi, które są do dyspozycji w terminalach
256-kolorowych. Poniżej przykład:

![](/img/2016/01/1.terminal-urxvt-16-kolorow.png#medium)

Natomiast w przypadku terminali 256-kolorowych będziemy mieli jeszcze do dyspozycji kilka
dodatkowych opcji.

![](/img/2016/01/2.terminal-urxvt-256-kolorow.png#huge)

W obu tych przypadkach te pierwsze 16 kolorów (0-15) będzie takich samych. Poza tym, widzimy na
powyższym obrazku, że kolory rozpoczynają się od numerka 16, a nie od 0.

### Ilość pamiętanych linii

Terminal urxvt jest w stanie zapamiętać skończoną ilość komunikatów, które pojawiają się na ekranie.
Każdy osobny komunikat zajmuje jedną linijkę. Nie musi to być jednak fizyczna linia, bo tę można
dzielić za pomocą znaku `\` . W każdym razie, ilość pamiętanych linijek możemy zmienić poprzez
dopisanie do pliku `~/.Xresources` tego poniższego parametru:

    URxvt*saveLines: 10000

Trzeba jednak mieć na uwadze, że im większą wartość tutaj ustawimy, tym więcej pamięci RAM będzie
konsumował terminal.

### Tryb iso14755

[Tryb iso14755](https://en.wikipedia.org/wiki/ISO_14755) przydaje się w celu wprowadzenia do
terminala znaków, których normalnie nie ma na klawiaturze. Problem w tym, że by mieć dostęp do tych
znaków, trzeba skorzystać z kombinacji klawiszy LCtrl-LShift , przez co traci się możliwość używania
skrótów opartych o te klawisze. Jeśli używamy tego typu kombinacji klawiszy, to ten tryb musimy
wyłączyć przez dodanie do pliku `~/.Xresources` poniższych linijek:

    URxvt*iso14755: false
    URxvt*iso14755_52: false

### Odstępy między wierszami

Jeśli nie pasują nam zbytnio odstępy między kolejnymi wierszami w terminalu, to nic nie stoi na
przeszkodzie, by i ten aspekt urxvt sobie dostosować. Musimy tylko dopisać do pliku `~/.Xresources`
ten poniższy parametr i odpowiednio poeksperymentować z jego wartością:

    URxvt*lineSpace: 0

### Odstępy między literami

Dostosowanie odstępów między literami tekstu wyświetlanymi w terminalu również może mieć miejsce.
Niemniej jednak, zwykle zmiana parametru odpowiedzialnego za te odstępy nie niesie ze sobą nic
dobrego, no chyba, że mamy bardzo nietypową czcionkę i jej elementy zachodzą na siebie. W tym
przypadku czcionka była zwyczajna ale odstępy między literkami były bardzo duże. Być może jest to
jakiś bliżej nieznany błąd. W każdym razie, można go wyeliminować przez dodanie do pliku
`~/.Xresources` tego poniższego parametru i przestawienie jego wartości na `-1` :

    URxvt*letterSpace: -1

## Rozszerzenia dla urxvt

Na początku artykułu wspomniałem, że jesteśmy w stanie rozbudować funkcjonalność terminala urxvt
poprzez skrypty perl'a. Działają one mniej więcej na takiej samej zasadzie co addony w Fifrefox'ie.
By z nich skorzystać, musimy pierw je aktywować. Dodajmy zatem do pliku `~/.Xresources` tę poniższą
linijkę:

    URxvt*perl-ext-common: default,matcher,font-size,clipboard

Powyżej włączyliśmy 3 dodatki, z czego dwa są zewnętrzne. Dodatki dla urxvt są przechowywane w
katalogu `/usr/lib/urxvt/perl/` . Jeśli napisaliśmy własny skrypt perl'a, to wystarczy go umieścić w
tym katalogu. Każdy z tych dodatków ma własne wpisy konfiguracyjne, które należy dodać do pliku
`~/.Xresources` . By włączyć jakiś skrypt, trzeba uwzględnić w tej powyższej linijce nazwę pliku ze
skryptem. Jeśli skrypty mamy zamiar umieszczać w innym katalogu, to możemy go zmienić za pomocą
poniższego parametru:

    URxvt*perl-lib: /usr/lib/urxvt/perl/

### Schowek (clipboard)

Skrypt `clipboard` wprowadza bardziej przyjazne kopiowanie informacji, które opiera się o narzędzie
`xsel` oraz schowek `CLIPBOARD` . By całość działała jak należy, musimy przypisać także odpowiednie
klawisze. Dodajemy zatem te poniższe linijki do pliku `~/.Xresources` :

    URxvt.keysym.C-c:         perl:clipboard:copy
    URxvt.keysym.C-v:         perl:clipboard:paste
    URxvt.keysym.C-M-v:       perl:clipboard:paste_escaped
    URxvt.clipboard.copycmd:  xsel -ib
    URxvt.clipboard.pastecmd: xsel -ob

Kopiowanie wyjścia będzie odbywać się za pomocą klawiszy Ctrl-C . Wklejanie zaś przy pomocy Ctrl-V ,
czyli standardowo. Ostatnie dwie opcje, odpowiadają za polecenia, które zostaną wywołane po
przyciśnięciu tych w/w skrótów. Więcej informacji o tym dodatku można znaleźć [pod tym
linkiem](https://github.com/muennich/urxvt-perls).

Skoro jesteśmy już przy schowku, to warto posuszyć problem, który większość z nas zdążyła raczej
zauważyć. Chodzi o usunięcie zawartości schowka systemowego po zamknięciu, np. edytora tekstu. Jest
to wina Xserver'a, a właściwie sposobu w jaki komunikują się ze sobą okienka (procesy). Generalnie
rzecz biorąc, mamy dostęp do trzech schowków, nazwanych odpowiednio: PRIMARY, SECONDARY oraz
CLIPBOARD.

Na schowku PRIMARY operujemy, gdy wykorzystujemy myszę. By wrzucić do niego jakąś treść, zaznaczamy
tekst kursorem myszy. Z kolei, by tak skopiowany tekst wydobyć z tego schowka, wciskamy środkowy
przycisk myszy. Schowek CLIPBOARD jest używany przy standardowej operacji kopiowania, np. przy
pomocy skrótów Ctrl-C i Ctrl-V . Jest jeszcze schowek SECONDARY i na dobrą sprawę nie mam pojęcia
jak on działa. W każdym razie mamy dwa schowki, które są niezależne od siebie. Nie możemy skopiować
tekstu przy pomocy Ctrl-C i wkleić go za pomocą środkowego przycisku myszy, tzn. możemy, bo jak
zaznaczymy tekst myszą, to ten automatycznie powędruje też i do schowka myszy. Nie działa to jednak
w drugą stronę, np. gdy już zaznaczymy jakąś porcję tekstu i będziemy próbowali wkleić ją przy
pomocy Ctrl-V . W takim przypadku albo nam nic nie wklei, albo zostanie wklejone coś zupełnie
innego.

Operacje na schowku działają w oparciu o właściciela. Przykładowo, jeśli kopiujemy adres URL w
Firefox'ie, to Firefox staje się właścicielem schowka i umieszcza tam skopiowaną zawartość. Załóżmy
teraz, że chcemy wkleić ten adres do innej aplikacji, np. edytora tekstu Geany. Geany poprosi
Firefox'a o tą treść i po jej uzyskaniu, my zobaczymy, że adres został wklejony. Co się jednak
stanie, gdy teraz skopiujemy ten adres w Geany? Geany przejmie kontrolę nad schowkiem, skopiuje do
niego zawartość i od tej chwili do Geany będą wędrować zapytania o wklejanie zawartości schowka.
Takie zachowanie ma jedną wadę. Jeśli zamkniemy aplikację, która rości sobie prawa do schowka
(wrzuciła tam dane), te dane zostaną stracone i żadna aplikacja nie będzie mogła ich uzyskać, tj.
wkleić. Istnieje oczywiście sposób, by temu zapobiec. Można się posłużyć programikiem `xsel` . On ma
tam opcję `--keep` , która pozwala zachować zawartość schowka nawet w przypadku, gdy proces będący
jego właścicielem zostanie zakończony.

### Powiększenie tekstu (font-size)

Niekiedy tekst w terminalu wymaga powiększenia. Rozmiar czcionki w urxvt jest stały choć można nim
operować dowolnie za sprawą określonych poleceń. Niemniej jednak, najprostszym wyjściem jest
skorzystanie ze skryptu `font-size` , który wymaga od nas jedynie określenia skrótu klawiszowego. By
zaimplementować sobie tę funkcjonalność w naszym urxvt, dodajemy do pliku `~/.Xresources` te
poniższe linijki:

    URxvt.keysym.C-S-Up:          perl:font-size:increase
    URxvt.keysym.C-S-Down:        perl:font-size:decrease

Skrót Ctrl-Shift-Up zwiększy rozmiar czcionki, natomiast Ctrl-Shift-Down zmniejszy jej rozmiar. W
ten sposób jesteśmy w stanie dynamicznie zmieniać rozmiar, nawet w przypadku, gdy w terminalu jest
uruchomiona jakaś aplikacja, choć to może powodować pewne problemy natury wizualnej. Więcej
informacji na temat tego dodatku można znaleźć [na stronie
projektu](https://github.com/majutsushi/urxvt-font-size).

### Otwieranie linków w przeglądarce

Terminal urxvt jest w stanie wyróżnić linki, które pojawiają się na jego wyjściu. Zwykle są one
podkreślone ale też możemy oznaczyć je innym kolorem. Jeśli interesuje nas nadanie linkom
niebieskiego koloru, to dodajemy do pliku `~/.Xresources` ten poniższy parametr:

    URxvt*colorUL: #4682B4

Problem w tym, że nie mamy możliwości kliknięcia w taki odnośnik, tak by strona została załadowana w
wybranej przez nas przeglądarce. Niemniej jednak, również i tę niedogodność możemy wyeliminować.
Musimy tylko skorzystać z dodatku `matcher` , który jest wewnętrznym pluginem urxvt. Zatem jeśli
chcemy mieć klikalne odnośniki w terminalu, to dodajemy te dwie poniższe linijki:

    URxvt*matcher.button: 2
    URxvt*url-launcher: /opt/firefox/firefox

Pierwszy z tych parametrów definiuje, który przycisk myszy będzie wykorzystywany do klikania w
linki. W tym przypadku jest to środkowy przycisk. Drugi parametr zaś określa ścieżkę do
przeglądarki, do której ten link ma zostać przesłany.

## Relacje klient-server (urxvtd)

Urxvt ma dwa tryby działania. Pierwszy wykorzystuje standardowe rozwiązanie samego klienta i w tym
trybie działa jak każdy inny terminal. Drugi zaś operuje na zasadzie klient-serwer. Zaletą tego
drugiego rozwiązania jest mniejsze zużycie zasobów. W przypadku kilku klientów, każdy z nich pobiera
taką samą ilość pamięci RAM i jest to około 9 MiB. W przypadku odpalenia 10 instancji terminala
urxvt, będą one zużywać 90 MiB. Jeśli zaś wykorzystujemy serwer, to zjada on, co prawda, 10 MiB
ekstra ale za to każdy kolejny klient podłączony do tego serwera konsumuje około 1 MiB. Także w
przypadku dwóch instancji terminala jesteśmy już do przodu o 7 MiB. Jeśli jednak korzystamy tylko z
jednego terminala, to będziemy 2 MiB w plecy. No i oczywiście w przypadku posiadania serwera, ten
może nawalić. Jeśli taka sytuacja zaistnieje, to tracimy wszystkich klientów i wszystkie
zgromadzone w nich informacje. Jeśli jednak chcemy włączyć sobie tryb klient-serwer, to dodajemy do
autostartu środowiska graficznego to poniższe wywołanie:

    /usr/bin/urxvtd -q -f -o

By teraz podłączyć się do takiego serwera, odpalamy terminal przy pomocy `urxvtc` .

Warto jeszcze wspomnieć, że do części opcji konfiguracyjnych terminala urxvt mamy dostęp przez
wciśnięcie LCtrl i środkowego przycisku myszki.

## Skrót do umieszczenia na panelu

W środowisku graficznym często wykorzystujemy jakiś rodzaj aktywatorów aplikacji GUI. Zwykle są one
umieszczane na panelach czy tego typu podobnych miejscach skąd za pomocą pojedynczego kliku jesteśmy
w stanie wywołać pożądaną aplikację. Poniżej jest przykład umieszczenia takiego skrótu na panelu
tint2.

Wszystko czego nam trzeba, to utworzenie pliku `urxvt.desktop` o poniższej zawartości:

    [Desktop Entry]
    Type=Application
    Version=1.0
    Name=Urxvt
    Comment=Terminal
    Comment[pl]=Terminal
    TryExec=urxvtc -name "standard"
    Exec=urxvtc -name "standard"
    Icon=/home/morfik/.config/launchers/icons/urxvt-standard-32x32.png
    Categories=System;TerminalEmulator;
    Keywords=terminal;

Ten plik wrzucamy następnie do katalogu, który w konfiguracji tint2 jest określony w parametrze
`launcher_apps_dir` . Po zresetowaniu panelu, skrót powinien na nim zagościć.
