---
author: Morfik
categories:
- OpenWRT
date: "2016-10-16T20:36:17Z"
date_gmt: 2016-10-16 18:36:17 +0200
published: true
status: publish
tags:
- usb
- chaos-calmer
- router
- tp-link
- uart
title: Konsola szeregowa, adapter USB-UART i uszkodzony router TP-LINK
---

Każdy z nas słyszał o alternatywnym firmware na bezprzewodowe routery WiFi. Mam tutaj na myśli
oczywiście [OpenWRT][1]/[LEDE][2] oraz jego GUI [Gargoyle][3] i [LUCI][4]. Przy zabawach z takim
oprogramowaniem bardzo łatwo jest uszkodzić router w sytuacji, gdy tak na dobrą sprawę nie wiemy co
robimy. Mi się jeszcze nie zdarzyło ubić żadnej z moich maszyn, a mam ich kilka. Problem w tym, że
tak naprawdę nie wiem jak wygląda proces odzyskiwania routera w przypadku zaistnienia takiego złego
scenariusza. Dlatego też postanowiłem zainicjować zdarzenie, które doprowadziło do ubicia systemu w
moim [TL-WR1043ND][5] V2 od TP-LINK. Co zrobić w takim przypadku, gdy system routera nie chce
wystartować, a na obudowie diody sygnalizują nieprawidłową pracę urządzenia? W takiej sytuacji
będziemy musieli rozebrać sprzęt i podłączyć się do portu szeregowego na PCB za pomocą adaptera
USB-UART, najlepiej na układzie CP2102, który bez problemu działa pod linux. Ten artykuł nie
powstałby (tak szybko), [gdyby nie pomoc ze strony @Heinz][6].

<!--more-->
## Jaki adapter USB-UART wybrać

Przed przystąpieniem do procesu odzyskiwania władzy nad systemem routera, musimy się zaopatrzyć w
odpowiednie narzędzia, które umożliwią na bardzo niskopoziomową komunikację z ubitym urządzeniem.
Może i system takiego routera nie może się załadować ale jest duża szansa na to, że bootloader w
dalszym ciągu pracuje prawidłowo (można to poznać po migających diodach), z tym, że z jakiegoś
powodu nie może on uruchomić systemu.

Jest wiele adapterów USB-UART, które możemy zakupić w sklepach internetowych. Co ciekawe, w
fizycznych sklepach w ogóle o takich urządzeniach nie słyszeli (przynajmniej u mnie na wsi) i raczej
pozostaje nam zakup zdalny. Cena takiego adaptera zamyka się w granicach 10-20 zł.

Przy zakupie musimy brać pod uwagę jedną bardzo istotną rzecz. Chodzi o to, że standardowe napięcie
w porcie USB komputera jest na poziomie 5 V. Te adaptery USB-UART na takim napięciu standardowo
działają w przeciwieństwie do obwodu szeregowego w większości routerów. Nie możemy zatem się
podłączyć do routera od tak. Jeśli ten obwód w naszym routerze jest zasilany innym napięciem,
zwykle 3,3 V, to przed zakupem musimy się upewnić, że adapter USB-UART na pinach RX i TX będzie miał
takie właśnie napięcie, ewentualnie będzie wyposażony w konwerter napięć. Patrząc na fotki
urządzenia w sklepie możemy czasem na nich zobaczyć informacje o dwóch rodzajach napięć 3,3 V i 5
V. Popatrzmy na te poniższe adaptery USB-UART na układach CP2102:

![](/img/2016/10/002.usb-uart-adapter-konsola-szeregowa-serial.jpg#huge)

![](/img/2016/10/001.usb-uart-adapter-konsola-szeregowa-serial.jpg#small)

![](/img/2016/10/003.usb-uart-adapter-konsola-szeregowa-serial.jpg#huge)

![](/img/2016/10/004.usb-uart-adapter-konsola-szeregowa-serial.jpg#small)

Na pierwszym adapterze konwerter napięć jest w postaci trzech padów i zespolenie dwóch z nich
steruje napięciem. Na tym drugim zaś mamy nieco bardziej cywilizowane rozwiązanie, bo napięciem
operujemy przy użyciu zworki. Niemniej jednak, w obu przypadkach sterujemy jedynie napięciem na
pinie VCC, a nas interesują piny RX i TX. W przypadku adapterów na układzie CP2102 napięcie na
pinach RX i TX jest zawsze takie samo i wynosi 3,3 V, zatem możemy zakupić dowolny adapter o ile
jest on na tym właśnie układzie.

## Test napięcia w konwerterze USB-UART

Jeśli nie jesteśmy pewni czy zakupiliśmy odpowiedni adapter USB-UART i nie chcemy spalić routera
(czy tylko jego obwodu szeregowego), to zawsze możemy sprawdzić jakie napięcie na pinach RX/TX tego
adaptera faktycznie jest. Do tego celu posłuży nam multimetr. Napięcie mierzymy między GND i RX lub
TX. Powinniśmy uzyskać wskazanie na poziomie około 3,3 V, tak jak to widać na poniższej fotce:

![](/img/2016/11/005.usb-uart-adapter-konsola-szeregowa-test.jpg#huge)

Jak widać z portu USB dostarczanych jest 5 V, a a między pinem GND i TX adaptera mamy już ~3,3 V.
Możemy zatem bez obaw wykorzystać to urządzenie i wpiąć się do portu szeregowego routera. W tym
przypadku położenie zworki nie ma żadnego znaczenia i wskazania są takie same nawet po jej
usunięciu. W przypadku, gdyby zmiana położenia zworki dawała inne napięcia, to oczywiście wybieramy
tę pozycję, która wskazuje 3,3 V.

Warto też zaznaczyć, że oferty adapterów USB-UART znacznie się różnią pod względem dodatków, które
dostajemy razem z adapterem. Zwykle kupujemy gołe urządzenie ale czasem też dorzucane są kolorowe
kabelki albo też i goldpiny. Mi do jednego zestawu dorzucili kabelki, a drugiego trochę goldpinów:

![](/img/2016/10/007.usb-uart-adapter-goldpin.jpg#huge)

![](/img/2016/10/008.usb-uart-adapter-przewody.jpg#huge)

Jeśli jednak nabyliśmy taki adapter bez kabelków i goldpinów, to albo musimy dokupić je osobno, albo
też zrobić je samemu.

## Port szeregowy routera (serial port)

Port szeregowy znajduje się na płycie głównej routera. By się dostać do płyty, trzeba ściągnąć
obudowę. Czasami nie jest to proste zadanie i trzeba poszukać odpowiedniej instrukcji. Zwykle
informacje na ten temat są podane na wiki OpenWRT przy specyfikacji konkretnego modelu routera.

Jeśli zaś chodzi o TL-WR1043ND, to raczej ze ściągnięciem górnej części obudowy nie powinno być
problemu. Po odkręceniu śrubek, trzeba wypiąć ją z zatrzasków. Jest ich 8: 1 na tylnej krawędzi, 2
na prawej, 2 na lewej i 3 na przedniej. Są one rozlokowane w rogach i mniej więcej po środku.

Otwieranie obudowy najlepiej zacząć od jednego z rogów przy tylnej krawędzi zahaczając palcami o
wystający część obudowy i podnosząc ją lekko do góry, tak by można było w powstały otwór wsunąć
cienki ale sztywny kawałek metalu czy czegoś podobnego. Później wystarczy przesunąć się jak
najbliżej rogu i pewnie podważyć, zatrzask powinien bez problemu puścić. Później wystarczy
przesuwać się wzdłuż krawędzi i powtarzać czynność podważenia w miejscu, w którym już nie możemy
dalej się przemieścić.

Po zdjęciu obudowy, mamy dostęp do płyty głównej. Port szeregowy, a właściwie tylko jego
wyprowadzenia są ulokowane w lewej górnej części płyty, to te cztery dziurki widoczne na poniższej
fotce:

![](/img/2016/10/009.usb-uart-tp-link-router-serial-konsola-szeregowa.jpg#huge)

Trzeba tam przylutować jakaś wtyczkę albo chociaż gołe goldpiny. Niemniej jednak, by to zrobić,
musimy wyciągnąć ten PCB z routera. Trzeba pierw odkręcić gniazda anten. Niektóre z nich mogą mieć
problemy z przeciśnięciem się przez otwór. Trzeba nimi trochę pokręcić i poruszać na boki:

![](/img/2016/10/010.usb-uart-tp-link-router-anteny.jpg#huge)

![](/img/2016/10/011.usb-uart-tp-link-router-anteny.jpg#huge)

## Konstrukcja wtyczki dla portu szeregowego routera

Po wyciągnięciu płyty z routera, możemy zabrać się za przylutowanie goldpinów czy wtyczki. Ja sobie
skonstruowałem wtyczkę z elementów odzyskanych ze starej płyty głównej. Odlutowałem z niej 4
goldpiny i ściągnąłem kawałek plastiku, który był do nich przymocowany:

![](/img/2016/10/012.usb-uart-tp-link-router-serial-konsola-szeregowa-wtyczka.jpg#huge)

![](/img/2016/10/013.usb-uart-tp-link-router-serial-konsola-szeregowa-wtyczka.jpg#huge)

![](/img/2016/10/014.usb-uart-tp-link-router-serial-konsola-szeregowa-wtyczka.jpg#huge)

Później takie ustrojstwo wsadziłem na płytę routera i przylutowałem. Całość się trzyma w miarę
pewnie, choć ciężko było to przylutować (brak profesjonalnych narzędzi i wprawy). Tak czy inaczej,
całość działa.

![](/img/2016/10/015.usb-uart-tp-link-router-serial-konsola-szeregowa-wtyczka.jpg#huge)

![](/img/2016/10/016.usb-uart-tp-link-router-serial-konsola-szeregowa-wtyczka.jpg#huge)

Warto przed podłączeniem zasilania routera sprawdzić multimetrem czy nie ma tam czasem jakiegoś
zwarcia. Jeśli miernik nie zabrzęczy, tzn. że jest wszytko w porządku.

## Konfiguracja linux'a na potrzeby połączenia szeregowego

Ja korzystam z linux'a, a konkretnie z dystrybucji Debian. Dlatego też opiszę na jego przykładzie
jak przebiega proces konfiguracji systemu pod kątem połączenia komputera z portem szeregowym
routera.

Przede wszystkim, podepnijmy sam adapter USB-UART do portu USB komputera. Układ CP2102 powinien
zostać bez większego problemu rozpoznany w naszym systemie ( `idVendor=10c4` , `idProduct=ea60` ).
Poniżej log ze zdarzenia:

    kernel: usb 2-1.3: new full-speed USB device number 6 using ehci-pci
    kernel: usb 2-1.3: New USB device found, idVendor=10c4, idProduct=ea60
    kernel: usb 2-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    kernel: usb 2-1.3: Product: CP2102 USB to UART Bridge Controller
    kernel: usb 2-1.3: Manufacturer: Silicon Labs
    kernel: usb 2-1.3: SerialNumber: 0001
    kernel: usbcore: registered new interface driver usbserial
    kernel: usbcore: registered new interface driver usbserial_generic
    kernel: usbserial: USB Serial support registered for generic
    kernel: usbcore: registered new interface driver cp210x
    kernel: usbserial: USB Serial support registered for cp210x
    kernel: cp210x 2-1.3:1.0: cp210x converter detected
    kernel: usb 2-1.3: cp210x converter now attached to ttyUSB0

Został zarejestrowany nowy interfejs `ttyUSB0` i to przy jego pomocy będziemy się komunikować z
routerem. Standardowo tylko użytkownik root oraz członkowie grupy `dialout` mają prawa dostępu do
urządzenia `/dev/ttyUSB0` . Dodajmy zatem standardowego użytkownika do tej grupy w poniższy sposób:

    # adduser morfik dialout

Potrzebne nam jest także jakieś oprogramowanie, które zinterpretuje nasze polecenia i prześle je do
routera. Musimy zatem mieć dostęp do terminala i zainstalować program do komunikacji szeregowej, np.
`minicom` czy `picocom` . W Debianie znajdują się one w pakietach pod tymi samymi nazwami. Wybieramy
sobie jeden z nich i instalujemy w systemie. Ja będę korzystał z `picocom` .

Kolejna sprawa, to konfiguracja `picocom` . By skomunikować się z routerem za pomocą konsoli
szeregowej, musimy zdefiniować kilka parametrów dla połączenia. Konkretne ustawienie zależą od
sprzętu lub jego firmware. Zwykle będziemy musieli dostosować parametr baud, którego najczęściej
używane wartości to 9600, 38400 lub 115200 bit/s. W przypadku korzystania tylko z 3 przewodów (na
PCB routera widzieliśmy 4 piny), tj. RX, TX, GND, powinniśmy ustawić flow control: none. W zasadzie
to są te najważniejsze rzeczy ale możemy także sprecyzować data bits: 8, parity: none, stop bits: 1.

## Oznaczenia pinów na adapterze USB-UART i PCB routera

By zestawić połączenie z routerem potrzebujemy podłączyć do jego płyty co najmniej trzy przewody.
Jeden z nich to masa (GND), drugi odpowiada za przesył danych (TX), a ostatni zaś za odbiór danych
(RX). Jeśli chodzi zaś o sposób podłączenia przewodów, to łączymy następująco: GND-GND, RX-TX i
TX-RX.

Na Adapterze USB-UART mamy stosownie oznaczone piny. Pytanie tylko, które piny na płycie głównej
routera to GND, RX i TX? Zwykle możemy te dane znaleźć na wiki OpenWRT. A co jeśli w przypadku
naszego modelu routera tych danych nie znajdziemy na wiki? W takiej sytuacji trzeba ręcznie ustalić
oznaczenia pinów, a do tego potrzebny nam będzie multimetr.

### Pin GND

Wyłączamy zatem nasz router i przełączamy multimetr w tryb omomierza na brzęczyk. Podłączamy czarną
sondę do znanego nam punktu masy na PCB routera. Jak ustalić gdzie znajduje się znany punkt masy na
PCB routera? Najprościej skorzystać z zasilacza, a konkretnie z gniazda zasilania w routerze. Na
zasilaczu mamy oznaczenia + i - . Wygląda to mniej więcej tak jak na force poniżej:

![](/img/2016/10/017.usb-uart-tp-link-router-serial-konsola-szeregowa-zasilacz.jpg#medium)

W tym przypadku wiemy, że masa jest na zewnątrz. Gdybyśmy nie mieli oznaczeń na zasilaczu, to trzeba
by sprawdzić ten stan rzeczy multimetrem. W tym celu podłączamy sam zasilacz do gniazdka i wtykamy
jedną sondę we wtyczkę, a drugą sondą dotykamy zewnętrznej jej części, tak jak pokazane na poniższej
fotce:

![](/img/2016/10/018.usb-uart-tp-link-router-serial-konsola-szeregowa-masa.jpg#huge)

Wskazanie multimetru przy takim podłączeniu przewodów zarówno do miernika jak i wtyczki zasilacza
jest dodatnie, zatem wiemy, że masa jest tam, gdzie mamy podpięty czarny przewód, tj. na zewnątrz,
co potwierdza oznaczenie na zasilaczu. Mając tę wiedzę, możemy przyjrzeć się bliżej routerowi. Ma on
gniazdo zasilania:

![](/img/2016/10/019.usb-uart-tp-link-router-serial-konsola-szeregowa-gniazdo-zasilania.jpg#huge)

Widać tam bolec w środku i blaszkę na spodzie. Nasz znany punkt masy to właśnie ta blaszka, bo to
ona styka się z zewnętrzną częścią wtyczki zasilacza. To do niej musimy podłączyć naszą czarną
sondę, a czerwoną do każdego kolejnego pinu konsoli szeregowej. Przy jednym z pinów, multimetr
zacznie nam brzęczeń, bo wskaże nam prawie zerową rezystancję. Oznacza to, że znaleźliśmy pin GND
konsoli szeregowej:

![](/img/2016/10/020.usb-uart-tp-link-router-serial-konsola-szeregowa-pin-gnd.jpg#huge)

W przypadku pozostałych pinów, rezystancja będzie sporo wyższa:

![](/img/2016/10/021.usb-uart-tp-link-router-serial-konsola-szeregowa-pin-nie-gnd.jpg#huge)

### Pin VCC

Mając pin GND, możemy włączyć router do prądu. Przełączamy multimetr w tryb woltomierza na zakres
najbliższy 5 V (u mnie 20 DCV) i podpinamy czarną sondę do pinu GND, a czerwoną do każdego z
pozostałych trzech pinów. Wynik jednego ze wskazań powinien nam pokazać 3,3 V (ewentualnie 5 V).
Ten pin oznaczamy VCC. Lepiej wyprowadzić sobie kabelki z pinów na płycie, by czasem nie zewrzeć
pinów, bo może nam to uszkodzić router.

### Pin RX i TX

Pozostały nam dwa piny RX i TX. Odróżnienie ich odbywa się na zasadzie prób i błędów. Mamy w sumie
dwie konfiguracje na linii komputer-router: RX-RX/TX-TX lub RX-TX/TX-RX. Niewłaściwe podłączenie
przewodów sprawi, że na konsoli nie zobaczymy żadnego wyjścia lub też będzie ono w ogóle nieczytelne
dla nas. Z kolei poprawne połączenie RX i TX powinno pokazać nam na konsoli komunikaty z fazy startu
routera.

Z reguły powinniśmy podłączyć te piny naprzemiennie, tj. RX-TX i TX-RX ale czasami oznaczenia RX/TX
na adapterze USB-UART mogą już uwzględniać zamianę tych pinów i wtedy trzeba łączyć RX-RX i TX-TX i
tak było w tym przypadku.

## Nawiązywanie połączenia z routerem za pomocą konsoli szeregowej

Mając odpowiedni adapter USB-UART i oznaczone piny na routerze, możemy spróbować nawiązać połączenie
z routerem przez konsolę szeregową. Podpinamy zatem przewody do adaptera USB-UART oraz do wtyczki
konsoli szeregowej na płycie routera. Pamiętajmy, interesują nas jedynie piny GND, RX i TX. Adapter
umieszczamy w porcie USB komputera.

![](/img/2016/10/022.usb-uart-tp-link-router-serial-konsola-szeregowa-podlaczenie.jpg#huge)

Odpalamy terminal i wpisujemy w nim to poniższe polecenie:

    $ picocom --baud 115200 --flow=none --parity=none --databits 8 /dev/ttyUSB0

    port is        : /dev/ttyUSB0
    flowcontrol    : none
    baudrate is    : 115200
    parity is      : none
    databits are   : 8
    escape is      : C-a
    local echo is  : no
    noinit is      : no
    noreset is     : no
    nolock is      : no
    send_cmd is    : sz -vv
    receive_cmd is : rz -vv
    imap is        :
    omap is        :
    emap is        : crcrlf,delbs,

    Terminal ready

Teraz włączamy przycisk power na routerze. Urządzenie powinno się nam włączyć (zapalą się diody):

![](/img/2016/10/023.usb-uart-tp-link-router-serial-konsola-szeregowa-podlaczenie.jpg#huge)

Jednocześnie obserwujemy co się będzie działo na konsoli. Przy prawidłowej konfiguracji powinniśmy
zobaczyć na niej jakieś komunikaty.

![](/img/2016/10/024.usb-uart-tp-link-router-serial-konsola-szeregowa-log-bootloader.png#huge)

Jak widać, pierwsza część logu to sekwencja bootloader'a. Na samym dole zaś mamy zainicjowany start
kernela. Ta powyższa sytuacja dotyczy sprawnego routera. Co jednak w przypadku, gdy naszemu
routerowi coś dolega? Jeśli jesteśmy w stanie dojść do momentu, gdzie widoczny jest napis
`Autobooting in 1 seconds` (pod koniec logu na powyższej fotce), to jak tylko się on pojawi nam na
ekranie, to musimy wpisać `tpl` . Nie jest to łatwe zadanie i pewnie będziemy musieli powtórzyć tę
czynność parokrotnie. Gdy wpiszemy tę fazę w wyznaczonym czasie, pojawi się prompt:

![](/img/2016/10/025.usb-uart-tp-link-router-serial-konsola-szeregowa-bootloader.png#huge)

## Mapa obszaru flash'a routera

W przypadku, gdy udało nam się uzyskać dostęp do bootloader'a, możemy spróbować wgrać obraz firmware
na router. Zanim jednak przejdziemy bezpośrednio do wgrywania obrazu, musimy ustalić kilka rzeczy.
Gdy nasz router jest sprawny, lub też ktoś inny posiada dokładnie takie samo urządzenie, które
działa bez zarzutu, można z niego wyciągnąć informacje na temat układu partycji na flash'u. Te dane
są dla nas bardzo cenne, bo bez nich nie uda nam się z powodzeniem odratować naszego padniętego
urządzenia.

Układ flash'a routera można odczytać z dwóch plików: `/proc/partitions` oraz `/proc/mtd` . Poniżej
jest zawartość pierwszego z nich:

    root@OpenWrt:/# cat /proc/partitions
    major minor  #blocks  name

      31        0        128 mtdblock0
      31        1       1158 mtdblock1
      31        2       6841 mtdblock2
      31        3       4352 mtdblock3
      31        4         64 mtdblock4
      31        5       8000 mtdblock5

Rozmiar bloku równy jest 1024 KiB. Z powyższych informacji wiemy, że jakiś bliżej nieznany nam
obszar na flash'u routera ma ściśle określony rozmiar. Nie mamy jednak żadnych ludzkich nazw, które
byłyby w stanie nam coś o tych przestrzeniach powiedzieć. Bardziej czytelnym plikiem jest zaś
`/proc/mtd` :

    root@OpenWrt:/# cat /proc/mtd
    dev:    size   erasesize  name
    mtd0: 00020000 00010000 "u-boot"
    mtd1: 00121b70 00010000 "kernel"
    mtd2: 006ae490 00010000 "rootfs"
    mtd3: 00440000 00010000 "rootfs_data"
    mtd4: 00010000 00010000 "art"
    mtd5: 007d0000 00010000 "firmware"

Jak widać wyżej, poza rozmiarem konkretnego obszaru (wartość w HEX) mamy również jego ludzką nazwę.
Cały obraz firmware routera TL-WR1043ND V2 ma 8192 KiB, czyli 8 MiB. Możemy to odczytać sumując
u-boot, firmware i art. Da nam to wartość 0x00800000, czyli po przeliczeniu właśnie 8 MiB. Kernel i
rootfs jest ulokowany w firmware. Z kolei rootfs_data zawiera się w rootfs.

Jeśli nasz router nie chce się uruchomić z jakiegoś powodu ale mamy jednocześnie dostęp do
bootloader'a przez konsolę szeregową, to znaczy, że obszar u-boot nie został naruszony i mamy sporą
szansę na odzyskanie urządzenia. Niemnie jednak, musimy sobie zdawać sprawę, że nie będziemy
zapisywać całej przestrzeni flash'a i z tego względu, ten proces jest iście niebezpieczny, bo źle
wgrany firmware z poziomu bootloader'a może nam kompletnie uwalić router. Urządzenia po źle wgranym
firmware już odratować się tak prosto nie da. Trzeba będzie odlutować flash z PCB routera i podpiąć
go do zewnętrznego programatora. Jak się to robi, to nie mam zielonego pojęcia, więc lepiej się
kilka razy zastanowić przed wydaniem jakichkolwiek poleceń będąc w konsoli bootloader'a.

## Procedura wgrania firmware z poziomu bootloader'a

Przechodzimy zatem na stronę z obrazami firmware OpenWRT/LEDE i pobieramy plik dla modelu naszego
routera mający w nazwie `factory`. Upewnijmy się też, że ten obraz jest przeznaczony na wersję
sprzętową taką, którą router ma wypisaną na spodzie obudowy.

Mając pobrany obraz z firmware, musimy go jakoś przesłać na router. Do tego celu wykorzystuje się
[protokół TFTP][7] (uboższa wersja protokołu FTP). Do zestawienia połączenia wymagany jest klient i
serwer. Klientem będzie router. Serwerem zaś nasz komputer. Bootloader routera posiada odpowiednie
oprogramowanie kliencie (narzędzie `tftp` ). Na komputerze musimy zaś postawić serwer.

### Konfiguracja serwera TFTP pod linux

W dystrybucji Debian jest kilka pakietów mających w nazwie `tftp` . Nas interesują pakiety
zawierające demona umożliwiającego postawienie serwera TFTP, np. `atftpd` lub `tftpd` . Instalujemy
zatem pierwszy z brzega, tj. `atftpd` .

W procesie ratowania routera nie działa WiFi. Musimy zatem łączyć się przy wykorzystaniu przewodu.
Nie będzie nam działał też serwer DHCP, przez co nie uzyskamy adresacji dynamicznej dla komputera.
Musimy określić ją statycznie w poniższy sposób:

    # ifdown eth0
    # ip link set dev eth0 up
    # ip addr add 192.168.1.100/24 brd + dev eth0

Adres widoczny wyżej (192.168.1.100) nie może być przypadkowy. Będąc w konsoli bootloader'a, przy
pomocy `printenv` możemy podejrzeć kilka zmiennych, które u-boot wykorzystuje do swojej pracy. Tam z
kolei mamy min. te dwie poniższe:

    ipaddr=192.168.1.111
    serverip=192.168.1.100

Zatem router będzie miał adres `192.168.1.111` i będzie próbował połączyć się z adresem
`192.168.1.100` . Dlatego to ten adres musimy ustawić w konfiguracji karty sieciowej komputera.
Jeśli  kogoś interesują pozostałe zmienne zwrócone przez `printenv` , to są one opisane na [wiki
OpenWRT][8].

Musimy jeszcze uruchomić serwer TFTP. W tym celu edytujemy pierw plik `/etc/default/atftpd` i
przepisujemy go do poniższej postaci:

    USE_INETD=false
    OPTIONS="--tftpd-timeout 300 --retry-timeout 5 --maxthread 100 --verbose=5 --bind-address=192.168.1.100 /srv/tftp-openwrt"

Tworzymy jeszcze katalog `/srv/tftp-openwrt/` i nadajemy mu odpowiednie uprawnienia:

    # mkdir /srv/tftp-openwrt/
    # chown nobody:nogroup /srv/tftp-openwrt/

Teraz stawiamy serwer poniższym poleceniem, weryfikując przy tym czy został on faktycznie
uruchomiony:

    # /etc/init.d/atftpd start
    # netstat -tupan | grep atftpd
    udp     0   0 192.168.1.100:69    0.0.0.0:*   25408/atftpd

### Jak wgrać firmware przez konsolę szeregową z poziomu bootloader'a

Serwer TFTP został uruchomiony, a jego katalog główny to `/srv/tftp-openwrt/` . Do tego katalogu
musimy wrzucić plik z obrazem firmware, nazywając go przy tym, np. `obraz.bin` , nazwa dowolna:

    # cp /home/morfik/Desktop/openwrt-15.05-ar71xx-generic-tl-wr1043nd-v2-squashfs-factory.bin /srv/tftp-openwrt/obraz.bin

Przechodzimy teraz do konsoli szeregowej. U-boot dysponuje kilkoma użytecznymi poleceniami. My
będziemy wykorzystywać `tftp` , `erase` oraz `cp.b` . Poniżej znajdują się polecenia, które ja
wykorzystałem przy ratowaniu swojego TL-WR1043ND V2.

Polecenie `tfpt` pobierze plik obrazu firmware z dysku komputera i wrzuci go do pamięci RAM routera.
Ta komenda przyjmuje dwa argumenty. Są nimi adres pamięci RAM, od którego obraz firmware będzie
zapisywany, oraz nazwa pliku firmware, który zostanie pobrany z serwera TFTP naszego komputera.
Nazwę znamy ale problematyczne może być określenie adresu pamięci RAM. O tym będzie w dalszej
części artykułu.

    ap135> tftp 0x80060000 obraz.bin
    Using eth1 device
    TFTP from server 192.168.1.100; our IP address is 192.168.1.111
    Filename 'obraz.bin'.
    Load address: 0x80060000
    Loading: #################################################################
    Bytes transferred = 8126464 (7c0000 hex)

Polecenie `erase` służy do zerowania pamięci flash. Przyjmuje ono również dwa argumenty. Pierwszym z
nich jest adres pamięci flash, od którego ma się rozpocząć zerowanie. (ustalenie go również może być
problematyczne). Drugim argumentem jest ilość bajtów, które zostaną wyzerowane (musi pasować do
0x7c0000). Operację zerowania musimy wykonać ze względu na [budowę komórek pamięci flash][9].
Dopiero po wyzerowaniu interesującego nas obszaru, możemy zacząć umieszczać na nim dane z firmware.

    ap135> erase 0x9f020000 +0x7c0000
    Erasing flash...
    First 0x2 last 0x7d sector size 0x10000
     125
    Erased 124 sectors

Polecenie `cp.b` ma za zdanie skopiować firmware z pamięci RAM i wgrać je na pamięć flash. Tutaj z
kolei musimy określić trzy argumenty. Pierwszym z nich jest początkowy adres pamięci RAM, czyli to
miejsce, w którym znajduje się początek obrazu firmware. Drugim argumentem jest początkowy adres
pamięci flash, od którego zamierzamy zacząć wgrywać firmware. Trzecim zaś argumentem jest ilość
bajtów, które zostaną skopiowane z pamięci RAM na pamięć flash routera.

    ap135> cp.b 0x80060000 0x9f020000 0x7c0000
    Copy to Flash... write addr: 9f020000
    done

Teraz można zrestartować router. Jeśli nie pomyliliśmy adresów i przeprowadziliśmy powyższe kroki
prawidłowo, to będziemy w stanie się zalogować na router z wykorzystaniem protokołu telnet.

### Jak ustalić odpowiednie adresy pamięci RAM i flash routera

Powyższe adresy są unikatowe dla konkretnego modelu routera. Nie możemy zatem korzystać z nich w
przypadku każdego routera, który nawali z jakiegoś bliżej nieznanego nam powodu. Nasuwa się zatem
pytanie: jak ustalić te adresy?

Zwykle tego typu informacje powinny być podane na wiki OpenWRT. W przypadku części
routerów,polecenie `printenv` (wydane z poziomu bootloader'a) jest nam w stanie zwrócić kilka
zmiennych zawierających pewne polecenia. Poniżej przykład z mojego routera:

    ...
    bootcmd=bootm 0x9f020000
    ...
    lu=tftp 0x80060000 ${dir}u-boot.bin&&erase 0x9f000000 +$filesize&&cp.b $fileaddr 0x9f000000 $filesize
    lf=tftp 0x80060000 ${dir}ap135${bc}-jffs2&&erase 0x9f050000 +0x630000&&cp.b $fileaddr 0x9f050000 $filesize
    lk=tftp 0x80060000 ${dir}vmlinux${bc}.lzma.uImage&&erase 0x9f680000 +$filesize&&cp.b $fileaddr 0x9f680000 $filesize
    ...

Na podstawie użytych tutaj adresów można ustalić adres pamięci RAM i flash, od których powinniśmy
zacząć zapisywać dane.

W `bootcmd` mamy adres pamięci flash, który ma zostać odczytany podczas fazy startu routera. Przed
nim znajduje się tylko obszar bootloader'a. Wiemy zatem od którego adresu trzeba będzie flash
wyczyścić. Znając rozmiar firmware (ten wgrany do pamięci RAM), wiemy jaki obszar należy wyzerować.
W `lu` widzimy zaś `tftp 0x80060000` . Jako, że polecenie `tftp` ładuje obraz do pamięci RAM
routera, to mamy adres pamięci od którego możemy zacząć zapisywać dane w RAM.

Nie zawsze tego typu zmienne będziemy mieć dostępne w środowisku bootloader'a i odpowiednie adresy
trzeba będzie ustalić w bliżej mi nie znany jeszcze sposób. :D

Warto tutaj nadmienić, że wyżej widoczny adres odnoszący się do pamięci flash, tj. 0x9f000000,
zostawiamy w spokoju. Określa on początek pamięci i nie możemy się nim posługiwać. Przy czyszczeniu
i wgrywaniu danych na pamięć flash ( `erase` i `cp.b` ) musimy uwzględnić offset na bootloader.
Standardowo jest to 128 KiB, które w zapisie HEX przyjmują wartość 0x20000. Trzeba zatem tę wartość
dodać do 0x9f000000 i w ten sposób otrzymamy początkowy adres pamięci flash, od którego możemy
zacząć kasować/zapisywać dane z firmware, tj. 0x9f020000, czyli uzyskaliśmy dokładnie taki sam
adres co w `bootcmd` .

Trzeba także uważać na długość wgrywanego obrazu (ilość bajtów). Na końcu flash'a routera mającego
podzespoły od Qualcomm Atheros znajduje się obszar art (Atheros Radio Test). Zawiera on dane
kalibracyjne dla WiFi (EEPROM). Informacje zawarte na tej partycji są unikatowe dla każdego
indywidualnego routera. Jeśli uszkodzimy ten obszar, np. zapisując zbyt wiele danych, to trwale
uwalimy WiFi w routerze, no chyba, że mamy backup tej przestrzeni albo i całego flash'a routera i
będziemy w stanie ten obszar odtworzyć.

## Przykład z życia, czyli planowe ubicie TL-WR1043ND

Nigdy mi się jeszcze nie zdarzyło ubić żadnego z posiadanych przeze mnie sprzętów. Niemniej jednak,
przydałoby się odpowiedzieć na kilka pytań, a do tego potrzebny jest test na żywym organizmie.
Postanowiłem zatem wgrać na mój router TL-WR1043ND V2 kilka różnych obrazów, w tym też te
nieprzeznaczone dla niego. Obrazy są różne: oficjalne (z/bez boot) oraz OpenWRT/LEDE
factory/sysupgrade. Przetestowałem też zanik zasilania podczas wgrywania firmware. Wszystko po to,
by sprawdzić co tak naprawdę jest w stanie uszkodzić router i w jakim stopniu.

### Czy obraz "factory" uszkodzi router z OpenWRT/LEDE

Mając wgrany firmware OpenWRT/LEDE zwykle flash'ujemy go obrazem mającym w nazwie `sysupgrade` . Jak
jednak zareaguje nasz router po podaniu mu pełnego obrazu `factory` ? W sumie zawsze chciałem to
sprawdzić ale jakoś nigdy nie było okazji. Kopiujemy zatem firmware do pamięci routera via `scp` ,
logujemy się na router i wgrywamy firmware via `sysupgrade` :

![](/img/2016/10/026.usb-uart-tp-link-router-serial-konsola-szeregowa-factory.png#huge)

Jak widzimy wyżej, router działa bez problemu. Zatem wiemy, że obrazy OpenWRT/LEDE mające w nazwie
`factory` nie ubiją nam routera. Niemniej jednak, w porównaniu do tych mających w nazwie
`sysupgrade` są zwykle parokrotnie większe przez co lepiej jest korzystać z tych drugich, a
`factory` używać jedynie przy zmianie firmware z oficjalnego na OpenWRT/LEDE.

### Czy obraz przeznaczony na inną wersję/model routera uszkodzi router

Wgranie obrazu firmware przeznaczonego na inną wersję/model routera powinno uszkodzić router. W
przypadku innej wersji sprzętowej nie zawsze jest to regułą. Pytanie zwykle sprowadza się do różnicy
w podzespołach routera. Jeśli te się zmieniły nieznacznie, to jest duża szansa, że router wstanie,
choć może nie działać tak jak byśmy tego oczekiwali.

Ja dysponuję routerem TL-WR1043ND w wersji V2, a on się zmienił dość znacznie w porównaniu do wersji
V1. Możemy zatem przypuszczać, że wgranie firmware dla wersji V1 ubije router. Sprawdźmy zatem co
faktycznie się stanie (bez znaczenia czy użyjemy `sysupgrade` czy `factory` ).

![](/img/2016/10/027.usb-uart-tp-link-router-serial-konsola-szeregowa-inny-model-wersja.png#huge)

Nie mamy możliwości wgrania firmware przeznaczonego na inną wersję sprzętową routera. Podobnie
sprawa będzie wyglądać w przypadku zupełnie innego modelu routera. Jak widać w powyższym
komunikacie, został porównany ID: `Invalid image, hardware ID mismatch, hw:10430002
image:10430001` . Zatem ID sprzętu nie pasuje do ID uzyskanego z obrazu, w efekcie system odmawia
próby wgrania takiego firmware z oczywistych względów.

Może i mechanizmy obronne wbudowane w firmware OpenWRT/LEDE działają i są w stanie nas ochronić
przed tak banalnymi błędami, to w dalszym ciągu możemy uszkodzić router wgrywając oprogramowanie na
siłę przez `sysupgrade` (przełącznik `-F` ). Innym sposobem flash'owania, który może nam napytać
biedy, jest nieumiejętne korzystanie z narzędzia `mtd` , przy pomocy którego można wykonywać
operacje bezpośrednio na pamięci flash.

Jeśli zatem zamierzamy przy pomocy `sysupgrade -F` lub `mtd write` wgrywać jakiś obraz firmware, to
upewnijmy się, że nie jest on przeznaczony na inne wersje sprzętowe routerów albo i całkowicie inne
modele.

### Czy zanik prądu uszkodzi router z OpenWRT/LEDE

Zanik prądu jest znanym mordercą routerów. Niemniej jednak, z racji różnicy w oficjalnym firmware
TP-LINK'a i firmware OpenWRT/LEDE, sprawa może się inaczej potoczyć. Przede wszystkim, zanik prądu
może pojawić się w różnych momentach. W przypadku obrazów TP-LINK'a mających w nazwie `boot` ,
system będzie próbował przepisać bootloader (bez części konfiguracyjnej). By to zrobić musi
wyczyścić ten obszar. Jeśli w tym czasie nastąpi zanik napięcia, to nie tylko router zostanie
uszkodzony ale też już nie damy rady go odzyskać przez konsolę szeregową i trzeba będzie korzystać z
programatora. W przypadku firmware OpenWRT/LEDE nie musimy się aż tak martwić, bo tutaj nie jest
ruszany w ogóle bootloader. W efekcie nawet jeśli nastąpi zanik napięcia, to obszar u-boot zostaje
nietknięty i możemy wgrać się później przez konsolę szeregową. Podobnie sprawa ma się w przypadku
obrazów oficjalnych bez frazy `boot` w nazwie. Tutaj również nie jest przepisywany bootloader i przy
zaniku napięcia będziemy w stanie odzyskać router.

Sprawdźmy zatem co się stanie po puszczeniu procesu aktualizacji firmware via `sysupgrade` i
odłączeniu zasilania routera tak po trzech sekundach. Oczywiście flash'ujemy router z poziomu
`sysupgrade` na firmware OpenWRT/LEDE:

![](/img/2016/10/028.usb-uart-tp-link-router-serial-konsola-szeregowa-zanik-zasilania.png#huge)

W momencie uwiecznionym powyżej nastąpił symulowany zanik zasilania. System routera po włączeniu nie
startuje. Zapalają się wszystkie diody, po czym część z nich gaśnie. Dioda Power, System i WPS
świecą się ciągle. Dioda WLAN jest zgaszona cały czas, natomiast wszystkie pięć diod od portów
ethernet miga. Ten proces zdaje się być zapętlony, czyli jakaś aktywność routera się zachowała.

Jako, że diody migają i router się cały czas restartuje, oznacza to, że bootloader działa raczej
prawidłowo i jego obszar nie uległ przepisaniu. Niemniej jednak, bootloader nie jest w stanie
podnieść systemu. Napotyka jakiś "bliżej nieznany błąd" i restartuje system mając nadzieję, że to
coś da. Poniżej jest komunikat, który można zobaczyć na konsoli szeregowej:

    Autobooting in 1 seconds
    ## Booting image at 9f020000 ...
       Uncompressing Kernel Image ... ERROR: LzmaDecode.c, 543

    Decoding error = 1
    LZMA ERROR 1 - must RESET board to recover

Router jest do odzyskania ale musimy zrobić użytek z konsoli szeregowej i opisanym wyżej sposobem
wgrać firmware z poziomu bootloader'a.

### Czy obraz przeznaczony na inną wersję/model routera ubije router z oficjalnym firmware

Podobnie jak w przypadku firmware OpenWRT/LEDE, oficjalny firmware TP-LINK'a również weryfikuje
obrazy przed wgraniem ich przez panel administracyjny. Nie musimy się zatem obawiać, że przez
przypadek wgramy obraz nieprzeznaczony dla naszego modelu/wersji routera.

![](/img/2016/10/029.usb-uart-tp-link-router-serial-konsola-szeregowa-inny-router.png#huge)

![](/img/2016/10/030.usb-uart-tp-link-router-serial-konsola-szeregowa-inny-router.png#huge)

![](/img/2016/10/031.usb-uart-tp-link-router-serial-konsola-szeregowa-inna-wesja.png#huge)

![](/img/2016/10/032.usb-uart-tp-link-router-serial-konsola-szeregowa-inna-wersja.png#huge)

### Czy wgranie oficjalnego firmware z "boot" w nazwie via sysupgrade uszkodzi router z OpenWRT/LEDE

Firmware OpenWRT/LEDE nie dotyka praktycznie wcale obszaru flash, w którym siedzi bootloader. Jeśli
teraz zamierzamy powrócić do oficjalnego firmware, to musimy pozyskać stosowny obraz. Wszystkie
obrazy na stronie TP-LINK mają frazę `boot` . Oznacza to, że zawierają one część bootloader'a, którą
proces flash'owania przepisuje podczas aktualizacji firmware z poziomu panelu administracyjnego
TP-LINK'a. Jeśli taki obraz podamy w `sysupgrade` , to ta część bootloader'a z oficjalnego firmware
powędruje na flash routera w miejsce, gdzie powinno znajdować się faktyczne oprogramowanie. W
efekcie router nie będzie chciał się uruchomić ale będziemy w stanie go odzyskać przez konsolę
szeregową.

By uniknąć tego typu problemów, potrzebny nam jest obraz nieposiadający w nazwie frazy `boot` .
Takie obrazy są dostępne w różnych miejscach: [tutaj][10], [tutaj][11], [tutaj][12] i pewnie
jeszcze w wielu innych, o których nie wiem.

Jeśli jednak dysponujemy obrazem mającym w nazwie `boot` i z jakiegoś powodu nie możemy pozyskać
obrazu bez części bootloader'a, to możemy ten zbędny i niebezpieczny zarazem kawałek usunąć ręcznie.
Możemy to zrobić przy pomocy `dd` z poziomu każdego linux'a (nawet OpenWRT/LEDE) w poniższy sposób:

    # dd if=firmware_z_boot.bin of=firmware_bez_boot.bin skip=257 bs=512

To powyższe polecenie usunie z początku obrazu pierwsze 131584 bajów (0x20200 w HEX) i taki obraz
możemy już wgrać bez problemu na router via `sysupgrade` .

### Czy wgranie oficjalnego firmware NIEmającego "boot" w nazwie przez panel admina uszkodzi router

Gdy oficjalny plik z firmware nie zawiera frazy `boot` w nazwie, to jest to mniej więcej taka sama
wersja obrazu co w przypadku firmware OpenWRT/LEDE, który ma w nazwie `factory` . Wgranie
oficjalnego obrazu bez `boot` z poziomu panelu administracyjnego TP-LINK'a nie uszkodzi naszego
routera.

![](/img/2016/10/033.usb-uart-tp-link-router-serial-konsola-szeregowa-bez-boot.png#huge)

![](/img/2016/10/034.usb-uart-tp-link-router-serial-konsola-szeregowa-bez-boot.png#huge)

Wgrywanie oficjalnych obrazów bez `boot` w nazwie jest nawet bezpieczniejsze, bo w tym przypadku nie
jest przepisywany obszar u-boot. Dlatego też nawet w przypadku utraty zasilania podczas procesu
flash'owania routera, będziemy w stanie odzyskać router przez konsolę szeregową.

### Czy obraz "sysupgrade" wgrany z panelu TP-LINK uszkodzi router

Przy przechodzeniu z oficjalnego firmware na OpenWRT/LEDE stosuje się obrazy mające w nazwie
`factory` . Z kolei zaś obrazy mające w nazwie `sysupgrade` są przeznaczone dla aktualizacji
firmware routera. Czy wgrywając obraz `sysupgrade` można uszkodzić router? Sprawdźmy:

![](/img/2016/10/035.usb-uart-tp-link-router-serial-konsola-szeregowa-sysupgrade.png#huge)

Wygląda na to, że oficjalne oprogramowanie nie przyjmuje takiego obrazu zupełnie. Obraz `factory` i
`sysupgrade` praktycznie niczym się nie różnią pod kątem zawartości. W przypadku `factory` na
początku obrazu znajduje się tylko dodatkowy nagłówek i prawdopodobnie jego brak uniemożliwia
wgranie obrazu na router z poziomu panelu administracyjnego TP-LINK'a.

## Backup pamięci flash routera

Jak widać z powyższych obserwacji bardzo ciężko jest doprowadzić router do stanu nieużywalności.
Jeśli nie przytrafi nam się nagły zanik zasilania podczas procesu flash'owania, to raczej nie mamy
możliwości uszkodzić routera, no chyba, że bardzo się o taki stan rzeczy postaramy. By uniknąć
problemów przy ewentualnym eksperymentowaniu z routerem, najlepiej postarać się o backup całej
przestrzeni flash.

### Backup z poziomu działającego firmware

Jeśli nasz router działa bez problemu, możemy się na niego zalogować po ssh i przy pomocy `dd`
zrzucić obrazy wszystkich pozycji, które widnieją w pliku `/proc/mtd` . Oczywiście moglibyśmy zrobić
backup tylko u-boot, firmware i art ale później musielibyśmy kroić obraz by [wyodrębnić konkretne
jego części][13]. Backup robimy przy pomocy `dd` zapisując dane w pamięci RAM routera:

    root@OpenWrt:/tmp# dd if=/dev/mtdblock0 of=/tmp/mtdblock0-u-boot.bin
    root@OpenWrt:/tmp# dd if=/dev/mtdblock1 of=/tmp/mtdblock1-kernel.bin
    root@OpenWrt:/tmp# dd if=/dev/mtdblock2 of=/tmp/mtdblock2-rootfs.bin
    root@OpenWrt:/tmp# dd if=/dev/mtdblock3 of=/tmp/mtdblock3-rootfs_data.bin
    root@OpenWrt:/tmp# dd if=/dev/mtdblock4 of=/tmp/mtdblock4-art.bin
    root@OpenWrt:/tmp# dd if=/dev/mtdblock5 of=/tmp/mtdblock5-firmware.bin

Następnie tak utworzone pliki trzeba przesłać na komputer, np. via `scp` :

    $ scp root@192.168.1.1:/tmp/mtd* ./

### Odtwarzanie backup'u z poziomu firmware

Odtworzenie backup'u uszkodzonego obszaru pamięci flash z poziomu działającego firmware jest
stosunkowo proste. Załóżmy, że uszkodziliśmy sobie obszar art, w wyniku czego uszkodziliśmy WiFi. By
przywrócić ten obszar, logujemy się na router i wrzucamy do pamięci plik backup'u. Później przy
pomocy `mtd write` wgrywamy backup na odpowiedni obszar pamięci flash w poniższy sposób:

    # mtd write /tmp/mtdblock4-art.bin art

### Backup i odtwarzanie go z poziomu bootloader'a

W przypadku, gdy nasz router znajduje się w stanie agonalnym ale mamy jeszcze możliwość zalogowania
się do bootloader'a, to możemy pokusić się o zrobienie backup'u flash'a za sprawą konsoli
szeregowej. Niemniej jednak, nie wszystkie wersje u-boot dysponują komendami umożliwiającymi
przesłanie backup'u na serwer TFTP. Jeśli u-boot naszego routera takimi poleceniami nie dysponuje,
to musimy postarać się o obraz initramfs, który załadujemy do pamięci RAM routera. W tym initramfs
będą znajdować się potrzebne nam polecenia i backup wykonamy bez trudu. Obraz initramfs musimy
sobie sami zbudować albo tez pozyskać od kogoś, kto już go zbudował. Jak przeprowadzić proces budowy
obrazu initramfs wykracza poza ramy tego artykułu. (link FIXME).

Jeśli zaś chodzi o kwestie odtworzenia backupu za pomocą bootloader'a, to raczej nie będziemy
potrzebować takiej funkcjonalności. Poza tym, nie wszystkie bootloader'y dają nam możliwość
skorzystania z `mtd` , tak jak to ma miejsce przy działającym firmware OpenWRT/LEDE. Nawet jeśli
nasz bootloader wspiera taką możliwość, to trzeba uważać, bo nie zawsze obszary mtd widziane przez
bootloader muszą się pokrywać z tymi udostępnianymi przez kernel w pliku `/proc/mtd` . Dlatego też
lepiej posługiwać się offset'ami.

## Problemy z uruchomieniem routera przy podłączonym adapterze USB-UART

Mój routera TL-WR1043ND V2 miał wgrany firmware OpenWRT Chaos Calmer. Był też jak najbardziej
sprawny ale po podłączeniu adaptera USB-UART nie chciał się uruchomić. Zatrzymywał się na etapie
bootloader'a z poniższą informacją:

    U-Boot 1.1.4 (Sep 10 2015 - 12:05:08)

    ap135 - Scorpion 1.0DRAM:
    sri
    Scorpion 1.0
    ath_ddr_initial_config(211): (16bit) ddr1 init
    tap = 0x00000002
    Tap (low, high) = (0xaa55aa55, 0x0)
    Tap values = (0x8, 0x8, 0x8, 0x8)
     4 MB

Niektóre adaptery USB-UART mogą uniemożliwić start routera. No to pojawia się pytanie jak próbować
odzyskać router, gdy nie mamy możliwości zainicjowania bootloader'a?

Najprościej jest kupić normalny adapter (jakieś polecane modele?), który nie zawiesi startu routera.
Innym wyjściem jest podłączenie przewodu do pinu GND po włączeniu przycisku power routera. Trzeba
tylko ustalić czas, po którym możemy ten pin GND podłączyć, a to już robimy metodą prób i błędów. W
moim przypadku mogłem praktycznie od razu się podłączyć (tuż przed zgaśnięciem wszystkich diod na
routerze). W innych sytuacjach trzeba będzie poczekać jedną czy kilka sekund i dopiero wtedy się
wpiąć. Nie trzeba się oczywiście spieszyć z tym faktem, bo bootloader się będzie w kółko resetował,
przez co i tak zobaczymy wszystkie komunikaty, które nam zostaną zwrócone.


[1]: https://openwrt.org/
[2]: https://lede-project.org/
[3]: https://www.gargoyle-router.com/
[4]: http://eko.one.pl/?p=openwrt-luci
[5]: http://www.tp-link.com.pl/products/details/TL-WR1043ND.html
[6]: http://tplink-forum.pl/index.php?/topic/5322-jak-ustali%C4%87-oznaczenia-port%C3%B3w-konsoli-szeregowej-na-pcb/
[7]: https://pl.wikipedia.org/wiki/Trivial_File_Transfer_Protocol
[8]: https://wiki.openwrt.org/doc/techref/bootloader/uboot.config
[9]: https://pl.wikipedia.org/wiki/Pami%C4%99%C4%87_flash#Ograniczenia
[10]: ftp://tplink-forum.pl/orgin_bez_boot/
[11]: http://dl.eko.one.pl/orig/
[12]: http://www.friedzombie.com/tplink-stripped-firmware/
[13]: https://wiki.openwrt.org/doc/techref/flash.layout
