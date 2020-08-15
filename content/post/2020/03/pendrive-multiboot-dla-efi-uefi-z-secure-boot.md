---
author: Morfik
categories:
- Linux
date: "2020-03-23T21:50:00Z"
published: true
status: publish
tags:
- debian
- efi
- uefi
- multiboot
- pendrive
- secure-boot
- refind
- iso
title: Pendrive multiboot dla EFI/UEFI z Secure Boot
---

Przeniesienie mojego Debiana z laptopa mającego konfigurację BIOS i tablicę partycji MBR/MS-DOS do
maszyny wyposażonej w firmware EFI/UEFI nie było jakoś stosunkowo trudnym zadaniem. Nawet [kwestia
włączenia Secure Boot][1] okazała się o wiele mniej skomplikowana niż w rzeczywistości mogłoby się
człowiekowi wydawać. Problem jednak pojawił się w przypadku płytek czy pendrive z systemami live.
Nie chodzi przy tym o uruchamianie nośników z dopiero co wypalonymi obrazami ISO/IMG, bo te również
nie sprawiają kłopotów. Chodzi bardziej o rozwiązanie multiboot, które oferuje wgranie wielu
obrazów live na jedno urządzenie i odpalanie tego systemu, który sobie użytkownik w danym momencie
zażyczy. Do tej pory korzystałem z [projektu GLIM][2] i może on posiada wsparcie dla EFI/UEFI ale
już wsparcia dla Secure Boot mu zabrakło. W efekcie w konfiguracji EFI/UEFI + Secure Boot, GLIM stał
się bezużyteczny i trzeba było rozejrzeć się za nieco innym rozwiązaniem. Okazało się, że nie
trzeba daleko szukać, bo [rEFInd][3] jest w stanie natywnie uruchomić system z obrazu ISO
praktycznie każdej dystrybucji linux'a (Ubuntu/Debian/Mint/GParted/CloneZilla) i w zasadzie trzeba
tylko nieco inaczej przygotować nośnik, by móc na nowo cieszyć się korzyściami jakie oferuje
pendrive multiboot.

<!--more-->
## Tablica partycji GPT

By stworzyć pendrive multiboot, który ruszy nam na komputerze z firmware EFI/UEFI z włączonym
Secure Boot, potrzebny nam będzie kawałek nośnika, który podzielimy na kilka partycji. Przede
wszystkim potrzebujemy utworzyć na takim nośniku tablicę partycji GPT. Możemy do tego celu
wykorzystać standardowo `gdisk` lub `gparted` . W tym przypadku będzie wykorzystany `gdisk` .
Podłączamy zatem pendrive do portu USB komputera i wydajemy w terminalu te poniższe polecenia:

    # gdisk /dev/mmcblk0
    GPT fdisk (gdisk) version 1.0.5

    Partition table scan:
      MBR: not present
      BSD: not present
      APM: not present
      GPT: not present

    Creating new GPT entries in memory.

    Command (? for help): o
    This option deletes all partitions and creates a new protective MBR.
    Proceed? (Y/N): y

    Command (? for help): p
    Disk /dev/mmcblk0: 60506112 sectors, 28.9 GiB
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): F6E3888A-F13D-444F-88DC-2ECD91E61B05
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 60506078
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 60506045 sectors (28.9 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name

### Partycja ESP

Technicznie rzecz biorąc, to nie ma potrzeby tworzyć osobnej partycji ESP na pendrive w przypadku,
gdy mamy zainstalowanego rEFInd'a na dysku komputera. Niemniej jednak, biorąc pod uwagę, że
dystrybucje zwykle instalują Grub2 jako bootloader, to dobrze jest stworzyć małą partycję ESP na
pendrive i w niej zainstalować rEFInd'a. Tworzymy zatem niewielką partycję ESP, której rozmiar może
nie przekraczać 100M. Określamy jej typ jako `EF00` :

    Command (? for help): n
    Partition number (1-128, default 1):
    First sector (34-60506078, default = 2048) or {+-}size{KMGTP}:
    Last sector (2048-60506078, default = 60506078) or {+-}size{KMGTP}: +100M
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): ef00
    Changed type of partition to 'EFI system partition'

    Command (? for help): p
    Disk /dev/mmcblk0: 60506112 sectors, 28.9 GiB
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): F6E3888A-F13D-444F-88DC-2ECD91E61B05
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 60506078
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 60301245 sectors (28.8 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048          206847   100.0 MiB   EF00  EFI system partition

Zapisujemy tablicę partycji na nośniku:

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): y
    OK; writing new GUID partition table (GPT) to /dev/mmcblk0.
    The operation has completed successfully.

Formatujemy teraz partycję ESP z wykorzystaniem systemu plików FAT32:

    # mkfs.vfat -v -n SD_ESP /dev/mmcblk0p1

W przypadku tworzenia partycji ESP w `gparted` , pamiętajmy o ustawieniu jej flagi `esp` .

## Partycje pod obrazy ISO/IMG

Teraz musimy sobie zadać kilka pytań. Pierwsze z nich dotyczy kwestii ilości obrazów ISO/IMG, które
nasz pendrive multiboot ma obsługiwać. Drugie pytanie tyczy się zaś rozmiarów samych obrazów. Od
tych dwóch rzeczy zależeć będzie sposób w jaki zostanie podzielona pozostała przestrzeń pendrive'a.
Z reguły systemy live głównych dystrybucji takich jak Debian czy Ubuntu, nie przekraczają rozmiarowo
3G. Zatem można utworzyć kilka partycji o rozmiarach 3G każda. Jeśli potrzebujemy wgrać na pendrive
4 obrazy live, to trzeba będzie utworzyć 4 takie partycje -- po jednej na każdy obraz ISO/IMG. W
tym przypadku będą potrzebne 4 partycje, na które zostaną wgrane obrazy live Ubuntu LTS, Ubuntu
Latest, Linux Mint oraz Debian ze środowiskiem Gnome. Tworzymy zatem przy pomocy `gdisk` te cztery
dodatkowe partycje:

    Command (? for help): n
    Partition number (2-128, default 2):
    First sector (34-60506078, default = 206848) or {+-}size{KMGTP}:
    Last sector (206848-60506078, default = 60506078) or {+-}size{KMGTP}: +3000M
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300):
    Changed type of partition to 'Linux filesystem'

Nazwijmy sobie jeszcze w miarę po ludzku poszczególne partycje:

    Command (? for help): c
    Partition number (1-5): 2
    Enter name: Ubuntu LTS

    Command (? for help): c
    Partition number (1-5): 3
    Enter name: Ubuntu Latest

    Command (? for help): c
    Partition number (1-5): 4
    Enter name: Linux Mint

    Command (? for help): c
    Partition number (1-5): 5
    Enter name: Debian Gnome

Powinniśmy mieć mniej więcej poniższy układ partycji:

    Command (? for help): p
    Disk /dev/mmcblk0: 60506112 sectors, 28.9 GiB
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): B36B76F8-412C-4150-A476-312A656F71AB
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 60506078
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 35725245 sectors (17.0 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048          206847   100.0 MiB   EF00  sd_esp
       2          206848         6350847   2.9 GiB     8300  Ubuntu LTS
       3         6350848        12494847   2.9 GiB     8300  Ubuntu Latest
       4        12494848        18638847   2.9 GiB     8300  Linux Mint
       5        18638848        24782847   2.9 GiB     8300  Debian Gnome

Zapisujemy tablicę partycji:

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): Y
    OK; writing new GUID partition table (GPT) to /dev/mmcblk0.
    The operation has completed successfully.

## Wgrywanie obrazów live na pendrive

Pobieramy naturalnie stosowne obrazy live udostępniane prze dystrybucje [Ubuntu][4],
[Linux Mint][5] i [Debiana][6]. Następnie przy pomocy narzędzia `dd` wgrywamy kolejno każdy obraz
na osobne urządzenie blokowe. Ważne tutaj jest by podać ścieżkę do partycji, a nie do nośnika jako
takiego. W tym przypadku nośnik jest pod `/dev/mmcblk0` , a partycje pod `/dev/mmcblk0p1`
`/dev/mmcblk0p2` , etc.

    # dd if=/media/morfik/GLIM/boot/iso/ubuntu/ubuntu-18.04.4-desktop-amd64.iso of=/dev/mmcblk0p2 status=progress bs=1M
    # dd if=/media/morfik/GLIM/boot/iso/ubuntu/ubuntu-19.10-desktop-amd64.iso of=/dev/mmcblk0p3 status=progress bs=1M
    # dd if=/media/morfik/GLIM/boot/iso/linuxmint/linuxmint-19.3-mate-64bit.iso' of=/dev/mmcblk0p4 status=progress bs=1M
    # dd if=/media/morfik/GLIM/boot/iso/debian/debian-live-10.3.0-amd64-gnome.iso of=/dev/mmcblk0p5 status=progress bs=1M

Po wgraniu obrazów, tak się prezentować powinien nasz pendrive w `lsblk` :

    # lsblk /dev/mmcblk0
    NAME         SIZE FSTYPE  TYPE LABEL                       MOUNTPOINT UUID
    mmcblk0     28.9G         disk
    ├─mmcblk0p1  100M vfat    part SD_ESP                                 AD3E-8645
    ├─mmcblk0p2    3G iso9660 part Ubuntu 18.04.4 LTS amd64               2020-02-03-18-40-13-00
    ├─mmcblk0p3    3G iso9660 part Ubuntu 19.10 amd64                     2019-10-17-12-53-34-00
    ├─mmcblk0p4    3G iso9660 part Linux Mint 19.3 MATE 64-bit            2019-12-16-14-03-06-00
    └─mmcblk0p5    3G iso9660 part d-live 10.3.0 gn amd64                 2020-02-08-12-48-50-00

Jak widać każda partycja ma etykietę obrazu ISO/IMG i każdą partycję można zwyczajnie zamontować
w systemie, choć będzie ona dostępna tylko do odczytu.

A tu jest jeszcze widok w `gparted` :

![]({{< baseurl >}}/img/2020/03/001-efi-uefi-firmware-secure-boot-linux-live-system-iso-refind-gparted.png#huge)

Ta niewykorzystana część nośnika może zostać przeznaczona na dodatkowe partycje pod obrazy ISO/IMG
lub też można tam stworzyć zwykłą partycję na dane. Można tam nawet upchnąć zwykły system i rEFInd
również będzie w stanie go uruchomić bez większego problemu. Zatem jest tutaj pełna dowolność
jeśli chodzi o konfigurację.

## Instalacja rEFInd na partycji ESP

Kolejnym krokiem jest wgranie menadżera rozruchu rEFInd na partycję ESP pendrive'a. Warto tutaj
wspomnieć, że jeśli mamy na dysku komputera zainstalowanego rEFInd'a, to on bez większego problemu
będzie w stanie ustalić co za systemy znajdują się na pendrive i zwróci nam listę wszystkich
obrazów ISO/IMG, które na ten nośnik zostały wgrane. Potrzebny nam będzie jedynie sterownik
`iso9660_x64.efi` .

Niemniej jednak, jeśli chcemy by nasz pendrive był niezależny od oprogramowania komputera, do
którego będziemy podłączać ten nośnik, to lepiej jest zainstalować rEFInd'a na na partycji ESP
pendrive'a. Ten proces instalacji nie różni się zbytnio od instalacji rEFInd'a na dysku twardym i
sprowadza się do wydania poniższego polecenia:

    # refind-install --usedefault /dev/mmcblk0p1
    ShimSource is none
    Installing rEFInd on Linux....
    Note: IA32 (x86) binary not installed!
    Installing driver for ext4 (ext4_x64.efi)
    Copied rEFInd binary files

    Copying sample configuration file as refind.conf; edit this file to configure
    rEFInd.

Przy instalowaniu rEFInd'a bez uprzednio zamontowanej partycji ESP, na potrzeby procesu
instalacyjnego zostanie ona automatycznie zamontowana w katalogu `/tmp/refind_install/` .

Warto tutaj dodać, że można także skorzystać z opcji `--localkeys` , tak by podczas instalacji
rEFInd'a jego binarki (w tym sterowniki) zostały automatycznie podpisane (o czym za chwile).

### Sterownik iso9660_x64.efi

Jak można było zauważyć wyżej, domyślnie został wgrany sterownik `ext4_x64.efi` . Obrazy ISO/IMG
potrzebują zaś sterownika `iso9660_x64.efi` , który znajduje się w katalogu
`/usr/share/refind/refind/drivers_x64/` . Kopiujemy ten sterownik na partycję ESP (można także
usunąć sterownik `ext4_x64.efi` z pendrive'a, bo nie będzie on nam do niczego potrzebny):

    # cp /usr/share/refind/refind/drivers_x64/iso9660_x64.efi /tmp/refind_install/EFI/BOOT/drivers_x64/
    # rm /tmp/refind_install/EFI/BOOT/drivers_x64/ext4_x64.efi

W przypadku gdybyśmy mieli rEFInd'a wgranego na partycję ESP dysku komputera, to wystarczyłoby
skopiować sterownik `iso9660_x64.efi` na partycję ESP komputera i kłopot z głowy.

### Podpisanie rEFInd'a i jego sterowników

Standardowo w trybie Secure Boot żadna niepodpisana binarka nie zostanie uruchomiona przez firmware
EFI/UEFI komputera. W przypadku rEFInd i jego sterowników trzeba będzie zadbać o to by taki podpis
pod tymi plikami się pojawił. Mamy w zasadzie trzy opcje do wyboru.

Pierwszą z nich jest skorzystanie z [podpisanych plików przez dewelopera rEFInd'a][8]. W takim
przypadku trzeba będzie dodać certyfikat rEFInd'a do bazy danych MOK i po problemie. To rozwiązanie
jest także najbardziej uniwersalne jeśli chodzi o podłączanie pendrive na różnych maszynach, bo
wystarczy na każdej takiej maszynie dodać ten konkretny certyfikat (jeśli nie będzie go w
standardzie) i już można rEFInd'a uruchomić z takiego nośnika. Niemniej jednak, w Debianie póki co
pliki rEFInd'a nie są w żaden sposób podpisane, więc trzeba będzie zainstalować rEFInd'a pobierając
 odpowiednie pliki z podlinkowanego wyżej artykułu, co niekoniecznie może być wygodne, przynajmniej
 na razie.

W przypadku, gdy [wymieniliśmy certyfikaty firmware EFI/UEFI][1], to w zasadzie wystarczy podpisać
swoim kluczem prywatnym `db` binarkę rEFInd'a oraz jego sterowniki i po sprawie, przynajmniej jeśli
chodzi o uruchamianie rEFInd'a z pendrive na naszym komputerze. Jeśli chodzi zaś o inne maszyny, to
trzeba będzie nasz certyfikat dodać do bazy danych MOK firmware EFI/UEFI każdego komputera, na
którym zamierzamy korzystać z naszego pendrive'a.

Z kolei ostania opcja sprowadza się do zainstalowania rEFInd'a z flagą `--localkeys` . W taki
sposób podczas instalacji rEFInd'a na partycji ESP pendrive'a automatycznie zostaną stworzone
stosowne klucze i umieszczone w katalogu `/etc/refind.d/keys/` . Następnie instalowane binarki
przed skopiowaniem i umieszczeniem na partycji ESP zostaną podpisane. W takim przypadku trzeba
będzie zaimportować do bazy MOK certyfikat, który siedzi w katalogu `/etc/refind.d/keys/` na każdej
maszynie, na której mamy zamiar używać naszego pendrive'a.

Jako, że ja wymieniłem u siebie klucze w firmware EFI/UEFI na swoje własne, to mi wygodniej jest
podpisać binarkę rEFInd'a i sterowniki swoim kluczem prywatnym:

    # cd /etc/kernel_key/

    # sbsign --cert db.crt --key db.key /tmp/refind_install/EFI/BOOT/bootx64.efi
    Signing Unsigned original image

    # mv /tmp/refind_install/EFI/BOOT/bootx64.efi.signed /tmp/refind_install/EFI/BOOT/bootx64.efi

    # sbsign --cert db.crt --key db.key /tmp/refind_install/EFI/BOOT/drivers_x64/iso9660_x64.efi
    Signing Unsigned original image

    # mv /tmp/refind_install/EFI/BOOT/drivers_x64/iso9660_x64.efi.signed /tmp/refind_install/EFI/BOOT/drivers_x64/iso9660_x64.efi

## Certyfikaty dystrybucji systemów live

Jest bardzo dużo dystrybucji linux'a, z którymi możemy się zetknąć, i część z nich oferować będzie
obrazy z systemami live. To czy obraz konkretnej dystrybucji nam ruszy zależy już od wsparcia
takiej dystrybucji dla Secure Boot. Jeśli dystrybucja nie wspiera Secure Boot to nam się jej system
live nie uruchomi. Debian i pochodne, takie jak Ubuntu czy Linux Mint, wspierają Secure Boot i
wystarczy wgrać certyfikaty tych dystrybucji do bazy danych MOK, by te obrazy live działały bez
większego problemu, tak jak to zostało zobrazowane niżej. [Opis dodawania certyfikatów Debiana i
Ubuntu do bazy MOK][7] znajduje się tutaj.

Wygląda też na to, że spora część mniejszych dystrybucji, takich jak GParted albo CloneZilla,
korzysta z binarek Grub'a i shim'a podpisanych odpowiednio przez Debian/Canonical i Microsoft. W
takiej sytuacji mając certyfikaty tych trzech podmiotów zaszyte w firmware EFI/UEFI komputera nie
powinno być problemu z uruchomieniem większości systemów live.

## Test multiboot z włączonym Secure Boot

Przyszła pora by uruchomić ponownie komputer z podłączonym do portu USB pendrive'm, który sobie
wyżej utworzyliśmy. Menadżer rozruchu komputera powinien wykryć bez problemu, że na pendrive
znajduje się dodatkowa opcja rozruchu, co widoczne jest jako pierwsza ikonka na liście systemów
mająca w swoim prawym dolnym rogu małą ikonkę pendrive:

![]({{< baseurl >}}/img/2020/03/002-efi-uefi-firmware-secure-boot-linux-live-system-iso-refind-boot-screen-disk.jpg#huge)

Gdy wejdziemy w tę pozycję, to rEFInd z pendrive zostanie załadowany i zwróci nam listę obrazów
ISO/IMG, które mamy wgrane na poszczególne partycje:

![]({{< baseurl >}}/img/2020/03/003-efi-uefi-firmware-secure-boot-linux-live-system-iso-refind-boot-screen-pendrive.jpg#huge)

Uruchommy przykładowo Ubuntu LTS. Powinien się załadować Grub2 obrazu ISO/IMG, w którym wybieramy
pierwszą pozycję, by uruchomić Ubuntu live:

![]({{< baseurl >}}/img/2020/03/004-efi-uefi-firmware-secure-boot-linux-live-system-iso-boot-ubuntu.jpg#huge)

![]({{< baseurl >}}/img/2020/03/005-efi-uefi-firmware-secure-boot-linux-live-system-iso-ubuntu-booted.jpg#huge)

Podobnie sprawa wygląda w przypadku Ubuntu Latest:

![]({{< baseurl >}}/img/2020/03/006-efi-uefi-firmware-secure-boot-linux-live-system-iso-ubuntu-boot.jpg#huge)

![]({{< baseurl >}}/img/2020/03/007-efi-uefi-firmware-secure-boot-linux-live-system-iso-ubuntu-booted.jpg#huge)

W przypadku Linux Mint jest podobnie:

![]({{< baseurl >}}/img/2020/03/008-efi-uefi-firmware-secure-boot-linux-live-system-iso-mint-boot.jpg#huge)

![]({{< baseurl >}}/img/2020/03/009-efi-uefi-firmware-secure-boot-linux-live-system-iso-mint-booted.jpg#huge)

No i jeszcze na koniec Debian:

![]({{< baseurl >}}/img/2020/03/010-efi-uefi-firmware-secure-boot-linux-live-system-iso-debian-boot.jpg#huge)

![]({{< baseurl >}}/img/2020/03/011-efi-uefi-firmware-secure-boot-linux-live-system-iso-debian-booted.jpg#huge)

Co ciekawe, mając wgrany sterownik `iso9660_x64.efi` na partycji ESP dysku twardego komputera,
możemy w zasadzie obyć się bez zewnętrznych nośników USB by system live odpalić. Wystarczy utworzyć
niewielką partycję na dysku i tam wgrać obraz live. W przypadku awarii głównego systemu będzie
można uruchomić system live i dokonać prac naprawczych -- taki dość zaawansowany tryb recovery.

## Czy mój pendrive multiboot zadziała na innych komputerach

Ten pendrive, co go przygotowaliśmy, będzie w stanie bez problemu działać w zasadzie na każdym
komputerze mającym firmware EFI/UEFI. Na komputerach z BIOS taki pendrive będzie miał problem się
uruchomić, bo rEFInd działa tylko z firmware EFI/UEFI. Choć pewnie w niedługim czasie Grub2 będzie
w stanie czytać obrazy ISO/IMG wgrane na partycje i bezpośrednio z nich uruchamiać system dokładnie
w taki sam sposób jak to robi rEFInd, jeśli oczywiście Grub2 jeszcze tego nie potrafi.

Jeśli zaś chodzi o maszyny z firmware EFI/UEFI i mające do tego włączony tryb Secure Boot, to
niestety nasz pendrive nie zadziała na takich maszynach bez uprzedniego zaimportowania stosownych
certyfikatów do bazy danych MOK.

Na pewno będzie potrzebny certyfikat klucza, którym podpisana jest binarka rEFInd'a i sterownik.
Dodatkowo, będą potrzebne certyfikaty każdej dystrybucji, którą mamy zamiar uruchamiać z naszego
pendrive multiboot.

Jeśli już jakaś maszyna działa w trybie Secure Boot pod kontrolą linux'a, to ma zaszyty certyfikat
swojej dystrybucji i ta pozycja live na pendrive będzie działać. Gdyby binarka rEFInd'a i jego
sterowniki były podpisane kluczem dystrybucji mniej więcej w taki sam sposób jak podpisany jest
Grub2, to odpadłoby ręczne dodawanie certyfikatów do bazy danych MOK. Póki ta binarka jest
niepodpisana, to trzeba niestety kombinować.

## Jak aktualizować obrazy ISO/IMG

Pewnie kiedyś w przyszłości dana dystrybucja linux'a wypuści nowszą płytkę z systemem live. Czy w
takim przypadku trzeba będzie przechodzić przez cały proces opisany wyżej (lub jego większą część)?
Odpowiedź brzmi -- nie. Z chwilą, gdy wpadnie nam w łapki nowy obraz ISO/IMG, to wgrywamy go przy
pomocy `dd` na konkretną partycję i koniec zabawy. Nie trzeba przy tym bawić się partycjami czy
zmieniać konfiguracji w którymś miejscu, bo sama tablica partycji wraz z numerami UUID pozostaje
taka sama. Jedynie co to zawartość partycji ulega zmianie, a przecie o to chodzi przy aktualizacji
obrazu ISO/IMG. Jak widać cały proces aktualizacji obrazu ISO/IMG jest maksymalnie uproszczony i
sprowadza się do wydania w zasadzie jednego polecenia w terminalu.

## Jak poprawić wygląd ekranu rozruchowego

Domyślny wygląd rEFInd'a nie zachwyca zbytnio i jeśli komuś on przeszkadza, to zawsze może go sobie
dostosować (tak jak ja to zrobiłem wyżej) z racji, że konfiguracja rEFInd'a jest przechowywana na
partycji ESP. Możemy sobie tam wgrać tapetę, ikonki, czcionki i skonfigurować to wszystko za sprawą
pliku `EFI/BOOT/refind.conf` .

## Jak dostosować pozycje w menu

Domyślnie rEFInd za sprawą swoich sterowników skanuje w poszukiwaniu systemów, które jest w stanie
uruchomić. Mając w zasadzie jeden sterownik ( `iso9660_x64.efi` ), to w menu rozruchowym zostaną
nam wylistowane wszystkie systemy, które ten sterownik wykryje. Czasami to automatyczne wykrywanie
nie działa za dobrze lub jest niepożądane i chcielibyśmy ręcznie skonfigurować to jakie systemy
pojawią się na liście w menu rozruchowym. Mamy naturalnie taką możliwość przez dostosowanie pliku
`EFI/BOOT/refind.conf` . Trzeba w nim na samym początku określić:

    scanfor manual

Następnie tworzymy zwrotki `menuentry` podobne do tej poniżej:

    menuentry "Ubuntu LTS" {
        icon EFI/BOOT/icons/os_ubuntu.png
        volume 67F11057-F4C1-4504-82EC-B6EE2A558B9F
        loader   /EFI/BOOT/grubx64.efi
    #    disabled
    }

[Opcji naturalnie jest więcej][9] ale chyba tylko te określone wyżej będą nas interesować
najbardziej. Pozycja `icon` przyjmuje ścieżkę względem zamontowanej partycji ESP. Numerek UUID
widoczny w `volume` odnosi się do UUID partycji, który można wyciągnąć z `sgdisk` :

    # sgdisk -i 2 /dev/mmcblk0
    ...
    Partition unique GUID: 67F11057-F4C1-4504-82EC-B6EE2A558B9F
    ...
    Partition name: 'Ubuntu LTS'

Jeśli zaś chodzi o `loader` to tutaj ścieżka jest określana względem tego co znajduje się w obrazie.
Ścieżka `/EFI/BOOT/grubx64.efi` znajdzie zastosowanie w sporej części przypadków ale nie we
wszystkich. Jeśli z jakiegoś powodu obraz ISO/IMG nie będzie się chciał uruchomić, to trzeba
zamontować partycję z obrazem ISO/IMG i znaleźć ścieżkę do bootloader'a Grub2. Opcja `disabled` ,
jeśli określona, sprawi, że w menu nie pokaże się ten wpis.

W taki oto sposób mamy całkowitą kontrolę nad tym jakie obrazy live się pojawią w menu. Jest to
bardzo użyteczna funkcja zwłaszcza w przypadku, gdy mamy do czynienia z większych rozmiarów
pendrive'm i tych partycji mamy na nim paręnaście czy kilkadziesiąt.

## Podsumowanie

Jakby nie patrzeć, Secure Boot trochę miesza przy korzystaniu z systemów live dystrybucji linux'a.
Niemniej jednak, jeśli chodzi o firmware EFI/UEFI bez włączonego Secure Boot, to proces utworzenia
pendrive multiboot jest wręcz banalny, tak samo jak i jego późniejsza obsługa i utrzymanie.

Największym problemem dla pendrive multiboot nie jest jako taki Secure Boot ale certyfikaty,
których zwykle brakuje w firmware EFI/UEFI. Bez uprzedniego stworzenia/dodania odpowiednich
certyfikatów, użytek z takiego nośnika live jest praktycznie żaden. Na szczęście jeśli nasz linux
działa już z włączonym Secure Boot, to obrazy live tej dystrybucji również będą nam śmigać bez
problemu, choć pewnie trzeba będzie podpisać samemu binarki rEFInd'a czy też dodać jego certyfikat
do bazy MOK ale to zadanie również do trudnych nie należy.

By poradzić sobie jakoś z portowalnością pendrive multiboot, dobrze jest trzymać stosowne
certyfikaty na partycji ESP, by w razie potrzeby je łatwo zaimportować na docelowej maszynie.


[1]: {{< baseurl >}}/post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/
[2]: https://github.com/thias/glim
[3]: https://www.rodsbooks.com/refind/
[4]: https://ubuntu.com/download/desktop
[5]: https://www.linuxmint.com/download.php
[6]: https://www.debian.org/CD/live/
[7]: {{< baseurl >}}/post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/#dodawanie-kluczy-dystrybucji-do-bazy-mok
[8]: https://www.rodsbooks.com/refind/getting.html
[9]: https://www.rodsbooks.com/refind/configfile.html
