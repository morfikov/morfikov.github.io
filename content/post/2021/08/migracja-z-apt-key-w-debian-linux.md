---
author: Morfik
categories:
- Linux
date:    2021-08-21 14:35:00 +0200
lastmod: 2021-08-21 14:35:00 +0200
published: true
status: publish
tags:
- debian
- apt
- repozytorium
- gpg
- bezpieczeństwo
- apt-key
title: Migracja z apt-key w Debian linux
---

Z okazji wypuszczenia parę dni temu nowego Debiana, przeglądałem sobie [notki dla wydania
stabilnego][3] pod kątem aktualizacji z buster (10) -> bullseye (11). Niby ja i tak korzystam cały
czas z unstable/experimental i zwykle jestem na bieżąco ze zmianami wprowadzanymi w tej dystrybucji
linux'a ale też zawsze coś może umknąć uwadze z perspektywy wielu miesięcy czy nawet kilku lat
(a dokładnie 25 miesięcy). Sporo z tych rzeczy, które w tych podlinkowanych notatkach wyczytałem,
miałem już załatwione wcześniej ale zapomniałem rozprawić się z repozytoriami APT. Konkretnie
chodzi tutaj o odejście od `apt-key` , czyli narzędzia, które w Debianie używane jest do dodawania
kluczy GPG do systemowego keyring'a APT. Te klucze zwykle wykorzystywane są do weryfikacji sygnatur
złożonych pod zewnętrznymi repozytoriami, a że ja mam ich sporo, to musiałem się nieco zagłębić w
temat i ustalić w jaki sposób od następnego wydania Debiana (bookworm/12) będzie się te klucze GPG
od takich repozytoriów ogarniać. No i właśnie o tym będzie ten poniższy kawałek artykułu.

<!--more-->
## Debian rezygnuje z apt-key

Od dłuższego już czasu nawet w manualu `apt-key` można było wyczytać, że to narzędzie ostatecznie
zostanie z Debiana usunięte i zastąpione innym rozwiązaniem. Generalnie ilekroć tylko dodawaliśmy
klucze GPG do zewnętrznych repozytoriów, to lądowały one w pliku `/etc/apt/trusted.gpg` . Zwykle
użytkownicy Debiana dodawali klucze do repozytoriów posługując się jednym z poniższych poleceń:

    # apt-key adv --keyserver keyserver.ubuntu.com --recv-keys  0xhash_klucza

    # wget -qO- https://morfikownia.tld/klucz.asc | apt-key add –

W notkach do nowego wydania stabilnego Debiana można jednak wyczytać, że:

> bullseye is the final Debian release to ship apt-key. Keys should be managed by dropping files
> into /etc/apt/trusted.gpg.d instead, in binary format as created by gpg --export with a .gpg
> extension, or ASCII armored with a .asc extension.

Zgodnie z tym powyższym info, aktualne stabilne wydanie Debiana będzie ostatnim, które będzie
wspierać dodawanie kluczy GPG via `apt-key` . Jest też informacja, że klucze powinny być teraz
umieszczane bezpośrednio w katalogu `/etc/apt/trusted.gpg.d/` w formacie binarnym tworzonym przez
`gpg --export ...` lub też w odpowiedniku ASCII, czyli formie tekstowej z blokiem kodu umieszczonym
między `-----BEGIN PGP PUBLIC KEY BLOCK-----`  oraz `-----END PGP PUBLIC KEY BLOCK-----` .

Starsze wydania Debiana wspierały jedynie format binarny z rozszerzeniem `.gpg` i zalecane jest by
korzystać właśnie z tego formatu, choć ta informacja jest bardziej dla właścicieli repozytoriów
chcących wpierać te niezbyt aktualne wydania. Tak czy inaczej, możemy się spodziewać, że oba formaty
będą w użyciu i nie będzie to nic niezwykłego.

## Pozbycie się pliku /etc/apt/trusted.gpg

Co jednak zrobić, gdy mamy trochę tych kluczy GPG w pliku `/etc/apt/trusted.gpg` ? W moim przypadku
przez ostatnich parę lat zebrało się ich ponad 260 KiB. Jeśli planujemy migrację na nowy system,
który odchodzi od narzędzia `apt-key` i przez to też od pliku `/etc/apt/trusted.gpg` , to najlepiej
jest ten plik usunąć w całości:

    # rm /etc/apt/trusted.gpg

W tej chwili powinniśmy mieć w systemie jedynie klucze `debian-archive-*` w katalogu
`/etc/apt/trusted.gpg.d/` (no i oczywiście klucze w `/usr/share/keyrings/` ), czyli stan rzeczy
odpowiadający standardowemu Debianowi, przynajmniej jeśli chodzi o same klucze GPG.

Jeśli odświeżymy teraz przy pomocy `apt-get update` listy pakietów, to powinno nam się pojawić
trochę błędów z informacjami o braku kluczy dla zewnętrznych repozytoriów. Poniżej przykład dla
repozytorium Signal'a:

    # apt-get update
    ...
    Err:3 https://updates.signal.org/desktop/apt xenial InRelease
      The following signatures couldn't be verified because the public key is not available:
      NO_PUBKEY D980A17457F6FB06
    ...
    W: An error occurred during the signature verification. The repository is not updated and
    the previous index files will be used. GPG error:
    https://updates.signal.org/desktop/apt xenial InRelease: The following signatures couldn't
    be verified because the public key is not available: NO_PUBKEY D980A17457F6FB06
    ...
    W: Failed to fetch https://updates.signal.org/desktop/apt/dists/xenial/InRelease  The
    following signatures couldn't be verified because the public key is not available:
    NO_PUBKEY D980A17457F6FB06
    ...
    W: Some index files failed to download. They have been ignored, or old ones used instead.

Komunikat `NO_PUBKEY D980A17457F6FB06` świadczy naturalnie o braku publicznego klucza GPG o numerku
`0xD980A17457F6FB06` , którego prywatnym odpowiednikiem zostało podpisane repozytorium
`https://updates.signal.org/desktop/apt` . Ten klucz trzeba będzie pozyskać teraz ręcznie i wrzucić
go do katalogu `/etc/apt/trusted.gpg.d/` .

Obojętne jest czy wybierzemy format binarny klucza (rozszerzenie `.gpg` , czy tekstowy `.asc` ), bo
oba z nich są wspierane przez APT. Ja jednak będę korzystał z formatu binarnego, bo wszystkie
klucze od oficjalnych repozytoriów Debiana mają ten format. Tak czy inaczej, jeśli chcemy być
konsekwentni i używać binarnego formatu kluczy GPG, to zapewne w przyszłości spotkamy się z
kluczami w formacie `.asc` . Warto wiedzieć, że można plik `.asc` przekonwertować do formatu `.gpg`
przy pomocy poniższego polecenia:

    # wget -O- https://morfikownia.tld/klucz.asc | gpg --dearmor > klucz.gpg

## Gdzie przechowywać klucze do zewnętrznych repozytoriów

Nie mam w zasadzie nic do katalogu `/etc/apt/trusted.gpg.d/` i trzymania w nim kluczy do
zewnętrznych repozytoriów ale jednak wolałbym nie mieszać oficjalnych kluczy z tymi nieoficjalnymi.
Dlatego też ja sobie utworzyłem osobny katalog `/etc/keyrings/` (wzorowany na
`/usr/share/keyrings/` ) , gdzie będę umieszczał klucze do wszystkich zewnętrznych repozytoriów,
które zamierzam włączyć w swoim Debianie. To rozwiązanie wymaga też zastosowania parametru
`signed-by` w konfiguracji repozytorium ale o tym będzie w dalszej części artykułu.

Warto w tym miejscu też wspomnieć, że wszystkie klucze umieszczone w katalogu
`/etc/apt/trusted.gpg.d/` będą z automatu honorowane przez APT, czyli za wiele w naszym systemie
się nie zmieni względem pojedynczego pliku `/etc/apt/trusted.gpg` -- teraz każdy klucz GPG będzie w
osobnym pliku, a nie wszystkie klucze w jednym zbiorczym keyring'u.

## Dodawanie kluczy do repozytoriów z pominięciem apt-key

Weźmy zatem ten powyższy przykład aplikacji Signal i jej repozytorium APT. Jak widzieliśmy wyżej,
podczas odświeżania repozytorium został zwrócony komunikat o braku klucza `0xD980A17457F6FB06` .
Twórcy Signal'a mają na swojej stronie [opis na temat dodawania ich klucza GPG][4], tak by został
on uwzględniony przez APT. Zatem nie będzie tutaj zbytniego kombinowania. Wszystko co musimy zrobić,
to wklepać w terminal kilka poniższych poleceń.

Na sam początek pobieramy klucz GPG, którym możemy zweryfikować repozytorium Signal'a:

    $ wget -O- https://updates.signal.org/desktop/apt/keys.asc | gpg --dearmor > signal-desktop-keyring.gpg
    $ cat signal-desktop-keyring.gpg | sudo tee -a /etc/apt/trusted.gpg.d/signal-desktop-keyring.gpg > /dev/null

Następnie dodajemy repozytorium Signal'a do pliku `/etc/apt/sources.list` :

    deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/signal-desktop-keyring.gpg] https://updates.signal.org/desktop/apt xenial main'

Jeśli nie chcemy ruszać pliku `/etc/apt/sources.list` , to zawsze można też stworzyć plik
`signal-desktop.list` w katalogu `/etc/apt/sources.list.d/` i do niego dodać powyższą zawartość:

    $ echo 'deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/signal-desktop-keyring.gpg] https://updates.signal.org/desktop/apt xenial main' |\
    sudo tee -a /etc/apt/sources.list.d/signal-desktop.list

Teraz już wystarczy tylko zaktualizować listę repozytoriów:

    $ sudo apt-get update

Komunikaty, które mieliśmy wcześniej (te z `NO_PUBKEY` ) powinny nam zniknąć.

### Brak klucza GPG do pobrania ze strony repozytorium

Może się też zdarzyć tak, że w instrukcji dodawania repozytorium nie będzie podana lokalizacja
pliku klucza publicznego do prostego pobrania go i wrzucenia do określonego katalogu z kluczami GPG,
tak jak to zostało opisane powyżej w przypadku Signal'a. W takiej sytuacji trzeba będzie poszukać
klucza od repozytorium na publicznym serwerze kluczy GPG i pobrać go z tego serwera. Poniżej jest
przykładowa instrukcja jak tego dokonać.

Przy pomocy `gpg` szukamy klucza o ID, który pojawił się w logu podczas aktualizacji listy pakietów
via `apt-get update` . Mieliśmy tam `NO_PUBKEY D980A17457F6FB06` , czyli szukamy klucza o ID
`0xD980A17457F6FB06` :

    $ gpg --search-keys 0xD980A17457F6FB06
    gpg: data source: https://162.213.33.8:443
    (1)     Open Whisper Systems <support@whispersystems.org>
              4096 bit RSA key 0xD980A17457F6FB06, created: 2017-04-05
    Keys 1-1 of 1 for "0xD980A17457F6FB06".  Enter number(s), N)ext, or Q)uit > 1
    gpg: key 0xD980A17457F6FB06: public key "Open Whisper Systems <support@whispersystems.org>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1

Jak widać, klucz został odnaleziony. Wybieramy numerek `1` i dodajemy klucz do keyring'a lokalnego
użytkownika systemu. Następnie eksportujemy ten klucz do pliku. W zależności od formatu wybieramy
pierwsze (dla tekstowego) lub drugie (dla binarnego) polecenie:

    $ gpg --armor --export 0xD980A17457F6FB06 > klucz.asc

    $ gpg --export 0xD980A17457F6FB06 > klucz.gpg

Tak wyeksportowany plik klucza publicznego wrzucamy teraz do `/etc/apt/trusted.gpg.d/` .

### Podpisanie konkretnego repozytorium określonym kluczem

Jeśli się uważnie przyjrzymy wpisowi umieszczonemu w pliku `/etc/apt/sources.list` , to możemy tam
dostrzec, że mamy w nim określone opcje dla APT. Chodzi o to co jest w nawiasie, tj.
`[arch=... signed-by=...]` .

Mamy tutaj `arch=` , który określa dla jakich architektur jest przeznaczone to repozytorium i w
tym przypadku jest ono jedynie dla maszyn 64-bitowych. Nas jednak bardziej interesuje ta druga
opcja, tj. `signed-by=` . Ma ona na celu wskazanie konkretnego pliku (klucza GPG) w obrębie systemu
plików naszego linux'a, który posłuży do zweryfikowania podpisu złożonego pod tym repozytorium
podczas instalowania z niego jakichś pakietów.

#### Marginalna poprawa bezpieczeństwa

Przyznam, że pierwszy raz widzę taką opcję w pliku `/etc/apt/sources.list` i postanowiłem poszukać
trochę informacji na temat jej zastosowania. [Z informacji, które znalazłem tutaj][5] wynika, że
bezpieczeństwo z tak skonfigurowanego repozytorium jest w zasadzie bardzo marginalne.

Chodzi generalnie o to, że jeden klucz GPG może zostać wykorzystany do podpisania wielu
repozytoriów. Zatem gdyby Signal miał kilka repozytoriów i dodalibyśmy kilka wpisów do pliku
`/etc/apt/sources.list` , to mając jedno repozytorium Signal'a skonfigurowane w powyższy sposób,
pozostałe repozytoria podczas aktualizacji pakietów zwrócą błąd klucza i nie będziemy w stanie z
tych repozytoriów nic zainstalować do momentu dodania im stosownych opcji `signed-by=` . W ten
sposób unikniemy sytuacji, w której możemy przeoczyć, że kilka repozytoriów jest podpisanych tym
samym kluczem i nieświadomie z nich korzystać, co mogło mieć miejsce w przypadku korzystania z
`apt-key` i przechowywania zebranych kluczy w zbiorczym keyring'u pod `/etc/apt/trusted.gpg` .

Niemniej jednak, bardziej świadoma konfiguracja repozytoriów to jedna sprawa, a instalowanie z nich
pakietów to wręcz osobna bajka. Instalując pakiety `.deb` , wołane są skrypty maintainer'a, które
są w stanie poczynić bardzo zaawansowane zmiany w systemie (bo są uruchamiane na etapie instalacji
jako root i można w nich wykonać dowolną akcję), o czym użytkownicy Debiana instalujący pakiety z
zewnętrznych repozytoriów często zapominają. Dlatego też taka paczka `.deb` może sobie dodać swój
klucz GPG i wpis w pliku `/etc/apt/sources.list` bez naszej świadomej zgody.

Przykładem takiego zachowania może być, np. MEGAsync, w którym to możemy spotkać się m.in. z czymś
takim w skrypcie `postinst` :

    ...
    if [ -d /etc/apt/sources.list.d ]; then
    cat >/etc/apt/sources.list.d/megasync.list <<EOF
    deb https://mega.nz/linux/MEGAsync/$str/ ./
    EOF
    fi

    apt-key add - >/dev/null 2>&1 <<KEY
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Version: GnuPG v2

    mI0EVj3AgQEEAO2XyJgpvE5HDRVsggcrMhf5+KpQepl7m7OyrPSgxLi72Wuy5GWp
    hO64BX1UzmdUirIEOc13YxdeuhwJ3YP0wnKUyUrdWA0r2HjOz555vN6ldrPlSCBI
    RxKBWRMQaR4cwNKQ8V4xV9tVdPGgrQ9L/4H3fM9fYqCwEMKBxxLZsF3PABEBAAG0
    IE1lZ2FMaW1pdGVkIDxzdXBwb3J0QG1lZ2EuY28ubno+iL8EEwECACkFAlY9wIEC
    GwMFCRLMAwAHCwkIBwMCAQYVCAIJCgsEFgIDAQIeAQIXgAAKCRADw606fwaOXfOS
    A/998rh6f0wsrHmX2LTw2qmrWzwPj4m+vp0m3w5swPZw1x4qSNsmNsIXX8J0ZcSE
    qymOwHZ0B9imBS3iz+U496NSfPNWABbIBnUAu8zq0IR28Q9pUcLe5MWFsw9NO+w2
    5dByoN9JKeUftZt1x76NHn5wmxB9fv7WVlCnZJ+T16+nh7iNBFY9wIEBBADHpopM
    oXNkrGZLI6Ok1F5N7+bSgiyZwkvBMAqCkPawUgwJztFKGf8F/sSbydsKRC2aQcuJ
    eOj0ZPUtJ80+o3w8MsHRtZDSxDIxqqiHeupoDRI3Be9S544vI5/UmiiygTuhmNTT
    NWgStoZz7hEK4IsELAG1EFodIMtBSkptDL92HwARAQABiKUEGAECAA8FAlY9wIEC
    GwwFCRLMAwAACgkQA8OtOn8Gjl3HlAQAoOckF5JBJWekmlX+k2RYwtgfszk31Gq+
    Jjiho4rUEW8c1EUPvK8v1jRGwjYED3ihJ6510eblYFPl+6k91OWlScnxuVVAmSn4
    35RW3vR+nYUvf3s8rctbw97gJJZAA7p5oAowTux3oHotKSYhhxKcz15goMXzSb5G
    /h7fJRhMnw4=
    =fp/e
    -----END PGP PUBLIC KEY BLOCK-----
    KEY
    ...

Zatem instalacja tej paczki czy to ręcznie, czy z repozytorium, doda osobny wpis w
`/etc/apt/sources.list.d/megasync.list` i do tego jeszcze wgra nam klucz GPG przy pomocy `apt-key`
do zbiorczego keyring'a APT, dzięki czemu dowolne repo może zostać zweryfikowane tym powyższym
kluczem, oczywiście jeśli zostało nim podpisane. Dlatego właśnie trzeba uważać na to co instalujemy
w systemie i zwracać uwagę na to, co siedzi w skryptach maintainer'a w pakietach pochodzących
spoza oficjalnego repozytorium Debiana (tyczy się to też aktualizacji takich pakietów, bo skrypty
mogą ulec zmianie między wersjami paczki).

#### Klucze GPG osadzone w plikach .sources

[Trwają prace][6] nad umożliwieniem osadzenia kluczy GPG bezpośrednio w plikach `.sources` . Warto
tutaj wyraźnie zaznaczyć, że plik `.sources` to nie to samo co plik `.list` . Te dwa pliki odróżnia
dość mocno format. Tak czy inaczej, po osadzeniu klucza GPG bezpośrednio w pliku `.sources` , jego
zawartość prezentowałaby się mniej więcej w poniższy sposób:

    Types: deb
    URIs: https://myrepo.example/ https://myotherrepo.example/
    Suites: stable not-so-stable
    Components: main
    Signed-By:
     -----BEGIN PGP PUBLIC KEY BLOCK-----
     .
     mDMEYCQjIxYJKwYBBAHaRw8BAQdAD/P5Nvvnvk66SxBBHDbhRml9ORg1WV5CvzKY
     CuMfoIS0BmFiY2RlZoiQBBMWCgA4FiEErCIG1VhKWMWo2yfAREZd5NfO31cFAmAk
     IyMCGyMFCwkIBwMFFQoJCAsFFgIDAQACHgECF4AACgkQREZd5NfO31fbOwD6ArzS
     dM0Dkd5h2Ujy1b6KcAaVW9FOa5UNfJ9FFBtjLQEBAJ7UyWD3dZzhvlaAwunsk7DG
     3bHcln8DMpIJVXht78sL
     =IE0r
     -----END PGP PUBLIC KEY BLOCK-----

Dysponując takim plikiem `.sources` , właściciele zewnętrznych repozytoriów Debiana mogliby
dostarczać go użytkownikowi systemu, a ten mógłby wrzucić go sobie do katalogu
`/etc/apt/sources.list.d/` i to wszystko co by było wymagane do skonfigurowania nowego
repozytorium. Nie było by przy tym żadnego bawienia się w szukanie i dodawanie kluczy GPG. Wszystko
by było w jednym miejscu i do tego w jednym pliku. Mam nadzieje, że uda się ten mechanizm
zaimplementować, bo znacząco uprasza on dodawanie i zarządzanie zewnętrznymi repozytoriami w
Debianie, zwłaszcza, gdy ma się ich dość sporo.

## Narzędzie extrepo

Teoretycznie istnieje [narzędzie extrepo][1], które jest nam w stanie pomóc w oganianiu na
Debianie zewnętrznych repozytoriów. Po instalacji pakietu `extrepo` , wszystko co musimy zrobić, to
przeszukać aktualną listę repozytoriów, które możemy w swoim systemie włączyć. Możemy to zrobić przy
pomocy poniższego polecenia:

    # extrepo search
    ...

        Found opera_stable:
    ---
    description: The Opera Browser (final releases)
    gpg-key-checksum:
      sha256: 4104b80be9afbf8ac83624b7ed67b50f98218d8f68cda72f03fc4cea4ba62f14
    gpg-key-file: opera_stable.asc
    policy: non-free
    source:
      Architectures: amd64 i386
      Components: non-free
      Suites: stable
      Types: deb
      URIs: https://deb.opera.com/opera-stable

    ...

Naturalnie tych repozytoriów jest więcej ([pełna lista jest także dostępna tutaj][2]) ale nam w
celach demonstracyjnych wystarczy tylko to powyższe. By włączyć to repozytorium, w terminalu
wpisujemy poniższe polecenie:

    # extrepo enable opera_stable

Włączenie konkretnego repozytorium z listy, którą nam zwróci `extrepo` sprawi, że w katalogu
`/etc/apt/sources.list.d/` zostanie utworzony plik `extrepo_*.sources` , gdzie `*` określa nazwę
włączonego repozytorium. Poniżej jest przykładowa treść utworzonego pliku:

    # cat /etc/apt/sources.list.d/extrepo_opera_stable.sources
    Uris: https://deb.opera.com/opera-stable
    Components: non-free
    Architectures: amd64 i386
    Suites: stable
    Types: deb
    Signed-By: /var/lib/extrepo/keys/opera_stable.asc
    Enabled: yes

Z kolei by to repozytorium wyłączyć, wystarczy w terminalu wpisać poniższe polecenie:

    # extrepo disable opera_stable

W takim przypadku, widoczny wyżej plik pozostanie na swoim miejscu ale na pozycji `Enabled:`
będziemy mieli `no` , co efektywnie wyłączy to repozytorium przy aktualizacji list pakietów i
żadnego pakietu z tego repozytorium już nie da się zainstalować.

Wydawać by się mogło, że `extrepo` to bardzo przydatne narzędzie do ogarniania zewnętrznych
repozytoriów. I zapewne dla wielu użytkowników Debiana tak będzie. Ja jednak mam z nim drobny
problem. Chodzi generalnie o to, że klucze, [którymi podpisane jest repozytorium, są dociągane][2]
w czasie włączania repozytorium w systemie zamiast siedzieć na stałe w paczce `extrepo` . Tak  czy
inaczej, takie rozwiązanie automatyzuje cały proces włączania repozytorium, bo nie musimy tych
kluczy szukać i dodawać ręcznie do keyring'a APT. Co jednak w przypadku, gdy ktoś ten klucz
podmieni, np. konto/usługa `extrepo` zostanie skompromitowana w jakiś sposób? Jeśli nie
zweryfikujemy klucza, który zostanie pobrany podczas dodawania repozytorium, to może się okazać, że
nie jest to ten klucz, którego byśmy się spodziewali i któremu powinniśmy zaufać.

Poniżej jest przykładowy plik repozytorium, które włączyliśmy wyżej:

    ---
    # Added by Wouter Verhelst
    opera_stable:
      description: The Opera Browser (final releases)
      source:
        Types: deb
        URIs: https://deb.opera.com/opera-stable
        Suites: stable
        Components: non-free
        Architectures: amd64 i386
      suites:
      - squeeze
      - wheezy
      - jessie
      - stretch
      - buster
      - bullseye
      policy: non-free

      gpg-key: |
        -----BEGIN PGP PUBLIC KEY BLOCK-----

        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
        =Grob
        -----END PGP PUBLIC KEY BLOCK-----

Klucz publiczny jest umieszczony między `-----BEGIN PGP PUBLIC KEY BLOCK-----` a
`-----END PGP PUBLIC KEY BLOCK-----` . Wypadałoby zatem zadać sobie pytanie: co się stanie, gdy ten
klucz (albo/i URI) ulegnie zmianie w sposób nieuprawniony? Trzeba zatem zaufać autorom projektu w
kwestii tak krytycznej jak repozytoria systemowe, a na to mnie ździebko nie stać. Ja zawsze ręcznie
muszę zweryfikować dane repozytorium, by wiedzieć czy je mogę dodać do systemu, czy nie. I jeśli
ktoś by podmienił klucz GPG w usłudze `extrepo` , to w żaden sposób nie wpłynie to na mój system,
tj. on dalej będzie korzystał z tego klucza, który ja mu wskażę ręcznie.

Już wielokrotnie przytrafiła mi się sytuacja, w której, np. klucz wygasł i trzeba było ustalić co
się z takim repozytorium dzieje oraz czy ono w ogóle żyje i rokuje na przyszłość, albo czy można
sobie z nim dać spokój i poszukać jakiejś alternatywy dla oprogramowania, które to repo dostarczało.

## Podsumowanie

Jakby nie patrzeć, odejście Debiana od narzędzia `apt-key` mającego na celu dodawanie kluczy GPG do
jednego zbiorczego pliku keyring'a APT, to krok w dobrą stronę. Trzeba jednak mieć na uwadze, że
ten nowy system ogarniania kluczy do zewnętrznych repozytoriów nie zapewnia jakiejś dodatkowej
ochrony systemu. Kluczowy z perspektywy bezpieczeństwa naszego linux'a był, jest i zawsze będzie sam
proces instalowania pakietu `.deb` z takiego repozytorium. Skrypty maintainer'a są w stanie wykonać
dowolny kod i to jako uprzywilejowany użytkownik root. Jeśli nie zweryfikujemy tego, co te skrypty
dokładnie robią podczas instalacji pakietu, to żadne zabezpieczenia nas nie uchronią przed
kompromitacją naszego systemu.


[1]: https://salsa.debian.org/extrepo-team/extrepo
[2]: https://salsa.debian.org/extrepo-team/extrepo-data/-/tree/master/repos/debian
[3]: https://www.debian.org/releases/bullseye/amd64/release-notes/ch-information.en.html
[4]: https://signal.org/download/linux/
[5]: https://blog.jak-linux.org/2021/06/20/migrating-away-apt-key/
[6]: https://salsa.debian.org/apt-team/apt/-/merge_requests/176
