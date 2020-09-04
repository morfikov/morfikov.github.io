---
author: Morfik
categories:
- Hardware
date: "2016-06-01T17:14:59Z"
date_gmt: 2016-06-01 15:14:59 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
title: 'Recenzja: karta WiFi TP-LINK Archer T4U'
---

Ilość punktów dostępowych w paśmie 2,4 GHz rośnie bardzo szybko. Obecnie spora część użytkowników
laptopów czy smartfonów zdolnych łączyć się bezprzewodowo korzysta z ruterów WiFi w swoim domowym
zaciszu. W przypadku zatłoczonych blokowisk i innych większych skupisk ludzkich, transfer w paśmie
2,4 GHz może nie być za duży. By rozwiązać ten problem, wprowadzono urządzenia nadające w paśmie 5
GHz. Są one nieco droższe, bo i technologia jest nieco nowsza. Niemnie jednak, pasmo 5 GHz idealnie
nadaje się do zastosowań miejskich. Chodzi generalnie o mniejszy zasięg punktu dostępowego, a co z
tym się wiąże, mniejszą ilością zakłóceń, które osłabiają transfer w sieci bezprzewodowej. Sporo
routerów WiFi ma już zaimplementowane dwa radia, po jednym dla każdego z w/w pasm i przydałoby się
powoli postarać o zakup karty/adaptera pracującego w standardzie AC. Tak się składa, że mam na
składzie dwuzakresowy adapter Archer T4U (czip Realtek RTL8812AU), który został mi sprezentowany
przez firmę TP-LINK. Postaram go zatem opisać w tym artykule.

<!--more-->
## Zawartość opakowania

Pudełko jest trochę duże ale adapter Archer T4U również do najmniejszych nie należy. Poniżej
znajduje się kilka fotek przedstawiających samo opakowanie jak i wszystkie jego elementy
składowe.

![]({{< baseurl >}}/img/2016/08/1.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/2.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/3.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/4.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/5.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/6.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/7.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/8.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/9.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

![]({{< baseurl >}}/img/2016/08/10.karta-adapter-TP-LINK-Archer-T4U-linux.jpg#huge)

Jak widzimy, port USB tej karty jest niebieski, czyli jest to standard USB 3.0 . Karta ma wbudowany
przycisk WPS, a zielona dioda sygnalizuje stan urządzenia (włączone, transfer danych). Do zestawu
jest także dołączony dobrej jakości kabel USB 3.0 (przedłużka). Zabrakło tej karcie jedynie
zewnętrznej anteny.

## Sterowniki do Archer T4U

W kernelu linux'a 4.5 nie ma póki co odpowiedniego modułu, który by zapewniał wsparcie dla adaptera
Archer T4U. Trzeba zatem ręcznie skompilować sobie moduł, by móc z tej karty korzystać po linux'em.
[Proces kompilacji modułu 8812au dla Archer
T4U]({{< baseurl >}}/post/sterowniki-karty-tp-link-archer-t4u-8812au/) został opisany w osobnym
artykule.

## Parametry adaptera Archer T4U

Zakładając, że mamy już skompilowany moduł, to po podłączeniu adaptera do portu USB powinien on
zostać rozpoznany przez system pod idVendor=`2357` oraz idProduct=`0101` :

    kernel: usb 2-1.3: new high-speed USB device number 10 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0101
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: 802.11n NIC
    kernel: usb 2-1.3: Manufacturer: Realtek
    kernel: usb 2-1.3: SerialNumber: 123456
    systemd-udevd[457]: Network interface NamePolicy= disabled on kernel command line, ignoring.
    mtp-probe[15674]: checking bus 2, device 10: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[15674]: bus: 2, device: 10 was not an MTP device
    systemd[1]: Starting Load/Save RF Kill Switch Status...
    kernel: usbcore: registered new interface driver rtl8812au
    kernel: rtl8812au 2-1.3:1.0 wlan2: renamed from wlan1
    systemd[1]: Started Load/Save RF Kill Switch Status.
    kernel: brcmsmac bcma0:1: wl0: brcms_c_d11hdrs_mac80211:  txop exceeded phylen 153/256 dur 1730/1504
    kernel: usb 2-1.3: USB disconnect, device number 10
    systemd[1]: Starting Load/Save RF Kill Switch Status...
    systemd[1]: Started Load/Save RF Kill Switch Status.

Poniżej jest trochę informacji na temat samego adaptera:

    # lsusb -v -d 2357:0101

    Bus 002 Device 013: ID 2357:0101
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x2357
      idProduct          0x0101
      bcdDevice            0.00
      iManufacturer           1 Realtek
      iProduct                2 802.11n NIC
      iSerial                 3 123456
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           53
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
          bNumEndpoints           5
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass    255 Vendor Specific Subclass
          bInterfaceProtocol    255 Vendor Specific Protocol
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x03  EP 3 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x04  EP 4 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x85  EP 5 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               1
    Device Qualifier (for other device speed):
      bLength                10
      bDescriptorType         6
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      bNumConfigurations      1
    Device Status:     0x0002
      (Bus Powered)
      Remote Wakeup Enabled

    # iw list
    Wiphy phy3
            max # scan SSIDs: 9
            max scan IEs length: 2304 bytes
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Supported Ciphers:
                    * WEP40 (00-0f-ac:1)
                    * WEP104 (00-0f-ac:5)
                    * TKIP (00-0f-ac:2)
                    * CCMP (00-0f-ac:4)
            Available Antennas: TX 0 RX 0
            Supported interface modes:
                     * IBSS
                     * managed
                     * AP
                     * monitor
                     * P2P-client
                     * P2P-GO
            Band 1:
                    Bitrates (non-HT):
                            * 1.0 Mbps
                            * 2.0 Mbps
                            * 5.5 Mbps
                            * 11.0 Mbps
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
            Band 2:
                    Bitrates (non-HT):
                            * 6.0 Mbps
                            * 9.0 Mbps
                            * 12.0 Mbps
                            * 18.0 Mbps
                            * 24.0 Mbps
                            * 36.0 Mbps
                            * 48.0 Mbps
                            * 54.0 Mbps
                    Frequencies:
                            * 5170 MHz [34] (disabled)
                            * 5180 MHz [36] (20.0 dBm)
                            * 5190 MHz [38] (20.0 dBm)
                            * 5200 MHz [40] (20.0 dBm)
                            * 5210 MHz [42] (20.0 dBm)
                            * 5220 MHz [44] (20.0 dBm)
                            * 5230 MHz [46] (20.0 dBm)
                            * 5240 MHz [48] (20.0 dBm)
                            * 5260 MHz [52] (20.0 dBm) (radar detection)
                              DFS state: usable (for 136 sec)
                              DFS CAC time: 60000 ms
                            * 5280 MHz [56] (20.0 dBm) (radar detection)
                              DFS state: usable (for 136 sec)
                              DFS CAC time: 60000 ms
                            * 5300 MHz [60] (20.0 dBm) (radar detection)
                              DFS state: usable (for 136 sec)
                              DFS CAC time: 60000 ms
                            * 5320 MHz [64] (20.0 dBm) (radar detection)
                              DFS state: usable (for 136 sec)
                              DFS CAC time: 60000 ms
                            * 5500 MHz [100] (disabled)
                            * 5520 MHz [104] (disabled)
                            * 5540 MHz [108] (disabled)
                            * 5560 MHz [112] (disabled)
                            * 5580 MHz [116] (disabled)
                            * 5600 MHz [120] (disabled)
                            * 5620 MHz [124] (disabled)
                            * 5640 MHz [128] (disabled)
                            * 5660 MHz [132] (disabled)
                            * 5680 MHz [136] (disabled)
                            * 5700 MHz [140] (disabled)
                            * 5745 MHz [149] (disabled)
                            * 5765 MHz [153] (disabled)
                            * 5785 MHz [157] (disabled)
                            * 5805 MHz [161] (disabled)
                            * 5825 MHz [165] (disabled)
                            * 5920 MHz [184] (disabled)
                            * 5940 MHz [188] (disabled)
                            * 5960 MHz [192] (disabled)
                            * 5980 MHz [196] (disabled)
                            * 6000 MHz [200] (disabled)
                            * 6020 MHz [204] (disabled)
                            * 6040 MHz [208] (disabled)
                            * 6060 MHz [212] (disabled)
                            * 6080 MHz [216] (disabled)
            Supported commands:
                     * new_interface
                     * set_interface
                     * new_key
                     * start_ap
                     * new_station
                     * set_bss
                     * join_ibss
                     * set_pmksa
                     * del_pmksa
                     * flush_pmksa
                     * remain_on_channel
                     * frame
                     * set_channel
                     * connect
                     * disconnect
            Supported TX frame types:
                     * IBSS: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * managed: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * AP: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * AP/VLAN: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * P2P-client: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
                     * P2P-GO: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
            Supported RX frame types:
                     * IBSS: 0xd0
                     * managed: 0x40 0xd0
                     * AP: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                     * AP/VLAN: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
                     * P2P-client: 0x40 0xd0
                     * P2P-GO: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
            software interface modes (can always be added):
                     * monitor
            interface combinations are not supported
            Device supports scan flush.

Wsparcie dla modułu `8812au` , który obsługuję tę kartę, jest średnie ale ten Archer T4U działa
raczej bez większych problemów. Jedyny mankament to [tryb oszczędzania
energii]({{< baseurl >}}/post/tryb-oszczedzania-energii-w-kartach-wifi/), który jest
implementowany masowo w kartach WiFi. Jeśli obserwujemy wysoki `ping` w stanie spoczynku karty,
oznacza to, że ten tryb powinien zostać wyłączony.

## Konfiguracja karty Archer T4U na linux'ie

Generalnie rzecz biorąc, jeśli udało nam się skompilować moduł, to karta powinna działać po podaniu
jej odpowiednich parametrów sieci WiFi. W różnych dystrybucjach linux'a używa się innych narzędzi do
konfiguracji połączenia bezprzewodowego. [Proces konfiguracji połączenia WiFi pod
debianem]({{< baseurl >}}/post/konfiguracja-polaczenia-wifi-pod-debianem/) został opisany w
osobnym artykule. W tym podlinkowanym wpisie zostało to pokazane na przykładzie narzędzi zawartych w
pakietach `wpasupplicant` oraz `ifupdown` .

## Podsumowanie

Jako, że moduł do adaptera Archer T4U nie znajduje się w kernelu, to radziłbym unikać tej karty w
przypadku, gdy zamierzamy używać jej głównie pod linux'em. Działa ona po samodzielnym skompilowaniu
modułu, z tym, żę obsługa tego adaptera nie jest bezproblemowa. Weźmy przykładowo skanowanie pasma w
poszukiwaniu innych sieci, które są w naszym zasięgu. Pod debianem mamy do tego celu kilka narzędzi,
np. `wpa_cli` , `wavemon` i `linssid` . Poniżej są wyniki zwracane przez te aplikacje:

![]({{< baseurl >}}/img/2016/06/3.tp-link-archer-t4u-adapter-wifi-wavemon.png#huge)

![]({{< baseurl >}}/img/2016/06/4.tp-link-archer-t4u-adapter-wifi-linssid.png#huge)

![]({{< baseurl >}}/img/2016/06/5.tp-link-archer-t4u-adapter-wifi-wpa-cli.png#huge)

Widzimy wyraźnie, że nie ma co się opierać na tych wynikach. System zwyczajnie nie jest w stanie
oszacować jakości sygnału. Ta z kolei jest stała i nie ulega zmianie. Jak dla mnie, to za wcześnie
jest na tę kartę i lepiej dobrać jakąś sprawdzoną konstrukcję opartą o czipy firmy QUALCOMM ATHEROS,
bo te akurat mają odpowiednie moduły w kernelu i działają bez zarzutu po linux'em.
