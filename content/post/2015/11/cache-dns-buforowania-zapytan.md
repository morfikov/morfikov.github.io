---
author: Morfik
categories:
- Linux
date:    2015-11-16 20:13:06 +0200
lastmod: 2015-11-16 20:13:06 +0200
published: true
status: publish
tags:
- dns
- cache
- dnsmasq
- systemd
- resolver
GHissueID: 261
title: Cache DNS, czyli włączenie buforowania zapytań
---

Większość z nas wie, że standardowe instalacje systemu linux nie buforują żadnych zapytań do
serwerów DNS. Dzieje się tak dlatego, że te systemy domyślnie nie mają zainstalowanego żadnego
oprogramowania, które by im to umożliwiało. Niesie to ze sobą zwiększenie opóźnień transakcji
krótkoterminowych, np. tych w protokole http czy https. Za każdym razem gdy odwiedzamy jakiś serwis
www, musimy wykonać szereg zapytań DNS, by rozwiązać nazwy domen na adresy IP. W przypadku gdybyśmy
mieli cache DNS, to te nazwy nie musiałyby być za każdym każdym razem rozwiązywane na nowo,
przynajmniej nie przez odpytywanie zdalnego serwera DNS, do którego RTT wynosi jakieś 20-40ms.
Przydałoby się zatem nieco poprawić wydajność stron www i w tym wpisie postaramy się zaimplementować
prosty cache DNS z wykorzystaniem [narzędzia dnsmasq][1].

<!--more-->
## Konfiguracja dnsmasq

Z `dnsmasq` spotkaliśmy się już przy okazji [konfiguracji AP WiFi (Access Point) pod debianem][2].
Poniższy wpis nie będzie miał jednak nic wspólnego z sieciami bezprzewodowymi. Dlatego też dla tych,
który nie są zaznajomieni z tym narzędziem, warto nadmienić, że `dnsmasq` realizuje zdania serwera
DHCP, oraz ma także możliwość obsługi zapytań DNS i potrafi także te zapytania buforować. Obie te
funkcjonalności (DHCP i DNS) są niezależne i w tym przypadku serwer DHCP nie będzie nam do niczego
potrzebny. Tak czy inaczej, instalujemy pakiet `dnsmasq` i przechodzimy od razu do konfiguracji
pliku `/etc/dnsmasq.conf` . Umieszczamy w nim te poniższe wpisy:

    domain-needed
    bogus-priv
    no-resolv
    server=127.0.2.1#5353
    server=/pool.ntp.org/208.67.222.222
    interface=bond0
    listen-address=127.0.0.1
    bind-interfaces
    no-dhcp-interface=bond0
    no-hosts
    cache-size=10000
    min-cache-ttl=600
    max-cache-ttl=3600
    dns-forward-max=500
    no-negcache

Opcje `domain-needed` oraz `bogus-priv` odfiltrowują zapytania, na które nie potrafią odpowiedzieć
publiczne serwery DNS. Pierwsza z nich nie forwarduje nazw bez kropek (bez części domeny), zaś druga
nie zezwala na przekazywanie adresów z prywatnej przestrzeni adresowej. Jeśli zaś chodzi o
`no-resolv` , to za jego sprawą nie będzie czytany plik `/etc/resolv.conf` w poszukiwaniu
upstream'owych serwerów DNS (o tym za moment). Dalej mamy dwa wywołania `server` , z których
pierwszy określa adres upstream'owego serwera DNS, natomiast drugi przesyła zapytania do serwera
czasu (domeny `*pool.ntp.org` ) do osobnego serwera DNS. Kolejne trzy opcje, tj. `interface` ,
`listen-address` oraz `bind-interfaces` , mają na celu określenie interfejsu i adresu, na którym
będzie nasłuchiwał serwer DNS. Jako, że jest to dość wrażliwa usługa i nie mamy zamiaru jej
upubliczniać, to dobrze jest ją odseparować od reszty świata przenosząc ją na interfejs pętli
zwrotnej. Nie chcemy także by działał serwer DHCP, dlatego też precyzujemy opcję
`no-dhcp-interface` . Dalej w pliku mamy `no-hosts` , który zabroni czytania pliku `/etc/hosts` .
Kolejne trzy opcje dotyczą cache serwera DNS i mamy tam `cache-size` odpowiadający za ilość wpisów
w cache (o tym też za moment), oraz `min-cache-ttl` i `max-cache-ttl` , które ustalają [czas
ważności rekordu DNS][3], po upłynięciu którego trzeba będzie ponownie odpytać upstream'owy serwer
DNS. Opcja `dns-forward-max` odpowiada za ilość jednoczesnych zapytań, które mogą być obsługiwane
przez serwer DNS. Z kolei ostatnia opcja, tj. `no-negcache` sprawi, że do cache nie będą dodawane
wpisy, których nie udało się rozwiązać.

## Przesyłanie zapytań do lokalnego serwera DNS

Ta powyższa konfiguracja dotyczy jedynie serwera DNS i po zresetowaniu demona `dnsmasq`, ten
powinien już zacząć realizować swoją funkcję. Jest tylko jeden problem. Mianowicie, trzeba przesłać
wszystkie zapytania o domeny do `dnsmasq` , a za to odpowiada plik `/etc/resolv.conf` . Możemy
oczywiście bezpośrednio go edytować ale nie jest to zalecane, zwłaszcza w przypadku systemd. Jeśli
korzystamy z tego init'u, to dobrze jest rzucić okiem na plik `/etc/systemd/resolved.conf` i to w
nim ustawić odpowiedni adres resolver'a, którym w tym przypadku jest `127.0.0.1` :

    [Resolve]
    DNS=127.0.0.1

Możemy to także robić w plikach konfiguracyjnych interfejsów w `/etc/systemd/network/` dodając
odpowiedni parametr, przykładowo:

    [Network]
    ...
    DNS=127.0.0.1

Trzeba także pamiętać, że jeśli korzystamy dodatkowych wynalazków jeśli chodzi o zapytania DNS, np.
ich [szyfrowanie przy pomocy dnscrypt-proxy][4], to musimy odpowiednio przekierować ruch.
Generalnie rzecz biorąc, wszystkie zapytania o domeny mają lecieć na adres 127.0.0.1, port 53/udp,
na którym nasłuchuje `dnsmasq` . Po tym jak te zapytania dotrą na ten adres, zostaną przekierowane
na adres, który został określony w konfiguracji `dnsmasq` (parametr `server` ), tj. `127.0.2.1` ,
port 5353/udp. Na nim z kolei nasłuchuje [dnscrypt-proxy][5], który przesyła otrzymane od `dnsmasq`
zapytania do upstream'owego serwera DNS określonego w konfiguracji `dnscrypt-proxy` .

## Rozmiar i ważność cache serwera DNS

W pliku `dnsmasq.conf` określiliśmy parametr `cache-size` odpowiadający za ilość wpisów, które mogą
trafić do cache `dnsmasq` . W tym przypadku jest to 10.000. Czy to mało, czy to dużo? Generalnie
rzecz biorąc, to zależy od sprzętu jakim dysponujemy. W dużej mierze decyduje tylko ilość pamięci
RAM. Jak możemy wyczytać [tutaj][6], nie powinniśmy z rozmiarem cache przeginać zanadto. Na
maszynach 32 bitowych, każdy wpis w cache ma 82 lub 74 bajty w zależności od protokołu, tj. IPv6 lub
IPv4. W przypadku maszyn 64 bitowych, rozmiar wpisów wynosi odpowiednio 94 oraz 86 bajtów. Zatem
10.000 wpisów to około 1MiB pamięci. Na desktopach to praktycznie nie ma większego znaczenia.

Każdy taki wpis w cache ma określony termin ważności. Zwykle jest to dyktowane przez główny serwer
DNS. Ten czas możemy zweryfikować sobie w każdej chwili za pomocą narzędzia `dig` , przykładowo:

    $ dig wp.pl
    ...
    ;; ANSWER SECTION:
    wp.pl.                  590     IN      A       212.77.100.101
    wp.pl.                  590     IN      A       212.77.98.9
    ...

Widoczna wyżej liczba `590` w tym przypadku oznacza ilość sekund, które muszą upłynąć, aby zapytanie
o domenę `wp.pl` zostało wysłane do upstream'owego serwera DNS. Zatem przez następne 10 minut,
zapytania o domenę `wp.pl` będą rozwiązywane przez lokalny serwer DNS. Problem w tym, że w przypadku
zmiany tych rekordów, będzie musiało upłynąć prawie 10 minut, zanim zostaną one zaktualizowane.
Jeśli w między czasie spróbujemy odwiedzić tę domenę, to możemy nie uzyskać połączenia. Dlatego też
dobrze jest sobie dostosować ten czas podając w konfiguracji `dnsmasq` parametry `min-cache-ttl` i
`max-cache-ttl` . Nie należy ustawić zbyt niskich wartości, bo wtedy większość zapytań będziemy
otrzymywać z upstream'owego serwera DNS. Nie ma też co przesadzać ze zbyt wielkimi wartościami.

## Statystyki cache

Mając skonfigurowany lokalny cacher DNS, możemy sprawdzić czy działa tak jak powinien. Odpalamy
zatem terminal i przy pomocy `dig` sprawdzamy czas realizacji zapytania:

    $ dig onet.pl | grep "Query time"
    ;; Query time: 30 msec

    $ dig onet.pl | grep "Query time"
    ;; Query time: 0 msec

Jak widzimy wyżej, za pierwszym razem, zapytanie trwało 30 ms. Z kolei za drugim sporo mniej.
Wskazuje to na fakt, że zapytanie o domenę zostało przechwycone przez `dnsmasq` . W tym momencie w
cache pojawił się stosowny wpis i przy drugiej próbie odpytania tej domeny, rekord DNS został
zaserwowany bezpośrednio z cache. Zatem w sporej części przypadków te zapytania będą realizowane
lokalnie i oszczędzą nam sporo czasu przy ładowaniu stron www.

Statystyki całego cache możemy wyciągnąć z logu systemowego, tylko pierw trzeba wysłać odpowiedni
sygnał do demona `dnsmasq` :

    # kill -s USR1 108091

Po wydaniu tego powyższego polecenia, w logu powinniśmy zarejestrować poniższy komunikat:

    dnsmasq[1880]: cache size 10000, 0/13310 cache insertions re-used unexpired cache entries.
    dnsmasq[1880]: queries forwarded 8973, queries answered locally 10057
    dnsmasq[1880]: queries for authoritative zones 0
    dnsmasq[1880]: server 208.67.222.222#53: queries sent 2, retried or failed 0
    dnsmasq[1880]: server 127.0.2.1#5353: queries sent 8971, retried or failed 109

Jak czytać te powyższe statystyki? Zgodnie z tym co wyczytałem na [liście mailingowej
dnsmasq][7], mając w pierwszej linijce `0/2431` , `0` oznacza brak usuniętych wpisów z cache zanim
utraciły one ważność, natomiast `2431` oznacza ilość wpisów w cache ogółem. Te dwie wartości mówią
nam czy ustawienia rozmiaru cache, czy też czasu ważności wpisów w nim należy dostosować.
W przypadku gdy pierwsza wartość zacznie się zwiększać, trzeba pomyśleć nad zmianą parametrów cache.
Dalej w logu nie ma już nic niezwykłego. Są zapytania, które zostały przesłane do upstream'owego
serwera DNS ( `1502` ) i ilość rozwiązanych lokalnie ( `660` ). Są także statystyki dla konkretnych
serwerów DNS.


[1]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[2]: /post/konfiguracja-trybu-ap-kart-wifi-na-debianie/
[3]: https://en.wikipedia.org/wiki/Time_to_live#DNS_records
[4]: /post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/
[5]: https://dnscrypt.org/
[6]: https://flux242.blogspot.fr/2012/06/dnsmasq-cache-size-tuning.html
[7]: http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2013q2/007331.html
