---
author: Morfik
categories:
- Hardware
date: "2016-08-28T14:50:36Z"
date_gmt: 2016-08-28 12:50:36 +0200
published: true
status: publish
tags:
- usb
- recenzja
- tp-link
- hub
GHissueID: 552
title: 'Recenzja: Aktywny HUB USB 3.0 UH720 od TP-LINK'
---

Nasze potrzeby są zawsze nieograniczone i raczej nigdy nie uda się nam ich zaspokoić. Weźmy sobie
dla przykładu urządzenia, które można podpiąć do komputera. Obecnie mamy całą masę sprzętu, a nasz
PC, laptop czy router ma skończoną i często też bardzo niewielką ilość portów USB. Problematyczne
może być zatem podłączenie do komputera w tym samym czasie kamer, pendrive, dysków,
telefonów/smartfonów, mp3player'ów, drukarek, myszy czy klawiatury. Można oczywiście wyciągać z
gniazdek jedne urządzenia, by zrobić miejsce dla innych ale istnieje przecie taki wynalazek jak HUB
USB, który umożliwiają nam podłączone kilku urządzeń do jednego portu USB. Jakość tych HUB'ów oraz
ich właściwości są różne i w tym wpisie zobaczymy co ma nam do zaoferowania[HUB USB 3.0
UH720](http://www.tp-link.com.pl/products/details/UH720.html)od TP-LINK

<!--more-->
## Zawartość opakowania

Zobaczmy sobie na początek jak prezentuje się HUB UH720. Poniżej są fotki opakowania i jego
zawartości:

![hub-usb3-UH270-tp-link-pudelko](/img/2016/08/1.hub-usb3-UH270-tp-link-pudelko.jpg#big)

![hub-usb3-UH270-tp-link-zawartosc-pudelka](/img/2016/08/2.hub-usb3-UH270-tp-link-zawartosc-pudelka.jpg#big)

![hub-usb3-UH270-tp-link-zawartosc-pudelka](/img/2016/08/3.hub-usb3-UH270-tp-link-zawartosc-pudelka.jpg#big)

![hub-usb3-UH270-tp-link-zawartosc-pudelka](/img/2016/08/4.1.hub-usb3-UH270-tp-link-zawartosc-pudelka.jpg#big)

Jak widzimy, nie jest to małe urządzenie. Jego wymiary oscylują w granicach 165 x 65,5 x 17,5 mm. Na
spodzie HUB'a zaś mamy dwie gumowe nóżki zapobiegające jego przemieszczaniu na śliskich
powierzchniach.

![hub-usb3-UH270-tp-link-gumowe-nozki](/img/2016/08/4.2.hub-usb3-UH270-tp-link-gumowe-nozki.jpg#big)

W pudełku poza samym HUB'em USB, mamy jeszcze przewód USB3 (długość około 1 metra), przy pomocy
którego możemy to urządzenie podłączyć do komputera:

![hub-usb3-UH270-tp-link-przewod](/img/2016/08/5.hub-usb3-UH270-tp-link-przewod.jpg#big)

Do zestawu jest także dołączony zasilacz 12V/3,3A:

![hub-usb3-UH270-tp-link-zasilacz](/img/2016/08/6.hub-usb3-UH270-tp-link-zasilacz.jpg#big)

## Porty USB3

HUB UH720 dysponuje siedmioma portami USB3. Zatem mamy możliwość wykorzystania do 5 gbit/s przy
transferze danych z i do urządzeń. Trzeba jednak pamiętać, że faktyczna prędkość zwykle będzie
limitowana przez same urządzenia podpięte do HUB'a.

![hub-usb3-UH270-tp-link-porty](/img/2016/08/7.hub-usb3-UH270-tp-link-porty.jpg#big)

Nad każdym niebieskim gniazdkiem jest malutka dioda, która świeci białym światłem po podłączeniu
urządzenia do HUB'a. Myślałem, że może ta dioda będzie sygnalizować, np. transfer danych ale tak
nie jest. Ta dioda zwyczajnie świeci i nic poza tym.

Zabrakło mi też przełączników na tych portach. Chodzi generalnie o to, by zasilanie w każdym z
portów HUB'a mogło zostać odłączone, co eliminuje potrzebę wyciągania urządzenia z portu USB. W
sumie to tylko z jednym HUB'em się spotkałem, który takie przełączniki oferował ale to naprawdę
użyteczna właściwość tych urządzeń i mogłaby być już implementowana masowo.

Plusem tego HUB'a jest fakt, że można z jego pomocą dość przyzwoicie rozbudować funkcjonalność
routerów z alternatywnym firmware OpenWRT/LEDE. Routery WiFi zwykle dysponują jednym lub dwoma
portami USB, co skutecznie ogranicza ilość urządzeń, które możemy do nich podpiąć. Z HUB'em UH720
raczej ten problem zostanie wyeliminowany na dobre.

## Porty ładujące

Na bocznym panelu HUB'a UH720 mamy dwa dodatkowe porty USB, z tym, że są to typowe porty ładujące.
Umożliwiają one ładowanie urządzeń, które pobierają maksymalnie 2,4 A (smartfony, tablety).

![hub-usb3-UH270-tp-link-porty-ladujace](/img/2016/08/8.hub-usb3-UH270-tp-link-porty-ladujace.jpg#big)

Problem z tymi portami jest taki, że nie damy rady przetransferować przez nie żadnych danych do
komputera. Czyli za pomocą tych portów możemy wprawdzie naładować naszego smartfona ale by przesłać
do niego jakieś pliki, to już trzeba go wpiąć do jednego z pozostałych siedmiu portów USB.

Warto tutaj dodać, że nasz router WiFi za sprawą HUB'a UH720 może zostać również przerobiony na dość
przyzwoitą ładowarkę różnego rodzaju urządzeń. Każdy z tych siedmiu portów jest w stanie ładować
pomniejsze urządzenia pod warunkiem, że przewód USB HUB'a jest podłączony do jakiegoś komputera.
Zatem w sumie możemy mieć do wykorzystania 9 portów ładujących (7x0,9 A i 2x2,4 A).

Te dwa żółte porty są w stanie ładować urządzenia nawet, gdy HUB UH720 nie jest podłączony do
komputera -- wystarczy podpiąć zasilacz.

## Problemy z sieciami WiFi 2,4 GHz

W instrukcji HUB'a UH720 jest informacja, że może on zakłócać sieci WiFi pracujące w paśmie 2,4 GHz.
Dalej czytamy zaś, że nie powinniśmy umieszczać tego urządzenia zbyt blisko bezprzewodowych
nadajników/odbiorników. Więcej informacji na temat [zakłóceń powodowanych przez urządzenia USB 3.0
jest tutaj](http://www.usb.org/developers/docs/whitepapers/327216.pdf), na wypadek, gdyby kogoś ten
temat interesował.

## Zasilanie HUB'a UH720

HUB UH720 może być zasilany z portu USB komputera (tryb pasywny) lub za pomocą zasilacza dołączonego
do zestawu (tryb aktywny). Z tyłu HUB'a UH720 mamy dwa gniazdka. Do jednego podpinamy przewód USB z
wtyczką Micro USB 3.0 Standard B. Do drugiego podłączamy zasilacz:

![hub-usb3-UH270-tp-link-zasilanie](/img/2016/08/9.hub-usb3-UH270-tp-link-zasilanie.jpg#big)

Warto zaznaczyć, że przypadku trybu pasywnego, takie urządzenie jest zasilane z portu USB komputera.
Jest zatem niemal pewne, że przy podpięciu dwóch bardziej prądożernych urządzeń, np. modem LTE i
pendrive, ten HUB nam wysiądzie lub będzie działał bardzo niestabilnie. Dlatego też lepiej jest
wykorzystywać zasilacz. Trzeba jednak wziąć pod uwagę fakt, że nawet przy zastosowaniu tego
zasilacza jesteśmy w pewien sposób ograniczeni.

HUB UH720 jest w stanie prawidłowo pracować pod warunkiem, że maksymalne natężenie prądu urządzeń do
niego podłączonych nie przekroczy 8 A. Port USB3 charakteryzuje się większą wydajnością prądową w
stosunku do portu USB2. W przypadku USB3 jest to 0,9 A, a w przypadku USB2 jest to 0,5 A. Jak łatwo
możemy policzyć, mamy 7 portów USB3, co daje nam 6,3 A. Zatem w przypadku tych niebieskich gniazdek
mieścimy się w normie. Trzeba również brać poprawkę na sprawność przetwornicy w HUB'ie, która zwykle
jest na poziomie 80% i tak z tych 8 A, zostaje nam 6,5 A ale w dalszym ciągu się mieścimy.

Problem natomiast pojawia się w przypadku, gdybyśmy chcieli jeszcze dodatkowo podłączyć jakieś inne
urządzenia do portów ładujących. Każdy z tych portów może wyciągnąć dodatkowe 2,4 A. I już widzimy,
że zasilacz nie wyrobi, bo maksymalny pobór prądu może być na poziomie 11.1 A, a zasilacz jest nam
w stanie dostarczyć jedynie 6,5 A.

Oczywiście to są maksymalne wartości prądowe jakie porty mogą dostarczyć urządzeniom i nie oznacza
to, że każde z urządzeń będzie tyle pobierać. Zwykle pobierają one mniej energii i nie powinniśmy
napotkać problemów przy użytkowaniu tego HUB'a. Niemniej jednak, należy pamiętać, że przy zbyt dużym
obciążeniu tego urządzenia, może ono zachowywać się bardzo nieprzewidywalnie.

W lewym górnym rogu na powierzchni HUB'a UH720 jest ulokowany przycisk power. Działa on tylko w
przypadku podłączenia zasilacza. Przy pomocy tego przycisku, jesteśmy w stanie całkowicie odciąć
zasilanie HUB'a, tj. system operacyjny przestanie nam widzieć to urządzenie. W przypadku, gdy HUB
działa w trybie pasywnym, przycisk nie reaguje w żaden sposób na przyciśnięcie, a dioda, która jest
w przycisku power, święci się cały czas.

![hub-usb3-UH270-tp-link-przycisk-power](/img/2016/08/10.hub-usb3-UH270-tp-link-przycisk-power.jpg#big)

## HUB UH720 pod linux'em

HUB UH720 ma [chipset VIA VL812](http://www.via-labs.com/product_show.php?id=41). Działa on pod
linux'em bez większego problemu i nie są potrzebne żadne dodatkowe sterowniki czy inne
oprogramowanie. Niemniej jednak, kernel rozpoznaje UH720 jako dwa mniejsze HUB'y identyfikowane jako
`idVendor=0bda` i `idProduct=0411` ( lub też `idProduct=5411`) .

Czip VIA VL812 jest zdolny obsłużyć cztery porty USB3 ale HUB UH720 dysponuje siedmioma. W tym
urządzeniu są na dobrą sprawę dwa mniejsze HUB'y czteroportowe (dwa czipy VIA VL812) i kernel
właśnie te dwa urządzenia widzi. Z tym, że jeden z tych HUB'ów jest wpięty do jednego z portów
drugiego HUB'a i w wyniku otrzymujemy do dyspozycji 7 portów USB3.

Po podłączeniu urządzenia do komputera, w logu systemowym pojawiają się poniższe komunikaty:

    kernel: usb 2-1.1: new SuperSpeed USB device number 62 using xhci_hcd
    kernel: usb 2-1.1: New USB device found, idVendor=0bda, idProduct=0411
    kernel: usb 2-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    kernel: usb 2-1.1: Product: 4-Port USB 3.0 Hub
    kernel: usb 2-1.1: Manufacturer: Generic
    kernel: hub 2-1.1:1.0: USB hub found
    kernel: hub 2-1.1:1.0: 4 ports detected
    kernel: usb 2-1.1.1: new SuperSpeed USB device number 63 using xhci_hcd
    kernel: usb 2-1.1.1: New USB device found, idVendor=0bda, idProduct=0411
    kernel: usb 2-1.1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    kernel: usb 2-1.1.1: Product: 4-Port USB 3.0 Hub
    kernel: usb 2-1.1.1: Manufacturer: Generic
    kernel: hub 2-1.1.1:1.0: USB hub found
    kernel: hub 2-1.1.1:1.0: 4 ports detected

Poniżej zaś znajduje się wynik polecenia `lsusb` :

    # lsusb -vvv -d 0bda:0411

    Bus 004 Device 011: ID 0bda:0411 Realtek Semiconductor Corp.
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               3.00
      bDeviceClass            9 Hub
      bDeviceSubClass         0 Unused
      bDeviceProtocol         3
      bMaxPacketSize0         9
      idVendor           0x0bda Realtek Semiconductor Corp.
      idProduct          0x0411
      bcdDevice            1.17
      iManufacturer           1 Generic
      iProduct                2 4-Port USB 3.0 Hub
      iSerial                 0
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           31
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          4 USB3.0 Hub
        bmAttributes         0xe0
          Self Powered
          Remote Wakeup
        MaxPower                0mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         9 Hub
          bInterfaceSubClass      0 Unused
          bInterfaceProtocol      0 Full speed (or root) hub
          iInterface              5 Interrupt In Interface
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes           19
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Feedback
            wMaxPacketSize     0x0002  1x 2 bytes
            bInterval               8
            bMaxBurst               0
    Hub Descriptor:
      bLength              12
      bDescriptorType      42
      nNbrPorts             4
      wHubCharacteristic 0x0009
        Per-port power switching
        Per-port overcurrent protection
      bPwrOn2PwrGood        0 * 2 milli seconds
      bHubContrCurrent      8 milli Ampere
      bHubDecLat          0.2 micro seconds
      wHubDelay          3202 nano seconds
      DeviceRemovable    0x00
     Hub Port Status:
       Port 1: 0000.02a0 5Gbps power Rx.Detect
       Port 2: 0000.02a0 5Gbps power Rx.Detect
       Port 3: 0000.02a0 5Gbps power Rx.Detect
       Port 4: 0000.02a0 5Gbps power Rx.Detect
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength           42
      bNumDeviceCaps          3
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x0000f41e
          Link Power Management (LPM) Supported
      SuperSpeed USB Device Capability:
        bLength                10
        bDescriptorType        16
        bDevCapabilityType      3
        bmAttributes         0x00
        wSpeedsSupported   0x000e
          Device can operate at Full Speed (12Mbps)
          Device can operate at High Speed (480Mbps)
          Device can operate at SuperSpeed (5Gbps)
        bFunctionalitySupport   1
          Lowest fully-functional device speed is Full Speed (12Mbps)
        bU1DevExitLat          10 micro seconds
        bU2DevExitLat        1023 micro seconds
      Container ID Device Capability:
        bLength                20
        bDescriptorType        16
        bDevCapabilityType      4
        bReserved               0
        ContainerID             {e5cdb920-3970-11e0-a935-0002a5d5c51b}
    Device Status:     0x000d
      Self Powered
      U1 Enabled
      U2 Enabled

    Bus 004 Device 010: ID 0bda:0411 Realtek Semiconductor Corp.
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               3.00
      bDeviceClass            9 Hub
      bDeviceSubClass         0 Unused
      bDeviceProtocol         3
      bMaxPacketSize0         9
      idVendor           0x0bda Realtek Semiconductor Corp.
      idProduct          0x0411
      bcdDevice            1.17
      iManufacturer           1 Generic
      iProduct                2 4-Port USB 3.0 Hub
      iSerial                 0
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           31
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          4 USB3.0 Hub
        bmAttributes         0xe0
          Self Powered
          Remote Wakeup
        MaxPower                0mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass         9 Hub
          bInterfaceSubClass      0 Unused
          bInterfaceProtocol      0 Full speed (or root) hub
          iInterface              5 Interrupt In Interface
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes           19
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Feedback
            wMaxPacketSize     0x0002  1x 2 bytes
            bInterval               8
            bMaxBurst               0
    Hub Descriptor:
      bLength              12
      bDescriptorType      42
      nNbrPorts             4
      wHubCharacteristic 0x0009
        Per-port power switching
        Per-port overcurrent protection
      bPwrOn2PwrGood        0 * 2 milli seconds
      bHubContrCurrent      8 milli Ampere
      bHubDecLat          0.2 micro seconds
      wHubDelay          3202 nano seconds
      DeviceRemovable    0x00
     Hub Port Status:
       Port 1: 0000.0263 5Gbps power suspend enable connect
       Port 2: 0000.02a0 5Gbps power Rx.Detect
       Port 3: 0000.02a0 5Gbps power Rx.Detect
       Port 4: 0000.02a0 5Gbps power Rx.Detect
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength           42
      bNumDeviceCaps          3
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x0000f41e
          Link Power Management (LPM) Supported
      SuperSpeed USB Device Capability:
        bLength                10
        bDescriptorType        16
        bDevCapabilityType      3
        bmAttributes         0x00
        wSpeedsSupported   0x000e
          Device can operate at Full Speed (12Mbps)
          Device can operate at High Speed (480Mbps)
          Device can operate at SuperSpeed (5Gbps)
        bFunctionalitySupport   1
          Lowest fully-functional device speed is Full Speed (12Mbps)
        bU1DevExitLat          10 micro seconds
        bU2DevExitLat        1023 micro seconds
      Container ID Device Capability:
        bLength                20
        bDescriptorType        16
        bDevCapabilityType      4
        bReserved               0
        ContainerID             {e5cdb920-3970-11e0-a935-0002a5d5c51b}
    Device Status:     0x000d
      Self Powered
      U1 Enabled
      U2 Enabled
