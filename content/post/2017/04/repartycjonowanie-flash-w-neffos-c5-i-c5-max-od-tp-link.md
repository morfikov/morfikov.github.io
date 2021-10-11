---
author: Morfik
categories:
- Android
date:    2017-04-03 19:51:32 +0200
lastmod: 2017-04-03 19:51:32 +0200
published: true
status: publish
tags:
- flash
- tp-link
- smartfon
- neffos
- neffos-c5
- neffos-c5-max
- twrp
- adb
GHissueID: 43
title: Repartycjonowanie flash'a w Neffos C5 i C5 MAX od TP-LINK
---

Analizując sobie fabryczny podział przestrzeni flash w TP-LINK'owych smartfonach, a konkretnie w
modelach Neffos C5 i C5 MAX, doszedłem do wniosku, że producent tych urządzeń nieco zaszalał
przeznaczając aż 4 GiB przestrzeni na partycję `/system/` . W zasadzie ROM w tych telefonach zajmuje
około 2 GiB. Zatem pozostałe 2 GiB zwyczajnie się marnuje i przeciętny użytkownik smartfona nie
będzie w stanie tego obszaru w żaden sposób wykorzystać. Można by zatem inaczej przepartycjonować
ten flash, tak by nieco skurczyć samą partycję `/system/` , przeznaczając jednocześnie odzyskane
miejsce na powiększenie partycji `/data/` . W tym wpisie postaramy się właśnie taki zabieg zmiany
rozmiaru partycji `/system/` przeprowadzić dla tych dwóch wyżej wymienionych modeli smartfonów.

<!--more-->
## Czy zmieniać układ flash'a na smartfonie

Zarówno Neffos C5 jak i Neffos C5 MAX dysponują flash'em 16 GiB. Jak wspomniane zostało we wstępie,
4 GiB z tej przestrzeni jest zarezerwowane na partycję `/system/` . Rzadko się jednak zdarzy tak, że
stock'owy ROM TP-LINK'a będzie tyle zajmował. W zasadzie to raczej jest bardzo małe
prawdopodobieństwo, że rozmiar tego ROM'u przekroczy 2,5 GiB. Możemy zatem spokojnie wykroić 1,5
GiB z tej partycji i mieć więcej miejsca na dane użytkownika, np. zdjęcia czy filmy.

Oczywiście partycji `/system/` nie można przyciąć za bardzo. Nawet jeśli wykroimy sobie dokładnie
tyle miejsca, by zmieścił się nam na tej partycji stock'owy ROM, to w dalszym ciągu cześć aplikacji
wymagających uprawnień root może chcieć zapisywać pewne informacje na partycji `/system/` . Jeśli te
programiki nie będą w stanie tego uczynić, to może to doprowadzić do niestabilności systemu, czy
nawet problemów z uruchomieniem się Androida. Trzeba zatem zostawić trochę miejsca ale znowu bez
przesady.

Musimy także liczyć się z powiększeniem ROM'u po ewentualnych aktualizacjach, które będziemy w
stanie pobrać via OTA Update. Te aktualizacje nie są duże, a sam ROM od nich zwykle nie tyje ale w
przypadku gdyby TP-LINK wypuścił aktualizację Androida z Lollipop do Marshmallow, to już zmiana w
rozmiarze ROM'u może być widoczna. W dalszym jednak ciągu uważam, że przeznaczenie 4 GiB na partycję
`/system/` to szaleństwo i przydałoby się nieco zoptymalizować podział flash'a.

## Fabryczny układ flash'a w Neffos C5 i Neffos C5 MAX

Zacznijmy może od fabrycznego układu flash'a w Neffos C5 i C5 MAX. Układ w tych dwóch modelach jest
bardzo podobny ale nie jest dokładnie taki sam. W zasadzie ta rzecz, która nas interesuje zdaje się
być stała. Chodzi o kolejność partycji. W przypadku tych dwóch smartfonów, partycja `/system/` na
numer 20, partycja `/cache/` ma numer 21, a partycja `/data/` ma numer 22. Zatem mamy trzy partycje
jedna po drugiej, które musimy usunąć i stworzyć na nowo. Telefon po usunięciu tych trzech partycji
powinien się nam uruchomić bez problemu, choć z oczywistych względów nie załaduje nam systemu.

Użytkowników smartfonów Neffos Y5 i Y5L, którzy zastanawiają się dlaczego nie opisuję w tym artykule
również i tych dwóch urządzeń, informuję, że układ flash'a w tych dwóch telefonach znacznie się
różni w stosunku do tego, z którym mamy do czynienia w Neffos C5 i C5 MAX. W przypadku Y5 i Y5L
trzeba usunąć 9 z 29 partycji wliczając w to partycję `/recovery/` . Istnieje zatem podwyższone
ryzyko uwalenia telefonu na dobre, a biorąc pod uwagę, że przeprowadzane kroki i tak będą się
różniły, to postanowiłem nie mieszać wszystkich czterech urządzeń i zrobić 2 osobne artykuły.

Poniżej jest fotka obrazująca układ flash'a dla smartfona Neffos
C5.

![](/img/2017/04/001.tp-link-neffos-c5-max-flash-repartycjonowanie-fabryczny-uklad.png#huge)

## Tworzenie nowego układu partycji na flash'u smartfona

Wiemy zatem, że mamy usunąć trzy partycje: `/system/` , `/cache/` oraz `/data/` . Problem w tym, że
by usunąć partycję z flash'a smartfona, potrzebne nam są prawa administratora root. Nie koniecznie
musimy przeprowadzać proces ukorzeniania Androida. [Wystarczy wgrać na telefon obraz TWRP
recovery][1], czy to przy pomocy `fastboot` , czy też via [SP Flash Tool][2]. Mając wgrany TWRP,
odpalamy telefon w trybie recovery (przyciski VolUp+Power). Będąc w trybie recovery, podłączamy
smartfon do portu USB komputera, na którym to uruchamiamy terminal i wpisujemy poniższe polecenie:

    # adb shell
    ~ #

To polecenie zapewni nam dostęp root i będziemy mogli bardziej swobodnie operować na smartfonie przy
pomocy naszego komputera.

### Backup tablicy partycji do pliku

Zanim cokolwiek zaczniemy zmieniać w układzie partycji na flash'u smartfona, zróbmy sobie kopię
tablicy partycji i zapiszmy ją w pliku gdzieś na karcie SD. Nie zapisujmy tego pliku w pamięci
telefonu, bo jak coś uszkodzimy w układzie partycji, to nie będziemy mieć dostępu do backup'u.
Backup tablicy partycji robimy w poniższy sposób:

    ~ # sgdisk --backup=/sdcard1/backup-partition-table /dev/block/mmcblk0

### Jak usunąć partycje /system/ , /cache/ i /data/

Mając backup tablicy partycji, możemy przystąpić do usuwania wpisów z tej tablicy. Przed usunięciem
jakiekolwiek partycji, trzeba ją pierw odmontować. Wpiszmy zatem w terminalu te poniższe polecenia:

    ~ # umount /cache
    ~ # umount /data
    ~ # umount /sdcard
    ~ # umount /system

Teraz możemy usunąć te trzy partycje przy pomocy narzędzia `sgdisk` w poniższy sposób:

    ~ # sgdisk --delete=20 /dev/block/mmcblk0
    ~ # sgdisk --delete=21 /dev/block/mmcblk0
    ~ # sgdisk --delete=22 /dev/block/mmcblk0

### Jak zmienić rozmiar partycji /system/ , /cache/ i /data/

Teraz tworzymy na nowo partycję `/system/` , zaraz za partycją numer 19. Musimy określić trzy
wartości. Numer tworzonej partycji, sektor początkowy oraz rozmiar partycji. Numer jest znany,
tj. 20. Rozmiar też znamy, np. 2,5 GiB. Jak określić początek tej partycji? Możemy go wywnioskować z
wyjścia poniższego polecenia:

    ~ # sgdisk --print /dev/block/mmcblk0
    Number  Start (sector)    End (sector)  Size       Code  Name

    ...
      19          284672          360447   37.0 MiB    0700  metadata
    ...

Ta partycja numer 19 była przed partycją `/system/` . Widzimy, że mamy określony tam końcowy sektor,
tj. `360447` . Następna partycja ma zaczynać się na sektorze o jeden większym, czyli `360448` .
Rozmiar nowej partycji `/system/` podajemy jako +2560M, czyli 2,5G. W terminalu zatem wpisujemy
poniższe polecenie:

    ~ # sgdisk --new=20:360448:+2560M /dev/block/mmcblk0

Nadajemy jeszcze nazwę tej partycji:

    ~ # sgdisk --change-name=20:system /dev/block/mmcblk0

Powyższy krok z tworzeniem partycji i zmianą nazwy powtarzamy dla partycji `/cache/` :

    ~ # sgdisk --new=21:5603328:+512M /dev/block/mmcblk0
    ~ # sgdisk --change-name=21:cache /dev/block/mmcblk0

Podobnie postępujemy w przypadku partycji `/data/` , z tym, że tutaj jako rozmiar partycji podajemy
sektor początkowy ostatniej partycji odejmując od tej wartości 1:

    ~ # sgdisk --new=22:6651904:30501887 /dev/block/mmcblk0
    ~ # sgdisk --change-name=22:userdata /dev/block/mmcblk0

Ustawiamy typy nowo utworzonych partycji na `0700` :

    ~ # sgdisk --typecode=20:0700 /dev/block/mmcblk0
    ~ # sgdisk --typecode=21:0700 /dev/block/mmcblk0
    ~ # sgdisk --typecode=22:0700 /dev/block/mmcblk0

### Podejrzenie nowego układu flash'a

Po utworzeniu nowych partycji i określeniu ich rozmiaru, nazw oraz typów sprawdzamy czy wszystko się
zgadza:

![](/img/2017/04/002.tp-link-neffos-c5-max-flash-repartycjonowanie-zmiana-ukladu.png#huge)

Jak widać, w tej chwili nasz smartfon ma około 11,5 GiB na partycji `/data/` w stosunku do starych
10 GiB, czyli zysk na poziomie 15%.

### Formatowanie partycji

Jeśli nowy układ partycji spełnia nasze oczekiwania, to wypadałoby teraz sformatować wszystkie trzy
partycje systemem plików EXT4 przy pomocy narzędzia `mke2fs` :

    ~ # mke2fs -t ext4 /dev/block/mmcblk0p20
    ~ # mke2fs -t ext4 /dev/block/mmcblk0p21
    ~ # mke2fs -t ext4 /dev/block/mmcblk0p22

Montujemy te zasoby, czy to przy pomocy polecenia `mount` , czy też z GUI TWRP. Sprawdzamy też, czy
te partycje zostały poprawnie zamontowane w systemie:

    ~ # mount | grep -i block
    /dev/block/mmcblk0p20 on /system type ext4 (rw,seclabel,relatime,data=ordered)
    /dev/block/mmcblk0p21 on /cache type ext4 (rw,seclabel,relatime,data=ordered)
    /dev/block/mmcblk0p22 on /data type ext4 (rw,seclabel,relatime,data=ordered)

Powyższy wynik świadczy o tym, że repartycjonowanie flash'a w naszym smartfonie przebiegło z
powodzeniem.

## Problemy z wgraniem systemu z pliku update.zip przez ADB Sideload

Nasz smartfon w tej chwili nie ma jeszcze wgranego żadnego systemu operacyjnego. Musimy zatem jakoś
ten system zainstalować. Opcji jest kilka. Najprostszym rozwiązaniem wydawałoby się postaranie się o
[plik update.zip i wgranie go via ADB Sideload z poziomu TWRP recovery][3]. Plik z firmware można
pobrać z oficjalnej strony TP-LINK/Neffos. Problem w tym, że tak łatwo nie będzie i wgranie systemu
z pliku `update.zip` zakończy się niepowodzeniem.

Z początku nie wiedziałem w czym rzecz. No bo przecież TP-LINK'owy ROM do Neffos C5 i C5 MAX ma
około 2 GiB, a przy wgrywaniu go za pomocą ADB Sideload przez TWRP recovery, system wyrzuca
informację o braku miejsca:

    sideload-host file size 993462838 block size 65536
    Installing zip file '/sideload/package.zip'
    I:Update binary zip
    I:Zip contains SELinux file_contexts file in its root. Extracting to /file_contexts
    I:Legacy property environment initialized.

    ====== Updater-Script:
    getprop("ro.product.device") == "C5" || abort("This package is for \"C5\" devices; this is a \"" + getprop("ro.product.device") + "\".");
    show_progress(0.750000, 0);
    ui_print("Patching system image unconditionally...");
    block_image_update("system", package_extract_file("system.transfer.list"), "system.new.dat", "system.patch.dat");
    show_progress(0.050000, 5);
    assert(package_extract_file("boot.img", "/tmp/boot.img"),
           write_raw_image("/tmp/boot.img", "bootimg"),
           delete("/tmp/boot.img"));
    assert(package_extract_file("mobicore.bin", "/tmp/tee1.img"),
           write_raw_image("/tmp/tee1.img", "tee1"),
           delete("/tmp/tee1.img"));
    show_progress(0.200000, 10);
    apply_sig(package_extract_file("sig/boot.sig"), "bootimg");

    Patching system image unconditionally...
    blockimg version is 2
      erasing 1048576 blocks
        blkdiscard failed: Invalid argument
      writing 521761 blocks of new data

    ioctl(): blank: Invalid argument
    write failed: No space left on device
    lseek64 failed: Invalid argument
    Updater process ended with ERROR: 1
    I:Legacy property environment disabled.
    I:Install took 274 second(s).
    I:Signaling child sideload process to exit.
    I:Waiting for child sideload process to exit.
    sideload_host finished

Postanowiłem poszukać informacji na ten temat i [użytkownicy @maxprzemo oraz @piskorfa z
forum.android.com.pl nakierowali mnie na właściwy trop][4]. Okazuje się bowiem, że trzeba będzie
się trochę napracować, by wgrać system ze stock'owego pliku `update.zip` .

Rozchodzi się o to, że aktualizacja czy też instalacja systemu z pliku `update.zip` w Androidach
począwszy od wersji 5.0 (Lollipop) [wykorzystuje zapis blokowy][5]. Ten mechanizm zatem będzie
próbował wgrać na partycję `/system/` obraz o pewnym określonym rozmiarze (w tym przypadku 4 GiB).
Nie można więc skrócić tej partycji do 2,5 GiB, tak jak to zrobiliśmy wyżej, tzn. można ale by
zainstalować system z pliku `update.zip` , trzeba go pierw przepakować i zmienić w nim informację
dotyczącą rozmiaru obrazu z tych 4 GiB na 2,5 GiB.

## Przepakowanie stock'owego obrazu system.img z pliku update.zip

Wiemy co mamy czynić, zatem do roboty. Przede wszystkim, musimy się zaopatrzyć w odpowiednie
narzędzia. W repozytorium dystrybucji Debian znajduje się pakiet `android-tools-fsutils` . W nim
zaś mamy narzędzia takie jakie `make_ext4fs` , `img2simg` oraz `simg2img` . Brakuje jednak dwóch
narzędzi, których będziemy również potrzebować, tj. [sdat2img][6] oraz [img2sdat][7]. Trzeba te dwa
skrypty dociągnąć z GitHub'a.

Dokładna [instrukcja przepakowania obrazu][8] znajduje się na forum XDA. Poniżej zaś znajduje się
skrócony zapis poszczególnych kroków.

### Wypakowanie zawartości pliku update.zip

Na sam początek pobieramy ze strony TP-LINK/Neffos plik firmware dla Neffos C5 lub C5 MAX w
zależności od tego, który z ww. modeli smartfona posiadamy. Ten plik trzeba wypakować. Zawartość
tego pliku po wypakowaniu jest następująca:

    # tree -sh img
    img
    ├── [4.0K]  META-INF
    │   ├── [1.5K]  CERT.RSA
    │   ├── [1.2K]  CERT.SF
    │   ├── [1.0K]  MANIFEST.MF
    │   └── [4.0K]  com
    │       ├── [4.0K]  android
    │       │   ├── [ 129]  metadata
    │       │   └── [1.4K]  otacert
    │       └── [4.0K]  google
    │           └── [4.0K]  android
    │               ├── [564K]  update-binary
    │               └── [ 738]  updater-script
    ├── [7.2M]  boot.img
    ├── [ 32K]  file_contexts
    ├── [ 60K]  mobicore.bin
    ├── [ 431]  scatter.txt
    ├── [4.0K]  sig
    │   ├── [ 256]  boot.sig
    │   └── [ 256]  recovery.sig
    ├── [2.0G]  system.new.dat
    ├── [   0]  system.patch.dat
    ├── [ 778]  system.transfer.list
    └── [   1]  type.txt

    6 directories, 17 files

Obraz partycji `/system/` znajduje się w pliku `system.new.dat` . Ten plik, jak widzimy wyżej,
zajmuje 2 GiB, a nie 4 GiB. Jest on zatem w jakiś sposób skompresowany i musimy go pierw
zdekompresować.

### Dekompresja pliku system.new.dat do surowego obrazu EXT4

Do dekompresji pliku `system.new.dat` posłuży nam skrypt `sdat2img.py` . Przyjmuje on trzy
argumenty, z których pierwszy to ścieżka do pliku `system.transfer.list` , drugi to ścieżka do pliku
`system.new.dat` , z trzeci na nazwa surowego obrazu zawierającego już system plików EXT4, który
zostanie utworzony. Wpisujemy zatem w terminal poniższe polecenie:

    $ ./sdat2img.py img/system.transfer.list img/system.new.dat system.img
    sdat2img binary - version: 1.0

    Android Lollipop 5.1 detected!

    Skipping command erase...
    Copying 32770 blocks into position 0...
    Copying 2 blocks into position 33025...
    Copying 31996 blocks into position 33539...
    ...
    Copying 2 blocks into position 983040...
    Copying 2 blocks into position 1015808...
    Done! Output image: /media/Kabi/neffos/system.img

Po chwili w katalogu roboczym powinniśmy ujrzeć plik `system.img` :

    # ls -al system.img
    -rw-r--r-- 1 root root 4.0G 2017-04-03 09:29:38 system.img

Jak widać, z 2 GiB zrobiło nam się 4 GiB i to jest właśnie surowy obraz, który można wgrać na
partycję `/system/` , gdy ta ma rozmiar 4 GiB. My ten rozmiar zmieniliśmy do 2,5 GiB i dlatego
trzeba ten obraz nieco przyciąć.

### Tworzenie surowego obrazu EXT4 o mniejszym rozmiarze

Musimy teraz wydobyć pliki ze starego obrazu i z nich wygenerować nowy surowy obraz EXT4. Wobec tego
montujemy ten powyższy plik `system.img` w jakimś katalogu:

    # mount -t ext4 -o loop system.img /mnt

Przy pomocy narzędzia `make_ext4fs` tworzymy obraz o określonym rozmiarze z zawartości katalogu, do
którego podmontowaliśmy wcześniej obraz `system.img` :

    # make_ext4fs -T 0 -S img/file_contexts -l 2684354560 -a system system_new.img /mnt/
    Creating filesystem with parameters:
        Size: 2684354560
        Block size: 4096
        Blocks per group: 32768
        Inodes per group: 8192
        Inode size: 256
        Journal blocks: 10240
        Label:
        Blocks: 655360
        Block groups: 20
        Reserved block group size: 159
    Created filesystem with 2788/163840 inodes and 526016/655360 blocks

W parametrze `-S` musimy podać ścieżkę do pliku `file_contexts` . Natomiast opcja `-l` odpowiada za
rozmiar (w bajtach) nowo wygenerowanego obrazu (2,5 GiB). Po zakończeniu procesu pakowania
powinniśmy otrzymać plik `system_new.img` o zmniejszonym rozmiarze:

    # ls -al system_new.img
    -rw-r--r-- 1 root root 2.5G 2017-04-03 00:30:59 system_new.img

No i nasz obraz ma już taki rozmiar jaki chcemy. Ten plik jeszcze trzeba skompresować.

### Kompresja surowego obrazu EXT4

Przy pomocy narzędzia `img2simg` kompresujemy wyżej utworzony obraz systemu:

    # img2simg system_new.img system_new.img-sparse

    # ls -al system_new.img-sparse
    -rw-r--r-- 1 root root 2.0G 2017-04-03 09:43:00 system_new.img-sparse

### Konwersja do pliku DAT

Tak powstały plik `system_new.img-sparse` trzeba jeszcze przekonwertować do formatu DAT. Do tego
celu wykorzystamy skrypt `img2sdat.py` podając w argumencie ścieżkę do skompresowanego obrazu:

    # ./img2sdat/img2sdat.py system_new.img-sparse
    img2sdat binary - version: 1.2

                1. Android Lollipop 5.0
                2. Android Lollipop 5.1
                3. Android Marshmallow 6.0
                4. Android Nougat 7.0

    Choose system version: 2
    Total of 655360 4096-byte output blocks in 2605 input chunks.
    Generating digraph...
    Finding vertex sequence...
    Reversing backward edges...
      0/0 dependencies (0.00%) were violated; 0 source blocks stashed.
    Improving vertex order...
    Reticulating splines...
    max stashed blocks: 0  (0 bytes), limit:

    Done! Output files: .

W katalogu roboczym powinny zostać utworzone trzy nowe pliki: `system.new.dat` , `system.patch.dat`
oraz `system.transfer.list` :

    # ls -al
    -rw-r--r-- 1 root   root   2.0G 2017-04-03 09:54:11 system.new.dat
    -rw-r--r-- 1 root   root      0 2017-04-03 09:54:11 system.patch.dat
    -rw-r--r-- 1 root   root    43K 2017-04-03 09:54:11 system.transfer.list

### Tworzenie nowego pliku update.zip

Te trzy pliki trzeba podmienić w katalogu z zawartością, która została wyodrębniona z pliku
`update.zip` :

    # mv system.new.dat system.patch.dat system.transfer.list ./img/

Następnie całość trzeba spakować:

    # cd ./img/
    # zip -r update.zip *
      adding: META-INF/ (stored 0%)
      adding: META-INF/CERT.RSA (deflated 25%)
      adding: META-INF/MANIFEST.MF (deflated 46%)
      adding: META-INF/com/ (stored 0%)
      adding: META-INF/com/google/ (stored 0%)
      adding: META-INF/com/google/android/ (stored 0%)
      adding: META-INF/com/google/android/update-binary (deflated 35%)
      adding: META-INF/com/google/android/updater-script (deflated 56%)
      adding: META-INF/com/android/ (stored 0%)
      adding: META-INF/com/android/metadata (deflated 6%)
      adding: META-INF/com/android/otacert (deflated 28%)
      adding: META-INF/CERT.SF (deflated 51%)
      adding: boot.img (deflated 0%)
      adding: file_contexts (deflated 80%)
      adding: mobicore.bin (deflated 57%)
      adding: scatter.txt (deflated 52%)
      adding: sig/ (stored 0%)
      adding: sig/recovery.sig (stored 0%)
      adding: sig/boot.sig (stored 0%)
      adding: system.new.dat (deflated 52%)
      adding: system.patch.dat (stored 0%)
      adding: system.transfer.list (deflated 71%)
      adding: type.txt (stored 0%)

W ten sposób przygotowany plik `update.zip` nadaje się do wgrania przez ADB Sideload ale tylko i
wyłącznie z poziomu TWRP. Chodzi o to, że firmware wypuszczany przez TP-LINK/Neffos jest podpisany
w przeciwieństwie do naszej
paczki:

![](/img/2017/04/003.tp-link-neffos-c5-max-flash-repartycjonowanie-update.png#huge)

My nie możemy wykorzystać klucza, którym takie archiwum zostało podpisane i nasza paczka nie będzie
honorowana przez mechanizm aktualizacji. Na szczęście w ustawieniach TWRP można pominąć weryfikację
pliku ZIP i wgrać system bez względu na to czy firmware jest podpisany czy też i nie jest.

## Wgrywanie świeżego systemu na partycję /system/

Wracamy na smartfon i włączamy tryb ADB Sideload z menu TWRP (Advanced => ADB Sideload). W
terminalu na komputerze zaś wydajemy poniższe polecenie:

    # adb sideload update.zip
    Total xfer: 1.00x

Z logu TWRP możemy wyczytać, że ta operacja zakończyła się powodzeniem:

    sideload-host file size 995870531 block size 65536
    Installing zip file '/sideload/package.zip'
    I:Update binary zip
    I:Zip contains SELinux file_contexts file in its root. Extracting to /file_contexts
    I:Legacy property environment initialized.

    ====== Updater-Script:
    getprop("ro.product.device") == "C5" || abort("This package is for \"C5\" devices; this is a \"" + getprop("ro.product.device") + "\".");
    show_progress(0.750000, 0);
    ui_print("Patching system image unconditionally...");
    block_image_update("system", package_extract_file("system.transfer.list"), "system.new.dat", "system.patch.dat");
    show_progress(0.050000, 5);
    assert(package_extract_file("boot.img", "/tmp/boot.img"),
           write_raw_image("/tmp/boot.img", "bootimg"),
           delete("/tmp/boot.img"));
    assert(package_extract_file("mobicore.bin", "/tmp/tee1.img"),
           write_raw_image("/tmp/tee1.img", "tee1"),
           delete("/tmp/tee1.img"));
    show_progress(0.200000, 10);
    apply_sig(package_extract_file("sig/boot.sig"), "bootimg");


    Patching system image unconditionally...
    blockimg version is 2
      writing 512 blocks of new data
      writing 512 blocks of new data
      writing 512 blocks of new data
    ...
      zeroing 1024 blocks
      zeroing 1024 blocks
      zeroing 1024 blocks
    ...
      writing 512 blocks of new data
      writing 512 blocks of new data
      writing 512 blocks of new data
    ...
    wrote 655360 blocks; expected 655360
    max alloc needed was 4096
    kernel_page_cnt:3117 ramdisk_page_cnt:565 second_page_cnt:0 partion_end_offset:0x731800
    script result was [pass]
    I:Updater process ended with RC=0
    I:Legacy property environment disabled.
    I:Install took 357 second(s).
    I:Signaling child sideload process to exit.
    I:Waiting for child sideload process to exit.
    sideload_host finished

Czyścimy cache i robimy Factory Reset, by potwierdzić, że z pozostałymi partycjami nie ma żadnych
problemów:

![](/img/2017/04/004.tp-link-neffos-c5-max-flash-repartycjonowanie-factory-reset-twrp.png#small)

Cały proces ze zmianą układu flash'a w smartfonach Neffos C5 i C5 MAX nie był jakoś szczególnie
skomplikowany ale takie przepakowywanie pliku `update.zip` nie należy do przyjemnych. Na dobrą
sprawę, każdy plik `update.zip` z nowszym firmware wypuszczony przez TP-LINK/Neffos, który będziemy
chcieli wgrać na smartfon przy pomocy ADB Sideload, musimy w powyższy sposób przepakować. Oczywiście
aktualizacje OTA Update będą działać bez zarzutu ale w przypadku uszkodzenia w jakiś sposób systemu
telefonu, warto taką przerobioną paczkę `.zip` posiadać.

## Alternatywna metoda na wgranie obrazu system.img

Ponowne instalowanie systemu z wykorzystaniem mechanizmu ADB Sideload nie jest jednak jedynym
rozwiązaniem, które możemy wykorzystać, by przywrócić system w telefonie. Ten sposób jest nieco
prostszy ale opiera się na backup'ie stock'owej partycji `/system/` , który można wykonać albo przez
SP Flash Tool, albo też z poziomu TWRP recovery przy pomocy narzędzia `dd` . Niezależnie od sposobu,
na dysku powinniśmy mieć surowy obraz partycji `/system/` o rozmiarze 4 GiB. Rozmiar tego pliku
trzeba przyciąć do 2,5 GiB i wgrać na smartfon.

### Jak przyciąć stock'owy obraz system.img

Wyciągnijmy sobie najpierw obraz `system.img` z backup'u flash'a. Montujemy obraz przy pomocy
poniższego polecenia (być może trzeba będzie [dostosować moduł loop][9]):

    # losetup /dev/loop0 tp-link-neffos-c5-orig.img

Upewniamy się co do numeru partycji `/system/` w tym obrazie:

    # gdisk -l tp-link-neffos-c5-orig.img | grep system
      20          360448         8749055   4.0 GiB     0700  system

Kopiujemy zawartość tej partycji na dysk przy pomocy `dd` :

    # dd if=/dev/loop0p20 of=./c5-p20-system.orig

Sprawdzamy system plików pod kątem błędów i zmieniamy rozmiar systemu plików w obrazie na 2,5 GiB:

    # e2fsck -f c5-p20-system.orig

    # resize2fs -p c5-p20-system.orig 2560M
    resize2fs 1.43.4 (31-Jan-2017)
    Resizing the filesystem on c5-p20-system.orig to 655360 (4k) blocks.
    Begin pass 2 (max = 147513)
    Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 3 (max = 32)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on c5-p20-system.orig is now 655360 (4k) blocks long.

Po zakończonym procesie, rozmiar pliku `c5-p20-system.orig` powinien wynosić 2,5 GiB:

    # ls -al c5-p20-system.orig
    -rw-r--r-- 1 morfik morfik 2.5G 2017-04-03 02:08:18 c5-p20-system.orig

Ten obraz trzeba teraz załadować na smartfon, z tym, że jeśli chcemy do tego celu wykorzystać SP
Flash Tool, to musimy odpowiednio przepisać sobie plik `scatter.txt` . Możemy też naturalnie ten
obraz przesłać przez ADB na partycję `/data/` i wgrać go przy pomocy `dd` na stosowne urządzenie
blokowe. Sposób z SP Flash Tool jest nieco mniej inwazyjny, bo nie trzeba zapisywać dwa razy pamięci
telefonu. Którą opcję wybierzemy, to już zależy w zasadzie tylko od nas.

### Wgrywanie przyciętego obrazu przez SP Flash Tool

Narzędzie SP Flash Tool operuje na pliku `scatter.txt` . Jeśli wykonaliśmy backup flash'a przy
pomocy tego oprogramowania, to [powinniśmy mieć ten plik][10]. Niestety nie możemy go wykorzystać w
momencie, gdy zmieniamy układ partycji na flash. W zasadzie, to musimy w tym pliku przepisać pozycje
odpowiadające za opis partycji `/system/` , `/cache/` oraz `/data/` , bo to one uległy zmianie.
Przepisujemy te pozycje do poniższej postaci (zawsze możemy pomagać sobie plikiem
`/proc/partinfo` ):

    ...
    - partition_index: SYS21
      partition_name: system
      file_name:
      is_download: true
      type: EXT4_IMG
      linear_start_addr: 0xb000000
      physical_start_addr: 0xb000000
      partition_size: 0xa0000000
      region: EMMC_USER
      storage: HW_STORAGE_EMMC
      boundary_check: true
      is_reserved: false
      operation_type: UPDATE
      reserve: 0x00

    - partition_index: SYS22
      partition_name: cache
      file_name:
      is_download: true
      type: EXT4_IMG
      linear_start_addr: 0xab000000
      physical_start_addr: 0xab000000
      partition_size: 0x20000000
      region: EMMC_USER
      storage: HW_STORAGE_EMMC
      boundary_check: true
      is_reserved: false
      operation_type: UPDATE
      reserve: 0x00

    - partition_index: SYS23
      partition_name: userdata
      file_name:
      is_download: true
      type: EXT4_IMG
      linear_start_addr: 0xcb000000
      physical_start_addr: 0xcb000000
      partition_size: 0x2d7d80000
      region: EMMC_USER
      storage: HW_STORAGE_EMMC
      boundary_check: true
      is_reserved: false
      operation_type: UPDATE
      reserve: 0x00
    ...

Zapisujemy plik `scatter.txt` i ładujemy go w SP Flash Tool. Wskazujemy także plik obrazu dla
partycji `/system/` no i oczywiście wciskamy przycisk
"Download":

![](/img/2017/04/005.tp-link-neffos-c5-max-flash-repartycjonowanie-sp-flash-tool.png#huge)

### Wgrywanie przyciętego obrazu przez TWRP recovery

Odpalamy smartfon w trybie recovery (TWRP) i upewniamy się, że partycja `/system/` nie jest
zamontowana oraz, że partycja `/data/` jest zamontowana. W terminalu na komputerze wpisujemy
poniższe polecenie:

    # adb push c5-p20-system.orig /data/

Czekamy, aż plik zostanie przesłany. Następnie wgrywamy ten obraz na urządzenie blokowe
odpowiadające za partycję `/system/` :

    # adb shell

    ~ # dd if=/data/c5-p20-system.orig of=/dev/block/bootdevice/by-name/system
    5242880+0 records in
    5242880+0 records out
    2684354560 bytes (2.5GB) copied, 530.597107 seconds, 4.8MB/s

## Sprawdzenie nowego układu partycji z poziomu Androida

Po wgraniu systemu na smartfon dowolną metodą, restartujemy urządzenie. Telefon będzie się
uruchamiał dłuższą chwilę ale ostatecznie powinniśmy zobaczyć ekran wyboru języka. Przechodzimy
naturalnie przez proces wstępnej konfiguracji telefonu. Następnie sprawdzamy już tylko w ramach
formalności czy system widzi poprawną ilość miejsca na partycji `/system/` , `/cache/` i `/data/` ,
np. w [aplikacji Diskinfo][11]:

![](/img/2017/04/006.tp-link-neffos-c5-max-flash-repartycjonowanie-diskinfo.png#huge)

Jak widać repartycjonowanie flash'a w smartfonach Neffos C5 i C5 MAX wyposażonych w system Android w
wersji 5.1 (Lollipop) nie obyło się bez komplikacji. Grunt jednak, że smartfon nie ma problemów już
z nowym układem flash'a, a my odzyskaliśmy 1,5 GiB, które możemy sobie dodatkowo przeznaczyć na dane
użytkownika.


[1]: http://tplink-forum.pl/index.php?/topic/5854-jak-przeprowadzi%C4%87-root-na-smartfonach-neffos-od-tp-link/#comment-49408
[2]: http://spflashtool.com/
[3]: /post/android-wgrywanie-update-zip-przez-adb-sideload-via-twrp-recovery/
[4]: http://forum.android.com.pl/topic/313401-na-jakiej-zasadzie-dzia%C5%82a-adb-sideload/
[5]: https://source.android.com/devices/tech/ota/block
[6]: https://github.com/xpirt/sdat2img/blob/master/sdat2img.py
[7]: https://github.com/xpirt/img2sdat/blob/master/img2sdat.py
[8]: https://forum.xda-developers.com/android/software-hacking/how-to-conver-lollipop-dat-files-to-t2978952
[9]: /post/obsluga-wielu-partycji-w-module-loop/
[10]: http://tplink-forum.pl/pub/neffos/
[11]: https://play.google.com/store/apps/details?id=me.kuder.diskinfo
