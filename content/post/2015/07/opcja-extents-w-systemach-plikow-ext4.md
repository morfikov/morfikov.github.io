---
author: Morfik
categories:
- Linux
date: "2015-07-10T15:20:44Z"
date_gmt: 2015-07-10 13:20:44 +0200
published: true
status: publish
tags:
- system-plików
- hdd
- ssd
- ext4
GHissueID: 151
title: Opcja extents w systemach plików ext4
---

Dziś postanowiłem sprawdzić jak wygląda struktura plików mojego dysku. Chodzi oczywiście o ich
fragmentację. Zgodnie z tym co pokazał mi `fsck` , pofragmentowanych plików jest 350. Po
zapuszczeniu defragmentacji via `e4defrag` ilość tych plików spadła do nieco ponad 100 i jeśli by
się przyjrzeć procesowi defragmentacji, to można było zauważyć linijki mające
`extents: 100 -> 10` . Wychodzi na to, że plik dalej jest w kawałkach i nie idzie go
zdefragmentować. Jak rozumieć taki zapis?

<!--more-->
## Extents, czyli zakresy bloków

[System plików ext4][1] ma opcję `extents` , która odpowiada za przydział pewnej ilości bloków w
formie ciągłej. Chodzi o to, że domyślny blok danych na dysku ma 4 KiB i jeśli wgrywamy plik, który
waży 1 GiB, to zostanie on podzielony na 262,144 jednostki. By odczytać później taki plik, trzeba
będzie się tyle razy odwołać do dysku, a to spowalnia transfer danych, zwłaszcza w dyskach hdd.
Poza tym, ilość metadanych opisująca ten plik jest przeogromna. W przypadku opcji `extents` , system
plików jest w stanie oznaczyć zakres bloków do 128 MiB i jeśli w takim przypadku wgramy ten 1 GiB
plik, to jego odczyt będzie można zrobić w 8 podejściach. Poprawia to wydajność, drastycznie
zmniejsza ilość metadanych i redukuje fragmentację plików.

Cały ten mechanizm zakresów działa mniej więcej w poniższy sposób. Przy tworzeniu systemu plików są
definiowane grupy bloków. Jako, że system plików ext4 ma domyślny rozmiar bloków 4 KiB oraz, że
każdy i-węzeł ma 256 bajtów, daje nam to 16 i-węzłów na blok. Każda grupa bloków ma 8192 i-węzły,
co przekłada się na 512 bloków, które są przeznaczone na same i-węzły (2 MiB danych). Grupa zaś
składa się z 32768 bloków, czyli 128 MiB i to właśnie kryje się pod nazwą extents.

W każdym i-węźle zarezerwowanych jest 60 bajtów na zakresy bloków. Każdy z zakresów ma 12 bajtów, do
tego dochodzi jeszcze 12 bajtowy nagłówek. Zatem każdy i-węzeł jest w stanie obsłużyć 4 zakresy.
Taki zakres można porównać do klastra w systemach plików ntfs. Mamy tam określony początkowy adres
bloku oraz liczbę kolejnych bloków składających się na ten zakres. Zatem jeśli plik by miał 512 MiB,
to bez problemu taki i-węzeł jest go w stanie obsłużyć. Natomiast w przypadku większych plików,
system plików ext4 buduje drzewo zakresowe ([extent tree][2]). System plików ext4 dba dość mocno o
to by pliki były ciągłe ale zdarzają się takie, które mają więcej niż jeden zakres i to te pliki
zwykliśmy nazywać pofragmentowanymi.

## Fragmentacja plików

Każdy system plików posiada statystyki, a w nich znajdują się przeróżne informacje. Nas interesują
głównie dane na temat pofragmentowanych plików, które możemy odczytać posługując się narzędziem
`fsck` :

    # fsck.ext4 -nfv /dev/mapper/kabi
    ...
          111 non-contiguous files (0.0%)
    ...

W tym przypadku jest to 111 pofragmentowanych plików, co oznacza, że pliki nie są ciągłe i mają
kilka zakresów bloków. Z powyższego logu nie dowiemy się jednak ile kawałków mają te pliki. To zaś
możemy ustalić posługując się narzędziem `filefrag` , przykładowo:

    # filefrag -v /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024
    Filesystem type is: ef53
    File size of /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024 is 1073741824 (262144 blocks of 4096 bytes)
     ext:     logical_offset:        physical_offset: length:   expected: flags:
       0:        0..   32767:      34816..     67583:  32768:
       1:    32768..   63487:      67584..     98303:  30720:
       2:    63488..   96255:     100352..    133119:  32768:      98304:
       3:    96256..  126975:     133120..    163839:  30720:
       4:   126976..  159743:     165888..    198655:  32768:     163840:
       5:   159744..  190463:     198656..    229375:  30720:
       6:   190464..  223231:     231424..    264191:  32768:     229376:
       7:   223232..  253951:     264192..    294911:  30720:
       8:   253952..  262143:     296960..    305151:   8192:     294912: last,eof
    /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024: 5 extents found

Powyżej mamy plik 1024MiB, który zdaje się mieć 9 zakresów ale na samym dole w logu widzimy, że jest
ich tylko 5. Tego typu informację należy traktować w taki sposób, że plik ma 9 zakresów bloków ale
tylko 5 fragmentów, tj. zakresów które nie występują jeden po drugim, w efekcie czego jest jakaś
przerwa między nimi i to jest faktyczna fragmentacja w systemie plików ext4.

Połowa z tych zakresów ma 128MiB (32768 bloków), czyli maksimum dla systemu plików ext4, natomiast
pozostałe mają nieco mniej, co oznacza, że istnieje możliwość uzyskania mniejszej fragmentacji. Poza
tym w kolumnie `expected` mamy 4 oczekiwane offsety bloków i jest to `physical_offset` + `length`
poprzedniego zakresu. Idealna sytuacja to ta, w której mamy ciąg zakresów jeden za drugim. Możemy
spróbować zdefragmentować ten plik kilka razy przy pomocy `e4defrag` . Choć jeśli posiadamy dużą
ilości danych na dysku, cały proces może nic nam nie dać. Mi jednak się udało nieco zoptymalizować
ten plik:

    # filefrag -v /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024
    Filesystem type is: ef53
    File size of /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024 is 1073741824 (262144 blocks of 4096 bytes)
     ext:     logical_offset:        physical_offset: length:   expected: flags:
       0:        0..   32767:    1161216..   1193983:  32768:
       1:    32768..   65535:    1193984..   1226751:  32768:
       2:    65536..   98303:    1226752..   1259519:  32768:
       3:    98304..  131071:    1259520..   1292287:  32768:
       4:   131072..  163839:    1292288..   1325055:  32768:
       5:   163840..  196607:    1325056..   1357823:  32768:
       6:   196608..  229375:    1357824..   1390591:  32768:
       7:   229376..  262143:    1390592..   1423359:  32768:             last,eof
    /media/morfik/sdb7-usb-WDC_WD15_EARS-00/1024: 1 extent found

Jest to ten sam plik zdefragmentowany kilka razy. Jak widzimy nie ma już pozycji w kolumnie
`expected` , zaś `length` wszędzie wskazuje na `32768` , czyli 128MiB zakres. Mimo, że ten plik ma 8
zakresów, to nie jest traktowany przez system jako pofragmentowany.

Ten powyższy plik został wgrany na partycję, która praktycznie była pusta i miała około 458 GiB.
Dlatego zakresy są 128 MiB. Jeśli jednak weźmiemy sobie bardziej zapchany system plików, to jego
wolna przestrzeń może się prezentować np. w poniższy sposób:

    # e2freefrag /dev/mapper/kabi
    ...
    HISTOGRAM OF FREE EXTENT SIZES:
    Extent Size Range :  Free extents   Free Blocks  Percent
        4K...    8K-  :          3955          3955    0.06%
        8K...   16K-  :          3495          8194    0.13%
       16K...   32K-  :          2601         13165    0.20%
       32K...   64K-  :          2622         28991    0.45%
       64K...  128K-  :          2565         58267    0.90%
      128K...  256K-  :          1576         71371    1.11%
      256K...  512K-  :          1331        118346    1.83%
      512K... 1024K-  :          1058        190532    2.95%
        1M...    2M-  :          1202        444210    6.89%
        2M...    4M-  :          1211        884489   13.71%
        4M...    8M-  :          1249       1803998   27.97%
        8M...   16M-  :           622       1643226   25.48%
       16M...   32M-  :           198       1024999   15.89%
       32M...   64M-  :            16        163082    2.53%

Na tej partycji jest wolnych jeszcze 25 GiB ale jak widzimy wyżej, w obrębie tego systemu plików nie
mamy wolnej przestrzeni, która by miała więcej niż 32-64 MiB. Zatem jeśli byśmy wgrali plik 1GiB, to
zostanie on podzielony na dość sporą ilość zakresów. Dlatego też warto pamiętać, by nie tworzyć
małych partycji i nie zapychać ich w dość znacznym stopniu, bo to powoduje drastyczną fragmentację
wolnej przestrzeni i pliki zamiast być alokowane w kilku zakresach obok siebie, zaczynają być
oznaczane przez dziesiątki, setki czy nawet tysiące z nich i do tego porozrzucane w obrębie całej
partycji, co znacznie spowalnia operacje na plikach.


[1]: https://en.wikipedia.org/wiki/Ext4
[2]: https://en.wikipedia.org/wiki/HTree
