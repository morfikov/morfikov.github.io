---
author: Morfik
categories:
- Android
date:    2017-02-11 18:59:25 +0100
lastmod: 2017-02-11 18:59:25 +0100
published: true
status: publish
tags:
- szyfrowanie
- smartfon
- fastboot
- adb
- eryptfs
- twrp
title: Problem z zaszyfrowaniem partycji /data/ na smartfonie z Androidem
---

Bawiąc się ostatnio trochę mechanizmami szyfrującymi w moich smartfonach Neffos podesłanych przez
TP-LINK, po raz kolejny coś nieopatrznie uszkodziłem. Tym razem sprawa wygląda nieco bardziej
poważnie, bo uwalony został cały moduł szyfrujący urządzenie. Chodzi generalnie o to, że w
Androidzie w wersji 4.4/5.0 została wprowadzona [możliwość zaszyfrowania wszystkich danych
użytkownika][1], tj. informacji przechowywanych na partycji `/data/` . Do odszyfrowania tej partycji
potrzebny jest klucz szyfrujący. Problem w tym, że Android musi gdzieś ten klucz trzymać i to w taki
sposób, by proces Factory Reset był w stanie ten klucz usunąć, choćby na wypadek zapomnienia hasła i
próby odzyskania w takiej sytuacji władzy nad smartfonem. Pech chciał, że akurat na moim Neffos Y5
mam wgrane TWRP recovery i z jakiegoś powodu nie mogłem zresetować ustawień telefonu do fabrycznych
przez ten tryb i posłużyłem się narzędziem `fastboot` . Ono najwyraźniej nieco inaczej formatuje
partycję `/data/` i w ten sposób uwala cały mechanizm szyfrowania oferowany przez Androida. Czy da
radę jakoś poprawić ten problem, a jeśli tak to w jaki sposób?

<!--more-->
## Factory Reset przez tryb recovery, TWRP i fastboot

Proces Factory Reset można przeprowadzić na kilka sposobów. Ten najpopularniejszy to oczywiście
zresetowanie ustawień z poziomu działającego systemu. W zasadzie to po wybraniu odpowiedniej opcji z
menu, Android uruchomi telefon ponownie w trybie recovery i zainicjuje sekwencję formatowania
różnych partycji, zwykle `/data/` i `/cache/` . Ten proces praktycznie nie różni się zbytnio od
ręcznego przejścia w tryb recovery i wybrania z listy opcji "Factory Reset".

W tym przypadku jednak mamy do czynienia z TWRP, a nie ze stock'owym obrazem recovery. TWRP może w
pewnych sytuacjach powodować problem z resetowaniem ustawień smartfona do fabrycznych, zwłaszcza gdy
w konfiguracji obrazu nie została ustawiona flaga `TW_INCLUDE_CRYPTO` . W takiej sytuacji TWRP
recovery nie ma wkompilowanego wsparcia dla szyfrowania i nie potrafi poprawnie obsłużyć
zaszyfrowanej partycji `/data/` . Problemy pojawiają się także przy czyszczeniu tej partycji, a
jeśli nie wyczyścimy jej, to nie przeprowadzimy z powodzeniem całego procesu Factory Reset.

![](/img/2017/02/001.android-szyfrowanie-partycja-data-recovery-twrp-blad-data.png#big)

Jako, że narzędzie `fastboot` , które operuje na trybie bootloader'a naszego smartfona, jest również
w stanie przeprowadzić proces Factory Reset, to postanowiłem skorzystać z tej opcji i wpisałem w
terminalu poniższe polecenie:

    # fastboot -w
    Creating filesystem with parameters:
        Size: 13015953408
        Block size: 4096
        Blocks per group: 32768
        Inodes per group: 8192
        Inode size: 256
        Journal blocks: 32768
        Label:
        Blocks: 3177723
        Block groups: 97
        Reserved block group size: 775
    Created filesystem with 11/794624 inodes and 90399/3177723 blocks
    target reported max download size of 262144000 bytes
    Creating filesystem with parameters:
        Size: 268435456
        Block size: 4096
        Blocks per group: 32768
        Inodes per group: 8192
        Inode size: 256
        Journal blocks: 1024
        Label:
        Blocks: 65536
        Block groups: 2
        Reserved block group size: 15
    Created filesystem with 11/16384 inodes and 2089/65536 blocks
    erasing 'userdata'...
    OKAY [  6.678s]
    sending 'userdata' (137090 KB)...
    OKAY [  5.872s]
    writing 'userdata'...
    OKAY [  2.390s]
    erasing 'cache'...
    OKAY [  0.194s]
    sending 'cache' (6248 KB)...
    OKAY [  0.281s]
    writing 'cache'...
    OKAY [  0.170s]
    finished. total time: 15.584s

Jak widać, `fastboot` wyczyścił zarówno partycję `/data/` jak i `/cache/` . Zatem proces
przywracania smartfona do ustawień fabrycznych został zakończony powodzeniem. Problem w tym, że po
tak przeprowadzonym Factory Reset, Android zwyczajnie nie jest w stanie już dłużej zaszyfrować
telefonu.

## Dlaczego Android nie chce zaszyfrować smartfona

W zasadzie to nic nie wskazuje, że jakieś problemy mogą wyniknąć z przeprowadzenia procesu Factory
Reset przez `fastboot` . Telefon można używać jak gdyby nigdy nic, ale gdy w późniejszym czasie
postanowimy zaszyfrować ponownie to urządzenie, to w zasadzie się ono nam jedynie zresetuje
ponownie. Naturalnie po załadowaniu systemu, partycja `data` w dalszym ciągu będzie odszyfrowana.
Nie ma przy tym żadnych błędów ani wskazówek co do przyczyny takiego stanu rzeczy.

By ustalić co się popsuło trzeba sięgnąć nieco w głąb samego Androida, a konkretnie do logów
systemowych. Te z kolei można wyciągnąć przez `adb` , o ile oczywiście mamy zapewniony dostęp root.
Odpaliłem zatem terminal, podłączyłem się do smartfona i zalogowałem na użytkownika root via `su` .
Logi można uzyskać przez wpisanie polecenia `logcat` . W logu zaś przy próbie szyfrowania ale tuż
przed uruchomieniem się ponownie urządzenia można wyczytać min. poniższe komunikaty:

    # adb shell
    shell@Y5:/ $ su
    root@Y5:/ # logcat
    ...
    E Cryptfs : Bad magic for real block device /dev/block/bootdevice/by-name/userdata
    E Cryptfs : Orig filesystem overlaps crypto footer region.  Cannot encrypt in place.
    ...

Ta informacja jest raczej jednoznaczna i mówi o nałożeniu się systemu plików partycji `/data/` na
region stopki. Ta stopka to krańcowy obszar partycji `/data/` , a konkretnie jest to zwykle ostatnie
16 KiB. W tej stopce jest przechowywany klucz szyfrujący niezbędny w procesie szyfrowania danych na
partycji `/data/` . W ten sposób mamy możliwość włączenia lub wyłączenia szyfrowania, a proces
Factory Reset jest w stanie ten klucz usunąć przy czyszczeniu partycji.

W tym przypadku jednak po sformatowaniu partycji `/data/` przez `fastboot` , system plików wypełnia
całą dostępną na niej przestrzeń i nie zostawia żadnego miejsca na stopkę. W efekcie Android nie
może przeprowadzić procesu szyfrowania, bo nie ma gdzie upchnąć klucza szyfrującego dane.

## Jak poprawienie sformatować partycję /data/ na potrzeby szyfrowania

Ta powyżej opisana dolegliwość zdaje się mieć miejsce jedynie w przypadku formatowania partycji
`/data/` przez `fastboot` . Jeśli zabieg resetowania ustawień telefonu do fabrycznych zostanie
przeprowadzony przez stock'owy tryb recovery, to partycja `/data/` zostaje poprawnie sformatowana,
co zwraca nam możliwość szyfrowania danych. Nawet w TWRP recovery możemy taką partycję prawidłowo
sformatować na zakładce "WIPE" ale TWRP nie pomoże nam, gdy ta partycja została już zaszyfrowana.
Zatem jeśli znaleźliśmy się w takiej niemiłej sytuacji, to najlepiej załadować do pamięci czy też
wgrać na partycję recovery stock'owy obraz i przy jego pomocy przeprowadzić Factory Reset.

Problem jednak w tym, że w wielu sytuacjach skorzystanie ze stock'owego obrazu recovery nie będzie
dla nas możliwe, np. nie posiadamy już tego obrazu. Możemy też zwyczajnie nie być świadomi faktu
złego sformatowania partycji i połapiemy się dopiero wtedy, gdy będziemy chcieli zaszyfrować
smartfona i to akurat w momencie, gdy na telefonie będziemy mieć całą masę danych, których
wyczyszczenie nie wchodzi w grę.

## Zmiana rozmiaru partycji /data/

Gdy partycja `/data/` została źle sformatowana i mamy na niej całą masę danych, których z jakiegoś
powodu nie możemy skopiować w celu późniejszego odtworzenia, to w dalszym ciągu możemy cały
zaistniały problem poprawić. Potrzebny nam jest w zasadzie TWRP recovery, z poziomu którego musimy
zmienić rozmiar systemu plików partycji `/data/` tak, by zostawić na jej końcu trochę wolnego
miejsca. Praktycznie na każdym linux'ie wyposażonym w system plików EXT4, w tym też i na Androidzie,
ten zabieg można przeprowadzić bez utraty danych.

Oczywiście te 16 KiB, z którymi mamy do czynienia w tym przypadku, nie zawsze może być wartością
prawidłową. Jeśli nie jesteśmy pewni ile miejsca na końcu partycji Android w naszym smartfonie
rezerwuje sobie standardowo, to zawsze możemy to sprawdzić podglądając ilość wolnego miejsca po
przeprowadzeniu dwóch resetów do ustawień fabrycznych: jeden przez `fastboot` , drugi przez
stock'owy recovery.

Po każdym z tych resetów można wywołać `df` (tylko ten dostarczany przez Busybox) i sprawdzić jak
wygląda status systemu plików na partycji `/data/` przy pomocy tego poniższego polecenia:

    root@Y5:/ # busybox df -B 1

W moim przypadku w kolumnie `1-blocks` przy pozycji `/dev/block/bootdevice/by-name/userdata` miałem
wartości `12677435392` oraz `12677419008` , odpowiednio dla resetu przez `fastboot` i przez stock
recovery. Różnica tych dwóch wartości to 16384, czyli równo 16 KiB. Tyle wolnej przestrzeni
minimalnie trzeba zostawić na końcu partycji `/data/` . Jeśli nie chce nam się liczyć i
przeprowadzać procesu resetowania, to zawsze możemy przyjąć sobie wartość powiedzmy 1 czy 2 MiB i
to w zupełności powinno wystarczyć.

Odpalamy teraz smartfon w trybie recovery (VolumeUP + Power) i przechodzimy do terminala:

![](/img/2017/02/002.android-szyfrowanie-partycja-data-recovery-twrp-terminal.png#huge)

Sprawdzamy czy partycja `/data/` jest zamontowana. Jeśli tak, to trzeba ją odmontować:

![](/img/2017/02/003.android-szyfrowanie-partycja-data-recovery-twrp-umount-data.png#medium)

Skanujemy teraz system plików partycji `/data/` w poszukiwaniu ewentualnych błędów przy pomocy
narzędzia `e2fsck` :

![](/img/2017/02/004.android-szyfrowanie-partycja-data-recovery-twrp-fsck-data.png#medium)

Teraz zmieniamy rozmiar tej partycji za sprawą `resize2fs` :

![](/img/2017/02/005.android-szyfrowanie-partycja-data-recovery-twrp-resize-data.png#medium)

Tutaj mamy w zasadzie dwa polecenia. W pierwszym z nich, ta liczba na samym końcu jest duża
(1111111111), bo właściwie nie wiemy jeszcze jaki jest optymalny rozmiar tej partycji. Duża wartość
oczywiście zaowocuje błędem ale w tym komunikacie będziemy mieć maksymalny rozmiar partycji w
blokach 4096 bajtowych. W tym przypadku jest to 3177723 (13015953408 bajtów). Wystarczy od tej
wartości odjąć te nasze 16 KiB (czy też 1-2 MiB) i jeszcze raz wydać to polecenie. W tym przypadku
obciąłem 512 ostatnich bloków.

I to w zasadzie cała operacja. Teraz uruchamiamy smartfon ponownie i przechodzimy do Ustawienia =>
Zabezpieczenia i wybieramy opcję zaszyfrowania telefonu. Jeśli zostawiliśmy na końcu partycji
`/data/` wystarczającą ilość wolnego miejsca, to Android powinien zrestartować urządzenie i
przeprowadzić proces jego szyfrowania:

![](/img/2017/02/006.android-szyfrowanie-partycja-data-proces.jpg#big)

Po zaszyfrowaniu, smartfon zostanie uruchomiony ponownie raz jeszcze i już podczas startu systemu
będziemy proszeni o podanie hasła do klucza szyfrującego dane na partycji `/data/` (standardowo to
hasło jest takie samo jak to od blokady ekranu).


[1]: https://source.android.com/security/encryption/full-disk
