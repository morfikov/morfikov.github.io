---
author: Morfik
categories:
- Linux
date: "2016-05-18T15:17:25Z"
date_gmt: 2016-05-18 13:17:25 +0200
published: true
status: publish
tags:
- iptables
title: Target MARK w iptables (sumowanie oznaczeń)
---

Pakiety i połączenia można oznaczać w `iptables` za pomocą target `MARK` oraz `CONNMARK` . Ta
właściwość tego filtra jest znana chyba większość użytkowników linux'a. Z reguły nie interesujemy
się zbytnio szczegółami oznaczania pakietów/połączeń, bo zwykle jest ono przeprowadzane prawidłowo.
Niemniej jednak, są przypadki, w których markery nie będą ustawiane poprawnie. Chodzi o sytuacje,
gdzie mamy do czynienia z kilkoma mechanizmami oznaczającymi pakiety. Dla przykładu weźmy sobie
kształtowanie ruchu za pomocą narzędzia `tc` oraz failover czy load balancing łącza w oparciu o
różne tablice routingu. W obu tych mechanizmach zwykle używa się markowania pakietów w `iptables`
. Co się jednak dzieje z takim pakietem, gdy przechodzi przez reguły zarówno pierwszego jak i
drugiego mechanizmu? To właśnie spróbujemy ustalić w tym wpisie.

<!--more-->
## Różnica między target MARK i CONNMARK oraz domyślna maska

We wstępie wspomniałem o oznaczaniu pojedynczych pakietów oraz całych połączeń. W `iptables`
wykorzystuje się do tych celów dwa targety:
[MARK](http://ipset.netfilter.org/iptables-extensions.man.html#lbDE) oraz
[CONNMARK](http://ipset.netfilter.org/iptables-extensions.man.html#lbCS). Target `MARK` nakłada
oznaczenie na pojedynczy pakiet. Używa się go głównie przy dopasowaniu nowych połączeń, tj. pakietów
w stanie `NEW`. Natomiast z targetu `CONNMARK` korzysta się przy zapisywaniu i odtwarzaniu oznaczeń
w przypadku pakietów przypisanych do nawiązanego już połączenia. W ten sposób jesteśmy w stanie
odciążyć maszynę i zapobiec ciągłemu przepisywaniu oznaczeń w przypadku połączeń, które już
posiadają znacznik. Te markery są widoczne w tablicy conntrack'a pod `/proc/net/nf_conntrack` lub
`/proc/net/ip_conntrack` , w zależności od wykorzystywanego modułu do śledzenia połączeń. Poniżej
przykład takiego wpisu:

    ipv4     2 tcp      6 431994 ESTABLISHED src=192.168.1.150 dst=52.11.112.70 sport=54280 dport=443
    src=52.11.112.70 dst=192.168.1.150 sport=443 dport=54280 [ASSURED] mark=2 zone=0 use=2

W tym przypadku, mamy połączenie między maszynami o adresach 192.168.1.150 oraz 52.11.112.70 .
Widzimy wyżej wyraźnie, że został mu przypisany marker `2` . W jaki sposób oznacza się takie
połączenia? Poniżej zostaną rozpisane dwie bazy markujące, które rzucą nam nieco światła na ten
mechanizm oznaczania pakietów i połączeń. Na sam początek stwórzmy sobie w `iptables` kilka
łańcuchów w tablicy `mangle` . Do nich będzie przekierowany ruch:

    # iptables -t mangle -N base-1
    # iptables -t mangle -N base-2
    # iptables -t mangle -N marking-1
    # iptables -t mangle -N marking-2

    # iptables -t mangle -A PREROUTING -j base-1
    # iptables -t mangle -A PREROUTING -j base-2
    # iptables -t mangle -A POSTROUTING -j base-1
    # iptables -t mangle -A POSTROUTING -j base-2

W zależności od przeznaczenia oznaczeń (kontrola ruchu, routing) trzeba dobrać odpowiednie łańcuchy.
Chodzi generalnie o to, że pakiety w `iptables` przechodzą przez ten filtr w pewien określony
sposób. Jest on przedstawiony na poniższej fotce
([źródło](https://commons.wikimedia.org/wiki/File:Netfilter-packet-flow.svg)):

[![1.iptables-przeplyw-pakietow-netfilter]({{< baseurl >}}/img/2016/05/1.iptables-przeplyw-pakietow-netfilter-1024x335.png)]({{< baseurl >}}/img/2016/05/1.iptables-przeplyw-pakietow-netfilter.png)

Teraz w łańcuchach `base-1` oraz `base-2` umieszczamy po bazie oznaczającej całe połączenia:

Pierwsza baza markująca:

    # iptables -t mangle -A base-1 -j CONNMARK --restore-mark --nfmask 0x0000ff00 --ctmask 0x0000ff00
    # iptables -t mangle -A base-1 -m mark ! --mark 0x00000000/0x0000ff00 -j RETURN
    # iptables -t mangle -A base-1 -m mark --mark 0x00000000/0x0000ff00 -j marking-1
    # iptables -t mangle -A base-1 -j CONNMARK --save-mark --nfmask 0x0000ff00 --ctmask 0x0000ff00

Druga baza markująca:

    # iptables -t mangle -A base-2 -j CONNMARK --restore-mark --nfmask 0x000000ff --ctmask 0x000000ff
    # iptables -t mangle -A base-2 -m mark ! --mark 0x00000000/0x000000ff -j RETURN
    # iptables -t mangle -A base-2 -m mark --mark 0x00000000/0x000000ff -j marking-2
    # iptables -t mangle -A base-2 -j CONNMARK --save-mark --nfmask 0x000000ff --ctmask 0x000000ff

Dla lepszego zrozumienia tematu korzystałem z pełnych masek i oznaczeń. Chodzi o to, że target
`MARK` i `CONNMARK` są w stanie nakładać oznaczenia 32-bitowe. Zapis HEX to 0x00000000 do
0xffffffff. Zarówno maska jak i oznaczenie może przybrać wartość z tego przedziału. Jeśli nie
określamy maski, to domyślnie jest ustawiana 0xffffffff . Efektem tego jest przepisywanie oznaczeń.
W każdej z tych dwu baz wykorzystujemy inną maskę. W pierwszej mamy 0x0000ff00 , w drugiej zaś
0x000000ff . Skrótowo zawsze można zapisać je z pominięciem poprzedzających zer, tj. 0xff00 oraz
0xff. Tego typu oznaczenia będą zwracane przez `iptables` podczas listowania reguł.

Tak stworzony mechanizm markujący działa w następujący sposób. Pakiet w stanie `NEW` nie mający
jeszcze nałożonego oznaczenia (0x00000000, 0x0) dociera do tablicy `mangle` do łańcucha `PREROUTING`
. Tam wpada do pierwszej bazy markującej i przechodzi przez pierwszą regułę. Jako, że ten pakiet ma
oznaczenie 0x00000000, to ta pierwsza reguła nic zbytnio z nim nie robi. Pakiet po przejściu przez
nią dalej ma oznaczenie 0x00000000 . Druga reguła dopasowuje pakiet w oparciu o moduł `-m mark` i
jeśli pakiet ma inne oznaczenie niż 0x00000000, to zostaje zwrócony do łańcucha wyżej, czyli nie
przechodzi już przez pozostałe reguły tej bazy markującej. Trzecia reguła również porównuje
ustawione oznaczenie. Jeśli pakiet nie jest oznaczony, tzn. ma 0x00000000, to jest przesyłany do
łańcucha `marking-1` , w którym będą się znajdować reguły markujące. Te reguły nas na razie nie
interesują. Po przejściu przez łańcuch `marking-1` , pakiet trafia do ostatniej reguły w pierwszej
bazie markującej. To tutaj właśnie jest zapisywane oznaczenie dla całego połączenia, które
widzieliśmy wyżej w tablicy conntrack'a. Niemniej jednak, zapisywane są tylko bity określone przez
maskę 0x0000ff00. Podobnie sprawa wygląda w przypadku drugiej bazy markującej. W tym przypadku
wykorzystujemy maskę 0x000000ff .

Zatem w obu przypadkach maski są inne, czyli inne bity będą brane pod uwagę. W efekcie, pakiet
przechodząc przez obie bazy będzie miał ustawione oznaczenie sumaryczne, tj. znacznik ustawiony w
drugiej bazie zostanie dodany do tego oznaczenia nałożonego przez pierwszą bazę. W ten sposób
mechanizmy operujące na tych bazach mogą oznaczać pakiety niezależnie i operować na tak ustawionych
oznaczeniach bez przeszkadzania sobie.

## Reguły markujące

Te powyższe bazy markujące są ustawione w taki sposób, by pakiet, który nie został jeszcze
oznaczony, przechodził przez reguły markujące. W pierwszej bazie mamy zdefiniowany łańcuch
`marking-1` , w drugiej zaś `marking-2` . W tych łańcuchach musimy teraz umieścić reguły. Dla
przykładu dodajmy tam te poniższe:

    # iptables -t mangle -A marking-1 -p icmp -m icmp --icmp-type 8 -j MARK --set-xmark 0x00000100/0x0000ff00

    # iptables -t mangle -A marking-2 -d 8.8.8.8 -j MARK --set-xmark 0x00000002/0x000000ff
    # iptables -t mangle -A marking-2 -d 8.8.4.4 -j MARK --set-xmark 0x00000001/0x000000ff

W pierwszej bazie markującej dodaliśmy regułę, która oznacza pakiety protokołu ICPM echo-request.
Będzie im nadawane oznaczenie 0x100, czyli 256 po przeliczeniu na ludzki język. To oznaczenie po
przejściu przez tę bazę, zostanie przypisane do całego połączenia. Następnie pakiety trafią do
drugiej bazy markującej i przejdą przez dwie ostatnie reguły widoczne powyżej. W zależności od
miejsca przeznaczenia ping'u, pakiety zostaną tutaj oznaczone 0x1 lub 0x2. Jako, że korzystamy z
innej maski, to te wartości zostaną dodane do tej poprzedniej. Czyli pakiet po przejściu przez drugą
bazę markującą będzie miał oznaczenie 0x101 lub 0x102 , czyli 257 lub 258. Poniżej prezentuje się
listing tablicy `mangle` :

![]({{< baseurl >}}/img/2016/05/1.tablica-mangle-iptables-mark-oznaczanie-pakietow.png)

Sprawdźmy czy ten mechanizm działa jak należy. Odpalamy zatem terminal i wysyłamy żądania `ping` do
kilku serwisów:

[![2.ping-iptables-mark-oznaczanie-pakietow-conntrack]({{< baseurl >}}/img/2016/05/2.ping-iptables-mark-oznaczanie-pakietow-conntrack-1024x407.png)]({{< baseurl >}}/img/2016/05/2.ping-iptables-mark-oznaczanie-pakietow-conntrack.png)

W górnym okienku mamy podgląd tablicy conntrack'a. W pozostałych czterech oknach mamy ping'i.
Widzimy, że tablica conntrack'a jest filtrowana i zwracane są wyniki ze znacznikiem innym niż 0.
Wszystkie żądania `ping` zostały oznaczone znacznikiem minimum 0x100, czyli minim 256. Dwa z wpisów
conntrack'a mają przypisany mark 0x101 oraz 0x102, czyli 257 i 268 w zapisie dziesiętnym. Są to
połączenia, które zostały dopasowane w obu bazach markujących. Pozostałe zaś zostały dopasowane
tylko w pierwszej bazie i dlatego ich oznaczenie wskazuje na 0x100, czyli 256. Jeśli podejrzymy
jeszcze raz reguły w tablicy `mangle` , powinniśmy zobaczyć szereg dopasowań:

![]({{< baseurl >}}/img/2016/05/3.tablica-mangle-iptables-mark-oznaczanie-pakietow.png)

Mamy cztery sesje `ping` . W łańcuchu `marking-1` mamy cztery dopasowania, zaś w łańcuchu
`marking-2` mamy dwa dopasowania. W łańcuchach `base-1` oraz `base-2` w regule drugiej (ta z
`RETURN`) również mamy kilka dopasowań, co wskazuje, że oznaczenie części pakietów różni się od 0x0
na pozycjach bitów określonych przez maski. Ilość dopasowanych pakietów w obu przypadkach się różni.
W pierwszej bazie jest dopasowanych 80 pakietów, w drugiej zaś jedynie 40. Dzieje się tak dlatego,
że w pierwszej bazie zostały oznaczone wszystkie pakiety ICMP. Natomiast w drugiej, tylko połowa,
bo przecie tylko dwie z czterech sesji `ping` idzie na adresy 8.8.8.8 i 8.8.4.4 .

## Jak działa --set-xmark w target MARK

Przy oznaczaniu pakietów za pomocą target `MARK`, reguły mają postać `-j MARK
--set-xmark 0x100/0xff00` lub `-j MARK --set-xmark 0x2/0xff` (pomińmy te poprzedzające zera). By
zrozumieć co dokładnie zachodzi przy przetwarzaniu tych reguł, trzeba sobie rozpisać binarnie
wszystkie te wartości biorąc pod uwagę przepływ pakietów przez reguły.

Wszystkie pakiety, które trafiają do `iptables` są oznaczone markiem 0x0. W pierwszej bazie
markującej chcemy ustawić oznaczenie 0x100/0xff00. By tego dokonać, trzeba przeprowadzić dwie
operacje: `AND NOT` oraz `XOR` . Spójrzmy poniżej:

         MARK 0000 0000 0000 0000    bazowe oznaczenie (0x0)
         MASK 1111 1111 0000 0000    nasza maska (0xff00)
      AND NOT -------------------    w MASK trzeba pierw poodwracać bity (0->1, 1->0), po czym dać logiczny AND z MARK
       RESULT 0000 0000 0000 0000    w ten sposób stary MARK zawsze jest przepisywany (0x0)
    SET XMARK 0000 0001 0000 0000    ustawiamy nowy mark (0x100)
          XOR -------------------    przeprowadzamy operację XOR (przepisujemy różniące się bity)
       RESULT 0000 0001 0000 0000    otrzymujemy nowy mark (0x100) --> 256

W przypadku bazowego marka 0x0, ten proces nie jest tak klarowny jak być powinien. Na szczęście mamy
drugą fazę oznaczania i tu już proces powinien być już bardziej przejrzysty:


         MARK 0000 0001 0000 0000    bazowe oznaczenie (0x100)
         MASK 0000 0000 ffff ffff    nasza maska (0xff)
      AND NOT -------------------    w MASK trzeba pierw poodwracać bity (0->1, 1->0), po czym dać logiczny AND z MARK
       RESULT 0000 0001 0000 0000    w ten sposób stary MARK zawsze jest przepisywany (0x100)
    SET XMARK 0000 0000 0000 0001    ustawiamy nowy mark (0x1)
          XOR -------------------    przeprowadzamy operację XOR (przepisujemy różniące się bity)
       RESULT 0000 0001 0000 0001    otrzymujemy nowy mark (0x101) --> 257


Tak właśnie działa dodawanie (sumowanie) oznaczeń w `iptables` przy pomocy target `MARK` .
