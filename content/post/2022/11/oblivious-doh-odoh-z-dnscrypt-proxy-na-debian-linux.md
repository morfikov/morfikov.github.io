---
author: Morfik
categories:
- Linux
date:    2022-11-21 17:42:00 +0100
lastmod: 2022-11-21 17:42:00 +0100
published: true
status: publish
tags:
- debian
- dns
- dnscrypt
- doh
- odoh
- prywatność
- resolver
GHissueID: 592
title: Oblivious DoH (ODoH) z dnscrypt-proxy na Debian Linux
---

Parę miesięcy temu natrafiłem na taki oto [artykuł na blogu Cloudflare][1], który poświęcony jest
poprawie prywatności wykonywanych przez klientów zapytań DNS za sprawą wykorzystania mechanizmu
zwanego Oblivious DoH (ODoH). Oczywiście postanowiłem już wtedy zbadać czym ten ODoH jest ale
standardowe narzędzia dostępne w linux'ie (Debian/Ubuntu) jeszcze (przynajmniej wtedy) nie dojrzały
do obsługi ODoH. Obecnie już jest trochę lepiej ale Oblivious DoH wciąż jest w fazie
eksperymentów ([RFC9230][2]). Póki co, jedynym znanym mi narzędziem, które posiada wsparcie dla
ODoH jest [dnscrypt-proxy][3], którego instalację i konfigurację do współpracy z `dnsmasq`
już [jakiś czas temu opisywałem][5]. Mając wsparcie dla ODoH w `dnscrypt-proxy` możemy pokusić się
o wdrożenie Oblivious DoH w swoim systemie i sprawdzić czy faktycznie taki zabieg przyczyni się do
poprawy prywatności wykonywanych przez nas zapytań DNS i właśnie temu zagadnieniu będzie poświęcony
niniejszy wpis.

<!--more-->
##  Czym jest Oblivious DoH (ODoH)

Czytając wpis na blogu Cloudflare możemy wyłapać informację, że Oblivious DoH ma na celu
ograniczenie informacji, które serwer DNS, otrzymuje od klienta przy wykonywaniu zapytań DNS.
Chodzi generalnie o to, że przy przesyłaniu zapytania o domenę do serwera DNS, taki serwer
otrzymuje również informację o adresie IP klienta, który to zapytanie wykonał. Te dwie
informacje (domena + adres IP) mogą bez większego trudu pomóc w ustaleniu maszyny (albo też i jej
właściciela), która dokonała takiego zapytania (wystarczy zajrzeć w logi serwera DNS, nawet gdy
taki podmiot twierdzi, że takowych nie zbiera).

ODoH ma więc na celu rozdzielenie tych dwóch informacji przez zaprzęgnięcie do pracy serwera proxy.
W ten sposób do serwera proxy trafia zaszyfrowane zapytane DNS i serwer proxy nie jest w stanie
tego zapytania odczytać. Takie zaszyfrowane zapytanie jest jedynie przekazywane dalej, tj. do
właściwego serwera DNS i tam dopiero przez serwer DNS deszyfrowane i przetwarzane. Odpowiedź z
serwera DNS jest również szyfrowana i zwracana do serwera proxy. I ponownie, serwer proxy nie jest
w stanie tej odpowiedzi odczytać, a jedynie przekazuje ją do klienta, który to zapytanie o domenę
wykonał.

Dzięki takiemu rozwiązaniu, serwer DNS zna jedynie informację na temat domeny ale nie zna adresu IP
klienta, bo ten adres wskazuje na IP serwera proxy. Jeśli teraz serwer DNS i serwer proxy będą
należeć do osobnych podmiotów, które mają swoje infrastruktury sieciowe w innych częściach
globu (np. w innych krajach czy kontynentach), to istnieje bardzo małe prawdopodobieństwo, że uda
się te dwie informacje pozyskać i poskładać razem, tak by namierzyć autora zapytania DNS.

## Dnscrypt-proxy w wersji 2.1.0

Wspomniałem we wstępie, że jedynym znanym mi (przynajmniej póki co) narzędziem posiadającym
wsparcie dla protokołu ODoH jest `dnscrypt-proxy` . Trzeba tutaj jednak wyraźnie zaznaczyć, że
wsparcie dla mechanizmu Oblivious DoH w `dnscrypt-proxy` pojawiło się dopiero w [wersji 2.1.0][4],
która została wypuszczona dnia 2021-08-14. Do momentu pisania tego artykułu, w Debianie dostępna
jest jedynie starsza wersja, tj. `2.0.45+ds1-1+b8` . Nie wiem dlaczego nowsza
wersja `dnscrypt-proxy` nie może się dostać do repozytorium Debiana ale jeśli chcemy mieć możliwość
wdrożenia ODoH, to trzeba będzie tę nowszą wersję `dnscrypt-proxy` w jakiś sposób sobie pozyskać.

### Prekompilowana binarka

Jednym ze sposobów na pozyskanie `dnscrypt-proxy` w najnowszej wersji jest pobranie prekompilowanej
binarki, która jest [udostępniana na GitHub'ie][6]. Jest tam też załączony plik `.minisig` , przy
pomocy którego to możemy zweryfikować podpis cyfrowy złożony pod odpowiadającym mu plikiem ZIP:

    $ wget https://github.com/DNSCrypt/dnscrypt-proxy/releases/download/2.1.2/dnscrypt-proxy-linux_x86_64-2.1.2.tar.gz

    $ wget  https://github.com/DNSCrypt/dnscrypt-proxy/releases/download/2.1.2/dnscrypt-proxy-linux_x86_64-2.1.2.tar.gz.minisig

    $ ls -al | grep dnscrypt
    -rw-r--r--  1 morfik morfik  4214328 2022-08-01 18:11:33 dnscrypt-proxy-linux_x86_64-2.1.2.tar.gz
    -rw-r--r--  1 morfik morfik      335 2022-08-01 18:11:32 dnscrypt-proxy-linux_x86_64-2.1.2.tar.gz.minisig

Podpis weryfikujemy w poniższy sposób (potrzebny nam będzie pakiet `minisign` , dostępny
standardowo w repozytorium Debiana):

    $ minisign -Vm dnscrypt-proxy-*.tar.gz -P RWTk1xXqcTODeYttYMCMLo0YJHaFEHn7a3akqHlb/7QvIQXHVPxKbjB5
    Signature and comment signature verified
    Trusted comment: timestamp:1659370290   file:dnscrypt-proxy-linux_x86_64-2.1.2.tar.gz   hashed

Tego typu weryfikacja zapewnia nas, że plik nie został w żaden sposób zmieniony przez osoby trzecie,
np. pracowników GitHub'a i jak najbardziej możemy go wgrać do swojego systemu.

### Kompilacja dnscrypt-proxy ze źródeł i budowa paczki .deb

Ja naturalnie wolę zbudować sobie paczkę `.deb` i umieścić ją w swoim prywatnym repozytorium, tak
by menadżer pakietów APT śledził wszystkie pliki, które za sprawą takiego pakietu zostaną
zainstalowane w systemie. [Artykuł na temat budowania paczek .deb znajduje się tutaj][7], a tutaj
jest [wpis na temat stworzenia własnego repozytorium przy pomocy reprepro][8]. Mając te dwie rzeczy
ogarnięte, zbudowanie  paczki `.deb` z `dnscrypt-proxy` jest stosunkowo proste.

Pobieramy źródła `dnscrypt-proxy` z repozytorium git:

    $ git clone --recursive https://github.com/DNSCrypt/dnscrypt-proxy

Pobieramy źródła z repozytorium Debiana (potrzebny nam będzie jedynie katalog `debian/` z
konfiguracją dla systemu budowania pakietów):

    $ apt-get source dnscrypt-proxy

Kopiujemy katalog `debian/` ze źródeł pobranej paczki `.deb` do źródeł git'a i przechodzimy do
katalogu ze źródłami z git'a. Tam przy pomocy `dch` podbijamy wersję nowego pakietu `.deb` :

    $ dch -i
    dnscrypt-proxy (2.1.2+git20221116-1.1) unstable; urgency=medium

      * Non-maintainer upload.
      * New upstream release
      * Git commit (#a89d961)

     -- Mikhail Morfikov <morfik@nsa.com>  Sat, 19 Nov 2022 15:42:48 +0100

Katalog ze źródłami git'a pakujemy:

    $ tar --exclude='.git' --exclude='.gitignore' --exclude='.pc' --exclude='debian' -cf - ../dnscrypt-proxy   | xz -9 -c - > ../dnscrypt-proxy_2.1.2+git20221116.orig.tar.xz

Następnie budujemy źródła via `debuild`

    $ debuild -S -sa -d -i --lintian-opts --profile debian

Teraz możemy zbudować pakiet `.deb` przy pomocy `debuilder` :

    $ sudo pbuilder --build ../dnscrypt-proxy_2.1.2+git20221116-1.1.dsc

Opcjonalnie można też w pliku `debian/control` dostosować zależności ale w tym
przypadku `dnscrypt-proxy` zbudował się bez problemów.

Warto tutaj jednak zaznaczyć, że przy wydawaniu polecenia `git` pojawiła się opcja `--recursive` ,
która dociągnęła niezbędne moduły i umieściła je w katalogu `vendor/` . Gdy `dnscrypt-proxy` jest
budowany oficjalnie, to ten katalog jest ze źródeł usuwany, a wszystkie niezbędne zależności są
pakietowane osobno i znajdują się w odpowiednich paczkach. Póki co, szereg zależności nie został
jeszcze spakietowanych i trzeba by pierw stosowne kroki poczynić, w co trzeba by trochę pracy
włożyć. Ten wyżej opisany sposób nie do końca jest debianowy ale robi to, co powinien przy
najmniejszym nakładzie sił z naszej strony.

Tak przygotowaną paczkę wrzucamy sobie do lokalnego repozytorium:

    $ reprepro include sid /media/debuilder/pbuilder/result/dnscrypt-proxy_2.1.2+git20221116-1.1_amd64.changes

I paczka jest gotowa do instalacji w systemie przy pomocy menadżera
pakietów `apt-get` / `aptitude` .

## Konfiguracja dnscrypt-proxy do pracy z ODoH

Cały proces konfiguracji samego `dnscrypt-proxy` został dość dokładnie opisany w osobnym artykule i
nie będę tego zagadnienia tutaj poruszał. Jeśli nie mamy jeszcze wgranego i/lub skonfigurowanego
`dnscrypt-proxy` , to polecam zapoznanie się z [tym artykułem][5]. Poniższa część wpisu zakłada, że
`dnscrypt-proxy` działa, a nasz linux potrafi przy jego pomocy realizować zapytania DNS.

Przy założeniu, że `dnscrypt-proxy` działa poprawnie, jego konfiguracja pod kątem zaimplementowania
mechanizmu ODoH jest relatywnie prosta. Całość sprowadza się w zasadzie do edycji pliku
`/etc/dnscrypt-proxy/dnscrypt-proxy.toml` . Stosowne sekcje tego pliku zostały zamieszczone poniżej
wraz z krótkim opisem:

Przede wszystkim musimy wybrać serwer ODoH. Sam serwer DoH nie wystarczy. Provider DNS musi
wspierać ODoH. Cloudflare naturalnie posiada wsparcie dla protokołu ODoH i możemy skorzystać tylko
z tego providera ale możemy także podać kilku providerów DNS ([pełna ich lista znajduje się
tutaj][9]):

    server_names = ['odoh-cloudflare', 'odoh-koki-ams']

By mój wybrać `odoh-cloudflare` lub/i `odoh-koki-ams`, musimy uwzględnić w konfiguracji
`dnscrypt-proxy` listę serwerów ODoH:

    odoh_servers = true

Musimy także w sekcji `[sources]` dopisać/odkomentować poniższy blok kodu:

    ...
    [sources]
    ...
      ### ODoH (Oblivious DoH) servers and relays

       [sources.odoh-servers]
         urls = ['https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v3/odoh-servers.md', 'https://download.dnscrypt.info/resolvers-list/v3/odoh-servers.md', 'https://ipv6.download.dnscrypt.info/resolvers-list/v3/odoh-servers.md']
         cache_file = 'odoh-servers.md'
         minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
         refresh_delay = 24
         prefix = ''
       [sources.odoh-relays]
         urls = ['https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v3/odoh-relays.md', 'https://download.dnscrypt.info/resolvers-list/v3/odoh-relays.md', 'https://ipv6.download.dnscrypt.info/resolvers-list/v3/odoh-relays.md']
         cache_file = 'odoh-relays.md'
         minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
         refresh_delay = 24
         prefix = ''
    ...

Następnie w sekcji `[anonymized_dns]` dodajemy trasy (przekaźniki/proxy) dla poszczególnych
serwerów DNS:

    ...
    [anonymized_dns]
    ...
     routes = [
     ...
        { server_name='odoh-cloudflare', via=['odohrelay-crypto-sx', 'odohrelay-surf'] },
        { server_name='odoh-koki-ams', via=['odohrelay-koki-ams', 'odohrelay-koki-bcn'] }
     ...
     ]
    ...

Ta powyższa konfiguracja określa trasę osobno do serwera `odoh-cloudflare` oraz dla serwera
`odoh-koki-ams` . Składnia jest następująca: w `server_name=` określamy to, co zostało wpisane w
parametrze `server_names =` (wyżej w pliku `/etc/dnscrypt-proxy/dnscrypt-proxy.toml` ), zaś w
`via=[]` definiujemy dowolną ilość przekaźników/proxy. Jeśli przekaźników będzie więcej niż jeden,
tak jak w tym przypadku mamy dwa, to zaszyfrowane zapytania DNS zostaną przesłane przez pierwszy
bądź drugi przekaźnik. Podobnie ma się sprawa w przypadku serwera `odoh-koki-ams` .

## Test ODoH w dnscrypt-proxy

Te powyższe wpisy w pliku `/etc/dnscrypt-proxy/dnscrypt-proxy.toml` są w zasadzie wszystkim, co
potrzebne jest by `dnscrypt-proxy` zaczął komunikować się z wykorzystaniem mechanizmu ODoH.
Wypadałoby teraz przetestować czy tak jest w istocie. Restartujemy zatem demona `dnscrypt-proxy` i
patrzymy w log:

    ...
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] dnscrypt-proxy 2.1.2
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Network connectivity detected
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Dropping privileges
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Network connectivity detected
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Now listening to 127.0.2.1:5353 [UDP]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Now listening to 127.0.2.1:5353 [TCP]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Now listening to [::1]:5353 [UDP]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Now listening to [::1]:5353 [TCP]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Now listening to https://127.0.2.1:3000/dns-query [DoH]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Source [public-resolvers] loaded
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Source [relays] loaded
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Source [odoh-servers] loaded
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Source [odoh-relays] loaded
    ...

Na powyższym logu widać, że źródła dla `[odoh-relays]` oraz `[odoh-servers]` zostały z powodzeniem
załadowane.

Dalej mamy informacje o skonfigurowanych trasach i serwerach DNS:

    ...
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Anonymized DNS: routing [odoh-cloudflare] via [odohrelay-crypto-sx odohrelay-surf]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Anonymized DNS: routing [odoh-koki-ams] via [odohrelay-koki-ams odohrelay-koki-bcn]
    ...
    dnscrypt-proxy[140118]: [2022-11-21 15:53:41] [NOTICE] Anonymizing queries for [odoh-cloudflare] via [odohrelay-crypto-sx]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:43] [INFO] [odoh-cloudflare] TLS version: 304 - Protocol: h2 - Cipher suite: 4865
    dnscrypt-proxy[140118]: [2022-11-21 15:53:43] [NOTICE] [odoh-cloudflare] OK (ODoH) - rtt: 75ms
    dnscrypt-proxy[140118]: [2022-11-21 15:53:44] [NOTICE] Anonymizing queries for [odoh-koki-ams] via [odohrelay-koki-ams]
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [INFO] [odoh-koki-ams] TLS version: 304 - Protocol: h2 - Cipher suite: 4865
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] [odoh-koki-ams] OK (ODoH) - rtt: 88ms
    ...
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] Sorted latencies:
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] -    75ms odoh-cloudflare
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] -    88ms odoh-koki-ams
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] Server with the lowest initial latency: odoh-cloudflare (rtt: 75ms)
    dnscrypt-proxy[140118]: [2022-11-21 15:53:49] [NOTICE] dnscrypt-proxy is ready - live servers: 2
    ...

Zatem wszystko zdaje się działać poprawnie.

Odpalmy zatem terminal i spróbujmy rozwiązać jakąś domenę:

    $ /usr/sbin/dnscrypt-proxy -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml -resolve blog.cloudflare.com
    Resolving [blog.cloudflare.com] using 127.0.2.1 port 5353

    Resolver      : 172.70.245.120

    Canonical name: blog.cloudflare.com.

    IPv4 addresses: 104.18.28.7, 104.18.29.7
    IPv6 addresses: -

    Name servers  : vin.ns.cloudflare.com., jule.ns.cloudflare.com.
    DNSSEC signed : yes
    Mail servers  : no mail servers found

    HTTPS alias   : -
    HTTPS info    : [alpn]=[h3,h3-29,h2], [ipv4hint]=[104.18.28.7,104.18.29.7], [ipv6hint]=[2606:4700::6812:1c07,2606:4700::6812:1d07]

    Host info     : -
    TXT records   : facebook-domain-verification=u7qg1p6sqxiw7gbbo14awsjuselbfa, google-site-verification=pk6_Y8biD39iFLnGSJleg1mheNNSOczzlDrJLPWeaHU

Jak widać, rozwiązanie domeny `blog.cloudflare.com` zakończyło się powodzeniem, zatem obsługa
protokołu ODoH w `dnscrypt-proxy` jest realizowana bez żadnych problemów.

## Podsumowanie

Im mniej informacji pozyskuje docelowy serwer DNS, tym lepiej dla nas, tj. użytkowników internetu,
nawet jeśli przesyłane do takiego serwera dane są w pełni zaszyfrowane i niedostępne dla osób
postronnych. Nasuwa się jednak pytanie, po co stosować Oblivious DoH skoro różnego rodzaju
providerzy (w tym też ci od DNS) zapewniają nas, że logów nie przechowują? Skoro nie ma logów, to
żadne organy ścigania nie będą w stanie powiązać zapytań DNS z adresami IP, więc po co to wszystko?
Liczne przypadki, o których można poczytać/posłuchać w mediach dowodzą potrzeby istnienia i
wdrożenia takiego mechanizmu jak ODoH, bo ci co ogłaszają się, że logów nie przechowują, jakimś
dziwnym trafem jednak te logi posiadają ku zdziwieniu całym rzeszom wielce zawiedzionych
użytkowników ([patrz NordVPN][10]). Dlatego też wyposażenie się w `dnscrypt-proxy` oraz
skonfigurowanie go tak, aby rozwiązywał domeny z wykorzystaniem protokołu ODoH jest konieczne i
powinno być obowiązkowe dla osób, które cenią sobie prywatność przy codziennym korzystaniu z sieci.


[1]: https://blog.cloudflare.com/oblivious-dns/
[2]: https://datatracker.ietf.org/doc/html/rfc9230
[3]: https://dnscrypt.info/
[4]: https://github.com/DNSCrypt/dnscrypt-proxy/releases/tag/2.1.0
[5]: /post/szyfrowany-dns-z-dnscrypt-proxy-i-dnsmasq-na-debian-linux/
[6]: https://github.com/DNSCrypt/dnscrypt-proxy/releases/
[7]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
[8]: /post/tworzenie-repozytorium-przy-pomocy-reprepro/
[9]: https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v3/odoh-servers.md
[10]: https://niebezpiecznik.pl/post/wazna-uwaga-dla-uzytkownikow-nordvpn/
