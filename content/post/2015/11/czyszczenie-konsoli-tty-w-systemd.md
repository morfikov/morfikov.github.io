---
author: Morfik
categories:
- Linux
date: "2015-11-07T11:48:02Z"
date_gmt: 2015-11-07 10:48:02 +0100
published: true
status: publish
tags:
- tty
- systemd
GHissueID: 279
title: Czyszczenie konsoli TTY w systemd
---

Podczas uruchamiania się systemu, na ekranie zwykle widzimy szereg komunikatów. Informują nas one o
tym jakie usługi są aktywowane oraz czy wystąpiły jakieś błędy. Po tym jak faza boot dobiegnie
końca, wszystkie te informacje dalej są wyświetlane na konsoli TTY i możemy je podejrzeć po
przyciśnięciu klawiszy Ctrl-Alt-F1 . W pewnych sytuacjach może to stwarzać zagrożenie
bezpieczeństwa, bo są tam zawarte informacje o tym jakie usługi zostały uruchomione. Przydałoby się
zatem wyczyścić tę konsolę TTY po załadowaniu się systemu i o tym będzie poniższy wpis.

<!--more-->
## Konfiguracja TTY w pliku /etc/inittab

Konsole w debianie, do których możemy przejść za pomocą skrótu Ctrl-Alt-Fx , gdzie `x` to numer od
1-6, [są uruchamiane za sprawą getty](https://wiki.debian.org/getty). W przypadku sysvinit, wszelkie
opcje dotyczące TTY, jak i ich rodzaj, były określane w pliku `/etc/inittab` . W ten sposób mogliśmy
kontrolować zachowanie tych konsol przez dopisywanie odpowiednich parametrów w linijkach podobnych
do tej poniżej:

    1:2345:respawn:/sbin/getty 38400 --noclear tty1

Zatem jeśli chcielibyśmy czyścić pierwszą konsolę TTY po załadowaniu się systemu, to wystarczyło
usunąć opcję `--noclear` sprzed nazwy konsoli, przykładowo:

    1:2345:respawn:/sbin/getty 38400 tty1

## Czyszczenie TTY w systemd

Niestety to powyższe rozwiązanie nie zadziała w przypadku systemd, bo ten zwyczajnie nie korzysta z
pliku `inittab` . Jeśli chcemy zaprogramować ten init, by czyścił pierwszą konsolę, to będziemy
musieli nieco przerobić usługę, która konfiguruje TTY.

W debianie pliki `.service` znajdują się w `/lib/systemd/system/` , a nas będzie interesował plik
`getty@.service` . Po znaku `@` wpisujemy nazwę konsoli, np. `tty1` . Nie będziemy oczywiście w
żaden sposób zmieniać nazwy tego pliku, czy też jego zawartości, bo tak wprowadzone zmiany by
zwyczajnie zostały nadpisane przy aktualizacji pakietu. Zamiast tego, przy pomocy poniższego
polecenia:

    # systemctl edit getty@.service

Edytujmy usługę `getty@.service` i dodajmy ten poniższy kod:

    [Service]
    ExecStart=
    ExecStart=-/sbin/agetty %I $TERM

W ten sposób zostanie utworzony plik `/etc/systemd/system/getty@.service.d/override.conf` , który
wyczyści poprzednie polecenie zawierające parametr `--noclear` i zastąpi go tym powyższym, które nie
zawiera już tej opcji. Nic więcej nie musimy robić.
