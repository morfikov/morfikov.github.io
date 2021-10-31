---
author: Morfik
categories:
- Linux
date: "2015-07-01T10:20:16Z"
date_gmt: 2015-07-01 08:20:16 +0200
published: true
status: publish
tags:
- tcp
- sysctl
- sieć
GHissueID: 155
title: Bufor połączeń w protokole TCP
---

Wraz ze zwiększaniem zapotrzebowania na szybsze łącza internetowe, ograniczenia wynikające z
protokółu TCP zaczęły powoli dawać się ludziom we znaki. Problemem była bariera prędkości, którą
ciężko było pokonać mając do dyspozycji domyślną formę nagłówka protokołu TCP. Było w nim zwyczajnie
za mało miejsca, co zapoczątkowało jego rozbudowę kosztem ilości danych, które można było przesłać w
pojedynczym segmencie. W tym wpisie skupię się głównie na dwóch opcjach jakie zostały dodane do
nagłówka TCP, tj. dynamiczne skalowanie okien oraz znaczniki czasu, bo te dwa parametry nie mogą
wręcz bez siebie istnieć, zwłaszcza gdy rozmawiamy o łączach pokroju 1 czy 10 gbit/s.

<!--more-->
## Mechanizm okien TCP

Mechanizm okien w protokole TCP pozwala kontrolować przepływ danych między dwoma punktami, które się
ze sobą komunikują. Jest to nic innego jak bufor, w którym są gromadzone dane w celu ich
późniejszego przetworzenia. Każda ze stron dysponuje osobnym buforem. W przypadku strony nadającej
jest to bufor na dane wysyłane do odbiorców `tcp_wmem` , natomiast zaś w przypadku strony
odbierającej jest to bufor na dane pochodzące od nadawcy `tcp_rmem` . Taki bufor ma skończoną
pojemność i określa ilość bajtów w tranzycie, które strona nadająca może przestać nie otrzymując
przy tym pakietu potwierdzającego ich odbiór. Trzeba pamiętać, że strona nadająca musi śledzić
wszystkie bajty, które wysłała i które jeszcze nie zostały potwierdzone przez odbiorcę. W przypadku
kiedy potwierdzenie nie nadejdzie przez pewien określony przedział czasu, nadawca retransmituje
pakiet jeszcze raz i robi to sięgając do swojego bufora nadawczego. W momencie gdy jakaś aplikacja
kliencka uczestnicząca w procesie wymiany informacji nie nadąża z ich obróbką, bufor może się
zapełnić sygnalizując tym samym nadającej maszynie, by wstrzymała się ona na moment z wysłaniem
nowych danych. Generalnie rzecz biorąc, nadający widzi w każdym potwierdzonym segmencie rozmiar okna
odbiorcy i bierze go pod uwagę gdy chce wysłać kolejne dane. Nie może jednak wysłać więcej danych
niż zostało to ściśle określone, przynajmniej do momentu aż wszystkie poprzednie segmenty zostaną
potwierdzone. W chwili gdy się to stanie, nowa porcja danych określona przez rozmiar okna może
zostać wysłana. Im takie okno jest mniejsze, tym wolniej dane będą napływać do punktu docelowego i
podobnie w drugą stronę, tj. im jest ono większe, tym więcej danych może zostać przesłane. Przez
taką zmianę rozmiaru okna, serwer i klient są w stanie zapewnić drugą ze stron, że dane są
przesyłane na tyle szybko, że odbiorca jest je w stanie przetworzyć.

Rozważmy dla przykładu sytuację zobrazowaną na poniżej fotce ([źródło](http://www.tcpipguide.com/)).

![bufor-polaczen-tcp](/img/2015/06/1.bufor-polaczen-tcp.png#big)

Klient wysłał żądanie o jakiś plik. Zarówno klient jak i serwer mają rozmiar okna TCP ustawiony na
560 bajtów. Serwer wysyła pierwsze pakiety. W powyższym przypadku, trzeci pakiet się zgubił. Do
klienta docierają pierwsze dane ( `4` ) i mogą one już zostać przesłane do warstwy aplikacji. Zatem
bufor ulega opróżnieniu i pozostaje na poziomie 560 bajtów. Jako, że serwer wysłał szereg segmentów,
musi czekać na potwierdzenie każdego z nich. W tym czasie wszystkie one rezydują w buforze u nadawcy
zostawiając tym samym jedynie 60 bajtów wolnego miejsca. Z chwilą gdy docierają do nadawcy pakiety
potwierdzające odbiór 200 bajtów ( `7` ), te dane mogą zostać usunięte z bufora, wobec czego rozmiar
okna ulega rozszerzeniu o ilość zwolnionego miejsca. Odbiorca otrzymał, co prawda, pakiet numer 4
ale nie wie co się stało z trzecim ( `6` ), dlatego też nie może wysłać do nadawcy potwierdzenia
odbioru tego pakietu. Gdyby to zrobił, sugerowałoby to nadawcy, że klient odebrał wszystkie 500
bajtów, a przecie tak się nie stało. Po pewnym czasie braku potwierdzenia ze strony klienta, serwer
retransmituje zaginiony pakiet ( `8` ) jeszcze raz i tym razem pakiet dociera do odbiorcy. Dane z
pakietu 3 i 4 mogą zostać przesłane do aplikacji, zwalniając tym samym zajmowane miejsce w buforze.
Serwer otrzymuje potwierdzenia odbioru tych dwóch pakietów, wobec czego, może usunąć zbędne dane z
bufora rozszerzając okno do 560 bajtów. I tak ten proces przebiega do wyczerpania danych, które
jedna z stron chce przesłać drugiej. Co ciekawe, tylko trzeci pakiet wymagał retransmisji, a nie
trzeci i czwarty. To za sprawą
[SACK](/post/sack-czyli-selektywne-potwierdzenia-pakietow/), czyli selektywnych
potwierdzeń segmentów.

Rozmiar okna jest kluczowy jeśli chodzi o możliwość przesyłania dużej ilości danych ale to nie jest
jedyny czynnik, który ma tutaj coś do powodzenia. Innym jest RTT (round trip time), czyli czas jaki
jest potrzeby na przesłanie pakietu do odbiorcy i uzyskanie od niego pakietu potwierdzającego
odebranie danych.

Dla przykładu, dla łącz 40 mbit/s o pingu 20 ms, optymalny rozmiar okna to 104,857 bajtów (BDP). BDP
to Bandwidth Delay Product i wylicza się go mnożąc prędkość przez czas opóźnień. W tym przypadku: 40
Mbit/s to 5,242,880 bajtów/s, a 20ms to 0,02/s. Po wymnożeniu tych dwóch wartości przez siebie
dostajemy BDP 104,857. Dobrze jest ustawić maksymalny rozmiar bufora 3-4 krotnie większy, by pakiety
z większymi opóźnieniami mogły podróżować z większą prędkością po kablach.

## Bufor, jego skalowanie i znaczniki czasu

Mamy zaś tylko jeden problem. Pole od rozmiaru okna w nagłówku TCP ma długość 16 bitów, co daje
maksymalny rozmiar okna 65,535 bajtów, czyli 64KiB. Zatem nawet jeśli mamy do dyspozycji ten
maksymalny rozmiar okna, to przy opóźnieniu 20 ms, największa przepustowość łącza jaką osiągniemy to
zaledwie 3,276,800 bajtów/s, czyli 25 mbitów/s. Więcej się nie da, chyba, że zmniejszymy czas
opóźnienia, a z tym jest ciężko. Co się dzieje w takim przypadku? Ano zwyczajnie tracimy 15
mbitów/s.

Zatem jak to jest możliwe, że nawet obecnie osiągamy o wiele większe prędkości? Robimy to przez
obejście ograniczenia rozmiaru okien przez dodanie do protokołu TCP odpowiednich rozszerzeń. Mowa
oczywiście o [skalowaniu okien oraz znacznikach czasu](https://tools.ietf.org/html/rfc1323) .
Skalowanie okna może rozciągnąć jego rozmiar nawet do 1,073,725,440 bajtów, czyniąc tym samym
większy próg dla danych, które mogą być przesyłane przez sieć bez konieczności czekania na pakiet
potwierdzający. Jako, że większa prędkość pociąga za sobą więcej danych, to i [numery
sekwencyjne](/post/numery-sekwencyjne-w-strumieniu-tcp/) skaczą jak szalone. Z
kolei ich pole w nagłówku TCP ma 32 bity, co daję liczbę 4,294,967,296 unikalnych numerów. Może i to
się wydaje dużo ale przy prędkościach rzędu 1 gbit/s, te numery sekwencyjne mogą się wyczerpać, w
zależności od uzyskanej prędkości, po czasie 17-34 sekund. Jeśli jakiś pakiet się w tym czasie
gdzieś zapodzieje, inny może dostać ten sam numer sekwencyjny i zostać zaakceptowany zamiast tego,
który powinien. W takim przypadku nastąpi uszkodzenie danych i my nawet nie zostaniemy o tym w żaden
sposób powiadomieni.

Opcja definiująca skalowanie okien zajmuje 3 bajty w nagłówku TCP. Natomiast jeśli chodzi o
znaczniki czasu, to jest to dodatkowe 10 bajtów. Łącznie daje to 13 bajtów ekstra. Przypominam, że
MTU to 1500 bajtów i im więcej zajmują nagłówki, tym mniej danych może do takiego pakietu się
zmieścić.

Obie te opcje możemy sobie oczywiście dostosować w zależności od tego jakim łączem dysponujemy.
Jeśli mamy jedno z tych nie za szybkich, np. 15 mbit/s , to nie potrzebujemy w sumie żadnych z tych
opcji. Oczywiście przy założeniu, że nie dysponujemy żadną siecią LAN i nie przesyłamy danych
lokalnie z większą prędkością. Jeśli chodzi zaś o sam znacznik czasu, to jeśli nie mamy do
dyspozycji łącz gigabitowych, ta opcja w nagłówku TCP jedynie marnuje nam 10 cennych bajtów, które
mogą zostać przeznaczone na faktyczne dane. Wobec czego przydałoby się wyłączyć ją zupełnie.

Reasumując, musimy skonfigurować dwa parametry i możemy to zrobić przez dopisanie do pliku
`/etc/sysctl.conf` tych poniższych linijek:

    net.ipv4.tcp_window_scaling = 1
    net.ipv4.tcp_timestamps = 0
    net.netfilter.nf_conntrack_timestamp = 0

## Konfiguracja zasobów pamięci

W kernelu mamy szereg innych opcji, które wymagają dostosowania w przypadku gdy przepustowość łącza
jest nie taka jak być powinna. Jako, że większe okna zapewniają większą przepustowość, to pociągają
za sobą także większe wykorzystanie zasobów. Głównie chodzi o pamięć operacyjną RAM. Im większe jest
okno, tym więcej pamięci będzie ono zjadać. Im więcej jest okien, tym jeszcze więcej pamięci będą
one zjadać. Generalnie rzecz biorąc, każde połączenie jakie nawiązujemy w celu pobierania/wysyłania
danych wykorzystuje pewną określoną ilość pamięci. Kiedyś pisałem, że gniazda TCP/UDP identyfikują
konkretne połączenia, w skład których wchodzą lokalny adres, lokalny port, zdalny adres, zdalny port
oraz protokół. Dla każdego z takich gniazd, kernel przydziela kilka stron pamięci. By przeliczyć
strony pamięci na faktyczną jej objętość, trzeba przemnożyć ilość stron przez rozmiar strony. Z
kolei rozmiar strony można uzyskać porównując te dwa poniższe parametry z plików `/proc/meminfo`
oraz `/proc/vmstat` :

    # cat /proc/meminfo | egrep -i "Mapped" && cat /proc/vmstat | egrep -i "nr_mapped"
    Mapped:           158524 kB
    nr_mapped 39631

Rozmiar strony to `Mapped/nr_mapped` , czyli 158524/39631=4KiB albo 4096 bajtów.

Poniższe 3 wartości określają minimalną, umiarkowaną i maksymalną ilość stron pamięci, które mogą
wykorzystać wszystkie gniazda TCP/UDP łącznie i odpowiadają odpowiednio za 64 MiB, 96 MiB i 144 MiB
pamięci. Ten parametr domyślnie jest kalkulowany przy starcie systemu w oparciu o ilość dostępnej
pamięci RAM i nie musimy go wyraźnie precyzować w pliku `/etc/sysctl.conf` :

    net.ipv4.tcp_mem = 16500 24750 37125
    net.ipv4.udp_mem = 16500 24750 37125

Dodatkowo musimy ustawić ilość pamięci, która zostanie przydzielona buforom połączeń. Poniższe
wartości są w bajtach i odnoszą się do wszystkich protokołów:

    net.core.rmem_default = 327680
    net.core.rmem_max = 327680
    net.core.wmem_default = 327680
    net.core.wmem_max = 327680

Musimy jeszcze określić rozmiar buforów dla poszczególnych gniazd TCP. Druga wartość poniższych
parametrów nadpisuje `net.core.rmem_default` oraz `net.core.wmem_default` widoczne wyżej:

    net.ipv4.tcp_wmem = 4096 81920 327680
    net.ipv4.tcp_rmem = 4096 81920 327680

W przypadku gdy system ma dostatecznie dużo pamięci sprecyzowanej w `tcp_mem` , przydziela każdemu
nowo utworzonemu gniazdu z automatu tyle RAMu ile widnieje na drugiej pozycji w parametrze wyżej.
Jeśli istnieje potrzeba, np. w przypadku szybkiego transferu danych czy większych opóźnień, kernel
może zwiększyć limit dla takiego połączenia ale nie może on przekroczyć wartości maksymalnej, w tym
przypadku `327680` bajtów. Z kolei zaś, gdy pamięć będzie na wyczerpaniu, połączenia już utworzone
będą musiały się podzielić pamięcią z nowo tworzonymi gniazdami. W przypadku tych połączeń,
zostanie ograniczony transfer poprzez zmniejszenie bufora.

Istnieje jeszcze jedna ciekawa opcja, która umożliwia automatyczne dostrajanie wielkości bufora
odbiorczego i co za tym idzie również okna TCP dla każdego połączenia. Maksymalna wartość bufora nie
może jednak przekroczyć tego zdefiniowanego w `tcp_rmem` . Jeśli interesuje nas tego typu mechanizm,
dopisujemy poniższą linijkę do pliku `/etc/sysctl.conf` :

    net.ipv4.tcp_moderate_rcvbuf = 1

Po dostosowaniu powyższych opcji, wystarczy wydać poniższe polecenie by zmiany weszły w życie:

    # sysctl -p
