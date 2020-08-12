---
author: Morfik
categories:
- WordPress
date: "2015-05-25T20:02:36Z"
date_gmt: 2015-05-25 18:02:36 +0200
published: true
status: publish
tags:
- blog
- baza-danych
title: 'WordPress: Jak zmienić prefiks bazy danych'
---

Każda instalacja tych popularniejszych (i wszystkich pozostałych) CMS'ów powinna być w miarę
niepowtarzalna, by utrudnić automatyzację ataków. Blog WordPress'a opiera się o bazę danych. Mamy
tam kilka tabel, z których każda jest poprzedzona pewnym prefiksem. Czy zmiana tego prefiksu `wp_`
na jakiś losowo wybrany przez nas ma jakiś sens? Czy ochroni nas to przed jakimiś formami ataku? W
tym wpisie postaramy się prześledzić proces zmiany prefiksu do tabel w bazie danych.

<!--more-->
## Dlaczego warto zmienić domyślny prefiks

Przede wszystkim, jeśli mamy do czynienia z bazami danych, to pierwsza rzecz, nad którą powinniśmy
się zastanowić, to oczywiście [SQL Injection](https://pl.wikipedia.org/wiki/SQL_injection) . W
skrócie polega to na przesyłaniu (np. w formularzach) części lub całych zapytań SQL, które po
niedostatecznym filtrowaniu mogą zostać przetworzone przez serwer. Pół biedy jeśli takie zapytanie
zwróci jedynie błąd. A co w przypadku gdy w ten sposób można będzie [wykasować, pobrać lub zmienić
dane](http://www.securitum.pl/baza-wiedzy/publikacje/sql-injection)?

WordPress raczej potrafi przeprowadzać walidację wysyłanych do serwera danych ale jest przy tym
pełno miejsc gdzie zabezpieczenia mogą nawalić. Może nam się trafić trefny theme, czy plugin, lub
też zwykły niezamierzony błąd ludzki (a są w ogóle takie?) i wtedy będziemy mieć problem. Niby na
stronie WordPressa [możemy wyczytać](https://codex.wordpress.org/Managing_Plugins), że te wszystkie
dodatki, które są im dostarczane, zostały przebadane pod kątem szkodliwości i można by założyć, że
skoro tam się znajdują, to nie powinny wyrządzić naszej witrynie żadnej szkody. Niemniej jednak, na
tej samej stronie możemy przecie dostrzec wiele przestarzałych pluginów, gdzie data ostatniej
aktualizacji wskazuje na kilka lat wstecz, a sam WordPress ogranicza się jedynie do umieszczenia
komunikatu o treści **This plugin hasn't been updated in over 2 years. It may no longer be
maintained or supported and may have compatibility issues when used with more recent versions of
WordPress.** Czemu taki plugin w ogóle jest jeszcze dostępny tam na stronie WordPressa? Tego nie
wiem ale na pewno nikt o zdrowych zmysłach nie pokusiłby się o jego instalację.

Także jak widać na powyższym przykładzie, różnie bywa i lepiej założyć, że jeśli coś może pójść źle,
to pójdzie. W przypadku baz danych przynajmniej większość (jak nie wszystkie) ataki są
przeprowadzane w oparciu o pewne informacje, min. nazwa bazy danych oraz prefiksy do tabel, w
których znajdują się dane. Obie te rzeczy są ustawiane domyślnie na `wordpress` oraz `wp_`
odpowiednio. W sumie nic nas to nie kosztuje, by te dwa parametry zmienić, to czemu by tego nie
zrobić?

## Zmiana prefiksu tabel podczas instalacji

Jak już wspomniałem wyżej, za każdym razem gdy instalujemy WordPressa, mamy możliwość wstępnej
konfiguracji, co wygląda mniej więcej tak jak na fotce poniżej:

![]({{< baseurl >}}/img/2015/05/1.instalacja-wordpressa.png)

Przepisujemy `Table Prefix` na coś dowolnego. Jeśli interesują nas względy estetyczne, to nie
zapomnijmy dodać na końcu prefiksu `_` , co oddzieli wprowadzoną frazę od nazwy tabel. Możemy także
zmienić nazwę bazy danych, by również nie była standardowa.

## Zmiana prefiksu tabel po instalacji

Jeśli podczas instalacji wybraliśmy standardowe ustawienia i teraz chcemy je zmienić, to musimy
trochę nad tym popracować. Przede wszystkim, edytujemy plik `wp-config.php` , bo to tam jest
definiowana część ustawień WordPressa, w tym też i informacje dotyczące bazy danych. Opcja, która
nas interesuje to `$table_prefix` :

    $table_prefix  = '666pw_';

Możemy także tutaj zmienić nazwę bazy danych:

    define('DB_NAME', 'litra_wyborowej_i_ogorki');

To jednak nie wszystko co musimy zrobić. Przede wszystkim musimy zmienić prefiksy tabel w bazie
danych. Możemy to sobie zwyczajnie wyklikać w jakimś graficznym SQL adminie, np. **mysql-workbench**
albo też wysłać zapytania do bazy w trybie tekstowym:

    ALTER TABLE `wordpress-test`.`wp_commentmeta` RENAME TO  `wordpress-test`.`666pw_commentmeta` ;
    ALTER TABLE `wordpress-test`.`wp_comments` RENAME TO  `wordpress-test`.`666pw_wp_comments` ;
    ALTER TABLE `wordpress-test`.`wp_links` RENAME TO  `wordpress-test`.`666pw_wp_links` ;
    ALTER TABLE `wordpress-test`.`wp_options` RENAME TO  `wordpress-test`.`666pw_wp_options` ;
    ALTER TABLE `wordpress-test`.`wp_postmeta` RENAME TO  `wordpress-test`.`666pw_wp_postmeta` ;
    ALTER TABLE `wordpress-test`.`wp_posts` RENAME TO  `wordpress-test`.`666pw_wp_posts` ;
    ALTER TABLE `wordpress-test`.`wp_term_relationships` RENAME TO  `wordpress-test`.`666pw_wp_term_relationships` ;
    ALTER TABLE `wordpress-test`.`wp_term_taxonomy` RENAME TO  `wordpress-test`.`666pw_wp_term_taxonomy` ;
    ALTER TABLE `wordpress-test`.`wp_terms` RENAME TO  `wordpress-test`.`666pw_wp_terms` ;
    ALTER TABLE `wordpress-test`.`wp_usermeta` RENAME TO  `wordpress-test`.`666pw_wp_usermeta` ;
    ALTER TABLE `wordpress-test`.`wp_users` RENAME TO  `wordpress-test`.`666pw_wp_users` ;

[Na necie spotkałem
się](http://www.wpbeginner.com/wp-tutorials/how-to-change-the-wordpress-database-prefix-to-improve-security/)
jeszcze z taką informacją, że musimy przepisać pewne wiersze w tabelach `666pw_options` oraz
`666pw_usermeta` , a to za sprawą kilku egzotycznych pluginów:

    SELECT * FROM `666pw_options` WHERE `option_name` LIKE '%wp_%'
    SELECT * FROM `666pw_usermeta` WHERE `meta_key` LIKE '%wp_%'

Powyższe linijki wyszukują frazę `wp_` w tych dwóch tabelach ale na świeżej instalacji WordPressa
nie było nic do przepisywania.
