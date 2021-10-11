---
author: Morfik
categories:
- Linux
date: "2015-06-15T15:03:21Z"
date_gmt: 2015-06-15 13:03:21 +0200
published: true
status: publish
tags:
- pendrive
- sysctl
GHissueID: 103
title: RAM, cache i dirty pages
---

Wiele czasu zajęło mi opanowanie w końcu tych bestii zwanych [dirty
pages](https://www.kernel.org/doc/Documentation/sysctl/vm.txt), które to są trzymane w cache pamięci
RAM i potrafią dać się nieźle we znaki, zwłaszcza gdy ma się mało pamięci operacyjnej i maszynę [64
bitową](https://lwn.net/Articles/572911/). Chodzi o to, że podczas kopiowania plików z/na pendrive,
system zaczyna się strasznie przycinać, bo następuje zrzucanie danych innych procesów z RAM do SWAP
by zrobić miejsce pod te dane, które są aktualnie kopiowane.

<!--more-->
## Zasada działania mechanizmu dirty pages

By zrozumieć jak działa ten mechanizm z wykorzystaniem dirty pages, trzeba rzucić pierw okiem na
statystyki danych w pamięci RAM podczas procesu kopiowania danych:

    $ cat /proc/meminfo | egrep -i "write|cache|dirty"
    Cached:           653180 kB
    SwapCached:            0 kB
    Dirty:            331544 kB
    Writeback:         15584 kB
    WritebackTmp:          0 kB

oraz:

    $ cat /proc/vmstat | egrep -i "dirty|writeback|cache"
    nr_dirty 82886
    nr_writeback 4346
    nr_writeback_temp 0
    nr_dirty_threshold 99210
    nr_dirty_background_threshold 66140

Dla kontrastu, zwykła praca systemu:

    $ cat /proc/meminfo | egrep -i "write|cache|dirty"
    Cached:           246312 kB
    SwapCached:            0 kB
    Dirty:              9952 kB
    Writeback:             0 kB
    WritebackTmp:          0 kB

oraz:

    $ cat /proc/vmstat | egrep -i "dirty|writeback|cache"
    nr_dirty 2488
    nr_writeback 0
    nr_writeback_temp 0
    nr_dirty_threshold 99940
    nr_dirty_background_threshold 66627

Jak widać, podczas kopiowania ilość danych oznaczonych jako `dirty` jest wręcz katastrofalna, ponad
300MiB. To dlatego system z małą ilością pamięci RAM się przycina przy kopiowaniu i dlatego również
dane z pamięci RAM wyskakują pod naporem tego brudnego cache.

Ci na nieco starszych maszynach mają nie lada problem do rozwiązania, bo jakby nie patrzeć używanie
komputera podczas kopiowania plików jest wręcz niemożliwe -- dźwięk się przycina, mysza freezuje co
parę sekund na parę kolejnych sekund. Na szczęście istnieje kilka parametrów w kernelu, które po
dostosowaniu mogą nam pomóc ogarnąć system zainstalowany już na dość leciwym sprzęcie.

Poniżej jest lista plików wraz z wartościami. To są domyślne ustawienia debiana:

	| Plik                                     | Wartość |
	|----------------------------------------  | ------- |
	| /proc/sys/vm/dirty\_background\_bytes    | 0       |
	| /proc/sys/vm/dirty\_background\_ratio    | 40      |
	| /proc/sys/vm/dirty\_bytes                | 0       |
	| /proc/sys/vm/dirty\_ratio                | 60      |
	| /proc/sys/vm/dirty\_writeback\_centisecs | 60000   |
	| /proc/sys/vm/dirty\_expire\_centisecs    | 3000    |

Wartość `dirty_background_bytes` jest wyrażona w bajtach, zaś `dirty_background_ratio` w procentach.
Obie odpowiadają za próg, po przekroczeniu którego proces zacznie zapisywać dirty pages na dysk.
Podobnie z `dirty_bytes` oraz `dirty_ratio`, z tym, że jest to globalny próg dla wszystkich
procesów, którego nie mogą przekroczyć. W obu przypadkach może być użyty tylko jeden parametr w tym
samym czasie, tj. albo ratio, albo bytes. Jeśli jeden parametr jest ustawiony, drugi automatycznie
dostaje wartość 0.

Jeśli chodzi o `dirty_expire_centisecs` , to jest to czas, po którym dane zawierające dirty pages
zostaną oznaczone jako stare. Ten parametr jest wyrażony w setnych częściach sekundy, w tym
przypadku jest to 30s. Parametr `dirty_writeback_centisecs` ma budzić kernelowski flusher, który ma
za zadanie zapisać stare dirty pages (oznaczone przez `dirty_expire_centisecs` ), w tym przypadku
wartość również jest wyrażona w setnych częściach sekundy, co daje interwał 600s.

Zgonie z powyższymi parametrami, dane przeznaczone do zapisu na dysk (dirty pages) rezydujące w
pamięci ponad 30 sekund będą oznaczane jako stare i przy przy następnym wybudzeniu flushera,
zostaną zrzucone na dysk, czyli po 10min, chyba, że wcześniej dojdzie do przekroczenia limitu
40%/60% maksymalnej dostępnej pamięci RAM.

## Ograniczenie cache pod dirty pages

By uporać się ze zbyt dużą ilością danych, które wędrują do cache przy kopiowaniu, można zrobić dwie
rzeczy. Pierwszą z nich jest ograniczenie domyślnego progu dla dirty pages (40%, 60%). Druga opcja,
to zmniejszenie czasu, który dirty pages spędzają w cache.

Samo trzymanie dirty pages w pamięci RAM nie jest zbyt dobrym pomysłem, bo jeśli byśmy dokonywali
zapisu szeregu plików w tym samym czasie i wartość `nr_dirty` zwiększy się, to nawet jeśli
kopiowanie zakończy się powodzeniem, część danych dalej rezyduje w pamięci RAM i czeka aż 10min
zanim zostanie zapisana na dysk. Ja ten parametr `dirty_expire_centisecs` zmniejszyłem do wartości
200 i po 2s dirty pages są oznaczane jako stare. Natomiast parametr `dirty_writeback_centisecs`
zmniejszyłem do wartości `commit` w /etc/fstab dla montowanych partycji. Domyślnie `commit` wynosi
5s (można zmienić przez dopisanie `commit=5` dobierając odpowiednio wartość). Wyższa wartość oznacza
mniej operacji na dysku, niższą temperaturę urządzenia i mniej ruchów głowicą, co też przełoży się w
jakimś tam stopniu na zużycie energii ale kosztem uszkodzenia większej ilości danych przy
powieszeniu się systemu czy odcięciu prądu. Jeśli mamy stabilny system, możemy ten parametr sobie
zwiększyć. Nie należy jednak mylić ich ze sobą, bo `commit=5` w /etc/fstab odpowiada za zapisanie
danych do dziennika systemu plików, by w przypadku zawału systemu było wiadomo jakie pliki uległy
uszkodzeniu. Natomiast `dirty_writeback_centisecs`, jak już wspomniałem, przerzuca dane z pamięci
RAM na dysk -- są to dwie osobne operacje.

Trzeba także wziąć pod uwagę, że zrzucanie dirty pages z pamięci RAM na dysk niczym się nie różni od
wydawania polecenia `sync` co określony przedział czasu. Jeśli zbyt często będziemy zrzucać dirty
pages, ucierpi na tym transfer. Podobnie ze zbyt niskim ograniczeniem progu dla dirty pages w cache
RAM. Jeśli chodzi o wartość parametru `dirty_background_ratio` , ustawiłem go sobie na 3%, zaś
`dirty_ratio` na 5% ogólnej wartości pamięci RAM posiadanej w systemie. W tym przypadku jest to
1GiB, także maksymalna wartość dla dirty pages to 50MiB. Transfer na moim dysku nie przekracza
30MiB/s, także te wartości wydają się być racjonalne. Po przekroczeniu 5% progu, dane automatycznie
będą zrzucane z pamięci RAM na dysk, co powinno skutecznie uniemożliwić zrzucenie normalnych danych
do SWAP, by zrobić tym sposobem więcej miejsca dla cache pod dirty pages.

Niemniej jednak, dirty pages powstałe przy kopiowaniu danych różnią się nieco od tych powstałych
przy zwykłym użytkowaniu systemu. Logi systemowe czy aplikacje użytkownika, jak by nie patrzeć,
generują trochę danych i zawsze mam około 5-10MiB dirty pages. Powyższe wartości, odpowiednio, 200 i
500 dla `dirty_expire_centisecs` i `vm.dirty_writeback_centisecs` powinny zapobiec trzymaniu zbyt
wielu dirty pages w cache pamięci RAM. Zatem rzućmy okiem jak obecnie wyglądają statystyki.

Podczas kopiowania danych:

    $ cat /proc/meminfo | egrep -i "write|cache|dirty"
    Cached:           382528 kB
    SwapCached:          204 kB
    Dirty:             11344 kB
    Writeback:          4444 kB
    WritebackTmp:          0 kB

oraz:

    $ cat /proc/vmstat | egrep -i "dirty|writeback|cache"
    nr_dirty 2861
    nr_writeback 1111
    nr_writeback_temp 0
    nr_dirty_threshold 4850
    nr_dirty_background_threshold 2910

A przy zwykłej pracy systemu:

    $ cat /proc/meminfo | egrep -i "write|cache|dirty"
    Cached:           291268 kB
    SwapCached:          204 kB
    Dirty:                 8 kB
    Writeback:             0 kB
    WritebackTmp:          0 kB

oraz:

    $ cat /proc/vmstat | egrep -i "dirty|writeback|cache"
    nr_dirty 2
    nr_writeback 0
    nr_writeback_temp 0
    nr_dirty_threshold 4888
    nr_dirty_background_threshold 2932

Jak widać powyżej, przy kopiowaniu danych nie ma już ponad 300MiB dirty pages. Zamiast tego mamy
około 11MiB. Za to sprawa z systemem w stanie spoczynku wygląda ciekawie, bo jak można
zaobserwować, jest bardzo niewiele dirty pages. Czasami parametr `nr_dirty` skacze na 100 ale po
chwili spada do 0.

## Konfiguracja dla sysctl.conf

Powyższe ustawienia można zapisać, tak by były ładowane ze startem systemu. Trzeba dopisać poniższe
linijki do `/etc/sysctl.conf` :

    #vm.dirty_background_bytes = 16777216
    vm.dirty_background_ratio = 3
    #vm.dirty_bytes = 50331648
    vm.dirty_ratio = 5
    vm.dirty_writeback_centisecs = 500
    vm.dirty_expire_centisecs = 200

Można także teraz załadować te ustawienia bez potrzeby resetowania systemu przez wpisanie do
terminala poniższego polecenia:

    # sysctl -p

Jako, że cześć parametrów może nie zostać załadowanych przez skrypty debianowe, najlepiej dodać do
/etc/rc.local poniższą linijkę:

    sysctl -p >/dev/null 2>&1
