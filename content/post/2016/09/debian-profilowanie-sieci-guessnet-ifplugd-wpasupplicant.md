---
author: Morfik
categories:
- Linux
date: "2016-09-05T13:01:08Z"
date_gmt: 2016-09-05 11:01:08 +0200
published: true
status: publish
tags:
- wifi
- debian
- openbox
- sieć
title: 'Debian: Profilowanie sieci z guessnet, ifplugd i wpasupplicant'
---

Kilka dni temu na [forum dug.net.pl](https://forum.dug.net.pl/viewtopic.php?id=28903) pojawił się
ciekawy wątek dotyczący problemu skonfigurowania profilowanych sieci. Chodzi o to, że praktycznie
każdy z nas jest po części w jakiś sposób mobilny i zabiera laptopa ze sobą w dziwne miejsca. Sieci
w tych lokalizacjach mogą cechować się różnym poziomem bezpieczeństwa. Dlatego też zamiast korzystać
z jednej konfiguracji sieci na linux'ie, można stworzyć szereg profili i w oparciu o nie dostosować
sobie połączenie sieciowe. W tym artykule spróbujemy zaimplementować takie rozwiązanie na Debianie
wyposażonym w menadżer okien Openbox. W skrócie stworzymy automat, który będzie nam działał w
oparciu o pakiety guessnet, [ifplugd](http://0pointer.de/lennart/projects/ifplugd/) oraz
[wpasupplicant](https://w1.fi/wpa_supplicant/). Cała konfiguracja zaś sprowadzać się będzie jedynie
do edycji plików `/etc/network/interfaces` oraz `/etc/wpa_supplicant/wpa_supplicant.conf` .

Niniejszy artykuł został nieco przerobiony po fazach eksperymentów. Przede wszystkim, zrezygnowałem
z zaprzęgania `guessnet` do rozpoznawania sieci WiFi i aplikowania roamingu. Zamiast tego zostały
wykorzystane natywne rozwiązania roamingowe oferowane przez `wpa_supplicant` . Zaowocowało to
uproszczeniem całej konfiguracji, co przełożyło się na wyeliminowanie pewnych błędów.

<!--more-->
## Potrzebne narzędzia: guessnet, ifplugd i wpasupplicant

Konfiguracja sieci musi zostać podzielona na interfejs przewodowy `eth0` i bezprzewodowy `wlan0` .
Te dwa interfejsy będą ze sobą współpracować ale inne oprogramowanie się będzie nimi opiekować.
Dlatego też każdy z tych interfejsów trzeba sobie osobno skonfigurować.

Przede wszystkim, musimy w systemie doinstalować pakiet `guessnet` . To za jego sprawą będzie się
dokonywać profilowanie sieci przewodowej. Dzięki skryptom zawartym w tym pakiecie, nasz system
będzie w stanie automatycznie wykryć środowisko w zależności od tego, gdzie się podepniemy do
sieci.

Przyda się nam także pakiet `ifplugd` . Będzie on miał za zadanie monitorowania interfejsu
przewodowego pod kątem podpięcia/odłączenia przewodu do/od portu ethernet. W taki sposób nie
będziemy musieli ręcznie podnosić i kłaść interfejsu `eth0` . Niemniej jednak, wymagana jest
poprawna [konfiguracja demona
ifplugd]({{< baseurl >}}/post/dynamiczna-konfiguracja-sieci-ifplugd/).

Do ogarniania interfejsów bezprzewodowych w linux'ie służy pakiet `wpasupplicant` . Powinien być on
już standardowo zainstalowany w naszym systemie. Jeśli nie jest, to koniecznie doinstalujmy ten
pakiet. Za jego sprawą będziemy w stanie [skonfigurować sobie roaming sieci
bezprzewodowych]({{< baseurl >}}/post/jak-skonfigurowac-roaming-wifi-wpa_supplicant-linux/).

Z instalacją tych powyższych narzędzi nie powinno być raczej większego problemu. Zatem przejdźmy od
razu do konfiguracji wszystkich niezbędnych elementów naszego mechanizmu profilującego sieć.

## Profilowanie interfejsu przewodowego w laptopie

Jak nadmieniłem we wstępie, spora część konfiguracji będzie sprowadzać się do edycji pliku
`/etc/network/interfaces` . Musimy tylko umieścić w nim odpowiednie sekcje, które skonfigurują nam
interfejs przewodowy `eth0` . Poniżej jest przykład konfiguracji:

    mapping eth0
        script guessnet-ifupdown
        map home-wire work-wire no_link no_network
        map default: no_network
        map timeout: 5
    #   map verbose: true
    #   map debug: true

    iface home-wire inet dhcp
        test peer address 192.168.1.1 mac ec:08:6b:84:15:d5
        pre-up ifdown --force wlan1
        post-down ifup wlan1

    iface work-wire inet dhcp
        test peer address 192.168.1.1 mac ec:08:6b:84:15:ff
        pre-up ifdown --force wlan1
        post-down ifup wlan1

    iface no_link inet manual
        test missing-cable
        pre-up echo "There's no link present, please plug a cable in."
        pre-up false

    iface no_network inet manual
        pre-up echo "There's no safe networks around, so I can't connect to one."
        pre-up false

Warto zaznaczyć, że nie korzystamy z opcji `auto` w przypadku interfejsu przewodowego. Odpowiadałaby
ona co prawda za automatyczną konfigurację tego interfejsu na starcie komputera ale pamiętajmy, że w
naszym przypadku to demon `ifplugd` monitoruje ten interfejs. Natomiast to czy interfejs sieciowy
zostanie podniesiony zależy od podłączenia przewodu do portu ethernet.

Zwrotka z `mapping eth0` odnosi się do `guessnet` , bo wykorzystywany jest skrypt
`guessnet-ifupdown` . W niej zaś mamy szereg parametrów `map` , które konfigurują zachowanie
skryptu. Przede wszystkim, mamy tam `map home-wire work-wire no_link no_network` . Wszystkie te
nazwy odnoszą się do wirtualnych interfejsów, które są określone niżej w pliku
`/etc/network/interfaces` . Nazwy oczywiście są dowolne ale muszą pasować do nazw wirtualnych
interfejsów.

W `map default` określamy interfejs, który ma zostać użyty w przypadku niespełnienia polityki przy
wykrywaniu środowiska sieci. Innymi słowy, chodzi o to co zrobić w przypadku podłączenia się do
nieznanej/niezaufanej nam sieci. Powyższa konfiguracja wypisze jedynie komunikat o braku zaufanych
sieci i interfejs `eth0` nie zostanie skonfigurowany. Oczywiście można dowolnie sobie zaprogramować
tę akcję i ogranicza nas chyba tylko nasza wyobraźnia.

Dalej mamy `map timeout` , który przyjmuje wartość liczbową wyrażoną w sekundach. Jest to czas,
przez który skrypt `guessnet-ifupdown` będzie próbował ustalić czy sieć, do której się
podłączyliśmy, jest nam znana. W przypadku przekroczenia tego czasu, zostanie wykonana akcja
domyślna określona w `map default` .

Z kolei zaś `map verbose` oraz `map debug` są w stanie wypisać obszerny log w przypadku, gdy coś nie
działa nam jak trzeba. Zwykle nie będziemy musieli używać tych opcji.

Dalej w pliku mamy szereg zwrotek `iface` , które konfigurują poszczególne interfejsy wirtualne.
Może być ich dowolna ilość, z tym, że `guessnet` operować będzie jedynie na tych, które zostały
określone w parametrze `map` .

Interfejsy wirtualne możemy konfigurować tak jak zwykłe interfejsy sieciowe. Interfejs `home-wire` i
`work-wire` odnoszą się do dwóch znanych nam sieci, w domu i w firmie. W obu tych przypadkach
konfiguracja ma być uzyskiwana za pośrednictwem protokołu DHCP. Niemniej jednak, zanim ta
konfiguracja zostanie pobrana, połączenie jest testowane na obecność określonego hosta w sieci.
Sprawdzany jest jego adres IP oraz MAC. W przypadku dopasowania takiego hosta, interfejs zostanie
skonfigurowany. W każdym innym przypadku nie zostaniemy podłączeni do sieci. Tutaj testujemy tylko
jednego hosta. Niemniej jednak, linijek z `test peer` może być dowolna ilość. W takiej sytuacji, by
się podłączyć do sieci, wszystkie te hosty muszą być w niej obecne.

Interfejs `no_link` ma za zadanie wykryć obecność przewodu w porcie ethernet. W przypadku jego
braku, interfejsu `eth0` nie będzie podlegał konfiguracji, co zaoszczędzi czas.

Z kolei zaś interfejs `no_network` jest domyślnym i jego zadaniem jest niedopuszczenie do
konfiguracji sieci w przypadku środowiska niezaufanego, tj. innego niż dom czy firma.

## Profilowanie interfejsu bezprzewodowego w laptopie

Nasz system powinien już być w stanie dobrać profil sieciowy ale tylko w przypadku interfejsu
przewodowego. Jeśli chodzi o laptopy, to raczej rzadko będziemy wykorzystywać ten interfejs. Musimy
zatem jeszcze skonfigurować interfejs WiFi, tak by i on podlegał profilowaniu. Edytujemy jeszcze raz
plik `/etc/network/interfaces` i dodajemy do niego poniższe zwrotki:

    auto wlan0
    iface wlan0 inet manual
        wpa-driver nl80211
        wpa-debug-level -1
        wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
        wpa-roam-default-iface default
        pre-up sh -c "sleep 1"

    iface home-wifi inet dhcp
    iface work-wifi inet dhcp

    iface default inet dhcp

Tryb `auto` w przypadku tego interfejsu musimy ustawić, by wyeliminować problem z brakiem połączenia
po starcie systemu w przypadku, gdy przewód ethernet nie jest podłączony do karty sieciowej laptopa.
Opcję `wpa-driver` dostosowujemy sobie w zależności od posiadanej karty WiFi. Spora część nowszych
kart korzysta z `nl80211` . W przypadku starszych kart trzeba ustawić `wext` . Parametr `wpa-roam`
określa plik konfiguracyjny z parametrami sieci bezprzewodowych dla narzędzia `wpa_supplicant` .
Ostatnia rzecz to linijka z `pre-up` . Ma ona za zadanie poprawić problem z niemożliwością uzyskania
adresacji podczas fazy boot systemu.

Przechodzimy teraz do konfiguracji roamingu WiFi. Jak widzimy wyżej, mamy skonfigurowane trzy
wirtualne interfejsy: `home-wifi` , `work-wifi` oraz `default` . Ten ostatni został określony jako
domyślny w `wpa-roam-default-iface` . Mamy także osobny wirtualny interfejs dla sieci domowej i
firmowej.

W tym przypadku nie korzystamy z `guessnet` , bo nie ma to większego sensu. Parametry sieci WiFi
jednoznacznie identyfikują daną sieć i w oparciu o ESSID, BSSID oraz hasło możemy bez problemu
stwierdzić czy zostaliśmy podłączeni do tej sieci, do której powinniśmy. Niemniej jednak, jeśli ktoś
jest ciekaw [rozwiązania roamingowego opartego o guessnet, to może je znaleźć
tutaj](http://manual.aptosid.com/pl/inet-setup-pl.htm).

Edytujemy teraz plik `/etc/wpa_supplicant/wpa_supplicant.conf` . Tutaj z kolei możemy nadać
priorytet sieciom lub też włączyć lub wyłączyć łączenie się do określonych sieci bezprzewodowych. By
powiązać interfejsy wirtualne z pliku `/etc/network/interfaces` z konkretnymi sieciami WiFi, musimy
posłużyć się parametrem `id_str` , poniżej przykład:

    ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev
    eapol_version=2
    ap_scan=1
    fast_reauth=1
    country=PL

    network={
        id_str="home-wifi"
        priority=20
        ssid="Home"
        bssid=ec:08:6b:8d:36:f2
        psk=12345678
        disabled=0
    }

    network={
        id_str="work-wifi"
        priority=89
        ssid="Work"
        bssid=e8:94:f6:c4:00:2b
        psk="87654321"
        disabled=0
    }

Zakładając, że siedzibę firmy mamy gdzieś w pobliżu naszego domu, to powyższa konfiguracja będzie
preferować firmową sieć bezprzewodową. Oba bloki są włączone i nasz laptop przy próbie podłączenia
będzie szukał obu tych sieci. Hasło i nazwa sieci identyfikują konkretne sieci WiFi, a przy
określeniu `bssid` możemy także precyzować z jakim AP mamy zamiar się połączyć. Kluczowa sprawa to
poprawne ustawienie `id_str` .

## Konfiguracja akcji po podłączeniu do profilowanej sieci

Mając tak wyprofilowane interfejsy sieciowe w laptopie, możemy teraz w pliku
`/etc/network/interfaces` w zwrotkach `iface` dodać sobie specyficzne akcje w zależności od sieci,
do której laptop zostanie podłączony. Robimy to przez określenie opcji `pre-up` , `pre-down` ,
`post-up` oraz `post-down` . Poniżej przykład dla interfejsu przewodowego:

    iface home-wire inet dhcp
        test peer address 192.168.1.1 mac C4:6E:1F:95:EF:FE
        post-up /bin/systemctl stop firewall.service

    iface work-wire inet dhcp
        test peer address 192.168.0.1 mac E8:94:F6:C4:00:2C
        post-up /bin/systemctl start firewall.service

Jak widać, nie jest to nic zaawansowanego. Tutaj firewall jest aktywowany po podłączeniu do sieci
firmowej i dezaktywowany w warunkach domowych. W przypadku sieci WiFi postępujemy podobnie, tylko
nie musimy określać już linijek z `test peer` .

## Test profilowanego połączenia sieciowego

Wypadałoby także na sam koniec przetestować jak działa taki automat profilująco-przełączający
interfejsy sieciowe. Wystarczy wpiąć się do sieci i zobaczyć co zostanie nam zwrócone w logu
systemowym. W moim przypadku, po podłączeniu do jednej sieci z wykorzystaniem przewodu mam poniższy
log:

    kernel: r8169 0000:02:00.0 eth0: link up
    ifplugd(eth0)[1730]: Link beat detected.
    ifplugd(eth0)[1730]: Executing '/etc/ifplugd/ifplugd.action eth0 up'.
    ...
    kernel: wlan1: deauthenticating from ec:08:6b:84:15:d5 by local choice (Reason: 3=DEAUTH_LEAVING)
    ifplugd(eth0)[1730]: client: OK
    wpa_action[28380]: WPA_IFACE=wlan1 WPA_ACTION=DISCONNECTED
    wpa_action[28383]: WPA_ID=0 WPA_ID_STR=home-wifi WPA_CTRL_DIR=/run/wpa_supplicant
    wpa_action[28386]: ifdown wlan1
    ...
    ifplugd(eth0)[1730]: client: guessnet: Will test for link beat.  If absent, will return no_link
    ...
    ifplugd(eth0)[1730]: client: guessnet: Got ARP reply from 192.168.1.1 ec:08:6b:84:15:d5
    ifplugd(eth0)[1730]: client: guessnet: ARP reply from 192.168.1.1 ec:08:6b:84:15:d5 matches
    ...
    ifplugd(eth0)[1730]: client: guessnet: Keeping candidate home-wire
    ...
    ifplugd(eth0)[1730]: client: bound to 192.168.1.150 -- renewal in 32872 seconds.
    systemd[1]: Stopping firewall...
    systemd[1]: Stopped firewall.

Log nieco skrócony dla czytelności. W każdym razie widzimy, że przewód został podłączony do portu
ethernet. Po chwili interfejs `wlan0` został położony i `guessnet` wysłał zapytanie ARP, na które
otrzymał odpowiedź. Pasuje ona do jednego ze zdefiniowanych hostów w pliku `/etc/network/interfaces`
. System rozpoznał, że jest to sieć domowa i skonfigurował połączenie z wykorzystaniem protokołu
DHCP. Po otrzymaniu adresacji zostały także dostosowane reguły firewall'a.

No to odłączmy teraz przewód od portu ethernet i zobaczmy co nam system w logu wypisze:

    kernel: r8169 0000:02:00.0 eth0: link down
    ifplugd(eth0)[1730]: Link beat lost.
    ifplugd(eth0)[1730]: Executing '/etc/ifplugd/ifplugd.action eth0 down'.
    ...
    kernel: wlan1: authenticated
    kernel: wlan1: associated
    ...
    wpa_action[32385]: ifup wlan1=work-wifi
    ...
    dhclient[32404]: bound to 192.168.2.146 -- renewal in 37710 seconds.
    ...
    systemd[1]: Starting firewall...
    systemd[1]: Started firewall.
    ..
    wpa_action[32489]: bssid=ec:08:6b:84:15:d5
    wpa_action[32489]: freq=2462
    wpa_action[32489]: ssid=Ever_Vigilant
    wpa_action[32489]: id=0
    wpa_action[32489]: id_str=work-wifi
    wpa_action[32489]: mode=station
    wpa_action[32489]: pairwise_cipher=CCMP
    wpa_action[32489]: group_cipher=CCMP
    wpa_action[32489]: key_mgmt=WPA2-PSK
    wpa_action[32489]: wpa_state=COMPLETED
    wpa_action[32489]: ip_address=192.168.2.146
    wpa_action[32489]: p2p_device_address=e8:94:f6:1e:15:e9
    wpa_action[32489]: address=e8:94:f6:1e:15:e9
    wpa_action[32489]: uuid=91269bbc-a5ed-4106-8f3d-a2489399b950

Widzimy, że system wykrył odłączenie przewodu i położył interfejs `eth0` . Następnie
`wpa_supplicant` przeskanował eter w poszukiwaniu zdefiniowanych w pliku
`/etc/wpa_supplicant/wpa_supplicant.conf` sieci WiFi. Nastąpiło uwierzytelnienie i powiązanie do
jednej z sieci. Widzimy, że sieć, do której zostaliśmy podłączeni korzysta z wirtualnego interfejsu
`wlan1=work-wifi` , zatem zostaliśmy podłączeni do sieci firmowej. Po chwili została pobrana
adresacja i założony firewall. Ostatnie komunikaty dotyczą konfiguracji sieci, do której zostaliśmy
podłączeni.

## Ten sam adres MAC

Adresy MAC interfejsu przewodowego i bezprzewodowego różnią się. W efekcie przy podłączaniu do sieci
będziemy otrzymywać dwa różne adresy IP. Próbowałem jakoś ogarnąć tę kwestię ale wygląda na to, że
przy roamingu nie można zmieniać adresu MAC na interfejsie sieciowym. Nie wiem dlaczego próba
ustawiania MAC zapętla demona roamingowego, który po podłączeniu się do sieci za chwilę się rozłącza
i tak w nieskończoność.

W warunkach domowych, gdzie zwykle mamy dostęp do konfiguracji głównego routera, możemy powiązać ten
sam adres IP z kilkoma adresami MAC, co wyeliminuje problem z różnymi adresami IP. W warunkach
firmowych trzeba będzie raczej korzystać z różnych adresów, chyba, że jakoś uda mi się rozwiązać
problem z zapętlaniem demona roamingowego.
