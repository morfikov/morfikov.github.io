---
author: Morfik
categories:
- Linux
date: "2015-06-15T18:30:28Z"
date_gmt: 2015-06-15 16:30:28 +0200
published: true
status: publish
tags:
- smart
- hdd/ssd
title: Problematyczny parametr "Offline Uncorrectable"
---

Udało mi się znaleźć trochę informacji ma temat parametru S.M.A.R.T `198` , tj. `Offline
Uncorrectable` . Wychodzi na to, że część dysków nie resetuje go, nawet po pomyślnym przejściu testu
`offline`. Za to demon `smartd` domyślnie ma ustawione informowanie o niezerowej wartości tego
atrybutu w logu systemowym i prawdopodobnie chyba nie da się nic zrobić w tej sprawie ale można
poinstruować `smartd` by wyrzucał komunikat tylko w przypadku gdy wartość tego atrybutu zostanie
zwiększona w stosunku do wartości zapisanej przy poprzednim skanowaniu, czyli jeśli teraz mamy
wartość, np. `2`, to ostrzeżenie pojawi się gdy będzie tam widniało `3` i więcej.

<!--more-->
## Komunikat z Offline Uncorrectable

W przypadku gdy na naszym dysku twardym pojawił się pierwszy uszkodzony sektor i podjęliśmy się jego
skutecznej naprawy, to wszystkie problemy z dyskiem znikają ale z jakichś powodów S.M.A.R.T zwraca
ten poniższy
    komunikat:

    Nov 22 01:05:51 morfikownia smartd[3463]: Device: /dev/sda [SAT], 2 Offline uncorrectable sectors

By zmienić konfigurację `smartd`, edytujemy plik `/etc/smartd.conf` i dopisujemy tam poniższą
linijkę:

    /dev/sda -H -l error -l selftest -t -R 1 -R 5 -R 7 -R 10 -R 11 -R 192 -R 196 -R 197 -R 198 -R 199 -R 200 -U 198+

Koniecznie musi ona się pojawić przed wystąpieniem `DEVICESCAN`, inaczej zostanie zignorowana. W
przypadku posiadania większej ilości dysków, po prostu dodajemy kolejną linijkę odpowiednio
zmieniając plik urządzenia. Poniżej zaś jest krótkie wyjaśnienie użytych parametrów:

  - `-H` -- monitoruje S.M.A.R.T Health Status i zgłasza jeśli są problemy.
  - `-l` -- Monitoruje logi S.M.A.R.T. Są możliwe do zdefiniowania dwie opcje: `error` oraz
    `selftest` . Jeśli któryś z nich zgłosi błąd, zostaniemy o tym poinformowani.
  - `-t` -- zgłasza zmiany atrybutów w tabeli S.M.A.R.T . Jeśli nie ma parametru `-R` , monitorowane
    są wszystkie.
  - `-R` -- definiuje atrybuty do monitorowania. Np. temperatura, która u mnie ma 28 stopni nie musi
    być monitorowana, temu nie jest uwzględniony atrybut `194` . Zawsze to trochę mniej operacji
    zapisu/odczytu na dysku.
  - `-U` -- domyślnie ta opcja wskazuje na parametr `198` (można wybrać inny) i jeśli zgłoszona jest
    wartość większa od `0`, zostaje wyrzucony komunikat w syslogu. Dodanie znaku `+` do parametru
    wskaże by `smartd` traktował wartość z ostatniego skanowania jako bazową, czyli w moim przypadku
    `2` .

Zapisujemy plik i restartujemy usługę:

    # systemctl restart smartd.service
    
    smartd[26236] Device: /dev/sda [SAT], state written to /var/lib/smartmontools/smartd.WDC_WD15EARS_00MVWB0-WD_WCAZA3607921.ata.state
    smartd[26236] smartd is exiting (exit status 0)
    smartd[29402] smartd 6.2 2013-07-26 r3841 [x86_64-linux-3.12-3.slh.2-aptosid-amd64] (local build)
    smartd[29402] Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org
    smartd[29402] Opened configuration file /etc/smartd.conf
    smartd[29402] Drive: DEVICESCAN, implied '-a' Directive on line 23 of file /etc/smartd.conf
    smartd[29402] Configuration file /etc/smartd.conf was parsed, found DEVICESCAN, scanning devices
    smartd[29402] Device: /dev/sda, type changed from 'scsi' to 'sat'
    smartd[29402] Device: /dev/sda [SAT], opened
    smartd[29402] Device: /dev/sda [SAT], WDC WD15EARS-00MVWB0, S/N:WD-WCAZA3607921, WWN:5-0014ee-2b01eac3e, FW:51.0AB51, 1.50 TB
    smartd[29402] Device: /dev/sda [SAT], found in smartd database: Western Digital Caviar Green (AF)
    smartd[29402] Device: /dev/sda [SAT], is SMART capable. Adding to "monitor" list.
    smartd[29402] Device: /dev/sda [SAT], state read from /var/lib/smartmontools/smartd.WDC_WD15EARS_00MVWB0-WD_WCAZA3607921.ata.state
    smartd[29402] Device: /dev/sda, duplicate, ignored
    smartd[29402] Monitoring 1 ATA and 0 SCSI devices
    smartd[29402] Device: /dev/sda [SAT], state written to /var/lib/smartmontools/smartd.WDC_WD15EARS_00MVWB0-WD_WCAZA3607921.ata.state
    smartd[29404] smartd has fork()ed into background mode. New PID=29404.
    smartd[29404] file /var/run/smartd.pid written containing PID 29404

Jak widać nie ma już info na temat `2 Offline uncorrectable sectors` .
