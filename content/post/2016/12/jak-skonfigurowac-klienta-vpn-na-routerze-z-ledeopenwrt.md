---
author: Morfik
categories:
- LEDE
date: "2016-12-08T18:13:41Z"
date_gmt: 2016-12-08 17:13:41 +0100
published: true
status: publish
tags:
- prywatność
- vpn
- router
- tp-link
- chaos-calmer
title: Jak skonfigurować klienta VPN na routerze z LEDE/OpenWRT
---

Ostatnio pisałem trochę o [konfiguracji serwera VPN na
Debianie]({{< baseurl >}}/post/jak-skonfigurowac-serwer-vpn-na-debianie-openvpn/) oraz podłączaniu
do niego różnych linux'owych klientów, w tym też [smartfonów wyposażonych w system
Android]({{< baseurl >}}/post/jak-skonfigurowac-polaczenie-vpn-na-smartfonie-z-androidem/). O ile
konfiguracja pojedynczego klienta OpenVPN nie jest jakoś szczególnie trudna, to mając w swojej sieci
domowej kilka urządzeń zdolnych łączyć się z internetem zarówno przewodowo jak i bezprzewodowo, to
dostosowanie konfiguracji na każdym z tych sprzętów może być ździebko problematyczne. To co łączy te
wszystkie urządzenia w naszym domu, to router WiFi. Zwykle każdy komputer, nawet ten najmniejszy,
łączy się z takim routerem w celu nawiązania połączenia ze światem. Dlatego też zamiast
konfigurować osobno wszystkie te urządzenia elektroniczne, możemy skonfigurować sobie router w taki
sposób, by cały zebrany ruch z sieci lokalnej przesłał do serwera VPN. Standardowej klasy routery
nie wspierają połączeń VPN i by taki mechanizm zaimplementować potrzebne nam będzie alternatywne
firmware pokroju LEDE/OpenWRT. W tym artykule postaramy się skonfigurować połączenie VPN dla sieci
domowej w oparciu o [router Archer
C2600](http://www.tp-link.com.pl/products/details/Archer-C2600.html) od TP-LINK, który ma wgrany
najnowszy snapshot LEDE Chaos Calmer (r2392).

<!--more-->
## Niezbędne oprogramowanie na potrzeby VPN

Nie da się ukryć, że oprogramowanie, które umożliwi nam nawiązanie połączenia VPN, trochę waży.
Trzeba także brać pod uwagę fakt szyfrowania całego ruchu, co też obciąży mocno procesor routera. Te
dwie rzeczy nie stanowią aż takiego problemu w przypadku Archer'a C2600, bo on ma flash 32 MiB i dwa
rdzenie taktowane 1,2 GHz:

![]({{< baseurl >}}/img/2016/12/001-openvpn-vpn-lede-openwrt-router-tp-link.png#huge)

Na innych routerach, które mają mniejszy flash i słabszy procesor, OpenVPN może sprawić drobne
problemy, zwłaszcza przy mocniejszych szyfrach. O ile w przypadku wolnego miejsca na flash możemy
coś poradzić, np przeprowadzić extroot, o tyle w przypadku procesora zbytnio nic nie zrobimy i taki
dodatkowy narzut (overhead) odbije się spowolnieniem transmisji danych.

Z tego co zauważyłem w LEDE mamy z grubsza trzy pakiety, które mogą nam posłużyć do skonfigurowania
klienta VPN, są to: `openvpn-nossl` , `openvpn-openssl` oraz `openvpn-polarssl` . Ten pierwszy
pakiet możemy sobie darować, bo nie zapewnia on wsparcia dla szyfrowanego połączenia z VPN. Musimy
zdecydować się na jeden z dwóch pozostałych pakietów. Generalnie rzecz biorąc [PolarSSL ma niby
zastąpić OpenSSL](https://tls.mbed.org/openssl-alternative) ale znowu [OpenVPN ma pewne
ograniczenia w przypadku wykorzystywania
PolarSSL](https://community.openvpn.net/openvpn/wiki/UsingPolarSSL). Dlatego też zainstalujemy sobie
`openvpn-openssl` , z tym, że nasz router musi mieć minimum 1 MiB wolnego miejsca na flash'u.
Logujemy się zatem na router i wydajemy w terminalu poniższe polecenia:

    # opkg update
    # opkg install openvpn-openssl

## Konfiguracja klienta VPN na LEDE/OpenWRT

Z pakietem `openvpn-openssl` został dostarczony skrypt startowy zlokalizowany w
`/etc/init.d/openvpn` i jeśli chcemy, aby klient VPN łączył się automatycznie z serwerem po starcie
routera, to naturalnie przydałoby się dodać ten skrypt do autostartu w poniższy sposób:

    # /etc/init.d/openvpn enable

Konfiguracja dla demona `openvpn` jest przechowywana w pliku `/etc/config/openvpn` i musimy ją sobie
dostosować w oparciu o konfigurację serwera VPN, z którym zamierzamy się połączyć. Ten plik
konfiguracyjny jest dość rozbudowany i ma około 400 linijek. Większość z nich to komentarze.

Plik `/etc/config/openvpn` jest podzielony na trzy sekcje. Pierwsza z nich dotyczy konfiguracji
klienta VPN za pomocą zewnętrznego pliku. Druga dotyczy konfiguracji serwera VPN na routerze (nas to
zbytnio nie interesuje). Trzecia sekcja zaś umożliwia skonfigurowanie klienta VPN za pomocą
interfejsu UCI.

### Konfiguracja OpenVPN w zewnętrznym pliku

W przypadku, gdy skonfigurowaliśmy sobie połączenie z serwerem VPN na komputerze, to powinniśmy
dysponować stosownym plikiem konfiguracyjnym, który możemy wgrać na router. Następnie w pliku
`/etc/config/openvpn` musimy ustawić jedynie poniższe parametry:

    ...
    config openvpn custom_config
        option enabled 1
        option config /etc/openvpn/morfitronik.conf
    ...

W pliku `/etc/openvpn/morfitronik.conf` umieszczamy konfigurację dla klienta OpenVPN:

    client
    dev tun
    proto udp
    remote 11.22.33.44 1194
    resolv-retry infinite
    nobind
    user nobody
    group nogroup
    persist-key
    persist-tun
    cipher AES-256-CBC
    tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
    auth SHA512
    keysize 256
    tls-version-min 1.2
    comp-lzo
    verb 3
    auth-nocache

    remote-cert-tls server
    verify-x509-name "morfitronik-server-vpn" name

    key-direction 1

    <ca>
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
    </ca>
    <cert>
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
    </cert>
    <key>
    -----BEGIN PRIVATE KEY-----
    ...
    -----END PRIVATE KEY-----
    </key>
    <tls-auth>
    -----BEGIN OpenVPN Static key V1-----
    ....
    -----END OpenVPN Static key V1-----
    </tls-auth>

Po wyjaśnienie użytych tutaj opcji odsyłam do wpisu podlinkowanego we wstępie.

### Konfiguracja OpenVPN przez UCI

Alternatywną metodą konfiguracji klienta OpenVPN na LEDE/OpenWRT jest skorzystanie z interfejsu UCI.
W pliku `/etc/config/openvpn` na samym dole mamy odpowiednią sekcję. Włączamy ją przestawiając
poniższy parametr:

    ...
    config openvpn sample_client
        option enabled 1
    ...

Dalej musimy uwzględnić wszystkie opcje, które zwykle podaje się w pliku konfiguracyjnym OpenVPN, z
tym, że musimy je poprzedzić słówkiem `option` . Poniżej przykład:

    ...
    option client 1
    option dev tun
    option proto udp
    list remote "11.22.33.44 1194"
    option resolv_retry infinite
    option nobind 1
    option persist_key 1
    option persist_tun 1
    option user nobody
    option ca /etc/openvpn/certs/morfitronik-ca.crt
    option cert /etc/openvpn/certs/morfitronik-client-vpn-router-c2600.crt
    option key /etc/openvpn/certs/morfitronik-client-vpn-router-c2600.key
    option ns_cert_type server
    option tls_auth "/etc/openvpn/certs/morfitronik-ta.key 1"
    option cipher AES-256-CBC
    option tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
    option auth SHA512
    option keysize 256
    option tls-version-min 1.2
    option remote-cert-tls
    option verify-x509-name "morfitronik-server-vpn name"
    option comp_lzo yes
    option verb 4

Problem z UCI jest taki, że klucze/certyfikaty klienta i serwera trzeba niestety umieszczać w
osobnych plikach.

## Certyfikaty klienckie

Ja jestem zwolennikiem uwierzytelniania użytkowników za pomocą certyfikatów klienckich. Dlatego też
mam w taki sposób skonfigurowany swój serwer VPN. Bez przedstawienia takiego certyfikatu, żaden
klient nie będzie w stanie zestawić połączenia. Opis [jak wygenerować takie certyfikaty w oparciu o
easy-rsa]({{< baseurl >}}/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/) został opisany
osobno. Tutaj warto nadmienić, że w LEDE/OpenWRT mamy dostępny pakiet `openvpn-easy-rsa` , którym
możemy się posłużyć przy generowaniu certyfikatów. Niemniej jednak, to rozwiązanie nie jest zbytnio
zalecane ze względu na niewielką moc obliczeniową takiego przeciętnego routera WiFi, przez cały
proces będzie trwał kilka godzin.

## Konfiguracja firewall'a na potrzeby OpenVPN

W zasadzie bez znaczenia jest sposób, który sobie wybierzemy w celu dostarczenia konfiguracji dla
demona OpenVPN. Jeśli w tej chwili byśmy uruchomili usługę, to router będzie w stanie przesłać
wszystkie swoje dane bezpośrednio do serwera VPN. Problem jednak jest w tym, że klienci w sieci LAN
za routerem nie mają połączenia z internetem. Możemy co prawda połączyć się z routerem ale nie mamy
wyjścia na świat. Problem tkwi w konfiguracji zapory. Musimy zatem skonfigurować sobie forwarding
dla nowych interfejsów i włączyć im również NAT. W tym celu trzeba poddać edycji dwa pliki i dodać
do nich poniższą zawartość.

Plik `/etc/config/network` :

    config interface 'vpn'
        option ifname 'tun0'
        option proto 'none'

Plik `/etc/config/firewall` :

    config zone
        option name             vpn
        list   network          'vpn'
        option input            DROP
        option output           ACCEPT
        option forward          DROP
        option masq             1
        option mtu_fix          1

    config forwarding
        option src              'lan'
        option dest             'vpn'

Zapisujemy i resetujemy router. I to w zasadzie cała konfiguracja jeśli chodzi o przesyłanie ruchu
ze wszystkich urządzeń w sieci domowej WiFi do serwera VPN. Jeśli zaś chodzi o ewentualne [przecieki
DNS]({{< baseurl >}}/post/przeciek-dns-dns-leak-w-vpn-resolvconf/), to zawsze możemy skonfigurować
sobie [szyfrowany DNS w oparciu o
dnscrypt-proxy]({{< baseurl >}}/post/konfiguracja-dnscrypt-proxy-w-openwrt/) również na routerze.

## Test połączenia z serwerem VPN

W zasadzie połączenie powinno działać OOTB po zresetowaniu routera. Jeśli jednak nie mamy pewności
do tego czy cały mechanizm został poprawnie skonfigurowany, to naturalnie możemy zajrzeć w log (
`logread` ) i poszukać wpisów zaczynających się od `openvpn` . Jeśli nie znajdziemy żadnych błędów i
będzie w logu figurować `Initialization Sequence Completed` , to oznacza to, że połączenie z
serwerem VPN zostało pomyślnie ustawione. Jeśli zaś mamy w logu również `PUSH_REPLY,redirect-gateway
def1` , to cały ruch z routera wędruje do serwera VPN w formie szyfrowanej. Dla pewności można także
podejrzeć regułki na zaporze ( `iptables -nvL` ) i poszukać interfejsu `tun0` . Jeśli pakiety są
zliczane, to ruch jest zarządzany przez OpenVPN. No i oczywiście możemy podejrzeć tablicę routingu
via `ip route show` , gdzie obecność tras `0.0.0.0/1` oraz `128.0.0.0/1` oznacza, że żaden pakiet
wysyłany do internetu nie leci poza szyfrowanym tunelem.
