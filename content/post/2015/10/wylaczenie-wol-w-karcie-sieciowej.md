---
author: Morfik
categories:
- Linux
date: "2015-10-04T16:36:26Z"
date_gmt: 2015-10-04 14:36:26 +0200
published: true
status: publish
tags:
- udev
- systemd
- powersave
title: Wyłączenie WoL w karcie sieciowej
---

[WoL][1] (Wake on LAN) to taki ficzer, który umożliwia włączenie komputera przez sieć. By tego typu
sytuacja miała miejsce, zarówno karta sieciowa oraz płyta główna komputera musi wspierać WoL.
Dodatkowo, BIOS maszyny musi mieć aktywowaną odpowiednią opcję. Obecnie WoL jest implementowany
praktycznie w każdej płycie głównej i karcie sieciowej, także raczej możemy przyjąć, że nasz
komputer może zostać wybudzony za pomocą kabla sieciowego. Stwarza to oczywiście zagrożenie
bezpieczeństwa ale poza tym, w grę wchodzi także większy pobór energii. Jeśli nie korzystamy z WoL,
to jest on dla nas zbędny i należałoby go wyłączyć.

<!--more-->
## Sprawdzenie stanu WoL

Jak wspomniałem na wstępie, wielu z nas nigdy nie korzystało z wybudzania maszyny za pomocą opcji
WoL. Domyślnie WoL jest aktywowany, zatem pobiera energię. Nie jest jej, co prawda, dużo ale zawsze.
Należy przy tym dodać, że karty Wi-Fi nie wspierają WoL. Dodatkowo, w większości laptopów, WoL nie
działa gdy te pracują na baterii i wymagane jest podpięcie ich do źródła zasilania.

Jeżeli nie wiemy do końca czy WoL jest włączony (czy w ogóle dostępny) w sterowniku karty sieciowej,
to możemy to sprawdzić wydając poniższe polecenie:

    # ethtool eth0 | grep Wake-on
            Supports Wake-on: pumbg
            Wake-on: g

W tym przypadku karta od interfejsu `eth0` obsługuje kilka rodzajów WoL, które są stosownie
oznaczane przy pomocy liter. Możemy się spotkać z poniższymi: `p` ([PHY][2]), `u` (ruch unicast),
`m` (ruch multicast), `b` (ruch broadcast), `a` (pakiety ARP), `g` ([magiczny pakiet][3]) oraz `d`
(wyłączony).

Zatem, wybudzenie tej maszyny może nastąpić po przesłaniu magicznego pakietu, co automatycznie
implikuje włączenie WoL dla tego interfejsu i wyższe zużycie energii.

## Wyłączenie WoL

WoL może zostać wyłączony przy pomocy narzędzia `ethtool` w poniższy sposób:

    # ethtool -s eth1 wol d

I sprawdzamy, czy ta operacja przyniosła jakieś zmiany:

    # ethtool eth1 | grep Wake-on
          Supports Wake-on: pumbg
          Wake-on: d

Litera `d` sugeruje, że WoL został z powodzeniem wyłączony. Jest tylko jeden problem, mianowicie, te
powyższe ustawienia nie przetrwają restartu maszyny. Trzeba jakoś temu zaradzić. [Jest wiele metod,
które mogą trwale wyłączyć WoL][4] i my skupimy się głównie na dwóch z nich, tj. przy pomocy reguły
dla udev'a oraz przy pomocy systemd.

### Reguła dla udev'a

Ten sposób zakłada iż orientujemy się nieco w zagadnieniach związanych z [pisałem reguł dla
udev'a][5]. Niemniej jednak, będąc obytym z udev'em, problematyczne może okazać się określenie
nazwy interfejsu sieciowego, dlatego też lepiej dopasowywać urządzenie po ścieżce PCI, lub też
odpowiednio dobrać numerek dla pliku reguły, tak by została ona zaaplikowana dopiero po zmianie
nazw interfejsów sieciowych, za które odpowiada plik `70-persistent-net.rules` .

W tym przypadku będziemy dopasowywać po ścieżce PCI i musimy najsampierw ją ustalić. W tym celu
odpalamy terminal i wpisujemy `lspci` :

    # lspci | grep -i ethernet
    02:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8101/2/6E PCI Express Fast/Gigabit Ethernet controller (rev 02)

Zatem ścieżka PCI, to `02:00.0` i to w oparciu o ten numerek zrobimy dopasowanie w regule udev'a.
Tworzymy zatem plik `50-wol.rules` w katalogu `/etc/udev/rules.d/` i dopisujemy tam poniższą
linijkę:

    ACTION=="add", SUBSYSTEM=="net", KERNELS=="0000:02:00.0", RUN+="/sbin/ethtool -s $name wol g"

### WoL i systemd

Alternatywną metodą na wyłączenie WoL jest zaprzęgnięcie do pracy systemd, i stworzenie
[pliku .link][6] , w którym to określane są parametry dla interfejsu sieciowego i niekoniecznie
muszą się one ograniczać tylko do opcji WoL.

By wyłączyć WoL przy pomocy systemd, tworzymy plik `/etc/systemd/network/05-eth0.link` o poniższej
treści:

    [Match]
    Path=pci-0000:02:00.0

    [Link]
    Description=Disable WoL on eth0 interface
    WakeOnLan=off

Podobnie jak w przypadku reguły dla udev'a, tutaj też dopasowanie dla urządzenia zrobiliśmy po
ścieżce PCI. Ścieżkę także możemy wyciągnąć przy pomocy `networkctl` :

    # networkctl status eth0
    ...
          Path: pci-0000:02:00.0
    ...

Po restarcie maszyny, WoL powinien być już wyłączony na dobre, o czym możemy się przekonać wydając
poniższe polecenie:

    # ethtool eth0 | grep Wake-on
            Supports Wake-on: pumbg
            Wake-on: d


[1]: https://pl.wikipedia.org/wiki/Wake_on_LAN
[2]: https://en.wikipedia.org/wiki/PHY_%28chip%29
[3]: https://pl.wikipedia.org/wiki/Magic_Packet
[4]: https://wiki.archlinux.org/index.php/Wake-on-LAN
[5]: /post/udev-czyli-jak-pisac-reguly-dla-urzadzen/
[6]: https://www.freedesktop.org/software/systemd/man/systemd.link.html
