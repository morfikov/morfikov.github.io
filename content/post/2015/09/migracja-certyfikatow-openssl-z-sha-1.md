---
author: Morfik
categories:
- Linux
date: "2015-09-05T17:15:12Z"
date_gmt: 2015-09-05 15:15:12 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
- certyfikaty
- openssl
GHissueID: 143
title: Migracja certyfikatów OpenSSL z SHA-1
---

Szukając co by tutaj jeszcze poprawić w moim środowisku testowym, w którym działa między innymi
apache, natrafiłem na komunikat w firefoxie, który oznajmił mi, że certyfikat mojego serwera
korzysta z przestarzałego już algorytmu mieszającego (hash). W tym przypadku jest to SHA-1. [Jak
można przeczytać na blogu
mozilli](https://blog.mozilla.org/security/2014/09/23/phasing-out-certificates-with-sha-1-based-signature-algorithms/),
algorytm SHA-1 wypadł z łask jakiś czas temu i obecnie nie zaleca się jego używania ze względów
bezpieczeństwa. Postanowiłem zatem poprawić tę lukę.

<!--more-->
## Potrzebny nowy certyfikat by zmienić SHA-1

Niestety nie da się inaczej wyeliminować problemu ze słabym hash'em jak poprzez wygenerowanie nowego
certyfikatu z poprawnie ustawionym algorytmem mieszającym. Można sobie wybrać np. SHA-256, czy też
SHA-512 i ja wybrałem właśnie ten drugi wariant. Jeśli nie zdecydujemy się na ten krok, firefox (i
inne przeglądarki również) będą zwracać komunikaty podobne do tych poniżej:

![](/img/2015/09/01.slaby-hash-sha-1-apache-certyfikat.png#huge)

Jak możemy przeczytać w wyżej przytoczonym linku, dzieje się tak, bo integralność algorytmu
hash'ującego używanego przy podpisywaniu certyfikatu odgrywa ogromną rolę i jest elementem
krytycznym jeśli chodzi o bezpieczeństwo samego certyfikatu. W przypadku wykorzystywania słabego
algorytmu mieszającego, atakujący może uzyskać fałszywy certyfikat i nim się posłużyć. Sam SHA-1
jest już dość leciwym algorytmem, bo ma prawie 20 lat i najwyższy czas porzucić go.

Rzecz w tym, że domyślne ustawienia OpenSSL, które są przechowywane w pliku `/etc/ssl/openssl.cnf`
nadal wskazują na algorytm SHA-1. Rozchodzi się o pozycję `default_md` i to jej wartość należy
zmienić, przykładowo na `sha512` :

    default_md      = sha512                # use public key default MD

Dzięki tej operacji, wszystkie certyfikaty, które zostaną wygenerowane od tego momentu, będą
automatycznie podpisywane algorytmem `sha512` i o to nam chodzi.

Jeśli nie mamy dostępu do globalnej konfiguracji OpenSSL lub też z jakichś względów nie chcemy jej
zmieniać, to zawsze możemy wygenerować sobie klucz i certyfikat dla serwera apache w poniższy
sposób:

    # openssl req -x509 -nodes -sha512 -days 365 -newkey rsa:4096 -keyout /ssl/localhost.key -out /ssl/localhost.crt

Za sprawą tej operacji zostaną utworzone dwa pliki `localhost.key ` oraz `localhost.crt ` , do
których to odwołujemy się w konfiguracji apache w pliku
`/etc/apache2/sites-enabled/default-ssl.conf` :

    ...
    SSLCertificateFile      /ssl/localhost.crt
    SSLCertificateKeyFile   /ssl/localhost.key
    ...

## Weryfikacja hash'a

Oczywiście przydałoby się także sprawdzić czy sygnatura została wygenerowana przy pomocy `sha512` .
W tym celu wdajemy poniższe polecenie:

    # openssl x509 -in /ssl/localhost.crt -text
    Certificate:
        Data:
    ...
        Signature Algorithm: sha512WithRSAEncryption
    ...

I jak widzimy powyżej, została. Restartujemy jeszcze serwer apache i jeśli teraz spojrzymy na
konsolę w firefox'ie, to wszystkie błędy związane z hash'em powinny zniknąć przy przeglądaniu
naszej witryny.
