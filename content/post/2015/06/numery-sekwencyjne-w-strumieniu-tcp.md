---
author: Morfik
categories:
- Linux
date: "2015-06-25T21:24:41Z"
date_gmt: 2015-06-25 19:24:41 +0200
published: true
status: publish
tags:
- tcp
- sysctl
title: Numery sekwencyjne w strumieniu TCP
---

Jeśli zastanawialiście się czym są numery sekwencyjne i potwierdzeń w strumieniach protokołu TCP, to
nie jesteście jedyni, którym to zagadnienie spędza sen z powiek. Dlatego też poniżej postanowiłem
opisać najdokładniej jak umiem proces jaki zachodzi przy przesyłaniu danych z jednego punktu
sieciowego na drugi. Bez zaprzęgnięcia sniffera sieciowego raczej nie da się zrozumieć tego tematu i
poniższy przykład zawiera szereg odwołań do programu wireshark.

<!--more-->
## Czym są numery sekwencyjne

Każda z maszyn na krańcach sesji TCP utrzymuje 32 bitowy numer sekwencyjny (sequence number), który
wykorzystuje do śledzenia ilości danych, które przesłała. Ten numer sekwencyjny jest dołączany do
każdego przesłanego pakietu i potwierdzany przez drugą stronę numerem potwierdzenia
(acknowledgement number) w celu poinformowania nadającego hosta, że te dane zostały odebrane z
powodzeniem.

Gdy jakiś host inicjuje sesję TCP, jego początkowy numer sekwencyjny jest dobierany losowo z
przedziału od 0 do 4,294,967,295 włącznie. Niemniej jednak, sniffery takie jak wireshark zwykle
pokazują względne numery sekwencyjne i potwierdzenia (relative sequence and acknowledgement numbers)
zamiast prawdziwych wartości. Te numery są względne w stosunku do początkowego numeru sekwencyjnego
danego strumienia, który zawsze wynosi 0. To takie ułatwienie, które pozwala śledzić małe i do tego
przewidywalne numerki zamiast ciągle analizować te liczby z w/w przedziału.

## Względne numery sekwencyjne

Odpalmy zatem jakiś sniffer sieciowy i zobaczmy na własne oczy jak wyglądają te numerki i gdzie
można je znaleźć. Na poniższej fotce jest zaprezentowany względny numer sekwencyjny 0 w pakiecie
rozpoczynającym nowe
połączenie:

[![1.wireshark-flagi-tcp]({{< baseurl >}}/img/2015/06/1.wireshark-flagi-tcp-1024x544.png)]({{< baseurl >}}/img/2015/06/1.wireshark-flagi-tcp.png)

Prawdziwa wartość tego numeru jest zapisana w postaci hexalnej:

![]({{< baseurl >}}/img/2015/06/2.wireshark-numery-sekwencyjne-hex.png)

Jak widzimy jest to `b0241fe1` , czyli po ludzku 2,955,157,473 i raczej trudno jest operować na
takich wartościach.

## Jak działają numery sekwencyjne

Dla ułatwienia zrozumienia zasady działania przepisywania numerów, dobrze jest wyłączyć w wiresharku
opcję relatywnych numerów sekwencyjnych (Edit =\> Preferences =\> Protocols =\> TCP). Będziemy
operować, co prawda, na większych liczbach ale będzie za to zauważalna różnica przy ich liczeniu.
Poniżej jest przykład tego co się dzieje w sytuacji gdy odwiedzamy stronę www i pobieramy z niej
zawartość. W tym przypadku jest to proces pobierania jakiegoś obrazka. Rzućmy zatem okiem na
pierwsze pakiety w oknie
wiresharka:

[![4.wireshark-numery-sekwencyjne-1]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-1-1024x57.png)]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-1.png)

Pakiet `126` ma ustawioną flagę `SYN` i został przesłany do serwera w celu nawiązania połączenia
(etap [three-way
handshake](https://pl.wikipedia.org/wiki/Transmission_Control_Protocol#Nawi.C4.85zywanie_po.C5.82.C4.85czenia)).
Każda ze stron zaczyna od losowo wygenerowanego numeru sekwencyjnego. Jako, że ten pakiet został
wysłany przez klienta, otrzymał on numer 2,955,157,473.

Pakiet `127` jest odpowiedzią serwera na pakiet `SYN` klienta. Jemu też został przypisany losowy
numer sekwencyjny 1,348,638,687, jako, że jest to pierwszy pakiet w tej sesji TCP przesłany przez
serwer. Dodatkowo, widzimy, że załączony jest numer potwierdzenia. Jego wartość jest większa o 1 w
stosunku do numeru sekwencyjnego pakietu `126` . Wskazuje on tym samym, że serwer otrzymał pakiet
`126` od klienta. Warto zwrócić uwagę, że numer potwierdzenia został podbity o 1, mimo, że pakiet
`126` nie zawiera żadnego payloadu. Dzieje się to z powodu ustawienia w tym w pakiecie flagi `SYN` .

Pakiet `128` jest odpowiedzią klienta na pakiet `127` serwera. Numer sekwencyjny tego pakietu jest
dokładnie taki sam jak numer potwierdzenia pakietu `127` . Natomiast numer potwierdzenia pakietu
`128` jest o 1 większy w stosunku do numeru sekwencyjnego pakietu `127` , a to z tego względu, że
pakiet `127` ma również ustawioną flagę `SYN` .

W tym momencie, tj. po zakończeniu procesu three-way handshake, numery sekwencyjne na obu hostach
wynoszą odpowiednio: 2,955,157,474 (klient) oraz 1,348,638,688 (serwer). Obie maszyny nie przesłały
jeszcze do siebie żadnych danych ale w procesie witania, każda z nich przesłała po jednym pakiecie z
ustawioną flagą `SYN` . To początkowe podbicie numerów o 1 występuje w przypadku ustanawiania
wszystkich sesji TCP.

Po zestawieniu połączenia, następuje faktyczna wymiana danych między klientem a
serwerem:

[![4.wireshark-numery-sekwencyjne-2]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-2-1024x158.png)]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-2.png)

Klient wysyła pakiet `129` i jest on pierwszym, który zawiera payload. Konkretnie jest to żądanie
HTTP. Numer sekwencyjny zostaje tak jak był, a to ze względu na fakt, że żadne dane nie zostały
przesłane do serwera od chwili otrzymania ostatniego pakietu w strumieniu. Z kolei numer
potwierdzenia również jest taki sam, bo żadne dane nie zostały przesłane z serwera do klienta.
Zwróćmy uwagę na fakt, że payload w tym pakiecie waży 189 bajtów.

Pakiet `132` jest odpowiedzią serwera na wysłane żądanie, która potwierdza odbiór danych w pakiecie
`129` . Numer potwierdzenia został podbity o 189 (2,955,157,663), czyli o wartość payloadu
otrzymanego przez serwer pakietu. Numer sekwencyjny serwera zostaje bez zmian, bo serwer nie wysłał
żadnych danych.

Następne pakiety są już odpowiedzią serwera
HTTP:

[![4.wireshark-numery-sekwencyjne-3]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-3-1024x164.png)]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-3.png)

Pakiet `133` określa początek odpowiedzi. Jego numer sekwencyjny jest cały czas taki sam, bo żaden z
poprzednich pakietów pochodzących z serwera nie zawierał payloadu. Z kolei ten pakiet zawiera dane
ważące `1460` bajtów.

Pakiet `134` ma numer sekwencyjny zwiększony o 189, a to za sprawą potwierdzenia przez serwer
ostatniego pakietu jaki klient przesłał (żądanie HTTP). Teraz z kolei klient potwierdza odebranie
1460 bajtów, które zostały przesłane w pakiecie `133` przez serwer. Podnosi to numer potwierdzenia
serwera o 1460 (1,348,640,148).

Dla większości sytuacji, ten cykl będzie się powtarzał, tj. numer potwierdzenia jednej z maszyn
będzie numerem sekwencyjnym drugiej ale zwiększonym o ilość bajtów w payloadzie przesyłanego
pakietu. Mogliśmy to zaobserwować w polu `Next sequence numer:` :

![]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-5.png)

Zatem 1,348,640,148 + 1460 = 1,348,641,608 .

Numer sekwencyjny klienta przez cały czas trwania komunikacji pozostanie ciągle taki sam, bo ten
klient nie przesyła żadnych danych do serwera, jedynie pakiety potwierdzające odbiór danych. Z kolei
numer sekwencyjny serwera będzie się zwiększał z każdym odesłanym pakietem, bo te zawierają payload
odpowiedzi żądania HTTP.

Ten proces trwa aż do momentu przesłania wszystkich potrzebnych pakietów, które zawierają dane
pobieranego obrazka z serwera
HTTP:

[![4.wireshark-numery-sekwencyjne-6]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-6-1024x203.png)]({{< baseurl >}}/img/2015/06/4.wireshark-numery-sekwencyjne-6.png)

Pakiet `178` jest potwierdzeniem ostatniego segmentu, który klient otrzymał od serwera.

W pakiecie `179` klient wysyła chęć zakończenia połączenia (flaga `FIN`). Numer sekwencyjny i
potwierdzenia tego pakietu pozostaje taki sam jak w przypadku pakietu `178` , a to z tego powodu, że
był to tylko zwykły pakiet potwierdzający odbiór danych ze wcześniejszego segmentu.

Pakiet `180` jest potwierdzeniem zakończenia połączenia przesyłanym przez serwer. Jako, że w
poprzednim pakiecie była ustawiona flaga `FIN` , numer potwierdzenia zostaje podbity o 1, zatem
serwer przyjął prośbę zakończenia połączenia. Numer sekwencyjny zaś pozostaje bez zmian. Podobnie ma
się sprawa w przypadku pakietu `181` , bo również potwierdza odebranie pakietu z ustawioną flagą
`FIN` .

## Graf przepływu pakietów

W opcjach wiresharka mamy miłe narzędzie, które nam ładnie rozrysuje informacje na temat tych
numerów. Znajduje się ono w Statistics =\> Flow Graph . W przypadku połączeń TCP, powinniśmy
otrzymać poniższy obrazek:

![]({{< baseurl >}}/img/2015/06/5.wireshark-graf-flow.png)

Każda linijka to osobny pakiet TCP. Z lewej strony mamy rozrysowane kierunki przesyłu pakietu,
porty, długość segmentu i oczywiście ustawione flagi. Po prawej zaś stronie mamy numery sekwencyjny
i potwierdzenia w zapisie decymalnym.
