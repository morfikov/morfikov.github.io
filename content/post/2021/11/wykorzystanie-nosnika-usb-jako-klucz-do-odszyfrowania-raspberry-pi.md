---
author: Morfik
categories:
- RaspberryPi
date:    2021-11-11 05:32:00 +0100
lastmod: 2021-11-11 05:32:00 +0100
published: true
status: publish
tags:
- luks
- raspberry-pi-4b
- raspios
- raspbian
- szyfrowanie
- udev
GHissueID: 580
title: Wykorzystanie nośnika USB jako klucz do odszyfrowania Raspberry Pi (LUKS)
---

Ostatnio udało mi się [zaszyfrować swojego Raspberry Pi 4B przez wdrożenie kontenera LUKS][1] i
upchnięcie w nim wszystkich danych z partycji `/` zainstalowanego w tym urządzeniu systemu
RasPiOS/Raspbian. To jednak nie był koniec pracy, bo taki zaszyfrowany system na RPI ma szereg wad.
Tym główniejszym uniedogodnieniem jest wpisywanie hasła na początku fazy boot, by pełny start
systemu był w ogóle możliwy. Sytuacja się komplikuje, gdy do naszego minikomputera nie mamy
podłączonego monitora i/lub klawiatury. Ten zaistniały problem można rozwiązać na kilka sposobów,
np. przez zaprzęgnięcie do pracy dodatkowego nośnika USB w celu umieszczenia na nim pliku klucza
(keyfile). Niemniej jednak, można pójść o krok dalej i wykorzystać samo urządzenie jako klucz i gdy
takiego pendrive nie podłączymy do portu USB naszej maliny, to system nam się nie uruchomi. Te dwa
rozwiązania są bardzo podobne do siebie ale to drugie jest nieco bardziej odporne na ewentualne
problemy ze skasowaniem pliku klucza, co może nam się przytrafić, gdy taki pendrive jest
wykorzystywany w roli regularnego nośnika danych przechowującego dla niepoznaki jakieś pliki. Tak
czy inaczej oba te sposoby zostaną opisane w niniejszym artykule.

<!--more-->
## Nagłówek LUKS i przechowywane w nim klucze

Jakiś czas temu opisywałem [sposób wykorzystania nośnika USB jako klucza do odblokowania kontenera
LUKS][2] ale tamto rozwiązanie było opracowane na potrzeby zaszyfrowanych partycji innych niż ta
systemowa. To opisane poniżej rozwiązanie nie jest specyficzne jedynie dla systemu RasPiOS/Raspbian
i może być wykorzystane na dowolnym linux'ie.

Mając zaszyfrowany system w Raspberry Pi, w nagłówku LUKS powinien być zajęty jeden keyslot. To
właśnie w tym slocie znajduje się klucz, którym zaszyfrowany jest klucz główny używany do
szyfrowania/deszyfrowania danych na partycji. Ten klucz w slocie jest również zaszyfrowany, tj. by
ten klucz ze slotu wydobyć, trzeba podać hasło, które standardowo wpisujemy przy otwieraniu
kontenera. Zatem wpisujemy hasło, klucz w slocie jest deszyfrowany, a potem przy pomocy tego klucza
deszyfrowany jest klucz główny, który wędruje do pamięci RAM i później szyfrowanie danych odbywa
się w sposób transparentny dla nas.

Nie tylko hasło może pełnić rolę klucza do kontenera LUKS. Inną formą klucza jest plik binarny,
który może rezydować w obrębie systemu plików danej partycji. Gdy korzystamy z takiej formy klucza,
to system przy dostępie do kontenera LUKS odczytuje stosowny plik ze wskazanej na dysku lokalizacji.
Taki plik to nic innego jak niezmienna sekwencja bitów, które podawane są do `cryptsetup` i
przetwarzane. Jeśli dany kefile znajduje się również w nagłówku LUKS, to klucz główny zostanie
zdeszyfrowany automatycznie, przez co my nie musimy wpisywać żadnego hasła, co niesamowicie
usprawnia cały proces odszyfrowania kontenera i przez to też startu systemu.

Poniżej jest przykład nagłówka kontenera LUKS, w którym zamknięty jest system mojego Raspberry Pi:

    root@raspberrypi:/home/pi# cryptsetup luksDump /dev/mmcblk0p2

    LUKS header information
    Version:       	2
    Epoch:         	11
    Metadata area: 	16384 [bytes]
    Keyslots area: 	16744448 [bytes]
    UUID:          	0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be
    Label:         	rpi
    Subsystem:     	(no subsystem)
    Flags:       	(no flags)

    Data segments:
      0: crypt
        offset: 16777216 [bytes]
        length: (whole device)
        cipher: aes-xts-plain64
        sector: 512 [bytes]

    Keyslots:
      0: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        Cipher key: 512 bits
        PBKDF:      argon2i
        Time cost:  4
        Memory:     524288
        Threads:    2
        Salt:       e5 83 26 da 06 e1 08 60 a0 20 08 60 00 b3 5f 9d
                    13 bf f3 46 2e 9c cc 78 dd 9a c5 90 b0 3c e8 4b
        AF stripes: 4000
        AF hash:    sha512
        Area offset:32768 [bytes]
        Area length:258048 [bytes]
        Digest ID:  0
    Tokens:
    Digests:
      0: pbkdf2
        Hash:       sha512
        Iterations: 119809
        Salt:       28 c0 19 bf 10 5b 2a c2 79 64 d6 5f 2f 3a 50 c1
                    be 71 27 f1 f0 2d 6b bc 2e ee 94 6f b3 0b 5c 85
        Digest:     c2 90 f2 b6 f0 58 39 65 4d 36 ba 50 63 70 bf 00 5d 10 db b2 50 91 11 09 c8 28 00 72 12 5d 9e 57
                    4a 55 c0 9f 25 d8 17 cb 62 4e 87 d9 92 ae 77 7b 4e 14 50 b9 0c c8 92 2a e2 49 b1 d9 9e 6b 75 41

Mamy tutaj całą masę informacji ale nas najbardziej interesuje zwrotka z `Keyslots` . Jak widać
mamy tylko jeden keyslot na pozycji `0` . To właśnie tutaj jest trzymany zaszyfrowany klucz
wykorzystywany do zdeszyfrowania klucza głównego po wpisaniu hasła. By móc zrobić użytek z keyfile,
musimy go dodać do kolejnego slotu w tym nagłówku.

### Tworzenie ramdysku na potrzeby wygenerowania keyfile

Zanim przejdziemy do generowania samego keyfile, dobrze jest stworzyć sobie niewielkich rozmiarów
ramdysk, co umożliwi nam wygenerowanie pliku klucza bezpośrednio w pamięci operacyjnej RAM. Takie
działanie ma na celu poprawę bezpieczeństwa pliku klucza ze względu na fakt, że nie zostanie on
zapisany na dysku twardym, z którego bardzo ciężko jest go później usunąć, zwłaszcza w przypadku
pamięci flash. Oczywiście jeśli mamy zaszyfrowany dysk, to ten problem dotyczy nas w mniejszym
stopniu ale lepiej wyrabiać sobie dobre nawyki od samego początku.

Taki ramdysk można stworzyć na dowolnym linux'ie w poniższy sposób:

    # mount -t tmpfs -o uid=root,gid=root,mode=1700,size=10M tmpfs /mnt

    # mount | grep -i mnt
    tmpfs on /mnt type tmpfs (rw,relatime,size=10240k,mode=1700)

Wszystkie dane zapisane w tym ramdysku zostaną usunięte wraz z odłączeniem zasilania od pamięci
operacyjnej, np. po wyłączeniu komputera, więc po utworzonym przez nas keyfile nie pozostanie żaden
ślad.

### Generowanie keyfile

Generujemy teraz keyfile przy pomocy narzędzia `dd` . Jako plik wejściowy podajemy urządzenie
`/dev/random` , a jako wyjściowy określamy ścieżkę do zapisu gdzieś w obrębie utworzonego ramdysku.
Rozmiar klucza możemy podać 2048/4096 bajtów. Większych kluczy nie ma co tworzyć, bo liczba
kombinacji oferowanych przez taki 4096-bajtowy keyfile jest i tak o wiele większa niż w przypadku
nawet sporych długości haseł alfanumerycznych. Wykorzystanie urządzenia `/dev/random` ma na celu
zapewnić większą losowość w stosunku do urządzenia `/dev/urandom` :

    root@raspberrypi:/home/pi# dd if=/dev/random of=/mnt/keyfile bs=1 count=4096

    root@raspberrypi:/home/pi# ls -al /mnt/keyfile
    -rw-r--r-- 1 root root 4096 Nov 10 05:05 /mnt/keyfile

### Dodawanie pliku klucza do nagłówka LUKS

Tak wygenerowany keyfile trzeba dodać do nagłówka zaszyfrowanego kontenera LUKS. Robimy to w
poniższy sposób:

    root@raspberrypi:/home/pi# cryptsetup luksAddKey /dev/mmcblk0p2 /mnt/keyfile --hash sha512
    Enter any existing passphrase:

W nagłówku LUKS powinien pojawić się dodatkowy keyslot z numerkiem `1` . Sprawdźmy czy faktycznie
tak się stało:

    root@raspberrypi:/home/pi# cryptsetup luksDump /dev/mmcblk0p2

    LUKS header information
    ...
      1: luks2
        Key:        512 bits
        Priority:   normal
        Cipher:     aes-xts-plain64
        Cipher key: 512 bits
        PBKDF:      argon2i
        Time cost:  4
        Memory:     185496
        Threads:    4
        Salt:       79 04 9f 36 26 2f da 5d 1c c0 a1 be 8a 73 6f c5
                    d8 c7 55 97 a0 cf ee 5c ec ae 20 11 06 d0 27 62
        AF stripes: 4000
        AF hash:    sha512
        Area offset:290816 [bytes]
        Area length:258048 [bytes]
        Digest ID:  0
    ...

Klucz został z powodzeniem dodany do nagłówka LUKS.

## Umieszczanie pliku klucza na pendrive

Dodanie klucza do nagłówka LUKS to tylko połowa pracy. Ten keyfile trzeba także wrzucić na pendrive,
który zamierzamy wykorzystać do otworzenia zaszyfrowanego kontenera. W tym przypadku keyfile
zostanie wgrany w pewne określone miejsce na pendrive z pominięciem systemu plików. Oczywiście
takie rozwiązanie sprawiłoby, że nasz klucz byłby bardzo podatny na uszkodzenia wskutek wgrywania
na nośnik jakichś informacji, np. plików. Dlatego też ten keyfile powędruje w miejsce, w którym
pliki nie są umieszczane, tj. w przestrzeni MBR-GAP.

### MBR-GAP

MBR-GAP to przestrzeń za tablicą partycji (pierwszym sektorem nośnika), której granicę wyznacza
początek pierwszej partycji dysku. Zwykle partycje są równane do 1MiB, więc MBR-GAP w takim
przypadku to 2047 sektorów 512-bajtowych. Tę przestrzeń trzeba zapisać losowymi danymi, po czym w
którymś miejscu umieścić wcześniej wygenerowany keyfile. By mieć pewność, że to właśnie ten obszar
trzeba wyczyścić, najlepiej sprawdzić offset pierwszej partycji via `fdisk` :

    root@raspberrypi:/home/pi# fdisk -l /dev/sda
    Disk /dev/sda: 7.3 GiB, 7864320000 bytes, 15360000 sectors
    Disk model: DataTraveler 3.0
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x8e8b4a71

    Device     Boot Start      End  Sectors  Size Id Type
    /dev/sda1        2048 15359999 15357952  7.3G 83 Linux

W kolumnie `Start` mamy wartość `2048` , zatem ta partycja jest wyrównana do 1MiB.

Czyścimy 2047 sektorów przy pomocy `dd` z pominięciem pierwszego, na którym znajduje się tablica
partycji:

    root@raspberrypi:/home/pi# dd if=/dev/urandom of=/dev/sda bs=512 count=2047 seek=1

Wczytajmy tablicę partycji na nowo, by sprawdzić czy wszystko z nią jest w porządku::

    root@raspberrypi:/home/pi# partprobe

Weryfikujemy wpisy w `fdisk` :

    root@raspberrypi:/home/pi# fdisk -l /dev/sda
    ...

    Device     Boot Start      End  Sectors  Size Id Type
    /dev/sda1        2048 15359999 15357952  7.3G 83 Linux

Tablica partycji jest na swoim miejscu, a przestrzeń MBR-GAP została wyczyszczona, tj. nadpisana
losowymi danymi.

Wgrywamy teraz plik klucz gdzieś w obrębie tej przestrzeni MBR-GAP, np. na offset 100:

    root@raspberrypi:/home/pi# dd if=/mnt/keyfile of=/dev/sda bs=512 seek=100

### Test keyfile wgranego na pendrive

Możemy przetestować czy przy pomocy tak wgranego na pendrive keyfile jesteśmy w stanie odszyfrować
kontener. Naturalnie do takiego testu potrzebny jest inny system, do którego podłączymy kartę SD
wyciągniętą z Raspberry Pi oraz też podepniemy pendrive z wypalonym kluczem. Tak czy inaczej,
poniżej znajduje się stosowne polecenie, którym taki kontener możemy odszyfrować:

    # dd if=/dev/sdb bs=512 skip=100 count=8 | \
       cryptsetup luksOpen /dev/mmcblk0p2 rpi_crypt --key-file=-

    # lsblk -o "NAME,SIZE,FSTYPE,TYPE,LABEL,MOUNTPOINT,UUID,PARTUUID" /dev/mmcblk0
    NAME           SIZE FSTYPE      TYPE  LABEL  MOUNTPOINT UUID                                 PARTUUID
    mmcblk0       28.8G             disk
    ├─mmcblk0p1    256M vfat        part  boot              B05C-D0C4                            0d03b705-01
    └─mmcblk0p2   28.6G crypto_LUKS part  rpi               0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be 0d03b705-02
      └─rpi_crypt 28.6G ext4        crypt rootfs            0ca2062b-142b-4826-bb74-d465ca89b554

I jak możemy zauważyć, kontener LUKS został otworzony bez wpisywania żadnego hasła.

## Nośnik USB dostępny zawsze pod tą samą nazwą

Problem z urządzeniami USB podczas startu systemu jest zawsze taki sam, tj. mogą one zostać wykryte
w innej kolejności niż miało to miejsce w przypadku poprzedniego rozruchu systemu. W taki sposób
nasz pendrive może zmienić ścieżkę z `/dev/sda` na `/dev/sdb` . W przypadku RPI ten problem raczej
nam nie zagraża, bo zbyt wielu urządzeń USB do niego nie będziemy podłączać. Niemniej jednak,
możemy napisać prostą regułę dla UDEV'a, która utworzy nam link do urządzenia blokowego w
określonym przez nas miejscu.

By taką regułę utworzyć, potrzebny nam będzie, np. numer seryjny pendrive. Ten numer seryjny możemy
uzyskać zaprzęgając do pracy narzędzie `udevadm` :

	root@raspberrypi:/home/pi# udevadm info --name /dev/sda
	...
	E: ID_SERIAL_SHORT=0019E06B9C8ABE41C7A2C3EC
	...

Mając numer seryjny, tworzymy plik `/etc/udev/rules.d/99-unlock-rpi.rules` . Dodajemy do tego pliku
poniższą zawartość:

    ACTION=="add", SUBSYSTEM=="usb", KERNEL=="sd?", \
     ENV{ID_SERIAL_SHORT}=="0019E06B9C8ABE41C7A2C3EC", \
     SYMLINK+="usbkey%n"

Przeładowujemy jeszcze konfigurację UDEV'a i testujemy regułę:

	root@raspberrypi:/home/pi# udevadm control --reload

	root@raspberrypi:/home/pi# udevadm test /devices/platform/scb/fd500000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0/usb1/1-1/1-1.1/1-1.1:1.0/host0/target0:0:0/0:0:0:0/block/sda
	...
	Reading rules file: /etc/udev/rules.d/99-unlock-rpi.rules
	...
	DEVLINKS=... /dev/usbkey ...
	...

Jak widać po powyższym logu, plik `/etc/udev/rules.d/99-unlock-rpi.rules` jest przetwarzany, a w
`DEVLINKS` widnieje link `/dev/usbkey` , który sobie określiliśmy.

## Skrypt identyfikujący pendrive jako klucz do kontenera

Potrzebny nam będzie prosty skrypt shell'owy, który wyciągnie z pendrive określone bity i poda je
do `cryptsetup` . Tworzymy zatem plik `/usr/sbin/unlock-rpi` i nadajemy mu prawa wykonywania:

    root@raspberrypi:/home/pi# touch /usr/sbin/unlock-rpi
    root@raspberrypi:/home/pi# chmod +x /usr/sbin/unlock-rpi

Do pliku wrzucamy poniższą zawartość:

    #!/bin/sh

    dd if=/dev/usbkey bs=512 skip=100 count=8

Gdzieniegdzie można się spotkać z przesłaniem wyjścia polecenia `dd` bezpośrednio do `cryptsetup`
za sprawą potoku (pipe), np. w poniższy sposób:

    dd if=/dev/usbkey bs=512 skip=100 count=8 | \
    cryptsetup luksOpen /dev/mmcblk0p2 rpi_crypt --key-file=-

Nie należy z tego rozwiązania korzystać, bo `cryptsetup` automatycznie podbiera wyjście ze skryptu,
który został określony w parametrze `keyscript` w pliku `/etc/crypttab` . Musimy zatem ten plik
dostosować:

    rpi_crypt  UUID=0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be   none  luks,initramfs,keyscript=/usr/sbin/unlock-rpi,keyslot=1

W stosunku do standardowej konfiguracji szyfrowania z wykorzystaniem hasła, w tej powyższej linijce
pojawił się dodatkowo parametr `keyscript=` , który w argumencie ma ścieżkę `/usr/sbin/unlock-rpi`
odpowiadającą za lokalizację naszego skryptu wyciągającego bity z pendrive. Wyjście tego skryptu
zostanie z automatu przesłane do `cryptsetup` . Mamy też określony parametr `keyslot=1` , który
wskazuje, że chcemy te bity z wyjścia skryptu porównać z tym co siedzi w slocie numer 1 w nagłówku
LUKS. Ma to na celu przyśpieszenie otwarcia zaszyfrowanego kontenera, bo domyślnie system by
przetworzył slot z numerkiem 0, dopiero później slot z numerkiem 1. Jeśli byśmy mieli zajętych
więcej slotów, to czas potrzebny na otworzenie kontenera uległby znacznemu wydłużeniu.

## Wstrzymanie rozruchu systemu do momentu podłączenia pendrive

To co chcemy uzyskać, to odszyfrowanie kontenera przy pomocy odpowiedniego urządzenia blokowego. Co
się jednak stanie w sytuacji, gdy w porcie USB nie będzie takiego urządzenia, albo będzie inne
urządzenie? Zapewne jakieś błędy na ekranie się pojawią ale sam system nie zostanie odszyfrowany.
Musimy zatem wypracować rozwiązanie, które zatrzyma start systemu w przypadku, gdy pendrive z
kluczem nie zostanie podłączony do portu USB. Możemy to zrobić przy wykorzystaniu skryptów
`initramfs-tools` .

Tworzymy zatem plik `/etc/initramfs-tools/scripts/local-top/delay-decrypt` i nadajemy mu prawa
wykonywania:

    root@raspberrypi:/home/pi# touch /etc/initramfs-tools/scripts/local-top/delay-decrypt
    root@raspberrypi:/home/pi# chmod +x /etc/initramfs-tools/scripts/local-top/delay-decrypt

Następnie wrzucamy do tego pliku poniższą zawartość:

    #!/bin/sh

    PREREQ="udev"
    prereqs()
    {
       echo "$PREREQ"
    }

    case $1 in
    prereqs)
       prereqs
       exit 0
       ;;
    esac

    # source for log_*_msg() functions, see LP: #272301
    . /scripts/functions

    # Default PATH differs between shells, and is not automatically exported
    # by klibc dash.  Make it consistent.
    export PATH=/sbin:/usr/sbin:/bin:/usr/bin

    DEVICE=/dev/usbkey
    if [ ! -L "$DEVICE" ]; then
        echo -e "\nWaiting for device..." >&2
        until [ -L "$DEVICE" ]; do
            sleep 1
        done
    fi

    exit 0

Zadaniem tego skryptu jest sprawdzenie czy link `/dev/usbkey` do pendrive został już utworzony. Ten
link zostanie utworzony przez UDEV dopiero w momencie, gdy podepniemy pendrive o określonym numerze
seryjnym do portu USB. Do tego czasu, system wyświetli komunikat `Waiting for device...` i zapętli
się uniemożliwiając dalszy start systemu, co będzie wyglądało mniej więcej tak:

![raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-wait](/img/2021/11/001.raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-wait.jpg#huge)

W interwale jednosekundowym, system będzie badał czy urządzenie zostało podłączone do portu USB.
Gdy taka sytuacja nastąpi, bity zawarte na pendrive zostaną automatycznie odczytane:

![raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-wait-success](/img/2021/11/002.raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-wait-success.jpg#huge)

## Regeneracja obrazu initramfs/initrd

Po dostosowaniu wszystkich tych powyższych plików, trzeba wygenerować na nowo obraz
initramfs/initrd przy pomocy `update-initramfs` :

    root@raspberrypi:/home/pi# update-initramfs -u -k $(uname -r)
    ln: failed to create hard link '/boot/initrd.img-5.10.63-v7l+.dpkg-bak' => '/boot/initrd.img-5.10.63-v7l+': Operation not permitted
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    Building v7l+ image, updating top initrd

### cryptsetup: ERROR: rpi_crypt: invalid value for 'keyscript' option, skipping

Może się zdarzyć tak, że podczas generowania obrazu initramfs/initrd zostanie nam wyrzucony
poniższy błąd:

    root@raspberrypi:/home/pi# update-initramfs -u -k $(uname -r)
    ln: failed to create hard link '/boot/initrd.img-5.10.63-v7l+.dpkg-bak' => '/boot/initrd.img-5.10.63-v7l+': Operation not permitted
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    cryptsetup: ERROR: rpi_crypt: invalid value for 'keyscript' option, skipping
    cryptsetup: WARNING: The initramfs image may not contain cryptsetup binaries
        nor crypto modules. If that's on purpose, you may want to uninstall the
        'cryptsetup-initramfs' package in order to disable the cryptsetup initramfs
        integration and avoid this warning.
    Building v7l+ image, updating top initrd

Efektem tego błędu będzie brak binarki `cryptsetup` w obrazie initramfs/initrd, przez co nie damy
rady odszyfrować kontenera LUKS podczas startu systemu.

Taka sytuacja ma miejsce za sprawą domyślnego przejęcia kontroli nad plikiem `/etc/crypttab` przez
systemd, a konkretnie przez [systemd cryptsetup generator][3]. Ten mechanizm jednak [nie wspiera
całej masy opcji][4], które plik `/etc/crypttab` obsługuje w standardzie, co można poznać po
poniższych komunikatach w logu systemowym:

    raspberrypi systemd-cryptsetup[533]: Encountered unknown /etc/crypttab option 'keyscript=/usr/sbin/unlock-rpi', ignoring.
    raspberrypi systemd-cryptsetup[533]: Encountered unknown /etc/crypttab option 'initramfs', ignoring.
    raspberrypi systemd-cryptsetup[533]: Encountered unknown /etc/crypttab option 'keyslot=1', ignoring.

By jakoś zaradzić temu problemowi, trzeba wyłączyć generator cryptsetup od systemd. Możemy to zrobić
przez dopisanie parametru `luks.crypttab=no` w kernel cmdline w pliku `/boot/cmdline.txt` :

## Test pendrive jako klucza do kontenera LUKS

Po wygenerowaniu obrazu initramfs/initrd uruchamiamy system naszego Raspberry Pi ponownie.
Podłączamy też pendrive do portu USB i obserwujemy co się dzieje na ekranie monitora. W moim
przypadku system bez większego problemu poradził sobie z wydobyciem określonych bitów z pendrive i
podał je do `cryptsetup` , co skutkowało odszyfrowaniem kontenera LUKS. Cały proces został
zobrazowany na poniższej fotce:

![raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-boot-system](/img/2021/11/003.raspberry-pi-rpi-cryptsetup-luks-pendrive-unlock-keyfile-boot-system.jpg#huge)

## Kefile jako plik w obrębie systemu plików

We wstępie do niniejszego artykułu wspomniałem, że istnieją dwa zbliżone do siebie rozwiązania
zakładające wykorzystanie nośnika USB w roli klucza do zaszyfrowanego kontenera LUKS. Jedno z tych
rozwiązań zostało opisane wyżej ale jakby nie patrzeć, trochę się trzeba było nad nim napracować.
Zapewne wielu użytkowników nie potrzebuje tak wyrafinowanego mechanizmu otwierającego kontener LUKS
i można by skorzystać z prostszej opcji, tj. umieszczenia pliku klucza w obrębie systemu plików na
jednej z partycji pendrive.

By ten poniżej opisany sposób zadziałał potrzebny nam będzie pendrive z co najmniej jedną partycją
sformatowaną systemem plików ext4. Taką partycję montujemy sobie gdzieś, np. w `/media/pendrive/` ,
po czym generujemy plik klucz w sposób opisany wyżej. Ten plik klucz też trzeba dodać do nagłówka
LUKS, co też zostało opisane wyżej. Następnie trzeba przypisać etykietę do systemu plików przy
pomocy `e2label` w poniższy sposób (link przez UDEV też się nada ale sposób z etykietą jest dużo
prostszy):

	root@raspberrypi:/home/pi# e2label /dev/sda1 pendrak

Ostatnim krokiem jest dostosowanie opcji dla kontenera w pliku `/etc/crypttab` . Dokładnie chodzi o
trzecią kolumnę, w której standardowo rezyduje fraza `none` . Trzeba ją przepisać do postaci
`/sciezka/do/urzadzenia:/sciezka/do/pliku` , przykładowo:

    rpi_crypt  UUID=0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be   /dev/disk/by-label/pendrak:/keyfile  luks,initramfs,keyscript=passdev,keyslot=1

W tym przypadku partycja pendrive ma etykietę `pendrak` i jest dostępna pod ścieżką
`/dev/disk/by-label/pendrak` , a klucz jest dostępny w pliku `keyfile` w głównym katalogu na
pendrive. By całość zadziałała, musimy zaprzęgnąć do pracy skrypt `passdev` , który wskazujemy w
parametrze `keyscript=` .

Po edycji pliku `/etc/crypttab` trzeba na nowo wygenerować obraz initramfs/initrd przy pomocy
`update-initramfs` . To w zasadzie wszystko co trzeba zrobić, aby wdrożyć keyfile w postaci pliku
umieszczonego w obrębie systemu plików przykładowej partycji na pendrive. Przy pomocy takiego pliku
klucza można odszyfrować kontener LUKS bez wpisywania jakiegokolwiek hasła.

## Podsumowanie

Takie rozwiązanie uzależnienia startu systemu Raspberry Pi od faktu podłączenia konkretnego
urządzenia pamięci masowej do portu USB niewątpliwie jest wielce użyteczne. Odpada nam w ten sposób
potrzeba podłączania monitora i/lub klawiatury, przez co cały proces uruchamiania systemu przebiega
o wiele szybciej i sprawniej. Trzeba jednak zdawać sobie sprawę, że jeśli my jesteśmy w stanie bez
większego problemu odszyfrować RPI przy pomocy takiego pendrive, to każda inna osoba również będzie
w stanie to zrobić, jak tylko to urządzenie wpadnie w jej łapki. Dlatego trzeba się zatroszczyć o
fizyczny dostęp do pendrive i trzymać go w bezpiecznym miejscu. Opisane wyżej dwa rozwiązania
różnią się trochę ale efekt dają mniej więcej ten sam. Niemniej jednak, sposób z wypaleniem keyfile
w MBR-GAP może ździebko utrudnić wyciągnięcie klucza z takiego pendrive (na działającym systemie) w
stosunku do przechowywania tego klucza w regularnym pliku, zwłaszcza w sytuacji, gdy ta pamięć
masowa będzie używana do czegoś więcej niż tylko do odszyfrowania systemu naszej maliny.


[1]: /post/jak-zaszyfrowac-raspberry-pi-raspios-raspbian-luks/
[2]: /post/keyfile-trzymany-w-glebokim-ukryciu/
[3]: https://www.freedesktop.org/software/systemd/man/systemd-cryptsetup-generator.html
[4]: https://manpages.debian.org/unstable/cryptsetup/crypttab.5.en.html
