---
author: Morfik
categories:
- Linux
date: "2020-03-10T03:28:00Z"
published: true
status: publish
tags:
- debian
- efi
- uefi
- hdd
- ssd
- refind
GHissueID: 19
title: Jak przygotować dysk pod instalację Debian linux z EFI/UEFI
---

Instalacja linux'a w trybie EFI/UEFI nieco inaczej wygląda niż tradycyjna instalacja systemu, zwana
często dla odróżnienia trybem BIOS, przynajmniej przy wykorzystaniu `debootstrap` Jeśli kupujemy
nowego desktopa czy laptopa, to zwykle będziemy mieli na dysku twardym zainstalowanego windows'a i
tym samym przygotowany cały układ partycji niezbędny do prawidłowego uruchomienia systemu w trybie
EFI/UEFI. Co jednak w przypadku, gdy kupimy komputer bez systemu operacyjnego? W takiej sytuacji
trzeba będzie ręcznie podzielić dysk na partycje oraz zainstalować menadżer rozruchu (rEFInd) lub
też bootloader (grub/grub2/syslinux/extlinux) i skonfigurować wszystkie te elementy samodzielnie.
Prawdopodobnie instalator Debiana jest w stanie za nas te kroki przeprowadzić automatycznie ale my
nie będziemy korzystać z automatycznych rozwiązań, bo one nieco odmóżdżają. Spróbujemy za to
stworzyć sobie uniwersalną konfigurację, która pozwoli nam zainstalować i odpalić dowolną
dystrybucję linux'a w trybie EFI/UEFI.

<!--more-->
## Różnice między EFI/UEFI, a BIOS

Postanowiłem trochę poczytać na temat tego całego firmware EFI/UEFI oraz zebrać w paru słowach te
co ważniejsze rzeczy, które powinny przygotować nas do instalacji linux'a w trybie EFI/UEFI.
Większość informacji zawartych w niniejszym artykule pochodzi z [dokumentacji rEFInd][3], a pod tym
linkiem znajduje się [specyfikacja UEFI wersja 2.8 z marca 2019 roku][13], na którą przydałoby
się od czasu do czasu rzucić okiem. Dlatego też zanim przejdziemy do sedna artykułu jakim będzie
konfiguracja dysku SSD/HDD na potrzeby EFI/UEFI, to wypadałoby sobie parę tych istotniejszych
rzeczy wyjaśnić, tak by rozwiać ewentualne wątpliwości, które z pewnością się pojawią w późniejszym
procesie konfiguracyjnym

### Tablica partycji GPT

Jeśli nigdy nie instalowaliśmy linux'a w trybie EFI/UEFI i w zasadzie trzymaliśmy się mocno trybu
BIOS, to musimy poznać różnice między tymi dwoma trybami. Przede wszystkim zmianie ulega rodzaj
tablicy partycji. W przypadku trybu BIOS była to [tablica partycji MS-DOS][10] bardziej znana jako
MBR, a w przypadku EFI/UEFI mamy do czynienia z [tablicą partycji GPT][11] (GUID Partition Table).
Jest to dość istotna zmiana i nie damy rady uruchomić systemu w natywnym trybie EFI/UEFI, jeśli
nasz dysk twardy nie posiada tablicy partycji GPT. Niemniej jednak, specyfikacja EFI/UEFI wymaga,
by wparcie dla tablicy partycji MS-DOS/MBR było zachowane, co daje możliwość uruchomienia systemu
za sprawą modułu CSM (Compatibility Support Module).

### Partycja ESP

Drugą istotną zmianą jest zastosowanie [partycji ESP][12], która w zasadzie jest odpowiednikiem
linux'owej partycji BOOT. Niemniej jednak, te dwie partycje różnią się dość znacznie. Przede
wszystkim, systemem plików na partycji ESP jest określony przez specyfikację firmware EFI/UEFI,
która wymaga zastosowania FAT32 oraz dodatkowo FAT16/FAT12 dla nośników wymiennych. Producenci
sprzętu mogą jednak dorobić wsparcie też i dla innych systemów plików ale nie jest to od nich
wymagane. Dodatkowo, partycja ESP nie może rezydować w obrębie zaszyfrowanego kontenera LUKS czy
też być częścią LVM/RAID. Zatem ESP musi być fizyczną partycją dysku.

Partycja ESP efektywnie znosi potrzebę używania osobnej partycji BOOT pod linux ale wciąż niektóre
instalacje linux'a wymagają, by ta partycja była obecna. Nic jednak nie stoi na przeszkodzie, by
partycję ESP zamontować w `/boot/` zamiast w `/efi/` lub `/boot/efi/` .

Pierwszy sektor partycji ESP jest zawsze zarezerwowany na potrzeby starszych bootloader'ów, które
instalują się w MBR w trybie BIOS. Jeśli kod bootloader'a zostanie odnaleziony w tym pierwszym
sektorze partycji ESP, to firmware EFI/UEFI przekazuje mu kontrolę nad dalszą procedurą startu
systemu (tzw. legacy boot). W natywnym trybie EFI/UEFI, to co się znajduje w tym sektorze nie ma
najmniejszego znaczenia i firmware ignoruje ten sektor kompletnie.

W przeciwieństwie do BIOS, firmware EFI/UEFI jest w stanie odczytać bez większego problemu tablicę
partycji i umiejętnie operować na systemie plików, co umożliwia koegzystencję wielu bootloader'ów w
obrębie pojedynczej partycji. Dzięki takiemu rozwiązaniu możemy określić, który bootloader
zainicjować i tym samym, który system operacyjny uruchomić. Nie musimy przy tym ciągle przepisywać
bootloader'a w MBR, czy też innych częściach dysku, co zwykle prowadzi do problemów z konfiguracją
w przypadku instalowania wielu systemów na jednym dysku, w szczególności windows'a i linux'a.

Partycja ESP poza bootloader'ami jest w stanie przechowywać kernele, sterowniki, skrypty, czy też
zwykle aplikacje takie jak np. [memtest86][23]. W zasadzie firmware EFI/UEFI jest w stanie wykonać
dowolny kod PE/COFF ([Portable Executable][24]/[Common Object File Format][25]), który wrzucimy na
partycję ESP.

### EFI boot stub

Technicznie rzecz biorąc, to firmware EFI/UEFI jest w stanie uruchomić dowolny system, w tym też
sam kernel linux'a, bez dodatkowego menadżera rozruchu czy też bootloader'a. Taki scenariusz jest
możliwy za sprawą wbudowanego w firmware EFI/UEFI menadżera rozruchu. Ten cały mechanizm nosi nazwę
[EFI boot stub][14]. Niemniej jednak szereg warunków musi być spełnionych, by system nam się
uruchomił w ten sposób. Poza tym EFI boot stub jest bardzo prostym i ubogim mechanizm nieoferujący
praktycznie żadnej konfiguracji. Więcej na temat [EFI boot stub][15] można znaleźć w osobnym
artykule, jeśli kogoś interesuje ten temat.

### Secure Boot

Secure Boot ma na celu minimalizowanie szkód jakie mogą wyrządzić trudne do wykrycia i usunięcia
boot kit'y, które infekują bootloader komputera. By takiego wirusa wykryć i usunąć potrzebny jest
inny niezainfekowany system, a to ze względu na fakt, że taki wirus uruchamia się przed inicjacją
kernela i może w pełni się zamaskować, co w zasadzie nie jest możliwe w przypadku, gdyby ten sam
wirus się dostał do systemu po tym jak kernel już zostanie do pamięci RAM załadowany i uruchomiony.

Gdy Secure Boot jest włączony, to firmware EFI/UEFI sprawdza obecność sygnatur kryptograficznych w
każdym programie, który może zostać przez ten firmware załadowany do pamięci i wykonany. W
przypadku braku sygnatur lub też jeśli one nie pasują do tych przechowywanych w NVRAM maszyny (albo
są na czarnej liście), to firmware EFI/UEFI odmówi uruchomienia takiego programu i start systemu
się nie powiedzie. W ten sposób tylko bootloader'y podpisane zaufanymi kluczami mogą zostać
uruchomione, co znacznie poprawia bezpieczeństwo systemu, przynajmniej w teorii.

Oczywiście Secure Boot ma też wady i jedną taką poważniejszą jest utrudnienie życia linux'iarzom. Z
racji, że Microsoft wymaga by windows 8+ na desktopach i laptopach miał Secure Boot włączony, to
klucz, którym te systemy są podpisywane musi być zawarty w firmware EFI/UEFI ogromnej większości
komputerów. Klucze innych systemów operacyjnych z kolei niekoniecznie muszą być dodane do tego
firmware przez producenta sprzętu. Brak klucza z kolei uniemożliwi zweryfikowanie podpisu złożonego
pod bootloader'em, co efektywnie uniemożliwi uruchomienie naszego linux'a.

Warto zaznaczyć tutaj, że Microsoft współpracuje z Verisign w celu umożliwienia każdemu podpisania
jego bootloader'a kluczem MS (choć nie jest to ten sam klucz, którym jest podpisany windows).
Niemniej jednak, taka usługa kosztuje 99 doclów. Część dystrybucji linux'a skorzystała z tej opcji
ale pozostali nie mają zamiaru ze względu na uzależnienie się od Microsoft w kwestii tak prostej
jak odpalenie bootloader'a. Dlatego też powstały takie projekty jak [shim][17] i [PreLoader][18] od
Linux Foundation, które dystrybucje linux'a (jak i my sami) mogą wykorzystać by zaimplementować
wsparcie dla Secure Boot, choć nie do końca może być to trywialny proces.

Niektóre dystrybucje wspierają Secure Boot lepiej niż inne, zatem wybór określonej dystrybucji
będzie decydujący w kwestii czy ten mechanizm będzie włączony czy też nie. Jeśli dystrybucja,
której zamierzamy używać nie posiada wsparcia dla Secure Boot, to musimy ten [mechanizm
wyłączyć][9] w konfiguracji firmware EFI/UEFI.

![](/img/2020/03/001-secure-boot-efi-uefi-config-firmware.jpg#huge)

![](/img/2020/03/002-secure-boot-efi-uefi-config-firmware.jpg#huge)

Warto tutaj zaznaczyć fakt, że czasami interfejs EFI/UEFI może bardzo przypominać wygląd
tradycyjnego BIOS'u ale to tylko pozory. W zasadzie to od producenta sprzętu zależy jak to menu
konfiguracji firmware EFI/UEFI będzie wyglądać. W tym przypadku mamy do czynienia z trybem
tekstowym ale często można się spotkać z pełnym trybem graficznym.

W przypadku sporej części komputerów użytkownik ma pozostawioną opcję całkowitego wyłączenia
Secure Boot, ze względu na fakt, że Microsoft wymaga tego w przypadku windows 8. Z kolei w
przypadku windows 10, Microsoft już to obostrzenie złagodził jedynie do sugestii ale wciąż ogromna
większość producentów desktopów i laptopów wspiera wyłączenie Secure Boot.

Jeśli chodzi o systemy linux'owe, to wirusy nie są aż tak rozpowszechnione jak w przypadku windows
i nie do końca jest pewne czy Secure Boot daje w przypadku tych systemów jakąś wymierną korzyść.
Jeśli jednak zamierzamy uruchamiać więcej systemów niż jeden na danej maszynie (w tym windows), to
Secure Boot powinien być włączony, by chronić windows'a, a my powinniśmy poszukać sobie takiej
dystrybucji linux'a która wspiera ten mechanizm.

### Fast Boot

Fast Boot (inne nazwy czasem też są spotykane) to mechanizm, który ma za zadanie jak najbardziej
przyśpieszyć start systemu operacyjnego. Jest to możliwe do osiągnięcia zwykle kosztem pełnej
inicjacji sprzętu, np. nie zostaną zainicjowane urządzenia USB, w efekcie czego nie będziemy w
stanie uruchomić systemu z pendrive czy innych podobnych nośników. Jeśli nie planujemy uruchamiać
systemu z nośników USB, to raczej nie ma przeciwwskazań, by Fast Boot pozostał włączony. Niemniej
jednak, jeśli napotkamy jakieś problemy podczas startu systemu, to warto mieć ten mechanizm na
uwadze.

### Moduł CSM

Nie powinno się instalować różnych systemów w różnych trybach, np. windows'a w trybie EFI/UEFI, a
linux'a w trybie BIOS, bo tego typu mieszanka zawsze wygeneruje całą masę problemów. Jeśli mamy
jakiś system już zainstalowany w trybie EFI/UEFI, to kolejny system również w tym trybie trzeba
zainstalować i uruchamiać. Jeśli natomiast instalujemy pierwszy system, to go instalujmy w trybie
EFI/UEFI, chyba, że mamy pewien specyficzny sprzęt, który wymaga od nas trybu BIOS (np. sterowniki
sprzętu, zwykle karty graficznej).

Na wypadek tych wyżej nadmienionych problemów, [firmware EFI/UEFI ma zaszyty moduł CSM][27]
(wspomniany wcześniej), który umożliwia uruchomienie systemów operacyjnych opartych o tryb BIOS. W
ten sposób mamy niby zapewnione wstecznie kompatybilne wsparcie dla starszego trybu uruchamiania
systemów. Trzeba jednak pamiętać, że ten cały moduł CSM czyni proces startu systemu o wiele
bardziej skomplikowanym i podatnym na błędy, przez co nasz komputer może zacząć zachowywać się w
sposób bardzo nieprzewidywalny. Dlatego też zaleca się wyłączenie CSM w konfiguracji EFI/UEFI:

![](/img/2020/03/003-csm-module-efi-uefi-config-firmware.jpg#huge)

Niemniej jednak, jeśli ktoś już bardzo nalega na tryb BIOS, to naturalnie może ten moduł CSM sobie
włączyć.

## Przygotowanie dysku HDD/SSD

Mając odpowiednio skonfigurowany firmware EFI/UEFI, możemy przejść do kluczowego etapu, czyli do
przygotowania dysku. Niemniej jednak, zanim to uczynimy, to musimy sobie zadać pytanie czy
będziemy instalować linux'a na dysku, który już ma zainstalowany jakiś system operacyjny (zwykle
windows), czy też zaczynamy całą zabawę od zera ze świeżym dyskiem wyciągniętym wprost z pudełka?

Jeśli instalujemy linux'a na dysku, który wcześniej zawierał (lub dalej zawiera) windows'a, to nic
nie stoi na przeszkodzie by wykorzystać tę już istniejącą partycję ESP, bo nie powinno być z nią
żadnego problemu. Nie powinniśmy więc tworzyć czy formatować tej partycji na nowo, bo taki zabieg
wyczyści z niej wszystkie dane wliczając w to bootloader windows'a, bez którego nam ten system już
się później nie odpali.

Dobrze jest też zrobić binarny backup partycji ESP (przy pomocy `dd` ), tak na wszelki wypadek.
Gdyby pojawiły się jakieś problemy, to zawsze tę partycję będziemy w stanie przywrócić do stanu
pierwotnego i umożliwić start tym systemom, które mieliśmy wcześniej zainstalowane na dysku.

Czasami może się też zdarzyć tak, że będziemy chcieli przenieść dysk ze starego komputera do
nowszego i w takim przypadku będziemy musieli albo stworzyć na nowo tablicę partycji wraz ze
wszystkimi niezbędnymi partycjami, albo też [dokonać konwersji tablicy partycji MS-DOS na GPT][2].
Jeśli chodzi o konwersję tablicy partycji, to można ten proces przeprowadzić bez utraty danych
zgromadzonych na dysku. Po procesie konwersji trzeba będzie utworzyć nową partycję ESP dokładnie w
taki sam sposób jakbyśmy ją tworzyli na pustym nowym dysku twardym.

W tym przypadku jednak, odpowiedź na to powyżej zadane pytanie jest taka, że mamy czysty nośnik, na
którym będzie rezydować jeden system operacyjny, tj. Debian, wobec czego będziemy musieli utworzyć
partycję ESP i ją w pełni zainicjować.

Z reguły instalacja linux'a z płytki czy pendrive powinna być w pełni automatyczna, a cały ten
proces powinien przebiegać przy minimum interakcji z naszej strony. Instalator powinien rozpoznać z
jakim trybem (BIOS czy EFI/UEFI) mam do czynienia i wszystko dobrze skonfigurować za nas.

My nie będziemy tutaj bawić się w automaty instalacyjne, czy też w ogóle instalować linux'a.
Przygotujemy sobie za to dysk pod instalację dowolnego systemu spod znaku pingwina. Taki nośnik
będzie można w późniejszym czasie wykorzystać i zainstalować na nim (dowolną metodą jaką uznamy za
stosowne) każdą dystrybucję linux'a, która nas zainteresuje. Zatem najwyższy czas się wziąć do
pracy.

### Tworzenie tablicy partycji GPT za sprawą gdisk

Mając świeży dysk twardy, nie mamy na nim żadnej partycji czy nawet tablicy partycji. Wypadałoby
zatem ją stworzyć. Jak już wiemy, EFI/UEFI wymaga, by tablica partycji była w standardzie GPT i nie
możemy posłużyć się tablicą MS-DOS/MBR. Nową tablicę partycji możemy stworzyć wykorzystując do tego
celu graficzne narzędzie `gparted` , gdzie w zasadzie wszystko można sobie bardzo łatwo wyklikać.
Można także posłużyć się tekstowym `gdisk` i my skorzystamy z tej drugiej opcji.

Partycjonowanie dysku można przeprowadzić z poziomu działającego już systemu podłączając do niego
nośnik albo też możemy uruchomić na nowej maszynie płytkę czy pendrive z systemem live. Czasami w
takim systemie może brakować pakietu `gdisk` ale bez problemu powinniśmy być w stanie go
doinstalować Gdy już wszystko mamy na swoim miejscu, w terminal wpisujemy poniższe polecenia i
postępujemy według instrukcji:

    # gdisk /dev/sda
    GPT fdisk (gdisk) version 1.0.5

    Partition table scan:
      MBR: not present
      BSD: not present
      APM: not present
      GPT: not present

    Creating new GPT entries in memory.

    Command (? for help):

No jak widać wyżej, `gdisk` nie wykrył obecności żadnej tablicy partycji, zatem tworzymy nową
podając polecenie `o` , a na pytanie, które się pojawi, odpowiadamy twierdząco:

    Command (? for help): o
    This option deletes all partitions and creates a new protective MBR.
    Proceed? (Y/N): Y

Tablica partycji powinna zostać stworzona, choć póki co nie ma w niej jeszcze żadnego wpisu. Możemy
ten stan rzeczy zweryfikować wpisując w terminalu `p` :

    Command (? for help): p
    Disk /dev/sda: 30299520 sectors, 14.4 GiB
    Model: DT 101 G2
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): FB250EA3-2D37-414D-92E9-AEBA170D270D
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 30299486
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 30299453 sectors (14.4 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name

W tym powyższym wyjściu mamy informację o rozmiarze dysku, o tym, że możemy utworzyć do 128
partycji oraz, że partycje będą wyrównane do 1 MiB (2048 sektorów 512 bajtowych).

### Tworzenie partycji BOOT i ESP

Poza głównymi partycjami dla naszego linux'a, których nie będziemy tutaj tworzyć, potrzebne nam
będą dwie dodatkowe partycje: BOOT oraz ESP. Warto tutaj wspomnieć, że w przypadku instalatorów
linux'a, partycja ESP nie zawsze może być pokazywana, tak by można było sprecyzować jej manualnie
punkt montowania. Jeśli nie widzimy partycji ESP w przypadku instalowania linux'a z płytki/pendrive,
a jesteśmy pewni, że jest ona na dysku, to instalator powinien z niej zrobić użytek bez naszej
ingerencji i zamontować ją w ściśle określonym przez siebie miejscu. Zwykle będzie to katalog
`/boot/efi/` , `/efi/` lub `/boot/` .

W przypadku manualnego określania punktu montowania partycji ESP, zaleca się ustawić go jako
`/boot/efi/` , choć ja będę obstawał przy `/efi/` . Nie znalazłem żadnej informacji, która by
rozstrzygała czy `/boot/efi/` lub `/efi/` powinno się używać i mam wrażenie, że są to zwykłe
zamienniki i można sobie wybrać ten, który bardziej komuś odpowiada. Niemniej jednak, jeśli chodzi
o montowanie partycji ESP w `/boot/` , to w tym przypadku sprawa się może skomplikować, bo wtedy na
partycji ESP będą również rezydować obrazy kernela linux wraz z obrazami initrd/initrmfs oraz być
może też konfiguracją linux'owego bootloader'a. Dlatego też odradzałbym montowanie partycji ESP w
tym ostatnim miejscu, chyba, że wiemy co robimy.

Zatem potrzebne nam będą dwie osobne partycje. Partycja BOOT będzie miała punkt montowania jako
`/boot/` i będzie sformatowana systemem plików `ext4` . Zaś partycja ESP będzie montowana w
`/efi/` i sformatujemy ją systemem plików `fat32` .

Lokalizacja tych partycji na dysku nie ma większego znaczenia ale, że tworzymy je jako pierwsze, to
stwórzmy je na początku dysku. Zalecane rozmiary do partycji BOOT to 1-2 GiB, zaś dla [ESP to
minimum 512 MiB][5].

Wpisujemy w `gdisk` polecenie `n` i tworzymy kolejno dwie partycje:

    Command (? for help): n
    Partition number (1-128, default 1): 1
    First sector (34-30299486, default = 2048) or {+-}size{KMGTP}: 2048
    Last sector (2048-30299486, default = 30299486) or {+-}size{KMGTP}: +1G
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): 8300
    Changed type of partition to 'Linux filesystem'

    Command (? for help): n
    Partition number (2-128, default 2): 2
    First sector (34-30299486, default = 2099200) or {+-}size{KMGTP}:
    Last sector (2099200-30299486, default = 30299486) or {+-}size{KMGTP}: +512M
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): ef00
    Changed type of partition to 'EFI system partition'

W tej chwili tablica partycji powinna zwierać dwa wpisy. Zweryfikujmy to podając polecenie `p` :

    Command (? for help): p
    Disk /dev/sda: 30299520 sectors, 14.4 GiB
    Model: DT 101 G2
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): FB250EA3-2D37-414D-92E9-AEBA170D270D
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 30299486
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 27153725 sectors (12.9 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048         2099199   1024.0 MiB  8300  Linux filesystem
       2         2099200         3147775   512.0 MiB   EF00  EFI system partition

Dla lepszej czytelności możemy tym partycjom zmienić nazwy:

    Command (? for help): c
    Partition number (1-2): 1
    Enter name: linux boot

    Command (? for help): p
    Disk /dev/sda: 30299520 sectors, 14.4 GiB
    Model: DT 101 G2
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): FB250EA3-2D37-414D-92E9-AEBA170D270D
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 30299486
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 27153725 sectors (12.9 GiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048         2099199   1024.0 MiB  8300  linux boot
       2         2099200         3147775   512.0 MiB   EF00  EFI system partition

Poniżej są jeszcze szczegółowe informacje na temat każdej z tych partycji:

    Command (? for help): i
    Partition number (1-2): 1
    Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
    Partition unique GUID: 7C9A5843-6F1B-4A19-891D-3E90E066DF3A
    First sector: 2048 (at 1024.0 KiB)
    Last sector: 2099199 (at 1025.0 MiB)
    Partition size: 2097152 sectors (1024.0 MiB)
    Attribute flags: 0000000000000000
    Partition name: 'linux boot'

    Command (? for help): i
    Partition number (1-2): 2
    Partition GUID code: C12A7328-F81F-11D2-BA4B-00A0C93EC93B (EFI system partition)
    Partition unique GUID: 45B07135-E224-4D6D-B8EC-DFACD68CD98B
    First sector: 2099200 (at 1.0 GiB)
    Last sector: 3147775 (at 1.5 GiB)
    Partition size: 1048576 sectors (512.0 MiB)
    Attribute flags: 0000000000000000
    Partition name: 'EFI system partition'

Do tej pory wszystkie przeprowadzane zmiany były dokonywane w pamięci operacyjnej RAM. Tak
utworzoną tablicę partycji musimy teraz zapisać na dysk poleceniem `w` i tę operację będziemy
musieli potwierdzić:

    Command (? for help): w

    Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
    PARTITIONS!!

    Do you want to proceed? (Y/N): y
    OK; writing new GUID partition table (GPT) to /dev/sda.
    The operation has completed successfully.

#### Obecność flagi boot i esp na partycji ESP

Jeśli uważnie prześledziliśmy proces tworzenia partycji ESP, to możemy zauważyć, że nie
ustawiliśmy tam w zasadzie żadnych flag. Niemniej jednak, na necie można się spotkać z ustawianiem
flagi `boot` i `esp` na partycji ESP. Może i my ręcznie tych flag nie ustawialiśmy ale typ partycji,
który określiliśmy dla partycji ESP ( `EF00` ), zrobił to za nas automatycznie. Możemy się o tym
przekonać podglądając układ partycji w `parted` :

    # parted /dev/sda
    GNU Parted 3.3
    Using /dev/sda
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) unit s
    (parted) p
    Model: Kingston DT 101 G2 (scsi)
    Disk /dev/sda: 30299520s
    Sector size (logical/physical): 512B/512B
    Partition Table: gpt
    Disk Flags:

    Number  Start     End       Size      File system  Name                  Flags
     1      2048s     2099199s  2097152s               linux boot
     2      2099200s  3147775s  1048576s               EFI system partition  boot, esp

Warto tutaj jednak zaznaczyć, że flaga `esp` jest nadmiarowa i w zasadzie służy jedynie do
oznaczenia partycji ESP. Jeśli zaś chodzi o flagę `boot` , to w przypadku tablicy partycji GPT
[oznacza ona co innego][28] niż w przypadku MS-DOS/MBR (zmienia typ partycji na `EF00` ). Flagi
`boot` nie powinno się ustawiać na innej partycji niż partycja ESP.

#### System plików dla partycji BOOT i ESP

Patrycje mamy stworzone ale potrzebujemy jeszcze na nich dorobić system plików. Partycję BOOT
formatujemy standardowo jako `ext4`, natomiast partycję ESP musimy sformatować systemem plików
`fat32` . Systemy plików tworzymy przy pomocy `mkfs.ext4` i `mkfs.vfat` w poniższy sposób:

    # mkfs.ext4 -m 0 -L boot /dev/sda1
    mke2fs 1.45.5 (07-Jan-2020)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: 75436842-85f3-485d-ad2a-83cd1980693a
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done

    # mkfs.vfat -v -n ESP /dev/sda2
    mkfs.fat 4.1 (2017-01-24)
    Auto-selecting FAT32 for large filesystem
    /dev/sda2 has 64 heads and 32 sectors per track,
    hidden sectors 0x200800;
    logical sector size is 512,
    using 0xf8 media descriptor, with 1048576 sectors;
    drive number 0x80;
    filesystem has 2 32-bit FATs and 8 sectors per cluster.
    FAT size is 1024 sectors, and provides 130812 clusters.
    There are 32 reserved sectors.
    Volume ID is ee132b21, volume label ESP        .

### Pozostałe partycje na dysku

Oczywiście stworzenie tylko tych dwóch powyższych partycji, to nieco za mało na pełnowymiarową
instalację linux'a ale nam w tym artykule chodzi jedynie o przygotowanie dysku pod instalację
linux'a w trybie EFI/UEFI i naturalnie będą nam potrzebne dodatkowe partycje, by linux'a na nich
zainstalować.

Tak czy inaczej, [pozostałą część dysku HDD/SSD możemy podzielić według uznania][26]. Ja generalnie
tworzę jedną dużą partycję pod szyfrowany kontener LUKS, w którym implementuję LVM i tworzę kilka
dysków logicznych, którymi mogę zarządzać o wiele łatwiej niż fizycznymi partycjami. Także to jaki
wariant instalacyjny linux'a sobie wybierzemy, to już osobna sprawa.

## Menadżer rozruchu rEFInd dla trybu EFI/UEFI

Mając przygotowane partycje BOOT oraz ESP, musimy wgrać szereg plików na partycję ESP. Z tych
plików korzystać będzie firmware EFI/UEFI i są one niezbędne w procesie startu systemu. Najprostszym
sposobem na wgranie odpowiednich plików na partycję ESP jest [zainstalowanie menadżera rozruchu
rEFInd][21]. Z tym krokiem jednak trzeba uważać, bo potrzebny nam jest do tego system uruchomiony w
trybie EFI/UEFI. Jeśli nie dysponujemy żadnym takim systemem, a wszystkie te powyższe kroki
przeprowadzaliśmy na systemie w trybie BIOS, to nie powinniśmy dalej kontynuować z instalacją
rEFInd'a. Musimy pierw pozyskać odpowiednią płytkę czy pendrive live i uruchomić z tego nośnika
system w trybie EFI/UEFI.

### System live

Jeśli nie wiemy czy system live naszego linux'a został [odpalony w trybie EFI/UEFI][19], to łatwo
możemy ten stan rzeczy zweryfikować sprawdzając obecność katalogu `/sys/firmware/efi/` . Brak tego
katalogu świadczy o uruchomieniu systemu w trybie BIOS. Warto też rzucić okiem na log systemowy:

    # dmesg | grep -i EFI.
    [    0.000000] Command line: \\EFI\BOOT\vmlinuz-5.4.0-4-amd64 root=/dev/mapper/sd_debian-root rootfstype=ext4 ro add_efi_memmap initrd=\EFI\BOOT\initrd.img-5.4.0-4-amd64
    [    0.000000] efi: EFI v2.31 by Lenovo
    [    0.000000] efi:  ACPI=0xdaffe000  ACPI 2.0=0xdaffe014  SMBIOS=0xdae9e000
    [    0.030589] ACPI: UEFI 0x00000000DAFE0000 00003E (v01 LENOVO TP-G1    00001160 PTL  00000002)
    [    0.030595] ACPI: UEFI 0x00000000DAFDF000 000042 (v01 PTL    COMBUF   00000001 PTL  00000001)
    [    0.030634] ACPI: UEFI 0x00000000DAFD9000 0002A6 (v01 LENOVO TP-G1    00001160 PTL  00000002)
    [    0.272110] Kernel command line: \\EFI\BOOT\vmlinuz-5.4.0-4-amd64 root=/dev/mapper/sd_debian-root rootfstype=ext4 ro add_efi_memmap initrd=\EFI\BOOT\initrd.img-5.4.0-4-amd64
    [    0.791725] pci 0000:00:02.0: BAR 2: assigned to efifb
    [    0.802816] Registered efivars operations
    [    1.899497] efifb: probing for efifb
    [    1.899524] efifb: framebuffer at 0xe0000000, using 5628k, total 5625k
    [    1.899529] efifb: mode is 1600x900x32, linelength=6400, pages=1
    [    1.899533] efifb: scrolling: redraw
    [    1.899537] efifb: Truecolor: size=8:8:8:8, shift=24:16:8:0
    [    1.912565] fb0: EFI VGA frame buffer device
    [   16.953113] EFI Variables Facility v0.08 2004-May-17
    [   17.016091] pstore: Registered efi as persistent store backend
    [   18.669759] fb0: switching to inteldrmfb from EFI VGA

W zależności od maszyny, ten powyższy log może być mniej lub bardziej rozbudowany. Niemniej jednak,
brak wpisów z EFI będzie świadczył dobitnie o braku trybu EFI/UEFI i trzeba ten problem poprawić
zanim przejdziemy do instalacji rEFInd'a. Najprościej, to wejść w ustawienia BIOS'u i wymusić start
maszyny w trybie EFI/UEFI. Jeśli mamy błędnie przygotowany nośnik z systemem live, to linux nam się
nie powinien uruchomić.

#### Moduł efivars

Mając już uruchomiony system live w trybie EFI/UEFI (w tym przypadku jest to Ubuntu), upewnijmy się,
że mamy załadowany moduł `efivars` . Ten moduł często jest wbudowany na stałe i polecenie `lsmod`
może go nie zwrócić. Zawsze można posłużyć się `systool` (z pakietu `sysfsutils` ). Tak czy inaczej,
obecność katalogu `/sys/firmware/efi/efivars/` , a w nim plików, świadczy, że moduł działa.
Podobnie można sprawdzić czy system zamontował ten powyższy zasób w wyjściu polecenia `mount` :

    # mount | grep efivars
    efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,relatime)

Warto zwrócić uwagę, czy ten zasób jest podmontowany w trybie do zapisu. Jeśli jest on zamontowany
w trybie tylko do odczytu ( `ro` zamiast `rw` ), to trzeba go przemontować w tryb do zapisu:

    # mount -o remount,rw  /sys/firmware/efi/efivars

Brak modułu `efivars` sprawi, że rEFInd wyrzuci poniższą informację podczas instalacji:

    EFI variables are not supported on this system.

Nie będzie przez to w stanie dodać stosownego wpisu do tego wbudowanego w EFI/UEFI menadżera
rozruchu. Niemniej jednak, w dalszym ciągu będziemy w stanie uruchomić system bezpośrednio z dysku
twardego.

#### Instalacja rEFInd na partycji ESP

Na systemie live zwykle będzie brakować pakietu `refind` . Dlatego też trzeba będzie go ręcznie
doinstalować.

    # apt-get install refind

Przy instalacji pakietu zostanie wyrzucony monit dotyczący wgrania rEFInd'a na partycję ESP:

![](/img/2020/03/004-refind-config-debian-ubuntu-live-install-efi-uefi.png#huge)

Odpowiadamy tutaj przecząco, bo instalacją rEFInd'a zajmiemy się później sami.

Montujemy jeszcze partycję ESP w `/mnt/efi/` oraz partycję BOOT w `/mnt/boot/` :

    # mkdir /mnt/{efi,boot}
    # mount /dev/sda1 /mnt/boot/
    # mount /dev/sda2 /mnt/efi/

Teraz już wystarczy zainstalować rEFInd przy pomocy polecenia `refind-install` . Jako, że
instalujemy ten menadżer rozruchu z poziomu systemu live, to musimy dodatkowo określić parametr
`--root` , którego argument musi wskazywać na nadrzędny katalog w stosunku do tego, gdzie zostały
zamontowane partycje BOOT i ESP , w tym przypadku jest to folder `/mnt/`:

    # refind-install --root /mnt
    ShimSource is none
    Installing rEFInd on Linux....
    ESP was found at /mnt/efi using vfat
    Installing driver for ext4 (ext4_x64.efi)
    Copied rEFInd binary files

    Copying sample configuration file as refind.conf; edit this file to configure
    rEFInd.

    Creating new NVRAM entry
    rEFInd is set as the default boot manager.
    Creating /mnt/boot/refind_linux.conf; edit it to adjust kernel options.

Mamy tutaj informację, że partycja ESP została odnaleziona w katalogu `/mnt/efi/` oraz, że używa
systemu plików `vfat` ( `fat32` ). Pliki rEFInd'a zostały z powodzeniem skopiowane na partycję ESP
oraz została wgrana przykładowa konfiguracja, którą w późniejszym czasie trzeba będzie sobie
dostosować. W zasadzie to rEFInd powinien nam bez problemu wykryć linux'a jeśli tylko go w
późniejszym czasie zainstalujemy. Niemniej jednak, pewne aspekty wizualne, np. rozdzielczość ekranu,
mogą nie do końca spełniać nasze oczekiwania i wypadałoby przejrzeć plik konfiguracyjny rEFInd'a
(ten na partycji ESP) i go sobie dostosować wedle uznania.

Wyżej mieliśmy też informację, że rEFInd stworzył nowy wpis w NVRAM ( w
`/sys/firmware/efi/efivars/` ). Ten wpis możemy podejrzeć przy pomocy narzędzia `efibootmgr` :

    # efibootmgr -v
    BootCurrent: 0018
    Timeout: 0 seconds
    BootOrder: 0018,0000,0001,0002,0003,000D,0007,0008,0009,000A,000B,000C,000E,000F,0010,0011,0012
    ...
    Boot0018* rEFInd Boot Manager	HD(2,GPT,45b07135-e224-4d6d-b8ec-dfacd68cd98b,0x38b4000,0xff800)/File(\EFI\refind\refind_x64.efi)

Była też wzmianka o stworzeniu pliku `/boot/refind_linux.conf` , który zawiera konfigurację
parametrów dla kernel command line. Jako, że my będziemy określać wpisy bezpośrednio w głównym
pliku konfiguracyjnym zlokalizowanym na partycji ESP (pod `EFI/refind/refind.conf` ), to możemy ten
plik `/boot/refind_linux.conf` usunąć, tak by nie mieszał nam na późniejszym etapie przy
konfiguracji rEFInd'a.

Pomyślnie zainstalowany rEFInd powinien mieć poniższą strukturę katalogów na partycji ESP
(podmontowanej w katalogu `/mnt/efi/` ):

    # tree -L 3 /mnt/efi/
    /efi/
    └── EFI
        ├── refind
        │   ├── BOOT.CSV
        │   ├── drivers_x64
        │   ├── icons
        │   ├── keys
        │   ├── refind.conf
        │   └── refind_x64.efi
        └── tools

    6 directories, 3 files

#### Sterowniki systemu plików

Czasami może się zdarzyć tak, że nie zostaną wgrane [sterowniki od systemu plików][22] partycji
BOOT. W takim przypadku najprawdopodobniej coś namieszaliśmy w konfiguracji, bo rEFInd powinien bez
problemu ustalić system plików na partycji BOOT i dobrać odpowiedni sterownik, który następnie
przekopiuje do katalogu `drivers_x64/` -- po to właśnie zamontowaliśmy tę partycję BOOT:

    # ls -al /mnt/efi/EFI/refind/drivers_x64/ext4_x64.efi
    -rwxr-xr-x 1 root root 68170 Mar  2 07:13 /mnt/efi/EFI/refind/drivers_x64/ext4_x64.efi

W razie czego można usunąć zawartość partycji ESP i ponowić proces instalacyjny rEFInd z
wykorzystaniem polecenia `refind-install` . Brak sterownika uniemożliwi wczytanie obrazu kernela i
initrd bezpośrednio z partycji BOOT (menadżer rozruchu się zawiesi). Niemniej jednak, w dalszym
ciągu można bez problemu przekopiować obraz kernela i initrd na partycję ESP i po dostosowaniu
konfiguracji rEFInd'a uruchomić kernela bezpośrednio z partycji ESP.

#### Tryb fallback

Czasami rEFInd może się zainstalować w `EFI/BOOT/bootx64.efi` na partycji ESP. Obecność pliku
`bootx64.efi` w katalogu `EFI/BOOT/` oznacza, że rEFInd [zainstalował się w trybie fallback][20].
Przyczyn może być kilka, choć taką najpopularniejszą jest instalacja rEFInd pod systemem
działającym w trybie BIOS lub też brak modułu `efivars` .

#### Instalować czy nie instalować bootloader Grub2/Syslinux

Gdy instalacja rEFInd'a została zakończona powodzeniem, to w zasadzie mamy już przygotowany dysk do
pracy z systemem linux'a w trybie EFI/UEFI. Nie musimy przy tym instalować już dodatkowo
bootloader'a, bo rEFInd będzie w stanie załadować kernel i obraz initrd bez większego problemu,
czyli wykona pracę, którą standardowo w konfiguracji BIOS wykonywał bootloader Grub2 czy
syslinux/extlinux.

Niemniej jednak, w standardowej konfiguracji Debiana za każdym razem jak tylko będzie wypuszczana
nowa wersja kernela, to trzeba będzie przepisywać plik `refind.conf` , by dostosować numerki obecne
w nazwach kernela i initrd, które zostaną wgrane na partycję BOOT. Możemy jednak poinstruować nasz
[system by tworzył dowiązania symboliczne initrd.img i vmlinuz][8] do plików w katalogu `/boot/` ,
tak by tej konfiguracji rEFInd'a nie trzeba było w ogóle ruszać. Naturalnie jeśli chcielibyśmy
uruchomić konkretną wersję kernela, to stosowny wpis trzeba już dopisać do `refind.conf` . Warto
jest też rzucić okiem na [manual refind.conf][4] w celu łatwiejszego dostosowania konfiguracji.

#### Memtest86

Memtest86 to bardzo użyteczne narzędzie do testowania pamięci operacyjnej RAM komputera. Jeśli
chcielibyśmy mieć w rEFInd opcję, która nam umożliwi skanowanie pamięci, to musimy pobrać paczkę
[memtest86-usb.zip][6] ze strony projektu. Problem w tym, że ten obraz, który zostanie wypakowany,
jest przeznaczony do wgrania na pendrive. Jeśli chcielibyśmy, by rEFInd był w stanie uruchomić
memtest86 bezpośrednio, to z tego obrazu trzeba [wydobyć plik memtest86_x64.efi i umieścić go na
partycji ESP][7]. Cały ten proces został opisany w osobnym artykule.

Warto w tym miejscu wspomnieć, że memtest86 nie jest jedynym [dodatkiem][29], który możemy sobie
wgrać, tak by rEFInd zrobił z niego użytek.

## Instalacja linux'a

Tak przygotowany nośnik można bez problemu wykorzystać do instalacji dowolnej dystrybucji linux'a.
Bez znaczenia przy tym jest fakt czy będziemy [instalować system via debootstrap][16], czy przy
pomocy płytki instalacyjnej lub pendrive konkretnej dystrybucji.

## A co z windows

Ja nie przewidziałem w swoim nowym laptopie miejsca dla tego przestarzałego już systemu. Przez
ostatnich parę lat w zasadzie nie zdarzyło mi się bym zasiadł na windows 7, który miałem
zainstalowany obok mojego Debiana na starym laptopie. Ostatecznie windows 7 stracił wsparcie, a
10-ki i tak nie miałem zamiaru już nigdy nabywać. Także jeśli ktoś zastanawia się czy ten opisany
tutaj setup zadziała mu z windows, to odpowiedź z mojej strony jest taka, że powinien. Choć mnie w
zasadzie bardziej interesuje, czy ten setup zadziała z linux, a tu akurat odpowiedź jest
prosta -- zadziała.


[1]: https://en.wikipedia.org/wiki/GUID_Partition_Table
[2]: /post/konwersja-tablicy-partycji-ms-dos-na-gpt/
[3]: https://www.rodsbooks.com/refind/
[4]: https://www.rodsbooks.com/refind/configfile.html
[5]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions
[6]: https://www.memtest86.com/
[7]: /post/memtest86-dla-efi-uefi-i-refind/
[8]: /post/jak-przepisac-linki-initrd-img-old-i-vmlinuz-old-do-boot/
[9]: https://www.rodsbooks.com/efi-bootloaders/secureboot.html#disable
[10]: https://en.wikipedia.org/wiki/Master_boot_record
[11]: https://en.wikipedia.org/wiki/GUID_Partition_Table
[12]: https://en.wikipedia.org/wiki/EFI_system_partition
[13]: https://uefi.org/sites/default/files/resources/UEFI_Spec_2_8_final.pdf#G17.1019485
[14]: https://www.kernel.org/doc/Documentation/efi-stub.txt
[15]: /post/linux-kernel-efi-boot-stub-i-zaszyfrowany-debian-luks-lvm/
[16]: /post/instalacja-debiana-z-wykorzystaniem-debootstrap/
[17]: https://github.com/rhboot/shim
[18]: https://blog.hansenpartnership.com/uefi-secure-boot/
[19]: https://www.rodsbooks.com/refind/bootmode.html
[20]: https://www.rodsbooks.com/efi-bootloaders/fallback.html
[21]: https://www.rodsbooks.com/refind/installing.html
[22]: https://www.rodsbooks.com/refind/drivers.html
[23]: /post/memtest86-dla-efi-uefi-i-refind/
[24]: https://en.wikipedia.org/wiki/Portable_Executable
[25]: https://en.wikipedia.org/wiki/COFF
[26]: /post/jak-optymalnie-podzielic-dysk-hdd-ssd-na-partycje-pod-linux/
[27]: https://www.rodsbooks.com/efi-bootloaders/csm-good-bad-ugly.html
[28]: https://gist.github.com/leodutra/8779d468e9062058a3e90008295d3ca6
[29]: https://www.rodsbooks.com/refind/installing.html#addons
