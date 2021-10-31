---
author: Morfik
categories:
- Hardware
date: "2016-08-30T17:44:53Z"
date_gmt: 2016-08-30 15:44:53 +0200
published: true
status: publish
tags:
- wifi
- lte
- router
- recenzja
- tp-link
GHissueID: 569
title: 'Recenzja: Przenośny router 3G/4G TL-MR3020  od TP-LINK'
---

Coraz więcej użytkowników migruje od lokalnych providerów internetowych na technologię LTE, która
zapewnia nam połączenie z siecią globalną niezależnie od naszej fizycznej lokalizacji. Do obsługi
LTE potrzebny jest odpowiedni modem lub router WiFi, który już ma taki modem wbudowany. Jeśli
posiadamy modem LTE, to może się pojawić problem z udostępnianiem połączenia, np. w sieci domowej.
Jakby nie patrzeć taki modem jest przeznaczony na jedno urządzenie. Oczywiście w dalszym ciągu
możemy przerobić nasz router WiFi i dodać do niego obsługę modemów LTE ale takie rozwiązanie wymaga
firmware OpenWRT/LEDE. Istnieje prostsza alternatywa, która, można by rzec, działa OOTB i nie trzeba
się zbytnio wysilać przy jej implementacji. Musimy jednak posiadać odpowiednie urządzenie. W tym
artykule obadamy sobie [przenośny router 3G/4G
TL-MR3020](http://www.tp-link.com.pl/products/details/TL-MR3020.html) od TP-LINK.

<!--more-->
## Zawartość opakowania

Opakowanie routera TL-MR3020 jest dość spore ale to z tego powodu, że do zestawu jest dołączonych
jeszcze kilka rzeczy. Poniżej fotki zawartości pudełka:

![router-TL-MR3020-pudelko](/img/2016/08/1.router-TL-MR3020-pudelko.jpg#huge)

![router-TL-MR3020-zawartosc-opakowania](/img/2016/08/2.router-TL-MR3020-zawartosc-opakowania.jpg#huge)

![router-TL-MR3020-zawartosc-opakowania](/img/2016/08/3.router-TL-MR3020-zawartosc-opakowania.jpg#huge)

Jak widzimy wyżej, poza samym routerem, mamy jeszcze dołączony płaski przewód ethernet CAT5:

![router-TL-MR3020-zawartosc-opakowania-ethernet](/img/2016/08/4.router-TL-MR3020-zawartosc-opakowania-ethernet.jpg#huge)

Mamy także biały zasilacz (5V/1A), do którego możemy podłączyć przewód USB:

![router-TL-MR3020-zawartosc-opakowania-zasilacz](/img/2016/08/5.router-TL-MR3020-zawartosc-opakowania-zasilacz.jpg#huge)

![router-TL-MR3020-zawartosc-opakowania-zasilacz](/img/2016/08/6.router-TL-MR3020-zawartosc-opakowania-zasilacz.jpg#huge)

Router można zasilać na kilka sposobów. Najpopularniejszym jest oczywiście podłączenie zasilacza.
Ten przewód w zestawie jest zakończony z jednej strony wtykiem mini USB (do gniazda w routerze), a
drugiej strony ma dwie standardowe wtyki USB typ A. Jedną z nich podłączamy do zasilacza, a zasilacz
wpinamy do gniazdka.

![router-TL-MR3020-zawartosc-opakowania-usb](/img/2016/08/7.router-TL-MR3020-zawartosc-opakowania-usb.jpg#huge)

Po co nam dwie wtyczki USB na tym przewodzie? Otóż w przypadku braku dostępu do gniazdka, możemy
posiłkować się portami USB w komputerze. Niemniej jednak, standardowy port USB 2.0 dysponuje
natężeniem 0,5 A i może okazać się to niewystarczające, by router działał stabilnie, zwłaszcza po
podłączeniu modemu LTE lub/i odpaleniu WiFi. W takim przypadku, router będzie potrzebował więcej niż
0,5 A. Dlatego właśnie są dwie wtyczki, które możemy podłączyć do dwóch portów USB i w ten sposób
zasilić router.

Jeśli dysponujemy power bankiem, np. takim jak ten ostatnio recenzowany
[TL-PB10400](/post/recenzja-power-bank-tl-pb10400-tp-link/), to za jego pomocą
również jesteśmy w stanie zasilić router TL-MR3020:

![router-TL-MR3020-tp-link-zasilanie-power-bank](/img/2016/08/8.router-TL-MR3020-tp-link-zasilanie-power-bank.jpg#huge)

![router-TL-MR3020-tp-link-zasilanie-power-bank](/img/2016/08/9.router-TL-MR3020-tp-link-zasilanie-power-bank.jpg#huge)

Zakładając, że ten router będzie konsumował średnio 1 A (z podłączonym modemem LTE), to ten power
bank jest w stanie go zasilać przez jakieś 10 godzin. Przyda się to na wypadek chwilowej awarii
zasilania w okolicy.

W opakowaniu routera mamy jeszcze instrukcję obsługi, również w języku polskim oraz płytkę CD, na
której jest dołączona bardziej rozbudowana instrukcja obsługi, z tym, że już w języku angielskim
(format `.pdf` ).

## Specyfikacja routera TL-MR3020

Router TL-MR3020 jest bardzo niewielkich rozmiarów: 74x67x22 milimetrów i waży dosłownie kilka gram.
Na jego obudowie mamy widocznych kilka diod:

![router-TL-MR3020-diody-przyciski](/img/2016/08/9.router-TL-MR3020-diody-przyciski.jpg#big)

Licząc od lewej strony, diody sygnalizują stan zasilania, stan połączenia z internetem, stan WiFi,
stan portu ethernet oraz stan WPS. Dioda WPS robi także za przycisk WPS/RESET. Jeśli ten przycisk
zostanie przytrzymany przez czas do 5 sekund, to zadziała funkcja WPS. W przypadku przytrzymania
przycisku przez 10 sekund i więcej, router zostanie zresetowany do ustawień fabrycznych.

Na dwóch z czterech boków routera TL-MR3020 mamy szereg gniazdek i jeden przełącznik:

![router-TL-MR3020-port-ethernet-przelacznik](/img/2016/08/10.router-TL-MR3020-port-ethernet-przelacznik.jpg#big)

![router-TL-MR3020-port-usb](/img/2016/08/11.router-TL-MR3020-port-usb.jpg#big)

Przełącznik jest w stanie przestawić tryb pracy routera. Mamy z grubsza trzy tryby do wyboru: 3G/4G
(LTE), WISP (Wireless ISP) oraz AP (punkt dostępowy). Dalej mamy port fast ethernet (100 mbit/s)
oraz gniazdo do podłączenia zasilacza.

Na drugim boku zaś mamy złącze USB 2.0 , do którego możemy podłączyć jedynie modem LTE. Nie damy
rady podłączyć tutaj pendrive, przynajmniej nie na oryginalnym firmware. W przypadku wgrania
OpenWRT/LEDE, ten port będzie zachowywał się jak zwykły port USB i będziemy w stanie podłączyć do
niego każde urządzenie.

Router TL-MR3020 ma [kilka różnych
wersji](https://wiki.openwrt.org/toh/tp-link/tl-mr3020#supported_hardware_versions), a ten który
trafił do mnie ma wersję 1.9. Wewnątrz obudowy znajduje się układ [WiSoC
AR9331](https://wikidevi.com/wiki/Atheros_AR9331) (Wi-Fi System-On-Chip) od Qualcomm Atheros z
procesorem o taktowaniu 400 MHz. Ten router posiada także układ WiFi pracujący w paśmie 2,4 GHz
(standard N do 150 mbit/s). Brak zewnętrznej anteny odbija się także dość mocno na zasięgu WiFi.
Pamięć operacyjna RAM tego urządzenia to 32 MiB. Natomiast flash jest rozmiarów jedynie 4 MiB.

Nie są to może jakieś pokaźne gabaryty ale z racji swojej funkcjonalności i minimalistycznych
rozmiarów, ten router ma nieco inne przeznaczenie w stosunku do swoich standardowych kolegów
zapewniających nam dostęp do internetu w naszych domach. Router TL-MR3020 nadaje się wręcz idealnie
jako urządzenie zapasowe, na wypadek awarii głównego routera. Poza tym, to urządzenie pobiera
niezmiernie mało energii, średnio w granicach 0,6 W.

Poniżej są fotki podzespołów.

![pcb-TL-MR3020-tp-link-top](/img/2016/08/12.pcb-TL-MR3020-tp-link-top.jpg#huge)

![pcb-TL-MR3020-tp-link-bottom](/img/2016/08/13.pcb-TL-MR3020-tp-link-bottom.jpg#huge)

WiSoC AR9331:

![pcb-TL-MR3020-tp-link-soc](/img/2016/08/14.pcb-TL-MR3020-tp-link-soc.jpg#huge)

Pamięć RAM Windbond W9425G6JH:

![pcb-TL-MR3020-tp-link-pamiec-ram](/img/2016/08/15.pcb-TL-MR3020-tp-link-pamiec-ram.jpg#huge)

Flash Windbond 26232FVSIG:

![pcb-TL-MR3020-tp-link-pamiec-flash](/img/2016/08/16.pcb-TL-MR3020-tp-link-pamiec-flash.jpg#huge)

## Konfiguracja routera TL-MR3020

Router możemy konfigurować przez panel administracyjny dostępny pod adresem `192.168.0.254` .
Każdorazowe przełączenie trybu za pomocą przełącznika na obudowie zrestartuje router w celu
zaaplikowania nowego trybu pracy. Część ustawień poszczególnych trybów pracy, jak nazwa ESSID i
hasło sieci WiFi czy konfiguracja adresacji zostanie zapisana w routerze i przetrwa taki restart.
Część pozostałych opcji zostanie zresetowana do ustawień domyślnych dla danego trybu pracy. Tryby
routera możemy zmieniać także z poziomu panela administracyjnego. Musimy tylko aktywować opcję
"Software Switch":

![router-TL-MR3020-software-switch](/img/2016/08/17.router-TL-MR3020-software-switch.png#huge)

Wyżej mamy informację, że po aktywowaniu tej funkcji, fizyczny przycisk przełączający tryby
urządzenia na routerze zostanie dezaktywowany. Zatem mamy do wyboru przełączanie trybu za pomocą
przycisków lub z poziomu panela administracyjnego i nie można korzystać z obu tych rozwiązań
jednocześnie.

Router ma trzy główne tryby pracy. Są to 3G/4G (LTE), WISP (Wireless ISP) oraz AP. Tryb AP zaś ma
cztery mniejsze tryby: punkt dostępowy, repeater, bridge + AP oraz zwykły klient. Każdy z tych
trybów ma swoje indywidualne opcje w panelu administracyjnym, które możemy sobie dowolnie
skonfigurować.

### Tryb AP

W trybie AP nie działa serwer DHCP routera, a port ethernet jest skonfigurowany jako WAN. Możemy
jednak podłączyć do niego komputer za pomocą skrętki, z tym, że musimy ustawić statyczną
konfigurację sieci 192.168.0.0/24 . Po podłączeniu możemy przejść do panela administracyjnego. Tam
z kolei wchodzimy w `Quick Setup` i możemy zacząć konfigurować poszczególne tryby AP:

![router-TL-MR3020-tryb-pracy-ap](/img/2016/08/18.router-TL-MR3020-tryb-pracy-ap.png#huge)

W przypadku trybu AP, z routerem łączymy się zwykle przez WiFi. Adres IP otrzymamy z serwera DHCP
sieci, do której jest podłączony router za pomocą poru WAN. Jest też zdjęty firewall i inne tego
typu zabezpieczenia. Nie następuje także translacja adresów (NAT).

### Tryb WISP (Wireless ISP)

Router w trybie WISP jest w stanie podłączyć się do sieci bezprzewodowej i udostępnić połączenie tak
jak zwykły router WiFi. W tym przypadku port ethernet robi za port LAN, przez który możemy podłączyć
się przewodowo do routera i nim zarządzać przez panel administracyjny. W panelu mamy możliwość
skonfigurowania zarówno klienta sieci WiFi jak i samego AP, za pomocą którego będziemy mogli się
łączyć z routerem również bezprzewodowo.

![router-TL-MR3020-tryb-pracy-wisp](/img/2016/08/19.router-TL-MR3020-tryb-pracy-wisp.png#huge)

Hasło do AP routera możemy ustawić w opcjach bezpieczeństwa sieci WiFi. Jeśli tego nie zrobimy, to
domyślnie obowiązuje hasło to, które jest nadrukowane na obudowie routera TL-MR3020.

Router w trybie WISP dysponuje serwerem DHCP, zatem nie musimy ręcznie konfigurować połączenia na
klientach. Firewall i inne mechanizmy bezpieczeństwa są włączone. Translacja adresów (NAT) również
występuje.

### Tryb 3G/4G (LTE)

W routerze TL-MR3020 mamy jeden port USB, który jest przeznaczony na modem LTE. Jeśli dysponujemy
takim urządzeniem, to możemy je podłączyć do tego portu, a router przełączyć w tryb 3G/4G. Router
powinien skonfigurować automatycznie połączenie LTE.

Ciekawą opcją jest możliwość skonfigurowania routera na dwóch ISP -- jednego przewodowego i jednego
3G/4G. W prawdzie tylko jedno połączenie może być wykorzystywane w danej chwili ale w przypadku
braku sygnału LTE powinno nastąpić przełączenie na internet z portu WAN. Podobnie też w drugą
stronę, w zależności od ustawionych preferencji:

![router-TL-MR3020-tryb-pracy-lte-3g-4g](/img/2016/08/20.router-TL-MR3020-tryb-pracy-lte-3g-4g.png#huge)

Oczywiście możemy zabronić takiego trybu failover i korzystać jedynie z łącza przewodowego lub LTE.
W tym pierwszym przypadku, port ethernet zostanie skonfigurowany jako WAN, a w tym drugim jako LAN i
będziemy w stanie podłączyć do niego jakiś komputer.

Naturalnie sieć WiFi routera działa cały czas i można się również łączyć z tym urządzeniem
bezprzewodowo. Jedyny problem na jaki możemy natrafić w trybie 3G/4G, to niewspierany modem LTE. Ten
modem, którym ja dysponuję, tj. Huawei E3372s w wersji NON-HiLink, bez problemu współpracuje z tym
routerem. Lista wspieranych modeli modemów LTE jest dostępna [pod tym
linkiem](http://www.tp-link.com.pl/support/3g-comp-list/?model=TL-MR3020). Jeśli nasz modem jest na
liście ale nie działa po podpięciu do routera, to prawdopodobnie powinniśmy zaktualizować firmware
routera.

Domyślnie mamy szereg predefiniowanych operatorów GSM (w zależności od kraju):

![router-TL-MR3020-tryb-pracy-lte-3g-4g-operator-gsm](/img/2016/08/21.router-TL-MR3020-tryb-pracy-lte-3g-4g-operator-gsm.png#huge)

Jeśli nie ma na tej liście tego operatora, z którego usług korzystamy, to zawsze ręcznie możemy
określić parametry połączenia:

![router-TL-MR3020-tryb-pracy-lte-3g-4g-operator-gsm](/img/2016/08/22.router-TL-MR3020-tryb-pracy-lte-3g-4g-operator-gsm.png#huge)

### WPS

Router TL-MR3020 dysponuje przyciskiem WPS i jest możliwość za jego pomocą podłączyć klientów do
sieci WiFi. Standardowo router obsługuje parowanie przez PIN jak i PBC. Przydałoby się ten PIN jak
najszybciej wyłączyć i ustawić jedynie parowanie za pomocą przycisków:

![router-TL-MR3020-wps-pin-pbc](/img/2016/08/23.router-TL-MR3020-wps-pin-pbc.png#big)

## Wsparcie OpenWRT/LEDE dla routera TL-MR3020

Router TL-MR3020 jest [wspierany przez alternatywny firmware
OpenWRT/LEDE](https://wiki.openwrt.org/toh/tp-link/tl-mr3020) i nie powinno być z nim większych
problemów.

![router-TL-MR3020-openwrt-lede](/img/2016/08/24.router-TL-MR3020-openwrt-lede.png#big)

Poniżej znajduje się wynik kilku poleceń.

Log systemowy:

    # dmesg
    [    0.000000] Linux version 3.18.36 (cezary@eko.one.pl) (gcc version 4.8.3 (OpenWrt/Linaro GCC 4.8-2014.04 r46943) ) #54 Sat Jul 9 07:46:06 CEST 2016
    [    0.000000] MyLoader: sysp=5306ac2e, boardp=8e303482, parts=f904f9ea
    [    0.000000] bootconsole [early0] enabled
    [    0.000000] CPU0 revision is: 00019374 (MIPS 24Kc)
    [    0.000000] SoC: Atheros AR9330 rev 1
    [    0.000000] Determined physical RAM map:
    [    0.000000]  memory: 02000000 @ 00000000 (usable)
    [    0.000000] Initrd not found or empty - disabling initrd
    [    0.000000] Zone ranges:
    [    0.000000]   Normal   [mem 0x00000000-0x01ffffff]
    [    0.000000] Movable zone start for each node
    [    0.000000] Early memory node ranges
    [    0.000000]   node   0: [mem 0x00000000-0x01ffffff]
    [    0.000000] Initmem setup node 0 [mem 0x00000000-0x01ffffff]
    [    0.000000] On node 0 totalpages: 8192
    [    0.000000] free_area_init_node: node 0, pgdat 8037c2b0, node_mem_map 81000000
    [    0.000000]   Normal zone: 64 pages used for memmap
    [    0.000000]   Normal zone: 0 pages reserved
    [    0.000000]   Normal zone: 8192 pages, LIFO batch:0
    [    0.000000] Primary instruction cache 64kB, VIPT, 4-way, linesize 32 bytes.
    [    0.000000] Primary data cache 32kB, 4-way, VIPT, cache aliases, linesize 32 bytes
    [    0.000000] pcpu-alloc: s0 r0 d32768 u32768 alloc=1*32768
    [    0.000000] pcpu-alloc: [0] 0
    [    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 8128
    [    0.000000] Kernel command line:  board=TL-MR3020  console=ttyATH0,115200 rootfstype=squashfs,jffs2 noinitrd
    [    0.000000] PID hash table entries: 128 (order: -3, 512 bytes)
    [    0.000000] Dentry cache hash table entries: 4096 (order: 2, 16384 bytes)
    [    0.000000] Inode-cache hash table entries: 2048 (order: 1, 8192 bytes)
    [    0.000000] Writing ErrCtl register=00000000
    [    0.000000] Readback ErrCtl register=00000000
    [    0.000000] Memory: 28324K/32768K available (2606K kernel code, 128K rwdata, 544K rodata, 232K init, 193K bss, 4444K reserved)
    [    0.000000] SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
    [    0.000000] NR_IRQS:51
    [    0.000000] Clocks: CPU:400.000MHz, DDR:400.000MHz, AHB:200.000MHz, Ref:25.000MHz
    [    0.000000] Calibrating delay loop... 265.42 BogoMIPS (lpj=1327104)
    [    0.080000] pid_max: default: 32768 minimum: 301
    [    0.080000] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.090000] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.100000] NET: Registered protocol family 16
    [    0.100000] MIPS: machine is TP-LINK TL-MR3020
    [    0.370000] Switched to clocksource MIPS
    [    0.370000] NET: Registered protocol family 2
    [    0.380000] TCP established hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.380000] TCP bind hash table entries: 1024 (order: 0, 4096 bytes)
    [    0.390000] TCP: Hash tables configured (established 1024 bind 1024)
    [    0.390000] TCP: reno registered
    [    0.390000] UDP hash table entries: 256 (order: 0, 4096 bytes)
    [    0.400000] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes)
    [    0.410000] NET: Registered protocol family 1
    [    0.410000] PCI: CLS 0 bytes, default 32
    [    0.410000] futex hash table entries: 256 (order: -1, 3072 bytes)
    [    0.430000] squashfs: version 4.0 (2009/01/31) Phillip Lougher
    [    0.430000] jffs2: version 2.2 (NAND) (SUMMARY) (LZMA) (RTIME) (CMODE_PRIORITY) (c) 2001-2006 Red Hat, Inc.
    [    0.440000] msgmni has been set to 55
    [    0.440000] io scheduler noop registered
    [    0.450000] io scheduler deadline registered (default)
    [    0.450000] Serial: 8250/16550 driver, 16 ports, IRQ sharing enabled
    [    0.460000] ar933x-uart: ttyATH0 at MMIO 0x18020000 (irq = 11, base_baud = 1562500) is a AR933X UART
    [    0.470000] console [ttyATH0] enabled
    [    0.480000] bootconsole [early0] disabled
    [    0.490000] m25p80 spi0.0: found w25q32, expected m25p80
    [    0.490000] m25p80 spi0.0: w25q32 (4096 Kbytes)
    [    0.500000] 5 tp-link partitions found on MTD device spi0.0
    [    0.500000] Creating 5 MTD partitions on "spi0.0":
    [    0.510000] 0x000000000000-0x000000020000 : "u-boot"
    [    0.510000] 0x000000020000-0x000000141b28 : "kernel"
    [    0.520000] 0x000000141b28-0x0000003f0000 : "rootfs"
    [    0.520000] mtd: device 2 (rootfs) set to be root filesystem
    [    0.530000] 1 squashfs-split partitions found on MTD device rootfs
    [    0.530000] 0x000000390000-0x0000003f0000 : "rootfs_data"
    [    0.540000] 0x0000003f0000-0x000000400000 : "art"
    [    0.540000] 0x000000020000-0x0000003f0000 : "firmware"
    [    0.580000] libphy: ag71xx_mdio: probed
    [    1.170000] ag71xx ag71xx.0: connected to PHY at ag71xx-mdio.1:04 [uid=004dd041, driver=Generic PHY]
    [    1.180000] eth0: Atheros AG71xx at 0xb9000000, irq 4, mode:MII
    [    1.180000] TCP: cubic registered
    [    1.180000] NET: Registered protocol family 17
    [    1.190000] bridge: automatic filtering via arp/ip/ip6tables has been deprecated. Update your scripts to load br_netfilter if you need this.
    [    1.200000] Bridge firewalling registered
    [    1.210000] 8021q: 802.1Q VLAN Support v1.8
    [    1.210000] drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
    [    1.220000] VFS: Mounted root (squashfs filesystem) readonly on device 31:2.
    [    1.230000] Freeing unused kernel memory: 232K (80396000 - 803d0000)
    [    2.530000] init: Console is alive
    [    2.530000] init: - watchdog -
    [    4.620000] usbcore: registered new interface driver usbfs
    [    4.630000] usbcore: registered new interface driver hub
    [    4.630000] usbcore: registered new device driver usb
    [    4.690000] SCSI subsystem initialized
    [    4.700000] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
    [    4.700000] ehci-platform: EHCI generic platform driver
    [    4.710000] ehci-platform ehci-platform: EHCI Host Controller
    [    4.710000] ehci-platform ehci-platform: new USB bus registered, assigned bus number 1
    [    4.720000] ehci-platform ehci-platform: irq 3, io mem 0x1b000000
    [    4.750000] ehci-platform ehci-platform: USB 2.0 started, EHCI 1.00
    [    4.750000] hub 1-0:1.0: USB hub found
    [    4.750000] hub 1-0:1.0: 1 port detected
    [    4.760000] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
    [    4.770000] ohci-platform: OHCI generic platform driver
    [    4.780000] usbcore: registered new interface driver usb-storage
    [    5.600000] init: - preinit -
    [    6.360000] random: procd urandom read with 10 bits of entropy available
    [    8.500000] eth0: link up (100Mbps/Full duplex)
    [    9.740000] mount_root: loading kmods from internal overlay
    [   10.110000] jffs2: notice: (380) jffs2_build_xattr_subsystem: complete building xattr subsystem, 0 of xdatum (0 unchecked, 0 orphan) and 0 of xref (0 dead, 0 orphan) found.
    [   10.130000] block: attempting to load /tmp/jffs_cfg/upper/etc/config/fstab
    [   10.140000] block: extroot: not configured
    [   10.170000] jffs2: notice: (377) jffs2_build_xattr_subsystem: complete building xattr subsystem, 0 of xdatum (0 unchecked, 0 orphan) and 0 of xref (0 dead, 0 orphan) found.
    [   10.320000] block: attempting to load /tmp/jffs_cfg/upper/etc/config/fstab
    [   10.330000] block: extroot: not configured
    [   10.340000] mount_root: switching to jffs2 overlay
    [   10.380000] eth0: link down
    [   10.390000] procd: - early -
    [   10.390000] procd: - watchdog -
    [   11.290000] procd: - ubus -
    [   12.300000] procd: - init -
    [   13.240000] Loading modules backported from Linux version v4.4-rc5-1913-gc8fdf68
    [   13.250000] Backport generated by backports.git backports-20151218-0-g2f58d9d
    [   13.270000] nf_conntrack version 0.5.0 (446 buckets, 1784 max)
    [   13.320000] xt_time: kernel timezone is -0000
    [   13.370000] ip_tables: (C) 2000-2006 Netfilter Core Team
    [   13.450000] PPP generic driver version 2.4.2
    [   13.460000] NET: Registered protocol family 24
    [   13.530000] ath: EEPROM regdomain: 0x0
    [   13.530000] ath: EEPROM indicates default country code should be used
    [   13.530000] ath: doing EEPROM country->regdmn map search
    [   13.530000] ath: country maps to regdmn code: 0x3a
    [   13.530000] ath: Country alpha2 being used: US
    [   13.530000] ath: Regpair used: 0x3a
    [   13.540000] ieee80211 phy0: Selected rate control algorithm 'minstrel_ht'
    [   13.540000] ieee80211 phy0: Atheros AR9330 Rev:1 mem=0xb8100000, irq=2
    [   22.190000] device eth0 entered promiscuous mode
    [   24.690000] eth0: link up (100Mbps/Full duplex)
    [   24.690000] br-lan: port 1(eth0) entered forwarding state

Informacje o procesorze:

    # cat /proc/cpuinfo
    system type             : Atheros AR9330 rev 1
    machine                 : TP-LINK TL-MR3020
    processor               : 0
    cpu model               : MIPS 24Kc V7.4
    BogoMIPS                : 265.42
    wait instruction        : yes
    microsecond timers      : yes
    tlb_entries             : 16
    extra interrupt vector  : yes
    hardware watchpoint     : yes, count: 4, address/irw mask: [0x0ffc, 0x0ffc, 0x0ffb, 0x0ffb]
    isa                     : mips1 mips2 mips32r1 mips32r2
    ASEs implemented        : mips16
    shadow register sets    : 1
    kscratch registers      : 0
    package                 : 0
    core                    : 0
    VCED exceptions         : not available
    VCEI exceptions         : not available

Informacje o porcie USB:

    # cat /sys/kernel/debug/usb/devices

    T:  Bus=01 Lev=00 Prnt=00 Port=00 Cnt=00 Dev#=  1 Spd=480  MxCh= 1
    B:  Alloc=  0/800 us ( 0%), #Int=  0, #Iso=  0
    D:  Ver= 2.00 Cls=09(hub  ) Sub=00 Prot=01 MxPS=64 #Cfgs=  1
    P:  Vendor=1d6b ProdID=0002 Rev= 3.18
    S:  Manufacturer=Linux 3.18.36 ehci_hcd
    S:  Product=EHCI Host Controller
    S:  SerialNumber=ehci-platform
    C:* #Ifs= 1 Cfg#= 1 Atr=e0 MxPwr=  0mA
    I:* If#= 0 Alt= 0 #EPs= 1 Cls=09(hub  ) Sub=00 Prot=00 Driver=hub
    E:  Ad=81(I) Atr=03(Int.) MxPS=   4 Ivl=256ms

Informacje o WiFi:

    # iw list
    Wiphy phy0
            max # scan SSIDs: 4
            max scan IEs length: 2257 bytes
            max # sched scan SSIDs: 0
            max # match sets: 0
            Retry short limit: 7
            Retry long limit: 4
            Coverage class: 0 (up to 0m)
            Device supports AP-side u-APSD.
            Device supports T-DLS.
            Available Antennas: TX 0x1 RX 0x1
            Configured Antennas: TX 0x1 RX 0x1
            Supported interface modes:
                     * IBSS
                     * managed
                     * AP
                     * AP/VLAN
                     * WDS
                     * monitor
                     * mesh point
                     * P2P-client
                     * P2P-GO
                     * outside context of a BSS
            Band 1:
                    Capabilities: 0x116e
                            HT20/HT40
                            SM Power Save disabled
                            RX HT20 SGI
                            RX HT40 SGI
                            RX STBC 1-stream
                            Max AMSDU length: 3839 bytes
                            DSSS/CCK HT40
                    Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                    Minimum RX AMPDU time spacing: 8 usec (0x06)
                    HT TX/RX MCS rate indexes supported: 0-7
                    Frequencies:
                            * 2412 MHz [1] (15.0 dBm)
                            * 2417 MHz [2] (18.0 dBm)
                            * 2422 MHz [3] (18.0 dBm)
                            * 2427 MHz [4] (18.0 dBm)
                            * 2432 MHz [5] (18.0 dBm)
                            * 2437 MHz [6] (18.0 dBm)
                            * 2442 MHz [7] (18.0 dBm)
                            * 2447 MHz [8] (18.0 dBm)
                            * 2452 MHz [9] (18.0 dBm)
                            * 2457 MHz [10] (18.0 dBm)
                            * 2462 MHz [11] (15.0 dBm)
                            * 2467 MHz [12] (disabled)
                            * 2472 MHz [13] (disabled)
                            * 2484 MHz [14] (disabled)
            valid interface combinations:
                     * #{ managed } <= 2048, #{ AP, mesh point } <= 8, #{ P2P-client, P2P-GO } <= 1, #{ IBSS } <= 1,
                       total <= 2048, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz }

                     * #{ WDS } <= 2048,
                       total <= 2048, #channels <= 1, STA/AP BI must match
            HT Capability overrides:
                     * MCS: ff ff ff ff ff ff ff ff ff ff
                     * maximum A-MSDU length
                     * supported channel width
                     * short GI for 40 MHz
                     * max A-MPDU length exponent
                     * min MPDU start spacing
