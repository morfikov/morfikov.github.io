---
author: Morfik
categories:
- Android
date: "2017-03-09T14:24:44Z"
date_gmt: 2017-03-09 13:24:44 +0100
published: true
status: publish
tags:
- smartfon
- twrp
title: 'Android: Jak odratować smartfon po usunięciu partycji /system/'
---

Zawsze mnie zastanawiało jak to jest możliwe, by przez przypadek można było usunąć dane na jeden z
kluczowych partycji w smartfonie jaką jest partycja `/system/` . Ostatnio wiele osób do mnie pisało
z tego typu problemami i zapytaniem "jak odratować w takiej sytuacji telefon". Odpowiedź wydawała mi
się prosta: wystarczy wgrać uprzednio zrobiony backup wyczyszczonej partycji via `fastboot` .
Problem w tym, że po usunięciu danych z partycji `/system/` , `fastboot` nie działa. A skąd to wiem?
Ano "przez przypadek" usunąłem sobie dane na tej partycji. W sumie to tylko testowałem [ficzer w
TWRP zwany ADB Sideload](https://twrp.me/faq/ADBSideload.html), który niby ma za zadanie wgrać ROM z
paczki `.zip` . Coś poszło nie tak i w zasadzie zostałem z pustą partycją `/system/` . Przy
odpalaniu telefonu w takim stanie, ten w zasadzie jedynie się resetuje co kilka chwil. Może i
`fastboot` nie działa ale można wbić do trybu recovery. Jeśli tylko mamy wgrany TWRP, to jest spora
szansa na odratowanie smartfona. W tym artykule w rolach głównych wystąpi [Neffos
Y5L](http://www.neffos.pl/product/details/Y5L), który ma SoC od Qualcomm'a, zatem nie damy rady się
pobawić SP Flash Tool i całą robotę trzeba będzie odwalić ręcznie.

<!--more-->
## Wyczyszczona partycja /system/ i tryb recovery

TWRP recovery ma opcję przywracania uprzednio zrobionego backup'u dowolnej przestrzeni flash, którą
mamy określoną w pliku `fstab` . Niemniej jednak, by być w stanie skorzystać z tego wbudowanego w
TWRP mechanizmu, backup musi być przeprowadzony również za jego pomocą. Ja mam sporządzoną kopię
binarną całego flash'a mojego Neffos'a Y5, z której mogę sobie wydobyć partycję `/system/`. Ten
obraz jednak trzeba wgrać via `fastboot` , a ten nie działa. Dobrze, że TWRP jest nieco bardziej
zaawansowanym trybem recovery niż te stock'owe odpowiedniki. Tutaj możemy porozumiewać się ze
smartfonem za pośrednictwem `adb` , który oferuje nam dostęp root. Wystarczy na naszym linux'ie
odpalić pierwszy z brzega terminal i wpisać w nim:

    # adb shell

Jeśli ktoś nie wie [jak zainstalować narzędzie adb w
Debianie]({{< baseurl >}}/post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/) i innych
podobnych dystrybucjach linux'a, to tutaj znajdują się stosowne informacje.

W tej chwili możemy operować na smartfonie wydając mu polecenia bezpośrednio z komputera. Jest to o
wiele wygodniejsze rozwiązanie niż działanie na tym wbudowanym w TWRP terminalu.

## Przesyłanie plików na smartfon w trybie recovery

Backup dysku/flash'a zwykle przechowywany jest na innym urządzeniu z wiadomych raczej względów. Jest
niemal pewne, że nie mamy pliku backup'u bezpośrednio w telefonie i trzeba go pierw tam przesłać.
Możemy to zrobić za pomocą `adb` lub też via [protokół
MTP]({{< baseurl >}}/post/smartfon-android-linux-mtp-ptp/). Ten ostatni raczej nie powinien
sprawić problemów. Niżej zaś jest przykładowe polecenie `adb` :

    # adb push /neffos/y5l-orig-system.img /data/media/0/Download/
    3946 KB/s (1959579648 bytes in 484.940s)

Plik z obrazem partycji `/system/` zostanie wgrany w tym przypadku na flash smartfona, z tym, że na
partycję `/data/` . Nic nie stoi na przeszkodzie, by wgrać ten plik na kartę SD, bo niekiedy na
flash'u urządzenia może nam zwyczajnie zabraknąć miejsca.

## Ustalanie urządzenia pod partycję /system/

Musimy teraz ustalić, które urządzenie w katalogu `/dev/` odpowiada partycji `/system/` . Można to
oczywiście odczytać z pliku `fstab` ale też można podejrzeć dowiązania symboliczne w katalogu
`/dev/block/bootdevice/by-name/` , przykładowo:

    # ls -al /dev/block/bootdevice/by-name/ | grep system
    lrwxrwxrwx    1 root     root            21 Jan  7  1970 system -> /dev/block/mmcblk0p21

Wiemy zatem, że `/dev/block/mmcblk0p21` to nasze urządzenie, na którym będziemy operować.
Pamiętajmy, że ten numerek ma kluczowe znaczenie i pomylenie się przy nim może uwalić nasz telefon
całkowicie. Dlatego upewnijmy się parę razy, że wpisujemy poprawne polecenia.

## Wgrywanie obrazu partycji /system/ via dd

Mamy obraz partycji i odpowiadające mu urządzenie. Teraz wystarczy przy pomocy `dd` wgrać ten obraz
binarnie w stosowne miejsce. Zanim jednak wgramy sam obraz, odmontujmy partycję `/system/` . Można
to zrobić w menu Zamontuj w
TWRP:

[![]({{< baseurl >}}/img/2017/03/001.twrp-recovery-system-660x564.png)]({{< baseurl >}}/img/2017/03/001.twrp-recovery-system.png)

Lub też przy pomocy poniższego polecenia przez `adb` :

    # umount /system

Niezależnie od sposobu odmontowania, polecenie `mount` nie powinno zawierać wpisu z partycją
`/system/` :

    # mount | grep system

Teraz możemy przejść do wgrywania obrazu via `dd` . Robimy to w poniższy sposób:

    # dd if=/data/media/0/Download/y5l-orig-system.img of=/dev/block/mmcblk0p21
    3827304+0 records in
    3827304+0 records out
    1959579648 bytes (1.8GB) copied, 361.619535 seconds, 5.2MB/s

I to w zasadzie cała robota. Można w tej chwili uruchomić smartfon ponownie, a ten już powinien nam
standardowo załadować Androida. Nie powinniśmy też na nowo konfigurować urządzenia, bo jeśli
skasowaliśmy sobie tylko dane na partycji `/system/` , to wszelkie zmiany wprowadzane w konfiguracji
urządzenia zostają nietknięte.
