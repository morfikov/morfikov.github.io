---
author: Morfik
categories:
- Linux
date: "2016-01-15T20:28:24Z"
date_gmt: 2016-01-15 19:28:24 +0100
published: true
status: publish
tags:
- pendrive
- bezpieczeństwo
- luks
- szyfrowanie
GHissueID: 540
title: Nagłówek kontenera LUKS trzymany na pendrive
---

Jeśli kiedyś rozważaliśmy [umieszczenie pliku klucza (keyfile) do zaszyfrowanego kontenera LUKS na
pendrive](/post/keyfile-trzymany-w-glebokim-ukryciu/), to ciekawszą alternatywą
może okazać się umieszczenie całego nagłówka takiego kontenera na zewnętrznym nośniku. Ma to tę
przewagę nad keyfile, że wszystkie informacje zapewniające dostęp do kontenera, wliczając w to klucz
główny, są oddzielone od zaszyfrowanych danych. W ten sposób nawet jeśli kontener wpadnie w
niepowołane ręce, to nie ma żadnego sposobu na to, by ten ktoś te dane odzyskał, no bo przecie nie
ma klucza szyfrującego. Przechwycenie hasła również nic to nie zmieni, no chyba, że ten ktoś
zdobędzie również pendrive z nagłówkiem kontenera. Z ludzkiego punktu widzenia, to na takim dysku
będą znajdować się jedynie losowymi dane i do tego w formie kompletnie nieczytelnej dla człowieka
(brak systemu plików). Niemniej jednak, jest kilka rzeczy, o których warto pamiętać, gdy w grę
wchodzi nagłówek LUKS i to o nich porozmawiamy sobie w tym wpisie.

<!--more-->
## Nagłówek LUKS

Kontenery LUKS mają nagłówki. Zwykle jest to pierwsze 2 MiB danych, zaraz na początku zaszyfrowanej
partycji. Sam kontener i jego nagłówek są niezależne od siebie. Mogą zatem istnieć na innych
fizycznych mediach, np. kontener na jednym dysku, a jego nagłówek na pendrive. Jeśli taki schemat
zastosujemy, to do otworzenia kontenera będzie nam potrzebny ten zewnętrzny nośnik, o czym trzeba
pamiętać podczas startu systemu, oczywiście jeśli będziemy chcieli go w tym czasie montować ale tym
zajmiemy się później. Natomiast teraz stwórzmy sobie kontener,
    przykładowo:

    # cryptsetup \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --iter-time 5000 \
      --use-random \
      --verify-passphrase \
      --verbose \
      luksFormat /dev/sda2

    # cryptsetup luksOpen /dev/sda2 sda2

    # mkfs.ext4 /dev/mapper/sda2

    # cryptsetup luksClose sda2

Na dobrą sprawę, to pierwsze polecenie, zapisało jedynie dwa pierwsze megabajty na partycji
`/dev/sda2` i to właśnie jest nasz nagłówek, który musimy wydobyć i zapisać w osobnym pliku. Kopię
nagłówka możemy zrobić przy pomocy `dd` albo też użyć do tego celu `luksHeaderBackup` . To drugie
rozwiązanie nas uchroni przed popełnieniem ewentualnych błędów i dlatego to z niego skorzystamy:

    # cryptsetup luksHeaderBackup --header-backup-file header.img /dev/sda2

## Usuwanie nagłówka LUKS

Mając nagłówek w pliku, możemy teraz nadpisać odpowiedni obszar partycji. Jest tylko jeden problem.
Jeśli byśmy nadpisali cały nagłówek przy pomocy `dd` , to system nie rozpozna tej partycji jako
kontener LUKS. Jeśli jej nie rozpozna, to nie wykryje jej UUID, a bez UUID będą problemy z
automatycznym montowaniem takiego zasobu.

Zgodnie z informacjami jakie znalazłem w [FAQ
cryptsetup](https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions), nagłówek LUKS
ma pewien określony format:

    Refers to LUKS On-Disk Format Specification Version 1.2.1

    LUKS header:

    offset  length  name             data type  description
    -----------------------------------------------------------------------
    0x0000   0x06   magic            byte[]     'L','U','K','S', 0xba, 0xbe
         0      6
    0x0006   0x02   version          uint16_t   LUKS version
         6      3
    0x0008   0x20   cipher-name      char[]     cipher name spec.
         8     32
    0x0028   0x20   cipher-mode      char[]     cipher mode spec.
        40     32
    0x0048   0x20   hash-spec        char[]     hash spec.
        72     32
    0x0068   0x04   payload-offset   uint32_t   bulk data offset in sectors
       104      4                               (512 bytes per sector)
    0x006c   0x04   key-bytes        uint32_t   number of bytes in key
       108      4
    0x0070   0x14   mk-digest        byte[]     master key checksum
       112     20                               calculated with PBKDF2
    0x0084   0x20   mk-digest-salt   byte[]     salt for PBKDF2 when
       132     32                               calculating mk-digest
    0x00a4   0x04   mk-digest-iter   uint32_t   iteration count for PBKDF2
       164      4                               when calculating mk-digest
    0x00a8   0x28   uuid             char[]     partition UUID
       168     40
    0x00d0   0x30   key-slot-0       key slot   key slot 0
       208     48
    0x0100   0x30   key-slot-1       key slot   key slot 1
       256     48
    0x0130   0x30   key-slot-2       key slot   key slot 2
       304     48
    0x0160   0x30   key-slot-3       key slot   key slot 3
       352     48
    0x0190   0x30   key-slot-4       key slot   key slot 4
       400     48
    0x01c0   0x30   key-slot-5       key slot   key slot 5
       448     48
    0x01f0   0x30   key-slot-6       key slot   key slot 6
       496     48
    0x0220   0x30   key-slot-7       key slot   key slot 7
       544     48

Będąc w posiadaniu tych informacji, musimy zamazać deskryptory haseł (widoczne powyżej jako keyslot)
oraz bajty od 6-168. Chodzi o to, że w pierwszych 6 bajtach nagłówka jest zawarta informacja, że
jest to kontener LUKS. Linux potrzebuje tej informacji, by skutecznie wykryć typ partycji. Dlatego
też nie możemy jej zamazać. Na pozycji 168 jest ulokowany UUID tego kontenera. Wszystkie narzędzia,
które mają operować na tym kontenerze, potrzebują tego numerka. Dlatego te dwa pola muszą zostać
nietknięte, a całą resztę możemy zamazać. Robimy to w poniższy sposób:

    # cryptsetup luksKillSlot /dev/sdb2 0

    # dd if=/dev/zero of=/dev/sdb2 bs=1 seek=6 count=162

Jeśli teraz spróbujemy odczytać informację z nagłówka takiego kontenera przy pomocy `luksDump` , to
powinniśmy otrzymać mniej więcej taki komunikat:

    # cryptsetup luksDump /dev/sdb2
    Unsupported LUKS version 0

Mimo, że system nie jest w stanie odczytać nagłówka tego kontenera, to wciąż jest w stanie ustalić
jego UUID:

    # partprobe

    # lsblk -f /dev/sdb
    NAME         FSTYPE      LABEL UUID                                   MOUNTPOINT
    sdb
    ├─sdb1       LVM2_member       VsO7QD-1uxK-q0zr-KDa4-DVSM-Levr-p93ewp
    │ ├─lvm-root ext4        root  ee759aa3-4977-4c08-b14f-a62d17a47478   /
    │ └─lvm-swap swap              bf480ef5-e5f0-43b9-9282-c3f1d0d446fa
    └─sdb2       crypto_LUKS       11f480d6-950b-40ff-9739-e8244a4b5a30

I o takie coś nam chodzi.

## Umieszczanie nagłówka LUKS na pendrive

Przy zaszyfrowanych dyskach i pewnych niestandardowych ustawieniach systemu, katalog `/boot/` zwykle
jest ulokowany na osobnej partycji. Możemy ten mechanizm wykorzystać pozbywając się jednocześnie
problemów z niezabezpieczonym initramfs, czyli modułami jądra i innymi plikami, które mają na celu
zamontowanie głównego systemu plików. Jakby nie patrzeć, wszystkie pliki na partycji `/boot/`
pozostają w formie odszyfrowanej, co czyni je podatnymi na zmianę bez jakiejkolwiek kontroli i
weryfikacji z naszej strony. Obejdziemy tym też problemy związane z kodem bootloader'a, który
znajduje się w MBR. Odpadną nam przy tym też wszelkie ataki wymierzone w ten sektor. Jedyna
niedogodność jest taka, że ten pendrive będzie wymagany do uruchomienia naszego systemu. Choć to ma
też swoje zalety.

Wsadzamy zatem pendrive to portu USB i montujemy jego partycję w systemie. Następnie kopiujemy całą
zawartość katalogu `/boot/` i plik nagłówka LUKS na partycję pendrive:

    # mount /dev/sdc1 /mnt/
    # cp -a /boot/* /mnt/
    # cp -a header.img /mnt/
    # sync

W ten sposób możemy odmontować partycję `/boot/` i zamontować w jej miejsce partycję pendrive:

    # umount /mnt/
    # umount /boot/
    # mount /dev/sdc1 /boot/

Edytujemy teraz plik `/etc/fstab` i dostosowujemy odpowiednie wpisy, przykładowo:

    UUID=ee759aa3-4977-4c08-b14f-a62d17a47478 /                 ext4 defaults 0 1
    UUID=1bbe97b4-9116-4366-843c-187ba5f0323f /boot             ext4 defaults 0 2
    UUID=355933fa-7c8a-46df-93d8-3d3395327f29 /media/storage    ext4 defaults 0 2

Musimy także odpowiednio dostosować sobie plik `/etc/crypttab` :

    storage   UUID=11f480d6-950b-40ff-9739-e8244a4b5a30   none    luks,header=/boot/header.img

Dlatego właśnie potrzebowaliśmy UUID kontenera. Z kolei w parametrze `header` podajemy lokalizację
nagłówka.

Musimy także przereinstalować bootloader. W tym przypadku jest to `grub2` :

    # dpkg-reconfigure grub-pc
    Installing for i386-pc platform.
    Installation finished. No error reported.
    Generating grub configuration file ...
    Found background image: /usr/share/images/desktop-base/desktop-grub.png
    Found linux image: /boot/vmlinuz-4.3.0-1-amd64
    Found initrd image: /boot/initrd.img-4.3.0-1-amd64
    done

W momencie, gdy zostaniemy poproszeni o podanie dysku, na którym ma zostać umieszczony kod
bootloader'a, wybieramy pendrive:

![](/img/2016/01/1.grub-instalacja-pendrive-luks-naglowek.png#medium)

Następnie regenerujemy initramfs:

    # update-initramfs -u -k all
    update-initramfs: Generating /boot/initrd.img-4.3.0-1-amd64

## Usługa dla systemd

Wszystkie zaszyfrowane kontenery LUKS w systemd są otwierane przed zamontowaniem partycji `/boot/` .
W takim przypadku ten kontener, którego nagłówek obcięliśmy, nie odpali nam się. Musimy zatem nieco
przerobić domyślną usługę dla systemd. Tworzymy plik `systemd-cryptsetup@storage.service` w katalogu
`/etc/systemd/system/` i wrzucamy do niego tę poniższą treść:

    [Unit]
    Description=Cryptography Setup for %I
    Documentation=man:crypttab(5) man:systemd-cryptsetup-generator(8) man:systemd-cryptsetup@.service(8)
    DefaultDependencies=no
    Conflicts=umount.target
    BindsTo=dev-mapper-%i.device
    IgnoreOnIsolate=true
    After=cryptsetup-pre.target
    After=boot.mount
    BindsTo=dev-disk-by\x2duuid-11f480d6\x2d950b\x2d40ff\x2d9739\x2de8244a4b5a30.device
    After=dev-disk-by\x2duuid-11f480d6\x2d950b\x2d40ff\x2d9739\x2de8244a4b5a30.device
    Before=umount.target

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    TimeoutSec=0
    ExecStart=/lib/systemd/systemd-cryptsetup attach 'storage' '/dev/disk/by-uuid/11f480d6-950b-40ff-9739-e8244a4b5a30' 'none' 'luks,header=/boot/header.img'
    ExecStop=/lib/systemd/systemd-cryptsetup detach 'storage'

Plik tej powyższej usługi został jedynie przekopiowany z katalogu `/run/systemd/generator/` i
dostosowany pod kątem zależności. Nie musimy go pisać od początku.

Przeładowujemy systemd i dodajemy tę usługę do autostartu:

    # systemctl daemon-reload
    # systemctl enable systemd-cryptsetup@storage.service

By sprawdzić czy systemd jest w stanie odszyfrować i zamontować ten zasób, możemy zwyczajnie
wystartować tę usługę lub też uruchomić komputer ponownie. Niemniej jednak, podczas startu systemu,
zostaniemy poproszeni o hasło. Jeśli nie chce nam się dodatkowo wpisywać hasła podczas startu, to
zawsze możemy korzystać z keyfile, który również można umieścić na pendrive. Choć nie zalecałbym
tego rozwiązania. Można także skorzystać z [keyring'u kernela, który jest w stanie przechowywać
hasła LUKS](/post/odszyfrowanie-kontenerow-luks-w-systemd/), o ile ma się
zaszyfrowaną jakąś partycję, najlepiej tę systemową. W ten sposób unikniemy wprowadzania wielu haseł
do zaszyfrowanych zasobów. Po odszyfrowaniu systemu, partycję `/boot/` można zwyczajnie odmontować,
a pendrive wyciągnąć. Pamiętajmy jednak, by nie aktualizować bootloader'a czy kernela, gdy partycja
`/boot/` jest niezamontowana.
