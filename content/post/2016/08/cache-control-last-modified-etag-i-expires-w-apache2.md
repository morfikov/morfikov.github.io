---
author: Morfik
categories:
- Linux
date: "2016-08-01T18:50:48Z"
date_gmt: 2016-08-01 16:50:48 +0200
published: true
status: publish
tags:
- apache2
- cache
GHissueID: 571
title: Cache-Control, Last-Modified, ETag i Expires w Apache2
---

Każda przeglądarka internetowa potrafi buforować dane w swoim cache w celach optymalizacji
przeglądanych stron www. Dzięki temu, szereg elementów odwiedzonych już witryn nie musi być
ponownie pobieranych z serwera. Zyskuje na tym nie tylko serwer ale również i sam klient, któremu
strona ładuje się parokrotnie szybciej. Pod ten mechanizm podpadają nie tylko pliki graficzne ale
również style CSS, skrypty JS, a nawet pliki `.html` . Generalnie rzecz ujmując, wszystko co serwer
www jest w stanie przesłać przeglądarce. Problemem może jednak się okazać zbyt krótki/długi okres
ważności cache. Jeśli ten czas jest za krótki, to elementy strony będą niepotrzebnie utylizować
łącze, nie tylko nasze ale również i serwera www. Z kolei, jeśli okres ważności będzie za długi,
to będziemy odwiedzać nieaktualną stronę. Optymalnym rozwiązaniem byłaby taka konfiguracja serwera
www, gdzie dla konkretnych elementów strony sami moglibyśmy ustalić czas ważności cache. Apache2
daje nam taką możliwość przez ustawienie nagłówków HTTP `Cache-Control` , `Expires` , `ETag` oraz
`Last-Modified` .

<!--more-->
## Jak działa cache

Po wejściu na przykładową stronę www, serwer przesyła treść witryny klientowi, którym zwykle jest
przeglądarka internetowa, ewentualnie jakiś serwer proxy. Nagłówki protokołu HTTP są wysyłane zanim
zostanie przesłany faktyczny zasób, który ma być wyświetlony u nas na ekranie. Dopiero po otrzymaniu
tego nagłówka przesyłana jest treść komunikatu, a przeglądarka w oparciu o dane w nagłówku wie co ma
z tym dokumentem zrobić. Tych nagłówków może być kilka czy nawet kilkanaście, w zależności od
konfiguracji serwera www.

Po otrzymaniu dokumentu, przeglądarka zapisuje sobie jego kopię w swoim lokalnym cache. Podobnie ona
postępuje w przypadku wysłanego żądanie do serwera oraz otrzymanej odpowiedzi. Poniżej jest przykład
zapytania i odpowiedzi:

![](/img/2016/08/1.etag-last-modified-header-naglowek-przegladarka.png#huge)

`Request Headers` , to są nagłówki zapytania, które przesłała przeglądarka serwerowi. Po otrzymaniu
ich, serwer wysłał odpowiedź z szeregiem nagłówków, które są widoczne w `Response Headers` . Nas na
dobrą sprawę interesują tylko dwa nagłówki w odpowiedzi: `Last-Modified` oraz `ETag` .

## Nagłówki Last-Modified i ETag

Powyższa strona jest obsługiwana przez serwer Apache2. Ten z kolei domyślnie przesyła dwa
standardowe nagłówki HTTP, które dotyczą konfiguracji cache. Są to `ETag` oraz `Last-Modified` .
Gdybyśmy teraz w przeglądarce załadowali następną stronę w obrębie tej witryny, to praktycznie
wszystkie jej elementy (za wyjątkiem głównego pliku `text/html` ) zostaną wczytane z cache:

![](/img/2016/08/2.przegladarka-cache-zasobow.png#huge)

Nagłówki `Last-Modified` oraz `ETag` to walidatory (validators), które mają za zadanie zweryfikować
aktualność zasobu, czyli nakazać przeglądarce wysłanie do serwera zapytania warunkowego (conditional
request) w odpowiednim momencie. Zapytania warunkowe są wysyłane w oparciu o informacje dotyczące
świeżości zasobów, a te z kolei mogą być dostarczane przez nagłówki `Expires` i `Cache-Control` .
Niemniej jednak, tutaj żaden z tych nagłówków nie został uwzględniony w odpowiedzi serwera. To nie
problem, bo w przypadku braku tych nagłówków, przeglądarka sama może zdecydować jaką wartość mają
przybrać atrybuty świeżości.

Nagłówki `Last-Modified` oraz `ETag` przesyłają przeglądarce ważne informacje na temat stanu danego
zasobu. Nagłówek `Last-Modified` mówi jej, kiedy dokładnie na serwerze został zmodyfikowany dany
plik. Z kolei [nagłówek ETag zawiera sumę kontrolną][1] kilku elementów składowych. Mogą to być:
`INode` (numer i-węzła) , `MTime` (czas modyfikacji) oraz `Size` (rozmiar w bajtach). Domyślnie są
wykorzystywane jedynie te dwa ostatnie. Im więcej elementów zostanie uwzględnionych, tym wolniej
będzie przetwarzane zapytanie ale też tym bardziej akuratny wynik.

Nagłówek `Etag` jest też bardziej wiarygodny niż `Last-Modified` w sytuacjach, gdzie nie przechowuje
się (nie utrzymuje) się dat modyfikacji plików, lub też w przypadku, gdy rozdzielczość zegara HTTP
(1s) jest niewystarczająca. Stwarza on także zagrożenie dla prywatności klientów i [powoduje
problemy][2] przy implementacji mechanizmu równoważącego ruch.

Domyślnie Apache2 robi użytek z obu tych walidatorów i nie generuje w odpowiedziach informacji na
temat świeżości zasobu (brak nagłówków `Expires` i `Cache-Control`). Jeśli zrezygnujemy z obu tych
walidatorów, to przeglądarka [w ogóle nie będzie przechowywać odpowiedzi w cache][3].

Zakładając jednak, że chcemy, aby pewne elementy naszej witryny były buforowane, musimy
skonfigurować co najmniej jeden walidator. Oczywiście można korzystać z obu ale to przyczynia się
do wzrostu obciążenia serwera oraz też nagłówek dostaje kilka dodatkowych bajtów. Którą opcję
wybierzemy, zależy głównie od rodzaju serwisu, który prowadzimy. W przypadku statycznych plików
galerii WordPress'a, raczej wystarczy nam nagłówek `Last-Modified` .

Jeśli zdecydujemy się na usunięcie nagłówka `ETag` , to do konfiguracji Apache2, w pliku
`/etc/apache2/apache2.conf` musimy dodać poniższą zwrotkę:

    <Directory /apache2>
        Header unset ETag
        FileETag None
    </Directory>

### Kod błędu 304

Na tej pierwszej fotce wyżej mamy datę pobrania nagłówka oraz datę ostatniej modyfikacji pliku.
Jeśli te dwie wartości są zbliżone do siebie, to zasób jest traktowany jako świeży i będzie
serwowany bezpośrednio z cache przeglądarki bez odpytywania serwera www. W przypadku, gdy data
modyfikacji odbiega znacząco od daty zapytania, wtedy klient poprosi serwer o walidację tego zasobu,
by upewnić się, że wciąż jest on ważny.

Za każdym razem, gdy zachodzi potrzeba walidacji zasobu, wartości nagłówków `ETag` i `Last-Modified`
są przesyłane przez przeglądarkę do serwera. Serwer może odpowiedzieć na to zapytanie pozytywnie
albo negatywnie. W pierwszym przypadku zostanie zwrócony kod błędu 304 (Not Modified) i plik
zostanie załadowany z cache przeglądarki. Natomiast w drugim przypadku, przeglądarka będzie musiała
wysłać zapytanie GET i pobrać raz jeszcze cały zasób, który stracił ważność. Poniżej jest
zobrazowana ta sytuacja:

![](/img/2016/08/3.etag-last-modified-304-walidacja.png#huge)

Widzimy tutaj dwie dyrektywy: `If-Modified-Since:Mon, 01 Aug 2016 09:53:38 GMT` oraz
`If-None-Match:"717-538ff93078200"` . To są warunki zapytania warunkowego. Jeśli, któryś z nich nie
jest spełniony, to zasób zostanie pobrany ponownie. W tym przypadku `If-Modified-Since` wskazuje tę
samą wartość co nagłówek `Last-Modified` na pierwszej fotce wyżej, a wartość `If-None-Match` jest
dokładnie taka sama co w nagłówku `ETag` . Zatem zasób nie uległ zmianie i nie ma potrzeby przesyłać
go przez sieć.

Tego typu zachowanie generuje jednak ruch. Za każdym razem, gdy zasób musi przejść procedurę
walidacji pakiety latają na linii przeglądarka - serwer. Nie jest to pożądany efekt, bo pociąga to
za sobą dodatkowe opóźnienia. Tę niedogodność można wyeliminować stosując jeden z dwóch dodatkowych
nagłówków: `Cache-Control` lub `Expires` .

## Nagłówek Expires

Nagłówek `Expires` odpowiada za informacje o świeżości zasobu, tj. przez jaki czas klient nie musi
pobierać danego pliku z serwera. Generalnie rzecz ujmując, ten nagłówek nadpisuje ustawienia
przeglądarki. Jeśli go określimy na serwerze, zostanie także ustawiony parametr `max-age` w
nagłówku `Cache-Control` .

Za ustawienie nagłówka `Expires` w Apache2 odpowiada [moduł mod_expires][4]. Jeśli zamierzamy ten
nagłówek włączyć w odpowiedzi serwera, musimy pierw włączyć moduł `mod_expires` w konfiguracji
Apache2:

    # a2enmod expires
    # systemctl restart apache2

Następnie w pliku `/etc/apache2/apache2.conf` (ew. w pliku `.htaccess` ) musimy dodać zwrotkę
podobną do tej poniżej:

    <Directory /apache2>
        <IfModule mod_expires.c>
            ExpiresActive On
            ExpiresDefault M172800
            ExpiresByType text/css A7200
            ExpiresByType image/png M604800
        </IfModule>
    </Directory>

Dyrektywa `ExpiresActive` aktywuje dodawanie nagłówka `Expires` i `Cache-Control` w odpowiedzi
serwera dla plików znajdujących się w katalogu określonym przez dyrektywę [Directory][5].
Oczywiście możemy skorzystać z innych dyrektyw do oznaczenia plików, np. [Files][6],
[FilesMatch][7] czy też [DirectoryMatch][8]. Dalej mamy dyrektywę `ExpiresDefault` , która ustawia
domyślny czas świeżości dla wszystkich plików w tym katalogu. Z kolei dyrektywy `ExpiresByType` są
w stanie ustawić inny czas dla plików o określonych typach MIME.

Wartości liczbowe widoczne wyżej są wyrażone w sekundach. Każda z tych wartości musi być poprzedzona
literką `A` (czas dostępu) lub `M` (czas modyfikacji). W przypadku ustawienia `A` na danym typie
plików, każdy klient będzie miał ustawiony osobny zegar w swoim cache w oparciu o czas dostępu do
zasobu. Z kolei, gdy ustawimy `M` , to wszyscy klienci będą mięć ustawiony taki sam czas świeżości
cache, który tym razem będzie się odnosił do czasu modyfikacji pliku na serwerze. Poniżej jest fotka
obrazująca nagłówek `Expires` :

![](/img/2016/08/4.expires-cache-control-header-naglowek-przegladarka.png#huge)

Widzimy, że nagłówek `Expires` wskazuje konkretną datę w przyszłości: `Mon, 08 Aug 2016 14:56:15
GMT` . Jeśli przeliczyć sekundy, to okaże się, że są one równe tej wartości, która figuruje w
nagłówku `Cache-Control` w `max-age=604800` , czyli równo 7 dni. Przez ten czas, przeglądarka
będzie wczytywać zasoby ze swojego cache.

## Nagłówek Cache-Control

Wszystkie instrukcje zawarte w nagłówkach `Last-Modified` , `Etag` czy `Expires` mogą zostać
nadpisane przez nagłówek `Cache-Control` . Nagłówek ten został wprowadzony w HTTP/1.1 i miał
zaadresować problem z ograniczeniami wynikającymi ze stosowania nagłówka `Expires` . Jeśli chcemy
mieć większą swobodę w dostosowaniu cache, możemy skorzystać z tych dodatkowych opcji, które oferuje
nam nagłówek `Cache-Control` . Niemniej jednak, by z nich skorzystać, musimy ustawić poszczególne
parametry nagłówka przy pomocy dyrektywy `Header` , która jest obsługiwana przez moduł
[mod_headers][9]. Ten moduł musi być włączony w konfiguracji Apache2:

    # a2enmod headers
    # systemctl restart apache2

Dyrektywa `Header` może przeprowadzać szereg akcji na nagłówku. Nas głównie będą interesować te
poniższe:

  - `append` -- dodaje wartości do istniejącego już nagłówka w odpowiedzi serwera.
  - `set` -- tworzy nowy nagłówek i ewentualnie zastępuje inne, które mają tę samą nazwę.
  - `unset` -- usuwa nagłówek o określonej nazwie, o ile ten istnieje.

Generalnie, to do ustawienia nagłówka `Cache-Control` będziemy używać opcji `set` . Jeśli będziemy
korzystać także z nagłówka `Expires` , to zamiast `set` trzeba korzystać z `append` . Podobnie jak w
przypadku nagłówka `Expires` , możemy korzystać z dyrektyw dopasowujących pliki i katalogi.

Opcje używane [w nagłówku Cache-Control są opisane tutaj][10]. Generalnie są dwa rodzaje opcji: te
używane w zapytaniu oraz te używane w odpowiedzi. Jako, że my konfigurujemy serwer Apache2, to nas
interesują nagłówki przesyłane w odpowiedziach:

  - `must-revalidate` -- gdy odpowiedź w cache przestaje być świeża, nie może zostać użyta bez
    uprzedniej walidacji na serwerze.
  - `no-cache` -- odpowiedź z serwera nie może zostać użyta bez uprzedniej pomyślnej walidacji na
    serwerze.
  - `no-store` -- przeglądarka lub proxy nie mogą buforować zasobów.
  - `no-transform` -- proxy [nie może zmienić payload'u wiadomości][11].
  - `public` -- zarówno przeglądarka jak i proxy mogą buforować zasoby.
  - `private` -- tylko przeglądarka może buforować zasoby.
  - `proxy-revalidate` -- ma takie samo znaczenie co `must-revalidate` z tym, że nie odnosi się do
    cache prywatnego.
  - `max-age` -- po tym okresie czasu, odpowiedź z serwera ma przestać być traktowana jako świeża, a
    klient będzie musiał zweryfikować ją w następnym żądaniu.
  - `s-maxage` -- w przypadku cache współdzielonego, wiek określony w tej dyrektywie nadpisze wiek w
    `max-age` lub w nagłówku `Expires` . Dodatkowo, ta dyrektywa implikuje także zachowanie
    określone przez dyrektywę `proxy-revalidate` .

W przypadku, gdy zarówno nagłówek `Expires` i `Cache-Control` są określone w odpowiedzi serwera,
[max-age zawsze jest barny pod uwagę w pierwszej kolejności.][12]

Poniżej jest przykład ustawienia nagłówka `Cache-Control` . Ten kawałek kodu trzeba dopisać do pliku
`/etc/apache2/apache2.conf` :

    <FilesMatch ".(flv|ico|pdf|avi|mov|ppt|doc|mp3|wmv|wav)$">
        Header set Cache-Control "max-age=29030400, private"
    </FilesMatch>

## Z których nagłówków HTTP korzystać

Na dobrą sprawę to mamy cztery różne nagłówki, które są w stanie skonfigurować cache przeglądarek:
`Last-Modified` , `ETag` , `Expires` oraz `Cache-Control` . Dwa pierwsze to walidatory, a dwa
ostatnie dostarczają informacje o świeżości. Zdania na temat ich wykorzystania są podzielone. Jedni
uważają, że wystarczy korzystać z jednego walidatora i jednego nagłówka zawierającego informacje o
świeżości zasobu. Inni zaś mówią, że najlepiej korzystać ze wszystkich czterech. Z kolei na necie
notuje się przeważnie trzy nagłówki i są to kombinacje `Last-Modified`-`Expires`-`Cache-Control` lub
`ETag`-`Expires`-`Cache-Control` .

Chyba najlepszą opcją są własne eksperymenty. Jeśli potrzebujemy jakiegoś mechanizmu, który nam
zweryfikuje ustawienia nagłówków konkretnych zasobów w serwisie, to zawsze możemy [skorzystać z
redbot][13], który nam wyrzuci przyjazne podsumowanie:

![](/img/2016/08/5.cache-control-expires-last-modified-etag-redbot-test.png#huge)

[Pod tymlinkiem][14] z kolei jest ciekawa rozpiska na temat wykorzystania nagłówków HTTP
odpowiedzialnych za instrukcje dla cache przeglądarek. Jest tam min. ta poniższa fotka:

![](/img/2016/08/http-cache-decision-tree.png#huge)

Widzimy zatem, że są wykorzystane jedynie dwa nagłówki: `Cache-Control` i `ETag` . Z tym, że ten
drugi jest używany jedynie w przypadku, gdy odpowiedź z serwera ma podlegać buforowaniu. Wszystkie
opcje dla cache są dostosowywane przez poszczególne parametry nagłówka `Cache-Control` . Dlaczego
zatem niektóre strony wykorzystują nawet wszystkie cztery nagłówki? Być może to przez kompatybilność
wsteczną dla bardzo starych przeglądarek, które nie wiedzą jak zinterpretować te nowe nagłówki
wprowadzone w wersji 1.1 protokołu HTTP. To jedyne wytłumaczenie, które przychodzi mi do głowy.


[1]: https://httpd.apache.org/docs/2.2/mod/core.html#FileETag
[2]: https://stackoverflow.com/questions/499966/etag-vs-header-expires
[3]: https://www.mnot.net/cache_docs/#VALIDATE
[4]: https://httpd.apache.org/docs/current/mod/mod_expires.html
[5]: https://httpd.apache.org/docs/2.4/mod/core.html#directory
[6]: https://httpd.apache.org/docs/2.4/mod/core.html#files
[7]: https://httpd.apache.org/docs/2.4/mod/core.html#filesmatch
[8]: https://httpd.apache.org/docs/2.4/mod/core.html#directorymatch
[9]: https://httpd.apache.org/docs/current/mod/mod_headers.html
[10]: https://tools.ietf.org/html/rfc7234
[11]: https://stackoverflow.com/questions/20134257/any-reason-not-to-add-cache-control-no-transform-header-to-every-page
[12]: https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers#expires
[13]: https://redbot.org/
[14]: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching
