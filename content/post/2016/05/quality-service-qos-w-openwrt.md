---
author: Morfik
categories:
- OpenWRT
date: "2016-05-19T15:07:54Z"
date_gmt: 2016-05-19 13:07:54 +0200
published: true
status: publish
tags:
- iptables
- chaos-calmer
- router
- qos
title: Quality of Service (QoS) w OpenWRT
---

Wszyscy spotkaliśmy się z sytuacją, w której z bliżej nieokreślonego powodu nie mogliśmy przeglądać
stron w internecie. Niby połączenie jest, mamy też dobrej klasy ISP ale net nam muli. W olbrzymiej
części przypadków winą można obarczyć sieć [Peer to Peer
(P2P)](https://pl.wikipedia.org/wiki/Peer-to-peer). Przy nieodpowiedniej konfiguracji klientów tej
sieci może dojść do nawiązywania całej masy połączeń w ułamku chwili. W ten sposób nawet jeśli nic
aktualnie nie wysyłamy lub nie pobieramy, to i tak te połączenia same w sobie zapychają łącze, no bo
przecież nawet puste pakiety w protokołach TCP/UDP zawierają nagłówki, a te już swoje ważą. Gdy sami
korzystamy z łącza, to taka sytuacja nie stanowi większego problemu, bo wystarczy odpalić klienta
torrent'a w wolnym czasie. Natomiast w przypadku, gdy zachodzi potrzeba wejścia na jakiś serwis www,
to możemy zwyczajnie tego klienta wyłączyć. Nie jest to jednak wygodne. Jak zatem wyeliminować
problemy związane z siecią P2P? Jeśli na routerze mamy wgrany firmware OpenWRT, to możemy
zaimplementować na nim [mechanizm QoS (Qality of
Service)](https://pl.wikipedia.org/wiki/Quality_of_service). W ten sposób możemy nadać usługom
priorytety. W niniejszym wpisie postaramy się wdrożyć takie rozwiązanie w oparciu o narzędzia
`iptables` oraz `tc` .

<!--more-->
## Interfejsy IMQ oraz IFB

W OpenWRT nie ma prostego sposobu na wdrożenie QoS'a. Wcześniej były do dyspozycji [interfejsy IMQ
(Intermediate Queueing Device)](https://github.com/imq/linuximq/) ale te zostały zastąpione przez
interfejsy IFB (Intermediate Functional Block). Różnica między nimi sprowadza się do lepszego
kształtowania ruchu docierającego do routera w przypadku interfejsów IMQ. Natomiast jeśli chodzi o
interfejsy IFB, to praktycznie wcale nie mamy możliwości kontrolować ruchu przychodzącego. Możemy w
prawdzie ograniczyć ilość pakietów, który router będzie w stanie przetworzyć w danej jednostce czasu
ale nie mamy kontroli nad tym, który z pakietów znajdujących się w kolejce dotrze jako pierwszy. W
ten sposób mogą powstać znaczne opóźnienia, które czasami mogą poważnie wpłynąć na jakość usługi. Z
tym, że ta sytuacja dotyczy tylko ruchu przychodzącego na interfejs IFB.

W routerach zwykle mamy kilka interfejsów fizycznych. Ograniczmy się w tym momencie do dwóch. Jeden
WAN i jeden LAN. Na każdym z tych interfejsów jesteśmy w stanie kształtować ruch wychodzący bez
zaprzęgania do tego interfejsów IMQ czy IFB. Chodzi generalnie o to, że ruch pakietów jest
przesyłany między interfejsami WAN i LAN. Kolejki pakietów są zamienione w przypadku tych
interfejsów. Czyli to co jest na wyjściu LAN jest jednocześnie na wejściu WAN i odwrotnie. W ten
sposób możemy ograniczyć zarówno download i upload stacji roboczych w sieci domowej.

## Instalacja potrzebnego oprogramowania

Nie obejdzie się bez instalacji dodatkowych pakietów. Trzeba jednak zaznaczyć, że w tym przypadku
nie będziemy korzystać ani z interfejsów IMQ, ani z interfejsów IFB. Dlatego też zestaw pakietów
będzie nieco inny w porównaniu do tutoriali, które traktują o kontroli ruchu w OpenWRT. Musimy
doinstalować pakiety `ip` oraz `tc` . Dodatkowo, potrzebne będą nam pewne moduły dla kernela oraz
`iptables` . Wszystkie te rzeczy są wypisane poniżej:

    # opkg update
    # opkg install ip \
    tc \
    iptables-mod-ipopt \
    iptables-mod-conntrack-extra \
    iptables-mod-extra \
    kmod-sched-connmark \
    kmod-sched \
    kmod-ipt-u32 \

Być może nie wszystkie moduły zostaną załadowane po instalacji powyższych pakietów. Dla pewności,
dobrze jest w tym momencie uruchomić router ponownie.

## Skrypt startowy

Musimy także utworzyć [skrypt startowy]({{< baseurl >}}/post/skrypty-startowe-init-w-openwrt/).
Taki skrypt będzie miał za zadanie tworzyć odpowiednie kolejki i [oznaczać stosownie pakiety przy
pomocy target MARK]({{< baseurl >}}/post/target-mark-w-iptables/). Na podstawie nałożonych
markerów, pakiety będą kierowane do swoich kolejek. W tym przypadku szkielet skryptu init będzie
wyglądał następująco:

    #!/bin/sh /etc/rc.common

    START=61
    EXTRA_COMMANDS="status"


    . /lib/functions/network.sh
    network_get_physdev if_wan wan
    network_get_physdev if_lan lan

    nic_link="1000mbit"
    isp_uplink="980kbit"
    isp_downlink="13500kbit"
    home_uplink="999mbit"
    home_downlink="986mbit"

    status() {
    }

    start() {
    }

    stop() {
    }

Jako, że OpenWRT dysponuje własnym skryptem startowym konfigurującym filtr pakietów `iptables` , to
ten nasz skrypt init musi się uruchomić po nim. Dlatego w `START` ostawiliśmy odpowiednią sekwencję
startu. W pliku `/lib/functions/network.sh` są ciekawe funkcje, które pomogą nam w ustaleniu nazw
interfejsów WAN i LAN. Te nazwy zostaną zapisane w zmiennych `if_wan` oraz `if_lan` . Mamy także
określone parametry łącza. Zmienna `nic_link` odpowiada za przepustowość portów. W przypadku routera
[TP-Link TL-WDR3600 v1](http://www.tp-link.com.pl/products/details/TL-WDR3600.html) mamy gigabitowy
switch, zatem ustawiamy `1000mbit` . W zmiennych `isp_*` określamy parametry łącza od naszego ISP.
Muszą one być trochę niższe w stosunku do deklarowanej przez providera prędkości. Chodzi o to, że
kolejki na interfejsach tworzą się tam, gdzie jest najwęższe ogniwo przy przesyle informacji. Jeśli
określimy zbyt dużą wartość, to kolejki będą się tworzyć na interfejsach ISP i to on będzie
kształtował ruch. Te dwa parametry trzeba dostosować sobie indywidualnie w drodze eksperymentów.
Jeśli chodzi zaś o zmienne `home_*` , to muszą one przybrać wartość, która dopełni wartość
określoną w zmiennej `nic_link` , czyli:

    isp_uplink + home_uplink =~ nic_link
    isp_downlink + home_downlink =~ nic_link

Teraz musimy odpowiednio uzupełnić pozostałą część skryptu.

W bloku `status() {}` znajdują się jedynie polecenia zwracające statystyki:

    status() {
        echo "# qdiscs $if_wan #"
        tc -s -d qdisc show dev $if_wan

        echo "# qdiscs $if_lan #"
        tc -s -d qdisc show dev $if_lan

        echo "# class $if_wan #"
        tc -s -d class show dev $if_wan

        echo "# class $if_lan #"
        tc -s -d class show dev $if_lan

        echo "# filter $if_wan #"
        tc -s -d filter show dev $if_wan parent 2:0

        echo "# filter $if_lan #"
        tc -s -d filter show dev $if_lan parent 1:0
    }

W bloku `start() {}` na samym początku umieszczamy reguły oznaczające ruch:

    start() {
        iptables -t mangle -N qos_traffic
        iptables -t mangle -N qos_lan
        iptables -t mangle -N qos_wan

        iptables -t mangle -I PREROUTING -j qos_traffic
        iptables -t mangle -I POSTROUTING -j qos_traffic

        iptables -t mangle -A qos_traffic -j CONNMARK --restore-mark --nfmask 0x00ff --ctmask 0x00ff
        iptables -t mangle -A qos_traffic -m mark ! --mark 0x0/0x00ff -j RETURN
        iptables -t mangle -A qos_traffic -o $if_wan -m mark --mark 0x0/0x00ff -j qos_wan
        iptables -t mangle -A qos_traffic -o $if_lan -m mark --mark 0x0/0x00ff -j qos_lan
        iptables -t mangle -A qos_traffic -j CONNMARK --save-mark --nfmask 0x00ff --ctmask 0x00ff

        iptables -t mangle -A qos_lan -s 192.168.1.0/24 -d 192.168.1.0/24 -j MARK --set-xmark 0x000a/0x00ff -m comment --comment "lan"
        iptables -t mangle -A qos_lan -p icmp -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "ICMP"
        iptables -t mangle -A qos_lan -p udp -m multiport --sports 53 -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "DNS"
        iptables -t mangle -A qos_lan -p tcp -m multiport --sports 80,443,20,21 -j MARK --set-xmark 0x0002/0x00ff -m comment --comment "http,ftp"

        iptables -t mangle -A qos_wan -p tcp -m multiport --dports 80,443,20,21 -j MARK --set-xmark 0x0002/0x00ff  -m comment --comment "http,ftp"
        iptables -t mangle -A qos_wan -p icmp -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "ICMP"
        iptables -t mangle -A qos_wan -p tcp -m multiport --dports 993,995,465,143,110,25 -j MARK --set-xmark 0x0003/0x00ff  -m comment --comment "mail"
        iptables -t mangle -A qos_wan -p udp -m multiport --dports 53 -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "DNS"
        iptables -t mangle -A qos_wan -p udp -m multiport --dports 37,123 -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "time"
        iptables -t mangle -A qos_wan -p tcp -m multiport --dports 37,123 -j MARK --set-xmark 0x0001/0x00ff -m comment --comment "time"
        iptables -t mangle -A qos_wan -p tcp -m multiport --dports 5222,5223 -j MARK --set-xmark 0x0002/0x00ff -m comment --comment "jabber"
    ...

Dalej w bloku `start() {}` tworzymy kolejki dla tych wyżej oznaczonych pakietów i kierujemy do nich
ruch:

    ...
        tc qdisc add dev $if_lan parent root handle 2:0 htb default 600 r2q 100
            tc class add dev $if_lan parent 2:0 classid 2:1 htb rate $nic_link ceil $nic_link quantum 15140
                tc class add dev $if_lan parent 2:1 classid 2:10 htb rate $isp_downlink ceil $isp_downlink
                    tc class add dev $if_lan parent 2:10 classid 2:100 htb rate 1500kbit ceil $isp_downlink prio 10
                    tc class add dev $if_lan parent 2:10 classid 2:200 htb rate 3000kbit ceil $isp_downlink prio 30
                    tc class add dev $if_lan parent 2:10 classid 2:300 htb rate 3000kbit ceil $isp_downlink prio 50
                    tc class add dev $if_lan parent 2:10 classid 2:400 htb rate 750kbit ceil $isp_downlink prio 55 quantum 1514
                    tc class add dev $if_lan parent 2:10 classid 2:500 htb rate 3000kbit ceil $isp_downlink prio 20
                    tc class add dev $if_lan parent 2:10 classid 2:600 htb rate 3750kbit ceil $isp_downlink prio 60
                tc class add dev $if_lan parent 2:1 classid 2:20 htb rate $home_downlink ceil $nic_link quantum 15140
                    tc class add dev $if_lan parent 2:20 classid 2:1000 htb rate 99mbit ceil $home_uplink prio 10 quantum 15140

        tc qdisc add dev $if_lan parent 2:100 handle 100: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:200 handle 200: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:300 handle 300: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:400 handle 400: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:500 handle 500: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:600 handle 600: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_lan parent 2:1000 handle 1000: fq_codel quantum 1514 limit 1024 noecn

        tc qdisc add dev $if_wan parent root handle 1:0 htb default 600 r2q 10
            tc class add dev $if_wan parent 1:0 classid 1:1 htb rate $nic_link ceil $nic_link quantum 15140
                tc class add dev $if_wan parent 1:1 classid 1:10 htb rate $isp_uplink ceil $isp_uplink
                    tc class add dev $if_wan parent 1:10 classid 1:100 htb rate 100kbit ceil $isp_uplink prio 10
                    tc class add dev $if_wan parent 1:10 classid 1:200 htb rate 200kbit ceil $isp_uplink prio 30
                    tc class add dev $if_wan parent 1:10 classid 1:300 htb rate 200kbit ceil $isp_uplink prio 50
                    tc class add dev $if_wan parent 1:10 classid 1:400 htb rate 50kbit ceil $isp_uplink prio 55 quantum 1514
                    tc class add dev $if_wan parent 1:10 classid 1:500 htb rate 200kbit ceil $isp_uplink prio 20
                    tc class add dev $if_wan parent 1:10 classid 1:600 htb rate 250kbit ceil $isp_uplink prio 60
                tc class add dev $if_wan parent 1:1 classid 1:20 htb rate $home_uplink ceil $nic_link quantum 15140
                    tc class add dev $if_wan parent 1:20 classid 1:1000 htb rate 85mbit ceil $home_uplink prio 10 quantum 15140

        tc qdisc add dev $if_wan parent 1:100 handle 100: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:200 handle 200: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:300 handle 300: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:400 handle 400: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:500 handle 500: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:600 handle 600: fq_codel quantum 1514 limit 1024 noecn
        tc qdisc add dev $if_wan parent 1:1000 handle 1000: fq_codel quantum 1514 limit 1024 noecn

        # ICMP, DNS, TIME
        tc filter add dev $if_wan parent 1:0 prio 1 protocol ip handle 1 fw classid 1:100
        tc filter add dev $if_lan parent 2:0 prio 1 protocol ip handle 1 fw classid 2:100

        # HTTP, HTTPS, FTP
        tc filter add dev $if_wan parent 1:0 prio 20 protocol ip handle 2 fw classid 1:200
        tc filter add dev $if_lan parent 2:0 prio 20 protocol ip handle 2 fw classid 2:200

        # MAIL
        tc filter add dev $if_wan parent 1:0 prio 40 protocol ip handle 3 fw classid 1:300
        tc filter add dev $if_lan parent 2:0 prio 40 protocol ip handle 3 fw classid 2:300

        # Jabber
        tc filter add dev $if_wan parent 1:0 prio 5 protocol ip handle 4 fw classid 1:400
        tc filter add dev $if_lan parent 2:0 prio 5 protocol ip handle 4 fw classid 2:400

        # Home Network
        tc filter add dev $if_lan parent 2:0 prio 100 protocol ip handle 10 fw classid 2:1000
    }

W bloku `stop() {}` zaś będą reguły czyszczące:

    stop() {
        tc qdisc del dev $if_wan root
        tc qdisc del dev $if_lan root

        iptables -t mangle -D PREROUTING -j qos_traffic
        iptables -t mangle -D POSTROUTING -j qos_traffic

        iptables -t mangle -F qos_lan
        iptables -t mangle -F qos_wan
        iptables -t mangle -F qos_traffic

        iptables -t mangle -X qos_wan
        iptables -t mangle -X qos_lan
        iptables -t mangle -X qos_traffic
    }

## Zasada działania QoS w OpenWRT

Niżej zostaną opisane linijki wykorzystane w bloku `start() {}` . Oczywiście kształt tego bloku może
się różnić w zależności od gabarytów łącza oraz od tego jakie jego proporcje chcemy przeznaczyć pod
określone usługi. Trzeba także będzie określić politykę QoS. Przykładowo, możemy chcieć, by każdy z
użytkowników naszej sieci miał do dyspozycji określony limit łącza, poza który nie może wyjść.
Możemy też skupić się na nadaniu priorytetów określonym usługom. My korzystamy z tego drugiego
modelu QoS, bo w każdym innym przypadku część łącza będzie marnowana.

Na początku musimy przygotować interfejs przez założenie `qdisc` (Queueing Discipline) i określenie
parametrów dla kolejki głównej:

    tc qdisc add dev $if_lan parent root handle 2:0 htb default 600 r2q 100
    tc class add dev $if_lan parent 2:0 classid 2:1 htb rate $nic_link ceil $nic_link quantum 15140

W tym przypadku korzystamy z [htb](http://luxik.cdi.cz/~devik/qos/htb/manual/userg.htm) i ten
`qdisc` dotyczy interfejsu określonego przez zmienną `$if_lan` . Opcje `parent root` oraz
`handle 2:0` mówią nam, że ten `qdisc` jest nałożony bezpośrednio na interfejs sieciowy i jego
kolejki będą oznaczane `2:` . Dalej mamy `default` i ten parametr odpowiada za ustawienie domyślnej
kolejki dla ruchu, który nie zostanie dopasowany (2:600). W drugiej linijce zaś podpinamy kolejkę o
numerze `classid` pod ten `qdisc` i przy pomocy `rate` (gwarantowany) oraz `ceil` (maksymalny)
określamy przepustowość kolejki. Standardowo `quantum` jest wyliczane przez `rate`/`r2q` . Z kolei
`r2q` domyślnie wynosi `10` . Natomiast wartość parametru `quantum` powinna być większa niż MTU i
mniejsza niż 60000. Więcej informacji można znaleźć [tutaj](http://www.docum.org/faq/cache/31.html).

Mamy oznaczone pasmo, wewnątrz którego tworzymy dwie kolejki. Łącze od ISP różni się gabarytami od
przepustowości domowego łącza, które zwykle ma 1 gbit/s. Dlatego też musimy podzielić to łącze na
dwie kolejki o odpowiednich proporcjach. W przeciwnym razie wewnątrz sieci domowej będziemy
ograniczeni przepustowością ISP:

    tc class add dev $if_lan parent 2:1 classid 2:10 htb rate $isp_downlink ceil $isp_downlink
    tc class add dev $if_lan parent 2:1 classid 2:20 htb rate $home_downlink ceil $nic_link quantum 15140

Każda z tych kolejek może korzystać z pasma drugiej jeśli ta go nie wykorzystuje. Numerki tych
kolejek są określone przez `classid` i są przypisane do głównej kolejki określonej w `parent` .

Po rozdzieleniu dostępnego pasma, możemy w końcu stworzyć właściwe kolejki, które zajmą się łapaniem
pakietów:

    tc class add dev $if_lan parent 2:10 classid 2:100 htb rate 1500kbit ceil $isp_downlink prio 10

Dla każdej z kolejek mamy określony limit gwarantowany oraz maksymalny, z którego mogą skorzystać,
jeśli łącze jest nieobciążone. Mamy również ustawione priorytety w opcjach `prio` . Im niższa
wartość tym wyższy priorytet i kolejka może więcej zapożyczyć od innych podczas wysokiego
obciążenia łącza.

Na każdą kolejkę nakładamy także bezklasowy `qdisc` w celach optymalizacyjnych. Obecnie zaleca się
korzystanie z `fq_codel` :

    tc qdisc add dev $if_lan parent 2:100 handle 100: fq_codel quantum 1514 limit 1024 noecn

Tutaj mamy określony `limit` , który odpowiada za ilość pakietów w kolejce. Jeśli ten limit zostanie
osiągnięty, to pakiety będą zrzucane. Na końcu możemy podać parametr `ecn`/`noecn` . Jeśli zostanie
określony `ecn` , to pakiety zamiast zrzucane będą oznaczane.

Na samym końcu `start() {}` mamy reguły kierowania ruchu do odpowiednich kolejek. Jako, że do
oznaczania ruchu wykorzystujemy target `MARK` w `iptables` , to w oparciu o te markery kierujemy
ruch używając reguł podobnych do tej poniższej:

    tc filter add dev $if_lan parent 2:0 prio 1 protocol ip handle 1 fw classid 2:100

Nas w tych regułach interesuje głównie `handle 1 fw` oraz `classid 2:100` . Pierwszy z tych
parametrów mówi, że dopasowanie dotyczy FWMARK, którym w tym przypadku jest 1. Jeśli taki pakiet
zostanie dostrzeżony, to ma zostać przesłany do kolejki w drugim z w/w parametrów, czyli do 2:100. I
to w zasadzie cała filozofia kształtowania ruchu.

## Testy QoS

Przydałoby się jeszcze na własne oczy zobaczyć czy QoS działa. Odpaliłem zatem torrenta i wrzuciłem
do pobierania kilka obrazów z linux'ami. Po chwili transfer osiągnął szczyt możliwości mojego łącza
i powstałe w ten sposób opóźnienia zostały zobrazowane na fotce poniżej:

![]({{< baseurl >}}/img/2016/05/1.kontrola-ruchu-openwrt-router-tc-iptables-qos.png#huge)

Pierwsze okienko pokazuje aktualne wykorzystanie łącza. Na drugim zaś widać statystyki pingu. W
ostatnim, po SSH zaaplikowaliśmy wyżej stworzony skrypt. Pingi wyraźnie spadły. Wynik mówi raczej
sam za siebie. Odpalmy teraz jakiś filmik na YouTube i sprawdźmy jak wyglądają kolejki na
interfejsach:

![]({{< baseurl >}}/img/2016/05/2.kontrola-ruchu-openwrt-router-tc-iptables-qos.png#huge)

Ruch z sieci P2P idzie na domyślną kolejkę na obu interfejsach (:600). Natomiast ruch www idzie na
kolejkę :200. Każda z tych kolejek wykorzystuje ponad 100% swojego przydziału. Jest to dozwolone i w
takim przypadku te kolejki pożyczają pasmo od tych, które go nie wykorzystują. Przy kolejkach 1:10
oraz 2:10 widzimy, że łącze nie jest w pełni wykorzystane.
