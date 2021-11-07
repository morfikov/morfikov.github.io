---
author: Morfik
categories:
- RaspberryPi
- Linux
date:    2021-11-07 10:20:00 +0100
lastmod: 2021-11-07 10:20:00 +0100
published: true
status: publish
tags:
- raspberry-pi-4b
- raspios
- raspbian
- debian
- qemu
- kernel
- moduły-kernela
GHissueID: 578
title: Chroot do 32-bit systemu ARM z poziomu 64-bit linux'owego hosta
---

Eksperymentując ostatnio z moją maszynką Raspberry Pi 4B, zaszła potrzeba, by zejść do systemu
RasPiOS/Raspbian przy pomocy mechanizmu chroot. Problem w tym, że system wyrzuca  komunikat:
`chroot: failed to run command ‘/bin/bash’: Exec format error` . Niby wszystko jest na swoim
miejscu ale ta widoczna wyżej wiadomość nie chce zniknąć uniemożliwiając tym samym dalszą zabawę z
RPI. Okazało się, że winna jest tutaj architektura CPU. Mój laptop działa pod kontrolą 64-bitowego
Intel'owskiego procesora (x64, x86-64, AMD64), na którym uruchomiony jest również 64-bitowy Debian
linux. Z kolei Raspberry Pi ma 64-bitowy procesor ARM (ARMv8-A) działający pod kontrolą 32-bitowego
systemu operacyjnego. Te dość spore rozbieżności sprawiają, że nie damy rady skorzystać z chroot,
przynajmniej nie bez zaprzęgnięcia do tego celu emulatora QEMU.

<!--more-->
## Instalacja QEMU

Warto na samym początku zaznaczyć, że nie będziemy tworzyć żadnych [maszyn wirtualnych na bazie
QEMU/KVM][3], a jedynie będziemy korzystać z [emulatora QEMU][4]. Niemniej jednak, by móc z tego
emulatora skorzystać, musimy w naszym linux'ie zainstalować te trzy poniższe pakiety:

    # aptitude install qemu binfmt-support qemu-user-static

Przy instalacji tych powyższych pakietów, w logu systemowym powinny zostać zanotowane te poniższe
komunikaty:

	systemd[1]: proc-sys-fs-binfmt_misc.automount: Got automount request for /proc/sys/fs/binfmt_misc, triggered by 9794 (update-binfmts)
	systemd[1]: Mounting proc-sys-fs-binfmt_misc.mount...
	systemd[1]: Mounted proc-sys-fs-binfmt_misc.mount..
	systemd[1]: Starting binfmt-support.service...
	systemd[1]: Finished binfmt-support.service.

Następnie przy pomocy `update-binfmts` sprawdzamy czy wpisy binfmt zostały z powodzeniem
zarejestrowane. Nas najbardziej interesuje zwrotka z `qemu-arm` :

	# update-binfmts --display
	...
	qemu-arm (enabled):
		 package = qemu-user-static
			type = magic
		  offset = 0
		   magic = \x7f\x45\x4c\x46\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00
			mask = \xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff
	 interpreter = /usr/libexec/qemu-binfmt/arm-binfmt-P
		detector =
	...

Gdyby po instalacji tych powyższych pakietów przy `qemu-arm` widniał status `(disabled)` , to
prawdopodobnie oznaczałoby to, że [brakuje w kernelu modułu binfmt_misc][1], za który to odpowiada
opcja `CONFIG_BINFMT_MISC=y` .

## Chroot do systemu RasPiOS/Raspbian

Montujemy teraz system plików partycji `/` naszego Raspberry Pi w przykładowym katalogu. Montujemy
też katalogi `/dev/` , `/dev/pts/` , `/sys/` , `/proc/` oraz `/run/` z opcją `bind` , po czym
wydajemy polecenie `chroot` :

    # mount /dev/sdb2 /media/rpi

    # for f in dev dev/pts sys proc run ; do mount -o bind /$f /media/rpi/$f ; done

    # chroot /media/rpi /bin/bash
    root@morfikownia:/#

Jak widać, tym razem dało radę zrobić chroot bez najmniejszego problemu. Dla pewności możemy
jeszcze sprawdzić co zwróci nam polecenie `lsb_release` , które zostanie uruchomione wewnątrz
środowiska chroot:

    root@morfikownia:/# lsb_release -a
    No LSB modules are available.
    Distributor ID: Raspbian
    Description:    Raspbian GNU/Linux 10 (buster)
    Release:        10
    Codename:       buster

Zatem nie ma już chyba wątpliwości, że udało nam się zrobić chroot na 32-bitowy system ARM z
poziomu 64-bitowego systemu x86-64.

## Problemy z QEMU

Na [wiki Debiana][2] można wyczytać, że starsze wersje tej dystrybucji linux'a miały problemy z
tego typu chroot. W Debian Stretch (i starszych) trzeba było udostępnić emulator QEMU w środowisku
chroot przez skopiowanie (lub zamontowanie z opcją `bind` ) pliku `/usr/bin/qemu-arm-static` ,
przykładowo:

    # cp /usr/bin/qemu-arm-static /media/rpi/usr/bin

lub:

    # mount --bind /usr/bin/qemu-arm-static /media/rpi/usr/bin

W nowszych dystrybucjach Debiana już nie trzeba tego robić.


[1]: https://en.wikipedia.org/wiki/Binfmt_misc
[2]: https://wiki.debian.org/QemuUserEmulation
[3]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/
[4]: https://www.qemu.org/
