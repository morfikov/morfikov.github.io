---
author: Morfik
categories:
- Linux
date: "2019-02-16T11:05:40Z"
published: true
status: publish
tags:
- debian
- iptables
- nftables
title: Migracja z iptables na nftables w Debianie
---

Zgodnie z [informacją][1], która pojawiła się już ponad pół roku temu, dystrybucje linux'a powoli
zaczynają odchodzić od `iptables` . Prawdopodobnie w niedługim czasie `iptables` zostanie już
całkowicie wyparty i zastąpiony przez `nftables` , przynajmniej jeśli chodzi o desktopy. Nawet
[Debian zakomunikował][2], że następne wydanie stabilne tej dystrybucji (Buster) będzie domyślnie
wykorzystywało `nftables` . Wypadałoby zatem się przenieść na ten nowy framework i przygotować
sobie kilka podstawowych reguł firewall'a, które zabezpieczoną naszą maszynę przed nieautoryzowanym
dostępem z sieci.

<!--more-->
## Narzędzia iptables-nft i iptables-legacy

Aktualnie Debian posiada wsparcie zarówno dla `iptables` jak i `nftables` i pewnie przez długi czas
ten stan rzeczy nie ulegnie zmianie, nawet gdy przyszłe wydania stabilne tej dystrybucji będą
wykorzystywać domyślnie `nftables` . Jeśli chcemy dokonać migracji na `nftables` w Debianie, to
musimy zdać sobie sprawę z paru rzeczy, co pomoże nam odpowiednio skonfigurować ten system.

Przede wszystkim, w pakiecie `iptables` mamy narzędzia `xtables-nft-multi` oraz
`xtables-legacy-multi` .  Do tych dwóch binarek zostało stworzonych szereg dowiązań symbolicznych.
Poniżej znajduje się ich lista:

    /usr/sbin/arptables-nft -> xtables-nft-multi*
    /usr/sbin/arptables-nft-restore -> xtables-nft-multi*
    /usr/sbin/arptables-nft-save -> xtables-nft-multi*

    /usr/sbin/ebtables-nft -> xtables-nft-multi*
    /usr/sbin/ebtables-nft-restore -> xtables-nft-multi*
    /usr/sbin/ebtables-nft-save -> xtables-nft-multi*

    /usr/sbin/iptables-nft -> xtables-nft-multi*
    /usr/sbin/iptables-nft-restore -> xtables-nft-multi*
    /usr/sbin/iptables-nft-save -> xtables-nft-multi*
    /usr/sbin/iptables-restore-translate -> xtables-nft-multi*
    /usr/sbin/iptables-translate -> xtables-nft-multi*

    /usr/sbin/ip6tables-nft -> xtables-nft-multi*
    /usr/sbin/ip6tables-nft-restore -> xtables-nft-multi*
    /usr/sbin/ip6tables-nft-save -> xtables-nft-multi*
    /usr/sbin/ip6tables-restore-translate -> xtables-nft-multi*
    /usr/sbin/ip6tables-translate -> xtables-nft-multi*

    /usr/sbin/iptables-legacy -> xtables-legacy-multi*
    /usr/sbin/iptables-legacy-restore -> xtables-legacy-multi*
    /usr/sbin/iptables-legacy-save -> xtables-legacy-multi*

    /usr/sbin/ip6tables-legacy -> xtables-legacy-multi*
    /usr/sbin/ip6tables-legacy-restore -> xtables-legacy-multi*
    /usr/sbin/ip6tables-legacy-save -> xtables-legacy-multi*

Zatem mamy do wyboru dwie opcje: albo będziemy korzystać ze starego API kernela (linki z
`-legacy` ), albo z tego nowszego API (linki z `-nft` ). Aktualną konfigurację `iptables` w
Debianie można wyciągnąć przez system alternatyw:

     # update-alternatives --display iptables
    iptables - auto mode
      link best version is /usr/sbin/iptables-nft
      link currently points to /usr/sbin/iptables-nft
      link iptables is /usr/sbin/iptables
      slave iptables-restore is /usr/sbin/iptables-restore
      slave iptables-save is /usr/sbin/iptables-save
    /usr/sbin/iptables-legacy - priority 10
      slave iptables-restore: /usr/sbin/iptables-legacy-restore
      slave iptables-save: /usr/sbin/iptables-legacy-save
    /usr/sbin/iptables-nft - priority 20
      slave iptables-restore: /usr/sbin/iptables-nft-restore
      slave iptables-save: /usr/sbin/iptables-nft-save

    # update-alternatives --display ip6tables
    ip6tables - auto mode
      link best version is /usr/sbin/ip6tables-nft
      link currently points to /usr/sbin/ip6tables-nft
      link ip6tables is /usr/sbin/ip6tables
      slave ip6tables-restore is /usr/sbin/ip6tables-restore
      slave ip6tables-save is /usr/sbin/ip6tables-save
    /usr/sbin/ip6tables-legacy - priority 10
      slave ip6tables-restore: /usr/sbin/ip6tables-legacy-restore
      slave ip6tables-save: /usr/sbin/ip6tables-legacy-save
    /usr/sbin/ip6tables-nft - priority 20
      slave ip6tables-restore: /usr/sbin/ip6tables-nft-restore
      slave ip6tables-save: /usr/sbin/ip6tables-nft-save

    # update-alternatives --display arptables
    arptables - auto mode
      link best version is /usr/sbin/arptables-nft
      link currently points to /usr/sbin/arptables-nft
      link arptables is /usr/sbin/arptables
      slave arptables-restore is /usr/sbin/arptables-restore
      slave arptables-save is /usr/sbin/arptables-save
    /usr/sbin/arptables-nft - priority 20
      slave arptables-restore: /usr/sbin/arptables-nft-restore
      slave arptables-save: /usr/sbin/arptables-nft-save

    # update-alternatives --display ebtables
    ebtables - auto mode
      link best version is /usr/sbin/ebtables-nft
      link currently points to /usr/sbin/ebtables-nft
      link ebtables is /usr/sbin/ebtables
      slave ebtables-restore is /usr/sbin/ebtables-restore
      slave ebtables-save is /usr/sbin/ebtables-save
    /usr/sbin/ebtables-nft - priority 20
      slave ebtables-restore: /usr/sbin/ebtables-nft-restore
      slave ebtables-save: /usr/sbin/ebtables-nft-save

Jeśli podążymy za przykładowym linkiem:

    #  ls -al /usr/sbin/iptables
    lrwxrwxrwx 1 root root 26 2018-11-22 19:36:34 /usr/sbin/iptables -> /etc/alternatives/iptables*

    # ls -al /etc/alternatives/iptables
    lrwxrwxrwx 1 root root 22 2019-02-16 11:36:05 /etc/alternatives/iptables -> /usr/sbin/iptables-nft*

    # ls -al /usr/sbin/iptables-nft
    lrwxrwxrwx 1 root root 17 2018-12-28 12:57:19 /usr/sbin/iptables-nft -> xtables-nft-multi*

Zatem Debian jest domyślnie skonfigurowany w taki sposób, by wykorzystywać nowe API kernela (to
samo, z którego korzysta `nftables` ) ale robi to za sprawą modułu `nft_compat` . Ten moduł daje
możliwość wykorzystywania dokładnie tej samej składni, z którą można się zetknąć w natywnych
narzędziach, tj.  `iptables` , `ip6tables` , `arptables` i `ebtables` , przez co z naszej
perspektywy nie zmienia się praktycznie nic jeśli chodzi o operowanie na filtrze pakietów, bo nie
musimy dokonywać żadnych zmian w składni reguł i przerabiać przy tym skryptów firewall'a.

Niestety pewne narzędzia mogą mieć problemy z tą nową konfiguracją i jeśli ich doświadczamy, to
trzeba przekonfigurować Debiana tak, by korzystał z tego starszego API kernela. Możemy to zrobić za
sprawą systemu alternatyw ale tylko dla `iptables` i `ip6tables` , bo dla `ebtables` i `arpatbles`
jest tylko jedna alternatywa:

    # update-alternatives --config iptables
    There are 2 choices for the alternative iptables (providing /usr/sbin/iptables).

      Selection    Path                       Priority   Status
    ------------------------------------------------------------
    * 0            /usr/sbin/iptables-nft      20        auto mode
      1            /usr/sbin/iptables-legacy   10        manual mode
      2            /usr/sbin/iptables-nft      20        manual mode

    Press <enter> to keep the current choice[*], or type selection number: 1
    update-alternatives: using /usr/sbin/iptables-legacy to provide /usr/sbin/iptables (iptables) in manual mode

    # update-alternatives --config ip6tables
    There are 2 choices for the alternative ip6tables (providing /usr/sbin/ip6tables).

      Selection    Path                        Priority   Status
    ------------------------------------------------------------
    * 0            /usr/sbin/ip6tables-nft      20        auto mode
      1            /usr/sbin/ip6tables-legacy   10        manual mode
      2            /usr/sbin/ip6tables-nft      20        manual mode

    Press <enter> to keep the current choice[*], or type selection number: 1
    update-alternatives: using /usr/sbin/ip6tables-legacy to provide /usr/sbin/ip6tables (ip6tables) in manual mode

Gdy wykorzystywane jest nowe API kernela, to za każdym razem, gdy będziemy operować na
`iptables`/`ip6tables` na końcu wyjścia poleceń będziemy mieli to poniższe ostrzeżenie:

    # iptables -nvL
    ...
    # Warning: iptables-legacy tables present, use iptables-legacy to see them

Jeśli `iptables-legacy -S` zwróci nam jakieś reguły za wyjątkiem tych poniższych:

    -P INPUT ACCEPT
    -P FORWARD ACCEPT
    -P OUTPUT ACCEPT

To znaczy, że nie powinniśmy korzystać z tego nowego API kernela i trzeba skonfigurować system
alternatyw tak, by nasz linux używał `iptables-legacy` . Ewentualnie trzeba poprawić sam
program/skrypt, by wykorzystywał ten nowy interfejs kernela. Chodzi generalnie o to, że używanie
dwóch różnych API kernela realizujących to samo zadanie w tym samym czasie może prowadzić do
nieprzewidzianych w skutkach efektów i jest wysoce odradzane.

## Narzędzie nft

Jeśli jednak chcemy odpuścić sobie korzystanie z `iptables` czy też tego trybu kompatybilności, to
najlepszym wyjściem jest po prostu zainstalowanie w systemie pakietu `nftables` . Znajduje się w
nim narzędzie `nft` , które będziemy wykorzystywać tam, gdzie byśmy normalnie używali tych
przestarzałych narzędzi z pakietu `iptables` .

## Migracja reguł iptables na nftables

Ogarnięcie nowej składni reguł `nftables` , może nam zająć trochę czasu. By ułatwić to zadanie i
jednocześnie złagodzić nieco proces przejścia z `iptables` na `nftables` , deweloperzy `nftables`
zaprojektowali narzędzia, które mają na celu przetłumaczyć składnię reguł, co w sporej części
przypadków jest nam w stanie zaoszczędzić sporo czasu podczas migracji. Oczywiście nic nie stoi na
przeszkodzie by podpierając się [wiki nftables][3], w szczególnościtą [stroną][4], zbudować sobie
filtr pakietów od podstaw.

Reguły iptables [można tłumaczyć][5] jedna po drugiej lub też możemy przetłumaczyć je wszystkie
naraz. Warto tutaj zaznaczyć, że te narzędzia do tłumaczenia reguł nie są w 100% akuratne i pewnych
reguł nie będą w stanie przetłumaczyć (zwykle tych, które nie są wspierane w `nftables` , np.
reguły od `ipset` ). Cześć reguł nawet może zostać przetłumaczona błędnie poprzez zgubienie pewnych
ich elementów, np. port docelowy, czy wychodzący interfejs sieciowy, itd. Niemniej jednak,
większość standardowych reguł, które spotyka się zwykle na desktopach, powinna zostać
przetłumaczona poprawnie bez większego problemu. Dobrze jest jednak po procesie translacji
zweryfikować każdą pojedynczą regułę i porównać ją z tym co poddaliśmy tłumaczeniu, by wyeliminować
jakiekolwiek nieścisłości, które mogłyby zaistnieć.

Na sam początek musimy wyeksportować reguły, które mamy aktualnie załadowane w filtrze. Do tego
celu służą narzędzia `iptables-save` oraz `ip6tables-save` odpowiednio dla reguł protokołu IPv4 i
IPv6:

    # iptables-save > inet4.txt
    # ip6tables-save > inet6.txt

Po wydaniu tych dwóch poleceń zostaną stworzone dwa pliki `inet4.txt` oraz `inet6.txt` zawierające
reguły `iptables` , które następnie musimy wrzucić do translatora:

    # iptables-restore-translate -f inet4.txt > ruleset4.nft
    # ip6tables-restore-translate -f inet6.txt > ruleset6.nft

W ten sposób uzyskamy kolejne dwa pliki `ruleset4.nft` oraz `ruleset6.nft` , które będą zawierać
odpowiedniki reguł `iptables` . I tu w zasadzie proces tłumaczenia reguł się kończy.

Niemniej jednak, `nftables` posiada szereg cech, które sprawiają, że reguły filtra można dość mocno
skompresować. Chodzi generalnie o fakt, że jesteśmy w stanie stworzyć jedną tablicę dla obu
protokołów IPv4 i IPv6, przez co nie musimy dublować całej masy reguł. Naturalnie cześć reguł z
racji różnicy w wersjach tych protokołów trzeba będzie napisać osobno dla IPv4 i IPv6, np. w
momencie gdy będziemy chcieli określić adres źródłowy/docelowy, bo te mają przecie inny format. Za
to, gdy będziemy określać, np. jedynie port źródłowy/docelowy dla jakiejś usługi, to wystarczy nam
jedna reguła i ona ogarnie nam oba protokoły. Nawet jeśli tych reguł specyficznych dla konkretnego
protokołu IP będzie sporo, to zawsze będą one w jednym miejscu. Niektórzy mogą pomyśleć, że
przetwarzanie reguł IPv4, gdy w grę wchodzi ruch IPv6 (i odwrotnie) może dodać nieco opóźnień zanim
pakiet zostanie wysłany lub odebrany. Oczywiście jakieś opóźnienia się pojawią ale one dopiero będą
zauważane, gdy liczba reguł w filtrze będzie szła w dziesiątki tysięcy. Mój filtr jest dość
zaawansowany ale i tak nie udało mi się wyjść poza 100 reguł. No i warto pamiętać jeszcze o tym, że
w zasadzie to tylko pierwszy pakiet połączenia będzie przechodził przez te dodatkowe reguły, bo
zwykle jedną z pierwszych reguł w filtrze jest ta akceptująca połączenia mające stan ESTABLISHED i
RELATED, co znacznie odciąża FW.

## Optymalizacja filtra nftables

Po przetłumaczeniu reguł `iptables` , możemy zabrać się za optymalizację reguł `nftables` , przy
czym warto pamiętać o kilku kwestiach przystępując do tego zadania.

Przede wszystkim, startujemy z czystym filtrem. Oznacza to, że w odróżnieniu od `iptables` , gdzie
mieliśmy predefiniowane tablice ( `filter` czy `nat` ) oraz łańcuchy ( `INPUT` czy `PREROUTING`), w
`nftables` nie mamy dosłownie nic. Tablice czy łańcuchy musimy zatem sobie stworzyć sami. Po
przetłumaczeniu reguł, tablice i łańcuchy w plikach z regułami `nftables` , tj. `ruleset4.nft` i
`ruleset6.nft` , mają takie same nazwy, jakie miały w przypadku `iptables` . Możemy je zachować lub
też dowolnie przepisać.

Kolejna kwestia dotyczy sposobu komunikacji naszego hosta z siecią. Trzeba sobie zadać pytanie czy
wszystkie tablice i łańcuchy obecne w `iptables` są nam również niezbędne w `nftables` . Jeśli nasz
host jest jedynie klientem i nie świadczy żadnych usług sieciowych, czy też nie realizuje NAT i nie
ma włączonego forwarding'u (krótko mówiąc nie udostępnia połączenia sieciowego), to ogromną część
struktury filtra, która jest obecna w `iptables` , możemy pominąć. W zasadzie to dla stacji
klienckiej wystarczą nam te poniższe reguły:

    #!/usr/sbin/nft -f

    flush ruleset

    create table inet filter

    create chain inet filter INPUT { type filter hook input priority 0; policy drop; }
    create chain inet filter OUTPUT { type filter hook output priority 0; policy accept; }

    create chain inet filter state-check

    add rule inet filter state-check ct state { established, related } counter accept
    add rule inet filter state-check ct state invalid counter drop

    add rule inet filter INPUT counter jump state-check
    add rule inet filter INPUT iifname "lo" counter accept
    add rule inet filter INPUT counter drop

    add rule inet filter OUTPUT counter jump state-check

Nie jest to zbyt rozbudowany firewall ale idealnie się nadaje na podstawę filtra, którą można
rozbudować dodając do niej różne elementy.

Jeśli się uważnie przyjrzymy, to mamy w tym filtrze tylko jedną tablicę ( `filter` ) odnoszącą się
zarówno do protokołu IPv4 jak i IPv6 ( `inet` ). W tej tablicy mamy tylko dwa łańcuchy podstawowe
( `INPUT` oraz `OUTPUT` ) oraz jeden łańcuch zwykły ( `state-check` ). Te podstawowe łańcuchy mogą
mieć szereg właściwości, które się definiuje między `{ }` (o tym później).

Zarówno w łańcuchu `INPUT` jak i `OUTPUT` jest widoczne przekierowanie do łańcucha `state-check` ,
w którym znajdują się dwie reguły. Jedna z nich akceptuje pakiety połączeń już nawiązanych, druga
zaś zrzuca błędne pakiety. W łańcuchu `INPUT` nie może także zabraknąć reguły akceptującej ruch
pętli zwrotnej (interfejs `lo` ). Domyślną polityką łańcucha `INPUT` jest zrzucanie wszystkich
pakietów, które nie zostaną wcześniej dopasowane. Natomiast w łańcuchu `OUTPUT` akceptujemy
wszystkie nowe połączenia wychodzące.

Kolejna rzecz, to przetwarzanie reguł od lewej strony do prawej. W pewnych sytuacjach, zwłaszcza,
gdy wykorzystywany jest mechanizm limitowania połączeń, reguły, które mogłyby dla nas oznaczać
dokładnie to samo, dla `nftables` będą mieć różne znaczenie. Dla przykładu rozpatrzmy sobie te dwie
poniższe reguły:

    add rule inet filter INPUT limit rate 1/minute burst 1 packets log prefix "* INPUT * " counter accept
    add rule inet filter INPUT log prefix "* INPUT * " limit rate 1/minute burst 1 packets counter accept

Mają one na celu ograniczyć ilość komunikatów, które zostaną zalogowane w logu systemowym. Ta
pierwsza reguła zaloguje pakiety w tempie 1 na minutę. Natomiast ta druga zaloguje komunikaty
ilekroć pakiet będzie przez tę regułę przetwarzany. Niby te dwie reguły są podobne i dla człowieka
oznaczają to samo, to jednak, gdy się zacznie je czytać od lewej strony, a nie jako całość, to
wtedy widać, że pierwsza reguła najpierw próbuje dopasować limit i jeśli pakiet zostanie dopasowany
w tym miejscu, to jest sprawdzany dalej w tej samej regule, a tu już pozostaje tylko zalogować
komunikat i zaakceptować pakiet. Natomiast w przypadku drugiej reguły, najpierw jest logowany
komunikat, bo każdy pakiet trafiający do tej reguły zostanie pierw zalogowany ale w dalszej części
reguły mamy limit i tylko 1 pakiet na minutę zostanie zaakceptowany, a reszta będzie przetwarza
przez następną regułę w filtrze. Dlatego też warto wziąć pod uwagę ten czynnik podczas tworzenia
reguł, bo może on nam znacznie zoptymalizować przechodzenie reguł przez filtr. Warto też przeczytać
sobie artykuł na wiki poświęcony [różnicom między iptables i nftables][6].

## NAT i forwarding pakietów w nftables

Jeśli potrzebujemy realizować udostępnianie połączenia sieciowego, bez względu na to, czy w grę
wchodzą inne fizyczne komputery, czy też mamy w systemie maszyny wirtualne albo korzystamy z
jakiegoś mechanizmu konteneryzacji (Docker, LXC), to by pakiety sieciowe były w stanie przejść
przez naszego hosta musimy stworzyć tablicę `nat` w niej szereg łańcuchów oraz dodać łańcuch
`FORWARD` w utworzonej wcześniej tablicy `filter` . Poniżej znajdują się odpowiednie regułki:

    create chain inet filter FORWARD { type filter hook forward priority 0; policy drop; }

    create table ip nat
    create table ip6 nat

    create chain ip  nat PREROUTING { type nat hook prerouting priority -100; policy accept; }
    create chain ip  nat INPUT { type nat hook input priority 100; policy accept; }
    create chain ip  nat POSTROUTING { type nat hook postrouting priority 100; policy accept; }
    create chain ip  nat OUTPUT { type nat hook output priority -100; policy accept; }
    create chain ip6 nat PREROUTING { type nat hook prerouting priority -100; policy accept; }
    create chain ip6 nat INPUT { type nat hook input priority 100; policy accept; }
    create chain ip6 nat POSTROUTING { type nat hook postrouting priority 100; policy accept; }
    create chain ip6 nat OUTPUT { type nat hook output priority -100; policy accept; }

Niektóre tablice, np. widoczna wyżej `nat` , nie mogą zostać stworzone w formie `inet` , czyli by
ogarniać połączenia zarówno protokołu IPv4 i IPv6. Dlatego też w ich przypadku trzeba stworzyć
osobne tablice dla każdego z tych protokołów.
([Wsparcie dla nat w inet zostało dodane w v0.9.1][7]).

Widoczna wyżej struktura tablicy `nat` odpowiada tej, która była standardowo obecna w `iptables` .
Mając odpowiednie tablice i łańcuchy możemy dodawać w nich
[regułki realizuje NAT czy inny port forwarding][8].

## Typy, zaczepy i priorytety łańcuchów nftables

Każdy łańcuch podstawowy w `nftables` jest podczepiony do stosu sieciowego w określone miejsce i ma
również przypisany jakiś priorytet. Dla przykładu weźmy sobie łańcuch `INPUT` tablicy `filter` .
Chodzi o tę poniższą regułę:

    create chain inet filter INPUT { type filter hook input priority 0; policy drop; }

Widzimy tutaj `type filter` , który określa typ tego łańcucha. Do wyboru
[są w zasadzie trzy typy][9]:

- `filter` -- ten typ jest wykorzystywany do filtrowania pakietów.
- `route` -- ten typ jest wykorzystywany do przeroutowania pakietów, gdy zmianie ulega stosowne
pole w nagłówku IP lub też, gdy oznaczenie pakietu ulega modyfikacji. Ten typ jest odpowiednikiem
tablicy `mangle` w `iptables` ale tylko dla zaczepu `hook output` .
- `nat` -- ten typ jest wykorzystywany w mechanizmie translacji adresów NAT (pierwszy pakiet
połączenia zawsze trafia w łańcuch tego typu).

Łańcuchów tego samego typu może być kilka, a to w które miejsce zostaną one podczepione do stosu
sieciowego zależy od zdefiniowanego zaczepu (w tym przypadku mamy do czynienia z `hook input` ).
Zaczepów też [jest kilka][10]:

- `prerouting` -- przed routingiem.
- `input` -- pakiety przeznaczone dla lokalnej maszyny.
- `forward` -- po routingu ale pakiety nie są przeznaczone dla lokalnej maszyny.
- `output` -- pakiety wychodzące z lokalnej maszyny.
- `postrouting` -- po routingu
- `ingress` -- daleko przed routingiem, zaraz po odebraniu pakietów z NIC (karty sieciowej). Ten
zaczep może być alternatywą dla narzędzia `tc` .

Możemy podczepić wiele łańcuchów w to samo miejsce stosu ale by określić, przez który z nich pakiet
przejdzie wcześniej lub później, to trzeba podać priorytet (tutaj `priority 0` ). Priorytety mogą
mieć wartość ujemną lub dodatnią i możemy je dowolnie definiować. Poniżej jest odpowiednik
priorytetów wykorzystywanych przez `iptables` :

- NF_IP_PRI_CONNTRACK_DEFRAG ( `-400` ): priorytet defragmentacji.
- NF_IP_PRI_RAW ( `-300` ): priorytet tablicy `raw` , która jest umieszczona przed mechanizmem
śledzenia połączeń.
- NF_IP_PRI_SELINUX_FIRST ( `-225` ): priorytet mechanizmu SELinux.
- NF_IP_PRI_CONNTRACK ( `-200` ): priorytet mechanizmu śledzenia połączeń.
- NF_IP_PRI_MANGLE ( `-150` ): priorytet tablicy `mangle` .
- NF_IP_PRI_NAT_DST ( `-100` ): priorytet destination NAT .
- NF_IP_PRI_FILTER ( `0` ): priorytet tablicy `filter` .
- NF_IP_PRI_SECURITY ( `50` ): priorytet tablicy `security` , gdzie może być nałożony przykładowo
`secmark` .
- NF_IP_PRI_NAT_SRC ( `100` ): priorytet source NAT .
- NF_IP_PRI_SELINUX_LAST ( `225` ): priorytet mechanizmu SELinux przy wyjściu pakietów.
- NF_IP_PRI_CONNTRACK_HELPER ( `300` ): priorytet mechanizmu śledzenia połączeń przy wyjściu
pakietów.

Te wyżej podane wartości priorytetów są jedynie referencyjne, a to jakie priorytety nadamy w
`nftables` , to już zależy od nas. Podobnie sprawa wygląda w przypadku tworzenia kolejnych
tablic -- możemy ich stworzyć wiele, przez co nie musimy pakować wszystkich reguł do jednej
tablicy, a przepływem pakietów możemy sterować odpowiednio dobierając priorytety. To rozwiązanie
jest w stanie ułatwić nieco budowę samego filtra i analizę jego działania.

Warto tutaj zaznaczyć, że jeśli pakiet zostanie zrzucony na pewnym etapie przechodząc przez kolejne
tablice i łańcuchy, to już jego przetwarzanie się kończy.

Poniżej są jeszcze reguły, które są w stanie odtworzyć nam strukturę `iptables` :

    create table inet raw
    create table inet filter
    create table ip mangle
    create table ip nat
    create table ip6 mangle
    create table ip6 nat

    create chain inet raw PREROUTING { type filter hook prerouting priority -300; policy accept; }
    create chain inet raw OUTPUT { type filter hook output priority -300; policy accept; }

    create chain inet filter INPUT { type filter hook input priority 0; policy drop; }
    create chain inet filter FORWARD { type filter hook forward priority 0; policy drop; }
    create chain inet filter OUTPUT { type filter hook output priority 0; policy accept; }

    create chain ip mangle PREROUTING { type filter hook prerouting priority -150; policy accept; }
    create chain ip mangle INPUT { type filter hook input priority -150; policy accept; }
    create chain ip mangle FORWARD { type filter hook forward priority -150; policy accept; }
    create chain ip mangle OUTPUT { type route hook output priority -150; policy accept; }
    create chain ip mangle POSTROUTING { type filter hook postrouting priority -150; policy accept; }
    create chain ip nat PREROUTING { type nat hook prerouting priority -100; policy accept; }
    create chain ip nat INPUT { type nat hook input priority 100; policy accept; }
    create chain ip nat POSTROUTING { type nat hook postrouting priority 100; policy accept; }
    create chain ip nat OUTPUT { type nat hook output priority -100; policy accept; }

    create chain ip6 mangle PREROUTING { type filter hook prerouting priority -150; policy accept; }
    create chain ip6 mangle INPUT { type filter hook input priority -150; policy accept; }
    create chain ip6 mangle FORWARD { type filter hook forward priority -150; policy accept; }
    create chain ip6 mangle OUTPUT { type route hook output priority -150; policy accept; }
    create chain ip6 mangle POSTROUTING { type filter hook postrouting priority -150; policy accept; }
    create chain ip6 nat PREROUTING { type nat hook prerouting priority -100; policy accept; }
    create chain ip6 nat INPUT { type nat hook input priority 100; policy accept; }
    create chain ip6 nat POSTROUTING { type nat hook postrouting priority 100; policy accept; }
    create chain ip6 nat OUTPUT { type nat hook output priority -100; policy accept; }

## Pliki reguł nftables

W tych bardziej zaawansowanych konfiguracjach `iptables` użytkownicy zwykle tworzyli sobie własne
skrypty z regułami zamiast używać narzędzi `iptables-save` i `iptables-restore` . Posiadanie
własnego skryptu ma całą masę zalet ale wywoływanie wiele razy `iptables -A INPUT...` (i jemu
podobnych) spowalnia nieco ładowanie się filtra, no i oczywiście marnują się cykle procesora, co
wydłuża cały proces odpalania się zapory sieciowej. Naturalnie jeśli mamy tylko kilka reguł, to
raczej nie zauważymy żadnej różnicy ale w przypadku `nftables` możemy sobie stworzyć własny skrypt
filtra i podać taki plik narzędziu `nft` .

Struktura [pliku .nft][11] jest prosta. W zasadzie każde polecenie, które można podać w `nft` ,
można zapisać w pliku usuwając z niego wywołanie `nft` . Poniżej znajduje się przykładowa reguła,
którą można dodać do filtra przy pomocy narzędzia `nft` wpisując ją w terminal:

    # nft add rule inet filter INPUT iifname "lo" counter accept

By teraz tę regułę umieścić w pliku `.nft` robimy to w następujący sposób:

    #!/usr/sbin/nft -f
    ...
    add rule inet filter INPUT iifname "lo" counter accept

Nie musimy też wszystkich reguł pakować do jednego pliku, co przy bardziej zaawansowanych
konfiguracjach linux'owej zapory sieciowej może wpływać na czytelność takiego pliku. Dla przykładu,
mój podstawowy plik z konfiguracją filtra prezentuje się w poniższy sposób:

    #!/usr/sbin/nft -f

    flush ruleset

    include "./raw.nft"
    include "./mangle.nft"
    include "./nat.nft"
    include "./filter.nft"

    include "./sets.nft"

Przy pomocy dyrektywy `include` jesteśmy w stanie dołączyć w dane miejsce pliku kolejny plik z
regułami. Możemy sobie zatem pogrupować pewne rzeczy, tak jak to widać wyżej. Z kolei `flush
ruleset` wyczyści poprzednie reguły.

Problematyczne są jednak ścieżki do plików. Nie można stosować absolutnych ścieżek (tych
zaczynających się od `/` ). Natomiast jeśli wpisałoby się jedynie nazwę pliku bez widocznego wyżej
`./` , to wtedy plik jest szukany w miejscu, które zostało ustawione podczas kompilacji `nftables` .
Jedyną rozsądną opcją jest skorzystanie z `./` , za sprawą którego pliki muszą być umieszczone w
tym samym katalogu (lub niżej) w stosunku do głównego pliku z regułami.

I to w zasadzie tyle jeśli chodzi o rozpoczęcie przygody z `nftables` . Teraz została nam już
jedynie rozbudowa filtra przez tworzenie nowych reguł, tak by odpowiednio sobie skonfigurować ten
linux'owy firewall. Niemniej jednak, warto też pamiętać, że nie wszystkie rzeczy jeszcze zostały
(lub w ogóle zostaną) przeportowane z `iptables` . Pod tym linkiem znajduje
się [lista wspieranych ficzerów nftables][12] w odniesieniu do tych, które można spotkać w
`iptables` . Część ma natywne odpowiedniki, np. `ipset` , a implementacja części z pozostałych
jest [pozbawiona sensu w obecnych czasach, np. SYNPROXY][13], bo kernel linux'a dorobił się nowych
mechanizmów ochronnych. Warto tutaj dodać, że
[wsparcie dla SYNPROXY w nftables pojawiło się v0.9.2][14].



[1]: http://ral-arturo.org/2018/06/16/nfws2018.html
[2]: https://wiki.debian.org/nftables#Current_status
[3]: https://wiki.nftables.org/wiki-nftables/index.php/Main_Page
[4]: https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes
[5]: https://wiki.nftables.org/wiki-nftables/index.php/Moving_from_iptables_to_nftables
[6]: https://wiki.nftables.org/wiki-nftables/index.php/Main_differences_with_iptables
[7]: https://git.netfilter.org/nftables/commit/?id=fbe27464dee4588d90649274925145421c84b449
[8]: https://wiki.nftables.org/wiki-nftables/index.php/Performing_Network_Address_Translation_(NAT
[9]: https://wiki.nftables.org/wiki-nftables/index.php/Configuring_chains#Base_chain_types
[10]: https://wiki.nftables.org/wiki-nftables/index.php/Configuring_chains#Base_chain_hooks
[11]: https://wiki.nftables.org/wiki-nftables/index.php/Scripting
[12]: https://wiki.nftables.org/wiki-nftables/index.php/Supported_features_compared_to_xtables
[13]: https://lwn.net/Articles/659199/
[14]: https://git.netfilter.org/nftables/commit/?id=1188a69604c3df2a63daca9e735fdb535e8f6b63
