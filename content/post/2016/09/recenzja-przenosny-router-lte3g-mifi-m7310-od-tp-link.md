---
author: Morfik
categories:
- Hardware
date: "2016-09-25T22:06:06Z"
date_gmt: 2016-09-25 20:06:06 +0200
published: true
status: publish
tags:
- lte
- router
- recenzja
- tp-link
GHissueID: 382
title: 'Recenzja: Przenośny router LTE/3G (MiFi) M7310 od TP-LINK'
---

Wielu zaawansowanych użytkowników komputera, zwłaszcza tych korzystających z linux'a, bardzo ceni
sobie możliwość bezgranicznego zarządzania posiadanym sprzętem elektronicznym. Nie chodzi tylko o
zwykłego desktopa czy laptopa ale też o urządzenia z systemami wbudowanymi jak, np. routery WiFi.
Większość routerów nie obsługuje modemów LTE ale mogłaby, gdyby pozwalało im na to zainstalowane na
nich oprogramowanie. Na moim blogu jest kilka artykułów dotyczących [instalacji i konfiguracji
modemów LTE](/tags/modem/) właśnie na takich routerach, z tym, że w oparciu o
firmware OpenWRT/LEDE. Niemniej jednak, w przypadku takiego alternatywnego firmware wymagana jest
drobna znajomość obsługi komputera, a cała masa użytkowników nie chce poświęcać czasu na zgłębianie
technik informatycznych. Te osoby chcą zwyczajnie podłączyć dane urządzenie do prądu i móc z niego
korzystać tuż po wyjęciu go z pudełka. Mam właśnie jedno takie urządzenie, które byłoby w stanie
zaspokoić większość osób z tego grona. Nie jest to co prawda pełnowymiarowy router WiFi z modemem
LTE na pokładzie ale w przeciwieństwie do swoich kolegów jest o wiele bardziej mobilny. Mowa o
[przenośnym hotspocie LTE M7310](http://www.tp-link.com.pl/products/details/M7310.html) od TP-LINK.

<!--more-->
## Zawartość opakowania

Poniżej są fotki opakowania i jego zawartości. Sorry za pomięte pudło ale najwyraźniej ktoś tym
routerem rzucał podczas transportu. Niemniej jednak, sam M7310 dotarł sprawny i przeżył podróż w
tych nieludzkich warunkach zafundowanych za free przez pocztę polską.

![mobilny-router-wifi-lte-hotspot-M7310-pudelko](/img/2016/09/1.mobilny-router-wifi-lte-hotspot-M7310-pudelko.jpg#huge)

![mobilny-router-wifi-lte-hotspot-M7310-pudelko-zawartosc](/img/2016/09/2.mobilny-router-wifi-lte-hotspot-M7310-pudelko-zawartosc.jpg#huge)

Jak widać, M7310 jest raczej niewielkich rozmiarów: 98 x 60 x 16 mm (dł/sz/wy). Ma on wbudowany
wyświetlacz TFT 1,44 cala. Ten wyświetlacz jest bardzo niskiej jakości i widać na nim pojedyncze
piksele, co trochę wali po oczach. Efekt mniej więcej taki jak w przypadku pierwszych telefonów
komórkowych z kolorowym wyświetlaczem. Da radę patrzeć ale nie zbyt długo. Obok wyświetlacza są
także ulokowane dwa przyciski, przy pomocy których można sterować urządzeniem:

![mobilny-router-wifi-lte-hotspot-M7310-wyglad](/img/2016/09/3.mobilny-router-wifi-lte-hotspot-M7310-wyglad.jpg#huge)

Na boku obudowy mamy także ulokowany port mikro USB typ B:

![mobilny-router-wifi-lte-hotspot-M7310-port-usb](/img/2016/09/4.mobilny-router-wifi-lte-hotspot-M7310-port-usb.jpg#huge)

Przy jego pomocy jesteśmy w stanie podłączyć router do portu USB komputera w celu ładowania i
ewentualnej interakcji z kartą SD. Jeśli chodzi zaś o samą kwestię ładowania, to możemy tutaj
również podpiąć zwykłą ładowarkę 5V/1A czy też 5V/2A i przy jej pomocy zasilić i naładować
urządzenie. Ładowarki nie ma w zestawie, jedynie jest tylko widoczny wyżej krótki przewód USB. Co
ciekawe, po podłączeniu do komputera, w systemie pojawia się nowy interfejs sieciowy, który można
skonfigurować w taki sam sposób jak interfejs zwyczajnej karty sieciowej.

Może i jesteśmy w stanie zasilić M7310 bezpośrednio z sieci elektrycznej ale bez baterii ten router
się nam nie uruchomi. Na szczęście do zestawu jest dołączony akumulator litowo-jonowy (model
TBL-55A2000) o pojemności 2000 mAh.

![mobilny-router-wifi-lte-hotspot-M7310-bateria-akumulator](/img/2016/09/5.mobilny-router-wifi-lte-hotspot-M7310-bateria-akumulator.jpg#huge)

Czytając to info na etykiecie baterii, nie wiem dokładnie co ona robi po wrzuceniu w ogień ale
lepiej tego nie sprawdzać. :D

Spód routera jest standardowy, tj. śliski i bez jakichkolwiek nóżek, choć przydałyby mu się one, bo
w końcu to urządzenie ma należeć do tych wysoce mobilnych. Jeden z rogów M7310 umożliwia łatwe
otwarcie obudowy urządzenia:

![mobilny-router-wifi-lte-hotspot-M7310-spod](/img/2016/09/6.mobilny-router-wifi-lte-hotspot-M7310-spod.jpg#huge)

W środku obudowy znajdziemy zaś jeden slot na kartę Mini SIM oraz drugi slot dla kart mikro SD (max.
32 GiB). Oba sloty mają metalowe zabezpieczenia przed ewentualnym wysunięciem lub przemieszczeniem
się kart:

![mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd](/img/2016/09/7.mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd.jpg#huge)

![mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd](/img/2016/09/8.mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd.jpg#huge)

![mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd](/img/2016/09/9.mobilny-router-wifi-lte-hotspot-M7310-slot-karta-sim-sd.jpg#huge)

W przypadku, gdy dysponujemy jedynie kartą SIM w rozmiarze mikro, to trzeba będzie skorzystać z
adaptera mini =\> mikro, który jest dołączony do zestawu. Niestety te zwykłe karty SIM są za duże i
nie zmieszczą się nam w ogóle do slotu. Poniżej fotka adaptera mini =\> mikro SIM:

![mobilny-router-wifi-lte-hotspot-M7310-adapter-karta-sim](/img/2016/09/10.mobilny-router-wifi-lte-hotspot-M7310-adapter-karta-sim.jpg#huge)

Oba te sloty są przysłaniane baterią i nie ma możliwości wyciągnięcia karty SD bez wyciągania
akumulatora, co skutkuje wyłączeniem urządzenia:

![mobilny-router-wifi-lte-hotspot-M7310-bateria](/img/2016/09/11.mobilny-router-wifi-lte-hotspot-M7310-bateria.jpg#huge)

## Specyfikacja routera M7310

Zgodnie ze specyfikacją podaną na opakowaniu (i stronie TP-LINK), router M7310 potrafi obsługiwać
LTE w technologi FDD: B1/B3/B7/B8/B20(2100/1800/2600/900/800MHz) oraz TDD: B38/B40/B41
(2600/2300/2500MHz). Deklarowana prędkość to 150/50 mbit/s down/up. Ten router jest także w stanie
obsługiwać standardy DC-HSPA+/HSPA/UMTS: B1/B8(2100/900MHz) oraz EDGE/GPRS/GSM:
850/900/1800/1900MHz. Niemniej jednak, nas będzie interesować jedynie LTE.

Warto zaznaczyć, że nie mamy możliwości wyboru konkretnej częstotliwości pracy modemu LTE. Jest ona
dobierana automatycznie w zależności od panujących warunków, tj. dostępności BTS'ów w okolicy.

Jeśli chodzi zaś o WiFi, to M7310 obsługuje zarówno pasmo 2,4 GHz jak i 5 GHz ale tylko jedno z tych
dwóch pasm może być wykorzystywane w danym momencie. Deklarowana prędkość transmisji to 300 mbit/s,
zatem mamy do czynienia ze standardem N w obu przypadkach.

## Sterowanie routerem M7310 za pomocą przycisków na obudowie

Wsadźmy zatem katę SIM do slotu i podłączmy baterię. Router włączamy przyciskiem power trzymanym
przez około 5 sekund. Po chwili router powinien zalogować się do sieci 3G/4G oraz zestawić sieć
WiFi, do której będziemy mogli podłączyć do 10 stacji klienckich. Aktualna ilość podłączonych
urządzeń jest cały czas wyświetlana na tym małym ekraniku na ikonce WiFi:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-lte](/img/2016/09/12.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-lte.jpg#huge)

Jako, że router M7310 jest w stanie nadawać w paśmie 2,4 GHz jak i 5 GHz, to możemy to pasmo sobie
dostosować bez większego problemu:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-wifi](/img/2016/09/13.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-wifi.jpg#huge)

Dane logowania do sieci WiFi (ESSID i hasło) w przypadku tego mobilnego hotspotu można uzyskać na
dwa sposoby. Jednym z nich jest zdjęcie tylnej klapki i zajrzenie na jej wewnętrzną stronę:

![mobilny-router-wifi-lte-hotspot-M7310-klapa-haslo-wifi-](/img/2016/09/14.mobilny-router-wifi-lte-hotspot-M7310-klapa-haslo-wifi-.jpg#huge)

Drugim i do tego bardziej wygodnym i przystępnym sposobem jest odczytanie tych danych z wyświetlacza
(można je ukryć za pomocą panelu admina):

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-haslo-wifi](/img/2016/09/15.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-haslo-wifi.jpg#huge)

M7310 ma także wbudowaną programową funkcję WPS, dzięki której nie będziemy musieli ręcznie wpisywać
tych danych przy podłączaniu urządzeń do sieci WiFi:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-wps](/img/2016/09/16.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-wps.jpg#huge)

Ciekawą funkcją jest także tryb oszczędzania energii. W jego skład wchodzi dostosowanie mocy
transmisyjnej (mała, średnia, duża) oraz automatyczne wyłączenie WiFi po pewnym czasie od
wylogowania się ostatniego klienta bezprzewodowego. Przy pomocy przycisków mamy możliwość jedynie
włączyć lub wyłączyć ten tryb. Jego konfigurację trzeba przeprowadzić przez panel administracyjny:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-power-saving](/img/2016/09/17.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-power-saving.jpg#huge)

Jako, że ten router ma wbudowany modem LTE, to możemy także wybrać jego tryb pracy. Mamy do wyboru
wymuszony LTE, wymuszony 3G oraz preferowany LTE nad 3G:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-tryb-lte-3g](/img/2016/09/18.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-tryb-lte-3g.jpg#huge)

Możliwe jest także włączenie lub wyłączenie roamingu:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-roaming](/img/2016/09/19.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-roaming.jpg#huge)

I ostatnia rzecz, którą możemy zrobić za pomocą przycisków routera, to wyświetlenie kodu QR w celu
zeskanowania go i pobrania aplikacji tpMiFi na smartfona, która umożliwi nam zdalne zarządzanie
urządzeniem:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-tpmifi-app-kod-qr](/img/2016/09/20.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-tpmifi-app-kod-qr.jpg#huge)

Warto w tym miejscu zaznaczyć, że routerem można także sterować za pomocą każdego dowolnego
komputera, który jest w stanie się podłączyć do routera M7310. To urządzenie ma standardowy panel
administracyjny dostępny po HTTP. Znajduje się on pod adresem `http://192.168.0.1/` .

## Sterowanie M7310 przez webowy panel administracyjny

Podłączamy komputer do sieci WiFi. Następnie odpalamy przeglądarkę i przechodzimy na adres
`http://192.168.0.1/` . Naszym oczom powinna się pokazać poniższa strona:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-logowanie](/img/2016/09/21.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-logowanie.png#huge)

Podajemy hasło logowania `admin` i powinniśmy zostać zalogowani do panelu administracyjnego, gdzie
możemy przeprowadzać nieco więcej działań niż w przypadku kontroli M7310 przy pomocy przycisków na
jego obudowie:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-status](/img/2016/09/22.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-status.png#huge)

Poza statusem połączenia LTE/3G oraz WiFi i statystykami pobranych/wysłanych danych, mamy także
możliwość odczytu i tworzenia SMS, co jest bardzo użyteczną rzeczą:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-sms](/img/2016/09/23.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-sms.png#huge)

Oczywiście nie musimy logować się co chwilę do panelu admina, by sprawdzić czy ktoś nam nie przysłał
czasem jakiegoś SMS'a. Na tym malutkim ekranie co jest na routerze, możemy odczytać status SMS. Może
nie mamy możliwość podejrzeć samej treści komunikatu ale informację o tym, że jakaś wiadomość zalega
na skrzynce jest jak najbardziej widoczna:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-sms](/img/2016/09/24.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-sms.jpg#huge)

Niemniej jednak nie mamy żadnego powiadomienia o zdarzeniu, przez co musimy zerkać co chwila na
ekran. Choć i tak jest to lepsza sytuacja niż logowanie się do panelu administracyjnego i tam
sprawdzać czy ktoś nam wysłał SMS. Może kiedyś w przyszłości doczekamy się jakiegoś dźwięku czy
wibracji po odebraniu SMS w tego typu urządzeniach.

Dalej w panelu mamy możliwość dostosowania całej gamy ustawień, wliczając w to wszystkie funkcje,
które byliśmy w stanie zmienić przez przyciski na routerze. Z tych ważniejszych rzeczy warto
wymienić dostosowanie mocy transmisyjnej nadajnika. Do wyboru są trzy wartości: mała, średnia i
duża. Im wyższą wartość ustawimy tutaj, tym z dalszej odległości będziemy w stanie się połączyć do
routera ale też tym szybciej będzie nam siadać bateria. Możemy także dostosować sobie czas
automatycznego wyłączenia WiFi. Inną ciekawą funkcją jest udostępnianie zasobów karty SD w sieci
WiFi. Standardowo te zasoby są dostępne jedynie po podłączeniu urządzenia do portu USB komputera.
Jeśli aktywujemy sobie taką funkcję, to dostęp do plików będziemy mogli uzyskać za pomocą protokołu
FTP ( `ftp://192.168.0.1/` ) oraz SMB ( `smb://192.168.0.1/` ).

![mobilny-router-wifi-lte-hotspot-M7310-karta-sd-www](/img/2016/09/25.mobilny-router-wifi-lte-hotspot-M7310-karta-sd-www.png#huge)

Pozostałe opcje są raczej standardowe dla wszystkich routerów WiFi i nie ma raczej co się nad nimi
rozwodzić.

### Konfiguracja połączenia LTE/3G przez panel admina

Najważniejszą jednak rzeczą jest konfiguracja połączenia LTE/3G. W panelu administracyjnym mamy
oczywiście zakładkę `Quick Setup` , dzięki której takie połączenie jesteśmy w stanie zestawić w
mniej niż minutę. Wystarczy podać dane APN operatora GSM, z którego usług korzystamy:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-apn-gsm](/img/2016/09/26.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-apn-gsm.png#huge)

I to w zasadzie cała robota. Pamiętajmy też o tym, by ewentualnie dostosować sobie także tryb pracy
modemu LTE tak, by np. wymusić LTE bez możliwości przełączenia w tryb 3G.

Jeśli nasza karta SIM jest chroniona kodem PIN, to by uzyskać połączenie musimy ten kod PIN
uwzględnić w panelu administracyjnym:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-pin-sim](/img/2016/09/27.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-pin-sim.png#huge)

Jeśli obawiamy się, że możemy przekroczyć limit danych oraz, że może nas to słono kosztować, to
zawsze możemy ustawić sobie limit transferu danych oraz ostrzeżenie w przypadku zbliżania się do
tego limitu. Po jego osiągnięciu zaś, M7310 odetnie nam automatycznie internet:

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-limit-danych](/img/2016/09/28.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-limit-danych.png#huge)

Jak widzimy, TP-LINK pomyślał też o darmowych godzinach, w których operatorzy GSM zwykle nie
naliczają transferu danych. Możemy je sobie dowolnie dostosować. I to w zasadzie cała konfiguracja
połączenia LTE na routerze M7310.

## Sterowanie M7310 przy pomocy aplikacji tpMiFi

Jeśli dysponujemy smartfonem, to możemy pobrać [aplikację
tpMiFi](https://play.google.com/store/apps/details?id=com.tplink.tpmifi&hl=pl) dla Androida i przy
jej pomocy skonfigurować sobie pracę routera M7310.

![mobilny-router-wifi-lte-hotspot-M7310-tpmifi](/img/2016/09/29.mobilny-router-wifi-lte-hotspot-M7310-tpmifi.png#medium)

Mając zainstalowaną aplikację możemy sterować hotspotem podobnie jak to można robić z poziomu panelu
www. Niemniej jednak, ta aplikacja nie może się równać funkcjonalnością z panelem administracyjnym.
Choć wszystkie te bardziej użyteczne funkcje jak, np. dostosowanie APN operatorów GSM zostały
zaimplementowane.

![mobilny-router-wifi-lte-hotspot-M7310-tpmifi-status](/img/2016/09/30.mobilny-router-wifi-lte-hotspot-M7310-tpmifi-status.png#medium)

Myślałem jednak, że może chociaż przy pomocy tej aplikacji da radę jakoś powiadomić użytkownika
telefonu, że router odebrał SMS ale nic z tego, szkoda.

## Obsługa Aero2 przez M7310

M7310 jest w stanie obsługiwać darmowy internet od Aero2. Trzeba tylko przełączyć urządzenie w tryb
`Prefer LTE` i włączyć roaming. Dodatkowo musimy ustawić parametry połączenia z siecią. Nie damy
rady tego zrobić przez przyciski na routerze. Ta opcja jest dostępna tylko w panelu administracyjnym
(ewentualnie przez aplikację tpMiFi), do którego możemy wbić z każdego komputera po podłączeniu się,
np. przez WiFi. Jeśli byśmy spróbowali się podłączyć z marszu do sieci Aero2, to przywita nas taki
komunikat na wyświetlaczu M7310:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-aero2-blad](/img/2016/09/31.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-aero2-blad.jpg#huge)

By zmienić ten stan rzeczy, logujemy się do panelu administracyjnego i przechodzimy na zakładkę
Wizard. Wybieramy strefę czasową i przechodzimy na `Dial-up Settings` . Tutaj z kolei dodajemy nowy
APN i uzupełniamy dane w taki sposób, by APN przyjął wartość `darmowy` , a login i hasło
`aero`/`aero` :

![mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-aero2](/img/2016/09/32.mobilny-router-wifi-lte-hotspot-M7310-panel-administracyjny-aero2.png#huge)

Po zapisaniu powinniśmy zostać automatycznie podłączeni do sieci Aero2, co możemy odczytać z
wyświetlacza routera:

![mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-aero2-polaczenie](/img/2016/09/33.mobilny-router-wifi-lte-hotspot-M7310-wyswietlacz-aero2-polaczenie.jpg#huge)

Transfer jak widać wyżej jest w granicach 62 KiB/s, co daje około 500 kbit/s, czyli mniej więcej
tyle ile oferuje usługa darmowy internet od tego operatora. Trzeba jednak też pamiętać o kodzie
CAPTCHA, który będziemy musieli wpisywać co godzinę ale o tym fakcie powinniśmy zostać powiadomieni
podczas surfowania po internecie.

## Podłączenie linux'owego klienta przewodowego

Na opakowaniu M7310 znajduje się informacja, że ten router jest w stanie zapewnić połączenie
przewodowe jednemu komputerowi. Trochę się zdziwiłem, bo przecież to urządzenie nie ma żadnego portu
ethernet. Okazuje się, że to połączenie jest realizowane przez przewód USB. Po podłączeniu routera
do portu USB komputera, w logu systemowym pojawiają się takie oto komunikaty:

    kernel: usb 2-1.3: new high-speed USB device number 7 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=000a
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: M7310
    kernel: usb 2-1.3: Manufacturer: TP-LINK Technologies Co., Ltd.
    kernel: usb 2-1.3: SerialNumber: ef200a715739
    kernel: usb-storage 2-1.3:1.2: USB Mass Storage device detected
    kernel: scsi host4: usb-storage 2-1.3:1.2
    kernel: usbcore: registered new interface driver cdc_ether
    kernel: rndis_host 2-1.3:1.0 usb0: register 'rndis_host' at usb-0000:00:1d.0-1.3, RNDIS device, ee:3f:49:c2:d5:72
    kernel: usbcore: registered new interface driver rndis_host
    kernel: usb 2-1.3: USB disconnect, device number 7
    kernel: rndis_host 2-1.3:1.0 usb0: unregister 'rndis_host' usb-0000:00:1d.0-1.3, RNDIS device

Po chwili pojawia się także coś takiego:

    kernel: usb 2-1.3: new high-speed USB device number 8 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0006
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: M7310
    kernel: usb 2-1.3: Manufacturer: TP-LINK Technologies Co., Ltd.
    kernel: usb 2-1.3: SerialNumber: ef200a715739
    kernel: rndis_host 2-1.3:1.0 usb0: register 'rndis_host' at usb-0000:00:1d.0-1.3, RNDIS device, 46:9c:a0:4b:77:26

W tym momencie właśnie został wykryty interfejs sieciowy `usb0` , który można skonfigurować sobie
jak każdy inny interfejs przewodowy. Przepustowość tego interfejsu to 100 mbit/s, w końcu USB 2.0.
Po tych powyższych komunikatach, w logu pojawia się jeszcze poniższe wiadomości:

    kernel: usb-storage 2-1.3:1.2: USB Mass Storage device detected
    kernel: scsi host4: usb-storage 2-1.3:1.2
    kernel: scsi 4:0:0:0: Direct-Access     TP-LINK  MMC Storage           PQ: 0 ANSI: 2
    kernel: sd 4:0:0:0: Attached scsi generic sg2 type 0
    kernel: sd 4:0:0:0: [sdb] 3852288 512-byte logical blocks: (1.97 GB/1.84 GiB)
    kernel: sd 4:0:0:0: [sdb] Write Protect is off
    kernel: sd 4:0:0:0: [sdb] Mode Sense: 0f 00 00 00
    kernel: sd 4:0:0:0: [sdb] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
    kernel:  sdb:
    kernel: sd 4:0:0:0: [sdb] Attached SCSI removable disk

I to jest karta SD w routerze, którą możemy zamontować w systemie bez większego problemu.

## Czas pracy na baterii

TP-LINK deklaruje, że router M7310 jest w stanie pracować do 10 godzin na dołączonym do zestawu
akumulatorze (w trybie czuwania do 600 godzin). Pytanie tylko jak liczony jest ten czas. Raczej
wątpię by to urządzenie tyle pociągnęło na tych 2000 mAh przy podłączonych 10 klientach WiFi i
korzystaniu z internetu 50 mbit/s w technologi LTE. Postanowiłem jednak sprawdzić jak wyglądają te
osiągi, bo bardzo mnie ta kwestia interesowała.

Zanim jednak przejdziemy do faktycznego czasu rozładowania akumulatora, warto wspomnieć, że by go
naładować od 0% do 100%, proces ładowania musi trwać około 2,5 godziny, oczywiście przy założeniu,
że będziemy ten router ładować dedykowaną ładowarką 5V/1A, której standardowo nie ma w zestawie
oraz że router nie będzie w tym czasie wykorzystywany do połączenia z internetem. Jeśli zamierzamy
ładować ten router z portu USB komputera, to trzeba brać drobną poprawkę, bo port USB w wersji 2.0
ma maksymalną wydajność prądową na poziomie 0,5 A, przez co ładowanie może ulec znacznemu
wydłużeniu.

Czas rozładowania akumulatora M7310 przy standardowej pracy tego routera jest w granicach 8 i pół
godziny. Standardowa praca rozumiana jest jako podpięte 4 urządzenia po WiFi (2x laptop, 2x
smartfon), a aktywność w sieci sprowadza się jedynie do przeglądania stron internetowych w
przeglądarce www i ewentualnie pobierania małych plików/filmów. W sumie transfer danych przekroczył
nico 4 GiB przez te 8,5 godziny. Jeśli zamierzamy intensywnie pobierać dane, to trzeba liczyć się ze
skróceniem czasu pracy routera na baterii. Dlatego też dobrze jest mieć ze sobą zawsze jakiś power
bank.

Warto też zaznaczyć, że wskazania na wyświetlaczu co do stanu baterii nie są wiarygodne. Router
potrafi przez 2 godziny mieć 100%, a przy 20% zjechać do 0 w jakieś 20 minut. Podobnie z ładowaniem
baterii, gdzie przez 10 minut można ją niby naładować do poziomu 50%.

## Problemy z wydajnością

Podczas przeprowadzania testów hotspotu M7310 napotkałem pewne problemy z połączeniem. U mnie na
linux w logu systemowym można dostrzec tego typu
    komunikaty:

    kernel: wlan1: AP ec:08:6b:00:fb:b0 changed bandwidth, new config is 2462 MHz, width 1 (2462/0 MHz)
    kernel: wlan1: AP ec:08:6b:00:fb:b0 changed bandwidth in a way we can't support - disconnect
    kernel: wlan1: failed to follow AP ec:08:6b:00:fb:b0 bandwidth change, disconnect
    kernel: wlan1: authenticate with ec:08:6b:00:fb:b0
    kernel: wlan1: send auth to ec:08:6b:00:fb:b0 (try 1/3)
    kernel: wlan1: authenticated
    kernel: wlan1: associate with ec:08:6b:00:fb:b0 (try 1/3)
    kernel: wlan1: RX AssocResp from ec:08:6b:00:fb:b0 (capab=0x411 status=0 aid=1)
    kernel: wlan1: AP has invalid WMM params (AIFSN=1 for ACI 0), will use 2
    kernel: wlan1: AP has invalid WMM params (AIFSN=1 for ACI 2), will use 2
    kernel: wlan1: AP has invalid WMM params (AIFSN=1 for ACI 3), will use 2
    kernel: wlan1: associated

Nie wiem dokładnie o co chodzi ale ten problem występuje tylko w przypadku AP na routerze M7310.
Próba wymiany danych między dwiema maszynami (smartfon i laptop) w sieci WiFi powoduje pojawienie
się takiego komunikatu co kilkanaście sekund i klient jest rozłączany. Może i łączy się
automatycznie za chwilę ale transfer danych w sieci jest wręcz katastrofalnie niski 5-8 mbit/s z
ciągłymi spadkami prędkości do 0. Nie mam pojęcia co jest przyczyną takiego stanu rzeczy ale nie
wszystkie karty WiFi się w ten sposób zachowują. Ta wyżej jest na czipie od Qualcomm Atheros
(TL-WN722N).

Co do pozostałych kart, to sytuacja jest mniej więcej standardowa. Trzeba jednak pamiętać, że skoro
w przypadku tego routera komunikacja odbywa się głównie za pomocą WiFi, to faktyczny transfer jest
rozdzielony między dwóch klientów mniej więcej po połowie. Zatem zakładając, że w okolicy mamy inne
sieci WiFi, to M7310 może mieć problemy z przełączeniem się na kanał 40 MHz. Stracimy zatem połowę
przepustowości i z obiecanych 300 mbit/s zostanie nam 150 mbit/s. Do tego realny transfer jest na
poziomie 66% i tak zostaje nam 100 mbit/s do rozdysponowania w sieci WiFi. Zakładając, że przesyłamy
dane między dwoma klientami w tej sieci, to maksymalnie osiągniemy 50 mbit/s. Oczywiście przy
niewielkich zakłóceniach. W dziczy te osiągi powinny być 2x lepsze.

Połączenie LAN =\> WAN (LTE) też mogłoby być nieco bardziej przyzwoite. W tym przypadku nie udało mi
się wyjść poza granicę 30 mbit/s. Wiem, że transfer po LTE zależy głównie od lokalizacji i
obciążenia BTS ale w moim przypadku BTS jest dość blisko (\< 900 m, widoczność zachowana), a test
był przeprowadzany w godzinach 2-3 w nocy, z tym, że w budynku. Dla porównania, przekładając kartę
SIM z routera M7310 do smartfona Neffos C5, transfer LTE jest na poziomie 42 mbit/s:

![mobilny-router-wifi-lte-hotspot-M7310-test-predkosc-lte-wifi](/img/2016/09/34.mobilny-router-wifi-lte-hotspot-M7310-test-predkosc-lte-wifi.png#medium)

Trzy pierwsze wyniki dotyczą połączenia przez router M7310, a dwa ostatnie przez Neffos'a C5.
Różnica jest widoczna gołym okiem. Trochę daleko do tych obiecanych 150/50 mbit/s, a za dnia może
być tylko gorzej.

## Stacjonarne podłączenie routera M7310

Może i ten hotspot M7310 jest mobilny ale biorąc pod uwagę fakt, że jest on także w stanie zapewnić
połączenie dziesięciu komputerom po WiFi i jednemu po USB, to możemy taki router postawić u siebie w
domu i wykorzystywać go jako odpowiednik tych stacjonarnych routerów LTE. Niemniej jednak, trzeba
liczyć się z faktem, że M7310 nigdy nie dorówna osiągom tradycyjnym routerom.

Osobną kwestią jest sprawa zasilania M7310. W zestawie jest dołączony tylko przewód USB. Nie ma
ładowarki. Jeśli chcielibyśmy się bawić w zastosowania stacjonarne, to trzeba by dokupić ładowarkę
5V/1A i przy jej pomocy podłączyć M7310 do sieci elektrycznej w domu. Oczywiście nic nie stoi na
przeszkodzie, by korzystać z portu USB komputera, z tym, że wtedy pokrycie zasięgiem WiFi może nie
być optymalne.

To urządzenie nie potrafi pracować bez baterii ale nie ma z nim problemów przy realizacji połączenia
internetowego, gdy jest ono jednocześnie ładowane z gniazdka. Dlatego też może ono robić za
stacjonarny router LTE, choć nie jest to zbyt opłacalne przedsięwzięcie.
