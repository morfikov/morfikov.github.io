---
author: Morfik
categories:
- Linux
date: "2015-06-29T21:35:12Z"
date_gmt: 2015-06-29 19:35:12 +0200
published: true
status: publish
tags:
- tcp
- sysctl
- ip
- systemd
- sieć
GHissueID: 100
title: Fragmentacja pakietu i zmiana wartości MTU
---

MTU (Maximum Transmission Unit) to maksymalna długość pakietu jaki może zostać przesłany przez sieć.
Może ona wynosić do 64KiB ale większość punktów sieciowych, takich jak routery, wymusza o wiele
mniejsze rozmiary pakietów. Domyślna wartość MTU dla protokołu Ethernet to 1500 bajtów, oczywiście
bez nagłówka warstwy fizycznej, który ma dodatkowe 14 bajtów. Czasami te standardowe ustawienia mogą
powodować problemy w przypadku pewnych konfiguracji sieci i gdy ich doświadczamy, przydałoby się
zmienić rozmiar MTU przesyłanych pakietów.

<!--more-->
## Fragmentacja pakietów

Maksymalny rozmiar segmentu TCP (MSS) określa jak dużo danych określony host jest w stanie
zaakceptować w pojedynczym datagramie IP. Generalnie rzecz biorąc, host A porównuje wartość MTU (-40
bajtów na nagłówki) swojego interfejsu wyjściowego z rozmiarem bufora odbiorczego i wybiera te
wartość, która jest niższa. Następnie tak określony MSS jest transmitowany w pakiecie `SYN` podczas
fazy three-way handshake. Host B porównuje otrzymany MSS z wartością jaką ma w MTU na swoim
interfejsie wyjściowym (-40 bajtów na nagłówki) i wybiera niższą wartość oraz ustawia ją jako MSS
dla pakietów przesyłanych do hosta A. I podobnie w drugą stronę, czyli host B porównuje MTU na swoim
interfejsie wyjściowym (-40 bajtów na nagłówki) z rozmiarem bufora odbiorczego i wybiera najniższą
wartość, po czym transmituje ją w pakiecie `SYN-ACK` na drugą stronę. Host A odbiera ją i porównuje
z wartością MTU swojego interfejsu wyjściowego (-40 bajtów na nagłówki). Wybiera niższą z nich i
ustawia jako MSS dla pakietów przesyłanych do hosta B.

Teoretycznie cały proces wygląda w porządku ale tylko w przypadku połączenia tych dwóch maszyn
bezpośrednio. Jeśli na drodze znajdzie się jakiś router, wtedy może wystąpić fragmentacja pakietu
IP, bo router nie jest świadom tej powyższej negocjacji i ustalonych wartości. Jeśli obie ze stron
wynegocjują sobie wartość MTU, która jest większa niż ta ustawiona na pośredniczącym routerze, mamy
pofragmentowane pakiety. A to z tego powodu, że router nie jest w stanie przetworzyć danych o
segmencie większym niż ma to zdefiniowane w swoich opcjach. Dlatego też taki pakiet musi zostać
podzielony na kilka mniejszych. Po dotarciu do odbiorcy, te fragmenty są składane w całość w oparciu
o numer identyfikacyjny pakietu, flagę `More fragments` oraz `offset` danych przed przesłaniem go do
wyższej warstwy. Wszystkie te informacje są podane w nagłówku IP:

![mtu-fragmentacja-pakietow](/img/2015/06/1.mtu-fragmentacja-pakietow.png#huge)

Fragmentacja pakietów pociąga za sobą kilka nieprzyjemnych następstw. Przede wszystkim, zwiększone
zapotrzebowanie na procesor i pamięć operacyjną, bo przecie trzeba te pakiety podzielić przed
wysłaniem, a potem je jeszcze złożyć na drugim końcu połączenia. Nie jest to znowu jakiś wielki
koszt. Głównym problemem jest forwardowanie pakietów, np. przez routery, które powinny przez ten
proces przechodzić jak najszybciej, a nie dodatkowo zawracać sobie głowę fragmentacją. Kolejny
problem widoczny jest przy retransmisji. Jeśli taki pofragmentowany kawałek ulegnie zgubieniu,
trzeba będzie przesłać wszystkie części jeszcze raz. Weźmy na przykład zasoby NFS. Jeśli ten system
plików w naszym przypadku ma rozmiar bloku 8192 bajtów, to po uwzględnieniu wszystkich nagłówków
(NFS, UDP, IP), taki datagram będzie ważył jakieś 8500 bajtów. Zatem trzeba będzie ten pakiet
podzielić na 6 kawałków (5x1500+1100). Jeśli któryś z nich zginie, np. z powodu zbyt dużego tłoku na
kablach, całe 8500 bajtów będzie trzeba zretransmitować, przy czym znów je będzie trzeba podzielić
przed wysłaniem. Do tego dochodzi jeszcze problem z bezpieczeństwem, gdzie przy pomocy [fragmentacji
można przepisywać pola nagłówka IP](https://en.wikipedia.org/wiki/IP_fragmentation_attack).

## Wyłączenie fragmentacji

Mając na uwadze wszystko powyższe, zaleca się ustawienie bitu `Don't fragment` w nagłówku IP, który
nie pozwoli na fragmentację pakietów. Możemy to zrobić za pomocą pliku `/etc/sysctl.conf` przez
dopisanie do niego tej poniższej linijki:

    net.ipv4.ip_no_pmtu_disc = 0

Path MTU Discovery decyduje o tym, czy połączenie TCP ma korzystać ze stałej, maksymalnej wartości
MTU, czy też ma próbować ją sobie samodzielnie ustalić. W tym celu stos IP ustawia w każdym
wychodzącym pakiecie bit `Don't fragment` . Gdy nadejdzie odpowiedni komunikat statusu ICMP,
wartość MTU jest zmieniana zgodnie z wiadomością.

Działanie tego mechanizmu może zaobserwować posługując się narzędziem `tracepath` :

    # tracepath -n wp.pl
     1?: [LOCALHOST]                                         pmtu 1500
     1:  192.168.1.1                                           1.669ms
     1:  192.168.1.1                                           1.600ms
     2:  192.168.22.51                                        20.157ms asymm  3
     3:  192.168.22.51                                        10.155ms
     4:  213.199.236.17                                       31.509ms
     5:  88.199.219.118                                       16.038ms asymm  8
     6:  153.19.0.154                                         24.546ms asymm  8
     7:  153.19.102.6                                         20.238ms asymm  8
     8:  212.77.96.69                                         22.116ms
     9:  212.77.100.101                                       18.338ms reached
         Resume: pmtu 1500 hops 9 back 9

Badanie MTU może się nie sprawdzać w przypadku źle skonfigurowanych firewalli, które odrzucają
wszelkie pakiety ICMP lub też w przypadku źle skonfigurowanych interfejsów, np. gdy oba końce
połączenia point-to-point nie zgadzają się na MTU.

## Zmiana wartości MTU

Aktualną wartość MTU możemy odczytać w statystykach każdego interfejsu sieciowego. W poniższym
przykładzie została ona odczytana za pomocą narzędzia `ip` dostępnego w pakiecie `iproute2` ale
można także posłużyć się narzędziem `ifconfig` :

    # ip link show dev bond0
    7: bond0:  mtu 1500 qdisc noqueue state UP mode DEFAULT group default
        link/ether 3c:4a:92:00:4c:5b brd ff:ff:ff:ff:ff:ff

Jeśli mamy problem z nawiązaniem połączenia z internetem, musimy pokombinować z wartością MTU, która
jest widoczna wyżej. Choć zwykle domyślna wartość powinna działać bez zarzutu. Odpalmy zatem jakiś
sniffer sieciowy (np. wireshark) i wydajmy w terminalu to poniższe polecenie:

    # ping -c 1 -s 1472 onet.pl
    PING onet.pl (213.180.141.140) 1472(1500) bytes of data.
    1480 bytes from sg1.any.onet.pl (213.180.141.140): icmp_seq=1 ttl=57 time=21.7 ms

    --- onet.pl ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 21.729/21.729/21.729/0.000 ms

Jeśli w snifferze widzimy, że pakiety ulegają fragmentacji, tak jak to widoczne jest na poniższym
obrazku:

![mtu-fragmentacja-pakietow-wireshark](/img/2015/06/2.mtu-fragmentacja-pakietow-wireshark.png#huge)

Oznacza to, że MTU o wartości 1500 jest nieodpowiedni. Co prawda, wydaliśmy polecenie z parametrem
`-s 1472` ale do tego dochodzi jeszcze nagłówek UDP ( `8` bajtów), no i oczywiście nagłówki IP
(kolejne `20` bajtów) i nagłówek ethernetowy ( `14` bajtów). Łącznie daje nam to 1514 bajtów, z tym,
że w MTU nie wliczany jest nagłówek warstwy fizycznej, dlatego MTU wynosi 1500 bajtów.

W przypadku gdy już dobraliśmy optymalną wartość MTU, trzeba ją jakoś ustawić w systemie. W
przypadku korzystania z initu systemd, trzeba stworzyć plik `/etc/systemd/network/05-bond0.link` ,
np. o poniższej treści:

    [Match]
    Driver=bonding

    [Link]
    Description=MTU for bond interface
    MTUBytes=1440
    BitsPerSecond=100M

Oczywiście dopasowań może być wiele, ja wykorzystuje do tego sterowniki kart sieciowych/wirtualnych
interfejsów. W przypadku gdy chcemy ustalić jaki sterownik jest od naszej karty, wtedy korzystamy z
`ethtool` :

    # ethtool -i eth1
    driver: r8169
    ...

I to ten sterownik trzeba podać w pliku `.link` . Po zresetowaniu maszyny, MTU powinno zostać
prawidłowo ustawione.

    # ip link show bond0
    7: bond0:  mtu 1440 qdisc noqueue state UP mode DEFAULT group default
        link/ether 3c:4a:92:00:4c:5b brd ff:ff:ff:ff:ff:ff

W przypadku korzystania z pliku `/etc/network/interfaces` , wystarczy dodać poniższą linijkę do
bloku konfiguracyjnego interfejsu:

    auto bond0
    iface bond0 inet dhcp
        ...
        mtu 1440
        ...
