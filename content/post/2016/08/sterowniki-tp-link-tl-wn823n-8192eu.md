---
author: Morfik
categories:
- Linux
date: "2016-08-12T20:50:39Z"
date_gmt: 2016-08-12 18:50:39 +0200
published: true
status: publish
tags:
- dkms
- moduły-kernela
- wifi
- usb
GHissueID: 567
title: Sterowniki do karty TP-LINK TL-WN823N (8192eu)
---

Systemy operacyjne nie są w stanie wejść w interakcję ze sprzętem, do którego nie posiadają
sterowników. Linux już od dość dawna żyje sobie wśród nas i coraz bardziej pcha się na desktopy.
Niemniej jednak producenci tych wszystkich urządzeń niechętnie wypuszczają sterowniki dla
alternatywnych systemów. Ostatnio próbowałem uruchomić [adapter TL-WN823N
V2](http://www.tp-link.com.pl/products/details/TL-WN823N.html) od firmy TP-LINK. Na opakowaniu
widnieje napis sugerujący, że ta karta działa pod linux'em. Rzeczywistość jednak okazała się
zupełnie inna. Mianowicie, mój Debian w ogóle nie rozpoznał tej karty. Jedyne informacje jakie mi
zwrócił to nazwę producenta czipu, którym okazał się być `Realtek` , oraz `idVendor=2357` i
`idProduct=0109` . [Sterowników dostępnych na stronie
TP-LINK'a](http://www.tp-link.com/en/download/TL-WN823N.html#Driver) nie szło zbudować na obecnym
kernelu 4.6 . Trzeba było zatem poszukać innej alternatywy. Na szczęście udało się znaleźć moduł
8192eu (rtl8192eu), który się skompilował i zainstalował bez problemu. Karta TL-WN823N V2 została
wykryta i działa. W tym wpisie zostanie pokazany proces kompilacji tego modułu.

<!--more-->
## Kompilacja modułu 8192eu dla TL-WN823N V2

Kod źródłowy sterownika dla karty TL-WN823N V2 znajduje się na github'ie. Pozyskać go możemy
odwiedzając [ten link](https://github.com/Mange/rtl8192eu-linux-driver) lub też wykorzystać do tego
celu narzędzie `git` . Który ze sposobów wybierzemy jest kompletnie bez znaczenia. W tym przypadku
korzystamy z git'a:

    $ git clone https://github.com/Mange/rtl8192eu-linux-driver

Moduł możemy zbudować ręcznie za pomocą poleceń `make` i `make install` ale nie będziemy tego robić
ze względów "estetycznych". Chodzi generalnie o to, że w przypadku nowszych wersji kernela, trzeba
będzie ten moduł ponownie ręcznie budować. Zamiast tak sobie życie utrudniać, zaprzęgniemy
mechanizm DKMS.

Po wydaniu powyższego polecenia, w katalogu roboczym został utworzony folder
`rtl8192eu-linux-driver` . Kopiujemy go do katalogu `/usr/src/` i zmieniamy jego nazwę na
`8192eu-1.0` :

    # mv rtl8192eu-linux-driver /usr/src/8192eu-1.0

By mechanizm DKMS zadziałał, potrzebna jest mu krótka instrukcja. W katalogu `/usr/src/8192eu-1.0/`
musimy zatem utworzyć plik `dkms.conf` . Do niego zaś dodajemy poniższą treść:

    PACKAGE_NAME="8192eu"
    PACKAGE_VERSION="1.0"
    BUILT_MODULE_NAME="8192eu"
    DEST_MODULE_LOCATION="/kernel/drivers/net/wireless/"
    REMAKE_INITRD="yes"
    AUTOINSTALL="yes"
    MAKE="'make' all"
    CLEAN="'make' clean"

Teraz w terminalu wydajemy to poniższe polecenie:

    # dkms install -m 8192eu -v 1.0
    ...
    DKMS: install completed.

## Informacje o module 8192eu

Moduł powinien się zbudować bez większego problemu. Sprawdźmy czy jesteśmy w stanie uzyskać jakieś
informacje na jego temat:

    # modinfo 8192eu
    filename:       /lib/modules/4.6.0-1-amd64/updates/dkms/8192eu.ko
    version:        v4.3.1.1_11320.20140505
    author: Morfik         Realtek Semiconductor Corp.
    description:    Realtek Wireless Lan Driver
    license:        GPL
    srcversion:     DC4944779709F5666CC49A7
    alias:          usb:v2357p0109d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2357p0108d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2357p0107d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3319d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0BDAp818Cd*dc*dsc*dp*icFFiscFFipFFin*
    alias:          usb:v0BDAp818Bd*dc*dsc*dp*icFFiscFFipFFin*
    depends:        usbcore
    vermagic:       4.6.0-1-amd64 SMP mod_unload modversions
    parm:           rtw_ips_mode:The default IPS mode (int)
    parm:           rtw_usb_rxagg_mode:int
    parm:           rtw_qos_opt_enable:int
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
    parm:           rtw_tx_pwr_lmt_enable:0:Disable, 1:Enable, 2: Depend on efuse (int)
    parm:           rtw_tx_pwr_by_rate:0:Disable, 1:Enable, 2: Depend on efuse (int)
    parm:           rtw_phy_file_path:The path of phy parameter (charp)
    parm:           rtw_load_phy_file:PHY File Bit Map (int)
    parm:           rtw_decrypt_phy_file:Enable Decrypt PHY File (int)

Poniżej jest jeszcze standardowa konfiguracja modułu:

    # systool -v -m 8192eu
    Module = "8192eu"

      Attributes:
        coresize            = "909312"
        initsize            = "0"
        initstate           = "live"
        refcnt              = "0"
        srcversion          = "DC4944779709F5666CC49A7"
        taint               = "OE"
        uevent              = <store method only>
        version             = "v4.3.1.1_11320.20140505"

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
        rtw_channel_plan    = "88"
        rtw_channel         = "1"
        rtw_chip_version    = "0"
        rtw_decrypt_phy_file= "0"
        rtw_enusbss         = "0"
        rtw_ht_enable       = "1"
        rtw_hw_wps_pbc      = "1"
        rtw_hwpdn_mode      = "2"
        rtw_hwpwrp_detect   = "0"
        rtw_initmac         = "(null)"
        rtw_ips_mode        = "1"
        rtw_lbkmode         = "0"
        rtw_load_phy_file   = "68"
        rtw_low_power       = "0"
        rtw_lowrate_two_xmit= "1"
        rtw_max_roaming_times= "2"
        rtw_mc2u_disable    = "0"
        rtw_mp_mode         = "0"
        rtw_network_mode    = "0"
        rtw_notch_filter    = "0"
        rtw_phy_file_path   =
        rtw_power_mgnt      = "1"
        rtw_qos_opt_enable  = "0"
        rtw_rf_config       = "5"
        rtw_rfintfs         = "2"
        rtw_rx_stbc         = "1"
        rtw_smart_ps        = "2"
        rtw_tx_pwr_by_rate  = "0"
        rtw_tx_pwr_lmt_enable= "0"
        rtw_usb_rxagg_mode  = "2"
        rtw_vcs_type        = "1"
        rtw_vrtl_carrier_sense= "2"
        rtw_wifi_spec       = "0"
        rtw_wmm_enable      = "1"

      Sections:
        .bss                = "0xffffffffc0d01200"
        .data               = "0xffffffffc0ce9000"
        .exit.text          = "0xffffffffc0cc3cb4"
        .gnu.linkonce.this_module= "0xffffffffc0d00ec0"
        .init.text          = "0xffffffffc0d27000"
        .note.gnu.build-id  = "0xffffffffc0cc4000"
        .parainstructions   = "0xffffffffc0ce7bf0"
        .rodata             = "0xffffffffc0cc4040"
        .rodata.str1.1      = "0xffffffffc0cdf773"
        .rodata.str1.8      = "0xffffffffc0cd09c0"
        .smp_locks          = "0xffffffffc0ce7ba0"
        .strtab             = "0xffffffffc0d39448"
        .symtab             = "0xffffffffc0d28000"
        .text               = "0xffffffffc0c48000"
        __mcount_loc        = "0xffffffffc0ce4250"
        __param             = "0xffffffffc0ce7c10"

## Testy modułu 8192eu

Moduł przydałoby się jeszcze przetestować i sprawdzić czy aby karta TL-WN823N V2 w ogóle działa.
Podpinamy zatem nasz adapter do portu USB i po chwili w logu powinniśmy ujrzeć sporo komunikatów:

    kernel: usb 2-1.3: new high-speed USB device number 14 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0109
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: 802.11n NIC
    kernel: usb 2-1.3: Manufacturer: Realtek
    kernel: usb 2-1.3: SerialNumber: 00e04c000001
    ...
    kernel: rtl8192eu 2-1.3:1.0 wlan20: renamed from wlan1

Wyżej widzimy wyraźnie, że pojawił się w systemie nowy interfejs `wlan20` . Teraz możemy bez
większego problemu skonfigurować sobie [bezprzewodowe połączenie
sieciowe](/post/konfiguracja-polaczenia-wifi-pod-debianem/) i wydać `ifup wlan20` :

![moduł-8192eu-karta-TL-WN823N-v2](/img/2016/08/1.moduł-8192eu-karta-TL-WN823N-v2.png#huge)

## Problemy z modułem 8192eu

Moduł `8192eu` jest w stanie obsłużyć kartę TL-WN823N V2, z tym, że działa ona bardzo niestabilnie.
Problematyczne może być ponowne podłączenie adaptera do portu USB. W takim przypadku, moduł może
zachować się nieprzewidywalnie. Przy replug'u pomaga całkowite wyładowanie modułu za pomocą
`modproble -r` . Później nie trzeba go ponownie ładować, zrobi to event przy podłączaniu adaptera do
portu USB.
