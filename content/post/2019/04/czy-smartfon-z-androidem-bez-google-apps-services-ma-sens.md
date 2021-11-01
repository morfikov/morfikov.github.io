---
author: Morfik
categories:
- Android
date:    2019-04-16 21:00:15 +0200
lastmod: 2019-04-16 21:00:15 +0200
published: true
status: publish
tags:
- smartfon
- aosp
- microg
- safetynet
- push
- youtube
- gps
- f-droid
- magisk
- xposed
GHissueID: 288
title: Czy smartfon z Androidem bez Google Apps/Services ma sens
---

Jakiś czas temu [natknąłem się na artykuł][1] chwalący Google Play Services i sugerujący zarazem,
że nasz smartfon bez tych usług (i appek zależnych od nich) na niewiele się zda człowiekowi. Nie
jest to jednak do końca prawdą i postanowiłem pokazać na żywym przykładzie jak wygląda operowanie na
telefonie z Androidem pozbawionym jakichkolwiek usług czy aplikacji własnościowych od Google. W
rolach głównych wystąpi mój smartfon LG G4C, który jest już dość leciwy ale można na niego wgrać
LineageOS (lub też inny ROM na bazie AOSP). Po wgraniu ROM'u, w telefonie znajduje się jedynie
garstka podstawowych aplikacji (przeglądarka, galeria, itp), które po pierwsze są opensource, a po
drugie można je bez problemu wyłączyć jeśli nie zamierzamy z nich korzystać. Z telefonu można
dzwonić, przeglądać net (WiFi/LTE), robić zdjęcia i używać tego urządzenia do różnego rodzaju
multimediów. W zasadzie czego oczekiwać więcej od telefonu? Niektórzy jednak chcieli by mieć
możliwość używania, np. nawigacji. No i tu już zaczynają się schody, bo na takim w pełni
otwartoźródłowym Androidzie, GPS nie zadziała OOTB i potrzebna nam jest jakaś alternatywa w postaci
pośrednika między aplikacjami a GPS. Standardowo w Andkach tym zadaniem zajmują się właśnie te
usługi Google. Jak więc zatem zmusić GPS do poprawnej pracy nie chcąc przy tym wgrywać sobie tego
rozbudowanego w uprawnieniach szpiega od Google? Problemów naturalnie może być więcej, a to czy
doświadczymy któregokolwiek z nich zależy głównie od odpowiedniej konfiguracji systemu. Niniejszy
artykuł postara się zebrać wszystkie te niezbędne informacje mające na celu zaimplementowanie w
naszym smartfonie otwartoźródłowej alternatywy dla Google Play Services w postaci microG.

<!--more-->
## Problemy związane z brakiem Google Play Services

W zasadzie każdy Android jest pozbawiony tego całego wynalazku zwanego Google Play Services, jako,
że jest to produkt własnościowy od Google, a licencja Androida jednoznacznie takie elementy
wyklucza. Google Play Services jest kompletnie niezależnym elementem, który można (ale
niekoniecznie trzeba) doinstalować po wgraniu nowego ROM'u. Te stock'owe ROM'y producentów
telefonów (czy też operatorów GSM) zawierają już szereg aplikacji od Google, w tym też ten cały
Google Play Services.

W tym przypadku został wgrany [Resurrection Remix OS][2]. Jest to w zasadzie czysty Android 7.1.2 z
szeregiem modyfikacji (dorzuconych przez zespół tworzący ten ROM) umożliwiających nieco bardziej
zaawansowaną konfigurację systemu. W tym ROM'ie nie znajdziemy żadnych własnościowych aplikacji od
Google. Można naturalnie wgrać osobno [pakiet GAPPS][3] ale my tego robić nie będziemy, przez co
trzeba będzie uporać się z szeregiem problemów.

### Konto Google

Zanim jednak przejdziemy bezpośrednio do samych problemów, wypadałoby by powiedzieć kilka słów o
koncie Google. Przez cały ten czas byliśmy przyzwyczajani, że w Androidzie trzeba mieć dodane i
skonfigurowane jakieś konto Google, tak by np. można było instalować aplikacje ze sklepu Google
Play. Trzeba w tym miejscu wyraźnie zaznaczyć, że te stock'owe Androidy różnych producentów
telefonów mają opcję umożliwiającą pominięcie procesu dodawania konta, co otwiera drogę do
używania telefonu bez potrzeby logowania się na to konto. Oczywiście sporo rzeczy nam albo nie
będzie działać (np. ten wspomniany wyżej sklep z aplikacjami), albo też funkcjonalność danej
aplikacji zostanie dość mocno ograniczona. Niemniej jednak, ze smartfona jak najbardziej się da
korzystać nie mając w nim skonfigurowanego konta Google.

Konto Google jest powiązane bezpośrednio z Google Play Services. Gdy nie zamierzamy korzystać z
Google Play Services (czy ogólnie aplikacji Google), to owe konto nie jest nam w zasadzie do
niczego potrzebne i bez problemu możemy się bez niego obejść, co ukróci nieco zbieranie danych na
nasz temat przez tego giganta. Niemniej jednak, jeśli zamierzamy korzystać z pojedynczych aplikacji
od Google, to takie konto musimy dodać do telefonu, z tym, że aplikacje wykorzystujące konto
Google nie zadziałają nam bez Google Play Services, przynajmniej nie bez dodatkowej konfiguracji.

### Brak GPS

Nie mając na pokładzie Google Play Services nie będzie nam na pewno działał GPS. Nie mając GPS,
wszystkie aplikacje próbujące ustalić naszą pozycję w oparciu o niego nie będą w stanie tego
uczynić. Jakby nie patrzeć, to spory plus do prywatności. Niemnie jednak, jeśli nie potrafimy
operować na mapach bez kropki wskazującej nasze aktualne położenie i kierunek ruchu, to możemy mieć
spory problem z odnalezieniem się w terenie i lepiej by ten GPS nam działał poprawnie. Warto tu
podkreślić, że mapy jako takie działać będą.

### Problemy z działaniem aplikacji

Szereg aplikacji może mieć nieco upośledzoną funkcjonalność, np. wspomniane wyżej mapy, lub w ogóle
te appki odmówią nam współpracy przy uruchamianiu. Niektóre aplikacje są przygotowane na scenariusz
braku Google Play Services i są nas w stanie poinformować komunikatami na ekranie co powinniśmy
zrobić aby aplikacja była w stanie poprawnie działać. Poniżej przykład aplikacji Signal:

|     |    |
| --- | ---|
| ![](/img/2019/04/001-signal-google-play-services-missing.png#small) | ![](/img/2019/04/002-signal-google-play-services-missing.png#small) |

### Brak sklepu Google Play

Skoro nie mamy zainstalowanych w systemie Google Play Services, to o sklepie Google Play również
możemy zapomnieć. Nawet jeśli byśmy go zainstalowali ręcznie (pobierając skądś plik `.apk` ), to
nie ruszy on nam bez usług Google. Jak zatem instalować aplikacje na Androida bez posiadania sklepu
Google Play?

Warto tutaj zaznaczyć, że przy odpowiedniej konfiguracji microG można zaimplementować sobie
funkcjonalność sklepu Google Play. Trzeba tylko skorzystać z [BlankStore][4] lub też Yalp Store.
Obie te appki są otwartoźródłowymi alternatywami dla aplikacji Google Play. Każda z nich wymaga od
nas posiadania konta Google. Na potrzeby tego artykułu będziemy korzystać z tego drugiego
rozwiązania. Trzeba sobie jednak też zdawać sprawę, że używanie z tego typu alternatywnych klientów
sklepu Google Play może doprowadzić do zablokowania naszego konta Google. Jeśli się tego obawiamy,
to możemy stworzyć osobne konto Google i to je podpiąć pod te aplikacje. Można również skorzystać z
F-Droid.

#### F-Droid

W zasadzie to wszystkie aplikacje opensource (albo też znaczna ich większość) dostępnych na
Androida nie wymaga do poprawnego działania Google Play Services. Te otwartoźródłowe appki bez
problemu można ogarnąć przy pomocy [F-Droid][5] i jego [uprzywilejowanego dodatku][6], który
umożliwia zbiorcze aktualizacje w tle bez potrzeby przeprowadzania działań ze strony użytkownika.
Bez tego dodatku, użytkownik musiałby ręcznie każdą aplikację zaktualizować osobno, a to wymagałoby
trochę czasu i byłoby przy tym trochę upierdliwe.

![f-droid-google-play](/img/2019/04/003-f-droid-google-play.png#small)

#### Yalp Store

Jeśli zaś chodzi o aplikacje, które figurują w sklepie Google Play, to możemy je dociągnąć za
pomocą [Yalp Store][7]. Jest to aplikacja, która pobiera pliki `.apk` z tego sklepu i umożliwia ich
ręczną (choć w pełni zautomatyzowaną) instalację/aktualizację w systemie. Takie plik `.apk` mają
stosowne sygnatury (takie same jak te w Google Play), co zapewnia integralność danych konkretnej
aplikacji. Mamy zatem pewność, że jest to ten sam plik co w sklepie Google Play oraz, że nie został
on w żaden sposób zmieniony przez podmioty trzecie (oczywiście jeśli zweryfikowaliśmy jego podpis i
mu ufamy). Podobnie w późniejszym czasie, gdy zajdzie potrzeba aktualizacji aplikacji -- nie da się
tego zrobić w przypadku gdyby podpisy się różniły, czyli dokładnie tak samo jak w przypadku sklepu
Google Play.

![yalp-store-google-play](/img/2019/04/004-yalp-store-google-play.png#small)

Yalp Store jest dostępny w repozytorium F-Droid i można go bez większego trudu zainstalować na
każdym urządzeniu. Do pobierania plików `.apk` potrzebne jest jednak konto Google. Możemy albo
wykorzystać własne konto, albo też posłużyć się tym oferowanym przez samą aplikację.

#### Instalowanie aplikacji z nieznanych źródeł.

Problemem natomiast może być potrzeba przełączenia opcji umożliwiającej instalowanie aplikacji z
nieznanych źródeł. Ta czynność może negatywnie odbić się na bezpieczeństwie naszego Androida. Po
przestawieniu tej opcji, w systemie będzie mogła być zainstalowana praktycznie dowolna aplikacja,
co może budzić pewne obawy.

F-Droid ma swój uprzywilejowany dodatek, który wymaga praw administratora systemu. Gdy nada mu się
stosowne uprawnienia, to przy instalowaniu aplikacji (i ich aktualizowaniu) z repozytorium F-Droid
nie będzie potrzeby ruszania opcji umożliwiającej instalowanie aplikacji z nieznanych źródeł.

Podobnie sprawa wygląda w przypadku Yalp Store, z tym, że tutaj nie mamy żadnego dodatku. Yalp
Store musi zostać zainstalowany jako aplikacja systemowa. Tylko w takiej sytuacji ta appka będzie
się zachowywać podobnie do F-Droid, co umożliwi bezproblemową instalację i aktualizację pakietów w
tle bez potrzeby zdejmowania tego jakże przydatnego zabezpieczenia.

### Synchronizacja i backup danych

Być może część użytkowników przywykła do funkcjonalności oferowanej przez Google w kwestii backup'u
prywatnych danych, takich jak zdjęcia, filmy, kontakty, historia połączeń, wysłane/odebrane
wiadomości SMS, czy też konfiguracja konkretnych aplikacji. Bez usług Google, synchronizacja i
backup danych za jej sprawą do serwerów Google przestanie działać. Trzeba będzie zatem rozejrzeć
się za alternatywną formą backup'u danych.

Praktycznie każda aplikacja umożliwia eksport swoich ustawień ale też mając w systemie ponad setkę
aplikacji, nie uśmiechałoby się raczej nikomu przeprowadzać proces eksportu ustawień dla każdej z
tych aplikacji oddzielnie, bo jest to zwyczajnie mało praktyczne. Jeśli bawimy się w custom ROM'y
(tak jak w tym przypadku), to mamy wgrany również TWRP recovery. Przez tryb recovery jesteśmy w
stanie zrobić backup danych użytkownika (w zasadzie całego telefonu) i skopiować go sobie na
komputer, a wszystko to bez potrzeby dostępu do internetu i narażania się na wyciek jakby nie
patrzeć dość wrażliwych danych. Wypadałoby jedynie zadbać albo o zaszyfrowanie tworzonej kopi
zapasowej systemu smartfona (TWRP to umożliwia) jeśli ma ona być przechowywana na niezaszyfrowanym
dysku, albo też zaszyfrować dysk w komputerze. Przeprowadzenie samej czynności tworzenia backup'u
telefonu trwa kilka minut, zatem może ona być wykonana praktycznie w dowolnym czasie, choć trzeba
liczyć się z chwilowym wyłączeniem telefonu.

### Znajdź moje urządzenie

Google jest w stanie pomóc nam odszukać nasz telefon w przypadku, gdy nam się on gdzieś zawieruszy
albo nam zwyczajnie ktoś go ukradnie. Wszystko za sprawą usługi ["znajdź moje urządzenie"][8]. Gdy
pozbawimy się Google Play Services, to możemy pożegnać się z opcją odzyskania urządzenia za pomocą
wspomnianego serwisu.

Wypadałoby jednak zadać sobie pytanie: ile razy z tej usługi szukania telefonu korzystaliśmy? Ja na
przestrzeni kilku lat w zasadzie ani razu. No może poza testowaniem jak ta usługa działa. Zwykle
też, gdy zajdzie potrzeba zlokalizować to zaginione urządzenie, to będzie to raczej niemożliwe.
Usługa "znajdź moje urządzenie" wymaga, by to szukane urządzenie było podłączone do internetu.
Jeśli połączenie z siecią zostanie odcięte, to już smartfona nie odnajdziemy. Niekoniecznie ktoś
musi celowo nam wyłączyć WiFi czy wyciągnąć kartę SIM z telefonu, bo możemy to urządzenie zgubić w
miejscu, w którym akurat nie ma zasięgu sieci GSM, np. w lesie. Poza tym, przy odrobinie wiedzy
technicznej (i szczęścia), takie urządzenie można zresetować (z pominięciem wszelkich blokad) i
Google już nie będzie w stanie go nigdy namierzyć, przez co ta usługa będzie bezużyteczna.

Usługa "znajdź moje urządzenie" jest chyba w zasadzie adresowana tylko do tych osób, które nie
pamiętają, w którym miejscu położyły telefon w obrębie swojego mieszkania. W każdym innym przypadku
jest spore prawdopodobieństwo, że się przeliczymy oczekując jakichś wymiernych rezultatów, gdy nam
faktycznie ten telefon rozpłynie się w powietrzu. W zasadzie w zamian za komfort psychiczny
wywodzący się z fałszywego poczucia bezpieczeństwa, które oferuje nam Google, dajemy mu praktycznie
nieograniczony wgląd w nasze życie, bo ta usługa oprócz połączenia internetowego wymaga również
włączonego GPS. Google jest zatem w stanie monitorować nasze położenie praktycznie non stop, a
kwestii czy to robi (lub będzie robił) [bez naszej świadomej zgody][9] lepiej nie poruszać.

### Powiadomienia PUSH

Obecnie sporo aplikacji wykorzystuje powiadomienia PUSH. Co to takiego? Jest to komunikat ze strony
serwera, który ma zwykle na celu wywołać u nas jakąś interakcję, np. sprawdzenie nowej poczty, czy
obejrzenie filmu na YouTube, który został dodany przez jeden z subskrybowanych przez nas kanałów.

Dana aplikacja zamiast łączyć się bezpośrednio z serwerem co określony interwał czasu nasłuchuje
jedynie powiadomień PUSH. Jeśli serwer aplikacji będzie chciał abyśmy, np. sprawdzili pocztę, bo
ktoś nam wysłał wiadomość email, to do aplikacji trafi stosowne powiadomienie dosłownie chwilę po
nadejściu tego mail'a. Dzięki takiemu rozwiązaniu oszczędzamy pakiet danych oraz wydłuża się czas
pracy na baterii, bo aplikacja nie utrzymuje żadnego połączenia z serwerem i nie wybudza przy tym
co chwilę telefonu, by przesłać jakieś dane przez sieć.

Problem w tym, że te wszystkie powiadomienia PUSH [przechodzą dodatkowo przez serwery Google][10].
Ten cały mechanizm może działać tylko dlatego, że aplikacja sklepu Google Play otwiera połączenie z
serwerem Google i przełącza je w tryb nasłuchu. Dlatego też ta aplikacja musi być zainstalowana w
telefonie, by te powiadomienia PUSH działały prawidłowo, co z kolei pociąga zależności w postaci
Google Play Services. Jeśli teraz nie mamy w systemie usług Google, to możemy mieć problem z
odebraniem tych powiadomień PUSH, a bez nich aplikacje nie będą działać tak jak powinny.

## MicroG jako alternatywa dla Google Play Services

Jakby nie patrzeć, w tym krótkim podsumowaniu wyżej pojawiła się cała masa problemów, które trzeba
by zaadresować jeśli w ogóle mamy zamiar używać telefonu do czegoś innego niż tylko wykonywanie
połączeń głosowych, odbieranie SMS'ów i używanie przeglądarki internetowej. No może to nieco zbyt
duże uproszczenie ale widać, że ewidentnie coś jest na rzeczy i trzeba się z tymi problemami jakoś
uporać. Z pomocą przychodzi nam [projekt microG][11].

MicroG to otwartoźródłowa alternatywa dla Google Play Services, która jest w stanie rozwiązać
problemy z GPS i powiadomieniami PUSH. MicroG jest także w stanie umożliwić poprawne działanie
indywidualnym aplikacjom od Google, które buntują się, gdy w Androidzie nie ma zainstalowanego
Google Play Services, np. aplikacja YouTube. Oczywiście jeśli chcemy korzystać z aplikacji Google,
to zwykle potrzebne będzie dodanie konta Google.

## Integracja microG z AOSP

Wykorzystując microG możemy mieć zatem w pełni funkcjonalny smartfon, z tym, że pierw musimy sobie
zintegrować microG z jego systemem. Na stronie microG w dziale [Download][12] mamy zestaw plików,
które musimy pobrać i zainstalować w telefonie. Jest także dostępne [oficjalne repozytorium microG
dla F-Droid][13], dzięki czemu nie będzie potrzeby ręcznej aktualizacji microG w późniejszym czasie.
Włączamy także opcje deweloperskie w ustawieniach Androida, by mieć dostęp do ADB .

W przypadku Androida w wersji 7 i wyższych trzeba nieco inaczej podejść do instalacji microG w
systemie. Mianowicie, trzeba ręcznie pobrać pliki microG i umieścić je przy pomocy `adb push`w
katalogu `/system/priv-app` :

    $ adb push \
       com.google.android.gms-13280012.apk \
       com.android.vending-16.apk \
       com.google.android.gsf-8.apk \
       /sdcard

    $ adb shell
    c90:/ $ su
    c90:/ # mount -o remount,rw /system
    c90:/ # cp /sdcard/com.*.apk /system/priv-app/
    c90:/ # chmod 644 /system/priv-app/com.*.apk

Po wykonaniu tych powyższych czynności restartujemy urządzenie. Na liście aplikacji powinna się nam
pojawić nowa pozycja `microG settings` . Odpalamy ją i przyznajemy uprawnienia, o które nas poprosi:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/005-microg-missing-permissions.png#small) | ![](/img/2019/04/006-microg-missing-permissions.png#small) | ![](/img/2019/04/007-microg-missing-permissions.png#small) |![](/img/2019/04/008-microg-missing-permissions.png#small) |

Następnie przeprowadzamy `Self-Check` :

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/009-microg-self-check-fail.png#small) | ![](/img/2019/04/010-microg-self-check-fail.png#small) | ![](/img/2019/04/011-microg-self-check-fail.png#small) | ![](/img/2019/04/012-microg-self-check-fail.png#small) |

No jak widać test nie wypadł po naszej myśli, bo powinny się zapalić wszystkie kontrolki, a w tym
przypadku paru rzeczy brakuje. Te wszystkie zaistniałe problemy trzeba poprawić. Jeśli któryś z
nich nie zostanie wyeliminowany, to nasz telefon nie będzie działał jak należy.

### Signature spoofing

Jak było widać wyżej, system w moim smartfonie nie posiada wsparcia dla [signature spoofing][14].
Dzięki `signature spoofing`, GmsCore będzie w stanie udawać, że w telefonie jest obecna oficjalna
wersja Google Play Services w momencie, gdy aplikacje będą się odwoływały do API Google. Trzeba
tutaj zaznaczyć, że nie wszystkie ROM'y wspierają `signature spoofing` , a bez tego microG nie
będzie działał poprawnie.

#### Magisk, Xposed i FakeGApps
W tym przypadku mamy do czynienia z Resurrection Remix OS, który najwyraźniej nie wspiera tego
całego `signature spoofing` . Część ROM'ów ma już wszystko na swoim miejscu i poniższe kroki nie są
potrzebne. Niemniej jednak, jeśli odnajdziemy się w takiej sytuacji jak ta, to pozostaje nam w
zasadzie zainstalować [Xposed][15] i dociągnąć stosowny moduł. Najlepiej jest instalować Xposed przy
pomocy [Magisk'a][16]. Przesyłamy zatem `MagiskManager-v7.1.1.apk` oraz `Magisk-v18.1.zip` na
telefon:

    $ adb push MagiskManager-v7.1.1.apk Magisk-v18.1.zip /sdcard

Paczkę `.zip` instalujemy z poziomu TWRP recovery, a plik `.apk` bezpośrednio w systemie jako
zwykłą aplikację.

![twrp-magisk-flash-recovery](/img/2019/04/013-twrp-magisk-flash-recovery.png#small)

Następnie odpalamy aplikację `MagiskManager` i instalujemy Xposed:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/013-microg-magisk-xposed-installation.png#small) | ![](/img/2019/04/014-microg-magisk-xposed-installation.png#small) | ![](/img/2019/04/015-microg-magisk-xposed-installation.png#small) |

I dociągamy również [XposedInstaller_3.1.5-Magisk.apk][17] i wrzucamy na telefon:

    $ adb push XposedInstaller_3.1.5-Magisk.apk /sdcard

Naturalnie instalujemy tę paczkę w standardowy sposób i uruchamiamy smartfon ponownie. Od tego
momentu Xposed powinien nam już działać poprawnie:

![xposed-systemless](/img/2019/04/016-xposed-systemless.png#small)

Po skończonym procesie instalacji Xposed, instalujemy moduł `FakeGApps by thermatk` :

|     |    |
| --- | ---|
| ![](/img/2019/04/017-xposed-systemless-microg-fakegapps.png#small) | ![](/img/2019/04/018-xposed-systemless-microg-fakegapps.png#small) |

Po raz kolejny uruchamiamy urządzenie ponownie i sprawdzamy w logu Xposed czy są jakieś wzmianki na
temat zwracania podrobionych sygnatur przez moduł `FakeGApps`:

![xposed-systemless-microg-fakegapps-signature-spoofing](/img/2019/04/019-xposed-systemless-microg-fakegapps-signature-spoofing.png#small)

Ich obecność świadczy, że moduł wykonuje swoją robotę poprawnie.

Mając Zainstalowany moduł `FakeGApps` powinniśmy mieć już w systemie wsparcie dla `signature
spoofing` . Odpalamy zatem ustawienia microG i sprawdzamy czy dostaliśmy check przy stosownej
pozycji:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/020-microg-signature-spoofing-check.png#small) | ![](/img/2019/04/021-microg-signature-spoofing-check.png#small) | ![](/img/2019/04/022-microg-signature-spoofing-check.png#small) |

No i jak widać, obecnie system mojego smartfona ma zapalone praktycznie wszystkie check'i. Pora
zająć się tymi dwoma ostatnimi, czyli konfiguracją backend'ów lokalizacji.

### Backend'y Unified NLP i GPS

Kolejną rzeczą, którą musimy zrobić by zintegrować microG z systemem telefonu to skonfigurowanie
co najmniej jednego backend'u lokalizacji dla Unified NLP ([Unified Network Location Provider][18]).
Ten cały Unified NLP jest zawarty w paczce z microG, którą pobraliśmy wcześniej i nie trzeba nic
dodatkowo już pobierać, by ten Unified NLP działał poprawnie. Potrzebny nam jest jednak co najmniej
jeden backend. [Tych backend'ów NLP jest kilka][19]. Na nasze potrzeby wystarczą:
`NominatimNlpBackend` oraz `MozillaNlpBackend` -- obie te rzeczy można pobrać przez F-Droid.

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/023-f-droid-nlp-backend-list.png#small) | ![](/img/2019/04/024-f-droid-nlp-backend-mozillanlpbackend.png#small) | ![](/img/2019/04/025-f-droid-nlp-backend-nominatimnlpbackend.png#small) |

Teraz konfigurujemy backend'y w ustawieniach microG:

|     |    |
| --- | ---|
| ![](/img/2019/04/026-microg-nlp-backend-configuration.png#small) | ![](/img/2019/04/027-microg-nlp-backend-configuration.png#small) |

Poniżej są ustawienia dla backend'u `MozillaNlpBackend` :

|     |    |
| --- | ---|
| ![](/img/2019/04/028-f-microg-nlp-backend-mozillanlpbackend-configuration.png#small) | ![](/img/2019/04/029-f-microg-nlp-backend-mozillanlpbackend-configuration.png#small) |

A niżej ustawienia dla backend'u `NominatimNlpBackend`

|     |    |
| --- | ---|
| ![](/img/2019/04/030-f-microg-nlp-backend-nominatimnlpbackend-configuration.png#small) | ![](/img/2019/04/031-f-microg-nlp-backend-nominatimnlpbackend-configuration.png#small) |

W ustawieniach Androida przestawiamy jeszcze tryb lokalizacji na ten o największej dokładności:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/032-android-localization-mode.png#small) | ![](/img/2019/04/033-android-localization-mode.png#small) | ![](/img/2019/04/034-android-localization-mode-microg-fix.png#small) |

Jeśli ostatnie dwie pozycje nie przejdą testu, to trzeba trochę poczekać, aż system ustali nasze
położenie.

#### Weryfikacja poprawnego działania GPS

Wypadałoby teraz zainstalować jakąś aplikację od sprawdzenia koordynatów GPS i zweryfikować czy GPS
działa prawidłowo. Można zainstalować mapy OsmAnd+ i poczekać na pojawienie się kropki.

|     |    |
| --- | ---|
| ![](/img/2019/04/035-microg-gps-test-osmand-maps.png#small) | ![](/img/2019/04/036-microg-gps-test-osmand-maps.png#small) |

No i jak widać, mapy OsmAnd+ nie miały problemów ze skorzystaniem z GPS. W ustawieniach Androida
widzimy także, że i inne aplikacje są w stanie bez problemu żądać lokalizacji GPS od microG, zatem
GPS mamy z głowy.

### Notyfikacje PUSH

Pora się zająć jeszcze notyfikacjami PUSH. W zasadzie to musimy jedynie włączyć dwie opcje w
ustawieniach microG. Pierwszą z nich jest `Google device registration` , która umożliwia
zarejestrowanie naszego urządzenia w usługach Google i nadaniu mu unikalnego identyfikatora. MicroG
usuwa z tego identyfikatora wszystkie zbędne bity za wyjątkiem nazwy konta z danych rejestracyjnych.
Bez tego identyfikatora, powiadomienia PUSH nie będą w stanie działać, wiec jeśli chcemy je mieć,
to musimy zezwolić na proces rejestracji swojego urządzenia. Drugą rzeczą jest zaś włączenie
`Google Cloud Messaging` :

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/037-microg-google-services-configuration.png#small) | ![](/img/2019/04/038-microg-google-device-registration.png#small) | ![](/img/2019/04/039-microg-google-cloud-messaging.png#small) |

Od tego momentu nasz telefon będzie w stanie otrzymać powiadomienia PUSH przekazywane z serwerów
Google.

#### Test powiadomień PUSH

Zainstalujmy zatem w systemie jakąś appkę, która ma wsparcie dla powiadomień PUSH. Firefox jest
jedną z takich aplikacji. Po jej uruchomieniu i skonfigurowaniu synchronizacji historii między
urządzeniami, Firefox powinien zarejestrować się i otrzymać kilka wiadomości:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/040-microg-push-notification-test-firefox.png#small) | ![](/img/2019/04/041-microg-push-notification-test-firefox.png#small) | ![](/img/2019/04/042-microg-push-notification-test-firefox.png#small) |

Widać wyraźnie, że microG jest w stanie nawiązać i utrzymać połączenie z serwerami Google oraz
odebrać od nich notyfikacje PUSH i przekazać je do konkretnych aplikacji. Zatem również i
powiadomienia PUSH mamy z głowy.

### A co z aplikacjami Google

Jeśli ktoś chciałby używać pojedynczych aplikacji Google, np. [YouTube Vanced][20] (taka bardziej
podrasowana wersja przeglądarki YouTube), to wystarczy, że doinstaluje stosowną aplikację i
opcjonalnie stworzy konto Google. 

Poniższy przykład odnosi się do YouTube Vanced ale można go zaaplikować chyba do dowolnej aplikacji
Google, bo póki co wszystkie appki od Google, z których czasem korzystam (YT, Translate i Street
View), bez problemu działają bez Google Play Services w takim ustawieniu microG jak sobie wyżej
skonfigurowaliśmy.

Jeśli chodzi akurat o YouTube Vanced, to jest on dostępny w repozytorium Magisk'a. Inne aplikacje
Google trzeba pobrać za pomocą Yalp Store.

![youtube-vanced-magisk-systemless-install](/img/2019/04/043-youtube-vanced-magisk-systemless-install.png#small)

Po zainstalowaniu appki YouTube Vanced możemy bez problemu przeglądać serwis YouTube, choć
funkcjonalność tej aplikacji zbliżona jest bardziej do tej znanej ze SkyTube albo NewPipe. Jeśli
jednak chcielibyśmy mieć nasze subskrypcje i ogólnie wszystkie te rzeczy, które YouTube oferuje, np.
powiadomienie o strimach, czy możliwość komentowania materiałów video, to potrzebne będzie nam
konto Google:

|     |    |     |
| --- | ---| --- |
| ![](/img/2019/04/044-microg-youtube-vanced-google-account.png#small) | ![](/img/2019/04/045-microg-youtube-vanced-google-account.png#small) | ![](/img/2019/04/046-microg-youtube-vanced-google-account.png#small) |
|![](/img/2019/04/047-microg-youtube-vanced-google-account.png#small) | ![](/img/2019/04/048-microg-youtube-vanced-google-account.png#small) | ![](/img/2019/04/049-microg-youtube-vanced-google-account.png#small) |

Stworzenie konta Google jak widać jest bezproblemowe i w pełni wspierane przez microG. Po
zalogowaniu się w YouTube Vanced mamy w pełni działającą aplikację od YouTube. Zarejestrowała się
ona też w tym całym mechanizmie notyfikacji:

|     |    |
| --- | ---|
| ![](/img/2019/04/050-microg-youtube-vanced-push-nitification.png#small) | ![](/img/2019/04/051-microg-youtube-vanced-push-nitification.png#small) |

Co do samego konta Google, to wypadałoby powiedzieć, że nawet po jego dodaniu cała masa rzeczy
wymagających konta Google nie do końca będzie działać, np. synchronizacji danych:

|     |    |
| --- | ---|
| ![](/img/2019/04/052-google-account-data-sync.png#small) | ![](/img/2019/04/053-google-account-data-sync.png#small) |

Oczywiście w przypadku innych aplikacji, które wspierają synchronizację, np. widoczny wyżej
Firefox, ten proces jest jak najbardziej możliwy, bo to już od samej aplikacji zależy czy oferuje
ona tego typu funkcjonalność:

|     |    |
| --- | ---|
| ![](/img/2019/04/054-firefox-sync-microg.png#small) | ![](/img/2019/04/055-firefox-sync-microg.png#small) |

W przypadku aplikacji od Google do synchronizacji wymagane jest Google Play Services i dlatego ta
synchronizacja danych w naszym przypadku nie będzie możliwa.

### SafetyNet

W zależności od tego jaki ROM posiadamy i czy ma on wsparcie dla `signature spoofing` zależeć
będzie przejście testów SafetyNet. W tym przypadku niestety trzeba było się ratować modułem dla
Xposed. Może i Xposed został zainstalowany w formie systemless, czyli bez wprowadzania zmian na
partycji `/system/` ale ten fakt na nic nie wpływa. Po prostu jeśli zainstalujemy Xposed, to nasz
telefon nigdy tego testu SafetyNet nie przejdzie. W przypadku, gdy korzystamy z w miarę przyzwoicie
wspieranego ROM'u, który ma zaimplementowane `signature spoofing` , to odpada nam potrzeba
instalowania Xposed, a to z kolei otwiera nam drogę do zdania testu SafetyNet.

No niestety nie mam jak skonfigurować na tym telefonie SafetyNet i raczej nie opiszę tego procesu
tutaj. Niemniej jednak, zgodnie z tym co można wyczytać w ustawieniach microG, to musimy jedynie
włączyć opcję Google SafetyNet i dociągnąć paczkę `DroidGuard Helper` , która jest dostępna
w [repozytorium microG][21].

Oczywiście później czeka nas również proces dostosowania systemu pod kątem SafetyNet, tak by ten
test wypadł poprawnie ale to już nie powinno sprawiać problemów, a nawet jeśli, to jest to
zagadnienie na zupełnie inny artykuł.

## Podsumowanie

Jakby nie patrzeć trochę się napracowaliśmy by tego w pełni otwartoźródłowego Androida doprowadzić
do stanu używalności, choć i tak jego lwia część działała bez problemu przed przeprowadzeniem tych
wszystkich wymienionych wyżej czynności. Niemniej jednak, widać na żywym przykładzie, że smartfon z
Androidem jest w stanie normalnie funkcjonować bez Google Play Services, o ile zaimplementuje się w
nim poprawnie microG.

Można było zauważyć też, że uruchamianie pojedynczych aplikacji od Google nie stanowi żadnego
wyzwania i w zasadzie użytkownik może sam zdecydować czy ma ochotę w ogóle z nich korzystać. Nic
nie przychodzi preinstalowane i nie zmusza się użytkownika telefonu do korzystania z aplikacji,
których on zwyczajnie nie chce używać, bo ma alternatywne w pełni otwartoźródłowe rozwiązania,
np. `gmail` vs. `k9mail` .

Trzeba także pamiętać o zachowaniu rozwagi przy korzystaniu z telefonu, bo jeśli wpadnie on w
niepowołane łapki, to może się to skończyć źle i nie chodzi tutaj tylko o przypadek kradzieży
urządzenia ale również o codzienne korzystanie z niego. Część mechanizmów bezpieczeństwa w takim
smartfonie nie działa, przez co jest on mniej odporny na niezbyt przemyślane zachowania swojego
użytkownika.

No i oczywiście synchronizacja danych w takim telefonie odbywać się będzie na poziomie aplikacji.
Jeśli aplikacja nie wspiera synchronizacji danych do chmury, tak by po ponownym jej zainstalowaniu
te dane z serwera pobrać, to trzeba raz na jakiś czas robić backup danych telefonu za pomocą TWRP
recovery. Być może dla niektórych osób chwilowe wyłączenie telefonu na czas tworzenia kopi
zapasowej może być ździebko niepożądane ale też taki backup zawierać będzie w zasadzie cały
snapshot systemu i w razie czego będziemy w stanie w minutę czy dwie odtworzyć sobie system
całkowicie i to bez dostępu do internetu oraz polegania na podmiotach trzecich.

Powyżej przedstawione rozwiązanie nie jest dla wszystkich. Nawet jeśli trud konfiguracji zostanie
przezwyciężony, to w późniejszym czasie korzystania z telefonu trzeba zwracać uwagę na to co się
faktycznie na nim robi, by czasem systemowi (i przy okazji sobie) krzywdy nie zrobić. Ci bardziej
zaawansowani użytkownicy Androida (czy ogólnie linux'a) bez problemu będą w stanie z takiego
telefonu bezpiecznie korzystać, a inni powinni się tego po prostu nauczyć.


[1]: https://android.com.pl/programowanie/188397-po-co-nam-uslugi-google-play/
[2]: https://www.resurrectionremix.com/
[3]: https://opengapps.org/
[4]: https://forum.xda-developers.com/showthread.php?t=1715375
[5]: https://f-droid.org/
[6]: https://f-droid.org/en/packages/org.fdroid.fdroid.privileged/
[7]: https://github.com/yeriomin/YalpStore
[8]: https://www.google.com/android/find
[9]: https://niebezpiecznik.pl/post/google-wie-gdzie-jestes-i-gdzie-byles-i-dzieli-sie-tym-z-policja/
[10]: https://android.com.pl/programowanie/159193-powiadomienia-push/
[11]: https://microg.org/
[12]: https://microg.org/download.html
[13]: https://microg.org/fdroid/repo
[14]: https://github.com/microg/android_packages_apps_GmsCore/wiki/Signature-Spoofing
[15]: https://forum.xda-developers.com/showthread.php?t=3034811
[16]: https://forum.xda-developers.com/apps/magisk/official-magisk-v7-universal-systemless-t3473445
[17]: https://forum.xda-developers.com/xposed/unofficial-systemless-xposed-t3388268
[18]: https://github.com/microg/android_packages_apps_UnifiedNlp
[19]: https://f-droid.org/en/packages/com.google.android.gms/
[20]: https://forum.xda-developers.com/android/apps-games/app-youtube-vanced-edition-t3758757
[21]: https://microg.org/download.html
