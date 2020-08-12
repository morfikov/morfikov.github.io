---
author: Morfik
categories:
- Linux
date: "2016-01-21T21:49:23Z"
date_gmt: 2016-01-21 20:49:23 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- pliki/foldery
title: Ukrywanie informacji w plikach (steganografia)
---

Jak możemy wyczytać na wikipedii, [steganografia](https://pl.wikipedia.org/wiki/Steganografia) to
nauka, która ma na celu ukrycie faktu prowadzenia komunikacji. Odróżnia ją to nieco od kryptografii,
gdzie wiadomość jest wprawdzie nieczytelna ale wiadome jest, że dokonywana jest wymiana informacji
między dwoma punktami. W przypadku steganografii możemy ukryć pewną informację, np. w pliku
graficznym maskując tym samym cały proces przekazywania danych. W taki sposób osoba, która nie ma
pojęcia o fakcie ukrycia informacji, zobaczymy jedynie zwykły obrazek. Poniższy wpis ma na celu
sprawdzenie jak skuteczna jest ta metoda i czy nadaje się do zastosowania dla przeciętnego zjadacza
chleba.

<!--more-->
## Steganografia za sprawą steghide

Jedną z bardziej rozpowszechnionych technik ukrywania informacji w plikach są różnego rodzaju
transformacje dokonywane na pliku źródłowym. Chodzi tutaj głównie o podmianę [najmniej znaczących
bitów (LSB)](https://pl.wikipedia.org/wiki/Najmniej_znacz%C4%85cy_bit) , które mogą wprowadzić pewne
zakłócenia ale nie powodują widocznego dla ludzkiego oka uszkodzenia danych zawartych w takim pliku.
W ten sposób, przy nieznacznej utracie jakości plików graficznych czy dźwiękowych, jesteśmy w stanie
przechować jakąś ograniczoną ilość dodatkowych informacji bez zmiany rozmiaru pliku. Kluczowe
znaczenie ma format pliku oraz to jakie dane on zawiera. Plik plikowi pod tym względem nie jest
równy i każdy trzeba traktować indywidualnie. Dlatego właśnie
[steghide](http://steghide.sourceforge.net/) operuje jedynie na plikach `.jpg` , `.bmp` , `.wav` ora
`.au` .

Narzędzie `steghide` jest już nieco leciwe i od parunastu lat nie miało wypuszczonej nowszej wersji.
Niemniej jednak, wciąż działa i realizuje powierzone mu zadania. Jeśli ktoś by szukał alternatyw, to
na [wiki można doszukać się kilku dodatkowych
narzędzi](https://en.wikipedia.org/wiki/Steganography_tools). Zainstalujmy sobie zatem `steghide` .

Spróbujmy osadzić jakąś informację w pliku graficznym. Potrzebne nam będą w sumie dwa pliki. Jeden
graficzny i drugi tekstowy. Plik tekstowy nazwijmy `wiadomosc.txt` , graficzny zaś `obrazek.jpg` .
Do pliku tekstowego wpisujemy przykładową wiadomość: `matrix has you` . By teraz osadzić tę
wiadomość w pliku graficznym, wydajemy to poniższe polecenie:

    $ steghide embed -cf obrazek.jpg -ef wiadomosc.txt
    Enter passphrase:
    Re-Enter passphrase:
    embedding "wiadomosc.txt" in "obrazek.jpg"... done

Proces osadzania informacji wygląda zawsze tak samo. Określamy, że chcemy osadzić jakąś wiadomość
przy pomocy `embed` . Następnie w parametrze `-cf` podajemy plik, który ma robić za przykrywkę, a
opcja `-ef` odpowiada za plik, który chcemy ukryć. Po wydaniu tego powyższego polecenia zostaniemy
poproszeni o hasło, po podaniu którego ta dołączona wiadomość zostanie zaszyfrowana. Hasło jest
opcjonalne.

Jeśli teraz chcielibyśmy wydobyć zawartą w tym obrazie wiadomość, to musimy posłużyć się tym
poniższym poleceniem:

    $ steghide extract -sf obrazek.jpg
    Enter passphrase:
    wrote extracted data to "wiadomosc.txt".
    
    $ cat wiadomosc.txt
    matrix has you

Jak widzimy, wiadomość została wydobyta. Sam tekst jest zaś w formie niezmienionej i możliwy do
odczytania.

## Problemy wynikające ze stosowania steganografii

Wszystko ma swoją cenę i steganografia również. Chodzi generalnie o to, że dane, które chcemy w ten
sposób przemycić muszą być gdzieś przechowywane. W tym przypadku plik graficzny miał nieco ponad 700
KiB, natomiast wiadomość 15 bajtów. Zatem tutaj nie jest to aż tak widoczne. Niemniej jednak, każdy
plik host ma skończoną ilość tych bitów mało ważnych, co przekłada się na ilość dostępnego miejsca,
które możemy wykorzystać. Wartość tej wolnej przestrzeni możemy podejrzeć w następujący sposób:

    $ steghide info obrazek.jpg
    "obrazek.jpg":
      format: jpeg
      capacity: 39.0 KB

Zatem w tym pliku mamy do dyspozycji jedynie 39 KiB. Niemniej jednak wiadomość podlega kompresji.
Zatem jesteśmy w stanie umieścić nieco więcej informacji niż 39 KiB. Mi udało się upchnąć tam prawie
7 MiB tekstu:

    $ steghide info obrazek.jpg
    "obrazek.jpg":
      format: jpeg
      capacity: 39.0 KB
    Try to get information about embedded data ? (y/n) y
    Enter passphrase:
      embedded file "wiadomosc.txt":
        size: 6.7 MB
        encrypted: rijndael-128, cbc
        compressed: yes

Oczywiście, im dany plik host jest większy, tym więcej poufnych informacji jest w stanie przemycić.
Dla przykładu, plik graficzny 10 MiB ma ładowność 645 KiB.

## Steganografia na przykładzie dd i cat

Jako, że `steghide` zdaje się być trochę ograniczony, to zawsze możemy nieco obejść system i
zaimplementować sobie pseudo steganografię w oparciu o `dd` i `cat` . Przy pomocy `cat` jesteśmy w
stanie połączyć ze sobą szereg plików. Załóżmy na chwilę, że mamy dwa zdjęcia (formaty mogą być
dowolne) i chcemy jednym z nich przysłonić to drugie. Możemy to zrobić w poniższy sposób:

    $ cat img2.jpg >> img1.img

W taki sposób fotka `img2.jpg` zostanie doklejona na koniec pliku `img1.png` . Przy odczytywaniu tak
powstałego pliku, tylko `img1.png` będzie widoczny.

By teraz te dwa pliki rozdzielić, tak by znów można było odczytać każdy z nich, musimy posłużyć się
`dd` oraz wymagane jest byśmy znali rozmiary plików źródłowych. W tym przypadku pliki miały
następujące rozmiary (bajty): 90235 oraz 11472062 i to ten większy został doczepiony do tego
mniejszego. Rozdzielmy je zatem:

    $ dd if=./img3.png of=img1.png bs=1 count=90235
    $ dd if=./img3.png of=img2.jpg bs=1 skip=90235

Pierwsze polecenie czyta 90235 bajtów z pliku połączonego i zapisuje je w osobnym pliku. Druga
komenda również czyta połączony plik, z tym, że od bajta 90235 i również zapisuje tak uzyskane dane
w osobnym pliku. Gdyby tych plików było więcej, musielibyśmy zsumować offset'y. Warto też zaznaczyć,
że niekoniecznie musimy w tym przypadku ograniczać się do plików graficznych. Możemy każdy rodzaj
plików połączyć ze sobą.

Problem z tego typu ukrywaniem danych jest oczywisty. Jeśli połączony plik został zrobiony z głową,
to nie wzbudzi raczej on naszego zainteresowania. Jeśli jednak fotka 640x480 będzie zajmować 100
MiB, to raczej ktoś zacznie coś podejrzewać.
