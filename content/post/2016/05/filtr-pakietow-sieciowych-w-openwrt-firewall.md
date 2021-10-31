---
author: Morfik
categories:
- OpenWRT
date: "2016-05-09T23:26:58Z"
date_gmt: 2016-05-09 21:26:58 +0200
published: true
status: publish
tags:
- iptables
- chaos-calmer
- router
GHissueID: 488
title: Filtr pakietów sieciowych w OpenWRT (firewall)
---

Router wyposażony w firmware OpenWRT posiada wbudowany firewall, który ma za zadnie stać na straży
bezpieczeństwa naszej sieci domowej. Standardowo ta zapora przepuszcza cały ruch sieciowy z obszaru
LAN do WAN, czyli z sieci lokalnej do sieci naszego ISP. W ten sposób komputery znajdujące się w
naszej sieci mają dostęp do internetu i mogą z niego korzystać bez przeszkód. Niemniej jednak, ten
mechanizm nie działa tak samo w drugą stronę, czyli z WAN do LAN. Tutaj są już blokowane wszystkie
próby nawiązania nowych połączeń ([za wyjątkiem żądań
ping](/post/kompromitacja-firewalla-openwrt-za-sprawa-ping/)) ale nic nie stoi na
przeszkodzie, by zastosować przekierowanie portów. Dzięki takiemu rozwiązaniu możemy przekierować
ruch, który jest kierowany na dany port w routerze, do określonego hosta w sieci lokalnej. Wszystkie
te zadania realizowane są przez `iptables` i w tym wpisie postaramy się ogarnąć to narzędzie.

<!--more-->
## Własny firewall, czy ten dostarczany z OpenWRT

Przede wszystkim, trzeba powiedzieć, że skoro będziemy operować na `iptables` , to możemy
zrezygnować całkowicie z mechanizmu firewall'a oferowanego przez OpenWRT. Niemniej jednak, na samym
początku będzie nam trudno opanować składnie reguł dla tego filtra. Jeśli nigdy nie mieliśmy do
czynienia z linux'em i pierwszy raz na oczy widzimy nazwę `iptables` , to lepiej zostać przy
mechanizmie oferowanym przez OpenWRT. W tym artykule tak właśnie zrobimy. Jeśli jednak ktoś chciałby
się zagłębić w [tematykę iptables, to może zacząć od tego
linku](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html). Na tym blogu poruszałem
też już zagadnienie dotyczące [budowy firewall'a dla klientów działających pod
linux'em](/post/firewall-na-linuxowe-maszyny-klienckie/). Jako, że wszystkie
zawarte w tamtym wpisie informacje można zastosować w tym przypadku, to zachęcam też do zajrzenia i
zapoznania się z tamtym artykułem.

Firewall w OpenWRT składa się z grubsza z dwóch plików konfiguracyjnych, na których będziemy
operować. Pierwszym z nich jest `/etc/config/firewall` . W tym pliku znajduje się cała domyślna
konfiguracja, która zabezpiecza nasz router i sieć przed dostępem od strony interfejsu WAN. To w nim
będziemy definiować podstawowe regułki, które będą mieć za zadanie przekierować czy też zablokować
przepływ pakietów. Drugim z plików jest `/etc/firewall.user` . Do niego wędrują reguły, które z
jakiegoś powodu nie mogą trafić do pliku `/etc/config/firewall` . Zwykle w nim będziemy umieszczać
bardziej zaawansowane rzeczy, których nie da się zbytnio wyrzeźbić w oparciu od ten standardowy
mechanizm filtrowania pakietów.

Firewall dostępny w OpenWRT został podzielony na szereg pakietów. Wszystkie niestandardowe lub też
rzadko używane moduły zostały upchnięte w osobnych paczkach. Zatem nie wszystkie opcje oferowane
przez `iptables` są dostępne. Nic jednak nie stoi na przeszkodzie, by sobie doinstalować dodatkowe
pakiety. Każdy z takich pakietów ma opis na temat tego jaki moduł zostanie zaimplementowany po jego
instalacji. Listę pakietów dostępnych w repozytorium możemy uzyskać wpisując w terminalu to poniższe
polecenie:

    # opkg list | grep iptables-mod

Skrypt aplikujący reguły `iptables` znajduje się w `/etc/init.d/firewall` . Przy jego pomocy możemy
wyczyścić aktualnie zaaplikowane reguły lub też przeładować konfigurację:

    # /etc/init.d/firewall reload

## Analiza struktury firewall'a

Zanim jednak się weźmiemy za analizę plików `/etc/config/firewall` oraz `/etc/firewall.user` , warto
przybliżyć sobie nieco strukturę firewall'a. Przede wszystkim, `iptables` składa się z czterech
tablic `raw` , `mangle` , `nat` oraz `filter` . W każdej z tablicy mamy szereg wbudowanych
łańcuchów, przez które przechodzą pakiety. Są też z grubsza trzy kierunki przepływu pakietów: do
routera, z routera i przez router. Wszystkie te rzeczy są dokładnie zobrazowane na poniższych
fotkach ([źródło](https://commons.wikimedia.org/wiki/File:Netfilter-packet-flow.svg)):

![firewall-iptables-przeplyw-pakietow](/img/2015/06/1.firewall-iptables-przeplyw-pakietow.png#medium)

oraz:

![przeplyw-pakietow-netfilter-iptables-openwrt-firewall](/img/2016/05/1.przeplyw-pakietow-netfilter-iptables-openwrt-firewall.png#huge)

Te schematy zostały jednak nieco rozbudowane przez OpenWRT. Chodzi generalnie o to, że w każdej z
tych czterech tablic można definiować własne łańcuchy i kierować do nich ruch. OpenWRT dostarcza
szereg takich łańcuchów. Każdy z nich pełni pewną rolę w strukturze filtra. Bez zrozumienia zasady
przepływu pakietów w oparciu o te dodatkowe łańcuchy, nie będziemy w stanie komfortowo operować na
filtrze udostępnianym przez OpenWRT.

Z reguły będziemy operować na tablicach `nat` oraz `filter` . W `nat` będą umieszczane reguły
związane z translacją adresów, np. przekierowanie portów. Z klei w tablicy `filter` będziemy
zarządzać blokowaniem i przepuszczaniem pakietów.

### DROP vs. REJECT

Reguły mogą być akceptowane (ACCEPT), odrzucane (REJECT) oraz zrzucane (DROP). Te dwie ostatnie
opcje dotyczą negatywnej odpowiedzi na przesłane żądanie połączenia. Która zatem z nich jest lepsza,
DROP czy REJECT? Na dobrą sprawę nie ma jednoznacznej odpowiedzi i musimy się na którąś z tych opcji
zdecydować. Przede wszystkim, DROP nie udostępnia żadnych informacji osobie, która próbuje się
podłączyć. Jeśli taki pakiet zostanie zrzucony, to druga strona o tym może z początku nawet nie
wiedzieć. Utrudnia to trochę ustalanie przyczyny ewentualnych problemów z siecią. Poza tym, szereg
aplikacji sieciowych może reagować w bardzo powolny sposób, jako że nie dostają żadnej odpowiedzi
od routera. W przypadku REJECT mamy te powyższe informacje i te problemy nas nie dotyczą. Z drugiej
zaś strony, skoro my posiadamy te informacje, to i potencjalny atakujący również. Dodatkowo, jako,
że router udziela odpowiedzi na szereg nieprawidłowych żądań komunikatami ICMP, to może to w
pewnych sytuacjach wykończyć router.

## Analiza reguł dostępnych w pliku /etc/config/firewall

Przejdźmy zatem do analizy pliku `/etc/config/firewall` . Jest on dość rozbudowany ale zaraz sobie
wyjaśnimy za co odpowiadają konkretne sekcje i linijki w nich zawarte. Wszystkie dostępne opcje,
które można umieścić w tym pliku, są do odszukania na [wiki
OpenWRT](https://wiki.openwrt.org/doc/uci/firewall). Plik `/etc/config/firewall` jest podzielony
szereg na sekcji. Poniżej znajduje się opis każdej z nich.

### Konfiguracja zachowania firewall'a (sekcja config defaults)

W sekcji `config defaults` znajdują się ustawienia firewall'a, które nie są przypisane do żadnej
innej sekcji. Poniżej przykład ustawień:

    config defaults
        option drop_invalid        1
        option synflood_protect    1
        option synflood_rate       25/second
        option synflood_burst      50
        option tcp_syncookies      1
        option tcp_window_scaling  1
        option accept_redirects    0
        option accept_source_route 0
        option input               DROP
        option output              ACCEPT
        option forward             DROP
    #   option disable_ipv6        1

Jak widzimy wyżej, mamy szereg opcji, które sterują zachowaniem firewall'a. Opcja `drop_invalid`
jest w stanie blokować pakiety, które mają przykładowo błędną konfigurację flag w protokole TCP. Z
kolei `synflood_protect` chroni przed nawiązywaniem przez router zbyt dużej ilości nowych połączeń w
danej chwili. Limit zaś jest określony w opach `synflood_rate` oraz `synflood_burst` , czyli z
grubsza 25 nowych połączeń na sekundę. Wszystkie połączenia ponad ten limit zostaną zrzucone. Mamy
również [mechanizm ciasteczek (SYN
cookies)](/post/mechanizm-syn-cookies-w-protokole-tcp/) włączony za sprawą opcji
`tcp_syncookies` . Z kolei opcja `tcp_window_scaling` włącza [skalowanie okna TCP (bufor dla
połączeń)](/post/bufor-polaczen-w-protokole-tcp/). Dalej mamy opcje
`accept_redirects` oraz `accept_source_route` , które chronią przed zmianami tablic routingu, co
często wykorzystywane jest w różnego rodzaju atakach. Kolejne trzy opcje, tj. `input` , `output`
oraz `forward` odpowiadają za ustawienie domyślnej polityki dla zdefiniowanych stref (o tym za
moment). Opcja `disable_ipv6` jest w stanie wyłączyć reguły IPv6. Nie oznacza to, że obsługa
protokołu IPv6 zostanie wyłączona, a jedynie nie będą aplikowane same reguły firewall'a, co może
nie być bezpieczne.

### Strefy (sekcja config zone)

Dalej w pliku `/etc/config/firewall` są zdefiniowane strefy, które są wykorzystywane później w
dalszej konfiguracji firewall'a. Poniżej znajduje się przykład strefy `wan` :

    config zone
        option name     wan
        list   network      'wan'
        list   network      'wan6'
        option input        DROP
        option output       ACCEPT
        option forward      DROP
        option masq     1
        option conntrack    1
        option mtu_fix      1
        option log      0
        option log_limit    60/minute

By móc się odwołać do takiej strefy w regułach, które będziemy umieszczać również w pliku
`/etc/config/firewall` , musimy nadać strefie nazwę i za to odpowiada opcja `name` . W `list
network` mamy logiczne interfejsy, które są podpięte pod tę strefę. Nazwy tych logicznych
interfejsów są do odczytania z pliku `/etc/config/network` . Opcje `input` , `output` oraz
`forward` ustawiają politykę domyślną dla pakietów w tej strefie. Dalej mamy opcję `masq` , przy
pomocy której ruch z sieci lokalnej może zostać przesłany przez ten interfejs (maskarada). Z kolei
`conntrack` włącza śledzenie połączeń dla tej strefy. Zwykle ta opcja jest automatycznie aktywowana
jeśli poprzednia jest włączona. Opcja `mtu_fix` ustawia rozmiar MTU w oparciu o
[PMTUD](https://en.wikipedia.org/wiki/Path_MTU_Discovery). Ostatnie dwie opcje, tj. `log` oraz
`log_limit` odpowiadają za logowanie pakietów odrzuconych w tej strefie.

### Forwarding pakietów (sekcja config forwarding)

Mając zdefiniowane co najmniej dwie strefy, `lan` i `wan` , możemy ustawić między nimi forwarding
pakietów. W ten sposób hosty z sieci lokalnej będą mieć dostęp do internetu. Poniżej jest przykład
odpowiedniej sekcji:

    config forwarding
        option src      lan
        option dest     wan

Tutaj mamy tylko dwie opcje, `src` oraz `dest` , które określają kierunek przepływu pakietów. W tym
przypadku pakiety z sieci lokalnej będą forward'owane do internetu bez problemu.

### Reguły (sekcja config rule)

Dalej w pliku `/etc/config/firewall` mamy sekcje z regułami. To one sterują przepływem pakietów
przez filtr i decydują czy dane połączenie zostanie zaakceptowane lub odrzucone. Poniżej przykładowe
reguły dla protokołu IPv4:

    config rule
        option name         Allow-DHCP-Renew
        option src          wan
        option family       ipv4
        option proto        udp
        option dest_port    68
        option target       ACCEPT
        option src_ip       "192.168.0.0/16 10.0.0.0/8"
        option enabled      1

oraz dla IPv6:

    config rule
        option name         Allow-ICMPv6-Input
        option src          wan
        option proto        icmp
        list icmp_type      echo-request
        list icmp_type      echo-reply
        list icmp_type      destination-unreachable
        list icmp_type      packet-too-big
        list icmp_type      time-exceeded
        list icmp_type      bad-header
        list icmp_type      unknown-header-type
        list icmp_type      router-solicitation
        list icmp_type      neighbour-solicitation
        list icmp_type      router-advertisement
        list icmp_type      neighbour-advertisement
        option limit        1000/sec
        option family       ipv6
        option target       ACCEPT
        option enabled      1

Reguły dobrze jest nazywać, tak by przy listowaniu ich w filtrze były stosownie oznaczone. Do tego
służy opcja `name` . Przy pomocy `src` jest oznaczany kierunek z którego pakiety mają docierać do
routera. W tym przypadku reguły dotyczą strefy `wan` , tj. tej samej, którą określiliśmy wyżej w
pliku. Tak skonstruowana reguła trafi do łańcucha `zone_wan_input` . Jeśli opcja `family` nie
zostanie podana, to reguła tyczy się zarówno protokołu IPv4 jak i IPv6, chyba, że inne opcje reguły
(np. IP) będą wskazywać, że ma być to reguła dla określonego protokołu. Następnie mamy szereg opcji,
które są podawane do `iptables` . Jeśli opcja `enabled` jest ustawiona, to taka reguła jest dodawana
do firewall'a. W przypadku takich reguł jak te powyższe, gdzie w jakiejś opcji mamy więcej niż jeden
argument, to ostatecznie zostanie utworzonych po jednej regule dla każdego argumentu.

Poniżej znajduje się lista tych częściej używanych opcji, które można zdefiniować w regule:

  - `src` oraz `dest` -- źródłowa i docelowa strefa. Jeśli obie są określone, to ruch jest
    forward'owany.
  - `src_ip` oraz `dest_ip` -- źródłowy i docelowy adres IP.
  - `src_port` oraz `dest_port` -- źródłowy i docelowy port. Można podać w formie '80 443', jak i
    też w postaci zakresu '1000:2000'.
  - `proto` -- nazwa (numer) protokołu. Do wyboru tcp, udp, tcpudp, udplite, icmp, esp, ah, sctp,
    lub all.
  - `list icmp_type` -- w przypadku, gdy `proto` wskazuje na `icmp` , to można dodatkowo określić
    rodzaj komunikatu w formie nazwy lub jego numeru ID.
  - `target` -- akcja. Możliwe do wyboru ACCEPT, REJECT, DROP, MARK oraz NOTRACK.
  - `limit` oraz `limit_burst` -- ustawiają limit dla połączeń. Limit można określić w formie
    1/minute, 1/min, lub 1/m.

### Przekierowania i port forwarding (sekcja config redirect)

Zanim przejdziemy do samych reguł, trzeba zapoznać się z [Network Address Translation
(NAT)](https://pl.wikipedia.org/wiki/Network_Address_Translation), czyli z translacją adresów. Każdy
komputer wpięty w naszą sieć LAN chce uzyskać dostęp do internetu. Niestety liczba adresów IP jest
skończona i do tego na wyczerpaniu, dlatego musimy korzystać z NAT. W tej sytuacji wszystkie te
maszyny lokalne wysyłają pakiety do routera z adresem docelowym znajdującym się gdzieś w internecie.
By komunikacja mogła nastąpić, router podmienia w tych pakietach źródłowy adres IP na swój własny i
taki pakiet przesyła przez sieć. Później przychodzi odpowiedź na adres IP routera, a ten w oparciu o
mechanizm śledzenia połączeń wie, do której maszyny w sieci LAN przesłać dany pakiet. W efekcie
wszystkie maszyny w lokalnej sieci mają ten sam adres zewnętrzny, który otrzymał od ISP nasz router.

Proces translacji adresów jest dokonywany przez firewall, a konkretnie przez tablicę `nat` w
`iptables` . W zależności od kierunku pakietów, mamy z grubsza dwa rodzaje NAT. W przypadku
przesyłania pakietów z sieci lokalnej do internetu stosowany jest Source NAT (SNAT). Z kolei w
drugą stronę korzysta się z DNAT (Destination NAT). Istnieje również maskarada (MASQUERADE) i ona
jest specjalną odmianą SNAT, która adaptuje się do zmiennego adresu IP przydzielanego interfejsowi
WAN (najczęściej stosowana).

Translacja adresów w firewall'u OpenWRT jest z reguły włączona na interfejsie WAN. Jest to
warunkowane dodaniem opcji `masq` do powyżej zdefiniowanych stref w pliku `/etc/config/firewall` . W
takim przypadku, wszystkie nowe połączenia (stan NEW) przechodzą przez tablicę `nat` w `iptables` i
jeśli taki pakiet rozpoczynający nowe połączenie opuszcza router przez interfejs WAN, to jest
przepuszczany przez regułę maskarady. Wszystkie następne pakiety powiązane z ustanowionym
połączeniem są opisywane w oparciu o tablicę conntrack'a, gdzie kernel śledzi sobie dokładnie
każde połączenie. Poniżej znajduje się taki przykładowy wpis (do wglądu w
`/proc/net/nf_conntrack` ):

    ipv4     2 tcp      6 3575 ESTABLISHED src=77.68.11.22 dst=82.160.11.22 sport=41369 dport=8398 packets=195 bytes=13899 src=192.168.1.150 dst=77.68.11.22 sport=8398 dport=41369 packets=193 bytes=143243 [ASSURED] mark=4 use=2

Mamy tutaj połączenie TCP w protokole IPv4 w stanie ustanowionym (ESTABLISHED). Pakiety są
wymieniane na linii 192.168.1.150:8398 77.68.11.22:41369 . W zależności od tego, która strona
zainicjowała połączenie (wysłała pakiet w stanie NEW), to mielibyśmy do czynienia z SNAT lub DNAT.
Jako, że połączenie jest ustanowione, to pakiety są przesyłane w obu kierunkach automatycznie i
przeadresowane w oparciu o ten powyższy wpis. Każde połączenie jest identyfikowane w oparciu o
protokół, źródłowy i docelowy adres IP oraz źródłowy i docelowy port. Router natomiast potrafi w
oparciu o ten powyższy wpis powiązać ze sobą te informacje i wie gdzie przesłać jaki pakiet.

Biorąc pod uwagę powyższe informacje, pora zapoznać się z regułami, które przekierowują ruch na
routerze. Używane są one w dużej mierze przy port forwarding'u (otwieraniu portów od strony WAN).
Taka reguła różni się nieco od tych opisanych powyżej. Wygląda ona mniej więcej tak:

    config redirect
        option name       'ssh'
        option src        'wan'
        option dest       'lan'
        option proto      'tcp'
        option src_dport  '80'
        option dest_ip    '192.168.1.234'
        option dest_port  '80'
        option target     'DNAT'

Część z powyższych opcji ma takie samo znaczenie co w przypadku zwykłych reguł opisanych wyżej. W
opcji `target` można określić wartości DNAT lub SNAT. W zależności od wyboru zmienia się nieco
znaczenie opcji `src_dip` , `src_dport` , `dest_ip` oraz `dest_port` . Poniżej jest rozpiska
dotycząca tych opcji:

  - `dest_ip` -- w przypadku DNAT, ruch kierowany jest do określonego tutaj hosta. Dla SNAT
    dopasowywany jest ruch w oparciu i ten adres.
  - `src_dip` -- w przypadku DNAT, ruch jest kierowany na ten adres IP. Dla SNAT przepisywany jest
    źródłowy adres na wartość ustawioną w tej opcji.
  - `src_dport` -- w przypadku DNAT, ruch jest kierowany na dany port lub zakres portów docelowych.
    Dla SNAT przepisywane są porty źródłowe na wartość ustawioną w tej opcji.
  - `dest_port` -- w przypadku DNAT, ruch jest kierowany na dany port hosta. Dla SNAT dopasowuje
    ruch kierowany na podany port.

Dodatkowo, po określeniu opcji `src_dport` ustawiany jest także [NAT Loopback/NAT
Reflection](/post/nat-reflection-oraz-nat-loopback-w-openwrt/).

## Sprawdzanie reguł firewall'a

Wszystkie reguły, które określiliśmy w pliku `/etc/config/firewall` oraz `/etc/firewall.user` możemy
w dowolnym momencie sobie podejrzeć. Wystarczy zalogować się na router i w terminalu wpisać to
poniższe polecenie:

    # iptables -nvL

Zwróci ono dość obszerny log. Domyślnie są listowane reguły dla tablicy `filter` . By zobaczyć
reguły w innych tablicach, musimy podać przełącznik `-t` , przykładowo:

    # iptables -nvL -t nat

Wszystkie reguły w formie takiej jak są podawane bezpośrednio do `iptables` można uzyskać posługując
się przełącznikiem `-S` :

    # iptables -S -t filter

Tak wygenerowane reguły nadają się do dalszego studiowania i analizy. W oparciu o nie można poznać
sporą część składki dla firewall'a, co w przyszłości przełoży się na własny skrypt z regułami, który
ma o wiele większe możliwości niż mechanizm oferowany przez OpenWRT.
