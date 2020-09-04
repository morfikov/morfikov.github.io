---
author: Morfik
categories:
- Linux
date: "2016-01-30T03:30:12Z"
date_gmt: 2016-01-30 02:30:12 +0100
published: true
status: publish
tags:
- terminal
title: Implementacja multipleksera tmux
---

Wszystko zaczęło się od pewnego posta na [forum DUGa](https://forum.dug.net.pl/), w którym to jeden
użytkownik polecał innemu, aby ten zainteresował się programem o nazwie `tmux` . Nie wiem czy tamta
osoba to zrobiła ale ja postanowiłem się przyjrzeć temu wynalazkowi zwanemu [terminal
multiplekser](https://en.wikipedia.org/wiki/Terminal_multiplexer). Po niezbyt wnikliwym przejrzeniu
[strony projektu](https://tmux.github.io/) rzuciło mi się w oczy dzielenie okna jednego terminala na
szereg mniejszych. Ten ficzer znany był mi min. z [terminala
terminator](https://gnometerminator.blogspot.fr/p/introduction.html). Zasadniczą różnicą tych dwóch
aplikacji jest to, że `tmux` może być uruchomiony również pod TTY, efektywnie dzieląc obszar jednej
konsoli. Nie to bym ciągle siedział w trybie tekstowym ale skoro `tmux` potrafi to samo co
`terminator` oraz działa zarówno w trybie graficznym jak i tekstowym przy zaznaczeniu, że zjada
także mniej pamięci RAM, to czemu nie zaimplementować sobie jego obsługi? W trakcie użytkowania
tmux'a okazało, że potrafi on sporo więcej i dlatego właśnie powstał ten wpis.

<!--more-->
## Dobór terminala pod tmux'a

Przede wszystkim, by móc skorzystać z tmux'a, potrzebujemy mieć do dyspozycji jakiś terminal. W
przypadku TTY, to żaden problem. W środowiskach graficznych musimy za to skorzystać z dostarczanych
przez nie pseudo terminali. To z jakiego terminala będziemy korzystać zależy głównie od naszych
upodobań. Jako, że ja staram się zwykle w swojej konfiguracji dbać o jak najmniejsze wykorzystanie
zasobów systemowych przez wykorzystywane aplikacje, to zalecam do tego celu wykorzystanie [terminala
urxvt]({{< baseurl >}}/post/konfiguracja-terminala-urxvt/), którego konfiguracja została opisana w
osobnym wpisie i nie będę jej tutaj ponownie opisywał.

Po zdecydowaniu się na określony terminal, musimy doinstalować w systemie kilka pakietów. Przede
wszystkim, musimy zainstalować sam pakiet `tmux` . Do tego, musimy także zainstalować `xsel` , który
odpowiada za zarządzanie schowkiem systemowym.

## Konfiguracja skrótów klawiszowych tmux'a

Tmux'a uruchamia się przez wpisanie w terminalu polecenia `tmux` . Pierwsze czego człowiek musi się
nauczyć odnośnie w tmux'a, to obsługa skrótów klawiszowych. Nie będziemy jednak korzystać z tych
domyślnych, bo nie są one zbyt wygodne. Choć na dobrą sprawę, to zależy od przyzwyczajenia. Jeśli
jednak chcemy zostać przy standardowej konfiguracji skrótów klawiszowych, to trzeba będzie brać
poprawkę do tego co zostanie opisane poniżej.

Te częściej używane skróty zostały wyszczególnione poniżej. Dzięki nim będziemy w stanie poruszać
się po tmux'ie w miarę swobodnie. Może nie od razu wszystkie z tych skrótów zapamiętamy ale
zapewniam, że po 2-3 dniach korzystania z nich, w końcu wejdą nam do głowy. Poniżej jest także
zawarta konfiguracja, którą należy umieścić w pliku `/etc/tmux.conf` lub też `~/.tmux.conf` w
zależności od tego czy ma to być konfiguracja globalna czy dla konkretnego użytkownika. Aktualną
konfigurację skrótów można zawsze podejrzeć za pomocą tego poniższego polecenia:

    $ tmux list-keys

We wszystkich niżej wymienionych skrótach, będzie ustawiony prefiks `Ctrl-a` . Po wciśnięciu
prefiksu, kombinacja klawiszy nie jest od razu podawana do serwera. Zamiast tego `tmux` czeka na
kolejny znak, który musimy wprowadzić. W momencie, gdy to uczynimy, skrót zostaje przesłany. Dzięki
temu można spokojnie rozluźnić łapki i po prostu wcisnąć Ctrl-a , a następnie pożądany klawisz.
Prefiks naturalnie może być dowolny, a ustawiamy go w poniższy sposób:

    unbind C-b
    set -g prefix C-a

Znak `C` oznacza `Ctrl` . Domyślnym prefiksem jest Ctrl-b i dlatego pierw skorzystaliśmy z
`unbind` , by zwolnić ten skrót. W drugiej linijce przypisaliśmy nowy skrót dla prefiksu.

Mając określony prefiks, możemy przejść do kolejnych skrótów klawiszowych. Poniżej mamy skrót
`Ctrl-a-n` odpowiedzialny za tworzenie nowego okna w tmux:

    bind n new-window

Skoro mowa o oknach, to warto również wiedzieć jak się między nimi przełączać. Za te zdarzenia
odpowiadać będą skróty `Ctrl-a-Space` oraz `Ctrl-a-Backspace` . Możemy także bezpośrednio podać
numer okna za pomocą `Ctrl-a-5` lub też skorzystać z listy okien sesji, która jest dostępna pod
klawiszem `Ctrl-a-'` :

    bind -r Space next-window
    bind -r BSpace previous-window
    bind "'" choose-window

Każde okno tmux'a można zamknąć przy pomocy skrótu `Ctrl-a-Q` . Trzeba jednak pamiętać, że
nieostrożne posługiwanie się tym skrótem może spowodować utratę wszystkich informacji w danym oknie
tmux'a. Dlatego też skorzystamy z opcji `confirm-before` , która wymusi potwierdzenie operacji
zamknięcia okna:

    bind Q confirm-before kill-window

Jako, że `tmux` jest w stanie dzielić okienka, to skróty `Ctrl-a-|` oraz `Ctrl-a-\` zdają się być
odpowiednie dla operacji wertykalnego i horyzontalnego dzielenia przestrzeni okna:

    bind - split-window -v
    bind _ split-window -v
    bind | split-window -h

Mając kilka okienek (pane) w oknie terminala, możemy zmieniać ich rozmiar w poziomie za pomocą
skrótów `Ctrl-a->` oraz `Ctrl-a-<` . Możemy także zmieniać ich rozmiar w pionie przy pomocy
`Ctrl-a-+` `oraz Ctrl-a-=` . Możemy także dostosować ilość wierszy czy kolumn, o
które powiększy/zmniejszy się okienko:

    bind -r < resize-pane -L 3
    bind -r > resize-pane -R 3
    bind -r + resize-pane -U 1
    bind -r = resize-pane -D 1

Warto też wiedzieć jak się przełączać między tymi okienkami. W poziomie robimy to przy pomocy
skrótów `Ctrl-a-Left` oraz `Ctrl-a-Right` , zaś w pionie będą to `Ctrl-a-Up` oraz `Ctrl-a-Down` .
Są to ustawienia domyślne i nic tutaj nie musimy zmieniać.

Każde takie okienko powstałe poprzez podział głównego okna tmux'a można zamknąć. Służy do tego skrót
`Ctrl-a-q` . Jest bardzo podobny do zamknięcia całego okna, dlatego też zalecana jest ostrożność.
Podobnie jak w tamtym przypadku, tutaj też możemy się nieco zabezpieczyć opcją `confirm-before` :

    bind q confirm-before kill-pane

W przypadku, gdy mamy dużo podzielonych okienek, może zajść potrzeba zwiększenia obszaru
obejmowanego przez któreś z nich. Możemy naturalnie utworzyć osobne okno i tam wykonać określone
polecenie albo też skorzystać z opcji zoom, która jest dostępna pod skrótem `Ctrl-a-z` . Ponowne
przyciśnięcie tego skrótu zaowocuje przejściem do trybu okienkowego. W tym przypadku, skrót również
jest domyślny i nie musimy nic dopisywać do pliku konfiguracyjnego tmux'a.

Podzielone okienka, czy też całe okna tmux'a, są zarządzane przez serwer. Ten serwer możemy ubić
przy pomocy skrótu `Ctrl-a-\` . Również tutaj przyda się zastosowanie opcji `confirm-before` :

    bind \ confirm-before kill-server

Listę aktywnych sesji tmux'a możemy uzyskać posługując się skrótem `Ctrl-a-"` . Wygląda to mniej
więcej tak:

![]({{< baseurl >}}/img/2016/01/1.tmux-sesje.png#huge)

Między tymi sesjami możemy się przełączać przy pomocy strzałek i klawisza `Enter` . Zawartość
innych sesji nie jest tracona i zawsze możemy do nich powrócić:

    bind '"' choose-session

Jeśli chcemy odłączyć się od tmux'a i zwolnić tym samym terminal, to możemy skorzystać ze skrótu
`Ctrl-a-d` . Możemy nawet odłączyć dowolnego klienta skrótem `Ctrl-a-D` :

    bind d detach
    bind D choose-client

Zmiany w pliku konfiguracyjnym `/etc/tmux.conf` lub `~/.tmux.conf` nie są uwzględniane do momentu
ponownego uruchomienia serwera. Możemy jednak zainicjować przeładowanie tego pliku za pomocą skrótu
`Ctrl-a-r` :

    bind r source-file /etc/tmux.conf \; display-message "Configuration reloaded"

### Skróty klawiszowe bez prefiksu

Jeśli chodzi o wprowadzanie skrótów klawiszowych, to istnieje jeszcze jeden mechanizm, o którym
warto wiedzieć. Chodzi o to, że niektóre polecenia, takie jak zmiana rozmiaru okienek czy
przełączanie się między nimi, nie byłoby zbytnio użyteczne, gdybyśmy musieli ciągle wciskać
prefiks. Wyżej korzystaliśmy z opcji `-r` , która umożliwia ustawienie timeout'u dla skrótu
klawiszowego, dzięki któremu przez pewien określony czas `tmux` będzie traktował kolejne
przyciśnięcia konkretnych klawiszy jako pełny skrót z prefiksem. Dla przykładu weźmy ten poniższy
wpis:

    bind -r + resize-pane -U 1

Po przyciśnięciu prefiksu `Ctrl-a` , `tmux` będzie oczekiwał na wciśnięcie kolejnego klawisza,
którym w tym wypadku jest `r` . Po jego przyciśnięciu zostanie ustawiony zegar, który będzie
odliczał czas. Jeśli podczas tego okresu ponownie przyciśniemy klawisz r ale już bez prefiksu, to
akcja określona wyżej zostanie ponownie zainicjowana. Czas zegara zostanie zresetowany i ponownie
zacznie się odliczanie. `tmux` przestanie interpretować przyciśnięcie klawisza `r` jako skrót
dopiero  po upłynięciu czasu, który jest liczony od ostatniego wciśnięcia klawisza `r` . Jeśli
chcemy sobie ten czas dostosować, to możemy to zrobić dopisując w pliku konfiguracyjnym tmux'a tę
poniższą linijkę (czas w milisekundach):

    set -g repeat-time 800

## Zarządzanie sesjami

Skróty klawiszowe mamy mniej więcej z głowy. Teraz zajmijmy się sesjami w tmux'ie. Wyżej
wspomniałem, że by zacząć używać tmux'a musimy w oknie terminala wpisać polecenie `tmux` . W
zasadzie jest to prawdą ale to polecenie przyjmuje szereg argumentów, za pomocą których możemy
sterować sesją jak i serwerem. Poniżej znajduje się lista tych częściej używanych poleceń:

  - `tmux attach-session -t sesja` -- podłącza nas do określonej sesji, skrót `attach` .
  - `tmux detach-client` -- odłącza klienta od sesji, skrót `detach` .
  - `tmux start-server` -- uruchamia serwer bez aktywnej sesji.
  - `tmux kill-server` -- zabija serwer.
  - `tmux kill-session -t sesja` -- zamyka określoną sesję.
  - `tmux list-sessions` -- pokazuje aktywne sesje, skrót `ls` .
  - `tmux new-session -t sesja` -- tworzy nową sesję, skrót `new` .

Oczywiście te powyższe polecenia nadają się jedynie do prostego zarządzania sesjami z poziomu
terminala. Co jednak w przypadku, gdy chcemy uruchomić gotową sesję, w skład której wchodzi szereg
okien, z których to cześć jest podzielona na mniejsze i zawiera uruchomione aplikacje? W takiej
sytuacji musimy stworzyć sobie blok sesji w pliku konfiguracyjnym `/etc/tmux.conf` lub
`~/.tmux.conf` , przykładowo:

    new -d -s main -n main "htop"
    splitw -t 1 -v -p 50 "newsbeuter"
    neww -n cmd
    selectw -t main:1
    selectp -t main:2

Składnia takiego bloku jest następująca. Słowo `new` oznacza tworzenie nowej sesji (w tym nowego
okna) i wyżej przyjmuje ono trzy argumenty. Argument `-d` odpowiada za odłączenie sesji od
terminala, `-s` za nazwa sesji, `-n` to nazwa okna. Wszystko to, co zostało ujęte w `" "` , to
polecenie, które ma zostać wywołane zaraz po utworzeniu tego okna. Oczywiście mogą to być dowolne
polecenia, niekoniecznie nazwy aplikacji. Wszystko to, co jesteśmy w stanie wpisać w terminalu,
możemy umieścić tutaj.

W kolejnej linijce mamy słówko `splitw` , które dzieli okno. Mamy tam trzy argumenty. Argument `-t`
określa numerek okna, które należy podzielić. Dalej mamy `-v` dzielący okno wertykalnie (możliwe
również do określenia `-h` ). Ostatni parametr `-p` określa procent podziału okna. W tak powstałym
oknie uruchamiana jest zdefiniowana aplikacja.

Dalej mamy słowo `neww` , które odpowiada za utworzenie nowego okna w sesji. W argumencie `-n`
podajemy jego nazwę i, podobnie jak w przypadku wyżej, uruchamiamy program. Oczywiście możemy nie
nazywać okien ale wtedy automat zrobi to za nas. Podobnie też z poleceniami, których nie musimy
podawać. Jeśli nie podamy, to pojawi się w okienku jedynie prompt.

Pozostałe dwa polecenia, tj. `selectw` oraz `selectp` określają, które okno ma zostać ustawione jako
aktywne. Tutaj mamy tylko jeden argument `-t` , który odwołuje się do konkretnej sesji i jej okna.

## Konfiguracja tmux'a

Wiemy już zatem jak sterować tmux'em i jak konfigurować jego skróty klawiszowe. Poniżej zaś znajdują
się te bardziej użyteczniejsze opcje, które można dodać do pliku konfiguracyjnego `/etc/tmux.conf`
lub też `~/.tmux.conf` . Cały ten multiplekser jest nieco rozbudowany, dlatego też w miarę
zgłębiania jego konfiguracji będę starał się dodawać tutaj nowe sekcje. Kompletny plik
konfiguracyjny [znajduje się na
github'ie](https://github.com/morfikov/files/blob/master/configs/etc/tmux.conf).

### Wsparcie dla myszy w tmux'ie

Przede wszystkim, włączamy support dla myszy. Będziemy mogli za jej pomocą aktywować okienka,
zmieniać ich rozmiar, a nawet przełączać się między nimi klikając w odpowiednie pozycje na ekranie.
Działa również przewijanie rolką. Niestety nie udało mi się zmusić tmux'a do współpracy z `gpm` (to
taki support dla myszy pod TTY), a z informacji, które znalazłem, wychodzi na to, że `tmux` nie wie
jak rozmawiać z `gpm` , w efekcie czego można zapomnieć o bawieniu się myszą pod TTY. Oczywiście
nadal kopiowanie przy pomocy myszy działa ale nie można robić tych wszystkich rzeczy co w przypadku
graficznej sesji Xserver'a. W każdym razie, by włączyć wsparcie dla myszy, dopisujemy do pliku
konfiguracyjnego tę poniższą linijkę:

    set -g mouse on

Jeśli chcemy sobie dostosować poszczególne akcje, które są przypisane do przycisków myszy, to musimy
pierw ustalić nazwy tych przycisków. Poniżej jest tabelka, którą możemy wykorzystać:

    MouseDown1    MouseUp1    MouseDrag1
    MouseDown2    MouseUp2    MouseDrag2
    MouseDown3    MouseUp3    MouseDrag3
    WheelUp       WheelDown

Na końcu tych powyższych nazw musimy także dopisać położenie, które może być jednym z `Pane` ,
`Border` lub `Status` . Przykładowo, jeśli chcielibyśmy aktywować sobie wklejanie tekstu za pomocą
środkowego przycisku myszki, to do konfiguracji dodajemy poniższy wpis:

    bind -n MouseDown2Pane paste-buffer

### Zmiana kolorystyki

W przypadku, gdy nie odpowiadają nam domyślne kolory tmux'a, możemy je sobie dobrać ręcznie.
Praktycznie cała kolorystyka jest zawarta w tych poniższych opcjach:

    set -g display-panes-active-colour colour33
    set -g display-panes-colour colour166
    set -g message-style fg=colour166,bg=colour235,bright
    set -g status-style fg=colour136,bg=colour235
    set -gw clock-mode-colour colour166
    set -gw pane-active-border-style fg=colour166,bg=colour16
    set -gw pane-border-style fg=colour166,bg=colour16
    set -gw window-status-activity-style fg=colour166,bg=colour52,bold
    set -gw window-status-current-style fg=colour166,bg=default,underscore
    set -gw window-status-style fg=colour244,bg=default,dim

Jak widzimy, mamy tutaj szereg ustawień odpowiedzialnych za konfigurację wyglądu paska statusu, okna
głównego czy też podzielonych okienek. Nazwy opcji powinny być zrozumiałe. Jeśli zaś chodzi o
format, który został określony, to każda z opcji przyjmuje trzy wartości, kolor tła, kolor czcionki
oraz listę atrybutów. Pozycja atrybutów może przyjąć wartość `none` lub też jedną z `bright` ,
`dim` , `underscore` , `blink` , `reverse` , `hidden` lub `italics` . Każde z tych powyższych opcji
może zostać poprzedzone za pomocą `no` i w ten sposób wyłączyć którąś opcję. Kolor tła jest
oznaczony przez `bg=` , kolor czcionki zaś za pomocą `fg=` . Poszczególne opcje oddziela się od
siebie za pomocą `,` . Kolory mogą być w formie nazwy (red), postaci HEX ( `#000000` ) lub też
`colour0` - `colour255` o ile dysponujemy terminalem 256-kolorowym.

### Pasek statusu (status bar)

Możemy także dostosować sobie wygląd paska statusu i to nie tylko kolory ale także to, co faktycznie
będzie nam on pokazywał. Poniżej jest potrzebny nam blok kodu, który dodajemy do pliku
konfiguracyjnego:

    set -g status on
    set -g status-interval 1
    set -g status-justify centre
    set -g status-left "#[fg=green,dim,bg=default]Session: #S #[fg=yellow,dim,bg=default]#I #[fg=cyan,dim,bg=default]#P"
    set -g status-left-length 40
    set -g status-position bottom
    set -g status-right "#[fg=red,dim,bg=default] #(cat /proc/loadavg) #[fg=white,dim,bg=default]%H:%M:%S #[fg=blue,dim,bg=default]%Y-%m-%d"
    set -g status-right-length 140
    set -g status-style fg=colour136,bg=colour233

Ze zrozumieniem za co odpowiadają poszczególne opcje nie powinno być większych problemów. Natomiast
cześć z nich (np. `status-right` ) ma dość zawiłą konstrukcję i przydałoby się dodać dwa słowa
komentarza.

Przede wszystkim, poszczególne elementy paska statusu są ujęte w nawiasy: `#[]` oraz `#()` . W
nawiasach kwadratowych określamy informacje dotyczącą formatowania tekstu. W nawiasach okrągłych
wpisujemy polecenia, które zostaną uruchomione w shell'u. Mamy także kilka zmiennych, np. `#S`
odpowiadającą za nazwę sesji. Nie wszystkie zmienne mają takie skróty, a że jest ich cała masa, to
odsyłam do [man tmux, (sekcja FORMATS)](http://man7.org/linux/man-pages/man1/tmux.1.html#FORMATS).
To o czym warto pamiętać w przypadku zmiennych, to fakt, że ich długie nazwy trzeba ując w `#{}` .

### Problemy pod TTY

W przypadku konsoli TTY, domyślne nie działają min. skróty zmiany rozmiaru okienek (pane). Problem
tkwi w interpretacji klawiszy Ctrl , Alt czy Shift przez tę linux'ową konsolę. Dlatego też jeśli
korzystamy ze skrótów, które wykorzystują te powyższe przyciski, to jedyną opcją jest ich
przemapowanie. Oczywiście prefiks, który również wykorzystuje klawisz Ctrl , będzie działał bez
przeszkód. Problemem są jedynie te skróty klawiszowe, które przesyłamy do tmux'a po przyciśnięciu
prefiksu.

### Historia tmux'a

`tmux` posiada swoją własną historię tekstu, która pojawia się na ekranie. Ten mechanizm jest
zaimplementowany praktycznie w każdym pseudo terminalu. Podobnie jak w przypadku tej historii
oferowanych przez terminale, szereg parametrów historii tmux'a również możemy dostosować. Jeśli
zatem chcemy ustawić sobie limit pamiętanych linii, to wystarczy dodać do pliku konfiguracyjnego ten
poniższy wpis:

    set -g history-limit 10000

Jest to limit globalny i każde poszczególne okno będzie w stanie zapisać w historii tyle linii ile
zostało określone powyżej. Warto o tym pamiętać, bo przy sporej liczbie okien, bufor może się dość
znacząco rozrosnąć i konsumować sporo pamięci operacyjnej. Niemniej jednak, historia każdego okna
może zostać wyczyszczona ręcznie za pomocą skrótu `Ctrl-k` :

    bind -n C-k clear-history

Warto zauważyć, że nie stosujemy tutaj prefiksu. To czy prefiks będzie wymagany, zależy od
przełącznika `-n` .

### Numerowanie okienek

Standardowo `tmux` numeruje okienka od `0`. Jeśli jednak nie przepadamy za tym systemem i
preferujemy, by numerki zaczynały się od `1` , czy jakiejkolwiek innej liczby, to możemy dopisać te
dwa poniższe parametry do pliku konfiguracyjnego tmux'a i ustawić im odpowiednie wartości:

    set -g base-index 1
    set -gw pane-base-index 1

### Obsługa 256-kolorowych terminali

Jeśli nasz terminal jest w stanie wyświetlić 256 kolorów, to możemy poinstruować tmux'a, by
odpowiednio ustawił zmienną TERM. Standardowo przyjmuje ona wartość `tmux` lub `screen` . Natomiast,
by włączyć obsługę 256 kolorów, musi ona zawierać frazę `256color` , przykładowo:

    set -s default-terminal screen-256color

### Kopiowanie wyjścia terminala do pliku

Niekiedy istnieje potrzeba, by zalogować praktycznie całe wyjście terminala do pliku. Tak
zgromadzone informacje można wykorzystać w późniejszym czasie. Możemy oczywiście po kawałku kopiować
wyjście i wklejać je ręcznie do pliku ale `tmux` potrafi tę czynność przeprowadzić za nas
automatycznie przy pomocy polecenia `pipe-pane` . Jak nazwa wskazuje, cały tekst, który pojawi się w
danym okienku zostanie skopiowany i przesłany gdzieś dalej. Dobrze byłoby zatem ustawić sobie skrót
`Ctrl-a-Ctrl-p` pod tę akcję:

    bind C-p pipe-pane -o 'ansifilter >> ~/tmux-output.log'

Problem w tym, że zwykle, gdy się koloruje, np. prompt, czy wyjście polecenia `ls`, to w pliku będą
uwzględnione [znaki escape](https://en.wikipedia.org/wiki/Escape_character), czyli coś co się
zaczyna od `^[` . To zwykle czyni wyjście mało czytelnym. Na szczęście ktoś pokusił się o napisanie
filtra, dzięki któremu tekst w pliku będzie przypominał ten, który widzimy w terminalu, tyle, że bez
kolorów. Niestety programu `ansifilter` nie ma w repozytorium debiana i trzeba go ręcznie sobie
zbudować. Kod źródłowy jest dostępny pod [tym
adresem](http://www.andre-simon.de/doku/ansifilter/en/ansifilter.php). Programik `ansifilter` działa
na podobnej zasadzie co narzędzie `script` . Dlatego też możemy się nim posłużyć, jeśli `ansifilter`
jest poza naszym zasięgiem.

### Tryb copy-mode

`tmux` posiada także tryb kopiowania (copy-mode). By wejść w ten tryb, musimy posłużyć się skrótem
`Ctrl-a-[` . W tym trybie jesteśmy w stanie przeszukiwać i kopiować dosłownie każdą treść jaka
widnieje w historii tmux'a. Tak skopiowany tekst można wkleić do terminala przy pomocy `Ctrl-a-]` .
W trybie kopiowania po tekście poruszamy się za sprawą strzałek. By zaznaczyć tekst, wciskamy
`Space` , klawisz `Enter` zaś kopiuje podświetloną zawartość. Skróty klawiszowe odpowiadające za
copy-mode możemy sobie dostosować w poniższy sposób:

    bind Escape copy-mode
    bind p paste-buffer

Możemy także dostosować sobie szereg skrótów, które są dostępne jedynie w copy-mode:

    bind -t vi-copy v begin-selection
    bind -t vi-copy V select-line
    bind -t vi-copy y copy-selection
    bind -t vi-copy r rectangle-toggle
    bind -t vi-copy Escape cancel

Każdy tak skopiowany tekst jest buforowany i przetrzymywany do momentu zresetowania serwera lub też
ręcznego usunięcia bufora z listy. Listę buforów możemy podejrzeć za pomocą skrótu Ctrl-a-b .
Konkretny bufor wybieramy przy pomocy `Ctrl-a-p` . Natomiast jeśli chcemy usunąć ostatni skopiowany
tekst z bufora, to wciskamy `Ctrl-a-x` :

    bind b list-buffers
    bind p choose-buffer
    bind x delete-buffer

Problem z trybem copy-mode jest taki, że bufor, w którym przechowywane są skopiowane wycinki tekstu,
jest tylko do dyspozycji tmux'a. Możemy zatem skopiować pewne informacje, zamknąć terminal,
podłączyć się ponownie przez innego klienta i wkleić w nim to, co skopiowaliśmy wcześniej. To
działa niezależnie od klienta ale nie damy rady z tego bufora przenieść danych do schowka
systemowego, tak by wkleić je, np. do edytora tekstu geany. Możemy jednak przesłać te informacje do
`xsel` , który zarządza schowkiem systemowym:

    bind c run "tmux save-buffer - | xsel --clipboard --input"

Po wejściu w tryb kopiowania, bez znaczenia czy za pomocą myszy, czy przez skrót klawiszy,
zaznaczamy jakąś porcję tekstu i opuszczamy copy-mode. Następnie przyciskamy skrót `Ctrl-a-c` i to
co skopiowaliśmy przed chwilą zostanie przesłane do schowka systemowego skąd będzie mogło
powędrować do innych aplikacji.

### Bufor terminala vs. bufor tmux'a

Jako, że `tmux` posiada własną historię komunikatów, które przewijają się w terminalu, to pojawia
się pewien problem. Chodzi o to, że, jak wspomniałem wyżej, każdy terminal ma swój własny bufor,
zatem mamy teraz dwa. Na dobrą sprawę, nie przeszkadza to w niczym, za wyjątkiem tego, że dwukrotnie
więcej pamięci RAM będzie konsumowane. Jeśli korzystamy z terminala `urxvt` , to możemy pokusić się
o wyłączenie jego buforu. Oczywiście nie musimy robić tego globalnie, a jedynie dla określonej
instancji, w której wywołujemy tmux'a. W ten sposób nie stracimy historii w terminalu bez tmux'a.
Dodajmy zatem do pliku `~/.Xresources` te poniższe wpisy:

    URxvt*saveLines: 10000
    *tmux.saveLines: 0

Nazwa `*tmux.saveLines` zależy od tytułu okna terminala ( `urxvtc -name "tmux"` ). Po dostosowaniu
tego pliku, musimy jeszcze przeładować konfigurację:

    $ xrdb  ~/.Xresources

### Problematyczne sesje tmux'a

W pliku konfiguracyjnym tmux'a można definiować bardzo rozbudowane sesje i nawet może być ich kilka.
Powodować one mogą problemy przy przeładowaniu konfiguracji za pomocą polecenia `source-file` .
Chodzi o to, że po przeładowaniu pliku, w aktywnej sesji będą tworzone dodatkowe okienka. By
wyeliminować tę niedogodność, trzeba skorzystać z warunków, które dostarcza komenda `if-shell` .

Weźmy sobie przykładowe wywołanie sesji, z którym mieliśmy do czynienia wyżej:

    new -d -s main -n main "htop"
    splitw -t 1 -v -p 50 "newsbeuter"
    neww -n cmd
    selectw -t main:1
    selectp -t main:2

By ten powyższy blok sesji był zainicjowany tylko w momencie, gdy ta sesja nie istnieje, musimy
dorobić jej warunki, przykładowo:

    if 'tmux list-sessions| grep "main"'       '' 'new -d -s main -n main "htop"'
    if 'tmux list-windows | grep "newsbeuter"' '' 'neww -n newsbeuter "newsbeuter"'
    if 'tmux list-windows | grep "cmd"'        '' 'neww -n cmd'
    selectw -t main:1
    selectp -t main:2

Polecenie `if-shell` ma swój alias `if` i przyjmuje z grubsza trzy argumenty. Pierwszy z nich to
polecenie, które ma zostać wywołane w shell'u. Następne dwa argumenty to polecenia tmux'a. Jeśli
shell'owa komenda zwróci błąd, to zostanie wykonane drugie z tych tmux'owych poleceń. W powyższym
bloku sprawdzamy czy istnieje sesja i czy istnieją jej okna. Jeśli sesji czy okien nie ma, to
zostaną utworzone. W przypadku, gdy są, np. przy przeładowaniu pliku konfiguracyjnego, to nie
zostanie wykonana żadna akcja, bo pierwsze polecenie tmux'a jest puste.

## Jak uruchamiać tmux'a

Sesje tmux'a mogą nie być przypisane do żadnego terminala i mogą swobodnie działać nawet bez żadnego
klienta. Powyżej mieliśmy przykład sesji, która była tworzona w pliku `/etc/tmux.conf` i na dobrą
sprawę, to są polecenia dla tmux'a. By zatem odpalić serwer i utworzyć sesje, trzeba ten plik
konfiguracyjny załadować. Możemy to zrobić na dwa sposoby. Pierwszy zakłada wywołanie odpowiedniego
polecenia przy starcie sesji graficznej Xserver'a lub też po zalogowaniu się na TTY. Drugi sposób
dotyczy utworzenia pliku `.desktop` , który zainicjuje serwer tmux'a w momencie, gdy będziemy
chcieli skorzystać z terminala.

Jeśli jest to środowisko graficzne, to do jego autostartu dodajemy poniższe polecenie:

    urxvtc -name 'cdesktop' -e bash -c "tmux attach-session -t main" &

Kluczowe jest tutaj wywołanie polecenia `attach-session` oraz, by parametr `-t` wskazywał na jedną z
sesji istniejących w pliku konfiguracyjnym. W tym przypadku, jeśli serwer nie istnieje, to zostanie
podniesiony tworząc wszystkie sesje. Po ich stworzeniu, zostaniemy podłączeni do tmux'a.

W przypadku skrótu, który można umieścić, np. na panelu tint2, tworzymy plik `urxvt-tmux.desktop` o
poniższej treści:

    [Desktop Entry]
    Type=Application
    Version=1.0
    Name=Urxvt
    Comment=Multiple terminals in one window
    Comment[pl]=Wiele terminali w jednym oknie
    TryExec=urxvtc -name "tmux" -e bash -c "tmux attach-session -t main"
    Exec=urxvtc -name "tmux" -e bash -c "tmux attach-session -t main"
    Icon=/home/morfik/.config/launchers/icons/urxvt-tmux-32x32.png
    Categories=System;TerminalEmulator;
    Keywords=terminal;tmux;

Dobrze jest sobie stworzyć drugi taki plik, który wywołuje sam terminal bez tmux'a. To na wypadek,
gdyby z jakiegoś powodu `tmux` nie chciał się nam odpalić. Wrzucamy oba pliki do katalogu, który
został określony w konfiguracji tint2 w parametrze `launcher_apps_dir` i restartujemy panel.

Jeśli chcemy automatycznie być zrzucani do tmux'a po zalogowaniu się, np. na czwartą konsolę TTY, to
do pliku `~/.bashrc` dodajemy tę poniższą zwrotkę:

    if [[ $(tty) = /dev/tty4 ]]; then
        tmux attach-session -t main
    fi

Tmux automatycznie synchronizuje zawartość okien, dlatego nawet jeśli jedno jest większe niż inne,
rozmiar tmux'a będzie dostosowany do tego terminala o najmniejszych wymiarach. W tych większych
okienkach, część przestrzeni zostanie zakropkowana. Jeśli przeszkadza nam to, możemy bez problemu
odłączyć klienta od sesji przy pomocy `tmux detach` albo po prostu zamknąć terminal. Wtedy tmux od
razu zwiększy swój rozmiar i dostosuje go do rozmiarów klienta, który pozostał.
