---
author: Morfik
categories:
- Linux
date: "2016-12-10T15:18:18Z"
date_gmt: 2016-12-10 14:18:18 +0100
published: true
status: publish
tags:
- prywatność
- vpn
- debian
- sieć
title: Jak ukryć ruch OpenVPN przy pomocy stunnel
---

Ci z nas, którzy korzystają codziennie z internetu, wiedzą, że większość nawiązywanych połączeń
między dwoma punktami w tej sieci globalnej przechodzi przez szereg węzłów i jest podatnych na
przechwycenie i podsłuchanie. Nawet jeśli ruch z określonymi serwisami jest szyfrowany, to w dalszym
ciągu nie jesteśmy w stanie ukryć pewnych newralgicznych informacji, takich jak docelowy adres IP i
port, na którym nasłuchuje zdalna usługa. Wszystkie połączenia sieciowe zestawiane z naszego
komputera czy routera domowego przechodzą przez infrastrukturę ISP, u którego mamy wykupiony
internet. Tacy ISP są nam w stanie pod naciskiem rządu zablokować połączenia z konkretnymi adresami
wprowadzając na terenie danego kraju cenzurę treści dostępnej w internecie, czego obecnie jesteśmy
świadkami w Europie, no i w Polsce. Można oczywiście posiłkować się rozwiązaniami opartymi o VPN,
np. [stawiając serwer OpenVPN w innym
kraju]({{< baseurl >}}/post/jak-skonfigurowac-serwer-vpn-na-debianie-openvpn/). Problem jednak w
tym, że ruch OpenVPN różni się od tego, z którym mamy do czynienia w przypadku choćby HTTPS. Jest
zatem możliwość rozpoznania ruchu VPN i zablokowania go stosując [głęboką analizę
pakietów](https://en.wikipedia.org/wiki/Deep_packet_inspection) (Deep Packet Inspection, DPI). By
się przed tego typu sytuacją ochronić, trzeba upodobnić ruch generowany przez OpenVPN do zwykłego
ruchu SSL/TLS. Do tego celu służy [narzędzie stunnel](https://www.stunnel.org/index.html).

<!--more-->
## Konfiguracja usługi stunnel na serwerze

Konfiguracja dla demona `stunnel` jest trzymana standardowo w katalogu `/etc/stunnel/` . Domyślnie
jest tam tylko plik `README` z instrukcjami, które pomogą nam skonfigurować usługę. Jest tam zawarta
informacja, że przykładowy plik konfiguracyjny znajduje się w
`/usr/share/doc/stunnel4/examples/stunnel.conf-sample` . Trzeba go będzie dostosować zarówno na
serwerze jak i na kliencie. Przekopiujmy zatem ten plik do katalogu `/etc/stunnel/` nazywając go
dowolnie ale tak, by jego rozszerzenie wskazywało na `.conf` ,
    przykładowo:

    # cp /usr/share/doc/stunnel4/examples/stunnel.conf-sample /etc/stunnel/morfitronik-stunnel-server.conf

Teraz musimy poddać edycji tak utworzony plik i zmienić w nim szereg rzeczy:

    foreground = yes
    setuid = stunnel4
    setgid = stunnel4
    pid = /var/run/stunnel4/stunnel4.pid
    ;chroot = /var/lib/stunnel4/
    debug = 7
    ;output = /var/log/stunnel.log
    socket = l:TCP_NODELAY=1
    sslVersion = TLSv1.2
    ciphers = ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-DSS-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDH-RSA-AES256-GCM-SHA384:ECDH-ECDSA-AES256-GCM-SHA384:AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:ECDH-RSA-AES128-GCM-SHA256:ECDH-ECDSA-AES128-GCM-SHA256:AES128-GCM-SHA256

    [openvpn]
    client = no
    accept = 11.22.33.44:443
    connect = 127.0.0.1:1194
    delay = yes
    CAfile = /etc/openvpn/certs/morfitronik-ca.crt
    cert = /etc/openvpn/certs/morfitronik-server-vpn-stunnel.crt
    key = /etc/openvpn/certs/morfitronik-server-vpn.key
    requireCert = yes
    verifyChain = yes
    ;verifyPeer = yes

Ten powyższy plik składa się z dwóch części. W pierwszej mamy parametry dla samego demona, w drugiej
zaś konfigurację usługi. Większość opcji powinna być raczej zrozumiała ale wypadałoby powiedzieć
kilka słów na temat sekcji konfigurującej usługę `openvpn` .

Generalnie rzecz biorąc, nazwa samej sekcji jest dowolna ale musi być ujęta w `[ ]`. Parametr
`client` przełącza demona w tryb klienta lub serwera. W `accept` definiujemy adres i port serwera,
na który będą przychodzić zapytania z sieci (zewnętrzny adres IP). Z kolei w `connect` określamy
adres pętli zwrotnej i jej port, na którym będzie nasłuchiwał demon OpenVPN. Pozostałe opcje dotyczą
konfiguracji certyfikatów i akceptowania jedynie tych klientów, którzy przedstawią ważny certyfikat.

## Konfiguracja certyfikatów

Każda usługa oferująca szyfrowanie danych, a do takich zalicza się `stunnel` , wymaga zestawienia
bezpiecznego połączenia. Potrzebne nam zatem będą odpowiednie certyfikaty i klucze szyfrujące. Nie
musimy ich jednak generować na nowo. Mając działający serwer OpenVPN możemy wykorzystać jego
certyfikaty i podać tylko odpowiednie ścieżki w pliku `/etc/stunnel/morfitronik-stunnel-server.conf`
, tak jak to zostało zrobione wyżej. Trzeba tylko pamiętać, że `stunnel` akceptuje certyfikaty
jedynie w formacie PEM. Dodatkowo, parametry Diffie-Hellman'a (DH), które w przypadku OpenVPN są
zawarte w pliku `dh4096.pem` trzeba dołączyć do pliku z certyfikatem dla demona `stunnel` :

    # cat /etc/openvpn/certs/dh4096.pem >> /etc/openvpn/certs/morfitronik-server-vpn-stunnel.crt

W konfiguracji demona `stunnel` mogliśmy zauważyć prametry `requireCert` , `verifyChain` oraz
`verifyPeer` . Odpowiadają one, jak nazwa wskazuje, za wymaganie i weryfikacje certyfikatów
klienckich. W zasadzie są to parametry jedynie opcjonalne i generalnie potrzebny nam jest zwykle
jeden z nich, tj. `verifyChain` lub `verifyPeer` . W przypadku wykorzystania któregoś z nich,
parametr `requireCert` jest implikowany automatycznie.

Jaka jest zatem różnica między `verifyChain` i `verifyPeer` ? Certyfikaty zarówno serwera jak i
klienta `stunnel` są podpisane przez CA. Jeśli klient przedstawi certyfikat podpisany przez inne CA
niż to, którym został podpisany certyfikat serwera, to `verifyChain` nie zezwoli na podłączenie się
takiej maszynie. Jeśli zaś chodzi o `verifyPeer` , to w tym przypadku certyfikat klienta musi zostać
dołączony do pliku zdefiniowanego w opcji `CAfile` w konfiguracji `stunnel` .

## Konfiguracja usługi stunnel na kliencie

W przypadku drugiej strony połączenia, tj. klienta, również musimy w odpowiedni sposób skonfigurować
sobie demona `stunnel` . Kopiujemy zatem przykładowy plik konfiguracyjny do katalogu `/etc/stunnel/`
:

    # cp /usr/share/doc/stunnel4/examples/stunnel.conf-sample /etc/stunnel/morfitronik-stunnel-client.conf

Plik poddajemy edycji i dostosowujemy w nim poniższe parametry:

    foreground = yes
    setuid = stunnel4
    setgid = stunnel4
    pid = /var/run/stunnel4/stunnel4.pid
    ;chroot = /var/lib/stunnel4/
    debug = 7
    ;output = /var/log/stunnel.log
    socket = r:TCP_NODELAY=1
    sslVersion = TLSv1.2
    ciphers = ECDHE-RSA-AES256-GCM-SHA384

    [openvpn]
    client = yes
    accept = 127.0.0.1:1194
    connect = 11.22.33.44:443
    delay = yes
    CAfile = /etc/openvpn/certs/morfitronik-ca.crt
    cert = /etc/openvpn/certs/morfitronik-client-vpn-laptop.crt
    key = /etc/openvpn/certs/morfitronik-client-vpn-laptop.key
    requireCert = yes
    verifyChain = yes
    ;verifyPeer = yes
    checkHost = morfitronik-server-vpn

Konfiguracja klienta `stunnel` jest podobna do tej po stronie serwera. Sekcja z parametrami dla
samego demona nie różni się zbytnio, no może za wyjątkiem opcji `socket` (o niej będzie później)
oraz zawęziliśmy też możliwe do wykorzystania szyfry. O ile serwer powinien wspierać wszystkie
szyfry TLS 1.2, o tyle klient może ograniczyć się do jednego.

Jeśli zaś chodzi o sekcję usługi `openvpn` , to naturalnie przestawiamy parametr `client` tak, by
demon `stunnel` wiedział, że ma pracować w roli klienta. Z kolei w `accept` podajemy adres pętli
zwrotnej i port, na którym działa klient OpenVPN, a w `connect` wpisujemy adres zdalnego serwera i
jego port.

Konfiguracja certyfikatów i uwierzytelniania drugiego końca połączenia jest mniej więcej taka sama
co w przypadku serwera. Możemy także skorzystać z parametru `checkHost` weryfikującego pole CN w
certyfikacie serwera. Generalnie możemy dowolnie sobie te cztery ostatnie opcje dobrać, tylko
pamiętajmy o różnicach między `verifyChain` i `verifyPeer` .

## Usługa dla systemd

W paczce `stunnel4` próżno szukać usługi dla systemd. Możemy jednak takową dorobić sami. Wystarczy
stworzyć plik `/etc/systemd/system/stunnel@.service` i dodać do niego poniższą zawartość:

    [Unit]
    Description=SSL tunnel for %I
    Documentation=man:stunnel(8)
    After=syslog.target network.target

    [Service]
    Type=simple
    PrivateTmp=true
    ExecStart=/usr/bin/stunnel /etc/stunnel/%i.conf
    ExecReload=/bin/kill -HUP $MAINPID
    PIDFile=/var/run/stunnel4/stunnel4.pid

    [Install]
    WantedBy=multi-user.target

Pamiętajmy tylko, by dodać do pliku konfiguracyjnego demona `stunnel` opcję `foreground=yes` , bo
ten demon nie może nam forkować przy takiej konfiguracji usługi systemd jak widzimy powyżej.

Musimy także stworzyć katalog, w którym będzie przechowywany PID demona `stunnel` tak, by nie było
problemów ze zrzucaniem uprawnień. Tworzymy zatem plik `/etc/tmpfiles.d/stunnel.conf` i wrzucamy do
niego poniższą zawartość:

    #Type   Path              Mode    UID       GID        Age   Arguments
    d       /run/stunnel4     0755    stunnel4  stunnel4   -     -

Usługę uruchamiamy standardowo podając po znaku `@` nazwę pliku konfiguracyjnego obecnego w katalogu
`/etc/stunnel/` ale bez końcówki `.conf` przykładowo:

    # systemctl enable stunnel@morfitronik-stunnel-server
    # systemctl start stunnel@morfitronik-stunnel-server

## Dostosowanie konfiguracji OpenVPN

Generalnie rzecz biorąc, jeśli dysponowaliśmy wcześniej działającym serwerem VPN i klienci byli w
stanie z nim nawiązywać połączenia bez większego problemu, to dostosowanie całej konfiguracji
OpenVPN sprowadza się jedynie do zmiany kilku parametrów w plikach serwera i klienta (katalog
`/etc/openvpn/` ). W obu przypadkach musimy przepisać opcje odpowiadające za protokół i adres IP.
Wykorzystywany protokół musi wskazywać na TCP, natomiast jako adres IP trzeba określić pętlę zwrotną
(loopback). Jako, że demon `stunnel` będzie zbierał ruch od OpenVPN i go szyfrował przed przesłaniem
na drugi koniec połączenia, to możemy także zrezygnować z szyfrowania ruchu w samym VPN:

Dla serwera będą to te trzy poniższe wpisy:

    local 127.0.0.1
    proto tcp
    cipher none

A dla klienta te trzy:

    proto tcp
    remote 127.0.0.1 1194
    cipher none

Wykorzystywanie protokołu TCP nie jest najlepszym pomysłem w przypadku VPN, bo drastycznie odbije
się to na wydajności ogólnej połączenia ale nie mamy zbytnio innego wyjścia. Oprogramowanie
`stunnel` potrafi zabezpieczać jedynie komunikację opartą na protokole TCP. Oczywiście nic nie stoi
na przeszkodzie, by uruchomić dwie osobne instancje VPN i wykorzystywać je w zależności od
panujących warunków cenzorskich. Pamiętajmy tylko o dostosowaniu opcji `server` w konfiguracji
serwera OpenVPN odpowiadającej za adresację sieci LAN wykorzystywanej w VPN.

Dodatkowo, w przypadku klienta VPN trzeba także uwzględnić dodatkową trasę w tablicy routingu. Musi
ona wskazywać na adres zewnętrzny naszego serwera.

    route 11.22.33.44 255.255.255.255 net_gateway

## Dobór portu dla usługi stunnel

Przy określaniu portu, na którym ma nasłuchiwać demon `stunnel` , dobrze jest wybrać ten, który jest
wykorzystywany oficjalnie przez jakieś usługi. Poniżej jest krótka rozpiska takich portów:

    nsiiops      261/tcp   # IIOP Name Service over TLS/SSL
    https        443/tcp   # http protocol over TLS/SSL
    smtps        465/tcp   # smtp protocol over TLS/SSL (was ssmtp)
    nntps        563/tcp   # nntp protocol over TLS/SSL (was snntp)
    imap4-ssl    585/tcp   # IMAP4+SSL (use 993 instead)
    sshell       614/tcp   # SSLshell
    ldaps        636/tcp   # ldap protocol over TLS/SSL (was sldap)
    ftps-data    989/tcp   # ftp protocol, data, over TLS/SSL
    ftps         990/tcp   # ftp protocol, control, over TLS/SSL
    telnets      992/tcp   # telnet protocol over TLS/SSL
    imaps        993/tcp   # imap4 protocol over TLS/SSL
    ircs         994/tcp   # irc protocol over TLS/SSL
    pop3s        995/tcp   # pop3 protocol over TLS/SSL (was spop3)
    msft-gc-ssl  3269/tcp  # Microsoft Global Catalog with LDAP/SSL

Generalnie rzecz biorąc, chodzi tutaj o niewzbudzanie podejrzeń. Jeśli przy skanowaniu naszego
serwera zostanie wykryty port 993, który jest standardowo powiązany z bezpiecznym protokołem
pocztowym, to raczej mało kto pomyśli, że tutaj nasłuchuje serwer VPN chroniony przez demona
`stunnel` . Gdybyśmy na tym porcie umieścili jedynie sam serwer OpenVPN, to podłączenie do niego
(nawet zakończone niepowodzeniem) ujawniłoby, że na tym porcie jest VPN. A tak, ruch będzie wyglądał
jakby został zaszyfrowany zwykłym protokołem SSL/TLS i głęboka inspekcja pakietów nie będzie w
stanie wykazać co na tym porcie faktycznie działa.

## Problem z wydajnością połączenia VPN

W przypadku obserwowania znacznego spowolnienia transmisji danych przez VPN po wdrożeniu demona
`stunnel` , możemy spróbować włączyć opcję `TCP_NODELAY` na obu końcach połączenia dodając do pliku
konfiguracyjnego serwera poniższy wpis:

    socket = l:TCP_NODELAY=1

Podobną linijkę trzeba dodać do pliku konfiguracyjnego klienta:

    socket = r:TCP_NODELAY=1

W razie innych problemów z działaniem demona `stunnel` warto zajrzeć również do [do FAQ na stronie
projektu](https://www.stunnel.org/faq.html), jak i do [man
stunnel](http://manpages.ubuntu.com/manpages/zesty/en/man8/stunnel.8.html).
