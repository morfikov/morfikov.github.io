---
author: Morfik
categories:
- Linux
date: "2015-11-01T19:14:55Z"
date_gmt: 2015-11-01 17:14:55 +0100
published: true
status: publish
tags:
- gpg
- debian
- apt
- repozytorium
GHissueID: 248
title: Klucze do repozytoriów Debiana (trusted.gpg)
---

Obecnie systemy operacyjne stają się nieco bardziej stabilne i czasy, w których reinstalacja
takiego systemu, czy też nawet format dysku, odchodzą powoli w niebyt. [Data instalacji mojego
linux'a][1] wskazuje na prawie 2 lata wstecz. Jakby nie patrzeć jest to szmat czasu, w czasie
którego przez mojego Debiana przetoczyła się ogromna ilość oprogramowania. Nie zawsze były to
pakiety, które pochodziły z głównych repozytoriów tej dystrybucji. Niemniej jednak, każde
repozytorium z pakietami jest podpisane i by móc z nich bezpiecznie korzystać, trzeba pozyskać
[klucz GPG][2] i dokonać jego weryfikacji. Prędzej czy później przyjdzie czas, gdy takie klucze GPG
przestaną być ważne lub też zmianie ulegną źródła pakietów. W ten sposób baza danych kluczy
zawierać będzie szereg zbędnych pozycji. Może wielu ludziom nie przeszkadza ten fakt ale raz na
jakiś czas przydałoby się oczyścić keyring ze śmieci, które są już nam do niczego niepotrzebne.

<!--more-->
## Baza danych kluczy do repozytoriów

Debian wykorzystuje mechanizm [secure APT][3], do którego działania potrzebne są klucze GPG. W taki
sposób za każdym razem, gdy dodajemy adres nowego repozytorium do pliku `/etc/apt/sources.list` i
aktualizujemy listę pakietów via `apt-get update` , to w terminalu możemy zobaczyć komunikat, który
informuje nas, że nie posiadamy jakiegoś klucza GPG. Gdy pozyskujemy taki klucz i przy pomocy
`apt-key` dodajemy go do keyring'a APT, to jest on wrzucany do pliku `/etc/apt/trusted.gpg` . Gdy
teraz usuwamy repozytorium, to te dodane klucze dalej pozostają w tym keyring'u. Istnieje jednak
sposób, który umożliwia nam usuwanie z tego keyring'a tylko tych kluczy, których chcielibyśmy się
pozbyć.

### Aktualizacja wpisów w pliku trusted.gpg

Zanim przejdziemy do usuwania kluczy, upewnijmy się, że wszystkie wpisy w keyring'u są aktualne.
Standardowo możemy to zrobić przy pomocy `apt-key adv --refresh-key` (wcześniej się używało
`apt-key update` ale to wywołanie jest obecnie już przestarzałe). W `apt-key` mamy do dyspozycji
opcję `adv` , która umożliwia przesłanie szeregu parametrów bezpośrednio do narzędzia `gpg` .
Poniżej jest przykład aktualizacji kluczy:

    # apt-key adv --refresh-key \
    --keyserver hkps://hkps.pool.sks-keyservers.net \
    --keyserver-options timeout=60 \
    --keyserver-options no-honor-keyserver-url \
    --keyserver-options include-revoked

Jeśli skonfigurowaliśmy sobie [serwer kluczy GPG][4], tak by zapytania do niego były przesyłane
siecią TOR, to potrzebujemy ustawić również proxy. Obecnie jednak, usługa `dirmngr` [jest w stanie
wykryć działającego TOR'a][5] i automatycznie przesłać do niego pakiety bez dodatkowej konfiguracji
ze strony użytkownika.

Opcji, które może podać do `apt-key` może być sporo. Warto zatem rzucić sobie okiem na [plik
gpg.conf][6] i ocenić, które z nich będą nam tutaj potrzebne.

### Usuwanie kluczy z pliku trusted.gpg

Plik `trusted.gpg` nie jest zwykłym plikiem tekstowym i nie damy rady podejrzeć go w swoim
ulubionym edytorze tekstu. Na nim operują jedynie narzędzia `apt-key` oraz `gpg` i to przy ich
pomocy możemy podejrzeć zawartość keyring'a. Listę kluczy do repozytoriów Debiana możemy uzyskać w
poniższy sposób:

    # apt-key list
    /etc/apt/trusted.gpg
    --------------------
    ...
    pub   4096R/A8492E35 2013-07-03 [expired: 2015-07-03]
    uid                  Opera Software Archive Automatic Signing Key 2013b

    pub   4096R/F6D61D45 2015-05-27 [expires: 2017-05-26]
    uid                  Opera Software Archive Automatic Signing Key 2015
    sub   4096R/26451C35 2015-05-27 [expires: 2017-05-26]
    ...

Zatem w pliku `trusted.gpg` są przechowywane informacje podobne do tych powyżej. Mamy tutaj dwa
przykładowe klucze, które odnoszą się do repozytorium Opery. Część z kluczy ma ustawioną datę
ważności, tak jak w obu powyższych przypadkach. Nie jest to jednak obowiązkowe. Jeden z powyższych
kluczy przedawnił się -- to ten co ma `expired` . Z opisu obu kluczy możemy wywnioskować, że ten
pierwszy klucz został zastąpiony przez ten drugi, który ma datę ważności ustawioną na `2017-05-26` .
Przydałoby się więc usunąć ten nieaktualny klucz z pliku `trusted.gpg` . Robimy to przez podanie ID
klucza do `apt-key del` , przykładowo:

    # apt-key del A8492E35
    OK

W ten sposób możemy usunąć wszystkie przeterminowane klucze, no i oczywiście klucze do repozytoriów,
których już nie mamy w systemie.

### Łatwiejszy sposób

Gdy nie chce nam się przeglądać pliku `trusted.gpg` ale jednocześnie chcemy go oczyścić, to możemy
ten plik zwyczajnie usunąć z katalogu `/etc/apt/` . Następnie trzeba pobrać ręcznie wszystkie
klucze, których numery identyfikacyjne pojawią się po wydaniu polecenia `apt-get update` ,
przykładowo:

    # rm /etc/apt/trusted.gpg

    # apt-get update
    ...
    W: GPG error: http://deb.opera.com stable InRelease: The following signatures
       couldn't be verified because the public key is not available: NO_PUBKEY 63F7D4AFF6D61D45

    # apt-key adv --recv-keys 63F7D4AFF6D61D45

    # ls -al /etc/apt/trusted.gpg
    -rw-r--r-- 1 root root 20K 2015-11-01 17:52:57 /etc/apt/trusted.gpg

Po dodaniu kluczy do pliku `trusted.gpg` trzeba ponownie zaktualizować listę repozytoriów przy
pomocy `apt-get update` i to w zasadzie cała robota przy oczyszczeniu keyring'a APT.


[1]: /post/dokladna-data-instalacji-systemu-linux/
[2]: https://pl.wikipedia.org/wiki/GNU_Privacy_Guard
[3]: https://wiki.debian.org/SecureApt
[4]: /post/serwer-kluczy-gpg-i-kwestia-prywatnosci/
[5]: https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/GnuPG
[6]: /post/konfiguracja-gpg-w-pliku-gpg-conf/
