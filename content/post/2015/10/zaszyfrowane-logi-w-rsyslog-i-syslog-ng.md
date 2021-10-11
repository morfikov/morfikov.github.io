---
author: Morfik
categories:
- Linux
date: "2015-10-08T13:53:38Z"
date_gmt: 2015-10-08 11:53:38 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- openssl
- rsyslog
GHissueID: 185
title: Zaszyfrowane logi w rsyslog i syslog-ng
---

Jakiś czas temu, na forum DUG'a wyczytałem coś o przesyłaniu logów systemowych przez sieć. W sumie,
to nigdy mi to do głowy nie przyszło ale jeśli by się nad tym głębiej zastanowić, tego typu
mechanizm może okazać się całkiem użyteczny. Na dobrą sprawę nie wiem jak to jest rozwiązane w
debianie opartym o systemd, natomiast jeśli chodzi o inne init'y (openrc i sysvinit), to tego typu
funkcjonalność można zaimplementować wykorzystując narzędzie `rsyslog` lub `syslog-ng` . W tym
wpisie zostanie opisana konfiguracja debianowego serwera, na którym będzie nasłuchiwał daemon
`rsyslog` . Dodatkowo, zostanie przedstawiona konfiguracja dwóch klientów, z których jeden będzie
miał zainstalowanego `syslog-ng` , a drugi `rsyslog` . Z klientów logi zostaną przesłane do serwera.
Dodatkowo, postaramy się [zaszyfrować ruch przy pomocy kanału
TLS](http://www.rsyslog.com/doc/v8-stable/tutorials/tls_cert_summary.html).

<!--more-->
## Przygotowanie maszyn

Co prawda, `syslog-ng` i `rsyslog` się troszeczkę różnią między sobą jeśli chodzi o konfigurację ale
każdy z nich może być klientem/serwerem dla tego drugiego. Przesył logów przez sieć to jedna sprawa,
a zabezpieczenie komunikacji by te logi nie szły po kablach w formie niezaszyfrowanej, to całkiem
inna bajka. Na szczęście, oba te narzędzia są w kwestii szyfrowania logów kompatybilne i mogą ze
sobą współpracować bez większego problemu.

Zanim jednak przejdziemy do konfiguracji demonów, musimy przygotować odpowiednio wszystkie maszyny.
Na serwerze oraz jednym kliencie instalujemy pakiet `rsyslog` . Dodatkowo, na tych dwóch maszynach
musimy doinstalować pakiety `rsyslog-gnutls` oraz `openssl` . Na drugim kliencie instalujemy pakiet
`syslog-ng` i także `openssl` .

Jeśli nie posiadamy obsługi protokołu ipv6, możemy skonfigurować rsyslog'a by nasłuchiwał tylko na
protokole ipv4. Edytujemy zatem plik `/etc/default/rsyslog` i ustawiamy odpowiedni parametr:

    RSYSLOGD_OPTIONS="-4"

## Generowanie certyfikatów

By zestawić kanał TLS, potrzebne nam są certyfikaty. W debianie jest dostępne narzędzie `easy-rsa` ,
które ułatwia znacząco operowanie na certyfikatach ale w przypadku `rsyslog` i `syslog-ng` nie
możemy z niego skorzystać. Trzeba używać `certtool` dostępnego w paczce `gnutls-bin` .

Dobrze jest przyjąć sobie [pewną konwencję przy nazywaniu
plików](https://help.ubuntu.com/community/GnuTLS). W tym przypadku pliki certyfikatów będą nazywane
`host.domena` + odpowiednia końcówka. Klucze prywatne będą mieć `.key` , a certyfikaty będą mieć
`.crt` . Wyjątkiem będzie jedynie CA, które będzie miało formę `ca.domena.{key,crt}` .

Narzędzie `certtool` przyjmuje predefiniowaną konfigurację dla certyfikatów. Dlatego też stwórzmy
cztery pliki tekstowe, po jednym dla CA, serwera i dla każdego klienta. Te pliki umieścimy w
katalogu `/etc/CA/template/` . Same certyfikaty zaś będą zlokalizowane w `/etc/CA/` . Poniżej wzory
plików pod certyfikaty:

Plik `/etc/CA/template/ca_192.168.1.150` :

    # X.509 Certificate options
    #
    # DN options

    # The organization of the subject.
    organization = "morfikownia"

    # The organizational unit of the subject.
    #unit = "sleeping dept."

    # The state of the certificate owner.
    #state = "Example"

    # The country of the subject. Two letter code.
    country = GB

    # The common name of the certificate owner.
    cn = "192.168.1.150"

    # The serial number of the certificate. Should be incremented each time a new certificate is generated.
    serial = 001

    # In how many days, counting from today, this certificate will expire.
    expiration_days = 3650

    # Whether this is a CA certificate or not
    ca

    # Whether this key will be used to sign other certificates.
    cert_signing_key

    # Whether this key will be used to sign CRLs.
    crl_signing_key

Plik `/etc/CA/template/client_192.168.1.1` :

    # X.509 Certificate options
    #
    # DN options

    # The organization of the subject.
    organization = "morfikownia"

    # The organizational unit of the subject.
    #unit = "sleeping dept."

    # The state of the certificate owner.
    state = "localhost"

    # The country of the subject. Two letter code.
    country = GB

    # The common name of the certificate owner.
    cn = "192.168.1.1"

    # A user id of the certificate owner.
    #uid = "scertowner"

    # The serial number of the certificate. Should be incremented each time a new certificate is generated.
    serial = 004

    # In how many days, counting from today, this certificate will expire.
    expiration_days = 3650

    # X.509 v3 extensions

    # DNS name(s) of the server
    #dns_name = "server.example.com"
    #dns_name = "server_alias.example.com"

    # (Optional) Server IP address
    ip_address = "192.168.1.150"

    # Whether this certificate will be used for a TLS server
    tls_www_server

    # Whether this certificate will be used to encrypt data (needed
    # in TLS RSA ciphersuites). Note that it is preferred to use different
    # keys for encryption and signing.
    encryption_key
    signing_key

Plik `/etc/CA/template/client_192.168.1.166` :

    # X.509 Certificate options
    #
    # DN options

    # The organization of the subject.
    organization = "morfikownia"

    # The organizational unit of the subject.
    #unit = "sleeping dept."

    # The state of the certificate owner.
    state = "localhost"

    # The country of the subject. Two letter code.
    country = GB

    # The common name of the certificate owner.
    cn = "192.168.1.166"

    # A user id of the certificate owner.
    #uid = "scertowner"

    # The serial number of the certificate. Should be incremented each time a new certificate is generated.
    serial = 003

    # In how many days, counting from today, this certificate will expire.
    expiration_days = 3650

    # X.509 v3 extensions

    # DNS name(s) of the server
    #dns_name = "server.example.com"
    #dns_name = "server_alias.example.com"

    # (Optional) Server IP address
    ip_address = "192.168.1.150"

    # Whether this certificate will be used for a TLS server
    tls_www_server

    # Whether this certificate will be used to encrypt data (needed
    # in TLS RSA ciphersuites). Note that it is preferred to use different
    # keys for encryption and signing.
    encryption_key
    signing_key

Plik `/etc/CA/template/server_192.168.1.1` :

    # X.509 Certificate options
    #
    # DN options

    # The organization of the subject.
    organization = "morfikownia"

    # The organizational unit of the subject.
    #unit = "sleeping dept."

    # The state of the certificate owner.
    state = "localhost"

    # The country of the subject. Two letter code.
    country = GB

    # The common name of the certificate owner.
    cn = "192.168.1.150"

    # A user id of the certificate owner.
    #uid = "scertowner"

    # The serial number of the certificate. Should be incremented each time a new certificate is generated.
    serial = 002

    # In how many days, counting from today, this certificate will expire.
    expiration_days = 3650

    # X.509 v3 extensions

    # DNS name(s) of the server
    #dns_name = "server.example.com"
    #dns_name = "server_alias.example.com"

    # (Optional) Server IP address
    ip_address = "192.168.1.150"

    # Whether this certificate will be used for a TLS server
    tls_www_server

    # Whether this certificate will be used to encrypt data (needed
    # in TLS RSA ciphersuites). Note that it is preferred to use different
    # keys for encryption and signing.
    encryption_key
    signing_key

Kluczowe w powyższych plikach jest dobranie parametru `cn` , który musi odpowiadać nazwie lub
adresie IP danego hosta. Jeśli ta nazwa lub adres IP nie będzie się zgadzać z nazwą lub adresem IP
danego hosta, to przy uwierzytelnianiu podczas próby przesyłu logów, zostanie wyrzucony błąd i akcja
się nie powiedzie.

Przechodzimy teraz do katalogu `/etc/CA/` i generujemy klucze prywatne na podstawie zdefiniowanych
szablonów:

    # certtool --generate-privkey --rsa --sec-param high --outfile ca_192.168.1.150.key
    Generating a 3072 bit RSA private key...

    # certtool --generate-privkey --rsa --sec-param high --outfile server_192.168.1.150.key
    Generating a 3072 bit RSA private key...

    # certtool --generate-privkey --rsa --sec-param high --outfile client_192.168.1.1.key
    Generating a 3072 bit RSA private key...

    # certtool --generate-privkey --rsa --sec-param high --outfile client_192.168.1.166.key
    Generating a 3072 bit RSA private key...

Jeden z powyższych kluczy, a konkretnie ten mający w swojej nazwie `ca` musi zostać podpisany przez
samego siebie, tak by mógł podpisywać inne
    certyfikaty:

    # certtool --generate-self-signed --load-privkey ca_192.168.1.150.key  --template ./template/ca_192.168.1.150 --outfile ca_192.168.1.150.crt
    Generating a self signed certificate...
    ...
    Signing certificate...

Teraz za pomocą tak stworzonego CA generujemy certyfikaty dla
    serwera:

    # certtool --generate-certificate --template template/server_192.168.1.150 --load-privkey server_192.168.1.150.key --load-ca-certificate ca_192.168.1.150.crt --load-ca-privkey ca_192.168.1.150.key --outfile server_192.168.1.150.crt
    Generating a signed certificate...
    ...
    Signing certificate...

Oraz dwóch pozostałych
    klientów:

    # certtool --generate-certificate --template template/client_192.168.1.1 --load-privkey client_192.168.1.1.key --load-ca-certificate ca_192.168.1.150.crt --load-ca-privkey ca_192.168.1.150.key --outfile client_192.168.1.1.crt
    Generating a signed certificate...
    ...
    Signing certificate...

    # certtool --generate-certificate --template template/client_192.168.1.166 --load-privkey client_192.168.1.166.key --load-ca-certificate ca_192.168.1.150.crt --load-ca-privkey ca_192.168.1.150.key --outfile client_192.168.1.166.crt
    Generating a signed certificate...
    ...
    Signing certificate...

W tej chwili, katalog `/etc/CA/` powinien się prezentować następująco:

    # tree /etc/CA/
    /etc/CA/
    ├── ca_192.168.1.150.crt
    ├── ca_192.168.1.150.key
    ├── client_192.168.1.1.crt
    ├── client_192.168.1.1.key
    ├── client_192.168.1.166.crt
    ├── client_192.168.1.166.key
    ├── server_192.168.1.150.crt
    ├── server_192.168.1.150.key
    └── template
        ├── ca_192.168.1.150
        ├── client_192.168.1.1
        ├── client_192.168.1.166
        └── server_192.168.1.150

    1 directory, 12 files

Przesyłamy tak stworzone certyfikaty na odpowiednie maszyny przy pomocy `scp`. W tym przypadku,
certyfikaty CA i serwera nie będą kopiowane. Trzeba tylko przesłać certyfikaty do maszyn klienckich.
Trzeba także pamiętać by skopiować certyfikat samego CA:

    # scp ca_192.168.1.150.crt client_192.168.1.1.* 192.168.1.1:/etc/syslog-ng.cert/
    root@192.168.1.1's password:
    ca_192.168.1.150.crt                 100% 1525     1.5KB/s   00:00
    client_192.168.1.1.crt               100% 1643     1.6KB/s   00:00
    client_192.168.1.1.key               100% 8394     8.2KB/s   00:00

    # scp ca_192.168.1.150.crt client_192.168.1.166.* 192.168.1.166:/etc/rsyslog.cert/
    root@192.168.1.166's password:
    ca_192.168.1.150.crt                 100% 1525     1.5KB/s   00:00
    client_192.168.1.166.crt             100% 1647     1.6KB/s   00:00
    client_192.168.1.166.key             100% 8404     8.2KB/s   00:00

## Konfiguracja serwera rsyslog

Edytujemy plik `/etc/rsyslog.conf` na serwerze i dopisujemy tam poniższe linijki:

    #################
    #### MODULES ####
    #################

    # make gtls driver the default
    $DefaultNetstreamDriver gtls

    # certificate files
    $DefaultNetstreamDriverCAFile /etc/CA/ca_192.168.1.150.crt
    $DefaultNetstreamDriverCertFile /etc/CA/server_192.168.1.150.crt
    $DefaultNetstreamDriverKeyFile /etc/CA/server_192.168.1.150.key

    $ModLoad imuxsock # provides support for local system logging
    $ModLoad imklog   # provides kernel logging support
    #$ModLoad immark  # provides --MARK-- message capability

    # provides UDP syslog reception
    #$ModLoad imudp
    #$UDPServerRun 514

    # provides TCP syslog reception
    $ModLoad imtcp
    $InputTCPServerRun 514                                # start up listener at port 514

    $InputTCPServerStreamDriverMode 1                     # run driver in TLS-only mode
    #$InputTCPServerStreamDriverAuthMode anon             # client is NOT authenticated
    #$ActionSendStreamDriverAuthMode x509/name            # authenticate by hostname
    #$InputTCPServerStreamDriverPermittedPeer *.mhouse.lh
    $InputTCPServerStreamDriverPermittedPeer 192.168.1.*

W przypadku gdybyśmy nie chcieli sprawdzać certyfikatów klienckich, trzeba odkomentować
`$InputTCPServerStreamDriverAuthMode anon` i zakomentować wszystko poniżej. Jeśli wykorzystujemy
domeny zamiast adresów IP, w `$InputTCPServerStreamDriverPermittedPeer` podajemy nazwy oraz
odhashować trzeba także `$ActionSendStreamDriverAuthMode x509/name` . Wszelkie możliwe dyrektywy,
które mogą zostać użyte w powyższym pliku, są dostępne i przyzwoicie opisane [w dokumentacji
rsyslog'a](http://www.rsyslog.com/doc/v8-stable/configuration/index_directives.html).

## Konfiguracja klienta rsyslog

Mając ustawiony już serwer, zaprogramujemy klienta tak by przesłał wszystkie logi jakie są
generowane w systemie do serwera skonfigurowanego wyżej. W tym celu edytujemy plik
`/etc/rsyslog.conf` na kliencie i dopisujemy w nim poniższe linijki:

    #################
    #### MODULES ####
    #################

    # make gtls driver the default
    $DefaultNetstreamDriver gtls

    # certificate files
    $DefaultNetstreamDriverCAFile /etc/rsyslog.cert/ca_192.168.1.150.crt
    $DefaultNetstreamDriverCertFile /etc/rsyslog.cert/client_192.168.1.166.crt
    $DefaultNetstreamDriverKeyFile /etc/rsyslog.cert/client_192.168.1.166.key

    $ModLoad imuxsock # provides support for local system logging
    $ModLoad imklog   # provides kernel logging support
    #$ModLoad immark  # provides --MARK-- message capability

    # provides UDP syslog reception
    #$ModLoad imudp
    #$UDPServerRun 514

    # provides TCP syslog reception
    #$ModLoad imtcp
    #$InputTCPServerRun 514

    # set up the action
    $ActionSendStreamDriverMode 1                   # require TLS for the connection

    *.* @@(o)192.168.1.150:514

Ostatnia linijka będzie przesyłać wszystkie logi `*.*` protokołem TCP `@@` na zdalny serwer.
Gdybyśmy chcieli skorzystać z protokołu UDP, zamiast `@@` byłby jeden znak `@` .

## Konfiguracja klienta syslog-ng

W tym przypadku również chcemy przesłać wszystkie logi przez sieć, z tym, że musimy pierw zrobić
jeden zabieg na certyfikatach. Konkretnie to musimy nazwać certyfikat CA po jego hash'u. Przy pomocy
`openssl` możemy wygenerować hash. Na kliencie wydajemy poniższe polecenia:

    # openssl x509 -noout -hash -in ca_192.168.1.150.crt
    f31e2e50

    # ln -s ./ca_192.168.1.150.crt f31e2e50.0

    # ls -al
    drwxr-xr-x    2 root     root          4096 Oct 10 20:03 .
    drwxr-xr-x    1 root     root          4096 Oct 10 15:27 ..
    -rw-r--r--    1 root     root          1525 Oct 10 19:36 ca_192.168.1.150.crt
    -rw-r--r--    1 root     root          1643 Oct 10 19:36 client_192.168.1.1.crt
    -rw-------    1 root     root          8394 Oct 10 19:36 client_192.168.1.1.key
    lrwxrwxrwx    1 root     root            22 Oct 10 20:03 f31e2e50.0 -> ./ca_192.168.1.150.crt

Przechodzimy teraz do konfiguracji `syslog-ng` . Poniżej znajduje się mój plik konfiguracyjny:

    @version:3.0

    options {
          # disable the chained hostname format in logs
          chain_hostnames(off);

          # the time to wait before a died connection is re-established
          time_reopen (600);

          # the time to wait before an idle destination file is closed
          time_reap(0);

          # the number of lines buffered before written to file
          # you might want to increase this if your disk isn't catching with
          # all the log messages you get or if you want less disk activity
          # (say on a laptop)
          flush_lines(0);

          # the number of lines fitting in the output queue
          log_fifo_size(256);

          # enable or disable directory creation for destination files
          create_dirs(no);

          # default owner, group, and permissions for log files
          owner(root);
          perm(0600);
          #group("log");

          # default owner, group, and permissions for created directories
          #dir_owner(root);
          #dir_group(root);
          #dir_perm(0755);

          # enable or disable DNS usage
          # syslog-ng blocks on DNS queries, so enabling DNS may lead to
          # a Denial of Service attack
          use_dns(no);

          # maximum length of message in bytes
          # this is only limited by the program listening on the /dev/log Unix
          # socket, glibc can handle arbitrary length log messages, but -- for
          # example -- syslogd accepts only 1024 bytes
          log_msg_size(1024);

          # Disable statistic log messages.
          stats_freq(0);

          # "--MARK--" entries in the log
          mark_freq (1800);

          keep_hostname(yes);
          use_fqdn(no);
          long_hostnames(on);
    #     ts_format(iso);      #make ISO-8601 timestamps
    };


    source s_all {
          # message generated by Syslog-NG
          internal();
          # standard Linux log source (this is the default place for the syslog() function to send logs to)
          unix-stream("/dev/log");
          # messages from the kernel
          file("/proc/kmsg" program_override("kernel"));
    };

    source s_localhost {
          tcp(ip(127.0.0.1) port(514));
          udp(ip(127.0.0.1) port(514));
    };

    destination d_messages {
          file("/var/log/messages");
    };

    destination d_network {
    #     tcp( "192.168.1.150" port(514) );
          tcp( "192.168.1.150" port(514)
                tls( ca_dir("/etc/syslog-ng.cert")
                      key_file("/etc/syslog-ng.cert/client_192.168.1.1.key")
                      cert_file("/etc/syslog-ng.cert/client_192.168.1.1.crt")
    #                 peer_verify(optional-untrusted)
                      peer_verify(required-trusted)
                      )
                );
    };


    log {
          source(s_all);
          source(s_localhost);
          destination(d_messages);
          destination(d_network);
    };

Interesuje nas głównie zwrotka `destination d_network` . Jeśli jakieś opcje są niejasne, wszystkie z
nich są dokładnie opisane [w dokumentacji
syslog-ng](https://www.balabit.com/sites/default/files/documents/syslog-ng-ose-3.4-guides/en/syslog-ng-ose-v3.4-guide-admin/html/tlsoptions.html).

## Rozdzielenie logów

Zbierane w ten sposób logi są mieszane z logami systemowymi na serwerze. Jeśli tych logów nie jest
dużo, lub też mamy kilka stacji roboczych, z których zbieramy logi, taka niedogodność niezbyt może
się nam dać we znaki. Natomiast jeśli logów mamy dużo i pochodzą przy tym z wielu hostów, przydałoby
się jakoś te logi podzielić i poupychać w odpowiednich plikach, tak by ułatwić sobie ich
przeglądanie w późniejszym czasie.

By rozdzielić logi, dopisujemy poniższe linijki w sekcji `RULES` w pliku `/etc/rsyslog.conf` ale
tak, by znajdowały się one przed wszystkimi lokalnymi regułami, przykładowo:

    ###############
    #### RULES ####
    ###############

    if $fromhost-ip startswith "192.168.1.1" then /var/log/router.log
    & stop
    ...

Pierwsza linijka zapisze logi z adresu 192.168.1.1 do pliku `/var/log/router.log` . Druga linijka
zaś kończy przetwarzanie tych logów, tj. nie są one przetwarzane przez kolejne reguły w pliku
konfiguracyjnym rsyslog'a. Jeśli nie dodamy `& stop` , komunikaty zostaną zalogowane do powyższego
pliku oraz też do standardowych plików rsyslog'a.

## Problemy

Jeśli z jakichś powodów mamy problemy z połączeniem, możemy zastopować demona `rsyslog` i odpalić go
w interaktywnym trybie debug:

    # rsyslogd -nd

Powyższe polecenie wygeneruje dość pokaźny log, w którym na pewno się znajdują informacje wskazujące
gdzie leży problem.

Możemy także wygenerować sobie testową wiadomość i sprawdzić czy zostanie przesłana przez sieć i
ewentualnie zaszyfrowana:

    # logger -t test my syslog-test-message

Z tym, że by sprawdzić czy komunikat jest szyfrowany, trzeba posłużyć się jakimś snifferem, np.
wiresharkiem.

### Alternatywna składnia pliku konfiguracyjnego rsyslog.conf

Nowsze wersje rsyslog'a wykorzystują inną składnię w pliku konfiguracyjnym. Poniżej są przykładowe
sekcje z modułami dla dwóch maszyn korzystających z `rsyslog` .

Serwer:

    #################
    #### MODULES ####
    #################

    $DefaultNetstreamDriver gtls
    $DefaultNetstreamDriverCAFile /etc/CA/ca_192.168.1.150.crt
    $DefaultNetstreamDriverCertFile /etc/CA/server_192.168.1.150.crt
    $DefaultNetstreamDriverKeyFile /etc/CA/server_192.168.1.150.key

    module(load="imuxsock")
    module(load="imklog")
    #module(load="immark")

    module( load="imtcp"
        StreamDriver.Name="gtls"                    # ptcp or gtls
        StreamDriver.mode="1"                       # run driver in TLS-only mode
        StreamDriver.authmode="x509/name"          # authenticate by hostname
        #StreamDriver.authmode="x509/fingerprint"   # certificate fingerprint authentication
        #StreamDriver.authmode="x509/certvalid"     # certificate validation only
        #StreamDriver.authmode="anon"               # clients are NOT authenticated
        #PermittedPeer="192.168.1.1"
        PermittedPeer=["192.168.1.1","morfitronik.lh"]
    )
    input( type="imtcp"
        port="514"
        name="tcp-tls"
    )

Klient:

    #################
    #### MODULES ####
    #################

    module(load="imuxsock")
    module(load="imklog")
    #module(load="immark")

    $DefaultNetstreamDriver gtls
    $DefaultNetstreamDriverCAFile /etc/rsyslog.cert/ca_192.168.1.150.crt
    $DefaultNetstreamDriverCertFile /etc/rsyslog.cert/client_morfitronik.lh.crt
    $DefaultNetstreamDriverKeyFile /etc/rsyslog.cert/client_morfitronik.lh.key

    # provides UDP syslog reception
    #module(load="imudp")
    #input(type="imudp" port="514")

    # provides TCP syslog reception
    #module(load="imtcp")
    #input(type="imtcp" port="514")

    # set up the action
    $ActionSendStreamDriverMode 1                   # require TLS for the connection

    *.* @@(o)192.168.1.150:514
