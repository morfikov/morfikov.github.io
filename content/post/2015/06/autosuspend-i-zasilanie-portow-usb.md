---
author: Morfik
categories:
- Linux
date: "2015-06-23T15:34:50Z"
date_gmt: 2015-06-23 13:34:50 +0200
published: true
status: publish
tags:
- moduły-kernela
- usb
title: Autosuspend i zasilanie portów usb
---

Kernel w linuxie odcina zasilanie urządzeniom podpiętym do portów usb jeśli sterownik wspiera tego
typu możliwość oraz samo urządzenie nie jest używane przez pewien okres czasu. W taki oto sposób,
jeśli podłączymy, np. zewnętrzną klawiaturę usb do laptopa, możemy zaobserwować, że przy pisaniu
tekstu gubiony jest zwykle pierwszy znak. Może i klawiatura po przyciśnięciu klawisza wyszła ze
stanu bezczynności ale system nie zareagował na tyle szybko by złapać sygnał przycisku. Na necie
ludzie piszą, że jest to problem niekompatybilności urządzeń i tego typu sytuacja nie powinna się
zdarzać. Jeśli jednak natrafiliśmy na klawiaturę czy myszę, która cierpi z powodu automatycznego
zawieszania jej zasilania, możemy wyłączyć ten ficzer zupełnie.

<!--more-->
## Określanie statusu autosuspend

By się przekonać jak wyglądają ustawienia dla poszczególnych urządzeń usb w systemie, sprawdźmy co
zwracają te dwa poniższe polecenia:

    # cat /sys/bus/usb/devices/*/power/autosuspend_delay_ms
    2000
    0
    2000
    2000
    0
    0
    0
    0

    # cat /sys/bus/usb/devices/usb*/power/control
    auto
    auto

Zgodnie z [dokumentacją kernela](https://www.kernel.org/doc/Documentation/usb/power-management.txt)
, jeśli parametr `autosuspend_delay_ms` jest ustawiony na 0, to zawieszenie zasilania następuje
natychmiast po przejściu urządzenia w stan IDLE. Jeśli jest tam wartość większa od zera, oznacza to
czas po jakim zasilanie zostanie odłączone. W tym przypadku moja klawiatura łapie się na `2000`
milisekund, czyli po 2 sekundach od ostatniego wciśnięcia przycisku, zostanie jej zawieszone
zasilanie.

By wyłączyć funkcję autosuspend, musimy ustawić w pliku `autosuspend_delay_ms` wartość `-1` . Możemy
tego dokonać na wiele sposób. Począwszy od zwykłego zapisania tego pliku przy pomocy `echo` ,
skończywszy na edycji parametrów samego modułu odpowiedzialnego za komunikację z urządzeniami usb.

## Wyłączenie funkcji autosuspend dla określonych urządzeń

Jeśli chcemy wyłączyć zawieszanie zasilania jedynie dla określonych urządzeń, to najlepszym wyjściem
jest napisanie krótkiej [reguły dla
udev'a]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/). By tego dokonać musimy
znać dwa parametry urządzenia: `idVendor` oraz `idProduct` . Oba z nich możemy odczytać z wyniku
`lsusb` :

    $ lsusb
    Bus 002 Device 005: ID 046d:c30f Logitech, Inc. Logicool HID-Compliant Keyboard (106 key)

Mamy tam ciąg `046d:c30f` , który odpowiada `idVendor:idProduct` . By teraz napisać regułę dla
udev'a, tworzymy plik `/etc/udev/rules.d/95-keyboard.rules` i dodajemy tam poniższy kod:

    ACTION=="add", SUBSYSTEM=="usb", \
          ATTR{idVendor}=="046d", \
          ATTR{idProduct}=="c30f", \
          TEST=="power/autosuspend_delay_ms", \
                ATTR{power/autosuspend_delay_ms}="-1"

Zapisujemy plik i przeładowujemy bazę urządzeń:

    # udevadm control --reload

Od tego momentu, za każdym razem gdy tylko podłączymy klawiaturę do portu usb, zostanie jej
przepisany parametr w pliku `autosuspend_delay_ms` na `-1` .

W przypadku gdy podepniemy taką klawiaturę do jednego z portów huba usb, to nie tylko jej zostanie
wyłączony mechanizm zawieszania zasilania ale również i wszystkim pozostałym urządzeniom wpiętym do
tego huba.

## Globalne wyłączenie mechanizmu autosuspend

Jako, że ja niezbyt przepadam za tymi mechanizmami, które mają na celu oszczędzanie energii (pół
biedy gdyby działały jak trzeba), to postanowiłem wyłączyć globalnie możliwość usypiania urządzeń
usb. W tym celu sprawa jest o wiele prostsza, bo sprowadza się jedynie do ustawienia odpowiedniego
parametru w module `usbcore` .

Najpierw jednak sprawdźmy jak obecnie wyglądają ustawienia modułu `usbcore` :

    # systool -v -m usbcore | grep -i autosuspend
        autosuspend         = "2"

Cyfra `2` w tym przypadku oznacza czas w sekundach, po którym zasilanie urządzenia zostanie
zawieszone. Musimy te parametr przestawić, podobnie jak w poprzednim przypadku, na `-1` . Tworzymy
zatem plik `/etc/modprobe.d/modules.conf` i dopisujemy tam poniższą linijkę:

    options usbcore autosuspend=-1

Jeśli ktoś nie chce wyłączać sobie całkowicie trybu oszczędzania energii w urządzeniach usb, to
zawsze zamiast `-1` może zdefiniować wartość `5` , `10` czy coś podobnego.
