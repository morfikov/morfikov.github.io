---
author: Morfik
categories:
- Linux
date: "2016-09-10T12:13:09Z"
date_gmt: 2016-09-10 10:13:09 +0200
published: true
status: publish
tags:
- wifi
- sieć
title: Jak skonfigurować roaming WiFi z wpa_supplicant w linux'ie
---

Bezprzewodowe sieci WiFi są wszędzie. Wystarczy tylko przejść się kawałek po okolicy i przeskanować
eter, a sami zobaczymy, że wynik takiego skanowania zidentyfikuje nam całą masę domowych nadajników.
Oczywiście do większości z nich raczej nigdy nie będziemy mieć dostępu ale są wśród nich takie AP,
do których zwykliśmy się logować. Niekoniecznie te AP muszą należeć do naszej własnej domowej sieci
WiFi. Mogą to być, np. firmowe hotspoty. Nawet jeśli ograniczymy się tylko do naszego domu, to gdy
ten jest nieco większy, to prawdopodobnie jeden bezprzewodowy router nie wystarczy, by pokryć
zasięgiem całą dostępną przestrzeń użytkową. Będziemy musieli dokupić drugi router, czy jakiś
wzmacniacz sygnału. Być może też zdecydujemy się na [transmitery sieciowe
(powerline)](/post/transmitery-sieciowe-tl-wpa4226t-kit-tp-link/). Jest cała masa
urządzeń, które mogą nam pomóc rozwiązać problem słabego zasięgu. To co zwykle łączy te urządzenia,
to fakt, że wszystkie z nich zawierają dodatkowy AP, który trzeba skonfigurować. Pojawia się zatem
problem przełączania między tymi punktami dostępowymi. Można sobie z tym poradzić konfigurując
roaming. W takiej sytuacji przełączanie między sieciami będzie następowało automatycznie, szybko i
bez naszego udziału. W linux'ach roaming można włączyć przy pomocy narzędzia `wpa_supplicant` i w
tym artykule zobaczymy jak tego dokonać.

<!--more-->
## Czym jest roaming WiFi

Przede wszystkim, interesuje nas sytuacja poprawienia zasięgu na danym obszarze. Dlatego też szereg
punktów dostępowych musi być w swoim zasięgu, by zapewnić dobrej jakości sygnał w każdym miejscu
domu czy działki. Jest niemal pewne, że będziemy mieć równocześnie w zasięgu kilka punktów
dostępowych. Jak w takiej sytuacji zachowa się nasz laptop? Bez prawidłowo skonfigurowanego
roamingu będzie on próbował się trzymać jednego AP, aż straci z nim łączność całkowicie. Wtedy
nastąpi rozłączenie i podłączenie do innego AP, który nadaje mocniejszy sygnał.

Po zastosowaniu kilku AP, w pewnych miejscach naszego domu możemy mieć słaby sygnał z jednego AP ale
równocześnie możemy mieć przyzwoity sygnał z drugiego AP. Niemniej jednak, bez roamingu przełączenie
może w ogóle nie nastąpić. W efekcie jakość negocjowanych parametrów sieci WiFi może być bardzo
niska, co zwykle przekłada się na jeszcze słabszy transfer.

Na linux'ach, które wykorzystują jedynie niewielkie menadżery okien, jak Openbox, zwykle nie mamy
poprawnie skonfigurowanego roamingu. Na tych bardziej rozbudowanych środowiskach graficznych (GNOME,
KDE) są raczej dostępne automaty przełączające i nie musimy sobie roamingiem zawracać głowy. Ja z
tych środowisk nie korzystam i niestety musiałem się trochę wysilić, by skonfigurować na swoim
laptopie ten cały roaming WiFi.

## Konfiguracja interfejsu wlan0 w /etc/network/interfaces

Mój laptop dysponuje jednym interfejsem bezprzewodowym `wlan0` . By skonfigurować ten interfejs do
pracy, trzeba poddać edycji plik `/etc/network/interfaces` i dopisać w nim poniższy blok kodu:

    auto wlan0
    iface wlan0 inet manual
        wpa-driver nl80211
        wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
        wpa-roam-default-iface default

    iface default inet dhcp

Linijka z `iface wlan0` musi być ustawiona na `manual` , by mieć możliwość włączenia roamingu.
Sterownik w `wpa-driver` dobieramy w zależności od posiadanej karty. Te nowsze korzystają z
`nl80211` , a te starsze z `wext` . Parametr `wpa-roam` wskazuje plik, który zawiera konfigurację
sieci WiFi. W tym pliku może znajdować się dowolna ilość bloków z ustawieniami parametrów
poszczególnych sieci, do których mamy dostęp. Jeśli w każdej z tych sieci adresację uzyskujemy
przez protokół DHCP i nie stosujemy żadnej dodatkowej konfiguracji, to możemy ustawić sobie tylko
jeden wirtualny interfejs `default` . Z kolei parametr `wpa-roam-default-iface` ustawia domyślny
wirtualny interfejs, który będzie dobierany w przypadku, gdy w pliku
`/etc/wpa_supplicant/wpa_supplicant.conf` zabraknie opcji `id_str` , czyli informacji na temat
wirtualnego interfejsu, z którego ma korzystać dana sieć.

Na potrzeby tego artykułu rozważymy sobie jedynie dwa przypadki, które dotyczą sytuacji posiadania w
bliskim sąsiedztwie dwóch punktów dostępowych. Czemu dwa AP blisko siebie? Bo wtedy jest z nimi
najwięcej problemów podczas konfiguracji. W przypadku, gdy zmieniamy lokalizację o wiele kilometrów,
np. na linii dom \<-\> firma, to zwykle po odpaleniu systemu (np. z hibernacji) zostaniemy
zalogowani do takiej sieci, w której zasięgu jesteśmy. Te sieci ze sobą nie kolidują w żaden sposób
i system nie ma problemu z podłączeniem nas do sieci WiFi w takiej sytuacji.

## Konfiguracja wpa\_supplicant

Konfigurację interfejsu sieciowego mamy z głowy. Teraz musimy skonfigurować sobie narzędzie
`wpa_supplicant` . Powinno być ono już zainstalowane nawet w najbardziej okrojonych linux'ach (jak
coś to trzeba się rozejrzeć za pakietem `wpasupplicant` ). Dokładny opis konfiguracji tego narzędzia
znajduje się w artykule o [konfiguracji sieci WiFi pod
Debianem](/post/konfiguracja-polaczenia-wifi-pod-debianem/). Poniżej zaś znajduje
się jedynie opis potrzebnych nam opcji. Jeśli coś jest niezrozumiałe, to odsyłam do powyższego
artykułu. Przechodzimy zatem do edycji pliku `/etc/wpa_supplicant/wpa_supplicant.conf` .

### Roaming w tej samej sieci (ten sam ESSID i hasło)

Pierwszym przypadkiem, którym się zajmiemy, jest jedna bezprzewodowa sieć o określonych parametrach
(ten sam ESSID i hasło). To, że mamy do czynienia z jedną siecią, nie oznacza, że mamy w niej tylko
jeden AP. Takich punktów dostępowych może być kilka, a system odróżnia je po adresie MAC, tj. po
BSSID.

Załóżmy, że mamy w domu dwa punkty dostępowe: AP1 i AP2. AP1 jest ulokowany na paterze, a AP2 gdzieś
na piętrze. W ten sposób mamy w miarę dobry sygnał na obu kondygnacjach. Nasz cel jest prosty. Gdy
jesteśmy na parterze, chcemy być logowani do AP1, a gdy zabieramy laptopa na pięterko, to ma
automatycznie nastąpić przełączenie do AP2. Konfiguracja w pliku
`/etc/wpa_supplicant/wpa_supplicant.conf` wyglądałaby mniej więcej tak:

    ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev
    eapol_version=2
    ap_scan=1
    fast_reauth=1
    country=PL

    network={
    #   id_str="default"
        bgscan="simple:10:-50:300"
        priority=20
        ssid="Ever_Vigilant"
        psk="12345678"
        disabled=0
    }

Jest tylko jeden blok sieciowy, bo mamy jedną sieć. Natomiast na uwagę zasługuje parametr `bgscan` .
Odpowiada on, jak nazwa wskazuje, za skanowanie sieci w tle. Chodzi o to, że roaming może działać
jedynie w oparciu o pewne parametry sieci, które system musi analizować na bieżąco. Inaczej nie
byłby on w stanie oszacować, gdzie nas podłączyć. W tym przypadku interesuje nas siła sygnału,
który dociera do naszego laptopa.

Jako, że nas interesuje siła sygnału, to korzystamy z algorytmu `simple` , który przyjmuje trzy
parametry, w tym przypadku są to `10:-50:300` . Liczba `-50` odnosi się do progu siły sygnału, czyli
-50 dBm. Ten parametr jest analizowany na bieżąco. Gdy sygnał mamy dobrej jakości, tj. lepszy niż
-50 dBm, to skanowanie sieci jest dokonywane co 300 sekund. W przypadku, gdy siła odbieranego
sygnału spadnie poniżej określonego progu (np. -60, -70, etc), to skanowanie sieci będzie odbywać
się co 10 sekund.

Każde skanowanie sieci zwraca listę wszystkich AP w okolicy oraz jakość odbieranego od nich sygnału.
Takie skanowanie możemy sami zainicjować wpisując w terminalu polecenie `wpa_cli scan` i po chwili
podejrzeć wynik poleceniem `wpa_cli scan_results` , poniżej przykład:

![](/img/2016/09/1.roaming-wifi-linux-skan-sieci-wpasupplicant.png#huge)

Jak widać wyżej, mamy dwa AP z tą samą nazwą sieci `Ever_Vigilant` . Widzimy też, że poziom sygnału
docierającego do mojego laptopa to -13 dBm i -52 dBm. Jesteśmy naturalnie podłączeni w tej chwili do
tego AP z lepszym sygnałem. Niemniej jednak, różnica jest dość spora i dlatego bawimy się w roaming.

W przypadku przechodzenia do pokoju, gdzie jest zlokalizowany AP2, jakość sygnału z AP1 będzie
ulegać stopniowej degradacji, aż w pewnym momencie przekroczy próg -50 dBm. Taka sytuacja
zapoczątkuje agresywne skanowanie sieci co 10 sekund w celu zaktualizowania tej powyższej tabelki.
Wtedy okaże się, że w pewnym momencie sygnał będzie się prezentował tak jak na tym poniższym
obrazku:

![](/img/2016/09/2.roaming-wifi-linux-skan-sieci-wpasupplicant.png#huge)

Zatem nastąpiła zamiana miejsc i teraz AP2 dostarcza przyzwoity sygnał. W takim przypadku po
zastosowaniu opcji `bgscan` , system powinien nas automatycznie przełączyć między tymi dwoma AP i to
bez żadnych powiadomień czy wymagania podjęcia jakichś akcji z naszej strony. Poniżej jest fotka
zalogowanego zdarzenia:

![](/img/2016/09/3.roaming-wifi-linux-przelaczanie-sieci-ten-sam-essid.png#huge)

Wyżej w logu widzimy komunikat `wlan0: disconnect from AP c4:6e:1f:95:ef:fd for new auth to
ec:08:6b:8d:32:aa` . Jest to informacja, która oświadcza nam, że demon roamingowy znalazł AP o
lepszym sygnale i będzie próbował przełączyć się do niego. W powyższym logu mamy dwa zdarzenia,
które efektywnie testują roaming w tej samej sieci.

### Roaming w różnych sieciach (różny ESSID i hasło)

Drugą sytuacją, w której roaming jest wielce niezbędny, jest posiadanie na danym obszarze kilku AP
niepowiązanych ze sobą, tj. należą one do różnych sieci WiFi. W tym przypadku ESSID rozgłaszanej w
okolicy sieci się różni. Dlatego też nie możemy skorzystać z opisanej powyżej konfiguracji dla
narzędzia `wpa_supplicant` . Oczywiście część parametrów będzie taka sama ale musimy określić kilka
bloków sieci, a dokładnie tyle z iloma sieciami zamierzamy się łączyć. Poniżej jest przykładowy plik
`/etc/wpa_supplicant/wpa_supplicant.conf` :

    ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev
    eapol_version=2
    ap_scan=1
    fast_reauth=1
    country=PL

    network={
    #   id_str="default"
        bgscan="simple:10:-60:300"
        priority=20
        ssid="Ever_Vigilant"
        psk="12345678"
        disabled=0
    }
    network={
    #   id_str="default"
        bgscan="simple:10:-60:300"
        priority=20
        ssid="TP-LINK"
        psk="87654321"
        disabled=0
    }

Za wiele się ta konfiguracja nie różni od tej poprzedniej. Niemniej jednak, w każdym z tych dwóch
bloków sieciowych mamy określoną opcję `bgscan` . Informuje ona system, że tylko te dwie sieci mają
być brane pod uwagę przy roamingu, czyli będąc podłączonym do którejś z nich, sygnał będzie
analizowany i przy słabej jego jakości nastąpi przeskanowanie pasma sieciowego. Niemniej jednak, to
do której sieci zostaniemy podłączeni zależy od samej konfiguracji sieci w pliku
`/etc/wpa_supplicant/wpa_supplicant.conf` , np. priorytety mają znaczenie.

Tak obecnie wygląda eter w mojej okolicy:

![](/img/2016/09/4.roaming-wifi-linux-skan-sieci-wpasupplicant.png#huge)

Dwie moje sieci są oznaczone jako `Ever_Vigilant` oraz `TP-LINK` . Obie te sieci zostały określone w
konfiguracji `wpa_supplicant` i na obu z nich mamy ustawiony roaming. Wystarczy, że sygnał spadnie
poniżej tych -60 dBm, a system przełączy nas do drugiego AP:

![](/img/2016/09/5.roaming-wifi-linux-przelaczanie-sieci-rozne-essid.png#huge)

W tym przypadku, jako że są różne nazwy sieci, w logu pojawiła się informacja na temat tej, do
której zostaliśmy przyłączeni.

W powyższej konfiguracji definiowaliśmy sieci, które mają być objęte roamingiem przez określenie w
odpowiednich blokach sieciowych opcji `bgscan` . Ta opcja może także powędrować do sekcji globalnej.
W takiej sytuacji roaming zostanie włączony na wszystkich zdefiniowanych w pliku
`/etc/wpa_supplicant/wpa_supplicant.conf` sieciach.

## Roaming WiFi i bonding

Co ciekawe, roaming działa bez zarzutu z interfejsem `bond0` . Zatem jeśli [mamy na swoim linux'ie
skonfigurowany bonding](/post/konfiguracja-interfejsow-bond-bonding/) między
interfejsem przewodowym i bezprzewodowym i chcielibyśmy korzystać z roamingu WiFi, to nie powinniśmy
napotkać żadnych problemów przy korzystaniu z takiego ustawienia.
