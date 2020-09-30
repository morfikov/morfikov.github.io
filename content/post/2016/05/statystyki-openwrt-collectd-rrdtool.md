---
author: Morfik
categories:
- OpenWRT
date: "2016-05-19T21:38:00Z"
date_gmt: 2016-05-19 19:38:00 +0200
published: true
status: publish
tags:
- chaos-calmer
- statystyki
- router
- rrdtool
title: Statystyki routera w OpenWRT (collectd, rrdtool)
---

Domowe routery WiFi chodzą zwykle 24 godziny na dobę. Ich moc obliczeniowa, choć zwykle niewielka,
czasem się marnuje. Mając router z OpenWRT, możemy przerobić go tak, by zbierał różnego rodzaju dane
dla statystyki. Te dane mogą pochodzić z różnych źródeł i nie koniecznie muszą one dotyczyć samego
routera. Tego typu funkcjonalność mogą zapewnić nam narzędzia `collectd` oraz `rrdtool` . W tym
artykule spróbujemy zaprogramować router, by zbierał pewne dane dotyczące połączenia sieciowego. Na
podstawie tych informacji będą rysowane wykresy, które następnie będą udostępniane przez serwer
`uhttpd` .

<!--more-->
## Instalacja potrzebnego oprogramowania

Jest wiele różnych narzędzi realizujących to przedsięwzięcie, za które się zabieramy. Jak
wspomnieliśmy we wstępie, tutaj skupimy się głównie na `collecd` . To przy jego pomocy będziemy
tworzyć bazy danych zawierające określone wartości. Drugim potrzebnym nam programem będzie
`rrdtool` . Przy jego pomocy zaś będziemy wydobywać informacje z bazy i na ich podstawie generować
zrozumiałe dla człowieka wykresy. Dodatkowo, by mieć łatwy dostęp do tak stworzonych obrazków,
musimy postawić serwer www. Do tego celu znakomicie się nadaje `uhttpd` . Te fotki mogą być do
wglądu nawet i spoza sieci domowej, niemniej jednak, my się ograniczymy tylko do sieci lokalnej.
Instalujemy zatem potrzebne pakiety:

    # opkg update
    # opkg install \
        collectd \
        collectd-mod-rrdtool \
        rrdtool1 \
        uhttpd

Poza powyższymi pakietami, potrzebne nam są również dodatkowe moduły dla `collectd`. Jest ich dość
sporo, a dokładną listę możemy uzyskać wpisując w terminalu polecenie `opkg list | grep
collectd-mod` . Wybieramy te moduły, które według nas będą nam potrzebne i instalujemy. W tym
przypadku są to te poniższe pakiety:

    # opkg install \
        collectd-mod-conntrack \
        collectd-mod-cpu \
        collectd-mod-load \
        collectd-mod-memory \
        collectd-mod-ping

## Konfiguracja uhttpd

Zacznijmy może od konfiguracji serwera www. Plik konfiguracyjny dla `uhttpd` znajduje się w
`/etc/config/uhttpd` . Nie jest on zbytnio złożony. Interesują nas te poniższe pozycje:

    #   list listen_http        0.0.0.0:80
    #   list listen_http        [::]:80
        list listen_http        192.168.1.1:80

    #   list listen_https       0.0.0.0:443
    #   list listen_https       192.168.1.1:443
    #   list listen_https       [::]:443

        option home             /tmp/router/www

Wszystkie widoczne wyżej linijki, które zaczynają się od znaku `#` oznaczają, że wpis został
wykomentowany i nie będzie brany pod uwagę przez `uhttpd` . Chodzi generalnie o to, że nie
potrzebujemy, by demon nasłuchiwał na wszystkich możliwych interfejsach. No i również zbędne nam
jest połączenie SSL/TLS. W opcji `listen_http` ustawiliśmy za to adres interfejsu `br-lan` .
Zmieniliśmy także katalog główny, w którym są przechowywane pliki stron www przy pomocy opcji
`home` . Zapisujemy plik, tworzymy ręcznie w/w katalog i restartujemy demona poniższym poleceniem:

    # mkdir -p /tmp/router/www/
    # /etc/init.d/uhttpd restart

## Konfiguracja collectd

Serwer www powinien nam już działać w tle. Teraz przechodzimy do `collectd` . Plik konfiguracyjny
tego narzędzia znajduje się w `/etc/collectd.conf` . W zależności od zainstalowanych modułów, różne
sekcje się w tym pliku będzie umieszczać. Niemniej jednak, na początku tego pliku definiujemy
zachowanie samego demona `collectd` :

    Hostname        "the-mountain"
    #FQDNLookup     true
    BaseDir         "/var/run/collectd"
    #Include        "/etc/collectd/conf.d"
    PIDFile         "/var/run/collectd.pid"
    PluginDir       "/usr/lib/collectd"
    TypesDB         "/usr/share/collectd/types.db"
    Interval        10
    ReadThreads     2

Chodzi głównie o dostosowanie interwału w parametrze `Interval` . Będzie on nam potrzeby w
późniejszej fazie konfiguracji. W tym przypadku ustawiliśmy go na 10 sekund. Dostosujmy sobie
także parametr `Hostname` , bo to ta nazwa będzie widoczna min. na wykresach.

Kolejne linijki w pliku `/etc/collectd.conf` dotyczą ładowania modułów. Wszystkie z tych modułów,
które zainstalowaliśmy i chcemy aktywować, muszą zostać tutaj określone w opcji `LoadPlugin`
Poniżej został włączony moduł `ping` :

    LoadPlugin ping

Każdy z modułów ma specyficzne dla siebie opcje. Są one ujmowane w sekcje `<Plugin nazwa>` , gdzie
nazwa określa załadowany moduł. Niestety nie da rady opisać ich tutaj wszystkich możliwych modułów.
Niemniej jednak, na potrzeby tego artykułu muszą zostać opisane `conntrack` , `cpu` , `load` ,
`memory` , `ping` oraz `rddtool` .

Poniżej jest konfiguracja modułu `ping` . Ma on za zadanie odpytywać co pewien okres czasu hosty o
adresach określonych w parametrach `Host` . Czas jest zdefiniowany w opcji `Interval` . Musi się on
pokrywać z tym interwałem, który ustawiliśmy wcześniej. Częściej nie ma potrzeby odpytywać serwerów,
bo i tak tylko jeden ping na 10 sekund zostanie zanotowany. Ping'owany host ma też skończony czas na
przesłanie odpowiedzi, którą określa parametr `Timeout` . Jeśli w tym czasie nie zostanie udzielona
odpowiedź, to próba zostanie oznaczona jako nieudana. Ważne jest też, by podać przy pomocy opcji
`Device` interfejs, przez który będzie wychodził `ping` . W tym przypadku jest to interfejs WAN.

    <Plugin ping>
        Host "8.8.8.8"
        Host "208.67.220.220"
        Host "212.77.100.101"
        Interval 10.0
        Timeout 2.0
        TTL 64
        Device "eth0.2"
        MaxMissed -1
    </Plugin>

W celu generowania statystyk z wykorzystaniem `rddtool` , musimy załadować ten modów przy pomocy
`LoadPlugin` i dodać poniższą zwrotkę do pliku `/etc/collectd.conf` :

    LoadPlugin rrdtool

    <Plugin rrdtool>
        DataDir "/tmp/router/stats"
        RRARows 720
        RRASingle false
        RRATimespan 1800
        RRATimespan 3600
        RRATimespan 43200
        RRATimespan 86400
        RRATimespan 172800
        RRATimespan 604800
    </Plugin>

Opcja `DataDir` określa gdzie zapisywać zbierane dane. Kluczowe jest wyliczenie ile PDP (Primary
Data Point) przypada na jeden CDP (Consolidated Data Point) i robi się to ze wzoru: liczba PDP =
timespan/(stepsize * rrarows). W oparciu o powyższą konfigurację otrzymujemy: liczba CDP =
1800/(10*720), czyli 0.25. Podobnie z pozostałymi przedziałami czasu określonymi w opcjach
`RRATimespan` : 0.5 , 6 , 12 oraz 84 . Chodzi o to by wyszły w miarę okrągłe liczby. Co one nam
dają? Tam gdzie CDP jest mniejsze lub równe 1, to nie ma większego problemu z określeniem tego co
zostanie pokazane na wykresie. Natomiast jeśli wartość ta jest większa od 1, np. dla przedziału
czasu 2 dni, CDP przyjmuje wartość 12, wtedy rzecz jasna nie można umieścić wszystkich 12 wartości
na wykresie. Zostanie wybrana tylko jedna z nich i możemy określić ją na podstawie parametrów
`MAX` , `MIN`, `AVERAGE` albo też `LAST` . Jeśli określimy `MAX` , wtedy najwyższa z tych 12
wartości zostanie uwzględniona na grafie. Jeśli `MIN`, to najmniejsza, itd. O tym będzie nieco
później. Jeśli chcemy korzystać z innych wartości niż uśrednione, to przestawiamy `RRASingle` na
`false` .

Pozostałe moduły nie wymagają precyzowania konfiguracji. Wystarczy zatem załadować je w poniższy
sposób:

    LoadPlugin cpu
    LoadPlugin load
    LoadPlugin memory
    LoadPlugin conntrack

Więcej informacji na temat wszystkich modułów oraz możliwych parametrów jakie idzie ustawić w pliku
`/etc/collectd.conf` , można znaleźć [pod tym linkiem][1]. Zapisujemy plik i restartujemy demona
`collectd` :

    # /etc/init.d/collectd restart

## Generowanie statystyk

Potrzebny nam będzie również skrypt, który wygeneruje odpowiednie wykresy. Poniżej nagłówek mojego
skryptu:

    #!/bin/sh

    HOST="the-mountain"
    DATADIR="/tmp/router/stats/"
    IMAGEDIR="/tmp/router/www/stats/$HOST/"
    PERIOD="1hour 12hours 24hours 48hours 1week"
    PLUGINS="cpu-0 load memory ping conntrack"

    for PLUGIN in $PLUGINS
    do
        mkdir -p $IMAGEDIR/$PLUGIN
    done
    ...

Powyżej mamy zdefiniowanych kilka zmiennych. Pliki statystyk ( `DATADIR` ) oraz obrazki z grafiami (
`IMAGEDIR` ) będą przechowywane w pamięci operacyjnej routera. Można oczywiście określić ścieżki do
plików na routerze albo nawet na zewnętrznym nośniku. Z tym, że niesie to za sobą komplikacje. Po
pierwsze, statystyki są zbierane w czasie rzeczywistym, zatem nośnik jest ciągle w fazie zapisu i
przypadkowe odłączenie go zwykle doprowadza do utraty całej bazy danych. Nierzadko też potrafi się
popsuć system plików. Druga kwestia dotyczy wydajności. Statystyki są generowane w pamięci RAM o
wiele szybciej i obciążenie routera (loadavg) jest przy tym 2-3 razy mniejsze niż przy zapisie tych
plików bezpośrednio na dysk. Jeśli dysponujemy routerem o pojemności pamięci RAM większej niż 64
MiB, to bez większego problemu możemy te obrazki zapisywać w pamięci operacyjnej. Trzeba tylko
pilnować się z modułami i nie włączać wszystkich możliwych naraz.

W powyższym wycinku mamy także zmienną `PERIOD` . Określa ona przedziały czasu jakie będą
uwzględniane przy generowaniu obrazków. W tym przypadku jest to 1 godzina, 12 godzin, 24 godziny,
48 godzin i 1 tydzień pracy urządzenia. Pliki ze statystykami nie rozrastają się i zajmują zawsze
tyle samo miejsca. Niemniej jednak, im dłuższy przedział czasu, tym oczywiście baza zajmuje więcej
miejsca. U mnie wszystkie potrzebne statystyki i wykresy zajmują łącznie 5-10 MiB. Także nie jest to
znowu tak mało. Zmienna `PLUGINS` definiuje włączone moduły, na podstawie których są tworzone
odpowiednie katalogi. Jakby nie patrzeć, statystyki będą przechowane w RAM i po zresetowaniu
routera, szlag je trafi. Dlatego też za każdym razem jak router będzie się budził do życia, trzeba
będzie tworzyć te katalogi na nowo.

W zależności od wybranych modułów, musimy umieścić w skrypcie sekcje podobne do tej poniżej. Dla
przykładu określiłem tylko jedną sekcję ale cały skrypt jest na [moim github'ie][2].

    ...
    ##############################
    #        load section        #
    ##############################
    load_load(){
        for TIME in $PERIOD
        do
            rrdtool graph $IMAGEDIR/load/load-$TIME.png --title "$HOST/load/load $TIME" --imgformat PNG --width 600 --height 100 --start now-$TIME --end now --interlaced \
            DEF:longterm_min=$DATADIR/$HOST/load/load.rrd:longterm:MIN \
            DEF:longterm_avg=$DATADIR/$HOST/load/load.rrd:longterm:AVERAGE \
            DEF:longterm_max=$DATADIR/$HOST/load/load.rrd:longterm:MAX \
            DEF:midterm_min=$DATADIR/$HOST/load/load.rrd:midterm:MIN \
            DEF:midterm_avg=$DATADIR/$HOST/load/load.rrd:midterm:AVERAGE \
            DEF:midterm_max=$DATADIR/$HOST/load/load.rrd:midterm:MAX \
            DEF:shortterm_min=$DATADIR/$HOST/load/load.rrd:shortterm:MIN \
            DEF:shortterm_avg=$DATADIR/$HOST/load/load.rrd:shortterm:AVERAGE \
            DEF:shortterm_max=$DATADIR/$HOST/load/load.rrd:shortterm:MAX \
            AREA:longterm_max#ffe8e8 \
            AREA:midterm_max#e8e8ff \
            AREA:shortterm_max#e2ffe2 \
            LINE2:shortterm_max#55ff55:1min \
            GPRINT:shortterm_avg:MIN:" Minimum\:%8.2lf %s" \
            GPRINT:shortterm_avg:AVERAGE:"Average\:%8.2lf %s" \
            GPRINT:shortterm_max:MAX:"Maximum\:%8.2lf %s\n" \
            LINE2:midterm_max#7777ff:5min \
            GPRINT:midterm_avg:MIN:" Minimum\:%8.2lf %s" \
            GPRINT:midterm_avg:AVERAGE:"Average\:%8.2lf %s" \
            GPRINT:midterm_max:MAX:"Maximum\:%8.2lf %s\n" \
            LINE2:longterm_max#ff7777:15min \
            GPRINT:longterm_avg:MIN:"Minimum\:%8.2lf %s" \
            GPRINT:longterm_avg:AVERAGE:"Average\:%8.2lf %s" \
            GPRINT:longterm_max:MAX:"Maximum\:%8.2lf %s\n" \
            >/dev/null 2>&1
        done
    }
    ##############################
    ...

Generalnie rzecz biorąc, to co widzimy powyżej zawiera się w jednej linijce ale dla większej
czytelności skorzystałem ze znaku łamania lini ( `\` ). W ten sposób mogłem przenieść kolejne
parametry `rrdtool` do osobnych linijek. Powyższy blok jest ujęty w funkcję określoną przez
`load_load(){}` . W niej mamy pętle `for` , która to z kolei wygeneruje obrazki w oparciu o
zdefiniowane wcześniej okresy czasu.

Pierwsza linijka (ta z `rrdtool` ) określa gdzie zapisać obrazek ( `graph` ) po wygenerowaniu grafu,
tytuł grafu ( `--title` ), format w jakim obrazek zostanie zapisany ( `--imgformat` ), jego wymiary
( `--width` oraz `--height` ), czas uwzględniony na obrazku ( `--start` i `--end now` ), z tym, że
`now-$TIME` oznacza czas cofnięcia się wstecz. Przykładowo, jeśli określimy `now-1hour` oznaczać to
będzie jedną godzinę wcześniej w stosunku do godziny aktualnej. Opcja `--interlaced` ma zapewnić
szybsze ładowanie się obrazków w przeglądarkach. Możemy także dopisać parametr `--vertical-label=""`
w celu opisania pionowej osi wykresu.

Dalej mamy linijki z `DEF` . Przypisują one zmienne `longterm_avg` , `longterm_max` , etc. do danych
zebranych przez `collecd` w odpowiednich plikach. By sprawdzić jakie dane możemy wyciągnąć z pliku,
trzeba je odczytać przy pomocy poniższego polecenia:

    # rrdtool info /tmp/router/stats/the-mountain/load/load.rrd
    ...
    ds[shortterm]
    ...
    ds[midterm]
    ...
    ds[longterm]
    ...

Dane na wykresach są określane przez CF (Consolidation Function) i ten parametr również możemy
odczytać z powyższego pliku:

    ...
    rra[0].cf = "AVERAGE"
    ...
    rra[1].cf = "MIN"
    ...
    rra[2].cf = "MAX"
    ...

Kolejne linijki w powyższym bloku naszego skryptu to te zaczynające się od `AREA` . Rysują one
słupki i oznaczają je odpowiednimi kolorami, tj. `#ffe8e8` , `#e8e8ff` oraz `#e2ffe2` (RGB, zapis
hexalny). Wartości, w oparciu o które zostaną narysowane te wykresy, są brane ze zdefiniowanych
przez nas zmiennych.

Z kolei wpisy zaczynające się od `LINE`, rysują wykres liniowy i odnoszą się także do legendy pod
wykresem. Cyfra `2` zaraz po `LINE` oznacza grubość kreski, którą rysowany jest wykres. Dalej mamy
określone zmienne, które mają być brane pod uwagę. Mamy także zdefiniowane kolory, odpowiednio
`#55ff55` , `#7777ff` i `#ff7777` . Ostatnia wartość to opis, ten który pojawi się pod grafem.

Linijki zawierające `GPRINT` określają wartość maksymalną ( `MAX` ) , średnią ( `AVERAGE` ) i
minimalną ( `MIN` ) dla danego wykresu. Te informacje również są umieszczane w legendzie.

Każdy moduł ma mniej więcej taki sam zapis. Różni się tylko ilością zdefiniowanych parametrów i po
części też ich nazwami. Po skonfigurowaniu wszystkich modułów, na końcu skryptu wywołujemy
odpowiednie funkcje. W tym przypadku jest póki co tylko jedna:

    ...
    load_load

Tak stworzony skrypt zapisujemy sobie na routerze, np. w katalogu `/etc/skrypty/mkgraph.sh` .

Statystyki może i są zbierane w czasie rzeczywistym i to przez cały czas pracy routera ale obrazki z
wykresami trzeba wygenerować ręcznie. Proces zbierania danych nie jest zbytnio zasobożerny.
Natomiast generowanie wykresów już tak. Naturalnie, nie będziemy ciągle wbijać na router, by
wygenerować sobie kilka grafów. No chyba, że będziemy potrzebować aktualnych danych. Zamiast tego,
wykorzystamy narzędzie `cron` , by cyklicznie te wykresy generował za nas. Dobrze jest ustawić sobie
interwał na 30 minut tak, by nie zarżnąć routera ciągłym generowaniem obrazków. Mając gotowy
powyższy skrypt, musimy dodać wywołanie w tablicy `cron` . Wpisujemy zatem w terminalu `crontab -e`
i dodajemy tam poniższy wpis:

    # min   hour    day     mon     dow     command
    */30    *       *       *       *       /etc/skrypty/mkgraph.sh

## Odczytywanie statystyk

Serwer www mamy już postawiony ale, by obejrzeć statystyki, musimy jeszcze stworzyć prostą stronę
www. Możemy umieszczać wszystkie moduły w jednym pliku albo każdy moduł w osobnym. Możemy też wybrać
odpowiednie obrazki i stworzyć sobie jedną stronę tak, by wszystkie potrzebne nam wykresy były w
jednym miejscu. Szkielet stron zapisujemy w katalogu `/etc/collectd_www/` . Poniżej jest przykładowy
plik `load.html` :

    <html>
        <body>
            <img src="/stats/the-mountain/load/load-1hour.png">
            <img src="/stats/the-mountain/load/load-12hours.png">
            <img src="/stats/the-mountain/load/load-24hours.png">
            <img src="/stats/the-mountain/load/load-48hours.png">
            <img src="/stats/the-mountain/load/load-1week.png">
        </body>
    </html>

By to wszystko teraz puścić w ruch, potrzebny nam jest jeszcze mały skrypt startowy, który
przekopiuje odpowiednie pliki w odpowiednie miejsce podczas fazy boot routera. Tworzymy zatem plik
`/etc/init.d/statistics` o poniższej zawartości:

    #!/bin/sh /etc/rc.common

    START=81

    start() {
        if [ ! -d /tmp/router/www ]
        then
            mkdir -p /tmp/router/www
        fi

        cp -a /etc/collectd_www/* /tmp/router/www/
        /etc/skrypty/mkgraph.sh
    }

Dodajemy go do autostartu i odpalamy skrypt:

    # /etc/init.d/statistics enable
    # /etc/init.d/statistics start

Trzeba jednak pamiętać, że generowanie statystyk zajmuje czas i trochę spowolni start routera.
Dlatego też zawsze możemy wykomentować odpalanie skryptu `mkgraph.sh` . Odpalamy teraz przeglądarkę
internetową i przechodzimy na adres `http://192.168.2.1/load.html` . Jeśli wszystkie kroki
przeprowadziliśmy zgodnie z powyższym opisem, to naszym oczom powinny się ukazać wykresy podobne do
tych poniżej:

Moduł ping:

![](/img/2016/05/1.statystyki-ruch-openwrt-router-collectd-rrdtool.png#huge)

Moduł load:

![](/img/2016/05/2.statystyki-ruch-openwrt-router-collectd-rrdtool.png#huge)


[1]: https://collectd.org/documentation/manpages/collectd.conf.5.shtml
[2]: https://github.com/morfikov/files/blob/master/configs/openwrt/mkgraph.sh
