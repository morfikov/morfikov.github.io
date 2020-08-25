---
author: Morfik
categories:
- Linux
date: "2015-11-09T21:27:51Z"
date_gmt: 2015-11-09 20:27:51 +0100
published: true
status: publish
tags:
- openbox
- terminal
- rsyslog
title: Osadzanie urxvt na pulpicie przy pomocy openbox'a
---

Wszyscy wiemy, że ogromna rzesza ludzi nie patrzy w logi systemowe. Nawet jeśli części z nas zdarza
się to raz na jakiś czas, to zwykle nie wtedy, gdy coś złego się dzieje z naszym systemem. W
przypadku jakichkolwiek problemów, mamy spore prawdopodobieństwo, że szereg zdarzeń może zostać
zalogowanych w dzienniku systemowym. Dlaczego zatem nie osadzić jakiegoś terminala na pulpicie, w
którym będą zbierane logi w czasie rzeczywistym? W takim przypadku co kilka (czy kilkanaście) minut
będziemy w stanie podejrzeć wszystkie komunikaty jakie zostały zalogowane przez system. W tym wpisie
postaramy się osadzić na pulpicie terminal urxvt i posłużymy się w tym celu menadżerem okien openbox
.

<!--more-->
## Devilspie, devilspie2, openbox i terminale

Istnieją dedykowane narzędzia, które mają na celu identyfikować i zmieniać wywoływane w środowisku
graficznym okna. Jednym z nich jest
[devilspie](https://wiki.gnome.org/action/show/Projects/DevilsPie?action=show&redirect=DevilsPie).
Ten projekt nie jest już rozwijany i został zastąpiony przez
[devilspie2](http://www.gusnan.se/devilspie2/). Niemniej jednak, menadżery okien, takie jak
[openbox](http://openbox.org/wiki/Main_Page), jak sama nazwa wskazuje, potrafią zarządzać oknami.
Taki openbox jest w stanie bez większego problemu zrobić to samo, co devilspie czy devilspie2. Nie
ma zatem potrzeby bawić się w instalowanie dodatkowego oprogramowania. No chyba, że nasz menadżer
okien nie potrafi rysować okienek. W takim przypadku najlepiej zrezygnować z niego i dobrać sobie
taki menadżer okien, który to potrafi. Zatem nie będziemy się w tym wpisie zajmować devilspie, a
zamiast niego spróbujemy osadzić konsolę na pulpicie jedynie za pomocą menadżera okien.

Druga kwestia dotyczy terminali. Na rynku jest ich całe mnóstwo i każdy z nich się czymś wyróżnia. W
przypadku osadzania konsoli na pulpicie, najlepiej jest dobrać sobie taki terminal, który posiada
opcję dzielenia okna, oraz ma do dyspozycji karty, takie jak te w przeglądarkach internetowych.
Kiedyś korzystałem z [terminatora](https://gnometerminator.blogspot.fr/p/introduction.html) ale miał
on kilka rzeczy, które mnie strasznie drażniły. W efekcie czego zrezygnowałem z niego na rzecz
multipleksera [tmux](https://tmux.github.io/) , który w połączeniu z prostym terminalem jest w
stanie pobić terminatora pod względem funkcjonalności. Oczywiście w tym drugim przypadku, terminal
nie musi obsługiwać dzielenia okienek, czy zakładek, bo to już realizuje tmux. Wobec czego, możemy
zainstalować sobie terminal [urxvt](https://wiki.archlinux.org/index.php/Rxvt-unicode) (pakiet
`rxvt-unicode-256color` ) i mieć konsolę na pulpicie, która nie zjada za wiele zasobów.

## Identyfikacja okna przy pomocy obxprop

Zacznijmy zatem od zidentyfikowania okna terminala urxvt w gąszczu tych, które są wyświetlane
aktualnie na naszym pulpicie. Do tego celu mogą nam posłużyć dwa narzędzia: `xprop` oraz `obxprop` .
Jako, że wykorzystujemy menadżer okien openbox, to skorzystamy z tego drugiego narzędzia. Odpalamy
zatem terminal i wpisujemy w nim `obxprop` . Następnie wskazujemy myszą to okno , które chcemy
umieścić na pulpicie. Po chwili w terminalu powinien nam się pokazać dość obszerny log z
informacjami opisującymi to okno. Nas interesują głównie te poniższe dane:

    $ obxprop
    ...
    _OB_APP_TYPE(UTF8_STRING) = "normal"
    _OB_APP_TITLE(UTF8_STRING) = "bash"
    _OB_APP_GROUP_CLASS(UTF8_STRING) =
    _OB_APP_GROUP_NAME(UTF8_STRING) =
    _OB_APP_CLASS(UTF8_STRING) = "URxvt"
    _OB_APP_NAME(UTF8_STRING) = "cdesktop"
    _OB_APP_ROLE(UTF8_STRING) =
    ...

To w oparciu o nie, openbox jest w stanie zidentyfikować konkretne okno. Zwykle opcje terminali
zezwalają nam na dostosowanie nazwy, która jest zwracana przez `_OB_APP_NAME` . W przypadku
terminala urxvt, możemy tę nazwę określić precyzując ją w skrócie, który odpala ten terminal,
przykładowo:

    urxvtc -name 'cdesktop' -e bash -c "tmux attach-session -t system-logs"

W powyższej linijce, parametr `-name` ustawia nazwę na `cdesktop` . Mając ustawioną unikalną nazwę,
możemy teraz przejść do napisania regułki, która odpowiednio skonfiguruje wygląd tego okna.

## Konfigurowanie okna

Praktycznie cała konfiguracja okna odbywa się za sprawą jednego z plików konfiguracyjnych openbox'a.
Chodzi oczywiście o plik `~/.config/openbox/rc.xml` . To w nim min. będziemy umieszczać konfigurację
dla okien poszczególnych aplikacji. Każde okno jest ujęte w poniższy blok:

    <application class="URxvt" name="cdesktop" type="normal">
          ...
    </application>

Między znacznikami `application` umieszczamy stosowne opcje. W przypadku terminala na pulpicie,
interesują nas te poniższe:

    <focus>no</focus>
    <decor>no</decor>
    <shade>no</shade>
    <layer>below</layer>
    <skip_pager>yes</skip_pager>
    <skip_taskbar>yes</skip_taskbar>
    <size>
          <width>1000</width>
          <height>300</height>
    </size>
    <position force="yes">
          <x>50</x>
          <y>200</y>
    </position>

Opcji, jakie możemy określić w pliku `rc.xml` , [jest oczywiście
więcej](http://git.icculus.org/?p=mikachu/openbox.git;a=blob_plain;f=data/rc.xml;hb=master). Te
powyższe są nam potrzebne do osadzenia okna pozbawionego obramowania ( `decor` ), które będzie na
samym spodzie pulpitu ( `layer` ), tak by nie wchodziło na okna innych aplikacji. To okno nie
powinno też łapać fokusu, gdy się pojawia ( `focus` ) i nie powinno rzucać żadnych cieni ( `shade` )
. Nie chcemy też by się ono pojawiało na pasku zadań ( `skip_taskbar` ) czy też na pager'ach
(`skip_pager` ). Położenie i rozmiar okna określamy zaś w `position` i `size` .

W przypadku gdyby tak ustawione okno miało obramowanie, trzeba w pliku `rc.xml` odnaleźć pozycję
`keepBorder` i przestawić jej wartość na `No` .

### Przezroczystość okna (transparency)

W taki sposób skonfigurowany terminal powinien pojawić się na pulpicie. Jeśli mamy jakąś wypasioną
tapetę, to możemy ustawić lekką przezroczystość tego okna. Do tego celu potrzebne nam są dwie
rzeczy: menadżer kompozycji i odpowiednie ustawienia terminala. Jeśli chodzi o menadżer kompozycji,
to można skorzystać z [compton'a](https://github.com/chjj/compton) . Nie będę tutaj opisywał jego
konfiguracji, bo po instalacji pakietu `compton` i uruchomieniu tego menadżera, przezroczystość
powinna działać OOTB. Z kolei jeśli chodzi o konfigurację urxvt, to możemy albo dostosować polecenie
wywołujące sam terminal, albo też wpisać całą konfigurację do pliku `~/.Xresources` . Skorzystamy z
tego drugiego rozwiązania. Dodajemy zatem te poniższe linijki do wspomnianego pliku:

    URxvt*scrollBar:   false
    URxvt.depth:                  32
    URxvt*foreground:             #D75F00
    *cdesktop.background:         [00]#000000
    *cdesktop.highlightColor:     #000000
    *cdesktop.highlightTextColor: #D75F00

Parametr `scrollBar` wyłącza pasek przewijania. Naturalnie wyjście jakie pojawia się w terminalu w
dalszym ciągu można przewijać. Dalej mamy `depth` , który włącza prawdziwą przezroczystość okna, do
której obsługi potrzebny jest wyżej wspomniany menadżer kompozycji. Z kolei `foreground` oraz
`background` ustawiają kolor czionki i tła. W przypadku tła mamy `[00]` , co oznacza, że to okno
będzie w 100% przezroczyste. Gdyby tutaj okreslić `[100]` , tło będzie miało taki kolor jaki
zostanie ustawiony. Ostatnie dwa parametry, tj. `highlightColor` i `highlightTextColor` , określają
jaki kolor przyjmie tło i tekst podczas zaznaczania. W przypadku ustawienia przezroczystości na
100%, zaznaczony tekst zwyczajnie zniknie i będzie to wyglądać mniej więcej
tak:

[![1.konsola-na-pulpicie-przezroczystosc]({{< baseurl >}}/img/2015/11/1.konsola-na-pulpicie-przezroczystosc-1024x199.png)]({{< baseurl >}}/img/2015/11/1.konsola-na-pulpicie-przezroczystosc.png)

Z lewej strony jest terminal, który ma czarne tło. Ten z prawej zaś jest przezroczysty. Dlatego
właśnie nam są potrzebne te dwa parametry. Mając już osadzony i skonfigurowany terminal na
pulpicie, przyszła pora na wyświetlenie w nim logów systemowych.

## Logi systemowe

Logi w debianie są trzymane w katalogu `/var/log/` . Od czasu, gdy domyślnym init'em w debianie jest
systemd, logi są zbierane przez journal. Jak wiadomo, format plików journala ma strukturę binarną i
w prosty sposób nie idzie wydobyć z tego pliku informacji. Dodatkowo, wszystkie aplikacje, które
mają własne pliki logów, np. apache2, nie logują tych informacji do journala. Mamy zatem dwa
problemy. Pierwszym z nich jest wyciągnięcie informacji z journala i umieszczenie ich w terminalu na
pulpicie. Drugim zaś jest zebranie wszystkich pozostałych komunikatów rozrzuconych w plikach w
katalogu `/var/log/` i wyświetlenie ich również w tym terminalu.

### Pliki logów w katalogu /var/log/

Przede wszystkim musimy utworzyć urządzenie, podobne do tych w katalogu `/dev/` . Można je
oczywiście utworzyć w dowolnym miejscu ale skoro tam są urządzenia, to lepiej wszystkie trzymać w
jednej lokalizacji. By stworzyć takie urządzenie logujemy się na konto root i w terminalu wydajemy
poniższe dwa polecenia:

    # mkfifo -m 0740 /dev/logi
    # chown root:adm /dev/logi

Stworzone w ten sposób urządzenie nie będą jednak trwałe i po resecie systemu trzeba będzie je
tworzyć na nowo. Dlatego lepszym rozwiązaniem jest zdefiniowanie tych plików urządzeń w
`/etc/tmpfiles.d/logi.conf` , przykładowo:

    # Type | Path | Mode | UID | GID | Age | Argument
    p /dev/logi 0640 root adm - -

Teraz edytujemy plik `/etc/rsyslog.conf ` i dodajemy w nim tę poniższą linijkę:

    *.*;auth,daemon,kern,user      -/dev/logi
    #& stop

Za jej sprawą zostaną skopiowane pewne wpisy, które następnie zostaną wysłane do naszego urządzenia
`/dev/logi` . To jakie opcje można dodać są zawarte pod [tym
linkiem](https://wiki.archlinux.org/index.php/rsyslog#Facility_Levels). Jeśli usuniemy znak
komentarza stojący przed `& stop` , log nie będzie przetwarzany przez kolejne reguły. Ważne jest, by
wszystkie reguły kopiujące informacji do urządzeń w katalogu `/dev/` pojawiły się jak najwyżej w
sekcji `RULES` w pliku konfiguracyjnym rsyslog'a.

Samo urządzenie posiada grupę `adm` oraz prawa tylko do odczytu. Nikt poza użytkownikami w tej
grupie nie będzie w stanie podejrzeć logów. Dlatego też trzeba dodać określonych użytkowników do tej
grupy:

    # adduser morfik adm

Można także stworzyć kilka urządzeń i rozdzielić odpowiednie obiekty jeśli komuś przeszkadza, że
wszystko wędruje do jednego pliku.

### Journal i jego logi

Problem z journalem można rozwiązać w bardzo prosty sposób. Mianowicie, wszystkie, pliki jakie
tworzy journal, mają grupę `systemd-journal` . By mieć dostęp do tych plików i móc je podejrzeć,
potrzebne nam są prawa odczytu, które są nadawane użytkownikom będącym we wspomnianej powyżej
grupie. Musimy zatem się dodać do tej grupy:

    # adduser morfik systemd-journal

### Wyświetlenie logów w terminalu na pulpicie

Część z terminali ma możliwość uruchamiania poleceń ilekroć są wywoływane. Tak przynajmniej było w
przypadku terminatora. Tmux również posiada taką właściwość. Jako, że ja korzystam jedynie z tmux'a,
to niżej przedstawię rozwiązanie, które będzie się aplikować jedynie do tego narzędzia. Myślę, że da
radę bez problemu przepisać je na inne terminale. Edytujemy zatem plik `/etc/tmux` i dodajemy w nim
poniższą
    zwrotkę:

    new -d -s system-logs -n system "journalctl -b --no-pager --since -10m | ccze -m ansi && systemctl --failed --no-pager | ccze -m ansi && journalctl -n 0 -f | ccze -m ansi"
    neww -n logi "cat /dev/logi | ccze -m ansi -p syslog -C"
    selectw -t system-logs:1
    selectp -t system-logs:1

Mamy tutaj z grubsza po jednym przykładzie dla zwykłych logów, jak i dla tych pochodzących z
journala. Pierwsza linijka zawiera kilka poleceń, które są wywoływane po sobie. Ich zadaniem jest
wyciągnięcie z journala logów, które do niego trafiły w ostatnich 10 minutach. Chodzi o to by logi
ze startu były wyświetlane, bo są one przecie dość istotne. Niemniej jednak, nie chcemy by podczas
restartu środowiska graficznego, czy tego terminala na pulpicie, te wszystkie logi były znów
wczytywane. Następnie mamy wywołanie `systemctl` z opcją `--failed` zwracającą usługi, których start
zakończył się błędem z jakiegoś powodu. Na samym końcu mamy ponownie wywołany `journalctl` , który
tym razem będzie zwracał komunikaty, które trafiają do journala w trybie rzeczywistym. Druga linijka
zaś, cat'uje urządzenie `/dev/logi` wyświetlając tym samym jego zawartość na konsoli. Pozostałe dwie
linijki są specyficzne dla tmux'a.

### Kolorowanie logów

Wszystkie te powyższe polecenia przepuszczają swoje wyjście przez narzędzie `ccze` , które jest
dostępne w debianie w pakiecie pod tą samą nazwą. Jego zadaniem jest kolorowanie poszczególnych
fraz, co czyni log o wiele bardziej czytelnym. W ten sposób, np. słowa `error` czy `warn` wyróżniają
się i nie trzeba ich zbytnio szukać w logu.

## Gotowy terminal na pulpicie

Polecenie, które wywołuje terminal z odpowiednimi opcjami, trzeba oczywiście dodać jeszcze do
autostartu openbox'a. Odbywa się to przez plik `~/.config/openbox/autostart` . Sam kod zaś wygląda
mniej więcej tak:

    ### urxvt daemon
    if [ -z "$(pidof urxvtd)" ] ; then
          /usr/bin/urxvtd -q -f -o
    fi

    ### urxvt client
    (sleep 2 && urxvtc -name 'cdesktop' -e bash -c "tmux attach-session -t system-logs") &

Od tego momentu, po zalogowaniu się w sesji graficznej, terminal powinien się automatycznie
uruchomić, a openbox powinien go umieścić w odpowiednim miejscu na pulpicie. U mnie prezentuje się
tak:

[![2.konsola-na-pulpicie-efekt-koncowy]({{< baseurl >}}/img/2015/11/2.konsola-na-pulpicie-efekt-koncowy-1024x267.png)]({{< baseurl >}}/img/2015/11/2.konsola-na-pulpicie-efekt-koncowy.png)

Jako, że to okienko nie posiada żadnego obramowania, to nie mamy zbytnio możliwości jego
przesuwania. Jeśli chcielibyśmy jednak zmienić jego pozycję lub rozmiar, to musimy wcisnąć klawisz
Alt i za pomocą lewego lub prawego przycisku myszy odpowiednio dostosować sobie wygląd tego okna.
