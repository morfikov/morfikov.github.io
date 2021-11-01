---
author: Morfik
categories:
- Android
date:    2021-08-31 19:45:00 +0200
lastmod: 2021-08-31 19:45:00 +0200
published: true
status: publish
tags:
- xiaomi
- redmi-9
- recovery
- twrp
- magisk
- root
- smartfon
- safetynet
GHissueID: 338
title: Jak wgrać TWRP recovery i Magisk w Xiaomi Redmi 9 (galahad/lancelot)
---

Parę dni temu udało mi się [odblokować bootloader w moim smartfonie Xiaomi Redmi 9
(galahad/lancelot)][10]. Nie licząc błędnego URI przy logowaniu na konto Mi wewnątrz appki
XiaoMiTool, nie było zbytnio problemów z tym procesem. Takie odblokowanie bootloader'a w telefonie
w zasadzie nic nam samo z siebie nie daje, no może poza ściągnięciem z niego zabezpieczeń, co
ułatwia dostanie się syfu na Androida, no i też ułatwia złodziejom robotę, bo gdy takie odblokowane
urządzenie wpadnie w ich łapki, to mamy praktycznie pozamiatane. Niemniej jednak, odblokowany
bootloader daje nam możliwość wgrania custom recovery, np. TWRP, co z kolei otwiera nam drogę do
uzyskania praw administratora systemu root, np. za sprawą zainstalowania Magisk'a. Mając dostęp do
root, będziemy mogli takim urządzeniem dowolnie zarządzać. Naturalnie, TWRP daje nam też możliwość
wgrania alternatywnych ROM'ów na bazie AOSP/LineageOS ale w tym artykule skupimy się jedynie na
wrzuceniu TWRP recovery na tego Xiaomi Redmi 9 i ukorzenimy jego Androida przy pomocy wspomnianej już aplikacji Magisk.

<!--more-->
## Lancelot czy Galahad​

Technicznie rzecz biorąc, nazwa kodowa dla telefonu Xiaomi Redmi 9 to Lancelot. Niemniej jednak,
gdy się będziemy logować via ADB do systemu telefonu, to zobaczymy tam Galahad. Taki stan rzeczy
powoduje sporo nieporozumień, bo człowiek może pomyśleć, że Galahad i Lancelot to dwa różne
urządzenia. W rzeczywistości to tak naprawdę ten sam telefon, choć gdzieniegdzie można wyczytać, że
[Galahad to wersja z NFC, a Lancelot to wersja bez NFC][11] i to chyba jedyna rzecz, która różni te
smartfony. W przypadku wgrywania TWRP i uzyskiwania praw root za sprawą aplikacji Magisk, ten fakt
obecności NFC (lub jego braku) jest dla nas bez większego znaczenia. Może on mieć znaczenie przy
wgrywaniu custom ROM'ów ale tutaj możemy ten aspekt zwyczajnie pominąć.

Dodatkowo, jeśli zajrzymy w `/proc/cpuinfo` będąc zalogowanym via ADB na telefonie, to możemy tam
wyczytać, że Xiaomi Redmi 9 używa SoC MT6769T i zapewne tak jest w istocie. Problem jednak w tym,
że gdzie nie spojrzeć, to zamiast tego powyższego SoC, zwracany jest MT6768, nawet CPU-Z zwraca
MT6768 jako SoC. Początkowo szukałem konfiguracji dla MT6769T ale nic ciekawego nie znalazłem.
Natomiast bardziej owocne okazały się poszukiwania w kontekście MT6768. Dlatego też miejmy na
uwadze te drobne różnice, gdy będziemy szukać jakichś dodatkowych informacji odnośnie tego telefonu.

Poniżej znajduje się trochę informacji na temat Xiaomi Redmi 9, który pochodzi z oficjalnej
polskiej dystrybucji i został zakupiony w Black Friday w 2020 roku:

    galahad:/ # getprop  | grep -i model
    [ro.product.model]: [M2004J19C]
    [ro.product.odm.model]: [M2004J19C]
    [ro.product.product.model]: [M2004J19C]
    [ro.product.system.model]: [M2004J19C]
    [ro.product.vendor.model]: [M2004J19C]

    galahad:/ # getprop  | grep -i ro.build.version.
    [ro.build.version.base_os]: [Redmi/galahad_eea/galahad:10/QP1A.190711.020/V12.0.0.1.QJCEUXM:user/release-keys]
    [ro.build.version.incremental]: [V12.0.1.0.QJCEUXM]
    [ro.build.version.security_patch]: [2021-01-05]

    galahad:/ # getprop  | grep -i baseband
    [gsm.version.baseband]: [MOLY.LR12A.R3.MP.V98.P75,MOLY.LR12A.R3.MP.V98.P75]
    [ro.baseband]: [unknown]
    [vendor.gsm.project.baseband]: [HUAQIN_Q0MP1_MT6769_SP(LWCTG_CUSTOM)]

    $ fastboot getvar all
    ...
    (bootloader) product: lancelot
    ...
    (bootloader) version-baseband: MOLY.LR12A.R3.MP.V98.P75
    (bootloader) version-bootloader: lancelot-2b1e22f-20201123162228-2021011
    (bootloader) version-preloader:
    (bootloader) version: 0.5
    ...

## Odblokowany bootloader

Zanim w ogóle zaczniemy myśleć o przeprowadzeniu procesu flash'owania partycji recovery w
smartfonie Xiaomi Redmi 9 (Galahad/Lancelot) obrazem TWRP, musimy posiadać odblokowany bootloader
w tym urządzeniu. Nie jest to jakiś skomplikowany proces, bo istnieją stosowne narzędzia jak [Mi
Unlock][4] czy [XiaoMiTool][5], które do tego celu można wykorzystać, odpowiednio na windows i
linux.

Jako, że parę dni temu opisałem [cały proces ściągania blokady z bootloader'a w Xiaomi Redmi 9][10],
to nie będę tutaj poruszał tej kwestii. Jeśli jednak nie mamy jeszcze odblokowanego bootloader'a,
to zachęcam do zapoznania się z tym podlinkowanym wyżej artykułem w celu nadrobienia braków. Tak
czy inaczej, w ustawieniach deweloperskich powinniśmy mieć `Bootloader is already unlocked` w `OEM
unlocking` oraz `Unlocked` w `Mi Unlock status` , czyli mniej więcej to, co widnieje na poniższej
fotce:

![xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-unlocked-bootloader](/img/2021/08/001-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-unlocked-bootloader.jpg#small)

Gdy bootloader w naszym telefonie został już odblokowany, to możemy kontynuować proces wgrywania
TWRP i ukorzeniania Androida.

## Własny obraz TWRP recovery

Na necie można spotkać się z szeregiem [artykułów na temat uzyskiwania praw root i wgrywania
TWRP][1] recovery na telefon Xiaomi Redmi 9 (Galahad/Lancelot). Problem z tymi wszystkimi tekstami
jest taki, że za dużo w nich jest fraz typu "nie wiem" czy "nie mam pojęcia". Poza tym, same pliki
często też są niewiadomego pochodzenia, czyli nie były budowane z określonych plików źródłowych. Po
prostu, ktoś skopiował beż żadnej refleksji taki obraz TWRP recovery i wrzucił do artykułu.

Przestrzegałbym przed wgrywaniem tego typu obrazów czy plików i nie chodzi tutaj już o samo
uwalenie smartfona (bo zwykle takie obrazy działają), a raczej o wgranie sobie jakiegoś
niepożądanego syfu, który później będzie już bardzo ciężko usunąć, o ile w ogóle będzie to możliwe.

Mając na uwadze powyższe, postanowiłem zrobić sobie własny obraz TWRP i wgrać go na partycję
recovery.

### Budowanie obrazu TWRP ze źródeł

Nic nie stoi na przeszkodzie by sobie wgrać na partycję recovery pierwszy lepszy obraz TWRP
znaleziony gdzieś na necie. Na [forum XDA utworzyłem wątek][12], który stosowny plik zawiera, także
nie trzeba go pobierać z jakichś innych podejrzanych miejsc. Zachęcam jednak do zbudowania własnego
obrazu, bo stosowna [konfiguracja dla Xiaomi Redmi 9][2] istnieje.

Jakiś czas temu opisywałem dokładnie [proces budowania obrazu TWRP][13] ze [źródeł OMNI ROM][3] i co
ciekawe praktycznie wszystkie informacje zawarte w tym artykule wciąż są w mocy. Zmieniła się
jedynie wersja Androida z 6 na 10, no i też narzędzie `repo` jest dostępne w repozytorium Debiana.
Dlatego też te potrzebne polecenia z tego podlinkowanego artykułu zamieszczę niżej, a po
wyjaśnienie co i jak odsyłam do samego artykułu.

    $ mkdir omni-twrp-10.0/ && cd omni-twrp-10.0/
    $ repo init -u git://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-10.0 --depth=1
    $ repo sync --current-branch --jobs=4
    $ mkdir -p device/xiaomi/
    $ git clone https://github.com/Redmi-MT6768/omni_device_xiaomi_lancelot lancelot

Określamy urządzenie, dla którego będziemy budować TWRP recovery:

    $ make clobber
    $ . ./build/envsetup.sh
    $ lunch

    You're building on Linux

    Lunch menu... pick a combo:
         1. aosp_arm-eng
         2. aosp_arm64-eng
         3. aosp_x86-eng
         4. aosp_x86_64-eng
         5. omni_emulator-userdebug
         6. omni_lancelot-eng
         7. omni_lancelot-userdebug

    Which would you like? [aosp_arm-eng] omni_lancelot-userdebug

    ============================================
    PLATFORM_VERSION_CODENAME=REL
    PLATFORM_VERSION=16.1.0
    TARGET_PRODUCT=omni_lancelot
    TARGET_BUILD_VARIANT=userdebug
    TARGET_BUILD_TYPE=release
    TARGET_ARCH=arm64
    TARGET_ARCH_VARIANT=armv8-2a
    TARGET_CPU_VARIANT=generic
    TARGET_2ND_ARCH=arm
    TARGET_2ND_ARCH_VARIANT=armv8-2a
    TARGET_2ND_CPU_VARIANT=generic
    HOST_ARCH=x86_64
    HOST_2ND_ARCH=x86
    HOST_OS=linux
    HOST_OS_EXTRA=Linux-5.13.11-amd64-x86_64-Debian-GNU/Linux-bookworm/sid
    HOST_CROSS_OS=windows
    HOST_CROSS_ARCH=x86
    HOST_CROSS_2ND_ARCH=x86_64
    HOST_BUILD_TYPE=release
    BUILD_ID=QQ3A.200805.001
    OUT_DIR=/media/Android/omni-twrp-10.0/out
    ROM_BUILDTYPE=HOMEMADE
    ============================================

I teraz już wystarczy podać w `make` target `recoveryimage` :

    $ make clean && make -j2 recoveryimage
    02:10:56 Entire build directory removed.

    #### build completed successfully (6 seconds) ####

    ============================================
    PLATFORM_VERSION_CODENAME=REL
    PLATFORM_VERSION=16.1.0
    TARGET_PRODUCT=omni_lancelot
    TARGET_BUILD_VARIANT=userdebug
    TARGET_BUILD_TYPE=release
    TARGET_ARCH=arm64
    TARGET_ARCH_VARIANT=armv8-2a
    TARGET_CPU_VARIANT=generic
    TARGET_2ND_ARCH=arm
    TARGET_2ND_ARCH_VARIANT=armv8-2a
    TARGET_2ND_CPU_VARIANT=generic
    HOST_ARCH=x86_64
    HOST_2ND_ARCH=x86
    HOST_OS=linux
    HOST_OS_EXTRA=Linux-5.13.11-amd64-x86_64-Debian-GNU/Linux-bookworm/sid
    HOST_CROSS_OS=windows
    HOST_CROSS_ARCH=x86
    HOST_CROSS_2ND_ARCH=x86_64
    HOST_BUILD_TYPE=release
    BUILD_ID=QQ3A.200805.001
    OUT_DIR=/media/Android/omni-twrp-10.0/out
    ROM_BUILDTYPE=HOMEMADE
    ============================================
    ...
    [100% 20282/20282] ----- Making recovery image ------

    #### build completed successfully (18:47 (mm:ss)) ####

Po dłuższej chwili, w katalogu `omni-twrp-10.0/out/target/product/lancelot/` powinien znajdować się
plik `recovery.img` i to jest właśnie ten obraz TWRP, który musimy wgrać na partycję recovery w
naszym telefonie.

### Wgrywanie obrazu TWRP na telefon

Gdy pozyskaliśmy już obraz TWRP (wszystko jedno czy pobraliśmy go z internetu, czy budowaliśmy go
własnoręcznie), trzeba go wgrać na partycję recovery w naszym telefonie Xiaomi Redmi 9
(Galahad/Lancelot) przy pomocy fastboot. W dystrybucji Debian wystarczy doinstalować pakiet
`fastboot` . Gdy mamy już fastboot w systemie, wyłączamy telefon i uruchamiamy go trzymając
przyciśnięte przyciski volumeDown+power. Podłączamy telefon do portu USB komputera i wydajemy w
terminalu poniższe polecenie:

    $ fastboot flash recovery omni-twrp-10.0/out/target/product/lancelot/recovery.img

Warto zaznaczyć tutaj, że taki proces wgrywania obrazu TWRP recovery jest jedynie tymczasowy. Gdy
uruchomimy ponownie telefon w normalnym trybie, to obraz recovery zostanie przez system urządzenia
automatycznie odtworzony i wgrany ponownie na partycję recovery. Dlatego też, po wgraniu obrazu
TWRP na partycję recovery, musimy uruchomić telefon w trybie recovery:

    $ fastboot reboot recovery

Po tym jak smartfon zostanie uruchomiony w trybie recovery, zapyta nas o hasło:

![xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-password](/img/2021/08/002-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-password.jpg#small)

To hasło jest dokładnie tym samym hasłem, które wykorzystujemy w przypadku odblokowania ekranu. W
przypadku wpisania prawidłowego hasła, uzyskamy pełny dostęp do telefonu z poziomu TWRP (wliczając
w to uprawnienia root):

|   |   |
|---|---|
| ![](/img/2021/08/003-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-password-encryption.png#small) | ![](/img/2021/08/004-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp.png#small) |

## Utworzenie kopi zapasowej flash'a telefonu

Ten tymczasowy tryb TWRP recovery przyda nam się do stworzenia backup'u całego flash'a telefonu.
Jedyną partycją, w której poczyniliśmy jakieś zmiany, to właśnie partycja recovery. Inne partycje
pozostają póki co nietknięte. Tworząc kopię zapasową flash'a telefonu, utworzymy także backup
partycji, które przechowują numery IMEI, adresy MAC WiFi/BT, dane kalibracyjne dla modemu GSM, czy
też inne wrażliwe informacje, bez których urządzenie nie będzie działać poprawnie. Mając backup w
sytuacji, gdy coś pójdzie nie tak, bez większego problemu będziemy mogli przy pomocy fastboot i
obrazów partycji przywrócić telefon do stanu, który był tuż po odblokowaniu jego bootloader'a.

Backup całego flash'a telefonu wykonujemy przy pomocy `adb` , gdy telefon jest uruchomiony w trybie
TWRP recovery i podłączony przy tym do komputera via port USB:

    $ adb pull /dev/block/mmcblk0 mmcblk0.img
    /dev/block/mmcblk0: 1 file pulled. 14.0 MB/s (62537072640 bytes in 4266.682s)

To powyższe polecenie wydajemy z poziomu komputera, a nie z poziomu telefonu. Oczywiście, nic nie
stoi na przeszkodzie by skorzystać z `dd` przechodząc do terminala w TWRP i skopiować flash
telefonu na zewnętrzną kartę SD. Mój telefon ma jedynie kartę SD 32G i kopia całego flash'a, który
waży 64G, się zwyczajnie na tę kartę nie zmieści. Poza tym, lepiej jest przechowywać backup flash'a
telefonu na komputerze, bo będzie nam potrzebny w późniejszym czasie.

Proces robienia backup'u telefonu jest dość powolny. By zgrać cały flash, potrzebne będzie nam czas
około 2 godzin (prędkość około 14M/s). Po tym jak ten proces dobiegnie końca, sprawdzamy czy flash
pobrał się w całości. Do tego celu potrzebne nam będzie jedynie narzędzie `gdisk` , które powinno
być już standardowo zainstalowane chyba w każdej dystrybucji linux'a:

    # gdisk -l mmcblk0.img
    GPT fdisk (gdisk) version 1.0.7

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.
    Disk mmcblk0.img: 122142720 sectors, 58.2 GiB
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

Jak widać, mamy tutaj zapis układu całego flash'a wraz z jego partycjami w niezmienionym stanie, no
może poza wyjątkiem partycji recovery. Jeśli coś pójdzie nie tak, zawsze można wyodrębnić każdą z
tych widocznych wyżej partycji. Trzeba tylko zamontować ten obraz flash'a telefonu wykorzystując do
tego celu rządzenia `loop` (być może będzie trzeba odpowiednio [skonfigurować sobie moduł
loop][14]):

    # losetup /dev/loop5 /media/Zami/mmcblk0.img
    # losetup -a
    /dev/loop5: [64769]:12 (/media/Zami/mmcblk0.img)

Obraz flash'a telefonu wykorzystuje w tym przypadku urządzenie `/dev/loop5` . Jako, że w tym
obrazie mamy partycje, to każda z nich dostanie osobne urządzenie `/dev/loop5p*` , gdzie `*`
oznacza numerek partycji w obrazie:

    # ls -al /dev/loop5*
    brw-rw---- 1 root disk 7, 320 2021-08-29 02:54:11 /dev/loop5
    brw-rw---- 1 root disk 7, 321 2021-08-29 02:54:11 /dev/loop5p1
    brw-rw---- 1 root disk 7, 330 2021-08-29 02:54:11 /dev/loop5p10
    brw-rw---- 1 root disk 7, 331 2021-08-29 02:54:11 /dev/loop5p11
    brw-rw---- 1 root disk 7, 332 2021-08-29 02:54:11 /dev/loop5p12
    brw-rw---- 1 root disk 7, 333 2021-08-29 02:54:11 /dev/loop5p13
    brw-rw---- 1 root disk 7, 334 2021-08-29 02:54:11 /dev/loop5p14
    brw-rw---- 1 root disk 7, 335 2021-08-29 02:54:11 /dev/loop5p15
    brw-rw---- 1 root disk 7, 336 2021-08-29 02:54:11 /dev/loop5p16
    brw-rw---- 1 root disk 7, 337 2021-08-29 02:54:11 /dev/loop5p17
    brw-rw---- 1 root disk 7, 338 2021-08-29 02:54:11 /dev/loop5p18
    brw-rw---- 1 root disk 7, 339 2021-08-29 02:54:11 /dev/loop5p19
    brw-rw---- 1 root disk 7, 322 2021-08-29 02:54:11 /dev/loop5p2
    brw-rw---- 1 root disk 7, 340 2021-08-29 02:54:11 /dev/loop5p20
    brw-rw---- 1 root disk 7, 341 2021-08-29 02:54:11 /dev/loop5p21
    brw-rw---- 1 root disk 7, 342 2021-08-29 02:54:11 /dev/loop5p22
    brw-rw---- 1 root disk 7, 343 2021-08-29 02:54:11 /dev/loop5p23
    brw-rw---- 1 root disk 7, 344 2021-08-29 02:54:11 /dev/loop5p24
    brw-rw---- 1 root disk 7, 345 2021-08-29 02:54:11 /dev/loop5p25
    brw-rw---- 1 root disk 7, 346 2021-08-29 02:54:11 /dev/loop5p26
    brw-rw---- 1 root disk 7, 347 2021-08-29 02:54:11 /dev/loop5p27
    brw-rw---- 1 root disk 7, 348 2021-08-29 02:54:11 /dev/loop5p28
    brw-rw---- 1 root disk 7, 349 2021-08-29 02:54:11 /dev/loop5p29
    brw-rw---- 1 root disk 7, 323 2021-08-29 02:54:11 /dev/loop5p3
    brw-rw---- 1 root disk 7, 350 2021-08-29 02:54:11 /dev/loop5p30
    brw-rw---- 1 root disk 7, 351 2021-08-29 02:54:11 /dev/loop5p31
    brw-rw---- 1 root disk 7, 352 2021-08-29 02:54:11 /dev/loop5p32
    brw-rw---- 1 root disk 7, 353 2021-08-29 02:54:11 /dev/loop5p33
    brw-rw---- 1 root disk 7, 354 2021-08-29 02:54:11 /dev/loop5p34
    brw-rw---- 1 root disk 7, 355 2021-08-29 02:54:11 /dev/loop5p35
    brw-rw---- 1 root disk 7, 356 2021-08-29 02:54:11 /dev/loop5p36
    brw-rw---- 1 root disk 7, 357 2021-08-29 02:54:11 /dev/loop5p37
    brw-rw---- 1 root disk 7, 358 2021-08-29 02:54:11 /dev/loop5p38
    brw-rw---- 1 root disk 7, 359 2021-08-29 02:54:11 /dev/loop5p39
    brw-rw---- 1 root disk 7, 324 2021-08-29 02:54:11 /dev/loop5p4
    brw-rw---- 1 root disk 7, 360 2021-08-29 02:54:11 /dev/loop5p40
    brw-rw---- 1 root disk 7, 361 2021-08-29 02:54:11 /dev/loop5p41
    brw-rw---- 1 root disk 7, 362 2021-08-29 02:54:11 /dev/loop5p42
    brw-rw---- 1 root disk 7, 363 2021-08-29 02:54:11 /dev/loop5p43
    brw-rw---- 1 root disk 7, 364 2021-08-29 02:54:11 /dev/loop5p44
    brw-rw---- 1 root disk 7, 365 2021-08-29 02:54:11 /dev/loop5p45
    brw-rw---- 1 root disk 7, 366 2021-08-29 02:54:11 /dev/loop5p46
    brw-rw---- 1 root disk 7, 367 2021-08-29 02:54:11 /dev/loop5p47
    brw-rw---- 1 root disk 7, 368 2021-08-29 02:54:11 /dev/loop5p48
    brw-rw---- 1 root disk 7, 325 2021-08-29 02:54:11 /dev/loop5p5
    brw-rw---- 1 root disk 7, 326 2021-08-29 02:54:11 /dev/loop5p6
    brw-rw---- 1 root disk 7, 327 2021-08-29 02:54:11 /dev/loop5p7
    brw-rw---- 1 root disk 7, 328 2021-08-29 02:54:11 /dev/loop5p8
    brw-rw---- 1 root disk 7, 329 2021-08-29 02:54:11 /dev/loop5p9

By wydobyć przykładową partycję `boot` z tego powyższego obrazu, wpisujemy w terminal poniższe
polecenie:

    # dd if=/dev/loop5p34 of=./34-stock-boot.img

W taki sposób możemy wydobyć dowolną partycję i zapisać ją w pliku. Taki plik możemy następnie
podać w fastboot lub też bezpośrednio wgrać na flash przy pomocy `dd` z poziomu TWRP recovery.
Także tak długo, jak mamy działający tryb fastboot lub TWRP recovery, to nie powinniśmy popsuć
trwale telefonu. Oczywiście pewne wpadki mogą nam się przytrafić ale powinny być one jedynie
tymczasowe do momentu wgrania stock'owych obrazów. Dlatego też miejmy na uwadze to, które partycje
zmieniamy, by później bez problemu móc je odtworzyć.

## Bootloop za sprawą Magisk

Podsumowując, mamy zrobiony backup całego flash'a telefonu, wgraliśmy tymczasowo TWRP na partycję
recovery, no i mamy też uruchomiony telefon w trybie TWRP recovery. Teraz przyszedł czas by wgrać
Magisk z poziomu TWRP recovery i ukorzenić naszego Xiaomi Redmi 9 (Galahad/Lancelot).

Nie spieszmy się jednak z tym wgrywaniem Magisk'a via TWRP, bo gdybyśmy to teraz zrobili, to nasz
telefon podczas startu złapie bootloop i się nie będzie chciał już uruchomić. Wszystko przez
[mechanizm zwany Android Verified Boot][8] (w skrócie AVB), który ciągle działa, nawet po
odblokowaniu bootloader'a w telefonie. Ten mechanizm AVB ma na celu weryfikowanie hash'ów różnych
partycji, głównie partycji boot. W ten sposób, producent telefonu może określić, które obrazy
partycji boot będą mogły zostać wykorzystane przy starcie telefonu.

Wgrywając Magisk'a, dokonujemy zmian w partycji boot, zmian, które nie będą autoryzowane przez
Xiaomi, czego efektem będzie zapętlenie się telefonu podczas jego startu. Dodatkowo, stracimy
możliwość wejścia w tryb recovery, by ewentualnie móc cofnąć poczynione zmiany przy wgrywaniu
Magisk'a. By z takiego bootloop' wybrnąć, trzeba wgrać stock'owy obraz partycji boot. Dlatego
właśnie ogromne znaczenie miało utworzenie backup'u całego flash'a telefonu. Z nim odratowanie
urządzenia jest banalnie proste. Jeśli chodzi nam po głowie wgranie obrazu partycji boot z innego
telefonu, to lepiej uważać, bo efekty mogą być nieprzewidywalne (przez różny software/firmware/ROM),
choć w sumie jak telefon i tak nam się nie chce uruchomić, to co nam zależy.

### Jak wyłączyć Android Verified Boot (AVB)

By uniknąć takiej bardzo nieprzyjemnej sytuacji, jaką jest bootloop telefonu, musimy dezaktywować
mechanizm AVB zanim będziemy próbowali wgrać na telefon Magisk'a. Ten mechanizm dezaktywuje się bez
większego problemu, gdy mamy ściągniętą blokadę bootloader'a. Wystarczy przy pomocy fastboot
przesłać na telefon stock'owy obraz partycji vbmeta podając dwa dodatkowe parametry:

    # dd if=/dev/loop5p6 of=./6-stock-vbmeta.img

    $ fastboot --disable-verity --disable-verification flash vbmeta 6-stock-vbmeta.img

Po wyłączeniu mechanizmu AVB, można przejść do wgrywania Magisk'a z poziomu TWRP recovery. Jeśli
nasz telefon przywrócił w międzyczasie stock'ową partycję recovery, to trzeba będzie jeszcze raz
wgrać obraz TWRP na nią.

### Instalacja Magisk'a z pliku APK

Tak czy inaczej, uruchamiamy telefon w trybie TWRP recovery i pobieramy najnowszą dostępną wersję
Magisk'a (obecnie jest to [Magisk-v23.0.apk][9]). Tak, wiem, że jest to plik APK, i tak, trzeba go
wgrać przez TWRP recovery, dokładnie w taki sam sposób jak instaluje się inny pliki przez TWRP
(zwykle `.zip` ). Podczas procesu instalowania Magisk'a na ekranie powinny pojawić się komunikaty
mówiące o tym, że partycja boot jest przepakowywana i wgrywana ponownie:

|   |   |
|---|---|
| ![](/img/2021/08/005-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-magisk-install.png#small) | ![](/img/2021/08/006-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-magisk-install-log.png#small) |

To jest ten moment, w którym telefon przestanie przepisywać partycję recovery podczas startu. Po
zainstalowaniu się Magisk'a, partycja recovery z obrazem TWRP nie będzie już zmieniana i nie będzie
trzeba wgrywać obrazu TWRP kolejny raz. Czyścimy również cache.

Po wgraniu pliku APK, uruchamiamy telefon ponownie w zwykłym trybie, by dokończyć instalację
Magisk'a z poziomu Androida. Po uruchomieniu się systemu, zostaniemy poproszeni o kontynuowanie
procesu instalacji Magisk'a, więc podążamy za instrukcjami.

### Obejście SafetyNet

Gdy proces instalacji Magisk'a z poziomu Androida dobiegnie końca, uruchamiamy aplikację Magisk i
sprawdzamy jak wygląda status SafetyNet. Nasz telefon z początku nie będzie w stanie przejść tych
wszystkich testów, które wchodzą w skład SafetyNet, o czym zostaniemy poinformowani odpowiednim
komunikatem:

|   |   |
|---|---|
| ![](/img/2021/08/007-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-magisk-app.jpg#small) | ![](/img/2021/08/008-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-safetynet-fail.jpg#small) |

By tę sytuację poprawić, wchodzimy w ustawienia Magisk'a i ukrywamy aplikację ( `Hide the Magisk
app` ) oraz włączamy `MagiskHide` :

|   |   |
|---|---|
| ![](/img/2021/08/009-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-magisk-settings.jpg#small) | ![](/img/2021/08/010-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-magisk-settings.jpg#small) |

Sprawdzamy ponownie SafetyNet i tym razem już powinniśmy go bez problemu przejść:

![xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-safetynet-success](/img/2021/08/011-xiaomi-redmi-9-galahad-lancelot-root-magisk-twrp-safetynet-success.jpg#small)

## Podsumowanie

Podobnie jak w przypadku zdejmowania blokady z bootloader'a, wgranie TWRP recovery i Magisk'a w
telefonie Xiaomi Redmi 9 (galahad/lancelot) oraz uzyskanie praw administratora systemu root nie
stanowiło zbytnio większego problemu. Największą trudność stwarza zbudowanie samego obrazu TWRP,
który trzeba wgrać na partycję recovery. Nie trzeba przy tym tworzyć całej konfiguracji dla TWRP od
zera, bo już ktoś ten krok poczynił za nas. Jeśli nie chce nam się budować TWRP, to naturalnie
można skorzystać już ze zbudowanego obrazu. Trzeba jednak pamiętać, by wyłączyć mechanizm Android
Verified Boot (AVB), który jeśli pozostanie aktywny przy wgrywaniu Magisk'a, uwali nam telefon.
Ważnym też jest, by wykonać backup całego flash'a telefonu przy pomocy `adb` i trzymać go w
bezpiecznym miejscu na wypadek jakichś problemów z telefonem. Tak ukorzeniony Android przechodzi
testy SafetyNet bez większego problemu, zatem można na tym urządzeniu korzystać ze wszystkich
aplikacji, które wymagają by urządzenie było w stanie nienaruszonym, np. aplikacje bankowe.


[1]: https://forum.xda-developers.com/t/guide-twrp_3-5-2-magisk-root-access-for-redmi-9-galahad.4286789/
[2]: https://github.com/Redmi-MT6768/omni_device_xiaomi_lancelot
[3]: https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni
[4]: https://en.miui.com/unlock/
[5]: https://xiaomitool.com/V2/
[6]: https://github.com/francescotescari/XiaoMiToolV2/issues/23
[7]: https://morfikov.github.io/post/jak-odblokowac-bootloader-w-xiaomi-redmi-9-galahad-lancelot/#xiaomi-procedure-failed-getservicetoken-missing-servicetoken-cookie
[8]: https://android.googlesource.com/platform/external/avb/+/1614f552a53d9dcd85ff404156422d935ebdb1a4/README.md
[9]: https://github.com/topjohnwu/Magisk/releases
[10]: /post/jak-odblokowac-bootloader-w-xiaomi-redmi-9-galahad-lancelot/
[11]: https://forum.xda-developers.com/t/redmi-9-with-nfc-which-rom-can-i-download.4214295/
[12]: https://forum.xda-developers.com/t/guide-how-to-unlock-and-root-xiaomi-redmi-9-galahad-lancelot.4326097/
[13]: /post/budowanie-obrazu-twrp-recovery-ze-zrodel-omni-rom/
[14]: /post/obsluga-wielu-partycji-w-module-loop/
