---
author: Morfik
categories:
- Linux
date:    2023-12-15 17:00:00 +0100
lastmod: 2024-06-16 16:30:00 +0200
published: true
status: publish
tags:
- debian
- ssd
- trim
- discard
- luks
- lvm
- system-plików
- ext4
- sysctl
- udev
- usb
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
LUKS/dm-crypt, czy korzystaniu z voluminów logicznych LVM. Dlatego też postanowiłem przyjrzeć się
nieco bliżej zagadnieniu konfiguracji mechanizmu trim/discard na dysku SSD w przypadku
zaszyfrowanego systemu na bazie LUKS+LVM.

<!--more-->
## Czy kupowanie używanego dysku SSD to dobry pomysł

We wstępie wspomniałem, że dysk SSD, który zakupiłem, nie był nowy ino używany. Konkretnie jest to
[model IR-SSDPR-S25A-240 od Goodram][1], czyli dysk o pojemności 240 GB, którego komórki flash są
wykonane w technologi MLC. Dlaczego zdecydowałem się na dysk używany, a nie nowy? Z kilku powodów.

Ten używany dysk SSD kosztował w granicach 50 zł, gdzie orientacyjna cena nowego urządzenia (tego
samego modelu) waha się w granicach 250 zł. Druga sprawa, to raport SMART, który został opublikowany
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

Gdyby naszego dysku SSD nie było w [bazie danych SMART][31], co możemy wywnioskować po informacji:
`Device is: Not in smartctl database 7.3/5319` oraz atrybutach mających w nazwach
`Unknown_Attribute` , to być może albo mamy za starą bazę danych, albo ten konkretny nośnik
zwyczajnie jeszcze nie został do niej dodany, bo nie wielu ludzi ma ten model dysku SSD i korzysta
z niego pod linux. Tak czy inaczej, dobrze jest wydać to poniższe polecenie:

    # update-smart-drivedb

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

Zaglądając na stronę producenta, możemy doszukać się informacji, że parametr `TBW` (możliwa ilość
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

## Budowa i zasada działania dysku SSD

By być w stanie prawidłowo skonfigurować dysk SSD do pracy pod linux (choć na dobrą sprawę, to pod
każdym innym systemem operacyjnym), trzeba się zaznajomić nieco z budową takich dysków oraz tym jak
one w rzeczywistości działają. Informacje zawarte poniżej zostały zebrane w oparciu o te artykuły:

- [The SSD Anthology: Understanding SSDs and New Drives from OCZ][21]
- [Solid-state revolution: in-depth on how SSDs really work][22]
- [How NAND flash degrades and what vendors do to increase SSD endurance][23]
- [SSD endurance myths and legends][29]

Zaznajomienie się z tą wiedzą pozwoli nam zrozumieć kilka istotnych kwestii związanych z
technologią flash, przez co unikniemy całej masy rozczarowań czy innych nieporozumień.

### NOR i NAND flash

Gdy rozmawiamy o dyskach SSD, pendrive czy kartach SD, zwykle referujemy do nich jako nośniki
pamięci flash. Niemniej jednak, pamięć flash można podzielić na dwa typy: NOR i NAND. Z jakiego
zatem typu pamięci flash są zbudowane nasze nośniki pamięci masowych?

Konstrukcja pamięci NOR flash jest niby dość prosta ale jednak czipy budowane w oparciu o tę
technologię są dość skomplikowane. Zaletą pamięci NOR flash jest możliwość zapisywania i
odczytywania pojedynczych bitów. Są miejsca, w których pamięć NOR flash idealnie znajduje
zastosowanie, ale gdy w grę wchodzą dysk twarde, to zapis tutaj odbywa się na sektorach 512 (lub
4096) bajtów, przez co ta zaleta pamięci NOR flash nie ma tutaj większego znaczenia.

Jeśli zaś chodzi o pamięć NAND flash, to ich budowa jest o wiele prostsza, choć nie jest pozbawiona
wad. W tym typie pamięci, komórki są pogrupowane na strony i bloki, co ma swoje konsekwencje w
przypadku zapisu i odczytu danych, tj. nie da się już zapisać/odczytać tylko pojedynczego bitu, tak
jak to miało miejsce w pamięci NOR flash -- tutaj trzeba zapisać/odczytać cały zestaw bitów
jednocześnie (zwany stroną), zwykle 4-16 KiB, co w wpasowuje się w budowę systemu plików, który
również operuje na klastrach danych o podobnym rozmiarze.

Ze względu właśnie na te różnice w konstrukcji pamięci flash, obecnie do budowy pamięci masowych
(nie tylko dysków SSD) wykorzystuje się pamięć NAND flash. Chodzi o to, że jesteśmy w stanie upchnąć
w niej około 60% więcej danych, tj. na tej samej przestrzeni fizycznej, którą taka pamięć flash
będzie zajmować na nośniku danych.

### Różnice w komórkach SLC, MLC, TLC, QLC i PLC

Modele dysków SSD odróżnia od siebie wewnętrzna budowa ich komórek flash. [Obecnie mamy][20] dyski
z komórkami SLC (Single Level Cell), MLC (Multi Level Cell), TLC (Triple Level Cell), QLC (Quad
Level Cell) i niebawem także PLC (Penta Level Cell).

Powszechnie uważa się, że każdy z tych typów komórek charakteryzuje się inną żywotnością. W
zasadzie to ta żywotność jest taka sama w przypadku wszystkich tych typów komórek. Problem tkwi
jednak w stanach napięć, które oprogramowanie dysku SSD musi być w stanie rozpoznać, by nie doszło
do uszkodzenia danych.

Ilości stanów napięć, które taka komórka flash jest w stanie przechowywać wzrasta wraz z rozwojem
nowszych technologi, tj. 2 stany dla SLC (1 bit), 4 dla MLC (2 bity), 8 dla TLC (3 bity), 16 dla
QLC (4 bity) i 32 dla PLC (5 bitów). W przypadku każdej z tych technologii oprogramowanie dysku
musi być w stanie poprawnie zinterpretować napięcie na komórce. To zadanie jest proste w przypadku,
gdy w komórce trzeba zapisać tylko 1 bit informacji, czyli 0 (brak napięcia) albo 1 (obecność
napięcia). Gdy trzeba rozpoznać więcej stanów napięć, to zaczynają się problemy, bo próg
rozróżnienia tych napięć ulega obniżeniu.

Dodatkowo każdy cykl programowania/wymazywania komórki sprawia, że zaczyna ona ulegać degradacji.
Chodzi generalnie o fakt, że proces wymazywania komórki flash polega na oddziaływaniu na nią
stosunkowo dużym ładunkiem energii elektrycznej. Taki stan rzeczy powoduje, że warstwa
półprzewodnikowa czipu ulega niewielkiej degradacji, co z czasem zwiększa współczynnik błędów przy
odczytywaniu bitów z takiej komórki.

Początkowo błędy te są korygowane programowo ale ostatecznie procedury kodu korekcji błędów w
kontrolerze pamięci flash nie będą w stanie nadążyć za tymi błędami i komórka pamięci flash stanie
się niestabilna, a odczyt jej zawartości zupełnie już nie będzie możliwy. Dlatego właśnie komórki
SLC potrafiły wytrzymać do 100K cykli, MLC do 10K cykli (zwykle około 3K), TLC do 3K cykli (zwykle
około 1K), QLC do 1K cykli (zwykle koło 100), a PLC szacuje się, że wytrzymają 10 cykli (pewnie
będą jednorazowego użytku).

### Technologia flash, a degradacja wydajności

Żywotność komórek flash to nie wszystko, bo z racji potrzeby rozpoznawania większej ilości stanów
napięć, to czasy dostępu do tych komórek muszą zostać zwiększone dość znacznie. I jeśli zestawimy
sobie te wszystkie technologie budowy komórek flash, to zobaczymy coś na wzór poniższej tabelki:

                            SLC     MLC     TLC     QLC     PLC
    Czas odczytu (us)       30      50      75      b/d     b/d
    Czas programowania (us) 300     600     1000    b/d     b/d
    Czas wymazywania (us)   1500    3000    5000    6000+   b/d

Może i dyski TLC/QLC są dość sporych rozmiarów ale czas wymazywania w połączeniu z czasem
programowania komórki (zapis danych na dysku) przekraczają już czasy dostępów w przypadku dysków
HDD z głowicą magnetyczną... Wciąż jednak czas odczytu danych jest na w miarę przyzwoitym poziomie.

### Struktury danych w dyskach SSD

Komórki na dysku SSD są pogrupowane w strony (pages), czyli najmniejsze struktury, które mogą
zostać zapisane lub odczytane przez taki nośnik. Takie strony mogą mieć różny rozmiar w zależności
od producenta dysku SSD, choć zwykle można się spotkać z wartością 4-16 KiB. Te strony są grupowane
razem tworząc bloki. Taki blok jest to najmniejsza struktura, która może zostać wymazana na flash'u.
Rozmiar bloku też może się różnic w zależności od producenta dysku ale typowy rozmiar to 128-512
stron, czyli 512 KiB - 4 MiB.

Tego typu rozwiązanie powoduje, że zapis i odczyt z dysku SSD może być przeprowadzany w porcjach,
np. 4 KiB ale by ponownie zapisać daną stronę, to trzeba wyczyścić obszar całego bloku. Robi się to
zwykle przez skopiowane danych takiego bloku do cache dysku (lub dedykowanej pamięci RAM), po czym
czyści/modyfikuje się daną stronę i cały blok ponownie jest zapisywany na dysk. I tu właśnie
zaczynają się problemy z wydajnością dysków SSD.

### Spadek wydajności wraz z zapełnianiem się dysku SSD

W przypadku dysku HDD, gdzie głowica magnetyczna musi się wypozycjonować nad odpowiednim miejscem
na talerzu, czasy dostępu do sektorów są rzędu 7-10 ms i to w najlepszym wypadku. Oddalając się od
zewnętrznej krawędzi talerza w stronę jego środka, czas dostępu oraz przepustowość dysku ulegają
degradacji.

Dyski SSD nie cierpią z powodu ruchów głowicy magnetycznej, bo jej tam zwyczajnie nie ma, przez co
czasy dostępu do danych zgromadzonych na dysku sięgają około 0,1 ms i to jest czas dostępu do
każdej komórki bez znaczenia gdzie ona by nie była zlokalizowana, przez co transfer danych również
jest na wyższym poziomie, w stosunku do dysku HDD.

Przynajmniej taka jest teoria, bo realne testy pokazują coś zupełnie innego. Chodzi o fakt spadku
wydajności dysku SSD wraz z zapełnianiem go danymi, czyli pusty dysk SSD będzie nam działał bardzo
szybko i sprawnie, natomiast pełny dysk SSD będzie nam mulił i to nierzadko bardziej niż dysk HDD.

### Garbage Collection

By zrozumieć czym jest proces zwany Garbage Collection, trzeba pierw zrozumieć jak dysk SSD
zapisuje, czyści i ponownie zapisuje dane na nośniku flash. Przede wszystkim, taki nośnik flash nie
może zostać przepisany od tak na poziomie bitowym, tak jak to ma miejsce w przypadku dysków
talerzowych z głowicą magnetyczną.

W przypadku dysku SSD zmiany są zapisywane w nowych blokach, zaś stare dane obecne w oryginalnych
blokach są oznaczane jako możliwe do skasowania. Gdy cały nośnik flash zostanie zapełniony po raz
pierwszy, kontroler musi wymazać dane w całych blokach zanim nastąpi zapis nowych danych. Taki stan
rzeczy może dość znacznie degradować wydajność dysku SSD przy zapisie danych.

Dlatego też został opracowany mechanizm zwany Garbage Collection, który to czyści stare bloki
poprzez przenoszenie dobrych danych z częściowo pełnych bloków do nowego obszaru pamięci flash. Ten
proces opiera się na konsolidacji dobrych danych w istniejących blokach, tj. tych danych, które nie
są oznaczone do usunięcia. Przykładowo, mamy 2 bloki (każdy po 512 KiB danych), które są zapisane w
połowie, bo wcześniej skasowaliśmy z dysku jakieś pliki. Zadaniem Garbage Collection jest
skopiowanie tych bloków do cache (lub dedykowanej pamięci operacyjnej) dysku SSD, wydobycie z nich
tych dobrych danych (256 KiB z jednego bloku i 256 KiB z drugiego bloku), połączenie ich w jeden
blok 512 KiB i zapisanie tych danych w wolnym bloku na dysku. Te dwa poprzednie bloki jako, że nie
zawierają już użytecznych danych, mogą zostać w całości wyczyszczone.

Trzeba jednak zdawać sobie sprawę, że ten proces może przepisywać więcej danych niż my faktycznie
zapisujemy na dysku (tzw. Write Amplification), np. skopiowaliśmy plik 4 KiB, a na flash zostanie
zapisany cały blok 512 KiB. Te dodatkowe cykle zapisu komórek mogą degradować wydajność dysku przez
kolizję z operacjami zapisu/odczytu innych danych, a ponadto nie pozostają bez znaczenia dla
żywotności samych komórek flash. Dlatego też proces Garbage Collection stara się przeprowadzać
konsolidację danych, gdy dysk nie jest aktywnie używany lub też projektuje się te nośniki w taki
sposób, by były w stanie zapewnić deklarowaną wydajność nawet w przypadku, gdy GC przeprowadza swoje
operacje podczas normalnej aktywności dysku.

### Wolne miejsce na dysku SSD

Prawdziwy dramat zaczyna się jednak w momencie, gdy dysk SSD zaczynamy zapełniać prawie w całości.
W takim przypadku, mechanizm Garbage Collection będzie miał ogromne problemy z odzyskiwaniem
bloków -- trzeba będzie kopiować więcej bloków zajętych częściowo, a im więcej danych dysk
odczytuje, modyfikuje i zapisuje, tym więcej operacji taki nośnik przeprowadza, co przekłada się
czas w jakim to robi, czyli obniżeniu jego wydajności.

Dlatego też producenci dysków SSD projektują te nośniki w taki sposób, by część dostępnej
przestrzeni nie była nigdy widoczna dla użytkownika, tzw. Over Provisioning, zwykle koło 10%
deklarowanej pojemności.

### Trim lekiem na całe zło

Te problemy z wydajnością dysków SSD biorą się ze względu na fakt, że taki nośnik nie wie, że pliki
zostały skasowane, w przeciwieństwie do systemu plików, który to jest na bieżąco z tymi
informacjami. Dopiero w momencie, gdy trzeba nadpisać konkretne komórki flash, dysk SSD zyskuje tę
potrzebną informację i przystępuje do ich wyczyszczenia i ponownego zapisania. I tu właśnie do gry
wchodzi mechanizm TRIM, którego zadaniem jest powiadomienie dysku SSD, że pewne dane zostały
usunięte z systemu plików, a co za tym idzie, można je wyczyścić z nośnika fizycznego. W ten sposób,
czyszczenie komórek flash ma miejsce w innym czasie niż ich ponowny zapis, co sprawia, że wydajność
dysku SSD ponownie wzrasta do tej deklarowanej przez producenta.

## Różnica między discard, trim i fstrim

Szukając informacji na temat konfiguracji dysku SSD pod linux, można natknąć się na dwa lub trzy
terminy, tj. `trim` , `fstrim` oraz `discard` . Z początku myślałem, że są to osobne rzeczy
ale [okazuje się, że trim, fstrim i discard odnoszą się do tego samego mechanizmu][4] i użytkownicy
z reguły stosują te pojęcia zamiennie. Dlatego dobrze byłoby doprecyzować czym jest `trim` , `fstrim`
oraz `discard` .

Generalnie `discard` to opcja montowania systemu plików i po jej ustawieniu firmware dysku będzie
powiadamiany o blokach, które już nie są wykorzystywane przez ten konkretny system plików. Efektem
ustawienia opcji `discard` dla takiego systemu plików jest wyczyszczenie bloków fizycznych dysku
praktycznie natychmiast po usunięciu jakiegoś pliku. Z kolei `trim` (albo bardziej precyzyjnie
polecenie `fstrim` ) robi dokładnie to samo ale nie w chwili, w której operacja usuwania danych ma
miejsce. Czyli zarówno opcja `discard` jak i `trim`/`fstrim` informują urządzenie blokowe o
przestrzeni, która była używana przez system plików ale została zwolniona i wróciła z powrotem do
wolnej puli.

Takie powiadamianie o zwolnionych blokach nie jest niczym niezwykłym, bo urządzenia oparte o
technologię flash muszą pierw wyczyścić komórki zanim będzie możliwy w nich ponowny zapis
faktycznych danych. Ten proces czyszczenia jest jednak bardzo powolny w porównaniu do operacji
zapisu realnych danych. Jeśli teraz weźmiemy dysk SSD bez opcji `discard`, który był wcześniej
zapisany w całości i będziemy chcieli zapisać w nim jakieś dane, to określone komórki flash będzie
trzeba uprzednio wyczyścić i dopiero po zakończeniu tego procesu zapisać te interesujące nas dane.
Widać zatem, że dysk może mieć problemy z wydajnością w takim przypadku, bo nie zapisuje od razu
tych danych, które byśmy sobie życzyli. Dlatego też by zachować dobrą wydajność przy zapisie danych
na dysku SSD trzeba korzystać z opcji `discard` lub cyklicznie wydawać polecenie `fstrim` .

### Co lepsze trim/fstrim czy discard

Zarówno `trim` (wywoływanie polecenia `fstrim` ) jak i opcja montowania `discard` mogą negatywnie
odbić się na wydajności, jak i żywotności dysku SSD. Nie powinniśmy zatem korzystać z opcji
`discard` , np. w pliku `/etc/fstab` . Z kolei jeśli chodzi o `trim` , to nie powinno się korzystać
z polecenia `fstrim` [częściej niż raz w tygodniu][5].

Dodatkowo dyski SSD mogą wspierać coś co się nazywa `queued trim` , czyli w określonych interwałach
czasowych lub przy określonych zdarzeniach, dysk SSD będzie automatycznie czyścił wolne bloki, tak
by te operacje trim/discard nie kolidowały z operacjami zapisu/odczytu danych na dysku. Jeśli nasz
dysk nie wspiera `queued trim` , to stosowanie opcji `discard` [może znacznie degradować wydajność
dysku SSD][15]. Ta sytuacja jest na tyle uporczywa, że deweloperzy kernela wpisują na [czarną
listę][16] ( `ata_device_blacklist` ) takie dyski SSD, by jakoś radzić sobie z tym problemem.

## Czy mój dysk SSD wspiera trim/discard

Wsparcie dla trim/discard można odczytać z raportu SMART lub z wyjścia polecenia `hdparm`:

    # smartctl -x  /dev/sdb | grep -i trim
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

Zatem, niby mamy dysk wspierający trim/discard, to jednak część partycji/dysków logicznych nie
wspiera tego mechanizmu. Dlaczego?

### Problematyczny device-mapper (LUKS/LVM)

Zgodnie z tym co można [wyczytać tutaj][6], developerzy device-mapper nie włączą mechanizmu
trim/discard dla zaszyfrowanych voluminów ze względów bezpieczeństwa. Wypadałoby w tym miejscu
sobie zadać pytanie o te względy bezpieczeństwa, tj. jakie dokładnie informacje mogą zostać
ujawnione, gdy włączymy trim/discard dla zaszyfrowanego kontenera LUKS/dm-crypt. [Odpowiedź można
znaleźć w tym artykule][7].

Po przeczytaniu tego posta naszły mnie mieszane uczucia. Argumenty przemawiające za tym, by nie
włączać domyślnie trim/discard dla LUKS/dm-crypt są takie, że będzie wiadomo, w którym miejscu na
dysku twardym są przechowywane jakieś dane, a które miejsce zostało wyczyszczone i nie zawiera
danych. Ponadto, może wyciec informacja z jakiego systemu plików korzystamy na tym urządzeniu.
Można też ustalić czy w obrębie danego wycinku dysku mamy ukryty jakiś zaszyfrowany volumin. No
tak, z czym tu polemizować. Jaki system plików może być wykorzystywany w przypadku linux'a, no
jaki... nie wiem... W którym miejscu na dysku są przechowywane dane, hmmm... a jakie dane? Nie
wiadomo ale wiadomo, że jakieś są... albo ich nie ma... Poważnie to są argumenty przeciw włączeniu
trim/discard? Nie mogłem się powstrzymać.

No tak czy inaczej z tym ukrytym zaszyfrowanym voluminem to tu już jakiś sens to by miało. Nie
chodzi o fakt zatajenia korzystania z szyfrowania, tylko o coś na wzór opisywanego przeze mnie
jakiś czas temu [rozwiązania mającego na celu obejście cenzury przy przekraczaniu graniczy][8]. W
przypadku posiadania tak ukrytego kontenera, mechanizm trim/discard zwyczajnie go wyczyści (lub
wyczyści jego część) przy usuwaniu danych, co efektywnie go zniszczy.

Poza tymi powyższymi argumentami, dochodzi też fakt braku możliwości odwrócenia operacji usuwania
danych.  W przypadku dysku HDD (talerzowego), jak usuniemy plik, to on dalej na tym dysku rezyduje
do momentu ponownego zapisania danego sektora przez głowicę magnetyczną. Można kopię takiego pliku
wydobyć znając fizyczne jego położenie. Z kolei na dysku SSD z załączonym trim/discard, jak
usuniemy jakiś plik, to już tych danych nie będziemy w stanie odzyskać, bo stosowne komórki flash
zostaną natychmiast (lub po jakimś krótszym czasie) wyczyszczone. Jak dla mnie to spory plus, by
mieć pewność, że skasowanych danych się nie da odzyskać, przynajmniej nie standardowymi technikami
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

Możemy też zajrzeć w `dmsetup` , gdzie powinna figurować flaga `allow_discards` :

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

Jeśli korzystamy z mechanizmu LVM snapshot, to możemy ustawić sobie `issue_discards = 1` w
pliku `/etc/lvm/lvm.conf` i w takim przypadku, bloki wykorzystywane przez volumin snapshot'a będą
odzyskiwane po usunięciu takiego snapshot'a.

### Opcja issue_discards=1 w przypadku stosowania e2scrub

Jest chyba jeden przypadek, w którym to nie powinniśmy ustawiać `issue_discards=1` w pliku
`/etc/lvm/lvm.conf` . Chodzi o wykorzystywanie [mechanizmu LVM snapshot][9] do sprawdzania systemu
plików ext4 w poszukiwaniu błędów, tzw. Online ext4 Metadata Check.

Podczas sprawdzania systemu plików tworzony będzie volumin LVM o określonym rozmiarze, np. 4 GiB.
Ten volumin jest jedynie tymczasowy, by przechować dane, które zostaną zmienione lub utworzone w
czasie, gdy ma miejsce sprawdzanie systemu plików. Danych raczej za dużo w tym czasie nie zapiszemy
ale po zakończonym procesie skanowania, ten volumin LVM zostanie usunięty, przez co wszystkie jego
bloki zostaną oznaczone do wyczyszczenia, czyli całe 4 GiB, nawet gdy tylko kilka MiB zostało
zapisanych (ekwiwalent skorzystania z `blkdiscard /dev/grupa/volumin` ). Jeśli teraz mamy sporo
voluminów w strukturze LVM i wszystkie one będą sprawdzane w poszukiwaniu błędów, to wiele GiB
będzie czyszczonych przez dysk SSD, a tego chcielibyśmy uniknąć.

### Brak możliwości odtworzenia zmian w strukturze LVM (backup)

Wygląda na to, że włączenie opcji `issue_discards=1` nie pozostaje bez wpływu na działanie
struktury LVM, a konkretnie chodzi o możliwość jej odtworzenia po dokonywaniu ewentualnych zmian
([a raczej jej braku][25]) przy pomocy polecenia `vgcfgrestore` . Dlatego biorąc pod uwagę fakt, że
ta opcja nie ma wpływu na zapytania trim systemu plików przesyłane do dysku SSD, to lepiej
pozostawić ją wyłączoną.

### Wyrównanie struktury LVM

Kolejna sprawa, to wyrównanie struktury LVM, by uniknąć niepotrzebnych operacji
odczyt-modyfikacja-zapis. Obecnie w zasadzie nie ma potrzeby nic ręcznie ustawiać, bo struktura LVM
zdaje się być już odpowiednio wyrównana i fizyczny volumin zaczyna się na granicy 1 MiB:

    # pvs /dev/mapper/debian_crypt -o+pe_start
      PV                       VG          Fmt  Attr PSize   PFree  1st PE
      /dev/mapper/debian_crypt goodram_ssd lvm2 a--  221.05g 21.05g   1.00m

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
będzie przesyłał polecenie `fstrim` do firmware dysku w celu wyczyszczenia tych zwolnionych komórek
flash. Tego typu działanie może odbić się bardzo negatywnie na wydajności dysku SSD oraz skrócić
jego żywotność. Chodzi generalnie o to, że przykładowo usuniemy plik 4 KiB, a kontroler dysku
będzie musiał wyczyścić cały blok komórek flash (512 KiB - 2 MiB w zależności od modelu dysku), co
z kolei wiązać będzie się z zapisem na dysku całego takiego bloku (odczyt-modyfikacja-zapis) i w
ten sposób więcej komórek flash zostanie ponownie zapisanych. Dlatego w zasadzie nie zaleca się
korzystania z tej opcji systemu plików. Lepszym rozwiązaniem jest okresowe wywoływanie polecenia
`fstrim`, czy to ręcznie czy też automatycznie, np. za sprawą usługi systemd.

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
	Mon 2024-01-01 01:35:46 CET   6 days Mon 2023-12-25 00:54:11 CET            - fstrim.timer                 fstrim.service
	...

Za około 6 dni usługa mająca na celu wydanie polecenia `fstrim` zostanie uruchomiona.

### Dostosowanie zegara dla fstrim.service w zależności od obciążenia dysku SSD

Warto tutaj zaznaczyć, że wywoływanie usługi `fstrim.service` w odstępie cotygodniowym powinno
wystarczyć w przypadku przeciętnego użytkownika linux'a (lub tych użytkowników, którzy nie chcą za
dużo zmieniać w swoich systemach). Może się jednak zdarzyć tak, że w naszym linux'ie albo
zapisujemy bardzo dużo danych na dysku SSD, albo też tych danych niewiele zmieniamy.

W jednym i drugim przypadku przydałoby się nieco dostosować czas wywoływania tej usługi systemd.
Jeśli wiemy, że za wiele danych nasz system nie będzie zmieniał, to usługę `fstrim.service` możemy
wywoływać raz na miesiąc. Podobnie w drugą stronę, gdy danych zapisujemy całą masę, to lepszym
rozwiązaniem jest wywołać tę usługę raz dziennie.

By dostosować czas zegara dla usługi `fstrim.service` , edytujemy plik tej usługi:

    # systemctl edit fstrim.timer

I dopisujemy poniższą treść:

    [Timer]
    OnCalendar=monthly

To dostosowanie czasu zegara dla usługi `fstrim.service` ma za zadanie ograniczyć narzut związany z
czymś, co się nazywa Write Amplification przy wywoływaniu przez dysk SSD procesu zwanego Garbage
Collection. Chodzi o to, by dysk SSD niekoniecznie czyścił wszystkie wolne komórki flash, tj. te, z
których dane zostały usunięte, co może powodować wzrost ilości danych, które ten dysk będzie
zapisywał na nośnik przyśpieszając w ten sposób degradację komórek flash.

Optimum w tej kwestii zależy od wykorzystania dysku, tj. im więcej danych zapisujemy, tym bardziej
pożądane jest, by tych komórek gotowych do zapisu (wyczyszczonych) było więcej, co przekłada się na
wydajność. Jeśli jednak nie zapisujemy dużo danych, to możemy sobie pozwolić, by w części komórek
dalej zalegały dane, tj. nie zostały jeszcze oznaczone przez system plików do wyczyszczenia.

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

Wiemy zatem, że polecenie `fstrim` wysłało stosowne zapytanie do dysku SSD.

## Przygotowanie systemu plików EXT4 do pracy z dyskiem SSD

Ze względu na budowę dysków SSD, a konkretnie struktur przechowywania w nich danych, gdzie taki
nośnik jest w stanie zapisać lub odczytać pojedyncze strony (4-16 KiB) ale wymazać już musi cały
blok danych (512 KiB - 4 MiB), system plików powinien zostać odpowiednio sformatowany do pracy pod
tego typu dyskami SSD. [Chodzi o parametry stride= oraz stripe_width=][24] .

> Parametr `stride=` konfiguruje system plików do macierzy RAID z rozmiarem części (ang. stride size
> lub chunk size) bloków systemu plików. Jest to liczba bloków odczytywanych lub zapisywanych na
> dysk przed przejściem na następny dysk. To ustawienie wpływa głównie na położenie metadanych
> systemu plików, takich jak mapy bitów podczas wykonywania mke2fs, aby zapobiec umieszczeniu ich
> na jednym dysku, co mogłoby negatywnie wpłynąć na wydajność. Może być również użyte przez alokator
> bloków.

> Parametr `stripe_width=` konfiguruje system plików do macierzy for RAID z rozmiarem-paska (ang.
> stripe width) bloków systemu plików na pasek. Zazwyczaj jest to rozmiar-części * N, gdzie N jest
> liczbą dysków z danymi w macierzy RAID (np. przy RAID 5, gdzie jest jeden dysk parzystości, M jest
> liczbą dysków w macierzy minus jeden). Pozwala to alokatorowi bloków na przeciwdziałanie cyklowi
> odczytu-modyfikacji-zapisu w pasku RAID, jeśli to możliwe, przy zapisie danych.

Tłumacząc te powyższe słowa na ludzki język. Opcja `stride=` określa minimalną ilość danych, które
można zapisać (lub odczytać) na dysk. Natomiast `stripe-width` sprawia, że system plików spróbuje
zorganizować zapisy w blokach o tym rozmiarze, aby zminimalizować kosztowne operacje. W
konfiguracjach RAID wykorzystujących parzystość (takich jak RAID 5) operacje
odczyt-modyfikacja-zapis są kosztowne, ponieważ oryginalne dane i ich parzystość są najpierw
odczytywane, dane następnie są poddawane modyfikacji, po czym obliczana jest nowa parzystość, a
następnie nowe dane i nowa parzystość są zapisywane z powrotem na dysku.

Co parametry RAID mają wspólnego z dyskami SSD? Dane na dysku SSD są przechowywane w komórkach,
które są zbierane w strony, czyli minimalną strukturę, którą kontroler takiego nośnika jest w
stanie zapisać albo odczytać z dysku. Strony są grupowane w bloki, tj. minimalną strukturę, którą
kontroler dysku SSD jest w stanie wymazać. Wygląda na to, że te parametry RAID pokrywają się ze
sposobem działania dysku SSD, czyli określając te opcje, struktury systemu plików mogą zostać
dopasowane do struktur nośnika, na którym ten system plików rezyduje.

Zatem jak dobrać te dwa parametry i czy w ogóle je dobierać przy tworzeniu nowego systemu plików
EXT4 na dysku SSD? Zdania na ten temat są podzielone. Jedni mówią, by tego nie robić i zdać się na
automatyczny dobór parametrów przy tworzeniu systemu plików, inni zaś mówią, że dobrze jest te
parametry określić, by zminimalizować ilość tych kosztownych operacji. Jeśli jednak już zdecydujemy
się na podanie tych dwóch parametrów przy tworzeniu systemu plików EXT4, to sposób wyliczenia ich
wartości również się nieco różni w zależności od tego, kogo spytamy.

W moim odczuciu najlepiej uargumentowany dobór wartości dla parametrów `stride=` oraz
`stripe_width=` jest przedstawiony w powyżej podlinkowanym artykule. Przy założeniu, że rozmiar
bloku systemu plików jest standardowy, tj. 4 KiB i mamy do czynienia z dyskiem SSD, który posiada
rozmiar strony 8 KiB i rozmiar bloku wymazywania 2 MiB, to `stride = 8 KiB / 4 KiB = 2` , zaś
`stripe-width = 2 MiB / 4 KiB = 512` .

### Jak ustalić rozmiar strony i rozmiar bloku wymazywania dysku SSD

Pytanie pozostanie w zasadzie jedno -- jak ustalić rozmiar strony i rozmiar bloku wymazywania dla
konkretnego modelu dysku SSD? Wygląda na to, że nie ma innej opcji jak zapytać producenta dysku SSD.
Ja wysłałem stosowne zapytanie do marki Goodram ale nie wiem czy mi w ogóle odpiszą. Jeśli tak by
się stało, to stosowne wartości dla tych parametrów tego konkretnego modelu dysku SSD zamieszczę w
tutaj.

### Parametry stride= oraz stripe_width= w przypadku nowszych dysków SSD

Przeglądając internet [natrafiłem na ten artykuł][26], który nieco podważa potrzebę stosowania tych
powyżej opisanych parametrów. Czytając dalej, [natknąłem się na ten wpis][27], w którym to możemy
wyczytać, że te powyższe parametry były krytyczne dla dysków SSD pierwszej generacji. W nowszych
dyskach SSD już raczej ma potrzeby ich stosowania, bo oprogramowanie tych nośników (i kernel
linux'a) jest nieco bardziej inteligentne i powinno sobie poradzić, nawet w przypadku, gdy tych
parametrów nie określimy. Niemniej jednak, wyrównanie systemu plików i dopasowanie go do struktury
komórek flash konkretnego dysku SSD nie zaszkodzi.

## Opcje systemu plików EXT4

Poniżej zostały zebrane te bardziej użyteczne opcje systemu plików EXT4, które są w stanie wpłynąć
w pozytywny sposób na działanie systemu operacyjnej oraz przy okazji odciążyć nieco nasz dysk SSD.

### Opcje relatime, atime, noatime, strictatime, nodiratime i lazytime

W metadanych systemu plików EXT4 jest przechowywanych szereg czasów, które są aktualizowane, gdy
jakieś operacje na plikach (np. utworzenie pliku czy jego zmiana) mają miejsce. W taki sposób mamy
dostępne cztery czasy: czas utworzenia pliku (Birth), czas ostatniego dostępu do pliku (Access,
atime), czas ostatniej modyfikacji pliku (Modify, mtime) i czas ostatniej zmiany pliku (Change,
ctime). Te dwa ostatnie brzmią podobnie ale różnica w nich tyczy się zmiany zawartości samego pliku
oraz zmiany i-węzła opisującego dany plik, czyli metadanych pliku, np. zmiana uprawnień w dostępie
do pliku. Te czasy można podejrzeć, np. w `stat` :

    $ stat plik.txt
      File: plik.txt
      Size: 41              Blocks: 8          IO Block: 4096   regular file
    Device: 253,4   Inode: 132350      Links: 1
    Access: (0644/-rw-r--r--)  Uid: ( 1000/  morfik)   Gid: ( 1000/  morfik)
    Access: 2024-01-05 14:33:50.468578496 +0100
    Modify: 2024-01-05 14:34:02.259752485 +0100
    Change: 2024-01-05 14:34:02.259752485 +0100
     Birth: 2024-01-05 14:33:50.468578496 +0100

Te powyższe czasy będą aktualizowane ilekroć tylko będziemy dokonywać jakichś operacji na plikach.
Czas utworzenia danego pliku w zasadzie nie ulega zmianie. Czasy ostatniej modyfikacji/zmiany są
aktualizowane przy zapisie danych na dysk. Natomiast czas ostatniego dostępu do pliku jest
uaktualniany za każdym razem, gdy plik jest odczytywany. Pojawia się tutaj pewien problem, bo przy
odczycie plików jest także dokonywany zapis danych na dysk i jeśli tych plików odczytujemy bardzo
dużo (ewentualnie też bardzo często), to sporo danych na dysku może zostać zapisanych, co może
skrócić żywotność dysku SSD, czy też bardzo degradować wydajność dysków HDD. Dlatego też by jakoś
poradzić sobie z tym problemem, zostały oddane użytkownikowi do dyspozycji [pewne opcje montowania
systemu plików EXT4][30], tj. `relatime` , `atime` , `noatime` , `strictatime` , `nodiratime` i
`lazytime` .

Poniżej jest krótkie wyjaśnienie tych opcji:

- `atime`       -- domyślne ustawienia kernela dla danego systemu plików (obecnie `relatime` ).
- `relatime`    -- aktualizuje czas ostatniego dostępu tylko w przypadku, gdy jest on wcześniejszy
                    niż aktualny czas modyfikacji/zmiany. Jeśli te czasy będą takie same lub czas
                    ostatniego dostępu będzie nowszy niż czas ostatniej modyfikacji/zmiany, to przy
                    ponownej próbie odczytu pliku nie nastąpi aktualizacja czasu ostatniego dostępu,
                    chyba że ten czas będzie starszy niż 1 dzień w stosunku do aktualnej daty.
- `noatime`     -- sprawia, że czas ostatniego dostępu nie będzie w ogóle aktualizowany (niektóre
                    aplikacje mogą przestać poprawnie funkcjonować).
- `nodiratime`  -- podobnie jak wyżej, z tą różnicą, że dla katalogów. Określenie opcji `noatime`
                    implikuje wykorzystanie `nodiratime` .
- `strictatime` -- aktualizuje czas ostatniego dostępu za każdym razem, gdy uzyskiwany jest dostęp
                    do pliku lub jego kopi w cache pamięci operacyjnej. Przydatne jedynie w
                    przypadku serwerów.
- `lazytime`    -- sprawia, że aktualizacja czasów (utworzenia, ostatniego dostępu i ostatniej
                    modyfikacji pliku) jest dokonywana jedynie w pamięci operacyjnej. Niemniej
                    jednak, aktualizacja tych czasów też będzie dokonywana na dysku ale tylko
                    w czterech przypadkach. Po pierwsze, gdy trzeba dokonać zmian w i-węźle, które
                    nie są powiązane z powyższymi znacznikami czasu. Po drugie, gdy zostanie
                    wywołane polecenie `sync` . Po trzecie, gdy nieskasowany i-węzeł zostanie
                    usunięty z pamięci operacyjnej. I po czwarte, jeśli minęło więcej niż 24 godziny
                    od ostatniego zapisu na dysk kopi znacznika czasowego obecnej w pamięci
                    operacyjnej. Opcja `lazytime` może być stosowana jako dodatek do wszystkich
                    pozostałych opcji.

Od już dłuższego czasu domyślnie wykorzystywana jest opcja `relatime` i w zasadzie jest to dobre
rozwiązanie wszędzie tam, gdzie ryzyko utraty zasilania wchodzi w grę. Niemniej jednak, jeśli
dysponujemy jakimś UPS'em, czy mamy laptopa, w którym jest obecna bateria, to możemy skorzystać
dodatkowo z opcji `lazytime` i zoptymalizować w ten sposób zachowanie kernela, by te znaczniki
czasów przechowywał w pamięci RAM i co jakiś czas je synchronizował sobie na dysk.

Opcję `lazytime` określamy w pliku `/etc/fstab/` , przykładowo:

    UUID=bcc4149e-7ec6-417c-97f4-23c5b6793450       /home            ext4    defaults,nodev,nosuid,noexec,lazytime,errors=remount-ro 0 2

### Opcja commit

Wszystkie zmiany w systemie plików są przechowywane w pamięci operacyjnej i dopiero po określonym
czasie są synchronizowane na dysk. Ten czas w przypadku systemu plików EXT4 wynosi 5 sekund. Tego
typu zachowanie ma poprawić wydajność, bo po co nieustannie zapisywać na dysku dane w małych
porcjach, gdy można chwilę poczekać i zapisać więcej danych w jednym podejściu.

Czas synchronizacji danych (i metadanych) systemu plików EXT4 możemy dostosować wykorzystując do
tego celu opcję `commit` . Ustawienie w jej argumencie niskiej wartości będzie degradować wydajność
ale sprawi, że dane będą bezpieczne. Ustawienie większych wartości poprawi wydajność ale w przypadku
utraty zasilania stracimy niezapisane dane z okresu, który tutaj ustawimy. Nie zaleca się ustawiać
więcej niż 300 sekund (5 minut).

Opcję `commit` również określamy w pliku `/etc/fstab` , przykładowo:

    UUID=bcc4149e-7ec6-417c-97f4-23c5b6793450       /home            ext4    defaults,nodev,nosuid,noexec,lazytime,commit=15,errors=remount-ro 0 2

## Kompresja danych w obrębie systemu plików

By zwiększyć żywotność komórek flash na dysku SSD, możemy skorzystać z kompresji danych oferowanej
przez system plików. Stosując kompresję danych będziemy mniej informacji zapisywać bezpośrednio na
nośniku SSD, oczywiście kosztem cykli procesora, bo operacje kompresji/dekompresji danych nie są za
free. Niestety system plików EXT4 nie wspiera póki co kompresji danych. Warto jednak wiedzieć, że
tego typu opcja dostępna jest w [systemie plików BTRFS][28].

## Czy hibernować system mając dysk SSD

Bardzo cenię sobie hibernację ze względu na fakt możliwości odtworzenia stanu pracy systemu po
odcięciu zasilania. Aktualnie mój laptop ma 16 GiB pamięci RAM, z czego zwykle połowa jest w użyciu.
Jeśli bym hibernował system, to z każdą hibernacją, te 8 GiB by było zapisywane na dysk. Widać
zatem, że hibernacja trochę mija się z celem w przypadku dysku SSD, bo dziennie, 20-50 GiB by szło
na jej obsługę i komórki flash by się dość szybko zużyły.

Biorąc pod uwagę fakt, że stary dysk HDD w laptopie mi został (w kieszeni w miejscu wyciągniętego
już dawno temu cd-rom'u), to kawałek tego dysku mogę przeznaczyć na SWAP -- dokładnie ten sam
kawałek, który robił za SWAP, gdy system działał na tym dysku. W zasadzie nic się tutaj nie
zmieni, nawet wpis od SWAP zostanie ten sam w pliku `/etc/fstab` :

    UUID=063c13f4-39b8-4965-bafd-7edb8b078774       swap             swap    defaults,pri=1024 0 0

Podobnie zawartość pliku `/etc/initramfs-tools/conf.d/resume` zostanie taka sama:

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

    tmpfs /tmp      tmpfs mode=1777,defaults,strictatime,size=50%,nr_inodes=1m 0 0

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

### Katalog /var/tmp/

Pod tym linkiem możemy zapoznać się z informacjami na temat [różnic między katalogiem /tmp/ i
/var/tmp/][19]. W zasadzie różnica jest bardzo subtelna i sprowadza się do tego czy pliki
tymczasowe powinny przetrwać cykl restartu systemu czy nie. Niestety deweloperzy różnych aplikacji
bardzo swobodnie podchodzą do tego tematu i pliki, które powinny powędrować do `/tmp/` są
umieszczane w `/var/tmp/` . Czy mając zatem dysk SSD powinniśmy zamontować w pamięci RAM również i
katalog `/var/tmp/` ?

Analizując zawartość katalogu `/var/tmp/` po uruchomieniu systemu, nie zauważyłem w nim w zasadzie
żadnych plików za wyjątkiem katalogów podobnych do `systemd-private-*.service-*/` , które są efektem
ustawienia w plikach usług opcji `PrivateTmp=true` . Ten mechanizm ma na celu poprawę
bezpieczeństwa, bo wszystkie pliki tymczasowe utworzone przez taką usługę zostaną usunięte po jej
zatrzymaniu. Można zatem przyjąć, że zawartość tych katalogów nie ma znaczenia z punktu widzenia
restartu systemu.

Niemniej jednak, są pewne polecenia, np. `update-initramfs` (a dokładniej `mkinitramfs` ) , które
domyślnie umieszcza pliki w katalogu `/var/tmp/` . Proces generowania takiego obrazu
initramfs/initrd zapisuje sporo danych na dysku tyko po to, by po jego zakończeniu przenieść
wynikowy plik do katalogu `/boot/` . Możemy co prawda zmienić zachowanie polecenia `mkinitramfs`
przez zastosowanie zmiennej `$TMPDIR` ale to wiązałoby się z przepisywaniem konfiguracji systemowej,
bo `update-initramfs` generuje obrazy initramfs/initrd przy aktualizacji systemu czy instalacji
nowego kernela.

Reasumując, ja na swoim systemie postanowiłem zamontować katalog `/var/tmp/` w RAM, mniej więcej w
ten sam sposób co w przypadku katalogu `/tmp/` .

#### Plik /etc/fstab

Poniżej jest stosowny wpis do umieszczenia w pliku `/etc/fstab` :

    tmpfs /var/tmp      tmpfs mode=1777,defaults,strictatime,size=50%,nr_inodes=1m 0 0

#### Plik usługi dla systemd

A niżej usługa dla systemd:

    # cat /etc/systemd/system/var-tmp.mount

    #  SPDX-License-Identifier: LGPL-2.1-or-later
    #
    #  This file is part of systemd.
    #
    #  systemd is free software; you can redistribute it and/or modify it
    #  under the terms of the GNU Lesser General Public License as published by
    #  the Free Software Foundation; either version 2.1 of the License, or
    #  (at your option) any later version.

    [Unit]
    Description=Temporary Directory /var/tmp
    Documentation=https://systemd.io/TEMPORARY_DIRECTORIES
    Documentation=man:file-hierarchy(7)
    Documentation=https://www.freedesktop.org/wiki/Software/systemd/APIFileSystems
    ConditionPathIsSymbolicLink=!/var/tmp
    DefaultDependencies=no
    Conflicts=umount.target
    Before=local-fs.target umount.target
    After=swap.target

    [Mount]
    What=tmpfs
    Where=/var/tmp
    Type=tmpfs
    Options=mode=1777,defaults,strictatime,size=50%,nr_inodes=1m

    [Install]
    WantedBy=local-fs.target

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
    ConditionPathIsSymbolicLink=!/var/cache/apt
    Conflicts=umount.target
    Before=local-fs.target umount.target
    After=swap.target

	[Mount]
	Where=/var/cache/apt
	What=tmpfs
	Type=tmpfs
    Options=mode=0755,defaults,strictatime,size=50%,nr_inodes=1m

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

    d /home/morfik/.cache/ 0700 morfik morfik 24h -
    d /var/cache/apt/ 0755 root root 2h -
    D /tmp/ 1777 root root 24h -

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

#### Parametry vm.swappiness i vm.vfs_cache_pressure

Jeśli zdecydujemy się na zaprzęgnięcie do pracy mechanizmu ZRAM i zrobimy sobie w oparciu o niego
przestrzeń wymiany SWAP, to dobrze jest też rzucić okiem na parametry `vm.swappiness` oraz
`vm.vfs_cache_pressure` , które to można określić w pliku `/etc/sysctl.conf` :

    vm.swappiness = 0
    vm.vfs_cache_pressure = 50

Parametr `vm.swappiness` określa tendencje do zrzucania nieużywanych (z punktu widzenia kernela)
danych z pamięci RAM do przestrzeni wymiany SWAP. Im wyższa jest wartość tego parametru (max 100,
od kernela 5.8 jest to 200), tym kernel chętniej to robi, co może czasem niebyt pozytywnie się
odbić na wydajności systemu. Ustawienie zbyt niskiej wartości może doprowadzić do przywieszania
się systemu w chwili wyczerpywania się dostępnej ilości pamięci operacyjnej. W przypadku
ustawienia "0", kernel nie będzie przenosił danych do SWAP do momentu aż ilość wolnych stron (free
and file-backed pages) będzie mniejsza niż wartość "high" w danej strefie pamięci (zone). Aktualną
wartość "high" można podejrzeć w pliku `/proc/zoneinfo` .

Te problemy z wydajnością dotyczą głównie przestrzeni wymiany ulokowanej na dysku HDD i gdy
korzystamy z mechanizmu ZRAM, to one nas praktycznie nie dotyczą. Niemniej jednak, trzeba rozsądnie
dobrać wartość parametru `vm.swappiness` w zależności od przeznaczenia systemu i tego ile mamy w
nim zainstalowanej (i do tego wolnej) pamięci RAM.

Jeśli zaś chodzi o parametr `vm.vfs_cache_pressure` , to trzeba zdawać sobie sprawę, że system
plików jest reprezentowany w pamięci za pomocą i-węzłów (i-node, index-node) oraz dentrów (dentry).
I-węzły reprezentują podległe im pliki lub katalogi, zaś dentry są to obiekty składające się z
nazwy, linku do i-węzła oraz linku do katalogu nadrzędnego, innymi słowy, jest to struktura drzewa
katalogów.  Przy przeglądaniu struktury plików i katalogów, dane na jej temat są buforowane w
pamięci RAM. Przy sporej ilości operacji dyskowych, ten cache pod i-wężły i dentry może zająć sporo
miejsca. Ustawienie niższych wartości w parametrze `vm.vfs_cache_pressure` sprawi, że kernel będzie
chciał zachować te dane w pamięci RAM dłużej. W przypadku ustawienia tego parametru na "0", nigdy
ich nie usunie, co może zakończyć się brakiem pamięci (OOM). Z kolei zwiększenie wartości (> 100)
sprawi, że kernel będzie się upominał o pamięć, którą zajmują i-węzły i dentry w przypadku sporego
obciążenia maszyny. Zwiększenie wartości >100 może odbić się negatywnie na wydajności maszyny.

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

## Dobór dyspozytora/planisty dla dysku SSD (scheduler)

Dyspozytor/planista dla operacji I/O to taki mechanizm, który ma na celu zoptymalizować dostęp do
danych na nośniku pamięci masowej. Spora część planistów I/O była projektowana w erze dysków HDD i
była przystosowana do pojedynczej kolejki I/O. W obecnych czasach z kolei, gdzie już powoli
zaczynają dominować dyski SSD, [ci planiści są już trochę bezużyteczni][32], bo taki dysk SSD
potrzebuje planisty zdolnego obsługiwać wiele kolejek I/O jednocześnie (Multi-Queue, MQ), by
wydajność nośnika nie uległa degradacji. Dlatego też w kernelu pojawiło się kilku nowych planistów
I/O czyniąc swoich poprzedników przestarzałymi.

W kernelu mamy czterech planistów I/O. Są nimi `none` , `mq-deadline` , `bfq` oraz `kyber` .
Aktualnie dostępnych w systemie planistów możemy sprawdzić w pliku
`/sys/block/sda/queue/scheduler` :

    $ cat /sys/block/sda/queue/scheduler
    none mq-deadline kyber [bfq]

Poniżej znajduje się krótki opis powyżej widocznych planistów I/O:

- `none`        -- zastąpił uprzednio używany `noop` . Jego [zadaniem jest][33] umieszczenie
                    zapytań w dowolnej kolejce I/O bez ingerowania w kolejność tych zapytań. Mówi
                    się, że czasami lepiej jest wybrać tego planistę ze względu na najmniejszy
                    narzut związany z obsługą takich kolejek I/O, co może poprawić wydajność dysków
                    SSD, w sytuacjach gdzie ilość IOPS ma dla nas ogromne znaczenie.
- `mq-deadline` -- zastąpił uprzednio używany `deadline` , który nie posiadał wsparcia dla wielu
                    kolejek I/O.
- `kyber`       -- jest takim `noop`'em z dodatkową optymalizacją dla bardzo szybkich dysków SSD.
- `bfq`         -- zastąpił uprzednio używany `cfq` (Completely Fair Queuing). Ten
                    planista [zalecany jest w przypadku dysków HDD][34], ze względu na dość znaczną
                    poprawę przepustowości tego typu nośników.

Biorąc pod uwagę te powyższe informacje, dla standardowych dysków SSD (tych z interfejsem
SATA/mSATA) powinniśmy ustawić zwykle `mq-deadline` . Dla dysków NVMe SSD powinniśmy wybrać albo
`kyber` , albo `none` . Natomiast `bfq` powinien zostać oddelegowany do pracy z dyskami HDD.

Wyboru planisty I/O dokonujemy zapisując plik `/sys/block/sda/queue/scheduler` . Można też
wykorzystać do tego celu plik `/etc/sysfs.conf` i dopisać w nim poniższą linijkę:

    block/sda/queue/scheduler = mq-deadline

Niemniej jednak, jeśli w systemie mamy wiele dysków SSD i HDD, to podczas startu systemu te literki
`sda` , `sdb` , etc mogą się zamieniać miejscami i lepszym rozwiązaniem jest napisanie sobie reguł
dla UDEV'a i umieścić je w pliku `/etc/udev/rules.d/95-iosched.rules` , przykładowo:

    ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
    ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"

Po dodaniu reguł, restartujemy politykę UDEV'a i wyzwalamy reguły bez potrzeby restartowania
systemu:

    # udevadm control --reload-rules
    # udevadm trigger --type=devices --action=change

Weryfikujemy czy planista I/O został prawidłowo wybrany:

    # cat /sys/block/sda/queue/scheduler
    none [mq-deadline] kyber bfq

## Fragmentacja danych na dysku SSD

Pliki na dysku SSD podlegają tym samym mechanizmom fragmentacji, co w przypadku dysków HDD. Czy
powinniśmy zatem raz na jakiś czas defragmentować pliki na dysku SSD? Wygląda na to, że
niekoniecznie. Czas dostępu do komórek flash jest taki sam dla wszystkich komórek na dysku SSD,
zatem co nam za różnica ile części ma dany plik i w którym miejscu nośnika taka część rezyduje?

Dodatkowo dyski SSD są projektowane w taki sposób, by dane były porozrzucane po całym nośniku,
zamiast je gromadzić w jednej części dysku, tak jak to ma miejsce w przypadku dysku HDD.
Porozrzucanie danych po całym dysku SSD ma na celu poprawić przepustowość takiego nośnika. Można to
zagadnienie zobrazować na przykładzie pendrive USB.

Każdy z nas miał styczność z pendrive podpinanym do portu USB. Komórki takiego pendrive są
dokładnie takim samym rodzajem pamięci flash co w przypadku dysków SSD. Niemniej jednak, jeśli
popatrzymy na wydajność pendrive i porównamy ją z wydajnością dysków SSD, to zobaczymy dość sporą
przepaść.

Co by się jednak stało, gdyby zebrać kilka czy kilkadziesiąt takich pendrive i upchnąć je w jednej
obudowie? Wtedy przepustowość takiego nośnika byłaby te kilkadziesiąt razy większa ale tylko w
przypadku, gdyby te dane zapisywać na wszystkie te nośniki jednocześnie.

I mniej więcej tak wygląda budowa dysku SSD, czyli posiada on kilka-kilkadziesiąt kontrolerów
kanałowych (channel controllers) i jeśli dane, które zamierzamy odczytać lub zapisać, będą
transferowane różnymi kanałami, to w tej samej jednostce czasu jesteśmy w stanie tych danych
przesłać więcej.

Dlatego też nie ma co sobie zaprzątać głowy ewentualną defragmentacją danych na dyskach SSD, bo
mija się ona z celem. Dodatkowo, zapis kilkudziesięciu czy kilkuset GiB danych tylko po to, by były
one obok siebie, nie jest warte skracania żywotności komórek flash.

## Test wydajności dysku SSD pod linux

Na zakończenie możemy jeszcze pokusić się o niedestruktywny test wydajności dysku SSD przy pomocy
`hdparm` . Poniżej przykład wydajności dysku SSD użytkownika @Jacekalex z forum DUG przed
zastosowaniem mechanizmu TRIM:

    # root ~> hdparm -tT /dev/sdb

    /dev/sdb:
     Timing cached reads:   21454 MB in  1.99 seconds = 10780.56 MB/sec
     Timing buffered disk reads: 108 MB in  3.01 seconds =  35.84 MB/sec

oraz po zastosowaniu TRIM:

    # root ~> hdparm -tT /dev/sdb

    /dev/sdb:
     Timing cached reads:   23136 MB in  1.99 seconds = 11630.34 MB/sec
     Timing buffered disk reads: 806 MB in  3.00 seconds = 268.31 MB/sec

Także różnica jest dość spora.

W przypadku mojego dysku SSD, ten krótki teścik prezentuje się następująco:

    # hdparm -tT /dev/sda

    /dev/sda:
     Timing cached reads:   12914 MB in  1.99 seconds = 6477.70 MB/sec
     Timing buffered disk reads: 1470 MB in  3.00 seconds = 489.98 MB/sec

Także jest trochę szybciej ale raczej jest to zaleta technologi MLC, w stosunku do TLC, która
została zastosowana w przypadku dysku użytkownika @Jacekalex.

## Problematyczny trim/discard w dyskach SSD na USB

Wygląda na to, że w przypadku dysków SSD podpiętych do portu USB komputera przy pomocy adaptera
USB-SATA (lub zewnętrznej obudowy USB), mechanizm TRIM jest domyślnie wyłączony, bo w takich
konfiguracjach może on doprowadzić do uszkodzenia danych lub też całkowitego uwalenia nośnika SSD.
Zwykle jednak, mechanizm TRIM w tego typu sytuacjach da radę bez obaw włączyć, choć trzeba mieć na
uwadze kilka istotnych kwestii. Zagadnienie [włączenia TRIM dla dysków SSD podpiętych po USB][35]
zostało opisane w osobnym artykule.

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
[19]: https://systemd.io/TEMPORARY_DIRECTORIES/
[20]: https://en.wikipedia.org/wiki/Multi-level_cell
[21]: https://www.anandtech.com/show/2738
[22]: https://arstechnica.com/information-technology/2012/06/inside-the-ssd-revolution-how-solid-state-disks-really-work/
[23]: https://www.techtarget.com/searchstorage/podcast/How-NAND-flash-degrades-and-what-vendors-do-to-increase-SSD-endurance
[24]: https://thelastmaimou.wordpress.com/2013/05/04/magic-soup-ext4-with-ssd-stripes-and-strides/
[25]: https://wiki.archlinux.org/title/Solid_state_drive#LVM
[26]: https://utcc.utoronto.ca/~cks/space/blog/linux/Ext4AndRAIDStripes
[27]: https://tytso.livejournal.com/2009/02/20/
[28]: https://btrfs.readthedocs.io/en/latest/Compression.html
[29]: https://www.storagesearch.com/ssdmyths-endurance.html
[30]: https://smarttech101.com/relatime-atime-noatime-strictatime-lazytime/
[31]: https://github.com/smartmontools/smartmontools/blob/198175f77e5e8332cdd4bcbd87a6e112979f4e32/smartmontools/drivedb.h
[32]: https://docs.kernel.org/block/blk-mq.html
[33]: https://en.wikipedia.org/wiki/Noop_scheduler
[34]: https://www.kernel.org/doc/html/latest/block/bfq-iosched.html
[35]: /post/trim-unmap-w-dyskach-ssd-podlaczonych-via-adapter-usb-sata-na-linux/
