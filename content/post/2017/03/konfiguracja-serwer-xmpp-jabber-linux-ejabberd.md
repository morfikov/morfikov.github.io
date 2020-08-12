---
author: Morfik
categories:
- Linux
date: "2017-03-20T20:26:15Z"
date_gmt: 2017-03-20 19:26:15 +0100
published: true
status: publish
tags:
- debian
- xmpp/jabber
title: Konfiguracja serwera XMPP/Jabber pod linux (ejabberd)
---

Użytkownicy internetu mają całą masę różnych sposobów na komunikację miedzy sobą. Kiedyś wszyscy
korzystali z komunikatorów typu Gadu-Gadu. Ja byłem jedyną osobą, która od samego początku wolała
alternatywne rozwiązania i jechałem na komunikatorze Tlen (ten od O2), a w niedługim czasie
przesiadłem się na Jabber'a i tak z niego korzystam do dziś. W zasadzie GG i Tlen są obecnie już
chyba na wymarciu, bo większość ludzi (jak nie wszyscy) przerzuciła się na Facebook'a czy Twitter'a.
Niemniej jednak, pisanie o sprawach prywatnych w tych serwisach nie jest najlepszym rozwiązaniem.
Jeśli chcemy zadbać o poufność przesyłanych przez internet komunikatów, to trzeba to robić na inne
sposoby. Jednym z nich jest właśnie korzystanie z [protokołu XMPP/Jabber](https://xmpp.org/). To co
odróżnia Jabber'a od innych technologii na rynku, to fakt zdecentralizowania sieci, czyli mamy całą
masę serwerów Jabber'a, na których możemy sobie stworzyć konto. Uwalenie jednego serwera nie wpływa
na działanie pozostałych. Google także wykorzystuje protokół XMPP/Jabber i mając konto na gmail'u,
mamy również stosowny JID w postaci adresu email, który możemy sobie wklepać do jednego z klientów
Jabber'a, np. [PSI czy Gajim](https://xmpp.org/software/clients.html), i jesteśmy już w stanie
rozmawiać z osobami, które mają konta na innych serwerach. Właśnie, inne serwery, a może by tak
sobie postawić własny serwer Jabber'a? Tak się składa, że w repozytorium Debiana znajduje się
[oprogramowanie ejabberd](https://www.ejabberd.im/), które jest nam w stanie umożliwić realizację
tego przedsięwzięcia.

<!--more-->
## Zalety posiadania własnego serwera Jabber'a

Protokół XMPP/Jabber jest protokołem otwartym i darmowym, tzn. każdy z nas może z niego korzystać
bez potrzeby kupowania jakichś licencji czy wnoszenia opłat. Komunikacja w tym protokole może być
szyfrowana na linii klient-serwer lub/i serwer-serwer. To czy tak w istocie, zależy od kilku
czynników, min. od konfiguracji serwera na którym mamy konto, a mając własny serwer mamy też pełnię
władzy nad jego konfiguracją.

Komunikacja klientów z serwerem może odbywać się w zasadzie na dwóch portach. Standardowo jest to
port `5222` . We wcześniejszych wersjach oprogramowania Jabber'a był używany port `5223` ale obecnie
jest on uważany za przestarzały i się go już nie stosuje, przynajmniej nie jest to zalecane. Różnica
między tymi dwoma portami tkwi głównie w nawiązywaniu szyfrowanego połączenia między klientem a
serwerem. W przypadku portu `5222` ruch może nie być w ogóle szyfrowany lub też wykorzystywana jest
[metoda STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS) (zwykle ta opcja jest spotykana).
Natomiast jeśli chodzi o port `5223` , to tutaj używany jest klasyczny SSL/TLS, z którym mamy do
czynienia przeglądając, np. strony WWW po HTTPS.

Nie wszystkie serwery Jabber'a równają do najwyższych standardów bezpieczeństwa. Część z serwerów
może obsługiwać jedynie słabe szyfry, np. SSLv2 czy SSLv3, których stosowanie zagraża
bezpieczeństwu serwera i prywatności jego użytkowników. Nawet jeśli serwer wymusza od swoich
klientów negocjowanie szyfrów TLSv1.2, to i tak nie daje to 100% odporności na próby przechwycenia
komunikacji.

Gdy klienci mają konto na tym samym serwerze, to ruch między nimi zawsze będzie szyfrowany i tutaj
decyduje tylko i wyłącznie konfiguracja tego konkretnego serwera. W przypadku, gdy konta są na
różnych serwerach, to wiadomości między serwerami mogą podróżować w formie niezaszyfrowanej.
Dzieje się tak dlatego, że serwery muszą między sobą wynegocjować parametry dla połączenia. Jeśli
jeden z nich nie wspiera szyfrowania, lub oferuje jedynie słabe szyfry, to mamy do wyboru albo
przystać na takie warunki i stosować te słabe szyfry (lub w ogóle ich nie stosować), albo też
serwery te nie nawiążą połączenia i komunikacja nie będzie w ogóle możliwa.

Przykładem takiego serwera, który nie chce współpracować z wymuszonym szyfrowaniem jest ten od
Google (adresy GMAIL). W przypadku tego typu serwerów możemy naturalnie wykorzystać
[OTR](https://pl.wikipedia.org/wiki/Off-the-record_messaging) (Off-The-Record messaging) czy [klucze
szyfrujące PGP/GPG](https://pl.wikipedia.org/wiki/GNU_Privacy_Guard), by we własnym zakresie
zabezpieczyć komunikację. Problem w tym, że jest to mało wygodne. Może i OTR dość znacznie ułatwia
cały proces ale i tak wymaga od użytkownika przeprowadzenia kilku dodatkowych czynności. A w
przypadku posiadania własnego serwera Jabber'a (z wymuszonym TLSv1.2) w zasadzie nie trzeba robić
nic więcej, by ruch zaszyfrować, przynajmniej w przypadku użytkowników mających konto na naszym
serwerze. Oczywiście w dalszym ciągu możemy wykorzystać OTR czy klucze PGP/GPG jako dodatkową
warstwę zabezpieczającą.

## Instalacja i konfiguracja serwera ejabberd

W dystrybucji Debian jest dostępny pakiet `ejabberd` oraz również kilka pakietów z różnymi modułami
`ejabberd-mod-*` . W stabilnym wydaniu, te pakiety są w dość leciwej wersji (14.07). Dlatego też
lepiej [dodać sobie repozytorium Debian backports](https://backports.debian.org/Instructions/) i to
z niego zainstalować stosowne pakiety, bo tutaj mamy wersję 16.09 .

Po zainstalowaniu pakietu `ejabberd` , w katalogu `/etc/ejabberd/` zostanie utworzony plik
`ejabberd.yml` . W tym pliku znajduje się konfiguracja naszego serwera Jabber'a. Niemniej jednak,
zanim przejdziemy do edycji tego pliku, musimy zrobić kilka innych rzeczy.

### Subdomena

Morfitronik jest hostowany na OVH i jego domena to `morfitronik.pl` . Serwer Jabber'a będzie zaś
podpięty pod subdomenę `jabber.morfitronik.pl` . Tę subdomenę trzeba pierw stworzyć, tj. dodać
stosowny rekord A/AAAA w konfiguracji DNS. Można się spotkać z informacją, że zamiast tworzyć osobny
rekord A/AAAA dla subdomeny, lepiej jest utworzyć rekord CNAME i tam określić docelowy adres. Z tego
[co znalazłem na sieci](https://prosody.im/doc/dns) wynika, że późniejsze wskazanie w rekordzie SRV
docelowego adresu z CNAME będzie powodować problemy i tak jest w przypadku
OVH:

[![]({{< baseurl >}}/img/2017/03/000-0.jabber-ejabberd-serwer-debian-linux-cname-problem-660x101.png)]({{< baseurl >}}/img/2017/03/000-0.jabber-ejabberd-serwer-debian-linux-cname-problem.png)

Dlatego też ja stworzyłem sobie osobny rekord A/AAAA zamiast CNAME.

Zarządzanie rekordami DNS może wyglądać inaczej u różnych DNS provider'ów. Niemniej jednak, dla
przykładu opiszę jak to wygląda w przypadku panelu webowego OVH. Udajmy się zatem do panelu
administracyjnego i przejdźmy na zakładkę `WWW`
.

![]({{< baseurl >}}/img/2017/03/000-1.jabber-ejabberd-serwer-debian-linux-ovh-domena.png)

Po wybraniu domeny, pojawią nam się informacje ogólne dotyczące konfiguracji. Mamy tam też min.
zakładkę `Strefa DNS` . Po przejściu na nią, po prawej stronie będziemy mieć poniższe
przyciski:

[![]({{< baseurl >}}/img/2017/03/001.jabber-ejabberd-serwer-debian-linux-dns-przyciski-660x216.png)]({{< baseurl >}}/img/2017/03/001.jabber-ejabberd-serwer-debian-linux-dns-przyciski.png)

Klikamy naturalnie w `Dodaj wpis` , a następnie wybieramy rodzaj pola DNS i tu wskazujemy `A` dla
IPv4 lub/i `AAAA` dla
IPv6:

![]({{< baseurl >}}/img/2017/03/002.jabber-ejabberd-serwer-debian-linux-dns-rekord-a.png)

Określamy subdomenę na `jabber` oraz wskazujemy adres
docelowy:

![]({{< baseurl >}}/img/2017/03/003.jabber-ejabberd-serwer-debian-linux-dns-rekord-subdomena.png)

Akceptujemy i powinniśmy ujrzeć poniższe
podsumowanie:

![]({{< baseurl >}}/img/2017/03/004.jabber-ejabberd-serwer-debian-linux-dns-rekord-subdomena.png)

Niżej jest jeszcze tekstowa wersja wpisu, gdyby ktoś chciał ręcznie go dodać w konfiguracji DNS:

    jabber.morfitronik.pl IN A  151.80.57.162

### Rekord SRV

Mamy zatem skonfigurowaną subdomenę. Teraz musimy jeszcze dodać rekordy SRV. Rekord SRV umożliwia
transparentne przekierowanie zapytań na inną domenę lub port, czyli tam, gdzie znajduje się usługa.
W naszym przypadku będzie to subdomena `jabber.morfitronik.pl` .

W przypadku protokołu XMPP/Jabber [jest kilka typów rekordów
SRV](https://wiki.xmpp.org/web/SRV_Records). Nas interesują client-to-server (c2s) oraz
server-to-server (s2s). W ten sposób klienci używają rekordu c2s, natomiast inne serwery używają
rekordu s2s i wiedzą na jakie porty słać zapytania. Standardowe porty dla protokołu XMPP/Jabber to:
5222 (XMPP client connection (RFC 6120)) oraz 5269 (XMPP server connection (RFC 6120)). To właśnie
te dwa porty musimy podać w konfiguracji DNS wskazując jednocześnie subdomenę
`jabber.morfitronik.pl` .

Ponownie odwiedzamy panel OVH i dodajemy kolejny wpis. Tym razem wybieramy jedno z pól
rozszerzonych, tj. `SRV`
:

![]({{< baseurl >}}/img/2017/03/006.jabber-ejabberd-serwer-debian-linux-dns-rekord-srv.png)

I uzupełniamy tak jak to widać na poniższym
obrazku:

![]({{< baseurl >}}/img/2017/03/007.jabber-ejabberd-serwer-debian-linux-dns-rekord-srv-konfiguracja.png)

![]({{< baseurl >}}/img/2017/03/008.jabber-ejabberd-serwer-debian-linux-dns-rekord-srv-konfiguracja.png)

Podobnie postępujemy dla drugiego
rekordu:

![]({{< baseurl >}}/img/2017/03/009.jabber-ejabberd-serwer-debian-linux-dns-rekord-srv-konfiguracja.png)

![]({{< baseurl >}}/img/2017/03/010.jabber-ejabberd-serwer-debian-linux-dns-rekord-srv-konfiguracja.png)

Poniżej jeszcze dodatkowo tekstowa wersja tych dwóch rekordów na wypadek, gdyby ktoś chciał ręcznie
edytować konfigurację DNS:

    _xmpp-client._tcp.morfitronik.pl. 18000 IN SRV    0 5 5222 jabber.morfitronik.pl.
    _xmpp-server._tcp.morfitronik.pl. 18000 IN SRV    0 5 5269 jabber.morfitronik.pl.

Starsze serwery Jabber'a mogą korzystać z rekordu `_jabber._tcp` zamiast `_xmpp-server._tcp` i można
również dodać ten rekord SRV:

    _jabber._tcp.morfitronik.pl. 18000 IN SRV    0 5 5269 jabber.morfitronik.pl.

### Certyfikat Let's Encrypt

Mając skonfigurowaną domenę oraz wiedząc, że ruch w protokole XMPP/Jabber powinien być szyfrowany,
musimy postarać się o odpowiednie certyfikaty dla naszego serwera. Te z kolei można wygenerować
sobie ręcznie, np. [przy pomocy pakietu
easy-rsa]({{< baseurl >}}/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/), ale też można
skorzystać z [gotowego rozwiązania oferowanego przez Let's Encrypt](https://letsencrypt.org/) i my
właśnie z niego skorzystamy.

Na Morfitroniku znajduje się już kilka usług, które zapewniają szyfrowane połączenie. Jest tutaj
zainstalowany min. serwer www. Tak się składa, że [Let's Encrypt dostarcza już certyfikat dla tego
bloga]({{< baseurl >}}/post/certyfikat-letsencrypt-dla-bloga-certbot/), a konkretnie dla domeny
`morfitronik.pl` . Nie da rady wygenerować certyfikatu osobno dla drugiej domeny i trzeba ten proces
przeprowadzić dla obu domen jednocześnie. Robimy to w poniższy sposób:

    # certbot certonly \
        -d morfitronik.pl,www.morfitronik.pl,jabber.morfitronik.pl \
        --standalone

[![]({{< baseurl >}}/img/2017/03/011.jabber-ejabberd-serwer-debian-linux-certyfikat-letsencrypt-660x212.png)]({{< baseurl >}}/img/2017/03/011.jabber-ejabberd-serwer-debian-linux-certyfikat-letsencrypt.png)

![]({{< baseurl >}}/img/2017/03/012.jabber-ejabberd-serwer-debian-linux-certyfikat-letsencrypt.png)

Podczas generowania certyfikatu, trzeba tymczasowo wyłączyć serwer www jeśli takowy posiadamy. Warto
też wspomnieć, że ta subdomena standardowo będzie wskazywać na główny katalog serwisu. W końcu
serwer DNS odsyła zapytania pod ten sam adres IP po wywołaniu zarówno domeny jak i subdomeny. Trzeba
będzie zatem stworzyć nowego wirtualnego hosta w konfiguracji Apache2 (czy innego serwera www) i
rozdzielić w ten sposób zapytania.

Let's Encrypt przechowuje certyfikaty i klucze w osobnych plikach. Na potrzeby `ejabberd` trzeba dwa
z tych plików połączyć w jeden:

    # cd /etc/letsencrypt/live/morfitronik.pl/
    # cat privkey.pem fullchain.pem > /etc/ejabberd/ejabberd.pem

Generujemy jeszcze plik `dh4096.pem` :

    # openssl dhparam 4096 > /etc/ejabberd/dh4096.pem

Upewnijmy się przy tym, że te powyższe pliki mają odpowiednie uprawnienia:

    # chown root:ejabberd /etc/ejabberd/dh4096.pem
    # chown root:ejabberd /etc/ejabberd/ejabberd.pem
    # chmod 440 /etc/ejabberd/dh4096.pem
    # chmod 640 /etc/ejabberd/ejabberd.pem

Certyfikaty Let's Encrypt są odnawiane co trzy miesiące. Przydałoby się zatem co miesiąc łączyć
pliki `privkey.pem` i `fullchain.pem` i resetować serwer Jabber'a. Jeśli tego nie uczynimy, to w
końcu nasz serwer będzie działał bez ważnego certyfikatu. To zadanie można zautomatyzować tworząc
sobie prosty skrypt i wywołując go cyklicznie przy pomocy cron'a. Poniżej jest taki skrypt:

    #!/bin/bash

    cert_dir="/etc/letsencrypt/live/morfitronik.pl"

    systemctl stop ejabberd.service
    #systemctl stop epmd.socket
    systemctl stop epmd.service

    cat $cert_dir/privkey.pem $cert_dir/fullchain.pem > /etc/ejabberd/ejabberd.pem

    chown root:ejabberd /etc/ejabberd/ejabberd.pem
    chmod 640 /etc/ejabberd/ejabberd.pem

    systemctl start ejabberd.service

A tak wygląda zadanie dla cron'a:

    27  02  1   *   *   /skrypty/ejabberd-cert.sh

### Firewall i reguły iptables

Wiemy, że jabber będzie nasłuchiwał dwóch rodzajów połączeń na portach 5222 oraz 5269 . Są to
standardowe porty i jest niemal pewne, ze jakieś boty się do naszego serwera przyczepią. Dlatego
musimy się postarać o zabezpieczenie tej usługi przy pomocy filtra pakietów `iptables` . W zasadzie
musimy zrobić trzy rzeczy. Pierwszą z nich jest akceptowanie połączeń w stanie NEW na te ww. porty.
Drugą jest limitowanie ilości jednoczesnych połączeń dla konkretnego adresu IP. A trzecią rzeczą
jest zaimplementowanie czarnej listy adresów, która uniemożliwi pewnym osobom/botom na połączenie z
serwerem.

#### Czarna lista adresów IP

Może zacznijmy od ostatniej kwestii czyli czarnej listy adresów. Nie musimy jej budować od podstaw,
bo stosowne listy są już utrzymywane i regularnie aktualizowane. My musimy tylko te listy jakoś
wrzucić na zaporę sieciową naszego serwera. Najprościej do tego celu wykorzystać
[ipset](http://ipset.netfilter.org/). W dystrybucji Debian, to narzędzie jest dostępne w pakiecie
`ipset` i wystarczy je zainstalować.

List z adresami można poszukać na Google ale ktoś już tę pracę wykonał za nas i stworzył prosty
skrypt, który jest w stanie bardzo łatwo dodać całe zakresy adresów IP i filtrować w oparciu o nie
ruch sieciowy docierający do naszego serwera. Projekt, o którym mowa, [znajduje się
tutaj](https://github.com/trick77/ipset-blacklist). W tym linku mamy dokładną instrukcję obsługi
tego narzędzia i raczej nie powinno być problemów z zaimplementowaniem go w systemie. Można
oczywiście sobie dostosować pewne rzeczy, np. pozycję reguły w filtrze przenosząc ją z łańcucha
INPUT w tablicy FILTER do łańcucha PREROUTING w tablicy RAW, przez co żądania połączeń nie będą
przechodzić przez sporą część samego filtra.

#### Limitowanie połączeń

Może i czarna lista adresów została załadowana do `iptables` ale nie damy rady zbanować nią
wszystkich złych użytkowników internetu. Zawsze znajdzie się ktoś, kto będzie próbował nasz serwer
zaatakować i uzyskać do niego nieuprawniony dostęp czy wykorzystać jego zasoby w sposób
nieodpowiedni. Dlatego też drugim zabezpieczeniem jakie sobie zaimplementujemy będzie ograniczenie
ilości połączeń jakie pojedynczy klient będzie w stanie nawiązać. Do tego celu posłużą nam dwa
moduły: [connlimit](http://ipset.netfilter.org/iptables-extensions.man.html#lbAM) oraz
[hashlimit](http://ipset.netfilter.org/iptables-extensions.man.html#lbAY).

Przekierujmy sobie zatem wszystkie nowe połączenia w protokole TCP do łańcucha `tcp` :

    iptables -N tcp

    iptables -A INPUT -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -m comment --comment "Connections not started by SYN" -j DROP
    iptables -A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j tcp

Teraz w łańcuchu `tcp` tworzymy te dwie poniższe reguły:

    iptables -A tcp -p tcp -m tcp --dport 5222 \
        -m connlimit --connlimit-upto 20 --connlimit-mask 0 --connlimit-saddr \
        -m hashlimit --hashlimit-upto 5/min --hashlimit-burst 5 --hashlimit-mode srcip --hashlimit-name xmpp \
        -m comment --comment xmpp-client \
        -j ACCEPT

    iptables -A tcp -p tcp -m tcp --dport 5269 \
        -m connlimit --connlimit-upto 20 --connlimit-mask 0 --connlimit-saddr \
        -m hashlimit --hashlimit-upto 10/min --hashlimit-burst 10 --hashlimit-mode srcip --hashlimit-name xmpp \
        -m comment --comment xmpp-server \
        -j ACCEPT

Pierwsza z tych reguł limituje połączenia dla klientów naszego serwera, tj. tych osób które będą
miały u nas konto. Z grubsza to maksymalna ilość połączeń jakie może utrzymywać w danym czasie
konkretny adres IP z naszym serwerem to 20. Ilość nowych połączeń, które mogą zostać zainicjowane z
danego IP w czasie jednej minuty to 5. Podobnie sprawa wygląda w przypadku drugiej reguły odnoszącej
się do serwerów, z tym, że tutaj mamy limit 10 nowych połączeń na minutę. Po przekroczeniu tych
limitów zadziała mechanizm bezpieczeństwa i kolejne połączenia będą zrzucane, no chyba, że klient
lub serwer pozamyka poprzednie połączenia.

### Plik ejabberd.yml

Przyszła pora poskładać te wszystkie wyżej opisane rzeczy i stworzyć z nich konfigurację dla serwera
Jabber'a. Interesuje nas plik `/etc/ejabberd/ejabberd.yml` , w którym to trzeba co nieco dopisać i
zmienić. Przede wszystkim, musimy określić porty, na których `ejabberd` będzie nasłuchiwał. Każdy z
tych portów trzeba skonfigurować. W tym przypadku konfigurujemy trzy porty: dla połączeń klienckich,
serwerowych oraz dla serwera www.

Port 5222 (dla połączeń klienckich):

    listen:
      -
        port: 5222
        module: ejabberd_c2s
        certfile: "/etc/ejabberd/ejabberd.pem"
        protocol_options:
          - "no_sslv3"
          - "no_tlsv1"
        ciphers: 'TLS_CIPHERS'
        max_stanza_size: 65536
        shaper: c2s_shaper
        access: c2s
        zlib: false
        resend_on_timeout: if_offline
        tls_compression: false
        dhfile: "/etc/ejabberd/dh4096.pem"

Port 5269 (dla połączeń serwerowych):

    -
        port: 5269
        module: ejabberd_s2s_in
        max_stanza_size: 524288

        ...

    s2s_use_starttls: optional
    s2s_certfile: "/etc/ejabberd/ejabberd.pem"
    s2s_dhfile: "/etc/ejabberd/dh4096.pem"

    s2s_protocol_options:
      - "no_sslv3"
      ##- "no_tlsv1"
    s2s_ciphers: 'TLS_CIPHERS'
    s2s_tls_compression: false

Port 5280 (dla serwera www):

    -
        port: 5280
        module: ejabberd_http
        request_handlers:
          "/websocket": ejabberd_http_ws
        ##  "/pub/archive": mod_http_fileserver
        web_admin: true
        http_bind: true
        ## register: true
        ## captcha: true
        tls: true
        certfile: "/etc/ejabberd/ejabberd.pem"

W przypadku portów 5222 oraz 5269 mamy wykorzystane makro o nazwie `TLS_CIPHERS` odnoszące się do
obsługiwanych szyfrów. Musimy to makro określić w konfiguracji `ejabberd` (można także dodać `!SHA1`
i tym samym wyłączyć protokoły TLSv1.1):

    define_macro:
      'TLS_CIPHERS': "ECDH:DH:!CAMELLIA128:!3DES:!MD5:!RC4:!aNULL:!NULL:!EXPORT:!LOW:!MEDIUM"

Tak skonfigurowany serwer Jabber'a będzie akceptował jedynie próby połączeń, które oferują szyfry
TLSv1.1 oraz TLSv1.2. W przypadku klientów wymagane jest szyfrowanie. Natomiast z racji problemów
przy wymuszeniu szyfrowania w przypadku serwerów, np. GMAIL, szyfrowanie połączenia jest opcjonalne.
Można oczywiście wymusić szyfrowanie na linii serwer=serwer manipulując parametrem
`s2s_use_starttls` ale wtedy trzeba mieć świadomość, że nasz serwer nie da rady nawiązać połączenia
z wieloma serwerami Jabber'a.

Dla poprawy bezpieczeństwa połączeń szyfrowanych korzystamy z własnego pliku z parametrami DH
(Diffie-Hellman). Gdybyśmy nie wygenerowali wcześniej pliku `dh4096.pem` , to `ejabberd` by
wykorzystywał liczbę pierwszą o długości 1024 bitów.

Konfigurujemy także domenę oraz uwierzytelnianie przez [mechanizm
SASL](https://pl.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) (Simple Authentication
and Security Layer):

    disable_sasl_mechanisms: "digest-md5"

    hosts:
      - "morfitronik.pl"

    fqdn: "jabber.morfitronik.pl"

## Test konfiguracji serwera Jabber'a

Gdy już poustawiamy wszystko jak należy, przydałoby się sprawdzić konfigurację serwera pod kątem
szyfrowania oferowanego przy połączeniu z klientami i innymi serwerami. Tego typu test możemy
przeprowadzić w serwisie [xmpp.net](https://xmpp.net). Są tam dwa osobne testy. Mój serwer Jabber'a
w obu przypadkach otrzymał ocenę `A` .

[Link do testu
klienckiego](https://xmpp.net/result.php?id=687741):

[![]({{< baseurl >}}/img/2017/03/013.jabber-ejabberd-serwer-debian-linux-wynik-klient-660x189.png)]({{< baseurl >}}/img/2017/03/013.jabber-ejabberd-serwer-debian-linux-wynik-klient.png)

[Link to testu
serwerowego](https://xmpp.net/result.php?id=687742):

[![]({{< baseurl >}}/img/2017/03/014.jabber-ejabberd-serwer-debian-linux-wynik-server-660x189.png)]({{< baseurl >}}/img/2017/03/014.jabber-ejabberd-serwer-debian-linux-wynik-server.png)

## Tworzenie kont użytkownikom i podłączenia do serwera

Nasz serwer Jabber'a powinien już działać ale jeszcze nie ma na nim stworzonego konta użytkownika,
które można by wykorzystać przy logowaniu. Konto możemy stworzyć z poziomu terminala lub też przy
pomocy panelu administracyjnego udostępnianego przez serwer www.

Poniżej znajduje się przykładowe polecenie konsolowe, które tworzy konto na serwerze Jabber'a:

    # ejabberdctl register user morfitronik.pl haslo
    User user@morfitronik.pl successfully registered

Warto sobie obadać narzędzie `ejabberdctl` ([tutaj znajduje się link do
manuala](https://docs.ejabberd.im/admin/guide/managing/)).

A tak wygląda panel
admina:

![]({{< baseurl >}}/img/2017/03/015.jabber-ejabberd-serwer-debian-linux-panel-admina.png)

## Zmiana adresu i zabezpieczenie panelu admina

Standardowo panel admina jest uruchamiany przez `ejabberd` na wskazanym w konfiguracji adresie
(domenie `morfitronik.pl` ) i porcie 5280. Problemy z tym adresem są dwa. Pierwszy z nich dotyczy
wystawienia tego panelu na widok publiczny, co nie jest zbyt rozważne. Drugi problem to
niestandardowy port dla stron WWW, przez co trzeba go za każdym razem wpisywać ilekroć chcemy
zarządzać serwerem. Można by oczywiście w konfiguracji `ejabberd` zdefiniować port 80 czy 443 dla
tego panelu ale w tym przypadku na tych portach nasłuchuje już Apache2 i serwuje tego bloga. Nie da
rady zatem mieć dwóch usług, które będą nasłuchiwać na tym samym porcie. Można natomiast dorobić
proxy do serwera Apache2 i przekierować zapytania pod wskazany adres i port. W ten sposób wpisując w
przeglądarce adres `http://jabber.morfitronik.pl` (port 80) lub `https://jabber.morfitronik.pl`
(port 443) zostaniemy z automatu przekierowani na adres `https://jabber.morfitronik.pl:5280/admin/`
, gdzie zostanie nam wyrzucone okienko logowania do serwera Jabber'a, które zabezpieczymy sobie
dodatkowo certyfikatami x509.

Zaczniemy od napisania konfiguracji dla wirtualnego hosta. Stwórzmy sobie zatem plik `ejabberd.conf`
w katalogu `/etc/apache2/sites-available/` i dodajmy do niego poniższą treść:

    <VirtualHost *:80>
        ServerName jabber.morfitronik.pl
        ServerAdmin morfik@nsa.com
        DocumentRoot /apache2/jabber
        Redirect permanent / https://jabber.morfitronik.pl/

        LogLevel info
        ErrorLog ${APACHE_LOG_DIR}/error_jabber.morfitronik.log
        CustomLog ${APACHE_LOG_DIR}/access_jabber.morfitronik.log combined

        php_admin_value open_basedir "/apache2/jabber/"
    </VirtualHost>

    <IfModule mod_ssl.c>
        <VirtualHost *:443>
            ServerName jabber.morfitronik.pl
            ServerAdmin morfik@nsa.com
            DocumentRoot /apache2/jabber

            LogLevel info ssl:warn
            ErrorLog ${APACHE_LOG_DIR}/error_jabber.morfitronik.log
            CustomLog ${APACHE_LOG_DIR}/access_jabber.morfitronik.log combined

            php_admin_value open_basedir "/apache2/jabber/"

            SSLEngine on
            SSLCertificateFile      /etc/letsencrypt/live/morfitronik.pl/fullchain.pem
            SSLCertificateKeyFile   /etc/letsencrypt/live/morfitronik.pl/privkey.pem

            <FilesMatch "\.(cgi|shtml|phtml|php)$">
                    SSLOptions +StdEnvVars
            </FilesMatch>
            <Directory /usr/lib/cgi-bin>
                    SSLOptions +StdEnvVars
            </Directory>

            BrowserMatch "MSIE [2-6]" \
                    nokeepalive ssl-unclean-shutdown \
                    downgrade-1.0 force-response-1.0
            BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown

        </VirtualHost>

    </IfModule>

Konfigurację dla tego wirtualnego hosta włączamy linkując wyżej utworzony plik do katalogu
`/etc/apache2/sites-enabled/` .

Teraz włączmy w Apache2 [moduł proxy\_http](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html),
za sprawą którego nasz serwer www zyska umiejętność przekierowania zapytań:

    # a2enmod proxy_http

Musimy jeszcze dodać do pliku `/etc/apache2/sites-available/ejabberd.conf` konfigurację dla proxy,
tj. które zapytania mają być brane pod uwagę. Jako, że w sekcji z portem 80 mamy przekierowanie na
port 443, to tylko do tej drugiej sekcji dopisujemy poniższy kod:

    SSLProxyEngine on
    ProxyRequests Off
    ProxyVia Off

    <Proxy *>
         Require all granted
    </Proxy>

    ProxyPass /admin https://jabber.morfitronik.pl:5280/admin
    ProxyPassReverse /admin https://jabber.morfitronik.pl:5280/admin

W ten sposób zapytania o `jabber.morfitronik.pl/admin` zostaną wewnętrznie z automatu przekierowane
przez Apache2 na `jabber.morfitronik.pl:5280/admin` i odejdzie nam potrzeba wpisywania tego
niestandardowego z punktu widzenia WWW portu.

By jeszcze bardziej uprościć sobie życie, możemy również odpuścić sobie wpisywanie zasobu `/admin` i
zrobić przekierowanie z `jabber.morfitronik.pl` na `jabber.morfitronik.pl/admin` przy pomocy
dyrektywy `RedirectMatch` lub modułu `mod_rewrite` :

    # RedirectMatch ^/$ https://jabber.morfitronik.pl/admin/

    <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteRule   ^/$  /admin/  [R]
    </IfModule>

Ostatnią rzeczą jest zabezpieczenie panelu admina certyfikatami. Tutaj musimy już posiadać własne CA
oraz odpowiednie certyfikaty. Jeśli nie mamy stosownych plików, to trzeba je sobie pierw
wygenerować. Cały [proces generowania certyfikatów przy pomocy
easy-rsa]({{< baseurl >}}/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/) został opisany
osobno. My tutaj już wykorzystamy gotowe certyfikaty. W zasadzie to musimy przesłać na serwer
certyfikat CA i dodać w pliku `/etc/apache2/sites-available/ejabberd.conf` poniższą linijkę:

    SSLCACertificateFile    /etc/CA/keys/ca.crt

Wszystkie certyfikaty klienckie wystawione przez nasze CA będą w ten sposób zaufane. W tym samym
pliku dodajemy też poniższy blok kodu:

    <Location "/admin">
    #   SSLOptions +StdEnvVars
        SSLVerifyClient optional
        SSLVerifyDepth 1
    #   SSLRenegBufferSize 524288

        <IfModule mod_rewrite.c>
                RewriteEngine on
                RewriteCond %{SSL:SSL_CLIENT_VERIFY} !^SUCCESS$
        #       RewriteCond %{REMOTE_ADDR} !^127.0.0.1$
                RewriteRule ^(.*) https://morfitronik.pl [R=301,L]
        </IfModule>

    </Location>

Ma on za zadanie zabezpieczyć zasób, który znajdzie się pod adresem `jabber.morfitronik.pl/admin/` .
Każdy, kto będzie starał się odwiedzić ten URL, będzie musiał przestawić ważny certyfikat. W
przeciwnym wypadku dostęp zostanie odmówiony, a odwiedzający zostanie przekierowany pod adres
`https://morfitronik.pl` .

Zapisujemy konfigurację i restartujemy serwer Apache2:

    # systemctl restart apache2

Certyfikat kliencki musimy zaimportować sobie w przeglądarce. W tym przypadku zrobimy to na
przykładzie Firefox'a. Przechodzimy zatem kolejno do Preferences \> Advanced. Następnie na zakładce
Certificates klikamy View Certificates. W okienku, które się pojawi, przechodzimy na zakładkę Your
Certificates i klikamy w przycisk Import, gdzie podajemy ścieżkę do pliku `client.p12`
:

![]({{< baseurl >}}/img/2017/03/016.jabber-ejabberd-serwer-debian-linux-certyfikat-zasoby.png)

Odwiedzamy teraz adres `jabber.morfitronik.pl` . Bez znaczenia jest tutaj czy dodamy z na początku
`http`/`https` oraz czy określimy zasób `/admin/` , bo i tak wylądujemy docelowo w tym samym
miejscu. Dysponując odpowiednim certyfikatem, powinniśmy zobaczyć okienko logowania do panelu
admina.

## Baza danych MySQL/MariaDB

`ejabberd` domyślnie korzysta z wbudowanej bazy danych zwanej Mnesia. Jeśli na naszym VPS mamy już
zainstalowany jakiś serwer bazy danych, np. MySQL czy MariaDB, to naturalnie możemy ten serwer
podpiąć do serwera Jabber'a. Musimy tylko pozyskać schemat, na podstawie którego stworzymy sobie
bazę danych na potrzeby `ejabberd` . [Tutaj znajdują się wzory schematów
bazy](https://github.com/processone/ejabberd/tree/master/sql), które możemy wykorzystać. Musimy
tylko zdawać sobie sprawę, że jeśli nasz serwer bazy danych to MySQL, to musi on być w wersji co
najmniej 5.6 . W przeciwnym wypadku, przy importowaniu schematu bazy danych napotkamy błąd przy
tworzeniu indeksu FULLTEXT. Pobieramy plik `mysql.sql` i importujemy na serwerze baz danych.
Najpierw tworzymy nową bazę danych:

    # mysql -u root -p

    mysql> CREATE DATABASE ejabberd;

A następnie ładujemy schemat bazy:

    # mysql -u root -p ejabberd < ./mysql.sql

W pliku `/etc/ejabberd/ejabberd.yml` musimy jeszcze załączyć stosowną konfigurację. Zmieniamy metodę
uwierzytelniania z `internal` na `sql` :

    ## auth_method: internal
    auth_method: sql

Następnie definiujemy konfigurację dla bazy danych:

    sql_type: mysql
    sql_server: "localhost"
    sql_database: "ejabberd"
    sql_username: "ejabberd"
    sql_password: "pass-pass"
    ## sql_port: 3306

Możemy także ustawić domyślną bazę dla modułów:

    default_db: sql

Zapisujemy konfigurację i restartujemy resetujemy serwer Jabber'a.

## Dalsza konfiguracja serwera Jabber'a

Powyżej udało nam się jedynie postawić bardzo podstawowy serwer Jabber'a i dość dobrze go
zabezpieczyć ale to jedynie początek. `ejabberd` ma całą masę modułów, które trzeba sobie już we
własnym zakresie skonfigurować podpierając się przy tym [oficjalnym
manualem](https://docs.ejabberd.im/admin/configuration/).
