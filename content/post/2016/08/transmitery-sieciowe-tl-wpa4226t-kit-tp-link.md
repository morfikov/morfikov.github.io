---
author: Morfik
categories:
- Hardware
date: "2016-08-29T18:01:22Z"
date_gmt: 2016-08-29 16:01:22 +0200
published: true
status: publish
tags:
- wifi
- sieć
- recenzja
- tp-link
- ekstender
title: 'Recenzja: Transmitery sieciowe TL-WPA4226T KIT od TP-LINK'
---

Wszystkim nam jest znany problem z niewielkim zasięgiem bezprzewodowych sieci WiFi. Nierzadko bywa
też tak, że sygnał z routera nie może się przebić przez grube ściany naszego domu, czy też większego
mieszkania. W takiej sytuacji szereg osób próbuje przestawiać router w bardziej dogodne miejsce, by
pokryć zasięgiem całą przestrzeń użytkową. Nie zawsze tego typu rozwiązanie jest jednak możliwe. Ci
bardziej majętni użytkownicy dokupują drugi router i spinają oba urządzenia mostem bezprzewodowym
WDS. O ile takie rozwiązanie może nam pomóc, to trzeba wziąć pod uwagę fakt, że punkty dostępowe
(AP) muszą być w swoim zasięgu oraz muszą wysyłać i odbierać dane, zatem użytkowa przepustowość
sieci WiFi spadnie nam dwukrotnie. Gdy do tego w grę wejdą nam jeszcze zakłócenia generowane przez
inne sieci w okolicy, to efekty mogą być niezbyt zadowalające. Poza tym, trzeba mieć również na
uwadze kompatybilność protokołu WDS, bo różni producenci inaczej go implementują. By rozwiązać ten
cały problem zasięgu w sieciach bezprzewodowych, możemy pokusić się o zakup czegoś, co nazywa się
transmiter sieciowy (ekstender powerline). Te urządzenia są w stanie "przebić się przez każdą
ścianę" i zapewnić nam połączenie z routerem WiFi na dystansie nawet do 300 metrów. Jak to
możliwe? W tym artykule odpowiemy na to pytanie testując jeden z [ekstenderów powerline TL-WPA4226T
KIT (AV500)](http://www.tp-link.com.pl/products/details/TL-WPA4226T-KIT.html) od TP-LINK'a.

<!--more-->
## Jak działa transmiter sieciowy

Pasmo WiFi, w którym operują urządzenia codziennego użytku, np. routery czy katy WiFi, ma skończone
możliwości. Nie chodzi tylko o nasycenie eteru falami radiowymi o konkretnej częstotliwości ale też
o obniżenie sygnału wraz ze wzrostem odległości od nadajnika. Jeśli dodatkowo sygnał napotka na
drodze jakieś przeszkody, to zasięg maleje drastycznie.

Jedną z najczęstszych przyczyn problemów z sygnałem w warunkach domowych są ściany i stropy. Można
oczywiście zwiększać siłę nadajnika (ew. wymienić anteny routera) i próbować pokonać takie
przeszkody ale trzeba liczyć się z faktem, że moc nadajnika w naszych routerach jest [po pierwsze
niewielka, a po drugie jest ona
skończona](http://tomatousb.org/tut:increasing-wrt54g-transmit-power). Poza tym, moc transmisyjna
nadajnika to jedna kwestia, a moc transmisyjna odbiornika (karty WiFi) to osobna sprawa. Dlatego też
zamiast się bawić w zmianę mocy transmisyjnej routera WiFi, czy wymianę jego anten, dużo lepszym
rozwiązaniem będzie implementacja ekstenderów powerline.

Transmiter sieciowy wykorzystuje inne medium do przenoszenia informacji. Nie korzysta on z sieci
bezprzewodowej, a z sieci elektrycznej. Taka sieć jest zamkniętym obwodem, do którego możemy się
wpiąć przez szereg wyprowadzeń, które każdy z nas ma w swoim domu. Wszystkie te gniazdka są zatem
ze sobą połączone i wpinając jeden transmiter do gniazdka w ścianie gdzieś w piwnicy, a drugi do
gniazdka na piętrze, możemy kompletnie zignorować ściany i słać sygnał tak jak by ich nie było. W
efekcie limituje nas jedynie jakość wykonania instalacji elektrycznej, którą mamy w domu. Jeśli nie
oszczędzaliśmy na materiałach, to sygnał może powędrować nawet 300 metrów po kablach, przynajmniej
tak zapewnia TP-LINK.

## Zawartość pudełka ekstendera powerline TL-WPA4226T KIT

Zatem sposób działania transmiterów sieciowych mamy z grubsza obgadany. Pora teraz przejść do
prezentacji samego ekstendera powerline TL-WPA4226T KIT. Na dobrą sprawę, w przypadku tego zestawu
dostajemy trzy
transmitery.

![]({{< baseurl >}}/img/2016/08/1.transmiter-sieciowy-TL-WPA4226T-KIT-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/2.transmiter-sieciowy-TL-WPA4226T-KIT-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/3.transmiter-sieciowy-TL-WPA4226T-KIT-TP-LINK.jpg#huge)

Te dwa transmitery z lewej strony to TL-WPA4220, zaś ten z prawej to TL-PA4020P.

Te transmitery sieciowe są zgodne ze [standardem HomePlug
AV](https://pl.wikipedia.org/wiki/HomePlug) i są w stanie przesyłać dane w sieci elektrycznej z
prędkością do 500 mbit/s.

### Transmiter TL-PA4020P

Transmiter TL-PA4020P ma dwa porty 100 mbit/s, dzięki którym możemy podpiąć do sieci dwa urządzenia
za pomocą przewodu RJ-45 (skrętka). Na obudowie tego transmitera mamy przycisk PAIR oraz trzy diody
LED.

![]({{< baseurl >}}/img/2016/08/8.transmiter-sieciowy-TL-PA4020P-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/9.transmiter-sieciowy-TL-PA4020P-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/10.transmiter-sieciowy-TL-PA4020P-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/11.transmiter-sieciowy-TL-PA4020P-TP-LINK.jpg#huge)

Zatem mamy trzy diody: POWER, POWERLINE, ETHERNET. Te diody mogą nam coś sygnalizować. Każda z nich
może znajdować się w trzech stanach: może się świecić, może się nie świecić oraz może też migać.

Jeśli dioda POWER się świeci oznacza to, że transmiter jest podłączony do sieci elektrycznej. Z
kolei jeśli się nie świeci, to zwykle oznacza brak zasilania. Gdy ta dioda miga, to transmiter
przeszedł w tryb czuwania lub też rozpoczęło się parowanie urządzeń.

Jeśli dioda POWERLINE się świeci, to transmiter jest podłączony do sieci, tj. gdzieś w obwodzie
elektrycznym znajduje się inny sparowany transmitter, z którym ten adapter zestawił połączenie. Gdy
ta dioda się nie świeci, oznacza to, że transmiter nie jest podłączony do sieci lub też działa on w
trybie oszczędzania energii. Migająca zaś dioda sugeruje, że transfer danych odbywa się przez sieć
elektryczną.

Jeśli dioda ETHERNET się świeci, oznacza to, że do co najmniej jednego portu ethernet jest
podłączone jakieś urządzenie. Przy czym, dane z/do tego urządzenia nie są transferowane.
Analogicznie w przypadku, gdy dioda się nie świeci. Wtedy do portów ethernet nie ma podłączonego
żadnego urządzenia. W przypadku, gdy ta dioda miga, oznacza to, że co najmniej jeden port ethernet
transmituje dane.

Tryb oszczędzania energii załącza się automatycznie po 5 minutach od wyłączenia ostatniego
urządzenia podpiętego do transmitera sieciowego. Po ponownym włączeniu tego urządzenia, transmiter
zostanie tym faktem wybudzony.

### Transmiter TL-WPA4220

Transmiter TL-WPA4220 dysponuje dwoma portami fast ethernet (100 mbit/s). Posiada także punkt
dostępowy WiFi pracujący w standardzie N do 300 mbit/s. Na obudowie tego transmitera mamy cztery
diody oraz przyciski: PAIR, WIFI CLONE i RESET.

![]({{< baseurl >}}/img/2016/08/4.transmiter-sieciowy-TL-WPA4220-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/5.transmiter-sieciowy-TL-WPA4220-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/6.transmiter-sieciowy-TL-WPA4220-TP-LINK.jpg#huge)

![]({{< baseurl >}}/img/2016/08/7.transmiter-sieciowy-TL-WPA4220-TP-LINK.jpg#huge)

W tym przypadku mamy cztery diody: POWER, POWERLINE, ETHERNET i WiFi/WiFi CLONE. Każda z tych diod
może znajdować się w trzech stanach: może się świecić, może się nie świecić oraz może też migać.

Jeśli dioda POWER się świeci oznacza to, że transmiter jest podłączony do sieci elektrycznej. Z
kolei jeśli się nie świeci, to zwykle oznacza brak zasilania. Gdy ta dioda miga, to adapter
realizuje proces parowania urządzeń.

Jeśli dioda POWERLINE się świeci, to transmiter jest podłączony do sieci, tj. gdzieś w obwodzie
elektrycznym znajduje się inny sparowany transmitter, z którym ten adapter tworzy sieć. Gdy ta dioda
się nie świeci, oznacza to, że transmiter nie jest podłączony do sieci lub też działa on w trybie
oszczędzania energii. Migająca zaś dioda sugeruje, że transfer danych odbywa się przez sieć
elektryczną.

Jeśli dioda ETHERNET się świeci, oznacza to, że do co najmniej jednego portu ethernet jest
podłączone jakieś urządzenie. Przy czym, dane z/do tego urządzenia nie są transferowane.
Analogicznie w przypadku, gdy dioda się nie świeci. Wtedy do portów ethernet nie ma podłączonego
żadnego urządzenia. W przypadku, gdy ta dioda miga, oznacza to, że co najmniej jeden port ethernet
transmituje dane.

Jeśli dioda WiFi/WiFI CLONE się nie świeci, to tryb AP transmitera został wyłączony. Jeśli ta dioda
miga wolno, to trwa proces klonowania sieci WiFi. W przypadku, gdy ta dioda miga szybko, tryb AP
transmitera działa.

## Parowanie transmiterów

Do rozszerzenia zasięgu naszej sieci bezprzewodowej potrzebne nam będą co najmniej dwa transmitery
(modele dowolne). Choć w przypadku, gdy mamy router WiFi, to lepiej jest oddelegować do niego ten
transmiter, który nie ma AP. Po połączeniu go skrętką z routerem, sygnał trafi do sieci
elektrycznej, a my będziemy mieli do rozlokowania po domu dwa transmitery z AP umożliwiające również
podłączenie urządzeń bezprzewodowo.

Ten transmiter, który jest połączony przewodem z routerem musi zaszyfrować (AES 128 bit) wszystkie
informacje, które zostaną mu przesłane przez router za pośrednictwem skrętki. Kolejne transmitery
zaś będą te informacje deszyfrować i wręczać klientom. W ten sposób żadna nieautoryzowana stacja
nie podłączy się do sieci elektrycznej, np. przez dodatkowy transmiter, i nie uzyska połączenia z
naszą siecią domową.

Każdy z transmiterów posiada przycisk PAIR odpowiadający za parowanie urządzeń. Wszystkie
transmitery, które będziemy wykorzystywać, muszą być powiązane ze sobą. By bezpiecznie przeprowadzić
proces parowania urządzeń, każdy nowy transmiter podłączany do naszej sieci musi rozpocząć proces
parowania, tj. musimy na nim wcisnąć przycisk PAIR jako pierwszy. Później wciskamy przycisk PAIR na
dowolnym adapterze, który już należy do naszej sieci. Przyciski trzymamy wciśnięte przez 1 sekundę.
Odstęp czasu między wciśnięciem dwóch przycisków PAIR nie może przekraczać 2 minut. Po przyciśnięciu
obu przycisków, proces parowania powinien zostać ukończony maksymalnie w około 60 sekund.

## Klonowanie sieci WiFi

Standardowo, każdy z transmiterów TL-WPA4220 ma domyślnie swoją konfigurację sieci WiFi (inny ESSID
oraz hasło). Trochę to problematyczne, zwłaszcza przy przemieszczaniu się między pokojami czy
kondygnacjami domu. Przy podłączaniu nowego urządzenia do sieci zwykle nie chce nam się zastanawiać
jakie dane logowania ma ten transmiter, który jest w zasięgu naszego wzroku.

Na szczęście, transmitery TL-WPA4220 posiadają przycisk WiFi CLONE, który umożliwia im skopiowanie
ustawień domowej sieci bezprzewodowej. Niemniej jednak, by sklonować sieć WiFi, nasz router musi
posiadać przycisk WPS. W sumie większość routerów taki przycisk posiada. Problemy mogą pojawić się
za to w przypadku alternatywnego firmware OpenWRT/LEDE ale o tym będzie w dalszej części artykułu. W
tym momencie załóżmy, że mamy router WiFi z działającym przyciskiem WPS.

By sklonować ustawienia sieci, wystarczy, że przyciśniemy przycisk WiFi CLONE przez dwie sekundy.
Później na routerze wciskamy przycisk WPS. Po chwili transmiter powinien uzyskać konfigurację
sieci i zaaplikować ją do swojego punktu dostępowego. W efekcie powinniśmy mieć dwa lub trzy punkty
z taką samą nazwą sieci ale różnym adresem MAC:

![]({{< baseurl >}}/img/2016/08/12.skanowanie-sieci-transmitery-sieciowe.png#huge)

## Jak zresetować transmiter sieciowy

Transmiter TL-WPA4220 w odróżnieniu do TL-PA4020P posiada przycisk RESET. Umożliwia on przywrócenie
ustawień fabrycznych urządzenia. Gdy chcemy zresetować taki transmiter, to wystarczy wcisnąć ten
przycisk i przytrzymać go przez pięć sekund. Powinno to spowodować zgaśnięcie wszystkich diod i
ponowne uruchomienie adaptera. Po chwili urządzenie będzie gotowe do pracy na domyślnych
ustawieniach.

W przypadku, gdy chcemy jedynie wyłączyć dany transmiter z sieci tak, by nie był już dłużej jej
częścią, to zamiast resetu ustawień transmitera, możemy przycisnąć przycisk PAIR przez 10 sekund.
Efektem tego działania powinno być zgaśnięcie diody POWERLINE. Pamiętajmy jednak, że w tym przypadku
ustawienia AP nie zostaną zresetowane.

## Test transmiterów sieciowych

Przydałoby się sprawdzić jak te transmitery sieciowe działają w praktyce. Wpiąłem zatem jeden z nich
do gniazdka w pokoju, w którym znajdował się [router Archer
C7](http://www.tp-link.com.pl/products/details/cat-9_Archer-C7.html). Zaś drugi transmiter
ulokowałem w piwnicy mojego bloku, cztery piętra w dół. Byłem w sumie ciekaw czy da radę w ogóle
uzyskać połączenie, bo jakość okablowania tutaj nie jest najlepsza.

W internecie można znaleźć informacje, że te transmitery mogą pracować tylko wewnątrz jednego obwodu
elektrycznego (ta sama faza). Niemniej jednak, u mnie udało się uzyskać połączenie z piwnicą w bloku
przy dwóch różnych fazach. Problem w tym, że w takiej sytuacji sygnał może być bardzo mocno
zakłócany i w efekcie transfer danych może nam mocno siąść.

Ta poniższa fotka obrazuje transfer wewnątrz jednej fazy (w mieszkaniu):

![]({{< baseurl >}}/img/2016/08/14.iperf-transmiter-sieciowy-powerline-test-transfer.png#huge)

Porty transmitera są 100 mbit/s, zatem wynik w miarę w porządku. Po przeniesieniu transmitera do
piwnicy, transfer obniżył się troszeczkę:

![]({{< baseurl >}}/img/2016/08/13.iperf-transmiter-sieciowy-powerline-test-transfer.png#huge)

Postanowiłem wyciągnąć [kamerę IP model
NC250](http://www.tp-link.com.pl/products/details/cat-19_NC250.html) również od TP-LINK'a i zrobić
prowizoryczny monitoring piwnicy. Cały setup działa bez potrzeby stosowania jakiegokolwiek
okablowania, no za wyjątkiem zasilacza kamery. Kamera łączy się po WiFi z ekstenderem, a ten zaś śle
sygnał siecią elektryczną.

Te urządzenia pracowały sobie tak 24 godziny. Średnio 12 godzin był transmitowany podgląd z kamery w
rozdzielczości 1280x720 przy 30 FPS. Transfer danych do klienta był na poziomie 15-20 mbit/s, obraz
bez zarzutu zarówno przy zapalonym świetle jak i przy zgaszonym. Ani transmiter ani kamera się nie
powiesiły w fazie testów, także bardzo przyzwoite rozwiązanie w przypadku, gdy ktoś potrzebuje
zaimplementować sobie monitoring w trudnych warunkach.

## Problemy

Przede wszystkim, unikajmy podpinania transmiterów sieciowych do listw zasilających i innych tego
typu przedłużaczy mających więcej niż jedno gniazdko. W przypadku podpięcia większej ilości urządzeń
do takiej listwy mogą być generowane zakłócenia, przez co ucierpi transfer. Jeśli już chcemy
wykorzystywać listwę, to wepnijmy ją w transmiter (ten z gniazdkiem), a sam transmiter wsadźmy
bezpośrednio do gniazdka w ścianie.

Może i transmitery sieciowe idealnie nadają się do rozbudowy sieci domowej ale troszeczkę się
grzeją. Pobierają zatem prąd i nie jest go mało. Cały zestaw może pobierać jakieś 20 W. Transmiter
TL-WPA4220 jest w stanie pobierać około 8 W, natomiast TL-PA4020P mniej więcej 6 W. Jest to dość
sporo. Na pocieszenie można dodać, że w trybie oszczędzania energii, TL-PA4020P pobiera w granicach
1 W, a TL-WPA4220 około 4 W.

W przypadku posiadania routera WiFi z alternatywnym firmware na pokładzie, np. OpenWRT/LEDE, trzeba
pokusić się o [implementację przycisku
WPS]({{< baseurl >}}/post/wps-czyli-wifi-protected-setup-w-openwrt/). Bez niego, przycisk WiFi
CLONE na transmiterach z AP będzie zwyczajnie bezużyteczny.

Parametry sieci możemy oczywiście określić ręcznie logując się do panelu administracyjnego
transmitera sieciowego posiadającego punkt dostępowy. Domyślny adres to 192.168.1.1 , czyli
dokładnie taki sam, co w przypadku routerów z firmware OpenWRT/LEDE. Tego typu sytuacja będzie
generować problemy, dlatego też zanim sparujemy taki transmiter, podłączmy się do niego i [przez
jego panel administracyjny zmieńmy adres IP na jakiś
inny]({{< baseurl >}}/post/transmiter-sieciowy-panel-admina-linux/).

TP-LINK wypuścił oprogramowanie, za pomocą którego możemy zarządzać transmiterami. Niestety nie ma
ono wersji dla linux'a, co utrudnia trochę operowanie na transmiterach. Nie powinno być to zbytnio
problematyczne, zwłaszcza, gdy będziemy mieć dostęp do panelu administracyjnego z poziomu www.
