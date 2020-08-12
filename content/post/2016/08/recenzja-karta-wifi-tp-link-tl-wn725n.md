---
author: Morfik
categories:
- Hardware
date: "2016-08-11T21:00:33Z"
date_gmt: 2016-08-11 19:00:33 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
title: 'Recenzja: Karta WiFi TP-LINK TL-WN725N (r8188eu)'
---

Świat ciągle idzie do przodu i oferuje nam coraz to nowsze rozwiązania, które są w większym stopniu
dostosowane do naszych potrzeb. Mobilność w obecnych czasach to podstawa. Niekoniecznie jednak
musimy upychać naszą wirtualną rzeczywistość w urządzeniu wielkości smartfona. Może takie
rozwiązanie idealnie nadaje się dla osób podróżujących ale też nic nie stoi na przeszkodzie, by
zabrać ze sobą nieco większy komputer w postaci laptopa. Niemniej jednak, w terenie ciężko o
przewód, którym moglibyśmy się podpiąć do sieci. Dlatego też technologia WiFi jest nam zwykle
niezbędna. Na rynku jest wiele kart i adapterów, które mogą nam umożliwić połączenie bezprzewodowe,
choć nie wszystkie z nich są na tyle poręczne, abyśmy rozważali ich zakup i użytkowanie. Potrzebne
jest nam raczej urządzenie na tyle minimalistyczne, że na pierwszy rzut oka nie zauważymy żadnej
różnicy po jego podłączeniu do komputera. Firma TP-LINK oferuje kilka rozwiązań tego typu i w tym
wpisie przyjrzymy się nieco bliżej [nano adapterowi WiFi
TL-WN725N](http://www.tp-link.com.pl/products/details/TL-WN725N.html) i przetestujemy jego
kompatybilność z linux'ami.

<!--more-->
## Zawartość pudełka

Pudełko gabarytowo jest ogromne w porównaniu do rozmiarów adaptera TL-WN725N. W zasadzie rozmiary
opakowania są dyktowane makulaturą (instrukcja obsługi) i płytką CD ze sterownikami. Poniżej są
fotki opakowania oraz jego
zawartości:

[![1.karta-adapter-wifi-TL-WN725N]({{< baseurl >}}/img/2016/08/1.karta-adapter-wifi-TL-WN725N-1024x577.jpg)]({{< baseurl >}}/img/2016/08/1.karta-adapter-wifi-TL-WN725N.jpg)

[![2.karta-adapter-wifi-TL-WN725N-zawartosc-opakowania]({{< baseurl >}}/img/2016/08/2.karta-adapter-wifi-TL-WN725N-zawartosc-opakowania-1024x577.jpg)]({{< baseurl >}}/img/2016/08/2.karta-adapter-wifi-TL-WN725N-zawartosc-opakowania.jpg)

Instrukcja obsługi jest również w języku polskim. Niżej zaś jest przedstawiony sam
adapter:

[![3.karta-adapter-wifi-TL-WN725N-rozmiar]({{< baseurl >}}/img/2016/08/3.karta-adapter-wifi-TL-WN725N-rozmiar-1024x577.jpg)]({{< baseurl >}}/img/2016/08/3.karta-adapter-wifi-TL-WN725N-rozmiar.jpg)

[![4.karta-adapter-wifi-TL-WN725N-rozmiar]({{< baseurl >}}/img/2016/08/4.karta-adapter-wifi-TL-WN725N-rozmiar-1024x577.jpg)]({{< baseurl >}}/img/2016/08/4.karta-adapter-wifi-TL-WN725N-rozmiar.jpg)

Jest on mały, choć 2/3 jego wielkości to złącze USB 2.0 i raczej mniejszego urządzenia się już
zrobić nie da. No może się i da ale raczej problematyczne byłoby wyciąganie go z portu USB.

Po podłączeniu do laptopa, karta jest praktycznie niezauważalna. Jedynie podczas pracy mieni się
zielonym kolorem sygnalizującym przesył danych w sieci WiFi. Wygląda to mniej więcej
tak:

[![5.karta-adapter-wifi-TL-WN725N-laptop-dioda]({{< baseurl >}}/img/2016/08/5.karta-adapter-wifi-TL-WN725N-laptop-dioda-1024x577.jpg)]({{< baseurl >}}/img/2016/08/5.karta-adapter-wifi-TL-WN725N-laptop-dioda.jpg)

Plusem tak małych rozmiarów jest fakt, że praktycznie nie uszkodzimy tego urządzenia (czy samego
portu USB) przez przypadkowe zahaczenie o nie, tak jak to ma miejsce w przypadku innych kart
bezprzewodowych, które wystają kilka centymetrów poza obudowę laptopa.

## Jaką wersję adaptera TL-WN725N posiadam

Na rynku można spotkać dwie wersje karty TL-WN725N. W tym przypadku mamy do czynienia z
[V2](https://wikidevi.com/wiki/TP-LINK_TL-WN725N_v2) ale też gdzieniegdzie widuje się
[V1](https://wikidevi.com/wiki/TP-LINK_TL-WN725N_v1) . Po wyglądzie praktycznie nie damy rady ich
odróżnić. Jeśli nie posiadamy pudełka, to jedyną opcją, jaka nam pozostaje, to podłączenie karty do
portu USB i identyfikacja jej przez system. Każda z wersji działa na linux'ie w oparciu o inny
moduł. Wersja V1 wykorzystuje `rtl8192cu` zaś wersja V2 `r8188eu` .

## Sterowniki i firmware dla TL-WN725N

Na opakowaniu adaptera TL-WN725N widnieje informacja, że to urządzenie jest wspierane min. przez
alternatywny system operacyjny jakim jest linux. Sprawdzimy czy faktycznie tak jest i podłączmy tę
kartę do portu USB naszego komputera. W logu systemowym powinny pojawić się poniższe komunikaty:

    kernel: usb 2-1.3: new high-speed USB device number 12 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=0bda, idProduct=8179
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: 802.11n NIC
    kernel: usb 2-1.3: Manufacturer: Realtek
    kernel: usb 2-1.3: SerialNumber: 00E04C0001
    mtp-probe[19903]: checking bus 2, device 12: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[19903]: bus: 2, device: 12 was not an MTP device
    kernel: r8188eu: module is from the staging directory, the quality is unknown, you have been warned.
    kernel: Chip Version Info: CHIP_8188E_Normal_Chip_TSMC_D_CUT_1T1R_RomVer(0)
    kernel: usbcore: registered new interface driver r8188eu
    kernel: r8188eu 2-1.3:1.0 wlan10: renamed from wlan1

Widzimy zatem, że adapter TL-WN725N jest identyfikowany przez kernel linux'a jako `idVendor=0bda`
oraz `idProduct=8179` . Dalej mamy informację, że karta działa w oparciu o [moduł
r8188eu](https://github.com/lwfinger/rtl8188eu). Z tym, że w aktualnym kernelu (4.6), ten moduł
pochodzi z tzw. [staging directory](https://lwn.net/Articles/324279/), przez co mogą pojawić się
problemy natury użytkowej.

Jeśli byśmy z miejsca chcieli podłączyć się do sieci WiFi, to nie będziemy w stanie tego zrobić:

    kernel: r8188eu 2-1.3:1.0: firmware: failed to load rtlwifi/rtl8188eufw.bin (-2)
    kernel: r8188eu 2-1.3:1.0: Direct firmware load for rtlwifi/rtl8188eufw.bin failed with error -2
    kernel: r8188eu 2-1.3:1.0: Firmware rtlwifi/rtl8188eufw.bin not available

Mamy tutaj informację, że karta TL-WN725N wymaga do pracy firmware. W repozytorium Debiana figuruje
pakiet `firmware-realtek` , w którym to znajduje się wymieniony wyżej plik powiązany z modułem
`r8188eu` , tj. `/lib/firmware/rtlwifi/rtl8188eufw.bin` . Zatem bez instalacji tego pakietu się nie
obejdzie.

### Sterowniki dla TL-WN725N oferowane przez TP-LINK

Na płytce dołączonej do karty jest informacja, że na stronie TP-LINK znajdują się [linux'owe
sterowniki](http://www.tp-link.com/en/download/TL-WN725N.html#Driver) dla tego adaptera. Jest nawet
instrukcja jak skompilować i zainstalować wymagany moduł, z tym, że dla kernela od wersji 2.6.18 do
3.19.3 , czyli nieco leciwego. Oczywiście moduł może działać i na późniejszych wersjach ale nikt nie
gwarantuje, że da się ten moduł zbudować oraz, że będzie on działał stabilnie. Przy obecnej wersji
kernela w gałęzi testowej pozostaje nam raczej korzystanie z modułu oferowanego przez sam kernel.

## Specyfikacja karty TL-WN725N

Kawałek logu widoczny powyżej mówi nam tylko tyle, że karta TL-WN725N jest w stanie pracować pod
linux'em. Jeśli chodzi zaś o bardziej szczegółowe informacje, to wiemy, że na pokładzie tego
adaptera znajduje się chipset Realtek RTL8188EUS ale nasz sterownik cierpi na brak wsparcia dla
[interfejsu nl80211](https://wireless.wiki.kernel.org/en/developers/documentation/nl80211), przez co
są drobne problemy z obsługą tego czipa w narzędziach `iw` , `crda` , `hostapd` oraz
`wpa_supplicant` .

Poniżej zaś jest zamieszczone wyjście polecenia `lsusb` :

    # lsusb -vvv -d 0bda:8179

    Bus 002 Device 012: ID 0bda:8179 Realtek Semiconductor Corp. RTL8188EUS 802.11n Wireless Network Adapter
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x0bda Realtek Semiconductor Corp.
      idProduct          0x8179 RTL8188EUS 802.11n Wireless Network Adapter
      bcdDevice            0.00
      iManufacturer           1 Realtek
      iProduct                2 802.11n NIC
      iSerial                 3 00E04C0001
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           39
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower              500mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           3
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

Karta działa w standardzie N do 150 mbit/s. Niestety nie mogę załączyć również wyjścia polecenia `iw
list` z przyczyn opisanych powyżej.

### Parametry modułu r8188eu

Moduł `r8188eu` obsługujący adapter TL-WN725N ma sam w sobie dość sporo parametrów, które możemy
dostosować. Poniżej znajduje się kompletna lista wraz z domyślnymi ustawieniami dla tej karty:

    # systool -v -m r8188eu
    ...
      Parameters:
        debug               = "1"
        if2name             = "wlan%d"
        ifname              = "wlan%d"
        monitor_enable      = "N"
        rtw_80211d          = "0"
        rtw_ampdu_amsdu     = "0"
        rtw_ampdu_enable    = "1"
        rtw_antdiv_cfg      = "2"
        rtw_antdiv_type     = "0"
        rtw_busy_thresh     = "40"
        rtw_cbw40_enable    = "3"
        rtw_channel_plan    = "66"
        rtw_channel         = "1"
        rtw_chip_version    = "0"
        rtw_enusbss         = "0"
        rtw_fw_iol          = "1"
        rtw_ht_enable       = "1"
        rtw_hw_wps_pbc      = "1"
        rtw_hwpdn_mode      = "2"
        rtw_hwpwrp_detect   = "0"
        rtw_initmac         = "(null)"
        rtw_ips_mode        = "1"
        rtw_lbkmode         = "0"
        rtw_low_power       = "0"
        rtw_lowrate_two_xmit= "1"
        rtw_max_roaming_times= "2"
        rtw_mc2u_disable    = "0"
        rtw_network_mode    = "0"
        rtw_notch_filter    = "0"
        rtw_power_mgnt      = "1"
        rtw_rf_config       = "5"
        rtw_rfintfs         = "2"
        rtw_rx_stbc         = "1"
        rtw_smart_ps        = "2"
        rtw_vcs_type        = "1"
        rtw_vrtl_carrier_sense= "2"
        rtw_wifi_spec       = "0"
        rtw_wmm_enable      = "1"

### Tryb monitor

Lista opcji zwróciła nam min. pozycję `monitor_enable` , która sugeruje, że adapter TL-WN725N może
posiadać obsługę [trybu monitor](https://pl.wikipedia.org/wiki/Promiscuous_mode). Standardowo jest
on wyłączony ale jeśli chcielibyśmy włączyć go, to trzeba załadować moduł z opcją `monitor_enable=1`
. A to z kolei można zrobić, np. za pomocą pliku `/etc/modprobe.d/modules.conf` przez dodanie w nim
poniższej linijki:

    options r8188eu monitor_enable=1

Następnie trzeba przeładować moduł:

    # ifdown wlan10
    # modprobe -r r8188eu
    # modprobe r8188eu
    # ifup wlan10

Można także załadować moduł bezpośrednio z linii poleceń:

    # modprobe -r r8188eu
    # modprobe r8188eu monitor_enable=1

W każdym z tych dwóch przypadków, w systemie powinniśmy zobaczyć nowy interfejs `mon0` , który można
wykorzystać choćby w `wireshark` czy `airodump-ng` .

### Tryb oszczędzania energii

W opcjach modułu można także doszukać się parametru `rtw_power_mgnt` , który odpowiada za tryb
oszczędzania energii w kartach i adapterach WiFi. To nic nowego ale na linux'ach ten tryb często
sprawia problemy i zaleca się jego wyłączenie. Jeśli obserwujemy wysoki `ping` podczas spoczynku
karty, czy też idzie wyraźnie odczuć opóźnienie związane z przeglądaniem stron www, to powinniśmy
wyłączyć automatyczne zarządzanie energią. By to uczynić, w pliku `/etc/modprobe.d/modules.conf`
dodajemy poniższy wpis:

    options r8188eu rtw_power_mgnt=0

Do końca nie wiem czy obsługa trybu oszczędnego jest zaimplementowana w tym module, czy też i nie
jest. Karta zdaje się pracować poprawnie bez względu na stan opcji `rtw_power_mgnt` . Niemniej
jednak, w `iwconfig` widnieje `Power Management:off` zarówno w przypadku ustawienia tej opcj na 0
jak i na 1.

### Programowy WPS

Karta jest na tyle mała, że nie sposób na niej zamontować przycisku WPS. Niemożliwa zatem jest
szybka i bezproblemowa konfiguracja połączenia WiFi za pomocą jednego wciśnięcia przycisku. Niemniej
jednak, przyciski WPS pod linux'em są znane z tego, że nie działają. Zatem niewiele tracimy przez
fakt braku przycisku WPS na obudowie adaptera.

Ważniejszą rzeczą zaś jest programowy WPS, który działa bez problemu na linux'ach. Na liście opcji
modułu mamy pozycję `rtw_hw_wps_pbc` implementującą obsługę wirtualnego przycisku, który "można
wcisnąć" programowo z poziomu systemu operacyjnego. Opcja jest domyślnie włączona zatem powinniśmy
być w stanie bez problemu wynegocjować parametry połączenia.

### Tryb AP

Istnieje także możliwość przełączenia tej karty w tryb AP, z tym, że standardowo w linux'ie się tego
nie da zrobić -- brak wsparcia dla sterownika `nl80211` w `hostapd` . Z tej sytuacji można wybrnąć
przez wgranie łaty na `hostapd` implementującą [sterownik
rtl871xdrv](https://github.com/pritambaral/hostapd-rtl871xdrv). Niemniej jednak dokładny opis jak
uruchomić tryb AP na karcie TL-WN725N jest poza zakresem tego artykułu. Na pewno taki opis pojawi
się w przyszłości i zostanie tutaj podlinkowany (FIXME).

## Test karty TL-WN725N

Karta została poprawnie rozpoznana przez system linux i działa. Połączenie z siecią WiFi można
nawiązać bez większego problemu i nie widać zbytnio błędów. Karta zachowuje się w miarę stabilnie i
nie zrywa połączenia. Przydałoby się zatem sprawdzić jeszcze jaki transfer jesteśmy w stanie uzyskać
za pomocą karty TL-WN725N.

Poniższa fotka odnosi się do sytuacji, w której laptop znajduje się w tym samym pomieszczeniu co
router WiFi TP-LINK Archer C7 v2 (wewnętrzne anteny 2.4 GHz). Odległość nie przekracza 2
metrów:

![]({{< baseurl >}}/img/2016/08/4.iperf-test-transfer-karta-adapter-wifi-TL-WN725N.png)

Zatem mamy około 80 mbit/s. Całkiem przyzwoity wynik jak na tak bardzo mały adapter, choć na
opakowaniu widnieje 150 mbit/s. Sprawdźmy jak będzie wyglądał transfer po zmianie położenia laptopa
i dołożeniu przeszkody w postaci niezbyt grubej ściany. Odległość około 3-4
metry:

![]({{< baseurl >}}/img/2016/08/5.iperf-test-transfer-karta-adapter-wifi-TL-WN725N.png)

Transfer lekko się obniżył, choć nadal pozostaje powyżej 50 mbit/s. Dorzućmy jeszcze jedną ścianę i
zobaczmy jak adapter TL-WN725N poradzi sobie w takiej sytuacji. Odległość 5-6
metrów:

![]({{< baseurl >}}/img/2016/08/6.iperf-test-transfer-karta-adapter-wifi-TL-WN725N.png)

Tutaj już balansujemy na granicy 50 mbit/s. Choć i tak jestem pod wrażeniem, że taki transfer się
utrzymuje.

## Problemy z TL-WN725N pod linux'em

Dostępny w kernelu linux'a moduł może i zapewnia w miarę normalne użytkowanie adaptera WiFi
TL-WN725N ale jeszcze trzeba nad nim trochę popracować. Problemy są widoczne, np. w `wavemon` ,
gdzie nie można ustalić siły i jakości sygnału. W przypadku `iw` i `rfkill` w ogóle nie zostają
zwrócone żadne informacje. Ta sytuacja nie wpływa jednak w widoczny sposób na działanie samego
adaptera.

W przypadku obsługi sieci WiFi na linux'ie realizowanej za sprawą narzędzia `wpa_supplicant` trzeba
wziąć poprawkę w stosunku do wykorzystywanego sterownika. Standardowo w pliku
`/etc/network/interfaces` określilibyśmy coś na wzór `wpa-driver nl80211` . Niemniej jednak, w
przypadku TL-WN725N, ten sterownik nie zadziała i przy próbie podłączenia zostanie zwrócony poniższy
komunikat:

    # ifup wlan10
    wpa_supplicant: /sbin/wpa_supplicant daemon failed to start
    run-parts: /etc/network/if-pre-up.d/wpasupplicant exited with return code 1
    Failed to bring up wlan10.

Musimy zatem korzystać z `wpa-driver wext` lub też w ogóle nie precyzować tej opcji w konfiguracji
(na jedno wyjdzie). Reasumując, zwrotka konfigurująca interfejs sieciowy w pliku
`/etc/network/interfaces` powinna wyglądać mniej więcej tak:

    iface wlan10 inet dhcp
          wpa-driver wext
          wpa-debug-level -1
          wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Oczywiście parametry specyficzne dla samej sieci WiFi podajemy już w konfiguracji narzędzia
`wpa_supplicant` . Poniżej znajduje się log z podłączania do sieci bezprzewodowej:

![]({{< baseurl >}}/img/2016/08/7.karta-TL-WN725N-dzialanie-pod-linux.png)

Także trochę błędów jest ale na szczęście nic poważnego.
