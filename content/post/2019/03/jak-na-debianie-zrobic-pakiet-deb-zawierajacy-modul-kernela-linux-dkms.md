---
author: Morfik
categories:
- Linux
date: "2019-03-17T09:37:28Z"
published: true
status: publish
tags:
- debian
- kernel
- dkms
title: Jak na Debianie zrobić pakiet .deb zawierający moduł kernela linux (DKMS)
---

Kernel linux'a jest dość złożonym organizmem, który może zostać rozbudowany przy pomocy dodatkowego
kodu ładowanego w postaci zewnętrznych modułów. Czasem ze względu na wczesny etap prac nad nową
funkcjonalnością jądra, taki moduł może zachowywać się dość nieprzewidywalnie, co przekreśla jego
szanse na pojawienie się w stabilnych wydaniach kernela. Czasem też z jakiegoś niezrozumiałego
powodu pewne rzeczy nie są celowo dodawane do jądra operacyjnego. Jedną z nich
jest [moduł Trusted Path Execution](https://github.com/cormander/tpe-lkm) (TPE), który jest w
stanie znacznie poprawić bezpieczeństwo systemu uniemożliwiając przeprowadzenie w nim szeregu
podejrzanych działań. W Debianie tego typu niedogodności związane z brakiem pewnych modułów można
obejść przez
zastosowanie [mechanizmu DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support), który
przy instalacji modułu spoza głównego drzewa kernela linux'a jest nam go w stanie automatycznie
zbudować. W repozytorium dystrybucji Debiana znajduje się już szereg pakietów z modułami (mają
końcówki `-dkms` ) i w prosty sposób można je sobie doinstalować. Co jednak w przypadku, gdy mamy
moduł, którego nikt jeszcze nie przygotował i nie wrzucił do repozytorium? Co, gdy takich modułów
mamy kilka, a przy tym korzystamy z najnowszego stabilnego kernela, który jest aktualizowany
średnio co kilka dni? Ręczna budowa wszystkich zewnętrznych modułów za każdym razem jak tylko
wyjdzie nowsza wersja kernela, to nie najlepsze wyjście, zwłaszcza jak dojdzie nam do tego
aktualizacja samych modułów. Można za to zrobić sobie paczkę `.deb` tak, by przy instalacji nowego
jądra operacyjnego, system nam automatycznie zbudował wszystkie dodatkowe moduły, których nasz
komputer wymaga do poprawnej pracy.

<!--more-->
## Parę słów na temat budowy pakietu .deb

Cały proces związany z budową pakietów `.deb` dla Debiana jest dość szerokim zagadnieniem i nie
będę go omawiał w tym artykule szczegółowo. Kiedyś napisałem spory kawałek tekstu poświęcony
właśnie [zagadnieniu budowy paczek dla tej dystrybucji](/post/poradnik-maintainera-czyli-jak-zrobic-pakiet-deb/).
Cała wiedza zawarta w tym podlinkowanym artykule nie jest jednak niezbędna do zbudowania prostej
paczki `.deb` . Można pójść nawet na skróty i skorzystać z `dpkg-buildpackage -b -us -uc` . Jeśli
jednak mamy już przygotowane środowisko pod `pbuilder` (w tym przypadku jest) i do tego jeszcze
własne [lokalne repozytorium na bazie reprepro](/post/tworzenie-repozytorium-przy-pomocy-reprepro/)
(i to również jest), to ogarnięcie `pbuilder`'a i budowanie paczek `.deb` za jego sprawą bardzo
ułatwia życie. Zatem nie trzeba studiować tamtego poradnika by zrobić sobie pakiet z modułem DKMS.
Niemniej jednak, jeśli podczas procesu budowy wystąpią jakieś błędy, to możemy nie być w stanie ich
przezwyciężyć. Przydałoby się zatem chociaż pobieżnie tamten artykuł przeczytać.

W niniejszym artykule skupię się głównie na aspektach budowy pakietu związanych z mechanizmem DKMS
i to te kwestie będą tutaj opisane bardziej szczegółowo. Cały proces zostanie przeprowadzony na
przykładzie pakietowania modułu TPE.

## Przygotowanie źródeł modułu kernela

Każdy moduł kernela ma swoje źródła, które musimy sobie pierw pozyskać. Tutaj sprawa jest prosta,
bo projekt jest hostowany na GitHub'ie i wystarczy sklonować repozytorium w poniższy sposób:

    $ git clone https://github.com/cormander/tpe-lkm tpe-2.0.3

Mając źródła, trzeba je zdebianizować. Ten proces polega na stworzeniu katalogu `debian/` i
uzupełnieniu w nim szeregu plików. Te pliki można tworzyć ręcznie ale lepiej jest zaprzęgnąć do
pracy narzędzie `dh_make` , które znajduje się w pakiecie `debhelper` . Skorzystanie z `dh_make`
będzie możliwe dopiero, gdy format nazwy katalogu będzie miał postać `nazwapaczki-wersja` .

W przypadku modułów DKMS ogromne znaczenie ma nazwa samego modułu. Nie możemy sobie dowolnie wybrać
nazwy pakietu, bo nawet najdrobniejsza różnica w tych nazwach sprawi, że mechanizm DKMS nam tego
modułu nie będzie potrafił zbudować.

Nazwę modułu najlepiej jest odczytać z pliku `Makefile` , który powinien się znajdować w głównym
katalogu ze źródłami. W pierwszej linijce tego pliku mamy coś takiego:

    MODULE_NAME := tpe

Dlatego nazwa katalogu, do którego klonowaliśmy repozytorium zaczyna się od `tpe` , po którym mamy
numerek wersji `2.0.3` . Ta nazwa jest jedynie tymczasowa. W przypadku budowania modułów spoza
głównego drzewa kernela będzie zwykle normalnym stanem rzeczy, że ich autorzy dość często te moduły
będą aktualizować (dodawać nowe commit'y) ale niekoniecznie będą oni przy tym wydawać nową wersję
modułu. Dlatego też będzie trzeba posługiwać się pośrednią wersją, np. w postaci daty ostatniego
commit'a w repozytorium git. Ta powyższa nazwa katalogu źródeł `tpe-2.0.3` ma jedynie na celu
pomóc w wygenerowaniu plików szkieletowych, które później poddamy edycji.

Mając przygotowaną nazwę katalogu źródeł, przechodzimy do niego i wpisujemy w terminalu poniższe
polecenie:

    $ dh_make --single --packagename tpe --email morfik@nsa.com --native --copyright gpl2
    Maintainer Name     : Mikhail Morfikov
    Email-Address       : morfik@nsa.com
    Date                : Sun, 17 Mar 2019 00:45:27 +0100
    Package Name        : tpe
    Version             : 2.0.3
    License             : gpl2
    Package Type        : single
    Are the details correct? [Y/n/q]

To powyższe polecenie stworzyło nam katalog `debian/` . Z plików w tym folderze zostawiamy jedynie
katalog `source/` oraz pliki `changelog` , `compat` , `control` , `copyright` i `rules` .

### Plik debian/control

Do pliku `control` wrzucamy poniższą zawartość:

    Source: tpe
    Section: kernel
    Priority: optional
    Maintainer: Mikhail Morfikov <morfik@nsa.com>
    Build-Depends: debhelper (>= 11~),
      dkms
    Standards-Version: 4.3.0
    Homepage: https://github.com/cormander/tpe-lkm
    Vcs-Git: https://github.com/cormander/tpe-lkm.git
    Vcs-Browser: https://github.com/cormander/tpe-lkm

    Package: tpe-dkms
    Architecture: all
    Depends: ${misc:Depends},
     dkms
    Recommends: linux-headers
    Description: Trusted Path Execution (TPE) Linux Kernel Module, Version 2
     Trusted Path Execution is a security feature that denies users from
     executing programs that are not owned by root, or are writable. This
     closes the door on a whole category of exploits where a malicious
     user tries to execute his or her own code to attack the system.
     .
     Since this module doesn't use any kind of ACLs, it works out of the
     box with no configuration. It isn't complicated to test or deploy to
     current production systems. Just install it and you're done!

Pozycja `Source:` ma wskazywać na nazwę, która była w pliku `Makefile` . Każda paczka zawierająca
moduły DKMS musi kończyć się frazą `-dkms` . Dlatego też w `Package:` mamy `tpe-dkms` . Zależności
zawsze będą takie same. Pozostałe pola trzeba sobie dostosować w zależności od modułu.

### Plik debian/changelog

Plik `changelog` edytujemy wpisując w terminalu `dch -i` i uzupełniamy w poniższy sposób:

    tpe (2.0.3~git20181202-1) unstable; urgency=medium

      * Initial release. (Closes: #123456)
      * Upstream commit: e68ba85

     -- Mikhail Morfikov <morfik@nsa.com>  Sat, 16 Mar 2019 19:25:16 +0100

Tutaj już się nam zmieniła wersja `2.0.3~git20181202-1` , bo od `2.0.3` zostały w repozytorium git
poczynione pewne zmiany i dlatego nie powinniśmy dawać samego `2.0.3` .

### Plik debian/copyright

Plik copyright nie jest aż taki ważny, gdy zamierzamy budować paczkę z modułem głównie dla siebie
ale wypadłoby go uzupełnić tak, by miał ręce i nogi:

    Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
    Upstream-Name: tpe
    Source: https://github.com/cormander/tpe-lkm

    Files: *
    Copyright: 2011-2018 Corey Henderson
    License: GPL-2

    Files: debian/*
    Copyright: 2019 Mikhail Morfikov <morfik@nsa.com>,
    License: GPL-2

    License: GPL-2
     This program is free software; you can redistribute it and/or modify
     it under the terms of the GNU General Public License as published by
     the Free Software Foundation; version 2 dated June, 1991.
     .
     On Debian systems, the complete text of version 2 of the GNU General
     Public License can be found in '/usr/share/common-licenses/GPL-2'.

### Plik debian/rules

Sednem całej zabawy z pakietowaniem modułów DKMS jest odpowiednie uzupełnienie pliku
`debian/rules` . Poniżej jest przykładowa zawartość:

    #!/usr/bin/make -f
    # -*- makefile -*-

    export DH_VERBOSE=1
    export DH_OPTIONS = -v
    export DEB_BUILD_MAINT_OPTIONS = hardening=+all
    DPKG_EXPORT_BUILDFLAGS = 1
    include /usr/share/dpkg/buildflags.mk
    include /usr/share/dpkg/pkg-info.mk

    %:
        dh $@ --with dkms

    override_dh_install:
        dh_install -p tpe-dkms scripts/ usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p tpe-dkms conf/ usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p tpe-dkms README usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p tpe-dkms Makefile usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p tpe-dkms *.c usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)
        dh_install -p tpe-dkms *.h usr/src/$(DEB_SOURCE)-$(DEB_VERSION_UPSTREAM)

    override_dh_dkms:
        dh_dkms -V $(DEB_VERSION_UPSTREAM)

    override_dh_auto_configure:
    override_dh_auto_build:
    override_dh_auto_test:
    override_dh_auto_install:
    override_dh_auto_clean:

Jako, że budujemy moduł DKMS, to określamy `--with dkms` . Na samym dole zaś mamy kilka pozycji
`override_dh_auto_*` . Wszystkie z nich są puste i nadpisują odpowiadające im targety. Ma to na
celu uniemożliwienie budowania modułu podczas tworzenia pakietu --  w końcu nie chcemy budować
samego kodu, a jedynie przygotować paczkę, która przy instalacji zbuduje moduł na docelowej
maszynie.

Przy pomocy `override_dh_install:` przepisujemy domyślny target instalacji plików. Musimy w nim
skopiować wszystkie pliki źródłowe wykorzystywane w procesie budowania modułu kernela. Zwykle są
to pliki mające sufiks `*.c` i `*.h` ale mogą być również i inne. Wszystkie te pliki trzeba
umieścić w odpowiednim katalogu pod `/usr/src/` . W zasadzie to edycja tego targetu w przypadku
różnych modułów kernela sprowadza się do określenia plików, które chcemy skopiować oraz zmiany
nazwy pakietu w `-p tpe-dkms` .

Z kolei `override_dh_dkms:` nadpisze nam target konfigurujący DKMS. W tym przypadku będzie
dostosowana wersja w pliku `dkms.conf` bez potrzeby ręcznej edycji tego pliku. Plik `dkms.conf`
musimy za to sobie stworzyć ręcznie.

### Plik debian/tpe-dkms.dkms

Wspomniany wyżej plik `dkms.conf` będzie po części generowany ale musimy i tak stworzyć jego
szkielet. Nazwa pliku `debian/tpe-dkms.dkms` odpowiada temu co wpisaliśmy w polu `Package:` w pliku
`control` plus sufiks `.dkms` . Poniżej znajduje się jego zawartość:

    PACKAGE_NAME="tpe"
    PACKAGE_VERSION="#MODULE_VERSION#"
    BUILT_MODULE_NAME[0]="$PACKAGE_NAME"
    MAKE[0]="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build"
    CLEAN="make -C ${kernel_source_dir} M=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build clean"
    DEST_MODULE_LOCATION[0]="/updates/dkms"
    REMAKE_INITRD="no"
    AUTOINSTALL="yes"

W zmiennej `$PACKAGE_NAME` podajemy nazwę modułu z pliku `Makefile` . W zmiennej `$PACKAGE_VERSION`
nie określamy wersji modułu. Będzie ona automatycznie generowana przez mechanizm budowy pakietu.
Pozostałe zmienne również zostawiamy bez zmian. Niekiedy będzie potrzeba dostosowania zmiennych
`$MAKE` i `$CLEAN` . Jeśli nasz moduł nie jest niezbędny w fazie initramfs/initrd, to nie musimy
ustawiać zmiennej `$REMAKE_INITRD` na `yes` , co przyśpieszy instalację modułu w systemie.

### Plik debian/watch

Przydałoby się także stworzyć plik `debian/watch` , który ma za zadanie monitorować czy upstream
wypuścił nowszą wersję modułu. Niemniej jednak, jeśli będziemy podbierać wersję z git'a, to trzeba
będzie synchronizować repozytorium z pominięciem pliku `debian/watch` . Tak czy inaczej poniżej
znajduje się jego zawartość:

    version=4
    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/tpe-lkm-$1\.tar\.gz/ \
     https://github.com/cormander/tpe-lkm/releases .*/archive/(\d\S*)\.tar\.gz

### Plik debian/source/format

Na koniec musimy jeszcze zmienić format źródeł z `3.0 (native)` na `3.0 (quilt)` . Edytujemy zatem
plik `debian/source/format` i przepisujemy go do poniższej postaci:

    3.0 (quilt)

### Paczka .tar z oryginalnymi źródłami

Musimy teraz stworzyć sobie paczkę `.tar` , w której będą przechowywane oryginalne źródła.
Pamiętajmy, że nie możemy po prostu skompresować katalogu roboczego, bo w nim znajdują się choćby
nasz katalog `debian/` oraz pliki od repozytorium git `.git/` . Tych wszystkich rzeczy trzeba się z
paczki `.tar` pozbyć. Robimy to w poniższy sposób:

    $ cd ..
    $ tar --exclude='.git*' --exclude='.pc' --exclude='debian' -cf - tpe-2.0.3 | xz -9 -c - > tpe_2.0.3~git20181202.orig.tar.xz

Kluczowe tutaj jest określenie nazwy paczki `.tar` , która musi pasować do wzoru
`nazwapaczki_wersja.orig.tar.xz` . Jeśli w pliku `debian/changelog` uwzględniliśmy także datę
ostatniego commit'a, to ją również trzeba uwzględnić w nazwie paczki `.tar` .

## Budowanie paczki .deb zawierającej źródła modułu kernela

Mając przygotowane pliki źródłowe, możemy przejść do stworzenia paczki `.deb` , która nam je ładnie
ogarnie. Budujemy pierw źródła:

    $ cd tpe-2.0.3
    $ debuild -S -sa -d -i

I budujemy pakiet `.deb` :

    $ cd ..
    $ sudo pbuilder --build tpe_2.0.3\~git20181202-1.dsc

Z racji, że nie będziemy nic kompilować, a jedynie zainstalujemy trochę zależności, to proces
budowania paczki `.deb` zajmie jedynie chwilę. Po paru minutach paczka powinna być gotowa do
instalacji w docelowym systemie.

## Instalacja modułu kernela przy pomocy DKMS w systemie

Jeśli wszystkie kroki do tej pory przeprowadziliśmy poprawnie, to możemy przejść do etapu
instalacji paczki `.deb` w systemie. Mając lokalne repozytorium stworzone za sprawą `reprepro`
dodajemy pierw paczkę do niego:

    $ debsign /media/Kabi/pbuilder/result/tpe_2.0.3\~git20181202-1_amd64.changes
    $ reprepro --basedir /media/Kabi/repozytorium/debian/ include sid /media/Kabi/pbuilder/result/tpe_2.0.3\~git20181202-1_amd64.changes.

Odświeżamy listę pakietów w repozytorium i instalujemy paczkę `tpe-dkms` :

    # apt-get update
    # aptitude install tpe-dkms
    The following NEW packages will be installed:
      tpe-dkms
    0 packages upgraded, 1 newly installed, 0 to remove and 112 not upgraded.
    Need to get 0 B/17.3 kB of archives. After unpacking 75.8 kB will be used.
    Get: 1 file:/media/Kabi/repozytorium/debian sid/main amd64 tpe-dkms all 2.0.3~git20181202-1 [17.3 kB]
    Retrieving bug reports... Done
    Parsing Found/Fixed information... Done
    Selecting previously unselected package tpe-dkms.
    (Reading database ... 325463 files and directories currently installed.)
    Preparing to unpack .../tpe-dkms_2.0.3~git20181202-1_all.deb ...
    Unpacking tpe-dkms (2.0.3~git20181202-1) ...
    Setting up tpe-dkms (2.0.3~git20181202-1) ...
    ...

Po chwili powinien rozpocząć się automatyczny proces budowania modułu kernela za sprawą mechanizmu
DKMS. W zasadzie to wszystkie kroki, które w tym procesie mają miejsce są dokładnie takie same jak
przy [ręcznej budowie modułu kernela via DKMS](/post/dkms-czyli-automatycznie-budowane-moduly/),
z tym, że to system bez naszej ingerencji przeprowadzi cały ten proces:

    ...
    Loading new tpe-2.0.3~git20181202 DKMS files...
    Building for 4.20.16-amd64-morficzny+
    Building initial module for 4.20.16-amd64-morficzny+
    Done.

    tpe.ko:
    Running module version sanity check.
     - Original module
       - No original module exists within this kernel
     - Installation
       - Installing to /lib/modules/4.20.16-amd64-morficzny+/updates/dkms/

    do_depmod 4.20.16-amd64-morficzny+
    DKMS: install completed.

Proces budowy modułu się zakończył, podobnie jak i proces instalacji pakietu. Niektóre moduły będą
się budować dłużej, a inne krócej, przez co cała instalacja będzie sprawiać wrażenie, że nic się
nie kompilowało. Na wszelki wypadek można jeszcze sprawdzić status zainstalowanych w systemie
modułów DKMS:

    # dkms status
    tpe, 2.0.3~git20181202, 4.20.16-amd64-morficzny+, x86_64: installed

Informacja na końcu, tj. `installed` oznacza, że moduł został dodany do mechanizmu DKMS, zbudowany
i zainstalowany przez niego, co potwierdza wyjście polecenia `modinfo` :

    # modinfo tpe
    filename:       /lib/modules/4.20.16-amd64-morficzny+/updates/dkms/tpe.ko
    version:        2.0.3
    description:    Trusted Path Execution (TPE) Module
    license:        GPL v2
    author:         Corey Henderson
    srcversion:     59FE85B47CF025933F0DEC7
    depends:
    retpoline:      Y
    name:           tpe
    vermagic:       4.20.16-amd64-morficzny+ SMP preempt modversions RANDSTRUCT_PLUGIN_0eaedf097c03a5950fbbc3dbbc78d213cd5e0408fdc9f6643749049983af6ba9
    sig_id:         PKCS#7
    signer:         Key for morfikownias kernel
    sig_key:        08:D5:CC:EF:11:AF:1F:63:0C:A6:C7:2A:B2:DB:71:A7:F0:41:DF:24
    sig_hashalgo:   sha512
    signature:      3A:BB:B3:40:07:35:59:25:30:B0:89:53:01:94:B1:DC:36:04:90:60:
                    14:91:8B:5B:67:25:CD:03:62:B3:19:6F:5F:3D:9F:EF:15:1E:AA:23:
                    25:6D:A6:73:E1:FC:4F:7B:1C:65:34:82:35:CC:F7:9D:71:FF:0A:08:
                    65:E9:F5:79:33:52:00:0C:BB:CC:5A:3F:64:97:5D:62:5C:D2:76:61:
                    2B:95:1D:0C:67:A7:59:89:4A:D4:14:A4:54:D5:DB:5A:E1:06:C0:98:
                    B5:26:67:46:D6:C3:ED:D1:1E:89:3D:3E:4B:7D:D3:D9:23:97:8A:75:
                    E4:91:36:9A:D1:8E:50:57:3C:84:40:9F:7A:2A:C7:D1:61:74:D8:1D:
                    79:9D:9D:3B:D0:13:30:8F:9F:86:6D:B9:C3:4B:1A:B5:7E:84:B9:48:
                    5B:A3:4C:5C:78:E5:45:A5:BB:84:E8:DC:20:A5:F2:A1:E7:6D:E2:A4:
                    33:ED:CA:92:B5:74:B6:0D:09:1C:A2:8C:2F:FE:A6:A4:8D:AD:2A:99:
                    7D:B1:F9:FF:83:B3:A0:A2:3A:50:29:D7:B5:B9:C8:36:30:10:CA:98:
                    43:1B:1E:18:52:76:50:F3:1B:60:15:D1:82:6E:B8:58:1C:43:4C:71:
                    BD:C6:CF:1F:62:6F:5A:65:4B:D4:62:56:AC:32:36:44:65:BF:22:FE:
                    E3:24:36:31:EC:14:99:18:60:54:42:CF:76:FF:5B:09:7B:6F:15:49:
                    9A:1A:30:B9:B4:C5:98:78:19:1D:BB:02:09:24:C9:B8:44:66:A0:87:
                    E8:87:85:F3:3A:CA:40:72:70:65:01:0C:46:88:77:F2:8F:4D:5C:D4:
                    7A:01:2A:16:20:68:97:18:07:80:50:B3:9D:1D:A4:94:76:11:4A:AB:
                    30:33:87:8E:EE:4C:75:86:86:F8:34:A7:B4:64:EF:02:09:C9:3B:3A:
                    AB:AF:AD:DA:36:C5:B7:14:2B:5B:9B:43:73:6A:F8:43:BF:AB:BF:3F:
                    CC:56:52:F9:E2:18:D9:A2:F9:75:EC:8A:17:9E:D5:BC:36:FB:4C:9D:
                    6B:96:BE:8E:3F:FB:2C:0C:06:82:5D:8B:87:A5:CB:5C:90:EF:98:52:
                    11:9C:ED:E5:90:CD:F3:6C:2D:74:1F:C3:A6:AF:AB:8F:B7:1D:0F:2C:
                    BF:8D:85:5C:6D:EE:59:5F:66:07:C5:71:37:A4:86:69:A7:71:18:63:
                    C4:0B:69:C3:88:03:62:15:03:BA:28:FE:79:A4:85:64:DC:C4:18:04:
                    C1:26:61:E6:FF:BA:72:86:45:AD:D3:C6:80:E1:9E:9E:20:83:7B:72:
                    FB:7E:F1:90:64:AC:3F:93:48:ED:14:47

No i jak widać, moduł `tpe.ko` siedzi na swoim miejscu, zatem cały proces przygotowania pakietu
`.deb` z modułem kernela zakończył się powodzeniem. Podobnie sprawa będzie wyglądać, gdy
zaktualizujemy paczkę, czy też będziemy instalować nowszy kernel.

Warto też zadbać o to, by moduł instalowany przy pomocy mechanizmu DKMS ładował się przy starcie
systemu tworząc stosowną konfigurację w katalogach `/etc/modules-load.d/` oraz `/etc/modprobe.d/` .

## Podpisywanie modułów DKMS

Jeśli nasz kernel wykorzystuje podpisy cyfrowe i nie zezwala na załadowanie modułów, które nie
zostaną podpisane tym samym kluczem co kernel, to trzeba również zadbać
o [podpisanie modułów tworzonych za sprawą mechanizmu DKMS](/post/automatyczne-podpisywanie-modulow-kernela-przez-dkms/).
