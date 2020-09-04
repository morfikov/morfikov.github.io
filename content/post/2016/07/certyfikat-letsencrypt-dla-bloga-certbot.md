---
author: Morfik
categories:
- Linux
date: "2016-07-23T12:00:28Z"
date_gmt: 2016-07-23 10:00:28 +0200
published: true
status: publish
tags:
- apache2
- blog
- certyfikaty
- ssl
- tls
title: Certyfikat Let's Encrypt dla bloga WordPress (certbot)
---

Jeszcze nie tak dawno temu, rzadko który serwis internetowy wykorzystywał certyfikaty SSL/TLS do
zabezpieczenia komunikacji między serwerem, a łączącymi się do niego klientami. W dalszym ciągu
jednak notuje się strony bez "zielonej kłódki" ale na szczęście jest ich coraz mniej w naszym
otoczeniu. Liczbę tych stron można by z powodzeniem ograniczyć jeszcze bardziej, gdyby takie
certyfikaty były za free, łatwe do zaimplementowania i dostępne praktycznie dla każdego od tak. No i
na dobrą sprawę są, tylko ludzie jeszcze nie zdają sobie z tego sprawy. Istnieje bowiem [projekt
Let's Encrypt](https://letsencrypt.org/), który umożliwia stosunkowo bardzo proste wdrożenie
certyfikatu na serwerze www opartym, np. o oprogramowanie Apache2. W tym wpisie zobaczymy jak ta
cała procedura implementacji certyfikatu SSL/TLS przebiega.

<!--more-->
## Konfiguracja Apache2 pod SSL/TLS

Przede wszystkim, nasz serwer www musi być dostępny w internecie po SSL/TLS. Let's Encrypt będzie
próbował nawiązać z nim połączenie w fazie generowania certyfikatów. W przypadku, gdy firewall lub
inne nieznane czynniki przeszkodzą w komunikacji, to certyfikaty nie zostaną wygenerowane i proces
będzie trzeba powtórzyć. Moduł `ssl` w konfiguracji Apache2 włączamy w poniższy sposób:

    # a2enmod ssl
    # a2ensite default-ssl.conf
    # systemctl restart apache2

Standardowo w dystrybucji Debian mamy uwzględnione tymczasowe certyfikaty `snakeoil` (dostępne w
`/etc/ssl/certs/` ), przez co możemy łączyć się do serwera po SSL/TLS. Niemniej jednak, taki
certyfikat nie jest honorowany przez żadną przeglądarkę i ta zwyczajnie zwróci nam błąd połączenia
ale to nam w niczym nie przeszkadza.

Drugą ważną rzeczą jest odpowiednia konfiguracja vHostów. Każdy vHost w Apache2 musi mieć przypisaną
domenę. Te domeny zaś będą przedmiotem certyfikatu. Standardowo Apache2 ma skonfigurowanego jednego
vHosta zarówno dla protokołu HTTP jak i HTTPS. Jako, że my implementujemy certyfikaty SSL/TLS,
interesuje nas plik `/etc/apache2/sites-enabled/default-ssl.conf` . W nim musimy uwzględnić
dyrektywę `ServerName` oraz (opcjonalnie) `ServerAlias` . Poniżej jest przykład konfiguracji:

    <VirtualHost _default_:443>
        ServerName morfitronik.pl
        ServerAlias www.morfitronik.pl
     </VirtualHost>

Użyte wyżej `morfitronik.pl` oraz `www.morfitronik.pl` to nazwy domen, dla których wyrabiamy
certyfikat. Oczywiście można skorzystać jedynie z pierwszej opcji ale niektóre sytuacje wymagają od
nas również uwzględnienia `www` . Przykładem może być [dodawanie rekordu do listy
HSTS]({{< baseurl >}}/post/http-strict-transport-security-hsts-apache2/). Dlatego też najlepiej
jest określić te dwa adresy i generować certyfikat dla obu domen. Gdybyśmy tego nie uczynili, to
klienci przy przeglądaniu naszego serwisu dostaliby poniższy błąd:

![]({{< baseurl >}}/img/2016/07/1.letsencrypt-blad-domenta-www.png#big)

## Let's Encrypt, certbot i Debian backports

Większość VPS'ów oferuje różne dystrybucje linux'a, z tym, że w wersji stabilnej. Dla Debiana
oznacza to zawsze przedział czasu liczony w latach. Przez taki okres czasu bardzo dużo rzeczy może
się zmienić i tak jest w przypadku Let's Encrypt . Jest to dość świeży projekt i odpowiednie pakiety
nie trafiły do wydania stabilnego. Dlatego też musimy się posiłkować
[backports'ami](https://backports.debian.org/) dodając odpowiednie adresy w pliku
`/etc/apt/sources.list` :

    deb http://ftp.debian.org/debian jessie-backports main

Aktualizujemy listy repozytoriów i [instalujemy pakiet
certbot](https://certbot.eff.org/#debianjessie-apache), bo to przy jego pomocy będziemy generować i
odnawiać certyfikaty.

## Generowanie certyfikatów Let's Encrypt

Mając już zainstalowane odpowiednie pakiety, możemy przejść do generowania certyfikatów. W przypadku
Apache2 można to zrobić na dwa sposoby: automatycznie lub ręcznie. My wybierzemy wersję ręczną, bo
nie warto ufać automatom. Ten sposób sprowadza się do wydania w terminalu tego poniższego polecenia:

    # certbot --apache certonly

Teraz musimy odpowiedzieć na szereg pytań. Pierwsze z nich dotyczy adresu email, który ma być
powiązany z certyfikatem, np. na wypadek utraty:

![]({{< baseurl >}}/img/2016/07/2.letsencrypt-konfiguracja-apache-email.png#big)

Dalej mamy zapytanie dotyczące wyboru domen w oparciu o skonfigurowane vHosty:

![]({{< baseurl >}}/img/2016/07/3.letsencrypt-konfiguracja-apache-vhost.png#big)

W przypadku późniejszego dodawania kolejnych domen czy subdomen będzie trzeba na nowo wybrać
wszystkie vHosty, co zaowocuje poniższym komunikatem:

![]({{< baseurl >}}/img/2016/07/4.letsencrypt-konfiguracja-apache-vhost-dodanie-domeny.png#big)

I w zasadzie to wszystko, przynajmniej w przypadku generowania certyfikatu. Zróbmy sobie także
backup całego katalogu `/etc/letsencrypt/` , bo to tam są przechowywane wszystkie dane dotyczące
certyfikatów.

## Konfiguracja Apache2 pod certyfikaty Let's Encrypt

Musimy jeszcze odpowiednio skonfigurować sam serwer Apache2. W tym celu edytujemy plik
`/etc/apache2/mods-available/ssl.conf` i zmieniamy istniejącą konfigurację dla modułu `ssl` . Ma ona
zawierać te poniższe wpisy:

    SSLProtocol             all -SSLv2 -SSLv3
    #SSLCipherSuite          ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA
    SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH

    SSLHonorCipherOrder     on
    SSLCompression          off
    SSLOptions              +StrictRequire

Następnie edytujemy plik `/etc/apache2/sites-available/default-ssl.conf` , w którym to dodajemy
poniższy kod:

    <VirtualHost _default_:443>
    ...
    LogLevel info ssl:warn
    LogLevel warn
    CustomLog /var/log/apache2/access.log vhost_combined
    ErrorLog /var/log/apache2/error.log
    Header edit Set-Cookie (?i)^(.*)(;\s*secure)??((\s*;)?(.*)) "$1; Secure$3$4"
    ...
    </VirtualHost>

Ten wpis z `Header edit` ma na celu wymuszenie przesyłania ciasteczek kanałem szyfrowanym przez
ustawienie w nich [flagi Secure](https://www.owasp.org/index.php/SecureFlag).

Niżej w pliku `/etc/apache2/sites-available/default-ssl.conf` uzupełniamy dane dotyczące
certyfikatów. Wskazujemy pliki zlokalizowane w katalogu `/etc/letsencrypt/live/morfitronik.pl/` :

    <VirtualHost _default_:443>
    ...
    SSLCertificateFile          /etc/letsencrypt/live/morfitronik.pl/fullchain.pem
    SSLCertificateKeyFile       /etc/letsencrypt/live/morfitronik.pl/privkey.pem
    ...
    </VirtualHost>

## Test certyfikatu Let's Encrypt

Pozostaje nam już tylko restart serwera Apache2 i przetestowanie szyfrowania. W tym celu odpalamy
przeglądarkę i udajemy się pod adres jednej z domen, dla których wygenerowaliśmy certyfikat. Obok
pola adresu powinniśmy ujrzeć zieloną kłódkę:

![]({{< baseurl >}}/img/2016/07/5.letsencrypt-test-certyfikat-www.png#huge)

Widzimy wyraźnie, że certyfikat został zweryfikowany przez Let's Encrypt.

Skoro już testujemy certyfikat, to wypadałoby także przetestować cały protokół SSL/TLS, którego
obsługę mamy zaimplementowaną na serwerze. By taki test przeprowadzić, udajemy się pod [ten
link](https://www.ssllabs.com/ssltest/index.html) i wpisujemy adres domeny, którą zamierzamy poddać
sprawdzeniu. W przypadku tego serwisu wyszło coś takiego:

![]({{< baseurl >}}/img/2016/07/6.test-https.png#huge)

## Cykliczne odnawianie certyfikatu Let's Encrypt via cron

Certyfikat Let's Encrypt jest ważny tylko przez trzy miesiące. Nie oznacza to, że nie możemy
przedłużyć certyfikatu przed datą jego wygaśnięcia. W pakiecie `certbot` był zawarty plik (
`/etc/cron.d/certbot` ) z instrukcją dla cron'a, która cyklicznie może nam ten certyfikat odnawiać.
Standardowo ten proces odbywa się 2 razy dziennie. Przynajmniej tak zalecają devy Let's Encrypt .
Niemniej jednak, proces odnowy nastąpi dopiero, gdy do daty wygaśnięcia certyfikatu pozostanie mniej
niż 30 dni.
