---
author: Morfik
categories:
- Linux
date: "2015-12-15T14:38:30Z"
date_gmt: 2015-12-15 13:38:30 +0100
published: true
status: publish
tags:
- moduły-kernela
- iptables
- kernel
- sieć
- qos
title: Konfiguracja interfejsów IMQ w linux'ie
---

W linux'ie, kształtowanie przychodzącego ruchu sieciowego stwarza dość poważne problemy. Na dobrą
sprawę, obecnie w kernelu nie ma żadnego mechanizmu, który byłby w stanie to zadanie realizować.
Istnieją, co prawda, [interfejsy IFB][1] ale za ich pomocą jesteśmy w stanie z powodzeniem
kształtować jedynie ruch wychodzący. W przypadku pakietów napływających, możemy jedynie ograniczyć
im przepustowość. W tym powyższym linku jest wzmianka, że te interfejsy IFB są następcą
[interfejsów IMQ][2]. Niemniej jednak, ten drugi projekt zdaje się działać, choć nie jest obecnie
wspierany przez kernel linux'a. W tym wpisie postaramy się skonfigurować działające interfejsy IMQ,
tak, by za ich pomocą skutecznie kształtować ruch przychodzący.

<!--more-->
## Kompilacja kernela na potrzeby IMQ

Domyślny kernel w debianie nie ma zaimplementowanej obsługi interfejsów IMQ. Musimy zatem pokusić
się o źródła tego kernela oraz o nałożenie na nie odpowiedniej łaty. Tak przygotowany kernel będzie
trzeba skompilować. Jeśli chcemy mieć w systemie interfejsy IMQ, to bez rekompilacji kernela się
niestety nie obejdzie.

Patch'e na konkretne wersje kernela są dostępne [pod tym adresem][3]. To, którą łatę musimy zassać
zależy od posiadanego kernela. W tym przypadku jest to `4.3.0-1-amd64` , zatem potrzebny jest nam
patch `linux-4.3-imq.diff` . Musimy także skołować źródła kernela. Możemy je pobrać albo ze [strony
kernela][4] albo też posłużyć się tymi źródłami dostępnymi w debianie. Skorzystamy z tej drugiej
opcji.

By pobrać źródła kernela z repozytorium debiana, potrzebujemy dodać do pliku `/etc/apt/sources.list`
wpis z `deb-src` , przykładowo:

    deb-src http://ftp.pl.debian.org/debian/ sid main non-free contrib

Następnie aktualizujemy listę pakietów przy pomocy `apt-get update` , po czym pobieramy źródła via
`apt-get source linux` .

Te debianowe źródła kernela mają zaaplikowane własne patch'e. Może się zdarzyć tak, że nałożenie
patch'a IMQ będzie stwarzać problemy. W takiej sytuacji wyjścia są dwa. Pierwsze z nich zakłada
pobranie czystych źródeł (tych ze strony kernela). Drugie zaś wyjście na próbie ustalenia, które
uprzednio nałożonych łat powodują problemy z tym patch'em. Tak czy inaczej w przypadku debianowego
kernela 4.3, patch wchodzi prawie czysto ale najważniejsze, że bez błędów:

    # cd linux-4.3.1/
    # patch -p1 --dry-run < ../linux-4.3-imq.diff

Wyżej skorzystałem z opcji `--dry-run` po to, by nie aplikować łaty bezpośrednio na źródła. Chodziło
generalnie o przetestowanie czy zostanie ona zaaplikowana bez większych problemów. Teraz mogę ten
patch dodać do listy patch'y zarządzanych przez `quilt` i w prosty sposób [zbudować pakiet .deb][5]
wykorzystując do tego celu natywne narzędzia debiana. Oczywiście nic nie stoi na przeszkodzie, by
pozbyć się parametru `--dry-run` , nałożyć łatę, po czym ręcznie skompilować i zainstalować sobie
kernel.

W przypadku, gdy będziemy pobierać czyste źródła kernela, [zweryfikujmy sygnaturę GPG][6].

Kopiujemy starą konfigurację kernela, odpalamy `menuconfig` i dostosowujemy konfigurację włączając
odpowiednie opcje odpowiedzialne za interfejsy IMQ. Interesują nas te poniższe pozycje:

![]({{< baseurl >}}/img/2015/12/1.konfiguracja-kernel-interfejsy-imq.png#huge)

oraz:

![]({{< baseurl >}}/img/2015/12/2.konfiguracja-kernel-interfejsy-imq.png#huge)

W konfiguracji kernela (plik `.config` ) powinny znaleźć się te poniższe wpisy:

    CONFIG_NETFILTER_XT_TARGET_IMQ=m
    CONFIG_IMQ=m
    # CONFIG_IMQ_BEHAVIOR_AA is not set
    CONFIG_IMQ_BEHAVIOR_AB=y
    # CONFIG_IMQ_BEHAVIOR_BA is not set
    # CONFIG_IMQ_BEHAVIOR_BB is not set
    CONFIG_IMQ_NUM_DEVS=16

Zapisujemy konfigurację i robimy pakiet `.deb` . W przypadku, gdy nie zaprzęgamy debianowego
mechanizmu do tworzenia pakietów, to zawsze możemy taki pakiet zrobić ręcznie przy pomocy tych
poniższych poleceń:

    # aptitude build-dep linux-image-`uname -r`
    # make-kpkg -j2 --revision=morfik --initrd kernel-image kernel-headers

Trzeba mieć jednak na uwadze, że zostanie zainstalowanych sporo pakietów, które są wymagane jedynie
przy kompilacji kernela. Te pakiety można w późniejszym czasie zwyczajnie odinstalować.

## Kompilacja iptables na potrzeby IMQ

Przebudować musimy nie tylko samego kernela ale również filtr pakietów `iptables` . Standardowy
`iptables` nie zawiera interesujących nas targetów, a bez nich nie będziemy w stanie przekierować
ruchu do odpowiednich interfejsów. Podobnie jak w przypadku kernela, musimy pozyskać [źródła
iptables][7] oraz odpowiednią [łatę][8].

    # apt-get -t unstable source iptables
    # cd iptables-1.4.21/
    # patch -p1 --dry-run < ../iptables-1.4.13-IMQ-test1.diff
    checking file extensions/libxt_IMQ.c
    checking file extensions/libxt_IMQ.man
    checking file include/linux/netfilter/xt_IMQ.h

Teraz pozowało nam już jedynie zbudowanie pakietu `.deb` . Podobnie jak w przypadku kernela, jeśli
nie korzystamy z debianowego mechanizmu budowy pakietów, to taką paczkę zawsze możemy stworzyć przy
pomocy tych poniższych poleceń:

    # aptitude build-dep iptables
    # dpkg-buildpackage -uc -us -b

## Instalacja pakietów i test interfejsów IMQ

Po tym jak kompilacja i budowanie paczek dobiegnie końca, instalujemy je w systemie i restartujemy
maszynę. Sprawdzamy czy moduł został dodany:

    # modinfo imq
    filename:       /lib/modules/4.3.0-1-amd64/kernel/drivers/net/imq.ko
    ...

Ładujemy go także za pomocą `modprobe` :

    # modprobe imq numqueues=2 numdevs=2

Parametr `numqueues` odpowiada za liczbę kolejek i zależy od ilości rdzeni procesora. W tym
przypadku procesor ma dwa rdzenie. Z kolei `numdevs` odpowiada za ilość tworzonych urządzeń IMQ. Nam
są potrzebne dwa, po jednym na ruch przychodzący i wychodzący. Sprawdzamy, czy interfejsy pojawiły
się w systemie przy pomocy `ip` lub `ifconfig` :

    # ip link
    ...
    26: imq0:  mtu 16000 qdisc noop state DOWN mode DEFAULT group default qlen 11000
        link/void
    27: imq1:  mtu 16000 qdisc noop state DOWN mode DEFAULT group default qlen 11000
        link/void

Wygląda w porządku, z tym, że interfejsy nie są jeszcze podniesione. Aktywujmy je zatem, by
sprawdzić czy wszystko gra:

    # ip link set imq0 up
    # ip link set imq1 up

Jeśli nie ma żadnych błędów, to możemy zautomatyzować proces ładowania modułu przy starcie systemu.
Edytujemy plik `/etc/modules` i dodajemy w nim tę poniższą linijkę:

    imq

Konfigurację dla modułu określamy w pliku `/etc/modprobe.d/modules.conf` :

    options imq numqueues=2 numdevs=2

## Rozdzielenie ruchu na osobne interfejsy IMQ

Mając skonfigurowane interfejsy IMQ, przydałoby się zrobić z nich jakiś użytek. Potrzebujemy jednak
odpowiednich kolejek i konfiguracji dla `iptables` . Kolejki tworzymy przy pomocy narzędzia `tc` w
poniższy sposób:

    tc qdisc add dev imq0 root handle 1: htb default 400 r2q 10
        tc class add dev imq0 parent 1:0 classid 1:1 htb rate 1000mbit ceil 1000mbit quantum 60000
            tc class add dev imq0 parent 1:1 classid 1:10 htb rate 980kbit ceil 980kbit
                tc class add dev imq0 parent 1:10 classid 1:200 htb rate 480kbit ceil 980kbit prio 0
                tc class add dev imq0 parent 1:10 classid 1:300 htb rate 200kbit ceil 980kbit prio 2
                tc class add dev imq0 parent 1:10 classid 1:400 htb rate 150kbit ceil 980kbit prio 4
                tc class add dev imq0 parent 1:10 classid 1:500 htb rate 150kbit ceil 980kbit prio 5
            tc class add dev imq0 parent 1:1 classid 1:20 htb rate 999mbit ceil 999mbit quantum 60000
                tc class add dev imq0 parent 1:20 classid 1:1000 htb rate 99mbit ceil 999mbit prio 1 quantum 60000

    tc qdisc add dev imq1 root handle 2: htb default 400 r2q 100
        tc class add dev imq1 parent 2:0 classid 2:1 htb rate 1000mbit ceil 1000mbit quantum 60000
            tc class add dev imq1 parent 2:1 classid 2:10 htb rate 14500kbit ceil 14500kbit
                tc class add dev imq1 parent 2:10 classid 2:200 htb rate 4500kbit ceil 14500kbit prio 0
                tc class add dev imq1 parent 2:10 classid 2:300 htb rate 3000kbit ceil 14500kbit prio 2
                tc class add dev imq1 parent 2:10 classid 2:400 htb rate 4500kbit ceil 14500kbit prio 4
                tc class add dev imq1 parent 2:10 classid 2:500 htb rate 2500kbit ceil 14500kbit prio 5
            tc class add dev imq1 parent 2:1 classid 2:20 htb rate 985mbit ceil 985mbit quantum 60000
                tc class add dev imq1 parent 2:20 classid 2:1000 htb rate 85mbit ceil 985mbit prio 1 quantum 60000

Powyższe linijki stworzą nam 2 klasy główne oraz 5 klas podrzędnych dla każdego interfejsu. W tym
przypadku korzystamy z [algorytmu HTB][9] ale jest też kilka innych, a wszystkie można podzielić na
dwie kategorie: bezklasowe (pfifo, bfifo, pfifo_fast, red, sfq, tbf) i klasowe (CBQ, HTB, PRIO).
Informacje na temat każdego z powyższych algorytmów można znaleźć [tym linkiem][10].

Składnia HTB jest prosta i tam gdzie nie precyzujemy opcji, to są one ustawiane na wartości
domyślne. W powyższym przykładzie tworzymy qdisc na interfejsie imq0 i określamy kolejkę 400 jako
domyślą, do której będzie trafiał ruch. Jeśli nie określimy domyślnej kolejki, to ruch, który nie
trafi do żadnej z pozostałych kolejek, nie będzie podlegał kształtowaniu. Zakładamy główną kolejkę
(1:1) i ustawiamy jej limit 1000 mbit/s. Niemniej jednak, łącze jakim dysponujemy (zwłaszcza upload)
będzie miało sporo niższe parametry. Chodzi o to, że możemy posiadać sieć domową, a ona ma zwykle
większą przepustowość i nie chcemy sobie jej przyciąć do, np. 10 mbit/s. Dlatego też mamy wyraźny
podział i stąd dwie główne kolejki. Na jednej z nich dajemy faktyczną przepustowość łącza (tyle ile
daje ISP), z tym, zaleca się, by ten limit był troszeczkę mniejszy. A to z tego względu, że to nasza
maszyna musi być najciaśniejszym ogniwem w tym całym procesie przesyłu pakietów. W przeciwnym razie
nie damy rady kształtować ruchu, bo kolejki pakietów będą się tworzyć gdzieś indziej w sieci.
Następnie tworzymy sobie kilka kolejek wewnątrz wyznaczonego pasma. Konfigurujemy je określając
`rate` i `ceil` . Parametr `rate` odpowiada za gwarantowaną przepustowość jaką otrzyma kolejka. W
przypadku, gdy łącze będzie obciążone na full, kolejka 1:200 będzie miała do dyspozycji 480 kbitów i
nikt jej tego nie pozbawi. Podobnie z pozostałymi kolejkami. Z kolei parametr `ceil` ustawia górny
limit, czyli ile dana kolejka może pożyczyć łącza od innych kolejek ale tylko w przypadku, gdy te
nie wykorzystują one swojego pasma w pełni. Przykładowo, proces `A` przesyła swoje pakiety do
kolejki 1:200, gdzie jest ustawiony limit 480/980 kbitów/s upload'u. Gdy limit 480 kbitów/s zostanie
osiągnięty, kernel sprawdzi czy może jej coś pożyczyć z pozostałych kolejek. Jeśli łącze nie będzie
obciążone, to zwiększy jej dostępne zasoby do 980 kbitów/s. Gdy tylko procesy `B` i `C` zaczną
przesyłać pakiety do pozostałych kolejek, kernel natychmiast przykręci kurek procesowi `A` . W
chwili, gdy proces `B` lub `C` (ewentualnie oba) zakończy komunikację sieciową, odciążając tym samym
łącze, to kernel przeznaczy tę niewykorzystaną część pasma pod pozostałe procesy w zależności od
ustawionych priorytetów. Jeśli kolejka ma wyższy priorytet (mniejszy numer), to będzie miała dostęp
do niewykorzystanych zasobów jako pierwsza. Podobnie w przypadku download'u.

Ruch do interfejsów IMQ trzeba jeszcze jakoś przekierować. Najprościej to zrobić przy pomocy reguł
`iptables` :

    iptables -t mangle -N qos_egress
    iptables -t mangle -N qos_ingress

    iptables -t mangle -A PREROUTING -i bond0 -j IMQ --todev 1
    iptables -t mangle -A PREROUTING -j CONNMARK --restore-mark
    iptables -t mangle -A PREROUTING -m mark ! --mark 0 -j ACCEPT
    iptables -t mangle -A PREROUTING -i bond0 -j qos_ingress
    iptables -t mangle -A PREROUTING -j CONNMARK --save-mark

    iptables -t mangle -A POSTROUTING -o bond0 -j IMQ --todev 0
    iptables -t mangle -A POSTROUTING -j CONNMARK --restore-mark
    iptables -t mangle -A POSTROUTING -m mark ! --mark 0 -j ACCEPT
    iptables -t mangle -A POSTROUTING -o bond0 -j qos_egress
    iptables -t mangle -A POSTROUTING -j CONNMARK --save-mark

    iptables -t mangle -A qos_egress -s 192.168.0.0/16 -d 192.168.0.0/16 -j MARK --set-mark 10
    iptables -t mangle -A qos_egress -s 192.168.0.0/16 -d 192.168.0.0/16 -j RETURN
    iptables -t mangle -A qos_egress -m owner --gid-owner p2p -j MARK --set-mark 5
    iptables -t mangle -A qos_egress -m owner --gid-owner p2p -j RETURN
    iptables -t mangle -A qos_egress -m owner --gid-owner morfik -j MARK --set-mark 2
    iptables -t mangle -A qos_egress -m owner --gid-owner morfik -j RETURN
    iptables -t mangle -A qos_egress -m owner --gid-owner root -j MARK --set-mark 3
    iptables -t mangle -A qos_egress -m owner --gid-owner root -j RETURN
    iptables -t mangle -A qos_egress -s 192.168.10.0/24 -j MARK --set-mark 4
    iptables -t mangle -A qos_egress -s 192.168.10.0/24 -j RETURN

    iptables -t mangle -A qos_ingress -p tcp --dport 12345 -j MARK --set-mark 5
    iptables -t mangle -A qos_ingress -p tcp --dport 12345 -j RETURN

Ten patch co założyliśmy na `iptables` daje nam możliwość zastosowania celu `-j IMQ` , a ten
przyjmuje tylko argument `--todev` i numer interfejsu. Zatem rozdzieleniem i kierowaniem ruchu na
poszczególne interfejsy zajmują się te dwie linijki z `-j IMQ` . W zależności od tego czy jest to
ruch wychodzący czy przychodzący, pakiety biegną odpowiednio do interfejsów imq0 i imq1. Reguły z
`-j CONNMARK` zapiszą oznaczenia pakietów w pliku `/proc/net/ip_conntrack` lub
`/proc/net/nf_conntrack` . Oznaczanie pakietów odbywa się za sprawą reguł z `-j MARK --set-mark` .
Trzeba jednak uważać, by czasem nie nadpisać marków. Pakiet przechodząc przez pierwszą regułę z `-j
MARK` nie kończy podróży w łańcuchu. Dlatego właśnie został zrobiony osobny łańcuch do markowania
pakietów i tam skorzystaliśmy z targetu `-j RETURN` . W takim przypadku, w momencie dopasowania
reguły, pakiet zostanie oznaczony i zwrócony do łańcucha wyżej, w tym przypadku `POSTROUTING` ,
gdzie zacznie przechodzić przez kolejne reguły tego łańcucha. Bez `-j RETURN` , pakiet by przeszedł
przez wszystkie reguły w łańcuchu `qos_egress` , co w wielu sytuacjach ustawi złego marka. Mamy
dodatkowo większe opóźnienie spowodowane dłuższą trasą pakietu. Pakiety można oznaczać w dowolny
sposób. Powyżej jest tylko kilka przykładów.

Potrzebujemy jeszcze odpowiednich filtrów `tc` , by przekierować konkretne pakiety do przeznaczonych
dla nich kolejek:

    tc filter add dev imq0 protocol ip parent 1:0 prio 1 handle 10 fw classid 1:1000
    tc filter add dev imq0 protocol ip parent 1:0 prio 0 handle 2 fw classid 1:200
    tc filter add dev imq0 protocol ip parent 1:0 prio 2 handle 3 fw classid 1:300
    tc filter add dev imq0 protocol ip parent 1:0 prio 4 handle 4 fw classid 1:400
    tc filter add dev imq0 protocol ip parent 1:0 prio 5 handle 5 fw classid 1:500

    tc filter add dev imq1 protocol ip parent 2:0 prio 1 handle 10 fw classid 2:1000
    tc filter add dev imq1 protocol ip parent 2:0 prio 0 handle 2 fw classid 2:200
    tc filter add dev imq1 protocol ip parent 2:0 prio 2 handle 3 fw classid 2:300
    tc filter add dev imq1 protocol ip parent 2:0 prio 4 handle 4 fw classid 2:400
    tc filter add dev imq1 protocol ip parent 2:0 prio 5 handle 5 fw classid 2:500

Filtr przypisujemy do określonego interfejsu ( `tc filter add dev imq0 protocol ip` ) i nadajemy mu
priorytet ( `prio 1` ) . Ustawiamy też znacznik ( `handle 10 fw` ), na podstawie którego pakiet
zostanie przekierowany do odpowiedniej kolejki ( `classid 1:1000` ). Liczba w `handle` musi
odpowiadać tej określonej przez `--set-mark` w `iptables` .

Przy pomocy narzędzia `bmon` możemy zaobserwować czy pakiety trafiają do odpowiednich kolejek.
Poniżej przykład:

![]({{< baseurl >}}/img/2015/12/3.test-interfejsow-imq-bmon.png#huge)

Jak można zaobserwować, obciążenie łącza jest w granicach w 8% i 9%, odpowiednio dla upload'u i
download'u. Jest to prawie full, bo karta sieciowa w moim laptopie ma przepustowość 100 mbit/s.
Kolejka :1000 jest od sieci domowej i ten ruch leci zupełnie na osobny blok. Dalej mamy 4 kolejki,
które odnoszą się jedynie do internetu. Kolejka :400 jest domyślna, a na pozostałe jest kierowany
ruch zgodnie z tymi regułami, które sobie określiliśmy wyżej. Widzimy też, że niektóre pozycję mają
grubo ponad 100%. Dzieje się to za sprawą parametru `ceil` , który zezwala na pożyczanie przydziału
pasma od innych kolejek. Po prawej stronie w okienku mamy statystyki pingów i jak widać przy dość
mocnym obciążeniu sieci, połączenie z internetem działa stabilnie, choć pingi są nieco podwyższone.
Oczywiście w tle działa również sieć P2P (kolejka :500).

Wszystkie powyższe regułki możemy bez problemu dodać sobie do [skryptu firewall'a][11], tak, by
były aplikowane wraz ze startem systemu.


[1]: https://wiki.linuxfoundation.org/networking/ifb
[2]: https://github.com/imq/linuximq/wiki/WhatIs
[3]: https://github.com/imq/linuximq/tree/master/kernel
[4]: https://www.kernel.org/
[5]: {{< baseurl >}}/post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
[6]: https://www.kernel.org/category/signatures.html
[7]: http://www.netfilter.org/
[8]: https://github.com/imq/linuximq/tree/master/iptables
[9]: http://luxik.cdi.cz/~devik/qos/htb/
[10]: https://lukasz.bromirski.net/docs/translations/lartc-pl.html#LARTC.QDISC
[11]: {{< baseurl >}}/post/firewall-na-linuxowe-maszyny-klienckie/
