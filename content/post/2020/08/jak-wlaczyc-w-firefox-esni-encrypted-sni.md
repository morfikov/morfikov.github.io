---
author: Morfik
categories:
- Linux
date: "2020-08-10T18:15:00Z"
published: true
status: publish
tags:
- debian
- dns
- dot
- doh
- prywatność
- szyfrowanie
- firefox
title: Jak włączyć w Firefox ESNI (Encrypted SNI)
---

Obecnie szyfrowanie zapytań DNS staje się powoli normą za sprawą protokołu DoH ([DNS over HTTPS][1])
lub DoT ([DNS over TLS][2]). Można by zatem pomyśleć, że wraz z implementacją szyfrowania tego
kluczowego dla działania internetu protokołu (przynajmniej z naszego ludzkiego punktu widzenia),
poprawie ulegnie również nasza prywatność w kwestii odwiedzanych przez nas stron WWW. Niemniej
jednak, w dalszym ciągu można bez problemu wyciągnąć adresy domen, które zamierzamy odwiedzić. Nie
ma przy tym żadnego znaczenia ile stron jest hostowanych na danym adresie IP, ani nawet fakt, że
ruch do serwera WWW będzie szyfrowany (w pasku adresu wpiszemy `https://` ) z wykorzystaniem
protokołu SSL/TLS (w tym również TLS v1.3). Wszystko przez rozszerzenie SNI ([Server Name
Indication][3]), którego to zadaniem jest umożliwienie jednemu serwerowi na prezentowanie wielu
certyfikatów hostowanych w jego obrębie domen. Dzięki takiemu rozwiązaniu, każda domena może
szyfrować ruch niezależnie od siebie na linii serwer<->klient (używać innych kluczy szyfrujących).
Niemniej jednak, podczas nawiązywania szyfrowanego połączenia, w pakiecie ClientHello przesyłanym
do takiego serwera musi znaleźć się nazwa domeny, której to certyfikat serwer będzie musiał nam
przedstawić. Niestety ten pakiet jest przesyłany przez sieć otwartym tekstem, przez co każdy, kto
podsłuchuje naszą komunikację (w tym też nasz ISP), bez problemu może ustalić na jakie strony
internetowe wchodzimy. Ostatnimi czasy jednak pojawiły się dwa rozszerzenia ECH ([Encrypted Client
Hello][10]) oraz ESNI ([Encrypted SNI][8]), które mają zaadresować problemy związane z prywatnością
przez pełne zaszyfrowanie pakietu ClientHello lub też zaszyfrowanie jedynie pola SNI w tym pakiecie.
Póki co, prace nad tymi rozszerzeniami nie są jeszcze skończone ale Firefox w połączeniu z
CloudFlare powoli testują ESNI. Postanowiłem zatem dobrowolnie przyłączyć się do grupy testerów i
wdrożyć na swoim linux'ie to rozszerzenie ESNI dla przeglądarki Firefox.

<!--more-->
## Jak SNI potrafi skompromitować prywatność

By zrozumieć dlaczego wiele stron hostowanych na jednym IP, ani też szyfrowanie ruchu między
serwerem WWW a naszą przeglądarką (w tym również szyfrowanie zapytań DNS), nie daje nam praktycznie
żadnej ochrony w kwestii naszej prywatności przy odwiedzaniu stron WWW, dobrze jest uruchomić sobie
sniffer `wireshark` i przy jego pomocy podejrzeć jakie pakiety są przesyłane przez sieć. Poniżej
mamy standardowe zapytanie o domenę `www.cloudflare.com` , naturalnie odwiedzając w przeglądarce
adres `https://www.cloudflare.com` :

![](/img/2020/08/007-sni-esni-encryypted--debian-linux-privacy-firefox-wireshark.png#huge)

Jak widać, mamy do czynienia z komunikacją szyfrowaną TLS v1.3, a mimo to bez problemu udało nam
się ustalić jaką domenę człowiek próbuje odwiedzić.

Sprawdźmy co się stanie, gdy zaszyfrujemy SNI:

![](/img/2020/08/008-sni-esni-encryypted--debian-linux-privacy-firefox-wireshark.png#huge)

I tu już w miejscu `server_name` mamy `encrypted_server-name` , którego wartości nie jesteśmy w
stanie odszyfrować. Tego typu zabieg sprawia zatem, że mając też zaszyfrowane zapytania DNS, nikt
nie jest w stanie zidentyfikować domeny, która odwiedzamy.

Jeśli ktoś jest [zainteresowany tematem ESNI i jak on dokładnie działa][9], to tutaj jest ciekawy
artykuł na blogu CloudFlare.

### ESNI nie wszędzie jest wspierany

Trzeba tutaj wyraźnie zaznaczyć, że by zrobić użytek z zaszyfrowanego SNI, serwer hostujący strony
WWW (czy też inne usługi) musi wspierać ESNI. W powyższym przypadku, CloudFlare wsparcie dla ESNI
ma zaimplementowane, przez co my nie byliśmy w stanie zidentyfikować nazwy odwiedzanej domeny
przechwytując surowe pakiety sieciowe. Jeśli jednak serwer nie wspiera ESNI, to będziemy musieli do
niego przesłać nazwę domeny otwartym tekstem w pakiecie ClientHello, by otrzymać stosowny
certyfikat. Póki co niewiele serwisów w ogóle wspiera ESNI (chyba tylko CloudFlare). Dodatkowo,
ESNI to rozszerzenie protokołu TLS tylko i wyłącznie v1.3 i jeśli połączymy się z serwerem, który
nie wspiera TLS v1.3 (a takich jest bardzo dużo), to zostanie zestawiony tunel TLS v1.2 czy też
v1.1 i w takim przypadku użytku z ESNI również się nie da zrobić. Zatem nawet mając w Firefox
aktywowany ESNI, to wciąż możemy być narażeni na posłuch.

Trzeba też mieć na uwadze, że włączenie ESNI skutkuje dodatkowym zapytaniem DNS dla każdej nowej
domeny, nawet w przypadku tych serwerów, które nie wspierają ESNI. W ten sposób net może nam net
ździebko spowolnić.

Jako, że ESNI wciąż jest w fazie eksperymentalnej i nie jest wdrożony na szerszą skalę, to może
powodować problemy przy odwiedzaniu pewnych stron WWW. W takim przypadku mogą pojawić się błędy
typu `Secure connection failed - SSL_ERROR_NO_CYPHER_OVERLAP` lub
`SSL_ERROR_MISSING_ESNI_EXTENSION` . Póki co jednak nie zauważyłem problemów z działaniem stron
WWW, a ich przeglądam dość sporo.

## Jak aktywować ESNI w Firefox

By włączyć ESNI w Firefox, wpisujemy w pasku adresu przeglądarki `about:config` i wyszukujemy klucz
`network.security.esni.enabled` . Jak nazwa może sugerować, ten klucz włącza rozszerzenie ESNI z
tym, że nie do końca. Póki co włączenie ESNI w Firefox uzależnione jest od skonfigurowania w tej
przeglądarce TRR ([Trusted Recursive Resolver][4]), którym w zasadzie jest serwer DoH. Zatem by
mieć możliwość w ogóle używać ESNI, trzeba dodatkowo jeszcze przestawić wartość `network.trr.mode`
na `2` lub `3` , w zależności od tego czy chcemy aby w przypadku ewentualnych problemów z
resolver'em Firefox'a był wykorzystywany systemowy resolver naszego linux'a (wartość `2`).

Włączenie TRR spowoduje aktywowanie opcji szyfrowania ruchu DNS i przesłanie go domyślnie do
serwerów CloudFlare:

![](/img/2020/08/001-sni-esni-encryypted--debian-linux-privacy-firefox-dns.png#huge)

Możemy jednak wybrać innego DNS provider'a, choć CloudFlare zdaje się być dobrym rozwiązaniem.
Możemy teraz przejść na [stronę testową][5] i sprawdzić czy udało nam się z powodzeniem włączyć
ESNI:

![](/img/2020/08/002-sni-esni-encryypted--debian-linux-privacy-firefox.png#huge)

Jak widać, wszystkie cztery pozycje zapaliły się na zielono, zatem ESNI zostało włączone prawidłowo.

## Problemy z dnscrypt-proxy

W przypadku, gdy nie korzystamy z bardziej zaawansowanych technik anonimizacji, to powyższe
rozwiązanie powinno nam wystarczyć. Ja jednak od wielu lat korzystam z `dnsmasq` + `dnscrypt-proxy`
do konfiguracji i szyfrowania zapytań DNS w swoim Debianie. Problem jaki pojawia się przy włączaniu
ESNI jest taki, że póki co nie da rady go aktywować [jeśli zamierzamy korzystać z zewnętrznego
oprogramowania do szyfrowania zapytań DNS][6], tj. będziemy szyfrować zapytania DNS poza
przeglądarką Firefox. Przynajmniej standardowo nam to nie zadziała i po odwiedzeniu strony
[cloudflare.com/ssl/encrypted-sni/][5] uzyskamy poniższy wynik:

![](/img/2020/08/003-sni-esni-encryypted--debian-linux-privacy-firefox.png#huge)

Można jednak skonfigurować Firefox'a w taki sposób, by wskazać w jego konfiguracji własny serwer DoH.

## Dnscrypt-proxy i lokalny serwer DoH

Oprogramowanie `dnscrypt-proxy` od jakiegoś już czasu [umożliwia postawienie lokalnego serwera
DoH][7]. Trzeba tylko go skonfigurować, co zajmie nam dosłownie chwilę. W tym celu edytujemy
konfigurację `dnscrypt-proxy` znajdującą się w pliku `/etc/dnscrypt-proxy/dnscrypt-proxy.toml` i
dodajemy do niej te poniższe wpisy:

    [local_doh]
    listen_addresses = ['127.0.0.1:3000']
    path = "/dns-query"
    cert_file = "localhost.pem"
    cert_key_file = "localhost.pem"

Musimy jeszcze wygenerować klucz prywatny i certyfikat:

    # openssl req -x509 -nodes -newkey rsa:2048 -days 3560 -sha256 -keyout \
      localhost.pem -out localhost.pem
    Generating a RSA private key
    ........+++++
    ..........+++++
    writing new private key to 'localhost.pem'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:
    State or Province Name (full name) [Some-State]:
    Locality Name (eg, city) []:
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:
    Email Address []:

Pola, o których wypełnienie zostaniemy poproszeni, mogą zostać puste. Ten plik `localhost.pem`
umieszczamy w katalogu `/etc/dnscrypt-proxy/` i zmieniamy mu prawa dostępu, tak by użytkownik
`_dnscrypt-proxy` miał do niego dostęp:

    # chown _dnscrypt-proxy localhost.pem

Restartujemy też usługę `dnscrypt-proxy.service` :

    # systemctl restart dnscrypt-proxy

W logu systemowym powinny pojawić się poniższe komunikaty:

    systemd[1]: Stopping dnsmasq.service...
    dnsmasq[32881]: exiting on receipt of SIGTERM
    systemd[1]: dnsmasq.service: Succeeded.
    dnscrypt-proxy[32872]: [2020-08-10 04:54:01] [NOTICE] Stopped.
    systemd[1]: Stopped dnsmasq.service.
    systemd[1]: Stopping dnscrypt-proxy.service...
    systemd[1]: dnscrypt-proxy.service: Succeeded.
    systemd[1]: Stopped dnscrypt-proxy.service.
    systemd[1]: dnscrypt-proxy.service: Consumed 2.157s CPU time, no IP traffic.
    systemd[1]: Started dnscrypt-proxy.service.
    systemd[1]: Starting dnsmasq.service...
    dnsmasq[102591]: dnsmasq: syntax check OK.
    dnsmasq[102704]: started, version 2.82 cachesize 10000
    dnsmasq[102704]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect inotify dumpfile
    dnsmasq-dhcp[102704]: DHCP, IP range 192.168.122.2 -- 192.168.122.254, lease time 12h
    dnsmasq-dhcp[102704]: DHCP, sockets bound exclusively to interface virbr0
    dnsmasq[102704]: using only locally-known addresses for domain lh
    dnsmasq[102704]: using nameserver 192.168.1.1#53 for domain mhouse
    dnsmasq[102704]: using nameserver 127.0.2.1#53
    dnsmasq[102704]: read /etc/hosts - 14 addresses
    systemd[1]: Started dnsmasq.service.
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] dnscrypt-proxy 2.0.44
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Network connectivity detected
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Dropping privileges
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Network connectivity detected
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Now listening to 127.0.2.1:53 [UDP]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Now listening to 127.0.2.1:53 [TCP]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Now listening to https://127.0.0.1:3000/dns-query [DoH]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Source [public-resolvers] loaded
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Source [relays] loaded
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Loading the set of whitelisting rules from [/etc/dnscrypt-proxy/whitelist.txt]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Firefox workaround initialized
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Loading the set of blocking rules from [/etc/dnscrypt-proxy/blacklist.txt]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Loading the set of cloaking rules from [/etc/dnscrypt-proxy/cloaking-rules.txt]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:02] [NOTICE] Loading the set of forwarding rules from [/etc/dnscrypt-proxy/forwarding-rules.txt]
    dnscrypt-proxy[102590]: [2020-08-10 04:54:04] [INFO] [cloudflare] TLS version: 304 - Protocol: h2 - Cipher suite: 4865
    dnscrypt-proxy[102590]: [2020-08-10 04:54:04] [NOTICE] [cloudflare] OK (DoH) - rtt: 68ms
    dnscrypt-proxy[102590]: [2020-08-10 04:54:04] [NOTICE] Server with the lowest initial latency: cloudflare (rtt: 68ms)
    dnscrypt-proxy[102590]: [2020-08-10 04:54:04] [NOTICE] dnscrypt-proxy is ready - live servers: 1

Kluczowa w tym logu jest linijka `Now listening to https://127.0.0.1:3000/dns-query [DoH]` . Ten
adres musimy podać w konfiguracji Firefox'a w ustawieniach sieci:

![](/img/2020/08/004-sni-esni-encryypted--debian-linux-privacy-firefox-dnscrypt-proxy.png#huge)

Testujemy połączenie z naszym lokalnym serwerem DoH przez wpisanie w pasku adresu przeglądarki
`https://127.0.0.1:3000/dns-query` . Powinno nam wyskoczyć ostrzeżenie o certyfikacie typu
self-signed, co wygląda mniej więcej tak:

![](/img/2020/08/005-sni-esni-encryypted--debian-linux-privacy-firefox-cert.png#huge)

Akceptujemy ostrzeżenie, po czym naszym oczom powinna pokazać się biała strona z napisem
`dnscrypt-proxy local DoH server` .

Warto dodać tutaj, że bez tego powyższego kroku mającego zaakceptowanie naszego certyfikatu,
rozwiązywanie domen DNS nie będzie działać, zaś w logu systemowym będzie widnieć sporo poniższych
komunikatów:

    dnscrypt-proxy[102590]: 2020/08/10 17:52:59 http: TLS handshake error from 127.0.0.1:32948: remote error: tls: bad certificate

Po dodaniu certyfikatu do przeglądarki, przechodzimy ponownie do testu na ESNI:

![](/img/2020/08/006-sni-esni-encryypted--debian-linux-privacy-firefox.png#huge)

I jak widać, udało nam się znów zapalić wszystkie cztery kontrolki, z tą różnicą, że obecnie
korzystamy z `dnscrypt-proxy` , a nie z wbudowanego w Firefox resolver'a.


[1]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[2]: https://en.wikipedia.org/wiki/DNS_over_TLS
[3]: https://en.wikipedia.org/wiki/Server_Name_Indication
[4]: https://wiki.mozilla.org/Trusted_Recursive_Resolver
[5]: https://www.cloudflare.com/ssl/encrypted-sni/
[6]: https://bugzilla.mozilla.org/show_bug.cgi?id=1500289
[7]: https://github.com/DNSCrypt/dnscrypt-proxy/wiki/Local-DoH
[8]: https://tools.ietf.org/html/draft-ietf-tls-esni-07
[9]: https://blog.cloudflare.com/encrypted-sni/
[10]: https://en.wikipedia.org/wiki/Server_Name_Indication#Encrypted_Client_Hello
