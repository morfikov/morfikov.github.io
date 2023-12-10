---
author: Morfik
categories:
- Linux
date: "2019-06-06T16:10:15Z"
published: true
status: publish
tags:
- debian
- udev
- mysz
- usb
GHissueID: 598
title: Wybudzanie linux'a ze stanu uśpienia za sprawą myszy
---

Parę dni temu na jednych z forów, które czasem odwiedzam, [pojawił się wątek][1] dotyczący problemu
jaki może nieść ze sobą budzenie linux'a ze stanu uśpienia/wstrzymania (Suspend to RAM, STR) za
sprawą myszy. O ile w przypadku klawiatury sprawa wybudzania komputera zdaje się być dość oczywista,
to w przypadku tego małego gryzonia już niekoniecznie, bo wystarczy lekko mysz przemieścić po
blacie stołu czy innego biurka i system się nam wybudzi. Część komputerów ma stosowne opcje w
BIOS/UEFI i można za ich sprawą skonfigurować to jakie urządzenia będą mieć możliwość wybudzania
systemu. Niekiedy jednak, opcje w BIOS są tak ubogie, że nie mamy możliwości skonfigurowania tego
aspektu pracy naszej maszyny. Trzeba zatem w nieco inny sposób podejść do tego zagadnienia. Na
necie można się spotkać z radami odnośnie zapisu pliku `/proc/acpi/wakeup` przez przesłanie do
niego czteroznakowych kodów, np. `EHC1` czy `USB1` . Takie rozwiązanie może nieść ze sobą negatywne
konsekwencje i powinno się go unikać. Lepszym rozwiązaniem jest napisanie reguły dla UDEV'a dla
konkretnego urządzenia, gdzie będziemy mogli łatwo sterować (przez plik `power/wakeup` ) tym czy
dane urządzenie ma mieć możliwość wybudzania systemu czy też nie.

<!--more-->
## Dlaczego /proc/acpi/wakeup to zły pomysł

Wypadałoby pierw zająć się tym co możemy znaleźć w pliku `/proc/acpi/wakeup` . W zasadzie w każdym
przypadku, wyjście poniższego polecenia zwróci inny wynik ale część rzeczy zawsze będzie wspólna i
można się do nich odnieść. Odpalamy zatem terminal i wpisujemy w nim to poniższe polecenie:

    # cat /proc/acpi/wakeup
    Device  S-state   Status   Sysfs node
    P0P2      S4    *disabled
    PEGP      S4    *disabled
    P0P1      S4    *disabled  pci:0000:00:1e.0
    EHC1      S3    *enabled   pci:0000:00:1d.0
    EHC2      S3    *enabled   pci:0000:00:1a.0
    HDEF      S0    *disabled  pci:0000:00:1b.0
    RP01      S4    *disabled  pci:0000:00:1c.0
    PXSX      S4    *disabled  pci:0000:02:00.0
    RP02      S4    *disabled
    PXSX      S4    *disabled
    RP03      S4    *disabled  pci:0000:00:1c.2
    PXSX      S4    *disabled  pci:0000:03:00.0
    RP04      S4    *disabled
    PXSX      S4    *disabled
    RP05      S4    *disabled
    PXSX      S4    *disabled
    RP07      S4    *disabled
    PXSX      S4    *disabled
    RP08      S4    *disabled
    PXSX      S4    *disabled
    PEG3      S4    *disabled
    PEG5      S4    *disabled
    PEG6      S4    *disabled

W kolumnie `Device` są czteroznakowe oznaczenia urządzeń w komputerze. W kolumnie `S-state` są
[stany uśpienia według standardu ACPI][2], w których dane urządzenie może wybudzić system. Nas
interesuje jedynie stan `S3` , bo to on odpowiada za uśpienie maszyny (stan `S4` , to hibernacja) i
w tym przypadku mamy jedynie `EHC1` oraz `EHC2` . W `Status` jest zawarta informacja na temat tego
czy dane urządzenie może wybudzać system ze stanu uśpienia, a w `Sysfs node` znajduje się node
urządzenia (w katalogu `/sys/` ).

### Kody urządzeń

To co będzie znajdować się w kolumnie `Device` w pliku `/proc/acpi/wakeup` da nam wskazówkę z jakim
typem urządzenia możemy mieć do czynienia. Oznaczenia te jednak są brane z DSDT ([Differentiated
System Description Table][3]) BIOS'u płyt głównych komputera, a ich producenci mogą stosować
dowolne oznaczenia, bo nie ma tutaj żadnego utrwalonego standardu. W tym przypadku mieliśmy takie
oznaczenia:

- `EHC*` - kontroler USB 2.0 (EHCI). Oznaczane czasem `USB*` .
- `HDEF` - specyficzny dla producenta. Coś wspólnego z High DEFinition, urządzenie audio.
- `P0P*` - mostek PCI (PCI bridge).
- `PEG*` - slot PCI Express dla grafiki.
- `RP0*` - slot PCI Express. Oznaczane czasem `EXP*` . Dla każdego wpisu `RP0*` jest odpowiadający
mu wpis `PXSX` .

Na necie znalazłem jeszcze te poniższe:

- `PS2K` - klawiatura PS/2.
- `PS2M` - mysz PS/2.
- `PWRB` - przycisk Power.
- `LID`  - pokrywa laptopa.
- `XHC`  - kontroler USB 3.0 (XHCI).
- `IGBE` - kontroler LAN (Integrated GigaBit Ethernet).

Jeśli w kolumnie `Sysfs node` widnieje node urządzenia, to przy pomocy `lspci` możemy ustalić co to
jest za urządzenie i próbować domyśleć się w jaki sposób mogłoby ono wybudzić system ze stanu
uśpienia.

## EHC*/USB*/XHC, a porty USB

Jako, że chcemy się uporać jedynie z problemem wybudzania systemu ze stanu uśpienia za sprawą myszy
USB, to interesują nas tylko wpisy w pliku `/proc/acpi/wakeup` odnoszące się do `EHC*` , `USB*` i
`XHC` , w zależności, które z nich posiadamy.

W tym przypadku mamy do czynienia z laptopem, który ma 3 zewnętrzne porty USB ale w pliku
`/proc/acpi/wakeup` są tylko `EHC1` oraz `EHC2` . Te oznaczenia niekoniecznie muszą się odnosić do
określonego portu USB, tak jak mamy w tym przypadku. Jeśli się rzuci okiem na wyjście `lsusb` , to
można zobaczyć coś na poniższy wzór:

    # lsusb -t
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M

Mamy tu dwie szyny USB. Za obsługę pierwszej szyny USB odpowiada kontroler `EHC1` , a za obsługę
drugiej `EHC2` . Pierwsza szyna obsługuje wszystkie zewnętrzne porty USB (te do których się
podłącza różne urządzenia). Druga szyna USB obsługuje zaś urządzenia wbudowane w laptop, w tym
przypadku na pewno webcam (być może coś jeszcze).

Przełączając to samo urządzenie po każdym z portów laptopa, zmianie ulegną jedynie numery portów na
szynie pierwszej (wyżej `Bus 02`), co wygląda tak:

    # lsusb -t
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
            |__ Port 3: Dev 50, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/3p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M

`Port 3` przy urządzeniu oznacza, że to urządzenie zostało wpięte w trzeci port USB na konkretnej
szynie. Jakby wpiąć to urządzenie w drugi port USB komputera, to wyżej mielibyśmy `Port 2`, wciąż
na tej samej szynie USB.

## Wyłączenie wybudzania systemu na danym porcie USB

Część ludzi, by uporać się z wybudzaniem linux'a za sprawą myszy, przesyła do pliku
`/proc/acpi/wakeup` odpowiedni kod urządzenia (w tym przypadku byłoby to `EHC1` lub `EHC2` , albo
oba z nich) w poniższy sposób:

    # echo "EHC1" > /proc/acpi/wakeup

Niemniej jednak, jeśli wyłączymy na kontrolerze USB możliwość wybudzania maszyny mając szereg
portów USB na podpiętej do niego szynie, to żadne urządzenie podłączone do portów USB tego laptopa
nie będzie w stanie już go wybudzić, bez znaczenia czy to będzie klawiatura, mysz czy cokolwiek
innego. Dlatego tego typu rozwiązanie upośledzające kontroler bym odradzał. Nawet jeśli ktoś
chciałby się w to bawić, to w dalszym ciągu czeka go napisanie jakiegoś skryptu, który będzie
inicjowany za każdym razem, gdy wartości w pliku `/proc/acpi/wakeup` będą ulegać zmianie, np. przy
starcie systemu.

### Reguła dla UDEV'a

Zamiast podejścia opisanego wyżej, lepszym rozwiązaniem jest napisanie reguły dla UDEV'a, która
skonfiguruje odpowiednio parametry urządzenia bez ruszania samego kontrolera USB. Takie rozwiązanie
daje nam możliwość określenia, które z urządzeń USB mają mieć możliwość wybudzania systemu, a które
nie. Tworzymy zatem plik `99-local.rules` w katalogu `/etc/udev/rules.d/` i dodajemy do niego
poniższą zawartość:

    ACTION=="add", SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", \
        ATTRS{idVendor}=="09da", ATTRS{idProduct}=="000a",  \
        RUN+="/bin/sh -c 'echo disabled > /sys$env{DEVPATH}/power/wakeup'"

By ta reguła zadziałała, potrzebne nam są pewne informacje, które trzeba w niej uwzględnić. Zwykle
pierwsza linijka będzie taka sama w przypadku każdej myszy USB. Natomiast musimy określić numery
sprzętu ( `idVendor` oraz `idProduct` ) podpierając się wyjściem `lsusb` , przykładowo:

    # lsusb
    Bus 002 Device 085: ID 09da:000a ... Mouse ...
    ...

Jeśli chcielibyśmy mieć mniej lub bardziej zwięzłe reguły, to zawsze możemy uruchomić monitor
UDEV'a i popatrzeć na to co będzie nam się pojawiać podczas podłączania myszy do portu USB:

    # udevadm monitor --property

W regule mamy jeszcze linijkę z `RUN+=` , która w przypadku dopasowania sprzętu ma uruchomić
polecenie przesyłające `disabled` do pliku `power/wakeup` w ścieżce tego konkretnego urządzenia.
Ścieżka jest wyciągana zaś ze zmiennych środowiskowych UDEV'a:

    # udevadm info --path=/sys/class/input/mouse0 --attribute-walk
    ...
    looking at parent device '/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3':
    ...
    #  udevadm info --path=/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3
    ...
    E: DEVPATH=//devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3
    ...
    E: DEVTYPE=usb_device
    ...
    E: SUBSYSTEM=usb
    ...
    E: ID_VENDOR_ID=09da
    ...
    E: ID_MODEL_ID=000a
    ...

Po dodaniu reguły trzeba jeszcze przeładować politykę UDEV'a lub uruchomić komputer ponownie, by
zmiany zaczęły obowiązywać:

    # udevadm control --reload

### Brak pliku power/wakeup

Pełna ścieżka do urządzenia była określona w postaci
`/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/2-1.3:1.0/0003:09DA:000A.0061/input/input375/mouse0`
ale pod tą lokalizacją nie było żadnego pliku `power/wakeup` . Ten plik występuje nieco wcześniej w
`/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/` . Ta ścieżka odnosi się do portu USB, a nie do
konkretnego urządzenia podpinanego do portu USB.

By uniknąć statycznej konfiguracji portów USB, w poleceniu określonym przez `RUN+=` w naszej regule
mieliśmy ścieżkę w postaci `/sys$env{DEVPATH}/power/wakeup` i ten `$env{DEVPATH}` odpowiada za
wykrytą przez UDEV ścieżkę przy podłączaniu urządzenia, która w tym przypadku ma postać
`/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/` . Jeśli podłączymy naszą myszę w inny port USB,
to ta ścieżka ulegnie zmianie ale to nie ma już większego znaczenia.

Podobnie sprawa wygląda w przypadku podłączenia innego urządzenia do tego samego portu USB. Jeśli
jego numery identyfikacyjne będą inne od tych, którymi włada nasz gryzoń, to plik `power/wakeup`
nie zostanie w żaden sposób ruszony, a my za sprawą tego urządzenia będziemy jak najbardziej w
stanie obudzić naszego linux'a.



[1]: https://forum.linuxmint.pl/showthread.php?tid=323
[2]: https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface#Global_states
[3]: https://wiki.archlinux.org/index.php/DSDT
