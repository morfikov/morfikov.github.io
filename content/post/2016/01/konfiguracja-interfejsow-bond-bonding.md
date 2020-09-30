---
author: Morfik
categories:
- Linux
date: "2016-01-02T15:07:13Z"
lastmod: 2020-03-29 16:20:00 +0100
published: true
status: publish
tags:
- debian
- moduły-kernela
- wifi
- systemd
- sieć
- bonding
title: Jak skonfigurować bonding w Debian linux (eth0+wlan0)
---

W artykule poświęconym [konfiguracji sieci WiFi na Debianie z wykorzystaniem narzędzia
wpa_supplicant][1] wspomniałem parę słów na temat [interfejsu bond][2]. Bonding na linux
wykorzystywany jest w zasadzie do spięcia kilku interfejsów sieciowych, w tym przewodowych
( `eth0` ) i bezprzewodowych ( `wlan0` ) w jeden (zwykle `bond0` ). Takie rozwiązanie sprawia, że
w przypadku awarii któregoś z podpiętych interfejsów, my nie tracimy połączenia z siecią i nie
musimy nic nigdzie przełączać, by to połączenie przywrócić. To rozwiązanie jest o tyle użyteczne,
że w przypadku, gdy podepniemy przewód do gniazda RJ-45 w naszym laptopie, to komunikacja będzie
odbywać się po kablu. Natomiast jeśli przewód zostanie odłączony, to system automatycznie przejdzie
na komunikację bezprzewodową. W tym wpisie spróbujemy zaprojektować sobie właśnie tego typu
mechanizm zarówno za sprawą pakietu `ifupdown` , gdzie konfiguracja interfejsów sieciowych jest
zarządzana przez plik `/etc/network/interfaces` , jak i przy pomocy natywnego rozwiązania jakie
oferuje systemd/networkd.

<!--more-->
## Sterowniki i firmware do karty WiFi

By nasza karta bezprzewodowa WiFi działała bez problemów z bonding'iem, musi [być ona wspierana
przez kernel linux'a][3]. Dodatkowo, potrzebne nam będą odpowiednie pakiety z firmware. Na Debianie,
wszystko to, co tyczy się firmware jest ulokowane w pakietach mających nazwę `firmware-*` ,
przykładowo: `firmware-atheros` . Przy pomocy narzędzia `lspci` możemy ustalić jaką mamy kartę WiFi
w laptopie i w ten sposób dociągnąć stosowny pakiet za sprawą menadżera pakietów
`apt-get`/`aptitude` .

## Konfiguracja bonding'u przy pomocy ifupdown

Standardowo w Debianie do konfiguracji sieci wykorzystuje się pakiet `ifupdown` . Nie ma potrzeby
jego ręcznej instalacji, bo powinien już siedzieć w naszym systemie. Musimy jednak doinstalować
inny pakiet: `ifenslave` . Bez niego nie będziemy w stanie skonfigurować interfejsu `bond0` . Poza
tymi pakietami, potrzebny nam będzie też moduł kernela `bonding` , który musimy załadować wraz ze
startem systemu. Dopisujemy zatem do pliku `/etc/modules` tę poniższą linijkę:

    bonding

Konfiguracja interfejsu `bond0` może być podana bezpośrednio przy ładowaniu modułu jako jego opcje.
My jednak ten interfejs skonfigurujemy sobie przez plik `/etc/network/interfaces` . U mnie ten plik
wygląda mniej więcej tak:

    source /etc/network/interfaces.d/*

    auto lo
    iface lo inet loopback

    auto bond0
    iface bond0 inet dhcp
        metric 10
        bond-mode active-backup
        bond-miimon 100
        bond-downdelay 200
        bond-updelay 200
        bond-primary eth0
        bond-primary-reselect always
        bond-fail-over-mac none
        bond-slaves eth0 wlan0

    allow-bond0 wlan0
    iface wlan0 inet manual
        bond-give-a-chance 2
        wpa-driver nl80211
        wpa-debug-level -1
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Jak można wyżej zauważyć, nie mamy tam bloku odpowiedzialnego za konfigurację interfejsu
przewodowego `eth0` , a to z tego względu, że nie jest ona konieczna. Natomiast wymagany jest blok
z konfiguracją interfejsu bezprzewodowego `wlan0` . To w nim określamy kilka niezbędnych
parametrów dla połączenia WiFi.

Wyjaśnienie opcji użytych w bloku `iface wlan0` :

 - `bond-give-a-chance` -- czeka określoną ilość sekund po podniesieniu interfejsu bezprzewodowego.
                             Chodzi o opóźnienia związane z procesem logowania do sieci.
 - `wpa-driver` -- sterownik dla karty WiFi. Zwykle tutaj wykorzystywany jest `nl80211` ale w
                     przypadku niektórych kart może być wymagane ustawienie tutaj `wext` .
 - `wpa-debug-level` -- obniża poziom logowania, tak by nie łapać komunikatów `WPA: Group rekeying
                          completed` .
 - `wpa-conf` -- plik z konfiguracją sieci WiFi (np. dane logowania).

Przydałoby się także kilka słów wyjaśnień dotyczących samego bloku `iface bond0` . Każdy interfejs,
nawet te wirtualne, muszą posiadać jakiś adres MAC. Jako, że pod interfejs `bond0` są podpięte dwa
fizyczne interfejsy, to każdy z nich ma inny adres MAC. Gdy opcja `bond-fail-over-mac` jest
ustawiona na `none` , wszystkie zniewolone interfejsy, w tym sam interfejs `bond0` , mają ustawiony
dokładnie ten sam adres MAC. Adres MAC jest pobierany z pierwszego interfejsu określonego w
`bond-slaves` . Zatem interfejs `bond0` będzie miał taki sam MAC, co interfejs `eth0` . Każdy
dodatkowy interfejs podczepiany pod interfejs `bond0` , również będzie miał ten sam adres MAC.
Chodzi generalnie o to, że [standard IEEE 802.11 nie zezwala na transmisję pakietów o innym adresie
MAC][4] niż ten przypisany na sztywno bezprzewodowej karcie sieciowej. Dlatego też ten adres MAC
musi pasować do tego, który jest ustawiony na interfejsie `bond0` .

Opcje `bond-downdelay` oraz `bond-updelay` odpowiadają za czas (ms) podnoszenia i opuszczania
interfejsów. Ważne jest, by ustawić je na wielokrotność wartości określonej przez parametr
`bond-miimon` . Dalej mamy już tylko zdefiniowany główny interfejs na pozycji `bond-primary` oraz
politykę jego wybierania w `bond-primary-reselect` . W przypadku, gdy `bond-primary-reselect` jest
ustawiony na `always` , interfejs określony w `bond-primary` będzie ustawiany jako główny za każdym
razem, gdy będzie dostępny. Takie zachowanie przydaje się w przypadku, gdy w `bond-mode` określimy
`active-backup` . Ten tryb, jak nazwa sugeruje, ma na celu wykorzystanie tylko jednego interfejsu w
tym samym czasie, ze wskazaniem na interfejs przewodowy `eth0` . Gdy zdarzy się taka sytuacja, że
odłączymy przewód sieciowy od portu RJ-45, to ten interfejs zostanie wyłączony i jednocześnie
zostanie aktywowany interfejs bezprzewodowy. Natomiast w przypadku, gdy podłączymy kabel ponownie,
to pakiety znów będą lecieć przez ten interfejs przewodowy.

Jeśli chodzi zaś o [konfigurację samej sieci WiFi][1], to została ona opisana w osobnym artykule i
nie będę poruszał tutaj tego tematu.

## Konfiguracja bonding'u w systemd/networkd

Konfiguracja interfejsu `bond0` w systemd/networkd przebiega nieco inaczej. Niemniej jednak, nic
nie stoi na przeszkodzie, by wykorzystywać ten powyższy mechanizm zamiast natywnego rozwiązania
oferowanego przez systemd. Jeśli jednak chcielibyśmy się ograniczyć jedynie do systemd, to musimy
poczynić szereg zmian w konfiguracji systemu.

Przede wszystkim musimy wyłączyć usługi `networking.service` i `ifupdown-wait-online` oraz
dezaktywować je, tak by nie były uruchamiane wraz ze startem systemu:

    # systemctl stop networking.service ifupdown-wait-online.service
    # systemctl disable networking.service ifupdown-wait-online.service

Jeśli mamy załadowany moduł `bonding` to musimy go wyładować:

    # modprobe -r bonding

Odinstalować trzeba także pakiet `ifenslave` oraz usunąć linijkę z modułem z pliku `/etc/modules` .
W ten sposób powrócimy do pozycji wyjściowej z początku artykułu.

Następnie włączamy szereg usług dostarczanych z systemd:

    # systemctl enable \
                systemd-networkd.service \
                systemd-networkd.socket \
                systemd-networkd-wait-online.service \
                systemd-timesyncd.service

Dodajemy do pliku `/etc/modprobe.d/modules.conf` opcję dla modułu `bonding` , który będzie ładowany
automatycznie za sprawą systemd:

    options bonding max_bonds=0

Widoczny wyżej parametr `max_bonds` trzeba ustawić na `0` , a to z tego względu, że zapobiegnie to
tworzeniu interfejsu `bond0` tuż po załadowaniu modułu. Ten interfejs będzie tworzony i zarządzany
przez systemd. W przypadku, gdy nie dodamy tej opcji, to [możemy mieć problem z konfiguracją
interfejsu bond0][5].

Tworzymy teraz kilka plików w katalogu `/etc/systemd/network/` . Poniżej przykłady.

Plik `10-bond0.netdev` :

    [NetDev]
    Description=Net device for bond0
    Name=bond0
    Kind=bond

    [Bond]
    Mode=active-backup
    MIIMonitorSec=100ms
    UpDelaySec=200ms
    DownDelaySec=200ms
    FailOverMACPolicy=none
    PrimaryReselectPolicy=always
    LACPTransmitRate=fast
    MinLinks=1

Sekcja `[NetDev]` ma za zadanie utworzyć interfejs rodzaju `bond` i nadać mu nazwę `bond0` . Zaś
zwrotka `[Bond]` konfiguruje moduł kernela `bonding` . W zasadzie wszystkie opcje określone tutaj
zostały już wyjaśnione przy okazji analizowania pliku `/etc/network/interfaces` . Jedyne co, to na
wyjaśnienie zasługuje `MinLinks` , który odpowiada za wskazanie systemowi czy ten interfejs `bond0`
działa prawidłowo. Jeśli tylko jeden z interfejsów `eth0` lub `wlan0` będzie nieaktywny (bo o to
nam chodzi), to wtedy interfejs `bond0` nie zostanie oznaczony jako niedziałający.

Plik `30-bond0-eth0.network` :

    [Match]
    MACAddress=00:21:cc:c3:05:b0

    [Network]
    Description=Slave eth0 interface for bond0
    Bond=bond0
    PrimarySlave=yes

Plik `30-bond0-wlan0.network` :

    [Match]
    MACAddress=60:67:20:42:56:9c

    [Network]
    Description=Slave wlan0 interface for bond0
    Bond=bond0

Te dwa pliki ( `30-bond0-eth0.network` oraz `30-bond0-wlan0.network` ) mają zbliżoną zawartość z
racji, że przy ich pomocy określamy interfejsy, które będą podpięte pod mechanizm bonding'u. W
sekcji `[Match]` podajemy adres MAC interfejsu `eth0` i `wlan0` , zaś w sekcji `[Network]`
wskazujemy interfejs `bond0` , bo to pod niego podpinamy te dwa fizyczne interfejsy. Dodatkowo, w
przypadku przewodowego interfejsu określony został jeszcze parametr `PrimarySlave=yes` , który
czyni z interfejsu przewodowego główny interfejs bonding'u -- to nim będą płynąć pakiety nawet w
przypadku, gdy połączenie WiFi zostanie nawiązane.

Plik `40-bond.network` :

    [Match]
    Name=bond0

    [Link]
    RequiredForOnline=yes

    [Network]
    Description=Bonding of eth0 and wlan0
    DHCP=ipv4
    DHCPServer=no
    LinkLocalAddressing=no
    DNS=127.0.2.1
    IPForward=no
    IPMasquerade=no
    IPv6PrivacyExtensions=kernel

    [DHCPv4]
    UseDNS=no
    RoutesToDNS=no
    UseSIP=no
    UseMTU=yes
    UseNTP=no
    SendHostname=no
    UseHostname=no
    UseDomains=no
    UseRoutes=yes
    UseTimezone=no
    MaxAttempts=4
    SendRelease=yes

Tutaj już jak widać tych parametrów jest nieco więcej. W sekcji `[Match]` określamy interfejs
`bond0` , który za sprawą `RequiredForOnline=yes` będzie traktowany jako swojego rodzaju główny
interfejs sieciowy maszyny, na którego konfigurację system będzie czekał podczas startu komputera.
W zwrotce `[Network]` określone zostały standardowe parametry konfiguracji interfejsu sieciowego,
ze wskazaniem na uzyskanie tej konfiguracji via protokół DHCP. Ostatnia sekcja, tj. `[DHCPv4]` ,
tyczy się konfiguracji samego protokołu DHCP dla IPv4. Te wszystkie wymienione wyżej parametry są
dokładniej opisane w [dokumentacji systemd][6].

By całość nam jeszcze zadziałała, musimy włączyć usługę `wpa_supplicant` na interfejsie `wlan0`,
która uwierzytelni nas w sieci bezprzewodowej, do której mamy zamiar się podłączyć:

    # systemctl enable wpa_supplicant.service wpa_supplicant@wlan0.service

Pozostało nam już tylko uruchomienie całego mechanizmu:

    # systemctl start systemd-networkd

W logu systemowym zaś powinniśmy odnotować m.in. ten poniższy komunikat:

    systemd-networkd[43223]: bond0: DHCPv4 address 192.168.1.150/24 via 192.168.1.1
    systemd-networkd[43223]: bond0: Configured

Zaś narzędzie `networkctl` powinno nam zwrócić poniższą konfigurację:

    # networkctl
    IDX LINK  TYPE     OPERATIONAL SETUP
      1 lo    loopback carrier     unmanaged
      2 eth0  ether    enslaved    configured
      3 wlan0 wlan     enslaved    unmanaged
      4 bond0 bond     routable    configured

    4 links listed.


[1]: /post/konfiguracja-polaczenia-wifi-pod-debianem/
[2]: https://www.kernel.org/pub/linux/kernel/people/marcelo/linux-2.4/Documentation/networking/bonding.txt
[3]: https://wiki.debian.org/WiFi
[4]: http://lists.shmoo.com/pipermail/hostap/2009-February/019190.html
[5]: https://lists.freedesktop.org/archives/systemd-devel/2015-March/029033.html
[6]: https://www.freedesktop.org/software/systemd/man/systemd.network.html
