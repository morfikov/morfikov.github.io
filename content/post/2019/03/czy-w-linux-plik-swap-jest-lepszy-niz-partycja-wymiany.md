---
author: Morfik
categories:
- Linux
date: "2019-03-22T16:23:42Z"
published: true
status: publish
tags:
- ext4
- system-plików
- swap
title: Czy w linux plik SWAP jest lepszy niż partycja wymiany
---

Ostatnimi czasy, z racji rozwoju technologicznego, mamy do dyspozycji coraz to szybsze komputery,
co przekłada się w znacznym stopniu na prędkość wykonywania operacji przez ich systemy operacyjne.
Obecnie przeciętnej klasy desktop czy laptop jest już wyposażony w 16G czy nawet 32G pamięci
operacyjnej (w niedługim czasie
nawet [smartfony będą posiadać 12G RAM](https://android.com.pl/news/200097-samsung-12-gb-ram-lpddr4x/)).
Spada zatem zapotrzebowanie wykorzystania dysku twardego jako pamięci RAM. W linux używanie
dysku twardego jako rozszerzenie pamięci operacyjnej było i jest w dalszym ciągu realizowane za
sprawą przestrzeni wymiany SWAP. Ta przestrzeń wymiany może być zaimplementowana w postaci osobnej
partycji dysku twardego albo też jako plik umieszczony w obrębie systemu plików, np. ext4. Część
dystrybucji linux'a decyduję się na porzucenie partycji wymiany na rzecz pliku SWAP. Czy taki krok
jest uzasadniony i czy korzystając aktualnie z partycji wymiany powinniśmy zmigrować na plik SWAP?

<!--more-->
## Osobna partycja, a plik w systemie plików

Wydawać by się mogło, że osobna partycja wymiany będzie miała jakiś dodatkowy bonus w stosunku do
pliku SWAP z racji zastosowania w tym drugim przypadku dodatkowej warstwy logicznej w postaci
systemu plików (w tym przypadku ext4). Obecnie jednak oba rozwiązania, tj. SWAP w pliku i SWAP jako
partycja, [mają bardzo zbliżoną wydajność](https://lkml.org/lkml/2006/5/29/3) i przestrzeń wymiany
nie cierpi z powodu umieszczenia jej wewnątrz systemu plików.

Dodatkowo, trzymanie przestrzeni wymiany w pliku sprawia też, że nie potrzebna nam jest dedykowana
partycja. W przypadku, gdy SWAP nam nie będzie już do niczego potrzebny, to plik można zwyczajnie
skasować odzyskując natychmiastowo miejsce, które zajmował. W przypadku fizycznych partycji, ten
zabieg ze zwolnieniem miejsca może być bardziej skomplikowany. Podobnie też i w drugą stronę, gdy
zajdzie potrzeba utworzenia przestrzeni wymiany lub jej powiększenia.

Zatem gdybyśmy rozpatrywali tylko tę jedną kwestię, to plik SWAP byłby faworytem i można by na tym
stwierdzeniu skończyć artykuł. Niemniej jednak, jest jeszcze kilka rzeczy, które wypadałoby
poruszyć, i dopiero po przeanalizowaniu ich wszystkich dokonać oceny, które z tych dwóch rozwiązań
wdrożenia przestrzeni wymiany na linux jest tym lepszym.

## Fragmentacja pliku SWAP

Może i utworzenie przestrzeni wymiany SWAP w postaci regularnego pliku nie wpływa na jej wydajność
ale co z fragmentacją danych? Przecież w dalszym ciągu plik w systemie plików może ulec
fragmentacji. Większe pliki zwykle oznaczają fragmentację danych, a do takich dużych plików się
będzie zaliczał plik SWAP. Zatem można założyć, że im bardziej będzie pofragmentowana przestrzeń
wymiany, tym bardziej niekorzystnie może się to odbić na wydajności systemu.

Spójrzmy na przykład poniżej, gdzie mamy do czynienia z plikiem SWAP o rozmiarze 1,5 GiB. Został on
umieszczony na partycji `/` , gdzie zwykle plik SWAP będzie się znajdował. Jego stopień
fragmentacji prezentuje się następująco:

    # filefrag -ve /swapfile
    Filesystem type is: ef53
    File size of /swapfile is 1610612736 (393216 blocks of 4096 bytes)
    ext:     logical_offset:        physical_offset: length:   expected: flags:
      0:        0..     512:     361983..    362495:    513:
      1:      513..    1023:     362747..    363257:    511:     362496:
    ...
    111:   387072..  389119:    2899968..   2902015:   2048:    2873344:
    112:   389120..  393215:    2922496..   2926591:   4096:    2902016: last,eof
    /swapfile: 113 extents found

Zatem plik 1,5 GiB ma 113 zakresów bloków. Optymalna wartość dla tego pliku to 12 (1536MiB/128MiB).
Naturalnie z ludzkiej perspektywy 12 zakresów ciągłych bloków następujących jeden po drugim, gdy
plik ma rozmiar 1,5 GiB, jest równoznaczne z posiadaniem ciągłego pliku.

Oczywiście to, czy nasz plik SWAP ulegnie fragmentacji zależy od momentu, w którym go będziemy
chcieli stworzyć. Gdy proces tworzenia pliku SWAP przeprowadzimy przed instalacją systemu (nie wiem
jak do tego zadania podchodzą instalatory dystrybucji linux'a), gdzie na dysku nie ma jeszcze
żadnych plików, to naturalnie jest spora szansa na to, że plik SWAP będzie w miarę ciągły. Jeśli
jednak chcemy stworzyć plik SWAP po instalacji systemu lub też po dłuższym czasie używania systemu,
to wtedy jest niemal pewne, że plik SWAP będzie miał sporo części z racji dość dużej fragmentacji
wolnej przestrzeni. Dla przykładu, partycja `/` w moim systemie ma wielkość 30 GiB, z czego ponad
połowa jest pusta. A jak na tej partycji prezentuje się wolna przestrzeń? Zobaczmy:

    # e2freefrag /dev/mapper/wd_black_label-root
    Device: /dev/mapper/wd_black_label-root
    Blocksize: 4096 bytes
    Total blocks: 7864320
    Free blocks: 4175483 (53.1%)

    Min. free extent: 4 KB
    Max. free extent: 85632 KB
    Avg. free extent: 596 KB
    Num. free extent: 27934

    HISTOGRAM OF FREE EXTENT SIZES:
    Extent Size Range :  Free extents   Free Blocks  Percent
       4K...    8K-  :          5767          5767    0.14%
       8K...   16K-  :          3749          8890    0.21%
      16K...   32K-  :          2560         13146    0.31%
      32K...   64K-  :          4138         49365    1.18%
      64K...  128K-  :          3109         73707    1.77%
     128K...  256K-  :          1435         66024    1.58%
     256K...  512K-  :          1446        131012    3.14%
     512K... 1024K-  :          1492        269814    6.46%
       1M...    2M-  :          1934        703334   16.84%
       2M...    4M-  :          1409        968281   23.19%
       4M...    8M-  :           617        843599   20.20%
       8M...   16M-  :           200        524080   12.55%
      16M...   32M-  :            61        321823    7.71%
      32M...   64M-  :            15        158138    3.79%
      64M...  128M-  :             2         38503    0.92%

Niby mamy do dyspozycji ponad 50% wolnego miejsca, a największy ciągły kawałek pustej przestrzeni
ma jedynie około 80 MiB (85632 KiB), a średnio wychodzi to 0,5 MiB. Ja bym jednak wzbraniał się
umieścić plik SWAP w obrębie takiego systemu plików.

### Czy fragmentacja pliku SWAP ma znaczenie

[Po rozmowie](https://www.spinics.net/lists/linux-ext4/msg64788.html) z deweloperami systemu plików
ext4, okazało się jednak, że z tą fragmentacją danych też nie jest do końca tak jakby mogłoby się z
początku wydawać. O ile faktycznie fragmentacja dużego pliku jest całkiem prawdopodobna, toteż
spadek wydajności, jaki może za sobą ten proceder nieść, niekoniecznie musi wpłynąć negatywnie na
pracę systemu operacyjnego, o ile w ogóle będzie można tę różnice w jakiś sposób zmierzyć.

Plik w systemie plików jest zakodowany jako osobne kawałki (maksymalnie 128 MiB na pojedynczy
zakres bloków w ext4). Jeśli kilka zakresów bloków znajduje się obok siebie, to mamy ciągły obszar
danych na dysku. Z perspektywy maszyny są to osobne kawałki ale dla człowieka takie fragmenty jeden
za drugim mogą być traktowane jako jedna większa całość. Niemniej jednak, nawet ciągły obszar
bloków może być czytany przez system w kawałkach, z racji budowy komputera i ogólnie systemu
operacyjnego, bo wiele procesów próbuje zapisywać/odczytywać dysk w tym samym czasie, a tylko jeden
z nich w danym konkretnym momencie może to zrobić, przez co trzeba kolejkować operacje I/O dysku,
co przekłada się na ciągłość transferu danych (albo raczej na jej brak). Nawet jeśli plik ma wiele
ciągłych bloków danych na dysku, to zwykle w systemie zachodzą zdarzenia, które uniemożliwią
czytanie takiego ciągłego pliku w pojedynczej operacji I/O. Zatem jeśli plik będzie składał się z
jednego ciągłego kawałka i zostałby przy tym odczytany na raty w 20 operacjach, to z perspektywy
tej konkretnej operacji odczytu, plik można traktować tak jakby miał 20 części, a to w ilu
częściach figuruje on na dysku nie ma w tym konkretnym przypadku większego znaczenia. Oczywiście im
bardziej pofragmentowany jest plik, tym więcej operacji I/O trzeba przeprowadzić, by go odczytać
ale te parę dodatkowych zapytań większej różnicy w ogólnej wydajności systemu już raczej nie zrobi.

To co jest ciekawe w przypadku systemu plików ext4, to możliwość buforowania w pamięci RAM
wszystkich zakresów bloków pliku, który zamierzamy odczytać (przed dokonywaniem faktycznych
operacji I/O). Zatem lokalizacja wszystkich bloków danych jest znana przed rozpoczęciem transferu
danych i nie trzeba wykonywać dodatkowych zapytań do dysku, by poznać lokalizację następnych bloków.
Teoretycznie możliwe jest nawet odczytanie całego pliku, np. 10G, w jednym podejściu (jedna
operacja I/O) mniej więcej tak samo jak przy odczycie surowym np. przez `dd if=/dev/sda` , bo
głowica dysku może przejść płynie do następnego zakresu bloków bez ponownego pozycjonowania, co
wiązałoby się z dodatkowymi opóźnieniami. Czyli sam system plików jako taki nie wywołuje
dodatkowych opóźnień w stosunku do surowego odczytu danych z nośnika, oczywiście jeśli mamy do
odczytu kolejne ciągłe bloki danych.

Generalnie w tej całej zabawie chodzi o szybkość transferu danych pojedynczej operacji, która nie
jest jakoś specjalnie duża. Może i prędkość surowego transferu przeciętnego dysku HDD jest w
graniach 120 MiB/s ale w pojedynczej sekundzie system może chcieć przeprowadzić całą masę operacji
I/O, np. zapis logów systemowych czy odczyt plików z katalogu `/usr/` czy `/home/` przy
uruchamianiu aplikacji. Te wszystkie operacje wymagają osobnych zapytań, przez co trzeba je
kolejkować, co generuje opóźnienia. Dlatego też mając ciągły plik, może on zostać odczytany na raty.

I teraz wypadłoby zestawić te powyższe informacje z ilością zapytań pod kątem przestrzeni wymiany.
No jakby nie patrzeć, przy dość sporym wykorzystaniu pliku SWAP, tych operacji będzie bardzo dużo.
Niemniej jednak, zwykle jedynie małe porcje danych będą odczytywane lub zapisywane. Nie będzie przy
tym sytuacji, w których system będzie próbował odczytać lub zapisać naraz 1 GiB danych w pliku SWAP.
Dlatego też w przypadku przestrzeni wymiany, wpływ fragmentacji pliku SWAP nie degraduje zbytnio
w większym stopniu wydajności systemu w stosunku do partycji SWAP.

### Defragmentacja pliku SWAP

Jeśli już mowa o tych fragmentach pliku SWAP, które są fizycznie porozrzucane w obrębie danego
systemu plików, to zawsze można spróbować taki plik zdefragmentować przy pomocy `e4defrag` .
Niemniej jednak, trzeba się liczyć z faktem, że niekoniecznie w tej kwestii da się coś więcej
zrobić, przykładowo:

    # e4defrag -v /swapfile
    e4defrag 1.45.0 (6-Mar-2019)
    ext4 defragmentation for /swapfile
    [1/1]/swapfile: 100%  extents: 113 -> 113       [ OK ]
    Success:                       [1/1]

No w tym przypadku `e4defrag` nic nie zdziałał ze względu na dość mocną fragmentację wolnej
przestrzeni.

## Miejsce dla przestrzeni wymiany

No jak widać fragmentacja plików też nie jest jakoś specjalnie na niekorzyść dla pliku SWAP w
stosunku do partycji wymiany, no może poza pewnymi specyficznymi przypadkami, w których
fragmentacja plików jest bardzo ale to bardzo duża. Wypadałoby jednak się jeszcze zastanowić nad
kwestią miejsca, w którym plik SWAP powinien się znaleźć.

Jeśli mamy do dyspozycji drugi (niesystemowy) dysk, to przestrzeń wymiany (bez znaczenia czy plik
SWAP czy partycja) wypadałoby umieścić na tym właśnie nośniku. Taki zabieg rozłoży operacje I/O na
dwa urządzenia przez co poprawi ogólną wydajność systemu.

W przypadku, gdy dysponujemy tylko jednym dyskiem, to wypadałoby się postarać, by przestrzeń
wymiany znalazła się w miarę blisko początku dysku. Niekoniecznie musi być ona na samym początku
ale z racji lepszych czasów dostępu w tym obszarze nośnika, umieszczenie przestrzeni wymiany w nim
przełoży się na poprawę wydajności systemu. Spójrzmy na tę poniższą fotkę:

![](/img/2019/03/001-hdd-disk-performance-degradation-linux-partition.jpg#big)

Widać na niej, że początkowy obszar dysku ma zarówno większy transfer jak i niższe czasy dostępu do
danych. Gdybyśmy tworzyli partycję wymiany, to można by ją utworzyć w konkretnym miejscu. Z plikiem
już tak dobrze nie jest, bo może on powędrować w dowolne miejsce limitowane obszarem systemu
plików. Możemy za to zrobić pewien trik, który umieści plik wymiany w konkretnym wycinku dysku. Nie
możemy zatem tworzyć jednej dużej partycji dla naszego linux'a. Jeśli mamy do dyspozycji dysk 320G,
to spokojnie można utworzyć partycję `/` mającą 30 GiB zaraz na początku dysku i w niej utworzyć
już sobie plik SWAP. Taki zabieg sprawi, że zarówno system jak i przestrzeń wymiany będą
umieszczone w pierwszych 10-15% dysku, a to już im powinno zapewnić w miarę dobrą wydajność.

### Zmiana rozmiaru pliku SWAP

Czasami przez zbytnie niedoszacowanie, plik SWAP podczas instalacji systemu, czy też przed tym
procesem, może zostać stworzony za mały lub za duży. Zmiana rozmiaru pliku przestrzeni wymiany może
czasami nieść ze sobą utratę wydajności. Jeśli korzystamy z systemu już dłuższy czas, to można
założyć, że fragmentacja wolnej przestrzeni na partycji `/` poszła w najlepsze (tak jak to było
widać wyżej w moim przypadku).

Tworzenie nowego pliku SWAP lub rozszerzenie tego istniejącego sprawi, że część przestrzeni wymiany
ulegnie pofragmentowaniu. Wypadałoby zatem rozeznać się wcześniej jak wygląda struktura wolnej
przestrzeni na dysku, bo może się okazać, że przeznaczenie kolejnej części nośnika w tym konkretnym
systemie plików może nie być zbytnio wskazane, zwłaszcza, gdy na danej partycji mamy już bardzo
niewiele wolnego miejsca.

## Problemy z hibernacją za sprawą pliku SWAP

Wygląda na to, że są pewne problemy w przypadku hibernacji systemu, gdy w roli przestrzeni wymiany
występuje plik
SWAP. [Problem tkwi w nagłówku przestrzeni wymiany](https://www.kernel.org/doc/Documentation/power/swsusp-and-swap-files.txt),
którego lokalizacja w przypadku partycji jest znana, a w przypadku pliku trzeba ją systemowi
wskazać. Wszystko dlatego, że system plików, na którym znajduje się plik SWAP, trzeba by pierw
zamontować. Kłopot jedynie w tym, że systemu plików zawierającego dziennik (np. ext3, czy ext4) nie
można zamontować podczas wybudzania maszyny ze stanu hibernacji. Hibernacja w przypadku pliku SWAP
jest naturalnie możliwa ale trzeba
się [trochę wysilić przy konfiguracji](/post/problemy-z-plikiem-wymiany-swap-przy-hibernacji-linuxa/),
by wszystko działało jak należy.

## Plik SWAP vs. partycja wymiany

Po przeanalizowaniu wszystkich za i przeciw, przynajmniej tych, które przyszły mi do głowy,
wychodzi na to, ze plik SWAP (przy odpowiednim podejściu) jest nieco lepszym rozwiązaniem w
stosunku do partycji wymiany. Może i pewne rzeczy związane z hibernacją wymagają dodatkowego
wysiłku ale poza tym, to nie widziałbym przeszkód, by zmienić nieco konfigurację swojego systemu,
tak by zamiast partycji wymiany korzystać z pliku SWAP. Oczywiście w obecnej sytuacji pozostaje mi
jedynie używanie przestrzeni wymiany w postaci partycji.
