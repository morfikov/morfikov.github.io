---
author: Morfik
categories:
- Linux
date: "2019-04-26T21:10:15Z"
published: true
status: publish
tags:
- debian
- udisks
- policykit
- hdd
- ssd
- usb
title: Montowanie dysków jako zwykły użytkownik z UDisks i PolicyKit
---

Jeszcze nie tak dawno temu przeciętnej klasy desktop był wyposażony w pojedynczy i do tego
niewielkiej pojemności dysk twardy, który był w stanie pomieścić wszystkie pliki swojego
właściciela. Obecnie jednak większość maszyn ma tych nośników już kilka. Mowa tutaj nie tylko o
dyskach systemowych, które są fizycznie na stałe zamontowane w komputerze ale również o tych
wszystkich urządzeniach, które można podłączyć do portu USB. Polityka linux'a wymusza, by wszystkie
nośniki pamięci masowej (HDD, SSD, pendrive czy też karty SD) były montowane w systemie jedynie
przez użytkowników posiadających uprawnienia administratora. Domyślnie taki przywilej ma jedynie
root. Zatem by uzyskać dostęp do danych na takim zewnętrznym nośniku musimy logować się na root'a.
Jakby nie patrzeć ma to swoje plusy patrząc z perspektywy bezpieczeństwa, niemniej jednak czy
naprawdę potrzebny nam jest root do wgrania czegoś na nasz ulubiony pendrive? Widać nie tylko ja
zadawałem sobie takie pytanie i ktoś postanowił stworzyć narzędzie UDisks (lub jego nowszą wersję
UDisks2), które za pomocą mechanizmu PolicyKit (zwanym też PolKit) jest w stanie nadać stosowne
uprawnienia konkretnym użytkownikom systemu, przez co można określić zespół akcji, które będą oni w
stanie przeprowadzić bez potrzeby podawania hasła, np. montowanie czy odmontowanie zasobu.
Postanowiłem zatem zobaczyć jak ten duet sobie radzi na moim Debianie przy tradycyjnym użytkowaniu
systemu i ocenić jego stopień przydatności.

<!--more-->
## UDisks i UDisks2

Lata temu, gdy przenośna forma pamięci masowej zyskiwała na popularności i przy okazji rosło coraz
bardziej niezadowolenie użytkowników z racji niedogodności wynikających z dodatkowej konfiguracji
jaką trzeba było poczynić, by te zewnętrzne nośniki można było jakoś po ludzku montować w linux'ie,
powstało narzędzie UDisks. W późniejszym czasie powstała jego druga wersja zwana
obecnie [UDisks2][1], z tym, że w dalszym ciągu referuje się do tego narzędzia jako UDisks, co może
być nieco mylące. Z Debiana pakiet [udisks wyleciał dnia 2015-06-22][2]. Od tak już 2012 roku
rozpoczęła się migracja na [udisks2][3] i obecnie od lat się używa już tylko tej drugiej wersji.

## PolicyKit 0.115 vs. 0.105

Aktualnie w Debianie jest jakiś większy problem dotyczący PolicyKit. Chodzi generalnie o to, że od
lat jest już nowsza wersja pakietu `policykit-1` (tj. `0.106+` ) ale w gałęzi niestabilnej w
dalszym ciągu jest wersja `0.105` . Do końca nie mam pojęcia czemu deweloperzy Debiana nie chcą
wpuścić tej nowszej wersji (aktualnie `0.115` ) do głównego repozytorium. Być może przez fakt, że
składnia reguł nie jest wstecznie kompatybilna z tą, która była w poprzedniej wersji PolKit'a i są
obawy o popsucie ludziom ich systemów. Niemniej jednak, różnica wersji niesie ze sobą pewne
konsekwencje. Mianowicie, niektórych rzeczy na starszej wersji PolicyKit nie da rady wykonać.
Dlatego też potrzebna jest ta nowsza wersja, która obecnie od lat figuruje w repozytorium
experimental.

Ten artykuł zakłada wykorzystanie właśnie tej nowszej wersji PolicyKit i jeśli chcemy by nam
poniższa składnia reguł zadziałała, to musimy sobie zaktualizować pakiety PolKit'a do wersji z
experimental. W tym celu dodajemy do pliku `/etc/apt/sources.list` poniższy wpis:

      deb     https://deb.debian.org/debian/ experimental main contrib non-free
    # deb-src https://deb.debian.org/debian/ experimental main contrib non-free

Oraz tworzymy plik `/etc/apt/preferences` i wrzucamy do niego poniższy kod:

    Package: libpolkit-* policykit-*
    Pin: release o=Debian,a=experimental
    Pin-Priority: 995

    Package: *
    Pin: origin deb.debian.org
    Pin: release o=Debian,a=experimental
    Pin-Priority: 130

    Package: *
    Pin: origin deb.debian.org
    Pin: release o=Debian,a=unstable
    Pin-Priority: 990

Więcej na temat [konfiguracji pinning'u APT][4] można znaleźć tutaj.

## Agent autoryzacyjny PolicyKit

PolicyKit wymaga agenta, który będzie pośredniczył między użytkownikiem, a demonem PolKit'a. Z
reguły każde środowisko graficzne ma swojego własnego agenta, np. KDE ma `polkit-kde-agent-1` . Nie
wszystkie środowiska, np. te oparte o Openbox, mają stosownego agenta i jeśli korzystamy z takiego
środowiska, to trzeba rozejrzeć się za jakimś agentem PolicyKit i go sobie doinstalować we własnym
zakresie. W przeciwnym razie okienka z prośbą o autoryzację nie będą pokazywane.

## Akcje UDisks dla PolicyKit

UDisks nie jest jedynym narzędziem, które potrafi współpracować z PolicyKit. Jest ich dość sporo i
w zasadzie każda aplikacja może robić użytek z PolKit'a jeśli ich deweloperzy się postarali o to. W
przypadku UDisks mamy [sporo akcji][5], których przeprowadzenie można indywidualnie przypisać
poszczególnym użytkownikom w systemie. Wszystkie akcje wraz z ich opisami można także wyciągnąć z
pliku, który znajduje się w katalogu `/usr/share/polkit-1/actions/` :

    $ cat /usr/share/polkit-1/actions/org.freedesktop.UDisks2.policy | grep \<action -A1
      <action id="org.freedesktop.udisks2.filesystem-mount">
        <description>Mount a filesystem</description>
    --
      <action id="org.freedesktop.udisks2.filesystem-mount-system">
        <description>Mount a filesystem on a system device</description>
    --
      <action id="org.freedesktop.udisks2.filesystem-mount-other-seat">
        <description>Mount a filesystem from a device plugged into another seat</description>
    --
      <action id="org.freedesktop.udisks2.filesystem-fstab">
        <description>Mount/unmount filesystems defined in the fstab file with the x-udisks-auth option</description>
    --
      <action id="org.freedesktop.udisks2.filesystem-unmount-others">
        <description>Unmount a device mounted by another user</description>
    --
      <action id="org.freedesktop.udisks2.filesystem-take-ownership">
        <description>Take ownership of a filesystem</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-unlock">
        <description>Unlock an encrypted device</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-unlock-system">
        <description>Unlock an encrypted system device</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-unlock-other-seat">
        <description>Unlock an encrypted device plugged into another seat</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-unlock-crypttab">
        <description>Unlock an encrypted device specified in the crypttab file with the x-udisks-auth option</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-lock-others">
        <description>Lock an encrypted device unlocked by another user</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-change-passphrase">
        <description>Change passphrase for an encrypted device</description>
    --
      <action id="org.freedesktop.udisks2.encrypted-change-passphrase-system">
        <description>Change passphrase for an encrypted device</description>
    --
      <action id="org.freedesktop.udisks2.loop-setup">
        <description>Manage loop devices</description>
    --
      <action id="org.freedesktop.udisks2.loop-delete-others">
        <description>Delete loop devices</description>
    --
      <action id="org.freedesktop.udisks2.loop-modify-others">
        <description>Modify loop devices</description>
    --
      <action id="org.freedesktop.udisks2.manage-swapspace">
        <description>Manage swapspace</description>
    --
      <action id="org.freedesktop.udisks2.manage-md-raid">
        <description>Manage RAID arrays</description>
    --
      <action id="org.freedesktop.udisks2.power-off-drive">
        <description>Power off drive</description>
    --
      <action id="org.freedesktop.udisks2.power-off-drive-system">
        <description>Power off a system drive</description>
    --
      <action id="org.freedesktop.udisks2.power-off-drive-other-seat">
        <description>Power off a drive attached to another seat</description>
    --
      <action id="org.freedesktop.udisks2.eject-media">
        <description>Eject media</description>
    --
      <action id="org.freedesktop.udisks2.eject-media-system">
        <description>Eject media from a system drive</description>
    --
      <action id="org.freedesktop.udisks2.eject-media-other-seat">
        <description>Eject media from a drive attached to another seat</description>
    --
      <action id="org.freedesktop.udisks2.modify-device">
        <description>Modify a device</description>
    --
      <action id="org.freedesktop.udisks2.modify-device-system">
        <description>Modify a system device</description>
    --
      <action id="org.freedesktop.udisks2.modify-device-other-seat">
        <description>Modify a device</description>
    --
      <action id="org.freedesktop.udisks2.rescan">
        <description>Rescan a device</description>
    --
      <action id="org.freedesktop.udisks2.open-device">
        <description>Open a device</description>
    --
      <action id="org.freedesktop.udisks2.open-device-system">
        <description>Open a system device</description>
    --
      <action id="org.freedesktop.udisks2.modify-system-configuration">
        <description>Modify system-wide configuration</description>
    --
      <action id="org.freedesktop.udisks2.read-system-configuration-secrets">
        <description>Modify system-wide configuration</description>
    --
      <action id="org.freedesktop.udisks2.modify-drive-settings">
        <description>Modify drive settings</description>
    --
      <action id="org.freedesktop.udisks2.ata-smart-update">
        <description>Update SMART data</description>
    --
      <action id="org.freedesktop.udisks2.ata-smart-simulate">
        <description>Set SMART data from blob</description>
    --
      <action id="org.freedesktop.udisks2.ata-smart-selftest">
        <description>Run SMART self-test</description>
    --
      <action id="org.freedesktop.udisks2.ata-smart-enable-disable">
        <description>Enable/Disable SMART</description>
    --
      <action id="org.freedesktop.udisks2.ata-check-power">
        <description>Check power state</description>
    --
      <action id="org.freedesktop.udisks2.ata-standby">
        <description>Send standby command</description>
    --
      <action id="org.freedesktop.udisks2.ata-standby-system">
        <description>Send standby command to a system drive</description>
    --
      <action id="org.freedesktop.udisks2.ata-standby-other-seat">
        <description>Send standby command to drive on other seat</description>
    --
      <action id="org.freedesktop.udisks2.ata-secure-erase">
        <description>Securely erase a hard disk</description>
    --
      <action id="org.freedesktop.udisks2.cancel-job">
        <description>Cancel job</description>
    --
      <action id="org.freedesktop.udisks2.cancel-job-other-user">
        <description>Cancel job started by another user</description>

### Domyślne ustawienia autoryzacji akcji

Każda akcja ma domyślne ustawienia autoryzacji akcji. Poniżej przykład autoryzacji dla jednej z
akcji UDisks, a konkretnie dla `org.freedesktop.udisks2.filesystem-mount` :

    <defaults>
      <allow_inactive>auth_admin</allow_inactive>
      <allow_active>yes</allow_active>
      <allow_any>auth_admin</allow_any>
    </defaults>

Poniżej wyjaśnienie opcji:

- `allow_inactive` -- aplikuje się do lokalnych klientów, którzy mają nieaktywne sesje, np. mają
zablokowany ekran.
- `allow_active` -- aplikuje się do lokalnych klientów, którzy mają aktywne sesje, np. użytkownicy
zalogowani na TTY, czy mający odpalone środowisko graficzne.
- `allow_any` -- aplikuje się do każdego klienta (w tym też zdalnego), który próbuje zamontować
zasób.

Wszystkie te trzy elementy, tj. `allow_any` , `allow_inactive` oraz `allow_active` mogą przyjąć
jedną z poniższych wartości:

- `no` -- nieautoryzowany.
- `yes` -- autoryzowany.
- `auth_admin` -- wymagane uwierzytelnienie przez użytkownika z prawami administratora.
- `auth_self` -- wymagane jest uwierzytelnienie przez właściciela sesji, z której klient się wywodzi.
Niezalecane w systemach mających kilku użytkowników z powodu niedostatecznych restrykcji (zalecane
używanie `auth_admin` ).
- `auth_self_keep` -- podobne do `auth_self` ale autoryzacja jest zachowywana przez krótki przedział
czasu, np. 5 minut.
- `auth_admin_keep` -- podobne do `auth_admin` ale autoryzacja jest zachowywana przez krótki
przedział czasu, np. 5 minut.

## Konfiguracja PolicyKit

Domyślna konfiguracja PolicyKit dla montowania zasobów z wykorzystaniem UDisks zezwala każdemu
użytkownikowi lokalnemu posiadającemu aktywną sesję na zamontowanie dowolnego systemu plików bez
podawania jakiegokolwiek hasła. Można więc podłączyć dowolny nośnik i bez problemu zacząć z nim
wymieniać informacje. Tego typu zachowanie jest niedopuszczalne z punktu widzenia bezpieczeństwa
systemu. Musimy zatem napisać parę reguł dla PolicyKit, które skonfigurują nam zachowanie UDisks i
zezwolą tylko określonym użytkownikom w systemie (innym niż root) na montowanie nośników pamięci.

Pliki reguł PolicyKit umieszcza się w katalogu `/etc/polkit-1/rules.d/` . Nazwy plików mają postać
`nn-nazwa.rules` , czyli dwucyfrowy numerek oddzielony od nazwy za pomocą myślnika, a na końcu
sufiks `.rules` . Tylko pliki w takim systemie nazewniczym są brane pod uwagę przez PolKit. Trzeba
tutaj zaznaczyć jeszcze, że kolejność reguł w plikach również ma znaczenie. Generalnie te reguły
obecne wyżej w danym pliku mają pierwszeństwo przed tymi obecnymi w dalszej części pliku. Dlatego
też te bardziej specyficzne reguły trzeba umieścić na początku, a te bardziej rozległe na końcu
pliku.

### Logowanie reguł

Przy tworzeniu polityki dla PolicyKit warto jest zaprzęgnąć do pracy jego mechanizm logujący. W tak
powstałym logu będą dostępne wszystkie informacje, które będziemy w stanie wykorzystać przy pisaniu
reguł. Niemniej jednak, ten mechanizm logujący trzeba pierw włączyć, a robi się to przez stworzenie
pliku `00-log-access.rules` w katalogu `/etc/polkit-1/rules.d/` i dodanie do niego poniższej treści:

    polkit.addRule(function(action, subject) {
        polkit.log("action=" + action);
        polkit.log("subject=" + subject);
    });

Od tej pory za każdym razem jak tylko mechanizm PolicyKit zadziała w jakiś sposób, to będzie
generował stosowny komunikat w logu.

### Domyślne blokowanie montowania zasobów

Stwórzmy sobie jeszcze jeden plik w katalogu `/etc/polkit-1/rules.d/` , tj. `20-udisks2.rules` . Z
racji, że nie chcemy aby każdy użytkownik miał możliwość montować dowolny nośnik w naszym systemie,
to dodajemy na końcu tego utworzonego wyżej pliku taki oto kod:

    polkit.addRule(function(action) {
      if (action.id.indexOf("org.freedesktop.udisks2.") == 0) {
            return polkit.Result.NO;
          }
    });

Mówi on w zasadzie tyle, że jeśli jakiś użytkownik będzie próbował przeprowadzić akcję, której `id`
zaczyna się od `org.freedesktop.udisks2.` , to PolicyKit ma zwrócić `NO` , co oznacza brak
autoryzacji.

Jeśli byśmy w takiej sytuacji spróbowali zamontować system plików jakiegoś pendrive czy dysku
twardego, który podłączyliśmy do portu USB, to przywita nas taki oto komunikat:

![](/img/2019/04/001-udisks-udisks2-polkit-policykit-linux-debian-hdd-pendrive-deny-mount.png#small)

Z kolei gdybyśmy skorzystali z `return polkit.Result.AUTH_ADMIN;` , to byśmy zobaczyli poniższe
okienko, w którym trzeba by podać hasło:

![](/img/2019/04/002-udisks-udisks2-polkit-policykit-linux-debian-hdd-pendrive-auth-mount.png#medium)

Jeśli zawartość pliku `/etc/polkit-1/rules.d/40-debian-sudo.rules` zostanie wykomentaowana, to
wtedy będziemy proszeni o hasło użytkownika root. W przeciwnym wypadku członkowie grupy `sudo`
będą pytani o ich własne hasło, by potwierdzić swoją tożsamość.

Zatem dodając ten powyższy kod blokujemy użytkownikom możliwość korzystania z akcji UDisks.

W logu systemowym zaś przy próbie montowania zasobu możemy wyczytać te poniższe wiadomości:

    Apr 25 18:43:46 morfikownia polkitd[2457]: /etc/polkit-1/rules.d/00-log-access.rules:2:
      action=[Action id='org.freedesktop.udisks2.filesystem-mount' id.usage='filesystem'
      drive.serial='D57CA0A36079' id.label='ntfs_data' partition.flags='0x00000000'
      polkit.gettext_domain='udisks2' drive.removable.bus='usb' drive='Kingston DataTraveler 3.0 (/dev/sdc2)'
      partition.number='2' id.uuid='4CF875EB65AC2B02' partition.uuid='208bbb20-01'
      drive.vendor='Kingston' device='/dev/sdc2' id.type='ntfs' partition.type='0x07'
      polkit.message='Authentication is required to mount $(drive)' drive.revision='AB51'
      drive.model='DataTraveler 3.0' drive.removable='true']

    Apr 25 18:43:46 morfikownia polkitd[2457]: /etc/polkit-1/rules.d/00-log-access.rules:3:
      subject=[Subject pid=1928 user='morfik'
      groups=morfik,sudo,audio,dip,video,plugdev,systemd-journal,bluetooth,scanner,dane,p2p,docker,wireshark,wheel
      seat='seat0' session='4' local=true active=true]

Widzimy kto i kiedy próbował skorzystać z jakich akcji UDisks. Widoczne są też parametry
montowanego urządzenia oraz trochę dodatkowych informacji o samym użytkowniku.

### Zezwolenie użytkownikom na montowanie zasobów

Obecnie po zaaplikowaniu tej powyższej reguły sytuacja nie różni się niczym od zaprzęgania root'a
do zamontowania systemu plików jakiejś partycji. Użytkowników, którzy powinni mieć prawo montować
zasoby w systemie, trzeba potraktować nieco bardziej ulgowo. Trzeba zatem dorobić kilka wyjątków,
poniżej przykład:

    polkit.addRule(function(action, subject) {
      if (action.id.indexOf("org.freedesktop.udisks2.filesystem-mount") == 0 &&
          subject.local && subject.active &&
          subject.user == "morfik") {
            return polkit.Result.YES;
          }
    });

Ta powyższa zwrotka zezwoli lokalnemu użytkownikowi `morfik` w sesji aktywnej na skorzystanie z
akcji `org.freedesktop.udisks2.filesystem-mount` , która umożliwia zamontowanie systemu plików
dowolnego nośnika podłączonego do komputera. Ten użytkownik będzie również w stanie odmontować
zasoby ale tylko te, które sam zamontował. Wszystko co zamontuje sobie `morfik` będzie dostępne w
katalogu `/media/morfik/` .

Możliwe jest także zacieśnienie polityki, tak by konkretny użytkownik był w stanie montować tylko i
wyłącznie określone urządzenia w systemie. Dla przykładu:

    polkit.addRule(function(action, subject) {
      if (action.id.indexOf("org.freedesktop.udisks2.filesystem-mount") == 0 &&
          action.lookup("drive.serial") == "D57CA0A36079" &&
          action.lookup("id.type") == "ntfs" &&
          action.lookup("id.uuid") == "4CF875EB65AC2B02" &&
          action.lookup("partition.uuid") == "208bbb20-01" &&
          subject.local && subject.active &&
          subject.user == "morfik") {
            return polkit.Result.YES;
          }
    });

Dopasowań w `action.lookup` można określić ile się chce. Im ich będzie więcej, tym bardziej
specyficzna reguła. W tym przypadku musi się zgadzać numer seryjny urządzenia, typ systemu plików,
numer UUID systemu plików oraz numer UUID partycji. Jeśli któraś z tych wartości będzie inna, np.
urządzenie zostanie sformatowane nowym systemem plików, to użytkownik już nie będzie w stanie
zamontować takiego urządzenia. [Pełna lista atrybutów][6], na podstawie których można próbować
dopasować urządzenie, znajduje się tutaj.

W logu, który zostanie wygenerowany podczas montowania urządzenia, te wszystkie wartości potrzebne
by dopasować urządzenie będą wynotowane. Można zatem bez problemu skopiować je sobie i uzupełnić
wedle potrzeby.

## Nośniki systemowe vs. niesystemowe

UDisks rozróżnia pamięć masową i dzieli ją na systemową oraz niesystemową. Ten pierwszy rodzaj
pamięci masowej, to zwykle są dyski HDD/SSD, np. ten, na którym mamy zainstalowany system.
Natomiast drugi rodzaj pamięci dotyczy urządzeń podpinanych do portu USB (pendrive czy też karty
SD), które można odłączyć w dowolnym czasie podczas pracy systemu komputera. UDisks jest w stanie
rozróżnić te dwa typy pamięci w oparciu o `ATTR{removable}` , który można wyciągnąć przez
`udevadm` :

    # udevadm info --attribute-walk --name /dev/sdc

      looking at device '/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/2-1.3:1.0/host4/target4:0:0/4:0:0:0/block/sdc':
        ...
        ATTR{removable}=="1"
        ...

Jeśli `ATTR{removable}` ma wartość `0` to mamy do czynienia z dyskiem systemowym, jeśli zaś
widnieje tam wartość `1` , to z pamięcią przenośną. Warto sobie z tego faktu zdawać sprawę, bo
UDisks ma dwie osobne akcje `org.freedesktop.udisks2.filesystem-mount` oraz
`org.freedesktop.udisks2.filesystem-mount-system` w zależności od tego czy chcemy montować pamięć
przenośną czy systemową.

## Automatyczne montowanie podłączonych dysków

Wypadałoby jeszcze wspomnieć o możliwości automatycznego montowania systemu plików podłączanych
urządzeń. W takiej sytuacji po podłączeniu pendrive czy innego dysku USB, jego system plików
automatycznie zostanie zamontowany. Niemniej jednak, sam UDisks nie realizuje automatycznego
montowania zasobów -- one muszą być montowane za pomocą jakiegoś zewnętrznego mechanizmu i zwykle
menadżery plików (czy inne częci środowiska graficznego) go oferują. Tak czy inaczej, istnieje
także dedykowane narzędzie do tego celu, tj. `udiskie` (taki frontend dla UDisks).

Mechanizm automatycznego montowania nośników działa w oparciu o zmienne środowiskowe UDEV'a, są
m.in. `ENV{UDISKS_IGNORE}` , `ENV{UDISKS_AUTO}` oraz `ENV{UDISKS_SYSTEM}` . Wartości tych zmiennych
można podejrzeć w wyjściu `udisksctl` :

    $ udisksctl info -b /dev/sdc2
    /org/freedesktop/UDisks2/block_devices/sdc2:
      org.freedesktop.UDisks2.Block:
        ...
        HintAuto:                   true
        HintIgnore:                 false
        HintSystem:                 false
        ...

Domyślnie włączone jest automatyczne montowanie nośników i jeśli nam ten fakt przeszkadza, to
możemy powstrzymać nasz system przed tego typu zachowaniem. Wystarczy, że dorobimy jedną prostą
regułę dla UDEV'a. Tworzymy zatem w katalogu `/etc/udev/rules.d/` plik `99-udisks2-automount.rules`
i dodajemy w nim poniższą zawartość:

    SUBSYSTEMS=="usb", \
      ENV{UDISKS_AUTO}="0", \
      ENV{UDISKS_IGNORE}="1"

Generalnie rzecz biorąc, to zmienna `ENV{UDISKS_IGNORE}` informuje aplikacje, że mają one dany
zasób zignorować. W tym przypadku dopasowanie jest po interfejsie USB, zatem wszystkie urządzenia
USB będą przez aplikacje ignorowane. Oczywiście jeśli chcemy, aby tylko określone dyski USB były
montowane automatycznie, to naturalnie trzeba by napisać im osobne reguły określając przy tym
dopasowanie na podstawie `ATTRS{idProduct}` oraz `ATTRS{idVendor}`. Teoretycznie nie powinno się
korzystać ze zmiennej `ENV{UDISKS_IGNORE}` w przypadku zasobów, które powinny być widoczne w
aplikacjach, a jeśli nie chcemy montować ich automatycznie to powinno się korzystać ze zmiennej
`ENV{UDISKS_AUTO}` . Niemniej jednak, w przypadku mojego systemu zmienna `ENV{UDISKS_AUTO}` nie
działa i dlatego musiałem się posłużyć tą drugą zmienną. Można też ustawić nośnikom wymiennym
zmienną `ENV{UDISKS_SYSTEM}` , która przerobi nośniki przenośne na systemowe i tym samym sprawi, że
UDisks ich nie będzie automatycznie montował.

Po dodaniu reguły trzeba jeszcze przeładować konfigurację UDEV'a:

    # udevadm control --reload

Od tego momentu nasz system powinien zaprzestać automatycznie montować zewnętrzne nośniki
podłączane do portu USB.

## USBguard vs. UDisks + PolicyKit

Jakiś już czas temu [opisywałem mechanizm USBguard][7], który był poświęcony zagadnieniu ochrony
portów USB przed niezbyt przyjaznymi urządzeniami. Teraz mamy jeszcze opcję w postaci PolicyKit +
UDisks. Które z tych dwóch rozwiązań jest lepsze?

No jakby nie patrzeć, to UDisks + PolicyKit ogranicza się jedynie do szeroko rozumianej pamięci
masowej. Ten mechanizm jest w stanie wyprofilować system w taki sposób, by konkretni użytkownicy
mieli możliwość montować jedynie określone nośniki pamięci. Jeśli zaś chodzi o USBguard, to on z
kolei chroni nasz system przed każdym urządzeniem USB, a nie tylko tymi wyposażonymi w pamięć
masową. Dodatkowo USBguard działa w warstwie niższej i jest w stanie uniemożliwić kernelowi
komunikację z takim urządzeniem (dobór modułu). Te dwa mechanizmy (albo raczej trzy) uzupełniają
się wzajemnie i w mojej ocenie stanowią ideale połączenie, które jest w stanie ochronić nasz system
przed chyba wszystkimi atakami z udziałem pamięci masowej podłączanej do portów USB.


[1]: http://storaged.org/doc/udisks2-api/latest/
[2]: https://tracker.debian.org/pkg/udisks
[3]: https://tracker.debian.org/pkg/udisks2/
[4]: https://wiki.debian.org/AptPreferences
[5]: http://storaged.org/doc/udisks2-api/latest/udisks-polkit-actions.html
[6]: http://storaged.org/doc/udisks2-api/latest/udisks-polkit-actions.html
[7]: /post/jak-przy-pomocy-usbguard-zabezpieczyc-porty-usb-przed-zlosliwymi-urzadzeniami/
