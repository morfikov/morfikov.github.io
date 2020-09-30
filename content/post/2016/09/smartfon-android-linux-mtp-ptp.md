---
author: Morfik
categories:
- Linux
date: "2016-09-22T21:54:23Z"
date_gmt: 2016-09-22 19:54:23 +0200
published: true
status: publish
tags:
- debian
- smartfon
- mtp
title: Smartfon z Androidem pod linux'em (MTP/PTP)
---

Wpadł mi w łapki [smartfon Neffos C5](http://www.neffos.pl/product/details/C5) od TP-LINK, który ma
na pokładzie Androida. Chciałem nim zrobić parę fotek, tylko pojawił się problem uzyskania dostępu
do zasobów tego telefonu. Samo urządzenie pod linux'em identyfikowane jest jako `idVendor=2357` oraz
`idProduct=0314` ale po jego podłączeniu do portu USB komputera nie pojawiły się żadne nowe dyski,
które można by przejrzeć w celu zgania ich zawartości. Problem tkwił w konfiguracji mojego Debiana,
w którym to brakowało obsługi protokołu MTP. Po chwili rozgryzłem tę zagadkę instalując w systemie
pakiet `jmtpfs` , co umożliwiło interakcję z systemem plików telefonu i zgranie zrobionych zdjęć.

<!--more-->
## USB Mass Storage i protokół MTP/PTP

W przypadku starszych telefonów i smartfonów była wykorzystywana pamięć masowa USB (USB Mass
Storage) w celu przesyłania plików z i do komputera. Problem z tym rozwiązaniem był taki, że tylko
jedno urządzenie mogło z tej pamięci korzystać w danym czasie. W efekcie, gdy podłączaliśmy telefon
do komputera, to linux operował na zasobach urządzenia i nie mogliśmy z nimi wejść w interakcję z
poziomu telefonu. Obecnie smartfony korzystają z protokółu MTP ([Media Transfer
Protocol](https://en.wikipedia.org/wiki/Media_Transfer_Protocol)) lub PTP ([Picture Transfer
Protocol](https://en.wikipedia.org/wiki/Picture_Transfer_Protocol)).

Jako, że MTP operuje na poziomie plików, to nie ma potrzeby wyłączności w dostępie do zasobów.
System operacyjny komputera zwyczajnie wysyła żądania do smartfona z prośbą o konkretny plik. Po
zaakceptowaniu takiej prośby, plik jest przesyłany i zapisywany na dysku naszego komputera. Podobnie
w drugą stronę, gdzie to nasz linux chce przesłać plik na telefon, z tym, że wtedy to system
telefonu (w tym przypadku Android) decyduje, czy odebrać taki plik i gdzie go zapisać. Jeśli zaś
chcemy skasować plik z telefonu, to wysyłamy jedynie prosty sygnał, który Android interpretuje jako
"skasuj plik xyz" i plik ulega skasowaniu.

By wiedzieć, jakimi plikami dysponuje smartfon, po podłączeniu go do komputera jest przesyłana lista
wszystkich udostępniony przez telefon plików i katalogów. Trzeba jednak pamiętać, że część
plików/katalogów może być przed nami ukryta i w efekcie ich nie zobaczymy na liście. Podobnie może
być z prawami do modyfikacji konkretnych plików i tak dla przykładu nie wszystkie z nich będziemy w
stanie skasować czy poddać edycji.

Niewątpliwą zaletą protokołu MTP jest wyeliminowanie potrzeby wspierania systemu plików przez
systemy operacyjne. Chodzi o to, że pamięć masowa USB wymaga od naszego linux'a, by ten wpierał
system plików, który na takim urządzeniu się znajduje. W przypadku, gdy nasz OS nie wspiera takiego
systemu plików, to nie może wejść z nim w interakcję. Dlatego z reguły na pendrive czy innych
pamięciach flash wykorzystuje się dość ograniczony system plików FAT, bo jest on wspierany przez
wszystkie systemy operacyjne.

W przypadku smartfonów, gdzie rozmiary kart SD i pamięci telefonu mogą iść już w setki GiB, trzeba
korzystać z innego systemu plików, np. EXT4, czy NTFS. Z tym, że znowu wybór każdego z tych systemów
plików będzie powodował problemy czy to pod windowsem, czy też pod linux'em. Protokół MTP uwalnia
nas od potrzeby wspierania systemu plików smartfona, z którym zamierzamy wymieniać dane, mniej
więcej na takiej samej zasadzie jak odbywa się nasza komunikacja z serwerami HTTP czy FTP.

## Brak zasobów smartfona pod linux'em

W opcjach Androida możemy sobie wybrać czy zamierzamy korzystać z protokołu MTP czy też i PTP. W tym
celu wystarczy podłączyć smartfon do portu USB komputera. Na ekranie smartfona powinna nam się
pojawić poniższa wiadomość:

![](/img/2016/09/1smartfon-linux-protokol-mtp-ptp-podlaczanie-usb.png#medium)

Jak widzimy wyżej, jeśli nasz komputer nie wspiera protokołu MTP, to wybieramy PTP. Linux'y na
szczęście nie mają problemów z protokołem MTP i dlatego to on został wyżej zaznaczony.

Po podłączeniu smartfona do portu USB, za wiele w logu systemowym nie zobaczymy. Mi pojawiły się
jedynie te poniższe komunikaty:

    kernel: usb 2-1.3: new high-speed USB device number 69 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0314
    kernel: usb 2-1.3: New USB device strings: Mfr=2, Product=3, SerialNumber=4
    kernel: usb 2-1.3: Product: Neffos
    kernel: usb 2-1.3: Manufacturer: TP-LINK
    kernel: usb 2-1.3: SerialNumber: TSL7DA69OBSO49PJ

Niemniej jednak, gdy rzucimy okiem bezpośrednio na to urządzenie przez `lsusb` , to już mamy tam
informację o jakimś interfejsie MTP:

    # lsusb -vvv -d 2357:0314

    Bus 002 Device 069: ID 2357:0314
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x2357
      idProduct          0x0314
      bcdDevice           ff.ff
      iManufacturer           2 TP-LINK
      iProduct                3 Neffos
      iSerial                 4 TSL7DA69OBSO49PJ
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           39
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xc0
          Self Powered
        MaxPower              500mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           3
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass    255 Vendor Specific Subclass
          bInterfaceProtocol      0
          iInterface             17 MTP
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
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x001c  1x 28 bytes
            bInterval               6
    Device Qualifier (for other device speed):
      bLength                10
      bDescriptorType         6
      bcdUSB               2.00
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      bNumConfigurations      1
    Device Status:     0x0001
      Self Powered

Zatem smartfon chce komunikować się z naszym linux'em za pomocą protokołu MTP, z tym, że ten system
jeszcze nie ma zaimplementowanej obsługi tego protokołu (brak odpowiednich pakietów).

## Wsparcie Debiana dla protokołu MTP (libmtp9 i mtp-tools)

[Jest sporo narzędzi](https://wiki.debian.org/mtp), które umożliwiają linux'om operowanie na
protokole MTP. Jeśli nie korzystamy z żadnych środowisk graficznych, to jest niemal pewne, że nie
mamy w systemie wymaganych pakietów. W Debianie, wsparcie dla tego protokołu jest dostarczane z
pakietem `libmtp9` . Do tego są również narzędzia zawarte w pakiecie `mtp-tools` . W zasadzie to te
pakiety wystarczą, by możliwa była interakcja z telefonem.

Po instalacji ww. pakietów, w systemie będziemy mieli dostęp do szeregu poleceń `mtp-*` . Teraz
wystarcz podpiąć telefon do portu USB i wpisać w terminalu `mtp-detect` :

    $ mtp-detect
    libmtp version: 1.1.12

    Listing raw device(s)
    Device 0 (VID=2357 and PID=0314) is UNKNOWN in libmtp v1.1.12.
    Please report this VID/PID and the device model to the libmtp development team
       Found 1 device(s):
       2357:0314 @ bus 2, dev 69
    Attempting to connect device(s)
    Android device detected, assigning default bug flags
    USB low-level info:
       bcdUSB: 512
       bDeviceClass: 0
       bDeviceSubClass: 0
       bDeviceProtocol: 0
       idVendor: 2357
       idProduct: 0314
       IN endpoint maxpacket: 512 bytes
       OUT endpoint maxpacket: 512 bytes
       Raw device info:
          Bus location: 2
          Device number: 69
          Device entry info:
             Vendor: (null)
             Vendor id: 0x2357
             Product: (null)
             Vendor id: 0x0314
             Device flags: 0x18008106
    Configuration 0, interface 0, altsetting 0:
       Interface description contains the string "MTP"
       Device recognized as MTP, no further probing.
    Device info:
       Manufacturer: TP-LINK
       Model: Neffos C5
       Device version: 1.0
       Serial number: TSL7DA69OBSO49PJ
       Vendor extension ID: 0x00000006
       Vendor extension description: microsoft.com: 1.0; android.com: 1.0;
       Detected object size: 64 bits
       Extensions:
            microsoft.com: 1.0
            android.com: 1.0
    Supported operations:
       1001: Unknown(1001)
       1002: Unknown(1002)
       1003: Unknown(1003)
       1004: Unknown(1004)
       1005: Unknown(1005)
       1006: Unknown(1006)
       1007: Unknown(1007)
       1008: Unknown(1008)
       1009: Unknown(1009)
       100a: Unknown(100a)
       100b: Unknown(100b)
       100c: Unknown(100c)
       100d: Unknown(100d)
       1014: Unknown(1014)
       1015: Unknown(1015)
       1016: Unknown(1016)
       1017: Unknown(1017)
       101b: Unknown(101b)
       9801: Unknown(9801)
       9802: Unknown(9802)
       9803: Unknown(9803)
       9804: Unknown(9804)
       9805: Unknown(9805)
       9810: Unknown(9810)
       9811: Unknown(9811)
       95c1: Unknown(95c1)
       95c2: Unknown(95c2)
       95c3: Unknown(95c3)
       95c4: Unknown(95c4)
       95c5: Unknown(95c5)
    Events supported:
       0x4002
       0x4003
       0x4004
       0x4005
       0x4006
       0x4007
       0x400c
    Device Properties Supported:
       0xd401: Synchronization Partner
       0xd402: Friendly Device Name
       0x5003: Image Size
       0x5001: Battery Level
    Playable File (Object) Types and Object Properties Supported:
       3000: Undefined Type
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       3001: Association/Directory
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       3004: Text
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       3005: HTML
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       3008: MS Wave
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc9b: Album Artist STRING data type READ ONLY
          dc8b: Track UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc99: Original Release Date STRING data type DATETIME FORM READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc8c: Genre STRING data type READ ONLY
          dc96: Composer STRING data type READ ONLY
          de99: Audio WAVE Codec UINT32 data type ANY 32BIT VALUE form READ ONLY
          de92: Bit Rate Type UINT16 data type enumeration: 1, 2,  READ ONLY
          de9a: Audio Bit Rate UINT32 data type range: MIN 1, MAX 1536000, STEP 1 READ ONLY
          de94: Number Of Channels UINT16 data type enumeration: 1, 2, 3, 4, 5, 6, 7, 8, 9,  READ ONLY
          de93: Sample Rate UINT32 data type range: MIN 8000, MAX 48000, STEP 1 READ ONLY
       3009: MP3
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc9b: Album Artist STRING data type READ ONLY
          dc8b: Track UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc99: Original Release Date STRING data type DATETIME FORM READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc8c: Genre STRING data type READ ONLY
          dc96: Composer STRING data type READ ONLY
          de99: Audio WAVE Codec UINT32 data type ANY 32BIT VALUE form READ ONLY
          de92: Bit Rate Type UINT16 data type enumeration: 1, 2,  READ ONLY
          de9a: Audio Bit Rate UINT32 data type range: MIN 1, MAX 1536000, STEP 1 READ ONLY
          de94: Number Of Channels UINT16 data type enumeration: 1, 2, 3, 4, 5, 6, 7, 8, 9,  READ ONLY
          de93: Sample Rate UINT32 data type range: MIN 8000, MAX 48000, STEP 1 READ ONLY
       300b: MPEG
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc48: Description STRING data type READ ONLY
       3801: JPEG
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc48: Description STRING data type READ ONLY
       3802: TIFF EP
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       3804: BMP
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc48: Description STRING data type READ ONLY
       3807: GIF
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc48: Description STRING data type READ ONLY
       3808: JFIF
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       380b: PNG
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc48: Description STRING data type READ ONLY
       380d: TIFF
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       b901: WMA
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc9b: Album Artist STRING data type READ ONLY
          dc8b: Track UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc99: Original Release Date STRING data type DATETIME FORM READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc8c: Genre STRING data type READ ONLY
          dc96: Composer STRING data type READ ONLY
          de99: Audio WAVE Codec UINT32 data type ANY 32BIT VALUE form READ ONLY
          de92: Bit Rate Type UINT16 data type enumeration: 1, 2,  READ ONLY
          de9a: Audio Bit Rate UINT32 data type range: MIN 1, MAX 1536000, STEP 1 READ ONLY
          de94: Number Of Channels UINT16 data type enumeration: 1, 2, 3, 4, 5, 6, 7, 8, 9,  READ ONLY
          de93: Sample Rate UINT32 data type range: MIN 8000, MAX 48000, STEP 1 READ ONLY
       b902: OGG
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc9b: Album Artist STRING data type READ ONLY
          dc8b: Track UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc99: Original Release Date STRING data type DATETIME FORM READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc8c: Genre STRING data type READ ONLY
          dc96: Composer STRING data type READ ONLY
          de99: Audio WAVE Codec UINT32 data type ANY 32BIT VALUE form READ ONLY
          de92: Bit Rate Type UINT16 data type enumeration: 1, 2,  READ ONLY
          de9a: Audio Bit Rate UINT32 data type range: MIN 1, MAX 1536000, STEP 1 READ ONLY
          de94: Number Of Channels UINT16 data type enumeration: 1, 2, 3, 4, 5, 6, 7, 8, 9,  READ ONLY
          de93: Sample Rate UINT32 data type range: MIN 8000, MAX 48000, STEP 1 READ ONLY
       b903: AAC
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc9b: Album Artist STRING data type READ ONLY
          dc8b: Track UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc99: Original Release Date STRING data type DATETIME FORM READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc8c: Genre STRING data type READ ONLY
          dc96: Composer STRING data type READ ONLY
          de99: Audio WAVE Codec UINT32 data type ANY 32BIT VALUE form READ ONLY
          de92: Bit Rate Type UINT16 data type enumeration: 1, 2,  READ ONLY
          de9a: Audio Bit Rate UINT32 data type range: MIN 1, MAX 1536000, STEP 1 READ ONLY
          de94: Number Of Channels UINT16 data type enumeration: 1, 2, 3, 4, 5, 6, 7, 8, 9,  READ ONLY
          de93: Sample Rate UINT32 data type range: MIN 8000, MAX 48000, STEP 1 READ ONLY
       b982: MP4
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       b983: MP2
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       b984: 3GP
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
          dc46: Artist STRING data type READ ONLY
          dc9a: Album Name STRING data type READ ONLY
          dc89: Duration UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc48: Description STRING data type READ ONLY
       ba05: Abstract Audio Video Playlist
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       ba10: WPL Playlist
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       ba11: M3U Playlist
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       ba14: PLS Playlist
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       ba82: XMLDocument
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
       b906: FLAC
          dc01: Storage ID UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc02: Object Format UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc03: Protection Status UINT16 data type ANY 16BIT VALUE form READ ONLY
          dc04: Object Size UINT64 data type READ ONLY
          dc07: Object File Name STRING data type GET/SET
          dc09: Date Modified STRING data type DATETIME FORM READ ONLY
          dc0b: Parent Object UINT32 data type ANY 32BIT VALUE form READ ONLY
          dc41: Persistant Unique Object Identifier UINT128 data type READ ONLY
          dc44: Name STRING data type READ ONLY
          dce0: Display Name STRING data type READ ONLY
          dc4e: Date Added STRING data type DATETIME FORM READ ONLY
    Storage Devices:
       StorageID: 0x00010001
          StorageType: 0x0003 fixed RAM storage
          FilesystemType: 0x0002 generic hierarchical
          AccessCapability: 0x0000 read/write
          MaxCapacity: 10362871808
          FreeSpaceInBytes: 8785158144
          FreeSpaceInObjects: 1073741824
          StorageDescription: Pamięć telefonu
          VolumeIdentifier: (null)
       StorageID: 0x00020001
          StorageType: 0x0004 removable RAM storage
          FilesystemType: 0x0002 generic hierarchical
          AccessCapability: 0x0000 read/write
          MaxCapacity: 1968508928
          FreeSpaceInBytes: 62480384
          FreeSpaceInObjects: 1073741824
          StorageDescription: Karta SD
          VolumeIdentifier: (null)
    Special directories:
       Default music folder: 0xffffffff
       Default playlist folder: 0xffffffff
       Default picture folder: 0xffffffff
       Default video folder: 0xffffffff
       Default organizer folder: 0xffffffff
       Default zencast folder: 0xffffffff
       Default album folder: 0xffffffff
       Default text folder: 0xffffffff
    MTP-specific device properties:
       Friendly name: Neffos C5
       Synchronization partner: Neffos C5
       Battery level 41 of 100 (40%)
    libmtp supported (playable) filetypes:
       Folder
       Text file
       HTML file
       RIFF WAVE file
       ISO MPEG-1 Audio Layer 3
       MPEG video stream
       JPEG file
       BMP bitmap file
       GIF bitmap file
       JFIF file
       Portable Network Graphics
       TIFF bitmap file
       Microsoft Windows Media Audio
       Ogg container format
       Advanced Audio Coding (AAC)/MPEG-2 Part 7/MPEG-4 Part 3
       MPEG-4 Part 14 Container Format (Audio+Video Emphasis)
       ISO MPEG-1 Audio Layer 2
       Abstract Playlist file
       XML file
       Free Lossless Audio Codec (FLAC)

Powyżej został wykryty smartfon. Niemniej jednak, mamy tam informację, że `libmtp` nie zna tego
urządzenia ( `UNKNOWN` ). Jeśli mamy podobny problem, to wypadałoby zgłosić numery urządzenia
(VID=2357 i PID=0314) [do developerów](https://sourceforge.net/p/libmtp/feature-requests/). Te
powyższe zostały właśnie zgłoszone. W międzyczasie możemy napisać regułę dla UDEV'a wzorując się na
pliku `/lib/udev/rules.d/69-libmtp.rules` . Pamiętajmy tylko, by nową regułę umieścić w pliku
`/etc/udev/rules.d/69-libmtp.rules` . W przeciwnym razie, przy aktualizacji UDEV'a, zmiany w pliku
`/lib/udev/rules.d/69-libmtp.rules` zostaną nadpisane. Poniżej reguła dla Neffos'a C5:

    # TP-LINK NEFFOS C5
    ATTR{idVendor}=="2357", ATTR{idProduct}=="0314", SYMLINK+="libmtp-%k", MODE="660", GROUP="audio", ENV{ID_MTP_DEVICE}="1", ENV{ID_MEDIA_PLAYER}="1"

Jako, że pakiet `mtp-tools` zawiera całą masę poleceń `mtp-*` , to można się trochę pogubić w
operowaniu na nich. Na szczęście nie musimy się nimi bawić, by wejść w interakcję z naszym
smartfonem.

## Pakiety mtpfs oraz jmtpfs

Jeśli operowanie na tych wszystkich narzędziach zawartych w pakiecie `mtp-tools` jest dla nas trochę
przytłaczające, to zawsze możemy skorzystać z automatu na bazie
[FUSE](https://pl.wikipedia.org/wiki/FUSE). Mamy z grubsza dwa pakiety `mtpfs` oraz `jmtpfs` . Są
one w gruncie rzeczy podobne do siebie ale `mtpfs` został osierocony na początku 2014 roku. Biorąc
pod uwagę, że [ostatnia wersja mtpfs](https://www.adebenham.com/mtpfs/) jest datowana na rok 2010,
to raczej nie ma sobie co tym pakietem głowy zawracać. Z kolei jeśli chodzi o `jmtpfs` , to jest on
nieco nowszy ale i tak [ostatni commit pojawił się trzy lata
temu](https://github.com/JasonFerrara/jmtpfs). Grunt, że ciągle ktoś się tym projektem interesuje
oraz, że on działa. Dlatego też jeśli już chcemy instalować któryś z powyższych pakietów, to
zdecydujmy się na `jmtpfs` .

Musimy jeszcze sobie nieco dostosować konfigurację FUSE w pliku `/etc/fuse.conf` . Otwieramy go
zatem i dopisujemy poniższą linijkę, która zezwala użytkownikom innym niż root na korzystanie z
opcji `allow_other` przy montowaniu zasobów:

    user_allow_other

## Montowanie zasobów smartfona

W przypadku, gdy mamy podpięty do komputera tylko jeden smartfon, to korzystanie z narzędzia
`jmtpfs` znacznie się upraszcza. Nie musimy określać urządzenia za pomocą opcji `-device` . Po
prostu wybrane zostanie pierwsze urządzenie na liście, którą z kolei możemy podejrzeć podając
parametr `-l` . Poniżej jest lista urządzeń, które są w stanie komunikować się z moim systemem za
pomocą protokołu MTP:

    $ jmtpfs -l
    Device 0 (VID=2357 and PID=0314) is UNKNOWN in libmtp v1.1.12.
    Available devices (busLocation, devNum, productId, vendorId, product, vendor):
    2, 71, 0x0314, 0x2357, UNKNOWN, UNKNOWN

Jako, że mamy tylko jedno urządzenie, to możemy zamontować jego zasoby wpisując w terminalu to
poniższe polecenie:

    $ jmtpfs /home/morfik/Desktop/android
    Device 0 (VID=2357 and PID=0314) is UNKNOWN in libmtp v1.1.12.
    Android device detected, assigning default bug flags

W przypadku, gdy tych urządzeń byłoby więcej i chcielibyśmy skorzystać z drugiego z tych, które
widnieją na liście, to powyższe polecenie przybierze poniższą formę:

    $ jmtpfs -device=2,71 /home/morfik/Desktop/android/

W tej chwili powinniśmy uzyskać dostęp do systemu plików urządzenia, zarówno do karty SD jak i do
pamięci telefonu:

    $ mount
    ...
    fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
    jmtpfs on /home/morfik/Desktop/android type fuse.jmtpfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

Gdy skończymy zgrywać/wgrywać dane ze/na smartfona, system plików możemy odmontować w poniższy
sposób:

    $ fusermount -u /home/morfik/Desktop/android/

## Przeglądanie zasobów za sprawą protokołu mtp://

Możemy również pominąć etap montowania systemu plików i zwyczajnie przeglądać zasoby bezpośrednio na
urządzeniu korzystając z adresu `mtp://[USB:001,002]/` w menadżerze plików. Oczywiście menadżer
plików musi wspierać protokół MTP, inaczej nic z tego nie będzie. Numerki widoczne wyżej można
odczytać z wyjścia polecenia `lsusb` lub też są one uwzględnione w informacji zwracanej przez
polecenie `jmtpfs -l` :

    $ jmtpfs -l
    ...
    2, 71, 0x0314, 0x2357, UNKNOWN, UNKNOWN

W tym przypadku zostały zwrócone numerki `2` oraz `71` , zatem adres, który musimy wpisać w
menadżerze plików to `mtp://[usb:002,071]/` . Niestety mój menadżer plików nie obsługuje tego typu
adresów. Można natomiast posłużyć się [narzędziem gmtp](https://gmtp.sourceforge.io/):

![](/img/2016/09/2.smartfon-linux-protokol-mtp-ptp-gmtp.png#huge)

I jeszcze info o urządzeniu zwracane przez `gmtp` :

![](/img/2016/09/3.smartfon-linux-protokol-mtp-ptp-gmtp.png#huge)

## Problemy z przeglądaniem zasobów smartfona

Może się zdarzyć też tak, że zasoby smartfona zostaną podmontowane bez większego problemu ale, gdy
tylko będziemy chcieli przejść do punktu montowania, to zostanie nam zwrócony taki oto błąd:

    $ ls -al /home/morfik/Desktop/android/
    ls: cannot access '/home/morfik/Desktop/android/': Input/output error

W niektórych smartfonach ekran musi być odblokowany. Dlatego też zanim podepniemy telefon do
komputera, upewnijmy się, że blokada ekranu została dezaktywowana.
