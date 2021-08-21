---
author: Morfik
categories:
- Linux
date: "2015-10-23T14:42:17Z"
date_gmt: 2015-10-23 12:42:17 +0200
published: true
status: publish
tags:
- debian
- apt
- cron
title: Pobieranie pakietów przy pomocy cron-apt
---

Jakiś czas temu opisywałem konfigurację dla menadżera pakietów `apt` i `aptitude` [w pliku
apt.conf][1]. Ten wpis również tyczy się konfiguracji wspomnianych menadżerów, z tym, że zostanie
tutaj opisana pewna funkcjonalność, która może nam zaoszczędzić trochę czasu przy aktualizacji
systemu. Chodzi o to, że pakiety praktycznie zawsze muszą być pobrane na dysk przed ich instalacją.
Gdy nie dysponujemy dobrym pod względem przepustowości łączem, proces pobierania pakietów jest
zwykle dłuższy niż sama ich instalacja. Przydałoby się zatem zaprogramować pobieranie plików w tle,
tak by nie musieć ich pobierać tuż przez przed procesem instalacyjnym.

<!--more-->
## Wady i zalety pobierania pakietów w tle

Przy korzystaniu z debiana w wersji testowej, lub też i niestabilnej, podczas aktualizacji z dnia na
dzień zwykle jest pobieranych sporo pakietów. Problemy z wolnym łączem, o których wspomniałem we
wstępie, nie są jedynym mankamentem, który może nas lekko irytować. Innym czynnikiem, który może
spowolnić proces aktualizacji systemu, jest obciążenie łącz i serwerów. Co nam po łączu 100 mbit/s
jeśli jesteśmy w stanie przesłać w danym czasie jedynie 10 mbit/s ? Człowiek z reguły działa w dzień
i śpi w nocy, wobec czego, raczej wszyscy zgodzimy się, że łącza jak i serwery są bardziej obciążone
w dzień niż w nocy. Oczywiście, bierzemy pod uwagę jedynie mirrory lokalne, a nie te z drugiego
końca świata, gdzie ludzie wstają do roboty, podczas gdy my akurat kładziemy się spać.

Dodatkowym problemem mogą być operatorzy, którzy w jakiś sposób limitują transfer danych,
przykładowo, internet mobilny. Zwykle w jego przypadku mamy przydział 100GB na miesiąc i po ich
wykorzystaniu, różne rzeczy mogą zacząć się dziać. Zwykle następuje drastyczne obniżenie prędkości
przesyłu danych. Tacy providerzy zdają sobie jednak sprawę, że w pewnych godzinach, zwykle w nocy,
ich łącza są wykorzystane w kilku procentach. Dlatego też czasem można się spotkać z ofertą, gdzie w
określonych godzinach, np. od północy do 6 rano, dane jakie prześlemy nie są nam doliczane do
rachunku lub/i prędkość nie jest w żaden sposób limitowana w tym czasie. Codzienne aktualizacje
systemu mogą nam zjeść lwią część transferu. Dlatego też moglibyśmy zaprogramować nasze systemy tak,
by pobierały aktualizacje w tych określonych godzinach.

## Cron-apt

By nauczyć system automatycznego pobierania pakietów, musimy doinstalować [pakiet cron-apt][2]. Po
jego instalacji, zostanie utworzony szereg plików konfiguracyjnych i jeden z nich zostanie ulokowany
w `/etc/cron.d/cron-apt` . Zawiera on kilka przykładów ale najważniejsza jest ta poniższa linijka:

    0 4    * * *   root    test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt

Sprawdza ona czy plik `/usr/sbin/cron-apt` ma odpowiednie uprawnienia, a jeśli tak, to ten skrypt
zostanie wykonany o godzinie 4 rano każdego dnia. Czas, oczywiście, możemy sobie dostosować według
życzenia. Nie koniecznie też musi to być tylko jedno zdarzenie na dzień. Skrypt może zostać
wywołany wiele razy w ciągu jednego dnia, jak i też może zostać wywołany, np. raz w tygodniu. Nie
ma jednak co wywoływać tego skryptu częściej niż co 6 godzin, bo mirrory są synchronizowane 4 razy
w ciągu dnia.

Teraz pozostaje nam już tylko skonfigurowanie samego pakietu `cron-apt` . Potrzebne nam pliki
znajdują się w katalogu `/etc/cron-apt/action.d/` . Domyślnie są tam dwa pliki: `0-update` , który
odpowiada za automatyczną aktualizację listy pakietów i `3-download` , który pobiera same pakiety.
Poniżej znajduje się moja konfiguracja.

Plik `/etc/cron-apt/0-update` :

    update -o quiet=2

Plik `/etc/cron-apt/3-download` :

    autoclean -y
    dist-upgrade -d -y -o APT::Get::Show-Upgraded=true

Pliki składają się z parametrów, które są przekazane bezpośrednio do menadżera pakietów `apt` .
Numerki w nazwach plików odpowiadają za kolejność przekazywania tych plików do skryptu `cron-apt`.
Zatem najpierw zostanie zaktualizowana lista pakietów, a dopiero po tym zdarzeniu zostaną pobrane
pakiety i dokładnie o taki schemat działania tego mechanizmu nam chodzi.


[1]: /post/konfiguracja-apt-i-aptitude-w-pliku-apt-conf/
[2]: https://packages.debian.org/pl/sid/cron-apt
