---
author: Morfik
categories:
- Linux
date: "2016-01-21T16:58:55Z"
date_gmt: 2016-01-21 15:58:55 +0100
published: true
status: publish
tags:
- pliki
- foldery
- prywatność
- gps
- exif
- grafika
- firefox
GHissueID: 537
title: Metadane plików graficznych (EXIF)
---

Każdy plik posiada szereg opisujących go atrybutów. Możemy się o tym przekonać wykorzystując
narzędzia `ls` lub `stat` . W ich przypadku zostaną nam zwrócone takie informacje jak rozmiar
pliku, data modyfikacji czy też prawa dostępu. To właśnie są metadane opisujące pliki w obszarze
systemu plików i są one wymagane, by system operacyjny działał prawidłowo. To jednak nie jedyne
metadane, z którymi spotykamy się na co dzień. Najlepszym przykładem są zdjęcia czy filmy robione
smartfonami czy też aparatami lub kamerami cyfrowymi. Każdy plik stworzony za pomocą tych urządzeń
zawiera w sobie bardzo rozbudowane informacje, które nie zawsze chcielibyśmy udostępniać. W tym
wpisie skupimy się głównie na [danych
EXIF](https://pl.wikipedia.org/wiki/Exchangeable_Image_File_Format) zawartych w plikach graficznych,
które postaramy się wydobyć, zmienić i usunąć.

<!--more-->
## Wydobywanie informacji EXIF

Jeśli mamy dostęp do telefonu z wbudowanym aparatem, czy też do tych nieco bardziej zaawansowanych
cud techniki kosztujących dziesiątki tysięcy dolców, to możemy zrobić sobie przykładowe zdjęcie.
Posłuży nam ono jako obiekt doświadczalny. Będziemy także potrzebować szeregu narzędzi, dzięki
którym będziemy w stanie zarządzać metadanymi EXIF. Dokładną rozpiskę programów, które są w stanie
dobrać się do metadanych plików, [można znaleźć na
wiki](https://en.wikipedia.org/wiki/Comparison_of_metadata_editors). W dystrybucji Debian możemy
wykorzystać pakiety `imagemagick` czy też `libimage-exiftool-perl` .

### Dane EXIF w exiftool

W pakiecie `libimage-exiftool-perl` mamy zawarte narzędzie
[exiftool](http://owl.phy.queensu.ca/~phil/exiftool/), które jest w stanie nam wypisać szereg
informacji opisujących dane zdjęcie. Sprawdźmy zatem co to narzędzie nam powie o naszej przykładowej
fotce:

    $ exiftool DSC00026.JPG
    ...
    Camera Model Name               : NIKON D4
    ...
    Create Date                     : 2015:07:20 09:27:58
    ...

## Usuwanie informacji EXIF

Analizując metadane EXIF, możemy dojść do wniosku, że jest ich tam dość sporo i w pewnych sytuacjach
mogą zagrażać naszej prywatności. Chodzi głównie o wysyłanie prywatnych zdjęć na wszystkie te
portale społecznościowe czy różnego rodzaju serwisy, które umożliwiają dzielenie się naszymi
fotkami. Możemy jednak te pliki poddać edycji i usunąć im część danych EXIF, tak by zainteresowane
nami osoby nie poznały, np. naszego miejsca zamieszkania.

Poniżej jest przykład zdjęcia z danymi GPS:

    $ exiftool fotka.jpg | grep -i gps
    GPS Version ID                  : 0.0.2.2
    GPS Latitude Ref                : South
    GPS Longitude Ref               : East
    GPS Altitude Ref                : Above Sea Level
    GPS Altitude                    : 0 m Above Sea Level
    GPS Latitude                    : 33 deg 51' 21.91" S
    GPS Longitude                   : 151 deg 13' 11.73" E
    GPS Position                    : 33 deg 51' 21.91" S, 151 deg 13' 11.73" E

Jeśli chcemy usunąć te dane, to niekoniecznie musimy czyścić wszelkie inne informacje EXIF, które
mogą być czasem bardzo przydatne. By wyczyścić tylko te widoczne powyżej pozycje, wydajemy jedno z
tych poniższych poleceń:

    $ exiftool -gps:all= fotka.jpg
    $ exiftool -geotag=  fotka.jpg

Jeśli natomiast chcemy usunąć wszystkie dane EXIF, to możemy to zrobić w poniższy sposób:

    $ exiftool -all= fotka.jpg

## Przepisywanie metadanych EXIF

Metadane EXIF można nie tylko kasować ale też i zmieniać. Załóżmy na chwilę, że chcemy dorobić inną
lokalizację dla danej fotki w celu oszukania kogoś. Usunięcie danych GPS nie pomoże nam w
przekonaniu tego kogoś, że faktycznie byliśmy gdzieś, gdzie nas nie było. Jeśli jednak zmienimy dane
GPS, tak by wskazywały na lokalizację, o której mówimy, to jest niemal pewne, że ten ktoś nam
uwierzy bez większego problemu.

W przypadku edycji danych GPS trzeba nieco pokombinować, by uwiarygodnić zmianę metadanych. Przede
wszystkim musimy ustalić, gdzie dokładnie zrobiono przykładową fotkę i w tym pomoże nam
[google](https://support.google.com/maps/answer/18539?hl=en). Koordynaty widoczne w informacjach
EXIF trzeba zmieniać na format akceptowany przez mapy google. W tym przypadku mamy `33
deg 51' 21.91" S, 151 deg 13' 11.73" E` , co trzeba przepisać do `-33 51.2191, 151 13.1173` lub
`33°51'21.91"S 151°13'11.73"E` . Odpalamy zatem mapy google i wyszukujemy to miejsce. Po chwili
otrzymujemy:

![](/img/2016/01/1.exif-linux-gps.png#huge)

Jak teraz zmienić te dane, tak by lokalizacja zdjęcia wskazywała np. na Nowy York w US? Odszukujemy
pożądaną lokalizację i uzyskujemy jej współrzędne:

![](/img/2016/01/2.exif-linux-gps-podmiana.png#huge)

Mamy zatem `40.687653, -74.054682` i jest to format DD (Decimal degrees). Musimy to przekonwertować
do postaci DMS (Degrees, minutes, and seconds). Robimy to w następujący sposób. Odrzucamy wszystko
po kropce, co daje nam `40` . Wartość dodatnia oznacza północ ( `N` ), ujemna zaś południe ( `S` ).
Następnie bierzemy pozostałą część liczby ( `0.687653` ) i mnożymy ją przez `60`. Daje nam to
wartość `41.25918` , którą zaokrąglamy do czwartego miejsca po kropce. W taki sposób otrzymujemy
jedną ze współrzędnych: `40 deg 41' 25.92" N` . Podobnie postępujemy z drugą ze współrzędnych. Mamy
zatem `74` , a wartość ujemna oznacza zachód ( `W` ), dodatnia zaś wschód ( `E` ) . Pozostałą cześć
liczby mnożymy przez 60 i otrzymujemy wartość `3.28092` . Zaokrąglamy do 4 miejsca po kropce i
otrzymujemy drugą ze współrzędnych, tj. `74 deg 03' 28.09" W` . Uzupełniamy teraz pola GPS w danych
EXIF:

    $ exiftool \
        -GPSLatitudeRef="North" \
        -GPSLongitudeRef="West" \
        -GPSLatitude="40 deg 41' 25.92\" N" \
        -GPSLongitude="74 deg 03' 28.09\" W" \
        fotka.jpg

        1 image files updated

Wartość `GPSPosition` zostanie automatycznie wyliczona w oparciu o `GPSLatitude` i `GPSLongitude` .
Poniżej zaś znajdują się zmienione dane
    EXIF:

    $ exiftool -GPSLatitudeRef -GPSLongitudeRef -GPSAltitudeRef -GPSAltitude -GPSLatitude -GPSLongitude -GPSPosition fotka.jpg

    GPS Latitude Ref                : North
    GPS Longitude Ref               : West
    GPS Altitude Ref                : Above Sea Level
    GPS Altitude                    : 0 m Above Sea Level
    GPS Latitude                    : 40 deg 41' 25.92" N
    GPS Longitude                   : 74 deg 3' 28.09" W
    GPS Position                    : 40 deg 41' 25.92" N, 74 deg 3' 28.09" W

Powyższy przykład z przepisaniem danych GPS to nie jedyna opcja, którą mamy do dyspozycji.
Praktycznie każdy atrybut EXIF można przepisać. Nazwy tych atrybutów muszą być bez spacji, a
wielkość liter nie ma znaczenia. Więcej informacji na temat medatanych zdjęć można znaleźć w [man
exiftool](http://owl.phy.queensu.ca/~phil/exiftool/exiftool_pod.html).

## Dodatek dla Firefox'a odczytujący dane EXIF

Jeśli interesuje nas jedynie poznanie lokalizacji przeglądanych na necie fotek, to niekoniecznie
musimy zaciągać w tym celu `exiftool` . Możemy zwyczajnie doinstalować jeden z kilku dodatków do
Firefox'a, który umożliwia analizę danych EXIF i automatycznie podaje link do map google, by
sprawdzić wykrytą lokalizację. Poniżej przykład [FxIF](http://christian-eyrich.de/mozilla/fxif/) :

![](/img/2016/01/3.exif-fxif-firefox.png#big)
