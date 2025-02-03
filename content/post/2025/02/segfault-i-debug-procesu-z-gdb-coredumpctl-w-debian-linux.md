---
author: Morfik
categories:
- Linux
date:    2025-02-02 23:25:00 +0100
lastmod: 2025-02-02 23:25:00 +0100
published: true
status: publish
tags:
- debian
- gdb
- backtrace
- debug
- segfault
- crash
- coredump
GHissueID: 603
title: Segfault i debug procesu z GDB/coredumpctl w Debian Linux
---

Każdy z nas korzysta z całej masy aplikacji na swoich linux'ach. Na ogół te appki działają w sposób
oczekiwany i nie ma z nimi większych problemów. Czasem jednak zdarza się tak, że z jakiegoś powodu
taki program niespodziewanie kończy swoje działanie i w konsoli (czy logu systemowym) zostaje
wydrukowany komunikat zawierający frazę `segfault` (segmentation fault, naruszenie ochrony pamięci).
Ostatnio taki problem dotknął jedną z moich ulubionych aplikacji, z których korzystam na co dzień,
tj. `strawberry` . Gdyby ta sytuacja wystąpiła raz, to pewnie nawet bym się nią bardziej nie
zainteresował ale w tym przypadku ten cały `segfault` potrafił wyskoczyć kilka-kilkanaście razy w
ciągu godziny, co trochę zaczęło mnie irytować. Postanowiłem poszukać przyczyny i nawet [znalazłem
stosowny bug na git projektu strawberry][1], no ale sytuacja niby nowa, a mi ta appka crash'owała
od dobrych kilku tygodni, jeśli nie miesięcy. Dlatego też zacząłem udzielać się w tamtym wątku i
przy okazji nauczyłem się jak przy pomocy `gdb` (GNU Debugger) pozyskać nieco bardziej użyteczne
informacje i podesłać je deweloperowi aplikacji, tak by był on w stanie namierzyć przyczynę i
wyeliminować zaistniały problem. W tym artykule została zebrana garść użytecznych informacji, które
mogą przydać się użytkownikowi Debiana, na wypadek gdyby i jego aplikacje cierpiały na podobne
problemy.

<!--more-->

## Debugger GNU (gdb) oraz symbole debugowania

Przede wszystkim, by móc mówić o debugowaniu aplikacji, musimy zaopatrzyć się w odpowiednie
narzędzia. Pierwszym z nim jest `gdb` , tj. [Debugger GNU][2]. W Debianie ten debugger znajduje się
w pakiecie `gdb` i raczej nie powinno być problemów z jego instalacją w systemie.

Drugą rzeczą, która będzie nam potrzebna, to symbole debugowania (debugging symbols). Każda
aplikacja obecna w repozytoriach Debiana posiada takie symbole. By mieć do nich dostęp, trzeba
dodać [archiwum AutomaticDebugPackages][3] do pliku `/etc/apt/sources.list` . W zależności z
jakiego wydania Debiana korzystamy, to ten wpis będzie się nieco różnił. Poniżej jest przykład
dla `sid` i `experimental` :

    deb https://deb.debian.org/debian-debug/ sid-debug main
    deb https://deb.debian.org/debian-debug/ experimental-debug main

Posiadając odpowiednie wpisy w pliku `/etc/apt/sources.list` musimy pobrać nie tylko symbole dla
aplikacji, która wyrzuca nam `segfault` ale też dla wszystkich bibliotek, z których ten program
korzysta. W przeszłości z wyszukaniem tych dodatkowych pakietów mógł być problem ale obecnie mamy
do dyspozycji narzędzie `find-dbgsym-packages` z pakietu `debian-goodies` , który jest nam w stanie
te wszystkie pakiety bez większego problemu pomóc ustalić. Poniżej przykład dla `strawberry`

    # find-dbgsym-packages /usr/bin/strawberry

    libasound2t64-dbgsym libasyncns0-dbgsym libatomic1-dbgsym libb2-1-dbgsym libblkid1-dbgsym
    libbrotli1-dbgsym libbz2-1.0-dbgsym libcap2-dbgsym libcdio19t64-dbgsym libchromaprint1-dbgsym
    libcom-err2-dbgsym libcurl3t64-gnutls-dbgsym libdbus-1-3-dbgsym libdouble-conversion3-dbgsym
    libduktape207-dbgsym libdw1t64-dbgsym libebur128-1-dbgsym libegl1-dbgsym libelf1t64-dbgsym
    libexpat1-dbgsym libffi8-dbgsym libfftw3-double3-dbgsym libflac12t64-dbgsym
    libfontconfig1-dbgsym libfreetype6-dbgsym libgcc-s1-dbgsym libgcrypt20-dbgsym
    libgdk-pixbuf-2.0-0-dbgsym libglib2.0-0t64-dbgsym libglvnd0-dbgsym libglx0-dbgsym
    libgmp10-dbgsym libgnutls30t64-dbgsym libgomp1-dbgsym libgpg-error0-dbgsym libgpod4t64-dbgsym
    libgraphite2-3-dbgsym libgstreamer-plugins-base1.0-0-dbgsym libgstreamer1.0-0-dbgsym
    libharfbuzz0b-dbgsym libhogweed6t64-dbgsym libicu72-dbgsym libidn2-0-dbgsym
    libimobiledevice-1.0-6-dbgsym libimobiledevice-glue-1.0-0-dbgsym libjpeg62-turbo-dbgsym
    libkeyutils1-dbgsym libkrb5-dbg libldap-2.5-0-dbgsym liblzma5-dbgsym libmd4c0-dbgsym
    libmount1-dbgsym libmp3lame0-dbgsym libmpg123-0t64-dbgsym libmtp9t64-dbgsym
    libnettle8t64-dbgsym libnghttp2-14-dbgsym libnghttp3-9-dbgsym libngtcp2-16-dbgsym
    libngtcp2-crypto-gnutls8-dbgsym libogg0-dbgsym libopengl0-dbgsym libopus0-dbgsym
    liborc-0.4-0t64-dbgsym libp11-kit0-dbgsym libpcre2-16-0-dbgsym libpcre2-8-0-dbgsym
    libplist-2.0-4-dbgsym libpng16-16t64-dbgsym libproxy1v5-dbgsym libpsl5t64-dbgsym
    libpulse0-dbgsym libqt6concurrent6-dbgsym libqt6core6t64-dbgsym libqt6dbus6-dbgsym
    libqt6gui6-dbgsym libqt6network6-dbgsym libqt6sql6-dbgsym libqt6widgets6-dbgsym librtmp1-dbgsym
    libsasl2-2-dbgsym libselinux1-dbgsym libsndfile1-dbgsym libsqlite3-0-dbgsym libssh2-1t64-dbgsym
    libssl3t64-dbgsym libstdc++6-dbgsym libsystemd0-dbgsym libtag2-dbgsym libtasn1-6-dbgsym
    libudev1-dbgsym libunistring5-dbgsym libunwind8-dbgsym libusb-1.0-0-dbgsym
    libusbmuxd-2.0-7-dbgsym libvorbis0a-dbgsym libvorbisenc2-dbgsym libx11-6-dbgsym
    libx11-xcb1-dbgsym libxau6-dbgsym libxcb1-dbgsym libxdmcp6-dbgsym libxkbcommon0-dbgsym
    libxml2-dbgsym libzstd1-dbgsym zlib1g-dbgsym

No jakby nie patrzeć trochę tego jest.

Wszystko co zostało wydrukowane wyżej przez `find-dbgsym-packages` trzeba zainstalować przez
menadżer pakietów `apt` / `aptitude` . W przypadku gdybyśmy zainstalowali jedynie sam pakiet
`strawberry-dbgsym` , to wynik zwracany przez `gdb` w sporej części zawierałby jedynie same adresy
pamięci, a nie o to nam chodzi. Poniżej przykład:

    ...
	Thread 1 "strawberry" received signal SIGSEGV, Segmentation fault.
	0x00007b9659bb209f in ?? ()
	(gdb) bt
	#0  0x00007b9659bb209f in ?? ()
	#1  0x00005b3cc60ddab0 in ?? ()
	#2  0x00005b3cc61b7850 in ?? ()
	#3  0x00005b3cc60ddab0 in ?? ()
	#4  0x00005b3cc2c47950 in ?? ()
	#5  0x00005b3c845a6b90 in stdout ()
	#6  0x00007b9657b87928 in ?? ()
	#7  0x0000003000000028 in ?? ()
	#8  0x00007ffd30b448a0 in ?? ()
	#9  0x00005b3cc61b7850 in ?? ()
	#10 0x00005b3cc60ddab0 in ?? ()
	#11 0x00007ffd30b447cf in ?? ()
	#12 0xf8062d8205732a00 in ?? ()
	#13 0x00005b3cc2c47950 in ?? ()
	#14 0x00005b3cc60ddab0 in ?? ()
	#15 0x0000000000000000 in ?? ()

No z tych powyższych informacji to za wiele się wyczytać nie da.

### Automatycznie pobieranie symboli debugowania przez internet

Istnieje również [metoda pozyskiwania symboli debugowania][5] bezpośrednio przez internet w
procesie debugowania aplikacji. Wystarczy doinstalować pakiet `debuginfod` i wyeksportować w
terminalu zmienną `DEBUGINFOD_URLS` , przykładowo:

    export DEBUGINFOD_URLS="https://debuginfod.debian.net"

I teraz już tylko wystarczy odpalić `gdb` :

    $ gdb strawberry
    GNU gdb (Debian 15.2-1+b1) 15.2
    ...
    Enable debuginfod for this session? (y or [n]) y
    Debuginfod has been enabled.
    To make this setting permanent, add 'set debuginfod enabled on' to .gdbinit.
    Downloading 34.41 M separate debug info for /usr/bin/strawberry
    [##########       ]   63% (64.41 M)
    Reading symbols from /home/morfik/.cache/debuginfod_client/da5aa053968f350dd1dbaefaf2a2dbbf8bb9565b/debuginfo...
    (gdb)

Jak widać wyżej, stosowne symbole debugowania zostały pobrane i umieszczone w katalogu
`/home/morfik/.cache/debuginfod_client/` i `gdb` powinien z nich zrobić użytek.

## Konfiguracja kernela

W dystrybucyjnych kernelach Debiana wszystkie stosowne opcje są już włączone i w zasadzie nie
trzeba nic ruszać. Niemniej jednak, jeśli [sami budujemy własne jądro][4], to pamiętajmy by włączyć
w konfiguracji kernela `CONFIG_COREDUMP` .

## Konfiguracja systemd i sysctl

W przypadku korzystania z systemd, przydałoby się też doinstalować dodatkowo
pakiet `systemd-coredump` . Jest on opcjonalny jeśli chcemy korzystać manualnie z `gdb` ale może
się okazać iście użyteczny. Podczas instalacji pakietu `systemd-coredump` , system skonfiguruje nam
też te poniższe parametry `sysctl` :

    kernel.core_pattern=|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h
    kernel.core_pipe_limit=16
    fs.suid_dumpable=2

Skoro już mowa o pliku `/etc/sysctl.conf` , to jeśli posiadamy w nim jakieś ustawienia poprawiające
bezpieczeństwo systemu, to upewnijmy się, że `kernel.yama.ptrace_scope` jest ustawiony na `0` na
czas debugowania aplikacji:

    kernel.yama.ptrace_scope = 0

## Backtrace procesu

W przypadku korzystania z systemd, nie musimy nic więcej robić, tj. uruchamiamy sobie naszą
aplikację, tak jak to zwykle robimy i czekamy, aż zaliczy crash. Gdy to nastąpi, to przy pomocy
narzędzia `coredumpctl` patrzymy co się wydarzyło:

    # coredumpctl list
    TIME                          PID  UID  GID SIG     COREFILE EXE                   SIZE
    Mon 2025-01-13 21:42:41 CET 17440 1000 1000 SIGSEGV present  /usr/bin/strawberry  49.7M
    Mon 2025-01-13 21:51:24 CET 19897 1000 1000 SIGSEGV present  /usr/bin/strawberry  41.6M
    Mon 2025-01-13 22:54:37 CET 23453 1000 1000 SIGSEGV present  /usr/bin/strawberry 133.3M

Widzimy tutaj crash'e procesu `/usr/bin/strawberry` . By podejrzeć którykolwiek z nich, w
argumencie `coredumpctl` podajemy kolejno nazwę debugger'a oraz numerek coredump'a. Przykładowo,
jeśli chcemy podejrzeć drugą pozycję widoczną wyżej, to w terminalu wpisujemy:

    $ coredumpctl gdb -2
               PID: 19897 (strawberry)
               UID: 1000 (morfik)
               GID: 1000 (morfik)
            Signal: 11 (SEGV)
         Timestamp: Mon 2025-01-13 21:51:20 CET (2min 57s ago)
      Command Line: strawberry
        Executable: /usr/bin/strawberry
     Control Group: /morfikownia.slice/libcgroup.scope/apps-user/strawberry
              Unit: libcgroup.scope
             Slice: morfikownia.slice
           Boot ID: 9b3e8d11480f41b0bff7c41b8220cf9a
        Machine ID: c59721c157404b63a846368d354b7ccf
          Hostname: morfikownia
           Storage: /var/lib/systemd/coredump/core.strawberry.1000.9b3e8d11480f41b0bff7c41b8220cf9a.19897.1736801480000000.zst (present)
      Size on Disk: 41.6M
           Message: Process 19897 (strawberry) of user 1000 dumped core.

                    Module libuuid.so.1 from deb util-linux-2.40.3-1.amd64
                    Module libgomp.so.1 from deb gcc-14-14.2.0-13.amd64
                    Module libudev.so.1 from deb systemd-257.2-1.amd64
                    Module libblkid.so.1 from deb util-linux-2.40.3-1.amd64
                    Module libsystemd.so.0 from deb systemd-257.2-1.amd64
                    Module libzstd.so.1 from deb libzstd-1.5.6+dfsg-2.amd64
                    Module libatomic.so.1 from deb gcc-14-14.2.0-13.amd64
                    Module libmount.so.1 from deb util-linux-2.40.3-1.amd64
                    Module libgcc_s.so.1 from deb gcc-14-14.2.0-13.amd64
                    Module libstdc++.so.6 from deb gcc-14-14.2.0-13.amd64
                    Stack trace of thread 19897:
                    ...
                    #4  0x00007400b098e897 postEventSourceDispatch (libQt6Core.so.6 + 0x38e897)
                    #5  0x00007400b0eb381f g_main_dispatch (libglib-2.0.so.0 + 0x5a81f)
                    ...
                    ELF object binary architecture: AMD x86-64

    GNU gdb (Debian 15.2-1+b1) 15.2
    ...
    Reading symbols from /usr/bin/strawberry...
    Reading symbols from /usr/lib/debug/.build-id/e0/0d220ad54e9112741ff9ace5286a6401d8c96d.debug...
    ...
    [New LWP 19897]
    ...
    [New LWP 19910]
    ...
    [New LWP 20039]
    ...
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
    Core was generated by `strawberry'.
    Program terminated with signal SIGSEGV, Segmentation fault.

    #0  0x00005bf45f01ee52 in QtPrivate::FunctorCall<QtPrivate::IndexesList<>, QtPrivate::List<>, void, void (TagReaderReply::*)()>::call(void (TagReaderReply::*)(), TagReaderReply*, void**) (arg=<optimized out>, f=&virtual table offset 104, o=<optimized out>)
        at /usr/include/x86_64-linux-gnu/qt6/QtCore/qobjectdefs_impl.h:118
    118         template<typename Obj> inline void assertObjectType(QObject *o)
    [Current thread is 1 (Thread 0x7400ab86ad00 (LWP 19897))]

Powyższe wyjście zostało trochę okrojone dla czytelności.

W tym przypadku, deweloper projektu `strawberry` poprosił mnie o wydanie dwóch poleceń w obrębie debugger'a `gdb` , tj. `bt` oraz `thread apply all bt` :

Poniżej jest polecenie `bt` :

    (gdb) bt

    #0  0x00005bf45f01ee52 in QtPrivate::FunctorCall<QtPrivate::IndexesList<>, QtPrivate::List<>, void, void (TagReaderReply::*)()>::call(void (TagReaderReply::*)(), TagReaderReply*, void**) (arg=<optimized out>, f=&virtual table offset 104, o=<optimized out>)
        at /usr/include/x86_64-linux-gnu/qt6/QtCore/qobjectdefs_impl.h:118
    #1  QtPrivate::FunctionPointer<void (TagReaderReply::*)()>::call<QtPrivate::List<>, void>(void (TagReaderReply::*)(), TagReaderReply*, void**) (f=&virtual table offset 104, o=<optimized out>, arg=<optimized out>)
        at /usr/include/x86_64-linux-gnu/qt6/QtCore/qobjectdefs_impl.h:182
    ...
    #15 0x00007400b078a908 in QCoreApplication::exec () at ./src/corelib/global/qflags.h:74
    #16 0x00005bf45ef1f220 in main (argc=<optimized out>, argv=<optimized out>) at ./src/main.cpp:330

A niżej polecenie `thread apply all bt` :

    (gdb) thread apply all bt

    Thread 33 (Thread 0x7400367fc6c0 (LWP 20139)):
    #0  syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38
    #1  0x00007400b0ee5c04 in g_cond_wait_impl (cond=0x5bf4640ff420, mutex=0x5bf4640ff3d8) at ../../../glib/gthread-posix.c:1007

    ...

    #6  0x00007400b009c043 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:447
    #7  0x00007400b011a778 in __GI___clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:78

    Thread 32 (Thread 0x74000dffb6c0 (LWP 20141)):
    #0  0x00007400b00989de in __futex_abstimed_wait_common64 (private=0, futex_word=0x5bf4641e3440, expected=0, op=137, abstime=0x74000dffa780, cancel=true) at ./nptl/futex-internal.c:57
    #1  __futex_abstimed_wait_common (futex_word=futex_word@entry=0x5bf4641e3440, expected=expected@entry=0, clockid=clockid@entry=1, abstime=abstime@entry=0x74000dffa780, private=private@entry=0, cancel=cancel@entry=true) at ./nptl/futex-internal.c:87

    ...

    #12 0x00007400b009c043 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:447
    --Type <RET> for more, q to quit, c to continue without paging--c
    #13 0x00007400b011a778 in __GI___clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:78

    ...
    ...
    ...

    Thread 2 (Thread 0x7400377fe6c0 (LWP 20026)):
    #0  futex_wait (futex_word=0x74002c02f880, expected=2, private=0) at ../sysdeps/nptl/futex-internal.h:146
    #1  __GI___lll_lock_wait (futex=futex@entry=0x74002c02f880, private=0) at ./nptl/lowlevellock.c:49

    ...

    #23 0x00007400b009c043 in start_thread (arg=<optimized out>) at ./nptl/pthread_create.c:447
    #24 0x00007400b011a778 in __GI___clone3 () at ../sysdeps/unix/sysv/linux/x86_64/clone3.S:78

    Thread 1 (Thread 0x7400ab86ad00 (LWP 19897)):
    #0  0x00005bf45f01ee52 in QtPrivate::FunctorCall<QtPrivate::IndexesList<>, QtPrivate::List<>, void, void (TagReaderReply::*)()>::call(void (TagReaderReply::*)(), TagReaderReply*, void**) (arg=<optimized out>, f=&virtual table offset 104, o=<optimized out>) at /usr/include/x86_64-linux-gnu/qt6/QtCore/qobjectdefs_impl.h:118
    #1  QtPrivate::FunctionPointer<void (TagReaderReply::*)()>::call<QtPrivate::List<>, void>(void (TagReaderReply::*)(), TagReaderReply*, void**) (f=&virtual table offset 104, o=<optimized out>, arg=<optimized out>) at /usr/include/x86_64-linux-gnu/qt6/QtCore/qobjectdefs_impl.h:182

    ...

    #15 0x00007400b078a908 in QCoreApplication::exec () at ./src/corelib/global/qflags.h:74
    #16 0x00005bf45ef1f220 in main (argc=<optimized out>, argv=<optimized out>) at ./src/main.cpp:330

Powyższe wyjścia obu poleceń zostały przycięte dla czytelności. Tak czy inaczej to właśnie takiej
mniej więcej informacji będzie oczekiwał od nas deweloper projektu, którego crash zaobserwujemy i
który chcemy zgłosić. W tym przypadku [szybko udało się ustalić przyczynę i kod aplikacji został
odpowiednio poprawiony][7]. No i co najważniejsze, `strawberry` już nie wyrzuca segfault'ów.

## Czyszczenie plików coredump

Od czasu do czasu przydałoby się przeczyścić stare pliki coredump, które automatycznie systemd
zapisuje w katalogu `/var/lib/systemd/coredump/` . Możemy naturalnie ręcznie usunąć całą zawartość
tego katalogu (albo jedynie pojedyncze pliki), tylko w takim przypadku wywołanie
polecenia `coredumpctl list` zwróci nam `missing` w kolumnie `COREFILE` , przykładowo:

    # rm /var/lib/systemd/coredump/*.zst

    # coredumpctl list
    TIME                          PID  UID  GID SIG     COREFILE EXE               SIZE
    Sat 2025-01-18 10:53:54 CET 65549 1000 1000 SIGABRT missing  /usr/bin/birdtray    -
    Sat 2025-01-18 10:55:56 CET 65854 1000 1000 SIGABRT missing  /usr/bin/birdtray    -
    Sat 2025-01-18 10:57:49 CET 66119 1000 1000 SIGABRT missing  /usr/bin/birdtray    -

I jak widzimy wyżej, niby usunęliśmy pliki coredump powiązane z procesem `birdtray` ale wpisy na
liście dalej występują. Jak je usunąć? No trzeba się pierw zastanowić skąd te wpisy na listingu
powyżej się biorą. [Biorą się one z journal'a systemd][6]. Zatem nawet jeśli usuniemy plik coredump,
to wpisy o zdarzeniu dalej są obecne w dzienniku systemowym i narzędzie `coredumpctl` je widzi.
Trzeba by przeczyścić dziennik i do tego celu możemy posłużyć się poniższym poleceniem:

    # journalctl --rotate && journalctl --vacuum-time=1s
    Vacuuming done, freed 2.7M of archived journals from /var/log/journal/b52721c15af44b63a846368d354b7aad.
    Vacuuming done, freed 0B of archived journals from /var/log/journal.

Opcja `--rotate` ma za zadanie wymusić otwarcie nowego pliku z logami oraz oznaczenie wszystkich
poprzednich jako archiwalne. Z kolei opcja `--vacuum-time` usuwa pliki archiwalne starsze niż czas
określony w tym parametrze. My tutaj ustawiliśmy jedną sekundę, bo chcieliśmy wyczyścić całkowicie
dziennik. Jeśli teraz wywołamy polecenie `coredumpctl list` , to już żadne wpisy nie zostaną nam
zwrócone:

    # coredumpctl list
    No coredumps found.

Trzeba tutaj tylko pamiętać o tym, że jak wyczyścimy dziennik (wyżej opisaną metodą), to pliki
coredump nie są w żaden sposób ruszane, przez co `coredumpctl list` zwróci nam pusty wynik mimo
obecności tych plików na dysku.

## Konfiguracja usługi systemd-coredump

Jeśli nie chcemy by systemd zapisywał pliki coredump na dysku, możemy go poprosić by tego nie robił.
W tym celu edytujemy plik `/etc/systemd/coredump.conf` i przestawiamy `Storage` na `none` :

    Storage=none

Od tego momentu pliki nie będą już zapisywane w katalogu `/var/lib/systemd/coredump/` , zaś
`coredumpctl list` zwróci nam `none` w kolumnie `COREFILE`

    # coredumpctl list
    TIME                          PID  UID  GID SIG     COREFILE EXE               SIZE
    Sat 2025-01-18 11:10:53 CET 66309 1000 1000 SIGABRT none     /usr/bin/birdtray    -

Niemniej jednak, w dalszym ciągu usługa `systemd-coredump` przetworzy nam taki wygenerowany
coredump i zapisze w logu systemowym.

    systemd-coredump[67188]: Process 66309 (birdtray) of user 1000 terminated abnormally with signal 6/ABRT, processing...
    ystemd[1]: Started systemd-coredump@5-67188-0.service - Process Core Dump (PID 67188/UID 0).
    systemd-coredump[67189]: Process 66309 (birdtray) of user 1000 dumped core.

             Module libblkid.so.1 from deb util-linux-2.40.4-1.amd64
             Module libmount.so.1 from deb util-linux-2.40.4-1.amd64
             ....
             Stack trace of thread 66321:
             #0  0x000075eeb709dd8c __pthread_kill_implementation (libc.so.6 + 0x93d8c)
             ...

Jeśli i to zachowanie jest niepożądane, to w pliku `/etc/systemd/coredump.conf` dopiszmy sobie
jeszcze parametr `ProcessSizeMax` :

    ProcessSizeMax=0

I teraz w logu systemowym powinniśmy mieć jedynie informację o tym, że proces zakończył się w
sposób inny niż oczekiwany:

    systemd-coredump[67630]: Process 67214 (birdtray) of user 1000 terminated abnormally with signal 6/ABRT, processing...
    systemd[1]: Started systemd-coredump@6-67630-0.service - Process Core Dump (PID 67630/UID 0).
    systemd-coredump[67631]: Process 67214 (birdtray) of user 1000 terminated abnormally without generating a coredump.
    systemd[1]: systemd-coredump@6-67630-0.service: Deactivated successfully.

Taka informacja mówi nam jedynie, że coś w systemie dzieje się nie tak, a podglądając wyjście
polecenia `coredumpctl list` bardzo łatwo możemy ustalić z jakimi procesami i jak często:

    # coredumpctl list
    TIME                          PID  UID  GID SIG     COREFILE EXE               SIZE
    Sat 2025-01-18 11:10:53 CET 66309 1000 1000 SIGABRT none     /usr/bin/birdtray    -
    Sat 2025-01-18 11:14:34 CET 67214 1000 1000 SIGABRT none     /usr/bin/birdtray    -
    Sat 2025-01-18 11:22:19 CET 67851 1000 1000 SIGABRT none     /usr/bin/birdtray    -

Chodzi tutaj generalnie o to, by te pliki coredump nie zapchały nad dysku twardego, bo potrafią one
swoje ważyć. Do tego jeśli korzystamy z nośnika SSD, to raczej nie chcemy marnować cykli życia
komórek flash na tego typu zapis danych.

## Podsumowanie

Posiadając odpowiednie narzędzia w swoich linux'ach jesteśmy w stanie w dość łatwy sposób namierzyć
problem dręczący działające w nim programy. Nawet niekoniecznie potrzebujemy fachowej wiedzy z
zakresu debugowania aplikacji, czy też sami być programistami ale mając na wyposażeniu te powyżej
opisane narzędzia możemy dostarczyć nieocenionych informacji deweloperom aplikacji pomagając im
zdiagnozować i wyeliminować problem.

[1]: https://github.com/strawberrymusicplayer/strawberry/issues/1633
[2]: https://www.gnu.org/software/gdb/
[3]: https://wiki.debian.org/AutomaticDebugPackages
[4]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[5]: https://wiki.debian.org/HowToGetABacktrace#Automatically_loading_debugging_symbols_from_the_Internet
[6]: https://www.freedesktop.org/software/systemd/man/latest/coredumpctl.html#Examples
[7]: https://github.com/strawberrymusicplayer/strawberry/commit/decd0a1dc617f0f561636b281f3f369468e21b5b
