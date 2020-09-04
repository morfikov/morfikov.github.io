---
author: Morfik
categories:
- Linux
date: "2016-11-19T19:40:24Z"
date_gmt: 2016-11-19 18:40:24 +0100
published: true
status: publish
tags:
- pulseaudio
- dźwięk
- debian
- bluetooth
- smartfon
title: Przesyłanie dźwięku i plików ze smartfona przez bluetooth
---

Praktycznie każdy telefon czy komputer/laptop jest wyposażony już w adapter bluetooth. Obecnie coraz
rzadziej ten protokół jest wykorzystywany do przesyłania plików jako takich, bo przecie mamy WiFi.
Niemniej jednak, w starszych modelach urządzeń bluetooth może być dla nas jedyną opcją, by przesłać
pliki bezprzewodowo. Nawet jeśli dysponujemy smartfonem z jednym z nowszych Androidów, to i tak są
pewne sytuacje, w których bluetooth może znaleźć ciekawe zastosowanie, np. streaming dźwięku.
Urządzenia bluetooth bardzo często sprawiają problemy pod linux i przydałoby się zebrać trochę
informacji na temat ich konfiguracji, by przesyłanie dźwięku i plików nie stanowiło dla nas
większego wyzwania w sytuacjach podbramkowych, gdzie wszelkie inne alternatywy komunikacji zawodzą.

<!--more-->
## Oprogramowanie bluetooth w Debianie

W zależności od wykorzystywanego środowiska graficznego, inne oprogramowanie jest wykorzystywane do
zarządzania urządzeniami bluetooth. To co się zdaje łączyć wszystkie te graficzne nakładki, to fakt
możliwości równoległego wykorzystywania niskopoziomowego narzędzia `bluetoothctl` . Ja nie korzystam
z żadnego środowiska graficznego, dlatego też nie będę tutaj opisywał jak z poziomu GUI
skonfigurować do pracy adapter bluetooth (czy to ten na USB, czy też ten wbudowany w nasz laptop),
a jedynie skupię się na tym konsolowym `bluetoothctl` .

Narzędzie `bluetoothctl` jest dostarczane z pakietem `bluez` . W tym pakiecie znajdują się również
narzędzia takie jak `hcitool` i `hciconfig` , które są w stanie skonfigurować sam interfejs `hci0` .
Jest tam również `sdptool` umożliwiający wysyłanie zapytań do serwera SDP, choć w przypadku
korzystania Bluez 5 mogą pojawić się pewne problemy z tymi zapytaniami (o tym później). Generalnie
to `sdptool` może nam zwrócić informacje na temat usług jakie jest w stanie świadczyć urządzenie, z
którym zamierzamy łączyć się po bluetooth.

Jeśli zamierzamy przesyłać dźwięk ze smartfona bezpośrednio na komputer (lub też i w drugą stronę),
np. chcemy skorzystać z głośników komputera w celu odtworzenia mp3, które mamy na smartfonie, to nie
obejdzie się bez zaprzęgnięcia do tego celu serwera dźwięku PulseAudio. O ile większość linux'ów
domyślnie wykorzystuje już PulseAudio, to nie wszystkie jego moduły są standardowo zainstalowane w
każdej dystrybucji. Dlatego też upewnijmy się, że mamy w swoim Debianie wgrany pakiet
`pulseaudio-module-bluetooth` .

W przypadku przesyłania plików, to sprawa jest trochę skomplikowana. Standardowo jest wykorzystywany
[protokół OBEX](https://en.wikipedia.org/wiki/OBject_EXchange) (OBject EXchange), który wymaga
klienta i serwera. W przypadku mojego starego telefonu, byłem w stanie zamontować jego zasoby i
przeglądać je lokalnie na komputerze. W przypadku smartfona z Androidem 5.1 (Lollipop) mogę jedynie
przesłać pojedynczy plik ale tylko z komputera na telefon. Nie jest to wygodne rozwiązanie ale i
tak, by móc z niego skorzystać potrzebny nam jest pakiet `ussp-push` . By można było przesłać plik
również z telefonu na komputer, musimy zainstalować w Debianie także pakiet `obexpushd`. Jeśli
chcemy zaś przeglądać zasoby telefonu, to ten musi wspierać OBEX FTP oraz w systemie trzeba
doinstalować `obexfs` .

Podsumowując, potrzebne nam są te poniższe pakiety:

    # aptitude install \
    bluez \
    bluez-firmware \
    bluez-obexd \
    obexfs \
    ussp-push \
    obexpushd \
    pulseaudio-module-bluetooth

W pakiecie `bluez` jest także zawarta usługa `bluetooth.service` , którą możemy śmiało włączyć. Nie
zostanie ona jednak wystartowana do momentu wykrycia w systemie odpowiedniego adaptera bluetooth.

## Adapter bluetooth

Ja dysponuję dwoma różnymi adapterami bluetooth (Pentagram i Quer) ale obydwa są rozpoznawane jako
`idVendor=0a12` i `idProduct=0001` . Tak czy inaczej, taki adapter pracuje pod linux bez większych
niespodzianek. To na wypadek, gdyby ktoś potrzebował sobie takie urządzenie zakupić i szukał czegoś
co działa bezproblemowo.

    # lsusb -vvv -d 0a12:0001


    Bus 002 Device 007: ID 0a12:0001 Cambridge Silicon Radio, Ltd Bluetooth Dongle (HCI mode)
    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               1.10
      bDeviceClass          224 Wireless
      bDeviceSubClass         1 Radio Frequency
      bDeviceProtocol         1 Bluetooth
      bMaxPacketSize0        16
      idVendor           0x0a12 Cambridge Silicon Radio, Ltd
      idProduct          0x0001 Bluetooth Dongle (HCI mode)
      bcdDevice            1.34
      iManufacturer           0
      iProduct                0
      iSerial                 0
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength          108
        bNumInterfaces          2
        bConfigurationValue     1
        iConfiguration          0
        bmAttributes         0x80
          (Bus Powered)
        MaxPower              100mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           3
          bInterfaceClass       224 Wireless
          bInterfaceSubClass      1 Radio Frequency
          bInterfaceProtocol      1 Bluetooth
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x81  EP 1 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0010  1x 16 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            2
              Transfer Type            Bulk
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval               0
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass       224 Wireless
          bInterfaceSubClass      1 Radio Frequency
          bInterfaceProtocol      1 Bluetooth
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0000  1x 0 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x03  EP 3 OUT
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0000  1x 0 bytes
            bInterval               1
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       1
          bNumEndpoints           2
          bInterfaceClass       224 Wireless
          bInterfaceSubClass      1 Radio Frequency
          bInterfaceProtocol      1 Bluetooth
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0009  1x 9 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x03  EP 3 OUT
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0009  1x 9 bytes
            bInterval               1
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        1
          bAlternateSetting       2
          bNumEndpoints           2
          bInterfaceClass       224 Wireless
          bInterfaceSubClass      1 Radio Frequency
          bInterfaceProtocol      1 Bluetooth
          iInterface              0
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x83  EP 3 IN
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0011  1x 17 bytes
            bInterval               1
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x03  EP 3 OUT
            bmAttributes            1
              Transfer Type            Isochronous
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0011  1x 17 bytes
            bInterval               1
    Device Status:     0x0000
      (Bus Powered)

Poniżej jest dokładny log z tego co się dzieje w systemie po podłączeniu tego adaptera do portu USB:

    kernel: usb 2-1.3: new full-speed USB device number 7 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=0a12, idProduct=0001
    kernel: usb 2-1.3: New USB device strings: Mfr=0, Product=0, SerialNumber=0
    kernel: Bluetooth: Core ver 2.21
    kernel: NET: Registered protocol family 31
    kernel: Bluetooth: HCI device and connection manager initialized
    kernel: Bluetooth: HCI socket layer initialized
    kernel: Bluetooth: L2CAP socket layer initialized
    kernel: Bluetooth: SCO socket layer initialized
    kernel: usbcore: registered new interface driver btusb
    systemd[1]: Starting Load/Save RF Kill Switch Status...
    systemd[1]: Started Load/Save RF Kill Switch Status.
    systemd[1]: Starting Bluetooth service...
    bluetoothd[3404]: Bluetooth daemon 5.43
    systemd[1]: Started Bluetooth service.
    systemd[1]: Reached target Bluetooth.
    bluetoothd[3404]: Starting SDP server
    kernel: Bluetooth: BNEP (Ethernet Emulation) ver 1.3
    kernel: Bluetooth: BNEP filters: protocol multicast
    kernel: Bluetooth: BNEP socket layer initialized
    bluetoothd[3404]: Bluetooth management interface 1.13 initialized
    bluetoothd[3404]: Failed to obtain handles for "Service Changed" characteristic
    bluetoothd[3404]: Sap driver initialization failed.
    bluetoothd[3404]: sap-server: Operation not permitted (1)
    bluetoothd[3404]: Endpoint registered: sender=:1.29 path=/MediaEndpoint/A2DPSource
    bluetoothd[3404]: Endpoint registered: sender=:1.29 path=/MediaEndpoint/A2DPSink
    kernel: Bluetooth: RFCOMM TTY layer initialized
    kernel: Bluetooth: RFCOMM socket layer initialized
    kernel: Bluetooth: RFCOMM ver 1.11

Jak widać proces podnoszenia usług systemowych jest praktycznie automatyczny. Niemniej jednak, w
zależności od usług, z których zamierzamy korzystać przez bluetooth, musimy jeszcze nieco czasu
poświęcić przy konfiguracji adaptera.

## Problemy z sdptool w Bluez 5

Narzędzie `sdptool` w Bluez 5 nie działa jak należy, w efekcie czego nie mamy możliwości nawiązania
połączenia z serwerem SDP. Możemy sprawdzić czy ten problem nas dotyczy logując się na użytkownika
root i wydając w terminalu poniższe polecenia:

    # sdptool browse FF:FF:FF:00:00:00
    Failed to connect to SDP server on FF:FF:FF:00:00:00: Connection refused

    # sdptool get Lan
    Failed to connect to SDP server on FF:FF:FF:00:00:00: Connection refused

Można tej sytuacji zaradzić instalując pakiet `bluez` w wersji 4, co nie zawsze jest możliwe, lub
też edytując plik usługi systemd odpowiedzialny za odpalenie demona `bluetoothd` :

    # systemctl edit --full bluetooth.service

Teraz wystarczy zmienić linijkę z `ExecStart` dopisując na jej końcu parametr `--compat` :

    ExecStart=/usr/lib/bluetooth/bluetoothd --compat

Zapisujemy plik i restartujemy usługę:

    # systemctl reenable bluetooth
    # systemctl daemon-reload
    # systemctl restart bluetooth

W tej chwili powinniśmy już być wstanie podłączyć się do lokalnego serwera SDP:

    # sdptool get Lan
    Service RecHandle: 0x0
    Service Class ID List:
      "SDP Server" (0x1000)

    # sdptool browse FF:FF:FF:00:00:00

    Browsing FF:FF:FF:00:00:00 ...
    Service RecHandle: 0x10000
    Service Class ID List:
      "PnP Information" (0x1200)
    Profile Descriptor List:
      "PnP Information" (0x1200)
        Version: 0x0103

    Browsing FF:FF:FF:00:00:00 ...
    Service Search failed: Invalid argument
    Service Name: Generic Access Profile
    Service Provider: BlueZ
    Service RecHandle: 0x10001
    Service Class ID List:
      "Generic Access" (0x1800)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 31
      "ATT" (0x0007)
        uint16: 0x0001
        uint16: 0x0005

    Service Name: Generic Attribute Profile
    Service Provider: BlueZ
    Service RecHandle: 0x10002
    Service Class ID List:
      "Generic Attribute" (0x1801)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 31
      "ATT" (0x0007)
        uint16: 0x0006
        uint16: 0x0009

    Service Name: AVRCP CT
    Service RecHandle: 0x10003
    Service Class ID List:
      "AV Remote" (0x110e)
      "AV Remote Controller" (0x110f)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0103
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0106

    Service Name: AVRCP TG
    Service RecHandle: 0x10004
    Service Class ID List:
      "AV Remote Target" (0x110c)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0103
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0105

    Service Name: Audio Source
    Service RecHandle: 0x10005
    Service Class ID List:
      "Audio Source" (0x110a)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 25
      "AVDTP" (0x0019)
        uint16: 0x0103
    Profile Descriptor List:
      "Advanced Audio" (0x110d)
        Version: 0x0103

    Service Name: Audio Sink
    Service RecHandle: 0x10006
    Service Class ID List:
      "Audio Sink" (0x110b)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 25
      "AVDTP" (0x0019)
        uint16: 0x0103
    Profile Descriptor List:
      "Advanced Audio" (0x110d)
        Version: 0x0103

    Service Name: Headset Voice gateway
    Service RecHandle: 0x10007
    Service Class ID List:
      "Headset Audio Gateway" (0x1112)
      "Generic Audio" (0x1203)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 12
    Profile Descriptor List:
      "Headset" (0x1108)
        Version: 0x0102

Niemniej jednak, te powyższe polecenia w dalszym ciągu nie zadziałają nam na zwykłym użytkowniku. Ta
opcja `--compat` dodaje unix'owy socket w lokalizacji `/var/run/sdp` . Jeśli chcemy korzystać z
bluetooth jako normalny użytkownik, to musimy nadać odpowiednie uprawnienia na ten plik, bo
standardowo tylko root może z niego korzystać. Problem w tym, że po zrestartowaniu demona
`bluetoothd` , ten plik zostanie utworzony na nowo i znowu będzie trzeba mu te uprawnienia
dostosować. Tworzymy zatem dwa poniższe pliki.

Plik `/etc/systemd/system/var-run-sdp.path`

    [Path]
    PathExists=/var/run/sdp
    Unit=var-run-sdp.service

    [Install]
    WantedBy=bluetooth.service

Plik `/etc/systemd/system/var-run-sdp.service`

    [Unit]
    Description=Setting /var/run/sdp permissions
    ConditionPathExists=/var/run/sdp

    [Service]
    Type=simple
    RemainAfterExit=true
    ExecStart=/bin/chgrp bluetooth /var/run/sdp

    [Install]
    RequiredBy=var-run-sdp.path

Zadanie jakie ma realizować ta usługa polega na monitorowaniu pliku `/var/run/sdp` . Ilekroć tylko
zostanie on stworzony, systemd przepisze mu uprawnienia, a konkretnie zmieni mu grupę z `root` na
`bluetooth` . Włączmy jeszcze tę usługę:

    # systemctl daemon-reload
    # systemctl enable var-run-sdp.path
    # systemctl start var-run-sdp.path

Na koniec dodajemy pożądanych użytkowników do grupy `bluetooth` :

    # adduser morfik bluetooth

Restartujemy jeszcze demona `bluetoothd` i sprawdzamy, czy plik `/var/run/sdp` ma odpowiednie
uprawnienia:

    # systemctl restart bluetooth

    # ls -al /var/run/sdp
    srw-rw---- 1 root bluetooth 0 2016-11-17 14:54:30 /var/run/sdp=

Teraz już wystarczy zrestartować sesję (zalogować się ponownie w systemie) i zwykły użytkownik
powinien bez większego problemu podłączyć się do serwera SDP.

## Parowanie komputera ze smartfonem

Cały proces parowania urządzeń w protokole bluetooth można przeprowadzić posługując się narzędziem
`bluetoothctl` . Zakładając, że odpowiednie usługi już działają nam w tle, to możemy przejść do
konfiguracji bluetooth na smartfonie i na Debianie. Na smartfonie nie powinno być większych
problemów. Przechodzimy do Ustawienia => Bluetooth i aktywujemy nadajnik:

![]({{< baseurl >}}/img/2016/11/001.linux-debian-bluetooth-smartfon-aktywacja-telefon.png#huge)

Smartfon powinien zacząć wyszukiwać dostępne urządzenia ale nie znajdzie jeszcze naszego komputera.
Musimy pierw uczynić go możliwym do odnalezienia. Odpalamy zatem terminal i jako zwykły użytkownik
wpisujemy `bluetoothctl` (działa autouzupełnianie za pomocą klawisza Tab):

![]({{< baseurl >}}/img/2016/11/002.linux-debian-bluetooth-smartfon-informacje-usb-adapter.png#huge)

Mamy tutaj informacje na temat aktualnie podpiętego adaptera bluetooth do portu USB komputera. Nazwa
widoczna przy skanowaniu to wykorzystywany hostname. Klasa z kolei określa usługi jakie dana maszyna
jest w stanie świadczyć. [Dokładna lista jest uwzględniona
tutaj](http://bluetooth-pentest.narod.ru/software/bluetooth_class_of_device-service_generator.html).
Mamy też informacje na temat zasilania i możliwości parowania. Z kolei zaś wszystkie pozycje z UUID
odnoszą się do usług jakie maszyna jest w stanie zapewnić (można konfigurować dowolnie).

Jako, że naszym celem jest sprawowanie komputera ze smartfonem, to w konsoli bluetooth wpisujemy
kolejno poniższe polecenia:

    [bluetooth]# power on
    [bluetooth]# discoverable on
    [bluetooth]# pairable on

![]({{< baseurl >}}/img/2016/11/003.linux-debian-bluetooth-smartfon-aktywacja-komputer.png#huge)

Teraz możemy sprawować urządzenia. Ten proces może zostać zainicjowany ze smartfona lub z komputera.
W przypadku smartfona raczej nie powinno być problemów, to sparujmy te urządzenia inicjując cały
proces na komputerze.

Przede wszystkim musimy aktywować agenta, który wynegocjuje odpowiednie parametry połączenia,
wliczając w to sposób przekazania sekretu. Ten sekret trzeba wpis zarówno na komputerze jak i
smartfonie. Zwykle jest to czterocyfrowy PIN, np. 0000 lub 1234. Zarówno komputer jak i smartfon
dysponują klawiaturami, dlatego też możemy wybrać sobie agenta `KeyboardOnly` :

    [bluetooth]# agent KeyboardOnly
    [bluetooth]# default-agent

![]({{< baseurl >}}/img/2016/11/004.linux-debian-bluetooth-smartfon-agent.png#huge)

Teraz wysyłamy prośbę parowania:

    [bluetooth]# scan on
    [bluetooth]# pair 40:3F:8C:02:41:AD

![]({{< baseurl >}}/img/2016/11/005.linux-debian-bluetooth-smartfon-parowanie-urzadzen.png#huge)

Wyżej widzimy, że agent poprosił nas o wpisanie kodu PIN, oraz że wpisany kod to 1234. Teraz
przechodzimy na telefon. Powinno nam się pojawić to poniższe okienko, w którym wpisujemy PIN:

![]({{< baseurl >}}/img/2016/11/006.linux-debian-bluetooth-smartfon-parowanie-urzadzen.png#medium)

Urządzenia mamy sparowane. Dla pewności możemy sprawdzić czy faktycznie system zarówno komputera jak
i telefonu honoruje te ustawienia.

Komputer:

![]({{< baseurl >}}/img/2016/11/007.linux-debian-bluetooth-smartfon-sparowane-urzadzenia.png#huge)

Telefon:

![]({{< baseurl >}}/img/2016/11/008.linux-debian-bluetooth-smartfon-sparowane-urzadzenia.png#huge)

Wygląda dobrze. Podłączmy zatem ze sobą te dwa sprzęty:

    [bluetooth]# connect 40:3F:8C:02:41:AD

![]({{< baseurl >}}/img/2016/11/009.linux-debian-bluetooth-smartfon-laczenie-urzadzen.png#huge)

Na telefonie zaś powinien wyskoczyć status połączony:

![]({{< baseurl >}}/img/2016/11/010.linux-debian-bluetooth-smartfon-polaczone-urzadzenia.png#medium)

Warto zauważyć, że prompt zmienił nazwę z `bluetooth` na nazwę urządzenia, w tym przypadku
`NeffosC5` . Połączenie zostało ustanowione i przydałoby się oznaczyć nasz smartfon jako zaufany:

    [NeffosC5]# trust 40:3F:8C:02:41:AD

![]({{< baseurl >}}/img/2016/11/011.linux-debian-bluetooth-smartfon-zaufane-urzadzenia.png#huge)

Na koniec sprawdzamy jeszcze status smartfona:

    [NeffosC5]# info 40:3F:8C:02:41:AD

![]({{< baseurl >}}/img/2016/11/012.linux-debian-bluetooth-smartfon-informacje-o-urzadzeniu.png#huge)

Wygląda dobrze. Możemy zatem przejść do skonfigurowania przesyłania plików jak i streaming'u
dźwięku.

## Problemy przy połączeniu ze smartfonem przez bluetooth

Czasami podczas konfiguracji systemu pod streaming dźwięku można napotkać problemy z podłączeniem
się do smartfona. Po skorzystaniu z polecenia `connect` dostajemy jedynie poniższy błąd:

    [bluetooth]# connect 40:3F:8C:02:41:AD
    Attempting to connect to 40:3F:8C:02:41:AD
    Failed to connect: org.bluez.Error.Failed

W logu systemowym zaś można dostrzec następujący komunikat:

    bluetoothd[63148]: a2dp-source profile connect failed for 40:3F:8C:02:41:AD: Protocol not available

Ten problem można poprawić instalując w systemie pakiet `pulseaudio-module-bluetooth` . Po
zainstalowaniu tego pakietu musimy załadować odpowiednie moduły w poniższy sposób:

    $ pactl load-module module-bluetooth-discover
    $ pactl load-module module-bluetooth-policy

    $ pactl list modules | grep blue
        Name: module-bluetooth-discover
        Name: module-bluez5-discover
        Name: module-bluetooth-policy
        Name: module-bluez5-device

Jeśli z jakiegoś powodu te moduły nie są ładowane automatycznie na starcie PulseAudio, to musimy
poddać edycji plik `/etc/pulse/default.pa` i dopisać w nim następujące linijki:

    .ifexists module-bluetooth-policy.so
    load-module module-bluetooth-policy
    .endif

    .ifexists module-bluetooth-discover.so
    load-module module-bluetooth-discover
    .endif

Teraz nie powinniśmy mieć już problemów z podłączeniem się do smartfona. Obecność w logu systemowym
poniższych informacji wskazuje, że problem został wyeliminowany:

    ...
    bluetoothd[63148]: Endpoint registered: sender=:1.95 path=/MediaEndpoint/A2DPSource
    bluetoothd[63148]: Endpoint registered: sender=:1.95 path=/MediaEndpoint/A2DPSink
    ...
    kernel: input: 40:3F:8C:02:41:AD as /devices/virtual/input/input34
    ...

## Jak strumieniować dźwięk ze smartfona przez bluetooth

W opcjach urządzenia bluetooth na smartfonie mogliśmy zobaczyć, że była tam zaznaczona pozycja
"Dźwięk multimediów". Poniżej jest jeszcze raz ta fotka:

![]({{< baseurl >}}/img/2016/11/013.linux-debian-bluetooth-smartfon-streaming-dzwieku.png#medium)

Jeśli jest ona zaznaczona, to dźwięk powinien być przesłany po bluetooth do komputera praktycznie
OOTB. Tam powinien go odebrać PulseAudio. Możemy to w bardzo prosty sposób sprawdzić. Uruchommy
zatem na smartfonie jakąś aplikację, która jest w stanie odtwarzać dźwięk. Ja korzystałem z VLC. Jak
tylko plik `.mp3` zaczął być odtwarzany, głośniki mojego komputera zaczęły grać muzykę. Podglądając
w takim momencie co się dzieje na na serwerze dźwięku (przez `pavucontrol` ) można zobaczyć coś
takiego:

![]({{< baseurl >}}/img/2016/11/014.linux-debian-bluetooth-smartfon-streaming-dzwiek-pulseaudio.png#huge)

Ta pozycja `loopback from NeffosC5` to jest właśnie strumień naszego smartfona, którym można
dowolnie zarządzać, tak jakby to lokalnie była odtwarzana jakaś aplikacja dźwiękowa. Operowanie
głośnością na smartfonie również ma efekt na komputerze. Oczywiście przykład z komputerem nie jest
najlepszy, bo przecie można bez problemu taka muzykę zgrać na dysk z telefonu ale korzystając, np. z
systemu live czy mając zwykłe głośniki bluetooth, to już taki setup nabiera większego sensu.

Warto też wspomnieć, że smartfony mogą nadawać lepszej jakości dźwięk przez bluetooth. Trzeba tylko
tę opcję włączyć w Ustawienia => Dźwięki i Powiadomienia => Funkcje poprawy dźwięku:

![]({{< baseurl >}}/img/2016/11/015.linux-debian-bluetooth-smartfon-poprawa-jakosci-dzwieku.png#huge)

Problem ze smartfonami jest taki, że standardowe ROM'y nie umożliwiają przesłania dźwięku z PC na
smartfona. Póki co jeszcze nie wiem jak tego typu przedsięwzięcie zorganizować. Jak natknę się na
jakieś ciekawe info odnośnie tej kwestii, to dorzucę je tutaj. (FIXME)

## Przesyłanie plików przez bluetooth (OBEX)

Pora skupić się na przesyłaniu plików przez bluetooth. Standardowo w linux'ie możemy przesłać
jedynie plik z komputera na telefon. Można to zrobić posługując się dowolną graficzną nakładką
zdolną zarządzać modułem bluetooth, np. `blueman` . Jeśli zaś chodzi o polecenia konsolowe, to
najpierw musimy ustalić na jakich kanałach są dostępne usługi bluetooth ale zanim do tego
przejdziemy, to skonfigurujmy sobie interfejs `hci0` :

    # hcitool dev
    Devices:
        hci0    E8:03:9A:CB:0C:24

    # hciconfig hci0 up

    # hciconfig -a hci0
    hci0:   Type: BR/EDR  Bus: USB
        BD Address: E8:03:9A:CB:0C:24  ACL MTU: 1022:8  SCO MTU: 183:5
        UP RUNNING PSCAN ISCAN
        RX bytes:30365733 acl:46626 sco:0 events:576 errors:0
        TX bytes:2920 acl:70 sco:0 commands:77 errors:0
        Features: 0xff 0xfe 0x0d 0xfe 0xd8 0x7f 0x7b 0x87
        Packet type: DM1 DM3 DM5 DH1 DH3 DH5 HV1 HV2 HV3
        Link policy: RSWITCH HOLD SNIFF
        Link mode: SLAVE ACCEPT
        Name: 'morfikownia'
        Class: 0x1c010c
        Service Classes: Rendering, Capturing, Object Transfer
        Device Class: Computer, Laptop
        HCI Version: 4.0 (0x6)  Revision: 0x102
        LMP Version: 4.0 (0x6)  Subversion: 0x1
        Manufacturer: Atheros Communications, Inc. (69)

Skanujemy teraz w poszukiwaniu urządzeń bluetooth:

    # hcitool scan
    Scanning ...
        40:3F:8C:02:41:AD   NeffosC5

Jeśli `hcitool` nie jest w stanie wykryć żadnych urządzeń, oznacza to, że adapter w komputerze jest
w jakiś sposób uszkodzony. Jeśli zaś cześć urządzeń jest wykrywana poprawnie ale brakuje tego
naszego na liście, to oznacza, że prawdopodobnie to urządzenie ma uszkodzony moduł bluetooth.
Oczywiście przy założeniu, że oba sprzęty są poprawnie skonfigurowane do pracy.

Mając adres MAC smartfona, możemy teraz przeszukać jego serwer SDP pod kątem usług:

    $ sdptool browse 40:3F:8C:02:41:AD
    Browsing 40:3F:8C:02:41:AD ...
    Service RecHandle: 0x10001
    Service Class ID List:
      "Generic Access" (0x1800)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 31
      "ATT" (0x0007)
        uint16: 0x0001
        uint16: 0x0005

    Service Name: Headset Audio Gateway
    Service RecHandle: 0x10002
    Service Class ID List:
      "Headset Audio Gateway" (0x1112)
      "Generic Audio" (0x1203)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 1
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Headset" (0x1108)
        Version: 0x0102

    Service Name: Handsfree Audio Gateway
    Service RecHandle: 0x10003
    Service Class ID List:
      "Handsfree Audio Gateway" (0x111f)
      "Generic Audio" (0x1203)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 2
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Handsfree" (0x111e)
        Version: 0x0106

    Service Name: Network Access Point Service
    Service Description: Bluetooth NAP Service
    Service RecHandle: 0x10004
    Service Class ID List:
      "Network Access Point" (0x1116)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 15
      "BNEP" (0x000f)
        Version: 0x0100
        SEQ8: 0 6
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Network Access Point" (0x1116)
        Version: 0x0100

    Service RecHandle: 0x10006
    Service Class ID List:
      "AV Remote Target" (0x110c)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0100
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0103

    Service RecHandle: 0x10007
    Service Class ID List:
      "AV Remote Controller" (0x110f)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0100
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0100

    Service Name: Advanced Audio
    Service RecHandle: 0x10008
    Service Class ID List:
      "Audio Source" (0x110a)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 25
      "AVDTP" (0x0019)
        uint16: 0x0102
    Profile Descriptor List:
      "Advanced Audio" (0x110d)
        Version: 0x0102

    Service Name: 000eSMS/MMS
    Service RecHandle: 0x10009
    Service Class ID List:
      "Message Access - MAS" (0x1132)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 23
      "OBEX" (0x0008)
    Profile Descriptor List:
      "Message Access" (0x1134)
        Version: 0x0101

    Service Name: OBEX Phonebook Access Server
    Service RecHandle: 0x1000a
    Service Class ID List:
      "Phonebook Access - PSE" (0x112f)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 24
      "OBEX" (0x0008)
    Profile Descriptor List:
      "Phonebook Access" (0x1130)
        Version: 0x0101

    Service Name: OBEX Object Push
    Service RecHandle: 0x1000b
    Service Class ID List:
      "OBEX Object Push" (0x1105)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 25
      "OBEX" (0x0008)
    Profile Descriptor List:
      "OBEX Object Push" (0x1105)
        Version: 0x0100

### Narzędzie ussp-push

Mamy tutaj min. `OBEX Object Push` . Możemy zatem przesłać plik na smartfon. Musimy tylko zanotować
sobie kanał, który przy tej pozycji jest uwzględniony, tj. `Channel: 25` . Polecenie, które prześle
plik z komputera na telefon będzie miało zatem poniższą postać:

    $ ussp-push 40:3F:8C:02:41:AD@25 plik_z_kompa plik_na_smartfonie

### Narzędzie obexpushd

Jeśli zaś chodzi o przesyłanie plików ze smartfona na komputer, to możemy skorzystać z poniższego
polecenia:

    $ obexpushd -o ~/bluetooth/

Ta komenda uruchamiana demona, który nasłuchuje zapytań. Pliki odbierane ze smartfona będą
przesyłane do katalogu `~/bluetooth/` .

### OBEX FTP i Bluetooth File Transfer

Alternatywnym podejściem do przesyłania plików przez bluetooth jest zamontowanie systemu plików
telefonu lokalnie i operowanie na nim jak na zwyczajnym systemie plików. Niemniej jednak, taki
zabieg wymaga doinstalowania odpowiedniego oprogramowania na smartfonie. Wyżej w listingu usług
zwróconych przez `sdptool browse` mieliśmy co prawda dwie pozycje od OBEX'a ale żadna z nich nie
zawierała frazy FTP. Z pobieżnego rozeznania wyśledziłem aplikację [Bluetooth File
Transfer](https://play.google.com/store/apps/details?id=it.medieval.blueftp). Jej zainstalowanie w
Androidzie uaktywni usługę OBEX FTP:

![]({{< baseurl >}}/img/2016/11/016.linux-debian-bluetooth-smartfon-file-transfer.png#huge)

Po uruchomieniu tej aplikacji, na dole powinna pojawić się poniższa informacja:

![]({{< baseurl >}}/img/2016/11/017.linux-debian-bluetooth-smartfon-file-transfer.png#medium)

Zatem OBEX FTP został uruchomiony. Sprawdźmy czy faktycznie tak jest przez odpytanie serwera SDP na
smartfonie:

    $ sdptool browse 40:3F:8C:02:41:AD
    Browsing 40:3F:8C:02:41:AD ...
    Service RecHandle: 0x10001
    Service Class ID List:
      "Generic Access" (0x1800)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 31
      "ATT" (0x0007)
        uint16: 0x0001
        uint16: 0x0005

    Service Name: Headset Audio Gateway
    Service RecHandle: 0x10002
    Service Class ID List:
      "Headset Audio Gateway" (0x1112)
      "Generic Audio" (0x1203)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 1
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Headset" (0x1108)
        Version: 0x0102

    Service Name: Handsfree Audio Gateway
    Service RecHandle: 0x10003
    Service Class ID List:
      "Handsfree Audio Gateway" (0x111f)
      "Generic Audio" (0x1203)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 2
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Handsfree" (0x111e)
        Version: 0x0106

    Service Name: Network Access Point Service
    Service Description: Bluetooth NAP Service
    Service RecHandle: 0x10004
    Service Class ID List:
      "Network Access Point" (0x1116)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 15
      "BNEP" (0x000f)
        Version: 0x0100
        SEQ8: 0 6
    Language Base Attr List:
      code_ISO639: 0x656e
      encoding:    0x6a
      base_offset: 0x100
    Profile Descriptor List:
      "Network Access Point" (0x1116)
        Version: 0x0100

    Service RecHandle: 0x10006
    Service Class ID List:
      "AV Remote Target" (0x110c)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0100
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0103

    Service RecHandle: 0x10007
    Service Class ID List:
      "AV Remote Controller" (0x110f)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 23
      "AVCTP" (0x0017)
        uint16: 0x0100
    Profile Descriptor List:
      "AV Remote" (0x110e)
        Version: 0x0100

    Service Name: Advanced Audio
    Service RecHandle: 0x10008
    Service Class ID List:
      "Audio Source" (0x110a)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
        PSM: 25
      "AVDTP" (0x0019)
        uint16: 0x0102
    Profile Descriptor List:
      "Advanced Audio" (0x110d)
        Version: 0x0102

    Service Name: 000eSMS/MMS
    Service RecHandle: 0x10009
    Service Class ID List:
      "Message Access - MAS" (0x1132)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 23
      "OBEX" (0x0008)
    Profile Descriptor List:
      "Message Access" (0x1134)
        Version: 0x0101

    Service Name: OBEX OPP
    Service RecHandle: 0x1000a
    Service Class ID List:
      "OBEX Object Push" (0x1105)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 24
      "OBEX" (0x0008)
    Profile Descriptor List:
      "OBEX Object Push" (0x1105)
        Version: 0x0100

    Service Name: OBEX FTP
    Service RecHandle: 0x1000b
    Service Class ID List:
      UUID 128: 00001106-0000-1000-8000-00805f9b34fb
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 25

    Service Name: OBEX Phonebook Access Server
    Service RecHandle: 0x1000c
    Service Class ID List:
      "Phonebook Access - PSE" (0x112f)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 26
      "OBEX" (0x0008)
    Profile Descriptor List:
      "Phonebook Access" (0x1130)
        Version: 0x0101

    Service Name: OBEX Object Push
    Service RecHandle: 0x1000d
    Service Class ID List:
      "OBEX Object Push" (0x1105)
    Protocol Descriptor List:
      "L2CAP" (0x0100)
      "RFCOMM" (0x0003)
        Channel: 27
      "OBEX" (0x0008)
    Profile Descriptor List:
      "OBEX Object Push" (0x1105)
        Version: 0x0100

I tutaj już widzimy, że poza tymi wcześniejszymi `OBEX Object Push` i `OBEX Phonebook Access Server`
doszły nam również `OBEX OPP` i `OBEX FTP` . Ten ostatni ma kanał `Channel: 25` . Zapamiętujemy
sobie tę wartość i łączymy się z telefonem w poniższy sposób:

    $ obexfs -b 40:3F:8C:02:41:AD -B 25 ~/bluetooth/

Udostępniany katalog na smartfonie powinien nam się zamontować we wskazanym wyżej miejscu:

![]({{< baseurl >}}/img/2016/11/018.linux-debian-bluetooth-smartfon-obex.png#huge)

Ten katalog można oczywiście sobie dostosować w opcjach aplikacji na smartfonie. Można również
autoryzować każde podłączenie. Generalnie, to polecam zajrzenie w opcje aplikacji Bluetooth File
Transfer.

By ten katalog odmontować, korzystamy z `fusermount` :

    $ fusermount -u ~/bluetooth
