---
author: Morfik
categories:
- Linux
date: "2015-06-14T16:37:34Z"
date_gmt: 2015-06-14 14:37:34 +0200
published: true
status: publish
tags:
- moduły-kernela
- loop
GHissueID: 136
title: Obsługa wielu partycji w module loop
---

Wszelkie obrazy `.iso` płyt cd/dvd czy nawet pliki `.img` zawierające strukturę obrazów live możemy
zamontować lokalnie na komputerze uzyskując tym samym dostęp do ich systemu plików. W większości
przypadków, każdy taki obraz zawiera tylko jedną partycję, czasem lekko przesuniętą względem
początku ale generalnie nie ma większych problemów z zamontowaniem tego typu plików. Najwyżej
trzeba podać jeden dodatkowy parametr, tj. `offset` . A co w przypadku obrazów dysków, które zwykle
mają więcej niż jedną partycję? Gdybyśmy spróbowali zamontować taki plik w systemie, to zostanie nam
zwrócony błąd, no bo przecież kernel nie wie za bardzo jak taki plik ma odczytać. Możemy za to
skorzystać z urządzeń `loop` , które po odpowiedniej konfiguracji, są w stanie nam takie obrazy z
powodzeniem zamontować na naszym linux'ie.

<!--more-->
## Urządzenia loop

Przy montowaniu obrazów zwykle wykorzystujemy wirtualne urządzenia `loop` i standardowo by
zamontować taki plik z wieloma partycjami musimy dokładnie powyliczać offsety i rozmiary każdej z
partycji. Dla przykładu weźmy sobie poniższy obraz `binary.img` . Na samym początku podglądamy go w
`fdisk` :

    # fdisk binary.img
    ...
    Device      Boot  Start    End Sectors   Size Id Type
    binary.img1        2048 400000  397953 194.3M 83 Linux
    binary.img2      400001 819199  419199 204.7M 83 Linux

Mamy tam dwie partycje. Teraz by podpiąć pod nie urządzenia `loop` , musimy posłużyć się poleceniami
podobnymi do tych poniżej:

    # losetup -o 1048576 --sizelimit 203751936 /dev/loop1 binary.img
    # losetup -o 204800512 --sizelimit 214629888 /dev/loop2 binary.img

Opcja `-o` odpowiada za offset, który widzimy na pozycji `Start` pomnożony przez 512. Z kolei
`--sizelimit` to wartość `End-Start+1` również pomnożona przez 512. Jeśli wskazaliśmy poprawne
parametry, powinniśmy uzyskać poniższy wynik:

    # losetup
    NAME       SIZELIMIT    OFFSET AUTOCLEAR RO BACK-FILE
    /dev/loop1 203751936   1048576         0  0 /home/morfik/Desktop/binary.img
    /dev/loop2 214629888 204800512         0  0 /home/morfik/Desktop/binary.img

Powinniśmy być także w stanie zamontować obie partycje za pomocą standardowego polecenia `mount` :

    # mount /dev/loop1 /mnt
    # mount /dev/loop2 /mnt2

    # mount | grep loop
    /dev/loop1 on /mnt type ext4 (rw,relatime,data=ordered)
    /dev/loop2 on /mnt2 type ext4 (rw,relatime,data=ordered)

## Partycjonowanie urządzeń loop

Ten powyższy sposób nie jest to zbytnio wygodny. Poza tym, trzeba dużo liczyć i wszystko montować
samodzielnie, a łatwo się też przy tym pomylić. Przeglądając opcje modułu `loop` doszukałem się
parametru `max_part` , który odpowiada za ilość dodatkowych urządzeń, które zostaną utworzone w
zależności od wykrytych partycji w obrazie. W dystrybucji Debian, ta wartość jest ustawiona na 0:

    $ cat /sys/module/loop/parameters/max_part
    0

I to właśnie z tego powodu takie cuda trzeba wyrabiać, by operować na obrazach `.img` posiadających
wiele partycji. Wyładujmy zatem ten moduł i załadujmy go jeszcze raz, tym razem z ustawioną opcją
`max_part` , przykładowo:

    # modprobe -r loop
    # modprobe loop max_part=31

By teraz załadować taki obraz na urządzenie `loop` , wpisujemy poniższe polecenie:

    # losetup /dev/loop0 binary.img

    # losetup
    NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
    /dev/loop0         0      0         0  0 /home/morfik/Desktop/binary.img

Jak możemy zauważyć, zostało wykorzystane tylko jedno urządzenie `loop0` , natomiast jeśli zajrzymy
do katalogu `/dev/` , to ujrzymy kilka urządzeń mających w nazwie `loop0` :

    # ls -a /dev/loop0*
    /dev/loop0  /dev/loop0p1  /dev/loop0p2

Są tam dwa dodatkowe urządzenia odpowiadające ilości partycji w obrazie. By operować na
poszczególnych partycjach, wystarczy się do nich odwołać i nie musimy przy tym zawracać sobie głowy
zbędnymi obliczeniami.

Trzeba jednak pamiętać, że powyższe przeładowanie modułu ma efekt tylko tymczasowy, tj. do restartu
komputera. By ten moduł zawsze był ładowany z określonymi opcjami, trzeba stworzyć plik
`/etc/modprobe.d/modules.conf` i dodać tam poniższą linijkę:

    options loop max_part=31

## Moduł loop wkompilowany na stałe w kernel

W części dystrybucji linux'owych, np. Ubuntu, moduł `loop` jest wkompilowany na stałe w kernel
(buil-in). Dlatego też te powyższe zabiegi pod kątem dostosowania ilości partycji nic nam nie dadzą,
a próba wyładowania modułu zwróci poniższy błąd:

    # modprobe -r loop
    modprobe: FATAL: Module loop is builtin.

Tego typu moduły można konfigurować jedynie za pomocą parametrów podawanych w linijce kernela przez
bootloader, np. Grub czy extlinux. Edytujemy zatem konfigurację bootloader'a. W tym przypadku jest
to plik `/boot/extlinux/extlinux.conf` i dopisujemy parametr `loop.max_part=31` w linijce z
`APPEND` , przykładowo:

    APPEND root=/dev/mapper/debian_laptop-root ... loop.max_part=31 ... quiet splash ro

Zmiany będą obowiązywać po ponownym uruchomieniu komputera.
