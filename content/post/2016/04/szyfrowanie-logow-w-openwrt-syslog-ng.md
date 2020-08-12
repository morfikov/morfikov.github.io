---
author: Morfik
categories:
- OpenWRT
date: "2016-04-29T02:36:00Z"
date_gmt: 2016-04-29 00:36:00 +0200
published: true
status: publish
tags:
- szyfrowanie
- logi
- router
- barrier-breaker
title: Szyfrowanie logów w OpenWRT (syslog-ng)
---

[We wpisie dotyczącym logread]({{< baseurl >}}/post/logread-czyli-system-logowania-w-openwrt/)
została podniesiona kwestia przesłania logów przez sieć. OpenWRT jest w stanie tego typu zadanie
realizować po określeniu kilku dodatkowych opcji w pliku `/etc/config/system` . Trzeba jednak zdawać
sobie sprawę, że tak przesyłane komunikaty nie będą w żaden sposób zabezpieczone. W sieci domowej
raczej nie musimy sobie zawracać głowy tym mankamentem. Niemniej jednak, gdy w grę wchodzi
przesyłanie logów do zdalnego serwera zlokalizowanego gdzieś w internecie, to taką komunikację
należy zabezpieczyć przed podsłuchem. Niestety OpenWRT standardowo nie wspiera takich udziwnień ale
dysponuje on pakietami, które mogą nam zapewnić taką funkcjonalność. Szyfrowanie logów możemy w
łatwy sposób wdrożyć za pomocą pakietu `syslog-ng3` . W nim znajduje się demon `syslog-ng` , który
jest kompatybilny w pełni z innymi linux'owymi demonami logowania. Nie powinno zatem być problemów
ze skonfigurowaniem tego całego mechanizmu.

W OpenWRT w wersji Chaos Calmer nie ma pakietu `syslog-ng3` . W efekcie szyfrowanie logów routera
nie jest obecnie możliwe. Ten wpis dotyczy jedynie wydania Barrier Breaker i zostanie zaktualizowany
jak tylko wspomniany pakiet trafi do repozytorium.

<!--more-->
## Certyfikaty

Będzie nam potrzebny szereg narzędzi, by wygenerować odpowiednie certyfikaty. Jeśli mamy mało
miejsca na flash'u routera, [generowanie
certyfikatów]({{< baseurl >}}/post/generowanie-certyfikatow/) możemy przeprowadzić spod innego
systemu operacyjnego. Najlepiej jest do tego zaprzęgnąć pierwszą lepszą dystrybucję linux'a. Możemy
także posłużyć się płytką czy pendrive z wgranym systemem live. Temat [generowania certyfikatów z
wykorzystaniem narzędzi
easy-rsa]({{< baseurl >}}/post/generowanie-certyfikatow-przy-pomocy-easy-rsa/) jak i [przy pomocy
certtool]({{< baseurl >}}/post/zaszyfrowane-logi-w-rsyslog-i-syslog-ng/) był już poruszany i nie
będziemy tutaj opisywać tego procesu. Niemniej jednak, dalsza część tego artykułu zakłada, że
dysponujemy już odpowiednimi certyfikatami.

## Wyłączenie standardowego logowania w OpenWRT

OpenWRT ma własny mechanizm logowania, który jest odpalany via skrypt `/etc/init.d/log` . To za jego
pomocą wszystkie komunikaty są zapisywane w pamięci operacyjnej RAM. Jeśli zamierzamy przesyłać logi
na zdalny serwer logów, będziemy zmuszeni zrezygnować z `logd` oraz `logread` , które są
odpowiedzialne za logowanie komunikatów na routerze. W przeciwnym razie, `syslog-ng` otrzyma jedynie
tylko część logów. Komunikaty takie jak próby logowania na SSH nie zostaną przesłane przez sieć. A
chcemy raczej otrzymać wszystkie wiadomości generowane na routerze. Wyłączamy zatem skrypt
`/etc/init.d/log` z autostartu poniższym poleceniem:

    # /etc/init.d/log disable

Poniższa konfiguracja routera zakłada, że dysponujemy już zewnętrznym serwerem logów, do którego
zamierzamy przesłać komunikaty z routera. Jeśli by się zdarzyło tak, że nie posiadamy jeszcze takiej
maszyny, to wyżej jest link do materiału, który pomoże nam postawić serwer logów na debianie w
oparciu o oprogramowanie `rsyslog` .

## Konfiguracja syslog-ng w OpenWRT

Logujemy się na router i dociągamy potrzebne oprogramowanie. Znajduje się ono w pakiecie
`syslog-ng3` :

    # opkg update
    # opkg install syslog-ng3

Następnie na router przy pomocy `scp` kopiujemy uprzednio przygotowane certyfikaty. Pamiętajmy, by
utworzyć na nim pierw katalog `/etc/syslog-ng.cert/` . W przeciwnym wypadku, to poniższe polecenie
zwróci
    błąd:

    # scp ca_192.168.1.150.crt client_192.168.1.1.crt client_192.168.1.1.key root@192.168.1.1:/etc/syslog-ng.cert/

Musimy także przepisać nazwę certyfikatu CA. `syslog-ng` wymaga, by nazwa tego certyfikatu zawierała
jego hash. Ten hash możemy wyciągnąć przy pomocy narzędzia `openssl` . Poniżej przykład:

    # openssl x509 -noout -hash -in ca_192.168.1.150.crt
    f31e2e50

Wracamy na router i przechodzimy do katalogu `/etc/syslog-ng.cert/` . Tworzymy w nim dowiązanie
symboliczne do certyfikatu CA, z tym, że do widocznej wyżej nazwy dodajemy końcówkę `.0` :

    # ssh root@192.168.1.1
    # cd /etc/syslog-ng.cert/
    # ln -s ./ca_192.168.1.150.crt f31e2e50.0

Katalog z certyfikatami dla `syslog-ng` powinien się prezentować następująco:

    # ls -al
    drwxr-xr-x    2 root     root          4096 Oct 10 20:03 .
    drwxr-xr-x    1 root     root          4096 Oct 10 15:27 ..
    -rw-r--r--    1 root     root          1525 Oct 10 19:36 ca_192.168.1.150.crt
    -rw-r--r--    1 root     root          1643 Oct 10 19:36 client_192.168.1.1.crt
    -rw-------    1 root     root          8394 Oct 10 19:36 client_192.168.1.1.key
    lrwxrwxrwx    1 root     root            22 Oct 10 20:03 f31e2e50.0 -> ./ca_192.168.1.150.crt

Edytujemy teraz plik konfiguracyjny `/etc/syslog-ng.conf` . [Dokładna jego budowa jest opisana na
stronie
projektu](https://www.balabit.com/sites/default/files/documents/syslog-ng-ose-latest-guides/en/syslog-ng-ose-guide-admin/html-single/index.html#chapter-quickstart).
Na nasze potrzeby, do tego pliku dodajemy poniższą treść ([aktualna wersja pliku znajduje się na
moim github'ie](https://github.com/morfikov/files/blob/master/configs/openwrt/syslog-ng.conf)):

    @version:3.0

    options {
        chain_hostnames(off);
        time_reopen (600);
        time_reap(0);
        flush_lines(0);
        log_fifo_size(256);
        create_dirs(no);
        owner(root);
        perm(0600);
        use_dns(no);
        log_msg_size(1024);
        stats_freq(0);
        mark_freq (3600);
        keep_hostname(yes);
        use_fqdn(no);
        long_hostnames(off);
    };

    source s_all {
        internal();
        unix-stream("/dev/log");
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
    #   tcp( "192.168.1.150" port(514) );
        tcp( "192.168.1.150" port(514)
            tls( ca_dir("/etc/syslog-ng.cert")
                key_file("/etc/syslog-ng.cert/client_192.168.1.1.key")
                cert_file("/etc/syslog-ng.cert/client_192.168.1.1.crt")
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

Konfiguracja demona `syslog-ng` sprowadza się do zdefiniowania bloków, w których umieszczamy źródła
pochodzenia logów ( `source` ), oraz takich samych bloków dotyczących miejsca przeznaczenia
przetwarzanych komunikatów ( `destination` ). Blok `source s_all` zbiera logi z domyślnych
lokalizacji systemowych. Dalej jest `source s_localhost` ale to tylko na wypadek, gdyby jakieś
usługi nie łapały się do tego powyższego bloku. Z kolei `destination d_messages` określa, że logi
mają powędrować do sprecyzowanego tam pliku. Kolejny blok ( `destination d_network` ) odpowiada za
przesyłanie logów przez sieć. Ostatni kawałek spina źródła z miejscami docelowymi. W skrócie,
wszystkie logi w systemie OpenWRT powędrują zarówno do lokalnego pliku jak i do zdalnego serwera
logów.

Przyjrzyjmy się nieco bliżej blokowi odpowiedzialnemu za przesył logów przez sieć. Gdybyśmy
odkomentowali pierwszą linijkę i zakomentowali wszystko poniżej do znaku pierwszego średnika ( `;`
), to przesył logów przez sieć nie byłby szyfrowany. Czyli mieli byśmy takie samo zachowanie, co w
przypadku określania zdalnego logowania via plik `/etc/config/system` . Z tym, że ten wbudowany
mechanizm nie umożliwia szyfrowania komunikatów. I tu właśnie do gry wchodzi pozostała część bloku
`destination d_network` . Pierwsza linijka, ta zaczynająca się od `tcp` , określa gdzie przesłać
logi (adres IP oraz port). W następnej linijce definiujemy, że chcemy zaszyfrować logi przy pomocy
modułu `tls` , na którego konfigurację składa się szereg parametrów. Pierwszy z nich to `ca_dir` i
określa on katalog z certyfikatami, tymi, które wcześniej utworzyliśmy i przesłaliśmy na router.
Dodatkowo, to w tym katalogu trzeba umieścić certyfikat CA oraz link z nazwą jego hasha. Dalej mamy
`key_file` oraz `cert_file` wskazujące na nazwy kluczy routera, odpowiednio prywatnego i
publicznego. Ostatnia opcja ( `peer_verify` ) określa czy weryfikować tożsamość serwera logów (na
podstawie certyfikatu CA).

## Testowanie komunikacji z serwerem logów

Aktywujemy skrypt `/etc/init.d/syslog.ng` i generujemy sobie testową wiadomość:

    # /etc/init.d/syslog.ng start
    # logger -t test my syslog-test-message

Powinna ona zostać zapisana w pliku `/var/log/messages` . Jeśli tak się stało, oznacza to, że część
roboty wykonaliśmy prawidłowo. Trzeba także zalogować się na zdalny serwer logów i zobaczyć czy
wiadomość również powędrowała przez sieć i została odszyfrowana. W tym przypadku wszystko
    gra:

    Oct 25 15:27:28 the-mountain.mhouse.lh s_all@the-mountain syslog-ng[2782] syslog-ng starting up; version='3.0.5'
    Oct 25 15:27:28 the-mountain.mhouse.lh s_all@the-mountain syslog-ng[2782] Syslog connection established; fd='11', server='AF_INET(192.168.1.150:514)', local='AF_INET(0.0.0.0:0)'
    Oct 25 15:27:28 the-mountain.mhouse.lh s_all@the-mountain syslog-ng[2782] Certificate subject matches configured hostname; hostname='192.168.1.150', certificate='192.168.1.150'

    Oct 25 15:27:31 the-mountain.mhouse.lh s_all@the-mountain test: my syslog-test-message

Z tym, że by sprawdzić czy komunikat jest faktycznie szyfrowany, a jego treść jest nieczytelna dla
osób próbujących podsłuchać transmisję, trzeba posłużyć się jakimś sniffer'em, np.
[wireshark](https://www.wireshark.org/).
