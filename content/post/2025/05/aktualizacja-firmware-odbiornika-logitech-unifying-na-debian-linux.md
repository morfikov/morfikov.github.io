---
author: Morfik
categories:
- Linux
date:    2025-05-16 18:20:00 +0200
lastmod: 2025-05-16 18:20:00 +0200
published: true
status: publish
tags:
- debian
- klawiatura
- firmware
- logitech
- k350
- unifying
- fwupd
GHissueID: 604
title: Aktualizacja firmware odbiornika Logitech Unifying na Debian Linux
---

Od wielu lat próbowałem rozstać się z moją leciwą już klawiaturą Logitech Media Keyboard Elite,
która działała u mnie od początków przesiadki na linux'a, czyli ponad 15 lat. Nic tej klawiaturze
nie dolega i dalej sprawnie działa ale ma jedną cechę, której nie da się wyeliminować. Mianowicie
jest ona przewodowa. Przez te wszystkie lata próbowałem znaleźć klawiaturę bezprzewodową ale nie
dało się. Takie klawiatury kosztowały blisko 500-1000zł, a te tańsze były pod względem jakości
typowo chińskie. Do tego doszła nowa technologia komplikująca niesamowicie budowę klawiszy, przez co
te klawiatury praktycznie są nierozbieralne, np. na potrzeby czyszczenia. Nie wiem ile razy przez te
prawie 20 lat czyściłem mojego Logitech'a (kompletne rozkręcenie i powyciąganie klawiszy) ale
obstawiam, że co najmniej kilkanaście razy (średnio raz do roku). Z przycisków nawet jedno
oznaczenie się nie starło. Dlatego też miałem ogromne problemy znaleźć coś bezprzewodowego, w miarę
taniego, co by mi posłużyło kolejne dwie dekady. Gdy straciłem już wiarę w producentów klawiatur,
udało mi się za 50zł dorwać klawiaturę (używaną) od Logitech. Jest to [model Logitech K350][1] i
jest on bardzo podobny do tej mojej starej klawiatury, z tą różnicą że jest bezprzewodowa. Do tej
klawiatury jest dostarczony [odbiornik Logitech Unifying][2] i całość w zasadzie działa
bezproblemowo na moim Debianie, choć trzeba było parę rzeczy skonfigurować. Nie obyło się także bez
aktualizacji firmware w tym odbiorniku, bo czynił on to urządzenie podatne na atak typu MouseJack.

<!--more-->

## Odbiornik Logitech Unifying

Zapewne większość z nas ma mysz czy klawiaturę bezprzewodową. Do tych urządzeń z reguły dostarczane
są dedykowane odbiorniki w postaci urządzenia USB, które podłącza się do portu USB. Problem z tym
rozwiązaniem jest taki, że im więcej mamy takich urządzeń, tym więcej potrzeba nam odbiorników, a
więc i portów USB, co może być problematyczne w przypadku laptopów. Ten Logitech Unifying adresuje
właśnie tę niedogodność i można posiadać kilka urządzeń (maksymalnie 6) i tylko jeden odbiornik. Do
tego dochodzi możliwość ręcznego parowania i odparowywania urządzeń, a nie tak jak jest w tych
standardowych odbiornikach, że są one z danym urządzeniem (albo klasą urządzeń) trwale powiązane i
nie da się w tej kwestii nic zmienić.

### Podatność na zdalne ataki

Może i ten cały odbiornik Unifying ma kilka użytecznych funkcji, to jednak nie działa on tak jak
standardowe odbiorniki innych klawiatur czy myszy, tj. wykorzystując Bluetooth. Ten odbiornik od
Logitech korzysta ze swojej własnościowej technologi bezprzewodowej i może stać
się [celem ataków][3]. No i w 2016 roku ogłoszono, że m.in. ten odbiornik Unifying jest podatny
na [atak MouseJack][4]. Oczywiście wypuszczono stosowne aktualizacje firmware ale większość
użytkowników linux'a (czy też i windows'a) może nie być takich rzeczy świadoma w stosunku do
swojego sprzętu albo też nie potrafią firmware tych urządzeń zaktualizować. Dlatego też w dalszej
części artykułu postaramy się ten firmware uaktualnić.

### Konfiguracja kernela

Jeśli mamy dystrybucyjne jądro linux'a, to stosowne moduły do obsługi odbiornika Unifying od
Logitech już są w nim uwzględnione i nic w zasadzie nie trzeba zmieniać czy konfigurować. Niemniej
jednak, w moim przypadku, gdzie sam kompiluję sobie kernel, musiałem włączyć w jego konfiguracji te
poniższe opcje:

    CONFIG_HID_LOGITECH=y
    CONFIG_HID_LOGITECH_DJ=y
    CONFIG_HID_LOGITECH_HIDPP=y

Po podłączeniu odbiornika do portu USB komputera, w logu systemowym powinien nam się pojawić
poniższy komunikat:

    kernel: usb 2-2: new full-speed USB device number 21 using xhci_hcd
    kernel: usb 2-2: New USB device found, idVendor=046d, idProduct=c52b, bcdDevice=12.10
    kernel: usb 2-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    kernel: usb 2-2: Product: USB Receiver
    kernel: usb 2-2: Manufacturer: Logitech
    kernel: usb 2-2: Device is not authorized for usage
    kernel: logitech-djreceiver 0003:046D:C52B.003A: hiddev0,hidraw0: USB HID v1.11 Device [Logitech USB Receiver] on usb-0000:00:14.0-2/input2
    kernel: usb 2-2: authorized to connect
    kernel: input: Logitech K350 as /devices/pci0000:00/0000:00:14.0/usb2/2-2/2-2:1.2/0003:046D:C52B.003A/0003:046D:200A.003B/input/input42
    kernel: logitech-hidpp-device 0003:046D:200A.003B: input,hidraw5: USB HID v1.11 Keyboard [Logitech K350] on usb-0000:00:14.0-2/input2:1
    systemd-logind[2015]: Watching system buttons on /dev/input/event13 (Logitech K350)

Zatem ten odbiornik Unifying ma numerki `idVendor=046d` oraz `idProduct=c52b` .

Poniżej mamy jeszcze wyjście `lsusb` dla tego odbiornika:

    # lsusb -vd 046d:c52b

    Bus 002 Device 022: ID 046d:c52b Logitech, Inc. Unifying Receiver
    Negotiated speed: Full Speed (12Mbps)
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass            0 [unknown]
      bDeviceSubClass         0 [unknown]
      bDeviceProtocol         0
      bMaxPacketSize0         8
      idVendor           0x046d Logitech, Inc.
      idProduct          0xc52b Unifying Receiver
      bcdDevice           12.10
      iManufacturer           1 Logitech
      iProduct                2 USB Receiver
      iSerial                 0
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength       0x0054
        bNumInterfaces          3
        bConfigurationValue     1
        iConfiguration          4 RQR12.10_B0032
        bmAttributes         0xa0
          (Bus Powered)
          Remote Wakeup
        MaxPower               98mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         3 Human Interface Device
          bInterfaceSubClass      1 Boot Interface Subclass
          bInterfaceProtocol      1 Keyboard
          iInterface              0
            HID Device Descriptor:
              bLength                 9
              bDescriptorType        33
              bcdHID               1.11
              bCountryCode            0 Not supported
              bNumDescriptors         1
              bDescriptorType        34 (null)
              wDescriptorLength      59
              Report Descriptors:
                ** UNAVAILABLE **
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0008  1x 8 bytes
            bInterval               8
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         3 Human Interface Device
          bInterfaceSubClass      1 Boot Interface Subclass
          bInterfaceProtocol      2 Mouse
          iInterface              0
            HID Device Descriptor:
              bLength                 9
              bDescriptorType        33
              bcdHID               1.11
              bCountryCode            0 Not supported
              bNumDescriptors         1
              bDescriptorType        34 (null)
              wDescriptorLength     148
              Report Descriptors:
                ** UNAVAILABLE **
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0008  1x 8 bytes
            bInterval               2
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        2
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         3 Human Interface Device
          bInterfaceSubClass      0 [unknown]
          bInterfaceProtocol      0
          iInterface              0
            HID Device Descriptor:
              bLength                 9
              bDescriptorType        33
              bcdHID               1.11
              bCountryCode            0 Not supported
              bNumDescriptors         1
              bDescriptorType        34 (null)
              wDescriptorLength      93
              Report Descriptors:
                ** UNAVAILABLE **
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0020  1x 32 bytes
            bInterval               2
    Device Status:     0x0000
      (Bus Powered)

### Oprogramowanie do obsługi odbiornika pod linux

Do obsługi odbiornika Logitech Unifying mamy kilka natywnych linux'owych narzędzi. Jeśli
potrzebujemy graficznego interfejsu, to doinstalujmy sobie pakiet `solaar` ([strona projektu][5]) .
Jeśli wystarczy nam opcja tekstowa, to instalujemy `ltunify` . Poniżej aplikacja `solaar` z
wykrytym odbiornikiem Unifying oraz sparowaną klawiaturą Logitech K350:

|   |   |
|---|---|
|![solaar-logitech-unifying](/img/2025/05/001-solaar-logitech-unifying.png#medium) | ![solaar-logitech-k350-keyboard](/img/2025/05/002-solaar-logitech-k350-keyboard.png#medium) |

Oprogramowania `solaar` można także używać z poziomu terminala:

    $ solaar -dd show
    2025-05-10 14:17:12,428,428     INFO [MainThread] hidapi.udev_impl: Found device /dev/hidraw0 BID 0003 VID 0000046D PID 0000C52B HID++ True True USB 2 2
    2025-05-10 14:17:12,429,429     INFO [MainThread] hidapi.udev_impl: OPEN PATH /dev/hidraw0
    2025-05-10 14:17:12,429,429     INFO [MainThread] logitech_receiver.base: New lock 9
    solaar version 1.1.14-4

    Unifying Receiver
      Device path  : /dev/hidraw0
      USB id       : 046d:C52B
      Serial       : 5AD29F67
      C Pending    : ff
        0          : 12.03.B0025
        1          : 02.15
        3          : AA.AA
      Has 1 paired device(s) out of a maximum of 6.
      Notifications: wireless (0x000100)
      Device activity counters: 1=73
    2025-05-10 14:17:12,628,628     INFO [MainThread] hidapi.udev_impl: OPEN PATH /dev/hidraw5
    2025-05-10 14:17:12,629,629     INFO [MainThread] logitech_receiver.receiver: <UnifyingReceiver(/dev/hidraw0,9)>: found new device 1 (200A)

    2025-05-10 14:17:12,629,629     INFO [MainThread] logitech_receiver.base: New lock 10
      1: Wireless Wave Keyboard K350
         Device path  : /dev/hidraw5
         WPID         : 200A
         Codename     : K350
         Kind         : keyboard
         Protocol     : HID++ 1.0
         Report Rate : 20ms
         Serial number: DE95F0EB
                     0: 65.00.B0003
                     3: 00.01
         The power switch is located on the base.
         Notifications: battery status (0x100000).
         Features: (none)
         Battery: 90%, 0.

## Aktualizacja firmware w odbiorniku Logitech Unifying

W moim przypadku, odbiornik Logitech Unifying miał wersje firmware `RQR12.03_B0025` i ta wersja
firmware jest według autorów ataku MouseJack podatna. Zatem nie ma innej opcji jak zaktualizować
firmware w tym odbiorniku, jeśli chcemy bezpiecznie korzystać ze sprawowanych z nim urządzeń.

By zaktualizować firmware w takich dedykowanych urządzeniach podłączanych do naszego komputera
będziemy potrzebować stosowne oprogramowanie, tj. musimy zainstalować pakiet `fwupd` . Po
zainstalowaniu tego pakietu odpalamy terminal i korzystając z `fwupdmgr` wydajemy poniższe
polecenia:

    # fwupdmgr refresh --force
    Updating lvfs
    Downloading…             [************************************** ]
    Successfully downloaded new metadata: Updates have been published for 1 local device

    # fwupdmgr get-updates
    Devices with no available firmware updates:
     • UEFI dbx
     • BCM20702 Bluetooth 4.0 [ThinkPad]
     • IR-SSDPR-S25A-240
     • WD10JPVX-00JC3T0
     • morfikov's kernel-signing key
     • morfikov's key-exchange-key
    LENOVO 2349BM5
    │
    └─Unifying Receiver:
      │   Device ID:          fdb5c2807386a6014025a352dec974ac08d4ab84
      │   Summary:            Miniaturised USB wireless receiver
      │   Current version:    RQR12.03_B0025
      │   Bootloader Version: BOT01.02_B0015
      │   Vendor:             Logitech, Inc. (USB:0x046D, HIDRAW:0x046D)
      │   Install Duration:   30 seconds
      │   GUIDs:              aa995882-da42-574a-8338-c8dfdca447e3 ← UFY\VID_046D&PID_C52B
      │                       279ed287-3607-549e-bacc-f873bb9838c4 ← HIDRAW\VEN_046D&DEV_C52B
      │                       9d131a0c-a606-580f-8eda-80587250b8d6 ← USB\VID_046D&PID_AAAA
      │   Device Flags:       • Updatable
      │                       • Supported on remote server
      │                       • Unsigned Payload
      │                       • Can tag for emulation
      │
      ├─Unifying USB Receiver Update:
      │     New version:      RQR12.10_B0032
      │     Remote ID:        lvfs
      │     Release ID:       3581
      │     Summary:          Firmware for the Logitech Unifying Receiver (RQR12.xx)
      │     Variant:          RQR12
      │     License:          Proprietary
      │     Size:             56.8 kB
      │     Created:          2019-07-18
      │     Urgency:          High
      │       Tested:         2024-07-05
      │       Distribution:   fedora 40 (workstation)
      │       Old version:    RQR12.10_B0032
      │       Version[fwupd]: 1.9.19
      │       Tested:         2024-04-15
      │       Distribution:   chromeos 125
      │       Old version:    RQR12.01_B0019
      │       Version[fwupd]: 1.9.15
      │     Vendor:           Logitech
      │     Duration:         30 seconds
      │     Release Flags:    • Trusted metadata
      │                       • Is upgrade
      │     Description:
      │     This release addresses an encrypted keystroke injection vulnerability sent by pointing devices.The vulnerability is complex to replicate and would require a hacker to be physically close to a target.
      │
      │     A few of Logitech's devices used to send select buttons in an unencrypted way, and in an effort to protect against this vulnerability, Logitech removed the feature.Affected hardware is:
      │
      │     • Wireless Mouse M335
      │     • Zone Touch Mouse T400
      │     • Wireless Mouse M545
      │     • Wireless Mouse M560
      │     • Touch Mouse M600
      │     • Touch Mouse T620
      │     • Wireless Rechargeable Touchpad T650
      │
      │     Although Logitech does not recommend it, these features may be re-activated by keeping/downgrading the receiver to an older firmware.
      │     Checksum:         be1b52aa9e112c8f237a517a668da0991a7cd64c7d121c66edab621d4253356f
      │
      ├─Unifying USB Receiver Update:
      │     New version:      RQR12.08_B0030
      │     Remote ID:        lvfs
      │     Release ID:       2438
      │     Summary:          Firmware for the Logitech Unifying receiver
      │     Variant:          RQR12
      │     License:          Proprietary
      │     Size:             72.7 kB
      │     Created:          2017-06-26
      │     Urgency:          High
      │     Vendor:           Logitech
      │     Duration:         30 seconds
      │     Release Flags:    • Trusted metadata
      │                       • Is upgrade
      │     Description:
      │     This release addresses an encrypted keystroke injection issue known as Bastille security issue #13.The vulnerability is complex to replicate and would require a hacker to be physically close to a target.
      │
      │     A few of Logitech's devices used to send select buttons in an unencrypted way, and in an effort to protect against this vulnerability, Logitech removed the feature.Affected hardware is:
      │
      │     • Wireless Mouse M335
      │     • Zone Touch Mouse T400
      │     • Wireless Mouse M545
      │     • Wireless Mouse M560
      │     • Touch Mouse M600
      │     • Touch Mouse T620
      │     • Wireless Rechargeable Touchpad T650
      │
      │     Although Logitech does not recommend it, these features may be re-activated by keeping/downgrading the receiver to an older firmware.
      │     Checksum:         c956adf983164fc77c65a69b8a09a76b5f42cfe86750aa1635059385c147a3e9
      │
      └─Unifying USB Receiver Update:
            New version:      RQR12.07_B0029
            Remote ID:        lvfs
            Release ID:       194
            Summary:          Firmware for the Logitech Unifying receiver
            Variant:          RQR12
            License:          Proprietary
            Size:             69.6 kB
            Created:          2017-05-03
            Urgency:          High
            Vendor:           Logitech
            Duration:         30 seconds
            Release Flags:    • Trusted metadata
                              • Is upgrade
            Description:
            This release addresses an unencrypted keystroke injection issue known as Bastille security issue #11.The vulnerability is complex to replicate and would require a hacker to be physically close to a target.
            Checksum:         dea915e48f53e0d8187599e6814de961e5766d9006ba644e78312e9dafb481b5

Jak widzimy wyżej, do tego odbiornika Unifying zostały wypuszczone 3 aktualizacje,
tj. `RQR12.07_B0029` , `RQR12.08_B0030` oraz `RQR12.10_B0032` . By załatać podatność MouseJack,
potrzebny nam firmware co najmniej `RQR12.08.00030` .

By zaktualizować firmware, wpisujemy w terminal poniższe polecenie:

    # fwupdmgr update
    ╔══════════════════════════════════════════════════════════════════════════════╗
    ║ Upgrade Unifying Receiver from RQR12.03_B0025 to RQR12.10_B0032?             ║
    ╠══════════════════════════════════════════════════════════════════════════════╣
    ║ This release addresses an encrypted keystroke injection vulnerability sent   ║
    ║ by pointing devices.The vulnerability is complex to replicate and would      ║
    ║ require a hacker to be physically close to a target.                         ║
    ║                                                                              ║
    ║ A few of Logitech's devices used to send select buttons in an unencrypted    ║
    ║ way, and in an effort to protect against this vulnerability, Logitech        ║
    ║ removed the feature.Affected hardware is:                                    ║
    ║                                                                              ║
    ║ • Wireless Mouse M335                                                        ║
    ║ • Zone Touch Mouse T400                                                      ║
    ║ • Wireless Mouse M545                                                        ║
    ║ • Wireless Mouse M560                                                        ║
    ║ • Touch Mouse M600                                                           ║
    ║ • Touch Mouse T620                                                           ║
    ║ • Wireless Rechargeable Touchpad T650                                        ║
    ║                                                                              ║
    ║ Although Logitech does not recommend it, these features may be re-activated  ║
    ║ by keeping/downgrading the receiver to an older firmware.                    ║
    ║                                                                              ║
    ║ Unifying Receiver and all connected devices may not be usable while          ║
    ║ updating.                                                                    ║
    ╚══════════════════════════════════════════════════════════════════════════════╝
    Perform operation? [Y|n]: Y

Wyżej widzimy, że firmware zostanie zaktualizowany z `RQR12.03_B0025` do `RQR12.10_B0032` . Po
procesie aktualizacji sprawdzamy jaką wersję ma nasz odbiornik:

    #  fwupdmgr get-devices
    ...
    LENOVO 2349BM5
    │
    └─Unifying Receiver:
          Device ID:          fdb5c2807386a6014025a352dec974ac08d4ab84
          Summary:            Miniaturised USB wireless receiver
          Current version:    RQR12.10_B0032
          Bootloader Version: BOT01.02_B0015
          Vendor:             Logitech, Inc. (USB:0x046D, HIDRAW:0x046D)
          Install Duration:   30 seconds
          GUIDs:              aa995882-da42-574a-8338-c8dfdca447e3 ← UFY\VID_046D&PID_C52B
                              279ed287-3607-549e-bacc-f873bb9838c4 ← HIDRAW\VEN_046D&DEV_C52B
                              9d131a0c-a606-580f-8eda-80587250b8d6 ← USB\VID_046D&PID_AAAA
          Device Flags:       • Updatable
                              • Supported on remote server
                              • Unsigned Payload
                              • Can tag for emulation
    ...

Widzimy, że `Current version:` wskazuje na `RQR12.10_B0032` , czyli podatność została załatana.

## Podsumowanie

Przy zakupie urządzeń bezprzewodowych, pokroju mysz czy klawiatura, trzeba zdawać sobie sprawę, że
mogą być one narażone na zdalne ataki z racji iż te sprzęty komunikują się ze swoimi odbiornikami
za pomocą Bluethooth, WiFi czy innych własnościowych bezprzewodowych protokołów wymiany danych. Na
przykładzie tej używanej klawiatury Logitech K350 i jej odbiornika Unifying widzimy, że użytkownicy
nie zawsze są tego faktu świadomi i nie wiedzą o tym, że firmware (odbiornika i/lub klawiatury)
trzeba regularnie aktualizować. Dobrą wiadomością dla użytkowników linux'a jest fakt, że nawet dla
tych alternatywnych dla windows systemach istnieje dedykowane oprogramowanie `fwupd` , które jest
nam w stanie bez większego problemu zaktualizować firmware urządzeń naszego komputera.


[1]: https://www.logitech.com/en-us/eol/keyboards-eol/k350-keyboard.920-001996.html
[2]: https://www.logitech.com/pl-pl/resource-center/what-is-unifying.html
[3]: https://en.wikipedia.org/wiki/Logitech_Unifying_receiver#Security
[4]: https://www.bastille.net/research/vulnerabilities-mousejack/
[5]: https://github.com/pwr-Solaar/Solaar
