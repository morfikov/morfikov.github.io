---
author: Morfik
categories:
- Linux
date: "2019-04-12T20:53:40Z"
published: true
status: publish
tags:
- debian
- nftables
title: Blokowanie niepożądanej komunikacji z nftables na linux
---

Minęło już trochę czasu od momentu, w którym postanowiłem się przerzucić z `iptables` na `nftables`
w swoim Debianie i w zasadzie większość mechanizmów obronnych mojego laptopowego firewall'a została
już z powodzeniem przeportowana na ten nowy filtr pakietów. Poza tymi starymi regułami próbuję
czasem ogarniać nieco bardziej wyrafinowane sposoby na unikanie zagrożeń sieciowych, choć
implementacja niektórych rzeczy nie zawsze jest taka oczywista, z tym, że niekoniecznie niemożliwa
do zrealizowana. Tak było w przypadku mechanizmu automatycznego blokowania hostów próbujących się
łączyć z daną maszyną, która sobie najwyraźniej tego nie życzy. Dla przykładu, jest serwer
udostępniający usługę SSH na porcie `11111` i tylko ten port jest wystawiony na świat. Wszelkiego
rodzaju boty próbujące dostać się do maszyn linux'owych próbkują z kolei głównie standardowe porty,
w tym przypadku `22` . Ci użytkownicy, którzy powinni mieć dostęp do usługi SSH, wiedzą na jakim
porcie ona nasłuchuje. Można zatem założyć, że wszystkie połączenia na port `22` będą dokonywane
przez boty albo przez użytkowników, którzy mają niecne zamiary. Wszystkie te połączenia można by
zatem zablokować tworząc mechanizm automatycznego banowania hostów w oparciu o czas ostatniej próby
połączenia, tak jak to zostało opisane mniej
więcej [w tym wątku na forum](https://forum.dug.net.pl/viewtopic.php?pid=269383). Problem w tym, że
tamto rozwiązanie dotyczy jedynie `iptables` w połączeniu z `ipset` , co nieco komplikuje wdrożenie
go w przypadku `nftables` ale to zadanie jest jak najbardziej możliwe.

<!--more-->
## Tablica netdev

Jako, że będziemy blokować dostęp do naszej maszyny, to najlepiej robić to jak najwcześniej
(patrząc z perspektywy drogi pakietów przez filtr), by zaoszczędzić trochę zasobów sprzętowych.
Najlepszym rozwiązaniem jest zatem posłużenie się tablicą `netdev` . Jeśli ktoś się
zastanawia [czym jest tablica netdev]({{< baseurl >}}/post/unikanie-syn-icmp-udp-ping-flood-w-linux-z-nftables/)
to tutaj jest artykuł na jej temat.

Poniżej znajduje się struktura tablicy `netdev` , którą musimy dodać do konfiguracji `nftables` :

    #!/usr/sbin/nft -f

    create table netdev traffic-control

    create chain netdev traffic-control INGRESS { type filter hook ingress device bond0 priority 0; policy accept; }

W zasadzie to jedyne co trzeba tutaj dostosować to interfejs sieciowy zdefiniowany w `device` .

## Autoban w praktyce

Mając stworzoną tablicę `netdev` możemy w niej utworzyć set przechowujący listę źródłowych adresów
IP:

    add set netdev traffic-control autoban { type ipv4_addr; timeout 1h; size 128000; }

Przy takiej konfiguracji, set będzie w stanie przechować `128000` wpisów z adresami `IPv4`. Przy
próbie połączenia, rekord z adresem IP hosta zostanie dodany do tego seta z czasem wygaśnięcia
`1h` . [Więcej informacji o setach w nftables]({{< baseurl >}}/post/brak-wsparcia-dla-ipset-w-nftables/)
można znaleźć tutaj.

Dodajemy także te dwie poniższe reguły:

    add rule netdev traffic-control INGRESS meta iif !="lo" ip saddr @autoban update @autoban { ip saddr timeout 1d } counter drop
    add rule netdev traffic-control INGRESS meta iif !="lo" tcp dport { 22 } add @autoban { ip saddr } counter drop

Pierwsza reguła sprawdza obecność w secie adresu IP, z którego nastąpiła próba połączenia. Jeśli
adres ten zostanie w secie odnaleziony, to czas wygaśnięcia wpisu będzie przepisany na `1d` , czyli
24 godziny. Druga reguła z kolei monitoruje zapytania kierowane na port `22` . W przypadku
odnotowania próby połączenia na ten port, adres źródłowy delikwenta zostanie dodany do seta. W
przypadku dopasowania pakietu do którejkolwiek z tych dwóch reguł, host próbujący nawiązać
połączenie zostanie zablokowany -- przy pierwszej próbie na jedną godzinę, a jeśli podczas tej
godziny spróbuje ponownie, to na 24 godziny, itd.

Warto tutaj zaznaczyć, że pierwsza reguła nie posiada wskaźnika portu czy też innego dopasowania za
wyjątkiem adresu źródłowego. Zatem jeśli dany host spróbuje tylko połączyć się z portem `22` , to
cała jego komunikacja z naszą maszyną będzie zrzucana z automatu przez następna godzinę (lub 24h)
bez względu na to, na którym porcie/protokole będzie on chciał operować. Poniżej przykład.

Tutaj mamy sytuację wysłania pojedynczego pakietu na port `22` :

    set autoban {
    type ipv4_addr
    size 128000
    timeout 1h
    elements = { 192.168.1.181 expires 59m55s101ms }
    }

    chain INGRESS {
        type filter hook ingress device "bond0" priority filter; policy accept;
        iif != "lo" ip saddr @autoban update @autoban { ip saddr timeout 1d } counter packets 0 bytes 0 drop
        iif != "lo" udp dport { 22 } add @autoban { ip saddr } counter packets 1 bytes 46 drop
    }

Jak widać, adres IP trafił do seta, a sam rekord wygaśnie za nieco ponad 59 minut i 55 sekund.
Pakiet trafił tylko wyłącznie w drugą regułę, bo tego adresu nie było póki co w secie.

Z kolei niżej mamy następne próby połączenia (na różne porty) podczas tego jednogodzinnego bana:

    set autoban {
        type ipv4_addr
        size 128000
        timeout 1h
        elements = { 192.168.1.181 expires 23h59m47s827ms }
    }

    chain INGRESS {
        type filter hook ingress device "bond0" priority filter; policy accept;
        iif != "lo" ip saddr @autoban update @autoban { ip saddr timeout 1d } counter packets 8 bytes 368 drop
        iif != "lo" udp dport { 22 } add @autoban { ip saddr } counter packets 1 bytes 46 drop
    }

I tu już widać, że czas wygaśnięcia rekordu w secie został przepisany na 24h i wszystkie pakiety od
tego konkretnego hosta trafiają w pierwszą regułę i tak będzie przez następne 24 godziny. Jeśli ten
host choć raz spróbuje się połączyć z naszą maszyną w ciągu następnych 24 godzin, to ten interwał
zostanie odświeżany i ponownie ustawiany na 24 godziny uniemożliwiając takiemu natrętowi
jakąkolwiek komunikację z naszym serwerem.

Jak widać ten mechanizm działa i to nawet dobrze i nie było zbytnio większego problemu z
przeniesieniem go z `iptables`+`ipset` na `nftables` .
