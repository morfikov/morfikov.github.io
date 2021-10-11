---
author: Morfik
categories:
- WordPress
date: "2015-06-02T09:36:02Z"
date_gmt: 2015-06-02 07:36:02 +0200
published: true
status: publish
tags:
- blog
GHissueID: 139
title: 'WordPress: Wiersz poleceń wp-cli'
---

Narzędzie [wp-cli][1] to wiersz poleceń upchnięty w pliku `.phar` (PHP Archive), przy pomocy
którego możemy zarządzać instalacją WordPress'a bez potrzeby zaprzęgania do tego przeglądarki. Przy
pomocy tego skryptu będziemy w stanie instalować i aktualizować rdzenne pliki WordPress'a, jego
wtyczki i motywy, a także dokonywać szeregu operacji na bazie danych. Projekt jest na licencji MIT,
zaś jego źródła są dostępne na [githubie][2].

<!--more-->
## Instalacja wp-cli

Prawdopodobnie wszystkie pakiety, potrzebne do prawidłowego działania tej aplikacji będą już
zainstalowane na serwerze gdzie będziemy trzymać WordPress'a. Niemniej jednak, warto wiedzieć, że
skoro to jest paczka `.phar`, to wymaga do działania PHP (min. `php5-cli` ). Dodatkowo, jako że
będziemy się łączyć z bazą danych, potrzebujemy modułu do PHP, który (przynajmniej na debianie)
jest dostępny w pakiecie `php5-mysql` .

By skorzystać z wp-cli, będzie nam także potrzebny dostęp do shella na serwerze. Gdy już takowy mamy
zapewniony, możemy przejść do [instalacji][3] tego narzędzia. W tym celu przy pomocy `curl` lub
`wget` pobieramy paczkę `.phar` :

    $ curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

Sprawdzamy czy aby na pewno wszystko jest w należytym porządku i skrypt jest w stanie zwrócić nam
informację o wersji PHP obecnej na serwerze:

    $ php wp-cli.phar --info
    PHP binary:     /usr/bin/php5
    PHP version:    5.6.9-1
    php.ini used:   /etc/php5/cli/php.ini
    WP-CLI root dir:        phar://wp-cli.phar
    WP-CLI global config:
    WP-CLI project config:
    WP-CLI version: 0.19.1

Jeśli nie zostaną nam zwrócone żadne komunikaty błędów, oznacza to, że jest dobrze.

Wpisywanie za każdym razem `php wp-cli.phar` nie jest zbyt wygodne. Możemy z tego typu wywołania
zrezygnować na rzecz bardziej przyjaznego użytkownikowi, z tym, że będziemy musieli wrzucić plik
`.phar` do jednego z katalogów w zmiennej `$PATH` . Jeśli mamy prawa zapisu do jednego z tych
folderów, to by ułatwić sobie pracę z wp-cli, przeprowadzamy poniższe kroki:

    $ echo $PATH
    /usr/local/bin:/usr/bin:/bin

    $ chmod +x wp-cli.phar
    $ mv wp-cli.phar /usr/local/bin/wp

Od tego momentu wywołanie skryptu będzie sprowadzać się do wpisania w terminalu jedynie samego
`wp` .

Pamiętajmy o tym, że każda aplikacja wymaga aktualizacji, nawet `wp-cli` . Dlatego też co jakiś czas
sprawdzajmy czy nie została wypuszczona nowa wersja skryptu, która zwykle będzie dostosowana do
najnowszej wersji samego WordPress'a.

By jeszcze bardziej ułatwić sobie życie, możemy zaprzęgnąć do pracy auto uzupełnianie poleceń
(wymagany pakiet `bash-completion`) przez naciśnięcie klawisza Tab . W tym celu musimy pozyskać [ten
plik][4] i wrzucić go do, np. katalogu domowego, pod nazwą `wp-completion.bash` , po czym
dopisujemy poniższą linijkę do `~/.profile`

    [ -r ~/wp-completion.bash ] && . ~/wp-completion.bash

## Operowanie na wp-cli

By korzystać z wp-cli, musimy cały czas przechodzić do katalogu z instalacją WordPress'a oraz
definiować określone parametry, co nie jest zbyt wygodne. Możemy jednak utworzyć plik konfiguracyjny
dla wp-cli i tam wpisać wszystkie te wprowadzane przez nas parametry, które będą brane pod uwagę
ilekroć będziemy chcieli pracować z WordPress'em. Możemy także przestawić domyślą ścieżkę do plików
WordPress'a przy pomocy opcji `path:` , co wyeliminuje nawet przechodzenie do katalogu z instalacją.

Mamy do dyspozycji trzy pliki. Najpierw jest czytany `wp-cli.local.yml` , który musi znajdować się w
katalogu roboczym lub wyżej. W przypadku jego braku, czytany jest `wp-cli.yml` , który również musi
znajdować w się w katalogu roboczym lub wyże. Jeśli natomiast żaden z tych dwóch plików nie zostanie
odnaleziony, wtedy pod uwagę jest brany `~/.wp-cli/config.yml` . natomiast dokładną listę opcji
możemy znaleźć [pod tym linkiem][5].

Skrypt wp-cli jest wykonywany z prawami konkretnego użytkownika w systemie i wszelkie stworzone za
jego pomocą pliki będą dziedziczyć te prawa, co może czasem zablokować możliwość korzystania z
niektórych funkcji WordPress'a.


[1]: http://wp-cli.org/
[2]: https://github.com/wp-cli/wp-cli
[3]: http://wp-cli.org/#install
[4]: https://raw.githubusercontent.com/wp-cli/wp-cli/master/utils/wp-completion.bash
[5]: https://make.wordpress.org/cli/handbook/config/
