---
author: Morfik
categories:
- OpenWRT
date: "2016-06-20T16:46:48Z"
date_gmt: 2016-06-20 14:46:48 +0200
published: true
status: publish
tags:
- pendrive
- udev
- chaos-calmer
- router
- modem
- huawei
- e3372
GHissueID: 347
title: Stałe nazwy urządzeń w OpenWRT (hotplug, udev)
---

Bezprzewodowy router WiFi to w miarę proste urządzenie, które w zasadzie realizuje kilka
podstawowych aspektów pracy sieci domowej. Wielu użytkownikom jednak jest nieustannie potrzebna
jakaś nowa funkcjonalność, której oryginalny firmware producenta nie oferuje. Dlatego też mamy do
dyspozycji OpenWRT będący minimalistyczną formą bardziej rozbudowanej dystrybucji linux'a. Może i
OpenWRT daje nam możliwość zaawansowanej konfiguracji naszej sieci ale tego typu opcja powoduje też
szereg problemów. Chodzi o to, że kernel dynamicznie tworzy nazwy dla wszystkich podłączanych
urządzeń do routera. W dużych dystrybucjach linux'a do ogarnięcia tych nazw wykorzystywany jest
UDEV. W przypadku OpenWRT też możemy skorzystać tego mechanizmu. Jeśli jednak mamy niewiele miejsca
na pamięci flash routera, to możemy też skorzystać ze zdarzeń hotplug. W tym wpisie postaramy się
przepisać nazwy pendrive/dysków twardych oraz modemów USB (LTE), tak by ich kolejność podłączania do
routera nie stwarzała problemów w konfiguracji.

<!--more-->
## Problematyczne nazwy urządzeń

Jak zostało już nadmienione w wstępie, kernel linux'a musi jakoś oznaczyć urządzenia, do których się
odwołują programy. Wszystkie te urządzenia znajdują się w katalogu `/dev/` . Po podpięciu do
routera, np. pendrive, system je rozpoznaje i udostępnia nam kilka interfejsów (pliki w katalogu
`/dev/` ). Ile tych interfejsów zostanie dodanych, to już zależy generalnie od sprzętu, który
zostanie podłączony. Niemniej jednak, wspólną cechą nazewniczą jest podbijanie numerka końcowego
danej klasy urządzeń. Dla przykładu weźmy sobie wspomniany pendrive lub dysk twardy. Pierwszy dysk
zostanie rozpoznany jako `sda` , drugi zaś `sdb` , itd. Każda partycja na takim nośniku jest
numerowana od 1 wzwyż. W taki sposób otrzymujemy `sda1` , `sda2` , `sdb1` , `sdb2` . Podobnie sprawa
wygląda w przypadku modemów USB. Z tym, że tutaj mamy nazwy typu `ttyUSB0` , `ttyUSB1` , itd.
Logiczne jest zatem, że gdy w konfiguracji demona, który działa na routerze, określimy ścieżkę do
urządzenia jako `/dev/ttyUSB1` , to przy innej kolejności podłączenia urządzeń, ten demon będzie
odwoływał się nie do tego sprzętu co trzeba.

Oczywiście, w przypadku routerów zwykle nie podłączamy całej masy dodatkowego sprzętu, tak jak to ma
miejsce przy standardowych PC. Niemnie jednak, OpenWRT zachęca do eksperymentów i ani się nie
obejrzymy, a taki router pod względem funkcjonalności zacznie przypominać nasz desktop. Wypadałoby
zatem pomyśleć już zawczasu, jak rozwiązać kwestię nazywania sprzętu oraz tego, by te nazwy były
stałe i powiązane z konkretnym urządzeniem. Wtedy mielibyśmy pewność, że konfiguracja, np. demona
wysyłającego i odbierającego SMS, działa bez względu na to w jakiej kolejności podłączymy urządzenia
do routera.

W OpenWRT mamy dostępny pakiet `udev` i można skorzystać z tego narzędzia pisząc odpowiednie reguły,
które przepiszą nazwy urządzeń. Niemniej jednak, nie jest to jedyny sposób. Szukając rozwiązania,
trafiłem także na [ten wpis](http://eko.one.pl/?p=openwrt-linkidoportowszeregowych). Zdaje się on
adresować po części ten problem, któremu i nam przyszło stawić czoła. W podlinkowanym artykule jest
jedynie informacja dotycząca bezpośrednio samych modemów USB ale nam to raczej nie powinno
przeszkadzać i myślę, że uda nam się wypracować jakieś rozsądne rozwiązanie ułatwiające operowanie
na urządzeniach podłączanych do routera domowego.

## Zmiana nazwy urządzeń w OpenWRT przy pomocy UDEV'a

Jednym z bardziej wyrafinowanych rozwiązań jest zaimplementowanie na routerze mechanizmu UDEV. Wiąże
się to z zainstalowaniem kilku pakietów ważących łącznie około 300-400 KiB. Jeśli mamy tyle miejsca
na flash'u, to wgrajmy sobie te poniższe pakiety (zależności zostaną automatycznie pociągnięte):

    # opkg update
    # opkg install udev blkid

W wersji Chaos Calmer jest jaki błąd związany z `blkid` . Chodzi generalnie o ścieżkę dostępu do
pliku wykonywalnego. W efekcie w logu UDEV'a pojawia się ten poniższy
    komunikat:

    [2252.968182] [2080] spawn_read: '/sbin/blkid -o udev -p /dev/sda'(err) 'failed to execute '/sbin/blkid' '/sbin/blkid -o udev -p /dev/sda': No such file or directory'

Plik o który chodzi to `/sbin/blkid` ale pakiet `blkid` wgrywa ten plik do `/usr/sbin/blkid` .
Możemy to w prosty sposób poprawić linkując ten plik do wskazanej lokalizacji:

    # ln -s /usr/sbin/blkid /sbin/blkid

Problem jest też w samym UDEV'ie, bo ten nie ma dostarczanego skryptu startowego. W efekcie demon
`udevd` nie jest uruchamiany wraz ze startem routera. Musimy zatem sobie taki [skrypt startowy
napisać](/post/skrypty-startowe-init-w-openwrt/). W tym celu tworzymy plik
`/etc/init.d/udevd` i wrzucamy do niego poniższy kod:

    #!/bin/sh /etc/rc.common

    START=05

    start() {
        /sbin/udevd --daemon
    }

Nadajemy skryptowi prawa wykonywania ( `chmod +x` ) oraz dodajemy go do autostartu:

    # /etc/init.d/udevd enable

### Zmiana nazwy pendrive i dysku twardego

Poniżej jest przykład rozpoznania przez system routera pendrive (log pochodzi z `logread`
    ):

    kern.info kernel: [   93.730000] usb 1-1.2: new high-speed USB device number 3 using ehci-platform
    kern.info kernel: usb-storage 1-1.2:1.0: USB Mass Storage device detected
    kern.info kernel: scsi host0: usb-storage 1-1.2:1.0
    kern.notice kernel: scsi 0:0:0:0: Direct-Access     Kingston DataTraveler 3.0 PMAP PQ: 0 ANSI: 6
    kern.notice kernel: sd 0:0:0:0: [sda] 15360000 512-byte logical blocks: (7.86 GB/7.32 GiB)
    kern.notice kernel: sd 0:0:0:0: [sda] Write Protect is off
    kern.debug kernel: sd 0:0:0:0: [sda] Mode Sense: 23 00 00 00
    kern.err kernel: sd 0:0:0:0: [sda] No Caching mode page found
    kern.err kernel: sd 0:0:0:0: [sda] Assuming drive cache: write through
    kern.info kernel:  sda: sda1 sda2 sda3
    kern.notice kernel: sd 0:0:0:0: [sda] Attached SCSI removable disk

Mamy zatem dysk `sda` z trzema partycjami `sda1` , `sda2` oraz `sda3` . W przypadku, gdy chcemy, by
te poszczególne partycje miały prefiks `king` , musimy [utworzyć stosowne reguły dla
UDEV'a](/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/). Nie będę tutaj opisywał
całej procedury tworzenia reguł ale jeśli ktoś jest ciekaw jak ten proces przebiega, to zapraszam
pod podlinkowany wyżej artykuł. Tutaj ograniczymy się jedynie do stworzenia pliku reguły w
`/etc/udev/rules.d/90-dysk.rules` . Do tego pliku wrzucamy poniższą zawartość:

    ACTION=="add", SUBSYSTEM=="block", \
        ENV{ID_SERIAL_SHORT}=="0019E06B9C8ABE41C7A2C3EC", \
        SYMLINK+="king%n"

Zmienna `ENV{ID_SERIAL_SHORT}` odpowiada za numer seryjny pendrive. Możemy go odczytać w poniższy
sposób:

    # udevadm info --name /dev/sda --attribute-walk | grep -i serial
        ATTRS{serial}=="0019E06B9C8ABE41C7A2C3EC"

Po uzupełnieniu pliku, musimy jeszcze przeładować reguły UDEV'a:

    # udevadm control --reload

No i oczywiście wyciągamy i podłączamy ponownie pendrive. Po chwili nowe nazwy urządzeń powinny
widnieć w katalogu `/dev/` :

    # ls -al /dev/king*
    lrwxrwxrwx    1 root     root             3 Jun 20 13:10 /dev/king -> sda
    lrwxrwxrwx    1 root     root             4 Jun 20 13:13 /dev/king1 -> sda1
    lrwxrwxrwx    1 root     root             4 Jun 20 13:14 /dev/king2 -> sda2
    lrwxrwxrwx    1 root     root             4 Jun 20 13:14 /dev/king3 -> sda3

### Zmiana nazwy modemu

Drugim popularnym urządzeniem podpinanym do portów USB routera jest modem LTE. W tym przypadku mamy
model Huawei E3372s-153 w wersji NON-HiLink. Jest on wykrywany przez OpenWRT w poniższy
    sposób:

    kern.info kernel: [ 4118.360000] usb 1-1.2: new high-speed USB device number 9 using ehci-platform
    kern.info kernel: option 1-1.2:1.0: GSM modem (1-port) converter detected
    kern.info kernel: usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB0
    kern.info kernel: option 1-1.2:1.1: GSM modem (1-port) converter detected
    kern.info kernel: usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB1
    kern.info kernel: usb-storage 1-1.2:1.3: USB Mass Storage device detected
    kern.info kernel: scsi host6: usb-storage 1-1.2:1.3
    kern.notice kernel: scsi 6:0:0:0: Direct-Access     HUAWEI   TF CARD Storage  2.31 PQ: 0 ANSI: 2
    kern.notice kernel: sd 6:0:0:0: [sda] Attached SCSI removable disk

Widzimy wyżej, że zostały utworzone dwa interfejsy `ttyUSB0` oraz `ttyUSB1` . To właśnie te dwie
nazwy musimy przepisać. Trzeba jednak pamiętać, że modem USB może mieć więcej interfejsów w
zależności od konfiguracji urządzenia. Tak czy inaczej, podobnie jak w przypadku pendrive, musimy
pokusić się o napisanie odpowiedniej reguły, która przepisze nazwy wszystkich interfejsów tego
urządzenia. Tworzymy zatem plik `/etc/udev/rules.d/90-modem.rules` i wrzucamy do niego poniższą
zawartość:

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
        ENV{ID_VENDOR_ID}=="12d1", \
        ENV{ID_MODEL_ID}=="15b6", \
        SYMLINK+="huawei-E3372-%n"

Zmienne `ENV{ID_VENDOR_ID}` oraz `ENV{ID_MODEL_ID}` możemy odczytać w poniższy sposób:

    # udevadm info --name /dev/ttyUSB0 --attribute-walk | egrep "idV|idP"
        ATTRS{idVendor}=="12d1"
        ATTRS{idProduct}=="15b6"
        ATTRS{idVendor}=="05e3"
        ATTRS{idProduct}=="0608"
        ATTRS{idVendor}=="1d6b"
        ATTRS{idProduct}=="0002"

Mamy tutaj kilka identyfikatorów. Nas jednak interesują te dwa pierwsze. Na koniec przeładowujemy
konfigurację UDEV'a

    # udevadm control --reload

Wyciągamy teraz modem LTE z portu USB i podłączamy go ponownie. W katalogu `/dev/` powinny widnieć
teraz poniższe urządzenia:

    /# ls -al /dev/ | grep huawei
    lrwxrwxrwx    1 root     root             7 Jun 20 14:22 huawei-E3372-0 -> ttyUSB0
    lrwxrwxrwx    1 root     root             7 Jun 20 14:22 huawei-E3372-1 -> ttyUSB1

## Zmiana nazwy urządzeń w OpenWRT przez zdarzenia hotplug

Jeśli nie mamy na flash'u routera wystarczającej ilości miejsca, to możemy odpuścić sobie sposób z
przepisywaniem nazw przy pomocy UDEV'a. Alternatywą jest skorzystanie ze [zdarzeń
hotplug](https://wiki.openwrt.org/doc/techref/hotplug). W OpenWRT, hotplug zastępuje mechanizm
UDEV'a, który zaimplementowaliśmy wyżej. Nie musimy też instalować żadnych dodatkowych pakietów.

Hotplug wykonuje skrypty zlokalizowane w katalogu `/etc/hotplug.d/` w momencie zaistnienia
określonego zdarzenia, np. podniesienie interfejsu sieciowego, przyciśnięcie jakiegoś przycisku na
routerze, czy też wykrycie jakiegoś urządzenia. W w/w katalogu mamy kilka podkatalogów. Jako, że my
chcemy przepisywać nazwy urządzeń podpiętych do portu USB, to interesuje nas katalog
`/etc/hotplug.d/usb/` (modemy) oraz `/etc/hotplug.d/block/` (dyski/pendrive). W zależności od
urządzenia, wybieramy odpowiedni folder i tworzymy w nim plik `10-env` . Następnie wrzucamy do
niego poniższą treść:

    #!/bin/sh
    echo "----" >> /tmp/env
    env >> /tmp/env

W pliku `/tmp/env` znajdować się będzie szereg zmiennych, w oparciu o które będziemy mogli napisać
skrypt.

### Zmiana nazwy modemu

Podłączamy teraz do portu USB modem LTE i zaglądamy do pliku `/tmp/env` :

    ----
    UDEV_LOG=3
    USER=root
    ACTION=add
    SHLVL=2
    HOME=/
    SEQNUM=497
    HOTPLUG_TYPE=usb
    DEVPATH=/devices/platform/ehci-platform/usb1/1-1/1-1.2/1-1.2:1.2
    DEVICENAME=1-1.2:1.2
    LOGNAME=root
    TERM=linux
    SUBSYSTEM=usb
    board=TL-WDR4300
    PATH=/usr/sbin:/usr/bin:/sbin:/bin
    MODALIAS=usb:v12D1p15B6d0102dc00dsc00dp00icFFisc03ip16in02
    TYPE=0/0/0
    INTERFACE=255/3/22
    PRODUCT=12d1/15b6/102
    PWD=/
    DEVTYPE=usb_interface

Widzimy tutaj szereg zmiennych, które są przekazywane do skryptu, który ma się wykonać w momencie
podłączenia modemu LTE do portu USB. W oparciu o te zmienne możemy zatem napisać instrukcję, która
będzie aplikowana jedynie w przypadku konkretnego urządzenia. Poniżej jest przykładowy kod, który
zapisujemy w `/etc/hotplug.d/usb/10-serial` :

    #!/bin/sh
    if [ "$DEVTYPE" = "usb_interface" ] && [ "$ACTION" = "add" ]; then
        for tty in /sys/$DEVPATH/ttyUSB*; do
            [ -d "$tty" ] || continue
            OLDD=${tty##*/}

            # to jest E3372s-153
            if [ "x$PRODUCT" = "x12d1/15b6/102" ]; then
                NEWD="modem_e3372s_"${DEVPATH##*.}
                rm /dev/$NEWD
                ln -s /dev/$OLDD /dev/$NEWD
            fi
        done
    fi

Instrukcja warunkowa pod `# to jest E3372s-153` odnosi się tylko tego konkretnego modemu. Jeśli mamy
kilka różnych modemów USB, musimy dodać kolejne instrukcje i odpowiednio dostosować warunek. Efektem
są poniższe dowiązania w katalogu `/dev/` :

    /# ls -al /dev/modem*
    lrwxrwxrwx    1 root     root            12 Jun 20 15:14 /dev/modem_e3372s_0 -> /dev/ttyUSB0
    lrwxrwxrwx    1 root     root            12 Jun 20 15:14 /dev/modem_e3372s_1 -> /dev/ttyUSB1

### Zmiana nazwy pendrive i dysku twardego

Jeśli chcemy przepisać nazwy urządzeń takich jak pendrive czy dyski twarde przy pomocy zdarzeń
hotplug, to musimy nieco bardziej wytężyć nasz umysł. Chodzi generalnie o to, że skrypt `10-env`
umieszczony w katalogu `/etc/hotplug.d/block/` nie zwróci nam ani numeru seryjnego urządzenia, ani
też żadnego identyfikatora (VendorID, ProductID). Poniżej są zmienne, które zostały zalogowane po
podłączeniu pendrive do routera:

    ----
    DEVNAME=sda3
    USER=root
    ACTION=add
    SHLVL=2
    HOME=/
    SEQNUM=502
    HOTPLUG_TYPE=block
    MAJOR=8
    DEVPATH=/devices/platform/ehci-platform/usb1/1-1/1-1.2/1-1.2:1.0/host0/target0:0:0/0:0:0:0/block/sda/sda3
    DEVICENAME=sda3
    LOGNAME=root
    TERM=linux
    SUBSYSTEM=block
    board=TL-WDR4300
    PATH=/usr/sbin:/usr/bin:/sbin:/bin
    MINOR=3
    PWD=/
    DEVTYPE=partition

Niemniej jednak, numer seryjny pendrive możemy wyciągnąć odpowiednio zmieniając ścieżkę w zmiennej
`$DEVPATH` . Mając numer seryjny, możemy napisać warunek. Poniżej przykładowy skrypt, który
umieszczamy w pod `/etc/hotplug.d/block/10-pen` :

    #!/bin/sh

    new_devpath="$(echo "$DEVPATH" | cut -d/ -f 1-7)"
    serial="$(cat /sys/$new_devpath/serial)"

    if [ "$serial" = "0019E06B9C8ABE41C7A2C3EC" ] && [ "$ACTION" = "add" ]; then

        old_dev="$DEVICENAME"
        new_dev="king"${DEVICENAME##sd[a-z]}

        rm /dev/$new_dev
        ln -s /dev/$old_dev /dev/$new_dev

    fi

### Problemy ze zdarzeniami hotplug

Wszystkie linki utworzone za pomocą mechanizmu hotplug nie są usuwane przy wyciąganiu urządzenia z
portu USB. W ten sposób inne urządzenie podpięte do portu USB może przejąc te linki i o tym trzeba
pamiętać. Niemniej jednak, gdy podłączymy urządzenie, do którego skrypt ma się odwołać, to te stare
linki zostaną zaktualizowane.
