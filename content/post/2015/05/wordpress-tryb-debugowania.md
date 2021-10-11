---
author: Morfik
categories:
- Blog
date: "2015-05-27T12:08:17Z"
date_gmt: 2015-05-27 10:08:17 +0200
published: true
status: publish
tags:
- wordpress
- debug
GHissueID: 229
title: 'WordPress: Tryb debugowania'
---

Gdy wprowadzamy sporo zmian na swoim blogu i nie mam tutaj na myśli dodawania/edytowania/usuwania
postów, a jedynie przeróbki dotyczące zmiany wyglądu motywu czy też instalowanie i konfigurowanie
nowych wtyczek, to istnieje spore prawdopodobieństwo, że gdzieś wystąpi jakiś błąd, który może
zagrażać bezpieczeństwu całej witryny. Dlatego też warto wiedzieć, że WordPress [posiada obsługę
trybu debug](https://codex.wordpress.org/Debugging_in_WordPress), który może nam te wszystkie
problemy z działaniem serwisu uwidocznić. Nie jest on jednak standardowo włączony, z oczywistych
względów ale jeśli zajdzie taka potrzeba, to możemy sobie ten tryb debugowania aktywować i sprawdzić
czy są jakieś błędy, ostrzeżenia czy też inne dodatkowe informacje, które mogą nam w jakiś sposób
pomóc.

<!--more-->
## Aktywacja trybu debug

Jeśli mamy przeczucie, że coś dolega naszemu serwisowi ale nie mamy żadnych wizualnych komunikatów
by potwierdzić te przypuszczenia, to nie wahajmy się i przełączmy się jak najszybciej w tryb
debugowania. Odradzam jednak aktywowanie tej opcji w przypadku publicznie dostępnych stron www, z
tym, że nie zawsze będziemy mieli możliwość postawienia swojego własnego środowiska testowego, gdzie
będziemy mogli przeprowadzić wszystkie te niezbędne eksperymenty i czasem możemy zwyczajnie nie mieć
innego wyjścia jak uruchomić tryb debug bezpośrednio na żywym organizmie.

Wszystkie poniższe parametry muszą pojawić się **PRZED** wystąpieniem `/* That's all, stop editing!
Happy blogging. */` w pliku `wp-config.php` .

Tryb debugowania w WordPressie aktywujemy za pośrednictwem pliku `wp-config.php` i mamy kilka opcji
do wyboru. Najprościej dopisać w tym pliku poniższy kod:

    // Enable WP_DEBUG mode
    define( 'WP_DEBUG', true );

Odpowiada on za zwracanie komunikatów błędów, ostrzeżeń i innych dodatkowych informacji. Ponadto, za
jego pośrednictwem, mogą zostać aktywowane także dwie dodatkowe opcje, tj. `WP_DEBUG_DISPLAY` oraz
`WP_DEBUG_LOG` , z których pierwszy odpowiada za pokazywanie komunikatów bezpośrednio na stronie
bloga, drugi zaś loguje je do pliku `wp-content/debug.log`. Domyślnie jednak tylko
`WP_DEBUG_DISPLAY` zostanie włączony. Jeśli chcemy dostosować sobie odpowiednio te opcje, to zawsze
możemy je dopisać do pliku `wp-config.php` :

    // Enable display of errors and warnings
    define( 'WP_DEBUG_DISPLAY', true );
    @ini_set('display_errors',1);

    // Enable Debug logging to the wp-content/debug.log file
    define('WP_DEBUG_LOG', true);

Jeśli już musimy włączać tryb debugowania w środowisku produkcyjnym, to rozważmy chociaż aktywowanie
jedynie tej drugiej opcji. Zadbajmy także o to, by odpowiednio ukryć sam plik logu, bo standardowo
jest on dostępny publicznie i każdy będzie mógł go sobie przejrzeć wywołując, np. w przeglądarce,
adres z doczepioną ścieżką `wp-content/debug.log` .

Pamiętajmy też, że nie wszystkie komunikaty muszą od razu oznaczać, że z serwisem dzieje się coś
niedobrego, bo przecież WordPress podlega nieustannym zmianom i wprowadzane są nowe (oraz
przerabiane stare) rozwiązania. Dlatego też, sporo komunikatów może dotyczyć przestarzałego kodu,
który w późniejszych wersjach WordPressa zostanie usunięty na dobre i jeśli tego typu informację
ujrzymy na swoim blogu, to prawdopodobnie będziemy musieli uaktualnić używany theme bądź też jakiś
plugin ale w tym konkretnym momencie wszystko będzie działać jak trzeba.

Jeśli planujemy modyfikować szereg wbudowanych w WordPress skryptów JS lub stylów CSS, to przyda nam
się również opcja debugowania tego typu kodu. Odpowiada za nią inny parametr, bo przecie zwykle
administratorzy nie ingerują w rdzenny kod WordPressa. Tak czy inaczej, by mieć możliwość
sprawdzenia czy wszystko i w tej kwestii jest w porządku, dodajmy do pliku `wp-config.php` poniższy
kod:

    // Use dev versions of core JS and CSS files (only needed if you are modifying these core files)
    define( 'SCRIPT_DEBUG', true );

Dzięki niemu zostaną załadowane pełne wersje skryptów i stylów z katalogów `wp-includes/js` ,
`wp-includes/css` oraz `wp-admin/js` , a nie ich minimalistyczne wersje, czyli te z `.min.` w
nazwie.

## Debugowanie zapytań bazy danych

Jak widzimy wyżej, jeśli chodzi o debugowanie błędów kodu WordPressa, to mamy spore możliwości.
Warto też wiedzieć, że istnieje opcja podejrzenia zapytań kierowanych do bazy danych, tj. tego jak
skrypt WordPressa rozmawia bezpośrednio z bazą. Możemy w ten sposób prześledzić ile zapytań zostało
wysłanych podczas ładowania się strony, oraz treść każdego z nich. Jeśli chcemy obejrzeć sobie te
zapytania, to dodajemy do pliku `wp-config.php` poniższy kod:

    // Save the database queries
    define( 'SAVEQUERIES', true );

Zapisane w ten sposób zapytania będą przechowywane w tablicy `$wpdb->queries` ale by zostały one nam
jeszcze pokazane, to musimy w stopce używanego przez naszą witrynę motywu dodać poniższą zwrotkę:

    <?php
    if ( current_user_can( 'administrator' ) ) {
        global $wpdb;
        echo "<pre>";
        print_r( $wpdb->queries );
        echo "</pre>";
    }
    ?>

Jak widać wyżej, mamy tam warunek, by te zapytania zostały pokazane jedynie użytkownikom, którzy
mają status administratora witryny. Poza tym, lepiej nie włączać tej opcji jeśli jej nie
potrzebujemy, bo dość poważnie spowolni działanie całego serwisu.
