---
author: Morfik
categories:
- Linux
date:    2023-12-10 22:00:00 +0100
lastmod: 2023-12-10 22:00:00 +0100
published: true
status: publish
tags:
- debian
- ssd
- trim
- discard
- luks
- lvm
GHissueID: 599
title: Trim/discard przy LUKS/LVM na dysku SSD pod Debian linux
---

Z okazji zbliżającego się końca roku, postanowiłem nieco ogarnąć swojego Debiana, tj. postawić go
na nowo. Jakby nie patrzeć 4 lata korzystania z tego linux'a z włączonymi gałęziami unstable i
experimental sprawiło, że trochę syfu się nazbierało. Nie chciałem też czyścić całego kontenera
LUKS czy samej struktury LVM z systemowymi voluminami logicznymi na starym dysku HDD, bo
zainstalowany tam system zawsze może się do czegoś przydać, np. do odratowania tego nowego linux'a.
Dlatego też postanowiłem zakupić niedrogi dysk SSD (MLC, używany) i to na nim [postawić świeżego
Debiana z wykorzystaniem narzędzia debootstrap][2]. Sama instalacja linux'a na dysku SSD nie różni
się zbytnio od instalacji na dysku HDD, za wyjątkiem skonfigurowania w takim systemie mechanizmu
trim/discard. Standardowi użytkownicy linux'a nie muszą zbytnio nic robić, aby ten mechanizm został
poprawnie skonfigurowany. Sprawa się nieco komplikuje, gdy wykorzystywany jest [device-mapper][3],
który mapuje fizyczne bloki urządzenia na te wirtualne, np. przy szyfrowaniu dysku z wykorzystaniem
LUKS/dm-crypt czy korzystaniu z voluminów logicznych LVM. Dlatego też postanowiłem przyjrzeć się
nieco bliżej zagadnieniu konfiguracji mechanizmu trim/discard na dysku SSD w przypadku
zaszyfrowanego systemu na bazie LUKS+LVM.

<!--more-->
## Czy kupowanie używanego dysku SSD to dobry pomysł

We wstępie wspomniałem, że dysk SSD, który zakupiłem, nie był nowy ino używany. Konkretnie jest to
[model IR-SSDPR-S25A-240 od Goodram][1], czyli dysk o pojemności 240GB, którego komórki flash są
wykonane w technologi MLC. Dlaczego zdecydowałem się na dysk używany, a nie nowy? Z kilku powodów.

Ten używany dysk SSD kosztował w granicach 50zł, gdzie orientacyjna cena nowego urządzenia (tego
samego modelu) waha się w granicach 250zł. Druga sprawa, to raport SMART, który został opublikowany
przez sprzedającego, a na który ja przed zakupem bardzo lubię sobie popatrzeć. W oparciu o ten
raport podejmuję decyzję czy dysk zakupić.

### Raport SMART i parametr Lifetime_Writes_GiB

Poniżej znajduje się pełny raport SMART tego dysku SSD:

    # smartctl -x /dev/sdb
    smartctl 7.4 2023-08-01 r5530 [x86_64-linux-6.6.1-amd64] (local build)
    Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF INFORMATION SECTION ===
    Model Family:     Phison Driven SSDs
    Device Model:     IR-SSDPR-S25A-240
    Serial Number:    GUX033073
    Firmware Version: SBFM91.3
    User Capacity:    240,057,409,536 bytes [240 GB]
    Sector Size:      512 bytes logical/physical
    Rotation Rate:    Solid State Device
    Form Factor:      2.5 inches
    TRIM Command:     Available
    Device is:        In smartctl database 7.3/5528
    ATA Version is:   ACS-4 (minor revision not indicated)
    SATA Version is:  SATA 3.2, 6.0 Gb/s (current: 6.0 Gb/s)
    Local Time is:    Fri Dec  8 09:47:18 2023 CET
    SMART support is: Available - device has SMART capability.
    SMART support is: Enabled
    AAM feature is:   Unavailable
    APM feature is:   Unavailable
    Rd look-ahead is: Enabled
    Write cache is:   Enabled
    DSN feature is:   Unavailable
    ATA Security is:  Disabled, NOT FROZEN [SEC1]
    Wt Cache Reorder: Unavailable

    === START OF READ SMART DATA SECTION ===
    SMART overall-health self-assessment test result: PASSED

    General SMART Values:
    Offline data collection status:  (0x00) Offline data collection activity
                                            was never started.
                                            Auto Offline Data Collection: Disabled.
    Self-test execution status:      (   0) The previous self-test routine completed
                                            without error or no self-test has ever
                                            been run.
    Total time to complete Offline
    data collection:                (65535) seconds.
    Offline data collection
    capabilities:                    (0x79) SMART execute Offline immediate.
                                            No Auto Offline data collection support.
                                            Suspend Offline collection upon new
                                            command.
                                            Offline surface scan supported.
                                            Self-test supported.
                                            Conveyance Self-test supported.
                                            Selective Self-test supported.
    SMART capabilities:            (0x0003) Saves SMART data before entering
                                            power-saving mode.
                                            Supports SMART auto save timer.
    Error logging capability:        (0x01) Error logging supported.
                                            General Purpose Logging supported.
    Short self-test routine
    recommended polling time:        (   2) minutes.
    Extended self-test routine
    recommended polling time:        (  30) minutes.
    Conveyance self-test routine
    recommended polling time:        (   6) minutes.

    SMART Attributes Data Structure revision number: 16
    Vendor Specific SMART Attributes with Thresholds:
    ID# ATTRIBUTE_NAME          FLAGS    VALUE WORST THRESH FAIL RAW_VALUE
      1 Raw_Read_Error_Rate     PO-R--   100   100   050    -    0
      9 Power_On_Hours          -O--C-   100   100   000    -    5738
     12 Power_Cycle_Count       -O--C-   100   100   000    -    820
    168 SATA_Phy_Error_Count    -O--C-   100   100   000    -    0
    170 Bad_Blk_Ct_Lat/Erl      PO----   100   100   010    -    0/370
    173 MaxAvgErase_Ct          -O--C-   100   100   000    -    135 (Average 85)
    192 Unsafe_Shutdown_Count   -O--C-   100   100   000    -    27
    194 Temperature_Celsius     PO---K   067   067   000    -    33 (Min/Max 33/33)
    218 CRC_Error_Count         PO-R--   100   100   050    -    0
    231 SSD_Life_Left           PO--C-   100   100   000    -    97
    241 Lifetime_Writes_GiB     -O--C-   100   100   000    -    9037
                                ||||||_ K auto-keep
                                |||||__ C event count
                                ||||___ R error rate
                                |||____ S speed/performance
                                ||_____ O updated online
                                |______ P prefailure warning

    General Purpose Log Directory Version 1
    SMART           Log Directory Version 1 [multi-sector log support]
    Address    Access  R/W   Size  Description
    0x00       GPL,SL  R/O      1  Log Directory
    0x01           SL  R/O      1  Summary SMART error log
    0x02           SL  R/O     51  Comprehensive SMART error log
    0x03       GPL     R/O     64  Ext. Comprehensive SMART error log
    0x04       GPL,SL  R/O      8  Device Statistics log
    0x06           SL  R/O      1  SMART self-test log
    0x07       GPL     R/O      1  Extended self-test log
    0x09           SL  R/W      1  Selective self-test log
    0x10       GPL     R/O      1  NCQ Command Error log
    0x11       GPL     R/O      1  SATA Phy Event Counters log
    0x30       GPL,SL  R/O      9  IDENTIFY DEVICE data log
    0x80-0x9f  GPL,SL  R/W     16  Host vendor specific log

    SMART Extended Comprehensive Error Log Version: 1 (64 sectors)
    No Errors Logged

    SMART Extended Self-test Log Version: 1 (1 sectors)
    Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
    # 1  Extended offline    Completed without error       00%      5714         -

    SMART Selective self-test log data structure revision number 0
    Note: revision number not 1 implies that no selective self-test has ever been run
     SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
        1        0        0  Not_testing
        2        0        0  Not_testing
        3        0        0  Not_testing
        4        0        0  Not_testing
        5        0        0  Not_testing
    Selective self-test flags (0x0):
      After scanning selected spans, do NOT read-scan remainder of disk.
    If Selective self-test is pending on power-up, resume after 0 minute delay.

    SCT Commands not supported

    Device Statistics (GP Log 0x04)
    Page  Offset Size        Value Flags Description
    0x01  =====  =               =  ===  == General Statistics (rev 1) ==
    0x01  0x008  4             820  ---  Lifetime Power-On Resets
    0x01  0x010  4            5738  ---  Power-on Hours
    0x01  0x018  6     18953043829  ---  Logical Sectors Written
    0x01  0x028  6     26744769960  ---  Logical Sectors Read
    0x04  =====  =               =  ===  == General Errors Statistics (rev 1) ==
    0x04  0x008  4               0  ---  Number of Reported Uncorrectable Errors
    0x05  =====  =               =  ===  == Temperature Statistics (rev 1) ==
    0x05  0x008  1              33  ---  Current Temperature
    0x05  0x020  1              33  ---  Highest Temperature
    0x05  0x028  1              33  ---  Lowest Temperature
    0x06  =====  =               =  ===  == Transport Statistics (rev 1) ==
    0x06  0x008  4            2890  ---  Number of Hardware Resets
    0x06  0x018  4               0  ---  Number of Interface CRC Errors
    0x07  =====  =               =  ===  == Solid State Device Statistics (rev 1) ==
    0x07  0x008  1               2  ---  Percentage Used Endurance Indicator
                                    |||_ C monitored condition met
                                    ||__ D supports DSN
                                    |___ N normalized value

    Pending Defects log (GP Log 0x0c) not supported

    SATA Phy Event Counters (GP Log 0x11)
    ID      Size     Value  Description
    0x0001  2            0  Command failed due to ICRC error
    0x0003  2            0  R_ERR response for device-to-host data FIS
    0x0004  2            0  R_ERR response for host-to-device data FIS
    0x0006  2            0  R_ERR response for device-to-host non-data FIS
    0x0007  2            0  R_ERR response for host-to-device non-data FIS
    0x0008  2            0  Device-to-host non-data FIS retries
    0x0009  4          134  Transition from drive PhyRdy to drive PhyNRdy
    0x000a  4            2  Device-to-host register FISes sent due to a COMRESET
    0x000f  2            0  R_ERR response for host-to-device data FIS, CRC
    0x0010  2            0  R_ERR response for host-to-device data FIS, non-CRC
    0x0012  2            0  R_ERR response for host-to-device non-data FIS, CRC
    0x0013  2            0  R_ERR response for host-to-device non-data FIS, non-CRC

To co nas interesuje najbardziej w tym raporcie SMART, to parametr `241 Lifetime_Writes_GiB` ,
czyli ilość danych, które użytkownik zapisał na takim nośniku od momentu wyciągnięcia tego dysku z
pudełka do chwili, gdy trafił on w nasze łapki. Czasami ta wartość może być zakodowana w HEX ale
tutaj mamy wartość dziesiętną ( `18953043829 Logical Sectors Written` ), czyli
(18953043829×512)/1024/1024/1024= `9037` ).

Patrząc na wspomniany parametr, widzimy wartość `9037` , czyli około 9 TiB. Dodatkowo z parametru
`Power_On_Hours` (czas pracy w godzinach) i `Power_Cycle_Count` (ilość cykli zasilania) możemy
wyciągnąć nieco informacji o wykorzystaniu dysku do momentu jego sprzedaży przez właściciela.
Wartość `5738` w `Power_On_Hours` odpowiada 239 przepracowanym dniom, a gdy podzielimy tę wartość
przez 820, to otrzymamy około 7 godzin na cykl zasilania. Jeśli teraz podzielimy ilość zapisanych
danych przez ilość przepracowanych godzin, to wychodzi nam średnio około 1,5 GiB zapisanych danych
na godzinę. Z kolei jeśli podzielimy ilość zapisanych danych przez ilość cykli zasilania, to
dostaniemy około 11 GiB (można przyjąć, że poprzedni użytkownik tego dysku SSD zapisywał około 11
GiB dziennie). Czy zapisanie 11 GiB na dzień to dużo czy mało?

Zaglądając na stronę producenta, możemy doszukać się wiadomości, że parametr `TBW` (możliwa ilość
zapisanych danych w TB), po przekroczeniu którego produkt już nie podlega gwarancji ma wartość
`180` . Jest to 180 TB, czyli jakieś 172 TiB i tyle komórki flash tego dysku SSD powinny wytrzymać.
Zapisane 9 TiB stanowi około 5%, zatem komórki dysku nie są jakoś szczególnie zajechane cyklami
(130 GiB dołożyłem instalując i przenosząc stary system na ten dysk SSD). Gdyby przyjąć, że
użytkownik będzie zapisywał na tym dysku 100 GiB dziennie, to ten nośnik posłuży mu jeszcze przez
co najmniej 4,5 roku, choć trzeba tutaj jeszcze wyraźnie zaznaczyć, że po takim czasie
wyrobilibyśmy jedynie gwarantowaną normę danych do zapisu, co nie oznacza, że dysk nam od razu
przestanie działać.

### Czy dysk zawiera błędy

Drugą ważną sprawą jest przeskanowanie dysku i sprawdzenie czy są na nim jakieś błędy. W tym
przypadku błędów brak:

    # smartctl -l selftest /dev/sdb
    smartctl 7.4 2023-08-01 r5530 [x86_64-linux-6.6.1-amd64] (local build)
    Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF READ SMART DATA SECTION ===
    SMART Self-test log structure revision number 1
    Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
    # 1  Extended offline    Completed without error       00%      5714         -

    # smartctl -l error /dev/sdb
    smartctl 7.4 2023-08-01 r5530 [x86_64-linux-6.6.1-amd64] (local build)
    Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF READ SMART DATA SECTION ===
    SMART Error Log Version: 1
    No Errors Logged

Mając na uwadze powyższe informacje, 50zł za taki dysk SSD wydaje się być dobrą ceną, biorąc pod
uwagę fakt, że jest on w stanie znacznie przyśpieszyć pracę i responsywność naszego linux'a.

## Różnica między discard, trim i fstrim

Szukając informacji na temat konfiguracji dysku SSD pod linux, można natknąć się na dwa lub trzy
terminy, tj. `trim` , `fstrim` oraz `discard` . Z początku myślałem, że są to osobne rzeczy
ale [okazuje się, że trim, fstrim i discard odnoszą się do tego samego mechanizmu][4] i użytkownicy
z reguły stosują te pojęcia zamiennie. Dlatego dobrze byłoby doprecyzować czym jest `trim` , `fstrim`
oraz `discard` .

Generalnie `discard` to opcja montowania systemu plików i po jej ustawieniu firmware dysku jest
powiadamiany o blokach, które już nie są wykorzystywane przez ten konkretny system plików. Efektem
ustawienia opcji `discard` dla takiego systemu plików jest zerowanie bloków fizycznych dysku
praktycznie natychmiast po usunięciu jakiegoś pliku. Z kolei `trim` (albo bardziej precyzyjnie
polecenie `fstrim` ) robi dokładnie to samo ale nie w chwili, w której operacja usuwania danych ma
miejsce. Czyli zarówno opcja `discard` jak i `trim`/`fstrim` informują urządzenie blokowe o
przestrzeni, która była używana przez system plików ale została zwolniona i wróciła z powrotem do
wolnej puli.

Takie powiadamianie o zwolnionych blokach nie jest niczym niezwykłym, bo urządzenia oparte o
technologię flash muszą pierw wyzerować komórki zanim będzie możliwy w nich ponowny zapis
faktycznych danych. Ten proces zerowania jest jednak bardzo powolny w porównaniu do operacji zapisu
realnych danych. Jeśli teraz weźmiemy dysk SSD bez opcji `discard`, który był wcześniej zapisany w
całości i będziemy chcieli zapisać w nim jakieś dane, to określone komórki flash będzie trzeba
uprzednio wyzerować i dopiero po zakończeniu tego procesu zapisać te interesujące nas dane. Widać
zatem, że dysk może mieć problemy z wydajnością w takim przypadku, bo nie zapisuje od razu tych
danych, które byśmy sobie życzyli. Dlatego też by zachować dobrą wydajność przy zapisie danych na
dysku SSD trzeba korzystać z opcji `discard` lub cyklicznie wydawać polecenie `fstrim` .

### Co lepsze trim/fstrim czy discard

Zarówno `trim` (wywoływanie polecenia `fstrim` ) jak i opcja montowania `discard` mogą negatywnie
odbić się na wydajności, jak i żywotności dysku SSD. Nie powinniśmy zatem korzystać z opcji
`discard` , np. w pliku `/etc/fstab` . Z kolei jeśli chodzi o `trim` , to nie powinno się korzystać
z polecenia `fstrim` [częściej niż raz w tygodniu][5].

Dodatkowo dyski SSD mogą wspierać coś co się nazywa `queued trim` , czyli w określonych interwałach
czasowych lub przy określonych zdarzeniach, dysk SSD będzie automatycznie czyścił wolne bloki, tak
by te operacje trim/discard nie kolidowały z operacjami zapisu/odczytu danych na dysku. Jeśli nasz
dysk nie wspiera `queued trim` , to stosowanie opcji `discard` [może znacznie degradować wydajność
dysku SSD][15]. Ten sytuacja jest na tyle uporczywa, że deweloperzy kernela wpisują na [czarną
listę][16] ( `ata_device_blacklist` ) takie dyski SSD, by jakoś radzić sobie z tym problemem.

## Czy mój dysk SSD wspiera trim/discard

Wsparcie dla trim/discard można odczytać z raportu SMART lub z wyjścia polecenia `hdparm`:

    #  smartctl -x  /dev/sdb | grep -i trim
    TRIM Command:     Available

    # hdparm -I /dev/sdb | grep -i trim
               *    Data Set Management TRIM supported (limit 8 blocks)

Trzeba tutaj wyraźnie zaznaczyć, że fakt wspierania trim/discard przez dysk SSD nie oznacza
automatycznie, że partycje utworzone w obrębie takiego dysku również będą posiadały wsparcie dla
tego mechanizmu. Możemy się o tym fakcie przekonać wydając poniższe polecenie:

    # lsblk --discard /dev/sdb
    NAME                        DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
    sdb                                0      512B       2G         0
    ├─sdb1                             0      512B       2G         0
    ├─sdb2                             0      512B       2G         0
    └─sdb3                             0      512B       2G         0
      └─debian_crypt                   0        0B       0B         0
        ├─goodram_ssd-ccache           0        0B       0B         0
        ├─goodram_ssd-debuilder        0        0B       0B         0
        ├─goodram_ssd-root             0        0B       0B         0
        └─goodram_ssd-home             0        0B       0B         0

Jak widać, pozycje `/dev/sdb[1-3]` posiadają wsparcie dla trim/discard z tym, że na `/dev/sdb3` mamy
zaszyfrowany kontener LUKS, a w nim LVM z 4 voluminami logicznymi. Patrząc po kolumnie `DISC-GRAN`
(discard granularity) oraz `DISC-MAX` (discard max bytes) możemy stwierdzić, że dysk `/dev/sdb`
oraz jego partycje `/dev/sdb[1-3]` wspierają trim/discard ale kontener LUKS i dyski logiczne LVM
już nie.

Jeśli w takim przypadku, jak wyżej, wydamy polecenie `fstrim`, to uzyskamy poniższy wynik:

    # fstrim -v /efi
    /efi: 501.6 MiB (526008320 bytes) trimmed

    # fstrim -v /boot
    /boot: 1.8 GiB (1928638464 bytes) trimmed

    # fstrim -v /home
    fstrim: /home: the discard operation is not supported

    # fstrim -v /
    fstrim: /: the discard operation is not supported

Zatem, niby mamy dysk wspierający trim/discard , to jednak część partycji/dysków logicznych nie
wspiera tego mechanizmu. Dlaczego?

### Problematyczny device-mapper (LUKS/LVM)

Zgodnie z tym co można [wyczytać tutaj][6], developerzy device-mapper nie włączą mechanizmu
trim/discard dla zaszyfrowanych voluminów ze względów bezpieczeństwa. Wypadałoby w tym miejscu
sobie zadać pytanie o te względy bezpieczeństwa, tj. jakie dokładnie informacje mogą zostać
ujawnione, gdy włączymy trim/discard dla zaszyfrowanego kontenera LUKS/dm-crypt. [Odpowiedź można
znaleźć w tym artykule][7].

Po przeczytaniu tego posta naszły mnie mieszane uczucia. Argumenty przemawiające za tym, by nie
włączać domyślnie trim/discard dla LUKS/dm-crypt są takie, że będzie wiadomo, w którym miejscu na
dysku twardym są przechowywane jakieś dane, a które miejsce zostało wyzerowane i nie zawiera
danych. Ponadto, może wyciec informacja z jakiego systemu plików korzystamy na tym urządzeniu.
Można też ustalić czy w obrębie danego wycinku dysku mamy ukryty jakiś zaszyfrowany volumin. No
tak, z czym tu polemizować. Jaki system plików może być wykorzystywany w przypadku linux'a, no
jaki... nie wiem... W którym miejscu na dysku są przechowywane dane, hmmm... a jakie dane? Nie
wiadomo ale wiadomo, że jakieś są... albo ich nie ma... Poważnie to są argumenty przeciw włączeniu
trim/discard? Nie mogłem się powstrzymać.

No tak czy inaczej z tym ukrytym zaszyfrowanym voluminem to tu już jakiś sens to by miało. Nie
chodzi o fakt zatajenia korzystania z szyfrowania, tylko o coś na wzór opisywanego przeze mnie
jakiś czas temu [rozwiązania mającego na celu obejście cenzury przy przekraczaniu graniczy][8]. W
przypadku posiadania tak ukrytego kontenera, mechanizm trim/discard zwyczajnie go wyzeruje (lub
wyzeruje jego część) przy usuwaniu danych, co efektywnie go zniszczy.

Poza tymi powyższymi argumentami, dochodzi też fakt braku możliwości odwrócenia operacji usuwania
danych.  W przypadku dysku HDD (talerzowego), jak usuniemy plik, to on dalej na tym dysku rezyduje
do momentu ponownego zapisania danego sektora przez głowicę magnetyczną. Można kopię takiego pliku
wydobyć znając fizyczne jego położenie. Z kolei na dysku SSD z załączonym trim/discard , jak
usuniemy jakiś plik, to już tych danych nie będziemy w stanie odzyskać, bo stosowne komórki flash
zostaną natychmiast (lub po jakimś krótszym czasie) wyzerowane. Jak dla mnie to spory plus, by mieć
pewność, że skasowanych danych się nie da odzyskać, przynajmniej nie standardowymi technikami
wykorzystywanymi w przypadku dysków HDD.

## Włączenie trim/discard dla LUKS/dm-crypt

W moim przypadku, gdzie wszyscy wiedzą, że ja w zasadzie wszystko i zawsze szyfruję, to mogę sobie
włączyć trim/discard dla zaszyfrowanych kontenerów LUKS. A to, że ktoś pozna, że korzystam z
systemu plików NTFS na swoim Debianie, to dla mnie nie ma większego znaczenia. :]

By włączyć trim/discard dla kontenerów LUKS/dm-crypt, trzeba edytować plik `/etc/crypttab` i
dopisać w nim opcję `discard` przy pozycji z kontenerem, przykładowo:

    # <target name>	<source device>		<key file>	<options>
    debian_crypt  UUID=9e3c1bb4-570f-4eb2-a1c5-51a4aabedeb4   c1  luks,keyscript=decrypt_keyctl,initramfs,discard
    sys_crypt     UUID=66861f93-9fc7-46f9-b969-1ade25dcb898   c1  luks,keyscript=decrypt_keyctl,initramfs

Pierwsza pozycja odpowiada za kontener na dysku SSD, druga za kontener na dysku HDD.

Po uzupełnieniu pliku `/etc/crypttab` trzeba na nowo wygenerować obraz initramfs/initrd:

    # update-initramfs -u -k all

Po wygenerowaniu obrazu initramfs/initrd trzeba uruchomić ponownie system.

Po tym jak się już system uruchomi, sprawdzamy, czy wsparcie dla trim/discard w kontenerze LUKS
zostało włączone:

    # lsblk --discard /dev/sdb
    NAME                        DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
    sdb                                0      512B       2G         0
    ├─sdb1                             0      512B       2G         0
    ├─sdb2                             0      512B       2G         0
    └─sdb3                             0      512B       2G         0
      └─debian_crypt                   0      512B       2G         0
        ├─goodram_ssd-ccache           0      512B       2G         0
        ├─goodram_ssd-debuilder        0      512B       2G         0
        ├─goodram_ssd-root             0      512B       2G         0
        └─goodram_ssd-home             0      512B       2G         0

I jak widzimy wyżej, pozycja z kontenerem `debian_crypt` posiada już niezerowe wartości w kolumnach
`DISC-GRAN` oraz `DISC-MAX` .

Możemy też zajrzeć w `dmsetup` :

    # dmsetup table
    debian_crypt: 0 463583232 crypt aes-xts-plain64 :64:logon:cryptsetup:000etc000-d0 0 8:19 32768 1 allow_discards

Sprawdźmy zatem czy trim/discard działa na tych logicznych voluminach LVM, które znajdują się
wewnątrz tego kontenera LUKS:

    # fstrim  --all -v
    /efi: 501.6 MiB (526008320 bytes) trimmed on /dev/sdb2
    /boot: 1.8 GiB (1928650752 bytes) trimmed on /dev/sdb1
    /media/ccache: 12.7 GiB (13622022144 bytes) trimmed on /dev/mapper/goodram_ssd-ccache
    /home: 43.7 GiB (46881816576 bytes)  trimmed on /dev/mapper/goodram_ssd-home
    /media/debuilder: 29.7 GiB (31883202560 bytes) trimmed on /dev/mapper/goodram_ssd-debuilder
    /: 21.9 GiB (23515930624 bytes) trimmed on /dev/mapper/goodram_ssd-root

Wygląda na to, że teraz wszystko działa tak jak powinno.

### LUKSv2 i jego flagi

Jeśli korzystamy z kontenerów LUKSv2, to opcji dla tego kontenera nie musimy określać w pliku
`/etc/crypttab` , bo możemy to zrobić trwale przy pomocy flag oferowanych przez LUKSv2 (dany
kontener będzie tak samo skonfigurowany OOTB na różnych systemach). Robimy to standardowo przy
pomocy narzędzia `cryptsetup` :

    # cryptsetup --allow-discards --persistent refresh debian_crypt
    Enter passphrase for /dev/sdb3:

Po czym weryfikujemy czy flagi zostały poprawnie ustawione:

    # cryptsetup luksDump /dev/sdb3 | grep -i flags
    Flags:          allow-discards

By usunąć flagi, trzeba wydać to powyższe polecenie bez flag, czyli:

    # cryptsetup refresh debian_crypt

## Włączenie trim/discard dla LVM

Jak widzieliśmy wyżej, dodanie do pliku `/etc/crypttab` opcji `discard` włączyło mechanizm
trim/discard zarówno dla kontenera LUKS, jak i dla wszystkich voluminów logicznych LVM wewnątrz
tego kontenera. Niemniej jednak, jak się przejrzy plik `/etc/lvm/lvm.conf` , to mamy tam do
dyspozycji opcję `issue_discards` :

    issue_discards = 1

Ta opcja jest domyślnie wyłączona, a mimo to, i tak trim/discard na tych logicznych voluminach się
pojawił.

Opcja `issue_discards` dotyczy nieco innego aspektu pracy LVM. Zgodnie z opisem tej opcji,
trim/discard jest wysyłany do fizycznego voluminu, do którego odnosi się logiczny volumin, w
momencie gdy ten logiczny volumin już nie korzysta z konkretnego miejsca fizycznego voluminu, np.
po wydaniu poleceń `lvremove` czy `lvreduce` . W takiej sytuacji LVM informuje dysk, że
dany region, który był we władaniu jakiegoś logicznego voluminu nie jest już używany. Jeśli zaś
chodzi o system plików obecny na takim logicznym voluminie LVM, to opcja `issue_discards` nie ma
tutaj większego znaczenia i jeśli urządzenie wspiera trim/discard (fizyczny nośnik albo kontener
LUKS), to polecenie `fstrim` wysłane przez system plików do dysku przejdzie transparentnie przez
warstwę LVM.

Jeśli korzystamy z [mechanizmu LVM snapshot][9], to możemy ustawić sobie `issue_discards = 1` w
pliku `/etc/lvm/lvm.conf` i w takim przypadku, bloki wykorzystywane przez volumin snapshot'a będą
odzyskiwane po usunięciu takiego snapshot'a.

## Opcja discard w pliku /etc/fstab

Przeglądając internet, natknąłem się na szereg postów, których autorzy doradzali, by w pliku
`/etc/fstab` dopisać parametr `discard` przy wszystkich systemach plików, które zamierzamy montować,
np. `/` czy `/home/` , coś na poniższy wzór:

    UUID=bcc4149e-7ec6-417c-97f4-23c5b6793450  /home  ext4  defaults,discard,nodev,nosuid,noexec,lazytime,errors=remount-ro 0 2

Lub przy pomocy tego poniższego polecenia ustawić opcję `discard` w superbloku danego systemu
plików bez potrzeby ruszania pliku `/etc/fstab` , co może przydać się nam, gdy dysk jest używany na
kilku systemach, np. dysk zewnętrzny:

    # tune2fs -o discard /dev/mapper/goodram_ssd-home

Opcja `discard` powoduje jednak, że system plików za każdym, gdy zostanie skasowany jakiś plik,
będzie przesyłał polecenie `fstrim` do firmware dysku w celu wyzerowania tych zwolnionych bloków.
Tego typu działanie może odbić się bardzo negatywnie na wydajności dysku SSD oraz skrócić jego
żywotność. Dlatego w zasadzie nie zaleca się korzystania z tej opcji systemu plików. Lepszym
rozwiązaniem jest okresowe wywoływanie polecenia `fstrim`, czy to ręcznie czy też automatycznie, np.
 za sprawą usługi systemd.

## Cykliczny trim/discard za sprawą usługi systemd

Na usługę systemd odpowiedzialną za ogarnianie mechanizmu trim/discard składają się dwa pliki:
`fstrim.service` oraz `fstrim.timer` .

Plik `fstrim.service` odpowiada za wywołanie polecenia `fstrim` na każdym z zamontowanych systemów
plików, który opcję `discard` wspiera:

    # systemctl cat fstrim.service
    # /usr/lib/systemd/system/fstrim.service
    ...
    [Service]
    ...
    ExecStart=/sbin/fstrim --listed-in /etc/fstab:/proc/self/mountinfo --verbose --quiet-unsupported
    ...

Plik `fstrim.timer` to zegar, którego zadaniem jest uruchomić w odpowiednim czasie usługę z pliku
`fstrim.service` :

	# systemctl cat fstrim.timer
	# /usr/lib/systemd/system/fstrim.timer
    ...
	[Timer]
	OnCalendar=weekly
	AccuracySec=1h
	Persistent=true
	RandomizedDelaySec=100min
    ...

Jak widzimy, zegar domyślnie jest ustawiony na około 7 dni. Czyli co 7 dni system będzie wywoływał
polecenie `fstrim` na każdym zamontowanym systemie plików.

Wszystko co musimy zrobić to aktywować ten zegar:

    # systemctl enable fstrim.timer
    Created symlink /etc/systemd/system/timers.target.wants/fstrim.timer → /usr/lib/systemd/system/fstrim.timer.

Sprawdźmy jak wygląda status zegara:

	# systemctl list-timers
	NEXT                            LEFT LAST                              PASSED UNIT                         ACTIVATES
	...
	Mon 2023-12-11 01:12:15 CET       8h Wed 2023-12-06 22:26:57 CET            - fstrim.timer                 fstrim.service
	...

Za około 8 godzin usługa mająca na celu wydanie polecenia `fstrim` zostanie uruchomiona.

## Manualny trim/discard

W każdym momencie pracy maszyny, polecenie `fstrim` możemy zainicjować ręcznie. Wystarczy odpalić
usługę `fstrim.service` lub wpisać polecenie tej usługi w terminalu:

	# systemctl restart fstrim.service

W logu systemowym zaś można zaobserwować poniższe komunikaty:

	systemd[1]: Starting fstrim.service - Discard unused blocks on filesystems from /etc/fstab...
	fstrim[18369]: /media/debuilder: 29.7 GiB (31879254016 bytes) trimmed on /dev/mapper/goodram_ssd-debuilder
	fstrim[18369]: /media/ccache: 12.7 GiB (13622022144 bytes) trimmed on /dev/mapper/goodram_ssd-ccache
	fstrim[18369]: /boot: 1.8 GiB (1928536064 bytes) trimmed on /dev/sdb1
	fstrim[18369]: /home: 45.2 GiB (48578965504 bytes) trimmed on /dev/mapper/goodram_ssd-home
	fstrim[18369]: /: 26.8 GiB (28801794048 bytes) trimmed on /dev/mapper/goodram_ssd-root
	systemd[1]: fstrim.service: Deactivated successfully.
	systemd[1]: Finished fstrim.service - Discard unused blocks on filesystems from /etc/fstab.

Wiemy zatem, że polecenie `fstrim` zostało przesłane do dysku SSD.

## Czy hibernować system mając dysk SSD

Bardzo cenię sobie hibernację ze względu na fakt możliwości odtworzenia stanu pracy po odcięciu
zasilania i wyłączeniu maszyny. Aktualnie mój laptop ma 16 GiB pamięci RAM, z czego zwykle połowa
jest w użyciu. Jeśli bym hibernował system, to z każdą hibernacją, te 8 GiB by było zapisywane na
dysk. Widać zatem, że hibernacja trochę mija się z celem w przypadku dysku SSD, bo dziennie, 20-50
GiB by szło na jej obsługę i komórki flash by się dość szybko zużyły.

Biorąc pod uwagę fakt, że stary dysk HDD w laptopie mi został (w kieszeni w miejscu wyciągniętego
już dawno temu cd-rom'u), to kawałek tego dysku mogę przeznaczyć na SWAP -- dokładnie ten sam
kawałek, który robił za SWAP, gdy system działał na tym dysku. W zasadzie nic się tutaj nie
zmieni, nawet wpis od SWAP zostanie ten sam w pliku `/etc/fstab` . Podobnie zawartość pliku
`/etc/initramfs-tools/conf.d/resume` zostanie taka sama:

    # cat /etc/initramfs-tools/conf.d/resume
    RESUME=/dev/mapper/wd_blue_label-swap

Czyli system będzie sobie działał na dysku SSD ale hibernował się będzie na starym dysku HDD
oszczędzając tym samym zapis cennych gigabajtów. Oczywiście ten stary dysk też jest na bazie
LUKSv2+LVM, przez co dane w SWAP pozostają zaszyfrowane.

A co w przypadku, gdy nasz laptop ma tylko jeden dysk i jest nim SSD? W takim przypadku bym
zrezygnował z opcji hibernacji, bo skróciłaby ona znacząco żywotność dysku, przynajmniej w moim
przypadku, gdzie potrafię hibernować system parę razy dziennie.

## Odciążenie dysku SSD przez wykorzystanie ramdysków

Część danych na dysku twardym podczas pracy systemu operacyjnego czy też korzystania z określonych
aplikacji jest zapisywana w formie cache. Niby cache ma na celu przyśpieszenie pracy systemu ale
niekiedy mija się to z celem, np. gdy w grę wchodzi przeglądarka internetowa. Jasne, bez cache
strony będą się ładować wolniej i więcej danych trzeba będzie pobrać przez sieć ale w obecnych
czasach strony WWW podlegają nieustannym zmianom z racji pojawiania się na nich nowego kontentu, a
gdy taka strona zostanie zmieniona, to cache trzeba na nowo wygenerować. Taki cache jest ważny
(daje nam jakiś boost) jedynie przez kilka minut, może parę godzin, po czym znów trzeba go
wygenerować i zapisać na dysku. Do takich aplikacji lepszym rozwiązaniem jest cache przechowywany w
pamięci operacyjnej RAM. Nie dość, że będzie on o wiele szybszy, to nawet jeśli nam on zniknie po
wyłączeniu maszyny, to nie zauważymy spowolnienia w działaniu systemu albo będzie ono na tyle
niewielkie, że go realnie nie odczujemy.

### Katalog ~/.cache/

Pierwszym miejscem, które przydałoby się zamontować w RAM, to katalog cache użytkownika, tj.
`~/.cache/` . Przy intensywnym korzystaniu z internetu, ten katalog może nam się bardzo szybko
rozrastać lub też stare pliki mogą być zastępowane nowym. Montując ten katalog w RAM, dziennie
możemy zaoszczędzić na zapisie dysku SSD nawet kilka GiB (może nawet i kilkanaście).

#### Plik /etc/fstab

Montowanie katalogu `~/.cache/` w RAM możemy przeprowadzić na dwa sposoby. Pierwszym z nich jest
dodanie poniższego wpisu w pliku `/etc/fstab` :

    tmpfs /home/morfik/.cache tmpfs uid=morfik,gid=morfik,mode=1700,size=6000M,nodev,nosuid,noexec 0 0

#### Plik usługi dla systemd

Drugim sposobem jest zaś napisanie pliku usługi dla systemd:

    # cat /etc/systemd/system/home-morfik-.cache.mount
    [Unit]
    Description = Mount Morfik's cache directory as tmpfs
    DefaultDependencies=no
    RequiresMountsFor=/home/morfik/.cache
    Before=local-fs.target

    [Mount]
    Where=/home/morfik/.cache
    What=tmpfs
    Type=tmpfs
    Options=uid=morfik,gid=morfik,mode=1700,size=6000M,nodev,nosuid,noexec

    [Install]
    WantedBy = multi-user.target

Włączamy usługę:

    # systemctl enable home-morfik-.cache.mount
    Created symlink /etc/systemd/system/multi-user.target.wants/home-morfik-.cache.mount → /etc/systemd/system/home-morfik-.cache.mount.

Przed aktywowaniem tej usługi, dobrze jest wyczyścić pierw katalog `~/.cache/` .

    # systemctl start home-morfik-.cache.mount

Sprawdzamy, czy zasób został zamontowany:

    $ mount | grep cache
    tmpfs on /home/morfik/.cache type tmpfs (rw,nosuid,nodev,noexec,relatime,size=6144000k,mode=1700,uid=1000,gid=1000)

### Katalog /tmp/

W przypadku, gdy nasza maszyna jest wyposażona w większą ilość pamięci operacyjnej, możemy
zamontować sobie cały katalog `/tmp/` w RAM. Tutaj również możemy skorzystać z pliku `/etc/fstab`
lub wykorzystać dedykowaną do tego celu usługę systemd.

#### Plik /etc/fstab

Poniżej jest stosowny wpis do umieszczenia w pliku `/etc/fstab` :

    tmpfs /tmp      tmpfs nodev,nosuid,noexec,mode=1777,size=50% 0 0

#### Plik usługi dla systemd

W przypadku korzystania z systemd, do ogarniania plików tymczasowych w katalogu `/tmp/` mamy
specjalnie tego celu zaprojektowaną usługę, tj. `tmp.mount` :

    # cp /usr/share/systemd/tmp.mount /etc/systemd/system
    # systemctl enable tmp.mount
    Created symlink /etc/systemd/system/local-fs.target.wants/tmp.mount → /etc/systemd/system/tmp.mount.

Domyślnie maksymalnie 50% RAM będzie mogło zostać wykorzystane pod katalog `/tmp/` . Jeśli
potrzebujemy zmienić tę wartość na wyższą/niższą, to robimy to w pliku usługi manipulując wartością
parametru `size=50%` .

Po włączeniu tej usługi, dobrze jest wyłączyć środowisko graficzne i z poziomu TTY usunąć
wszystkie pliki zalegające w katalogu `/tmp/` .

### Katalog /var/cache/

W katalogu  `/var/cache/` znajduje się cache różnych aplikacji, który nie powinien być usuwany z
każdym restartem komputera. Niemniej jednak, są tam pewne katalogi, np. `/var/cache/apt/` , który
przechowuje pobrane przez menadżer pakietów paczki `.deb` . Przy standardowych instalacjach
Debiana, pliki w tym katalogu znajdują zastosowanie z reguły jedynie podczas
instalacji/aktualizacji pakietów. Z chwilą, gdy te pakiety zostaną już zainstalowane, to te pliki
są zbędne i z jednej strony tylko zajmują nam miejsce na dysku, z drugiej zaś strony utylizują
komórki flash. Jeśli do tego dojdzie nam jeszcze wykorzystywanie gałęzi unstable/experimental, to
dziennie może nam się tych aktualizacji uzbierać nawet kilkaset MiB, a nierzadko nawet 1-2 GiB.
Jeśli z tym cache APT nic nie robimy, np. [nie budujemy sobie lokalnie pakietów .deb][13], to nie
ma większego sensu zapisywanie tych danych na dysk, bo i tak z nich większego użytku nie zrobimy.

#### Plik /etc/fstab

Poniżej jest stosowny wpis do umieszczenia w pliku `/etc/fstab` :

    tmpfs /var/cache/apt      tmpfs 0 0

#### Plik usługi dla systemd

W przypadku korzystania z systemd, możemy napisać sobie plik usługi:

	# cat /etc/systemd/system/var-cache-apt.mount
	[Unit]
	Description = Mount APT cache
	DefaultDependencies=no
	RequiresMountsFor=/var/cache/apt
	Before=local-fs.target

	[Mount]
	Where=/var/cache/apt
	What=tmpfs
	Type=tmpfs

	[Install]
	WantedBy = multi-user.target

Włączamy usługę:

    # systemctl enable var-cache-apt.mount
    Created symlink /etc/systemd/system/multi-user.target.wants/var-cache-apt.mount → /etc/systemd/system/var-cache-apt.mount.

I podobnie jak w poprzednich przypadkach, czyścimy katalog `/var/cache/apt/` i uruchamiamy usługę:

    # systemctl start var-cache-apt.mount

    # mount | grep apt
    tmpfs on /var/cache/apt type tmpfs (rw,relatime)

Dobrze jest przetestować sobie jeszcze czy menadżer pakietów działa prawidłowo:

    # apt-get update
    # aptitude safe-upgrade

    # ls -al /var/cache/apt
    total 98296
    drwxrwxrwt  3 root root      100 2023-12-09 15:24:02 ./
    drwxr-xr-x 24 root root     4096 2023-12-09 15:17:08 ../
    drwxr-xr-x  3 root root      540 2023-12-09 15:20:56 archives/
    -rw-r--r--  1 root root 50353616 2023-12-09 15:24:02 pkgcache.bin
    -rw-r--r--  1 root root 50291843 2023-12-09 15:18:32 srcpkgcache.bin

    # ls -al /var/cache/apt/archives
    total 106808
    drwxr-xr-x 3 root root      540 2023-12-09 15:20:56 ./
    drwxrwxrwt 3 root root      100 2023-12-09 15:24:02 ../
    drwx------ 2 _apt root       40 2023-12-09 15:20:56 partial/
    -rw-r--r-- 1 root root 12826068 2023-12-08 09:24:30 groovy_2.4.21-10_all.deb
    -rw-r--r-- 1 root root  1451740 2023-12-09 01:14:23 libglib2.0-0_2.78.3-1_amd64.deb
    -rw-r--r-- 1 root root  1223328 2023-12-09 00:59:12 libglib2.0-data_2.78.3-1_all.deb
    ...

### Logi

Przeciętny użytkownik bardzo rzadko kiedy zagląda do logów systemowych, bo albo w swoim systemie
nie napotyka żadnych problemów (więc nie ma po co), albo nawet nie wie co to takiego te logi
systemowe i jak w nie zajrzeć. Niemniej jednak, te logi mogą się rozrastać (zwłaszcza na systemach,
o które się słabo dba) za sprawą np. całej masy takich samych (lub podobnych komunikatów), które są
efektem jakiegoś błędu. Pół biedy, gdy zareagujemy w porę, ustalimy przyczynę i ją wyeliminujemy.
Ale jeśli zostawimy taki system sam sobie, to setki MiB w ciągu minut będą zapisane na dysk, a tego
chcielibyśmy uniknąć mając dysk SSD.

#### Systemd i jego journal

W przypadku systemd, możemy skonfigurować sobie logowanie komunikatów systemowych do pamięci
operacyjnej RAM. Wystarczy w pliku `/etc/systemd/journald.conf` zmienić wartość parametru
`Storage` . Mamy w zasadzie do wyboru te trzy poniższe opcje:

- `persistent`-- logi będą przechowywane na dysku, tj. w katalogu `/var/log/journal/` .
- `volatile` -- logi będą przechowywane w pamięci RAM, tj. w katalogu `/run/log/journal/` .
- `none` -- wyłącza logowanie.

Jeśli bardzo rzadko zaglądamy w logi systemowe, to nie musimy ich trzymać na dysku. Nie zalecałbym
całkowitego wyłączania logowania, dlatego też lepszym rozwiązaniem będzie opcja `volatile` :

	[Journal]
	...
	Storage=volatile

By nam się te logi w RAM nie rozrastały za bardzo, możemy także ograniczyć ich rozmiar, powiedzmy
do 128 MiB:

	RuntimeMaxUse=128M
	RuntimeMaxFileSize=128M

#### Sesja graficzna (plik ~/.xsession-errors)

Logi systemowe to nie jedyne logi w naszym linux'ie. Sesja graficzna również generuje całą masę
logów, a konkretnie to te wszystkie okienkowe aplikacje te logi mogą generować i ich też może się
sporo nazbierać. Wszystkie te logi wędrują do pliku `~/.xsession-errors` , przynajmniej gdy
korzystamy z Xserver'a.

Swojego czasu przestałem korzystać z tego pliku na rzecz przekierowania tych logów do urządzenia
FIFO celem wyświetlenia ich na konsoli, co efektywnie wrzuciło te logi do pamięci RAM i odciążyło
wtedy mój dysk HDD. Nie będę opisywał tutaj całego tego zagadnienia jedynie skrótowo nadmienię, że
w pliku `/etc/X11/Xsession` znajduje się ten poniższy kod:

    ERRFILE=$HOME/.xsession-errors
    ...
    exec >>"$ERRFILE" 2>&1

Możemy np. zmienić lokalizację tego pliku na `$HOME/.cache/.xsession-errors` , który już powinien
być zamontowany w RAM i po sprawie.

## Optymalizacja wykorzystania pamięci RAM

Takie montowanie katalogów w RAM niesie ze sobą ryzyko wyczerpania się w pewnym momencie pamięci
operacyjnej. Nawet jeśli mamy jej dużo, to ciągłe wgrywanie kolejnych plików do tych folderów
zamontowanych w RAM ostatecznie sprawi, że nam tej pamięci RAM zabraknie. By się przed takim stanem
rzeczy zabezpieczyć, dobrze jest co jakiś czas czyścić te katalogi z nieużywanych danych. Do tego
celu systemd ma stosowną usługę, którą konfiguruje się przez pliki w katalogu `/etc/tmpfiles.d/` .

Stwórzmy sobie zatem w katalogu `/etc/tmpfiles.d/` plik `tmp-cache.conf` o poniższej zawartości:

    # Type	Path	Mode	User	Group	Age		Argument

    d /home/morfik/.cache/ 0700 morfik morfik 6h -
    d /var/cache/apt/ 0755 root root 2h -
    D /tmp/ 1777 root root 6h -

Pliki i katalogi obecne w folderach `/home/morfik/.cache/` , `/var/cache/apt/` oraz `/tmp/`  będą
czyszczone według zadanego interwału czasowego. Katalogu `/tmp/` raczej nie powinno być potrzeby by
go czyścić ale na wszelki wypadek stosowny wpis został dodany.

### Kompresja danych w RAM za sprawą ZRAM

W przypadku, gdy naszemu systemowi zaczyna brakować pamięci RAM, możemy pokusić się o
zaprzęgnięcie [mechanizmu ZRAM][17]. Ma on na celu poddać kompresji dane obecne w pamięci
operacyjnej i w ten sposób niejako więcej danych będziemy w stanie do niej załadować, oczywiście
kosztem cykli procesora. ZRAM jest w stanie oddalić trochę w czasie potrzebę stosowania fizycznej
przestrzeni wymiany SWAP (tej obecnej na dysku).

Warto też zainteresować się [projektem zram-init][18], który to przy pomocy usług dla systemd jest
w stanie zamontować katalogi takie jak `/tmp/` czy `/var/tmp/` w pamięci RAM oraz też jest w stanie
zaimplementować SWAP na bazie ZRAM, czyli przestrzeń wymiany SWAP ale będzie ona obecna również w
pamięci operacyjnej.

## Optymalizacja mechanizmu równoważenia zużycia komórek flash

W artykule, który był poświęcony [optymalnemu podziałowi dysku HDD/SSD pod linux][14], wspomniałem,
że na dysku SSD powinno się zostawić trochę miejsca, by proces równoważenia zużycia komórek flash
przebiegał swobodnie. Z informacji, które znalazłem na necie, ta wartość wolnej przestrzeni powinna
być w granicach 20-30% w skali całego dysku. Jako, że ja wykorzystuję LVM, to postanowiłem około
10% przestrzeni dysku pozostawić w formie nieużywanej, tak by na wszelki wypadek trochę wolnego
miejsca zostało, nawet w przypadku, gdy cały dysk by się zapchał z jakiegoś powodu.

	# pvscan
	  PV /dev/mapper/sys_crypt      VG wd_blue_label   lvm2 [928.98 GiB / 4.00 GiB free]
	  PV /dev/mapper/debian_crypt   VG goodram_ssd     lvm2 [221.05 GiB / 21.05 GiB free]
	  Total: 2 [1.12 TiB] / in use: 2 [1.12 TiB] / in no VG: 0 [0   ]

To wolne miejsce przyda się także pod mechanizm LVM snapshot, ze względu na fakt, że tylko
tymczasowo taki volumin jest wykorzystywany, np. podczas sprawdzania systemu plików w poszukiwaniu
błędów, czy też podczas bardziej ryzykownej aktualizacji systemu.

## Poprawa wydajności szyfrowania w dyskach SSD

[W kontenerze LUKSv2 możemy określić więcej flag][10]. Dwie z nich, tj. `--perf-no_read_workqueue`
oraz `--perf-no_write_workqueue` zdają się być wielce użyteczne, bo [są w stanie poprawić wydajność
szyfrowania][11] i przez to zwiększyć przepustowość dysku SSD dwukrotnie przy jednoczesnym
zredukowaniu opóźnień I/O o połowę. Te dwie flagi możemy dodać do kontenera LUKSv2:

    # cryptsetup --allow-discards --perf-no_read_workqueue --perf-no_write_workqueue --persistent refresh debian_crypt
    Enter passphrase for /dev/sdb3:

    # cryptsetup luksDump /dev/sdb3 | grep -i flags
    Flags:          allow-discards perf-no_read_workqueue perf-no_write_workqueue

Trzeba jednak tutaj zaznaczyć, by móc skorzystać z tych flag, [trzeba posiadać w miarę nowy kernel,
tj. 5.9+][12].

Gdy korzystamy ze starszej wersji kontenerów LUKS, to te dwie opcje trzeba określić w pliku
`/etc/crypttab` :

    debian_crypt  UUID=9e3c1bb4-570f-4eb2-a1c5-51a4aabedeb4   c1  luks,keyscript=decrypt_keyctl,initramfs,discard,perf-no_read_workqueue,perf-no_write_workqueue

## Podsumowanie

Poprawne skonfigurowanie trim/discard w przypadku dysków SSD na linux, to podstawa jeśli chcemy z
tego typu nośników pamięci masowej korzystać. Proces konfiguracji tego mechanizmu w aktualnych
dystrybucjach linux'a sprowadza się w zasadzie do aktywacji jednego zegara oferowanego przez
systemd. W przypadku nieco bardziej zaawansowanej konfiguracji systemu zakładającej wykorzystanie
zaszyfrowanych kontenerów LUKS oraz dysków logicznych na bazie LVM, trzeba się nieco bardziej się
wysilić ale wciąż wszystkie kroki sprowadzają się do edycji tylko jednego pliku. Ważniejszym
aspektem konfiguracji linux'a działającego na dysku SSD jest odpowiednie przygotowanie systemu
plików, tj. wykorzystanie ramdysków wszędzie tam, gdzie wiemy, że dane z konkretnych katalogów
niekoniecznie muszą być przechowywane w formie umożliwiającej do nich dostęp po odcięciu zasilania,
co pozwoli nam na znaczne ograniczenie operacji zapisu nośnika SSD, przez co zostanie dość znacznie
wydłużona jego żywotność.


[1]: https://www.goodram.com/produkty/ssd-irdm-gen-2-sata-iii-25/
[2]: /post/instalacja-debiana-z-wykorzystaniem-debootstrap/
[3]: https://en.wikipedia.org/wiki/Device_mapper
[4]: https://wiki.gentoo.org/wiki/Discard_over_USB
[5]: https://man7.org/linux/man-pages/man8/fstrim.8.html
[6]: https://wiki.archlinux.org/title/Dm-crypt/Specialties
[7]: https://asalor.blogspot.com/2011/08/trim-dm-crypt-problems.html
[8]: /post/jak-ukryc-zaszyfrowany-kontener-luks-pod-linux/
[9]: /post/backup-przy-pomocy-lvm-snapshot/
[10]: https://www.kernel.org/doc/html/latest/admin-guide/device-mapper/dm-crypt.html
[11]: https://blog.cloudflare.com/speeding-up-linux-disk-encryption/
[12]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/drivers/md/dm-crypt.c?id=39d42fa96ba1b7d2544db3f8ed5da8fb0d5cb877
[13]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/

[14]: /post/jak-optymalnie-podzielic-dysk-hdd-ssd-na-partycje-pod-linux/
[15]: https://en.wikipedia.org/wiki/Trim_(computing)#Disadvantages
[16]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/drivers/ata/libata-core.c
[17]: https://docs.kernel.org/admin-guide/blockdev/zram.html
[18]: https://github.com/vaeth/zram-init/
