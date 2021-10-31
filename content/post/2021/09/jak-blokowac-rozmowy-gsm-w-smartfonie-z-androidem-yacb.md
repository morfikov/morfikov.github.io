---
author: Morfik
categories:
- Android
date:    2021-09-18 15:22:00 +0200
lastmod: 2021-09-18 15:22:00 +0200
published: true
status: publish
tags:
- smartfon
- aplikacje
- f-droid
- spam
- call-center
- prywatność
GHissueID: 330
title: Jak blokować rozmowy GSM w smartfonie z Androidem (YACB)
---

Czy przytrafiła może wam się taka sytuacja, w której to ktoś dzwonił do was z niewiadomego numeru i
przy tym nie był to nikt, kto by zasługiwał na jakąś większą uwagę z waszej strony? Zwykle, gdy mam
do czynienia z kimś takim, to nawet nie odbieram telefonu. Niemniej jednak, najwyraźniej jeden z
moich numerów ostatnio stał się celem ataku jakiegoś spamera z bliżej nieokreślonego call center i
ten czteroliterowy osobnik wydzwania do mnie od kilku dni średnio co 20 minut i to z wielu różnych
numerów. Do tej pory spamerzy w zasadzie omijali mnie szerokim łukiem ale zachowanie tej dość
nieuprzejmej osoby sprawiło, że postanowiłem opracować jakieś rozwiązanie na wypadek, gdyby takich
osobników z gatunku "brakującego ogniwa ewolucji Homo sapiens" było więcej. Po chwili wertowania
netu udało mi się znaleźć appkę, co się zwie [Yet Another Call Blocker][1] (YACB), która w zasadzie
jest tym, czego standardowo brakuje w każdym smartfonie z Androidem na pokładzie, tj. dość
zaawansowanego filtra rozmów przychodzących realizowanych w przestarzałej już technologi GSM, czyli
w zwykłych rozmowach telefonicznych na komórkę. Jako, że ten YACB jest bardzo ciekawym kawałkiem
oprogramowania i do tego w pełni OpenSource (dostępnym w repozytorium F-Droid), to postanowiłem
napisać o nim kilka słów, bo jest to wręcz nieocenione narzędzie w walce z wszelkim syfem pokroju
różnych spamerów z call center.

<!--more-->
## Identyfikacja rozmówcy i spamu w Androidzie

W Androidzie już od dłuższego czasu są [mechanizmy][7], które umożliwiają identyfikację osób, które
chcą z nami nawiązywać połączenie. Gdy taki spamer będzie do nas dzwonił, to zostanie nam
wyświetlona stosowna informacja na ekranie smartfona (tuż pod numerem), np. spam, i wtedy wiemy czy
powinniśmy takie połączenie odebrać. To rozwiązanie ma jednak szereg wad.

Jedną z nich jest to, że takie rozmowy nie są zrzucane z automatu, przynajmniej nie wszystkie,
przez co i tak dochodzą do nas i wymagają od nas jakiegoś działania, no i odrywają przy tym nas od
codziennych czynności czy pracy.

Drugi problem zaś dotyczy naszej prywatności. Jeśli byśmy zdecydowali się korzystać z tego
góglowskiego mechanizmu identyfikacji spamu (domyślnie włączony w każdym Androidzie), to trzeba
liczyć się z faktem przesyłania na serwery google informacji na temat nawiązywanych przez nas
połączeń (prawdopodobnie zarówno tych przychodzących, jak i wychodzących).

|   |   |
|---|---|
| ![](/img/2021/09/005.yacb-call-blocker-android-spam-filer-dialer-caller-id.jpg#small) |  ![](/img/2021/09/006.yacb-call-blocker-android-spam-filer-dialer-caller-id.jpg#small) |

Kolejny problem związany jest z wersją Androida, którą mamy w telefonie. Jeśli mamy Andka 9 (i
niższe), to w tych systemach aplikacja dialer'a jest zintegrowana z mechanizmem identyfikacji spamu.
By na tych systemach móc korzystać z innego filtra połączeń przychodzących niż ten standardowy,
trzeba wymienić całą appkę dialer'a, co nie zawsze jest pożądane czy nawet możliwe.

Dobre wieści są takie, że od Androida 10, [filtr spamu został odseparowany od aplikacji
dialer'a][8], przez co została nam dana możliwość określenia zewnętrznego mechanizmu, który z tym
spamem będzie walczył, no i też nie będzie przesyłał tak wrażliwych danych jak informacje o
nawiązywanych połączeniach do google.

## Zubożona czarna lista numerów

Każdy Android ma też zaszyty w sobie mechanizm czarnej listy połączeń, które system po cichu zrzuci
ilekroć ktoś z numeru uwzględnionego na tej liście będzie próbował się z nami skontaktować. Odnoszę
jednak wrażenie, że funkcjonalność tej listy uszczupla się z każdym kolejnym wydaniem Androida. Dla
przykładu, w Andku 7 można było korzystać z maski, przez co dodanie, np. `+48*` , umożliwiało
zablokowanie danego kraju bez większego problemu. W Androidzie 11 już takiej opcji maski nie widzę
i by zablokować takiego spamera, który używa setek czy tysięcy numerów, to już naprawdę jest nie
lada sztuka.

|   |   |
|---|---|
| ![](/img/2021/09/001.yacb-call-blocker-android-spam-filer-stock.jpg#small) |  ![](/img/2021/09/002.yacb-call-blocker-android-spam-filer-stock-phone-number-add.jpg#small) |

Nie mam pojęcia, czy można tutaj korzystać z maski, bo we wcześniejszych wersjach Androida było to
wyraźnie określone via `prefix` . Tutaj jak widać jest jedynie opcja wpisania numeru telefonu.

Wyżej można też zauważyć, że widnieje tam informacja o numerach typu `unknown` . Dotyczy ona (chyba)
połączeń z numerów, które nie zostały dodane przez nas na listę naszych kontaktów w telefonie.
Zaznaczenie tej opcji sprawi, że nikt spoza osób uwzględnionych na liście kontaktów nie będzie w
stanie się z nami połączyć. No i w zasadzie ta opcja wydaje się być jedyną, by jakoś sobie z
takimi spamerami poradzić ale jednocześnie pozbawia nas możliwości odbierania połączeń od osób,
które spamerami nie są. Gdy prowadzimy firmę, to raczej nie odważymy się tej opcji przełączyć, bo
skutkowałoby to odcięciem się od nowych klientów i w nieodległej przyszłości ubiciem naszego
biznesu.

## Czym jest YACB i jak go włączyć

Cała masa wątpliwości i pytań nasuwa się w stosunku do tych wbudowanych w Androida mechanizmów
mających nas chronić przed spamem. Dodatkowo, słaba konfiguracja filtrów sprawiła, że trzeba było
się rozejrzeć za czymś bardziej zaawansowanym i tak właśnie w łapki wpadł mi Yet Another Call
Blocker, który można naturalnie [pobrać z repozytorium F-Droid][2].

### Jak ustawić YACB jako domyślny filtr spamu

Trzeba tutaj wyraźnie zaznaczyć, że YACB nie jest aplikacją dialer'a, a jedynie filtrem, który
dialer może wykorzystać, by filtrować połączenia przychodzące na nasz telefon. Idealnie zatem
nadaje się na smartfony, które wyposażone są w co najmniej Androida 10. Po zainstalowaniu YACB,
musimy go ustawić jako domyślną appkę dla identyfikowania rozmów przychodzących. By to uczynić
wchodzimy w ustawienia systemu i wpisujemy `Caller ID & Spam` , po czym wskazujemy YACB:

|   |   |
|---|---|
| ![](/img/2021/09/003.yacb-call-blocker-android-spam-filer-caller-id-settings.jpg#small) |  ![](/img/2021/09/004.yacb-call-blocker-android-spam-filer-caller-id-switch.jpg#small) |

Mając ustawiony Yet Another Call Blocker jako domyślną appkę filtrującą ze spamu połączenia
przychodzące, otwieramy ustawienia YACB.

## Baza danych numerów telefonów dla YACB

Pierwsza rzecz jaką musimy sobie skonfigurować, to baza danych, z której YACB będzie pozyskiwał
informacje na temat numerów chcących nawiązać z nami połączenie.

Warto tutaj zaznaczyć, że bez większego problemu z tej bazy danych można korzystać w formie offline,
zatem połączenie z internetem nie jest w ogóle wymagane, by telefoniczny filtr połączeń
przychodzących spełniał swoje zadanie. Trzeba jednak liczyć się z faktem, że z czasem wpisy w tej
bazie danych mogą się dość mocno zdezaktualizować. Tak czy inaczej, filtrowanie numerów w oparciu o
tę bazę danych powinniśmy sobie włączyć:

![yacb-call-blocker-android-spam-filer-settings-block-by-rating](/img/2021/09/007.yacb-call-blocker-android-spam-filer-settings-block-by-rating.jpg#small)

### Auto update bazy

Baza danych numerów dostępna jest w [osobnym repozytorium][3] i pozyskiwana jest z [serwisu
odebractelefon.pl][4]. Jest też ona aktualizowana co 24 godziny, zatem upewnijmy się, że mamy
zaznaczony auto update tej bazy. Rozmiar bazy numerów nie jest mały, tj. zajmuje około 100 MiB ale
na szczęście aktualizacja tej bazy jest przyrostowa, a nie pobierana w całości ilekroć tylko update
zostanie wypuszczony.

Aktualny status bazy danych zawsze można sprawdzić w ustawieniach YACB:

![yacb-call-blocker-android-spam-filer-database-auto-update](/img/2021/09/008.yacb-call-blocker-android-spam-filer-database-auto-update.jpg#small)

### Optymalizacja bazy danych numerów

Domyślnie w bazie danych są przechowywane wszystkie możliwe numery z całego świata. Jest jednak
mało prawdopodobne, że ktoś z zagranicy będzie próbował się z nami skontaktować. Dlatego też możemy
wstępnie odfiltrować wszystkie zbędne numery przez określenie prefiksu naszego kraju, tj. `+48` :

![yacb-call-blocker-android-spam-filer-database-country-prefix](/img/2021/09/009.yacb-call-blocker-android-spam-filer-database-country-prefix.jpg#small)

## Lista kontaktów

W ustawieniach YACB powinniśmy także zaznaczyć opcję od kontaktów. Chodzi tutaj o to, że nawet
jeśli nasz kontakt zostanie pomyłkowo (albo i celowo) dodany do bazy numerów, to YACB bez problemu
zezwoli takiemu numerowi na połączenie i go nie zablokuje.

## Blokowanie w oparciu o ranking numerów

Jak na złość, gdy zabrałem się do pisania artykułu, to ten delikwent dzwoniący do mnie co 20 minut
nagle zamilkł i nie ma po nim praktycznie żadnego śladu już od kilku dni. Zatem nie do końca jestem
w stanie zobrazować jak wygląda proces blokowania potencjalnych rozmów z numerów, które zostały
oznaczone przez społeczność jako spam, call center, czy inny telemarketer. Tak czy inaczej, jeśli
włączymy sobie blokowanie połączeń w oparciu o ranking numerów, to wszystkie pozycje mające
negatywne oceny będą z automatu zrzucane. W logu YACB będzie za to widniał stosowny wpis zawierający
wielce użyteczne dla nas informacje:

|   |   |
|---|---|
| ![](/img/2021/09/010.yacb-call-blocker-android-spam-filer-rating-block.jpg#small) | ![](/img/2021/09/011.yacb-call-blocker-android-spam-filer-rating-block.jpg#small) |

### Zgłaszanie spamerów do bazy numerów

Może się czasem zdarzyć tak, że jakiś spamer przedostanie się przez filtr i uda się mu do nas
dodzwonić. W takim przypadku trzeba go będzie naturalnie wpisać na czarną listę bezpośrednio w
aplikacji YACB. Powinniśmy też zgłosić taki numer do serwisu `odebractelefon.pl` , gdzie po
weryfikacji, ten numer zostanie dopisany i zapewne uwzględniony w kolejnej aktualizacji bazy danych,
przez co ten spamer powinien być już blokowany u innych użytkowników appki Yet Another Call Blocker.

![yacb-call-blocker-android-spam-filer-report-number](/img/2021/09/012.yacb-call-blocker-android-spam-filer-report-number.jpg#small)

## Lokalna czarna lista

YACB posiada lokalną czarną listę, na którą to możemy dodawać numery niebędące aktualnie
uwzględnione w bazie danych `odebractelefon.pl` .  Ta blacklista nie wyróżnia się zbytnio niczym na
tle tej standardowej dostępnej w Androidzie, no może za wyjątkiem maski. W YACB mamy możliwość
określenia zarówno `*` , która odpowiada za zero lub więcej cyferek, oraz `#` , który odpowiada za
dokładnie jedną cyfrę, co nieco bardziej ułatwia blokowanie numerków.

|   |   |
|---|---|
| ![](/img/2021/09/013.yacb-call-blocker-android-spam-filer-black-list.jpg#small) | ![](/img/2021/09/014.yacb-call-blocker-android-spam-filer-black-list-add-number.jpg#small) |

Trzeba jednak pamiętać, że połączenia od osób na liście kontaktów nie będą blokowane. Jeśli z
jakichś powodów chcielibyśmy zablokować połączenie z takim kontaktem, to trzeba go będzie usunąć z
listy kontaktów i dopiero wtedy dodać jego numer na czarną listę.

## Zaawansowany tryb blokowania połączeń

YACB posiada zaawansowany tryb blokowania połączeń (advanced call blocking mode). Zgodnie z tym co
można wyczytać w [FAQ][5], ten tryb wykorzystuje [CallScreeningService][6] do blokowania połączeń,
co umożliwia zablokowanie takiego połączenia zanim telefon wyda z siebie jeszcze jakikolwiek dźwięk.
W normalnym trybie, telefon mógł dzwonić przez krótką chwilę ale w tym zaawansowanym trybie już nie
będzie, co bardzo cieszy, bo zrzucanie spamerów jest transparentne dla nas i nie musimy sobie tym
zadaniem wcale głowy zawracać. Dlatego też upewnijmy się, że ten tryb mamy zaznaczony.

## Usługa monitorująca

Kolejną opcją, na którą możemy się natknąć w ustawieniach YACB, to usługa monitorująca, która ma na
celu zapewnienie, że YACB będzie w stanie filtrować połączenia. Ta nazwa jest trochę myląca, bo
YACB nie instaluje (czy korzysta) z dodatkowej usługi, która ma działać w tle. Ta dodatkowa usługa
to jedynie permanentne powiadomienie, które będzie wyświetlane non stop (do wglądu przez belkę
powiadomień).

![yacb-call-blocker-android-spam-filer-monitoring-service](/img/2021/09/021.yacb-call-blocker-android-spam-filer-monitoring-service.jpg#small)

To powiadomienie ma na celu poprawić działanie YACB na niektórych ROM'ach, np. MIUI. Zwykle jednak
tej opcji nie ma potrzeby aktywować. Jeśli się zastanawiamy czy włączyć tę usługę, to nic się nie
stanie, gdy zaznaczymy stosowną opcję w ustawieniach, bo nie ma ona żadnego wpływu na żywotność
baterii.

## Powiadomienia o nieodebranych połączeniach

Ciche zrzucanie spamerów bez powiadamiania nas o takich zdarzeniach cieszy i to niezmiernie.
Niemniej jednak, od czasu do czasu przydałoby się rzucić okiem na te blokowane przez filtr numery,
by upewnić się chociaż czy ten wdrożony przez nas mechanizm w ogóle działa albo też czy nie blokuje
on numerów, których nie powinien.

Stosowna opcja od powiadomień jest dostępna w ustawieniach YACB. Domyślna konfiguracja nie
wyświetli nam jednak żadnych powiadomień i przydałoby się te powiadomienia skonfigurować chociaż w
taki sposób, by były one rejestrowane ale w trybie cichym. Do tego celu można skorzystać z
natywnego mechanizmu Androida umożliwiającego niezależną konfigurację szeregu kanałów powiadomień
YACB. W tym celu wchodzimy w informację o aplikacji YACB i wybieramy `Notifications` :

|   |   |   |
|---|---|---|
| ![](/img/2021/09/015.yacb-call-blocker-android-spam-filer-notifications.jpg#small) | ![](/img/2021/09/016.yacb-call-blocker-android-spam-filer-notifications.jpg#small) | ![](/img/2021/09/017.yacb-call-blocker-android-spam-filer-notifications.jpg#small) |

No jak widać, YACB rejestruje trochę tych kanałów notyfikacji. Ja sobie wszystkie z nich ustawiłem
w zasadzie w ten poniższy sposób, w którym powiadomienie jak najbardziej będzie się pojawiać ale bez
dzwonka, zapalania ekranu czy innych wibracji telefonu:

|   |   |
|---|---|
| ![](/img/2021/09/018.yacb-call-blocker-android-spam-filer-notifications.jpg#small) | ![](/img/2021/09/019.yacb-call-blocker-android-spam-filer-notifications.jpg#small) |

W ten sposób zrzucanie połączeń z call center nie będzie u mnie w ogóle widoczne, no chyba, że
wejdę sobie w powiadomienia.

### Usługa operatora GSM "kto dzwonił"

Jedyny problem z YACB, który może być dla nas dość uciążliwy, dotyczy usługi operatora odnośnie
tego kto próbował się z nami skontaktować. Ta usługa działa na zasadzie takiej, że jak nie
odbierzemy jakiegoś połączenia (albo je zrzucimy/zakończymy bez odbierania, tak jak to robi YACB),
to po chwili przychodzi do nas SMS od operatora z informacją, że dany numer próbował się z nami
skontaktować. Zatem blokujemy numer na dialerze ale co nam po tym, skoro zaraz przychodzi SMS i
jesteśmy o fakcie przyjścia tego SMS powiadomieni?

Na szczęście większość operatorów GSM (albo i wszyscy) w Polsce oferuje możliwość wyłączenia usług
typu "kto dzwonił?". W Virgin Mobile można bez większego problemu tę usługę wyłączyć z poziomu ich
appki:

![yacb-call-blocker-android-spam-filer-disable-sms-notifications](/img/2021/09/020.yacb-call-blocker-android-spam-filer-disable-sms-notifications.jpg#small)

W ten sposób te SMS'y przestaną do nas przychodzić, a jedyny ślad po spamerze zostanie odnotowany
tylko i wyłącznie w cichym powiadomieniu (oraz logu YACB), na które możemy rzucić okiem w wolnej
chwili.

## Podsumowanie

YACB to bardzo przydatne narzędzie w walce z całej maści spamem pokroju call center (czy innymi
telemarketerami), którego w okół nas jest coraz więcej. Ta aplikacja ma szereg zalet w stosunku
do natywnego rozwiązania oferowanego przez Androida w kwestii identyfikacji i blokowania połączeń
przychodzących, zwłaszcza jeśli chodzi o poprawę naszej prywatności. Jedyną wadą, jaką YACB posiada,
to fakt, że nie jest on domyślnie preinstalowany w smartfonach, przez co wymaga manualnej
konfiguracji, co z kolei może zniechęcić sporą grupę użytkowników. Nie ma go też w sklepie Google
Play Store i jedyną opcją, by go zainstalować jest repozytorium F-Droid. Niemniej jednak, YACB nie
wymaga uprawnień root i można z niego korzystać nawet na stock'owym ROM'ie producenta telefonu,
choć przydałoby się posiadać Androida w wersji 10 lub nowszej.


[1]: https://gitlab.com/xynngh/YetAnotherCallBlocker
[2]: https://f-droid.org/en/packages/dummydomain.yetanothercallblocker/
[3]: https://gitlab.com/xynngh/YetAnotherCallBlocker_data
[4]: https://odebractelefon.pl/
[5]: https://gitlab.com/xynngh/YetAnotherCallBlocker/-/blob/master/FAQ.md#whats-that-advanced-call-blocking-mode
[6]: https://developer.android.com/reference/android/telecom/CallScreeningService
[7]: https://support.google.com/phoneapp/answer/3459196?hl=en
[8]: https://developer.android.com/about/versions/10/features#call-screening
