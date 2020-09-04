---
author: Morfik
categories:
- Linux
date: "2015-06-16T20:16:26Z"
date_gmt: 2015-06-16 18:16:26 +0200
published: true
status: publish
tags:
- mbr
- hdd
- ssd
- gpt
- partycja
title: Konwersja tablicy partycji GPT na MS-DOS
---

W poprzednim wpisie poruszyłem temat [konwersji tablicy partycji MS-DOS (MBR) na
GPT]({{< baseurl >}}/post/konwersja-tablicy-partycji-ms-dos-na-gpt/). Jak można było zauważyć, ten
proces nie był skomplikowany i niemalże automatyczny. Nie towarzyszyło mu także zjawisko utraty
jakichkolwiek danych, jedynie w przypadku posiadania systemu operacyjnego, trzeba było
przeinstalować bootloader. Nasuwa się zatem pytanie, czy również w tak prosty sposób można
przerobić tablicę partycji GPT na MS-DOS? Jest to wykonalne z tym, że trzeba odpowiednio
przygotować sobie do tego celu dysk.

<!--more-->
## Powrót do tablicy partycji MS-DOS

Konwertowanie partycji z GPT na MBR jest trochę trudniejsze. Oczywiście zależy to od samej struktury
dysku, tj. ile partycji mamy do dyspozycji. Jeśli jest ich mniej niż 4, to praktycznie nie ma
żadnego problemu z procesem konwersji. Natomiast jeśli jest ich więcej, to w grę wchodzi już
partycja rozszerzona i jej [sektory
EBR]({{< baseurl >}}/post/mbr-ebr-i-tablica-partycji-dysku-twardego/) , a te mają przecie 512
bajtów. Dodatkowo by wyrównać partycje do 1MiB trzeba również dodać 2047 sektorów wolnego miejsca.

Pamiętacie te dziury w strukturze partycji po przejściu na tablicę GPT? Niżej przypomnienie:

![]({{< baseurl >}}/img/2015/06/1.konwersja-ms-dos-gpt-layout-dysk.png#huge)

Jeśli ich nie mamy na dysku, prawdopodobnie nie da rady przenieść niektórych partycji z GPT na
MS-DOS. Natomiast, zawsze można je stworzyć kurcząc odpowiednie partycje. Te dziury między
partycjami muszą mieć 2048 sektorów. Odpalamy narzędzie `gdisk` i wpisujemy kolejno `r` , `g` oraz
`p` :

                                                       Can Be   Can Be
    Number  Boot  Start Sector   End Sector   Status   Logical  Primary   Code
       1                264192     78125055   primary              Y      0x83
       2              78125056     99096575   primary              Y      0x83
       3                  2048       264191   primary     Y        Y      0xEF
       4              99096576    128460799   primary              Y      0x83
       6             128460800    156299263   omitted                     0x83

Została nam wyświetlona informacja na temat tego jaką rolę może pełnić dana partycja. W tym
przypadku mam 5 partycji. Z tym, że jedna przeznaczona na kod bootloadera. Jako, że mam taki a nie
inny układ partycji, to muszę się pozbyć jednej z nich, by wszystkie pozostałe można było uwzględnić
w tablicy partycji MS-DOS.

Po usunięciu tej i tak zbędnej partycji, wszystkie pozostałe są uwzględniane w nowej tablicy
partycji:

                                                       Can Be   Can Be
    Number  Boot  Start Sector   End Sector   Status   Logical  Primary   Code
       1                264192     78125055   primary     Y        Y      0x83
       2              78125056     99096575   primary              Y      0x83
       4              99096576    128460799   primary              Y      0x83
       6             128460800    156299263   primary              Y      0x83

Jeśli udało nam się uzyskać układ partycji taki jak chcieliśmy, to wpisujemy `w` :

    MBR command (? for help): w

    Converted 4 partitions. Finalize and exit? (Y/N): y
    GPT data structures destroyed! You may now partition the disk using fdisk or
    other utilities.

W ramach sprawdzenia czy aby konwersja na MS-DOS zakończyła się powodzeniem, odpalamy `parted` :

    (parted) print free
    Model: WDC WD80 0JB-00JJA0 (scsi)
    Disk /dev/sdc: 156299375s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start       End         Size       Type     File system  Flags
            63s         264191s     264129s             Free Space
     1      264192s     78125055s   77860864s  primary  ext4         boot
     2      78125056s   99096575s   20971520s  primary  ext4
     3      99096576s   128460799s  29364224s  primary  ext4
     4      128460800s  156299263s  27838464s  primary  ext4
            156299264s  156299374s  111s                Free Space

Jedyna różnica to wolne miejsce z początku dysku powstałe w wyniku usunięcia partycji wymaganej
przez bootloader grub. Cała reszta jest dokładnie bez zmian, no może zamiast rozszerzonej partycji
mamy wszystkie podstawowe.

W przypadku gdy posiadamy na tym dysku jakiś system operacyjny, trzeba pomocy [systemu
live]({{< baseurl >}}/post/wlasny-system-live-i-tworzenie-go-od-podstaw/) przeinstalować
bootloader grub (lub extlinux).
