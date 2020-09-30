---
author: Morfik
categories:
- OpenWRT
date: "2016-05-04T20:26:35Z"
date_gmt: 2016-05-04 18:26:35 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- sieć
- router
title: Sieć bezprzewodowa WiFi w OpenWRT (WLAN)
---

Sieć bezprzewodowa w dzisiejszych czasach to podstawa. Z reguły routery posiadają jedno radio
operujące na częstotliwości 2,4 GHz. Te nieco nowsze (i droższe) modele mają do dyspozycji dwa
radia: 2,4 GHz oraz 5 GHz. OpenWRT zapewnia wsparcie zarówno dla sieci pracującej w paśmie 2.4 GHz
jak i tej nadającej w 5 GHz. Konfiguracja tych pasm w OpenWRT różni się nieco. Weźmy dla przykładu
obsługę kanału 12 i 13, którą dotyczy tylko sieci pasma 2.4 GHz. Podobnie sprawa ma się z
szerokością kanałów, która jest inna w przypadku obu tych pasm. Niemniej jednak, większość opcji
pozostaje taka sama i w tym artykule rzucimy okiem na zagadnienie konfiguracji sieci WiFi w OpenWRT.

<!--more-->
## Ustawienia sieci WiFi

Sieć bezprzewodowa różni się trochę od tej przewodowej. W przypadku sieci po kablu nie musimy
martwić się o zabezpieczenia transmisji, tak jak to ma miejsce w przypadku WiFi. Dlatego też z tego
powodu po świeżym flash'owaniu routera, sieć bezprzewodowa w OpenWRT jest domyślnie wyłączona.
Musimy pierw odpowiednio ją skonfigurować i ustawić wszystkie niezbędne do poprawnej pracy
parametry. Aktualny stan sieci WiFi możemy sprawdzić z poziomu routera wydając w terminalu polecenie
`wifi status` . Zwróci ono szereg informacji dotyczących każdego radia.

![](/img/2016/05/1.konfiguracja-radia-wifi-openwrt.png#huge)

W powyższym przypadku mamy do czynienia z siecią 5 GHz ( `"hwmode": "11a",` ). Jest ona podniesiona
`"up": true,` . Mamy tam także informacje dotyczącą konfiguracji samego radia (aktualny kanał i jego
szerokość, kraj czy moc transmisyjna). Niżej zaś mamy przypisany interfejs `wlan0` oraz konfigurację
sieci wliczając w to identyfikator sieci SSID, rodzaj szyfrowania, hasło do sieci i szereg innych
aspektów pracy sieci, które możemy skonfigurować przez plik `/etc/config/wireless` .

Jeśli z jakichś powodów rozsypie się nam konfiguracja sieci WiFi, to w prosty sposób możemy
przywrócić ją do ustawień domyślnych. Możemy to zrobić wydając w terminalu polecenie
`wifi detect` . Trzeba tylko pamiętać, by przed jego wydaniem usunąć plik `/etc/config/wireless` .
Jeśli zapomnimy to zrobić, to wspomniane polecenie zwróci pusty wynik.

Przejdźmy zatem do konfiguracji sieci bezprzewodowej. Jeśli jeszcze nie jesteśmy zalogowani na
router, to zróbmy to czym prędzej. Następnie przy pomocy `vi` edytujemy plik
`/etc/config/wireless` . W zależności od routera, a konkretnie od tego jakie pasma częstotliwości
obsługuje, w tym pliku możemy mieć kilka sekcji `config wifi-device` oraz `config wifi-iface` .
Poniżej znajdują się przykładowe sekcje:

![](/img/2016/05/2.konfiguracja-wifi-radio-interfejs-openwrt.png#huge)

Każda taka para konfiguruje jedną sieć WiFi. W `config wifi-device` określamy konfigurację dla
urządzenia (karta WiFi). Natomiast w `config wifi-iface` konfigurujemy interfejs. Takich sekcji
może być dowolna ilość. Niemniej jednak, tylko jedna sekcja `config wifi-device` może być aktywna w
tym samym czasie. Te restrykcje nie dotyczą zaś sekcji `config wifi-iface` . W ten sposób możemy na
jednej karcie WiFi uruchomić szereg sieci bezprzewodowych z różnymi SSID. To która sekcja jest
włączona możemy określić przez odpowiednie ustawienie opcji `disabled` . Jeśli nie jesteśmy pewni,
który interfejs jest przypisany do jakiej karty, to zawsze możemy to ustalić za pomocą polecenia
`iwinfo` :

![](/img/2016/05/3.iwinfo-openwrt-interfejs-wifi.png#big)

W `config wifi-device` zostały użyte następujące opcje:

  - `type` -- odpowiada za typ użytego sterownika i jest on wykrywany podczas pierwszego
    uruchomienia routera. Nie wymaga ręcznego dostosowania.
  - `phy` -- określa fizyczny interfejs powiązany z tą sekcją.
  - `channel` -- określa kanał, do którego będzie podpięta sieć WiFi. Na warunki polskie, w
    przypadku pasma 2,4 GHz, mamy do dyspozycji 1-13. Z kolei jeśli chodzi o pasmo 5GHz, to mamy
    36-64 oraz 100-140 z przeskokiem co 4. Możemy także wybrać tryb `auto` , który przeskanuje eter
    w poszukiwaniu ewentualnych zakłóceń i podłączy nas do najlepszego z dostępnych kanałów.
  - `hwmode` -- określa z jakiego pasma korzystamy: 11g dla 2.4 GHz, 11a dla 5 GHz.
  - `path` -- to ścieżka do urządzenia i jest ona wykrywana podczas pierwszego uruchomienia routera.
    Zwykle nie wymaga ręcznego dostosowania.
  - `htmode` -- określa szerokość kanałów.
  - `noscan` -- dezaktywuje mechanizm wykrywania zakłóceń przy korzystaniu z kanałów o szerokości 40
    MHz.
  - `country*` -- konfigurują kod kraju, co warunkuje dostępne kanały i maksymalną moc transmisyjną.
  - `txpower` -- ustawia moc transmisyjną.
  - `log_level` -- odpowiada za poziom logowania zdarzeń. Im niższy numerek, ty więcej logów będzie
    generowanych. Dostępne wartości od 0-4.
  - `disabled` -- wyłącza z konfiguracji daną sekcję.

W sekcji `config wifi-iface` zostały użyte następujące opcje:

  - `device` -- określa wykorzystywaną kartę WiFi, tę którą została skonfigurowana w jednej z sekcji
    `wifi-device` .
  - `network` -- odpowiada za logiczny interfejs określony w pliku `/etc/config/network` , do
    którego sieć bezprzewodowa zostanie podpięta.
  - `mode` -- definiuje tryb działania radia.
  - `ssid` -- określa nazwę sieci.
  - `encryption` -- ustawia rodzaj wykorzystywanego szyfrowania.
  - `key` -- definiuje hasło do sieci WiFi.
  - `disabled` -- wyłącza z konfiguracji daną sekcję.
  - `hidden` -- wyłącza rozgłaszanie sieci.
  - `isolate` -- izoluje poszczególnych klientów sieci WiFi.
  - `mac*` -- opcje konfigurujące mechanizm kontroli adresów MAC.
  - `wps_*` -- opcje dotyczące protokołu WPS.

Wykorzystane wyżej opcje mogą się różnić w zależności od posiadanej karty bezprzewodowej. W tym
przypadku mamy do czynienia z routerem [TP-Link Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html) mającym na pokładzie czipy Qualcomm
Atheros. Dokładna rozpiska możliwych do wykorzystania opcji w zależności od posiadanego sprzętu
znajduje się na [wiki OpenWRT](https://wiki.openwrt.org/doc/uci/wireless).

## Tryby pracy routera WiFi

Bezprzewodowe karty sieciowe dostępne w routerze WiFi są zwykle dość wszechstronne. Mogą one
pracować w kilku trybach, choć nie zawsze można je ze sobą łączyć. Podstawowym trybem pracy jest
tryb AP, czyli punkt dostępu do sieci bezprzewodowej (Access Point). Zwykle takiej funkcjonalności
będziemy oczekiwać od routera. Pewne tryby pracy, bądź też określona funkcjonalność może wymagać
zainstalowania na routerze określonych pakietów. Weźmy sobie na przykład obsługę protokołu WPS.
Standardowo OpenWRT nie ma oprogramowania, które byłoby w stanie nam zapewnić tę funkcjonalność.
Jeśli jednak dysponujemy routerem o większej ilości pamięci flash, to możemy odinstalować pakiet
`wpad-mini` i doinstalować w jego miejsce `wpad` oraz `hostapd-utils` .

Jako, że głównym trybem pracy czipów WiFi w routerach bezprzewodowych jest tryb AP, to wypadałoby
powiedzieć nieco więcej na jego temat. Konfiguracja punktu dostępu do sieci może być różna.
Najpopularniejszym schematem w warunkach domowych jest WPA2 Personal (WPA2-PSK). W jego przypadku
wszystkie urządzenia w sieci mają jedno i to samo hasło zezwalające na dostęp. Wszystkie karty WiFi
w routerach posiadają ten tryb, w końcu to routery bezprzewodowe. By podłączenie do sieci mogło
nastąpić, wymagane jest podanie dwóch parametrów: identyfikator (nazwa) sieci ( `ssid` ) i hasło (
`key` ). Każda sieć musi posiadać unikalny identyfikator sieci. Unikajmy także stosowania spacji w
nazwie sieci.

Bezpieczeństwo naszej sieci w dużej mierze będzie zależeć od wykorzystywanego hasła oraz stosowanego
szyfrowania. Mamy do wyboru dwa rodzaje haseł. Jednym z nich jest hasło w postaci czystego tekstu i
musi ono posiadać od 8 do 63 znaków. Drugim typem hasła jest ciąg HEX o długości 64 znaków. Nikomu
za bardzo nie chce się wymyślać długich i skomplikowanych haseł ale istnieje narzędzie, które może
wygenerować hasło szesnastkowe w oparciu o nazwę sieci (ssid) oraz o jakąś frazę, dajmy na to
faktyczne hasło. To rozwiązanie świetnie sprawuje się w przypadku systemów z rodziny linux. Poniżej
przykład generowania hasła z wykorzystaniem narzędzia `wpa_passphrase` pod debianem:

    $ wpa_passphrase "Siec" "haslo1haslo2"
    network={
            ssid="Siec"
            #psk="haslo1haslo2"
            psk=6d0e99e5d90b07cfb9e24f2052583be5a65c2dbe6f6a856e1bd26aef3bb52d85
    }

W taki sposób mając słabe hasło, można dość solidnie zabezpieczyć swoją sieć. Przy czym trzeba
pamiętać, że ten ciąg zawsze będzie taki sam, o ile SSID i hasło nie zostaną zmienione. Każdy kto
zna te dwie wartości może wygenerować sobie ten 64 znakowy ciąg.

Jeśli natomiast chodzi o kwestię szyfrowania, która została ustawiona w opcji `encryption` , to
określa ona sposób zabezpieczenia sieci. Wszystkie dostępne wartości, można znaleźć na [wiki
OpenWRT](https://wiki.openwrt.org/doc/uci/wireless#wpa_modes). Trzeba jednak powiedzieć, że mamy
możliwość implementacji różnych typów szyfrów w konfiguracji. Im więcej ich zdefiniujemy tym więcej
urządzeń będziemy mogli podłączyć do sieci. Nie wszystkie urządzenia, szczególnie te starsze,
potrafią obsługiwać obecne standardy szyfrowania. Starsze szyfry może i potrafią odciążyć router
oraz dają możliwość podłączenia większej ilości urządzeń ale jakby nie patrzeć obniżają one
bezpieczeństwo sieci. Jeśli ktoś potrzebuje dobrze zabezpieczyć swoją sieć, to nie ma innego wyboru
poza ustawieniem `psk2+aes` w opcji `encryption` .

## Szerokość kanału dla pasma 2.4 GHz (htmode)

Pasmo 2,4GHz w Polsce jest podzielone na 13 kanałów. Każdy z nich jest przesunięty o 5 MHz. Kanał
pierwszy ma częstotliwość 2412 MHz. Następne mają zwiększane o te 5 MHz, czyli 2417 MHz, 2422 MHz i
tak dalej aż do kanału 13 o częstotliwości 2472 MHz. Wygląda to mniej więcej tak
([źródło](https://en.wikipedia.org/wiki/List_of_WLAN_channels)):

![](/img/2016/05/4.kanaly-wifi-2.4ghz.png#huge)

Większe prędkości transmisji można uzyskać przez zwiększenie zakresu częstotliwości, na której
router operuje. Jest to tzw. szerokość kanału i pod OpenWRT wynosi ona standardowo 20 MHz. Widzimy,
że przy szerokości większej niż 10 MHz będziemy mieć do czynienia z nakładaniem się kilku
sąsiadujących kanałów na siebie, w efekcie będą występować zakłócania transmisji. Niemniej jednak,
przy standardowej szerokości kanałów, mamy do dyspozycji tylko 3 kanały, które nie będą siebie
zakłócać: 1, 6 oraz 11 (ewentualnie 12 lub 13). Ta szerokość kanałów zezwala teoretycznie na
uzyskanie prędkości transmisji 150 mbit/s. Obecnie routery WiFi potrafią obsługiwać kanały o
szerokości 40 MHz, co może generować ogromną ilość zakłóceń zwłaszcza w warunkach miejskich, gdzie
na blokowiskach mamy kilkadziesiąt sieci bezprzewodowych. W ten sposób przy zwiększonej szerokości
kanału zamiast polepszyć sobie transfer, to tylko go pogorszymy, obniżając przy tym jakość sygnału
również innym użytkownikom eteru w naszej okolicy.

Zmianę domyślnych ustawień dotyczących szerokości kanałów przeprowadzamy dodając lub zmieniając te
dwa poniższe parametry do pliku `/etc/config/wireles` w sekcji `config wifi-device` :

    config wifi-device 'radio1'
        ...
        option htmode 'HT20'
        option noscan '0'

W opcji `htmode` można sprecyzować `HT20` dla pojedynczego kanału 20 MHz , lub też `HT40-`/`HT40+` w
przypadku wykorzystywania dwóch kanałów, każdy po 20 MHz. Jeśli planujemy umieścić sieć WiFi na
jednym z kanałów 5-13, to wybieramy `HT40-` . Z kolei jeśli chcemy umiejscowić się na kanale 1-9,
ustawiamy `HT40+` .

Jeśli chodzi jeszcze o obsługę kanałów o szerokości 40 MHz, to nie zawsze będziemy mieć taką
możliwość. Router potrafi przebadać pasmo i oszacować poziom zakłóceń. Niemniej jednak, pasmo 2,4
GHz jest mocno zapchane. Podczas testów, nigdy mi nie udało ustawić sieci WiFi tak, by działała ona
automatycznie na `HT40+` albo `HT40-` . Istnieje jednak opcja `noscan` , która po przestawieniu na
`1`, sprawi, że router nie będzie skanował dostępnego pasma. W ten sposób bez problemu zacznie
korzystać z dwóch kanałów i szerokości 40 MHz. Lepiej jednak nie korzystać z tej opcji, zwłaszcza w
warunkach miejskich, bo dochodzi nam jeszcze ten problem, że część zakłóceń, która pogarsza jakość
sygnału WiFi jest generowana przez pewne sprzęty gospodarstwa domowego oraz przez sprzęt
elektroniczny inny niż bezprzewodowe routery czy karty WiFi, np. mikrofalówki, czy bezprzewodowe
myszy albo klawiatury. Tego typu zakłóceń jest sporo mniej w paśmie 5GHz.

## Szerokość kanału dla pasma 5 GHz (htmode)

Kanały w paśmie 5 GHz są mniej więcej takie same. Każdy z nich również ma szerokość 5 MHz. Różnica
sprowadza się do numeru konkretnego kanału. W Europie, w tym też i w Polsce, mamy możliwość
skorzystania z kanałów od 36 do 64 oraz od 100 do 140. Niemniej jednak szereg kanałów z tych dwóch
zakresów nie jest używanych. W ten sposób kolejny kanał na numer o 4 większy, zatem 36, 40, 44, 100,
104, 108, itd. Kanał 36 ma częstotliwość 5180 MHz, kanał 40 ma 5200 MHz, itd. Poniżej fotka
obrazująca kanały w paśmie 5 GHz
([źródło](https://networkengineering.stackexchange.com/questions/12720/cannot-use-5ghz-band-wi-fi-from-channels-100-140)):

![](/img/2016/05/5.kanaly-wifi-5ghz-.png#huge)

W konfiguracji OpenWRT mamy zatem do wyboru nieco inne wartości. Są to `VHT20` , `VHT40` , `VHT80`
oraz `VHT160` , odpowiednio dla 20 MHz, 40 MHz, 80 MHz i 160 MHz. Te dwa ostatnie są dla standardu
`ac` . Nie określamy przy tym `+` albo `-` , tak jak to ma miejsce w przypadku pasma 2,4 GHz.
Dodatkowe kanały są dobierane automatycznie. By skonfigurować szerokość kanałów, w pliku
`/etc/config/network` dodajemy lub zmieniamy poniższe opcje:

    config wifi-device 'radio1'
        ...
        option htmode 'VHT80'
        option noscan '0'

## Włączenie kanału 12 i 13 w paśmie 2,4 GHz (country)

Domyślne ustawienia routera są określone pod kątem USA. Tam nie ma możliwości używania kanałów o
numerach większych niż 11. Niemniej jednak, w Polsce mamy możliwość korzystania z kanału 12 i 13. By
móc ich używać, musimy określić odpowiedni kod kraju, w którym nadaje router. Robimy to w pliku
`/etc/config/wireles` w sekcji `config wifi-device` :

    config wifi-device 'radio1'
        ...
        option country 'PL'
        option country_ie 'PL'

## Moc transmisyjna (distance, txpower)

Standardowa moc transmisyjna routera WiFi to `20 dBm` . Aktualną moc możemy odczytać, np. z wyjścia
`iwinfo` . Z pewnych przyczyn możemy chcieć zwiększyć lub zmniejszyć moc sygnału. O ile w przypadku
zwiększenia mocy nie wiele zwykle da się zrobić, bo limitowani jesteśmy przez sprzęt, to w przypadku
zmniejszenia mocy mamy większe pole manewru. Przede wszystkim w OpenWRT mamy do dyspozycji dwie
opcje: `distance` (metry) oraz `txpower` (dBm) . Obie z nich określają moc nadajnika, z tym, że
pierwszy z nich dostosowuje ją w zależności od określonej odległości. W przypadku, gdy zamierzamy
sobie dostosować moc transmisyjną, to do pliku `/etc/config/wireless` dopisujemy te poniższe opcje:

    config wifi-device 'radio1'
        ...
    #   option distance '5'
        option txpower '10'

## Ukrycie sieci WiFi (hidden)

Niektóre osoby chciałyby ukryć swoją sieć WiFi przed ludźmi, których mają w swoim zasięgu.
Generalnie nie da się ukryć sieci, bo bez problemu można ją wykryć i nie trzeba być przy tym specem
od technologi bezprzewodowych. W każdym razie, w OpenWRT jest dostępna opcja `hidden` , która może
pomóc nam z ukryciem sieci wyłączając rozgłaszanie identyfikatora SSID. Na wiele to się nie zda ale
jeśli ktoś potrzebuje, to może dodać sobie do pliku `/etc/config/wireless` poniższą linijkę:

    config wifi-iface
        ...
        option hidden '0'

## Filtr adresów MAC (macfilter, maclist)

Innym pseudo zabezpieczeniem jest filtr adresów MAC. Teoretycznie każde urządzenie sieciowe ma mieć
inny adres MAC. Co z tego, skoro taki MAC można bez problemu
[zespoofować](/post/jak-sklonowac-adres-mac-w-openwrt/) czy [w dowolny sposób
wygenerować](/post/jak-przypisac-losowy-adres-mac-interfejsu/) i przypisać
interfejsowi karty sieciowej? Poza tym, węsząc po eterze za za pomocą sniffer'a `wireshark` możemy
bez problemu ustalić adresy MAC każdej stacji, która się łączy do określonego AP:

![](/img/2016/05/1.ukrywanie-sieci-wifi-openwrt-wireshark.png#huge)

W każdym razie OpenWRT daje też możliwość zdefiniowania adresów MAC, które należy
blokować/przepuszczać przy podłączaniu do sieci bezprzewodowej. Jeśli chcemy korzystać z tego
mechanizmu, dopisujemy do pliku `/etc/config/wireless` te poniższe linijki:

    config wifi-iface
        ...
        option macfilter 'disable'
        option maclist '3c:4a:92:00:4c:5b 40:2b:a1:4d:50:7f'

Włączenie filtrowania adresów MAC osiągniemy przez przypisanie opcji `macfilter` wartości `deny` lub
`allow` . W zależności od tego, którą z nich wybierzemy, adresy określone w opcji `maclist` będą
blokowane lub akceptowane. W przypadku, gdy w opcji `macfilter` ustawimy `disable` , filtrowanie
adresów MAC zostanie wyłączone.
