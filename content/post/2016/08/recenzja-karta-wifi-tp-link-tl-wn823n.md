---
author: Morfik
categories:
- Hardware
date: "2016-08-17T19:16:51Z"
date_gmt: 2016-08-17 17:16:51 +0200
published: true
status: publish
tags:
- wifi
- recenzja
- tp-link
title: 'Recenzja: Karta WiFi TP-LINK TL-WN823N'
---

Jako mobilny osobnik wiem jak ważne są niewielkie rozmiary sprzętu, na którym przyjdzie mi operować
gdzieś poza miejscem mojego zamieszkania. Wielkość urządzeń ma zatem dla mnie ogromne znaczenie.
Nikogo raczej nie trzeba przekonywać, że wraz z miniaturyzacją sprzętu, maleje również jego
funkcjonalność. Idealne urządzenie to takie, które ma na tyle małe wymiary, by za ich sprawą nie
ucierpiały wszystkie te niezbędne nam dobrodziejstwa oferowane przez nowe technologie. Bez internetu
w obecnych czasach ani rusz, zatem potrzebne nam są różnego rodzaju adaptery i karty WiFi
umożliwiające naszym komputerom bezprzewodowe połączenie sieciowe. Jest cała masa urządzeń, które
moglibyśmy podłączyć do naszych laptopów ale nie wszystkie z nich mają na tyle małe wymiary, by ich
zastosowanie było dla nas iście komfortowe. TP-LINK dysponuje w swojej ofercie kilkoma
bezprzewodowymi kartami sieciowymi w standardzie micro/mini. W tym wpisie zobaczymy jak na linux'ie
będzie sprawował się [mini adapter
TL-WN823N](http://www.tp-link.com.pl/products/details/TL-WN823N.html).

<!--more-->
## Zawartość pudełka

W pudełku poza samym adapterem TL-WN823N mamy również trochę makulatury, w tym też instrukcję
obsługi, oraz płytkę ze sterownikami. Instrukcja jest w kilku językach, min. również i po polsku.
Poniżej znajdują się fotki opakowania oraz jego zawartości:

![](/img/2016/08/1.karta-adapter-TL-WN823N-tp-link.jpg#huge)

![](/img/2016/08/2.karta-adapter-TL-WN823N-tp-link.jpg#huge)

![](/img/2016/08/3.karta-adapter-TL-WN823N-tp-link.jp#hugeg)

![](/img/2016/08/4.karta-adapter-TL-WN823N-tp-link.jpg#huge)

![](/img/2016/08/5.karta-adapter-TL-WN823N-tp-link.jpg#huge)

Jak widzimy, karta ma interfejs USB 2.0 oraz posiada przycisk WPS. Na dobrą sprawę w przypadku
mojego laptopa, ta karta jest zwrócona tym przyciskiem w dół. W efekcie zostaje mi może 1,5 cm
przestrzeni, by ten przycisk wcisnąć, co nie jest chyba wykonalne bez uszkadzania samej karty czy
też i portu USB w laptopie. Sam przycisk bardzo ciężko się wciska, zwłaszcza, gdy adapter jest w
porcie USB. Oczywiście można korzystać z przedłużki ale takowej do zestawu nie dołączono, w końcu to
mini adapter. Poniżej jest fotka adaptera TL-WN823N w porcie USB mojego laptopa:

![](/img/2016/08/6.TP-LINK-TL-WN823N-laptop.jpg#huge)

Nie przypadła mi do gustu również ta skuwka, z którą karta może i dobrze się prezentuje ale trzeba
ten zbędny dodatek zdejmować i ciągle pilnować, by go nie zgubić. To mniej więcej tyle z pierwszego
wrażenia po wyjęciu adaptera z pudełka. Zobaczmy, czy zadziała on nam bezproblemowo pod linux'em.

## Adapter TL-WN823N i jego dwie wersje

Zgodnie z tym, co można wyczytać na stronie TP-LINK'a, adapter TL-WN823N ma dwie różne wersje:
[V1](https://wikidevi.com/wiki/TP-LINK_TL-WN823N_v1) i V2 . W tym przypadku trafiła mi się wersja
V2. Niestety to urządzenie w tej wersji nie jest w ogóle rozpoznawane przez mojego Debiana z
kernelem 4.6 i chyba jest to regułą w przypadku wszystkich pozostałych linux'ów. Po podłączeniu
urządzenia do portu USB, w logu systemowym pojawiają się tylko te poniższe komunikaty:

    kernel: usb 2-1.3: new high-speed USB device number 10 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=2357, idProduct=0109
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: 802.11n NIC
    kernel: usb 2-1.3: Manufacturer: Realtek
    kernel: usb 2-1.3: SerialNumber: 00e04c000001
    mtp-probe[33755]: checking bus 2, device 10: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[33755]: bus: 2, device: 10 was not an MTP device

Urządzenie jest wykrywane jako `idVendor=2357` oraz `idProduct=0109` ale kernel nie ma dla niego
stosownego modułu. W rezultacie nie jest rejestrowany nowy interfejs sieciowy sieciowy i nie damy
rady wejść w interakcję z kartą TL-WN823N v2 na linux'ie, przynajmniej nie zaraz po jej podłączeniu
do portu USB naszego laptopa czy komputera.

## Sterowniki dla TL-WN823N od TP-LINK

[Na stronie TP-LINK'a są niby sterowniki](http://www.tp-link.com/en/download/TL-WN823N.html) do
karty TL-WN823N V2. Niemniej jednak, przy ich kompilacji na kernelu 4.6 jest zwracany bliżej nie
znany mi błąd. W efekcie kompilacja nie może być kontynuowana i nici ze sterownika. Oczywiście na
stronie mamy wzmiankę, że sterownik jest przeznaczony dla kernela w wersji 2.6.18 - 3.10.10 ale
przydałoby się go od czasu do czasu zaktualizować. Nawet stabilny Debian ma kernela w wersji 3.16 ,
a wszyscy wiemy, że nikt z niego nie korzysta, bo za stary.

## Moduł 8192eu (rtl8192eu)

No nic, zostawmy te nieszczęsne sterowniki od TP-LINK'a i spróbujmy znaleźć jakieś alternatywne
rozwiązanie. Z tego co można było wyczytać na wikidev, adapter w wersji V1 był rozpoznawany przez
system i działał w oparciu o moduł `rtl8192cu` . W przypadku V2 potrzebny nam jest [moduł 8192eu
(rtl8192eu)](https://github.com/Mange/rtl8192eu-linux-driver/), a takim kernel linux'a póki co nie
dysponuje. Niemniej jednak, sami [możemy skompilować sobie
moduł 8192eu](/post/sterowniki-tp-link-tl-wn823n-8192eu/) i podczepić go pod
mechanizm DKMS. Trzeba tylko wyraźnie zaznaczyć, że moduł jest bardzo ale to bardzo niestabilny i
może powodować całą masę problemów. Niemniej jednak, karta TL-WN823N potrafi działać na nim dość
dobrze.

## Specyfikacja karty TL-WN823N

Karta TL-WN823N działa w paśmie 2,4 GHz (standard N) i teoretycznie może osiągnąć 300 mbit/s.
Niestety nie dam rady zamieścić wyjścia polecenia `iw list` z powodu braku wsparcia sterownika karty
dla [interfejsu nl80211](https://wireless.wiki.kernel.org/en/developers/documentation/nl80211). Inne
narzędzia operujące na tym interfejsie, np. `crda` , `hostapd` czy też `wpa_supplicant` również będą
mieć problemy z obsługą tej karty.

Poniżej jest zaś jedynie wyjście polecenia `lsusb` :

    # lsusb -vvv -d 2357:0109

    Bus 002 Device 047: ID 2357:0109
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               2.10
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0
      bDeviceProtocol         0
      bMaxPacketSize0        64
      idVendor           0x2357
      idProduct          0x0109
      bcdDevice            2.00
      iManufacturer           1 Realtek
      iProduct                2 802.11n NIC
      iSerial                 3 00e04c000001
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           53
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0xe0
          Self Powered
          Remote Wakeup
        MaxPower              500mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           5
          bInterfaceClass       255 Vendor Specific Class
          bInterfaceSubClass    255 Vendor Specific Subclass
          bInterfaceProtocol    255 Vendor Specific Protocol
          iInterface              2 802.11n NIC
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
            bEndpointAddress     0x05  EP 5 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x06  EP 6 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0200  1x 512 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x87  EP 7 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               3
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x08  EP 8 OUT
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
    Device Status:     0x0001
      Self Powered

## Tryb monitor

Na chwilę obecną moduł `8192eu` nie pozwala na przełączenie karty TL-WN823N w tryb monitor.

## Tryb oszczędzania energii

Tryb oszczędzania energii w przypadku adaptera TL-WN823N zdaje się działać i to nawet poprawnie.
Choć oczywiście jak to na linux'ach bywa, ten tryb może powodować problemy. Jeśli przeglądanie
stron www odbywa się z wyraźnym opóźnieniem lepiej jest ten tryb wyłączyć. Można to zrobić przez
załadowanie modułu `8192eu` z odpowiednimi opcjami. W tym celu wystarczy dopisać do pliku
`/etc/modprobe.d/modules.conf` poniższą linijkę:

    options 8192eu rtw_power_mgnt=0

## Programowy WPS

Może i karta TL-WN823N posiada przycisk WPS ale w przypadku laptopów jest on kompletnie
bezużyteczny. Poza tym, na linux'ie i tak nie działa. Niemniej jednak ten adapter ma
zaimplementowaną obsługę programowego WPS, który działa mniej więcej na takiej samej zasadzie co ten
fizyczny, z tym, że z poziomu systemu operacyjnego "wciskamy" wirtualny przycisk.

## Tryb AP

Na chwilę obecną moduł `8192eu` nie pozwala na przełączenie karty TL-WN823N w tryb AP.

## Test karty TL-WN823N

No to już wiemy z grubsza czym karta TL-WN823N dysponuje. Jak widać, średnio się ta karta nadaje na
linux'a pod względem funkcjonalności. Oczywiście większość z nas raczej chciałaby jedynie podłączyć
się do istniejącej sieci WiFi i pobierać lub wysyłać dane. Sprawdźmy zatem jak ten adapter będzie
się zachowywał w takim przypadku.

Poniższa fotka odnosi się do sytuacji, w której laptop znajduje się w tym samym pomieszczeniu co
router WiFi TP-LINK Archer C2600. Odległość nie przekracza 2 metrów:

![](/img/2016/08/6.karta-adapter-TL-WN823N-tp-link-test-iperf-linux.png#huge)

Około 170 mbit/s. Całkiem przyzwoicie jak na taki mały adapter. Na opakowaniu może i widnieje i 300
mbit/s ale wszyscy wiemy, że te liczby lekko odbiegają od stanu faktycznego. Sprawdźmy jak będzie
wyglądał transfer po zmianie położenia laptopa i dołożeniu przeszkody w postaci niezbyt grubej
ściany. Odległość około 3-4 metry:

![](/img/2016/08/7.karta-adapter-TL-WN823N-tp-link-test-iperf-linux.png#huge)

Dość spory spadek transfery o około 30 mbit/s. Dorzućmy jeszcze jedną ścianę i zobaczmy jak adapter
sobie poradzi w takiej sytuacji. Odległość 5-6 metrów:

![](/img/2016/08/8.karta-adapter-TL-WN823N-tp-link-test-iperf-linux.png#huge)

Poniżej 100 mbit/s ale generalnie nie jest źle, choć mogłoby być trochę lepiej.

## Problemy z TL-WN823N pod linux'em

Jako, że sterownik nie ma póki co wsparcia dla interfejsu nl80211, to nie możemy go określać w
konfiguracji różnych narzędzi. W przypadku osób, korzystających z `wpa_supplicant` do konfiguracji
sieci WiFi, zamiast określać w pliku `/etc/network/interfaces` opcji `wpa-driver nl80211` , musimy
dać `wpa-driver wext` , przykładowo:

    iface wlan20 inet dhcp
          wpa-driver wext
          wpa-debug-level -1
          wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Parametry sieci umieszczamy standardowo w pliku określonym przez parametr `wpa-conf` .
