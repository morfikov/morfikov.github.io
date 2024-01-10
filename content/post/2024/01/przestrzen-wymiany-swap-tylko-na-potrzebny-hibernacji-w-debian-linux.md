---
author: Morfik
categories:
- Linux
date:    2024-01-10 20:05:00 +0100
lastmod: 2024-01-10 20:05:00 +0100
published: true
status: publish
tags:
- debian
- ssd
- systemd
- swap
GHissueID: 600
title: Przestrzeń wymiany SWAP tylko na potrzebny hibernacji w Debian linux
---

W obecnych czasach, rola przestrzeni wymiany w zasadzie ograniczyła się z grubsza do umożliwienia
nam przeprowadzenia procesu hibernacji. Inne aspekty pracy systemu, w których SWAP brał udział
zostały albo zepchnięte na margines (bardzo stare maszyny z niewielką ilością RAM), albo zupełnie
wyeliminowane za sprawą nowszych technologi lub/i posiadania większej ilości pamięci operacyjnej w
takich systemach. Obecnie domowy komputer z 32 GiB pamięci RAM to nic niezwykłego i przestrzeń
wymiany na takich maszynach nie jest już nam w zasadzie do niczego potrzebna, bo raczej jest mało
prawdopodobne, że nam tej pamięci zabraknie przy codziennym użytkowaniu naszego linux'a. Niemniej
jednak, hibernacja systemu jest czymś pozytywnym z perspektywy użytkownika, bo umożliwia mu
zapisanie stanu pracy przed wyłączeniem komputera i odtworzenie tego stanu po ponownym uruchomieniu
systemu. Jeśli chcielibyśmy się cieszyć urokami hibernacji, to bez konfiguracji SWAP'a się nie
obejdzie. Problem jednak zaczyna się, gdy mamy dysk SSD i taka przestrzeń wymiany zostanie na nim
skonfigurowana. Chodzi generalnie o pewne aplikacje, które są w stanie generować bardzo dużo cache
w pamięci operacyjnej, czego efektem będzie przenoszenie danych z RAM do SWAP, a to może bardzo
szybko wykończyć taki nośnik flash. Dlatego też przydałoby się nieco zabezpieczyć przestrzeń
wymiany przed tego typu sytuacjami, tak by można było jej używać jedynie w przypadku hibernacji.

<!--more-->
## Requested hibernate operation is not supported...

Technicznie rzecz biorąc, nic nie stoi na przeszkodzie, by ręcznie aktywować sobie przestrzeń
wymiany ilekroć tylko chcemy zahibernować system. To rozwiązanie ma jednak kilka wad, bo po starcie
systemu czy jego odhibernowaniu, przestrzeń wymiany dalej będzie aktywna i trzeba będzie ją
dezaktywować ręcznie. Poza tym, w pośpiechu możemy zapomnieć włączyć SWAP i możemy oddalić się od
naszego linux'a nie będąc świadomymi, że nie został on zahibernowany, a jedynie uśpiony.

Spójrzmy co się stanie jeśli zażądamy hibernacji od systemu ale nie będzie w nim skonfigurowanej
przestrzeni wymiany. W logu systemowy zostanie wydrukowany poniższy komunikat tuż po przyciśnięciu
przycisku zasilania:

    ...
    systemd-logind[245006]: Power key pressed short.
    systemd-logind[245006]: Requested hibernate operation is not supported, using regular suspend instead.
    systemd-logind[245006]: Suspending...
    systemd[1]: Reached target sleep.target - Sleep.
    systemd[1]: Starting systemd-suspend.service - System Suspend...
    systemd-sleep[247002]: Performing sleep operation 'suspend'...
    ...

Widzimy tutaj wiadomość `Requested hibernate operation is not supported` , czyli w skrócie
hibernacja nie jest wspierana, bo przestrzeń wymiany nie została aktywowana. Dlatego właśnie
systemd odmawia zahibernowania systemu i zamiast hibernacji aktywuje regularny stan uśpienia.
Podobnie sprawa będzie wyglądać, gdy rozmiar SWAP będzie za mały, by pomieścić dane zgromadzone w
RAM -- w takim przypadku system również odmówi hibernacji.

### Wpis od SWAP w pliku /etc/fstab

Taka ręczna aktywacja/dezaktywacja przestrzeni wymiany jest dość prymitywna i trochę pozbawiona
sensu. Na szczęście nie musimy korzystać z takiego rozwiązania (czy też próbować go zmodernizować
skryptami), bo wygląda na to, że [systemd przygotował się na tego typu okoliczność][1] aktywacji
przestrzeni wymiany SWAP tuż przed zahibernowaniem systemu i dezaktywacji jej chwilę po
odhibernowaniu. Nie ma tutaj znaczenia fakt czy ta przestrzeń wymiany będzie znajdować się na
osobnej partycji, czy [w dedykowanym pliku][2] gdzieś w obrębie jakiegoś systemu plików.

Zakładam, że dysponujemy już stosownym plikiem albo partycją SWAP oraz, że mieliśmy wcześniej
skonfigurowany poprawnie działający proces hibernacji. W takim systemie zapewne mamy odpowiedni
wpis w pliku `/etc/fstab` odnoszący się do przestrzeni wymiany. Poniżej przykład:

    # swapon -s
    Filename        Type            Size            Used            Priority
    /dev/dm-11      partition       6291452         0               -2

    # ls -al /dev/disk/by-uuid/ | grep dm-11
    lrwxrwxrwx 1 root root  11 2024-01-09 20:48:47 063c13f4-39b8-4965-bafd-7edb8b078774 -> ../../dm-11

    # cat /etc/fstab | grep swap
    UUID=063c13f4-39b8-4965-bafd-7edb8b078774  swap  swap    defaults,pri=1024 0 0

Ten powyższy wpis z numerkiem UUID trzeba wykomentować (albo usunąć). Dobrze jest też wcześniej
dezaktywować tego SWAP'a:

    # swapoff /dev/dm-11

Ten zabieg ma na celu powstrzymać system od automatycznej aktywacji przestrzeni wymiany, np.
podczas rozruchu komputera.

Ważne jest tutaj by nie ruszać w żaden sposób pliku `/etc/initramfs-tools/conf.d/resume` , tj.
powinien on zawierać taki sam wpis co zawsze, przykładowo:

    # cat /etc/initramfs-tools/conf.d/resume

    RESUME=/dev/mapper/wd_blue_label-swap

## Zmienna SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK

Drugim krokiem jest edycja pliku usługi `systemd-logind.service` przez określenie w nim zmiennej
`SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1` . Zadaniem tej zmiennej jest obejście mechanizmu
weryfikacji czy plik/partycja SWAP jest zamontowana oraz czy przestrzeń wymiany dysponuje wymaganą
ilością miejsca, by proces hibernacji został przeprowadzony z powodzeniem.

Edytujemy zatem usługę `systemd-logind.service` i dodajemy w niej ten poniższy wpis:

    # systemctl edit systemd-logind.service

    [Service]
    Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1

Po ustawieniu zmiennej trzeba zrestartować usługę `systemd-logind.service` . Niestety w tym celu
trzeba albo wyłączyć środowisko graficzne i z poziomu TTY ją zrestartować, albo trzeba uruchomić
ponownie system. Tak czy inaczej, po ponownym uruchomieniu tej usługi, sprawdzamy czy zmienna
`SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK` została poprawnie ustawiona w środowisku tej usługi:

    # systemctl show systemd-logind.service --property=Environment
    Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1

Można też przez plik `environ` :

    # cat /proc/$(pidof systemd-logind)/environ
    LANG=en_US.UTF-8\
    LANGUAGE=en_US.UTF-8\
    LC_ADDRESS=en_US.UTF-8\
    LC_COLLATE=C\
    LC_CTYPE=en_US.UTF-8\
    LC_IDENTIFICATION=en_US.UTF-8\
    LC_MEASUREMENT=en_US.UTF-8\
    LC_MESSAGES=C\
    LC_MONETARY=en_US.UTF-8\
    LC_NAME=en_US.UTF-8\
    LC_NUMERIC=en_US.UTF-8\
    LC_PAPER=en_US.UTF-8\
    LC_TELEPHONE=en_US.UTF-8\
    LC_TIME=en_DK.UTF-8\
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin\
    NOTIFY_SOCKET=/run/systemd/notify\
    FDSTORE=512\
    WATCHDOG_PID=256314\
    WATCHDOG_USEC=180000000\
    USER=root\
    INVOCATION_ID=fc9f052a550241818f5df41325833f40\
    JOURNAL_STREAM=7:2575363\
    RUNTIME_DIRECTORY=/run/systemd/inhibit:/run/systemd/seats:/run/systemd/sessions:/run/systemd/shutdown:/run/systemd/users\
    STATE_DIRECTORY=/var/lib/systemd/linger\
    SYSTEMD_EXEC_PID=256314\
    SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1

## Usługa systemd zarządzająca przestrzenią wymiany SWAP

Pozostało nam jeszcze napisanie usługi dla systemd, która aktywuje przestrzeń wymiany tuż przed
procesem hibernacji i dezaktywuje ją tuż po tym procesie. Poniższą zawartość zapisujemy, np. w
pliku `/etc/systemd/system/swap-to-hibernate.service` :

    [Unit]
    Description=Mount and unmount swap file when hibernating or resuming
    DefaultDependencies=no
    Before=systemd-hibernate.service systemd-hybrid-sleep.service
    StopWhenUnneeded=yes
    RefuseManualStart=yes
    RefuseManualStop=yes

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/usr/sbin/swapon /dev/mapper/wd_blue_label-swap
    ExecStop=/usr/sbin/swapoff /dev/mapper/wd_blue_label-swap
    TimeoutStopSec=infinity

    [Install]
    RequiredBy=systemd-hibernate.service systemd-hybrid-sleep.service

Pozostaje nam już tylko włączyć tę usługę:

    # systemctl enable swap-to-hibernate.service

## Test SWAP przy hibernacji

Pora przetestować czy mechanizm aktywacji/dezaktywacji przestrzeni wymiany SWAP działa prawidłowo.
Przyciskamy zatem przycisk power na obudowie komputera i obserwujemy jego zachowanie. Maszyna
powinna się po chwili wyłączyć. Przyciskamy jeszcze raz przycisk power i czekamy na załadowanie się
systemu. Po uruchomieniu się linux'a patrzymy w log.

W tym przypadku możemy zaobserwować poniższe komunikaty:

    systemd-logind[256314]: Power key pressed short.
    systemd-logind[256314]: Hibernating...
    systemd[1]: Created slice system-resume.slice - Slice /system/resume.
    systemd[1]: Created slice system-suspend.slice - Slice /system/suspend.
    systemd[1]: Reached target sleep.target - Sleep.
    systemd[1]: Starting swap-to-hibernate.service - Mount and unmount swap file when hibernating or resuming...
    kernel: Adding 6291452k swap on /dev/mapper/wd_blue_label-swap.  Priority:-2 extents:1 across:6291452k
    systemd[1]: Finished swap-to-hibernate.service - Mount and unmount swap file when hibernating or resuming.
    systemd[1]: Starting systemd-hibernate.service - Hibernate...
    systemd-sleep[259220]: Performing sleep operation 'hibernate'...
    kernel: PM: hibernation: hibernation entry.
    ...
    kernel: PM: hibernation: Creating image:
    kernel: PM: hibernation: Need to copy 1497968 pages
    kernel: PM: hibernation: Normal pages needed: 1497968 + 1024, available pages: 2632828

Widzimy tutaj uruchomienie naszej usługi `Starting swap-to-hibernate.service` , która aktywowała
przestrzeń wymiany `/dev/mapper/wd_blue_label-swap` , w celu zapisania utworzonego obrazu pamięci.
W tym miejscu maszyna się wyłącza.

Po uruchomieniu komputera z kolei mamy poniższe komunikaty:

    kernel: PM: hibernation: free pages cleared after restore
    kernel: ACPI: PM: Waking up from system sleep state S4
    ...
    kernel: PM: hibernation: Basic memory bitmaps freed
    ...
    systemd-sleep[259220]: System returned from sleep operation 'hibernate'.
    kernel: PM: hibernation: hibernation exit
    ...
    systemd[1]: systemd-hibernate.service: Deactivated successfully.
    systemd[1]: Finished systemd-hibernate.service - Hibernate.
    systemd[1]: Reached target hibernate.target - System Hibernation.
    systemd[1]: Stopped target sleep.target - Sleep.
    ...
    systemd-logind[256314]: Operation 'sleep' finished.
    ...
    systemd[1]: Stopping swap-to-hibernate.service - Mount and unmount swap file when hibernating or resuming...
    systemd[1]: Stopped target hibernate.target - System Hibernation.
    ...
    systemd[1]: dev-disk-by\x2duuid-063c13f4\x2d39b8\x2d4965\x2dbafd\x2d7edb8b078774.swap: Deactivated successfully.
    systemd[1]: dev-mapper-wd_blue_label\x2dswap.swap: Deactivated successfully.
    systemd[1]: dev-disk-by\x2did-dm\x2dname\x2dwd_blue_label\x2dswap.swap: Deactivated successfully.
    systemd[1]: dev-disk-by\x2did-dm\x2duuid\x2dLVM\x2dVUnBhi38ApbucSpgcKlw17d96ONzmqBXVnCLxsWUBT5eHhqNX6zD09SI2VeZdQYk.swap: Deactivated successfully.
    systemd[1]: dev-wd_blue_label-swap.swap: Deactivated successfully.
    systemd[1]: dev-dm\x2d11.swap: Deactivated successfully.
    systemd[1]: swap-to-hibernate.service: Deactivated successfully.
    systemd[1]: Stopped swap-to-hibernate.service - Mount and unmount swap file when hibernating or resuming.

Tutaj widzimy zatrzymanie naszej usługi `Stopping swap-to-hibernate.service` , czego efektem jest
dezaktywacja przestrzeni wymiany `/dev/mapper/wd_blue_label-swap` . Zatem cały mechanizm działa
prawidłowo.

## Podsumowanie

Jeśli jesteśmy świeżo upieczonymi posiadaczami dysku SSD i nie chcemy rezygnować z mechanizmu
hibernacji, bo dość bardzo cenimy sobie jej zalety, to możemy na takim nośniku flash skonfigurować
sobie przestrzeń wymiany tylko i wyłączenie na potrzeby procesu hibernacji. Tego typu zabieg
uchroni nas przed zbędnym zapisem pliku/partycji SWAP podczas regularnej pracy systemu i odciąży
przy tym dość znacznie dysk SSD, co powinno przełożyć się na zwiększoną żywotność jego komórek
flash.


[1]: https://github.com/systemd/systemd/pull/3064
[2]: /post/przestrzen-wymiany-swap-jako-plik/
