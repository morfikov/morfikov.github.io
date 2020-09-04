---
author: Morfik
categories:
- Linux
date: "2016-07-19T19:08:40Z"
date_gmt: 2016-07-19 17:08:40 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
- ssl
- tls
title: HTTP Strict Transport Security (HSTS) w Apache2
---

Ostatnio na niebezpieczniku czytałem [taki oto
post](https://niebezpiecznik.pl/post/podroze-kosztuja/). Historia jak historia, nieco długa ale
mniej więcej w połowie pojawiła się informacja na temat nagłówków HSTS ([HTTP Strict Transport
Security](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)), który jest przesyłany w
zapytaniach HTTP/HTTPS. Postanowiłem nieco się zainteresować tym tematem i zbadać czym są te
nagłówki HSTS i w jaki sposób są one w stanie poprawić bezpieczeństwo protokołów SSL/TLS
wykorzystywanych podczas szyfrowania zawartości stron internetowych.

<!--more-->
## Po co nam HSTS, skoro mamy SSL/TLS

Bardzo dużo osób zapytanych o to czy połączenie za pomocą protokołu SSL/TLS jest bezpieczne
odpowiada, że tak, bo nawet jeśli ktoś ten ruch przechwyci, to przecie w formie nieczytelnej i
niezrozumiałej dla człowieka. Po co nam zatem te całe nagłówki HSTS, skoro nasza komunikacja jest
należycie zabezpieczona? Szereg stron w sieci potrafi serwować zawartość serwisu przy wykorzystaniu
dwóch protokołów w zależności od tego czy klient wpisał w przeglądarce `http://` czy też `https://`
. Nawet jeśli zwykle odwiedzamy taka stronę po `https://` , to wystarczy raz ją odwiedzić po
`http://` (np. przez link wewnętrzny do innej strony w serwisie) i przez sieć lecą otwartym tekstem
hasła, ciasteczka i inne poufne dane. Jeśli teraz ktoś podsłuchiwał ruch, to pozyskał on również i
te newralgiczne informacje.

Rozwiązaniem wyżej opisanego problemu może wydawać się wymuszenie połączenia szyfrowanego. No i na
dobrą sprawę szereg stron internetowych podejmuje tego typu akcję i przy pomocy kodów 301 i 302
jesteśmy przekierowywani na nowy adres. W taki sposób nawet po wpisaniu samego `http://` wylądujemy
na zabezpieczonej stronie `https://` . Innym wyjściem może być umieszczanie linków na stronie, które
mają `https://` zamiast `http://` . Niemniej jednak, gdy ktoś między nami a serwerem (MITM) będzie
nasłuchiwał ruchu HTTP, może wyłowić z niego te bezpieczne przekierowania oraz linki `https://` .
Następnie może usunąć `s` ze wszystkich wyłapanych linków
([sslstrip](https://moxie.org/software/sslstrip/)) i robić za proxy między klientem a serwerem. Z
klientem ruch będzie odbywał się w formie nieszyfrowanej, z serwerem zaś, proxy zestawi połączenie
szyfrowane. Ruch przepływający przez proxy będzie zatem w formacie czystego teksu i ani
przeglądarka, ani serwer nie zgłoszą żadnych problemów przy przeglądaniu serwisu. Dlatego właśnie
powstał [HSTS](https://tools.ietf.org/html/rfc6797).

## Czy HSTS jest pozbawiony wad

Problem z HSTS jest taki, że nie uchroni on nas w sytuacji, gdy po raz pierwszy odwiedzamy jakiś
serwis www. Chodzi o to, że by ten cały mechanizm HSTS działał prawidłowo, serwer musi przekazać
klientowi (przeglądarce) nagłówek z rekordem STS. Dla witryn, które odwiedzamy po raz pierwszy,
przeglądarka jeszcze takiego rekordu nie posiada. Dodatkowo, taki rekord może zostać wzięty pod
uwagę przez przeglądarkę jedynie w przypadku, gdy łączymy się po `https://` . Dopiero wtedy
wszystkie połączenia wykonywane pod adresem serwera będą z automatu przepisane na `https://` przez
przeglądarkę. Trzeba o tym pamiętać, w przeciwnym razie ten mechanizm nie ma większego sensu.

Po tym jak już odwiedzimy serwis po raz pierwszy przez link lub wprowadzając adres z `https://` ,
nasza przeglądarka zanotuje sobie, że ten adres ma być odwiedzany przez pewien określony czas tylko
po `https://` . Jeśli w międzyczasie wystąpi zdarzenie, które będzie próbowało przekierować nas na
`http://` , połączenie takie zostanie przerwane, a my zobaczymy komunikat kontrolny. Nie będziemy
też w stanie ominąć tej blokady, przez "kliknięcie OK albo Anuluj". Krótko pisząc, nie damy rady
odwiedzić witryny.

By sobie jakoś radzić z tą podatnością pierwszego odwiedzenia serwisu, stworzono [projekt
hstspreload](https://hstspreload.org/). Ma on na celu stworzenie listy domen, które mają być
odwiedzane tylko po `https://` (o tym za moment).

## Jak włączyć HSTS w konfiguracji Apache2

Mając dostęp do konfiguracji serwera Apache2, możemy wymusić w nim przesyłanie nagłówka HSTS do
wszystkich klientów, którzy odwiedzają nasz serwis. Najlepiej też przy okazji pokusić się o
[wymuszenie szyfrowania całej zawartości strony
www]({{< baseurl >}}/post/wymuszenie-ssl-tls-przy-pomocy-vhostow-apache2/). Jeśli naprawdę zależy
nam na bezpieczeństwie naszej witryny, to powinniśmy ją zaszyfrować w pełni, a nie jedynie formularz
logowania.

Przede wszystkim, musimy włączyć w Apache2 moduł `headers` . Robimy to wpisując w terminalu poniższe
polecenie:

    # a2enmod headers

Następnie edytujemy plik `/etc/apache2/sites-enabled/default-ssl.conf` i dopisujemy do niego
poniższą linijkę:

    <VirtualHost *:443>
    ...
        Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains"
    ...
    </VirtualHost>

I restartujemy serwer.

Fraza `Header always set Strict-Transport-Security` sprawi, że serwer Apache2 zawsze będzie
przesyłał nagłówek HSTS. Dalej mamy kilka parametrów, które możemy sobie dostosować:

  - `max-age` -- czas (w sekundach), przez który przeglądarka będzie przekierowywać żądania do
    naszego serwisu na `https://` .
  - `includeSubDomains` -- szyfrowane mają być także wszystkie subdomeny.
  - `preload` -- wymagane do uwzględnienia naszej strony na liście hstspreload.

Z opcją `includeSubdomains` trzeba uważać w przypadku, gdy nie wszystkie subdomeny mają być
szyfrowane. Chodzi o to, że skoro przeglądarka będzie wymuszać połączenie po `https://` dla
wszystkich subdomen w naszym serwisie, a kilka z nich nie będzie obsługiwać szyfrowania, to takie
połączenie nie zostanie nigdy zrealizowane.

## Test nagłówka HSTS

Nagłówek możemy przetestować za pomocą narzędzia `curl` lub też za sprawą dodatku do Firefox'a [Live
HTTP headers](https://addons.mozilla.org/pl/firefox/addon/live-http-headers/).

Poniżej przykład `curl` :

![]({{< baseurl >}}/img/2016/07/1.naglowek-hsts-curl.png#big)

Niżej zaś prezentuje się Live HTTP headers:

![]({{< baseurl >}}/img/2016/07/2.naglowek-hsts-firefox-live-http-headers.png#big)

## Dodawanie rekordu do listy HSTS preload

Adres naszego serwisu www możemy bardzo łatwo dopisać do listy HSTS preload, z której korzystają
wszystkie większe przeglądarki (Chrome, Firefox, Opera, Safari, IE 11 oraz Edge). Musimy tylko
spełnić szereg punktów, które są wynotowane na stronie projektu.

Przede wszystkim, musimy dopisać w konfiguracji Apache2 opcję `preload` do nagłówka HSTS. Zatem cały
wpis od tego nagłówka ma mieć poniższą postać:

    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"

Pamiętajmy też, by utworzyć stosowny alias dla naszej domeny z uwzględnieniem `www`. W przeciwnym
wypadku nie damy rady dodać rekordu HSTS do listy. Trzeba też tutaj wyraźnie zaznaczyć, że
certyfikat SSL/TLS, który mamy w posiadaniu, musi być ważny dla obu domen. Alias dla domeny w
konfiguracji Apache2 definiujemy w poniższy sposób:

    <VirtualHost _default_:443>
    ...
        ServerName morfitronik.pl
        ServerAlias www.morfitronik.pl
    ...
    </VirtualHost>

Jeśli zapomnieliśmy o czymś, to przy dodawaniu rekordu zostaniemy o tym powiadomieni. Poniżej jest
przykład:

![]({{< baseurl >}}/img/2016/07/3.hsts-preload-lista-bledy.png#huge)

Wystarczy poprawić wyżej widoczne błędy i spróbować jeszcze raz. Tym razem już nie powinno być
problemów:

![]({{< baseurl >}}/img/2016/07/4.hsts-preload-lista-sprawdzanie-domeny.png#big)

Akceptujemy oba punkty i przesyłamy zgłoszenie:

![]({{< baseurl >}}/img/2016/07/5.hsts-preload-lista-dodawanie-domeny.png#big)

Po chwili powinno zostać one wysłane:

![]({{< baseurl >}}/img/2016/07/6.hsts-preload-lista-domena-dodana.png#big)

Adres naszego serwisu nie zostanie natychmiast dopisany do listy. Zgodnie z widoczną wyżej
informacją, może upłynąć nawet kilka tygodni zanim stosowny rekord zostanie do niej dodany.
