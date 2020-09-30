---
author: Morfik
categories:
- Linux
date: "2015-06-24T21:44:42Z"
date_gmt: 2015-06-24 19:44:42 +0200
published: true
status: publish
tags:
- tcp
- sieć
title: Flagi TCP i przełączanie stanów połączeń
---

Jakiś czas temu opisywałem [jak zaprojektować swój własny
firewall](/post/firewall-na-linuxowe-maszyny-klienckie/), wobec czego postanowiłem
nieco bardziej pochylić się nad zagadnieniem stanów połączeń i je dokładniej przeanalizować. Ten
wpis dotyczy głównie protokołu TCP, bo ten UDP jest bezpołączeniowy, więc nie ma tam żadnych stanów.
Dodatkowo opiszę tutaj poszczególne flagi, które mogą zostać ustawione w pakietach zmieniając tym
samym stan połączenia.

<!--more-->
## Schemat stanów połączeń

Zacznijmy zatem od rozrysowania schematu wszystkich możliwych stanów jakie może przyjąć połączenie
oparte na protokole TCP. Poniżej stosowna fotka
([źródło](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)):

![](/img/2015/06/1.schemat-przelaczania-stanow-flagi-tcp.png#huge)

Poniżej zaś znajduje się krótkie objaśnienie użytych nazw na powyższym schemacie:

  - `LISTEN` -- gotowość do przyjęcia połączenia na określonym porcie przez serwer.
  - `SYN-SENT` -- pierwsza faza nawiązywania połączenia przez klienta. Wysłano pakiet z flagą SYN.
    Oczekiwanie na pakiet SYN+ACK.
  - `SYN-RECEIVED` -- otrzymano pakiet SYN, wysłano SYN+ACK. Trwa oczekiwanie na ACK. Połączenie
    jest w połowie otwarte (half-open).
  - `ESTABLISHED` -- połączenie zostało prawidłowo nawiązane. Prawdopodobnie trwa transmisja danych.
  - `FIN-WAIT-1` -- wysłano pakiet FIN. Dane wciąż mogą być odbierane ale wysyłanie jest już
    niemożliwe.
  - `FIN-WAIT-2` -- otrzymano potwierdzenie własnego pakietu FIN. Oczekuje na przesłanie FIN od
    serwera.
  - `CLOSE-WAIT` -- otrzymano pakiet FIN, wysłano ACK. Oczekiwanie na przesłanie własnego pakietu
    FIN (gdy aplikacja skończy nadawanie).
  - `CLOSING` -- jednoczesne obustronne zamknięcie połączenia.
  - `LAST-ACK` -- otrzymano i wysłano FIN. Trwa oczekiwanie na ostatni pakiet ACK.
  - `TIME-WAIT` -- oczekiwanie w celu upewnienia się, że druga strona otrzymała potwierdzenie
    rozłączenia. Zgodnie z RFC 793 połączenie może być w stanie TIME-WAIT najdłużej przez 4
    minuty.
  - `CLOSED` -- połączenie jest zamknięte.

Stan połączenia możemy odczytać bezpośrednio przez plik `/proc/net/ip_conntrack` lub
`/proc/net/nf_conntrack` , są tam wpisy podobne do tych
    poniżej:

    ipv4     2 tcp      6 2 CLOSE src=192.168.1.150 dst=82.240.169.124 sport=53675 dport=30728 src=82.240.169.124 dst=192.168.1.150 sport=30728 dport=53675 [ASSURED] mark=5 zone=0 use=2
    ipv4     2 tcp      6 101 SYN_SENT src=192.168.1.150 dst=188.36.238.212 sport=53629 dport=56869 [UNREPLIED] src=188.36.238.212 dst=192.168.1.150 sport=56869 dport=53629 mark=5 zone=0 use=2
    ipv4     2 tcp      6 27464 ESTABLISHED src=192.168.1.1 dst=192.168.1.150 sport=36840 dport=514 src=192.168.1.150 dst=192.168.1.1 sport=514 dport=36840 [ASSURED] mark=10 zone=0 use=2
    ipv4     2 udp      17 3 src=84.123.97.152 dst=192.168.1.150 sport=4444 dport=6666 src=192.168.1.150 dst=84.123.97.152 sport=6666 dport=4444 mark=5 zone=0 use=2

Przeanalizujmy sobie pierwszy wpis. Zgodnie z [tym
linkiem](https://stackoverflow.com/questions/16034698/details-of-proc-net-ip-conntrack-nf-conntrack),
poszczególne kolumny odpowiadają:

  - `ipv4` -- nazwa protokółu warstwy sieciowej
  - `2` -- numer protokołu warstwy sieciowej
  - `tcp` -- nazwa protokołu warstwy transportowej
  - `6` -- numer protokołu warstwy transportowej
  - `2` -- czas ważności (w sekundach) wpisu w tablicy conntrack'a
  - `CLOSE` -- stan połączenia w przypadku protokołu TCP

Następne 4 kolumny są w formacie `klucz=wartość` i oznaczają kolejno:

  - `src=192.168.1.150` -- adres IP, który zapoczątkował połączenie
  - `dst=82.240.169.124` -- adres IP na drugim końcu połączenia
  - `sport=53675` -- port źródłowy, z którego został wysłany pakiet
  - `dport=30728` -- port docelowy, na który dotarł pakiet

Kolejne 4 kolumny są dokładnie w takiej samej formie jak te powyższe, z tym, że mają zamienione
wartości `src` z `dst` oraz `sport` z `dport` . Jeśli by to był router, wtedy drugie pole `dst`
zostanie przepisane na odpowiedni adres routera. Może to wyglądać, np.
    tak:

    ipv4     2 tcp      6 51 SYN_SENT src=192.168.1.150 dst=91.214.0.142 sport=52474 dport=38819 [UNREPLIED] src=91.214.0.142 dst=11.22.33.44 sport=38819 dport=52474 packets=0 bytes=0 mark=0 use=2

To dzięki takiemu mechanizmowi, router wie gdzie posłać konkretne pakiety.

Dalej mogą wystąpić dwie flagi: `UNREPLIED` lub `ASSURED` . Jeśli pierwsza z nich jest ustawiona,
oznacza to, że pakiety zostały wysłane na drugą stronę połączenia ale bez odpowiedzi. Z kolei jeśli
jest widoczna druga flaga, to ruch pakietów odbywa się w obu kierunkach.

Kolejne pola odpowiadają za:

  - `packets` -- ilość przesłanych pakietów
  - `bytes` -- ilość przesłanych bajtów
  - `mark` -- marker przypisany połączeniu
  - `zone` -- brak danych
  - `use` -- brak danych

## Flagi TCP

By stany mogły przechodzić z jednego w drugi, czyli zmieniać się, potrzebne są pakiety kontrolne.
Takie pakiety zwykle nie zawierają żadnych danych i mają ustawione konkretne flagi sterujące
połączeniem. W nagłówku TCP jest miejsce na 8 flag. Obrazuje je powinna fotka
([źródło](https://nmap.org/book/tcpip-ref.html)):

![](/img/2015/06/2.naglowek-tcp-flagi.png#huge)

Flagi `CWR` , `ECE` odpowiadają za mechanizm kontroli zatorów.

Flaga `URG` odpowiada za oznaczanie przychodzących danych jako pilne. Takie segmenty nie muszą
czekać aż wszystkie poprzednie zostaną przetworzone na drugim końcu połączenia. Zamiast tego są
przesyłane i przetwarzane praktycznie natychmiastowo. Przykładowo, flaga `URG` może zostać
zastosowana, np. w przesyłaniu strumienia danych (vod, voip) między dwiema maszynami, powiedzmy A i
B. W przypadku gdy z jakiegoś powodu maszyna A musi zakończyć transfer danych i te dane muszą jak
najszybciej przestać być przetwarzane na drugim końcu, to w normalnych okolicznościach musielibyśmy
zakolejkować pakiet kończący połączenie i czekać aż maszyna B go przetworzy. W przypadku ustawienia
flagi `URG` , ten pakiet dostaje się od razu na początek kolejki i jest natychmiast przetwarzany.
Czyli krótko mówiąc, pakiet ma większy priorytet niż wszystkie pozostałe w kolejce.

Można to zobrazować również na przykładzie poczty (raczej nie polskiej). Załóżmy, że mamy setki
ciężarówek, które rozładowują worki z listami z całego świata. Ponieważ jest ich dużo, muszą
ustawić się w kolejce jedna za drugą i czekać na swoją kolej rozładunku, co może skutkować dość
długą kolejką. Jeśli teraz nadjedzie tam uprzywilejowany pojazd mający na pace bardzo ważne listy
od Snowdena (oznaczmy go czerwoną flagą), to raczej nie powinien stać w kolejce, tylko od razu musi
zostać wpuszczony poza nią by jak najszybciej dojechać na miejsce przeznaczenia.

W przykładzie z pocztą, ciężarówki są odpowiednikiem segmentów przesyłanych przez sieć, które
docierają do swojego miejsca przeznaczenia i są kolejkowane w buforze czekając na przetworzenie.
Natomiast ciężarówka z czerwoną flagą robi za segment z ustawioną flagą `URG` .

Flaga `ACK` jest używana do potwierdzania pomyślnego odebrania danych w segmentach. Za każdym razem
gdy pakiet zawierający payload jest transmitowany przez sieć, te dane muszą zostać potwierdzone, co
odbywa się przez [numery sekwencyjne.](/post/numery-sekwencyjne-w-strumieniu-tcp/)

Flaga `PSH` jest bardzo podobna do flagi `URG` i istnieje w celu zapewnienia, że dane otrzymają
określony priorytet i zostaną przetworzone na jednym z końców transmisji. Ta flaga w szczególności
jest używana na początku i końcu transmisji danych -- po three-way handshake i przed pierwszym
pakietem z ustawioną flagą `FIN` . Ustawienie tej flagi w ostatnim segmencie przesyłanego pliku,
zapobiega zakleszczeniu się bufora TCP, co jest często widoczne gdy żądania HTTP są przesyłane przez
różnego rodzaju proxy.

Gdy host wysyła jakieś dane, to są one tymczasowo kolejkowane w buforze TCP. Jest to taki specjalny
obszar w pamięci, gdzie dane przebywają do czasu aż segment osiągnie określony rozmiar. Wtedy jest
on przesyłany do odbiorcy. Tego typu mechanizm gwarantuje, że transfer danych jest efektywny jak to
tylko możliwe nie marnując przy tym czasu i przepustowości łącza przez tworzenie wielu segmentów.
Zamiast tego, wszystkie one są połączone w jeden lub kilka większych segmentów.

Gdy taki segment dociera do odbierającego hosta, to zanim zostanie przesłany do warstwy aplikacji,
jest on umieszczany w buforze TCP przeznaczonym na pakiety przychodzące. Zakolejkowane w taki sposób
dane zostaną w tym buforze do czasu nadejścia pozostałych segmentów, a gdy to nastąpi, wszystkie te
dane zostaną przesłane do warstwy aplikacji, która ich oczekuje.

Ta procedura działa w większości przypadków, z tym, że nie zawsze takie kolejkowanie danych jest
pożądane. Pewne aplikacje nie działają jak należy gdy występują opóźnienia w wyniku kolejkowania
pakietów. Przykładem może być player, który odtwarza jakiś strumień video. By film był płynnie
odtwarzany (bez zacięć), dane muszą być przesyłane i przetwarzane natychmiast jak tylko się
pojawiają.

Flaga `RST` jest wykorzystywana w przypadku gdy docierający do odbiorcy segment nie jest
przeznaczony dla tego konkretnego połączenia. Innymi słowy, jeśli klient zamierzał wysłać pakiet do
serwera w celu nawiązania połączenia ale na serwerze dana usługa nie nasłuchiwała w tym czasie na
określonym porcie, to w takim przypadku zostanie wysłany pakiet z ustawioną flagą `RST` do maszyny
klienckiej odrzucając jej zapytanie. Wydaje się to być oczywiste i logiczne ale tego typu pakiety są
wykorzystywane w niecnych celach do skanowania otwartych portów.

Flaga `SYN` jest jedną z bardziej znanych flag, bo rozpoczyna każde nowe połączenie, na którego to
starcie są synchronizowane [początkowe numery
sekwencyjne](/post/numery-sekwencyjne-w-strumieniu-tcp/). Ta flaga jest ustawiana w
pierwszym i drugim pakiecie przywitania three-way handshake pomiędzy dwoma hostami.

Flaga `FIN` kończy połączone rozpoczęte przez pakiet z ustawioną flagą `SYN` . Ten pakiet pojawia
się po przesłaniu wszystkich segmentów w połączeniu sygnalizując tym samym koniec przesyłania
danych. Warto zanotować, że w przypadku gdy klient wysyła pakiet `FIN` do serwera, oznacza to, że
klient nie może przesłać już więcej danych na serwer ale w dalszym ciągu jest w stanie odbierać dane
z serwera do chwili gdy zdalny host również zakończy połączenie pakietem `FIN` . W chwili gdy
połączenie zostaje zakończone na obu końcach, bufory TCP są zwalnianie.
