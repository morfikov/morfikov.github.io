---
author: Morfik
categories:
- Linux
date: "2019-03-15T18:02:21Z"
published: true
status: publish
tags:
- debian
- nftables
- iptables
title: Czy linux'owy firewall powinien blokować pakiety not-syn w stanie NEW
---

Od czasu do czasu w logu systemowym mojego Debiana można zanotować szereg pakietów przychodzących,
które są zrzucane przez linux'owy firewall (`nftables`/`iptables` ). Po krótkiej analizie okazało
się, że są to pakiety protokołu TCP mające stan `NEW` (czyli są to nowe połączenia) ale
niezawierające przy tym flagi `SYN` . Mój laptop nie ma aktualnie przydzielonego zewnętrznego
routowalnego adresu IPv4/IPv6, więc nasunęło się pytanie o przyczynę takiego stanu
rzeczy -- przecie będąc za NAT, nikt spoza sieci nie powinien być w stanie nawiązać połączenia z
moją maszyną, a ewidentnie co się do jej bram dobija i to nie z adresu lokalnego. Niby mam też
odfiltrowane pakiety w stanie `INVALID` (np. te mające niepoprawny zestaw flag) ale widać te
pakiety, o których mowa, nie zaliczają się do tego stanu, więc wygląda na to, że wszystko z nimi
jest w porządku. Czy tego typu pakiety TCP w stanie `NEW` niemające ustawionej flagi `SYN`
stanowią jakieś zagrożenie dla naszego komputera? Czy powinno się je zablokować, a może przepuścić
w filtrze pakietów? A jeśli zablokować, to czy zwykły `DROP` wystarczy czy może powinno się te
pakiety potraktować przy pomocy `REJECT` ?

<!--more-->
## Dlaczego pakiet not-SYN w stanie NEW nie zalicza się do INVALID

Wydawać by się mogło, że wszystkie pakiety w protokole TCP ustanawiające nowe połączenia powinny
zawierać flagę `SYN` , która ma na celu zsynchronizować końcówki połączenia w celu wymiany danych.
Dlaczego zatem stan `INVALID` nie jest przypisany do pakietu `not-SYN` w stanie `NEW` ? Okazuje
się, że tego typu sytuacja jest notowana nawet dość często i nie jest ona niczym niezwykłym w ruchu
sieciowym. Istnieje bowiem szereg przyczyn, które mogą taki stan rzeczy zainicjować i nie wszystkie
z nich mają na celu zaatakowanie naszej maszyny.

Przykładem sytuacji, w której pakiet `non-SYN` w stanie `NEW` może się pojawić, jest choćby zmiana
adresu IP. Jeśli mamy przydzielony dynamiczny adres IP, lub też korzystamy z usług ISP, u którego
jest spora rotacja klientów (np. operatorzy GSM), to może się zdarzyć tak, że po podniesieniu
interfejsu sieciowego zostanie nam przydzielony inny adres IP w stosunku do tego, który mieliśmy
przypisany przed opuszczeniem tego interfejsu. Ten nasz nowy adres będący wcześniej w innych
łapkach mógł nawiązać jakieś połączenia ze zdalnym serwerem. W międzyczasie ten adres IP został
przydzielony nam, a serwer nie jest świadom tego całego zdarzenia. Jeśli teraz dodamy do tego
przedsięwzięcia pakiety `keep-alive` , to po jakimś czasie serwer pośle do nas pakiet `ACK` , by
się upewnić, że druga strona nie zakończyła połączenia, przez co sesja powinna być w dalszym ciągu
utrzymywana.

Może się zdarzyć też tak, że serwer, do którego się łączymy jest dość poważnie obciążony i nasz
linux zakończy połączenie zanim otrzyma odpowiedź od serwera. Niemniej jednak, taka odpowiedź do
nas może trafić później i wtedy taki pakiet zostanie zakwalifikowany jako `non-SYN` w stanie `NEW` .

Generalnie to powinniśmy zwracać uwagę na to jakie pakiety w stanie `NEW` do nas docierają, bo mogą
nam one nieco ułatwić odnalezienie przyczyny. Poniżej przykład logu z `nftables`/`iptables` :

    kernel: [ 9271.326139] * new-not-syn * IN=bond0 OUT= MACSRC=ec:08:6b:00:fb:b0
    MACDST=3c:4a:92:00:4c:5b MACPROTO=0800 SRC=54.38.143.252 DST=192.168.0.135 LEN=1440
    TOS=0x18 PREC=0x00 TTL=53 ID=17441 DF PROTO=TCP SPT=443 DPT=49412 SEQ=4233876898
    ACK=924634312 WINDOW=245 RES=0x00 ACK URGP=0 OPT (0101080A0588C9ECF077CEED)

Wyżej widać ustawioną flagę `ACK` oraz ilość potwierdzonych bajtów `ACK=924634312` . Więc
ewidentnie mamy do czynienia z pierwsza sytuacją opisaną wyżej (najprawdopodobniej przerwany
transfer pliku pobieranego z serwera), a nie z obciążonym serwerem.

Są też i inne sytuacje, w których takich pakietów `not-SYN` w stanie `NEW` możemy doświadczyć, np.
jeśli mamy do czynienia z asymetrycznym routingiem (docierają do nas pakiety `SYN-ACK` w stanie
`NEW` ). Zwykle jednak tego typu sytuacja związana jest z jakimś błędem w konfiguracji routingu w
przyległych sieciach i z tym problemem przydałoby się jak najszybciej uporać.

Jeśli z kolei przeładujemy politykę linux'owej zapory sieciowej i przy tym czyścimy tablicę
conntrack'a chcąc mieć pewność, że wszystkie nawiązane sesje zostaną ubite (klienci będą musieli
jeszcze raz nawiązać połączenie), to w takim przypadku również zanotujemy sporo pakietów `not-SYN`
w stanie NEW.

### Czy pakiety not-SYN w stanie NEW mogą być groźne

Poza tymi sytuacjami opisanymi wyżej, pakiety TCP w stanie `NEW` niezawierające flagi `SYN` mogą
być także wykorzystywane w złych celach. Przykładem mogą być skany portów z wykorzystaniem pakietu
`ACK` do [ustalenia czy dana maszyna ma zaimplementowany firewall](https://nmap.org/man/pl/man-port-scanning-techniques.html)
i by poznać po części jego konfigurację, np. czy wykorzystywane są stany połączeń i które porty są
filtrowane.

## DROP czy REJECT dla pakietów not-SYN w stanie NEW

Jako, że pakiety TCP niemające flagi `SYN` ale będące w stanie `NEW` są bardzo charakterystyczne,
to możemy je bardzo prosto zablokować na firewall'u, z tym, że przydałoby się rozważyć kwestię
stosowania `DROP` lub `REJECT` . Która z tych dwóch opcji jest lepsza? W zasadzie jeśli mamy
odpowiednio skonfigurowany firewall w linux'ie i mamy wystawione na świat tylko te usługi, które
faktycznie chcemy udostępnić, a wszystkie inne porty mamy pozamykane, to te skany portów za wiele
nowych informacji o naszej maszynie nie dostarczą. Za to zrzucając cicho pakiety `not-SYN` w stanie
`NEW` jedynie przyczyniamy się do nieinformowania serwera, że coś jest nie tak z połączeniem, które
powinno zostać już dawno zakończone. Dlatego też dobrze jest wysłać pakiet `RST` w odpowiedzi na
taką próbę połączenia i zresetować połączenie, tak by serwer je poprawnie zakończył i przestał przy
tym marnować na nie zasoby sprzętowe.

Pakiety protokołu TCP mające stan `NEW` ale bez ustawionej flagi `SYN` można w
`nftables`/`iptables` wyłowić przy pomocy poniższych reguł.

Dla `DROP` :

    # nft add rule inet filter INPUT meta l4proto tcp tcp flags & (fin|syn|rst|ack) != syn ct state new counter drop

    # iptables -A INPUT -p tcp ! --syn -m conntrack --ctstate NEW -j DROP

Dla `REJECT` :

    # nft add rule inet filter INPUT meta l4proto tcp tcp flags & (fin|syn|rst|ack) != syn ct state new counter reject with tcp reset

    # iptables -A INPUT -p tcp ! --syn -m conntrack --ctstate NEW -j REJECT --reject-with tcp-reset

Warto zauważyć, że sporo użytkowników daje te reguły przed regułą, która ma na celu przepuścić
pakiety mające ustawioną flagę `SYN` i będące przy tym w stanie `NEW` . Takie zachowanie nie ma
większego sensu -- pakiet niemający flagi `SYN` ale będący przy tym w stanie `NEW` nigdy nie
zostanie dopasowany do reguły, która sprawdza obecność flagi `SYN` . Taka kolejność reguł spowalnia
tylko przechodzenie pakietów przez zaporę. Dlatego dobrze jest dodać te reguły gdzieś pod koniec
filtra Pamiętajmy też, że jeśli ręcznie nie odfiltrujemy tych pakietów, to zostaną one albo
zrzucone gdzieś pod koniec listy reguł, albo też zadziała domyślna polityka łańcucha `INPUT` ,
którą zwykle jest `DROP` .
