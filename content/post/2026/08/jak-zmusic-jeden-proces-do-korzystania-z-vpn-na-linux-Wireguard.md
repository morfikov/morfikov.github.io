---
author: Morfik
categories:
- Linux
date:    2026-08-07 02:42:00 +0200
lastmod: 2026-08-07 02:42:00 +0200
params:
  params:
  published: true
status: publish
tags:
- debian
- wireguard
- vpn
- proton-vpn
- nftables
- routing
- prywatność
- sieć
GHissueID: 607
title: Jak zmusić jeden proces do korzystania z VPN na linux (Wireguard)
---

Jakiś już dłuższy czas temu opisywałem mechanizm [przekierowania ruchu sieciowego jednego lub
wybranych procesów do VPN na przykładzie OpenVPN][1] pod moim Debianem. Zawarta w tamtym artykule
konfiguracja jak najbardziej działa do tej pory, choć minęło już prawie 6 lat od jej wdrożenia w
życie. Od tamtego momentu jednak, sporo się zmieniło w świecie linux'a, a konkretnie pojawił się
[wireguard][3], który jest nieco bardziej dostosowany do otaczającej nas rzeczywistości i kładzie
na łopatki wysłużonego już OpenVPN. Nie mam nic do OpenVPN ale przydałoby się zrobić podobne
rozwiązanie opisane w powyżej podlinkowanym artykule ale na przykładzie właśnie wireguard.
Samo [skonfigurowanie do pracy VPN na bazie wireguard][2] jest w zasadzie dość proste i nie trzeba
pisać kolejnego artykułu na ten temat. Niemniej jednak, stawiając VPN w opisany na stronie
wireguard sposób będziemy przesyłać przez to połączenie cały ruch sieciowy, czego raczej nie chcemy.
To tutaj właśnie zaczynają się problemy z wireguard, których w OpenVPN nie szło zaobserwować, a
które uniemożliwiają poprawne działanie (zapętlanie się) połączenia VPN. Postaramy się zatem
przyjrzeć nieco bliżej samej konfiguracji wireguard i spróbować sprawić, by ruch sieciowy tylko
wybranych przez nas procesów był kierowany w tunel VPN.

<!--more-->

## Potrzebne narzędzia

By poprawnie skonfigurować połączenie VPN na bazie wireguard będziemy potrzebować narzędzi z
pakietu `wireguard-tools` oraz `nftables` . Można oczywiście korzystać również i z `iptables` ale w
tym przypadku będziemy pisać reguły dla `nftables` .

## Konfiguracja kernela

Dodatkowo będzie nam potrzebna odpowiednia konfiguracja samego kernela linux. Dystrybucyjne jądra
Debiana powinny mieć już wszystko, co jest potrzebne. Tak czy inaczej, jeśli [budujemy sobie własny
kernel][4], to upewnijmy się, że mamy włączoną poniższą opcję:

    CONFIG_WIREGUARD

### Obsługa debugfs

Dobrze jest sobie włączyć też te dwie opcje odpowiadające za wirtualny system plików `debugfs` ,
które przydadzą nam się przy ewentualnym debugowaniu wireguard'a:

    CONFIG_DEBUG_FS=y
    CONFIG_DEBUG_FS_ALLOW_NONE=y

Opcja `CONFIG_DEBUG_FS_ALLOW_NONE` domyślnie wyłącza dostęp do systemu plików `debugfs` i potrzebny
nam będzie dodatkowy parametr `debugfs=on` , który trzeba będzie dopisać w linijce kernela w
konfiguracji EFI/UEFI czy też konfiguracji samego bootloader'a. Tego typu ustawienie sprawi, że
domyślnie system będzie nam startował z wyłączonym `debugfs` , a gdy zajdzie potrzeba, to będziemy
mogli go sobie włączyć bez ponownej kompilacji kernela.

## Konfiguracja wireguard

By być w stanie zestawić połączenie VPN przy pomocy wireguard, potrzebujemy jakaś maszynę, do
której chcemy się podłączyć w sposób bezpieczny. Tutaj skorzystamy sobie z VPN oferowanego przez
[protonvpn][5], bo on wspiera wireguard i jest darmowy. [Standardowa konfiguracja wireguard][6],
którą można pobrać od tego operatora VPN, wygląda mniej więcej tak:

    [Interface]
    # Key for morfikownia
    # Bouncing = 3
    # NAT-PMP (Port Forwarding) = off
    # VPN Accelerator = off
    PrivateKey = scisle-tajne-lamane-przez-poufne
    Address = 10.2.0.2/32
    DNS = 10.2.0.1

    [Peer]
    # US-FREE#70
    PublicKey = MNTtBSAv3VQ8W059Rbf7pQ37WDxQ4h5vkhmBARjmmiU=
    AllowedIPs = 0.0.0.0/0, ::/0
    Endpoint = 151.243.141.159:51820

    PersistentKeepalive = 25

Klucz widoczny w `PrivateKey` generujemy przy pomocy poniższego polecenia:

    # wg genkey
    scisle-tajne-lamane-przez-poufne

Gdybyśmy zapisali tę powyższą konfigurację w pliku `/etc/wireguard/wgvpn0.conf` i skonfigurowali
interfejs `wgvpn0` przy pomocy skryptu `wg-quick` , to po chwili mielibyśmy działający tunel VPN i
cały ruch sieciowy (tj. tylko ten kierowany przez bramę domyślną) z naszego hosta byłby szyfrowany i
przesyłany na adres określony w `Endpoint` .

## Poprawienie standardowej konfiguracji wireguard

Problemów z tą domyślną konfiguracją wireguard'a jest kilka. Pierwszy problem dotyczy braku
możliwości określenia procesów, które chcemy puścić przez VPN. Drugi problem to przesyłanie ruchu
DNS przez połączenie VPN. A trzeci problem to dotykanie konfiguracji `iptables` / `nftables` , choć
dla osób mało technicznych (niemających własnej konfiguracji firewall'a) to akurat uzasadnione.
Musimy wyeliminować wszystkie te trzy problemy.

### Puszczenie ruchu DNS innym kanałem niż przez VPN

Standardowo cały ruch sieciowy naszej maszyny (w tym ruch DNS) jest kierowany do VPN wireguard, o
ile to połączenie konfigurowaliśmy via `wg-quick` . W pobranej od operatora VPN konfiguracji
wireguard mieliśmy opcję `DNS` , która odpowiada za skonfigurowanie adresu resolver'a DNS na
maszynie hosta, tak by zapobiec przeciekom DNS ([DNS leak][7]). Jeśli zatem mamy umieszczoną
opcję `DNS` w konfiguracji wireguard, to system podmieni nam wpisy w pliku `/etc/resolv.conf` za
sprawą automatu `resolvconf` , który zwykle jest obecny w standardowych instalacjach Debiana.

U mnie w systemie nie ma `resolvconf` , ani też żadne procesy czy użytkownicy (w tym też root) nie
mają prawa dotykać pliku `/etc/resolv.conf` za sprawą nadania temu pliku atrybutu odporności,
tj. `chattr +i` . W tym pliku mam w zasadzie jedynie wpis `nameserver 127.0.0.1` , który wskazuje
na adres pętli zwrotnej, na którym to nasłuchuje `dnsmasq` , który to z kolei łapie wszystkie
zapytania DNS i kieruje je do `dnscrypt-proxy` , który również nasłuchuje lokalnie. To ten
cały [mechanizm dnsmasq i dnscrypt-proxy][8] odpowiadają za przesyłanie w moim Debianie zapytań DNS
do upstream'owego serwera DNS niezależnym zaszyfrowanym kanałem.

U mnie zawsze tylko część procesów korzysta z VPN, zatem równolegle są wysyłane zapytania DNS przez
procesy korzystające i niekorzystające z VPN. Mając `dnscrypt-proxy` i `dnsmasq` oraz atrybut
`chattr +i` na pliku `/etc/resolv.conf` mam pewność, że zapytania DNS będą szyfrowane bez względu
na to, z którego (i w ogóle czy) VPN korzystam (wireguard/openvpn).

Dla osób, które uważają, że ten powyższy mechanizm to lekka przesada (albo i nieco większa), to
trzeba tutaj zaznaczyć, że jeśli puścimy ruch DNS przez tunel VPN, którym będzie leciał także
pozostały ruch sieciowy, to operator VPN analizując ruch DNS jest w stanie pozyskać informacje o
domenach, które odwiedzamy. Jeśli kierujemy ruch DNS osobnym szyfrowanym kanałem, to te informacje
operatorowi VPN odbieramy i widzi on jedynie adresy IP (choć naturalnie [trzeba pamiętać o SNI][9]).
Z kolei operator DNS widzi domeny, które odwiedzamy ale nie ma żadnych informacji dotyczących
samych połączeń (np. czas trwania takiego połączenia czy ilość danych wysłanych/odebranych). Tego
typu rozdział ról między dwa podmioty rzutuje dość istotnie na kwestię naszej prywatności, a
przecież nie po to używamy VPN, by za chwilę wręczać naszą prywatność w ręce tegoż operatora VPN.

Dlatego jeśli mamy własną konfigurację resolver'a, np. tę wspomnianą wyżej, to najlepiej pozbyć się
opcji `DNS` z pliku `/etc/wireguard/wgvpn0.conf` .

### Wyłączenie automatycznej konfiguracji iptables/nftables

Wireguard standardowo sam sobie konfiguruje filtr pakietów `iptables` / `nftables` dodając do niego
własne łańcuchy i określając w nich odpowiednie reguły, np. te od przepisywania adresu źródłowego
przy pomocy `masquerade` . Niemniej jednak, na nasze potrzeby będzie trzeba utworzyć kilka
dodatkowych reguł dla `nftables` i dobrze jest całkowicie wyłączyć dotykanie firewall'a przez
wireguard. Możemy to zrobić dodając do tej standardowej konfiguracji, którą pobraliśmy ze strony
protonvpn, parametr `Table` , przykładowo:

    Table = off

W ten sposób skrypt `wg-quick` całkowicie odpuści sobie konfigurowanie filtra pakietów i zrzuci ten
obowiązek na nas.

### Problematyczne oznaczanie pakietów

By ruch VPN mógł się realizować, trzeba rozdzielić pakiety, które są puszczane w tunel VPN od tych
pakietów, które chcemy wypuścić przez fizyczny interfejs karty sieciowej naszego hosta. Przy
ustawionym parametrze `Table = off` , skrypt `wg-quick` w ogóle nie tyka reguł filtra pakietów
`nftables` , przez co wszystkie pakiety sieciowe domyślnie są kierowane na fizyczny interfejs karty
sieciowej. Gdybyśmy teraz uruchomili wireguard'a, to pakiety mimo podniesionego interfejsu VPN i
tak polecą bezpośrednio w świat.

Trzeba zatem obmyślić własny mechanizm, który będzie stosownie oznaczał ruch VPN, tak by pakiety
sieciowe zostały skierowane w tunel VPN zanim zostaną wysłane w świat. Do tego celu służy mechanizm
oznaczania pakietów w `nftables` , tj. `fwmark` . W pliku konfiguracyjnym
wireguard `/etc/wireguard/wgvpn0.conf` możemy taki mark również ustawić przy pomocy
parametru `FwMark` . Trzeba jednak tutaj wyraźnie zaznaczyć, że ten mark nie odnosi się do pakietów
kierowanych do wnętrza tunelu VPN, tylko do samego połączenia VPN, tj. tych pakietów, które
będziemy wysyłać w świat. Naturalnie ten ruch musimy sobie również oznaczyć, przykładowo:

    FwMark = 0x5555

O ile oznaczenie ruchu samego połączenia VPN mamy z głowy, to trzeba jeszcze pomyśleć jak
zrealizować oznaczenie ruchu pakietów, które mają trafiać do tunelu VPN ale o tym za moment.

## Tablica i reguły routingu na potrzeby wireguard

Jako, że konfiguracja wireguard w takiej postaci, do której ją doprowadziliśmy, ogranicza się
jedynie do ustawienia samego interfejsu sieciowego `wgvpn0` , to trzeba samemu skonfigurować sobie
routing pakietów sieciowych połączenia VPN.

Na sam początek tworzymy nową tablicę routingu w pliku `/etc/iproute2/rt_tables` , przykładowo:

    300 wireguard

Mając tablice routingu, dodajemy jeszcze regułę routingu, której zadaniem będzie skierować
określone pakiety do tej tablicy:

    # ip rule add fwmark 0x2222 priority 2005 table wireguard

Ta reguła ma za zadanie przekierować pakiety oznaczone markiem `0x2222` do
tablicy `wireguard` -- to będą te pakiety, które trzeba zaszyfrować przed wypuszczeniem ich w
świat. Nie tworzymy osobno reguły dla pakietów z markiem `0x5555` , bo one domyślnie i tak będą
trafiać to tablicy `main` . Teraz już wystarczy zająć się konfiguracją wpisów w tablicy `wireguard`
oraz stosownym oznaczaniem pakietów sieciowych.

### Mechanizm kill switch

Przy takiej konfiguracji, jaką my tutaj zrobiliśmy, warto pomyśleć o jakiejś formie kill switch'a,
czyli mechanizmu uniemożliwiającemu wysłanie pakietów z pominięciem VPN, np. w przypadku zniknięcia
z systemu interfejsu `wgvpn0` . Tego typu sytuacja może nam się przytrafić w skutek jakiegoś
nieprzewidzianego błędu lub też gdy intencjonalnie usuniemy ten interfejs zatrzymując usługę
systemd dla wireguard'a ( `wg-quick@wgvpn0.service` ) albo też gdy ręcznie położymy interfejs via
`wg-quick down wgvpn0` lub też będziemy manipulować interfejsami sieciowymi przy pomocy
polecenia `ip` . W żadnym z tych przypadków nie może być mowy o tym, że procesy, których ruch
kierujemy do tunelu VPN, będą przesyłać pakiety bezpośrednio na adres docelowy.

By tego typu mechanizm kill switch zaprojektować, skorzystamy z tablicy routingu, którą
utworzyliśmy wcześniej, tj. `wireguard` . To tej tablicy trzeba dodać wpis z trasą
typu [blackhole][11], przykładowo:

    # ip route append blackhole default table wireguard

W ten sposób, gdy interfejs `wgvpn0` zniknie z systemu, pakiety będą w dalszym ciągu kierowane do
tablicy `wireguard` ale zastaną tam jedynie trasę `blackhole` , przez co będą cicho niszczone przez
kernel bez dalszego ich przetwarzania, czego efektem będzie brak połączenia sieciowego w
aplikacjach puszczonych przez VPN. Wszystkie pozostałe aplikacje jak najbardziej będą mieć dostęp
do sieci i będą mogły bez problemu komunikować się ze światem.

## Ręczna konfiguracja nftables

Przyszła pora na skonfigurowanie filtra pakietów `nftables` . Tutaj musimy zrobić w zasadzie dwie
rzeczy. Pierwszą z nich jest przepisanie źródłowego adresu IP pakietów VPN i w jego miejsce
postawić IP interfejsu karty sieciowej naszej maszyny. Drugą zaś oznaczenie pakietów ruchu
sieciowego puszczonego w tunel VPN.

### Podmiana źródłowego adresu IP (masquerade)

By zmienić źródło pakietów sieciowych ruchu VPN tworzymy w konfiguracji `nftables` tablicę `nat`
podobną do tej poniżej:

    table inet nat {

        chain POSTROUTING {
            # priority 100=srcnat
            type nat hook postrouting priority 100; policy accept;
            oifname "wgvpn0" counter masquerade
        }

    }

### Oznaczanie pakietów sieciowych (fwmark)

Z kolei by oznaczyć pakiety ruchu sieciowego, tworzymy w konfiguracji `nftables` tablicę `mangle` o
poniższej zawartości:

    table inet mangle {

        chain PREROUTING {
            # priority -150=mangle
            type filter hook prerouting priority -150; policy accept;
            iifname != "lo" jump marking
        }

        chain OUTPUT {
            # priority -150=mangle
            type route hook output priority -150; policy accept;
            oifname != "lo" jump marking
        }

        chain marking {
            counter meta mark set ct mark & 0x0000ffff
            meta mark & 0x0000ffff != 0x00000000 counter return
            meta skgid 8888 meta mark & 0x0000ffff == 0x00000000 meta mark set 0x00002222 counter
            counter ct mark set meta mark & 0x0000ffff
        }

    }

Oznaczanie pakietów wyjściowych dokonujemy w łańcuchu `OUTPUT` tablicy `mangle` ze względu na fakt
badania zmiany trasy, co jest kluczowe przy wirtualnych interfejsach tworzonych w systemie na
potrzeby ruchu VPN. Gdybyśmy to oznaczanie ruchu dokonali w łańcuchu `POSTROUTING` , to byłoby już
za późno i pakiety nie zostałyby skierowane do tunelu VPN. Jeśli zaś chodzi o łańcuch `PREROUTING` ,
to oznaczanie tutaj pakietów przychodzących ma głównie za zadanie odtworzenie marka dla konkretnego
połączenia śledzonego przez mechanizm `conntrack` , tak by pakiety ruchu sieciowego VPN powracające
do naszej maszyny mogły być z powodzeniem poprawnie oznaczone i skierowane do interfejsu VPN.

Dla lepszego zrozumienia, poniżej jest fotka obrazująca przepływ pakietów przez filtr w linux'ie:

![nftables-packet-flow-linux-firewall-netdev](/img/2019/04/001-nftables-packet-flow-linux-firewall-netdev.png#huge)

#### Zapętlenie się wireguard'a (routing loop)

Ta powyższa baza markująca działała w moim systemie prawie przez dekadę i np. OpenVPN nie sprawiał
mi żadnych problemów przez cały ten czas. Niemniej jednak, wireguard przy takim markowaniu nie chce
działać. Niby interfejs `wgvpn0` się podnosi i wszystko jest w należytym porządku, nie ma przy tym
żadnych błędów w terminalu czy logu systemowym, no tylko pakiety kierowane do tunelu VPN giną bez
wieści.

[Problem tutaj dotyczy zapętlania się ruchu VPN][10]. By to nieco bardziej zobrazować, prześledźmy
to, co dokładnie system chce zrobić z pakietami sieciowymi. Uruchamiamy sobie przykładową aplikację,
np. firefox z grupa `8888` . Chcemy oznaczyć ruch tej aplikacji w `nftables` markiem `0x00002222` .
W regułach routingu mamy `2005:	from all fwmark 0x2222 lookup wireguard` , czyli pakiety są
kierowane do tablicy routingu `wireguard` , gdzie widnieje wpis `default dev wgvpn0 scope link` i w
ten sposób pakiety sieciowe generowane przez firefox, trafiają na interfejs `wgvpn0` , czyli do
tunelu VPN. Następne pakiety są szyfrowane i opuszczają interfejs `wgvpn0` , a w  `nftables` jest
im przepisywany źródłowy adres IP, na adres IP karty sieciowej (przy pomocy maskarady). Pakiety są
gotowe do puszczenia na interfejs fizyczny karty sieciowej ale zamiast tego, są ponownie wpuszczane
w tunel VPN. Dlaczego? Wygląda na to, że kernel wiąże jakoś połączenie oznaczone
markiem `0x00002222` (firefox) i markiem `0x00005555` (ustawionym w konfiguracji wireguard) i
przepisuje mark `0x00005555` na `0x00002222` i przez to ruch połączenia VPN (tego kierowanego w
świat) ma dokładnie takie samo oznaczenie co ruch kierowany do tunelu VPN. Taki stan rzeczy sprawia,
że kernel dochodzi do wniosku, że trzeba ten pakiet jeszcze raz puścić w tunel VPN i tak w kółko.

Rozwiązaniem tego zapętlania się połączenia VPN jest dodanie na samym początku `chain marking`
dodatkowej reguły, tj. `meta mark 0x00005555 counter return` , co wygląda mniej więcej tak:

    ...
            chain marking {
                meta mark 0x00005555 counter return
                counter meta mark set ct mark & 0x0000ffff
                meta mark & 0x0000ffff != 0x00000000 counter return
                meta skgid 8888 meta mark & 0x0000ffff == 0x00000000 meta mark set 0x00002222 counter
                counter ct mark set meta mark & 0x0000ffff
            }
    ...

Ta dodatkowa reguła sprawia, że wszystkie pakiety posiadające mark `0x00005555` nie będą
przetwarzane przez następne reguły markujące, co rozwiązuje problem z zapętlaniem się VPN.

## Automatyzacja konfiguracji interfejsu VPN

Ostatnie co musimy zrobić, to dodać kilka wpisów `PostUp` do pliku `/etc/wireguard/wgvpn0.conf` ,
tak by konfigurowanie interfejsu `wgvpn0` przebiegało automatycznie. Poniżej znajduje się
ostateczna wersja tego pliku, a niżej wyjaśnienie dodanych linijek:

    [Interface]
    # Key for wgvpn0
    # Bouncing = 1
    # NAT-PMP (Port Forwarding) = off
    # VPN Accelerator = off
    PrivateKey = scisle-tajne-lamane-przez-poufne
    Address = 10.2.0.2/32
    #DNS = 10.2.0.1
    Table = off
    FwMark = 0x5555

    PostUp = if ip route show table wireguard | grep -q blackhole; then ip route prepend default dev %i table wireguard; else ip route add default dev %i table wireguard && ip route append blackhole default table wireguard; fi

    PostUp = if ! ip rule show table wireguard | grep -q "fwmark 0x2222"; then ip rule add fwmark 0x2222 priority 2005 table wireguard; fi

    [Peer]
    # CA-FREE#9
    PublicKey = 7nj3Zh17Dzx+1SKIPE+dPfFmDbTTOggDK6SfK6tlEgE=
    #AllowedIPs = 0.0.0.0/0, ::/0
    AllowedIPs = 0.0.0.0/0
    Endpoint = 149.22.82.1:51820
    PersistentKeepalive = 25

Mamy tutaj dwie linijki `PostUp` . Pierwsza z nich ma za zadanie sprawdzić czy w
tablicy `wireguard` znajduje się trasa typu `blackhole` i jeśli tak, to ma dodać wpis z trasą do
interfejsu naszego VPN. Z tym, że ten wpis musi być przed trasą typu `blackhole` , bo inaczej to
pakiety będą cicho niszczone przez kernel. Jeśli jednak nie ma trasy typu `blackhole` , to trzeba
dodać zarówno trasę do interfejsu VPN, jak i trasę typu `blackhole` .

Druga linijka `PostUp` zaś bada czy w regułach routingu znajduje się wpis odnoszący się do
tablicy `wireguard` , tj. ten który przy pomocy marka `nftables` będzie kierował ruch sieciowy w
tunel VPN. Jeśli takiej reguły nie ma, to ma ją dodać.

Te dwa wpisy `PostUp` sprawiają, że konfiguracja tras i reguł routingu będzie dla nas transparentna
i mamy pewność, że routing zawsze będzie skonfigurowany poprawnie, przynajmniej z naszego punktu
widzenia i to bez względu na to czy połączenie VPN będzie aktywne czy nie.

W przypadku, gdy interfejs VPN nam z jakiegoś powodu zniknie, to bama domyślna powiązana z tym
interfejsem również nam zniknie. Niemniej jednak, reguła z markiem oraz trasa typu `blackhole`
pozostają nieruszone, przez co ruch przy rozłączonym połączeniu VPN będzie cicho niszczony i nic
bezpośrednio w świat nie poleci.

## Testowanie konfiguracji

Plik `/etc/wireguard/wgvpn0.conf` w zasadzie mamy już gotowy i możemy zestawić połączenie VPN przy
pomocy `wg-quick` . Odpalamy zatem terminal i wpisujemy w nim poniższe polecenie:

    # wg-quick up wgvpn0
    [#] ip link add dev wgvpn0 type wireguard
    [#] wg addconf wgvpn0 /dev/fd/63
    [#] ip -4 address add 10.2.0.2/32 dev wgvpn0
    [#] ip link set mtu 1420 up dev wgvpn0
    [#] if ip route show table wireguard | grep -q blackhole; \
           then ip route prepend default dev wgvpn0 table wireguard; \
           else ip route add default dev wgvpn0 table wireguard && \
           ip route append blackhole default table wireguard; \
        fi
    [#] if ! ip rule show table wireguard | grep -q "fwmark 0x2222"; \
           then ip rule add fwmark 0x2222 priority 2005 table wireguard; \
        fi

Wyżej widzimy komendy, które `wg-quick` wydaje. Nie ma przy tym żadnych błędów, zatem zdaje się, że
połączenie VPN działa. Podejrzymy jeszcze status wireguard'a wpisując w terminal polecenie `wg` :

    # wg
    interface: wgvpn0
      public key: SNq4tmGcDoDCZQkm1x9SdXuVs2Qbe+G/bJqGz5eChDE=
      private key: (hidden)
      listening port: 36032
      fwmark: 0x5555

    peer: 7nj3Zh17Dzx+1SKIPE+dPfFmDbTTOggDK6SfK6tlEgE=
      endpoint: 149.22.82.1:51820
      allowed ips: 0.0.0.0/0
      latest handshake: 27 seconds ago
      transfer: 27.16 KiB received, 18.85 KiB sent
      persistent keepalive: every 25 seconds

Jak widać wyżej, `latest handshake: 27 seconds ago` oznacza, że nasza maszyna i ta określona
w `endpoint` były w stanie się ze sobą przywitać i wymienić klucze szyfrujące. Przy dobrze
skonfigurowanym wireguard, te klucze są wymieniane co 2 minuty. Nigdy zatem w zasadzie nie
powinniśmy tutaj widzieć czasu dłuższego niż te 2 minuty. Jeśli tak się dzieje, oznacza to, że coś
w konfiguracji wireguard'a jest nie tak, a maszyny mają problem z komunikacją.

Natomiast w linijce `transfer` mamy `27.16 KiB received` oraz `18.85 KiB sent` . Jeśli te dwie
wartości ulegają zwiększeniu, to pakiety wysyłane przez nasze aplikacje będą w stanie dotrzeć na
adres określony w `endpoint` oraz z niego powrócić. Jeśli by się zdarzyło tak, że żadna z tych
wartości nie rośnie z czasem, to pakiety od ruchu VPN są gdzieś gubione albo blokowane, zwykle na
naszym firewall'u. Jeśli zaś wartość `sent` (dane, które wysyłamy) ulega zwiększeniu ale wartość
`received` pozostaje niezmienna, to zwykle oznacza to problem opisywaną wcześniej pętlą routingu.

### Testowanie tablic i reguł routingu

Możemy teraz sprawdzić jeszcze czy wpisy w tablicy routingu oraz reguły routingu są prawidłowe
wydając w terminalu to poniższe polecenie przy podniesionym interfejsie `wgvpn0` (uruchomionym
wireguard):

    # ip route get 8.8.8.8 mark 0x2222
    8.8.8.8 dev wgvpn0 table wireguard src 10.2.0.2 mark 0x2222 uid 0
        cache

To polecenie służy do przetestowania (bez faktycznego przesyłania pakietów w świat) reguły routingu
w oparciu o mark, który nakładamy na pakiety w `nftables` aplikacjom, których ruch sieciowy
zamierzamy przesyłać przez VPN. Jak widzimy, pakiety kierowane na przykładowy adres `8.8.8.8` ,
które zostaną oznaczone markiem `0x2222` polecą przez interfejs `wgvpn0` , a to za sprawą tablicy
routingu `wireguard` . Wszystkie te pakiety będą mieć przypisany adres źródłowy `10.2.0.2` .
Natomiast `uid 0` określa ID użytkownika w naszym linux'ie, który wygenerował ten ruch. Widzimy
również słówko `cache` , które informuje nas, że do obsługi tej trasy został [wykorzystany cache
kernela][12] (chodzi o FIB Nexthop Caching, nie mylić z routing cache, który został usunięty w
kernelu 3.6).

Jeśli zaś położymy interfejs `wgvpn0` (rozłączymy VPN), to wynik powinien być następujący:

    # ip route get 8.8.8.8 mark 0x2222
    RTNETLINK answers: Invalid argument

Zwrócona wyżej odpowiedź mówi nam, że kernel nie potrafi znaleźć poprawnej trasy dla tak
oznaczonych pakietów, a konkretnie trafiają one we wpis zawierający `blackhole` w
tablicy `wireguard` i stąd komunikat o błędzie.

## Uruchamianie aplikacji puszczanych w tunel VPN

Przyszła pora uruchomić jakaś przykładową aplikację, by sprawdzić czy tunel VPN faktycznie działa.
Do tego celu trzeba sobie w Debianie stworzyć nową grupę i dodać do niej naszego użytkownika,
przykładowo:

    # groupadd -g 8888 forcewg
    # adduser morfik forcewg

W tym momencie musimy zakończyć wszystkie sesje użytkownika, tj. trzeba wylogować się ze wszystkich
terminali, w tym również i z graficznej sesji. Wystarczy zrestartować sesję graficzną ale można też
zrestartować cały system, choć nie jest to konieczne.

Gdy już się przelogowaliśmy, wpisujemy w terminal jedno z tych dwóch poniższych poleceń:

    $ newgrp forcewg -c "curl -s https://ifconfig.me"
    149.22.82.3

    $ sg forcewg "curl http://ifconfig.me"
    149.22.82.3

Jeśli adres IP zwrócony tutaj różni się od tego, który przypisał nam ISP, to pakiety sieciowe
procesów uruchomionych z grupa `forcewg` poszły w tunel VPN, a nie bezpośrednio w świat. Jeśli
teraz byśmy położyli interfejs `wgvpn0` i ponownie wydali te powyższe polecenia, to wynik powinien
być następujący:

    # wg-quick down wgvpn0
    [#] ip link delete dev wgvpn0

    $ sg forcewg "cgexec http://ifconfig.me"
    curl: (28) Failed to connect to ifconfig.me:80 after 133566 ms: Could not connect to server

Możemy także posłużyć się poleceniem `ping` , by sprawdzić czy ruch zamiera na czas wyłączenia VPN.

## Podsumowanie

Jak widać, wireguard nie jest taki straszny i konfiguracja VPN przy jego pomocy może i trochę się
różni od tej znanej z OpenVPN ale też nie jest to nic z czym przeciętny śmiertelnik nie mógłby
sobie poradzić. Tutaj trudność polega głównie na realizacji zadania przesyłania ruchu
sieciowego tylko wybranych aplikacji/procesów przez VPN w stosunku do całego ruchu sieciowego
generowanego przez nasz komputer. W standardowych konfiguracjach wireguard nie trzeba tych
dodatkowych kroków przeprowadzać, a całe zadanie zestawienia połączenia upraszcza się w zasadzie do
pobrania jednego pliku od operatora VPN i wydaniu polecenia `wg-quick` . Niemniej jednak, warto się
zatroszczyć również o ruch DNS, by go nie przesyłać przez połączenie VPN tylko osobno szyfrować
oraz też dobrze jest sobie zaimplementować mechanizm kill switch, który uniemożliwi wysłanie
pakietów bezpośrednio w świat na wypadek, gdyby z jakiegoś powodu interfejs VPN nam znikł z systemu.


[1]: /post/jak-zmusic-jeden-proces-do-korzystania-z-vpn-na-linux-openvpn/
[2]: https://www.wireguard.com/quickstart/
[3]: https://www.wireguard.com/
[4]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[5]: https://protonvpn.com/
[6]: https://account.protonvpn.com/downloads
[7]: https://dnsleaktest.com/what-is-a-dns-leak.html
[8]: /post/szyfrowany-dns-z-dnscrypt-proxy-i-dnsmasq-na-debian-linux/
[9]: https://www.cloudflare.com/learning/ssl/what-is-encrypted-sni/
[10]: https://serverfault.com/questions/1147215/prevent-routing-loop-with-fwmark-in-wireguard/1147291#1147291
[11]: https://linux.die.net/man/8/ip
[12]: https://thermalcircle.de/doku.php?id=blog:linux:routing_decisions_in_the_linux_kernel_2_caching
