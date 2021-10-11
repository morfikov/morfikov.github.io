---
author: Morfik
categories:
- Android
date:    2016-10-18 22:53:19 +0200
lastmod: 2016-10-18 22:53:19 +0200
published: true
status: publish
tags:
- smartfon
- aplikacje
- f-droid
- kamera
- aparat
GHissueID: 448
title: 'Android: OpenCamera i aparat w Neffos C5'
---

Ci z was, którzy czytali moją [recenzję na temat smartfona Neffos C5][1] od TP-LINK, widzą, że
niezbyt spodobał mi się aparat/kamera zaimplementowany w tym telefonie. Niby jest tutaj 8 mpix na
aparacie głównym (i 5 mpix na selfie) ale przy niezbyt dobrym oświetleniu jakość zdjęć siada dość
znacznie. Abstrahując od samej jakości aparatu, chciałbym się nieco bardziej skupić na
oprogramowaniu do jego obsługi, które Neffos C5 oferuje. Jest ono dość ubogie pod względem
funkcjonalności i mi generalnie przydałoby się nieco więcej opcji, z których mógłbym zrobić jakiś
użytek. Jest wiele aplikacji na Androida, które oferują poszerzenie możliwości aparatu czy kamery w
telefonie. Większość z nich zawiera jednak reklamy, które niezbyt pasują na smartfonie
wyrafinowanego linux'iarza. Postanowiłem zatem poszukać nieco głębiej i w [repozytorium
F-Droid'a][2] znalazłem [OpenCamera][3]. Programik bardzo przyzwoity, bez reklam, no i najważniejsze
jest on OpenSource. W tym artykule rzucimy sobie okiem na ten kawałek oprogramowania i zobaczymy
jaką funkcjonalność ono oferuje.

<!--more-->
## Standardowe oprogramowanie aparatu/kamery w Neffos C5

Przeanalizujmy sobie pierw standardowe oprogramowanie kamery/aparatu, które jest zainstalowane
domyślnie w Neffos C5. Po wejściu w aplikację "Aparat" przywita nas bardzo prosty interfejs z
możliwością łatwego przełączania się miedzy trybem aparatu i kamery. Z głównego okna interfejsu
mamy też możliwość wyboru aparatu głównego lub selfie:

![](/img/2016/10/001.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-interfejs-standard.png#medium)

### Aparat główny

Sprawdźmy zatem jakie opcje zostały nam oddane do dyspozycji jeśli chodzi o aparat główny. Przede
wszystkim, mamy możliwość dostosowania lampy błyskowej. Możemy ją wyłączyć, włączyć na stałe, lub
też zostawić tę kwestię smartfonowi, który na podstawie danych z czujnika światła oceni czy lampa
ma zostać włączona podczas robienia zdjęcia. Możemy także wybrać sobie filtr w celu lepszego efektu
wizualnego robionych fotek:

![](/img/2016/10/002.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-filtry.png#huge)

Z lewej krawędzi ekranu możemy także wyciągnąć menu, w którym to mamy możliwość ustawienia kilku
przydanych opcji. Możemy tutaj włączyć stempel czasu, za sprawą którego na fotce zostanie
uwzględniona data i godzina (w prawym dolnym rogu). Możemy również ustawić samowyzwalacz 5/10
sekund, który sygnalizuje odliczany czas napisami na ekranie oraz też dźwiękiem na chwilę przed
cyknięciem fotki.

Mamy także możliwość dostosowania trybu robienia zdjęć. Teoretycznie to oprogramowanie umożliwia
robienie zdjęć po tapnięciu w ekran, jak przy wykryciu ruchu czy uśmiechu. Te dwa ostatnie u mnie
nie działają lub też nie mam pojęcia jak je zmusić do działania. :D No i oczywiście fotki można
robić przez wciśnięcie niebieskiego przycisku widocznego na ekranie jak i za pomocą przycisków
Volume Up/Down na obudowie smartfona.

Z poziomu opcji aparatu możemy także wyłączyć dźwięk migawki co daje nam możliwość robienia zdjęć z
ukrycia. Bardzo przydatna rzecz.

Aparat główny ma 8 mpix ale to nie znaczy, że musimy robić fotki w takiej rozdzielczości. Im większa
rozdziałka, tym większy rozmiar zdjęcia, a przecie nie wszystkie fotki wymagają takiej
szczegółowości. W domyślnym oprogramowaniu Neffos'a C5 mamy możliwość przełączania się między 8
mpix (3264x2448 px, 4:3), 5 mpix (2560x1920 px, 4:3), 4 mpix (2560x1440 px, 16:9), 3 mpix (2048x1536
px, 4:3) i 2 mpix (1600x1200 px, 4:3).

Wybrane ustawienia zawsze można przywrócić do domyślnych za pomocą jednego prostego tapnięcia w
ekran. Natomiast nie mamy możliwości dostosowania zachowania autofokusa. Przy repozycjonowaniu
aparatu, pojawia nam się białe kółko, które po wyostrzeniu obrazu przybiera kolor zielony, po czym
znika. Można naturalnie wybrać obszar na którym aparat ma się skupić przed zrobieniem fotki stukając
w interesującą nas część ekranu.

![](/img/2016/10/003.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-interfejs.png#huge)

### Kamera główna

Aparat możemy także przełączyć w tryb kamery, w efekcie czego będziemy mieli możliwość nagrywania
filmów. Jeśli chodzi zaś o dostępne opcje w trybie kamery, to jest ich niewiele. Mamy możliwość
włączenia i wyłączenia mikrofonu, przez co na nagraniu może pojawić się ścieżka dźwiękowa. Jeśli
zaś chodzi o jakoś nagrywanego materiału video, to mamy do dyspozycji trzy opcje: wysoka (1920x1080
px, 30 FPS, 128 kbit/s audio), średnia (1280x720 px, 30 FPS, 128 kbit/s audio) i niska (640x480 px,
30 FPS, 128 kbit/s audio). W ustawieniach aparatu nie mamy informacji na temat rozdzielczości i
bitrate, choć ten można wyciągnąć z informacji o pliku a/v. Jest też możliwość włączenia lampy
błyskowej na stałe podczas kręcenia filmu, choć nie ma możliwości wyłączenia jej w trakcie
nagrywania.

![](/img/2016/10/004.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje-video.png#medium)

### Aparat selfie

Neffos C5 dysponuje także aparatem selfie 5 mpix. Nie posiada on jednak autofokusa w stosunku do
głównego aparatu i ma trochę mniej opcji. Niemniej jednak, opcja stempla czasu, samowyzwalacza,
trybu robienia zdjęć jak i dźwięku migawki jest dokładnie taka sama co w przypadku głównego aparatu.
Jedynymi widocznymi różnicami ą uszczuplony wybór filtrów, jak i rozdzielczość robionych fotek. W
tym przypadku mamy do wyboru 5 mpix (2560x1920 px, 4:3), 4 (2560x1440 px, 16:9) mpix, 3 (2048x1536
px, 4:3) mpix, 2 (1600x1200 px, 4:3) mpix, 1,3 mpix (1280x960 px, 4:3).

![](/img/2016/10/005.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-selfie.png#huge)

### Kamera selfie

Po przełączeniu aparatu selfie w tryb kamery, praktycznie nie mamy dostępu do żadnych opcji za
wyjątkiem zmiany rozdzielczości materiału video, która może być wysoka (640x480 px, 30 FPS, 128
kbit/s audio) albo niska (176x144 px, 30 FPS, 64 kbit/s audio) oraz też włączenia i wyłączenia
mikrofonu.

![](/img/2016/10/006.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje-video.png#medium)

Jak widać, to oprogramowanie, które jest domyślnie zainstalowane w Neffos C5 daje nam jedynie
podstawową funkcjonalność aparatu i kamery. Te wyżej wymienione opcje raczej powinny zadowolić
przeciętnych użytkowników smartfonów. Ja jednak potrzebuję czegoś więcej. Dla przykładu weźmy
geotagowanie zdjęć na podstawie aktualnego położenia z GPS. Idąc dalej, chciałbym mieć także
możliwość dostosowania stopnia kompresji fotek jak i bitrate video. Sprawdźmy zatem co może nam
zaoferować alternatywne oprogramowanie w postaci OpenCamera.

## Instalacja oprogramowania OpenCamera

W wstępie nadmieniłem, że aplikacja OpenCamera jest dostępna w alternatywnym repozytorium aplikacji
jakim jest F-Droid. Oczywiście nie musimy instalować sobie od razu F-Droid'a, bo [OpenCamera jest
także dostępny w sklepie Google Play][4]. Poniżej jest ofotkowany proces instalacji:

![](/img/2016/10/007.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-instalacja.png#huge)

A tak z kolei wygląda interfejs OpenCamera:

![](/img/2016/10/008.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-intefejs.png#medium)

Informacja na temat aktualnie ustawionej rozdzielczości fotki/video zawsze jest pokazywana po
uruchomieniu aplikacji. Praktycznie wszystkie wyżej widoczne elementy interfejsu można sobie
dowolnie dostosować, wliczając w to całkowite ich ukrycie.

## Konfiguracja aparatu/kamery z OpenCamera

Przejdźmy zatem do tej kwestii, którą linux'iarze lubią najbardziej, tj. do opcji konfiguracyjnych,
które oferuje OpenCamera. W porównaniu do opcji dostępnych w standardowej aplikacji będącej na
wyposażeniu Neffos'a C5, przeciętny użytkownik telefonu może czuć się nieco przytłoczony, bo jakby
nie patrzeć opcji jest naprawdę sporo:

![](/img/2016/10/009.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje.png#huge)

![](/img/2016/10/010.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje.png#huge)

To co odróżnia OpenCamera od standardowego oprogramowania zawartego w Neffos C5, to przede wszystkim
możliwość dostosowania sobie praktycznie każdego aspektu pracy aparatu czy kamery. Mamy po prostu
większą dowolność i swobodę przy dostosowaniu poszczególnych parametrów.

Praktycznie cała funkcjonalność domyślnej aplikacji aparatu została w OpenCamera zaimplementowana.
Takie opcje jak wykrywanie twarzy, robienie zdjęć poprzez dotyk ekranu, powiadamianie dźwiękowe przy
robieniu zdjęcia, czy reset ustawień do domyślnych są naturalnie dostępne w OpenCamera. Są też
opcje, których w standardowym oprogramowaniu nie znajdziemy. Poniżej jest krótka ich lista.

### Dostosowanie czasu wyzwalacza

Czas wyzwalacza w domyślnym sofcie można było również dostosować. Z tym, że mieliśmy do wyboru
między 5 i 10 sekund. W przypadku OpenCamera również możemy ustawić 5 i 10 sekund ale mamy także do
dyspozycji 1, 2, 3, 15, 20, 30 sekund oraz 1, 3 i 5 minut.

![](/img/2016/10/011.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-timer.png#huge)

Przy robieniu zdjęć z opóźnieniem mamy także opcję beeper'a, który nam będzie sygnalizował dźwiękiem
upływający czas (co sekundę). Jest też możliwość aktywowania odliczania głosowego i wtedy system
ludzkim głosem nam to odliczanie przeprowadzi (<60 sekund). Te dwie funkcje mogą być włączone
równocześnie ale też można z nich zrezygnować zupełnie, przez co odliczanie będzie przeprowadzane
po cichu. Wciąż jednak będziemy widzieli na ekranie upływający czas.

### Robienie kilku zdjęć pod rząd

Inną ciekawą opcją jest robienie szeregu zdjęć jedno po drugim, tzw. burst. Co ciekawe, mamy
możliwość określenia co ile sekund ma być robione kolejne zdjęcie oraz ile fotek życzymy sobie
pstryknąć naraz. Nie ma minimalnego interwału, natomiast maksymalny to 2 godziny. Jeśli zaś chodzi o
ilość robionych zdjęć, to mogą być to dwie, trzy, cztery, pięć, dziesięć lub bez ograniczeń. Trzeba
tylko pamiętać o autofokusie, który między fotkami może wymagać czasu na wyostrzenie zdjęcia, przez
co może się pojawić dodatkowe opóźnienie.

![](/img/2016/10/016.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-burst.png#huge)

### Głosowa aktywacja aparatu/kamery

Mizianie ekranu, czy wciskanie przycisków by zrobić zdjęcie czy rozpocząć rejestrowanie obrazu
wymaga od nas fizycznego kontaktu z urządzeniem. Możemy jednak sterować aparatem/kamerą przy pomocy
dźwięku. Generalnie są dwie opcje, z których możemy skorzystać: słówko "cheese" lub głośny hałas. By
być szczerym "cheese" średnio działa, natomiast nie udało mi się aktywować ani aparatu, ani kamery
głośnymi dźwiękami. Nie wiem czy to problem z samą aplikacją czy moim Neffos'em C5.

![](/img/2016/10/012.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-kontrola-dzwiekiem.png#huge)

### Przypisywanie akcji pod klawisze Volume Up/Down

Standardowo do klawiszy Volume Up/Down jest przypisana akcja robienia zdjęcia i w domyślnym
oprogramowaniu nie możemy jej zmienić. Jeśli zaś chodzi o OpenCamera, to tutaj te przyciski możemy
całkowicie wyłączyć. Możemy też przy ich pomocy robić fotki czy zainicjować/zatrzymywać nagrywanie
materiału video. Możemy także zmienić poziom ekspozycji jak i zwiększyć/zmniejszyć zoom. Jest też
opcja włączenia i wyłączenia stabilizacji obrazu oraz zmiany głośności urządzenia. Możemy także
aktywować fokus, z tym, że zawsze będzie on skupiał się na środkowej części obszaru robionego
zdjęcia.

![](/img/2016/10/013.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-przyciski.png#medium)

### Ścieżki do zapisu i nazwy plików

To co mi się także bardzo podoba w OpenCamera, to możliwość wyboru ścieżki zapisu zdjęć czy nagrań
video. Nie tylko możemy stworzyć sobie drzewo katalogów i zapisywać fotki z pewnych sytuacji w
konkretnych folderach ale także możemy przenieść tworzony materiał na kartę SD. Problem w tym, że
Android od wersji 4.4 nie zezwoli OpenCamera na zapis fotek na karcie. Można jednak to obejść [przez
uchylenie pewnych obostrzeń][5] ale do tego jest znowu potrzebny [root smartfona][6].

Kolejna sprawa, to nazwy zapisywanych plików. Przede wszystkim, jesteśmy w stanie zmienić prefiks
nazwy pliku. Dla zdjęć jest osobny prefiks, a dla materiałów video osobny. Nazwa pliku zawsze
przybierze format daty i czasu. My natomiast możemy ten czas dostosować w oparciu o czas lokalny lub
też o czas UTC. Dla Polski różnica będzie wynosić jedną lub dwie godziny (czas zimowy/letni).

![](/img/2016/10/015.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-nazwy-plikow.png#huge)

Te powyższe opcje dają nam możliwość grupowania fotek/filmów bezpośrednio na telefonie w czasie
rzeczywistym, co z pewnością przekłada się na lepszą organizację plików w telefonie, przez co
łatwiej możemy odszukać interesujące nas materiały.

### Opcje specyficzne dla aparatu (zdjęcia)

Przejdźmy zatem do opcji specyficznych dla aparatu. które możemy ustawić. Jest ich również dość
sporo:

![](/img/2016/10/017.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje-aparatu.png#huge)

#### Rozdzielczość robionych zdjęć

W przypadku domyślnego oprogramowania mieliśmy do dyspozycji tylko kilka rozdzielczości aparatu
określanych w mpix (megapikselach). W przypadku OpenCamera jest nieco inaczej. Tak samo mamy
informację o mpix ale dodatkowo mamy też określoną rozdzielczość fotki oraz proporcje ekranu. Te
informacje naprawdę ułatwiają rozeznanie co do tego jaką fotkę spodziewamy się zrobić:

![](/img/2016/10/018.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-rozdzielczosc-zdjec.png#huge)

#### Stopień kompresji zdjęć

OpenCamera udostępnia nam możliwość kontroli kompresji plików `.jpg` , przez co możemy wpływać na
jakość robionych zdjęć. Jesteśmy w stanie ustawić tutaj zarówno jakość zdjęcia na 100% (brak
kompresji) jak i na 1% (duży stopień kompresji). Możemy również wybrać pośredni stopień kompresji.
Domyślnym jest 90%:

![](/img/2016/10/019.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-jakosc-zdjec.png#huge)

#### Znakowanie zdjęć

Sprawa oznaczeń zdjęć też wygląda o wiele lepiej w stosunku do standardowego oprogramowania. Możemy
sobie wybrać format daty (np. rrrr/mm/dd) jak i format czasu (12/24/brak). Gdy robimy zdjęcia przy
włączonym GPS, możemy także na fotce zawrzeć informacje o
lokalizacji.

![](/img/2016/10/020.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-znakowanie-zdjec.png#huge)

Jakby tego było mało to możemy umieścić własny tekst na zdjęciu oraz dostosować styl czcionki jej
kolor i wielkość:

![](/img/2016/10/021.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-format-znakowania.png#huge)

Po ustawieniu wszystkich tych opcji otrzymamy mniej więcej taki podpis:

![](/img/2016/10/023.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-oznaczenie.png#medium)

#### Pauzowanie po zrobionym zdjęciu

Gdy robimy zdjęcia, to są one automatycznie zapisywane na flash (ew. kartę SD) telefonu. Jeśli
znajdujemy się w sytuacji takiej, że po cyknięciu fotki chcemy coś z nią zrobić, to możemy
zaoszczędzić trochę czasu zaznaczając opcję pauzowania. W takim przypadku po cyknięciu fotki ekran
zostanie zamrożony, a my będziemy mieli podgląd zdjęcia, które właśnie zostało wykonane. Ponadto
będziemy mieli także opcję skasowania pliku czy podzielenia się nim w sieci.

![](/img/2016/10/022.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-pauza.png#medium)

### Opcje specyficzne dla kamery (obraz)

Opcje zdjęć mamy z grubsza z głowy. Pora zając się opcjami video. Na dobrą sprawę mamy ich jeszcze
więcej niż w przypadku fotek.

![](/img/2016/10/024.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-opcje-video.png#huge)

#### Rozdzielczość, bitrate i FPS video

W standardowym oprogramowaniu kamery mieliśmy możliwość wyboru kilku trybów video. Wszystkie z nich
były określone w mpix. W przypadku OpenCamera mamy również informację o danej rozdzielczości użytej
przy nagrywaniu materiału video. Możemy także określić jakość takiego materiału definiując ręcznie
bitrate obrazu. Jest też opcja od ustawienia niestandardowej ilości FPS (klatek na sekundę):

![](/img/2016/10/025.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-rozdzielczosc-jakosc-fps.png#huge)

Może i mamy bardzo dużą swobodę w dostosowaniu tych trzech powyższych parametrów ale trzeba sobie
zdawać sprawę, że nasz smartfon niekoniecznie musi obsługiwać dany tryb pracy kamery. Generalnie to
jesteśmy tutaj ograniczeni przez możliwości sprzętowe.

#### Maksymalna długość/rozmiar video i automatyczny restart nagrywania

Ciekawą funkcją tez jest ustawienie maksymalnej długości czy rozmiaru nagrywanego materiału video.
Możemy także określić czy po osiągnięciu ustawionego progu ma nastąpić restart nagrywania,
oczywiście materiał będzie zapisywany w osobnym pliku.

![](/img/2016/10/026.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-czas-trwania.png#huge)

![](/img/2016/10/027.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-restart.png#huge)

#### Określenie źródła nagrywanego dźwięku

W przypadku, gdy zamierzamy nagrywać jedynie obraz video, możemy bez większego problemy wyłączyć
nagrywanie dźwięku. Jeśli jednak chcemy, by w nagraniu się pojawił dźwięk, to mamy możliwość
określenia źródła dźwięku:

![](/img/2016/10/028.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-zrodlo-audio.png#huge)

#### Blokada ekranu przed przypadkowym wyłączeniem nagrywania

Kolejną ciekawą funkcją jest blokada ekranu tuż po wciśnięciu przycisku nagrywania filmu. W ten
sposób nie będziemy mieli możliwości wyłączenia nagrywania, no chyba, że wcześniej odblokujemy
ekran.

![](/img/2016/10/029.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-zabezpieczenie.png#medium)

Szkoda tylko, że przyciski w dalszym ciągu są w stanie wyłączyć nagrywanie.

#### Zatrzymanie nagrywania, gdy stan baterii jest krytycznie niski

Gdy nagrywamy jakiś materiał video, to najgorszą rzeczą jaka może nam się przytrafić jest
wyczerpanie się baterii. W takiej sytuacji jest niemal pewne, że film zostanie uszkodzony i stracimy
cały nagrany materiał albo jakąś jego cześć. Można się przed tym zabezpieczyć przerywając nagrywanie
przed wyczerpaniem się baterii (~3%). OpenCamera oferuje nam tego typu funkcjonalność.

![](/img/2016/10/030.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-bateria.png#medium)

#### Dioda sygnalizująca proces nagrywania

Jeśli zamierzamy zostawić naszego smartfona gdzieś w oddali w celu uwiecznienia zdarzeń na filmie,
możemy włączyć opcję sygnalizowania procesu nagrywania przy pomocy lampy błyskowej. Wtedy nawet
siedząc kilkanaście metrów od telefonu wystarczy spojrzeć na niego by być pewnym, że nic nie
przerwało nagrywania.

![](/img/2016/10/031.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-flash.png#medium)

### Opcje lokalizacji (GPS)

Kolejną grupą ustawień są te dotyczące lokalizacji. Standardowo geotagowanie zdjęć/filmów jest
wyłączone. Możemy je nie tylko włączyć ale również je wymusić, przez co nie będziemy w stanie
zrobić zdjęcia, gdy funkcja GPS będzie wyłączona. W zdjęciach możemy zaszyć także informacje ze
wskazania kompasu.

![](/img/2016/10/032.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-gps-geotagging.png#huge)

Poniżej się prezentują dane EXIF, które można wyciągnąć po włączeniu GPS i otagowaniu obrazka przez
aplikację OpenCamera:

![](/img/2016/10/033.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-dane-gps.png#huge)

### Dostosowanie interfejsu OpenCamera

Ostatnią grupą opcji są te dotyczące wyglądu interfejsu graficznego aparatu/kamery. Możemy usunąć
czarne obramowanie, tak by podgląd obrazu wypełnił całą dostępną przestrzeń ekranu. Z kolei jeśli
zaś chcemy zmienić pozycję menu interfejsu, to możemy przenieść je z prawej strony na lewą. Możemy
także schować cały interfejs jeśli nam on przeszkadza. Oczywiście będziemy w stanie wywołać
interfejs w każdej chwil po tapnięciu w ekran. Jest także możliwość pokazania/ukrycia tylko pewnych
części interfejsu, np. przycisków/paska zoom, informacji o ilości wolnego miejsca na pamięci flash,
aktualnego czasu czy stanu baterii. Jest tez opcja zapobiegania wyłączeniu ekranu, gdy włączony jest
podgląd z kamery/aparatu. Jeśli natomiast drażni nas nieco maksymalna jasność ekranu w trybie
robienia zdjęć/video to też możemy tę funkcję łatwo tutaj wyłączyć.

![](/img/2016/10/034.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-osd-opcje.png#huge)

Te powyższe opcje dotyczą jedynie samego wyglądu interfejsu aparatu/kamery. Niemniej jednak,
istnieją jeszcze opcje, do których możemy uzyskać dostęp z poziomu tego interfejsu:

![](/img/2016/10/035.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-osd.png#huge)

Z lewej lub prawej strony mamy kilka ikonek. Licząc od góry, odpowiadają one za przełączenie aparatu
głównego/selfie, przełączenie trybu aparatu/kamery, dostosowanie i zablokowanie poziomu ekspozycji,
dostęp do bardziej rozbudowanego menu interfejsu oraz menu opcji. Jeśli zaś chodzi o rozbudowane
menu interfejsu jakim dysponuje aparat/kamera, to wygląda on mniej więcej tak:

![](/img/2016/10/036.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-osd.png#huge)

![](/img/2016/10/037.opencamera-kamera-aparat-smartfon-neffos-c5-tp-link-osd.png#huge)

Trochę tego jest i to przewijane menu jest moim zdaniem trochę zbyt długie. Przydałby się je pociąć
i ulokować pod osobnymi ikonkami na pasku interfejsu. Niemniej jednak, widzimy, że w bardzo łatwy
sposób można wybrać sobie filtry jak i rozdziałkę kamery/aparatu.

Co ciekawe lampa błyskowa może być włączona cały czas, nawet przed zrobieniem zdjęcia. Szkoda tylko,
że miga ona tuż przed zrobieniem fotki. Jeśli chodzi zaś opcje dotyczące ostrości obrazu, to mamy
możliwość nie tylko włączenia/wyłączenia autofokusa ale także możemy ustawić go na pewien określony
punkt i zablokować to ustawienie. Jest także [Focus Infinity][7] oraz możliwość dostosowania
wartości ISO.

### Informacje o aparacie/kamerze

Jako, że nie wszystkie wyżej opisane funkcje są dostępne w każdym smartfonie, to warto zaznajomić
się z informacjami, które widnieją pod pozycją "About" w menu opcji OpenCamera. Dla modelu Neffos
C5 od TP-LINK są one następujące:

    Open Camera v1.34
    Code: 41
    (c) 2013-2016 Mark Harman
    Released under the GPL v3 or later
    Package: net.sourceforge.opencamera
    Android API version: 22
    Device manufacturer: TP-LINK
    Device model: Neffos C5
    Device code-name: mt6735
    Device variant: C5
    Language: pl
    Standard max heap?: 128
    Large max heap?: 256
    Display size: 720x1184
    Current camera ID: 0
    Camera API: Camera
    Preview resolutions: 176x144, 320x240, 352x288, 480x320, 480x368, 640x480, 720x480, 800x480, 800x600, 864x480, 960x540, 1280x720, 1440x1080, 1920x1080, 1920x1088, 1680x1248
    Preview resolution: 1280x720
    Photo resolutions: 320x240, 640x480, 1024x768, 1280x720, 1280x768, 1280x960, 1600x1200, 1920x1088, 2048x1536, 2560x1440, 2560x1920, 3264x2448
    Photo resolution: 3264x2448
    Video qualities: 1, 5_r1920x1080, 5_r1280x736, 5, 4_r864x480, 4, 3_r640x480, 3_r480x320, 3, 7, 2
    Video resolutions: 1920x1088, 1920x1080, 1280x736, 1280x720, 864x480, 720x480, 640x480, 480x320, 352x288, 320x240, 176x144
    Video quality: 5_r1920x1080
    Video frame width: 1920
    Video frame height: 1080
    Video bit rate: 9000000
    Video frame rate: 30
    Auto-stabilise?: Available
    Auto-stabilise enabled?: false
    Face detection?: Available
    RAW?: Not available
    Video stabilization?: Not available
    Flash modes: flash_off, flash_auto, flash_on, flash_torch, flash_red_eye
    Focus modes: focus_mode_auto, focus_mode_infinity, focus_mode_macro, focus_mode_locked, focus_mode_continuous_picture, focus_mode_continuous_video
    Color effects: none, mono, negative, sepia, aqua, whiteboard, blackboard, posterize, nashville, hefe, valencia, xproll, lofi, sierra, walden
    Scene modes: auto, portrait, landscape, night, night-portrait, theatre, beach, snow, sunset, steadyphoto, fireworks, sports, party, candlelight, hdr
    White balances: auto, incandescent, fluorescent, warm-fluorescent, daylight, cloudy-daylight, twilight, shade
    ISOs: auto, 100, 200, 400, 800, 1600
    ISO key: iso-speed
    Using SAF?: false
    Save Location: OpenCamera
    Save Location SAF:
    Parameters: 3dnr-mode=on;
    3dnr-mode-values=on,off;
    afeng-max-focus-step=1023;
    afeng-min-focus-step=0;
    aflamp-mode=off;
    aflamp-mode-values=off,on,auto;
    antibanding=off;
    antibanding-values=off,50hz,60hz,auto;
    auto-exposure-lock-supported=true;
    auto-whitebalance-lock-supported=true;
    brightness=middle;
    brightness-values=low,middle,high;
    brightness_value=0;
    burst-num=1;
    cap-mode=normal;
    cap-mode-values=normal,face_beauty,continuousshot,smileshot,bestshot,autorama,mav,asd,motiontrack;
    capfname=/sdcard/DCIM/cap00;
    chutter-value=0;
    contrast=middle;
    contrast-values=low,middle,high;
    cshot-indicator=true;
    cshot-indicator-supported=true;
    dynamic-frame-rate=true;
    dynamic-frame-rate-supported=true;
    edge=middle;
    edge-values=low,middle,high;
    effect=none;
    effect-values=none,mono,negative,sepia,aqua,whiteboard,blackboard,posterize,nashville,hefe,valencia,xproll,lofi,sierra,walden;
    eng-mfll-e=false;
    eng-mfll-s=true;
    eng-s-shad-t=0;
    eng-shad-t=0;
    exposure-compensation=0;
    exposure-compensation-step=1.0;
    face-beauty=false;
    face-beauty-supported=true;
    fb-enlarge-eye=0;
    fb-enlarge-eye-max=4;
    fb-enlarge-eye-min=-4;
    fb-extreme-beauty=true;
    fb-extreme-beauty-supported=false;
    fb-face-pos=-2000:-2000;
    fb-sharp=0;
    fb-sharp-max=12;
    fb-sharp-max-values=12;
    fb-sharp-min=-12;
    fb-sharp-min-values=-12;
    fb-skin-color=0;
    fb-skin-color-max=12;
    fb-skin-color-max-values=12;
    fb-skin-color-min=-12;
    fb-skin-color-min-values=-12;
    fb-slim-face=0;
    fb-slim-face-max=12;
    fb-slim-face-max-values=12;
    fb-slim-face-min=-12;
    fb-slim-face-min-values=-12;
    fb-smooth-level=0;
    fb-smooth-level-max=12;
    fb-smooth-level-max-values=12;
    fb-smooth-level-min=-12;
    fb-smooth-level-min-values=-12;
    fb-touch-pos=-2000:-2000;
    feature-max-fps=24@VFB+EIS;
    flash-duty-max=1;
    flash-duty-min=0;
    flash-duty-value=-1;
    flash-mode=off;
    flash-mode-values=off,on,auto,red-eye,torch;
    flash-step-max=0;
    flash-step-min=0;
    focal-length=3.5;
    focus-areas=(0,0,0,0,0);
    focus-distances=0.95,1.9,Infinity;
    focus-fs-fi=0;
    focus-fs-fi-max=65535;
    focus-fs-fi-min=0;
    focus-mode=continuous-video;
    focus-mode-values=auto,macro,infinity,continuous-picture,continuous-video,manual,fullscan;
    gesture-shot=false;
    gesture-shot-supported=true;
    horizontal-view-angle=62;
    hsvr-size-fps=640x480x120;
    hsvr-size-fps-values=640x480x120;
    hue=middle;
    hue-values=low,middle,high;
    iso-speed=auto;
    iso-speed-values=auto,100,200,400,800,1600;
    jpeg-quality=90;
    jpeg-thumbnail-height=128;
    jpeg-thumbnail-quality=100;
    jpeg-thumbnail-size-values=0x0,160x128,256x192;
    jpeg-thumbnail-width=160;
    m-sr-g=0;
    m-ss=0;
    max-exposure-compensation=3;
    max-num-detected-faces-hw=15;
    max-num-detected-faces-sw=0;
    max-num-focus-areas=1;
    max-num-metering-areas=9;
    max-num-ot=1;
    max-zoom=10;
    metering-areas=(0,0,0,0,0);
    mfb=off;
    mfb-values=off,mfll,ais;
    min-exposure-compensation=-3;
    mnr-e=0;
    mnr-s=true;
    mtk-123-shad-s=true;
    mtk-awb-s=true;
    mtk-cam-mode=0;
    mtk-shad-s=true;
    native-pip=false;
    native-pip-supported=false;
    picture-format=jpeg;
    picture-format-values=jpeg;
    picture-size=2560x1440;
    picture-size-values=320x240,640x480,1024x768,1280x720,1280x768,1280x960,1600x1200,1920x1088,2048x1536,2560x1440,2560x1920,3264x2448;
    pip-fps-zsd-off=30;
    pip-fps-zsd-on=30;
    preferred-preview-size-for-video=1920x1088;
    preview-format=yuv420sp;
    preview-format-values=yuv420sp,yuv420p,yuv420i-yyuvyy-3plane;
    preview-fps-range=5000,60000;
    preview-fps-range-values=(5000,60000);
    preview-frame-rate=30;
    preview-frame-rate-values=10,20,15,24,30,60,120;
    preview-size=1280x720;
    preview-size-values=176x144,320x240,352x288,480x320,480x368,640x480,720x480,800x480,800x600,864x480,960x540,1280x720,1440x1080,1920x1080,1920x1088,1680x1248;
    rotation=0;
    saturation=middle;
    saturation-values=low,middle,high;
    scene-mode=auto;
    scene-mode-values=auto,portrait,landscape,night,night-portrait,theatre,beach,snow,sunset,steadyphoto,fireworks,sports,party,candlelight,hdr;
    sen-mode-s=0;
    sensor-type=252;
    smooth-zoom-supported=true;
    sr-awb-s=true;
    sr-shad-s=true;
    stereo-capture-frame-rate=15;
    stereo-preview-frame-rate=15;
    sv1-s=3;
    sv2-s=3;
    vdr-cc2m-s=true;
    vdr-r=0;
    vdr-r2m-s=true;
    vdr-r4k2k-s=true;
    vertical-view-angle=37;
    vfb-supported=true;
    vfb-supported-values=true;
    video-frame-format=yuv420p;
    video-hdr=off;
    video-hdr-values=off;
    video-size=640x480;
    video-size-values=176x144,320x240,352x288,480x320,640x480,864x480,1280x720,1920x1080,720x480,1280x736,1920x1088;
    video-snapshot-supported=true;
    video-stabilization=false;
    video-stabilization-supported=false;
    vr-buf-count=10;
    vrd-mfr-e=true;
    vrd-mfr-high=30;
    vrd-mfr-low=15;
    vrd-mfr-max=30;
    vrd-mfr-min=15;
    vrd-mfr-s=true;
    whitebalance=auto;
    whitebalance-values=auto,incandescent,fluorescent,warm-fluorescent,daylight,cloudy-daylight,twilight,shade;
    zoom=0;
    zoom-ratios=100,114,132,151,174,200,229,263,303,348,400;
    zoom-supported=true;
    zsd-mode=off;
    zsd-mode-values=off,on


[1]: /post/recenzja-smartfon-neffos-c5-od-tp-link/
[2]: /post/android-repozytorium-aplikacji-opensource-f-droid/
[3]: http://opencamera.org.uk/
[4]: https://play.google.com/store/apps/details?id=net.sourceforge.opencamera
[5]: /post/android-brak-mozliwosci-zapisu-danych-na-karcie-sd-neffos-c5/
[6]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[7]: https://en.wikipedia.org/wiki/Infinity_focus
