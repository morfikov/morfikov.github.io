---
author: Morfik
categories:
- Linux
date: "2015-06-11T23:24:07Z"
date_gmt: 2015-06-11 21:24:07 +0200
published: true
status: publish
tags:
- chroot
GHissueID: 137
title: Przygotowanie środowiska chroot do pracy
---

Linuxy mają tę właściwość, że bardzo ciężko jest stracić do któregoś z nich dostęp, nawet w
przypadku kompletnego zawału systemu. Jeżeli dysponujemy jakimś alternatywnym środowiskiem w postaci
płytki live cd/dvd czy pendrive albo też posiadamy gdzieś zainstalowanego innego linuxa, to istnieje
spore prawdopodobieństwo, że uda się nam reanimować nasz główny system. Wszystko za sprawą narzędzia
jakim jest [chroot](https://pl.wikipedia.org/wiki/Chroot) , przy którego to pomocy możemy zmienić
główny katalog systemu plików ( `/` ) dla wykonywanych procesów bez potrzeby przechodzenia całej
skomplikowanej procedury uruchamiania systemu operacyjnego. Jeśli tylko uda nam się uzyskać dostęp
do shella, to nie ma takiej możliwości by system nie stanął na nogi.

<!--more-->
## Katalogi niezbędne wewnątrz chroot

Sam chroot na nic nam się zda bez przygotowania odpowiednich katalogów i chodzi tutaj o te, bez
których system jest praktycznie bezużyteczny. W ich skład wchodzą te poniższe:

  - `/proc/` -- tu kernel przechowuje głównie informacje o uruchomionych procesach oraz parametrach
    pracy systemu operacyjnego, dotyczące procesora, pamięci, czy przerwań systemowych.
  - `/sys/` -- mniej więcej to samo co u poprzednika za wyjątkiem procesów. Generalnie rzecz biorąc,
    jest to nowsza i bardziej strukturalna wersja katalogu `/proc/` , w której znajdują się
    informacje na temat urządzeń, sterowników, magistral i ich wzajemnych powiązań.
  - `/dev/` -- tutaj są zlokalizowane pliki wszystkich wykrytych urządzeń w systemie, do których
    procesy mają dostęp i z którymi mogą się porozumiewać.
  - `/dev/pts/` -- ten katalog zawiera jedynie pliki urządzeń pseudo terminali, tak by procesy były
    w stanie przesyłać dane na wyjście i pobierać dane wejściowe [w tym samym czasie bez
    nieporozumień](https://unix.stackexchange.com/questions/93531/what-is-stored-in-dev-pts-files-and-can-we-open-them)
    i by było wiadomo, np. które dane wyjściowe należą do jakiego programu.

Wszystkie powyższe katalogi są generowane automatycznie podczas każdego startu systemu, a ich
zawartość jest przechowywana w pamięci RAM. Jeśli teraz podnosimy system za pomocą chroot, to w tym
środowisku nie ma obecnych tych wszystkich plików, a one są potrzebne do prawidłowej pracy systemu,
dlatego też musimy je pierw zamontować zanim zaczniemy operować wewnątrz takiego systemu.

## Przykład zastosowania chroot

Zakładając, że nasz padnięty system znajduje się na partycji `sdb2` , montujemy ją na samym
początku, np. w katalogu `/mnt/` :

    # mount /dev/sdb2 /mnt/

Następnie przy pomocy narzędzia `mount` z opcją `bind` odpowiadającą za możliwość zamontowania
jednego zasobu w wielu lokalizacjach, montujemy poszczególne pozycje wewnątrz struktury katalogów w
folderze `/mnt/` :

    # mount -o bind /proc /mnt/proc
    # mount -o bind /sys /mnt/sys
    # mount -o bind /dev /mnt/dev
    # mount -o bind /dev/pts /mnt/dev/pts

Dobrze jest także przekopiować plik `resolv.conf` ale to tylko na wypadek gdybyśmy nie mieli
połączenia z internetem wewnątrz środowiska chroot:

    # cp /etc/resolv.conf /mnt/etc/resolv.conf

Po tych działaniach możemy w końcu przejść do swojego głównego systemu:

    # chroot /mnt/ /bin/bash

Parametr `/bin/bash` jest opcjonalny, używany jedynie w przypadku ustawienia konkretnej powłoki
systemowej. I to cały proces odzyskiwania kontroli nad systemem. Oczywiście pozostaje nam jeszcze
jego naprawa ale teraz już powinno być z górki.
