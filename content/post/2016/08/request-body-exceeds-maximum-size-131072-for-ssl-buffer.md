---
author: Morfik
categories:
- Linux
date: "2016-08-24T22:52:29Z"
date_gmt: 2016-08-24 20:52:29 +0200
published: true
status: publish
tags:
- apache2
- wordpress
- ssl
- tls
GHissueID: 568
title: Request body exceeds maximum size (131072) for SSL buffer
---

Dziś chciałem zaktualizować jeden z moich bardziej obszerniejszych wpisów na tym blogu ale
oczywiście nie mogło odbyć się bez problemów. Gdy już wszystkie poprawki zostały naniesione i cały
artykuł trafił do formularza WordPress'a, przeglądarka zwróciła mi błąd `Request Entity Too Large` .
Z początku nie wiedziałem o co chodzi ale, że ten aktualizowany artykuł był naprawdę długi, to
domyśliłem się, że chodzi o ilość bajtów, które chciałem przesłać w zapytaniu. Przeglądając logi
serwera Apache2, znalazłem tam jeszcze dodatkowo komunikaty `[ssl:error] request body exceeds
maximum size (131072) for SSL buffer` oraz `[ssl:error] could not buffer message body to allow SSL
renegotiation to proceed` . Może ta cała sytuacja brzmi groźnie ale wybrnięcie z niej jest wręcz
banalne. Wystarczy dostosować wartość dyrektywy `SSLRenegBufferSize` w konfiguracji serwera Apache2.

<!--more-->
## Co to jest renegocjacja połączenia SSL/TLS (SSL renegotiation)

By zrozumieć pojęcie renegocjacji połączenia SSL/TLS zobrazujmy je sobie na [prostym
przykładzie](https://devcentral.f5.com/articles/ssl-profiles-part-6-ssl-renegotiation). Załóżmy, że
mamy użytkownika i jakiś sklep online. Ten użytkownik przegląda sobie ofertę sklepu nie będąc
zalogowanym na swoje konto. Niemniej jednak, komunikacja odbywa się kanałem szyfrowanym. Gdy ta
osoba znajdzie produkt, który chce kupić, loguje się na swoje konto i teraz się pojawia problem.

Gdyby ten klient w tym momencie się zalogował na swoje konto, to trzeba by stworzyć drugie
połączenie SSL/TLS. Te dwa szyfrowane połączenia nie byłyby ze sobą powiązane w żaden sposób i
informacje z pierwszej sesji zwyczajnie by przepadły. Przy renegocjacji połączenia SSL/TLS,
połączenie tego klienta jest cały czas utrzymywane. Z chwilą, gdy będzie on chciał się zalogować
do serwisu, to nowa sesja wykorzysta to zestawione już połączenie SSL/TLS (będą użyte te same
parametry i klucze szyfrujące). W efekcie sesje będzie można powiązać ze sobą i informacje z tej
poprzedniej nie przepadną.

Przy renegocjacji połączenia, komunikaty `Server Hello` oraz `Client Hello` są przesyłane wewnątrz
zaszyfrowanego tunelu. Później następuje proces witania poprzedzający proces renegocjacji
połączenia.

## Bufor połączenia SSL/TLS

W przypadku tego bloga WordPress, dostęp do panelu administracyjnego jest chroniony za pomocą
certyfikatów. Jest zatem wykorzystywana dyrektywa `SSLVerifyClient` min. na katalogu `wp-admin/` . W
takiej sytuacji, moduł `ssl` serwera Apache2 musi buforować w pamięci każde zapytanie dokonywane w
panelu administracyjnym do momentu, aż nowy proces witania (SSL handshake) się zakończy.

Zatem, gdy przygotowuję posta do publikacji, jego zawartość musi zostać zbuforowana przed wysłaniem.
Problem w tym, że ten bufor domyślnie ma ustawiony rozmiar jedynie 128 KiB. Stąd w przeglądarce
pojawia się komunikat:

    Request Entity Too Large
    The requested resource
    /wp-admin/post.php
    does not allow request data with POST requests, or the amount of data provided in the request exceeds the capacity limit.

A w logu serwera Apache2 poniższe wiadomości:

    [ssl:error] AH02018: request body exceeds maximum size (131072) for SSL buffer
    [ssl:error] AH02257: could not buffer message body to allow SSL renegotiation to proceed

Rozmiar `131072` jest w bajtach, a mój post miał około 140 KiB i wyszedł poza ustalony limit.

## Dyrektywa SSLRenegBufferSize

Napisać artykuł, który waży 140 KiB, to naprawdę sztuka. Niemniej jednak, trzeba jakoś dostosować
konfigurację serwera Apache2, by tego typu posty przechodziły bez problemu. Skoro udało mi się
przekroczyć granicę 128 KiB, to następna będzie 256 KiB i ten rozmiar przydałoby się jakoś
zdefiniować. Pamiętajmy jednak, że jeśli chcemy wysłać grafikę, to prawdopodobnie 256 KiB nam nie
wystarczy i trzeba będzie ten limit podnieść jeszcze bardziej.

Do zmiany rozmiaru cache przy renegocjowaniu szyfrowanych połączeń służy dyrektywa
[SSLRenegBufferSize](https://httpd.apache.org/docs/current/mod/mod_ssl.html#sslrenegbuffersize).
Trzeba ją określić wewnątrz dyrektywy `Directory` , zwykle tej, która zawiera również dyrektywę
`SSLVerifyClient` . Poniżej przykład:

    <Directory "/wordpress/wp-admin">
    ...
    SSLRenegBufferSize 262144
    ...
    SSLVerifyClient optional
    ...
    </Directory>

I to w zasadzie cała robota. Teraz już tylko wystarczy zresetować serwer Apache2 i możemy ponowić
żądanie publikacji wpisu.
