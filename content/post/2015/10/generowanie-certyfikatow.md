---
author: Morfik
categories:
- Linux
date: "2015-10-04T18:11:50Z"
date_gmt: 2015-10-04 16:11:50 +0200
published: true
status: publish
tags:
- szyfrowanie
- openssl
title: Generowanie certyfikatów
---

Certyfikaty mogą zostać wykorzystane wszędzie tam, gdzie mamy do czynienia z protokołami
szyfrującymi ruch. Zwykle są to serwisy hostujące strony www ale też mogą to być i inne usługi, np.
OpenVPN. Można je także spotkać w sieciach bezprzewodowych gdzie wykorzystywany jest protokół
WPA2-Enterprise. Jeśli operujemy na linux'ie, to prawdopodobnie spotkaliśmy się już z
oprogramowaniem, które wykorzystuje certyfikaty, np. serwer `apache2` . Dobrze jest sobie zatem
przyswoić wiedzę na temat generowania certyfikatów.

<!--more-->
## Przygotowanie CA

CA co centrum certyfikacji, które zajmuje się wystawianiem certyfikatów. Wszelkie oprogramowanie
wymaga aby wystawione certyfikaty były podpisane przez jakieś CA. Zwykle tego typu usługa jest
płatna i do tego trwa jakiś skończony okres czasu, po którym trzeba certyfikat odnowić. Dlatego
jeśli eksperymentujemy z oprogramowaniem, możemy sobie stworzyć własne CA do celów testowych i to
nim podpisywać certyfikaty wykorzystywane przez rozmaite usługi.

Na sam początek potrzebna będzie nam odpowiednia struktura katalogów, którą możemy stworzyć w
poniższy sposób:

    # mkdir /etc/CA
    # mkdir /etc/CA/signed_certs
    # mkdir /etc/CA/private
    # chmod 700 /etc/CA/private
    # cp /etc/ssl/openssl.cnf /etc/CA
    # touch /etc/CA/index.txt

W katalogu `signed_certs` będą przechowywane kopie wszystkich certyfikatów, które podpiszemy naszym
CA. W takim przypadku gdy zajdzie potrzeba cofnięcia jakiegoś certyfikatu, będziemy mieć jego kopię
lokalnie. Z kolei w katalogu `private` będzie trzymany klucz prywatny naszego CA i bardzo ważną
rzeczą jest by trzymać ten klucz w sekrecie, bo jeśli zostanie skompromitowany, jest niemal pewne,
że zostanie użyty do podpisania niezaufanych certyfikatów, a to doprowadzi do sytuacji gdzie
użytkownik prześle poufne dane do podejrzanych maszyn. Plik `index.txt` jest to tekstowa baza
danych wykorzystywana do śledzenia podpisanych certyfikatów. Został również skopiowany plik
`openssl.cnf` , w którym to zostanie skonfigurowany szereg parametrów pod certyfikaty.

Przechodzimy do edycji pliku `/etc/CA/openssl.cnf` i zmieniamy w nim poniższe linijki:

    ...
    [ CA_default ]
    
    dir             = /etc/CA               # Where everything is kept
    certs           = $dir                  # Where the issued certs are kept
    ...
    new_certs_dir   = $dir/signed_certs     # default place for new certs.
    ...
    default_days    = 3650                  # how long to certify for
    ...
    default_md      = sha512                # use public key default MD
    ...
    [ req ]
    default_bits    = 4096
    ...
    [ req_distinguished_name ]
    countryName                     = Country Name (2 letter code)
    countryName_default             = US
    countryName_min                 = 2
    countryName_max                 = 2
    
    stateOrProvinceName             = State or Province Name (full name)
    stateOrProvinceName_default     = California
    
    localityName                    = Locality Name (eg, city)
    localityName_default            = Los Angeles
    
    0.organizationName              = Organization Name (eg, company)
    0.organizationName_default      = NSA
    
    # we can do this but it is not needed normally :-)
    #1.organizationName             = Second Organization Name (eg, company)
    #1.organizationName_default     = World Wide Web Pty Ltd
    
    organizationalUnitName          = Organizational Unit Name (eg, section)
    organizationalUnitName_default  =
    
    commonName                      = Common Name (e.g. server FQDN or YOUR name)
    commonName_max                  = 64
    
    emailAddress                    = Email Address
    emailAddress_default            = morfik@nsa.com
    emailAddress_max                = 64

Dodatkowo na końcu pliku dodajemy poniższe wpisy:

    [ xpclient_ext]
    extendedKeyUsage = 1.3.6.1.5.5.7.3.2
    
    [ xpserver_ext]
    extendedKeyUsage = 1.3.6.1.5.5.7.3.1

Na dobrą sprawę nie testowałem tego i nie mam pojęcia czy te rozszerzenia odnoszą się tylko do
windowsów xp czy w ogóle do wszystkich windowsów.

### Generowanie certyfikatu lokalnego CA

Mając przygotowane środowisko, możemy przejść do generowania certyfikatu dla CA. Wchodzimy zatem do
katalogu `/etc/CA/` i generujemy certyfikat:

    # cd /etc/CA
    # openssl req -new -keyout private/cakey.pem -out careq.pem -config ./openssl.cnf

Musimy go jeszcze sami sobie podpisać by stał się
    CA:

    # openssl ca -create_serial -out cacert.pem -keyfile private/cakey.pem -selfsign -extensions v3_ca -config ./openssl.cnf -in careq.pem

W powyższym poleceniu użyliśmy parametru `-create_serial` , który tworzy hexalny numer seryjny dla
tego klucza. Z kolei `-extensions` określa sekcję w pliku openssl.cnf , której należy szukać w celu
odnalezienia określonych rozszerzeń, które zostaną dodane do nowo stworzonego certyfikatu (klucza
publicznego). W tym przypadku użyliśmy sekcji `v3_ca` , która zawiera między innymi
`basicConstraints = CA:true` , co pozwala na podpisywanie innych kluczy, czyli by ten klucz
zachowywał się jak CA.

Ostatnim krokiem jest stworzenie kopi certyfikatu CA zakodowanej w formacie DER, no bo windowsy
lubią tylko tego typu binarne certyfikaty:

    # openssl x509 -inform PEM -outform DER -in cacert.pem -out cacert.der

## Generowanie certyfikatów dla serwera i klientów

No to generowanie certyfikatu CA mamy z głowy. Trzeba nam jeszcze wygenerować parę kluczy dla
serwerów i klientów.

    # openssl req -new -config ./openssl.cnf -keyout server_key.pem -out server_req.pem

Podpisujemy ten certyfikat naszym CA:

    # openssl ca -config ./openssl.cnf -in server_req.pem -out server_cert.pem

Jeśli byśmy mieli w zamiarze wykorzystywać windowsa do zarządzania, np. połączeniami WiFi, to trzeba
użyć rozszerzeń X509v3 i wtedy powyższa linijka będzie miała poniższą
    postać:

    # openssl ca -config ./openssl.cnf -extensions xpserver_ext -in server_req.pem -out server_cert.pem

Klucze dla klientów tworzy się w taki sam sposób jak dla serwera
    wyżej:

    # openssl req -new -config ./openssl.cnf -keyout morfik_laptop_key.pem -out morfik_laptop_req.pem

Teraz jeszcze zostało nam podpisanie certyfikatu klienta:

    # openssl ca -config ./openssl.cnf -in morfik_laptop_req.pem -out morfik_laptop_cert.pem

Jeśli mamy zamiar używać certyfikatów na windowsie, w powyższym poleceniu musimy uwzględnić
rozszerzenia X509v3 i zmienić powyższą linijkę by wyglądała tak jak tak
    poniżej:

    # openssl ca -config ./openssl.cnf -extensions xpclient_ext -in morfik_laptop_req.pem -out morfik_laptop_cert.pem

Jeśli chodzi jeszcze o windowsy to musimy przekuć klucz prywatny i certyfikat klienta w plik
PKCS\#12
    :

    # openssl pkcs12 -export -clcerts -in morfik_laptop_cert.pem -inkey morfik_laptop_key.pem -out morfik_laptop.p12

Powyższe polecenie wykorzystuje narzędzie `pkcs12` oferowane przez OpenSSL do wyeksportowania (
`-export` ) nowego pliku PKCS\#12. Z kolei `-clcerts` mówi OpenSSL aby wyeksportował jedynie
certyfikat i jego klucz prywatny, bo w innych konfiguracjach można upchnąć wiele certyfikatów i
kluczy w jednym pliku PKCS\#12. Tego typu certyfikat również jest czytany na maszynach linux'owych.
