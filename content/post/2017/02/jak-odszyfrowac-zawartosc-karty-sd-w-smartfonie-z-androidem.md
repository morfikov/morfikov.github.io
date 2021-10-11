---
author: Morfik
categories:
- Android
date:    2017-02-09 18:01:56 +0100
lastmod: 2017-02-09 18:01:56 +0100
published: true
status: publish
tags:
- karta-sd
- smartfon
- szyfrowanie
- adoptable-storage
- root
- adb
- dmcrypt
GHissueID: 57
title: Jak odszyfrować zawartość karty SD w smartfonie z Androidem
---

W Androidzie 6.0 Marshmallow został wprowadzony ciekawy mechanizm zwany [Adoptable Storage][1],
który umożliwia [zamontowanie karty SD w smartfonie jako pamięć wewnętrzna][2]. W ten
sposób pamięć flash w telefonach, które mają jej niewiele, może zostać nieco rozbudowana. Jedyny
problem z tym całym Adoptable Storage jest taki, że Android szyfruje zawartość karty SD
automatycznie, przez co nie jesteśmy w stanie odczytać żadnych informacji z takiego nośnika na
innych urządzeniach. Istnieje jednak sposób, by rozszyfrować i tym samym uzyskać dostęp do danych
zgromadzonych na karcie SD z poziomu linux'a, np. dystrybucji Debian. W tym artykule prześledzimy
sobie właśnie ten proces na przykładzie [smartfona Neffos Y5][3] od TP-LINK.

<!--more-->
## Ukorzeniony Android Marshmallow (root)

Jako, że w grę wchodzi szyfrowanie danych, to ten proces zwykle jest należycie chroniony przez
system w naszym telefonie. Dlatego też dostęp do interesujących nas plików (np. via `adb` ) jest
bardzo ograniczony. By uzyskać dostęp do tych plików, musimy posiadać ukorzeniony system na
smartfonie, tj. przeprowadzić na nim proces root.

W przypadku tych TP-LINK'owych smartfonów dostępnych na polskim rynku, to tylko Neffos Y5 i Neffos
Y5L mają wgranego Androida w wersji 6.0, dlatego też cały proces zostanie opisany w oparciu o jedno
z tych urządzeń. W zasadzie mocniejszym sprzętem jest Neffos Y5 i dlatego zdecydowałem się nim
posłużyć. Informacje na temat [ukorzeniania Androida w Neffos Y5][4] znajdują się tutaj. Zakładam
zatem w tym miejscu, że nasz smartfon przeszedł proces root.

## Formatowanie karty SD jako pamięć wewnętrzna

Weźmy sobie zatem w łapki nasz smartfon i wsadźmy do niego jakąś kartę SD, którą zamierzamy
sformatować jako pamięć wewnętrzna. Cały proces jest niemal automatyczny i wystarczy przejść do
Ustawienia => Pamięć plików i kliknąć w "Karta SD". Mamy tam pozycję "Sformatuj jako pamięć wewn."
i to w nią klikamy.

![](/img/2017/02/001.smartfon-android-marshmallow-tp-link-neffos-y5-karta-sd.png#medium)

Postępujemy zgodnie z informacjami na ekranie i po chwili karta powinna zostać przygotowana do
pracy, tj. sformatowana, zaszyfrowana i zamontowana w systemie:

![](/img/2017/02/002.smartfon-android-marshmallow-tp-link-neffos-y5-karta-sd.png#huge)

Możemy też sprawdzić, gdzie Android montuje naszą kartę SD w systemie (przez `adb`) korzystając z
polecenia `mount` :

    root@Y5:/ # mount | grep expand
    /dev/block/dm-0 /mnt/expand/9acfb46c-f218-43e7-a14a-a3592183af6d ext4 rw,dirsync,seclabel,nosuid,nodev,noatime 0 0

## Poszukiwanie klucza szyfrującego dane karty SD

Kartę SD można naturalnie wyciągnąć ze smartfona (pierw trzeba ją odmontować w Androidzie) i
podłączyć do komputera przez czytnik kart SD. Gdybyśmy jednak taki zabieg zrobili, to okaże się,
że nasz linux w komputerze nie rozpozna nam tego nośnika. W prawdzie `gdisk` będzie nam w stanie
podać informacje o układzie partycji ale nie dostaniemy się do samego systemu plików:

    # gdisk -l /dev/sdb
    GPT fdisk (gdisk) version 1.0.1

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.
    Disk /dev/sdb: 3854336 sectors, 1.8 GiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): 1281EE27-00C5-4CC7-8004-94DDBCEF829C
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 3854302
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 2014 sectors (1007.0 KiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048           34815   16.0 MiB    FFFF  android_meta
       2           34816         3854302   1.8 GiB     FFFF  android_expand

Nas tutaj interesuje ta druga partycja, bo to tam są zgromadzone zaszyfrowane dane. By informacje na
tej partycji odszyfrować, potrzebny nam jest klucz szyfrujący, a ten z kolei znajduje się gdzieś na
smartfonie, pytanie tylko gdzie? Wzór nazwy klucza można wyciągnąć w poniższy sposób:

    root@Y5:/ # which vold
    /system/bin/vold

    root@Y5:/ # strings /system/bin/vold | grep -i expand
    --change-name=0:android_expand
    /mnt/expand/%s
    %s/expand_%s.key

Wiemy zatem, że klucz nazywa się `expand_%s.key` , z tym, że do końca nie wiem za co odpowiada
`%s` . Tak czy inaczej, wszystkie klucze są przechowywane pod `/data/misc/vold/` :

    root@Y5:/ # ls -al /data/misc/vold/
    drwx------ root     root              2017-02-08 22:40 bench
    -rw------- root     root           16 2017-02-08 22:40 expand_06bc225178c5900fe64800e4b70efd25.key

Ten plik z rozszerzeniem `.key` jest tworzony w momencie, gdy formatujemy kartę SD jako pamięć
wewnętrzna. Podobnie jest on usuwany ilekroć tylko będziemy nakazywać Androidowi zapomnienie tego
nośnika. Jak widać, mając ukorzeniony system, dostęp do tego klucza jest praktycznie niechroniony
(do tego brak zabezpieczenia hasłem), zatem każdy może go skopiować. Będąc w posiadaniu tego klucza,
możemy odszyfrować zawartość karty SD na dowolnej maszynie wyposażonej w pierwszą lepszą dystrybucję
linux'a.

## Odszyfrowanie zawartości karty SD

Znając położenie klucza szyfrującego na smartfonie, możemy ten klucz wydobyć w poniższy sposób
(zakodowany w HEX):

    root@Y5:/ # od -t x1 /data/misc/vold/expand_06bc225178c5900fe64800e4b70efd25.key
    37777776620 6c 33 be 5d df 7b a0 d0 ca 41 fe d7 a7 8f 75 db
    37777776620

Ten widoczny wyżej ciąg 16 bajtów (128 bitów) tj. `6c33be5ddf7ba0d0ca41fed7a78f75db` , to nasz klucz
szyfrujący dane na karcie SD, który teraz trzeba wskazać w `dmsetup` na linux. Dodatkowo,
potrzebnych jest nam jeszcze kilka parametrów. Musimy min. wiedzieć jaki rodzaj szyfru jest
wykorzystywany w procesie szyfrowania i deszyfrowania informacji na karcie SD oraz musimy znać
szereg offsetów. Zgodnie z [informacjami jakie znalazłem tutaj][5], Android korzysta z
`aes-cbc-essiv:sha256` . Offsety mogą być różne ale za punkt wyjścia można obrać `0` . Zatem linijka
z `dmsetup` przybierze poniższą postać:

    # dmsetup create sdcard --table "0 `blockdev --getsize /dev/sdb2` crypt aes-cbc-essiv:sha256 6c33be5ddf7ba0d0ca41fed7a78f75db 0 /dev/sdb2 0"

Dokładne [wyjaśnienie wszystkich opcji][6] użytych powyżej znajduje się tutaj.

Jeśli to powyższe polecenie nie zwróci żadnego błędu, to możemy spróbować zamontować ten zasób w
systemie. System plików, który został utworzony przez Androida w tym zaszyfrowanym kontenerze znamy
i jest to linux'owy EXT4. Odszyfrowany kontener zaś jest dostępny pod `/dev/mapper/sdcard` :

    # mount -t ext4 /dev/mapper/sdcard /mnt/

    # ls -al /mnt
    total 36K
    drwxr-xr-x.  8 root   root   4.0K 2017-02-08 22:40:50 ./
    drwxr-xr-x  25 root   root   4.0K 2017-01-14 10:22:16 ../
    drwxrwx--x.  2 morfik morfik 4.0K 2017-02-08 22:40:31 app/
    drwxr-x--x.  3 root   root   4.0K 2017-02-08 22:40:38 local/
    drwx------.  2 root   root   4.0K 1970-01-01 01:00:00 lost+found/
    drwxrwx---.  4   1023   1023 4.0K 2017-02-08 22:42:09 media/
    drwxrwx--t.  3 morfik   9998 4.0K 2017-02-08 22:40:51 misc/
    drwx--x--x.  3 morfik morfik 4.0K 2017-02-08 22:40:40 user/

Zatem udało nam się uzyskać dostęp do zaszyfrowanych danych. Jeśli teraz wgralibyśmy coś tutaj do
katalogu `user/` czy `media/` , to naturalnie Android w naszym Neffos Y5 będzie w stanie bez
problemu ten plik odczytać.

Standardowo w przypadku, gdy zamierzamy sformatować kartę SD jako pamięć wewnętrzna, to musimy się
liczyć z faktem bezpowrotnej utraty wszystkich danych zgromadzonych w telefonie, gdy ten ulegnie
jakiejś większej awarii. Posiadając jednak ukorzenionego Androida, możemy taki klucz szyfrujący
skopiować sobie na dysk komputera i tam go trzymać. Gdy telefon ulegnie uszkodzeniu, to dane z karty
SD będziemy w stanie odczytać. Trzeba jednak również pamiętać o tym, że z telefonu ten klucz można
wyciągnąć bez problemu o ile mamy w nim odblokowany bootloader (na potrzeby root, czy custom
recovery), a że nie jest on w żaden sposób zabezpieczony (PIN, hasło, itp), to każdy kto zna się
trochę na telefonach będzie w stanie uzyskać dostęp do danych zgromadzonych na tej karcie SD.


[1]: https://source.android.com/devices/storage/adoptable
[2]: /post/android-formatowanie-karty-sd-jako-pamiec-wewnetrzna/
[3]: http://www.neffos.pl/product/details/Y5
[4]: /post/android-root-smartfona-neffos-y5-od-tp-link/
[5]: https://source.android.com/devices/storage/adoptable#security
[6]: https://gitlab.com/cryptsetup/cryptsetup/wikis/DMCrypt
