---
author: Morfik
categories:
- Linux
date: "2015-09-27T15:10:04Z"
date_gmt: 2015-09-27 13:10:04 +0200
published: true
status: publish
tags:
- sql
- mysql
- mariadb
title: Użytkownik debian-sys-maint w MariaDB
---

Właśnie robiłem migrację z MySQL ma MariaDB na swoim debianie. Nie obyło się jednak bez problemów,
choć na dobrą sprawę serwer baz danych działał i na pierwszy rzut oka nic złego nie szło
zaobserwować. Dopiero po zajrzeniu w log okazało się, że są jakieś problemy z uprawnieniami, w
efekcie czego nie mogła być przeprowadzona akcja aktualizacji tabel.

<!--more-->
## Serwer baz danych działa bez zarzutu

Serwer baz danych wstaje bez problemu, przynajmniej do takiego wniosku można dojść patrząc po
procesach:

    # ps -eo "user,group,args" | grep mysql
    root     root     /bin/bash /usr/bin/mysqld_safe
    root     root     logger -p daemon err -t /etc/init.d/mysql -i
    mysql    mysql    /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --log-error=/var/log/mysql/error.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306

W logu natomiast można dostrzec poniższy błąd:

    mysql[96]: Starting MariaDB database server: mysqld.
    systemd[1]: Started LSB: Start and stop the mysql database server daemon.
    /etc/mysql/debian-start[347]: Upgrading MySQL tables if necessary.
    /etc/mysql/debian-start[370]: /usr/bin/mysql_upgrade: the '--basedir' option is always ignored
    /etc/mysql/debian-start[370]: Looking for 'mysql' as: /usr/bin/mysql
    /etc/mysql/debian-start[370]: Looking for 'mysqlcheck' as: /usr/bin/mysqlcheck
    /etc/mysql/debian-start[370]: Version check failed. Got the following error when calling the 'mysql' command line client
    /etc/mysql/debian-start[370]: ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
    /etc/mysql/debian-start[370]: FATAL ERROR: Upgrade failed
    /etc/mysql/debian-start[422]: Checking for insecure root accounts.
    mysql[96]: ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

Jak widzimy wyżej, z jakiegoś powodu skrypt próbuje wykonać się i chce uzyskać dostęp do bazy danych
w oparciu o użytkownika `root` no i brak hasła.

Instalacje MySQL oraz MariaDB różnią się nieco i to co obserwujemy wyżej, to właśnie efekt, który
powstał za sprawą złych plików konfiguracyjnych. Przy MySQL był wykorzystywany plik
`/etc/mysql/debian.cnf` i jego treść była generowana przy instalacji pakietów. Był tam zdefiniowany
użytkownik `debian-sys-maint` oraz zahashowane 16-znakowe hasło. W przypadku MariaDB, ten plik ma
nieco inną formę:

    # Automatically generated for Debian scripts. DO NOT TOUCH!
    [client]
    host     = localhost
    user     = root
    password =
    socket   = /var/run/mysqld/mysqld.sock
    [mysql_upgrade]
    host     = localhost
    user     = root
    password =
    socket   = /var/run/mysqld/mysqld.sock
    basedir  = /usr

Jak widzimy, jest użytkownik `root` oraz nieustawione hasło. Jako, że dokonywałem migracji i stara
baza danych mi została, to oczywistym jest, że te ustawienia nie zadziałają i serwer trzeba
skonfigurować ręcznie. Co zatem należałoby zrobić by rozwiązać zaistniały problem?

## Konfiguracja użytkownika debian-sys-maint

Do istniejącej bazy danych logujemy się przy pomocy konta `root` oraz hasła, z którego korzystaliśmy
przed migracją.

    # mysql -u root -p
    Enter password:
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 7
    Server version: 10.0.20-MariaDB-3 Debian unstable
    ...

Sprawdźmy jakich użytkowników mamy w bazie:

    MariaDB [(none)]> select User,Host from mysql.user;
    +--------------+-----------+
    | User         | Host      |
    +--------------+-----------+
    | root         | 127.0.0.1 |
    | root         | ::1       |
    | phpmyadmin   | localhost |
    | root         | localhost |
    +--------------+-----------+
    4 rows in set (0.00 sec)

Brak użytkownika `debian-sys-maint` . Musimy go zatem pierw utworzyć i przypisać mu jakieś hasło:

    MariaDB [(none)]> use mysql
    MariaDB [mysql]> CREATE USER 'debian-sys-maint'@'localhost';
    MariaDB [mysql]> UPDATE mysql.user SET Password=PASSWORD('jakies-haslo') WHERE User='debian-sys-maint';
    MariaDB [mysql]> FLUSH PRIVILEGES;

Powinniśmy być teraz w stanie wyciągnąć hash hasła bezpośrednio z bazy danych:

    MariaDB [mysql]> select User,Host,Password from mysql.user where User = 'debian-sys-maint';
    +------------------+-----------+-------------------------------------------+
    | User             | Host      | Password                                  |
    +------------------+-----------+-------------------------------------------+
    | debian-sys-maint | localhost | *EC29E219FADCD4207E46F19F46A41A5866AA49FF |
    +------------------+-----------+-------------------------------------------+
    1 row in set (0.00 sec)

Jak możemy wyczytać na stronie Oracle, we wcześniejszych wersjach MySQL, [ten hash miał 16 znaków,
obecnie ma 41](https://dev.mysql.com/doc/refman/5.7/en/password-hashing.html). Zatem jeśli
generujemy hasło z wykorzystaniem `PASSWORD()` na nowszej wersji MySQL, to nie damy rady odtworzyć
starego pliku konfiguracyjnego, bo tamten będzie zawierał inny hash. Wyżej jednak mamy 41 znakowy
ciąg, który możemy wykorzystać. Gwiazdka na początku sugeruje, że nie jest to hasło w czystej
postaci.

Otwieramy zatem plik `/etc/mysql/debian.cnf` i w linijkach z `password` umieszczamy powyższy hash
(razem z gwiazdką). Zmieniamy także nazwę użytkownika, na tę którą ustawiliśmy w bazie danych. Cały
plik powinien wyglądać zatem następująco:

    # Automatically generated for Debian scripts. DO NOT TOUCH!
    [client]
    host     = localhost
    user     = debian-sys-maint
    password = *EC29E219FADCD4207E46F19F46A41A5866AA49FF
    socket   = /var/run/mysqld/mysqld.sock
    [mysql_upgrade]
    host     = localhost
    user     = debian-sys-maint
    password = *EC29E219FADCD4207E46F19F46A41A5866AA49FF
    socket   = /var/run/mysqld/mysqld.sock
    basedir  = /usr

Ostatnia rzecz, którą musimy jeszcze zrobić, to nadać użytkownikowi `debian-sys-maint` odpowiednie
uprawnienia w bazie
    danych:

    MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'debian-sys-maint'@'localhost' IDENTIFIED BY '*EC29E219FADCD4207E46F19F46A41A5866AA49FF';

Resetujemy usługę mysql i w logu powinniśmy zaobserwować szereg faz sprawdzania tabel:

    /etc/mysql/debian-start[352]: Phase 1/6: Checking and upgrading mysql database
    /etc/mysql/debian-start[352]: Processing databases
    /etc/mysql/debian-start[352]: mysql
    /etc/mysql/debian-start[352]: mysql.column_stats                                 OK
    ...
    /etc/mysql/debian-start[352]: Phase 2/6: Fixing views
    /etc/mysql/debian-start[352]: Processing databases
    /etc/mysql/debian-start[352]: information_schema
    ...
    /etc/mysql/debian-start[352]: Phase 3/6: Running 'mysql_fix_privilege_tables'
    /etc/mysql/debian-start[352]: Phase 4/6: Fixing table and database names
    /etc/mysql/debian-start[352]: Processing databases
    /etc/mysql/debian-start[352]: information_schema
    ...
    /etc/mysql/debian-start[352]: Phase 5/6: Checking and upgrading tables
    /etc/mysql/debian-start[352]: Processing databases
    /etc/mysql/debian-start[352]: information_schema
    ...
    /etc/mysql/debian-start[352]: Phase 6/6: Running 'FLUSH PRIVILEGES'
    /etc/mysql/debian-start[352]: OK
    /etc/mysql/debian-start[647]: Checking for insecure root accounts.
    /etc/mysql/debian-start[651]: Triggering myisam-recover for all MyISAM tables

Oznacza to, że problemy z uprawnieniami użytkownika `debian-sys-maint` zostały z powodzeniem
poprawione.
