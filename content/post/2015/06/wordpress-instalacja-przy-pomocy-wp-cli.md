---
author: Morfik
categories:
- Blog
date: "2015-06-02T14:43:22Z"
date_gmt: 2015-06-02 12:43:22 +0200
published: true
status: publish
tags:
- wordpress
- cli
- terminal
GHissueID: 74
title: 'WordPress: Instalacja przy pomocy wp-cli'
---

[Ostatnio opisywałem skrypt wp-cli](/post/wordpress-wiersz-polecen-wp-cli/) , który
posiada ciekawe możliwości pod względem zarządzania instalacją i konfiguracją WordPressa. W tym
artykule postaram się przebrnąć przez ten proces wykorzystując jedynie powyższe narzędzie. Nie mam
zamiaru korzystać z przeglądarki i nie będę przy tym nawet potrzebował odwiedzać strony WordPressa w
celu pobrania jakichkolwiek plików. Wszystkie poniższe kroki zostaną przeprowadzone w terminalu na
serwerze i mam nadzieję, że uda mi się pobrać, zainstalować i przygotować WordPressa do pracy.

<!--more-->
## Pobieranie i instalowanie plików z wykorzystaniem wp-cli

Przechodzimy zatem do katalogu gdzie chcemy wgrać WordPressa. W moim przypadku do `/apache/blog/` --
to będzie katalog roboczy i to w nim będą dokonywane wszelkie zmiany. Zaczynamy zatem od pobrania
plików instalacyjnych:

    $ wp core download
    Downloading WordPress 4.2.2 (en_US)...
    Success: WordPress downloaded.

Następnie generujemy plik `wp-config.php`
    :

    $ wp core config --dbname=blog --dbuser=user --dbpass=pass --dbhost=localhost --dbprefix=wp_ --dbcharset=utf8 --locale=pl_PL
    Success: Generated wp-config.php file.

Powyższe parametry oznaczają kolejno:

  - `--dbname` -- nazwa bazy
  - `--dbuser` -- użytkownik bazy
  - `--dbpass` -- hasło do bazy
  - `--dbhost` -- adres hosta bazy, domyślnie `localhost`
  - `--dbprefix` -- prefix do tabel w bazie
  - `--dbcharset` -- kodowanie bazy danych
  - `--locale` -- ustawia stałą `WPLANG`

Po dostosowaniu tych powyższych opcji, instalujemy
    WordPressa:

    $ wp core install --url=https://blog.lh --title=morfiblog --admin_user=morfik --admin_password=pass --admin_email=morfik@nsa.com
    Success: WordPress installed successfully.

Powyższe parametry oznaczają kolejno:

  - `--url` -- adres strony
  - `--title` -- nazwę bloga
  - `--admin_user` -- nazwę administratora serwisu
  - `--admin_password` -- hasło administratora
  - `--admin_email` -- email administratora

Domyślnie pobierana i instalowana jest wersja angielska `en_US` , możemy także wyszukać jeżyki,
które chcemy dołączyć do instalacji (wynik przycięty dla czytelności):

    $ wp core language list
    +----------+-------------------------+-------------------------+-------------+--------+---------------------+
    | language | english_name            | native_name             | status      | update | updated             |
    +----------+-------------------------+-------------------------+-------------+--------+---------------------+
    | pl_PL    | Polish                  | Polski                  | uninstalled | none   | 2015-05-09 10:15:05 |
    +----------+-------------------------+-------------------------+-------------+--------+---------------------+

By teraz pobrać język polski, wklepujemy w terminal poniższe polecenie:

    $ wp core language install pl_PL
    Success: Language installed.

    $ wp core language activate pl_PL
    Success: Language activated.

Po tych powyższych krokach powinniśmy mieć działającą instalację WordPressa.

Będąc w posiadaniu zainstalowanego skryptu WordPressa, pamiętajmy by raz na jakiś czas sprawdzić
jego sumy kontrolne przy pomocy:

    $ wp core verify-checksums
    Warning: File doesn't verify against checksum: wp-cron.php
    Error: WordPress install doesn't verify against checksums.

Wykryje ono wszelkie zmiany w plikach instalacyjnych dając nam tym samym znak, że ktoś mógł w nich
coś namieszać.

Jeśli chcielibyśmy dostosować jakieś domyślne opcje, możemy to również zrobić z poziomu konsoli,
choć chyba czasem jednak wygodniej jest z poziomu panelu www. W każdym razie, przydałoby się
wylistować pierw konfigurację WordPressa:

    $ wp option list
    +------------------------------------------------+---------------------------------------------+
    | option_name                                    | option_value                                |
    +------------------------------------------------+---------------------------------------------+
    | siteurl                                        | https://blog.lh                             |
    | home                                           | https://blog.lh                             |
    | blogname                                       | morfiblog                                   |
    | blogdescription                                | Just another WordPress site                 |
    | users_can_register                             | 0                                           |
    | admin_email                                    | morfik@nsa.com                              |
    | start_of_week                                  | 1                                           |
    | use_balanceTags                                | 0                                           |
    | use_smilies                                    | 1                                           |
    ...

Załóżmy, że nie odpowiada nam `blogdescription` (opis bloga) i chcielibyśmy go zmienić. W tym celu
wydajemy poniższe polecenie:

    $ wp option update blogdescription "Blog Morfika"
    Success: Updated 'blogdescription' option.

    $ wp option get blogdescription
    Blog Morfika

Podobnie możemy postąpić w przypadku każdej innej opcji, a jest ich `106` , przynajmniej w
podstawowej instalacji WordPressa.

### Wgrywanie i aktualizacja wtyczek i motywów

Mając już zainstalowanego i wstępnie skonfigurowanego bloga, przechodzimy do instalacji jakiegoś
motywu. Najpierw jednak sprawdźmy co mamy aktualnie do dyspozycji:

    $ wp theme status
    3 installed themes:
      A twentyfifteen  1.2
      I twentyfourteen 1.4
      I twentythirteen 1.5

    Legend: A = Active, I = Inactive

    $ wp plugin status
    2 installed plugins:
      I akismet 3.1.1
      I hello   1.6

    Legend: I = Inactive

Niewiele tego ale mamy również informację, że literka `A` wskazuje na to, który z zainstalowanych
motywów jest aktualnie używany. Doinstalujmy zatem coś bardziej wypasionego. Pierw jednak musimy
przeszukać repo WordPressa, wpisujemy:

    $ wp theme search bootstrap
    Success: Showing 10 of 213 themes.
    +---------------------+---------------------+--------+
    | name                | slug                | rating |
    +---------------------+---------------------+--------+
    | Bootstrap Canvas WP | bootstrap-canvas-wp | 100    |
    | Flat Bootstrap      | flat-bootstrap      | 100    |
    | Bootstrap Basic     | bootstrap-basic     | 100    |
    | Bootstrap Ultimate  | bootstrap-ultimate  | 96     |
    | DevDmBootstrap3     | devdmbootstrap3     | 100    |
    | The Bootstrap       | the-bootstrap       | 94     |
    | habakiri            | habakiri            | 0      |
    | Pixel-Linear        | pixel-linear        | 0      |
    | ZackLive            | zacklive            | 0      |
    | Relic               | relic               | 100    |
    +---------------------+---------------------+--------+

Gdybyśmy chcieli więcej wyników, dodajemy parametr `--per-page=20` . Jeśli chodzi zaś o instalację,
to nazwa, którą musimy wskazać wp-cli znajduje się w kolumnie `slug` , przykładowo, by za instalować
pierwszy z wyżej widocznych motywów, wydajemy poniższe polecenie:

    $ wp theme install bootstrap-canvas-wp
    Installing Bootstrap Canvas WP (1.92)
    Pobieranie paczki instalacyjnej z https://downloads.wordpress.org/theme/bootstrap-canvas-wp.1.92.zip...
    Rozpakowywanie paczki...
    Instalacja motywu...
    Motyw został zainstalowany.

Podobnie postępujemy z pluginami:

    $ wp plugin search yoast
    Success: Showing 10 of 232 plugins.
    +-------------------------------------------------------------------+---------------------------------------+--------+
    | name                                                              | slug                                  | rating |
    +-------------------------------------------------------------------+---------------------------------------+--------+
    | WordPress SEO by Yoast                                            | wordpress-seo                         | 92     |
    | Google Analytics by Yoast                                         | google-analytics-for-wordpress        | 82     |
    | Surbma - Yoast Breadcrumb Shortcode                               | surbma-yoast-breadcrumb-shortcode     | 60     |
    | Integration of Yoast wordpress SEO module with mqtranslate module | wp-seo-yoast-integration-mq-translate | 86     |
    | Yoast SEO fix for qTranslate                                      | yoast-seo-fix-for-qtranslate          | 84     |
    | Clicky by Yoast                                                   | clicky                                | 98     |
    | IsSiY (Include second Sitemap into Yoast) Wordpress SEO Extension | issiy-for-yoast                       | 0      |
    | SEO Extended                                                      | seo-extended                          | 94     |
    | WP SEO HTML Sitemap                                               | wp-seo-html-sitemap                   | 100    |
    | NS Custom Fields for WordPress SEO                                | ns-seo-custom-fields                  | 74     |
    +-------------------------------------------------------------------+---------------------------------------+--------+

    $ wp plugin install wordpress-seo
    Installing WordPress SEO by Yoast (2.1.1)
    Pobieranie paczki instalacyjnej z https://downloads.wordpress.org/plugin/wordpress-seo.2.1.1.zip...
    Rozpakowywanie paczki...
    Instalacja wtyczki...
    Wtyczka została zainstalowana.

Sama instalacja motywu czy pluginu to nie wszystko, trzeba także jeszcze dokonać ich aktywacji:

    $ wp plugin activate wordpress-seo
    Success: Plugin 'wordpress-seo' activated.

    $ wp theme activate bootstrap-canvas-wp
    Success: Switched to 'Bootstrap Canvas WP' theme.

Z kolei by wyłączyć dodatek, posługujemy się opcją `deactivate` , a jeśli zajdzie potrzeba, to przy
pomocy opcji `delete` możemy go usunąć. W ten sposób możemy łatwo zarządzać motywami czy pluginami i
to bez potrzeby zaciągania przeglądarki.

### Aktualizacja WordPressa

Innym aspektem pracy wp-cli jest aktualizacja rdzennego kodu, pluginów i motywów WordPressa. Kiedyś
pisałem, że w pewnych warunkach mogą wystąpić problemy, np. z automatyczną aktualizacją kodu i było
to spowodowane nieodpowiednimi uprawnieniami, tj. serwer www nie miał prawa zapisu plików w katalogu
z instalacją WordPressa. W przypadku wp-cli, nie ma potrzeby rozszerzać praw, bo działamy w
przestrzeni użytkownika, który ma przecież prawa zapisu do wszystkich znajdujących się tam plików,
także aktualizacja bez problemu może mieć miejsce, a by przypadkiem o niej nie zapomnieć, to
najlepiej jest zaprzęgnąć crona do pomocy i dorzucić mu do wykonywania co godzinę poniższe linijki:

    10    *    *     *     *     wp core update
    15    *    *     *     *     wp plugin update --all
    20    *    *     *     *     wp theme update --all

Te reguły zostaną wywołane z 5 minutowym opóźnieniem, tak by sobie nie wchodziły w drogę i zadbają o
to by nasz serwis był zawsze aktualny.

### Baza danych

Z ważniejszych rzeczy, które potrafi realizować skrypt wp-cli, to oczywiście wysyłanie zapytań do
bazy danych oraz importowanie i eksportowanie bazy. Jakiś czas temu [opisywałem jak za pomocą min.
wp-cli](/post/wordpress-zapomniane-haslo-administratora/) zresetować hasło do konta
administratora i w sumie więcej przykładów nie przychodzi mi do głowy z zastosowaniem tego skryptu,
bo na dobrą sprawę każde zapytanie SQL, które można wykonać bezpośrednio na bazie, można również
zrealizować przy pomocy wp-cli. Jeśli chodzi o bazy danych, to ja akurat wolę operować przy pomocy
narzędzia mysql-workbench i raczej nie prędko się to zmieni, zwłaszcza, że baza danych mojego
serwera nie jest wystawiona na widok publiczny i by się do niej dostać trzeba logować się pierw do
systemu [przy pomocy kluczy SSH](/post/uwierzytelniajace-klucze-ssh/).

Jeśli chodzi natomiast o import/eksport bazy danych, to mamy spore ułatwienie i nie musimy zaprzęgać
do tego żadnych dodatkowych pluginów i to jeszcze bezpośrednio podczepionych pod instalację
WordPressa. Możemy zaprogramować automatyczny backup bazy danych, np. co 12 godzin przy pomocy
crona:

    0      */12       *       *       *       /bin/sh -c "/usr/local/bin/wp db export ~/backup/baza-$(date +'\%Y-\%m-\%d-\%H-\%M-\%S').sql"

W ten sposób będą produkowane pliki `baza-2015-06-02-15-10-01.sql` , które będzie można przesłać
zdalnie gdzieś poza serwer.

Powyższe czynności należą jedynie do tych podstawowych i na dobrą sprawę, wszystko to co możemy
zrobić w panelu admina, idzie osiągnąć przez skrypt wp-cli. Z tą tylko różnicą, że jeśli
przeprowadzamy jakieś akcje z poziomu przeglądarki, to prawa potrzebne, np. do zapisu plików, trzeba
nadawać również serwerowi, w przeciwnym wypadku będą problemy. Dlatego też, jako zwolennik
minimalnych praw dostępu, jestem za przeprowadzaniem wszystkich czynności administracyjnych przy
pomocy tego narzędzia. Dobrze jest stworzyć sobie plik konfiguracyjny `~/.wp-cli/config.yml`, na
wzór tego poniższego:

    path: /apache/blog

    core config:
            dbuser: user
            dbpass: pass

Oczywiście, w tym pliku można zdefiniować dowolne opcje, które mogą być wklepane w terminalu,
wliczając w to również i te bardziej precyzyjne polecenia.
