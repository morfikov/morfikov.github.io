---
author: Morfik
categories:
- Linux
date: "2015-06-23T20:23:09Z"
date_gmt: 2015-06-23 18:23:09 +0200
published: true
status: publish
tags:
- pliki/foldery
- usb
title: Struktura plików urządzeń usb w katalogu /sys/
---

Jeśli próbowaliśmy odnaleźć odpowiednią ścieżkę do urządzenia usb w linuxowym drzewie katalogów, to
wiemy, że nie jest to łatwe zadanie. Mamy, co prawda, do dyspozycji polecenie `lsusb` ale ono nie
daje nam precyzyjnych informacji na temat tego gdzie dokładnie w katalogu `/sys/` znajdują się
określone urządzenia. Na necie natrafiłem na [FAQ](http://www.linux-usb.org/) dotyczący tego
zagadnienia i postanowiłem napisać kilka zdań o tym jak zinterpretować ciągi typu `2-1.1.2:1.1` oraz
jakie to może być urządzenie.

<!--more-->
## Schemat obsługiwanych urządzeń usb

Obecnie urządzenia usb są wszędzie i prawdopodobieństwo, że mamy jakieś podpięte akurat w tej chwili
do swojego komputera jest nie mal bliskie jedności. Na linuxach możemy sprawdzić listę takich
urządzeń wydając w terminalu polecenie `lsusb` . U mnie ten wynik wygląda mniej więcej tak:

    # lsusb
    Bus 002 Device 059: ID 046d:c30f Logitech, Inc. Logicool HID-Compliant Keyboard (106 key)
    Bus 002 Device 058: ID 09da:000a A4 Tech Co., Ltd Optical Mouse Opto 510D
    Bus 002 Device 053: ID 1a40:0101 Terminus Technology Inc. 4-Port HUB
    Bus 002 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 001 Device 003: ID 5986:0149 Acer, Inc
    Bus 001 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

Jeśli zajrzymy do katalogu `/sys/bus/usb/devices/` , to zobaczymy tam poniższe pliki:

    # ls -a /sys/bus/usb/devices/
    ./   1-0:1.0  1-1.3      1-1.3:1.1  2-0:1.0  2-1.1    2-1.1.1:1.0  2-1.1.2:1.0  2-1.1:1.0  usb1
    ../  1-1      1-1.3:1.0  1-1:1.0    2-1      2-1.1.1  2-1.1.2      2-1.1.2:1.1  2-1:1.0    usb2

Jak teraz powiązać te urządzenia z `lsusb` z powyższymi plikami o dość dziwnym zapisie?

## Interpretacja nazw plików urządzeń usb

Przede wszystkim, musimy zobaczyć schemat ułożenia szyn i portów usb w systemie. Możemy tego dokonać
również przy pomocy polecenia `lsusb` , tylko tym razem z dodajmy opcję `-t` :

    # lsusb -t
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
            |__ Port 3: Dev 3, If 0, Class=Video, Driver=uvcvideo, 480M
            |__ Port 3: Dev 3, If 1, Class=Video, Driver=uvcvideo, 480M
    
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
            |__ Port 1: Dev 53, If 0, Class=Hub, Driver=hub/4p, 480M
                |__ Port 1: Dev 58, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
                |__ Port 2: Dev 59, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
                |__ Port 2: Dev 59, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M

W katalogu `/sys/bus/usb/devices/` mieliśmy dwa kontrolery usb: `usb1` oraz `usb2` . Konkretnie są
to główne huby powiązane z kontrolerami. Numerki zaś odpowiadają za numerki szyn i jak możemy
zobaczyć na schemacie wyżej, również są dwie szyny. Jeśli się przyjrzymy bliżej, to zobaczymy, że
każdy z powyższych plików ma charakterystyczny zapis: `bus`-`port`.`port`.`port` . Najpierw mamy
numerek szyny oddzielony od sekwencji portów za pomocą `-` . Każdy z portów jest oddzielony od
siebie za pomocą `.` . Dwa z tych powyższych plików, to specjalne przypadki: `1-0:1.0` oraz
`2-0:1.0` . Odnoszą się one do interfejsów głównych hubów.

Weźmy zatem jakieś przykładowe urządzenie, np. `2-1` . Jest ono wpięte na szynę `2` i w jej port `1`
. W powyższym przypadku okazuje się, że to urządzenie to hub ( `Class=Hub` ). Następnie w port `1`
tego huba jest wpięte inne urządzenie i to również się okazuje być kolejny hub ( `2-1.1` ). W porty
tego ostatniego huba są wpięte dwa urządzenia o numerkach `2-1.1.1` oraz `2-1.1.2` .

Zatem mamy rozpisane same urządzenia ale za numerkami urządzeń, mamy także `:` i cyferki oddzielone
za pomocą `.` . W tym przypadku są to `:1.0` oraz `:1.1` . Co one oznaczają? Cyfra przed `.` to
numer konfiguracyjny, zaś ta za nią to numer interfejsu. Jeśli policzymy uważnie to doliczymy się 7
plików mających w nazwie `:1.0` oraz trzech mających `:1.1` . Porównując to ze schematem wyżej,
odpowiada to prędkości portów `480M` i `1.5M` .

Huby nie mogą mieć więcej niż jeden interfejs, natomiast jeśli chodzi o inne urządzenia, to ta
reguła się do nich nie stosuje. Urządzenia mogą także posiadać wiele konfiguracji. W moim przypadku
mam podpięte dwa urządzenia na usb v1.1 (prędkość 1.5M) ale moja klawiatura multimedialna ma dwa
interfejsy `If 0` oraz `If 1` . Zatem, w schemacie widocznym wyżej są 3 urządzenia na usb v1.1, a
nie dwa. Warto o tym pamiętać, gdyby liczba urządzeń się nie zgadzała.

Dokładniejsze informacje można wyciągnąć za pomocą narzędzia `usb-devices` . Poniżej są informacje
jakie zostały zwrócone w stosunku do mojej klawiatury:

    # usb-devices
    ...
    T:  Bus=02 Lev=03 Prnt=53 Port=01 Cnt=02 Dev#= 59 Spd=1.5 MxCh= 0
    D:  Ver= 1.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS= 8 #Cfgs=  1
    P:  Vendor=046d ProdID=c30f Rev=23.00
    S:  Manufacturer=Logitech
    S:  Product=Logitech USB Keyboard
    C:  #Ifs= 2 Cfg#= 1 Atr=a0 MxPwr=100mA
    I:  If#= 0 Alt= 0 #EPs= 1 Cls=03(HID  ) Sub=01 Prot=01 Driver=usbhid
    I:  If#= 1 Alt= 0 #EPs= 1 Cls=03(HID  ) Sub=00 Prot=00 Driver=usbhid

Wszystkie powyższe parametry są dokładnie opisane
[tutaj](http://www.linux-usb.org/USB-guide/c607.html#AEN609). Raczej nie powinno być problemów z ich
analizą. To co zasługuje na największą uwagę to linijki zaczynające się od `C:` oraz `I:` . W
pierwszej z nich mamy określone `#Ifs= 2 Cfg#= 1` , czyli dwa interfejsy i jedna konfiguracja. Z
kolei dwie ostatnie linijki to specyfikacja każdego z interfejsu. Jak widzimy wyżej, oba z nich mają
przypisany sterownik `usbhid` i w przypadku gdy jakieś urządzenie posiada kilka interfejsów, można
dla każdego z nich ustawić osobny sterownik.
