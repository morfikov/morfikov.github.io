---
author: Morfik
categories:
- Android
date:    2017-01-27 18:52:06 +0100
lastmod: 2017-01-27 18:52:06 +0100
published: true
status: publish
tags:
- smartfon
- gps
- prywatność
title: Jak zlokalizować skradziony/zagubiony smartfon z Androidem
---

Smartfony towarzyszą nam w codziennym życiu praktycznie cały czas. Dlatego też zaczynamy
przechowywać w tych urządzeniach coraz to więcej informacji osobistych, które są w stanie dość
dokładnie opisać nasze życie prywatne. Co jednak w przypadku, gdy taki telefon zgubimy lub też
zostanie nam on skradziony przez kogoś? Gdy chodzi o urządzenia z Androidem, to Google oferuje
usługę, która jest w stanie połączyć się z naszym smartfonem i przy odrobinie szczęścia ujawnić
nam jego położenie geograficzne lub też pozwolić nam na zdalne zablokowanie systemu w telefonie.
Chodzi oczywiście o usługę "Znajdź telefon/smartfon" (find my phone), na którą rzucimy sobie okiem w
tym artykule.

<!--more-->
## Jak działa usługa "Znajdź telefon/smartfon"

Generalnie rzecz ujmując, usługa "Znajdź telefon/smartfon" jest dostępna w praktycznie każdym
Androidzie i jest automatycznie włączona. Nie musimy zatem nic dodatkowo instalować czy
konfigurować, no chyba, że chcemy tę usługę wyłącz w obawie o naszą prywatność ale do tej kwestii
przejdziemy nieco później. Na razie skupmy się na samej usłudze.

By usługa "Znajdź telefon/smartfon" była w stanie działać prawidłowo, potrzebne jest powiązanie
urządzenia z Androidem, w tym przypadku smartfon, z istniejącym kontem Google. Innymi słowy, trzeba
się zalogować na jakieś konto Google na tym urządzeniu. Innym ważnym punktem, który musi być
spełniony, jest dostęp telefonu do internetu. Nie ma przy tym znaczenia, czy ten dostęp jest po
WiFi czy przez operatora GSM (dane pakietowe). Bez dostępu do internetu, Android nie połączy się z
serwerami Google i usługa jest w zasadzie bezużyteczna.

Nic więcej nam nie jest potrzebne, no chyba, że chcemy uzyskać położenie geograficzne i zlokalizować
smartfon. W takim przypadku telefon musi mieć włączoną nawigację satelitarną (GPS), a ten moduł nie
zawsze jest aktywny i dlatego też nie od razu uda nam się taki zgubiony/skradziony telefon odnaleźć
na mapie.

Idąc dalej, jako, że ta usługa działa w oparciu o przypisanie konta Google do urządzenia z
Androidem, to zresetowanie ustawień do fabrycznych w taki sposób by [obejść blokadę Factory Reset
Protection Lock (FRP Lock)][1], skutecznie uniemożliwia nam przeprowadzenie jakichkolwiek działań
ratunkowych. Warto o tym fakcie pamiętać w przypadku, gdy many z jakiegoś powodu odblokowany
bootloader lub też posiadamy niezbyt przyzwoicie zabezpieczony smartfon, który umożliwia
przepisanie partycji `frp` w celu obejścia ww. blokady. Jeśli któraś z tych powyższych sytuacji ma
miejsce, to biorąc pod uwagę kwestię prywatności, dobrze jest wyłączyć usługę "Znajdź
telefon/smartfon".

## Jak włączyć/wyłączyć usługę "Znajdź telefon/smartfon"

Usługę "Znajdź telefon/smartfon" można wyłączyć z poziomu ustawień systemu smartfona. W zasadzie
zarówno w wersji 5.1 (Lollipop) jak i 6.0 (Marshmallow) stosowna konfiguracja znajduje się w
Ustawienia => Zabezpieczenia => Administratorzy
urządzenia:

![](/img/2017/01/001.lokalizacja-smartfon-telefon-android-kradziez-menadzer-urzadzen.png#huge)

Jak widać na powyższej fotce, w tym przypadku mamy kilka aplikacji, które są w stanie w jakimś
stopniu zarządzać naszym telefonem. Jeśli któreś z nich nie powinno tego robić, to naturalnie ze
względów bezpieczeństwa lepiej jest je odhaczyć. Domyślnie jest zaptakowana tylko pierwsza pozycja,
tj. Menadżer urządzeń Android (nie mylić z aplikacją, która ma dokładnie taką samą nazwę i jest do
pobrania ze sklepu Google Play).

Każda z tych pozycji po kliknięciu wyświetli nam informacje na temat operacji, które dana aplikacja
będzie w stanie przeprowadzić. Poniżej jest rozpisany Menadżer urządzeń
Android:

![](/img/2017/01/002.lokalizacja-smartfon-telefon-android-kradziez-administrator-urzadzenia.png#medium)

No i widzimy, że Menadżer urządzeń Android umożliwia nam na zdalne przeprowadzenie procesu
resetowania ustawień smartfona do fabrycznych, tj. Factory Reset. Oferuje on także zablokowanie
ekranu i zmianę hasła, tak by niepowołane osoby nie były w stanie korzystać z tego telefonu.

## Poszukiwania zgubionego/skradzionego smartfona

Załóżmy w tym momencie, że nie znamy położenia naszego telefonu oraz, że mamy pewne przypuszczenia,
że ktoś nam to urządzenie podprowadził bez naszej świadomej zgody. Co w takiej sytuacji możemy
zrobić? Musimy jak najszybciej uzyskać dostęp do zaufanego komputera i zalogować się na konto
Google, które jest powiązane z uprowadzonym smartfonem. W zasadzie dla przyśpieszenia procesu, można
w wyszukiwarce wpisać frazę "gdzie jest mój telefon" i zostaniemy przez Google odesłani pod właściwy
adres usługi:

![](/img/2017/01/003.lokalizacja-smartfon-telefon-android-kradziez-znajdz-google.png#huge)

Po wejściu w usługę "Znajdź telefon/smartfon", pojawią nam się urządzenia, które są powiązane z tym
kontem Google, i na których była notowana ostatnio jakaś aktywność:

![](/img/2017/01/004.lokalizacja-smartfon-telefon-android-kradziez-aktywnosc-google.png#big)

Załóżmy, że zawieruszył się ten pierwszy TP-LINK'owy Neffos Y5. Trzeba kliknąć w tą pozycję i podać
naturalnie hasło do konta Google. Pamiętajmy, by przy tego typu uwierzytelnianiu sprawdzić czy aby
faktycznie jesteśmy na stronach Google i czy strona jest zabezpieczona przez zaufany certyfikat
(info w zielonej kłódce obok URL).

![](/img/2017/01/005.lokalizacja-smartfon-telefon-android-kradziez-logowanie-google.png#medium)

Po podaniu hasła zostaniemy przeniesieni do listy opcji, które pomogą nam odzyskać telefon:

![](/img/2017/01/006.lokalizacja-smartfon-telefon-android-kradziez-opcje-odzyskiwania.png#big)

Jak widać, trochę tych opcji mamy. Przyjrzyjmy się im nieco bliżej.

### Aktywacja dzwonka

Gdy mamy podejrzenie, że telefon wciąż jest gdzieś w pobliżu, to naturalnie możemy spróbować go
wywołać aktywując w nim dzwonek. W taki sposób nasz telefon zacznie dzwonić z maksymalną siłą,
nawet gdy dźwięki w urządzeniu zostały wyciszone. W sumie to ta opcja jest przydatna nie tylko na
wypadek kradzieży ale też w momencie, gdy nie wiemy gdzie dokładnie położyliśmy telefon.

### Lokalizacja położenia geograficznego

Zakładając, że nasz telefon ma włączony moduł GPS, Google jest w stanie ustalić położenie naszego
smartfona praktycznie natychmiast. Zostanie nam również zwrócona mapka z dokładną lokalizacją
zgubionego urządzenia:

![](/img/2017/01/007.lokalizacja-smartfon-telefon-android-kradziez-gps.png#big)

Problem w tym, że chwilę po zlokalizowaniu urządzenia, na ekranie smartfona zostanie wyświetlony
monit o tym, że urządzenie zostało namierzone.

![](/img/2017/01/008.lokalizacja-smartfon-telefon-android-kradziez-powiadomienie.png#medium)

Nie wiem kto projektował to zabezpieczenie ale dawać informację potencjalnemu złodziejowi, że
"Urządzenie Zlokalizowane" raczej nie ułatwi w odnalezieniu ani smartfona, ani tego kto go sobie
przywłaszczył.

Można naturalnie ten problem wyeliminować ale trzeba wyłączyć notyfikacje dla aplikacji "Usługi
Google Play". Problem w tym, że notyfikacje zostaną wyłączone nie tylko dla Menadżera urządzeń
Android ale dla wszystkich usług Google. Nie wiem dlaczego tak ten mechanizm został zaprojektowany
ale definitywnie kuleje on pod względem bezpieczeństwa/prywatności i wymaga jak najszybszego
dopracowania.

Tak czy inaczej jeśli chcemy się pozbyć tych notyfikacji, to w różnych wersjach Androida potrzebne
nam opcje siedzą w nieco innych miejscach. W przypadku Androida 6.0 (Marshmallow) można je znaleźć
przechodząc w Ustawienia => Menadżer powiadomień. Tam z kolei na liście aplikacji odszukujemy
"Usługi Google Play" i wyłączamy w nich powiadomienia:

![](/img/2017/01/009.lokalizacja-smartfon-telefon-android-kradziez-wylaczenie-notyfikacji.png#huge)

W przypadku, gdy smartfon ma wyłączony moduł GPS, to naturalnie nie da rady go zdalnie włączyć przez
usługę "Znajdź telefon/smartfon" i trzeba ewentualnie czekać na błąd złodzieja, np. będzie on
próbował skorzystać z mapy czy innej tego typu aplikacji wymagającej do prawidłowego działania
modułu GPS.

![](/img/2017/01/010.lokalizacja-smartfon-telefon-android-kradziez-brak-gps.png#huge)

### Zablokowanie telefonu

Czasami przestępcy są nieco bardziej zaawansowani pod względem intelektualnym i mogą być świadomi
faktu, że właściciel telefonu może próbować ich namierzyć przez mechanizmy bezpieczeństwa wbudowane
w to urządzenie. Dlatego też mogą oni świadomie unikać włączenia modułu GPS, a my przez to w ogóle
możemy nie uzyskać lokalizacji smartfona. Jeśli zaś moduł GPS mieliśmy aktywny, to ci nieznani
jeszcze sprawcy mogą podjąć kroki, by ten śledzący dodatek dezaktywować. W takim przypadku jeśli nie
mamy ustawionej blokady ekranu w telefonie, to najlepiej jest zdalnie zablokować system telefonu.

![](/img/2017/01/011.lokalizacja-smartfon-telefon-android-kradziez-zablokuj-ekran.png#huge)

W zasadzie zablokowaniu ulegnie sam ekran. My zaś możemy dodatkowo dostosować informację, która na
tym ekranie zostanie wyświetlona. Nie mamy jednak możliwości zmiany kodu PIN jeśli blokada ekranu
jest już aktywna. Zatem jeśli złodziej zna nasz PIN, to mamy problem.

W przypadku niekontrolowanej utraty telefonu powinniśmy w zasadzie jak najszybciej założyć blokadę
ekranu. Bez niej, złodziej będzie w stanie zresetować ustawienia urządzenia do fabrycznych z poziomu
działającego systemu. Taki zabieg nie dość, że praktycznie nam uniemożliwi zlokalizowanie telefonu,
bo usługa Google przestanie zwyczajnie działać, to jeszcze nie zostanie nałożona na smartfon blokada
Factory Reset Protection Lock.

Oczywiście nie musimy z góry zakładać, że telefon wpadł w ręce złodzieja. Być może też jakiś
przyzwoity człowiek znalazł nasz smartfon i nie wie on za bardzo co w takiej sytuacji ma zrobić.
Jakby nie patrzeć na urządzeniu nie ma żadnej kartki z informacją czyj jest to smartfon. Kartki może
i nie ma ale możemy krótką informację na ekranie wyświetlić i jeszcze podać numer telefonu, z którym
znalazca zostanie połączony po kliknięciu ikonki słuchawki:

![](/img/2017/01/012.lokalizacja-smartfon-telefon-android-kradziez-blokada-aktywana.jpg#huge)

### Wylogowanie z telefonu

Kolejną opcją, która nieco podnosi poziom naszej prywatności i bezpieczeństwa konta Google, to
zdalne wylogowanie się z Androida na skradzionym urządzeniu.

![](/img/2017/01/014.lokalizacja-smartfon-telefon-android-kradziez-wylogowanie.png#huge)

Oczywiście w dalszym ciągu system będzie traktował nasze konto jako powiązane z tym konkretnym
urządzeniem i uniemożliwi zalogowanie się na inne konto.

![](/img/2017/01/013.lokalizacja-smartfon-telefon-android-kradziez-wylogowanie-blokada.png#huge)

Niemniej jednak, pozostała funkcjonalność telefonu będzie do dyspozycji złodzieja, w tym również
możliwość dzwonienia czy włączenia GPS. Taka osoba nie będzie mogła korzystać tylko z tej części
systemu, która wymaga zalogowania się na konto.

Warto w tym miejscu jednak zaznaczyć, że ten krok z wylogowaniem ochroni jedynie nasze konto Google.
Wszystkie prywatne dane, takie jak numery telefonów, treści wiadomości SMS czy zdjęcia/filmy z
aparatu już nie zostaną objęte ochroną i złodziej będzie mógł je swobodnie przejrzeć. Niemniej
jednak, pozostawienie telefonu funkcjonalnym sprawi, że będzie można go namierzyć przez operatora
GSM (ewentualnie GPS), o ile złodziej będzie z niego korzystał. Wypadałoby tylko zablokować kartę
SIM (u operatora), tak by nie ponosić opłat z tytułu nadużywania naszej gościnności.

Zdziwiłem się tylko, że mając tak zablokowane konto Google, zresetowanie ustawień telefonu do
fabrycznych nie nakłada blokady Factory Reset Protection Lock. Dla mnie było oczywistym, że ta
blokada powinna się pojawić ale widać ten Góglowski mechanizm blokowania urządzenia nie jest do
końca dopracowany.

### Wykasowanie wszystkich danych

Posiadając kopię zapasową danych zgromadzonych w telefonie, w sumie można od razu pokusić się o
zdalne przeprowadzenie procesu Factory Reset, co przy okazji założy blokadę Factory Reset Protection
Lock (FRP Lock). W ten sposób złodziej nie uzyska dostępu do danych zgromadzonych na flash'u
urządzenia i w zasadzie nie będzie w stanie w ogóle korzystać z telefonu. Warto tutaj zaznaczyć, że
jeśli w smartfonie podczas tego procesu czyszczenia będzie obecna karta SD, to dane zawarte na tym
nośniku również zostaną wykasowane.

![](/img/2017/01/015.lokalizacja-smartfon-telefon-android-kradziez-kasowanie-danych.png#huge)

Problem w tym, że inicjując takie zdalne czyszczenie pozbawiamy się jednocześnie możliwości
zlokalizowania telefonu przez usługi Google. Mamy przynajmniej pewność, że nikt nie uzyska dostępu
do naszych prywatnych informacji. Coś za coś.

## Aplikacja Android Device Manager

Próby lokalizacji smartfona niekoniecznie muszą być przeprowadzane z poziomu standardowego komputera
czy laptopa. Można do tego celu skorzystać z innego smartfona ale trzeba na nim zainstalować
aplikację Android Device Manager. Można również korzystać ze standardowej przeglądarki ale ta opcja
zostawia za dużo śladów.

W przypadku aplikacji Android Device Manager jesteśmy w stanie zlokalizować smartfon w oparciu o
dane GPS, zdalnie zablokować telefon, włączyć dzwonek i przeprowadzić proces Factory Reset, czyli
mniej więcej te same kroki, które były dostępne w serwisie Google.

Po za logowaniu się w aplikacji Android Device Manager, zostanie nam zwrócona lista urządzeń, które
są powiązane z kontem Google, do którego dane wprowadziliśmy w formularzu logowania:

![](/img/2017/01/016.lokalizacja-smartfon-telefon-android-kradziez-aplikacja.png#huge)

Wybieramy tutaj urządzenie, które nam zaginęło i przy pomocy kilku tapnięć w ekran smartfona możemy
w bardzo prosty sposób wywołać każdą z ww. akcji:

![](/img/2017/01/017.lokalizacja-smartfon-telefon-android-kradziez-opcje-aplikacji.png#huge)


[1]: /post/factory-reset-protection-frp-w-smartfonach-z-androidem/
