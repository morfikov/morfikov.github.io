---
author: Morfik
categories:
- RaspberryPi
date:    2020-08-24 21:00:00 +0200
lastmod: 2020-08-24 21:00:00 +0200
published: true
status: publish
tags:
- raspberry-pi-4b
- kodi
- xbmc
- libreelec
- ntfs
- hdd
- ssd
GHissueID: 35
title: Montowanie dysków USB na Raspberry Pi z LibreELEC w trybie tylko do odczytu
---

Po paru dniach zabawy z [LibreELEC][1] na Raspberry Pi 4B mogę powiedzieć, że ten system ma w
zasadzie wszystko to, co jest potrzebne przy zabawie z Kodi/XBMC i przerabianiu maliny na domowe
centrum multimediów. Niestety jednak, w moim przypadku pojawił się jeden problem, który dotyczył
montowania zasobów (konkretnie dysku HDD). Chodzi generalnie o to, że LibreELEC nie umożliwia
zdefiniowania własnych punktów montowania i opcji dla tych punktów. W ten sposób wszystkie dyski USB
podłączone do Raspberry Pi 4B będą działać w trybie do zapisu. Teoretycznie LibreELEC (czy Kodi)
nie powinien nic na tych dyskach zapisywać sam z siebie. Niemniej jednak, system plików NTFS
działający w trybie do zapisu na maszynie z linux, która nie ma podłączonego żadnego UPS'a, budzi u
mnie lekki dyskomfort psychiczny. Chciałem zatem swój dysk zamontować w trybie tylko do odczytu ale
polityka LibreELEC na to domyślnie nie pozwala, co nie znaczy oczywiście, że tego zadania się nie
da zrealizować.

<!--more-->
## Dlaczego LibreELEC nie zezwala na zmiany w punktach montowania

W klasycznych linux'ach jest cała masa sposobów na ogarnięcie dysków podłączanych do komputera.
Może i LibreELEC jest na bazie linux'a ale jego główny system plików jest zamontowany w trybie
tylko do odczytu za sprawą zastosowania [SquashFS][2]. W ten sposób nie mamy możliwości edycji
pliku `/etc/fstab` , w którym to moglibyśmy bez większego problemu określić co, gdzie i jak
montować. Można by w ten dość prosty sposób określić flagę `ro` i problem z głowy.

Skorzystanie z pliku `/etc/fstab` nie jest oczywiście jedynym wyjściem w sytuacjach, gdy trzeba w
systemie zamontować systemy plików partycji czy dysków. W przypadku LibreELEC mamy do dyspozycji
jeszcze dwie opcje, tj. usługi systemd oraz `udevil` .

### Brak możliwości zmiany konfiguracji udevil

Standardowo LibreELEC wykorzystuje usługi systemd jedynie do [montownia zasobów zdalnych][7], np.
NFS czy Samba. W przypadku tych lokalnie podłączanych dysków do Raspberry Pi 4B przy pomocy portów
USB wykorzystywany jest [udevil][4]. Kiedyś korzystałem z tego oprogramowania do ogarniania
[montowania zasobów jako zwykły użytkownik na swoim Debianie][3]. Niemniej jednak, [data ostatniego
commit'a w repo git udevil'a][5] sugeruje, że ten projekt jest martwy od ponad 5 lat już. Jak widać,
LibreELEC ten fakt nie przeszkadza ale "skoro działa, to nie naprawiać".

Problem w tym, że właśnie nie działa. Bo przy pomocy `udevil` te flagi montowania można by łatwo
określić, gdyby nie fakt, że plik `/etc/udevil/udevil.conf` , podobnie jak i `/etc/fstab` , jest
tylko do odczytu i nie można go w żaden sposób zmienić. Nie da rady też go nadpisać lokalnie (przy
pomocy plików w katalogu `/storage/` ), zatem jakiekolwiek próby dostosowania konfiguracji `udevil`
zakończą się niepowodzeniem.

### Nie do końca działające usługi systemd

LibreELEC jako system inicjacji procesów wykorzystuje systemd (aktualnie w wersji 242). Zatem
systemd jest dość świeży, przez co powinien posiadać wsparcie dla usług montowania zasobów (pliki
`.mount` ). Po krótkim rozeznaniu stworzyłem plik
`/storage/.config/system.d/var-media-ntfs_data.mount` o poniższej treści:

    [Unit]
    Description=NTFS media
    Before=kodi.service

    [Mount]
    What=/dev/sda1
    Where=/var/media/ntfs_data
    Options=ro,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other
    Type=ntfs
    TimeoutSec=10

    [Install]
    WantedBy=multi-user.target

W LibreELEC nie ma katalogu `/media/` i zamiast niego jest `/var/media/` i to tutaj domyślnie są
montowane wszystkie dyski podłączane przez porty USB. Urządzenie można określić po UUID, zamiast
`/dev/sda1` , tak by uniknąć problemów przy ewentualnym podłączeniu kilku dysków twardych.

Z jakiegoś jednak powodu systemd w LibreELEC nie potrafi wykryć, że dysk się pojawił podczas startu
systemu:

	18:28:36 LibreELEC systemd[1]: Started udev Coldplug all Devices.
	18:28:46 LibreELEC kernel: usb 1-1.4: new high-speed USB device number 3 using xhci_hcd
	18:28:46 LibreELEC kernel: usb 1-1.4: New USB device found, idVendor=152d, idProduct=2338, bcdDevice= 1.00
	18:28:46 LibreELEC kernel: usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=5
	18:28:46 LibreELEC kernel: usb 1-1.4: Product: USB to ATA/ATAPI Bridge
	18:28:46 LibreELEC kernel: usb 1-1.4: Manufacturer: JMicron
	18:28:46 LibreELEC kernel: usb 1-1.4: SerialNumber: D57CA0A36079
	18:28:46 LibreELEC kernel: usb-storage 1-1.4:1.0: USB Mass Storage device detected
	18:28:46 LibreELEC kernel: scsi host0: usb-storage 1-1.4:1.0
	18:28:47 LibreELEC kernel: scsi 0:0:0:0: Direct-Access     WDC WD15 EARS-00MVWB0     AB51 PQ: 0 ANSI: 2 CCS
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] 2930275055 512-byte logical blocks: (1.50 TB/1.36 TiB)
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] Write Protect is off
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] Mode Sense: 00 38 00 00
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] Asking for cache data failed
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] Assuming drive cache: write through
	18:28:47 LibreELEC kernel:  sda: sda1
	18:28:47 LibreELEC kernel: sd 0:0:0:0: [sda] Attached SCSI disk
	18:30:06 LibreELEC systemd[1]: dev-sda1.device: Job dev-sda1.device/start timed out.
	18:30:06 LibreELEC systemd[1]: Timed out waiting for device /dev/sda1.
	18:30:06 LibreELEC systemd[1]: Dependency failed for NTFS media.
	18:30:06 LibreELEC systemd[1]: var-media-ntfs_data.mount: Job var-media-ntfs_data.mount/start failed with result 'dependency'.
	18:30:06 LibreELEC systemd[1]: dev-sda1.device: Job dev-sda1.device/start failed with result 'timeout'.
	18:30:06 LibreELEC systemd[1]: Reached target Local File Systems.

Jak widać z powyższego logu, dysk został wykryty w zasadzie o `18:28:47` (po około 10 sekundach od
włączenia urządzenia) . Po około jednej minucie i 19 sekundach wybija z kolei zegar oczekiwania na
urządzenie `/dev/sda1` , w efekcie czego nasza usługa zwraca problem z zależnościami, przez co dysk
nie jest montowany. Pytanie dlaczego tak się dzieje? Przecież dysk został wykryty i usługa, którą
napisaliśmy wyżej, powinna bez większego problemu ten dysk zamontować.

Ze znacznika czasu wynika też, że do momentu osiągnięcia targetu `Local File Systems` , upłynęło
~1,5 minuty. Oczywiście to nie jest koniec procedury startu Raspberry Pi 4B ale przez cały ten czas
nie da rady się na malinę nawet zalogować po SSH, nie wspominając nawet o starcie Kodi.

Próbowałem także przepisać te domyślne zależności, tak by opóźnić nieco moment, w którym ten dysk
USB byłby montowany przez określenie tych poniższych regułek:

    [Unit]
    ...
    Before=kodi.service
    Before=shutdown.target
    After=basic.target
    DefaultDependencies=no
    Conflicts=shutdown.target
    ...

Te powyższe [zależności są tak dobrane][6], by ten dysk nie był montowany standardowo przed
`local-fs.target` , a dopiero po `basic.target` z tym, że przed usługą `kodi.service` . Niemniej
jednak, to rozwiązanie też nie zadziałało (o tym za moment).

#### Problem z flagami montowania

Gdy się podejrzy wyjście polecenia `mount` po uruchomieniu się systemu LibreELEC, to możemy
zobaczyć z jakimi flagami domyślnie jest montowana partycja naszego dysku twardego:

    LibreELEC:~ # mount | grep fuseblk
    /dev/sda1 on /var/media/ntfs_data type fuseblk (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096)

Wydawać by się mogło, że skoro znamy te domyślne flagi montowania, to wystarczy je określić w
linijce z `Options=`  w usłudze systemd. Gdy się tę usługę uruchomi ręcznie, to działa ona dobrze,
za wyjątkiem poprawnego ustawiania flag.

Gdy się nie określi żadnych flag w `Options=` , to wtedy w logu systemowym możemy zobaczyć poniższe
komunikaty:

    19:38:05 LibreELEC systemd[1]: Mounting NTFS media...
    19:38:07 LibreELEC ntfs-3g[809]: Version 2017.3.23 external FUSE 29
    19:38:07 LibreELEC ntfs-3g[809]: Requested device /dev/disk/by-uuid/4CF875EB65AC2B02 canonicalized as /dev/sda1
    19:38:07 LibreELEC ntfs-3g[809]: Mounted /dev/sda1 (Read-Write, label "ntfs_data", NTFS 3.1)
    19:38:07 LibreELEC ntfs-3g[809]: Cmdline options:
    19:38:07 LibreELEC ntfs-3g[809]: Mount options: allow_other,nonempty,relatime,fsname=/dev/sda1,blkdev,blksize=4096
    19:38:07 LibreELEC ntfs-3g[809]: Ownership and permissions disabled, configuration type 7
    19:38:07 LibreELEC systemd[1]: Mounted NTFS media.

W wyjściu `mount` , ten zasób jest widoczny mniej więcej tak:

    LibreELEC:~ # mount | grep fuseblk
    /dev/sda1 on /var/media/ntfs_data type fuseblk (rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other,blksize=4096)

Zatem mimo iż nie określiliśmy żadnych flag, system dobrał je sobie sam.

Określenie flagi `ro` w `Options=` na nic nie wpływa, tj. ten powyższy log będzie dokładnie taki
sam bez znaczenia czy tę flagę w opcjach określimy czy nie. Można by pomyśleć, że te flagi
montowania określone w usłudze systemd są ignorowane z jakiegoś powodu w LibreELEC. To założenie
jednak nie jest zgodne z prawdą, bo gdy się określi, np. flagę `default_permissions` , to jak
najbardziej ona zostanie uwzględniona:

    19:42:40 LibreELEC systemd[1]: Mounting NTFS media...
    19:42:41 LibreELEC ntfs-3g[869]: Version 2017.3.23 external FUSE 29
    19:42:41 LibreELEC ntfs-3g[869]: Requested device /dev/disk/by-uuid/4CF875EB65AC2B02 canonicalized as /dev/sda1
    19:42:41 LibreELEC ntfs-3g[869]: Mounted /dev/sda1 (Read-Write, label "ntfs_data", NTFS 3.1)
    19:42:41 LibreELEC ntfs-3g[869]: Cmdline options: default_permissions
    19:42:41 LibreELEC ntfs-3g[869]: Mount options: user_id=0,group_id=0,allow_other,nonempty,default_permissions,relatime,fsname=/dev/sda1,blkdev,blksize=4096
    19:42:41 LibreELEC ntfs-3g[869]: Ownership and permissions disabled, configuration type 7
    19:42:41 LibreELEC systemd[1]: Mounted NTFS media.

I w wyjściu polecenia `mount` również będzie ją widać:

    # mount  | grep fuseblk
    /dev/sda1 on /var/media/ntfs_data type fuseblk (rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096)

Dlaczego zatem flaga `ro` jest ignorowana? Nie mam pojęcia. Ta sama usługa działa grzecznie na
Debianie. Być może jakieś specyficzne ustawienia LibreELEC mają za zadanie dobrać określony zestaw
domyślnych flag i w ten sposób `ro` jest przepisywany na `rw` , przez co dysk jest montowany
`Read-Write` , czyli w trybie do zapisu.

### Konflikt systemd z udevil przy montowaniu dysków

Nawet gdyby te flagi udało się ustawić przy pomocy usługi systemd, to wygląda na to, że i tak byśmy
mieli problem z zamontowaniem dysku w trybie tylko do odczytu. Po podpięciu dysku do Raspberry Pi
4B, po jego wykryciu zostanie od razu aktywowana usługa `udevil-mount@.service` (po znaku `@`
generowana jest ścieżka do urządzenia, np. `-dev-sda1`), co można zauważyć w logu systemowym:

	19:20:11:33 LibreELEC kernel: usb 1-1.4: new high-speed USB device number 3 using xhci_hcd
	19:20:11:33 LibreELEC kernel: usb 1-1.4: New USB device found, idVendor=152d, idProduct=2338, bcdDevice= 1.00
	19:20:11:33 LibreELEC kernel: usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=5
	19:20:11:33 LibreELEC kernel: usb 1-1.4: Product: USB to ATA/ATAPI Bridge
	19:20:11:33 LibreELEC kernel: usb 1-1.4: Manufacturer: JMicron
	19:20:11:33 LibreELEC kernel: usb 1-1.4: SerialNumber: D57CA0A36079
	19:20:11:33 LibreELEC kernel: usb-storage 1-1.4:1.0: USB Mass Storage device detected
	19:20:11:33 LibreELEC kernel: scsi host0: usb-storage 1-1.4:1.0
	19:20:11:34 LibreELEC kernel: scsi 0:0:0:0: Direct-Access     WDC WD15 EARS-00MVWB0     AB51 PQ: 0 ANSI: 2 CCS
	19:20:11:34 LibreELEC kernel: sd 0:0:0:0: [sda] 2930275055 512-byte logical blocks: (1.50 TB/1.36 TiB)
	19:20:11:34 LibreELEC kernel: sd 0:0:0:0: [sda] Write Protect is off
	19:20:11:34 LibreELEC kernel: sd 0:0:0:0: [sda] Mode Sense: 00 38 00 00
	19:20:11:34 LibreELEC kernel: sd 0:0:0:0: [sda] Asking for cache data failed
	19:20:11:34 LibreELEC kernel: sd 0:0:0:0: [sda] Assuming drive cache: write through
	19:20:11:35 LibreELEC kernel:  sda: sda1
	19:20:11:35 LibreELEC kernel: sd 0:0:0:0: [sda] Attached SCSI disk
	19:20:11:35 LibreELEC systemd[1]: Created slice system-udevil\x2dmount.slice.
	19:20:11:35 LibreELEC systemd[1]: Starting Udevil mount service...
	19:20:11:35 LibreELEC kernel: fuse init (API version 7.27)
	19:20:11:35 LibreELEC systemd[1]: Condition check resulted in FUSE Control File System being skipped.
	19:20:11:37 LibreELEC ntfs-3g[507]: Version 2017.3.23 external FUSE 29
	19:20:11:37 LibreELEC ntfs-3g[507]: Mounted /dev/sda1 (Read-Write, label "ntfs_data", NTFS 3.1)
	19:20:11:37 LibreELEC ntfs-3g[507]: Cmdline options: big_writes,fmask=0133,uid=0,gid=0,utf8
	19:20:11:37 LibreELEC ntfs-3g[507]: Mount options: utf8,allow_other,nonempty,relatime,default_permissions,fsname=/dev/sda1,blkdev,blksize=4096
	19:20:11:37 LibreELEC ntfs-3g[507]: Global ownership and permissions enforced, configuration type 7
	19:20:11:37 LibreELEC udevil[454]: Mounted /dev/sda1 at /media/ntfs_data
	19:20:11:37 LibreELEC systemd[1]: Started Udevil mount service.

Mamy zatem problem, bo dwie usługi systemd będą próbować zamontować ten zasób. Tylko jednej z nich
się to uda i jak możemy przypuszczać, `udevil` będzie pierwszy. Zatem ta nasza usługa montowania
(ta ze zmienionymi zależnościami) zakończy się takim komunikatem przy starcie systemu:

	19:20:11:37 LibreELEC systemd[1]: Condition check resulted in NTFS media being skipped.
	19:20:11:37 LibreELEC systemd[1]: Unnecessary job for EARS-00MVWB0 ntfs_data was removed.

Wygląda zatem na to, że można sobie darować próby ogarniania montowania lokalnych dysków USB na
LibreELEC. Zatem co począć w takiej sytuacji?

## Autostart w LibreELEC

No trzeba przyznać, że chłopaki z LibreELEC się nieźle napracowali by uniemożliwić użytkownikowi
określenie co, gdzie i jak chciałby montować. Na szczęście linux jest linux i jeśli nie da rady
natywnie skonfigurować pewnych rzeczy w systemie, to trzeba posiłkować się skryptami i mechanizmem
autostartu.

[W LibreELEC plik autostart.sh][8] robi właśnie za listę poleceń, które mają zostać wykonane przed
uruchomieniem się Kodi. Ten plik autostartu przechowywany jest w `/storage/.config/autostart.sh` ,
choć domyślnie nie istnieje i trzeba go sobie samemu utworzyć:

    LibreELEC:~ # touch /storage/.config/autostart.sh
    LibreELEC:~ # chmod +x /storage/.config/autostart.sh
    LibreELEC:~ # echo "#!/bin/sh" > /storage/.config/autostart.sh

### Ustawienie flag montowania via plik autostart.sh

Mając plik autostartu, możemy teraz dodać do niego polecenie, które dostosuje flagi montowania.
Poniżej jest linijka, którą trzeba dodać do pliku `/storage/.config/autostart.sh` :

    (sleep 10 && mount -t ntfs-3g -o remount,ro /var/media/ntfs_data) &

Standardowo, wszystkie polecenia określone w pliku `/storage/.config/autostart.sh` będą wykonywane
na pierwszym planie przez co mogą opóźnić lub też nawet uniemożliwić start systemu. W tym przypadku
wykonywane jest polecenie `mount` , które ma za zadanie przemontować określony system plików w tryb
tylko do odczytu. To polecenie wykonałoby się praktycznie natychmiast ale z racji uruchamiania tego
skryptu autostartu przed wykryciem dysków podłączonych do Raspberry Pi 4B (opóźnienie ~10 sekund
związane z USB), to takie przemontowanie systemu plików by nam nic nie dało, jako że dysk by nawet
nie został przez LibreELEC wykryty. W efekcie w dalszym ciągu byśmy mieli system plików zamontowany
w trybie do zapisu. Dlatego też przed procesem przemontowania mamy `sleep 10 &&` , który ma na celu
opóźnienie wykonania polecenia `mount` o te 10 sekund od chwili wywołania skryptu autostartu.
Z kolei opóźnienie wykonania procesu przemontowania systemu plików powoduje, że start systemu
zostanie opóźniony o te 10 dodatkowych sekund. Dlatego całe polecenie zostało ujęte w `() &` , tak
by system wykonał je w tle. W ten sposób LibreELEC wystartuje jak gdyby nigdy nic, po paru sekundach
zostanie wykryty dysk, a po jeszcze paru sekundach zostanie wykonane to powyższe polecenie, czego
efektem będzie system plików dysku zamontowany w trybie tylko do odczytu, o czym możemy przekonać
się podglądając wyjście polecenia `mount` :

    LibreELEC:~ # mount | grep fuseblk
    /dev/sda1 on /var/media/ntfs_data type fuseblk (ro,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other,blksize=4096)


[1]: https://libreelec.tv/
[2]: https://en.wikipedia.org/wiki/SquashFS
[3]: /post/udevil-i-montowanie-zasobow-bez-uprawnien-root/
[4]: https://ignorantguru.github.io/udevil/
[5]: https://github.com/IgnorantGuru/udevil
[6]: https://www.freedesktop.org/software/systemd/man/bootup.html
[7]: https://libreelec.wiki/how_to/mount_network_share
[8]: https://wiki.libreelec.tv/autostart.sh
