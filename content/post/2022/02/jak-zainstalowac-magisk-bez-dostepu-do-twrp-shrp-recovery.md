---
author: Morfik
categories:
- Android
date:    2022-02-28 19:46:00 +0100
lastmod: 2022-02-28 19:46:00 +0100
published: true
status: publish
tags:
- smartfon
- xiaomi
- redmi-9
- root
- aosp
- crdroid
- twrp
- shrp
- magisk
GHissueID: 589
title: Jak zainstalować Magisk bez dostępu do TWRP/SHRP recovery
---

Parę dni temu na mój telefon Xiaomi Redmi 9 (lancelot/galahad/shiva/lava) [została wypuszczona
aktualizacja ROM'u crDroid][1], który bazuje na AOSP/LineageOS. Ten update nie tylko miał
uwzględnione najnowsze poprawki bezpieczeństwa ale również podbijał wersję Androida z 11 na 12.
Problem w tym, że TWRP/SHRP recovery ma problemy z obsługą sposobu szyfrowania partycji `/data/` ,
który najwyraźniej uległ przeobrażeniu w Androidzie 12. Efektem braku wsparcia dla szyfrowania w
TWRP/SHRP jest naturalnie brak możliwości wgrywania danych na partycję `/data/` . Niestety niesie
to za sobą przykre konsekwencje w postaci uniemożliwienia użytkownikowi przeprowadzenia procesu
patch'owania obrazu partycji `/boot/` z poziomu trybu recovery, czego efektem jest brak możliwości
zainstalowania Magisk'a w systemie smartfona. Bez Magisk'a nie damy rady ukorzenić systemu, tj.
uzyskać w nim praw administratora root. Na szczęście nie wszystko stracone. Magisk'a można
zainstalować w telefonie ręcznie przy pomocy ADB oraz trybu bootloader'a (fastboot) eliminując tym
samym potrzebę przełączenia się w tryb recovery. Niniejszy artykuł ma na celu pokazanie jak
zainstalować Magisk'a bez odwoływania się do trybu TWRP/SHRP recovery.

<!--more-->
## Android 12 i problemy z wgraniem Magisk'a via TWRP/SHRP recovery

Widząc, że na stronie crDroid pojawiła się aktualizacja ROM'u do mojego smartfona, postanowiłem jak
najszybciej wgrać tę paczkę `.zip` na telefon, jako że proces aktualizacji ROM'u crDroid nie wiąże
się z utratą danych, tj. nie trzeba formatować partycji `/data/` , tak jak to ma miejsce przy
przesiadce z innego oprogramowania. Naturalnie cały proces wgrywania nowszej wersji ROM'u jest
dokonywany z poziomu trybu recovery (w tym przypadku użyty był SHRP). Po przełączaniu się do trybu
recovery, system prosi o podanie hasła do blokady ekranu i w ten sposób deszyfrowana jest zawartość
partycji `/data/` . By przeprowadzić ten proces aktualizacji, trzeba wcześniej usunąć root'a z
systemu, tj. odinstalować Magisk'a. Naturalnie sam tryb TWRP/SHRP recovery w dalszym ciągu będzie
zapewniał dostęp root sam z siebie. Gdy aktualizacja systemu dobiegła końca, uruchomiłem smartfon
ponownie i wszystko zdawało się działać poprawnie, tyle że nie było dostępu do polecenia `su` , bo
Magisk jeszcze nie został zainstalowany (wymagany reboot po wgraniu aktualizacji ROM'u) .

Następnym krokiem było oczywiście wgranie Magisk'a via SHRP recovery i po przełączeniu się w ten
tryb okazało się, że nie zostałem poproszony o hasło do blokady ekranu. Trochę się zdziwiłem i
postanowiłem sprawdzić czy pliki na partycji `/data/` są widoczne. Poniżej jest zawartość głównego
folderu ten partycji:

    lancelot:/ # ls -al /sdcard/
    total 304
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-19 15:49 +XAQ9G21gmD5+VouAJgIBA
    drwxrws---  29 media_rw media_rw  4096 2022-02-27 18:46 .
    drwxr-xr-x  36 root     root         0 2022-02-27 19:35 ..
    drwxrwxr-x   2 media_rw media_rw  4096 2022-02-27 18:54 2K7k5AIOn53AWH6riH51dD
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-19 15:49 54YjhAl7Giashs9wl7,fpD
    drwxrwxrwx   2 media_rw media_rw  4096 2022-01-19 16:57 6JkPJIxQIHwuaC05GKQT1C
    drwxrwxr-x 426 media_rw media_rw 36864 2022-02-27 14:09 6U0BHhP9,dWkPTVtB+N4,B
    drwxrwxr-x   2 media_rw media_rw  4096 2022-02-27 17:29 7chZACaGNrhLoHSZdTzKBD
    drwxrws--x   5 media_rw media_rw  4096 2022-01-19 15:49 8miO+uSWMA5HbgM9poJU7C
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-19 15:49 9ZjUXw1pLPy+Az2x5h0b0D
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-25 23:45 A3UINmZfEX+6eUuq93M5hD
    drwxrwxr-x   4 media_rw media_rw  4096 2022-01-20 12:55 BLKZ5arZ9c,0LaUtzAw+0B
    drwxrwxr-x   3 media_rw media_rw  4096 2022-01-29 00:06 FP4mLcqvfOKf9jJ1+r9Z3A
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-22 21:45 IS7R6xR,LUKtwaVtgOUspC
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-20 12:10 IV0eJ2SZzJ4u0O4LFxPSGC
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-19 15:49 JvRJPfft5i4+Dxv+5fvsDB
    drwxrwxr-x   4 media_rw media_rw  4096 2022-01-22 14:27 OaFun2x,eZrvBeiZYFGbnA
    drwxrwxr-x   6 media_rw media_rw  4096 2022-02-20 18:20 P0UCF6R6s3tnKQnxnFOfGD
    drwxrwsr-x   4 media_rw media_rw  4096 2022-02-27 15:00 SirzFpKaxl+x7qWe7O,CmA
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-28 17:44 Uj3p3yTaciC1Y9,mYrDVcC
    drwxrwsr-x   3 media_rw media_rw  4096 2022-02-27 15:58 WJKiyYjskrj2ZPWR8pn8jD
    drwxr-xr-x   3 media_rw media_rw  4096 2022-02-27 14:43 dRYSOBXdTBMEvHAfJ4UjFB
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-28 20:42 fQ,FLpUe3EOKGVQijDb7oA
    drwxrwxr-x   6 media_rw media_rw  4096 2022-01-22 14:26 j2FnaUvnVNVrM2rhPvWBbB
    drwxrwxr-x   4 media_rw media_rw 12288 2022-02-27 18:47 naD739jjyB3eOxF2uJwqKC
    drwxrwxr-x   8 media_rw media_rw  4096 2022-02-11 14:17 oYFIbSq+nf1GOTXi2u+wOA
    drwxrwxr-x   3 media_rw media_rw  4096 2022-01-19 15:49 qBE8NHACJh78EtWpCaaAqC
    drwxrwsr-x   2 media_rw media_rw  4096 2022-02-27 15:00 sk,1hh1021HkVaU0tliITB
    drwxrwxr-x   2 media_rw media_rw  4096 2022-01-26 21:01 udzq58x5dypXf7wbjEfjKB
    -rw-rw-r--   1 media_rw media_rw  1278 2021-12-05 10:03 x,NOuPyaN6sSL4A19DTC9f,gkVG
    -rw-rw-r--   1 media_rw media_rw 27804 2022-01-20 11:53 ydCVzQqJAFRK2QJOslz7JFnnfN0VuxAf

Zawartość jest zaszyfrowana, zatem tryb SHRP recovery nie odszyfrował partycji `/data/` .

Zainstalowanie TWRP recovery w tym przypadku również nic nie da, bo SHRP bazuje na TWRP, a sam TWRP
jeszcze nie potrafi odszyfrować partycji data na Androidzie 12 ([wspierane są wersje jedynie do
Androida 11][2]).

Wiedziałem, że wersja Androida ulegnie zmianie ale nie byłem zupełnie świadomy faktu, że TWRP/SHRP
recovery będzie miał jakieś problemy z Androidem 12. Miałem zatem do wyboru w zasadzie dwie opcje.
Pierwsze rozwiązanie zakładało korzystanie ze smartfona bez root i czekanie aż w nieodległej (albo
odległej) przyszłości wsparcie dla Androida 12 w TWRP/SHRP zostanie zaimplementowane. Drugim
wyjściem było sformatowanie partycji `/data/` i wgranie starszej wersji ROM'u crDroid. Przyznam, że
ani jedno, ani drugie rozwiązanie mi niezbyt odpowiadało, dlatego też postanowiłem poszukać
bardziej zadowalającego sposobu na uporanie się z tym zaistniałym problemem.

## ADB z dostępem root

Po wgraniu nowego ROM'u na mojego Xiaomi Redmi 9 (lancelot/galahad/shiva/lava), postanowiłem
naturalnie popatrzeć co ten nowy Android 12 ma do zaoferowania i co się w tym systemie pozmieniało
względem poprzednika. Przeszukując opcje, w końcu dotarłem do ustawień deweloperskich i tam na
liście zauważyłem nową pozycję o nazwie `Rooted debugging` , poniżej fotka:

![crdroid-twrp-shrp-recovery-magisk-install-adb-root](/img/2022/02/001.crdroid-twrp-shrp-recovery-magisk-install-adb-root.png#small)

Skoro ta pozycja miała w nazwie coś związanego z root, to postanowiłem ją włączyć. Przełączyłem się
czym prędzej na terminal w moim Debianie i wydałem polecenie `adb shell` , no ale prompt dalej nie
zawierał `#` , ino zwykły `$` . Pomyślałem, że może niekoniecznie ta opcja działa, tak jak można by
przypuszczać ale kiedyś na necie widziałem, że ludzie wpisywali `adb root` zamiast `adb shell` . W
moim przypadku `adb root` nigdy nie działał ale postanowiłem to polecenie wpisać w terminalu mojego
linux'a i zobaczyć co się stanie. Ku mojemu zdziwieniu nie został zwrócony błąd, a jedynie to, co
widać poniżej:

    $ adb root
    restarting adbd as root

Wygląda na to, że wydanie z komputera polecenia `adb root` wpływa na zachowanie się demona `adbd` w
telefonie. Gdy teraz wpisałem `adb shell` , to okazało się, że prawa root zostały przyznane:

    $ adb shell
    galahad:/ #

Warto tutaj zaznaczyć, że polecenie `su` nie zostało użyte, tak jak to mia miejsce w przypadku
tradycyjnego root'a za sprawą Magisk'a.

### Ograniczenia root pod ADB

O ile dostęp root w ADB został przyznany, to wygląda na to, że w pewne miejsca nie da się zajrzeć,
bo dostaje się błąd `Permission denied` , poniżej przykład:

    galahad:/ # ls -al /storage/emulated/
    ls: /storage/emulated/: Permission denied

Do tych powyższych restrykcji trzeba też doliczyć fakt, że dostęp root jest jedynie dla ADB. Zwykłe
aplikacje w dalszym ciągu korzystać z root'a nie mogą. Zatem jak taki root może nam się przydać?

## Kopia obrazu partycji /boot/

Może i dostępu do pewnych katalogów nie mamy ale do tych miejsc, które nas najbardziej interesują,
wgląd już jak najbardziej jest zapewniony. Mowa tutaj o katalogu `/dev/` , w którym znajdują się
pliki urządzeń, m.in. surowe partycje flash'a smartfona (pod `/dev/block/` ). Jako, że Magisk
potrzebuje obrazu partycji `/boot/` do przepakowania, by móc go wgrać na tę partycję, to
postanowiłem przy pomocy `dd` skopiować sobie zawartość partycji `/boot/` do pliku i zapisać całość
na karcie SD:

    galahad:/ # cd /storage/2209-15F1/

    galahad:/storage/2209-15F1 # ls -al /dev/block/by-name/boot
    lrwxrwxrwx 1 root root 21 2022-02-27 16:23 /dev/block/by-name/boot -> /dev/block/mmcblk0p34

    galahad:/storage/2209-15F1 # dd if=/dev/block/by-name/boot of=./boot-crdroid-to-be-patched-by-magisk.img
    131072+0 records in
    131072+0 records out
    67108864 bytes (64 M) copied, 1.324659 s, 48 M/s

### Wyciąganie obrazu partycji /boot/ z pliku ROM'u

Nawet w przypadku, gdy nie mamy ADB z dostępem root, to i tak możemy obraz partycji `/boot/`
pozyskać. Potrzebna nam do tego jest paczka `.zip` ROM'u, który wgraliśmy sobie już na telefon.
Listując zawartość tej paczki, możemy zauważyć, że posiada ona plik `boot.img` :

    $ patool list crDroidAndroid-12.0-20220226-lava-v8.2.zip
    patool: Listing crDroidAndroid-12.0-20220226-lava-v8.2.zip ...
    patool: running /usr/bin/7z l -- crDroidAndroid-12.0-20220226-lava-v8.2.zip

    7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
    p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz (306A9),ASM,AES-NI)

    Scanning the drive for archives:
    1 file, 971135857 bytes (927 MiB)

    Listing archive: crDroidAndroid-12.0-20220226-lava-v8.2.zip

    --
    Path = crDroidAndroid-12.0-20220226-lava-v8.2.zip
    Type = zip
    Physical Size = 971135857
    Comment = signed by SignApk

       Date      Time    Attr         Size   Compressed  Name
    ------------------- ----- ------------ ------------  ------------------------
    2009-01-01 00:00:00 .....          354          354  META-INF/com/android/metadata
    2009-01-01 00:00:00 .....          252          252  META-INF/com/android/metadata.pb
    2009-01-01 00:00:00 .....    201249843    201249843  product.new.dat.br
    2009-01-01 00:00:00 .....            0            0  product.patch.dat
    2009-01-01 00:00:00 .....    544452175    544452175  system.new.dat.br
    2009-01-01 00:00:00 .....            0            0  system.patch.dat
    2009-01-01 00:00:00 .....    208806388    208806388  vendor.new.dat.br
    2009-01-01 00:00:00 .....            0            0  vendor.patch.dat
    2009-01-01 00:00:00 .....      2383248      1085833  META-INF/com/google/android/update-binary
    2009-01-01 00:00:00 .....         3729         1025  META-INF/com/google/android/updater-script
    2009-01-01 00:00:00 .....     67108864     15487183  boot.img
    2009-01-01 00:00:00 .....     16777216        38507  dtbo.img
    2009-01-01 00:00:00 .....          538          228  dynamic_partitions_op_list
    2009-01-01 00:00:00 .....         1190          568  install/bin/backuptool.functions
    2009-01-01 00:00:00 .....         4308         1508  install/bin/backuptool.sh
    2009-01-01 00:00:00 .....         2703          851  product.transfer.list
    2009-01-01 00:00:00 .....         6991         2008  system.transfer.list
    2009-01-01 00:00:00 .....         8192         2890  vbmeta.img
    2009-01-01 00:00:00 .....         2727          839  vendor.transfer.list
    2009-01-01 00:00:00 .....         1675          943  META-INF/com/android/otacert
    ------------------- ----- ------------ ------------  ------------------------
    2009-01-01 00:00:00         1040810393    971131395  20 files

Jest to dokładnie ten sam obraz, który został wrzucony na partycję `/boot/` w smartfonie podczas
wgrywania ROM'u w trybie TWRP/SHRP recovery. Ten plik `boot.img` trzeba wydobyć z paczki `.zip` i
ręcznie przesłać na telefon w celu wprowadzenia w nim zmian przez aplikację Magisk.

## Podanie Magisk'owi obrazu partycji /boot/

Mając obraz partycji `/boot/` , możemy go teraz podać Magisk'owi. Pytanie jak tego dokonać, skoro
nie możemy zainstalować Magisk'a? Pełnego Magisk'a naturalnie póki co nie zainstalujemy ale możemy
zainstalować jego część, a konkretnie chodzi o część systemową, tj. tę aplikację, którą uruchamiamy
podczas standardowej pracy systemu Androida. Pobieramy zatem [najnowszego Magisk'a][5] i instalujemy
plik `.apk` z poziomu działającego Androida, po czym uruchamiamy aplikację:

![crdroid-twrp-shrp-recovery-magisk-install-app](/img/2022/02/002.crdroid-twrp-shrp-recovery-magisk-install-app.png#small)

Jak widać na powyższej fotce, na pozycji `Installed` mamy `N/A` , co oznacza, że Magisk jako taki
nie jest na tym smartfonie zainstalowany.

Wygląda też na to, że w zależności od konfiguracji systemu, sam proces przepakowania obrazu
partycji `/boot/` może przebiegać nieco inaczej. Tutaj znajduje się [post na XDA][3] oraz [wpis na
stronie Magisk'a][4], które nieco obszerniej opisują wszystkie możliwe przypadki. Dlatego też
dobrze jest się z tymi artykułami zapoznać zanim przystąpimy do dalszych prac, by uniknąć
ewentualnego uszkodzenia smartfona. Poniższy opis zakłada, że w linijce z `Ramdisk` widnieje `Yes` .

Można by w tym miejscu zapytać o to jaki jest pożytek z aplikacji Magisk'a, skoro nie działa ona z
prawami administratora root? Aplikacja Magisk'a może przepakować obraz partycji `/boot/` i nie
potrzebne jej są na tym etapie żadne dodatkowe uprawnienia.

Mając uruchomioną aplikację Magisk, wskazujemy obraz partycji `/boot/` , który wcześniej
pozyskaliśmy . Po chwili dostaniemy gotowy plik, który Magisk by standardowo wrzucił na partycję
`/boot/` podczas instalacji via tryb TWRP/SHRP recovery:

|   |   |
|---|---|
| ![crdroid-twrp-shrp-recovery-magisk-install-app-image-create](/img/2022/02/003.crdroid-twrp-shrp-recovery-magisk-install-app-image-create.png#small) | ![crdroid-twrp-shrp-recovery-magisk-install-app-image-create](/img/2022/02/004.crdroid-twrp-shrp-recovery-magisk-install-app-image-create.png#small) |

Niemniej jednak, by wgrać obraz na surową partycję, potrzebne są już prawa root, których w
aplikacji Magisk'a nie mamy. Dlatego też ten przepakowany obraz partycji `/boot/` trzeba będzie
na tę partycję wgrać ręcznie.

### Czy można wgrać Magisk'a z poziomu trybu recovery mając zaszyfrowaną partycję /data/

Nie jestem pewien czy podczas instalacji Magisk'a via tryb TWRP/SHRP recovery są dokonywane jakieś
zmiany na partycji `/data/` , tj. tej partycji, na której są przechowywane dane użytkownika. Na
pewno Magisk korzysta z katalogu `/data/adb/` , a jego zawartość będzie zaszyfrowana. Gdyby Magisk
podczas instalacji z poziomu trybu recovery jedynie przepakowywał obraz i wgrywał go na partycję
`/boot/` , to jak najbardziej można by do tego celu skorzystać z TWRP/SHRP. Podobnie można postąpić
w przypadku [aktualizacji firmware telefonu][6], gdzie wgrywane są nowe obrazy na poszczególne
partycje, bez dotykania przy tym partycji `/data/` . Niemniej jednak, skrypty instalacyjne Magisk'a
mogą jakieś dane na partycji `/data/` umieszczać i jeśli tak jest w istocie, to lepiej nie
instalować go via tryb recovery.

## Wgrywanie obrazu partycji /boot/ via fastboot

Być może wystarczyłoby przy pomocy `dd` przez ADB wgrać plik `magisk_patched-24100_bv4SY.img` ,
który został zapisany w `/storage/emulated/0/Download/` ale nie zawsze też będziemy mieli ADB z
obsługą root, czy też w ogóle tryb TWRP/SHRP recovery. Dlatego też zdecydowałem się opisać poniżej
bardziej uniwersalne rozwiązanie zakładające pobranie pliku obrazu partycji `/boot/` utworzonego
przez aplikację Magisk'a na komputer i wgranie tego pliku na partycję smartfona przy pomocy
`fastboot` z poziomu trybu bootloader'a.

Po zgraniu pliku `magisk_patched-24100_bv4SY.img` na komputer, przełączamy telefon w tryb
bootloader'a:

    $ adb pull /storage/emulated/0/Download/magisk_patched-24100_bv4SY.img
    /storage/emulated/0/Download/magisk_patched-24100_bv4SY.img: 1 file pulled, 0 skipped. 14.9 MB/s (67108864 bytes in 4.295s)

    $ adb reboot bootloader

Następnie wgrywamy przepakowany obraz na partycję `/boot/` posługując się narzędziem `fastboot` :

    $ fastboot flash boot magisk_patched-24100_bv4SY.img
    Sending 'boot' (65536 KB)                          OKAY [  1.758s]
    Writing 'boot'                                     OKAY [  0.868s]
    Finished. Total time: 2.627s

Po wgraniu obrazu, restartujemy smartfon:

    $ fastboot reboot
    Rebooting                                          OKAY [  0.000s]
    Finished. Total time: 0.051s

## Test root w Magisk

Po uruchomieniu systemu, wszystkie aplikacje, które korzystają z root'a, powinny poprosić o
zezwolenie im na dostęp do praw administratora systemu:

|   |   |
|---|---|
| ![crdroid-twrp-shrp-recovery-magisk-installed](/img/2022/02/005.crdroid-twrp-shrp-recovery-magisk-installed.png#small) | ![crdroid-twrp-shrp-recovery-magisk-installed-root-prompt](/img/2022/02/006.crdroid-twrp-shrp-recovery-magisk-installed-root-prompt.png#small) |

Jak widzimy też na fotce powyżej, pozycja `Installed` już nie wskazuje `N/A` , a widnieje tam
numerek wersji Magisk'a, tj. `24.1 (24100)` , co oznacza, że Magisk został zainstalowany z
powodzeniem i to przy całkowitym pominięciu trybu recovery.

## Podsumowanie

Jak widać na przykładzie niniejszego artykułu, tryb TWRP/SHRP recovery nie jest niezbędny, by
Magisk'a w smartfonie z Androidem można było zainstalować. Trzeba jednak pamiętać, że może i prawa
root udało nam się uzyskać, to tryb recovery w dalszym ciągu pozostaje w dużej mierze bezużyteczny,
tj. dalej pliki na partycji `/data/` będą zaszyfrowane. Jedyne co będziemy w stanie zrobić w trybie
recovery w stosunku do partycji `/data/` , to ją sformatować. Trzeba zatem zdawać sobie sprawę z
faktu, że jeśli z jakiegoś powodu nie będziemy w stanie uruchomić systemu w telefonie, to przez ten
tryb TWRP/SHRP recovery nie uda nam się smartfona odratować. Do momentu zaimplementowania wsparcia
dla Android 12 w TWRP/SHRP recovery, każdy problem, który można by było poprawić (albo zaradzić mu)
przez tryb recovery, będzie w zasadzie wiązał się z przywróceniem systemu do ustawień fabrycznych.


[1]: https://crdroid.net/lava/8
[2]: https://twrp.me/site/update/2021/11/28/twrp-3.6.0-released.html
[3]: https://www.xda-developers.com/how-to-install-magisk/
[4]: https://topjohnwu.github.io/Magisk/install.html
[5]: https://github.com/topjohnwu/Magisk/releases
[6]: /post/jak-zaktualizowac-firmware-custom-rom-w-smartfonach-xiaomi/
