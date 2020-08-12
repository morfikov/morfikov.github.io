---
author: Morfik
categories:
- Linux
date: "2015-10-08T14:29:34Z"
date_gmt: 2015-10-08 12:29:34 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- openssl
title: Generowanie certyfikatów przy pomocy easy-rsa
---

Jakiś czas temu przedstawiłem [manualny sposób na generowanie
certyfikatów]({{< baseurl >}}/post/generowanie-certyfikatow/), które można z powodzeniem
wykorzystać przy ssl, openvpn czy freeradius. Nie było tego znowu aż tak dużo ale jakby nie patrzeć
trochę parametrów trzeba znać, a najlepiej mieć przygotowane odpowiednie linijki, by sam proces
generowania certyfikatów przebiegł dość sprawnie. Jednak wychodzi na to, że nie trzeba się znowu aż
tak wysilać, bo istnieją dedykowane narzędzia, które wygenerują nam wszystkie potrzebne pliki. Mowa
o `easy-rsa` i to niego będzie dotyczył ten wpis.

<!--more-->
## Pakiet easy-rsa

W sumie, to tylko przypadek sprawił, że się natknąłem na pakiet `easy-rsa` . Po jego instalacji w
systemie, mamy dostępne narzędzie `make-cadir` , które po wskazaniu pustego katalogu, podlinkuje
szereg plików, które posłużą nam później przy generowaniu certyfikatów.

Tworzymy zatem katalog, w którym przechowywać będziemy certyfikaty, przechodzimy do niego i w nim
tworzymy podkatalog `/CA/` . Następnie wydajemy polecenie `make-cadir` podając w argumencie ten
powyższy folder:

    $ mkdir certs/
    $ cd ./certs/
    $ make-cadir ./CA/

Jeśli ostatnie polecenie nie zwróciło żadnych błędów, w katalogu `CA` powinniśmy mieć szereg
skryptów, przy pomocy których możemy wygenerować certyfikaty dla różnych usług sieciowych. Z tym,
że one chyba raczej były projektowane głównie na potrzeby
[openvpn](https://wiki.debian.org/OpenVPN). Przynajmniej pakiet `easy-rsa` się bardzo często pojawia
w tekstach o tworzeniu własnego VPN.

Zobaczmy zatem jak wygląda tworzenie certyfikatów przy pomocy dostarczonych nam skryptów. Na sam
początek edytujemy plik `vars` . Są w nim definiowane zmienne konfiguracyjne używane przy
generowaniu certyfikatów. Poniżej znajduje się zrzut mojej konfiguracji:

    # easy-rsa parameter settings

    # NOTE: If you installed from an RPM,
    # don't edit this file in place in
    # /usr/share/openvpn/easy-rsa --
    # instead, you should copy the whole
    # easy-rsa directory to another location
    # (such as /etc/openvpn) so that your
    # edits will not be wiped out by a future
    # OpenVPN package upgrade.

    # This variable should point to
    # the top level of the easy-rsa
    # tree.
    export EASY_RSA="`pwd`"

    #
    # This variable should point to
    # the requested executables
    #
    export OPENSSL="openssl"
    export PKCS11TOOL="pkcs11-tool"
    export GREP="grep"


    # This variable should point to
    # the openssl.cnf file included
    # with easy-rsa.
    #export KEY_CONFIG=`$EASY_RSA/whichopensslcnf $EASY_RSA`
    export KEY_CONFIG=$EASY_RSA/openssl-1.0.0.cnf


    # Edit this variable to point to
    # your soon-to-be-created key
    # directory.
    #
    # WARNING: clean-all will do
    # a rm -rf on this directory
    # so make sure you define
    # it correctly!
    export KEY_DIR="$EASY_RSA/keys"

    # Issue rm -rf warning
    echo NOTE: If you run ./clean-all, I will be doing a rm -rf on $KEY_DIR

    # PKCS11 fixes
    export PKCS11_MODULE_PATH="dummy"
    export PKCS11_PIN="dummy"

    # Increase this to 2048 if you
    # are paranoid.  This will slow
    # down TLS negotiation performance
    # as well as the one-time DH parms
    # generation process.
    export KEY_SIZE=4096

    # In how many days should the root CA key expire?
    export CA_EXPIRE=3650

    # In how many days should certificates expire?
    export KEY_EXPIRE=3650

    # These are the default values for fields
    # which will be placed in the certificate.
    # Don't leave any of these fields blank.
    export KEY_COUNTRY="US"
    export KEY_PROVINCE="CA"
    export KEY_CITY="SanFrancisco"
    export KEY_ORG="NSA"
    export KEY_EMAIL="morfik@nsa.com"
    export KEY_OU="Security Division"

    # X509 Subject Field
    export KEY_NAME="morfik"

    # PKCS11 Smart Card
    # export PKCS11_MODULE_PATH="/usr/lib/changeme.so"
    # export PKCS11_PIN=1234

    # If you'd like to sign all keys with the same Common Name, uncomment the KEY_CN export below
    # You will also need to make sure your OpenVPN server config has the duplicate-cn option set
    # export KEY_CN="CommonName"

I to w zasadzie tyle jeśli chodzi o nasz wkład w proces tworzenia certyfikatów. Na dobrą sprawę to
jedyną wartością, którą musimy dostosować sobie w procesie generowania certyfikatów, jest `Common
Name` . Teraz już tylko wydajemy poniższe polecenia:

    $ source ./vars
    $ ./clean-all
    $ ./build-ca
    Generating a 4096 bit RSA private key
    ...
    $ ./build-dh
    Generating DH parameters, 4096 bit long safe prime, generator 2
    This is going to take a long time
    ...
    $ ./build-key-server server
    Generating a 4096 bit RSA private key
    ...

Jeśli chodzi zaś o certyfikaty klienckie, to mamy trochę więcej opcji, bo możemy określić hasło oraz
istnieje też możliwość zapakowania certyfikatów w plik `.p12` . W zależności od potrzeb wpisujemy w
terminal nazwę odpowiedniego skryptu. W tym przypadku jest to:

    $ ./build-key-pkcs12 client
    Generating a 4096 bit RSA private key
    ...

## Szyfrowanie kluczy prywatnych

Taka mała uwaga -- wszystkie pliki `.key` nie są zaszyfrowane i pewnie w wielu przypadkach nie ma
potrzeby ich szyfrować. Jeśli jednak życzymy sobie kluczy w formie zaszyfrowanej, możemy je
zaszyfrować ręcznie po całym procesie generowania certyfikatów, przykładowo:

    $ openssl rsa -aes256 -in keys/client.key -out keys/client.key.encrypted
    writing RSA key
    Enter PEM pass phrase:
    Verifying - Enter PEM pass phrase:

A jeśli nie mamy pojęcia czy dany klucz jest zaszyfrowany czy też nie, możemy to sprawdzić przez:

    $ openssl rsa -text -noout -in keys/client.key
    Private-Key: (4096 bit)
    modulus:
    ...

Jeśli po wydaniu powyższego polecenia nie zostaniemy poproszeni o hasło, znaczy, że klucz nie jest
zaszyfrowany.
