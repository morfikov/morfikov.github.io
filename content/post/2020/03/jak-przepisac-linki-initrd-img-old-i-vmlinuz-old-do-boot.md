---
author: Morfik
categories:
- Linux
date: "2020-03-01T20:30:00Z"
published: true
status: publish
tags:
- debian
- kernel
- initrd
- initramfs
- efi
- uefi
- refind
- luks
- lvm
title: Jak przepisać linki initrd.img{,.old} i vmlinuz{,.old} z / do /boot/
---

Mając możliwość skonfigurowania EFI/UEFI na moim laptopie, postanowiłem jak najbardziej się za to
przedsięwzięcie zabrać. Okazało się jednak, że w przypadku takiej dystrybucji linux'a jak Debian,
to zadanie może być nieco problematyczne, zwłaszcza gdy chce się korzystać jedynie z menadżera
rozruchu jakim jest [rEFInd][1], czyli bez dodatkowego bootloader'a (grub/grub2/syslinux/extlinux)
instalowanego bezpośrednio na dysku twardym i jednocześnie posiadając w pełni zaszyfrowany system
(LUKSv2 + LVM). Rzecz w tym, że w takiej sytuacji w konfiguracji rEFInd trzeba podawać ścieżki
bezpośrednio do plików `initrd.img` oraz `vmlinuz` (obecnych na partycji `/boot/` ). W Debianie
nazwy tych plików mają format `initrd.img-x.x.x-x-amd64` i `vmlinuz-x.x.x-x-amd64` . Za każdym
razem, gdy wypuszczany jest nowy kernel, to ten numerek ( `x.x.x-x` ) ulega zmianie, co pociąga za
sobą potrzebę ręcznego dostosowania konfiguracji rEFInd. Może i aktualizacje kernela w Debianie nie
są jakoś stosunkowo częste ale może istnieje sposób, by ten problem z dostosowaniem konfiguracji
rozwiązać?

<!--more-->
## Symboliczne linki do initrd.img i vmlinuz w /

Z reguły oprogramowanie mające na celu uruchomienie naszego linux'a jest w stanie bez większego
problemu automatycznie dostosować konfigurację bootloader'a, tak by w przypadku aktualizacji
kernela użytkownik nie musiał przeprowadzać żadnych dodatkowych czynności. Są jednak pewne
specyficzne instalacje, np. wspomniany we wstępie w pełni zaszyfrowany system w konfiguracji EFI +
rEFInd, gdzie dodatkowe kroki trzeba poczynić, by system działał nam jak należy. Czasami można
jednak pewne rzeczy zautomatyzować. Przeglądając strukturę drzewa katalogów, można w katalogu root
( `/` ) natrafić na takie poniższe wpisy:

    # ls -al /
    ...
    lrwxrwxrwx   1 root   root           29 2020-02-14 17:22:18 initrd.img -> boot/initrd.img-5.4.0-4-amd64
    lrwxrwxrwx   1 root   root           27 2020-02-24 00:37:53 initrd.img.old -> boot/initrd.img-5.4.0-3-amd64
    ...
    lrwxrwxrwx   1 root   root           26 2020-02-14 17:22:18 vmlinuz -> boot/vmlinuz-5.4.0-4-amd64
    lrwxrwxrwx   1 root   root           24 2020-02-24 00:37:53 vmlinuz.old -> boot/vmlinuz-5.4.0-3-amd64

Mamy tutaj linki symboliczne do plików kernela i obrazu initrd, które znajdują się w katalogu
`/boot/` . Podając ścieżki do tych dowiązań w konfiguracji bootloader'a, tj. do `/vmlinuz` zamiast
bezpośrednio do `/boot/vmlinuz-5.4.0-4-amd64` (i podobnie w przypadku obrazu initrd) można
zaoszczędzić sobie trochę pracy w sytuacji, gdy kernel będzie aktualizowany -- ten link zostanie
automatycznie przepisany i będzie wskazywał na odpowiedni plik w katalogu `/boot/` .

Tworzeniem tych powyższych dowiązań do plików `initrd.img` oraz `vmlinuz` zajmuje się [narzędzie
linux-update-symlinks][2] . Jest ono standardowo zainstalowane w każdym Debianie, zatem nie trzeba
go dodatkowo wgrywać. Ten cały `linux-update-symlinks` jest wołany w skryptach maintainer'a
( `postinst` i `postrm` ) obecnych w paczkach kernela ( `linux-image-*.deb` ). Za każdym razem, gdy
tylko instalowany/usuwany jest kernel, to wołane jest jedno z tych poniższych poleceń:

    # linux-update-symlinks install $version $image_path
    # linux-update-symlinks upgrade $version $image_path
    # linux-update-symlinks remove  $version $image_path

W zasadzie to `linux-update-symlinks` przyjmuje trzy argumenty. Pierwszy z nich określa czy
zamierzamy te linki stworzyć ( `install` ), zaktualizować ( `upgrade` ) lub usunąć ( `remove` ).
Drugi argument określa wersję kernela (w formacie, który można uzyskać via `uname -r` ), natomiast
trzeci parametr definiuje ścieżkę do pliku `vmlinuz` .

W standardowej konfiguracji, `linux-update-symlinks` stworzy nam dowiązania w katalogu `/` . Warto
zaznaczyć tutaj, że nie podaje się nigdzie ścieżki do obrazu initrd ale mimo to link do niego
zostanie stworzony poprawnie. Jeśli jednak tego typu zachowanie niezbyt nam się podoba i
potrzebujemy tych dowiązań w innym katalogu, np. w `/boot/` , to musimy to narzędzie sobie inaczej
skonfigurować.

## Jak tworzyć linki do initrd.img i vmlinuz w /boot/

W przypadku, gdy mamy do czynienia z w pełni zaszyfrowanym system, to te linki w `/` są dla nas
praktycznie bezużyteczne. By się do nich dostać trzeba odszyfrować główny system plików, a by go
odszyfrować, trzeba załadować pierw kernel. Z kolei zaś jeśli mamy załadowany kernel w pamięci RAM,
to nie potrzebne nam są już linki. Możemy więc albo podawać pełne ścieżki do plików
`initrd.img-5.4.0-4-amd64` i `vmlinuz-5.4.0-4-amd64` i edytować za każdym razem konfigurację
rEFInd, albo też możemy dopisać poniższy parametr do pliku `/etc/kernel-img.conf` (jeśli plik nie
istnieje, to trzeba go stworzyć):

    link_in_boot = Yes

Zgodnie z [man kernel-img.conf][3], parametr `link_in_boot` (jeśli ustawiony) ma za zadanie tworzyć
linki w katalogu `/boot/` . Warto jest sobie rzucić okiem na ten manual, bo w pliku
`/etc/kernel-img.conf` opcji, których możemy określić, jest nieco więcej. Tak czy inaczej na nasze
potrzeby wystarczy ta powyższa.

Od tego momentu, za każdym razem jak tylko nowy kernel będzie instalowany w systemie, to te linki
będą już tworzone w katalogu `/boot/` i o takie zachowanie nam cały czas chodziło.

### Ręcznie tworzenie linków

Jeśli z jakiegoś powodu zajdzie potrzeba, by linki do plików `initrd.img` i `vmlinuz` stworzyć
samodzielnie, to poniżej jest przykładowe polecenie, którym możemy się posłużyć (pamiętajmy by
odpowiednio dobrać numerki):

    # linux-update-symlinks install 5.4.0-4-amd64 /boot/vmlinuz-5.4.0-4-amd64
    I: /boot/vmlinuz.old is now a symlink to vmlinuz-5.4.0-3-amd64
    I: /boot/initrd.img.old is now a symlink to initrd.img-5.4.0-3-amd64
    I: /boot/vmlinuz is now a symlink to vmlinuz-5.4.0-4-amd64
    I: /boot/initrd.img is now a symlink to initrd.img-5.4.0-4-amd64

Dla pewności możemy jeszcze sprawdzić, czy aby na pewno te linki zostały utworzone w pożądanej
przez nas lokalizacji:

    $ ls -al /boot/ | egrep  "vmlinuz|initrd"
    lrwxrwxrwx  1 root root       22 2020-03-01 15:18:21 initrd.img -> initrd.img-5.4.0-4-amd64
    -rw-r--r--  1 root root 39127233 2020-02-14 17:23:07 initrd.img-5.4.0-3-amd64
    -rw-r--r--  1 root root 16005450 2020-03-01 14:41:38 initrd.img-5.4.0-4-amd64
    lrwxrwxrwx  1 root root       24 2020-03-01 15:18:21 initrd.img.old -> initrd.img-5.4.0-3-amd64
    lrwxrwxrwx  1 root root       19 2020-03-01 15:18:21 vmlinuz -> vmlinuz-5.4.0-4-amd64
    -rw-r--r--  1 root root  5627632 2020-02-13 06:14:49 vmlinuz-5.4.0-3-amd64
    -rw-r--r--  1 root root  9331760 2020-02-26 09:38:52 vmlinuz-5.4.0-4-amd64
    lrwxrwxrwx  1 root root       21 2020-03-01 15:18:21 vmlinuz.old -> vmlinuz-5.4.0-3-amd64

### A co ze starymi linkami w /

No jak widać wyżej, linki są już obecne na partycji `/boot/` i wskazują na odpowiednie pliki.
Niemniej jednak, stare linki w `/` są nadal dostępne, mimo, że system już raczej nie powinien sobie
nimi głowy zawracać. Nieaktualne linki w `/` mogą być przyczyną późniejszych problemów, dlatego też
przydałoby się te linki zwyczajnie usunąć. Możemy to zrobić poniższym poleceniem:

    # rm /vmlinuz /initrd.img /initrd.img.old /vmlinuz.old



[1]: https://www.rodsbooks.com/refind/
[2]: https://manpages.debian.org/unstable/linux-base/linux-update-symlinks.1.en.html
[3]: https://manpages.debian.org/unstable/linux-base/kernel-img.conf.5.en.html
