---
author: Morfik
categories:
- Hardware
date: "2016-05-30T22:23:26Z"
date_gmt: 2016-05-30 20:23:26 +0200
published: true
status: publish
tags:
- wifi
- router
- recenzja
- tp-link
title: 'Recenzja: router TP-LINK Archer C7 v2'
---

Routery bezprzewodowe to bardzo użyteczne urządzenia i na dobrą sprawę każdy dom powinien posiadać
jeden taki wynalazek. Na rynku jest cała masa sprzętu WiFi, który w większości przypadków zaspokoi
gust kupujących ale też trzeba wziąć pod uwagę fakt, że nie wszystkie routery posiadają pożądane
przez nas ficzery. Część routerów ma tylko jedno radio, zwykle o zakresie 2,4 GHz. Te droższe modele
mają także radio 5GHz. Routery różnią się także ilością portów w switch'u, ilością pamięci RAM,
częstotliwością pracy procesora, no i rozmiarem pamięci flash. Wszystkie te cechy trzeba wziąć pod
uwagę przy próbie ewentualnego zakupu routera bezprzewodowego. Oczywiście spora część z tych
gabarytów będzie miała znaczenie tylko w przypadku zainstalowania alternatywnego firmware, np.
OpenWRT. Niemniej jednak, dobrze zdawać sobie sprawę z tego co dany router ma pod obudową. W tym
wpisie zostanie przedstawiony router [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html), obsługujący pasma 2.4 GHz i 5 GHz,
mający 128 MiB pamięci RAM, 16 MiB flash, gigabitowy switch 5-cio portowy i dwa porty USB 2.0 .

<!--more-->
## Zawartość pudełka routera TP-LINK Archer C7 v2

W pudełku mamy oczywiście router, który wygląda jak większość routerów produkowanych przez TP-LINK.
Na pierwszy rzut oka, patrząc po samej obudowie, ciężko by określić jaki to jest właściwie model:

![]({{< baseurl >}}/img/2016/05/1.tp-link-archer-c7-v2-wyglad.jpg#big)

Router działa w dwóch różnych pasmach WiFi 2,4 GHz oraz 5 GHz i ma w sumie 6 anten: 3 wewnętrzne
(2,4 GHz) oraz 3 zewnętrzne (5 GHz) widoczne na powyższej fotce. Poniżej jest kilka fotek tych anten
zewnętrznych:

![]({{< baseurl >}}/img/2016/05/2.tp-link-archer-c7-v2-antena.jpg#big)

![]({{< baseurl >}}/img/2016/05/3.tp-link-archer-c7-v2-antena.jpg#big)

![]({{< baseurl >}}/img/2016/05/4.tp-link-archer-c7-v2-antena.jpg#big)

Do zestawu jest dołączony także zasilacz 12V/2,5A. Zatem router Archer C7 v2 jest w stanie pobrać
nawet 30W, choć w rzeczywistości pobiera znacznie mniej:

![]({{< baseurl >}}/img/2016/05/5.tp-link-archer-c7-v2-zasilacz.jpg#big)

![]({{< baseurl >}}/img/2016/05/6.tp-link-archer-c7-v2-zasilacz-parametry.jpg#big)

Do zestawu jest także dołączona [skrętka KAT5](https://en.wikipedia.org/wiki/Category_5_cable),
długość około 1,2 metra:

![]({{< baseurl >}}/img/2016/05/7.tp-link-archer-c7-v2-skretka-przewod.jpg#big)

Poniżej jest jeszcze fotka obrazująca panel tylny routera. Mamy tutaj 3 gniazda na anteny, wyjście
zasilacza, dwa porty USB 2.0 (po jednej diodzie na każdy z tych portów). Dalej mamy gigabitowy
switch z wydzielonym portem WAN. Przyciski standardowe, tj. wyłącznik zasilania, wł/wył WiFi, i
reset razem z WPS:

![]({{< baseurl >}}/img/2016/05/8.tp-link-archer-c7-v2-wyglad-panel-tylny.jpg#big)

## Co router TP-LINK Archer C7 v2 ma pod maską

Przydałoby się jeszcze zajrzeć routerowi Archer C7 pod maskę i zobaczyć jakie podzespoły wchodzą w
skład całego urządzenia. Na sam początek spód obudowy, w którym jest szereg otworów wentylacyjnych
ułatwiających odprowadzanie ciepła:

![]({{< baseurl >}}/img/2016/05/9.tp-link-archer-c7-v2-wyglad-spod.jpg#big)

![]({{< baseurl >}}/img/2016/05/10.tp-link-archer-c7-v2-wyglad-spod-naklejka.jpg#big)

Poniżej są zaś fotki obrazujące poszczególne podzespoły. Widok ogólny:

![]({{< baseurl >}}/img/2016/05/11.tp-link-archer-c7-v2-podzespoly.jpg#big)

Anteny wewnętrzne:

![]({{< baseurl >}}/img/2016/05/12.tp-link-archer-c7-v2-podzespoly-antena-wewnetrza.jpg#big)

![]({{< baseurl >}}/img/2016/05/13.tp-link-archer-c7-v2-podzespoly-antena-wewnetrza.jpg#big)

![]({{< baseurl >}}/img/2016/05/14.tp-link-archer-c7-v2-podzespoly-antena-wewnetrza.jpg#big)

Szereg układów pochodzi od Qualcomm Atheros. Mamy tutaj czip WiFi 5 GHz na MiniPCIe (QCA9880-BR4A
(v2) 3x3 a/n/ac):

![]({{< baseurl >}}/img/2016/05/15.tp-link-archer-c7-v2-podzespoly-wifi-5ghz.jpg#big)

![]({{< baseurl >}}/img/2016/05/16.tp-link-archer-c7-v2-podzespoly-wifi-5ghz.jpg#big)

[System on a chip (SoC)](https://pl.wikipedia.org/wiki/System-on-a-chip) wraz ze zintegrowanym
układem WiFi 2,4 GHz (QCA9558 3x3 b/g/n):

![]({{< baseurl >}}/img/2016/05/17.tp-link-archer-c7-v2-podzespoly-wifi-2ghz.jpg#big)

Czip gigabitowego switch'a (AR8327N-BL1A):

![]({{< baseurl >}}/img/2016/05/18.tp-link-archer-c7-v2-podzespoly-gigabitowy-switch.jpg#big)

Pamięć operacyjna RAM 2 x 64 MiB. Ciekawa sprawa, bo na wiki OpenWRT można znaleźć informacje, że
ten model powinien zawierać kości od producenta Winbond. Niemniej jednak, na fotce widać wyraźnie,
że układy są od firmy Zentel:

![]({{< baseurl >}}/img/2016/05/19.tp-link-archer-c7-v2-podzespoly-pamiec-ram.jpg#big)

Porty od konsoli szeregowej:

![]({{< baseurl >}}/img/2016/05/20.tp-link-archer-c7-v2-konsola-szeregowa.jpg#big)

## Wsparcie dla routera TP-LINK Archer C7 v2 w OpenWRT

Router Archer C7 v2 jest bez większych problemów obsługiwany przez firmware OpenWRT od wydania Chaos
Calmer. W Barrier Breaker i wcześniejszych wersjach były problemy z przełącznikiem WiFi. Były też
problemy z diodami i przyciskiem reset. Wszystkie te niedogodności zostały poprawione w najnowszym
wydaniu.

Poniżej znajduje się rozpiska informacji uzyskanych z poziomu firmware OpenWRT.

Taktowanie:

    # dmesg | grep -i clocks
    [    0.000000] Clocks: CPU:720.000MHz, DDR:600.000MHz, AHB:200.000MHz, Ref:40.000MHz

Procesor:

    # cat /proc/cpuinfo
    system type             : Qualcomm Atheros QCA9558 ver 1 rev 0
    machine                 : TP-LINK Archer C7
    processor               : 0
    cpu model               : MIPS 74Kc V5.0
    BogoMIPS                : 358.80
    wait instruction        : yes
    microsecond timers      : yes
    tlb_entries             : 32
    extra interrupt vector  : yes
    hardware watchpoint     : yes, count: 4, address/irw mask: [0x0ffc, 0x0ffc, 0x0ffb, 0x0ffb]
    isa                     : mips1 mips2 mips32r1 mips32r2
    ASEs implemented        : mips16 dsp dsp2
    shadow register sets    : 1
    kscratch registers      : 0
    package                 : 0
    core                    : 0
    VCED exceptions         : not available
    VCEI exceptions         : not available

Porty USB:

    # cat /sys/kernel/debug/usb/devices

    T:  Bus=02 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=480  MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 2.00 Cls=09(hub  ) Sub=00 Prot=01 MxPS=64 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0002 Rev= 3.18
    S:  Manufacturer=Linux 3.18.29 ehci_hcd
    S:  Product=EHCI Host Controller
    S:  SerialNumber=ehci-platform.1
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

    T:  Bus=01 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=480  MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 2.00 Cls=09(hub  ) Sub=00 Prot=01 MxPS=64 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0002 Rev= 3.18
    S:  Manufacturer=Linux 3.18.29 ehci_hcd
    S:  Product=EHCI Host Controller
    S:  SerialNumber=ehci-platform.0
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

Pamięć operacyjna RAM (128 MiB):

    # dmesg | grep Memory
    [    0.000000] Memory: 125788K/131072K available (2607K kernel code, 127K rwdata, 544K rodata, 232K init, 193K bss, 5284K reserved)

Czipy bezprzewodowe WiFi:

    # iw list
    Wiphy phy1
            max # scan SSIDs: 4
            max scan IEs length: 2257 bytes
            max # sched scan SSIDs: 0
            max # match sets: 0
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports AP-side u-APSD.
            Device supports T-DLS.
            Available Antennas: TX 0x7 RX 0x7
            Configured Antennas: TX 0x7 RX 0x7
            Supported interface modes:
                     * IBSS
                     * managed
                     * AP
                     * AP/VLAN
                     * WDS
                     * monitor
                     * mesh point
                     * P2P-client
                     * P2P-GO
                     * outside context of a BSS
            Band 1:
                    Capabilities: 0x11ef
                            RX LDPC
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            TX STBC
                            RX STBC 1-stream
                            Max AMSDU length: 3839 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-23
                    Frequencies:
                            * 2412 MHz [1] (20.0 dBm)
                            * 2417 MHz [2] (20.0 dBm)
                            * 2422 MHz [3] (20.0 dBm)
                            * 2427 MHz [4] (20.0 dBm)
                            * 2432 MHz [5] (20.0 dBm)
                            * 2437 MHz [6] (20.0 dBm)
                            * 2442 MHz [7] (20.0 dBm)
                            * 2447 MHz [8] (20.0 dBm)
                            * 2452 MHz [9] (20.0 dBm)
                            * 2457 MHz [10] (20.0 dBm)
                            * 2462 MHz [11] (20.0 dBm)
                            * 2467 MHz [12] (20.0 dBm)
                            * 2472 MHz [13] (20.0 dBm)
                            * 2484 MHz [14] (disabled)
            valid interface combinations:
                     * #{ managed } <= 2048, #{ AP, mesh point } <= 8, #{ P2P-client, P2P-GO } <= 1, #{ IBSS } <= 1,
                       total <= 2048, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz }

                     * #{ WDS } <= 2048,
                       total <= 2048, #channels <= 1, STA/AP BI must match
            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing

    Wiphy phy0
            max # scan SSIDs: 16
            max scan IEs length: 199 bytes
            max # sched scan SSIDs: 0
            max # match sets: 0
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports AP-side u-APSD.
            Available Antennas: TX 0x7 RX 0x7
            Configured Antennas: TX 0x7 RX 0x7
            Supported interface modes:
                     * managed
                     * AP
                     * AP/VLAN
                     * monitor
                     * mesh point
            Band 2:
                    Capabilities: 0x19ef
                            RX LDPC
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            TX STBC
                            RX STBC 1-stream
                            Max AMSDU length: 7935 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-23
                    VHT Capabilities (0x338001b2):
                            Max MPDU length: 11454
                            Supported Channel Width: neither 160 nor 80+80
                            RX LDPC
                            short GI (80 MHz)
                            TX STBC
                            RX antenna pattern consistency
                            TX antenna pattern consistency
                    VHT RX MCS set:
                            1 streams: MCS 0-9
                            2 streams: MCS 0-9
                            3 streams: MCS 0-9
                            4 streams: not supported
                            5 streams: not supported
                            6 streams: not supported
                            7 streams: not supported
                            8 streams: not supported
                    VHT RX highest supported: 0 Mbps
                    VHT TX MCS set:
                            1 streams: MCS 0-9
                            2 streams: MCS 0-9
                            3 streams: MCS 0-9
                            4 streams: not supported
                            5 streams: not supported
                            6 streams: not supported
                            7 streams: not supported
                            8 streams: not supported
                    VHT TX highest supported: 0 Mbps
                    Frequencies:
                            * 5180 MHz [36] (20.0 dBm)
                            * 5200 MHz [40] (20.0 dBm)
                            * 5220 MHz [44] (20.0 dBm)
                            * 5240 MHz [48] (20.0 dBm)
                            * 5260 MHz [52] (20.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5280 MHz [56] (20.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5300 MHz [60] (20.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5320 MHz [64] (20.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5500 MHz [100] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5520 MHz [104] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5540 MHz [108] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5560 MHz [112] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5580 MHz [116] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5600 MHz [120] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5620 MHz [124] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5640 MHz [128] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5660 MHz [132] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5680 MHz [136] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5700 MHz [140] (27.0 dBm) (radar detection)
                              DFS state: usable (for 90770 sec)
                              DFS CAC time: 60000 ms
                            * 5720 MHz [144] (disabled)
                            * 5745 MHz [149] (disabled)
                            * 5765 MHz [153] (disabled)
                            * 5785 MHz [157] (disabled)
                            * 5805 MHz [161] (disabled)
                            * 5825 MHz [165] (disabled)
            valid interface combinations:
                     * #{ AP, mesh point } <= 8, #{ managed } <= 1,
                       total <= 8, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz, 80 MHz }

            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing
            Device supports VHT-IBSS.

Konfiguracja switch'a:

    # swconfig dev switch0 show
    Global attributes:
            enable_vlan: 1
            enable_mirror_rx: 0
            enable_mirror_tx: 0
            mirror_monitor_port: 0
            mirror_source_port: 0
            arl_table: address resolution table
    Port 0: MAC c4:6e:1f:95:ef:fe
    Port 4: MAC e8:94:f6:68:79:f1
    Port 6: MAC c4:6e:1f:95:ef:ff

    Port 0:
            mib: Port 0 MIB counters
    ...
            enable_eee: ???
            pvid: 1
            link: port:0 link:up speed:1000baseT full-duplex txflow rxflow
    Port 1:
            mib: Port 1 MIB counters
    ...
            enable_eee: 0
            pvid: 2
            link: port:1 link:down
    Port 2:
            mib: Port 2 MIB counters
    ...
            enable_eee: 0
            pvid: 1
            link: port:2 link:down
    Port 3:
            mib: Port 3 MIB counters
    ...
            enable_eee: 0
            pvid: 1
            link: port:3 link:down
    Port 4:
            mib: Port 4 MIB counters
    ...
            enable_eee: 0
            pvid: 1
            link: port:4 link:up speed:1000baseT full-duplex txflow rxflow eee100 eee1000 auto
    Port 5:
            mib: Port 5 MIB counters
    ...
            enable_eee: 0
            pvid: 1
            link: port:5 link:down
    Port 6:
            mib: Port 6 MIB counters
    ...
            enable_eee: ???
            pvid: 2
            link: port:6 link:up speed:1000baseT full-duplex txflow rxflow
    VLAN 1:
            vid: 1
            ports: 0 2 3 4 5
    VLAN 2:
            vid: 2
            ports: 1 6

Po zainstalowaniu świeżego firmware OpenWRT, do dyspozycji mamy nieco ponad 12 MiB wolnego miejsca
na flashu. Czyli dość sporo.

## Podsumowanie

Router Archer C7 v2 od TP-LINK'a z racji zastosowanych podzespołów firmy Qualcomm jest bardzo dobrze
wspierany przez alternatywne oprogramowanie bazujące na dystrybucjach linux'a, min. OpenWRT. Jest
idealny jeśli chodzi o możliwości rozbudowy pod względem programowym na alternatywnym firmware. Mamy
do dyspozycji dużo pamięci operacyjnej, no i też sporej wielkości flash, który jest w stanie
pomieścić masę dodatkowego oprogramowania. Oczywiście, nic nie stoi na przeszkodzie, by używać ten
router z oryginalnym firmware producenta. Radio 5 GHz w połączeniu z zewnętrznymi antenami sprawia,
że router znakomicie sprawuje się w warunkach miejskich, gdzie mamy duże zagęszczenie sieci
bezprzewodowych. W domkach jednorodzinnych będzie radził sobie nieco gorzej ze względu na brak
zewnętrznych anten i zwykle grubsze ściany. Ja osobiście jestem bardzo zadowolony z pracy tego
routera. Działa u mnie już prawie dwa lata bez zarzutu, oczywiście na firmware OpenWRT.
