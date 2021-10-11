---
author: Morfik
categories:
- Hardware
date: "2017-01-28T19:12:22Z"
date_gmt: 2017-01-28 18:12:22 +0100
published: true
status: publish
tags:
- usb
- recenzja
- tp-link
GHissueID: 61
title: 'Recenzja: Gigabitowy adapter Ethernet na USB UE300 od TP-LINK'
---

Jakiś czas temu opisywałem [kartę sieciową na USB
UE200](/post/recenzja-karta-sieciowa-ethernet-usb-ue200-tp-link/) od TP-LINK. Taki
adapter jest bardzo przydatny w momencie, gdy nie posiadamy z jakiegoś powodu standardowej karty
sieciowej, tak by za jej sprawą przewodowo połączyć komputer do domowej sieci. Jeden z moich
komputerów cierpi właśnie na tego typu przypadłość z powodu jakiegoś bliżej nieznanego mi
uszkodzenia jego wbudowanej karty sieciowej. Generalnie rzecz biorąc w ofercie TP-LINK poza tym
wspomnianym UE200 jest także dostępny [adapter
UE300](http://www.tp-link.com.pl/products/details/cat-5688_UE300.html), który różni się głównie tym,
że ma gigabitowy port Ethernet oraz karta jest na USB 3.0. Jako, że jestem w posiadaniu adaptera
UE300, to postanowiłem sprawdzić jak (i czy w ogóle) jest rozpoznawany przez system z rodziny linux,
a konkretnie na dystrybucji Debian, której używam na co dzień.

<!--more-->
## Zawartość opakowania UE300

Na początek jak zawsze fotki omawianego urządzenia. W przypadku adaptera UE300 nie ma ich zbyt
wiele, bo sam adapter jest w miarę prosty i niezbyt skomplikowany.

![](/img/2016/12/001.ue300-tp-link-ethernet-adapter-usb3-pudelko.jpg#big)

![](/img/2016/12/002.ue300-tp-link-ethernet-adapter-usb3-zawartosc-pudelka.jpg#big)

## Specyfikacja adaptera UE300

Adapter UE300 jakby nie patrzeć to w końcu zwykły kawałek sieciowego czipu w plastikowej obudowie z
wyprowadzonym interfejsem USB 3.0 z jednej strony i gigabitowym portem Ethernet (RJ-45) z drugiej.
Jeśli ktoś jest ciekaw wymiarów tego adaptera, to oto i one: 71x26x16 mm (dł/sz/wy).

![](/img/2016/12/004.ue300-tp-link-ethernet-adapter-usb3-interfejs.jpg#big)

Na górnej części obudowy, znajduje się także biała dioda sygnalizująca stan połączenia/zasilania
urządzenia:

![](/img/2016/12/003.ue300-tp-link-ethernet-adapter-usb3-dioda.jpg#big)

### Czip RTL8153

Zgodnie z informacją podaną na stronie TP-LINK, adapter UE300 został wyposażony w czip RTL8153.
Według producenta, ta karta powinna pracować bez większego problemu pod linux'em. Sprawdźmy zatem
czy faktycznie te informacje zostaną potwierdzone. Po podłączeniu karty do portu USB, w logu
systemowym są wypisywane poniższe komunikaty:

    kernel: usb: new SuperSpeed USB device number 2 using xhci-hcd
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0601
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=6
    kernel: usb 2-1.3: Product: USB 10/100/1000 LAN
    kernel: usb 2-1.3: Manufacturer: TP-LINK
    kernel: usb 2-1.3: SerialNumber: 000001000000
    kernel: cdc_ether 2-1.3:2.0 eth0: register 'cdc_ether' at usb-0000:00:1d.0-1.3, CDC Ethernet Device, ec:08:6b:1c:87:0c
    kernel: usbcore: registered new interface driver cdc_ether

Jak widzimy wyżej, karta została rozpoznawana w systemie jako `idVendor=2357` oraz `idProduct=0601`
i obsługiwana jest przez moduł kernela `cdc_ether` . W systemie naturalnie pojawił się nowy
interfejs sieciowy `eth0` i w zasadzie, by korzystać z tego adaptera trzeba jedynie ten interfejs
skonfigurować tak, by została mu przydzielona adresacja IP, a z tym zadaniem nie powinno być już
większego problemu.

Poniżej jest także załączone wyjście polecenia `lsusb` :

    # lsusb -vvv -d 2357:0601

    Bus 004 Device 002: ID 2357:0601
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               3.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0         9
      idVendor           0x2357
      idProduct          0x0601
      bcdDevice           30.00
      iManufacturer           1 TP-LINK
      iProduct                2 USB 10/100/1000 LAN
      iSerial                 6 000001000000
      bNumConfigurations      2
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           57
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower               64mA
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
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst               3
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst               3
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
            bMaxBurst               0
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           98
        bNumInterfaces          2
        bConfigurationValue     2
        iConfiguration          0
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower               64mA
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
            iMacAddress                      3 EC086B1C870C
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
            bMaxBurst               0
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
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst               3
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0400  1x 1024 bytes
            bInterval               0
            bMaxBurst               3
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength           22
      bNumDeviceCaps          2
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x00000002
          Link Power Management (LPM) Supported
      SuperSpeed USB Device Capability:
        bLength                10
        bDescriptorType        16
        bDevCapabilityType      3
        bmAttributes         0x02
          Latency Tolerance Messages (LTM) Supported
        wSpeedsSupported   0x000e
          Device can operate at Full Speed (12Mbps)
          Device can operate at High Speed (480Mbps)
          Device can operate at SuperSpeed (5Gbps)
        bFunctionalitySupport   2
          Lowest fully-functional device speed is High Speed (480Mbps)
        bU1DevExitLat          10 micro seconds
        bU2DevExitLat        2047 micro seconds
    Device Status:     0x0000
      (Bus Powered)

## Praca adaptera UE300 pod linux

Może i adapter UE300 został rozpoznany bez większego problemu na Debianie ale sterownik tej karty
(ten sam co przy UE200) nie daje nam zbytnio dostępu do zaawansowanych opcji konfiguracyjnych:

    # ethtool eth0
    Settings for eth0:
            Current message level: 0x00000007 (7)
                                   drv probe link
            Link detected: no

    # ethtool -s eth0 autoneg off
    Cannot get current device settings: Operation not supported
      not setting autoneg

    # ethtool -i eth0
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

Niestety nie byłem w stanie przetestować faktycznej przepustowości tego adaptera, bo mój laptop nie
ma portów USB 3. A na tych co ma, to maksymalny transfer jaki udało mi się uzyskać był w granicach
250 mbit/s, czyli około 30 MiB/s i to jest graniczna wartość transferu dla portów USB 2, mimo, że
standard USB 2.0 mówi coś o 480 mbit/s. Niemniej jednak, jeśli dysponujemy faktycznym portem USB 3 w
komputerze, to nie powinno być problemów z osiągnięciem tego 1 gbit/s.
