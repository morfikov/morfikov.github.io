---
author: Morfik
categories:
- OpenWRT
date: "2016-05-03T02:08:31Z"
date_gmt: 2016-05-03 00:08:31 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
GHissueID: 496
title: Konfiguracja interfejsów sieciowych w OpenWRT
---

Routery, które posiadamy w naszych domach, znane są z tego, że mają szereg interfejsów sieciowych.
Taki przeciętny router jest wyposażony w
[switch](https://pl.wikipedia.org/wiki/Prze%C5%82%C4%85cznik_sieciowy) z 5 portami
[RJ-45](https://pl.wikipedia.org/wiki/RJ-45). Zwykle jest on też [wirtualnie podzielony
(VLAN)](https://pl.wikipedia.org/wiki/Wirtualna_sie%C4%87_lokalna) na kilka interfejsów, standardowo
LAN i WAN. Do tego z reguły dochodzą jeszcze interfejsy bezprzewodowe WLAN na pasmo 2.5GHz i 5GHz.
Jakby tego było mało, to mamy jeszcze wirtualny interfejs mostka, który spina ze sobą lokalne
interfejsy switch'a z interfejsami bezprzewodowymi tworząc w ten sposób jeden interfejs, przez który
pakiety wydostają się z naszej sieci i lecą dalej w świat przez interfejs WAN. Jest to trochę
skomplikowane, dlatego też w tym wpisie przyjrzymy się całej tej konfiguracji interfejsów sieciowych
w OpenWRT.

<!--more-->
## Jakie interfejsy sieciowe ma router

Na sam początek ustalmy to jakimi interfejsami dysponuje nasze urządzenie. Logujemy się zatem na
router i w terminalu wpisujemy polecenie `ifconfig` . Jak sama nazwa wskazuje, powinno ono zwrócić
listę aktywnych interfejsów sieciowych:

![konfiguracja-interfejsow-sieciowych-archer-c7-openwrt](/img/2016/05/1.konfiguracja-interfejsow-sieciowych-archer-c7-openwrt.png#big)

Powyżej mamy szereg interfejsów fizycznych, czyli tych, które zostały poprawnie rozpoznane przez
system operacyjny routera. Do nich zaliczają się `eth0` , `eth1` , `wlan0` oraz `wlan1` . Mamy tam
także dwa interfejsy wirtualne `lo` oraz `br-lan` . Te dwa ostatnie są dynamicznie tworzone przez
OpenWRT. Do poprawnej pracy, każdy z tych interfejsów wymaga odpowiedniej konfiguracji. Do tego celu
oddelegowane są różne skrypty, które w oparciu o plik `/etc/config/network` ustawiają wszystkie z
powyższych interfejsów.

W pliku `/etc/config/network` mamy sprecyzowanych kilka bloków, które zostaną poniżej opisane. W
domyślnej konfiguracji OpenWRT może brakować pewnych linijek. Dlatego też jeśli czegoś nie
posiadamy, to wystarczy zwyczajnie dany parametr dopisać. Oczywiście w przypadku, gdy jest on nam
potrzeby. Zwykle domyślna konfiguracja jest adresowana do większości użytkowników i raczej nie
będziemy musieli nic zmieniać czy dopisywać. Trzeba też wziąć ewentualną poprawkę na nazwy
interfejsów sieciowych, bo te mogą nie być jednakowe we wszystkich modelach routerów. Ten artykuł
jest pisany w oparciu o konfigurację routera [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html), który od wydania Chaos Calmer działa
pod OpenWRT bardzo przyzwoicie.

## Interfejs pętli zwrotnej (lo)

Pierwszy blok w pliku `/etc/config/network` odpowiada za konfigurację pętli zwrotnej. Jest to
interfejs, na którym działają zwykle wewnętrzne usługi routera, z którymi nikt spoza tego urządzenia
nie jest w stanie się połączyć. Jest to taka jego wirtualna sieć. Ten interfejs zawsze jest
oznaczany jako `lo` , a jego konfiguracja pod OpenWRT wygląda mniej więcej tak jak poniżej
przedstawiono:

    config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

Nazwa interfejsu jest określana przez opcję `ifname` . Lepiej jest nie używać nazw dłuższych niż 15
znaków. Może to prowadzić do różnego rodzaju błędów w konfiguracji. Dalej mamy opcję `proto`
odpowiadającą za protokół konfiguracji tego interfejsu. Pełną listę obsługiwanych protokołów można
znaleźć na [wiki OpenWRT](https://wiki.openwrt.org/doc/uci/network). W przypadku interfejsu `lo` ,
zawsze konfiguruje się go statycznie. W zależności od wybranego protokołu, możemy definiować szereg
opcji specyficznych dla danego protokołu. Jako, że tutaj mamy protokół statyczny, to określić musimy
dwie opcje: `ipaddr` (adres IP) oraz `netmask` (maskę podsieci). W ten sposób interfejs pętli
zwrotnej działa w sieci 127.0.0.0/8 i ma adres 127.0.0.1 . Ten interfejs musi być skonfigurowany, w
przeciwnym razie lokalne usługi przestaną działać lub też zaczną działać nieprawidłowo.

## Globalne parametry interfejsów

Nieco pod blokiem dotyczącym interfejsu `lo` mamy sekcję `config globals 'globals'` . W niej są
umieszczane parametry niezależne od interfejsu. Standardowo po instalacji OpenWRT, w tej sekcji jest
umieszczony tylko jeden parametr. Jest on odpowiedzialny za [konfigurację prefiksu lokalnego adresu
protokołu IPv6](https://en.wikipedia.org/wiki/Unique_local_address). Poniżej ta sekcja:

    config globals 'globals'
        option ula_prefix 'fd42:156b:900b::/48'

## Interfejs mostka (br-lan)

Dalej w pliku `/etc/config/network` mamy blok odpowiadający za konfigurację wirtualnego interfejsu
sieciowego. Jest to [mostek
(bridge)](https://pl.wikipedia.org/wiki/Most_%28sie%C4%87_komputerowa%29), który spina ze sobą
szereg fizycznych interfejsów routera. W ten sposób mamy do skonfigurowania tylko jeden interfejs,
podczas gdy router może operować na szeregu fizycznych interfejsów. W tym przypadku są to `wlan0`,
`wlan1` oraz `eth1` (lokalne porty switch'a) . Usługi, które będziemy chcieli udostępnić w sieci
lokalnej, np. serwer FTP czy www, będą na tym interfejsie operować. Poniżej znajduje się
konfiguracja mostka w routerze:

    config interface 'lan'
        option ifname 'eth1'
        option force_link '1'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
    #   option broadcast '192.168.1.255'
        option ip6assign '60'

Widzimy tutaj, że opcja `type` tego interfejsu została ustawiona na `bridge` . W opcji `ifname`
definiujemy wszystkie interfejsy fizyczne, które mają być podpięte pod mostek. [Z racji pewnych
problemów z interfejsami
bezprzewodowymi](https://forum.openwrt.org/viewtopic.php?pid=203784#p203784), nie określa się ich
statycznie w pliku `/etc/config/network` . Niemniej jednak, one również będą dodawane do tego
mostka, z tym, że już za sprawą skryptu `/lib/netifd/hostapd.sh` . Po odrzuceniu interfejsów `wlan0`
i `wlan1` zostaje nam tylko interfejs `eth1` . Ten interfejs, podobnie jak w przypadku `lo` , będzie
miał ustawioną statyczną adresację. Adres IP to 192.168.1.1 z maską 255.255.255.0, co tworzy sieć
192.168.1.0/24 . Opcjonalnie można także zdefiniować `broadcast` , który standardowo jest ustawiany
na ostatni adres sieci, czyli 192.168.1.255 . Dzięki temu adresowi dany host w sieci może wysłać
pakiety do wszystkich znajdujących się w niej maszyn, np. w przypadku poszukiwania [serwera
DHCP](/post/dhcp-dns-czyli-konfiguracja-sieci-w-openwrt/). Jako, że ustawiliśmy
statyczny protokół konfiguracji tego interfejsu, musimy także ustawić opcję `force_link` . Odpowiada
ona za przypisanie do tego interfejsu adresu IP, maski oraz opcjonalnie bramy sieciowej nawet w
przypadku, gdy połączenie nie jest aktywne. Dalej mamy opcję `ip6assign` określającą długość
prefiksu dla sieci protokołu IPv6.

## Interfejs WAN (eth0)

Może i router posiada szereg interfejsów przewodowych i bezprzewodowych, za pomocą których możemy
się do niego podłączyć od strony LAN, to w przypadku WAN mamy do dyspozycji standardowo tylko jeden
interfejs. Jest on wydzielony ze switch'a i w tym przypadku jest to `eth0` . Konfiguruje się go
podobnie co w dwóch powyższych przypadkach, tj. przez odpowiednią sekcję w pliku
`/etc/config/network` , przykładowo:

    config interface 'wan'
        option ifname 'eth0'
        option proto 'dhcp'
    #   option macaddr 'E8:94:F6:68:79:F1'
    #   option hostname 'the-mountain'
        option peerdns '0'
        option dns '208.67.220.220 208.67.222.222'

Jako, że za pomocą tego interfejsu będzie się odbywać łączność ze światem, to jego konfiguracja jest
zwykle ustawiona na dynamiczną przez protokół DHCP. Można oczywiście ustawić sobie statyczny adres,
tak jak to miało w przypadku interfejsu `br-lan` . Niemniej jednak, trzeba będzie ręcznie
dostosowywać sobie konfigurację połączenia z ISP jeśli ulegnie ona zmianie wliczając w to `ipaddr` ,
`netmask` `gateway` i opcjonalnie `broadcast` . W `ifname` określamy fizyczny interfejs switch'a,
tym razem po stronie WAN. Jeśli potrzebujemy [sklonować adres
MAC](/post/jak-sklonowac-adres-mac-w-openwrt/), to w opcji `macaddr` wpisujemy
adres poprzedniej maszyny, która widnieje w bazie ISP. Opcja `hostname` sprawi, że klient `udhcpc`
wyśle do serwera DHCP nazwę hosta określoną w tym parametrze. Zwykle nie jest wysyłana żadna nazwa.
Za pomocą opcji `peerdns` oraz `dns` jesteśmy w stanie nadpisać serwery DNS otrzymane w lease od
serwera DHCP naszego ISP. Po zresetowaniu routera, adresy określone tutaj pojawią się w pliku
`/tmp/resolv.conf.auto` :

    # cat /tmp/resolv.conf.auto
    # Interface wan
    nameserver 208.67.220.220
    nameserver 208.67.222.222

Jeśli chodzi o serwery DNS dla hostów znajdujących się w sieci lokalnej, to takim serwerem jest sam
router. Wszystkie zapytania DNS z sieci wędrują pod adres określony w pliku `/tmp/resolv.conf` . Po
ich otrzymaniu przez router, te zapytania są przesyłane do upstream'owych serwerów DNS, tych
określonych w pliku `/tmp/resolv.conf.auto` . Możemy oczywiście ustawić, by zapytania z sieci szły
bezpośrednio do serwerów google czy OpenDNS. Z tym, że wtedy stracimy możliwość buforowania zapytań,
co trochę spowalnia rozwiązywanie domen na adresy IP.

## IPv6 na interfejsie WAN (eth0)

OpenWRT wspiera protokół IPv6. Jeśli tylko nasz router wspiera tę nową adresację, to jesteśmy w
stanie na porcie WAN skonfigurować klienta DHCP, który pobierze stosowną konfigurację. Niestety
niewielu providerów internetowych zapewnia wsparcie dla IPv6, w efekcie poniższy blok na nic nie
wpływa:

    config interface 'wan6'
        option ifname    'eth0'
        option proto     'dhcpv6'

Niemniej jednak, jeśli jesteśmy w posiadaniu zewnętrznego adresu IPv4, to możemy pokusić się o
[stworzenie tunelu 6in4](/post/konfiguracja-tunelu-6in4-w-openwrt-ipv6/).

## Konfiguracja switch'a (VLAN)

Ostatnie trzy bloki w pliku `/etc/config/network` dotyczą podziału switch'a na VLAN'y. Te sekcje są
różne w zależności od posiadanego routera. Trzeba też zaznaczyć, że nie każdy switch jest
programowalny i nie zawsze będziemy mieć opcję jego podziału wedle upodobania, np. jeśli
chcielibyśmy wydzielić drugi port WAN. Poniżej przykład konfiguracji:

    config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

    config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0t 2 3 4 5'

    config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '0t 1'

Konfiguracja switch'a wykracza poza ramy tego artykuły i nie będę poruszał tutaj tego tematu.
Niemniej jednak, jest on bardzo ciekawy i na tyle obszerny, że został opisany w artykule dotyczącym
[podziału switch'a na VLAN'y](/post/podzial-switcha-na-kilka-vlan-w-openwrt/).
