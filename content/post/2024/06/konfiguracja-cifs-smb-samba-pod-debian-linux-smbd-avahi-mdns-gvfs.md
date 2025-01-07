---
author: Morfik
categories:
- Linux
date:    2024-06-03 20:00:00 +0200
lastmod: 2024-06-16 16:30:00 +0200
published: true
status: publish
tags:
- samba
- smb
- cifs
- avahi
- wsdd
- iptables
- nftables
- firewall
- serwer-plików
- keepassxc
GHissueID: 601
title: Konfiguracja CIFS/SMB/SAMBA pod Debian Linux (smbd/avahi/mdns/gvfs)
---

Z racji, że w mojej sieci domowej pojawia się coraz więcej maszyn, które mają na pokładzie różne
systemy operacyjne (począwszy od linux'ów, przez androidy, a na windows'ach skończywszy), to
postanowiłem w końcu wypracować jakiś w miarę uniwersalny sposób na dzielenie się plikami w tej
sieci LAN. Rozwiązania pokroju SSH/SSHFS są w porządku ale jedynie w przypadku linux'ów. Serwery
HTTP/FTP są trochę mało poręczne i słabo się nadają do tego celu. Z kolei NFS miał u mnie jakieś
większe/mniejsze problemy z wydajnością, a i też nie wiem jak wygląda wsparcie dla NFS na windows.
W zasadzie to pozostała mi ostatnia opcja w postaci protokołu [CIFS/SMB/SAMBA][4], który
przynajmniej na windows powinien działać bez zarzutu. Trzeba jednak będzie przystosować mojego
Debiana do pracy z tym protokołem. Chodzi generalnie o to, by bez większego problemu być w
stanie udostępnić z jednej maszyny kilka katalogów innym użytkownikom sieci domowej i to bez
znaczenia czy robimy to z windows'a i docelowo siedzimy na linux'ie, czy też w drugą stronę, tj.
udostępniamy katalogi z linux'a, by być w stanie przeglądać ich zawartość na windows. Wdrożenie
obsługi protokołu CIFS/SMB/SAMBA na linux jest stosunkowo proste, bo wystarczy zainstalować pakiet
`samba` , wydać w terminalu kilka poleceń i po sprawie. Problem jednak zaczyna się, gdy chcemy
sobie nieco ułatwić życie, tak by w menadżerze plików (np. `nemo` , `caja` , `pcmanfm` ) te
wszystkie zasoby były z automatu widoczne po przejściu do zakładki sieć/network. Pozostają też do
rozważenia kwestie związane z bezpieczeństwem tego całego mechanizmu wymiany plików.

<!--more-->

## Różnice między CIFS, SMB, SAMBA

Najprościej rzecz ujmując, pakiet SAMBA to zbiór programów, które implementują w systemach UNIX
protokół SMB (Server Message Block) i zapewniają usługi Active Directory. Pierwsza wersja protokołu
SMB jest czasami określana także jako CIFS (Common Internet File System). SAMBA implementuje również
protokół NetBIOS w `nmbd` . [Więcej informacji na temat historii CIFS/SMB/SAMBA][6] można znaleźć
tutaj.

## Potrzebne oprogramowanie (smbd, avahi, mdns, gvfs)

Opisywany w niniejszym artykule sposób konfiguracji protokołu CIFS/SMB/SAMBA będzie zakładał
wdrożenie niezbędnej funkcjonalności potrzebnej w celu udostępniania plików w sieci lokalnej. Do
tego celu będzie nam potrzebny demon `smbd` (z pakietu `samba` ), demon `avahi-daemon` (z pakietu
`avahi-daemon` ), biblioteka `libnss-mdns` oferująca rozwiązywanie nazw hosta via Multicast DNS
przy wykorzystaniu Zeroconf zwanej także Apple Bonjour / Apple Rendezvous oraz demon `gvfsd` (z
pakietu `gvfs-daemons` ) wraz z paroma backend'ami gvfs (z pakietu `gvfs-backends` ).

Poniżej jest lista rzeczy, które musimy zainstalować w swoim linux'ie.

    # aptitude install \
        samba \
        libnss-mdns
        avahi-daemon avahi-utils \
        gvfs gvfs-daemons gvfs-backends gvfs-fuse

Jako, że będziemy udostępniać pliki via CIFS/SMB/SAMBA na wyraźne żądanie, to ze względów
bezpieczeństwa wszystkie te nowo zainstalowane usługi lepiej jest pozostawić w Debianie domyślnie
wyłączone. Gdy tylko zajdzie potrzeba wymiany plików, to te usługi najzwyczajniej w świecie można
sobie uruchomić ręcznie i nie ma większej potrzeby, by startowały one wraz z podnoszeniem się
naszego linux'a:

    # systemctl disable smbd.service nmbd.service
    # systemctl disable avahi-daemon.service avahi-daemon.socket

### Kwestia nmbd, wsdd/wsdd2 i winbind

Starsze serwery CIFS/SMB/SAMBA mogą być dodatkowo wyposażone w demony  `nmbd` , `wsdd` / `wsdd2`
oraz `winbindd` , które obecnie zdają się być już przestarzałe.

Demon `nmbd` jest w stanie odpowiadać na żądania obsługi nazw NetBIOS poprzez IP takie, jakie są
tworzone przez klientów CIFS/SMB/SAMBA (Windows 95/98, Windows NT i klientów LanManager).
Partycypuje również w protokołach przeglądania, które tworzą widok "Otoczenie Sieciowe" Windows.
Demon `nmbd` może być również użyty jako serwer WINS (Windows Interner Name Server). To w praktyce
oznacza, że będzie funkcjonował jako serwer bazy danych WINS tworząc bazę danych z żądań
rejestracyjnych, które otrzymuje i odpowiadając na zapytania od klientów o nazwy. Dodatkowo, `nmbd`
może funkcjonować jako serwer proxy WINS, przekazujący zapytania rozgłaszane przez klientów, którzy
nie wiedzą jak rozmawiać protokołem WINS z serwerem WINS.

WSD to skrót od Web Services on Devices i jest to interfejs API firmy Microsoft upraszczający
programowanie połączeń z urządzeniami obsługującymi usługi sieciowe, takie jak drukarki, skanery i
udostępnianie plików. [Demon wssd][1] rozgłasza i odpowiada na żądania klientów Windows szukających
udostępnionych plików. Implementuje on również usługi wyszukiwania nazw LLMNR (Link-Local Multicast
Name Resolution). LLMNR został wprowadzony w systemie Microsoft Windows Vista i jest następcą
NBT-NS (NetBIOS Name Service).

Demon `winbindd` dostarcza usług do funkcjonalności Przełącznika Usług Nazewniczych (Name Service
Switch, który jest obecny w większości nowych bibliotek C. Name Service Switch pozwala na uzyskanie
informacji użytkownika i systemu z różnych usług bazodanowych, takich jak NIS lub DNS. Dokładne
zachowanie może być skonfigurowane poprzez plik `/etc/nsswitch.conf` . Użytkownicy i grupy są
alokowani, gdy tylko zostaną odwzorowani w obrębie zakresu identyfikatorów użytkowników i grup
określonego przez administratora systemu Samba. Usługa dostarczana przez `winbindd` może być użyta
do odwzorowania danych użytkownika i grupy z serwera Windows NT. Może również dostarczyć usług
uwierzytelniających poprzez moduł PAM.

Te wymienione wyżej demony/usługi miały na celu obsługę protokołów LLMNR (Link-Local Multicast Name
Resolution) oraz NBT-NS (NetBIOS Name Service) i oferować tym samym usługi rozwiązywania nazw
hostów w sieci lokalnej. Niemniej jednak, te protokoły nie są zbyt bezpieczne i lepiej się
zastanowić, czy chcemy ich obsługę implementować na naszym linux'ie. [O problemach natury
bezpieczeństwa protokołów LLMNR i NBT-NS][11] można przeczytać tutaj.

### Microsoft rezygnuje z LLMNR/NBT-NS na rzecz mDNS

Na wiki można wyczytać, że od kwietnia 2022 [Microsoft rozpoczął proces stopniowego wycofywania][12]
zarówno LLMNR, jak i rozpoznawania nazw NetBIOS na rzecz [mDNS][15]. Wygląda zatem na to, że jeśli
w sieci mamy tylko nowsze windows'y, to można całkowicie zrezygnować z `nmbd` , `wsdd` / `wsdd2`
oraz `winbindd` .

Jeśli jednak mamy w sieci starsze klienty windows, to trzeba będzie doinstalować także poniższe
pakiety:

    # apptitude install \
        wsdd wsdd2 \
        winbind

## Konfiguracja mDNS (plik /etc/nsswitch.conf)

Multicast DNS (mDNS) jest w Debianie konfigurowany za sprawą pliku `/etc/nsswitch.conf` . W tym
pliku znajduje się linijka z `hosts` , która to jest automatycznie dostosowywana za nas przy
instalacji pakietu `libnss-mdns`. W przypadku, gdy nie korzystamy z `systemd-resolved` lub też
robimy z niego jakiś użytek ale bez funkcjonalności oferowanej przez pakiet `libnss-resolve` , to
ta linijka z `hosts` zostanie przepisana do poniższej postaci:

    hosts:          files mdns4_minimal [NOTFOUND=return] dns

Gdy `libnss-resolve` w `systemd-resolved` jest włączony, to wtedy ta linijka będzie miała poniższą
postać:

    hosts:          files mdns4_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] dns

W razie problemów z rozwiązywaniem nazw maszyn lokalnych, dobrze jest się upewnić czy wpisy w pliku
`/etc/nsswitch.conf` są takie jak być powinny.

## Konfiguracja DNS (plik /etc/resolv.conf)

Standardowo w linux'ach konfiguracja DNS jest pobierana z serwera DHCP naszego domowego routera
WiFi. Po jej uzyskaniu, dostosowywany jest plik `/etc/resolv.conf` , w którym to powinno się
znaleźć szereg potrzebnych wpisów, przykładowo:

    nameserver 127.0.0.1
    nameserver ::1
    options edns0 trust-ad
    options timeout:2
    search mhouse

Te opcje, które nas interesują najbardziej to `nameserver` oraz `search` . Parametr `nameserver`
wskazuje na adres serwera DNS, który będzie rozwiązywał nazwy hostów/domen na adresy IP. Z kolei
parametr `search` ma na celu dodać do wszystkich zapytań DNS nieposiadających części domeny (brak
choć jednej `.` ) frazę, którą w nim sobie określimy. Zatem próbując rozwiązać, np. nazwę
`janek-laptop` , do serwera DNS powędruje tak naprawdę zapytanie o adres hosta
`janek-laptop.mhouse` , które zostanie lokalnie rozwiązane przez serwer DNS naszego domowego
routera WiFi, bo to on jest odpowiedzialny za rozwiązywanie nazw na adresy IP w tej domenie.
Sprawdźmy czy tak faktycznie się stanie:

    $ ping -4 janek-laptop -c 4
    PING janek-laptop (192.168.1.193) 56(84) bytes of data.
    64 bytes from janek-laptop.mhouse (192.168.1.193): icmp_seq=1 ttl=64 time=2.01 ms
    64 bytes from janek-laptop.mhouse (192.168.1.193): icmp_seq=2 ttl=64 time=1.35 ms
    64 bytes from janek-laptop.mhouse (192.168.1.193): icmp_seq=3 ttl=64 time=3.15 ms
    64 bytes from janek-laptop.mhouse (192.168.1.193): icmp_seq=4 ttl=64 time=4.88 ms

    --- janek-laptop ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3019ms
    rtt min/avg/max/mdev = 1.354/2.848/4.880/1.337 ms

Jak widać, odpowiedź `ping` do hosta `janek-laptop` została zwrócona z adresu `192.168.1.193` ,
czyli tego, który jest tej maszynie przypisany. Widzimy też, że niby podaliśmy w żądaniu `ping`
jedynie nazwę `janek-laptop` ale została ona automatycznie przepisana do postaci
`janek-laptop.mhouse` .

### Konfiguracja reverse DNS lookups

Gdyby w tym powyższym logu widniał jedynie sam adres IP, to wtedy mamy problemy z
konfiguracją [rekordów DNS PTR][17], które są wykorzystywane przy [reverse DNS lookups][21]. Zwykle
nie trzeba nic dodatkowo zmieniać w systemie, jednak trzeba mieć na uwadze fakt, że jeśli korzystamy
z oprogramowania oferującego cache DNS, np. `dnsmasq` , to stosowny wpis w jego konfiguracji musimy
zamieścić, by nam ten cały mechanizm działał prawidłowo, przykładowo:

    ...
	server=::1#5333
	server=127.0.2.1#5333
    server=/mhouse/192.168.1.1#53
    server=/1.168.192.in-addr.arpa/192.168.1.1
    ...

W tym przypadku, wszystkie zapytania o domeny będą  przesyłane do serwera DNS, który nasłuchuje na
porcie `5333` (tutaj akurat jest to `dnscrypt-proxy` ). Niemniej jednak, robimy wyjątek dla
lokalnej domeny `mhouse` i zapytania o hosty w tej domenie będą przesyłane na adres naszego routera
WiFi i tam rozwiązywane. Ostatnia linijka konfiguruje właśnie reverse DNS lookups , tak by zapytania
o adresy IP z zakresu 192.168.1.0/24, były przesyłane również do routera WiFi, bo tylko tam może
zostać na nie udzielona odpowiedź.

By przetestować czy reverse DNS lookups działa prawidłowo, możemy posłużyć się do tego celu
narzędziem `nslookup` :

    $ nslookup janek-laptop
    Server:         127.0.0.1
    Address:        127.0.0.1#53

    Name:   janek-laptop.mhouse
    Address: 192.168.1.193
    Name:   janek-laptop.mhouse
    Address: fda9:1892:e9f3::707

    $ nslookup 192.168.1.193
    193.1.168.192.in-addr.arpa      name = janek-laptop.mhouse.

    Authoritative answers can be found from:

Jak widzimy, przy próbie odpytania po IP o nazwę hosta, został zwrócony
`name = janek-laptop.mhouse` , zatem wszystko działa poprawnie.

## Usługa avahi-daemon.service

[Avahi][9] to system, który ułatwia wykrywanie usług w sieci lokalnej za pośrednictwem zestawu
protokołów mDNS/DNS-SD ([Multicast DNS][15]/[DNS Service Discovery][14]). Umożliwia to podłączenie
laptopa lub komputera do sieci i natychmiastowe wyświetlenie innych osób, z którymi można czatować.
Podobnie sprawa wygląda w przypadku drukarek, którymi można coś wydrukować lub też z udostępnionymi
plikami, które możemy przeglądać. Pliki, o których mowa, to zasoby CIFS/SMB/SAMBA, dlatego też
`avahi-daemon` musi być aktywny w systemie, by bez problemów te zasoby mogły być automatycznie
odnalezione. Kompatybilna technologia znajduje się w systemach Apple MacOS X (pod nazwą "Bonjour",
czasami zwaną też "[Zeroconf][16]").

### Konfiguracja avahi-daemon (plik /etc/avahi/avahi-daemon.conf)

Konfiguracja `avahi-daemon` odbywa się przez plik `/etc/avahi/avahi-daemon.conf` , w którym to
musimy zmienić kilka rzeczy. Poniżej znajduje się zawartość mojego pliku:

    [server]
    host-name=morfikownia
    domain-name=local
    #browse-domains=
    use-ipv4=yes
    use-ipv6=no
    allow-interfaces=bond0
    #deny-interfaces=eth1
    check-response-ttl=yes
    #use-iff-running=no
    enable-dbus=yes
    disallow-other-stacks=yes
    #allow-point-to-point=no
    cache-entries-max=4096
    clients-max=4096
    objects-per-client-max=1024
    entries-per-entry-group-max=32
    ratelimit-interval-usec=1000000
    ratelimit-burst=1000

    [wide-area]
    enable-wide-area=no

    [publish]
    disable-publishing=no
    disable-user-service-publishing=no
    add-service-cookie=yes
    publish-addresses=yes
    publish-hinfo=no
    publish-workstation=no
    publish-domain=yes
    #publish-dns-servers=127.0.0.1
    publish-resolv-conf-dns-servers=yes
    publish-aaaa-on-ipv4=yes
    publish-a-on-ipv6=no

    [reflector]
    enable-reflector=no
    #reflect-ipv=no
    #reflect-filters=_airplay._tcp.local,_raop._tcp.local

    [rlimits]
    #rlimit-as=
    rlimit-core=0
    rlimit-data=8388608
    rlimit-fsize=0
    rlimit-nofile=32
    rlimit-stack=8388608
    rlimit-nproc=3

Większość użytych parametrów raczej jest zrozumiała, jeśli jednak mamy z nimi problemy, to warto
zajrzeć do [man avahi-daemon.conf][8]. Na wyjaśnienie zasługuje w zasadzie jedynie `domain-name=` .
Najwyraźniej nazwa domeny `local` jest swojego rodzaju standardem i sporo systemów i usług w nich
korzysta właśnie z tej nazwy. Jeśli nie chcemy zmieniać konfiguracji pozostałych systemów, to
dobrze jest w tym parametrze określić właśnie `local` jako nazwę domeny.

Po zapisaniu konfiguracji i zrestartowaniu usługi `avahi-daemon.service` dobrze jest przetestować
czy nazwy hostów w domenie `.local` są poprawnie rozwiązywane na adresy IP i podobnie też w drugą
stronę:

    $ avahi-resolve -n -4 morfikownia.local
    morfikownia.local       192.168.1.150

    $ avahi-resolve -a 192.168.1.150
    192.168.1.150   morfikownia.local

Jeśli mamy w sieci jakieś usługi, to możemy sprawdzić czy `avahi-browse` je odnajdzie:

    $ avahi-browse -art
    +  bond0 IPv4 LibreELEC                                     SSH Remote Terminal  local
    +  bond0 IPv4 LibreELEC                                     SFTP File Transfer   local
    +  bond0 IPv4 root@LibreELEC: Dummy Output                  PulseAudio Sound Sink local
    =  bond0 IPv4 LibreELEC                                     SSH Remote Terminal  local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [22]
       txt = []
    +  bond0 IPv4 root@LibreELEC                                PulseAudio Sound Server local
    =  bond0 IPv4 LibreELEC                                     SFTP File Transfer   local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [22]
       txt = ["u=root" "path=/storage"]
    =  bond0 IPv4 root@LibreELEC: Dummy Output                  PulseAudio Sound Sink local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [4713]
       txt = ["icon-name=computer" "class=abstract" "description=Dummy Output" "subtype=virtual" "channel_map=front-left,front-right" "format=s16le" "channels=2" "rate=44100" "device=auto_null" "cookie=0x51620f5e" "fqdn=LibreELEC" "uname=Linux aarch64 6.6.28 #1 SMP Thu Apr 25 10:02:13 UTC 2024" "machine-id=1c89e42466b4894bf5195fc45caf6b34" "user-name=root" "server-version=pulseaudio 17.0"]
    +  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-events._udp    local
    +  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-jsonrpc._tcp   local
    =  bond0 IPv4 root@LibreELEC                                PulseAudio Sound Server local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [4713]
       txt = ["cookie=0x51620f5e" "fqdn=LibreELEC" "uname=Linux aarch64 6.6.28 #1 SMP Thu Apr 25 10:02:13 UTC 2024" "machine-id=1c89e42466b4894bf5195fc45caf6b34" "user-name=root" "server-version=pulseaudio 17.0"]
    +  bond0 IPv4 Kodi (LibreELEC)                              Web Site             local
    +  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-jsonrpc-h._tcp local
    +  bond0 IPv4 LIBREELEC                                     Device Info          local
    +  bond0 IPv4 MORFIKOWNIA                                   Device Info          local
    +  bond0 IPv4 LIBREELEC                                     Microsoft Windows Network local
    +  bond0 IPv4 MORFIKOWNIA                                   Microsoft Windows Network local
    =  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-events._udp    local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [9777]
       txt = []
    =  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-jsonrpc._tcp   local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [9090]
       txt = ["uuid=29bb0091-9d68-4e3f-8360-3df837e16f2e" "txtvers=1"]
    =  bond0 IPv4 Kodi (LibreELEC)                              Web Site             local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [8080]
       txt = ["uuid=29bb0091-9d68-4e3f-8360-3df837e16f2e" "txtvers=1"]
    =  bond0 IPv4 Kodi (LibreELEC)                              _xbmc-jsonrpc-h._tcp local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [8080]
       txt = ["uuid=29bb0091-9d68-4e3f-8360-3df837e16f2e" "txtvers=1"]
    =  bond0 IPv4 LIBREELEC                                     Device Info          local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [0]
       txt = ["model=Xserve"]
    =  bond0 IPv4 MORFIKOWNIA                                   Device Info          local
       hostname = [morfikownia.local]
       address = [192.168.1.150]
       port = [0]
       txt = ["org.freedesktop.Avahi.cookie=2384891314" "model=Xserve"]
    =  bond0 IPv4 LIBREELEC                                     Microsoft Windows Network local
       hostname = [LibreELEC.local]
       address = [192.168.1.238]
       port = [445]
       txt = []
    =  bond0 IPv4 MORFIKOWNIA                                   Microsoft Windows Network local
       hostname = [morfikownia.local]
       address = [192.168.1.150]
       port = [445]
       txt = ["org.freedesktop.Avahi.cookie=2384891314"]

W sieci mam maszynkę RPI z zainstalowanym [LibreELEC][10] i ta maszyna sporo usług zgłasza. Jest
też moja morfikownia, z usługą CIFS/SMB/SAMBA. Zatem konfiguracja demona Avahi zdaje się być
poprawna.

## Usługi smbd.service oraz nmbd.service

Jeśli zaś chodzi o pakiet `samba` , to interesować nas będą dwie usługi, tj. `smbd.service`
uruchamiający [demona smbd][3] oraz `nmbd.service` uruchamiający [demona nmbd][2] . Poza tymi
usługami trzeba też będzie rzucić okiem na plik z konfiguracją `/etc/samba/smb.conf` .

Jak już zostało wyżej wspomniane, w przypadku linux'ów i nowszych windows'ów możemy zrezygnować z
obsługi NetBIOS w naszym serwerze CIFS/SMB/SAMBA. Co ciekawe usługa `nmbd.service` posiada warunek
do uruchomienia:

    ExecCondition=/usr/share/samba/is-configured nmb

W skrypcie `/usr/share/samba/is-configured` mamy m.in. tę poniższą zwrotkę:

    ( nmb | nmbd )
        [ "$addc" = 1 ] && exit 1
        disable_netbios=$(testparm -s --parameter-name="disable netbios" 2>/dev/null)
        [ Yes = "$disable_netbios" ] && exit 1 || exit 0
        ;;

Zatem jeśli w pliku `/etc/samba/smb.conf` znajdzie się parametr `disable netbios = yes` , to demon
`nmbd` nie zostanie uruchomiony -- trzeba o tym pamiętać.

## Konfiguracja CIFS/SMB/SAMBA (plik /etc/samba/smb.conf)

Jeśli chodzi o plik konfiguracyjny `/etc/samba/smb.conf` , to trzeba będzie go nieco dostosować.
Poniżej znajduje się zawartość mojego pliku:

    #======================= Global Settings =======================

    [global]

        server role = standalone server
        server string = morfikownia
        server smb encrypt = disabled
        netbios name = MORFIKOWNIA
        workgroup = MHOUSE
        interfaces = lo 192.168.1.0/24
        bind interfaces only = yes
        client min protocol = SMB3_11
        server min protocol = SMB3_11
        client ipc min protocol = SMB3_11
        ntlm auth = ntlmv2-only
        #security = user

        #name resolve order = lmhosts wins host bcast
        name resolve order = host
        hostname lookups = yes
        dns proxy = no
        hosts allow = 127.0.0.0/8, 192.168.1.150, 192.168.1.128, janek-laptop.local, janek-laptop.mhouse
        hosts deny = ALL

        deadtime = 30

        wins support = no
        disable netbios = yes
        disable spoolss = yes

        usershare allow guests = no
        restrict anonymous = 2

        browseable = yes
        read only = yes
        printable = no

        map to guest = never
        #map to guest = Bad User

        invalid users = root nobody
        valid users = samba sambina
        #guest account = nobody

        create mask = 0660
        directory mask = 0770
        mangled names = no
        #mangled names = illegal

        load printers = no
        printing = cups
        printcap name = cups
        #printcap name = /etc/printcap

        enable core files = yes

        passdb backend = tdbsam
        #passdb backend = smbpasswd
        #smb passwd file = /etc/samba/smbpasswd
        #username map = /etc/samba/smbusers


        # samba tuning options
        socket options = TCP_NODELAY IPTOS_LOWDELAY
        min receivefile size = 16384
        aio read size = 16384
        aio write size = 16384
        use sendfile = yes

        local master = yes
        preferred master = auto
        domain master = auto
        os level = 20

        strict allocate = no

        fruit:model = Xserve

        #logging = file
        #log file = /var/log/samba/log.%m.
        #max log size = 1000
        log level = 5

        panic action = /usr/share/samba/panic-action %d

        obey pam restrictions = yes
        unix password sync = yes
        passwd program = /usr/bin/passwd %u
        passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
        pam password change = yes


    #======================= Share Definitions =======================

    [samba-android]
        comment = Samba na /media/Android/
        path = /media/Android/samba
        available = yes
        browseable = yes
        read only = yes
        printable = no
        guest ok = no
        write list = samba
        read list = samba sambina

    [samba-zami]
        comment = Samba na /media/Zami/
        path = /media/Zami/samba
        available = yes
        browseable = yes
        read only = yes
        printable = no
        guest ok = no
        write list = samba
        read list = samba sambina


Konfiguracja w pliku `/etc/samba/smb.conf` jest podzielona na dwie sekcje. Wszystko co jest pod
`[global]` określa ustawienia globalne dla serwera CIFS/SMB/SAMBA. Mamy także dwie zwrotki
`[samba-*]` , które są odpowiedzialne za sekcję usług. Część parametrów konfiguracyjnych jest
charakterystyczna jedynie dla sekcji globalnej. Natomiast wszystkie parametry z sekcji usług można
dodać również do sekcji globalnej. W ten sposób można zdefiniować konfigurację dla wszystkich usług
w jednym podejściu i mogą one robić za ustawienia domyślne.

Konfiguracja, którą wyżej widzimy, ma w założeniu umożliwić dostęp do dwóch katalogów, tj.
`/media/Android/samba/` oraz `/media/Zami/samba/` . Dostęp do tych katalogów ma być wyłącznie dla
zalogowanych użytkowników ( `usershare allow guests = no` oraz `guest ok = no` ), którzy będą w
stanie jedynie odczytać zawartość tych folderów bez praw do zapisu ( `read only = yes` oraz
`printable = no` ). W tym modelu, dostęp dla anonimowych użytkowników jest wyłączony i tylko
podanie nazwy użytkownika i hasła będzie skutkować możliwością przeglądania zasobów.

### Parametr "restrict anonymous"

Mogłoby się wydawać, że skoro mamy wyłączony anonimowy dostęp do zasobów, to nie da się zalogować
na nasz serwer CIFS/SMB/SAMBA anonimowo. Wygląda jednak na to, że część funkcjonalności serwera
jest dostępna bez logowania, o czym możemy się przekonać wydając w terminalu poniższe polecenie:

    $ net rpc user
    Password for [MHOUSE\morfik]:
    Anonymous login successful
    samba
    sambina

Jak widać, nawet nie podając hasła, zostaje zwrócony komunikat `Anonymous login successful` oraz
lista użytkowników serwera CIFS/SMB/SAMBA.

Wszystko dlatego, że domyślnie w konfiguracji SAMBA mamy ustawiony parametr
`restrict anonymous = 0` , który umożliwia anonimowy dostęp do usług SAMRP (Security Account
Manager Remote Protocol) oraz LSA DCERPC (Local Security Authority, DCE 1.1: Remote Procedure Call).
Anonimowy dostęp do tych usług jest wymagany podczas przeglądania zasobów przez stare klienty,
które polegają na NetBIOS. Nowsze wersje windows nie powinny mieć problemów, z ustawieniem
restrykcji dla anonimowych zapytań, choć niektóre aplikacje wciąż mogą polegać na anonimowym
dostępie.

W zasadzie to `restrict anonymous` możemy ustawić na `1` i przez to wyłączyć anonimowy dostęp do
SAMRP, lub też `2` , co zaowocuje dodatkowo brakiem pozwolenia na anonimowe połączenia do `IPC$`
(chyba że w sekcji usługi ustawi się parametr `guest ok = yes` ).

Gdy ustawimy `restrict anonymous = 2` , to powyższe polecenie zwróci nam błąd:
`NT_STATUS_ACCESS_DENIED` dla użytkowników anonimowych lub też tych, których nie ma w bazie danych:

    $ net rpc user -U morfik
    Password for [MHOUSE\morfik]:
    Anonymous login successful
    tree connect failed: NT_STATUS_ACCESS_DENIED

    $ net rpc user -U samba
    Password for [MHOUSE\samba]:
    samba
    sambina

Jeśli po ustawieniu `restrict anonymous = 2` i przeładowaniu serwera nie widzimy zasobów
CIFS/SMB/SAMBA w menadżerze plików, to prawdopodobnie trzeba zrestartować sesję graficzną.

### Minimalna wersja protokołu

Ze względów bezpieczeństwa dobrze jest ustawić minimalną wersję protokołu CIFS/SMB/SAMBA zarówno
serwera, jak i podłączających się do niego klientów. Do wyboru mamy:

 - `CORE`     - Najwcześniejsza wersja.
 - `COREPLUS` - Nieznaczne ulepszenia `CORE` w celu zwiększenia wydajności.
 - `LANMAN1`  - Pierwsza nowocześniejsza wersja protokołu. Obsługa długich nazw plików.
 - `LANMAN2`  - Aktualizacja do protokołu Lanman1.
 - `NT1`      - Aktualna wersja protokołu. Używany przez system Windows NT. Znany jako CIFS.
 - `SMB2`     - Reimplementacja protokołu SMB. Używany przez Windows Vista i późniejsze wersje
                 Windows. SMB2 ma dostępne podprotokoły. Domyślnie SMB2 wybiera wariant SMB2_10.
 - `SMB2_02`  - Najwcześniejsza wersja SMB2.
 - `SMB2_10`  - Wersja SMB2 systemu Windows 7.
 - `SMB3`     - Taki sam jak SMB2. Używany przez Windows 8. SMB3 ma dostępne podprotokoły.
                 Domyślnie SMB3 wybiera wariant SMB3_11.
 - `SMB3_00`  - Wersja SMB3 dla systemu Windows 8.
 - `SMB3_02`  - Wersja SMB3 dla systemu Windows 8.1.
 - `SMB3_11`  - Wersja SMB3 dla systemu Windows 10.

W manualu można wyczytać, że nie powinno się ustawiać na sztywno wersji protokołu SMB, ze względu
na fakt automatycznej negocjacji między klientem a serwerem przy nawiązywaniu połączenia. Ja jednak
jestem zdania, że takim automatycznym rozwiązaniom lepiej nie zawierzać i lepiej określić wersję
protokołu, z której zamierzamy korzystać:

        client min protocol = SMB3_11
        server min protocol = SMB3_11
        client ipc min protocol = SMB3_11

### Wyłączenie obsługi NetBIOS i WINS

W założeniach naszego serwera CIFS/SMB/SAMBA postawiliśmy na bezpieczeństwo i obsługę najnowszych
protokołów. Dlatego też wyłączany obsługę NetBIOS i WINS:

    wins support = no
    disable netbios = yes
    disable spoolss = yes
    dns proxy = no

### Dostosowanie parametru "name resolve order"

Po wyłączeniu NetBIOS i WINS możemy też nieco dostosować parametr `name resolve order` , w którym to
możemy określić te poniższe opcje:

  - `lmhosts` - Wyszukuje adres IP w pliku lmhosts SAMBA.
  - `host`    - Wykonuje standardowe rozwiązywanie nazwy hosta na adres IP, używając pliku
                 `/etc/hosts` lub DNS.
  - `wins`    - Wysyła zapytanie o nazwę do serwera WINS, którego adres IP figuruje w parametrze
                 WINSSERVER. Jeśli nie określono serwera WINS, ta metoda zostanie zignorowana.
  - `bcast`   - Wykonuje rozgłaszanie na każdym ze znanych interfejsów lokalnych wymienionych w
                 parametrze `interfaces` . Jest to najmniej niezawodna z metod rozwiązywania nazw,
                 ponieważ zależy ona od tego czy docelowy host znajduje się w lokalnie podłączonej
                 podsieci.

Biorąc pod uwagę te powyższe informacje, to jedyną interesującą nas opcją będzie `host` :

    #name resolve order = lmhosts wins host bcast
    name resolve order = host
    hostname lookups = yes

### Restrykcje dostępu do serwera/zasobów

Serwer CIFS/SMB/SAMBA posiada wbudowany mechanizm restrykcji dostępu, dzięki któremu jesteśmy w
stanie ograniczyć dostęp do zasobów (czy też całego serwera) określonym hostom w sieci lokalnej.
Jeśli chcemy ograniczyć dostęp do udostępnianych na naszej maszynie katalogów, to do pliku
`/etc/samba/smb.conf` trzeba dodać te dwa poniższe parametry:

    hosts allow = 127.0.0.0/8, 192.168.1.150, 192.168.1.128
    hosts deny = ALL

Ta powyższa konfiguracja zezwoli na połączenie jedynie użytkownikom z adresów `192.168.1.150` oraz
`192.168.1.128` (no i oczywiście z localhost, którego nie możemy zablokować) . Naturalnym jest fakt,
że adresy IP mogą i będą ulegać zmianie, dlatego też istnieje opcja zidentyfikowania użytkowników
po nazwie hosta, przykładowo:

    hosts allow = 127.0.0.0/8, 192.168.1.150, 192.168.1.128, janek-laptop.mhouse
    hosts deny = ALL

Te dwa parametry możemy także określić w sekcji usług i w ten sposób zezwolić określonym hostom na
dostęp do konkretnych katalogów udostępnianych na serwerze.

By restrykcje dostępu działały w oparciu o nazwy hostów, nie można zapomnieć o parametrze
`hostname lookups` .

### Użytkownicy

Wiemy już, że by być w stanie zajrzeć w udostępniony katalog, będzie trzeba się zalogować na
jakiegoś użytkownika. Pytanie tylko na jakiego użytkownika? Odpowiedź brzmi: na użytkownika
dedykowanego, którego musimy sobie utworzyć zarówno w Debianie, jak i w bazie danych SAMBA.

Zatem tworzymy użytkownika w systemie przy pomocy narzędzia `useradd` :

    # useradd -u 7777 --home /run/samba --no-create-home -s /usr/sbin/nologin samba

W tym powyższym poleceniu kluczowe jest określenie kilku dodatkowych parametrów:

 - `-u`               -- odpowiada za wybór konkretnego identyfikatora użytkownika (UID) w systemie.
                          Jeśli go nie określimy, to system przypisze dla tego konta pierwszy UID,
                          który będzie dostępny dla zwykłego użytkownika, zwykle 1001.
 - `--home`           -- zmienia lokalizację katalogu domowego. Normalnie lokalizacja tego katalogu
                          domowego wskazywała by na `/home/$USER/` , choć w tym przypadku to i tak
                          ma marginalne znaczenie.
 - `--no-create-home` -- sprawi, że katalog domowy dla tego użytkownika nie zostanie utworzony.
 - `-s`               -- ma na celu ustawić shell dla tego konta na `/usr/sbin/nologin` , co
                          efektywnie uniemożliwi temu użytkownikowi logowanie się na konto shell.

Ustawiamy także hasło dla tego konta:

    # passwd samba
    New password:
    Retype new password:
    passwd: password updated successfully

Teraz przechodzimy do utworzenia użytkownika w bazie danych SAMBA.

#### SAMBA i backend'y tdbsam oraz smbpasswd

Domyślnym backend'em dla haseł w SAMBA jest `tdbsam` , który przechowuje hasła w pliku
`/var/lib/samba/private/passdb.tdb` . Możemy także wybrać backend `smbpasswd` , który hasła trzyma
w pliku `/etc/samba/smbpasswd` ). Jedno lub drugie rozwiązanie powinno bez problemu wystarczyć dla
naszych lokalnych potrzeb.

Wybór backend'u określamy w pliku `/etc/samba/smb.conf` . Po określeniu backend'u dla haseł,
tworzymy użytkownika w bazie danych SAMBA:

    # smbpasswd -a samba
    New SMB password:
    Retype new SMB password:
    Added user samba.

lub:

    # pdbedit -a -u samba
    new password:
    retype new password:

W przypadku korzystania z backend'u `smbpasswd` jesteśmy w stanie w prosty sposób podejrzeć wpisy
w tej tekstowej bazie danych, tj. plik `/etc/samba/smbpasswd` :

    # cat /etc/samba/smbpasswd

    samba:7777:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:11437065C4BA9F865A30488E7FFCF8FA:[U          ]:LCT-663C6155:

Mamy tutaj:

  - `samba`   -- nazwa użytkownika.
  - `7777`    -- identyfikator użytkownika (UID).
  - `XX...XX` -- hash Lanman'a hasła użytkownika (w tym przypadku mamy same `X` , co oznacza, że
                  to konto jest wyłączone w systemie i nie można się na nie zalogować).
  - `11...FA` -- hash Windows NT hasła użytkownika.
  - `[ ]`     -- flagi dla konta użytkownika (w tym przypadku `U` oznacza konto zwykłego
                  użytkownika).
  - `LCT-`    -- czas ostatniej zmiany konta (Last Change Time).

Jeśli zaś chodzi o podejrzenie bazy haseł w przypadku korzystania z backend'u `tdbsam` , to tutaj
trzeba już posłużyć się narzędziem `pdbedit` , przykładowo:

    # pdbedit -L -v

    ---------------
    Unix username:        samba
    NT username:
    Account Flags:        [U          ]
    User SID:             S-1-5-21-935451880-3819277430-3145174144-1001
    Primary Group SID:    S-1-5-21-935451880-3819277430-3145174144-513
    Full Name:
    Home Directory:       \\MORFIKOWNIA\samba
    HomeDir Drive:
    Logon Script:
    Profile Path:         \\MORFIKOWNIA\samba\profile
    Domain:               MORFIKOWNIA
    Account desc:
    Workstations:
    Munged dial:
    Logon time:           0
    Logoff time:          Wed, 06 Feb 2036 16:06:39 CET
    Kickoff time:         Wed, 06 Feb 2036 16:06:39 CET
    Password last set:    Sun, 12 May 2024 09:47:02 CEST
    Password can change:  Sun, 12 May 2024 09:47:02 CEST
    Password must change: never
    Last bad password   : 0
    Bad password count  : 0
    Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

Posiadając tylko jeden wpis w bazie danych SAMBA będziemy mieli tak naprawdę możliwość dostępu do
udostępnionych katalogów podając nazwę użytkownika `samba` oraz określone dla niego hasło. Jeśli
chcielibyśmy rozdzielić dostęp na wielu użytkowników, to naturalnie trzeba dodać więcej kont
(zarówno w Debianie, jak i bazie danych SAMBA).

### Uprawnienia do zasobów

Jeśli chcemy dać użytkownikom możliwość zapisu katalogów, to wystarczy nazwy tych użytkowników
określić w linijce z `write list` w pliku `/etc/samba/smb.conf` . Niemniej jednak, trzeba także
nadać odpowiednie uprawnienia w systemie dla udostępnianych katalogów.

Tworzymy sobie w naszym Debianie przykładową grupę i dodajemy do niej wszystkich użytkowników,
którzy mają mieć możliwość korzystania z udostępnionych zasobów CIFS/SMB/SAMBA:

    # groupadd -g 8000 sambafiles
    # usermod -a -G sambafiles samba

Zmieniamy grupę na udostępnionych katalogach i nadajemy im uprawnienia `2770` :

    # chgrp -R sambafiles /media/Android/samba/
    # chmod 2770 /media/Android/samba

Cyfra `2` w `chmod 2770` odpowiada za bit SGID, tak by wszystkie nowo tworzone pliki i katalogi
dziedziczyły tę grupę.

## Sprawdzenie konfiguracji serwera CIFS/SMB/SAMBA via testparm

Gdy skończymy się bawić z konfiguracją serwera i ustawieniem jej tych wszystkich niezbędnych
parametrów, dobrze jest sprawdzić przy pomocy `testparm` czy nie narobiliśmy jakichś mniejszych czy
większych błędów. To narzędzie jest też w stanie sprawdzić czy nie korzystamy przestarzałych opcji,
oraz też podpowiedzieć nam, które opcje powinniśmy zmienić/usunąć, by nasz serwer CIFS/SMB/SAMBA,
był możliwie bezpieczny. Wpiszmy zatem w terminalu to poniższe polecenie i zobaczmy co zostanie
wydrukowane:

    # testparm -s
    Load smb config files from /etc/samba/smb.conf
    Loaded services file OK.
    Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

    Server role: ROLE_STANDALONE

    # Global parameters
    ...

Nie ma żadnych błędów czy komunikatów ostrzegawczych za wyjątkiem tego `Weak crypto is allowed by
GnuTLS` .

### Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Szukając informacji na temat tego całego `Weak crypto is allowed by GnuTLS` , [natknąłem się na ten
wątek][18], w którym ludzie piszą, że problem ze słabymi szyframi nie dotyczy bezpośrednio samego
oprogramowania SAMBA, a biblioteki GnuTLS, której konfiguracja umożliwia wykorzystanie
starych/połamanych już szyfrów, np. TLS_1.0. Gdzieniegdzie też można przeczytać, że ten komunikat
pojawia się w przypadku, gdy serwer SAMBA nie spełnia [warunków FIPS][19], czyli np. nie
wykorzystuje [protokołu Kerberos][20] podczas uwierzytelniania użytkowników przy dostępie do
udostępnianych zasobów. Raczej nic w tej sprawie nie da się zrobić bez odpowiedniej konfiguracji
biblioteki GnuTLS i/lub protokołu Kerberos w SAMBA.

## Demon gvfsd i backend'y gvfs

[Gnome Virtual File System][5] (GnomeVFS, gvfs) to wirtualny system plików przestrzeni użytkownika,
w którym punkty montowania działają jako oddzielne procesy, z którymi można się komunikować za
pośrednictwem magistrali D-Bus. Gvfs zawiera również moduł GIO (Gnome Input/Output), który dodaje
obsługę gvfs do wszystkich aplikacji korzystających z API GIO.

W skład gvfs wchodzi kilka usług, które będą podnoszone na żądanie w sesji użytkownika, np. przy
uruchomieniu po raz pierwszy aplikacji, która korzysta z tej funkcjonalności. Później przykład:

    ├─systemd(4551)─┬─(sd-pam)(4552)
    ...
    │               ├─gvfs-afc-volume(5586)─┬─{gvfs-afc-volume}(5587)
    │               │                       ├─{gvfs-afc-volume}(5588)
    │               │                       ├─{gvfs-afc-volume}(5589)
    │               │                       └─{gvfs-afc-volume}(5591)
    │               ├─gvfs-goa-volume(5581)─┬─{gvfs-goa-volume}(5582)
    │               │                       ├─{gvfs-goa-volume}(5583)
    │               │                       └─{gvfs-goa-volume}(5584)
    │               ├─gvfs-gphoto2-vo(5592)─┬─{gvfs-gphoto2-vo}(5593)
    │               │                       ├─{gvfs-gphoto2-vo}(5594)
    │               │                       └─{gvfs-gphoto2-vo}(5596)
    │               ├─gvfs-mtp-volume(5597)─┬─{gvfs-mtp-volume}(5598)
    │               │                       ├─{gvfs-mtp-volume}(5599)
    │               │                       └─{gvfs-mtp-volume}(5601)
    │               ├─gvfs-udisks2-vo(5564)─┬─{gvfs-udisks2-vo}(5566)
    │               │                       ├─{gvfs-udisks2-vo}(5567)
    │               │                       ├─{gvfs-udisks2-vo}(5568)
    │               │                       └─{gvfs-udisks2-vo}(5570)
    │               ├─gvfsd(4810)─┬─gvfsd-computer(758797)─┬─{gvfsd-computer}(758798)
    │               │             │                        ├─{gvfsd-computer}(758799)
    │               │             │                        └─{gvfsd-computer}(758800)
    │               │             ├─gvfsd-dnssd(5678)─┬─{gvfsd-dnssd}(5679)
    │               │             │                   ├─{gvfsd-dnssd}(5680)
    │               │             │                   └─{gvfsd-dnssd}(5681)
    │               │             ├─gvfsd-http(783582)─┬─{gvfsd-http}(783583)
    │               │             │                    ├─{gvfsd-http}(783584)
    │               │             │                    └─{gvfsd-http}(783585)
    │               │             ├─gvfsd-network(5659)─┬─{gvfsd-network}(5660)
    │               │             │                     ├─{gvfsd-network}(5661)
    │               │             │                     ├─{gvfsd-network}(5662)
    │               │             │                     └─{gvfsd-network}(5664)
    │               │             ├─gvfsd-smb(2501577)─┬─{gvfsd-smb}(2501578)
    │               │             │                    ├─{gvfsd-smb}(2501579)
    │               │             │                    ├─{gvfsd-smb}(2501580)
    │               │             │                    └─{gvfsd-smb}(2501583)
    │               │             ├─gvfsd-smb(2508944)─┬─{gvfsd-smb}(2508945)
    │               │             │                    ├─{gvfsd-smb}(2508946)
    │               │             │                    ├─{gvfsd-smb}(2508947)
    │               │             │                    └─{gvfsd-smb}(2508950)
    │               │             ├─gvfsd-smb-brows(5665)─┬─{gvfsd-smb-brows}(5666)
    │               │             │                       ├─{gvfsd-smb-brows}(5667)
    │               │             │                       ├─{gvfsd-smb-brows}(5668)
    │               │             │                       └─{gvfsd-smb-brows}(5670)
    │               │             ├─gvfsd-trash(5605)─┬─{gvfsd-trash}(5606)
    │               │             │                   ├─{gvfsd-trash}(5607)
    │               │             │                   ├─{gvfsd-trash}(5608)
    │               │             │                   └─{gvfsd-trash}(5612)
    │               │             ├─gvfsd-wsdd(9603)─┬─{gvfsd-wsdd}(9605)
    │               │             │                  ├─{gvfsd-wsdd}(9606)
    │               │             │                  └─{gvfsd-wsdd}(9607)
    │               │             ├─{gvfsd}(4853)
    │               │             ├─{gvfsd}(4854)
    │               │             └─{gvfsd}(4857)
    │               ├─gvfsd-fuse(4888)─┬─{gvfsd-fuse}(4903)
    │               │                  ├─{gvfsd-fuse}(4907)
    │               │                  ├─{gvfsd-fuse}(4908)
    │               │                  ├─{gvfsd-fuse}(4909)
    │               │                  ├─{gvfsd-fuse}(4910)
    │               │                  ├─{gvfsd-fuse}(4932)
    │               │                  ├─{gvfsd-fuse}(994851)
    │               │                  └─{gvfsd-fuse}(1831004)
    │               ├─gvfsd-metadata(5622)─┬─{gvfsd-metadata}(5623)
    │               │                      ├─{gvfsd-metadata}(5624)
    │               │                      └─{gvfsd-metadata}(5625)
    ...

Oczywiście nie wszystkie procesy widoczne wyżej są odpowiedzialne za realizację postawionego w tym
artykule zadania i nie wszystkie też muszą być uruchomione w naszym Debianie. Dużo zależy od tego
jakie programy mamy w swoim linux'ie, bo to za ich sprawą te procesy będą uruchamiane automatycznie
w chwili, gdy jakaś aplikacja użytkownika będzie chciała skorzystać z danej funkcjonalności
oferowanej przez te procesy/usługi gvfs. Nam najbardziej zależy by uruchomione były
`gvfsd-smb-browse` , `gvfsd-smb` , `gvfsd-network` , `gvfsd-dnssd` , bo to one nam są potrzebne do
prawidłowej integracji protokołu CIFS/SMB/SAMBA w menadżerach plików.

## Testy konfiguracji

Pora odpalić wszystkie te usługi i sprawdzić czy będziemy w stanie uzyskać dostęp do lokalnego
zasobu CIFS/SMB/SAMBA:

    # systemctl restart smbd.service avahi-daemon.service

By przetestować konfigurację, w terminalu wpisujemy poniższe polecenie:

    $ smbclient -U samba -L morfikownia.local
    Password for [MHOUSE\samba]:

            Sharename       Type      Comment
            ---------       ----      -------
            samba-android   Disk      Samba na /media/Android/
            samba-zami      Disk      Samba na /media/Zami/
            IPC$            IPC       IPC Service (morfikownia)
    SMB1 disabled -- no workgroup available

Gdy podamy błędne hasło albo innego użytkownika, powinniśmy otrzymać poniższy komunikat:

    $ smbclient -U samba -L morfikownia.local

    Password for [MHOUSE\samba]:
    session setup failed: NT_STATUS_LOGON_FAILURE

Spróbujmy podejrzeć listę plików:

    $ smbclient -U samba //morfikownia.local/samba-android
    Password for [MHOUSE\samba]:
    Try "help" to get a list of possible commands.
    smb: \> ls
      .                                   D        0  Mon May 13 19:27:56 2024
      ..                                  D        0  Mon May 13 19:27:56 2024
      test                                A        0  Mon May 13 19:27:49 2024
      test2                               D        0  Mon May 13 19:28:00 2024

                    164029204 blocks of size 1024. 114452312 blocks available
    smb: \>

Wygląda na to, że jesteśmy się w stanie lokalnie podłączyć do tego zasobu CIFS/SMB/SAMBA,
przynajmniej korzystając z konsolowego narzędzia `smbclient` . Nas jednak interesuje jak sprawa
wygląda w przypadku menadżera plików, np. `nemo` po przejściu na zakładkę `network` . W tym
przypadku lokalne zasoby CIFS/SMB/SAMBA zostały wyświetlone bez większych problemów:

|   |   |
|---|---|
| ![cifs-smb-samba-file-manager-nemo-connect-listing-share](/img/2024/06/001.cifs-smb-samba-file-manager-nemo-connect-listing-share.png#huge) | ![cifs-smb-samba-file-manager-nemo-connect-listing-share](/img/2024/06/002.cifs-smb-samba-file-manager-nemo-connect-listing-share.png#huge) |

|   |   |
|---|---|
| ![cifs-smb-samba-file-manager-nemo-connect-listing-share](/img/2024/06/003.cifs-smb-samba-file-manager-nemo-connect-listing-share.png#huge) | ![cifs-smb-samba-file-manager-nemo-connect-listing-share](/img/2024/06/004.cifs-smb-samba-file-manager-nemo-connect-listing-share.png#huge) |

Oczywiście nie wdrożyliśmy sobie wszystkich tych usług, by cieszyć się funkcjonalnością oferowaną
przez CIFS/SMB/SAMBA na localhost. Trzeba teraz jeszcze skonfigurować naszą maszynę do pracy w
sieci, tj. trzeba jej dodać kilka regułek dla firewall'a `nftables` / `iptables` , tak by choć
trochę ograniczyć różnym osobom dostęp do naszego komputera.

Jeśli udostępniliśmy użytkownikom możliwość zapisu zasobów CIFS/SMB/SAMBA, to możemy przetestować
jak wygląda wgrywanie plików, np. przy pomocy pierwszego lepszego menadżera plików. Po utworzeniu
testowego pliku i katalogu listujemy lokalnie zasób CIFS/SMB/SAMBA:

    # ls -al /media/Android/samba
    total 12
    drwxrws---  3 morfik sambafiles 4096 2024-05-13 19:27:56 ./
    drwxr-xr-x 17 morfik morfik     4096 2024-05-10 20:54:58 ../
    drwxrws---  2 samba  sambafiles 4096 2024-05-13 19:28:00 test2/
    -rw-rw----  1 samba  sambafiles    0 2024-05-13 19:27:49 test

Pliki zostały z powodzeniem utworzone. Właściciel plików odpowiada temu użytkownikowi, na którego
się zalogowaliśmy przy dostępie do zasobu CIFS/SMB/SAMBA. Grupa jest dziedziczona w nowo utworzonych
plikach/katalogach. Prawa dostępu do katalogów/plików wskazują na 770/660, czyli są zgodne z maską
ustawioną w pliku `/etc/samba/smb.conf` . Zatem wszystko zostało skonfigurowane poprawnie i działa
bez zarzutu.

### Zdalne podłączenie

Możemy teraz przy pomocy innej maszyny spróbować podłączyć się do naszego serwera CIFS/SMB/SAMBA.
Po podłączeniu, wszystkie aktywne połączenia będą widoczne w `smbstatus` , przykładowo:

    # smbstatus

    Samba version 4.20.1-Debian
    PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing
    ----------------------------------------------------------------------------------------------------------------------------------------
    28993   samba        samba        Redmi-Note-9-Pro.mhouse (ipv4:192.168.1.128:48112) SMB3_11           -                    partial(AES-128-CMAC)

    Service      pid     Machine       Connected at                     Encryption   Signing
    ---------------------------------------------------------------------------------------------
    samba-android 28993   Redmi-Note-9-Pro.mhouse Mon Jun  3 16:57:36 2024 CEST    -            AES-128-CMAC
    samba-zami   28993   Redmi-Note-9-Pro.mhouse Mon Jun  3 16:57:39 2024 CEST    -            AES-128-CMAC


    Locked files:
    Pid          User(ID)   DenyMode   Access      R/W        Oplock           SharePath   Name   Time
    --------------------------------------------------------------------------------------------------
    28993        7777       DENY_WRITE 0x120089    RDONLY     NONE             /media/Zami/samba   Film.mkv   Mon Jun  3 17:00:44 2024

Widzimy tutaj połączenie z adresu `192.168.1.128` , które korzysta z protokołu `SMB3_11` i uzyskało
dostęp do zasobów `samba-android` oraz `samba-zami` po uprzednim zalogowaniu się na użytkownika
`samba` .  Dodatkowo widzimy, że plik  `Film.mkv` jest w użyciu (film oglądany w mpv ze smartfona z
androidem).

Wszystkie zamontowane zasoby będą dostępne pod `/run/user/1000/gvfs/smb-share:*/` i bez większego
problemu będziemy w stanie takie zasoby przeglądać również za pośrednictwem terminala.

## Konfiguracja iptables/nfstables na potrzeby CIFS/SMB/SAMBA

Skupimy się tutaj w zasadzie jedynie na konfiguracji `nftables` , choć reguły powinno się dać bez
większego problemu przetłumaczyć na `iptables` .

### Ruch przychodzący (INPUT)

Osoby, które filtrują jedynie ruch przychodzący, mają w miarę ułatwione zadanie, bo potrzebują w
zasadzie poniższych regułek:

    table inet filter {
        chain INPUT {
            type filter hook input priority 0; policy drop;
            ...
            tcp flags syn / fin,syn,rst,ack ct state { new } counter jump input-tcp
            meta l4proto udp ct state { new } counter jump input-udp
            ...
        }

        chain input-tcp {
        #   tcp dport { 137 } ip saddr 192.168.1.0/24 counter accept comment "samba smb netbios name service"
        #   tcp dport { 139 } ip saddr 192.168.1.0/24 counter accept comment "samba smb netbios session service"
            tcp dport { 445 } ip saddr 192.168.1.0/24 counter accept comment "samba smb active directory"
        #	tcp dport { 3702 } ip saddr { 192.168.1.0/24 } counter accept comment "wsdd Unicast SOAP HTTP WS-Discovery responder"
        #	tcp dport { 5355 } ip saddr { 192.168.1.0/24 } counter accept comment "wsdd Unicast LLMNR responder"
        }

        chain input-udp {
        #   udp dport { 137 } ip saddr 192.168.1.0/24 counter accept comment "samba smb netbios name service"
        #   udp dport { 138 } ip saddr 192.168.1.0/24 counter accept comment "samba smb netbios datagram service"
            udp dport { 445 } ip saddr 192.168.1.0/24 counter accept comment "samba smb active directory"
        #   udp dport { 3702 } udp sport { 3702 } ip daddr { 239.255.255.250 } counter accept comment "wsdd WSD multicast address"
        #   udp dport { 3702 } udp sport { 3702 } ip6 daddr { ff02::c } counter accept comment "wsdd WSD multicast address"
        #   udp sport { 3702 } ip saddr { 192.168.1.0/24 } counter accept comment "wsdd UDP"
        #   udp dport { 5355 } udp sport { 5355 } ip daddr { 224.0.0.252 } counter accept comment "wsdd LLMNR"
        #   udp dport { 5355 } udp sport { 5355 } ip6 daddr { ff02::1:3 } counter accept comment "wsdd LLMNR"
            udp sport { 5353 } udp dport { 5353 } ip daddr { 224.0.0.251 } counter accept comment "Bonjour/mDNS"
        }
    }

Jak widać wyżej ruch został rozbity na dwa łańcuchy w zależności od protokołu TCP i UDP. Po słówku
`comment` jest krótki opis reguły. Szereg reguł jest wykomentowanych, ze względu na fakt, że w
przypadku naszego serwera CIFS/SMB/SAMBA został wyłączony NetBIOS i nie ma potrzeby otwierać na
firewall'u portów `137/tcp`, `139/tcp` , `137/udp` oraz `138/udp` . Zbędne nam też są reguły z
portami `3702/tcp` , `3702/udp` , `5355/tcp` i `5355/udp` , bo nie korzystamy z demona `wsdd` .
Jedyne co musi zostać otwarte, to porty od usługi Active Directory ( `445/tcp` i `445/udp` ) oraz
port `5353/udp` , na którym operuje Avahi i realizuje rozwiązywanie nazw na adresy IP. Jeśli
jednak korzystamy ze starszych wersji windows, to te reguły na zaporze jak najbardziej trzeba
uwzględnić.

### Ruch wychodzący (OUTPUT)

Jeśli ktoś filtruje, tak jak ja, ruch wychodzący z maszyny, to trzeba napisać dodatkowo regułki dla
poszczególnych demonów/usług/procesów, tak by były w stanie wysyłać pakiety sieciowe bez przeszkód.

    chain OUTPUT {
        type filter hook output priority 0; policy drop;
        ...
        socket cgroupv2 level 1 "morfikownia/" counter jump check-cgroup
        ...
    }

	chain check-cgroup {
        ...
        socket cgroupv2 level 2 "morfikownia/user/" jump check-cgroup-user
        socket cgroupv2 level 2 "morfikownia/system/" jump check-cgroup-system
        ...
	}

	chain check-cgroup-system {
		...
		socket cgroupv2 level 3 "morfikownia/system/samba/" udp sport { 137 } ip daddr 192.168.1.0/24 counter accept comment "samba smb netbios name service"
		socket cgroupv2 level 3 "morfikownia/system/samba/" tcp sport { 137 } ip daddr 192.168.1.0/24 counter accept comment "samba smb netbios name service"
		socket cgroupv2 level 3 "morfikownia/system/samba/" udp dport { 138 } ip daddr 192.168.1.0/24 counter accept comment "samba smb netbios datagram service"
		socket cgroupv2 level 3 "morfikownia/system/samba/" tcp dport { 139 } ip daddr 192.168.1.0/24 counter accept comment "samba smb netbios session service"
		socket cgroupv2 level 3 "morfikownia/system/samba/" tcp dport { 445 } ip daddr 192.168.1.0/24 counter accept comment "samba smb active directory"
		socket cgroupv2 level 3 "morfikownia/system/samba/" udp dport { 445 } ip daddr 192.168.1.0/24 counter accept comment "samba smb active directory"
		socket cgroupv2 level 3 "morfikownia/system/avahi/" udp dport { 5353 } ip daddr { 224.0.0.251 } counter accept comment "Bonjour/mDNS/Avahi"
		socket cgroupv2 level 3 "morfikownia/system/avahi/" udp dport { 5353 } udp sport { 5353 } counter accept comment "Bonjour/mDNS/Avahi"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" udp dport { 3702 } ip daddr { 239.255.255.250 } counter accept comment "wsdd WSD multicast address"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" udp dport { 3702 } ip6 daddr { ff02::c } counter accept comment "wsdd WSD multicast address"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" udp dport { 3702 } udp sport { 3702 } ip daddr { 192.168.1.0/24 } counter accept comment "wsdd UDP"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" tcp dport { 3702 } ip daddr { 192.168.1.0/24 } counter accept comment "wsdd Unicast SOAP HTTP WS-Discovery responder"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" tcp dport { 5355 } tcp sport { 5355 } ip daddr { 192.168.1.0/24 } counter accept comment "wsdd Unicast LLMNR responder"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" udp dport { 5355 } udp sport { 5355 } ip daddr { 224.0.0.252 } counter accept comment "wsdd LLMNR"
		#socket cgroupv2 level 3 "morfikownia/system/wsdd/" udp dport { 5355 } udp sport { 5355 } ip6 daddr { ff02::1:3 } counter accept comment "wsdd LLMNR"
        ...
	}

Powyższe reguły `nftables` zakładają wykorzystanie [mechanizmu kernela cgroups w wersji 2][13].
Ścieżki typu `morfikownia/system/samba/` odpowiadają pozycjom w drzewie katalogów pod
`/sys/fs/cgroup/` i w każdym takim podfolderze są dodawane numerki PID określonych procesów za
sprawą demona `cgrulesengd` z pakietu `cgroup-tools` , przykładowo:

    *:wsdd               cpu,memory,pids,io     morfikownia/system/wsdd/
    *:wsdd2              cpu,memory,pids,io     morfikownia/system/wsdd/
    *:smbd               cpu,memory,pids,io     morfikownia/system/samba/
    *:nmbd               cpu,memory,pids,io     morfikownia/system/samba/
    *:avahi-daemon       cpu,memory,pids,io     morfikownia/system/avahi/

Jak skonfigurować system do pracy z cgroupsv2 jest poza zakresem tego artykułu. Niemniej jednak,
mechanizm cgroups nie jest niezbędny w konfiguracji `nftables` i można bez większego problemu się
bez niego obyć, tylko trzeba zastosować inne reguły filtrujące OUTPUT (porty i adresy są podane
wyżej).

### Test firewall'a

Po zaaplikowaniu reguł, próbujemy odwiedzić zasoby CIFS/SMB/SAMBA z naszej maszyny lub/i innych
maszyn i sprawdzamy czy zdefiniowane reguły rejestrują jakieś pakiety. Jeśli nie widzimy przy tym,
by ruch był w jakiś sposób blokowany, oznacza to, że zapora sieciowa realizuje swoje zadanie
poprawnie, poniżej przykład:

    # nft list chain inet filter input-tcp
    table inet filter {
        chain input-tcp {
            tcp dport 445 ip saddr 192.168.1.0/24 counter packets 7 bytes 420 accept comment "samba smb active directory"
        }
    }

    # nft list chain inet filter input-udp
    table inet filter {
        chain input-udp {
            udp dport 445 ip saddr 192.168.1.0/24 counter packets 0 bytes 0 accept comment "samba smb active directory"
            udp sport 5353 udp dport 5353 ip daddr 224.0.0.251 counter packets 73 bytes 25054 accept comment "Bonjour/mDNS/Avahi"
        }
    }

## Apparmor

By dodatkowo poprawić bezpieczeństwo systemu,  `nmbd` , `smbd` oraz `avahi-daemon` mają
dedykowane profile dla apparmor (w pakiecie `apparmor-profiles` ). Jeśli planujemy udostępniać
zasoby w sieci, to dobrze jest sobie te profile aktywować.

## Wpisy w /etc/fstab

Oczywiście ten cały skonfigurowany wyżej mechanizm nie jest niezbędny, by nasz linux był w stanie
przeglądać zasoby CIFS/SMB/SAMBA innych użytkowników w sieci. Jeśli nie planujemy udostępniać
katalogów z naszej maszyny, to możemy jedynie się ograniczyć do zwykłych wpisów w pliku
`/etc/fstab` , przykład:

    //192.168.1.238/bigdata  /media/rpi-tv-bigdata  cifs guest,domain=domainauto,users,noauto,nofail,nodev,nosuid,noexec,vers=3.11,iocharset=utf8 0 0

    //192.168.1.238/bigdata  /media/rpi-tv-bigdata  cifs username=user,password=pass,domain=domainauto,users,noauto,nofail,nodev,nosuid,noexec,vers=3.11,iocharset=utf8, 0 0

    //192.168.1.238/bigdata  /media/rpi-tv-bigdata  cifs credentials=/home/morfik/.smbcredentials,users,noauto,nofail,nodev,nosuid,noexec,vers=3.11,iocharset=utf8 0 0

Mamy tutaj trzy opcje montowania zdalnych zasobów CIFS/SMB/SAMBA. Pierwsza pozycja montuje zasób
niewymagający jakiegokolwiek uwierzytelnienia. Druga zaś zasób, który takiego uwierzytelnienia
wymaga, a dane do logowania określone są przez `username=user` , `password=pass` oraz
`domain=domainauto` . Natomiast ostatnia pozycja jest podobna do poprzedniej, z tą różnicą, że dane
do logowania są określone w zewnętrznym pliku przy pomocy
`credentials=/home/morfik/.smbcredentials` , który ma poniższą postać:

    username=user
    password=pass
    domain=domainauto

### Różnica między mount.cifs i mount.smb3

W powyższym przykładzie w pliku `/etc/fstab` typ systemu plików określiliśmy na `cifs` . Niemniej
jednak, możemy tam także sprecyzować `smb3` , przykładowo:

    //192.168.1.238/bigdata  /media/rpi-tv-bigdata  smb3 guest,users,noauto,nofail,nodev,nosuid,noexec,vers=3.11,iocharset=utf8 0 0

Jaka jest różnica między `cifs` i `smb3` ? W zasadzie różnica sprowadza się do wersji protokołu SMB.
Jeśli nie mamy pojęcia jaka wersja protokołu SMB jest wykorzystywana na serwerze, którego zasób
chcemy zamontować, to lepiej jest określić `cifs` . Natomiast jeśli wiemy, że na serwerze jest
wykorzystywany protokół SMB w wersji 3 (i wyższych), wtedy możemy bez problemu skorzystać z `smb3` .

Wersję protokołu CIFS/SMB/SAMBA możemy określić także za pomocą parametru `vers=` w pliku
`/etc/fstab` . Do wyboru mamy:

  - `1.0`            - protokół CIFS/SMBv1.
  - `2.0`            - protokół SMBv2.002. Wprowadzony po raz pierwszy w Windows Vista Service Pack
                        1 oraz Windows Server 2008. Trzeba zaznaczyć tutaj, że pierwsze wydanie
                        Windows Vista wykorzystywało nieco inny dialekt, tj. 2.000, który nie jest
                        wspierany.
  - `2.1`            - protokół SMBv2.1, który został wprowadzony w Microsoft Windows 7 i Windows
                        Server 2008R2.
  - `3.0`            - protokół SMBv3.0, który został wprowadzony w Microsoft Windows 8 i Windows
                        Server 2012.
  - `3.02` / `3.0.2` - protokół SMBv3.0.2, który został wprowadzony w Microsoft Windows 8.1 i
                        Windows Server 2012R2.
  - `3.1.1` / `3.11` - protokół SMBv3.1.1, który został wprowadzony w Microsoft Windows 10 i
                        Windows  Server 2016.
  - `3`              - protokół SMBv3.0 i wyżej.
  - `default`        - próbuje negocjować najwyższą wersję protokołu SMB2+ wspieraną zarówno przez
                        klienta jak i serwer.

Warto w tym miejscu dodać, że jeśli z jakichś względów mamy problemy z zamontowanie zdalnego zasobu
CIFS/SMB/SAMBA, to zmiana wersji w parametrze `vers=` może pomóc. Dobrze jest też rzucić okiem na
konfigurację serwera, a konkretnie na te dwa parametry: `client min protocol` oraz
`server min protocol` .

### Kwestia cifsd

Posiadając odpowiednie wpisy w pliku `/etc/fstab` możemy zamontować zdalny zasób wydając poniższe
polecenie w terminalu (jako zwykły użytkownik):

    $ mount /media/rpi-tv-bigdata

W ten sposób zostanie uruchomiony kernelowski demon `cifsd` , który będzie realizował połączenie, i
ten demon pozostanie aktywny do momentu odmontowania zdalnego zasobu. Trzeba to wziąć pod uwagę,
gdy filtrujemy OUTPUT, tak by nie zapomnieć dodać odpowiedniej regułki zezwalającej naszej maszynie
na połączenie na porty 139/tcp i 445/tcp:

    chain OUTPUT {
        ...
        tcp dport { 139, 445 } ip daddr { 192.168.1.0/24 }  counter accept comment "kernel cifsd"
        ...
    }

## Zapisywanie haseł SAMBA w keepassxc

Menadżery plików pokroju `nemo` są w stanie współpracować z menadżerem haseł `keepassxc` za sprawą
obsługi [Secret Services][7] ( `libsecret` ). Jeśli mamy wspomniany menadżer haseł i włączymy w nim
integrację Secret Services (Tools > Settings > Secret-Service-Integration), to od tego momentu
`keepassxc` będzie odpytywany o dane uwierzytelniające ilekroć będziemy chcieli zamontować zdalny
zasób CIFS/SMB/SAMBA. Tego typu integracja z menadżerem haseł może nam zaoszczędzić sporo czasu, bo
nie będziemy musieli pamiętać danych logowania i podawać ich ilekroć tylko będziemy z takich zasobów
korzystać.

![cifs-smb-samba-password-keepassxc](/img/2024/06/005.cifs-smb-samba-password-keepassxc.png#huge)

## Problemy z wydajnością CIFS/SMB/SAMBA przez GVFS

Wygląda na to, że ta opisana tutaj automatyczna konfiguracja zasobów CIFS/SMB/SAMBA ma dość spore
problemy z wydajnością, gdy wykorzystywany jest GVFS. W moim przypadku osiągane transfery plików są
na poziomie 35-40MiB/s, co jest wynikiem dość słabym. Dla porównania, transfery osiągane, gdy ten
sam zasób jest montowany z wykorzystaniem polecenia `mount` i pliku `/etc/fstab` , wysycają
praktycznie w całości łącze gigabitowe. [Ten problem jest znany od lat i do tej pory nie został
rozwiązany][22]. Zatem jeśli zależy nam na wydajności i szybkim transferze (zwłaszcza dużych
plików), to CIFS/SMB/SAMBA przez GVFS raczej odpada.

## Kwestia domeny .local

W mojej sieci domowej od lat była skonfigurowana domena `.mhouse` . Niemniej jednak, w tym artykule
została skonfigurowana również druga domena, tj. `.local` . Pytanie nasuwa się wiec takie, czy
potrzebne nam są dwie domeny, czy wystarczy jedna z nich?

Teoretycznie wystarczyłaby nam jedynie domena `.local` , bo rozwiązywanie nazw w protokole mDNS na
tej domenie domyślnie operuje. Niemniej jednak, ta domena jest dość wyjątkowa i [została
zarezerwowana na potrzeby specjalnego wykorzystania][23]. W tym podlinkowanym artykule są
wymienione za i przeciw dla stosowania tej domeny w sieciach lokalnych/domowych. Ja jednak
skłaniałbym się do korzystania z dwóch osobnych domen, tj. jedna (w tym przypadku `.mhouse` ) dla
sieci lokalnej, druga zaś `.local` dla mechanizmu mDNS.

Nic raczej nie stoi na przeszkodzie, by korzystać jedynie z domeny `.local` w sieciach, gdzie mamy
obecne jedynie same maszyny z system operacyjnym na bazie linux'a. Trzeba jednak pamiętać, że
domyślna konfiguracja systemu (określona w pliku `/etc/nsswitch.conf` ) będzie wymagać od nas, by
`avahi-daemon` był w naszym Debianie aktywny. Chodzi generalnie o to, że `avahi-daemon` będzie
rozwiązywał zapytania w domenie `.local` i  jeśli go zabraknie, to zapytania o tę domenę nie będą
rozwiązywane już dalej przez system DNS. Przynajmniej taka jest domyślna konfiguracja, którą
naturalnie możemy sobie zmienić.

## Podsumowanie

Trzeba tutaj wyraźnie zaznaczyć, że mechanizm opisany w niniejszym artykule nie jest niezbędny do
przeglądania sieciowych zasobów CIFS/SMB/SAMBA. Jeśli znamy adres, pod którym taki zasób się
znajduje, to bez większego problemu możemy zamontować go ręcznie przy pomocy `mount` podając
wszystkie niezbędne parametry, bądź też możemy posłużyć się plikiem `/etc/fstab` i w nim
zdefiniować potrzebne opcje montowania zdalnych katalogów. Wdrożenie opisanych w tym artykule kilku
dodatkowych usług ma jedynie na celu zautomatyzować cały proces odnajdywania i montowania tych
zasobów, tak by wszystkie one były dostępne w zakładce sieć/network w menadżerze plików, przez co
może nam to zaoszczędzić sporo czasu. Oczywiście, jeśli mamy w sieci tylko dwa komputery, to
raczej nie potrzebujemy tych wszystkich usług, ale gdy mamy do czynienia z większą ilością maszyn i
dojdą nam do tego różne systemy operacyjne (Windows/Linux/Android), to dobrze sobie taki mechanizm
zaimplementować, trzeba tylko pamiętać o jego zabezpieczeniu.


[1]: https://github.com/Netgear/wsdd2
[2]: https://www.samba.org/samba/docs/current/man-html/nmbd.8.html
[3]: https://www.samba.org/samba/docs/current/man-html/smbd.8.html
[4]: https://www.samba.org/
[5]: https://wiki.gnome.org/Projects/gvfs
[6]: http://www.ubiqx.org/cifs/
[7]: https://c3pb.de/blog/keepassxc-secrets-service.html
[8]: https://linux.die.net/man/5/avahi-daemon.conf
[9]: https://avahi.org/
[10]: https://libreelec.tv/
[11]: https://kapitanhack.pl/2019/12/09/killchain/atak-na-uzytkownikow-w-sieci-lokalnej-poprzez-sfalszowanie-komunikacji-llmnr-i-nbt-ns-przyklad-i-porady/
[12]: https://en.wikipedia.org/wiki/Link-Local_Multicast_Name_Resolution
[13]: https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html
[14]: http://www.dns-sd.org/
[15]: http://www.multicastdns.org/
[16]: http://www.zeroconf.org/
[17]: https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/
[18]: https://bugzilla.samba.org/show_bug.cgi?id=14583
[19]: https://ubuntu.com/security/fips
[20]: https://en.wikipedia.org/wiki/Kerberos_(protocol)
[21]: https://www.cloudflare.com/learning/dns/glossary/reverse-dns/
[22]: https://gitlab.gnome.org/GNOME/gvfs/-/issues/292
[23]: https://en.wikipedia.org/wiki/.local
