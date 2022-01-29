---
author: Morfik
categories:
- Android
date:    2021-09-21 19:33:00 +0200
lastmod: 2021-09-21 19:33:00 +0200
published: true
status: publish
tags:
- smartfon
- aplikacje
- f-droid
- lokalizacja
- gps
- wifi
- bluetooth
- prywatność
GHissueID: 326
title: Pozorowanie lokalizacji GPS w Androidzie (mock location)
---

Wiele osób nie pozostawia suchej nitki na mechanizmie geolokalizacji zaszytym w smartfonach z
Androidem na pokładzie. No i faktycznie w sporej części przypadków zastrzeżenia jakie są kierowane
pod adresem lokalizacji/GPS w kwestii permanentnego śledzenia nas przez Google nie są przesadzone.
Niektórzy spierają się, że wystarczy włączyć tryb samolotowy (Airplane Mode) w telefonie i nasza
pozycja przestaje być rejestrowana w czasie rzeczywistym ale nie jest to prawdą. Jakiś czas temu
świat obiegła informacja, że [Android rejestruje dane geolokalizacji][1] nawet, gdy tryb samolotowy
jest włączony, a telefon nie ma włożonych kart SIM i to przy jednoczesnym wyłączeniu WiFi i BT. Jak
to do końca jest z tą lokalizacją i czy faktycznie nie da się jej wyłączyć w Androidzie? A może
powinniśmy pójść w drugą stronę i zamiast starać się wyłączać lokalizację, to spróbować oszukać
system przez jej pozorowanie? Taki zabieg jest możliwy ale wymagana jest zewnętrzna aplikacja do
pozorowania lokalizacji, np. [Fake Traveler][2], którą trzeba określić w ustawieniach
deweloperskich Androida. Czy ten zabieg wpłynie pozytywnie na lokalizację w kontekście naszej
prywatności? Czy może lepiej jest jednak zostawić telefon w domu i pójść na niejawne spotkanie bez
tego urządzenia?

<!--more-->
## Zasada działania GPS/A-GPS

By [GPS][6] mógł spełniać swoje zadanie, potrzebne nam są nadajniki i odbiorniki (oraz po części
też jakaś infrastruktura zarządzająca). Nadajnikami są satelity latające po orbicie Ziemi (ok 20km),
a odbiornikami zaś są różne urządzenia na Ziemni, np. nasze smartfony. Technologia GPS nie wymaga od
naszych telefonów przesyłania jakichkolwiek informacji do satelitów -- wyobraźmy sobie setki
milionów czy miliardy urządzeń chcących komunikować się z kilkudziesięcioma satelitami praktycznie
w tym samym czasie. Nie ma opcji, by można było zapewnić tak ogromną przepustowość dla całej rzeszy
urządzeń chcących korzystać z GPS. Dlatego to właśnie jedynie satelity nadają sygnał, który
odbiorniki GPS w naszych telefonach są w stanie odebrać i odpowiednio zinterpretować.

Każdy satelita jest wyposażony w kilka dość precyzyjnych zegarów atomowych, dzięki którym jest w
stanie wysłać sygnał zawierający bardzo dokładny czas. Ten sygnał jest wysyłany nieprzerwanie i
odbiornik GPS na jego podstawie jest w stanie ustalić położenie konkretnych satelitów na niebie w
chwili wysłania takiego sygnału. Znajomość dokładnego położenia satelitów jest kluczowa, by system
GPS podał bardzo dokładną lokalizację samego odbiornika.

Układ satelitów na orbicie (konstelacja) jest tak zaprojektowany, by z każdego miejsca na Ziemi w
dowolnym czasie były widoczne co najmniej 4 satelity, choć w praktyce zwykle widać ich więcej, a
każdy dodatkowy satelita jest w stanie wpłynąć na dokładniejszy pomiar czasu, a przez to dokładniej
wskazać lokalizację odbiornika GPS.

Do ustalenia położenia danego odbiornika w przestrzeni trójwymiarowej potrzebne są dane z co
najmniej czterech satelitów i wykorzystuje się do tego celu [trilaterację][7] (często myloną lub
zamiennie stosowaną z terminem [triangulacja][8] ale to nie to samo). Czemu zatem cztery satelity,
a nie trzy? Trzeba ustalić cztery zmienne: długość (x), szerokość (y), wysokość (z) oraz czas (t).

Gdybyśmy szukali odbiornika GPS na poziomie morza, wtedy wysokość w tym trójwymiarowym modelu (z)
byłaby znana i do ustalenia pozycji GPS [wystarczyłyby tylko trzy satelity][9]. Jako, że Ziemia
jest bardziej trójwymiarowa i zwykle musimy znać dokładną wysokość nad poziomem morza, to potrzebny
nam jest dodatkowo czwarty satelita do poprawnego wyliczenia wysokości, na której znajduje się
odbiornik GPS.

Gdyby odbiornik GPS był wyposażony w zegar atomowy, tak jak to ma miejsce w przypadku satelitów, to
w takiej sytuacji również do wyznaczenia położenia potrzebne byłyby jedynie trzy satelity, bo czas
byłby znany. Niestety, zegar atomowy to wydatek na poziomie 100K dolców i znacząco by podniósł
koszt odbiornika GPS, dlatego też w odbiornikach GPS stosuje się zwykły zegar na bazie kwarcu,
który jest sporo tańszy ale też bardzo niedokładny, przynajmniej w stosunku do swojego atomowego
kuzyna.

Do tego wszystkiego dochodzą jeszcze problemy wynikające z [Teorii Względności Einstein'a][11],
gdzie grawitacja wpływa na tempo upływu czasu obiektów na Ziemi i tych latających na orbicie około
20K km nad nią, oraz [dylatacja czasu][10] z racji różnic prędkości poruszających się dwóch obiektów
względem siebie, no i też jeszcze różnice czasu wynikające z dobowego ruchu obrotowego Ziemi, ruchu
orbitalnego satelitów (efekt Sagnac'a), jak i dobowego obrotu pola magnetycznego Ziemi. Tak czy
inaczej, system GPS bez stosownych poprawek czasu nie byłby za wiele warty, bo dzienne różnice w
czasie są [na poziomie 38us/dobę][20] (między zegarami na orbicie i tymi na Ziemi), co sprawia, że
rozrzut odległości położenia odbiornika GPS względem jego faktycznego miejsca pobytu jest na
poziomie większym niż 10km. Niemniej jednak, istnieją stosowne mechanizmy korekcji czasu, które do
tak drastycznych rozbieżności nie dopuszczają i ciągle ten czas korygują, co sprawia, że GPS może
spełniać z powodzeniem swoje funkcje.

Technologia GPS do najszybszych nie należy i to pomimo faktu, że fale radiowe biegną z prędkością
światła (przynajmniej w próżni). By nie nadwyrężać baterii w satelitach, transfer danych z satelity
jest na poziomie 50 bitów na sekundę. Jeśli zatem byśmy korzystali jedynie z technologi GPS do
ustalenia naszego położenia, to uzyskanie pierwszego fix'a mogłoby nam zająć bardzo dużo czasu
(kilkanaście do kilkudziesięciu minut).

Generalnie to komunikat nawigacyjny wysyłany z satelity składa się z 30-sekundowych ramek o
długości 1500 bitów (50 bitów/s * 30s). Pojedyncza ramka podzielona jest na pięć 6-sekundowych
podramek, a w skład każdej z nich wchodzi dziesięć słów 30-bitowych. Każda podramka zawiera czas
GPS w odstępach 6-sekundowych. Podramka nr. 1 zawiera datę GPS (numer tygodnia) oraz informacje o
korekcji zegara satelitarnego, statusie i stanie satelity. Podramki nr. 2 i 3 łącznie zawierają
dane dotyczące położenia nad Ziemią satelity nadającego (efemerydy). Podramki nr. 4 i 5 zawierają
strony almanachu, tj. informacje o ogólnym położeniu wszystkich satelitów w konstelacji. Almanach
ma długość 15.000 bitów, przez co nie jest w stanie zmieścić się cały w jednym
komunikacie -- potrzeba 25 takich komunikatów, by przesłać do nadajnika wszystkie części almanachu,
przez co jego transmisja trwa 12,5 minuty (25 komunikatów * 30 sekund).

Jakby nie patrzeć to dość długi okres na poznanie dokładnej lokalizacji GPS, oczywiście zakładając,
że nic nie zakłóci sygnału, bo wtedy całość trzeba będzie pobierać od nowa, co jeszcze dodatkowo
wydłuży czas oczekiwania na pierwszego fix'a.

Dlatego też stworzono A-GPS ([Assisted GPS][5]), który to jest w stanie te wszystkie dane o
położeniu satelitów przesłać nam na nasze żądanie wykorzystując do tego celu czy to naziemne anteny
(GSM, CDMA, WCDMA, LTE) czy też korzystając z połączenia WiFi -- w skrócie potrzebny nam jest do
tego celu internet lub też infrastruktura operatora komórkowego, która sama w sobie może wspierać
A-GPS, przez co będzie buforować dane GPS (te, które byśmy pobrali bezpośrednio od satelitów).

Gdy mamy odbiornik GPS, który jest w stanie obsłużyć technologię A-GPS, to nasze urządzenie prześle
zapytanie do stosownych serwerów prosząc o dane z położeniem satelitów, które w bardzo krótkim
czasie zostaną mu przesłane i praktycznie natychmiast GPS złapie fix'a. O ile sama technologia GPS
nie wymagała od nas wysyłania żadnych danych z naszego telefonu w świat, o tyle A-GPS wymaga od nas
przesłania zapytań czy to via modem GSM, czy też po WiFi. Ta informacja przyda nam się później.

## Ustawienia lokalizacji w Androidzie

Każdy smartfon z Androidem jest standardowo wyposażony w odbiornik GPS, który wspiera A-GPS. W
systemie mamy szereg opcji, które mogą posłużyć do konfiguracji pracy tego odbiornika. Przede
wszystkim, w ustawieniach Androida mamy zwykle osobną pozycję dla lokalizacji:

|   |   |
|---|---|
| ![android-mock-fake-location-gps-settings](/img/2021/09/001.android-mock-fake-location-gps-settings.jpg#small) | ![android-mock-fake-location-gps-settings](/img/2021/09/002.android-mock-fake-location-gps-settings.jpg#small) |

To tutaj włączany jest odbiornik GPS, który umożliwia różnym aplikacjom korzystanie z danych
lokalizacji. Warto zauważyć tutaj, że po włączeniu lokalizacji w tym okienku powyżej, na pasku
statusu nie pojawi się żadna ikonka sugerująca, że odbiornik GPS działa. Może to być mylące i
sprawiać wrażenie, że GPS jest wyłączony, a tak wcale nie jest. Jeśli mamy włączoną lokalizację,
to moduł GPS zaszyty w naszym smartfonie odbiera sygnał GPS z satelitów i po cichu go ignoruje,
przynajmniej powinien, a w praktyce to różnie bywa. Ikonka lokalizacji na pasku statusu pojawi się
dopiero wtedy, gdy jakaś aplikacja użytkownika będzie chciała skorzystać z modułu GPS:

![android-mock-fake-location-gps-notification](/img/2021/09/003.android-mock-fake-location-gps-notification.jpg#small)

Także pamiętajmy, że nawet jeśli nie widzimy tej ikonki lokalizacji na pasku statusu, to
niekoniecznie oznacza to, że odbiornik GPS jest nieaktywny i Android z niego nie korzysta, bo
zwykle jest wręcz odwrotnie.

### Skanowanie Bluetooth i WiFi

Do końca nie wiadome jest jak działa skanowanie Bluetooth i WiFi. Niektórzy piszą, że te opcje
znajdują jedynie zastosowanie, gdy GPS jest wyłączony. W takim przypadku Android może być w stanie
bardzo orientacyjnie określić położenie urządzenia nawet bez podłączania go do internetu.

![android-mock-fake-location-gps-wifi-bluetooth-scan](/img/2021/09/007.android-mock-fake-location-gps-wifi-bluetooth-scan.jpg#small)

Kiedyś Google Street View zbierał informacje na temat publicznie dostępnych AP (ich ESSID oraz MAC).
Znając lokalizację samochodu Google, można było powiązać AP z konkretną lokalizacją i ustalić
położenie telefonu, choć jest to jedynie przybliżona lokalizacja, a nie dokładne wskazanie, tak jak
w przypadku GPS. Jednak od wielu lat [Google nie używa usługi Street View do mapowania punktów
dostępowych WiFi][12]. Robimy to za niego my sami przy pomocy naszych telefonów, o ile mamy
włączone w ustawieniach lokalizacji Google Location Accuracy (o tym za moment).

Zdaniem niektórych, włączenie skanowania BT czy WiFi poprawia dokładność GPS, choć nie wiadomo w
jaki sposób. Przejrzałem sporo opinii i wychodzi na to, że nikt nie widzi żadnej poprawy z
włączonym skanowaniem w stosunku do jego braku (albo te różnice są tak niewielkie, że się ich nie
da realnie stwierdzić).

Jedno jest pewne, takie skanowanie w tle sprawia, że radio WiFi i BT wykazują się jakąś aktywnością,
zatem zjadają prąd, przez co cierpi na tym nasza bateria. Trzeba też wziąć pod uwagę fakt, że
skanowanie za sieciami WiFi w okolicy (czy też za urządzeniami BT) skutkuje zbieraniem informacji i
przesyłaniem ich do Google, co naturalnie zagraża naszej prywatności.

#### Automatyczne podłączanie do sieci WiFi

W ustawieniach sieci WiFi jest kilka dodatkowych funkcji, na które wypadałoby rzucić okiem. Mamy
tam m.in. skanowanie w poszukiwaniu znanych/zapisanych sieci WiFi (przy włączonym radiu), czyli
tzw. autoconnect:

![android-mock-fake-location-gps-wifi-auto-connect](/img/2021/09/008.android-mock-fake-location-gps-wifi-auto-connect.jpg#small)

Ten mechanizm działa na takiej zasadzie, że jeśli nie wyłączymy radia WiFi w naszym telefonie, to
ten co chwilę skanuje eter, by ustalić czy jakieś znane mu sieci WiFi są dostępne, a jeśli tak, to
łączy się do nich z automatu.

Z jednej strony jest to dość użyteczna funkcja, bo po przyjściu do domu nie trzeba włączać telefonu
i klikać na ikonkę WiFi by uzyskać połączenie z internetem ale takie zachowanie ma też sporo wad,
przynajmniej jeśli chodzi o naszą prywatność. Głównym problemem jest tutaj nadajnik, który co
chwila szuka określonej sieci WiFi. Do tego skanuje non stop wszystkie sieci WiFi na drodze naszego
poruszania się i rejestruje przy tym znajdujące się w naszym pobliżu AP. Z tych danych Google też
robi użytek.

Kolejna sprawa to powiadomienie o publicznych sieciach WiFi oraz ich rankingi:

![android-mock-fake-location-gps-wifi-auto-on-public-notif](/img/2021/09/009.android-mock-fake-location-gps-wifi-auto-on-public-notif.jpg#small)

Te ustawienia również są w stanie pomóc w ustaleniu naszego położenia. No i też potrafią
skonsumować jakiś procent baterii. Jeśli zależy nam na prywatności i przy tym na jak najdłuższej
pracy telefonu na baterii, to te powyższe opcje powinniśmy powyłączać.

Jeśli jednak z jakiegoś powodu potrzebujemy tego skanowania za sieciami WiFi, to powinniśmy tez
rzucić okiem w ustawienia deweloperskie i zainteresować się przełącznikiem `WiFi scan throttling` .

![android-mock-fake-location-gps-wifi-scan-throttling](/img/2021/09/010.android-mock-fake-location-gps-wifi-scan-throttling.jpg#small)

Gdy to [dławienie skanowania WiFi][13] jest włączone, to aplikacje działające w tle będą mogły
dokonywać skanowania jedynie raz na 30 minut. Wciąż jednak będą mogły to robić, choć sporo rzadziej
(30-krotnie) niż ma to miejsce w przypadku, gdy ta opcja nie jest włączona. Choć ja jestem zdania,
że skanowanie za sieciami WiFi powinniśmy wyłączyć kompletnie.

### Google Location Accuracy/Services

Usługa Google Location Accuracy (znana też jako Google Location Services) ma na celu poprawić
dokładność lokalizacji poprzez zaprzęgnięcie do tego celu dodatkowych mechanizmów, takich jak
sieci GSM, WiFi czy wskazania sensorów smartfona.

![android-mock-fake-location-gps-google-accuracy](/img/2021/09/014.android-mock-fake-location-gps-google-accuracy.jpg#small)

[Z informacji które znajdują się tutaj][12] wynika, że ta usługa działa na zasadzie skanowania i
rejestrowania komórek operatora GSM (Cell-ID) oraz publicznych sieci WiFi (ESSID, MAC) i
przesyłania tych danych do Google. Proces skanowania jest przeprowadzany w tle i nie trzeba
aktywnie korzystać z nawigacji czy innych aplikacji, które próbują ustalić nasze położenie. Zatem
jeśli ta opcja zostanie zaznaczona, to Google będzie miał dostęp do naszej lokalizacji praktycznie
non stop.

W przypadku, gdybyśmy tę opcję wyłączyli, to naturalnie odetniemy Google od tych dość wrażliwych
informacji ale wtedy tylko dane z odbiornika GPS będą wykorzystywane do ustalenia naszej
lokalizacji, co może zająć sporo czasu, zwłaszcza w przypadku pierwszego fix'a po dłuższej
nieaktywności modułu GPS.

### Emergency Location Service (ELS)

Kolejną pozycją w zakładce lokalizacji jest Emergency Location Service (ELS). To taka usługa, która
ma na celu przesłać dane z GPS z telefonu do służb ratunkowych po wybraniu numeru alarmowego (w
Europie 112).

![android-mock-fake-location-gps-els](/img/2021/09/011.android-mock-fake-location-gps-els.jpg#small)

Jak można [przeczytać na stronie Google][14], ten cały ELS jest wbudowany w Androida (Google Play
Services), no i ma m.in. dostęp do naszej lokalizacji praktycznie non stop. Jasne, że w przypadku
zagrożenia życia, taki ELS może nam się przydać. Jednak Google już wielokrotnie udowodnił nam, że w
kwestii prywatności nie ma co mu ufać i jest wielce prawdopodobne, że ELS będzie wykorzystywany nie
tylko zgodnie z jego przeznaczeniem, tj. po wybraniu numeru 112. Idę o zakład, że na pewnym etapie
ta usługa będzie sobie działała w tle i monitorowała nas na wypadek, gdybyśmy nie byli w stanie
wybrać numeru 112.

Dlatego wszelkie takie usługi powinniśmy wyłączać z automatu, no chyba, że faktycznie wybieramy się
gdzieś na Everest i mamy dość spore ryzyko załamania się pogody będąc gdzieś w pobliżu jego szczytu.

No i też trzeba pamiętać, że nasz operator GSM zwykle posiada usługę na wzór ELS, która również po
wybraniu numeru alarmowego jest w stanie zgłosić nasze położenie w oparciu o dane z odbiornika GPS
w telefonie lub też podać przybliżoną lokalizację w oparciu o fizyczne rozmieszczenie stacji BTS. W
tym przypadku nie znalazłem żadnej konkretnej opcji, która by i tę usługę była w stanie wyłączyć.

### Historia lokalizacji Google

Kolejna opcja dotyczy zapisywania danych naszej lokalizacji w strukturach Google. Warto tutaj
zaznaczyć, że historia lokalizacji będzie zbierana nawet wtedy, gdy nie korzystamy z usług Google,
tj. nie mamy np. odpalonej nawigacji. Zatem jeśli tylko mamy telefon przy sobie, to będzie on
rejestrował naszą pozycję i przesyłał te dane do Google, tak by każdy chętny mógł się z nimi
zapoznać (oczywiście za naszą świadomą i dobrowolną zgodą).

![android-mock-fake-location-gps-google-history](/img/2021/09/012.android-mock-fake-location-gps-google-history.jpg#small)

Jeśli nie mamy problemów z pamięcią i doskonale potrafimy się odnaleźć w otaczającej nas
rzeczywistości (szczególnie tej realnej), to nie ma praktycznie żadnego powodu, by historię
lokalizacji włączać.

### Udostępnianie lokalizacji w czasie rzeczywistym

Kolejna sprawa dotyczy udostępniania naszego aktualnego położenia w czasie rzeczywistym osobom czy
urządzeniom, które wskażemy. Dzięki tej opcji, każdy (za naszym świadomym przyzwoleniem) może
sprawdzić, gdzie aktualnie się znajdujemy i co robimy, np. czy nie ma nas w domu.

![android-mock-fake-location-gps-google-share](/img/2021/09/013.android-mock-fake-location-gps-google-share.jpg#small)

Na telefonie kochanki (albo jej męża) taka opcja jak najbardziej się przydaje ale nie zalecam jej
włączać u siebie.

## Tryby pracy lokalizacji

W moim Androidzie 11, kafelek lokalizacji skrywa w zasadzie cztery położenia: brak (none), wysoki
(high), tylko sensory (sensors only) oraz oszczędność baterii (battery saving). Czym one się
różnią?

W trybie braku lokalizacji naturalnie mamy wyłączoną lokalizację. Tryb z bardzo wysoką dokładnością
zakłada wykorzystanie wszystkich dostępnych w urządzeniu mechanizmów (GSM, WiFi, BT, sensory/GPS).
Jeśli zaś chodzi o tryb z sensorami, to tutaj jest wykorzystywany jedynie moduł GPS (dość mocno
zjada baterię). W przypadku oszczędności energii, lokalizacja jest ustalana w oparciu o sieci WiFi
i stacje bazowe operatorów komórkowych (BTS) -- może i niezbyt dokładne ale potrafi odciążyć
baterię.

W przypadku mojego ROM'u ten kafelek chyba nie działa za dobrze, bo przestawienie go na różne opcje
(za wyjątkiem wł/wył) nie wpływa na żadne opcje zdefiniowane w ustawieniach Androida. Także nie
wiem czy przestawienie tego kafelka powinno w jakiś bardziej widoczny sposób coś zmienić w
ustawieniach systemu.

## Dostęp aplikacji do lokalizacji

Samo włączenie lokalizacji/GPS nic jeszcze nie oznacza, przynajmniej nie powinno. W Androidzie
każda aplikacja, która ma zamiar korzystać z lokalizacji, musi zgłosić taką chęć użytkownikowi (no
może za wyjątkiem pewnych systemowych appek), a ten może taką prośbę zaakceptować bądź też odrzucić.
Kontrolowanie ustawień lokalizacji dla danej aplikacji odbywa się w informacjach o tej konkretnej
appce:

|   |   |   |
|---|---|---|
| ![android-mock-fake-location-gps-app-permissions](/img/2021/09/004.android-mock-fake-location-gps-app-permissions.jpg#small) | ![android-mock-fake-location-gps-app-permissions](/img/2021/09/005.android-mock-fake-location-gps-app-permissions.jpg#small) | ![android-mock-fake-location-gps-app-permissions](/img/2021/09/006.android-mock-fake-location-gps-app-permissions.jpg#small) |

### ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION oraz ACCESS_BACKGROUND_LOCATION

W starszych Androidach nie było dostępnych niektórych opcji od kontroli dostępu do lokalizacji.
We wcześniejszych wersjach można było albo zezwolić appce na korzystanie z lokalizacji albo
zablokować jej taką możliwość. [W nowszych Andkach][18] możemy kontrolować również czy zezwolić
aplikacji na dostęp do lokalizacji na stałe, czy tylko, gdy appka jest w użyciu (na pierwszym
planie). Jeśli byśmy zezwolili jej na ciągły dostęp, to będzie mogła ona rejestrować naszą
aktywność na bieżąco, o ile oczywiście włączyliśmy lokalizację w ustawieniach Androida.

## Brak kart SIM w telefonie

Nie tylko GPS jest w stanie namierzyć nasze położenie, choć akurat on robi to najdokładniej, o ile
oczywiście nie jesteśmy w miejskiej dżungli wewnątrz kanionu złożonego z wieżowców czy innych
drapaczy chmur, albo wewnątrz żelbetowych budynków pokroju bunkra na Wilczym Szańcu. Sporo osób
zapomina o naziemnych stacjach bazowych operatorów komórkowych (BTS), które rejestrują nasze
urządzenia, gdy te tylko znajdą się w zasięgu takich stacji. Te stacje bazowe posiadają znaną
lokalizację i przybliżony zasięg, zatem jeśli nasz telefon się w takim BTS zaloguje do sieci
operatora GSM, to wiadome jest, że znajdujemy się gdzieś w jego pobliżu. Do tego cały czas działa
roaming i nasz telefon przełącza się między BTS'ami tracąc zasięg u jednej stacji bazowej, wchodząc
jednocześnie w zasięg drugiej (dla uproszczenia pomińmy obciążenie komórek i siłę sygnału). W ten
sposób przemieszczając się z jednego krańca kraju na drugi, zostawiamy za sobą orientacyjny ślad
naszej podróży i miejsc pobytu razem z czasem zatrzymania się w nich (czas podłączenia do BTS w
konkretnej lokalizacji).

Czasami użytkownicy telefonów komórkowych wyjmują karty SIM, by taki smartfon się u danego
operatora GSM nie zalogował (nie podłączył do BTS'a) i by ten operator nie poznał ich przybliżonego
położenia. Taki zabieg wyjęcia karty SIM jest jednak pozbawiony sensu, bo już od dłuższego czasu
operatorzy GSM muszą świadczyć usługę połączeń alarmowych. Z kolei na połączenia alarmowe można
dzwonić ze smartfonów, które [nie posiadają kart SIM czy mają też zablokowany ekran][15]. Zatem
telefon bez kart SIM jest w stanie się bez większego problemu zalogować do sieci operatora
komórkowego (nawet tego, z którym nie mamy podpisanej żadnej umowy). W jaki sposób?

By zalogować się do sieci operatora GSM (i być przy tym w stanie nawiązywać połączenia z innymi
abonentami), nasz [telefon musi podać dwie kluczowe dane][19]: identyfikator urządzenia (IMEI)
oraz identyfikator abonenta (dane z karty SIM). Niemniej jednak, w przypadku numerów alarmowych,
ten drugi czynnik uwierzytelniający nie jest wymagany. Zatem nasze urządzenie może zostać
zaakceptowane przez sieć operatora komórkowego jedynie zgłaszając mu numer IMEI. W takim przypadku,
to urządzenie będzie w stanie wykonywać jedynie połączenia alarmowe, a jeśli chcielibyśmy mieć
możliwość nawiązywania innych połączeń, to potrzebne jest zweryfikowanie abonenta za sprawą danych z
karty SIM.

Tak czy inaczej, nasze urządzenie jest w stanie bez problemu podać operatorowi swoje unikalne
numery identyfikacyjne. Jeśli jeszcze dodatkowo zakupiliśmy to urządzenie płacąc kartą, czy też
zamówiliśmy je wysyłkowo na swój adres zamieszkania, to bez większego problemu można ustalić nasze
dane osobowe (normalnie by zostały wzięte z karty SIM) oraz z mniejszą lub większą granicą błędu
określić, gdzie w danym czasie się znajdowaliśmy (przy założeniu że GPS był wyłączony).

Jeśli zamierzamy uniknąć mapowania naszej lokalizacji przez sieć GSM, to jedynym wyjściem jest
wyłączenie modemu GSM. Ogromna większość smartfonów oferuje jedynie tryb samolotowy do tego celu.
Gdzieniegdzie można też spotkać się z wyłączeniem kart SIM w ustawieniach Androida ale najwyraźniej
od Andka 10 czy 11 ten zabieg powoli przestaje być możliwy, i to nawet w przypadku urządzeniach
wyposażonych w więcej niż jedną kartę SIM.

## Tryb samolotowy (Airplane Mode)

Tryb samolotowy ma na celu wyłączenie w naszym telefonie źródeł fal radiowych, czyli nadajników.
Odbiorników nie ma po co wyłączać, bo w żaden sposób nie wpływają one na generowanie fal
radiowych -- odbiorniki je tylko odbierają. Niemniej jednak, każdy smartfon posiada moduł GSM zdolny
łączyć się ze stacjami bazowymi operatorów komórkowych (BTS).

Po aktywacji trybu samolotowego, moduł GSM zostanie efektywnie wyłączony uniemożliwiając nawiązanie
połączenia z takiego telefonu, no i też komunikacja ze stacjami bazowymi zostanie zerwana, przez co
nikt nie będzie się w stanie z nami połączyć. Każdy smartfon wyposażony jest też w moduł WiFi i
Bluetooth. Jak najbardziej te moduły potrafią generować fale radiowe i te rzeczy również zostaną
wyłączone po aktywacji trybu samolotowego.

Czy którykolwiek z tych wyżej wymienionych modułów jest w stanie działać w trybie nasłuchu
(podobnie do GPS) lub sam z siebie się aktywować, gdy uzna to za stosowne? Tego nie wiem ale wcale
bym się nie zdziwił, gdyby to było wykonalne. Jeśli nasz smartfon nie posiada żadnych sprzętowych
przełączników (hard kill switch) umożliwiających odcięcie zasilania modułom GSM/WiFi/BT, to tryb
samolotowy nie daje nam żadnych gwarancji, że te rzeczy faktycznie pozostaną nieaktywne po ich
wyłączeniu.

I to w zasadzie wszystko co tryb samolotowy robi z naszym smartfonem. Jeśli jednak mamy włączoną
lokalizację w ustawieniach Androida i jej nie wyłączyliśmy ręcznie, to nawet po aktywacji trybu
samolotowego, nasz telefon jest w stanie bez większego problemu rejestrować swoje położenie i to w
czasie rzeczywistym. Oczywiście z racji wyłączenia modemu GSM i WiFi, nie mamy w komórce internetu.
Nie możemy zatem tych danych przesłać do serwerów Google, tak jak to ma miejsce, gdy transmisja
3G/4G jest aktywna. Niemniej jednak, te dane są zbierane w formie offline i z chwilą, gdy tylko
wyłączymy tryb samolotowy i uzyskamy połączenie z internetem, to wszystkie dane na temat naszej
lokalizacji (tej z okresu bycia offline) zostaną przesłane na serwery Google.

Niektóre ROM'y są w stanie blokować modem GSM, WiFi, BT oraz GPS, gdy się w nich włączy tryb
samolotowy. Wygląda jednak na to, że to są domieszki producenta telefonów, a nie standardowe
zachowanie Androida. U mnie na każdym telefonie zachowanie się urządzenia po przełączeniu mu trybu
samolotowego wyglądało inaczej. Zwykle jednak jest wyłączany modem GSM, WiFi i BT, z tym, że te dwa
ostatnie można w późniejszym czasie włączyć i system nie wychodzi z trybu samolotowego, czyli można
np. korzystać z WiFi w trybie samolotowym. GPS z kolei nie jest ruszany w żaden sposób, tj. jeśli
był aktywny, to włączenie trybu samolotowego nie wyłączy odbiornika GPS ani nie wyłączy lokalizacji
w ustawieniach Androida. Jedna rzecz, której nie da się włączyć w trybie samolotowym, a która jest
wyłączana automatycznie po aktywacji trybu samolotowego, to modem GSM i dane pakietowe 3G/4G.

## Blokowanie sensorów

Wygląda na to, że w Androidzie (począwszy od wersji 10) znalazł się mechanizm, który znakomicie
dopełnia tryb samolotowy. Mowa tutaj o [blokowaniu wszystkich sensorów (w tym GPS)][21], co skutkuje
brakiem możliwości uzyskania jakiegokolwiek położenia przez aplikacje korzystające z lokalizacji.
Bardzo ciekawa alternatywa w stosunku do tych powyżej opisanych opcji, choć jeden mechanizm
naturalnie nie wyklucza stosowania drugiego.

## Pozorowanie lokalizacji

Z powyżej zgromadzonych informacji wychodzi jasno, że tryb samolotowy w żaden sposób nie ma tykać
GPS czy jakichkolwiek ustawień lokalizacji, które można określić w ustawieniach Androida. Ciężko
jest zatem winić tryb samolotowy, że nie blokuje GPS (czy ogólnie dostępu do lokalizacji), przez co
Google jest w stanie poznać nasze położenie, gdy tylko będzie miał na to ochotę.

Teoretycznie wyłączenie usługi lokalizacji powinno uniemożliwić aplikacjom rejestrowanie danych
pochodzących z odbiornika GPS, czy też innych źródeł lokalizacji. Niemniej jednak, biorąc pod uwagę
fakt, że cała masa aplikacji wymaga do poprawnego działania informacji o lokalizacji, to pozbawienie
ich tych danych może w przypadku takich appek nieść za sobą nieprzewidziane konsekwencje. W
najlepszym wypadku, taka aplikacja będzie zgłaszać mniej lub bardziej widoczne błędy ale też może
zwyczajnie odmówić uruchomienia się, gdy w telefonie mamy wyłączoną lokalizację. Co w takim
przypadku zrobić? Możemy pokusić się o pozorowanie lokalizacji.

Pozorowanie lokalizacji ma miejsce wtedy, gdy GPS zdaje się działać prawidłowo ale zamiast danych z
satelitów, zwracana jest lokalizacja, którą sobie ręcznie określimy. Taki zabieg był możliwy w
Androidzie już od dawna i nigdy nie były potrzebne do tego celu prawa administratora systemu root.
Zatem praktycznie każdy może pozorować swoją lokalizację, co potencjalnie może nam pomóc w kwestii
naszej prywatności.

Trzeba jednak sobie zdawać sprawę, że odbiornik GPS to tylko jedno ze źródeł danych mogących pomóc
w ustaleniu naszej (nie zawsze dokładnej) lokalizacji. Zatem pozorowanie lokalizacji może wpłynąć
jedynie na wskazania GPS. Stacje bazowe operatorów komórkowych, sieci WiFi rozgłaszające się w
sąsiedztwie czy urządzenia BT w dalszym ciągu mogą naszą lokalizację zdradzić lub przynajmniej
wykazać niekonsekwencję lub próbę oszustwa, co te bardziej cwane aplikacje mogą wykryć. Dlatego też
przydałoby się te wszystkie wyżej opisane ustawienia lokalizacji w Androidzie powyłączać.

### Aplikacja do pozorowania lokalizacji

Gdy już odpowiednio skonfigurowaliśmy system, pobieramy z [repozytorium F-Droid aplikację Fake
Traveler][3]. Zanim jednak przejdziemy do jej uruchomienia, musimy tę appkę określić w ustawieniach
deweloperskich jako aplikację do pozorowania lokalizacji:

|   |   |   |
|---|---|---|
| ![android-mock-fake-location-gps-fake-traveler-app](/img/2021/09/015.android-mock-fake-location-gps-fake-traveler-app.jpg#small) | ![android-mock-fake-location-gps-fake-traveler-app](/img/2021/09/016.android-mock-fake-location-gps-fake-traveler-app.jpg#small) | ![android-mock-fake-location-gps-fake-traveler-app](/img/2021/09/017.android-mock-fake-location-gps-fake-traveler-app.jpg#small) |

### Pozorowanie lokalizacji z Fake Traveler

Pozostało nam już w zasadzie uruchomienie appki Fake Traveler i wskazanie na mapie położenia, które
chcemy pozorować, po czym klikamy `APPLY` widoczny w prawym górnym rogu:

![android-mock-fake-location-gps-fake-traveler-app-set](/img/2021/09/018.android-mock-fake-location-gps-fake-traveler-app-set.jpg#small)

Naturalnie, nie ma co wierzyć tej aplikacji na słowo, że nasza lokalizacja została zamaskowana.
Możemy ten fakt zweryfikować przez uruchomienie dowolnej appki, która korzysta z lokalizacji, np.
OsmAnd+ czy Google Maps. W moim przypadku te appki zwracają poniższe wyniki:

|   |   |
|---|---|
| ![android-mock-fake-location-gps-fake-traveler-app-check](/img/2021/09/019.android-mock-fake-location-gps-fake-traveler-app-check.jpg#small) | ![android-mock-fake-location-gps-fake-traveler-app-check](/img/2021/09/020.android-mock-fake-location-gps-fake-traveler-app-check.jpg#small) |

No i proszę, według mojego telefonu jestem gdzieś na Syberii.

Jeśli mamy jakieś problemy z działaniem aplikacji Fake Traveler, np. appki w dalszym ciągu widzą
nasze faktyczne położenie, to prawdopodobnie wartość w `How many times you want to mock the
location` w ustawieniach Fake Traveler jest niezerowa. Powinniśmy tutaj mieć `0` , co efektywnie
oznacza nieskończoność. Warto też rzucić okiem na uprawnienia Fake Traveler i upewnić się, że ma on
dostęp do lokalizacji cały czas:

|   |   |
|---|---|
| ![android-mock-fake-location-gps-fake-traveler-app-settings](/img/2021/09/021.android-mock-fake-location-gps-fake-traveler-app-settings.jpg#small) | ![android-mock-fake-location-gps-fake-traveler-app-settings](/img/2021/09/022.android-mock-fake-location-gps-fake-traveler-app-settings.jpg#small) |

## Robienie zdjęć z pozorowaną lokalizacją

Pozorowanie lokalizacji nie tylko przydaje się do oszukiwania aplikacji i zwracania im błędnych
danych GPS. Możemy też podrobić geotagi w fotkach czy filmach robionych naszym smartfonem. Zróbmy
zatem przykładową fotkę mając uruchomioną appkę Fake Traveler i zajrzyjmy w metadane EXIF takiego
zdjęcia:

    $ exiftool IMG_20210920_154223.jpg | grep -i gps
    GPS Latitude Ref                : North
    GPS Altitude Ref                : Above Sea Level
    GPS Processing Method           : gps
    GPS Version ID                  : 2.2.0.0
    GPS Longitude Ref               : East
    GPS Time Stamp                  : 13:42:22
    GPS Date Stamp                  : 2021:09:20
    GPS Img Direction               : 288.28
    GPS Img Direction Ref           : Magnetic North
    GPS Altitude                    : 3 m Above Sea Level
    GPS Date/Time                   : 2021:09:20 13:42:22Z
    GPS Latitude                    : 62 deg 2' 11.12" N
    GPS Longitude                   : 129 deg 45' 31.53" E
    GPS Position                    : 62 deg 2' 11.12" N, 129 deg 45' 31.53" E

No i jak widzimy, w tagach GPS została zapisana lokalizacja `62 deg 2' 11.12" N, 129 deg 45'
31.53" E` , czyli dokładnie ta sama, którą określiliśmy w Fake Traveler. Znajdujemy się też 3m
ponad poziomem morza i to wskazanie również jest błędne.

## Podsumowanie

Android ma dość rozbudowane opcje dotyczące zbierania informacji o lokalizacji naszego smartfona.
Nasze położenie można odczytać nie tylko z danych satelitów systemu GPS  ale zdradzić je mogą
również do pewnego stopnia stacje bazowe operatora komórkowego (BTS), sieci WiFi, których przecie
jest cała masa, czy nawet urządzenia Bluetooth, których też w ostatnim czasie dość sporo przybyło.

Jeśli wierzyć w to, co Google pisze odnośnie uzyskiwania danych o położeniu swoich użytkowników, to
technicznie rzecz biorąc można całkowicie wyłączyć rejestrowanie lokalizacji w ustawieniach
Androida, choć jak mogliśmy zauważyć, jest kilka sytuacji (np. usługa numerów alarmowych), w
których jest wielce prawdopodobne, że nasz smartfon prześle informacje, czy to pozyskane z GPS czy
ze stacji BTS, do której nasz telefon się zalogował.

Szereg aplikacji w Androidzie może nie działać poprawnie bez aktywnej usługi lokalizacji lub też
całkowicie odmówić w takich sytuacjach współpracy. Na takie wredne appki jest Fake Traveler, który
radzi sobie pierwszorzędnie z pozorowaniem lokalizacji, a same aplikacje pokroju map od Google nie
są nawet świadome faktu, że wskazania GPS zostały podrobione.

Fake Traveler jest w stanie podnieść poziom naszej prywatności przy korzystaniu ze smartfona
maskując nasze położenie ale trzeba sobie też zdawać sprawę, że inne mechanizmy są w stanie to
położenie zdradzić, choć nie zawsze tak dokładnie jak ma to miejsce w przypadku GPS. Nawet jeśli
dokładność będzie na poziomie kilkuset metrów czy paru kilometrów, to wciąż pewne wnioski można z
takiej informacji wyciągnąć, a korelacja zdarzeń w czasie może przyczynić się do poznania gdzie
dokładnie byliśmy i co tam robiliśmy.

Z racji, że zbieranie danych na temat lokalizacji smartfona przez Google nie jest do końca
transparentne co do tego jakie dane są zbierane (i dokładnie w jaki sposób), to jeśli faktycznie
zależy nam na prywatności, to powinniśmy zrezygnować z zabierania telefonu na te bardziej prywatne
spotkania. Jeśli już musimy zabierać ze sobą komórkę wszędzie tam, gdzie się zamierzamy wybrać, to
powinniśmy odejść od rozwiązań mających Androida jako bazę systemu operacyjnego i kierować się
bardziej w stronę projektów [Librem5][16] czy [PinePhone][17], które posiadają sprzętowe
przełączniki do odcięcia konkretnych obwodów elektrycznych w telefonie, tak by mieć pewność, że nie
działają one po ich wyłączeniu lub też żaden malware ich nam nie załączy sam z siebie bez naszej
świadomej i dobrowolnej zgody.


[1]: https://www.thesun.co.uk/tech/7811918/google-is-tracking-you-even-with-airplane-mode-turned-on/
[2]: https://github.com/mcastillof/FakeTraveler
[3]: https://f-droid.org/en/packages/cl.coders.faketraveler/
[4]: https://en.wikipedia.org/wiki/GPS_signals#Navigation_message
[5]: https://en.wikipedia.org/wiki/Assisted_GNSS
[6]: https://en.wikipedia.org/wiki/Global_Positioning_System
[7]: https://en.wikipedia.org/wiki/True-range_multilateration
[8]: https://en.wikipedia.org/wiki/Triangulation
[9]: https://gis.stackexchange.com/a/41093/193381
[10]: https://en.wikipedia.org/wiki/Time_dilation
[11]: https://en.wikipedia.org/wiki/Theory_of_relativity
[12]: https://www.zdnet.com/article/how-google-and-everyone-else-gets-wi-fi-location-data/
[13]: https://www.xda-developers.com/android-pie-throttling-wi-fi-scans-crippling-apps/
[14]: https://crisisresponse.google/emergencylocationservice/how-it-works/
[15]: https://en.wikipedia.org/wiki/Emergency_telephone_number#Emergency_numbers_and_mobile_telephones
[16]: https://shop.puri.sm/shop/librem-5/
[17]: https://www.pine64.org/pinephone/
[18]: https://developer.android.com/training/location/permissions
[19]: https://www.quora.com/If-a-cell-phone-doesn-t-have-a-SIM-card-how-can-it-make-emergency-calls
[20]: https://www.youtube.com/watch?v=btzFoBncPF4
[21]: /post/blokowanie-dostepu-do-sensorow-kamera-mikrofon-gps-aplikacjom-w-androidzie/
