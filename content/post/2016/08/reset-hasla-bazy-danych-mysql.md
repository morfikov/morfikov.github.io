---
author: Morfik
categories:
- Linux
date: "2016-08-16T18:36:20Z"
date_gmt: 2016-08-16 16:36:20 +0200
published: true
status: publish
tags:
- mysql
GHissueID: 557
title: Jak zresetować hasło root do bazy danych MySQL
---

Dziś podczas przenoszenia jednej z baz danych przytrafiła mi się bardzo dziwna sytuacja. Niby
wszystkie kroki zostały przeprowadzone poprawnie i nic bazie nie dolega ale jest jeden problem.
Okazuje się, że po wszystkim nie sposób do tej bazy uzyskać dostęp. Tak to się już czasem zdarza, że
człowiek ustawi hasło administratora bazy i po chwili je zapomni. Generalnie rzecz biorąc, to
komputery za mnie mają pamiętać hasła do różnych aplikacji, w tym też i do baz danych. Ja tylko
ograniczam się zawsze do kilku fraz, które odblokowują keyring. Niemniej jednak, jakimś dziwnym
trafem, w tym keyring'u zabrakło hasła do tej nieszczęsnej bazy danych. Jak zatem odzyskać to
zagubione hasło do bazy MySQL? Odpowiedź jest nawet bardzo prosta, o ile się posiada dostęp do
użytkownika root na serwerze i na szczęście takowy posiadałem, więc w sumie nikt nic nie zauważył.

<!--more-->
## Czy można zresetować hasło root nie mając dostępu do serwera

Problem wynikł z faktu posiadania w bazie MySQL tylko jednego uprzywilejowanego użytkownika, który
mógł nadawać i odbierać uprawnienia innym użytkownikom bazy MySQL. Jest to z reguły standardowa
sytuacja, bo tylko użytkownik root powinien takie uprawnienia posiadać. Niemniej jednak, w
przypadku, gdy nie pamiętamy hasła do konta root, to nie możemy zwyczajnie się zalogować do bazy. W
efekcie nie możemy zmienić sobie tego zapomnianego hasła.

Hasła w bazie są z reguły zahashowane i próba ich odzyskania z jakiegoś backup'u bazy (pliku)
również nam nic nie da. Może i wyciągniemy hash hasła ale nie użyjemy go przecież w celu
zalogowania się do bazy. Zatem jeśli nie mamy dostępu do serwera, to mamy poważny problem.
Oczywiście niekoniecznie musi to być fizyczny dostęp. Możemy posiadać jedynie konto shell'owe i
łączyć się zdalnie do serwera. Wymagany jest jednak dostęp do konta administratora takiego
serwera. Jeśli takowym władamy, to bez problemu możemy odzyskać hasło.

## Reset hasła root w MySQL za sprawą --skip-grant-tables

No to skoro już siedzimy na serwerze i jesteśmy zalogowani na użytkownika root, to wypadałoby na
chwilę zatrzymać serwer baz danych:

    # systemctl stop mysql.service

Teraz możemy go odpalić z parametrem `--skip-grant-tables`:

    # /usr/bin/mysqld_safe --skip-grant-tables
    mysqld_safe Logging to '/var/log/mysql/error.log'.
    mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql

Opcja
[--skip-grant-tables](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_skip-grant-tables)
sprawia, że serwer MySQL startuje z pominięciem systemu przywilejów. W efekcie nie ma żadnych
restrykcji co do korzystania z bazy. Każdy może się do takiej bazy zalogować bez podawania
jakiegokolwiek hasła. Oczywiście nie chcielibyśmy uruchamiać serwera z taką opcją i wystawiać go na
widok publiczny.

Teraz już wystarczy się zalogować do bazy podając użytkownika `root` :

    # mysql --user=root mysql
    ...
    mysql>

Hasło zaś zmieniamy w poniższy sposób:

    mysql> UPDATE user SET PASSWORD=PASSWORD('jakies-haslo') WHERE User='root';
    mysql> flush privileges;

Po skończonej robocie opuszczamy bazę i zatrzymujemy serwer:

    # killall -9 mysqld

Po chwili można odpalić serwer baz danych w trybie normlanym:

    # systemctl start mysql.service

Teraz już powinniśmy być w stanie zalogować się na użytkownika root bazy danych podając hasło, które
wcześniej ustawiliśmy.
