---
author: Morfik
categories:
- WordPress
date: "2015-05-31T16:19:13Z"
date_gmt: 2015-05-31 14:19:13 +0200
published: true
status: publish
tags:
- apache2
- blog
title: 'WordPress: Ukrycie wp-login.php oraz wp-admin/'
---

By dokonywać jakichkolwiek zmian w WordPressie (i mowa tu nie tylko o dodawaniu treści ale także
zmianie jego plików źródłowych), to musimy uzyskać dostęp do panelu administracyjnego, a tu z kolei
możemy się dostać tylko za pośrednictwem formularza logowania. Panel admina możemy wywołać przez
dopisanie do adresu strony `wp-admin/` , z tym, że jeśli nie jesteśmy zalogowani, to automatycznie
zostaniemy przekierowani na `wp-login.php` . Nie oznacza to bynajmniej, że nie możemy wykonać
zapytania do plików znajdujących się w katalogu `wp-admin/` . Obie te powyższe lokalizacje muszą być
traktowane ze szczególną ostrożnością, a dostęp do nich limitowany i bardzo restrykcyjny.

<!--more-->
## Dodatkowe wtyczki

Możemy się spotkać z szeregiem pluginów, które w mniejszym lub większym stopniu próbują adresować
ten problem ale czy jest sens zaciągać do pracy dodatkowy kod, gdy możemy w bardzo prosty sposób
limitować dostęp do tych krytycznych miejsc w serwisie? [W tym artykule są
opisane](https://codex.wordpress.org/Brute_Force_Attacks) z grubsza dwa wektory ataku na skrypt
WordPressa. Pierwszym są podatności samego kodu tego CMSa, natomiast drugi z nich dotyczy ataku
Brute Force z wykorzystaniem formularza logowania w celu złamania hasła do konta administratora.

## Pliki w wp-admin/ oraz skrypt wp-login.php

Jak już wspomniałem wyżej, osobie niezalogowanej, która przegląda naszego bloga, zostanie
przedstawiony formularz logowania, gdy ta odwiedzi `wp-admin/` lub `wp-login.php` . Jeśli prowadzimy
serwis samotnie i mamy przy tym zaimplementowany zewnętrzny system komentarzy, np. disqus, to mamy
bardzo uproszczone zadanie, bo ochrona naszej strony sprowadzać będzie się do wskazania serwerowi
komu ma wyświetlać formularz logowania i panel admina.

### Analiza zapytania

Zamieńmy się na chwilę w internetowego bota i odwiedźmy naszą stronę próbując wyciągnąć jakiś plik z
katalogu `wp-admin/` . W tym celu posłużymy się narzędziem `wget` , przykładowo:

    $ wget --spider https://morfitronik.lh/wp-admin/about.php --no-check-certificate

Poniżej przeanalizujemy sobie log z tego co się tak naprawdę stanie po wykonaniu powyższego
polecenia. Opcja `--no-check-certificate` jest dodana by obejść problemy z własnoręcznie
wygenerowanym certyfikatem i bez niej nie dałoby rady wykonać zapytania. Z kolei parametr `--spider`
sprawi, że wget będzie zachowywał się bardziej jak zwykły bot (coś na wzór googlowskiego) i nie
pobierze żadnej części witryny, jedynie zwróci szereg komunikatów.

Po wklepaniu powyższego polecenia w terminalu, najsampierw jest rozwiązywana nazwa hosta
`morfitronik.lh` na adres IP, po czym pod ten adres jest wysyłane żądanie:

    ...
    Spider mode enabled. Check if remote file exists.
    --2015-05-31 15:27:10--  https://morfitronik.lh/wp-admin/about.php
    Resolving morfitronik.lh (morfitronik.lh)... 192.168.10.10
    Connecting to morfitronik.lh (morfitronik.lh)|192.168.10.10|:80... connected.
    ...

Jako, że ja u siebie w środowisku testowym mam wymuszone korzystanie z protokołu SSL, to chwilę po
wysłaniu zapytania, zostanie zwrócony komunikat, że zasób został przeniesiony permanentnie (kod
błędu `301` ) na inny adres:

    ...
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://morfitronik.lh/wp-admin/about.php [following]
    ...

Po czym zapytanie zostanie przetworzone jeszcze raz, z tym, że w oparciu o nowy URL:

    ...
    Spider mode enabled. Check if remote file exists.
    --2015-05-31 15:27:10--  https://morfitronik.lh/wp-admin/about.php
    Connecting to morfitronik.lh (morfitronik.lh)|192.168.10.10|:443... connected.
    WARNING: The certificate of 'morfitronik.lh' is not trusted.
    WARNING: The certificate of 'morfitronik.lh' hasn't got a known issuer.
    The certificate's owner does not match hostname 'morfitronik.lh'
    ...

Po wysłaniu zapytania, klient czeka na odpowiedź z serwera i po chwili ją uzyskuje, ale jako że ten
bot nie jest zalogowany w systemie, zamiast żądanego pliku zostanie przekierowany na `wp-login.php`:

    ...
    HTTP request sent, awaiting response... 302 Found
    Location: https://morfitronik.lh/wp-login.php?redirect_to=https%3A%2F%2Fmorfitronik.lh%2Fwp-admin%2Fabout.php&AMP;reauth=1 [following]
    ...

Po przekierowaniu, zostaje przetworzone kolejne zapytanie, tym razem w oparciu o skrypt logowania
`wp-login.php` :

    ...
    Spider mode enabled. Check if remote file exists.
    --2015-05-31 15:27:11--  https://morfitronik.lh/wp-login.php?redirect_to=https%3A%2F%2Fmorfitronik.lh%2Fwp-admin%2Fabout.php&reauth=1
    Connecting to morfitronik.lh (morfitronik.lh)|192.168.10.10|:443... connected.
    WARNING: The certificate of 'morfitronik.lh' is not trusted.
    WARNING: The certificate of 'morfitronik.lh' hasn't got a known issuer.
    The certificate's owner does not match hostname 'morfitronik.lh'
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [text/html]
    Remote file exists and could contain further links,
    but recursion is disabled -- not retrieving.

Kod błędu `200` oznacza, że zasób istnieje i został zwrócony do bota. Serwer nie odmówił dostępu do
pliku `wp-admin/about.php` . Zamiast tego przekierował nas na stronę logowania i tutaj również
serwer nie dorzucił żadnych obostrzeń ze swojej strony.

Dla porównania, jeśli byśmy spróbowali uzyskać dostęp do nieistniejącego pliku w katalogu
`wp-admin/` , to zostanie mam zwrócony poniższy log:

    $ wget --spider  https://morfitronik.lh/wp-admin/about.php-test --no-check-certificate
    ...
    HTTP request sent, awaiting response... 404 Not Found
    Remote file does not exist -- broken link!!!

I tu już mamy kod błędu `404` i właśnie z tego powodu, powinniśmy zablokować możliwość wejścia w
katalog `wp-admin/` nieuprawnionym osobnikom.

Trzeba jednak pamiętać, że niektóre pluginy czy motywy wymagają do poprawnego działania pewnych
plików z katalogu `wp-admin/` , np. `admin-ajax.php` .

Na tym blogu tylko ja będę miał możliwość korzystania z logowania do panelu WordPressa, no bo
wszyscy ludzie, którzy będą chcieli komentować wpisy, będą to już robić przy pomocy disqus'a, a tam
już jest osobny mechanizm logowania i kompletnie niepowiązany ze tym WordPressowskim. Dzięki czemu
nie muszę utrzymywać i martwić się o bazę danych użytkowników, oraz jeśli ktoś posiada konto na
disqus'ie, to będzie miał możliwość komentowania nie tylko moich treści ale również i kontentu całej
masy innych blogerów.

## Zakładanie blokady na wp-login.php i wp-admin/

Mamy kilka opcji co do wprowadzenia restrykcji na te dwa zasoby. Pierwszą z nich jest moduł Apache
`mod_rewrite` , przy pomocy którego to przepiszemy odpowiednie adresy chowając w ten sposób te dwie
lokalizacje. Drugi sposób zaś wykorzystuje dyrektywy `Files` oraz `MatchFiles` do odmawiania dostępu
do plików w obrębie instalacji WordPressa.

### Moduł mod_rewrite

Jeśli chcemy skorzystać z modułu `mod_rewrite` , to w konfiguracji serwera Apache dla katalogu z
instalacją WordPressa lub w pliku `.htaccess` dopisujemy poniższy kod:

    <IfModule mod_rewrite.c>
        RewriteCond %{REQUEST_URI} wp-login.php|wp-admin
        RewriteCond %{REMOTE_ADDR} !^192.168.10.100$
        RewriteRule . - [R=403,L]
    </IfModule>

Mamy tam zdefiniowane dwa warunki i oba z nich muszą być spełnione by serwer zwrócił kod błędu `403`
i zakończył przetwarzanie zapytania. Pierwszym z nich jest wywoływanie adresów mających w nazwie
`wp-login.php` lub `wp-admin` . Drugim zaś adres IP i w tym przypadku każdy adres, za wyjątkiem
`192.168.10.100` będzie pasował.

### Dyrektywy Files i MatchFiles

Jeśli nie chcemy korzystać z powyższego rozwiązania, możemy zaprzęgnąć do roboty dyrektywę `Files`
lub `MatchFiles` . Nie różnią się one zbytnio, jednak ten drugi powinien być używany [w kontekście
szerszego dopasowania](http://httpd.apache.org/docs/2.4/mod/core.html#files) plików, np. za pomocą
wyrażeń regularnych. Zatem jeśli chcemy przy pomocy tych dyrektyw zablokować dostęp do panelu admina
i formularza logowania, dopisujemy w konfiguracji serwera albo również w pliku `.htaccess` poniższe
zwrotki:

    <Files wp-login.php>
        Require all denied
        Require ip 192.168.10.100
    </Files>
    <Files wp-admin>
        Require all denied
        Require ip 192.168.10.100
    </Files>

### Test zabezpieczeń

Odpalmy zatem jeszcze raz `wget` i sprawdźmy czy jest zwracany ten komunikat co trzeba:

    $ wget --spider  https://morfitronik.lh/wp-admin/about.php --no-check-certificate
    ...
    HTTP request sent, awaiting response... 403 Forbidden
    Remote file does not exist -- broken link!!!

Zatem blokada zadziałała. Istnieje możliwość przestawienia kodu błędu na `404` ale nie zalecam tego
robić, bo niekoniecznie googlowi może się podobać, że w naszym serwisie są linki do nieistniejących
zasobów. Od tej pory jedynie klienci z adresu IP `192.168.10.100` będą mieć dostęp do panelu admina
i formularzu logowania.
