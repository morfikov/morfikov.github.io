---
author: Morfik
categories:
- Linux
date: "2015-06-23T11:45:07Z"
date_gmt: 2015-06-23 09:45:07 +0200
published: true
status: publish
tags:
- dkms
- moduły-kernela
- wifi
- usb
title: Sterowniki do karty TP-LINK Archer T4U (8812au)
---

Póki co, w kernelu linux'a (4.5) nie ma odpowiednich sterowników do [adaptera WiFi Archer
T4U](http://www.tp-link.com.pl/products/details/cat-11_Archer-T4U.html) i trzeba je sobie
skompilować ręcznie. Trochę to dziwne, bo przecie kod sterownika jest na licencji GPLv2 i dostępny
już szmat czasu na [github'ie](https://github.com/abperiasamy/rtl8812AU_8821AU_linux). W każdym
razie, jeśli zakupiliśmy w/w kartę i nie jest ona wykrywana po wsadzeniu jej do portu USB, to czeka
nas proces kompilacji modułu `8812au` i jego automatyzacja przy pomocy [mechanizmu
DKMS]({{< baseurl >}}/post/dkms-czyli-automatycznie-budowane-moduly/).

<!--more-->
## Kompilacja modułu 8812au dla Archer T4U

Klonujemy zatem repozytorium github'a, na którym jest kod źródłowy modułu `8812au` :

    $ git clone https://github.com/abperiasamy/rtl8812AU_8821AU_linux.git

Przenosimy tak utworzony folder do katalogu `/usr/src/` zmieniając jednocześnie jego nazwę na
`rtl8812AU_8821AU_linux-1.0` :

    # mv rtl8812AU_8821AU_linux/ /usr/src/rtl8812AU_8821AU_linux-1.0/

W pliku `dkms.conf` mamy już przygotowaną odpowiednią konfigurację dla DKMS. Także by zainstalować
sterowniki do karty wifi Archer T4U, wydajemy to poniższe polecenie:

    # dkms install -m rtl8812AU_8821AU_linux -v 1.0

## Testy modułu 8812au

Budowa modułu `8812au` potrwa chwilę. Po tym jak proces dobiegnie końca, karta powinna zostać
wykryta przez system po jej podłączeniu do portu USB:

    kernel: usb 2-1.3: new high-speed USB device number 18 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0101
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: 802.11n NIC
    kernel: usb 2-1.3: Manufacturer: Realtek
    kernel: usb 2-1.3: SerialNumber: 123456
    mtp-probe[111475]: checking bus 2, device 18: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[111475]: bus: 2, device: 18 was not an MTP device
    kernel: usbcore: registered new interface driver rtl8812au
    systemd[1]: Starting Load/Save RF Kill Switch Status of rfkill4...
    systemd-networkd[1188]: wlan1: Renamed to wlan2

Jak widzimy, został również utworzony interfejs `wlan2` . Po [konfiguracji tego interfejsu
bezprzewodowego]({{< baseurl >}}/post/konfiguracja-polaczenia-wifi-pod-debianem/), karta powinna
nawiązać połączenie z punktem dostępowym WiFi:

    systemd[1]: Started WPA supplicant (wlan2).
    kernel: RTL871X: set ssid [Valar_Morghulis] fw_state=0x00000008
    kernel: RTL871X: start auth
    kernel: RTL871X: auth success, start assoc
    kernel: RTL871X: assoc success
    kernel: IPv6: ADDRCONF(NETDEV_CHANGE): wlan2: link becomes ready
    kernel: cfg80211: Calling CRDA for country: PL
    kernel: UpdateHalRAMask8812A => mac_id:0, networkType:0x44, mask:0xfffffff0
                                                 ==> rssi_level:0, rate_bitmap:0xfffff010
    systemd-networkd[1188]: wlan2: Gained carrier
    kernel: cfg80211: Regulatory domain changed to country: PL
    kernel: cfg80211:  DFS Master region: ETSI
    kernel: cfg80211:   (start_freq - end_freq @ bandwidth), (max_antenna_gain, max_eirp), (dfs_cac_time)
    kernel: cfg80211:   (2402000 KHz - 2482000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)
    kernel: cfg80211:   (5170000 KHz - 5250000 KHz @ 80000 KHz, 160000 KHz AUTO), (N/A, 2000 mBm), (N/A)
    kernel: cfg80211:   (5250000 KHz - 5330000 KHz @ 80000 KHz, 160000 KHz AUTO), (N/A, 2000 mBm), (0 s)
    kernel: cfg80211:   (5490000 KHz - 5710000 KHz @ 160000 KHz), (N/A, 2700 mBm), (0 s)
    kernel: cfg80211:   (57000000 KHz - 66000000 KHz @ 2160000 KHz), (N/A, 4000 mBm), (N/A)
    kernel: RTL871X: set pairwise key to hw: alg:4(WEP40-1 WEP104-5 TKIP-2 AES-4) camid:4
    kernel: UpdateHalRAMask8812A => mac_id:0, networkType:0x44, mask:0xfffffff0
                                                 ==> rssi_level:1, rate_bitmap:0xfe3f8000
    systemd-networkd[1188]: wlan2: DHCPv4 address 192.168.1.133/24 via 192.168.1.1
    systemd-networkd[1188]: wlan2: Configured

Klient został powiązany, uwierzytelnił się i otrzymał od serwera DHCP adres IP. Zatem połączenie
zostało skonfigurowane prawidłowo.

## Informacje o module 8812au

Poniżej są informacje na temat samego modułu `8812au` . Wszystkie wylistowane niżej opcje zostały
opisane we wpisie dotyczącym [parametrów
kernela]({{< baseurl >}}/post/modul-kernela-i-wartosci-jego-parametrow/) .

    # modinfo 8812au
    filename:       /lib/modules/4.5.0-2-amd64/updates/dkms/8812au.ko
    version:        v4.2.2_7502.20130517
    author: Morfik         Realtek Semiconductor Corp.
    description:    Realtek Wireless Lan Driver
    license:        GPL
    srcversion:     AC3D7329A53018EC3DC0D33
    alias:          usb:v2001p3318d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0411p0242d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0E66p0023d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3318d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v04BBp0953d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0846p9052d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3314d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v7392pA812d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDApA811d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v7392pA811d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v056Ep4007d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp0820d*dc*dsc*dp*icFFiscFFipFFin*
    alias:          usb:v0BDAp8822d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp0821d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp0811d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0411p025Dd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2357p0103d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v148Fp9097d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2357p0101d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v13B1p003Fd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v20F4p805Bd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3316d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3315d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v07B8p8812d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2019pAB30d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v1740p0100d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v1058p0632d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3313d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0586p3426d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0E66p0022d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0B05p17D2d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0409p0408d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0789p016Ed*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v04BBp0952d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0DF6p0074d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v7392pA822d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p330Ed*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v050Dp1109d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v050Dp1106d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp881Cd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp881Bd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp881Ad*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp8812d*dc*dsc*dp*ic*isc*ip*in*
    depends:        cfg80211,usbcore
    vermagic:       4.5.0-2-amd64 SMP mod_unload modversions
    parm:           rtw_ips_mode:The default IPS mode (int)
    parm:           rtw_regulatory_id:int
    parm:           ifname:The default name to allocate for first interface (charp)
    parm:           if2name:The default name to allocate for second interface (charp)
    parm:           rtw_initmac:charp
    parm:           rtw_channel_plan:int
    parm:           rtw_chip_version:int
    parm:           rtw_rfintfs:int
    parm:           rtw_lbkmode:int
    parm:           rtw_network_mode:int
    parm:           rtw_channel:int
    parm:           rtw_mp_mode:int
    parm:           rtw_wmm_enable:int
    parm:           rtw_vrtl_carrier_sense:int
    parm:           rtw_vcs_type:int
    parm:           rtw_busy_thresh:int
    parm:           rtw_ht_enable:int
    parm:           rtw_bw_mode:int
    parm:           rtw_ampdu_enable:int
    parm:           rtw_rx_stbc:int
    parm:           rtw_ampdu_amsdu:int
    parm:           rtw_vht_enable:int
    parm:           rtw_lowrate_two_xmit:int
    parm:           rtw_rf_config:int
    parm:           rtw_power_mgnt:int
    parm:           rtw_smart_ps:int
    parm:           rtw_low_power:int
    parm:           rtw_wifi_spec:int
    parm:           rtw_antdiv_cfg:int
    parm:           rtw_antdiv_type:int
    parm:           rtw_enusbss:int
    parm:           rtw_hwpdn_mode:int
    parm:           rtw_hwpwrp_detect:int
    parm:           rtw_hw_wps_pbc:int
    parm:           rtw_max_roaming_times:The max roaming times to try (uint)
    parm:           rtw_mc2u_disable:int
    parm:           rtw_80211d:Enable 802.11d mechanism (int)
    parm:           rtw_notch_filter:0:Disable, 1:Enable, 2:Enable only for P2P (uint)

Oraz informacje dotyczące standardowej konfiguracji:

    # systool -v -m 8812au
    Module = "8812au"

      Attributes:
        coresize            = "1015808"
        initsize            = "0"
        initstate           = "live"
        refcnt              = "0"
        srcversion          = "AC3D7329A53018EC3DC0D33"
        taint               = "OE"
        uevent              =
        version             = "v4.2.2_7502.20130517"

      Parameters:
        if2name             = "wlan%d"
        ifname              = "wlan%d"
        rtw_80211d          = "0"
        rtw_ampdu_amsdu     = "0"
        rtw_ampdu_enable    = "1"
        rtw_antdiv_cfg      = "2"
        rtw_antdiv_type     = "0"
        rtw_busy_thresh     = "40"
        rtw_bw_mode         = "33"
        rtw_channel_plan    = "66"
        rtw_channel         = "1"
        rtw_chip_version    = "0"
        rtw_enusbss         = "0"
        rtw_ht_enable       = "1"
        rtw_hw_wps_pbc      = "1"
        rtw_hwpdn_mode      = "2"
        rtw_hwpwrp_detect   = "0"
        rtw_initmac         = "(null)"
        rtw_ips_mode        = "0"
        rtw_lbkmode         = "0"
        rtw_low_power       = "0"
        rtw_lowrate_two_xmit= "1"
        rtw_max_roaming_times= "2"
        rtw_mc2u_disable    = "0"
        rtw_mp_mode         = "0"
        rtw_network_mode    = "0"
        rtw_notch_filter    = "0"
        rtw_power_mgnt      = "0"
        rtw_regulatory_id   = "255"
        rtw_rf_config       = "5"
        rtw_rfintfs         = "2"
        rtw_rx_stbc         = "1"
        rtw_smart_ps        = "2"
        rtw_vcs_type        = "1"
        rtw_vht_enable      = "1"
        rtw_vrtl_carrier_sense= "2"
        rtw_wifi_spec       = "0"
        rtw_wmm_enable      = "1"

      Sections:
        .bss                = "0xffffffffc0d2ef80"
        .data               = "0xffffffffc0cd2000"
        .exit.text          = "0xffffffffc0cc37ef"
        .gnu.linkonce.this_module= "0xffffffffc0d2ec40"
        .init.text          = "0xffffffffc0d51000"
        .note.gnu.build-id  = "0xffffffffc0cc4000"
        .parainstructions   = "0xffffffffc0cd1548"
        .rodata             = "0xffffffffc0cc4040"
        .rodata.str1.1      = "0xffffffffc0ccf810"
        .rodata.str1.8      = "0xffffffffc0cd0a28"
        .smp_locks          = "0xffffffffc0cd1500"
        .strtab             = "0xffffffffc0d61198"
        .symtab             = "0xffffffffc0d52000"
        .text               = "0xffffffffc0c58000"
        __bug_table         = "0xffffffffc0cd10ea"
        __mcount_loc        = "0xffffffffc0ccc240"
        __param             = "0xffffffffc0cd1568"
