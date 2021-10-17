---
author: Morfik
categories:
- Linux
date:    2021-10-14 23:52:00 +0200
lastmod: 2021-10-14 23:52:00 +0200
published: true
status: publish
tags:
- debian
- ipv4
- ipv6
- sysctl
- bezpieczeństwo
- sieć
GHissueID: 575
title: Jak wyłączyć mapowanie adresów IPv4 na IPv6 w Debian linux
---

Przeglądając ostatnio połączenia w swoim telefonie z Androidem, natknąłem się na wpisy mające w
swoich adresach źródłowych ustawione IP takie, jak `::ffff:192.168.1.188` . Podobnie sprawa wygląda
w przypadku adresów docelowych tego samego połączenia, tj. widnieje tam np. `::ffff:216.58.215.1` .
Na pierwszy rzut oka taka konstrukcja adresu IP przypomina IPv6, przynajmniej jej pierwsza część,
tj. `::ffff:` , natomiast drugi kawałek już bardzo przypomina adres IPv4. Okazuje się, że za taki
format adresów IP odpowiada [mechanizm mapowania adresów IPv4 na adresy IPv6][8]. Nie jest to
jednak tożsame z [6to4][9] czy [6in4][10], gdzie host ze stałym publicznym adresem IPv4 jest w
stanie komunikować się z hostami nadającymi po IPv6. W przypadku mapowania adresów IPv4 na IPv6,
połączenia ze zdalnymi hostami mogą odbywać się zarówno po IPv4 jak i IPv6, a to z którego z tych
protokołów nasz linux skorzysta zależy od tego czy taka maszyna dysponuje przydzielonym adresem
IPv6. Jeśli ISP nie zapewnia nam adresu IPv6, to wtedy pakiety sieciowe będą przesyłane z
wykorzystaniem protokołu IPv4. Wciąż jednak te mieszane konstrukcje adresów IP będą widoczne na
wykazie połączeń w `netstat`/`ss` . Okazuje się jednak, że ten mechanizm mapowania adresów może
powodować szereg problemów związanych z bezpieczeństwem systemu. Dlatego też tam gdzie to możliwe
przydałoby się go wyłączyć.

<!--more-->
## Konstrukcja adresów IPv4 mapowanych na IPv6

Szukając nieco informacji na temat mapowania adresów IPv4 na IPv6 znalazłem [taki oto artykuł][1].
Dla lepszego zrozumienia tematu postanowiłem przetłumaczyć część tego tekstu i wrzucić go poniżej.

Protokół IPv6 może w wielu przypadkach przypominać IPv4 ale są to dwa różne protokoły mające inną
przestrzenią adresową. Programy chcące odbierać połączenia przy wykorzystaniu któregokolwiek z tych
protokołów muszą otworzyć osobne gniazda (socket) dla każdej z tych dwóch różnych rodzin adresów,
tj. `AF_INET` dla IPv4 oraz `AF_INET6` dla IPv6. Gdy jakiś proces chciałby akceptować połączenia na
każdym interfejsie hosta korzystając z obu tych protokołów, musi stworzyć gniazdo AF_INET przypisane
do adresu `0.0.0.0` oraz gniazdo AF_INET6 przypisane do adresu `::` . W ten sposób taki proces
musiałby nasłuchiwać na obu gniazdach, przynajmniej tak mogłoby się wydawać.

Niemniej jednak, wiele lat temu ([RFC 3493 z 2003 roku][3]) inżynierowie z IETF obmyślili mechanizm,
dzięki któremu program mógł komunikować się przy pomocy obu protokołów korzystając tylko z
pojedynczego gniazda IPv6. Gdy taki mechanizm zostanie włączony w systemie, to dany program może
stworzyć jedynie gniazdo z adresem `::` i w ten sposób odbierać połączenia na wszystkich
interfejsach przy wykorzystaniu obu protokołów IP. Gdy tworzone jest połączenie w protokole IPv4 na
dany prot, adres źródłowy będzie mapowany na IPv6 ([zgodnie z RFC 2373][4]). Dla przykładu,
połączenia z adresu 192.168.1.1 będą w źródłowym IP posiadać wpis `::ffff:192.168.1.1` . Podobnie
też w drugą stronę, tj. gdy program chce nawiązać połączenie z adresem IPv4 może to zrobić mapując
te adresy docelowe w dokładnie ten sam sposób.

Zatem typowy adres IPv6 ma 128 bitów, z których 96 bitów robi za prefiks i stosuje się tutaj
standardowy format dla adresu IPv6. Natomiast pozostałe 32 bity są zapisane w standardowej notacji,
którą się wykorzystuje przy adresach IPv4. Z kolei te pierwsze 96 bitów jest podzielone na 80 bitów
zer i 16 bitów jedynek, w ten sposób dostajemy zapis `::ffff:` , po którym mamy już zwykły adres
IPv4. Poniżej przykład wyjścia `ss` z mojego smartfona:

    galahad:/ # ss -napetu | grep -i conver
    Netid State    Recv-Q Send-Q    Local Address:Port            Peer Address:Port
    ...
    tcp    ESTAB        0      0    [::ffff:192.168.1.188]:41466  [::ffff:37.187.98.68]:5222                users:(("s.conversations",pid=22634,fd=70)) uid:10256 ino:2356503 sk:134b1 fwmark:0x67 <->
    ...

Jak widać, mamy tutaj wpis wskazujący na protokół TCP. Mimo iż jest tutaj wpis z adresem
przypominającym adres IPv6, to z racji braku IPv6 u operatora ISP, komunikacja dla tego połączenia
odbywa się za pośrednictwem protokołu IPv4.

Co ciekawe, w przypadku `netstat` wyjście zdaje się nie do końca być prawidłowe.

    galahad:/ # netstat -napletu | grep -i conver
    Active Internet connections (established and servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode       PID/Program Name
    ...
    tcp6       0      0 ::ffff:192.168.1.:41466 ::ffff:37.187.98.6:5222 ESTABLISHED 10256      2356503     22634/eu.siacs.conversations
    ...

Tutaj już mamy TCP6, który wskazuje na wykorzystanie protokołu IPv6. Dalej też mamy ucięty ostatni
oktet w źródłowym adresie IP. Być może jest to związane z faktem, że `netstat` nie jest już
rozwijany od ponad dekady i nie powinno się już z niego korzystać.

## Problemy z mapowaniem adresów w aplikacjach

To powyżej opisane zachowanie ma być implementowane domyślnie i większość systemów operacyjnych
podąża za tymi wytycznymi obecnymi w przytoczonych RFC. Są jednak pewne wyjątki, z których jednym
jest OpenBSD. Tutaj aplikacje muszą tworzyć niezależne gniazda dla każdego z protokołów IP.
Dyktowane jest to względami bezpieczeństwa. W przypadku innych systemów, ten mechanizm mapowania
adresów jest domyślnie włączony, co z kolei może powodować problemy, gdzie jednoczesne otworzenie
gniazd IPv4 i IPv6 przez dany program skutkować będzie podwójną próbą utworzenia gniazda IPv4, tj.
raz za sprawą adresu `::` , a drugi raz przez `0.0.0.0` . Biorąc pod uwagę fakt, że tylko jedno
gniazdo z danym adresem IP (i portem) może być aktywne w konkretnym czasie, to ta druga próba
będzie wiązać się z błędem, bo gniazdo o takim adresie już zostało utworzone.

By rozwiązać ten problem, programy mogą korzystać z wywołania `setsockopt()` , by włączyć opcję
`IPV6_V6ONLY` . W ten sposób, programy, które ustawią sobie `IPV6_V6ONLY` , mogą działać na każdym
systemie, co zapewnia portowalność aplikacji, ale nie wszystkie programy z tej opcji korzystają lub
implementują ją prawidłowo. Jednym z przykładów są aplikacje w Androidzie (Java), które po
wyłączeniu mapowania adresów nie potrafią nawiązywać połączeń internetowych. Dlatego też tego
mechanizmu raczej nie da się wyłączyć w Androidach, przynajmniej jeśli chcemy by aplikacje w
telefonie miały dostęp do internetu. Oczywiście cześć aplikacji w Androidzie jest w stanie działać
przy wyłączeniu mapowania adresów ale nie są to wszystkie appki.

## Problemy z bezpieczeństwem mapowania adresów

Dana maszyna może mieć skonfigurowane reguły firewall'a odpowiadające za limitowanie dostępu do
danego portu w systemie. Mogą też istnieć inne mechanizmy, np. [TCP wrappers][5] albo [filtry
oparte na BPF][6], lub też router w sieci może przeprowadzać własne filtrowanie połączeń stanowych.
Rezultatem takiego działania mogą być dziury w zaporze sieciowej i no też może to generować niemałe
zamieszanie wynikające z faktu, że ten sam adres IPv4 jest osiągalny przez dwa różne protokoły.
Jeśli mapowanie adresów odbywa się na brzegu sieci, to sytuacja staje się jeszcze bardziej
skomplikowana.

[Tutaj jest RFC z 2003 roku][2], który opisuje szereg różnych scenariuszy ataku, które stają się
możliwe do przeprowadzenia, gdy zmapowane adresy są przekazywane pomiędzy hostami. Przykładem może
być adres `::ffff:127.0.0.1` , z którego pakiety mogłyby zostać potraktowane jak te pochodzące z
adresu 127.0.0.1 pętli zwrotnej (lo). Podobnie sprawa wygląda z adresami prywatnymi IPv4, np.
`::ffff:10.1.1.1` , gdzie mechanizmy bezpieczeństwa sieci mogłyby potraktować ten pakiet, jako
pochodzący z wnętrza sieci, przez co można by te zabezpieczenia sieci obejść.

## Kontrolowanie mapowania adresów przez sysctl

Jak już zostało wyżej wspomniane, OpenBSD w ogóle nie obsługuje gniazd mapowanych adresów IPv4 na
IPv6. Nawet jeśli jakiś program próbowałby włączyć mapowanie adresów poprzez ustawienie opcji
`IPV6_V6ONLY` na `0` , to nic mu to nie da, bo to ustawienie nie ma żadnego wpływu na systemy
OpenBSD. A jak jest w przypadku linux'ów?

W [Debianie mechanizm mapowania adresów jest domyślnie włączony][7] już od ponad dekady. Niemniej
jednak, ja w swoim systemie nie zauważyłem, by jakiekolwiek hybrydy adresów pokroju
`::ffff:192.168.1.1` na liście połączeń się u mnie kiedykolwiek pojawiły. Tak czy inaczej, kernel
linux'a udostępnia opcję `net.ipv6.bindv6only` , która odpowiada za kontrolowanie mechanizmu
mapowania adresów. Przy pomocy tej opcji jesteśmy w stanie ustawić odpowiednią wartość w
`IPV6_V6ONLY` .

Jeśli parametr `net.ipv6.bindv6only` zostanie ustawiony na `0` , to mapowanie adresów będzie
włączone i to jest domyśle zachowanie kernela linux. Jeśli zaś ustawimy tutaj `1` , to mapowanie
adresów zostanie wyłączone dla wszystkich aplikacji w systemie. Zatem sprawa wyłączenia mapowania
adresów IPv4 na IPv6 jest banalnie prosta i wystarczy odpalić w naszym linux'ie terminal i wpisać w
nim to poniższe polecenie:

    # sysctl -w net.ipv6.bindv6only=1

By zmiany miały efekt permanentny, dopisujemy poniższą linijkę do pliku `/etc/sysctl.conf` :

    net.ipv6.bindv6only = 1

## Podsumowanie

Czasami ludzie zamieszczają w RFC bardzo nietrafione pomysły. Takie do końca nieprzemyślane
koncepcje prowadzą do sytuacji, w których proste rzeczy stają się dość skomplikowanymi, przez co
wymagają od przeciętnego Kowalskiego bycia ekspertem w dziedzinie IT, przynajmniej jeśli chodzi o
administrację systemami operacyjnymi. Włączenie mapowania adresów IPv4 na adresy IPv6 może nieć ze
sobą zagrożenie bezpieczeństwa i jeśli nam zależy na minimalizowaniu powierzchni ewentualnych
ataków, to ten mechanizm powinniśmy wyłączyć. Trzeba jednak zdawać sobie sprawę, że szereg
aplikacji może nie działać prawidłowo bez tego mapowania adresów. Niemniej jednak, póki co na
moich maszynach działających pod kontrolą linux'a (za wyjątkiem Androida) nie spotkałem się z
sytuacją, by ten mechanizm mapowania adresów był potrzebny, czy też w ogóle wykorzystywany.


[1]: https://lwn.net/Articles/688462/
[2]: https://tools.ietf.org/html/draft-itojun-v6ops-v4mapped-harmful-02
[3]: https://tools.ietf.org/html/rfc3493#section-3.7
[4]: https://tools.ietf.org/html/rfc2373#page-10
[5]: https://en.wikipedia.org/wiki/TCP_Wrappers
[6]: https://www.kernel.org/doc/html/latest/networking/filter.html
[7]: https://lists.debian.org/debian-devel/2009/10/msg00541.html
[8]: https://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses
[9]: /post/implementacja-protokolu-ipv6-za-pomoca-tunelu-6to4/
[10]: /post/konfiguracja-tunelu-6in4-w-openwrt-ipv6/
