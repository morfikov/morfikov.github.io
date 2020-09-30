---
author: Morfik
categories:
- OpenWRT
date: "2016-06-15T07:17:07Z"
date_gmt: 2016-06-15 05:17:07 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- sieć
- router
- radius
title: Router OpenWRT jako serwer i klient RADIUS
---

Ten poniższy artykuł ma na celu pokazanie w jaki sposób stworzyć infrastrukturę WiFi w oparciu o
oprogramowanie `freeradius` (serwer RADIUS), które zostanie zainstalowane na przykładowym routerze
[TP-Link Archer C7 v2](http://www.tp-link.com.pl/products/details/Archer-C7.html). Router ma wgrany
firmware OpenWRT Chaos Calmer w wersji 15.05.1 (r49294). Zostanie tutaj opisane dokładnie jak
wdrożyć protokół WPA2 Enterprise z obsługą trzech metod uwierzytelniania: EAP-TLS, EAP-TTLS oraz
PEAP (v0) .

<!--more-->
## Instalacja pakietów na potrzeby RADIUS'a

Jakiś czas temu opisywałem wdrożenie [WPA2 Enterprise w oparciu o oprogramowanie
freeradius](/post/wpa2-enterprise-serwer-freeradius/), z tym, że na debianie.
Tamten mechanizm był podobny do tego, który zamierzam przedstawić w tym artykule. W podlinkowanym
wyżej wpisie był robiony użytek z dwóch maszyn. Jedna z nich robiła za serwer RADIUS'a, druga zaś
za klienta. W tym przypadku zarówno klientem jak i serwerem RADIUS'a będzie nasz router. Logujemy
się zatem na to urządzenie i przechodzimy do instalacji potrzebnych nam pakietów. Na te poniższe
rzeczy potrzebne jest około 1,5 MiB na flash'u routera. Trzeba jednak przy tym zaznaczyć, że nie są
to wszystkie moduły, które możemy zainstalować oraz, że z części z nich możemy zrezygnować.

    # opkg update
    # opkg install freeradius2 \
    freeradius2-mod-eap \
    freeradius2-mod-eap-mschapv2 \
    freeradius2-mod-eap-peap \
    freeradius2-mod-eap-tls \
    freeradius2-mod-eap-ttls \
    freeradius2-mod-files \
    freeradius2-mod-mschap \
    freeradius2-mod-radutmp \
    freeradius2-mod-realm \
    freeradius2-mod-exec \
    freeradius2-mod-pap \
    freeradius2-mod-chap \
    freeradius2-mod-sql \
    freeradius2-mod-logintime

Musimy także wymienić nieco oprogramowania. Mowa o domyślnie zainstalowanym pakiecie `wpad-mini` ,
który nie zawiera obsługi WPA2 Enterprise. Jeśli chcemy mieć obsługę tego protokołu, to musimy
doinstalować pakiet `wpad` uprzednio usuwając wspomniany wyżej pakiet `wpad-mini` (potrzeba około
0,5 MiB na flash):

    # opkg remove wpad-mini
    # opkg install wpad

## Certyfikaty

Serwer RADIUS, który zamierzamy skonfigurować, wymaga do prawidłowej pracy kilku certyfikatów. Te
certyfikaty musimy sobie stworzyć. Nie będę tutaj opisywał procesu generowania certyfikatów, a to z
tego względu, że zostało to zrobione w osobnych artykułach. Odradzam tworzenie samych certyfikatów
bezpośrednio na routerze, bo to jest proces bardzo zasobożerny i dużo lepiej do tego celu jest
wykorzystać jakąś standardową dystrybucję linux'a. Niemniej jednak, jeśli nie mamy jeszcze
wymaganych certyfikatów, to możemy je sobie stworzyć przy pomocy takich narzędzi jak, np.
[easy-rsa](/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/). Oczywiście to
tylko jeden ze sposobów generowania certyfikatów. Jeśli nie chce nam się korzystać z narzędzi
specjalnie przeznaczonych do tego typu celów, to zawsze możemy takie [certyfikaty wygenerować sobie
ręcznie](/post/generowanie-certyfikatow/).

Zakładam, że mamy już potrzebne pliki. Musimy je wgrać na router. Potrzebny nam będzie klucz
serwera, certyfikat serwera oraz certyfikat CA. Dodatkowo, będziemy potrzebować plik `dh` (parametry
potrzebne do wymiany kluczy w protokole Diffie-Hellman'a). Tworzymy zatem na routerze katalog
`/etc/freeradius2/certs/` i wrzucamy do niego wszystkie wyżej wymienione pliki przy pomocy `scp`
    :

    # scp /etc/CA_RADIUS/keys/{radius.key,radius.crt,ca.crt,dh2048.pem} root@192.168.1.1:/etc/freeradius2/certs

## Konfiguracja demona radiusd

Na sam początek edytujemy plik `/etc/freeradius2/radiusd.conf` . Musimy w nim dostosować odpowiednie
parametry dotyczące interfejsu, adresu i portów na jakich serwer RADIUS będzie operował:

    ...
    listen {
        type = auth
        ipaddr = 127.0.0.1
        port = 0
        interface = lo
    ...
    listen {
        type = acct
        ipaddr = 127.0.0.1
        port = 0
        interface = lo
    ...

Port `0` w tym wypadku oznacza port domyślny. Wartość tego portu jest odczytywana z pliku
`/etc/services` . Standardowo są to `1812` (auth) i `1813` (acct).

## Konfiguracja klienta RADIUS'a

W tym przypadku router będzie robił zarówno za klient jak i za serwer RADIUS'a. Musimy zatem
odpowiednio zmienić plik `/etc/freeradius2/clients.conf` . To w tym pliku określamy hosty, które
mogą przesyłać zapytania do serwera RADIUS. Domyślna jego konfiguracja już wskazuje na adres
`127.0.0.1` . Jedyne co musimy zrobić, to określić parametr `secret` :

    ...
    client localhost {
        ipaddr = 127.0.0.1
        secret = 1846445a84caa742f082efed78e786d9f
        require_message_authenticator = yes
        nastype = other
    }
    ...

Musimy także odpowiednio przepisać plik `/etc/config/wireless` , gdzie ustawiamy te poniższe opcje:

    config wifi-iface
        option device     'radio0'
        option network    'lan'
        option mode       'ap'
        option ssid       'Ever_Vigilant'
        option encryption 'wpa2+aes'
        option key        '1846445a84caa742f082efed78e786d9f'
        option server     '127.0.0.1'
        option port       '1812'
        option disabled   '0'
        option hidden     '0'

Jeśli ktoś jest ciekaw użyty wyżej opcji, to zapraszam do zapoznania się z artykułem, który został
poświęcony [konfiguracji sieci WiFi na routerze z
OpenWRT](/post/siec-bezprzewodowa-wifi-w-openwrt-wlan/). To na co musimy zwrócić
uwagę, to opcja `key` , której wartość wskazuje na sekret określony wyżej w pliku
`/etc/freeradius2/clients.conf` . Te dwie wartości muszą do siebie pasować. Uzupełniamy też opcje
`server` oraz `port` . Dodatkowo, jeśli korzystaliśmy wcześniej z sieci opartej o protokół WPA2-PSK,
to przydałoby się ją wyłączyć (za pomocą `disabled` ) zostawiając w powyższym pliku tylko jeden
aktywny blok z konfiguracją sieci WiFi.

## Klienci sieci WiFi

Jeśli chodzi zaś o konta użytkowników, którzy będą mieli prawo łączyć się do naszej sieci WiFi, to
definiujemy ich w pliku `/etc/freeradius2/users` . Poniżej są zawarte przykładowe wpisy dla kont dla
konfiguracji EAP-TTLS i PEAP:

    "anonymous"       Auth-type := EAP

    "morfik"          Cleartext-Password := "e6a664c54f6427"
                      Reply-Message = "Witaj, %{User-Name}"

Niżej zaś jest konfiguracja dla kont EAP-TLS:

    "morfik_laptop"   Auth-type := EAP
                      Reply-Message = "Witaj, Morfiku!"

Musimy także dopisać jedną dodatkową regułę, która będzie definiować domyślną akcję. To właśnie ta
akcja zostanie podjęta jeśli żądanie nie zostanie dopasowane do jednego z tych powyżej określonych
kont. Dodajemy zatem ten poniższy wpis:

    DEFAULT           Auth-type := Reject
                      Reply-Message := "SPADAJ NA DRZEWO!"

## Konfiguracja uwierzytelniania EAP (EAP-TLS, EAP-TTLS, PEAP)

[Jest wiele metod uwierzytelniania
EAP](http://www.networkworld.com/article/2223672/access-control/which-eap-types-do-you-need-for-which-identity-projects.html)
i nas interesować będą trzy z nich EAP-TLS, EAP-TTLS i PEAP. Metoda `EAP-TLS`, w odróżnieniu od
EAP-TTLS i PEAP, wymaga wzajemnej weryfikacji certyfikatów na linii serwer-klient. Z kolei
`EAP-TTLS` i `PEAP` zbytnio się nie różnią między sobą, no może za wyjątkiem wsparcia, bo ten drugi
jest rozwijany przez MS. Obie te metody wykorzystują szyfrowany tunel TLS, który jest również
podstawą dla uwierzytelniania przy metodzie EAP-TLS i by korzystać z EAP-TTLS lub PEAP, trzeba po
części skonfigurować uwierzytelnianie EAP-TLS. Same certyfikaty klienckie w przypadku EAP-TTLS i
PEAP są nieobowiązkowe i gdy nie korzystamy z nich, uwierzytelnianie klienta odbywa się przy pomocy
loginu i hasła, co znacznie upraszcza wdrożenie protokołu WPA2 Enterprise. Jeśli chodzi o samą
konfigurację klientów, to windowsy standardowo nie działają z metodą EAP-TTLS i muszą używać PEAP,
co trzeba wziąć pod uwagę, przy konfiguracji serwera RADIUS.

W związku z powyższym, ustawimy domyślny protokół EAP na PEAP, zakładając, że najwięcej klientów
będzie właśnie go używać. Konfiguracja uwierzytelniania EAP jest trzymana w pliku
`/etc/freeradius2/eap.conf` . Edytujmy go zatem i ustawmy odpowiednie ścieżki do certyfikatów. W
zależności od tego czy wybieraliśmy opcje szyfrowania plików certyfikatów przy ich tworzeniu, to
będziemy musieli także określić hasło do certyfikatów. W tym przypadku, wszystkie certyfikaty są
bez haseł, zatem pole od hasła w konfiguracji freeradius'a pozostawiamy puste:

    tls {
    ...
        private_key_password =
        private_key_file = ${certdir}/radius.key
        certificate_file = ${certdir}/radius.crt
        CA_file = ${cadir}/ca.crt
        dh_file = ${certdir}/dh2048.pem
    ...

Mając skonfigurowany tunel TLS, możemy skonfigurować także protokół EAP-TTLS:

    ...
    ttls {
        default_eap_type = md5
        copy_request_to_tunnel = yes
        use_tunneled_reply = yes
    }
    ...

Podobnie postępujemy w przypadku protokołu PEAP:

    ..
    peap {
       default_eap_type = mschapv2
       copy_request_to_tunnel = yes
       use_tunneled_reply = yes
    }
    ...

## Test serwera RADIUS

Przydałoby się wstępnie przetestować serwer RADIUS'a odpalając go w trybie `debug` przy pomocy tego
poniższego polecenia:

    # radiusd -XXX

W terminalu powinna pokazać nam się cała masa wiadomości. Na końcu zaś powinniśmy zobaczyć wpisy
podobne do tych poniżej:

    Debug: Listening on authentication interface lo address 127.0.0.1 port 1812
    Debug: Listening on accounting interface lo address 127.0.0.1 port 1813
    Info: Ready to process requests.

Jeśli z jakiegoś powodu dostaniemy błąd `radiusd: can't resolve symbol 'EC_KEY_new_by_curve_name'` ,
to musimy sprawdzić jaką wersję OpenSSL mamy zainstalowaną na routerze oraz w oparciu o jaką wersję
był kompilowany pakiet `freeradius2` . Poniżej przykład:

    # opkg list-installed | grep openssl
    libopenssl - 1.0.2g-1

    # radiusd -XXX
    ...
    Debug:   ssl: OpenSSL 1.0.2g  1 Mar 2016

Jeśli numerki w obu przypadkach się nie pokrywają, to jest wielce prawdopodobne, że nie uda nam się
odpalić serwera RADIUS. Jeśli mimo posiadania tych samych wersji nadal dostajemy w/w błąd, to być
może pakiet `libopenssl` pochodzi z innego źródła niż pakiet `freeradius` . W tym przypadku
wszystko jest w należytym porządku.

## Test podłączania klientów do sieci WiFi

Serwer RADIUS'a stoi już na routerze i teraz trzeba zaprzęgnąć do pracy jakąś maszynę kliencką w
celu przetestowania połączenia. Jako, że mamy skonfigurowaną obsługę trzech metod
uwierzytelniających, to potrzebne nam są odpowiednie profile dla klientów sieci WiFi, by ci mogli
się bez problemu podłączyć. Poniżej są 3 zwrotki dla linux'owego narzędzia `wpasupplicant`
zajmującego się konfiguracją połączeń bezprzewodowych.

Konfiguracja dla EAP-TLS

    network={
        id_str="home_wifi_static"
        priority=10
        ssid="Winter_Is_Coming"
        bssid=E8:94:F6:68:79:EF
        proto=RSN
        key_mgmt=WPA-EAP
        eap=TLS
        pairwise=CCMP
        group=CCMP
        auth_alg=OPEN
        identity="morfik_laptop"
        ca_cert="/etc/CA_RADIUS/keys/ca.crt"
        client_cert="/etc/CA_RADIUS/keys/morfik_laptop.crt"
        private_key="/etc/CA_RADIUS/keys/morfik_laptop.key"
    #   private_key="/etc/CA_RADIUS/keys/morfik_laptop.p12"
    #   private_key_passwd="jakies-haslo-client"
        scan_ssid=0
        disabled=0
    }

Konfiguracja dla EAP-TTLS

    network={
        id_str="home_wifi_static"
        priority=10
        ssid="Winter_Is_Coming"
        bssid=E8:94:F6:68:79:EF
        proto=RSN
        key_mgmt=WPA-EAP
        eap=TTLS
        pairwise=CCMP
        group=CCMP
        auth_alg=OPEN
        phase1="peaplabel=1"
        phase2="auth=MD5"
        anonymous_identity="anonymous"
        identity="morfik"
        password="e6a664c54f6427"
        ca_cert="/etc/CA_RADIUS/keys/ca.crt"
        scan_ssid=0
        disabled=0
    }

Konfiguracja dla PEAP:

    network={
        id_str="home_wifi_static"
        priority=15
        ssid="Winter_Is_Coming"
        bssid=E8:94:F6:68:79:EF
        proto=RSN
        key_mgmt=WPA-EAP
        eap=PEAP
        pairwise=CCMP
        group=CCMP
        auth_alg=OPEN
        phase2="auth=MSCHAPV2"
        anonymous_identity="anonymous"
        identity="morfik"
        password="e6a664c54f6427"
        ca_cert="/etc/CA_RADIUS/keys/ca.crt"
        scan_ssid=0
        disabled=0
    }

Interesują nas głównie linijki zawierające `ca_cert` , `client_cert` , `private_key` ,
`private_key_passwd` . Z tym, że w zależności od konfiguracji, nie wszystkie bloki posiadają każdy z
tych parametrów. Jeśli nie szyfrujemy kluczy, parametr `private_key_passwd` może zostać
wykomentowany. W przypadku protokołów EAP-TTLS oraz PEAP mamy możliwość sprecyzowania parametru
`anonymous_identity` . Zwykle przyjmuje on wartość `anonymous` . Jest to nic innego jak zewnętrzna
tożsamość używana do zestawiania tunelu TLS. W przypadku gdybyśmy nie podali w konfiguracji tego
parametru, zostanie użyta realna tożsamość, czyli ta, która służy do logowania się do sieci WiFi.

Powyższe ustawienia dla protokołów EAP-TTLS i PEAP dają gwarancję, że nie da się podsłuchać realnych
tożsamości używanych przy logowaniu się do sieci. Atakujący węsząc w sieci zobaczy jedynie login
`anonymous` , z którym nic nie da się zrobić.

Jeśli chodzi jeszcze o protokoły EAP-TTLS oraz PEAP nie trzeba definiować parametru `ca_cert` w
konfiguracji narzędzia `wpasupplicant`. W przypadku, gdy tego nie zrobimy, to nie zweryfikujemy tym
samym certyfikatu serwera RADIUS. Jeśli mamy podaną ścieżkę do certyfikatu serwera w konfiguracji,
to w przypadku niezgodności certyfikatu serwera, zostaniemy o tym fakcie poinformowani. Jeśli nie
przeprowadzimy weryfikacji certyfikatu serwera (brak parametru `ca_cert` ), to zostaniemy podłączeni
do sieci jak gdyby nigdy nic. Jeśli ktoś w takiej sytuacji spróbuje postawić AP/NAS, na którym
będzie miał taką samą sieć jak nasza (ten sam BSSID i ESSID), to istnieje spore prawdopodobieństwo,
że podłączymy się do infrastruktury tego kogoś i wszystkie dane logowania do naszego konta w naszej
sieci WiFi podamy mu jak na tacy.
