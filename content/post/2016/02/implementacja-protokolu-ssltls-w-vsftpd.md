---
author: Morfik
categories:
- Linux
date: "2016-02-13T22:30:54Z"
date_gmt: 2016-02-13 21:30:54 +0100
published: true
status: publish
tags:
- debian
- sieć
- ftp
- ssl
- tls
GHissueID: 517
title: Implementacja protokołu SSL/TLS w vsftpd
---

Kwestię [konfiguracji serwera FTP na debianie w oparciu o
vsftpd](/post/konfiguracja-vsftpd-w-debianie/) już przerabialiśmy. Została nam
jeszcze do omówienia implementacja protokołu SSL/TLS. FTP nie jest bezpiecznym protokołem i wszelkie
dane logowania są przesyłane przez sieć otwartym tekstem. W przypadku, gdy stawiamy lokalny serwer
FTP w zaufanej sieci lub też będziemy korzystać jedynie z dostępu anonimowego, to raczej nie
potrzebujemy szyfrować danych. Trzeba pamiętać, że każde szyfrowanie dość mocno obciąża procesor,
który może stanowić wąskie gardło przy przesyle danych. W tym wpisie założenie jest takie, że
bezpieczeństwo danych, które będziemy przesyłać za pomocą protokołu FTP, jest rzeczą najważniejszą i
dlatego wdrożyć szyfrowanie.

<!--more-->
## Certyfikat i klucze szyfrujące

Implementacje protokołu SSL/TLS w `vsftpd` rozpoczynamy od wygenerowania certyfikatu i kluczy
szyfrujących. [Generowanie certyfikatów](/post/generowanie-certyfikatow/) możemy
przeprowadzić ręcznie krok po kroku. Możemy też skorzystać z [narzędzia
easy-rsa](/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/) lub [certtool
(pokazane na przykładzie
rsyslog'a)](/post/zaszyfrowane-logi-w-rsyslog-i-syslog-ng/). Poniżej skorzystamy z
tej metody oferowanej przez narzędzie `certtool` dostępne w debianie w pakiecie `gnutls-bin` .

Nie będę tutaj tworzył od nowa całego [CA](https://pl.wikipedia.org/wiki/Urz%C4%85d_certyfikacji),
bo ten proces został dokładnie opisany w podlinkowanych wyżej artykułach. Tutaj stworzymy jedynie
potrzebne nam elementy. Przechodzimy zatem do katalogu `/etc/CA/` i tworzymy klucz RSA:

    # cd /etc/CA/
    # certtool --generate-privkey \
          --rsa \
          --sec-param high \
          --outfile server-vsftpd-morfikownia.lh.key

Tworzymy także plik `/etc/CA/template/server-vsftpd-morfikownia.lh` , w którym to definiujemy
konfigurację dla
    OpenSSL:

    dn = "C=PL,ST=Mazowieckie,L=Warsaw,CN=morfikownia.lh,O=Morfikownia,OU=Security dept.,DC=morfikownia.lh,UID=morfik"
    serial = 010
    expiration_days = 3650
    ip_address = "192.168.1.150"
    email = "morfik@nsa.com"
    tls_www_server
    signing_key
    encryption_key

Przykładową konfigurację certyfikatu można znaleźć w katalogu
`/usr/share/doc/gnutls-bin/examples/` . Sam certyfikat generujemy w poniższy sposób:

    # certtool --generate-certificate \
          --template template/server-vsftpd-morfikownia.lh \
          --load-privkey server-vsftpd-morfikownia.lh.key \
          --load-ca-certificate ca_192.168.1.150.crt \
          --load-ca-privkey ca_192.168.1.150.key \
          --outfile server-vsftpd-morfikownia.lh.crt \
          --hash sha512

Informacje o kluczu jak i certyfikacie serwera FTP możemy podejrzeć za pomocą poniższych poleceń:

    # certtool --key-info --infile server-vsftpd-morfikownia.lh.key
    # certtool --certificate-info --infile server-vsftpd-morfikownia.lh.crt

Możemy także wykorzystać do tego celu `gcr-viewer` (pakiet `gcr` ) :

![](/img/2016/02/1.certyfikat-vsftpd-gcr-viewer.png#medium)

## Konfiguracja SSL/TLS w vsftpd

Zatem mamy przygotowane trzy pliki: certyfikat CA oraz klucz i certyfikat serwera FTP. Te pliki
będziemy musieli uwzględnić w konfiguracji `vsftpd` . Tworzymy zatem katalog `/etc/vsftpd/certs/` i
kopiujemy do niego te pliki:

    # cp /etc/CA/ca_192.168.1.150.crt /etc/vsftpd/certs/
    # cp /etc/CA/server-vsftpd-morfikownia.lh.* /etc/vsftpd/certs/

### Konfiguracja certyfikatów

Otwieramy teraz plik konfiguracyjny `/etc/vsfptd.conf` i dopisujemy w nim te poniższe parametry:

    ssl_enable=YES
    ca_certs_file=/etc/vsftpd/certs/ca_192.168.1.150.crt
    rsa_cert_file=/etc/vsftpd/certs/server-vsftpd-morfikownia.lh.crt
    rsa_private_key_file=/etc/vsftpd/certs/server-vsftpd-morfikownia.lh.key
    ssl_request_cert=YES
    require_cert=NO
    validate_cert=YES

Obsługę protokołu SSL/TLS w `vsftpd` włączamy za pomocą opcji `ssl_enable` . Następnie w parametrach
`ca_certs_file` , `rsa_cert_file` oraz `rsa_private_key_file` podajemy ścieżki do plików z
certyfikatami CA i serwera oraz kluczem prywatnym serwera. Certyfikat CA podajemy w przypadku, gdy
zamierzamy weryfikować certyfikaty podłączających się klientów. W przypadku, gdy certyfikat i klucz
prywatny znajdują się w tym samym pliku, to ścieżkę do niego podajemy w `rsa_cert_file` przy
jednoczesnym wykomentowaniu `rsa_private_key_file` .

Klienci mogą posługiwać się własnymi certyfikatami podpisanymi przez to samo CA, którego certyfikat
jest określony w opcji `ca_certs_file` . Możemy nakazać `vsftpd` , by żądał tych certyfikatów
podczas podłączania się użytkowników włączając opcję `ssl_request_cert` . Niekoniecznie ten
certyfikat musi być wymagany, co możemy określić w `require_cert` . W takim przypadku zarówno ci
klienci, którzy mają certyfikat, jak i ci, którzy go nie posiadają, zostaną wpuszczeni. Dobrze jest
także ustawić opcję `validate_cert` , gdy żądamy certyfikatów od klientów.

### Dozwolone protokoły i szyfry

Wyłączamy obsługę protokołów SSLv2 i SSLv3 przy pomocy parametrów `ssl_sslv2` oraz `ssl_sslv3` ,
zostawiając jedynie protokół TLS (`ssl_tlsv1`). Ustawiamy również szyfr SSL/TLS wykorzystywany do
zabezpieczenia połączenia w parametrze `ssl_ciphers` :

    ssl_sslv2=NO
    ssl_sslv3=NO
    ssl_tlsv1=YES
    ssl_ciphers=ECDHE-RSA-AES256-GCM-SHA384

Więcej informacji na temat szyfrów wykorzystanych w parametrze `ssl_ciphers` można znaleźć w [man
ciphers](http://manpages.ubuntu.com/manpages/wily/en/man1/ciphers.1ssl.html) lub też wpisując w
terminalu to poniższe polecenie:

    $ openssl ciphers -tls1 -v

### Wymuszanie szyfrowania

Za sprawą `vsftpd` możemy wymusić na użytkownikach, by korzystali z bezpiecznego połączenia zarówno
w przypadku logowania jak i przesyłania danych. Jako, że użytkowników dzielimy na dwie grupy
(anonimowi i lokalni), to dla każdej z tych grup mamy osobne opcje. W przypadku lokalnych
użytkowników wykorzystujemy `force_local_logins_ssl` oraz `force_local_data_ssl` . Natomiast w
przypadku anonimowych klientów używamy `force_anon_logins_ssl` oraz `force_anon_data_ssl` .
Dodatkowo, musimy wyraźnie zezwolić anonimowym na korzystanie z dobrodziejstw szyfrowania przy
pomocy opcji `allow_anon_ssl` :

    allow_anon_ssl=YES
    force_anon_logins_ssl=NO
    force_anon_data_ssl=NO
    force_local_logins_ssl=NO
    force_local_data_ssl=NO

### Różnice w metodach podłączenia (explicit vs. implicit)

W przypadku wykorzystywania szyfrowania SSL/TLS na FTP'ie, klienci mogą wykorzystać jedną z dwóch
metod podczas nawiązywania połączenia. Pierwszą z nich jest `Implicit FTP over SSL` , która szerzej
znana jest po prostu jako protokół FTPS. Przy nawiązywaniu połączenia, serwer i klient tworzą
szyfrowany tunel od razu na samym początku. Powoduje to problemy z klientami FTP, które nie
obsługują szyfrowania, np. Firefox. Tacy klienci nie będą w stanie się połączyć do FTP'a nawet
jeśli nie wymagamy od nich szyfrowania. Druga metoda zaś to `Explicit FTP over SSL` , w której
klient korzysta ze zwykłego protokołu FTP przy nawiązywaniu połączenia, po czym dopiero jest
ustanawiany tunel SSL/TLS. Ta metoda nie powoduje problemów z klientami, które nie chcą korzystać z
szyfrowania. W dalszym ciągu dane logowania jak i same dane mogą być szyfrowane. Jeśli `vsftpd` ma
wspierać obie metody podłączenia do FTP, to trzeba uruchomić dwa osobne procesy. My jednak
ograniczymy się jedynie do metody `Explicit FTP over SSL` :

    implicit_ssl=NO

Domyślnym portem dla SFTP jest `990` . W przypadku konfiguracji klientów musimy o tym fakcie
pamiętać. Dlatego też jeśli zamierzamy korzystać z metody `Implicit FTP over SSL` , to albo
zmieńmy domyślny port w ustawieniach `vsftpd` , albo zdefiniujmy port w konfiguracji klienta FTP i
ustawmy go na 21.

### Logowanie

`vsftpd` może także logować komunikaty SSL/TLS, co przydaje się na wypadek ewentualnych problemów z
połączeniem. By te wiadomości zaczęły nam się pokazywać w logu, musimy dodać do konfiguracji
`vsftpd` te poniższe opcje:

    xferlog_enable=YES
    xferlog_std_format=NO
    vsftpd_log_file=/var/log/vsftpd.log
    xferlog_file=/var/log/xferlog
    syslog_enable=NO
    log_ftp_protocol=YES
    debug_ssl=NO

Poniżej komunikaty z przykładowego logowania z wykorzystaniem metody `Explicit FTP over SSL` bez
certyfikatów klienckich:

![](/img/2016/02/2.log-vsftpd-ssl-tls-debug.png#huge)

## Test połączenia z vsftpd

Odpalamy teraz klienta FTP, w tym przypadku będzie to `filezilla` i próbujemy zalogować się
wykorzystując jedną z powyższych metod. Chwilę po wysłaniu żądania połączenia, powinien naszym oczom
pokazać się mniej więcej taki certyfikat:

![](/img/2016/02/3.vsftpd-certyfikat-polaczenie-filezilla.png#big)

Sprawdzamy go pod kątem poprawności i jeśli wszystko jest w porządku, to akceptujemy.

Cały plik konfiguracyjny `vsftpd.conf` znajduje się [na moim
github'ie](https://github.com/morfikov/files/blob/master/configs/etc/vsftpd.conf).
