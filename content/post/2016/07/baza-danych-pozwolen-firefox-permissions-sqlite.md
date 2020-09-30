---
author: Morfik
categories:
- Linux
date: "2016-07-17T21:43:14Z"
date_gmt: 2016-07-17 19:43:14 +0200
published: true
status: publish
tags:
- firefox
- baza-danych
title: Baza danych pozwoleń w Firefox'ie (permissions.sqlite)
---

Praktycznie każda przeglądarka, w tym też i Firefox, oferuje możliwość nadania określonym domenom
praw dostępu do zasobów systemowych. Chodzi generalnie o wykorzystywanie wtyczek, np. flash, które
są aktywowane na danej stronie internetowej jeśli ta ich potrzebuje. Po części też sprawa dotyczy
korzystania z urządzeń takich jak wbudowane w laptop kamera i mikrofon oraz szeregu dodatkowych
rzeczy, np. ciasteczka, pop-up'y i inne takie. Obecnie Firefox standardowo blokuje dostęp do
pluginów, a gdy zachodzi potrzeba skorzystania z któregoś z nich, to zostaje nam zaprezentowane
okienko, w którym możemy zdecydować co zrobić. Gdy często odwiedzamy daną witrynę, to naturalnie
prosimy naszą przeglądarkę, by ta zapisała ustawienia dla tej strony. Firefox robi to przez dodanie
wyjątku w pliku `permissions.sqlite` . W sporej części przypadków będziemy mogli cofnąć pozwolenia w
dość prosty sposób. Niemniej jednak, nie we wszystkich z nich da się to tak łatwo zrobić.

<!--more-->
## Nadawanie i odbieranie pozwoleń domenom w Firefox'ie

Odpalmy sobie zatem czysty profil Firefox'a i wejdźmy w jego opcje. Konkretnie to udajemy się pod
Add-ons -> Plugins. Mamy tutaj wypisane wszystkie wtyczki, które zostały wykryte przez naszą
przeglądarkę:

![](/img/2016/07/1.firefox-plugins-wtyczki-konfiguracja-pozwolenia.png#huge)

Każda strona w internecie może uzyskać do nich dostęp jeśli będzie chciała. Zwróćmy zatem szczególną
uwagę na przyciski przy każdej z wtyczek i upewnijmy się, że mamy tam `Ask to Acticate` . W ten
sposób każdy taki plugin trzeba będzie aktywować ręcznie za każdym razem, gdy strona będzie chciała
z niego korzystać. Przejdźmy teraz na stronę Adobe, gdzie przetestujemy sobie ten mechanizm
aktywacji na wtyczce flash. Na samej stronie powinniśmy ujrzeć szereg obrazków podobnych do tego
poniżej:

![](/img/2016/07/2.firefox-strona-test-aktywacja-wtyczki.png#small)

Każdy taki obrazek oznacza, że strona chciała skorzystać z danej wtyczki ale Firefox zablokował te
żądania. Informacje o tym jaka wtyczka jest potrzebna, znajduje się na widocznym wyżej obrazku.
Jeśli faktycznie chcemy aktywować daną wtyczkę na tej stronie, to klikamy w taki obrazek i po
chwili pojawia nam się taki oto pop-up:

![](/img/2016/07/3.firefox-pop-up-allow-and-remember.png#huge)

Mamy tutaj widoczne dwa przyciski: `Allow Now` oraz `Allow and Remember` . W przypadku wybrania
pierwszej opcji, strona będzie mogła skorzystać z pluginu flash ale tylko tymczasowo. Gdy z kolei
wybierzemy drugą opcję, to do pliku `permissions.sqlite` zostanie dodany wyjątek i w późniejszym
czasie nie będziemy musieli sobie głowy zawracać aktywacją tej konkretnej wtyczki na tej stronie.

Problem natomiast zaczyna się w chwili odbierania tych uprawnień. O ile znamy adres domeny, to po
jej załadowaniu możemy wybrać z menu kontekstowego `View Page Info` :

![](/img/2016/07/4.firefox-uprawnienia-strony-wtyczki.png#small)

W okienku, które nam się pojawi, przechodzimy do `Permissions` i tutaj już mamy wyszczególnione
informacje na temat wszystkich zezwoleń dotyczących danego serwisu www:

![](/img/2016/07/5.firefox-uprawnienia-strony-wtyczki.png#big)

Wszędzie tam, gdzie status różni się od `Use Default` są tworzone wyjątki, a informacje o nich
wędrują do pliku `permissions.sqlite` .

## Czyszczenie pliku permissions.sqlite

Wyobraźmy sobie teraz, że przez parę lat przeglądamy strony www i dodajemy całą masę wyjątków dla
różnych domen zmieniając te powyższe ustawienia. Jednym z problemów będzie nadmierny rozrost pliku
`permissions.sqlite` . Choć to jeszcze nie jest najgorsza rzecz jaka może nas spotkać. Załóżmy
teraz, że jakaś domena wygasła i dla tej domeny mieliśmy specyficzną konfigurację uprawnień. Jeśli
kto inny zarejestruje sobie tę domenę, to zwyczajnie odziedziczy ona uprawnienia, które nadaliśmy
jej kilka lat wstecz, a to nie jest zbytnio bezpieczne.

W przypadku domen, których adresy są nam znane, dostosowanie uprawnień nie powinno stanowić
większego problemu. Co jednak w przypadku tych zapomnianych stron, które nawet nie figurują w
naszych ulubionych zakładach? Skąd mamy wiedzieć, czy daną domenę odwiedzaliśmy już i czy
zmienialiśmy jej uprawnienia? Oczywiście można to wyciągnąć z informacji o stronie ale kto by się w
to bawił. Jako, że w pliku `permissions.sqlite` znajduje się lista wyjątków, to przydałoby się
podejrzeć ten plik i usunąć wszystkie zbędne wpisy. Problem w tym, że Firefox nie umożliwia nam
tego. Zatem zostają nam z grubsza dwie opcje: zmiana zezwoleń dla każdej pojedynczej strony oraz
zwyczajne usunięcie tego pliku z katalogu
profilu ( `~/.mozilla/firefox/12345678.default/permissions.sqlite` ).

## Edytowanie pliku permissions.sqlite przez SQLite Manager

Może i Firefox nie umożliwia nam ręcznej edycji pliku `permissions.sqlite` ale są dostępne dodatki,
które operują bezpośrednio na nim. Jednym z nich jest [SQLite
Manager](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/). Przy pomocy tego dodatku
jesteśmy w stanie uzyskać dostęp do tabeli z uprawnieniami, gdzie mamy ładnie opisane domeny.
Poniżej przykład:

![](/img/2016/07/6.firefox-sql-manager-add-on.png#huge)

W kolumnie `permission` mamy dwie wartości: `1` lub `2` . Wartość `1` oznacza zezwolenie dostępu,
zaś `2` odmowę dostępu. Każde ustawienie, które mogliśmy zmienić przez `View Page Info` na danej
stronie, będzie miało w tej tabeli osobny wpis. Jeśli chcemy teraz cofnąć wszystkim domenom prawa do
korzystania z pluginu flash, to wystarczy kliknąć na odpowiednich pozycjach prawym przyciskiem
myszki i wybrać `Delete` :

![](/img/2016/07/7.firefox-sql-manager-add-on.png#huge)

Można także wykonać zwykłe zapytanie do bazy danych i przy jego pomocy usunąć konkretne rekordy. Dla
przykładu przechodzimy na zakładkę `Execute SQL` i wybieramy Data Manipulation > DELETE . Następnie
wpisujemy to poniższe zapytanie:

    DELETE FROM moz_perms WHERE type='plugin:flash'

Zapytanie powinno wykonać się bez błędów, a z tabeli uprawnień powinny zniknąć wszystkie wpisy,
które miały w kolumnie `type` wartość `plugin:flash` :

![](/img/2016/07/8.firefox-sql-manager-add-on.png#huge)

Po edycji bazy danych musimy jeszcze uruchomić ponownie przeglądarkę.
