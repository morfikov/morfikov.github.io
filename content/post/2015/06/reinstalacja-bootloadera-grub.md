---
author: Morfik
categories:
- Linux
date: "2015-06-16T21:13:57Z"
date_gmt: 2015-06-16 19:13:57 +0200
published: true
status: publish
tags:
- hdd
- ssd
- bootloader
- grub
GHissueID: 133
title: Reinstalacja bootloadera grub
---

Domyślnym bootloaderem w systemie linux jest [grub](https://www.gnu.org/software/grub/) i jako że
to oprogramowanie jest ładowane do pamięci jako pierwsze, ma ono kluczowe zadanie w procesie startu
systemu operacyjnego. Przy jego pomocy możemy także przekazać szereg parametrów dla modułów kernela,
tym samym odpowiednio go konfigurując. Czasem z pewnych przyczyn, najczęściej gdy inny system
nadpisze MBR, system operacyjny nie chce się podnieść i musimy przeinstalować bootloader,
zakładając, że problem tkwi w nim.

<!--more-->
## Środowisko chroot

Jeśli nie możemy się dostać do naszego głównego systemu, musimy posłużyć się jakimś innym. Najlepsze
do tego są [systemy live](/post/wlasny-system-live-i-tworzenie-go-od-podstaw/).
Oczywiście możemy [pobrać gotowe obrazy](https://www.debian.org/CD/live/) i wypalić je na płytce czy
wrzucić na pendrive, po czym odpalić taki system i zamontować zasoby dysku wykorzystując do tego
[środowisko chroot](/post/przygotowanie-srodowiska-chroot-do-pracy/).

## Naprawa bootloadera grub

W przypadku gdy uszkodzeniu/nadpisaniu uległ jedynie sektor MBR, to zadanie sprowadza się do
przeinstalowania bootloadera przy pomocy poniższego polecenia:

    # grub-install /dev/sda

Czasem jednak sprawa może wyglądać nieco poważniej, np. możemy nie mieć plików na partycji `/boot/`
(lub ten katalog może być pusty) z jakiegoś powodu, np. gdy ktoś próbował skompromitować szyfrowany
system i podjęliśmy procedurę jego odzysku. W takim przypadku zaczynamy od przeinstalowania
wszystkich potrzebnych pakietów:

    # aptitude reinstall grub-common grub-pc grub-pc-bin grub2-common os-prober

Teraz już zostało nam zainstalowanie same bootloadera na dysku systemowym oraz wygenerowanie jego
konfiguracji przy pomocy tych dwóch poleceń:

    # grub-install /dev/sda
    # update-grub
