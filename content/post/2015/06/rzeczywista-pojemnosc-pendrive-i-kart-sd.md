---
author: Morfik
categories:
- Hardware
date: "2015-06-14T09:08:09Z"
date_gmt: 2015-06-14 07:08:09 +0200
published: true
status: publish
tags:
- karta-sd
- pendrive
- flash
GHissueID: 132
title: Rzeczywista pojemność pendrive i kart SD
---

Parę dni temu [jednemu z użytkowników forum DUG](https://forum.dug.net.pl/viewtopic.php?id=26668)
przytrafiła się niezbyt miła sytuacja. Rozchodzi się o to iż zakupił on kartę SD i, jak sprzedawca
zapewniał, miała mieć pojemność 128 GiB. Wszyscy wiemy, że sprzedawcy nieco zawyżają te numerki na
opakowaniach, bo operują na potęgach o podstawie 10, a nie 2, i tak ze 100 GB robi się zaraz 93 GiB.
Do tego oczywiście jeszcze dochodzi rezerwacja miejsca na potrzebę obsługi systemu plików. Niemniej
jednak, w tym przypadku, różnica była trochę większa i tutaj mamy do czynienia z czymś co się nazywa
[fake flash](http://www.ebay.com/gds/All-About-Fake-Flash-Drives-2013-/10000000177553258/g.html).

Jest taka żelazna zasada, by po zakupie jakiegoś sprzętu, sprawdzić go czy aby działa jak należy i
czy jest z nim wszystko w porządku. Jest to wręcz obowiązek przy zakupie pamięci opartych o
technologię flash, bez znaczenia z jakiego to źródła by one nie pochodziły.

<!--more-->
## Weryfikacja parametrów karty SD / pendrive

Jeśli chodzi o karty SD czy pendrive, pewne osobniki mogą tak zaprogramować te urządzonka, by
pokazywały większą pojemność niż w rzeczywistości one posiadają. Oczywiście nie musi to być świadome
działanie sprzedawcy. Może on zwyczajnie nawet o tym nie wiedzieć. Nie uchroni nas też kupowanie
markowego sprzętu, bo opakowanie i wszystkie inne efekty wizualne można bez problemu podrobić tak,
by konkretne pamięci czy pendrive łudząco przypominały te oryginalne odpowiedniki. Po zakupie
takiego urządzenia, człowiek używa go jak gdyby nigdy nic, do momentu aż wyjdzie poza granicę
faktycznej pojemności flash'a. Teoretycznie niby kopiowanie do/z pena/karty jest w porządku ale, gdy
chcemy, np. odtworzyć film czy też podejrzeć zdjęcia, to okazuje się, że część plików jest
uszkodzonych, tj. zawiera same zera. W pewnych przypadkach, nowe dane mogą zacząć nadpisywać te
upchnięte już na pamięci flash niszcząc je przy tym zupełnie.

Jednym z zabezpieczeń jakie może nas uchronić przed podróbkami jest weryfikacja parametrów oraz
numeru seryjnego kupowanego modelu. Jeśli urządzenie nie posiada numeru lub mamy przed nosem
pendrive o rozmiarze 2 TiB ale ten model nigdy takiej pojemności się nie doczekał, to najlepiej
zostawić go w spokoju. Zawsze też możemy poprosić sprzedawcę o 95% rabat. Jeśli chodzi o same karty
SD, to po właściwościach standardu możemy stwierdzić czy dana karta to fake. [Na wiki można
wyczytać](https://pl.wikipedia.org/wiki/Secure_Digital), iż stowarzyszenie SD Card Association
wyznacza określone standardy dla kart SD. Jeśli chcemy kupić kartę 128 GB i sprzedawca reklamuje ją
jako kartę `SDHC` , to wiemy na 100%, że to podróba. Karty `SDHC` nie mogą mieć więcej niż 32 GB,
tzn. może i mogą ale z racji ograniczeń licencyjnych, żadna większa firma nie produkuje takich.
Oczywiście karty w dalszym ciągu mogą mieć 64 GB - 2 TB ale to reguluje standard `SDXC` . Jedna
literka robi różnicę. Podróbki z reguły nie zważają na standardy, temu łatwo je odróżnić, a jeśli
mamy z tym problemy, to najlepiej zajrzeć na stronę producenta karty/pendrive i upewnić się czy ten
model o takich gabarytach w ogóle jest przez niego produkowany.

Poniżej jest fotka tej trefnej karty SD:

![trefna-karta-sd](/img/2015/06/1.trefna-karta-sd.png#small)

Jak widać, nadruk jest sprzeczny sam ze sobą. Karta `SDHC` i do tego 128 GB, czyli już na pierwszy
rzut oka wiadomo, że coś jest nie tak.

## Test karty SD

Nasuwa się zatem pytanie. Jak to się dzieje, że taka karta/pendrive może zgłaszać większą pojemność
niż faktycznie posiada i skąd taki sprzęt się bierze? Ogólnie rzecz biorąc, te wszystkie fake flash,
to materiał produkcyjny oryginalnych fabryk i są to głównie urządzenia o mniejszej pojemności.
Reszta przypadków tyczy się uszkodzonego sprzętu, który został odrzucony w procesie produkcyjnym i
cały ten złom został przy tym odsprzedany komuś, kto chciał to kupić. Potem ten ktoś przeprogramował
chip kontrolera odpowiedzialnego za zarządzenie pamięcią przez wgranie nowego firmware, który
zawiera inne ustawienia dla sterownika i ten myśli, że faktycznie ma do dyspozycji więcej pamięci.
Potem taki pendrive/kara jest formatowany w oparciu o tę zawyżoną wartość i tak mamy widzianą w
systemie kartę o większej pojemności. Do stworzenia partycji potrzebny nam jest jedynie pierwszy
sektor, bo tam jest tablica partycji. System plików z kolei tworzy metadane oraz superblok i jego
kopie w wydzielonym już obszarze pamięci i póki nie wychodzi on poza fizyczne ramy pamięci flash,
nic się nie będzie dziać. W skrócie wszystko gra, gdy korzystamy z systemu plików FAT i jest tylko
jedna partycja. Nawet skanowanie powierzchni dysku w poszukiwaniu bad bloków nie wykryje oszustwa,
bo te fake flash nie zwracają błędów odczytu nawet, gdy system próbuje czytać z nieistniejącego
sektora.

Poniżej jest przeprowadzony test sprawdzający faktyczną ilość dostępnej pamięci flash na podesłanej
mi przez użytkownika davidoski karcie SD. By taki test przeprowadzić, potrzebne nam będą dwa
narzędzia: `f3write` i `f3read`. Oba z nich znajdują się w debianie w pakiecie `f3` . Strona
projektu zaś jest dostępna [pod tym adresem](http://oss.digirati.com.br/f3/).

Podłączamy zatem nowo zakupioną kartę/pena do naszego PC:

    usb 2-1.1: new high-speed USB device number 17 using ehci-pci
    usb 2-1.1: New USB device found, idVendor=05e3, idProduct=0723
    usb 2-1.1: New USB device strings: Mfr=3, Product=4, SerialNumber=2
    usb 2-1.1: Product: USB Storage
    usb 2-1.1: Manufacturer: Generic
    usb 2-1.1: SerialNumber: 000000009451
    usb-storage 2-1.1:1.0: USB Mass Storage device detected
    usb-storage 2-1.1:1.0: Quirks match for vid 05e3 pid 0723: 8000
    scsi5 : usb-storage 2-1.1:1.0
    mtp-probe: checking bus 2, device 17: "/sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.1"
    mtp-probe: bus: 2, device: 17 was not an MTP device
    kernel: [  397.527000] scsi 5:0:0:0: Direct-Access     Generic  STORAGE DEVICE   9451 PQ: 0 ANSI: 0
    kernel: [  397.527590] sd 5:0:0:0: Attached scsi generic sg2 type 0
    kernel: [  398.181613] sd 5:0:0:0: [sdb] 262144000 512-byte logical blocks: (134 GB/125 GiB)
    kernel: [  398.182981] sd 5:0:0:0: [sdb] Write Protect is off
    kernel: [  398.182988] sd 5:0:0:0: [sdb] Mode Sense: 03 00 00 00
    kernel: [  398.184354] sd 5:0:0:0: [sdb] No Caching mode page found
    kernel: [  398.184360] sd 5:0:0:0: [sdb] Assuming drive cache: write through
    kernel: [  398.192694]  sdb: sdb1
    kernel: [  398.196814] sd 5:0:0:0: [sdb] Attached SCSI removable disk

Jak widać wyżej karta została rozpoznana przez system, a pojemność wykryta to 134 GB. Spróbujmy
zmienić system plików tego urządzenia na EXT4. U mnie udało się bez większego problemu ale przy
próbie zamontowania zasobu dostaję taki komunikat:

    kernel: [  655.601584] JBD2: no valid journal superblock found
    kernel: [  655.601593] EXT4-fs (sdb1): error loading journal

Wróciłem zatem do fabrycznego FAT32, po czym podmontowałem kartę w systemie i zacząłem wgrywać pliki
via `f3write` . Narzędzie `f3write` zapisuje tylko wolną przestrzeń na urządzeniu, zatem przydałoby
się wyczyścić kartę zanim przystąpimy do jej sprawdzania.

    # f3write /media/morfik/rootfs/
    Free space: 124.97 GB
    Creating file 1.h2w ... OK!
    Creating file 2.h2w ... OK!
    ...
    Creating file 124.h2w ... OK!
    Creating file 125.h2w ... OK!
    Free space: 0.00 Byte
    Average writing speed: 8.26 MB/s

Trwało to co prawda parę godzin ale zapis całych 125 GiB zakończył się powodzeniem. Po
przemontowaniu karty, przy listowaniu zawartości powinniśmy zobaczyć wszystkie wgrane pliki ale sam
fakt, że system zapisał 125 GiB na karcie SD, nic jeszcze nie oznacza. Musimy zweryfikować czy aby
na pewno da radę odczytać te pliki. Odpalamy zatem `f3read` :

    # f3read /media/morfik/rootfs
                      SECTORS      ok/corrupted/changed/overwritten
    Validating file 1.h2w ... 2097152/        0/      0/      0
    Validating file 2.h2w ... 2097152/        0/      0/      0
    Validating file 3.h2w ... 2097152/        0/      0/      0
    Validating file 4.h2w ... 2097152/        0/      0/      0
    Validating file 5.h2w ... 2097152/        0/      0/      0
    Validating file 6.h2w ... 2097152/        0/      0/      0
    Validating file 7.h2w ... 2097152/        0/      0/      0
    Validating file 8.h2w ... 1182951/   914201/      0/      0
    Validating file 9.h2w ...       0/  2097152/      0/      0
    Validating file 10.h2w ...       0/  2097152/      0/      0
    ...

Jak widać wyżej, 7 plików ze 125 wcześniej utworzonych zostało odczytanych bez problemów. Część
ósmego pliku rezyduje poza obszarem pamięci flash, tj. niby plik jest widoczny ale nie ma sektorów,
z których mógłby on zostać odczytany, dlatego zaczynają pojawiać się błędy. To samo jest już z
kolejnymi plikami i żaden następny sektor nie może zostać odczytany. Zatem faktyczna wielkość tej
karty SD to 8 GB, a konkretnie (7*2097152*512)+(1182951*512)=8121863680 bajtów, lub
8121863680/512=15863015 sektorów.

## Czy fake flash nadaje się do użytku

Tę kartę można używać, z tym, że trzeba utworzyć na niej odpowiednią partycję, tj. taką, której
ostatni sektor będzie rezydował na pozycji 15863015-1=15863014 . Chodzi o to, że że sektory na dysku
są numerowane od 0, a nie od 1. Odpalamy zatem `fdisk` , usuwamy starą partycję i tworzymy na niej
miejsce nową:

    # fdisk /dev/sdb

    Welcome to fdisk (util-linux 2.25.2).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): p
    Disk /dev/sdb: 125 GiB, 134217728000 bytes, 262144000 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x00055cd9

    Device     Boot Start       End   Sectors  Size Id Type
    /dev/sdb1        2048 262143999 262141952  125G  b W95 FAT32

    Command (m for help): d
    Selected partition 1
    Partition 1 has been deleted.

    Command (m for help): n
    Partition type
       p   primary (0 primary, 0 extended, 4 free)
       e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (2048-262143999, default 2048): 2048
    Last sector, +sectors or +size{K,M,G,T,P} (2048-262143999, default 262143999): 15863014

    Created a new partition 1 of type 'Linux' and of size 7.6 GiB.

Sprawdzamy czy wszystko się zgadza:

    Command (m for help): p
    Disk /dev/sdb: 125 GiB, 134217728000 bytes, 262144000 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x00055cd9

    Device     Boot Start      End  Sectors  Size Id Type
    /dev/sdb1        2048 15863014 15860966  7.6G 83 Linux

Rozmiar partycji jest mniejszy o te początkowe 2048 sektory (równanie do 1 MiB). Zapisujemy i
przeładowujemy info o karcie:

    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.

    # partprobe

Formatujemy partycję z wykorzystaniem systemu plików EXT4 i montujemy zasób:

    # mkfs.ext4 -m 0 -L karta /dev/sdb1
    # mount /dev/sdb1 /mnt/

Nie ma błędów, zatem testujemy zapis/odczyt danych:

    # f3write /mnt/
    Free space: 7.30 GB
    Creating file 1.h2w ... OK!
    Creating file 2.h2w ... OK!
    Creating file 3.h2w ... OK!
    Creating file 4.h2w ... OK!
    Creating file 5.h2w ... OK!
    Creating file 6.h2w ... OK!
    Creating file 7.h2w ... OK!
    Creating file 8.h2w ... OK!
    Free space: 16.01 MB
    Average writing speed: 5.72 MB/s

    # f3read /mnt/
                      SECTORS      ok/corrupted/changed/overwritten
    Validating file 1.h2w ... 2097152/        0/      0/      0
    Validating file 2.h2w ... 2097152/        0/      0/      0
    Validating file 3.h2w ... 2097152/        0/      0/      0
    Validating file 4.h2w ... 2097152/        0/      0/      0
    Validating file 5.h2w ... 2097152/        0/      0/      0
    Validating file 6.h2w ... 2097152/        0/      0/      0
    Validating file 7.h2w ... 2097152/        0/      0/      0
    Validating file 8.h2w ...  601992/        0/      0/      0

      Data OK: 7.29 GB (15282056 sectors)
    Data LOST: 0.00 Byte (0 sectors)
                   Corrupted: 0.00 Byte (0 sectors)
            Slightly changed: 0.00 Byte (0 sectors)
                 Overwritten: 0.00 Byte (0 sectors)
    Average reading speed: 16.19 MB/s

I jak widać, partycja nadaje się do odczytu/zapisu danych, z tym, że zamiast 125 GiB jest nieco
ponad 7 GiB. Narzędzie `gparted` też bez problemu wykrywa tę partycję:

![okrojona-karta-sd](/img/2015/06/2.okrojona-karta-sd.png#huge)
