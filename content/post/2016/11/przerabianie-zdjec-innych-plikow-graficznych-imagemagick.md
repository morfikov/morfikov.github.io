---
author: Morfik
categories:
- Linux
date: "2016-11-06T12:40:11Z"
date_gmt: 2016-11-06 11:40:11 +0100
published: true
status: publish
tags:
- imagemagick
- grafika
GHissueID: 503
title: Przerabianie zdjęć i innych plików graficznych z Imagemagick
---

[Imagemagick to zestaw tekstowych narzędzi](https://www.imagemagick.org/script/index.php) do
manipulacji obrazami graficznymi. Łapie on takie formaty jak DPX, EXR, GIF, JPEG, JPEG-2000, PDF,
PhotoCD, PNG, Postscript, SVG, oraz TIFF. Przy pomocy Imagemagick można przeprowadzać cała masę
operacji na plikach począwszy od tych podstawowych, kończąc na tych najbardziej zaawansowanych. W
sumie to nie wiem czy jest coś czego by nie dało się zrobić przy pomocy tego magika. Czasami
korzystanie z wiersza poleceń jest też o wile wygodniejsze w przypadku przeróbki całej masy zdjęć,
gdzie graficzne narzędzia typu GIMP raczej średnio się nadają. Postanowiłem zatem zrobić wpis o
Imagemagick (pakiet `imagemagick` w dystrybucji Debian) i zawrzeć w nim wszystkie te ciekawsze
polecenia, na które w przyszłości natrafię podczas przerabiania fotek, skrinów czy innych grafik.

<!--more-->
## Dane EXIF w Imagemagick

[Dane EXIF](https://pl.wikipedia.org/wiki/Exchangeable_Image_File_Format) (Exchangeable Image File
Format) to metadane zapisane w plikach, które są zwykle uzupełniane przez aparaty cyfrowe podczas
robienia zdjęć. Przeglądając dane EXIF możemy sprawdzić, np. kiedy i gdzie zdjęcie zostało zrobione,
czy też jakie urządzenie zostało wykorzystane do tego celu. Oczywiście samych informacji jest więcej
ale nie zawsze wiadomo jak takie dane EXIF wyciągnąć ze zdjęcia. Pakiet `imagemagick` dostarcza min.
polecenie `identify` . Podając mu parametr `-verbose` jesteśmy w wstanie wydobyć całą masę
informacji, w tym również i dane EXIF. Poniżej jest przykład:

    $ identify -verbose fotka.jpg' | grep -i exif
        exif:DateTime: 2016:11:06 11:23:00
        exif:DateTimeDigitized: 2016:11:06 11:23:00
        exif:DateTimeOriginal: 2016:11:06 11:23:00
        exif:ExifOffset: 595
        exif:ExposureTime: 6/100
        exif:Flash: 0
        exif:FNumber: 2/1
        exif:FocalLength: 350/100
        exif:GPSAltitude: 97/1
        exif:GPSAltitudeRef: 0
        exif:GPSDateStamp: 2016:11:06
        exif:GPSImgDirection: 14218/100
        exif:GPSImgDirectionRef: M
        exif:GPSInfo: 358
        exif:GPSLatitude: 11/1, 33/1, 111111/10000
        exif:GPSLatitudeRef: N
        exif:GPSLongitude: 22/1, 44/1, 111111/10000
        exif:GPSLongitudeRef: E
        exif:GPSProcessingMethod: 0
        exif:GPSTimeStamp: 10/1, 23/1, 1/1
        exif:ImageLength: 3264
        exif:ImageWidth: 2448
        exif:ISOSpeedRatings: 562
        exif:JPEGInterchangeFormat: 625
        exif:JPEGInterchangeFormatLength: 9902
        exif:LightSource: 255
        exif:Make: TP-LINK
        exif:MeteringMode: 2
        exif:Model: C5
        exif:Orientation: 1
        exif:SubSecTime: 99
        exif:SubSecTimeDigitized: 99
        exif:SubSecTimeOriginal: 99
        exif:WhiteBalance: 0
        Profile-exif: 10533 bytes

Wszystkie te powyżej wypisane informacje można usunąć przy pomocy polecenia `convert` z
przełącznikiem `-strip` :

    $ convert -strip fotka.jpg fotka-bez-exif.jpg

Trzeba brać pod uwagę, że `-strip` usunie także wszystkie profile zdjęć. Jeśli nie jest to z
jakiegoś powodu pożądane lub też nie chcemy usuwać wszystkich danych EXIF, a jedynie dane z GPS, to
zawsze możemy skorzystać z [narzędzia
exiftool](/post/metadane-plikow-graficznych-exif/).

## Horyzontalne i wertykalne łączenie plików

Czasem istnieje potrzeba połączenia kilku plików graficznych w jeden. Manualne wycinanie fotek i
wklejanie ich w GIMP'ie jest raczej mało praktycznym rozwiązaniem. W Imagemagick możemy tę kwestię
sprowadzić do jednego
    polecenia.

    $ convert -background white -gravity center fotka1.png fotka2.png fotka3.png +append fotka-full.png
    $ convert -background white -gravity center fotka1.png fotka2.png fotka3.png -append fotka-full.png

Pierwsze z powyższych poleceń łączy pliki w poziomie, tj. trzy zdjęcia będą obok siebie. Drugie
polecenie zaś łączy pliki w pionie, czyli zdjęcia będą jedno pod drugim. Różnica sprowadza się do
ustawienia `+append` lub `-append` . Opcje `-background` oraz `-gravity` nie są wymagane ale bardzo
przydają się w sytuacji, gdy rozdzielczość plików graficznych jest różna. W takiej sytuacji przy
powyższych ustawieniach zostanie użyte białe tło, a mniejsze obrazki zostaną przyciągnięte do środka
w pliku wynikowym.

## Zmiana prędkości animowanego GIF'a

Plik GIF to seria pojedynczych zdjęć połączonych ze sobą w celu odtworzenia jakiejś prostej
animacji. Te poszczególne klatki w GIF'ie są wyświetlane w pewnych określonych odstępach czasu.
Jeśli mamy GIF'a, którego prędkość chcielibyśmy sobie dostosować (przyśpieszyć lub spowolnić), to
również możemy tego typu zabieg przeprowadzić za pomocą Imagemagick.

Przede wszystkim, musimy ustalić jakie jest aktualne opóźnienie w pojawianiu się następujących po
sobie obrazów. Do tego celu weźmy `identify` :

    $ identify -verbose plik.gif | grep -i delay
      Delay: 20x100
      Delay: 20x100
      Delay: 20x100
      Delay: 20x100
      Delay: 20x100

Ten plik GIF ma 5 klatek, z których każda jest wyświetlana co 0,2 sekundy (20/100). W zależności od
tego czy chcemy przyśpieszyć odtwarzanie tych klatek czy też je spowolnić, wystarczy odpowiednio
ustawić parametr `Delay` , a to z kolei można zrobić via `convert` :

    $ convert -delay 50x100 plik.gif wolny.gif
    $ convert -delay 10x100 plik.gif szybki.gif

Pierwsze z powyższych poleceń spowolni wyświetlanie poszczególnych klatek do 0,5 sekundy, a drugie
zaś przyśpieszy animacje do 0,1 sekundy.

## Format i jakość zdjęcia

Mając do dyspozycji aparat, np. ten w telefonie, często robimy całą masę zdjęć. Takie urządzenia
mają zwykle zdefiniowany format plików wynikowych. Może być to, np. JPEG czy PNG. Ręczna zmiana
formatu kilkuset zdjęć nie jest zbyt przyjemna. Możemy nieco zautomatyzować ten proces przez
zaprzęgnięcie `mogrify` . Poniższe polecenie przerobi wszystkie pliki `.png` na `.jpg` , przy czym
zmiana nie polega jedynie na przepisaniu rozszerzenia pliku:

    $ mogrify -format jpg *.png

Szereg plików graficznych umożliwia nam dostosowanie jakości zdjęcia, np. użyte wyżej JPEG/PNG.
Jeśli chcemy dostosować tę jakość przy zmianie formatu, to możemy dorzucić przełącznik `-quality` i
podać wartość od 0-100%:

    $ mogrify -quality 50 -format jpg *.png

Naturalnie jakość zdjęcia można dostosować również bez zmiany jego formatu:

    $ mogrify -quality 5 *.png

Przy zmianie tylko jakości zdjęcia, kluczową rolę odgrywa jego format. [Dostępne opcje zmiany
jakości są opisane tutaj](https://www.imagemagick.org/script/command-line-options.php#quality).

## Rozdzielczość zdjęcia

Obecnie nawet aparaty w smartfonach potrafią robić zdjęcia w rozdzielczości 8 mpix czy nawet 13
mpix. Nie zawsze jednak potrzebne nam są tak szczegółowe fotki. Jeśli natomiast chcielibyśmy zmienić
rozdzielczość plików, by ich rozmiar był nieco mniejszy, to możemy posłużyć się jednym z poniższych
poleceń:

    $ mogrify -resize 75% *.png
    $ mogrify -path 75/ -resize 75% *.png
    $ mogrify -path 1024x768/ -resize 1024x768 *.png
    $ mogrify -path 1024x768/ -resize

gt;1024x768 \*.png

Jak widać mamy kilka opcji. Przede wszystkim, możemy zmienić rozdzielczość fotki w oparciu o
procenty lub sztywne wymiary przy pomocy parametru `-resize`. Możemy także zmienić jedynie te fotki,
które wykraczają poza pewną akceptowaną dla nas wielkość. W tym celu trzeba poprzedzić rozmiar
wyeskejpowanym znaczkiem `<` lub `>` . Jeśli natomiast chcemy zachować oryginalne zdjęcia, to
przeskalowane fotografie można zapisać w osobnym katalogu podając opcję `-path` (katalog musi
istnieć). Warto tutaj też nadmienić, że tak przerobione fotki zachowają oryginalne proporcje
zdjęcia. Dla przykładu, mając zdjęcie w rozdzielczości 2164x1184, nie zostanie ono przycięte do
1024x768. Zamiast tego uzyskamy rozdziałkę 1024x560.
