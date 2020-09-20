---
author: Morfik
categories:
- OpenWRT
date:    2016-05-02 16:21:20 +0200
lastmod: 2016-05-02 16:21:20 +0200
published: true
status: publish
tags:
- dns
- chaos-calmer
- router
- dhcp
- dnsmasq
- cache
- resolver
title: DHCP i DNS, czyli konfiguracja sieci w OpenWRT
---

Rutery WiFi są w stanie zorganizować przewodową i/lub bezprzewodową sieć w naszych domach. By taka
sieć działała bez zarzutu, potrzebna jest odpowiednia adresacja wszystkich komputerów wewnątrz niej.
W obecnych czasach już praktycznie nie stosuje się statycznej konfiguracji, bo to zadanie zostało
zrzucone na barki serwera DHCP. W OpenWRT do tego celu oddelegowane jest [oprogramowanie
dnsmasq][1]. Zapewnia ono nie tylko wspomniany wyżej serwer DHCP ale także serwer cache'ujący
zapytania DNS. Ten drugi z kolei jest niezastąpiony w przypadku przekazywania zapytań o nazwy domen
do upstream'owego serwera DNS, który zajmuje się rozwiązywaniem tych nazw na odpowiadające im adresy
IP. Bez `dnsmasq` ogarnięcie naszej sieci przerodziłoby się w istne piekło. Dlatego też w tym
artykule przybliżymy sobie nieco konfigurację tego narzędzia.

<!--more-->
## Jak konfigurować dnsmasq w OpenWRT

W przypadku OpenWRT, `dnsmasq` można konfigurować na dwa sposoby. Pierwszym z nich jest konfiguracja
przez plik `/etc/config/dhcp` . I jest to standardowe podejście. Przy starcie routera, po
przetworzeniu skryptu `/etc/init.d/dnsmasq` jest generowany plik `/var/etc/dnsmasq.conf` . Niemniej
jednak, tak utworzona konfiguracja nie zawiera wszystkich opcji możliwych do określenia w
`dnsmasq` . Z tego właśnie powodu developerzy OpenWRT zostawili furtkę bezpieczeństwa, dzięki której
możemy korzystać z dodatkowego pliku konfiguracyjnego `/etc/dnsmasq.conf` . I to jest właśnie drugi
ze sposobów konfiguracji. Wszystkie niestandardowe opcje są umieszczane właśnie w tym pliku. Zatem
jeśli jakieś opcje nie są uwzględniane przez natywny mechanizm OpenWRT, to zawsze możemy go obejść
przez plik `/etc/dnsmasq.conf` i to tam dodać stosowne wpisy.

O samej konfiguracji narzędzia `dnsmasq` wspominałem już wielokrotnie na tym blogu w kontekście
różnych aspektów pracy zarówno systemów z rodziny linux jak i OpenWRT. Niemniej jednak, tutaj
postaram się zebrać wszystkie te bardziej użyteczne opcje, tak by były w jednym miejscu. Warto też
wspomnieć, że cała masa parametrów jest opisana na [wiki OpenWRT][2]. Przejdźmy zatem do edycji
pliku `/etc/config/dhcp` .

## Serwer DNS

Poniższe dwie opcje są w stanie odfiltrować zapytania, na które nie potrafią odpowiedzieć publiczne
serwery DNS. Pierwsza z nich, tj. `domainneeded` , nie forwarduje nazw bez kropek (bez części
domeny). Zaś `boguspriv` nie zezwala na przekazywanie adresów z prywatnej przestrzeni adresowej.

    option domainneeded '1'
    option boguspriv '1'

Opcja `local` sprawia, że wszystkie zapytania we wskazanej domenie będą rozwiązywane lokalnie, tj.
w oparciu o plik `/etc/hosts` lub lease wydawane za sprawą protokołu DHCP. Opcja `domain` ustawia
domenę dla routera. W efekcie wszystkie hosty, które otrzymują konfiguracje od serwera DHCP, będą
skonfigurowane na tę domenę. Dodatkowo, nazwa określona tutaj jest wykorzystywana w opcji
`expandhosts` , która z kolei odpowiada za dodanie części domenowej do nazwy hostów, które jej nie
posiadają.

    option local '/lan/'
    option domain 'lan'
    option expandhosts '1'

Opcja `nohosts` określa czy wpisy znajdujące się w pliku `/etc/hosts` mają być brane pod uwagę przy
rozwiązywaniu nazw hostów.

    option nohosts '0'

Opcja `rebind_protection` zapobiega przekierowaniu w oparciu od DNS, tj. wpisujemy jedną domenę, a
jest nam zwracana inna, tzw [DNS rebinding][3]. Za pomocą opcji `rebind_localhost` z tego
mechanizmu jest wyłączony blok 127.0.0.0/8. A to ze względu na fakt, że szereg lokalnych usług
wymaga tego typu operacji do poprawnego działania. Jeśli jednak są domeny, które życzylibyśmy sobie
również wyłączyć spod mechanizmu DNS rebinding, to możemy je sprecyzować w `list rebind_domain` .

    option rebind_protection '1'
    option rebind_localhost '1'
    list rebind_domain 'free.aero2.net.pl'

W przypadku nieudanego rozwiązania nazwy na adres IP, takie zapytanie może być buforowane i zajmować
miejsce w pamięci. Za pomocą opcji `nonegcache` możemy wyłączyć lub włączyć dodawanie takich adresów
do pamięci cache.

    option nonegcache '1'

Zapytania DNS zajmują trochę czasu i w przypadku, gdy odwiedzamy jedną stronę kilka razy, to za
każdym razem trzeba rozwiązać jej nazwę na adres IP. Przynajmniej takie jest domyślne zachowanie w
systemach z rodziny linux, bo te nie posiadają cache DNS. Co prawda, realizacja takiego żądania nie
trwa długo ale zawsze jest to parę milisekund. W tym celu stworzono mechanizm, który buforuje
zapytania DNS. Gdy odwiedzamy ponownie ten sam adres, odpowiedź na żądanie pochodzi od `dnsmasq` i
jest zwracana niemal natychmiast. Oczywiście, o ile taki wpis figuruje w cache i jest ważny. Rozmiar
takiego cache (ilość wpisów) można dostosować sobie przy pomocy opcji `cachesize` .

    option cachesize '10000'

Kolejne dwie opcje, tj. `min-cache-ttl` oraz `max-cache-ttl` , nie mogą zostać określone w pliku
`/etc/config/dhcp` i trzeba je umieścić w pliku `/etc/dnsmasq.conf` . Odpowiadają one za [czas
ważności rekordu DNS w cache][4]. Po upłynięciu tego czasu trzeba będzie ponownie odpytać
upstream'owy serwer DNS, by rozwiązać nazwę.

    min-cache-ttl=3600
    max-cache-ttl=7200

Opcja `dnsforwardmax` odpowiada za ilość jednoczesnych zapytań, które mogą być obsługiwane przez
serwer DNS.

    option dnsforwardmax '1000'

Przy pomocy opcji `logqueries` zapytania DNS można logować w celu późniejszego debug'owa.

    option logqueries '1'

Za pomocą opcji `resolvfile` możemy określić nazwę pliku, w którym to będą znajdować się wpisy z
adresami upstream'owych serwerów DNS. Standardowo taki plik jest generowany automatycznie przy
starcie routera i umieszczany w katalogu `/tmp/` .

    option resolvfile '/tmp/resolv.conf.auto'

W OpenWRT plik `/etc/resolv.conf` jest podlinkowany do `/tmp/resolv.conf` . Znajduje się w nim
lokalny adres serwera DNS (routera), do którego to wędrują wszelkie zapytania o domeny z sieci
lokalnej. Opcja `noresolv` sprawi, że `dnsmasq` nie będzie używał DNS'ów pobranych z lease DHCP od
ISP. Trzeba jednak pamiętać, że by nazwy domen były rozwiązywane, to musimy określić jakiś serwer
DNS. Za to odpowiadają wpisy z `list server` . Natomiast ostatni wpis poniżej sprawi, że wszelkie
zapytania o domeny (i subdomeny) `google.com` będą rozwiązywane przez góglowski serwer DNS.

    option noresolv '1'
    list server '208.67.222.222'
    list server '208.67.220.220'
    list server '/google.com/8.8.8.8'

Jeśli chcemy określić statyczny wpis DNS dla jakiegoś adresu IP, to możemy to zrobić albo przez
wspomniany wyżej plik `/etc/hosts` , albo definiując wpis w konfiguracji `dnsmasq` . Jeśli
zdecydujemy się na tę drugą opcję, to w pliku `/etc/config/dhcp` musimy zdefiniować blok `config
domain` . Ten blok ma poniższą postać:

    config domain
        option name 'bdi.free.aero2.net.pl'
        option ip '10.2.37.78'

## Serwer DHCP

Opcje dotyczące modułu DNS w `dnsmasq` mamy z głowy. Teraz przyszła pora, by rozprawić się z opcjami
dotyczącymi serwera DHCP. Przede wszystkim, jeśli mamy w sieci tylko jeden router i żadna inna
maszyna nie świadczy usługi DHCP, to serwer obecny w routerze jest typu `authoritative` . W
warunkach domowych jest to prawdziwa sytuacja praktycznie w 100%. Dlatego też by przyśpieszyć
przydzielanie lease w konfiguracji OpenWRT dla `dnsmasq` mamy ustawioną poniższą opcję.

    option authoritative '1'

Trzeba także określić zakres adresów IP przydzielanych lokalnym maszynom przez serwer DHCP na
routerze. Przy czym, interesuje nas jedynie interfejs `lan` . Dlatego też interfejs `wan` poniżej ma
ustawioną flagę `ignore` . W ten sposób wskazujemy `dnsmasq` , by uruchamiał serwer DHCP tylko na
określonych interfejsach sieciowych.

    config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option limit '150'
        option leasetime '12h'
        option dhcpv6 'server'
        option ra 'server'

    config dhcp 'wan'
        option interface 'wan'
        option ignore '1'

Standardowa sieć w OpenWRT to 192.168.1.0/24 (192.168.1.1-192.168.1.254). Wyżej w bloku `config dhcp
'lan'` mamy szereg opcji, które mówią serwerowi DHCP jak ma się zachowywać. Według opcji `start` ,
pierwszy adres IP otrzymany via DHCP będzie miał numer `192.168.1.100` . Z kolei opcja `limit` ,
określa ile adresów może być do dyspozycji serwera. W tym wypadku `150` , czyli pula adresów
przydzielana przez serwer DHCP to 192.168.1.100-192.168.1.249. Możemy także określić ważność lease
DHCP. Jako, że są to komputery w sieci domowej, możemy ten parametr ustawić na 24-48 godzin. Po
upływie połowy określonego czasu, lease traci ważność. W takim wypadku host ponownie odpytuje
serwer DHCP i poprosi o konfigurację sieci. Opcje `dhcpv6` oraz `ra` odpowiadają za konfigurację
protokołu IPv6.

Niżej mamy parametr określający maksymalną ilość lease DHCP. Dobrze jest ten parametr skontrastować
z pulą adresów, którą dysponuje serwer DHCP (widoczną powyżej).

    option dhcpleasemax '150'

## Statyczne lease DHCP

Przy pomocy `dnsmasq` jesteśmy także w stanie definiować statyczne lease DHCP. Możemy to zrobić z
grubsza na dwa sposoby. Pierwszy z nich polega na skorzystaniu z opcji `readethers` w pliku
`/etc/config/dhcp` . Za jej sprawą przeszukiwany jest plik `/etc/ethers` , gdzie znajduje się baza
adresów MAC wraz z przypisanymi im adresami IP.

    option readethers '1'

Sam plik `/etc/ethers` wygląda następująco:

    3c:4a:92:00:4c:5b      192.168.1.150
    00:11:22:33:44:55      192.168.1.200

Statyczne lease DHCP możemy także zdefiniować bezpośrednio w konfiguracji `dnsmasq` przez dodawanie
bloków podobnych do tego poniżej:

    config host
        option ip       '192.168.1.166'
        option mac      '00:16:e6:34:c4:e0'
        option name     'the-hound'

Taki zapis sprawi, że hostowi identyfikującemu się adresem MAC `00:16:e6:34:c4:e0` zostanie
przypisany adres IP `192.168.1.166` i nazwa `the-hound` .

## Opcje dla klientów DHCP

W pliku `/etc/confg/dhcp` możemy także zdefiniować opcje, które będą przesyłane z lease DHCP do
hostów w sieci lokalnej. Pełna lista wszystkich dostępnych parametrów [znajduje się tutaj][5].
Poniżej przykład:

    config dhcp 'lan'
        ...
        list 'dhcp_option' '3,192.168.1.1'
        list 'dhcp_option' '6,8.8.8.8,8.8.4.4'

Mamy tutaj dwie opcje, z których pierwsza posiada numer `3` , druga zaś `6` . Zgodnie z powyższym
linkiem są to opcje `router` oraz `serwer DNS` . Za pomocą pierwszej opcji jesteśmy w stanie ustawić
adres bramy sieciowej na hostach. Oczywiście w przypadku, gdy maszyna z serwerem DHCP nie jest
jednocześnie bramą sieciową. Natomiast jeśli chodzi o drugą opcję, to określa ona adresy serwerów
DNS, które powinny zostać ustawione na hostach pobierających lease DHCP. Standardowo router
uzupełnia to pole wpisując swój własny adres, po czym wszystkie zapytania DNS skierowane do niego
są forward'owane zwykle do serwerów DNS naszego ISP.


[1]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[2]: https://wiki.openwrt.org/doc/uci/dhcp
[3]: https://en.wikipedia.org/wiki/DNS_rebinding
[4]: https://en.wikipedia.org/wiki/Time_to_live#DNS_records
[5]: https://www.iana.org/assignments/bootp-dhcp-parameters/bootp-dhcp-parameters.xml
