---
author: Morfik
categories:
- Hardware
date:    2021-01-27 18:03:00 +0100
lastmod: 2021-01-27 18:03:00 +0100
published: true
status: publish
tags:
- drukarka
- debian
- cups
- sieć
- hewlett-packard
- laserjet
- p2055dn
- logi
title: Drukarka laserowa pod linux (HP LaserJet P2055dn)
---

Parę tygodni temu moja leciwa już (20 letnia) drukarka atramentowa Epson Stylus Color 760
postanowiła odmówić współpracy i zaprzestała dalszego drukowania stron. Po rozkręceniu jej, okazało
się, że ma ona w sobie tyle syfu (m.in. w okolicach dysz), że mechanizmy tej drukarki nie były już
w stanie najwyraźniej tego doczyścić. Pomyślałem, że najwyższy już czas pozbyć się tej drukarki i
poszukać jej następcy. W wytycznych kupna nowej drukarki były przede wszystkim cena (do 500 zł),
bezproblemowa praca pod linux (Debian/Ubuntu) oraz możliwość pracy w sieci (preferowane WiFi) bez
zaprzęgania do tego celu innych sprzętów, np. routera czy komputera stacjonarnego/laptopa. Wybór
padł na poleasingową drukarkę laserową Hewlett Packard (HP) LaserJet P2055dn, ze względu na fakt,
że posiada ona wszystkie te wyżej wymienione właściwości (no może poza WiFi) i można nią zarządzać
bez większego problemu czy to przy pomocy dedykowanych narzędzi z pakietu `hplip` / `hplip-gui` ,
czy też bezpośrednio za sprawą CUPS.

<!--more-->
## Kryteria doboru nowej drukarki

Drukarka HP LaserJet P2055dn nie została wybrana przypadkowo, bo zależało mi na paru szczegółach,
które zadecydowały o wyborze akurat tego konkretnego modelu. Poniżej znajduje się krótkie
zestawienie opcji, które brałem pod uwagę przy rozglądaniu się za kupnem nowej, choć używanej
drukarki. W zasadzie kupno nowego sprzętu w moim przypadku odpada. Chodzi generalnie o to, że nowe
urządzenia są aktualnie dość drogie i kupowanie drukarki w obecnym czasie nie jest zbyt mądrym
zabiegiem. Poza tym, na rynku jest dość spory przesyt różnorakiego używanego sprzętu
elektronicznego (nie tylko drukującego), który jest relatywnie tani, przynajmniej w stosunku do
swojej początkowej wartości, którą miał w dniu premiery. Dlatego też ja od już dłuższego czasu
rozglądam się głównie za używanymi maszynami i tak samo było w przypadku tej drukarki.

### Drukarka atramentowa vs. laserowa

Drukarka, którą zamierzałem nabyć, musiała być drukarką laserowa z tego względu, że patrząc na
wnętrze rozkręconego Epson'a doszedłem do wniosku, że atramentówki marnują niesamowite ilości tuszu
(gąbki pod dyszami wręcz pływały w nim). Do tej pory uważałem, że większość tuszu ląduje na
papierze przy drukowaniu kartek ale teraz już taki pewny nie jestem. Jako, że tusz do tanich nie
należy (nawet stosując zamienniki, o ile się w ogóle jeszcze je znajdzie), to postanowiłem odejść
już od tej przestarzałej nieco technologi druku na rzecz drukarek laserowych.

#### Drukarka kolorowa czy monochromatyczna/czarnobiała

Nie da się ukryć, że póki co nowe drukarki laserowe i do tego kolorowe są dość drogie. Dlatego też
interesowały mnie jedynie drukarki monochromatyczne/czarnobiałe. W dzisiejszych czasach i tak już
nie drukuje się dokumentów, tak jak to jeszcze zwykliśmy robić 20 lat temu. Obecnie mamy nieco
bardziej rozbudowany internet i drukowanie fizycznych dokumentów tylko po to, by przesłać/zanieść je
do kogoś innego, stało się ździebko bez sensu. Niemniej jednak, zadrukowane tekstem kartki papieru
w dalszym ciągu mogą okazać się wielce użyteczne.

Ogromna ilość drukowanych dokumentów zawiera przede wszystkim tekst, a nie obrazki, co sprawia, że
drukarki kolorowe tracą nieco (a może nawet i bardzo) na znaczeniu. Poza tym, obrazki w dalszym
ciągu na czarnobiałych drukarkach można drukować tyle, że w skali szarości. Kto z nas nie drukował
na drukarkach atramentowych obrazków kolorowych czarnym tuszem, bo tusz kolorowy był zbyt drogi?
Kto z nas nie przekładał zużytego pojemnika tuszu kolorowego ponownie do drukarki by ominąć jej
zabezpieczenia pustego pojemnika? W moim Epson'ie był taki problem, że nawet jak się nic nie
drukowało kolorowego (w ustawieniach strony w systemie, był wymuszony druk czarnobiały/skala
szarości), to i tak drukarka po jakimś czasie odmawiała drukowania, bo pojemnik z tuszem kolorowym
był pusty. Na szczęście można było ten pojemnik z tuszem kolorowym wyciągnąć i wsadzić go ponownie,
by znów  było można dalej drukować czarnym tuszem. Dlatego właśnie postanowiłem zrezygnować w
drukarek kolorowych.

### Sama drukarka czy ze skanerem (3w1, kopiarka)

Gdy człowiek myśli o kupnie nowej drukarki, automatycznie przychodzi mu na myśl również i skaner.
Taka mentalność była na porządku dziennym 20 lat temu. Jak ktoś miał drukarkę, no to musiał mieć do
niej i skaner, by drukować zeskanowane dokumenty. Ja skanerów mam dwa i z żadnego z nich nie
korzystałem od ponad 10 lat, czyli mniej więcej od czasu, gdy upowszechniły się telefony/smartfony
z dość dobrymi aparatami. Dzisiaj praktycznie każdy smartfon dysponuje aparatem 10+ mpix (te lepsze
nawet i 50-80 mpix), przez co wykonanie fotki (skanu) dokumentu jest banalnie proste i do tego
natychmiastowe.

Kiedyś trzeba było podłączać telefon do komputera, by takie fotki zgrać na dysk twardy i dopiero
myśleć o ich drukowaniu. Dzisiaj telefony mają WiFi i bez problemu można tak ofotkowane dokumenty
drukować bezpośrednio ze smartfona. Oczywiście jeśli nasza drukarka ma dostęp do sieci.

Gdy widzę oferty drukarek 3w1 (urządzeń wielofunkcyjnych), to odnoszę wrażenie, że takie sprzęty są
w ofercie jedynie po to, by doić ludzi z hajsu. Jakiś czas temu, producenci doszli zapewne do
wniosku, że ludzie powoli odchodzą od posiadania drukarek i skanerów w domach, choć te pierwsze
urządzenia jeszcze są skłonni kupić. Pojawił się zatem problem ze skanerami, których już nikt
nabywać nie chce, bo są to urządzenia w otaczającej nas rzeczywistości zupełnie zbędne i pozbawione
jakiegokolwiek sensu. No ale przecież sama drukarka kosztuje mniej niż drukarka i skaner. Więc
trzeba było tak zmanipulować człowieka, by ten kupił sprzęt kompletnie mu do niczego nieprzydany i
by jeszcze taka osoba była zadowolona z transakcji. Tak właśnie powstały te urządzenia
wielofunkcyjne 3w1 do zastosowania w domowym zaciszu, w których korzysta się w zasadzie tylko z
drukarki, a płaci się również i za pozostałe elementy urządzenia. Dodatkowo, taka drukarka 3w1 jest
o wiele bardziej skomplikowana w budowie niż sama drukarka, a to z kolei sprawia, że więcej rzeczy
może się popsuć. A jak się popsuje nam drukarka w tym urządzeniu wielofunkcyjnym, to musimy również
wymienić i skaner, no bo po co nam drukarka 3w1 skoro nie potrafi drukować. Dlatego też jeśli
potrzebujemy drukarki, to kupujemy drukarkę.

### Drukarka sieciowa (WiFi) czy bez dostępu do sieci

Jeśli chodzi o sieć w przypadku drukarek, to można te urządzenia podzielić z grubsza na trzy
kategorie. Do pierwszej kategorii trafiają drukarki, które mają wbudowany port rj-45 (ethernet)
umożliwiający przewodowe podłączenie drukarki do routera/switch'a i w ten sposób każdy host w sieci
domowej będzie w stanie się z tym urządzeniem komunikować i bezpośrednio drukować dokumenty. Druga
kategoria drukarek, to te, które dysponują WiFi, czyli są w stanie komunikować się bezprzewodowo z
urządzeniami w sieci, w tym też bezpośrednio, np. za pomocą [WiFi Direct][11]. Do ostatniej
kategorii można zaliczyć drukarki, które nie mają żadnej możliwości sieciowego komunikowania się.
Niemniej jednak, nie oznacza to, że nie można z takiej drukarki zrobić drukarki sieciowej, choć to
już zależy od firmware naszego routera domowego.

Mój router dysponuje kilkoma portami USB i do tego ma wgrany alternatywny firmware OpenWRT. Mogę
zatem podłączyć drukarkę po USB do routera i przy pomocy linux'owego demona `p910nd` udostępnić
drukarkę USB w sieci lokalnej. Jest to mniej więcej takie samo rozwiązanie co w przypadku portu
rj-45, choć w przypadku demona `p910nd` trzeba poczynić dodatkowe kroki, by całość działała nam
należycie. W obu tych przypadkach wymagane jest jednak pociągnięcie fizycznego przewodu (USB czy
ethernet) od drukarki do routera, co utylizuje też jeden port USB lub gniazdo w switch'u routera.
Dlatego właśnie ja bym się skłaniał bardziej ku rozwiązaniu WiFi.

### Dupleks, czyli obustronne zadrukowanie kartki

W standardzie drukarki oferują drukowanie tylko i wyłączenie po jednej stronie karki. Są jednak na
rynku modele drukarek oferujące dupleks, czyli możliwość obustronnego zadrukowania papieru.
Niemniej jednak, taki ficzer tani nie jest (dodatkowy koszt 100-200zł), no i też znacznie
komplikuje budowę drukarki, a jak wiadomo im bardziej skomplikowany jest sprzęt, tym więcej rzeczy
w nim może się popsuć, co przekłada się na częstszą awaryjność takiego urządzenia.

Niewątpliwą zaletą dupleksu jest oszczędność papieru. Gdy drukujemy dokumenty dwustronnie, to
zużywamy dwukrotnie mniej papieru. Ryza papieru (500 sztuk), kosztuje w granicach 15-20zł, zatem na
1000 zadrukowanych stron mamy potencjalny zysk 20zł. Oczywiście nie każdy dokument będzie miał
ilość stron podzielną przez dwa, a im mniejszy dokument, tym mniejszy będzie nasz końcowy zysk.
Gdybyśmy drukowali tylko i wyłącznie dokumenty jednostronicowe, to dupleks w ogóle by nam się do
niczego nie przydał. Dlatego też trzeba się zastanowić, czy będziemy drukować dużo i co to
dokładnie będzie, bo może się okazać, że dupleks w naszym przypadku będzie zupełnie zbędny i tylko
niepotrzebnie wydajemy hajs. Trzeba też pamiętać, że można zawsze zastosować manualny dupleks,
czyli ręczne przewrócenie kartki po wydrukowaniu jednej strony i wrzuceniu jej ponownie do
podajnika papieru, choć na szerszą skalę takie rozwiązanie może być ździebko męczące.

### Materiały eksploatacyjne

Pewne elementy drukarek się zużywają. W drukarkach laserowych jest to zespół bębna (ładowanego
ujemnie przez laser i przyciągającego dodatnio naładowany toner), no i oczywiście sam toner, czyli
proszek, który po naniesieniu na papier i podgrzaniu do około 200-250°C jest w niego wtapiany. Póki
co spotkałem się z dwoma rozwiązaniami bębnów. Pierwsze z nich to zintegrowany bęben z tonerem
(chyba najbardziej popularny). W takim przypadku, gdy skończy się toner, to trzeba wymienić również
i bęben. Drugie rozwiązanie z kolei rozdziela bęben od toneru i wymiana toneru nie owocuje wymianą
bębna. To drugie rozwiązanie tnie koszty, bo bęben ma zwykle sporo wyższą żywotność w stosunku do
tonera (chodzi o ilość kartek, które przewiną się przez drukarkę). Są zasobniki tonera, którymi
można wydrukować 1,5K stron, 3K stron 6K stron, czy nawet i więcej (do 10K), ale to i tak niewiele
w stosunku do wytrzymałości bębna, który jest w stanie znieść 20-30K stron. Zatem tak średnio na 10
załadowanych tonerów przydałoby się wymienić również i bęben.

Warto tutaj wspomnieć, że ta ilość zadrukowanych stron, które rzekomo drukarka będzie nam w stanie
wydrukować na danym tonerze jest szacowana przy 5% pokryciu strony. Jeśli drukujemy obrazki albo
sporo tekstu na stronie, to pokrycie strony tonerem jest sporo wyższe niż te 5%. Zdarzają się
strony, które będą mieć mniej niż te 5% pokrycia. Niemniej jednak, zwykle na stronie będziemy mieć
nieco więcej zadrukowane niż 5% i z tych 3K deklarowanych stron może nam wyjść 1K (albo i mniej) i
nie będzie to nic dziwnego. Po prostu taki kolejny marketingowy bełkot sprawiający, że taki toner
jest wart w naszych oczach coś więcej niż w rzeczywistości -- czy zapłacilibyście za toner 200zł
oferujący druk 3000 stron? A gdyby na opakowaniu była informacja, że zamiast 3000 stron jest ich
300?

#### Tonery/bębny oryginalne/producenta i zamienniki

Na rynku dostępne są zamienniki tonerów/bębnów, które kosztują czasami sporo mniej, np. 20zł za
zamiennik w stosunku do 200 zł za oryginał oferowany przez producenta drukarki. Czy powinniśmy w
ogóle rozważać stosowanie zamienników? W przypadku drukarki, która kosztuje 300-500 zł, kupowanie
oryginalnego toneru za 150 zł raczej mija się z celem. Dlatego też powinniśmy zapoznać się z
informacjami dotyczącymi cen materiałów eksploatacyjnych przed zakupem drukarki. Dotyczy to zarówno
materiałów oryginalnych, jak i zamienników.

Pamiętajmy, że są różnej wielkości tonery, które można załadować do tej samej drukarki laserowej.
Dlatego zwracajmy uwagę na ten szczegół, bo może się okazać, że w oryginalnym tonerze możemy
wydrukować do 6K stron, a na zamienniku do 1,5K stron, przy czym zamiennik będzie kosztował 20 zł,
a oryginał 100zł. Nawet jeśli oryginał wyniesie nas więcej niż zamienniki, to niewielka różnica w
cenie zawsze przemawia na korzyść producenta sprzętu. Niemniej jednak, te dysproporcje cenowe w
stosunku do szacowanej liczby zadrukowanych stron mogą być (i zwykle będą) sporo wyższe i co w takim
przypadku zrobić?

Zasada jest generalnie taka, że im tańszy zamiennik, tym gorsza jakość tonera. Im gorszy toner, tym
gorszej jakości będą drukowane dokumenty, oraz mogą pojawiać się smugi czy inne
artefakty/zabrudzenia w miejscach zadrukowanych. Wizualnie może to nie wyglądać za dobrze.
Powinniśmy raczej unikać najtańszych tonerów, a najlepiej korzystać z tych już sprawdzonych,
których wyniki możemy zobaczyć na własne oczy.

Kolejna sprawa jeśli chodzi o tonery, to problem z magnetyzowaniem. Te tańsze tonery mogą słabo
przylegać do bębna i w procesie nanoszenia toneru na papier mogą się odrywać i latać w powietrzu
wewnątrz obudowy drukarki. Taki pył może zanieczyścić laser. W efekcie trzeba będzie serwisować
drukarkę, a taka usługa do tanich zwykle nie należy. Trzeba też zdawać sobie sprawę, że producent
drukarki może odmówić naprawy, gdy nie korzystamy z jego materiałów eksploatacyjnych. Nawet jeśli
wymienimy zamiennik na oryginał i taką drukarkę będziemy próbować serwisować, to zwykle w logu
urządzenia są rejestrowane pewnie zdarzenia. Poniżej jest przykład z mojej drukarki:

![](/img/2021/01/001-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-raport.png#huge)

Jak widać parę zdarzeń tam zostało zalogowanych podczas pracy tego urządzenia. Mamy tam m.in. kod
`10.4000` , który można robić w przypadku tego producenta drukarki na `10.400` oraz `0` . Ta
pierwsza część oznacza że nowe materiały eksploatacyjne (toner i bęben w tym przypadku) zostały
zainstalowane w drukarce. Druga część, mówi jaki kolor tonera został wsadzony, tutaj `0` odpowiada
za czarny. Wiemy zatem mniej więcej co ile stron był wymieniany toner. Nie wiem czy taki sam kod
jest wykorzystywany w przypadku zamienników.

Niektóre modele drukarek mogą odmówić pracy przy zamiennikach. Dlatego lepiej jest się upewnić
przed zakupem drukarki, że będzie ona działać z tymi zamiennikami, które zamierzamy nabywać.

### Toner i bębny w drukarce po jej zakupie

Zwykle przy zakupie nowej drukarki, materiały eksploatacyjne są już załadowane do drukarki i po
wyjęciu urządzenia z pudełka można od razu zacząć drukować dokumenty. Przy kupowaniu używanych
drukarek te materiały eksploatacyjne nie zawsze mogą być dołączone do drukarki, lub też mogą być
niepełne. Zwróćmy zatem uwagę czy kupowana przez nas drukarka ma na wyposażeniu materiały
eksploatacyjne.

## Nowe drukarki czy używane (poleasingowe)

Obecnie nowe drukarki są bardzo drogie. Z porównania cen, które były jeszcze rok czy dwa lata temu,
wychodzi, że ceny drukarek poszły ostro w górę -- 2-3 krotnie i nie zapowiada się, by te ceny w
jakiejś dającej przewidzieć się przyszłości zostały obniżone. Być może dobrym wyjściem okaże się
zakup używanej lub poleasingowej drukarki. Oczywiście mówimy tutaj o drukarkach laserowych, a nie
atramentowych -- tych drugich w życiu bym używanych nie kupił, przynajmniej już nie, bo po tym co
zobaczyłem w swojej drukarce, to teraz mam koszmary.

Laserowe drukarki mają to do siebie, że aż tak bardzo się nie brudzą podczas drukowania, zakładając
oczywiście wykorzystanie dobrej jakości materiałów eksploatacyjnych, a zwykle oryginalne materiały
są ładowane do tych drukarek poleasingowych (zanim jeszcze się stały poleasingowe). Oczywiście w
dalszym ciągu kurz czy inny pył środowiskowy może się dostać do wnętrza obudowy, co może
zanieczyścić laser czy w inny sposób odbić się negatywnie na jakości drukowanych dokumentów.

Zaletą jednak drukarek laserowych jest fakt braku dysz i tuszu, przez co toner w takiej kasecie
może leżeć wiele lat i nic się z nim złego nie dzieje. Ta moja drukarka pochodzi właśnie z oferty
poleasingowej, gdzie według informacji uzyskanych z obudowy drukarki wychodzi, że to urządzenie
zostało wyprodukowane w lutym 2011 roku. W dzienniku systemowym zaś można się doszukać daty
pierwszej instalacji, która wskazuje na 2014-05-02, zaś ostatnio używana była w 2016-03-03. Zatem
ta drukarka przeleżała 5 lat w magazynie/magazynach, a przez niecałe dwa lata swojej pracy
wydrukowała około 8K stron na 4 tonerach, z czego jeden w dalszym ciągu był w drukarce w chwili
zakupu ze stanem uzupełnienia około 53%. Po testach okazało się, że drukarka bez problemu drukuje,
a sam druk wychodzi czysty bez żadnych zabrudzeń. Zatem toner przeleżał 5 lat i nic się z nim złego
nie stało.

### Przebieg używanych drukarek laserowych

Przy kupowaniu drukarki laserowej używanej/poleasingowej koniecznie zwracajmy uwagę na jej przebieg.
Ta informacja jest do odczytania z logu systemowego drukarki, [choć można jej ten licznik
wyzerować różnymi sposobami][1] ale bez dodatkowych zabiegów (i kosztów) się raczej nie obejdzie.
Gdy kupujemy drukarkę do 500zł, to jest niemal pewne, że licznik przebiegu będzie nietknięty i po
jego stanie możemy oszacować zużycie sprzętu. Generalnie to im mniej stron ma wydrukowanych dany
model drukarki, tym lepiej. Mi akurat się trafił 10 letni model, który ma mniej niż 10K stron
wydrukowanych w około 2 lata swojej pracy, a pozostałe 8 lat spędził w różnych magazynach
najpewniej. Także, nie jest jakoś mocno zajechany ten sprzęt, bo zdarzają się drukarki mające
100-150K przebiegu i w nich najpewniej trzeba by szereg mechanizmów powymieniać, co naturalnie
podwyższyłoby całościowy koszt zakupu takiej drukarki.

## Wsparcie pod linux

Skoro kupujemy już drukarkę, to najważniejsze dla nas ma znaczenie, czy sprzęt, który nabywamy,
będzie nam bezproblemowo działał z używanymi przez nas systemami operacyjnymi. Ja w zasadzie
używam tylko i wyłącznie linux'a -- zarówno na routerze mam wgrany OpenWRT, jak i w na laptopach
mam Debiana/Ubuntu, a na fonach mam Androida. Jest kilka różnych konfiguracji, które możemy
zastosować, by być w stanie drukować dokumenty na zakupionej dopiero co drukarce.

### Drukarka podłączona po USB do komputera

Zwykle drukarki mają już port USB, przez który możemy podłączyć taki sprzęt bezpośrednio do
komputera. Niemniej jednak, takie rozwiązanie jest przeznaczone jedynie dla osób, które mają w
zasadzie jeden komputer i tylko z niego zamierzają wchodzić w interakcję z drukarką (wciąż można
taką drukarkę z takiego komputera udostępnić w sieci).

W przypadku, gdy tych maszyn w domu jest więcej, np. część domowników ma swoje laptopy czy telefony,
to lepszym rozwiązaniem jest przerobienie drukarki na drukarkę sieciową. Praktycznie każdy model
drukarki da radę przerobić na drukarkę sieciową, nawet te, które nie mają w swoich obudowach
zaszytych gniazdek rj-45 czy czipów WiFi. Niemniej jednak, drukarka, którą podłączamy do systemu,
musi być przez niego wspiera, tj. muszą w nim być dostępne odpowiednie sterowniki.

#### Drukarki HP i oprogramowanie hplip

W moim przypadku, wybór padł na drukarkę Hewlett Packard LaserJet P2055dn. Jak widać po samej
nazwie, jest to drukarka laserowa, z dupleksem ( `d` ) i siecią ( `n` , choć jedynie przewodową).
Urządzenia HP mają tę zaletę pod linux, że mamy tutaj [dedykowane oprogramowanie do tych
drukarek][3], które znajduje się w pakiecie `hplip` . Jest również nakładka graficzna w pakiecie
`hplip-gui` . To oprogramowanie jest przeznaczone do pracy tylko/głównie ze sprzętem HP, choć
naturalnie w przypadku drukarek HP możemy również korzystać z innych narzędzi pokroju
`system-config-printer` . Bez znaczenia z których nakładek GUI skorzystamy, to i tak za nimi
będzie siedział CUPS, z którego również bezpośrednio możemy korzystać.

W skład pakietu `hplip` / `hplip-gui` wchodzi sporo skryptów python'a, które mogą być
wykorzystywane zarówno do skonfigurowania drukarki podłączonej przez port USB do komputera, czy też
drukarek udostępnianych w sieci z innych maszyn, jak i samych drukarek sieciowych. Poniżej fotka
obrazująca wygląd `hplip-gui` :

![](/img/2021/01/002-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip.png#huge)

Jak widać, z poziomu tego oprogramowania jesteśmy w stanie bez większego problemu zarządzać
drukarką, jak i zadaniami, które do niej będą przesyłane. Oprogramowanie `hplip` bez większego
problemu wykryje model drukarki HP i powinno ono dobrać odpowiedni sterownik dla niej:

![](/img/2021/01/003-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-usb.png#huge)
![](/img/2021/01/004-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-usb.png#huge)
![](/img/2021/01/005-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-usb.png#huge)

I to w zasadzie tyle, jeśli chodzi o konfigurowanie drukarki HP podłączonej do portu USB komputera.
Niemniej jednak, ta drukarka ma własny system operacyjny i może działać w sieci bez potrzeby
podłączania jej do pełnowymiarowego komputera stacjonarnego czy laptopa. Dlatego też poniżej jest
trochę info na temat konfiguracji tego typu.

### Drukarka sieciowa (przewodowa/WiFi)

Te nowsze modele drukarek mają zapewne jakieś wsparcie dla sieci, czy to przewodowe czy
bezprzewodowe. Podobnie sprawa wygląda w przypadku HP LaserJet P2055dn. Ta drukarka dysponuje
portem rj-45, do którego można podłączyć skrętkę i podpiąć się bezpośrednio pod router/switch. W
takim przypadku drukarka dostanie własny adres IP i będzie pod nim osiągalna.

Zwykle drukarki oferują szereg usług, które możemy wykorzystać do konfiguracji takich urządzeń
sieciowych. Tak prezentuje się drukarka HP LaserJet P2055dn podczas skanowania jej portów przy
pomocy `nmap` :

    # nmap 192.168.1.211 -p 1-65535
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-21 14:33 CET
    Nmap scan report for 192.168.1.211
    Host is up (0.00030s latency).
    Not shown: 65524 closed ports
    PORT     STATE SERVICE
    23/tcp   open  telnet
    80/tcp   open  http
    280/tcp  open  http-mgmt
    443/tcp  open  https
    515/tcp  open  printer
    3910/tcp open  prnrequest
    5355/tcp open  llmnr
    9032/tcp open  unknown
    9033/tcp open  unknown
    9100/tcp open  jetdirect
    9877/tcp open  x510
    MAC Address: 98:4B:E1:3B:56:7C (Hewlett Packard)

    Nmap done: 1 IP address (1 host up) scanned in 337.50 seconds

Jak widać, ta drukarka oferuje sporo usług sieciowych. Udostępnia ona m.in. panel webowy na porcie
80/280/443, z poziomu którego możemy bez większego problemu skonfigurować praktycznie każdy aspekt
pracy urządzenia. Jest też dostęp shell (via telnet). Niektóre drukarki mogą mieć naturalnie więcej
lub mniej otwartych portów. Tutaj znajduje się [lista portów używanych przez drukarki HP][2]. Sporo
tych wyżej wylistowanych portów jest uwzględniona pod tym linkiem ale paru z nich brakuje, tj. nie
ma 3910/tcp, 5355/tcp, 9032/tcp, 9033/tcp, oraz 9877/tcp.

Szukając trochę informacji na temat tych dodatkowych portów, udało się ustalić, że pod portem
9877/tcp jest dostęp do shell'a `ksh` . Na porty 9032/tcp, 9033/tcp można się dostać przez telnet,
choć komunikacja z drukarką przy ich pomocy [nie jest zbytnio możliwa][5].

Nas na linux interesuje głównie port `9100` (ewentualnie też port `515`) , resztę usług można
wyłączyć w panelu webowym/przez telnet, chyba, że potrzebujemy ich do czegoś. Dobrze jest też
rzucić okiem na te instrukcje dotyczące [zabezpieczenia drukarek sieciowych HP][4]. Oczywiście my
raczej nie będziemy wystawiać drukarki na świat, tak by każdy użytkownik internetu był w stanie coś
u nas drukować ale ja uważam, że jeśli jakichś usług nie potrzebujemy, to lepiej jest je od razu
wyłączyć. Przez panel webowy możemy odznaczyć te poniższe opcje:

![](/img/2021/01/006-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-protokoly.png#huge)

Co ciekawe, nawet po odznaczeniu tych wszystkich widocznych powyżej pozycji, porty 9032/tcp,
9033/tcp, 9877/tcp pozostają aktywne. Nie mam pojęcia jaki jest cel tych portów, co na nich
nasłuchuje i czy da się je w jakiś sposób w ogóle wyłączyć.

Mając na drukarce otwarty port 9100/tcp (lub 515/tcp) możemy bez problemów skonfigurować taką
drukarkę na każdym linux'ie. Trzeba tylko tutaj wyraźnie zaznaczyć, że narzędzia z pakietu
`hplip` / `hplip-gui` wymagają do poprawnej pracy tego pierwszego portu oraz włączonego protokołu
SNMP ([Simple Network Management Protocol][10]). Bez tych dwóch rzeczy, te narzędzia będą mieć
problem z wykryciem drukarki sieciowej. Naturalnie inne narzędzia konfigurujące drukarki w Debianie
czy Ubuntu (np. `system-config-printer` czy CUPS bezpośrednio) bez problemu poradzą sobie z portem
515/tcp. Zatem jeśli zamierzamy korzystać z graficznej nakładki `hplip-gui` , to upewnijmy się że
zarówno port 9100/tcp  w drukarce jest aktywny oraz, że mamy włączony w niej protokół SNMP.

Poniżej przykład konfiguracji drukarki z wykorzystaniem portu 9100 na moim Debianie przy pomocy
graficznej nakładki `hplip-gui` :

![](/img/2021/01/007-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-info-cups.png#huge)
![](/img/2021/01/008-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-siec.png#huge)
![](/img/2021/01/009-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-siec.png#huge)
![](/img/2021/01/010-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-siec.png#huge)

Warto zauważyć, że został tutaj manualnie określony adres IP. Jest też możliwość automatycznej
detekcji drukarki ale jeśli powyłączamy jej zbędne funkcje, m.in. [Service Location Protocol
(SLP)][6], to te automatyczne mechanizmy umożliwiające wykrycie drukarki w sieci nie zadziałają i
trzeba będzie ręcznie określić adres IP, tak jak to zostało zrobione wyżej. W przypadku gdybyśmy
próbowali skorzystać z automatycznej detekcji drukarki w `hplip-gui` nie mając włączonego protokołu
SLP, to przywita nas błąd `HPLIP cannot detect printers in your network` :

![](/img/2021/01/011-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-hplip-siec.png#huge)

### Drukarka sieciowa udostępniana przez router WiFi

W przypadku, gdy wolimy podłączyć drukarkę po USB, to naturalnie możemy ją podłączyć również do
routera WiFi. W ten sposób zostanie ona udostępniona przez router w sieci, co może nam pomóc
odciążyć switch routera, który ma skończoną liczbę gniazdek. Nie trzeba będzie także włączać innego
komputera by skorzystać z drukarki, bo przecie router WiFi zwykle pracuje 24h/d, no i do tego
pobiera sporo mniej prądu niż taki pełnowymiarowy komputer. Jeśli zdecydujemy się na takie
rozwiązanie, to pamiętajmy, by na routerze z wgranym firmware OpenWRT doinstalować pakiet `p910nd`
([dokładna instrukcja konfiguracji][9] jest opisana tutaj) . Następnie w pliku `/etc/config/p910nd`
dodajemy poniższą treść

    config p910nd
            option device '/dev/usb/lp0'
            option port '0'
            option bidirectional '1'
            option runas_root '0'
            option mdns '0'
            option mdns_ty 'My Printer Manufacturer/Model'
            option mdns_note 'Basement'
            option enabled '1'
            option bind '192.168.1.1'

Zapisujemy plik, dodajemy demona do autostartu i uruchamiamy go:

    root@RedViper:~# /etc/init.d/p910nd enable
    root@RedViper:~# /etc/init.d/p910nd start

Następnie trzeba będzie dodać na komputerze drukarkę dokładnie w taki sam sposób jak dodajemy każdą
inną drukarkę sieciową, np. via panel webowy CUPS (opis poniżej). Trzeba tylko pamiętać, by podać w
adresie IP adres określony wyżej w pliku konfiguracyjnym, tj. `192.168.1.1` (adres routera) oraz
odpowiedni port. W tym przypadku w konfiguracji `p910nd` został podany port `0` , który odpowiada
portowi 9100 i to pod takim adresem/portem będzie dostępna drukarka na routerze.

Warto zaznaczyć tutaj, że najwyraźniej narzędzia z pakietu `hplip` / `hplip-gui` nie chcą
współpracować z tak udostępnianą drukarką i jej zwyczajnie nie wykrywają. CUPS naturalnie nie ma z
nią żadnych problemów.

### Konfiguracja drukarki w CUPS

Praktycznie każdą drukarką na linux można [zarządzać za pomocą panelu webowego CUPS][7] dostępnego
pod adresem `http://127.0.0.1:631` , który trzeba wpisać w pasku adresu przeglądarki. W przypadku
drukarki Hewlett Packard LaserJet P2055dn również możemy skorzystać bezpośrednio z CUPS.
Przechodzimy zatem na podany wyżej adres i dodajemy nową drukarkę `AppSocket/HP JetDirect` podając
jej adres IP (w tym przypadku 192.168.1.211) oraz port 9100:

![](/img/2021/01/012-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)
![](/img/2021/01/013-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups-appsocket-hp-jetdirect.png#huge)
![](/img/2021/01/014-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)

Następnie wybieramy producenta drukarki oraz jej model:

![](/img/2021/01/015-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)
![](/img/2021/01/016-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)

Po chwili drukarka powinna zostać z powodzeniem dodana:

![](/img/2021/01/017-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)

Możemy naturalnie wydrukować stronę testową, by sprawdzić czy wszytko działa jak należy.

Co ciekawe, CUPS również jest w stanie wyszukać dostępne drukarki sieciowe ale [najwyraźniej trzeba
go jakoś odpowiednio skonfigurować][8]. W moim przypadku CUPS nie potrafił znaleźć żądnej drukarki
sieciowej. Potrafił za to znaleźć drukarkę podpiętą do portu USB:

![](/img/2021/01/018-drukarka-linux-hewlett-packard-laserjet-p2055dn-hp-cups.png#huge)

Wygląda też na to, że drukarki dodane przez CUPS nie są widoczne w `hplip-gui` . Z kolei drukarki
dodane przez `hplip-gui` są widoczne w panelu webowym CUPS. Tak czy inaczej, bez znaczenia w jaki
sposób dodamy drukarkę do systemu, to będzie można z niej bez problemu korzystać i drukować
dokumenty w różnych aplikacjach.

## Zdalne logowanie komunikatów drukarki (syslog)

Standardowo drukarka HP LaserJet P2055dn ma zaszyty w sobie demon logowania syslog ale jest on w
stanie logować komunikaty jedynie lokalnie, tj. w obrębie systemu drukarki. Przynajmniej do takiego
wniosku można dojść sądząc po tych poniższych informacjach w webowym panelu konfiguracyjnym
drukarki:

    Funkcja Syslog:     LPR
    Serwer Syslog:      Nie określone

Próżno jednak szukać w tym panelu ustawień od serwera syslog. Jeśli chcemy skonfigurować adres IP
tego serwera, to musimy to zrobić przez protokół telnet. Odpalamy zatem w naszym linux'ie terminal
i wpisujemy w nim  `telnet 192.168.1.211` , gdzie adres IP wskazuje na adres IP drukarki, pod
którym jest ona dostępna w naszej sieci. Po podłączeniu się do drukarki, wpisujemy `/` by podejrzeć
aktualną konfigurację ustawień. Powinniśmy mieć tam m.in. te poniższe wpisy:

    > /
    ...
          TCP/IP OTHER_______________________________
          Syslog Config      : Enabled
          Syslog Server      : Not Specified
          Syslog MaxMsg/Min  : 10
          Syslog Priority    : 7
    ...

Serwer określamy podając nazwę parametru `syslog-svr` , w argumencie którego podajemy adres IP
serwera z uruchomionym demonem syslog. Dobrze jest też podbić limit wiadomości, które mogą zostać
wygenerowane w ciągu jednej minuty do wartości maksymalnej, tj. 1000, tak by nie przegapić żadnych
komunikatów, przynajmniej w pierwszych dniach pracy drukarki:

    > syslog-svr 192.168.1.150
    > syslog-max 1000

    > /
    ...
          TCP/IP OTHER_______________________________
          Syslog Config      : Enabled
          Syslog Server      : 192.168.1.150
          Syslog MaxMsg/Min  : 1000
          Syslog Priority    : 7

Nie mamy tutaj możliwości konfiguracji portu, zatem będzie wykorzystywany domyślny port dla usługi
syslog, tj. 514/udp.

Zapisujemy jeszcze ustawienia i restartujemy drukarkę ponownie:

    > save

Drukarka powinna już próbować komunikować się ze zdalnym demonem syslog, co można zauważyć np. w
`wireshark`:

![](/img/2021/01/019-drukarka-linux-hewlett-packard-laserjet-p2055dn-syslog.png#huge)

Jeśli rsyslog w Debianie nie wyświetla żadnych komunikatów otrzymanych od drukarki, to upewnijmy
się, że w konfiguracji tego demona w pliku `/etc/rsyslog.conf` widnieje poniższy wpis:

    module(load="imudp")
    input(type="imudp" port="514")

Po zresetowaniu demona rsyslog, powinien on nasłuchiwać zapytań także na porcie 514/udp:

    # netstat -napletu | grep -i sysl
    tcp        0      0 0.0.0.0:514   0.0.0.0:*     LISTEN      0    12400110   1406821/rsyslogd
    tcp6       0      0 :::514        :::*          LISTEN      0    12400111   1406821/rsyslogd
    udp        0      0 0.0.0.0:514   0.0.0.0:*                 0    12399535   1406821/rsyslogd
    udp6       0      0 :::514        :::*                      0    12399536   1406821/rsyslogd

Najwyraźniej ta usługa syslog drukarki HP LaserJet P2055dn jest bardzo uboga, bo nie zauważyłem, by
tutaj były przesyłane jakiekolwiek komunikaty odnośnie zdarzeń zachodzących w systemie, np.
informacje co drukarka w danej chwili robi. Jedyne komunikaty jakie można zauważyć, to te widoczne
w logu `wireshark` i to w zasadzie cała użyteczność tej usługi.


[1]: https://hackaday.com/2012/03/05/resetting-the-page-count-on-a-laser-printer/
[2]: https://support.hp.com/nz-en/document/c02480766
[3]: https://developers.hp.com/hp-linux-imaging-and-printing
[4]: https://support.hp.com/pl-pl/document/c03802198
[5]: https://forum.dug.net.pl/viewtopic.php?pid=330214#p330214
[6]: https://en.wikipedia.org/wiki/Service_Location_Protocol
[7]: /post/cups-konfiguracja-drukarki-pod-linuxem/
[8]: https://github.com/OpenPrinting/cups-filters/issues/10
[9]: /post/drukarka-sieciowa-w-openwrt-serwer-wydruku/
[10]: https://pl.wikipedia.org/wiki/Simple_Network_Management_Protocol
[11]: https://support.hp.com/pl-pl/document/c04615809
