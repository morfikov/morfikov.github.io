---
author: Morfik
categories:
- Linux
date: "2015-09-23T19:31:27Z"
date_gmt: 2015-09-23 17:31:27 +0200
published: true
status: publish
tags:
- tcp
- sysctl
title: Retransmisja i duplikaty pakietów w TCP
---

[Retransmisja](https://pl.wikipedia.org/wiki/Retransmisja) pakietu w przypadku sieci opartych na
protokołach TCP/IP nie jest niczym niezwykłym. Oczywiście, pozostaje kwestia samego realizowania
tego przedsięwzięcia ale generalnie rzecz biorąc, systemy linux'owe mają szereg opcji w kernelu,
które możemy sobie dostosować konfigurując tym samym, to w jaki sposób nasz system reaguje na
zjawisko utraty pakietów podczas ich przesyłu między dwoma punktami sieciowymi.

<!--more-->
## Wyliczanie czasu oczekiwania na retransmisję (RTO)

Czas oczekiwania na retransmisję (RTO -- Retransmission TimeOut) jest oparty na zmierzonej wartości
RTT (Round-Trip Time) pomiędzy nadawcą i odbiorcą. By zapobiec zbędnym retransmisjom segmentów,
które uległy jedynie opóźnieniu i nie zostały zgubione, minimalne RTO (stała TCP\_RTO\_MIN
zdefiniowana w kenrelu) zostało ustawione na minimum 200ms. Możemy się o tym przekonać analizując
wyjście `ss -i` , gdzie nie znajdziemy wpisu z RTO\<200 .

Jeśli się przyjrzymy uważniej, dostrzeżemy, że pakiety `SYN` mają jakąś niestandardową wartość,
która sugeruje iż te pakiety nie są podpięte pod tę powyższą zasadę wyliczania czasu RTO. W sumie
nie może być przecież inaczej, bo pakiety `SYN` rozpoczynają nowe połączenie i żaden pakiet z
odpowiedzią jeszcze nie dotarł do nadawcy. W efekcie czego nie ma jak oszacować RTT i na jego
podstawie wyliczyć RTO. Dlatego też pakiety `SYN` mają przypisaną wartość RTO, która przyjmuje 1
sekundę. Z tym, że każdy retransmitowany pakiet `SYN` ma zwiększoną wartość RTO dwukrotnie:

    pakiet SYN | czas oczekiwania na retransmisję (RTO)
    -----------+-----------------------------------------
         1     |   1
         2     |   1+2=3
         3     |   1+2+4=7
         4     |   1+2+4+8=15
         5     |   1+2+4+8+16=31
         6     |   1+2+4+8+16+32=63
         7     |   1+2+4+8+16+32+64=127

Zatem jeśli RTO pakietu `SYN` wynosi 1 sekundę i ten pakiet nie dotrze do miejsca przeznaczenia, to
jego retransmisja nastąpi po jednej sekundzie. Jeśli i ten pakiet nie zostanie dostarczony, to
kolejny będzie wysłany za kolejne dwie sekundy, czyli łącznie po trzech. Z kolei następny pakiet za
dodatkowe cztery, itd.

## Nadmierne retransmisje

Nadmierne retransmisje tego samego segmentu przez stos TCP wskazują na pewne problemy z połączeniem
na jego drugim końcu lub też gdzieś po drodze. Dlatego w linux'owym kernelu mamy możliwość
określenia dwóch progów: `tcp_retries1` oraz `tcp_retries2` , które mają za zadanie zliczać ilość
dokonanych retransmisji w przypadku poszczególnych pakietów. Gdy zostanie przekroczony pierwszy próg
(domyślnie ma wartość 3), stos TCP zgłasza warstwie sieciowej, że prawdopodobnie są jakieś problemy
z takim połączeniem. W przypadku gdy ilość retransmisji tego samego pakietu przekroczy wartość
zdefiniowaną w drugim progu (domyślnie 15), połączenie zostanie zakończone.

Jeśli chcemy określić sobie te progi ręcznie, do w pliku `/etc/sysctl.conf` dodajemy te dwa poniższe
wpisy:

    net.ipv4.tcp_retries1 = 3
    net.ipv4.tcp_retries2 = 7

## Szybka retransmisja

W przypadku normalnej retransmisji, serwer musi czekać na timeout otrzymania pakietu `ACK` od
odbiorcy. Jeśli chodzi o szybką retransmisję, to system klienta reaguje w taki sposób, że na każdy
otrzymany poza kolejnością pakiet wysyła zduplikowane potwierdzenie `ACK` . Jeśli serwer otrzyma
kilka takich pakietów (zwykle 3), musi natychmiast wstrzymać transmisję następnych danych i dokonać
retransmisji zagubionego segmentu. Dopiero po otrzymaniu od klienta potwierdzenia tego pakietu,
transmisja wraca do normy.

Poniżej jest przykładowa sytuacja, gdzie nastąpiła szybka retransmisja
pakietu:

[![1.szybka-retransmisja-pakietu-wireshark]({{< baseurl >}}/img/2015/07/1.szybka-retransmisja-pakietu-wireshark-1024x222.png)]({{< baseurl >}}/img/2015/07/1.szybka-retransmisja-pakietu-wireshark.png)

Jak widzimy, pakiet 344 został dostarczony do klienta i potwierdzenie (254041) w pakiecie 345
zostało wysłane do nadawcy. Następny pakiet się zagubił i do klienta dotarł inny pakiet, który ma
zły numer sekwencyjny. W pakiecie 347, klient wysyła duplikat potwierdzenia z numerem takim jak był
wcześniej, tj. 254041, sygnalizując tym samym, że prawdopodobnie jakiś pakiet uległ zagubieniu.
Jako, że część danych jest w tranzycie, tj. została wcześniej przesłana przez serwer, to kilka
kolejnych segmentów dociera do klienta. I jak widzimy, pakiet 348 oraz 350, to kolejne segmenty z
danymi i każdy z nich ma inny numer sekwencyjny, których klient nie może jeszcze potwierdzić,
dlatego też wysyła kolejne zduplikowane pakiety `ACK` . Po tym jak trzeci duplikat dociera do
serwera, następuje szybka retransmisja zagubionego segmentu (pakiet 352). Po jego dotarciu do
klienta, ten potwierdza odbiór danych, z tym, że nie tylko tego segmentu ale również wszystkich
pozostałych, które w między czasie do niego doszły. Robi to także w pojedynczym pakiecie `ACK` .
Jego numer potwierdzenia to 259881 i jak łatwo można policzyć odejmując ten poprzedni numer
potwierdzenia (254041), daje nam to 5840 bajtów, czyli cztery zaakceptowane segmenty (1460 bajtów
każdy) -- jeden zagubiony i trzy otrzymane poza kolejnością.

Cały mechanizm szybkiej retransmisji pakietów zależy od [selektywnych potwierdzeń
SACK]({{< baseurl >}}/post/sack-czyli-selektywne-potwierdzenia-pakietow/) i jeśli nie mamy
włączonej ich obsługi, nie aktywujemy funkcji odpowiedzialnej za szybką retransmisję.

## Wczesna retransmisja

Mechanizm wczesnej retransmisji (Early Retransmit) pozwala w pewnych sytuacjach zredukować liczbę
zduplikowanych pakietów `ACK` , które są potrzebne by zapoczątkować z kolei proces szybkiej
retransmisji zagubionego segmentu. Innymi słowy, stos TCP jest w stanie odzyskać zagubiony segment
wcześniej, bo nie musi czekać aż upłynie czas oczekiwania na retransmisję. Ten mechanizm przydaje
się bardzo w przypadku gdy w grę wchodzi niewielkie okno zatorowe (Congestion Window), gdzie zwykle
nie możliwe jest wygenerowanie wymaganej liczby zduplikowanych pakietów `ACK`.

Małe okna zatorowe mogą pojawić się za sprawą wielu czynników, np. gdy połączenie zostało
ograniczone przez jakiś algorytm przeciwzatorowy, czy też BDP (Bandwidth-Delay Product) jest zbyt
mały. Mały rozmiar okna może być także wynikiem początkowej fazy wolnego startu lub rozgłaszanego
okna/bufora odbiorczego (Receive Window) w pakietach, czy też zostać ograniczony za sprawą
aplikacji, gdzie ilość danych do przesłania na drugą stronę jest niewielka, np. przy zakończeniu
połączenia.

### Tail Loss Probe (TLP)

Czas oczekiwania na retransmisję pakietu jest bardzo szkodliwy gdy w grę wchodzą krótkie połączenia,
np. z serwerem www, gdzie taki timeout często zajmuje więcej czasu niż cała lub też pozostała część
danej transakcji. Szybka czy też wczesna retransmisja pakietu nie zawsze może okazać się pomocna. W
przypadku gdy ma miejsce koniec połączenia, to nie ma więcej pakietów, które można przesłać. Jeśli
taki pakiet, który kończy przesył danych, ulegnie zagubieniu, trzeba czekać aż wybije zegar RTO (w
przypadku linux'a to min. 200ms) i tu właśnie do gry wchodzi [algorytm Tail Loss
Probe](https://tools.ietf.org/html/draft-dukkipati-tcpm-tcp-loss-probe-01). Trzeba jednak zaznaczyć,
że opisana wyżej sytuacja nie jest jedyną, w której algorytm TLP znajduje zastosowanie. Generalnie
rzecz biorąc, są to wszystkie sytuacje, gdzie nadawca nie otrzymuje potwierdzenia przez określony
przedział czasu i nie może z jakiegoś powodu przesłać następnej porcji danych, czyli gdzie ryzyko
oczekiwania na RTO jest wysokie.

TLP został zaprojektowany z myślą o nadawcy w procesie komunikacji i wykorzystywany jest jedynie w
przypadku połączeń otwartych (stan Open), gdzie nadawca otrzymuje pakiety `ACK` w odpowiedniej
kolejności i nie zanotował on jeszcze jakiejkolwiek utraty pakietu. Musi być także włączony
mechanizm SACK.

W przypadku gdy ma miejsce koniec przesyłu danych i nastąpi utrata ostatniego segmentu, TLP zaczyna
wysyłać pakiety-próbki (Loss Probe) w czasie 2xRTT (Probe Timeout). Te próbki pakietów prowokują
odbiorcę by wysłał duplikaty potwierdzeń, które z kolei zainicjują proces szybkiej retransmisji tego
zagubionego segmentu bez potrzeby czekania aż wybije zegar RTO.

Zarówno mechanizm wczesnej retransmisji jak i algorytm TLP w kernelu linux'owym są wpięte pod jedną
zmienną, którą możemy zmienić via plik `/etc/sysctl.conf` :

    net.ipv4.tcp_early_retrans = 3

W przypadku ustawienia wartości `0` lub `1` , wczesne retransmisje zostaną odpowiednio wyłączone i
włączone. Możemy także podać wartość `2` i w takim przypadku zostanie włączony mechanizm wczesnych
retransmisji ale szybkie odtwarzanie (fast recovery) oraz szybkie retransmisje zostaną opóźnione o
1/4 czasu RTT. Ma to na celu ograniczenie liczby połączeń, które fałszywie odzyskały sprawność w
sieci o niewielkim stopniu zmiany kolejności pakietów (mniej niż 3). Istnieje także możliwość
ustawienia `3` , gdzie zostanie włączona wczesna retransmisja oraz protokół TLP. Jeśli zaś ustawimy
`4` , to zostanie aktywowany jedynie protokół TLP.

## Selektywna retransmisja

W przypadku komunikacji, gdzie nadawca i odbiorca są w stanie obsłużyć selektywne potwierdzenia
protokołu TCP, istnieje możliwość zaistnienia retransmisji selektywnej (Selective Retransmission lub
Selective Repeat), czyli takiej, której celem jest zretransmitowanie jedynie tych segmentów, które
nie dotarły jeszcze do odbiorcy. By zrozumieć w pełni ten rodzaj retransmisji, trzeba się zaznajomić
z tym jak działają selektywne potwierdzenia (SACK).

## Retransmisja pakietów SYN

Pakiety `SYN` rozpoczynają nowe połączenia i mogą z tego względu być traktowane ulgowo, tj. możemy
im przypisać więcej prób niż ma to miejsce w przypadku retransmisji zwykłych pakietów zawierających
jakieś dane. Ustawiony zatem limit będzie tym samym ograniczał czas, przez który kernel będzie
próbował ustanowić nowe połączenie. By zmienić ilość retransmisji pakietu `SYN` , edytujemy plik
`/etc/sysctl.conf` i dodajemy poniższą linijkę:

    net.ipv4.tcp_syn_retries = 4

Wartość 4 oznacza zatem, że kernel, po nieudanej próbie nawiązania połączenia, spróbuje
zretransmitować pakiet `SYN` 4 razy i jeśli żadna z tych prób się nie powiedzie, po czasie 15 sekund
kernel odpuści. Popatrzmy zatem jak takie zachowanie wygląda w
praktyce:

[![1.retransmisja-syn-wireshark]({{< baseurl >}}/img/2015/07/1.retransmisja-syn-wireshark-1024x124.png)]({{< baseurl >}}/img/2015/07/1.retransmisja-syn-wireshark.png)

### Retransmisja pakietów SYN-ACK

Gdy serwer otrzymuje zapytanie `SYN`, natychmiast wysyła odpowiedź w postaci pakietu z ustawionymi
flagami `SYN` i `ACK` umieszczając jednocześnie to połączenie w kolejce (backlog queue) do chwili aż
nadejdzie od klienta pakiet z ustawioną flagą `ACK` . Gdy taki pakiet nie nadchodzi, serwer
retransmituje swój pakiet `SYN-ACK` kilka razy, dając tym samym szansę klientowi by spróbował
odesłać pakiet `ACK` jeszcze raz.

Połączenia w [stanie SYN RCVD]({{< baseurl >}}/post/flagi-tcp-i-przelaczanie-stanow-polaczen/)
zajmują cenne zasoby i by odciążyć trochę maszynę można przyśpieszyć proces zamykania tego typu
połączeń przez zmianę ilości retransmisji pakietów `SYN-ACK` . Dopisujemy zatem do pliku
`/etc/sysctl.conf` poniższy parametr:

    net.ipv4.tcp_synack_retries = 2

Trzeba jednak pamiętać, że obniżenie wartości tego parametru w przypadku słabych łącz może powodować
problemy. Poniżej fotka z próbą retransmisji dwóch pakietów `SYN-ACK`
:

[![2.retransmisja-syn-ack-wireshark]({{< baseurl >}}/img/2015/07/2.retransmisja-syn-ack-wireshark-1024x114.png)]({{< baseurl >}}/img/2015/07/2.retransmisja-syn-ack-wireshark.png)

## Zbędna retransmisja (Spurious retransmission)

Warto też wiedzieć, że istnieją także przypadki gdzie protokół TCP może zainicjować retransmisję
segmentu, który nie został utracony. Oczywistym jest, że tego typu sytuacja nie powinna mieć miejsca
ale zdarza się, np. za sprawą przedwczesnych timeout'ów (Spurious timeouts), które wystąpiły
wcześniej niż oczekiwano, tj. w momencie gdy opóźnienie przy przesyłaniu pakietu momentalnie
wzrasta dość gwałtownie i przekracza RTO, np. w sieciach bezprzewodowych.

Zbędne retransmisje mogą także wystąpić w innych sytuacjach. Jedną z nich jest zmiana kolejności
pakietów, które docierają do odbiorcy. Zwykle taka sytuacja ma miejsce gdy pakiety podróżują po
internecie różnymi trasami, z których jedna jest krótsza i mniej zatłoczona od pozostałych, co
przekłada się na szybkość z jaką docierają pakiety na drugi koniec połączenia. Zamiana pakietów
może mieć miejsce przy ich transferze w obu kierunkach, tj. pakiety wysłane przez nadawcę do
odbiorcy mogą u tego drugiego dotrzeć w innej kolejności niż zostały wysłane, oraz pakiety `ACK`
wracające do nadawcy mogą także dotrzeć w nie tej kolejności co potrzeba. Jeśli chodzi o zamianę
kolejności pakietów u odbiorcy, to taka sytuacja może zostać mylnie zinterpretowana jako utrata
pakietów. Jako, że utrata pakietów i docieranie ich w nieodpowiedniej kolejności powodują duplikaty
potwierdzeń `ACK` , to jeśli ilość takich pakietów przekroczy 3 (domyślne ustawienia linux'a), to
pakiet zostanie zretransmitowany ponownie. W przypadku gdy ilość pakietów zamienionych miejscami
będzie mniejsza, to protokół TCP bez problemu sobie z taką sytuacją poradzi bez jakiejkolwiek
retransmisji. Jeśli chodzi zaś o docieranie pakietów `ACK` w złej kolejności, to może to prowadzić
do impulsowości (burstiness), czyli do chwilowego szybkiego zwiększenia prędkości transmisji, co
może przełożyć się na problemy z wykorzystaniem dostępnej przepustowości łącza.

Innym przypadkiem, gdzie mogą nastąpić zbędne retransmisje, jest powielanie pakietów. Jest to bardzo
rzadkie zjawisko, w którym protokół IP dostarcza dany pakiet więcej niż jeden raz. Ta sytuacja może
zostać zaobserwowana gdy protokół warstwy sieciowej (model TCP/IP) nadawcy wykonuje retransmisje i
powiela dany pakiet kilka razy. To z kolei może zmylić samego nadawcę, który otrzyma kilka
duplikatów pakietów `ACK` i dokona szybkiej retransmisji segmentu, który dotarł do odbiorcy. W
przypadku wykorzystywania mechanizmu SACK (z opcją DSACK), takie duplikaty pakietów `ACK` mogą
zostać wyłapane przez protokół TCP z powodzeniem, bo zawierają informację, że dany segment został
już odebrany przez odbiorcę i szybka retransmisja w takim przypadku często nie będzie mieć miejsca.
