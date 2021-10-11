---
author: Morfik
categories:
- Linux
date: "2016-08-18T14:35:52Z"
date_gmt: 2016-08-18 12:35:52 +0200
published: true
status: publish
tags:
- dkms
- moduły-kernela
- wifi
- usb
GHissueID: 550
title: Sterowniki dla karty WiFi Archer T1U (mt7610u_sta)
---

Dziś postanowiłem się wziąć za ostatnią kartę WiFi, którą podesłał mi TP-LINK. Jest to nano adapter
Archer T1U V1 na [czipie MediaTek MT7610U](https://wikidevi.com/wiki/TP-LINK_TL-WDN5200)
identyfikowany w systemie jako `idVendor=2357` , `idProduct=0105` . Na opakowaniu pisało, że ta
karta działa na linux'ach ale oczywiście w przypadku mojego Debiana, ten adapter nie został w ogóle
wykryty. Winą są zbyt stare sterowniki, które nie zostały zaktualizowane przez MediaTek od 2013
roku. TP-Link może i ma u siebie na stronie [nieco nowszą wersję
sterowników](http://www.tp-link.com/en/download/Archer-T1U.html#Driver), bo z 2015 roku ale nie
udało mi się za ich sprawą zbudować poprawnie modułu `mt7610u_sta` na kernelu 4.6 . Na szczęście
mamy jedną alternatywę, która pomoże nam jako tako wybrnąć z tej sytuacji.

<!--more-->
## Kompilacja modułu mt7610u\_sta dla Archer T1U V1

Nowszą wersję kodu źródłowego modułu `mt7610u_sta` możemy pozyskać z [z
github'a](https://github.com/xaep/mt7610u_wifi_sta_v3002_dpo_20130916). Ten sterownik ma kilka
fork'ów, niemniej jednak niektóre z nich działają, a inne nie. Generalnie trzeba przygotować się na
spore problemy. Ja zdecydowałem się na tę wersję modułu, bo się buduje, choć z błędami. Wymagane
jest jednak dostosowanie kilku rzeczy, by karta WiFi nam zadziałała. Pobierzmy sobie zatem źródła za
pomocą narzędzia `git` :

    $ git clone https://github.com/xaep/mt7610u_wifi_sta_v3002_dpo_20130916

### Problem z DKMS

W przypadku modułu `mt7610u_sta` , jako że jest on w opłakanym stanie, nie damy rady wykorzystać
mechanizmu DKMS do automatycznego budowania modułów ilekroć tylko będziemy instalować w systemie
nowszą wersję kernela. Na wypadek, gdyby kod tego modułu został poprawiony, zostawiam instrukcję dla
DKMS.

Kopiujemy utworzony za sprawą git'a katalog do folderu `/usr/src/` i nazywamy go `mt7610u_sta-1.0` :

    # mv mt7610u_wifi_sta_v3002_dpo_20130916/ /usr/src/mt7610u_sta-1.0

W źródłach nie ma odpowiedniej instrukcji dla DKMS i musimy takową napisać. Przechodzimy zatem do
katalogu `/usr/src/mt7610u_sta-1.0/` i tworzymy w nim plik `dkms.conf` o poniższej treści:

    PACKAGE_NAME="mt7610u_sta"
    PACKAGE_VERSION="1.0"
    BUILT_MODULE_NAME="mt7610u_sta"
    DEST_MODULE_LOCATION="/kernel/drivers/net/wireless/"
    REMAKE_INITRD="yes"
    AUTOINSTALL="yes"
    MAKE="'make' all"
    CLEAN="'make' clean"

Moduł budujemy w następujący sposób:

    # dkms install -m mt7610u_sta -v 1.0

### Make i make install

Póki moduł `mt7610u_sta` nie zostanie doprowadzony do porządku, musimy korzystać z `make` oraz `make
install` . W przeciwnym razie nie uda nam się zbudować modułu i karta nam nie będzie działać.
Odpalamy zatem terminal i wpisujemy w nim te oto polecenia:

    # make
    # make install

## Informacje o module mt7610u\_sta

Moduł powinien się już znajdować na swoim miejscu, a my być w stanie odczytać trochę informacji o
nim:

    # modinfo mt7610u_sta
    filename:       /lib/modules/4.6.0-1-amd64/kernel/drivers/net/wireless/mt7610u_sta.ko
    version:        3.0.0.2
    license:        GPL
    description:    RT2870 Wireless Lan Linux Driver
    author: Morfik         Paul Lin <paul_lin@ralinktech.com>
    license:        GPL
    srcversion:     0DF01AEB28FDA9F7C0AEE9F
    alias:          usb:v0E8Dp7650d*dc*dsc*dp*icFFisc02ipFFin*
    alias:          usb:v0E8Dp7630d*dc*dsc*dp*icFFisc02ipFFin*
    alias:          usb:v20F4p806Bd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2019pAB31d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v293Cp5702d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v057Cp8502d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v04BBp0951d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v07B8p7610d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0586p3425d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2001p3D02d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0DF6p0075d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0B05p17DBd*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0B05p17D1d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v148Fp760Ad*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v148Fp761Ad*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v7392pB711d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v7392pA711d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v13B1p003Ed*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v0E8Dp7610d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v148Fp7610d*dc*dsc*dp*ic*isc*ip*in*
    alias:          usb:v2357p0105d*dc*dsc*dp*ic*isc*ip*in*
    depends:        cfg80211,usbcore
    vermagic:       4.6.0-1-amd64 SMP mod_unload modversions
    parm:           mac:wireless mac addr (charp)
    parm:           debug:log verbosity level (0: off, 1: error only [default], 2: warnings, 3: trace, 4: info, 5: loud) (long)

## Test modułu mt7610u\_sta

System dysponuje już odpowiednim modułem. Wyżej widzimy również alias dla adaptera Archer T1U V1 (
`idVendor=2357` , `idProduct=0105` ), zatem powinien on zostać rozpoznany przez kernel. Podłączmy
kartę WiFi do portu USB. W systemie powinien zostać utworzony nowy interfejs sieciowy `ra0` . Teraz
wystarczy już tylko skonfigurować sam interfejs, np. przez plik `/etc/network/interfaces` :

    iface ra0 inet dhcp
          wpa-driver wext
          wpa-debug-level -1
          wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Po podniesieniu interfejsu `ra0` , powinniśmy zostać podłączeni do sieci bezprzewodowej:

    # iwconfig ra0
    ra0       Ralink STA  ESSID:"TP-LINK_15D5_5G"  Nickname:"MT7610U_STA"
              Mode:Managed  Frequency=5.22 GHz  Access Point: EC:08:6B:84:15:D4
              Bit Rate=40.5 Mb/s
              RTS thr:off   Fragment thr:off
              Encryption key:C422-4DA8-3E32-42F0-4A4A-2281-5A2F-ABEC [2]   Security mode:restricted   Security mode:open
              Link Quality=60/100  Signal level:-76 dBm  Noise level:-114 dBm
              Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
              Tx excessive retries:0  Invalid misc:0   Missed beacon:0

Nie za dobrze wyglądają te parametry.

## Problemy z modułem mt7610u\_sta

Ten moduł jest z bez wątpienia najgorszym z jakim miałem styczność. Może i kartę Archer T1U V1 udało
się ostatecznie uruchomić ale dla linux'a to ona nie jest przeznaczona zupełnie. Brak wsparcia w
sterowniku dla interfejsu `nl80211` jeszcze tylko bardziej pogarsza sprawę, bo większość aktualnych
narzędzi linux'owych nam zwyczajnie z tą kartą nie zadziała.
