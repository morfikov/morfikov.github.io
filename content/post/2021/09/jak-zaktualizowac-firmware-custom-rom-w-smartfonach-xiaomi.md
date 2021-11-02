---
author: Morfik
categories:
- Android
- Linux
date:    2021-09-30 02:43:00 +0200
lastmod: 2021-09-30 02:43:00 +0200
published: true
status: publish
tags:
- debian
- smartfon
- xiaomi
- redmi-9
- root
- aosp
- firmware
- twrp
- shrp
GHissueID: 322
title: Jak zaktualizować firmware custom ROM'ów w smartfonach Xiaomi
---

Te bardziej szanujące się marki produkujące smartfony zwykle zapewniają wsparcie dla swoich
urządzeń przez co najmniej dwa lata (a czasem nawet i dłużej) od momentu ich wypuszczenia na rynek.
Po wgraniu sobie alternatywnego ROM'u na nasz telefon, oprogramowanie w nim może być aktualizowane
przez opiekuna czy dewelopera takiego ROM'u znacznie dłużej niż producent przewidział. W ten sposób
nie musimy wydawać pieniążków na nowy sprzęt, oczywiście zakładając, że mu nic nie dolega, np. pod
względem wydajności, czy też ewentualnie nie zużył on się nam jakoś bardziej podczas eksploatacji.
Jedną rzeczą, o której posiadacze smartfonów z Androidem zapominają po wgraniu custom ROM'ów na
bazie AOSP/LineageOS, to fakt, że o ile ROM faktycznie dostaje aktualizacje czy to bezpieczeństwa,
czy też upgrade do nowszej wersji Androida, o tyle sam firmware zwykle pozostaje nietknięty. W
przypadku mojego modelu smartfona Redmi 9, Xiaomi od czasu do czasu wypuszcza aktualizacje firmware
do tego urządzenia i przydałoby się ten firmware co jakiś czas zaktualizować. Na szczęście nie
trzeba w tym celu powracać do stock'owego oprogramowania, a cały proces możemy przeprowadzić z
poziomu dowolnej dystrybucji linux'a.

<!--more-->
## Skąd pobrać firmware do Xiaomi Redmi 9

Najprostszym sposobem na pozyskanie firmware do smartfona Xiaomi Redmi 9, jest odwiedzenie [strony
xiaomifirmwareupdater.com][1] i pobranie stosownej paczki `.zip` , przykładowo
`stable V12.5.1.0.RJCEUXM 11.0 Europe` (2021-09-13). Jest to firmware dla Android 11 mającego MIUI
12.5.1.0. Jeśli szukamy wcześniejszych wersji, np. nasz telefon działa pod Androidem 10, to wtedy
musimy [zajrzeć do archiwum][2] i wybrać sobie tę wersję, która nas najbardziej interesuje.

Użytkowników europejskich smartfonów Xiaomi interesują w zasadzie firmware dla regionu `Europe` ale
też można korzystać z tych mających w nazwie `Global` . Nie wiem czy sam firmware się jakoś różni
między tymi dwoma regionami ale jeśli chodzi o sam ROM, to ten `Global` ma nieco więcej funkcji,
które w wariancie europejskim zostały obcięte ze względu na restrykcje pod kątem prywatności, które
takie oprogramowanie musi w Europie spełnić. Dlatego też w przypadku firmware lepiej jest pozostać
również przy wariancie europejskim. Sam smartfon zdaje się działać zarówno na jednym jak i drugim
firmware bez zarzutu.

Czasami jednak ta powyższa strona może nie być dostępna lub też nie uśmiecha nam się pobierać tak
kluczowego w kwestii bezpieczeństwa kawałka oprogramowania, jakim jest firmware, z nieoficjalnych
źródeł. W takim przypadku warto wiedzieć, że paczkę z firmware możemy sobie zrobić sami, a wszystko
czego nam do szczęścia potrzeba, to oficjalny ROM Xiaomi, który można albo pobrać z [oficjalnej
strony Xiaomi][3] lub też z [xiaomifirmwareupdater.com][4], w tym drugim przypadku również jest
[archiwum wcześniejszych wersji][5].

## Jak zrobić paczkę z firmware do wgrania via TWRP/SHRP

Oficjalne obrazy ROM'ów Xiaomi posiadają wszystkie niezbędne rzeczy, które podczas aktualizacji
telefonu są wgrywane na odpowiednie partycje, w tym też te partycje, które wchodzą w skład
firmware. Partycje, które w aktualizacji firmware biorą udział to: `spmfw` , `scp1` (oraz `scp2` ),
`sspm_1` (oraz `sspm_2` ) `tee1` (oraz `tee2` ) `lk` (oraz `lk2` ) i `md1img` . Te dodatkowe
partycje (mające zwykle `2` w nazwie) są zapasowe ale flash'owane są jednym i tym samym obrazem, tj.
tym obrazem, który wgrywany jest na te główne ich odpowiedniki.

Dodatkowo aktualizacja firmware zwykle niesie ze sobą aktualizację preloader'a ( `mmcblk0boot0` ),
przynajmniej można tyle wywnioskować po obecności obrazów mających w nazwie `preloader*.img` w
paczce z firmware. Zatem potencjalnie można taką aktualizacją zabić telefon na śmierć, choć ta
operacja jest jakoś szczególnie bardziej niebezpieczna w stosunku do aktualizacji całego fabrycznego
ROM'u, tylko lepiej nie przerywać tego procesu i nie wyłączać telefonu do momentu aż ten proces
dobiegnie końca sam z siebie. Podobnie sprawa może wyglądać w przypadku wgrania firmware nie od
tego urządzenia co potrzeba. Lepiej sprawdzić kilkakrotnie, czy ten firmware, który zamierzamy
wrzucić na telefon, jest przeznaczony dokładnie na to urządzenie. Dlatego najlepiej jest wydobyć
firmware z oficjalnego ROM'u zamiast pobierać go z innych źródeł, bo wtedy mamy pewność, że dany
firmware będzie pasował do naszego telefonu.

Gdy już pozyskaliśmy obraz z oficjalnym ROM'em do naszego Xiaomi Redmi 9 (lancelot/galahad), to
musimy jeszcze pozyskać stosowne narzędzia, by odpowiednie pliki z tego obrazu wydobyć i zrobić z
nich paczkę z firmware do wgrania via TWRP/SHRP recovery. Póki co w Debianie nie ma takich narzędzi
i trzeba będzie je dociągnąć przy pomocy `pip3` (instalator dla python'a) -- w Debianie jest
dostępny w paczce `python3-pip` . Przy pomocy `pip3` trzeba dociągnąć
`xiaomi_flashable_firmware_creator_gui` :

    $ pip3 install xiaomi_flashable_firmware_creator_gui

Wszystkie potrzebne pliki zostaną wgrane do `~/.local/bin/` :

    $ ls -al
    total 24
    drwxr-xr-x 2 morfik morfik 4096 2021-09-10 08:58:43 ./
    drwxr-xr-x 5 morfik morfik 4096 2021-09-10 08:58:42 ../
    -rwxr-xr-x 1 morfik morfik 1517 2021-09-10 08:58:42 remotezip*
    -rwxr-xr-x 1 morfik morfik  209 2021-09-10 08:58:42 tabulate*
    -rwxr-xr-x 1 morfik morfik  266 2021-09-10 08:58:42 xiaomi_flashable_firmware_creator*
    -rwxr-xr-x 1 morfik morfik  241 2021-09-10 08:58:43 xiaomi_flashable_firmware_creator_g*

Teraz już wystarczy `odpalić xiaomi_flashable_firmware_creator_g` i powinniśmy zobaczyć to poniższe
okienko:

![xiaomi-redmi-9-firmware-create](/img/2021/09/001.xiaomi-redmi-9-firmware-create.png#huge)

Wskazujemy naturalnie plik `.zip` ze stock'owym ROM'em od Xiaomi, który wcześniej pobraliśmy.
Zaznaczamy też opcję `Firmware` i wciskamy `Create` :

![xiaomi-redmi-9-firmware-create](/img/2021/09/002.xiaomi-redmi-9-firmware-create.png#huge)

Po chwili paczka z firmware powinna zostać utworzona:

![xiaomi-redmi-9-firmware-create](/img/2021/09/003.xiaomi-redmi-9-firmware-create.png#huge)

Możemy jeszcze dla pewności podejrzeć jej zawartość, np. przy pomocy `patool` :

    $ patool list  'fw_lancelot_miui_LANCELOTEEAGlobal_V12.0.1.0.QJCEUXM_77fd23f7f7_10.0.zip'
    patool: Listing fw_lancelot_miui_LANCELOTEEAGlobal_V12.0.1.0.QJCEUXM_77fd23f7f7_10.0.zip ...
    patool: running /usr/bin/7z l -- fw_lancelot_miui_LANCELOTEEAGlobal_V12.0.1.0.QJCEUXM_77fd23f7f7_10.0.zip

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz (306A9),ASM,AES-NI)

    Scanning the drive for archives:
    1 file, 40185787 bytes (39 MiB)

    Listing archive: fw_lancelot_miui_LANCELOTEEAGlobal_V12.0.1.0.QJCEUXM_77fd23f7f7_10.0.zip

    --
    Path = fw_lancelot_miui_LANCELOTEEAGlobal_V12.0.1.0.QJCEUXM_77fd23f7f7_10.0.zip
    Type = zip
    Physical Size = 40185787

       Date      Time    Attr         Size   Compressed  Name
    ------------------- ----- ------------ ------------  ------------------------
    2021-09-29 21:08:20 D....            0            0  META-INF
    2021-09-29 21:08:20 .....     59066208     35685406  md1img.img
    2021-09-29 21:08:20 .....            1            3  type.txt
    2021-09-29 21:08:20 .....          859          364  scatter.txt
    2021-09-29 21:08:20 .....       278224       169453  preloader.img
    2021-09-29 21:08:20 .....       504592       482338  sspm.img
    2021-09-29 21:08:20 .....        37984         7454  spmfw.img
    2021-09-29 21:08:20 .....       278224       169453  preloader_emmc.img
    2021-09-29 21:08:18 .....      1646848       527385  lk.img
    2021-09-29 21:08:20 .....      2363616      2011977  tee.img
    2021-09-29 21:08:20 .....       352368       144725  scp.img
    2021-09-29 21:08:18 .....         5655         5440  compatibility.zip
    2021-09-29 21:08:18 D....            0            0  META-INF/com
    2021-09-29 21:08:20 .....         2275         1052  META-INF/MANIFEST.MF
    2021-09-29 21:08:20 .....         2328         1127  META-INF/CERT.SF
    2021-09-29 21:08:20 .....         1634         1143  META-INF/CERT.RSA
    2021-09-29 21:08:20 D....            0            0  META-INF/com/android
    2021-09-29 21:08:18 D....            0            0  META-INF/com/google
    2021-09-29 21:08:18 .....          316          219  META-INF/com/android/metadata
    2021-09-29 21:08:20 .....         1594         1077  META-INF/com/android/otacert
    2021-09-29 21:08:18 D....            0            0  META-INF/com/google/android
    2021-09-29 21:08:18 .....      2143032       973739  META-INF/com/google/android/update-binary
    2021-09-29 21:08:20 .....         3541          866  META-INF/com/google/android/updater-script
    ------------------- ----- ------------ ------------  ------------------------
    2021-09-29 21:08:20           66689299     40183221  18 files, 5 folders

## Wgrywanie firmware via TWRP/SHRP

Tak utworzoną paczkę z firmware można już wgrać na naszego Xiaomi Redmi 9 via TWRP lub SHRP
recovery. Przełączamy zatem smartfon w tryb recovery. Z menu wybieramy `Flash` i wskazujemy ścieżkę
do pliku z firmware:

|   |   |   |
|---|---|---|
| ![xiaomi-redmi-9-firmware-shrp-update](/img/2021/09/004.xiaomi-redmi-9-firmware-shrp-update.png#small) | ![xiaomi-redmi-9-firmware-shrp-update](/img/2021/09/005.xiaomi-redmi-9-firmware-shrp-update.png#small) | ![xiaomi-redmi-9-firmware-shrp-update](/img/2021/09/006.xiaomi-redmi-9-firmware-shrp-update.png#small) |

## Podsumowanie

No jak widać, proces tworzenia paczki z firmware nie jest jakoś specjalnie trudny, przynajmniej w
przypadku modelu smartfona Redmi 9, czy ogólnie urządzeń marki Xiaomi. Wszystko czego nam potrzeba
to obraz oficjalnego ROM'u, kawałek maszyny z linux'em i prosty programik python'owy. W opisany
wyżej sposób możemy dokonać aktualizacji firmware praktycznie w dowolnym momencie, o ile oczywiście
posiadamy odblokowany bootloader. Jak tylko wyjdzie nowszy oficjalny ROM na nasze urządzenie, to
jest to dla nas znak, że firmware wymaga aktualizacji. Trzeba jednak pamiętać, że custom ROM'y nie
zawsze są przystosowane do obsługi nowszych firmware, choć zdarza się to rzadko i stosowna
informacja na stronie takiego ROM'u powinna być zamieszczona. Aktualizując regularnie firmware
smartfona załatamy też wszystkie wykryte w nim podatności i błędy (przynajmniej w teorii).


[1]: https://xiaomifirmwareupdater.com/firmware/lancelot/
[2]: https://xiaomifirmwareupdater.com/archive/firmware/lancelot/
[3]: https://c.mi.com/oc/miuidownload/detail?device=1900381
[4]: https://xiaomifirmwareupdater.com/miui/lancelot/
[5]: https://xiaomifirmwareupdater.com/archive/miui/lancelot/
