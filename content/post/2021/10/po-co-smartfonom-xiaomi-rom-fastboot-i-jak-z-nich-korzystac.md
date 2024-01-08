---
author: Morfik
categories:
- Android
date:    2021-10-05 00:47:00 +0200
lastmod: 2024-01-08 11:45:00 +0100
published: true
status: publish
tags:
- smartfon
- xiaomi
- redmi-9
- fastboot
- root
- miui
GHissueID: 335
title: Po co smartfonom Xiaomi ROM'y fastboot i jak z nich korzystać
---

Przeglądając jakiś czas temu oficjalną stronę Xiaomi w poszukiwaniu nowszych wersji oficjalnego
ROM'u na mój smartfon Redmi 9, zauważyłem, że są tam dostępne [instrukcje na temat wgrania][1]
takiego oprogramowania za pomocą trybu `fastboot`. Trochę się zdziwiłem, bo przecie ROM'y
dostarczane są w paczkach `.zip` , przez co nie są one przeznaczone do wgrywania w tym trybie. Tak
czy inaczej, zgodnie z informacją, która widnieje na tamtej stronie, te linki do obrazów fastboot
jeszcze nie zostały wypuszczone, przynajmniej oficjalnie. Nieoficjalnie zaś można je pobrać ze
strony [xiaomifirmwareupdater.com][2]. To, co się rzuca od razu w oczy, to rozmiar takiego pliku,
bo standardowo pliki z ROM'em MIUI ważą około 2,2 GiB. ROM'y fastboot mają rozmiar około 4,5 GiB.
Kolejna sprawa, to rozszerzenie samego pliku. W przypadku standardowego ROM'u mamy `.zip` , a ROM'y
fastboot mają już rozszerzenie `.tar.gz` lub `.tgz` .  Nie mogłem przejść obojętnie obok tej
zagadki i postanowiłem sprawdzić, co taka paczka w sobie zawiera i do czego ewentualnie ona może
nam się przydać w kontekście alternatywnego oprogramowania wrzucanego na telefon.

<!--more-->
## Zawartość ROM'u fastboot

Wszystkie alternatywne ROM'y na bazie AOSP/LineageOS zalecają, czy też wręcz wymagają, od nas by
przed instalacją takiego oprogramowania, w telefonie znajdował się stock'owy ROM w określonej
wersji, zwykle MIUI v12.0.1.0. Biorąc pod uwagę taki stan rzeczy, postanowiłem zaciągnąć na swój
komputer właśnie ten najbardziej pożądany ROM, tj. plik
[lancelot_eea_global_images_V12.0.1.0.QJCEUXM_20210119.0000.00_10.0_eea_a197df7d17.tgz][3]. Po
wypakowaniu tej paczki, jej zawartość prezentuje następująco:

    $ tree -pugsh lancelot_eea_global_images_V12.0.1.0.QJCEUXM_20210119.0000.00_10.0_eea
    lancelot_eea_global_images_V12.0.1.0.QJCEUXM_20210119.0000.00_10.0_eea
    ├── [drwxr-xr-x morfik   morfik   4.0K]  AP
    │   ├── [-rw-r--r-- morfik   morfik   191K]  APDB_MT6768_S01__W2006
    │   └── [-rw-r--r-- morfik   morfik    20K]  APDB_MT6768_S01__W2006_ENUM
    ├── [drwxr-xr-x morfik   morfik   4.0K]  BP
    │   ├── [-rw-r--r-- morfik   morfik   3.5M]  DbgInfo_LR12A.R3.MP_HUAQIN_Q0MP1_MT6769_SP_MOLY_LR12A_R3_MP_V98_P75_2020_12_18_14_28_1_ulwctg_n
    │   ├── [-rw-r--r-- morfik   morfik   211K]  MDDB.META.ODB_MT6768_S00_MOLY_LR12A_R3_MP_V98_P75_1_ulwctg_n.XML.GZ
    │   ├── [-rw-r--r-- morfik   morfik   310K]  MDDB.META_MT6768_S00_MOLY_LR12A_R3_MP_V98_P75_1_ulwctg_n.EDB
    │   └── [-rw-r--r-- morfik   morfik    16M]  MDDB_InfoCustomAppSrcP_MT6768_S00_MOLY_LR12A_R3_MP_V98_P75_1_ulwctg_n.EDB
    ├── [drwxr-xr-x morfik   morfik   4.0K]  BP_IN
    ├── [-rwxr-xr-x morfik   morfik    40K]  CheckSum_Gen
    ├── [drwxr-xr-x morfik   morfik   4.0K]  Log
    │   └── [-rw-r--r-- morfik   morfik    204]  ADPT_20210119-212711_0.log
    ├── [-rwxr-xr-x morfik   morfik     99]  check_sum.sh
    ├── [-rwxr-xr-x morfik   morfik   2.7K]  flash_all.bat
    ├── [-rwxr-xr-x morfik   morfik   3.3K]  flash_all.sh
    ├── [-rwxr-xr-x morfik   morfik   2.6K]  flash_all_except_data_storage.bat
    ├── [-rwxr-xr-x morfik   morfik   3.2K]  flash_all_except_data_storage.sh
    ├── [-rwxr-xr-x morfik   morfik   3.1K]  flash_all_lock.bat
    ├── [-rwxr-xr-x morfik   morfik   3.6K]  flash_all_lock.sh
    ├── [-rwxr-xr-x morfik   morfik   5.3K]  flash_gen_crc_list.py
    ├── [-rwxr-xr-x morfik   morfik   1.3K]  flash_gen_md5_list.py
    ├── [-rwxr-xr-x morfik   morfik   842K]  flash_gen_resparsecount
    ├── [-rwxr-xr-x morfik   morfik    989]  hat_extract.py
    ├── [-rwxr-xr-x morfik   morfik    591]  hat_flash.sh
    ├── [drwxr-xr-x morfik   morfik   4.0K]  images
    │   ├── [-rw-r--r-- morfik   morfik    422]  Checksum.ini
    │   ├── [-rw-r--r-- morfik   morfik    20K]  MT6768_Android_scatter.txt
    │   ├── [-rw-r--r-- morfik   morfik    64M]  boot.img
    │   ├── [-rw-r--r-- morfik   morfik   176K]  cache.img
    │   ├── [-rw-r--r-- morfik   morfik    245]  crclist.txt
    │   ├── [-rw-r--r-- morfik   morfik   652M]  cust.img
    │   ├── [-rw-r--r-- morfik   morfik    16M]  dtbo.img
    │   ├── [-rw-r--r-- morfik   morfik    512]  efuse.img
    │   ├── [-rw-r--r-- morfik   morfik   200M]  exaid.img
    │   ├── [-rw-r--r-- morfik   morfik    16K]  gsort.img
    │   ├── [-rw-r--r-- morfik   morfik   1.6M]  lk.img
    │   ├── [-rw-r--r-- morfik   morfik   1.5M]  logo.bin
    │   ├── [-rw-r--r-- morfik   morfik    56M]  md1img.img
    │   ├── [-rw-r--r-- morfik   morfik    16K]  oem_misc1.img
    │   ├── [-rw-r--r-- morfik   morfik   270K]  preloader_lancelot.bin
    │   ├── [-rw-r--r-- morfik   morfik    64M]  recovery.img
    │   ├── [-rw-r--r-- morfik   morfik   344K]  scp.img
    │   ├── [-rw-r--r-- morfik   morfik     56]  sparsecrclist.txt
    │   ├── [-rw-r--r-- morfik   morfik    37K]  spmfw.img
    │   ├── [-rw-r--r-- morfik   morfik   493K]  sspm.img
    │   ├── [-rw-r--r-- morfik   morfik   4.1G]  super.img
    │   ├── [-rw-r--r-- morfik   morfik   2.3M]  tee.img
    │   ├── [-rw-r--r-- morfik   morfik   1.2G]  userdata.img
    │   ├── [-rw-r--r-- morfik   morfik   4.0K]  vbmeta.img
    │   ├── [-rw-r--r-- morfik   morfik   4.0K]  vbmeta_system.img
    │   └── [-rw-r--r-- morfik   morfik   4.0K]  vbmeta_vendor.img
    ├── [-rw-r--r-- morfik   morfik   1.6M]  libflashtool.so
    ├── [-rw-r--r-- morfik   morfik   7.0M]  libflashtool.v1.so
    ├── [-rw-r--r-- morfik   morfik   6.6M]  libflashtoolEx.so
    ├── [-rw-r--r-- morfik   morfik   2.2K]  md5sum.xml
    └── [-rw-r--r-- morfik   morfik     81]  misc.txt

Zatem mamy tutaj parę skryptów, no i też jest szereg obrazów `.img` , a konkretnie jest ich aż 22,
jeśli dobrze policzyłem. Jest tam też plik o bardzo dźwięcznej nazwie, tj. `flash_all.sh` . Jako,
że jest to skrypt shell'owy, to podejrzałem go sobie, by zobaczyć co takiego on skrywa, choć po
jego nazwie nietrudno zgadnąć. Sam skrypt nie jest jako rozbudowany, więc cała jego treść jest
poniżej:

    fastboot $* getvar product 2>&1 | grep -E "^product: *lancelot$"
    if [ $? -ne 0 ] ; then echo "error : Missmatching image and device"; exit 1; fi

    CURRENT_ANTI_VER=2
    version=`fastboot getvar rollback_ver 2>&1 | grep "rollback_ver:" | awk -F ": " '{print $2}'`
    if [  "${version}"x == ""x ] ; then version=0 ; fi
    if [ ${version} -gt ${CURRENT_ANTI_VER} ] ; then  echo "error : current device antirollback #version is greater than this package" ; exit 1 ; fi

    #fastboot $* erase boot
    #if [ $? -ne 0 ] ; then echo "Erase boot error"; exit 1; fi
    fastboot $* flash preloader `dirname $0`/images/preloader_lancelot.bin
    if [ $? -ne 0 ] ; then echo "Flash preloader error"; exit 1; fi
    #fastboot $* flash efuse `dirname $0`/images/efuse.img
    #if [ $? -ne 0 ] ; then echo "Flash efuse error"; exit 1; fi
    fastboot $* flash logo `dirname $0`/images/logo.bin
    if [ $? -ne 0 ] ; then echo "Flash logo error"; exit 1; fi
    fastboot $* flash tee1 `dirname $0`/images/tee.img
    if [ $? -ne 0 ] ; then echo "Flash tee1 error"; exit 1; fi
    fastboot $* flash tee2 `dirname $0`/images/tee.img
    if [ $? -ne 0 ] ; then echo "Flash tee2 error"; exit 1; fi
    fastboot $* flash scp1 `dirname $0`/images/scp.img
    if [ $? -ne 0 ] ; then echo "Flash scp1 error"; exit 1; fi
    fastboot $* flash scp2 `dirname $0`/images/scp.img
    if [ $? -ne 0 ] ; then echo "Flash scp2 error"; exit 1; fi
    fastboot $* flash sspm_1 `dirname $0`/images/sspm.img
    if [ $? -ne 0 ] ; then echo "Flash sspm_1 error"; exit 1; fi
    fastboot $* flash sspm_2 `dirname $0`/images/sspm.img
    if [ $? -ne 0 ] ; then echo "Flash sspm_2 error"; exit 1; fi
    fastboot $* flash lk `dirname $0`/images/lk.img
    if [ $? -ne 0 ] ; then echo "Flash lk error"; exit 1; fi
    fastboot $* flash lk2 `dirname $0`/images/lk.img
    if [ $? -ne 0 ] ; then echo "Flash lk2 error"; exit 1; fi
    fastboot $* flash super `dirname $0`/images/super.img
    if [ $? -ne 0 ] ; then echo "Flash super error"; exit 1; fi
    fastboot $* flash cache `dirname $0`/images/cache.img
    if [ $? -ne 0 ] ; then echo "Flash cache error"; exit 1; fi
    fastboot $* flash recovery `dirname $0`/images/recovery.img
    if [ $? -ne 0 ] ; then echo "Flash recovery error"; exit 1; fi
    fastboot $* flash boot `dirname $0`/images/boot.img
    if [ $? -ne 0 ] ; then echo "Flash boot error"; exit 1; fi
    fastboot $* flash dtbo `dirname $0`/images/dtbo.img
    if [ $? -ne 0 ] ; then echo "Flash dtbo error"; exit 1; fi
    fastboot $* flash vbmeta `dirname $0`/images/vbmeta.img
    if [ $? -ne 0 ] ; then echo "Flash vbmeta error"; exit 1; fi
    fastboot $* flash spmfw `dirname $0`/images/spmfw.img
    if [ $? -ne 0 ] ; then echo "Flash spmfw error"; exit 1; fi
    fastboot $* flash md1img `dirname $0`/images/md1img.img
    if [ $? -ne 0 ] ; then echo "Flash md1img error"; exit 1; fi
    fastboot $* flash vbmeta_system `dirname $0`/images/vbmeta_system.img
    if [ $? -ne 0 ] ; then echo "Flash vbmeta_system error"; exit 1; fi
    fastboot $* flash vbmeta_vendor `dirname $0`/images/vbmeta_vendor.img
    if [ $? -ne 0 ] ; then echo "Flash vbmeta_vendor error"; exit 1; fi
    fastboot $* flash cust `dirname $0`/images/cust.img
    if [ $? -ne 0 ] ; then echo "Flash cust error"; exit 1; fi
    fastboot $* flash exaid `dirname $0`/images/exaid.img
    if [ $? -ne 0 ] ; then echo "Flash exaid error"; exit 1; fi
    fastboot $* flash userdata `dirname $0`/images/userdata.img
    if [ $? -ne 0 ] ; then echo "Flash userdata error"; exit 1; fi
    fastboot $* reboot
    if [ $? -ne 0 ] ; then echo "Reboot error"; exit 1; fi

Jak możemy zauważyć, mamy tutaj szereg wywołań narzędzia `fastboot` , które wrzuca stosowne obrazy
zawarte w pobranej przez nas paczce na odpowiadające im partycje w telefonie. Zatem już wiemy skąd
w tym ROM'ie wzięła się nazwa fastboot.

## Do czego może nam się przydać ROM fastboot

Po zawartości paczki możemy stwierdzić, że są tam w zasadzie gołe obrazy, które wgrywane są na
smartfon przy pomocy narzędzia `fastboot` . By móc jednak skorzystać z ROM'u fastboot, potrzebny
nam będzie [odblokowany bootloader (tutaj artykuł na przykładzie Redmi 9)][7]. Zakładając, że
blokada z bootloader'a została już zdjęta, to nasuwa się jedno pytanie. Co przy pomocy takiego
ROM'u fastboot jesteśmy w stanie zrobić z naszym telefon od Xiaomi?

### Zmiana regionu z EEA na Global

Przy pomocy ROM'u fastboot jesteśmy w stanie wgrać oficjalny ROM Xiaomi, który nie jest
przeznaczony na nasze urządzenie, tj. chodzi o region, w którym taki telefon został dopuszczony do
użytku. Na terenie Europy smartfony mają wgrany ROM z regionem `EEA`, który z racji pewnych
europejskich standardów w kwestii prywatności, jest pozbawiony części funkcjonalności. Jeśli
prywatność dla nas nie ma znaczenia lub też bardziej cenimy sobie funkcjonalność urządzenia, to
moglibyśmy wgrać sobie ROM z regionem `Global` .

### Downgrade ROM'u

Zakładając, że Xiaomi wypuścił nowszą wersję ROM'u na nasz smartfon, oraz że ta aktualizacja
zawiera nowszego Androida, który nie działa za dobrze na tym urządzeniu, np. z powodu zbyt słabej
wydajności, to przy pomocy ROM'u fastboot będziemy w stanie zrobić downgrade do starszej wersji,
tj. tej, która u nas zdawała się działać lepiej.

### Możliwość odtworzenia stanu fabrycznego ROM'u

ROM fastboot powstał w zasadzie, by dać nam możliwość przywrócenia fabrycznego oprogramowania w
naszym smartfonach Xiaomi, w sytuacji gdy coś pochrzanimy, np. bawiąc się alternatywnym softem. Być
może są również i inne zastosowania dla tego ROM'u ale nas, tj. osoby korzystające z custom ROM'ów,
bardziej interesuje aspekt popsucia urządzenia wskutek wgrania trefnych obrazów, czy też
poczynienia zbyt śmiałych zmian w konfiguracji samego telefonu.

#### Odtworzenie partycji /boot/ i /recovery/

W pobranej przez nas paczce `.tgz` znajdują się obrazy partycji `/boot/` oraz `/recovery/` . Jeśli
korzystamy z TWRP albo SHRP, to one przepisują zawartość partycji `/recovery/` . System telefonu
zwykle jest w stanie tę partycję odtworzyć sobie sam jeśli wykryje w niej zmiany ale tylko, gdy
partycja `/boot/` pozostaje nietknięta. Zwykle jednak po wgraniu TWRP/SHRP, wrzucany jest na
telefon także Magisk (w celu uzyskania praw root), który modyfikuje partycję `/boot/` na swoje
potrzeby. Może się zatem zdarzyć tak, że któraś z tych partycji nam się uszkodzi i jeśli nie mamy
zrobionego [backup'u flash'a telefonu][5], to przy pomocy ROM'u fastboot będziemy w stanie te dwie
partycje przywrócić do stanu fabrycznego i potencjalnie przywrócić nasz smartfon do życia.

#### Odtworzenie ustawień mechanizmu AVB

Następna rzecz to mechanizm AVB (Android Verified Boot), przy którym zwykle coś majstrujemy, by móc
wgrać na smartfon TWRP/SHRP recovery lub też alternatywne ROM'y. Chodzi generalnie o zmiany
dokonywane w obrazach `vbmeta` , `vbmeta_system` oraz `vbmeta_vendor` . Może się zatem zdarzyć tak,
że nie będziemy w stanie tego mechanizmu włączyć ponownie, np. gdyby nam się już znudziło bawienie
otwartym oprogramowaniem, lub też gdy zwyczajnie chcielibyśmy to urządzenie komuś odsprzedać.

#### Przywrócenie oryginalnego oprogramowania

Tej paczce z ROM'em fastboot mamy też naturalnie obraz całej partycji `/super/` , w której skład
wchodzi partycja `/system/` , `/product/` oraz `/vendor/` . Mamy zatem tutaj stock'owe
oprogramowanie, na wypadek, gdybyśmy chcieli w późniejszym czasie do niego powrócić. Naturalnie nie
tylko partycja `/super/` jest potrzebna do odtworzenia fabrycznego oprogramowania ale to tutaj
znajduje się Android, z którego tak energicznie korzystamy na co dzień.

#### Naprawa partycji /data/ oraz /cache/

W paczce z ROM'em fastboot mogliśmy też ujrzeć obrazy partycji `/data/` oraz `/cache/` . Nie do
końca wiem po co one zostały tam umieszczone. Intuicja mi jednak podpowiada, że może chodzić o
sytuacje, w których użytkownik coś namiesza podczas zabawy z alternatywnym ROM'em, przez co system
może mieć jakieś problemy z umieszczeniem danych na tych dwóch partycjach. Być może te obrazy są
przeznaczone do fabrycznego odtworzenia ich zawartości (systemu plików, szyfrowania, etc.) i tym
samym usunięcia wszelkich zaistniałych problemów powstałych z winy użytkownika.

Raz zdarzyło mi się tak namieszać w partycji `/data/` , że by znów telefon zaczął działać poprawnie,
to trzeba było skorzystać właśnie z tego ROM'u fastboot. Pewnie szło wszystkie zmiany cofnąć
ręcznie, choć czasami jest to trudne, zwłaszcza w przypadku, gdy do końca się nie wie co się
popsuło i w jaki sposób.

#### Aktualizacja/downgrade firmware

Po rzuceniu okiem na partycje, które są przepisywane podczas flash'owania telefonu tym ROM'em
fastboot, widać, że [aktualizacji podlega między innymi cały firmware urządzenia][4], tj. pozycje
`/spmfw/` , `/scp1/` , `/scp2/` , `/sspm_1/` , `/sspm_2/` , `/tee1/` , `/tee2/` , `/lk/` , `/lk2/`
oraz `/md1img/` . Zatem już mamy kolejny sposób zastosowania tego ROM'u, tj. jeśli z jakiegoś
powodu potrzebujemy powrócić do określonej wersji firmware, to jak najbardziej wgranie tej paczki
na smartfon nam to umożliwi. Podobnie też i w drugą stronę, tj. gdy firmware chcielibyśmy
zaktualizować.

Naturalnie, gdy chodzi o proces wgrywania firmware, to można go przeprowadzić niezależnie od
wgrywania reszty ROM'u (szczegóły w podlinkowanym wyżej artykule). Niemniej jednak, warto wiedzieć,
że po wgraniu ROM'u fastboot, firmware również zostanie zmieniony.

#### Aktualizacja/downgrade preloader'a

Podobnie jak w przypadku firmware, preloader również może zostać zaktualizowany i podobnie możemy
przy pomocy ROM'u fastboot zmienić jego wersję na tę, która była obecna we wcześniejszych
oficjalnych ROM'ach od Xiaomi.

## Gdzie ROM fastboot nie znajdzie zastosowania

Praktycznie każda z tych 22 partycji może zostać odtworzona do stanu fabrycznego po skorzystaniu z
tego ROM'u fastboot. Trzeba jednak pamiętać, że Android na swoim flash'u ma tych partycji koło 50,
zatem ta paczka nie będzie nam w stanie zawsze pomóc ale w sporej większości przypadków na pewno
znajdzie zastosowanie.

### Utrata IMEI, MAC WiFi i Bluetooth

Jednym z takich kluczowych problemów, na które użytkownik smartfona może trafić podczas zabawy z
alternatywnymi ROM'ami, to uszkodzenie krytycznych z punktu widzenia funkcjonalności numerów
urządzenia, takich jak np. IMEI, adresy MAC kart WiFi/Bluetooth czy też dane kalibracyjne do modemu
GSM. Te dane są specyficzne dla danego urządzenia i z reguły [przechowywane na poniższych
partycjach][6]:

 - `/nvcfg/`     -- przechowuje zmienną konfigurację dla `/nvdata/` i `/nvram/`
 - `/nvdata/`    -- przechowuje zmienne informacje identyfikacyjne danego urządzenia, m.in. IMEI,
                  WiFi MAC, Bluetooth MAC, dane kalibracyjne.
 - `/nvram/`     -- przechowuje niezmienne informacje identyfikacyjne danego urządzenia, m.in. IMEI,
                  WiFi MAC, Bluetooth MAC, dane kalibracyjne.
 - `/persist/`   -- przechowuje niezmienne dane użyteczne dla mechanizmu FRP (Factory Reset
                  Protection), takie jak dane do konta Google czy MIaccount/MIcloud.
 - `/protect_f/` -- przechowuje zmienne dane od ustawień SIM/RADIO/MODEM/BASEBAND.
 - `/protect_s/` -- przechowuje zmienne dane od ustawień SIM/RADIO/MODEM/BASEBAND.

Jako, że żadna z tych partycji nie znajduje się w ROM'ie fastboot, to ich uszkodzenie, wyzerowanie
czy nadpisanie w inny sposób może uczynić nasz telefon złomem elektronicznym, przynajmniej jeśli
chodzi o dzwonienie i korzystanie z internetu. Dlatego też ROM fastboot nie będzie nam w stanie
pomóc w przypadku utraty IMEI czy adresów MAC. Zawsze możemy zrobić sobie [backup IMEI czy
adresów MAC WiFi i Bluetooth we własnym zakresie][5], tak by w razie czego być w stanie przywrócić
te numerki.

### Telefon się nie uruchamia

ROM fastboot, jak nazwa sugeruje, wymaga, by w naszym telefonie działał tryb fastboot. Jeśli nasz
smartfon się nie uruchamia, to siłą rzeczy nie zostanie w nim odpalona stosowna usługa, która
byłaby w stanie zaakceptować i wgrać na partycje odpowiednie obrazy. Dlatego też w tym przypadku z
tego ROM'u nie będzie za wiele pożytku.

## Jak wgrać ROM fastboot na smartfon Xiaomi

ROM fastboot na smartfony Xiaomi możemy wgrać zarówno z poziomu windows'a, jak i linux'a. Trzeba
tylko posiadać odpowiednią paczkę z ROM'em oraz naturalnie stosowne oprogramowanie zainstalowane w
systemie operacyjnym, z którego korzystamy na co dzień.

### Linux i XiaoMiTool

Technicznie rzecz biorąc, to można bez większego problemu taki ROM fastboot wgrać na smartfon z
poziomu dowolnej dystrybucji linux'a przy pomocy [narzędzia XiaoMiTool][11]. Wystarczy odpalić
XiaoMiTool i wybrać opcję po prawej stronie na poniższej fotce:

![xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux](/img/2021/10/001.xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux.png#huge)

Podczas uruchamiania się urządzenia przełączamy nasz smartfon w tryb fastboot przy pomocy
przycisków Power + VolumeDown. Po podłączeniu telefonu do portu USB komputera, XiaoMiTool powinien
wykryć nasz sprzęt:

![xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux](/img/2021/10/002.xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux.png#huge)

Wskazujemy XiaoMiTool jakie to jest dokładnie urządzenie, w tym przypadku jest to Redmi 9
(lancelot/galahad):

![xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux](/img/2021/10/003.xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux.png#huge)

Po chwili rozpocznie się pobieranie odpowiedniego ROM'u fastboot:

![xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux](/img/2021/10/004.xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-linux.png#huge)

Problem z XiaoMiTool jest jednak taki, że nie możemy określić jaką wersję ROM'u dokładnie chcemy
wgrać i w zasadzie wgrywana jest ta najnowsza dostępna. Być może XiaoMiTool pobiera ROM w oparciu o
Androida, którego mamy wgranego aktualnie na smartfon. Przykładowo, jeśli mamy Androida 10, to
pobierany jest najnowszy dostępny ROM z Androidem 10, a jeśli 11, to z Andkiem 11. Tak czy inaczej,
nie idzie pobrać w ten sposób konkretnej wersji ROM'u Xiaomi, a to nie dobrze. Nie ma też opcji, by
wskazać uprzednio pobraną paczkę `.tgz` .

Drugi problem z XiaoMiTool jest natury technicznej, bo coś to pobieranie ROM'u via XiaoMiTool nie
działa najlepiej. Mniej więcej w połowie pojawia się błąd pobrania i cały proces trzeba rozpocząć
od nowa. Pół biedy, gdyby ponowne pobieranie rozpoczynało się od tego momentu, w którym się urwało
to poprzednie ale pobieranie rozpoczyna się zupełnie od początku i przerywane jest znów w połowie.

Nie byłem w stanie dokończyć tego procesu z poziomu linux'a, a biorąc pod uwagę fakt braku
możliwości określenia jaki ROM dokładnie chciałbym wgrać, to pozostaje opcja z windows'em lub
ręczne wgrywanie obrazów via `fastboot` .

### Ręczne wgrywanie obrazów via fastboot

Nic nie stoi na przeszkodzie, by ręcznie wgrać te wszystkie obrazy z paczki `.tgz` wykorzystując
narzędzie `fastboot` z dowolnego systemu operacyjnego, w tym z linux'a. Praktycznie wszystko co
trzeba zrobić, to postępować według skryptu, który jest załączony w ROM'ie, pilnując tylko, by w
poleceniu `fastboot` wskazać odpowiedni obraz i partycję, a efekt powinien być dokładnie taki sam,
co przy korzystaniu z XiaoMiTool.

Choć lepszym rozwiązaniem będzie odpalić jeden ze skryptów, tj. `flash_all.sh` ,
`flash_all_except_data_storage.sh` albo `flash_all_lock.sh` . Te dwa pierwsze skrypty różnią się od
siebie tym czy dane użytkownika zostaną wyczyszczone. Natomiast ten ostatni skrypt wyczyści dane
użytkownika i dodatkowo zablokuje bootloader po całym procesie flash'owania, co przydaje się gdy
powracamy do fabrycznego oprogramowania producenta telefonu.

Poniżej przykład wywołania skryptu `flash_all_lock.sh` :

    $ ./flash_all_lock.sh
    product: lancelot
    ./flash_all_lock.sh: 6: [: 2x: unexpected operator
    Erasing 'boot'                                     OKAY [  0.006s]
    Finished. Total time: 0.007s
    Sending 'crclist' (0 KB)                           OKAY [  0.011s]
    Writing 'crclist'                                  OKAY [  0.001s]
    Finished. Total time: 0.013s
    Sending 'preloader' (280 KB)                       OKAY [  0.020s]
    Writing 'preloader'                                OKAY [  0.010s]
    Finished. Total time: 0.053s
    Sending 'logo' (1168 KB)                           OKAY [  0.042s]
    Writing 'logo'                                     OKAY [  0.029s]
    Finished. Total time: 0.076s
    Sending 'tee1' (2488 KB)                           OKAY [  0.079s]
    Writing 'tee1'                                     OKAY [  0.036s]
    Finished. Total time: 0.115s
    Sending 'tee2' (2488 KB)                           OKAY [  0.077s]
    Writing 'tee2'                                     OKAY [  0.036s]
    Finished. Total time: 0.114s
    Sending 'scp1' (344 KB)                            OKAY [  0.020s]
    Writing 'scp1'                                     OKAY [  0.009s]
    Finished. Total time: 0.030s
    Sending 'scp2' (344 KB)                            OKAY [  0.020s]
    Writing 'scp2'                                     OKAY [  0.009s]
    Finished. Total time: 0.031s
    Sending 'sspm_1' (494 KB)                          OKAY [  0.025s]
    Writing 'sspm_1'                                   OKAY [  0.010s]
    Finished. Total time: 0.036s
    Sending 'sspm_2' (494 KB)                          OKAY [  0.024s]
    Writing 'sspm_2'                                   OKAY [  0.011s]
    Finished. Total time: 0.036s
    Sending 'lk' (1130 KB)                             OKAY [  0.041s]
    Writing 'lk'                                       OKAY [  0.027s]
    Finished. Total time: 0.069s
    Sending 'lk2' (1130 KB)                            OKAY [  0.041s]
    Writing 'lk2'                                      OKAY [  0.018s]
    Finished. Total time: 0.060s
    Sending sparse 'super' 1/46 (131071 KB)            OKAY [  3.883s]
    Writing 'super'                                    OKAY [  2.770s]
    Sending sparse 'super' 2/46 (130397 KB)            OKAY [  3.778s]
    Writing 'super'                                    OKAY [  2.257s]
    Sending sparse 'super' 3/46 (130574 KB)            OKAY [  3.829s]
    Writing 'super'                                    OKAY [  2.490s]
    Sending sparse 'super' 4/46 (130785 KB)            OKAY [  4.030s]
    Writing 'super'                                    OKAY [  2.259s]
    Sending sparse 'super' 5/46 (131064 KB)            OKAY [  4.136s]
    Writing 'super'                                    OKAY [  1.822s]
    Sending sparse 'super' 6/46 (130645 KB)            OKAY [  3.846s]
    Writing 'super'                                    OKAY [  2.310s]
    Sending sparse 'super' 7/46 (129901 KB)            OKAY [  4.038s]
    Writing 'super'                                    OKAY [  3.337s]
    Sending sparse 'super' 8/46 (130514 KB)            OKAY [  3.996s]
    Writing 'super'                                    OKAY [  2.518s]
    Sending sparse 'super' 9/46 (130511 KB)            OKAY [  4.121s]
    Writing 'super'                                    OKAY [  2.693s]
    Sending sparse 'super' 10/46 (131068 KB)           OKAY [  4.293s]
    Writing 'super'                                    OKAY [  1.836s]
    Sending sparse 'super' 11/46 (126823 KB)           OKAY [  3.999s]
    Writing 'super'                                    OKAY [  5.190s]
    Sending sparse 'super' 12/46 (131068 KB)           OKAY [  4.214s]
    Writing 'super'                                    OKAY [  1.924s]
    Sending sparse 'super' 13/46 (124176 KB)           OKAY [  3.960s]
    Writing 'super'                                    OKAY [  1.819s]
    Sending sparse 'super' 14/46 (131068 KB)           OKAY [  3.745s]
    Writing 'super'                                    OKAY [  1.805s]
    Sending sparse 'super' 15/46 (116312 KB)           OKAY [  3.346s]
    Writing 'super'                                    OKAY [  1.622s]
    Sending sparse 'super' 16/46 (128981 KB)           OKAY [  3.680s]
    Writing 'super'                                    OKAY [  2.026s]
    Sending sparse 'super' 17/46 (117493 KB)           OKAY [  3.258s]
    Writing 'super'                                    OKAY [  1.984s]
    Sending sparse 'super' 18/46 (131069 KB)           OKAY [  3.697s]
    Writing 'super'                                    OKAY [  2.263s]
    Sending sparse 'super' 19/46 (120084 KB)           OKAY [  3.299s]
    Writing 'super'                                    OKAY [  1.775s]
    Sending sparse 'super' 20/46 (121336 KB)           OKAY [  3.333s]
    Writing 'super'                                    OKAY [  1.865s]
    Sending sparse 'super' 21/46 (131068 KB)           OKAY [  3.636s]
    Writing 'super'                                    OKAY [  1.841s]
    Sending sparse 'super' 22/46 (131069 KB)           OKAY [  3.676s]
    Writing 'super'                                    OKAY [  2.099s]
    Sending sparse 'super' 23/46 (120810 KB)           OKAY [  3.551s]
    Writing 'super'                                    OKAY [ 11.678s]
    Sending sparse 'super' 24/46 (126768 KB)           OKAY [  3.542s]
    Writing 'super'                                    OKAY [  1.744s]
    Sending sparse 'super' 25/46 (116196 KB)           OKAY [  3.245s]
    Writing 'super'                                    OKAY [  1.716s]
    Sending sparse 'super' 26/46 (130900 KB)           OKAY [  3.685s]
    Writing 'super'                                    OKAY [  1.885s]
    Sending sparse 'super' 27/46 (131032 KB)           OKAY [  3.677s]
    Writing 'super'                                    OKAY [  1.809s]
    Sending sparse 'super' 28/46 (130984 KB)           OKAY [  3.717s]
    Writing 'super'                                    OKAY [  1.905s]
    Sending sparse 'super' 29/46 (130793 KB)           OKAY [  3.690s]
    Writing 'super'                                    OKAY [  2.093s]
    Sending sparse 'super' 30/46 (129879 KB)           OKAY [  3.785s]
    Writing 'super'                                    OKAY [  3.913s]
    Sending sparse 'super' 31/46 (130996 KB)           OKAY [  3.755s]
    Writing 'super'                                    OKAY [  1.853s]
    Sending sparse 'super' 32/46 (130007 KB)           OKAY [  3.681s]
    Writing 'super'                                    OKAY [  3.912s]
    Sending sparse 'super' 33/46 (130348 KB)           OKAY [  3.700s]
    Writing 'super'                                    OKAY [  1.824s]
    Sending sparse 'super' 34/46 (129941 KB)           OKAY [  3.666s]
    Writing 'super'                                    OKAY [  3.348s]
    Sending sparse 'super' 35/46 (129795 KB)           OKAY [  3.633s]
    Writing 'super'                                    OKAY [  2.745s]
    Sending sparse 'super' 36/46 (131052 KB)           OKAY [  3.629s]
    Writing 'super'                                    OKAY [  1.813s]
    Sending sparse 'super' 37/46 (131036 KB)           OKAY [  3.654s]
    Writing 'super'                                    OKAY [  1.844s]
    Sending sparse 'super' 38/46 (131036 KB)           OKAY [  3.724s]
    Writing 'super'                                    OKAY [  1.812s]
    Sending sparse 'super' 39/46 (115698 KB)           OKAY [  3.323s]
    Writing 'super'                                    OKAY [  3.471s]
    Sending sparse 'super' 40/46 (131068 KB)           OKAY [  3.840s]
    Writing 'super'                                    OKAY [  4.703s]
    Sending sparse 'super' 41/46 (131068 KB)           OKAY [  3.634s]
    Writing 'super'                                    OKAY [  1.835s]
    Sending sparse 'super' 42/46 (117208 KB)           OKAY [  3.240s]
    Writing 'super'                                    OKAY [  1.684s]
    Sending sparse 'super' 43/46 (131068 KB)           OKAY [  3.636s]
    Writing 'super'                                    OKAY [  1.829s]
    Sending sparse 'super' 44/46 (121936 KB)           OKAY [  3.386s]
    Writing 'super'                                    OKAY [  1.741s]
    Sending sparse 'super' 45/46 (127036 KB)           OKAY [  3.516s]
    Writing 'super'                                    OKAY [  1.936s]
    Sending sparse 'super' 46/46 (71052 KB)            OKAY [  1.975s]
    Writing 'super'                                    OKAY [  1.026s]
    Finished. Total time: 288.741s
    Sending 'cache' (68 KB)                            OKAY [  0.013s]
    Writing 'cache'                                    OKAY [  0.042s]
    Finished. Total time: 0.056s
    Sending 'recovery' (65536 KB)                      OKAY [  1.781s]
    Writing 'recovery'                                 OKAY [  1.333s]
    Finished. Total time: 3.116s
    Sending 'boot' (65536 KB)                          OKAY [  1.998s]
    Writing 'boot'                                     OKAY [  1.343s]
    Finished. Total time: 3.366s
    Sending 'dtbo' (129 KB)                            OKAY [  0.015s]
    Writing 'dtbo'                                     OKAY [  0.008s]
    Finished. Total time: 0.023s
    Sending 'vbmeta' (4 KB)                            OKAY [  0.011s]
    Writing 'vbmeta'                                   OKAY [  0.002s]
    Finished. Total time: 0.014s
    Sending 'spmfw' (48 KB)                            OKAY [  0.012s]
    Writing 'spmfw'                                    OKAY [  0.005s]
    Finished. Total time: 0.018s
    Sending 'md1img' (58066 KB)                        OKAY [  1.615s]
    Writing 'md1img'                                   OKAY [  1.195s]
    Finished. Total time: 2.811s
    Sending sparse 'cust' 1/3 (129728 KB)              OKAY [  3.969s]
    Writing 'cust'                                     OKAY [  1.794s]
    Sending sparse 'cust' 2/3 (129160 KB)              OKAY [  3.794s]
    Writing 'cust'                                     OKAY [  1.763s]
    Sending sparse 'cust' 3/3 (52632 KB)               OKAY [  1.600s]
    Writing 'cust'                                     OKAY [  0.731s]
    Finished. Total time: 13.684s
    Sending 'vbmeta_system' (4 KB)                     OKAY [  0.011s]
    Writing 'vbmeta_system'                            OKAY [  0.002s]
    Finished. Total time: 0.027s
    Sending 'vbmeta_vendor' (4 KB)                     OKAY [  0.011s]
    Writing 'vbmeta_vendor'                            OKAY [  0.002s]
    Finished. Total time: 0.014s
    Invalid sparse file format at header magic
    Sending sparse 'exaid' 1/1 (328 KB)                OKAY [  0.022s]
    Writing 'exaid'                                    OKAY [  2.942s]
    Finished. Total time: 3.060s
    Sending sparse 'userdata' 1/13 (129064 KB)         OKAY [  3.920s]
    Writing 'userdata'                                 OKAY [  1.754s]
    Sending sparse 'userdata' 2/13 (129012 KB)         OKAY [  3.806s]
    Writing 'userdata'                                 OKAY [  1.754s]
    Sending sparse 'userdata' 3/13 (129028 KB)         OKAY [  3.826s]
    Writing 'userdata'                                 OKAY [  1.736s]
    Sending sparse 'userdata' 4/13 (129012 KB)         OKAY [  3.900s]
    Writing 'userdata'                                 OKAY [  1.747s]
    Sending sparse 'userdata' 5/13 (129028 KB)         OKAY [  3.832s]
    Writing 'userdata'                                 OKAY [  1.732s]
    Sending sparse 'userdata' 6/13 (129012 KB)         OKAY [  3.823s]
    Writing 'userdata'                                 OKAY [  1.748s]
    Sending sparse 'userdata' 7/13 (129028 KB)         OKAY [  3.834s]
    Writing 'userdata'                                 OKAY [  1.748s]
    Sending sparse 'userdata' 8/13 (129012 KB)         OKAY [  3.932s]
    Writing 'userdata'                                 OKAY [  1.753s]
    Sending sparse 'userdata' 9/13 (129028 KB)         OKAY [  3.803s]
    Writing 'userdata'                                 OKAY [  1.741s]
    Sending sparse 'userdata' 10/13 (129012 KB)        OKAY [  3.648s]
    Writing 'userdata'                                 OKAY [  1.753s]
    Sending sparse 'userdata' 11/13 (129024 KB)        OKAY [  3.699s]
    Writing 'userdata'                                 OKAY [  1.746s]
    Sending sparse 'userdata' 12/13 (129020 KB)        OKAY [  3.927s]
    Writing 'userdata'                                 OKAY [  1.756s]
    Sending sparse 'userdata' 13/13 (73608 KB)         OKAY [  2.178s]
    Writing 'userdata'                                 OKAY [  1.013s]
    Finished. Total time: 70.261s
    FAILED (remote: 'unknown command')
    fastboot: error: Command failed
    (bootloader) Start lock flow

    OKAY [  2.145s]
    Finished. Total time: 2.145s

W tym powyższym logu pojawiło się trochę błędów ale żaden z nich nie jest krytyczny i nie wpływa na
poprawne wgranie ROM'u na telefon.

#### FAILED (remote: 'update sparse crc list failed')

Czasami przy powrocie do fabrycznego oprogramowania producenta telefonu mogą się pojawić błędy
uniemożliwiające wgranie ROM'u fastboot. Poniżej mamy właśnie taki przykład, w którym to z jakiegoś
powodu próba wgrania `sparsecrclist` kończy się błędem:

    $ ./flash_all_lock.sh
    product: lancelot
    ./flash_all_lock.sh: 6: [: 2x: unexpected operator
    Erasing 'boot'                                     OKAY [  0.006s]
    Finished. Total time: 0.007s
    Warning: skip copying crclist image avb footer (crclist partition size: 0, crclist image size: 277).
    Sending 'crclist' (0 KB)                           OKAY [  0.011s]
    Writing 'crclist'                                  OKAY [  0.001s]
    Finished. Total time: 0.014s
    Warning: skip copying sparsecrclist image avb footer (sparsecrclist partition size: 0, sparsecrclist image size: 725).
    Sending 'sparsecrclist' (0 KB)                     OKAY [  0.011s]
    Writing 'sparsecrclist'                            FAILED (remote: 'update sparse crc list failed')
    fastboot: error: Command failed
    Flash sparsecrclist error

Z jakiego powodu aktualizacja tego `sparsecrclist` kończy się niepowodzeniem, to nie mam pojęcia.
Niektórzy [winą obarczają program, który wypakowuje paczkę .tgz z ROM'em fastboot][12]. W tym
przypadku został użyty `patool` :

    $ patool extract lancelot_eea_global_images_V13.0.3.0.SJCEUXM_20230111.0000.00_12.0_eea_04fc56f80c.tgz
    INFO patool: Extracting lancelot_eea_global_images_V13.0.3.0.SJCEUXM_20230111.0000.00_12.0_eea_04fc56f80c.tgz ...
    INFO patool: running /usr/bin/tar --force-local --help
    INFO patool:     with stderr='-3', input=''
    INFO patool: running /usr/bin/tar --extract -z --force-local --file lancelot_eea_global_images_V13.0.3.0.SJCEUXM_20230111.0000.00_12.0_eea_04fc56f80c.tgz --directory ./Unpack_mv6qph5g
    INFO patool:     with input=''
    INFO patool: ... lancelot_eea_global_images_V13.0.3.0.SJCEUXM_20230111.0000.00_12.0_eea_04fc56f80c.tgz extracted to `lancelot_eea_global_images_V13.0.3.0.SJCEUXM_20230111.0000.00_12.0_eea'.

No jak widać, `patool` woła bezpośrednio binarkę `/usr/bin/tar` , a co innego może się bardziej
nadawać do wypakowania paczki `.tgz` niż `tar` , który te paczki tworzy?

Inni użytkownicy z kolei [widzą problem w samym narzędziu fastboot][13], bo najwyraźniej Google ma
swoją implementację tego narzędzia, a Xiaomi swoją. Wgyląda na to, ze na tym `fastboot` od Google
mogą się pojawić tego typu błędy, gdy w grę wchodzi ROM od Xiaomi.

W moim przypadku, na jednym z telefonów podmiana binarki `fastboot` pomogła. I faktycznie problem
się rozwiązał sam z siebie. Niestety w przypadku drugiego telefonu, ani jeden `fastboot` , ani
drugi nie były w stanie tego błędu poprawić.

Czy to znaczy, że możemy zapomnieć o wgraniu takiego ROM'u i zwyczajnie uruchomić telefon ponownie
i jakoś żyć dalej? Prawdę mówiąc to nie miałem odwagi tego sprawdzić, bo z tego powyższego logu
można było wyczytać, że `Erasing 'boot' OKAY` , czyli partycja `/boot/` została wyczyszczona i
lepiej nie uruchamiać teraz tego telefonu ponownie.

#### Pozbycie się plików z sumami kontrolnymi CRC (crclist.txt oraz sparsecrclist.txt)

Analizując skrypt `flash_all_lock.sh` można w nim zauważyć te poniższe wpisy:

    fastboot $* flash crclist `dirname $0`/images/crclist.txt
    if [ $? -ne 0 ] ; then echo "Flash crclist error"; exit 1; fi
    fastboot $* flash sparsecrclist `dirname $0`/images/sparsecrclist.txt
    if [ $? -ne 0 ] ; then echo "Flash sparsecrclist error"; exit 1; fi

W tym przypadku to drugie wywołanie polecenia `fastboot` powoduje błąd ale może się też zdarzyć tak,
że to pierwsze będzie przyczyną problemów. Jeśli nie możemy wgrać ROM'u fastboot na telefon, bo coś
z tymi plikami (albo samymi sumami kontrolnymi CRC) jest nie tak, to te powyższe wpisy możemy
zwyczajnie usunąć ze skryptu i jeszcze raz ten skrypt wywołać.

Czy usunięcie wpisów z `crclist.txt` oraz `sparsecrclist.txt` będzie  miało jakiś wpływ na
działanie telefonu? Tego nie jestem do końca pewny, bo telefon się uruchomił bez problemu na
stock'owym ROM'ie i wszystko zdaje się działać bez najmniejszego problemu.

### Windows i MiFlash

Oczywiście na moim laptopie nie ma windows'a i muszę posiłkować się [maszyną wirtualną
QEMU/KVM][10]. Dobre wieści są jednak takie, że bez problemu można przy pomocy takiej maszyny
wirtualnej przeprowadzić cały proces wgrywania ROM'u fastboot na smartfon, tylko trzeba naturalnie
odpowiednio sobie wcześniej taką maszynę przygotować. Cały proces tworzenia maszyny wirtualnej
został opisany przy okazji [odblokowania bootloader'a w moim Xiaomi Redmi 9][7] i tutaj nie będę
opisywał go ponownie. Jeśli mamy problemy z przygotowaniem odpowiedniej maszyny wirtualnej, to
zapraszam pod ten podlinkowany wyżej artykuł.

Tutaj skupimy się jedynie na samym wgraniu takiego ROM'u na telefon. Zatem przełączamy nasze
urządzenie w tryb fastboot (Power + VolumeDown), podłączamy je do portu USB komputera i uruchamiamy
maszynę wirtualną. W systemie powinno pojawić się nowe urządzenie. W przypadku windows10 nie trzeba
nic dodatkowo instalować w systemie. Pobieramy [MiFlash][8] i uruchamiamy go. Potrzebne nam
naturalnie będą wypakowane pliki z paczki z ROM'em fastboot (możemy je wgrać na maszynę wirtualną
via SMB). Po uruchomieniu MiFlash trzeba wskazać katalog, w którym te pliki się znajdują, po czym
kliknąć przycisk `flash` :

![xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-windows-kvm-qemu](/img/2021/10/005.xiaomitool-fastboot-rom-xiaomi-redmi-9-flash-windows-kvm-qemu.png#huge)

Cały proces flash'owania zajmie dłuższą chwilę. W tym przypadku trwało to prawie 20 minut. Gdy
proces dobiegnie końca, uruchamiamy telefon ponownie.

## Zablokowany bootloader po wgraniu ROM'u fastboot

Po ponownym uruchomieniu telefonu, powinniśmy mieć w nim oryginalne oprogramowanie i w zasadzie
wszystkie problemy, które nas pchnęły do wgrania ROM'u fastboot powinny natychmiast zniknąć.
Niemniej jednak, po wgraniu ROM'u fastboot zostanie w telefonie zablokowany bootloader. Z początku
myślałem, że wystarczy w ustawieniach deweloperskich ściągnąć blokadę OEM i wydać to poniższe
polecenie:

    $ fastboot flashing unlock
    (bootloader) Start unlock flow

    FAILED (remote: 'Token verification failed')
    fastboot: error: Command failed

Ale jak widać, został zwrócony błąd weryfikacji tokenu, tj. tego tokenu, który trzeba uzyskać od
Xiaomi. By odblokować taki telefon, trzeba będzie przejść procedurę odblokowania bootloader'a, tj.
przy pomocy [MiUnlock][9] (windows) lub XiaoMiTool (linux). Nie będzie trzeba za to czekać ponownie
7 dni na pozwolenie ze strony Xiaomi. Zatem odblokowanie bootloader'a będzie natychmiastowe, choć
połączenie z internetem i możliwość nawiązania kontaktu z serwerami Xiaomi są wymagane, podobnie
jak zalogowanie się w aplikacji na czas ściągania blokady z bootloader'a. Warto zatem pamiętać o
tej niedogodności.

## Podsumowanie

Dla osób mojego pokroju, które przed rozpoczęciem prac są w stanie zadbać o zrobienie kopi
zapasowej całego ROM'u w swoim smartfonie, to taki ROM fastboot w zasadzie nie znajduje jakiegoś
większego zastosowania, no może za wyjątkiem cofnięcia wersji fabrycznego softu. W przypadku innych
osób, taki kawałek ROM'u może okazać się wielce użyteczny, choć naturalnie będą takie sytuacje, w
których nic się nie da zrobić, no może poza wizytą w serwisie i wydaniem paru stów na naprawę.
Niemniej jednak, postępując zdroworozsądkowo nie powinniśmy napotkać sytuacji, w których trzeba by
z takiego ROM'u fastboot korzystać.


[1]: https://c.mi.com/oc/miuidownload/detail?guide=2
[2]: https://xiaomifirmwareupdater.com/miui/lancelot/
[3]: https://xiaomifirmwareupdater.com/miui/lancelot/stable/V12.0.1.0.QJCEUXM/
[4]: /post/jak-zaktualizowac-firmware-custom-rom-w-smartfonach-xiaomi/
[5]: /post/jak-wgrac-twrp-recovery-i-magisk-w-xiaomi-redmi-9-galahad-lancelot/#utworzenie-kopi-zapasowej-flasha-telefonu
[6]: https://forum.xda-developers.com/t/guide-how-to-restore-imei-baseband-mac-fix-nvram-warning-and-fix-nvdata-corrupted-on-merlin-redmi-note-9-redmi-10x-4g.4230423/
[7]: /post/jak-odblokowac-bootloader-w-xiaomi-redmi-9-galahad-lancelot/
[8]: https://www.xiaomiflash.com/
[9]: https://en.miui.com/unlock/
[10]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
[11]: https://www.xiaomitool.com/V2/
[12]: https://www.reddit.com/r/Xiaomi/comments/xz9wly/error_on_miflash_tool_check_crc_failed_redmi_note/
[13]: https://xdaforums.com/t/solved-fastboot-flash-super-error-failed-remote-failed-to-check-sparse-crc.4429745/
