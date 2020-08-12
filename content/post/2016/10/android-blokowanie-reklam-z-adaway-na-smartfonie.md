---
author: Morfik
categories:
- Android
date: "2016-10-30T16:21:42Z"
date_gmt: 2016-10-30 15:21:42 +0100
published: true
status: publish
tags:
- smartfon
- adblock
- aplikacje
title: 'Android: Blokowanie reklam z AdAway na smartfonie'
---

Dzięki [dnscrypt-proxy](https://dnscrypt.org/) jesteśmy w stanie [zaszyfrować zapytania
DNS]({{< baseurl >}}/post/jak-zaszyfrowac-zapytania-dns-na-smartfonie-dnscrypt-proxy/)
bezpośrednio na naszych smartfonach. Niemniej jednak, w przypadku mojego Neffos'a C5 od TP-LINK, w
wielu aplikacjach pojawiły się reklamy po wdrożeniu mechanizmu szyfrującego. Wcześniej oczywiście
wykorzystywałem [adblock'a bezpośrednio na routerze z wgranym firmware
OpenWRT/LEDE]({{< baseurl >}}/post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/), gdzie
zapytania DNS do adserwerów były filtrowane i blokowane bezpośrednio na tym urządzeniu. Po
zaszyfrowaniu ruchu DNS w telefonie, straciłem dostęp do mojego filtra reklam na routerze.
Przydałoby się zatem zaimplementować podobny mechanizm blokujący bezpośrednio na Androidzie, tak by
ponownie wszystkie te reklamy zniknęły przy jednoczesnym zachowaniu całej funkcjonalności płynącej
za sprawą szyfrowanego ruchu DNS. Jednym z rozwiązań jest wykorzystanie [narzędzia
AdAway](https://adaway.org/), które przy pomocy pliku `/etc/hosts` i lokalnego serwera www jest w
stanie zablokować sporą większość reklam, na które możemy natknąć się w internecie. Opis instalacji
i konfiguracji AdAway zostanie przedstawiony w niniejszym wpisie.

<!--more-->
## Root Androida na potrzeby AdAway

AdAway niestety wymaga ukorzenionego Androida (dostęp root). Chodzi o to, że to narzędzie operuje na
pliku `/system/etc/hosts` oraz musi być też w stanie uruchomić serwer www, który ma nasłuchiwać na
porcie 80. Do tych celów są wymagane prawa administratora systemu. Jeśli nie wiemy jak je zdobyć w
przypadku naszego telefonu, to AdAway nie jest dla nas. Dla tych którzy posiadają ten sam model
smartfona co i ja, tj. [Neffos C5]({{< baseurl >}}/post/recenzja-smartfon-neffos-c5-od-tp-link/)
od TP-LINK, mogą przejść przez [proces root'owania systemu na tym
telefonie]({{< baseurl >}}/post/android-root-smartfona-neffos-c5-od-tp-link/).

## Brak AdAway w Google play oraz instalacja z F-Droid

W sklepie Google Play można się natknąć na wiele narzędzi, które mają realizować zadanie blokowania
reklam. Nie znajdziemy tam jednak AdAway. Zgodnie z tym co możemy wyczytać na stronie projektu,
[AdAway został ze sklepu Google Play usunięty przez naruszenie
umowy](https://play.google.com/about/developer-distribution-agreement.html) (punkt 4.4). Wygląda na
to, że Google wywala z tego sklepu wszystkie niewygodne aplikacje. Oczywiście AdAway w dalszym ciągu
możemy pobrać, z tym, że trzeba skorzystać z [alternatywnego repozytorium jakim jest
F-Droid]({{< baseurl >}}/post/android-repozytorium-aplikacji-opensource-f-droid/).

[![001.adaway-blokowanie-reklam-smartfon-android-instalacja-f-droid]({{< baseurl >}}/img/2016/10/001.adaway-blokowanie-reklam-smartfon-android-instalacja-f-droid-660x361.png)]({{< baseurl >}}/img/2016/10/001.adaway-blokowanie-reklam-smartfon-android-instalacja-f-droid.png)

## Źródła plików hosts

Przede wszystkim, interesują nas [źródła pliku
hosts](https://github.com/AdAway/AdAway/wiki/HostsSources). Warto tutaj zaznaczyć, że możemy
korzystać z kilku źródeł jednocześnie. W takim przypadku, kilka plików `hosts` zostanie ze sobą
połączonych, a zduplikowane wpisy zostaną usunięte. Jesteśmy w stanie również korzystać z własnych
plików `hosts` , czy dodawać niestandardowe źródła bezpośrednio w opcjach
aplikacji.

[![002.adaway-blokowanie-reklam-smartfon-android-zrodla-hosts]({{< baseurl >}}/img/2016/10/002.adaway-blokowanie-reklam-smartfon-android-zrodla-hosts-660x271.png)]({{< baseurl >}}/img/2016/10/002.adaway-blokowanie-reklam-smartfon-android-zrodla-hosts.png)

Przy dodawaniu źródeł plików `hosts` trzeba zdawać sobie sprawę, że wraz ze zwiększaniem się ilości
wpisów w tym pliku, potrzebna jest większa moc obliczeniowa, która będzie w stanie te informacje
przetworzyć z każdym zapytaniem DNS. Jest zatem wielce prawdopodobne, że czas pracy na baterii
naszego telefonu ulegnie skróceniu w zależności od intensywności wykorzystywania internetu.

Kolejną sprawą jest zablokowanie pewnych usług czy aplikacji przez zbyt eksensywne filtrowanie
reklam. Trzeba mieć na uwadze, że plik `hosts` to nic innego jak mapa adresów IP i domen, taki
tekstowy lokalny serwer DNS. W tym przypadku interesują nas co prawda domeny adserwerów ale niektóre
z nich mogą hostować również inne zasoby. Blokując kontakt z tymi domenami, zablokujemy także
pozostałe zasoby. Dlatego do kwestii blokowania reklam trzeba podejść z należytą rozwagą.

## Czarne i białe listy adserwerów

Może się zdarzyć też tak, że bardzo przypadnie nam do gustu jeden z plików `hosts` , który
znajdziemy sobie gdzieś w internecie. Niemniej jednak, po dodaniu go jako źródło adserwerów
przestaną nam działać pewne połączenia czy usługi. Jednym z wyjść byłoby usunięcie tego pliku
`hosts` ze źródeł ale to rozwiązanie niekoniecznie może w naszym przypadku wchodzić w grę. Na
szczęście AdAway wspiera czarne i białe listy domen, które możemy ręcznie zablokować lub
odblokować już po zaaplikowaniu plików `hosts` z
adserwerami.

[![003.adaway-blokowanie-reklam-smartfon-android-biala-czarna-lista]({{< baseurl >}}/img/2016/10/003.adaway-blokowanie-reklam-smartfon-android-biala-czarna-lista-660x271.png)]({{< baseurl >}}/img/2016/10/003.adaway-blokowanie-reklam-smartfon-android-biala-czarna-lista.png)

Trzeba jednak pamiętać, że po dodaniu adresu, zmiany nie są natychmiastowe. Trzeba ręcznie odświeżyć
listę w pliku `hosts` .

## Logowanie zapytań DNS

Aplikacje mogą nie działać prawidłowo w przypadku, gdy komunikacja z pewnymi domenami jest
zablokowana. Jeśli nie wiemy z jakich adresów korzysta taki problematyczny program, to możemy
spróbować to ustalić włączając logowanie zapytań DNS. Wszystkie adresy domen, które nasz smartfon
próbuje odwiedzić zostaną zalogowane. My zaś w łatwy sposób będziemy w stanie zablokować lub
odblokować konkretne wpisy w zależności od
potrzeb:

[![004.adaway-blokowanie-reklam-smartfon-android-logowanie-dns]({{< baseurl >}}/img/2016/10/004.adaway-blokowanie-reklam-smartfon-android-logowanie-dns-660x542.png)]({{< baseurl >}}/img/2016/10/004.adaway-blokowanie-reklam-smartfon-android-logowanie-dns.png)

## Pozostałe opcje AdAway

Poza tymi wymienionymi wyżej opcjami, AdAway ma jeszcze kilka dodatkowych ustawień. Możemy w nich
zmienić zachowanie mechanizmu aktualizacji pliku `host` . Jest też opcja określenia adresu IP, na
który mają być przesyłane żądania do adserwerów. Można również włączyć opcję przekierowania domen
na inne adresy IP, choć może to stwarzać zagrożenie dla
bezpieczeństwa.

[![005.adaway-blokowanie-reklam-smartfon-android-ustawienia]({{< baseurl >}}/img/2016/10/005.adaway-blokowanie-reklam-smartfon-android-ustawienia-660x361.png)]({{< baseurl >}}/img/2016/10/005.adaway-blokowanie-reklam-smartfon-android-ustawienia.png)

## Czy AdAway blokuje reklamy

Po dostosowaniu źródeł pliku `hosts` i ewentualnym ustawieniu dodatkowych wpisów na białej i czarnej
liście, przydałoby się sprawdzić czy AdAway w ogóle działa. By uniknąć problemów za sprawą cache,
dobrze jest uruchomić smartfon ponownie. Praktycznie przy każdej aktualizacji pliku `hosts` będziemy
proszeni o tę
czynność:

[![006.adaway-blokowanie-reklam-smartfon-android-aktualizacja]({{< baseurl >}}/img/2016/10/006.adaway-blokowanie-reklam-smartfon-android-aktualizacja-660x361.png)]({{< baseurl >}}/img/2016/10/006.adaway-blokowanie-reklam-smartfon-android-aktualizacja.png)

Po zrestartowaniu Androida warto sprawdzić czy plik `/system/etc/hosts` zawiera jakieś wpisy.
Standardowo są tam tylko tylko linijki z localhost. Jeśli aktualizacja pliku `hosts` przebiegła
pomyślnie, to ten plik powinien być nieco
większy:

[![007.adaway-blokowanie-reklam-smartfon-android-test-hosts]({{< baseurl >}}/img/2016/10/007.adaway-blokowanie-reklam-smartfon-android-test-hosts-660x401.png)]({{< baseurl >}}/img/2016/10/007.adaway-blokowanie-reklam-smartfon-android-test-hosts.png)

Wystarczy teraz odwiedzić jedną z domen, które są na liście, np. w przeglądarce internetowej. Jeśli
zostanie nam zwrócona biała strona, oznacza to, że mechanizm blokowania reklam działa należycie.
Większość albo i wszystkie reklamy, które są wyświetlane w aplikacjach dostępnych w Google Play,
również powinny zniknąć.
