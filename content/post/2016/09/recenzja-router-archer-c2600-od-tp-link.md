---
author: Morfik
categories:
- Hardware
date: "2016-09-14T18:20:30Z"
date_gmt: 2016-09-14 16:20:30 +0200
published: true
status: publish
tags:
- wifi
- router
- recenzja
- tp-link
title: 'Recenzja: Router Archer C2600 od TP-LINK'
---

Któż z nas nie ma gdzieś koło swojego domowego ogniska takiego małego urządzenia z antenami, które
nic tylko sobie leży gdzieś na półce i od czasu do czasu tylko mignie do nas swoimi diodami? Mowa
oczywiście o routerach WiFi, które można powoli chyba zacząć traktować jak nowych członków naszej
rodziny. Te elektroniczne stworzenia są tak bardzo użyteczne, że wielu z nas już sobie życia bez
nich nie wyobraża. Technologia bezprzewodowej łączności rozwija się bardzo prężnie i, gdy niecałe
dwa lata temu sprawiłem sobie mój pierwszy router WiFi (model
[TL-WR1043ND](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) v2), to myślałem, że
wystarczy mi on na jakiś dłuższy czas. Wtedy 450 mbit/s w paśmie 2,4 GHz to były naprawdę przyzwoite
osiągi, przynajmniej teoretycznie rzecz ujmując. Od tamtego czasu bardzo dużo się zmieniło w kwestii
WiFi, nie tylko pod względem przepustowości ale także technologii, które znajdują zastosowanie w tym
bezprzewodowym wymiarze otaczającej nas rzeczywistości. Tak się składa, że mam jedno z tych bardziej
zaawansowanych urządzeń i postanowiłem sobie je dokładnie obadać. Jest to [router Archer
C2600](http://www.tp-link.com.pl/products/details/Archer-C2600.html) od TP-LINK, który ma być zdolny
przesłać sygnał po WiFi z prędkością 800 mbit/s w paśmie 2,4 GHz i 1733 mbit/s w paśmie 5 GHz, także
zobaczmy co to jest za maszyna.

<!--more-->
## Zawartość pudełka

Jak zawsze, zacznijmy od tego co znajduje się wewnątrz opakowania. Poniżej są fotki samego pudełka
oraz wszystkich elementów załączonych w
zestawie:

![]({{< baseurl >}}/img/2016/09/1.router-archer-c2600-pudelko.jpg#huge)

![]({{< baseurl >}}/img/2016/09/2.router-archer-c2600-pudelko-zawartosc.jpg#huge)

Przede wszystkim, w pudełku mamy router. Jak widać jest to dość byczy sprzęt (mniej więcej 2x
TL-WR1043ND). Jego dokładne wymiary to jakieś 264x198x38mm (dł-sz-wy) i waży prawie 1 kilogram.
Połowa powierzchni tego routera robi za matową kratkę wentylacyjną, a połowa za coś co się świeci
jak nie powiem komu co. Przynajmniej router jest tylko w połowie łatwy do zarysowania, także krok w
dobrą stronę.

![]({{< baseurl >}}/img/2016/09/3.router-archer-c2600-wyglad.jpg#huge)

Na środku tej dwurodzajowej powierzchni znajdują się diody. Diody świecą się na kolor biały, za
wyjątkiem tej, która ma sygnalizować połączenie ze światem. W przypadku podłączenia przewodu i
braku łączności z internetem, router podświetla diodę na pomarańczowo. Diody można wyłączyć
przyciskiem lub też zaprogramować ich automatyczne włączanie i wyłączanie w panelu administracyjnym
w zależności od pory dnia. Bardzo przydatna rzecz jeśli w nocy nam te diody nie dają spać.

Spód oczywiście jest cały zakratkowany i na szczęście matowy:

![]({{< baseurl >}}/img/2016/09/4.router-archer-c2600-wyglad-spod.jpg#huge)

Do zestawu są dołączone cztery anteny dwupasmowe:

![]({{< baseurl >}}/img/2016/09/5.router-archer-c2600-anteny.jpg#huge)

Jeden przewód ethernet
CAT5E:

![]({{< baseurl >}}/img/2016/09/6.router-archer-c2600-skretka.jpg#huge)

Nie mogło też zabraknąć zasilacza:

![]({{< baseurl >}}/img/2016/09/7.router-archer-c2600-zasilacz.jpg#huge)

![]({{< baseurl >}}/img/2016/09/8.router-archer-c2600-zasilacz.jpg#huge)

Jak widzimy, na zasilaczu widnieje 12V/4A, zatem ten router jest w stanie wyciągnąć prawie 50 W.
Oczywiście w przypadku standardowej pracy tego urządzenia, pobiera ono sporo mniej energii i jest to
rząd wielkości 7-10 W. No chyba, że podłączymy do niego urządzenia USB. Archer C2600 posiada również
dwa porty USB 3.0:

![]({{< baseurl >}}/img/2016/09/9.router-archer-c2600-porty-usb-przyciski.jpg#huge)

Wyżej poza portami widzimy również przyciski WiFi i WPS oraz małą dziurkę, która skrywa przycisk
reset. Wymagana będzie igła czy inny tego typu przedmiot, by ten ostatni przycisk wcisnąć.

Tylny panel routera Archer C2600 prezentuje się raczej standardowo:

![]({{< baseurl >}}/img/2016/09/10.router-archer-c2600-switch-power.jpg#huge)

Mamy cztery żółte gniazdka RJ-45 przeznaczone na porty LAN. Dalej jest też jeden niebieski port WAN.
No i oczywiście włącznik oraz gniazdo zasilania. Widzimy również cztery złącza antenowe RP-SMA.

To mniej więcej tyle jeśli chodzi o zawartość pudełka i wygląd samego routera. Zobaczmy czym
dokładnie ten sprzęt jest nas w stanie do siebie przekonać.

## Specyfikacja routera Archer C2600

Router Archer C2600 pracuje w standardzie 802.11ac i jest w stanie osiągać przepustowość 800 mbit/s
w paśmie 2,4 GHz oraz 1733 mbit/s w paśmie 5 GHz. Niemniej jednak, szereg warunków musi być
spełniony, by być w stanie osiągnąć taki transfer. Przede wszystkim, szerokość kanału musi być
ustawiona na 40 MHz w paśmie 2,4 GHz oraz 80 MHz w paśmie 5 GHz. Oczywiście ta wysoka przepustowość
zależy także od zastosowanej techniki modulacji (QAM-256), liczby strumieni przestrzennych (4),
sprawności kodera (5/6) oraz od interwału ochronnego (krótki).

Trzeba jednak pamiętać, że te cyferki 800/1733 mbit/s są wartościami teoretycznymi, a realny
transfer jest nieco niższy. To jakie wyniki uzyskamy zależą głównie od warunków w jakich router
będzie pracował oraz parametrów, które klient wynegocjuje w oparciu o jakość odbieranego sygnału
WiFi. Więcej informacji na temat teoretycznej przepustowości sieci WiFi w warstwie fizycznej
(interfejs radiowy) można znaleźć w [indeksie MCS](http://mcsindex.com/) (Modulation and Coding
Scheme).

### Szerokość kanałów

Termin szerokości kanału jest nam raczej wszystkim znany. Zasada jest zawsze taka sama, tj. im
szerszy kanał, tym więcej danych jesteśmy w stanie przesłać w jednostce czasu. Niemniej jednak,
szerokie kanały, zwłaszcza w warunkach miejskich, mogą powodować degradację sygnału w skutek
zakłóceń generowanych przez sąsiednie sieci, zwłaszcza w przypadku blokowisk, czy innych tego
większych skupisk ludzkich. Trzeba ten czynnik brać pod uwagę w przypadku, gdy mieszkamy w tak
zatłoczonym miejscu.

Oczywiście częstotliwość 5 GHz nie jest tak bardzo zaszumiona jak pasmo 2,4 GHz i tutaj sytuacja
powinna być nieco lepsza ale też trzeba liczyć się z większym tłumieniem, które towarzyszy falom o
częstotliwości 5 GHz.

### Technika modulacji i sprawność kodera (coding rate)

W routerze Archer C2600 mamy do czynienia z techniką modulacji QAM-256 (kwadraturowa modulacja
amplitudowo-fazowa) ze sprawnością kodera 5/6, czyli 83% danych przesyłanych po WiFi to są faktyczne
dane przesyłane przez sieć, reszta to narzut protokołu (overhead). Ta technika jest wykorzystywana
zarówno w paśmie 2,4 GHz jak i 5 GHz. Więcej informacji na temat [technik wykorzystywanych do
przesyłania sygnału drogą radiową](https://www.cyberbajt.pl/raport/499/0/503/) można znaleźć tutaj.
Z kolei na YT jest [ciekawy materiał opisujący i porównujący różne techniki
modulacji](https://www.youtube.com/watch?v=d7l5NbFfBiU).

Dzięki wykorzystaniu modulacji QAM-256 5/6, możliwe jest zakodowanie większej ilości informacji w
kanale o takiej samej szerokości. Dla porównania, w przypadku modulacji 64 QAM 5/6 w jednym [symbolu
OFDM](https://www.cyberbajt.pl/raport/443/0/449/) można było zakodować 6 bitów. Modulacja QAM-256
5/6 podnosi ten limit do 8 bitów, co przekłada się na zwiększenie przepustowości o jakieś 33%.

Najważniejszą rzeczą, z której powinniśmy sobie zdawać sprawę w przypadku tych technik modulacji
jest fakt, że wraz ze wzrostem złożoności techniki, maleje dość znacznie margines błędu jaki
odbiornik może popełnić przy dekodowaniu symboli OFDM. Sygnał musi być zatem bardzo dobrej jakości,
czyli posiadać duży współczynnik sygnału do szumu (SNR). W warunkach miejskich może być z tym
problem, zwłaszcza w przypadku WiFi w paśmie 2,4 GHz.

### Anteny i strumienie przestrzenne (spatial sterams)

Router Archer C2600 dysponuje czterema dwupasmowymi zewnętrznymi antenami nadawczo-odbiorczymi (8
wzmacniaczy, 4 na każde pasmo). Zatem nie mamy tutaj słabych anten wewnętrznych dla jednego z pasm,
tak jak to mają niektóre słabsze modele routerów.

Cztery anteny w routerze są w stanie transmitować dane czterema różnymi strumieniami przestrzennymi.
Każdy strumień jest w stanie przesłać 200 mbit/s i 433 mbit/s odpowiednio w paśmie 2,4 GHz i 5 GHz.
Dzięki zastosowanej technologi MIMO, tego typu zabieg jest nam w stanie poprawić przepustowość sieci
WiFi czterokrotnie.

### Interwał ochronny (Guard Interval)

Interwał ochronny ma za zadanie minimalizować zakłócenia wynikające z transmisji danych w sieci
bezprzewodowej. Chodzi o to, że transmisja symboli OFDM może ulec opóźnieniu, czego efektem będzie
nałożenie się na siebie dwóch lub więcej symboli po dotarciu do odbiornika. Taka sytuacja powoduje
osłabienie współczynnika SNR, czyli sygnał ulega w ten sposób degradacji i spada nam transfer.

Interwał ochronny określa czas ciszy pomiędzy symbolami i jest wyrażony w mikrosekundach. Mamy dwa
rodzaje Guard Interval: krótki 0,4 us i długi 0,8 us. W przypadku długiego interwału będziemy
notować mniej zakłóceń. W warunkach domowych, gdzie odległości odbiorników od routera nie są duże i
nie ma w okolicy za wiele urządzeń zdolnych zakłócać transmisję bezprzewodową, można korzystać z
krótkiego Guard Interval, przez co transfer danych może zostać zwiększony o jakieś 10% (bonus już
wliczony).

### Technologia MU-MIMO

Całą masa kart bezprzewodowych jest wyposażona tylko w jedną antenę, przez co pasmo WiFi nie jest
efektywnie wykorzystywane. Chodzi o to, że większość routerów jest w stanie rozmawiać tylko z jednym
urządzeniem w danej jednostce czasu (technologia SU-MIMO, Single-User Multiple Input Multiple
Output).

Jeśli mamy smartfona z jedną anteną w paśmie 5 GHz, to moglibyśmy wykorzystać tylko jeden strumień
routera. W przypadku, gdy router dysponuje większą ilością strumieni, to cześć pasma nie może być
wykorzystana w tej jednostce czasu, a pozostałe klienty muszą czekać na swoją kolej. Mniej więcej
jest to taka sama sytuacja co w przypadku HUB'a, tego urządzenia sieciowego, które było
wykorzystywane lata temu do rozdzielania sygnału na kilka komputerów w sieci przewodowej.

W przypadku routera Archer C2600 sprawa wygląda nieco inaczej, bo w tym urządzeniu został
zaimplementowany mechanizm MU-MIMO (Multi-User Multiple Input Multiple Output). Ma on za zadanie
zwiększyć ilość obsługiwanych jednocześnie urzadzeń tak, by klienci WiFi nie czekali na swoją
kolej. W tym przypadku router się zachowuje bardziej jak switch, a nie HUB. Niemniej jednak, ta
technologia nie jest taka wypasiona jak mogłoby się wydawać na pierwszy rzut oka.

Przede wszystkim dotyczy ona jedynie pasma 5 GHz. Zatem jeśli posiadamy szereg urządzeń zdolnych
łączyć się po WiFi i większość (albo i wszystkie) z nich potrafi to robić jedynie w paśmie 2,4
GHz, to MU-MIMO nam zbytnio nic nie da. By odczuć różnicę, trzeba by zmodernizować swoje domowe
zabawki.

Drugi problem z technologią MU-MIMO jest taki, że jest ona w stanie obsłużyć jednocześnie tyle
urządzeń, iloma strumieniami przestrzennymi (spatial streams) dysponuje router. W przypadku routera
Archer C2600 będą to maksymalnie cztery urządzenia. Każde następne urządzenie będzie kolejkowane.
Niemniej jednak, cztery urządzenia w stosunku do jednego, to całkiem przyzwoity krok na przód. Na
warunki domowe się nada w sam raz.

### Kształtowanie wiązki (Transmit Beamforming)

Mając do dyspozycji cztery anteny, router Archer C2600 jest w stanie ukształtować nieco wiązkę
wysyłanego sygnału do stacji odbiorczych. W ten sposób sygnał docierający do klienta ulega
poprawie. Owocuje to zwiększonym zasięgiem, co przekłada się na lepszy transferem danych u klientów
sieci WiFi, także podczas ich przemieszczania się.

[Zasada działania beamforming'u](https://www.cyberbajt.pl/raport/443/0/446/) jest stosunkowo dość
prosta. Cztery anteny routera Archer C2600 wysyłają cztery sygnały radiowe. Gdy takie sygnały
docierają do odbiornika, są one sumowane. Problem w tym, że drogą, jaką musi przebyć sygnał radiowy
wysłany z oddalonych od siebie fizycznie anten, jest różna. To z kolei powoduje przesunięcie faz
sygnałów i w standardowej sytuacji sygnał ulega wytłumieniu. W przypadku zastosowania kształtowania
wiązki, fazy wysłanych sygnałów są precyzyjnie dobierane, przez co likwidowane jest ich przesunięcie
i do odbiornika dociera mocniejszy sygnał (zwiększa się SNR). [Tutaj jest materiał obrazujący ten
proces](https://www.youtube.com/watch?v=8rMtqRObvvU).

## Podzespoły routera Archer C2600

Przyjrzyjmy się zatem nieco podzespołom z jakich zbudowany jest router Archer C2600. By się do tego
urządzenia dobrać, trzeba pierw wyciągnąć cztery nóżki ze spodu obudowy i odkręcić skrywane pod nimi
śrubki:

![]({{< baseurl >}}/img/2016/09/11.router-archer-c2600-obudowa-otwieranie.jpg#huge)

Później wystarczy podważyć nieco pokrywę routera i powinna ona bez problemu zeskoczyć z zaczepów.
Dobrze jest poszukać ciemniejszych miejsc, w których znajdują się zaczepy i tam podważać. Po
ściągnięciu pokrywy trzeba nieco uważać, bo diody na niej są połączone z płytą główną:

![]({{< baseurl >}}/img/2016/09/12.router-archer-c2600-obudowa-otwieranie.jpg#huge)

Po odłączeniu przewodu, mamy dostęp do płytki. Pierwsze co rzuca się w oczy do dwa masywne
radiatory:

![]({{< baseurl >}}/img/2016/09/13.router-archer-c2600-wnętrze-obudowy-pcb.jpg#huge)

A tu jeszcze widok płytki od spodu:

![]({{< baseurl >}}/img/2016/09/14.router-archer-c2600-wnętrze-obudowy-pcb.jpg#huge)

By wyjąc płytkę z obudowy, trzeba pierw odłączyć widoczne wyżej cztery przewody antenowe. Po
ściągnięciu radiatorów, płytka prezentuje się następująco:

![]({{< baseurl >}}/img/2016/09/13.router-archer-c2600-pcb.jpg#huge)

![]({{< baseurl >}}/img/2016/09/15.router-archer-c2600-pcb.jpg#huge)

Poniżej zaś jest nieco bardziej dokładny opis poszczególnych elementów.

### Gigabitowy switch (QCA8337)

Gigabitowy switch jest na układzie [Qualcomm Atheros
QCA8337](https://www.google.pl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjs2K6c1ozPAhXHiCwKHexhBEcQFggeMAA&url=http%3A%2F%2Fwww.deyisupport.com%2Fcfs-file.ashx%2F__key%2Ftelligent-evolution-components-attachments%2F00-25-01-00-00-20-73-71%2FQCA8337N_5F00_Data_5F00_Sheet_5F00_MKG_2D00_17793_5F00_v1.0.pdf&usg=AFQjCNFzHXL7j-83SrjvtndZswPFYAdl_g&sig2=VwASZsnSRnCtmSgk1IXBVA):

![]({{< baseurl >}}/img/2016/09/16.router-archer-c2600-pcb-switch.jpg#huge)

Oraz układy [Group-Tek HST-24002SAR i
HST-48002SAR](https://www.google.pl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjex76_0IrPAhWHhSwKHYOxBmwQFgggMAA&url=http%3A%2F%2Fhands.com%2F~lkcl%2Feoma%2Frouter%2FHST-24002SAR.pdf&usg=AFQjCNFmi-mKi1kTlXJXvyXYcB9l8gFyzw&sig2=y-mSyYASUsoJHITuKkpQoQ):

![]({{< baseurl >}}/img/2016/09/17.router-archer-c2600-pcb-switch.jpg#huge)

### WiFi 2,4 GHz (QCA9980 + SKY2623L)

W sekcji radia 2,4 GHz (4x4:4, standard N) znajduje się czip WiFi QCA9980 oraz 4 [wzmacniacze
sygnału
SKY2623L](https://www.google.pl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjhrIb4yorPAhWE1ywKHT5qCZ8QFggeMAA&url=http%3A%2F%2Fwww.skyworksinc.com%2Fuploads%2Fdocuments%2FSE2623L_202397E.pdf&usg=AFQjCNGEtCcUcuYoHMzJ7rY825QiaI7GXw&sig2=I0IpxTVVN9zgLKKnC5jWyw).

![]({{< baseurl >}}/img/2016/09/18.router-archer-c2600-pcb-wifi-wireless.jpg#huge)

![]({{< baseurl >}}/img/2016/09/19.router-archer-c2600-pcb-wifi-wireless.jpg#huge)

### WiFi 5 GHz (QCA9980 + SKY85405)

W sekcji radia 5 GHz (4x4:4, standard AC) znajduje się czip WiFi QCA9980 oraz 4 [wzmacniacze sygnału
SKY85405](https://www.google.pl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjh7N-JyorPAhXHCCwKHZ-KAA8QFggeMAA&url=http%3A%2F%2Fwww.skyworksinc.com%2Fuploads%2Fdocuments%2FSKY85405_11_PS_203617A.pdf&usg=AFQjCNGTGHYofR6d2cZ4uiC9wetuYSNQvA&sig2=fLRx2OE2s8g5MGjnaTW9fQ)

![]({{< baseurl >}}/img/2016/09/20.router-archer-c2600-pcb-wifi-wireless.jpg#huge)

![]({{< baseurl >}}/img/2016/09/21.router-archer-c2600-pcb-wifi-wireless.jpg#huge)

### System-On-Chip (IPQ8064)

W routerze Archer C2600 znajduje się [SoC
IPQ8064](http://www.anandtech.com/show/7526/qualcomm-atheros-announces-new-internet-processor-lineup-ipq8064-and-ipq8062),
w którym to z kolei mamy dwa procesory Krait 300 z taktowaniem 1,4 GHz.

![]({{< baseurl >}}/img/2016/09/22.router-archer-c2600-pcb-soc-procesor.jpg#huge)

### Pamięć operacyjna RAM

Router Archer C2600 jest wyposażony w 512 MiB pamięci operacyjnej RAM. Zatem jest to dość sporo i na
pewno nie będzie się marnować pod alternatywnym firmware OpenWRT/LEDE. Na pokładzie mamy dwie kości
po 256 MiB DDR3. Producentem układów jest zaś Micron, model
[MT41K128M16JT-125:K](https://www.micron.com/parts/dram/ddr3-sdram/mt41k128m16jt-125):

![]({{< baseurl >}}/img/2016/09/23.router-archer-c2600-pcb-pamiec-ram.jpg#huge)

### Flash

Router Archer C2600 ma flash o wielkości 32 MiB. Układ Spansion FL256SAIFR0:

![]({{< baseurl >}}/img/2016/09/24.router-archer-c2600-pcb-flash.jpg#medium)

## Zarządzanie routerem Archer C2600

Routerem Archer C2600 zarządza się przez panel administracyjny, który jest dostępny pod adresem
`http://192.168.0.1/` . Dane logowania są standardowe, tj. `admin`/`admin` :

![]({{< baseurl >}}/img/2016/09/25.router-archer-c2600-panel-admina-logowanie.png#huge)

Po zalogowaniu się na router, zostaniemy poproszeni o zmianę hasła:

![]({{< baseurl >}}/img/2016/09/26.router-archer-c2600-panel-admina-zmiana-hasla.png#huge)

Następnie zostaniemy przeprowadzeni przez szybki proces konfiguracji routera Archer C2600, podczas
którego będziemy w stanie dostosować kraj oraz strefę czasową:

![]({{< baseurl >}}/img/2016/09/27.router-archer-c2600-panel-admina-strefa-czasowa.png#huge)

Adresację IP:

![]({{< baseurl >}}/img/2016/09/28.router-archer-c2600-panel-admina-adresacja.png#huge)

Nazwy i hasła sieci WiFi:

![]({{< baseurl >}}/img/2016/09/29.router-archer-c2600-panel-admina-wifi.png#huge)

Jak widzimy, proces konfiguracji routera Archer C2600 jest bardzo prosty, praktycznie automatyczny i
sprowadza się jedynie do kilku klików myszką:

![]({{< baseurl >}}/img/2016/09/30.router-archer-c2600-panel-admina-konfiguracja-pomysla.png#huge)

Sam panel administracyjny jest podzielony na dwie części. Jedna z nich skupia się na prostocie
konfiguracji, przez co nawet osoby, które nie znają się zbytnio na komputerach, powinny sobie
poradzić ze skonfigurowaniem routera Archer C2600:

![]({{< baseurl >}}/img/2016/09/31.router-archer-c2600-panel-admina-tryb-prosty.png#huge)

Dla bardziej wymagających są ustawienia zaawansowane, gdzie można dostosować sobie praktycznie każdy
aspekt pracy routera Archer C2600:

![]({{< baseurl >}}/img/2016/09/32.router-archer-c2600-panel-admina-tryb-zaawansowany.png#huge)

W tym panelu admina można skonfigurować naprawdę całą masę rzeczy i zamiast je tutaj wszystkie
wymieniać, to zachęcam do zapoznania się z tym panelem w [emulatorze jaki jest udostępniany przez
TP-LINK](http://static.tp-link.com/resources/simulator/C2600_Emulator/index.html).

## Kilka słów o firmware TP-LINK'a

Na tym oryginalnym firmware od TP-LINK, router Archer C2600 jest gotowy do pracy w jakieś 70-80
sekund od włączenia go. Trochę się wlecze zatem. Druga sprawa to zasięg. W porównaniu do mojego
routera Archer C7 v2, w paśmie 5 GHz nie ma praktycznie żadnej różnicy w kwestii zasięgu. Natomiast
jeśli chodzi o pasmo 2,4 GHz, to już tutaj widać dość znaczny boost, bo w granicach 10 dBm na
korzyść routera Archer C2600, zatem te zewnętrzne anteny jednak trochę pomagają.

W przypadku portów USB, Archer C2600 uzyskał ciągły transfer na poziomie 37-38 MiB/s (ok, 300
mbit/s), także całkiem nieźle. W końcu można podpiąć zewnętrzny dysk do routera, który może robić za
dość przyzwoity NAS. Problem tylko w systemie plików takich urządzeń. Wygląda na to, że nie jest
wspierany linux'owy system plików EXT4. Partycje dało radę zamontować, tylko gdy te miały system
plików FAT32 lub NTFS.

Problemem też jest logowanie się do panelu administracyjnego. Może samo logowanie jest jak
najbardziej w porządku ale tylko jeden host może być zalogowanym w panelu w danej chwili. Tę kwestię
jeszcze można by przemilczeć ale po utracie połączenia (np. wyciągnięciu wtyczki z portu ethernet),
system routera w dalszym ciągu traktuje taką maszynę jako podłączoną i zalogowaną jako admin. W
efekcie nie ma możliwość przez pewien czas wbić do panelu administracyjnego. Jak długo, tego
dokładnie nie wiem, prawdopodobnie 5-10 minut.

Męczący jest także ciągle ten prompt o zmianę hasła w panelu admina. Za każdym razem, gdy człowiek
się loguje do panelu z wykorzystaniem domyślnych danych logowania, ciągle mu jest prezentowana
informacja, by zmienić te dane i to jeszcze w formie okienka, które trzeba zamknąć.

## Wsparcie w OpenWRT/LEDE

Mnie oczywiście najbardziej interesuje czy ten router Archer C2600 jest wspierany w alternatywnym
firmware. Wygląda na to, że ten model [ma wsparcie (częściowe póki co) w
LEDE](https://forum.openwrt.org/viewtopic.php?pid=332184). Jest nawet stosowna [strona na wiki
OpenWRT](https://wiki.openwrt.org/toh/tp-link/tp-link_archer_c2600_v1). Obraz można zaś znaleźć na
[dl.eko.one.pl](http://dl.eko.one.pl/lede/17.01-SNAPSHOT/targets/ipq806x/generic/). Jako, że jest
obraz, to postanowiłem go wgrać na router i zobaczyć, czy aktualnie ten firmware jest zjadliwy.

![]({{< baseurl >}}/img/2016/09/33.router-archer-c2600-panel-admina-flash-openwrt-lede-firmware.png#big)

Router nawet działa:

![]({{< baseurl >}}/img/2016/09/34.router-archer-c2600-openwrt-lede-firmware.png#huge)

Niemniej jednak, są z nim problemy. Przede wszystkim, część przycisków nie działa. Działają tylko
reset i WPS. Nie świecą się też diody od WiFi, no i brakuje pomarańczowej diody sygnalizującej
status połączenia z internetem. Pozostałe rzeczy zdają się działać poprawnie. Da radę się połączyć
po WiFi, zarówno po 2,4 GHz jak i 5 GHz, a na dysku podpiętym do portu USB udało mi się wyciągnąć
transfer około 26 MiB/s przy zapisie. Także router jest jak najbardziej używalny. Poniżej znajduje
się wyjście szeregu poleceń

Log systemowy:

    root@lede:~# dmesg
    [    0.000000] Booting Linux on physical CPU 0x0
    [    0.000000] Linux version 4.4.19 (cezary@eko.one.pl) (gcc version 5.4.0 (LEDE GCC 5.4.0 r1491) ) #0 SMP PREEMPT Thu Sep 8 13:28:39 2016
    [    0.000000] CPU: ARMv7 Processor [512f04d0] revision 0 (ARMv7), cr=10c5787d
    [    0.000000] CPU: PIPT / VIPT nonaliasing data cache, PIPT instruction cache
    [    0.000000] Machine model: TP-Link Archer C2600
    [    0.000000] Memory policy: Data cache writealloc
    [    0.000000] On node 0 totalpages: 122880
    [    0.000000] free_area_init_node: node 0, pgdat c07f9cc0, node_mem_map ddc39000
    [    0.000000]   Normal zone: 960 pages used for memmap
    [    0.000000]   Normal zone: 0 pages reserved
    [    0.000000]   Normal zone: 122880 pages, LIFO batch:31
    [    0.000000] PERCPU: Embedded 11 pages/cpu @ddc10000 s14912 r8192 d21952 u45056
    [    0.000000] pcpu-alloc: s14912 r8192 d21952 u45056 alloc=11*4096
    [    0.000000] pcpu-alloc: [0] 0 [0] 1
    [    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 121920
    [    0.000000] Kernel command line:
    [    0.000000] Bootloader command line (ignored): console=ttyHSL1,115200n8 root=mtd:rootfs rootfstype=squashfs
    [    0.000000] PID hash table entries: 2048 (order: 1, 8192 bytes)
    [    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
    [    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
    [    0.000000] Memory: 480452K/491520K available (4249K kernel code, 246K rwdata, 1608K rodata, 204K init, 293K bss, 11068K reserved, 0K cma-reserved, 0K highmem)
    [    0.000000] Virtual kernel memory layout:
    [    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    [    0.000000]     fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
    [    0.000000]     vmalloc : 0xde800000 - 0xff800000   ( 528 MB)
    [    0.000000]     lowmem  : 0xc0000000 - 0xde000000   ( 480 MB)
    [    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
    [    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
    [    0.000000]       .text : 0xc0208000 - 0xc07c0978   (5859 kB)
    [    0.000000]       .init : 0xc07c1000 - 0xc07f4000   ( 204 kB)
    [    0.000000]       .data : 0xc07f4000 - 0xc0831800   ( 246 kB)
    [    0.000000]        .bss : 0xc0834000 - 0xc087d504   ( 294 kB)
    [    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
    [    0.000000] Preemptible hierarchical RCU implementation.
    [    0.000000]  RCU restricting CPUs from NR_CPUS=4 to nr_cpu_ids=2.
    [    0.000000] RCU: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=2
    [    0.000000] NR_IRQS:16 nr_irqs:16 16
    [    0.000000] clocksource: dg_timer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 305801671480 ns
    [    0.000007] sched_clock: 32 bits at 6MHz, resolution 160ns, wraps every 343597383600ns
    [    0.000022] Switching to timer-based delay loop, resolution 160ns
    [    0.000202] Calibrating delay loop (skipped), value calculated using timer frequency.. 12.50 BogoMIPS (lpj=62500)
    [    0.000227] pid_max: default: 32768 minimum: 301
    [    0.000360] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.000380] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.001069] CPU: Testing write buffer coherency: ok
    [    0.001391] CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
    [    0.001496] Setting up static identity map for 0x42208280 - 0x422082d8
    [    0.094469] CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
    [    0.094631] Brought up 2 CPUs
    [    0.094652] SMP: Total of 2 processors activated (25.00 BogoMIPS).
    [    0.094665] CPU: All CPU(s) started in SVC mode.
    [    0.104943] VFP support v0.3: implementor 51 architecture 64 part 4d variant 2 rev 0
    [    0.105402] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
    [    0.105567] pinctrl core: initialized pinctrl subsystem
    [    0.107180] NET: Registered protocol family 16
    [    0.107458] DMA: preallocated 256 KiB pool for atomic coherent allocations
    [    0.134930] cpuidle: using governor ladder
    [    0.165946] cpuidle: using governor menu
    [    0.179884] gpiochip_add: registered GPIOs 0 to 68 on device: 800000.pinmux
    [    0.179911] GPIO chip 800000.pinmux: created GPIO range 0->68 ==> 800000.pinmux PIN 0->68
    [    0.181543] qcom_rpm 108000.rpm: RPM firmware 3.0.16777342
    [    0.230169] pps_core: LinuxPPS API ver. 1 registered
    [    0.230189] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
    [    0.230237] PTP clock support registered
    [    0.231517] clocksource: Switched to clocksource dg_timer
    [    0.233501] NET: Registered protocol family 2
    [    0.234394] TCP established hash table entries: 4096 (order: 2, 16384 bytes)
    [    0.234440] TCP bind hash table entries: 4096 (order: 3, 32768 bytes)
    [    0.234497] TCP: Hash tables configured (established 4096 bind 4096)
    [    0.234555] UDP hash table entries: 256 (order: 1, 8192 bytes)
    [    0.234583] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
    [    0.234828] NET: Registered protocol family 1
    [    0.234911] PCI: CLS 0 bytes, default 64
    [    0.236746] futex hash table entries: 512 (order: 3, 32768 bytes)
    [    0.236888] No memory allocated for crashlog
    [    0.245925] squashfs: version 4.0 (2009/01/31) Phillip Lougher
    [    0.245954] jffs2: version 2.2 (NAND) (SUMMARY) (LZMA) (RTIME) (CMODE_PRIORITY) (c) 2001-2006 Red Hat, Inc.
    [    0.249170] io scheduler noop registered
    [    0.249201] io scheduler deadline registered (default)
    [    0.250607] qcom-pcie 1b500000.pci: GPIO lookup for consumer perst
    [    0.250631] qcom-pcie 1b500000.pci: using device tree for GPIO lookup
    [    0.250653] of_get_named_gpiod_flags: can't parse 'perst-gpios' property of node '/soc/pci@1b500000[0]'
    [    0.250683] of_get_named_gpiod_flags: parsed 'perst-gpio' property of node '/soc/pci@1b500000[0]' - status (0)
    [    0.250794] 1b500000.pci supply vdda not found, using dummy regulator
    [    0.250912] 1b500000.pci supply vdda_phy not found, using dummy regulator
    [    0.251008] 1b500000.pci supply vdda_refclk not found, using dummy regulator
    [    0.251503] PCI host bridge /soc/pci@1b500000 ranges:
    [    0.251547]    IO 0x0fe00000..0x0fefffff -> 0x0fe00000
    [    0.251574]   MEM 0x08000000..0x0fdfffff -> 0x08000000
    [    0.281066] qcom-pcie 1b500000.pci: PCI host bridge to bus 0000:00
    [    0.281095] pci_bus 0000:00: root bus resource [bus 00-ff]
    [    0.281119] pci_bus 0000:00: root bus resource [io  0x0000-0xfffff] (bus address [0xfe00000-0xfefffff])
    [    0.281137] pci_bus 0000:00: root bus resource [mem 0x08000000-0x0fdfffff]
    [    0.281156] pci_bus 0000:00: scanning bus
    [    0.281210] pci 0000:00:00.0: [17cb:0101] type 01 class 0x060400
    [    0.281340] pci 0000:00:00.0: calling pci_fixup_ide_bases+0x0/0x44
    [    0.281418] pci 0000:00:00.0: supports D1
    [    0.281435] pci 0000:00:00.0: PME# supported from D0 D1 D3hot
    [    0.281459] pci 0000:00:00.0: PME# disabled
    [    0.282109] pci_bus 0000:00: fixups for bus
    [    0.282140] PCI: bus0: Fast back to back transfers disabled
    [    0.282163] pci 0000:00:00.0: scanning [bus 01-01] behind bridge, pass 0
    [    0.282390] pci_bus 0000:01: scanning bus
    [    0.282554] pci 0000:01:00.0: [168c:0040] type 00 class 0x028000
    [    0.282913] pci 0000:01:00.0: reg 0x10: [mem 0x00000000-0x001fffff 64bit]
    [    0.283331] pci 0000:01:00.0: calling pci_fixup_ide_bases+0x0/0x44
    [    0.283768] pci 0000:01:00.0: PME# supported from D0 D3hot D3cold
    [    0.283812] pci 0000:01:00.0: PME# disabled
    [    0.284266] pci_bus 0000:01: fixups for bus
    [    0.284336] PCI: bus1: Fast back to back transfers disabled
    [    0.284353] pci_bus 0000:01: bus scan returning with max=01
    [    0.284378] pci 0000:00:00.0: scanning [bus 01-01] behind bridge, pass 1
    [    0.284401] pci_bus 0000:00: bus scan returning with max=01
    [    0.284486] pci 0000:00:00.0: fixup irq: got 132
    [    0.284502] pci 0000:00:00.0: assigning IRQ 132
    [    0.284572] pci 0000:01:00.0: fixup irq: got 132
    [    0.284587] pci 0000:01:00.0: assigning IRQ 132
    [    0.284660] pci 0000:00:00.0: BAR 8: assigned [mem 0x08000000-0x081fffff]
    [    0.284687] pci 0000:01:00.0: BAR 0: assigned [mem 0x08000000-0x081fffff 64bit]
    [    0.284808] pci 0000:00:00.0: PCI bridge to [bus 01]
    [    0.284834] pci 0000:00:00.0:   bridge window [mem 0x08000000-0x081fffff]
    [    0.285264] pcieport 0000:00:00.0: Signaling PME through PCIe PME interrupt
    [    0.285283] pci 0000:01:00.0: Signaling PME through PCIe PME interrupt
    [    0.285307] pcie_pme 0000:00:00.0:pcie01: service driver pcie_pme loaded
    [    0.285530] aer 0000:00:00.0:pcie02: service driver aer loaded
    [    0.285854] qcom-pcie 1b700000.pci: GPIO lookup for consumer perst
    [    0.285874] qcom-pcie 1b700000.pci: using device tree for GPIO lookup
    [    0.285893] of_get_named_gpiod_flags: can't parse 'perst-gpios' property of node '/soc/pci@1b700000[0]'
    [    0.285918] of_get_named_gpiod_flags: parsed 'perst-gpio' property of node '/soc/pci@1b700000[0]' - status (0)
    [    0.286006] 1b700000.pci supply vdda not found, using dummy regulator
    [    0.286115] 1b700000.pci supply vdda_phy not found, using dummy regulator
    [    0.286222] 1b700000.pci supply vdda_refclk not found, using dummy regulator
    [    0.286691] PCI host bridge /soc/pci@1b700000 ranges:
    [    0.286731]    IO 0x31e00000..0x31efffff -> 0x31e00000
    [    0.286755]   MEM 0x2e000000..0x31dfffff -> 0x2e000000
    [    0.319729] qcom-pcie 1b700000.pci: PCI host bridge to bus 0001:00
    [    0.319753] pci_bus 0001:00: root bus resource [bus 00-ff]
    [    0.319772] pci_bus 0001:00: root bus resource [mem 0x2e000000-0x31dfffff]
    [    0.319787] pci_bus 0001:00: scanning bus
    [    0.319834] pci 0001:00:00.0: [17cb:0101] type 01 class 0x060400
    [    0.319943] pci 0001:00:00.0: calling pci_fixup_ide_bases+0x0/0x44
    [    0.320008] pci 0001:00:00.0: supports D1
    [    0.320024] pci 0001:00:00.0: PME# supported from D0 D1 D3hot
    [    0.320044] pci 0001:00:00.0: PME# disabled
    [    0.320421] pci_bus 0001:00: fixups for bus
    [    0.320447] PCI: bus0: Fast back to back transfers disabled
    [    0.320466] pci 0001:00:00.0: scanning [bus 01-01] behind bridge, pass 0
    [    0.320692] pci_bus 0001:01: scanning bus
    [    0.320856] pci 0001:01:00.0: [168c:0040] type 00 class 0x028000
    [    0.321208] pci 0001:01:00.0: reg 0x10: [mem 0x00000000-0x001fffff 64bit]
    [    0.321622] pci 0001:01:00.0: calling pci_fixup_ide_bases+0x0/0x44
    [    0.322148] pci 0001:01:00.0: PME# supported from D0 D3hot D3cold
    [    0.322193] pci 0001:01:00.0: PME# disabled
    [    0.322646] pci_bus 0001:01: fixups for bus
    [    0.322716] PCI: bus1: Fast back to back transfers disabled
    [    0.322731] pci_bus 0001:01: bus scan returning with max=01
    [    0.322754] pci 0001:00:00.0: scanning [bus 01-01] behind bridge, pass 1
    [    0.322779] pci_bus 0001:00: bus scan returning with max=01
    [    0.322827] pcieport 0000:00:00.0: fixup irq: got 132
    [    0.322843] pcieport 0000:00:00.0: assigning IRQ 132
    [    0.322911] pci 0000:01:00.0: fixup irq: got 132
    [    0.322928] pci 0000:01:00.0: assigning IRQ 132
    [    0.323005] pci 0001:00:00.0: fixup irq: got 165
    [    0.323020] pci 0001:00:00.0: assigning IRQ 165
    [    0.323089] pci 0001:01:00.0: fixup irq: got 165
    [    0.323104] pci 0001:01:00.0: assigning IRQ 165
    [    0.323165] pci 0001:00:00.0: BAR 8: assigned [mem 0x2e000000-0x2e1fffff]
    [    0.323191] pci 0001:01:00.0: BAR 0: assigned [mem 0x2e000000-0x2e1fffff 64bit]
    [    0.323305] pci 0001:00:00.0: PCI bridge to [bus 01]
    [    0.323328] pci 0001:00:00.0:   bridge window [mem 0x2e000000-0x2e1fffff]
    [    0.323717] pcieport 0001:00:00.0: Signaling PME through PCIe PME interrupt
    [    0.323734] pci 0001:01:00.0: Signaling PME through PCIe PME interrupt
    [    0.323757] pcie_pme 0001:00:00.0:pcie01: service driver pcie_pme loaded
    [    0.323987] aer 0001:00:00.0:pcie02: service driver aer loaded
    [    0.327429] gsbi 16300000.gsbi: GSBI port protocol: 6 crci: 0
    [    0.328759] gsbi 1a200000.gsbi: GSBI port protocol: 3 crci: 0
    [    0.329846] Serial: 8250/16550 driver, 16 ports, IRQ sharing enabled
    [    0.335144] msm_serial 16340000.serial: msm_serial: detected port #0
    [    0.335269] msm_serial 16340000.serial: uartclk = 1843200
    [    0.335347] 16340000.serial: ttyMSM0 at MMIO 0x16340000 (irq = 166, base_baud = 115200) is a MSM
    [    0.335394] msm_serial: console setup on port #0
    [    1.016609] console [ttyMSM0] enabled
    [    1.021429] msm_serial: driver initialized
    [    1.026511] spi_qup 1a280000.spi: IN:block:16, fifo:64, OUT:block:16, fifo:64
    [    1.028716] of_get_named_gpiod_flags: parsed 'cs-gpios' property of node '/soc/gsbi@1a200000/spi@1a280000[0]' - status (0)
    [    1.030133] m25p80 spi32766.0: s25fl256s1 (32768 Kbytes)
    [    1.036185] 25 ofpart partitions found on MTD device spi32766.0
    [    1.041199] Creating 25 MTD partitions on "spi32766.0":
    [    1.046932] 0x000000000000-0x000000020000 : "SBL1"
    [    1.064377] 0x000000020000-0x000000040000 : "MIBIB"
    [    1.076233] 0x000000040000-0x000000060000 : "SBL2"
    [    1.087986] 0x000000060000-0x000000090000 : "SBL3"
    [    1.099759] 0x000000090000-0x0000000a0000 : "DDRCONFIG"
    [    1.111551] 0x0000000a0000-0x0000000b0000 : "SSD"
    [    1.123424] 0x0000000b0000-0x0000000e0000 : "TZ"
    [    1.135323] 0x0000000e0000-0x000000100000 : "RPM"
    [    1.147150] 0x000000100000-0x000000170000 : "fs-uboot"
    [    1.158977] 0x000000170000-0x0000001b0000 : "uboot-env"
    [    1.170784] 0x0000001b0000-0x0000001f0000 : "radio"
    [    1.182605] 0x0000001f0000-0x0000003f0000 : "os-image"
    [    1.194394] 0x0000003f0000-0x000001ef0000 : "rootfs"
    [    1.206170] mtd: device 12 (rootfs) set to be root filesystem
    [    1.206466] 1 squashfs-split partitions found on MTD device rootfs
    [    1.210900] 0x0000006d0000-0x000001ef0000 : "rootfs_data"
    [    1.228957] 0x000001ef0000-0x000001ef0200 : "default-mac"
    [    1.240853] 0x000001ef0200-0x000001ef0400 : "pin"
    [    1.252664] 0x000001ef0400-0x000001f00000 : "product-info"
    [    1.264481] 0x000001f00000-0x000001f10000 : "partition-table"
    [    1.276305] 0x000001f10000-0x000001f20000 : "soft-version"
    [    1.277846] 0x000001f20000-0x000001f30000 : "support-list"
    [    1.282274] 0x000001f30000-0x000001f40000 : "profile"
    [    1.287669] 0x000001f40000-0x000001f50000 : "default-config"
    [    1.292877] 0x000001f50000-0x000001f90000 : "user-config"
    [    1.308894] 0x000001f90000-0x000001fd0000 : "qos-db"
    [    1.320733] 0x000001fd0000-0x000001fe0000 : "usb-config"
    [    1.332537] 0x000001fe0000-0x000002000000 : "log"
    [    1.334887] libphy: Fixed MDIO Bus: probed
    [    1.336538] of_get_named_gpiod_flags: parsed 'gpios' property of node '/soc/mdio[0]' - status (0)
    [    1.336566] of_get_named_gpiod_flags: parsed 'gpios' property of node '/soc/mdio[1]' - status (0)
    [    1.336587] of_get_named_gpiod_flags: can't parse 'gpios' property of node '/soc/mdio[2]'
    [    1.336788] libphy: GPIO Bitbanged MDIO: probed
    [    1.365116] switch0: Atheros AR8337 rev. 2 switch registered on gpio-0
    [    1.633168] of_get_named_gpiod_flags: can't parse 'link-gpios' property of node '/soc/ethernet@37200000/fixed-link[0]'
    [    1.633954] stmmac - user ID: 0x10, Synopsys ID: 0x37
    [    1.633986]  Ring mode enabled
    [    1.637982]  DMA HW capability register supported
    [    1.640938]  Enhanced/Alternate descriptors
    [    1.645755]  Enabled extended descriptors
    [    1.649704]  RX Checksum Offload Engine supported (type 2)
    [    1.654007]  TX Checksum insertion supported
    [    1.659251]  Wake-Up On Lan supported
    [    1.663894]  Enable RX Mitigation via HW Watchdog Timer
    [    1.668464] of_get_named_gpiod_flags: can't parse 'link-gpios' property of node '/soc/ethernet@37400000/fixed-link[0]'
    [    1.669146] stmmac - user ID: 0x10, Synopsys ID: 0x37
    [    1.672432]  Ring mode enabled
    [    1.677476]  DMA HW capability register supported
    [    1.680432]  Enhanced/Alternate descriptors
    [    1.685240]  Enabled extended descriptors
    [    1.689198]  RX Checksum Offload Engine supported (type 2)
    [    1.693515]  TX Checksum insertion supported
    [    1.698747]  Wake-Up On Lan supported
    [    1.703361]  Enable RX Mitigation via HW Watchdog Timer
    [    1.707540] i2c /dev entries driver
    [    1.713240] Speed bin: 0
    [    1.715237] PVS bin: 1
    [    1.720536] L2 @ QSB rate. Forcing new rate.
    [    1.720694] L2 @ 384000 KHz
    [    1.724921] CPU0 @ 800000 KHz
    [    1.727132] CPU1 @ QSB rate. Forcing new rate.
    [    1.730340] CPU1 @ 384000 KHz
    [    1.736264] NET: Registered protocol family 10
    [    1.739380] NET: Registered protocol family 17
    [    1.742216] bridge: automatic filtering via arp/ip/ip6tables has been deprecated. Update your scripts to load br_netfilter if you need this.
    [    1.746739] 8021q: 802.1Q VLAN Support v1.8
    [    1.759347] Registering SWP/SWPB emulation handler
    [    1.770942] hctosys: unable to open rtc device (rtc0)
    [    1.787608] VFS: Mounted root (squashfs filesystem) readonly on device 31:12.
    [    1.787796] Freeing unused kernel memory: 204K (c07c1000 - c07f4000)
    [    1.963474] random: nonblocking pool is initialized
    [    3.003566] init: Console is alive
    [    3.003713] init: - watchdog -
    [    4.880544] usbcore: registered new interface driver usbfs
    [    4.880622] usbcore: registered new interface driver hub
    [    4.885141] usbcore: registered new device driver usb
    [    4.897743] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-keys/wifi[0]' - status (0)
    [    4.897767] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-keys/reset[0]' - status (0)
    [    4.897783] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-keys/wps[0]' - status (0)
    [    4.897798] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-keys/ledgeneral[0]' - status (0)
    [    4.901263] SCSI subsystem initialized
    [    4.903241] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
    [    4.904329] ehci-platform: EHCI generic platform driver
    [    4.912712] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
    [    4.915988] ohci-platform: OHCI generic platform driver
    [    5.408478] xhci-hcd xhci-hcd.0.auto: xHCI Host Controller
    [    5.408527] xhci-hcd xhci-hcd.0.auto: new USB bus registered, assigned bus number 1
    [    5.413173] xhci-hcd xhci-hcd.0.auto: hcc params 0x0228f065 hci version 0x100 quirks 0x00010010
    [    5.420561] xhci-hcd xhci-hcd.0.auto: irq 168, io mem 0x11000000
    [    5.429849] hub 1-0:1.0: USB hub found
    [    5.435508] hub 1-0:1.0: 1 port detected
    [    5.439252] xhci-hcd xhci-hcd.0.auto: xHCI Host Controller
    [    5.443074] xhci-hcd xhci-hcd.0.auto: new USB bus registered, assigned bus number 2
    [    5.448354] usb usb2: We don't know the algorithms for LPM for this host, disabling LPM.
    [    5.456447] hub 2-0:1.0: USB hub found
    [    5.464248] hub 2-0:1.0: 1 port detected
    [    5.468103] xhci-hcd xhci-hcd.1.auto: xHCI Host Controller
    [    5.471887] xhci-hcd xhci-hcd.1.auto: new USB bus registered, assigned bus number 3
    [    5.477256] xhci-hcd xhci-hcd.1.auto: hcc params 0x0228f065 hci version 0x100 quirks 0x00010010
    [    5.484746] xhci-hcd xhci-hcd.1.auto: irq 169, io mem 0x10000000
    [    5.493934] hub 3-0:1.0: USB hub found
    [    5.499593] hub 3-0:1.0: 1 port detected
    [    5.503527] xhci-hcd xhci-hcd.1.auto: xHCI Host Controller
    [    5.507218] xhci-hcd xhci-hcd.1.auto: new USB bus registered, assigned bus number 4
    [    5.512668] usb usb4: We don't know the algorithms for LPM for this host, disabling LPM.
    [    5.520596] hub 4-0:1.0: USB hub found
    [    5.528475] hub 4-0:1.0: 1 port detected
    [    5.533103] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/lan[0]' - status (0)
    [    5.533226] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/usb4[0]' - status (0)
    [    5.533329] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/usb2[0]' - status (0)
    [    5.533422] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/wps[0]' - status (0)
    [    5.533510] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/wan_blue[0]' - status (0)
    [    5.533603] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/status[0]' - status (0)
    [    5.533701] of_get_named_gpiod_flags: parsed 'gpios' property of node '/gpio-leds/ledgnr[0]' - status (0)
    [    5.535133] usbcore: registered new interface driver usb-storage
    [    5.541315] init: - preinit -
    [    5.888513] usb 3-1: new high-speed USB device number 2 using xhci-hcd
    [    6.024820] usb-storage 3-1:1.0: USB Mass Storage device detected
    [    6.035320] scsi host0: usb-storage 3-1:1.0
    [    7.045942] scsi 0:0:0:0: Direct-Access     WDC WD15 EARS-00MVWB0     AB51 PQ: 0 ANSI: 2 CCS
    [    7.047336] sd 0:0:0:0: [sda] 2930275055 512-byte logical blocks: (1.50 TB/1.36 TiB)
    [    7.054105] sd 0:0:0:0: [sda] Write Protect is off
    [    7.061171] sd 0:0:0:0: [sda] Mode Sense: 00 38 00 00
    [    7.061699] sd 0:0:0:0: [sda] Asking for cache data failed
    [    7.065872] sd 0:0:0:0: [sda] Assuming drive cache: write through
    [    7.089722]  sda: sda1 sda2 sda3 sda4
    [    7.093002] sd 0:0:0:0: [sda] Attached SCSI disk
    [    8.392119] ipq806x-gmac-dwmac 37400000.ethernet eth1: Link is Up - 1Gbps/Full - flow control off
    [    9.528118] mount_root: loading kmods from internal overlay
    [   11.803122] jffs2: notice: (145) jffs2_build_xattr_subsystem: complete building xattr subsystem, 0 of xdatum (0 unchecked, 0 orphan) and 0 of xref (0 dead, 0 orphan) found.
    [   11.807111] block: attempting to load /tmp/jffs_cfg/upper/etc/config/fstab
    [   11.831647] block: extroot: not configured
    [   12.602397] jffs2: notice: (142) jffs2_build_xattr_subsystem: complete building xattr subsystem, 0 of xdatum (0 unchecked, 0 orphan) and 0 of xref (0 dead, 0 orphan) found.
    [   12.608037] mount_root: loading kmods from internal overlay
    [   15.020562] block: attempting to load /tmp/jffs_cfg/upper/etc/config/fstab
    [   15.025038] block: extroot: not configured
    [   15.052977] mount_root: switching to jffs2 overlay
    [   15.065987] urandom-seed: Seeding with /etc/urandom.seed
    [   15.139207] procd: - early -
    [   15.139279] procd: - watchdog -
    [   15.753728] procd: - ubus -
    [   15.805367] procd: - init -
    [   15.949330] ip6_tables: (C) 2000-2006 Netfilter Core Team
    [   15.954508] Loading modules backported from Linux version wt-2016-06-20-0-gbc17424
    [   15.954542] Backport generated by backports.git backports-20160216-7-g5735958
    [   15.993204] ath10k_pci 0000:01:00.0: enabling device (0140 -> 0142)
    [   15.993295] ath10k_pci 0000:01:00.0: enabling bus mastering
    [   15.993769] ath10k_pci 0000:01:00.0: pci irq msi oper_irq_mode 2 irq_mode 0 reset_mode 0
    [   16.126945] ath10k_pci 0000:01:00.0: Direct firmware load for ath10k/pre-cal-pci-0000:01:00.0.bin failed with error -2
    [   16.126983] ath10k_pci 0000:01:00.0: Falling back to user helper
    [   17.155370] firmware ath10k!pre-cal-pci-0000:01:00.0.bin: firmware_loading_store: map pages failed
    [   17.763291] ath10k_pci 0000:01:00.0: qca99x0 hw2.0 target 0x01000000 chip_id 0x003b01ff sub 168c:0002
    [   17.763328] ath10k_pci 0000:01:00.0: kconfig debug 0 debugfs 1 tracing 0 dfs 1 testmode 1
    [   17.773068] ath10k_pci 0000:01:00.0: firmware ver 10.4.1.00030-1 api 5 features no-p2p crc32 d2901e01
    [   17.779951] ath10k_pci 0000:01:00.0: failed to fetch board data for bus=pci,vendor=168c,device=0040,subsystem-vendor=168c,subsystem-device=0002 from ath10k/QCA99X0/hw2.0/board-2.bin
    [   17.789272] ath10k_pci 0000:01:00.0: board_file api 1 bmi_id N/A crc32 7e56fd07
    [   18.973235] ath10k_pci 0000:01:00.0: htt-ver 2.2 wmi-op 6 htt-op 4 cal file max-sta 512 raw 0 hwcrypto 1
    [   19.035116] ath: EEPROM regdomain: 0x0
    [   19.035125] ath: EEPROM indicates default country code should be used
    [   19.035132] ath: doing EEPROM country->regdmn map search
    [   19.035141] ath: country maps to regdmn code: 0x3a
    [   19.035148] ath: Country alpha2 being used: US
    [   19.035154] ath: Regpair used: 0x3a
    [   19.050584] ath10k_pci 0001:01:00.0: enabling device (0140 -> 0142)
    [   19.050688] ath10k_pci 0001:01:00.0: enabling bus mastering
    [   19.051170] ath10k_pci 0001:01:00.0: pci irq msi oper_irq_mode 2 irq_mode 0 reset_mode 0
    [   19.183351] ath10k_pci 0001:01:00.0: Direct firmware load for ath10k/pre-cal-pci-0001:01:00.0.bin failed with error -2
    [   19.183388] ath10k_pci 0001:01:00.0: Falling back to user helper
    [   19.208950] firmware ath10k!pre-cal-pci-0001:01:00.0.bin: firmware_loading_store: map pages failed
    [   19.213513] ath10k_pci 0001:01:00.0: qca99x0 hw2.0 target 0x01000000 chip_id 0x003b01ff sub 168c:0002
    [   19.216807] ath10k_pci 0001:01:00.0: kconfig debug 0 debugfs 1 tracing 0 dfs 1 testmode 1
    [   19.227954] ath10k_pci 0001:01:00.0: firmware ver 10.4.1.00030-1 api 5 features no-p2p crc32 d2901e01
    [   19.234507] ath10k_pci 0001:01:00.0: failed to fetch board data for bus=pci,vendor=168c,device=0040,subsystem-vendor=168c,subsystem-device=0002 from ath10k/QCA99X0/hw2.0/board-2.bin
    [   19.243684] ath10k_pci 0001:01:00.0: board_file api 1 bmi_id N/A crc32 7e56fd07
    [   20.428647] ath10k_pci 0001:01:00.0: htt-ver 2.2 wmi-op 6 htt-op 4 cal file max-sta 512 raw 0 hwcrypto 1
    [   20.485638] ath: EEPROM regdomain: 0x0
    [   20.485646] ath: EEPROM indicates default country code should be used
    [   20.485652] ath: doing EEPROM country->regdmn map search
    [   20.485661] ath: country maps to regdmn code: 0x3a
    [   20.485668] ath: Country alpha2 being used: US
    [   20.485675] ath: Regpair used: 0x3a
    [   20.492108] ip_tables: (C) 2000-2006 Netfilter Core Team
    [   20.495981] nf_conntrack version 0.5.0 (7510 buckets, 30040 max)
    [   20.598928] xt_time: kernel timezone is -0000
    [   20.602620] PPP generic driver version 2.4.2
    [   20.603266] NET: Registered protocol family 24
    [   23.245823] device eth1 entered promiscuous mode
    [   23.248538] IPv6: ADDRCONF(NETDEV_UP): br-lan: link is not ready
    [   25.221939] ipq806x-gmac-dwmac 37400000.ethernet eth1: Link is Up - 1Gbps/Full - flow control off
    [   25.251927] ipq806x-gmac-dwmac 37200000.ethernet eth0: Link is Up - 1Gbps/Full - flow control off
    [   26.492501] IPv6: ADDRCONF(NETDEV_UP): wlan0: link is not ready
    [   26.492574] br-lan: port 1(eth1) entered forwarding state
    [   26.497257] br-lan: port 1(eth1) entered forwarding state
    [   26.510429] device wlan0 entered promiscuous mode
    [   26.511224] IPv6: ADDRCONF(NETDEV_CHANGE): br-lan: link becomes ready
    [   27.975188] IPv6: ADDRCONF(NETDEV_UP): wlan1: link is not ready
    [   27.993478] device wlan1 entered promiscuous mode
    [   28.125087] ath10k_pci 0001:01:00.0: no channel configured; ignoring frame(s)!
    [   28.126407] ath10k_pci 0001:01:00.0: no channel configured; ignoring frame(s)!
    [   28.131234] ath10k_pci 0001:01:00.0: no channel configured; ignoring frame(s)!
    [   28.138546] ath10k_pci 0001:01:00.0: no channel configured; ignoring frame(s)!
    [   28.364942] ath10k_pci 0001:01:00.0: no channel configured; ignoring frame(s)!
    [   28.492129] br-lan: port 1(eth1) entered forwarding state
    [   28.816855] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
    [   28.816991] br-lan: port 2(wlan0) entered forwarding state
    [   28.822373] br-lan: port 2(wlan0) entered forwarding state
    [   28.827740] IPv6: ADDRCONF(NETDEV_CHANGE): wlan1: link becomes ready
    [   28.833253] br-lan: port 3(wlan1) entered forwarding state
    [   28.839590] br-lan: port 3(wlan1) entered forwarding state
    [   30.825418] br-lan: port 2(wlan0) entered forwarding state
    [   30.831817] br-lan: port 3(wlan1) entered forwarding state

Informacje o procesorze:

    # cat /proc/cpuinfo
    processor       : 0
    model name      : ARMv7 Processor rev 0 (v7l)
    BogoMIPS        : 21.87
    Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32
    CPU implementer : 0x51
    CPU architecture: 7
    CPU variant     : 0x2
    CPU part        : 0x04d
    CPU revision    : 0

    processor       : 1
    model name      : ARMv7 Processor rev 0 (v7l)
    BogoMIPS        : 45.57
    Features        : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32
    CPU implementer : 0x51
    CPU architecture: 7
    CPU variant     : 0x2
    CPU part        : 0x04d
    CPU revision    : 0

    Hardware        : Qualcomm (Flattened Device Tree)
    Revision        : 0000
    Serial          : 0000000000000000

Porty USB:

    root@lede:~# cat /sys/kernel/debug/usb/devices

    T:  Bus=04 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=5000 MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 3.00 Cls=09(hub  ) Sub=00 Prot=03 MxPS= 9 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0003 Rev= 4.04
    S:  Manufacturer=Linux 4.4.19 xhci-hcd
    S:  Product=xHCI Host Controller
    S:  SerialNumber=xhci-hcd.1.auto
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

    T:  Bus=03 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=480  MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 2.00 Cls=09(hub  ) Sub=00 Prot=01 MxPS=64 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0002 Rev= 4.04
    S:  Manufacturer=Linux 4.4.19 xhci-hcd
    S:  Product=xHCI Host Controller
    S:  SerialNumber=xhci-hcd.1.auto
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

    T:  Bus=02 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=5000 MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 3.00 Cls=09(hub  ) Sub=00 Prot=03 MxPS= 9 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0003 Rev= 4.04
    S:  Manufacturer=Linux 4.4.19 xhci-hcd
    S:  Product=xHCI Host Controller
    S:  SerialNumber=xhci-hcd.0.auto
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

    T:  Bus=01 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=480  MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 2.00 Cls=09(hub  ) Sub=00 Prot=01 MxPS=64 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0002 Rev= 4.04
    S:  Manufacturer=Linux 4.4.19 xhci-hcd
    S:  Product=xHCI Host Controller
    S:  SerialNumber=xhci-hcd.0.auto
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

Informacje o WiFi:

    root@lede:~# iw list
    Wiphy phy1
            max # scan SSIDs: 16
            max scan IEs length: 209 bytes
            max # sched scan SSIDs: 0
            max # match sets: 0
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports AP-side u-APSD.
            Available Antennas: TX 0xf RX 0xf
            Configured Antennas: TX 0xf RX 0xf
            Supported interface modes:
                     * managed
                     * AP
                     * AP/VLAN
                     * monitor
                     * mesh point
            Band 1:
                    Capabilities: 0x19ef
                            RX LDPC
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            TX STBC
                            RX STBC 1-stream
                            Max AMSDU length: 7935 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-31
                    Frequencies:
                            * 2412 MHz [1] (20.0 dBm)
                            * 2417 MHz [2] (20.0 dBm)
                            * 2422 MHz [3] (20.0 dBm)
                            * 2427 MHz [4] (20.0 dBm)
                            * 2432 MHz [5] (20.0 dBm)
                            * 2437 MHz [6] (20.0 dBm)
                            * 2442 MHz [7] (20.0 dBm)
                            * 2447 MHz [8] (20.0 dBm)
                            * 2452 MHz [9] (20.0 dBm)
                            * 2457 MHz [10] (20.0 dBm)
                            * 2462 MHz [11] (20.0 dBm)
                            * 2467 MHz [12] (20.0 dBm)
                            * 2472 MHz [13] (20.0 dBm)
                            * 2484 MHz [14] (disabled)
            valid interface combinations:
                     * #{ managed } <= 1, #{ AP, mesh point } <= 16,
                       total <= 16, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz, 80 MHz }

            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing
            Device supports VHT-IBSS.

    Wiphy phy0
            max # scan SSIDs: 16
            max scan IEs length: 199 bytes
            max # sched scan SSIDs: 0
            max # match sets: 0
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports AP-side u-APSD.
            Available Antennas: TX 0xf RX 0xf
            Configured Antennas: TX 0xf RX 0xf
            Supported interface modes:
                     * managed
                     * AP
                     * AP/VLAN
                     * monitor
                     * mesh point
            Band 2:
                    Capabilities: 0x19ef
                            RX LDPC
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            TX STBC
                            RX STBC 1-stream
                            Max AMSDU length: 7935 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-31
                    VHT Capabilities (0x339b79b2):
                            Max MPDU length: 11454
                            Supported Channel Width: neither 160 nor 80+80
                            RX LDPC
                            short GI (80 MHz)
                            TX STBC
                            SU Beamformer
                            SU Beamformee
                            MU Beamformer
                            MU Beamformee
                            RX antenna pattern consistency
                            TX antenna pattern consistency
                    VHT RX MCS set:
                            1 streams: MCS 0-9
                            2 streams: MCS 0-9
                            3 streams: MCS 0-9
                            4 streams: MCS 0-9
                            5 streams: not supported
                            6 streams: not supported
                            7 streams: not supported
                            8 streams: not supported
                    VHT RX highest supported: 0 Mbps
                    VHT TX MCS set:
                            1 streams: MCS 0-9
                            2 streams: MCS 0-9
                            3 streams: MCS 0-9
                            4 streams: MCS 0-9
                            5 streams: not supported
                            6 streams: not supported
                            7 streams: not supported
                            8 streams: not supported
                    VHT TX highest supported: 0 Mbps
                    Frequencies:
                            * 5180 MHz [36] (20.0 dBm)
                            * 5200 MHz [40] (20.0 dBm)
                            * 5220 MHz [44] (20.0 dBm)
                            * 5240 MHz [48] (20.0 dBm)
                            * 5260 MHz [52] (20.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5280 MHz [56] (20.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5300 MHz [60] (20.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5320 MHz [64] (20.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5500 MHz [100] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5520 MHz [104] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5540 MHz [108] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5560 MHz [112] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5580 MHz [116] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5600 MHz [120] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5620 MHz [124] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5640 MHz [128] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5660 MHz [132] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5680 MHz [136] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5700 MHz [140] (27.0 dBm) (radar detection)
                              DFS state: usable (for 438 sec)
                              DFS CAC time: 60000 ms
                            * 5720 MHz [144] (disabled)
                            * 5745 MHz [149] (disabled)
                            * 5765 MHz [153] (disabled)
                            * 5785 MHz [157] (disabled)
                            * 5805 MHz [161] (disabled)
                            * 5825 MHz [165] (disabled)
            valid interface combinations:
                     * #{ managed } <= 1, #{ AP, mesh point } <= 16,
                       total <= 16, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz, 80 MHz }

            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing
            Device supports VHT-IBSS.
