---
author: Morfik
categories:
- OpenWRT
date: "2016-05-03T20:49:50Z"
date_gmt: 2016-05-03 18:49:50 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- switch
title: Podział switch'a na kilka VLAN'ów w OpenWRT
---

Każdy router ma w swoim wyposażeniu [switch][1], czyli przełącznik umożliwiający rozdzielenie
sygnału na kilka portów. W ten sposób przy pomocy przewodu możemy podłączyć więcej komputerów niż
by to miało miejsce w przypadku posiadania tylko jednego gniazda [RJ-45][2]. Na oryginalnym firmware
zwykle nie mamy możliwości dodatkowej konfiguracji switch'a. Z kolei ta co jest, oferuje nam jedynie
jeden port WAN i kilka portów dla podłączenia komputerów w sieci lokalnej. Jeśli jednak
pokusilibyśmy się o wgranie OpenWRT na nasz router, to będziemy mieli możliwość zarządzania
konfiguracją switch'a. W ten sposób będziemy mogli tworzyć dowolne konfiguracje portów (VLAN),
wliczając w to utworzenie, np. kilku portów WAN. Tego typu rozwiązanie może się przydać do bardziej
zaawansowanych konfiguracji sieciowych, np. [failover łącza][3] czy [load balancing][4] w przypadku,
gdy posiadamy kilku ISP i chcemy wykorzystać w pełni łącze oferowane przez te podmioty.

<!--more-->
## Domyślna konfiguracja switch'a

Przede wszystkim należy powiedzieć, że nie każdy switch jest programowalny. Jeśli trafimy na taki,
to OpenWRT nam nie pomoże w jego podziale. Zbierzmy zatem trochę informacji na temat domyślnej
konfiguracji przełącznika, który jest w routerze. W tym celu pomoże nam [polecenie swconfig][5].
Przy jego pomocy najpierw ustalamy nazwę przełącznika:

    # swconfig list
    Found: switch0 - ag71xx-mdio.0

W tym przypadku jest to `switch0` . Dostępne opcje dla tego switch'a możemy uzyskać wydając w
terminalu to poniższe polecenie:

    # swconfig dev switch0 help
    switch0: ag71xx-mdio.0(Atheros AR8327), ports: 7 (cpu @ 0), vlans: 128
         --switch
            Attribute 1 (int): enable_vlan (Enable VLAN mode)
            Attribute 2 (none): reset_mibs (Reset all MIB counters)
            Attribute 3 (int): enable_mirror_rx (Enable mirroring of RX packets)
            Attribute 4 (int): enable_mirror_tx (Enable mirroring of TX packets)
            Attribute 5 (int): mirror_monitor_port (Mirror monitor port)
            Attribute 6 (int): mirror_source_port (Mirror source port)
            Attribute 7 (string): arl_table (Get ARL table)
            Attribute 8 (none): apply (Activate changes in the hardware)
            Attribute 9 (none): reset (Reset the switch)
         --vlan
            Attribute 1 (int): vid (VLAN ID (0-4094))
            Attribute 2 (ports): ports (VLAN port mapping)
         --port
            Attribute 1 (none): reset_mib (Reset single port MIB counters)
            Attribute 2 (string): mib (Get port's MIB counters)
            Attribute 3 (int): enable_eee (Enable EEE PHY sleep mode)
            Attribute 4 (int): pvid (Primary VLAN ID)
            Attribute 5 (string): link (Get port link information)

Wyżej mamy podział na opcje dotyczące konfiguracji samego switch'a ( `switch` ), VLAN'ów ( `vlan` )
oraz portów ( `port` ). Na tych parametrach operujemy przy pomocy `set` oraz `get` , przykładowo:

    # swconfig dev switch0 get enable_vlan
    # swconfig dev switch0 get arl_table
    # swconfig dev switch0 vlan 1 get ports
    # swconfig dev switch0 port 4 set disable 1
    # swconfig dev switch0 set apply

Pełną konfigurację switch'a możemy uzyskać wpisując poniższe polecenie:

    # swconfig dev switch0 show
    Global attributes:
            enable_vlan: 1
            enable_mirror_rx: 0
            enable_mirror_tx: 0
            mirror_monitor_port: 0
            mirror_source_port: 0
            arl_table: address resolution table
    Port 0: MAC c4:6e:1f:95:ef:fe
    Port 1: MAC 00:17:10:89:06:03
    Port 3: MAC 3c:4a:92:00:4c:5b
    Port 6: MAC c4:6e:1f:95:ef:ff

    Port 0:
    ...
            pvid: 1
            link: port:0 link:up speed:1000baseT full-duplex txflow rxflow
    Port 1:
    ...
            pvid: 2
            link: port:1 link:up speed:1000baseT full-duplex auto
    Port 2:
    ...
            pvid: 1
            link: port:2 link:down
    Port 3:
    ...
            pvid: 1
            link: port:3 link:up speed:1000baseT full-duplex txflow rxflow eee100 eee1000 auto
    Port 4:
    ...
            pvid: 1
            link: port:4 link:down
    Port 5:
    ...
            pvid: 1
            link: port:5 link:down
    Port 6:
    ...
            pvid: 2
            link: port:6 link:up speed:1000baseT full-duplex txflow rxflow
    VLAN 1:
            vid: 1
            ports: 0 2 3 4 5
    VLAN 2:
            vid: 2
            ports: 1 6

Ta powyższa konfiguracja switch'a jest domyślną dla routera [TP-LINK Archer C7 v2][6].
Przeanalizujmy zatem ten log. Przede wszystkim został on skrócony dla czytelności. To co zostało
wycięte, to statystyki każdego z portu switch'a. Ten przełącznik jest podzielony na dwa VLAN'y:
`VLAN 1` (vid 1) oraz `VLAN 2` (vid 2). Poniżej mamy fotkę tego routera, która pokazuje jego panel
tylni:

![](/img/2016/05/1.router-switch-vlan-openwrt.jpg#big)

Port oznaczony na niebiesko to VLAN 2 (WAN). Z kolei porty oznaczone na pomarańczowo, to VLAN 1
(LAN). [Na wiki OpenWRT][7] można znaleźć trochę pożytecznych informacji na temat samej
konfiguracji switch'a, jak i jego portów. Tak dla przykładu, powyższy router ma rozpisane porty w
następujący sposób: 0 eth1, 1 WAN, 2 LAN1, 3 LAN2, 4 LAN3, 5 LAN4, 6 eth0. Z tym, że na panelu jest
5 gniazdek, a nie 7. Jeśli się przyjrzymy nieco uważniej, to porty od LAN (te pomarańczowe) mają
dodatkowy numerek `0` , podczas gdy port WAN ma przypisaną `6` . Zarówno `eth0` jak i `eth1`
odpowiadają interfejsom routera, które są widoczne w systemie, np. przy wydawaniu polecenia
`ifconfig` . Zatem wiemy, który z tych dwóch interfejsów będzie obsługiwał połączenie z internetem,
a który sieć lokalną. Z reguły tylko interfejs WAN będzie miał przypisany adres IP, bo interfejs
LAN jest zwykle spięty mostem (bridge) z interfejsami bezprzewodowymi WLAN. W ten sposób wszystkie
interfejsy poza WAN są spięte razem i temu w systemie mamy dodatkowo interfejs wirtualny `br-lan` i
to on dostaje adres IP po stronie LAN.

## Konfiguracja switch'a w OpenWRT

Konfiguracja switch'a w OpenWRT obywa się przez plik `/etc/config/network` . To jakie wpisy tam
zastaniemy zależy w dużej mierze od modelu routera i zastosowanego w nim przełącznika. W tym
przypadku domyślna konfiguracja wygląda następująco:

    config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

    config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0 2 3 4 5'

    config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '1 6'

Blok `config switch` odpowiada za konfigurację podstawowych parametrów przełącznika. Mamy tam opcję
`enable_vlan` , która umożliwia grupowania portów, tak by uzyskać różne interfejsy w systemie i móc
na ich podstawie odpowiednio zarządzać ruchem, który przez ten switch przechodzi. Sekcje `config
switch_vlan` konfigurują VLAN'y. Domyślnie w tym routerze mamy dwa, oznaczone odpowiednio 1 i 2 przy
pomocy opcji `vlan` . Trzeba pamiętać o tym, że na podstawie opcji `vlan` interfejsy sieciowe będą
odpowiednio nazywane w przypadku, gdy będziemy chcieli zrezygnować z domyślnej konfiguracji
switch'a. W przypadku opcji `ports` czasem możemy się spotkać z literką `t` dopisaną do portu, np.
`0t` . Jeśli widzimy ją przy jakimś porcie, to pakiety wychodzące z tego portu będą tagowane
odpowiednim numerem `vid` , który możemy odczytać z `swconfig` . W tym przypadku pakiety nie są
tagowane, bo nie ma takiej potrzeby.

## Konfiguracja VLAN na dwa porty WAN

Załóżmy teraz, że chcemy aby nasz router posiadał dwa porty WAN. Domyślnie jednak posiada tylko
jeden. W takim przypadku trzeba by wyodrębnić jeden port z tych oznaczonych na pomarańczowo (na
obrazku wyżej) i zrobić z niego osobny VLAN. Porty LAN są spięte z portem `0` i mają interfejs
`eth1` . Jeśli podzielimy ten interfejs na dwa VLAN'y (lub więcej), jego nazwa również ulegnie
zmianie na `eth1.x` , gdzie `x` to numer określony w opcji `vlan` w pliku `/etc/config/network` . By
wyodrębnić port i zrobić z niego drugi WAN, przerabiamy nieco konfigurację switch'a do poniższej
postaci:

    config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

    config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0t 3 4 5'

    config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '1 6'

    config switch_vlan
        option device 'switch0'
        option vlan '3'
        option ports '0t 2'

Mamy utworzoną dodatkową sekcję `switch_vlan` o numerze `vid 3` , w której to widnieją dwa porty `0`
i `2` . Ten port `2` został usunięty z VLAN'u 1 . W ten sposób mamy 3 VLAN'y, z których dwa dzielą
jeden interfejs i ruch z tych dwóch VLAN'ów trzeba otagować. Stąd `0t` w obu VLAN'ach. Tak otagowane
pakiety będą miały numerki `1` i `3` , w zależności od tego, z którego VLAN'u pochodzą i tylko ten
VLAN będzie w stanie odebrać takie pakiety. W ten sposób system stworzy dwa dodatkowe interfejsy
wirtualne `eth1.1` oraz `eth1.3` i to one będą brane pod uwagę przy dalszej konfiguracji, zatem
trzeba zmienić nieco ustawienia sieci na routerze.

W tym przypadku jeden z tych VLAN'ów, ten z większa ilością portów, będzie obsługiwał sieć lokalną.
Zatem, musimy przepisać odpowiedni blok w pliku `/etc/config/network` :

    config interface 'lan'
    #   option ifname 'eth1'
        option ifname 'eth1.1'
    ...

Dodatkowo musimy dopisać blok z konfiguracją dla nowego interfejsu `eth1.3` oraz, jeśli
potrzebujemy, blok z konfiguracją dla protokołu IPv^:

    config interface 'wan3'
        option ifname 'eth1.3'
        option proto 'dhcp'

    config interface 'wan3_6'
        option ifname 'eth1.3'
        option proto 'dhcpv6'

Teraz w pliku `/etc/config/dhcp` definiujemy, że nie chcemy by `dnsmasq` interesował się nowym
interfejsem, jako, że jest to interfejs WAN. Dopisujemy tam poniższy blok:

    config dhcp 'wan3'
        option interface 'wan3'
        option ignore '1'

Po zresetowaniu routera, w systemie powinny być widoczne dodatkowe interfejsy:

![](/img/2016/05/2.switch-podzial-vlan-openwrt-wan.png#big)

Jeśli się przyjrzymy adresom MAC jakie widnieją przy interfejsach `eth1`, `eth1.1` oraz `eth1.3` ,
to zauważymy, że są one takie same. Niemniej jednak, w takiej konfiguracji switch'a to niczemu nie
przeszkadza. Naturalnie można zmienić sobie MAC interfejsów, choć nie jest to wymagane. Jeśli jednak
chcemy to zrobić, to w pliku `/etc/config/network` w sekcji dla nowego interfejsu dodajemy poniższą
linijkę:

    config interface 'wan3'
    ...
        option macaddr 'E8:94:F6:68:80:F1'

[Prawidłowy adres MAC możemy wygenerować][8] w oparciu o ten artykuł.

Musimy także skonfigurować firewall. Robimy to w pliku `/etc/config/firewall` i dopisujemy do niego
te dwa poniższe bloki kodu:

    config zone
        option name         wan
        list   network      'wan2'
        option input        REJECT
        option output       ACCEPT
        option forward      REJECT
        option masq         1
        option conntrack    1
        option mtu_fix      1

    config forwarding
        option src      lan
        option dest     wan2


[1]: https://pl.wikipedia.org/wiki/Prze%C5%82%C4%85cznik_sieciowy
[2]: https://pl.wikipedia.org/wiki/RJ-45
[3]: https://en.wikipedia.org/wiki/Failover
[4]: https://pl.wikipedia.org/wiki/R%C3%B3wnowa%C5%BCenie_obci%C4%85%C5%BCenia
[5]: https://wiki.openwrt.org/doc/techref/swconfig
[6]: http://www.tp-link.com.pl/products/details/Archer-C7.html
[7]: https://wiki.openwrt.org/toh/tp-link/archer-c5-c7-wdr7500
[8]: /post/jak-przypisac-losowy-adres-mac-interfejsu/
