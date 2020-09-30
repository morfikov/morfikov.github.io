---
author: Morfik
categories:
- Linux
date: "2016-12-06T19:22:49Z"
date_gmt: 2016-12-06 18:22:49 +0100
published: true
status: publish
tags:
- prywatność
- vpn
- debian
- sieć
title: Jak skonfigurować serwer VPN na Debianie (OpenVPN)
---

Co raz częściej słychać w mediach o próbie cenzurowania internetu i blokowaniu dostępu do kolejnych
serwisów i stron www. Ostatnio znowu podnieśli kwestię blokowania pokemonów bezpośrednio u ISP tak,
by za zgodą ISP (władzy) można przeglądać tego typu serwisy. W sumie mam już tego dość i korzystając
z okazji, że posiadam za granicą niewielkich rozmiarów VPS, który i tak nie jest zbytnio obciążony,
to postanowiłem sobie skonfigurować na nim serwer VPN w oparciu o [oprogramowanie
OpenVPN](https://openvpn.net/index.php/open-source/documentation/howto.html), które jest standardowo
dostępne w każdej dystrybucji linux'a. Proces konfiguracji serwera jak i klienta z zainstalowanym
Debianem zostanie opisany poniżej. Niemniej jednak, inne urządzenia takie jak routery WiFi i
smartfony również wymagają zaimplementowania w nich mechanizmu szyfrującego ruch ale te rozwiązania
zostaną opisane w osobnych wątkach.

<!--more-->
## Instalacja oprogramowania na potrzeby VPN

VPN, który zamierzamy sobie skonfigurować, wymaga postawienia serwera w celu możliwości nawiązania
połączenia przez określonych klientów. Zarówno na serwerze jak i klientach musimy doinstalować sobie
pakiet `openvpn` . W zasadzie to jest całe niezbędne nam oprogramowanie ale trzeba także zdawać
sobie sprawę, że skoro VPN szyfruje cały ruch na linii serwer-klient, to potrzebne nam też będą
certyfikaty i klucze szyfrujące, a tu z kolei również trzeba będzie postawić CA. Do tego celu
najlepiej skorzystać ze skryptów dostarczanych w pakiecie `easy-rsa` . Oba te wyżej wymienione
pakiety standardowo są dostępne w repozytorium dystrybucji Debian i nie powinno być żadnych
problemów z ich instalacją w systemie.

## Centrum certyfikacji i generowanie certyfikatów

Przed przystąpieniem do konfiguracji VPN musimy sobie stworzyć Centrum Certyfikacji (CA). Zadaniem
takiego CA jest podpisanie certyfikatów zarówno serwerowi jak i klientom, którzy będą się łączyć z
serwerem VPN. Takie CA powinno być postawione na innej maszynie w stosunku do serwera VPN i jego
ewentualnych klientów.

Stosowanie certyfikatów do uwierzytelniania klientów podczas nawiązywania połączenia z serwerem VPN
jest opcjonalne. W przypadku publicznych serwerów VPN, takie certyfikaty nie są zbyt pożądane, bo
tylko komplikują cały proces. Jeśli jednak stawiamy serwer tylko dla siebie i konkretnych osób z
naszego otoczenia, którym chcemy zezwolić na korzystanie z naszego serwera VPN, to dobrze jest dla
każdego takiego klienta wygenerować stosowny certyfikat i podpisać go kluczem CA. W przypadku
wykorzystywania obustronnej weryfikacji certyfikatów, serwer zaakceptuje jedynie te połączenia,
które pochodzą od klientów mających certyfikat podpisany przez CA.

Nie będę tutaj opisywał dokładnego procesu tworzenia CA oraz generowania poszczególnych
certyfikatów. Proces [generowania certyfikatów z wykorzystaniem narzędzia easy-rsa na
Debanie](/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/) został opisany
osobno. Zatem jeśli nie dysponujemy jeszcze odpowiednimi certyfikatami dla serwera i klienta oraz
nie posiadamy jeszcze własnego CA, to zapraszam do zapoznania się z wyżej podlinkowanym artykułem.
Poniżej jest tylko skrót z tego artykułu:

    $ cd ./certs/
    $ make-cadir ./CA/
    $ source ./vars
    $ ./clean-all
    $ ./build-ca
    $ ./build-dh
    $ ./build-key-server server-vpn
    $ ./build-key-pkcs12 client-vpn-morfik

Po wydaniu tych powyższych poleceń w katalogu `CA/keys/` zostało utworzonych kilka plików. Część z
nich trzeba przesłać na serwer, a część na klienta/klientów. Na serwer VPN przesyłamy następujące
pliki: `ca.crt` , `server-vpn.crt` , `server-vpn.key` oraz `dh4096.pem` . Na klienta zaś: `ca.crt` ,
`client-vpn-morfik.crt` , `client-vpn-morfik.key` . Te pliki dobrze umieścić w katalogu
`/etc/openvpn/` .

## Dobór adresacji IP VPN

Kolejnym krokiem przy konfigurowaniu VPN jest określenie adresacji, która będzie wykorzystywana przy
routingu pakietów. Do wyboru mamy trzy zakresy adresów: 10.0.0.0 - 10.255.255.255 (10/8 prefix),
172.16.0.0 - 172.31.255.255 (172.16/12 prefix) oraz 192.168.0.0 - 192.168.255.255 (192.168/16
prefix). Pierwszy i ostatni zakres jest często wykorzystywany w sieciach LAN czy to naszego ISP, czy
też naszej domowej sieci WiFi. Jeśli chcemy korzystać z tych bloków adresowych, to upewnijmy się, że
zakresy adresów nie będą ze sobą kolidować, np. wybierając 192.168.201.0/24 zamiast 192.168.0.0/24 .

## Konfiguracja serwera VPN

W katalogu `/usr/share/doc/openvpn/examples/` znajdują się przykładowe pliki konfiguracyjne, z
których możemy skorzystać podczas konfiguracji zarówno serwera jak i klienta VPN. Nas generalnie
interesują dwa pliki `server.conf.gz` oraz `client.conf` . Ten pierwszy naturalnie trzeba pierw
rozpakować. Zacznijmy może od konfiguracji samego serwera VPN. Kopiujemy zatem zawartość pliku
`/usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz` do katalogu `/etc/openvpn/`
    :

    # zcat /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server-vpn.conf

Następnie poddajemy edycji ten plik, tak by znalazła się w nim poniższa zawartość:

    local 11.22.33.44
    port 1194
    proto udp
    dev tun
    topology subnet
    server 10.8.0.0 255.255.255.0
    ifconfig-pool-persist ipp.txt
    push "redirect-gateway def1 bypass-dhcp"
    push "dhcp-option DNS 208.67.222.222"
    push "dhcp-option DNS 208.67.220.220"
    ;client-to-client
    ;duplicate-cn
    keepalive 10 120
    cipher AES-256-CBC
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
    auth SHA512
    keysize 256
    tls-version-min 1.2
    comp-lzo
    ;max-clients 100
    user nobody
    group nogroup
    persist-key
    persist-tun
    ;status openvpn-status.log
    ;log         openvpn.log
    ;log-append  openvpn.log
    verb 3
    ;mute 20

    ;client-cert-not-required
    ;plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so login

    ;ca /etc/openvpn/certs/ca.crt
    ;cert /etc/openvpn/certs/server-vpn.crt
    ;key /etc/openvpn/certs/server-vpn.key
    dh /etc/openvpn/certs/dh4096.pem
    ;tls-auth /etc/openvpn/certs/ta.key 0
    key-direction 0

    <ca>
    ...
    </ca>
    <cert>
    ...
    </cert>
    <key>
    ...
    </key>
    <tls-auth>
    ...
    </tls-auth>

Generalnie rzecz biorąc, nie wszystkie opcje musimy precyzować ale te które nie zostaną określone w
powyższym pliku przyjmą domyślne wartości. Dlatego też dla czytelności konfiguracji lepiej jest je
załączyć. Poniżej znajduje się krótka rozpiska każdego parametru:

  - `local` -- określa adres IP, na którym będzie nasłuchiwał demon `openvpn` , przez co mamy
    możliwość określenia interfejsu sieciowego dla VPN. Jeśli ta opcja nie zostanie podana, to
    demon będzie nasłuchiwał na każdym interfejsie.
  - `port` -- port serwera VPN, na który mają być przesyłane zapytania, domyślnie `1194` .
  - `proto` -- protokół wykorzystywany przy realizowaniu połączenia VPN. Do wyboru są `tcp` i `udp`
    , z czego ten drugi jest domyślny. Nie jest zalecane korzystanie z `tcp` i generalnie powinniśmy
    unikać go, no chyba, że nie działa u nas `udp` . Warto tutaj nadmienić, że wydajność VPN przy
    protokole `tcp` dość znacznie spada.
  - `dev` -- określa nazwę interfejsów sieciowych, które będą wykorzystywane przez serwer. Możliwe
    do wyboru są `tun` lub `tap` . Jeśli nie podamy numerka, to demon `openvpn` dobierze go
    automatycznie w zależności od wykorzystywanych już interfejsów przez kernel, tj. jeśli `tun0`
    będzie zajęty, to zostanie stworzony `tun1` , itp.
  - `topology` -- w zależności od wykorzystywanej wersji `openvpn` ten parametr może przyjąć
    domyślną wartość `net30` lub `subnet` . Różnica między nimi sprowadza się do określenia
    adresacji dla klientów. W przypadku `net30` , serwer oddeleguje 4 adresy IP dla każdego
    pojedynczego klienta. W przypadku `subnet` adresacja jest przydzielana przez określenie adresu
    IP oraz maski, czyli mniej więcej tak jak w standardowym połączeniu. Nie znaczy to jednak, że
    klienci VPN będą mogli się komunikować ze sobą.
  - `client-to-client` -- zezwala na komunikowanie się klientów, tak jakby byli w jednej sieci LAN.
  - `server` -- określa zakres adresów IP wykorzystywanych prze VPN. Pierwszy z tych adresów jest
    zawsze przydzielany serwerowi.
  - `push` -- umożliwia przesłanie klientowi VPN konkretnych opcji dla połączenia, np. jesteśmy w
    stanie przepisać adres IP serwera DNS czy też dodać niestandardową trasę do tablicy routingu.
  - `push "redirect-gateway def1 bypass-dhcp"` -- przekierowuje cały ruch klienta do serwera VPN
    przez dodanie do jego tablicy routingu wpisów `0.0.0.0/1` oraz `128.0.0.0/1` efektywnie
    nadpisując domyślną trasę klienta bez potrzeby jej usuwania z tablicy routingu. Użyty tutaj
    `bypass-dhcp` ma za zadnie przesłać ruch DHCP do lokalnego serwera DHCP tak, by klient VPN był w
    stanie bez problemu odnowić lease i nie stracić połączenia ze swoim ISP.
  - `persist-key` oraz `persist-tun`-- mają za zadanie zachować interfejs jak i klucze podczas
    resetowania tunelu VPN. Te opcje są użyteczne zwykle przy zrzucaniu uprawnień via `user` i
    `group` .
  - `user` oraz `group` -- mają na celu zrzucenie uprawnień po wystartowaniu demona `openvpn` , co
    popoprawia bezpieczeństwo.
  - `status` , `log` oraz `log-append` -- odpowiadają za logowanie komunikatów do pliku. Standardowo
    wszystkie wiadomości są logowane przez syslog czy inny mechanizm logujący lub są wyrzucane na
    standardowe wyjście.
  - `verb` -- określa poziom logowania. Możliwe wartości od 0 do 11. W przypadku 0 logowanie zostaje
    wyłączone za wyjątkiem krytycznych błędów i ostrzeżeń.
  - `mute` -- ogranicza ilość logowanych wiadomości w przypadku, gdy są one takie same i następują
    jedna po drugiej.
  - `cipher` oraz `keysize` -- określają rodzaj algorytmu szyfrującego i długość klucza
    wykorzystywanego do zabezpieczenia ruchu w kanale danych. Dostępne konfiguracje szyfrów można
    uzyskać wydając polecenie `openvpn --show-ciphers` .
  - `auth` -- określa wykorzystywany hash przy uwierzytelnianiu pakietów za pomocą HMAC. Listę
    dostępnych algorytmów można uzyskać wpisując polecenie `openvpn --show-digests` .
  - `tls-version-min` -- umożliwia negocjowanie wersji protokołu TLS podczas nawiązywania połączenia
    i określa wersję minimalną, na którą muszą przystać obie strony, np. "1.0", "1.1" czy "1.2".
  - `tls-cipher` -- zestaw szyfrów wykorzystywanych do zaszyfrowania kanału kontrolnego. Ten
    wykorzystywany tutaj wymaga min. TLS 1.2 (pakiet `openvpn` w wersji 2.3.3 i wyższej). Dostępne
    szyfry można uzyskać wydając polecenie `openvpn --show-tls` .
  - `keepalive` -- zapewnia wiarygodność połączenia przez wysyłanie żądań ping na drugą stronę
    połączenia. W tym przypadku zostały użyte wartości `10 120` , co oznacza, że serwer będzie
    odpytywał klienta co 10 sekund i oczekiwał od niego odpowiedzi maksymalnie przez 240 sekund
    (2x120) zanim zdecyduje się zrestartować połączenie.
  - `comp-lzo` -- umożliwia kompresję danych wewnątrz zaszyfrowanego tunelu.
  - `max-clients` -- określa maksymalną ilość klientów, którzy mogą w danej chwili być jednocześnie
    połączeni z serwerem.
  - `ca` -- ścieżka do pliku z certyfikatem CA. Zawartość tego pliku można załączyć bezpośrednio w
    pliku konfiguracyjnym ujmując ją w znaczniki `<ca>...</ca>` .
  - `cert` -- ścieżka do pliku z certyfikatem serwera VPN. Zawartość tego pliku można załączyć
    bezpośrednio w pliku konfiguracyjnym ujmując ją w znaczniki `<cert>...</cert>` .
  - `key` -- ścieżka do pliku z kluczem prywatnym serwera VPN. Zawartość tego pliku można załączyć
    bezpośrednio w pliku konfiguracyjnym ujmując ją w znaczniki `<key>...</key>` .
  - `dh` -- ścieżka do pliku z parametrami Diffie-Hellman'a wykorzystywanymi przy tworzeniu
    tymczasowych kluczy sesyjnych przy zestawianiu połączenia między klientem a serwerem VPN.
  - `tls-auth` oraz `key-direction` -- umożliwiają skonfigurowanie dodatkowego mechanizmu obronnego
    w przypadku pakietów docierających do serwera VPN, co zabezpiecza przed atakami DoS. Zawartość
    pliku określonego w `tls-auth` można załączyć bezpośrednio w pliku konfiguracyjnym serwera
    ujmując ją w znaczniki `<tls-auth>...</tls-auth>` . W przypadku załączenia zawartości trzeba
    także określić kierunek klucza w `key-direction` . Zwykle się używa 0 dla serwera i 1 dla
    wszystkich klientów. Sam klucz zaś można wygenerować przez `openvpn --genkey --secret ta.key` .
  - `plugin` -- umożliwia wykorzystanie dodatkowych wtyczek, w tym przypadku mamy możliwość
    włączenia uwierzytelniania klientów VPN w oparciu o dane logowania do systemowych kont na
    serwerze przy udziale mechanizmu PAM. Jeśli chcemy zrezygnować zupełnie z certyfikatów
    klienckich, to dodatkowo załączamy opcję `client-cert-not-required` .
  - `duplicate-cn` -- ten parametr jest w stanie rozwiązać problemy w przypadku wykorzystywania tego
    samego certyfikatu na kilku klientach próbujących w danej chwili nawiązać połączenie z serwerem
    VPN.

### Forwarding pakietów

W naszym przypadku, oprogramowanie OpenVPN korzysta z wirtualnych interfejsów `tun` . Niemniej
jednak, pakiety muszą opuścić maszynę jednym z fizycznych interfejsów. Podobnie sprawa ma się w
drugą stronę, gdzie pakiety pierw docierają na fizyczny interfejs, a później są podawane na
wirtualne interfejsy stworzone przez demona `openvpn` . By taki zabieg był możliwy, musimy włączyć
na serwerze forwarding pakietów między tymi interfejsami. Możemy to zrobić dopisując poniższą
linijkę do pliku `/etc/sysctl.conf` :

    net.ipv4.ip_forward = 1

Zmiany aplikujemy wydając jeszcze w terminalu poniższe polecenie:

    # sysctl -p

### Konfiguracja firewall'a

Ostatnią rzeczą, którą musimy skonfigurować na serwerze VPN jest firewall. Przede wszystkim, w
łańcuchu INPUT tablicy filter otwieramy port na interfejsie fizycznym (w tym przypadku `eth0` ),
na którym ma nasłuchiwać demon `openvpn` . Musimy także w łańcuchu FORWARD przepuścić ruch między
interfejsami `eth0` i `tun0` . Dodatkowo każdy pakiet, który został stworzony przez demona `openvpn`
będzie miał inne źródło w stosunku do fizycznego interfejsu sieciowego na serwerze. Dlatego też
musimy to źródło przepisać stosując mechanizm translacji adresów (NAT). Poniżej są stosowne reguły
`iptables`:

    # iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
    # iptables -t filter -A FORWARD -s 10.8.0.0/24 -i tun0 -o eth0 -j ACCEPT
    # iptables -t filter -A FORWARD -d 10.8.0.0/24 -i eth0 -o tun0 -j ACCEPT
    # iptables -t filter -A INPUT -p udp -m udp --dport 1194 -m conntrack --ctstate NEW -m comment --comment vpn -j ACCEPT

### Test konfiguracji serwera VPN

Mając tak skonfigurowany serwer VPN możemy go uruchomić bezpośrednio z konsoli będąc, np.
zalogowanym po SSH. Powinniśmy ujrzeć całą masę komunikatów (w zależności od ustawionego numerka w
`verb`). Poniżej jest log z prawidłowo uruchomionego serwera VPN z `verb 3` :

    # openvpn /etc/openvpn/server-vpn.conf
    OpenVPN 2.3.4 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [EPOLL] [PKCS11] [MH] [IPv6] built on Nov 12 2015
    library versions: OpenSSL 1.0.2j  26 Sep 2016, LZO 2.08
    Diffie-Hellman initialized with 4096 bit key
    Control Channel Authentication: tls-auth using INLINE static key file
    Outgoing Control Channel Authentication: Using 512 bit message hash 'SHA512' for HMAC authentication
    Incoming Control Channel Authentication: Using 512 bit message hash 'SHA512' for HMAC authentication
    Socket Buffers: R=[212992->131072] S=[212992->131072]
    TUN/TAP device tun0 opened
    TUN/TAP TX queue length set to 100
    do_ifconfig, tt->ipv6=0, tt->did_ifconfig_ipv6_setup=0
    /sbin/ip link set dev tun0 up mtu 1500
    /sbin/ip addr add dev tun0 10.8.0.1/24 broadcast 10.8.0.255
    GID set to nogroup
    UID set to nobody
    UDPv4 link local (bound): [AF_INET]11.22.33.44:1194
    UDPv4 link remote: [undef]
    MULTI: multi_init called, r=256 v=256
    IFCONFIG POOL: base=10.8.0.2 size=252, ipv6=0
    ifconfig_pool_read(), in='client-vpn-morfik,10.8.0.4', TODO: IPv6
    succeeded -> ifconfig_pool_set()
    IFCONFIG POOL LIST
    client-vpn-morfik,10.8.0.4
    Initialization Sequence Completed

Serwer działa i nie ma żadnych błędów w logu. Gdyby pojawiły się jakieś, to trzeba zwiększyć poziom
`verb` na 4+ i przejrzeć komunikaty w poszukiwaniu wskazówek.

### Autostart serwera VPN

Obecnie w dystrybucji Debian, pakiet `openvpn` dostarcza usługę dla systemd, którą możemy sobie
dodać do autostartu systemu przez wpisanie w terminalu poniższego polecenia:

    # systemctl enable openvpn@server-vpn.service

Nazwa, która znajduje się za znakiem `@` odpowiada za plik konfiguracyjny, z tym, że bez końcówki
`.conf` .

Do tak postawionego serwera VPN możemy się już podłączyć. Niemniej jednak, musimy jeszcze
odpowiednio skonfigurować sobie klientów.

## Konfiguracja klienta VPN

Konfigurując klienta VPN trzeba wziąć pod uwagę fakt, że nie zawsze w grę będą wchodzić komputery
stacjonarne czy laptopy mające zainstalowane systemy typu linux/windows. Być może będziemy chcieli
nawiązać bezpieczne połączenie z naszego smartfona albo też podpiąć całą sieć domową przesyłając
ruch sieciowy ze wszystkich maszyn bezpośrednio z routera do serwera VPN. W niniejszym artykule
opiszę jedynie konfigurację linux'owego klienta.

Podobnie jak w przypadku serwera VPN, w paczce `openvpn` jest zawarty plik z przykładową
konfiguracją maszyn klienckich. Interesuje nas plik
`/usr/share/doc/openvpn/examples/sample-config-files/client.conf` , który kopiujemy do katalogu
`/etc/openvpn/`
    :

    # cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client-vpn.conf

Przy konfigurowaniu klientów ogromną rolę mają parametry uwzględnione w konfiguracji serwera. Dla
przykładu jeśli serwer ma zamiar kompresować szyfrowany ruch przed przesłaniem go do klientów, to
opcja kompresji również musi znaleźć się w pliku konfiguracyjnym klienta VPN. Podobnie sprawa ma się
z rodzajem wykorzystywanych szyfrów/hashy i sporej części innych opcji. Jako, że wyżej
wykorzystywaliśmy dość rozbudowany plik dla serwera VPN, to poniżej jest odpowiednik pliku, który
trzeba wgrać na klienta:

    client
    dev tun
    proto udp
    remote 11.22.33.44 1194
    resolv-retry infinite
    nobind
    ;user nobody
    ;group nogroup
    persist-key
    persist-tun
    cipher AES-256-CBC
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
    auth SHA512
    keysize 256
    tls-version-min 1.2
    comp-lzo
    verb 3
    ;mute 20
    script-security 2
    up /etc/openvpn/update-resolv-conf.sh
    down /etc/openvpn/update-resolv-conf.sh

    auth-nocache
    #auth-user-pass /etc/openvpn/auth/morfitronik.auth

    remote-cert-tls server
    verify-x509-name "server-vpn" name

    ;ca /etc/openvpn/certs/ca.crt
    ;cert /etc/openvpn/certs/client-vpn-morfik.crt
    ;key /etc/openvpn/certs/client-vpn-morfik.key
    ;tls-auth /etc/openvpn/certs/ta.key 1
    key-direction 1

    <ca>
    ..
    </ca>
    <cert>
    ...
    </cert>
    <key>
    ...
    </key>
    <tls-auth>
    ...
    </tls-auth>

Część opcji wykorzystanych wyżej jest dokładnie taka sama co w przypadku pliku konfiguracyjnego
serwera VPN. Dlatego też pominę ich opis i skupię się jedynie na tych parametrach, które są
specyficzne jedynie dla klienta VPN.

  - `client` -- każdy klient musi mięć taką opcję w swoim piku konfiguracyjnym, by demon `openvpn`
    wiedział, że ma do czynienia z klientem, a nie serwerem VPN.
  - `dev` -- rodzaj wykorzystywanych interfejsów musi pasować do tych, które zostały określone na
    serwerze. Nie można ich mieszać, tj. zastosować na serwerze `tun` , a na kliencie `tap` i vice
    versa.
  - `remote` -- określa adres i port serwera VPN, z którym zamierzamy się połączyć.
  - `resolv-retry` -- umożliwia ponowne rozwiązanie nazwy domeny zdefiniowanej w `remote` na wypadek
    ewentualnych problemów przy odpytywaniu serwera DNS. Można tutaj także określić liczbę, którą
    wyraża czas w sekundach.
  - `cipher` , `auth` oraz `keysize` -- te opcje muszą pasować do tych zdefiniowanych na serwerze.
  - `comp-lzo` -- ta opcja również musi pasować do tej określonej na serwerze.
  - `script-security` -- umożliwia wykonywanie skryptów. Im wyższy poziom (0-3), tym mniej
    restrykcyjny jest mechanizm obronny. W tym przypadku 2 daje nam możliwość załadowania
    zewnętrznego skryptu użytkownika, który dostosowuje reguły zapory, by [zapobiec min. przeciekom
    DNS (DNS leak)](/post/przeciek-dns-dns-leak-w-vpn-resolvconf/).
  - `auth-user-pass` oraz `auth-nocache` -- te dwa parametry mają znaczenie przy uwierzytelnianiu
    klienta VPN z wykorzystaniem loginu i hasła, które są odczytywane z pliku. Obecność opcji
    `auth-nocache` sprawia, że demon `openvpn` zapomni dane logowania po ustanowieniu połączenia i
    pozbędzie się ich z pamięci operacyjnej.
  - `tls-auth` oraz `key-direction` -- te dwie opcje w przypadku klienta są podobne do tych, które
    mieliśmy na serwerze. O ile klucz jest współdzielony, to kierunek trzeba zmienić z 0 na 1.
  - `remote-cert-tls` -- wymaga od drugiej strony połączenia przedstawienia certyfikatu z
    określonymi opcjami (key usage oraz extended key usage), które są wykorzystywane jedynie w
    przypadku certyfikatów serwerowych. Taki certyfikat można wygenerować przy pomocy skryptu
    `build-key-server` dostarczanego z pakietem `easy-rsa` .
  - `verify-x509-name` -- ten parametr jest w stanie zweryfikować określone pola w certyfikacie
    serwera. Domyślnie jest wykorzystywane CN (Common Name), które również jest ustawiane podczas
    generowania certyfikatu serwera via `easy-rsa` .
  - `nobind` -- ta opcja sprawia, że demon `openvpn` nie będzie nasłuchiwał na porcie określonym w
    `port` . Zamiast tego zostanie użyty jeden z portów nieuprzywilejowanych, tj. \> 1024.

### Test połączenia klienta z serwerem VPN

Jako, że właśnie napisaliśmy konfigurację dla klienta, to możemy spróbować podłączyć się do już
działającego serwera VPN. Poniżej jest log obrazujący proces łączenia przy `verb 3` :

    # openvpn /etc/openvpn/morfitronik.conf
    OpenVPN 2.3.11 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [EPOLL] [PKCS11] [MH] [IPv6] built on May 23 2016
    library versions: OpenSSL 1.0.2j  26 Sep 2016, LZO 2.08
    NOTE: the current --script-security setting may allow this configuration to call user-defined scripts
    Control Channel Authentication: tls-auth using INLINE static key file
    Outgoing Control Channel Authentication: Using 512 bit message hash 'SHA512' for HMAC authentication
    Incoming Control Channel Authentication: Using 512 bit message hash 'SHA512' for HMAC authentication
    Socket Buffers: R=[212992->212992] S=[212992->212992]
    UDPv4 link local: [undef]
    UDPv4 link remote: [AF_INET]11.22.33.44:1194
    TLS: Initial packet from [AF_INET]11.22.33.44:1194, sid=bfd79a16 18fa4caa
    VERIFY OK: depth=1, C=PL, ST=Mazowieckie, L=Warsaw, O=Morfitronik, OU=Auth, CN=morfitronik.lh, name=Morfitronik, emailAddress=morfik@nsa.com
    Validating certificate key usage
    ++ Certificate has key usage  00a0, expects 00a0
    VERIFY KU OK
    Validating certificate extended key usage
    ++ Certificate has EKU (str) TLS Web Server Authentication, expects TLS Web Server Authentication
    VERIFY EKU OK
    VERIFY X509NAME OK: C=PL, ST=Mazowieckie, L=Warsaw, O=Morfitronik, OU=Auth, CN=server-vpn, name=Morfitronik, emailAddress=morfik@nsa.com
    VERIFY OK: depth=0, C=PL, ST=Mazowieckie, L=Warsaw, O=Morfitronik, OU=Auth, CN=server-vpn, name=Morfitronik, emailAddress=morfik@nsa.com
    Data Channel Encrypt: Cipher 'AES-256-CBC' initialized with 256 bit key
    Data Channel Encrypt: Using 512 bit message hash 'SHA512' for HMAC authentication
    Data Channel Decrypt: Cipher 'AES-256-CBC' initialized with 256 bit key
    Data Channel Decrypt: Using 512 bit message hash 'SHA512' for HMAC authentication
    Control Channel: TLSv1.2, cipher TLSv1/SSLv3 DHE-RSA-AES256-GCM-SHA384, 4096 bit RSA
    [server-vpn] Peer Connection Initiated with [AF_INET]11.22.33.44:1194
    SENT CONTROL [server-vpn]: 'PUSH_REQUEST' (status=1)
    PUSH: Received control message: 'PUSH_REPLY,redirect-gateway def1 bypass-dhcp,dhcp-option DNS 208.67.222.222,dhcp-option DNS 208.67.220.220,route-gateway 10.8.0.1,topology subnet,ping 10,ping-restart 120,ifconfig 10.8.0.4 255.255.255.0'
    OPTIONS IMPORT: timers and/or timeouts modified
    OPTIONS IMPORT: --ifconfig/up options modified
    OPTIONS IMPORT: route options modified
    OPTIONS IMPORT: route-related options modified
    OPTIONS IMPORT: --ip-win32 and/or --dhcp-option options modified
    ROUTE_GATEWAY 192.168.1.1/255.255.255.0 IFACE=bond0 HWADDR=3c:4a:92:00:4c:5b
    TUN/TAP device tun0 opened
    TUN/TAP TX queue length set to 100
    do_ifconfig, tt->ipv6=0, tt->did_ifconfig_ipv6_setup=0
    /sbin/ip link set dev tun0 up mtu 1500
    /sbin/ip addr add dev tun0 10.8.0.4/24 broadcast 10.8.0.255
    /etc/openvpn/update-resolv-conf.sh tun0 1500 1602 10.8.0.4 255.255.255.0 init
    dhcp-option DNS 208.67.222.222
    dhcp-option DNS 208.67.220.220
    /sbin/ip route add 11.22.33.44/32 via 192.168.1.1
    /sbin/ip route add 0.0.0.0/1 via 10.8.0.1
    /sbin/ip route add 128.0.0.0/1 via 10.8.0.1
    Initialization Sequence Completed

Od tego momentu cały ruch na kliencie jest wpuszczany w szyfrowany tunel i przesyłany do serwera
VPN.

## Problemy z połączeniem

Taka konfiguracja powinna działać bez większego problemu. Niemniej jednak, zwykle reguły `iptables`
mogą powodować problemy z przepływem pakietów przez firewall. Jeśli za jakiegoś powodu nie działa
nam połączenie przez VPN i nic ciekawego w logu po stronie serwera i klienta nie zaobserwowaliśmy,
to najlepszym miejscem na poszukiwanie przyczyny problemu jest `ping` na lokalne końcówki tunelu,
tj. te mające adres z puli 10.8.0.0/24 . Jeśli `ping` nie lata, to gdzieś zapora blokuje ruch między
klientem i serwerem. Jeśli zaś jesteśmy w stanie pingnąć końcówki tunelu ale nie możemy przeglądać
stron www w przeglądarce, to problem tkwi najprawdopodobniej w forwarding'u pakietów. Inne problemy
mogą wynikać z niespójnych plików konfiguracyjnych, tj. innej konfiguracji na kliencie i innej na
serwerze ale tego typu problemy powinny być widoczne już w logu przy `verb 4` .

Warto też zaznaczyć, że by połączenie VPN między klientem, a serwerem zostało zrealizowane
poprawnie, czas na obu maszynach musi być w miarę dokładnie zsynchronizowany. Dlatego też w razie
problemów zsynchronizujmy sobie czas z serwerami czasu NTP na obu maszynach.
