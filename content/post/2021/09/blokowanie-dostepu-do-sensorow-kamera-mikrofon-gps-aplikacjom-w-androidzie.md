---
author: Morfik
categories:
- Android
date:    2021-09-22 02:04:00 +0200
lastmod: 2021-09-22 02:04:00 +0200
published: true
status: publish
tags:
- smartfon
- bateria
- kamera
- mikrofon
- gps
- prywatność
- aplikacje
- f-droid
title: Blokowanie dostępu do sensorów (kamera/mikrofon/gps) aplikacjom w Androidzie
---

Raczej już nikogo nie dziwią różne moduły elektroniczne implementowane w smartfonach pokroju
żyroskopu, akcelerometru, magnetometru czy też i innych czujników realizujących pewne określone
funkcje. Te wszystkie rzeczy potocznie zwykło nazywać się sensorami i na dobrą sprawę zdążyły się
one zadomowić na dobre w tych naszych telefonach i to do tego stopnia, że nawet sobie nie zdajemy
sprawy, że w ogóle one istnieją i non stop działają w tle. Przekopując ostatnio ustawienia Androida,
natknąłem się w opcjach deweloperskich na dość enigmatycznie brzmiącą opcję `Sensors Off` , która
była ukryta pod `Quick settings developer tiles` . Zaznaczenie tej opcji sprawiło, że do dyspozycji
użytkownika został oddany dodatkowy kafelek (dostępny z poziomu górnej belki), którego zadaniem
jest sterowanie dostępem aplikacji do sensorów smartfona (nie tylko tych wymienionych wyżej ale też
do kamery mikrofonu, czy nawet GPS). Postanowiłem się trochę pobawić tym mechanizmem i zobaczyć czy
można z niego zrobić jakieś użyteczne narzędzie w kontekście ochrony prywatności.

<!--more-->
## Skąd się wziął Sensors Off w Androidzie

Prawdę mówiąc, to nie przypominam sobie, by we wcześniejszych wersjach Androida (<10) tego typu
opcja wyłączenia sensorów była w ogóle dostępna. Szukając trochę informacji na temat tego całego
`Sensors Off` , który widnieje w opcjach deweloperskich mojego smartfona z Andkiem 11, natrafiłem
na artykuł, który nieco mi wyjaśnił po co jest ta opcja oraz, że została wprowadzona stosunkowo
niedawno, bo w [Androidzie 10][1].

Standardowo w smartfonach z Androidem nie mamy możliwości (przynajmniej do tej pory nie było)
sterowania dostępem do sensorów smartfona. Jeśli jednak włączymy sobie `Sensors Off` , to ten stan
rzeczy ulegnie zmianie. Co prawda, nie ma póki co rozdziału sensorów na poszczególne moduły ale
zawsze jest lepsza jedna opcja, która pozwala wyłączyć wszystkie moduły, niż brak jakiejkolwiek
opcji wyłączającej cokolwiek.

Zatem cóż takiego ten cały `Sensors Off` robi? No jak nazwa może sugerować, ta opcja ma na celu
wyłączenie sensorów. Do sensorów zaliczają się: akcelerometr, żyroskop, magnetometr, czujnik
światła, czujnik zbliżeniowy, kamera/aparat (tylny/przedni), mikrofony, głośniki, nawet GPS i
pewnie jeszcze inne sensory, którymi nie włada mój telefon.

Jako, że nie możemy sterować wyłączeniem/włączeniem konkretnego sensora, to niestety trzeba będzie
się liczyć z faktem, że jeśli postanowimy z tej opcji skorzystać, to trzeba będzie wyłączyć
wszystkie sensory naraz, co niekoniecznie może nas ucieszyć. Mnie jednak bardziej interesuje
użyteczność tej opcji w kontekście prywatności, czyli np. w sytuacji, gdy nie korzystam ze
smartfona i wiem, że przez dłuższy czas nie będę tego robił.

Kiedyś sensory smartfona nie były utożsamiane z zagrożeniem dla naszej prywatności ale od pewnego
czasu takie zagrożenie ze sobą te moduły mogą nieść. Oczywiście kamera czy mikrofon zawsze
zagrożenie dla prywatności ze sobą niosły ale, np. magnetometr (który może robić za cyfrowy kompas),
czy akcelerometr (który może zwrócić informacje na temat prędkości z jaką się poruszamy)? Szkoda,
że po ponad dekadzie obecności Androida na rynku wciąż nie ma opcji do kontrolowania poszczególnych
sensorów z osobna ale opcja `Sensors Off` , to jak najbardziej krok w dobrą stronę.

## Co się stanie po przełączeniu opcji Sensors Off

Raczej nic złego po przełączeniu opcji `Sensors Off` nie powinno się stać. Nawet jeśli coś nam
przestanie działać w Androidzie, to należy pamiętać, że sensory zawsze możemy włączyć ponownie. Do
testów sensorów najlepiej zaciągnąć jakąś aplikację, np [phyphox][2], którą naturalnie
można [pobrać z repozytorium F-Droid][3], jako, że jest to appka w pełni OpenSource. W zależności
od tego jakie sensory posiada nasz telefon, przy pomocy phyphox możemy sprawdzić czy działają one
prawidłowo.

|   |   |   |
|---|---|---|
| ![](/img/2021/09/001.block-sensors-android-phyphox-menu.jpg#small) | ![](/img/2021/09/002.block-sensors-android-phyphox-menu.jpg#small) | ![](/img/2021/09/003.block-sensors-android-phyphox-menu.jpg#small) |

No jak widać po samym menu phyphox'a, opcji do testowania sensorów smartfona mamy dość sporo. Na
potrzeby testu wybrałem sobie wskazania tych bardziej rozpoznawanych sensorów, tj. akcelerometru,
żyroskopu i magnetometru:

|   |   |   |
|---|---|---|
| ![](/img/2021/09/004.block-sensors-android-phyphox-accelerometer.jpg#small) | ![](/img/2021/09/005.block-sensors-android-phyphox-gyroscope.jpg#small) |   ![](/img/2021/09/006.block-sensors-android-phyphox-magnetometer.jpg#small) |

Po wykresach możemy stwierdzić, że moduły zdają się działać prawidłowo. Co jednak się stanie, gdy
przełączymy kafelek `Sensors Off` i zablokujemy sensory?

|   |   |
|---|---|
| ![](/img/2021/09/007.block-sensors-android-tile.jpg#small) | ![](/img/2021/09/008.block-sensors-android-phyphox-gyroscope-failed.jpg#small) |

Jak widać (a właściwie nie widać), przykładowy sensor żyroskopu przestał zwracać dane, które
phyphox mógłby sobie umieścić na grafie. No i praktycznie wszystkie sensory w ten sposób "zamilkły",
tj. żaden sensor już nie daje znaku życia, co naturalnie może cieszyć nie tylko w kontekście
prywatności ale również jeśli chodzi o żywotność baterii. Jeśli sensory nie są aktywne, to nie
trzeba na nie przeznaczać energii z baterii. Można zatem wydłużyć czas pracy urządzenia między
kolejnymi cyklami procesu ładowania.

## Wyłączenie sensorów, a GPS

Wyżej wspomniałem, że do sensorów zalicza się również GPS, no i tak faktycznie jest. Jeśli
wyłączymy sobie sensory i spróbujemy ustalić położenie czy to korzystając z mapy Google, czy też z
OsmAnd+, to zobaczymy, że nasz telefon nie może złapać fix'a (oczywiście przy założeniu, że
lokalizację uzyskuje jedynie za pośrednictwem GPS, a nie przy wsparciu WiFi, BT, czy stacji
bazowych operatora komórkowego). Zatem opcja `Sensors Off` wydaje się być idealnym uzupełnieniem
trybu samolotowego, który nie blokuje aplikacjom dostępu do lokalizacji.

## Dzwonienie ze smartfona z wyłączonymi sensorami

Naturalnie tryb samolotowy i wyłączenie sensorów nie są ze sobą sprzęgnięte, co daje nam możliwość
ustawienia ich w dowolnej konfiguracji. Jeśli jednak zdecydujemy się na niewłączanie trybu
samolotowego ale przy tym wyłączymy sensory, to mogłoby się wydawać, że korzystanie ze smartfona w
kontekście prowadzenia zwykłych komórkowych rozmów głosowych będzie pozbawione sensu, bo przecie
aplikacja dialer'a nie będzie miała dostępu choćby do mikrofonu. Takie stwierdzenie nie jest jednak
do końca prawdą, bo jak [można wyczytać w dokumentacji Androida][1], nie wszystkie aplikacje
uzyskują dostęp do sensorów w ten sam sposób.

Wyłączenie sensorów przy pomocy `Sensors Off` sprawia, że aplikacje tracą do nich dostęp ale
jedynie w sytuacji, gdy próbują go uzyskać via `SensorService`, `CameraService` lub
`AudioPolicyService` . Stock'owa appka dialer'a, czy też ogólnie funkcje telefoniczne w aplikacjach,
z tych usług nie korzystają, przez co dialer będzie mieć dostęp do mikrofonu i bez większego
problemu będzie można dzwonić i odbierać połączenia nawet przy zablokowanych sensorach.

## Nie trzeba już zaklejać kamery/aparatu taśmą

Po wyłączeniu sensorów, upośledzeniu ulegnie również kamera/aparat, przedni i tylny i do tego każda
ich liczba, w którą wyposażony jest nasz smartfon. Przy próbie dostępu do kamery pojawi się jedynie
taki oto komunikat: `Failed to open camera. Camera may be in use by another application` , co
wygląda mniej więcej tak:

![](/img/2021/09/009.block-sensors-android-camera-failed.jpg#small)

Jak widać, podglądu na kamerze nie ma, zatem smartfon nie jest w stanie nic bez naszej zgody
zarejestrować. No i teraz już mamy pewność, że nikt nas kamerką w telefonie nie podejrzy, choć na
wszelki wypadek lepiej jest obiektyw kamery jednak zakleić, na wypadek, gdybyśmy zapomnieli
zablokować te sensory ponownie, albo coś by nam je bez naszej wiedzy odblokowało.

## Podsumowanie

Muszę przyznać, że podoba mi się opcją globalnego wyłączenia wszystkich sensorów w telefonie, choć
naturalnie chciałbym, by każdy sensor można było wyłączyć niezależnie, oraz też opcjonalnie dla
każdej appki z osobna. Fakt, że aplikacja dialer'a działa przy wyłączonych sensorach, sprawia, ze
mamy dostęp do podstawowych funkcji telefonu. Zastanawiają mnie jednak aplikacje, które są w stanie
ominąć wyłączenie sensorów (tak jak wspomniany dialer). Ile jest takich appek i czy są im stawiane
jakieś wymagania, tj. czy pierwsza lepsza aplikacja może uzyskać dostęp do sensorów innymi
sposobami i czy jakieś konsekwencje z tego powodu będą mogły ją za to spotkać? Na pewno opcja
wyłączenia sensorów przyda się osobom, które często korzystają z trybu samolotowego, by mieć
pewność, że żadne dane lokalizacji nie zostaną przez przypadek ujawnione, choć ja bym aż tak bardzo
Androidowi nie zawierzał i na prywatne spotkania wybierał się bez smartfona.


[1]: https://source.android.com/devices/sensors/sensors-off
[2]: https://phyphox.org/
[3]: https://f-droid.org/en/packages/de.rwth_aachen.phyphox/
