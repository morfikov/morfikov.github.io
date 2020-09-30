---
author: Morfik
categories:
- Android
date: "2016-11-03T19:33:54Z"
date_gmt: 2016-11-03 18:33:54 +0100
published: true
status: publish
tags:
- wifi
- tp-link
- smartfon
- aplikacje
title: Wifi Roaming Fix i SWIFI, czyli roaming w smartfonie
---

Ostatnimi czasy coraz więcej sieci domowych zaczyna być wyposażanych w sprzęt umożliwiający
połączenie bezprzewodowe. Router WiFi ma już chyba znaczna większość z nas ale nie są to jedyne
urządzenia, które są w stanie świadczyć bezprzewodowe usługi sieciowe. Im większy dystans dzieli
odbiornik od nadajnika lub też im więcej przeszkód stoi na bezpośredniej drodze komunikacji, tym
sygnał ulega większej degradacji. Zwykle w takiej sytuacji dokupujemy drugi router WiFi, ewentualnie
[prosty AP](http://www.tp-link.com.pl/products/list-12.html), [wzmacniacz sygnału
WiFi](http://www.tp-link.com.pl/products/list-10.html) czy też [ekstendery
powerline](http://www.tp-link.com.pl/products/list-18.html) (PLC). Wszystko po to, by jakoś
przyzwoicie pokryć sygnałem całą przestrzeń użytkową naszego domu czy też miejsc, w których spędzamy
wolny czas. Każde takie urządzenie realizuje połączenie WiFi mniej więcej w ten sam sposób, tj.
zestawia punkt dostępu, do którego podłączamy komputer albo smartfona. O ile w przypadku desktopów
czy laptopów przełączanie się między tymi AP w zależności od siły sygnału nie stanowi większego
problemu, o tyle w przypadku smartfonów z Androidem nie jest już tak różowo, bo przełączenie
następuje jedynie przy całkowitej utracie sygnału z AP. Takiej sytuacji można zaradzić ale trzeba
posiłkować się dodatkowymi aplikacjami. W poniższym artykule zostaną opisane dwa takie programiki:
[SWIFI](https://play.google.com/store/apps/details?id=com.seah0rse.swififree) i [Wifi Roaming
Fix](https://play.google.com/store/apps/details?id=com.heleron.wifiroamingfix).

<!--more-->
## Roaming w przypadku różnych i tych samych ESSID

Sieci WiFi zwykliśmy rozróżniać po nazwie. Nasza bezprzewodowa sieć domowa może posiadać kilka AP, z
tym, że na każdym takim punkcie dostępowym trzeba skonfigurować parametry sieci (ESSID i hasło). Ten
proces może być automatyczny albo manualny, to już zależy od posiadanego przez nas urządzenia. Z
reguły jednak dla uproszczenia zwykliśmy stosować te same parametry dla konkretnej sieci bez względu
na to ile ona posiada AP. Dla poprawienia jakości sygnału możemy wybrać inny kanał, tak by sygnał z
różnych punktów dostępowych nie zakłócał się wzajemnie.

To powyższe rozwiązanie może i jest najczęściej spotykanym ale nie jest ono jedynym. Być może z
jakiegoś powodu chcemy rozdzielić segmenty sieci bezprzewodowej i tworzymy dwie lub więcej sieci
WiFi nadając AP różne nazwy (ew. też i hasła).

Każdy z tych dwóch przypadków w przypadku smartfona z Androidem wymaga innego podejścia i dlatego
też we wstępie wspominałem o dwóch aplikacjach. Wifi Roaming Fix znajduje zastosowanie w przypadku
roamingu w tej samej sieci, czyli wszystkie AP mają ten sam ESSID i hasło. Z kolei zaś SWIFI
specjalizuje się w roamingu w różnych sieciach. Trzeba tutaj jednak zaznaczyć, że te aplikacje nie
zawsze muszą działać na naszym modelu smartfona, czy wersji Androida. W przypadku mojego Neffos'a C5
z Androidem 5.1 (Lollipop) oba te programiki zdają się pracować bez zarzutu i realizują roaming w
miarę w porządku.

Problem z tymi aplikacjami jest natomiast taki, że przełączanie między sieciami odbywa się przy
całkowitym rozłączeniu połączenia. W przypadku różnych ESSID, taka sytuacja jest zrozumiała.
Natomiast rozłączanie połączenia w celu podłączenia się do innego AP z tym samym ESSID może być
lekko uciążliwe. Chodzi o to, że przy przemieszczaniu się między pokojami w domu nie zawsze chcemy
zostać rozłączeni. Jeśli akurat nie mamy nawiązanych żadnych sesji, np. nie pobieramy plików, czy
nie gramy w CS'a, to zwykle nawet nie zauważymy problemu. Natomiast jeśli takie sesje są nawiązane,
to zostaną zerwane przy przełączaniu między AP.

Ten problem można poprawić, przynajmniej tak mi się wydaje, ale trzeba do tego celu zaprzęgnąć root
i ukorzenić Androida. Na nim pracuje `wpa_supplicant` , ten sam, który realizuje połączenie WiFi w
linux'ach. Z nieznanym mi jeszcze powodów, ta konfiguracja roamingu, którą mam u siebie w Debianie,
nie działa na Androidzie. Póki co nie udało mi się skonfigurować roamingu z wykorzystaniem tego
Androidowego `wpa_supplicant` w moim telefonie ale jak tylko mi się ten proces uda, to podlinkuję go
tutaj (FIXME).

Póki co, pozostaje nam albo ręczne rozłączanie sieci albo korzystanie z SWIFI lub Wifi Roaming Fix w
zależności od sytuacji. Te narzędzia generalnie automatyzują ręczny proces rozłączania ale lepsze to
niż nic.

## SWIFI i sieci WiFI o różnych ESSID

Pierwszym z przypadków jest sytuacja, w której mamy do czynienia z AP skonfigurowanymi na różne
sieci WiFi. By nasz smartfon był w stanie przełączać się między takimi sieciami w zależności od siły
sygnału, musimy w Androidzie doinstalować narzędzie SWIFI:

![](/img/2016/11/1.swifi-roaming-smartfon-tp-link-instalacja.png#huge)

SWIFI ma również opcję płatną ale w tym artykule została wykorzystana darmowa wersja. Warto tutaj
zaznaczyć, że w darmowej wersji nie ma reklam. Sama aplikacja jest za to bardzo prosta w obsłudze,
bo ma niewiele opcji:

![](/img/2016/11/2.swifi-roaming-smartfon-tp-link-konfiguracja.png#medium)

Konfiguracja roamingu w SWIFI sprowadza się do ustawienia siły sygnału, poniżej której smartfon
rozpocznie skanowanie eteru w poszukiwaniu lepszej alternatywy dla połączenia. Zakres jaki możemy
tutaj ustawić jest w granicach od -55 dBm do -85 dBm. W zasadzie wartość w granicach -60 dBm powinna
być w miarę optymalna.

Jako, że smartfon zacznie szukać innych sieci, gdy siła sygnału spadnie poniżej ustawionego progu,
to trzeba brać pod uwagę większe zużycie energii i krótszy czas pracy na baterii. Dlatego też musimy
umiejętnie dobrać różnicę wartości w oparciu o siłę sygnałów docierających z pozostałych punktów
dostępowych.

Unikajmy sytuacji, gdzie przesiadując w jednym pokoju, sygnał z aktualnego AP będziemy mieć na
poziomie oscylującym na -55 dBm, a zarazem sygnał z drugiego AP nie przekroczy wartości określonej w
różnicy sygnałów. W takim przypadku, smartfon będzie skanował eter bez przełączenia na nową sieć.
Obie wartości, które widać na powyższej fotce, trzeba dobrać w zależności od konkretnej sytuacji i
panujących warunków.

SWIFI ma też opcję, która pozwala na przełączanie się między sieciami 2,4 GHz i 5 GHz. Przydatna
rzecz jeśli nasz smartfon dysponuje dwoma zakresami WiFi. Warto tutaj zaznaczyć, że możemy nadać
priorytet sieci 5 GHz i SWIFi przełączy nas na to pasmo nawet jeśli sygnał z AP na 2,4 GHz będzie
nieco mocniejszy. Nie znam wartości różnicy w sile sygnału, a nie mam zbytnio jak sprawdzić na moim
Neffos C5, bo on obsługuje jedynie pasmo 2,4 GHz. Jeśli ktoś orientuje się jaka jest to wartość, to
niech da mi znać.

Przełączanie między sieciami możemy również aktywować lub dezaktywować przełącznikiem w prawym
górnym rogu.

## Wifi Roaming Fix i sieci WiFi o tych samych ESSID

Ze SWIFI nie damy rady skorzystać w przypadku sieci opartych o wiele AP skonfigurowanych na tę samą
sieć WiFi. Musimy zatem zainstalować Wifi Roaming Fix. W przypadku tej aplikacji ważna jest nie
tylko nazwa sieci (ESSID) i hasło, które muszą pasować do siebie ale również każdy z AP musi być
dostępny na innym kanale:

![](/img/2016/11/3.wifi-roaming-fix-smartfon-tp-link-instalacja.png#huge)

Po odpaleniu aplikacji zostanie wykryta ilość AP, które mają ten sam ESSID. W moim przypadku są to 3
AP. W menu można wybrać pozycję info, która jest w stanie nam zwrócić informacje o sile sygnału
docierającego z każdego punktu dostępowego w okolicy.

![](/img/2016/11/4.wifi-roaming-fix-smartfon-tp-link-sila-sygnalu.png#big)

Są też proste ustawienia, które umożliwiają włączenie aplikacji Wifi Roaming Fix przy starcie
systemu, pokazywanie notyfikacji przy przełączaniu między AP oraz konfigurację poziomu sygnału,
poniżej którego smartfon zdecyduje się przełączyć.

![](/img/2016/11/5.wifi-roaming-fix-smartfon-tp-link-konfiguracja.png#big)

Moje AP są rozlokowane na kanałach 1, 6 i 11, gdzie siła sygnału w tym konkretnym miejscu mojego
domu jest na poziomie -52 dBm, -74 dBm i -33 dBm. Jestem aktualnie podłączony do AP na kanale 11,
jako, że sygnał z tego punktu dostępowego jest najmocniejszy. Jeśli zmieniłbym teraz położenie i
przeszedł w pobliże któregoś z dwóch pozostałych AP, to nastąpi automatyczne rozłączenie sieci i
podłączenie do tego punktu, z którego jest najmocniejszy sygnał.

![](/img/2016/11/6.wifi-roaming-fix-smartfon-tp-link-przelaczanie-sieci.png#medium)
