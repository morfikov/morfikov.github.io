---
author: Morfik
categories:
- Blog
date: "2016-07-20T20:15:44Z"
date_gmt: 2016-07-20 18:15:44 +0200
published: true
status: publish
tags:
- wordpress
- baza-danych
- mysql
GHissueID: 390
title: 'WordPress: Kilku użytkowników bazy danych'
---

Za wysokie uprawnienia zawsze prowadzą do problemów, zwłaszcza, gdy w grę wchodzą komputery i
serwisy www. Wszyscy wiemy, że WordPress nie należy do bezpiecznych rozwiązań, mimo, że cała masa
stron na necie opiera się właśnie o tego CMS'a. Można jednak wypracować sobie bezpieczny setup, pod
warunkiem, że będziemy się zawsze kierować jedna prostą zasadą. Mianowicie chodzi o ograniczenie
uprawnień. Standardowo WordPress ma zdefiniowanego jednego użytkownika w pliku `wp-config.php` ,
którym skrypt się posługuje. Zwykle też ten użytkownik ma wszystkie możliwe prawa do wszystkich
tabel w bazie danych naszego bloga czy serwisu. Nie musi tak być, a my możemy wykorzystać kilku
użytkowników i nadać im inne uprawnienia w zależności od tego jakie operacje na bazie danych będą
oni przeprowadzać. W tym wpisie zobaczymy jak zaprzęgnąć wielu użytkowników do pracy z bazą danych
Wordpress'a.

<!--more-->
## Rozgraniczenie zadań na poszczególnych użytkowników

Przede wszystkim, trzeba powiedzieć jasno, że nie wszystkie operacje, które są wykonywane w obrębie
naszego serwisu, wymagają tych samych uprawnień. Spora większość zapytań do bazy danych będzie
kierowana głównie po to, by dane z tej bazy wydobyć i zaprezentować je użytkownikom odwiedzającym
naszego bloga. W tym przypadku mamy do czynienia z dość niestandardową instalacją WordPress'a. Na
blogu nie wykorzystuje się kont dla zwykłych użytkowników. Na dobrą sprawę, to w ogóle nie trzeba
posiadać tutaj konta, by móc serwis przeglądać. Ale by ulżyć rzeszom osobników chcącym komentować
zamieszczane tutaj posty, trzeba było zaimplementować system komentarzy Disqus. Niemniej jednak, w
dalszym ciągu są potrzebne konta dla tych osób, które chcą pisać lub zarządzać tym blogiem.

Jak widzimy, z grubsza mamy trzy typy użytkowników: administratorzy, autorzy oraz czytelnicy. Możemy
na podstawie tych informacji pobieżnie sklasyfikować przywileje, np. czytelnikom nie są potrzebne
prawa zapisu do bazy danych, przynajmniej nie we wszystkich miejscach. Z kolei ani czytelnicy ani
autorzy nie muszą wykonywać zaawansowanych działań na bazie danych, bo ta kwestia przypada
administratorom. Jak zatem rozgraniczyć żądania przesyłane do bazy przez poszczególne grup
użytkowników?

Jednym z rozwiązań może być [ochrona katalogu wp-config i pliku wp-login.php przy pomocy
certyfikatów klienckich](/post/certyfikat-chroniacy-wp-login-php-wp-admin/). W ten
sposób możemy rozgraniczyć zapytania poszczególnych użytkowników, bo identyfikują się oni
konkretnymi certyfikatami. Następnie w pliku `wp-config.php` możemy dorobić poszczególne warunki,
które będą się odnosić do informacji zawartych w certyfikatach przedstawianych przez klientów.
Różne warunki, to i różni użytkownicy bazy danych. A z kolei różni użytkownicy bazy danych, to
różne uprawnienia.

## Uprawnienia użytkowników bazy danych

Zakładam, że mamy już stosowny certyfikat (jeśli nie, to odsyłam pod link wyżej), a katalog
`wp-admin/` i plik `wp-login.php` są już przy jego pomocy zabezpieczone. Będziemy potrzebować z
grubsza trzech użytkowników, którzy będą operować na bazie danych WordPress'a. Musimy ich sobie
pierw stworzyć. Logujemy się do bazy danych ( `mysql -u root -p` ) i tworzymy trzech użytkowników:

    MariaDB [mysql]> CREATE USER 'czytelnik'@'localhost';
    MariaDB [mysql]> CREATE USER 'autor'@'localhost';
    MariaDB [mysql]> CREATE USER 'admin'@'localhost';

Każdemu z tych użytkowników przypisujemy inne hasło:

    MariaDB [mysql]> SET PASSWORD FOR 'czytelnik'@'localhost' = PASSWORD('haslo');
    MariaDB [mysql]> SET PASSWORD FOR 'autor'@'localhost' = PASSWORD('inne-haslo');
    MariaDB [mysql]> SET PASSWORD FOR 'admin'@'localhost' = PASSWORD('jeszcze-inne-haslo');

Każdy z tych trzech użytkowników wymaga innego zestawu uprawnień, by móc poruszać się w wyznaczonej
strefie naszego serwisu. Poniżej są uwzględnione jedynie te uprawnienia, o które prosił WordPress w
logu Apache2:

    GRANT SELECT ON `wpdb`.* TO 'czytelnik'@'localhost' IDENTIFIED BY "haslo";
    GRANT SELECT,INSERT,UPDATE,DELETE ON `wpdb`.`wortpes_options` TO 'czytelnik'@'localhost' IDENTIFIED BY "haslo";
    GRANT SELECT,INSERT,UPDATE,DELETE ON `wpdb`.`wortpes_postmeta` TO 'czytelnik'@'localhost' IDENTIFIED BY "haslo";

    GRANT SELECT,INSERT,UPDATE,DELETE ON `wpdb`.* TO 'autor'@'localhost' IDENTIFIED BY "inne-haslo";

    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,INDEX,ALTER,DROP,LOCK TABLES ON `wpdb`.* TO 'admin'@'localhost' IDENTIFIED BY "jeszcze-inne-haslo";

## Konfiguracja Wordpress'a w wp-config.php

Ostatnim krokiem, który musimy przeprowadzić, jest edycja pliku `wp-config.php` w celu zawarcia w
nim warunku przełączania się uprawnień do bazy w zależności od tego kto odwiedzi nasz serwis.
Dodajemy zatem do tego pliku poniższy kod:

    if ( defined( 'WP_CLI' ) && WP_CLI ) {
        define('DB_USER', 'admin');
        define('DB_PASSWORD', 'jeszcze-inne-haslo');
    } elseif (strstr($_SERVER["SSL_CLIENT_VERIFY"], 'SUCCES')) {
      if (strstr($_SERVER["SSL_CLIENT_COMMONNAME"], 'morfik')) {
            define('DB_USER', 'autor');
            define('DB_PASSWORD', 'inne-haslo');
      }
    } else {
        define('DB_USER', 'czytelnik');
        define('DB_PASSWORD', 'haslo');
    }

Pamiętajmy przy tym, by usunąć wcześniejsze wystąpienia `define('DB_USER', '');` oraz
`define('DB_PASSWORD', '');` . Oczywiście tych warunków może być dowolna ilość. Tutaj mamy po jednym
dla [wp-cli](https://wp-cli.org/) oraz certyfikatu, a jeśli warunki nie zostaną dopasowane, to
system uzna, że blog przeglądany jest przez czytelnika, tj. osobę, która ma najniższy poziom
uprawnień w dostępie do bazy.

W przypadku, gdy chcemy rozdzielić czy też pogrupować użytkowników w oparciu o ich certyfikaty
klienckie, musimy nie tylko zweryfikować krok weryfikacji przesłanego certyfikatu ale także zobaczyć
co siedzi u niego w polu `COMMONNAME` . Może być to, np. nick użytkownika albo też fraza "admin",
czy cokolwiek innego, co nam przyjdzie do głowy.
