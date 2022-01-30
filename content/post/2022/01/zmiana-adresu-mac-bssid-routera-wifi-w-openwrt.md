---
author: Morfik
categories:
- OpenWRT
date:    2022-01-30 20:23:00 +0100
lastmod: 2022-01-30 20:23:00 +0100
published: true
status: publish
tags:
- wifi
- prywatność
- router
- bssid
- mac
GHissueID: 586
title: Zmiana adresu MAC (BSSID) routera WiFi w OpenWRT
---

Pisząc ostatnio artykuł o tym [jak uchronić nasz prywatny punkt dostępu do sieci WiFi przed
zmapowaniem][4] i umieszczeniem jego fizycznej lokalizacji w serwisach pokroju Wigle.net czy też w
usługach Google/Mozilla, poruszony został temat ewentualnego usuwania wpisów z takich baz danych AP
za sprawą dopisywania do nazwy sieci (ESSID) sufiksu `_nomap` . Nie wszystkie organizacje/firmy
respektują naszą prywatność i raczej też nie będą tej końcówki w nazwie sieci honorować. Jako, że
jednym z dwóch parametrów, na podstawie których takie mapowanie punktów dostępowych się odbywa,
jest adres MAC interfejsu bezprzewodowego danego AP (BSSID), to możemy od czasu do czasu pokusić
się o zmianę tego adresu. Jest to mniej więcej to samo rozwiązanie, co zastosowane przy [klonowaniu
adresu MAC dla portu WAN routera][1], które z reguły użytkownicy wykorzystują do obchodzenia
zabezpieczeń tych bardziej wrednych ISP. Jako, że temat zmiany BSSID trochę się różni od zmiany
adresu MAC dla portu WAN, to w niniejszym artykule postanowiłem opisać jak w routerze z wgranym
firmware OpenWRT podejść do tego zadania.

<!--more-->
## Czy wato bawić się w zmianę adresu MAC (BSSID) routera WiFi

Bazy danych przechowujące informacje na temat fizycznej lokalizacji punktów dostępowych WiFi łączy
w zasadzie jedna wspólna cecha. Jest nią brak usuwania starych wpisów, co z kolei przekłada się na
znacznie zwiększoną objętość takiej bazy, a przez to też i jej atrakcyjność w oczach potencjalnych
klientów tej firmy, co te informacje zbiera. Niemnie jednak, w takiej nieczyszczonej od lat (albo i
dekad) bazie danych może istnieć cała masa nieaktualnych AP, które już od dawna zostały zezłomowane
przez swoich użytkowników. Istnieje zatem spore prawdopodobieństwo, że nazwa sieci WiFi, którą
sobie obraliśmy, będzie figurować w kilku różnych fizycznych lokalizacjach, nie dając przy tym
jednoznacznej informacji o położeniu naszego routera WiFi. Niemniej jednak, osoby, które z jakiegoś
powodu zainteresowane by były naszym konkretnym AP musiałyby również pozyskać adres MAC
bezprzewodowego interfejsu tego routera. Jeśli im się tę informację uda pozyskać, to bez większego
problemu będą te osoby w stanie ustalić położenie naszego AP eliminując te pozycje, które mają inny
BSSID. Niektóre bazy danych operują jedynie na adresach MAC (BSSID) lub nazwach sieci (ESSID) w
połączeniu z BSSID. Zatem by nieco zatrzeć ślady możemy pokusić się o zmianę adresu MAC
bezprzewodowego interfejsu naszego routera.

## Zmiana BSSID w OpenWRT (plik /etc/config/wireless)

By zmienić adres MAC/BSSID punktu dostępowego WiFi na routerze, który ma wgrany firmware OpenWRT,
musimy edytować plik `/etc/config/wireless` i dopisać [parametr macaddr][2] w sekcji z `config
wifi-iface` , przykładowo:

    config wifi-iface 'default_radio1'
        ...
        option macaddr 'aa:aa:aa:aa:aa:aa'
        ...

Po zapisaniu konfiguracji, restartujemy radio w routerze wpisując w terminalu polecenie `wifi` . Po
chwili sieć WiFi powinna zostać ponownie udostępniona przez router. Wystarczy teraz przy pomocy
polecenia `ifconfig` albo `ip` sprawdzić czy adres MAC na interesującym nas interfejsie
bezprzewodowym uległ zmianie, przykładowo:

    root@RedViper:~# ip addr show
    ...
    48: wifi2g: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-lan state UP group default qlen 1000
        link/ether aa:aa:aa:aa:aa:aa brd ff:ff:ff:ff:ff:ff
    ....

I jak możemy zauważyć, adres MAC interfejsu serwującego WiFi 2,4GHz uległ zmianie na ten, który
sobie określiliśmy wyżej w pliku `/etc/config/wireless` .

##  Weryfikacja ustawień BSSID ze strony klienta

By mieć pewność, że podczas skanowania za punktami dostępowymi, ten ustawiony przez nas adres MAC
zostanie zgłoszony, a nie ten, który to urządzenie ma przypisane fabrycznie, odpalamy terminal na
pierwszej lepszej stacji klienckiej (w tym przypadku jest to mój laptop z zainstalowanym Debianem)
i przy pomocy polecenia `wpa_cli` przeprowadzamy skanowanie eteru w poszukiwaniu AP:

    # wpa_cli scan && sleep 5 && wpa_cli scan_results
    Selected interface 'wlan0'
    OK
    Selected interface 'wlan0'
    bssid / frequency / signal level / flags / ssid
    ...
    aa:aa:aa:aa:aa:aa       2412    -34     [WPA2-PSK-CCMP][ESS][UTF-8]     Ever_Vigilant
    ...

Jak widzimy, na liście pojawił się mój AP ze zmienionym BSSID. Jeśli teraz ktoś by dokonał
skanowania okolicy w celu dodania takiego punktu dostępu do bazy danych, to w bazie zostanie
uwzględniony ten widoczny wyżej MAC, a nie ten fabryczny urządzenia.

## Parę słów o losowości adresów MAC/BSSID

Oczywiście taki BSSID, który ustawiliśmy sobie wyżej, jest dość podejrzany. Dlatego też by wtopić
się w tłum  lepiej jest [wygenerować sobie MAC][3] możliwie nieodbiegający od przyjętych standardów.
Możemy naturalnie ręcznie sobie wygenerować taki MAC (tak jak to zostało opisane w podlinkowanym
wyżej artykule) albo też zaprzęgnąć do pomocy [narzędzie macchanger][5] i wygenerowany przez niego
MAC podać routerowi.

## Podsumowanie

Poprzez określenie ESSID/BSSID w bazach danych punktów dostępowych WiFi jesteśmy w stanie namierzyć
fizyczną lokalizację osób, które nieświadomie lub lekkomyślnie podały te informacje gdzieś w
internecie. Pozytywna wiadomość jest taka, że zarówno nazwę sieci, jak i adres adres MAC routera
WiFi, można zmienić praktycznie w dowolnym momencie pracy tego urządzenia, przynajmniej gdy w grę
wchodzi firmware OpenWRT. Regularna zmiana tych parametrów sieci raczej może przynieść więcej szkód
niż pożytku za sprawą problemów, które niesie ze sobą rekonfiguracja wszystkich klientów takiej
sieci. Jeśli nie zamierzamy zmieniać fizycznej lokalizacji, np. nie przeprowadzamy się do innego
miasta, to zmiana BSSID (i ewentualnie ESSID) mija się z celem i może nam dać jedynie fałszywe
poczucie prywatności na zasadzie takiej, że skoro teraz używamy innej sieci, to nikt na podstawie
poprzednich parametrów (tych ujawnionych publicznie) nie będzie w stanie ustalić naszego położenia.
Dlatego też jeśli zależy nam na prywatności, to lepszym rozwiązaniem jest niepublikowanie tych
informacji w internecie.


[1]: /post/jak-sklonowac-adres-mac-w-openwrt/
[2]: https://openwrt.org/docs/guide-user/network/wifi/basic
[3]: /post/jak-przypisac-losowy-adres-mac-interfejsu/
[4]: /post/jak-ukryc-nazwe-sieci-wifi-essid-przed-wigle-net-google-mozilla/
[5]: https://github.com/alobbs/macchanger
