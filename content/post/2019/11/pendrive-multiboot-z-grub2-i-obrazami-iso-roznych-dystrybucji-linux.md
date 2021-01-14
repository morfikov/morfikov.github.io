---
author: Morfik
categories:
- Linux
date: "2019-11-08T18:02:33Z"
published: true
status: publish
tags:
- grub
- iso
- img
- pendrive
- multiboot
- live
title: Pendrive multiboot z GRUB2 i obrazami ISO różnych dystrybucji Linux
---

Obrazy ISO różnych dystrybucji Linux, szczególnie te live, bywają niezastąpione w sytuacjach
kryzysowych. Dzięki takiej płytce CD/DVD czy pendrive (może być też i karta SD) można wybrnąć nawet
z najgorszych opresji bez potrzeby rezygnowania przy tym z graficznego środowiska pracy
podłączonego do internetu. Zwykle jednak użytkownicy są stawiani przed wyborem systemu, który mogą
sobie wgrać na zewnętrzny nośnik, by w późniejszym czasie przeprowadzać ewentualne prace naprawcze.
Chodzi generalnie o fakt, że taki obraz ISO czy IMG przy wgrywaniu konsumuje całe urządzenie bez
względu na jego rozmiar, i tak mając 32G pamięci na flash możemy wgrać w zasadzie tylko jeden
obraz, np. Debiana, a by wgrać obraz Ubuntu, to już trzeba albo osobnego pendrive albo nadpisać ten
poprzednio wgrany obraz. Takie rozwiązanie jest mało praktyczne i też generuje koszty. Na szczęście
można stworzyć boot'owalny pendrive (w oparciu o GRUB/GRUB2), na którym można umieścić dowolną
ilość obrazów ISO i w fazie rozruchu wybrać sobie ten system, który nas interesuje, a wszystko
dzięki [projektowi GLIM][1] (GRUB Live ISO Multiboot).

<!--more-->
## Czym jest GLIM

GLIM to w zasadzie kawałek prostego skryptu shell'owego (bash), który ma za zadanie umożliwić nam
wykorzystanie odpowiednio sformatowanego wcześniej nośnika (pendrive lub karta SD) pod zapis wielu
obrazów ISO udostępnianych przez szereg dystrybucji Linux (m.in. `debian` i `ubuntu` ), oraz
narzędzi typu `clonezilla` czy `gparted` . Pełna lista wpieranych obrazów znajduje się poniżej:

    antergos, antix, arch, bodhi, centos, clonezilla, debian, elementary,
    fedora, gparted, grml, ipxe, kali, linuxmint, manjaro, netrunner,
    peppermint, porteus, rhel, sabayon, supergrub2disk, sysrescd, tails,
    ubuntu, void, xubuntu

Zatem jak widać jest ich całkiem sporo. Dodatkowo, nie jesteśmy limitowani przez jedną wersję
obrazu, którą możemy trzymać na pendrive. Możemy ich mieć kilka, a podczas fazy boot będziemy w
stanie wybrać sobie tę wersję, która nam odpowiada, co przydaje się w przypadku obrazów testowych
czy niestabilnych takich dystrybucji jak Debian, gdzie często te obrazy potrafią zawierać błędy
uniemożliwiające start/instalację systemu.

### Bootloader GRUB/GRUB2

Sam skrypt działa w oparciu o bootloader GRUB/GRUB2, a konkretnie wołany jest `grub-install` .
Wymagane zatem jest posiadanie w systemie pakietu `grub2` (lub też `grub-legacy` ). Problem możemy
napotkać w sytuacji, gdy korzystamy z innego bootloader'a do rozruchu naszego Linux'a. Dla
przykładu, ja odpalam swojego Debiana przy pomocy `syslinux`/`extlinux` , a przy instalacji pakietu
`grub2` pojawia się informacja o próbie zainstalowania tego bootloder'a w MBR dysku (i na partycji
`/boot/` ). Niemniej jednak, nie musimy instalować GRUB'a na dysku, by móc korzystać z narzędzi
dostarczanych w zainstalowanych w systemie pakietach.

#### Konfiguracja

Bootloader GRUB wymaga pliku konfiguracyjnego ( `grub.cfg` ). Nie musimy na szczęście pisać go sami,
bo jest on dostarczany przez projekt GLIM. Podczas instalacji GRUB'a na pendrive czy karcie SD, ta
konfiguracja zostanie automatycznie wgrana w stosowne miejsce na nośniku.

### Sudo

Niektóre polecenia w skrypcie są wołane przy pomocy `sudo` . Można albo nieco ten skrypt przerobić
sobie, albo też dopisać poniższą linijkę do konfiguracji `sudo` (za pomocą `visudo` ) ,
przynajmniej na czas wykonania się samego skryptu:

    ...
    Host_Alias HOSTY = localhost, morfikownia, morfikownia.mhouse
    ...
    morfik      HOSTY = (root) NOPASSWD: /usr/sbin/grub-install

### Zmienna $PATH

W moim Debianie zmienna `$PATH` dla zwykłego użytkownika jest ustawiona w poniższy sposób:

    $ echo $PATH
    /usr/local/bin:/usr/bin:/bin:/usr/games

Biorąc teraz pod uwagę fakt, że `grub-install` znajduje się w `/sbin/` , to skrypt GLIM będzie miał
problem ze znalezieniem tego polecenia. Można albo wyeksportować w terminalu zmienną `$PATH` tak,
by uwzględniała `/sbin/` , albo też zatroszczyć się o eksport tej zmiennej w samym skrypcie:

    export PATH=$PATH:/usr/sbin

## Przygotowanie nośnika pod GLIM

Możemy teraz przejść do przygotowania naszego pendrive albo karty SD pod GLIM. Przede wszystkim, na
takim nośniku trzeba utworzyć tablicę partycji (zalecane MBR/MS-DOS) oraz tylko jedną partycję
sformatowaną jakimś systemem plików. Teoretycznie można wykorzystać dowolny system plików, np.
`FAT32` czy `EXT4` , a nawet `NTFS` ale trzeba pamiętać o fakcie, że wybór innego systemu plików
niż `FAT32` może generować problemy z uruchamianiem obrazów ISO. Dlatego też lepiej pozostać przy
tej bezpiecznej opcji, nie zapominając przy tym, że `FAT32` ma ograniczenie co do wielkości
obsługiwanych plików  i jest to 4 GiB, przez co większych obrazów nie będziemy w stanie wgrać na
pendrive/kartę SD. Na szczęście obrazy live zwykle trzymają się poniżej 3 GiB i nie musimy się tym
limitem zbytnio przejmować. Dodatkowo, trzeba ustawić etykietę systemu plików na `GLIM` , w
przeciwnym razie szukana partycja nie zostanie odnaleziona przez skrypt.

Wszystkie te wyżej wymienione rzeczy bez problemu można sobie wyklikać w `gparted` , zatem nie ma
co tutaj opisywać tego procesu. Poniżej znajduje się fotka odpowiednio przygotowanego nośnika:

![](/img/2019/11/001.pendrive-karta-sd-multiboot-iso-linux-gparted.png#huge)

### Instalacja GRUB'a

Zostało nam już w zasadzie zainstalowanie bootloader'a na pendrive, a tym zadaniem zajmie się już
sam skrypt `glim.sh` . Montujemy zatem system plików nośnika, który przygotowaliśmy sobie wcześniej,
w dowolnym miejscu i uruchamiamy skrypt. Jeśli nośnik został przygotowany poprawnie, to cały proces
instalacji GRUB'a powinien się dokonać bez większych problemów, jedynie trzeba potwierdzić
przeprowadzenie operacji, zatem warto też zweryfikować co tam na ekranie się pojawi.

#### Wersja z EFI/UEFI

Jeśli mamy na wyposażeniu komputer z EFI/UEFI (lub zamierzamy używać naszego pendrive na tego typu
maszynach), to przydałoby się stworzyć nośnik, który będzie uruchamiany w natywnym trybie EFI, a
nie w trybie kompatybilności BIOS za sprawą modułu CSM (Compatibility Support Module), co może
prowadzić do [szeregu problemów][2]. By stworzyć pendrive, który będzie zawierał niezbędne elementy,
trzeba doinstalować sobie w systemie pierw pakiet `grub-efi-amd64-bin` , a później przy instalacji
GLIM odpowiedzieć twierdząco na poniższe pytanie:

    $ ./glim.sh
    Found partition with label 'GLIM' : /dev/sdc1
    Found block device where to install GRUB2 : /dev/sdc
    Found mount point for filesystem : /media/morfik/GLIM
    Install for EFI in addition to standard BIOS? (Y/n) y

W przypadku, gdy nie będziemy posiadać w systemie wspomnianego wyżej pakietu, to instalator nam
wyrzuci poniższe ostrzeżenie o braku katalogu `/usr/lib/grub/x86_64-efi/` :

    WARNING: no /usr/lib/grub/x86_64-efi dir (grub2-efi-x64-modules rpm missing?)

Gdy jesteśmy gotowi do instalacji GLIM'a, to również twierdząco odpowiadamy na poniższe pytanie:

    Ready to install GLIM. Continue? (Y/n) y
    Running grub-install --target=i386-pc --boot-directory=/media/morfik/GLIM/boot /dev/sdc (with sudo) ...
    Installing for i386-pc platform.
    Installation finished. No error reported.
    Running grub-install --target=x86_64-efi --efi-directory=/media/morfik/GLIM --removable --boot-directory=/media/morfik/GLIM/boot /dev/sdc (with sudo) ...
    Installing for x86_64-efi platform.
    Installation finished. No error reported.
    Running rsync -rpt --delete --exclude=i386-pc --exclude=x86_64-efi --exclude=fonts --exclude=icons/originals ./grub2/ /media/morfik/GLIM/boot/grub ...
    GLIM installed! Time to populate the boot/iso directory.

### Obrazy ISO z dystrybucjami Linux

Po zainstalowaniu bootloader'a na pendrive można przejść do wgrywania obrazów ISO. W tym celu
tworzymy pierw folder pod konkretną dystrybucję w katalogu `boot/iso/` na nośniku, np.
`/media/morfik/GLIM/boot/iso/debian/` i w tym katalogu umieszczamy obrazy charakterystyczne dla tej
dystrybucji Linux. Jeśli chcielibyśmy wgrać obraz `gparted` , to tworzymy katalog
`/media/morfik/GLIM/boot/iso/gparted/` i to w nim umieszczamy stosowny plik.

Nazwa pliku nie ma zbytnio znaczenia, tj. powinna ona pasować do konkretnego wyrażenia regularnego
(można podejrzeć w plikach `inc-*.cfg` w katalogu `boot/grub/`) i w zasadzie oryginalne nazwy
obrazów pobranych ze stron projektów będą w 100% zgodne, wliczając w to nowsze wersje obrazów (inne
numerki). Zatem nie musimy w zasadzie nic robić, za wyjątkiem wgrania pobranego z sieci obrazu na
nasz pendrive czy kartę SD.

## Test multiboot

Menu GRUB, które zostanie nam zaprezentowane podczas fazy boot, jest generowane w locie w oparciu o
konfigurację dostarczoną przez projekt GLIM. Każda dystrybucja, której katalog stworzyliśmy w
`boot/iso/` będzie miała swoją pozycję w menu, a jeśli dodatkowo umieścimy w takim katalogu choć
jeden obraz, to będzie go można wybrać w submenu. Poniżej jest poglądowa fotka zaczerpnięta z git'a
projektu GLIM prezentująca menu:

![](/img/2019/11/002.pendrive-karta-sd-multiboot-iso-linux-grub-menu.png#huge)

Oczywiście w zależności od tego co sobie wgramy na pendrive/kartę SD, to inne pozycje w menu
zobaczymy ale najważniejszy jest fakt, że wystarczy nam już tylko jedno urządzenie, by w wygodny
sposób przechowywać obrazy udostępniane przez rozmaite dystrybucje Linux i uruchamiać dowolny
system, który nam odpowiada i gdy zajdzie taka potrzeba.


[1]: https://github.com/thias/glim
[2]: https://www.rodsbooks.com/efi-bootloaders/csm-good-bad-ugly.html
