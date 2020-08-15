---
author: Morfik
categories:
- Linux
date: "2015-09-06T12:45:04Z"
date_gmt: 2015-09-06 10:45:04 +0200
published: true
status: publish
tags:
- kernel
- initrd
- initramfs
title: Zawartość obrazu z modułami kernela (initrd)
---

Podczas instalowania jądra operacyjnego w jakiejś dystrybucji linux'a, w katalogu `/boot/` jest
tworzonych kilka plików. Mamy tam między innymi
[initrd.img](https://www.ibm.com/developerworks/linux/library/l-initrd/index.html) i jest to obraz
posiadający swój własny system plików, który jest ładowany do pamięci RAM w fazie boot (via
bootloader). W tym obrazie znajdują się moduły i narzędzia, przy pomocy których to główny system
plików naszego linux'a może zostać zamontowany. Czasem jednak potrzebujemy zajrzeć wgłąb tego
obrazu, a to nie jest znowu taka prosta sprawa.

<!--more-->
## Struktura initrd/initramfs

Na potrzeby tego artykułu, posłużyłem się obrazem kernela 4.1.0-2-amd64, który obecnie jest dostępny
w gałęzi testowej debian'a. Na początek sprawdźmy jakie informacje o obrazie zwróci nam system. W
tym celu korzystamy z narzędzia `file` wskazując w jego argumencie ścieżkę do pliku z obrazem:

    # file /boot/initrd.img-4.1.0-2-amd64
    /boot/initrd.img-4.1.0-2-amd64: gzip compressed data, last modified: Sun Sep  6 10:05:53 2015, from Unix

Zatem plik jest skompresowany i musimy go pierw wypakować. Robimy to w poniższy sposób:

    # gzip -dc ./initrd.img-4.1.0-2-amd64 > initrd

Sprawdzamy informacje jakie zwróci nam `file` w stosunku do utworzonego w ten sposób pliku:

    # file initrd
    initrd: ASCII cpio archive (SVR4 with no CRC)

Zatem jest to kolejne archiwum, tym razem cpio. Je również musimy wypakować:

    # cpio -i < initrd
    149091 blocks

Oczywiście możemy pójść na skróty i skorzystać z tego poniższego polecenia, a efekt będzie dokładnie
taki sam:

    # gzip -dc ./initrd.img-4.1.0-2-amd64 | cpio -i
    149091 blocks

Niezależnie od wybranego sposobu, w folderze roboczym powinniśmy mieć szereg plików i katalogów. To
właśnie jest zawartość obrazu, którą możemy przejrzeć w celu inspekcji odpowiednich plików.

## Wprowadzanie zmian w obrazie initrd

Możemy także [wprowadzić zmiany](https://openvz.org/Modifying_initrd_image) do tego systemu plików w
celach testowych i zbudować obraz na nowo przy pomocy poniższego polecenia:

    # find ./ | cpio -H newc -o < ../initrd
    149566 blocks

Następnie kompresujemy jeszcze sam obraz przy pomocy `gzip`

    # gzip ../initrd
    # ls -al ../initrd*
    -rw-r--r-- 1 root root 26M 2015-09-06 11:54:53 ../initrd.gz

Taki plik może zostać wykorzystany z powodzeniem przez linux'owy booloader.

## Podejrzenie zawartości obrazu bez wypakowywania

Jeśli chcemy jedynie sprawdzić zawartość obrazu, to możemy go sobie zwyczajnie zamontować. Nie ma
przy tym znaczenia czy posługujemy się plikiem `gzip` czy `cpio` , bo ta metoda działa w obu
przypadkach. Ważne jest by przy montowaniu podać odpowiedni system plików, którym jest `sysfs` ,
zatem linijka montująca initrd wygląda tak:

    # mount -t sysfs /boot/initrd.img-4.1.0-2-amd64 /mnt

Istnieje także narzędzie
[lsinitramfs](http://manpages.ubuntu.com/manpages/wily/en/man8/lsinitramfs.8.html), które może nam
wypisać zawartość initramfs bez potrzeby jego montowania. Poniżej przykład użycia:

    # lsinitramfs /boot/initrd.img-4.3.0-1-amd64

## Mikrokod (Microcode)

Jak możemy przeczytać na [wiki Archlinux'a](https://wiki.archlinux.org/index.php/Microcode) ,
producenci procesorów wypuszczają aktualizacje mające na celu poprawienie bezpieczeństwa i
stabilności mikrokodu procesora. Sam kod może być uaktualniony za pośrednictwem BIOS'u komputera
ale linux'owy kernel jest także w stanie nałożyć te aktualizacje w fazie startu systemu
operacyjnego.

Jako, że mikrokod jest ważny, społeczność linux'owa zwykle instaluje dedykowane pakiety i dla
procesorów intel jest to `intel-microcode` , dla amd zaś `amd64-microcode` . W przypadku
niezainstalowania tych pakietów, nasz system może działać bardzo niestabilnie. Jednak gdy te paczki
są obecne w systemie, powyżej opisany sposób podejrzenia zawartości obrazu nie powiedzie się.

W przypadku wypakowania obrazu initrd zawierającego mikrokod, naszym oczom ukaże się tylko jeden
folder, co wygląda mniej więcej tak:

    # cpio -i < initrd.img-4.1.0-2-amd64
    8 blocks

    # tree
    .
    ├── initrd.img-4.1.0-2-amd64
    └── kernel
        └── x86
            └── microcode
                └── GenuineIntel.bin

    3 directories, 2 files

Naturalnie, ta cała struktura katalogów jest uwzględniona w obrazie ale nie może zostać wyciągnięta
i jeśli chcielibyśmy podejrzeć taki obraz, trzeba pierw wyrzucić z systemu paczki zawierające
`microcode-*` . Trzeba także pamiętać, że mikrokod nie jest wolny i [brak dokumentacji na temat jego
zawartości](http://stackoverflow.com/questions/4366837/what-is-intel-microcode) , dlatego też nie
jest on włączony bezpośrednio do debiana, a jedynie znajduje się w sekcji `non-free` .
