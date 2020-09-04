---
author: Morfik
categories:
- OpenWRT
date: "2016-04-27T00:32:19Z"
date_gmt: 2016-04-26 22:32:19 +0200
published: true
status: publish
tags:
- pendrive
- hdd
- ssd
- chaos-calmer
- router
title: Dysk, pendrive i inne nośniki pod OpenWRT
---

Czym by był router bez portów USB? Obecnie chyba wszystkie routery posiadają przy najmniej jeden
taki port. Umożliwia to podłączenie pendrive, dysku USB, drukarki i innych urządzeń posiadających
interfejs USB. W przypadku, gdy posiadamy jeden port USB i chcemy podłączyć dwa (lub więcej)
urządzenia, musimy skorzystać z hubów USB. Ja w przypadku swojego routera [TP-LINK TL-WR1043N/ND
v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) mam zastosowane właśnie takie
rozwiązanie. Niby potrzebuję trzy porty USB, a ten router ma tylko jeden. Co prawda, pojawia się
problem z zapotrzebowaniem na energię ale aktywne huby USB wyposażone w zasilacze niwelują tę
dolegliwość. Ten wpis ma na celu przedstawić zarządzanie tymi wszystkimi nośnikami pod OpenWRT.
Zostanie tutaj pokazane jak stworzyć i zamontować partycje o systemie plików EXT4, NTFS, FAT.

<!--more-->
## Problemy związane z nieodpowiednim wyłączaniem routera

W przypadku, gdy w grę wchodzą systemy plików, trzeba mieć jedną rzecz na uwadze. Jeśli zamontujemy
na routerze jakąś partycję, to nie możemy wyłączać routera za pomocą przycisku na jego obudowie.
Podobnie sprawa ma się z zanikami napięcia, czy resetowaniem routera przez wyciągnięcie zasilacza z
kontaktu. Tego typu akcje mogą uszkodzić dane na tym zewnętrznym nośniku USB. W ten sposób prawie na
pewno stracimy część plików, zwłaszcza przypadku, gdy procesy na routerze będą dokonywać zapisu
danych. Router niczym się praktycznie nie różni w działaniu od zwykłego PC, jest tylko nieco
mniejszy i nie ma tych wszystkich podzespołów, które mają standardowe komputery. Dlatego też trzeba
postępować z nim podobnie jak z komputerami. A tych przecie nie wyłączamy via przycisk na zasilaczu.
Jeśli jednak zapomnimy zdemontować system plików, w skrajnych przypadkach może to nawet doprowadzić
do uszkodzenia całej partycji. Wtedy trzeba będzie ponownie zakładać na niej system plików, co
wyczyści wszelkie dane zgromadzone na tej partycji.

Nie ma się co jednak zniechęcać. Routery nie pobierają znowu aż tak dużo prądu. Jest to wielkość
rzędu 3-4W i nawet jeśli chodzą nonstop, to nie podniosą znacząco rachunku za prąd. W przypadku,
gdy chcemy wyłączyć router, możemy to zrobić logując się przez `ssh` na router, po czym wpisać w
terminalu to poniższe polecenie:

    # sync && poweroff

To polecenie (a w zasadzie dwa) zsynchronizuje bufory nośników wymuszając zapisanie wszelkich danych
przebywających w pamięci RAM na dysk oraz wyłączy system. W ten sposób stracimy połączenie z
routerem ale nie wyłączy mu to zasilania. W efekcie, diody dalej będą się świecić czy migać.
Niemniej jednak, w takim stanie, router może zostać bezpiecznie odcięty od zasilania w dowolny
sposób bez ryzyka utraty danych zgromadzonych na dyskach.

## Tworzenie partycji i systemu plików

Partycje na dysku czy pendrive można tworzyć na kilka sposobów. Można to zrobić pod windowsem, pod
linux'em lub pod OpenWRT. Na windowsie jesteśmy nieco ograniczeni co do wyboru systemu plików dla
partycji. Zwykle tam mamy NTFS lub FAT. Odpada nam też możliwość podziału pendrive na partycje. Na
linux'ach, w tym OpenWRT, możemy tworzyć partycje o dowolnym systemie plików i kroić nośniki jak nam
się podoba. Tutaj nie będziemy korzystać z zewnętrznych systemów/narzędzi i ograniczymy się tylko do
tego co zapewnia nam firmware OpenWRT.

Przede wszystkim, musimy się zdecydować na rodzaj systemu plików, który chcemy wykorzystać na
pendrive czy dysku. [OpenWRT wspiera całą masę systemu
plików](https://wiki.openwrt.org/doc/howto/storage). Dokładną listę modułów kernela
odpowiedzialnych za obsługę systemu plików można uzyskać wpisując w terminalu polecenie `opkg list |
grep kmod-fs` . Każdy typ systemu plików wymaga swojego modułu i trzeba go zainstalować. Jako, że
OpenWRT to linux, to najbardziej wpieranym systemem plików jest `ext2` , `ext3` i `ext4` . Obecnie
się używa tego ostatniego. By zaimplementować obsługę systemu plików z rodziny EXT na routerze
trzeba zainstalować pakiet `kmod-fs-ext4` . Jako, że dostęp do urządzeń będzie odbywał się po
interfejsie USB, to musimy także zainstalować pakiet `kmod-usb-storage` . W zależności od
specyfikacji portu USB musimy także dociągnąć moduł `kmod-usb-ohci` (USB 1.1) , `kmod-usb-uhci` (USB
1.1) lub `kmod-usb2` (USB 2) oraz `kmod-usb3` (USB3). Narzędzia do obsługi systemu plików EXT są
zawarte w pakiecie `e2fsprogs` . To przy ich pomocy możemy utworzyć system plików, czy sprawdzić go
pod kątem ewentualnych błędów. By być w stanie w ogóle partycjonować dyski pod OpenWRT, musimy także
doinstalować pakiet `fdisk` .

    # opkg update
    # opkg install kmod-fs-ext4 kmod-usb-storage kmod-usb-ohci kmod-usb-uhci kmod-usb2 e2fsprogs fdisk

Proces partycjonowania dysku czy pendrive zaczyna się od określenia przestrzeni pod konkretną
partycję. Podłączamy zatem nasze urządzenie do portu USB. Powinno ono zostać wykryte i stosownie
oznaczone. Możemy się o tym przekonać zaglądając w log przy pomocy `logread` :

    kernel: usb 1-1.1: new high-speed USB device number 10 using ehci-platform
    kernel: usb-storage 1-1.1:1.0: USB Mass Storage device detected
    kernel: scsi host7: usb-storage 1-1.1:1.0
    kernel: scsi 7:0:0:0: Direct-Access     Kingston DT 101 G2        PMAP PQ: 0 ANSI: 0 CCS
    kernel: sd 7:0:0:0: [sdb] 30299520 512-byte logical blocks: (15.5 GB/14.4 GiB)
    kernel: sd 7:0:0:0: [sdb] Write Protect is off
    kernel: sd 7:0:0:0: [sdb] Mode Sense: 23 00 00 00
    kernel: sd 7:0:0:0: [sdb] No Caching mode page found
    kernel: sd 7:0:0:0: [sdb] Assuming drive cache: write through
    kernel:   sdb: sdb1
    kernel: sd 7:0:0:0: [sdb] Attached SCSI removable disk

W tym przypadku, pendrive uzyskał oznaczenie `sdb` . By rozpocząć proces formatowania tego
urządzenia, w terminalu wydajemy to poniższe polecenie:

    # fdisk /dev/sdb

Wskazówki dotyczące formatowania możemy wyświetlić przez wpisanie `m` . Wygląda to mniej więcej tak
jak na tej fotce poniżej:

![]({{< baseurl >}}/img/2016/04/1.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#big)

W przypadku, gdy mamy do czynienia z nowym dyskiem, który nie posiada jeszcze taliby partycji, to
trzeba mu ją stworzyć. Pendrive zwykle mają tablicę MS-DOS. W przypadku tradycyjnych dysków,
zwłaszcza tych większych, dobrze jest utworzyć tablicę GPT. W zależności od preferencji, tablicę
tworzymy wpisując `o` (MS-DOS) lub `g` (GPT). W tym przypadku skorzystamy z tablicy MS-DOS:

![]({{< baseurl >}}/img/2016/04/2.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#big)

Teraz sprawdzamy parametry dysku, w tym też ile miejsca mamy do dyspozycji. Robimy to za pomocą
`p` :

![]({{< baseurl >}}/img/2016/04/3.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#big)

Mamy nieco ponad 14 GiB. Podzielmy zatem ten pendrive na 4 partycje. Tworzenie partycji aktywujemy
przez `n` . Następnie określamy typ partycji (podstawowa `p`, rozszerzona `e` ) oraz jej numer.
Jeśli nie planujemy umieszczać na pendrive więcej partycji niż 4, to określamy wszystkie jako
podstawowe. Dalej podajemy początkowy sektor partycji. Zwykle zostanie automatycznie wykryty, tj.
zaraz za końcem poprzedniej partycji. Pierwsza partycja zawsze zaczyna się na 2048 sektorze, z racji
równania do 1 MiB. Na koniec podajemy rozmiar partycji, np. +3G (2 jako podstawa potęgi). Podobnie
postępujemy z kolejnymi partycjami, aż do wykorzystania całej wolnej przestrzeni nośnika.

![]({{< baseurl >}}/img/2016/04/4.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#huge)

Jeśli pomylimy się i określimy złe parametry partycji, to zawsze możemy taką partycję usunąć za
pomocą `d` :

![]({{< baseurl >}}/img/2016/04/5.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#huge)

Po utworzeniu wszystkich partycji, sprawdzamy wygląd tablicy partycji wpisując `p` . Jeśli wszystko
się zgadza, zapisujemy tablicę partycji przy pomocy `w` :

![]({{< baseurl >}}/img/2016/04/6.fdisk-openwrt-tworzenie-partycji-dysk-pendrive.png#huge)

Jeśli z jakiegoś powodu nie podoba nam się ten układ partycji albo zwyczajnie wybraliśmy nie ten
dysk co potrzeba, to wszelkie zmiany możemy cofnąć wpisując `q` zamiast tego powyższego `w` .

Samo stworzenie partycji to nie wszystko. Musimy jeszcze stworzyć system plików na tej partycji.
Jako, że zdecydowaliśmy się na system plików EXT4, to korzystamy z narzędzia `mkfs.ext4` :

    # mkfs.ext4 -m 0 -L dane /dev/sdb1

Wyżej podaliśmy dwa parametry do `mkfs.ext4` . Standardowo wszystkie linux'y rezerwują 5% wolnego
miejsca na partycji na potrzeby użytkownika root. Opcja `-m 0` wyłącza rezerwację. Z kolei opcja
`-L` nadaje systemowi plików odpowiednią etykietę. W przypadku, gdy byśmy tworzyli system plików na
partycji, która już posiada system plików, to zostaniemy o tym fakcie uprzedzeni i będziemy musieli
świadomie wyrazić zgodę na to:

![]({{< baseurl >}}/img/2016/04/6.mkfs-ext4-openwrt-tworzenie-system-plikow-dysk-pendrive.png#huge)

## Montowanie zasobów w OpenWRT

Mając przygotowany pendrive lub dysk możemy przejść do zamontowania poszczególnych jego partycji. By
zamontować przykładową partycję w katalogu `/mnt/` posługujemy się narzędziem `mount` , przykładowo:

    # mount /dev/sdb3 /mnt/

I od tej pory możemy zapisywać dane na partycji. Gdy skończymy się bawić, system plików należy
odmontować w poniższy sposób:

    # umount /mnt/

## Automatyczne montowanie zasobów w fazie boot z block-mount

Wyżej był przestawiony manualny sposób montowania zasobów. Działa on zawsze, z tym, że wymagana
naszej ingerencji za każdym razem, gdy resetujemy router. Jeśli chcemy mieć podpięty dysk albo
pendrive na stałe do routera i udostępniać system plików takiego urządzenia w sieci lokalnej, to
przydałoby się zautomatyzować montowanie zasobów przy starcie systemu. Za taką funkcjonalność
odpowiada [pakiet block-mount](https://wiki.openwrt.org/doc/techref/block_mount). Zainstalujmy go
zatem:

    # opkg install block-mount

Konfiguracja tego pakietu jest trzymana w pliku `/etc/config/fstab` . Standardowo jest on pusty i
musimy go sami uzupełnić. Możemy to zrobić przy pomocy polecenia `block` . Można mu podać kilka
opcji, min. `detect` , która wykrywa i konfiguruje wszystkie podpięte urządzenia. Wystarczy zatem
podpiąć pendrive czy dysk i wykonać to poniższe polecenie:

    # block detect > /etc/config/fstab

W przypadku, gdy mamy już jakieś wpisy w pliku `/etc/config/fstab` , to powyższe polecenie nadpisze
ją nam. Dlatego też nie kierujmy wyjścia tego polecenia do pliku. Jeśli tego nie zrobimy, to
konfiguracja zostanie nam wyświetlona w oknie terminala. Później możemy tylko skopiować odpowiednie
wpisy i umieścić w pliku `/etc/config/fstab` .

Poleceniu `block` może także podać opcję `info` , po której wywołaniu zostanie zwróconych trochę
informacji o podłączonych zasobach. Z kolei za pomocą opcji `mount` oraz `umount` możemy montować i
demontować zasoby w oparciu o konfigurację w pliku `/etc/config/fstab` .

Poniżej znajduje się przykładowy plik `/etc/config/fstab` :

    config 'global'
          option      anon_swap   '0'
          option      anon_mount  '0'
          option      auto_swap   '1'
          option      auto_mount  '1'
          option      delay_root  '5'
          option      check_fs    '0'

    config 'mount'
          option  target  '/mnt/sdb3/'
    #     option  device  '/dev/sdb3'
          option  uuid    '409d5fe7-46f0-4887-bf23-b3882e906c14'
          option  fstype  'ext4'
          option  enabled '1'

Wszystkie partycje, które chcemy montować automatycznie na starcie routera musimy umieścić w
sekcjach `config 'mount'` . Poza tymi sekcjami, mamy także ustawienia globalne, które są w sekcji
`config 'global'` . Na początek omówmy sobie sekcję z ustawieniami globalnymi.

Opcje `anon_swap` oraz `anon_mount` odpowiadają za montowanie zasobów, które nie posiadają
konfiguracji w pliku `/etc/config/fstab` . Lepiej nie ruszać ich wartości domyślnych. Następnie mamy
opcje `auto_swap` oraz `auto_mount` , które montują zasoby określone w pliku `/etc/config/fstab` i
mające status `enabled` (o tym za moment). Jeśli chodzi zaś o opcję `delay_root` , to ma ona
kluczowe znaczenie [w przypadku
extroot/whole_root]({{< baseurl >}}/post/extroot-whole_root-fullroot-pod-openwrt/). Może się tak
zdarzyć, że nasz pendrive zostanie wykryty z pewnym opóźnieniem podczas startu systemu routera. W
takim przypadku partycja nie zostanie zamontowana. Wartość tego parametru jest w sekundach i oznacza
ile czasu system ma czekać zanim zamontuje partycję. Ostatnia opcja, tj. `check_fs` , odpowiada za
sprawdzanie systemu plików aktywnych zasobów.

Dalej w pliku mamy sekcje `config 'mount'` . Opcje w niej trochę się różnią w stosunku do tych z
sekcji globalnej. Mamy tutaj opcję `target` , który określa gdzie zamontować daną partycję. W
`device` jest ścieżka do urządzenia/partycji. Niemniej jednak, w przypadku podpięcia innego
pendrive, OpenWRT będzie próbował zamontować jedną z jego partycji. Dlatego też, lepiej unikać
definiowania zasobów w taki sposób. Zamiast tego, lepiej jest wykorzystać opcję `uuid` , która
identyfikuje konkretne urządzenie (partycję) podpięte do portu USB routera. Następnie mamy opcję
`fstype` , która określa system plików partycji. Opcja `enabled` ustawiona na `1` sprawi, że zasób
zostanie zamontowany w fazie boot routera.

## Przestrzeń wymiany SWAP

OpenWRT to taka minimalistyczna dystrybucja linux'a. Obowiązują ją te same zasady, co każdego innego
pingwina w tej branży. Jak wiadomo, routery domowe mają niewiele pamięci operacyjnej RAM. W efekcie
niektórych aplikacji możemy nie być w stanie uruchomić. Jeśli routerowi braknie pamięci, to
zwyczajnie się powiesi. Możemy nieco złagodzić skutki niewielkich zasobów RAM za pomocą przestrzeni
wymiany SWAP. Takie rozwiązanie może nie nie powala pod względem osiągów ale jeśli alternatywą jest
kupienie nowego sprzętu, to co nam szkodzi rzucić okiem na to rozwiązanie?

Przestrzeń wymiany SWAP, to zwykła partycja. Każdy linux, w tym też i OpenWRT, ma możliwość jej
utworzenia. Wyżej stworzyliśmy przy pomocy `fdisk` kilka partycji. Jedna z nich w dalszym ciągu nie
ma utworzonego systemu plików. Przeznaczmy ją na SWAP. W terminalu wpisujemy zatem to poniższe
polecenie:

    # mkswap /dev/sdb4

Tak utworzoną przestrzeń wymiany można aktywować i dezaktywować ręcznie w poniższy sposób:

    # swapon /dev/sdb4
    # swapoff /dev/sdb4

Statystyki wykorzystania SWAP można odczytać przez `swapon -s` :

    # swapon -s
    Filename                      Type        Size  Used  Priority
    /dev/sdb4                               partition     2097148     0     -1

Podobnie jak w przypadku zwykłych partycji, również i przestrzeń wymiany może być automatycznie
montowana wraz ze startem systemu routera. W tym celu musimy dopisać sekcję `config 'swap'` do pliku
`/etc/config/fstab` . Poniżej przykład:

    config 'swap'
    #     option device '/dev/sda3'
          option  uuid    '1bd99fa0-bba9-453e-b042-1e593082f88e'
          option  enabled '1'

W przypadku włączenia SWAP, trzeba rozważyć przestawienie jednego parametru kernela. Mowa o
`/proc/sys/vm/swappiness` . Domyślna wartość w tym pliku to `60`. Zakres na jakim możemy operować to
od 0-100. Im większa wartość tym kernel chętniej zrzuca dane z RAM do SWAP. Parametry kernela
ustawiamy w pliku `/etc/sysctl.conf` . Dopisujemy tam poniższą linijkę:

    vm.swappiness = 30

Aplikujemy zmiany przez wpisanie w terminalu `sysctl -p` . Następnie odczytujemy wartość tego
parametru by się upewnić czy został poprawnie ustawiony:

    # sysctl -a | grep swap
    vm.swappiness = 30

Ten parametr będzie ustawiany automatycznie za każdym razem przy starcie systemu.
