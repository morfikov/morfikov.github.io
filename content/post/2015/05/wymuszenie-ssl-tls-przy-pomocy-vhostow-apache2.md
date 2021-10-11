---
author: Morfik
categories:
- Linux
date: "2015-05-26T11:24:13Z"
date_gmt: 2015-05-26 10:24:13 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
- ssl
- tls
GHissueID: 236
title: Wymuszenie SSL/TLS przy pomocy vhost'ów w Apache2
---

Mając na stronie www formularze logowania czy rejestracji (ewentualnie panele administracyjne),
rozmyślnym krokiem jest implementacja protokołu SSL/TLS. Jeśli mamy potrzebę, możemy także pokusić
się o zaszyfrowanie całego ruchu w obrębie naszej witryny. Problem w tym, że odwiedzający naszą
stronę użytkownicy mogą korzystać z linków, czy wpisywać adresy, które nie rozpoczynają się od
`https://` , a jedynie od `http://` . W takim przypadku, nawet jeśli szyfrujemy ruch w serwisie, to
odwiedzenie tego typu adresu zwróci nam stronę kanałem nieszyfrowanym, co może godzić w
bezpieczeństwo samej strony jak i również w naszą/czyjąś prywatność. W tym krótki artykule
spróbujemy tak skonfigurować serwer Apache2, by przekierował tego typu zapytania i słał je
szyfrowanym tunelem.

<!--more-->
## Jak wymusić SSL w Apache2?

Jeśli chodzi o zwyczajne przekierowanie ruchu z portu 80 na 443, to nie potrzebujemy do tego nawet
modułu `mod_rewrite` , a już na pewno nie zaleca się tego robić via plik `.htaccess` . [Możemy za to
posłużyć](https://wiki.apache.org/httpd/RedirectSSL) się dyrektywą `Redirect` . Oczywiście, będzie
nam także potrzebna konfiguracja dla wirtualnych hostów na obu tych portach. W debianie standardowo
mamy już wszystko przygotowane, tj. konfiguracja hostów dla portu 80 jest trzymana w pliku
`/etc/apache2/sites-available/000-default.conf` , natomiast tych, które mają korzystać z SSL/TLS
(port 443) w pliku `/etc/apache2/sites-available/default-ssl.conf` .

Zakładam, że mamy już włączone oba te pliki w konfiguracji Apache2 (linki do katalogów
`/etc/apache2/sites-enabled/`) oraz, że mamy zdefiniowane działające wirtualne hosty i jesteśmy w
stanie przeglądać swój serwis na obu powyższych portach.

Ja u siebie w pliku `000-default.conf` mam ten poniższy blok:

    <VirtualHost *:80>
        ServerName morfitronix.lh
        ServerAdmin morfik@mhouse.lh
        DocumentRoot /apache/morfitronix
        ErrorLog ${APACHE_LOG_DIR}/error_morfitronix.log
        CustomLog ${APACHE_LOG_DIR}/access_morfitronix.log combined
    </VirtualHost>

Zaś w pliku `default-ssl.conf` taką zwrotkę:

    <IfModule mod_ssl.c>
        <VirtualHost *:443>
            ServerName morfitronix.lh
            ServerAdmin morfik@mhouse.lh
            DocumentRoot /apache/morfitronix
            ErrorLog ${APACHE_LOG_DIR}/error_morfitronix.log
            CustomLog ${APACHE_LOG_DIR}/access_morfitronix.log combined
            SSLEngine on
            SSLCertificateFile      /etc/apache2/ssl/localhost.crt
            SSLCertificateKeyFile   /etc/apache2/ssl/localhost.key
        </VirtualHost>
    </IfModule>

By wymusić przekierowanie z portu 80 na 443, musimy w pliku `000-default.conf` , wewnątrz dyrektywy
`<VirtualHost *:80>` , dopisać poniższą linijkę:

    Redirect permanent / https://morfitronik.pl/

Od tej chwili każdy niezabezpieczony adres czy link na naszej stronie, zostanie automatycznie
przepisany na `https://` przy próbie jego odwiedzenia.
