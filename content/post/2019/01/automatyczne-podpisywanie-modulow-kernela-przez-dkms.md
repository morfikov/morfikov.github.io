---
author: Morfik
categories:
- Linux
date: "2019-01-05T04:10:10Z"
lastmod: 2020-03-15 23:37:00 +0100
published: true
status: publish
tags:
- debian
- dkms
- kernel
- moduły-kernela
title: Automatyczne podpisywanie modułów kernela przez DKMS
---

Budując kernel linux'a trzeba zastanowić się nad kwestią modułów, które nie są wbudowane
bezpośrednio w samo jądro operacyjne. Nie chodzi tutaj bezpośrednio o funkcjonalność kernela,
którą można zbudować jako moduł w procesie jego kompilacji ale raczej o wszystkie zewnętrzne
moduły, które przez zespół kernela są traktowane jako `out-of-tree` . By poprawić nieco
bezpieczeństwo związane z takimi modułami, można wdrożyć podpisy cyfrowe, które takie moduły
muszą okazać podczas próby załadowania ich w systemie. Gdy moduł nie został podpisany, to kernel
go nie załaduje zwracając przy tym błąd `modprobe: ERROR: could not insert 'module': Required key
not available` . W ten sposób można ochronić się przed częścią ataków, w których moduły pochodzenia
trzeciego mogą zostać załadowane i poczynić nam ogromne spustoszenie w systemie. Problem w tym, że
w dystrybucji Debian wykorzystywany jest mechanizm DKMS (Dynamic Kernel Module Support). Może i
mamy dzięki niemu możliwość instalacji w systemie całej masy dodatkowych modułów ale ich również
nie będzie można załadować, bo nie zostały podpisane kluczem, którego certyfikat został zaszyty
w kernelu. Można jednak zmusić mechanizm DKMS, by wskazane przez nas moduły podpisywał
automatycznie przy ich instalacji i aktualizacji.

<!--more-->
## Opcjonalne podpisy modułów

Technicznie rzecz ujmując, to w konfiguracji linux'a mamy możliwość określenia czy żądać podpisów
przy ładowaniu modułów, a jeśli tak to czy ich wymagać. Oczywiście nie musimy wymagać, by ładowane
moduły były podpisane ale w takiej sytuacji system nam zwróci poniższą informację (sam moduł
zostanie jak najbardziej załadowany):

    kernel: module verification failed: signature and/or required key missing - tainting kernel

Dla niektórych osób tego typu rozwiązanie może być wyjściem, choć jest ono w pewnych sytuacjach
mało bezpieczne. Można co prawda zwiększyć tutaj nieco bezpieczeństwo przez zainteresowanie się
poniższym parametrem `sysctl` :

    kernel.modules_disabled = 1

Gdy ten parametr zostanie dopisany do pliku `/etc/sysctl.conf` , to chwilę po starcie systemu,
żadne moduły już nie będą mogły być załadowane do momentu restartu maszyny. Niemniej jednak, w
dalszym ciągu nasz system jest narażony na załadowanie trefnych modułów podczas początkowej fazy
startu systemu. Jeśli chcemy wyeliminować to okno, to trzeba wymusić na kernelu, by akceptował
jedynie tylko te moduły, które zostały podpisane tylko wyłączenie naszym kluczem prywatnym, tj.
przez ustawienie tych poniższych opcji:

    # zcat /proc/config.gz | grep -i module_sig
    CONFIG_MODULE_SIG=y
    CONFIG_MODULE_SIG_FORCE=y
    CONFIG_MODULE_SIG_ALL=y
    CONFIG_MODULE_SIG_SHA512=y
    CONFIG_MODULE_SIG_HASH="sha512"
    CONFIG_MODULE_SIG_KEY="/etc/kernel_key/signing_key.pem"

## Generowanie klucza do podpisywania modułów

Kompilując kernel z ustawioną opcją `CONFIG_MODULE_SIG` , klucz jest generowany automatycznie i
przechowywany w katalogu `certs/` patrząc względem katalogu ze źródłami kernela, zwykle
`/usr/src/linux-source-*/` . Klucz prywatny możemy też sobie stworzyć sami i podać go w parametrze
`CONFIG_MODULE_SIG_KEY` . W ten sposób kernel już nie będzie automatycznie tworzył klucza, np. w
momencie, gdy będziemy kompilować nowszą wersję kernela.

Stwórzmy sobie zatem katalog roboczy i przejdźmy do niego:

    # mkdir /etc/kernel_key/
    # chmod 700 /etc/kernel_key/
    # cd /etc/kernel_key/

Następnie kopiujemy do tego katalogu plik `certs/x509.genkey` ze źródeł kernela:

    # /usr/src/linux-source-4.20/certs/x509.genkey ./

Edytujemy ten plik i uzupełniamy, by wyglądał mniej więcej tak:

    [ req ]
    default_bits = 4096
    distinguished_name = req_distinguished_name
    prompt = no
    string_mask = utf8only
    x509_extensions = myexts

    [ req_distinguished_name ]
    C = IT
    L = LocalHost
    O = Morfikownia
    CN = morfikov's kernel-signing key
    emailAddress = morfik@nsa.com

    [ myexts ]
    basicConstraints=critical,CA:FALSE
    keyUsage=digitalSignature
    subjectKeyIdentifier=hash
    authorityKeyIdentifier=keyid

Zapisujemy plik i wpisujemy w terminal poniższe polecenie:

    # openssl req -new -nodes -utf8 -sha256 -days 36500 -batch -x509 \
       -config x509.genkey -outform PEM -out signing_key.pem \
       -keyout signing_key.pem
    Generating a RSA private key
    ............++++
    .......................++++
    writing new private key to 'kernel_key.pem'
    -----

W katalogu roboczym powinniśmy mieć dwa dodatkowe pliki: `signing_key.pem` (klucz prywatny) oraz
`signing_key.x509` (certyfikat).

## Sprawdzenie certyfikatu

Po skompilowaniu i zainstalowaniu kernela, uruchamiamy system ponownie i patrzymy w log, czy w
kernelu został zaszyty certyfikat, który wygenerowaliśmy wyżej:

    # dmesg | grep -i 'x.*509'
    Asymmetric key parser 'x509' registered
    Loading compiled-in X.509 certificates
    Loaded X.509 cert 'morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633'

Można też zajrzeć do pliku `/proc/keys` :

    # cat /proc/keys | egrep "X509|builtin"
    055e9058 I------     1 perm 1f010000     0     0 asymmetri morfikov's kernel-signing key: 8d9ac6d38bbf26e45a55454c54c448059f530633: X509.rsa 9f530633 []
    1833ebdf I------     1 perm 1f0b0000     0     0 keyring   .builtin_trusted_keys: 1

Widać zatem, że kernel posiada jeden zaufany klucz, którego odcisk wskazuje na ten użyty przy
podpisywaniu modułów podczas kompilacji kernela.

Gdybyśmy w tej chwili próbowali załadować jakiś zewnętrzny moduł, np. któryś dostarczany wraz z
paczkami DKMS, to kernel nam na to nie pozwoli:

    # modprobe sysdig-probe
    modprobe: ERROR: could not insert 'sysdig-probe': Required key not available

## Jak ustalić czy moduł jest podpisany

Praktycznie każdy moduł zewnętrzny, który będziemy instalować w systemie nie będzie podpisany.
Możemy ten fakt zweryfikować na kilka sposobów. Najprościej jest po prostu odpalić `modinfo` i
poszukać informacji o sygnaturach i kluczach użytych w procesie podpisywania:

    # modinfo sysdig-probe | grep '^sig'

W przypadku, gdy powyższe polecenie nam nic nie zwróci, to informacji o podpisie brak, zatem moduł
nie został podpisany i dlatego też nie może zostać załadowany.

Podpisany moduł prezentowałby się mniej więcej tak:

    # modinfo sysdig-probe | grep '^sig'
    sig_id:         PKCS#7
    signer:         morfikov's kernel-signing key
    sig_key:        41:72:D5:0F:5C:43:2C:B4:D7:90:E7:6C:9D:07:74:7F:57:C6:7F:C7
    sig_hashalgo:   sha512
    signature:      76:C3:FC:CD:99:2E:49:11:A2:E2:EA:38:43:4A:9D:58:BC:7B:AD:9D:

Jakiś czas temu był [błąd w kmod][2], przez który system nie był w stanie poprawnie zinterpretować
danych dotyczących podpisu i zwracał coś na poniższy wzór:

    # modinfo sysdig-probe | grep '^sig'
    sig_id:         PKCS#7
    signer:
    sig_key:
    sig_hashalgo:   md4
    signature:      30:82:02:A5:06:09:2A:86:48:86:F7:0D:01:07:02:A0:82:02:96:30:

Brakowało tutaj klucza, a algorytm użyty wskazywał na `md4` , który od lat już nie jest używany.
Nawet w przypadku kernela, który tego algorytmu nie ma wkompilowanego, zostanie zwrócony tego typu
komunikat, co może sprawiać mylne wrażenie, że moduł nie został podpisany albo został podpisany
błędnie lub jakoś niewłaściwie. Warto o tym pamiętać, gdy mamy do czynienia ze starszą wersją
`kmod` .

By się upewnić czy moduł jest podpisany, możemy skorzystać z `hexdump` , bo [dokumentacja kernela
jednoznacznie wskazuje][3], że podpisane moduły mają sygnaturę doczepianą na końcu pliku `.ko` :

    # hexdump -C /lib/modules/4.20.0-amd64-morficzny/updates/dkms/sysdig-probe.ko | tail
    000a91a0  08 00 00 00 00 00 00 00  18 00 00 00 00 00 00 00  |................|
    000a91b0  09 00 00 00 03 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a91c0  00 00 00 00 00 00 00 00  28 48 09 00 00 00 00 00  |........(H......|
    000a91d0  b6 13 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a91e0  01 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a91f0  11 00 00 00 03 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a9200  00 00 00 00 00 00 00 00  f0 86 0a 00 00 00 00 00  |................|
    000a9210  7f 01 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a9220  01 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
    000a9230

Brak wzmianki na temat sygnatury, zatem moduł nie jest podpisany. Gdyby sygnatura była, to powyższe
wyjście wyglądałoby mniej więcej tak:

    # hexdump -C /lib/modules/4.20.0-amd64-morficzny/updates/dkms/sysdig-probe.ko | tail
    000a9480  f8 d9 fe 6d b2 28 0f bc  ba f6 62 cd 8a 58 6b a0  |...m.(....b..Xk.|
    000a9490  d6 df 3e 7e fa 42 7b f8  a9 b1 83 6d 3b f9 12 aa  |..>~.B{....m;...|
    000a94a0  6f 80 c8 09 9c 4a e8 76  31 32 f7 e4 c5 87 87 43  |o....J.v12.....C|
    000a94b0  a2 52 6f 53 5d 51 43 e9  03 de c8 1e a7 f1 ba db  |.RoS]QC.........|
    000a94c0  16 2b d6 6f 72 60 65 3a  79 a5 b8 7d f4 03 2e b7  |.+.or`e:y..}....|
    000a94d0  12 a5 e9 9e de 8a 55 23  8d 00 00 02 00 00 00 00  |......U#........|
    000a94e0  00 00 00 02 a9 7e 4d 6f  64 75 6c 65 20 73 69 67  |.....~Module sig|
    000a94f0  6e 61 74 75 72 65 20 61  70 70 65 6e 64 65 64 7e  |nature appended~|
    000a9500  0a                                                |.|
    000a9501

Informacja na końcu, tj. `~Module signature appended~` jednoznacznie wskazuje, że moduł został
podpisany, choć nie wiadomo jakim kluczem.

Sygnaturę modułu można również usunąć przy pomocy `strip` :

    # strip --strip-debug /lib/modules/4.20.0-amd64-morficzny/updates/dkms/sysdig-probe.ko

## Podpisywanie modułów DKMS

Wszystko czego nam trzeba przy podpisywaniu modułów DKMS, to klucz prywatny, który już sobie
wygenerowaliśmy. By podpisać przykładowy moduł DKMS instalujemy stosowny pakiet z modułem oraz w
terminalu wydajemy poniższe polecenie:

    # /usr/src/linux-source-4.20/scripts/sign-file \
        sha512 \
        signing_key.pem \
        signing_key.x509 \
        /lib/modules/4.20.0-amd64-morficzny/updates/dkms/sysdig-probe.ko

Wykorzystane wyżej polecenie przyjmuje cztery argumenty. Pierwszym z nich jest rodzaj użytego
hash'a, który niekoniecznie musi pasować do tego algorytmu używanego przy kompilacji kernela. Drugi
argument, to ścieżka do klucza prywatnego. Trzeci zaś to ścieżka do certyfikatu (klucza
publicznego), a ostatnim argumentem jest naturalnie ścieżka do podpisywanego modułu.

Tak podpisany moduł przetrwa w naszym systemie jedynie do momentu aktualizacji paczki DKMS. Dlatego
też trzeba zastosować nieco inne podejście przy podpisywaniu modułów.

## Konfiguracja DKMS na potrzeby podpisywania modułów kernela

Pliki konfiguracyjne DKMS znajdują się w katalogu `/etc/dkms/` . W tym katalogu tworzymy plik
`kernel-module.conf` i dodajemy do niego poniższą treść:

    POST_BUILD=../../../../../../etc/kernel_key/sign-kernel.sh

Następnie tworzymy skrypt `sign-kernel.sh` w katalogu `/etc/kernel_key/` i wrzucamy do niego
poniższy kod:

    #!/bin/sh

    [ -z "$kernelver" ] && kernelver=$(uname -r)
    [ -z "$arch" ] && arch=$(arch)

    cd ../$kernelver/$arch/module/

    for kernel_object in *.ko; do
        echo "Signing kernel_object: $kernel_object"
          /usr/src/linux-headers-$kernelver/scripts/sign-file \
          sha512 \
          /etc/kernel_key/signing_key.pem \
          /etc/kernel_key/signing_key.x509 \
          "$kernel_object";
    done

Skryptowi nadajemy naturalnie prawa do wykonywania:

    # chmod 700 /etc/kernel_key/sign-kernel.sh

Następnie tworzymy dowiązania symboliczne do tego skryptu dla każdej paczki DKMS jaką mamy w
systemie. Interesują nas nazwy podfolderów, które są widoczne w katalogu `/var/lib/dkms/` . W tym
przypadku mamy tam: `sysdig/` oraz `xtables-addons/` , zatem tworzymy dwa dowiązania (dowiązania
symboliczne muszą mieć sufiks `.conf` ):

    # ln -s /etc/dkms/kernel-module.conf /etc/dkms/sysdig.conf
    # ln -s /etc/dkms/kernel-module.conf /etc/dkms/xtables-addons.conf

Każdy link odpowiada za konkretny moduł DKMS, tj. jeśli byśmy stworzyli tylko dowiązanie do
`sysdig` , to pliki `*.ko` z `xtables-addons` nie zostałyby podpisane. Dlatego też jeśli w
systemie znajdzie się kolejny moduł DKMS, to bez tej dodatkowej konfiguracji nie zostanie
automatycznie podpisany i nie będzie można go używać OOTB.

### Budowanie modułu via DKMS i brak klucza prywatnego

Brak klucza nie powstrzyma mechanizmu DKMS przed zbudowaniem modułu ale z racji, że wymagany jest
podpis, to nawet jeśli jakiś złowrogi proces w naszym systemie już zbuduje sobie moduł, to w żaden
sposób go nie załaduje. Możemy sprawdzić jak tego typu sytuacja wygląda w praktyce budując
przykładowy moduł ręcznie.

Na początku trzeba jednak usunąć stary moduł, który został wgrany za sprawą instalacji paczki
`-dkms.deb` :

    # dkms remove sysdig/0.24.1 --all

Poniżej jest wariant budowania i instalacji modułu DKMS bez dostępu do klucza prywatnego:

    # dkms install sysdig/0.24.1 -k 4.20.0-amd64-morficzny

    Creating symlink /var/lib/dkms/sysdig/0.24.1/source ->
                     /usr/src/sysdig-0.24.1

    DKMS: add completed.

    Kernel preparation unnecessary for this kernel.  Skipping...

    Building module:
    make -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build clean
    make: Entering directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'
    make: Leaving directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'

    { make -j2 KERNELRELEASE=4.20.0-amd64-morficzny -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build; } >> /var/lib/dkms/sysdig/0.24.1/build/make.log 2>&1


    Warning: The post_build script is not executable.
    make -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build clean
    make: Entering directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'
      CLEAN   /var/lib/dkms/sysdig/0.24.1/build/.tmp_versions
      CLEAN   /var/lib/dkms/sysdig/0.24.1/build/Module.symvers
    make: Leaving directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'


    DKMS: build completed.

    sysdig-probe.ko:
    Running module version sanity check.
     - Original module
       - No original module exists within this kernel
     - Installation
       - Installing to /lib/modules/4.20.0-amd64-morficzny/updates/dkms/

    do_depmod 4.20.0-amd64-morficzny


    DKMS: install completed.

Mamy tam informację `Warning: The post_build script is not executable` , czyli, że skrypt, który ma
zostać wykonany po procesie budowy modułu nie ma atrybutu wykonywalności. W sumie ten komunikat
jest nieco mylący ale oddaje stan faktyczny, bo przecie skryptu, którego nie ma w systemie również
nie da rady wykonać. Jeśli teraz taki moduł byśmy spróbowali załadować, to kernel się zbuntuje, a
przecie dokładnie o takie jego zachowanie nam chodzi.

Poniżej zaś jest wariant budowy modułu DKMS, gdy mamy dostęp do klucza:

    # dkms install sysdig/0.24.1

    Creating symlink /var/lib/dkms/sysdig/0.24.1/source ->
                     /usr/src/sysdig-0.24.1

    DKMS: add completed.

    Kernel preparation unnecessary for this kernel.  Skipping...

    Building module:
    make -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build clean
    make: Entering directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'
    make: Leaving directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'

    { make -j2 KERNELRELEASE=4.20.0-amd64-morficzny -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build; } >> /var/lib/dkms/sysdig/0.24.1/build/make.log 2>&1


    Running the post_build script:
    Signing kernel_object: sysdig-probe.ko
    make -C /lib/modules/4.20.0-amd64-morficzny/build M=/var/lib/dkms/sysdig/0.24.1/build clean
    make: Entering directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'
      CLEAN   /var/lib/dkms/sysdig/0.24.1/build/.tmp_versions
      CLEAN   /var/lib/dkms/sysdig/0.24.1/build/Module.symvers
    make: Leaving directory '/usr/src/linux-headers-4.20.0-amd64-morficzny'


    DKMS: build completed.

    sysdig-probe.ko:
    Running module version sanity check.
     - Original module
       - No original module exists within this kernel
     - Installation
       - Installing to /lib/modules/4.20.0-amd64-morficzny/updates/dkms/

    do_depmod 4.20.0-amd64-morficzny


    DKMS: install completed.

I tu już widzimy informację `Signing kernel_object: sysdig-probe.ko` i to w tym momencie właśnie
skrypt podpisuje moduł DKMS naszym kluczem prywatnym.

## Automatyczne podpisywanie modułów, a bezpieczeństwo systemu

Problem z tego typu rozwiązaniem jest taki, że moduły mogą być podpisywane transparentnie. Jeśli
jakiś proces stworzy sobie dowiązania opisane wyżej i będzie próbował zainstalować moduł za pomocą
mechanizmu DKMS, to taki moduł zostanie podpisany, co otworzy mu drogę do bycia załadowanym przez
kernel już bez jakiejkolwiek kontroli i tego faktu nawet nie będziemy świadomi. To rozwiązanie może
dawać złudne i fałszywe poczucie bezpieczeństwa w przypadku, gdy klucz prywatny służący do
podpisywania modułów będzie przechowywany w systemie, którego moduły kernela ma podpisywać, nawet
jeśli do klucza będzie miał dostęp jedynie root.

Rozwiązaniem tego problemu jest naturalnie przeniesienie klucza prywatnego na jakiś zewnętrzny
nośnik, z racji, że klucz prywatny służący do pospisywania modułów będzie wykorzystywany w
zasadzie tylko przy aktualizacji pakietów (ewentualnie też przy ręcznej instalacji modułów DKMS).

[2]: https://bugzilla.redhat.com/show_bug.cgi?id=1320921
[3]: https://www.kernel.org/doc/html/v4.17/admin-guide/module-signing.html#signed-modules-and-stripping
