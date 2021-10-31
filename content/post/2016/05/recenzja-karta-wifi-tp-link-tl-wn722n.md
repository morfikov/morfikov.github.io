---
author: Morfik
categories:
- Hardware
date: "2016-05-31T17:22:40Z"
date_gmt: 2016-05-31 15:22:40 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
GHissueID: 475
title: 'Recenzja: karta WiFi TP-LINK TL-WN722N'
---

Karty WiFi implementowane w różnego rodzaju laptopach zwykle są dość przyzwoite. Przynajmniej do
czasu aż użytkownik zapragnie korzystać z alternatywnego systemu operacyjnego z rodziny linux. W
takim przypadku często okazuje się, że większość kart bezprzewodowych pracuje na układach, których
sterowniki nie są wolne i nie ma ich dostępnych w linux'owym kernelu. Czasem takie karty można
zmusić do pracy ale uzyskana w ten sposób prędkość transferu zbytnio nie zachwyca. Jedynym
przyzwoitym rozwiązaniem jest zakup zewnętrznego adaptera WiFi, który pracuje na czipie wspieranym
przez kernel. Zwykle są to karty na układach od firmy [QUALCOMM
ATHEROS](https://en.wikipedia.org/wiki/Qualcomm_Atheros). W tym artykule weźmiemy sobie pod lupę
adapter [TL-WN722N](http://www.tp-link.com.pl/products/details/TL-WN722N.html) od firmy TP-LINK,
który działa w standardzie N (pasmo 2,4 GHz, transfer do 150 mbit/s).

<!--more-->
## Wygląd adaptera TP-LINK TL-WN722N

Adapter TL-WN722N jest wyposażony w port USB 2.0 i posiada również przycisk WPS do uzyskiwania
automatycznej konfiguracji połączenia bezprzewodowego. Pod linux'em jednak, z jakichś nieznanych mi
powodów, ten przycisk WPS nie chce zbytnio działać. Karta ma doczepianą antenkę 4
[dBi](https://pl.wikipedia.org/wiki/DBi). W zestawie jest także dołączony kabelek USB. Teoretycznie
ta karta jest w stanie nadawać z prędkością 150 mbit/s. Poniżej jest fotka prezentująca jej wygląd
zewnętrzny:

![tp-link-tl-wn722n-wyglad-adapter-wifi](/img/2016/05/1.tp-link-tl-wn722n-wyglad-adapter-wifi.jpg#big)

Zobaczmy co ten adapter skrywa w sobie:

![tp-link-tl-wn722n-adapter-wifi](/img/2016/05/2.tp-link-tl-wn722n-adapter-wifi.jpg#big)

Niżej jest wyciągnięty cały układ z obudowy:

![tp-link-tl-wn722n-adapter-wifi-uklad](/img/2016/05/4.tp-link-tl-wn722n-adapter-wifi-uklad.jpg#big)

Jeszcze jego spód:

![tp-link-tl-wn722n-adapter-wifi-uklad](/img/2016/05/5.tp-link-tl-wn722n-adapter-wifi-uklad.jpg#big)

Z jednej ze stron układu jest wyprowadzone gniazdo antenowe:

![tp-link-tl-wn722n-adapter-wifi-gniazdo-antena](/img/2016/05/6.tp-link-tl-wn722n-adapter-wifi-gniazdo-antena.jpg#big)

Z drugiej zaś przycisk WPS:

![tp-link-tl-wn722n-adapter-wifi-przycisk-wps](/img/2016/05/7.tp-link-tl-wn722n-adapter-wifi-przycisk-wps.jpg#big)

Karta TL-WN722N posiada także zieloną diodę, sygnalizującą stan pracy urządzenia (włączony, transfer
danych):

![tp-link-tl-wn722n-adapter-wifi-podlaczony](/img/2016/05/8.tp-link-tl-wn722n-adapter-wifi-podlaczony.jpg#big)

![tp-link-tl-wn722n-adapter-wifi-test](/img/2016/05/9.tp-link-tl-wn722n-adapter-wifi-test.jpg#big)

Poniżej zaś jest fotka anteny dołączanej do zestawu (ta mniejsza). Ta większa zaś pochodzi od
[routera TP-LINK TL-WR1043ND
v2](/post/recenzja-sprzetu-router-tp-link-tl-wr1043nd-v2/). Porównane są one
jedynie w celach orientacyjnych.

![tp-link-tl-wn722n-adapter-wifi-antena](/img/2016/05/10.tp-link-tl-wn722n-adapter-wifi-antena.jpg#big)

![tp-link-tl-wn722n-adapter-wifi-antena](/img/2016/05/11.tp-link-tl-wn722n-adapter-wifi-antena.jpg#big)

Mimo znacznej różnicy w rozmiarze, gniazda anten są takie same i nic nie stoi na przeszkodzie, by
korzystać z tej większej anteny, oczywiście jeśli posiadamy takową na zbyciu.

![tp-link-tl-wn722n-adapter-wifi-antena](/img/2016/05/12.tp-link-tl-wn722n-adapter-wifi-antena.jpg#big)

## Instalacja adaptera TL-WN722N w systemie linux (debian)

Zgodnie z informacją dostępną [tutaj](https://wikidevi.com/wiki/TP-LINK_TL-WN722N), adapter
TL-WN722N ma czip Atheros AR9271, który jest obsługiwany przez moduł `ath9k_htc` . Ten moduł jest
dostępny w kernelu ale, by ta karta nam działała pod linux'em musimy zainstalować jeszcze firmware.
Ten z kolei dostępny jest w debianie w pakiecie `firmware-atheros` . Jeśli nie doinstalowalibyśmy
tego firmware i próbowali odpalić ten adapter, w logu zobaczymy poniższy komunikat:

    kernel: [18738.288728] usb 2-1.1: new high-speed USB device number 13 using ehci-pci
    kernel: [18738.397468] usb 2-1.1: New USB device found, idVendor=0cf3, idProduct=9271
    kernel: [18738.397479] usb 2-1.1: New USB device strings: Mfr=16, Product=32, SerialNumber=48
    kernel: [18738.397485] usb 2-1.1: Product: USB2.0 WLAN
    kernel: [18738.397489] usb 2-1.1: Manufacturer: ATHEROS
    kernel: [18738.397493] usb 2-1.1: SerialNumber: 12345
    kernel: [18738.398035] usb 2-1.1: ath9k_htc: Firmware htc_9271.fw requested
    kernel: [18738.398319] usb 2-1.1: firmware: failed to load htc_9271.fw (-2)
    kernel: [18738.398324] usb 2-1.1: Direct firmware load failed with error -2
    kernel: [18738.398328] usb 2-1.1: Falling back to user helper
    kernel: [18738.428503] usb 2-1.1: ath9k_htc: USB layer deinitialized

Po instalacji firmware, karta powinna zostać wykryta przez system bez większego problemu po ponownym
podłączeniu do portu USB:

    kernel: usb 2-1.3: new high-speed USB device number 7 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=0cf3, idProduct=9271
    kernel: usb 2-1.3: New USB device strings: Mfr=16, Product=32, SerialNumber=48
    kernel: usb 2-1.3: Product: USB2.0 WLAN
    kernel: usb 2-1.3: Manufacturer: ATHEROS
    kernel: usb 2-1.3: SerialNumber: 12345
    kernel: usb 2-1.3: ath9k_htc: Firmware ath9k_htc/htc_9271-1.4.0.fw requested
    kernel: usbcore: registered new interface driver ath9k_htc
    kernel: usb 2-1.3: firmware: direct-loading firmware ath9k_htc/htc_9271-1.4.0.fw
    kernel: usb 2-1.3: ath9k_htc: Transferred FW: ath9k_htc/htc_9271-1.4.0.fw, size: 51008
    kernel: ath9k_htc 2-1.3:1.0: ath9k_htc: HTC initialized with 33 credits
    kernel: ath9k_htc 2-1.3:1.0: ath9k_htc: FW Version: 1.4
    kernel: ath9k_htc 2-1.3:1.0: FW RMW support: On
    kernel: ath: EEPROM regdomain: 0x809c
    kernel: ath: EEPROM indicates we should expect a country code
    kernel: ath: doing EEPROM country->regdmn map search
    kernel: ath: country maps to regdmn code: 0x52
    kernel: ath: Country alpha2 being used: CN
    kernel: ath: Regpair used: 0x52
    kernel: ieee80211 phy1: Atheros AR9271 Rev:1
    systemd[1]: Starting Load/Save RF Kill Switch Status...
    systemd[1]: Started Load/Save RF Kill Switch Status.

Ten adapter jest wykrywany jako idVendor=`0cf3` oraz idProduct=`9271` . Poniżej znajduje się wyjście
kilku poleceń, które pokażą charakterystykę karty TL-WN722N:

    # lsusb -v -d 0cf3:9271

    Bus 002 Device 008: ID 0cf3:9271 Atheros Communications, Inc. AR9271 802.11n
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass          255 Vendor Specific Class
      bDeviceSubClass       255 Vendor Specific Subclass
      bDeviceProtocol       255 Vendor Specific Protocol
      bMaxPacketSize0        64
      idVendor           0x0cf3 Atheros Communications, Inc.
      idProduct          0x9271 AR9271 802.11n
      bcdDevice            1.08
      iManufacturer          16 ATHEROS
      iProduct               32 USB2.0 WLAN
      iSerial                48 12345
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           60
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0x80
          (Bus Powered)
        MaxPower              500mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           6
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      0
          bInterfaceProtocol      0
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x01  EP 1 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x04  EP 4 OUT
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x05  EP 5 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x06  EP 6 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
    Device Qualifier (for other device speed):
      bLength                10
      bDescriptorType         6
      bcdUSB               2.00
      bDeviceClass          255 Vendor Specific Class
      bDeviceSubClass       255 Vendor Specific Subclass
      bDeviceProtocol       255 Vendor Specific Protocol
      bMaxPacketSize0        64
      bNumConfigurations      1
    Device Status:     0x0000
      (Bus Powered)

    # iw list
    Wiphy phy2
            max # scan SSIDs: 4
            max scan IEs length: 2257 bytes
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports RSN-IBSS.
            Device supports T-DLS.
            Supported Ciphers:
                    * WEP40 (00-0f-ac:1)
                    * WEP104 (00-0f-ac:5)
                    * TKIP (00-0f-ac:2)
                    * CCMP (00-0f-ac:4)
                    * 00-0f-ac:10
                    * GCMP (00-0f-ac:8)
                    * 00-0f-ac:9
                    * CMAC (00-0f-ac:6)
                    * 00-0f-ac:13
                    * 00-0f-ac:11
                    * 00-0f-ac:12
            Available Antennas: TX 0x1 RX 0x1
            Configured Antennas: TX 0x1 RX 0x1
            Supported interface modes:
                     * IBSS
                     * managed
                     * AP
                     * AP/VLAN
                     * monitor
                     * mesh point
                     * P2P-client
                     * P2P-GO
                     * Unknown mode (11)
            Band 1:
                    Capabilities: 0x116e
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            RX STBC 1-stream
                            Max AMSDU length: 3839 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-7
                    Bitrates (non-HT):
                            * 1.0 Mbps
                            * 2.0 Mbps (short preamble supported)
                            * 5.5 Mbps (short preamble supported)
                            * 11.0 Mbps (short preamble supported)
                            * 6.0 Mbps
                            * 9.0 Mbps
                            * 12.0 Mbps
                            * 18.0 Mbps
                            * 24.0 Mbps
                            * 36.0 Mbps
                            * 48.0 Mbps
                            * 54.0 Mbps
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
            Supported commands:
                     * new_interface
                     * set_interface
                     * new_key
                     * start_ap
                     * new_station
                     * new_mpath
                     * set_mesh_config
                     * set_bss
                     * authenticate
                     * associate
                     * deauthenticate
                     * disassociate
                     * join_ibss
                     * join_mesh
                     * remain_on_channel
                     * set_tx_bitrate_mask
                     * frame
                     * frame_wait_cancel
                     * set_wiphy_netns
                     * set_channel
                     * set_wds_peer
                     * tdls_mgmt
                     * tdls_oper
                     * probe_client
                     * set_noack_map
                     * register_beacons
                     * start_p2p_device
                     * set_mcast_rate
                     * channel_switch
                     * Unknown command (104)
                     * connect
                     * disconnect
            Supported TX frame types:
                     * IBSS: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * managed: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * AP: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * AP/VLAN: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * mesh point: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * P2P-client: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * P2P-GO: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * P2P-device: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
            Supported RX frame types:
                     * IBSS: 0x40 0xb0 0xc0 0xd0
                     * managed: 0x40 0xd0
                     * AP: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                     * AP/VLAN: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                     * mesh point: 0xb0 0xc0 0xd0
                     * P2P-client: 0x40 0xd0
                     * P2P-GO: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                     * P2P-device: 0x40 0xd0
            software interface modes (can always be added):
                     * AP/VLAN
                     * monitor
            valid interface combinations:
                     * #{ managed, P2P-client } <= 2, #{ AP, mesh point, P2P-GO } <= 2,
                       total <= 2, #channels <= 1
            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing
            Device supports TX status socket option.
            Device supports HT-IBSS.
            Device supports SAE with AUTHENTICATE command
            Device supports low priority scan.
            Device supports scan flush.
            Device supports AP scan.
            Device supports per-vif TX power setting
            Driver supports full state transitions for AP/GO clients
            Driver supports a userspace MPM

Warto odnotować, że karta posiada obsługę trybu AP. Niemniej jednak, zdolności tego punktu
dostępowego są dość ograniczone. Maksymalna ilość sieci jakie możemy stworzyć to 2. Natomiast ilość
klientów, które mogłyby się z tą kartą podłączyć to 7. Jeśli jednak chcielibyśmy się pokusić o
stworzenie takiego Access Point'a, to zachęcam do zaznajomienia się z artykułem dotyczącym
[instalacji i konfiguracji adaptera TL-WN722N w trybie AP na
linux'ie](/post/konfiguracja-trybu-ap-kart-wifi-na-debianie/) (dystrybucja debian).

Jeśli zaś chodzi o konfigurację tej karty w trybie klienta (STA), to z kolei odsyłam do artykułu
opisującego [konfigurację połączenia WiFi pod
debianem](/post/konfiguracja-polaczenia-wifi-pod-debianem/).

## Podsumowanie

Może i na tym adapterze TL-WN722N od TP-LINK'a jest napisane 150 mbit/s ale w mojej okolicy (centrum
miasta) nie udało się osiągnąć więcej niż 80 mbit/s. Poniżej jest test przeprowadzony za pomocą
narzędzia `iperf` .

Test dla szerokości kanałów 20 MHz:

    # iperf -i 10 -t 100 -c 192.168.1.1
    ------------------------------------------------------------
    Client connecting to 192.168.1.1, TCP port 5001
    TCP window size: 85.0 KByte (default)
    ------------------------------------------------------------
    [  3] local 192.168.1.160 port 52887 connected with 192.168.1.1 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  3]  0.0-10.0 sec  53.5 MBytes  44.9 Mbits/sec
    [  3] 10.0-20.0 sec  53.5 MBytes  44.9 Mbits/sec
    [  3] 20.0-30.0 sec  55.0 MBytes  46.1 Mbits/sec
    [  3] 30.0-40.0 sec  55.5 MBytes  46.6 Mbits/sec
    [  3] 40.0-50.0 sec  55.6 MBytes  46.7 Mbits/sec
    [  3] 50.0-60.0 sec  55.9 MBytes  46.9 Mbits/sec
    [  3] 60.0-70.0 sec  54.9 MBytes  46.0 Mbits/sec

Test dla szerokości kanałów 40 MHz:

    # iperf -i 10 -t 100 -c 192.168.1.1
    ------------------------------------------------------------
    Client connecting to 192.168.1.1, TCP port 5001
    TCP window size: 85.0 KByte (default)
    ------------------------------------------------------------
    [  3] local 192.168.1.160 port 52944 connected with 192.168.1.1 port 5001
    [ ID] Interval       Transfer     Bandwidth
    [  3]  0.0-10.0 sec  90.1 MBytes  75.6 Mbits/sec
    [  3] 10.0-20.0 sec  93.1 MBytes  78.1 Mbits/sec
    [  3] 20.0-30.0 sec  93.6 MBytes  78.5 Mbits/sec
    [  3] 30.0-40.0 sec  93.6 MBytes  78.5 Mbits/sec
    [  3] 40.0-50.0 sec  93.0 MBytes  78.0 Mbits/sec
    [  3] 50.0-60.0 sec  94.8 MBytes  79.5 Mbits/sec
    [  3] 60.0-70.0 sec  93.8 MBytes  78.6 Mbits/sec

Większej prędkości w moich warunkach nie da się raczej osiągnąć. Przy czym należy powiedzieć, że nie
jest znowu aż tak źle. Na tej karcie WiFi, która jest dołączana w standardzie do mojego laptopa
(jakiś Broadcom), wynik jest ponad trzykrotnie gorszy.
