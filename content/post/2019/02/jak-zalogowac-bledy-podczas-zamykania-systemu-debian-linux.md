---
author: Morfik
categories:
- Linux
date: "2019-02-02T06:12:18Z"
published: true
status: publish
tags:
- debian
- systemd
- logi
GHissueID: 306
title: Jak zalogować błędy podczas zamykania systemu Debian Linux
---

By wyłączyć komputer, jego system operacyjny musi pierw zatrzymać (lub też ubić siłowo) wszystkie
działające usługi za wyjątkiem tego mającego PID z numerkiem 1. Zwykle proces zamykania się systemu
Debian Linux nie trwa więcej niż parę sekund ale czasami pojawiają się dziwne problemy, które mogą
to zadanie utrudnić lub też całkowicie uniemożliwić. Nawet jeśli system będzie się w stanie
zresetować, to zanim to nastąpi, to na konsoli mogą pojawić się komunikaty mogące pomóc nam w
zdiagnozowaniu dolegliwości, która doskwiera naszej maszynie. Problem w tym, że część tych
wiadomości nie zostanie nigdy zalogowana do pliku, gdzie moglibyśmy ich poszukać. Dzieje się tak
dlatego, że w pewnym określonym momencie zamykania się systemu trzeba wyłączyć usługę logowania, co
zwykle widać w logu jako `systemd-journald[]: Journal stopped` . Gdy dziennik zostanie zatrzymany,
żadna wiadomość, która od tego momentu pojawi się na ekranie, nie zostanie już zalogowana do pliku.
Jeśli teraz pojawią nam się ostrzeżenia lub błędy, a po chwili komputer się zresetuje, to możemy
mieć nie lada problem z ustaleniem przyczyny mimo, że system nam ją zgłasza. Przydałoby się zapisać
te komunikaty, tylko jak to zrobić, skoro usługa logowania jest już nieaktywna?

<!--more-->
## Jak Linux wyłącza komputer

Z chwilą, gdy w działającym Debianie wpiszemy na konsoli `reboot`/`poweroff` , systemd zaczyna
zamykać usługi (zwykle w kolejności odwrotnej niż zostały one uruchomione). Gdy wszystko idzie
według planu, to spora część procesów zostaje zatrzymana zgodnie z określonymi w plikach usług
wytycznymi. Gdy procesy są już zatrzymane, to synchronizowane są bufory dysków, a po chwili
następuje próba odmontowania wszystkich systemów plików, które do tej pory jeszcze nie zostały
odmontowane. Warto tutaj zaznaczyć, że system plików, który jest zamontowany w miejsce `/` , nie
zostanie nigdy odmontowany z racji, że działają na nim procesy, min. ten mający PID 1, a system
plików może być odmontowany jedynie w sytuacji, gdy nie jest wykorzystywany przez żaden proces.
Systemd doskonale zdaje sobie z tego faktu sprawę i dlatego tuż przed zresetowaniem/wyłączeniem
maszyny, montuje `/` w trybie `ro` , czyli tylko do odczytu. Zanim to jednak nastąpi, to do
każdego procesu (za wyjątkiem PID1), który jeszcze działa w systemie, są wysyłane sygnały
`SIGTERM` oraz `SIGKILL` W ten sposób pod koniec fazy zamykania systemu działa nam w zasadzie
tylko jeden proces i mamy podmontowany z reguły jeden system plików w trybie tylko do odczytu.

Poniżej jest końcówka logu z zamykania systemu:

    $  journalctl -b-1
    ...
    systemd[1]: Reached target Unmount All Filesystems.
    systemd[1]: Reached target Final Step.
    systemd[1]: systemd-reboot.service: Succeeded.
    systemd[1]: Started Reboot.
    systemd[1]: Reached target Reboot.
    systemd[1]: Shutting down.
    systemd[1]: Hardware watchdog 'iTCO_wdt', version 0
    systemd[1]: Set hardware watchdog to 10min.
    kernel: watchdog: watchdog0: watchdog did not stop!
    systemd-shutdown[1]: Syncing filesystems and block devices.
    systemd-shutdown[1]: Sending SIGTERM to remaining processes...
    systemd-journald[396]: Journal stopped

Z komunikatu wyżej widać, że usługa logowania `systemd-journald` została ubita po tym jak
`systemd-shutdown` posłał jej sygnał `SIGTERM` umożliwiając jej tym samym zapisanie danych na dysk.
System jednak działa dalej, choć nie wyczytamy tego z powyższego logu.

## Logi kernela (dmesg) oraz logi systemd

Może i już nie jesteśmy w stanie zalogować komunikatów do pliku, bo stosowna usługa umożliwiająca
nam to nie działa ale kernel posiada swój własny mechanizm logowania, tj. `dmesg` . Dane w `dmesg`
są przechowywane w buforze zlokalizowanym w pamięci RAM, którego rozmiar można dostosować przez
dopisanie do linijki kernela w bootloaderze parametru `log_buf_len` , np. `log_buf_len=24M` .
Domyślnie jednak, w tym buforze znajdą się jedynie logi kernela, przez co niewiele się z nich
dowiemy, gdy problem będzie tkwił w jakiejś usłudze systemowej. By zalogować nie tylko komunikaty
kernela ale również i te pochodzące od systemd (oraz jego usług), trzeba również użyć parametru
`systemd.log_target=kmsg` . Jeśli w dalszym ciągu logi nic nam nie powiedzą, to możemy przełączyć
systemd w tryb debugowania za pomocą parametru `systemd.log_level=debug` , z tym, że tutaj
przydałoby się również wyłączyć ratelimit logów przez określenie `printk.devkmsg=on` . Zatem do
linijki kernela w bootloaderze powinniśmy dodać to poniższe:

    APPEND root=... systemd.log_level=debug systemd.log_target=kmsg log_buf_len=24M printk.devkmsg=on ...

Poniżej znajduje się przykład tego, co zostaje zalogowane od momentu, gdy usługa `systemd-journald`
zostaje zatrzymana:

    [97745.444190] systemd-shutdown[1]: Sending SIGTERM to remaining processes...
    [97745.451446] systemd-journald[482]: Received SIGTERM from PID 1 (systemd-shutdow).
    [97745.649032] systemd-shutdown[1]: Sending SIGKILL to remaining processes...
    [97745.656809] systemd-shutdown[1]: Hardware watchdog 'iTCO_wdt', version 0
    [97745.659726] systemd-shutdown[1]: Unmounting file systems.
    [97745.663229] [47842]: Remounting '/' read-only in with options 'errors=remount-ro'.
    [97745.755769] EXT4-fs (dm-4): re-mounted. Opts: errors=remount-ro
    [97745.791525] systemd-shutdown[1]: All filesystems unmounted.
    [97745.794368] systemd-shutdown[1]: Deactivating swaps.
    [97745.796662] systemd-shutdown[1]: All swaps deactivated.
    [97745.798859] systemd-shutdown[1]: Detaching loop devices.
    [97745.806223] systemd-shutdown[1]: All loop devices detached.
    [97745.807701] systemd-shutdown[1]: Detaching DM devices.
    [97745.812357] systemd-shutdown[1]: Detaching DM 253:9.
    [97745.833097] systemd-shutdown[1]: Detaching DM 253:8.
    [97745.835815] systemd-shutdown[1]: Could not detach DM /dev/dm-8: Device or resource busy
    [97745.838396] systemd-shutdown[1]: Detaching DM 253:7.
    [97745.855184] systemd-shutdown[1]: Detaching DM 253:6.
    [97745.878200] systemd-shutdown[1]: Detaching DM 253:5.
    [97745.896543] systemd-shutdown[1]: Detaching DM 253:3.
    [97745.908330] systemd-shutdown[1]: Detaching DM 253:2.
    [97745.926478] systemd-shutdown[1]: Detaching DM 253:12.
    [97745.947184] systemd-shutdown[1]: Detaching DM 253:11.
    [97745.964219] systemd-shutdown[1]: Detaching DM 253:10.
    [97745.977158] systemd-shutdown[1]: Detaching DM 253:1.
    [97745.989103] systemd-shutdown[1]: Detaching DM 253:0.
    [97745.991164] systemd-shutdown[1]: Could not detach DM /dev/dm-0: Device or resource busy
    [97745.993165] systemd-shutdown[1]: Not all DM devices detached, 3 left.
    [97745.995581] systemd-shutdown[1]: Detaching DM devices.
    [97745.998183] systemd-shutdown[1]: Detaching DM 253:8.
    [97746.012280] systemd-shutdown[1]: Detaching DM 253:0.
    [97746.013710] systemd-shutdown[1]: Could not detach DM /dev/dm-0: Device or resource busy
    [97746.015103] systemd-shutdown[1]: Not all DM devices detached, 2 left.
    [97746.016793] systemd-shutdown[1]: Detaching DM devices.
    [97746.019221] systemd-shutdown[1]: Detaching DM 253:0.
    [97746.020676] systemd-shutdown[1]: Could not detach DM /dev/dm-0: Device or resource busy
    [97746.022143] systemd-shutdown[1]: Not all DM devices detached, 2 left.
    [97746.023867] systemd-shutdown[1]: Detaching DM devices.
    [97746.026179] systemd-shutdown[1]: Detaching DM 253:0.
    [97746.027658] systemd-shutdown[1]: Could not detach DM /dev/dm-0: Device or resource busy
    [97746.028776] systemd-shutdown[1]: Not all DM devices detached, 2 left.
    [97746.029878] systemd-shutdown[1]: Cannot finalize remaining DM devices, continuing.
    [97746.205732] EXT4-fs (dm-4): re-mounted. Opts: errors=remount-ro

Tych komunikatów naturalnie może być więcej, choć jak widać, to w zasadzie tylko `systemd-shutdown`
i kernel na tym etapie zamykania systemu są jeszcze w stanie coś powiedzieć.

## Jak zrzucić dmesg przy zamykaniu systemu do pliku

By te powyższe wiadomości zapisać do pliku, potrzebny jest
nam [krótki skrypt](https://freedesktop.org/wiki/Software/systemd/Debugging/), który zamontuje nam
na chwilę partycję `/` w trybie do zapisu ( `rw` ), zrzuci bufor `dmesg` do pliku (przykładowo
`/shutdown-log.txt` ), zsynchronizuje bufor dysku i przemontuje z powrotem system plików w tryb
tylko do odczytu ( `ro` ). Poniżej jest taki skrypt:

    #!/bin/sh
    mount -o remount,rw /
    dmesg > /shutdown-log.txt
    sync
    mount -o remount,ro /

Ten skrypt trzeba zapisać w katalogu `/lib/systemd/system-shutdown/` i nadać mu prawa do
wykonywania:

    # chmod +x /lib/systemd/system-shutdown/debug.sh

Teraz już wystarczy zrestartować system i po ponownym jego uruchomieniu zajrzeć w plik
`/shutdown-log.txt` . Warto tutaj zaznaczyć, że w tym pliku znajdą się również logi z
wcześniejszego etapu pracy systemu i trzeba w nich odszukać moment, w którym proces wyłączania
maszyny zostaje zainicjowany ale to zadanie nie powinno już sprawić problemu.
