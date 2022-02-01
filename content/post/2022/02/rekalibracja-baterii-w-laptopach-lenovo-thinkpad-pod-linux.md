---
author: Morfik
categories:
- Linux
date:    2022-02-01 18:09:00 +0100
lastmod: 2022-02-01 18:09:00 +0100
published: true
status: publish
tags:
- debian
- bateria
- akumulator
- laptop
- thinkpad
- lenovo
- t430
GHissueID: 587
title: Rekalibracja baterii w laptopach Lenovo ThinkPad pod linux
---

Jakiś już czas temu opisywałem temat [konfiguracji progów ładowania baterii w laptopie Lenovo
ThinkPad T430][1], tak by przedłużyć parokrotnie jej żywotność. Niemniej jednak, ustawienie na
dłuższy czas progów 30-40% nie pozostaje bez wpływu na pracę maszyny. Może i same baterie
litowo-jonowe nie posiadają efektu pamięci ale elektronika, która tymi bateriami zarządza, już tę
pamięć może posiadać. Chodzi generalnie o to, że brak pełnych cyklów baterii (rozładowanie do 0% i
ładowanie do 100%) może powodować różne błędy w ocenie ile faktycznie tego ładunku w takim
akumulatorze pozostało. Efektem tych błędów są wskazania sugerujące, że nasz laptop może jeszcze
popracować, np. 20-30 minut dłużej, gdy stan faktyczny na to nie pozwala, co skończyć może się
utratą niezapisanych danych w skutek nagłego odcięcia zasilania. Dlatego też jeśli wykorzystujemy
jedynie pewien zakres pojemności baterii w laptopie, to co pewien czas (co 30-50 takich niepełnych
cykli albo raz na 2 miesiące) przydałoby się przeprowadzić w laptopie proces rekalibracji baterii
(a raczej rekalibrację jej kontrolera/elektroniki), tak by ewentualne straty pojemności zostały
uwzględnione w szacunkach systemu. Zabieg rekalibracji baterii można bez większego problemu
przeprowadzić z poziomu dowolnej dystrybucji linux'a, np. Debian/Ubuntu, i tym zagadnieniem się
zajmiemy w niniejszym artykule.

<!--more-->
## Prymitywne podejście do rekalibracji baterii

Jako, że proces rekalibracji baterii nie wymaga zbytniego zaangażowania z naszej strony, to nie
potrzeba do jego przeprowadzenia żadnych zaawansowanych narzędzi. Wszystko co musimy zrobić, to
zmienić progi ładowania (jeśli je wcześniej oczywiście ustawiliśmy), tak by wartości wskazywały na
0-100%, przykładowo:

    # echo 0 > /sys/class/power_supply/BAT0/charge_start_threshold
    # echo 100 > /sys/class/power_supply/BAT0/charge_stop_threshold

Po zmianie wartości progów ładowania, trzeba doładować baterię do 100%. Jak tylko akumulator
zostanie podładowany do 100%, odłączamy zewnętrzne zasilanie i czekamy aż poziom baterii spadnie do
0%, po czym ponownie ładujemy baterię do 100%.

Te opisane wyżej kroki są niezbędne, by odczyt poziomu naładowania (State of Charge, SoC) był
akuratny. Chodzi o to, że w kontrolerze baterii zostaną sprecyzowane nowe wartości dla stanu
pełnego naładowania i pełnego rozładowania akumulatora.

### Problematyczne osiągnięcie 0% na akumulatorze

Przeprowadzenie procesu rekalibracji baterii w opisany powyżej sposób ma jedną zasadniczą wadę.
Jeśli poziom naładowania baterii spadnie nam do 0%, to laptop nam się wyłączy, co może nieść ze
sobą poważne skutki, np. rozsypanie się systemu plików wskutek nagłej utraty zasilania. Oczywiście
można powyłączać zbędne aplikacje i przemontować systemy plików wszystkich partycji w tryb tylko do
odczytu. Niemniej jednak, wciąż pozostają aktywne pewne mechanizmy mające na celu monitorowanie
stanu akumulatora i podejmowanie stosownych akcji, gdy ten stan jest zbyt niski. Tutaj też można te
usługi powyłączać ale właśnie w tym tkwi cały problem, tj. by przeprowadzić proces rekalibracji
baterii w laptopie, użytkownik musi się zatroszczyć o całą masę rzeczy, czego zwykle mu się robić
nie chce.

## Rekalibracja baterii za sprawą TLP

W Debianie od dłuższego czasu dostępne jest [narzędzie TLP][2], które ma na celu zatroszczyć się o
żywotność baterii w laptopie. W zasadzie nie korzystałem z tego oprogramowania, bo u siebie
potrzebowałem skonfigurować jedynie progi ładowania, które to można ustawić z powodzeniem także i
ręcznie. Niemniej jednak, TLP w wersji 1.5 zyskał bardzo ciekawą cechę, a jest nią rekalibracja
baterii w laptopach Lenovo ThinkPad, które korzystają ze sterownika `thinkpad_acpi` (standardowo
obecny w jądrze linux'a). Wcześniejsze wersje TLP również oferowały rekalibrację baterii ale trzeba
było własnoręcznie kompilować [zewnętrzny moduł kernela acpi_call][3], co było trochę upierdliwe,
nie wspominając już o tym, że ten moduł od prawie dekady nie miał żadnej aktualizacji.

### TLP v1.5 i patch na kernel

Póki co, w Debianie pakiet `tlp` dostępny jest jedynie w starszej wersji 1.4 i by móc przeprowadzić
proces rekalibracji baterii bez potrzeby instalowania zewnętrznych modułów kernela, trzeba będzie
albo poczekać, aż ta nowsza wersja trafi do repozytorium Debiana, albo możemy sobie [paczkę deb
zbudować ręcznie][5].

Samo posiadanie TLP w wersji 1.5 to nie wszystko. Póki co, w najnowszym stabilnym kernelu, tj.
5.16.4, nie ma jeszcze stosownego kodu, który by zapewniał możliwość przeprowadzenia procesu
rekalibracji akumulatora przy wykorzystaniu modułu `thinkpad_acpi` (prawdopodobnie będzie w 5.17).
Potrzebny zatem będzie nam [stosowny patch][4], przez co bez [rekompilacji kernela][6] się nie
obejdzie.

### TLP w akcji

Zakładając, że udało nam się przezwyciężyć przeciwności i mamy TLP w wersji 1.5, a w kernelu linux'a
znalazł się stosowny kod, możemy przejść do przeprowadzenia procesu rekalibracji baterii w naszym
laptopie.

Przede wszystkim zweryfikujmy, czy wszystko mamy na swoim miejscu. Odpalamy zatem `tlp-stat` i
patrzymy na zwrotkę z `Battery Care` :

    # tlp-stat -b
    --- TLP 1.5.0 --------------------------------------------

    +++ Battery Care
    Plugin: thinkpad
    Supported features: charge thresholds, recalibration
    Driver usage:
    * natacpi (thinkpad_acpi) = active (charge thresholds, recalibration)
    Parameter value ranges:
    * START_CHARGE_THRESH_BAT0/1:  0(off)..96(default)..99
    * STOP_CHARGE_THRESH_BAT0/1:   1..100(default)
    ...

Jak widzimy wyżej, w przypadku mojego laptopa Lenovo ThinkPad T430 użyty został plugin `thinkpad` .
Widzimy też, że mamy wsparcie dla progów ładowania ( `charge thresholds` ) oraz również dla
rekalibracji baterii ( `recalibration` ).

Ładujemy zatem baterię laptopa do 100%. Po naładowaniu baterii do pełna nie odłączamy zasilania
sieciowego. Następnie w terminalu jako root wpisujemy polecenie `tlp recalibrate` podając w jego
argumencie tę baterię, którą zamierzamy skalibrować. Jeśli nie wiemy jaki numerek ma bateria, którą
chcemy poddać rekalibracji, to zawsze możemy ten fakt ustalić przy pomocy `upower` :

	$ upower -e | grep BAT
	/org/freedesktop/UPower/devices/battery_BAT0

    # tlp recalibrate BAT0
    Setting temporary charge thresholds for BAT0:
      stop  = 100
      start =  96
    Initiating discharge of battery BAT0

    Currently discharging battery BAT0:
    voltage            =  10840 [mV]
    remaining capacity =  19870 [mWh]
    remaining percent  =    100 [%]
    remaining time     =     89 [min]
    power              =  13315 [mW]
    state              = Discharging
    force-discharge    = 1
    Press Ctrl+C to cancel.
    ...

Wartym odnotowania tutaj jest fakt, że bateria ulega rozładowaniu ale przewód zasilający laptopa
jest cały czas podłączony. By uzyskać jak najlepsze wyniki, nie powinniśmy zbytnio obciążać maszyny
 podczas procesu rekalibracji, tj. przydałoby się ograniczyć zużycie przez nią energii do minimum,
 np. wyłączając monitor, czy przełączając kill-switch od WiFi/BT.

### Co się stanie gdy bateria osiągnie 0%

Przy prymitywnym procesie rekalibracji, gdy bateria osiągnęła stan naładowania 0%, to laptop nam
się wyłączał. W przypadku TLP, po osiągnięciu przez baterię stanu pełnego rozładowania, komputer
nam się nie wyłączy.

Mierząc pobór prądu podczas procesu rekalibracji z poziomu gniazdka sieciowego, do którego wpięty
był laptop, można było zauważyć, że laptop pobierał około 17W. Po wpisaniu tego powyższego
polecenia, zużycie prądu nie spadło do 0W, a do 5W (sam zasilacz w zasadzie nie pobiera prądu, od
czasu do czasu wskazania na mierniku pokazywały od 0,2W do 0,4W). Zatem komputer nie zaprzestał
pobierania energii z sieci elektrycznej, a jedynie ten pobór dość mocno ograniczył. Gdy poziom
baterii był już w zasadzie na 0%, system przełączył się na zasilanie z sieci i pobór prądu wzrósł
natychmiast do 55W.

### Nieoczekiwany spadek poziomu naładowania o x%

Podczas procesu rozładowywania baterii nie było większych niespodzianek. Bateria rozładowywała się
przez około 69 minut. Pod koniec zaś, gdy stan naładowania akumulatora wskazywał około 16%,
nastąpiło obniżenie się tego stanu z 16% do 5%, co zostało też odnotowane w logu:

    ...
    Currently discharging battery BAT0:
    voltage            =  10142 [mV]
    remaining capacity =   3270 [mWh]
    remaining percent  =     16 [%]
    remaining time     =     15 [min]
    power              =  12778 [mW]
    state              = Discharging
    force-discharge    = 1
    Press Ctrl+C to cancel.
    Currently discharging battery BAT0:
    voltage            =  10167 [mV]
    remaining capacity =   3230 [mWh]
    remaining percent  =     16 [%]
    remaining time     =     15 [min]
    power              =  12810 [mW]
    state              = Discharging
    force-discharge    = 1
    --------------------------------------
    Press Ctrl+C to cancel.
    Currently discharging battery BAT0:
    voltage            =  10081 [mV]
    remaining capacity =   1170 [mWh]
    remaining percent  =      5 [%]
    remaining time     =      5 [min]
    power              =  12722 [mW]
    state              = Discharging
    force-discharge    = 1
    Press Ctrl+C to cancel.
    Currently discharging battery BAT0:
    voltage            =   9865 [mV]
    remaining capacity =   1150 [mWh]
    remaining percent  =      5 [%]
    remaining time     =      5 [min]
    power              =  12627 [mW]
    state              = Discharging
    force-discharge    = 1
    ...

Pobór energii z baterii był mniej więcej stały i jak widać wyżej, stan naładowania z pojemność
3230 mWh w przeciągu jednego odświeżenia spadł do 1170 mWh, zatem ubytek jest na poziomie 2060 mWh.
Trzeba jednak pamiętać, że mWh to nie to samo co mAh. Formuła do wyliczenia mAh z mWh to mWh/V=mAh.
Zatem 2060/10=206 mAh.

Sprawdziłem jak sprawa będzie wyglądać podczas ponownej rekalibracji i tutaj już spadek pojemności
nastąpił z 10% (a nie z 16%) do 5%, tj. pojemność spadła o około 1030 mWh (103 mAh), a nie tak jak
wcześniej o 2060 mWh.

    ...
    Currently discharging battery BAT0:
    voltage            =   9963 [mV]
    remaining capacity =   2180 [mWh]
    remaining percent  =     10 [%]
    remaining time     =      8 [min]
    power              =  15970 [mW]
    state              = Discharging
    force-discharge    = 1
    Press Ctrl+C to cancel.
    --------------------------------------
    Currently discharging battery BAT0:
    voltage            =  10208 [mV]
    remaining capacity =   1150 [mWh]
    remaining percent  =      5 [%]
    remaining time     =      4 [min]
    power              =  16363 [mW]
    state              = Discharging
    force-discharge    = 1
    ...

Zatem jakoś ten poprzedni proces rekalibracji wpłynął na kontroler tej baterii. Być może trzeba by
proces rekalibracji powtórzyć jeszcze z raz czy dwa, by tak by ten spadek pojemności pod koniec
wyeliminować całkowicie, bo co by nie mówić ale przez te 2 lata użytkowania tego laptopa
praktycznie nie przeprowadzany był żaden proces rekalibracji baterii i tak sobie ta maszyna na tych
30-40% naładowania pracowała. Być może też wiek samej baterii się nieco tutaj dokłada, bo jakby
nie patrzeć to ten akumulator ma już ponad 11 lat i nie był nigdy wymieniany.

## Ocena stanu baterii po rekalibracji

Po przeprowadzeniu procesu rekalibracji baterii, dobrze jest rzucić okiem na szacowany pomiar
pojemności, który jest zwracany przez kontroler. Dane w ten sposób uzyskane porównujemy z
wcześniejszymi odczytami.

Informacje o aktualnym stanie baterii można wyciągnąć z `upower` . Przykładowo, gdy kupiłem tego
laptopa, stan jego baterii prezentował się tak jak to widać poniżej:

    # upower /org/freedesktop/UPower/devices/battery_BAT0  -d
    ...
        energy-full:         19.94 Wh
        energy-full-design:  56.16 Wh
        capacity:            35.5057%
    ...

Te powyższe wskazania zostały zarejestrowane chwilę po wprowadzeniu progów ładowania.

Obecnie zaś, tj. prawie dwa lata później, te powyższe wartości (po przeprowadzonym procesie
rekalibracji) wyglądają następująco:

    # upower /org/freedesktop/UPower/devices/battery_BAT0  -d
    ...
        energy-full:         19.87 Wh
        energy-full-design:  56.16 Wh
        capacity:            35.3811%
    ...

Zatem ubytek pojemności, jaki nastąpił przez ten okres prawie dwóch lat, nie jest zbyt duży i jest
to około 0,1246% fabrycznej pojemności akumulatora, tj. 7 mWh (albo 0,6 mAh). Oczywiście ta bateria
i tak była już mocno wyjechana przed zakupem laptopa i jej pojemność była szacowana na zaledwie
35,5% tej fabrycznej. Niemniej jednak, widać na żywym przykładzie, że progi ładowania dość mocno
ograniczają degradację baterii w laptopie.

## Podsumowanie

Przeprowadzanie procesu rekalibracji baterii w laptopie powinno mieć miejsce w sytuacjach, gdy nie
wykorzystujemy całej dostępnej w takim akumulatorze energii. Jeśli potrzebę tego procesu
zignorujemy, to z czasem szacowany przez kontroler czas pracy na baterii będzie na tyle odbiegał od
stanu faktycznego, że przestaniemy na te odczyty w ogóle zwracać uwagę. Dlatego też by uniknąć
nieporozumień, dobrze jest co około dwa miesiące przeprowadzić proces rekalibracji baterii, tak by
jej kontroler zaktualizował sobie wartości dla maksymalnego i minimalnego stanu naładowania
akumulatora, co z powodzeniem przełoży się na bardziej bliskie rzeczywistości wskazania i
dokładniejsze szacunki pracy laptopa na baterii.


[1]: /post/jak-ograniczyc-ladowanie-baterii-w-laptopie-thinkpad-t430/
[2]: https://linrunner.de/tlp/introduction.html
[3]: https://github.com/mkottman/acpi_call
[4]: https://patchwork.kernel.org/project/platform-driver-x86/cover/20211123232704.25394-1-linux@weissschuh.net/
[5]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
[6]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
