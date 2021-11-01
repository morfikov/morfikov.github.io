---
author: Morfik
categories:
- Linux
date:    2020-09-19 14:13:00 +0200
lastmod: 2020-09-19 14:13:00 +0200
published: true
status: publish
tags:
- debian
- dns
- dhcp
- dnscrypt
- doh
- dnsmasq
- prywatność
- bezpieczeństwo
- resolver
GHissueID: 6
title: Szyfrowany DNS z dnscrypt-proxy i dnsmasq na Debian linux
---

Ostatnio na forum dug.net.pl jeden z użytkowników [miał dość spory problem][1] z ogarnięciem
zadania polegającego na zaszyfrowaniu zapytań DNS z wykorzystaniem [dnscrypt-proxy][2] i
[dnsmasq][3]. Ładnych parę lat temu opisywałem [jak skonfigurować te dwa narzędzia na Debianie][4]
(i też na OpenWRT), choć od tamtego czasu w świecie linux'owym trochę rzeczy się pozmieniało. Dla
przykładu, dnscrypt-proxy przeszedł gruntowną przebudowę, no i też systemd jest w powszechniejszym
użyciu niż to miało miejsce w tamtych czasach, przez co w sporej części przypadków usługi takie jak
`systemd-networkd.service` czy `systemd-resolved.service` są już włączone domyślnie. Zatem sporo
informacji zawartych w tych napisanych przeze mnie artykułach już niekoniecznie może znaleźć
obecnie zastosowanie. Dlatego też pomyślałem, że nadszedł już czas, by ździebko zaktualizować tamte
wpisy. Ostatecznie stanęło jednak na tym, by w oparciu o te artykuły napisać kompletnie nowy tekst
na temat szyfrowania zapytań DNS na linux przy wykorzystaniu oprogramowania `dnscrypt-proxy` oraz
`dnsmasq` i zawrzeć w nim te wszystkie ciekawsze informacje, które udało mi się pozyskać przez te
ostatnie lata w kwestii poprawy bezpieczeństwa i prywatności przy przeglądaniu stron WWW.

<!--more-->
## Rozwiązywanie nazw DNS na linux

Jak zostało wspomniane we wstępnie, mamy zamiar zaszyfrować wszystkie zapytania DNS, które lokalne
aplikacje naszego linux'a będą generować podczas swojej codziennej pracy. Gdy chcemy odwiedzić
jakiś adres WWW (przykładowo w przeglądarce internetowej wpiszemy `kernel.org` ), to linux musi
pierw ustalić adres IP powiązany z domeną tego serwisu. Taki zabieg musi mieć miejsce z faktu, że
człowiek bardzo słabo operuje na liczbach, w przeciwieństwie do komputerów, i o wiele łatwiej jest
mu zapamiętać nazwę `kernel.org` niż adres IP w postaci `198.145.29.83` (o adresach IPv6 lepiej nie
wspominać). Dlatego potrzebna jest jakaś warstwa pośrednicząca, która te czytelne dla człowieka
nazwy zamieni na adresy IP i tym właśnie zajmuje się system DNS.

Aplikacje na linux korzystają przy rozwiązywaniu nazw domenowych na adresy IP z [Name Service
Switch][29] (NSS), który to jest częścią biblioteki GNU C (glibc). NSS daje możliwość dostarczania
systemowych baz danych jako osobne usługi, których kolejność przeszukiwania może zostać
skonfigurowana przez administratora systemu w pliku `/etc/nsswitch.conf` . W tym przypadku
najbardziej interesuje nas baza `hosts` , która na moim Debianie wygląda mniej więcej tak:

    ...
    hosts:         files dns myhostname
    ...

W skład bazy `hosts` wchodzi plik `/etc/hosts` (usługa `files`), resolver glibc, który czyta plik
`/etc/resolv.conf` (usługa `dns` ) oraz usługa `myhostname` . Zatem by rozwiązać nazwę domeny na
adres IP, system operacyjny przeszukuje pierw plik `/etc/hosts` ale tutaj zwykle odpowiedzi nie
znajdzie (chyba, że wycinamy adserwery i pozbywamy się w ten sposób reklam przy pomocy tego pliku).
Następnie do gry wchodzi resolver DNS, który przesyła zapytania DNS pod określony w pliku
`/etc/resolv.conf` adres IP. Zwykle server DNS zwraca odpowiedź w postaci adresu IP, pod którym
dana domena występuje. Jeśli zaś chodzi o `myhostname` , to ta usługa umożliwia dynamiczne
rozwiązywanie nazwy lokalnego hosta i na wielu systemach obecność i korzystanie z pliku `/etc/hosts`
jest jedynie opcjonalne. Chodzi generalnie o to, że szereg usług systemowych do prawidłowego
działania wymaga, by tę nazwę hosta można było łatwo rozwiązać na adres IP, a czasami zapis do
pliku `/etc/hosts` niekoniecznie jest możliwy (katalog `/etc/` w trybie tylko do odczytu) lub też
ciągłe przepisywanie tego pliku mija się z celem.

Większość aplikacji korzysta z NSS ale część z nich może bezpośrednio odczytywać pliki
`/etc/resolv.conf` i/lub `/etc/hosts` . Możliwe jest także pominięcie całej ten infrastruktury
systemowej przy rozwiązywaniu nazw, przez co aplikacje w swoim zakresie będą realizować funkcję
resolver'a, choć to komplikuje proces tworzenia aplikacji i raczej tego rozwiązania się nie stosuje.

### Wady linux'owego resolver'a

Ten resolver glibc ma szereg wad. Pierwszą z nich jest brak obsługi cache zapytań DNS, przez co
każde zapytanie (nawet o tę samą domenę chwilę później) trzeba przesyłać do upstream'owego serwera
DNS, co generuje opóźnienia i sprawia, że połączenie internetowe (zwłaszcza http/https) nam
trochę/bardzo spowalnia. Drugim problemem zaś jest brak możliwości szyfrowania zapytań DNS, przez
co cały ruch DNS zawierający wrażliwe pod kątem prywatności informacje przesyłany jest czystym
tekstem. Zatem każdy podmiot na drodze tych pakietów jest w stanie podejrzeć, jak i zmienić, nasze
zapytania DNS, co otwiera drogę dla szeroko rozumianej cenury treści w internecie, no i oczywiście
powoduje całą masę problemów natury bezpieczeństwa. Do tych dwóch kluczowych problemów trzeba
jeszcze dodać fakt, że ten linux'owy resolver nie jest w stanie przesyłać zapytań DNS na inny port
niż domyślny, tj. 53 (przynajmniej w Debianie). Jeśli chcemy wyeliminować którąkolwiek z tych wyżej
wymienionych wad, to musimy ratować się dodatkowym oprogramowaniem w postaci dnscrypt-proxy
(szyfrowanie zapytań DNS oraz cache DNS) lub/i dnsmasq (cache DNS oraz rozdzielanie zapytań DNS na
różne adresy/porty).

### Usługi systemd-networkd.service i systemd-resolved.service

W ostatnich latach, szereg większych dystrybucji linux'a (w tym Debian i Ubuntu) przesiadło się z
wykorzystywanego dotychczas [systemu inicjacji procesów SysVinit][30] na systemd. Systemd jest dość
rozbudowanym mechanizmem mającym za zadanie ogarniać procesy w systemie i gdyby skupiał się on
jedynie na tym zadaniu, to nie byłoby z nim problemu. Niemniej jednak, w gestii systemd leży także
m.in. obsługa sieci, w tym konfiguracja interfejsów sieciowych oraz rozwiązywanie nazw domen na
adresy IP.

W przypadku mojego Debiana, systemd po tylu latach wciąż nie potrafi ogarnąć konfiguracji mojej
sieci ze względu na fakt [wykorzystywania bonding'u][5] (połączenie interfejsów `eth0` oraz
`wlan0` ). No może i sam bonding jako tako można skonfigurować ale operowanie na nim przy użyciu
systemd jest ździebko upierdliwe oraz generuje [bardzo dziwne błędy][6], których nikt nie ma
pojęcia jak poprawić. Dlatego też ten cały moduł odpowiedzialny za konfigurację sieci w systemd, tj.
[usługę systemd-networkd.service][7], mam wyłączony:

    # systemctl disable systemd-networkd.service
    Removed /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service.
    Removed /etc/systemd/system/multi-user.target.wants/systemd-networkd.service.
    Removed /etc/systemd/system/dbus-org.freedesktop.network1.service.
    Removed /etc/systemd/system/sockets.target.wants/systemd-networkd.socket.

Usługa `systemd-networkd.service` w zasadzie konfiguruje jedynie interfejsy sieciowe. Wyłączenie
jej sprawia, że sami musimy zatroszczyć się o skonfigurowanie tych interfejsów. W moim Debianie
proces konfiguracji interfejsów sieciowych odbywa się za sprawą pakietu `ifupdown` , tj. przez plik
`/etc/network/interfaces` oraz usługę `networking.service` .

Oczywiście jeśli nie wykorzystujemy zaawansowanych mechanizmów sieciowych, np. wspomniany bonding,
to jest spore prawdopodobieństwo, że systemd będzie potrafił skonfigurować interfejsy sieciowe i
nie ma potrzeby wyłączać usługi `systemd-networkd.service` . Chciałem tutaj jedynie zaznaczyć, że w
tym artykule użytku z tej usługi robić nie będziemy i jeśli pojawią się jakieś problemy z nią
związane, to trzeba będzie samemu poszukać rozwiązania.

W skład modułu sieciowego systemd wchodzi także [usługa systemd-resolved.service][8], która ma za
zadanie zapewnić rozwiązywanie nazw DNS dla lokalnych aplikacji. Jeśli zamierzamy korzystać jedynie
z dnscrypt-proxy, to ta usługa `systemd-resolved.service` będzie w stanie poprawnie z nim pracować,
bo resolver dostarczany z systemd działa na innym adresie, tj. `127.0.0.53` , w stosunku do
`127.0.2.1` , na którym działa standardowo dnscrypt-proxy. Niemnie jednak, jeśli chcemy używać
dnsmasq, to trzeba wyłączyć `systemd-resolved.service` , bo te dwie usługi realizują podobne
zadania, tj. przesyłają otrzymane zapytania DNS do upstream'owego serwera DNS oraz każda z tych
usług ma zaimplementowany cache DNS. Jeśli chcemy uniknąć ewentualnych problemów, to lepiej nie
mieszać/wykorzystywać wielu rozwiązań mających realizować to samo zadanie w tym samym czasie.

Jako, że w tym artykule będziemy korzystać z dnscrypt-proxy i dnsmasq, to wyłączamy usługę
`systemd-resolved.service` :

    # systemctl disable systemd-resolved.service
    Removed /etc/systemd/system/multi-user.target.wants/systemd-resolved.service.
    Removed /etc/systemd/system/dbus-org.freedesktop.resolve1.service.

Generalnie chodzi o to, by przed przystąpieniem do pracy żadna usługa w systemie nie zajmowała
portu 53, co możemy sprawdzić przy pomocy `ss` czy `netstat` :

    # ss -lpn 'sport = :53'
    Netid     State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port     Process

    # netstat -napletu | grep :53

## Potrzebne oprogramowanie

Jeśli nie korzystamy z systemd i chcemy jedynie zaszyfrować zapytania DNS, to w zasadzie wystarczy
nam pakiet `dnscrypt-proxy` . Jeśli chcemy korzystać z bardziej zaawansowanych konfiguracji DNS,
np. rozdzielić ruch do różnych serwerów DNS i/lub część zapytań DNS słać w formie niezaszyfrowanej
(np. zapytania o lokalne domeny przesyłane do naszego domowego routera WiFi), to potrzebny nam
będzie także pakiet `dnsmasq` . Jeśli zaś zamierzamy korzystać z systemd i mamy włączoną usługę
`systemd-resolved.service` ), to możemy odpuścić sobie instalowanie dnsmasq. Tak czy inaczej, oba
te pakiety są dostępne w głównym repozytorium Debiana i bez większego problemu możemy je sobie
wgrać przy pomocy tego poniższego polecenia:

    # aptitude install dnsmasq dnscrypt-proxy

## Konfiguracja resolver'a glibc

Przede wszystkim nasz linux'owy resolver musi słać zapytania DNS na właściwy adres. Trzeba zatem
będzie dostosować plik `/etc/resolv.conf` . Standardowo do tego pliku trafiają wpisy typu
`nameserver 8.8.8.8` (DNS Google) czy `nameserver 1.1.1.1` (DNS CloudFlare). W naszej konfiguracji
musimy tutaj podać adres, na którym będzie nasłuchiwał dnsmasq i w tym przypadku jest to
`127.0.0.1` . Oczywiście ten adres można sobie dostosować. Grunt, by określić ten sam adres później
w konfiguracji dnsmasq. Zatem dodajemy do `/etc/resolv.conf` poniższy wpis:

    nameserver 127.0.0.1

Dodatkowo, z racji wykorzystywania lokalnego serwera DNS [musimy także ustawić bit AD][19]
(Authenticated Data), który oznaczy nasz lokalny serwer DNS jako zaufany:

    options trust-ad

Tylko te dwa powyższe wpisy powinny się znaleźć w pliku `/etc/resolv.conf` .

Warto tutaj jeszcze dodać, że jeśli wykorzystywaliśmy usługę `systemd-resolved.service` , to plik
`/etc/resolv.conf` w takiej konfiguracji jest zwykle linkiem do pliku
`/run/systemd/resolve/stub-resolv.conf` . Po wyłączeniu tej usługi, ten link nie jest w żaden
sposób ruszany, tj. rozwiązywanie nazw DNS nam przestanie działać. Dlatego też trzeba ten link
usunąć i w jego miejsce stworzyć zwykły plik oraz uzupełnić go zawartością podaną wyżej:

    # rm /etc/resolv.conf
    # touch /etc/resolv.conf

## Konfiguracja dnsmasq

W pliku `/etc/resolv.conf` przekierowujemy zapytania DNS do lokalnej instancji dnsmasq, która w
tym przypadku nasłuchuje zapytań na adresie `127.0.0.1` i porcie `53` . Dlatego też potrzebna nam
jest odpowiednia usługa dla systemd oraz plik konfiguracyjny dla dnsmasq. W Debianie, z pakietem
dnsmasq jest dostarczana usługa `/lib/systemd/system/dnsmasq.service` ale nie ma ona wymaganych
zależności i dlatego będziemy korzystać z własnej usługi dla systemd. Poniższą zawartość zapisujemy
w pliku `/etc/systemd/system/dnsmasq.service` :

    [Unit]
    Description=dnsmasq - A lightweight DHCP and caching DNS server
    Requires=network.target dnscrypt-proxy.service
    Wants=nss-lookup.target
    Before=nss-lookup.target
    After=network-online.target dnscrypt-proxy.service

    [Service]
    Type=forking
    PIDFile=/run/dnsmasq/dnsmasq.pid

    # Test the config file and refuse starting if it is not valid.
    ExecStartPre=/usr/sbin/dnsmasq --test

    # We run dnsmasq via the /etc/init.d/dnsmasq script which acts as a
    # wrapper picking up extra configuration files and then execs dnsmasq
    # itself, when called with the "systemd-exec" function.
    ExecStart=/etc/init.d/dnsmasq systemd-exec

    # The systemd-*-resolvconf functions configure (and deconfigure)
    # resolvconf to work with the dnsmasq DNS server. They're called like
    # this to get correct error handling (ie don't start-resolvconf if the
    # dnsmasq daemon fails to start).
    ExecStartPost=/etc/init.d/dnsmasq systemd-start-resolvconf
    ExecStop=/etc/init.d/dnsmasq systemd-stop-resolvconf


    ExecReload=/bin/kill -HUP $MAINPID

    [Install]
    WantedBy=multi-user.target

W zasadzie to zostały zmienione tutaj dwie rzeczy. Pierwszą z nich jest dodanie usługi
`dnscrypt-proxy.service` w `Requires=` i `After=` . Chodzi o to, by dnsmasq został uruchomiony po
dnscrypt-proxy, a nie odwrotnie, co może czasami powodować problemy z rozwiązywaniem nazw DNS.
Dodatkowo, te zależności sprawiają, że ilekroć będziemy restartować (czy też zatrzymywać) usługę
dnscrypt-proxy, to również i usługa dnsmasq będzie restartowana/zatrzymywana, jako że dnsmasq w
takiej konfiguracji nie może poprawnie działać bez dnscrypt-proxy i by oszczędzić sobie nerw przy
ewentualnym debugowaniu, dobrze jest te zależności uzupełnić.

Poniżej jest przykład logu w sytuacji, gdy postanowimy zatrzymać dnscrypt-proxy:

    systemd[1]: Stopping dnsmasq.service...
    dnsmasq[609718]: exiting on receipt of SIGTERM
    systemd[1]: dnsmasq.service: Succeeded.
    systemd[1]: Stopped dnsmasq.service.
    dnscrypt-proxy[609712]: [2020-09-17 16:43:03] [NOTICE] Stopped.
    systemd[1]: Stopping dnscrypt-proxy.service...
    systemd[1]: dnscrypt-proxy.service: Succeeded.
    systemd[1]: Stopped dnscrypt-proxy.service.

A tutaj z kolei jest proces restartu dnscrypt-proxy:

    systemd[1]: Stopping dnsmasq.service...
    dnsmasq[614691]: exiting on receipt of SIGTERM
    systemd[1]: dnsmasq.service: Succeeded.
    systemd[1]: Stopped dnsmasq.service.
    dnscrypt-proxy[614682]: [2020-09-17 16:45:44] [NOTICE] Stopped.
    systemd[1]: Stopping dnscrypt-proxy.service...
    systemd[1]: dnscrypt-proxy.service: Succeeded.
    systemd[1]: Stopped dnscrypt-proxy.service.
    systemd[1]: Started dnscrypt-proxy.service.
    systemd[1]: Starting dnsmasq.service...
    dnsmasq[614741]: dnsmasq: syntax check OK.
    dnsmasq[614751]: started, version 2.82 cachesize 4096
    dnsmasq[614751]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect inotify dumpfile
    dnsmasq[614751]: DBus support enabled: connected to system bus
    dnsmasq-dhcp[614751]: DHCP, IP range 192.168.122.2 -- 192.168.122.254, lease time 12h
    dnsmasq-dhcp[614751]: DHCP, sockets bound exclusively to interface virbr0
    dnsmasq[614751]: using only locally-known addresses for domain lh
    dnsmasq[614751]: using nameserver 192.168.1.1#53 for domain mhouse
    dnsmasq[614751]: using nameserver 127.0.2.1#53
    dnsmasq[614751]: read /etc/hosts - 14 addresses
    systemd[1]: Started dnsmasq.service.
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] dnscrypt-proxy 2.0.44
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Network connectivity detected
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Dropping privileges
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Network connectivity detected
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Now listening to 127.0.2.1:53 [UDP]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Now listening to 127.0.2.1:53 [TCP]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Now listening to https://127.0.2.1:3000/dns-query [DoH]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Source [public-resolvers] loaded
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Source [relays] loaded
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Loading the set of whitelisting rules from [/etc/dnscrypt-proxy/whitelist.txt]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Firefox workaround initialized
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Loading the set of blocking rules from [/etc/dnscrypt-proxy/blacklist.txt]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Loading the set of cloaking rules from [/etc/dnscrypt-proxy/cloaking-rules.txt]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Loading the set of forwarding rules from [/etc/dnscrypt-proxy/forwarding-rules.txt]
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [INFO] [cloudflare] TLS version: 304 - Protocol: h2 - Cipher suite: 4865
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] [cloudflare] OK (DoH) - rtt: 60ms
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] Server with the lowest initial latency: cloudflare (rtt: 60ms)
    dnscrypt-proxy[614740]: [2020-09-17 16:45:45] [NOTICE] dnscrypt-proxy is ready - live servers: 1

Jak widać, w obu przypadkach te dopisane zależności troszczą się o odpowiednią kolejność
zatrzymywania i uruchamiania tych dwóch usług. Oczywiście, możemy także obejść się bez tych
dodatkowych zależności.

Drugą zmianą, którą wprowadza nasza usługa, jest przepisanie w `After=` targetu `network.target` na
`network-online.target` . Chodzi generalnie o to, że dnscrypt-proxy potrzebuje aktywnego połączenia
sieciowego, by móc pobrać aktualną listę serwerów DNS. [Zgodnie z tym co możemy wyczytać tutaj][31],
target `network.target` nie daje gwarancji, że ta sieć będzie działać, w przeciwieństwie do
`network-online.target` i dlatego by uniknąć błędów w logu związanych z dnscrypt-proxy dobrze jest
przepisać również i te zależności.

### Plik /etc/dnsmasq.conf

Po skonfigurowaniu zależności w usłudze systemd dla dnsmasq, możemy przejść do napisania
konfiguracji dla  tego demona. Oczywiście dnsmasq jest rozbudowanym kawałkiem oprogramowania i
liczba jego zastosowań jest dość spora. Dlatego też poniżej znajdzie się opis konfiguracji, której
używam na co dzień w moim Debianie. Nie wszystkie opcje są potrzebne, by szyfrowanie zapytań DNS
wdrożyć, dlatego też trzeba uważnie czytać opis poszczególnych opcji i ocenić czy będą one
użyteczne w naszym przypadku czy też nie.

Poniższą zwrotkę zapisujemy w pliku `/etc/dnsmasq.conf` :

    enable-dbus
    port=53
    domain-needed
    bogus-priv
    stop-dns-rebind
    rebind-localhost-ok
    rebind-domain-ok=free.aero2.net.pl
    no-resolv
    no-poll
    server=127.0.2.1#53
    server=/mhouse/192.168.1.1#53
    local=/lh/
    address=/bdi.free.aero2.net.pl/10.2.37.78
    user=dnsmasq
    group=nogroup
    interface=lo
    no-dhcp-interface=lo
    bind-interfaces
    expand-hosts
    domain=lh
    dns-forward-max=256

Poniżej jest wyjaśnienie użytych opcji:

- `enable-dbus` włącza opcjonalne wsparcie dla D-Bus. Dnsmasq jest w stanie bez większego problemu
działać i bez D-Bus.
- `port` odpowiada za port, na którym dnsmasq będzie nasłuchiwał. Domyślnie jest to `53` i jeśli
nam on nie odpowiada, to możemy tutaj ustawić inny port, choć linux'owy resolver wspiera póki co
tylko ten port.
- `user` oraz `group` mają na celu zrzucenie uprawnień, tak by proces dnsmasq nie działał z
uprawnieniami root.
- `domain-needed` ma za zadanie odfiltrować zapytania, na które nie potrafią odpowiedzieć publiczne
serwery DNS, tj. nie forwarduje on nazw bez kropek (bez części domeny).
-  `bogus-priv` ma za zadanie uniemożliwić przekazywanie do upstream'owych serwerów DNS zapytań dla
prywatnych zakresów IP (np. 192.168.0.0/16) przy odwrotnej translacji adresów DNS
([Reverse DNS][28]).
- `local` sprawia, że wszystkie zapytania we wskazanej domenie będą rozwiązywane lokalnie, tj. w
oparciu o plik `/etc/hosts` lub lease wydawane za sprawą protokołu DHCP.
- `domain` jest bardzo podobny do `local` i ustawia domenę dla serwera DHCP, którego my nie
będziemy wykorzystywać. W efekcie wszystkie hosty, które otrzymują konfigurację od serwera DHCP
(np. maszyny wirtualne), będą skonfigurowane na tę domenę. Dodatkowo, nazwa określona tutaj jest
wykorzystywana w opcji `expand-hosts` .
- `expand-hosts` ma za zadanie dodać część domenową określoną w `domain` przy korzystaniu z
prostych nazw. Dla przykładu, gdyby odpytać `morfikownia` , to dnsmasq automatycznie doda do tej
nazwy `.lh` i w ten sposób powstanie `morfikownia.lh` i to ta nazwa zostanie rozwiązana na adres IP.
- `no-resolv` sprawia, że nie będzie czytany plik `/etc/resolv.conf` w poszukiwaniu upstream'owych
serwerów DNS.
- `no-poll` ma za zadanie powstrzymać dnsmasq przed monitorowaniem zmian w pliku `/etc/resolv.conf`
i innych podobnych plikach, które biorą udział w rozwiązywaniu nazw DNS.
- `server` określa upstream'owe serwery DNS, do których dnsmasq będzie przesyłał zapytania o domeny.
Wpisów może być kilka, a to do którego serwera poleci zapytanie DNS jest precyzowane po domenie.
Przykładowo, wpis `server=/mhouse/192.168.1.1#53` mówi, że zapytanie o domenę `mhouse` (i jej
wszystkie subdomeny, np. `morfikownia.mhouse` ) zostanie skierowane do serwera `192.168.1.1` na
port `53` bezpośrednio z pominięciem szyfrowania via dnscrypt-proxy. Natomiast wpis
`server=127.0.2.1#53` określa, że wszystkie pozostałe zapytania DNS mają być kierowane na adres
`127.0.2.1` port `53` , na którym to będzie nasłuchiwał dnscrypt-proxy i tylko te zapytania będą
podlegać szyfrowaniu.
- `interface` określa interfejs, na którym będzie nasłuchiwał serwer DNS. Można określić kilka
interfejsów podając parokrotnie nazwę tego parametru. Można także skorzystać z `listen-address` i
zamiast interfejsu postać adres (lub adresy) IP. Jako, że usługa DNS zalicza się do tych
wrażliwych, to nie powinniśmy jej wystawiać na interfejsach publicznych i powinniśmy się ograniczyć
jedynie do interfejsu pętli zwrotnej ( `lo` ).
- `no-dhcp-interface` ma za zadanie nie uruchamiać serwera DHCP na interfejsie pętli zwrotnej.
Chodzi o to, że dnsmasq poza usługami DNS jest w stanie także robić za serwer DHCP. Nam ta
funkcjonalność jest zupełnie zbędna, dlatego wyłączamy serwer DHCP dla pętli zwrotnej.
- `bind-interfaces` ma za zadanie przypisać proces dnsmasq do danego interfejsu, przez co gdy taki
interfejs zniknie i pojawi się ponownie, np. podczas rekonfiguracji połączenia sieciowego, to
dnsmasq będzie mógł działać jak gdyby nigdy nic. Ten parametr pozwala nam też uruchomić inne
oprogramowanie, które ma się zajmować rozwiązywaniem nazw DNS, np. dnscrypt-proxy. Bez
`bind-interfaces` , dnscrypt-proxy nie mógłby się uruchomić na interfejsie pętli zwrotnej i
zwróciłby błąd `[FATAL] listen udp 127.0.2.1:53: bind: address already in use` .
- `dns-forward-max` odpowiada za ilość jednoczesnych zapytań, które mogą być obsługiwane przez
serwer DNS.
- `stop-dns-rebind` zapobiega przekierowaniu w oparciu o DNS, tj. wpisujemy jedną domenę, a jest nam
zwracana inna, tzw. [DNS rebinding][20].
- `rebind-localhost-ok` zezwala na DNS rebinding ale tylko dla interfejsu pętli zwrotnej.
- `rebind-domain-ok` zezwala na DNS rebinding dla określonych domen. W tym przypadku chodzi o
konfigurację dla Aero2.
- `address` ma za zadanie przypisać określonej domenie konkretny adres IP. Podobnie jak w parametrze
wyżej, ta opcja dotyczy jedynie Aero2.

## Konfiguracja dnscrypt-proxy

Mamy zatem skonfigurowany resolver linux'owy wskazujący na adres IP, na którym nasłuchuje dnsmasq.
W konfiguracji dnsmasq mamy zaś określony upstream'owy serwer DNS jako `127.0.2.1#53` . Trzeba
teraz tak skonfigurować dnscrypt-proxy, by na tym adresie i porcie nasłuchiwał zapytań DNS. Możemy
to zrobić na dwa sposoby, tj. wykorzystując [mechanizm gniazd systemd][21] lub też go kompletnie
pomijając.

### Konfiguracja dnscrypt-proxy z gniazdami systemd

Standardowa konfiguracja dnscrypt-proxy w Debianie zakłada wykorzystanie mechanizmu gniazd systemd.
W skrócie, ten mechanizm ma za zadanie oszczędzanie zasobów systemowych przez powstrzymywanie
uruchamiania usług do momentu, aż będą one potrzebne. W tym przypadku, proces dnscrypt-proxy nie
zostanie uruchomiony chyba, że jakiś proces będzie chciał wysłać zapytanie DNS. To zapytanie
zostanie przechwycone, a w międzyczasie systemd uruchomi dnscrypt-proxy. Jak tylko ten proces
zostanie odpalony, to zapytanie DNS zostanie mu przekazane i już wszystko dalej odbywać się będzie
po staremu. By skorzystać z mechanizmu gniazd, potrzebne nam są dwa pliki, które standardowo są
zawarte w paczce dnscrypt-proxy.

Plik `/lib/systemd/system/dnscrypt-proxy.service` :

    [Unit]
    Description=DNSCrypt client proxy
    Documentation=https://github.com/DNSCrypt/dnscrypt-proxy/wiki
    Requires=dnscrypt-proxy.socket
    After=network.target
    Before=nss-lookup.target
    Wants=nss-lookup.target

    [Install]
    Also=dnscrypt-proxy.socket
    WantedBy=multi-user.target

    [Service]
    NonBlocking=true
    ExecStart=/usr/sbin/dnscrypt-proxy -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml
    ProtectHome=true
    ProtectKernelModules=true
    ProtectKernelTunables=true
    ProtectControlGroups=true
    MemoryDenyWriteExecute=true

    User=_dnscrypt-proxy
    CacheDirectory=dnscrypt-proxy
    LogsDirectory=dnscrypt-proxy
    RuntimeDirectory=dnscrypt-proxy

Plik `/lib/systemd/system/dnscrypt-proxy.socket` :

    [Unit]
    Description=dnscrypt-proxy listening socket
    Documentation=https://github.com/DNSCrypt/dnscrypt-proxy/wiki
    Before=nss-lookup.target
    Wants=nss-lookup.target
    Wants=dnscrypt-proxy-resolvconf.service

    [Socket]
    ListenStream=127.0.2.1:53
    ListenDatagram=127.0.2.1:53
    NoDelay=true
    DeferAcceptSec=1

    [Install]
    WantedBy=sockets.target

W pliku usługi `dnscrypt-proxy.service` mamy zależność `Requires=dnscrypt-proxy.socket` . W
konfiguracji gniazda mamy zaś `ListenStream=` (protokół TCP) oraz `ListenDatagram=` (protokół UDP),
w których mamy określony port i adres `127.0.2.1:53` . Zatem systemd podczas startu systemu
stworzy gniazdo dla pakietów TCP i UDP na tym konkretnym adresie i porcie. Ustawienie opcji
`NoDelay` ma za zadanie wyłączyć algorytm Nagle'a, którego celem jest połączenie wielu mniejszych
segmentów TCP w jeden większy i przesłanie większego pakietu przez sieć. Bez tej opcji, system
byłby w stanie zapakować wiele zapytań DNS w jeden pakiet i posłać go przez sieć. Takie podejście
oszczędza łącze internetowe ale generuje przy tym opóźnienia, których powinniśmy unikać w protokole
DNS. Dlatego algorytm Nagle'a dla gniazd dnscrypt-proxy lepiej jest wyłączyć, by te zapytania DNS
były wysyłane w świat jak tylko napłyną.

Jeśli chcemy wykorzystywać gniazda systemd, to upewnijmy się, że w pliku konfiguracyjnym
`/etc/dnscrypt-proxy/dnscrypt-proxy.toml` nie mamy ustawionego adresu, na którym dnscrypt-proxy ma
nasłuchiwać, tj. parametr `listen_addresses` ma zostać pusty:

    listen_addresses = []

Korzystanie z gniazd systemd ma tę zaletę, że to systemd będzie otwierał gniazda, a nie sam proces
aplikacji. Gdy pojawią się pierwsze pakiety adresowane na konkretny IP i port, to systemd uruchomi
usługę powiązaną z plikiem `.socket` i przekaże jej wszystkie otworzone gniazda.  W ten sposób
odpada nam przyznawanie uprawnień, np. `CAP_NET_BIND_SERVICE` , by proces mógł nasłuchiwać na
porcie o numerze niższym niż 1024, czy też `CAP_SETGID` i `CAP_SETUID` , które mają na celu pomóc
procesowi uruchomionemu jako root zrzucić uprawnienia, tak by działał on jako user `_dnscrypt-proxy`
z grupą `nogroup` . Przy korzystaniu z gniazd, systemd uruchamia dnscrypt-proxy jako określony
user/grupa i odpada nam ta cała zabawa ze zrzucaniem uprawnień i nie trzeba się martwić o to czy
wykorzystywane przez tę usługę porty są z zakresu uprzywilejowanego (<1024), do którego ma dostęp
jedynie root.

### Konfiguracja dnscrypt-proxy bez gniazd systemd

Alternatywnym podejściem jest rezygnacja z gniazd systemd. Niemniej jednak, to zadanie wymaga od
nas przepisania domyślnej usługi systemd oraz wyłączenia usługi gniazd dla dnscrypt-proxy. Tworzymy
zatem plik `/etc/systemd/system/dnscrypt-proxy.service` i dodajemy do niego poniższą treść:

    [Unit]
    Description=DNSCrypt client proxy
    Documentation=https://github.com/DNSCrypt/dnscrypt-proxy/wiki
    #Requires=dnscrypt-proxy.socket
    After=network-online.target
    Before=nss-lookup.target
    Wants=nss-lookup.target

    [Service]
    NonBlocking=true
    ExecStart=/usr/sbin/dnscrypt-proxy -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml
    AmbientCapabilities=CAP_NET_BIND_SERVICE,CAP_SETGID,CAP_SETUID
    NoNewPrivileges=true
    ProtectHome=true
    ProtectSystem=strict
    ProtectKernelModules=true
    ProtectKernelTunables=true
    ProtectControlGroups=true
    MemoryDenyWriteExecute=true
    SystemCallArchitectures=native
    PrivateTmp=true
    PrivateDevices=true
    RestrictRealtime=true
    RemoveIPC=true
    KeyringMode=private
    #User=_dnscrypt-proxy
    CacheDirectory=dnscrypt-proxy
    LogsDirectory=dnscrypt-proxy
    RuntimeDirectory=dnscrypt-proxy
    ReadWritePaths=/var/log/dnscrypt-proxy /var/cache/dnscrypt-proxy /etc/dnscrypt-proxy
    ReadOnlyPaths=/usr/bin/dnscrypt-proxy
    LogsDirectoryMode=0644

    [Install]
    #Also=dnscrypt-proxy.socket
    WantedBy=multi-user.target

Kluczowe w tej usłudze jest wykomentowanie linijek `Requires=dnscrypt-proxy.socket` w sekcji
`[Unit]` , `User=_dnscrypt-proxy` w sekcji `[Service]` oraz `Also=dnscrypt-proxy.socket` w sekcji
`[Install]` . Tak przygotowaną usługę zapisujemy. Wyłączamy przy tym też usługę gniazd i
przeładowujemy konfigurację systemd:

    # systemctl disable dnscrypt-proxy.socket
    # systemctl daemon-reload

### Plik /etc/dnscrypt-proxy/dnscrypt-proxy.toml

W zależności od wybranego sposobu uruchamiania trzeba będzie inaczej skonfigurować dnscrypt-proxy.
W zasadzie domyślny plik `/etc/dnscrypt-proxy/dnscrypt-proxy.toml` , który jest dostarczany z
pakietem Debiana, zawiera wszystkie niezbędne wpisy, które mają za zadanie sprawić, by
dnscrypt-proxy działał poprawnie w konfiguracji z gniazdami systemd i nie ma potrzeby ruszać tego
pliku. W przypadku, gdy nie korzystamy z gniazd systemd albo w ogóle nie używamy tego init'a, to
trzeba będzie nieco inaczej skonfigurować tego demona szyfrującego zapytania DNS.

W katalogu `/usr/share/doc/dnscrypt-proxy/examples/` znajduje się dość rozbudowany plik
`example-dnscrypt-proxy.toml` , którego nazwę trzeba zmienić na `dnscrypt-proxy.toml` i skopiować
do katalogu `/etc/dnscrypt-proxy/` . Poniżej zaś znajduje się krótki opis opcji, na które
powinniśmy rzucić okiem w celu ich dostosowania.

Przede wszystkim musimy zadbać, by w tym pliku znalazł się adres IP oraz port, na którym
dnscrypt-proxy będzie nasłuchiwał zapytań od dnsmasq, tj. `127.0.2.1` i port `53` (ewentualnie też
`[::1]` jeśli nasz ISP zapewnia połączenie z siecią za sprawą protokołu IPv6):

    #listen_addresses = ['127.0.0.1:53', '[::1]:53']
    listen_addresses = ['127.0.2.1:53']

Zapytania DNS muszą być przesyłane do jakiegoś upstream'owego serwera DNS. W tym przypadku
wykorzystywany będzie [CloudFlare][32] ale oczywiście możemy określić taki serwer DNS, który nam
pasuje i niekoniecznie musi to być CloudFlare. [Lista dostępnych serwerów znajduje się tutaj][27].
Istnieje także możliwość zdefiniowania wielu upstream'owych serwerów DNS, np. w celu automatycznego
doboru tego najszybszego z nich lub też przesyłania części zapytań DNS do jednego serwera, a części
do innego:

    # server_names = ['scaleway-fr', 'google', 'yandex', 'cloudflare']
    server_names = ['cloudflare']

Bez gniazd systemd, usługa dnscrypt-proxy jest uruchamiana jako użytkownik root. Nie powinna ona
jednak działać z prawami administratora systemu, dlatego też twórcy tej aplikacji zaprojektowali ją
w taki sposób, by w pewnym momencie po uruchomieniu procesu można było zrzucić uprawnienia, tak by
ten proces był słabiej uprzywilejowany. Do zrzucenia uprawnień potrzebujemy jednak określić
użytkownika. Przy instalacji pakietu dnscrypt-proxy w Debianie, tworzony jest użytkownik
`_dnscrypt-proxy` i to z niego możemy skorzystać:

    user_name = '_dnscrypt-proxy'

Możemy także wymusić, by zapytania DNS szły protokołem TCP ale lepiej tego nie robić. Jeśli nie
korzystamy z Tor'a do przesyłania zapytań DNS, to w zasadzie nie ma potrzeby wymuszać, by
komunikacja z upstream'owymi serwerami DNS odbywała się z wykorzystaniem protokołu TCP.
Dnscrypt-proxy zawsze szyfruje zapytania DNS, nawet w przypadku, gdy wykorzystywany jest protokół
UDP. Wymuszenie protokołu TCP nie poprawia w żaden sposób bezpieczeństwa procesu rozwiązywania nazw
domen na adresy IP i może powodować jedynie dodatkowe opóźnienia spowalniając tym samym cały ten
proces:

    force_tcp = false

Jeśli w dnsmasq określiliśmy maksymalną ilość jednoczesnych zapytań DNS, to możemy zsynchronizować
tę poniższą wartość z tym, co określiliśmy w konfiguracji dnsmasq w parametrze `dns-forward-max` :

    max_clients = 256

W przypadku, gdy nasz operator ISP nie przydzielił nam żadnego adresu IPv6, to dnscrypt-proxy na
powiązane z tym protokołem zapytania DNS może zwracać pustą odpowiedź. Takie zachowanie przyśpieszy
proces rozwiązywania nazw na hostach niemających skonfigurowanego połączenia IPv6, choć niekiedy
może ono powodować problemy:

    block_ipv6 = true

Warto też zadbać o to, by zapytania DNS bez części domenowej (np. `morfikownia` vs.
`morfikownia.mhouse` ) nie były przesyłane do upstream'owych serwerów DNS. Podobnie sprawa ma się z
zapytaniami DNS ze [stref lokalnych][22]. Przesyłanie tego typu zapytań DNS do zewnętrznych
podmiotów sprawia, że cierpi na tym nasza prywatność i lepiej tego nie robić:

    block_unqualified = true
    block_undelegated = true

Standardowo, dnscrypt-proxy używa tego samego certyfikatu (klucza publicznego) przy rozwiązywaniu
nazw domenowych na adresy IP. W pewnych sytuacjach, np. w mobilnym świecie laptopów/smartfonów i
ich przemieszczaniu się między sieciami, ten certyfikat może pomóc zidentyfikować konkretne
urządzenie. By temu zaradzić, został stworzony mechanizm kluczy efemerycznych (Ephemeral Keys),
które są generowane dla każdego nowego zapytania DNS. Może i takie zachowanie poprawia naszą
prywatność ale trzeba się liczyć z mocniejszym wykorzystaniem procesora. Nie zaleca się włączać tej
opcji w przypadku, gdy nasza maszyna generuje tych zapytań bardzo dużo w krótkim czasie. Gdy mamy
przydzielony statyczny adres IP, to również możemy sobie darować ustawianie tej opcji:

    dnscrypt_ephemeral_keys = true

Poniższy parametr wyłącza śledzenie sesji TLS ([TLS session tickets][24]), co poprawia prywatność
ale kosztem większych opóźnień przy rozwiązywaniu domen, bo trzeba na nowo negocjować informacje
sesji TLS między klientem i serwerem za każdym razem, gdy klient próbuje nawiązać szyfrowane
połączenie.

    tls_disable_session_tickets = true

Możemy też określić szyfr wykorzystywany przy [szyfrowaniu zapytań][25]. Domyślnie jest
wykorzystywana wartość `4865` , która w zapisie HEX ma postać `0x1301` i odpowiada za szyfr
`TLS_AES_128_GCM_SHA256` . Możemy także tutaj ustawić `4866` ( `0x1302` ) lub `4867` ( `0x1303` )
odpowiednio dla `TLS_AES_256_GCM_SHA384` oraz `TLS_CHACHA20_POLY1305_SHA256` . Te trzy wyżej
wymienione szyfry dotyczą jedynie protokołu TLS 1.3 (nie zaleca się używanie szyfrów protokołu TLS
1.2 i niższych). Dodatkowo, nie ma raczej potrzeby korzystać z `TLS_AES_256_GCM_SHA384` , bo jest
on obecnie tak samo bezpieczny jak `TLS_AES_128_GCM_SHA256` ale , gdy chodzi o wydajność, to jest
on niestety sporo wolniejszy.

    tls_cipher_suite = [4865]

Poniższe opcje mają za zadanie opóźnić start dnscrypt-proxy w przypadku, gdy sieć nie jest jeszcze
dostępna, np. podczas fazy rozruchu systemu. Niemniej jednak, mając odpowiednio skonfigurowane
usługi systemd, to próbkowanie sieci jest zbędne i można je z powodzeniem wyłączyć:

    netprobe_timeout = 60
    netprobe_address = '9.9.9.9:53'

Ruch przez fallback resolver jest puszczony nieszyfrowany i ma w zasadzie na celu jedynie
umożliwienie pobrania listy serwerów DNS. Adres określony w poniższym parametrze nie będzie
nigdy wykorzystywany do rozwiązywania domen, o które proszą aplikacje użytkownika:

    fallback_resolvers = ['9.9.9.9:53']

Poniższy parametr ma na celu upewnienie się, że w przypadku, gdy zajdzie potrzeba skorzystania z
fallback resolver'a, to będzie to ten określony powyżej, a nie ten ustawiony w konfiguracji systemu:

    ignore_system_dns = true

Powinniśmy mieć także skonfigurowane jakieś [źródła resolver'ów][23]. Te dwa poniższe są domyślne:

    [sources]
      [sources.'public-resolvers']
      urls = ['https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v2/public-resolvers.md', 'https://download.dnscrypt.info/resolvers-list/v2/public-resolvers.md']
      cache_file = 'public-resolvers.md'
      minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
      refresh_delay = 72
      prefix = ''

    [sources.'relays']
      urls = ['https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v2/relays.md', 'https://download.dnscrypt.info/resolvers-list/v2/relays.md']
      cache_file = 'relays.md'
      minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
      refresh_delay = 72
      prefix = ''

Poniższa zwrotka odpowiada za skonfigurowanie lokalnego serwera DoH, który można wykorzystać w celu
[wdrożenia szyfrowanego SNI (ESNI) w Firefox][26]:

    [local_doh]
    listen_addresses = ['127.0.0.1:3000']
    path = "/dns-query"
    cert_file = "localhost.pem"
    cert_key_file = "localhost.pem"

## Cache DNS w dnsmasq i dnscrypt-proxy

Nowy dnscrypt-proxy dość znacznie różni się od swojego poprzednika, tj. posiada wsparcie dla
protokołów [DNSCrypt v2][17], [DoH][15] i [DoT][16]. Posiada on także obsługę cache DNS, przez co w
zasadzie korzystanie z dnsmasq traci powoli na znaczeniu. Jeśli korzystaliśmy do tej pory w
dnsmasq by mieć ten cache zapytań DNS, to z powodzeniem możemy się tego oprogramowania pozbyć.
Niemniej jednak, dnsmasq czasem przydaje się do rozdzielania ruchu DNS do różnych serwerów DNS i
jeśli potrzebujemy takiej funkcjonalności, to naturalnie dnsmasq będzie musiał działać równolegle z
dnscrypt-proxy. Pojawia się zatem problem związany z cache DNS. Musimy się zdecydować, które z tych
dwóch narzędzi będzie ten cache obsługiwać. Lepszym rozwiązaniem wydaje się być obsługa cache DNS w
dnsmasq, bo przez niego będą przechodzić wszystkie zapytania DNS, w tym też te, które będą
szyfrowane przy pomocy dnscrypt-proxy. Dodatkowo, dnsmasq występuje wcześniej w tym całym łańcuchu
rozwiązywania domen na adresy IP, przez co jego cache będzie szybszy. Dlatego też w konfiguracji
dnscrypt-proxy w pliku `/etc/dnscrypt-proxy/dnscrypt-proxy.toml` wyłączmy cache:

    cache = false

Natomiast w konfiguracji dnsmasq w pliku `/etc/dnsmasq.conf` , cache konfigurujemy w poniższy
sposób:

    cache-size=4096
    min-cache-ttl=600
    max-cache-ttl=3600

Rozmiar cache będzie miał `4096` wpisów. Każdy z wpisów zajmuje trochę pamięci operacyjnej i na
maszynach 32-bitowych, taki wpis w cache ma 82 lub 74 bajty w zależności od protokołu, tj. IPv6 lub
IPv4. W przypadku maszyn 64-bitowych, rozmiar wpisów wynosi odpowiednio 94 oraz 86 bajtów. Zatem
4096 wpisów zajmie około 350 KiB pamięci -- na desktopach taka wartość nie ma praktycznie większego
znaczenia. Niemniej jednak, im większy cache, tym wolniej trwa jego przeszukiwanie. Trzeba zatem
uważać by nie zdegradować sobie wydajności za sprawą przechowywania starych nieużywanych lub też
rzadko używanych wpisów.

Opcje `min-cache-ttl` oraz `max-cache-ttl` odpowiadają za widełki czasowe (określone w sekundach)
ważności rekordu DNS w cache. Odpytując upstream'owy serwer DNS o jakaś domenę, tworzony jest
rekord DNS w cache z określonym TTL (zwykle ten czas podawany jest w odpowiedzi zwrotnej razem z
adresem IP). Niektóre TTL mogą być bardzo krótkie (rzędu kilku-kilkudziesięciu sekund), a inne zbyt
długie (parę godzin czy nawet dni). Do momentu upłynięcia tego czasu określonego w TTL, odpowiedź
na każde zapytanie o tę określoną domenę będzie brana z cache. Jeśli czasy TTL są zbyt długie, to
rekord w cache może nie odpowiadać stanu faktycznemu (zmieni się adres IP dla danej domeny) i
przydałoby się unikać takiej sytuacji. Dlatego te czasy TTL powinniśmy sobie dostosować, tak by
system z jednej strony nie odpytywał upstream'owego serwera DNS zbyt często (bonus dla prywatności),
a z drugiej strony by te wpisy w cache były aktualne. Dlatego też można ustawić minimalny TTL na
10-30 minut, zaś maksymalny na 1-4 godziny.

W przypadku nieudanego rozwiązania nazwy na adres IP, taka odpowiedź również może być buforowana.
Zajmuje ona jednak cenne miejsce w cache. Jeśli zatem dodatkowo określimy `no-negcache` , to możemy
wyłączyć dodawanie takich wpisów do cache, przez co więcej prawidłowych wpisów będzie mogło się w
tym cache znaleźć bez potrzeby zwiększania jego rozmiaru i spowalniania całego mechanizmu:

    no-negcache

Odpytując teraz przykładową domenę przy pomocy `dig` możemy zweryfikować jaki TTL zostanie
ustawiony dla wpisu w cache:

    # dig mozilla.org
    ...
    ;; ANSWER SECTION:
    mozilla.org.            40      IN      A       63.245.208.195

    ;; Query time: 205 msec
    ;; SERVER: 127.0.0.1#53(127.0.0.1)
    ;; WHEN: Sun Sep 13 13:51:13 CEST 2020
    ;; MSG SIZE  rcvd: 56

Po czasie 205 ms możemy stwierdzić, że to zapytanie było rozwiązane bezpośrednio. W sekcji
odpowiedzi widnieje liczba `40` -- to jest właśnie TTL dla tego rekordu DNS zwracany z
upstream'owego serwera DNS. Po odpytaniu domeny, rekord DNS powędrował do cache dnsmasq. Jeśli
jeszcze raz odpytamy o tę domenę, to otrzymamy poniższy wynik:

    # dig mozilla.org
    ...
    ;; ANSWER SECTION:
    mozilla.org.            596     IN      A       63.245.208.195

    ;; Query time: 1 msec
    ;; SERVER: 127.0.0.1#53(127.0.0.1)
    ;; WHEN: Sun Sep 13 13:51:17 CEST 2020
    ;; MSG SIZE  rcvd: 56

Czas zapytania już nie wynosi 205 ms ale 1ms, więc odpowiedź pochodzi z cache. Jak możemy również
zauważyć, czas TTL wynosi teraz 596s. Zatem ten TTL został przez dnsmasq przepisany do wartości
określonej w `min-cache-ttl` (600). Każdy wpis mający TTL niższy niż 600 sekund będzie miał
ustawiony tę wartość przy dodawaniu rekordu do cache DNS. Natomiast każda odpowiedź, która ma TTL
wyższy niż zostało to określone w `max-cache-ttl` , również zostanie przepisana, z tym, że do
wartości 3600. Jeśli natomiast trafi się wpis mający TTL z tego przedziału 600-3600, np. 1200, to
taka  wartość TTL zostanie zapisana w cache DNS.

### Statystyki cache w dnsmasq

Statystyki całego cache możemy wyciągnąć z logu systemowego, tylko pierw trzeba wysłać odpowiedni
sygnał do demona dnsmasq ( `USR1` ):

    # kill -s USR1 $(pidof dnsmasq)

Po wydaniu tego powyższego polecenia, w logu systemowym powinniśmy zarejestrować poniższy komunikat:

    dnsmasq[614751]: cache size 4096, 0/3599 cache insertions re-used unexpired cache entries.
    dnsmasq[614751]: queries forwarded 13596, queries answered locally 10157
    dnsmasq[614751]: queries for authoritative zones 0
    dnsmasq[614751]: pool memory in use 48, max 48, allocated 2400
    dnsmasq[614751]: server 192.168.1.1#53: queries sent 0, retried or failed 0
    dnsmasq[614751]: server 127.0.2.1#53: queries sent 13596, retried or failed 28

Jak czytać te powyższe statystyki? W pierwszej linijce mamy rozmiar cache `4096` . Dalej mamy
`0/3599` i zgodnie z tym co wyczytałem na [liście mailingowej dnsmasq][18], to `0` oznacza brak
usuniętych wpisów z cache zanim utraciły one ważność, natomiast `3599` oznacza ilość wpisów w cache
ogółem, zatem w cache jest jeszcze trochę wolnego miejsca. Te dwie wartości mówią nam czy
ustawienia rozmiaru cache (czy też czasu ważności wpisów) należy dostosować. W przypadku, gdy
pierwsza wartość zacznie się zwiększać, trzeba pomyśleć nad zmianą parametrów cache. Dalej w logu
nie ma już nic niezwykłego. W linijkach z `server` są zapytania, które zostały przesłane do
skonfigurowanych serwerów DNS. W tym przypadku `192.168.1.1` to adres serwera DNS domowego routera
WiFi, a na `127.0.2.1` nasłuchuje lokalnie dnscrypt-proxy. Praktycznie wszystkie zapytania lecą
właśnie do niego. W sumie system wysłał `23753` zapytań o domeny, z czego `13596` poleciało do
dnscrypt-proxy (cache miss) i dalej po zaszyfrowaniu do upstream'owego serwera DNS, zaś `10157`
zapytań zostało wzięte z cache dnsmasq (cache hit, ~42%) i nie było potrzeby odpytywać zdalnego
serwera DNS. Kilka zapytań się nie powiodło ale ten fakt nie jest niczym niezwykłym. Grunt by tych
błędów nie było dużo w stosunku do ilości wykonywanych zapytań DNS. Jeśli zaś chodzi o cache hit,
to jest on trochę mały i by go poprawić można ździebko podnieść minimalny jak i maksymalny TTL.

Statystyki cache możemy także uzyskać via `dig` :

    # dig +short chaos txt cachesize.bind insertions.bind evictions.bind misses.bind hits.bind auth.bind servers.bind

    "4096"
    "296"
    "0"
    "1021"
    "646"
    "0"
    "192.168.1.1#53 0 0" "127.0.2.1#53 1021 0"

## Test szyfrowanych zapytań DNS

To w zasadzie cała konfiguracja dnscrypt-proxy i dnsmasq, która trzeba było poczynić, by te
zapytania DNS mogły w końcu wędrować po kablach w formie zaszyfrowanej. Przydałoby się jednak
przetestować czy aby na pewno tak się dzieje. Szereg provider'ów DNS zapewnia stosowne usługi by
zweryfikować szyfrowany DNS, np. [CloudFlare ma tę stronę][33]. My jednak nie będziemy się opierać
na zewnętrznych stronach WWW i sprawdzimy ręcznie czy te zapytania DNS są faktycznie szyfrowane, a
posłuży nam do tego celu `wireshark` .

Musimy odpalić dwie instancję tego sniffer'a pakietów. Jedna z nich będzie monitorować interfejs
`lo` , a druga interfejs, którym pakiety wychodzą w świat z naszej maszyny, w tym przypadku jest to
`bond0` . Mając odpalonego wireshark'a, zapuszczamy proste polecenie `ping` na jakąś domenę, której
standardowo nie odwiedzamy. Chodzi o to by uniknąć ewentualnego trafienia w cache DNS, bo w takim
przypadku zapytanie DNS naturalnie nie zostanie rozwiązane, tylko odpowiedź zostanie wzięta z tego
cache.

Na poniższym obrazku mamy widoczny podgląd portu 53 protokołu TCP/UDP interfejsu `lo` :

![encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark](/img/2020/09/001-encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark.png#huge)

Jak widzimy, na interfejsie lokalnym ruch DNS odbywa się w postaci odszyfrowanej. Pierwsze dwa
pakiety (te ze źródłowym i docelowym adresem `127.0.0.1` ) to zapytania linux'owego resolver'a do
dnsmasq o adresy IPv4 (rekord `A` ) i IPv6 (rekord `AAAA` ).  Następnie dnsmasq przekazuje te
zapytania na adres `127.0.2.1` do dnscrypt-proxy (kolejne dwa pakiety). Dalej mamy odpowiedzi na
wysłane zapytania (kolejne cztery pakiety). Odpowiedzi przychodzą w odwrotnej kolejności, tj. pierw
dnscrypt-proxy przesyła je do dnsmasq, a na końcu dnsmasq do linux'owego resolver'a. Zatem dnsmasq
i dnscrypt-proxy rozmawiają ze sobą i przekazują między sobą zarówno zapytania jak i odpowiedzi. No
ale co się dzieje z tymi zapytaniami DNS, gdy docierają do interfejsu `bond0` ? Podejrzyjmy zatem
port 53 na interfejsie `bond0` :

![encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark](/img/2020/09/002-encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark.png#huge)

Brak jakiegokolwiek ruchu, zatem na interfejsie wychodzącym z naszej maszyny nie widać żadnych
zapytań DNS.

To co się właściwie dzieje z tymi zapytaniami DNS? Są one przesyłane do serwera DNS CloudFlare na
port 443 (jako że w opisanej wyżej konfiguracji korzystamy z DoH). Zobaczmy zatem co się dzieje na
interfejsie `bond0` na porcie 443:

![encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark](/img/2020/09/003-encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark.png#huge)

Jak widzimy, nasza maszyna rozmawia z serwerem `1.0.0.1` przy wykorzystaniu protokołu TLS 1.3.
Wiemy zatem, że nasze zapytania DNS są przesyłane przez sieć w formie zaszyfrowanej i nikt
nieuprawniony nie jest w stanie ich podejrzeć.

Jeśli się przyjrzymy uważnie, to zapytania `AAAA` (o adres IPv6) są blokowane, a właściwie
odpowiada na nie dnscrypt-proxy komunikatami `CPU: AAAA queries have been locally blocked by
dnscrypt-proxy` oraz `OS: Set block_ipv6 to false to disable this feature` :

![encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark](/img/2020/09/004-encrypted-dns-query-dnsmasq-dnscrypt-proxy-linux-debian-wireshark.png#huge)

Czyli wszystko działa prawidłowo.

## Szyfrowany DNS sam w sobie nie wystarczy

Szyfrowany DNS co do zasady uniemożliwia podejrzenie pakietów i podsłuchanie ruchu DNS na linii
klient-serwer. Niemniej jednak, trzeba także zatroszczyć się o zaszyfrowanie pozostałego ruchu
przesyłanego do i odbieranego z serwera. Zwykle w przypadku stron WWW wykorzystywany jest protokół
SSL/TLS. Niemniej jednak, by serwer był w stanie serwować wiele stron WWW z wykorzystaniem tego
bezpiecznego protokółu i szyfrować ruch niezależnie od skonfigurowanych na nim domen, to musi robić
użytek z rozszerzenia protokołu TLS zwanego SNI ([Server Name Indication][9]). Problem z tym całym
SNI jest taki, że nazwa odwiedzanej przez nas domeny jest przesyłana do serwera otwartym tekstem,
przez co nasz ISP (czy też inne podmioty) są w stanie bez najmniejszego problemu ustalić strony,
które odwiedzamy i ocenzurować je nawet jeśli szyfrujemy ruch DNS. Póki co, przed tym wyciekiem tak
kluczowej informacji jak nazwa odwiedzanej domeny nie ma ochrony. Niemniej jednak [Mozilla oraz
CloudFlare działają][10] na rzecz zmiany tego stanu rzeczy przez wdrażanie ESNI
([Encrypted SNI][11]). Ten proces zajmie jeszcze pewnie lata zanim wyciek informacji za sprawą SNI
zostanie załatany ale już w tej chwili można [ręcznie w przeglądarce Firefox włączyć szyfrowany
SNI][12]. Trzeba jednak mieć na uwadze fakt, że to czy nazwa domeny zostanie zaszyfrowana zależy
głównie od samego serwera WWW, na którym hostowana jest odwiedzana przez nas strona. Jeśli serwer
nie wspiera ESNI, to w dalszym ciągu nazwa domeny zostanie przesłana przez sieć otwartym tekstem i
szyfrowany DNS w zasadzie na nic nam się zda, przynajmniej w kwestii ochrony naszej prywatności.

## Czy chattr +i na /etc/resolv.conf to dobry pomysł

Wielu użytkowników linux'a w obawie przed przepisaniem przez system pliku `/etc/resolv.conf`
ustawia temu plikowi bit (atrybut) odporności, tj. Immutable Bit, przy pomocy `chattr` . To
rozwiązanie ma na celu uniemożliwienie wszystkim użytkownikom w systemie zmiany pliku
`/etc/resolv.conf` , przez co nawet administrator root nie będzie miał prawa tknąć tego pliku.

Z bitu odporności powinno się korzystać jedynie w przypadku, gdy wiemy, że żadne części systemu nie
będą próbować przepisać zawartości tego pliku w sposób nieuprawniony. Niemniej jednak, cała masa
użytkowników wykorzystuje bit odporności, by uniemożliwić systemowi jego prawidłową pracę, w
efekcie czego pewne usługi mogą generować błędy, przykładowo w katalogu `/etc/` może zacząć
pojawiać się cała masa plików pokroju `resolv.conf.dhclient-new.xxxx` . Te pliki są tworzone przez
`dhclient` (standardowy klient DHCP na Debianie), który w lease DHCP może otrzymać od routera
domowego (czy ISP) adres serwera DNS. Później ten adres jest umieszczany w pliku `/etc/resolv.conf`
ale, że system nie może zapisać tego pliku (za sprawą ustawienia mu bitu odporności), to generowany
jest błąd i te tymczasowe pliki nie są usuwane, bo proces nie został dokończony.

Zatem mamy sytuację, gdzie w uprawniony sposób klient DHCP chce przepisać plik `/etc/resolv.conf` i
zamiast w tym miejscu ustawiać bit odporności, powinniśmy tak skonfigurować klienta DHCP, by nie
ruszał pliku `/etc/resolv.conf` . W przypadku `dhclient` możemy napisać prosty skrypt o nazwie
`no-resolv` , który trzeba umieścić w katalogu `/etc/dhcp/dhclient-enter-hooks.d/` . Poniżej jest
treść tego skryptu:

    #/bin/sh

    # see /sbin/dhclient-script

    RUN="yes"

    if [ "$RUN" = "yes" ]; then
        if [ "${interface}" = "wwan0" ] ||
           [ "${interface}" = "bond0" ] ||
           [ "${interface}" = "usb0" ]  ||
           [ "${interface}" = "eth0" ]  ||
           [ "${interface}" = "wlan0" ]; then
            case $reason in
            BOUND|RENEW|REBIND|REBOOT)
                make_resolv_conf() {
                    return 0
                }
            ;;
            esac
        fi
    fi

W skrócie, `dhclient` standardowo wywołuje funkcję `make_resolv_conf() {}` , która ma za zadanie
przepisać plik `/etc/resolv.conf` podczas pobierania lease DHCP. Gdy `dhclient` będzie pobierał
lease DHCP dla jednego z uwzględnionych wyżej interfejsów, to zamiast korzystać ze standardowej
funkcji `make_resolv_conf() {}` będzie korzystał z tej naszej funkcji, która w zasadzie nie robi
nic. W ten sposób plik `/etc/resolv.conf` nie zostanie ruszony przez `dhclient` , co otwiera nam
drogę do skorzystania z bitu odporności.

Oczywiście w systemie może być więcej usług, które operują na tym pliku `/etc/resolv.conf` i trzeba
je wszystkie wyłapać tak, by te usługi odpowiednio skonfigurować. W tym przypadku tylko `dhclient`
próbował przepisywać ten plik. Po zastosowaniu tego powyższego skryptu, żadna inna aplikacja w
sposób uprawniony nie powinna pliku `/etc/resolv.conf` ruszać i by mieć pewność, że nic przez
przypadek (bez naszej świadomej i dobrowolnej zgody) nie ruszy tego pliku, możemy mu nałożyć bit
odporności. By ustawić bit odporności, w terminalu trzeba wpisać to poniższe polecenie:

    # chattr +i /etc/resolv.conf

Przy pomocy `lsattr` możemy dodatkowo zweryfikować czy bit został poprawnie nałożony:

    # lsattr /etc/resolv.conf
    ----i---------e----- /etc/resolv.conf

Tam gdzie są `-` , to dany atrybut nie jest ustawiony. W tym powyższym przypadku są ustawione dwa
atrybuty, tj. `i` oraz `e` . Atrybut odporności `i` sami ustawiliśmy przed chwilą, zaś `e` wskazuje,
że plik wykorzystuje zakresy ([extents w EXT4][14]) do mapowania bloków na dysku. Więcej informacji
o atrybutach można znaleźć w [man chattr][13].

## Czy szyfrować zapytania o domeny serwerów czasu

Niekiedy na necie można się spotkać z dodawaniem do konfiguracji dnsmasq osobnego wpisu dla domen
serwerów czasu w postaci `server=/pool.ntp.org/1.1.1.1` . W takim przypadku, wszystkie zapytania o
tę domenę (i jej wszystkie subdomeny) będą realizowane z pominięciem szyfrowania. Pojawia się tutaj
jednak dylemat czy te zapytania powinny być przesyłane przez sieć nieszyfrowane czy wręcz odwrotnie.
Z jednej strony nie po to chcemy zaszyfrować wszystkie zapytania DNS, by część z nich puszczać w
formie niezabezpieczonej. Z drugiej jednak strony, szyfrowanie zapytań DNS dodaje zwykle niedające
się do końca przewidzieć opóźnienia, bo system musi przecież poświęcić trochę dodatkowego czasu,
by dane trafiające do pakietów sieciowych zaszyfrować. Takie dodatkowe opóźnienia są bardzo
niepożądane z perspektywy synchronizacji czasu, a im bardziej procesor naszego komputera jest
obciążony, tym proces szyfrowania danych zajmuje więcej czasu. Niemniej jednak, rozwiązywanie nazw
domen na adresy IP dokonywane jest przed rozpoczęciem procesu synchronizacji czasu i nie wpływa na
niego w żaden sposób. Dlatego też unikajmy dodawania podobnych wpisów w konfiguracji dnsmasq.

## Podsumowanie

Sposobów na zabezpieczenie usługi DNS rozwiązującej nazwy domen na adresy IP jest co najmniej kilka.
W tym artykule został przedstawiony mechanizm, który szyfruje zapytania DNS przy wykorzystaniu
dnscrypt-proxy i dnsmasq, w efekcie czego zapytania DNS przestają być czytelne dla całej masy
wścibskich podmiotów zagrażających naszej prywatności i bezpieczeństwu w sieci. Niemniej jednak,
szyfrowanie ruchu DNS samo w sobie nie jest rozwiązaniem, które może nas uchronić przez masową
inwigilacją. Wymagane zatem jest stosowanie dodatkowych zabezpieczeń, np. przeglądanie stron WWW z
wykorzystaniem bezpiecznego protokołu HTTPS. Trzeba sobie jednak zdawać sprawę, że obecnie nie ma
sposobu by ukryć nazwę domeny odwiedzanego przez nas serwisu z winy SNI, przez co ta nazwa jest
przesyłana przez sieć w postaci niezabezpieczonej. Jeśli wdrażamy szyfrowany DNS głównie po to, by
uniemożliwić osobom trzecim ustalenie jakie strony WWW odwiedzamy, to lepszym rozwiązaniem będzie
postawienie i [skonfigurowanie serwera VPN][34]. W takim przypadku jedyną informacją, którą lokalny
ISP będzie w stanie pozyskać, będzie adres VPN. Wszystkie pozostałe dane (czy to DNS, czy też nazwa
domeny w SNI) będą przez tę jednostkę przechodzić w formie zaszyfrowanej. Niemniej jednak, takie
rozwiązanie jest nas w stanie ochronić jedynie przed lokalną cenzurą, bo w dalszym ciągu ten SNI z
VPN do serwera WWW będzie przesyłany w formie niezaszyfrowanej. Tak czy inaczej, ESNI (szyfrowany
SNI) być może uda się wdrożyć w niedalekiej przyszłości i wtedy szyfrowanie ruchu DNS (by poprawić
naszą prywatność) będzie miało nieco więcej sensu.


[1]: https://forum.dug.net.pl/viewtopic.php?id=31524
[2]: https://dnscrypt.info/
[3]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[4]: /tags/resolver/
[5]: /post/konfiguracja-interfejsow-bond-bonding/
[6]: https://github.com/systemd/systemd/issues/15257
[7]: https://www.freedesktop.org/software/systemd/man/systemd-networkd.service.html
[8]: https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html
[9]: https://en.wikipedia.org/wiki/Server_Name_Indication
[10]: https://blog.cloudflare.com/encrypt-that-sni-firefox-edition/
[11]: https://en.wikipedia.org/wiki/Server_Name_Indication#Encrypted_Client_Hello
[12]: /post/jak-wlaczyc-w-firefox-esni-encrypted-sni/
[13]: https://man7.org/linux/man-pages/man1/chattr.1.html
[14]: https://ext4.wiki.kernel.org/index.php/Ext4_Design#Ext4_Extents
[15]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[16]: https://en.wikipedia.org/wiki/DNS_over_TLS
[17]: https://dnscrypt.info/protocol/
[18]: http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2013q2/007331.html
[19]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=958045
[20]: https://en.wikipedia.org/wiki/DNS_rebinding
[21]: http://0pointer.de/blog/projects/socket-activation.html
[22]: https://www.iana.org/assignments/locally-served-dns-zones/locally-served-dns-zones.xhtml
[23]: https://github.com/DNSCrypt/dnscrypt-proxy/wiki/DNS-server-sources
[24]: https://tools.ietf.org/html/rfc5077
[25]: https://golang.org/src/crypto/tls/cipher_suites.go
[26]: /post/jak-wlaczyc-w-firefox-esni-encrypted-sni/
[27]: https://dnscrypt.info/public-servers/
[28]: https://en.wikipedia.org/wiki/Reverse_DNS_lookup
[29]: https://en.wikipedia.org/wiki/Name_Service_Switch
[30]: https://en.wikipedia.org/wiki/Init#SysV-style
[31]: https://www.freedesktop.org/wiki/Software/systemd/NetworkTarget/
[32]: https://developers.cloudflare.com/1.1.1.1/setting-up-1.1.1.1
[33]: https://www.cloudflare.com/ssl/encrypted-sni/
[34]: /tags/vpn/
