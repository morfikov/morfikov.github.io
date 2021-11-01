---
author: Morfik
categories:
- RaspberryPi
- Linux
date:    2020-08-25 19:21:00 +0200
lastmod: 2020-08-25 19:21:00 +0200
published: true
status: publish
tags:
- raspberry-pi-4b
- kodi
- xbmc
- libreelec
- rsyslog
- logi
- sieć
- debian
GHissueID: 39
title: Raspberry Pi, LibreELEC, Kodi i zdalne logi via rsyslog
---

Parę lat temu, gdy pojawił się u mnie w domu bezprzewodowy router WiFi, postanowiłem wgrać na niego
linux'a w postaci OpenWRT. Pierwszym kluczowym elementem konfiguracyjnym tego urządzenia było
[przesłanie jego logów systemowych przez sieć][1] do mojego laptopa, tak by wszystkie
zarejestrowane komunikaty zostały wyświetlone na konsoli mojego komputera z zainstalowanym Debianem.
W ten sposób nie musiałem się co chwila logować na router po SSH (czy też przez panel webowy), by
sprawdzić czy aby na pewno z tym urządzeniem jest wszystko w porządku. Teraz, po nabyciu Raspberry
Pi 4B i wgraniu na niego [LibreELEC][2] z preinstalowanym Kodi, mam dokładnie to samo zadanie do
zrealizowania. Trzeba zatem znaleźć sposób na przesłanie wszystkich logów generowanych przez system
LibreELEC do demona `rsyslogd` , który jest uruchomiony na zdalnej maszynie.

<!--more-->
## Instalowanie demona rsyslogd w LibreELEC

By można było w ogóle myśleć o przesyłaniu logów systemowych przez sieć, potrzebne jest stosowne
oprogramowanie zainstalowane na dwóch maszynach, między którymi te logi mają być przekazywane. W
LibreELEC nie da się standardowo nic zainstalować, bo tutaj nie ma żadnego menadżera pakietów
pokroju `apt-get`/`aptitude` czy `dpkg` . A poza tym, główny system plików LibreELEC i tak jest
tylko do odczytu, więc nic na niego nie udałby się nam wgrać. Niemniej jednak, w repozytorium
LibreELEC dla Kodi jest stosowny addon o nazwie `Rsyslog`, który (jak nazwa może sugerować)
dostarcza demona `rsyslogd` . By zainstalować ten dodatek, trzeba przejść kolejno do `Dodatki` =>
`Zainstaluj z repozytorium` => `LibreELEC Add-ons` => `Usługi` i wybrać pozycję `Rsyslog` :

![raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-install](/img/2020/08/001-raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-install.png#huge)

Gdy już zainstalujemy dodatek `Rsyslog` , przechodzimy do jego konfiguracji.

W zasadzie to mamy do wyboru konfigurację demona `rsyslogd` z poziomu Kodi oraz ręczne dostosowanie
pliku `rsyslog.conf` przy pomocy SSH. Jeśli potrzebujemy korzystać z nieco bardziej zaawansowanych
rzeczy oferowanych przez `rsyslogd` , to musimy aktywować w konfiguracji dodatku opcję `Configure
rsyslog.conf manually` . Nam jednak wystarczą opcje oferowane przez panel Kodi, bo bez problemu za
ich sprawą jesteśmy w stanie przesłać logi na zdalny serwer. Jedyne co musimy określić to adres IP
maszyny docelowej, jej port oraz protokół. Ja korzystam z protokołu TCP zamiast UDP. Zatem cała
konfiguracja dodatku `Rsyslog` wygląda u mnie mniej więcej tak:

![raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-config](/img/2020/08/002-raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-config.png#huge)

Jako, że ja chciałem przesłać wszystkie logi mojego Raspberry Pi 4B do zdalnego serwera, toteż
zaznaczyłem opcję `Log journal and kernel` . Bez tej opcji, jedynie logi Kodi by było przesyłane,
przez co niekoniecznie w tych logach mogą być zawarte interesujące nas ewentualne komunikaty błędów.
Z opcją `Log journal and kernel` będziemy mieli dokładnie taki sam wgląd w logi LibreELEC, co w
przypadku zalogowania się na ten system po SSH.

## Konfiguracja demona rsyslogd na Debianie

Jedną końcówkę połączenia mamy już skonfigurowaną. Trzeba teraz jeszcze skonfigurować drugą
końcówkę, na której jest zainstalowany Debian. W przypadku tej maszyny, pakiet `rsyslog` jest
zainstalowany i raczej powinien być on domyślnie instalowany na Debianie. Jeśli jednak nie
posiadamy tego pakietu, to trzeba go będzie doinstalować ręcznie. Konfiguracja demona `rsyslogd` na
Debianie jest przechowywana w pliku `/etc/rsyslog.conf` i ten  plik trzeba będzie poddać edycji.

Na początek odszukujemy ten poniższy blok kodu i usuwamy z jego dwóch ostatnich linijek znak `#` :

    #################
    #### MODULES ####
    #################

    module(load="imtcp")
    input(type="imtcp" port="514")

W ten sposób włączymy w `rsyslogd` możliwość otrzymywania pakietów sieciowych z wykorzystaniem
protokołu `TCP` na port `514` .

Następnie dodajemy reguły, które przekierują komunikaty otrzymane od Raspberry Pi 4B do osobnego
pliku na dysku oraz do urządzenia FIFO:

    ###############
    #### RULES ####
    ###############

    if $fromhost-ip startswith "192.168.1.239" then -/dev/log-rpi
    if $fromhost-ip startswith "192.168.1.239" then -/var/log/rpi.log
    & stop

Pierwsza reguła ma za zadanie dopasować źródło komunikatów z określonego adresu IP (ten adres
`192.168.1.239` jest przypisany Raspberry Pi 4B) i przesłać logi do urządzenia `/dev/log-rpi` .
Druga reguła jest bardzo podobna, z tą różnicą, że te logi powędrują do pliku `/var/log/rpi.log` .
W ostatniej linijce mamy zaś `& stop` , który to zakończy dalsze przetwarzanie komunikatów z
Raspberry Pi 4B.

Ja mam u siebie konsolę na pulpicie, do której mamy podpiętych szereg urządzeń FIFO, dlatego też i
w tym przypadku to urządzenie jest wykorzystane. Jeśli nie mamy u siebie takich egzotycznych
konfiguracji, to tę pierwszą regułę możemy zwyczajnie skasować i zostawić jedynie logowanie
komunikatów otrzymanych od Raspberry Pi 4B do pliku. Jeśli jednak chcielibyśmy przesyłać logi do
urządzenia FIFO, to trzeba to urządzenie pierw stworzyć w poniższy sposób:

    # mkfifo /dev/log-rpi
    # chown root:systemd-journal /dev/log-rpi
    # ls -al /dev/log-rpi
    prw-r--r-- 1 root systemd-journal 0 2020-08-25 19:35:26 /dev/log-rpi|

By to urządzenie FIFO było tworzone automatycznie przy starcie Debiana, to do pliku
`/etc/tmpfiles.d/rsyslog.conf` (jeśli nie mamy tego pliku, to naturalnie trzeba go utworzyć)
dodajemy poniższą linijkę:

    # Type | Path | Mode | UID | GID | Age | Argument
    p /dev/log-rpi 0640 root systemd-journal - -

Po skonfigurowaniu demona `rsyslogd` , trzeba zrestartować jeszcze usługę `rsyslog.service` :

    # systemctl restart rsyslog.service

W tej chwili demon `rsyslogd` powinien zacząć nasłuchiwać połączeń sieciowych, co możemy sprawdzić
przy pomocy `netstat` :

    # netstat -napletu | grep rsyslogd
    tcp    0   0 0.0.0.0:514  0.0.0.0:*  LISTEN   0   19920925   1810230/rsyslogd
    tcp6   0   0 :::514       :::*       LISTEN   0   19920926   1810230/rsyslogd

Jak widać demon `rsyslogd` nasłuchuje połączeń na porcie `514` protokołu `TCP` na każdym lokalnym
adresie zarówno IP jak i IPv6.

## Test przesyłania logów między LibreELEC a Debianem

Pozostało nam już w zasadzie przetestować czy obie końcówki połączenia działają. Odpalamy zatem
Raspberry Pi 4B i patrzymy czy demon `rsyslogd` z LibreELEC jest w stanie się połączyć z demonem
`rsyslogd` w Debianie:

    # netstat -napletu | grep rsyslogd
    tcp    0   0 0.0.0.0:514         0.0.0.0:*               LISTEN      0   19920925   1810230/rsyslogd
    tcp    0   0 192.168.1.150:514   192.168.1.239:38308     ESTABLISHED 0   21887301   1810230/rsyslogd
    tcp6   0   0 :::514              :::*                    LISTEN      0   19920926   1810230/rsyslogd

W `netstat` pojawił się dodatkowy wpis, którego stan wskazuje na `ESTABLISHED` . Zatem połączenie
sieciowe między LibreELEC a Debianem zostało poprawnie nawiązane.

Sprawdźmy jeszcze czy plik `/var/log/rpi.log` się rozrasta:

    # ls -alh /var/log/rpi.log
    -rw-r----- 1 root adm 74K 2020-08-25 19:52:30 /var/log/rpi.log

Wartość `74K` wskazuje na aktualny rozmiar tego pliku. Wiemy więc, że komunikaty są logowane do
pliku `/var/log/rpi.log` .

Jeśli dodatkowo stworzyliśmy sobie urządzenie FIFO, to w czasie rzeczywistym możemy podglądać te
komunikaty na konsoli, tak jak to widać na poniższym zrzucie ekranu:

![raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-test-console](/img/2020/08/003-raspberry-pi-libreelec-kodi-xbmc-rsyslog-addon-test-console.jpg#huge)


[1]: /post/logread-czyli-system-logowania-w-openwrt/
[2]: https://libreelec.tv/
