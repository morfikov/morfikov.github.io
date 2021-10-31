---
author: Morfik
categories:
- Linux
date: "2016-07-16T18:59:19Z"
date_gmt: 2016-07-16 16:59:19 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
- sysctl
- ftp
GHissueID: 391
title: 'nf_conntrack: automatic helper assignment is deprecated'
---

Jeśli ktoś uważnie śledzi logi systemowe, to od czasu do czasu można w nich znaleźć komunikat,
którzy brzmi mniej więcej tak: `nf_conntrack: automatic helper assignment is deprecated and it will
be removed soon. Use the iptables CT target to attach helpers instead` . Ta wiadomość odnosi się do
jednego z modułów linux'owego filtra pakietów `iptables` . Moduł, o którym mowa to `nf_conntrack` ,
który odpowiada za śledzenie połączeń nawiązywanych przez system. Sam komunikat zaś dotyczy
mechanizmów pomocniczych, których sposób aktywacji jest już nieco przestarzały i zostanie wkrótce
usunięty. Co to oznacza dla przeciętnego użytkownika linux'a i czym są w istocie te mechanizmy
pomocnicze, które znajdują zastosowane na zaporze sieciowej?

<!--more-->
## Mechanizmy pomocnicze przy śledzeniu połączeń

Szereg protokołów używa różnych portów do komunikacji. Weźmy sobie przykładowo protokół FTP. Mamy w
nim dwa porty. Jeden z nich ma numer 21 i jest to port poleceń. Drugi zaś ma numer 20 i jest to port
danych. Gdy w protokole FTP ma dojść do wymiany plików, port poleceń jest zwykle wykorzystywany do
negocjowania parametrów konfiguracyjnych dla nowego połączenia (adres IP i port), którym mają
popłynąć dane. W takiej sytuacji `iptables` nie może w prosty sposób filtrować ruchu. Dlatego też
w Netfilter zostały wbudowane mechanizmy pomocnicze (Connection Tracking helpers), które pomagają
zaporze uporać się z połączeniami protokołów takich jak wspomniany wyżej FTP. Przykładem innych
protokołów są [Session Initiation Protocol
(SIP)](https://pl.wikipedia.org/wiki/Session_Initiation_Protocol) oraz
[H.323](https://pl.wikipedia.org/wiki/H.323).

Ci "pomocnicy" tworzą coś na wzór "śledzonych połączeń oczekiwanych" (expectations), które są
trzymane w osobnej tablicy przez pewien okres czasu. Ta tablica jest ulokowana w pliku
`/proc/net/nf_conntrack_expect` , który można sobie bez problemu podejrzeć. Jeśli na zaporze przez
ten przedział czasu [zostanie dostrzeżony pakiet o określonych
parametrach](https://www.frozentux.net/iptables-tutorial/chunkyhtml/x1703.html), to kernel oznaczy
go jako powiązany (RELATED). Jeśli teraz na zaporze mamy poniższą regułę:

    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

To wszystkie te pakiety zostaną zaakceptowane. Niemniej jednak, takie zachowanie [stwarza zagrożenie
bezpieczeństwa](https://home.regit.org/netfilter-en/secure-use-of-helpers/), bo system musi polegać
na danych, które są zawarte w nagłówkach pakietów przesyłanych między klientem a serwerem. Dlatego
też powinniśmy wyłączyć za w czasu każdy helper, z którego nie zamierzamy korzystać. A najlepiej to
wyłączyć je wszystkie i włączyć tylko te, które są nam potrzebne.

## Jak wyłączyć helper pomagający w śledzeniu połączeń

Jeśli zdecydowaliśmy się na wyłączenie wszystkich helper'ów pomagających w śledzeniu połączeń, to
musimy odpowiednio skonfigurować sobie kernel linux'owy. Konkretnie chodzi o edycję pliku
`/etc/sysctl.conf` , w którym to trzeba ustawić parametr `net.netfilter.nf_conntrack_helper` .
Otwieramy zatem w/w plik i dodajemy do niego tę poniższą linijkę:

    net.netfilter.nf_conntrack_helper = 0

By zastosować zmiany, w terminalu jako root wpisujemy `sysctl -p` .

## Jak korzystać z helper'ów w iptables

Po dezaktywowaniu wszystkich helper'ów, jest wielce prawdopodobne, że niektóre połączenia zwyczajnie
przestaną nam działać. Jako, że FTP jest jednym z tych częściej wykorzystywanych protokołów, to na
jego przykładzie zostanie opisane jak korzystać z poszczególnych helper'ów. Na serwerze mamy
uruchomioną usługę FTP, która nasłuchuje na standardowym porcie 21. W przypadku, gdy parametr
`net.netfilter.nf_conntrack_helper` jest ustawiony na `1` , będziemy mogli, np. uzyskać listing
plików i katalogów serwera przy standardowej polityce firewalla, gdzie akceptujemy w łańcuchu INPUT
wszystkie połączenia w stanie RELATED i ESTABLISHED oraz połączenia w stanie NEW na port docelowy
21. Poniżej fotka:

![ftp-iptables-helper-sledzenie-polaczen-firewall](/img/2016/07/1.ftp-iptables-helper-sledzenie-polaczen-firewall.png#huge)

Na fotce, w dolnym okienku, mamy log, który informuje nas, że listing plików i katalogów był
dokonywany w trybie pasywnym. Rozłączmy się teraz, przestawmy parametr
`net.netfilter.nf_conntrack_helper` na `0` i podłączmy się ponownie do FTP'a. Połączenie powinniśmy
uzyskać jak poprzednio ale nie damy rady uzyskać już listingu plików i katalogów:

![ftp-iptables-helper-sledzenie-polaczen-firewall](/img/2016/07/2.ftp-iptables-helper-sledzenie-polaczen-firewall.png#huge)

Jak widzimy wyżej, został zalgoowany pakiet, który pochodzi z portu 36952 i jest przeznaczony na
port 59499. Ten port 59499 jest z przedziału portów pasywnych, który został określony w konfiguracji
serwera FTP. Niestety, zapora nie akceptuje tego połączenia, bo helper obsługujący protokół FTP
został dezaktywowany i to połączenie, które zostało wyżej zalogowane, nie zostało oznaczone jako
RELATED w stosunku do tego poprzedniego, które zostało ustanowione na porcie 21.

By FTP ponownie zaczął działać, musimy aktywować określony helper i podać mu zawężone informacje o
hostach, które ma przepuszczać. W skrócie, potrzebna jest nam poniższa reguła
    `iptables`:

    iptables -A PREROUTING -t raw -p tcp --dport 21 -s 192.168.10.0/24 -d 192.168.1.150 -j CT --helper ftp

Łączymy się ponownie z serwerem FTP i listujemy pliki i katalogi. Teraz już połączenie powinno
zostać zaakceptowane:

![ftp-iptables-helper-sledzenie-polaczen-firewall](/img/2016/07/3.ftp-iptables-helper-sledzenie-polaczen-firewall.png#huge)
