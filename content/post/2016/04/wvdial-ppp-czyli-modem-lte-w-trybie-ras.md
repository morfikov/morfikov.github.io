---
author: Morfik
categories:
- Linux
date: "2016-04-14T19:01:46Z"
date_gmt: 2016-04-14 17:01:46 +0200
published: true
status: publish
tags:
- sieć
- lte
- ppp
- debian
- modem
- huawei
- e3372
GHissueID: 410
title: Wvdial i PPP, czyli modem LTE w trybie RAS
---

Jak wielu użytkowników linux'a zapewne wie, modem GSM/UMTS/LTE może pracować w kilku trybach.
Najpopularniejszym z nich jest tryb RAS wykorzystujący interfejsy udostępniane przez to urządzenie
w katalogu `/dev/` , zwykle `ttyUSB0` , `ttyUSB1` , etc. By taki modem mógł nawiązać połączenie z
siecią, potrzebny jest demon [PPP](https://pl.wikipedia.org/wiki/Point_to_Point_Protocol). O trybie
RAS wspominałem już parokrotnie, min. we wpisach dotyczących [konfiguracji połączenia LTE w
RBM/Play](/post/darmowy-internet-lte-od-rbmplay/) jak i przy omawianiu [problemów
z resolver'em DNS w przypadku
Aero2](/post/aero2-w-polaczeniu-z-dnsmasq-dnscrypt-proxy/). Generalnie ten tryb
różni się trochę od [trybu
NDIS(NCM)](/post/konfiguracja-modemu-lte-w-trybie-ndis-ncm/) głównie tym, że tutaj
nie uzyskamy większych prędkości niż 20-30 mbit/s. Niemniej jednak, jeśli nie mamy dobrej jakości
połączenia LTE, lub nasz modem z jakiegoś powodu pod linux'em nie potrafi pracować w trybie NDIS,
to możemy skonfigurować połączenie w trybie RAS wykorzystując do tego celu `wvdial` oraz demona PPP.

<!--more-->
## Instalacja wvdial i demona PPP

Jeśli nasz modem nie jest wykrywany poprawnie, tj. po jego podłączeniu nie mamy w katalogu `/dev/`
żadnych urządzeń `ttyUSB*` , to będziemy musieli zainteresować się pakietem `usb-modeswitch` . W
przypadku, gdy w katalogu `/dev/` mamy do dyspozycji szereg interfejsów modemu, to wystarczy, że
doinstalujemy sobie pakiet `wvdial` . Nie musimy osobno instalować pakietu `ppp` , bo ten powinien
zostać dociągnięty w zależnościach. Obie te powyższe rzeczy znajdują się w repozytoriach debiana,
zatem instalacja ich nie powinno sprawić trudności.

## Polecenia AT

By umiejętnie operować na `wvdial` musimy nauczyć się obsługi poleceń AT. Na dobrą sprawę, to
wszystkie polecenia AT można przesłać za pomocą `echo` bezpośrednio na interfejs modemu w katalogu
`/dev/` . Problem z tymi poleceniami AT jest taki, że są one zwykle skomplikowane, długie i trudne
do zapamiętania. Dlatego będziemy posługiwać się plikiem `/etc/wvdial.conf` , gdzie te wszystkie
polecenia zostaną zebrane. Tak czy inaczej, każde polecenie AT ma swoją składnię, która wygląda
mniej więcej tak:

    command line prefix     delimited with semicolon         termination character
           |                     |     read command for checking   /
           |      subparameter   |    current subparameter values /
           |           |         /                |              /
           ATCMD1 CMD2=12; +CMD1; +CMD2=,,15; +CMD2?; +CMD2=?<CR>
              |              |           \                 |
              |        extended command   \      test command for checking
     basic command     (prefixed with +)   |    possible subparameter values
     (no + prefix)                   subparameters
                                     may be omitted

Najważniejsze z tego schematu, to zapamiętać, że wszystkie polecenia są poprzedzone prefiksem `AT` ,
chyba, że precyzujemy je w jednej linii. W takim przypadku oddzielamy te polecenia od siebie za
pomocą `;` i każda kolejna komenda jest już bez prefiksu. Mamy kilka typów poleceń: podstawowy
(basic), rozszerzony (extended), producenta (vendor), sprawdzający (checking) i testujący (testing).
Jeśli modem obsługuje polecenia AT, to prawdopodobnie obsługuje wszystkie albo znaczną większość
poleceń podstawowych i rozszerzonych. Te dwa typy można odróżnić po znaku `+` , który występuje
tylko w przypadku poleceń rozszerzonych. Producent modemu, może też wprowadzić własne polecenia,
które mogą dotyczyć tylko jego produktów lub pewnych określonych modeli. Wszystkie takie polecenia
zaczynają się od znaku `^` lub `%` . Polecenie sprawdzające ma na celu odpytanie o aktualną wartość
danego parametru. Takie polecenie jest zakończone `?` . Z kolei polecenie testujące zwraca możliwe
do ustawienia wartości. Jest ono zakończone `=?` .

Poniżej przykładowe wykorzystanie poleceń sprawdzających i testowych:

    --> Sending: AT+CFUN=?
    +CFUN: (0,1,4,5,6,7,8,10,11),(0,1)
    OK
    --> Sending: AT+CFUN?
    +CFUN: 1
    OK
    --> Sending: AT+CFUN=0
    OK

Polecenie `AT+CFUN=?` zwróciło nam możliwe do określenia wartości. Mamy także formę jaką to
polecenie może przyjąć. Następnie `AT+CFUN?` odpytuje modem o aktualnie ustawioną wartość i widzimy,
że została zwrócona `1` . Przy pomocy `AT+CFUN=0` przestawiamy wartość na `0` . W tym przypadku,
drugi parametr jest opcjonalny i nie trzeba go określać. Każde polecenie ma swój własny szyk i swoje
własne parametry i wartości. Niestety na necie nie znalazłem dobrego źródła opisującego polecenia AT
ale gdzieniegdzie można znaleźć mniej lub bardziej obszerne informacje na temat konkretnego
polecenia. Natomiast pełna lista poleceń obsługiwanych przez modem może zostać zwrócona przez
komendę `AT+CLAC` .

Mamy zatem krótkie wprowadzenie do tego czym tak naprawdę będziemy się zajmować. Przejdźmy teraz do
konfiguracji `wvdial` .

## Konfiguracja wvdial

W dużej mierze połączenie modemu w trybie RAS odbywa się automatycznie. Demon PPP będzie startowany
przez `wvdial` i jedyne czego nam potrzeba, to odpowiedniej konfiguracji dla tego narzędzia.
Wszystkie poniższe wpisy będziemy umieszczać w pliku `/etc/wvdial.conf` . Większość wpisów będzie
zawierać polecenia AT, które są przesyłane do modemu i w ten sposób mówią mu jak ma się zachowywać,
np. wymusić tryb LTE.

### Sekcja [Dialer Defaults]

Na sam początek wygenerujmy sobie sekcję `[Dialer Defaults]` . Będzie ona inna w zależności od
posiadanego modemu. W tym przypadku mamy do czynienia z modemem Huawei E3372s-153 w wersji
NON-HiLink. Pamiętajmy też, że tylko wersje NON-HiLink mogą być konfigurowane w trybie RAS.
Wszystkie HiLink'i mają swój interfejs webowy dostępny zwykle pod adresem `192.168.8.1` i nie da
rady zarządzać nimi w opisany niżej sposób. Podłączamy modem do portu USB i wydajemy w terminalu to
poniższe polecenie:

    # wvdialconf
    Scanning your serial ports for a modem.
    ...
    Modem configuration written to /etc/wvdial.conf.
    ttyUSB0<Info>: Speed 9600; init "ATQ0 V1 E1 S0=0"
    ttyUSB1<Info>: Speed 9600; init "ATQ0 V1 E1 S0=0"

Polecenie `wvdialconf` ma na celu wykrycie interfejsów modemu i odpowiednie ich skonfigurowanie. Jak
widzimy wyżej, modem Huawei E3372 ma dwa interfejsy `ttyUSB0` oraz `ttyUSB1` . Wygenerowana sekcja
domyśla nie jest zbyt rozbudowana ale, jak nazwa mówi, jest ona domyślna. Znajdują się w niej te
opcje, które zwykle są wykorzystywane przy wydawaniu każdego polecenia AT. Możemy ją zostawić w
spokoju albo też pokusić się o dostosowanie poszczególnych parametrów. Wszystkie opcje w `[Dialer
Defaults]` mogą zostać napisane, przez co konfiguracja w pliku `/etc/wvdial.conf` może być
rozdzielona na szereg sekcji (o tym później). Poniżej znajdują się te opcje, które zwykle powinny
powędrować do sekcji domyślnej.

#### Typ modemu

Typ modemu jest określany przez `wvdialconf` i raczej nie powinniśmy tej opcji ruszać:

    Modem Type = Analog Modem

#### Domyślny interfejs modemu

Jako, że ten modem ma kilka interfejsów, to któryś z nich możemy określić jako domyślny. Oczywiście
nie musimy tego robić ale w takim przypadku, we wszystkich pozostałych sekcjach będziemy musieli
dodać dodatkową linijkę, co może wpłynąć niekorzystnie na czytelność pliku konfiguracyjnego.

    Modem = /dev/ttyUSB1

#### Prędkość komunikacji z modemem

`wvdial` komunikuje się z modemem. Każda komunikacja może zachodzić szybciej lub wolniej, a zależy
to od prędkości z jaką się ona odbywa. [Parametr Baud](https://pl.wikipedia.org/wiki/Bod) nie wpływa
na prędkość połączenia internetowego.

    Baud = 9600

#### Próby i czas nawiązywania połączeń

W przypadku, gdy `Dial Attempts` zostanie ustawiony na `0` , modem będzie próbował połączyć się w
nieskończoność. Z kolei `Dial Timeout` określa czas (w sekundach) jednej próby połączenia.

    Dial Attempts = 3
    Dial Timeout = 30

#### Automatyczne nawiązywanie połączenia

Połączenie może zostać zerwane w losowy sposób przez drugą ze stron. W takim przypadku modem może
spróbować automatycznie nawiązać je ponownie:

    Auto Reconnect = 1

#### Konfiguracja DNS

Po zestawieniu połączenia z internetem, demonowi PPP zostanie przekazana konfiguracja sieci za
sprawą protokołu DHCP. W niej zawarta będzie także informacja o resolverze DNS. Możemy te wpisy
zignorować i ręcznie wskazać providera DNS, np. google, czy opendns. Bardzo przydatny parametr
zwłaszcza w przypadku, gdy [szyfrujemy zapytania DNS przy pomocy
dnscrypt-proxy](/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/).

    Auto DNS = 0

Trzeba jednak pamiętać, by zmienić także plik `/etc/ppp/peers/wvdial` . Konkretnie chodzi o
wykomentowanie poniższej linijki:

    #usepeerdns

#### Test ustawień DNS

Po tym jak połączenie zostanie nawiązane, `wvdial` może przetestować resolver DNS. Możemy także
określić dwie domeny, które w tym teście zostaną odpytane.

    Check DNS = 1
    DNS Test1 = wp.pl
    DNS Test2 = google.com

#### Test domyślnej trasy

Podobnie jak w przypadku resolver'a DNS możemy poddać testom domyślną trasę.

    Check Def Route = 1

#### Rozłącz, gdy nieaktywny

Istnieje także możliwość rozłączenia połączenia internetowego w przypadku, gdy nie jest ono
wykorzystywane. Wartość poniższego parametru jest w sekundach, natomiast `0` wyłącza tę opcję:

    Idle Seconds = 0

#### Domyślny numer

Z tego co wyczytałem na necie, to praktycznie każdy provider mobilnego internetu LTE operuje na
numerze `*99#` . Możemy go określić w tej sekcji, co powinno odciążyć plik konfiguracyjny w
przypadku, gdy mamy wiele sekcji odnoszących się do konfiguracji różnych operatorów.

    Phone = *99#

#### Domyślne polecenia AT

`wvdial` jest w stanie przesłać modemowi do 9 różnych poleceń ( `Init1` - `Init9` ). W każdej z tych
opcji, polecenia AT mogą być łączone za pomocą `;` . Zatem samych poleceń AT można wysłać więcej.

    Init1 = ATZ
    Init2 = ATQ0 V1 E0 H0 S0=0

Powyżej mamy podstawowe polecenia, które powinny być przesyłane do modemu za każdym razem ilekroć
zamierzamy korzystać z poleceń AT. W taki sposób pozostałe sekcje będą mieć do wykorzystania
`Init3` - `Init9` .

### Startowanie i stopowanie modemu

Sekcję `[Dialer Defaults]` mamy z głowy. Możemy przejść do konfiguracji szeregu sekcji, które będą
wykorzystywane ilekroć będziemy próbować nawiązać połączenie z internetem. Poniższe dwie sekcje
odpowiadają kolejno za włączenie i wyłączenie modemu:

    [Dialer modem-start]
    Init1 = AT+CFUN=1

    [Dialer modem-stop]
    Modem = /dev/ttyUSB0
    Init1 = AT+CFUN=0

Polecenie `+CFUN` obsługuje tryby pośrednie, choć nie wszystkie są dostępne w każdym modelu modemu.
Z reguły będą to numerki 2-9. Dobrze jest zajrzeć w dokumentację techniczną producenta, by ustalić
jakie dodatkowe tryby pracy nasz modem obsługuje.

### PIN do karty SIM

Karty SIM są z reguły zabezpieczone kodem PIN. W przypadku modemów, takie rozwiązanie jest dość
nieporęczne i nie zalecałbym stosowania tutaj PIN'u. A to z tego względu, że przy każdym połączeniu,
trzeba będzie ten PIN podawać. Można go, co prawda, zapisać w konfiguracji `wvdial` ale wtedy i tak
narażamy się na jego odczytanie. Tak czy inaczej, jeśli mamy kartę SIM zabezpieczoną kodem PIN i nie
chcemy tego kodu usuwać, to możemy stworzyć sekcję, która ten kod PIN poda za nas:

    [Dialer pin-use]
    Init3 = AT+CPIN=8888

### Informacje o jakości sygnału

Poniższa sekcja jest w stanie nam dostarczyć informacji na temat operatora i jakości sygnału, który
otrzymujemy od BTS.

    [Dialer info-gsm]
    Modem = /dev/ttyUSB0
    Init4 = AT+COPS?
    Init5 = AT^HCSQ?

### Informacje na temat konfiguracji modemu

Ta poniższa sekcja umożliwi nam ustalenie producenta i modelu modemu, wersji użytego oprogramowania
oraz szeregu numerów, np. numer seryjny karty SIM. Ponadto, zostaną nam tez wyświetlone informacje
na temat włączonych portów.

    [Dialer info-device]
    Modem = /dev/ttyUSB0
    Init4 = AT+CGMI;+CGMM;+CGSN;+CIMI;+CGMR;+CPAS;+CBC;^ICCID?
    Init5 = AT^SETPORT=?;^SETPORT?
    Init6 = AT^GETPORTMODE

Jeśli chodzi zaś o informacje o konfiguracji pasm i częstotliwości na jakich aktualnie operuje
modem, to powie nam to następująca sekcja:

    [Dialer info-config]
    Modem = /dev/ttyUSB0
    Init4 = AT^SYSCFG=?;^SYSCFG?
    Init5 = AT^SYSCFGEX=?;^SYSCFGEX?
    Init6 = AT^SYSINFO
    Init7 = AT^SYSINFOEX

### Konfiguracja pasm i częstotliwości

Jeśli nie jesteśmy zadowoleni z obecnej konfiguracji pasm i częstotliwości, to możemy
przeprogramować modem. W ten sposób możemy np. wymusić LTE, czy ustawić preferencję LTE później
UMTS i GSM. Jeśli mamy starszy modem, to potrzebna nam jest ta sekcja:

    [Dialer set-old]
    Init4 = AT^SYSCFG=2,0,3FFFFFFF,1,2

Jeśli jest to modem nowszy, to z kolei potrzebujemy tej sekcji:

    [Dialer set-new]
    Init4 = AT^SYSCFGEX="030201",3FFFFFFF,1,2,800C5,,

### Zmiana trybu pracy modemu

Modem może pracować w dwóch trybach: jako CD-ROM/pendrive lub faktyczny modem. Każdy z tych trybów
ma inny zestaw portów, które można sobie skonfigurować.

    [Dialer set-iface]
    Init4 = AT^SETPORT="FF;12,10,16,A2"
    Init5 = AT^RESET

### Restart modemu

Przyda nam się też sekcja umożliwiająca restart modemu.

    [Dialer modem-reboot]
    Init3 = AT^RESET

### Konfiguracja połączenia operatorów

Ostatnią rzeczą, której potrzebujemy to sekcje z ustawieniami dla poszczególnych operatorów GSM.
Każdy z operatorów zwykle dysponuje inną konfiguracją, którą musimy podać w `wvdial` . Dla
przykładu 3 sekcje:

    [Dialer aero2]
    Init6 = AT+CGDCONT=1,"IP","darmowy"
    Password = "aero"
    Username = "aero"

    [Dialer red_bull_mobile-auto]
    Init6 = AT+CGDCONT=1,"IP","internet"
    Username = "blank"
    Password = "blank"

    [Dialer red_bull_mobile-lte]
    Init4 = AT^SYSCFGEX="03",3FFFFFFF,1,2,800C5,,
    Init6 = AT+CGDCONT=1,"IPV4V6","internet"
    Username = "blank"
    Password = "blank"

## Operowanie na wvdial

Takich sekcji w pliku `/etc/wvdial.conf` możemy umieścić dowolną ilość. By przekazać polecenie do
`wvdial` , który z kolei prześle je do modemu, w argumencie podajemy nazwę sekcji. Możemy ich podać
kilka, w zależności od potrzeb. Poniżej przykładowe wywołanie programu:

    # wvdial modem-start info-config

Część z opcji użytych w konkretnych sekcjach może się nakładać na siebie, głównie te z `Init*` i
`modem` , tak jak to widać na powyższym przykładzie. Weźmy, np. parametr `modem` . Występuje on
zarówno w sekcji `[Dialer Defaults]` jak i w `[Dialer info-config]` . Zatem która wartość zostanie
użyta? Decyduje tutaj kolejność sekcji podanych w argumencie. Zatem parametr `modem` zostanie
przepisany z domyślnego `/dev/ttyUSB1` na `/dev/ttyUSB0` . Podobnie sprawa ma się w przypadku
pozostałych opcji.

### Informacje o portach modemu

We wpisie poświęconym [konfiguracji modemu bez
usb-modeswitch](/post/modem-lte-huawei-e3372-bez-usb-modeswitch/) wypracowałem
sobie rozwiązanie, które przestawiło modem w odpowiedni tryb pracy. Zachęcam zatem do zajrzenia i
przeczytania tego tekstu, który w dużej mierze pomoże nam skonfigurować modem do pracy pod linux'em,
czy nawet pod OpenWRT na routerze jeśli takowy posiadamy.

### Sprawdzenie operatora i jakości sygnału

Karty SIM mogą pochodzić od różnych operatorów GSM. Ci z kolei mają swoje nadajniki rozmieszczone w
różnych lokalizacjach i nie zawsze blisko naszego miejsca zamieszkania. Praktycznie każdy operator
udostępnia internet LTE, który można przetestować bez zobowiązań zamawiając czy kupując sobie
starter. Jeśli planujemy ocenić jakość połączenia u każdego z operatorów, to możemy posłużyć się
poniższym poleceniem:

    $ wvdial modem-start info-gsm
    ...
    --> Sending: AT+CFUN=1
    OK
    --> Sending: ATQ0 V1 E0 H0 S0=0
    OK
    --> Sending: AT+COPS?
    +COPS: 0,0,"Red Bull MOBILE",7
    OK
    --> Sending: AT^HCSQ?
    ^HCSQ:"LTE",53,45,181,18
    OK

W tym przypadku podłączeni jesteśmy do sieci RBM/Play . Cyfra `7` w wyniku `AT+COPS` , którą widzimy
na końcu mówi nam, że połączenie jest nawiązane po LTE. Tę samą wiadomość mamy w `AT^HCSQ` , z tym,
że już zapisaną czytelnie dla człowieka. Mamy tam również cztery inne wartości. Co one oznaczają i
jak je zinterpretować? Spójrzmy na poniższą rozpiskę:

    # <Signal<,<RSSI>,<RSRP>,<SINR>,<RSRQ>
    # RSSI -- Reference Signal Receive Power (wskaźnik siły odbieranego sygnału włącznie z zakłóceniami)
    # RSRP -- Reference Signal Received Quality (miara mocy sygnału)
    # SINR -- Signal-to-Interference-Plus-noise Ratio (miara jakości sygnału użytkowego w stosunku do zakłóceń szumu)
    # RSRQ -- Received Signal Strength Indicator, only for LTE (miara jakości sygnału)
    #---------------------------------------------------------------
    # 0 RSSI < -120 dBm            | 0 RSRP < -140 dBm             | RSSI formula:
    # 1 -120 dBm ≤ RSSI < -119 dBm | 1 -140 dBm ≤ RSRP < -139 dBm  |   dBm= value1 -120
    # 2 -119 dBm ≤ RSSI < -118 dBm | 2 -139 dBm ≤ RSRP < -138 dBm  | RSRP formula:
    # ...                          | ...                           |   dBm= value2 -140
    # 53 -68 dBm ≤ RSSI < -67 dBm  | 45 -96 dBm ≤ RSRP < -95 dBm   |
    # ...                          | ...                           |
    # 94 -27 dBm ≤ RSSI < -26 dBm  | 95 -46 dBm ≤ RSRP < -45 dBm   |
    # 95 -26 dBm ≤ RSSI < -25 dBm  | 96 -45 dBm ≤ RSRP < -44 dBm   |
    #                              | 97 -44 dBm ≤ RSRP             |
    #                              | 255 unknown or undetectable   |
    #---------------------------------------------------------------
    # 0 SINR < -20 dB              | 0 RSRQ < -19.5 dB             | SINR formula:
    # 1 -20 dB ≤ SINR < -19.8 dB   | 1 -19.5 dB ≤ RSRQ < -19 dB    |   dB= 0.2 x value3 -20
    # 2 -19.8 dB ≤ SINR < -19.6 dB | 2 -19 dB ≤ RSRQ < -18.5 dB    | RSRQ formula:
    # ...                          | ...                           |   dB= 0.5 x value4 -19.5
    # 181 16 dB ≤ SINR < 16.2 dB   | 18 -11 dB ≤ RSRQ < -10.5 dB   |
    # ...                          | ...                           |
    # 249 29.6 dB ≤ SINR < 29.8 dB | 32 -4 dB ≤ RSRQ < -3.5 dB     |
    # 250 29.8 dB ≤ SINR < 30 dB   | 33 -3.5 dB ≤ RSRQ < -3 dB     |
    # 251 30 dB ≤ SINR             | 34 -3 dB ≤ RSRQ               |
    # 255 unknown or undetectable  | 255 unknown or undetectable   |
    #--------------------------------------------------------------|

Zatem wskazania dla mojego połączenia wyglądają następująco: `RSSI -68 dBm` , `RSRP -96 dBm` ,
`SINR 16 dB` oraz `RSRQ -11 dB` . Czy to dużo czy mało? [Zgodnie z tym co znalazłem pod tym
linkiem](https://tplinkforum.pl/t/parametry-sygnalu-lte/7534), to nie jest najgorzej:

    RSSI (większa aktywność transferu danych, większe RSSI)

    RSRP (od -140 do -44 dBm)
    więcej niż -79dBm - bardzo dobrze blisko BTS-a
    od -80dBm do -90dBm - dobrze
    od -91 do -100 - trzeba pomyśleć o zmianie położenie modemu/routera lub zakupie anteny
    mniej niż -100 - bardzo źle praktycznie nie da się pracować.

    SINR
    więcej niż 21dB - bardzo dobrze blisko BTS-a
    od 13dB do 20dB - dobrze
    od 0dB do 12dB - trzeba pomyśleć o zmianie położenie modemu/routera lub zakupie anteny zewnętrznej
    mniej niż 0dB - bardzo źle praktycznie nie da się pracować.

    RSRQ (od -19.5dB do -3 dB)
    więcej niż -9dB - bardzo dobrze, blisko BTS-a
    od -10dB do -15dB - dobrze
    od -16dB do -20dB - trzeba pomyśleć o zmianie położenie modemu/routera lub zakupie anteny
    poniżej -20dB -  bardzo źle praktycznie nie da się pracować.

### Wymuszanie konkretnego trybu połączenia

Wyżej w pliku konfiguracyjnym mieliśmy sekcję `[Dialer set-new]` i `[Dialer set-old]` . Przy ich
pomocy jesteśmy w stanie wymusić, by modem pracował w konkretnym paśmie czy technologii. Bardzo
przydatna rzecz jeśli zamierzamy korzystać z [darmowego połączenia LTE od
RBM/Play](/post/darmowy-internet-lte-od-rbmplay/). Oczywiście to tylko przykład. By
odpowiednio ustawić modem, trzeba zapoznać się ze specyfikacją poleceń
[AT^SYSCFGEX](http://wiki.bez-kabli.pl/index.php?title=AT%5ESYSCFGEX) oraz
[AT^SYSCFG](http://wiki.bez-kabli.pl/index.php?title=AT%5ESYSCFG). Warto też podkreślić, że
wykorzystujemy tylko jedno z tych poleceń, w zależności od tego czy posiadamy nowszy czy też starszy
model modemu.

### Uzyskiwanie połączenia z siecią

Po włączeniu odpowiednich portów, ustawieniu częstotliwości, oraz skonfigurowaniu sekcji operatorów
GSM, możemy w końcu przejść do zestawienia połączenia. W tym przypadku spróbujemy nawiązać
połączenie w trybie preferującym LTE do sieci RMB/Play. W tym celu, w terminalu wpisujemy to
poniższe polecenie:

    # wvdial modem-start set-new red_bull_mobile-auto

W logu powinniśmy zaobserwować szereg poleceń AT, które definiowaliśmy w wyżej użytych sekcjach:

    --> WvDial: Internet dialer version 1.61
    --> Initializing modem.
    --> Sending: AT+CFUN=1
    OK
    --> Sending: ATQ0 V1 E0 H0 S0=0
    OK
    --> Sending: AT^SYSCFGEX="030201",3FFFFFFF,1,2,800C5,,
    OK
    --> Sending: AT+CGDCONT=1,"IP","internet"
    OK
    --> Modem initialized.

Następnie modem zadzwoni pod określony numer i spróbuje nawiązać połączenie:

    --> Sending: ATDT*99#
    --> Waiting for carrier.
    CONNECT

Połączenie zostało nawiązane, zatem `wvdial` startuje demona PPP.

    --> Carrier detected.  Starting PPP immediately.
    --> Starting pppd at Thu Apr 14 17:09:22 2016
    --> Pid of pppd: 80844
    --> Using interface ppp0

Teraz następuje uwierzytelnianie danymi podanymi w sekcji operatorów GSM:

    --> Authentication (CHAP) started
    --> Authentication (CHAP) successful

I po chwili jest uzyskiwana adresacja za pomocą protokołu DHCP:

    --> local  IP address 10.129.85.156
    --> remote IP address 10.64.64.64
    --> Script /etc/ppp/ip-up run successful

Sprawdzane są też DNS oraz trasa domyślna:

    --> Default route Ok.
    --> Nameserver (DNS) Ok.
    --> Connected... Press Ctrl-C to disconnect

I w taki oto sposób połączenie z siecią zostało nawiązane i możemy przeglądać sobie net. By
zakończyć połączenie, wciskamy Ctrl-C . I to w zasadzie cała sztuka operowania `wvidal` i demonem
PPP. Aktualna konfiguracja zawarta w pliku [/etc/wvdial.conf znajduje się zawsze na
github'ie](https://github.com/morfikov/files/blob/master/configs/etc/wvdial.conf).
