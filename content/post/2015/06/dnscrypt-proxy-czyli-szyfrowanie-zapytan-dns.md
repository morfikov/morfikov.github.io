---
author: Morfik
categories:
- Linux
date:    2015-06-18 20:00:54 +0200
lastmod: 2020-00-19 15:40:54 +0200
published: true
status: publish
tags:
- dns
- resolver
- szyfrowanie
- systemd
- dnscrypt
- prywatność
- bezpieczeństwo
title: DNScrypt-proxy, czyli szyfrowanie zapytań DNS
---

Do protokołów SSL/TLS w serwisach www chyba wszyscy już przywykli. Obecnie praktycznie na każdej
stronie, gdzie jest okienko logowania, mamy do czynienia z szyfrowaniem danych przesyłanych w
różnego rodzaju formularzach. Co prawda, klucze nie są zbyt długie (512-2048 bitów) ale zawsze to
lepsze niż nic. O ile dane służące do logowania czy też wszelkie operacje dokonywane w panelach
administracyjnych da radę ukryć bez większego problemu, o tyle zapytania DNS są przesyłane otwartym
tekstem i każdy może je sobie podejrzeć. Pytanie tylko, po co szyfrować ruch DNS? Czy są tam
przesyłane jakieś ważne informacje? Jeśli przyjrzymy się jak działa system DNS, możemy dojść do
wniosku, że szyfrowanie zapytań jest pozbawione sensu, bo z reguły nazwa domeny oznacza konkretny
adres IP. Znając adres IP, możemy ustalić kto na jakie strony wchodzi. Nie do końca jest to prawdą,
poza tym istnieją jeszcze inne czynniki, które sprawiają, że ukrycie zapytań DNS ma jak najbardziej
sens. W tym wpisie zaimplementujemy sobie na naszych linux'ach szyfrowanie zapytań DNS przy
wykorzystaniu [narzędzia dnscrypt-proxy][1].

<!--more-->

> Bardziej aktualna wersja tego wpisu znajduje się [tutaj][6].

## Wiele domen i cenzura

Czy naprawdę możemy ustalić kto jakie serwisy odwiedza na podstawie adresu IP? W sporej części
przypadków zapewne tak ale tylko pod warunkiem, że dany adres IP jest wykorzystywany tylko przez
jeden serwis. A co w przypadku, gdy jeden adres IP hostuje kilka czy kilkaset stron? Znając tylko
adres IP, nie będziemy mieć pewności, która strona została tak naprawdę odwiedzona. Co prawda,
będziemy mieli zawężony krąg podejrzanych ale nie ustalimy czy dana osoba odwiedziła ten konkretny
adres. Znając zapytania DNS, wiemy dokładnie kto gdzie wchodzi.

Każdy słyszał o blokadach domen dokonywanych przez rząd USA. Jak to ma się do zapytań DNS? Podobnie
jak rząd USA, rząd Polski może narzucić lokalnym ISP filtry stron zakazanych. O ile w przypadku
rządu USA, który jest właścicielem root serwerów DNS, nie możemy nic zrobić, bo blokady są
globalne, o tyle w sytuacji, gdy nasz ISP zapragnie nam blokować serwisy internetowe filtrując DNS,
możemy się bronić zmieniając serwery DNS. Niemniej jednak, cały ruch dalej przechodzi przez naszego
ISP i zmiana samych adresów DNS nic nie da. Frazy domen mogą zostać odfiltrowane ale tylko w
przypadku, gdy będą widoczne i tu właśnie znajduje zastosowanie szyfrowanie ruchu DNS.

## Zmiana serwerów DNS

Usługa OpenDNS oferuje dość zaawansowaną konfigurację DNS wliczając w to rozmaite filtry, w tym też
blokowanie pojedynczych stron www. Można nawet włączyć log odwiedzanych adresów i generować ciekawe
statystyki. Bardzo przydatne jeżeli mamy dzieci i chcemy wiedzieć na jakie strony wchodzą i czy są
wśród nich strony porno. Wtedy bez problemu można zablokować odpowiednie domeny. Inną użyteczną
opcją jaką dysponuje OpenDNS, to szyfrowanie zapytań DNS.

Obecnie każdy system operacyjny na starcie pobiera konfigurację sieci przez serwer DHCP od swojego
ISP. Jest tam też konfiguracja serwerów DNS, z tym, że jest ona opcjonalna. Możemy albo na sztywno
zdefiniować adresy DNS przy konfiguracji sieci, albo wpisać odpowiednie dane w pliku
`/etc/resolv.conf` . Ręczna edycja pliku `resolv.conf` może się wydawać bezzasadna ale nie w
przypadku, gdy chcemy zablokować możliwość przypadkowej zmiany tego pliku, np. przez zewnętrzne
oprogramowanie. Chodzi o nadanie atrybutu `chattr +i` . Teraz mamy pewność, że w tym pliku znajduje
się dokładnie taka zawartość jakiej oczekujemy i nie musimy sprawdzać co chwila, czy czasem adresy
serwerów DNS nie uległy zmianie.

Gdy chcemy korzystać z serwerów OpenDNS, dodajemy w pliku `/etc/resolv.conf` poniższe wpisy:

    nameserver 208.67.222.222
    nameserver 208.67.220.220

W przypadku korzystania z [systemd][2], plik `resolv.conf` jest dowiązaniem do
`/run/systemd/resolve/resolv.conf` i jako że ten plik znajduje się w katalogu `/run/` , jest on
tworzony na nowo przy starcie systemu. Zatem manualna edycja tego pliku nie jest zalecana. Lepszym
wyjściem jest konfiguracja resolver'a bezpośrednio przez [pliki .network][3].

## DNScrypt-proxy na straży prywatności

My jednak będziemy korzystać z opcji szyfrowania zapytań i nie możemy bezpośrednio użyć powyższych
adresów. Musimy pierw utworzyć lokalny serwer, który będzie zapytania szyfrował i przesyłał je do
jednego z resolver'ów, które mają zaimplementowaną obsługę szyfrowania. W tym przypadku posłużymy
się [OpenDNS][4].

Oprogramowanie, które posłuży nam do zaszyfrowania ruchu DNS to `dnscrypt-proxy` . W repozytorium
Debiana mamy dostępny pakiet o takiej nazwie, zatem z jego instalacją nie powinno być problemów.
Przejdźmy od razu do konfiguracji tego narzędzia.

Po wystartowaniu demona, wszystkie zapytania są gotowe do puszczenia w kanał TLS. Problem w tym, że
wpisy w `/etc/resolv.conf` wskazują na zły adres resolver'a. Całą zawartość tego pliku musimy usunąć
i na jej miejscu dodać tę poniższą linijkę:

    nameserver 127.0.2.1

Dla pewności, że plik nie zostanie przepisany przez jakieś automaty systemowe, możemy nadać atrybut
`+i` na ten plik:

    # chattr +i /etc/resolv.conf

## Testowanie konfiguracji resolver'a

By sprawdzić czy zapytania DNS są szyfrowane za sprawą `dnscrypt-proxy` , możemy odwiedzić [stronę
testową OpenDNS][5], gdzie powinniśmy zobaczyć ten poniższy komunikat:

![](/img/2015/06/1.test-konfiguracji-dnscrypt-proxy.png#medium)

Nie mamy tam żadnych informacji dotyczących szyfrowania ruchu ale skoro przekierowaliśmy ruch na
lokalny resolver i jesteśmy w stanie odwiedzić stronę OpenDNS, to znaczy, że ruch musi być przesłany
w formie szyfrowanej. Jeśli jednak chcemy zobaczyć odpowiednie komunikaty na własne oczy, to możemy
skorzystać z narzędzia `dig` odpytując nim adres `debug.opendns.com` . Jeśli wszystko działa jak
należy, to powinniśmy uzyskać mniej więcej taki log:

    $ dig debug.opendns.com txt
    ...
    debug.opendns.com.      0       IN      TXT     "dnscrypt enabled (71447764594D3377)"

    ;; Query time: 31 msec
    ;; SERVER: 127.0.2.1#53(127.0.2.1)
    ;; WHEN: Thu Jun 18 19:43:08 CEST 2015
    ;; MSG SIZE  rcvd: 282

Linijka z `dnscrypt enabled` jasno nas informuje, że ruch jest szyfrowany. Dodatkowo, niżej mamy
adres resolver'a, który wskazuje na nasz adres lokalny `127.0.2.1#53` .


[1]: https://dnscrypt.org/
[2]: https://www.freedesktop.org/wiki/Software/systemd/
[3]: https://www.freedesktop.org/software/systemd/man/systemd.network.html
[4]: https://www.opendns.com/
[5]: https://www.opendns.com/welcome/
[6]: /post/szyfrowany-dns-z-dnscrypt-proxy-i-dnsmasq-na-debian-linux/
