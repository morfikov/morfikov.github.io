---
author: Morfik
categories:
- Linux
date: "2016-04-06T18:51:58Z"
date_gmt: 2016-04-06 16:51:58 +0200
published: true
status: publish
tags:
- debian
- lte
- modem
- huawei
- e3372
GHissueID: 430
title: Modem LTE HUAWEI E3372 bez usb-modeswitch
---

Ten artykuł, podobnie jak kilka poprzednich, powstał w oparciu o moje badania przeprowadzane nad
modemem LTE HUAWEI E3372s-153 w wersji NON-HiLink. Niby rozwiązań dotyczących konfiguracji modemów
pod linux'em jest pełno w sieci ale zasadniczą różnicą niżej opisanego sposobu jest kompletne
pozbycie się pakietu `usb-modeswitch` . Dla przypomnienia, ten pakiet odpowiada za przełączanie
trybu modemu. Zwykle są dostępne dwa tryby. Pierwszy z nich jest w stanie dostarczyć sterowniki (pod
windows), po instalacji których modem przechodzi w drugi tryb, już ten właściwy. Na linux'ach to
przełączanie jest realizowane via `usb-modeswitch` . I tu się nasuwa pytanie, czy ten modem
faktycznie trzeba przełączać? A może istnieje sposób, który by automatycznie ustawił modem na taki
tryb, który linux'y lubią najbardziej? Okazało się, że istnieje i w tym wpisie zostanie on
przestawiony.

<!--more-->
## Czy mogę zrezygnować z pakietu usb-modeswitch

Na dobrą sprawę to nie wiem czy wszystkie modemy da się dostosować w sposób opisany poniżej ale
jeśli taki modem obsługuje polecenia AT, a większość z nich obsługuje, to bez problemu można
odinstalować ten cały pakiet `usb-modeswitch` i przy pomocy poleceń AT przestawić tryb modemu na
stałe. Ustawienia będą honorowane między kolejnymi restartami systemu, bo zmiany zostaną zapisane
bezpośrednio w modemie. Trzeba to wziąć pod uwagę, gdy chcemy uruchamiać modem na windowsach.

## Zmiana trybu modemu

Polecenie, które nasz modem musi umieć obsłużyć to `AT^SETPORT` . To w jaki sposób będziemy próbować
to polecenie przesłać do modemu zależy głównie od nas samych. Możemy bezpośrednio zapisać urządzenie
`/dev/usbTTY0` , skorzystać z dedykowanego oprogramowania do obsługi modemów, np. `cu` , `socat` czy
też `minicom` . Możemy także doinstalować `wvdial` i przy pomocy jego pliku konfiguracyjnego
porozmawiać sobie z modemem. W tym przypadku został wykorzystany ostatni sposób.

Edytujemy zatem plik `/etc/wvdial.conf` i dodajemy te poniższe bloki kodu:

    [Dialer Defaults]
    Modem Type = Analog Modem
    Modem = /dev/ttyUSB1
    ISDN = 0
    Baud = 9600
    Dial Command = ATDT
    Dial Attempts = 3
    Dial Timeout = 30
    Auto Reconnect = 1
    Stupid mode = 1
    Auto DNS = 0
    Check DNS = 1
    DNS Test1 = wp.pl
    DNS Test2 = google.com
    Check Def Route = 1
    Idle Seconds = 0
    Phone = *99#
    Init1 = ATZ
    Init2 = ATQ0 V1 E0 H0 S0=0

    [Dialer modem-start]
    Init1 = AT+CFUN=1

    [Dialer modem-stop]
    Modem = /dev/ttyUSB0
    Init1 = AT+CFUN=0

    [Dialer info-device]
    Modem = /dev/ttyUSB0
    Init5 = AT^SETPORT=?

    [Dialer info-config]
    Modem = /dev/ttyUSB0
    Init5 = AT^SETPORT?

    [Dialer set-iface]
    Init4 = AT^SETPORT="FF;12,10,16,A2"
    Init5 = AT^RESET

    [Dialer red_bull_mobile-lte]
    Init4 = AT^SYSCFGEX="03",3FFFFFFF,1,2,800C5,,
    Init6 = AT+CGDCONT=1,"IPV4V6","internet"
    Username = "blank"
    Password = "blank"

Zapisujemy konfigurację, podpinamy modem do portu USB i wydajemy w terminalu to poniższe polecenie:

    # wvdial modem-start info-device
    ...
    --> Sending: AT^SETPORT=?

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

Wyżej widzimy listę portów modemu HUAWEI E3372s-153. By sprawdzić aktualną ich konfigurację,
wpisujemy w terminal poniższe polecenie:

    # wvdial modem-start info-config
    ...
    --> Sending: AT^SETPORT?

    ^SETPORT:A1,A2;12,10,16,A1,A2

Składnia wyjścia powyższego polecenia jest następująca: `^SETPORT:<primary composition>;<secondary
composition>"` . Mamy tutaj dwie kompozycje, jedna z nich jest aplikowana w jednym trybie modemu,
druga zaś w drugim. Niewiele osób wie, że ten pierwszy tryb można całkowicie wyłączyć, co zapobiega
przełączaniu modemu na drugi tryb. W efekcie czego, modem od razu jest startowany w drugim trybie i
odpada potrzeba stosowania pakietu `usb-modeswitch` . By wyłączyć ten pierwszy tryb, który jest dla
linux'a zupełnie zbędny, w terminalu wydajemy to poniższe polecenie:

    # wvdial modem-start set-iface
    ...
    --> Sending: AT^SETPORT="FF;12,10,16,A2"
    OK
    --> Sending: AT^RESET
    OK

Kluczową rolę w bloku `[Dialer set-iface]` odgrywa `FF` w pierwszym trybie modemu. Możemy także
wyłączyć CD-ROM ( `A1` ) w drugim trybie, bo i tak niezbyt się on przydaje, a w logu generuje
tylko sporo błędów. Po chwili modem powinien się uruchomić ponownie, a w logu powinniśmy
zaobserwować te poniższe komunikaty:

    kernel: usb 2-1.3: new high-speed USB device number 11 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=12d1, idProduct=15b6
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: HUAWEI_MOBILE
    kernel: usb 2-1.3: Manufacturer: HUAWEI_MOBILE
    kernel: usb 2-1.3: SerialNumber: 0123456789ABCDEF
    kernel: option 2-1.3:1.0: GSM modem (1-port) converter detected
    kernel: usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB0
    kernel: option 2-1.3:1.1: GSM modem (1-port) converter detected
    kernel: usb 2-1.3: GSM modem (1-port) converter now attached to ttyUSB1
    kernel: huawei_cdc_ncm 2-1.3:1.2: MAC-Address: 00:1e:10:1f:00:00
    kernel: huawei_cdc_ncm 2-1.3:1.2: setting rx_max = 16384
    kernel: huawei_cdc_ncm 2-1.3:1.2: NDP will be placed at end of frame for this device.
    kernel: huawei_cdc_ncm 2-1.3:1.2: cdc-wdm0: USB WDM device
    kernel: huawei_cdc_ncm 2-1.3:1.2 wwan0: register 'huawei_cdc_ncm' at usb-0000:00:1d.0-1.3, Huawei CDC NCM device, 00:1e:10:1f:00:00
    kernel: usb-storage 2-1.3:1.3: USB Mass Storage device detected
    kernel: scsi host10: usb-storage 2-1.3:1.3
    mtp-probe[14361]: checking bus 2, device 11: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3"
    mtp-probe[14361]: bus: 2, device: 11 was not an MTP device
    kernel: scsi 10:0:0:0: Direct-Access     HUAWEI   TF CARD Storage  2.31 PQ: 0 ANSI: 2
    kernel: sd 10:0:0:0: Attached scsi generic sg2 type 0
    kernel: sd 10:0:0:0: [sdb] Attached SCSI removable disk

Jak widać, nie ma przełączania trybu. Połączenie z siecią nawiązujemy w standardowy sposób, tj:

    # wvdial modem-start red_bull_mobile-lte

## Modemy obsługujące tryb NDIS (NCM)

Może się zdarzyć tak, że w nasze łapki wpadnie modem, który może działać w trybie NDIS (NCM). W
takim przypadku możemy także zrezygnować z instalacji pakietu `wvdial` i skonfigurować sobie
interfejs `wwan0` ręcznie. [Opis konfiguracji modemu w trybie
NDIS](/post/konfiguracja-modemu-lte-w-trybie-ndis-ncm/) został przedstawiony w
osobnym artykule.
