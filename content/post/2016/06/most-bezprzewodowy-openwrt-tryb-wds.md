---
author: Morfik
categories:
- OpenWRT
date: "2016-06-19T16:32:10Z"
date_gmt: 2016-06-19 14:32:10 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- router
title: Most bezprzewodowy w OpenWRT (tryb WDS)
---

[Tryb WDS (Wireless Distribution
System)](https://pl.wikipedia.org/wiki/Wireless_Distribution_System) umożliwia stworzenie mostu
bezprzewodowego, w którym to nadrzędny AP (Access Point) przekazuje pakiety do klientów WDS. Ci z
kolei przesyłają te pakiety dalej do podrzędnych AP. Z reguły routery posiadają funkcję WDS i po jej
aktywowaniu, ten drugi AP będzie wzmacniał i rozsyłał otrzymany sygnał dalej w granicach swojego
zasięgu. Tryb WDS nie jest jednak standardem i producenci firmware oraz sterowników implementują go
inaczej, co przekłada się na problemy z kompatybilnością. Jeśli chcemy korzystać z opcji WDS w
OpenWRT, to najlepiej posiadać kilka takich samych urządzeń i dobrze jest mieć na nich wgrane to
samo oprogramowanie. W tym wpisie zaprojektujemy sobie taki most bezprzewodowy w oparciu o dwa
routery firmy TP-LINK: [TL-WDR3600 v1](http://www.tp-link.com.pl/products/details/TL-WDR3600.html)
oraz [Archer C7 v2](http://www.tp-link.com.pl/products/details/Archer-C7.html), na których jest
wgrany firmware OpenWRT Chaos Calmer 15.05.1 .

<!--more-->
## Sprzęt pod WDS

Mając do dyspozycji kilka routerów, musimy pierw ustalić, który z nich będzie głównym AP. W tym
przypadku router Archer C7 v2 będzie robił właśnie za ten główny punkt dostępu. Z kolei TL-WDR3600
v1 będzie robił za klienta WDS. Trzeba pamiętać o tym, że na obydwu routerach trzeba skonfigurować
tę samą sieć WiFi.

Karty WiFi tych dwóch w/w routerów działają w oparciu o sterownik mac80211. To jest ważna
informacja, bo nie wszystkie sterowniki mają zaimplementowaną obsługę trybu WDS. Dlatego też
wszystkie poniższe informacje dotyczą głównie routerów, które mają czipy bezprzewodowe działające na
sterowniku mac80211 . Typ sterownika wykorzystywanego przez nasze routery możemy sprawdzić
zaglądając do pliku `/etc/config/wireless` . Tam w bloku `config wifi-device` powinien być
określony typ:

    config wifi-device 'radio0'
        option type 'mac80211'
        ...

Poniżej jest fotka obrazująca wykorzystywane routery, ich adresację początkową i zainstalowane na
nich oprogramowanie:

![](/img/2016/06/1.openwrt-router-tryb-wds-konfiguracja.png#huge)

Konfiguracja bezprzewodowego mostu sprowadza się do edycji 3 plików konfiguracyjnych na każdym z
routerów. Są to `/etc/config/network` , `/etc/config/dhcp` oraz `/etc/config/wireless` . To w nich
będziemy dokonywać wszystkich zmian.

## Konfiguracja WDS na głównym routerze

Na pierwszy ogień idzie router główny, czyli Archer C7 v2 . Logujemy się na niego i edytujemy plik
`/etc/config/network` . W nim będzie nas interesował jedynie blok od interfejsu LAN. Adres tego
interfejsu musi być określony statycznie:

    config interface 'lan'
        option ifname 'eth1'
        option force_link '1'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

Następnie przechodzimy do edycji pliku `/etc/config/dhcp` . Tutaj ustawiamy konfigurację serwera
DHCP dla interfejsu LAN. Powinna ona wyglądać następująco:

    config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option limit '150'
        option leasetime '12h'
        option dhcpv6 'server'
        option ra 'server'

Pozostał nam do edycji już tylko plik `/etc/config/wireless` . Do konfiguracji naszej sieci WiFi,
którą powinniśmy mieć już określoną, musimy dopisać parametr `wds` . Wszystko inne zostawiamy w
spokoju:

    config wifi-device 'radio1'
        option type 'mac80211'
        option channel '11'
        option hwmode '11g'
        option path 'platform/qca955x_wmac'
        option htmode 'HT20'
        option country 'PL'
        option disabled '0'

    config wifi-iface
        option device 'radio1'
        option network 'lan'
        option mode 'ap'
        option ssid 'Ever_Vigilant'
        option encryption 'psk2+aes'
        option key 'haslo'
        option disabled '0'
        option hidden '0'
        option wds '1'

## Konfiguracja klienta WDS

W przypadku routera, który ma robić za klient WDS będzie trochę więcej roboty. Zaczynamy podobnie
jak w przypadku głównego routera, tj. od edycji pliku `/etc/config/network` . Musimy ustawić tutaj
adres statyczny w bloku odpowiedzialnym za interfejs LAN. Dodatkowo, ten adres musi być z tej samej
klasy adresowej co w przypadku głównego routera (ta sama sieć). W ten sposób będziemy mieli
możliwość wbicia po SSH zarówno na główny router jak i na tego klienta WDS. To tak na wypadek,
gdyby były jakieś problemy. Skoro router główny miał adres 192.168.1.1 , to tutaj ustawmy
192.168.1.2 :

    config interface 'lan'
        option ifname 'eth0.1'
        option force_link '1'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.1.2'
        option netmask '255.255.255.0'
        option ip6assign '60'

Następnie w pliku `/etc/config/dhcp` musimy wyłączyć serwer DHCP na interfejsie LAN. Adresacja dla
hostów w sieci będzie przydzielana z głównego routera. Nie musimy usuwać żadnych opcji z bloku
`config dhcp 'lan'` . Wystarczy, że dodamy do niego opcję `ignore` , a cały blok zostanie
zignorowany w konfiguracji:

    config dhcp 'lan'
        ...
        option ignore '1'

Można także wyłączyć z autostartu routera skrypt `dnsmasq` . Choć nie jest to wymagane:

    # /etc/init.d/dnsmasq disable

Tak czy inaczej, efekt będzie taki sam, czyli adresacja dla komputerów w sieci lokalnej będzie
przydzielana z głównego routera, a nie z klienta WDS.

W pliku `/etc/config/wireless` musimy odpowiednio skonfigurować sieć WiFi. Przede wszystkim,
konfiguracja radia musi być taka sama co w przypadku głównego routera. Sprawa dotyczy tego samego
kanału ( `channel` ), jego szerokości ( `htmode` ) oraz trybu pracy ( `hwmode` ). Jeśli chodzi zaś o
konfigurację samej sieci, to nazwa sieci ( `ssid` ), hasło do niej ( `key`) oraz protokół
( `encryption` ) również muszą do siebie pasować:

    config wifi-device 'radio0'
        option type 'mac80211'
        option channel '11'
        option hwmode '11g'
        option path 'platform/ar934x_wmac'
        option htmode 'HT20'
        option country 'PL'
        option disabled '0'

    config wifi-iface 'ap'
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option ssid 'Ever_Vigilant'
        option encryption 'psk2+aes'
        option key 'haslo'
        option disabled '0'
        option hidden '0'

Dodatkowo musimy stworzyć nowy blok `config wifi-iface` z konfiguracją WDS:

    config wifi-iface 'wds'
        option device 'radio0'
        option network 'lan'
        option mode 'sta'
        option ssid 'Ever_Vigilant'
        option encryption 'psk2+aes'
        option key 'haslo'
        option disabled '0'
        option hidden '0'
        option wds '1'

Blok, który zawiera opcję `wds` musi mieć opcję `mode` ustawioną na `sta` . To jest właśnie druga
końcówka bezprzewodowego mostu. Natomiast blok mający określoną opcję `mode` jako `ap` jest
konfiguracją zwykłego punktu dostępowego. Trzeba tutaj pamiętać, że nie wszystkie sterowniki czipów
WiFi potrafią obsługiwać jednocześnie kilka i do tego różnych trybów pracy. We wszystkich trzech
blokach `config wifi-iface` (jeden na routerze głównym oraz dwa na routerze WDS) konfiguracja sieci
WiFi musi być taka sama. W ten sposób będziemy mieli dwa AP z tą samą siecią i most między nimi. Z
kolei komputery będą się łączyć do tego AP, z którego będą otrzymywać mocniejszy sygnał, chyba, że
sprecyzujemy im inny BSSID w konfiguracji połączenia.

## Testowanie połączenia WDS

Po zresetowaniu obu routerów, przy skanowaniu sieci powinniśmy zobaczyć dwa punkty dostępu
charakteryzujące się tą samą nazwą sieci oraz jej parametrami. Jedyna różnica tkwi w BSSID. Poniżej
jest przykład skanowania pasma:

![](/img/2016/06/2.openwrt-linux-skanowanie-sieci-wds.png#huge)

Przyszedł czas przetestować czy ten most bezprzewodowy w ogóle działa. Najlepiej jest rozstawić oba
routery w sporej odległości od siebie, ewentualnie też w różnych pomieszczeniach. Następnie
przechodzimy tam, gdzie mamy router, który robi za klient WDS i próbujemy podłączyć się do sieci.
Powinniśmy podłączyć się przez tego klienta WDS jako, że jego sygnał jest sporo silniejszy w
porównaniu do sygnału docierającego od głównego routera. Jeśli ten test przebiegnie prawidłowo, to
możemy zasymulować utratę sygnału przez wyłączenie karty WiFi w kliencie WDS. Po tym jak to
uczynimy, stracimy połączenie z siecią ale tylko na chwilę. Po kilku sekundach nasz komputer
powinien nawiązać połączenie ponownie, tym razem z głównym routerem. Poniżej jest zalogowana właśnie
ta sytuacja:

![](/img/2016/06/3.openwrt-linux-przelaczenie-wds.png#huge)

O godzinie 15:25:40 nastąpiło połączenie z klientem WDS. Mój laptop został uwierzytelniony i
powiązany z AP. Otrzymał też adresację z głównego routera przez protokół DHCP. Z kolei o 15:25:49
nastąpiła utrata sygnału z klienta WDS. Mógł paść router lub też to mój laptop mógł stracić sygnał z
bliżej nieznanych powodów. Praktycznie w tej samej chwili nastąpiło podłączenie do głównego routera.
Jeśli przyjrzymy się adresowi MAC, to dostrzeżemy, że różni się od tego poprzedniego.
