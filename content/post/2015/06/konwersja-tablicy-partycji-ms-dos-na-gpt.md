---
author: Morfik
categories:
- Linux
date: "2015-06-16T19:26:00Z"
date_gmt: 2015-06-16 17:26:00 +0200
published: true
status: publish
tags:
- mbr
- hdd/ssd
- gpt
title: Konwersja tablicy partycji MS-DOS na GPT
---

Od jakiegoś czasu nosiłem się z zamiarem utworzenia na swoim dysku tablicy partycji GPT. Problem w
tym, że tam jest wgranych sporo cennych danych i nie mam gdzie ich przenieść. Jedyne co mi wpadło do
głowy to konwersja tablicy MS-DOS ([zwanej też
MBR](https://superuser.com/questions/700770/mbr-equals-msdos-for-gparted)) na GPT. Nie żebym jakoś
tego potrzebował ale tak z ciekawości chciałem zobaczyć czy da się to zrobić w sposób łatwy i
bezproblemowy. Z tego co wyczytałem na necie, to taka konwersja nie powinna sprawić problemów,
zarówno przy przechodzeniu z MS-DOS na GPT jak i odwrotnie, choć w tym drugim przypadku trzeba się
trochę bardziej wysilić.

<!--more-->
## Struktura GPT

Do celów testowych wykorzystam mój stary dysk, na którym to są obecne 4 partycje, w tym jedna
rozszerzona, na której to zostały ulokowane dwa dyski logiczne:

![]({{< baseurl >}}/img/2015/06/3.dysk-po-konwersji-tablicy-ms-dos-na-gpt.png)

Tak z kolei wygląda dysk widziany oczami `parted` , z tym, że z uwzględnieniem wolnych przestrzeni:

    (parted) print free
    Model: WDC WD80 0JB-00JJA0 (scsi)
    Disk /dev/sdb: 156299375s
    Sector size (logical/physical): 512B/512B
    Partition Table: msdos
    Disk Flags:

    Number  Start       End         Size       Type      File system  Flags
            63s         2047s       1985s                Free Space
     1      2048s       78125055s   78123008s  primary   ext4         boot
     2      78125056s   99096575s   20971520s  primary   ext4
     3      99096576s   156299263s  57202688s  extended
     5      99098624s   128458751s  29360128s  logical   ext4
     6      128460800s  156299263s  27838464s  logical   ext4
            156299264s  156299374s  111s                 Free Space

By przeprowadzić bezproblemowo konwersję tablicy partycji MS-DOS na GPT, trzeba mieć troszeczkę
wolnego miejsca na początku i na końcu dysku. Ile? Struktura GPT wygląda tak jak na rysunku poniżej
(zaczerpnięty z
[wiki](https://en.wikipedia.org/wiki/GUID_Partition_Table)):

[![2.gpt-schemat]({{< baseurl >}}/img/2015/06/2.gpt-schemat-745x1024.png)]({{< baseurl >}}/img/2015/06/2.gpt-schemat.png)

Mamy tam zatem po jednym sektorze na MBR oraz nagłówek GPT, oraz 32 sektory na tablicę partycji.
Łącznie daje to 34 sektory 512 bajtowe, co przekłada się na `17408` bajtów wolnego miejsca na
początku dysku. Jak możemy zaobserwować, struktura tablicy partycji GPT ma jeszcze backup swojej
zawartości, który jest zlokalizowany na końcu dysku i w jego skład wchodzi kopia głównej tablicy
partycji (32 sektory) oraz kopia nagłówka (1 sektor). Łącznie 33 sektory co daje `16896` bajtów
potrzebnego miejsca na końcu dysku.

Obecnie większość narzędzi tworzących partycje na dysku ustawia równanie do 1MiB co przekłada się na
2048 wolnych sektorów na początku dysku, więc tutaj nie powinno być problemów ze spełnieniem wymagań
stawianych przez tablicę partycji GPT. W moim przypadku, jak widać powyżej, zostało 111 sektorów
wolnego miejsca na końcu dysku, co daje `56832` bajtów. Także u mnie wszystko jest w porządku. W
przypadku jednak gdyby jakaś partycja wypełniła dysk do samego końca, trzeba będzie ją skrócić.

## Konwertowanie tablicy partycji na GPT

By dokonać konwersji tablicy partycji, odpalamy narzędzie `gdisk` podając w argumencie ścieżkę do
dysku:

    # gdisk /dev/sdb
    GPT fdisk (gdisk) version 1.0.0

    Partition table scan:
      MBR: MBR only
      BSD: not present
      APM: not present
      GPT: not present

    ***************************************************************
    Found invalid GPT and valid MBR; converting MBR to GPT format
    in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
    typing 'q' if you don't want to convert your MBR partitions
    to GPT format!
    ***************************************************************

Jak widać wyżej, `gdisk` wykrył, że ma do czynienia z tablicą partycji MS-DOS oraz poinformował nas,
że dokonał jej konwersji na GPT, z tym, że zmiany póki co są dokonane jedynie w pamięci operacyjnej.
Jeśli chcemy je wprowadzić w życie, wpisujemy `w`:

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): y
    OK; writing new GUID partition table (GPT) to /dev/sdb.
    The operation has completed successfully.

Sprawdzamy, czy faktycznie tablica partycji uległa zmianie:

    # gdisk /dev/sdb
    GPT fdisk (gdisk) version 1.0.0

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.

Czyli proces bez problemu się zakończył. Rzućmy jeszcze okiem na to jak wygląda struktura partycji w
`gparted` :

![]({{< baseurl >}}/img/2015/06/1.konwersja-ms-dos-gpt-layout-dysk.png)

Są widoczne jakieś dwie dziury. Może `parted` nam coś więcej podpowie:

    (parted) print free
    Model: WDC WD80 0JB-00JJA0 (scsi)
    Disk /dev/sdb: 156299375s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags:

    Number  Start       End         Size       File system  Name              Flags
            34s         2047s       2014s      Free Space
     1      2048s       78125055s   78123008s  ext4         Linux filesystem
     2      78125056s   99096575s   20971520s  ext4         Linux filesystem
            99096576s   99098623s   2048s      Free Space
     5      99098624s   128458751s  29360128s  ext4         Linux filesystem
            128458752s  128460799s  2048s      Free Space
     6      128460800s  156299263s  27838464s  ext4         Linux filesystem
            156299264s  156299341s  78s        Free Space

Te dziury są w miejscach gdzie zaczynały się dyski logiczne partycji rozszerzonej. Jest to wynikiem
równania do 1MiB (2048 sektorów). Można te przestrzenie pokasować przez [rozszerzenie partycji i
rozciągnięcie systemu plików, w tym przypadku
EXT4]({{< baseurl >}}/post/zmiana-rozmiaru-partycji-ext4/). Można też zauważyć, że ilość wolnego
miejsca na początku i końcu dysku uległa zmianie. Trzeba jednak pamiętać, że po zmianie partycji
zostaną wygenerowane nowe numerki UUID, co pociąga za sobą edycję i dostosowanie pliku `/etc/fstab`
. W przeciwnym razie system się nie odpali. Trzeba także wyznaczyć osobną partycję na kod
bootloadera o rozmiarze minimum 1MiB, najlepiej dać tam 128MiB. Ważne jest by ta partycja miała
ustawioną flagę `bios_grub` . Ja wyciąłem ten kawałek z pierwszej partycji. Cały układ powinien
zatem wyglądać tak jak na obrazku poniżej:

![]({{< baseurl >}}/img/2015/06/4.odpowiedni-uklad-partycji-gpt.png)

Jeśli na dysku mieliśmy system operacyjny, to trzeba także przeinstalować bootloader przy pomocy
[środowiska chroot]({{< baseurl >}}/post/przygotowanie-srodowiska-chroot-do-pracy/), najlepiej z
poziomu [systemu live]({{< baseurl >}}/post/wlasny-system-live-i-tworzenie-go-od-podstaw/) .
