---
author: Morfik
categories:
- OpenWRT
date: "2016-05-15T13:37:08Z"
date_gmt: 2016-05-15 11:37:08 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
- failover
- load-balancing
GHissueID: 471
title: Failover i load balancing w OpenWRT (mwan3)
---

Może się zdarzyć tak, że będziemy mieli kiedyś dostęp do łącz kilku różnych providerów
internetowych. Jeśli chcielibyśmy skorzystać z internetu w takiej sytuacji, to trzeba by się
zdecydować na jednego z tych dostępnych ISP. Natomiast łącze pozostałych ISP będzie niewykorzystane
w tym danym momencie, a przecie nie za to im płacimy. Jeśli mamy router z OpenWRT i
[skonfigurowaliśmy przy tym switch tak, by mieć kilka portów
WAN](/post/podzial-switcha-na-kilka-vlan-w-openwrt/), to możemy korzystać z usług
wielu ISP w tym samym czasie. Oczywiście, ten mechanizm działa również w przypadku, gdy ISP świadczy
nam usługi za pomocą technologi LTE. Trzeba tylko odpowiednio [skonfigurować modem USB do pracy na
routerze](/post/modem-lte-pod-openwrt/). W tym artykule zostanie opisane [narzędzie
mwan3](https://wiki.openwrt.org/doc/howto/mwan3), za pomocą którego zaprojektujemy sobie prosty
failover (łącze awaryjne) lub load balancing (równoważenie ruchu) mając do wykorzystania dwóch
różnych ISP.

<!--more-->
## Parametry i konfiguracja internetu od ISP

W tym przypadku będziemy mieli do czynienia z jednym ISP lokalnym, który oferuje łącze 15/1 mbit/s.
Drugim ISP jest operator GSM RBM/Play, który świadczy usługę internetu LTE. W przypadku lokalnego
ISP, dostęp odbywa się po kablu wpiętego do standardowego portu WAN routera [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html). Natomiast jeśli chodzi o internet
LTE, to jest on realizowany przez modem E3372 (E3372s-153) w wersji NON-HiLink podłączony do jednego
z portów USB routera. Parametry tego łącza są bardzo zróżnicowane. Wahają się od 30/30 mbit/s do
3/15 mbit/s w szczycie. Zakładając scenariusz idealny, to będziemy mieli do wykorzystania 45/31
mbit/s po zaimplementowaniu mechanizmu równoważącego ruch. Natomiast jeśli chodzi o failover, to w
przypadku zaniku sygnału od lokalnego ISP, zostanie aktywowany internet LTE. Po powrocie sygnału,
nastąpi przełączenie z powrotem na łącze lokalnego ISP.

Warto tutaj zauważyć, że w przypadku równoważenia obciążenia, czyli gdy wykorzystujemy łącza obu ISP
jednocześnie, maksymalny transfer dla pojedynczego połączenia nigdy nie przekroczy wartości
granicznych danego ISP. Dla przykładu, jeśli pobieramy jeden plik z internetu, to poleci on
pierwszym lub drugim kanałem. W rezultacie, transfer jaki osiągniemy to 15 lub 30 mbit/s, w
zależności od ISP. Jeśli natomiast uruchomimy kilka jednoczesnych połączeń przy pobieraniu tego
pliku, wtedy część pliku będzie pobierana za pomocą łącza jednego ISP, a pozostała część za pomocą
drugiego łącza. W przypadku stron www, każde wejście na jakiś serwis, generuje wiele jednoczesnych
połączeń. Zatem będą one wysyłane do różnych ISP i tu już boost powinien być widoczny gołym okiem.

Minimalna konfiguracja, jaka będzie nam potrzebna, to trzy interfejsy: 1 LAN i 2 WAN. Musimy także
upewnić się, że nie korzystamy z DNS żadnego z w/w ISP. Chodzi o to, że jeśli byśmy korzystali z
serwera DNS jednego z tych providerów, to po przełączeniu się na drugie łącze, w przypadku awarii
pierwszego lub zwyczajnie podczas balansowania ruchu, zapytania DNS będą blokowane przez któregoś z
ISP. No bo będą przecie pochodzić spoza jego sieci. Dlatego musimy korzystać z globalnych providerów
DNS, np. google czy OpenDNS. Podobnie z pozostałymi usługami, np. email. Nie możemy korzystać z nich
jeśli zamierzamy robić load balancing czy network failover. Poniżej znajduje się konfiguracja
interfejsów sieciowych routera.

Plik `/etc/config/network` :

    config interface 'lan'
       option ifname        'eth1'
       option force_link    '1'
       option type          'bridge'
       option proto         'static'
       option ipaddr        '192.168.1.1'
       option netmask       '255.255.255.0'
       option ip6assign     '60'
       option peerdns       '0'

    config interface 'lte'
       option ifname    'wwan0'
        option proto    'ncm'
        option mode     'lte'
        option pincode  ''
        option apn      'internet'
        option device   '/dev/ttyUSB0'
        option username ''
        option password ''
        option peerdns  '0'
        option dns      '208.67.220.220 208.67.222.222'

    config interface 'wan'
       option ifname  'eth0'
       option proto   'dhcp'
       option peerdns '0'
       option dns     '208.67.220.220 208.67.222.222'

Wszystkie interfejsy WAN muszą być wyjęte spod działania serwera DHCP w OpenWRT. Dlatego też w pliku
`/etc/network/dhcp` musimy uwzględnić te poniższe wpisy:

    config dhcp 'wan'
       option interface 'wan'
       option ignore '1'

    config dhcp 'lte'
       option interface 'lte'
       option ignore '1'

Musimy także skonfigurować przepływ pakietów na zaporze. Standardowy interfejs WAN w domyślnej
konfiguracji OpenWRT już jest poprawnie skonfigurowany. Dlatego poniżej znajdują się jedynie wpisy
dotyczące nowego interfejsu sieciowego:

    config zone
       option name             lte
       list   network          'lte'
       option input            DROP
       option output           ACCEPT
       option forward          DROP
       option masq             1
       option conntrack        1
       option mtu_fix          1
       option log              0
       option log_limit        60/minute

    config forwarding
       option src              'lan'
       option dest             'lte'

Zapisujemy konfigurację i restartujemy router. Po podniesieniu się systemu, router powinien uzyskać
konfigurację na obu interfejsach WAN. Niemniej jednak, obecnie mamy określoną tylko jedną bramę
domyślną. Dlatego też pakiety do internetu mogą lecieć tylko przez jednego z tych dwóch operatorów.
Którego? To zależy jak się te interfejsy skonfigurują po starcie routera. Zwykle konfiguracja tego
ostatniego interfejsu WAN przepisze bramę domyślną w tablicy routingu. I to on będzie wykorzystywany
do połączenia z internetem. W przypadku LTE, to modem zawsze konfiguruje się kilka sekund. Dlatego
to on na poniższym listingu określił swoją bramę jako domyślną:

![wwan3-openwrt-tablica-routingu-brama](/img/2016/05/2.wwan3-openwrt-tablica-routingu-brama.png#huge)

Na powyższej fotce widzimy tablicę routingu dla interfejsów routera. Mamy tam 3 interfejsy: `wwan0`
(LTE), `eth0` (WAN), oraz `br-lan` (LAN). Mamy zatem na trzech interfejsach skonfigurowane trzy
różne sieci: 10.136.111.216/30 (brama 10.136.111.218), 82.160.124.0/24 (brama 82.160.124.254) i
192.168.1.0/24 (bez bramy). Jako brama domyślna jest ustawiony adres 10.136.111.218 na interfejsie
`wwan0` . Zatem wszystkie pakiety, które nie są przeznaczone do tych 3 sieci, będą przesyłane przez
interfejs `wwan0` na adres 10.136.111.218 .

Konfiguracja interfejsów WAN routera wygląda zaś następująco:

![wwan3-openwrt-interfejsy-wan](/img/2016/05/1.wwan3-openwrt-interfejsy-wan.png#huge)

### Metryki

Jeśli chcemy korzystać z obu tych ISP jednocześnie, to musimy nakazać routerowi skonfigurowanie
bramy domyślnej ( `default` ) zarówno dla interfejsu `eth0` jak i `wwan0` . By móc tego typu zabieg
przeprowadzić, musimy określić metryki dla tras wszystkich interfejsów WAN. Domyślnie mają one
sprecyzowane 0 i możemy w systemie posiadać tylko jedną domyślną bramę. Z tego też względu są one
przepisywane przy konfiguracji następnego interfejsu WAN. Edytujemy zatem jeszcze raz plik
`/etc/config/network` i do bloków z konfiguracją WAN dodajemy opcję `metric` :

    config interface 'wan'
       option ifname 'eth0'
       option metric   '10'
       ...

Im niższą wartość ustawimy w opcji `metric` , tym brama ma większy priorytet. W ten sposób określimy
pożądaną przez nas kolejność bramek domyślnych. Niemniej jednak, W wydaniu OpenWRT Chaos Calmer nie
zawsze możemy określić metrykę w powyższy sposób. Dla przykładu, wykorzystywany tutaj modem pracuje
w trybie NDIS (NCM) i obecnie pakiet `comgt-ncm` nie ma zaimplementowanej obsługi metryk w pliku
`/lib/netifd/proto/ncm.sh` . Trzeba by ten plik przerobić. [Pod tym
linkiem](https://pastebin.com/iE64TT5a) znajduje się już gotowy plik, którego zawartość trzeba
przekopiować do w/w lokalizacji. W późniejszych wersjach OpenWRT nie będzie już trzeba tak
kombinować. W każdym razie, ten problem z metrykami dotyczy tylko modemów LTE. Wszelkie połączenia
przewodowe konfiguruje się tak jak widać powyżej.

Mając już przypisane metryki, restartujemy interfejsy. Sprawdzamy tablicę routingu oraz testujemy
połączenie za pomocą polecenia `ping` :

![wwan3-test-lacza](/img/2016/05/4.wwan3-test-lacza.png#huge)

Jak widzimy, bramy domyślne są dwie. Po jednej dla każdego interfejsu WAN. Również `ping` przechodzi
przez oba interfejsy, czyli internet jest osiągalny przez obu ISP. Taka konfiguracja otwiera nam
drogę do zainstalowania i skonfigurowania pakietu `mwan3` . To właśnie przy jego pomocy zrobimy
sobie failover i load balancing łącza.

## Instalacja i konfiguracja mwan3

Jeśli do tego miejsca udało nam się uzyskać wyniki podobne do powyższych i po zresetowaniu routera
mamy internet, który wychodzi przez główną bramę (tę z niższą metryką), oznacza to, że możemy
przejść do instalowania pakietu `mwan3` . Jest on dostępny standardowo w repozytoriach OpenWRT i
jego instalacja sprowadza się do wydania tych poniższych poleceń:

    # opkg update
    # opkg install mwan3

Konfiguracja `mwan3` znajduje się w pliku `/etc/config/mwan3` . Jako, że my dysponujemy tylko dwoma
interfejsami, to za dużo nie trzeba w tym pliku zmieniać. Całość działa, można by rzec, OOTB.
Przechodzimy zatem do edycji w/w pliku i włączamy w nim sekcję odpowiedzialną za drugi interfejs
WAN:

    config interface 'lte'
        option enabled '1'
        ...

Po resecie routera, oba łącza powinny zostać aktywowane i całość powinna działać. Oczywiście, nie
musimy resetować routera, by całość nam po prostu zadziałała. Możemy wpisać w terminalu to poniższe
polecenie:

    # mwan3 start

Plik konfiguracyjny `mwan3` zawiera dość sporo parametrów i przydałoby się je omówić. Opcja
`interface` wskazuje nazwę interfejsu, która jest określona w pliku `/etc/config/network` . W tym
przypadku są to `wan` oraz `lte` . Opcja `enabled` określa czy `mwan3` ma brać pod uwagę ten
interfejs. Dalej mamy pozycje `list track_ip` , które definiują adresy hostów, w oparciu o które
będzie przeprowadzany test łącza. Można sprecyzować kilka takich opcji. Można też ten parametr
całkowicie pominąć i w takim przypadku interfejs będzie uznawany zawsze za działający. Kolejna
opcja w pliku to `reliability` i przyjmuje ona wartość liczbową określającą ile hostów, z tych
sprecyzowanych w `list track_ip` musi udzielić odpowiedzi, by traktować dany interfejs jako
działający. Następnie mamy `count` , który określa ilość żądań `ping` wysyłanych do hostów podczas
testów. Z kolei `timeout` definiuje ile czasu (w sekundach) system będzie czekał na odpowiedź na
wysłane żądanie ping'u, po czym da sobie spokój i zgłosi błąd. Opcja `interval` określa
częstotliwość próbkowania łącza, czyli co ile sekund zostanie wysłany `ping` . Ostatnie dwie opcje
w bloku `config interface` , tj. `up` oraz `down` definiują ile testów musi przejść interfejs, by
został uznany odpowiednio za działający lub padnięty. Dobrze jest przemyśleć dobór wartości w
powyższych parametrach, by czasem interfejs nie był zbyt często uznawany przez pomyłkę za trupa.
Podobnie trzeba rozważyć podniesienie niektórych wartości w przypadku łącz o słabszej jakości.

Kolejne dwie sekcje w pliku, tj. `config member` i `config policy` są ze sobą powiązane. To tutaj,
na podstawie metryk ( `metric` ) i wag ( `weight` ) będzie zapadać decyzja o konfiguracji
przesyłanego ruchu. Wymagane jest utworzenie co najmniej dwóch sekcji `config member` do
prawidłowego działania `mwan3` . Chodzi o podział sumarycznego łącza. Jeśli są dwie sekcje, to mamy
podział na dwie części (dwóch providerów ) w proporcjach określanych przez powyższe parametry.
Domyślnie mamy określone cztery sekcje `config member` . W każdej z nich są zdefiniowane różne
metryki. Im mniejszy numer tym większy priorytet. W blokach `config policy` będą określone różne
polityki zachowań się łącza w przypadku, gdy zostaną spełnione określone warunki. Na podstawie
sprecyzowanych metryk będzie podejmowana decyzja czy używać jednego łącza czy też drugiego. W
przypadku, gdy metryka ma taki sam numerek na obu interfejsach, wtedy nastąpi podział łącza w
stosunku określonym przez parametr `weight` , w tym przypadku 2/3 czyli 40%/60%.

Jako, że mamy 2 interfejsy WAN, możemy ustawić następujące polityki zachowań:

  - Korzystamy tylko z pierwszego interfejsu WAN.
  - Korzystamy tylko z drugiego interfejsu WAN.
  - Korzystamy z obu interfejsów do równoważenia obciążenia.
  - Preferujemy korzystanie z pierwszego interfejsu WAN. Gdy ten zawiedzie, pakiety lecą przez
    drugi interfejs WAN.
  - Preferujemy korzystanie z drugiego interfejsu WAN. Gdy ten zawiedzie, pakiety lecą przez
    pierwszy interfejs WAN.

Wszystkie te powyższe przypadki zostały ujęte w bloki `config policy` . Zdefiniowanie polityki nic
jeszcze nie oznacza. Trzeba do nich dorobić reguły, a te są określane przez zwrotki z `config
rule` . Reguły można aplikować do portów źródłowych ( `src_port` ) i docelowych ( `dest_port` ),
adresów źródłowych ( `src_ip` ) i docelowych ( `dest_ip` ) oraz do protokołów ( `proto` ).
Kolejność w jakiej zostały określone reguły ma znaczenie. Dlatego też najlepiej na samym końcu
umieścić domyślną regułę, a powyżej niej te bardziej specyficzne.

Jeśli chcemy, aby nasz router zachowywał się jak load balancer, czyli równoważył ruch między dwóch
różnych ISP, to w pliku `/etc/config/mwan3` ustawiamy poniższą regułę:

    config rule 'default_rule'
        option dest_ip '0.0.0.0/0'
        option use_policy 'balanced'

W opcji `use_policy` podajemy jedną z nazw określonych w blokach `config policy` . Jeśli do tego
chcemy, by ruch z/do jednego hosta z naszej sieci zawsze szedł przez interfejs `wwan0` , to dodajemy
poniższy blok:

    config rule 'host_wan2'
        option src_ip '192.168.1.150'
        option use_policy 'lte_wan'

W ten sposób możemy decydować o tym gdzie przesłać określony ruch przechodzący przez router, np. gdy
chcemy posłać protokół `https` i `http` przez jednego ISP, a resztę, np. torrenty, przez drugiego. W
każdej regule może być określony jeden protokół, jeden port źródłowy i docelowy, oraz jeden adres
źródłowy i docelowy. W przypadku portów i adresów można także określić ich zakres. Jeśli chcemy
zdefiniować wszystkie protokoły, to trzeba je opisać jako `all` .

## Status łącza

Utworzenie reguł to jedna sprawa. Sprawdzenie czy one w ogóle działają, to osobna kwestia. Na
szczęście, `mwan3` jest nam w stanie zwrócić aktualny status łącza. W nim możemy zobaczyć czy
interfejsy są podniesione, czy też któryś z nich ma awarię:

![wwan3-status](/img/2016/05/5.wwan3-status.png#medium)

Mamy także rozpisaną politykę:

![wwan3-status](/img/2016/05/6.wwan3-status.png#medium)

Oraz reguły:

![wwan3-status](/img/2016/05/7.wwan3-status.png#huge)

Z powyższych informacji wynika, że mam dwa interfejsy podzielone w proporcji 2/3. Oba interfejsy są
aktywne i działają. Ruch na port 443/tcp oraz 80/tcp jest kierowany według zdefiniowanych wyżej
reguł. Natomiast w przypadku pozostałego ruchu mamy zastosowany failover. Czyli w przypadku awarii
interfejsu `eth0` , zostaniemy przełączeni na interfejs `wwan0` i ruch poleci drugim kanałem.
