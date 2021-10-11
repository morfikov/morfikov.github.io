---
author: Morfik
categories:
- Blog
date: "2016-07-19T14:07:05Z"
date_gmt: 2016-07-19 12:07:05 +0200
published: true
status: publish
tags:
- apache2
- wordpress
- certyfikaty
GHissueID: 389
title: Certyfikat chroniący wp-login.php i wp-admin/
---

Jednym z bardziej wyrafinowanych sposobów ochrony zasobów (katalogów) na serwerze www opartym o
oprogramowanie Apache2 jest wykorzystanie certyfikatów klienckich. Tego typu zabezpieczenie można
jednak zastosować tylko i wyłącznie w przypadku stron, które korzystają z szyfrowania SSL/TLS. Dla
przykładu weźmy sobie blog WordPress'a, gdzie mamy katalog `wp-admin/` i plik `wp-login.php` .
Formularz logowania oraz panel admina zwykle są szyfrowane. Zatem każdy taki blog powinien robić już
użytek z tunelu SSL/TLS w mniejszym lub większym stopniu. Jeśli teraz mamy dość niestandardową
instalację WordPress'a, to przy pomocy certyfikatów możemy weryfikować użytkowników, którzy chcą
uzyskać dostęp do tych w/w lokalizacji. Jest to nieco inne podejście w stosunku do tego, które
zostało opisane w artykule o [ukrywaniu wp-login.php oraz
wp-admin](/post/wordpress-ukrycie-wp-login-php-oraz-wp-admin/), gdzie był
wykorzystywany moduł `mod_rewrite` oraz dyrektywy `Files` i `MatchFiles` . Takie certyfikaty
klienckie dają nam jednak większe pole manewru, bo identyfikują konkretnego użytkownika chcącego
uzyskać dostęp do zasobów serwera. Ten wpis ma na celu pokazanie w jaki sposób zaimplementować
obsługę certyfikatów klienckich w Apache2.

<!--more-->
## Centrum certyfikacji i certyfikaty klienckie

W celu zaimplementowania obsługi certyfikatów klienckich w Apache2, musimy przygotować sobie pierw
centrum certyfikacji (Certificate Authority). CA możemy utworzyć na kilka sposobów. Najprościej jest
wgrać sobie pakiet [easy-rsa](/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/)
i wydać w terminalu poniższe polecenie:

    # make-cadir /etc/CA/

Edytujemy teraz plik `/etc/CA/vars` , dostosowujemy w nim poszczególne parametry i tworzymy CA:

    # cd /etc/CA/
    # source ./vars
    # ./clean-all
    # ./build-ca

CA mamy już przygotowane. Musimy jeszcze wygenerować certyfikat dla każdego klienta, który ma mieć
dostęp do określonych zasobów na serwerze:

    # ./build-key-pkcs12 client

## Nakładanie restrykcji na wp-login.php i wp-admin/ w Apache2

W katalogu `/etc/CA/keys/` zostało utworzonych trochę plików. W konfiguracji Apache2, tj. w pliku
`/etc/apache2/sites-enabled/default-ssl.conf` , musimy podać ścieżkę do `ca.crt` . Otwieramy zatem
ten plik i szukamy linijki z `SSLCACertificateFile` :

    SSLCACertificateFile /etc/CA/keys/ca.crt

Teraz możemy przejść do konfiguracji dostępu do zasobów na serwerze. W tym celu niżej w pliku
`/etc/apache2/sites-enabled/default-ssl.conf` dodajemy poniższy kod:

    <Directory "/apache2/wordpress/wp-admin">
    #   SSLOptions +StdEnvVars
        SSLVerifyClient optional
        SSLVerifyDepth 1
    </Directory>

    <Files "wp-login.php">
    #   SSLOptions +StdEnvVars
        SSLVerifyClient optional
        SSLVerifyDepth 1
    </Files>

Dyrektywa `Directory` wskazuje nam katalog, na który mają być założone restrykcje. Podobnie sprawa
ma się w przypadku dyrektywy `Files` , z tym, że ta dotyczy plików. Pozycja
[SSLVerifyClient](https://httpd.apache.org/docs/current/mod/mod_ssl.html#sslverifyclient) może
przyjąć poniższe wartości:

  - `none` -- nie jest wymagany żaden certyfikat.
  - `optional` -- klient może okazać ważny certyfikat.
  - `require` -- klient musi okazać ważny certyfikat.
  - `optional_no_ca` -- klient może okazać ważny certyfikat, z tym, że nie musi on być z powodzeniem
    zweryfikowany przez serwer.

Różnica między `optional` i `require` nie jest tak oczywista jak mogłoby się wydawać na pierwszy
rzut oka. Można by pomyśleć, że nas będzie interesować jedynie `require` , no bo przecie chcemy
wymusić przedstawienie certyfikatu przez klienta. Okazuje się jednak, że ustawienie którejkolwiek z
tych dwóch wartości będzie skutkowało żądaniem od klienta ważnego certyfikatu, który będzie podlegał
weryfikacji przez serwer. W przypadku, gdy cert nie zostanie zweryfikowany, to klient nie zostanie
dopuszczony do chronionego zasobu. Po co nam zatem dwie wartości skoro robią w grubsza to samo? W
przypadku ustawienia `optional` serwer będzie w stanie przekierować klienta na dowolną stronę
zgodnie z regułami w module `mod_rewrite` . O tym będzie w dalszej części artykułu. Zapisujemy plik
i restartujemy Apache2.

## Konfiguracja certyfikatu na klientach

Gdybyśmy w tym momencie otworzyli przeglądarkę internetową i spróbowali przejść na blogu do panelu
admina (katalog `wp-admin/`) lub zalogować się (plik `wp-login.php` ), to naszym oczom powinien
ukazać się poniższy komunikat:

![](/img/2016/07/1.wp-admin-wp-login.php-apache2-brak-certyfikat.png#big)

Serwis jako taki będzie działać bez problemu po SSL/TLS. Niemniej jednak, na obecną chwilę nie
uzyskamy dostępu do tych powyższych zasobów. Musimy posiadać certyfikat kliencki, który zostanie
zweryfikowany przez serwer. Ten certyfikat kliencki już sobie utworzyliśmy i jest to plik
`/etc/CA/keys/client.p12` . Musimy go teraz zaimportować w przeglądarce. W tym przypadku zrobimy to
na przykładzie Firefox'a. Przechodzimy zatem kolejno do Preferences > Advanced. Następnie na
zakładce Certificates klikamy View Certificates. W okienku, które się pojawi, przechodzimy na
zakładkę Your Certificates i klikamy w przycisk Import, gdzie podajemy ścieżkę do pliku
`client.p12` :

![](/img/2016/07/2.firefox-dodawanie-certyfikat.png#big)

Ponownie odwiedzamy stronę logowania lub panel administracyjny na naszym blogu. Tym razem powinno
nam wyskoczyć takie oto okienko:

![](/img/2016/07/3.firefox-potwierdzenie-certyfikat.png#huge)

Klikamy `OK` i po chwili powinniśmy uzyskać dostęp do chronionych zasobów serwera.

## Przekierowanie klientów bez certyfikatów

W przypadku, gdy jakiś nieuprawniony klient będzie próbował uzyskać dostęp do chronionych zasobów,
to zostanie mu zwrócony komunikat błędu, który widzieliśmy wyżej. Nie jest to zbytnio eleganckie
rozwiązanie i przydałoby się zadbać, aby tacy użytkownicy zostali przekierowani, np. na stronę
główną bloga. Musimy zatem nieco przerobić dyrektywy `Directory` oraz `Files` :

    <Directory "/apache/wordpress/wp-admin">
        #SSLOptions +StdEnvVars
        SSLVerifyClient optional
        SSLVerifyDepth 1

        <IfModule mod_rewrite.c>
            RewriteEngine   on
            RewriteCond %{SSL:SSL_CLIENT_VERIFY} !^SUCCESS$
            RewriteRule ^(.*) https://%{HTTP_HOST}/ [R=301,L]
        </IfModule>

    </Directory>

    <Files "wp-login.php">
        # SSLOptions +StdEnvVars
        SSLVerifyClient optional
        SSLVerifyDepth 1

        <IfModule mod_rewrite.c>
            RewriteEngine   on
            RewriteCond %{SSL:SSL_CLIENT_VERIFY} !^SUCCESS$
            RewriteRule ^(.*) https://%{HTTP_HOST}/ [R=301,L]
        </IfModule>

    </Files>

W obu dyrektywach pojawił się kod z regułami dla modułu `mod_rewrite` . Mamy w nim jeden warunek,
który odnosi się do weryfikacji certyfikatu klienta przez serwer. Jeśli wynik jest inny od
`SUCCESS` , oznacza to, że klient nie został zweryfikowany prawidłowo. W takim przypadku zostanie
dopasowania reguła, która przekieruje żądanie na stronę główną.
