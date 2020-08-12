---
author: Morfik
categories:
- Hardware
date: "2016-12-29T18:53:07Z"
date_gmt: 2016-12-29 17:53:07 +0100
published: true
status: publish
tags:
- lte
- router
- recenzja
- tp-link
title: 'Recenzja: Router LTE Archer MR200 od TP-LINK'
---

Do momentu upowszechnienia się technologii LTE, ludzkość była zdana na przewodowe łącza internetowe
oferowane przez lokalnych ISP. Jeśli chodzi o tych lokalnych providerów, to zwykle nie mają oni
praktycznie żadnej konkurencji w danej części miasta/wsi. Czy taki stan rzeczy jest wynikiem
dostarczania najlepszych jakościowo usług za najniższą cenę? W moim przypadku nie były to ani
najlepsze usługi, ani też najniższa cena, tylko tak dobrane przepisy prawne, by zewnętrznemu ISP nie
opłacało się świadczyć usług w mojej okolicy, bo za ten fakt dostawał on z miejsca szereg opłat/kar.
Teraz, gdy już praktycznie każdy z nas jest w zasięgu LTE, możemy porzucić tych lokalnych ISP i
obserwować ich nieuchronny upadek, no chyba, że w końcu zaczną dbać o swoich klientów. Niemniej
jednak, w dalszym ciągu, by korzystać z technologi LTE potrzebny nam jest odpowiedni sprzęt, zwykle
jest to jakiś modem, np. Huawei E3372s-153. Problem z modemem jest taki, że standardowo można go
podłączyć tylko do jednego komputera w danej chwili, no chyba, że mamy router WiFi z wgranym
alternatywnym oprogramowaniem pokroju OpenWRT/LEDE. Niemniej jednak, w dalszym ciągu trzeba nieco
wprawy, by ten modem ogarnąć i udostępnić połączenie internetowe hostom w lokalnej sieci. Dlatego
też od jakiegoś czasu na rynku zaczęły pojawiać się routery WiFi, które mają wbudowany modemem LTE.
Jedno z takich urządzeń dotarło do mnie kilka dni temu, a konkretnie jest [model Archer
MR200](http://www.tp-link.com.pl/products/details/cat-4691_Archer-MR200.html) od TP-LINK.
Postanowiłem zatem zobaczyć co oferuje ten sprzęt w standardzie, oraz czy jest dla niego jakieś
alternatywne oprogramowanie.

<!--more-->
## Zawartość pudełka routera Archer MR200

No to jedziemy z tym routerem LTE. Na początek jak zawsze fotki opakowania i tego co znajduje się w
środku:

[![]({{< baseurl >}}/img/2016/12/001.archer-mr200-tp-link-router-lte-pudelko-660x454.jpg)]({{< baseurl >}}/img/2016/12/001.archer-mr200-tp-link-router-lte-pudelko.jpg)

[![]({{< baseurl >}}/img/2016/12/002.archer-mr200-tp-link-router-lte-pudelko-zawartosc-660x529.jpg)]({{< baseurl >}}/img/2016/12/002.archer-mr200-tp-link-router-lte-pudelko-zawartosc.jpg)

Archer MR200 nie jest jakoś specjalnie duży. W sumie to jest on nawet mniejszy od TL-WR1043NDv2 i
jego dokładne wymiary to 202x141x34 mm. W zasadzie na górnej części obudowy tego urządzenia nie
widać nic specjalnego na pierwszy rzut oka. Obudowa czarna, połyskująca no i łatwo też się
rysująca.

[![]({{< baseurl >}}/img/2016/12/003.archer-mr200-tp-link-router-lte-wyglad-660x353.jpg)]({{< baseurl >}}/img/2016/12/003.archer-mr200-tp-link-router-lte-wyglad.jpg)

Na spodzie obudowy mamy zaś kartkę
wentylacyjną:

[![]({{< baseurl >}}/img/2016/12/004.archer-mr200-tp-link-router-lte-spod-660x415.jpg)]({{< baseurl >}}/img/2016/12/004.archer-mr200-tp-link-router-lte-spod.jpg)

W górnej części widzimy naklejkę z informacjami dotyczącymi numeru seryjnego urządzenia, jego wersji
sprzętowej oraz danymi logowania do sieci WiFi. Może i mamy także informację o adresie URL panelu
administracyjnego ale zabrakło adresu IP (w tym przypadku to 192.168.1.1) oraz nie ma danych
logowania do tego panelu (hasło się ustawia przy pierwszej próbie dostępu do
panelu).

[![]({{< baseurl >}}/img/2016/12/005.archer-mr200-tp-link-router-lte-info-660x123.jpg)]({{< baseurl >}}/img/2016/12/005.archer-mr200-tp-link-router-lte-info.jpg)

Otwory wentylacyjne znajdują się również po bokach routera, tuż pod
pokrywą:

[![]({{< baseurl >}}/img/2016/12/006.archer-mr200-tp-link-router-lte-bok-660x150.jpg)]({{< baseurl >}}/img/2016/12/006.archer-mr200-tp-link-router-lte-bok.jpg)

Panel tyny wygląda raczej zwyczajnie, choć są pewne różnice. Przede wszystkim switch ma cztery porty
Ethernet (RJ-45), które mogą działać w konfiguracji 4LAN lub 3LAN+1WAN. Próżno też szukać portu USB,
bo takiego zwyczajnie nie zaimplementowano. No i oczywiście mamy również slot na kartę SIM.
Przyciski nie odbiegają od przyjętych standardów, tj. Power, WPS/RESET oraz WiFi
ON/OFF:

[![]({{< baseurl >}}/img/2016/12/007.archer-mr200-tp-link-router-lte-tyl-panel-660x109.jpg)]({{< baseurl >}}/img/2016/12/007.archer-mr200-tp-link-router-lte-tyl-panel.jpg)

Archer MR200 ma w zasadzie 5 anten: 2 wewnętrzne dla sieci WiFi 2,4 GHz, 1 wewnętrzna dla sieci WiFi
5 GHz oraz 2 zewnętrzne (widoczne poniżej) dla sieci
LTE:

[![]({{< baseurl >}}/img/2016/12/008.archer-mr200-tp-link-router-lte-anteny-660x266.jpg)]({{< baseurl >}}/img/2016/12/008.archer-mr200-tp-link-router-lte-anteny.jpg)

[![]({{< baseurl >}}/img/2016/12/009.archer-mr200-tp-link-router-lte-antena-660x494.jpg)]({{< baseurl >}}/img/2016/12/009.archer-mr200-tp-link-router-lte-antena.jpg)

W zestawie mamy również jeden nieekranowany przewód Ethernet CAT5 od długości około 120
cm.

[![]({{< baseurl >}}/img/2016/12/010.archer-mr200-tp-link-router-lte-ethernet-660x549.jpg)]({{< baseurl >}}/img/2016/12/010.archer-mr200-tp-link-router-lte-ethernet.jpg)

Archer MR200 wymaga do zasilania energii elektrycznej i nie mogło w zestawie zabraknąć odpowiedniego
zasilacza
1A/12V:

[![]({{< baseurl >}}/img/2016/12/011.archer-mr200-tp-link-router-lte-zasilacz-660x474.jpg)]({{< baseurl >}}/img/2016/12/011.archer-mr200-tp-link-router-lte-zasilacz.jpg)

[![]({{< baseurl >}}/img/2016/12/012.archer-mr200-tp-link-router-lte-zasilacz-info-596x660.jpg)]({{< baseurl >}}/img/2016/12/012.archer-mr200-tp-link-router-lte-zasilacz-info.jpg)

Jako, że mamy do czynienia z routerem LTE, to do realizowania połączenia w tej technologii potrzebna
jest karta SIM, a ta z kolei może mieć różne wymiary w zależności od standardu tej karty (zwykła,
mini, mikro). Dobrze, że TP-LINK pomyślał o dołączeniu do zestawu dwóch adapterów (mikro/mini) i bez
problemu możemy do routera wsadzić kartę SIM w dowolnym
formacie.

[![]({{< baseurl >}}/img/2016/12/013.archer-mr200-tp-link-router-lte-sim-adaptery-660x273.jpg)]({{< baseurl >}}/img/2016/12/013.archer-mr200-tp-link-router-lte-sim-adaptery.jpg)

Poniżej są jeszcze fotki tylnego panelu routera z włożoną kartą
SIM:

[![]({{< baseurl >}}/img/2016/12/014.archer-mr200-tp-link-router-lte-sim-slot-660x284.jpg)]({{< baseurl >}}/img/2016/12/014.archer-mr200-tp-link-router-lte-sim-slot.jpg)

[![]({{< baseurl >}}/img/2016/12/015.archer-mr200-tp-link-router-lte-sim-slot-660x362.jpg)]({{< baseurl >}}/img/2016/12/015.archer-mr200-tp-link-router-lte-sim-slot.jpg)

Na koniec jeszcze fotka Archer'a MR200 z przykręconymi
antenami:

[![]({{< baseurl >}}/img/2016/12/016.archer-mr200-tp-link-router-lte-z-antenami-660x660.jpg)]({{< baseurl >}}/img/2016/12/016.archer-mr200-tp-link-router-lte-z-antenami.jpg)

Mało co i bym zapomniał o diodach, które naturalnie i ten router posiada. Oprócz standardowych
lampek, tj. sygnalizujących stan zasilania, połączenia z internetem, połączenia 4G, WiFi oraz portów
switch'a, mamy również informacje na temat poziomu sygnału
LTE.

[![]({{< baseurl >}}/img/2016/12/017.archer-mr200-tp-link-router-lte-led-660x376.jpg)]({{< baseurl >}}/img/2016/12/017.archer-mr200-tp-link-router-lte-led.jpg)

## Specyfikacja routera Archer MR200

Jak widzieliśmy wyżej, ten router LTE w sumie za wiele się nie różni pod względem wyglądu od
tradycyjnego routera WiFi. Niemniej jednak, te różnice są i to dość znaczne. Poniżej znajdują się
fotki z wnętrza obudowy routera Archer
MR200:

[![]({{< baseurl >}}/img/2016/12/018.archer-mr200-tp-link-router-lte-obudowa-wnetrze-660x443.jpg)]({{< baseurl >}}/img/2016/12/018.archer-mr200-tp-link-router-lte-obudowa-wnetrze.jpg)

[![]({{< baseurl >}}/img/2016/12/019.archer-mr200-tp-link-router-lte-pcb-660x495.jpg)]({{< baseurl >}}/img/2016/12/019.archer-mr200-tp-link-router-lte-pcb.jpg)

[![]({{< baseurl >}}/img/2016/12/020.archer-mr200-tp-link-router-lte-pcb-660x495.jpg)]({{< baseurl >}}/img/2016/12/020.archer-mr200-tp-link-router-lte-pcb.jpg)

[![]({{< baseurl >}}/img/2016/12/021.archer-mr200-tp-link-router-lte-pcb-660x446.jpg)]({{< baseurl >}}/img/2016/12/021.archer-mr200-tp-link-router-lte-pcb.jpg)

[![]({{< baseurl >}}/img/2016/12/022.archer-mr200-tp-link-router-lte-pcb-660x500.jpg)]({{< baseurl >}}/img/2016/12/022.archer-mr200-tp-link-router-lte-pcb.jpg)

### Modem LTE

To co z pewnością odróżnia Archer'a MR200 od reszty routerów, to fakt zainstalowania w nim modemu
LTE. Dokładnie nie wiadomo jednak jakiej kategorii jest ten modem. Jedni mówią, że jest to
standardowy CAT4 umożliwiający transfer 150/50 mbit/s. Z kolei zaś na stronie producenta jest
oznaczenie `LTE adv CAT4, LTE FDD/TDD CAT3` . [Poruszając tę kwestię forum
JDTech](http://forum.jdtech.pl/Watek-tp-link-archer-mr200-modem-lte-w-tym-routerze) okazało się, że
Qualcomm nieco nagina rzeczywistość i ten modem może ma frazę `adv` sugerującą agregację pasm (tj.
LTE-A) ale w zasadzie jest to agregacja wewnątrz tego samego pasma. Dokładnie nie wiem jak mam
rozumieć to powyższe oznaczenie ale jedyne co mi przyszło do głowy, to, że ten modem ma CAT3 (100/50
mbit/s), a dopiero po agregacji (w ciąż w tym samym paśmie) jest w stanie osiągnąć te 150/50 mbit/s.
Czy tak jest w istocie, tego nie wiem ale bezpiecznie można tak założyć, bo gdyby napisano te
parametry w przejrzysty i bardziej zrozumiały dla człowieka sposób, to nie byłoby wątpliwości.

Ten modem jest w stanie realizować połączenie LTE w technologi zarówno FDD, kanały B1/B3/B7/B8/B20
(2100/1800/2600/900/800 MHz), jak i TDD kanały B38 i B40 (2600/2300 MHz). Sam modem obsługuje także
DC-HSPA+/HSPA+/HSPA/UMTS, kanały B1/B8(2100/900 MHz) oraz EDGE/GPRS/GSM, kanały B2/B3/B5/B8
(1900/1800/900/850).

Tak ten modem wygląda z obu
stron:

[![]({{< baseurl >}}/img/2016/12/024.archer-mr200-tp-link-router-lte-modem-660x374.jpg)]({{< baseurl >}}/img/2016/12/024.archer-mr200-tp-link-router-lte-modem.jpg)

[![]({{< baseurl >}}/img/2016/12/025.archer-mr200-tp-link-router-lte-modem-660x402.jpg)]({{< baseurl >}}/img/2016/12/025.archer-mr200-tp-link-router-lte-modem.jpg)

[![]({{< baseurl >}}/img/2016/12/026.archer-mr200-tp-link-router-lte-modem-660x378.jpg)]({{< baseurl >}}/img/2016/12/026.archer-mr200-tp-link-router-lte-modem.jpg)

[![]({{< baseurl >}}/img/2016/12/027.archer-mr200-tp-link-router-lte-modem-660x414.jpg)]({{< baseurl >}}/img/2016/12/027.archer-mr200-tp-link-router-lte-modem.jpg)

Na PCB modemu znajduje się czip LTE [Qualcomm
MDM9225](https://www.qualcomm.com/news/releases/2013/02/25/qualcomm-first-demonstrate-lte-advanced-carrier-aggregation-enables)
(procek):

![]({{< baseurl >}}/img/2016/12/028.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

Modem ma 256 MiB NAND flash i 128 DDRII SDRAM pamięci operacyjnej, układ [ESMT
FM6BD2G1GA](http://www.esmt.com.tw/english/products_de.asp?CLASS_L1=13&CLASS_L2=231&CLASS_L3=0&CLASS_L4=0):

![]({{< baseurl >}}/img/2016/12/029.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

Znajdziemy też tam wzmacniacz sygnału LTE, moduł [SKYWORKS
SKY77814](http://www.skyworksinc.com/Product/1801/SKY77814-11) (FDD: B7 oraz TDD: B38, B40), oraz
przełącznik antenowy
[SKYWORKS 13492](http://www.skyworksinc.com/Product/1755/SKY13492-21).

[![]({{< baseurl >}}/img/2016/12/030.archer-mr200-tp-link-router-lte-modem-podzespoly-331x660.jpg)]({{< baseurl >}}/img/2016/12/030.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

Na modemie można także znaleźć jeszcze czip Qualcomm PM8019, który zarządza zasilaniem podpiętych do
niego
układów:

[![]({{< baseurl >}}/img/2016/12/031.archer-mr200-tp-link-router-lte-modem-podzespoly-660x526.jpg)]({{< baseurl >}}/img/2016/12/031.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

Jest też drugi czip LTE [Qualcomm
WTR1605L](http://www.anandtech.com/show/6541/the-state-of-qualcomms-modems-wtr1605-and-mdm9x25)
(część
RF):

[![]({{< baseurl >}}/img/2016/12/032.archer-mr200-tp-link-router-lte-modem-podzespoly-660x536.jpg)]({{< baseurl >}}/img/2016/12/032.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

I kolejny wzmacniacz sygnału LTE i GSM, moduł [SKYWORKS
SKY77629](http://www.skyworksinc.com/Product/1623/SKY77629) (FDD: B1, B3, B8, GSM: B2, B3 ,B5,
B8):

[![]({{< baseurl >}}/img/2016/12/033.archer-mr200-tp-link-router-lte-modem-podzespoly-625x660.jpg)]({{< baseurl >}}/img/2016/12/033.archer-mr200-tp-link-router-lte-modem-podzespoly.jpg)

### WiSoC MT7620A (MediaTek)

Archer MR200 został wyposażony w SoC MT7620A
([datasheet](http://www.anz.ru/files/mediatek/MT7620_Datasheet.pdf)) od MediaTek'a, z procesorem
MIPS 24KEc taktowanym zegarem 580 MHz. Ten SoC ma wbudowany czip WiFi 2,4 GHz (MediaTek RT5390) oraz
układ switch'a (MediaTek
MT7530).

[![]({{< baseurl >}}/img/2016/12/034.archer-mr200-tp-link-router-lte-podzespoly-wisoc-660x364.jpg)]({{< baseurl >}}/img/2016/12/034.archer-mr200-tp-link-router-lte-podzespoly-wisoc.jpg)

WiFi 2,4 GHz jest w standardzie N do 300 mbit/s (dwa strumienie przestrzenne). Dla tego pasma sieci
bezprzewodowej mamy dwie anteny wewnętrzne rozlokowane po bokach PCB (wlutowane na stałe). Poniżej
jest fotka jednej z tych
anten:

[![]({{< baseurl >}}/img/2016/12/035.archer-mr200-tp-link-router-lte-antena-wifi-2-4-ghz-660x495.jpg)]({{< baseurl >}}/img/2016/12/035.archer-mr200-tp-link-router-lte-antena-wifi-2-4-ghz.jpg)

### WiFi 5GHz

Router Archer MR200 jest także wyposażony w osobny układ dla WiFi w paśmie 5 GHz (MediaTek MT7610EN)
ze wzmacniaczem
[SKYWORKS 85703](http://www.skyworksinc.com/Product/1706/SKY85703-11):

[![]({{< baseurl >}}/img/2016/12/036.archer-mr200-tp-link-router-lte-podzespoly-wifi-5-ghz-660x505.jpg)]({{< baseurl >}}/img/2016/12/036.archer-mr200-tp-link-router-lte-podzespoly-wifi-5-ghz.jpg)

W tym przypadku mamy do czynienia ze standardem AC do 433 mbit/s, czyli jeden strumień przestrzenny,
no i jedną antenę
wewnętrzną:

[![]({{< baseurl >}}/img/2016/12/037.archer-mr200-tp-link-router-lte-antena-wifi-5-ghz-660x428.jpg)]({{< baseurl >}}/img/2016/12/037.archer-mr200-tp-link-router-lte-antena-wifi-5-ghz.jpg)

### Switch 100 mbit/s

Niestety Archer MR200 nie może się pochwalić jeśli chodzi o switch. Ma on cztery porty w stosunku do
standardowych 5, choć zwykle więcej i tak nie trzeba. Natomiast maksymalna przepustowość tych portów
to niestety tylko 100 mbit/s. Transformatory Ethernet pochodzą od GROUP-TEK,
model[HST-2027DAR](http://www.datasheetspdf.com/datasheet/download.php?id=805315)).

[![]({{< baseurl >}}/img/2016/12/038.archer-mr200-tp-link-router-lte-switch-660x350.jpg)]({{< baseurl >}}/img/2016/12/038.archer-mr200-tp-link-router-lte-switch.jpg)

### 64 MiB pamięci RAM

Jeśli zaś chodzi o kwestię pamięci operacyjnej RAM, to Archer MR200 ma jej raczej niewiele, bo 64
MiB. Układ pamięci to [Winbond
W9751G6KB-25](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjsnNL_5pbRAhXKiywKHVTGAAYQFggpMAA&url=https%3A%2F%2Fwww.winbond.com%2Fresource-files%2Fda00-w9751g6kbg1.pdf&usg=AFQjCNG_zAe0_qOGJC8OxwHLOAXlonmtYg&sig2=xqVJIHEirnRxZZwYiyLiiQ):

[![]({{< baseurl >}}/img/2016/12/039.archer-mr200-tp-link-router-lte-ram-660x323.jpg)]({{< baseurl >}}/img/2016/12/039.archer-mr200-tp-link-router-lte-ram.jpg)

### 8 MiB pamięć flash

Podobnie jak i pamięci RAM, pamięć flash również nie jest zbyt duża, bo jedyne 8 MiB. Flash pochodzi
od Macronix i jest to model
25L6433FM2I-08G

[![]({{< baseurl >}}/img/2016/12/040.archer-mr200-tp-link-router-lte-flash-660x461.jpg)]({{< baseurl >}}/img/2016/12/040.archer-mr200-tp-link-router-lte-flash.jpg)

## Konfiguracja routera Archer MR200

Nie da się ukryć, że posiadanie routera WiFi z wbudowanym modemem LTE, ma swoje liczne zalety. W
przypadku tego Archer'a MR200 na stock'owym firmware nie musimy przeprowadzać praktycznie żadnych
dodatkowych czynności, by udostępnić internet LTE komputerom w domowej sieci. Jedyne co musimy
zrobić to wsadzić kartę SIM i w zasadzie to wszystko.

Panel administracyjny routera znajduje się standardowo pod adresem `http://192.168.1.1/` i za jego
pomocą możemy dostosować sobie poszczególne aspekty pracy urządzenia. Ci, którzy chcą rzucić okiem
na ten panel i sprawdzić czy znajdą w nim określone opcje, zawsze mogą [skorzystać z
emulatora](http://static.tp-link.com/resources/simulator/MR200%20Emulator/index.htm) dostępnego na
stronie TP-LINK.

W zasadzie wszystkie funkcje standardowego routera WiFi jak i modemu LTE są zaimplementowane w
Archer MR200. Niemniej jednak, jest kilka rzeczy, o których warto wspomnieć.

### Tryby pracy routera

Sam router można przełączyć w tryb zwykły (WiFi) lub LTE. W przypadku trybu LTE, wszystkie cztery
porty switch'a są w konfiguracji LAN. Jeśli wybierzemy tryb zwykły (WiFi), to modem LTE zostanie
wyłączony, a jeden z portów switch'a zostanie skonfigurowany jako WAN i będziemy mogli podpiąć się
pod przewodowego
ISP.

[![]({{< baseurl >}}/img/2016/12/041.archer-mr200-tp-link-router-lte-panel-admina-tryby-660x199.png)]({{< baseurl >}}/img/2016/12/041.archer-mr200-tp-link-router-lte-panel-admina-tryby.png)

### Tryby pracy modemu LTE

Modem LTE może pracować w trzech trybach: auto, LTE lub 3G. Niestety nie mamy możliwości określenia
częstotliwości
pracy:

[![]({{< baseurl >}}/img/2016/12/042.archer-mr200-tp-link-router-lte-panel-admina-wan-660x420.png)]({{< baseurl >}}/img/2016/12/042.archer-mr200-tp-link-router-lte-panel-admina-wan.png)

Nie zabrakło także obsługi kodu PIN dla karty SIM z automatycznym odblokowaniem karty przy starcie
routera:

[![]({{< baseurl >}}/img/2016/12/043.archer-mr200-tp-link-router-lte-panel-admina-sim-pin-660x278.png)]({{< baseurl >}}/img/2016/12/043.archer-mr200-tp-link-router-lte-panel-admina-sim-pin.png)

Jak i również mamy możliwość ustawienia limitu transferu z opcją odłączenia od sieci po jego
przekroczeniu. Zabrakło jedynie ficzera, który by nie naliczał transferu w określonych
godzinach:

[![]({{< baseurl >}}/img/2016/12/044.archer-mr200-tp-link-router-lte-panel-admina-transfer-660x316.png)]({{< baseurl >}}/img/2016/12/044.archer-mr200-tp-link-router-lte-panel-admina-transfer.png)

### WPS

Jeśli zaś chodzi o WiFi, to mamy możliwość szybkiej konfiguracji parametrów połączenia za pomocą
funkcji WPS. Obsługiwane są zarówno PIN jak i PBC
(przyciski):

[![]({{< baseurl >}}/img/2016/12/045.archer-mr200-tp-link-router-lte-wifi-wps-660x388.png)]({{< baseurl >}}/img/2016/12/045.archer-mr200-tp-link-router-lte-wifi-wps.png)

### Sieć gościnna

Nie mogło zabraknąć też bezprzewodowej sieci gościnnej. W przypadku Archer'a MR200 mamy możliwość
skonfigurowania izolowania od siebie konkretnych jej użytkowników oraz też możemy zdecydować czy
zezwalamy im na ewentualny dostęp do sieci lokalnej. Sieć gościnną możemy włączyć osobno dla WiFi
2,4 GHz i 5 GHz i skonfigurować je niezależnie. Innym użytecznym dodatkiem jest także kontrola pasma
jakie jest przeznaczone dla sieci
gości:

[![]({{< baseurl >}}/img/2016/12/046.archer-mr200-tp-link-router-lte-panel-admina-siec-goscinna-660x452.png)]({{< baseurl >}}/img/2016/12/046.archer-mr200-tp-link-router-lte-panel-admina-siec-goscinna.png)

### Harmonogram pracy WiFi

Dostępny jest również harmonogram pracy sieci WiFi i to osobny dla pasma 2,4 GHz jak i 5 GHz. Przy
jego pomocy możemy sobie ustawić automatyczne włączanie i wyłączanie sieci WiFi w zależności od pory
dnia:

[![]({{< baseurl >}}/img/2016/12/047.archer-mr200-tp-link-router-lte-panel-admina-harmonogram-wifi-660x328.png)]({{< baseurl >}}/img/2016/12/047.archer-mr200-tp-link-router-lte-panel-admina-harmonogram-wifi.png)

### Pisanie i odbieranie SMS'ów

Routery LTE mają to do siebie, że odbierają one również sygnał sieci GSM, czyli jesteśmy w stanie
odbierać i wysyłać wiadomości
SMS.

[![]({{< baseurl >}}/img/2016/12/0471.archer-mr200-tp-link-router-lte-panel-admina-sms-660x345.png)]({{< baseurl >}}/img/2016/12/0471.archer-mr200-tp-link-router-lte-panel-admina-sms.png)

## Darmowy internet od Aero2

Jeśli posiadamy kartę SIM od Aero2, to Archer MR200 jest w stanie zapewnić naszej sieci darmowy
internet od tego operatora. Niemniej jednak, konfiguracja w tym przypadku nie jest automatyczna. Z
tego co można zauważyć w panelu administracyjnym, to błędnie jest konfigurowany APN. Dlatego też
trzeba stworzyć nowy profil i odpowiednio go
uzupełnić:

[![]({{< baseurl >}}/img/2016/12/0472.archer-mr200-tp-link-router-lte-panel-admina-aero2-apn-660x457.png)]({{< baseurl >}}/img/2016/12/0472.archer-mr200-tp-link-router-lte-panel-admina-aero2-apn.png)

Użytkownik i hasło może być albo puste, albo można też wpisać `aero` .

## Wydajność sieci WiFi i LTE

Generalnie rzecz biorąc, router Archer MR200 ma WiFi 2,4 GHz (max. transfer 300 mbit/s), WiFi 5 GHz
(max. transfer 433 mbit/s), LTE (max transfer 150/50 mbit/s) no i switch z portami Fast Ethernet
(max transfer 100 mbit/s). W efekcie komunikacja przewodowo-bezprzewodowa będzie limitowana z
automatu do 100 mbit/s. Jeśli zaś chodzi o bezprzewodowych klientów to powinniśmy wyczerpać
możliwości modemu LTE. Natomiast transfer w sieci bezprzewodowej (między dwoma klientami) powinien
wynieść 150 mbit/s i 215 mbit/s odpowiednio dla 2,4 GHz i 5 GHz. Oczywiście wszystkie wartości są
teoretyczne i przydałoby się je poddać weryfikacji.

Może i znamy realną przepustowość portów Ethernet (~93 mbit/s) ale na dobrą sprawę nie znamy
faktycznej przepustowości sieci WiFi 2 i 5 GHz. Dlatego też zanim sprawdzimy sobie na ile wydolny
jest modem LTE, to pierw ustalmy ile damy radę wyciągnąć po samym WiFi.

Transfer jaki udało mi się uzyskać w sieci WiFi w paśmie 2,4 GHz między dwoma bezprzewodowymi
klientami, to około 80 mbit/s, czy ogólny transfer jest na poziomie 160 mbit/s. Jeśli na drodze
staną dwie niezbyt grube ściany (dystans 5 metrów), wtedy ten transfer spada do około 30
mbit/s.

![]({{< baseurl >}}/img/2016/12/048.archer-mr200-tp-link-router-lte-transfer-wifi-2-4-ghz.png)

Jeśli zaś chodzi o transfer w paśmie 5 GHz, to udało się osiągnąć nieco ponad 120 mbit/s wewnątrz
sieci WiFi, czyli transfer na poziomie około 250 mbit/s. Generalnie ta niezbyt wypasiona antena
wewnętrzna powoduje, że zasięg tego routera w tym paśmie WiFi jest, by nikogo nie urazić, bardzo
ograniczony. Generalnie to nie jest problemem sama odległość ale ściany i inne przeszkody na drodze.
W przypadku tego routera wystarczy jedna ściana, by praktycznie zatrzymać sygnał 5 GHz, tj. transfer
spada poniżej 10 mbit/s. Dwóch ścian już sygnał nie jest w stanie pokonać. Jeśli rozważamy zakup
tego routera pod kątem korzystania z sieci 5 GHz i planujemy z niej korzystać w miarę komfortowo, to
upewnijmy się, że nic nie będzie stać na drodze
sygnału.

![]({{< baseurl >}}/img/2016/12/049.archer-mr200-tp-link-router-lte-trasnfer-wifi-5-ghz.png)

Wiemy zatem, że obie sieci WiFi są nam w stanie zapewnić prędkość, która jest porównywalna z
prędkością modemu LTE (100 mbit/s, w agregacji tego samego pasma 150 mbit/s). Przeprowadziłem w
sumie dwie serie testów posługując się moim smartfonem. Pierwsza seria testowała transfer modemu LTE
w routerze Archer MR200. Druga seria zaś badała mój modem Huawei E3372s-153, który jest podłączony
pod router Archer C2600. Poniżej jest zestawienie
wyników:

[![]({{< baseurl >}}/img/2016/12/050.archer-mr200-tp-link-router-lte-trasnfer-lte-660x519.png)]({{< baseurl >}}/img/2016/12/050.archer-mr200-tp-link-router-lte-trasnfer-lte.png)

O ile generalnie prędkość pobierania jest w miarę porównywalna, choć się bardzo waha (w końcu LTE),
to w przypadku transferu danych do BTS już widać różnicę. Transfer jest dwa razy większy, więc
jakość sygnału docierająca do BTS jest o wiele lepsza niż w przypadku modemu Huawei E3372s-153.
Niemniej jednak, jeśli przestawię te urządzenia okolice okna, to wtedy różnicy zbytnio już nie ma
(odległość od BTS to około 1 km, widoczność zachowana).

## Pobór energii

W zasadzie router Archer MR200 nie utylizuje za dużo energii elektrycznej. Podczas startu urządzenia
jest to wartość nieprzekraczająca 1,5 W. Przy włączonym WiFi i LTE router zjada w stanie spoczynku
około 3W. Zużycie energii wzrasta dość znacznie, gdy zaczynamy pobierać/wysyłać dane w sieci
WiFi=\>LTE. W takim przypadku pobieranie z prędkością około 50 mbit/s owocuje wzrostem poboru prądu
do około 5,5W. Przy wysyłaniu danych z prędkością około 25 mbit/s, ta wartość jest jeszcze większa i
sięga około 7-8W. Naturalnie im szybciej będziemy dane pobierać/wysyłać, tym bardziej prądożerny
będzie nasz router.

## Wsparcie w LEDE/OpenWRT

Router Archer MR200 doczekał się wsparcia w LEDE ale ten firmware można wgrać póki co jedynie przez
tryb recovery bootloader'a. Instrukcja jak to zrobić [znajduje się
tutaj](http://tplink-forum.pl/index.php?/topic/5294-archer-mr200-lede/). Jeśli zaś jest ktoś
zainteresowany alternatywnym firmware pod kątem tego routera, to tutaj jest wątek na [forum
OpenWRT](https://forum.openwrt.org/viewtopic.php?id=64293) oraz na [forum
eko.one.pl](http://eko.one.pl/forum/viewtopic.php?id=14291). Warto też dodać, że nie wszystkie
rzeczy jeszcze działają, np. WiFi w paśmie 5 GHz.

## TL;DR

Zewnętrzne anteny LTE zapewniają Archer'owi MR200 nawet dość przyzwoitą jakość sygnału w stosunku do
tradycyjnego modemu Huawei E3372s-153. Ten router oferuje zbliżoną prędkość pobierania danych z BTS
ale jeśli chodzi o prędkość wysyłania, to już tutaj te anteny zewnętrzne trochę pomagają mu.
Niemniej jednak, anteny wewnętrzne dla WiFi 2,4 GHz i 5 GHz sprawiają, że jakość sygnału
docierającego do komputerów/telefonów nie jest najlepszej jakości, przez co możemy zapomnieć o
bonusie płynącym z zewnętrznych anten LTE.

Jeśli BTS jest dość daleko to faktycznie te anteny mogą zaważyć na tym czy w ogóle jakiś sygnał LTE
zostanie odebrany przez modem w routerze. Niemniej jednak jeśli BTS jest w zasięgu wzroku i nie ma
żadnych przeszkód na drodze sygnału, to WiFi będzie wąskim gardłem jeśli sygnał będzie musiał się
przebijać przez ściany domu/mieszkania, by dotrzeć do naszych sprzętów. Pamiętajmy też, że może i te
anteny LTE są zewnętrzne ale nadal nie są zamocowane na zewnątrz budynku i nie są skierowane w
stronę BTS'a.

Porty Ethernet 100 mbit/s? Jakby nie patrzeć routery WiFi są rdzeniem naszej sieci domowej i powinny
być w stanie zapewnić przyzwoitą komunikację przynajmniej w jej obrębie. Nawet te tanie routery
pokroju TL-WR1043NDv2 mają gigabitowy switch, a 100 mbit/s po przewodzie w LAN, to już nie te czasy.

Brak portu USB boli chyba najbardziej. Ja generalnie korzystam z portu USB mojego głównego routera
Archer C2600 zwykle w celu udostępniania drukarki w sieci. W efekcie nie muszę ani przełączać samej
drukarki, ani też stawiać dedykowanej maszyny do jej obsługi. W przypadku Acher'a MR200 takie
rozwiązanie niestety odpada.

Archer MR200 jest stosunkowo drogim routerem (~540 zł) jak na te parametry, które oferuje. Gdybym
miał wybierać, to jednak zdecydowałbym się na zakup Archer'a C7 v2 + modem Huawei E3372s-153 (wersja
zwykła) i cenowo by wyszło podobnie (400+150 zł) ale pod względem wydajności, to ten zestaw bije
Archer'a MR200 praktycznie pod każdym względem (no za wyjątkiem LTE), już pomijając, że są to dwa
niezależne urządzenia, które w przypadku awarii/zużycia można wymienić bez potrzeby wymiany również
i drugiego urządzenia.

Dobrze chociaż, że pojawia się wsparcie dla tego routera w LEDE/OpenWRT. Niemniej jednak, jeśli
chodzi o alternatywny firmware, to uważam, że Archer C7 v2 + Huawei E3372s-153 jest lepszym wyborem,
gdy rozważamy zabawę z tym oprogramowaniem. Niemniej jednak, w przypadku stock'owego firmware,
Archer MR200 nie wymaga znajomości żadnych zagadnień informatycznych i praktycznie każdy jest go w
stanie skonfigurować bez większego problemu, co nie zawsze ma miejsce w przypadku LEDE/OpenWRT.
