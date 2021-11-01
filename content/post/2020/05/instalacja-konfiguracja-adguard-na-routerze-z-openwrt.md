---
author: Morfik
categories:
- OpenWRT
date: "2020-05-12T21:03:00Z"
lastmod: 2020-05-30 15:03:00 +0200
published: true
status: publish
tags:
- router
- tp-link
- adblock
- dns
- adguard
GHissueID: 16
title: Instalacja i konfiguracja AdGuard na routerze z OpenWRT
---

Jakiś już czas temu opisywałem w jaki sposób [skonfigurować AdBlock'a na routerze WiFi z wgranym
firmware OpenWRT][1] oraz jak wdrożyć [szyfrowanie zapytań DNS w oparciu o dnscrypt-proxy dla
wszystkich klientów naszej sieci domowej][2]. Zarówno AdBlock jak i dnscrypt-proxy można w dalszym
ciągu wykorzystywać, zwłaszcza na routerach wyposażonych w niewielkich rozmiarów flash i mało
pamięci RAM. Niemniej jednak, nie każdy lubi konfigurować swój bezprzewodowy router za
pośrednictwem terminala. Dla takich osób powstał właśnie [AdGuard Home][3], który ma na celu
możliwie uprościć konfigurację routera, przynajmniej jeśli chodzi o rzeczy związane z protokołem
DNS. W tym artykule przyjrzymy się nieco bliżej AdGuard'owi i zobaczymy czy można z niego zrobić
jakiś większy użytek.

<!--more-->
## Instalacja AdGuard na routerze

W OpenWRT jak na razie nie ma stosownego pakietu, który by zawierał AdGuard -- trzeba będzie
go zatem pozyskać i zainstalować ręcznie. [Stosowny pakiet można pobrać z github'a][4], trzeba
tylko wybrać odpowiedni plik. W przypadku routerów OpenWRT interesować nas będzie plik mający w
nazwie `AdGuardHome_linux_` , tylko który, bo jest ich tam kilka. Najprościej będzie zalogować się
na router przy pomocy SSH, podejrzeć plik `/proc/cpuinfo` i sprawdzić co tam widnieje przy pozycji
`cpu model` lub `model name` . Można także skorzystać z polecenia `uname -m` :

Poniżej przykład z routera Archer C7 v2:

    # cat /proc/cpuinfo
    ...
    cpu model               : MIPS 74Kc V5.0
    ...

    # uname -m
    mips

A niżej z routera Archer C2600:

    # cat /proc/cpuinfo
    ...
    model name      : ARMv7 Processor rev 0 (v7l)
    ...

    # uname -m
    armv7l

No jak widać, routery Archer C7 v2 i Archer C2600 mają inne architektury CPU i trzeba będzie dla
każdego z nich pobrać inny pakiet. W tym przypadku będzie wykorzystywany router Archer C2600 i dla
niego trzeba pobrać plik `AdGuardHome_linux_arm.tar.gz` . Pobraną paczkę `.tar.gz` przenosimy na
router przy pomocy `scp` do katalogu `/tmp/` i wypakowujemy do katalogu `/opt/` :

    $ scp AdGuardHome_linux_*.tar.gz 192.168.1.1:/tmp

    root@RedViper:~# mkdir /opt
    root@RedViper:~# tar zxf /tmp/AdGuardHome_linux_*.tar.gz  -C /opt

Binarka `AdGuardHome` swoje waży:

    # ls -alh /opt/AdGuardHome/AdGuardHome
    -rwxr-xr-x    1 root     root       12.4M Mar 13 10:32 /opt/AdGuardHome/AdGuardHome

Oczywiście, te widoczne wyżej 12,4M to rozmiar nieskompresowany i ta binarka będzie zajmować trochę
mniej miejsca na flash'u routera. Ze wstępnych ustaleń wynika, że potrzebnych jest około 6,3M na
flash, by tę binarkę `AdGuardHome` zmieścić na router, więc potrzebne będzie urządzenie dysponujące
flash'em o pojemności co najmniej 16M, ewentualnie można też posłużyć się [mechanizmem extroot][26].

## Wstępna konfiguracja AdGuard

Mając wgraną binarkę `AdGuardHome` na routerze, możemy przejść do uruchomienia tej aplikacji. W tym
celu wystarczy przejść do katalogu `/opt/AdGuardHome/` i uruchomić plik `AdGuardHome` :

    # cd /opt/AdGuardHome/
    # ./AdGuardHome
    2020/04/30 10:57:00 [error] Couldn't read config file /opt/AdGuardHome/AdGuardHome.yaml: open /opt/AdGuardHome/AdGuardHome.yaml: no such file or directory
    2020/04/30 10:57:00 [info] AdGuard Home, version v0.101.0, channel release
    , arch linux mips%!(EXTRA string=)
    2020/04/30 10:57:00 [info] This is the first launch of AdGuard Home, redirecting everything to /install.html
    2020/04/30 10:57:00 [info] AdGuard Home is available on the following addresses:
    2020/04/30 10:57:00 [info] Go to http://127.0.0.1:3000
    2020/04/30 10:57:00 [info] Go to http://10.217.228.189:3000
    2020/04/30 10:57:00 [info] Go to http://192.168.1.1:3000

Wyżej mamy informację, że jest to pierwsze uruchomienie AdGuard oraz, że musimy udać się pod jeden
z wypisanych adresów URL. Mamy co prawda trzy adresy ale nas interesuje ten, który ma adres IP
192.168.1.1, bo to z nim jesteśmy w stanie się połączyć będąc w naszej sieci domowej. Odpalamy
zatem przeglądarkę internetową i w pasku adresu wpisujemy adres `http://192.168.1.1:3000` . Naszym
oczom powinna pokazać się strona podobna do tej poniżej:

![adguard-home-openwrt-router-config](/img/2020/05/001-adguard-home-openwrt-router-config.png#huge)

Klikamy oczywiście w ten zielony przycisk i przechodzimy do wstępnej konfiguracji AdGuard. Na sam
początek musimy określić interfejs/adres IP oraz port, na którym będzie nasłuchiwał panel
administracyjny. Zwykle powinniśmy wybrać tutaj interfejs `br-lan` i określić port `80` ale w
przypadku mojego routera, na tym porcie nasłuchuje już panel LuCI i trzeba w takiej sytuacji wybrać
inny port, np. `8080` . Podobnie musimy określić adres i port dla serwera DNS. W tym przypadku
również wskazujemy interfejs `br-lan` ale z racji, że na domyślnym porcie `53` nasłuchuje już
`dnsmasq` , to musimy określić inny port, powiedzmy `5353` . Cała konfiguracja powinna zatem
wyglądać mniej więcej tak jak na poniższym obrazku:

![adguard-home-openwrt-router-config-ip-ports](/img/2020/05/002-adguard-home-openwrt-router-config-ip-ports.png#huge)

Następnie określamy login i hasło, którego będziemy używać przy logowaniu się do panela AdGuard:

![adguard-home-openwrt-router-config-user](/img/2020/05/003-adguard-home-openwrt-router-config-user.png#huge)

Dalej mamy szereg informacji dotyczących konfiguracji poszczególnych urządzeń, tak by
współpracowały one bez problemu z AdGuard:

![adguard-home-openwrt-router-config-device](/img/2020/05/004-adguard-home-openwrt-router-config-device.png#huge)

I to w zasadzie wszystko. Możemy teraz przejść do panelu konfiguracyjnego:

![adguard-home-openwrt-router-config-done](/img/2020/05/005-adguard-home-openwrt-router-config-done.png#huge)

Podajemy login i hasło, które ustawiliśmy wcześniej:

![adguard-home-openwrt-router-login](/img/2020/05/006-adguard-home-openwrt-router-login.png#medium)

Po podaniu poprawnych danych logowania powinniśmy zobaczyć miły dla oka panel administracyjny
AdGuard:

![adguard-home-openwrt-router-panel](/img/2020/05/007-adguard-home-openwrt-router-panel.png#huge)

## Konfiguracja routera na potrzeby AdGuard

Oczywiście samo wgranie AdGuard na router i jego uruchomienie nie sprawi, że będzie on realizował
swoje zadania, do których został zaprojektowany. Musimy jeszcze odpowiednio skonfigurować sam
router z OpenWRT. Standardowo w OpenWRT rozwiązywaniem nazw domen zajmuje się `dnsmasq` i to on
nasłuchuje na porcie 53. Klienty naszej sieci lokalnej w lease DHCP otrzymują namiar na serwer DNS
routera i to właśnie do `dnsmasq` lecą wszystkie zapytania o domeny, które później są przesyłane do
upstream'owego serwera DNS. W ten cały proces trzeba uwikłać teraz AdGuard, tj. tak skonfigurować
`dnsmasq` by przesłać odebrane z sieci zapytania DNS do AdGuard.

Pierwszy z plików, na który musimy rzucić okiem to `/etc/config/network` . W tym pliku przy
sekcjach z interfejsami WAN trzeba przy pomocy opcji `peerdns` wyłączyć branie pod uwagę serwerów
DNS jakie router uzyskuje od ISP w lease DHCP. Dodatkowo, trzeba za sprawą `list dns` określić
jakiś adres serwera DNS, bo w przeciwnym wypadku nie będzie działać nam rozwiązywanie domen. Musi
to być adres, na którym nasłuchuje `dnsmasq` , czyli 127.0.0.1 :

    config interface 'wan'
        ...
        option peerdns '0'
        list dns '127.0.0.1'
        ...

    config interface 'wan6'
        ...
        option peerdns '0'
        list dns '127.0.0.1'
        ...

Zatem każde zapytanie o domeny, które by wyszło przez jeden z tych powyższych interfejsów
sieciowych, zostanie skierowane do `dnsmasq` . Jako, że AdGuard ma pośredniczyć w rozwiązywaniu
nazw domen, to trzeba w pliku `/etc/config/dhcp` określić adres IP oraz port, na którym nasłuchuje
AdGuard, w tym przypadku adres to `192.168.1.1` , a port to `5353` :

    config dnsmasq
        ...
        option noresolv '1'
        list server '192.168.1.1#5353'
        ...

Opcja `noresolv` sprawia, że zapytania DNS będą przesyłane jedynie na adres określony w
`list server` tj. plik `/tmp/resolv.conf.auto` nie będzie brany w naszym przypadku pod uwagę.

Po zresetowaniu połączenia sieciowego, AdGuard powinien zacząć łapać zapytania DNS, co możemy
potwierdzić zaglądając do panelu admina:

![adguard-home-openwrt-router-panel-stats](/img/2020/05/008-adguard-home-openwrt-router-panel-stats.png#huge)

### Autostart

Domyślnie AdGuard nie instaluje żadnej usługi, która by umożliwiła mu uruchomienie się wraz ze
startem routera. Trzeba zatem ręcznie dodać stosowny wpis. Najlepiej jest zatem do pliku
`/etc/rc.local` dodać poniższą linijkę tuż przed `exit 0` :

    (/opt/AdGuardHome/AdGuardHome --work-dir /tmp/data --config /opt/AdGuardHome/AdGuardHome.yaml > /tmp/adguard.log 2>&1) &

Wszelkie komunikaty, które AdGuard będzie chciał zapisać, powędrują do pliku `/tmp/adguard.log` i
to tam powinniśmy szukać przyczyn ewentualnych problemów.

## Zaawansowana konfiguracja AdGuard

W zasadzie AdGuard powinien nam już działać, tj. filtrować zapytania DNS w oparciu o [filtr AdGuard
Simplified Domain Names filter][5], i większości użytkownikom tego typu rozwiązanie powinno w
zupełności wystarczyć. Niemniej jednak, przydałoby się nieco bliżej przyjrzeć konfiguracji AdGuard,
bo jest tam parę ciekawych rzeczy, które przydałoby się obadać.

### Dodatkowe filtry domen znane z AdBlock'a

Użytkownicy, którzy korzystali lub w dalszym ciągu korzystają z AdBlock'a dla OpenWRT, mieli
możliwość wyboru spośród kilku predefiniowanych list z domenami, w tym także mogli dodać własne
listy. Natomiast standardowo w AdGuard jest dostępna tylko jedna lista, tj. ten AdGuard Simplified
Domain Names filter. Nic jednak nie stoi na przeszkodzie by dodać sobie inne listy, w tym te, z
którymi mieliśmy do czynienia w przypadku AdBlock. By dodać listy domen, wystarczy przejść pod
Filters => DNS blocklists i tam dodać to co komu się podoba:

![adguard-home-openwrt-router-adblock-lists](/img/2020/05/009-adguard-home-openwrt-router-adblock-lists.png#huge)

Listy z AdBlock'a są w pełni kompatybilne z AdGuard i można ich bez problemu używać. Ja zwykle
korzystam z tych poniższych:

    https://adaway.org/hosts.txt
    https://www.malwaredomainlist.com/hostslist/hosts.txt
    https://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml&showintro=0&mimetype=plaintext'
    https://s3.amazonaws.com/lists.disconnect.me/simple_malvertising.txt
    http://adblocklist.org/adblock-pxf-polish.txt

#### Logowanie domen

Każda domena odwiedzana przez użytkowników naszej sieci domowej może być zalogowana. Taki log
naturalnie może posłużyć do inwigilacji członków naszej rodziny ale też w przypadku problemów z
internetem możemy zawsze zajrzeć w taki log domen i sprawdzić, które zapytania zostały zablokowane
lub przepuszczone. Można przy pomocy tego logu poprawić jakość samego AdBlock'a. Poniżej znajduje
się przykład logu:

![adguard-home-openwrt-router-domain-log-block](/img/2020/05/010-adguard-home-openwrt-router-domain-log-block.png#huge)

Jak widać wyżej, mamy również informację na temat listy, która zablokowała konkretne zapytanie o
domenę, co jest bardzo użyteczne w przypadku, gdy tych list mamy całkiem sporo.

Wszystkie zablokowane przez nas adresy są dodawane na listę, która znajduje się pod Filters =>
Custom filtering rules:

![adguard-home-openwrt-router-custom-log-rules](/img/2020/05/011-adguard-home-openwrt-router-custom-log-rules.png#huge)

Wpisy mające na początku `@@` oznaczają odblokowaną domenę.

Na powyższą listę możemy dodawać wpisy zarówno w formacie jaki występuje w regułach AdBlock'a, jak
i tym znanych z pliku `/etc/hosts` .

Zmiany w konfiguracji list są natychmiastowe i nie trzeba restartować AdGuard'a czy routera.

### Klienci AdGuard

AdGuard dla każdego klienta, który przesyła do niego zapytania DNS, jest w stanie utrzymywać osobną
konfigurację. Dla przykładu weźmy sobie upstream'owe serwery DNS, do których zapytania o domeny
będą forwardowane. Możliwe jest stworzenie konfiguracji takiej, że dla pewnych maszyn w sieci
będzie wykorzystywany jeden serwer DNS, a dla pozostałych komputerów inny serwer.

![adguard-home-openwrt-router-clients](/img/2020/05/012-adguard-home-openwrt-router-clients.png#huge)

W ten sposób możemy swoim pracownikom w firmie założyć blokadę na porno bez pozbawiania siebie
dostępu do tego typu serwisów. To oczywiście tylko jeden z przykładów, a tych naturalnie może być
więcej.

Niemniej jednak, w takiej konfiguracji jaką stworzyliśmy, jedynym klientem, który będzie przesyłał
zapytania DNS do AdGuard, jest nasz router, a konkretnie `dnsmasq` . Dlatego też w sekcji
`Top clients` będziemy zawsze widzieć jedno urządzenie, a nie wszystkie te maszyny, które aktywnie
korzystają z sieci.

Router w lease DHCP domyślnie rozsyła adres DNS jako swój, tj `192.168.1.1` . Do tego, protokół DNS
korzysta standardowo z portu 53. Dlatego też by mieć statystyki DNS per klient sieci LAN trzeba by
zamienić `dnsmasq` z AdGuard, tak by to ten drugi nasłuchiwał zapytań DNS na porcie 53 zamiast 5353
(tak jak to zostało w konfiguracji określone).

Niemniej jednak, jeśli chodzi o maszyny z linux'em na pokładzie, to w dalszym ciągu możemy bez
problemu te zapytania DNS przesłać bezpośrednio do AdGuard na adres 192.168.1.1 i port 5353. Nie
możemy przy tym korzystać z pliku `/etc/resolv.conf` , bo w tym pliku nie ma możliwości określenia
portu (przynajmniej [nie da się tego zrobić póki co na Debianie][6]). Natomiast zawsze można
postawić sobie [lokalny cache DNS w oparciu o dnsmasq][7] i w konfiguracji `dnsmasq` na linux
określić adres IP upstream'owego serwera DNS na 192.168.1.1 oraz port 5353. W takim przypadku bez
większego problemu nasz router domowy rozróżni ruch DNS od poszczególnych klientów:

![adguard-home-openwrt-router-clients](/img/2020/05/013-adguard-home-openwrt-router-clients.png#huge)

Oczywiście konfigurowanie wielu stacji w ten sposób mija się z celem i lepszym rozwiązaniem będzie
jednak zastąpienie `dnsmasq` na routerze przez AdGuard, przynajmniej jeśli chodzi o kwestię DNS, bo
`dnsmasq` robi także za serwer DHCP. Niby AdGuard posiada także serwer DHCP ale jest on póki co
oznaczony jako eksperymentalny i lepiej na nim aż tak nie polegać, przynajmniej jeszcze przez jakiś
czas.

### Szyfrowanie DNS (DoH, DoT, DNScrypt)

AdGuard zapewnia wsparcie dla szyfrowania ruchu DNS z wykorzystaniem [DoH][9], [DoT][10], a nawet
[DNScrypt][11]. To bardzo dobra wiadomość, bo w zasadzie użytkownik nie musi nic robić, by mieć
domyślnie szyfrowany DNS i to od razu dla całej swojej sieci domowej. [O różnicach między DoH i DoT
można poczytać tutaj][19].

Pytanie jakie wypadałoby tutaj zadać dotyczy serwera DNS, do którego te zapytania są przesyłane.
Domyślnie jest to [Quad9][16] ale nic nie stoi na przeszkodzie, by ten serwer zmienić i ustawić
sobie jakiś inny, który wspiera szyfrowane zapytania DNS, poniżej przykład:

![adguard-home-openwrt-router-upstream-dns-servers-doh-dot](/img/2020/05/014-adguard-home-openwrt-router-upstream-dns-servers-doh-dot.png#huge)

Powyżej został ustawiony serwer CloudFlare, zarówno dla protokołu DoH (ten z `https://` ), jak i
DoT (ten z `tls://`). By sprawdzić czy rozwiązywanie nazw DNS przez AdGuard faktycznie leci do
serwerów CloudFlare i odbywa się przy pomocy jednego z tych protokołów, wystarczy
[odwiedzić ten adres][17] i zrobić prosty test:

![adguard-home-openwrt-router-doh-dot-test](/img/2020/05/015-adguard-home-openwrt-router-doh-dot-test.png#big)

Wynik, jak widać, mówi raczej sam za siebie.

Możemy naturalnie określić tyle serwerów DNS ile tylko chcemy. Podobnie jest z protokołami. [Pełna
lista serwerów DNS, które możemy wykorzystać w AdGuard][18], jest dostępna tutaj.

W przypadku określenia więcej niż jeden serwer DNS, możemy rozłożyć zapytania tak, by cześć z nich
trafiła do jednego serwera, a część do drugiego i kolejnych. Takie podejście może przyśpieszyć cały
proces rozwiązywania nazw domen.

#### Czym są serwery Bootstrap DNS

Serwer Bootstrap DNS, to taki serwer, który wykorzystywany jest do rozwiązania domeny serwera DNS
określonego w konfiguracji AdGuard. Przykładowo, jeśli zdefiniowaliśmy upstream'owy serwer DNS jako
`https://cloudflare-dns.com/dns-query` , to mamy tutaj domenę. Ta domena nie jest znana routerowi,
np. po restarcie maszyny, i trzeba ją rozwiązać ale nie możemy tego zrobić przy pomocy
skonfigurowanego protokołu DoH, bo on zwyczajnie jeszcze nie działa. Dlatego właśnie na czas
rozwiązywania domeny ustawionego przez nas upstream'owego serwera DNS wykorzystywany jest jeden z
serwerów Bootstrap DNS, przez który to zapytanie o tę powyższą domenę (i tylko o nią) powędruje
przez sieć w formie niezaszyfrowanej. Jak tylko uda się rozwiązać domenę dla upstream'owego serwera
DNS, to każde zapytanie o zwykłe domeny stron WWW będzie już realizowane za sprawą protokołu DoH.

By skorzystać z CloudFlare i ustawić jego serwer DNS jako Bootstrap DNS, wystarczy wpisać `1.1.1.1`
w Bootstrap DNS servers (pod Settings =› DNS Settings):

![adguard-home-openwrt-router-bootsrap-dns](/img/2020/05/016-adguard-home-openwrt-router-bootsrap-dns.png#huge)

Gdy już skończymy się bawić ustawieniami upstream'owych serwerów DNS, warto jest przetestować ich
konfigurację klikając w przycisk `Test upstreams` widoczny powyżej. Jeśli konfiguracja jest
prawidłowa, to zostaniemy o tym fakcie powiadomieni.

Te serwery Bootstrap DNS są wykorzystywane jedynie w momencie określenia upstream'owych serwerów
DNS z wykorzystaniem domeny, tj. wspomniany wyżej `https://cloudflare-dns.com/dns-query` . Jeśli
zamiast domeny zostanie wykorzystany adres IP, przykładowo:

    https://1.1.1.1/dns-query
    https://1.0.0.1/dns-query

to w takim przypadku serwery Bootstrap DNS nie będą w ogóle wykorzystywane, co może pomóc w  walce
z ewentualną cenzurą przez zablokowanie ruchu do serwerów Bootstrap DNS.

Warto tutaj też zaznaczyć, że żadne inne zapytanie DNS nie zostanie przesłane za pomocą tych
serwerów Bootstrap DNS. W przypadku, gdyby jakieś zapytanie DNS się pojawiło przed rozwiązaniem
domeny upstream'owych serwerów DNS, to jego rozwiązanie zostanie opóźnione do momentu
skonfigurowania przez AdGuard tych upstream'owych serwerów DNS.

### Wymuszenie Safe Search w wyszukiwarkach

Wyszukiwarki takie jak Google, YouTube, Bing, DuckDuckGo, Yandex czy Pixabay mają do siebie to, że
potrafią zwracać czasami w wynikach treści rozumiane szeroko jako nieodpowiednie, np.
pornograficzne. Dlatego też każda z tych większych wyszukiwarek ma zaimplementowany [filtr Safe
Search][12], który ma za zadanie odfiltrować tego typu zwartość w wynikach wyszukiwania. Niemniej
jednak, użytkownik ten filtr Safe Search może sobie wyłączyć i dalej być w stanie uzyskać dostęp to
tego typu materiałów dla dorosłych.

AdGuard posiada opcję wymuszenia Safe Search w wyszukiwarkach. W przypadku, gdy ta opcja zostanie
aktywowana, to użytkownik nie będzie w stanie w ustawieniach któregoś z tych ww. serwisów zmienić
opcji odpowiedzialnych za filtr Safe Search. No może i będzie w stanie zmienić ustawienia ale
zostaną one przepisane przez AdGuard.

### Usługa AdGuard Browsing Security

[Usługa AdGuard Browsing Security][13] jest to mechanizm, który ma na celu odfiltrowanie z sieci
stron phishing'owych oraz tych, których odwiedzenie może skończyć się zawirusowaniem naszego
komputera. Dokładny [opis jak działa AdGuard Browsing Security][14] można znaleźć tutaj. Jeśli
chodzi zaś o kwestię prywatności tego rozwiązania, to operuje ono na prefiksach hash'ów, przez co
serwer AdGuard nie loguje i nie przechowuje żadnych informacji, które mogłyby pozwolić
zidentyfikować odwiedzane przez nas adresy WWW. Poza tym, nawet jeśli AdGuard by umożliwiał
zidentyfikowanie domen, które odwiedzamy, to zawsze trzeba pamiętać o [Server Name Indication][15]
(SNI). Za sprawą tego rozszerzenia w pierwszych pakietach przed zestawieniem tunelu SSL/TLS leci
certyfikat zwierający informację domenie, na którą został wystawiony, tj. tę samą, z którą
próbujemy uzyskać połączenie. Ta informacja nie jest w żaden sposób zabezpieczona, nawet w TLS 1.3.
Zatem jeśli ISP loguje nasze połączenia, to i tak informacje o domenie jest w stanie bez problemu
pozyskać i, co gorsze, taką domenę może bez większego wysiłku zablokować.

### Blokada serwisów społecznościowych

Jeżeli jesteśmy zainteresowani wycięciem na naszym domowym routerze wszystkich lub też tylko części
serwisów społecznościowych, to AdGuard ma również i taką funkcję. Poniżej znajduje się obrazek, na
którym są przedstawione wszystkie serwisy, które za pomocą stosownego przełącznika możemy
zablokować:

![adguard-home-openwrt-router-social-filter](/img/2020/05/017-adguard-home-openwrt-router-social-filter.png#huge)

Jak widać, tych serwisów jest dość sporo, a by je zablokować wystarczy kliknąć przy stosownej
pozycji i zapisać zmiany. Od tego momentu, żaden użytkownik naszej sieci nie będzie w stanie się
już połączyć z tymi zablokowanymi stronami.

### Blokada serwisów pornograficznych

Podobnie sprawa ma się w kwestii blokowania stron pornograficznych. Nie mamy tutaj co prawda
swobody w określeniu jaki serwis AdGuard może zablokować. Zatem możemy jedynie zablokować wszystkie
serwisy porno lub żadne. Mechanizm blokowania stron pornograficznych działa na podobnej zasadzie do
AdGuard Browsing Security.

### Wymuszenie korzystania z AdGuard

Jak widać z powyższego opisu, AdGuard jest całkiem ciekawym narzędziem, które jest w stanie
zabezpieczyć i jednocześnie odfiltrować niepożądany ruch DNS dla całej naszej sieci domowej.
Niemniej jednak, w dalszym ciągu istnieje możliwość obejścia tego mechanizmu i uzyskania dostępu
przykładowo do stron porno. Jedyne co użytkownicy w sieci LAN muszą zrobić, to zignorować serwery
DNS, które otrzymują wraz z lease DHCP, i ustawić w ich miejsce swoje własne serwery DNS, np.
góglowski 8.8.8.8. Każdy system operacyjny umożliwia tego typu zabieg w ustawieniach sieciowych.
Czy jest zatem jakiś sposób, by takiemu zachowaniu przeciwdziałać, tj. by restrykcje, które AdGuard
wprowadzi w naszej sieci, były wymuszone dla każdego określonego przez nas klienta?

W zasadzie ruch DNS odbywa się na porcie UDP/53 (lub/i też TCP/53) i ten port można by na routerze
przekierować na adres i port, na którym nasłuchuje AdGuard, czyli w tym przypadku 192.168.1.1:5353
(lub też przekierować do `dnsmasq` na 192.168.1.1:53). Takie przekierowanie nie jest niczym trudnym
i sprowadza się do dodania poniższych wpisów w pliku `/etc/firewall.user` :

    iptables -t nat -A PREROUTING -i br-lan ! -s 192.168.1.1 -p tcp --dport 53 -j DNAT --to 192.168.1.1:53
    iptables -t nat -A PREROUTING -i br-lan ! -s 192.168.1.1 -p udp --dport 53 -j DNAT --to 192.168.1.1:53

    ip6tables -t nat -A PREROUTING -i br-lan ! -s fde8:fd0c:849b::1 -p tcp --dport 53 -j DNAT --to [fde8:fd0c:849b::1]:53
    ip6tables -t nat -A PREROUTING -i br-lan ! -s fde8:fd0c:849b::1 -p udp --dport 53 -j DNAT --to [fde8:fd0c:849b::1]:53

Teraz trzeba jeszcze zrestartować router lub usługę firewall'a:

    # /etc/init.d/firewall restart

Reguły powinny zostać dodane, a przekierowanie powinno realizować swoje zadanie, co możemy
sprawdzić podglądając wyjście `iptables` po wydaniu paru poleceń `ping` do określonych domen:

    # iptables -nvL -t nat
    Chain PREROUTING (policy ACCEPT 36 packets, 2208 bytes)
     pkts bytes target     prot opt in     out     source               destination
     ...
        6   375 DNAT       udp  --  br-lan *      !192.168.1.1          0.0.0.0/0            udp dpt:53 to:192.168.1.1:53
        0     0 DNAT       tcp  --  br-lan *      !192.168.1.1          0.0.0.0/0            tcp dpt:53 to:192.168.1.1:53
     ...

lub też w panelu AdGuard:

![adguard-home-openwrt-router-dns-leak-test](/img/2020/05/018-adguard-home-openwrt-router-dns-leak-test.png#huge)

Zatem, nawet jeśli klient sobie ustawi w swoim systemie serwer DNS od Google, to i tak mu to nic
nie da, bo zapytania zostaną przekierowane na router i przejdą przez AdGuard.

#### A co jeśli klient korzysta z DoH, DoT lub DNScrypt bezpośrednio

Oczywiście, ten powyższy sposób sprawdzi się jedynie w przypadku tradycyjnych serwerów DNS, gdzie
znany jest port (53). Niemniej jednak, gdy w grę wchodzi szyfrowanie zapytań DNS (DoH, DoT czy
DNScrypt), to już tak łato nie będzie.

O ile DoT wykorzystuje również znany port 853, który można przekierować/zablokować, to DoH i
DNScrypt wykorzystują port 443, czyli ten sam, na którym działają strony po HTTPS. I
zablokowanie/przekierowanie tego portu będzie równoznaczne z brakiem możliwości odwiedzania
jakichkolwiek stron WWW. Krótko pisząc, jeśli użytkownik skonfiguruje DoH lub DNScrypt na swoim
komputerze, to przekierowanie portów nic nam nie da. W takiej sytuacji trzeba by stworzyć listę
serwerów DNS, które wspierają DoH i DNScrypt, i dodać do listy ipset ich adresy IP, po czym przy
pomocy jednej reguły na zaporze zablokować komunikację pochodzącą z LAN kierowaną na adresy DNS
obecne na liście ipset. W takim przypadku użytkownik naszej sieci będzie miał do wyboru albo
korzystać z adresów DNS, które mu poda router, albo korzystać z internetu bez DNS.

Trzeba jednak zdawać sobie sprawę, że obecnie protokoły DoH i DoT coraz powszechniej implementowane
są bezpośrednio w aplikacjach. Przykładem może być przeglądarka Firefox, która udostępnia opcję
[szyfrowania zapytań z wykorzystaniem DoH][20] w swoich opcjach:

![adguard-home-openwrt-router-firefox-doh](/img/2020/05/019-adguard-home-openwrt-router-firefox-doh.png#huge)

Po aktywowaniu tej opcji (jeśli nie jest jeszcze ona domyślnie włączona), Firefox będzie szyfrował
i przesyłał wszystkie swoje zapytania DNS do CloudFlare niezależnie od tego co zostało
skonfigurowane w systemie komputera czy też na routerze WiFi.

Podobnie sprawa wygląda w przypadku smartfonów z Androidem 9+, gdzie [pojawiło się wsparcie dla
szyfrowania DNS via DoT][21]. Wymuszanie korzystania z DNS routera w telefonach sprawi, że
prawdopodobnie nie będą one szyfrować ruchu DNS poza siecią domową, bo ciągle trzeba by zmieniać
ustawienia telefonu w przypadku, gdy ISP nie oferuje szyfrowania DNS, a tego nikt nie chce.

Wymuszając na użytkownikach korzystanie z określonego serwera DNS prędzej czy później sprawi, że
nie będą oni w ogóle chronieni, a chyba nie o to nam chodziło.

### Panel AdGuard po HTTPS i serwer DoT dla sieci lokalnej

Domyślnie po skonfigurowaniu, AdGuard udostępnia panel admina po HTTP. Istnieje możliwość
zaimplementowania szyfrowanego połączenia z tym panelem ale trzeba pierw sobie wyrobić klucz i
certyfikat. Te dwie rzeczy możemy wygenerować na dowolnej dystrybucji linux'a przy [pomocy
poniższego polecenia][25]:

    $ openssl req -x509 -days 3650 -out redviper.crt -keyout redviper.key \
      -newkey rsa:2048 -nodes -sha256 \
      -subj '/CN=redviper.mhouse' -extensions EXT -config <( \
       printf "[dn]\nCN=redviper.mhouse\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:redviper.mhouse\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")

    Generating a RSA private key
    .+++++
    ...+++++
    writing new private key to 'redviper.key'
    -----

Powinny zostać nam wygenerowane dwa pliki. Jeden z nich zawiera klucz prywatny, drugi zaś
certyfikat. Zawartość tych plików trzeba wpisać w panelu AdGuard w Settings => Encryption Settings.
Musimy także określić `Server Name` na `redviper.mhouse` (czy co tam podaliśmy przy generowaniu
certyfikatu) oraz dobrać porty pod panel WWW i serwer DoT:

![adguard-home-openwrt-router-panel-https](/img/2020/05/020-adguard-home-openwrt-router-panel-https.png#huge)

Mamy tam co prawda błąd `Certificate chain is invalid` ale on w niczym nie przeszkadza w tego typu
instalacji.

Po pomyślnym skonfigurowaniu AdGuard, powinniśmy mieć mniej więcej takie wpisy w `netstat` :

    # netstat -napletu | grep Ad
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 192.168.1.1:8080        0.0.0.0:*               LISTEN      18646/AdGuardHome
    tcp        0      0 192.168.1.1:4433        0.0.0.0:*               LISTEN      18646/AdGuardHome
    tcp        0      0 192.168.1.1:853         0.0.0.0:*               LISTEN      18646/AdGuardHome
    tcp        0      0 192.168.1.1:5353        0.0.0.0:*               LISTEN      18646/AdGuardHome
    tcp        0      0 10.221.41.129:37450     1.1.1.1:443             ESTABLISHED 18646/AdGuardHome
    tcp        0      0 10.221.41.129:58768     176.103.130.132:443     ESTABLISHED 18646/AdGuardHome
    tcp        0      0 10.221.41.129:58592     1.0.0.1:443             ESTABLISHED 18646/AdGuardHome
    udp        0      0 192.168.1.1:5353        0.0.0.0:*                           18646/AdGuardHome

Wpisy protokołu `TCP` z `LISTEN` odpowiadają kolejno za serwer WWW na porcie 8080 oraz 4433, dalej
mamy serwer DoT na porcie 853. Port 5353 zarówno dla TCP jak i UDP odpowiada za serwer DNS dla
sieci lokalnej. Zaś wpisy mające 1.1.1.1 oraz 1.0.0.1 w `Foreign Address` odpowiadają za
upstream'owe serwery DNS, do których będą trafiać zapytania. Ten ostatni adres, tj.
`176.103.130.132:443` dotyczy usługi bezpiecznego przeglądania stron WWW (AdGuard browsing security
web service).

## Integracja AdGuard Home z OpenWRT

Póki co AdGuard nie jest dostępny w repozytoriach OpenWRT i trzeba niestety ręcznie pobierać i
instalować stosowny pakiet. Niemniej jednak, osoby, które [budują własne obrazy z firmware
OpenWRT][8], mogą zawrzeć w nich binarkę i konfigurację dla AdGuard. W tym celu w zasadzie
wystarczy wypakować pobraną z github'a paczkę `.tar.gz` , a jej zawartość wrzucić do katalogu
`files/opt/` . By uniknąć ponownej konfiguracji Adguard, potrzebny nam także będzie plik
`AdGuardHome.yaml` , którego zawartość jest poniżej:

    bind_host: 192.168.1.1
    bind_port: 8080
    users:
    - name: root
      password: hash_hasla
    language: ""
    rlimit_nofile: 0
    web_session_ttl: 720
    dns:
      bind_host: 192.168.1.1
      port: 5353
      statistics_interval: 90
      querylog_enabled: true
      querylog_interval: 90
      querylog_memsize: 0
      protection_enabled: true
      blocking_mode: default
      blocking_ipv4: ""
      blocking_ipv6: ""
      blocked_response_ttl: 10
      ratelimit: 512
      ratelimit_whitelist: []
      refuse_any: true
      bootstrap_dns:
      - 1.1.1.1
      all_servers: true
      edns_client_subnet: false
      aaaa_disabled: true
      allowed_clients: []
      disallowed_clients: []
      blocked_hosts: []
      parental_block_host: family-block.dns.adguard.com
      safebrowsing_block_host: standard-block.dns.adguard.com
      cache_size: 4194304
      upstream_dns:
      - https://1.1.1.1/dns-query
      - https://1.0.0.1/dns-query
      filtering_enabled: true
      filters_update_interval: 24
      parental_enabled: false
      safesearch_enabled: false
      safebrowsing_enabled: true
      safebrowsing_cache_size: 1048576
      safesearch_cache_size: 1048576
      parental_cache_size: 1048576
      cache_time: 30
      rewrites: []
      blocked_services: []
    tls:
      enabled: true
      server_name: redviper.mhouse
      force_https: true
      port_https: 4433
      port_dns_over_tls: 853
      allow_unencrypted_doh: false
      strict_sni_check: false
      certificate_chain: |
        -----BEGIN CERTIFICATE-----
        ...
        -----END CERTIFICATE-----
      private_key: |
        -----BEGIN PRIVATE KEY-----
        ...
        -----END PRIVATE KEY-----
      certificate_path: ""
      private_key_path: ""
    filters:
    - enabled: true
      url: https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
      name: AdGuard Simplified Domain Names filter
      id: 1
    - enabled: true
      url: https://adaway.org/hosts.txt
      name: AdAway
      id: 2
    - enabled: false
      url: https://hosts-file.net/ad_servers.txt
      name: hpHosts - Ad and Tracking servers only
      id: 3
    - enabled: true
      url: https://www.malwaredomainlist.com/hostslist/hosts.txt
      name: MalwareDomainList.com Hosts List
      id: 4
    - enabled: true
      url: http://adblocklist.org/adblock-pxf-polish.txt
      name: regpl
      id: 1587891685
    - enabled: true
      url: https://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml&showintro=0&mimetype=plaintext'
      name: yoyo
      id: 1587891686
    - enabled: true
      url: https://s3.amazonaws.com/lists.disconnect.me/simple_malvertising.txt
      name: disconnect
      id: 1587891687
    whitelist_filters: []
    user_rules:
    - ""
    dhcp:
      enabled: false
      interface_name: ""
      gateway_ip: ""
      subnet_mask: ""
      range_start: ""
      range_end: ""
      lease_duration: 86400
      icmp_timeout_msec: 1000
    clients:
    - name: morfikownia
      tags:
      - device_laptop
      - os_linux
      - user_admin
      ids:
      - 192.168.1.150
      use_global_settings: true
      filtering_enabled: false
      parental_enabled: false
      safesearch_enabled: false
      safebrowsing_enabled: false
      use_global_blocked_services: true
      blocked_services: []
      upstreams: []
    - name: router
      tags:
      - device_other
      - os_linux
      - user_regular
      ids:
      - 192.168.1.1
      use_global_settings: true
      filtering_enabled: false
      parental_enabled: false
      safesearch_enabled: false
      safebrowsing_enabled: false
      use_global_blocked_services: true
      blocked_services: []
      upstreams: []
    log_file: ""
    verbose: false
    schema_version: 6

W `name:` i `password:` trzeba określić nazwę użytkownika i hash hasła. Podobnie trzeba postąpić z
kluczem oraz certyfikatem w `private_key` oraz `certificate_chain` . Te dane będą podane po
skonfigurowaniu AdGuard'a, zatem można sobie zrobić backup tego pliku.

Dodatkowo, musimy jeszcze zmienić konfigurację dnsmasq w `files/etc/uci-defaults/01-config.sh` :

    uci add_list dhcp.@dnsmasq[0].server='192.168.1.1#5353'
    uci set dhcp.@dnsmasq[0].noresolv='1'

    uci set network.wan.peerdns='0'
    uci add_list network.wan.dns='127.0.0.1'
    uci set network.wan6.peerdns='0'
    uci add_list network.wan6.dns='127.0.0.1'

Musimy też dodać sobie wpis do pliku `/etc/rc.local` , tak by nam ten AdGuard startował
automatycznie. Tworzymy zatem plik `files/etc/rc.local` i dodajemy w nim poniższą linijkę:

    (/opt/AdGuardHome/AdGuardHome --work-dir /tmp/data --config /opt/AdGuardHome/AdGuardHome.yaml > /tmp/adguard.log 2>&1) &

    exit 0

No i na koniec jeszcze tworzymy plik `files/etc/firewall.user` i dodajemy te poniższe wpisy:

    iptables -t nat -A PREROUTING -i br-lan ! -s 192.168.1.1 -p tcp --dport 53 -j DNAT --to 192.168.1.1:53
    iptables -t nat -A PREROUTING -i br-lan ! -s 192.168.1.1 -p udp --dport 53 -j DNAT --to 192.168.1.1:53

    ip6tables -t nat -A PREROUTING -i br-lan ! -s fde8:fd0c:849b::1 -p tcp --dport 53 -j DNAT --to [fde8:fd0c:849b::1]:53
    ip6tables -t nat -A PREROUTING -i br-lan ! -s fde8:fd0c:849b::1 -p udp --dport 53 -j DNAT --to [fde8:fd0c:849b::1]:53

## Problemy z AdGuard

Z reguły instalacja AdGuard na OpenWRT powinna przebiegać w miarę w bezproblemowy sposób. Niemniej
jednak, w moim przypadku pojawiło się trochę problemów, które postanowiłem opisać poniżej.

### Nie wszystkie reklamy są blokowane

Blokowanie reklam w AdGuard działa w oparciu o nazwy domen. Dlatego też ten filtr nie wytnie nam
reklam w 100%. Przykładem mogą być telefony z Androidem, a konkretnie aplikacja YouTube, gdzie
adresy adserwerów są zaszyte bezpośrednio w appce. W takim przypadku potrzebny jest inny klient
YouTube i AdGuard nic tutaj nie poradzi.

Podobnie sprawa wygląda w przypadku różnych serwisów www, gdzie by obejrzeć jakiś materiał video
trzeba pierw wyświetlić reklamę, i jeśli nie zostanie ona wyświetlona (adserwer zostanie
zablokowany przez AdGuard), to nie będziemy mieli możliwości w ogóle wyświetlić takiego filmu.

Większość reklam powinna jednak zniknąć, czy to w telefonach czy to w komputerach
stacjonarnych/laptopach, a jeśli już jakieś zostają, to trzeba posiłkować się różnymi
rozszerzeniami do przeglądarek pokroju [uBlock][22]/[uMartix][23]/[anti-adblock-killer][24], które
są w stanie wyciąć określone elementy stron WWW niezależnie od domen adserwerów.

### Protokół IPv6

Mój provider internetowy nie zapewnia wsparcia dla protokołu IPv6 i mogę korzystać jedynie ze
starszej wersji protokołu IPv4. W takim przypadku mogą pojawić się opóźnienia przy rozwiązywaniu
nazw i dobrze jest w takiej sytuacji wyłączyć rozwiązywanie zapytań DNS dla IPv6 (w Settings =>
DNS Settings).

### Couldn't initialize DNS server: Couldn't initialize statistics module

Czasami jednak możemy napotkać błąd podczas konfiguracji, który objawia się poniższym komunikatem

    Error: control/install/configure | Couldn't initialize DNS server: Couldn't initialize statistics module | 500` .

W logu systemowym możemy zaś zaobserwować poniższe wiadomości:

    [error] Stats: open DB: /opt/AdGuardHome/data/stats.db: invalid argument
    [error] AdGuard Home cannot be initialized due to an incompatible file system.
    Please read the explanation here: https://github.com/AdguardTeam/AdGuardHome/wiki/Getting-Started#limitations
    [info] Couldn't initialize DNS server: Couldn't initialize statistics module

Jedynym wyjściem w tym przypadku jest uruchomienie AdGuard'a przy pomocy poniższego polecenia:

    /opt/AdGuardHome/AdGuardHome --work-dir /tmp/data --config /opt/AdGuardHome/AdGuardHome.yaml

Opcja `--work-dir` określa katalog roboczy, który będzie umieszczony w RAM (cały `/tmp/` w OpenWRT
siedzi w RAM) i zrestartowanie routera będzie powodować utratę bazy danych (statystyk, logów, itp.).
Domyślnie także plik konfiguracyjny `AdGuardHome.yaml` zostałby wrzucony do katalogu `/tmp/data/` ,
co skutkowałoby ponowną konfiguracją AdGuard'a po restarcie routera. Dlatego też przy pomocy
`--config` określamy, w którym miejscu AdGuard ma tego pliku poszukiwać.

### libustream-mbedtls vs. libustream-openssl

Adguard wymaga do poprawnego działania biblioteki SSL. W OpenWRT mamy dostępne między innymi
pakiety: `libustream-mbedtls` (MbedTLS) i `libustream-openssl` (OpenSSL). W zależności od tego jak
budowany był obraz firmware, jeden lub drugi pakiet może być obecny w systemie routera. Jeśli nie
mamy żadnej biblioteki SSL wgranej na routerze, to trzeba jeden z tych dwóch pakietów doinstalować.
Czasami użytkownicy próbują doinstalować `libustream-mbedtls` mając `libustream-openssl` i w takim
przypadku instalacja tego dodatkowego pakietu skoczy się poniższym błędem:

    # opkg install libustream-mbedtls
    Installing libustream-mbedtls20150806 (2019-11-05-c9b66682-2) to root...
    Downloading http://downloads.openwrt.org/releases/19.07-SNAPSHOT/packages/mips_24kc/base/libustream-mbedtls20150806_2019-11-05-c9b66682-2_mips_24kc.ipk
    Installing libmbedtls12 (2.16.6-1) to root...
    Downloading http://downloads.openwrt.org/releases/19.07-SNAPSHOT/packages/mips_24kc/base/libmbedtls12_2.16.6-1_mips_24kc.ipk
    Configuring libmbedtls12.
    Collected errors:
     * check_data_file_clashes: Package libustream-mbedtls20150806 wants to install file /lib/libustream-ssl.so
            But that file is already provided by package  * libustream-openssl20150806
     * opkg_install_cmd: Cannot install package libustream-mbedtls.

Jak widać powstał konflikt o plik `/lib/libustream-ssl.so` , który dostarczany jest przez każdy z
tych dwóch pakietów. W OpenWRT nie ma możliwości posiadać w systemie równocześnie kilku pakietów
dostarczających ten sam plik. Dlatego trzeba się zdecydować na jedną z tych bibliotek SSL. AdGuard
bez problemu będzie działał zarówno z OpenSSL jak i MbedTLS. Zatem jeśli nasz router jest
skonfigurowany do pracy z OpenSSL, to nie musimy sobie zawracać głowy pakietem `libustream-mbedtls`
i podobnie w drugą stronę.

### x509: certificate signed by unknown authority

Po zalogowaniu się do panelu AdGuard, w logu na routerze można było zauważyć te poniższe komunikaty:

    2020/04/30 11:26:30 [info] Couldn't request filter from URL https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt, skipping: Get https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt: x509: certificate signed by unknown authority
    2020/04/30 11:26:30 [info] Failed to update filter https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt: Get https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt: x509: certificate signed by unknown authority

    2020/04/30 11:26:30 [info] Go to http://192.168.1.1:8080
    2020/04/30 11:29:45 [info] Auth: invalid cookie value: agh_session=b94b76b67bdd4af4fd20b7b6537f76fb86fbf97562e1db37b6d3a19ac4c18223
    2020/04/30 11:29:45 [info] Auth: invalid cookie value: agh_session=b94b76b67bdd4af4fd20b7b6537f76fb86fbf97562e1db37b6d3a19ac4c18223
    2020/04/30 11:30:51 [info] Couldn't get version check json from https://static.adguard.com/adguardhome/release/version.json: *url.Error Get https://static.adguard.com/adguardhome/release/version.json: x509: certificate signed by unknown authority

By poprawić te błędy, trzeba doinstalować na routerze pakiet `ca-certificates` oraz `ca-bundle` .

## Podsumowanie

AdGuard to dość przyzwoite narzędzie, którym możemy praktycznie bez większego wysiłku zabezpieczyć
ruch DNS naszej sieci domowej przez wdrożenie protokołu DoH, DoT lub DNScrypt. AdGuard posiada
także dość konfigurowalny filtr domen internetowych, przy którego to pomocy jesteśmy w stanie
zablokować niepożądany kontent, w tym też część reklam. Niemniej jednak, każdy filtr, w tym też ten
oferowany przez AdGuard, ma swoje słabości i nie jest w stanie wyciąć wszystkich reklam, na które
natkniemy się w sieci. Jeśli zaś chodzi o aspekt cenzorski, to każdą blokadę da radę obejść i
AdGuard nie jest tutaj żadnym wyjątkiem. Osoby, które będą chciały mieć dostęp do treści naszym
zdaniem nieodpowiednich, będą mogły bez problemu go uzyskać i AdGuard ich przed tym nie powstrzyma.

Podczas testów nie napotkałem większych problemów w działaniu AdGuard na swoim routerze Archer
C2600. Niemniej jednak, ten router dysponuje 512M RAM, 32M flash oraz posiada dwurdzeniowy procesor
1,4GHz, co nieco ułatwia wdrożenie mechanizmu AdGuard. Na słabszych routerach pewne problemy mogą
się pojawić, bo jakby nie patrzeć trochę pamięci ten panel GUI zjada (>32M). Podobnie sprawa ma się
w kwestii flash, gdzie potrzebne będzie urządzenie dysponujące flash'em minimum 16M. W przypadku
routerów z mniejszym flash trzeba będzie korzystać z extroot i przeznaczyć jeden port USB na
podłączenie pendrive. Dlatego też zawsze alternatywą dla AdGuard w przypadku tych słabszych
routerów będzie AdBlock w połączeniu z dnscrypt-proxy, który w zasadzie jest w stanie zapewnić tę
samą funkcjonalność ale niższym kosztem.


[1]: /post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/
[2]: /post/konfiguracja-dnscrypt-proxy-w-openwrt/
[3]: https://github.com/AdguardTeam/AdGuardHome#comparison
[4]: https://github.com/AdguardTeam/AdGuardHome/releases
[5]: https://github.com/AdguardTeam/AdguardSDNSFilter
[6]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=549190
[7]: /post/cache-dns-buforowania-zapytan/
[8]: /post/jak-zbudowac-uaktualnic-firmware-openwrt-dla-routera-wifi/
[9]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[10]: https://en.wikipedia.org/wiki/DNS_over_TLS
[11]: https://dnscrypt.info/
[12]: https://support.google.com/websearch/answer/510
[13]: https://kb.adguard.com/en/general/how-malware-protection-works
[14]: https://github.com/AdguardTeam/AdguardForAndroid/issues/162
[15]: https://en.wikipedia.org/wiki/Server_Name_Indication
[16]: https://www.quad9.net/
[17]: https://1.1.1.1/help
[18]: https://kb.adguard.com/en/general/dns-providers
[19]: https://www.cloudflare.com/learning/dns/dns-over-tls/
[20]: https://support.mozilla.org/en-US/kb/firefox-dns-over-https
[21]: https://android-developers.googleblog.com/2018/04/dns-over-tls-support-in-android-p.html
[22]: https://github.com/gorhill/uBlock
[23]: https://github.com/gorhill/uMatrix
[24]: https://github.com/reek/anti-adblock-killer
[25]: https://letsencrypt.org/docs/certificates-for-localhost/
[26]: /post/extroot-whole_root-fullroot-pod-openwrt/
