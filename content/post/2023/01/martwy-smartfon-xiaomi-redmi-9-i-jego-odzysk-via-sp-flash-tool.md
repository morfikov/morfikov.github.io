---
author: Morfik
categories:
- Android
date:    2023-01-29 19:33:00 +0100
lastmod: 2023-01-29 19:33:00 +0100
published: true
status: publish
tags:
- smartfon
- xiaomi
- redmi-9
- crdroid
- spflashtool
- aosp
- firmware
- hard-brick
GHissueID: 594
title: Martwy smartfon Xiaomi Redmi 9 i jego odzysk via SP Flash Tool
---

Przeglądając ostatnio stronę `xiaomifirmwareupdater.com` zauważyłem, że jest tam [dostępna nowsza
wersja firmware dla mojego telefonu Xiaomi Redmi 9][1] (lancelot/galahad). Patrząc po numerkach
`V13.0.1.0.SJCEUXM` (nowy dla Android 12) oraz `V12.5.4.0.RJCEUXM` (stary dla Android 11), miałem
pewne wątpliwości czy wgrać sobie ten nowszy firmware. Niby na tym smartfonie mam wgrany [ROM
crDrdoid v8.9][2], który dostarcza Androida 12.1, więc [aktualizacja firmware sposobem opisanym
tutaj][3] powinna przebiec bez żadnych problemów. No i przebiegła, tylko po zrestartowaniu telefonu,
ten już się nie uruchomił. Działał mi jedynie tryb fastboot (normalny boot i tryb recovery były
martwe). Postanowiłem przywrócić poprzedni firmware wydobywając obrazy ze starej paczki firmware i
ręcznie przy pomocy narzędzia `fastboot` wgrać te obrazy na odpowiadające im partycje w telefonie z
poziomu mojego Debiana. Tutaj jednak zostało poczynionych parę błędów (o tym później), które
doprowadziły do całkowitego uwalenia telefonu (hard brick), gdzie nawet tryb fastboot zdechł. W
efekcie telefon już nie reagował na żadne kombinacje przycisków, a ekran pozostawał czarny -- jednym
słowem nie działał żaden tryb pracy telefonu i wyglądało na to, że mam w łapkach już tylko sam złom
elektroniczny i tak by w istocie było, gdyby z pomocą nie przyszedł mi SP Flash Tool.

<!--more-->
## Problematyczny firmware

Gdy trzymamy się ROM'u producenta telefonu, to praktycznie każdy element takiego oprogramowania
jest ze sobą zgrany i działa prawidłowo, przynajmniej powinien. W przypadku custom ROM'ów, nie do
końca musi być to prawdą, bo taki ROM można wgrać na telefon w dowolnym czasie jego użytkowania,
czyli może on posiadać starsze lub nowsze wersje oficjalnego oprogramowania. Dlatego też przy
przechodzeniu z oficjalnego ROM'u na ten nieoficjalny, deweloperzy informują użytkownika odnośnie
wymagań, które muszą być spełnione, by zmiana ROM'u przebiegła w miarę bezproblemowo i zakończyła
się powodzeniem. Jednym z tych wymagań jest [odpowiednia wersja firmware][8].

Na firmware składa się kilka obrazów, które są wgrywane na odpowiadające im partycje w telefonie.
Poniżej przykład dla mojego smartfona Xiaomi Redmi 9 (lancelot/galahad):

    $ patool list fw_lancelot_miui_LANCELOTEEAGlobal_V12.5.4.0.RJCEUXM_67a1671939_11.0.zip'
    patool: Listing fw_lancelot_miui_LANCELOTEEAGlobal_V12.5.4.0.RJCEUXM_67a1671939_11.0.zip ...
    patool: running /usr/bin/7z l -- fw_lancelot_miui_LANCELOTEEAGlobal_V12.5.4.0.RJCEUXM_67a1671939_11.0.zip

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz (306A9),ASM,AES-NI)

    Scanning the drive for archives:
    1 file, 40808894 bytes (39 MiB)

    Listing archive: fw_lancelot_miui_LANCELOTEEAGlobal_V12.5.4.0.RJCEUXM_67a1671939_11.0.zip

    --
    Path = fw_lancelot_miui_LANCELOTEEAGlobal_V12.5.4.0.RJCEUXM_67a1671939_11.0.zip
    Type = zip
    Physical Size = 40808894

       Date      Time    Attr         Size   Compressed  Name
    ------------------- ----- ------------ ------------  ------------------------
    2022-02-28 13:40:44 D....            0            0  META-INF
    2022-02-28 13:40:40 .....       280488       171992  preloader_raw.img
    2022-02-28 13:40:40 .....       282536       172052  preloader_ufs.img
    2022-02-28 13:40:42 .....            1            3  type.txt
    2022-02-28 13:40:40 .....          859          364  scatter.txt
    2022-02-28 13:40:40 .....       282536       172052  preloader_emmc.img
    2022-02-28 13:40:40 .....     59329408     35869684  md1img.img
    2022-02-28 13:40:42 .....      2505440      2166963  tee.img
    2022-02-28 13:40:42 .....        37984         7454  spmfw.img
    2022-02-28 13:40:40 .....       352816       144110  scp.img
    2022-02-28 13:40:42 .....       505616       483321  sspm.img
    2022-02-28 13:40:24 .....      1302976       522804  lk.img
    2022-02-28 13:40:22 D....            0            0  META-INF/com
    2022-02-28 13:40:44 .....         1634         1144  META-INF/CERT.RSA
    2022-02-28 13:40:42 .....         2217          999  META-INF/MANIFEST.MF
    2022-02-28 13:40:42 .....         2270         1091  META-INF/CERT.SF
    2022-02-28 13:40:42 D....            0            0  META-INF/com/android
    2022-02-28 13:40:22 D....            0            0  META-INF/com/google
    2022-02-28 13:40:24 D....            0            0  META-INF/com/google/android
    2022-02-28 13:40:24 .....      2340536      1090127  META-INF/com/google/android/update-binary
    2022-02-28 13:40:44 .....         3559          863  META-INF/com/google/android/updater-script
    2022-02-28 13:40:22 .....          316          220  META-INF/com/android/metadata
    2022-02-28 13:40:42 .....         1594         1077  META-INF/com/android/otacert
    ------------------- ----- ------------ ------------  ------------------------
    2022-02-28 13:40:44           67232786     40806320  18 files, 5 folders

Interesują nas głównie pliki `md1img.img` , `tee.img` , `spmfw.img` , `scp.img` , `sspm.img` oraz
`lk.img` . Nie jestem pewien czy obrazy `preloader_raw.img` , `preloader_ufs.img` i
`preloader_emmc.img` biorą udział w procesie aktualizacji firmware. Gdy jakiś już czas temu
aktualizowałem firmware przez TWRP/SHRP, to nie było w logu wzmianki o tym, by preloader był w
jakiś sposób ruszany.

Gdy podejrzymy sobie układ partycji w smartfonie, to zobaczymy coś takiego:

    # gdisk -l mmcblk0-stock-original.img
    GPT fdisk (gdisk) version 1.0.9

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.
    Disk mmcblk0-stock-original.img: 122142720 sectors, 58.2 GiB
    Sector size (logical): 512 bytes
    Disk identifier (GUID): 00000000-0000-0000-0000-000000000000
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 122142686
    Partitions will be aligned on 16-sector boundaries
    Total free space is 61 sectors (30.5 KiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1              64          131135   64.0 MiB    0700  recovery
       2          131136          132159   512.0 KiB   0700  misc
       3          132160          133183   512.0 KiB   0700  para
       4          133184          174143   20.0 MiB    0700  expdb
       5          174144          176191   1024.0 KiB  0700  frp
       6          176192          192575   8.0 MiB     0700  vbmeta
       7          192576          208959   8.0 MiB     0700  vbmeta_system
       8          208960          225343   8.0 MiB     0700  vbmeta_vendor
       9          225344          271631   22.6 MiB    0700  md_udc
      10          271632          337167   32.0 MiB    0700  metadata
      11          337168          402703   32.0 MiB    0700  nvcfg
      12          402704          533775   64.0 MiB    0700  nvdata
      13          533776          632079   48.0 MiB    0700  persist
      14          632080          730383   48.0 MiB    0700  persistbak
      15          730384          746767   8.0 MiB     0700  protect1
      16          746768          770047   11.4 MiB    0700  protect2
      17          770048          786431   8.0 MiB     0700  seccfg
      18          786432          790527   2.0 MiB     0700  sec1
      19          790528          796671   3.0 MiB     0700  proinfo
      20          796672          797695   512.0 KiB   0700  efuse
      21          797696          850943   26.0 MiB    0700  boot_para
      22          850944          982015   64.0 MiB    0700  nvram
      23          982016          998399   8.0 MiB     0700  logo
      24          998400         1260543   128.0 MiB   0700  md1img
      25         1260544         1262591   1024.0 KiB  0700  spmfw
      26         1262592         1274879   6.0 MiB     0700  scp1
      27         1274880         1287167   6.0 MiB     0700  scp2
      28         1287168         1289215   1024.0 KiB  0700  sspm_1
      29         1289216         1291263   1024.0 KiB  0700  sspm_2
      30         1291264         1324031   16.0 MiB    0700  gz1
      31         1324032         1356799   16.0 MiB    0700  gz2
      32         1356800         1360895   2.0 MiB     0700  lk
      33         1360896         1364991   2.0 MiB     0700  lk2
      34         1364992         1496063   64.0 MiB    0700  boot
      35         1496064         1528831   16.0 MiB    0700  dtbo
      36         1528832         1539071   5.0 MiB     0700  tee1
      37         1539072         1549311   5.0 MiB     0700  tee2
      38         1549312         1582079   16.0 MiB    0700  gsort
      39         1582080         1844223   128.0 MiB   0700  minidump
      40         1844224         2630655   384.0 MiB   0700  exaid
      41         2630656         4727807   1024.0 MiB  0700  cust
      42         4727808         4744191   8.0 MiB     0700  devinfo
      43         4744192         4767743   11.5 MiB    0700  ffu
      44         4767744        19447807   7.0 GiB     0700  super
      45        19447808        20332543   432.0 MiB   0700  cache
      46        20332544       122021823   48.5 GiB    0700  userdata
      47       122021824       122109887   43.0 MiB    0700  otp
      48       122109888       122142655   16.0 MiB    0700  flashinfo

Jeśli się przyjrzymy uważniej, to dostrzeżemy, że niektóre partycje są dublowane (np. `lk` oraz
`lk2` ) i ten fakt ma dość istotne znaczenie przy wgrywaniu firmware. Można by pomyśleć, że skoro
mamy do czynienia z plikiem obrazu `lk.img` , to powinniśmy wgrać go jedynie na partycję oznaczoną
`lk` . Niemniej jednak, trzeba ten obraz wgrać zarówno na partycję `lk` jak i `lk2` . Podobnie
trzeba postąpić z pozostałymi obrazami w paczce z firmware:

 - `md1img.img` -- wgrywamy na partycję `md1img` (nr. 24)
 - `tee.img`    -- wgrywamy na partycje `tee1` i `tee2` (nr. 36 i 37)
 - `spmfw.img`  -- wgrywamy na partycję `spmfw` (nr. 25)
 - `scp.img`    -- wgrywamy na partycje `scp1` i `scp2` (nr. 26 i 27)
 - `sspm.img`   -- wgrywamy na partycje `sspm_1` i `sspm_2` (nr. 28 i 29)
 - `lk.img`     -- wgrywamy na partycje `lk` i `lk2` (nr. 32 i 33)
 - `preloader_raw.img`  -- brak danych
 - `preloader_ufs.img`  -- brak danych
 - `preloader_emmc.img` -- brak danych

Obrazy `sspm_1` , `tee1` , `scp1` i `lk` odpowiadają za aktualizację głównego loader'a, zaś obrazy
`sspm_2` , `tee2` , `scp2` i `lk2` za aktualizację alternatywnego loader'a. Nie wiem co skrywa się
na partycjach `md1img` i `spmfw` .

Po aktualizacji firmware z `V12.5.4.0.RJCEUXM` do `V13.0.1.0.SJCEUXM` , działał jedynie tryb
fastboot i gdyby przy jego pomocy wgrać stare obrazy na swoje miejsca, to smartfon po takim zabiegu
powróciłby do życia. Póki co nie wiem, który z obrazów mających w nazwie `preloader_*` trzeba użyć
(o ile w ogóle się je używa do czegoś). Tak czy inaczej, brak wgrania starych obrazów preloader'a
i/lub wgranie obrazów tylko na jedną z dwóch bliźniaczych partycji sprawia, że telefon zdycha na
amen i jakakolwiek interakcja z nim przestaje być możliwa.

## BootROM i jego tryb pobierania (MTK Download Mode)

Brak wibracji przy przyciskaniu przycisków na obudowie telefonu może oznaczać w zasadzie tylko
jedną rzecz, a jest nią brak możliwości włączenia urządzenia, co potwierdza niedziałający
wyświetlacz czy brak na nim animacji ładowania po podłączeniu pod ładowarkę. Gdy taki telefon
podłączyłem do portu USB komputera, to w logu nie został zanotowany żaden komunikat.

Ja nie należę do osób, które łatwo się poddają, więc spróbowałem jakoś ten smartfon przywrócić do
życia. Jako, że w tym przypadku mamy do czynienia z SoC Mediatek, to była jakaś szansa na to, że
BootROM (pierwszy poziom procesu boot Androida) lub też i preloader (drugi poziom procesu boot
Androida) w dalszym ciągu funkcjonują prawidłowo.

Brak komunikatów w logu nie wróżył najlepiej ale przy przyciskaniu kolejno różnych kombinacji
przycisków natrafiłem na `Power + VolDown` . Ta kombinacja sprawiła w systemie pojawiło się nowe
urządzenie USB o numerkach `idVendor=0e8d` oraz `idProduct=0003` władające portem szeregowym
`/dev/ttyACM0` :

    kernel: usb 3-1: new high-speed USB device number 10 using xhci_hcd
    kernel: usb 3-1: New USB device found, idVendor=0e8d, idProduct=0003, bcdDevice= 1.00
    kernel: usb 3-1: New USB device strings: Mfr=0, Product=0, SerialNumber=0
    kernel: usb 3-1: Device is not authorized for usage
    kernel: cdc_acm 3-1:1.0: ttyACM0: USB ACM device
    kernel: usb 3-1: authorized to connect
    kernel: usb 3-1: USB disconnect, device number 10

To urządzenie znika z systemu praktycznie natychmiast po pojawieniu się, choć ten fakt nie ma tutaj
najmniejszego znaczenia. Ponowne przyciśnięcie `Power + VolDown` owocuje podobnym komunikatem na
konsoli. Jeśli taki komunikat się pojawi przy całkowitym uwaleniu telefonu, oznacza to, że BootROM
i jego Download Mode działa prawidłowo, co otwiera nam drogę do odratowania telefonu przy pomocy SP
Flash Tool.

## Błąd autoryzacji w SP Flash Tool i obejście go via MTK Bypass Utility

SP Flash Tool to narzędzie, które jest w stanie przesłać na telefon obrazy poszczególnych partycji
przy wykorzystaniu trybu pobierania (Download Mode). Wymagane jest posiadanie surowych obrazów
partycji oraz mapy flash'a telefonu, tj. pliku `scatter.txt` .  Ten plik można samemu napisać albo
można go pobrać z sieci. Ja go [pozyskałem z obrazu inżynieryjnego][4] (plik
`Redmi_9_Engineering_Rom.zip` ) ale można też posłużyć się [ROM'em fastboot][5].

Nawet jeśli mamy już wszystkie pliki przygotowane do wgrania, to nie będziemy mogli tego zrobić, bo
zwykle nie będziemy autoryzowani do przeprowadzania operacji niskopoziomowych, których realizację
umożliwia SP Flash Tool. Dlaczego? Wszystko przez niezbyt miłych ludzi, którzy kierując się tylko i
wyłącznie zyskiem [zaczęli działać na niekorzyść marki Xiaomi oraz jej klientów][9].

Chodzi o to, że telefony z rynku chińskiego były wyprowadzane poza Chiny i sprzedawane, np. w
Europie. Tu się pojawił problem, bo te urządzenia oferowały jedynie język chiński i angielski, co
zaowocowało wgrywaniem przez sprzedawców różnych modyfikacji ROM'ów wizualnie podobnych do
oficjalnego oprogramowania, gdzie wybór języków był sporo większy. Dodatkowo te ROM'y często nie
działały albo miały masę błędów, co prowadziło do wzrostu liczby niezadowolonych klientów, którzy
uważali, że kupili oficjalne urządzenie od Xiaomi i żądali często zwrotu hajsu za niedziałający
sprzęt.

W późniejszym czasie, sprzedawcy handlujący telefonami marki Xiaomi zaczęli się jeszcze bardziej
zamieniać z pewnym organem na łby i dorzucali do oferowanych przez siebie smartfonów syf w postaci
adware/spyware/spamware czy innego rodzaju malware. Za takie działania sprzedawcy dostawali
oczywiście hajs od firm trzecich (tych, które chciały implementacji tych śmieciowych aplikacji),
psując przy tym dobre imię marki Xiaomi, która w pewnym momencie postanowiła raz na zawsze takie
praktyki ukrócić podpisując szereg kluczowych elementów procesu startu telefonu.

W ten sposób zablokowano możliwość zdjęcia blokady bootloader'a (trzeba wysłać żądanie do Xiaomi i
posiadać zarejestrowane konto powiązane z danym urządzeniem), oraz też zablokowano tryb Emergency
Download Mode (dla SoC Qualcomm) czy Download Mode (dla SoC Mediatek), które bardzo przydają się do
odzyskiwania telefonu w takich przypadkach jak ten opisywany w niniejszym artykule.

Póki co nie mam pojęcia jak uzyskać autoryzację od Xiaomi, by móc bez problemów korzystać z SP
Flash Tool. Wygląda jednak na to, że nie trzeba już się prosić o pozwolenie, przynajmniej w
przypadku modelu Redmi 9 (i też kilku innych), bo paru [deweloperom udało się obejść mechanizm
autoryzacyjny][10] w BootROM (Boot Read-Only Memory), który [zwykle ładuje binarkę preloader'a][12]
co w następstwie uruchamia system Androida.

Owocem działania tych deweloperów jest projekt [MTK Bypass Utility][5]. By MTK Bypass Utility mógł
realizować swoje zadanie, potrzebuje określonego pliku wsadowego, konfiguracji oraz [drobnej
modyfikacji kernela linux w sterowniku USB][11]. Wygląda na to, że nowsze kernele (mam aktualnie
6.1.6) nie wymagają już tego patch'a na kernel. Jeśli zaś chodzi o pliki wsadowe i konfigurację,
to te dwie rzeczy są [dostępne w osobnym repozytorium][7].

Przygotujmy sobie zatem katalog roboczy:

    $ git clone https://github.com/MTK-bypass/bypass_utility
    $ cd bypass_utility/
    $ git clone https://github.com/MTK-bypass/exploits_collection
    $ cd exploits_collection/
    $ cp ./default_config.json5 ../
    $ cp -a ./payloads/ ../
    $ cd ..

Następnie odpalamy program:

    $ python3 main.py
    [2023-01-28 12:04:55.807367] Waiting for device

W tym momencie mając podłączony telefon do portu USB komputera przyciskamy przyciski `Power +
VolDown` . Po chwili powinniśmy zobaczyć w logu poniższe komunikaty:

    [2023-01-28 12:05:06.892077] Found device = 0e8d:0003

    [2023-01-28 12:05:07.012749] Device hw code: 0x707
    [2023-01-28 12:05:07.012871] Device hw sub code: 0x8a00
    [2023-01-28 12:05:07.012936] Device hw version: 0xca00
    [2023-01-28 12:05:07.012994] Device sw version: 0x0
    [2023-01-28 12:05:07.013076] Device secure boot: True
    [2023-01-28 12:05:07.013140] Device serial link authorization: True
    [2023-01-28 12:05:07.013232] Device download agent authorization: True

    [2023-01-28 12:05:07.013301] Disabling watchdog timer
    [2023-01-28 12:05:07.014062] Disabling protection
    [2023-01-28 12:05:07.038921] Protection disabled

Komunikat `Protection disabled` świadczy o obejściu procesu autoryzacji. Nasz smartfon nie zostanie
też rozłączony, przez co urządzenie `/dev/ttyACM0` będzie dostępne w systemie cały czas:

    # ls -al /dev/ttyACM0
    crw-rw----+ 1 root dialout 166, 0 2023-01-28 11:38:45 /dev/ttyACM0

## Konfiguracja interfejsu UART w SP Flash Tool

By móc wgrać coś na telefon w tym stanie, do którego go doprowadziliśmy, musimy nawiązać z nim
połączenie. SP Flash Tool jest w stanie nawiązać takie połączenie albo przez interfejs USB, albo
przez interfejs szeregowy UART. Dla naszych celów niezbędne będzie wykorzystanie interfejsu UART, a
że urządzenie `/dev/ttyACM0` nie znika nam już z systemu, to możemy skonfigurować to połączenie w
opcjach SP Flash Tool:

![smartfon-xiaomi-redmi-9-sp-flash-tool-uart-config](/img/2023/01/001.smartfon-xiaomi-redmi-9-sp-flash-tool-uart-config.png#huge)

### Wgrywanie obrazów firmware

Po obejściu procesu autoryzacji, możemy przystąpić do wgrania obrazów firmware na telefon
wykorzystując SP Flash Tool. Jeśli posiadamy ROM inżynieryjny, to możemy wykorzystać dostarczony w
nim pliki `DA` oraz plik `scatter.txt` . Wystarczy podać ścieżki do nich, tak jak to widać poniżej:

![smartfon-xiaomi-redmi-9-sp-flash-tool-da-scatter-txt-file](/img/2023/01/002.smartfon-xiaomi-redmi-9-sp-flash-tool-da-scatter-txt-file.png#huge)

Po wczytaniu pliku `scatter.txt` zostanie załadowana lista partycji, które będziemy mogli wgrać na
telefon. Możemy skorzystać z obrazów dostarczonych w ROM'ie inżynieryjnym. Możemy też dociągnąć ROM
fastboot i skorzystać z oferowanych przez niego obrazów. Możemy także skorzystać z obrazów partycji
zawartych w różnego rodzaju custom ROM'ach.

W tym przypadku prawie wszystkie obrazy pochodzą z oficjalnego ROM'u fastboot od Xiaomi (był chyba
nowszy niż ten inżynieryjny). Wyjątkiem są obrazy `dtbo` i `boot` , które pochodzą z ROM'u crDroid
(aktualnie wgrany na telefon).

#### Problem niezgodności SoC

Zanim jednak wgramy pełny zestaw obrazów, dobrze jest wgrać testowo jedną pozycję, by sprawdzić czy
komunikacja na linii SP Flash Tool i nasz telefon w ogóle działa. Podczas próby wgrania obrazów
wyskoczył mi komunikat niezgodności SoC (tym w telefonie i tym w konfiguracji), co objawiało się
poniższym wiadomością w logu:

    [error] Chip mismatch! scatter: platform[MT6768] type[]; device: hw_code[0xb8e8],
    hw_subcode[0x9400], hw_ver[0x7fb2], sw_ver[0x0], chip_evolution[0] #(chip_mapping.cpp, line:259)

A tak to wglądało w SP Flash Tool:

![smartfon-xiaomi-redmi-9-sp-flash-tool-chip-mismatch](/img/2023/01/003.smartfon-xiaomi-redmi-9-sp-flash-tool-chip-mismatch.png#huge)

Jeśli natrafiliśmy na taki problem po wciśnięciu przycisku `Download` w SP Flash Tool, to wystarczy
zignorować ten błąd i jeszcze raz przycisnąć przycisk `Download` . Tym razem proces przesyłania
obrazu na telefon powinien się rozpocząć i zakończyć powodzeniem.

![smartfon-xiaomi-redmi-9-sp-flash-tool-flashing](/img/2023/01/004.smartfon-xiaomi-redmi-9-sp-flash-tool-flashing.png#huge)

![smartfon-xiaomi-redmi-9-sp-flash-tool-flashing-success](/img/2023/01/005.smartfon-xiaomi-redmi-9-sp-flash-tool-flashing-success.png#huge)

### Przywrócenie poprzedniej wersji preloader'a

Po wgraniu wszystkich obrazów na swoje miejsca, telefon w dalszym ciągu nie chciał dać żadnego znaku
życia. Przeglądając skrypt aktualizacji dostarczany w ROM'ie fastboot ( `flash_all.sh` ),
zauważyłem, że widnieje tam poniższa linijka:

    fastboot $* flash preloader `dirname $0`/images/preloader_lancelot.bin

Zajrzałem zatem do katalogu `images/` i obok obrazów partycji znalazłem też obraz preloader'a, tj.
`preloader_lancelot.bin` . Różni się on nieco od tych trzech poprzednich, tj. `preloader_raw.img` ,
`preloader_ufs.img` czy `preloader_emmc.img` ale mając do wyboru ten plik lub tamte, to jednak
zdecydowałem się wgrać `preloader_lancelot.bin` .

Dopiero po wgraniu `preloader_lancelot.bin` (oraz wszystkich poprzednich obrazów partycji) telefon
się uruchomił. No i oczywiście żadne dane nie zostały stracone. Jedynie trzeba było ponownie
przepakować obraz partycji `/boot/` , by wgrać Magisk'a w celu uzyskania praw administratora
systemu.

## Podsumowanie

Opisana w tym artykule sytuacja do doskonały przykład na to jak producenci smartfonów świadomie
działają na niekorzyść swoich klientów. O ile pewne działania marki Xiaomi mające na celu
powstrzymać rozprzestrzenianie się syfu w produkowanych przez nich urządzeniach są nawet
uzasadnione, to pozbawienie przy tym końcowego użytkownika możliwości naprawy telefonu we własnym
zakresie jest moim zdaniem niedopuszczalne. Parę klików i 800 zł zostało w kieszeni, szkoda tylko,
że trzeba było dopuszczać się tak niecnych czynów jak przełamywanie zabezpieczeń. Nie nastąpiła też
żadna utrata danych użytkownika telefonu, co zapewne nie miałoby miejsca, gdyby oddać ten telefon
do serwisu.


[1]: https://xiaomifirmwareupdater.com/firmware/lancelot/
[2]: https://crdroid.net/lava/8
[3]: /post/jak-zaktualizowac-firmware-custom-rom-w-smartfonach-xiaomi/
[4]: https://sourceforge.net/projects/boxaltroms/files/mirror/
[5]: https://xiaomifirmwareupdater.com/miui/lancelot/
[6]: https://github.com/MTK-bypass/bypass_utility
[7]: https://github.com/MTK-bypass/exploits_collection
[8]: https://telegra.ph/Firmware-Download-Link-09-30
[9]: https://www.xda-developers.com/xiaomi-edl-unbrick-authorized-mi-accounts/
[10]: https://www.xda-developers.com/bypass-mediatek-sp-flash-tool-authentication-requirement/
[11]: https://github.com/amonet-kamakiri/kamakiri/blob/master/kernel.patch
[12]: https://research.nccgroup.com/2020/10/15/theres-a-hole-in-your-soc-glitching-the-mediatek-bootrom/
