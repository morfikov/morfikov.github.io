---
author: Morfik
categories:
- Linux
date: "2016-08-22T22:03:02Z"
date_gmt: 2016-08-22 20:03:02 +0200
published: true
status: publish
tags:
- apache2
- chroot
- mysql
- debian
- php
GHissueID: 560
title: Chroot Apache2 vs dyrektywa open_basedir w PHP
---

Kilka dni temu wpadł mi w oko artykuł na temat wykonania [chroot serwera
Apache2](https://nfsec.pl/root/5874). Problem z tamtym tekstem jest taki, że nie uwzględnia on
serwera bazy danych MySQL. W efekcie, taki chroot'owany Apache2 będzie miał problemy z połączeniem
się do bazy, a nasz serwis bez niej raczej nie będzie działał prawidłowo. Przydałoby się zatem
dopracować nieco ten artykuł i wypracować takie rozwiązanie, które nie popsuje przy okazji naszego
serwisu www. Dlatego też w tym wpisie wykonamy sobie chroot zarówno serwera Apache2 z obsługą PHP i
bazy danych MySQL za sprawą modułu `unixd` .

<!--more-->
## Po co nam właściwie chroot

W konfiguracji Apache2 jest standardowo kilka dyrektyw `Directory` , które określają politykę
dostępu do zasobów serwera. Domyślnie w pliku `/etc/apache2/apache2.conf` mamy min. ten poniższy
wpis:

    <Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
    </Directory>

Dzięki niemu, odwiedzający nasz serwer użytkownicy, nie mają dostępu do katalogu `/` i do wszystkich
jego podfolderów. Na niewiele by nam się taka konfiguracja zdała, dlatego też niżej w tym pliku jest
kilka dodatkowych wpisów podobnych do tego powyższego. Jeden z nich zezwala na dostęp do katalogu
`/var/www/` i to z niego możemy serwować strony internetowe. Niemniej jednak, skrypty PHP mogą wyjść
poza ten katalog i zwrócić zawartość dowolnego pliku, który mamy w systemie. Poniżej jest prosty
przykład takiego skryptu:

    <?php
        $var = file_get_contents('/etc/hosts');
        echo 'Plik /etc/hosts: <br />' . $var;
    ?>

Po uruchomieniu skryptu przez odwiedzenie tego pliku w przeglądarce, naszym oczom powinna się
pokazać zawartość pliku `/etc/hosts` . Oczywiście w dalszym ciągu obowiązują restrykcje nałożone
przez prawa dostępu do plików, przez co nie da rady, np. podejrzeć pliku `/etc/shadow` ,
przynajmniej nie w przypadku serwera działającego z uprawnieniami użytkownika/grupy `www-data` .
Oczywiście zakładamy, że prawa tego pliku również nie zezwalają temu użytkownikowi i grupie na
dostęp.

Chroot może nam pomóc w takiej sytuacji. W efekcie, wykonanie się tego powyższego skryptu nic nam
nie zwróci, bo w środowisku chroot zwyczajnie nie będziemy mieli dostępu do plików systemowych.

## Chroot za sprawą modułu unixd

Serwer Apache2 ma statycznie wkompilowany [moduł
unixd](https://httpd.apache.org/docs/2.4/mod/mod_unixd.html). Umożliwia on bardzo szybkie i proste
zamknięcie serwera w środowisku chroot. Jedyne co musimy zrobić to określić katalog chroot'a. Robimy
to w pliku `/etc/apache2/apache2.conf` przez dodanie poniższego wpisu:

    ChrootDir /jail

Nazwę katalogu `/jail/` możemy sobie dowolnie dostosować. Proponowałbym także zostawienie starej
struktury folderów wewnątrz katalogu `/jail/` . Przykładowo, jeśli w konfiguracji wirtualnego hosta
wykorzystujemy katalog `/var/www/strona/` , to wewnątrz folderu `/jail/` utwórzmy dokładnie taką
samą strukturę katalogów, a do podfolderu `strona/` przenieśmy zawartość starego katalogu. W ten
sposób nie będziemy musieli nic ruszać w konfiguracji samego Apache2 czy też jego wirtualnych
hostów.

## Dostosowanie środowiska chroot

Po przekopiowaniu zawartości naszej strony do środowiska chroot, czeka nas jeszcze drobne
dostosowanie struktury katalogów wewnątrz folderu `/jail/` . Chodzi o to, że gdybyśmy przykładowo
odwiedzili katalog serwera, który umożliwia listing plików, to pierwsze co nam się rzuci w oczy to
brak ikonek. Zamiast nich zostaną nam jedynie pokazane `[ICO]` , `[DIR]` czy `[TXT]` :

![](/img/2016/08/1.chroot-apache2-mysql-php-bledy.png#medium)

Te ikonki znajdują się w katalogu `/usr/share/apache2/` i by je wyświetlić w środowisku chroot,
musimy stworzyć taki katalog wewnątrz folderu `/jail/` i przekopiować do niego odpowiednie pliki.
Oczywiście jeśli katalog zezwala na podlinkowanie zawartości, to możemy również i z tej opcji
skorzystać:

    # mkdir -p /jail/usr/share/
    # cp -a /usr/share/apache2/ /jail/usr/share/

W pliku `/etc/apache2/apache2.conf` dopisujemy jeszcze ten poniższy blok kodu:

    <Directory /usr/share/apache2>
        AllowOverride None
        Require all granted
    </Directory>

Ten powyższy przykład z ikonkami listingu strony zwracanej przez serwer Apache2 miał jedynie
zobrazować problem z dostępem do zasobów wewnątrz środowiska chroot. Praktycznie każdy plik, którego
wymaga nasz serwis czy też konfiguracja serwera www, będziemy musieli w ten sposób przenieść do
katalogu `/jail/` .

Blog WordPress wymaga, np. bazy stref czasowych, tj. katalogu `/usr/share/zoneinfo/` oraz również
katalogu `/tmp/` z odpowiednimi uprawnieniami:

    # cp -a /usr/share/zoneinfo/ /jail/usr/share/
    # mkdir /jail/tmp/
    # chmod 1777 /jail/tmp/

Skrypty PHP nie wymagają dodatkowego dopieszczenia i będą nam działać w takim środowisku chroot.
Problem jednak pozostaje z bazą danych MySQL, która już taka miła dla nas nie będzie i trzeba
poświęcić jej nieco więcej czasu.

## Problem z bazą danych MySQL po zaimplementowaniu chroot

Zakładając, że nasz serwer www został poprawnie skonfigurowany pod kątem pracy w środowisku chroot,
możemy teraz spróbować odwiedzić nasz blog. Z reguły taki serwis wymaga do pracy serwera bazy
danych. Po przejściu pod adres www, przywita nas ten poniższy komunikat:

![](/img/2016/08/2.chroot-apache2-mysql-php-bledy.png#medium)

Serwerowi bazy danych nic nie dolega ale skrypty PHP w środowisku chroot nie potrafią go odnaleźć. W
efekcie zwracają błąd połączenia: `Can't connect to local MySQL server through socket
'/var/run/mysqld/mysqld.sock` . By poprawić ten problem, musimy edytować dwa pliki.

Plik `/etc/mysql/my.cnf` :

    ...
    [client]
    ...
    socket          = /jail/var/run/mysqld/mysqld.sock
    ...
    [mysqld_safe]
    ...
    socket          = /jail/var/run/mysqld/mysqld.sock
    ...
    [mysqld]
    ...
    pid-file       = /jail/var/run/mysqld/mysqld.pid
    socket         = /jail/var/run/mysqld/mysqld.sock
    ...

Plik `/etc/mysql/debian.cnf` :

    ...
    [client]
    ...
    socket   = /jail/var/run/mysqld/mysqld.sock
    [mysql_upgrade]
    ...
    socket   = /jail/var/run/mysqld/mysqld.sock
    ...

Tworzymy także katalog `/jail/var/run/mysqld` :

    # mkdir -p /jail/var/run/mysqld/
    # chown mysql:mysql /jail/var/run/mysqld/

I resetujemy serwer bazy danych MySQL. W tej chwili nasz serwis www powinien już uzyskać dostęp do
bazy danych. Na wypadek błędów dobrze jest zajrzeć do katalogu `/var/log/apache2/` lub też
`/var/log/mysql/` i przejrzeć znajdujące się w tych folderach logi systemowe.

## Dyrektywa open_basedir w PHP

Jak widać, upchnięcie serwera www w środowisku chroot może być nieco problematyczne. Poza tym,
niektórzy użytkownicy niezbyt przychylnie są nastawieni do tego typu rozwiązań argumentując swoje
zdanie, tym, że "chroot i tak przed niczym nas nie ochroni". Pamiętajmy, że w tym przypadku, serwer
Apache2 nie działa z uprawnieniami root, zatem może mieć on lekki problem z ucieczką poza katalog
`/jail/` . Z jedną rzeczą mogę się zgodzić, mianowicie taki chroot nie jest wygodnym rozwiązaniem.

Jako, że w artykule, którego link został przytoczony we wstępie, nacisk został położony na
uniemożliwienie skryptom PHP odczytywania plików systemowych, to możemy skorzystać z nieco innego
rozwiązania. Chodzi o [parametr
open_basedir](http://php.net/manual/en/ini.core.php#ini.open-basedir), który jest do
skonfigurowania w pliku `/etc/php5/apache2/php.ini` :

    open_basedir = "/var/www/"

Gdy skrypt PHP spróbuje uzyskać dostęp do pliku/katalogu w innym miejscu systemu plików niż
określono w dyrektywie `open_basedir` , np. za pomocą `include` lub `fopen()` , to taka próba
zostanie udaremniona. Mamy mniej więcej ten sam efekt, co w przypadku środowiska chroot, z tym, że o
wiele mniej kombinowania.

W przypadku, gdy na serwerze jest hostowanych kilka serwisów, to opcję `open_basedir` możemy
określić osobno dla każdego wirtualnego hosta w konfiguracji serwera Apache2 wykorzystując
dyrektywę `php_admin_value`. Poniżej przykład:

    <VirtualHost *:80>
        ...
        php_admin_value open_basedir "/var/www/strona/"
        ...
    </VirtualHost>
