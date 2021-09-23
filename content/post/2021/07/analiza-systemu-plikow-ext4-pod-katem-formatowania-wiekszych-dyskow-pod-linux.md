---
author: Morfik
categories:
- Linux
date:    2021-07-25 18:40:00 +0200
lastmod: 2021-07-31 21:25:00 +0200
published: true
status: publish
tags:
- debian
- hdd
- ssd
- ext4
- udev
- system-plików
- apm
title: Analiza systemu plików EXT4 pod kątem formatowania większych dysków pod linux
---

Zapewne każdy użytkownik linux'a tworzył na dysku HDD/SSD partycje sformatowane systemem plików
EXT4. Prawdopodobnie też zastanawiało nas pytanie odnośnie ilości zajmowanego miejsca przez
strukturę samego systemu plików, zwłaszcza w przypadku dysków o sporych rozmiarach (setki GiB, czy
nawet kilka TiB). Jako, że musiałem ostatnio zmigrować kolekcję filmów ze strych dysków na
jeden większy, który miał zostać podłączony pod Raspberry Pi z działającym Kodi na bazie LibreELEC,
to przy okazji postanowiłem ten dysk sformatować w taki sposób, w jaki powinno się do tego zadania
podchodzić wiedząc, że ma się do czynienia z dużym dyskiem, na którym będą przechowywane głównie
duże pliki. Celem tego artykułu jest pokazanie jakie błędy przy tworzeniu systemu plików EXT4 można
popełnić przez posiadanie niezbyt wystarczającej wiedzy z jego zakresu, oraz jak te błędy
wyeliminować przed rozpoczęciem korzystania z tak nie do końca poprawnie przygotowanego do pracy
dysku twardego

<!--more-->
## Nie tylko pliki zajmują miejsce na dysku

Może to być ździebko nieintuicyjne ale nie tylko pliki czy katalogi (które też są plikami tylko
nieco innego rodzaju) zajmują nam miejsce na dysku twardym. By dane zgromadzone na dysku mogły być
plikami, potrzebna jest jakaś forma struktury opisowej, zwanej systemem plików. Ta struktura
również zawiera dane, z tym, że niezbyt czytelne dla człowieka. Natomiast maszyny są w stanie w
oparciu o tę strukturę pogrupować bity interesujących na informacji, tak by zostały one
użytkownikom komputera przedstawione w formie czytelnej w plikach graficznych, tekstowych, audio,
video czy też i innych formatach, które aplikacje użytkowe będą w stanie zrozumieć.

Jak można się domyśleć, te metadane opisujące pliki zajmują miejsce i potrafią go zajmować dość
sporo, w zależności od tego co zamierzamy na tym HDD czy SSD trzymać. Jedno jest jednak pewne, cała
struktura opisowa systemu plików EXT4 jest tworzona podczas inicjacji systemu plików (przy
wydawaniu polecenia `mke2fs ...` ). Dlatego też po utworzeniu systemu plików, część danych jest
już zajęta przez metadane systemu plików i zwykle jest to wiele ładnych GiB zwłaszcza w przypadku
większych systemów plików. Do tego dochodzi jeszcze kilka innych rzeczy, takie jak, np.
zarezerwowane miejsce dla użytkownika root, czy też rozmiar dziennika systemu plików, które też
potrafią nieźle utylizować wolną na dysku przestrzeń.

Gdy się pozbiera wszystkie te rzeczy do kupy, to nasz 2TB dysk może z powodzeniem stracić na
starcie nawet i 200GiB i nie mówię tutaj wcale o różnicy między GB (podstawa potęgi 10) oraz GiB
(podstawa potęgi 2) w wykonaniu producentów dysku, tylko o faktycznej utracie pojemności na
poziomie 200GiB. Kupując dysk 2TB, w systemie będziemy widzieć nośnik około 1820GiB. Z tego
wszystkiego może nam wyparować tak około 10%, czyli 182GiB, przez co na pliki zostanie nam jedynie
1638GiB. A miało być 2TB... Na szczęście jest kilka zabiegów, które mogą nam pomóc odzyskać lwią
część przestrzeni, która w przypadku trzymania na dysku samych dużych plików poszłaby na
zmarnowanie.

## Rzut oka na superblok systemu plików EXT4

Każdy system plików EXT4 na samym początku swojej struktury zawiera superblok. To taki blok,
którego celem jest połączenie pozostałej struktury systemu plików w całość. Jeśli superblok ulegnie
uszkodzeniu, to ulega też uszkodzeniu cały system plików. By jakoś działać przeciwko uszkodzeniu
całego systemu plików za sprawą tylko pojedynczego bloku, kopie superbloka są utrzymywane też w
innych obszarach systemu plików. W takim przypadku, gdy dochodzi do uszkodzenia superbloka, zawsze
można ratować się tymi zapasowymi blokami i zwykle odzyskać część albo nawet całość danych
przechowywanych w obrębie uszkodzonego systemu plików.

Rzućmy okiem zatem na przykładowy system plików przy pomocy `dumpe2fs` . To narzędzie ma na celu
zwrócić szereg informacji opisujących strukturę systemu plików, w tym również informacje zawarte w
superbloku:

    # dumpe2fs /dev/sda1
    dumpe2fs 1.46.2 (28-Feb-2021)
    Filesystem volume name:   boot
    Last mounted on:          /boot
    Filesystem UUID:          d6c1ab6c-1645-4b87-bdc7-7ae9c1708e1c
    Filesystem magic number:  0xEF53
    Filesystem revision #:    1 (dynamic)
    Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent 64bit flex_bg sparse_super large_file huge_file dir_nlink extra_isize metadata_csum
    Filesystem flags:         signed_directory_hash
    Default mount options:    user_xattr acl
    Filesystem state:         clean
    Errors behavior:          Continue
    Filesystem OS type:       Linux
    Inode count:              131072
    Block count:              524032
    Reserved block count:     0
    Free blocks:              398550
    Free inodes:              130950
    First block:              0
    Block size:               4096
    Fragment size:            4096
    Group descriptor size:    64
    Reserved GDT blocks:      255
    Blocks per group:         32768
    Fragments per group:      32768
    Inodes per group:         8192
    Inode blocks per group:   512
    Flex block group size:    16
    Filesystem created:       Tue Mar  3 17:11:45 2020
    Last mount time:          Wed Jul 21 11:54:47 2021
    Last write time:          Wed Jul 21 12:17:40 2021
    Mount count:              50
    Maximum mount count:      50
    Last checked:             Tue Mar 24 11:36:22 2020
    Check interval:           0 (<none>)
    Lifetime writes:          38 GB
    Reserved blocks uid:      0 (user root)
    Reserved blocks gid:      0 (group root)
    First inode:              11
    Inode size:               256
    Required extra isize:     32
    Desired extra isize:      32
    Journal inode:            8
    Default directory hash:   half_md4
    Directory Hash Seed:      848800c8-9fb0-4a76-b5ec-6c8e4a8f403f
    Journal backup:           inode blocks
    Checksum type:            crc32c
    Checksum:                 0xcc2eee2e
    Journal features:         journal_incompat_revoke journal_64bit journal_checksum_v3
    Total journal size:       32M
    Total journal blocks:     8192
    Max transaction length:   8192
    Fast commit length:       0
    Journal sequence:         0x0000258e
    Journal start:            0
    Journal checksum type:    crc32c
    Journal checksum:         0xceb54c66


    Group 0: (Blocks 0-32767) csum 0x9959 [ITABLE_ZEROED]
      Primary superblock at 0, Group descriptors at 1-1
      Reserved GDT blocks at 2-256
      Block bitmap at 257 (+257), csum 0x29bf2073
      Inode bitmap at 273 (+273), csum 0xbd985a57
      Inode table at 289-800 (+289)
      7861 free blocks, 8071 free inodes, 2 directories, 8059 unused inodes
      Free blocks: 7970-8480, 15143-16383, 26659-32767
      Free inodes: 28, 121-124, 127-8192
    Group 1: (Blocks 32768-65535) csum 0x9dff [ITABLE_ZEROED]
      Backup superblock at 32768, Group descriptors at 32769-32769
      Reserved GDT blocks at 32770-33024
      Block bitmap at 258 (bg #0 + 258), csum 0x647e6f93
      Inode bitmap at 274 (bg #0 + 274), csum 0x2cb0a6e4
      Inode table at 801-1312 (bg #0 + 801)
      7788 free blocks, 8191 free inodes, 1 directories, 8191 unused inodes
      Free blocks: 33026-33027, 33088-33125, 33769-33791, 33796-33855, 33897-34303, 42496-42751, 42793-42845, 42934-43007, 43784-44031, 45153-46079, 53248-54015, 54118-54153, 54245, 54248-54271, 54375-55869, 55872-56319, 56423-57343, 58399-59391, 64512-65407, 65412-65417, 65424-65535
      Free inodes: 8194-16384
    Group 2: (Blocks 65536-98303) csum 0x53c2 [INODE_UNINIT, ITABLE_ZEROED]
      Block bitmap at 259 (bg #0 + 259), csum 0x7311aaa4
      Inode bitmap at 275 (bg #0 + 275), csum 0x00000000
      Inode table at 1313-1824 (bg #0 + 1313)
      2991 free blocks, 8192 free inodes, 0 directories, 8192 unused inodes
      Free blocks: 68569-68607, 69743-69759, 69801-70655, 74799-74815, 74857-75775, 77798-77823, 82919-82943, 83055-83967, 93004-93183
      Free inodes: 16385-24576
    Group 3: (Blocks 98304-131071) csum 0xac8d [INODE_UNINIT, ITABLE_ZEROED]
      Backup superblock at 98304, Group descriptors at 98305-98305
      Reserved GDT blocks at 98306-98560
      Block bitmap at 260 (bg #0 + 260), csum 0x4c57e333
      Inode bitmap at 276 (bg #0 + 276), csum 0x00000000
      Inode table at 1825-2336 (bg #0 + 1825)
      22258 free blocks, 8192 free inodes, 0 directories, 8192 unused inodes
      Free blocks: 98561-98623, 98665-99327, 99453-100351, 103244-105291, 106261-107199, 107241-108543, 110592-114687, 116736-118783, 119752-119807, 120832-129023, 129121-131071
      Free inodes: 24577-32768
    ...
    Group 15: (Blocks 491520-524031) csum 0x443b [INODE_UNINIT, ITABLE_ZEROED]
      Block bitmap at 288 (bg #0 + 288), csum 0x8abeaabd
      Inode bitmap at 7969 (bg #0 + 7969), csum 0x00000000
      Inode table at 8487-8998 (bg #0 + 8487)
      32512 free blocks, 8192 free inodes, 0 directories, 8192 unused inodes
      Free blocks: 491520-524031
      Free inodes: 122881-131072

Mamy tutaj do czynienia z bardzo małym systemem plików, bo ma on rozmiar zaledwie 2GiB ale idealnie
nada się do analizy tego, co w strukturze EXT4 się znajduje.

W pierwszej części tego powyższego listingu mamy informacje zawarte w superbloku. W drugiej części
zaś znajdują się informacje dotyczące deskryptorów grup bloków (block group descriptors). Sam
superblok zajmuje tyle ile blok systemu plików (zwany też klastrem). W tym przypadku rozmiar bloku
to 4KiB, tj. `Block size: 4096` . Z kolei te 4096 bajtów to 8 fizycznych sektorów 512-bajtowych
dysku).

Taki klaster to najmniejsza jednostka alokacji plików, tj. nawet jeśli na takim systemie plików
umieścimy mniejszy plik, powiedzmy 500 bajtów, to i tak będzie on zajmował 4KiB. W obecnych czasach
pliki zwykle mają nieco większe rozmiary, grubo przekraczając te 4KiB, i gdyby rozmiar bloku był
równy rozmiarze pojedynczego sektora dysku, to takie rozwiązanie byłoby wysoce nieefektywne. W
niektórych przypadkach, np. gdy trzymamy same duże pliki na dysku, których rozmiar idzie w GiB,
taki 4KiB rozmiar klastra jest w dalszym ciągu bardzo nieefektywnym rozwiązaniem ale o tym później.

### Grupa bloków

Na powyższym listingu mamy 16 grup bloków oznaczonymi numerkami od 0 do 15. Każda taka grupa bloków
jest w stanie opisać (standardowo) do 128MiB danych, które będziemy umieszczać na dysku twardym i
nie więcej. Czemu tylko tyle? W superbloku widnieje informacja ile bloków przypada na jedną grupę
bloków: `Blocks per group: 32768` , czyli 32768*4096=134.217.728 bajtów, zatem 128MiB.

Weźmy zatem przykładową grupę bloków i przyjrzyjmy się jej nieco bliżej:

    Group 3: (Blocks 98304-131071) csum 0xac8d [INODE_UNINIT, ITABLE_ZEROED]
      Backup superblock at 98304, Group descriptors at 98305-98305
      Reserved GDT blocks at 98306-98560
      Block bitmap at 260 (bg #0 + 260), csum 0x4c57e333
      Inode bitmap at 276 (bg #0 + 276), csum 0x00000000
      Inode table at 1825-2336 (bg #0 + 1825)
      22258 free blocks, 8192 free inodes, 0 directories, 8192 unused inodes
      Free blocks: 98561-98623, 98665-99327, 99453-100351, 103244-105291, 106261-107199, 107241-108543, 110592-114687, 116736-118783, 119752-119807, 120832-129023, 129121-131071
      Free inodes: 24577-32768

Mamy tutaj informację, że w tej grupie bloków, zapasowy superblok znajduje się na pozycji 98304,
zaś zapasowe deskryptory grup bloków na 98305. Ten dodatkowy blok na superblok ma rozmiar 4096
bajtów, podobnie blok z informacjami o deskryptorach grup bloków również ma rozmiar 4096 bajtów.

#### Zarezerwowane bloki GDT i flaga resize_inode

Dalej mamy informację o zarezerwowanych blokach GDT (Group Descriptor Table) w liczbie 254. Te
zarezerwowane bloki GDT znajdują jedynie zastosowanie podczas poszerzania systemu plików i taki
jest ich jedyny cel, tj. umożliwić powiększenie systemu plików. Warto tutaj zaznaczyć, że raz
utworzonego systemu plików EXT4 nie da się powiększyć w nieskończoność, tylko o rząd wielkości 1024
razy (domyślnie). Więc jeśli stworzyliśmy system plików o rozmiarze 1GiB, to maksymalny rozmiar do
jakiego taki system plików można rozciągnąć, np. po zmianie rozmiaru partycji, to 1024 GiB. Niby
ten limit 1024 razy wydaje się spory i raczej nikt z nas nigdy w niego nie uderzył przy
powiększaniu systemu plików ale warto mieć go na uwadze przy znacznym powiększaniu bardzo małych
systemów plików.

W przypadku rozciągnięcia systemu plików, zarezerwowane bloki GDT ulegną skurczeniu ale
jednocześnie ulegnie zwiększeniu obszar zajmowany przez deskryptory grup bloków, które standardowo
mają do dyspozycji jeden blok 4096-bajtowy. Dlatego wyżej w zapisie mamy zakres `98305-98305`
zamiast pojedynczej liczby (tak jak to ma miejsce w przypadku superbloku), co oznacza dokładnie
jeden blok, co z kolei informuje nas, że system plików nie był poszerzany w żaden sposób.

Kopia superbloka, kopia deskryptorów grup bloków oraz zarezerwowane bloki GDT łącznie przy
standardowych opcjach przy tworzeniu systemu plików EXT4 dają 256 bloków 4096-bajtowych, czyli
1MiB.

Co ciekawe, jeśli nie planujemy w przyszłości powiększać systemu plików EXT4, np. tworzymy system
plików na partycji rozciągającej się na całym HDD/SSD, to te zarezerwowane bloki GDT będą nam
zupełnie zbędne i niepotrzebnie będą zajmować miejsce. Możemy zatem się ich pozbyć usuwając
domyślnie ustawioną flagę `resize_inode` przy tworzeniu systemu plików.

#### Tablica i-węzłów

Dalej mamy informację o tablicy i-węzłów (Inode table), która ma rozmiar 512 bloków 4096-bajtowych,
czyli 2MiB. Czemu akurat tyle? Każdy i-węzeł posiada swój rozmiar i obecnie domyślnym rozmiarem
i-węzłów w systemach plików EXT4 jest 256 bajtów. Każda grupa bloków ma określoną liczbę i-węzłów.
Standardowo jest to 8192, o czym możemy się przekonać rzucając okiem na linijkę z `Inodes per
group: 8192` . Jeśli pomnożymy ilość i-węzłów przez ich rozmiar, to otrzymamy 8192*256=2097152
bajtów, czyli 2MiB.

#### Opcje sparse_super i sparse_super2

Biorąc pod uwagę te powyższe informacje, w takiej pojedynczej grupie bloków, która standardowo
opisuje 128 MiB danych, 3 MiB trzeba odliczyć na strukturę samego systemu plików EXT4, czyli około
2,34%. No jest to dość sporo jakby nie patrzeć, co niekoniecznie może aż tak rzucać się w oczy w
przypadku mniejszych systemów plików (które też mogą mieć 128-bajtowe i-węzły). Niemniej jednak, by
zoptymalizować miejsce potrzebne na metadane systemu plików, w większości linux'ów (albo i we
wszystkich), system plików EXT4 jest tworzony z flagą `sparse_super` .

Gdyby utworzyć system plików bez tej flagi, kopie superbloku oraz deskryptorów grup bloków
zostałyby umieszczone w każdej grupie bloków. Jako, że w tym przypadku flaga `sparse_super` jest
obecna (widoczna na powyższym listingu pod `Filesystem features:` ), to te kopie są umieszczone
tylko w tych grupach, których numer wskazuje na 0 oraz jest potęgą liczby 3, 5 lub 7. Taki zabieg
ma na celu ograniczyć ilość tworzonych kopi zapasowych tych kluczowych elementów struktury do
minimum, przy jednoczesnym zachowaniu możliwości odzyskania danych w sytuacji, gdy system plików
ulegnie z jakiegoś powodu uszkodzeniu.

Oczywiście, gdyby system plików nie miał flagi `sparse_super` , to wtedy każda grupa bloków miałaby
ten dodatkowy narzut 1MiB związany z obsługą systemu plików EXT4. Jako, że my tę flagę
`sparse_super` mamy ustawioną, to niewiele grup posiada ten zarezerwowany dodatkowy 1 MiB
przeznaczony na kopię superbloku i kopię deskryptorów grup bloków. Zatem to zarezerwowane miejsce
jest marginalne w przypadku dużych dysków ale wciąż każda grupa musi mieć tablicę i-węzłów, przez
co na każde 128MiB, 2MiB jest właśnie na nią przeznaczone, co zajmuje około 1,6% wolnego miejsca na
dysku, które trzeba będzie na tę strukturę systemu plików przeznaczyć.

Niemniej jednak, w przypadku pojemniejszych dysków twardych, flaga `sparse_super` może nie do końca
znaleźć zastosowanie, bo jakby nie patrzeć te dodatkowe kopie superbloku i kopie deskryptorów grup
bloków fragmentują wolną przestrzeń, co nie jest czasem pożądanym zjawiskiem. Dlatego też
opracowano flagę `sparse_super2` , której celem jest utworzenie tylko dwóch dodatkowych kopi
zapasowych superbloka i kopi deskryptorów grup bloków. Po zastosowaniu tej flagi, grupa `0` będzie
przechowywać podstawowy superblok i podstawowe deskryptory grup bloków, natomiast ich kopie będą
się znajdować w `pierwszej` i `ostatniej` grupie. W takim przypadku, gdy główny superblok
ulegnie awarii, to bez problemu będzie można odzyskać system plików posługując się kopią superbloka
zlokalizowaną na początku lub na końcu dysku. Cała przestrzeń między tymi grupami będzie ciągła, co
widać na poniższym listingu:

    # e2freefrag /dev/sdb1
    Device: /dev/sdb1
    Blocksize: 4096 bytes
    Total blocks: 366284032
    Free blocks: 366202942 (100.0%)

    Min. free extent: 8512 KB
    Max. free extent: 2096896 KB
    Avg. free extent: 2086624 KB
    Num. free extent: 702

    HISTOGRAM OF FREE EXTENT SIZES:
    Extent Size Range :  Free extents   Free Blocks  Percent
        8M...   16M-  :             1          2128    0.00%
       64M...  128M-  :             2         64202    0.02%
        1G...    2G-  :           699     366136612   99.98%

A tak by wyglądała wolna przestrzeń po standardowym formatowaniu:

    # e2freefrag /dev/sdb1
    Device: /dev/sdb1
    Blocksize: 4096 bytes
    Total blocks: 366284032
    Free blocks: 366183742 (100.0%)

    Min. free extent: 125992 KB
    Max. free extent: 2096896 KB
    Avg. free extent: 2040020 KB
    Num. free extent: 718

    HISTOGRAM OF FREE EXTENT SIZES:
    Extent Size Range :  Free extents   Free Blocks  Percent
       64M...  128M-  :             7        227722    0.06%
      128M...  256M-  :             5        321680    0.09%
      256M...  512M-  :             2        195344    0.05%
      512M... 1024M-  :             6       1174720    0.32%
        1G...    2G-  :           698     364264276   99.48%

Może te kilkanaście dodatkowych kawałków wolnej przestrzeni wydaje się nie dużo ale gdy w grę
wchodzą duże pliki, to ta wolna przestrzeń będzie się łatwiej fragmentować niż w tym pierwszym
przypadku, zwłaszcza, gdy w późniejszym czasie tych plików będzie nam przybywać.

Można też pójść o krok dalej i mając ustawioną flagę `sparse_super2` , ustawić flagę
`num_backup_sb` na wartość `0` i w ten sposób wyłączyć tworzenie zapasowych kopi superbloka. Trzeba
jednak zdawać sobie sprawę, że gdy główny superblok ulegnie z jakiegoś powodu uszkodzeniu, to wtedy
odzyskanie danych może już nie być możliwe, albo to zadanie będzie o wiele trudniejsze niż w
przypadku, gdyby choć jedna kopia superbloku na dysku była obecna.

#### Bitmapa bloków i i-węzłów

Każda grupa bloków zawiera także bitmapę bloków i i-węzłów. Te bitmapy zawierają informację na
temat bloków i i-węzłów, które są w użyciu. Każda z tych bitmap zajmuje po jednym bloku
4096-bajtowym, czyli łącznie 8KiB, które można pominąć w wyliczeniach, bo nie stanową one jakiejś
większej wartości przy 128MiB.

## Optymalizacja systemu plików pod kątem przechowywania dużych plików

Mając już jakieś pojęcie na temat struktury systemu plików EXT4, możemy nieco zoptymalizować pewne
parametry, by niepotrzebnie nie tracić wolnego miejsca na dysku pod zbędną strukturę systemu plików,
z której i tak nie będziemy nigdy korzystać. Pierwsza optymalizacja jest w zasadzie domyślnie
włączona i była mowa o niej wyżej, tj. o flagach `sparse_super`/`sparse_super2` . Są one w stanie
mam zaoszczędzić około 1MiB na każde 128 MiB. Niemniej jednak, nie jest to jedyna opcja, którą
powinniśmy w przypadku dużych dysków przechowujących bardzo duże pliki włączyć czy zmienić.

### Flaga bigalloc

Jeśli dany system plików EXT4 ma zawierać głównie duże pliki (np. na moim 2TB dysku najmniejszy
plik miał rozmiar trochę ponad 1GiB), to jednostka alokacji (klaster) w postaci bloku o rozmiarze
4KiB nie jest optymalnym rozwiązaniem dla trzymania tak dużych plików. Przydałby się zatem nieco
większy klaster, np. 1MiB albo może i nawet 4MiB. W taki sposób, bitmapa alokacji bloków będzie
śledzić przykładowo klastry 4MiB zamiast pojedynczych bloków 4KiB.

Taki zabieg sprawi, że pojedyncza grupa bloków będzie w stanie opisać 128GiB (w stosunku do
poprzednich 128MiB), a to z racji, że każdy bit w bitmapie alokacji bloków (block allocation
bitmap) będzie adresował 1024 bloki o rozmiarze 4KiB (łącznie 4MiB). Taki zabieg sprawia, że
skurczeniu ulega rozmiar bitmapy alokacji bloków, dla przykładu dla 2TiB systemu plików mamy
redukcję z 64MiB do 64KiB (bo bitmapa bloków zajmuje 4KiB i jednocześnie ulega zmniejszeniu liczba
grup do 16, w stosunku do wcześniejszych 16384), co z kolei dość znacznie redukuje narzut związany
z obsługą metadanych samego systemu plików EXT4.

Idąc dalej, przy rozmiarze klastra 4MiB, plik 1GiB będzie o wile łatwiej umieścić w jednym kawałku
na dysku, niż gdy mamy do czynienia z klastrami 4KiB (256 kawałków vs. 262144). Zatem fragmentacja
systemu plików będzie sporo mniejsza przy stosowaniu klastrów o większym rozmiarze w stosunku do
standardowych klastrów 4KiB.

#### Problemy związane z bigalloc

Flaga `bigalloc` ma jedną bardzo poważną wadę. Jeśli zwiększeniu ulegnie klaster przykładowo z 4KiB
do 4MiB, to bardzo kosztowne będzie dla nas trzymanie w takim systemie plików bardzo małych plików,
np. 5-10KiB, bo będą one zajmować sporo miejsca, przykładowo 800-bajtowy plik tekstowy, będzie
ważył 4MiB. Podobnie sprawa będzie wyglądać z katalogami, czyli jeśli mamy ich sporo, to ich
struktura może sama z siebie bardzo dużo zajmować w takim systemie plików z ustawioną flagą
`bigalloc` . Jeśli jesteśmy pewni, że nie zamierzamy trzymać w takim systemie plików bardzo małych
plików albo też nie mamy rozbudowanej struktury drzewa katalogów, to bez problemu tę flagę
powinniśmy sobie ustawić.

Niestety jeśli utworzy się system plików z `bigalloc` (lub bez niego), to nie ma możliwości zmiany
tego ustawienia bez późniejszego usuwania danych zgromadzonych w obrębie takiego systemu plików.
Dlatego trzeba się mocno zastanowić czy tę flagę ustawić.

Poważniejszy problem z flagą `bigalloc` zdaje się być taki, że najwyraźniej nie jest ten ficzer
jeszcze do końca sprawdzony i [może powodować problemy][11] w pewnych sytuacjach, przynajmniej tak
można wyczytać w `man ext4` i pod powyższym linkiem.

Jako, że ta flaga `bigalloc` nie dawała mi spokoju, to postanowiłem [zapytać na mailing list
kernel-ext4][19] czy można z niej korzystać i czy wiąże się to z jakimiś przykrymi konsekwencjami.
Wygląda jednak na to, że nic nie stoi na przeszkodzie, by flagi `bigalloc` używać, przynajmniej
jeśli mamy w miarę nowy kernel, choć z tego co Theodore Ts'o napisał, to ta flaga nie została
jeszcze dobrze przetestowana jeśli chodzi o wsparcie dla `FALLOC_FL_COLLAPSE_RANGE` ,
`FALLOC_FL_INSERT_RANGE` oraz `FALLOC_FL_PUNCH_HOLE` . Mi to zbytnio nic nie mówi ale z tonu
wypowiedzi mogę wnioskować, że generalnie jest zielone światło dla flagi `bigalloc` i można z niej
korzystać.

#### EXT4-fs Online defrag not supported with bigalloc

Przy próbie defragmentacji systemu plików EXT4 z ustawioną flagą `bigalloc` , przywitał mnie
komunikat `kernel: EXT4-fs (sdb1): Online defrag not supported with bigalloc` . Muszę przyznać, że
trochę się zdziwiłem ale najwyraźniej wygląda na to, że nie ma opcji by dokonać defragmentacji
systemu plików EXT4, gdy ten ma ustawiony klaster o rozmiarze innym niż domyślna wielkość bloku,
tj. 4096 bajtów. Zatem trzeba się zdecydować, czy wolimy mniejsze kawałki plików z możliwością
defragmentacji, czy większe bez. Po chwili wgrywania danych na taki nośnik z ustawionym klastrem
4MiB, doszedłem jednak do wniosku, że ten cały `bigalloc` się nadaje jedynie do pozostawienia go w
spokoju, a może za parę lat ktoś go dopracuje na tyle, by można było z niego w jakiś racjonalny
sposób korzystać.

### Rozmiar oraz ilość i-węzłów

Przy formatowaniu partycji systemem plików EXT4, tworzona jest określona ilość i-węzłów (i-node).
Ich ilość jest proporcjonalna do wielkości naszego dysku twardego. Im ten dysk (czy partycja) jest
większy, tym więcej i-węzłów będziemy mieli w jego obrębie do dyspozycji.

Ilość i-węzłów przekłada się jeden w jeden na liczbę plików, które będziemy mogli na dysku utworzyć.
Trzeba jednak pamiętać, że i-węzeł swoje waży, tj. [w systemie plików EXT4 ma on standardowo
rozmiar 256 bajtów][1]. Zatem określenie zarówno zbyt małej (jak i zbyt dużej) liczby i-węzłów nie
jest dobrym rozwiązaniem. Z jednej strony może zabraknąć nam i-węzłów, przez co nie będziemy mogli
stworzyć nowych plików, mając jednocześnie sporo wolnego miejsca na dysku. Z drugiej strony, zbyt
wiele wolnych i-węzłów będzie nam marnować dużo wolnej przestrzeni.

Przy standardowym formatowaniu partycji systemem plików EXT4, na każde 16KiB wolnego miejsca na
dysku będzie przypadał jeden i-węzeł. Zatem teoretycznie możliwa jest sytuacja, w której tworząc
bardzo małe pliki, skończą nam się i-węzły i dalsze tworzenie nowych plików nie będzie możliwe mimo,
że np. 3/4 powierzchni dysku będzie leżeć odłogiem. Biorąc jednak po uwagę realia, bardzo rzadko
ktoś trzyma na dysku bardzo wiele plików o rozmiarze <16KiB. Dlatego takie uśrednienie (jeden
i-węzeł na każde 16KiB) jest w miarę do zaakceptowania, przynajmniej w standardowych warunkach.

Mając jednak do czynienia z 1820GiB dyskiem, gdybyśmy chcieli na nim utworzyć i-węzły w standardowy
sposób, to ich liczba by wyniosła około 120.000.000. Jeśli teraz przemnożymy tę wartość przez 256
bajtów, to otrzymamy 30.720.000.000 bajtów, czyli około 28.5GiB. Tyle trzeba by przeznaczyć miejsca
na dysku na same i-węzły -- tak, prawie 30GiB, czyli te wcześniej wyliczone 1,6% całkowitej
powierzchni użytkowej dysku.

Wiedząc jednak, że na dysku zamierzamy trzymać w zasadzie tylko i wyłączenie duże pliki, możemy
ograniczyć ilość i-węzłów, które zostaną stworzone w systemie plików. Możemy naturalnie wskazać tę
liczbę ręcznie podczas tworzenia systemu plików ale lepszym rozwiązaniem jest korzystanie z
opcji `-T largefile` lub `-T largefile4` , które trzeba podać w `mke2fs` . Obie z tych opcji
mają za zadanie zmienić domyślny współczynnik jeden i-węzeł/16KiB na jeden i-węzeł/1MiB lub jeden
i-węzeł/4MiB. Nie należy jednak tych opcji mylić z `bigalloc` . Tutaj tylko określamy ilość
i-węzłów i jeśli określimy sobie jeden i-węzeł/4MiB i zamiast dużych plików będziemy trzymać małe
pliki, to raz dwa nam tych i-węzłów zabraknie. Niemniej jednak, wciąż setki tysięcy plików będziemy
mogli w obrębie takiego systemu plików stworzyć, więc kilka mniejszych plików nam zbytniej różnicy
nie zrobi.

W przypadku opcji `-T largefile` lub `-T largefile4` również trzeba mieć na uwadze fakt, że raz
określona ilość i-węzłów w systemie plików nie podlega późniejszej negocjacji. Jeśli w późniejszym
czasie dojdziemy do wniosku, że źle obraliśmy ilość i-węzłów, to trzeba będzie na nowo tworzyć
system plików, co będzie wiązało się z utratą wszelkich danych na nim zgromadzonych.

Warto też wspomnieć, że im mniej i-węzłów będzie w strukturze systemu plików, tym szybciej będzie
przebiegał proces sprawdzania systemu plików pod kątem ewentualnych błędów via `fsck` .

### Pełne inicjowanie systemu plików przy jego tworzeniu

Raz na forum dug.net.pl był [wątek o ciągłej aktywności dysku tuż po utworzeniu na nim systemu
plików EXT4][3]. Zgodnie z tym, co autor wątku napisał, system plików EXT4 był tworzony z
domyślnymi opcjami. Ostatecznie po góglaniu za takimi dziwnymi objawami, [doszukałem się
informacji, które wskazywały na nie w pełni zainicjowany systemu plików podczas jego tworzenia][4],
o czym można również przeczytać w `man mke2fs` `i man ext4` :

> lazy_itable_init[= <0 to disable, 1 to enable>]
>
>   If  enabled and the uninit_bg feature is enabled, the inode table will not be fully
>   initialized by mke2fs. This speeds up filesystem initialization noticeably, but it
>   requires the kernel to finish initializing the  filesystem in the background when the
>   filesystem is first mounted. If the option value is omitted, it defaults to 1 to enable
>   lazy inode table zeroing.
>
> lazy_journal_init[= <0 to disable, 1 to enable>]
>
>   If enabled, the journal inode will not be fully zeroed out by mke2fs. This speeds up
>   filesystem initialization  noticeably, but carries some small risk if the system crashes
>   before the journal has been overwritten entirely one time. If the option value is
>   omitted, it defaults to 1 to enable lazy journal inode zeroing.
>
> uninit_bg
>
>   This ext4 file system feature indicates that the block group descriptors will be
>   protected using checksums, making it safe for mke2fs(8) to create a file system without
>   initializing all of the block groups. The kernel will keep a high watermark of unused
>   inodes, and initialize inode tables and blocks lazily. This feature speeds up the time
>   to check the file system using e2fsck(8), and it also speeds up the time required for
>   mke2fs(8) to create  the file system.

Zgodnie z tymi powyższymi informacjami, można utworzyć system plików nie w pełni zainicjowany, co
przyśpiesza dość znacznie czas całego procesu tworzenia systemu plików. Jeśli jednak nie
zainicjujemy w pełni systemu plików podczas jego tworzenia, to system w późniejszym czasie (przy
jego montowaniu) dokończy ten pominięty przez nas proces, co będzie objawiać się aktywnością
nośnika w czasie spoczynku.

Jeśli zatem tworzymy system plików EXT4 z domyślnymi opcjami, to zarówno opcja `lazy_itable_init`
jak i `lazy_journal_init` są ustawione na `1` i później proces pełnej inicjacji systemu plików
będzie musiał zostać dokończony w tle, co można poznać po aktywności procesów `jbd2` oraz
`ext4lazyinit` . Dlatego też można zatroszczyć się by te dwie opcje ustawić na `0` oraz by pozbyć
się opcji `uninit_bg` i tym samym zainicjować system plików EXT4 w pełni przy jego tworzeniu.

### Flagi packed_meta_blocks, flex_bg i flex_bg_size

Standardowo przy tworzeniu systemu plików EXT4 mamy włączoną flagę `flex_bg` , która odpowiada za
tworzenie elastycznych grup bloków (Flexible Block Groups). W takiej elastycznej grupie, kilka
zwykłych grup bloków jest wiązanych razem w jedną logiczną grupę bloków. W ten sposób miejsce
przeznaczone na bitmapy alokacji bloków/i-węzłów oraz na tablicę i-węzłów jest powiększane, tak by
uwzględnić w nim bitmapy i tablice i-węzłów pozostałych grup bloków, które wchodzą w skład tej
elastycznej grupy.

Standardowo rozmiar elastycznej grupy to `16` (do odczytania w superbloku, `Flex block group size:
16` ) . Grupa 0 będzie zawierać (w kolejności) superblok, deskryptory grupy, bitmapy bloków z
danymi dla grup 0-15, bitmapy i-węzłów dla grup 0-15, tablice i-węzłów dla grup 0-15, a pozostałe
wolne miejsce w grupie 0 będzie przeznaczone na dane plików. W taki sposób można pogrupować i
umieścić obok siebie metadane bloków, co przełoży się na szybsze ich wczytywanie. Ten zabieg ma też
na celu umożliwienie zapisania większych plików w formie ciągłej na dysku.

Biorąc pod uwagę te powyższe informacje, dla naszego przykładowego dysku 2T, na którym mamy zamiar
przechowywać same duże pliki, powinniśmy zwiększyć rozmiar elastycznych grup do górnej granicy,
jaką uda się nam ustawić. Zwykle można spotkać się z wartością `262144` , czyli w skład jednej
elastycznej grupy wejdzie kilkaset tysięcy zwykłych grup, z których każda standardowo opisuje
128MiB danych. Jeśli to przemnożymy przez siebie, to ta wartość pozwoli nam efektywnie uwzględnić
32TiB danych w jednej elastycznej grupie, zatem wszystkie metadane na dysku 2T powinny być w jednym
miejscu, tuż na początku systemu plików.

Teoretycznie ustawienie flagi `flex_bg_size` na `262144` powinno wystarczyć ale w [man mke2fs][21]
można także doszukać się opcji `packed_meta_blocks` , której ustawienie z kolei powoduje, że
bitmapy alokacji bloków/i-węzłów oraz tablice i-węzłów zostaną umieszczone na początku systemu
plików, czyli w zasadzie dokładnie to samo co zwiększenie wartości w parametrze `flex_bg_size` .
Dodatkowo, ustawienie flagi `packed_meta_blocks` sprawi, że dziennik systemu plików (journal)
zostanie umieszczony na początku systemu plików, a nie gdzieś w środku, tak jak to ma miejsce przy
domyślnej konfiguracji systemu plików przy jego tworzeniu.

Te trzy opcje są bardzo nieocenione gdy chcemy przechowywać na dysku duże pliki i ograniczyć stopień
ich fragmentacji do minimum.

### Rozmiar dziennika (journal)

System plików EXT4 standardowo wyposażony jest w dziennik (journal). Ten dziennik ma za zadanie
rejestrować operacje na plikach, tak by podczas awarii zasilania było wiadomo, które operacje
zapisu nie dokonały się w pełni i które pliki system powinien odzyskać. W zasadzie są trzy tryby
pracy dziennika: `writeback` , `ordered` i `journal` . Standardowo wykorzystywany jest `ordered`
jako ten, który jest połączeniem `writeback` i `journal` , z tym, że nie zapewnia ani zalet
`journal` ale za to ma wady `writeback` . W skrócie, dla wydajności zrezygnowano w tym trybie z
zapisu danych plików do dziennika. Zapisywane są jedynie metadane i to w sporej części przypadków
wystarcza by podnieść się po ewentualnym zawale systemu, np. nagłym odcięciu zasilania.

W przypadku, gdy na dysku mamy sporo plików, to naturalnie rozmiar dziennika powinien być nieco
większy aby wpisy w dzienniku nam się nie przekręciły. Jeśli jednak na dysku będziemy mieć kilka
tysięcy dużych plików i głównie je będziemy odczytywać, to zbędny nam jest rozbudowany dziennik
systemu plików.

Podczas zwykłego formatowania partycji, system tworzy dziennik o rozmiarze zależnym od rozmiaru
nowego systemu plików (zwykle około 1GiB). Zatem mniejsze systemy plików mają mniejsze dzienniki,
a większe systemy plików mają większe. Na szczęście jest również ograniczony maksymalny rozmiar
jaki journal może mieć i jest około 40GiB. Na moim 2TB dysku, został stworzony dziennik o rozmiarze,
jeśli dobrze pamiętam, 2GiB. Wiedząc, że te 2GiB by poszło na zmarnowanie, stworzyłem dziennik o
rozmiarze 128MiB przy pomocy opcji `-J size=128` , gdzie liczba określa rozmiar dziennika systemu
plików w megabajtach.

### Ilość zarezerwowanego miejsca na potrzeby root

Jeśli chodzi o partycje niesystemowe, a z taką tutaj mamy do czynienia, to trzeba wiedzieć, że w
ich przypadku nie ma większego sensu rezerwować miejsca pod aktywność użytkownika root. Domyślnie
jednak przy tworzeniu systemu plików EXT4, taka rezerwacja ma miejsce i jest to wycinek partycji o
wartości 5% jej maksymalnej pojemności. Zatem jeśli mamy 1820GiB dysk, to zostałoby zarezerwowane
około 91GiB, z których żaden użytkownik (poza root) nie mógłby skorzystać.

Dobrze zatem jest wyłączyć tę rezerwację, bo przyda nam się te dodatkowe 91GiB pod pliki
użytkownika. Dlatego też nie zapomnijmy dodać flagi `-m 0` do polecenia `mke2fs` , gdzie `0`
określa procent zarezerwowanego miejsca.

## Tablica partycji MS-DOS/GPT

Gdy w grę wchodzi taki duży dysk jak 2TB (i większe), to w zasadzie powinniśmy utworzyć na takim
dysku tablicę partycji GPT. Chodzi tutaj o graniczne możliwości tablicy partycji MS-DOS(MBR), która
obsługuje maksymalnie dyski do 2TiB (2^32*512 bajtów). Przy większych dyskach nie ma już opcji i
trzeba korzystać z tablicy partycji GPT. Co jednak w przypadku, gdy mamy dysk o tej granicznej
pojemności? Generalnie uważam, że tablica MS-DOS powinna już odejść na wysłużony odpoczynek i dla
mnie w zasadzie od lat istnieje jedynie tablica partycji GPT i każdy nośnik pamięci masowej ma
utworzony właśnie ten rodzaj tablicy. Poza tym, GPT dysponuje backup'em tablicy partycji, zatem w
przypadku ewentualnych problemów z nią będziemy mieć większe szanse na odzyskanie danych.

Tablicę partycji możemy utworzyć czy to w graficznym `gparted` , czy też bezpośrednio w konsolowym
`gdisk` w poniższy sposób:

    # gdisk /dev/sdb
    ...

    Command (? for help): o
    This option deletes all partitions and creates a new protective MBR.
    Proceed? (Y/N): y

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): y
    OK; writing new GUID partition table (GPT) to /dev/sdb.
    The operation has completed successfully.

Proces tworzenia nowej tablicy partycji jest iście destruktywny dla danych zgromadzonych na dysku,
dlatego też upewnijmy się, że `gdisk` w swoim argumencie ma odpowiednią ścieżkę do dysku.

## Advanced Format (sektory fizyczne 4096 bajtów)

Znakomita większość dysków HDD obecnie (od 2011 roku) jest produkowania w [technologi Advanced
Format][12], przez co fizyczne sektory na dysku nie są już 512-bajtowe, a 4096-bajtowe, czyli 4KiB,
co z kolei odpowiada klastrowi systemu plików, który standardowo również ma 4KiB. Niemniej jednak,
podczas wchodzenia tej technologi na rynek, pojawiły się obawy związane z narzędziami, które mogły
nie działać prawidłowo albo z nowymi dyskami mającymi 4096-bajtowe sektory, albo ze starymi dyskami,
które miały 512-bajtowe sektory. Dlatego wymyślono [mechanizm mapowania fizycznych sektorów
4096-bajtowych na logiczne sektory 512-bajtowe][2], tak by czasem nie popsuć starszego sprzętu i
zachować przy tym pełną kompatybilność wsteczną.

Niemniej jednak, pojawił się [problem związany z czytaniem/zapisem nie do końca tych sektorów, o
które system prosił][13]. Było to związane z nieprawidłowym wyrównaniem partycji i nieodpowiednim
jej rozmiarem (nie dzielił się przez 8), co mogło skutkować utratą wydajności głównie przy zapisie
danych. Chodzi generalnie o to, że przy złym wyrównaniu partycji, system odczytuje/zapisuje nie
jeden sektor o wielkości 4KiB, a dwa, bo pojedynczy blok systemu plików o wielkości 4KiB leży na
dwóch fizycznych sektorach dysku, z których każdy również ma 4 KiB. Poniżej jest fotka dla lepszego
zrozumienia (zaczerpnięta z tego podlinkowanego wyżej artykułu):

![](/img/2021/07/001-partition-alignment-advanced-format.png#huge)

Takie nieodpowiednie wyrównanie partycji może być bardzo kosztowne i niesamowicie degradować
wydajność dysku twardego. Dlatego też za wszelką cenę trzeba takich sytuacji unikać odpowiednio
wyrównując partycję.

Obecnie jednak praktycznie wszystkie narzędzia dyskowe z automatu są w stanie wyrównać partycję do
1MiB, oraz określić rozmiar podzielny przez 8, tak by dysk z fizycznymi sektorami 4096-bajtowymi
działał optymalnie. Niemniej jednak, warto zweryfikować jakie nasz dysk posiada sektory fizyczne
oraz logiczne, co możemy zrobić odczytując te dwa poniższe pliki.

    # cat /sys/block/sdb/queue/physical_block_size
    4096

    # cat /sys/block/sdb/queue/logical_block_size
    512

No i jak widać, w przypadku tego mojego dysku, mamy do czynienia z 4096-bajtowymi sektorami
fizycznymi i są one mapowane na 512-bajtowe sektory logiczne. Zatem trzeba odpowiednio utworzyć
partycję na tym dysku.

Niby firmware dysku powinien zwracać informację o ewentualnym offsecie, który narzędzia
partycjonujące powinny uwzględnić podczas tworzenia nowych partycji ale u mnie albo firmware dysku
nic nie zwraca, albo nie trzeba nic dodatkowo robić:

    # cat /sys/block/sdb/alignment_offset
    0

Czasami też mogą nam się trafić dyski, które błędnie/mylnie zwracają rozmiar sektora fizycznego.
Sam mam dysk, który ma fizyczne sektory w rozmiarze 4096 bajtów ale zwraca 512 bajtów. W niczym to
nie przeszkadza o ile odpowiednio się te partycje z danymi wyrówna.

Technicznie rzecz biorąc, ja zawsze każdemu radzę, by zarówno tablicę partycji jak i same partycje
tworzył w `gparted` . To narzędzie nigdy mnie nie zawiodło i zawsze partycje nim tworzone były
takie jakie powinny zostać utworzone. W przypadku innych narzędzi, np. `parted` , `fdisk`/`gdisk` ,
to różnie bywało.

### Różnica między Advanced Format 512e i Advanced Format 4kn

Warto w tym miejscu wspomnieć, że w przypadku tego powyższego dysku mamy do czynienia z technologią
Advanced Format 512e, jako że fizyczne sektory 4KiB muszą być tłumaczone na sektory logiczne
512-bajtowe. Niemniej jednak, od dłuższego czasu na rynku są dostępne dyski, które mają wsparcie
dla natywnych sektorów 4096 bajtów (tzw. Advanced Format 4kn). W przypadku takich dysków, system
odczytuje i zapisuje nośnik cząstkami danych właśnie o rozmiarze 4KiB i żadne tłumaczenie na
sektory 512-bajtowe nie ma już miejsca. Efektem jest naturalnie poprawa wydajności. Linux'y
wspierają technologię Advanced Format 4kn od już dłuższego czasu i nie sprawia ona na tych
systemach żadnego problemu.

## Ręczne tworzenie partycji

Jeśli nie jesteśmy w stanie skorzystać z graficznego narzędzia `gparted` przy tworzeniu partycji,
to poniżej znajduje się przykład tworzenia partycji dla 1,5TB dysku wraz z krótkim komentarzem.

Standardowo odpalamy `gdisk` i drukujemy aktualną tablicę partycji:

    # gdisk /dev/sdb
    GPT fdisk (gdisk) version 1.0.7

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.

    Command (? for help): p
    Disk /dev/sdb: 2930275055 sectors, 1.4 TiB
    Model: 2115
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): 446AE50C-FDC7-4320-8935-D46E7A29EEA2
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 2930275021
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 2930274988 sectors (1.4 TiB)

    Number  Start (sector)    End (sector)  Size       Code  Name

Jak widać, na dysku nie ma póki co żadnej partycji. Mamy za to informację że partycje zostaną
wyrównane do 1MiB (2048 sektorów 512 bajtowych) oraz, że pierwszy sektor z którego możemy
skorzystać, to 34, a ostatni to 2930275021.

Przy tworzeniu nowej partycji, na kilka pierwszych pytań można odpowiedzieć domyślnie:

    Command (? for help): n
    Partition number (1-128, default 1): 1
    First sector (34-2930275021, default = 2048) or {+-}size{KMGTP}: 2048

Niemniej jednak, dalej mamy zapytanie o ostatni sektor partycji i tutaj już tak łatwo nie będzie,
bo domyślnie zwracana jest wartość 2930275021.

	Last sector (2048-2930275021, default = 2930275021) or {+-}size{KMGTP}:

Trzeba zweryfikować czy aby ta wartość utworzy nam partycję, której rozmiar jest podzielny przez 8.
Możemy to zrobić korzystając z tego wzoru: 2930275021-2048+1=2930272974 . No i jak można
wywnioskować, 2930272974 nie jest podzielne przez 8. Musimy zatem określić inną wartość, poniżej
przykład obliczeń:

    2930275021/8 = 366284377.625
    366284377×8 = 2930275016

Ta powyżej uzyskana wartość, to jest ostatni sektor, który będzie podzielny przez 8. Biorąc pod
uwagę fakt, że partycja rozpoczyna się na granicy 1MiB i wartość 2048 jest podzielna przez 8, to
jeśli ostatni sektor będzie także podzielny przez 8, to rozmiar partycji również będzie podzielny
przez 8. Musimy tylko od tej powyższej wartości odjąć `1` z racji numerowania sektorów od 0. Zatem:
2930275016-1=2930275015 i to tę wartość musimy podać w zapytaniu o ostatni sektor:

	Last sector (2048-2930275021, default = 2930275021) or {+-}size{KMGTP}: 2930275015

I dalej już standardowo:

	Current type is 8300 (Linux filesystem)
	Hex code or GUID (L to show codes, Enter = 8300):
	Changed type of partition to 'Linux filesystem'

Następnie listujemy jeszcze tablicę partycji, by upewnić się co do wpisanych wartości:

    Disk /dev/sdb: 2930275055 sectors, 1.4 TiB
    Model: 2115
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): 446AE50C-FDC7-4320-8935-D46E7A29EEA2
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 2930275021
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 2020 sectors (1010.0 KiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048      2930275015   1.4 TiB     8300  Linux filesystem

Warto zwrócić uwagę na kolumnę `Start (sector)` i `End (sector)` . W kolumnie `Start (sector)`
powinniśmy mieć zawsze wartość 2048, natomiast w `End (sector)` wartość powinna być nieparzysta. W
tym przypadku mamy 2930275015-2048+1, co daje rozmiar 2930272968 sektorów 512 bajtowych, a sama
liczba 2930272968 jest podzielna przez 8, zatem wyrównanie partycji mamy za sobą.

Zapisujemy jeszcze nowy układ partycji:

	Command (? for help): w

	Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
	PARTITIONS!!

	Do you want to proceed? (Y/N): y
	OK; writing new GUID partition table (GPT) to /dev/sdb.
	The operation has completed successfully.

Tak przygotowaną partycję można już sformatować przy pomocy systemu plików EXT4 via `mke2fs` .

### Formatowanie dysku HDD/SSD pod duże pliki

Biorąc pod uwagę informacje zawarte w niniejszym artykule, poniżej znajdują się dwa polecenia,
których celem jest utworzenie odpowiednio zainicjowanego systemu plików EXT4 na sporych rozmiarów
dysku HDD/SSD:

    # mke2fs \
        -t ext4 \
        -m 0 \
        -L bigdata \
        -T largefile4 \
        -J size=128 \
        -O 64bit,has_journal,extents,huge_file,flex_bg,metadata_csum,dir_nlink,extra_isize,sparse_super2,^resize_inode,^uninit_bg \
        -G 262144 \
        -E lazy_itable_init=0,lazy_journal_init=0,num_backup_sb=2,packed_meta_blocks=1 \
        /dev/sdb1

Niżej zaś znajduje się wersja z włączoną opcją `bigalloc` , gdzie rozmiar klastra został ustawiony
na 4MiB:

    # mke2fs \
        -t ext4 \
        -m 0 \
        -L bigdata \
        -T largefile4 \
        -J size=128 \
        -O 64bit,has_journal,extents,huge_file,flex_bg,metadata_csum,dir_nlink,extra_isize,sparse_super2,bigalloc,^resize_inode,^uninit_bg \
        -C 4M \
        -G 262144 \
        -E lazy_itable_init=0,lazy_journal_init=0,num_backup_sb=2,packed_meta_blocks=1 \
        /dev/sdb1

#### Różnica w strukturze metadanych systemu plików

Gdybyśmy formatowali dysk z wykorzystaniem standardowych opcji systemu plików dla przykładowego
1,5TB dysku, to mielibyśmy 11178 grup bloków, z 8192 i-węzłami, z których każdy opisuje 16 KiB
danych umieszczonych na nośniku. Łącznie to daje:
(((16×1024)×8192)×11178)/1024/1024/1024=1397.25GiB. Jeśli byśmy zmienili nieco ustawienia, by
pojedynczy i-węzeł opisywał 4MiB danych, wtedy będziemy mieli dokładnie tyle samo grup, czyli 11178
ale w każdej z nich będziemy mieli tylko 32 i-węzły do dyspozycji, co jest sporą redukcją z 8192.
Za to jeśli dodatkowo skorzystamy z opcji `bigalloc` , to będziemy mieli tylko 11 grup (z
poprzednich 11178) z 32528 i-węzłami, z których każdy opisuje 4MiB danych. Zatem pojedyncza grupa
opisuje w pierwszym przypadku 128MiB danych, zaś w tym drugi nieco ponad 127GiB. Liczba i-węzłów
zmalała z 91.570.176 do 357.808, zaś ich rozmiar zmniejszył się z 21,83GiB  do 87,35MiB, zatem mamy
niesamowitą redukcję w strukturze metadanych systemu plików, co powinno upłynnić operacje
wykonywane na plikach przechowywanych w obrębie tego systemu plików.

### Domyślne opcje przy tworzeniu systemu plików

Jeśli nie chce nam się tych powyższych opcji (albo części z nich) wpisywać za każdym razem, gdy
tylko będziemy chcieli utworzyć nowy system plików EXT4, to warto wiedzieć, że istnieje plik
`/etc/mke2fs.conf` , w którym to możemy określić domyślne opcje dla nowo tworzonych systemów plików.
Poniżej znajduje się standardowa zawartość tego pliku:

    $ cat /etc/mke2fs.conf
    [defaults]
            base_features = sparse_super,large_file,filetype,resize_inode,dir_index,ext_attr
            default_mntopts = acl,user_xattr
            enable_periodic_fsck = 0
            blocksize = 4096
            inode_size = 256
            inode_ratio = 16384

    [fs_types]
            ext3 = {
                    features = has_journal
            }
            ext4 = {
                    features = has_journal,extent,huge_file,flex_bg,metadata_csum,64bit,dir_nlink,extra_isize
                    inode_size = 256
            }
            small = {
                    blocksize = 1024
                    inode_size = 128
                    inode_ratio = 4096
            }
            floppy = {
                    blocksize = 1024
                    inode_size = 128
                    inode_ratio = 8192
            }
            big = {
                    inode_ratio = 32768
            }
            huge = {
                    inode_ratio = 65536
            }
            news = {
                    inode_ratio = 4096
            }
            largefile = {
                    inode_ratio = 1048576
                    blocksize = -1
            }
            largefile4 = {
                    inode_ratio = 4194304
                    blocksize = -1
            }
            hurd = {
                 blocksize = 4096
                 inode_size = 128
            }

Można naturalnie przerobić zwrotkę z `ext4` i dodać tam stosowne opcje ale też można utworzyć
całkowicie nową zwrotkę i to w niej określić wszystkie interesujące nas opcje, przykładowo:

    bigdata = {
        errors = remount-ro
        features = has_journal,extent,huge_file,flex_bg,metadata_csum,64bit,dir_nlink,extra_isize,sparse_super2,^uninit_bg,^resize_inode,
        inode_size = 256
        inode_ratio = 4194304
        hash_alg = half_md4
        reserved_ratio = 0.0
        num_backup_sb = 2
        packed_meta_blocks = 1
        lazy_itable_init = 0
        lazy_journal_init = 0
        flex_bg_size = 262144
    }

Lub też z opcją `bigalloc` :

    bigdata = {
        errors = remount-ro
        features = has_journal,extent,huge_file,flex_bg,metadata_csum,64bit,dir_nlink,extra_isize,bigalloc,^uninit_bg,^resize_inode,sparse_super2
        inode_size = 256
        inode_ratio = 4194304
        cluster_size = 4194304
        hash_alg = half_md4
        reserved_ratio = 0.0
        num_backup_sb = 2
        packed_meta_blocks = 1
        lazy_itable_init = 0
        lazy_journal_init = 0
        flex_bg_size = 262144
    }

By teraz utworzyć nowy system plików z wykorzystaniem tych opcji, trzeba podać `mke2fs` nazwę
zwrotki (przy pomocy `-T bigdata` ). W ten sposób wszystkie te powyższe opcje zostaną uwzględnione
przy tworzeniu nowego systemu plików, przykładowo:

    # mke2fs -t ext4 -T bigdata -L bigdata /dev/sdb1

Jedyny problem jaki jest z tym powyższym rozwiązaniem, to taki, że nie udało mi się wskazać
rozmiaru dla dziennika systemu plików, czy innych opcji dotyczących samego dziennika (tych, które
się określa via `-J` ). Zatem wygląda na to, że jeśli chcemy określić, np. inny rozmiar dziennika,
to trzeba i tak będzie podawać ten parametr `-J` w `mke2fs` .

#### Przykładowe zwrotki dla systemu plików z bigalloc

Po rozmowach na mailing list kernel-ext4, Theodore Ts'o podesłał mi [dwie konfiguracje systemu
plików][20]. Różnią się one od tej mojej powyższej ale z informacji zawartych w tym podlinkowanym
poście wynika, że te konfiguracje są najlepiej przetestowane pod kątem formatowania nośnika pod
duże pliki. Postanowiłem je zatem uwzględnić te zwrotki poniżej, tak by się nigdzie nie
zawieruszyły, gdyby ktoś kiedyś potrzebował w późniejszym czasie taki system plików dla dużych
plików utworzyć.

Tu jest pierwsza konfiguracja:

    hugefiles = {
        features = extent,huge_file,flex_bg,uninit_bg,dir_nlink,extra_isize,^resize_inode,sparse_super2
        hash_alg = half_md4
        reserved_ratio = 0.0
        num_backup_sb = 0
        packed_meta_blocks = 1
        make_hugefiles = 1
        inode_ratio = 4194304
        hugefiles_dir = /storage
        hugefiles_name = chunk-
        hugefiles_digits = 5
        hugefiles_size = 4G
        hugefiles_align = 256M
        hugefiles_align_disk = true
        zero_hugefiles = false
        flex_bg_size = 262144
    }

A tu druga:

    hugefile = {
        features = extent,huge_file,bigalloc,flex_bg,uninit_bg,dir_nlink,extra_isize,^resize_inode,sparse_super2
        cluster_size = 32768
        hash_alg = half_md4
        reserved_ratio = 0.0
        num_backup_sb = 0
        packed_meta_blocks = 1
        make_hugefiles = 1
        inode_ratio = 4194304
        hugefiles_dir = /storage
        hugefiles_name = huge-file
        hugefiles_digits = 0
        hugefiles_size = 0
        hugefiles_align = 256M
        hugefiles_align_disk = true
        num_hugefiles = 1
        zero_hugefiles = false
    }

### Montowanie systemu plików

Ostatnia sprawa tyczy się montowania systemu plików w systemie. Nie ma tutaj zbytnio jakiejś
filozofii i w zasadzie wszystko czego nam potrzeba to jeden wpis w pliku `/etc/fstab` :

    UUID=59493a0a-ecb7-11eb-b7a5-0021ccc305b0 /media/bigdata ext4    defaults,nodev,nosuid,noexec,lazytime,errors=remount-ro 0 2

Dysk naturalnie dopasowujemy po UUID i montujemy go w wybranym katalogu, tutaj jest to folder
`/media/bigdata/` . Co do samych opcji, to `defaults` [odpowiada za ustawienie tych poniższych
opcji][14]:

 - `rw`     -- montuje system plików w trybie do zapisu.
 - `suid`   -- zezwala na operacje bitów `suid` i `sgid` .
 - `dev`    -- interpretuje urządzenia znakowe/blokowe obecne na tym systemie plików.
 - `exec`   -- zezwala na wykonywanie binarek.
 - `auto`   -- odpowiada za montowanie systemu plików podczas startu systemu lub gdy się wyda
               polecenie `mount -a` .
 - `nouser` -- tylko root może zamontować ten system plików.
 - `async`  -- operacje I/O powinny być wykonywane asynchronicznie.

O ile w przypadku partycji systemowej, te powyższe opcje są iście użyteczne, to w przypadku
partycji typu storage, część z tych flag przydałoby się usunąć. Dlatego też dodaliśmy w `/etc/fstab`
te poniższe opcje:

 - `nodev`  -- traktuje urządzenia znakowe/blokowe jako zwykłe pliki.
 - `nosuid` -- nie zezwala na operacje bitów `suid` i `sgid` .
 - `noexec` -- nie zezwala na wykonywanie binarek.

Dodatkowo została określona flaga `lazytime` , której zadaniem jest aktualizacja czasów (`atime` ,
`mtime` oraz `ctime` ) tylko na obecnej w pamięci RAM wersji i-węzła danego pliku. Montowanie
systemu plików z tą opcją drastycznie redukuje zapisy do tablicy i-węzłów. Oczywiście, każdemu
raczej zależy, by te wyżej wymienione czasy były aktualizowane również i na dysku twardym i jak
najbardziej będą ale tylko w tych określonych niżej przypadkach:

 - i-węzeł musi zostać zaktualizowany z jakiegoś innego powodu niepowiązanego ze znacznikami czasu.
 - aplikacja wywoła `fsync` , `syncfs` albo `sync` .
 - i-węzeł zostanie usunięty z pamięci RAM.
 - upłynie więcej czasu niż 24 godziny od ostatniego zapisania i-węzła na dysk.

Jeśli dysponujemy zasilaczem awaryjnym UPS, lub mamy zamiar podłączyć dysk pod komputer wyposażony
w jakiś rodzaj baterii, np. laptop, to te powyższe warunki nam jak najbardziej powinny wystarczyć.
Jeśli jednak zamierzamy restartować komputer przyciskiem, to lepiej opcji `lazytime` nie ustawiać.

## Parę słów o technologi IntelliPower/IntelliPark

Technologia IntelliPower (zwana też IntelliPark) została wprowadzona do dysków twardych z wielu
powodów i występuje tylko w przypadku talerzowych nośników HDD. Jej celem jest zaparkowanie głowicy
magnetycznej w sytuacji, gdy firmware dysku uzna, że system operacyjny nie ma zamiaru korzystać z
dysku. Gdy głowica jest zaparkowana, maleje zużycie energii przez dysk, przez co ten nośnik
wydziela mniej ciepła (o kilka stopni, zwykle 4-5°C). Gdy głowica jest zaparkowana, to również nie
może uderzyć ona w talerz, np. gdy kopniemy przez przypadek (albo i celowo) w komputer, przez co
sam dysk jest mniej podatny na uszkodzenia mechaniczne. Same plusy. Jest tylko jeden bardzo poważny
minus technologi IntelliPower/IntelliPark, albowiem każdy cykl parkowania głowicy wykańcza
mechanizm odpowiedzialny za takie parkowanie.

Zwykle producenci w notach katalogowych dysków twardych podają żywotność nośnika w liczbie parkowań
głowicy i jest to wartość od 300.000 do 1.000.000, w zależności od modelu dysku. Gdy ta liczba
zostanie osiągnięta, to oczekiwana jest awaria sprzętu i dysk zwykle jest do wyrzucenia. Aktualną
wartość parkowań głowicy można uzyskać z raportu SMART odczytując parametr `Load_Cycle_Count` :

    # smartctl -x /dev/sdb
    ...
      4 Start_Stop_Count        -O--CK   098   098   000    -    2800
    ...
      9 Power_On_Hours          -O--CK   076   076   000    -    17857
    ...
    193 Load_Cycle_Count        -O--CK   200   200   000    -    2629
    ...

Powyżej została też uwzględniona liczba uruchomień dysku wraz z długością jego pracy. W tym
przypadku parametr `Start_Stop_Count` oraz `Load_Cycle_Count` mają bardzo zbliżone wartości. Jest
to zasługą [wyłączenia parkowania głowicy całkowicie w firmware dysku twardego][5] przy pomocy
narzędzia [idle3-tools][6]. Niemniej jednak, to narzędzie obsługuje jedynie dyski Western Digital i
to też nie wszystkie. W przypadku niektórych modeli o większej pojemności, użytkownicy zdają się
raportować, że to narzędzie już nie spełnia swojego zadania. Poza tym, jeśli nie mamy dysku WD, to
też ten cały `idle3-tools` nam się do niczego nie przyda.

Tylko po co za sprawą `idle3-tools` wyłączyć parkowanie głowicy? Chodzi tutaj o to, że producenci
dysków nie dają użytkownikom praktycznie żadnej opcji kontroli mechanizmu parkowania głowicy dysku,
a czas bezczynności, po którym firmware dysku ma zaparkować głowicę, jest ustawiony na 30 sekund
([albo też nawet i na 8 sekund][9]). Krótko mówiąc, jeśli przez 8 sekund do dysku nie powędrowało
żadne zapytanie o zapis/odczyt sektorów, to firmware zaparkuje głowicę. Na linux'ach to parkowanie
głowicy zdaje się mieć miejsce bardzo często -- setki, a nawet tysiące razy w przeciągu
pojedynczego dnia, co po roku takiego zachowania sprawi, że dysk nam się nieuchronnie
rozleci -- akurat jak minie gwarancja.

Do tej pory kupowałem dyski wyłączenie firmy Western Digital, bo w zasadzie przez ostatnią dekadę
nigdy mnie nie zawiodły. W obawie jednak, że w tych pojemniejszych modelach nie będę w stanie
wyłączyć parkowania głowicy, przez co ich żywotność będzie stała pod znakiem zapytania,
postanowiłem odejść od marki WD na rzecz Thosiba. Z testów wychodzi, że dyski Thosiba, przynajmniej
ten model, który mi się trafił, a jest to L200 2TB 2,5", reagują na zarządzanie energią via
`hdparm` . Polecenie, które jest w stanie wysłać do dysku odpowiednie instrukcje co do wydajności
pracy nośnika jest uwzględnione poniżej:

    # hdparm -B 254 /dev/sdb

Wartość `254` działa w przypadku tego dysku. W innych modelach być może trzeba będzie [skorzystać z
innych wartości][7]. Warto też [rzucić okiem tutaj][8].

### Honorowanie ustawień po restarcie systemu

To powyższe ustawienie jest honorowane tylko do wykonania cyklu zasilania dysku. Po restarcie
maszyny lub odłączeniu dysku, trzeba by jeszcze raz korzystać z `hdparm` , by wyłączyć APM
(Advanced Power Management). Niemniej jednak, w każdym linux'ie (również na Raspberry Pi z Kodim)
można napisać regułkę dla UDEV'a, która każdy podłączony dysk potraktuje tym powyższym poleceniem
i jeśli dysk wspiera konfigurację APM, to z powodzeniem będzie można przestawić APM na 254, czyli
w tryb najwyższej wydajności. Wystarczy utworzyć plik `/etc/udev/rules.d/90-hdparm.rules` i dodać
do niego tę poniższą zwrotkę:

    ACTION=="add", SUBSYSTEM=="block", KERNEL=="[sh]d[a-z]", ATTRS{queue/rotational}=="1", \
            ENV{ID_SERIAL}=="TOSHIBA_HDWL120_0000AB123862-0:0", \
            RUN+="/sbin/hdparm -B 254 /dev/%k"

Ta powyższa reguła jest bardzo precyzyjna, za sprawą frazy
`ENV{ID_SERIAL}=="TOSHIBA_HDWL120_0000AB123862-0:0"` , która to określa dopasowanie dysku po jego
numerze seryjnym. Jeśli chcielibyśmy dopasować każdy dysk twardy, to naturalnie tę linijkę trzeba
by z tej powyższej reguły skasować. Ta reguła dotyczy też tylko i wyłącznie dysków talerzowych
( `ATTRS{queue/rotational}=="1"` ), bo dyski SSD nie posiadają głowicy i nie parkują jej.

Jak tylko w systemie zostanie wykryty nowy nośnik spełniający te powyższe kryteria, to system wyda
polecenie `/sbin/hdparm -B 254 /dev/%k"` , gdzie `%k` oznacza numerek dysku, np. `sdb` , co
przestawi APM na 254.

Na koniec trzeba jeszcze przeładować politykę UDEV'a (albo zrestartować system), by zmiany zaczęły
obowiązywać:

    # udevadm control --reload

I możemy podłączyć ponownie dysk do komputera by sprawdzić, czy system nie zwróci nam żadnego błędu
i ustawi parametry dysku zgodnie z naszymi oczekiwaniami. By mieć pewność że tryb oszczędności
energii został wyłączony, możemy podejrzeć raport SMART:

    # smartctl -x /dev/sdb | grep APM
    APM level is:     254 (maximum performance)

Trzeba jednak się liczyć z faktem, że czasem ten powyższy komunikat może być zwracany błędnie i
dysk pomimo ustawienia poprawnej wartości zwyczajnie ją zignoruje i dalej będzie ustawiony w trybie
oszczędzania energii.

### Problem z ustawieniem APM via hdparm

Czasami jednak, `hdparm` przy ustawianiu APM zwróci taki oto błąd:

    # /sbin/hdparm -B 254 /dev/sdb

    /dev/sdb:
     setting Advanced Power Management level to 0xfe (254)
     HDIO_DRIVE_CMD failed: Input/output error
     setting standby to 0 (off)
     APM_level      = not supported

Oznacza to, że nie damy rady skonfigurować dyskowi APM i jeśli parametr `Load_Cycle_Count` w
raporcie SMART nam rośnie w zastraszającym tempie, to trzeba będzie liczyć się z uszkodzeniem dysku
w niedalekiej przyszłości.

Co ciekawe [Western Digital wydało oświadczenie][10], że np. dyski z serii GREEN, są
niekompatybilne z linux'em i jeśli korzystamy z tego systemu, to lepiej dyski WD GREEN omijać
szerokim łukiem. Nie wiem jak sprawa wygląda w przypadku innych dysków tego producenta ale ja u
siebie wszystkie dyski WD traktowałem przy pomocy `idle3-tools` i wyłączałem w nich parkowanie
głowicy zaraz po wyjęcia nośnika z pudełka. Dlatego też lepiej przed zakupem poszukać opinii na
temat modelu dysku, który zamierzamy nabyć, i upewnić się, że nie będzie z nim większych problemów
pod linux.

## Parę słów o technologi CMR vs. SMR

Wielu użytkowników spotkało się z technologiami CMR ([Conventional Magnetic Recording][15]), czyli
konwencjonalnym ułożeniem ścieżek, oraz SMR ([Shingled Magnetic Recording][16]), czyli dachówkowym
ułożeniem ścieżek w kontekście budowy talerzowych dysków twardych. Technologia SMR jest stosowana w
przypadku dużych dysków (2TB+) zwłaszcza, gdy mamy do czynienia z dyskami laptopowymi 2,5". W takim
przypadku dysk 2TB zawsze będzie ze ścieżkami SMR, bo póki co nie da się zapakować do tak małej
obudowy tylu sektorów gwarantujących tak dużą pojemność. Możemy naturalnie zdecydować się na
mniejszy dysk CMR ale czy naprawdę SMR jest taki straszny?

Generalnie to użytkownicy mają z tą technologią problemy po aferze, w którą się uwikłała marka
Western Digital. Chodziło o to, że [nie informowali oni swoich klientów o stosowaniu w dyskach
technologi SMR][17], czego efektem były poważne problemy z wydajnością przy zapisie takich nośników
spinanych w RAID ale czy przeciętny użytkownik komputera ma jakieś powody do obaw z racji
stosowania dysków SMR?

Wypadłoby w tym miejscu zadać sobie pytanie do czego taki dysk nam będzie potrzebny? Jeśli
planujemy przeznaczyć taki dysk HDD pod typowy magazyn danych (w tym przypadku pod storage dla
filmów serwowanych z Raspberry Pi via Kodi), to nie ma dla nas większego znaczenia czy dysk będzie
w technologi CMR czy SMR, z tą różnicą, że praktycznie za tę samą cenę będziemy mieli większej
pojemności nośnik, a na tym chyba najbardziej nam zależy?

Po wielu testach przy wgrywaniu danych na mój nowo zakupiony dysk SMR, tylko parę razy transfer
danych spadł mi w okolice 20-30MiB/s, z tym, że nie do końca jestem pewien dlaczego. Być może był
też winny dysk systemowy, z którego dane były kopiowane, bo jakby nie patrzeć, ten dysk (również
talerzowy) jest wykorzystywany przez setki procesów systemowych i mogą one znacznie spowolnić
transfer danych. Poza tymi nielicznymi sytuacjami, transfer przy kopiowaniu ponad 1,5TiB danych na
ten nowy dysk wahał się w graniach 80-120MiB/s, także jakichś większych problemów z tym dyskiem
póki co nie miałem. Naturalnie przy odczycie rzędu kilku MiB/s na potrzeby oglądania filmów raczej
problemów z wydajnością nie uświadczę i myślę, że akurat w moim zastosowaniu, taki dysk SMR będzie
działał jak każdy inny dysk CMR.

Jeśi jednak planujemy bawić się w RAID albo bardzo dużo danych zapisywać na dysku, to naturalnie
powinniśmy wybrać dwa mniejsze dyski CMR. Po tej całej aferze, [producenci dysków stosownie
oznaczają modele SMR][18] i obecnie nie powinno być problemów z wybraniem dysku o odpowiedniej dla
nas technologi ułożenia ścieżek.


[1]: https://www.kernel.org/doc/html/latest/filesystems/ext4/dynamic.html#inode-size
[2]: https://web.archive.org/web/20210724121358/https://www.seagate.com/pl/pl/tech-insights/advanced-format-4k-sector-hard-drives-master-ti/
[3]: https://forum.dug.net.pl/viewtopic.php?pid=330424
[4]: https://www.thomas-krenn.com/en/wiki/Ext4_Filesystem
[5]: https://morfikov.github.io/post/parkowanie-glowicy-w-dyskach-wstern-digital/
[6]: http://idle3-tools.sourceforge.net/
[7]: https://ata.wiki.kernel.org/index.php/Known_issues#Drives_which_perform_frequent_head_unloads_under_Linux
[8]: https://wiki.ubuntu.com/DanielHahler/Bug59695#Workaround
[9]: https://wiki.archlinux.org/title/hdparm#Power_management_for_Western_Digital_Green_drives
[10]: https://web.archive.org/web/20190129000226/http://wdc.custhelp.com/app/answers/detail/a_id/5357
[11]: https://ext4.wiki.kernel.org/index.php/Bigalloc
[12]: https://en.wikipedia.org/wiki/Advanced_Format
[13]: https://web.archive.org/web/20210724125752/https%3A%2F%2Fwww.thomas-krenn.com%2Fen%2Fwiki%2FPartition_Alignment_detailed_explanation
[14]: https://wiki.debian.org/fstab#Field_definitions
[15]: https://en.wikipedia.org/wiki/Perpendicular_recording
[16]: https://en.wikipedia.org/wiki/Shingled_magnetic_recording
[17]: https://www.benchmark.pl/aktualnosci/western-digital-publikuje-liste-slabszych-dyskow-hdd-z-zapisem-sm.html
[18]: https://blog.westerndigital.com/wd-red-nas-drives/
[19]: https://www.spinics.net/lists/linux-ext4/msg78659.html
[20]: https://www.spinics.net/lists/linux-ext4/msg78709.html
[21]: https://man7.org/linux/man-pages/man8/mke2fs.8.html
