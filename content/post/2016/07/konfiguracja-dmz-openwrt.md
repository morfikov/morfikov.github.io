---
author: Morfik
categories:
- OpenWRT
date: "2016-07-05T18:05:31Z"
date_gmt: 2016-07-05 16:05:31 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- chaos-calmer
- router
- switch
GHissueID: 396
title: Konfiguracja DMZ w OpenWRT
---

Strefa zdemilitaryzowana ([DMZ, demilitarized
zone](https://pl.wikipedia.org/wiki/Strefa_zdemilitaryzowana_\(informatyka\))) to taki mechanizm,
który ma na celu poprawę bezpieczeństwa usług w sieci lokalnej. Generalnie chodzi o podział sieci
na kilka mniejszych podsieci i oddzielenie ich od siebie fizycznie lub logicznie. [W OpenWRT możemy
wydzielić taką strefę DMZ przy pomocy VLAN'ów](https://wiki.openwrt.org/doc/howto/dmz). Z kolei
odpowiednio skonfigurowany firewall odseparuje nam tę strefę od pozostałych maszyn pracujących w
sieci LAN. W taki sposób nawet w przypadku włamania mającego miejsce w strefie DMZ, maszyny w
pozostałych segmentach sieci będą bezpieczne. W tym wpisie postaramy się skonfigurować strefę
zdemilitaryzowaną na routerze wyposażonym w firmware OpenWRT.

<!--more-->
## Tworzenie VLAN'u pod strefę DMZ

By utworzyć strefę zdemilitaryzowaną musimy na samym początku wydzielić sobie jeden z portów
switch'a na routerze. Dokładny [opis podziału switch'a na VLAN'y znajduje się
tutaj](/post/podzial-switcha-na-kilka-vlan-w-openwrt/). Poniżej zaś jest
przedstawiona konfiguracja switch'a routera [TP-LINK TL-WR1043N/ND
v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html), którą trzeba umieścić lub
dostosować w pliku `/etc/config/network` :

    ...
    config interface 'lan'
        option ifname 'eth1.1'
        option force_link '1'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.3.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

    config interface 'dmz'
        option ifname 'eth1.3'
        option proto 'static'
        option ipaddr '192.168.30.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

    config interface 'wan'
        option ifname 'eth0'
        option proto 'dhcp'

    ...
    config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

    config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0t 2 3 4'

    config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '5 6'

    config switch_vlan
        option device 'switch0'
        option vlan '3'
        option ports '0t 1'

Dodaliśmy tutaj jeden blok `config interface 'dmz'` , którego zadaniem jest skonfigurowanie
adresacji dla strefy zdemilitaryzowanej. Musimy określić tutaj inną sieć w stosunku do tej, która
jest przypisana w bloku `config interface 'lan'` . W obu tych blokach trzeba także odpowiednio
dostosować nazwę fizycznego interfejsu. Ta nazwa zależeć będzie od numeru VLAN'u. W tym przypadku
porty LAN na switchu miały interfejs `eth1` , któremu zostały przypisane dwa VLAN'y o numerach 1 i
3. Dlatego też interfejsy, które musimy uwzględnić, mają nazwy `eth1.1` oraz `eth1.3` .

Po zresetowaniu routera, w systemie powinny być widoczne w/w interfejsy. Interfejs `eth1.1` powinien
być podpięty pod mostek. Natomiast interfejs `eth1.3` powinien mieć określoną wyżej adresację:

![](/img/2016/07/1.dmz-podzial-vlan-openwrt-router.png#huge)

W przypadku, gdy hosty w strefie zdemilitaryzowanej mają być konfigurowane dynamicznie za sprawą
protokołu DHCP, musimy także edytować plik `/etc/config/dhcp` i dopisać w nim tę poniższą zwrotkę:

    config dhcp 'dmz'
        option interface 'dmz'
        option start '100'
        option limit '150'
        option leasetime '12h'
        option dhcpv6 'server'
        option ra 'server'

## Konfiguracja firewall'a na potrzeby DMZ

Mamy zatem wydzieloną strefę DMZ na routerze. Musimy jednak jeszcze napisać szereg reguł dla filtra
`iptables` . [W OpenWRT firewall konfiguruje się przez plik
/etc/config/firewall](/post/filtr-pakietow-sieciowych-w-openwrt-firewall/).
Edytujemy zatem ten plik i dodajemy w nim te poniższe sekcje:

    config 'zone'
        option 'name' 'dmz'
        option 'input' 'REJECT'
        option 'output' 'ACCEPT'
        option 'forward' 'REJECT'
        option 'network' 'dmz'

    config 'forwarding'
        option 'src' 'dmz'
        option 'dest' 'wan'

    config 'forwarding'
        option 'src' 'lan'
        option 'dest' 'dmz'

Pierwszy blok definiuje nową strefę na zaporze i ustawia jej domyślą politykę przepływu pakietów.
Drugi blok zezwala hostom w strefie zdemilitaryzowanej na nawiązywanie połączenia z internetem. Z
kolei trzeci blok zezwala na nawiązywanie połączeń ze strefą DMZ wszystkim hostom w sieci LAN. Hosty
w strefie zdemilitaryzowanej nie mają jednak możliwości nawiązywania nowych połączeń z hostami w
sieci LAN. Podobnie sprawa ma się w przypadku prób połączeń z internetu do strefy DMZ.

By hosty w strefie zdemilitaryzowanej mogły bez problemy korzystać z sieci, musimy do pliku
`/etc/config/firewall` dodać jeszcze dwie reguły:

    config 'rule'
        option 'src' 'dmz'
        option 'proto' 'tcpudp'
        option 'dest_port' '53'
        option 'target' 'ACCEPT'

    config 'rule'
        option 'src' 'dmz'
        option 'proto' 'udp'
        option 'dest_port' '67'
        option 'target' 'ACCEPT'

Pierwsza z nich przepuszcza ruch protokołu DNS, druga zaś DHCP.

## Konfiguracja usług w strefie DMZ

No i w sumie całą strefę zdemilitaryzowaną mamy już przygotowaną. Teraz przydałoby się podpiąć jakąś
maszynę pod wydzielony port na routerze i skonfigurować na niej przykładową usługę, dajmy na to
serwer www. Oczywiście ten serwer jest widoczny póki co jedynie dla hostów w sieci LAN. Jeśli
jednak, ktoś spoza sieci chciałby się połączyć z serwerem w DMZ, to niestety nie uzyska on
połączenia. Możemy tej osobie jednak umożliwić połączenie dodając kolejną regułę do firewall'a w
pliku `/etc/config/firewall` :

    config 'redirect'
        option 'name' 'http'
        option 'src' 'wan'
        option 'dest' 'dmz'
        option 'proto' 'tcp'
        option 'src_dport' '80'
        option 'dest_ip' '192.168.30.234'
        option 'target' 'DNAT'
        option enabled '1'

Ta powyższa reguła ma na celu przekierować zapytania przesyłane na router na port 80 na serwer www,
który nasłuchuje zapytań na hoście w strefie zdemilitaryzowanej.
