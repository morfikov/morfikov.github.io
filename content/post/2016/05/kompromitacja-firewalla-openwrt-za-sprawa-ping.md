---
author: Morfik
categories:
- OpenWRT
date: "2016-05-09T15:40:47Z"
date_gmt: 2016-05-09 13:40:47 +0200
published: true
status: publish
tags:
- iptables
- chaos-calmer
- router
GHissueID: 485
title: Kompromitacja firewall'a OpenWRT za sprawą ping
---

Ten standardowy firewall, który oferuje OpenWRT, ma w zamiarze blokować wszystkie nowe próby
połączeń od strony WAN. Faktycznie tak jest w istocie. Niemniej jednak, mamy tam jedną regułę,
która zezwala na wysyłanie żądań `ping` . Niby te żądania wydają się być niepozorne ale przy takiej
konfiguracji `iptables` jaką oferuje OpenWRT istnieje ryzyko, że ktoś z zewnątrz może utworzyć sporo
sesji bez jakiegokolwiek nadzoru. Każda z tych sesji musi być śledzona przez kernel w tablicy
conntrack'a. Nie mając kontroli nad tym ile takich sesji może zostać utworzonych, łatwo może dojść
do zapełnienia tej tablicy. Jeśli do tego dojdzie, to router przestanie nawiązywać nowe połączenia.
Przydałoby się zatem jakoś ten cały `ping` ogarnąć i to niekoniecznie blokując go po stronie WAN. W
tym wpisie zaimplementujemy sobie mechanizm ochrony przez tego typu zagrożeniem.

<!--more-->
## Tablica conntrack'a

Jako, że na routerze z firmware OpenWRT mamy do czynienia z kernelem linux'a, to obowiązują nas te
same zasady co w przypadku pełnowymiarowych pingwinów. Tablica conntrack'a ma stały rozmiar, który
można odczytać via `sysctl` . Poniżej przykład:

    # sysctl -a | grep conntrack_max
    net.netfilter.nf_conntrack_max = 16384
    net.nf_conntrack_max = 16384

Zatem w przypadku tego routera, tj. [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html), rozmiar tej tablicy to 16384 wpisów.
Każde połączenie, które nawiązuje router jest śledzone przez kernel właśnie w oparciu o te wpisy.
Wszystkie z nich figurują w pliku `/proc/net/nf_conntrack` . Jeśli teraz spróbujemy ping'nąć router
od strony WAN, to w tym pliku powinien zostać utworzony nowy wpis opisujący połączenie. Wygląda to
mniej więcej tak jak na poniższym przykładzie:

    # cat /proc/net/nf_conntrack | grep -i icmp
    ipv4     2 icmp     1 28 src=11.22.33.44 dst=44.33.22.11 type=8 code=0 id=5557 packets=23 bytes=1932
    src=44.33.22.11 dst=11.22.33.44 type=0 code=0 id=5557 packets=23 bytes=1932 mark=1 use=2

Ten wpis nie pozostaje obojętny dla routera. Już nawet nie chodzi o informacje opisujące to
połączenie. Rzecz w tym, że taki wpis zjada dość sporo miejsca w pamięci. Jest to około 200
bajtów. Z tego właśnie względu, rozmiar tej tablicy jest ograniczony. Ilość wpisów w liczbie 16384
jest wystarczająca na domowe potrzeby przeciętnego Kowalskiego.

Jeśli nasz router dysponuje większą ilością pamięci RAM, to zawsze można podnieść ten limit. Robimy
to poprzez plik `/etc/sysctl.conf` , gdzie dopisujemy lub zmieniamy parametry
`net.netfilter.nf_conntrack_max` . Jeśli chcielibyśmy ustawić limit na 32768 wpisów, to dodajemy w
tym pliku tę poniższą linijkę:

    net.netfilter.nf_conntrack_max=32768

Niemniej jednak, podbicie limitu na niewiele się zda. Wzrośnie nam za to wykorzystanie pamięci
operacyjnej. Jeśli ktoś od strony WAN zacznie nawiązywać kolejne sesje ping'a, to ta tablica
conntrack'a i tak się może zapełnić. Zajmie to może trochę więcej czasu ale do zapełnienia dojdzie.
W efekcie nie będziemy w stanie nawiązać nowych połączeń, wliczając w to również połączenia
wychodzące z naszej sieci lokalnej.

## Ograniczanie reguły odpowiedzialnej za ping

By ten problem wyeliminować, można spróbować doszczelnić zaporę od strony WAN. Krótko mówiąc, trzeba
przerobić tę standardową regułę zezwalającą na `ping` . Znajduje się ona w pliku
`/etc/config/firewall` . Najprościej byłoby zmienić jej `target` z `ACCEPT` na `DROP` :

    config rule
        option name             Allow-Ping
        option src              wan
        option proto            icmp
        option icmp_type        echo-request
        option family           ipv4
        option target           DROP

Oczywiście to powyższe rozwiązanie nie jest zbytnio eleganckie, bo przecież raz na jakiś czas możemy
sami chcieć ping'nąć router od strony WAN i co wtedy? Możemy tę powyższą regułę nieco inaczej
przepisać. Zamiast blokować wszystkie żądania `ping` od strony WAN, możemy zezwolić tylko konkretnym
adresom/sieciom na ich przesyłanie. [Potrzebna nam będzie opcja
src\_ip](https://wiki.openwrt.org/doc/uci/firewall#rules). Poniżej jest cała reguła:

    config rule
        option name             Allow-Ping
        option src              wan
        option proto            icmp
        option icmp_type        echo-request
        option family           ipv4
        option src_ip           '192.168.0.0/16 10.0.0.0/8'
        option target           ACCEPT

W przypadku, gdy nie wiemy za bardzo z jakich adresów przyjdzie nam operować na naszym routerze od
strony WAN, możemy pokusić się o nieco bardziej wyrafinowany mechanizm, który ograniczy ilość
jednoczesnych sesji `ping` do 50 czy jakiejś innej niskiej wartości. W takim przypadku, tylko 50
wpisów w tablicy conntrack'a będzie przeznaczone na te połączenia. Wszystkie żądania ponad limit
będą zwyczajnie zrzucane. By tego typu rozwiązanie zaimplementować na routerze wyposażonym w
OpenWRT, musimy sięgnąć do pliku `/etc/firewall.user` . Edytujemy zatem ten plik i dodajemy w nim te
poniższe linijki:

    . /lib/functions/network.sh
    network_get_physdev ifname wan
    
    iptables -t filter -A input_rule -i $ifname -p icmp --icmp-type echo-request -m limit --limit 25/s --limit-burst 25 -j ACCEPT -m comment --comment "Limit Ping"
    iptables -t filter -A input_rule -i $ifname -p icmp --icmp-type echo-request -j DROP -m comment --comment "Drop Ping"

Dwie pierwsze linijki pomogą nam w ustaleniu nazwy interfejsu WAN. Nie musimy jej na sztywno
wpisywać, bo ona przecie może ulec zmianie. Ostatnie dwie linijki to reguły `iptables` , które
wędrują do łańcucha stworzonego przez OpenWRT, czyli `input_rule` . Pierwsza reguła akceptuje
wszystkie pakiety ping, które mieszczą się w limicie 25/s. Pobieżnie licząc, to wystarczy na 25
jednoczesnych sesji, które ping'ują router non stop. W tej regule został także określony parametr
`--limit-burst` , który robi za swojego rodzaju bufor. Został on ustawiony na 25 tokenów. W
przypadku, gdy limit 25 pakietów na sekundę zostanie przekroczony, to tokeny będą wybierane z tego
bufora. W ten sposób router tymczasowo będzie tolerował więcej pakietów `ping` niż 25/s. Jeśli to
będzie 26/s, to tokeny skończą się po 25 sekundach i zacznie się zrzucanie pakietów. W skrajnym
przypadku, atakujący mógłby nawiązać 50 jednoczesnych sesji. W ten sposób nie uda mu się przepełnić
tablicy i przeprowadzić ataku DOS lub DDOS na router z wykorzystaniem pospolitego ping'a.

## Czemu nie założyliśmy limitu w /etc/config/firewall

W pliku `/etc/config/firewall` mamy opcje, przy pomocy których możemy limitować poszczególne reguły.
Powodem, dla którego nie określiliśmy limitu dla umieszczonej tam reguły dotyczącej żądań `ping`
jest to, że nic by nam to nie dało. Reguły, które umieszczone są w pliku `/etc/config/firewall`
wędrują do zdefiniowanych łańcuchów. W przypadku interfejsu WAN jest to `zone_wan_input` . Reguły w
tym łańcuchu są przetwarzane po regule łapiącej pakiety z połączeń już nawiązanych (stan
ESTABLISHED,RELATED). W efekcie limitowalibyśmy jedynie ilość nowych żądań ping (stan NEW).
Wszystkie nawiązane już sesje, by były łapane przez wcześniejszą regułę z ESTABLISHED,RELATED. W
efekcie atakujący mógłby w ustalonym tempie zapełniać tablicę conntrack'a i w końcu udałoby się
osiągnąć zamierzony cel.
