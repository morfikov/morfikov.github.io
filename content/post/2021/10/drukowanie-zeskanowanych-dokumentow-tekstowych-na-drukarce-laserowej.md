---
author: Morfik
categories:
- Linux
date:    2021-10-26 19:57:00 +0200
lastmod: 2021-10-26 19:57:00 +0200
published: true
status: publish
tags:
- debian
- pdf
- drukarka
- imagemagick
- ocr
GHissueID: 577
title: Drukowanie zeskanowanych dokumentów tekstowych na drukarce laserowej
---

Posiadacze monochromatycznych (czarnobiałych) drukarek laserowych zapewne spotkali się z problemem
drukowania kolorowych dokumentów na tego typu urządzeniach. Nie chodzi tutaj o drukowanie obrazków
czy innych elementów graficznych ale o wydrukowanie, np. zeskanowanej książki. Ostatnio przyszło mi
wydrukować dokumentację techniczną dość starego urządzenia. Problem w tym, że nie był to zwykły
kawałek książki, a jedynie jej skany, z których ktoś postanowił zrobić plik PDF . Takiego
dokumentu "tekstowego" za bardzo nie da się wydrukować na drukarce laserowej, przynajmniej nie bez
pchania się w ogromne koszty. Dla przykładu, w miejsce pożółkniętej kartki ze skanu, taka drukarka
wstawi jakiś odcień szarości i w ten sposób zadrukuje tą szarością całą stronę marnując przy tym
niesamowite ilości toneru, za który my będziemy musieli później zapłacić, co czyni cały proces
drukowania zeskanowanych dokumentów bardzo kosztownym. Postanowiłem jednak znaleźć jakiś sposób, by
tę dokumentację wydrukować, choć trzeba było pierw zająć się samymi skanami, tj. doprowadzić je do
stanu, w którym można by mówić o ewentualnym ich wydrukowaniu. Okazało się, że nie jest to jakoś
specjalnie trudne zadanie, zwłaszcza, gdy na linux zaprzęgnie się do tego celu ImageMagick.

<!--more-->
## Problematyczne skany dokumentów

W zasadzie nie byłoby żadnego problemu z drukowaniem zeskanowanych książek, gdyby nie fakt, że
zeskanowana biała kartka nie jest do końca biała. Przyczyny takiego stanu rzeczy mogą być różne,
np. promienie słoneczne mogą z biegiem lat degradować biel papieru. Inną przyczyną może być też
nie najlepsze oświetlenie pomieszczenia w chwili robienia fotki takiej stronie zwykłym aparatem w
telefonie. Poniżej znajduje się przykład takiej białej kartki, która została mi podsunięta do
wydrukowania:

![scan-image-laser-printer-print-orig-pdf](/img/2021/10/001.scan-image-laser-printer-print-orig-pdf.jpg#huge)

No jak widać, ta biała strona nie przypomina za bardzo tego co każdy z nas rozumie pod pojęciem
"biały".

### Grzbiety i inne artefakty

Patrząc na tę powyższą fotkę ze skanu dokumentacji technicznej, możemy po lewej stronie zauważyć
grzbiet tego dokumentu, który również by został wydrukowany. Tego typu elementy trzeba wstępnie
wyciąć. Do tego celu można posłużyć się narzędziem `krop` :

![scan-image-laser-printer-print-krop](/img/2021/10/007.scan-image-laser-printer-print-krop.png#huge)

Każdy zaznaczony element na danej stronie w pliku wejściowym zostanie przekuty w osobną stronę i
zapisany w nowym dokumencie PDF.

W tym przypadku każda strona się nieco różniła i trzeba było przycinać je indywidualnie. W innych
przypadkach być może da radę skorzystać z opcji przycięcia dla wszystkich stron czy też stron
parzystych i nieparzystych. Jeśli jakieś strony ze źródłowego materiału chcemy pominąć, to `krop`
posiada stosowną opcję, by tych stron w wynikowym PDF nie uwzględnić. Podobnie sprawa wygląda w
przypadku ewentualnego obracania stron.

Tak czy inaczej, przy pomocy `krop` możemy wstępie przygotować sobie dokument, który trzeba będzie
naturalnie poddać kolejnym etapom czyszczenia.

## Podział dokumentu PDF na obrazki PNG/JPEG

Cała przykładowa dokumentacja techniczna składa się z około 50 takich stron, których przykład
mieliśmy na fotce wyżej. Część z tych skanów zawiera jedynie sam tekst, a na pozostałych mamy
rysunki. Przy pomocy `pdfimages` (z pakietu `poppler-utils` ) możemy uzyskać nieco informacji na
temat samych obrazków, które zostały umieszczone w dokumencie PDF:

    $ pdfimages -list skan.pdf
    page   num  type   width height color comp bpc  enc interp  object ID x-ppi y-ppi size ratio
    --------------------------------------------------------------------------------------------
       1     0 image     425   583  rgb     3   8  jpeg   no         9  0    52    50 22.1K 3.0%
       2     1 image     661   916  rgb     3   8  jpeg   no        15  0    80    79 35.6K 2.0%
       3     2 image     661   916  rgb     3   8  jpeg   no        20  0    80    79 44.8K 2.5%
       4     3 image     661   916  rgb     3   8  jpeg   no        25  0    80    79 37.3K 2.1%
       5     4 image     661   916  rgb     3   8  jpeg   no        30  0    80    79 57.1K 3.2%
       6     5 image     661   916  rgb     3   8  jpeg   no        35  0    80    79 60.6K 3.4%
       7     6 image     661   916  rgb     3   8  jpeg   no        40  0    80    79 71.1K 4.0%
       8     7 image     661   916  rgb     3   8  jpeg   no        45  0    80    79 77.7K 4.4%
       9     8 image     661   916  rgb     3   8  jpeg   no        50  0    80    79 76.7K 4.3%
      10     9 image     661   916  rgb     3   8  jpeg   no        55  0    80    79 74.9K 4.2%
    ...

Zatem definitywnie można powiedzieć, że mamy do czynienia ze skanem, a nie z obrazkami osadzonymi w
tekście. Mamy tutaj informacje o rozdzielczości obrazków. Wiemy też, że obrazki są kolorowe (RGB) i
zakodowane w stratnym formacie JPEG. Mamy też informacje o PPI oraz rozmiarze obrazków, który nie
napawa optymizmem.

Najlepiej ten dokument PDF rozbić na osobne strony, po czym zapisać każdą z tych stron w formacie
PNG/JPEG. Do tego celu można wykorzystać `pdfimages` lub `pdftocairo` .

    $ pdfimages skan.pdf page -png

    $ pdftocairo skan.pdf -png

### Skala szarości

Wyżej mieliśmy informacje, że obrazki są kolorowe, co było też widać na podglądzie przykładowej
strony. Można by te obrazki przekonwertować z trybu kolorowego do skali szarości ale w przypadku
drukowania na drukarkach monochromatycznych na niewiele nam się to zda, bo w dalszym ciągu szare
elementy materiału źródłowego będą widoczne na wydrukowanej kartce. Jeśli jednak ktoś preferuje
odcienie szarości, to może skorzystać z tego poniższego polecenia:

    $ pdftocairo skan.pdf -png -gray

Te zeskanowane obrazki nie powinny już tak bardzo odstraszać swoim wyglądem:

![scan-image-laser-printer-print-gray-pdf](/img/2021/10/002.scan-image-laser-printer-print-gray-pdf.jpg#huge)

Nie da się ukryć, że tło dokumentu jest teraz bardziej białe ale do idealnej bieli ździebko jeszcze
brakuje, a ta nieidealna biel zostanie na drukarce laserowej wydrukowana. W zasadzie to w kwestii
samego wydruku raczej nic się nie zmieniło.

### Głębia koloru 1-bit (mono)

Można naturalnie spróbować przerobić obrazek przy pomocy [głębi koloru 1-bit][4], tj. dwóch kolorów,
z których jednym będzie kolor czarny (dla tekstu), a drugim biały (dla tła strony). W przypadku
dokumentów, które mają niewielki stosunek szumu tła do treści/tekstu, taki zabieg może dać nawet
dość zadowalające efekty.

By rozdzielić dokument PDF na obrazki i potraktować je przy tym głębią koloru 1-bit, możemy
skorzystać z poniższego polecenia:

    $ pdftocairo skan.pdf -png -mono

Niemniej jednak, w przypadku tego przykładowego skanu dokumentacji, efekt jest dość daleki od
jakiekolwiek formy, która ma cokolwiek wspólnego ze słowem "zadowalający":

![scan-image-laser-printer-print-black-white-pdf](/img/2021/10/003.scan-image-laser-printer-print-black-white-pdf.png#huge)

No jak widać, mamy kartkę pokrytą śnieżnobiałą ... bielą ale najwyraźniej trochę zasp się porobiło
na tych czarnych literkach, przez co przeczytanie takiego tekstu graniczy z cudem.

## Zaprzęgnięcie do pracy ImageMagick

Może i niektóre osoby zadowoliłyby się wynikami uzyskanymi powyżej, zwłaszcza, gdy obrazki się
poprzycina i potraktuje głębią szarości ale mi ciągle brakowało jakiegoś bardziej cywilizowanego
rozwiązania. Z pomocą tutaj przyszedł ImageMagick.

Mając rozbity dokument PDF na pojedynce strony w postaci obrazków PNG/JPEG (kolorowych czy też w
skali szarości), możemy spróbować przerobić te obrazki przy pomocy narzędzia `convert` .

### Wyostrzenie tekstu

Najlepiej zacząć od wyostrzenia obrazka (tekstu) podając w `convert` parametr `-sharpen` ,
przykładowo:

    $ ls ./*.png | xargs -L1 -I {} convert {} -sharpen 0x1 -resize 200% {}-kopia.png

Im wyższą wartość określimy w opcji `-sharpen` , tym tekst może stać się ostrzejszy ale kosztem
pewnych elementów samego tekstu. Większe wartości zdają się mieć lepszy rezultat w przypadku
większego tekstu. Przy pomocy `-resize` możemy dodatkowo powiększyć sobie obrazek, co powinno
pozytywnie wpłynąć na jakość wyostrzenia. Poniżej są przykłady dla `-sharpen` o wartości `0x1` ,
`0x2` , `0x3` oraz `0x4` :

![scan-image-laser-printer-print-convert-sharpen](/img/2021/10/008.scan-image-laser-printer-print-convert-sharpen.png#huge)

### Czyszczenie tła dokumentu

Po opcjonalnym wyostrzeniu tekstu trzeba oczyścić stronę ze zbędnego szumu tła zostawiając tym
samym jedynie sam tekst. Możemy ten proces przeprowadzić przy pomocy tego poniższego polecenia:

    $ ls ./*.png | xargs -L1 -I {} convert {} -quality 100 -density 300 -fill white -fuzz 70% +opaque "#000000" {}-kopia.png

Kluczowe w tym poleceniu jest określenie białego koloru w parametrze `-fill` oraz odpowiednie
dobranie wartości w `-fuzz` . Jeśli `-fuzz` będzie wysoki (100%), to otrzymamy obrazek zbliżony do
oryginalnego, a jeśli niski (0%), to otrzymamy samą biel. Poniżej przykłady dla tej samej
zeskanowanej strony ale potraktowanej `-fuzz 25%` , `-fuzz 50%` oraz `-fuzz 80%` :

|   |   |   |
|---|---|---|
| ![scan-image-laser-printer-print-convert-fuzz](/img/2021/10/004.scan-image-laser-printer-print-convert-fuzz.jpg#small) | ![scan-image-laser-printer-print-convert-fuzz](/img/2021/10/005.scan-image-laser-printer-print-convert-fuzz.jpg#small) | ![scan-image-laser-printer-print-convert-fuzz](/img/2021/10/006.scan-image-laser-printer-print-convert-fuzz.jpg#small) |

No jak widać, w zależności od wartości `-fuzz` , tekst staje się mniej/bardziej czytelny. Im
bardziej tekst będzie czytelny, tym też więcej szumu tła będzie na kartce, który zostanie później
wydrukowany przez drukarkę laserową. Dlatego też trzeba metodą prób i błędów tę wartość `-fuzz`
sobie dobrać.

Można też skorzystać z poniższego polecenia:

    $ ls ./*.png | xargs -L1 -I {} convert {} -quality 100 -density 300 -colorspace rgb -monochrome -median 2 {}-kopia.png

Ma ono zbliżony efekt do tego poprzedniego polecenia, z tą różnicą, że tekst tutaj będzie już w
zasadzie jednolicie czarny. Podobnie jak we wcześniejszym przypadku, również i tutaj trzeba się
pobawić wartością parametru `-median` , by uzyskać jak najlepszy efekt dla danego dokumentu.
Poniżej są przykłady dla `-median` wynoszącego `1` , `2` , `3` oraz `4` :

![scan-image-laser-printer-print-convert-median](/img/2021/10/009.scan-image-laser-printer-print-convert-median.png#huge)

## Skrypt textcleaner

ImageMagick ma też na wyposażeniu [skrypt textcleaner][5], który jest w stanie oczyścić zeskanowane
dokumenty. Ten skrypt nie jest jednak dostępny w standardowym pakiecie `imagemagick` i trzeba go
dociągnąć sobie ręcznie ze strony projektu i wrzucić np. do `/usr/local/bin/` . Poniżej zaś
znajduje się przykładowe wywołanie skryptu `textcleaner` :

    $ textcleaner -g -e normalize -f 80 -o 15 -s 1 in.png out.png

To powyższe polecenie jest w stanie dać bardzo zbliżony wynik do tego, który został uzyskany w
przypadku `convert` . Tutaj też by uzyskać nieco lepsze (lub gorsze) efekty trzeba pobawić się
opcjami `-f` , `-o` oraz `-s` .

## Konwersja PNG/JPEG do PDF

Gdy już się skończymy bawić obrazkami, można je wydrukować, choć drukowanie kilkuset stron ręcznie
jedna po drugiej mija się trochę z celem. Dlatego z takich fotek lepiej zrobić pojedynczy dokument
PDF i dopiero ten plik wydrukować. Pliki PNG/JPEG można bez większego problemu skonwertować do PDF,
wystarczy skorzystać z narzędzia `convert` :

    $ convert *.png file.pdf

## Problemy z convert

Podczas bawienia się narzędziem `convert` natrafiłem na szereg błędów, które uniemożliwiały
przeprowadzenie pewnych procesów konwersji obrazków. Poniżej znajdują się komunikaty tych błędów
wraz z informacjami na temat jak je wyeliminować.

### convert-im6.q16: attempt to perform an operation not allowed by the security policy `PDF'

Najwyraźniej konwersja do PDF może nieść ze sobą [pewne problemy natury bezpieczeństwa][1] i
domyślnie nie będzie ona możliwa, co będzie objawiać się poniższym błędem:

    convert-im6.q16: attempt to perform an operation not allowed by the security policy `PDF'
    @ error/constitute.c/IsCoderAuthorized/421`

Możemy nieco rozluźnić politykę bezpieczeństwa określoną w pliku `/etc/ImageMagick-6/policy.xml`
przez wykomentowanie poniższej linijki:

    <!-- <policy domain="coder" rights="none" pattern="PDF" /> -->

### convert-im6.q16: cache resources exhausted

Do zabawy z obrazkami czasem potrzeba większych zasobów sprzętowych (pamięć, CPU, etc). Polityka
bezpieczeństwa ImageMagick'a domyślnie limituje wykorzystanie tych zasobów w pliku
`/etc/ImageMagick-6/policy.xml` , czego efektem może być pojawienie się komunikatów podobnych do
tych poniżej:

    convert-im6.q16: cache resources exhausted `./skan.pdf'
    @ error/cache.c/OpenPixelCache/4095`

Można nieco zwiększyć wartości w linijkach zawierających frazę `policy domain="resource"` albo też
te linijki czasowo/permanentnie wykomentować:

    <!-- <policy domain="resource" name="memory" value="256MiB"/> -->
    <!-- <policy domain="resource" name="map" value="512MiB"/> -->
    <!-- <policy domain="resource" name="width" value="16KP"/> -->
    <!-- <policy domain="resource" name="height" value="16KP"/> -->
    <!-- <policy domain="resource" name="list-length" value="128"/> -->
    <!-- <policy domain="resource" name="area" value="128MP"/> -->
    <!-- <policy domain="resource" name="disk" value="1GiB"/> -->
    <!-- <policy domain="resource" name="file" value="768"/> -->
    <!-- <policy domain="resource" name="thread" value="4"/> -->
    <!-- <policy domain="resource" name="throttle" value="0"/> -->
    <!-- <policy domain="resource" name="time" value="3600"/> -->

## OCR

W wątku założonym na [forum dug.net.pl][2], kilku użytkowników sugerowało skorzystanie z mechanizmu
OCR (Optical Character Recognition), którego zadaniem jest rozpoznanie i wyciągnięcie tekstu z
zeskanowanych obrazków i zapisanie go w zwyczajnym dokumencie tekstowym. Problem w tym, że te skany
przykładowej dokumentacji, z którymi mamy tutaj do czynienia, są bardzo słabej jakości i OCR
popełnia dość sporo błędów. W efekcie po takim wyciągnięciu tekstu z zeskanowanego obrazka trzeba
by bardzo dokładnie ten tekst przejrzeć i ręcznie poprawić wszystkie błędy. Ten proces jest jednak
żmudny i dość czasochłonny i w mojej opinii przy tak niskiej jakości materiału wejściowego lepszym
rozwiązaniem w stosunku do OCR byłoby ręczne przepisanie tego dokumentu, co powinno przebiec o
wiele sprawniej.

Jeśli jednak ktoś miałby dokumenty w nieco lepszej jakości i chciałby dać szansę OCR, to trzeba by
na linux doinstalować pakiety `tesseract-ocr` i `tesseract-ocr-pol` (dla rozpoznawania języka
polskiego). Później po wstępnej obróbce materiału wejściowego, np. wyostrzeniu tekstu, można by
spróbować rozpoznać tekst przy pomocy poniższego polecenia:

    $ tesseract -l pol --dpi 300 obrazek.png text.txt

## Scantailor oraz Scantailor Advanced

W tym podlinkowanym wyżej poście na forum pojawiły się także propozycje dedykowanych narzędzi do
obróbki skanowanych dokumentów tekstowych. Jednym z nich był [Scantailor][3], który jest już
martwym projektem i wyleciał z Debiana za sprawą zależności QT4. Kolejnym poleconym projektem był
[Scantailor Advanced][6], który jest forkiem tego poprzedniego projektu. Może i Scantailor Advanced
ma interfejs na bazie QT5, to wygląda na to, że sam projekt również jest od dłuższego czasu (około
2 lat) martwy i raczej nie ma co z nim wiązać jakichś większych nadziei. Można naturalnie ręcznie
sobie zbudować źródła tej aplikacji ([instrukcje tutaj][7]) i przy jej pomocy oczyścić skany
dokumentów ale w moim przypadku `convert` poradził sobie na tyle znośnie, że odpuściłem sobie
Scantailor'a zupełnie. Być może w przyszłości się pobawię tym oprogramowaniem.

## Podsumowanie

Drukowanie zeskanowanych dokumentów tekstowych może być ździebko problematyczne, zwłaszcza w
sytuacjach gdzie zależy nam na ograniczeniu zużycia zasobów tonera w drukarkach laserowych. Można
za to przy lekkim wysiłku podnieść jakość samego tekstu czyniąc go nieco bardziej czytelnym
przy jednoczesnym usunięciu kolorowego tła strony, co powinno ograniczyć dość znacznie ilość tonera
potrzebnego do wydrukowania pojedynczej kartki. Problem w tym, że nie ma w zasadzie dwóch takich
samych zeskanowanych dokumentów. Nawet pojedyncze strony w takim dokumencie mogą się bardzo różnić
pod względem jakości samego tekstu. Dlatego w przypadku tego tupu zeskanowanych dokumentów zawsze
trzeba będzie trochę poeksperymentować z wartościami różnych parametrów, co może nam dać nieco
lepszy (lub też i gorszy) wynik.


[1]: https://askubuntu.com/questions/1081895/trouble-with-batch-conversion-of-png-to-pdf-using-convert
[2]: https://forum.dug.net.pl/viewtopic.php?pid=331773
[3]: https://github.com/scantailor/scantailor
[4]: https://en.wikipedia.org/wiki/Color_depth#1-bit_color
[5]: http://www.fmwconcepts.com/imagemagick/textcleaner/
[6]: https://github.com/4lex4/scantailor-advanced
[7]: https://github.com/4lex4/scantailor-libs-build
