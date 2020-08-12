---
author: Morfik
categories:
- Linux
date: "2019-02-10T08:22:55Z"
published: true
status: publish
tags:
- debian
- sysctl
- ipv6
title: Jak włączyć IPv6 Privacy Extensions w Debianie (SLAAC)
---

Protokół IPv6 został opracowany już dość dawno temu, a jednak ilość hostów w internecie
komunikujących się za jego pomocą wciąż nie jest zbyt wysoka
i [oscyluje w granicach 25%](https://www.google.com/intl/en/ipv6/statistics.html). Faktem jest, że
migracja z IPv4 na IPv6 może być sporym kosztem dla niektórych podmiotów jeśli chodzi o kwestię
związaną z wymianą sprzętu i ze zmianą konfiguracji sieci, co pewnie zniechęca część ISP do
wdrożenia tego protokołu. Użytkownicy korzystający z sieci z kolei nie wiedzieć czemu też
preferują IPv4 nad IPv6. Jakiś czas temu czytałem nawet artykuł na temat zagrożenia prywatności
jakie może nieść ze sobą protokół IPv6. Chodzi generalnie o to, że obecnie wszyscy przywykliśmy do
rozwiązania jakie oferuje nam NAT, które jest w stanie utrudnić nieco naszą identyfikację i analizę
naszej aktywności w internecie. W przypadku IPv6 adresy IP są dość unikatowe w skali globalnej, a
część odpowiedzialna za identyfikację hosta (ostatnie 64 bity) stanowi identyfikator EUI64, który z
kolei jest generowany na podstawie adresu MAC karty sieciowej. W taki oto sposób interfejs tej
karty będzie miał stały identyfikator EUI64, a hosta będzie można zidentyfikować bez problemu i
bez względu na to u którego ISP podłączymy nasz komputer. Rozwiązaniem tego problemu jest mechanizm
zwany IPv6 Privacy Extensions. Przydałoby się zatem rzucić na niego okiem i jeśli okaże się
użyteczny, to wypadałoby go włączyć w naszym Debian Linux.

<!--more-->
## SLAAC i identyfikator EUI64

SLAAC (StateLess Address AutoConfiguration) to mechanizm automatycznej adresacji węzłów na
podstawie adresów MAC ich kart sieciowych. Po podniesieniu interfejsu sieciowego zdolnego
obsługiwać protokół IPv6, system musi automatycznie przypisać mu co najmniej jeden lokalny adres
dla łącza (link-local address) w postaci `FE80::/10` (spotykany też zapis `FE80::/64` ).
Pierwsze 54 bity adresu za 10-bitowym prefiksem "FE80" mają zawsze wartość "0", zaś następne 64
bity to identyfikator hosta (zwykle EUI64). W ten sposób interfejs zawsze będzie posiadał unikalny
adres IP, za pomocą którego maszyna będzie w stanie komunikować się z hostami w sieci będącymi w
zasięgu pojedynczego hop'a (i to bez serwera DHCP).

Gdy taki host zostanie podłączony do sieci, to wykorzystując mechanizm `Neighbor Discovery` oraz
adresy multicastowe zaczyna nasłuchiwać komunikatów `Router Advertisement` od routerów, które
mogą zawierać prefiksy sieci oraz inne parametry umożliwiające konfigurację adresacji IPv6. Przy
pomocy komunikatów `Router Solicitaion` host może wymusić na lokalnym routerze rozgłoszenie
prefiksu i pozostałych parametrów sieci. Po odebraniu komunikatu RA, system urządzenia konfiguruje
na interfejsie sieciowym kolejny adres IPv6, który składa się z 64-bitowego prefiksu sieci
(dostarczonego przez router) oraz z 64-bitowego identyfikatora hosta. Adres skonfigurowany przy
pomocy mechanizmu SLAAC jest adresem o zasięgu globalnym i umożliwia hostowi komunikację z
dowolnym adresem w sieci internet.

Ten 64-bitowy identyfikator EUI64 jest generowany na podstawie 48-bitowego (lub 64-bitowego)
adresu MAC karty sieciowej. Gdy mamy do czynienia z 48-bitowym adresem MAC, to w środku tego
adresu dodawane jest 16 bitów o wartości `0xFFFE` . Dla przykładu, jeśli karta sieciowa ma MAC
`34:56:78:9A:BC:DE` , to ten identyfikator przepisywany jest pierw do postaci
`3456:78FF:FE9A:BCDE` , a następnie przekształcany jest jego pierwszy najbardziej znaczący oktet.
W tym przypadku jest to `34` (wartości `3` i `4` , bo zapis HEX), gdzie trzeba zanegować
Uniwersalny/Lokalny bit (Universal/Local bit), czyli przepisać jego wartość z `0` na `1` lub z
`1` na `0` . Bit U/L to siódmy najstarszy bit (licząc od lewej strony), zatem przekształcenie tego
oktetu polega na zwiększeniu lub zmniejszeniu jego wartości o `2` , co wygląda mniej więcej tak:

             34 -> 36
    0011 01[0]0 -> 0011 01[1]0

Poniżej jeszcze trzy przykłady dla lepszego zrozumienia:

             00 -> 02
    0000 00[0]0 -> 0000 00[1]0

             3c -> 3e
    0011 11[0]0 -> 0011 11[1]0

             e6 -> e4
    1110 01[1]0 -> 1110 01[0]0

Zatem cały identyfikator EUI64 dla interfejsu sieciowego mającego adres MAC `34:56:78:9A:BC:DE` ma
postać `3656:78FF:FE9A:BCDE` , a adres IPv6, np. `2001:DB8:1:2:3656:78FF:FE9A:BCDE` . Jako, że ten
identyfikator jest zawsze stały dla danej karty sieciowej (czy jej interfejsu sieciowego), to
jest on w stanie identyfikować konkretną maszynę w internecie (zmieniane są tylko pierwsze 64
bity), co z kolei może budzić pewne obawy jeśli chodzi o kwestię prywatności.

## Randomizacja identyfikatora EUI64

Możemy jednak postarać się, by
ten [identyfikator EUI64 pochodzący od MAC był generowany losowo](https://tools.ietf.org/html/rfc3041).
Mało tego, możemy sobie dostosować czas życia takiego wygenerowanego adresu IPv6, co z kolei może
znacznie poprawić naszą prywatność, np. by nam ten adres zmieniał się co godzinę. Oczywiście żadne
połączenia z racji zmiany adresu nie zostaną przerwane. Taki adres, który wygaśnie, będzie w
dalszym ciągu w użytku ale tylko dla już nawiązanych połączeń. Wszystkie nowe połączenia będą
korzystać z nowego adresu. Poniżej przykład wyjścia polecenia `ip` :

    # ip -6 addr show eth0
    4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qlen 1000
    inet6 2a01:a22:7a21:ccaa:a2c1:1a21:ccb2:a8fa/64 scope global temporary dynamic
    valid_lft 63122sec preferred_lft 12123sec
    inet6 2a01:a22:7a21:ccaa:16af:ffa7:a980:a2b4/64 scope global temporary deprecated dynamic
    valid_lft 63122sec preferred_lft 0sec
    inet6 2a01:a22:7a21:ccaa:ef26:10da:1020:ff2a/64 scope global temporary deprecated dynamic
    valid_lft 63122sec preferred_lft 0sec
    inet6 2a01:a22:7a21:ccaa:3e4a:92ff:fe00:4c5b/64 scope global dynamic
    valid_lft 63122sec preferred_lft 12123sec
    inet6 fe80::3e4a:92ff:fe00:4c5b/64 scope link
    valid_lft forever preferred_lft forever

Mamy tutaj przypisanych 5 adresów IPv6 -- jeden o zakresie linku lokalnego ( `scope link` ) i
cztery o zakresie globalnym ( `scope global` ), z czego trzy adresy globalne są tymczasowe
( `temporary` ). Widzimy też, że dwa adresy tymczasowe straciły już ważność ( `deprecated` ) . Do
komunikacji z internetem używane są wszystkie adresy mające zakres globalny, nawet te, które
straciły ważność. Jeśli adres ma `deprecated` to może on być wykorzystywany jedynie w przypadku
połączeń już nawiązanych. Preferowany czas życia adresu wskazywany w `preferred_lft` określa przez
jaki okres czasu taki interfejs będzie wykorzystywany przy nawiązywaniu nowych połączeń. Te adresy,
które straciły ważność mają `preferred_lft=0` . Obok z kolei widnieje `valid_lft` i on informuje
nas o czasie, po upływie którego ten adres zostanie usunięty z interfejsu, co będzie skutkować
przerwaniem wszystkich połączeń wykorzystujących ten adres. Warto też zwrócić uwagę, na fakt, że
identyfikator EUI64 w przypadku adresu o zakresie linku lokalnego jest dokładnie taki sam jak ten
przypisany do tego nietymczasowego globalnego adresu IPv6.

Skoro korzystamy z tymczasowych adresów IPv6, to czy ten nasz faktyczny adres IPv6 nie powinien
zostać usunięty z interfejsu? Nie. Ten adres będzie używany w przypadku, gdy ktoś będzie chciał się
z nami połączyć, np. w przypadku hostowania przez nas serwera gier -- nie będziemy musieli za
każdym razem podawać nowego adresu, np. gdy będziemy chcieli sobie pograć z przyjaciółmi. Zatem
każdy będzie w stanie się z nami połączyć po tym adresie, ale gdy to my będziemy się łączyć z kimś,
to będą wykorzystywane adresy tymczasowe i tych dwóch adresów nie da się ze sobą w żaden sposób
powiązać.

## Jak włączyć IPv6 Privacy Extensions

Na Debianie IPv6 Privacy Extensions można włączyć w kernelu Linux za sprawą `sysctl` . Interesują
nas w zasadzie trzy parametry: `use_tempaddr`  ,  `temp_valid_lft` oraz `temp_prefered_lft` .
Dodajmy zatem poniższy kod do pliku `/etc/sysctl.conf` :

    net.ipv6.conf.all.use_tempaddr = 2
    net.ipv6.conf.default.use_tempaddr = 2
    net.ipv6.conf.all.temp_valid_lft = 43200
    net.ipv6.conf.default.temp_valid_lft = 43200
    net.ipv6.conf.all.temp_prefered_lft = 3600
    net.ipv6.conf.default.temp_prefered_lft = 3600

Jeśli chodzi o parametr `use_tempaddr` , to może on przyjąć wartości `0` , `1` lub `2` . W
przypadku ustawienia `0` , IPv6 Privacy Extensions zostaną wyłączone i jest to domyślna wartość. Z
kolei `1` i `2` włączają IPv6 Privacy Extensions, z tym, że w przypadku `1` będą preferowane
publiczne adresy nad tymi tymczasowym, natomiast w przypadku `2` będą preferowane tymczasowe adresy
ponad tymi publicznymi. Nie wiem jaki jest sens ustawienia tutaj wartości `1` . Interfejs pętli
zwrotnej (loopback) oraz interfejsy punkt-punkt (point-to-point), mają ustawione `-1` ale ta
wartość jest równoznaczna z wyłączeniem tego mechanizmu dla tych interfejsów.

Parametry `temp_valid_lft` oraz `temp_prefered_lft` określają czas życia i preferowany czas życia
adresu IPv6. Wartości domyślne to odpowiednio `604800` (7 dni) i `86400` (1 dzień). Wartości są
naturalnie w sekundach.

Nie zaleca się ustawiać zbyt niskich czasów życia adresów IPv6, bo może zostać przekroczony limit
adresów, które można przypisać (automatycznie) pojedynczemu interfejsowi sieciowemu w systemie. Gdy
limit zostanie osiągnięty, to ten najstarszy adres zostanie od razu usunięty, by zrobić miejsce
nowemu. Domyślnie może być tych adresów `16` . Jeśli potrzebujemy więcej, to możemy zwiększyć tę
liczbę przez określenie poniższych parametrów `sysctl` :

    net.ipv6.conf.all.max_addresses = 32
    net.ipv6.conf.default.max_addresses = 32

Możemy także ten limit wyłączyć całkowicie ustawiając tutaj `0` ale nie jest to dobrym pomysłem, bo
możemy sobie powiesić kernel, gdy tych adresów zostanie utworzonych dość sporo.

Czasami też kernel może mieć problemy z wygenerowaniem nowego adresu tymczasowego dla danego
interfejsu sieciowego, np. w skutek detekcji zduplikowanego identyfikatora EUI64/adresu IPv6. Gdy
taka sytuacja wystąpi, to kernel będzie próbował parokrotnie ten adres wygenerować zanim da sobie
spokój. My możemy za to dostosować ilość podjętych przez kernel prób (domyślnie `3` ):

    net.ipv6.conf.all.regen_max_retry = 5
    net.ipv6.conf.default.regen_max_retry = 5

Jak widać, ten protokół IPv6 nie jest taki straszny jakby się pewnym użytkownikom mogło wydawać. Po
zaaplikowaniu tych powyższych parametrów, IPv6 jest nam w stanie zapewnić nawet większą prywatność
niż NAT IPv4 i dlatego też nie powinniśmy się bać go używać.
