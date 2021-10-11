---
author: Morfik
categories:
- Hardware
date: "2016-08-19T19:10:44Z"
date_gmt: 2016-08-19 17:10:44 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
GHissueID: 563
title: 'Recenzja: Karta WiFi TP-LINK Archer T1U'
---

Ostatnio recenzowałem dwie bezprzewodowe karty WiFi w standardzie mini/mikro, które podesłał mi
TP-LINK. W zestawie był jeszcze jeden adapter, a konkretnie chodzi o [kartę Archer
T1U](http://www.tp-link.com.pl/products/details/Archer-T1U.html). Jest ona bardzo podobna do
[TL-WN725N](/post/recenzja-karta-wifi-tp-link-tl-wn725n/), przynajmniej pod
względem wizualnym. To co odróżnia od siebie te dwa adaptery, to pasmo, w którym mogą pracować oraz
oczywiście prędkość transferu. Archer T1U działa w 5 GHz (standard AC) i teoretycznie może pochwalić
się prędkością do 433 mbit/s. Zobaczmy zatem jak on spisze się w przypadku linux'ów.

<!--more-->
## Zawartość pudełka

Opakowanie standardowe, tj. bardzo duże jak na rozmiary samego adaptera. W środku poza kartą Archer
T1U mamy też płytkę ze sterownikami i całą masę makulatury. Jest też instrukcja w języku polskim.
Całość wygląda mniej więcej tak:

![](/img/2016/08/1.TP-LINK-Archer-T1U-pudelko.jpg#huge)

![](/img/2016/08/2.TP-LINK-Archer-T1U-zawartosc-pudelka.jpg#huge)

![](/img/2016/08/3.TP-LINK-Archer-T1U-karta-adapter-wifi.jpg#huge)

![](/img/2016/08/4.TP-LINK-Archer-T1U-karta-adapter-wifi.jpg#huge)

![](/img/2016/08/5.TP-LINK-Archer-T1U-karta-adapter-wifi.jpg#huge)

![](/img/2016/08/6.TP-LINK-Archer-T1U-karta-adapter-wifi.jpg#huge)

Jak widać, adapter jest malutki. Po wsadzeniu go do portu USB, idealnie komponuje się z obudową
laptopa. Nie wystaje zanadto i nie idzie go uszkodzić przez przypadkowe zahaczenie:

![](/img/2016/08/7.TP-LINK-Archer-T1U-laptop.jpg#huge)

## Sterowniki i firmware dla Archer T1U

Niestety, w porównaniu do swojego kolegi, tj TL-WN725N, adapter Archer T1U nie jest wykrywany w
ogóle przez mojego Debiana. Niby na opakowaniu jest wyraźnie napisane, że jest wsparcie dla linux'a
ale widać powoli trzeba odchodzić od wiary w to co producenci wypisują na opakowaniach. Po wsadzeniu
tej karty do portu USB, w logu systemowym zobaczymy jedynie te poniższe komunikaty.

    kernel: usb 2-1.3: new high-speed USB device number 9 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0105
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: WiFi
    kernel: usb 2-1.3: Manufacturer: MediaTek
    kernel: usb 2-1.3: SerialNumber: 1.0
    mtp-probe[51104]: checking bus 2, device 9: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[51104]: bus: 2, device: 9 was not an MTP device

Karta jest na czipie MediaTek MT7610U. Sterowniki OpenSource od MediaTek dla tego chipu ostatni raz
były aktualizowane w 2013 roku. Zaś te driver'y, które są na stronie TP-LINK'a, są datowane na
środek roku 2015. Świat nie stoi w miejscu i przez te parę lat kernel linux'a się trochę zmienił.
Także na obecnym kernelu 4.6 te sterowniki się nam zbytnio nie przydadzą, chyba, że ktoś je ogarnie
i zmodyfikuje w taki sposób, by się zbudowały bez problemu na kernelu 4.6 .

Po wielu trudach znalezienia rozwiązania problemu sterowników do karty Archer T1U, natrafiłem na
[fork bazowych sterowników](https://github.com/xaep/mt7610u_wifi_sta_v3002_dpo_20130916), które
znośnie się budują na najnowszym kernelu dostępnym w Debianie. Znośnie nie oznacza bez błędów, oraz
też pozostaje kwestia jakości samego modułu. [Proces budowy modułu
mt7610u_sta](/post/sterowniki-karta-wifi-archer-t1u-mt7610u_sta/) został opisany w
osobnym artykule.

## Specyfikacja karty Archer T1U

Trzeba wyraźnie zaznaczyć, że Archer T1U pod kontrolą sterownika `mt7610u_sta` zachowuje się bardzo
niestabilnie. Ten moduł cierpi także na brak wsparcia dla [interfejsu
nl80211](https://wireless.wiki.kernel.org/en/developers/documentation/nl80211). Zatem narzędzia
takie jak `iw` , `crda` , `hostapd` czy `wpa_supplicant` nie będą chciały z nim współpracować.

Poniżej zaś jest zamieszczone wyjście polecenia `lsusb` :

    # lsusb -vvv -d 2357:0105

    Bus 002 Device 012: ID 2357:0105
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.01
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x2357
      idProduct          0x0105
      bcdDevice            1.00
      iManufacturer           1 MediaTek
      iProduct                2 WiFi
      iSerial                 3 1.0
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           74
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower              160mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           8
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      2
          bInterfaceProtocol    255
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x84  EP 4 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x85  EP 5 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x08  EP 8 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x04  EP 4 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
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
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x06  EP 6 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x07  EP 7 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x09  EP 9 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               1
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength           12
      bNumDeviceCaps          1
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x00000002
          Link Power Management (LPM) Supported
    Device Status:     0x0000
      (Bus Powered)

## Problemy z Archer T1U pod linux'em

Adapter WiFi Archer T1U nie działa za dobrze pod linux'em. Jeśli miałbym być szczery, z tą kartą są
same problemy. Transfer jest wręcz tragiczny, a zasięg jeszcze gorszy. Zanim jednak przejdziemy do
testów, trzeba wspomnieć, o fakcie, że logu systemowym podczas pracy tego adaptera jest generowana
cała masa komunikatów podobnych do tego poniżej:

    kernel: DeQueueRunning[0]=] TRUE!

Co jakiś czas można też zauważyć informacje o temperaturze
    czipu:

    kernel: MlmePeriodicExec - Do VCORecalibration again!(LastTemperatureforVCO=28, NowTemperature = 49)
    kernel: MlmePeriodicExec - Do VCORecalibration again!(LastTemperatureforVCO=49, NowTemperature = 70)

Podczas 5 minut pracy (intensywnego pobierania danych), karta się zagotowała tak mocno, że przestała
działać. Trzeba było posiłkować się dodatkowym wiatrakiem chłodzącym, by dokończyć testy. Generalnie
odradzam tę bezprzewodową kartę wszystkim linux'iarzom.

## Testy Archer T1U

Jakieś testy trzeba przeprowadzić, by nie być gołosłownym. Zatem po podpięciu adaptera do portu USB,
zapuściłem `iperf` . Poniżej jest fotka obrazująca transfer z laptopa oddalonego od routera o jakieś
2 metry:

![](/img/2016/08/7.TP-LINK-Archer-T1U-test-linux.png#huge)

Biorąc pod uwagę fakt rozpoczęcia pracy i przepracowane 2 minuty, to transfer jest taki sobie.
Pamiętajmy, że Archer T1U ma pracować w paśmie 5 GHz i osiągać prędkość do 433 mbit/s. Póki co
nawet się nie zbliżył do połowy tej wartości.

Przestawmy laptopa trochę dalej (3-4 metry) od routera i między te dwa urządzenia wstawmy niezbyt
grubą ścianę:

![](/img/2016/08/8.TP-LINK-Archer-T1U-test-linux.png#huge)

Tu już mamy dość poważny problem z ubytkiem transferu.

No to pójdźmy jeszcze dalej (6 metrów) i dołóżmy jeszcze jedną ścianę:

![](/img/2016/08/9.TP-LINK-Archer-T1U-test-linux.png#huge)

Z początku nie było tak źle, w porównaniu do poprzedniej próby. Natomiast w połowie testu coś się
stało i transfer zdechł zupełnie. To jest właśnie ten moment, w którym karta uległa przegrzaniu.
Ostatnie dwie próbki, są po podłączeniu wiatraka. Zatem bez dodatkowego chłodzenia, Archer T1U pod
linux'em wysiądzie po 5 minutach transferu z prędkością 20 mbit/s.
