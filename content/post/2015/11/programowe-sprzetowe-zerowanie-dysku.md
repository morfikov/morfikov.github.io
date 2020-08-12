---
author: Morfik
categories:
- Linux
date: "2015-11-24T17:54:17Z"
date_gmt: 2015-11-24 16:54:17 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- hdd/ssd
title: Programowe i sprzętowe zerowanie dysku
---

Zerowanie dysku twardego ma na celu usunięcie wszystkich znajdujących się na nim danych. Generalnie
chodzi o zapisanie całej powierzchni danego nośnika samymi zerami. Ten proces różni się znacząco of
formatowania dysku, czyli utworzenia nowego systemu plików, gdzie praktycznie wszystkie dane można
bez większego problemu odzyskać. Zerowanie dysku (czy też pendrive) może w pewnych przypadkach
[naprawić logiczne błędy
sektorów]({{< baseurl >}}/post/uszkodzony-sektor-na-dysku-i-jego-realokoacja/) na dysku. Niemniej
jednak, nie usuniemy za jego pomocą fizycznych bad'ów. Generalnie rzecz biorąc, mamy do wyboru dwie
techniki zerowania. Jedna jest dokonywana na poziomie programowym, np. przy pomocy `dd` , druga zaś
na poziomie sprzętowym, np. w `hdparm` . W tym wpisie postaramy się wyzerować przykładowy dysk.

<!--more-->
## Programowe zerowanie

W przypadku gdy naszemu dyskowi nic nie dolega lub problem dotyczy jedynie sfery logicznej, to
prawdopodobnie wszelkie zaistniałem błędy będziemy w stanie poprawić przez wyzerowanie takiego
nośnika przy pomocy narzędzia `dd` . Trzeba być bardzo ostrożnym podczas tego procesu, bo wszelkie
dane zostaną utracone i jeśli wskażemy nie ten dysk lub partycję co potrzeba, to dopiero będziemy
mieć problem. Tak czy inaczej, podpinamy to problematyczne urządzenie do komputera i patrzymy w
`lsblk` jaki numerek został mu przypisany. W tym przypadku jest to `sdc` . Zatem plik urządzenia
znajduje się w `/dev/sdc` . Odpalamy teraz terminal, logujemy się jako root i wpisujemy to poniższe
polecenie:

    # dd if=/dev/zero of=/dev/sdc bs=16M

Teraz tylko trzeba poczekać aż system skończy zerować dysk. W przypadku gdy zerowanie miałoby
dotyczyć jedynie pojedynczej partycji, to zamiast `sdc` powinniśmy użyć `sdc1` , itd.

## Sprzętowe zerowanie

[Zerowanie na poziomie sprzętowym](https://ata.wiki.kernel.org/index.php/ATA_Secure_Erase) różni się
nieco od zerowania programowego. Zwykle w tym procesie są wykorzystywane mechanizmy oferowane przez
firmware konkretnego nośnika, tj. Secure Erase oraz Enhanced Secure Erase. Różnica między nimi
polega na tym, że ta druga metoda nadpisuje dane kilkoma różnymi wzorcami bitów, by mieć pewność, że
wszelkie informacje na nośniku zostały kompletnie zniszczone. Dodatkowo, Enhanced Secure Erase jest
w stanie także przepisać sektory, które nie są już używane ze względu na błędy zapisu/odczytu, czyli
przemapowane błędne sektory. Ta technika jest także w stanie wyzerować również takie obszary dysku
jak HPA ([Host Protected Area](https://en.wikipedia.org/wiki/Host_protected_area)), DCO ([Device
Configuration Overlay](https://en.wikipedia.org/wiki/Device_configuration_overlay)) czy nawet
ustawienia samego firmware. Zerowanie sprzętowe jest też szybsze niż to programowe.

### Czy mój dysk wspiera sprzętowe zerowanie

Nie wszystkie dyski wspierają zerowanie na poziomie sprzętowym, a specyfikacja tych, które to robią,
nie jest nigdzie określona i trzeba zwykle ustalać pewne informacje dla każdego nośnika z osobna.
Zacznijmy zatem od `hdparm` , który może nam nieco pomóc w określeniu czy nasz dysk obsługuje jakąś
metodę sprzętowego zerowania. Robimy to przez wydanie w terminalu jako root tego poniższego
polecenia:

    # hdparm -I /dev/sda
    ...
            device size with M = 1000*1000:      250059 MBytes (250 GB)
    ...
    Security:
    ...
                    supported: enhanced erase
            52min for SECURITY ERASE UNIT. 52min for ENHANCED SECURITY ERASE UNIT.
    ...

Jak widzimy w powyższym logu, ten dysk ma 250GB. Mamy tam także linijkę zawierająca `52min` dla obu
typów zerowania sprzętowego. Mamy zatem spore prawdopodobieństwo, że ten dysk wspiera zerowanie na
poziomie sprzętowym. Czas 52min na wyczyszczenie całej przestrzeni 250GB wydaje się być w miarę
sensowy (około 100MB/s zapisu). Jako, że czasy w przypadku SECURITY ERASE jak i ENHANCED SECURITY
ERASE są takie same, możemy założyć, że te dwie metody się niczym nie różnią, co w praktyce oznacza,
że jest zaimplementowana tylko jedna z nich.

### Zerowanie przy pomocy hdparm

Nigdy nie przeprowadzałem zerowania sprzętowego, bo na dobrą sprawę żaden mój dysk, za wyjątkiem
głównego, nie obsługuje tej opcji. Tak czy inaczej [pod tym
linkiem](https://tinyapps.org/docs/wipe_drives_hdparm.html) jest dość przyzwoity opis
przeprowadzonych operacji, który postaram się tutaj przepisać, tak by nigdzie się nie zawieruszył.
To tak na wszelki wypadek, gdybym kiedyś w przyszłości posiadał więcej dysków i chciał przeprowadzić
sprzętowe zerowanie dysku.

Przede wszystkim, musimy się upewnić, że mechanizm bezpieczeństwa dysku nie jest zamrożony. Możemy
to odczytać z wyjścia `hdparm` :

    # hdparm -I /dev/sda
    ...
    Security:
    ...
            not     frozen
    ...

W przypadku gdyby w powyższym logu nie było słówka `not` , oznaczałoby to, że musimy przepuścić ten
dysk przez cykl zasilania, tj. wyłączyć i włączyć ponownie. Jeśli to nie pomoże, to prawdopodobnie
BIOS coś z tym dyskiem robi i w takim przypadku trzeba go odłączyć na czas pierwszej fazy startu
maszyny. Zawsze można także uśpić system, a następnie go wybudzić bez odłączania czegokolwiek.
Możemy także skorzystać z opcji hotplug oferowanej przez urządzenia SATA, gdzie cykl odłączenia
urządzenia można przeprowadzić przez wydanie tych poniższych poleceń:

    # ls -ld /sys/block/sda
    lrwxrwxrwx 1 root root 0 2015-11-24 15:18:26 /sys/block/sda -> ../devices/pci0000:00/0000:00:1f.2/ata1/host0/target0:0:0/0:0:0:0/block/sda/

    echo 1 > /sys/block/sda/device/delete

    echo "- - -" > /sys/class/scsi_host/host0/scan

Po tym jak mechanizm bezpieczeństwa zostanie odmrożony, ustawiamy hasło:

    # hdparm --user-master u --security-set-pass p /dev/sda
    security_password="p"

Po wydaniu tego powyższego polecenia, mechanizm bezpieczeństwa dysku powinien zostać aktywowany:

    # hdparm -I /dev/sdx
    ...
    Security:
    ...
                enabled
    ...

Teraz możemy przeprowadzić zerowanie dysku przy pomocy poniższego polecenia:

    # hdparm --user-master u --security-erase p /dev/sdx

W przypadku gdy urządzenie wspiera Enhanced Secure Erase, zamiast `--security-erase` można
skorzystać z `--security-erase-enhanced` .

Po zakończeniu procesu zerowania, mechanizm bezpieczeństwa dysku powinien zostać automatycznie
wyłączony. Jeśli się tak nie stało, to możemy go wyłączyć ręcznie. Trzeba jednak mieć na
względzie, że skoro mechanizm bezpieczeństwa jest w dalszym ciągu włączony, to prawdopodobnie
zerowanie nie zostało ukończone z powodzeniem. W każdym razie, to czy mechanizm został zdjęty można
odczytać w `hdparm` :

    # hdparm -I /dev/sda
    ...
    Security:
    ...
                enabled
                locked
    ...

By wyłączyć ten mechanizm ręcznie, wpisujemy w terminal te dwa poniższe polecenia:

    # hdparm --user-master u --security-unlock p /dev/sda

    # hdparm --user-master u --security-disable p /dev/sda
