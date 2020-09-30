---
author: Morfik
categories:
- Linux
date: "2015-10-24T17:39:24Z"
date_gmt: 2015-10-24 15:39:24 +0200
published: true
status: publish
tags:
- iptables
- tcp
- sysctl
- dos
- ddos
- synproxy
title: Unikanie ataków DDoS z SYNproxy
---

Internet nie jest zbyt przyjaznym miejscem i jest wielce prawdopodobne, że prędzej czy później ktoś
zaatakuje jedną z naszych maszyn, która świadczy w nim jakieś usługi. Są różne typy ataków, w tym
przypadku chodzi o ataki DDoS z wykorzystaniem pakietów wchodzących w proces potrójnego witania
(three way handshake) przy nawiązywaniu połączenia w protokole TCP, tj. pakiety `SYN` , `SYN-ACK` i
`ACK` . Istnieje szereg mechanizmów, które adresują problem SYN flooding'u ale żaden z nich nie jest
doskonały. Jakiś czas temu, do kernela linux'owego trafił patch implementujący [mechanizm
SYNproxy](https://lwn.net/Articles/563151/) i w tym wpisie obadamy go sobie nieco dokładniej.

<!--more-->
## Jak działa SYNproxy

Większość systemów linux'owych ma zaimplementowany firewall, który zwykle operuje na stanach
połączeń. Stanów mamy z grubsza trzy: NEW, ESTABLISHED i RELATED. Wobec czego, każdy pakiet musi
być przypisany do jakiegoś z tych stanów by zapora wiedziała czy ma go przepuścić czy też
zablokować. Każde poprawnie nawiązane połączenie w protokole TCP zaczyna się od pakietu `SYN` , na
który druga strona połączenia odpowiada pakietem `SYN-ACK` , no i po otrzymaniu tego pakietu, ta
pierwsza strona wysyła pakiet `ACK` . Po tym procesie, połączenie zostaje ustanowione i można zacząć
wymieniać informacje. Jest jednak szereg pakietów, które nie pasują do tych powyższych stanów
połączeń, np. mają ustawione nieodpowiednie
[flagi](/post/flagi-tcp-i-przelaczanie-stanow-polaczen/) i wtedy takie pakiety
zostają zakwalifikowane do stanu INVALID, który zwykle jest blokowany na zaporze. Dokładny opis
poszczególnych stanów można znaleźć we [wpisie poświęconym budowie
firewall'a.](/post/firewall-na-linuxowe-maszyny-klienckie/) Może ten opisany wyżej
mechanizm wydaje się być w porządku ale jego podstawową wadą jest fakt, że wszystkie połączenia,
nawet te, które nie zostały jeszcze poprawnie nawiązane, są śledzone przez kernel za pomocą modułu
`conntrack`, a to zjada sporo cennych zasobów i tu właśnie do gry wchodzi SYNproxy.

W przypadku mechanizmu SYNproxy, mamy do czynienia z [dwoma etapami
witania](https://github.com/firehol/firehol/wiki/Working-with-SYNPROXY) (three way handshake).
Pierwszy z nich występuje na linii klient -\> SYNproxy , drugi zaś na linii SYNproxy -\> serwer.
Mając to na uwadze, gdy klient wysyła pakiet `SYN` do serwera, to ten trafiając na firewall jest
oznaczany jako UNTRACKED, a następnie przesyłany do SYNproxy, który odpowiada klientowi pakietem
`SYN-ACK` również w stanie UNTRACKED. Na ten pakiet klient odpowiada pakietem `ACK` , który może być
oznaczony na zaporze jako INVALID lub UNTRACKED. Dzięki takiemu stanowi rzeczy, nie są zjadane żadne
zasoby pamięci.

Po tym jak połączenie zostanie nawiązane z SYNproxy, ten automatycznie rozpoczyna proces witania z
faktycznym serwerem spoofując pakiet `SYN` , tak by serwer był w stanie zobaczyć, że ten oryginalny
klient próbuje się podłączyć. Odbywa się to przez nawiązanie nowego połączenia przez SYNproxy, które
w iptables jest łapane przez łańcuch OUTPUT. Źródłowy adres IP takiego pakietu jest podmieniany na
adres IP oryginalnego klienta. Następnie serwer odpowiada pakietem `SYN-ACK` do tego klienta. Ten
pakiet trafia jednak do SYNproxy, który odpowiada pakietem `ACK` i w ten sposób połączenie
przechodzi w stan ESTABLISHED. Z chwilą gdy połączenie zostaje ustanowione, SYNproxy nie bierze już
udziału w wymianie pakietów między klientem a serwerem.

## Dostosowanie kernela pod SYNproxy

By skorzystać z mechanizmu SYNproxy musimy posiadać kernel w wersji minimum 3.12 oraz iptables w
wersji minimum 1.4.21 . Innym wymaganym czynnikiem jest aktywacja [mechanizmu SYN
cookies](/post/mechanizm-syn-cookies-w-protokole-tcp/), a że on korzysta ze
[znaczników czasów (TCP
timestamp)](/post/znacznik-czasu-timestamp-w-protokole-tcp/), to również i one
muszą zostać aktywowane w ustawieniach kernela. Poniżej są zebrane wszystkie te rzeczy, które
musimy dopisać do pliku `/etc/sysctl.conf` :

    net.ipv4.tcp_timestamps = 1
    net.netfilter.nf_conntrack_timestamp = 1
    net.ipv4.tcp_syncookies = 1

Dodatkowo, musimy nieco inaczej skonfigurować sam moduł `conntrack` . Mianowicie, pakiety `ACK`
nieodnoszące się do żadnego ustanowionego połączenia muszą zostać wyjęte spod mechanizmu śledzenia
połączeń. Sprawi to, że takie pakiety zostaną zakwalifikowane jako INVALID i zarazem uchroni nas od
zalewów pakietów `ACK` (ACK flood). Do pliku `/etc/sysctl.conf` dopisujemy zatem również i ten
poniższy parametr:

    net.netfilter.nf_conntrack_tcp_loose = 0

By zmiany zostały zaaplikowane, musimy jako administrator jeszcze wpisać w terminalu `sysctl -p` .

## Implementacja SYNproxy w iptables

Będziemy operować na dwóch tablicach: `raw` oraz `filter` . Reguły są w sumie trzy. Pierwszą z nich
dodajemy do tablicy `raw` , a to z tego względu, że pakiety, które do niej trafiają, nie podpadają
jeszcze pod mechanizm śledzenia połączeń. Dopasowanie jest dowolne, tylko trzeba pamiętać, że
protokół, na którym operujemy to `TCP` . Z kolei, target , który musimy określić dla tej reguły to
`-j NOTRACK` . Informuje on system, by pakietów złapanych przez tę regułę nie śledził. Poniżej
przykładowa reguła:

    iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 80 --syn -j NOTRACK

Poniższa reguła zaś wędruje do tablicy `filter`
    .

    iptables -I INPUT -p tcp -m tcp -m conntrack --ctstate INVALID,UNTRACKED -j SYNPROXY --sack-perm --timestamp --wscale 7 --mss 1460

Ma ona za zadanie złapanie wszelkich pakietów w stanie INVALID (między innymi `ACK` ) oraz UNTRACKED
(pakiety `SYN` ) i przesłanie ich do SYNproxy.

Ostatnia reguła raczej powinna być już standardowo na naszej zaporze, bo odpowiada ona za blokowanie
pakietów w stanie INVALID ale moim zdaniem powinniśmy ją umieścić bezpośrednio za regułami z
SYNproxy. Poniżej ta reguła:

    iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

Reguły w iptables powinny łapać pakiety. Jeśli liczniki biją, znaczy, że wszystko działa jak należy.
By mieć absolutną pewność, że mechanizm SYNproxy działa, możemy także zajrzeć do pliku
`/proc/net/stat/synproxy` , który powinien pokazać nam informacje na temat otrzymanych pakietów
`SYN` i ilości ważnych i nieważnych ciasteczek.
