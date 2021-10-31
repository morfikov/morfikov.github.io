---
author: Morfik
categories:
- Hardware
date: "2016-09-22T17:24:55Z"
date_gmt: 2016-09-22 15:24:55 +0200
published: true
status: publish
tags:
- usb
- recenzja
- tp-link
GHissueID: 360
title: 'Recenzja: Karta sieciowa Ethernet na USB UE200 od TP-LINK'
---

Czasami bardzo dziwne rzeczy mogą się przytrafić naszym komputerom. Kilka tygodni temu, z
niewiadomych przyczyn port ethernet w moim laptopie zwyczajnie zamarł. Linux go nie rozpoznaje i nie
da rady za jego pośrednictwem podłączyć się do sieci. Można oczywiście skorzystać z WiFi ale w
pewnych sytuacjach ta opcja odpada i zostaje nam jedynie łącze przewodowe. Tak się składa, że mam na
wyposażeniu [adapter UE200](http://www.tp-link.com.pl/products/details/UE200.html) od TP-LINK. Jest
to karta sieciowa mająca z jednej strony gniazdko ethernet, z drugiej zaś złącze USB, dzięki któremu
łatwo tę kartę można podłączyć do komputera. Pytanie tylko jak taka sieciówka na USB zachowa się pod
linux'em. W tym wpisie obadamy sobie właśnie tę kwestię.

<!--more-->
## Zawartość pudełka

W sumie, to na dobrą sprawę nie ma co pokazywać, bo pudełko małe i w zasadzie znajduje się w nim sam
adapter i kawałek kartki z instrukcją dotyczącą tego jak taką kartę podłączyć do laptopa czy innego
komputera. Niemniej jednak, jak ktoś ciekaw to poniżej są fotki:

![adapter-ue200-karta-sieciowa-ethernet-usb-opakowanie](/img/2016/09/1.adapter-ue200-karta-sieciowa-ethernet-usb-opakowanie.jpg#huge)

![adapter-ue200-karta-sieciowa-ethernet-usb](/img/2016/09/2.adapter-ue200-karta-sieciowa-ethernet-usb.jpg#huge)

Do najmniejszych ten adapter nie należy. Jego wymiary to 71 x 26 x 16,2 mm. Na adapterze nie ma
żadnych przycisków. Jest tylko jedna mała dioda, która sygnalizuje status urządzenia, tj. czy jest
ono włączone oraz czy są przesyłane dane przez sieć. Dioda ma kolor bały i świeci dość jasno.

![adapter-ue200-karta-sieciowa-ethernet-usb-dioda](/img/2016/09/3.adapter-ue200-karta-sieciowa-ethernet-usb-dioda.jpg#huge)

Poniżej jest jeszcze widok od spodu:

![adapter-ue200-karta-sieciowa-ethernet-usb-spod](/img/2016/09/4.adapter-ue200-karta-sieciowa-ethernet-usb-spod.jpg#huge)

![adapter-ue200-karta-sieciowa-ethernet-usb-spod](/img/2016/09/5.adapter-ue200-karta-sieciowa-ethernet-usb-spod.jpg#huge)

## Specyfikacja adaptera UE200

Nazwa karty może lekko wprowadzać w błąd, bo UE200 może nam mylnie sugerować, że ten adapter jest w
stanie przesyłać dane z prędkością 200 mbit/s. Niemniej jednak, do wyboru zawsze mamy 10/100/1000
mbit/s. A, że jest to karta na USB2, to możemy przypuszczać, że raczej nie mamy tutaj do czynienia z
gigabitowym portem, zatem zostaje nam opcja 10/100 mbit/s.

Na pokładzie adaptera znajduje się czip RTL8152B, który pod linux'em jest obsługiwany przez moduł
`cdc_ether` . Ten moduł standardowo powinien być obecny w kernelu. Adapter UE200 nie wymaga też
instalacji żadnego dodatkowego firmware. Karta jest rozpoznawana przez mojego linux'a, a konkretnie
jest to dystrybucja Debian. Poniżej jest log zarejestrowany przez system po podłączeniu urządzenia
do portu USB:

    kernel: usb 2-1.3: new high-speed USB device number 12 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0602
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: USB 10/100 LAN
    kernel: usb 2-1.3: Manufacturer: TP-LINK
    kernel: usb 2-1.3: SerialNumber: 98DED01395AE
    kernel: cdc_ether 2-1.3:2.0 eth1: register 'cdc_ether' at usb-0000:00:1d.0-1.3, CDC Ethernet Device, 98:de:d0:13:95:ae
    kernel: usbcore: registered new interface driver cdc_ether

Jak widzimy, adapter identyfikuje się jako `idVendor=2357` oraz `idProduct=0602` . Został także
zarejestrowany nowy interfejs sieciowy `eth1` , który ma przypisany adres MAC `98:de:d0:13:95:ae` .
Załadowany został też moduł `cdc_ether` .

Poniżej znajduje się także wyjście polecenia `lsusb` :

    # lsusb -vvv -d 2357:0602

    Bus 002 Device 060: ID 2357:0602
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.10
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x2357
      idProduct          0x0602
      bcdDevice           20.00
      iManufacturer           1 TP-LINK
      iProduct                2 USB 10/100 LAN
      iSerial                 3 98DED01395AE
      bNumConfigurations      2
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
        MaxPower              100mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           3
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass    255 Vendor Specific Subclass
          bInterfaceProtocol      0
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
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0002  1x 2 bytes
            bInterval               8
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           80
        bNumInterfaces          2
        bConfigurationValue     2
        iConfiguration          0
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower              100mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         2 Communications
          bInterfaceSubClass      6 Ethernet Networking
          bInterfaceProtocol      0
          iInterface              5 CDC Communications Control
          CDC Header:
            bcdCDC               1.10
          CDC Union:
            bMasterInterface        0
            bSlaveInterface         1
          CDC Ethernet:
            iMacAddress                      3 98DED01395AE
            bmEthernetStatistics    0x00000000
            wMaxSegmentSize               1514
            wNumberMCFilters            0x0000
            bNumberPowerFilters              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0010  1x 16 bytes
            bInterval               8
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       0
          bNumEndpoints           0
          bInterfaceClass        10 CDC Data
          bInterfaceSubClass      0 Unused
          bInterfaceProtocol      0
          iInterface              0
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       1
          bNumEndpoints           2
          bInterfaceClass        10 CDC Data
          bInterfaceSubClass      0 Unused
          bInterfaceProtocol      0
          iInterface              4 Ethernet Data
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

## Praca adaptera UE200 pod linux

Może i linux nie ma problemów z wykryciem karty UE200 na kernelu 4.7 ale jej sterownik pozostawia
raczej sporo do życzenia, a oto dlaczego:

    # ethtool eth1
    Settings for eth1:
            Current message level: 0x00000007 (7)
                                   drv probe link
            Link detected: no

Coś niewiele zostało zwrócone w wyjściu polecenia `ethtool` , no i nie bez powodu. Podczas próby
zmiany jednego ze standardowych parametrów kart ethernet pojawia się taki oto błąd:

    # ethtool -s eth1 autoneg off
    Cannot get current device settings: Operation not supported
      not setting autoneg

    # ethtool -i eth1
    driver: cdc_ether
    version: 22-Aug-2005
    firmware-version: CDC Ethernet Device
    expansion-rom-version:
    bus-info: usb-0000:00:1d.0-1.3
    supports-statistics: no
    supports-test: no
    supports-eeprom-access: no
    supports-register-dump: no
    supports-priv-flags: no

I na dobrą sprawę nie zmienimy żadnych ustawień tej karty. Zatem UE200 działa ale nie mamy nad nim
praktycznie żadnej kontroli. Przydałoby się jeszcze sprawdzić jak adapter UE200 zachowuje się w
alternatywnym środowisku, o ile w ogóle on nam będzie działać. Podłączamy do tej karty przewód
ethernet i konfigurujemy interfejs `eth1` za pomocą pliku `/etc/network/interfaces` , czy też w inny
dogodny dla nas sposób. Po skonfigurowaniu podnosimy interfejs:

![adapter-ue200-linux-adresacja](/img/2016/09/6.adapter-ue200-linux-adresacja.png#huge)

Adresacja została uzyskana, zatem przetestujmy przy pomocy `iperf` jaką faktyczną przepustowość ma
ten adapter pod linux:

![adapter-ue200-linux-test](/img/2016/09/7.adapter-ue200-linux-test.png#huge)

Jak widać, udało się osiągną 94 mbit/s i jest w sumie maksimum dla takich kart ethernet. Pozostałe
5-6 mbit/s zjada narzut (overhead) protokołu TCP/IP. Karta działa stabilnie i nie grzeje się. Jedyny
jej minus to brak zmiany ustawień.
