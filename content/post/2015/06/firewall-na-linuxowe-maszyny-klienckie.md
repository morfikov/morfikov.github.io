---
author: Morfik
categories:
- Linux
date: "2015-06-22T23:33:58Z"
date_gmt: 2015-06-22 21:33:58 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
- systemd
title: Firewall na linux'owe maszyny klienckie
---

Prędzej czy później, każdy z nas będzie musiał zabezpieczyć dostęp do swojego komputera w sieci. Do
tego zadania na stacjach z linux'em jest oddelegowane narzędzie `iptables` wraz z szeregiem
dodatków, jak np. `ipset` . By zbudować solidny firewall nie potrzeba zbytnio się wysilać. Niemniej
jednak, temat zapory linux'owej jest [bardzo
rozległy](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html) i nie będziemy tutaj
opisywać wszystkich możliwych scenariuszy jej zastosowania. Zamiast tego skupimy się jedynie na
bazowym skrypcie, który można rozbudować bez przeszkód i dostosować go zarówno pod maszyny klienckie
jak i te serwerowe.

<!--more-->
## Przepływ pakietów przez firewall

By firewall realizował wyznaczone mu zadania, trzeba dodać do niego szereg reguł. Problem w tym, że
iptables składa się z kilku tablic. Do naszej dyspozycji są: `raw` , `magnle` , `nat` i `filter` .
Ponadto, każda z tablic ma pewien zestaw łańcuchów, w których skład mogą wchodzić: `INPUT` ,
`OUTPUT` , `FORWARD` , `PREROUTING` oraz `POSTROUTING` . Pakiety przychodzące, wychodzące i
przechodzące przez dowolną maszynę znajdującą się w sieci, bez względu na to czy to router, serwer,
czy klient, robią to w ściśle określonej kolejności. Najlepiej to zobrazować poniższą fotką:

![](/img/2015/06/1.firewall-iptables-przeplyw-pakietow.png#big)

Pakiety dzielimy na te otrzymane ( `RX` ) oraz te wysłane ( `TX` ). Każdy z nich przechodzi przez
powyższy schemat, z tym, że nieco inną drogą. Najprostsza sytuacja ma miejsce gdy pakiet jest
adresowany do naszej maszyny. Podobnie gdy to my wysyłamy pakiet pod adresem jakiegoś konkretnego
komputera w sieci. Dodatkowo, pakiet potrafi przechodzić przez szereg innych maszyn po drodze. Nie
jest on w takim przypadku bezpośrednio adresowany, np. do routera, ale by osiągnąć jakiś host w
sieci, musi przez niego przejść.

### Tablica filter

W tablicy `filter` , jak sama nazwa mówi, będziemy dokonywać głównego filtrowania pakietów i to w
dużej mierze tutaj zostaną one zaakceptowane lub też i odrzucone. Sama tablica ma trzy łańcuchy i
jeśli mamy do czynienia ze zwykłą maszyną kliencką, to będą nas interesować jedynie dwa z nich:
`INPUT` oraz `OUTPUT` . Jeżeli mamy w domu router, to wtedy również będzie wchodzić w grę łańcuch
`FORWARD` . Podobnie sprawa ma się w przypadku gdy udostępniamy połączenie sieciowe za pomocą
zwykłego komputera innym maszynom. Na dobrą sprawę, to wtedy nasz komputer staje się routerem i
wszystkie reguły, które aplikują się do routerów, będą mieć także zastosowanie na takiej maszynie
udostępniającej, np. internet w pokoju obok.

### Tablica nat

I tu właśnie zaczynają się problemy. Jak wspomniałem na wstępnie, mamy do dyspozycji cztery tablice,
a nie jedną. Jeśli jesteśmy już przy łańcuchu `FORWARD` i udostępnianiu połączenia, to nie sposób
ominąć tablicę `nat` . Jest ona bardzo podobna do tablicy `filter` , z tym, że nie posiada łańcucha
`FORWARD` , a zamiast niego ma dwa inne: `PREROUTING` i `POSTROUTING` .

Przez tę tablicę przechodzą jedynie pakiety rozpoczynające nowe połączenia i to w niej będziemy
określać translację adresów, dzięki czemu, jesteśmy w stanie podzielić jeden adres IP między szereg
hostów w sieci.

### Tablica mangle

Tej tablicy się praktycznie nie wykorzystuje w procesie filtrowania. Jej zadaniem nie jest przemiana
pakietów. Jeśli musimy dostosować TTL pakietu czy oznaczyć pakiety markerami, to robimy to w tej
tablicy.

### Tablica raw

Jej głównym zadaniem jest wyłączenie śledzenia określonych połączeń, gdyż pakiety trafiające do tej
tablicy robią to zanim mechanizm conntrack zacznie działać. Można ją oczywiście wykorzystać w
procesie filtrowania na bardzo wczesnym etapie analizy pakietu. Jeśli znamy adres IP z jakiego
nadszedł pakiet i chcemy zablokować mu dostęp, to nic nie stoi na przeszkodzie by to zrobić w tej
tablicy, choć o wiele lepszym miejscem do tego jest tablica `filter` .

## Stany pakietów na zaporze

My tutaj zajmiemy się budową zapory, która będzie operować w oparciu o stany pakietów. Iptables
potrafi rozróżnić cztery stany połączeń protokołu TCP: `NEW` , `ESTABLISHED` , `RELATED` oraz
`INVALID` . Skąd system wie, że dany pakiet ma zostać przypisany do połączenia w stanie `NEW` , a
nie `INVALID` ?

Każde otwarte połączenie okupuje odpowiednie gniazdo (socket), w skład którego wchodzi lokalny
adres, lokalny port, zdalny adres, zdalny port oraz protokół. Te dane są śledzone przez kernel w
tablicy conntrack'a (do wglądu w `/proc/net/ip_conntrack` lub `/proc/net/nf_conntrack` ) i to na
postawie tych wpisów ustalany jest status pakietu. Powiedzmy zatem kilka słów o [poszczególnych
stanach połączeń](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html#STATEMACHINE):

  - `NEW` identyfikuje pakiety rozpoczynające nowe połączenia. Jest to pierwszy pakiet, który został
    dostrzeżony przez moduł conntrack. Zawiera on ustawioną flagę `SYN` .
  - `ESTABLISHED` identyfikuje pakiety, które należą już do istniejących połączeń.
  - `RELATED` określa pakiety, które są powiązane ze stanem `ESTABLISHED` . Aby połączenie mogło być
    w stanie `RELATED` , musi pierw zostać zestawione połączenie w stanie `ESTABLISHED` . Następnie
    to ustanowione połączenie stworzy połączenie powiązane. Przykładem może być protokół FTP, gdzie
    sesja może odbywać się na porcie 21 ale przepływ danych odbywa się na porcie 20.
  - `INVALID` jest przypisywany wszystkim połączeniom niepasującym do tych trzech powyższych stanów.
    Każdy pakiet TCP ma ustawione flagi, które określają dokładnie co to jest za rodzaj pakietu, np.
    czy ma on na celu otwarcie nowego połączenia czy też jest to potwierdzenie przesłania pewnej
    ilości danych z jednego hosta do drugiego. Są pewne określone sekwencje flag, które umożliwiają
    komunikację sieciową. Niemniej jednak, można tymi flagami dowolnie manipulować i tak wykutymi
    pakietami można dokonywać skanowania portów w poszukiwaniu usług sieciowych, np. SSH, co zwykle
    może się skończyć atakiem, w przypadku gdy określona usługa zostanie przez atakującego wykryta.
    Stan `INVALID` zawiera wszystkie te niepoprawne kombinacje flag.

Biorąc pod uwagę te stany połączeń, musimy zatroszczyć się także o [czyszczenie wpisów w tablicy
conntrack'a](/post/jak-wyczyscic-tablice-conntrack-w-debianie/). Nie jest to
wprawdzie wymagana opcja ale znacznie poprawia bezpieczeństwo systemu.

## Jak zbudować dobry firewall

Nasz firewall powinien zezwalać na przetwarzanie pakietów w stanie `ESTABLISHED` oraz `RELATED` ,
zrzucać pakiety w stanie `INVALID` i manipulować pakietami w stanie `NEW` . Na każdej maszynie,
która ma dostęp do internetu, firewall buduje się w ten sam sposób. Najpierw blokujemy cały ruch
pochodzący z zewnątrz sieci. Blokujemy także pakiety przechodzące przez dany komputer, czyli te,
których przeznaczeniem nie jest ta konkretna maszyna. Z kolei pakiety generowane przez tego hosta
powinny bez problemu mieć wyjście na świat, chyba, że projektujemy jakiś wymyślny serwer, który
będzie nadawał tylko na określonych portach. Tak czy inaczej, my skupimy się na zbudowaniu zapory,
która zabezpieczy dostęp do komputera z zewnątrz.

Tworzymy zatem kilka plików skryptu, np. w katalogu `/etc/filtr/` . Struktura tego katalogu powinna
wyglądać mniej więcej tak:

    # ls -al
    total 72K
    drwxr-xr-x   2 root root 4.0K 2015-05-14 11:37:32 ./
    drwxr-xr-x 157 root root    12K 2015-06-24 07:39:50 ../
    -rwxr-----   1 root root 1.8K 2015-01-27 03:53:58 base.sh*
    -rwxr-----   1 root root  822 2015-01-26 22:51:59 ip6tables_filter.sh*
    -rwxr-----   1 root root   47 2015-01-26 22:51:29 ip6tables_mangle.sh*
    -rwxr-----   1 root root   44 2015-01-26 22:51:23 ip6tables_nat.sh*
    -rwxr-----   1 root root   44 2015-01-26 22:51:18 ip6tables_raw.sh*
    -rwxr-----   1 root root 8.6K 2015-03-05 21:13:24 iptables_filter.sh*
    -rwxr-----   1 root root 1.5K 2015-03-05 21:13:24 iptables_mangle.sh*
    -rwxr-----   1 root root  356 2015-03-05 21:13:23 iptables_nat.sh*
    -rwxr-----   1 root root 1.8K 2015-01-26 22:40:26 iptables_raw.sh*

Pliki mające w nazwie `6` są od filtra protokołu IPv6 . Każdy z plików (za wyjątkiem base.sh) ma
dodatkowo nazwę tablicy, której reguły będzie przechowywał. Natomiast `base.sh` , to podstawowy
firewall, którego reguły będą aplikowane, np. przy resetowaniu głównego filtra.

Plik `iptables_filter.sh` :

    #!/bin/sh

    ipt="$(which iptables) -t filter"

    $ipt -P INPUT DROP
    $ipt -P FORWARD DROP
    $ipt -P OUTPUT ACCEPT

    $ipt -F
    $ipt -X

    $ipt -N tcp
    $ipt -N udp
    $ipt -N icmp_in
    $ipt -N fw-interfaces
    $ipt -N fw-open

    $ipt -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    $ipt -A INPUT -i lo -j ACCEPT
    $ipt -A INPUT -m conntrack --ctstate INVALID -j DROP
    $ipt -A INPUT -p icmp -m conntrack --ctstate NEW -j icmp_in
    $ipt -A INPUT -p udp -m conntrack --ctstate NEW -j udp
    $ipt -A INPUT -p tcp ! --syn -m conntrack --ctstate NEW -j DROP -m comment --comment "Connections not started by SYN"
    $ipt -A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j tcp

    $ipt -A INPUT -p tcp -j REJECT --reject-with tcp-reset
    $ipt -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
    $ipt -A INPUT -j REJECT --reject-with icmp-proto-unreachable

    $ipt -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    $ipt -A FORWARD -m conntrack --ctstate INVALID -j DROP
    $ipt -A FORWARD -m conntrack --ctstate NEW -j fw-interfaces
    $ipt -A FORWARD -m conntrack --ctstate NEW -j fw-open
    $ipt -A FORWARD -j REJECT --reject-with icmp-host-unreachable

    $ipt -A icmp_in -p icmp -j LOG --log-prefix "* IPTABLES: ICMP_NEW * "
    $ipt -A icmp_in -p icmp -j DROP

Plik `ip6tables_filter.sh` :

    #!/bin/sh

    ip6t="$(which ip6tables) -t filter"

    $ip6t -P INPUT DROP
    $ip6t -P FORWARD DROP
    $ip6t -P OUTPUT ACCEPT
    $ip6t -F
    $ip6t -X
    $ip6t -N tcp
    $ip6t -N udp
    $ip6t -N icmp

    $ip6t -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    $ip6t -A INPUT -i lo -j ACCEPT
    $ip6t -A INPUT -m conntrack --ctstate INVALID -j DROP
    $ip6t -A INPUT -p icmpv6 -m conntrack --ctstate NEW -j icmp
    $ip6t -A INPUT -p udp -m conntrack --ctstate NEW -j udp
    $ip6t -A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j tcp
    $ip6t -A INPUT -p tcp -j REJECT --reject-with tcp-reset
    $ip6t -A INPUT -p udp -j REJECT --reject-with icmp6-port-unreachable

    $ip6t -A icmp -p icmpv6 -j ACCEPT -m limit --limit 6/s --limit-burst 10
    $ip6t -A icmp -p icmpv6 -j DROP

Plik `base.sh` :

    #!/bin/sh

    iptables_stop()
    {
          ipt="$(which iptables)"
          ip6t="$(which ip6tables)"

          $ipt -P INPUT DROP
          $ipt -P FORWARD DROP
          $ipt -P OUTPUT ACCEPT

          $ip6t -P INPUT DROP
          $ip6t -P FORWARD DROP
          $ip6t -P OUTPUT ACCEPT

          for iptable in $ipt $ip6t
          do
                for table in \
                      "-t raw" \
                      "-t mangle" \
                      "-t filter" \
                      "-t nat"
                do
                      $iptable $table -F
                      $iptable $table -X
                done
          done

          $ipt -t filter -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
          $ipt -t filter -A INPUT -i lo -j ACCEPT
          $ipt -t filter -A INPUT -m conntrack --ctstate INVALID -j DROP
          $ipt -t filter -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
          $ipt -t filter -A INPUT -p tcp -j REJECT --reject-with tcp-reset
          $ipt -t filter -A INPUT -j REJECT --reject-with icmp-proto-unreachable

          $ip6t -t filter -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
          $ip6t -t filter -A INPUT -i lo -j ACCEPT
          $ip6t -t filter -A INPUT -m conntrack --ctstate INVALID -j DROP
          $ip6t -t filter -A INPUT -p tcp -j REJECT --reject-with tcp-reset
          $ip6t -t filter -A INPUT -p udp -j REJECT --reject-with icmp6-port-unreachable
    }

    iptables_stop

Generalnie rzecz biorąc przepuszczamy ruch na interfejsie `lo` . Jeśli jakieś procesy potrzebują
adresu z klasy `127.0.0.0/8` , to będą mogły z niego swobodnie korzystać. Żadna maszyna za wyjątkiem
hosta lokalnego (localhost) nie może nawiązać połączenia z tą klasą adresów, dlatego też można ją
bez wahania odhaczyć. W przypadku gdybyśmy zablokowali ten ruch, pewne procesy systemowe mogą
przestać działać.

Stworzyliśmy także kilka dodatkowych łańcuchów: `tcp`, `udp`, `icmp_in`, `fw-interfaces`,
`fw-open` . Pierwsze trzy rozgraniczają trzy główne protokoły gdzie będą kierowane określone
pakiety. Z kolei czwarty z nich odpowiada za lokalne interfejsy, które mogą wysyłać pakiety w świat.
To na wypadek udostępniania połączenia. Z kolei ostatni łańcuch jest od otwierania i przekierowania
portów na maszyny lokalne. Cały pozostały ruch będzie trafiał z automatu na reguły zajmujące się
zwracaniem odpowiednich komunikatów hostom, które próbowały się z nami połączyć.

Ta powyższa konfiguracja zapory jest bardzo podstawowa i nadaje się jedynie na stację kliencką. Nie
akceptuje ona żadnych nowych połączeń z sieci. Wszystkie z nich muszą pierw zostać zainicjowane
lokalnie, co czyni naszą maszynę bezpieczną, chyba, że ktoś nam wgra wirusa, trojana czy inny syf,
który będzie nawiązywał zdalne połączenia automatycznie gdy tylko podłączymy się do sieci.

Ten firewall ma budowę modularną, tj. możemy tworzyć kolejne pliki w zależności od wersji protokołu
IP oraz tablicy, w której chcemy umieszczać dodatkowe reguły. Możemy także bez problemu podpiąć
dodatkowe moduły i dopisać konfigurację np. dla `ipset` .

## Usługa dla systemd

By odpalić taki firewall, potrzebujemy jeszcze odpowiedniego pliku `.service` , który zainicjuje te
powyższe skrypty. Tworzymy zatem `/etc/systemd/system/firewall.service` i dodajemy w nim poniższy
kod:

    [Unit]
    Description=firewall
    Documentation=man:iptables
    DefaultDependencies=no
    Wants=network-pre.target systemd-modules-load.service
    Before=network-pre.target
    After=systemd-modules-load.service

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/bin/sh -c "/etc/filtr/iptables_raw.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_mangle.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_nat.sh"
    ExecStart=/bin/sh -c "/etc/filtr/iptables_filter.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_raw.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_mangle.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_nat.sh"
    ExecStart=/bin/sh -c "/etc/filtr/ip6tables_filter.sh"
    ExecStop=/bin/sh -c "/etc/filtr/base.sh"

    [Install]
    WantedBy=multi-user.target

Dodajemy tę usługę do autostartu systemu i odpalamy ją:

    # systemctl daemon-reload
    systemctl enable firewall.service
    systemctl start firewall.service

Powyższa baza reguł została napisane w celu stworzenia zapory ściśle klienckiej ale ten schemat może
posłużyć także do udostępniania usług na danej maszynie, jak u również możemy przy jego pomocy
udostępnić połączenie sieciowe innym komputerom w sieci lokalnej.
