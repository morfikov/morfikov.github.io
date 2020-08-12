---
author: Morfik
categories:
- OpenWRT
date: "2016-06-18T22:13:09Z"
date_gmt: 2016-06-18 20:13:09 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- router
- switch
title: Konfiguracja WISP w OpenWRT (tryb STA + AP)
---

Routery wyposażone w firmware OpenWRT mają tę zaletę, że ich bezprzewodowe karty sieciowe można w
miarę dowolnie sobie skonfigurować. Oryginalne oprogramowanie producenta takiego sprzętu zwykle nie
umożliwia nam przełączenia kart routera w inny tryb niż AP (Access Point). W OpenWRT możemy ustawić
praktycznie dowolny tryb, o ile pozwala na to sterownik karty WiFi. W ten sposób możemy aktywować,
np. [tryb MONITOR]({{< baseurl >}}/post/karta-wifi-trybie-monitor-openwrt/). W tym wpisie jednak
będzie nas interesował tryb STA (STATION), czyli postaramy się przełączyć karty WiFi routera w tryb
klienta. Jest to bardzo przydatna opcja w przypadku, gdy mamy do czynienia z [Wireless Internet
Service Provider (WISP)](https://pl.wikipedia.org/wiki/WISP), czyli bezprzewodowymi dostawcami
internetu.

<!--more-->
## Dwa sposoby konfiguracji trybu STA na routerze

Tryb STA na routerze możemy skonfigurować na dwa sposoby. Pierwszy z nich dotyczy sytuacji, gdzie
komputery w sieci lokalnej są podłączone do routera tylko za pomocą przewodu. W tym przypadku
wystarczy nam standardowy tryb STA, który umożliwi podłączenie się routera do sieci bezprzewodowej
naszego WISP ale nie będziemy mogli się podłączyć po WiFi z samym routerem. Drugi sposób z kolei
zakłada skonfigurowanie trybu STA dodatkowo w stosunku do już uprzednio skonfigurowanego trybu AP
na routerze. W tym przypadku, zachowamy możliwość podłączania urządzeń w sieci po WiFi. Poniżej są
opisane obydwa sposoby.

## Konfiguracja standardowego trybu STA

Konfiguracja standardowego trybu STA sprowadza się do edycji dwóch plików: `/etc/config/network`
oraz `/etc/config/wireless` . Przejdźmy zatem do edycji pliku `/etc/config/network` , w którym to
musimy przepisać bloki odpowiadające za interfejsy LAN i WAN do poniższej postaci:

    config interface 'lan'
        option ifname     'eth0.1'
        option force_link '1'
        option type       'bridge'
        option proto      'static'
        option ipaddr     '192.168.2.1'
        option netmask    '255.255.255.0'
        option ip6assign  '60'

    config interface 'wan'
    #   option ifname 'eth0.2'
        option proto 'dhcp'

    config interface 'wan6'
    #   option ifname 'eth0.2'
        option proto 'dhcpv6'

W bloku `config interface 'lan'` określiliśmy standardową konfigurację sieci lokalnej. W przypadku
dwóch ostatnich bloków odpowiadających za konfigurację interfejsu WAN, musimy usunąć lub
wykomentować opcję `ifname` , która przypisuje interfejs przewodowy.

Natomiast jeśli chodzi o plik `/etc/config/wireless` , to przepisujemy go do poniższej postaci:

    config wifi-device 'radio0'
        option type     'mac80211'
        option hwmode   '11g'
        option path     'platform/ar934x_wmac'
        option htmode   'HT20'
        option disabled '0'
        option country  'PL'

    config wifi-iface 'sta'
        option 'device'     'radio0'
        option 'network'    'wan'
        option 'mode'       'sta'
        option 'ssid'       'Ever_Vigilant'
        option 'encryption' 'psk2'
        option 'key'        'haslo'

Kluczowe w bloku `config wifi-iface` jest przestawienie opcji `network` z `lan` na `wan` . Podobnie
w opcji `mode` ustawiamy `sta` zamiast `ap` . Z kolei `ssid` , `encryption` oraz `key` konfigurujemy
zgodnie z instrukcjami otrzymanymi od naszego WISP. Zapisujemy plik i restartujemy router. Po
wystartowaniu, router powinien nawiązać połączenie bezprzewodowe z WISP. Możemy się o tym przekonać
podglądając log systemowy via `logread` . Interfejsowi sieciowemu (w tym przypadku `wlan0` ) powinna
też zostać nadana odpowiednia adresacja. Możemy to sprawdzić przez `ifconfig` lub też `ip` :

![]({{< baseurl >}}/img/2016/06/2.openwrt-wisp-tryb-sta-interfejs-wlan0.png)

## Konfiguracja trybu STA równolegle z trybem AP

Nieco bardziej zaawansowanym mechanizmem jest tryb STA w połączeniu z AP. W tym przypadku, maszyny w
sieci domowej mogą zostać podłączone do routera zarówno za pomocą przewodu jak i bezprzewodowo.
Podobnie jak w przypadku standardowego trybu STA, edytujemy plik `/etc/config/network` i
przepisujemy go do poniższej postaci:

    config interface 'lan'
        option ifname     'eth0.1'
        option force_link '1'
        option type       'bridge'
        option proto      'static'
        option ipaddr     '192.168.2.1'
        option netmask    '255.255.255.0'
        option ip6assign  '60'

    config interface 'wan'
    #   option ifname 'eth0.2'
        option proto  'dhcp'

    config interface 'wan6'
    #   option ifname 'eth0.2'
        option proto  'dhcpv6'

Teraz w pliku `/etc/config/wireless` musimy utworzyć trzy bloki: jeden `config 'wifi-device'` oraz
dwa `config 'wifi-iface'` (po jednym dla trybu STA i AP):

    config wifi-device 'radio0'
        option type 'mac80211'
        option path 'platform/ar934x_wmac'
        option country 'PL'
        option channel '11'
        option hwmode '11g'
        option htmode 'HT20'
        option disabled '0'

    config wifi-iface 'ap'
        option 'device'     'radio0'
        option 'mode'       'ap'
        option 'network'    'lan'
        option 'ssid'       'OpenWRT'
        option 'encryption' 'psk2'
        option 'key'        'haslo'

    config wifi-iface 'sta'
        option 'device'     'radio0'
        option 'network'    'wan'
        option 'mode'       'sta'
        option 'ssid'       'Ever_Vigilant'
        option 'encryption' 'psk2'
        option 'key'        'haslo'

I to w zasadzie cała robota. Po restarcie routera, połączenie z WISP powinno zostać ustanowione, a
my możemy podłączać się do routera w wygodny nam sposób. Zarówno tryb STA jak i AP powinny być
widoczne w `iwinfo` :

![]({{< baseurl >}}/img/2016/06/3.openwrt-wisp-tryb-sta-ap-iwinfo.png)

### Konfiguracja switch'a

Standardowo switch routera jest podzielony na jeden port WAN i cztery porty LAN. W obecnej sytuacji,
tj. gdy korzystamy z trybu STA na routerze i zamierzamy się łączyć jedynie bezprzewodowo z naszym
WISP, to ten jeden port WAN jest nam kompletnie zbędny. W efekcie możemy do routera połączyć
przewodowo maksymalnie cztery urządzenia. Jeśli potrzebujemy wszystkich pięciu gniazdek, to możemy
albo [podzielić sobie inaczej
switch]({{< baseurl >}}/post/podzial-switcha-na-kilka-vlan-w-openwrt/) (jeśli jest taka
możliwość), albo nieco inaczej skonfigurować interfejsy w pliku `/etc/config/network` . W tym
przypadku, router ma przypisany interfejs `eth0.1` jako LAN oraz interfejs `eth0.2` jako WAN. Zatem
plik `/etc/config/network` powinien zostać przepisany do poniższej postaci:

    config interface 'lan'
        option ifname     'eth0.1 eth0.2'
        option force_link '1'
        option type       'bridge'
        option proto      'static'
        option ipaddr     '192.168.2.1'
        option netmask    '255.255.255.0'
        option ip6assign  '60'

W taki sposób po restarcie routera, wirtualny interfejs mostka ( `br-lan` ) powinien uwzględniać dwa
interfejsy sieciowe `eth0.1` oraz `eth0.2` (łącznie 5 gniazdek zamiast standardowych 4). Możemy to
zweryfikować przy pomocy narzędzia `brctl` :

![]({{< baseurl >}}/img/2016/06/4.openwrt-wisp-tryb-sta-mostek.png)
