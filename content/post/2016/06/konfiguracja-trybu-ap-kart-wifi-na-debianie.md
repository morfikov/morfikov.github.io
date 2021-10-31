---
author: Morfik
categories:
- Linux
date: "2016-06-04T15:26:58Z"
date_gmt: 2016-06-04 13:26:58 +0200
published: true
status: publish
tags:
- wifi
- debian
- sieć
GHissueID: 354
title: Konfiguracja trybu AP kart WiFi na Debianie
---

Jakiś czas temu dokonałem zakupu adaptera WiFi [TP-Link
TL-WN722n](/post/recenzja-karta-wifi-tp-link-tl-wn722n/), a to z tego względu, że
potrzebowałem zewnętrznej karty sieciowej do mojego laptopa. Chodziło generalnie o to, że ten
wbudowany w niego broadcom nie był w stanie robić kilku użytecznych rzeczy, min. testów
penetracyjnych mojej bezprzewodowej sieci domowej. Jak się później okazało, ten zakupiony adapter
posiada też dodatkowy ficzer, którym jest tryb AP (Access Point). Wprawdzie ta karta nie może się
równać z routerami WiFi, bo te zwykle mają więcej anten, z których każda jest lepszej jakości ale
jesteśmy w stanie połączyć ze sobą bezprzewodowo kilka stacji roboczych. Trzeba jednak wziąć po
uwagę, że zasięg jak i transfer będą w dużej mierze ograniczone. W tym wpisie postaramy się
przerobić zwykłą maszynę, na której jest zainstalowany Debian, na punkt dostępowy sieci WiFi.

<!--more-->
## Czy moja karta WiFi obsługuje tryb AP

Zwykle karty WiFi nie posiadają trybu AP, a jedynie tryb STA (Station). Czasem też jest dostępny
dodatkowy tryb Monitor. Jeśli dana karta nie posiada trybu AP, to bezpośrednia komunikacja dwóch
maszyn nie jest możliwa. Jeśli nie wiemy czy nasza karta WiFi potrafi przełączyć się w tryb AP,
możemy to sprawdzić przez wydanie poniższego polecenia:

    # iw list
    Wiphy phy0
    ...
        Supported interface modes:
            * IBSS
            * managed
            * AP
            * AP/VLAN
            * monitor
            * mesh point
            * P2P-client
            * P2P-GO
    ...

## Firmware i oprogramowanie

Adapter TP-Link TL-WN722n do poprawnego działania wymaga doinstalowania odpowiedniego
[firmware](https://pl.wikipedia.org/wiki/Firmware). W Debianie, pakietów zawierających firmware do
określonych urządzeń jest dość sporo, a to jaki pakiet musimy zainstalować możemy odczytać z logu
systemowego po podpięciu adaptera do portu USB. Poniżej przykład:

    usb 1-3: New USB device found, idVendor=0cf3, idProduct=9271
    ...
    usb 1-3: ath9k_htc: Firmware htc_9271.fw requested
    ...

Zatem potrzebny jest nam plik `htc_9271.fw` . Jeśli mamy problemy z odnalezieniem odpowiedniego
pakietu, to zawsze możemy [przeszukać ich zawartość przy pomocy
apt-file](/post/przeszukiwanie-zawartosci-pakietow-apt-file/). Tak czy inaczej, w
tym przypadku trzeba zainstalować pakiet `firmware-atheros` . Po zainstalowaniu firmware podłączamy
ponownie adapter do portu USB. W tym momencie powinniśmy mieć w systemie nowy interfejs sieciowy,
`wlan*` .

Musimy także zainstalować odpowiednie oprogramowanie. W sumie będą to trzy pakiety: `hostapd` ,
`dnsmasq` i `crda` . Jeśli ktoś miał do czynienia z [OpenWRT](https://openwrt.org/), to powinien
kojarzyć te narzędzia. W OpenWRT są one wykorzystywane dokładnie w tym samym celu, w którym my
zamierzamy ich używać na Debianie. Jeśli chcemy połączyć tylko dwie maszyny ze sobą, to
niekoniecznie potrzebujemy [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) . Odpowiada on
bowiem za usługi takie jak DNS i DHCP, z których możemy bez problemu zrezygnować na rzecz ręcznej
[konfiguracji połączenia WiFi](/post/konfiguracja-polaczenia-wifi-pod-debianem/) na
maszynie klienckiej. Natomiast, jeśli w grę wchodzi więcej maszyn, to praktycznym rozwiązaniem
byłoby zaprzęgnąć serwer DHCP, który automatycznie będzie konfigurował wszystkie stacje robocze. W
obu przypadkach możemy korzystać z zewnętrznego serwera DNS albo też skonfigurować sobie [szyfrowany
DNS przy pomocy
DNScrypt-proxy](/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/).

## Konfiguracja interfejsu punktu dostępowego

By AP był w stanie się komunikować z bezprzewodowymi stacjami roboczymi, musimy skonfigurować mu
interfejs sieciowy. Edytujemy zatem plik `/etc/network/interfaces` i dopisujemy w nim ten poniższy
blok kodu:

    auto wlan1
    #allow-hotplug wlan1
    iface wlan1 inet static
        address 192.168.20.1
        network 192.168.20.0/24
        netmask 255.255.255.0
        broadcast 192.168.20.255

Konfiguracja dla tego interfejsu musi być przypisana statycznie. Podnosimy teraz interfejs i w tej
chwili karta powinna już posiadać adres IP:

    # ifup wlan1

    # ip addr show wlan1
    11: wlan1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
        link/ether e8:94:f6:1e:15:e9 brd ff:ff:ff:ff:ff:ff
        inet 192.168.20.1/24 brd 192.168.20.255 scope global wlan1
           valid_lft forever preferred_lft forever

## Konfiguracja hostapd

Przechodzimy do konfiguracji `hostapd` . Edytujemy plik `/etc/default/hostapd` i dostosowujemy w nim
tę poniższą linijkę:

    DAEMON_CONF="/etc/hostapd/hostapd.conf"

Z pakietem `hostapd` jest dostarczany obszerny plik konfiguracyjny `hostapd.conf.gz` , który
znajduje się w katalogu `/usr/share/doc/hostapd/examples/` . Wypakowujemy go i kopiujemy do wyżej
określonej lokalizacji:

    # zcat /usr/share/doc/hostapd/examples/hostapd.conf.gz > /etc/hostapd/hostapd.conf

Następnie edytujemy plik `hostapd.conf` , tak by zawierał poniższą konfigurację:

    interface=wlan1
    driver=nl80211
    logger_syslog=-1
    logger_syslog_level=2
    logger_stdout=-1
    logger_stdout_level=2
    ctrl_interface=/var/run/hostapd
    ctrl_interface_group=0
    ssid=Winter_Is_Coming
    utf8_ssid=1
    country_code=PL
    ieee80211d=1
    ieee80211h=1
    hw_mode=g
    channel=11
    acs_num_scans=10
    beacon_int=100
    dtim_period=2
    max_num_sta=255
    rts_threshold=2347
    fragm_threshold=2346
    preamble=1
    macaddr_acl=0
    auth_algs=1
    ignore_broadcast_ssid=0
    wmm_enabled=0
    wmm_ac_bk_cwmin=4
    wmm_ac_bk_cwmax=10
    wmm_ac_bk_aifs=7
    wmm_ac_bk_txop_limit=0
    wmm_ac_bk_acm=0
    wmm_ac_be_aifs=3
    wmm_ac_be_cwmin=4
    wmm_ac_be_cwmax=10
    wmm_ac_be_txop_limit=0
    wmm_ac_be_acm=0
    wmm_ac_vi_aifs=2
    wmm_ac_vi_cwmin=3
    wmm_ac_vi_cwmax=4
    wmm_ac_vi_txop_limit=94
    wmm_ac_vi_acm=0
    wmm_ac_vo_aifs=2
    wmm_ac_vo_cwmin=2
    wmm_ac_vo_cwmax=3
    wmm_ac_vo_txop_limit=47
    wmm_ac_vo_acm=0
    ap_max_inactivity=300
    skip_inactivity_poll=0
    disassoc_low_ack=1
    ap_isolate=1
    ieee80211n=1
    ht_capab=[HT40-][SHORT-GI-20][SHORT-GI-40][RX-STBC1][MAX-AMSDU-3839][DSSS_CCK-40]
    require_ht=0
    eapol_version=2
    eapol_key_index_workaround=0
    eap_reauth_period=3600
    eap_server=0
    own_ip_addr=127.0.0.1
    wpa=2
    wpa_psk=
    wpa_psk_radius=0
    wpa_key_mgmt=WPA-PSK WPA-PSK-SHA256
    wpa_pairwise=CCMP
    rsn_pairwise=CCMP
    wpa_group_rekey=600
    wpa_gmk_rekey=21600
    wpa_ptk_rekey=600
    peerkey=1
    okc=0
    ap_table_max_size=255
    ap_table_expiration_time=3600
    wps_state=2
    wps_independent=0
    ap_setup_locked=1
    uuid=7b3fb0ea-f068-446d-b803-7c1924b69613
    wps_pin_requests=/var/run/hostapd_wps_pin_requests
    device_name=Winter_Is_Coming
    device_type=1-0050F204-1
    config_methods=push_button
    wps_rf_bands=g
    time_zone=CET-1CEST,M3.5.0/2,M10.5.0/3
    access_network_type=0
    internet=1

W `wpa_psk` wpisujemy hasło do sieci WiFi. Możemy tutaj określić zwykłe hasło (8-63 znaków), lub też
możemy je wygenerować przy pomocy `wpa_passphrase` . Istnieje także możliwość sprecyzowania więcej
niż jednego hasła do sieci WiFi. W tym celu potrzebny będzie osobny plik, za który odpowiada
parametr `wpa_psk_file` . My tutaj ograniczymy się do jednego hasła zakodowanego w base64. Nazwę
sieci wpisujemy w `ssid` . Musimy także określić parametr `ht_capab` , który różni się w zależności
od posiadanego sprzętu. By odpowiednio uzupełnić wartość tej opcji, musimy zobaczyć co nam zwróci
polecenie `iw list` :

    # iw list
    ...
        Band 1:
            Capabilities: 0x116e
                HT20/HT40
                SM Power Save disabled
                RX HT20 SGI
                RX HT40 SGI
                RX STBC 1-stream
                Max AMSDU length: 3839 bytes
                DSSS/CCK HT40
    ...

Każda z tych możliwości, które zostały wypisane powyżej, musi zostać uwzględniona w `ht_capab` .
Jako, że ta karta obsługuje szerokość kanałów 40 MHz, to określamy `[HT40-]` (dla kanałów 5-13) lub
`[HT40+]` (dla kanałów 1-7) w zależności od tego, który kanał wybraliśmy. Dla `RX HT20 SGI` oraz `RX
HT40 SGI` dodajemy `[SHORT-GI-20]` i `[SHORT-GI-40]` . W przypadku `RX STBC 1-stream` dajemy
`[RX-STBC1]` . Następnie mamy `Max AMSDU length: 3839 bytes` i tutaj dajemy `[MAX-AMSDU-3839]`
(długość musi pasować). Ostatnia możliwość to `DSSS/CCK HT40` i w jej przypadku ustawiamy
`[DSSS_CCK-40]` . Jeśli nasza karta dysponuje innymi możliwościami, to trzeba zajrzeć w opis
konfiguracji `hostapd` , gdzie jest krótka instrukcja na temat uzupełniania tych możliwości. Więcej
informacji na temat wykorzystanych wyżęj parametrów można znaleźć
[tutaj](https://wireless.wiki.kernel.org/en/users/Documentation/hostapd). Zapisujemy plik i
restartujemy demona `hostapd` .

## Skanowanie eteru w poszukiwaniu AP

Nasz AP powinien już rozgłaszać w eterze swoją obecność. Przechodzimy na stację kliencką i skanujemy
z niej eter w poszukiwaniu punktu dostępowego, który utworzyliśmy. Można to zrobić na kilka
sposobów. My skorzystamy do tego celu z narzędzia `wpa_cli` , które jest dostarczane z pakietem
`wpasupplicant` :

![wpa_cli-ap-access-point-punkt-dostepu-skanowanie](/img/2015/11/1.wpa_cli-ap-access-point-punkt-dostepu-skanowanie.png#huge)

Widzimy, że nasz AP jest na liście.

## Konfiguracja dnsmasq

Dostęp do sieci nam się na wiele nie zda bez odpowiedniej konfiguracji interfejsów na kliencie.
Jeśli mamy jedną maszynę, to raczej możemy ustawić wszystko ręcznie. Natomiast, jeśli posiadamy
wiele komputerów, które chcemy podłączyć do internetu, to dobrze jest skonfigurować sobie serwer
DHCP i DNS. Za te dwa w/w odpowiada narzędzie `dnsmasq` . Konfiguracja serwera DHCP i DNS trzymana
jest w pliku `/etc/dnsmasq.conf` ale zanim tam przejdziemy, zajrzyjmy pierw do pliku
`/etc/default/dnsmasq` i upewnijmy się, że `dnsmasq` jest włączony. Poniżej zaś znajdują się wpisy,
które trzeba umieścić w pliku `dnsmasq.conf` :

    domain-needed
    bogus-priv
    local=/mhouse/
    interface=wlan1
    no-hosts
    expand-hosts
    domain=mhouse.lh
    dhcp-range=192.168.20.100,192.168.20.250,2h
    #read-ethers
    dhcp-lease-max=150
    dhcp-leasefile=/etc/dnsmasq.leases
    dhcp-authoritative
    cache-size=1000
    no-negcache

Gdybyśmy chcieli statyczne lease DHCP, możemy usunąć hash z linijki zawierającej `read-ethers` . W
takim przypadku będzie czytany plik `/etc/ethers` , w którym znajdują się party MAC-IP. Możemy także
konfigurować hosty bezpośrednio w konfiguracji `dnsmasq` przy pomocy wpisów takich jak ten poniżej:

    dhcp-host=c0:cb:38:01:f0:f5,morfikownia,192.168.20.200,2h

Powyższa linijka przydziela hostowi o adresie MAC `c0:cb:38:01:f0:f5` adres IP `192.168.20.200` i
nazwę `morfikownia` na `2` godziny.

Od tego momentu interfejsy sieciowe maszyn klienckich będą konfigurowane za pomocą protokołu DHCP.
Pamiętajmy by odpowiednio dostosować plik `/etc/network/interfaces` na wszystkich klientach.

## Podłączanie się do AP

Tak powyżej skonfigurowana sieć będzie dostępna dla klientów, którzy będą posiadać poniższą
konfigurację wpisaną do pliku `/etc/wpa_supplicant/wpa_supplicant.conf` :

    network={
          id_str="home_WiFi_static"
          priority=5
          ssid="Winter_Is_Coming"
          bssid=e8:94:f6:1e:15:e9
          psk=
          proto=RSN
          key_mgmt=WPA-PSK-SHA256
          pairwise=CCMP
          group=CCMP
          auth_alg=OPEN
          scan_ssid=0
          disabled=0
    }

Sprawdźmy czy klientowi zostanie przydzielony adres IP:

    # ifup wlan0
    ...
    DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 6
    DHCPREQUEST on wlan0 to 255.255.255.255 port 67
    DHCPOFFER from 192.168.20.1
    DHCPACK from 192.168.20.1
    bound to 192.168.20.200 -- renewal in 2903 seconds.

Jeśli planujemy wdrożyć jakiś filtr pakietów na punkcie dostępowym, trzeba uwzględnić w nim regułki
przepuszczające ruch na portach 53/udp (DNS) oraz 67/udp i 68/udp (DHCP).

## AP z dostępem do internetu

Stworzony w powyższy sposób bezprzewodowy punkt dostępowy nie będzie miał połączenia z internetem. W
przypadku jednak gdybyśmy chcieli go podłączyć pod tę sieć globalną, to musimy przeprowadzić kilka
dodatkowych czynności. Przede wszystkim włączamy forwarding pakietów w kernelu poprzez dopisanie do
pliku `/etc/sysctl.conf` tych poniższych parametrów:

    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1

Musimy także odpowiednio skonfigurować sobie [skrypt
firewall'a](/post/firewall-na-linuxowe-maszyny-klienckie/):

    iptables -t nat -A POSTROUTING -o eth0 -s 192.168.20.0/24 -d 0/0 -j MASQUERADE

    iptables -t filter -A fw-interfaces -i wlan1 -o eth0 -s 192.168.20.0/24 -j ACCEPT

Parametr `-o eth0` określa, przez który interfejs jest wyjście na świat. W taki oto sposób mamy
bardzo niskobudżetowy AP sieci WiFi z Debianem na pokładzie.
