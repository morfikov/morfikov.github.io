---
author: Morfik
categories:
- Linux
date: "2016-08-05T15:52:05Z"
date_gmt: 2016-08-05 13:52:05 +0200
published: true
status: publish
tags:
- apache2
- debian
- ssl
- tls
GHissueID: 546
title: Problemy z dyrektywą SSLOpenSSLConfCmd w Apache2
---

W stabilnej dystrybucji linux'a Debian niewiele rzeczy ulega zmianie w przeciągu roku czy dwóch lat.
Dlatego też ta gałąź jest wykorzystywana głównie w przypadku serwerów, min. na tym VPS. Na co dzień
jednak korzystam z Debiana SID, czyli gałęzi niestabilnej, która jest nieco bardziej aktualna i
przystosowana do otaczającej nas tej wirtualnej rzeczywistości. Chodzi generalnie o nowsze
oprogramowanie implementujące całą masę ficzerów, których starsze wersje nie posiadają. W tym
przypadku problem dotyczy serwera Apache2, który ostatnimi czasy wypracował szereg mechanizmów
obronnych adresujących ataki na protokół SSL/TLS. Jedną z podatności jest słaba liczba pierwsza
wykorzystywana w [protokole Diffie-Hellman'a][1]. Ten problem można stosunkowo łatwo poprawić w
nowszej wersji Apache2 wykorzystując dyrektywę `SSLOpenSSLConfCmd` . W starszych wersjach ona
niestety nie działa. Niemniej jednak, w dalszym ciągu możemy użyć własnych parametrów dla protokołu
Diffie-Hellman'a, z tym, że trzeba to zrobić nieco inaczej.

<!--more-->
## Optymalna liczba pierwsza dla protokołu Diffie-Hellman'a

Zgodnie z [informacjami zawartymi na tej stronie][2]), liczby pierwsze wykorzystywane w protokole
Diffie-Hellman'a są podatne na atak w przypadku jeśli ich wielkość jest w granicach do 1024 bitów.
Jeśli nasz serwer wykorzystuje liczbę o takiej długości, to powinniśmy ją jak najszybciej zmienić.

Jak ustalić długość bitów liczby pierwszej? Generalnie rzecz biorąc, ilość bitów jest zależna od
OpenSSL, który w starszych wersjach ma uwzględnioną właśnie liczbę pierwszą o długości 1024 bitów.
Nie koniecznie tak samo musi być i w naszym przypadku. Najlepiej jest przeprowadzić sobie test i
ustalić długość tej liczby. [Sam test liczby pierwszej jest dostępny tutaj][3]. W przypadku tego
Debiana, co sieci na moim VPS, wynik się prezentuje następująco:

![dh-liczba-pierwsza-test](/img/2016/08/1.dh-liczba-pierwsza-test.png#huge)

Zatem tutaj wszystko jest w porządku. Co jednak w przypadku, gdy na powyższej fotce widniałaby
wartość 1024 albo nawet i mniejsza? W takiej sytuacji trzeba by wygenerować własne parametry dla
protokołu Diffie-Hellman'a, w których skład weszłaby większa liczba pierwsza. Załóżmy na chwilę, że
te 2048 bitów to za mało i chcemy zrobić użytek z liczby pierwszej o długości 4096 bitów.

## Dyrektywa SSLOpenSSLConfCmd

W konfiguracji modułu `ssl` , tj. w pliku `/etc/apache2/sites-enabled/default-ssl.conf` możemy
określić [dyrektywę SSLOpenSSLConfCmd][4]. W niej z kolei możemy podać opcję `DHParameters` i
ścieżkę do pliku z parametrami dla protokołu Diffie-Hellman'a. Poniżej przykład:

    SSLOpenSSLConfCmd       DHParameters "/etc/letsencrypt/dh/dhparam.pem"

Teraz przy pomocy narzędzia `openssl` musimy sobie ten plik wygenerować:

    # openssl dhparam -out dhparam.pem 4096

Wykorzystana wyżej wartość `4096` to ilość bitów dla liczby pierwszej. Im więcej bitów określimy,
tym proces generowania pliku będzie trwał dłużej. W tym przypadku zajęło to kilkanaście minut.

Trzeba mieć jednak na względzie, że wymagane jest od nas posiadanie serwera Apache2 w wersji minimum
2.4.8 oraz OpenSSL w wersji co najmniej 1.0.2 . W Debianie stable, mamy zaś:

    # apachectl -V
    Server version: Apache/2.4.10 (Debian)
    ...

    # openssl version
    OpenSSL 1.0.1t  3 May 2016

Zatem ten powyższy sposób zmiany liczby pierwszej nie przejdzie ale nic straconego.

## Dołączanie parametrów DH do certyfikatu

Starsze wersje serwera Apache2 nie obsługują dyrektywy `SSLOpenSSLConfCmd` . Możemy jednak dołączyć
zawartość wyżej wygenerowanego pliku `dhparam.pem` do pliku certyfikatu, którego ścieżka figuruje w
dyrektywie `SSLCertificateFile` w konfiguracji wirtualnego hosta. Jeśli [konfigurowaliśmy certyfikat
letsencrypt][5], to zawartość `dhparam.pem` dopisujemy do pliku
`/etc/letsencrypt/live/morfitronik.pl/fullchain.pem` . Robimy to w poniższy sposób:

    # cat /etc/letsencrypt/dh/dhparam.pem >> /etc/letsencrypt/live/morfitronik.pl/fullchain.pem

Teraz już wystarczy zresetować serwer Apache2 i przetestować czy ta liczba pierwsza została
uwzględniona przeprowadzając jeszcze raz test:

![dh-liczba-pierwsza-test](/img/2016/08/2.dh-liczba-pierwsza-test.png#huge)

I jak widzimy, ilość bitów z 2048 wzrosła do 4096.


[1]: https://pl.wikipedia.org/wiki/Protok%C3%B3%C5%82_Diffiego-Hellmana
[2]: https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html#Logjam_(DH_EXPORT
[3]: https://weakdh.org/sysadmin.html
[4]: https://httpd.apache.org/docs/trunk/mod/mod_ssl.html#sslopensslconfcmd
[5]: /post/certyfikat-letsencrypt-dla-bloga-certbot/
