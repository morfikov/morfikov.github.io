---
author: Morfik
categories:
- Linux
date: "2015-06-24T17:50:40Z"
date_gmt: 2015-06-24 15:50:40 +0200
published: true
status: publish
tags:
- iptables
- sysctl
title: Czas życia pakietów, czyli zmiana TTL
---

Czasem niektórzy ISP z jakiegoś bliżej nieokreślonego powodu blokują dostęp do internetu hostom
zlokalizowanym za routerem czy innym komputerem udostępniającym połączenie sieciowe. ISP zwykle
stara się blokować konkretną wartość pola TTL, która jest ustawiana w każdym przesyłanym przez nasze
maszyny pakiecie. Innym sposobem zablokowania możliwości udostępniania internetu jest ograniczenie
wartości pola TTL do jednego hopa wszystkim pakietom dochodzącym do routera od strony ISP. Jeśli
mamy nieszczęście trafić na takiego providera, to warto wiedzieć, że przy pomocy iptables możemy
ustawić/podbić/zmniejszyć czas życia pakietów, które docierają do routera zarówno od strony LAN jak
i WAN i tym samym bez większego trudu możemy sobie poradzić z tą blokadą.

<!--more-->
## Czas życia pakietów

Każdy system operacyjny ma skonfigurowaną domyślną wartość [pola
TTL](https://pl.wikipedia.org/wiki/Czas_%C5%BCycia_pakietu) dla wszystkich pakietów tworzonych na
obsługiwanej przez ten system maszynie. To pole występuje w nagłówku IP i by sobie nie nadwyrężać
wyobraźni, poniżej jest fotka obrazująca lokalizację tego pola
([źródło](https://nmap.org/book/tcpip-ref.html)):

![]({{< baseurl >}}/img/2015/06/1.ip-header-ttl.png)

Dla systemów operacyjny z rodziny windows, to pole ma wartość `128` . Z kolei jeśli zaś chodzi o
chyba wszystkie dystrybucje linuxa, wliczając w to też i OpenWRT, jest to `64`. Tą wartość możemy
oczywiście zweryfikować i odczytać z parametrów systemowych za pomocą narzędzia `sysctl` . W
przypadku mojego debiana jest to:

    # sysctl -a
    ...
    net.ipv4.ip_default_ttl = 64

Niemniej jednak, powyższe ustawienie dotyczy wyłącznie pakietów tworzonych na określonej maszynie i
nie ma wpływu na pakiety przechodzące przez nią.

W przypadku forwardowania pakietów, routery zmniejszają wartość pola odpowiadającego za żywotność
pakietów o 1. Zatem jeśli jakiś komputer otrzymuje pakiet od maszyny linuxowej, to po przetworzeniu
go, zostanie on wysłany do następnego routera czy innego hosta w sieci z TTL o wartości 63. W
przypadku ISP obniżających TTL do 1, router otrzyma pakiet z TTL=1, po czym obniży wartość tego pola
do 0 i jeśli pakiet nie jest przeznaczony dla hosta w bieżącej sieci, to router musi go zniszczyć.

Jako, że wartość pola TTL w nagłówku IP jest powiązana z czasem, to te bardziej obciążone routery
mogą zmniejszyć tę wartość o więcej niż 1. Zdarza się to rzadko ale dobrze jest wiedzieć, że tego
typu sytuacja czasem ma miejsce.

## Ustawianie TTL

By poradzić sobie z tą niedogodnością, linuxowy iptables jest wyposażony w cel `TTL` . Jako, że
będziemy zmieniać właściwości pakietów, musimy nieco przerobić [wcześniej utworzony
firewall]({{< baseurl >}}/post/firewall-na-linuxowe-maszyny-klienckie/) . Konkretnie to musimy
wyedytować plik `iptables_mangle.sh`. Ma on mieć mniej więcej poniższą postać:

    #!/bin/sh

    ipt="$(which iptables) -t mangle"

    $ipt -F
    $ipt -X

    $ipt -I PREROUTING -i eth0 -m ttl --ttl-lt 2 -j TTL --ttl-inc 1
    $ipt -I PREROUTING -i br-lan -m ttl --ttl-eq 64 -j TTL --ttl-inc 1
    $ipt -I PREROUTING -i br-lan -m ttl --ttl-eq 128 -j TTL --ttl-inc 1

Pierwsza reguła sprawdza wszystkie pakiety przychodzące z internetu ( `eth0` ) i próbuje je
dopasować w oparciu o pole TTL w nagłówku IP. Jeśli jest ono ustawione na mniejszą wartość od 2, to
pole zostanie przepisane i numerek zwiększony o 1.

Pozostałe dwie linijki sprawdzają pakiety pochodzące z sieci lokalnej ( `br-lan` ) pod kątem
standardowych czasów życia pakietów dla systemów operacyjnych linux (64) oraz windows (128) i
podobnie jak wyżej, w przypadku dopasowania, pole TTL zostanie przepisane, a jego wartość zwiększona
o 1.

W przypadku doświadczania problemów z internetem i bycia pewnym, że to właśnie nasz ISP go blokuje
tym tanim chwytem, możemy wykorzystać jeną lub drugą metodę, zatem potrzebujemy albo pierwszej, albo
dwóch ostatnich reguł. To, które z nich znajdą zastosowanie w naszym przypadku możemy zweryfikować w
bardzo prosty sposób. Najpierw dodajemy pierwszą z powyższych reguł do iptables i jeśli pakiety będą
uderzać w tą regułę, oznaczać to będzie, że ISP obniżył TTL do wartości 1. W przeciwnym wypadku,
musimy skorzystać z pozostałych reguł.
