---
author: Morfik
categories:
- DD-WRT
date: "2016-09-08T19:56:52Z"
date_gmt: 2016-09-08 17:56:52 +0200
published: true
status: publish
tags:
- router
- dd-wrt
title: 'DD-WRT: Aktualizacja firmware do najnowszej wersji'
---

Firmware, który jest dostępny na [stronie
DD-WRT](https://www.dd-wrt.com/site/support/router-database) nie jest zbyt aktualny. Ten obraz,
który pobrałem dla swojego routera TL-WDR3600 jest datowany na rok 2013. Szukając informacji na ten
temat, okazało się, że [nowsze obrazy są dostępne w nieco innym
miejscu](ftp://ftp.dd-wrt.com/betas/). Różnica między tymi obrazami jest taka, że te pierwsze są w
wersji stabilnej, a te drugie w wersji beta i wypuszczane tak średnio raz na tydzień. Przydałoby się
zatem co jakiś czas zaktualizować firmware DD-WRT do nowszej wersji.

<!--more-->
## Reset routera do ustawień fabrycznych

Na wiki DD-WRT jest cała masa informacji na temat aktualizacji firmware do nowszej wersji. Możemy
tam znaleźć min. zalecenie wyczyszczenia ustawień, które są przechowywane w NVRAM przed
przystąpieniem do procesu aktualizacji (procedura Hard Reset 30/30/30). Jest tam też info, że ten
proces nie jest tym samym, co zresetowanie ustawień routera do domyślnych (factory defaults).

Okazuje się jednak, że ten mechanizm przywracania ustawień przed flash'owaniem routera nowym
firmware jest nieco przestarzały i obecnie się z niego nie korzysta, zwłaszcza na routerach mających
podzespoły Qualcomm Atheros, tak jak w tym przypadku. Generalnie rzecz biorąc, nie musimy sobie
zawracać głowy resetem ustawień przed wgraniem obrazu na router.

## Aktualizacja firmware DD-WRT przez panel administracyjny

Logujemy się zatem do panelu administracyjnego i robimy to z wykorzystaniem połączenia przewodowego.
Ubijmy na wszelki wypadek sieć WiFi w swoich laptopach, by czasem nie dokonywać aktualizacji za
pośrednictwem sieci bezprzewodowej. W panelu przechodzimy na zakładkę Administration => Firmware
Upgrade, gdzie wskazujemy plik obrazu z firmware. Nowsze obrazy dla routera TL-WDR3600 są
[tutaj](ftp://ftp.dd-wrt.com/betas/2016/09-01-2016-r30534/tplink_tl-wdr3600v1/). Interesuje nas plik
`tl-wdr3600-webflash.bin` jako, że aktualizujemy firmware przez panel web.

![](/img/2016/09/2.aktualizacja-firmware-dd-wrt-flash.png#huge)

Ten mechanizm aktualizacji oprogramowania routera daje nam również możliwość zachowania lub resetu
ustawień po procesie flash'owania. Ten krok jest wymagany, gdy wersja pobranego obrazu różni się
znacząco od wersji firmware obecnego w routerze, a tak jest w tym przypadku.

Wyżej na fotce mamy informację, że aktualizacja DD-WRT może zająć kilka minut i by nie odcinać
zasilania czy resetować routera do momentu zakończenia tego procesu. Jeśli w taki sposób byśmy
przerwali aktualizację firmware, to prawdopodobnie uwalimy router. Warto też zaznaczyć, że podczas
aktualizacji firmware nie będziemy mieć dostępu do internetu.

Po kliknięciu w przycisk Upgrade rozpocznie się flash'owanie routera:

![](/img/2016/09/3.aktualizacja-firmware-dd-wrt-proces.png#huge)

Po pomyślnej aktualizacji firmware, router powinien się zresetować, a my zobaczyć poniższą
informację:

![](/img/2016/09/4.aktualizacja-firmware-dd-wrt-proces.png#huge)

Widzimy wyżej, że ustawienia routera zostały również zresetowane do fabrycznych. Możemy kontynuować
proces przez kliknięcie w przycisk Continue. Na wiki DD-WRT jest informacja, by odczekać 5 minut
zanim klikniemy w ten przycisk, tylko nie wiem po co. Ja nie czekałem i nic złego się z tego powodu
nie stało. Po wciśnięciu przycisku powinien nas przywitać znajomy panel administracyjny:

![](/img/2016/09/1.aktualizacja-firmware-dd-wrt-panel-admina.png#huge)

Zmieniamy hasło i logujemy się do panelu administracyjnego. W prawym górnym rogu jest informacja o
aktualnej wersji używanego oprogramowania na routerze: `Firmware: DD-WRT v3.0-r30534 (09/01/16)` .
Zatem proces aktualizacji zakończył się powodzeniem.

## Aktualizacja firmware DD-WRT przez SSH/telnet

Innym sposobem aktualizacji firmware jest zalogowanie się na router z wykorzystaniem protokołu
telnet lub SSH. W takim przypadku, plik z obrazem firmware będziemy musieli przesłać na router do
jego pamięci RAM. Proces aktualizacji zarówno przez telnet jak i SSH jest taki sam, zatem opiszę go
na przykładzie protokołu telnet, bo jest on domyślnie włączony w DD-WRT.

Odpalamy zatem terminal i logujemy się na router wpisując w nim polecenie `telnet 192.168.1.1`
(adres trzeba sobie dostosować). Podajemy login i hasło (domyślnie `root`/`admin`) i po chwili
powinniśmy zostać zalogowani do
systemu:

![](/img/2016/09/5.dd-wrt-aktualizacja-firmware-terminal-telnet-ssh.png#big)

Plik obrazu możemy przesłać za pomocą `scp` lub też możemy go pobrać bezpośrednio do pamięci routera
korzystając z `wget` . Ten drugi sposób jest o wiele prostszy, dlatego też to z niego skorzystamy. W
terminalu wpisujemy poniższe polecenia:

    # cd /tmp
    root@ddwrt:/tmp# wget ftp://ftp.dd-wrt.com/betas/2016/09-01-2016-r30534/tplink_tl-wdr3600v1/tl-wdr3600-webflash.bin -O obraz.bin
    Connecting to ftp.dd-wrt.com (185.84.6.100:21)
    obraz.bin            100% |******************|  7872k  0:00:00 ETA

Suma kontrolna obrazu jest zawarta już w obrazie i weryfikowana podczas procesu flash'owania
routera. [Nie musimy zatem sami sprawdzać sumy
kontrolnej](https://www.dd-wrt.com/phpBB2/viewtopic.php?t=287516&sid=6b1350cc51ec6c053de95243fa2bd95d),
by upewnić się co do integralności danych w obrazie. Wystarczy pobrać i puścić proces aktualizacji.

Poniżej jest układ pamięci flash routera:

    # cat /proc/mtd
    dev:    size   erasesize  name
    mtd0: 00020000 00010000 "RedBoot"
    mtd1: 007c0000 00010000 "linux"
    mtd2: 006a0000 00010000 "rootfs"
    mtd3: 00020000 00010000 "ddwrt"
    mtd4: 00010000 00010000 "nvram"
    mtd5: 00010000 00010000 "board_config"
    mtd6: 00800000 00010000 "fullflash"
    mtd7: 00020000 00010000 "fullboot"
    mtd8: 00010000 00010000 "uboot-env"

Musimy się wgrać na `linux` przy pomocy poniższego polecenia:

    # write /tmp/obraz.bin linux
    function stop_freeradius not found
    freeram=[97955840] bufferram=[2985984]
    The free memory is enough, writing image once.
    write=[8060928]
    linux: CRC OK
    Writing image to flash, waiting a moment...
    write block [7995392] at [0x007A0000]
    done [8060928]

Teraz można zrestartować router poleceniem `reboot` . Urządzenie powinno się po krótkiej chwili
uruchomić ponownie. Pamiętajmy, że przy takim flash'owaniu, wszystkie ustawienia obecne w NVRAM są
zachowane. Jeśli nie jest to pożądana sytuacja, to po aktualizacji oprogramowania logujemy się
ponownie na terminal używając protokołu telnet i podając stary adres IP, login i hasło, po czym
wpisujemy to poniższe polecenie:

    # erase nvram

Możemy także zrobić użytek z przycisku reset na obudowie routera i go przycisnąć przez okres czasu
10 sekund.
