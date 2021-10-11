---
author: Morfik
categories:
- OpenWRT
date: "2016-07-08T18:00:54Z"
date_gmt: 2016-07-08 16:00:54 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- qos
GHissueID: 393
title: Kształtowanie ruchu z qos-scripts w OpenWRT
---

Każdy bardziej zaawansowany administrator sieci prędzej czy później będzie chciał wdrożyć na swoim
routerze wyposażonym w firmware OpenWRT pewien mechanizm QoS umożliwiający kształtowanie ruchu
sieciowego. Ci, którzy się za ten temat zabierali, wiedzą, że nie jest on prosty w realizacji.
Zwłaszcza, gdy chce się cały ten system zarządzania pakietami skonfigurować od podstaw przy pomocy
narzędzi takich jak `iptables` , `tc` oraz `ip` . Z tego też względu OpenWRT umożliwia nam nieco
prostszą w konfiguracji alternatywę polegającą na zainstalowaniu narzędzi zawartych w pakietach
`wshaper` , `qos-scripts` lub `sqm-scripts` . Trzeba przy tym pamiętać, że mechanizm, który zostanie
stworzony z wykorzystaniem jednego z tych w/w pakietów nie będzie tak elastyczny jak przy ręcznej
konfiguracji od podstaw. Niemniej jednak, w tym artykule postaramy się ogarnąć kształtowanie ruchu
przy pomocy tych pakietów.

<!--more-->
## Wshaper, qos-scripts i sqm-scripts

Przede wszystkim, jeśli ktoś jest zainteresowany ręczną [konfiguracją mechanizmu QoS w OpenWRT][1],
to odsyłam go do tego artykułu. My tutaj skupimy się na wyborze odpowiednich narzędzi tak, by
możliwie bezproblemowo zaimplementować na routerze kształtowanie ruchu.

W OpenWRT mamy z grubsza trzy pakiet, które udostępniają nam możliwość implementacji mechanizmu QoS.
Są to `wshaper` , `qos-scripts` lub `sqm-scripts` . Jak możemy wyczytać [pod tym linkiem][2],
`wshaper` nie jest już rozwijany i lepszą alternatywą jest korzystanie z `qos-scripts` lub
`sqm-scripts` . Ten ostatni ma wsparcie w LUCI (graficzna nakładka na OpenWRT). My jednak będziemy
korzystać z konsoli i potrzebny nam będzie pakiet `qos-scripts` . Logujemy się zatem na router i
instalujemy wspomniany pakiet. Razem z nim zostanie pociągniętych szereg zależności. Dlatego też
upewnijmy się, że mamy około 400 KiB wolnego miejsca na pamięci flash:

    # opkg update
    # opkg install qos-scripts

## Konfiguracja qos-scripts

Konfiguracja dla skryptów dostarczanych w pakiecie `qos-scripts` znajduje się w pliku
`/etc/config/qos` . Sam plik jest dość obszerny ale w zasadzie interesować nas będzie tylko jego
górna połowa, a konkretnie ta poniższa część:

![](/img/2016/07/1.qos-scripts-openwrt-router-ksztaltowanie-ruchu.png#big)

### Interfejsy

Blok `config interface wan` określa interfejs, na którym ruch pakietów będzie ulegał kształtowaniu.
Standardowo jest określony tylko jeden interfejs i jest to ten, przez który pakiety lecą do
internetu. Jak widzimy wyżej na fotce, w tym bloku mamy określone 4 opcje:

  - `classgroup` -- przypisuje grupy klas do danego interfejsu.
  - `enabled` -- włącza kształtowanie ruchu na tym interfejsie.
  - `upload` oraz `download` -- definiują parametry łącza (w kbit/s).

Standardowo `qos-scripts` jest wyłączony dla interfejsu `wan` i trzeba go włączyć przestawiając
opcję `enabled` na `1` . Musimy także dostosować sobie odpowiednio opcje `upload` i `download` tak,
by odpowiadały parametrom technicznym łącza naszego ISP. Można nieco niższe wartości ustawić w
przypadku, gdy kształtowanie ruchu nie realizuje swojego zadania.

Bloków `config interface` można definiować dowolną ilość. Trzeba tylko określić nazwę interfejsu,
która widoczna jest w pliku `/etc/config/network` .

### Klasy

By zrozumieć opcję `classgroup` , która została wykorzystana w bloku `config interface wan` , musimy
zajrzeć niżej do pliku `/etc/config/qos` . Mamy tam mniej więcej taki kod:

![](/img/2016/07/2.qos-scripts-openwrt-router-ksztaltowanie-ruchu.png#big)

Interesuje nas tutaj `config classgroup "Default"` . Mamy zdefiniowane dwie opcje:

  - `classes` -- zawiera nazwy wykorzystywanych klas.
  - `default` -- określa klasę domyślą z tych określonych w `classes` .

Na powyższej fotce mamy również rozpisaną konfigurację poszczególnych klas. Z grubsza rzecz ujmując,
każda klasa inaczej traktuje pakiety, które trafiają na interfejs sieciowy. W ten sposób można nadać
określonym pakietom wyższy priorytet i przyśpieszyć proces ich przetwarzania, co skutkuje redukcją
opóźnień. Standardowo mamy zdefiniowanych kilka klas, które w zasadzie powinny wystarczyć na
ogarnięcie ruchu, który przepływa przez domowy router.

Nigdzie nie znalazłem jakiejś przyzwoitej dokumentacji opisującej opcje w blokach `config class` ,
także trzeba się posiłkować nazwami klas. [Pod tym linkiem][3] znajduje się także informacja, że
klasa `Priority` lepiej się nadaje do nadawania priorytetu małym pakietom, chyba do 400 bajtów oraz,
że klasa `Express` bardziej sprawdza się przy większych pakietach (do 1000 bajtów). Trzeba zatem
rozważnie dobierać te klasy. Klas zaś domyślnie mamy cztery, z których `Normal` jest klasą domyślą,
gdzie trafia ruch niesklasyfikowany.

Podobnie jak w przypadku bloku `config interface` , również blok `config classgroup` może być
definiowany wielokrotnie. Trzeba tylko dobrać nazwę i zestaw klas.

### Reguły

Jeśli chcemy nadać wyższy priorytet określonym pakietom, to korzystamy z reguł. Popatrzymy jeszcze
raz na fotkę, na której są widoczne reguły.

![](/img/2016/07/1.qos-scripts-openwrt-router-ksztaltowanie-ruchu.png#big)

Poza blokami `config classify` mamy tam również inne bloki: `config reclassify` oraz `config
default` . Co one oznaczają i czym się one różnią od `config classify` ? Wszystkie te reguły odnoszą
się do `iptables` . Generalnie to chodzi o kolejność reguł w filtrze, co warunkuje możliwość
przepisywania oznaczeń połączeń. Zatem `config classify` markuje wstępnie wszystkie nowe połączenia.
Jeśli zajdzie potrzeba przepisania oznaczeń, np. za sprawą dopasowania pakietu po protokole i
wielkości, wtedy trzeba taki pakiet oznaczyć dwukrotnie i do tego celu potrzebny jest `config
reclassify` . Z kolei `config default` odnosi się do pakietów, które nie zostały dopasowane do
żadnych reguł.

Wszystkie reguły umieszczone w pliku `/etc/config/qos` są przetwarzane w pewnej kolejności. Przede
wszystkim ma znaczenie typ danej reguły. Najpierw są przetwarzane reguły mające `config classify` ,
później `config reclassify` i na końcu `config default` . Kolejność występowania reguł tego samego
typu również ma znaczenie.

Poniżej znajduje się rozpiska opcji, w oparciu o które pakiety mogą zostać dopasowane (można
definiować kilka jednocześnie):

  - `target` -- jedna z klas (standardowo Priority, Express, Normal i Bulk).
  - `proto` -- protokół.
  - `srchost` -- adres źródłowy lub notacja CIDR.
  - `dsthost` -- adres docelowy lub notacja CDIR.
  - `ports` -- porty oddzielone przecinkami.
  - `srcports` -- porty źródłowe.
  - `dstports` -- porty docelowe.
  - `portrange` -- zakres portów.
  - `pktsize` -- rozmiar pakietu.
  - `tcpflags` -- flagi protokołu TCP.
  - `mark` -- oznaczenie pakietu w `iptables` .
  - `connbytes` -- liczba bajtów lub pakietów.
  - `tos` -- pole "Type of Service" w nagłówku IP.
  - `dscp` -- pole "Differentiated services code point" w nagłówku IP.
  - `direction` -- kierunek przepływu pakietów `in` lub `out` .

Część dopasowań opiera się o określone moduły filtra `iptables` . Prawdopodobnie będzie wymagana
instalacja dodatkowych pakietów, by móc skorzystać z części tych powyższych opcji. Więcej informacji
na temat tych modułów można znaleźć [tutaj][4].


[1]: /post/quality-service-qos-w-openwrt/
[2]: https://wiki.openwrt.org/doc/uci/qos
[3]: https://wiki.openwrt.org/doc/uci/qos#types_and_groups
[4]: http://ipset.netfilter.org/iptables-extensions.man.html
