---
author: Morfik
categories:
- Linux
date: "2015-06-24T13:57:02Z"
date_gmt: 2015-06-24 11:57:02 +0200
published: true
status: publish
tags:
- iptables
- sysctl
title: Udostępnianie połączenia internetowego
---

Każdy z nas ma router w domu, który potrafi rozdzielić sygnał na szereg komputerów w naszej sieci
lokalnej. Jeśli z jakichś powodów nie chcemy posiadać w domu tego cuda techniki i chcielibyśmy mieć
możliwość udostępniania połączenia internetowego za pośrednictwem innego komputera, to nic nie stoi
nam na przeszkodzie by to zrobić. Taki komputer stacjonarny niczym się nie różni od routera, no może
za wyjątkiem poboru prądu.

<!--more-->
## Forwardowanie pakietów

System operacyjny linux ma domyślnie zablokowaną możliwość forwardowania pakietów. Musimy pierw
włączyć tę funkcję przy pomocy pliku `/etc/sysctl.conf` . By to zrobić, dopisujemy w nim poniższą
linijkę:

    net.ipv4.ip_forward = 1

Po czym wydajemy poniższe polecenie:

    # sysctl -p

## Udostępnianie połączenia innej maszynie

By połączyć się z internetem z maszyny znajdującej się za tą, która ma go udostępniać (zwykle jest
to router), musimy nakazać temu komputerowi, by w miejsce adresu w nagłówku pakietu wpisywał swój
adres źródłowy ale tylko w przypadku gdy pakiety nie są adresowane bezpośrednio do niego. W tym celu
musimy stworzyć regułę dla iptables.

Możemy posłużyć się dwoma targetami: `-j SNAT` oraz `-j MASQUERADE` . Pierwszy z nich powinien być
używany w przypadku jeśli udostępnianie połączenia internetowego odbywa się za pomocą maszyny,
która ma przypisany stały adres IP. Drugi zaś, jeśli ten [adres IP się
zmienia](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html#MASQUERADETARGET). Choć
można używać także `-j MASQUERADE` ze stałym IP.

Zakładając, że mamy już wstępnie [skonfigurowany
firewall](/post/firewall-na-linuxowe-maszyny-klienckie/), tworzymy plik
`iptables_nat.sh` i wrzucamy do niego ten poniższy kod:

    #!/bin/sh

    ipt="$(which iptables) -t nat"

    $ipt -F
    $ipt -X

    $ipt -A POSTROUTING -o eth0 -s 192.168.10.0/24 -d 0/0 -j SNAT --to-source 192.168.1.150
    #$ipt -A POSTROUTING -o eth0 -s 192.168.10.0/24 -d 0/0 -j MASQUERADE

Kluczowe w ustawianiu translacji adresów jest określenie interfejsu, przez który pakiety mają
wychodzić do internetu. W tym przypadku jest to interfejs `eth0` . Jeśli korzystamy z `-j SNAT` ,
musimy ustawić dodatkowo adres IP, który został przypisany temu interfejsowi.

By móc się połączyć ze światem oraz z innymi komputerami w sieci, musimy jeszcze dodać odpowiednie
wpisy w łańcuchu `FORWARD`, w tablicy `filter` . Edytujemy zatem plik `iptables_filter.sh` . Gdzieś
w nim powinniśmy mieć te oto poniższe reguły:

    $ipt -N fw-interfaces
    $ipt -N fw-open
    ...
    $ipt -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    $ipt -A FORWARD -m conntrack --ctstate INVALID -j DROP
    $ipt -A FORWARD -m conntrack --ctstate NEW -j fw-interfaces
    $ipt -A FORWARD -m conntrack --ctstate NEW -j fw-open
    $ipt -A FORWARD -j REJECT --reject-with icmp-host-unreachable
    ...

Oznaczają one, że zapora będzie blokować pakiety w stanie `INVALID` , przepuszczać pakiety w stanie
`RELATED` i `ESTABLISHED` oraz, że pakiety w stanie `NEW` będą trafiać kolejno do dwóch łańcuchów:
`fw-interfaces` oraz `fw-open` . W pierwszym z tych dodatkowych łańcuchów są definiowane interfejsy
maszyny, między którymi pakiety mogą przechodzić, np. z `eth1` do `eth0` , co zapewni łączność z
internetem. W drugim łańcuchu zaś są definiowane regułki przekierowujące ruch do określonych maszyn
za głównym komputerem.

Brakuje nam jeszcze odpowiedniej reguły, która zezwoli na przepływ pakietów między interfejsami
maszyny, która realizuje udostępnianie połączenia internetowego. Dodajemy zatem do pliku
`iptables_filter.sh` tę poniższą regułę:

    ...
    $ipt -A fw-interfaces -i eth1 -o eth0 -s 192.168.10.0/24 -d 0/0 -j ACCEPT
    ...

Teraz powinno już działać połączenie z internetem. By zweryfikować czy aby faktycznie wszystko
działa jak należy, przy pomocy polecenia `ping` odpytujemy jakiś host w internecie oraz adres
interfejsu lokalnego maszyny udostępniającej połączenie. Dobrze jest to robić po adresie IP, by
wyeliminować ewentualne problemy związane z resolverem DNS.
