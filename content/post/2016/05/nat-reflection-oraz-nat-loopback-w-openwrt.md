---
author: Morfik
categories:
- OpenWRT
date: "2016-05-09T22:09:47Z"
date_gmt: 2016-05-09 20:09:47 +0200
published: true
status: publish
tags:
- iptables
- chaos-calmer
- sieć
- router
title: NAT Reflection oraz NAT Loopback w OpenWRT
---

[Mechanizm NAT Loopback](https://en.wikipedia.org/wiki/Network_address_translation#NAT_loopback)
nazywany też NAT Reflection lub NAT Hairpinning często jest pomijany przy omawianiu tematyki
firewall'a. Chodzi generalnie o możliwość uzyskiwania dostępu do zasobów w sieci lokalnej po
adresie, który jest na zewnętrznym interfejsie sieciowym routera. W taki sposób mając dwa hosty w
sieci lokalnej, jeden z nich jest w stanie uzyskać dostęp do usług znajdujących się na drugim hoście
przez wykorzystanie zewnętrznego często też publicznego adresu IP. W tym wpisie przybliżymy sobie
zasadę działania tego mechanizmu.

<!--more-->
## Zastosowanie NAT Loopback/NAT Reflection

Jako, że najlepiej jest operować na przykładach, to rozważmy sobie sytuację z życia wziętą. Mamy
sieć lokalną 192.168.1.0/24 podpiętą pod interfejs `br-lan` . Ten interfejs dysponuje adresem
192.168.1.1 . W tej sieci lokalnej mamy dwie maszyny o adresach 192.168.1.100 oraz 192.168.1.234 .
Na interfejsie zewnętrznym routera eth0 mamy adres publiczny 82.160.11.22 . Chcielibyśmy teraz z
maszyny o adresie 192.168.1.100 uzyskać dostęp do serwera www. W standardowej sytuacji nastąpi
połączenie z 192.168.1.100 do 192.168.1.234, bo przecie te dwa hosty są w tej samej sieci. W
pewnych przypadkach jednak chcielibyśmy uzyskać dostęp do tego serwera www wpisując także adres
82.160.11.22 . Problem w tym, że ten adres nie należy do sieci lokalnej i pakiet zostanie przesłany
do bramy domyślnej. Standardowy NAT, który zwykle implementujemy w `iptables` nam nie pomoże w tej
sytuacji. Musimy się zatem nauczyć korzystać z NAT Loopback/NAT Reflection.

NAT Loopback/NAT Reflection w wykonaniu OpenWRT jest automatyczny. Za każdym razem, gdy tworzymy
[sekcję config redirect](https://wiki.openwrt.org/doc/uci/firewall#redirects) w pliku
`/etc/config/firewall` i uwzględnimy w niej opcję `src_dport` , to zostaną stworzone dwie dodatkowe
reguły. Są one dodawane do łańcuchów `zone_lan_prerouting` oraz `zone_lan_postrouting` w tablicy
`nat` w `iptables` . Przyjrzymy się zatem jak wygląda przykładowa sekcja przekierowania:

    config redirect
        option name 'www'
        option src 'wan'
        option dest 'lan'
        option proto 'tcpudp'
        option src_dport '80'
        option dest_ip '192.168.1.234'
        option dest_port '80'
        option target 'DNAT'

W oparciu o ten blok, OpenWRT utworzy te poniższe reguły:

    iptables -t nat -A zone_wan_prerouting -p tcp -m tcp --dport 80 -m comment --comment www -j DNAT --to-destination 192.168.1.234:80

    iptables -t nat -A zone_lan_prerouting -s 192.168.1.0/24 -d 82.160.11.22/32 -p tcp -m tcp --dport 80 -m comment --comment "www (reflection)" -j DNAT --to-destination 192.168.1.234
    iptables -t nat -A zone_lan_postrouting -s 192.168.1.0/24 -d 192.168.1.234/32 -p tcp -m tcp --dport 80 -m comment --comment "www (reflection)" -j SNAT --to-source 192.168.1.1

Pierwsza reguła to standardowy NAT (DNAT) umożliwiający nawiązanie połączenia z serwerem www w sieci
lokalnej hostom znajdującym się w internecie (typowy port forwarding). Generalnie rzecz biorąc nie
interesuje nas ta reguła w tej chwili.

Dwie kolejne reguły odpowiadają za NAT Loopback/NAT Reflection. Operują one na tym samym pakiecie.
Pierwsza z reguł przepisuje nagłówek pakietu przed podjęciem decyzji o routingu (ustawia adres
docelowy). Druga reguła przepisuje nagłówek po tej decyzji (ustawia adres źródłowy). Obie te reguły
są wymagane do poprawnego działania NAT Loopback/NAT Reflection. Jeśli z jakichś przyczyn
zapomnielibyśmy określić ostatnią regułę, to klient z sieci lokalnej będzie miał problemy się
podłączyć. Pakiety nie zostaną zatrzymane na firewall'u ale hosty będą spodziewać się innych
adresów. Poniżej jest przykład ustanawiania połączenia w przypadku braku tej ostatniej reguły:

![](/img/2016/05/1.nat-reflection-nat-loopback-iptables-openwrt.png#huge)

Dla porównania, gdy mamy obie reguły określone, ten sam proces ustanawiania nowego połączenia z
serwerem www wyglądałby tak jak na tej poniższej fotce:

![](/img/2016/05/2.nat-reflection-nat-loopback-iptables-openwrt.png#huge)

Widzimy, że uległ zmianie adres źródłowy, z którego został przesłany pakiet `SYN-ACK` . W tym
pierwszym przypadku, pakiet `SYN-ACK` nie został zaakceptowany właśnie ze względu na inny adres
źródłowy. W efekcie obie strony połączenia sądzą, że pakiety nie dotarły na drugą stronę z
powodzeniem i zaczynają je retransmitować. Oczywiście w takim stanie nigdy nie uda im się
porozumieć. Dlatego właśnie wymagane są obie te powyższe reguły.
