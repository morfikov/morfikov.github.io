---
author: Morfik
categories:
- WordPress
date: "2016-02-15T15:17:56Z"
date_gmt: 2016-02-15 14:17:56 +0100
published: true
status: publish
tags:
- blog
GHissueID: 511
title: 'WordPress: Zmiana serwera i nazwy bloga'
---

Prędzej czy później przyjdzie taki czas, że będziemy musieli porzucić stary hosting, na którym
trzymamy swój blog WordPress'a. O ile samo przeniesienie całego kontentu nie powinno sprawić
trudności, bo to przecież zwykłe kopiowanie plików, to zmiana struktury strony, np. katalogu
głównego lub też nazwy bloga (domeny), [pociąga już za sobą pewne
komplikacje](https://codex.wordpress.org/Moving_WordPress). W tych bardziej zaawansowanych
przypadkach, zwłaszcza jeśli posiadamy niestandardową konfigurację, będzie potrzebna edycja
niektórych plików WordPress'a oraz trzeba będzie zmienić szereg wpisów w bazie danych. W tym
artykule zostanie opisany proces przenoszenia instalacji WordPress'a na nowy serwer wliczając w to
zmianę nazwy serwisu.

<!--more-->
## Backup obecnej instalacji WordPress'a

Zanim zaczniemy cokolwiek robić, musimy utworzyć backup obecnej instalacji WordPress'a. To na
wypadek, gdy coś pójdzie nie tak. Przechodzimy zatem na serwer i pakujemy katalog główny
WordPress'a:

    # tar -czpf morfitronix.tar.gz morfitronix/

Logujemy się również do bazy danych i robimy jej pełny zrzut do pliku. Możemy to zrobić za pomocą
[phpMyAdmin](https://www.phpmyadmin.net/), [WP-CLI](http://wp-cli.org/), czy też [MySQL
Workbench](https://www.mysql.com/products/workbench/). My skorzystamy z `wp-cli` . By przy jego
pomocy wykonać kopię bazy danych, potrzebny będzie nam plik konfiguracyjny `~/.wp-cli/config.yml` o
poniższej treści:

    path: /apache/morfitronix
    
    core config:
            dbuser: user
            dbpass: pass

Zapisujemy plik i wydajemy w terminalu to poniższe polecenie:

    $ wp db export baza-$(date +'%Y-%m-%d-%H-%M-%S').sql

Zatem mamy przygotowany backup bazy danych i katalogu z instalacja WordPress'a. Możemy przejść do
przeniesienia bloga na inny serwer.

## Zmiana serwera

W przypadku, gdy domena, nazwa bazy danych czy też struktura katalogów instalacji WordPress'a nie
ulega zmianie, to w zasadzie nie mamy za dużo do roboty. Ważne jest tylko, by upewnić się, że
uprawnienia do plików zgadzają się, w szczególności UID i GID. Być może będziemy potrzebować również
dostosować konfigurację serwera WWW. Niemniej jednak, samo przeniesienie plików powinno wystarczyć.

## Zmiana nazwy bloga (domeny)

Zmiana nazwy bloga zawsze powoduje problemy, bo ta nazwa jest uwzględniona w kilku miejscach. [Jest
też parę metod zmiany adresu strony](https://codex.wordpress.org/Changing_The_Site_URL). Możemy to
zrobić albo za pomocą pliku `wp-config.php` , albo też wykonać proste zapytanie do bazy danych, np.
za pomocą narzędzia `wp-cli` . W przypadku wykorzystania `wp-cli` , wystarczy, że w terminalu
wpiszemy sobie to poniższe polecenie:

    $ wp search-replace "https://morfitronik.lh" "https://morfitronik.pl" wp_options --dry-run

Następnie edytujemy plik `wp-config.php` . Na wypadek, gdyby ktoś zapomniał, znajduje się on w
głównym katalogu instalacji bloga. W pliku szukamy zmiennych `WP_CONTENT_URL` oraz `WP_PLUGIN_URL`
i dostosowujemy ich wartości pod kątem nowego adresu:

    define( 'WP_CONTENT_URL', 'https://morfitronik.lh/wp-content' );
    define( 'WP_PLUGIN_URL', 'https://morfitronik.lh/wp-content/uploads' );

Musimy jeszcze dokonać paru zmian w kwestii linków wewnętrznych naszego serwisu. Wszystkie linki,
które opublikowaliśmy w postach na blogu, będą odwoływać się do starej domeny. Nie możemy jednak
tak po prostu przeszukać bazy i zastąpić stare adresy nowymi. Doprowadziłoby to do problemów z
[serializacją danych](https://pl.wikipedia.org/wiki/Serializacja). By tego uniknąć, musimy ponownie
zaprzęgnąć do pracy `wp-cli` . Ważne jest, by zmienić jedynie tabelę `wp_posts` :

    $ wp search-replace "https://morfitronik.pl" "https://morfitronik.pl" wp_posts --dry-run

Parametr `--dry-run` sprawi, że żadne zmiany nie zostaną poczynione. Będziemy mieli jednak wgląd w
to, co tak naprawdę może zostać zmienione. W przypadku braku błędów, usuwamy parametr `--dry-run` i
ponawiamy polecenie. Powinniśmy zobaczyć komunikat: `Success: Made 797 replacements.` informujący
nas o powodzeniu wprowadzanych zmian.

Musimy także dostosować odpowiednio dyrektywy `VirtualHost` w konfiguracji Apache2 i jeśli
korzystamy z pliku `.htaccess` , to również trzeba i go przeszukać pod kątem wystąpienia starych
adresów.

## Zmiana nazwy katalogu instalacyjnego WordPress'a

Opcjonalnie możemy zmienić również nazwę katalogu z instalacją bloga WordPress'a. Zwykle w tym
przypadku będziemy musieli dostosować dyrektywę `VirtualHost` w Apache, tak by wskazywała na nowy
katalog na serwerze. Ponadto, jeśli korzystamy z WP-CLI, to musimy też dostosować jego plik
konfiguracyjny `~/.wp-cli/config.yml` , gdzie zmieniamy linijkę z `path:` . Jeśli adres strony
wykorzystywał nazwę starego katalogu WordPress'a, to również musimy przerobić ten adres (Settings \>
General). Podobnie sprawa ma się z permalink'ami (Settings \> Permalinks). Problemem może także
wystąpić w przypadku wszelkiego rodzaju odnośników do naszego bloga, np. obrazki. Tu mamy do
czynienia z takim samym problemem co w przypadku zmiany domeny bloga. I również tutaj musimy
skorzystać z `wp-cli` w celu zastąpienia odpowiednich nazw.

Trzeba też będzie przerobić ścieżki określonych wtyczek. Niekoniecznie musimy to robić ręcznie.
Zwykle rekonfiguracja takiego plugina z panelu administracyjnego powinna załatwić sprawę. Może w
niektórych przypadkach będzie trzeba wyłączyć i włączyć dany dodatek, by jego ustawienia dotyczące
ścieżek zostały zresetowane. Tak czy inaczej, poniżej jest przykład dostosowania wtyczki
`wp-super-cache` w pliku `wp-config.php` :

    define( 'WPCACHEHOME', '/apache/morfitronik/contentcisko/wtyki/wp-super-cache/' );

Trzeba pamiętać, że wszelkie zmiany dotyczące adresu strony odbiją się bardzo negatywnie na samym
serwisie. W przypadku, gdy ktoś linkuje do naszego bloga, to te łącza przestaną działać. Dlatego
też, trzeba zainteresować się kwestią przekierowań i napisać odpowiednie regułki, które te żądania
idące pod stare adresy przekierują automatycznie na nowe lokalizacje.

## Zmiana konfiguracji bazy danych WordPress'a

Ostatnia kwestia, która została do poruszenia, to zmiana konfiguracji bazy danych. W tym przypadku
zmianie może ulec użytkownik, hasło, prefiks do tabel czy też nazwa samej bazy danych. Wszystkie te
w/w informacje są konfigurowane w pliku `wp-config.php` , a konkretnie w tych zmiennych:

    define('DB_NAME', '');
    define('DB_USER', '');
    define('DB_PASSWORD', '');
    define('DB_HOST', '');
    $table_prefix  = '';
