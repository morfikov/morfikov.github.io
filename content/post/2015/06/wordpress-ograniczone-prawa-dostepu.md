---
author: Morfik
categories:
- Blog
date: "2015-06-06T11:41:24Z"
date_gmt: 2015-06-06 09:41:24 +0200
published: true
status: publish
tags:
- pliki
- foldery
- wordpress
- bezpieczeństwo
- baza-danych
GHissueID: 81
title: 'WordPress: Ograniczone prawa dostępu'
---

Mając już zainstalowanego i wstępnie skonfigurowanego WordPress'a, przydałoby się zadbać [prawa
dostępu do jego plików](https://codex.wordpress.org/Changing_File_Permissions) jak i również
ograniczyć nieco dostęp do samej bazy danych, tak by możliwie najmniejsze uprawnienia zostały
nadane, co poprawi znacznie bezpieczeństwo naszej witryny. Jeśli opanowaliśmy już operowanie na
instalacji WordPress'a przy pomocy
[skryptu](/post/wordpress-wiersz-polecen-wp-cli/)
[wp-cli](/post/wordpress-instalacja-przy-pomocy-wp-cli/) , to możemy bez wahania
dokręcić śrubę naszemu serwisowi. Jeśli nie zaznajomiliśmy się jeszcze z powyższym narzędziem, to
ograniczenie praw może spowodować problemy z aktualizacją rdzennych plików WordPress'a, jak i
instalacją/konfiguracją jego pluginów czy motywów.

<!--more-->
## Prawa do plików i katalogów

Jeśli instalowaliśmy, aktualizowaliśmy rdzenny kod oraz dodatki bez większych problemów z panelu
administracyjnego naszej strony, oznacza to prawdopodobnie, że właścicielem plików jest użytkownik,
z którego prawami działa serwer www, na debianie to jest `www-data`. Pierwsza rzecz, o którą musimy
się zatroszczyć to przepisanie praw na innego użytkownika. Do przeprowadzenia paru poniższych
czynności będą nam potrzebne prawa administratora systemu i jeśli takimi dysponujemy to
przechodzimy do katalogu gdzie mamy pliki WordPress'a i wpisujemy w terminal poniższe polecenia:

    $ cd /apache/
    # chown -R morfik:www-data ./blog

Grupa z kolei musi wskazywać na tę serwera www. Następnie upewniamy się, że każdy plik ma
odpowiednie uprawnienia korzystając z tych dwóch poniższych linijek:

    $ find ./blog -type d -exec chmod 750 {} \;
    $ find ./blog -type f -exec chmod 640 {} \;

Nadadzą one wszystkim plikom prawa zapisu/odczytu dla właściciela oraz odczytu dla grupy, czyli
serwera. Podobnie z katalogami, z tym, że tutaj również trzeba uwzględnić możliwość przejścia do
katalogu.

W skład instalacji WordPress'a wchodzi szereg plików i katalogów i nie wszystkie z nich będą
posiadać takie same uprawnienia. Dlatego też musimy nieco dostosować pozostałe pliki/katalogi
ręcznie.

Wszystkie pliki zlokalizowane w głównym katalogu WordPress'a ( `/` ) mają być zapisywane jedynie
przez właściciela. Natomiast muszą być także odczytywane przez serwer (grupę). Wyjątkami są dwa
pliki: `.htaccess` oraz `wp-config.php` . Jeśli chcemy aby wtyczki WordPress'a miały możliwość
dopisywania konfiguracji do `.htaccess` oraz `wp-config.php`, to musimy również nadać prawa zapisu
dla tych plików grupie. Ja jednak uważam, że po zakończeniu zabawy z instalacją WordPress'a,
powinniśmy nadać prawa `440` dla obu tych plików:

    $ chmod 440 .htaccess wp-config.php

Dalej w głównym katalogu mamy 3 foldery: `wp-content/` , `wp-admin/` oraz `wp-include/` . Do plików
znajdujących się w dwóch ostatnich katalogach, prawa zapisu ma mieć tylko właściciel. Jeśli chodzi
zaś o pliki w `wp-content/` , to w nim mamy kilka dodatkowych podkatalogów, które będą mieć inne
prawa w zależności od tego co tak naprawdę chcemy osiągnąć. Przede wszystkim mamy tam katalog
`themes/` i prawa zapisu do niego ma mieć tylko właściciel, z tym, że w takim przypadku stracimy
możliwość edycji plików motywów przy pomocy edytora w panelu administracyjnym. Choć nie powinniśmy
się tym przejmować, bo [ze względów bezpieczeństwa i tak powinniśmy mieć
wyłączony](/post/wordpress-edycja-i-modyfikacja-plikow-dodatkow/) ten edytor.
Jeśli chodzi o katalog `plugins/` , to również i w tym przypadku prawa do zapisu musi posiadać
jedynie właściciel. Trzeba jednak mieć na uwadze, że nie zainstalujemy/zaktualizujemy już pluginów z
poziomu panelu admina, przy najmniej nie z taką prostotą jak dotychczas:

![wordpress-problemy-przez-restrykcyjne-prawa](/img/2015/06/1.wordpress-problemy-przez-restrykcyjne-prawa.png#medium)

Po zmianie praw będziemy zmuszeni albo do korzystania z ftp/ftps, tak jak to widać wyżęj, albo z
`wp-cli` .

Jeśli chodzi o katalog `uploads/` , prawa zapisu trzeba nadać również dla serwera (grupy), bo
inaczej nie będziemy w stanie wgrywać plików, które będziemy załączać w publikowanych postach. Zatem
dla pewności, wydajmy poniższe polecenia:

    $ chmod -R g-w ./wp-content/plugins
    $ chmod -R g-w ./wp-content/themes
    $ chmod -R g+w ./wp-content/uploads

Jeśli posiadamy jakieś dodatkowe katalogi w `wp-content/` , trzeba ustalić jakich praw one wymagają
i jakie dodatki je tworzą. Przykładowo, jeśli mamy plugin, co odpowiada za cache stron www, to
prawdopodobnie stworzy on katalog `wp-content/cache/` i on musi być zapisywalny przez serwer, bo
inaczej generowane pliki nie będą tam umieszczane. Podobnie trzeba postąpić w przypadku pliku logu
zapisywanego w [trybie debugowania](/post/wordpress-tryb-debugowania/).

## Prawa do bazy danych

Pozostały nam jeszcze do omówienia przywileje użytkownika bazy danych. Przede wszystkim, nie
nadajemy wszystkich praw użytkownikowi, który zdefiniowany jest w `wp-config.php` . WordPress'a bez
problemu potrafi operować na bardziej restrykcyjnych prawach, w których skład wchodzą: `SELECT` ,
`INSERT` , `UPDATE` , `DELETE` . I na dobrą sprawę nie potrzeba niczego więcej by zamieszczać nowe
posty na blogu, dodawać użytkowników, wysyłać załączniki, dodawać komentarze pod wpisami, czy
instalować motywy/pluginy.

![wordpress-mysql-prawa-uzytkownika](/img/2015/06/2.wordpress-mysql-prawa-uzytkownika.png#big)

Jeśli ktoś woli tekstowego klienta, to poniżej są odpowiednie linijki:

    revoke all privileges on 'wp-blog'.* from 'wp-user'@'localhost';
    grant SELECT,INSERT,UPDATE,DELETE ON 'wp-blog'.* TO 'wp-user'@'localhost';
    FLUSH PRIVILEGES;

Sprawdzamy, czy aby wszystkie prawa zostały nadane pomyślnie:

    SHOW GRANTS FOR 'wp-user'@'localhost';
    +----------------------------------------------------------------------------------+
    | Grants for wp-user@localhost                                                     |
    +----------------------------------------------------------------------------------+
    | GRANT SELECT, INSERT, UPDATE, DELETE ON 'wp-blog'.* TO 'wp-user'@'localhost'     |
    +----------------------------------------------------------------------------------+

Z tym, że niektóre dodatki, czy nawet aktualizacja samego WordPress'a, mogą wymagać zmiany struktury
bazy danych, tj. dodawać/usuwać czy zmieniać właściwości tabel. Najlepszym wyjściem w tym przypadku
jest nadanie odpowiednich praw przed instalacją/aktualizacją wtyczek/WordPress'a, po czym te prawa
odebrać.
