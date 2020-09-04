---
author: Morfik
categories:
- Linux
date: "2016-08-04T11:10:24Z"
date_gmt: 2016-08-04 09:10:24 +0200
published: true
status: publish
tags:
- apache2
- logi
title: 'Apache2: Jak odchudzić nieco plik access.log'
---

Mając serwer www oparty o oprogramowanie Apache2, w pewnym momencie zacznie nas nieco przytłaczać
kwestia logowania do pliku `/var/log/apache2/access.log` wszystkiego co się nawinie. Jak nazwa pliku
sugeruje, znajdują się w nim komunikaty, które serwer generuje ilekroć tylko ktoś odwiedzi nasz
serwis. Każdy zasób przesłany do klienta, np. style CSS czy obrazki, zostanie zalogowany w powyższym
pliku. Generalnie rzecz ujmując, nie musimy logować wszystkich tych informacji, chyba, że ich
faktycznie potrzebujemy. Trzeba jednak wziąć pod uwagę fakt, że w przypadku obciążonych serwerów,
ilość operacji I/O dysku może być znaczna. Dodatkowo, miejsce na dysku za sprawą takiego obszernego
logu może bardzo szybko się wyczerpać. W tym artykule postaramy się nieco ograniczyć apetyt serwera
Apache2 na logi i oduczymy go logować większość zbędnych komunikatów za sprawą dyrektywy
`SetEnvIf`/`SetEnvIFNoCase` .

<!--more-->
## Całkowite wyłączenie logowania w pliku access.log

Na necie można natknąć się na całą masę postów, których autorzy sugerują całkowite wyłączenie
logowania wszystkich komunikatów trafiających do pliku `/var/log/apache2/access.log` . Moim zdaniem,
nie jest to zbyt rozważna decyzja. A to z tego względu, że pozbawiamy się informacji o próbie
uzyskania dostępu do zasobów zastrzeżonych. Każdy serwis www ma pewne miejsca, do których nie
wszyscy powinni mieć dostęp. Gdy takie osoby czy boty próbują się w ten obszar zapuścić, zwykle
zwiastuje to początek ataku na serwer. To, czy on się powiedzie zależy od całej masy czynników.
Niemniej jednak, jeśli przeglądamy jakiś log, to zawsze jesteśmy w stanie coś zauważyć. Dlatego też
lepiej nie wstawiać znaku `#` na początku tej poniższej linijki:

    CustomLog ${APACHE_LOG_DIR}/access.log combined

Zamiast tego rozwiązania, dużo lepszym sposobem jest ogarnięcie [modułu
mod_log_config](https://httpd.apache.org/docs/current/mod/mod_log_config.html) i wykorzystanie
drzemiącego w nim potencjału.

## Moduł mod_log_config w Apache2

Moduł `mod_log_config` jest standardowo włączony w konfiguracji i wykorzystywany praktycznie przez
każdy serwer Apache2. Jedyne co musimy zrobić, to wskazać mu, które wiadomości życzymy sobie
zalogować, w jakiej formie i do jakich plików takie komunikaty mają zostać przesłane. Do wszystkich
tych celów możemy skorzystać z dwóch dyrektyw `LogFormat` oraz `CustomLog` .

### Dyrektywa CustomLog

Dyrektywa `CustomLog` służy do logowania zapytań klientów odwiedzających nasz serwer www. Przyjmuje
ona dwa lub trzy argumenty. Dwa są wymagane, a jeden opcjonalny. Dla przykładu, weźmy sobie
standardowy wpis, który figuruje w konfiguracji Apache2 (dwa argumenty):

    CustomLog ${APACHE_LOG_DIR}/access.log combined

Ta powyższa linijka rozpoczyna się od nazwy dyrektywy. Następnie mamy pierwszy argument, tj. ścieżkę
do pliku, w której skład wchodzi zmienna `${APACHE_LOG_DIR}` oraz nazwa samego pliku `access.log` .
Obie te rzeczy możemy sobie dostosować. Nie musimy też korzystać ze zmiennej. Jeśli natomiast przy
jej pomocy chcielibyśmy zmienić domyślny katalog logów ( `/var/log/apache2/` ) to wystarczy
przypisać tej zmiennej inną ścieżkę w pliku `/etc/apache2/envvars` . Drugi argument zaś odpowiada
za format logowanych wiadomości. W tym przypadku `combined` odnosi się to aliasu, który figuruje w
pliku `/etc/apache2/apache.conf` :

    LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined

O tym trzecim opcjonalnym argumencie powiemy sobie za chwilę. Na razie spróbujmy wyjaśnić sobie
zapis dyrektywy `LogFormat` .

### Dyrektywa LogFormat

Dyrektywa `LogFormat` odpowiada za format logu trafiającego do pliku, który został określony w
dyrektywie `CustomLog` . Widzimy wyżej, że domyślna dyrektywa `LogFormat` ma dwa argumenty, choć ten
pierwszy jest dość długi. Mamy w nim ciąg `"%h %l %u %t \"%r\" %>s %O \"%{Referer}i\"
\"%{User-Agent}i\""` , który po przetłumaczeniu na ludzki język formatuje komunikat do poniższej
postaci:

  - `%h` -- zdalna nazwa hosta lub adres IP, w zależności czy opcja `HostnameLookups` jest włączona
    w `/etc/apache2/apache2.conf` .
  - `%l` -- zdalna nazwa logowania (za sprawą modułu `mod_ident` ).
  - `%u` -- nazwa użytkownika, w przypadku autoryzacji dostępu do zasobów.
  - `%t` -- czas zalogowania zapytania.
  - `%r` -- pierwsza linia zapytania.
  - `%s` -- status lokalnie przekierowanego zapytania. `%s` odpowiada za oryginalne zapytanie, zaś
    `%>s` za końcowe.
  - `%O` -- wysłane bajty wraz z nagłówkami (wymaga modułu `mod_logio` ).
  - `%{Referer}i` -- odnośnik, z którego klient został przekierowany do danego zasobu serwera.
  - `%{User-Agent}i` -- identyfikator agenta klienta.

Kompletna lista wszystkich możliwych do wykorzystania opcji [jest dostępna w dokumentacji
Apache2](https://httpd.apache.org/docs/current/mod/mod_log_config.html#formats). Poniżej zaś jest
przykładowy komunikat, który został zalogowany w pliku `access.log` :

    crawl-66-249-66-157.googlebot.com - - [03/Aug/2016:16:51:12 +0200] "GET /tag/dnsmasq/ HTTP/1.1" 200 13847 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"

Jak widzimy powyżej, format logu możemy sobie dostosować wedle uznania. Znak `-` oznacza brak
informacji.

Jest też możliwe logowanie warunkowe, tj. takie, gdzie w przypadku wystąpienia jakiegoś zdarzenia,
serwer Apache2 zaloguje dodatkowo, np. referer, lub też go pominie. Poniżej krótki przykład:

    %!200,304,302{Referer}i

Tak jak wcześniej wykorzystujemy `%{Referer}i` ale po znaku `%` dodajemy warunek. Liczby
wykorzystane powyżej odpowiadają za [kod błędu protokołu
HTTP](https://pl.wikipedia.org/wiki/Kod_odpowiedzi_HTTP): OK, NOT MODIFIED, FOUND. Dodatkowo został
użyty `!` , który oznacza negację. Zatem referer nie zostanie zalogowany w tych trzech powyższych
przypadkach.

## Moduł mod_setenvif i zaawansowane logowanie warunkowe

Opcjonalnym trzecim argumentem dyrektywy `CustomLog` jest warunek. Opiera się on o zmienne
środowiskowe, które mogą zostać ustawione za sprawą modułów `mod_setenvif` i/lub `mod_rewrite` .
Nam wystarczy jedynie [moduł
mod_setenvif](https://httpd.apache.org/docs/current/mod/mod_setenvif.html). By z niego skorzystać
trzeba go pierw włączyć w konfiguracji Apache2:

    # a2enmod setenvif
    # systemctl restart apache2

Teraz w konfiguracji wirtualnego hosta możemy określić dyrektywę `SetEnvIf`/`SetEnvIFNoCase` i podać
jej wzór dopasowania, przykładowo:

    SetEnvIf Request_URI "^\/wp-content\/uploads\/[0-9]*\/[0-9]*\/[\.\-0-9a-zA-Z]*\.(png)|(jpg)|(gif)|(jpeg)$" dontlog

W tym przypadku dopasowujemy żądany przez klienta zasób, a właściwie ścieżkę do niego, w oparciu o
[wyrażenia regularne](https://regex101.com/). Generalnie, to pliki `*.png` , `*.jpg` , `*.jpeg` oraz
`*.gif` w katalogu galerii WordPress'a, zostaną oznaczone tagiem `dontlog` . Ta nazwa może być
dowolna ale to ją trzeba będzie uwzględnić w dyrektywie `CustomLog` . Zakładając, że nie chcemy
logować odwołań do tych wszystkich obrazków, możemy w konfiguracji wirtualnego hosta zmienić
standardową dyrektywę `CustomLog` na tę poniższą:

    # CustomLog ${APACHE_LOG_DIR}/access.log combined
    CustomLog ${APACHE_LOG_DIR}/access.log combined env=!dontlog

Został dodany trzeci argument w postaci `env=!dontlog` . Znak `!`, to negacja, zatem jeśli serwer
Apache2 zobaczy komunikat, który nie ma ustawionego taga `dontlog` , to ma go zalogować do pliku
`access.log` . Automatycznie wszystkie komunikaty oznaczone tagiem `dontlog` nie zostaną zalogowane
w tym pliku, a że nie podaliśmy żadnego innego, to zwyczajnie w ogóle nie zostaną zalogowane i o to
nam chodzi.

My skorzystaliśmy jedynie z ustawienia prostego taga w komunikatach za sprawą `Request_URI` .
Niemniej jednak, dyrektywa `SetEnvIf`/`SetEnvIFNoCase` ma [szereg innych aspektów
dopasowań](https://httpd.apache.org/docs/current/mod/mod_setenvif.html#setenvif), które możemy
wykorzystać: `Remote_Host` , `Remote_Addr` , `Server_Addr` , `Request_Method` oraz
`Request_Protocol` . Dopasowania możemy także robić w oparciu o [pola w nagłówku
HTTP](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields). Poniżej kilka
przykładów:

    SetEnvIf User-Agent "Robot$" dontlog
    SetEnvIfNoCase Referer "^jakas.domena.com/link$ dontlog
    SetEnvIf Host "192.168.1.20" dontlog
