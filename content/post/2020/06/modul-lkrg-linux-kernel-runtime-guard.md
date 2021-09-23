---
author: Morfik
categories:
- Linux
date: "2020-06-09T20:56:00Z"
lastmod: 2020-06-26 20:56:00 +0200
published: true
status: publish
tags:
- debian
- dkms
- lkrg
- kernel
- moduły-kernela
title: Moduł LKRG (Linux Kernel Runtime Guard)
---

Jakiś czas temu opisywałem [moduł TPE][1] (Trusted Path Execution) dla kernela linux, który jest w
stanie dość znacznie poprawić bezpieczeństwo naszego systemu. Ostatnio jednak natknąłem się na
[moduł LKRG][2] (Linux Kernel Runtime Guard), którego to zadaniem jest stać na straży samego jądra
operacyjnego i chronić je w czasie rzeczywistym przed różnego rodzaju zagrożeniami poprzez
wykrywanie eksploitów wykorzystujących luki w jego mechanizmach bezpieczeństwa. Jako, że ja
bardzo lubię zbroić swojego Debiana, to postanowiłem się przyjrzeć nieco bliżej temu całemu LKRG i
sprawdzić jego użyteczność. Trzeba jednak wiedzieć, że LKRG jest dostarczany w formie osobnego
modułu zamiast łaty na kernel, przez co trzeba będzie także postarać się o automatyzację pewnych
rzeczy, m.in. procesu budowania modułu przy aktualizacji kernela via DKMS.

<!--more-->
## Moduł LKRG

Jak zostało już nadmienione we wstępie, LKRG jest dostępny jako osobny moduł dla kernela, a nie
jako patch na kernel. Ten fakt może odstraszyć część użytkowników, którzy kompilują wszystkie
niezbędne moduły kernela bezpośrednio w jądro, by uniknąć budowania zbędnej i często niebezpiecznej
funkcjonalności, np. opcji ładowania/wyładowania modułów podczas pracy systemu operacyjnego.
Dodatkowo, jeśli wykorzystujemy [mechanizm Secure Boot mając EFI/UEFI][3], to trzeba będzie
poczynić drobne przygotowania, by ten dodatkowy moduł podpisać, gdyż w przeciwnym wypadku nie
zostanie on załadowany przez nasz OS. Na szczęście, jak to zwykle bywa w przypadku zewnętrznych
modułów, możemy zaprzęgnąć do pracy DKMS, w którym to możemy nieco [zautomatyzować sam proces
podpisywania modułów][5] przy ich instalacji w systemie. Zatem nie jest tak źle jak mogłoby się
wydawać, choć oczywiście mogłoby być trochę lepiej.

[Źródła modułu LKRG][6] są udostępnione na licencji GPL2 i można je pobrać przy pomocy `git` w
poniższy sposób:

    $ git clone https://bitbucket.org/Adam_pi3/lkrg-main.git lkrg
    $ cd lkrg/

### Problemy z LKRG na kernel 5.7

Moduł LKRG bez problemu się budował na kernelu 5.6 i wcześniejszych ale począwszy od wersji 5.7 w
kernelu nastąpiły pewne zmiany, które uniemożliwiły budowanie aktualnej wersji LKRG (0.7) na tym
nowszym jądrze. Stosowne zmiany w źródłach LKRG zostały już poczynione by dostosować je do kernela
5.7 ale póki co te zmiany są jedynie w wersji git i dlatego jeśli korzystamy z tego nowszego
kernela, to musimy pobrać źródła bezpośrednio z repozytorium git.

Te pobrane źródła pakujemy przy pomocy `tar` i nazywamy `lkrg_0.7+git20200604.orig.tar.xz` , gdzie
ten numerek po `+git` odnosi się do daty ostatniego commit'a:

    $ tar --exclude='.git' --exclude='.gitignore' --exclude='.pc' --exclude='debian' -cf - ../lkrg  | xz -9 -c - > ../lkrg_0.7+git20200604.orig.tar.xz

## Konfiguracja kernela na potrzeby LKRG

By moduł LKRG nam się zbudował bez większych problemów, musimy upewnić się, że mamy odpowiednio
skonfigurowany kernel. Wymagane włączenie jest tych poniższych opcji:

    CONFIG_KPROBES
    CONFIG_HAVE_KRETPROBES
    CONFIG_MODULE_UNLOAD
    CONFIG_KALLSYMS_ALL
    CONFIG_JUMP_LABEL
    CONFIG_STACKTRACE

## Konfiguracja DKMS dla modułu LKRG

Nic nie stoi na przeszkodzie by po pobraniu źródeł zbudować moduł przy pomocy `make` i zainstalować
go via `make install` ale lepszym rozwiązaniem jest [mechanizm DKMS][4], który będzie nam ten moduł
budował automatycznie ilekroć tylko kernel będzie aktualizowany. Możemy również pójść o krok dalej
i [zrobić sobie paczkę .deb przy pomocy pbuilder][7].

Poniżej znajduje się plik `dkms.conf` z konfiguracją dla DKMS:

    PACKAGE_NAME="lkrg"
    PACKAGE_VERSION="0.7"
    #BUILT_MODULE_LOCATION[0]="output"
    BUILT_MODULE_NAME[0]="p_lkrg"
    DEST_MODULE_LOCATION[0]="/updates/dkms"
    DEST_MODULE_NAME[0]="p_lkrg"
    REMAKE_INITRD="no"
    AUTOINSTALL="yes"

    MAKE[0]="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build"
    CLEAN="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build clean"

Ta widoczna wyżej wykomentowana linijka odnosi się do katalogu, do którego moduł LKRG zostanie
skopiowany po skompilowaniu. Niemniej jednak, moduł po zbudowaniu będzie również siedział w
katalogu głównym i system DKMS bez problemu odnajdzie stosowny plik po nazwie. Można zatem
korzystać z `BUILT_MODULE_LOCATION` lub też pozostawić ją wykomentowaną.

## Przygotowanie paczki .deb ze źródłami LKRG

Jeśli zaś chcemy przygotować sobie paczkę `.deb` , która zainstaluje nam wszystkie niezbędne pliki
w odpowiednim miejscu, to potrzebna nam będzie dodatkowa konfiguracja dla `pbuilder` . Tworzymy
zatem katalog `debian/` w głównym katalogu ze źródłami LKRG, a w nim te poniższe pliki.

### Plik `debian/source/format`

Standardowo w tym pliku umieszczamy poniższą linijkę:

    3.0 (quilt)

### Plik `debian/changelog`

Tutaj z kolei najważniejszą rolę odgrywa wersja, która znajduje się w nawiasie ( `0.7+git20200604` ,
bez `-1` ), oraz nazwa pakietu ( `lkrg` ). Te dwie wartości muszą pasować do nazwy paczki `tar.gz` ,
którą zrobiliśmy wcześniej:

    lkrg (0.7+git20200604-1) unstable; urgency=medium

      * Initial Release (Closes: #123456)

     -- Mikhail Morfikov <morfik@nsa.com>  Thu, 04 Jun 2020 23:06:27 +0200

### Plik `debian/control`

W tym pliku określamy wszystko to, co tyczy się procesu budowania paczki `.deb` :

    Source: lkrg
    Section: kernel
    Priority: optional
    Maintainer: Mikhail Morfikov <morfik@nsa.com>
    Homepage: https://www.openwall.com/lkrg/
    Vcs-Browser: https://bitbucket.org/Adam_pi3/lkrg-main
    Vcs-Git: https://bitbucket.org/Adam_pi3/lkrg-main.git
    Standards-Version: 4.5.0
    Rules-Requires-Root: no
    Build-Depends: debhelper (>= 13), debhelper-compat (= 13),
     dkms

    Package: lkrg-dkms
    Architecture: all
    Depends: ${shlibs:Depends}, ${misc:Depends},
     dkms
    Description: Linux Kernel Runtime Guard (LKRG) Source Code and DKMS
     Linux Kernel Runtime Integrity Checking and Exploit Detection.
     .
     LKRG provides security through diversity.
     Similar to running an uncommon operating system (kernel) would.
     .
     This package uses DKMS to automatically build the LKRG kernel
     module.

### Plik `debian/copyright`

Tutaj z kolei określamy wszystkie rzeczy związane z prawami autorskimi i licencjami do plików
projektu:

    Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
    Upstream-Name: lkrg
    Upstream-Contact: pi3ki31ny <pi3@itsec.pl>
    Source: https://www.openwall.com/lkrg/

    Files: *
    Copyright: 2018-2020 Adam 'pi3' Zabrocki
    License: GPL-2.0

    Files: debian/*
    Copyright: 2020 Mikhail Morfikov <morfik@nsa.com>
    License: GPL-2.0

    License: GPL-2.0
     This package is free software; you can redistribute it and/or modify
     it under the terms of the GNU General Public License as published by
     the Free Software Foundation; either version 2 of the License, or
     (at your option) any later version.
     .
     This package is distributed in the hope that it will be useful,
     but WITHOUT ANY WARRANTY; without even the implied warranty of
     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
     GNU General Public License for more details.
     .
     You should have received a copy of the GNU General Public License
     along with this program. If not, see <http://www.gnu.org/licenses/>
     .
     On Debian systems, the complete text of the GNU General
     Public License version 2 can be found in "/usr/share/common-licenses/GPL-2".

### Plik `debian/lkrg-dkms.dkms`

W tym pliku definiujemy konfigurację dla mechanizmu DKMS. W `PACKAGE_VERSION` nie określamy jako
takiej wersji modułu tylko `"#MODULE_VERSION#"` , by system budowania pakietu `.deb` bez problemu
uzupełniał nam wersję bez angażowania nas w ten proces:

    PACKAGE_NAME="lkrg"
    PACKAGE_VERSION="#MODULE_VERSION#"
    #BUILT_MODULE_LOCATION[0]="output"
    BUILT_MODULE_NAME[0]="p_lkrg"
    DEST_MODULE_LOCATION[0]="/updates/dkms"
    DEST_MODULE_NAME[0]="p_lkrg"
    REMAKE_INITRD="no"
    AUTOINSTALL="yes"

    MAKE[0]="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build"
    CLEAN="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build clean"

### Plik `debian/lkrg-dkms.service`

Projekt LKRG dostarcza usługę dla systemd, która ma na celu ładowanie i wyładowanie modułu. Jeśli
nas interesuje ta usługa to możemy ten plik sobie stworzyć, choć moim zdaniem lepszym rozwiązaniem
jest ładować moduł via plik `/etc/modules` . Tak czy inaczej poniżej znajduje się plik usługi dla
systemd:

    [Unit]
    Description=Linux Kernel Runtime Guard
    Documentation=https://bitbucket.org/Adam_pi3/lkrg-main/src/master/README
    DefaultDependencies=no
    After=systemd-modules-load.service
    Before=systemd-sysctl.service
    Before=sysinit.target shutdown.target
    Conflicts=shutdown.target
    ConditionKernelCommandLine=!nolkrg

    [Service]
    Type=oneshot
    ExecStart=/sbin/modprobe -v p_lkrg
    #ExecStop=/sbin/modprobe -v -r p_lkrg
    RemainAfterExit=yes

    [Install]
    WantedBy=sysinit.target

### Plik `debian/rules`

W tym pliku określamy to, co tak naprawdę ma się znaleźć w wynikowej paczce, jak ją zbudować i
jakie akcje będą przeprowadzane przy instalacji (czy usuwaniu) jej w docelowym systemie. W tym
przypadku samej kompilacji jako takiej nie będzie, jedynie przenosimy pliki w odpowiednie miejsca
oraz instalujemy usługę dla systemd w trybie wyłączonym (bez jej aktywacji):

    #!/usr/bin/make -f

    export DH_VERBOSE=1
    export DH_OPTIONS = -v
    export DEB_BUILD_MAINT_OPTIONS = hardening=+all
    DPKG_EXPORT_BUILDFLAGS = 1
    include /usr/share/dpkg/buildflags.mk
    include /usr/share/dpkg/pkg-info.mk

    %:
        dh $@ --with dkms

    override_dh_install:
        dh_install -p lkrg-dkms scripts/ usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p lkrg-dkms src/ usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p lkrg-dkms Makefile usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)

    override_dh_dkms:
        dh_dkms -V $(DEB_VERSION_UPSTREAM)

    override_dh_auto_configure:
    override_dh_auto_build:
    override_dh_auto_test:
    override_dh_auto_install:
    override_dh_auto_clean:

    override_dh_installsystemd:
        dh_installsystemd --no-enable --no-stop-on-upgrade

    override_dh_missing:
        dh_missing --list-missing --fail-missing

### Plik `debian/watch`

Ten plik ma za zadanie umożliwić pobranie nowszej wersji projektu, gdyby się takowa pojawiła, za
sprawą narzędzia `uscan` . Dodatkowo, jako że projekt LKRG podpisuje cyfrowo swoje paczki ze
źródłami, to będzie ten podpis automatycznie sprawdzamy i weryfikowany:

    version=4
    opts=pgpsigurlmangle=s/$/.sign/ https://www.openwall.com/lkrg/lkrg-(.+).tar.gz

### Plik `debian/upstream/signing-key.asc`

By proces weryfikacji podpisu mógł mieć miejsce, potrzebny nam jest klucz publiczny by zweryfikować
sygnaturę dołączoną do paczki `tar.gz` . Ten klucz publiczny możemy albo dodać ręcznie:

    -----BEGIN PGP PUBLIC KEY BLOCK-----

    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
    =kkUp
    -----END PGP PUBLIC KEY BLOCK-----

Albo też możemy w terminalu wpisać poniższe polecenie (trzeba pierw uprzednio pozyskać klucz
`0x05C027FD4BDC136E` ):

    $ gpg --armor --export-options export-minimal --export 0x05C027FD4BDC136E >> debian/upstream/signing-key.asc

#### Budowanie paczki .deb

Mając już przygotowany katalog `debian/` możemy zbudować paczkę `.deb` w poniższy sposób:

    $ debuild -S -sa -d -i --lintian-opts --profile debian
    $ sudo pbuilder --build ../lkrg_0.7+git20200604-1.dsc

Zbudowanie paczki zajmie dosłownie chwilę, jako że nie będzie tutaj żadnego procesu kompilacji
źródeł. Z chwilą, gdy budowa paczki dobiegnie końca, to w katalogu wynikowym `pbuiler`'a powinno
pojawić się kilka plików, z których jeden ma rozszerzenie `.deb` i to ten plik właśnie musimy
zainstalować w swoim systemie via `dpkg` . Oczywiście dobrze jest sobie [przygotować też lokalne
repozytorium dla APT][8] na tego typu pakiety, tak by mieć je wszystkie w jednym miejscu do
wygodnej instalacji w systemie za sprawą `apt-get`/`aptitude` .

## Instalacja modułu LKRG

Podczas instalacji pakietu `lkrg-dkms` zostanie zainicjowany DKMS i moduł LKRG zostanie
automatycznie zbudowany dla aktualnie załadowanego w pamięci operacyjnej kernela. Skompilowany plik
zaś zostanie umieszczony w odpowiednim katalogu z modułami kernela:

    # aptitude install lkrg-dkms
    The following NEW packages will be installed:
      lkrg-dkms
    0 packages upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
    Need to get 0 B/85.4 kB of archives. After unpacking 872 kB will be used.
    Get: 1 file:/media/Kabi/repozytorium/debian sid/main amd64 lkrg-dkms all 0.7+git20200604-1 [85.4 kB]
    Retrieving bug reports... Done
    Parsing Found/Fixed information... Done
    Selecting previously unselected package lkrg-dkms.
    (Reading database ... 225996 files and directories currently installed.)
    Preparing to unpack .../lkrg-dkms_0.7+git20200604-1_all.deb ...
    Unpacking lkrg-dkms (0.7+git20200604-1) ...
    Setting up lkrg-dkms (0.7+git20200604-1) ...
    Loading new lkrg-0.7+git20200604 DKMS files...
    Building for 5.7.0-amd64
    Building initial module for 5.7.0-amd64
    Done.

    p_lkrg.ko:
    Running module version sanity check.
     - Original module
       - No original module exists within this kernel
     - Installation
       - Installing to /lib/modules/5.7.0-amd64/updates/dkms/

    do_depmod 5.7.0-amd64


    DKMS: install completed.
    lkrg-dkms.service is a disabled or a static unit, not starting it.

    Current status: 0 (+0) broken, 3 (+0) upgradable, 16302 (+0) new.

Jak widać, również usługa dla systemd została zainstalowana ale nie została ona aktywowana.

Sprawdźmy jeszcze jak moduł jest widziany przez nasz system:

    # modinfo p_lkrg
    filename:       /lib/modules/5.7.0-amd64/updates/dkms/p_lkrg.ko
    license:        GPL v2
    description:    pi3's Linux kernel Runtime Guard
    author:         Adam 'pi3' Zabrocki (http://pi3.com.pl)
    srcversion:     B1DF6F54F2541DEC0B0408B
    depends:
    retpoline:      Y
    name:           p_lkrg
    vermagic:       5.7.0-amd64 SMP preempt mod_unload modversions RANDSTRUCT_PLUGIN_fb4bb4f688474f3bb721a6ba02f1a3c2d66b2c1ebb8b8f7574514f2a33a6eb05
    sig_id:         PKCS#7
    signer:         morfikov's kernel-signing key
    sig_key:        41:72:D5:0F:5C:43:2C:B4:D7:90:E7:6C:9D:07:74:7F:57:C6:7F:C7
    sig_hashalgo:   sha512
    signature:      39:BE:F0:CB:24:6C:C7:B6:D9:F7:C5:4E:A2:F0:89:BF:54:66:C6:E3:
                    C7:3D:D9:B6:3B:49:29:95:69:8B:44:B8:8E:2A:35:BF:CD:50:4C:6F:
                    B7:7A:2E:A6:B1:A0:2B:E5:80:96:B2:4C:F2:16:1F:1F:30:BF:0E:79:
                    0F:D9:B2:31:D0:6F:A9:4F:8E:9E:81:1F:8D:44:07:C4:DC:4B:43:2A:
                    A6:10:D7:B7:01:CE:BD:EA:E4:79:39:CE:C6:3F:B6:E5:43:0E:95:55:
                    71:94:87:C2:9E:4E:66:CC:41:0F:E5:F5:12:56:7E:6B:A7:E3:DA:09:
                    5B:6D:40:D8:CB:F0:C2:7B:CF:BC:E0:71:25:A3:09:4D:B0:AC:CF:8B:
                    8C:33:06:EC:63:0D:1C:B5:8B:A9:62:BF:D0:92:7C:5B:E7:D1:47:14:
                    85:17:EB:1A:52:81:20:79:9C:05:A7:5A:AC:08:68:22:34:1C:47:69:
                    F0:C2:CE:90:EB:A3:F9:81:9D:AE:F2:D2:C2:84:4B:7E:D5:5E:FB:B2:
                    95:05:1B:D4:30:9B:68:2A:6B:C6:A2:73:3D:1B:9A:16:2B:40:11:98:
                    A5:D4:07:7C:A9:EC:A6:05:C1:6B:C0:45:BF:98:97:59:3B:6C:60:1A:
                    94:3C:E0:E2:1F:30:5E:1F:6A:2C:80:3A:83:D5:58:0D
    parm:           log_level:log_level [3 (warn) is default] (uint)
    parm:           heartbeat:heartbeat [0 (don't print) is default] (uint)
    parm:           block_modules:block_modules [0 (don't block) is default] (uint)
    parm:           interval:interval [15 seconds is default] (uint)
    parm:           kint_validate:kint_validate [3 (periodically + random events) is default] (uint)
    parm:           kint_enforce:kint_enforce [2 (panic) is default] (uint)
    parm:           msr_validate:msr_validate [1 (enabled) is default] (uint)
    parm:           pint_validate:pint_validate [2 (current + waking_up) is default] (uint)
    parm:           pint_enforce:pint_enforce [1 (kill task) is default] (uint)
    parm:           umh_validate:umh_validate [1 (whitelist UMH paths) is default] (uint)
    parm:           umh_enforce:umh_enforce [1 (prevent execution) is default] (uint)
    parm:           pcfi_validate:pcfi_validate [2 (fully enabled pCFI) is default] (uint)
    parm:           pcfi_enforce:pcfi_enforce [1 (kill task) is default] (uint)
    parm:           smep_validate:smep_validate [1 (enabled) is default] (uint)
    parm:           smep_enforce:smep_enforce [2 (panic) is default] (uint)
    parm:           smap_validate:smap_validate [1 (enabled) is default] (uint)
    parm:           smap_enforce:smap_enforce [2 (panic) is default] (uint)

Tak zainstalowany moduł trzeba jeszcze załadować. Możemy to zrobić ręcznie przy pomocy `modprobe` ,
by sprawdzić czy wszystko działa jak należy:

    kernel: [p_lkrg] Loading LKRG...
    kernel: [p_lkrg] System does NOT support SMAP. LKRG can't enforce SMAP validation :(
    kernel: Freezing user space processes ... (elapsed 0.023 seconds) done.
    kernel: OOM killer disabled.
    kernel: [p_lkrg] 8/23 UMH paths were whitelisted...
    kernel: [p_lkrg] LKRG initialized successfully!
    kernel: OOM killer enabled.
    kernel: Restarting tasks ... done.

Informacja `LKRG initialized successfully!` oznajmia nam jasno, że inicjacja modułu została
zakończona powodzeniem, zatem wszytko działa jak należy.

## Ładowanie modułu LKRG przy starcie systemu

Moduły kernela mają to do siebie, że zwykle są ładowane podczas startu systemu lub w późniejszym
czasie jego pracy, gdy są potrzebne. O załadowanie modułu LKRG musimy się zatroszczyć jednak sami i
po to była ta usługa systemd, która miała za zadanie załadować ten moduł wcześnie podczas startu
systemu. Jeśli z jakiegoś powodu nie chcemy korzystać z usługi systemd, to zawsze możemy dodać
nazwę modułu do pliku `/etc/modules` lub też stworzyć osobny plik w katalogu `/etc/modules-load.d/`
o poniższej treści:

    p_lkrg

Z chwilą, gdy moduł LKRG zostanie załadowany w systemie, to żadna [sekcja .text][22] [nie będzie
już mogła zostać zmodyfikowana][23]. Niemniej jednak, cześć modułów kernela wykorzystuje interfejs
`kprobes` , który wstrzykuje instrukcję 0xCC do monitorowanej przez LKRG funkcji. Dlatego też
ważnym jest, by wszystkie moduły korzystające z `kprobes` zostały załadowane zanim moduł LKRG
zostanie aktywowany.

Warto dodać w tym miejscu, że usługa systemd zawiera warunek `ConditionKernelCommandLine=!nolkrg` ,
który to nakazuje załadować moduł LKRG jedynie w przypadku, gdy linijka kernela w konfiguracji
bootloader'a nie zawiera parametru `nolkrg` . W ten sposób jeśli system nie będzie chciał nam z
jakiegoś powodu wystartować, to zawsze możemy dopisać ten parametr w linijce kernela, przez co
systemd nie załaduje tej usługi na starcie systemu.

## Konfiguracja modułu LKRG

Zachowanie LKRG można kontrolować zmieniając opcje modułu przy ładowaniu go w systemie, lub też
przepisując wartości parametrów `sysctl` .  Póki co dokumentacja LKRG jest dostosowana do wersji
0.7, która wyszła już jakiś czas temu, a od tego momentu trochę zmian w projekcie zaszło i ta
dokumentacja jest ździebko nieaktualna. Niedługo zostanie wypuszczona wersja 0.8 i ten [problem
zostanie zaadresowany][13]. Podpierając się jednak [tym commit'em][14], [plikiem README][15] na
git oraz [komentarzem autora][16], udało mi się opisać wszystkie parametry, które są dostępne w
`sysctl` .

 - `lkrg.block_modules` -- blokuje możliwość ładowania modułów przez kernel w przypadku, gdy ustawi
                            się tutaj wartość "1". Ten parametr jest podobny do `modules_disabled` ,
                            z tą różnicą, że nie wspiera blokowania wyładowania modułów oraz można
                            przepisać jego wartość z "1" na "0" bez potrzeby restartu maszyny.
                            (Domyślnie: "0")
 - `lkrg.heartbeat`     -- jeśli ustawiony na "1", określa czy wypisywać co jakiś czas w logu
                            systemowym wiadomość "System is clean!". (Domyślnie: "0")
 - `lkrg.interval`      -- kontroluje jak często zegar kernela, który woła funkcje sprawdzające
                            integralność systemu, będzie uruchamiany. Nie można tutaj ustawić mniej
                            niż 5 sekund, by nie zjadać zbyt wielu zasobów systemowych oraz nie
                            można ustawić więcej niż 1800s (pół godziny). (Domyślnie: "15")
 - `lkrg.log_level`     -- ten parametr może przyjąć wartość z zakresu "0-4" (lub "0-6" jeśli
                            debugowanie zostało użyte). Jeśli mamy problemy z LKRG, to te logi mogą
                            się okazać bardzo pomocne w namierzeniu przyczyny. (Domyślnie: "3")
 - `lkrg.hide`          -- ma za zadanie ździebko zamaskować wykorzystanie LKRG przez ukrycie go na
                            liście modułów (w `lsmod`). Niemniej jednak, trzeba liczyć się z faktem,
                            że LKRG może zostać wykryty za sprawą interfejsu `/sys` lub `/proc`
                            (przykładowo obecność katalogu `/proc/sys/lkrg/`), co może w przyszłości
                            zostanie zaadresowane. (Domyślnie: "0")
 - `lkrg.kint_validate` -- kontroluje weryfikację integralności kernela/systemu. Określenie
                            wartości "0" wyłącza sprawdzenie integralności. Można także określić "1"
                            i w takim przypadku sprawdzenie integralności odbywać się będzie tylko
                            na wyraźne żądanie (przez `sysctl -w lkrg.trigger=1`). Jest też do
                            określenia wartość "2", gdzie sprawdzenie integralności odbywać się
                            będzie co pewien ustalony interwał czasu zależny od `lkrg.interval`
                            (domyślnie co 15s). W przypadku "3" sprawdzenie integralności dokonywane
                            będzie na tej samej zasadzie co w przypadku "2" z tą różnicą, że
                            dodatkowo będzie ono odbywać się przy [różnych zdarzeniach systemowych]
                            [17], np. podłączanie urządzeń, zarządzanie energia, zdarzenia sieciowe,
                            itp. (Domyślnie: "3")
 - `lkrg.kint_enforce`  -- określa co zrobić w sytuacji, gdy sprawdzenie integralności kernela się
                            nie powiedzie. Możemy tutaj określić wartość "0", gdzie system zaloguje
                            ostrzeżenie tylko jeden raz i będzie kontynuował swoją pracę tak jak
                            to do tej pory robił. W przypadku podania "1" system również wypisze
                            stosowny komunikat w logu z tą różnicą, że za każdym razem, gdy
                            sprawdzenie integralności się nie powiedzie (te komunikaty mogą dość
                            mocno zaspamować log). Natomiast w przypadku ustawienia "2" kernel
                            spanikuje i dalsza praca systemu nie będzie już możliwa. (Domyślnie:
                            "2")
 - `lkrg.pint_validate` -- kontroluje logikę weryfikacji procesów. W przypadku ustawienia "0"
                            sprawdzanie procesów zostanie wyłączone. W przypadku "1" będą
                            weryfikowane tylko te procesy, które aktualnie są wykonywane. Można
                            także tutaj określić "2", gdzie poza aktualnie wykonywanymi procesami
                            sprawdzane będą także te procesy mające stan RUNNING. No i jest jeszcze
                            wartość "3", która weryfikuje wszystkie procesy w systemie bez względu
                            na ich stan (tryb dla paranoików). (Domyślnie: "2")
 - `lkrg.pint_enforce`  -- kontroluje zachowanie LKRG w przypadku, gdy kontrola integralności
                            procesów się nie powiedzie. W przypadku określenia wartości "0"
                            zostanie wypisany tylko komunikat ostrzegawczy i system będzie
                            kontynuował dalej swoją pracę. W przypadku ustawienia "1" zostanie
                            ubity proces, który nie przeszedł pomyślnie weryfikacji. Można też
                            określić wartość "2" i w takiej sytuacji, gdy jakiś proces nie
                            przejdzie weryfikacji, to kernel spanikuje. (Domyślnie: "1")
 - `lkrg.msr_validate`  -- włącza weryfikację rejestrów MSR (Model-Specific Register) w procesorach
                            o architekturach x86 i x86-64. Czasami bywają jednak sytuacje, w
                            których taka weryfikacja MSR jest niepożądana, np. gdy LKRG działa na
                            hoście, który zarządza maszynami wirtualnymi. W takim przypadku, host
                            może dynamicznie przekonfigurować część rejestrów MSR, które LKRG
                            weryfikuje. Ustawienie tutaj wartości "0" wyłączy weryfikację MSR,
                            natomiast wartość "1" tę weryfikację włącza. (Domyślnie: "1")
 - `lkrg.trigger`       -- ma na celu manualne zainicjowanie sprawdzenia integralności systemu
                            przez ustawienie tutaj "1". Ta wartość automatycznie zostanie
                            przepisana na "0" po przeprowadzeniu całego procesu. (Domyślnie: "0")
 - `lkrg.smep_validate` -- kontroluje weryfikację SMEP (Supervisor Mode Execution Prevention). By
                            ten mechanizm włączyć trzeba tutaj ustawić "1", a by wyłączyć "0".
                            (Default: "1")
 - `lkrg.smep_enforce`  -- kontroluje zachowanie LKRG w przypadku, gdy weryfikacja SMEP nie
                            zakończy się powodzeniem. W przypadku wartości "0" zostanie wypisany w
                            logu komunikat ostrzegawczy, a praca systemu nie zostanie w żaden
                            sposób przerwana. W przypadku ustawienia "1" zostanie wypisany
                            komunikat ostrzegawczy, a stan SMEP zostanie odtworzony do tego sprzed
                            dokonania modyfikacji. Zaś w przypadku ustawienia "2" kernel spanikuje.
                            (Domyślnie: "2")
 - `lkrg.smap_validate` -- podobny do `lkrg.smep_validate`, z tą różnicą, że odnosi się do SMAP
                            (Supervisor Mode Access Prevention). (Domyślnie: "0")
 - `lkrg.smap_enforce`  -- podobny do `lkrg.smep_enforce`, z tą różnicą, że odnosi się do SMAP.
                            (Domyślnie: "0")
 - `lkrg.umh_validate`  -- kontroluje weryfikację [UMH][18] (the kernel UserMode Helper). Wartość
                            "0" wyłącza weryfikację UMH. Z kolei wartość "1" zezwala UMH na
                            wykonanie binarek tylko z białej listy. Jest jeszcze wartość "2",
                            której ustawienie całkowicie blokuje UMH. (Domyślnie: "1")
 - `lkrg.umh_enforce`   -- kontroluje zachowanie LKRG w przypadku, gdy weryfikacja UMH nie zakończy
                            się powodzeniem. W przypadku ustawienia "0" zostanie wypisany jedynie
                            komunikat ostrzegawczy. W przypadku "1" wykonanie pliku binarnego
                            zostanie zabronione, a w przypadku "2" kernel spanikuje. (Domyślnie:
                            "1")
 - `lkrg.pcfi_validate` -- kontroluje weryfikację pCFI ([poor's man CFI][19], [Control-Flow
                            Integrity][20]). Wartość "0" wyłącza weryfikację. Wartość "1" włącza
                            słaby pCFI (weak pCFI), gdzie weryfikowany jest wskaźnik stosu (stack
                            pointer) oraz strona stosu (stack page) ale bez stackwalk, przez co
                            LKRG ma szanse na wykrycie jedynie ataków ROP (Return Oriented
                            Programming). Zaś wartość "2" włącza pełną weryfikację pCFI.
                            (Domyślnie: "2")
 - `lkrg.pcfi_enforce`  -- kontroluje zachowanie LKRG w przypadku, gdy weryfikacja pCFI nie
                            zakończy się powodzeniem. W przypadku ustawienia wartości "0" zostanie
                            zalogowane jedynie ostrzeżenie w logu systemowym. W przypadku wartości
                            "1" zostanie zabity proces, który nie przeszedł weryfikacji. W
                            przypadku wartości "2" kernel spanikuje. (Domyślnie: "1")

### Profilowanie ustawień

Stosunkowo niedawno też [pojawiła się możliwość profilowania ustawień LKRG][21] pozwalająca przy
pomocy jednego parametru określić wartości wszystkich opcji `*_validate` lub `*_enforce` . Chodzi o
`lkrg.profile_enforce` oraz `lkrg.profile_validate` , które domyślnie przyjmują wartość "2" i "3"
odpowiadającą za tryb mieszany (własne ustawienia). Parametr `lkrg.profile_validate` może przyjąć
wartości z zakresu 0-4, a `lkrg.profile_enforce` z zakresu 0-3.

Poniżej znajduje się krótka rozpiska dla wartości parametru `lkrg.profile_validate` :

    |=========================================================================================|
    |          0 (Disabled)          |        1 (Light)       |         2 (Balanced)          |
    |================================|========================|===============================|
    | -> kint_validate = 0 (Disabled)| 1 (Manual trigger only)| 2 (Triggered by timer)        |
    | -> pint_validate = 0 (Disabled)| 1 (Current task only)  | 2 (Current + weaking up task) |
    | -> pcfi_validate = 0 (Disabled)| 1 (Weak pCFI)          | 1 (Weak pCFI)                 |
    | ->  umh_validate = 0 (Disabled)| 1 (Whitelist)          | 1 (Whitelist)                 |
    | ->  msr_validate = 0 (Disabled)| 0 (Disabled)           | 0 (Disabled)                  |
    | -> smep_validate = 0 (Disabled)| 1 (Enabled)            | 1 (Enabled)                   |
    | -> smap_validate = 0 (Disabled)| 1 (Enabled)            | 1 (Enabled)                   |
    |=========================================================================================|
    |               3 (Heavy)               |                  4 (Paranoid)                   |
    |=========================================================================================|
    | 3 (Triggered by timer + random events)| 3 (Triggered by timer + random events)          |
    | 2 (Current + weaking up task)         | 3 (Verify all tasks in the system by every hook)|
    | 2 (Full pCFI)                         | 2 (Full pCFI)                                   |
    | 1 (Whitelist)                         | 2 (Full UMH lock-down)                          |
    | 1 (Disabled)                          | 1 (Enabled)                                     |
    | 1 (Enabled)                           | 1 (Enabled)                                     |
    | 1 (Enabled)                           | 1 (Enabled)                                     |
    |=========================================================================================|

Niżej zaś znajduje się rozpiska wartości dla parametru `lkrg.profile_enforce` :

    |================================================================================|
    |          0 (Log & Accept)         | 1 (Selective) | 2 (Strict)   | 3 (Paranoid)|
    |===================================|============================================|
    | -> kint_enforce = 0 (Log & accept)| 1 (Log only)  | 2 (Panic)    | 2 (Panic)   |
    | -> pint_enforce = 0 (Log & accept)| 1 (Kill task) | 1 (Kill task)| 2 (Panic)   |
    | -> pcfi_enforce = 0 (Log only)    | 1 (Kill task) | 1 (Kill task)| 2 (Panic)   |
    | ->  umh_enforce = 0 (Log only)    | 1 (Prev exec) | 1 (Prev exec)| 2 (Panic)   |
    | -> smep_enforce = 0 (Log & accept)| 2 (Panic)     | 2 (Panic)    | 2 (Panic)   |
    | -> smap_enforce = 0 (Log & accept)| 2 (Panic)     | 2 (Panic)    | 2 (Panic)   |
    |================================================================================|

Jak widać, jeśli nie chce nam się zbytnio myśleć na ustawieniami, to możemy wybrać jeden z
predefiniowanych profili i on już zajmie się wszystkimi zależnymi ustawieniami za nas.

## Podsumowanie

Jakby nie patrzeć, ten osobny moduł LKRG trochę miesza w systemach, które z modułów nie korzystają.
Nie chodzi tutaj nawet o zewnętrzne moduły kernela (te Out Of Tree) ale ogólnie o posiadanie
[własnego kernela dla konkretnej maszyny linux'owej][24], gdzie z zasady wszystkie potrzebne moduły
są wbudowane bezpośrednio w samo jądro operacyjne właśnie też m.in. ze względów bezpieczeństwa. Do
tego dochodzi oczywiście potrzeba zastosowana DKMS oraz problemy z podpisami w środowiskach
EFI/UEFI.

Póki co moduł LKRG da radę wykryć bez większego trudu i był/jest on też podatny na [różne metody
obejścia][11] (choć te metody zgromadzone w tym linku już nie działają). Być może problem z
wykryciem modułu LKRG zostanie zaadresowany w przyszłości ale póki co trzeba mieć ten fakt na
uwadze. Trzeba też pamiętać o tym, że LKRG nie ochroni naszego systemu, gdy problemy tkwią
bezpośrednio w hardware, np. CPU i [nie da rady nas ochronić przed choćby Meltdown][10]. Podobnie
sprawa wygląda w przypadku ataków wymierzonych w procedurę startu linux'a (MBR/BIOS/UEFI,
initramfs/initrd, hibernacja, itp) -- te ataki nie zostaną zatrzymane przez LKRG. Warto też
zaznaczyć tutaj, że LKRG nie chroni nas również przed postronnymi osobami zaglądającymi nam przez
ramię podczas wpisywania haseł przy logowaniu się na root czy przy otwieraniu zaszyfrowanych
kontenerów LUKS. Dlatego też dobrze jest się zapoznać się z [modelem zagrożenia LKRG][12], by nie
mieć później pretensji, że ochrona nie zadziałała. Dodatkowo, warto też jest zaznajomić się z [tym
opracowaniem poświęconym metodom wykrywania rootkit'ów w systemach unix/linux][27]. Może ono nieco
rozjaśnić umysł osobom, które nie do końca zdają sobie sprawę przed jakimi zagrożeniami LKRG ma
chronić system naszego komputera.

Utrata wydajności systemu jaką niesie ze sobą LKRG to około 0,7-2,5% w zależności od
skonfigurowanych opcji ochrony. Wcześniej te liczby wyglądały o wiele gorzej, tj. 6,5%. Zatem można
liczyć, że ten moduł z czasem będzie jeszcze mniej zasobożerny niż jest w tej chwili. Niemniej
jednak, jeśli chodzi o laptopy czy desktopy, to większego spadku wydajności raczej nie zauważymy.

Problemy z nowszymi wersjami kernela/gcc też mogą ździebko irytować ale też trzeba mieć na
względzie fakt, że taka jest specyfika oprogramowania, które nieustannie jest zmieniane i od czasu
do czasu pewne zależne projekty mogą mieć problemy z działaniem. Kluczowe w takiej sytuacji jest
zawsze podejście dewelopera do poprawienia zaistniałego problemu. Wsparcie dla kernela 5.7 zostało
w zasadzie dodane [w parę dni po zgłoszeniu całej sytuacji][25], zatem projekt jak najbardziej się
rozwija, a zgłaszane nieprawidłowości są dość szybko weryfikowane i poprawiane.

Od dłuższego też czasu [LKRG próbuje się wbić do repozytorium Debiana][26] ale coś słabo mu to
idzie. Jedno jest pewne, gdy mu się w końcu uda, to instalacja tego modułu w systemie ulegnie
sporemu uproszczeniu, co powinno przyczynić się do większego zainteresowania projektem.


[1]: /post/modul-tpe-trusted-path-execution-dla-kernela-linux/
[2]: https://www.openwall.com/lkrg/
[3]: /post/jak-dodac-wlasne-klucze-dla-secure-boot-do-firmware-efi-uefi-pod-linux/
[4]: /post/dkms-czyli-automatycznie-budowane-moduly/
[5]: /post/automatyczne-podpisywanie-modulow-kernela-przez-dkms/
[6]: https://bitbucket.org/Adam_pi3/lkrg-main/
[7]: /post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/
[8]: /post/tworzenie-repozytorium-przy-pomocy-reprepro/
[10]: https://zaufanatrzeciastrona.pl/post/linux-kernel-runtime-guard-ochrona-przed-jeszcze-nieznanymi-exploitami/
[11]: https://github.com/milabs/lkrg-bypass
[12]: https://openwall.info/wiki/p_lkrg/Threat_model
[13]: https://www.openwall.com/lists/lkrg-users/2020/06/08/2
[14]: https://bitbucket.org/Adam_pi3/lkrg-main/commits/2febcf467d6182e9bd180334e2601c79812f2cf5
[15]: https://bitbucket.org/Adam_pi3/lkrg-main/src/master/README
[16]: https://www.openwall.com/lists/lkrg-users/2020/06/09/3
[17]: https://openwall.info/wiki/p_lkrg/Main#When-is-the-LKRG-validation-routine-executed
[18]: https://kernelnewbies.org/KernelProjects/usermode-helper-enhancements
[19]: http://blog.pi3.com.pl/?cat=7
[20]: https://lwn.net/Articles/810077/
[21]: https://bitbucket.org/Adam_pi3/lkrg-main/commits/11da921d4196741b69f5156a9fd01be980141cb4
[22]: https://en.wikipedia.org/wiki/Code_segment
[23]: https://openwall.info/wiki/p_lkrg/Main#Caveats1
[24]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[25]: https://www.openwall.com/lists/lkrg-users/2020/06/02/1
[26]: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=944476
[27]: http://jultika.oulu.fi/files/nbnfioulu-202004201485.pdf
