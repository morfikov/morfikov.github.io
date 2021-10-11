---
author: Morfik
categories:
- Linux
date: "2015-12-01T17:57:39Z"
date_gmt: 2015-12-01 16:57:39 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- apparmor
GHissueID: 210
title: Profil AppArmor'a i jego dokładna budowa
---

[Budowanie samych profili dla AppArmor'a](/post/apparmor-profilowanie-aplikacji/)
nie jest jakoś szczególnie trudne, zwłaszcza, gdy do tego celu wykorzystujemy narzędzia dostępne w
pakiecie `apparmor-utils` . Dobrze jest jednak prześledzić sobie
[manual](http://manpages.ubuntu.com/manpages/xenial/en/man5/apparmor.d.5.html)
[AppArmor'a](http://manpages.ubuntu.com/manpages/xenial/en/man7/apparmor.7.html) oraz to, co twórcy
piszą na swojej stronie w [oficjalnej
dokumentacji](http://wiki.apparmor.net/index.php/Documentation) projektu. Poniższy wpis powstał w
celu zrozumienia składni profili AppArmor'a, tak by jeszcze bardziej uprościć proces ich budowania.

Opis składni znajdujący się poniżej został zaczerpnięty z
[wiki](http://wiki.apparmor.net/index.php/QuickProfileLanguage)
[AppArmor'a](http://wiki.apparmor.net/index.php/AppArmor_Core_Policy_Reference). Część z poniższych
informacji może nie mieć zastosowania w przypadku starszych wersji samego AppArmor'a.

<!--more-->
## Przykładowy plik profilu AppArmor'a

Profil AppArmor'a zawiera reguły, które są wymuszane, gdy jest uruchamiany powiązany z nimi program.
Te reguły, to taka biała lista praw dostępu do plików czy urządzeń, które mogą narzucić aplikacji
poruszanie się w pewnym określonym obszarze systemu.

Plik profilu jest podzielony na dwie sekcje: preambułę oraz profil właściwy. Preambuła musi znaleźć
się na początku pliku. W przypadku, gdy w dalszej części pliku zostanie określona jakaś sekcja
profilu, kolejne reguły preambuły nie mogą już być dodawane niżej w tym pliku. W przypadku, gdy
polityka AppArmor'a jest rozdzielona na szereg plików, każdy z nich może mieć własną sekcję dla
preambuły. Te pliki, które będziemy załączać w profilu (przy pomocy `#include` ), nie mogą posiadać
sekcji preambuły.

### Sekcja preambuły

Preambuła składa się z reguł, które mają globalny wpływ na zdefiniowane niżej w danym pliku sekcje
profili. Ogólnie rzecz biorąc, te reguły, które mogą zostać wykorzystane w sekcji preambuły nie mogą
pojawić się w sekcji profilu. Są to: definicje zmiennych, aliasy, reguły, które przepisują/zmieniają
(rewrite) coś, reguły warunkowe oraz także załączanie plików z regułami, które nie mogą być
definiowane bezpośrednio w obrębie sekcji profilu AppArmor'a.

Poniżej przykład preambuły:

    #include <tunables/global>

    @{MOZ_LIBDIR}=/opt/firefox
    ...

Mamy tutaj jedną dyrektywę `#include` , która definiuje (przy pomocy innych plików) szereg
zmiennych, które powinny być dostępne dla każdego profilu. Niżej zaś jest linijka ze zmienna
`MOZ_LIBDIR` , która może zostać wykorzystana w sekcji profilu.

#### Definiowanie zmiennych

W sekcji preambuły mogą także pojawiać się zmienne, które są definiowane przez `@{nazwa_zmiennej}` ,
tak jak to widać powyżej. Jest szereg zmiennych, które AppArmor ma już zdefiniowane, a właściwie są
one określone w plikach w katalogu `/etc/apparmor.d/tunables/` . Z reguły każdy profil w sekcji
preambuły powinien mieć zdefiniowaną jako pierwszą tę poniższą linijkę:

    #include <tunables/global>

W pliku `global` są zawarte kolejne dyrektywy `#include` , które pociągają szereg dodatkowych plików
z regułami.

#### Reguły rewrite i aliasy

Różnica między regułami rewrite i aliasami jest dość subtelna. Reguła rewrite przepisuje oryginalną
ścieżkę do pliku/katalogu/urządzenia i jest aplikowana jedynie do tego nowego zasobu i nie ma przy
tym możliwości odwołania się do starej ścieżki. Z kolei alias dodaje do starej ścieżki nową i obie z
nich są brane pod uwagę przy aplikowaniu pasujących reguł. Przykładowo, weźmy sobie ten poniższy
alias:

    alias /home/ -> /mnt/users/

Jeśli teraz byśmy dodali regułę w stylu tej poniższej:

    /home/foo rw,

to aplikacja będzie w stanie dokonać zapisu i odczytu nie tylko w przypadku pliku `/home/foo` ale
również i pliku `/mnt/users/foo` , czyli dokładnie taka sama sytuacja co w przypadku dodania
poniższej reguły:

    /mnt/users/foo rw,

Z kolei reguła rewrite wyglądałaby bardzo podobnie:

    rewrite /home/ -> /mnt/users/

Reguły rewrite oraz aliasty są brane pod uwagę dopiero po rozwiązaniu nazw zmiennych.

### Sekcja profilu

Sekcja profilu AppArmor'a składa się z jednego lub więcej profili. Ich kolejność nie ma większego
znaczenia, natomiast przyjęło się by główny profil był określany jako pierwszy w pliku. Każdy profil
AppArmor'a składa się z nagłówka, który określa nazwę profilu, po której zaś jest opcjonalne pole na
flagi, które sterują zachowaniem samego profilu. Ponadto, każdy profil posiada swoje ciało, które
jest ujęte w `{ }` , gdzie definiowane są wszystkie reguły dotyczące zezwoleń. Kolejność samych
reguł w profilu nie ma większego znaczenia, natomiast wszystkie z nich są oddzielone od siebie przy
pomocy `,` .

Nazwa profilu powinna zaczynać się od `/` . W przypadku, gdy nazwa się nie rozpoczyna od `/` , przed
nazwą profilu dodajemy słowo `profile` . Nazwa profili nie może jednak być zakończona znakiem `/` .
Nazwy też nie powinny zawierać spacji ale jeśli zawierają, to muszą być one poprzedzane znakiem `\`
lub też cała nazwa profilu powinna być ujęta w `" "` . Nazwy nie mogą także rozpoczynać się od `.` ,
`:` lub `+` , choć te znaki mogą występować gdzieś w środku nazwy.

Nazwy profili rozpoczynające się od `/` są wykorzystywane przy dopasowaniu profilu do plików
wykonywalnych, które taki profil ma ograniczać. Natomiast, nazwy profili, które nie rozpoczynają się
od `/` (zwykle te ze słówkiem `profile`) są używane jedynie w przypadku wywoływania innych aplikacji
z danego profilu. Zwykle używa się tego mechanizmu, gdy jakiś plik wykonywalny nie powinien być
ograniczony w systemie jako takim, natomiast powinien posiadać profil, gdy jest wywoływany przez
ograniczoną już aplikację. Przykładowo, narzędzie `grep` . Nie powinno ono mieć profilu jako takiego
ale użyteczny może okazać się wyspecjalizowany profil, który będzie aplikowany tylko w przypadku
wywoływania ze skryptu, który podlega ograniczeniu przez AppArmor.

Poniżej kilka przykładów poprawnych nazw profili:

    /bin/example
    /bin/example\ name
    "/bin/example name"
    @{var}
    @{var}/example
    "@{var}/example"
    "/bin/@{var}"
    profile /bin/example
    profile example
    profile "example name"
    profile "example\ name"
    /bin/example\,
    "/bin/example,"
    profile example /bin/example

Poniżej zaś przykłady NIEpoprawnych nazw profili:

    /bin/example/          # zakończony /
    /bin/example,          # zakończony przecinkiem, który nie jest escape'owany lub ujęty w cudzysłów
    /bin/example name      # dwie osobne nazwy zamiast jednej
    "/bin/example name     # na końcu brakuje cudzysłowu
    :profile               # zaczyna się od :
    +profile               # zaczyna się od +

Nie ma także większego powodu by każdy profil był umieszczany w osobnym pliku. Równie dobrze można
to zrobić w jednym ale przyjęto taką zasadę i dobrze jest jej się trzymać.

#### Flagi profilu

Jak zostało wspomniane wyżej, pole na flagi jest pozycją opcjonalną i w przypadku, gdy nie zostaną
określone żadne flagi, domyślne z nich są aplikowane i są to: `enforce`, `namespace_relative`,
`no_attach_disconnected` , `chroot_no_attach` oraz `v2` .

Poniżej znajduje się lista flag, które można wykorzystać:

  - `policy version flags` -- do wyboru `v2` lub `v3` , w zależności od wersji AppArmor'a.
  - `audit` -- wymusza logowanie wszelkich prób dostępu, bez względu na to czy są one dozwolone czy
    też i nie. Ta flaga generuje bardzo dużo komunikatów i zaleca się audytować tylko określone
    reguły wewnątrz profilu.
  - `debug` -- póki co ta flaga nie jest wspierana, niemniej jednak, włącza ona wypisywanie
    dodatkowych informacji.
  - `mode flag` -- określa tryb w jakim działa profil. Każdy z tych trybów się wzajemnie wyklucza.
    Te flagi są już nieco przestarzałe i zamiast nich powinno się korzyść z kontrolowanego stanu
    profili (Controlling Profile State). Niemniej jednak są do dyspozycji cztery opcje: `enforce` ,
    `complain` , `debug` oraz `kill` . Trzy pierwsze raczej nie powinny sprawiać problemu, natomiast
    tryb `kill` ma na celu ubijanie zadań, które próbują uzyskać nieuprawniony dostęp do zasobów
    systemowych oraz logowanie tego typu zdarzeń.
  - `relative flags` -- te flagi wzajemnie się wykluczają i kontrolują to w jaki sposób dokonuje się
    rozwiązywanie nazw ścieżek do plików dla danego profilu. W przypadku, gdy `chroot_relative`
    został określony, nazwy ścieżek są przeszukiwane względem bazy chroot, zaś nazwy zewnętrze w
    stosunku do chroot są ustalane przez flagi attach. Inną flagą do wyboru jest
    `namespace_relative` i jest ona bardzo podobna do tej poprzedniej, z tą różnicą, że nazwy
    wewnątrz środowiska chroot są rozwiązywane absolutnie w stosunku do całej przestrzeni nazw,
    natomiast zewnętrzne nazwy ścieżek są kontrolowane przez flagi attach.
  - `attach flags` -- te flagi kontrolują w jaki sposób dokonywana jest obsługa ścieżek odłączonych.
    Do wyboru są cztery różne flagi. Pierwsze dwie, tj. `attach_disconnect` oraz
    `no_attach_disconnected` wzajemnie się wykluczają i określają to czy nazwy ścieżek rozwiązane na
    te będące spoza przestrzeni nazw są poprzedzane znakiem `/` . Nie jest to dobry pomysł ponieważ
    zezwala to na dokonanie aliasu takiej odłączonej ścieżki, który może wskazywać na inny plik. Z
    kolei ostatnie dwie flagi to: `chroot_attach` oraz `chroot_no_attach` i one również się
    wzajemnie wykluczają. Kontrolują zaś to w jaki sposób są generowane nazwy ścieżek wewnątrz
    środowiska chroot w chwili, gdy następuje odwołanie do jakiegoś pliku, który jest zewnętrzny
    dla środowiska chroot ale nie dla samej przestrzeni nazw. Przy `chroot_attach_disconnected`
    nazwy ścieżek są poprzedzane przy pomocy `/` , w przeciwnym wypadku pozostają w formie
    odłączonej.

Poniżej przykłady poprawnego wykorzystania flag:

    flags=(enforce)
    flags=(complain)
    flags=(enforce debug)
    (complain chroot_relative)

### Komentarze

Komentarze są oznaczone za pomocą znaku `#` i może on znajdować się przed danym wyrażeniem lub też i
po nim. W obu przypadkach treść występująca po znaku `#` jest ignorowana, przykładowo:

    /usr/bin/firefox {  # komentarz
          # komentarz
          /home/foo rw,  # komentarz
    }

### Załączanie plików i katalogów

W pliku profilu może także znajdować się dyrektywa `#include` , która ma na celu dołączyć zewnętrzne
pliki z regułami, co z kolei ułatwia budowanie i późniejsze zarządzanie tworzonym profilem. Trzeba
dodać, że sam znak `#` jest opcjonalny i nie powinno się go używać przy budowaniu nowych profili.
Jeśli jednak zdecydujemy się na dodanie go, to nie może on być oddzielony przy pomocy spacji, bo
wtedy taka linijka będzie traktowana jak zwykły komentarz.

Pliki możemy załączać na kilka sposobów. Pierwszy z nich zakłada wykorzystanie ścieżek relatywnych
względem katalogu `/etc/apparmor.d/` i w takim przypadku ścieżkę musimy ująć w `< >` . Możemy także
określić położenie względem pliku profilu lub też podać absolutną ścieżkę w systemie plików. W obu
ostatnich przypadkach, ścieżkę dajemy w `" "` , przykładowo:

    #include <abstractions/base>

    #include "somefile"
    #include "../file_from_parent_directory"

    #include "/etc/apparmor.d/include/foo"

    include <abstractions/base>

Dyrektywa `#include` może także zostać wykorzystana do załączenia wszystkich plików ze wskazanego
folderu w przypadku, gdy ścieżka wskazuje na katalog. Ten mechanizm ma jednak kilka limitów. Po
pierwsze, jedynie pliki w danym katalogu zostaną uwzględnione. Nie nastąpi za to przeszukanie
podkatalogów i ich pliki nie zostaną załączone. Poza tym, wszystkie pliki zaczynające się od `.` nie
zostaną uwzględnione, podobnie jak i pliki, które kończą się na: `.dpkg-new` , `.dpkg-old` ,
`.dpkg-dist` , `.dpkg-bak` , `.rpmnew` , `.rpmsave` oraz `~` .

W przypadku dyrektyw `#include` nie stawiamy `,` na końcu linijki, tak jak to robimy w przypadku
pozostałych reguł.

### Możliwości (Capabilities)

W plikach profili AppArmor'a mamy także możliwość określenia kontroli dla
[capabilities](http://manpages.ubuntu.com/manpages/xenial/en/man7/capabilities.7.html). Takie reguły
zaczynają się od słowa `capability` , po którym zwykle umieszcza się nazwę możliwości z dodatkową
informacją (obecnie jeszcze niewspierane), lub też wiele możliwości oddzielonych przy pomocy spacji.
Na końcu tych reguł musi pojawić się `,` . W podlinkowanym wyżej manualu jest lista możliwości,
które możemy wykorzystać przy budowaniu profilu. Mamy tam jednak opcje typu `CAP_SETGID` . By móc
użyć ich w profilu AppArmor'a, usuwamy `CAP` , a całą resztę piszemy z małych liter.

Normalnie, AppArmor jedynie ogranicza te możliwości aplikacji, które ona już posiada, czyli niejako
maskuje je. W taki sposób, jeśli mamy do czynienia z programem, który korzysta z bitu setuid/setgid
i jest to pożądane, to musielibyśmy taką opcję uwzględnić w profilu samej aplikacji, co
sprowadzałoby się do dodania poniższego kodu do pliku profilu:

    capability setuid,
    capability setgid,

AppArmor jest też w stanie nadawać możliwości aplikacjom i do tego celu służy `set capability` . Ze
względów bezpieczeństwa, te ustawione reguły dotyczące możliwości aplikacji nie są dziedziczone. Gdy
tylko dany program wychodzi poza obręb profilu, traci te nadane przez AppArmor możliwości. Nie
trzeba także osobno precyzować reguły, która miałaby na celu zezwolenie na wykorzystanie takiej
możliwości, bo `set` automatycznie implikuje to zachowanie.

Samo nadanie aplikacji możliwości nie zawsze wystarcza. Trzeba także uwzględnić ewentualne prawa
dostępu do samego pliku, a to musi być określone przez dodatkową regułę.

### Sieć

Dostęp do sieci w przypadku pewnych aplikacji także może podlegać obostrzeniom. Możemy pozbawić ich
dostępu do internetu zupełnie, lub też kontrolować tworzenie soketów jak i przepływ danych
(pakietów). Tego typu mechanizm może robić za swojego rodzaju firewall lub też być dodatkiem do
firewall'a systemowego. Jeśli chcemy aby aplikacja miała dostęp do internetu i nie interesują nas
zbytnio detale, to potrzebowalibyśmy dopisać w pliku profilu `network` , przykładowo:

    network,

Jeśli nie chcemy stosować tak ogólnej reguły, zawsze możemy skorzystać z następujących uprawnień:
create, accept, bind, connect, listen, read, write, send, receive, getsockname, getpeername,
getsockopt, setsockopt, fcntl, ioctl, shutdown, getpeersec. Poniżej, krótki opis:

  - `create` -- zezwala na tworzenie gniazd o określonej domenie, typie i protokole.
  - `shutdown` -- zezwala na zamknięcie gniazda.
  - `listen[=X]` -- zezwala na nasłuchiwanie na gnieździe o określonej domenie, typie i protokole.
    Jeśli zostanie także zdefiniowany dodatkowy parametr `backlog`, to zostanie [ograniczone tempo
    nawiązywania nowych
    połączeń](http://www.linuxjournal.com/files/linuxjournal.com/linuxjournal/articles/023/2333/2333s2.html).
  - `bind` -- zezwala na przypisanie adresu, który pasuje do wyrażenia adresu źródłowego. Jeśli nie
    zostanie określony, zostanie przypisany do dowolnego adresu.
  - `connect` -- zezwala na połączenie gniazda z adresem, który pasuje do wyrażenia adresu
    docelowego. Jeśli nie zostanie określony, połączenie będzie możliwe z dowolnym adresem.
  - `accept` -- zezwala na akceptowanie połączeń z adresu, który pasuje do wyrażenia adresu
    docelowego. Jeśli nie zostanie określony, połączenie będą akceptowane z dowolnego adresu.
  - `read`/`receive` -- zezwala na czytanie/odbieranie pakietów i danych pochodzących z adresu
    docelowego.
  - `write`/`send` -- zezwala na zapis/wysyłąnie pakietów i danych z adresu źródłowego do adresu
    docelowego.
  - `getname` -- zezwala na uzyskanie adresu gniazda.
  - `getpeer` -- zezwala na uzyskanie zdalnego adresu gniazda.
  - `setopt` -- zezwala na ustawienie opcji dla gniazda.

Zatem bardziej restrykcyjny dostęp do sieci może wyglądać mniej więcej tak:

    network (read, write) inet,
    deny network bind inet,

Wyżej mamy podaną domenę `inet` , która odpowiada ipv4 , z kolei `inet6` jest dla ipv6. W pewnych
przypadkach nie musimy określać domeny, bo ta może być implikowana automatycznie na postawie typu,
protokołu czy samej adresacji, przykładowo:

    network tcp,
    # implikuje
    network inet stream,
    network inet6 stream,

Niektóre typy wymagają innych uprawnień niż te z `network` , przykładowo, `raw` potrzebuje także
odpowiedniej możliwości, tj. `cap_raw` . Podobnie jak w przypadku domeny, typu też czasem nie trzeba
określać, bo może być implikowany przez protokół i adresację, przykładowo:

    network inet tcp,
    # implikuje
    network inet stream,

Protokóły, które mogą pojawić się w regułach zależą od domeny i typów samych reguł. Adres zaś zależy
od typu domeny i gniazda. W przypadku, gdy nie odpowiadają nam ogóle reguły, które nie są w stanie
często ograniczyć się do określonego wyrażenia, możemy rozbić je na szereg mniejszych reguł,
podobnych do tej poniżej:

    network tcp src 192.168.1.1:80 dst 170.1.1.0:80,

Poniżej jest kilka przykładów, na temat tego jaką formę mogą przyjąć dopasowania dla protokołu ipv4:

    192.168.1.1
    192.168.1.1-254
    192.168.1.1:80
    192.168.1.1:80,81
    192.168.1.1:80,81,100-200

    192.168.1.*
    192.168.1.0-255
    192.168.1.0/24
    192.168.1.0/255.255.255.0

    192.0-255.1.1
    192.*.1.1
    192.0.1.1/255.0.255.255

### Prawa dostępu do plików

Każdy profil posiada reguły kontrolujące dostęp do określonych plików czy katalogów w systemie.
Składają się one ze ścieżki, uprawnień i dzielnika w postaci `,` . Tego typu reguły mogą być pisane
zarówno z uprawnieniami przed jak i po ścieżce do pliku, choć zwykle ścieżka jest podawana jako
pierwsza. Dodatkowo, poprawna ścieżka zawsze zaczyna się od `/` , przykładowo:

    /sciezka/do/pliku  rw,   # reguła zaczynająca się od ścieżki
    rw /sciezka/do/pliku2,   # reguła zaczynająca się od uprawnień

    /sciezka/do/pliku3       # regułą podzielona na kilka linii
       rw,

Kontrola dostępu do plików odbywa się za pomocą specjalnych oznaczań (dodawanych zaraz za ścieżką).
Mamy możliwość skorzystania z krótkich lub długich form. Długie formy są wypisane poniżej:

  - `create` -- możliwość tworzenia pliku.
  - `open` -- możliwość otwarcia pliku.
  - `delete` -- możliwość skasowania pliku.
  - `rename` -- możliwość zmiany nazwy pliku (przeniesienie) bez posiadania prawa do zapisu lub
    odczytu tego pliku.
  - `read` -- możliwość odczytu pliku.
  - `getattr` -- możliwość odczytu atrybutów pliku.
  - `getxattr` -- możliwość odczytu rozszerzonych atrybutów pliku.
  - `write` -- możliwość zapisu danych do pliku, włączając w to jego rozszerzanie (dodawanie nowych
    danych). Automatycznie implikuje `append` .
  - `append` -- możliwość dodawania nowych danych do pliku bez możliwości nadpisania tych starych.
    Używany w połączeniu z otwieraniem przez aplikację pliku przy pomocy O\_APPEND.
  - `trunc` -- możliwość skracania (przycinania) pliku.
  - `setattr` -- możliwość zapisu szeregu metadanych plików, tych niekontrolowanych przez
    wyspecjalizowane flagi.
  - `setxattr` -- możliwość zapisu rozszerzonych atrybutów pliku.
  - `chmod` -- możliwość zmiany uprawnień do pliku.
  - `chown` -- możliwość zmiany właściciela pliku.
  - `chgrp` -- możliwość zmiany grupy pliku.
  - `link` -- możliwość linkowania do pliku. Gdy użyty z `**` określa czy uprawnienia są aplikowane
    do całego poddrzewa plików i katalogów.
  - `snapshot` -- możliwość tworzenia migawek (snapshot) plików lub katalogów.
  - `lock` -- możliwość zablokowania pliku.
  - `mmap_r` -- oznaczenie pliku jako możliwy do odczytu przy pomocy mmap
  - `mmap_w` -- oznaczenie pliku jako możliwy do zapisu przy pomocy mmap
  - `mmap_x` -- oznaczenie pliku jako możliwy do wykonania przy pomocy mmap
  - `mprot_wx` -- zezwala na przejście z `w` do `x` .
  - `mprot_xw` -- zezwala na przejście z `x` do `w` .
  - `exec` -- możliwość wykonania pliku.

Poniżej zaś są wypisane krótkie formy:

  - `r` -- zezwala programowi na odczytanie pliku lub listing katalogu.
  - `w` -- zezwala programowi na zapis do pliku. Pliki i katalogi muszą mieć to uprawnienie by można
    było je usunąć, nie jest ono jednak wymagane w przypadku zmiany nazwy lub przy tworzeniu plików
    wewnątrz katalogu.
  - `a` -- zezwala programowi tylko na dodawanie treści do pliku. Ta opcja konfliktuje z `w` .
  - `m` -- zezwala plikowi na bycie zmapowanym w pamięci przy pomocy flagi PROT\_EXEC mmap'a. Ta
    flaga oznacza strony pamięci jako wykonywalne. AppArmor wykorzystuje ten tryb by wskazać
    aplikacji, które pliki mogą być wykorzystywane jako biblioteki.
  - `l` -- zezwala programowi na tworzenie dowiązań.
  - `k` -- pozwala programowi na zablokowanie dostępu do pliku.

Poza tymi powyższymi, mamy jeszcze jedną opcję: `x` , która zezwala profilowanemu programowi na
uruchamianie innych aplikacji. W przypadku tego parametru mamy kilka różnych sposobów dotyczących
tego jak dany program ma być wykonany i sam `x` będzie poprzedzony dodatkowym znakiem. W zapisie
może pojawić się wielka litera, co oznacza, że zmienne środowiskowe (np. LD\_PRELOAD) nie zostaną
zachowane przy wykonywaniu danego programu. Z kolei, jeśli wszystkie znaki będą pisane z małych
liter, środowisko zostanie zachowane. AppArmor czyści środowisko przy pomocy kernelowskiego
`unsafe_exec` podobnie jak w przypadku programów z ustawionym bitem setuid. Trzeba jednak brać pod
uwagę fakt, że czasem tak wykonane aplikacje mogą nie działać jak należy.

Poniżej lista trybów wykonania:

  - `ux/Ux` -- ten tryb zezwala programowi na uruchomienie innego programu bez jakichkolwiek
    ograniczeń ze strony AppArmor'a.
  - `px/Px` -- ten tryb wymaga by uruchamiany program posiadał profil bezpieczeństwa i w przypadku
    jego braku, aplikacja nie zostanie uruchomiona.
  - `cx/Cx` -- ten tryb wymaga by uruchamiany program posiadał lokalny profil bezpieczeństwa i w
    przypadku jego braku, aplikacja nie zostanie uruchomiona.
  - `ix` -- dziedziczenie profilu przy wykonywaniu programu.

Mamy także do dyspozycji kilka dodatkowych trybów rezerwowych (fallback), w których to profile będą
aplikowane jedynie w przypadku ich odnalezienia:

  - `pix/Pix` -- to samo co w przypadku `px/Px` , z tym, że jeśli dedykowany profil nie zostanie
    odnaleziony, to będzie dziedziczony ten, który ma profilowana aplikacja.
  - `pux/Pux` -- to samo co w przypadku `px/Px` , z tym, że w jeśli dedykowany profil nie zostanie
    odnaleziony, wykonanie kodu będzie możliwe bez jakichkolwiek ograniczeń.
  - `cix/Cix` -- to samo co w przypadku `cx/Cx` , z tym, że jeśli lokalny profil nie zostanie
    odnaleziony, będzie on dziedziczony od profilowanej aplikacji.
  - `cux/Cux` -- to samo co w przypadku `cx/Cx` , z tym, że jeśli lokalny profil nie zostanie
    odnaleziony, kod zostanie wykonany bez jakichkolwiek ograniczeń.

Poniżej jest zawarte mapowanie krótkich nazw na długie, tak by w prosty sposób można było
zinterpretować w logu AppArmor'a potrzebne uprawnienia:

  - `r` -- read, meta-read, mmap\_r
  - `w` -- create, delete, trunc, write, meta-write, chmod, chown, mmap\_w, mprot\_wx, partial
    rename
  - `a` -- append, create
  - `l` -- link
  - `k` -- lock
  - `m` -- mmap\_x, mprot\_wx
  - `x` -- exec, (w przypadku `ix` również mmap\_x)

Istnieją dwa sposoby na wywoływanie programów w profilowanej aplikacji. Pierwszym z nich jest
odwołanie się do nazwy pliku wykonywalnego, np. tak:

    /usr/bin/foo Pux,

Drugim zaś jest skorzystanie z `->` i odwołanie się do profilu po jego nazwie, przykładowo:

    /usr/bin/foo px -> profil1,

### Profile lokalne

AppArmor umożliwia tworzenie profili zagnieżdżonych (podprofile), które potocznie nazywane są
profilami lokalnymi lub też potomnymi. Mogą one posłużyć do zapewnienia bardziej ścisłego
ograniczenia wykonywanych zadań danej aplikacji. Taki profil potomny może zostać wykorzystany w
przypadku, gdy niekoniecznie chcemy aby dana aplikacja była ograniczona przy wywoływaniu jej w
systemie ale pożądane jest by restrykcje były narzucone, gdy ta sama aplikacja będzie wywoływana za
pośrednictwem wyprofilowanego już programu.

Lokalne profile określa się dokładnie w taki sam sposób co zwykle profile, z tą różnicą, że trzeba
je umieścić wewnątrz już istniejącego profilu. Poniżej przykład:

    /parent/profile {

          /path/to/child1 cx -> child1,

          /path/to/child2 cx,

          /path/to/* cx,    # jeśli * pasuje do child3 to nastąpi przejście do profilu /path/to/child3
                            # jeśli * pasuje do child4, child5, itd. to nastąpi przejście do profilu /path/to/child*
                            # jeśli profil potomny nie zostanie odnaleziony, proces nie zostanie uruchomiony

          /another/path/to/* cx -> child1,        # przesyła wszystkie dopasowania do profile child1

          profile child1 {

          }

          profile /path/to/child2 {

          }

          profile /path/to/child3 {

          }

          # ogólny profil rezerwowy (fallback)
          profile /path/to/child* {

          }
    }

Wszystkie profile lokalne są w logu AppArmor'a stosownie oznaczane, tj. przez odseparowanie ich od
głównego profilu przy pomocy `//` . W taki sposób, w polu `profile` otrzymujemy ciąg podobny do
`profile="/opt/google/chrome/chrome-sandbox//nacl"` , gdzie profil `nacl` jest profilem potomnym
profilu `opt.google.chrome.chrome-sandbox` .

### Modyfikatory reguł

W przypadku, gdy profil AppArmor'a nie ma zdefiniowanej jakiejś reguły dla określonego zasoby,
dostęp do niego zostanie zablokowany, a informacja o tej akcji zanotowana w logu systemowym. Z
kolei jeśli reguła istnieje, dostęp zostanie przyznany bez jakichkolwiek komunikatów w logu. Można
jednak użyć poniższych modyfikatorów, które wpłyną na to opisane powyżej zachowanie.

  - `audit` -- wymuszenie logowania
  - `deny` -- blokowanie dostępu bez logowania
  - `audit deny` -- blokowanie dostępu z logowaniem

Poniżej kilka przykładów dla lepszego zrozumienia zasady działania:

    /path/to/file*            r,  # zezwala na odczyt /path/to/file*
    /path/to/file1            w,  # zezwala na zapis do /path/to/file1
    deny /path/to/file2,      w,  # odmawia zapisu do /path/to/file2 bez logowania
    audit /path/to/file3      w,  # zezwala na zapis do /path/to/file3 z logowaniem
    audit deny /path/to/file4 r,  # odmawia odczytu /path/to/file4 z logowaniem

Reguły poprzedzone `deny` zawsze są brane pod uwagę jako pierwsze i nie mogą zostać nadpisane przez
reguły, które zezwalają na dostęp. W powyższym przykładzie, `audit deny /path/to/file4 r` nadpisuje
`/path/to/file* r` , zatem odczyt wszystkich plików pasujących do `file*` za wyjątkiem `file4`
będzie możliwy.

## Globbing

[Globbing](http://manpages.ubuntu.com/manpages/xenial/en/man7/glob.7.html) ma na celu zapisanie
wielu wzorów dopasowań w postaci jednej linijki. Globbing to nie wyrażenia regularne, a jedynie
kilka znaków znane jako wildcard'y. AppArmor korzysta z mechanizmu podobnego do tego jaki jest
wykorzystywany w shell'u bash. Nie jest on, co prawda, taki sam ale podstawy tego mechanizmu każdy z
nas powinien już jakieś posiadać. Poniżej znajduje się kilka przykładów, które powinny być
zrozumiane, gdy budujemy profil AppArmor'a:

  - `/dir/file ` -- pasuje do określonego pliku w katalogu `/dir/`
  - `/dir/* ` -- pasuje do każdego pliku w katalogu `dir` , wliczając w to pliki rozpoczynające się
    od `.`
  - `/dir/a*` -- pasuje do każdego pliku w katalogu `dir` rozpoczynającego się od `a`
  - `/dir/*.png` -- pasuje do każdego pliku w katalogu `dir` zakończonego `.png`
  - `/dir/[^.]*` -- pasuje do każdego pliku w katalogu `dir` za wyjątkiem plików rozpoczynających
    się od `.`
  - `/dir/` -- pasuje do katalogu `dir`
  - `/dir/*/` -- pasuje do każdego katalogu wewnątrz katalogu `dir`
  - `/dir/a*/` -- pasuje do każdego katalogu wewnątrz katalogu `dir` rozpoczynającego się od `a`
  - `/dir/*a/` -- pasuje do każdego katalogu wewnątrz katalogu `dir` , który kończy się na `a`
  - `/dir/**` -- pasuje do każdego pliku lub folderu wewnątrz i poniżej katalogu `/dir/`
  - `/dir/**/` -- pasuje do każdego folderu wewnątrz i poniżej katalogu `/dir/`
  - `/dir/**[^/]` -- pasuje do każdego pliku wewnątrz i poniżej katalogu `/dir/`
  - `/dir{,1,2}/**` -- pasuje do każdego pliku lub folderu wewnątrz lub poniżej katalogu `/dir/` ,
    `/dir1/` oraz `/dir2/`

Globbing może także zostać wykorzystany w nazwach profili, tak by reguły były aplikowane do szeregu
plików wykonywanych zamiast tylko do jednego z nich.

## Gotowe profile AppArmor'a

Co raz więcej aplikacji doczekuje się swoich indywidualnych profili i nie musimy ich pisać
samodzielnie. Te, które już posiadają swój profil, można [znaleźć
tutaj](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/AppArmorProfiles) i zwykle możemy je wgrać
do systemu instalując pakiety `apparmor-profiles` oraz `apparmor-profiles-extra` .
