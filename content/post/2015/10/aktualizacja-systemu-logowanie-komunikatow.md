---
author: Morfik
categories:
- Linux
date: "2015-10-08T12:39:32Z"
date_gmt: 2015-10-08 10:39:32 +0200
published: true
status: publish
tags:
- debian
- apt
- aptitude
title: Aktualizacja systemu i logowanie komunikatów
---

Aktualizacja systemu to chyba jedna z bardziej podstawowych czynności, które przeprowadzamy niemalże
codziennie. Tak się złożyło, że chwilę po zakończeniu tego procesu musiałem wyłączyć w pośpiechu
komputer. Nie zdążyłem przy tym przeczytać uważnie informacji, które zwrócił mi terminal. Oczywiście
mógłbym zahibernować maszynę i wrócić do logu instalacji w wolnej chwili ale nie zawsze hibernacja
jest możliwa. Poza tym, na myśl przychodzą mi osoby, które często zakładają wątki na forach o tym,
że aktualizacja uwaliła ich system. Zawsze w takiej sytuacji prosi się danego człowieka o podanie
logu z aktualizacji systemu albo przynajmniej próbuje się wyciągnąć od takiego delikwenta informację
na temat tego co było aktualizowane. W większości przypadków, taki człowiek nie ma o tym kompletnie
pojęcia, a jak już, to podaje bardzo nieprecyzyjne dane. Ten post ma na celu ułatwienie znalezienia
informacji o tym co było przedmiotem aktualizacji, tak by mieć nieco jaśniejszy obraz tego co mogło
nawalić.

<!--more-->
## Aktualizacja systemu za pomocą apt/aptitude

W debianie aktualizacja systemu jest przeprowadzana za pomocą jednego z dwóch narzędzi: `apt` lub
`aptitude` . Każde z nich posiada swoje pliki logu, w których zapisuje informacje z przeprowadzanych
operacji. Znajdują się one odpowiednio w `/var/log/apt/` oraz w `/var/log/aptitude` . W katalogu
`/var/log/apt/` jest kilka innych plików ale o tym za moment. Dodatkowo, aktualizacja systemu (jak i
instalowanie/usuwanie pakietów) przy pomocy `aptitude` jest także logowana do plików w katalogu
`/var/log/apt/`, jako, że `aptitutde` to nakładka na `apt`. Wiemy zatem gdzie są przechowywane logi
obu z powyższych narzędzi.

### Apt

Zajrzyjmy zatem do katalogu `/var/log/apt/` . Mamy tam z grubsza dwa pliki `history.log` oraz
`term.log` . W pierwszym są przechowywane informacje na temat wersji
aktualizowanych/instalowanych/usuwanych pakietów. Nie wiem kto zaprojektował format tego pliku ale
za wyjątkiem daty rozpoczęcia akcji jak i jej zakończenia, ten log jest zupełnie nieczytelny. Jeśli
chodzi o wersje pakietów, to dużo lepiej korzystać z logu `aptitude`, jeśli oczywiście aktualizacja
systemu jest przeprowadzana za jego pomocą.

#### Plik history.log

Tak czy inaczej, rzućmy okiem na plik `history.log` . Poniżej przykładowy wycinek z tego pliku:

    Start-Date: 2015-10-03  20:48:56
    Install: libvte-2.91-0:amd64 (0.42.0-1, automatic), kodi-data:amd64 (15.2~rc1+dfsg1-2, automatic)...
    Upgrade: libasan2:amd64 (5.2.1-18, 5.2.1-20), libgnutls-openssl27:amd64 (3.3.17-1, 3.3.18-1), libquadmath0:amd64 (5.2.1-18, 5.2.1-20)...
    End-Date: 2015-10-03  20:53:28

Dla większej czytelności, log został nieco przycięty. W każdym razie, mamy tutaj datę rozpoczęcia (
`Start-Date` ) i zakończenia ( `End-Date` ) operacji i w ten sposób jesteśmy w stanie określić kiedy
dokładnie miała miejsce przeprowadzana przez nas aktualizacja systemu. Dodatkowo, jesteśmy w stanie
podać dokładny czas jej trwania. Obie te informacje mogą mieć znaczenie przy ewentualnych
problemach.

Między znacznikami czasu, mamy informacje na temat tego jakie zmiany zostały poczynione. Wszelkie
pakiety, które są wypisane po `Upgrade:` zostały zaktualizowane ze starszych do nowszych wersji. Jak
możemy zaobserwować, w nawiasie, tuż za nazwą pakietu, mamy dwa numerki oddzielone przecinkiem.
Pierwszy z nich to numer starej wersji, zaś drugi nowej.

Czasem aktualizacja systemu pociąga za sobą konieczność instalowania/usuwania pewnych dodatkowych
zależności. W tym powyższym przykładzie, wymagane jest doinstalowanie szeregu pakietów. Informacje w
nawiasie przy nazwach pakietów odpowiadają za numerek wersji instalowanego pakietu, oraz precyzują
iż ten pakiet jest instalowany automatycznie (zależności).

Jeśli w grę wchodzi kilka pakietów, to czytanie tego logu nie stanowi aż takiego wyzwania. Problem
zaczyna się gdy tych pakietów są setki.

#### Plik term.log

Drugim z plików, które oferuje nam `apt` jest `term.log` , czyli dokładny zapis tego co się
wyświetlało w chwili gdy miała miejsce aktualizacja systemu. Rzućmy zatem na niego okiem:

    Log started: 2015-10-07  11:22:09
    (Reading database ...
    (Reading database ... 5%
    ...
    Preparing to unpack .../gimp-data_2.8.14-1.2_all.deb ...
    Unpacking gimp-data (2.8.14-1.2) over (2.8.14-1) ...
    Selecting previously unselected package libgegl-0.3-0:amd64.
    Preparing to unpack .../libgegl-0.3-0_0.3.0-5_amd64.deb ...
    Unpacking libgegl-0.3-0:amd64 (0.3.0-5) ...
    Preparing to unpack .../gimp_2.8.14-1.2_amd64.deb ...
    Unpacking gimp (2.8.14-1.2) over (2.8.14-1+b1) ...
    Processing triggers for libc-bin (2.19-22) ...
    Processing triggers for man-db (2.7.3-1) ...
    Processing triggers for hicolor-icon-theme (0.13-1) ...
    Processing triggers for mime-support (3.59) ...
    Processing triggers for desktop-file-utils (0.22-1) ...
    Processing triggers for menu (2.1.47) ...
    ...
    Removing libgegl-0.2-0:amd64 (0.2.0-7.1) ...
    ...
    Setting up gimp (2.8.14-1.2) ...
    Setting up linux-image-4.2.0-1-amd64 (4.2.3-1) ...
    /etc/kernel/postinst.d/initramfs-tools:
    update-initramfs: Generating /boot/initrd.img-4.2.0-1-amd64
    W: Possible missing firmware /lib/firmware/i915/skl_dmc_ver1.bin for module i915
    Setting up libgc1c2:amd64 (1:7.4.2-7) ...
    ...
    Log ended: 2015-10-07  11:25:50

Widok znajomy prawda? Jest to dokładnie ten sam log, który pojawia się w oknie terminala, z tym, że
jest zapisywany do pliku, który możemy sobie w dowolnym czasie przejrzeć i ewentualnie odwołać się
do niego gdy piszemy posta na forum.

### Aptitude

`aptitude` ma do zaoferowania nam tylko jeden plik: `/var/log/aptitude` , który jest odpowiednikiem
pliku `history.log` dla `apt` . Jak widzieliśmy wyżej, format pliku `history.log` nie należy do
przyjemnych. Inaczej zaś sprawa ma się w przypadku pliku logu dla `aptitude` . Wygląda on mniej
więcej tak:

    Aptitude 0.7.2: log report
    Wed, Oct  7 2015 11:21:48 +0200

    IMPORTANT: this log only lists intended actions; actions which fail due to
    dpkg problems may not be completed.

    Will install 37 packages, and remove 1 packages.
    7,111 kB of disk space will be used
    ===============================================================================
    [REMOVE, NOT USED] libgegl-0.2-0:amd64 0.2.0-7.1
    [INSTALL, DEPENDENCIES] libgegl-0.3-0:amd64 0.3.0-5
    [UPGRADE] binutils:amd64 2.25.1-4 -> 2.25.1-5
    [UPGRADE] blueman:amd64 2.0-1 -> 2.0.1-1
    ...
    [UPGRADE] pinentry-gnome3:amd64 0.9.6-1 -> 0.9.6-2
    [UPGRADE] pinentry-gtk2:amd64 0.9.6-1 -> 0.9.6-2
    [UPGRADE] uget:amd64 2.0-1 -> 2.0.2-1
    [UPGRADE] xdg-utils:amd64 1.1.0-1 -> 1.1.1-1
    ===============================================================================

    Log complete.

Jak widzimy, zamiast jednej długiej linijki, mamy jedną linijkę na pakiet, gdzie mamy określone
wersje oraz to co z danym pakietem zostało zrobione i z jakiego powodu.

Niestety `aptitude` nie dysponuje logiem z terminala ale jako, że `aptitude` jest nakładką na `apt`
, to log z akcji przeprowadzanych za pomocą `aptitude` zostanie przesłany również do pliku
`/var/log/apt/term.log` .
