---
author: Morfik
categories:
- Blog
date: "2016-07-29T13:08:24Z"
date_gmt: 2016-07-29 11:08:24 +0200
published: true
status: publish
tags:
- wordpress
- cache
- css
- js
GHissueID: 399
title: 'WordPress: Wersja plików .css/.js na blogu'
---

Gdy odwiedzamy jakiś blog WordPress'a po raz pierwszy, szereg jego elementów jest buforowanych w
cache przeglądarki. W ten sposób pewne pliki, np. `.css`, `.js` czy też obrazki, nie są pobierane
bezpośrednio z serwera www, bo mamy je lokalnie u siebie na dysku. Takie rozwiązanie zapewnia
szybsze załadowanie się strony przez minimalizowanie ruchu sieciowego. Niemniej jednak, jako że te
pliki siedzą w cache, to muszą mieć ustawiony pewien czas ważności. Może on być różny, a my możemy
go sobie dostosować dla poszczególnych elementów ustawiając im [nagłówek Cache-Control, Expires,
Last-Modified, czy
ETag](/post/cache-control-last-modified-etag-i-expires-w-apache2/). Gdy w takim
nagłówku określimy wysoką wartość `max-age` , przeglądarka klienta może przez bardzo długi czas nie
być świadoma faktu, że któreś elementy strony uległy zmianie. W efekcie może i pojawiła się nowa
wersja pliku `.css` ale klienci odwiedzający nasz serwis i tak nie zaobserwują żadnej różnicy do
momentu wygaśnięcia cache lub też odświeżenia strony z przytrzymanym klawiszem Shift . Możemy jednak
dodać numer wersji do określonych plików i uzależnić go od czasu modyfikacji danego zasobu na
serwerze. Jeśli zmianie ulegnie plik, klient automatycznie pobierze zmodyfikowany zasób.

<!--more-->
## Wersja zasobów bloga

Wersja plików `.css` czy `.js` może zostać określona przez doklejenie parametru do linku zasobu.
Zatem jeśli przykładowy styl CSS znajduje się pod `https://morfitronik.lh/css/style.css` , to nowy
adres będzie w formie `https://morfitronik.lh/css/style.css?ver=jakas_fraza` . Wszystko co jest
określone po `?` , to [ciąg znaków zapytania](https://en.wikipedia.org/wiki/Query_string). Można
sobie tutaj określić dowolne parametry w postaci `klucz=wartość` . Naszym parametrem będzie `ver` i
trzeba mu dynamicznie przypisać wartość w oparciu o czas modyfikacji pliku na serwerze.

Taki zabieg w WordPress'ie możemy przeprowadzić edytując plik `functions.php` , który znajduje się
zwykle w katalogu z motywem bloga. Weźmy sobie przykład standardowego pliku `style.css` , który
znajduje się w każdym motywie. W przypadku motywu wykorzystywanego na tym blogu, w pliku
`themes/nazwa_motywu/functions.php` mam poniższy kod odpowiadający za załadowanie się tego stylu:

    wp_register_style( 'theme-style', get_template_directory_uri() . '/style.css' );
    wp_enqueue_style( 'theme-style' );

Funkcje
[wp\_enqueue\_style()](https://developer.wordpress.org/reference/functions/wp_enqueue_style/) i
[wp\_enqueue\_script()](https://developer.wordpress.org/reference/functions/wp_enqueue_script/)
obsługują parametr `$ver` , który standardowo jest ustawiony na `false` . Powoduje to dodanie do
adresu URL odnoszącego się do zasobów, w tym przypadku `style.css` , końcówki `?ver=4.5.3'` . Czemu
4.5.3? Jest to wersja wykorzystywanego WordPress'a. Wszystkie dołączane pliki `.css` i `.js` mają tę
wersję uwzględnioną. Jeśli sobie życzymy, to ta [wersja WordPress'a widoczna wyżej może zostać
usunięta](/post/wordpress-wersja-obecna-w-kodzie-zrodlowym/). Niemniej jednak,
jest to stała wersja i zmiana zawartości pliku nie wymusi zmiany tego numerka. Dlatego też musimy
ten powyższy kod wykomentaować, a na jego miejsce dodać ten poniższy:

    function css_versioning() {
        wp_enqueue_style( 'style',
            get_stylesheet_directory_uri() . '/style.css' ,
            false,
            filemtime( get_stylesheet_directory() . '/style.css' ),
            'all' );
    }
    add_action( 'wp_enqueue_scripts', 'css_versioning' );

Cała magia tutaj sprowadza się do uzyskania przez `filemtime` daty modyfikacji pliku `style.css` i
zwrócenia jej w formie `style.css?ver=1469778354` . Ta duża liczba to [czas
unixowy](https://pl.wikipedia.org/wiki/Czas_uniksowy), który za każdym razem będzie inny. Wystarczy
teraz zapisać plik stylów, a przekonamy się, że ta liczba uległa zmianie, np. podglądając źródło
strony.
