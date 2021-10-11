---
author: Morfik
categories:
- Blog
date: "2015-05-30T17:59:03Z"
date_gmt: 2015-05-30 15:59:03 +0200
published: true
status: publish
tags:
- gettext
- wordpress
- język
GHissueID: 240
title: 'WordPress: Domyślny język instalacji'
---

WordPress, jako jeden z popularniejszych CMSów, [został przetłumaczony na wiele
języków](https://make.wordpress.org/polyglots/teams/), w tym też i na
[polski](https://pl.wordpress.org/). Czasem jednak przez zwykłe roztargnienie (lub też i z innych
przyczyn) możemy nie przestawić opcji odpowiedzialnej za język, wobec czego zainstaluje nam się
angielska wersja WordPressa. W tym krótkim wpisie zostanie zawartych kilka informacji na temat tego
jak zmienić język, na wypadek gdybyśmy odziedziczyli angielską wersję skryptu oraz też jak przebiega
sam proces lokalizacji tekstu w WordPressie i co się tak naprawdę dzieje po wybraniu innego języka.

<!--more-->
## Gdzie ustawić język?

Przede wszystkim, ciężko jest przeoczyć możliwość ustawienia języka swojego bloga, no bo po wgraniu
odpowiednich plików na serwer i wywołaniu w przeglądarce adresu instalacji, zostanie nam wyrzucone
to poniższe okienko:

![](/img/2015/05/1.wordpress-wybor-jezyk.png#small)

Jest tylko jedno "ale". W przypadku gdy prawa do plików nie będą takie jak powinny być, tj. serwer
nie będzie miał możliwości zapisu pewnych plików/katalogów, to powyższe okienko się nam nie pojawi i
WordPress od razu przejdzie do ustawiania parametrów samej instalacji i to w języku angielskim. W
niektórych wypadkach nie będziemy mieli możliwości nadania odpowiednich uprawnień (albo będzie to
wielce niewskazane), wobec czego by mieć możliwość przeprowadzenia instalacji w swoim natywnym
języku, będziemy musieli kilka rzeczy ustawić ręcznie.

Pliki językowe WordPressa są trzymane w katalogu `wp-content/languages/` . Standardowo jednak ten
folder nie istnieje i musimy go utworzyć. Zakładając, że dopiero co wrzuciliśmy pliki na serwer,
przechodzimy do folderu z instalacją WordPressa i tworzymy wspomniany katalog oraz nadajemy mu
odpowiednie uprawnienia (wymagane prawa roota):

    $ cd /apache/blog/
    $ mkdir wp-content/languages/
    # chown morfik:www-data wp-content/languages/

Zmieniamy katalog roboczy na ten nowo utworzony i pobieramy [paczkę
językową](https://make.wordpress.org/polyglots/teams/?locale=pl_PL) , po czym ją wypakowujemy.
Pamiętajmy tylko by odpowiednio dostosować numerek wersji:

    $ cd wp-content/languages/
    $ wget https://downloads.wordpress.org/translation/core/4.2/pl_PL.zip
    $ unzip pl_PL.zip
    Archive:  pl_PL.zip
      inflating: admin-network-pl_PL.po
      inflating: admin-pl_PL.po
      inflating: continents-cities-pl_PL.po
      inflating: pl_PL.po
      inflating: admin-network-pl_PL.mo
      inflating: admin-pl_PL.mo
      inflating: continents-cities-pl_PL.mo
      inflating: pl_PL.mo

    # chown morfik:www-data ./*

## Zmiana języka

Zmieniamy nazwę pliku `wp-config-sample.php` na `wp-config.php` i uzupełniamy pozycję od bazy
danych. Dodatkowo, dopisujemy w tym pliku poniższą linijkę:

    define('WPLANG', 'pl_PL');

Jeśli w tej chwili odpalimy instalację WordPressa w przeglądarce, naszym oczom powinny się ukazać
instrukcje w języku polskim:

![](/img/2015/05/2.wordpress-instalacja-po-polsku.png#huge)

Po zakończeniu instalacji, dobrze jest przy pomocy panelu admina (Ustawienia => Ogólne) zmienić
język na polski (po prostu kliknąć w przycisk `Zapisz zmiany`). A to z tego względu, że bez tego
kroku nie zostanie utworzona odpowiednia opcja w bazie danych w tabeli `wp_options`. Nie wiem czy to
ma jakiś wpływ na pluginy albo motywy, czy w ogóle na resztę skryptu WordPressa ale lepiej dmuchać
na zimne.
