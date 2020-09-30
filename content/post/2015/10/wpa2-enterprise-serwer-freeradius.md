---
author: Morfik
categories:
- Linux
date: "2015-10-13T18:42:28Z"
date_gmt: 2015-10-13 16:42:28 +0200
published: true
status: publish
tags:
- szyfrowanie
- wifi
- sieć
title: WPA/WPA2 Enterprise i serwer freeradius
---

Poniższy wpis ma na celu stworzenie infrastruktury WiFi w oparciu o oprogramowanie freeradius
zainstalowane na debianowym serwerze. Projekt zakłada wykorzystanie osobnego urządzenia NAS (AP), w
tym przypadku jest to router [TP-Link TL-WR1043N/ND v2][1], na którym jest zainstalowane
oprogramowanie OpenWRT. W oparciu o te dwie maszyny spróbujemy skonfigurować protokół WPA2
Enterprise z obsługą trzech metod uwierzytelniania, tj. EAP-TLS, EAP-TTLS oraz PEAP (v0) . Będziemy
również potrzebować kilku certyfikatów (w tym CA), bez których to pewne mechanizmy mogą nie działać.

<!--more-->
## Instalacja komponentów

W mojej sieci testowej router ma adres 192.168.1.1 , a serwer z debianem 192.168.1.166 . W tym
przypadku będzie jedna maszyna kliencka (laptop) z adresem 192.168.1.150. Zarówno do serwera debiana
jak i do routera będziemy się łączyć przez ssh. Serwer nie posiada środowiska graficznego, jedynie
podstawowy system + serwer ssh.

Na serwerze będziemy instalować oprogramowanie freeradius wraz z modułem obsługującym MySQL.
Potrzebny nam też będzie sam serwer bazy danych MySQL. Logujemy się zatem na serwer debiana via ssh
i instalujemy tam poniższe pakiety:

    $ ssh root@192.168.1.166
    ...
    # aptitude install freeradius freeradius-mysql freeradius-utils mysql-server
    ...

## Wstępna konfiguracja pakietu freeradius

Nie zaleca się odpalania demona freeradius'a z uprawnieniami użytkownika root. W debianie, wszystkie
niezbędne kroki, takie jak zmiana uprawnień plików w katalogu `/etc/freeradius/` , czy sama
konfiguracja użytkownika/grupy `freerad`, zostały przeprowadzone przez opiekuna pakietu i my nie
musimy sobie tym głowy zawracać. Warto jednak prześledzić jak wygląda cały setup poinstalacyjny.
Poniżej znajduje się listing plików konfiguracyjnych wraz z ich uprawnieniami:

    # tree -u -g -p /etc/freeradius/
    /etc/freeradius/
    ├── [-rw-r----- root     freerad ]  acct_users
    ├── [-rw-r----- root     freerad ]  attrs
    ├── [-rw-r----- root     freerad ]  attrs.access_challenge
    ├── [-rw-r----- root     freerad ]  attrs.access_reject
    ├── [-rw-r----- root     freerad ]  attrs.accounting_response
    ├── [-rw-r----- root     freerad ]  attrs.pre-proxy
    ├── [drwxr-s--x freerad  freerad ]  certs
    │   ├── [lrwxrwxrwx root     freerad ]  ca.pem -> /etc/ssl/certs/ca-certificates.crt
    │   ├── [-rw-r--r-- root     freerad ]  dh
    │   ├── [lrwxrwxrwx root     freerad ]  server.key -> /etc/ssl/private/ssl-cert-snakeoil.key
    │   └── [lrwxrwxrwx root     freerad ]  server.pem -> /etc/ssl/certs/ssl-cert-snakeoil.pem
    ├── [-rw-r----- root     freerad ]  clients.conf
    ├── [-rw-r--r-- root     freerad ]  dictionary
    ├── [-rw-r----- root     freerad ]  eap.conf
    ├── [-rw-r----- root     freerad ]  experimental.conf
    ├── [-rw-r----- root     freerad ]  hints
    ├── [-rw-r----- root     freerad ]  huntgroups
    ├── [-rw-r----- root     freerad ]  ldap.attrmap
    ├── [drwxr-xr-x root     root    ]  modules
    │   ├── [-rw-r--r-- root     root    ]  acct_unique
    │   ├── [-rw-r--r-- root     root    ]  always
    │   ├── [-rw-r--r-- root     root    ]  attr_filter
    │   ├── [-rw-r--r-- root     root    ]  attr_rewrite
    │   ├── [-rw-r--r-- root     root    ]  chap
    │   ├── [-rw-r--r-- root     root    ]  checkval
    │   ├── [-rw-r--r-- root     root    ]  counter
    │   ├── [-rw-r--r-- root     root    ]  cui
    │   ├── [-rw-r--r-- root     root    ]  detail
    │   ├── [-rw-r--r-- root     root    ]  detail.example.com
    │   ├── [-rw-r--r-- root     root    ]  detail.log
    │   ├── [-rw-r--r-- root     root    ]  digest
    │   ├── [-rw-r--r-- root     root    ]  dynamic_clients
    │   ├── [-rw-r--r-- root     root    ]  echo
    │   ├── [-rw-r--r-- root     root    ]  etc_group
    │   ├── [-rw-r--r-- root     root    ]  exec
    │   ├── [-rw-r--r-- root     root    ]  expiration
    │   ├── [-rw-r--r-- root     root    ]  expr
    │   ├── [-rw-r--r-- root     root    ]  files
    │   ├── [-rw-r--r-- root     root    ]  inner-eap
    │   ├── [-rw-r--r-- root     root    ]  ippool
    │   ├── [-rw-r--r-- root     root    ]  krb5
    │   ├── [-rw-r--r-- root     root    ]  ldap
    │   ├── [-rw-r--r-- root     root    ]  linelog
    │   ├── [-rw-r--r-- root     root    ]  logintime
    │   ├── [-rw-r--r-- root     root    ]  mac2ip
    │   ├── [-rw-r--r-- root     root    ]  mac2vlan
    │   ├── [-rw-r--r-- root     root    ]  mschap
    │   ├── [-rw-r--r-- root     root    ]  ntlm_auth
    │   ├── [-rw-r--r-- root     root    ]  opendirectory
    │   ├── [-rw-r--r-- root     root    ]  otp
    │   ├── [-rw-r--r-- root     root    ]  pam
    │   ├── [-rw-r--r-- root     root    ]  pap
    │   ├── [-rw-r--r-- root     root    ]  passwd
    │   ├── [-rw-r--r-- root     root    ]  perl
    │   ├── [-rw-r--r-- root     root    ]  policy
    │   ├── [-rw-r--r-- root     root    ]  preprocess
    │   ├── [-rw-r--r-- root     root    ]  radutmp
    │   ├── [-rw-r--r-- root     root    ]  realm
    │   ├── [-rw-r--r-- root     root    ]  redis
    │   ├── [-rw-r--r-- root     root    ]  rediswho
    │   ├── [-rw-r--r-- root     root    ]  replicate
    │   ├── [-rw-r--r-- root     root    ]  smbpasswd
    │   ├── [-rw-r--r-- root     root    ]  smsotp
    │   ├── [-rw-r--r-- root     root    ]  soh
    │   ├── [-rw-r--r-- root     root    ]  sql_log
    │   ├── [-rw-r--r-- root     root    ]  sqlcounter_expire_on_login
    │   ├── [-rw-r--r-- root     root    ]  sradutmp
    │   ├── [-rw-r--r-- root     root    ]  unix
    │   └── [-rw-r--r-- root     root    ]  wimax
    ├── [-rw-r----- root     freerad ]  policy.conf
    ├── [-rw-r----- root     freerad ]  policy.txt
    ├── [-rw-r----- root     freerad ]  preproxy_users
    ├── [-rw-r----- root     freerad ]  proxy.conf
    ├── [-rw-r----- root     freerad ]  radiusd.conf
    ├── [drwxr-s--x freerad  freerad ]  sites-available
    │   ├── [-rw-r--r-- root     root    ]  README
    │   ├── [-rw-r--r-- root     root    ]  buffered-sql
    │   ├── [-rw-r--r-- root     root    ]  coa
    │   ├── [-rw-r--r-- root     root    ]  control-socket
    │   ├── [-rw-r--r-- root     root    ]  copy-acct-to-home-server
    │   ├── [-rw-r--r-- root     root    ]  decoupled-accounting
    │   ├── [-rw-r--r-- root     root    ]  default
    │   ├── [-rw-r--r-- root     root    ]  dhcp
    │   ├── [-rw-r--r-- root     root    ]  dynamic-clients
    │   ├── [-rw-r--r-- root     root    ]  example
    │   ├── [-rw-r--r-- root     root    ]  inner-tunnel
    │   ├── [-rw-r--r-- root     root    ]  originate-coa
    │   ├── [-rw-r--r-- root     root    ]  proxy-inner-tunnel
    │   ├── [-rw-r--r-- root     root    ]  robust-proxy-accounting
    │   ├── [-rw-r--r-- root     root    ]  soh
    │   ├── [-rw-r--r-- root     root    ]  status
    │   ├── [-rw-r--r-- root     root    ]  virtual.example.com
    │   └── [-rw-r--r-- root     root    ]  vmps
    ├── [drwxr-s--x freerad  freerad ]  sites-enabled
    │   ├── [lrwxrwxrwx root     freerad ]  default -> ../sites-available/default
    │   └── [lrwxrwxrwx root     freerad ]  inner-tunnel -> ../sites-available/inner-tunnel
    ├── [-rw-r----- root     freerad ]  sql.conf
    ├── [-rw-r--r-- root     root    ]  sqlippool.conf
    ├── [-rw-r--r-- root     root    ]  templates.conf
    └── [-rw-r--r-- root     root    ]  users

    4 directories, 96 files

Niżej zaś znajduje się konfiguracja użytkownika `freerad` :

    # grep freerad /etc/group
    shadow:x:42:freerad
    freerad:x:109:
    ssl-cert:x:110:freerad

    # grep freerad /etc/passwd
    freerad:x:105:109::/etc/freeradius:/bin/false

    # grep freerad /etc/shadow
    freerad:*:16321:0:99999:7:::

Trzeba zaznaczyć, że użytkownik `freerad` jest członkiem grupy `shadow` i to umożliwia mu odczyt
zahashowanych haseł użytkowników systemowych, co jest przydatne, np. przy korzystaniu z modułu
`unix` .

W pliku `/etc/freeradius/radiusd.conf` są przechowywane informacje na temat interfejsu, adresu,
portów, użytkownika i grupy na jakich będzie operowało oprogramowanie freeradius. Część z opcji
musimy dostosować sobie według naszych potrzeb:

    ...
    user = freerad
    group = freerad
    ...
    listen {
            type = auth
            port = 0
            ipaddr = 192.168.1.166
            interface = eth0
    ...
    listen {
            type = acct
            port = 0
            ipaddr = 192.168.1.166
            interface = eth0
    ...

Port `0` w tym wypadku oznacza port domyślny, a jego wartość jest czytana z pliku `/etc/services` .
Są to 1812 (auth) i 1813 (acct).

Zapisujemy zmiany i testujemy wstępnie serwer odpalając go w trybie debug za pomocą poniższego
polecenia:

    # freeradius -XXX

Wygeneruje ono pokaźnych rozmiarów log, na końcu którego powinna być poniższa informacja:

    Debug: Listening on authentication interface eth0 address 192.168.1.166 port 1812
    Debug: Listening on accounting interface eth0 address 192.168.1.166 port 1813
    Debug: Listening on authentication address 127.0.0.1 port 18120 as server inner-tunnel
    Debug: Listening on proxy address 192.168.1.166 port 1814
    Info: Ready to process requests.

### Firewall

Poniższy krok nie jest konieczny, w końcu jakby nie patrzeć w swojej sieci lokalnej raczej wszystkie
maszyny mamy zaufane i nie trzeba dodatkowo dozbrajać serwera freeradius pod kątem restrykcji
dostępu. Czasem jednak potrzebny jest firewall, w którym to musimy przepuścić ruch na porcie
`22/tcp` (domyślny ssh) z konkretnego adresu IP, ruch na portach `1812/udp`, `1813/udp`, `1814/udp`
(domyślne porty freeradius'a) oraz zapytania do serwera bazy danych MySQL, który operuje standardowo
na porcie `3306/tcp` , choć nie powinniśmy wystawiać bazy danych na widok publiczny, bo o wiele
lepszym rozwiązaniem jest logowanie się po ssh i lokalne administrowanie bazą.

Jeśli chodzi o samego freeradius'a jeszcze, to portem `1812` lecą zapytania uwierzytelniające,
portem `1813` pakiety dotyczące wykorzystania zasobów sieci, a port `1814` odpowiada za
forwardowanie zapytań (proxy). Domyślnie freeradius nasłuchuje na wszystkich tych portach i jeśli
nie potrzebujemy, np. informacji na temat czasu trwania sesji danego użytkownika, czy bajtów ile
przesłał, lub zwyczajnie nie korzystamy z proxy, to możemy zahashować w pliku
`/etc/freeradius/radiusd.conf` odpowiednią sekcję.

Tak czy inaczej, reguły firewall'a, które trzeba wziąć pod uwagę, są wypisane poniżej:

    iptables -t filter -p udp --dport 1812 -j ACCEPT -m comment --comment "RADIUS AUTH"
    iptables -t filter -p udp --dport 1813 -j ACCEPT -m comment --comment "RADIUS ACCT"
    iptables -t filter -p udp --dport 1814 -j ACCEPT -m comment --comment "RADIUS PROXY"
    iptables -t filter -p tcp --dport 3306 -j ACCEPT -m comment --comment "MySQL"
    iptables -t filter -p tcp --dport 22 -s 192.168.1.150 -j ACCEPT -m comment --comment "SSH"

## Certyfikaty

Freeradius w katalogu `/etc/freeradius/certs/` tworzy linki do istniejących już w systemie
certyfikatów. Są to testowe certyfikaty dla usług, które ich wymagają do poprawnego działania i
pochodzą one z pakietu `ssl-cert` . Świetnie się one nadają do celów testowych i sprawdzenia czy
instalacja freeradius'a działa jak należy. Te certyfikaty nie są zalecane na serwerach
produkcyjnych, dlatego też musimy wygenerować sobie własne certyfikaty.

[Generowanie certyfikatów][2] można przeprowadzić ręcznie. Można także posłużyć się [pakietem
easy-rsa][3]. Istnieje także możliwość skorzystania z przygotowanego do tego celu narzędzi
zawartych w pakiecie freeradius. Informacje na temat korzystania z narzędzi freeradius'a są
dostępne w pliku `/usr/share/doc/freeradius/examples/certs/README` .

Po tym jak zainstalowaliśmy pakiet freeradius, kilka plików zostało utworzonych w katalogu
`/usr/share/doc/freeradius/examples/certs` i mamy tam między innymi `bootstrap` , `ca.cnf` ,
`client.cnf` , `server.cnf` oraz `xpextensions` . W zależności od konfiguracji freeradius'a przy
jego kompilacji, podczas uruchomienia serwera jako root w trybie debug ( `freeradius -X` ), plik
`bootstrap` może zostać wykonywany i zostaną stworzone w ten sposób certyfikaty dla określonych
maszyn. W przypadku debiana, to jednak nie ma miejsca ale wszystkie potrzebne nam pliki są
dostarczone i możemy ten skrypt wywołać ręcznie z takim samym efektem.

Po tym jak wygenerujemy własne certyfikaty (metoda bez znaczenia), musimy edytować plik
`/etc/freeradius/eap.conf` i wykomentować pozycję z `make_cert_command` w podsekcji `tls` , tak by
wyglądało to jak poniżej:

        tls {
        ...
    #   make_cert_command = "${certdir}/bootstrap"

Po tym kroku, przechodzimy do katalogu `/usr/share/doc/freeradius/examples/certs/` , gdzie będą
wykonywane kolejne polecenia. Następnie edytujemy pliki `ca.cnf` , `client.cnf` i `server.cnf` . Są
to kopie pliku konfiguracyjnego OpenSSL, który zwykle jest dostępny pod `/etc/ssl/openssl.cnf` .
Każdy z tych plików został zmieniony tak by maksymalnie uprościć procedurę generowania certyfikatów
i w sumie nasza ingerencja sprowadza się do edycji linijek zawierających `input_password` oraz
`output_password` . Można oczywiście dostosować szereg innych parametrów, np. termin ważności
certyfikatu, czy długość klucza RSA. Dobrze jest zatem przejrzeć te pliki.

Wszystkie certyfikaty generowane za pomocą skryptu dostarczonego z pakietem freeradius będą
domyślnie używać MD5 jako funkcji skrótu (hash). Dlatego warto zaznaczyć w tym miejscu by zmienić
wartość parametru `default_md` w każdym z trzech powyższych plików z `md5` na `sha256` lub nawet
`sha512`.

W przypadku klientów, ważne jest by odpowiednio dobrać parametr `commonName` , bo to on zostanie
później użyty przy uwierzytelnianiu podczas podłączania się klienta do sieci WiFi.

Teraz już tylko pozostało nam odpalenie skryptu `bootstrap` , który wygeneruje certyfikaty dla CA i
serwera:

    # cd /usr/share/doc/freeradius/examples/certs
    # ./bootstrap

Dodatkowo musimy stworzyć certyfikaty dla klientów:

    # make client

Teraz powinniśmy mieć już komplet plików.

Listując katalog roboczy możemy dostrzec, że te pliki [mają różne rozszerzenia][4]. Czym zatem
różnią się one między sobą? Plik `.crt` to certyfikat główny. W osobnym pliku `.key` znajduje się
klucz prywatny od tego certyfikatu. Można rozpoznać to po "-----BEGIN ENCRYPTED PRIVATE KEY-----" i
"-----END ENCRYPTED PRIVATE KEY-----". W przypadku gdy nie potrzebujemy oddzielać certyfikatu od
klucza, możemy posłużyć się plikiem `.p12` lub plikiem `.pem` . Mogą one zawierać zarówno certyfikat
jak i klucz prywatny i zamiast dwóch osobnych plików, jest tylko jeden. Różnica między nimi polega
na tym, że plik `.p12` jest używany głównie na windowsach, ten drugi zaś przez OpenSSL. Plik `.csr`
jest prośbą o podpisanie certyfikatu (Certificate Signing Request) wysyłaną przez aplikację do CA
(Certification Authority). Można to poznać po linijkach "-----BEGIN NEW CERTIFICATE REQUEST-----"
oraz "-----END NEW CERTIFICATE REQUEST-----". Dodatkowo, mamy plik binarny certyfikatu `.der`
zakodowany w DER. Nie ma jednak możliwości umieszczenia w tym pliku klucza prywatnego, czy też
certyfikatów ze struktury certyfikatów (certification path). Wygenerowany on został głównie na
potrzeby windowsów.

Oprogramowanie freeradius w swojej konfiguracji potrzebuje nastepujących plików: `server.key` ,
`server.pem` oraz `ca.pem` . Te pliki trzeba przenieść do katalogu `/etc/freeradius/certs/` . Z
kolei na kliencie potrzebne nam będą `ca.pem` jako certyfikat CA oraz `client.p12` albo `client.key`
i `client.crt` .

### Implementacja certyfikatów na serwerze

Narobiliśmy trochę plików i teraz trzeba je rozdzielić pomiędzy określone maszyny. Na początek
kopiujemy wszystko co dotyczy serwera freeradius do katalogu `/etc/freeradius/certs/` . Domyślnie
jednak znajdują się tam certyfikaty stworzone po instalacji pakietu freeradius, które musimy usunąć:

    # rm /etc/freeradius/certs/*
    # cp /etc/CA/ca.pem /etc/CA/server.key /etc/CA/server.pem /etc/freeradius/certs/

Przechodzimy do katalogu z certyfikatami dla freeradius'a i generujemy przy pomocy OpenSSL plik
`dh` , który zawiera [szereg parametrów wykorzystywanych przy wymianie kluczy w protokole
Diffie-Hellman'a][5]. Standardowo ta liczba ma 1024 bity i przydałoby się nieco zwiększyć:

    # cd /etc/freeradius/certs/
    # openssl dhparam -out dh 4096

### Implementacja certyfikatów na kliencie

By skonfigurować klienta, który będzie korzystał z WiFi, musimy przekopiować `ca.pem` oraz
`client.p12` (ewentualnie `client.crt` i `client.key` ) na maszynę kliencką, np. do katalogu
`/etc/wifi_cert/radius_192.168.1.166/` .

Na linuxach, do konfiguracji sieci WiFi są wykorzystywane narzędzia zawarte w pakiecie
`wpasupplicant` . Można oczywiście skorzystać z `wicd` albo `network-manager` jeśli ktoś nie
przepada za ręczą edycją plików tekstowych. W zależności od wybranej metody, nieco inaczej będzie
wyglądać konfiguracja suplikanta (klienta WiFi). Ja korzystałem z trybu tekstowego przy
[konfiguracji WiFi][6]. Nie będę tutaj opisywał całego procesu ale jeśli ktoś ma problemy związane
z konfiguracją linux'owego klienta, to odsyłam go do podlinkowanego wpisu.

### Niedogodności związane z certyfikatami

W przypadku wykorzystywania kluczy o większych rozmiarach, np. 4096 bitów, warto zrobić sobie test i
zobaczyć jak radzi sobie nasza maszyna w kalkulacjach RSA. Taki benchmark można przeprowadzić
wydając poniższe polecenie:

    # openssl speed rsa
    ...
                      sign    verify    sign/s verify/s
    rsa  512 bits 0.000196s 0.000016s   5099.1  61558.5
    rsa 1024 bits 0.000837s 0.000049s   1194.1  20247.7
    rsa 2048 bits 0.005616s 0.000173s    178.1   5772.5
    rsa 4096 bits 0.041405s 0.000668s     24.2   1496.4

Z powyższego wyniku interesuje nas głównie kolumna `sign/s` . Jest to ilość jednoczesnych prób
uwierzytelniania klientów WiFi jakie serwer jest w stanie obsłużyć wykorzystując mechanizm EAP-TLS,
EAP-TTLS albo PEAP.

## Protokoły EAP

[Jest wiele metod uwierzytelniania EAP][7] i nas interesować będą trzy z nich EAP-TLS, EAP-TTLS i
PEAP. Metoda `EAP-TLS`, w odróżnieniu od EAP-TTLS i PEAP, wymaga wzajemnej weryfikacji certyfikatów
na linii serwer-klient. Z kolei `EAP-TTLS` i `PEAP` zbytnio się nie różnią między sobą, no może za
wyjątkiem wsparcia, bo ten drugi jest rozwijany przez MS. Obie te metody wykorzystują szyfrowany
tunel TLS, który jest również podstawą dla uwierzytelniania przy metodzie EAP-TLS i by korzystać z
EAP-TTLS lub PEAP, trzeba po części skonfigurować uwierzytelnianie EAP-TLS. Same certyfikaty
klienckie w przypadku EAP-TTLS i PEAP są nieobowiązkowe i gdy nie korzystamy z nich,
uwierzytelnianie klienta odbywa się przy pomocy loginu i hasła, co znacznie upraszcza wdrożenie
protokołu WPA2 Enterprise.

Jeśli chodzi jeszcze o porównanie EAP-TTLS/PEAP w stosunku do EAP-TLS, to proces uwierzytelniania w
przypadku tych pierwszych odbywa się w dwóch fazach. EAP-TLS ma tylko jedną fazę, podczas której
wymieniane są klucze publiczne i zestawiany jest tunel TLS. Sam tunel tworzony jest na podobnej
zasadzie co w przypadku SSL na stronach www. W EAP-TTLS/PEAP występują dwie tożsamości -- zewnętrzna
i wewnętrzna (szyfrowana wewnątrz tunelu TLS). Zewnętrzna tożsamość w EAP-TTLS/PEAP odpowiada
tożsamości w TLS i w obu przypadkach ta tożsamość widnieje w pakietach sieciowych. Dlatego jeśli
zależy nam na anonimowości klientów, nie możemy korzystać z EAP-TLS. Niemniej jednak ta metoda jest
najbezpieczniejsza ze wszystkich dostępnych na rynku.

W protokołach EAP-TTLS/PEAP, wewnątrz tunelu TLS, wykorzystywane są inne protokoły EAP, którymi
przesyłane są login i hasło. Trzeba rozróżnić te dwie kwestie. Pierwszą jest wykorzystanie np.
EAP-MD5 przy uwierzytelnianiu jest poważnym ryzykiem, bo hasło w postaci hasha md5 jest przesyłane
przez sieć, a ten protokół w obecnych czasach nie daje już gwarancji bezpieczeństwa. Natomiast jeśli
wykorzystujemy tunel TLS, to typ zastosowanej metody wewnątrz tunelu nie ma już dla nas większego
znaczenia. Nawet możemy puścić hasło otwartym tekstem i tunel TLS daje nam gwarancję poufności
danych wewnątrz niego i dlatego nie trzeba tych danych dodatkowo szyfrować.

Jeśli chodzi o samą konfigurację klientów, to windowsy standardowo nie działają z metodą EAP-TTLS i
muszą używać PEAP, co trzeba wziąć pod uwagę, przy konfiguracji serwera freeradius.

W związku z powyższym, ustawmy domyślny protokół EAP na PEAP, zakładając, że najwięcej klientów
będzie właśnie go używać. Konfiguracja EAP odbywa się w pliku `/etc/freeradius/eap.conf` .
Przechodzimy zatem do jego edycji:

    ...
    default_eap_type = peap
    ...

### Konfiguracja tunelu TLS

Jeśli już wszystkie pliki są wgrane tam gdzie powinny, możemy przejść do konfigurowania tunelu TLS
pod uwierzytelnianie EAP-TLS, EAP-TTLS i PEAP. Dokonujemy tego w pliku `/etc/freeradius/eap.conf` w
sekcji `tls` :

    ...
    tls {
          certdir = ${confdir}/certs
          cadir = ${confdir}/certs

          private_key_password = jakies-haslo-server

          private_key_file = ${certdir}/server.key

          certificate_file = ${certdir}/server.pem

          CA_file = ${cadir}/ca.pem

          dh_file = ${certdir}/dh
          random_file = /dev/urandom
          CA_path = ${cadir}
    ...

By się zalogować do sieci WiFi wykorzystując EAP-TLS, potrzebujemy odpowiedniej konfiguracji dla
wpasupplicant'a na kliencie:

    network={
          id_str="home_wifi_static"
          priority=10
          ssid="Winter Is Coming"
          bssid=E8:94:F6:68:79:EF
          proto=RSN
          key_mgmt=WPA-EAP
          eap=TLS
          pairwise=CCMP
          group=CCMP
          auth_alg=OPEN
          identity="linux_laptop"
          ca_cert="/etc/wifi_cert/radius_192.168.1.166/ca.pem"
          client_cert="/etc/wifi_cert/radius_192.168.1.166/client.pem"
          private_key="/etc/wifi_cert/radius_192.168.1.166/client.key"
    #     private_key="/etc/wifi_cert/radius_192.168.1.166/client.p12"
          private_key_passwd="jakies-haslo-client"
          scan_ssid=0
          disabled=0
    }

Interesują nas głównie linijki zawierające `ca_cert` , `client_cert` , `private_key` oraz
`private_key_passwd` . Powyżej mamy wykomentowaną jedną pozycję i jeśli chcemy wykorzystać plik
`.p12`, który zawiera certyfikat i klucz prywatny, musimy wykomentować dwie linijki wyżej.

#### Konfiguracja EAP-TTLS

Mając tunel TLS, możemy przejść do konfiguracji EAP-TTLS. Jest ona trochę niżej, również w pliku
`/etc/freeradius/eap.conf` :

    ...
    ttls {
       default_eap_type = md5
       copy_request_to_tunnel = no
       use_tunneled_reply = no
       virtual_server = "inner-tunnel"
    }
    ...

Powyższa konfiguracja jest domyślna i nie musimy zbytnio nic w niej zmieniać. By podłączyć się do
sieci z wykorzystaniem protokołu EAP-TTLS, wrzucamy poniższą zwrotkę do pliku konfiguracyjnego
wpasupplicant'a na kliencie:

    network={
          id_str="home_wifi_static"
          priority=10
          ssid="Winter Is Coming"
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
          password="super-tajne+haslo"
          ca_cert="/etc/wifi_cert/radius_192.168.1.166/ca.pem"
          scan_ssid=0
          disabled=0
    }

EAP-TTLS nie jest obsługiwany przez systemy operacyjne spod znaku MS, bo te korzystają z rozwijanego
przez MS protokołu PEAP. Jeśli się chce korzystać z EAP-TTLS, to albo trzeba zmienić system, albo
klienta WiFi, na taki który potrafi obsłużyć EAP-TTLS jako metodę uwierzytelniania.

#### Konfiguracja protokołu PEAP

Jeszcze niżej w pliku `/etc/freeradius/eap.conf` mamy konfigurację dla PEAP:

    peap {
       default_eap_type = mschapv2
       copy_request_to_tunnel = no
       use_tunneled_reply = no
       virtual_server = "inner-tunnel"
    }

By się podłączyć do sieci używając protokołu PEAP, tworzymy poniższe wpisy w pliku konfiguracyjnym
wpasupplicant'a na kliencie:

    network={
          id_str="home_wifi_static"
          priority=15
          ssid="Winter Is Coming"
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
          password="super-tajne+haslo"
          ca_cert="/etc/wifi_cert/radius_192.168.1.166/ca.pem"
          scan_ssid=0
          disabled=0
    }

### Anonimowa tożsamość

W przypadku protokołów EAP-TTLS oraz PEAP mamy możliwość sprecyzowania w pliku konfiguracyjnym
wpasupplicant'a parametru `anonymous_identity` . Zwykle przyjmuje on wartość `anonymous` . Jest to
nic innego jak zewnętrzna tożsamość używana do zestawiania tunelu TLS. W przypadku gdybyśmy nie
podali w konfiguracji tego parametru, zostanie użyta realna tożsamość, czyli ta, która służy do
logowania się do sieci.

Poniżej jest krótki wycinek z logu freeradius'a, który obrazuje cały proces:

    rad_recv: Access-Request packet from host 192.168.1.1 port 59500, id=220, length=283
            User-Name = "anonymous"
    ...
    Sat Sep 20 12:09:25 2014 : Info: [ttls] Authenticate
    Sat Sep 20 12:09:25 2014 : Info: [ttls] processing EAP-TLS
    ...
    Sat Sep 20 12:09:25 2014 : Info: [ttls] Session established.  Proceeding to decode tunneled attributes.
    ...
    Sat Sep 20 12:09:25 2014 : Info: [ttls] Got tunneled request
    ...
    Sat Sep 20 12:09:25 2014 : Info: [ttls] Got tunneled identity of morfik
    ...

Powyższe ustawienia dla protokołów EAP-TTLS i PEAP dają gwarancję, że nie da się podsłuchać realnych
tożsamości używanych przy logowaniu się do sieci WiFi. W taki sposób atakujący jedyne co zobaczy
węsząc w sieci, to login `anonymous`, z którym nic nie da się zrobić.

### Weryfikacja certyfikatu serwera

W przypadku protokołów EAP-TTLS oraz PEAP nie trzeba definiować parametru `ca_cert` w konfiguracji
wpasupplicant'a. W przypadku gdy tego nie zrobimy, nie zweryfikujemy tym samym certyfikatu serwera
freeradius. Jeśli mamy podaną ścieżkę do certyfikatu serwera w konfiguracji, to w przypadku
niezgodności certyfikatu serwera, nie zostaniemy podłączeni do sieci i zostanie nam zwrócony
poniższy komunikat (na kliencie w wpasupplicant):

    SSL: SSL_connect:SSLv3 read server hello A
    TLS: Certificate verification failed, error 19 (self signed certificate in certificate chain) depth 1 for '/C=US/ST=California/O=NSA/CN=CA/emailAddress=morfik@nsa.com'
    wlan0: CTRL-EVENT-EAP-TLS-CERT-ERROR reason=1 depth=1 subject='/C=US/ST=California/O=NSA/CN=CA/emailAddress=morfik@nsa.com' err='self signed certificate in certificate chain'
    SSL: (where=0x4008 ret=0x230)
    SSL: SSL3 alert: write (local SSL3 detected an error):fatal:unknown CA
    SSL: (where=0x1002 ret=0xffffffff)
    SSL: SSL_connect:error in SSLv3 read server certificate B
    OpenSSL: openssl_handshake - SSL_connect error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
    SSL: 7 bytes pending from ssl_out
    SSL: Failed - tls_out available to report error
    SSL: 7 bytes left to be sent out (of total 7 bytes)

Krótko mówiąc, nie udało się zweryfikować certyfikatu serwera. W przypadku nieprzeprowadzenia
weryfikacji certyfikatu serwera (brak parametru `ca_cert` ), zostaniemy podłączeni do sieci jak
gdyby nigdy nic. Jeśli ktoś w takiej sytuacji spróbuje postawić AP/NAS, na którym będzie miał taką
samą sieć jak nasza (ten sam BSSID i ESSID), istnieje spore prawdopodobieństwo, że podłączymy się do
infrastruktury tego kogoś i wszystkie dane logowania do naszego konta podamy mu jak na tacy.

## Konfiguracja klientów serwera freeradius

W tym przypadku słowo klient nie jest równoznaczne ze słowem klient gdy mówimy o sieci WiFi. Tamten
klient jest obsługiwany przez AP, natomiast w przypadku serwera freeradius, klientem dla niego jest
nasz AP, w tym wypadku router. Musimy zdefiniować to urządzenie w konfiguracji freeradius'a w pliku
`/etc/freeradius/clients.conf` . Domyślnie we wspomnianym pliku widnieje klient `localhost` i
możliwe jest przetwarzanie jedynie zapytań lokalnych, np. w przypadku gdyby oprogramowanie
freeradius było zainstalowane na urządzeniu AP/NAS. My musimy dodać do tego pliku nasz router:

    ...
    client router_1043nd {
       ipaddr = 192.168.1.1
       netmask = 32
       secret = "tajna-fraza"
       require_message_authenticator = yes
       nastype = other
    }
    ...

Jeśli w powyższym pliku byśmy nie zdefiniowali odpowiedniego urządzenia lub byśmy zrobili to
błędnie, serwer freeradius zwróci nam poniższy
    komunikat:

    Error: Ignoring request to authentication interface eth0 address 192.168.1.166 port 1812 from unknown client 192.168.1.1 port 41805

Na podstawie parametru `secret`, serwer freeradius decyduje czy ma obsłużyć tego klienta. W
konfiguracji klienta (routera) trzeba będzie również uzupełnić ten parametr by wskazywał dokładnie
tę samą wartość. W przeciwnym wypadku serwer freeradius zrzuci wszystkie zapytania otrzymane od
tego konkretnego AP/NAS co uniemożliwi podłączenie się użytkownikom do sieci. W logu freeradius zaś
będzie widoczny poniższy komunikat:

    rad_recv: Access-Request packet from host 192.168.1.1 port 41805, id=10, length=177
    Thu Sep 18 11:54:59 2014 : Info: Received packet from 192.168.1.1 with invalid Message-Authenticator!  (Shared secret is incorrect.) Dropping packet without response.

### Konfiguracja AP/NAS

Logujemy się na router po ssh i edytujemy plik `/etc/config/wireless` , w którym to trzeba
zdefiniować parametry serwera freeradius:

    ...
    config wifi-iface
       option device 'radio0'
       option network 'lan'
       option mode 'ap'
       option ssid 'Winter Is Coming'
       option encryption 'wpa2+aes'
       option key 'tajna-fraza'
       option server '192.168.1.166'
       option port '1812'
       option disabled '0'
       option hidden '0'
       option isolate '0'
    ...

Parametr `key` odpowiada za `secret` ustawiony w `/etc/freeradius/clients.conf` .

Musimy także wymienić nieco oprogramowania, bo domyślnie zainstalowany pakiet `wpad-mini` nie
zawiera obsługi WPA2 Enterprise. Jeśli chcemy mieć obsługę WPA2 Enterprise, to musimy doinstalować
pakiet `wpad` uprzednio usuwając `pakiet wpad-mini` :

    # opkg remove wpad-mini
    # opkg install wpad

Teraz już tylko resetujemy WiFi na routerze:

    # wifi

### Konfiguracja użytkowników

By użytkownik mógł się podłączyć do sieci, potrzebne są mu login i hasło, a te zaś, na serwerze
freeradius, mogą być przechowywane w różny sposób. Domyślnie freeradius trzyma konfigurację
użytkowników w pliku `/etc/freeradius/users` . Innym rozwiązaniem jest zaprzęgnięcie do obsługi
użytkowników bazy danych, definiując w niej min. parametry logowania.

W przypadku jednoczesnego włączenia szeregu modułów, może się zdarzyć sytuacja, w której dany
użytkownik zostanie zdefiniowany, powiedzmy, dwa razy. W takiej sytuacji będzie decydować kolejność
wywołania modułów w pliku konfiguracyjnym i ostatni moduł przetwarzający danego użytkownika będzie
decydował czy przyznać mu dostęp.

Warto także pamiętać o tym, że każdy moduł posiada swój plik konfiguracyjny w katalogu
`/etc/freeradius/modules/` , za wyjątkiem kilku podstawowych modułów, które posiadają swoją
konfigurację w odpowiednich plikach w katalogu `/etc/freeradius/` .

#### Moduł files

W przypadku tego modułu, użytkowników definiuje się w pliku `/etc/freeradius/users` . Dodajmy na
końcu tego pliku dwóch użytkowników -- jeden z nich przetestuje uwierzytelnianie EAP-TTLS/PEAP , a
drugi EAP-TLS:

    "morfik"          Cleartext-Password := "morfik-ma-kota"
                      Reply-Message = "Witaj, %{User-Name}"

    "morfik_laptop"   Auth-type := EAP
                      Reply-Message = "Witaj, Morfiku!"

    DEFAULT           Auth-type := Reject
                      Reply-Message := "SPADAJ NA DRZEWO!"

Freeradius szuka w powyższym pliku dopasowań i domyślnie jeśli jakieś znajdzie, kończy
przetwarzanie. Takie zachowanie można dostosować ale na razie zostawmy tak jak jest. Jeśli
użytkownik nie zostanie odnaleziony, zadziała linijka z `DEFAULT` , czyli zapytanie zostanie
odrzucone z odpowiednim komunikatem. Przy czy, jeśli zamierzamy korzystać z anonimowej tożsamości
(przy EAP-TTLS/PEAP), to nie możemy sprecyzować domyślnego zrzucania zapytań, bo wtedy serwer
freeradius nie będzie chciał rozmawiać z użytkownikiem anonimowym.

Użytkownicy posiadający `Cleartext-Password` będą się łączyć do sieci przy pomocy protokołów
EAP-TTLS/PEAP. Z kolei ci posiadający `Auth-type := EAP` , przy pomocy jednej z pozostałych metod
EAP, w tym też EAP-TLS, w którym to nazwa zdefiniowana jako użytkownik zależy od pola `CommonName`
certyfikatu.

#### Moduł unix

Jeśli chodzi o samych użytkowników, nie jest wymagane korzystanie z zewnętrznego oprogramowania
bazodanowego. Możemy skorzystać z bazy danych użytkowników debiana. Po to właśnie freeradius ma
dostęp do pliku `/etc/shadow` by móc czytać zahashowane hasła użytkowników systemowych. Jeśli jakiś
klient WiFi posiada konto w systemie i jest ono aktywne, będzie mógł wykorzystać jego login i hasło
jako dane dostępowe do sieci WiFi.

Trzeba jednak pamiętać, że to z jakiego modułu będziemy korzystać przy weryfikacji użytkowników, ma
[wpływ na dostępne dla nas metody uwierzytelniania][8]. Protokół PAP może być użyty ze wszystkimi
dostępnymi metodami, bo przesyła hasła zwykłym tekstem. Podobnie protokół CHAP. Z kolei MS-CHAP
wymaga, albo haseł podanych zwykłym tekstem, albo hashów NT-Password. W przypadku haseł debiana,
musimy skorzystać z protokołu PAP. Nie ma zbytnio dla nas znaczenia czy przesyłanie haseł jest
jakoś zabezpieczone przez dany protokół, bo i tak wszystko wleci w kanał TLS. Jeśli zdecydujemy się
zmienić domyślne zarządzanie hasłami we freeradius, trzeba zmodyfikować odpowiednie sekcje w
plikach konfiguracyjnych. Edytujemy zatem pliki `/etc/freeradius/sites-enabled/default` oraz
`/etc/freeradius/sites-enabled/inner-tunnel` i w sekcji `authorize` zmieniamy sposób obsługiwania
użytkowników:

    ...
    authorize {
          ...
          unix
       ...
    #     files
    ...

Oczywiście można korzystać z obu tych modułów jednocześnie, tylko trzeba pilnować by dany użytkownik
nie powtarzał się, co może powodować problemy z uwierzytelnianiem. Dodatkowo by zabezpieczyć nieco
sam serwer, należy wyłączyć możliwość używania shella oraz zmienić katalog domowy w kontach
używanych przez freeradius. Na sam początek edytujemy plik `/etc/default/useradd` i zmieniamy w nim
poniższe linijki:

    ...
    #SHELL=/bin/sh
    SHELL=/usr/sbin/nologin_freeradius
    ...
    HOME=/var/run
    ...

Shell `/usr/sbin/nologin_freeradius` jest kopią `/usr/sbin/nologin` i jak każdy inny shell, który
nie jest faktycznie shellem, zostanie potraktowany przez freeradius'a jako błędny. Problem w tym, że
freeradius nie chce obsługiwać takiego shella. Wyjściem jest dodanie powyższego shella do listy
poprawnych. Dodajemy zatem ścieżkę `/usr/sbin/nologin_freeradius` do pliku `/etc/shells` :

    # echo "/usr/sbin/nologin_freeradius" >> /etc/shells

    # cat /etc/shells
    # /etc/shells: valid login shells
    /bin/sh
    /bin/dash
    /bin/bash
    /bin/rbash
    /usr/sbin/nologin_freeradius

Tworzymy testowe konto użytkownika:

    # useradd morfikanin

Sprawdźmy czy jego konfiguracja jest poprawna:

    # grep morfikanin /etc/passwd
    morfikanin:x:1001:1001::/var/run/morfikanin:/usr/sbin/nologin_freeradius

    # grep morfikanin /etc/group
    morfikanin:x:1001:

    # grep morfikanin /etc/shadow
    morfikanin:$6$ztDybjKr$w29hLJJIemqD940VyQBOw/VDRtXoxotWsE.DigutD3UjRa9.Zh/5LZ9fcILJ00iZQ4dCycwfL9gd5FOzmWlbA0:16324:0:99999:7:::

I to z grubsza tyle, przynajmniej jeśli chodzi o konfigurację modułu `unix` na serwerze freeradius.
Trzeba także zmienić nieco konfigurację wpasupplicant'a na stacji klienckiej. Chodzi o odpowiednie
dostosowanie fazy drugiej połączenia, tej wewnątrz tunelu TLS:

    ...
    phase2="auth=PAP"
    ...

### Restrykcje czasowe (moduł logintime)

Freeradius pozwala także na ustanowienie limitów czasu połączenia, np. jeśli chcemy by dany
użytkownik miał dostęp do sieci w godzinach 18-22. Do tego celu służy moduł `logintime` i jest
domyślnie włączony. By ustawić limit użytkownikom, musimy wyedytować plik `/etc/freeradius/users` i
zmienić nieco domyślne wpisy użytkowników, które ustawiliśmy wyżej, przykładowo:

    "morfik"        Cleartext-Password := "morfik-ma-kota", Login-Time :='Al1800-2200'

Parametr `Login-Time` składa się z trzech części. Dwie z nich to godziny, które precyzują czas
(format 24 godzinny) w jakim użytkownik może być zalogowanym do sieci -- start o 18:00 (1800) i
koniec o 22:00 (2200). Ostatnia z trzech części to, w tym wypadku `Al` (małe L), co oznacza `all` ,
czyli wszystkie dni tygodnia. Można sprecyzować też szereg innych wartości: `Mo` , `Tu` , `We` ,
`Th` , `Fr` , `Sa` , `Su` , czyli dwie pierwsze litery angielskiej nazwy dnia tygodnia. Można
również określić przedział, np. od poniedziałku do piątku w postaci `Mo-Fr` . Dodatkowo, można
stosować kombinacje, przykładowo: `"Mo-We1300-1400,Sa,Su2305-0630"` , co oznacza, że użytkownik może
się zalogować do sieci od poniedziałku do środy w godzinach 13-14. Dodatkowo może być zalogowany w
sobotę cały czas i w niedzielę w nocy od 23:05 do 6:30 rano w poniedziałek.

Przy logowaniu się do sieci po godzinie 22, serwer freeradius zwróci poniższy komunikat:

    Sun Sep 14 22:29:25 2014 : Debug: rlm_logintime: Checking Login-Time: 'Al1800-2200'
    Sun Sep 14 22:29:25 2014 : Debug: rlm_logintime: timestr returned reject
    Sun Sep 14 22:29:25 2014 : Info: [logintime]    expand: You are calling outside your allowed timespan   -> You are calling outside your allowed timespan
    Sun Sep 14 22:29:25 2014 : Info: ++[logintime] returns reject

A my nie będziemy w stanie się podłączyć do sieci. W przypadku gdyby użytkownik zalogował się
wcześniej, np. o godzinie 21:34, wtedy zostanie wyliczony timeout w sekundach, po którym użytkownik
wyleci z sieci (zostanie rozłączony), przykładowo:

    Sun Sep 14 21:34:19 2014 : Debug: rlm_logintime: Checking Login-Time: 'Al1800-1900'
    Sun Sep 14 21:34:19 2014 : Debug: rlm_logintime: timestr returned accept
    Sun Sep 14 21:34:19 2014 : Debug: rlm_logintime: Session-Timeout set to: 1620
    Sun Sep 14 21:34:19 2014 : Info: ++[logintime] returns ok

`Session-Timeout set to: 1620` to 1620 sekund, czyli 27 minut. O godzinie 22:01:19 powinno nastąpić
rozłączenie.

Rozłączanie w oparciu o `Session-Timeout` nie zadziała jeśli nie zmienimy domyślnej konfiguracji EAP
w pliku `/etc/freeradius/eap.conf` . Musimy przepisać poniższe parametry:

    ...
    ttls {
          ...
          copy_request_to_tunnel = yes
          use_tunneled_reply = yes
    ...
    peap {
    ...
       copy_request_to_tunnel=yes
       use_tunneled_reply = yes
    ...

### Liczba sesji podłączonych do sieci

Zwykle użytkownik nawiązuje jedno połączenie do sieci. Może ich nawiązać więcej, np. podłączyć
drugie urządzenie i wpisać w nim dokładnie takie same dane do logowania. Może także odsprzedać swój
login komuś trzeciemu i mogą oni we dwóch korzystać z naszej sieci. Możemy taki proceder ukrócić
przez ustawienie limitu jednoczesnych zalogowań do sieci per user.

Jeśli interesuje nas ograniczanie ilości aktywnych sesji użytkownika, edytujemy plik
`/etc/freeradius/users` :

    "morfik"        Cleartext-Password := "morfik-ma-kota", Simultaneous-Use := '1'

Jeśli teraz ten użytkownik spróbuje się zalogować na drugim urządzeniu mając przy tym już jedną
aktywną sesję, nowa sesja nie zostanie ustanowiona.

### Filtrowanie nazw użytkowników (moduł filter_username)

W celu zapewnienia rozsądnych nazw użytkowników używanych przy logowaniu do sieci WiFi, możemy
wykorzystać moduł `filter_username` . Jeśli trafią się zapytania, które będą pasować do wzorca
zdefiniowanego w `/etc/freeradius/policy.conf` , freeradius ich nie zaakceptuje. Jeśli chcemy takiej
weryfikacji, to w pliku `/etc/freeradius/sites-enabled/default` w sekcji `authorize` musimy
odkomentować:

    ...
    authorize {
       filter_username
    ...

Jeśli nie zadowalają nas domyślne reguły filtra ustawione w `/etc/freeradius/policy.conf` , możemy
część z nich wykomentować, a także zdefiniować własne filtry. Ja korzystam z tego poniższego:

    filter_username {
                #
                #  reject mixed case
                #  e.g. "UseRNaMe"
                #
                if (User-Name != "%{tolower:%{User-Name}}") {
                            reject
                }

                #
                #  reject all whitespace
                #  e.g. "user@ site.com", or "us er", or " user", or "user "
                #
                if (User-Name =~ / /) {
                            update reply {
                                        Reply-Message += "Rejected: Username contains whitespace"
                            }
                            reject
                }

                #
                #  reject Multiple @'s
                #  e.g. "user@site.com@site.com"
                #
                if(User-Name =~ /@.*@/ ) {
                            update reply {
                                        Reply-Message += "Rejected: Multiple @ in username"
                            }
                            reject
                }

                #
                #  reject double dots
                #  e.g. "user@site..com"
                #
                if (User-Name =~ /\.\./ ) {
                            update reply {
                                        Reply-Message += "Rejected: Username comtains ..s"
                            }
                            reject
                }

                #
                #  must have at least 1 string-dot-string after @
                #  e.g. "user@site.com"
                #
                if (User-Name !~ /@(.+)\.(.+)$/)  {
                            update reply {
                                        Reply-Message += "Rejected: Realm does not have at least one dot seperator"
                            }
                            reject
                }

                #
                #  Realm ends with a dot
                #  e.g. "user@site.com."
                #
                if (User-Name =~ /\.$/)  {
                            update reply {
                                        Reply-Message += "Rejected: Realm ends with a dot"
                            }
                            reject
                }

                #
                #  Realm begins with a dot
                #  e.g. "user@.site.com"
                #
                if (User-Name =~ /@\./)  {
                            update reply {
                                        Reply-Message += "Rejected: Realm begins with a dot"
                            }
                            reject
                }
    }

## Testowanie konfiguracji

Zaleca się by testować niewielkie zmiany dokonywane w konfiguracji freeradius'a. Także jeśli coś z
powyższego nie działa, najlepiej powrócić domyślne ustawienia całej konfiguracji i jeszcze raz
wprowadzić zmiany, tym razem mniejsze i po każdej zmianie dokonać sprawdzenia poprawności
konfiguracji opisanymi tutaj sposobami.

Jak już wspomniałem wcześniej, freeradius może zostać odpalony w trybie debug przy pomocy poniższego
polecenia:

    # freeradius -X

Im więcej `X` sprecyzujemy, tym bardziej szczegółowe będą logi, maksymalnie można ustawić trzy (
`-XXX` ). Weryfikacja plików konfiguracyjnych odbywa się przez sprecyzowanie opcji `-C` ,
przykładowo:

    root@radius:~# freeradius -XXXC
    ...
    Sun Sep 14 13:54:37 2014 : Debug: Configuration appears to be OK.

Jeśli wystąpią jakieś problemy i freeradius nie będzie się chciał nam odpalić, wtedy wystarczy
zajrzeć w log i tam z pewnością będzie wypisane co nawaliło i nawet jak to poprawić.

To mniej więcej tyle jeśli chodzi o debug serwera ale możemy także zajrzeć w logi klienta. Narzędzie
wpasupplicant wypluwa też pokaźne logi. Poniżej znajdują się dwie linijki, które trzeba wpisać jedna
po drugiej w osobnych terminalach,
    przykładowo:

    # /sbin/wpa_supplicant -dd -P /var/run/wpa_supplicant.wlan1.pid -i wlan1 -W -b bond0 -D nl80211,wext -c /etc/wpa_supplicant/wpa_supplicant.conf
    ...
    CTRL_IFACE - wlan1 - wait for monitor to attach

I teraz wpisujemy drugą z
    linijek:

    # /sbin/wpa_cli -P /var/run/wpa_action.wlan0.pid -i wlan0 -p /var/run/wpa_supplicant -a /sbin/wpa_action

Po jej wpisaniu powinno się rozpocząć podłączanie klienta do sieci WiFi i powinien zostać
wyświetlony obszerny log na pierwszej konsoli, w którym można prześledzić wszystkie etapy
połączenia, na wypadek gdyby coś poszło nie tak.

### Narzędzia testowe freeradius'a

Freeradius dostarcza kilka narzędzi testowych. Są one zawarte w pakiecie `freeradius-utils` . Nas
głównie interesują dwa z nich `radclient` oraz `radtest` , z tym, że `radtest` jest nakładką na
`radclient`. Oba programy się używa nieco inaczej ale każdy z nich dostarcza tych samych informacji
na temat przesyłanych pakietów.

Poniżej jest przedstawiony prosty test logowania się do sieci z wykorzystaniem obu wspomnianych
narzędzi:

    $ radtest morfik tajne-haslo 192.168.1.166 0 test-test
    Sending Access-Request of id 248 to 192.168.1.166 port 1812
            User-Name = "morfik"
            User-Password = "tajne-haslo
            NAS-IP-Address = 192.168.1.150
            NAS-Port = 0
            Message-Authenticator = 0x00000000000000000000000000000000
    rad_recv: Access-Accept packet from host 192.168.1.166 port 1812, id=248, length=20

    $ echo "User-Name = morfik, User-Password = tajne-haslo" | radclient -x 192.168.1.166 auth test-test
    Sending Access-Request of id 200 to 192.168.1.166 port 1812
            User-Name = "morfik"
            User-Password = "tajne-haslo"
    rad_recv: Access-Accept packet from host 192.168.1.166 port 1812, id=200, length=20

Jak widać wyżej, `radtest` jest nieco prostszy w użyciu. Zwykle z powyższych narzędzi będzie się
korzystać na maszynie, na której jest zainstalowane oprogramowanie freeradius i trzeba się upewnić,
że serwer freeradius'a nie ma sprecyzowanego interfejsu i adresu w pliku
`/etc/freeradius/radiusd.conf` oraz, że w pliku `/etc/freeradius/clients.conf` (lub w bazie danych)
widnieje wpis zezwalający przyjmować żądania z adresu `127.0.0.1` , czyli z lokalnej maszyny. Jeśli
któryś z powyższych warunków nie zostanie spełniony, testy się nie powiodą.

#### Test konfiguracji EAP

Oprogramowanie freeradius nie jest w stanie przeprowadzić testów EAP. Do tego celu potrzebne nam
będzie narzędzie `eapol_test` , które jest dostarczane z pakietem wpasupplicant. Problem w tym, że
domyślnie opcje od `eapol_test` są zahashowane i by móc używać tego narzędzia, trzeba je sobie
samemu skompilować. Inny problem w tym, że mi na debianie nie udało się tego narzędzia zbudować, bo
przy kompilacji ciągle mi wyrzucało jakiegoś wrednego błęda. W każdym razie, opis kompilacji jak i
samego narzędzia można znaleźć [pod tym linkiem][9].


[1]: http://wiki.openwrt.org/toh/tp-link/tl-wr1043nd
[2]: /post/generowanie-certyfikatow/
[3]: /post/generowanie-certyfikatow-przy-pomocy-easy-rsa/
[4]: https://blogs.msdn.microsoft.com/kaushal/2010/11/04/various-ssltls-certificate-file-typesextensions/
[5]: https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters
[6]: /post/konfiguracja-polaczenia-wifi-pod-debianem/
[7]: http://www.networkworld.com/article/2223672/access-control/which-eap-types-do-you-need-for-which-identity-projects.html
[8]: http://deployingradius.com/documents/protocols/compatibility.html
[9]: http://deployingradius.com/scripts/eapol_test/
