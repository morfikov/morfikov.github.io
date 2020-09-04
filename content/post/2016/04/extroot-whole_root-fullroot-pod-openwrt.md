---
author: Morfik
categories:
- OpenWRT
date: "2016-04-26T16:08:13Z"
date_gmt: 2016-04-26 14:08:13 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
title: Extroot i whole_root (fullroot) pod OpenWRT
---

Domowe routery WiFi zwykle nie dysponują flash'em o dużej pojemności. W ogromnej części przypadków
pamięć flash w takich urządzeniach nie przekracza 16 MiB. W zasadzie jest to wystarczająca ilość
miejsca ale tylko w przypadku korzystania z oryginalnego firmware producenta routera. Gdy w grę
wchodzi OpenWRT, to przy tak niewielkiej przestrzeni jest duże prawdopodobieństwo, że przy
instalowaniu dodatkowych pakietów zwyczajnie zabraknie nam miejsca. Jeśli nasz router dysponuje
chociaż jednym portem USB, to możemy rozszerzyć system plików routera do rozmiarów partycji
pendrive, który zostanie podłączony. W ten sposób z tych 16 MiB może nam się zrobić, np. 1-2 GiB, a
to już w zupełności wystarczy na instalację dowolnych pakietów z repozytorium OpenWRT. Cały ten
zabieg nosi nazwę `extroot` ([external root](https://wiki.openwrt.org/doc/howto/extroot)) lub
`whole_root` (fullroot) i w tym wpisie prześledzimy procedurę tworzenia tego mechanizmu.

<!--more-->
## Różnice między extroot i whole_root

Gdy chodzi o rozbudowanie systemu plików routera, to mamy do wyboru dwie opcje. Możemy to zrobić w
oparciu o `extroot` lub `whole_root` . Różnica między tymi dwoma mechanizmami polega na tym, że
`extroot` pozwala na przeniesienie części systemu routera (tj. zmian) na zewnętrzny nośnik. W
przypadku zaś `whole_root` jest przenoszony cały system routera. Dokładny [opis działania tych
mechanizmów można znaleźć tutaj](https://wiki.openwrt.org/doc/howto/extroot/extroot.theory). W obu
tych powyższych przypadkach partycja na pendrive musi być sformatowana w systemie plików z rodziny
EXT, zwykle EXT4. Jest to domyślny system plików stosowany na linux'ach.

## Wady i zalety extroot i whole_root

Wydawać by się mogło, że zarówno `extroot` jak i `whole_root` to bardzo przyzwoite mechanizmy, który
w znacznym stopniu przyczyną się do rozbudowy naszego routera, praktycznie za free. Trzeba mieć
jednak na uwadze jedną rzecz. Jeśli zamontujemy na routerze taką partycję, to nie możemy wyłączać
routera przez przycisk na jego obudowie. A już na pewno odpada nam możliwość wyciągnięcia wtyczki z
kontaktu. Tego typu akcje mogą uszkodzić system plików na pendrive, co może wiązać się z utratą
części lub wszystkich plików zgromadzonych na partycji. Router niczym się praktycznie nie różni w
działaniu od zwykłego PC. Jest tylko nieco mniejszy i nie ma tych wszystkich podzespołów, które mają
standardowe komputery. Dlatego też trzeba postępować z nim podobnie jak z ze zwykłym komputerem, a
tych przecie nie wyłączamy via przycisk na zasilaczu, prawda? By wyłączyć router, który korzysta z
`extroot` lub `whole_root` , musimy się zalogować się na niego przez SSH i położyć system przy
pomocy polecenia:

    # sync && poweroff

Po wydaniu tego powyższego polecenia, router pozornie będzie wyglądał na powieszony. Niby stracimy z
nim łączność ale lampki będą dalej się świecić czy migać. W tym stanie, router może zostać
pozbawiony zasiania bez stwarzania zagrożenia dla systemu plików na pendrive.

`extroot` oraz `whole_root` mają tę zaletę, że jeśli coś pochrzanimy w konfiguracji, to możemy
wyciągnąć pendrive z portu USB i załadować funkcjonalny system z flash'a routera. Wszystkie zmiany,
które przeprowadziliśmy mając podpięty pendrive zostaną cofnięte. Dodatkowo, jeśli dysponujemy jakąś
dystrybucją linux'a, lub też posiadamy system live, to partycję pendrive można zamontować w takim
systemie w celu przebadania zaistniałych problemów. W ten sposób będziemy w stanie edytować
poszczególne pliki. Coś jak [tryb
failsafe]({{< baseurl >}}/post/tryb-ratunkowy-failsafe-w-openwrt/), z tym, że podczas tej
operacji, nasz router działa i jest w pełni funkcjonalny. Zatem, jeśli przeprowadzamy jakieś
eksperymenty z nowym oprogramowaniem, dobrze jest pierw to robić na `extroot`/`whole_root` . Jeśli
wszystko będzie w porządku, możemy tę funkcjonalność zaimplementować na flash'u routera.

Pamiętajmy też, że po aktywowaniu mechanizmu `extroot` będziemy mieć kompletnie świeży system po
zresetowaniu routera. Wygląda to tak jakbyśmy go dopiero co flash'owali nową wersją firmware
OpenWRT. Całą konfigurację trzeba będzie ustawiać od początku.

Kolejna ważna sprawa, to upgrade firmware. W przypadku, gdy mamy zamiar dokonać aktualizacji
oprogramowania routera, musimy odłączyć pendrive i uruchomić system z flash'a routera. Dopiero wtedy
możemy wgrać obraz z nowym firmware.

Ostatnia rzecz, to UUID partycji z `extroot` lub `whole_root` . Narzędzie `block-mount` tworzy plik
`/etc/.extroot-uuid` na partycji pendrive. W nim umieszcza UUID partycji `mtd` zawierającej rootfs.
Jest to jedna z partycji routera. W fazie boot, podczas aktywacji `extroot`/`whole_root` ,
`block-mount` próbuje sprawdzić aktualną wartość UUID partycji `mtd` i porównuje ją z tą zapisaną w
pliku `/etc/.extroot-uuid` . Jeśli te dwie wartośći się różnią, partycja pendrive nie zostanie
zamontowana. Jeśli natomiast chcielibyśmy korzystać tej partycji po flash'owaniu routera, to musimy
ten plik pierw skasować.

## Instalacja potrzebnych pakietów

By zaimplementować `extroot`/`whole_root` , musimy doinstalować kilka pakietów na routerze. Poniżej
znajduje się pełna lista potrzebnych nam rzeczy:

    # opkg update
    # opkg install block-mount \
    kmod-fs-ext4 \
    kmod-crypto-crc32c \
    kmod-usb-storage \
    kmod-usb-core

Pakiet `kmod-crypto-crc32c` jest wymagany w przypadku wystąpienia błędu `EXT4-fs (sda4): Cannot load
crc32c driver.` podczas montowania zasobu (w `logread`) .

## Przygotowanie i konfiguracja partycji pod extroot i whole_root

Kluczem w tej całej zabawie z rozszerzaniem systemu plików routera jest utworzenie odpowiedniej
partycji. Tę zaś możemy przygotować bez większego problemu z płytki live mającą na pokładzie jakąś
dystrybucję linux'a. Możemy także doinstalować `fdisk` w OpenWRT i przeprowadzić proces tworzenia
partycji na routerze. W tym wpisie ograniczymy się jedynie do utworzenia partycji via `gparted` z
systemu live. Pendrive może zostać podzielony w dowolny sposób, np. tak jak to widać na poniższej
fotce:

![]({{< baseurl >}}/img/2016/04/1.gparted-linux-live-partycja-extroot-whole_root.png#huge)

Mając przygotowaną partycję, wsadzamy pendrive do jednego z portów USB w routerze. Urządzenie
powinno zostać wykryte przez system i stosownie oznaczone:

    kernel: [ 3020.600000] usb 1-1.2: new high-speed USB device number 3 using ehci-platform
    kernel: [ 3020.750000] usb-storage 1-1.2:1.0: USB Mass Storage device detected
    kernel: [ 3020.770000] scsi host0: usb-storage 1-1.2:1.0
    kernel: [ 3021.830000] scsi 0:0:0:0: Direct-Access     Kingston DataTraveler 3.0 PMAP PQ: 0 ANSI: 6
    kernel: [ 3022.400000] sd 0:0:0:0: [sda] 15360000 512-byte logical blocks: (7.86 GB/7.32 GiB)
    kernel: [ 3022.410000] sd 0:0:0:0: [sda] Write Protect is off
    kernel: [ 3022.410000] sd 0:0:0:0: [sda] Mode Sense: 23 00 00 00
    kernel: [ 3022.410000] sd 0:0:0:0: [sda] No Caching mode page found
    kernel: [ 3022.420000] sd 0:0:0:0: [sda] Assuming drive cache: write through
    kernel: [ 3022.470000]  sda: sda1 sda2 sda3 sda4
    kernel: [ 3022.490000] sd 0:0:0:0: [sda] Attached SCSI removable disk

Partycja, którą będziemy w tym przypadku wykorzystywać, to `sda4` . Przy pomocy `block detect`
generujemy odpowiednią konfigurację, którą trzeba umieścić w pliku `/etc/config/fstab` :

    # block detect > /etc/config/fstab

Następnie edytujemy plik `/etc/config/fstab` . Wszystkie partycje, które zostały rozpoznane przez
OpenWRT, zostaną automatycznie uwzględnione w tym pliku i umieszczone w sekcjach `config 'mount'` .
Poza tymi sekcjami, mamy także ustawienia globalne, które są w sekcji `config 'global'` . Na
początek omówmy sobie sekcję z ustawieniami globalnymi. Standardowo ta sekcja wygląda mniej więcej
tak:

    config 'global'
          option  anon_swap       '0'
          option  anon_mount      '0'
          option  auto_swap       '1'
          option  auto_mount      '1'
          option  delay_root      '5'
          option  check_fs        '0'

Opcje `anon_swap` oraz `anon_mount` odpowiadają za montowanie zasobów, które nie posiadają
konfiguracji w pliku `/etc/config/fstab` . Nas one nie dotyczą, a poza tym, lepiej nie ruszać ich
wartości domyślnych. Następnie mamy opcje `auto_swap` oraz `auto_mount` , które montują zasoby
określone w pliku `/etc/config/fstab` i mające status `enabled` (o tym za moment). Jeśli chodzi zaś
o opcję `delay_root` , to ma ona kluczowe znaczenie w przypadku `extroot`/`whole_root` . Może się
tak zdarzyć, że nasz pendrive zostanie wykryty z pewnym opóźnieniem podczas startu systemu routera.
W takim przypadku partycja nie zostanie zamontowana. Wartość tego parametru jest w sekundach i
oznacza ile czasu system ma czekać zanim zamontuje partycję z `extroot`/`fullroot`. Ostatnia opcja,
tj. `check_fs` , odpowiada za sprawdzanie systemu plików aktywnych zasobów.

Dalej w pliku mamy sekcje `config 'mount'` . Ta odpowiedzialna za `extroot`/`fullroot` jest
przedstawiona poniżej:

    config 'mount'
          option  target  '/overlay'
    #     option  target  '/'
    #     option  device  '/dev/sda4'
          option  uuid    'a2fc6334-0021-4468-8c21-48b2d2728ca2'
          option  fstype  'ext4'
          option  enabled '1'

Opcja `target` określa gdzie zamontować daną partycję. W przypadku `extroot` korzystamy z
`/overlay` . Natomiast jeśli chodzi o `whole_root` , to dajemy tutaj `/` . W obu przypadkach
wszelkie zmiany jakich dokonujemy w systemie, będą zapisywane partycji określonej w `device`.
Niemniej jednak, w przypadku podpięcia innego pendrive, OpenWRT będzie próbował zamontować jedną z
jego partycji. Dlatego też, lepiej unikać definiowania zasobów w taki sposób. Zamiast tego, lepiej
jest wykorzystać opcję `uuid` , która identyfikuje konkretne urządzenie (partycję) podpięte do
portu USB routera. Następnie mamy opcję `fstype` , która określa system plików partycji. Opcja
`enabled` ustawiona na `1` sprawi, że zasób zostanie zamontowany w fazie boot routera.

Jeśli nie chce nam się od początku konfigurować routera, możemy zgrać całą konfigurację. W przypadku
`extroot` ten krok jest opcjonalny. Natomiast jeśli chodzi o `whole_root` , to jest to wymagana
procedura.

Przy przenoszeniu konfiguracji należy pamiętać, aby w pliku `/etc/config/fstab` przy sekcjach
`config 'mount'` z `extroot`/`whole_root` przestawić poniższy parametr:

    option  enabled '0'

Jeśli zapomnimy to zrobić, wtedy router się zapętli. Tę opcję zawsze można przestawić po skopiowaniu
plików na partycję pendrive. Wystarczy skorzystać z polecenia `mount` .

Przenoszenie konfiguracji w przypadku `extroot` :

    # mount /dev/sda4 /mnt/
    # tar -C /overlay -cvf - . | tar -C /mnt/ -xf -
    # umount /mnt/

Przenoszenie konfiguracji w przypadku `whole_root` :

    # mount /dev/sda4 /mnt/
    # mkdir -p /tmp/cproot/
    # mount --bind / /tmp/cproot/
    # tar -C /tmp/cproot -cvf - . | tar -C /mnt/ -xf -
    # umount /tmp/cproot/
    # umount /mnt/

## Konfiguracja systemu na extroot

Po uruchomieniu routera, logujemy się na urządzenie standardowo za pomocą protokołu telnet. No
chyba, że skopiowaliśmy sobie konfigurację. Tak czy inaczej, naszym oczom powinien się pokazać taki
oto obrazek:

![]({{< baseurl >}}/img/2016/04/2.openwrt-extroot-whole_root.png#big)

Jak widzimy, ilość wolnego miejsca na flash'u to 1,7 GiB.

Niżej zaś jest trochę informacji na temat systemu plików po aktywacji `extroot` , zwrócone przez
`df` oraz `mount` :

    # df -h
    Filesystem                Size      Used Available Use% Mounted on
    rootfs                    1.8G      5.6M      1.7G   0% /
    /dev/root                 2.5M      2.5M         0 100% /rom
    tmpfs                    61.5M     72.0K     61.5M   0% /tmp
    /dev/sda4                 1.8G      5.6M      1.7G   0% /overlay
    overlayfs:/overlay        1.8G      5.6M      1.7G   0% /
    tmpfs                   512.0K         0    512.0K   0% /dev

    /# mount
    rootfs on / type rootfs (rw)
    /dev/root on /rom type squashfs (ro,relatime)
    proc on /proc type proc (rw,nosuid,nodev,noexec,noatime)
    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,noatime)
    tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noatime)
    /dev/sda4 on /overlay type ext4 (rw,relatime,data=ordered)
    overlayfs:/overlay on / type overlay (rw,noatime,lowerdir=/,upperdir=/overlay/upper,workdir=/overlay/work)
    tmpfs on /dev type tmpfs (rw,nosuid,relatime,size=512k,mode=755)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,mode=600)
    debugfs on /sys/kernel/debug type debugfs (rw,noatime

## Konfiguracja systemu na whole_root

W przypadku `whole_root` , informacje zwracane przez `df` i `mount` prezentują się następująco:

    # df -h
    Filesystem                Size      Used Available Use% Mounted on
    rootfs                    1.8G     14.0M      1.6G   1% /
    /dev/root                 2.5M      2.5M         0 100% /rom
    tmpfs                    61.5M     68.0K     61.5M   0% /tmp
    /dev/sda4                 1.8G     14.0M      1.6G   1% /
    tmpfs                   512.0K         0    512.0K   0% /dev

    # mount
    rootfs on / type rootfs (rw)
    /dev/root on /rom type squashfs (ro,noatime)
    proc on /proc type proc (rw,nosuid,nodev,noexec,noatime)
    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,noatime)
    tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noatime)
    /dev/sda4 on / type ext4 (rw,relatime,data=ordered)
    tmpfs on /dev type tmpfs (rw,nosuid,relatime,size=512k,mode=755)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,mode=600)
    debugfs on /sys/kernel/debug type debugfs (rw,noatime)

To co odróżnia `whole_root` od `extroot` , to fakt, że w tym pierwszym nie mamy wpisu
`overlayfs:/overlay` .

## Partycja rootfs_data (mtdblock3, mtd3)

Czasem przy korzystaniu z `extroot`/`whole_root` może zajść potrzeba, by podejrzeć pewne pliki
systemowe, które utworzyliśmy czy zmieniliśmy zanim przeszliśmy na któryś z tych powyższych
mechanizmów. Część użytkowników w takiej sytuacji resetuje router i odpala go bez podpiętego
pendrive. Jednak nie ma takiej potrzeby, bo partycję `rootfs_data` (tę zawierającą zmiany w obrazie
firmware) można bez problemu zamontować sobie z poziomu `extroot`/`whole_root` . Musimy tylko
sprawdzić, która to jest partycja. Zwykle jest to `mtd3` ale lepiej jest się upewnić zaglądając do
pliku `/proc/mtd` . W przypadku routera TP-LINK TL-WDR3600 v1 ten plik prezentuje się następująco:

    # cat /proc/mtd
    dev:    size   erasesize  name
    mtd0: 00020000 00010000 "u-boot"
    mtd1: 00121a00 00010000 "kernel"
    mtd2: 006ae600 00010000 "rootfs"
    mtd3: 00440000 00010000 "rootfs_data"
    mtd4: 00010000 00010000 "art"
    mtd5: 007d0000 00010000 "firmware"

Widzimy tutaj wyraźnie, że `rootfs_data` ma przypisane urządzenie `mtd3` i w katalogu `/dev/`
również mamy taki plik. Niemniej jednak, jest to urządzenie znakowe i nie możemy z niego
skorzystać. Zamiast tego musimy posłużyć się emulacją urządzenia blokowego `/dev/mtdblock3` .
Dokładna [różnica między urządzeniami mtd i mtdblock jest wyjaśniona
tutaj](http://www.linux-mtd.infradead.org/faq/general.html). Najważniejsza rzecz, to unikać zapisu
tego urządzenia. Wszelkie próby kontaktu z `/dev/mtdblock3` powinny być tylko w trybie do odczytu.
Niemniej jednak, jest możliwe operowanie na tym urządzenie jak na zwykłej partycji, a przez to
możemy ją zamontować w poniższy sposób:

    # mount -t jffs2 -o ro /dev/mtdblock3 /overlay-boot/

Możemy także wykorzystać do tego celu `block-mount` . Choć trzeba pamiętać, że partycje montowane za
jego pomocą będą w trybie do zapisu. Tak czy inaczej jeśli chcemy by partycja `/dev/mtdblock3` była
montowana na starcie routera, to do pliku `/etc/config/fstab` dodajemy tę poniższą treść:

    config 'mount'
          option target '/overlay-boot'
          option device '/dev/mtdblock3'
          option fstype 'jffs2'
          option enabled '1'
