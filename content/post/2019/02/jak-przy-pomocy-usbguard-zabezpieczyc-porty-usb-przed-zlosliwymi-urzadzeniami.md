---
author: Morfik
categories:
- Linux
date: "2019-02-24T12:00:33Z"
published: true
status: publish
tags:
- debian
- usb
- modem
- huawei
- e3372
GHissueID: 311
title: Jak przy pomocy USBguard zabezpieczyć porty USB przed złośliwymi urządzeniami
---

Ostatnio na
Niebezpieczniku [pojawił się artykuł](https://niebezpiecznik.pl/post/zlosliwy-kabel-usb-ktory-zmienia-sie-w-klawiature-i-infekuje-twoj-komputer/)
na temat niezbyt przyjaznych urządzeń podłączanych do komputera za sprawą portów USB (opisanych na
przykładzie niepozornego przewodu) i tego jaką szkodę tego typu hardware może nam wyrządzić w
systemie. Ataki z wykorzystaniem podstawionych urządzeń zadziałają nawet na linux, choć pewnie cała
masa użytkowników wyznaje jeszcze mit, że ich komputer jest bezpieczny, bo przecie używają
alternatywnego systemu operacyjnego, który jest OpenSource i za priorytet obrał sobie szeroko
rozumiane bezpieczeństwo. Niestety nie jest tak różowo jakby mogło się co niektórym wydawać ale
można ten stan rzeczy naturalnie zmienić i nie trzeba przy tym rekompilować kernela z zamiarem
wyłączenia obsługi modułu USB, co ten opisany w podlinkowanym artykule atak oczywiście by również
powstrzymało. Zamiast tego możemy zainstalować sobie narzędzie `usbguard` i przy jego pomocy
skonfigurować politykę podłączanych do portów USB urządzeń.

<!--more-->
## Dlaczego urządzenia USB mogą być takie groźne

Generalnie rzecz biorąc, to przy podłączaniu jakiegokolwiek urządzenia do portu USB, nasz system
komunikuje się z tym urządzeniem w celu ustalenia pewnych informacji umożliwiających wstępną
identyfikację sprzętu by dobrać dla niego odpowiedni sterownik. Gdy ten proces się zakończy,
odpowiedni moduł kernela jest ładowany (zwykle automatycznie, o ile jest dostępny) i po krótkiej
chwili urządzenie jest gotowe do pracy
(taki [PlugAndPlay](https://pl.wikipedia.org/wiki/Plug_and_play)). Wszystko fajnie, gdy w grę
wchodzą urządzenia, które nie mają złowrogich zamiarów. Niemniej jednak, podłączając jakiś trefny
hardware, to on również zostanie skonfigurowany i będzie w stanie się komunikować z naszym systemem.

Do tej automatycznej konfiguracji sprzętu dochodzi jeszcze jedna rzecz. Obecnie proces
miniaturyzacji jest na tyle zaawansowany, że zaszycie w takim niepozornym urządzeniu dodatkowych
modułów sprzętowych nie stanowi większego wyzwania. Z naszej perspektywy jest to tylko zwykły
przewód czy pendrive ale pod maską taki kawałek hardware może być nafaszerowany elektroniką i do
póki tego urządzenia nie rozbierzemy na czynniki pierwsze, to też możemy nawet tego faktu nigdy nie
być do końca świadomi. Weźmy np. modemy LTE. Niby mają modem GSM ale też można do nich podłączyć
kartę SD, która bez problemu zostanie w systemie wykryta jako osobne urządzenie blokowe (obok
modemu). Niby podłączyliśmy jedno urządzenie, a w systemie zostanie ich wykrytych kilka i każde z
nich zostanie skonfigurowane osobno i będzie wykorzystywać inny moduł kernela. Wszystkie te
informacje są zawarte w konfiguracji zwracanej przez urządzenie USB, po tym jak system operacyjny
zażąda od urządzenia identyfikacji. Poniżej przykład wyjścia polecenia `lsusb` :

    #  lsusb -vvv -d 12d1:15b6
    Bus 002 Device 012: ID 12d1:15b6 Huawei Technologies Co., Ltd.
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.10
      bDeviceClass            0
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x12d1 Huawei Technologies Co., Ltd.
      idProduct          0x15b6
      bcdDevice            1.02
      iManufacturer           1 HUAWEI_MOBILE
      iProduct                2 HUAWEI_MOBILE
      iSerial                 3 0123456789ABCDEF
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength       0x00c6
        bNumInterfaces          4
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xc0
          Self Powered
        MaxPower                2mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      3
          bInterfaceProtocol     18
          iInterface              0
          ** UNRECOGNIZED:  05 24 00 10 01
          ** UNRECOGNIZED:  04 24 02 02
          ** UNRECOGNIZED:  05 24 01 00 00
          ** UNRECOGNIZED:  05 24 06 00 00
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
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       0
          bNumEndpoints           3
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      3
          bInterfaceProtocol     16
          iInterface              0
          ** UNRECOGNIZED:  05 24 00 10 01
          ** UNRECOGNIZED:  04 24 02 02
          ** UNRECOGNIZED:  05 24 01 00 01
          ** UNRECOGNIZED:  05 24 06 00 00
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x000a  1x 10 bytes
            bInterval               9
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
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
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        2
          bAlternateSetting       0
          bNumEndpoints           1
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      3
          bInterfaceProtocol     22
          iInterface              7 NCM Network Control Model
          ** UNRECOGNIZED:  05 24 00 10 01
          ** UNRECOGNIZED:  06 24 1a 00 01 1f
          ** UNRECOGNIZED:  0d 24 0f 09 0f 00 00 00 ea 05 03 00 01
          ** UNRECOGNIZED:  05 24 06 02 02
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x85  EP 5 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0010  1x 16 bytes
            bInterval               5
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        2
          bAlternateSetting       1
          bNumEndpoints           3
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass      3
          bInterfaceProtocol     22
          iInterface              8 CDC Network Data
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x85  EP 5 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0010  1x 16 bytes
            bInterval               5
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x84  EP 4 IN
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
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        3
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass         8 Mass Storage
          bInterfaceSubClass      6 SCSI
          bInterfaceProtocol     80 Bulk-Only
          iInterface             11 Mass Storage
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x86  EP 6 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
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
    Binary Object Store Descriptor:
      bLength                 5
      bDescriptorType        15
      wTotalLength       0x0016
      bNumDeviceCaps          2
      USB 2.0 Extension Device Capability:
        bLength                 7
        bDescriptorType        16
        bDevCapabilityType      2
        bmAttributes   0x00000002
          HIRD Link Power Management (LPM) Supported
      SuperSpeed USB Device Capability:
        bLength                10
        bDescriptorType        16
        bDevCapabilityType      3
        bmAttributes         0x00
        wSpeedsSupported   0x000f
          Device can operate at Low Speed (1Mbps)
          Device can operate at Full Speed (12Mbps)
          Device can operate at High Speed (480Mbps)
          Device can operate at SuperSpeed (5Gbps)
        bFunctionalitySupport   1
          Lowest fully-functional device speed is Full Speed (12Mbps)
        bU1DevExitLat           1 micro seconds
        bU2DevExitLat         500 micro seconds
    can't get debug descriptor: Resource temporarily unavailable
    Device Status:     0x0001
      Self Powered

Kernel linux'a te wszystkie informacje zwrócone wyżej jest w stanie uzyskać zanim dobierze moduł
dla tego urządzenia. Dokładna rozpiska co każda z tych opcji oznacza
jest [dostępna tutaj](https://www.beyondlogic.org/usbnutshell/usb5.shtml). Te najważniejsze rzeczy,
które nas powinny zainteresować, to przede wszystkim `bNumConfigurations` określająca ilość
możliwych konfiguracji, które ten sprzęt wspiera. Zwykle będzie tutaj wartość `1` , bo przy
większej liczbie konfiguracji rośnie skomplikowalność modułu i co za tym idzie problematyczne staje
się wsparcie dla tego urządzenia w danym systemie operacyjnym. Dalej interesuje nas
`bNumInterfaces` , którego wartość określa liczbę interfejsów urządzenia (taki interfejs to w
zasadzie osobny ficzer, np. karta SD). W tym przypadku jest 5 interfejsów (numerowane od `0` ).
Każdy z interfejsów posiada identyfikator w postaci trzech składowych: `bInterfaceClass` ,
`bInterfaceSubClass` oraz `bInterfaceProtocol` . Wartości tych trzech parametrów są w stanie
zidentyfikować konkretny moduł, np. czy mamy na pokładzie urządzenia moduł klawiatury, co
mogłoby wzbudzać już podejrzenia w przypadku pendrive, który powinien mieć w zasadzie tylko jeden
interfejs wskazujący na "USB mass storage". Analizując w ten sposób interfejsy urządzenia USB,
możemy wyłapać pewne nieścisłości i określić z dużym prawdopodobieństwem czy takie urządzenie może
działać na nasza szkodę. W przytoczonym wyżej przykładzie mamy poniższe interfejsy:

    Interface Descriptor:
            ...
            Interface Descriptor:
              bInterfaceNumber        0
              bAlternateSetting       0
              ...
              bInterfaceClass       255 Vendor Specific Class
              bInterfaceSubClass      3
              bInterfaceProtocol     18
              iInterface              0
            ...
            Interface Descriptor:
            ...
              bInterfaceNumber        1
              bAlternateSetting       0
              ...
              bInterfaceClass       255 Vendor Specific Class
              bInterfaceSubClass      3
              bInterfaceProtocol     16
              iInterface              0
            ...
            Interface Descriptor:
            ...
              bInterfaceNumber        2
              bAlternateSetting       0
              ...
              bInterfaceClass       255 Vendor Specific Class
              bInterfaceSubClass      3
              bInterfaceProtocol     22
              iInterface              7 NCM Network Control Model
            ...
            Interface Descriptor:
            ...
              bInterfaceNumber        2
              bAlternateSetting       1
              ...
              bInterfaceClass       255 Vendor Specific Class
              bInterfaceSubClass      3
              bInterfaceProtocol     22
              iInterface              8 CDC Network Data
            ...
            Interface Descriptor:
            ...
              bInterfaceNumber        3
              bAlternateSetting       0
              ...
              bInterfaceClass         8 Mass Storage
              bInterfaceSubClass      6 SCSI
              bInterfaceProtocol     80 Bulk-Only
              iInterface             11 Mass Storage
            ...

Jeśli się uważnie przyjrzymy, to mamy dwa interfejsy z numerkiem `2` . W przypadku tych interfejsów
w grę wchodzi jeszcze atrybut `bAlternateSetting` , który w przypadku pierwszego z nich ma wartość
`0` , a w przypadku drugiego wartość `1` . Oznacza, to, że te dwa interfejsy mogą się przełączać w
locie.

Wartości, które widać wyżej, są w zapisie dziesiętnym. Niemniej jednak, pewne narzędzia (np.
`usbguard` ) używają zapisu HEX. Dla przykładu identyfikator interfejsów tego urządzenia wyglądałby
następująco: `ff:03:12 ff:03:10 ff:03:16 ff:03:16 08:06:50` . Interfejsy zaczynające się od `255`
( `FF` ) są specyficzne dla producenta sprzętu (Vendor Specific). Ostatni interfejs zaś obsługuje
kartę SD, którą można wsadzić do dedykowanego gniazda w tym modemie LTE.

Gdy mamy jakieś zaufane urządzenie to interfejsy w nim powinno się dać łatwo zidentyfikować. Będą
one także takie same za każdym razem przy podłączaniu tego urządzenia do portów USB komputera.
Każda zmiana tych interfejsów, np. pojawienie się nowego, może oznaczać podmianę urządzenia i z
wierzchu może ono nawet łudząco przypominać nasz ulubiony pendrive ale w środku może się on już
znacznie różnić. Niemniej jednak, standardowo linux skonfiguruje każde urządzenie, które się do
niego podłączy, oczywiście jeśli będzie miał odpowiedni sterownik i wypadałoby oduczyć nasz system
tego zachowania.

### Moduły kernela wykorzystywane przez interfejsy USB

Jeśli mamy zastrzeżenia co do interfejsów naszego urządzenia USB, to zawsze możemy sprawdzić jakie
moduły dobierze mu kernel. Oczywiście lepiej jest to robić z poziomu jakiegoś live cd/dvd/pendrive,
a nie z głównego systemu. Na systemie live powinno być dostępne polecenie `usb-devices` (w pakiecie
`usbutils` ). Zwróci nam ono szereg informacji na temat urządzeń USB podpiętych aktualnie do portów
USB, przykładowo:

    # usb-devices
    ...
    T:  Bus=02 Lev=02 Prnt=02 Port=02 Cnt=03 Dev#= 28 Spd=480 MxCh= 0
    D:  Ver= 2.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
    P:  Vendor=12d1 ProdID=15b6 Rev=01.02
    S:  Manufacturer=HUAWEI_MOBILE
    S:  Product=HUAWEI_MOBILE
    S:  SerialNumber=0123456789ABCDEF
    C:  #Ifs= 4 Cfg#= 1 Atr=c0 MxPwr=2mA
    I:  If#=0x0 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=03 Prot=12 Driver=option
    I:  If#=0x1 Alt= 0 #EPs= 3 Cls=ff(vend.) Sub=03 Prot=10 Driver=option
    I:  If#=0x2 Alt= 1 #EPs= 3 Cls=ff(vend.) Sub=03 Prot=16 Driver=huawei_cdc_ncm
    I:  If#=0x3 Alt= 0 #EPs= 2 Cls=08(stor.) Sub=06 Prot=50 Driver=usb-storage

Linijki mające `I` to interfejsy. Może i było ich 5 ale wykorzystywanych w danym czasie są tylko 4.
Interfejsów w tym modemie LTE
może [być naturalnie więcej lub mniej](/post/konfiguracja-modemu-lte-w-trybie-ndis-ncm/).
Wszystko zależy od jego konfiguracji (polecenie `AT^SETPORT=` ). Każdy włączony port w modemie
będzie miał osobny interfejs USB z pasującym numerkiem. Przykładowo, ten modem obsługuje poniższe
porty:

	^SETPORT:3: 3G DIAG
	^SETPORT:10: 4G MODEM
	^SETPORT:1: 3G MODEM
	^SETPORT:12: 4G PCUI
	^SETPORT:13: 4G DIAG
	^SETPORT:5: 3G GPS
	^SETPORT:14: 4G GPS
	^SETPORT:A: BLUE TOOTH
	^SETPORT:16: NCM
	^SETPORT:A1: CDROM
	^SETPORT:A2: SD

Numerki, które tutaj widać, pasują do `Prot=` zwróconym w wyjściu polecenia `usb-devices` (za
wyjątkiem tych mających `A` ) . Widać zatem, że porty `4G MODEM` oraz `4G PCUI` są obsługiwane
przez moduł kernela `option` , natomiast port `NCM` jest obsługiwany przez moduł `huawei_cdc_ncm` ,
a port `SD` przez moduł `usb-storage` .

Zagadkowy może się wydawać moduł `option` i mi w zasadzie  to on nic ze swojej nazwy nie powiedział.
Trochę informacji o tym module można wyciągnąć z `modinfo option`
ale [ostatecznie okazało](https://superuser.com/questions/691271/what-does-modprobe-option-do) się,
że ten moduł `option` istnieje, bo standardowy sterownik szeregowy dostępny w linux nie działa za
dobrze z modemami GSM i powoduje parę problemów (opisanych w pliku
`kernel-git/linux-kernel/drivers/usb/serial/option.c` ).

## USBguard

Jako, że podłączane do portu USB urządzenie musi pierw się zidentyfikować, to jesteśmy w
stanie [powstrzymać kernel linux'a od konfiguracji sprzętu](https://www.kernel.org/doc/Documentation/usb/authorization.txt)
niewiadomego pochodzenia, przez co nie będzie on miał nawet okazji poczynić jakiejkolwiek szkody w
systemie. By zatrzymać proces automatycznej konfiguracji sprzętu USB, musimy zaprzęgnąć do pracy
jakiegoś demona, który będzie monitorował podłączane urządzenia USB i weryfikował je pod względem
zaufania. Takim demonem jest [USBuard](https://usbguard.github.io/), który w Debianie jest zawarty
w pakiecie noszącym tę samą nazwę. Dodatkowo, by ułatwić nieco konfigurację polityki urządzeń,
dobrze jest zainstalować sobie również pakiet `usbguard-applet-qt` .

Zasada mechanizmu opartego o `usbguard` jest prosta. Domyślnie wszystkie urządzenia USB będą
traktowane jako niezaufane (lepiej zawczasu zrobić stosowne reguły dla klawiatury i myszy, by
czasem nie odciąć sobie dostępu do systemu). Po włączeniu mechanizmu bezpieczeństwa, `usbguard`
przepisze wartość plików  `/sys/bus/usb/devices/*/authorized _default` na `0` . Taki stan rzeczy
sprawi, że nowe urządzenia USB z automatu nie będą mogły zostać skonfigurowane. Po podłączeniu
nowego sprzętu, w logu zostanie zarejestrowany jedynie poniższy komunikat:

    kernel: usb 2-1.3: new high-speed USB device number 13 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=12d1, idProduct=15b6, bcdDevice= 1.02
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: HUAWEI_MOBILE
    kernel: usb 2-1.3: Manufacturer: HUAWEI_MOBILE
    kernel: usb 2-1.3: SerialNumber: 0123456789ABCDEF
    kernel: usb 2-1.3: Device is not authorized for usage

Ostatnia linijka świadczy dobitnie, że to urządzenie nie zostało dopuszczone do użytku. Natomiast
mamy wyżej trochę informacji, które możemy podać w `lsusb` i przejrzeć konfigurację urządzenia,
które właśnie podłączyliśmy do portu USB.

Dodatkowo, demon `usbguard` zwróci nam coś na wzór poniższego logu:

    usbguard-daemon[77129]: uid=0 pid=77127 device.rule='block id 12d1:15b6 serial "0123456789ABCDEF"
      name "HUAWEI_MOBILE" hash "at8BIODSI/yrJm+T+kx7pJCwnBO0+bLymnA0okqGYJk="
      parent-hash "oDU77vx1EsfYlDoXkU7iWjsvmBNCDNTcCHp/V0hIFXc=" via-port "2-1.3"
      with-interface { ff:03:12 ff:03:10 ff:03:16 ff:03:16 08:06:50 }' type='Device.Insert'
      result='SUCCESS' device.system_name='/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3'

    usbguard-daemon[77129]: uid=0 pid=77127 result='SUCCESS'
      device.system_name='/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3' target.new='block'
      type='Policy.Device.Update' device.rule='block id 12d1:15b6 serial "0123456789ABCDEF"
      name "HUAWEI_MOBILE" hash "at8BIODSI/yrJm+T+kx7pJCwnBO0+bLymnA0okqGYJk="
      parent-hash "oDU77vx1EsfYlDoXkU7iWjsvmBNCDNTcCHp/V0hIFXc=" via-port "2-1.3"
      with-interface { ff:03:12 ff:03:10 ff:03:16 ff:03:16 08:06:50 }' target.old='block'

    usbguard-daemon[77129]: Ignoring unknown UEvent action:
      sysfs_devpath=/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3 action=bind

Wartości atrybutów z tego logu są w stanie zidentyfikować konkretne urządzenie USB podłączane do
systemu. Jak widać wyżej, urządzenie USB można dopasować na wiele sposobów. Można używać
indywidualnych pól, np. `id` czy `serial` , można też je łączyć zacieśniając dość mocno politykę
bezpieczeństwa.

Większość z atrybutów, które się pojawiły w logu, raczej powinna być zrozumiała ale wypadałoby
poruszyć kwestię hash'ów. W logu mamy `hash` i `parent-hash` . Te hash'e są wyliczane z wartości
atrybutów urządzenia oraz danych deskryptora USB. Jeśli zachodzi potrzeba, by dane urządzenie było
podłączone tylko i wyłącznie do konkretnego portu USB, to `parent-hash` jest nam w stanie to
zadanie umożliwić. Jeśli chodzi zaś o interfejsy, to `usbguard` traktuje każdy zestaw interfejsów
jako indywidualne urządzenie. Jeśli szereg wartości identyfikujących urządzenie nie zmieniłby się
ale zamiast tych pięciu interfejsów było by ich cztery, to wtedy trzeba by dodać jeszcze jedną
regułę zawierającą tylko te cztery interfejsy.

Za każdym razem jak tylko demon `usbguard` uzna (w oparciu o politykę narzuconą przez reguły), że
dane urządzenie jest mu znane i powinno być dopuszczone do użytku, to wpisze `1` w pliku
`/sys/bus/usb/devices/*/authorized` .

## Konfiguracja demona usbguard

Politykę bezpieczeństwa portów USB w systemie możemy skonfigurować na kilka różnych sposobów. Całe
to zadanie sprowadza się w zasadzie do edycji pliku `/etc/usbguard/usbguard-daemon.conf` . Nie jest
on jakoś szczególnie rozbudowany, dlatego też wrzucę go tutaj w całości, a niżej opiszę
poszczególne parametry:

    RuleFile=/etc/usbguard/rules.conf
    ImplicitPolicyTarget=block
    PresentDevicePolicy=apply-policy
    PresentControllerPolicy=apply-policy
    InsertedDevicePolicy=apply-policy
    RestoreControllerDeviceState=false
    DeviceManagerBackend=uevent
    IPCAllowedUsers=root morfik
    IPCAllowedGroups=root
    IPCAccessControlFiles=/etc/usbguard/IPCAccessControl.d/
    DeviceRulesWithPort=false
    AuditBackend=FileAudit
    AuditFilePath=/var/log/usbguard/usbguard-audit.log

- `RuleFile=` -- definiuje ścieżkę do pliku, w którym będą przechowywane reguły dla urządzeń USB.
- `ImplicitPolicyTarget=` -- określa domyślną politykę dla nowych urządzeń (tych niemających
reguły w pliku określonym przez `RuleFile=` ) -- do wyboru `allow` , `block` lub `reject` . W
przypadku wybrania `reject` urządzenie nie tylko zostanie zablokowane ale również usunięte z
systemu (kernel linux'a przestanie je widzieć), tak samo jakbyśmy je zwyczajnie odłączyli od portu
USB, mimo, że fizycznie ono dalej tam będzie. By z takim urządzeniem wejść ponownie w interakcję
trzeba będzie je odłączyć i ponownie podłączyć do portu USB.
- `PresentDevicePolicy=` oraz `PresentControllerPolicy=` -- określają jak traktować urządzenia i
kontrolery USB, które są aktualnie podłączone do systemu w momencie, gdy demon `usbguard` jest
uruchamiany. Przykładem może być faza startu systemu, gdzie pierw są wykrywane urządzenia, a
dopiero później są startowane usługi (w tym `usbguard` ). W takim przypadku trzeba podjąć decyzję
co zrobić z tymi urządzeniami USB, które sobie już działają w systemie. Do wyboru mamy `allow` ,
`block` , `reject` oraz jeszcze są `keep` i `apply-policy` . W przypadku wybrania tej ostatniej
wartości, urządzenia zostaną sprawdzone pod kątem pliku reguł i jeśli któreś nie będzie miało
stosownej reguły, to zostanie podjęta akcja określona w `ImplicitPolicyTarget=` .
- `InsertedDevicePolicy=` -- określa co robić z urządzeniami USB, które zostaną odłączone i
ponownie podłączone (lub też podłączone w późniejszym czasie) do systemu (po uruchomieniu demona
`usbguard`). Do wyboru są `block` , `reject` oraz `apply-policy ` . Gdy ustawimy tutaj `block` lub
`reject` , to każde wykryte urządzenie (bez znaczenia czy ma ono regułę w pliku określonym przez
`RuleFile=` ) nie będzie już mogło zostać na nowo skonfigurowane w systemie. Jeśli zatem odłączymy
klawiaturę, to już jej ponownie nie będziemy w stanie podłączyć. Dobrze jest tutaj ustawić
`apply-policy` . Inne wartości mogą mieć zastosowanie,
np. [przy zablokowaniu ekranu](https://usbguard.github.io/blog/2017/Screen-Locking), gdzie raczej
nie spodziewamy się podłączania nowego sprzętu.
- `RestoreControllerDeviceState=` -- określa czy `usbguard` ma próbować odtworzyć wartości
atrybutów kontrolerów urządzeń, np. domyślny stan autoryzacji dla nowych instancji urządzeń
potomnych, przed zamknięciem systemu. Nie zaleca się ustawienie tutaj wartości `true` , bo może to
prowadzić do obejścia zdefiniowanych reguł polityki bezpieczeństwa.
- `DeviceManagerBackend=` -- definiuje backend menadżera urządzeń. Do wyboru `uevent` oraz
`umockdev` . Domyślnie jest ustawiony ten pierwszy.
- `IPCAllowedUsers=` , `IPCAllowedGroups=` oraz `IPCAccessControlFiles=` -- konfigurują połączenia
IPC (międzyprocesowe). Powinno się określić choć jednego użytkownika (lub grupę), który takie
połączenia z demonem `usbguard` może nawiązywać. Jeśli się nie określi ani użytkownika ani grupy,
to wtedy połączenia IPC z demonem będzie w stanie nawiązywać każdy użytkownik w systemie, co
zagraża polityce bezpieczeństwa.
- `DeviceRulesWithPort=` -- określa czy generowane reguły powinny zawierać atrybut `via-port` . Nie
zaleca się ustawiania tutaj wartości `true` , bo numerowanie portów w linux nie jest zbyt stabilne
i po ponownym uruchomieniu systemu numeracja portów może ulec zmianie, co zablokuje dostęp do
urządzeń.
- `AuditBackend=` oraz `AuditFilePath=` -- konfigurują backend audytu. W `AuditBackend=` można
określić `FileAudit` lub `LinuxAudit` . Jeśli zostanie wybrany ten pierwszy, to logi powędrują do
pliku wskazanego w `AuditFilePath=` . Jeśli zaś wybierze się `LinuxAudit` , to logi będą
rejestrowane przy pomocy podsystemu Linux Audit.

## Generowanie polityki dla USBguard

Mając skonfigurowanego demona `usbguard` oraz wiedząc gdzie szukać informacji o urządzeniu USB,
możemy przejść do pisania reguł polityki bezpieczeństwa. Na początek dobrze jest zacząć od
wygenerowania tych reguł, które pasują do podłączonego już do komputera sprzętu USB. W tym celu
wystarczy wpisać w terminal to poniższe polecenie:

    # usbguard generate-policy

Wyjście, które uzyskamy, zapisujemy w pliku `/etc/usbguard/rules.conf` . Te reguły są maksymalnie
dopasowane do konkretnych urządzeń podłączonych do określonych portów USB w komputerze. Jeśli
chcemy nieco poluzować politykę bezpieczeństwa, to trzeba będzie poddać ręcznej edycji ten plik i
pousuwać/pozmieniać konkretne rzeczy. Dla przykładu, reguła od tego modemu LTE wygląda tak (reguła
zawinięta dla lepszej czytelności):

    allow id 12d1:15b6 serial "0123456789ABCDEF" name "HUAWEI_MOBILE"
      hash "at8BIODSI/yrJm+T+kx7pJCwnBO0+bLymnA0okqGYJk="
      parent-hash "oDU77vx1EsfYlDoXkU7iWjsvmBNCDNTcCHp/V0hIFXc="
      with-interface { ff:03:12 ff:03:10 ff:03:16 ff:03:16 08:06:50 }

Ten `parent-hash` można usunąć jeśli nie zależy nam na tym, by ten modem był podłączany tylko do
tego konkretnego portu USB.

### Graficzna nakładka usbguard-applet-qt

Reguły można pisać ręcznie poddając edycji plik `/etc/usbguard/rules.conf` lub też można posłużyć
się graficzną nakładką `usbguard-applet-qt` . Ta nakładka jednak wymaga, by skonfigurować
użytkowników lub grupę dla IPC, umożliwiając tym samym komunikację interfejsowi GUI z demonem
`usbguard` uruchomionym z prawami konkretnego użytkownika w systemie.

W tym przypadku tylko użytkownik `root` i `morfik` oraz członkowie grupy `root` będą w stanie
wysyłać zapytania do demona `usbguard` . Jeśli odpalimy teraz `usbguard-applet-qt` , to w
przypadku wykrycia nowego sprzętu powiadomi on nas o tym fakcie:

![](/img/2019/02/001.usbguard-usb-device-linux-debian.png#medium)

Nazwa urządzenia, jego numery identyfikacyjne, serial oraz aktualnie udostępniane interfejsy są
widoczne i można łatwo ustawić czy to urządzenie jest tym, za które się faktycznie podaje. Możemy
zezwolić systemowi na skonfigurowanie tego urządzenia lub też zablokować ten proces. Przy braku
akcji z naszej strony, urządzenie zostanie zablokowane automatycznie. Jeśli mamy pewność, że dane
urządzenie nie poczyni nam szkód w systemie, to możemy dodać wyjątek na stałe.

Wszystkie aktualnie podłączone do komputera urządzenia USB można podejrzeć w głównym okienku
`usbguard-applet-qt` :

![](/img/2019/02/002.usbguard-usb-device-list-linux-debian.png#huge)

`Target` możemy w dowolnej chwili zmienić, a zaznaczając przy aplikowaniu ustawień opcję
`Permanently` , `usbguard-applet-qt` będzie w stanie automatycznie uzupełniać plik
`/etc/usbguard/rules.conf` odpowiednimi wartościami atrybutów. Naturalnie w dalszym ciągu
przydałoby się po każdej takiej zmianie przejrzeć ten plik i pousuwać z niego zbędne atrybuty.

## Czy porty USB są już bezpieczne

Mając odpowiednio skonfigurowanego demona `usbguard` i dodane reguły dla każdego znanego nam
urządzenia USB możemy czuć się jedynie względnie bezpieczni. O ile każde nowe urządzenie będzie
wymagać od nas podjęcia akcji czy zezwolić mu na dostęp do systemu czy też nie, to w dalszym ciągu
trzeba będzie za każdym razem weryfikować interfejsy urządzenia w wyjściu polecenia `lsusb` . Jeśli
nie będziemy tego robić, to prędzej czy później możemy trafić na podstawione urządzenie, a gdy
dodamy mu jeszcze stosowną regułę, to całą politykę bezpieczeństwa szlag trafi.

Trzeba także pamiętać o fazie startu systemu, gdzie na początku są wykrywane wszystkie urządzenia,
w tym też te USB, a dopiero po chwili startuje demon `usbguard` . Przez ten krótki moment nasz
system nie jest chroniony przez politykę bezpieczeństwa, którą sobie skonfigurowaliśmy wyżej.
Dlatego też trzeba zwracać uwagę na to co siedzi w portach USB zanim uruchomimy komputer. Warto też
w tym miejscu wspomnieć o konieczności wykorzystania pełnego szyfrowania dysku (FDE), bo bez niego
to takie zabezpieczenie jak `usbguard` jest raczej pozbawione sensu.

Ewidentnym plusem `usbguard` jest fakt, że nikt bez naszej wiedzy nie podłączy już do działającego
systemu jakiegoś trefnego urządzenia. Dodatkowo, również złowroga pokojówka nie podepnie do tak
chronionego kompa pendrive i nie zgra na niego naszych prywatnych danych w celu późniejszego
szantażowania nas. No i oczywiście nikt nic nie wgra nam do systemu z takiego dysku USB, bo żadna
dodatkowa pamięć masowa nie zostanie skonfigurowana. Wiem, że każdy z nas blokuje ekran jak tylko
odchodzi od kompa ale czasem można o tym albo zapomnieć, albo też zlekceważyć zagrożenie, bo
przecież "poszedłem tylko do WC na parę sekund". W dalszym jednak ciągu można te dane wyprowadzić
przez łącze internetowe (no chyba, że ktoś zaimplementował sobie linux'owy firewall aplikacyjny na
bazie `cgroups` ). I podobnie w drugą stronę -- można człowiekowi jakiś syf zassać z sieci i go
uruchomić lokalnie (no chyba, że ktoś używa [TPE](https://github.com/cormander/tpe-lkm)). Niemniej
jednak, nasza pokojówka może zwyczajnie nie być przygotowana na taki scenariusz wykorzystania
`usbguard` i jej niecne działania w tym konkretnym podejściu mogą nie przynieść żadnego efektu. My
natomiast będziemy mieć informację w logu, która wskaże nam, że ktoś konkretnego dnia i o
określonej godzinie próbował podłączyć jakieś urządzenie do naszego komputera i to powinno w nas
wzbudzić już jakieś podejrzenia i zarazem zawęzić krąg podejrzanych osób.

Trzeba też pamiętać o tym, że szereg urządzeń, np. smartfony, może posiadać kilka trybów pracy, np.
tryb ładowania czy też tryb przesyłania danych przez protokół MTP. Przełączenie takiego smartfona
między poszczególnymi trybami sprawia, że dla naszego systemu będzie on widziany jako zupełnie inne
urządzenia (inne zestaw interfejsów). Dlatego też warto jest posprawdzać poszczególne tryby pracy
konkretnych urządzeń i dodać reguły dla każdego z obsługiwanych trybów, oczywiście jeśli zamierzy z
nich korzystać.

Może i wspomniany modem LTE miał dwa podejrzane interfejsy, które z początku było ciężko
zidentyfikować ale jak widać niekoniecznie taki stan rzeczy oznacza od razu fakt wpakowania w
urządzenie USB jakichś modułów szpiegowskich czy też takich, które zagrażają bezpieczeństwu systemu.
Takie moduły zwykle też muszą być możliwie proste w budowie, by do ich obsługi nie były potrzebne
dodatkowe sterowniki, których instalacja w systemie byłaby wymagana -- bez nich dany interfejs
zwyczajnie by nie mógł realizować swojego zadania. Na linux'ie bardzo łatwo można ustalić z jakich
modułów korzysta sprzęt i co najważniejsze moduły w kernelu mają otwarte źródła, przez co można
również być pewnym, że nikt tam syfu świadomie nie wpakował. Dlatego też jest spore
prawdopodobieństwo, że te dodatkowe moduły szpiegowskie będą obsługiwane przez jakieś proste/ogólne
sterowniki, które są już dostępne w systemie, tak jak to się odbywa w przypadku klawiatur, myszy
czy pamięci masowych USB -- możemy mieć, np. kilka klawiatur USB i każda z nich będzie działać pod
linux bez instalacji w systemie dodatkowego oprogramowania (zwykle też na tym samym module kernela),
choć czasem pewne rzeczy mogą nie do końca być sprawne, np. cześć klawiszy multimedialnych. Mając
na uwadze powyższe informacje, identyfikator klasy urządzenia (atrybuty `bInterfaceClass` ,
`bInterfaceSubClass` oraz
`bInterfaceProtocol`) [musi być znany](https://www.usb.org/defined-class-codes), bo inaczej
system będzie miał problem z załadowaniem odpowiedniego driver'a, a wtedy atak może się nie powieść,
a nawet jeśli będzie można go przeprowadzić, to jedynie na wąskim gronie użytkowników, co
jednocześnie czyni ten atak niezbyt praktycznym w zastosowaniu.
