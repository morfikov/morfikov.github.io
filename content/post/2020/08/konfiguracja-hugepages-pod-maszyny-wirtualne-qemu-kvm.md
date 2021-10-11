---
author: Morfik
categories:
- Linux
date: "2020-08-09T11:11:00Z"
published: true
status: publish
tags:
- debian
- wirtualizacja
- kvm
- qemu
- libvirt
- kernel
- hugepages
- sysctl
GHissueID: 33
title: Konfiguracja HugePages pod maszyny wirtualne QEMU/KVM
---

W linux rozmiar stron pamięci operacyjnej RAM ma domyślnie 4096 bajtów (4 KiB). Maszyny wirtualne
QEMU/KVM mają to do siebie, że wykorzystują dość spore zasoby pamięci (wile GiB), przez co mały
rozmiar strony może niekorzystnie wpływać na wydajność systemów gościa. Chodzi generalnie o to, że
rozrostowi ulega tablica stron, której przeszukiwanie jest czasochłonną operacją. By temu zaradzić,
wymyślono TLB ([Translation Lookaside Buffer][1]), który ulokowany jest albo w CPU albo gdzieś
pomiędzy CPU i główną pamięcią operacyjną. TLB to mały ale za to bardzo szybki cache. W przypadku
systemów z duża ilością pamięci RAM, niewielki rozmiar TLB sprawia, że odpowiedzi na zapytania nie
są brane z cache, tylko system wraca do przeszukiwania normalnej tablicy stron zlokalizowanej w
pamięci RAM (TLB miss). Taka sytuacja jest bardzo kosztowna, spowalnia cały system i dlatego trzeba
jej unikać. Na szczęście jest [mechanizm HugePages][2], który pozwala na zwiększenie rozmiaru
strony pamięci z domyślnych 4 KiB do 2 MiB lub nawet do 1 GiB w zależności od właściwości głównego
procesora. W tym artykule postaramy się skonfigurować HugePages na potrzeby maszyn wirtualnych dla
systemu Debian Linux.

<!--more-->
## HugePages 2 MiB czy 1 GiB

Z reguły nowsze procesory wspierają obecnie już HugePages. Pytanie tylko jaki rozmiar stron pamięci
jest w stanie obsłużyć CPU naszego komputera. Jeśli we flagach procesora (w pliku `/proc/cpuinfo` )
widnieje `pse` , to procesor naszego komputera ma wsparcie dla 2 MiB stron pamięci. Jeśli zaś jest
dostępna flaga `pdpe1gb` , to procesor wspiera dodatkowo strony pamięci o rozmiarze 1 GiB. Poniżej
przykład:

	# cat /proc/cpuinfo
	processor       : 0
	vendor_id       : GenuineIntel
	cpu family      : 6
	model           : 58
	model name      : Intel(R) Core(TM) i5-3320M CPU @ 2.60GHz
	stepping        : 9
	microcode       : 0x21
	cpu MHz         : 1842.833
	cache size      : 3072 KB
	physical id     : 0
	siblings        : 4
	core id         : 0
	cpu cores       : 2
	apicid          : 0
	initial apicid  : 0
	fpu             : yes
	fpu_exception   : yes
	cpuid level     : 13
	wp              : yes
	flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36
	                  clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm
	                  constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid
	                  aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr
	                  pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c
	                  rdrand lahf_lm cpuid_fault epb pti ssbd ibrs ibpb stibp tpr_shadow vnmi
	                  flexpriority ept vpid fsgsbase smep erms xsaveopt dtherm ida arat pln pts
	                  md_clear flush_l1d
	vmx flags       : vnmi preemption_timer invvpid ept_x_only flexpriority tsc_offset vtpr mtf
	                  vapic ept vpid unrestricted_guest
	bugs            : cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs
	                  itlb_multihit srbds
	bogomips        : 5188.07
	clflush size    : 64
	cache_alignment : 64
	address sizes   : 36 bits physical, 48 bits virtual
	power management:

Jak widać, procesor w moim ThinkPad T430 wspiera jedynie HugePages o rozmiarze 2 MiB ale to i tak
jest o wiele lepsze rozwiązanie niż korzystanie ze stron o rozmiarze 4 KiB w przypadku maszyn
wirtualnych.

By być w stanie zrobić użytek z większych stron pamięci, trzeba w konfiguracji kernela włączyć te
poniższe opcje:

    CONFIG_HUGETLBFS
    CONFIG_HUGETLB_PAGE
    CONFIG_TRANSPARENT_HUGEPAGE
    CONFIG_TRANSPARENT_HUGEPAGE_ALWAYS
    CONFIG_ARCH_WANT_GENERAL_HUGETLB

W dystrybucyjnych kernelach linux, te powyższe opcje powinny być standardowo włączone. Jeśli jednak
[samodzielnie budujemy kernel dla konkretnej maszyny][3], to upewnijmy się, że te powyższe
parametry są zaznaczone.

## Przygotowanie /dev/hugepages/

Konfiguracja HugePages w systemach na bazie systemd jest w zasadzie automatyczna. Mamy tutaj
dostępną usługę `dev-hugepages.mount` , która zajmuje się przygotowaniem punktu montowania
`/dev/hugepages/` podczas startu systemu. Jeśli chcemy zrobić użytek z HugePages przy wirtualizacji
na linux, to musimy w zasadzie zmienić jedynie uprawnienia nadawane katalogowi `/dev/hugepages/` .
Robimy to poprzez stworzenie pliku `/etc/systemd/system/dev-hugepages.mount.d/override.conf` , w
którym umieszczamy poniższą treść:

    [Mount]
    Options=mode=01770,gid=136

Wartość parametru `gid=` wskazuje na ID grupy `kvm` , który można wyciągnąć z pliku `/etc/group` :

    # cat /etc/group  | grep kvm
    kvm:x:136:

Przeładowujemy konfigurację systemd i restartujemy usługę punktu montowania:

    # systemctl daemon-reload
    # systemctl restart dev-hugepages.mount

Punkt montowania powinien teraz wyglądać następująco:

    # mount | grep huge
    hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,gid=136,mode=1770,pagesize=2M)

Jeśli w `mode=` widzimy `1352` zamiast `1770` , to prawdopodobnie w pliku
`/etc/systemd/system/dev-hugepages.mount.d/override.conf` podaliśmy `Options=mode=1770` , a nie
`Options=mode=01770` (dodatkowe `0` na pierwszej pozycji).

## Konfiguracja HugePages via /etc/sysctl.conf

Mając przygotowany punkt montowania, trzeba teraz skonfigurować ilość HugePages, które system
będzie nam tworzył. Dodajemy zatem do pliku `/etc/sysctl.conf` poniższe wpisy:

    vm.nr_hugepages = 1024
    vm.nr_overcommit_hugepages = 0
    vm.hugetlb_shm_group = 136

Parametr `vm.nr_hugepages` ma na celu stworzenie określonej ilości HugePages podczas startu systemu,
gdzie wolna przestrzeń w pamięci operacyjnej nie jest jeszcze mocno pofragmentowana. Chodzi
generalnie o to, że te HugePages muszą figurować jako ciągłe kawałki pamięci RAM (2 MiB). Jeśli
teraz byśmy próbowali stworzyć 1024 takich 2 MiB kawałków (łącznie 2 GiB) w działającym systemie,
to może nam się to nie udać, przez co ilość faktycznych HugePages może być sporo niższa. Dla
przykładu, mój system ma do dyspozycji 8 GiB RAM i przy 1,5 GiB zajętej pamięci mogłem stworzyć
tylko 828 ciągłych kawałków o rozmiarze 2 MiB.

    # cat /proc/meminfo | grep -i huge
    AnonHugePages:    602112 kB
    ShmemHugePages:        0 kB
    FileHugePages:         0 kB
    HugePages_Total:     828
    HugePages_Free:      828
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    Hugetlb:         1695744 kB

Podczas startu systemu nie powinno być takich problemów, tj. powinno zostać stworzonych tyle
ciągłych segmentów ile sobie życzymy,

Jeśli zaś chodzi o `vm.nr_overcommit_hugepages` to ten parametr określa jak bardzo może się
rozrosnąć przestrzeń pamięci pod HugePages w przypadku, gdy maszyna wirtualna będzie potrzebować
zaalokować więcej stron po przekroczeniu limitu ustawionego w `vm.nr_hugepages` . Gdy maszyna
wirtualna straci apetyt na pamięć, to te strony zostaną zwolnione ([sterownik balloon][4]).

Parametr `vm.hugetlb_shm_group` ustala grupę, w skład której wchodzą użytkownicy mogący korzystać z
ficzeru HugePages za sprawą segmentów pamięci współdzielonej. Można tutaj pójść na skróty i wskazać
w tym parametrze grupę `kvm` (jej numerek z pliku `/etc/group` ). Procesy `qemu` mają ustawioną
grupę główną jako `kvm` i bez problemu będą w stanie korzystać z HugePages. Możemy też stworzyć
osobną grupę i dodać do niej użytkownika `libvirt-qemu` .

## Włączenie obsługi HugePages w konfiguracji maszyny wirtualnej

Mając skonfigurowane HugePages w systemie hosta, trzeba jeszcze odpowiednio skonfigurować HugePages
na maszynie wirtualnej. Przy założeniu, że mamy już stworzoną jakąś maszynę wirtualną, trzeba
edytować jej konfigurację w pliku XML (w katalogu `/etc/libvirt/qemu/` ):

    # virsh edit ubuntu20.04

i dodać w niej poniższy wpis:

    <memoryBacking>
      <hugepages/>
    </memoryBacking>

I to w zasadzie wszystko co się tyczy konfiguracji HugePages na rzecz maszyn wirtualnych.

## Test QEMU/KVM z włączonym HugePages

Jeśli chcemy tylko przetestować powyższe ustawienia, to wystarczy pozamykać część aplikacji i w
terminalu na hoście wydać te poniższe polecenia:

    # sync; echo 3 > /proc/sys/vm/drop_caches
    # sysctl -p

System powinien stworzyć żądaną ilość HugePages lub też ich możliwie zbliżoną ilość do tej
pożądanej przez nas.

Pozostaje nam już tylko uruchomienie maszyny wirtualnej i sprawdzenie, czy HugePages są
wykorzystywane:

    # cat /proc/meminfo| grep -i huge
    AnonHugePages:    534528 kB
    ShmemHugePages:        0 kB
    FileHugePages:         0 kB
    HugePages_Total:    1024
    HugePages_Free:      274
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    Hugetlb:         2097152 kB

Jak widzimy na powyższej rozpisce, zostały stworzone 1024 strony, z czego wolnych jest 274. Zatem
pod maszynę wirtualną zostało przeznaczone 750 HugePages (1,5 GiB). Przy rozmiarze strony 4 KiB,
tych stron pamięci byłoby 384.000. Zmniejszyliśmy przy tym fragmentację pamięci, która wpływa
bardzo niekorzystnie na pracę kernela wymagającej ciągłych obszarów pamięci RAM, np. zapis dysku
czy dostęp do sieci. Zwiększenie rozmiaru stron wpływa też na mniejszą ilość błędów strony (page
faults), które z kolei są w stanie spowolnić pracę wszystkich aplikacji w systemie. Dodatkowo, jako,
że HugePages nie mogą trafić do SWAP, nie przydarzy nam się sytuacja, w której część danych maszyny
wirtualnej zostanie wymieciona na dysk, co bardzo spowolniłoby pracę samej maszyny wirtualnej.

## Błędy związane z konfiguracją HugePages

Nie od razu udało mi się skonfigurować HugePages do pracy z QEMU/KVM, choć ostatecznie te większe
strony pamięci sprawują się całkiem przyzwoicie. Tak czy inaczej, napotkałem te poniższe błędy oraz
udało mi się je rozwiązać. To tak na wypadek, gdyby ktoś spotkał się z podobnymi problemami.

### Can't open backing store /dev/hugepages/libvirt/qemu/ for guest RAM: Permission denied

Po wydawać by się mogło poprawnym skonfigurowaniu HugePages, uruchamianie maszyny wirtualnej kończy
się poniższym błędem:

    Error starting domain: internal error: process exited while connecting to monitor:
    2020-07-26T15:32:56.007524Z qemu-system-x86_64: can't open backing store
    /dev/hugepages/libvirt/qemu/3-ubuntu20.04 for guest RAM: Permission denied

    Traceback (most recent call last):
      File "/usr/share/virt-manager/virtManager/asyncjob.py", line 75, in cb_wrapper
        callback(asyncjob, *args, **kwargs)
      File "/usr/share/virt-manager/virtManager/asyncjob.py", line 111, in tmpcb
        callback(*args, **kwargs)
      File "/usr/share/virt-manager/virtManager/object/libvirtobject.py", line 66, in newfn
        ret = fn(self, *args, **kwargs)
      File "/usr/share/virt-manager/virtManager/object/domain.py", line 1279, in startup
        self._backend.create()
      File "/usr/lib/python3/dist-packages/libvirt.py", line 1234, in create
        if ret == -1: raise libvirtError ('virDomainCreate() failed', dom=self)
    libvirt.libvirtError: internal error: process exited while connecting to monitor:
    2020-07-26T15:32:56.007524Z qemu-system-x86_64: can't open backing store
    /dev/hugepages/libvirt/qemu/3-ubuntu20.04 for guest RAM: Permission denied

Jak można zauważyć, problemem tutaj są uprawnienia, tylko do czego? Wychodzi na to, że do katalogu
`/dev/hugepages/`. Jeśli widzimy ten błąd, to prawdopodobnie nie ustawiliśmy parametru
`vm.hugetlb_shm_group` lub też ustawiliśmy go błędnie. Upewnijmy się, że użytkownik `libvirt-qemu`
figuruje w tej grupie, którą w tym parametrze ustawiliśmy.

### Unable to map backing store for guest RAM: Cannot allocate memory

Jeśli zaś przy starcie maszyny wirtualnej napotkamy poniższy komunikat błędu:

    Error starting domain: internal error: qemu unexpectedly closed the monitor:
    2020-07-26T16:01:08.200835Z qemu-system-x86_64: unable to map backing store for guest RAM:
    Cannot allocate memory

    Traceback (most recent call last):
      File "/usr/share/virt-manager/virtManager/asyncjob.py", line 75, in cb_wrapper
        callback(asyncjob, *args, **kwargs)
      File "/usr/share/virt-manager/virtManager/asyncjob.py", line 111, in tmpcb
        callback(*args, **kwargs)
      File "/usr/share/virt-manager/virtManager/object/libvirtobject.py", line 66, in newfn
        ret = fn(self, *args, **kwargs)
      File "/usr/share/virt-manager/virtManager/object/domain.py", line 1279, in startup
        self._backend.create()
      File "/usr/lib/python3/dist-packages/libvirt.py", line 1234, in create
        if ret == -1: raise libvirtError ('virDomainCreate() failed', dom=self)
    libvirt.libvirtError: internal error: qemu unexpectedly closed the monitor:
    2020-07-26T16:01:08.200835Z qemu-system-x86_64: unable to map backing store for guest RAM:
    Cannot allocate memory

oznacza to, że ilość HugePages, którą stworzyliśmy w systemie jest niewystarczająca. Trzeba zatem
zwiększyć wartość w parametrze `vm.nr_hugepages` albo też zmniejszyć przydział pamięci RAM maszynom
wirtualnym.

## Transparent HugePages a QEMU/KVM

Problem z HugePages zaczyna się w momencie, gdy nie będziemy robić żadnego użytku z maszyn
wirtualnych (będą wyłączone). Chodzi generalnie o to, że stworzone HugePages rezerwują pamięć
operacyjną i jeśli nie są one wykorzystywane, to w systemie jest do dyspozycji mniejsza ilość
pamięci RAM pod aplikacje. Do tego dochodzi jeszcze problem z brakiem możliwości przeniesienia
HugePages do SWAP.

By zaradzić tym problemom, zwłaszcza w przypadku desktopów/laptopów, gdzie te maszyny wirtualne nie
są odpalone non stop, wprowadzono mechanizm Transparent HugePages, który to automatyzuje tworzenie,
zarządzanie i używanie HugePages. Transparent HugePages potrafi mapować póki co jedynie regiony
pamięci anonimowej (anonymous memory regions), takie jak stos (stack) czy sterta (heap). Czy to
nam wystarczy by ręcznie nie musieć konfigurować HugePages? Jeśli popatrzymy na `AnonHugePages` w
pliku `/proc/meminfo` przed i po uruchomieniu maszyny wirtualnej to możemy ocenić ile HugePages
zostało wykorzystane przez tę konkretną maszynę wirtualną:

    AnonHugePages:   1015808 kB vs. 1994752 kB

Mamy zatem 496 HugePages, które były wykorzystywane przez system zanim maszyna wirtualna została
uruchomiona, a po jej uruchomieniu mamy 974, czyli różnica 478 HugePages, co przekłada się na 956
MiB. Ta maszyna wirtualna ma do dyspozycji 2 GiB RAM. Także nawet połowa obszaru pamięci nie
korzysta z HugePages. Pewnie można by ten wynik poprawić przez mniejszą fragmentację pamięci
operacyjnej ale trzeba sobie zdawać sprawę, że im dłużej system operacyjny działa, tym pamięć RAM
bardziej ulega fragmentacji. Wcale bym się nie zdziwił, gdyby ta uzyskana wartość spadła jeszcze
bardziej. Niemniej jednak, dla przeciętnego desktopa/laptopa, który wykorzystuje maszyny wirtualne
głównie do testów, Transparent HugePages powinny wystarczyć. Jeśli jednak potrzebujemy lepszej
wydajności, to trzeba skonfigurować sobie zwykłe HugePages.


[1]: https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[2]: https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt
[3]: /post/budowanie-kernela-linux-dla-konkretnej-maszyny-z-debianem/
[4]: /post/wirtualizacja-qemu-kvm-libvirt-na-debian-linux/#sterownik-balloon
