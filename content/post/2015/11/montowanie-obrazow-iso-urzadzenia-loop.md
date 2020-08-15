---
author: Morfik
categories:
- Linux
date: "2015-11-19T14:17:20Z"
date_gmt: 2015-11-19 13:17:20 +0100
published: true
status: publish
tags:
- loop
- cd
- dvd
- iso
title: Montowanie obrazów ISO (urządzenia loop)
---

Użytkownicy Debiana (i innych dystrybucji linux'a) mają czasem poważne problemy z zamontowaniem
[obrazu ISO](https://pl.wikipedia.org/wiki/ISO_%28obraz%29) pozyskanego czy to z internetu, czy też
od swoich znajomych. Na windowsach zwykliśmy korzystać z takich rozwiązań jak Daemon Tools, Alcohol
120%, czy też [WinCDEmu](http://wincdemu.sysprogs.org/). Na linux'ach z narzędzi, które mają GUI,
można chyba wyróżnić [furiusisomount](https://launchpad.net/furiusisomount) oraz
[acetoneiso](http://sourceforge.net/projects/acetoneiso/) ale nie będziemy się nimi zajmować w tym
wpisie. Na dobrą sprawę, to nie potrzebujemy żadnego zewnętrznego oprogramowania, by sprawnie i
szybko zamontować dowolny obraz ISO w swoim systemie. W tym wpisie zostanie przedstawiony sposób
montownia tychże obrazów, który zakłada wykorzystanie urządzeń `loop` .

<!--more-->
## Konfiguracja urządzeń loop

Wszystkie potrzebne narzędzia są dostarczane z pakietem `mount` . Jedyne o czym musimy pamiętać, to
by załadować odpowiedni moduł. Edytujmy zatem plik `/etc/modules-load.d/modules.conf` i upewnijmy
się, że widnieje w nim poniższa pozycja:

    loop

W ten sposób moduł `loop` będzie ładowany na starcie systemu i nie będziemy musieli nim sobie głowy
zawracać. Możemy go sobie także załadować ręcznie via `modprobe` w dowolnym momencie pracy systemu.
W każdym razie, w katalogu `/dev/` powinniśmy mieć szereg urządzeń `loop*` . Domyślnie jest ich 7,
co umożliwia zamontowanie do 7 różnych obrazów i niekoniecznie muszą to być obrazy ISO.

## Montowanie obrazu ISO

Weźmy sobie przykładowy obraz ISO i spróbujmy go zamontować. Do tego celu potrzebujemy pustego
katalogu oraz odpowiednich uprawnień. Mając oba te powyższe, obraz ISO montujemy w poniższy sposób:

    # mount -o loop debian.iso /mnt
    mount: /dev/loop0 is write-protected, mounting read-only

Wykorzystanie urządzeń `loop` można zawsze podejrzeć korzystając z `losetup` . Natomiast to gdzie
dany zasób został zamontowany, można odczytać z `mount`, przykładowo:

    # losetup
    NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
    /dev/loop0         0      0         1  0 /debian.iso

    # mount | grep debian
    /debian.iso on /mnt type iso9660 (ro,relatime)

## Odmontowanie obrazu ISO

Naturalnie rzecz biorąc, by odmontować obraz ISO, trzeba posłużyć się narzędziem `umount` . Czasem
jednak może się tak zdarzyć, że po wydaniu polecenia `umount` , plik z obrazem w dalszym ciągu
będzie siedział w urządzeniu `loop` . W takim przypadku musimy zwolnić je ręcznie. Cała procedura
odmontowania obrazu wygląda zatem mniej więcej tak:

    # umount /mnt
    # losetup -d /dev/loop0

## Montowanie obrazów ISO jako zwykły użytkownik

Wykorzystywanie konta administratora do tak prostej czynności jak montowanie obrazów ISO mija się z
celem. Użytkownik powinien być w stanie sam [zamontować dowolny obraz ISO bez uprawnień
administratora systemu]({{< baseurl >}}/post/udevil-i-montowanie-zasobow-bez-uprawnien-root/).
[Udevil](https://ignorantguru.github.io/udevil/) to proste konsolowe narzędzie, które odpowiednio
skonfigurowane potrafi umożliwić poszczególnym użytkownikom montowanie określonych zasobów. Znajduje
się ono w pakiecie `udevil` i nie powinno być problemów z jego instalacją w dystrybucji Debian.
Domyślna konfiguracja tego narzędzia jest tak dostosowana, by użytkownik był w stanie montować
obrazy ISO, dlatego też nie musimy przeprowadzać żadnych dodatkowych czynności pod kątem edycji
pliku `/etc/udevil/udevil.conf` . Natomiast montowanie obrazu ISO odbywa się w poniższy sposób:

    $ udevil mount /debian.iso /media/morfik/iso/

Nie musimy się także troszczyć o to, by docelowa lokalizacja istniała. W przypadku, gdy ten katalog
nie istnieje, to zostanie automatycznie stworzony.

By odmontować obraz ISO, w terminalu wpisujemy to poniższe polecenie:

    $ udevil umount /media/morfik/iso
