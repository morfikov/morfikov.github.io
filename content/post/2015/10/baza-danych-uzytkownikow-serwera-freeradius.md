---
author: Morfik
categories:
- Linux
date: "2015-10-13T19:31:05Z"
date_gmt: 2015-10-13 17:31:05 +0200
published: true
status: publish
tags:
- baza-danych
- wifi
title: Baza danych użytkowników serwera freeradius
---

Poprzedni wpis był o sieciach bezprzewodowych, a konkretnie dotyczył on [konfiguracji protokołu WPA
Enterprise w oparciu o serwer
freeradius](/post/wpa2-enterprise-serwer-freeradius/). Ten post będzie w podobnym
klimacie, z tym, że skupimy się tutaj na nieco innym podejściu to kwestii użytkowników, którzy mogą
się łączyć do sieci WiFi. Ich konfiguracja nie zostanie praktycznie w żaden sposób ruszona, no może
za wyjątkiem [przeniesienia jej do bazy danych MySQL](http://wiki.freeradius.org/guide/SQL-HOWTO).

<!--more-->
## Moduł sql

Baza danych przyda nam się nie tylko do przechowywania informacji o użytkownikach, którym zezwolimy
na dostęp do naszej sieci WiFI ale także do prowadzenia statystyk. Najpierw musimy jednak włączyć i
skonfigurować moduł `sql` . By włączyć ten moduł, edytujemy plik `/etc/freeradius/radiusd.conf` i
usuwamy `#` z poniższych linijek:

    ...
    $INCLUDE sql.conf
    ...
    $INCLUDE sql/mysql/counter.conf
    ...

Teraz przechodzimy do edycji pliku `/etc/freeradius/sql.conf` i ustawiamy odpowiednią konfigurację
dla bazy danych MySQL. Na dobrą sprawę domyślna konfiguracja jest w porządku, trzeba się jednak
upewnić, że ustawiliśmy odpowiedni typ bazy danych, port, adres, użytkownika i hasło. Z grubsza to
by było:

    sql {
    ...
       database = "mysql"
       ...
       server = "localhost"
       port = 3306
       ...
       login = "radius"
       password = "radpass"
       ...
       read_groups = yes
       ...
       readclients = yes
       ...

W powyższym pliku jest również zdefiniowana konfiguracja tabel. Jeśli nie odpowiadają nam domyślne
nazwy, można oczywiście zmienić wedle uznania. Dodatkowo, musimy pozmieniać odpowiednie linijki w
plikach `/etc/freeradius/sites-enabled/default` i `/etc/freeradius/sites-enabled/inner-tunnel` .
Chodzi konkretnie o te poniższe:

    ...
    authorize {
          ...
          sql
          ...

    accounting {
       ...
    #  detail
       ...
    #  unix
       ...
    #  radutmp
          ...
          sql
          ...

    session {
       ...
    #     radutmp
       ...
       sql
    ...

Jeśli potrzebujemy zalogować wszelkie zapytania z próbami logowania, możemy także odhashować moduł
`sql` w sekcji `post-auth` . Nie jest to jednak zalecane na produkcyjnych serwerach, bo ten log
zawierać może dość dużo informacji, a to może z kolei bardzo obciążyć serwer jak i zjeść sporo
miejsca na dysku. W każdym razie jeśli potrzebujemy tej funkcjonalności do testów, to usuwamy hasha
z poniższej linijki:

    post-auth {
       ...
       sql
       ...

W przypadku wykorzystania bazy danych, mamy także możliwość ogarnięcia ilości jednoczesnych sesji
jakie może nawiązać dany użytkownik. W tym celu trzeba edytować plik
`/etc/freeradius/sql/mysql/dialup.conf` i odhashować poniższe linijki:

    simul_count_query = "SELECT COUNT(*) \
       FROM ${acct_table1} \
       WHERE username = '%{SQL-User-Name}' \
       AND acctstoptime IS NULL"

## Baza danych na potrzeby freeradius'a

By stworzyć bazę danych i użytkownika, który będzie mógł na tej bazie operować, możemy posłużyć się
gotowymi plikami, które znajdują się w katalogu `/etc/freeradius/sql/mysql/` . Zostały one utworzone
za sprawą pakietu `freeradius-mysql` . Interesują nas z grubsza trzy pliki:
`/etc/freeradius/sql/mysql/schema.sql` , `/etc/freeradius/sql/mysql/nas.sql` oraz
`/etc/freeradius/sql/mysql/admin.sql` . Poniżej jest zawartość wszystkich trzech plików.

Plik `/etc/freeradius/sql/mysql/schema.sql` :

    #
    # Table structure for table 'radacct'
    #

    CREATE TABLE radacct (
      radacctid bigint(21) NOT NULL auto_increment,
      acctsessionid varchar(64) NOT NULL default '',
      acctuniqueid varchar(32) NOT NULL default '',
      username varchar(64) NOT NULL default '',
      groupname varchar(64) NOT NULL default '',
      realm varchar(64) default '',
      nasipaddress varchar(15) NOT NULL default '',
      nasportid varchar(15) default NULL,
      nasporttype varchar(32) default NULL,
      acctstarttime datetime NULL default NULL,
      acctstoptime datetime NULL default NULL,
      acctsessiontime int(12) default NULL,
      acctauthentic varchar(32) default NULL,
      connectinfo_start varchar(50) default NULL,
      connectinfo_stop varchar(50) default NULL,
      acctinputoctets bigint(20) default NULL,
      acctoutputoctets bigint(20) default NULL,
      calledstationid varchar(50) NOT NULL default '',
      callingstationid varchar(50) NOT NULL default '',
      acctterminatecause varchar(32) NOT NULL default '',
      servicetype varchar(32) default NULL,
      framedprotocol varchar(32) default NULL,
      framedipaddress varchar(15) NOT NULL default '',
      acctstartdelay int(12) default NULL,
      acctstopdelay int(12) default NULL,
      xascendsessionsvrkey varchar(10) default NULL,
      PRIMARY KEY  (radacctid),
      KEY username (username),
      KEY framedipaddress (framedipaddress),
      KEY acctsessionid (acctsessionid),
      KEY acctsessiontime (acctsessiontime),
      KEY acctuniqueid (acctuniqueid),
      KEY acctstarttime (acctstarttime),
      KEY acctstoptime (acctstoptime),
      KEY nasipaddress (nasipaddress)
    ) ;

    #
    # Table structure for table 'radcheck'
    #

    CREATE TABLE radcheck (
      id int(11) unsigned NOT NULL auto_increment,
      username varchar(64) NOT NULL default '',
      attribute varchar(64)  NOT NULL default '',
      op char(2) NOT NULL DEFAULT '==',
      value varchar(253) NOT NULL default '',
      PRIMARY KEY  (id),
      KEY username (username(32))
    ) ;

    #
    # Table structure for table 'radgroupcheck'
    #

    CREATE TABLE radgroupcheck (
      id int(11) unsigned NOT NULL auto_increment,
      groupname varchar(64) NOT NULL default '',
      attribute varchar(64)  NOT NULL default '',
      op char(2) NOT NULL DEFAULT '==',
      value varchar(253)  NOT NULL default '',
      PRIMARY KEY  (id),
      KEY groupname (groupname(32))
    ) ;

    #
    # Table structure for table 'radgroupreply'
    #

    CREATE TABLE radgroupreply (
      id int(11) unsigned NOT NULL auto_increment,
      groupname varchar(64) NOT NULL default '',
      attribute varchar(64)  NOT NULL default '',
      op char(2) NOT NULL DEFAULT '=',
      value varchar(253)  NOT NULL default '',
      PRIMARY KEY  (id),
      KEY groupname (groupname(32))
    ) ;

    #
    # Table structure for table 'radreply'
    #

    CREATE TABLE radreply (
      id int(11) unsigned NOT NULL auto_increment,
      username varchar(64) NOT NULL default '',
      attribute varchar(64) NOT NULL default '',
      op char(2) NOT NULL DEFAULT '=',
      value varchar(253) NOT NULL default '',
      PRIMARY KEY  (id),
      KEY username (username(32))
    ) ;


    #
    # Table structure for table 'radusergroup'
    #

    CREATE TABLE radusergroup (
      username varchar(64) NOT NULL default '',
      groupname varchar(64) NOT NULL default '',
      priority int(11) NOT NULL default '1',
      KEY username (username(32))
    ) ;

    #
    # Table structure for table 'radpostauth'
    #

    CREATE TABLE radpostauth (
      id int(11) NOT NULL auto_increment,
      username varchar(64) NOT NULL default '',
      pass varchar(64) NOT NULL default '',
      reply varchar(32) NOT NULL default '',
      authdate timestamp NOT NULL,
      PRIMARY KEY  (id)
    ) ;

Plik `/etc/freeradius/sql/mysql/nas.sql` :

    #
    # Table structure for table 'nas'
    #
    CREATE TABLE nas (
      id int(10) NOT NULL auto_increment,
      nasname varchar(128) NOT NULL,
      shortname varchar(32),
      type varchar(30) DEFAULT 'other',
      ports int(5),
      secret varchar(60) DEFAULT 'secret' NOT NULL,
      server varchar(64),
      community varchar(50),
      description varchar(200) DEFAULT 'RADIUS Client',
      PRIMARY KEY (id),
      KEY nasname (nasname)
    );

Plik `/etc/freeradius/sql/mysql/admin.sql` :

    #
    #  Create default administrator for RADIUS
    #
    CREATE USER 'radius'@'localhost';
    SET PASSWORD FOR 'radius'@'localhost' = PASSWORD('radpass');

    # The server can read any table in SQL
    GRANT SELECT ON radius.* TO 'radius'@'localhost';

    # The server can write to the accounting and post-auth logging table.
    #
    #  i.e.
    GRANT ALL on radius.radacct TO 'radius'@'localhost';
    GRANT ALL on radius.radpostauth TO 'radius'@'localhost';

O ile w przypadku dwóch pierwszych plików nie trzeba zbytnio nic zmieniać, o tyle plik tworzący
użytkownika bazy danych wymaga przejrzenia i zmienienia odpowiednich parametrów, np. hasła. Ja nie
korzystałem z powyższego pliku by nadać prawa do tabel użytkownikowi `radius` i zwyczajnie
przyznałem wszystkie prawa do określonej bazy przy pomocy poniższych linijek:

    CREATE USER 'radius'@'localhost';
    SET PASSWORD FOR 'radius'@'localhost' = PASSWORD('radpass');
    GRANT USAGE ON *.* TO 'radius'@'localhost' IDENTIFIED BY 'radpass';
    GRANT ALL PRIVILEGES ON `radius`.* TO 'radius'@'localhost';

Czego efektem jest:

    mysql> SHOW GRANTS FOR radius@localhost;
    +---------------------------------------------------------------------------------------------------------------+
    | Grants for radius@localhost                                                                                   |
    +---------------------------------------------------------------------------------------------------------------+
    | GRANT USAGE ON *.* TO 'radius'@'localhost' IDENTIFIED BY PASSWORD '*B4DE392161EF46F985AFA6C49CB2F1254B3F3BF2' |
    | GRANT ALL PRIVILEGES ON `radius`.* TO 'radius'@'localhost'                                                    |
    +---------------------------------------------------------------------------------------------------------------+
    2 rows in set (0.00 sec)

Moduł `sql` wykorzystuje dwa zestawy tabel z atrybutami przy autoryzacji. Jeden zestaw (tabele
`radcheck` i `radreply` ) jest stosowany do określonego użytkownika, drugi (tabele `radgroupcheck` i
`radgroupreply` ) obsługuje członków określonych grup. Tabela `usergroup` dostarcza listy grup wraz
z użytkownikami będącymi ich członkami, jak i również pole priorytetu odpowiedzialne za kontrolę
kolejności w jakiej grupy są przetwarzane.

Gdy zapytanie przychodzi do serwera i jest przetwarzane przez moduł `sql`, cały proces wygląda mniej
więcej tak:

  - 1\. Najpierw jest przeszukiwana tabela `radcheck` w poszukiwaniu określonych atrybutów
    sprawdzających dla użytkownika.
  - 2\. Jeśli jakieś atrybuty sprawdzające zostaną odnalezione i dopasowane, wyciągane są atrybuty
    odpowiedzi z tabeli `radreply` dla tego użytkownika i to je widzimy w zwracanej przez serwer
    odpowiedzi.
  - 3\. Następnie zachodzi przetwarzanie grup, jeśli są spełnione poniższe warunki:
      - użytkownik nie jest odnaleziony w tabeli `radcheck`
      - użytkownik jest odnaleziony w tabeli `radcheck` ale żaden z atrybutów sprawdzających nie
        może zostać dopasowany
      - użytkownik jest odnaleziony w tabeli `radcheck` , atrybuty sprawdzające zostały dopasowane i
        `Fall-Through` jest ustawiony w tabeli `radreply` .
      - użytkownik jest odnaleziony w tabeli `radcheck`, atrybuty sprawdzające zostały dopasowane i
        dyrektywa `read_groups` jest ustawiona na `yes` .
  - 4\. W przypadku gdy użytkownik podlega pod przetwarzanie grup, listowane są wszystkie grupy,
    których członkiem jest dany użytkownik, posortowane według priorytetu. Ten priorytet zezwala na
    kontrolę kolejności w jakiej grupy są przetwarzane, co może mieć znaczenie w wielu przypadkach.
  - 5\. Dla każdej z grup, do której użytkownik należy, odpowiadające atrybuty sprawdzające są
    wyciągane z tabeli `radgroupcheck` i porównywane z zapytaniem. Jeśli coś zostanie dopasowane,
    atrybuty odpowiedzi dla tej grupy są wyciągane z tabeli `radgroupreply` , po czym są aplikowane.
  - 6\. Następnie przetwarzana jest kolejna grupa jeśli:
      - nie było dopasowania dla atrybutu sprawdzającego dla poprzedniej grupy.
      - `Fall-Through` został ustawiony na ostatniej pozycji w atrybutach odpowiedzi danej grupy.
  - 7\. Jeśli użytkownik ma ustawiony atrybut `User-Profile` albo też w pliku `sql.conf` jest
    ustawiona opcja `Default Profile`, kroki 4-6 są powtarzane dla grup, których członkiem jest ten
    profil.

Mając już odpowiednio przygotowane wszystkie pliki, wklepujemy w terminal poniższe linijki:

    # mysql -u root -p radius < /etc/freeradius/sql/mysql/schema.sql
    # mysql -u root -p radius < /etc/freeradius/sql/mysql/nas.sql
    # mysql -u root -p radius < /etc/freeradius/sql/mysql/admin.sql

Sprawdźmy czy baza i tabele zostały utworzone zgodnie z naszymi oczekiwaniami:

    # mysql -u root -p
    Enter password:
    ...
    mysql> SHOW DATABASES;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | radius             |
    +--------------------+
    4 rows in set (0.00 sec)

    mysql> USE radius;
    Database changed

    mysql> SHOW TABLES;
    +------------------+
    | Tables_in_radius |
    +------------------+
    | nas              |
    | radacct          |
    | radcheck         |
    | radgroupcheck    |
    | radgroupreply    |
    | radpostauth      |
    | radreply         |
    | radusergroup     |
    +------------------+
    8 rows in set (0.00 sec)

Wygląda dobrze. Dodajmy testowego użytkownika do bazy, do tabeli `radcheck` . Musimy tylko poznać
jeszcze strukturę tej tabeli:

    mysql> DESCRIBE radcheck;
    +-----------+------------------+------+-----+---------+----------------+
    | Field     | Type             | Null | Key | Default | Extra          |
    +-----------+------------------+------+-----+---------+----------------+
    | id        | int(11) unsigned | NO   | PRI | NULL    | auto_increment |
    | username  | varchar(64)      | NO   | MUL |         |                |
    | attribute | varchar(64)      | NO   |     |         |                |
    | op        | char(2)          | NO   |     | ==      |                |
    | value     | varchar(253)     | NO   |     |         |                |
    +-----------+------------------+------+-----+---------+----------------+
    5 rows in set (0.00 sec)

Teraz wpisujemy odpowiednie wartości:

    mysql> INSERT INTO `radcheck` VALUES (NULL, 'test', 'Cleartext-Password', ':=', 'test');
    Query OK, 1 row affected (0.03 sec)

    mysql> SELECT * FROM radcheck;
    +----+----------+--------------------+----+-------+
    | id | username | attribute          | op | value |
    +----+----------+--------------------+----+-------+
    |  1 | test     | Cleartext-Password | := | test  |
    +----+----------+--------------------+----+-------+
    1 row in set (0.00 sec)

I w tej chwili powinniśmy być wstanie zalogować się do sieci z wykorzystaniem nazwy użytkownika
`test` i hasła `test` , które należy podać w konfiguracji wpasupplicant'a. Z tym, że urządzenia
AP/NAS są czytane z pliku `clients.conf` , a my wyżej stworzyliśmy tabelę `nas` , w której będziemy
przechowywać dane klientów freeradius'a. Dlatego też musimy odhaczyć przetwarzanie pliku
`client.conf` i sprecyzować odpowiednie urządzenia w bazie danych. Przechodzimy zatem do edycji
pliku `/etc/freeradius/radiusd.conf` i wstawiamy hasha w poniższej linijce:

    # $INCLUDE clients.conf

Teraz definiujemy wpisy dla localhost i dla naszego AP/NAS w bazie danych:

    # mysql -u radius -p radius
    ...
    mysql> DESCRIBE nas;
    +-------------+--------------+------+-----+---------------+----------------+
    | Field       | Type         | Null | Key | Default       | Extra          |
    +-------------+--------------+------+-----+---------------+----------------+
    | id          | int(10)      | NO   | PRI | NULL          | auto_increment |
    | nasname     | varchar(128) | NO   | MUL | NULL          |                |
    | shortname   | varchar(32)  | YES  |     | NULL          |                |
    | type        | varchar(30)  | YES  |     | other         |                |
    | ports       | int(5)       | YES  |     | NULL          |                |
    | secret      | varchar(60)  | NO   |     | secret        |                |
    | server      | varchar(64)  | YES  |     | NULL          |                |
    | community   | varchar(50)  | YES  |     | NULL          |                |
    | description | varchar(200) | YES  |     | RADIUS Client |                |
    +-------------+--------------+------+-----+---------------+----------------+
    9 rows in set (0.00 sec)

    mysql> INSERT INTO nas VALUES (NULL, '192.168.1.1', 'router_1043nd', 'other', NULL , 'tajna-fraza', NULL, NULL, 'RADIUS Client');
    Query OK, 1 row affected (0.03 sec)

    mysql> INSERT INTO nas VALUES (NULL, '127.0.1.1', 'localhost', 'other', NULL , 'test123', NULL, NULL, 'RADIUS Client');
    Query OK, 1 row affected, 1 warning (0.04 sec)

    mysql> SELECT * FROM nas;
    +----+-------------+---------------+-------+-------+-------------+--------+-----------+---------------+
    | id | nasname     | shortname     | type  | ports | secret      | server | community | description   |
    +----+-------------+---------------+-------+-------+-------------+--------+-----------+---------------+
    |  1 | 192.168.1.1 | router_1043nd | other |  NULL | tajna-fraza | NULL   | NULL      | RADIUS Client |
    +----+-------------+---------------+-------+-------+-------------+--------+-----------+---------------+
    |  2 | 127.0.0.1   | localhost     | other |  NULL | test123     | NULL   | NULL      | RADIUS Client |
    +----+-------------+---------------+-------+-------+-------------+--------+-----------+---------------+
    2 rows in set (0.00 sec)

Odpalamy serwer freeradius'a i logujemy się do sieci WiFi ze stacji klienckiej w celu sprawdzenia
czy użytkownik `test` zostanie wpuszczony. Nie powinno być problemów z zalogowaniem się.

Ogólnie rzecz biorąc, schemat bazy danych odzwierciedla układ pliku `users` . Dlatego też informacje
na temat atrybutów sprawdzających (check items) jak i atrybutów odpowiedzi (reply items) można
znaleźć w `man 5 users` oraz w przykładach w pliku `users` , a co za tym idzie, nie będę tutaj
opisywał tych wszystkich rzeczy, które zostały już poruszone poruszone przeze mnie podczas
opisywania pliku `users` w poprzednim wpisie. Na dobrą sprawę wystarczy Ctrl-C odpowiedniego
parametru z pliku `users` i Ctrl-V do odpowiedniego pola w bazie danych. Poniżej tylko zamieszczam
dla porównania konfigurację użytkowników przeniesioną z pliku `users` do bazy danych:

    INSERT INTO `radcheck` VALUES
    (3,'morfik','Cleartext-Password',':=','test'),
    (4,'morfik','Simultaneous-Use',':=','2'),
    (5,'morfik','Login-Time',':=','Mo-Th0800-2000,Fr,Sa,Su'),
    (6,'morfik_laptop','Auth-type',':=','EAP');

    INSERT INTO `radreply` VALUES
    (3,'morfik','Reply-Message','=','Czolem panie kapitanie!'),
    (4,'morfik','Fall-Through','=','No'),
    (5,'morfik_laptop','Reply-Message','=','Witaj, Morfiku!'),
    (6,'morfik_laptop','Fall-Through','=','No'),;

Co owocuje poniższymi tabelami:

    mysql> select * from radcheck;
    +----+---------------+--------------------+----+-------------------------+
    | id | username      | attribute          | op | value                   |
    +----+---------------+--------------------+----+-------------------------+
    |  3 | morfik        | Cleartext-Password | := | test                    |
    |  4 | morfik        | Simultaneous-Use   | := | 2                       |
    |  5 | morfik        | Login-Time         | := | Mo-Th0800-2000,Fr,Sa,Su |
    |  6 | morfik_laptop | Auth-type          | := | EAP                     |
    +----+---------------+--------------------+----+-------------------------+
    4 rows in set (0.00 sec)

    mysql> select * from radreply;
    +----+---------------+---------------+----+-------------------------+
    | id | username      | attribute     | op | value                   |
    +----+---------------+---------------+----+-------------------------+
    |  3 | morfik        | Reply-Message | =  | Czolem panie kapitanie! |
    |  4 | morfik        | Fall-Through  | =  | No                      |
    |  5 | morfik_laptop | Reply-Message | =  | Witaj, Morfiku!         |
    |  6 | morfik_laptop | Fall-Through  | =  | No                      |
    +----+---------------+---------------+----+-------------------------+
    4 rows in set (0.00 sec)
