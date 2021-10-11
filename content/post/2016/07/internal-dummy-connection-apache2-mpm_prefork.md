---
author: Morfik
categories:
- Linux
date: "2016-07-30T20:00:28Z"
date_gmt: 2016-07-30 18:00:28 +0200
published: true
status: publish
tags:
- apache2
- logi
GHissueID: 400
title: '"Internal dummy connection" w logu Apache2 (mpm_prefork)'
---

Od czasu do czasu przeglądam sobie logi Apache2 w poszukiwaniu pewnych nieprawidłowości. Na dobrą
sprawę, to nie ma tutaj zbytnio dużo roboty, przynajmniej póki co. Niemniej jednak, w pliku
`/var/log/apache2/access.log` co jakiś czas pojawiają się komunikaty zawierające "internal dummy
connection". Za co one odpowiadają i czy można je w zupełności zignorować bez stwarzania zagrożenia
bezpieczeństwa dla serwera www?

<!--more-->
## Proces główny i procesy potomne w Apache2

Standardowo Apache2 w debianie jest w stanie obsłużyć do 150 jednoczesnych połączeń. Niemniej
jednak, gdy serwer nie jest obciążony, to i tak kilka procesów zapasowych (w stanie IDLE) sobie
działa. Chodzi generalnie o to, że gdy nadejdzie połączenie, to może ono zostać od razu obsłużone i
nie trzeba czekać na stworzenie nowego procesu potomnego. Konfiguracja liczby procesów jest
zdefiniowana w pliku `/etc/apache2/mods-enabled/mpm_prefork.conf` :

    <IfModule mpm_prefork_module>
    StartServers              5
    MinSpareServers           5
    MaxSpareServers          10
    MaxRequestWorkers       150
    MaxConnectionsPerChild    0
    </IfModule>

Poniżej są wyjaśnione parametry:

  - `StartServers` -- z tyloma procesami startuje Apache2.
  - `MinSpareServers` -- minimalna liczba procesów zapasowych, które nie obsługują żadnych połączeń.
  - `MaxSpareServers` -- maksymalna liczba procesów zapasowych, które nie obsługują żadnych
    połączeń.
  - `MaxRequestWorkers` -- maksymalna liczba procesów jakie Apache2 może utworzyć.
  - `MaxConnectionsPerChild` -- kontroluje, jak często serwer przetwarza procesy ubijając stare i
    tworząc nowe. Wartość `0` oznacza, że proces nigdy nie wygaśnie i będzie mógł obsługiwać
    połączenia bez końca. Gdyby tutaj dać wartość, np. `1024` , wtedy proces potomny po obsłużeniu
    1024 połączeń zostanie ubity przez proces główny. Można w ten sposób limitować wycieki pamięci.

Przy takiej konfiguracji jak powyżej mamy, serwer Apache2 zostanie odpalony z 5 procesami i będzie
mógł ich posiadać maksymalnie 150. Jeśli serwer będzie realizował w danej chwili 40 połączeń z
klientami, to 40 procesów będzie wykorzystywanych do tego celu, plus 5-10 zapasowych na wypadek
obsługi kolejnych żądań.

Za każdym razem, gdy liczba procesów zapasowych spadnie poniżej progu ustalonego w
`MinSpareServers` , proces główny utworzy kolejny proces potomny. Podobnie sprawa ma się w
przypadku, gdy liczba przekroczy próg w `MaxSpareServers` . Wtedy proces główny ubije taki proces
potomny i zwolni wykorzystywane przez niego zasoby. W przypadku niedoboru procesów, będą one
tworzone w interwale jednej sekundy. Z każdą następną sekundą będzie tworzonych 1, 2, 4 ... 32
procesów, aż zostanie zaspokojony limit w `MaxSpareServers` . Więcej informacji na ten temat można
znaleźć w [dokumentacji Apache2][1].

## Skąd się bierze komunikat "internal dummy connection"

Na [wiki Apache2][2] znalazłem artykuł wyjaśniający kwestię tego całego "internal dummy connection".
Okazuje się, że ten komunikat pojawia się w logu za sprawą opisanych wyżej procesów potomnych, które
Apache2 musi co jakiś czas wybudzać. Robi to przez przesyłanie do samego siebie zwykłego zapytania
HTTP. To zapytanie jest rejestrowane w logu mniej więcej w poniższy sposób:

    ::1 - - [30/Jul/2016:13:05:58 +0200] "OPTIONS * HTTP/1.0" 200 110 "-" "Apache (Debian) OpenSSL/1.0.1 (internal dummy connection)"

Zalogowanie zapytanie przyszło z pętli zwrotnej. W tym przypadku z adresu IPV6 ( `::1` ). Jeśli
serwer nie obsługuje IPv6, wtedy zapytanie nadejdzie z adresu IPv4 ( `127.0.0.1` ). Te zapytania są
normalne, z tym, że generują trochę operacji I/O na dysku. Można zatem pokusić się o nielogowanie
tych wiadomości.

## Jak usunąć komunikat "internal dummy connection" z logu Apache2

W przypadku, gdy chcemy się pozbyć komunikatu "internal dummy connection" z logu Apache2, musimy
napisać regułkę, która odfiltruje nam wiadomości pochodzące z adresu `127.0.0.1` lub `::1` , w
zależności od konfiguracji serwera. Przechodzimy zatem do pliku wirtualnego hosta, np.
`/etc/apache2/sites-enabled/default-ssl.conf` i przerabiamy w nim już istniejącą regułkę od
logowania:

    SetEnvIf Remote_Addr "^127\.0\.0\.1$" dontlog
    SetEnvIf Remote_Addr "^\:\:1$" dontlog
    ErrorLog ${APACHE_LOG_DIR}/error.log
    #CustomLog ${APACHE_LOG_DIR}/access.log combined
    CustomLog ${APACHE_LOG_DIR}/access.log combined env=!dontlog

[Dyrektywie SetEnvIf][3] podajemy jako aspekt dopasowania `Remote_Addr` i porównujemy go z adresem
loopback dla IPv4/IPv6. Wyrażenie regularne musi być ujęte w `" "` . Na końcu zaś mamy dowolną
frazę, która otaguje wszystkie dopasowane wiadomości. Komunikat "internal dummy connection" pojawia
się w pliku `access.log` , zatem na końcu dyrektywy `CustomLog` dopisujemy `env=!dontlog` , by do
tego pliku leciało wszystko co nie ma ustawionego taga `dontlog` .


[1]: https://httpd.apache.org/docs/2.4/mod/prefork.html
[2]: https://wiki.apache.org/httpd/InternalDummyConnection
[3]: https://httpd.apache.org/docs/current/mod/mod_setenvif.html#setenvif
