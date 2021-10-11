---
author: Morfik
categories:
- OpenWRT
date: "2016-07-07T16:20:17Z"
date_gmt: 2016-07-07 14:20:17 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- sieć
- router
GHissueID: 395
title: Różne adresy LAN i WLAN w OpenWRT (Routed AP)
---

W standardowej konfiguracji OpenWRT, hosty łączące się za pomocą sieci bezprzewodowej jak i tej
przewodowej są spięte razem za pomocą mostka (bridge) i tworzą jedną sieć lokalną. Nie ma w tym nic
dziwnego, bo przecie chcemy, aby komunikacja między wszystkimi hostami w sieci LAN odbywała się bez
większych przeszkód. Przynajmniej takie jest standardowe podejście przy konfiguracji sieci domowej.
Niemniej jednak, w pewnych przypadkach istnieje potrzeba oddzielenia maszyn, które nawiązują
połączenie za pomocą sieci WiFi od tych, które łączą się przewodowo. Generalnie chodzi o różne
adresy, które zostaną przypisane sieciom LAN i WLAN. Rozwiązanie, które zostanie opisane w tym
artykule jest podobne do tworzenia [bezprzewodowej sieci
gościnnej](/post/bezprzewodowa-siec-goscinna-guest-wlan/) (guest WLAN), z tą
różnicą, że w tym przypadku będziemy mieli do czynienia tylko z jedną siecią WiFi (tzw. [Routed
AP](https://wiki.openwrt.org/doc/recipes/routedap)).

<!--more-->
## Jak skonfigurować różne adresy dla LAN i WLAN

By odseparować sieć WLAN od sieci LAN, musimy wyciągnąć interfejs bezprzewodowy routera z mostka
`br-lan` . W tym celu musimy utworzyć w pliku `/etc/config/network` nową sieć i określić jej inną
adresację w stosunku do sieci LAN. Edytujemy zatem w/w plik i dodajemy do niego ten poniższy blok:

    config interface 'wifi'
        option proto    'static'
        option ipaddr   '192.168.20.1'
        option netmask  '255.255.255.0'

Teraz musimy podpiąć tę sieć pod interfejs bezprzewodowy w pliku `/etc/config/wireless` .
Standardowo powinniśmy mieć w tym pliku już określoną konfigurację dla naszej sieci bezprzewodowej.
W takim przypadku wystarczy przepisać w niej ten poniższy parametr:

    config wifi-iface 'wifi'
    ...
        option network 'wifi'
    ...

Musimy jeszcze skonfigurować serwer DHCP, który będzie obsługiwał klientów sieci WiFi. W tym celu
edytujemy plik `/etc/config/dhcp` i dopisujemy do niego poniższą zwrotkę:

    config dhcp 'wifi'
        option interface    'wifi'
        option start        '50'
        option limit        '200'
        option leasetime    '2h'
        option dhcpv6       'server'
        option ra           'server'

Ostatnia rzecz to konfiguracja firewall'a. Edytujemy zatem plik `/etc/config/firewall` . Definiujemy
w nim nową strefę i ustawiamy w niej możliwość nawiązywania połączenia z internetem:

    config zone
        option name       wifi
        list   network    'wifi'
        option input      ACCEPT
        option output     ACCEPT
        option forward    REJECT

    config 'forwarding'
        option 'src'        'wifi'
        option 'dest'       'wan'

Zezwalamy także na ruch na linii LAN <=> WLAN:

    config 'forwarding'
        option 'src'        'lan'
        option 'dest'       'wifi'

    config 'forwarding'
        option 'src'        'wifi'
        option 'dest'       'lan'

I to w zasadzie cała konfiguracja. Restartujemy router i łączymy się po kablu i po WiFi w celu
sprawdzenia poprawności konfiguracji. Jak widzimy poniżej, interfejsowi sieciowemu klienta zostały
przypisane adresy z różnych sieci:

![](/img/2016/07/1.rozne-adresy-lan-wlan-router-openwrt-routed-ap.png#huge)

Zwróćmy też uwagę na fakt, że w tym przypadku nie mamy ustawionych żadnych obostrzeń w stosunku do
klientów sieci WiFi, tak jak to ma miejsce, np. w sieci gościnnej. Hosty tutaj mogą się łączyć
zarówno do routera jak i z maszynami w sieci LAN. Jedyna widoczna dla nas różnica, to inne adresy
IP w zależności od tego czy klient połączy się przewodowo, czy bezprzewodowo.
