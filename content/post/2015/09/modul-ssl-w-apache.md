---
author: Morfik
categories:
- Linux
date: "2015-09-05T18:10:04Z"
date_gmt: 2015-09-05 16:10:04 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
- openssl
title: Moduł ssl w Apache2
---

Przy okazji uszczelniania serwera Apache2, przypomniał mi się [atak
logjam]({{< baseurl >}}/post/logjam-czyli-nowa-podatnosc-w-ssltls/) , przez który to było niemałe
zamieszanie. Pamiętam, że w tamtym czasie próbowałem zabezpieczyć swój serwer testowy, by był na tę
formę ataku odporny. Niemniej jednak, wersja Apache2, która w tamtym czasie była u mnie
zainstalowana, nie do końca dawała taką możliwość. Dziś podszedłem do tej kwestii jeszcze raz i w
oparciu [o ten link](https://weakdh.org/sysadmin.html) udało mi się poprawnie skonfigurować mój
serwer www.

<!--more-->
## Moduł ssl

Serwer Apache2 ma budowę modularną i w zależności od tego, które z tych modułów nam są potrzebne, to
te sobie włączamy. Moduł `ssl` należy do tych, które są domyślnie włączone w większości dystrybucji
linux'owych ale na wypadek, gdyby ktoś jakimś cudem trafił na instalację, gdzie ten moduł nie byłby
włączony, to poniżej jest linijka, która go włączy:

    # a2enmod ssl

Każdy moduł (lub przynajmniej spora większość) posiada swoje pliki konfiguracyjne w katalogu
`/etc/apache2/mods-available/` oraz dowiązania symboliczne do tych plików w katalogu
`/etc/apache2/mods-enabled/` . Edytujemy zatem plik `ssl.conf` , w którym to musimy przepisać kilka
rzeczy.

Przede wszystkim, musimy wyłączyć te stare algorytmy jak SSLv3. Obecnie w Apache2, SSLv2 już nie
jest wspierany kompletnie. Dlatego też możemy go sobie odpuścić zupełnie i na dobre o nim zapomnieć.

    SSLProtocol all -SSLv3

Dodatkowo, definiujemy szereg algorytmów w pewnym określonym porządku -- od najbardziej
bezpiecznych, do tych mniej
    bezpiecznych:

    SSLCipherSuite          ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA

Apache2 musi także honorować tę powyższą kolejność algorytmów, co możemy wymusić w poniższy sposób:

    SSLHonorCipherOrder on

Poniższe kroki zadziałają jedynie dla Apache2 w wersji `2.4.8` i nowszej oraz OpenSSL w wersji
`1.0.2` i późniejszej.

Ostatnim krokiem jest wygenerowanie pliku `dhparams.pem` przy pomocy poniższego polecenia:

    # openssl dhparam -out /ssl/dhparams.pem 4096

Dopisujemy do konfiguracji modułu `ssl` w Apache2 jeszcze tę poniższą linijkę:

    SSLOpenSSLConfCmd DHParameters "/ssl/dhparams.pem"

Teraz już tylko wystarczy zresetować serwer, czy też przeładować konfigurację, i jeśli w logu
Apache2 ( `/var/log/apache2/error.log` ) nie będzie żadnych błędów, znaczy to ni mniej ni więcej jak
tylko tyle, że nowe ustawienia modułu `ssl` zaczęły właśnie obowiązywać. Dobrze jest także rzucić
okiem na certyfikat wykorzystywany przez moduł `ssl` i upewnić się, że [nie jest on podpisany przy
pomocy przestarzałego już algorytmu
SHA-1]({{< baseurl >}}/post/migracja-certyfikatow-openssl-z-sha-1/).
