---
author: Morfik
categories:
- Blog
date: "2015-05-25T15:03:42Z"
date_gmt: 2015-05-25 13:03:42 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- wordpress
GHissueID: 239
title: 'WordPress: Zmiana domyślnych nazw folderów'
---

Internet to bardzo niebezpieczne miejsce i do tego pełne różnego rodzaju botów szukających
ujawnionych już podatności i niekoniecznie rozchodzi się o kod WordPressa. Tak czy inaczej,
WodrPress to bardzo popularny CMS, którego instalacja przebiega (i wygląda) praktycznie tak samo i
to bez znaczenia gdzie nie zostałby on zainstalowany. Chodzi o to, że zawsze mamy do czynienia z
tymi samymi skryptami czy nazwami katalogów. Nie zmieniają się także ścieżki to plików, no i
oczywiście pluginy, których większa ilość może zagrozić bezpieczeństwu naszej strony. Nie chodzi mi
tutaj o pilnowanie instalacji i ciągłe aktualizowanie kodu, bo to rozumie się samo przez się. Chodzi
mi generalnie o [zmodyfikowanie nieco samej instalacji][1] WordPress'a, tak by nie przypominała
wszystkich pozostałych. W ten sposób możemy się uchronić przed zautomatyzowanymi robotami (i innym
syfem), które są w stanie podrzucić coś nam na stronę o ile ta ma domyślne ścieżki. Im wcześniej
zabierzemy się za zmianę domyślnych nazw katalogów, tym lepiej, bo jeśli w przyszłości będziemy
zmieniać te nazwy i to na blogu czy w serwisie gdzie mamy sporo kontentu w postaci grafiki czy innej
załączonej treści, wtedy trzeba będzie te linki przepisać, bo przestaną zwyczajnie działać.

<!--more-->
## Przenoszenie folderu wp-content

Jednym z takich krytycznych katalogów jest `wp-content` . To tam przechowywane są wszystkie wtyczki,
motywy czy wgrywane przez nas pliki, które później zamieszczamy w postach. Mając na uwadze powyższe,
przydałoby się zmienić nazwę (i ewentualną lokalizację) tego katalogu na jakąś inną niż ta
standardowa.

Trzeba jednak pamiętać, że potencjalny atakujący może bez problemu uzyskać informacje na temat nowej
nazwy folderu i jego położenia . Wystarczy, że podejrzy źródło naszej strony.

Zmianę nazwy i położenia folderu `wp-content` dokonujemy przez dopisanie do pliku `wp-config.php`
poniższego kodu:

    define( 'WP_CONTENT_DIR', dirname(__FILE__) . '/kot/pies' );
    define( 'WP_CONTENT_URL', 'https://morfitronik.pl/kot/pies' );

W powyższym przykładzie, nowa nazwa katalogu to `pies` . Dodatkowo, został ten folder umieszczony w
katalogu `kot` . Nie zapomnijmy także o fizycznej zmianie nazwy/położenia samego katalogu.

## Przenoszenie folderu plugins

By nieco bardziej wyróżnić się z tłumu, zmienimy również i położenie katalogu `plugins` . Robimy to
w podobny sposób co wyżej, tj. do pliku `wp-config.php` dopisujemy poniższy kod:

    define( 'WP_PLUGIN_DIR', dirname(__FILE__) . '/kot/pies/szczur' );
    define( 'WP_PLUGIN_URL', 'https://morfitronik.lh/kot/pies/szczur' );
    define( 'PLUGINDIR', dirname(__FILE__) . '/kot/pies/szczur' );

Prawdopodobnie kilka z już zainstalowanych pluginów może mieć problemy z działaniem. Zwykle "same"
się doprowadzą do porządku po przejściu do ich opcji konfiguracyjnych. Jeśli jednak tak się nie
stanie, dobrym wyjściem jest reinstalacja danej wtyczki.

## Przenoszenie folderu uploads

To właśnie do tego katalogu trafiają wszelkie obrazki i inne pliki, które przesyłamy do WordPressa.
Podobnie jak wyżej, by zmienić nazwę/położenie katalogu `uploads` dodajemy poniższy kod do pliku
`wp-config.php` :

    define( 'UPLOADS', 'kot/pies/krowa' );

Przy czym tutaj mała uwaga. Ścieżka, widoczna wyżej, nie może się zaczynać od `/` .

## Przenoszenie folderu themes

Ostatnim folderem, którym przydałoby się zająć to `themes` , tylko jest jeden mały problem.
Mianowicie, jego nazwa jest wkodowana na sztywno i nie możemy sobie jej od tak przepisać, tak jak to
miało miejsce wyżej. Niby jest opcja by skorzystać z funkcji `register_theme_directory` , choć
obecnie nie mam zielonego pojęcia jak to ugryźć.


[1]: https://codex.wordpress.org/Editing_wp-config.php#Moving_wp-content_folder
