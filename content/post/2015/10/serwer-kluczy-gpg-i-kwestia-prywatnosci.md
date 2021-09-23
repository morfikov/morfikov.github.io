---
author: Morfik
categories:
- Linux
date: "2015-10-16T18:59:04Z"
date_gmt: 2015-10-16 16:59:04 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- anonimowość
- proxy
title: Serwer kluczy GPG i kwestia prywatności
---

Czytając sobie artykuł na temat [TORyfikacji zapytań do serwerów kluczy
GPG](https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/GnuPG), pomyślałem, że w sumie
mógłbym zreprodukować przedstawione tam kroki dotyczące implementacji tego rozwiązania na windowsie
i wdrożyć je na linux'ie. Chodzi generalnie o to, by serwer kluczy GPG nie był odpytywany
bezpośrednio przy szukaniu/przesyłaniu kluczy przez sieć, bo to może identyfikować nas, jak i grupę
ludzi, która się z nami komunikuje. Jakby nie patrzeć, klucze GPG składają się z dość newralgicznych
informacji, typu imię, nazwisko czy adres email, a serwer kluczy GPG tych danych w żaden sposób nie
zabezpiecza i są one zwykle przesyłane otwartym tekstem przez sieć.

<!--more-->
## Potrzebne oprogramowanie

W debianie mamy dostępne wszystkie wymagane pakiety, a są to: `tor` , jego graficzna nakładka
`vidalia` oraz serwer cache'ujący zapytania `polipo` . Przy instalacji pakietu `vidalia` zostaniemy
poproszeni o wybranie użytkowników, którzy będą w stanie zarządzać demonem TOR'a.

Pakiety sieciowe podlegające TORyfikacji będą zbierane ze wszystkich aplikacji, które mają coś
wspólnego z narzędziami od GPG, np. Thunderbird, który wykorzystuje [dodatek
enigmail](https://www.enigmail.net/index.php/en/). Zatem zapytanie o klucze GPG będzie przesyłane
przez GnuPG do serwera Polipo na port 8118, po czym będzie ono forwardowane do TOR'a na port 9050, z
którego trafi do sieci TOR'a, a następnie na port 443 lub 11371, gdzie nasłuchuje serwer kluczy GPG.
Sama poczta zaś będzie dostarczana standardową drogą, a nie przez TOR'a.

## Konfiguracja Polipo

Zaczynamy od skonfigurowania serwera Polipo. Ten pakiet, który jest w repozytorium debiana nie
posiada usłúgi pod systemd. Nie stanowi to jakiegoś problemu ale można pokusić się o napisanie
własnej usługi w oparciu o skrypt init. Mój plik `/etc/systemd/system/polipo.service` wygląda tak:

    [Unit]
    Description=Polipo Proxy Server

    [Service]
    User=proxy
    Group=proxy
    ExecStart=/usr/bin/polipo

    [Install]
    WantedBy=multi-user.target

Przeładowujemy usługi i dodajemy `polipo.service` do autostartu systemu:

    # systemctl daemon-reload
    # systemctl enable polipo
    # systemctl start polipo

    # ps -eo user,group,args | grep polipo
    proxy    proxy    /usr/bin/polipo

Przechodzimy teraz do plików konfiguracyjnych serwera Polipo. Na potrzeby konfiguracji, którą
przeprowadzamy, nie musimy zbytnio zmieniać ustawień. Domyślnie w pliku `/etc/polipo/config` są
tylko opcje od logowania. Jeśli ktoś chciałby się bardziej zainteresować konfiguracja Polipo, to pod
`/usr/share/doc/polipo/examples/config.sample` jest plik szkieletowy z szeregiem opcji. Wszystkie
pozostałe opcje wraz z opisem można podejrzeć przez wydanie w terminalu poniższego polecenia:

    # polipo -v

Ja uzupełniłem plik konfiguracyjny tymi poniższymi parametrami:

    allowedClients = 127.0.0.1
    allowedPorts = 1-65535
    cacheIsShared = false
    censorReferer = maybe
    censoredHeaders = from, accept-language
    disableConfiguration = true
    disableLocalInterface = true
    diskCacheRoot = ""
    dnsQueryIPv6 = no
    localDocumentRoot = ""
    logFile = /var/log/polipo/polipo.log
    logSyslog = true
    proxyAddress = "127.0.0.1"
    proxyName = "localhost"
    proxyPort = 8118
    socksParentProxy = "localhost:9050"
    socksProxyType = socks5

Poniżej znajduje się lista użytych opcji:

  - `proxyAddress` oraz `proxyPort` określają jaki adres oraz jaki port zostanie przypisany
    serwerowi proxy. W przypadku niepodania portu, zostanie użyty domyślny, tj. 8123.
  - `allowedClients` oraz `allowedPorts` określają z jakiego adresu (sieci) oraz na jakich portach
    serwer proxy ma akceptować połączenia.
  - `proxyName` jest nazwą serwera proxy i domyślnie jest to nazwa maszyny (hostname). Wysyłana jest
    jest w nagłówkach http, co może przełożyć się na identyfikację konkretnego komputera. Dlatego
    zaleca się wpisanie w to miejsce jakiejś wartości, np. `localhost` , która jest zalecana przez
    TOR'a.
  - `cacheIsShared` powinien mieć wartość `false` w przypadku gdy tylko jeden użytkownik będzie
    korzystał z serwera proxy. W przypadku wielu użytkowników, przy `false` mogą wystąpić problemy z
    przeglądaniem stron www.
  - `socksParentProxy` oraz `socksProxyType` definiują nadrzędny adres i typ proxy (forwardowanie
    zapytań do TOR'a).
  - `diskCacheRoot` odpowiada za przechowywanie danych (cache) na dysku. W przypadku nie podania
    ścieżki, cache zostanie wyłączony. Możemy zdefiniować również niestandardowe miejsce jego
    przechowywania.
  - `localDocumentRoot` , `disableLocalInterface` oraz `disableConfiguration` umożliwiają
    konfigurację Polipo za pomocą przeglądarki z poziomu www.
  - `dnsQueryIPv6` odpowiada za rozwiązywanie nazw w protokole ipv6. Jeżeli chcemy komunikować się
    tylko z hostami ipv4, zalecane jest wyłączenie tej opcji, co może się przełożyć na nieco szybsze
    działanie zapytań DNS.
  - `censoredHeaders` oraz `censorReferer` pilnują by w nagłówkach http nie były przesyłane
    informacje o nas. Domyślne ustawienie jest dobrym kompromisem, jednak trzeba pamiętać, że im
    więcej opcji nagłówków tutaj zdefiniujemy, tym więcej stron przestanie działać.

Po wystartowaniu usługi, Polipo powinien nasłuchiwać na adresie `127.0.0.1` i porcie `8118` :

    # netstat -tupan | grep -i polipo
    tcp        0      0 127.0.0.1:8118          0.0.0.0:*               LISTEN      74306/polipo

## Konfiguracja TOR'a

Konfiguracja narzędzia `tor` znajduje się w pliku `/etc/tor/torrc` . Musimy w nim określić adres i
port nasłuchu. Muszą one być dokładnie takie same, co w przypadku opcji `socksParentProxy` , którą
zdefiniowaliśmy wyżej. W tym celu do wspomnianego pliku dodajemy poniższą linijkę:

    SocksPort 127.0.0.1:9050

Po zresetowaniu usługi, `tor` powinien już nasłuchiwać zapytań:

    # netstat -tupan | egrep -i /tor
    tcp        0      0 127.0.0.1:9050          0.0.0.0:*               LISTEN      76266/tor

## Konfiguracja vidalia

W przypadku graficznego interfejsu vidalia możemy napotkać problem objawiający się niemożliwością
wystartowania demona `tor` . Tuż po wystartowaniu aplikacji w logu można zanotować komunikat podobny
do tego poniżej:

    Could not bind to 127.0.0.1:9050: Address already in use. Is Tor already running?

Dzieje się tak ponieważ usługa systemowa TOR'a działa już w tle, a narzędzie `vidalia` , jako że ma
zarządzać demonem TOR'a, próbuje go uruchomić. By rozwiązać ten problem możemy albo zaprzestać
uruchamiania demona `tor` jako usługę systemową, albo też można skorzystać z gniazda kontrolnego
(Control Socket), który jest do skonfigurowania w opcjach `vidalia` :

![](/img/2015/10/1.serwer-kluczy-gpg-tor.png#big)

Po ponownym uruchomieniu programu, ten już bez problemu powinien nas podłączyć do aktualnie
uruchomionego demona `tor` :

![](/img/2015/10/2.serwer-kluczy-gpg-vidalia-tor-polaczenie.png#big)

## Konfiguracja GnuPG/GPG/PGP

W zasadzie kwestie konfiguracji proxy jak i TOR'a mamy już z głowy i powinny one działać. Teraz
musimy jeszcze odpowiednio skonfigurować pakiet `gnupg` (lub `gnupg2` ). Chodzi o nakazanie
narzędziu `gpg` by wszelkie zapytania do serwerów kluczy przesyłał do proxy. By tego dokonać,
musimy dodać poniższy parametr do pliku `~/.gnupg/gpg.conf` :

    keyserver-options http-proxy=http://127.0.0.1:8118

Możemy także nieco zwiększyć parametr `timeout` dla serwera, a to z racji takiej, że TOR nie
zachwyca jakością połączenia. Domyślnie `timeout` jest ustawiony na 30s i jeśli chcielibyśmy go
nieco dostosować, to dodajemy także poniższa opcję:

    keyserver-options timeout=60

Jako, że ten artykuł jest o poprawie bezpieczeństwa i poufności danych w kontekście kluczy GPG, to
przydałoby się także przyłożyć większą uwagę do tego [jakie parametry w pliku
gpg.conf](/post/konfiguracja-gpg-w-pliku-gpg-conf/) można umieścić.

## Testowe zapytanie wysłane pod serwer kluczy GPG

Musimy sprawdzić czy aby pakiety wędrują pierw do Polipo, a następnie przez TOR'a do sieci TOR i
dalej pod docelowy serwer kluczy GPG. Na sam początek dodajmy poniższą linijkę do pliku
`~/.gnupg/gpg.conf` :

    keyserver-options debug

Ma ona na celu zwrócenie większej ilości komunikatów przy wydawaniu poleceń `gpg` odwołujących się
do serwerów kluczy. Powinniśmy zaobserwować poniższy log:

    $ gpg --search-keys 0xCD046810771B6520
    gpg: searching for "0xCD046810771B6520" from hkps server hkps.pool.sks-keyservers.net
    gpgkeys: curl version = libcurl/7.45.0 GnuTLS/3.3.18 zlib/1.2.8 libidn/1.32 libssh2/1.5.0 nghttp2/1.3.2 librtmp/2.3
    gpgkeys: search type is 4, and key is "CD046810771B6520"
    *   Trying 127.0.0.1...
    * Connected to 127.0.0.1 (127.0.0.1) port 8118 (#0)
    * Establish HTTP proxy tunnel to hkps.pool.sks-keyservers.net:443
    > CONNECT hkps.pool.sks-keyservers.net:443 HTTP/1.1
    Host: hkps.pool.sks-keyservers.net:443
    Proxy-Connection: Keep-Alive

    < HTTP/1.1 200 Tunnel established
    <
    * Proxy replied OK to CONNECT request
    ...

Zatem wiemy, że zapytania trafiają do Polipo. Jeśli nie jesteśmy pewni czy aby na pewno są one także
forwardowane do sieci TOR, możemy to sprawdzić na dwa sposoby. Prostszy z nich polega na odpaleniu
narzędzia `vidalia` i popatrzeniu na graf tuż po wydaniu polecenia `gpg --refresh-keys` . Jeśli
kluczy mamy dużo, powinniśmy zaobserwować ruch na wykresie:

![](/img/2015/10/3.server-kluczy-gpg-vidalia-graf.png#big)

Powyższy wykres nie pokazuje, co prawda, połączeń jako takich ale jeśli chcemy się przekonać czy
faktycznie łączymy się z serwerem kluczy, to jego adres możemy odczytać w zakładce `Network Map` :

![](/img/2015/10/4.serwer-kluczy-gpg-vidalia-network-map.png#huge)

Dla pewności, możemy także wyłączyć demona `tor` i spróbować wysłać zapytanie pod serwer kluczy GPG:

    # systemctl stop tor

    $ gpg --search-keys 0xCD046810771B6520
    gpg: searching for "0xCD046810771B6520" from hkps server hkps.pool.sks-keyservers.net
    gpgkeys: curl version = libcurl/7.45.0 GnuTLS/3.3.18 zlib/1.2.8 libidn/1.32 libssh2/1.5.0 nghttp2/1.3.2 librtmp/2.3
    gpgkeys: search type is 4, and key is "CD046810771B6520"
    *   Trying 127.0.0.1...
    * Connected to 127.0.0.1 (127.0.0.1) port 8118 (#0)
    * Establish HTTP proxy tunnel to hkps.pool.sks-keyservers.net:443
    > CONNECT hkps.pool.sks-keyservers.net:443 HTTP/1.1
    Host: hkps.pool.sks-keyservers.net:443
    Proxy-Connection: Keep-Alive

    < HTTP/1.1 504 Couldn't connect: Connection refused
    < Connection: close
    ...
    gpg: keyserver internal error
    gpg: keyserver search failed: keyserver error

Zatem przy ubitym demonie `tor` nie damy rady pobrać kluczy, a to za sprawą opcji `http-proxy=` ,
która widnieje w pliku `~/.gnupg/gpg.conf` i wyraźnie odsyła nas pod konkretny adres, na którym w
tej chwili nic nie nasłuchuje.
