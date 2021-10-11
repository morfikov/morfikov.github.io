---
author: Morfik
categories:
- Linux
date: "2016-04-09T18:50:19Z"
date_gmt: 2016-04-09 16:50:19 +0200
published: true
status: publish
tags:
- udev
- lte
- debian
- modem
- huawei
- e3372
GHissueID: 405
title: Zmiana nazwy interfejsu modemu ttyUSB0
---

Spora część osób posiada różnego rodzaju urządzenia do komunikacji GSM/UMTS/LTE. Mogą to być
smartfony, czy też modemy USB. Zwykle po podpięciu takiego urządzenia do portu USB, system wykrywa
je i oddaje nam do dyspozycji kilka interfejsów w katalogu `/dev/` . W przypadku modemu Huawei
E3372s-153 w wersji NON-HiLink, standardowo są dwa interfejsy: `ttyUSB0` oraz `ttyUSB1` . Gdy
podłączamy tylko jedno urządzenie, to nie mamy problemy z tymi nazwami. Co się jednak stanie w
przypadku, gdzie tych urządzeń będzie więcej i podepniemy je w losowej kolejności? Nawet jeśli
będziemy wiedzieć które interfejsy są od jakich urządzeń, to i tak trzeba będzie przepisywać pliki
konfiguracyjne różnych aplikacji pod kątem dostosowania tych nazw. Możemy jednak stworzyć unikalne
nazwy interfejsów w oparciu o [reguły udev'a](https://en.wikipedia.org/wiki/Udev) i tym zajmiemy się
w niniejszym wpisie.

<!--more-->
## Standardowe dowiązania do interfejsów modemu

Jak zaznaczyłem we wstępie, po podłączeniu urządzenia system tworzy kilka plików `/dev/ttyUSB*` .
Nie są to jednak jedyne pliki, przez które możemy komunikować się z modemem. Mamy również katalog
`/dev/serial/` , w którym są dwa podkatalogi: `by-id/` oraz `by-path/` . W tym przypadku struktura
plików modemu prezentuje się następująco:

    $ ls -al /dev/ttyUSB*
    crw-rw---- 1 root dialout 188, 0 2016-04-09 12:17:10 /dev/ttyUSB0
    crw-rw---- 1 root dialout 188, 1 2016-04-09 12:17:10 /dev/ttyUSB1

    $ ls -al /dev/serial/by-id
    lrwxrwxrwx 1 root root 13 2016-04-09 12:17:10 usb-HUAWEI_MOBILE_HUAWEI_MOBILE_0123456789ABCDEF-if00-port0 -> ../../ttyUSB0
    lrwxrwxrwx 1 root root 13 2016-04-09 12:17:10 usb-HUAWEI_MOBILE_HUAWEI_MOBILE_0123456789ABCDEF-if01-port0 -> ../../ttyUSB1

    $ ls -al /dev/serial/by-path
    lrwxrwxrwx 1 root root 13 2016-04-09 12:17:10 pci-0000:00:1d.0-usb-0:1.3:1.0-port0 -> ../../ttyUSB0
    lrwxrwxrwx 1 root root 13 2016-04-09 12:17:10 pci-0000:00:1d.0-usb-0:1.3:1.1-port0 -> ../../ttyUSB1

Widzimy zatem, że te dowiązania są w miarę unikatowe ale niezbyt przyjazne da użytkownika. My
postaramy się zaprojektować takie dowiązania, które będą niezależne od portu USB oraz będą miały
krótsze i łatwiejsze do zapamiętania nazwy.

## Uzyskiwanie informacji o modemie

By napisać regułę dla udev'a, potrzebne nam są pewne unikatowe informacje, które charakteryzują
konkretne urządzenie. Zwykle wystarczą nam VendorID i ProductID, czyli numerki producenta oraz
modelu urządzenia. W przypadku, gdy mamy kilka takich samych urządzeń, np. dwa modemy, to wtedy
możemy jeszcze postarać się o uzyskanie numeru seryjnego. Z reguły te wszystkie informacje można
wydobyć w poniższy sposób:

    # udevadm info --name /dev/ttyUSB0
    ...
    E: ID_MODEL_ID=15b6
    ...
    E: ID_SERIAL=HUAWEI_MOBILE_HUAWEI_MOBILE_0123456789ABCDEF
    E: ID_SERIAL_SHORT=0123456789ABCDEF
    ...
    E: ID_USB_INTERFACE_NUM=00
    ...
    E: ID_VENDOR_ID=12d1
    ...

Mając te powyższe informacje, możemy [napisać regułę udev'a dla tego
urządzenia](/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/).

## Tworzenie dowiązania do urządzenia ttyUSB0

Przechodzi zatem do katalogu `/etc/udev/rules.d/` i tworzymy w nim plik `80-modem.rules` . To co w
tym pliku wpiszemy zależeć będzie głównie od tych informacji uzyskanych powyżej. W tym przypadku
jest to:

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
          ENV{ID_VENDOR_ID}=="12d1", \
          ENV{ID_MODEL_ID}=="15b6", \
          ENV{ID_USB_INTERFACE_NUM}=="00" , \
          SYMLINK+="huawei-E3372-0"

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
          ENV{ID_VENDOR_ID}=="12d1", \
          ENV{ID_MODEL_ID}=="15b6", \
          ENV{ID_USB_INTERFACE_NUM}=="01" , \
          SYMLINK+="huawei-E3372-1"

Reguły są dwie, bo ten modem ma dwa interfejsy. Odradzam jednak stosowanie jednej reguły z zapisem
`huawei-E3372-%n` , gdzie `%n` to numer interfejsu widziany przez kernel. To jest ten sam numer,
który widnieje, np. w `ttyUSB0`, czyli `0` . W przypadku, gdy podłączymy urządzenia w innej
kolejności, ten numerek uległby nam zmianie, a tego nie chcemy. Dlatego posłużyliśmy się
`ENV{ID_USB_INTERFACE_NUM}` , który określa numer interfejsu wewnątrz tego urządzenia i na jego
podstawie tworzymy link o konkretnej nazwie. W ten sposób te linki będą zawsze mieć taką postać jak
określono powyżej.

## Testowanie reguł udev'a

By przetestować tak stworzoną regułę, musimy odnaleźć pierw to urządzenie w strukturze systemu
plików `sysfs` . Robimy to w poniższy sposób:

    # udevadm control --reload
    # udevadm info -q path -n /dev/ttyUSB0
    /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/2-1.3:1.0/ttyUSB0/tty/ttyUSB0

Następnie ten długi ciąg podajemy w `udevadm test` , przykładowo:

    # udevadm test /devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/2-1.3:1.0/ttyUSB0/tty/ttyUSB0
    ...
    Reading rules file: /etc/udev/rules.d/80-modem.rules
    ...
    LINK 'huawei-E3372-0' /etc/udev/rules.d/80-modem.rules:6
    ...
    creating symlink '/dev/huawei-E3372-0' to 'ttyUSB0'
    ...

Widzimy, że reguła została odczytana z pliku oraz, że link został stworzony. W tej chwili możemy
ponownie podłączyć urządzenie. Po chwili powinniśmy zobaczyć kilka dowiązań w katalogu `/dev/` :

    # ls -al /dev/huawei-E3372-*
    lrwxrwxrwx 1 root root 7 2016-04-09 15:12:09 /dev/huawei-E3372-0 -> ttyUSB2
    lrwxrwxrwx 1 root root 7 2016-04-09 15:12:09 /dev/huawei-E3372-1 -> ttyUSB3
