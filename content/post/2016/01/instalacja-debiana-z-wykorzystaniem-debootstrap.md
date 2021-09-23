---
author: Morfik
categories:
- Linux
date: "2016-01-02T03:52:49Z"
date_gmt: 2016-01-02 02:52:49 +0100
published: true
status: publish
tags:
- debian
- luks
- lvm
- debootstrap
title: Instalacja debiana z wykorzystaniem debootstrap
---

Instalowanie debiana z wykorzystaniem `debootstrap` trochę się różni od instalacji z wykorzystaniem
instalatora. Chodzi generalnie o to, że wszystkie kroki instalacyjne trzeba przeprowadzać ręcznie.
Poza tym, cała konfiguracja będzie wymagać manualnego dostosowania. Plus tego rozwiązania jest
oczywisty, albowiem mamy całkowitą władzę nad tym co się w systemie znajdzie oraz jak będzie on
skonfigurowany. By mieć możliwość przeprowadzenia tego typu instalacji potrzebny będzie nam
działający system. Może to być płytka lub pendrive live z [Debianem][1] czy [Ubuntu][2]. Można też
wykorzystać już zainstalowany system operacyjny. Ważne jest tylko to, aby była możliwość
zainstalowania w takim systemie pakietu `debootstrap` , no i oczywiście wymagany jest dostęp do
internetu. W przeciwieństwie do instalatora debiana, mamy dostęp do graficznego środowiska, a w nim
do przeglądarki i w przypadku utknięcia gdzieś po drodze podczas instalacji, możemy sobie wygooglać
napotkane problemy nie przerywając przy tym prac instalacyjnych.

<!--more-->
## Live-cd

Będąc już zaopatrzonym w system live, startujemy go i odpalamy terminal. Wszystkie poniższe
polecenia muszą być przeprowadzane jako użytkownik root. Systemy live z reguły wykorzystują
mechanizm [sudo][3], dlatego też najlepiej wklepać w terminal `sudo su` . Na wypadek gdyby ktoś
potrzebował loginu i hasła użytkownika systemu live, to są to odpowiednio `user` i `live` .

Instalowany tutaj system będzie znajdował się na 2 partycjach fizycznych. Jedna z nich będzie
zaszyfrowana. Wewnątrz zaszyfrowanego kontenera LUKS zostaną umieszczone 4 voluminy LVM, po jednym
na `/` , `/home/` , `/tmp/` oraz przestrzeń wymiany `SWAP` . Druga partycja będzie przeznaczona na
pliki bootloader'a, tj. partycja `/boot/` . Będzie to klasyczna konfiguracja BIOS-MBR.

### Instalacja potrzebnych narzędzi

Do obsługi LVM będzie nam potrzebny pakiet `lvm2` . Z kolei do kontenerów LUKS będziemy potrzebować
pakietu `cryptsetup` . No i oczywiście potrzebny będzie nam sam pakiet `debootstrap` , który
zainstaluje nam minimalnego debiana. Pobieramy zatem potrzebne pakiety:

    $ sudo su
    # apt-get install debootstrap lvm2 cryptsetup
    # modprobe dm_mod

Prawdopodobnie nie będzie potrzeby ręcznego ładowania modułów. Jeśli jednak napotykamy problemy przy
wydawaniu poleceń, to dobrze jest sprawdzić przy pomocy `lsmod` czy moduł `dm_mod` jest załadowany.

### Przygotowywanie dysku

Przejdźmy zatem do partycjonowania dysku. W debianie jest kilka narzędzi, które mogą nam w tym
pomóc. Jeśli ktoś lubi narzędzia konsolowe, to może skorzystać z `fdisk`/`gdisk` . Nic nie stoi na
przeszkodzie, by również doinstalować w systemie live `gparted` i to w nim dostosować sobie
odpowiedni układ partycji. Sam proces partycjonowania nie powinien sprawić większego problemu,
dlatego też nie będę go opisywał tutaj.

### EFI/UEFI

W przypadku, gdy nasz komputer korzysta z firmware EFI/UEFI zamiast z przestarzałego już BIOS'u, to
trzeba będzie stworzyć dodatkowo partycję ESP i ją wstępnie zainicjować. Informacje o tym [jak
poprawnie przygotować partycję ESP pod instalację Debiana z EFI/UEFI][4] można znaleźć tutaj.

### Tworzenie szyfrowanego kontenera LUKS pod LVM

Pierwszym krokiem jaki musimy poczynić po stworzeniu partycji jest utworzenie zaszyfrowanego
kontenera LUKS, w którym umieścimy logiczne voluminy LVM pod instalowany system. Obecnie mamy do
wyboru dwie wersję LUKS: v1 i v2. Zalecane jest korzystanie z tej drugiej wersji, jako że ona
posiada wsparcie dla ochrony haseł z wykorzystaniem [Argon2][20], co znacząco utrudnia życie osobom
próbującym siłowego złamania hasła klastrami graficznymi.

Poniżej znajduje się linijka tworząca kontener LUKSv1:

    # cryptsetup luksFormat /dev/sda1 \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --iter-time 5000 \
      --label debian \
      --subsystem "" \
      --use-random \
      --verify-passphrase \
      --verbose

Niżej zaś jest linijka tworząca kontener LUKSv2 (można też ręcznie określić
`--pbkdf-force-iterations` , `--pbkdf-memory` oraz `--pbkdf-parallel`  ):

    # cryptsetup luksFormat /dev/sda1 \
      --type luks2 \
      --cipher aes-xts-plain64 \
      --key-size 512 \
      --hash sha512 \
      --pbkdf argon2i \
      --label debian \
      --subsystem "" \
      --use-random \
      --verify-passphrase \
      --verbose

Poniżej opis wykorzystanych parametrów:

  - `--cipher ` określa wykorzystany szyfr. W tym przypadku jest to `aes` w trybie `xts`. Z kolei
                `plain64` niweluje przecieki informacji z kontenerów o rozmiarze większym niż 2 TiB.
  - `--key-size` definiuje rozmiar klucza. W trybie `xts` jest on zawsze dwa razy większy, więc
                  faktyczny rozmiar to 256 bitów.
  - `--hash` odpowiada za użytą funkcję skrótu.
  - `--iter-time` odpowiada za czas (ms) spędzony na przetwarzaniu hasła (w przypadku LUKSv1).
  - `--pbkdf` określa funkcję PBKDF, która zostanie użyta do ochrony haseł. Można określić `pbkdf2`,
               `argon2i` oraz `argon2id` . Te dwie ostatnie są tylko dla LUKSv2.
  - `--label` i `--subsystem` mają za zadanie opisać kontener.
  - `--use-random` określa, które urządzenie będzie wykorzystane w celu dostarczenia liczb losowych.
                    W tym przypadku jest to `/dev/random` .
  - `--verify-passphrase ` weryfikuje hasło.
  - `--verbose` dostarcza więcej informacji.
  - `luksFormat` formatuje wskazane urządzenie i ustawia odpowiednie parametry w nagłówku kontenera.

Upewniamy się, że wskazaliśmy odpowiednie urządzenie i wciskamy enter:

    WARNING!
    ========
    This will overwrite data on /dev/sda1 irrevocably.

    Are you sure? (Type uppercase yes): YES
    Enter LUKS passphrase:
    Verify passphrase:
    Command successful.

Kontener został utworzony. Jego nagłówek możemy podejrzeć za pomocą `cryptsetup luksDump` :

    # cryptsetup luksDump /dev/sda1

Oczywiście można dowolnie tym nagłówkiem zarządzać ale nie będę opisywał tego w tym artykule. To
jest zagadnienie na inny tekst. W każdym razie, musimy jeszcze do tego kontenera uzyskać dostęp, by
mieć możliwość zapisywania w nim informacji. Otwieramy go zatem za pomocą tego poniższego polecenia:

    # cryptsetup luksOpen /dev/sda1 sda1_crypt

Nazwa otwieranego kontenera jest dowolna. Ja jednak wolę nazywać je po partycji, na której kontener
został utworzony i doczepiać na końcu `_crypt` . W taki sposób powstaje, np. `sda1_crypt` , bo
kontener tworzyliśmy na pierwszej partycji.

Po otwarciu, kontener powinien być widoczny przez system:

    # ls -al /dev/mapper/
    total 0
    drwxr-xr-x  2 root root      80 cze 11 09:12 .
    drwxr-xr-x 15 root root    4320 cze 11 09:12 ..
    crw-------  1 root root 10, 236 cze 11 09:01 control
    lrwxrwxrwx  1 root root       7 cze 11 09:13 sda1_crypt -> ../dm-0

### Tworzenie struktury LVM

Przechodzimy teraz do tworzenia voluminów logicznych. Za pomocą `pvcreate` wskażemy systemowi, by
używał danego dysku jako przestrzeń pod LVM. Przy czym, nie podajemy zaszyfrowanego `/dev/sda1` ,
tylko odszyfrowany `/dev/mapper/sda1_crypt` .

    # pvcreate /dev/mapper/sda1_crypt
      Physical volume "/dev/mapper/sda1_crypt" successfully created

Jeśli wszystko przebiegło pomyślnie, możemy sprawdzić jak wygląda powierzchnia przeznaczona pod LVM:

    # pvscan
      PV /dev/mapper/sda1_crypt         lvm2 [20.00 GiB]
      Total: 1 [20.00 GiB] / in use: 0 [0   ] / in no VG: 1 [20.00 GiB]

    # pvdisplay
      "/dev/mapper/sda1_crypt" is a new physical volume of "20.00 GiB"
      --- NEW Physical volume ---
      PV Name               /dev/mapper/sda1_crypt
      VG Name
      PV Size               20.00 GiB
      Allocatable           NO
      PE Size               0
      Total PE              0
      Free PE               0
      Allocated PE          0
      PV UUID               DDCYN1-Yggp-V52y-T0pL-mQbg-sHAZ-6Gtitd

Tworzymy grupę voluminów, w której zostaną utworzone wirtualne partycje:

    # vgcreate debian_laptop /dev/mapper/sda1_crypt
      Volume group "debian_laptop" successfully created

    # lvcreate -L 12G -n root debian_laptop
      Logical volume "root" created
    # lvcreate -L 5G -n home debian_laptop
      Logical volume "home" created
    # lvcreate -L 1G -n tmp debian_laptop
      Logical volume "tmp" created.
    # lvcreate -l +100%FREE -n swap debian_laptop
      Logical volume "swap" created

Wyjaśnienie opcji:

  - `-L` odpowiada za rozmiar voluminu.
  - `-n` to etykieta. Pod taką nazwą dysk będzie widziany w systemie.
  - `debian_laptop` to nazwa grupy voluminów, w której wirtualny dysk zostanie utworzony.
  - `-l` (małe L) ustawia rozmiar dysku w oparciu o `%` , a nie o GiB. W przypadku wybrania
    `+100%FREE` , zostanie użyta dostępna wolna przestrzeń.

W przypadku, gdy utworzyliśmy volumin, który nie spełnia naszych oczekiwań, to możemy go bez
problemu usunąć podając odpowiednią ścieżkę w `lvremove` :

    # lvremove /dev/debian_laptop/home
    Do you really want to remove active logical volume home? [y/n]: y
      Logical volume "home" successfully removed

W tej chwili przestrzeń pod LVM powinna być wykorzystana w 100%. Sprawdźmy zatem czy utworzyliśmy
wszystko tak jak tego chcieliśmy:

    # pvscan
      PV /dev/mapper/sda1_crypt   VG debian_laptop   lvm2 [20.00 GiB / 0    free]
      Total: 1 [20.00 GiB] / in use: 1 [20.00 GiB] / in no VG: 0 [0   ]

    # lvscan
      ACTIVE            '/dev/debian_laptop/root' [12.00 GiB] inherit
      ACTIVE            '/dev/debian_laptop/home' [5.00 GiB] inherit
      ACTIVE            '/dev/debian_laptop/tmp' [1.00 GiB] inherit
      ACTIVE            '/dev/debian_laptop/swap' [2.00 GiB] inherit

Tak prezentuje się obecnie sam dysk:

	# lsblk /dev/sda
	NAME                     MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
	sda                        8:0    0 232.9G  0 disk
	├─sda1                     8:1    0    20G  0 part
	│ └─sda1_crypt           254:0    0    20G  0 crypt
	│   ├─debian_laptop-root 254:1    0    12G  0 lvm
	│   ├─debian_laptop-home 254:2    0     5G  0 lvm
	│   ├─debian_laptop-tmp  254:3    0     1G  0 lvm
	│   └─debian_laptop-swap 254:4    0     2G  0 lvm
	├─sda2                     8:2    0     1G  0 part
	├─sda3                     8:3    0     1K  0 part
	├─sda4                     8:4    0    40G  0 part
	├─sda5                     8:5    0    30G  0 part
	└─sda6                     8:6    0 141.9G  0 part

Wygląda dobrze. Do zaszyfrowanych voluminów można się odwoływać przez pliki urządzeń w katalogach
`/dev/mapper/` , `/dev/nazwa_grupy/` oraz `/dev/dm-*` .

Polecenia od zarządzania LVM są bardzo proste w zapamiętaniu:

    tworzy   --   wyświetla info    --    usuwa
    lvcreate -- lvscan -- lvdisplay -- lvremove  # logiczne voluminy
    pvcreate -- pvscan -- pvdisplay -- pvremove  # przestrzeń pod LVM
    vgcreate -- vgscan -- vgdisplay -- vgremove  # grupa voluminów

Inne polecenia używane przy operacji na LVM można przejrzeć przez wpisanie `vg` , `pv` , `lv` +
klawisz Tab przyciśnięty dwukrotnie (automatycznie uzupełnianie).

### Tworzenie systemu plików

Mamy utworzone 4 voluminy wewnątrz zaszyfrowanego kontenera. Nie możemy ich jeszcze zamontować, a to
z tego względu, że potrzebują one systemu plików. Do ich utworzenia posłuży nam narzędzie
`mkfs.ext4` . Jest ich oczywiście więcej ale my ograniczymy się jedynie do systemu plików EXT4.
System plików tworzymy w poniższy sposób:

    # mkfs.ext4 -c 20 -m 0 -L root /dev/mapper/debian_laptop-root
    Filesystem UUID: 451861b7-fec1-49cb-9fec-33cc7a2fe5fb

    # mkfs.ext4 -c 20 -m 0 -L home /dev/mapper/debian_laptop-home
    Filesystem UUID: a530563d-e254-4b05-8f67-fe695941741b

    # mkfs.ext4 -c 20 -m 0 -L tmp /dev/mapper/debian_laptop-tmp
    Filesystem UUID: ad2501b1-0bf1-4dbe-b12e-26254bfdb3f7

    # mkfs.ext4 -c 20 -m 0 -L boot /dev/sda2
    Filesystem UUID: 7e58cc8f-0426-4b0b-81ea-06e8274f8af4

Użyte opcje odpowiadają za:

  - `-c` ustawia maksymalna liczbę montować dla danego systemu plików. Po przekroczeniu tej wartości
    nastąpi sprawdzanie systemu plików pod kątem ewentualnych błędów.
  - `-m` określa procent przestrzeni jaki zostanie zarezerwowany dla użytkownika root.
  - `-L` odpowiada za etykietę systemu plików.

Dodatkowo trzeba utworzyć przestrzeń wymiany:

    # mkswap /dev/mapper/debian_laptop-swap
    Setting up swapspace version 1, size = 2093052 KiB
    no label, UUID=61f16424-6ffd-43e0-b923-ba61d36c75e9

Te długie numerki zaczynające się od `UUID=` dobrze jest sobie zanotować, bo to w oparciu o nie
będziemy budować plik `/etc/fstab` . Oczywiście, w każdej chwili możemy te numerki wyciągnąć, np.
za pomocą narzędzia `tune2fs` . Nie bójmy się też, że przez przypadek nadpiszemy istniejący już
system plików. Narzędzie `mkfs` uprzednio zwróci nam odpowiedni komunikat, który poinformuje nas, że
próbujemy utworzyć system plików na partycji, która już go posiada, co wygląda mniej więcej tak:

    mke2fs 1.42.12 (29-Aug-2014)
    /dev/sda2 contains a ext4 file system labelled 'boot'
          last mounted on /boot on Wed Dec 30 13:34:34 2015
    Proceed anyway? (y,n) y

Musimy zatem świadomie potwierdzić ten krok.

### Montowanie systemów plików

Teraz już tylko trzeba zamontować wszystko w odpowiednich katalogach. Kolejność montowania jest
ważna. Jeśli korzystamy z zaszyfrowanych partycji i/lub LVM, to odwołujemy się do plików urządzeń w
katalogu `/dev/mapper/` :

    # mount /dev/mapper/debian_laptop-root /mnt
    # mkdir /mnt/{boot,home,tmp,media}
    # mount /dev/sda2 /mnt/boot/
    # mount /dev/mapper/debian_laptop-home /mnt/home/
    # mount /dev/mapper/debian_laptop-tmp /mnt/tmp

Aktywujemy także przestrzeń wymiany:

    # swapon /dev/mapper/debian_laptop-swap

W tej chwili nasz dysk powinien się prezentować tak jak poniżej:

    # lsblk /dev/sda
    NAME                     MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sda                        8:0    0 232.9G  0 disk
    ├─sda1                     8:1    0    20G  0 part
    │ └─sda1_crypt           254:0    0    20G  0 crypt
    │   ├─debian_laptop-root 254:1    0    12G  0 lvm   /mnt
    │   ├─debian_laptop-home 254:2    0     5G  0 lvm   /mnt/home
    │   ├─debian_laptop-tmp  254:3    0     1G  0 lvm   /mnt/tmp
    │   └─debian_laptop-swap 254:4    0     2G  0 lvm   [SWAP]
    ├─sda2                     8:2    0     1G  0 part  /mnt/boot
    ├─sda3                     8:3    0     1K  0 part
    ├─sda4                     8:4    0    40G  0 part
    ├─sda5                     8:5    0    30G  0 part
    └─sda6                     8:6    0 141.9G  0 part

Przygotowaliśmy sobie właśnie dysk pod instalację świeżego systemu. W przypadku gdybyśmy w
przyszłości musieli wgrać system na nowo, to możemy użyć tego ustawienia i zacząć instalację od
tego miejsca.

### Wykorzystanie debootstrap

Powinniśmy mieć już zainstalowane narzędzie `debootstrap`. W takim przypadku w katalogu
`/usr/share/debootstrap/scripts/` powinniśmy być w stanie zobaczyć jakie systemy mogą zostać
zainstalowane przy jego pomocy. Jako, że niedawno była aktualizacja wydania Debiana, starsze wersje
`debootstrap` mogą wrzucać błąd przy instalacji określonych wydań. W każdym razie, zawsze można
wybrać `testing` czy też i `sid` . Przejdźmy zatem do instalacji bardzo podstawowego systemu. By
tego dokonać, wpisujemy w terminal to poniższe polecenie:

    # debootstrap --verbose --arch amd64 sid /mnt http://ftp.pl.debian.org/debian
    I: Retrieving Release
    I: Retrieving Release.gpg
    I: Checking Release signature
    I: Valid Release signature (key id 126C0D24BD8A2942CC7DF8AC7638D0442B90D010)
    I: Retrieving Packages
    I: Validating Packages
    I: Resolving dependencies of required packages...
    I: Resolving dependencies of base packages...
    ...
    I: Checking component main on http://ftp.pl.debian.org/debian...
    I: Retrieving libacl1 2.2.52-2
    I: Validating libacl1 2.2.52-2
    ...
    I: Base system installed successfully.

Możemy też podejrzeć jakie pakiety zostaną zainstalowane dodając parametr `--print-debs` . Będą to
pakiety mające ustawiony priorytet `required` oraz `important` i raczej chcemy, by wszystkie te
pakiety znalazły się w naszym systemie. Sama instalacja nie trwała długo ale i też zbyt wiele nie
zostało zainstalowane. W każdym razie, bardzo podstawowy system mamy już wgrany. Co prawda jest
nieskonfigurowany i nie da się go jeszcze uruchomić ale popracujemy nad tym.

## Tworzenie środowiska pod chroot

By móc operować na nowym systemie, potrzebujemy odpowiednio [skonfigurować środowisko chroot][5]:

    # mount -o bind /dev/ /mnt/dev/
    # mount -o bind /dev/pts /mnt/dev/pts
    # mount -o bind /proc /mnt/proc
    # mount -o bind /sys /mnt/sys
    # chroot /mnt/ /bin/bash

Od tej chwili wszystkie polecenia będą wydawane w nowym systemie. Niemniej jednak, musimy się
upewnić, że mamy łączność, że światem. Wpisujemy zatem do terminala to poniższe polecenie:

    # ping wp.pl -c 3

Jeśli serwer odpowiada, to możemy przejść do wstępnej konfiguracji systemu.

### Konfiguracja menadżera pakietów

Na sam początek dostosujmy sobie źródła pakietów, które są przechowywane w pliku
`/etc/apt/sources.list` . Standardowe repozytoria Debiana zwierają zwykle po jednym wpisie `deb` i
`deb-src` , odpowiednio dla pakietów binarnych i źródłowych. No i też mamy w nich jedną sekcję
`main` . Dobrze jest nieco zmienić ten stan rzeczy i [przerobić wpisy w pliku sources.list][6] do
poniższej postaci:

    deb http://ftp.pl.debian.org/debian/ sid main non-free contrib
    #deb-src http://ftp.pl.debian.org/debian/ sid main non-free contrib

Zapisujemy plik i aktualizujemy listę repozytoriów:

    # apt-get update

Tworzymy teraz plik konfiguracyjny `/etc/apt/apt.conf` i dodajemy w nim tę poniższą treść:

    APT::Install-Recommends "false";
    APT::Install-Suggests "false";
    APT::AutoRemove::RecommendsImportant "false";
    APT::AutoRemove::SuggestsImportant "false";

Dodanie tych powyższych wpisów sprawi, że menadżery pakietów (zarówno `apt` jak i `aptitude` ) nie
będą instalowały automatycznie pakietów rekomendowanych i polecanych. Ma to na celu odchudzenie
instalacji. Jeśli ktoś jest ciekaw, to może pokusić się o dodanie [bardziej obszernej konfiguracji
apt do pliku apt.conf][7].

Zainstalujmy sobie też pakiet `aptitude` . Nie jest on wprawdzie wymagany i całą instalację systemu
możemy przeprowadzić za pomocą `apt` . Niemniej jednak, ja się przyzwyczaiłem do `aptitude` .

    # apt-get install aptitude

### Konfiguracja lokalizacji systemu (locales)

Na początek instalujemy pakiety odpowiedzialne za konfigurację lokalizacji systemu (język), układu
klawiatury oraz konfigurację terminala TTY:

    # aptitude install locales localepurge console-setup console-data kbd

Pakiet `locales` uchroni nas przed ciągłym wyrzucaniem komunikatów błędów podobnych do tego poniżej:

    perl: warning: Setting locale failed.
    perl: warning: Please check that your locale settings:
          LANGUAGE = (unset),
          LC_ALL = (unset),
          LANG = "en_US.UTF-8"
        are supported and installed on your system.
    perl: warning: Falling back to the standard locale ("C").
    locale: Cannot set LC_CTYPE to default locale: No such file or directory
    locale: Cannot set LC_MESSAGES to default locale: No such file or directory
    locale: Cannot set LC_ALL to default locale: No such file or directory

Z kolei pakiet `localepurge` ma za zadanie czyścić system ze zbędnych plików językowych.

W Debianie szereg pakietów można skonfigurować za pomocą narzędzia `dpkg-reconfigure` i tak też
postępujemy w przypadku w/w pakietów:

    # dpkg-reconfigure locales

Wybieramy kolejno: `en_US.UTF-8`, `pl_PL.UTF-8` -> `pl_PL.UTF-8` .

    # dpkg-reconfigure localepurge

Tutaj wybieramy: `en_US.UTF-8`, `pl_PL.UTF-8` , `en_US`, `pl_PL` > `Yes` > `Yes` .

Konfiguracja pakietu `locales` zaowocuje wygenerowaniem pliku `/etc/default/locale` zawierającego
poniższą treść:

    #  File generated by update-locale
    LANG="pl_PL.UTF-8"

#### Plik /etc/default/locale

W pliku `/etc/default/locale` są przechowywane zmienne językowe, tj. `LANG` oraz te z prefixem
`LC_` . Podczas generowania tego pliku, została utworzona jedynie zmienna `LANG` . Niemniej jednak,
do wyboru mamy sporo więcej opcji, przykładowo:

    LANG=
    LANGUAGE=
    LC_CTYPE=
    LC_NUMERIC=
    LC_TIME=
    LC_COLLATE=
    LC_MONETARY=
    LC_MESSAGES=
    LC_PAPER=
    LC_NAME=
    LC_ADDRESS=
    LC_TELEPHONE=
    LC_MEASUREMENT=
    LC_IDENTIFICATION=
    LC_ALL=

W przypadku, gdy został określony jedynie `LANG` , pozostałe zmienne są ustawiane na taką samą
wartość. By zmienić którąś ze zmiennych wypisanych wyżej, wystarczy dodać ją do tego pliku i
odpowiednio ustawić. Ja zwykle korzystam też z `en_US.UTF-8` . Dokładny [opis konfiguracji
lokalizacji systemu][8] został opisany już opisany w osobnym wpisie. Dlatego też jeśli coś z
powyższej konfiguracji jest niejasne, to zachęcam do zapoznania się z tym podlinkowanym tekstem.

#### Plik /etc/default/keyboard

Plik `/etc/default/keyboard` jest generowany przy konfiguracji pakietu `keyboard-configuration` :

    # dpkg-reconfigure keyboard-configuration

Wybieramy odpowiednio: `model` -> `Polish` -> `Right Alt (AltGr)` -> `Right Control` .

Plik `/etc/default/keyboard` możemy naturalnie sobie dostosować ręcznie. Trzeba tylko pamiętać, by
nie korzystać później z `dpkg-reconfigure` , bo wprowadzone przez nas zmiany przepadną. Poniżej
przykład mojego pliku:

    # KEYBOARD CONFIGURATION FILE

    # Consult the keyboard(5) manual page.

    XKBMODEL="logimel"
    XKBLAYOUT="pl"
    XKBVARIANT=""
    XKBOPTIONS="kpdl:dot,grp:alt_shift_toggle,lv3:ralt_switch,compose:rctrl,terminate:ctrl_alt_bksp,grp_led:scroll"
    BACKSPACE="guess"

[Konfiguracja ustawień klawiatury][9] została już opisana w osobnym wpisie. Jeśli któryś z
powyższych parametrów jest niejasny, to zachęcam do lektury podlinkowanego artykułu.

By ustawić odpowiedni układ klawiatury już podczas startu systemu, trzeba zmienić w pliku
`/etc/initramfs-tools/initramfs.conf` zmienną `KEYMAP=y` oraz wygenerować nowy initramfs za pomocą
`update-initramfs -u -k all`. Tego na razie nie musimy robić, przyda się dopiero w chwili jak
zainstalujemy kernela.

#### Plik /etc/default/console-setup

Wielu ludzi narzeka na [brak polskich znaków pod TTY][10]. Konfiguracja pakietu `console-setup`
powinna rozwiązać ten problem:

    # dpkg-reconfigure console-setup

Wybieramy kolejno: ustawienia terminala `UTF-8` -> `Latin2` -> `Terminus` -> `8x16` . W ten
sposób zostanie stworzony plik `/etc/default/console-setup` o poniższej treści:

    # CONFIGURATION FILE FOR SETUPCON

    # Consult the console-setup(5) manual page.

    ACTIVE_CONSOLES="/dev/tty[1-6]"

    CHARMAP="UTF-8"

    CODESET="Lat2"
    FONTFACE="Terminus"
    FONTSIZE="8x16"

    VIDEOMODE=

    # The following is an example how to use a braille font
    # FONT='lat9w-08.psf.gz brl-8x8.psf'

#### Plik /etc/timezone oraz /etc/localtime

W plikach `/etc/timezone` oraz `/etc/localtime` znajdują się ustawienia strefy czasowej.
Odpowiedzialny za nie jest pakiet `tzdata` :

    # dpkg-reconfigure tzdata

Wybieramy kolejno: `Europe` -> `Warsaw` . Czas powinien zostać zmieniony. Natomiast konfiguracja
obu plików powinna wyglądać mniej więcej tak:

    # cat /etc/timezone
    Europe/Warsaw

    # ls -al /etc/localtime
    lrwxrwxrwx 1 root root 35 2015-12-30 18:50:01 /etc/localtime -> ../usr/share/zoneinfo/Europe/Warsaw

Warto zauważyć, że plik `localtime` jest dowiązaniem do odpowiedniego pliku w katalogu
`/usr/share/zoneinfo/` . W standardowej konfiguracji Debiana tak nie jest i przydałoby się ten błąd
poprawić.

Dobrze jest także sprawdzić czy [w środowisku jest ustawiona zmienna TZ][11]. Choć jest wielce
prawdopodobne, że nie ma i będziemy musieli ją ręcznie wyeksportować przez dopisanie do pliku
`/etc/environment` tego poniższego wpisu:

    TZ="Europe/Warsaw"

### Instalacja i konfiguracja podstawowych narzędzi

Poniżej jest lista pakietów, które każdy system powinien moim zdaniem posiadać. Większość z tych
narzędzi to zwykle narzędzia konsolowe i przydają się w przypadku, gdy nie korzystamy ze środowiska
graficznego:

    # aptitude install vim mc htop iftop iotop powertop pv ccze ipcalc tree eject finger vnstat tty-clock \
    colordiff dfc ncdu gawk speedtest-cli tmux sudo \
    exim4-base bsd-mailx \
    debian-keyring keyutils busybox \
    zsh python-pygments command-not-found bash-completion \
    systemd libnss-myhostname policykit-1 libmtp-runtime libpam-cap libcap-ng-utils uuid-runtime

### Konfiguracja punktów montowania

By system działał prawidłowo, musi on wiedzieć gdzie mają być zamontowane określone zasoby. Jeśli
korzystamy z zaszyfrowanych kontenerów LUKS, to również musimy je uwzględnić w tym procesie. Jakby
nie patrzeć, taki kontener pierw trzeba otworzyć i dopiero wtedy uzyskujemy dostęp do systemu
plików, który może być zamontowany.

W przypadku wykorzystywania LVM i/lub LUKS, potrzebne nam będą dodatkowe narzędzia, które musimy
zainstalować:

    # aptitude install cryptsetup lvm2

#### Plik /etc/crypttab

W tym pliku musimy dopisać linijkę odpowiadającą za wskazanie zaszyfrowanej partycji, tak by system
podczas startu wiedział, którą partycje musi odszyfrować. Plik ma taką postać:

    # target name    source device                             key file  options
    sda1_crypt       UUID=1820485b-6587-47ab-baa4-761ecc104c25 none        luks

Wyjaśnienie użytych opcji:

  - `sda1_crypt` odpowiada za nazwę pod jaką ma być otwierany kontener LUKS.
  - `UUID` to numer identyfikacyjny kontneera. Można także podawać urządzenia w postaci `/dev/sda1`
    .
  - `none` oznacza brak pliku klucza, zatem ten kontener będzie otwierany przy pomocy zwykłego
    hasła.
  - `luks` definiuje typ kontenera.

UUID możemy odczytać przez `lsblk -o name,uuid` lub `blkid` , przykładowo:

    # lsblk -o name,uuid /dev/sda
    NAME                     UUID
    sda
    ├─sda1                   1820485b-6587-47ab-baa4-761ecc104c25
    │ └─sda1_crypt           DDCYN1-Yggp-V52y-T0pL-mQbg-sHAZ-6Gtitd
    │   ├─debian_laptop-root 451861b7-fec1-49cb-9fec-33cc7a2fe5fb
    │   ├─debian_laptop-home a530563d-e254-4b05-8f67-fe695941741b
    │   ├─debian_laptop-tmp  ad2501b1-0bf1-4dbe-b12e-26254bfdb3f7
    │   └─debian_laptop-swap 61f16424-6ffd-43e0-b923-ba61d36c75e9
    ├─sda2                   7e58cc8f-0426-4b0b-81ea-06e8274f8af4
    ├─sda3
    ├─sda4                   A0FCD66CFCD63BEA
    ├─sda5                   d314ed20-ffaf-4a18-98a7-91538e79d981
    └─sda6                   f3c10054-0583-4e10-937b-dcdc9a05a25c
      └─sda6                 b47e6dcd-924e-40fa-a8b1-7593419f86d7

Wartość `UUID` musi wskazywać na zaszyfrowaną partycję `/dev/sda1` . W przypadku, gdy dokonamy
jakichkolwiek zmian w pliku `/etc/crypttab` , trzeba zawsze wygenerować nowy initramfs (
`update-initramfs -u -k all` ), inaczej system nie odszyfruje odpowiedniej partycji. My nie mamy
jeszcze zainstalowanego kernela, także bez obaw.

#### Pliki /etc/fstab , /etc/mtab oraz /proc/mounts

Nie znalazłem żadnej satysfakcjonującej mnie metody generowania pliku `/etc/fstab` . Jest milusi
skrypt `genfstab` w Archlinux ale pakiety w Debianie są czasem zbyt stare, by ten skrypt chciał
działać prawidłowo. W przypadku, gdy nie używamy UUID do identyfikowania konkretnych systemów
plików, to możemy przekopiować zawartość pliku `/etc/mtab` do `/etc/fstab` i tylko usunąć zbędne
wpisy:

    # cat /etc/mtab > /etc/fstab

Oczywiście w naszym systemie jeszcze nie posiadamy pliku `/etc/mtab` , a ten z kolei jest
dowiązaniem do `/proc/mounts` . Podczas generowania initramfs przy instalacji kernela, zostanie
wyrzucony szereg błędów spowodowany brakiem tego skrótu. Dlatego też stwórzmy go sobie:

    # ln -s /proc/mounts /etc/mtab

Używanie UUID też nie jest konieczne. Jeżeli używamy jednego dysku na tej samej maszynie i nie
podpinamy żadnych nośników, z których można uruchomić inne systemy operacyjne, to UUID do niczego
nam się nie przyda, a partycja `/dev/sda1` będzie zawsze widoczna jako `/dev/sda1` i nigdy nie
ulegnie zmianie. W przypadku posiadania kilku dysków, nazwy urządzeń mogą ulec zmianie, zwłaszcza
przy innym fizycznym podłączeniu nośnika na płycie głównej. Wtedy z `/dev/sda1` może się
nieoczekiwanie zrobić `/dev/sdb1` i nie mając UUID, system się nie odpali i trzeba będzie te numerki
ręcznie pozmieniać. Ja korzystam z wielu urządzeń dlatego u mnie UUID jest niezbędne.

Oczywiście mamy do dyspozycji graficzny system live i odszukanie przykładowego pliku `fstab` na
sieci nie powinno stanowić trudność. Zmieniamy go nieco i wrzucić do naszego systemu. Ja jednak
postanowiłem stworzyć ten plik samemu. Może na pierwszy rzut oka wygląda skomplikowanie ale gdy się
zacznie go pisać, to okazuje się, że to wcale takie trudne nie jest. Ja i tak preferuję skrypt
`genfstab` , choćby ze względu oszczędności czasu. Mój plik `/etc/fstab` wygląda mniej więcej tak:

    # /etc/fstab: static file system information.
    #
    # file system                           mount point     type      options               dump  pass

    UUID=451861b7-fec1-49cb-9fec-33cc7a2fe5fb   /                 ext4  defaults,errors=remount-ro,commit=10      0 1
    #UUID=ad2501b1-0bf1-4dbe-b12e-26254bfdb3f7  /tmp              ext4  defaults,errors=remount-ro,commit=10      0 0
    UUID=7e58cc8f-0426-4b0b-81ea-06e8274f8af4   /boot             ext4  defaults,errors=remount-ro,commit=10      0 2
    UUID=a530563d-e254-4b05-8f67-fe695941741b   /home             ext4  defaults,errors=remount-ro,commit=10      0 2
    UUID=61f16424-6ffd-43e0-b923-ba61d36c75e9   swap              swap  defaults,pri=10         0 0

    /dev/sr0    /media/cdrom      udf,iso9660 noauto,ro,user,exec           0 0

Wyżej mamy określony wpis od napędu CD/DVD, dlatego też trzeba jeszcze utworzyć odpowiednie pliki i
katalogi pod to urządzenie:

    # cd /media/
    media/# mkdir cdrom0
    media/# ln -s cdrom0 cdrom
    media/# cd /
    # ln -s media/cdrom

Dobrze jest mieć dodane `udf,iso9660` zamiast jedynie `iso9660`. A to z tego względu, że `udf` jest
rozszerzeniem `iso9660` obchodzącym mnóstwo jego ograniczeń takich jak, np. wielkość zapisywanego
pliku (max 2 GiB).

#### Plik /etc/initramfs-tools/conf.d/resume

W przypadku korzystania z hibernacji, potrzebna nam będzie przestrzeń wymiany. Sama obecność SWAP'a
nie sprawi w jakiś magiczny sposób, że system będzie w stanie się z powodzeniem zahibernować i
odhibernować. Musimy poinformować system, o tym, gdzie przechowywany jest zrzut pamięci i za to
odpowiada plik `/etc/initramfs-tools/conf.d/resume` . Wygląda on mniej więcej tak:

    RESUME=/dev/mapper/debian_laptop-swap

Po edycji powyższego pliku trzeba wygenerować nowy initramfs ( `update-initramfs -u -k all` ).

### Konfiguracja sieci

Sieć w obecnych czasach to podstawa i żaden linux się bez niej obejść nie może. W Debianie mamy
kilka rozwiązań dotyczących konfiguracji sieci. Jedno z nich opiera się o systemd ale póki co nie
będę go tutaj opisywał. Skupimy się tutaj na tradycyjnym rozwiązaniu, które większość z nas zna z
czasów, gdy wszystkie dystrybucje korzystały domyślnie z sysvinit, tj pakietu `ifupdown` .

#### Plik /etc/network/interfaces

W przypadku używania standardowych ustawień sieci, trzeba stworzyć plik `/etc/network/interfaces`
lub odkomentować w nim odpowiednie sekcje. Tak wygląda mój plik:

    # interfaces(5) file used by ifup(8) and ifdown(8)
    # Include files from /etc/network/interfaces.d:
    source /etc/network/interfaces.d/*

    # We always want the loopback interface.
    #
    auto lo
    iface lo inet loopback

    # To use dhcp:
    #
    # auto eth0
    allow-hotplug eth0
    iface eth0 inet dhcp

    # An example static IP setup: (broadcast and gateway are optional)
    #
    #auto eth0
    #iface eth0 inet static
    #   address 10.1.3.61
    #   network 10.1.0.0/16
    #   netmask 255.255.0.0
    #   broadcast 10.1.255.255
    #   gateway 10.1.255.253

Mamy tutaj konfigurację dla DHCP, jak i również blok z wpisami statycznymi. Obecnie większość sieci
działa w oparciu o DHCP i nie musimy tutaj zbytnio nic przestawiać.

#### Plik /etc/hostname

Nazwa hosta jest trzymana w pliku `/etc/hostname` . To ta nazwa będzie nas później identyfikować w
sieci. Ważne jest, by odpowiednio uzupełnić później plik `/etc/hosts` . Tak czy inaczej, nazwę hosta
ustawiamy w poniższy sposób:

    # echo morfikownia > /etc/hostname

#### Plik /etc/hosts

Ten plik robi za swojego rodzaju lokalny resolver i bardzo fajnie się za jego pomocą blokuje reklamy
na necie. My jednak nie będziemy sobie tym na razie głowy zawracać i dodamy tutaj tylko te bardzo
podstawowe wpisy:

    127.0.0.1 localhost
    # 127.0.1.1 <host_name>.<domain_name> <host_name>
    # 127.0.1.1 morfikownia.mhouse morfikownia
    10.1.3.61 morfikownia.mhouse morfikownia

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ff02::3 ip6-allhosts

Zastanawiający jest wpis od `127.0.1.1` . Zgodnie z tym co wyczytałem na necie, ten adres jest
przypisywany automatycznie w przypadku braku stałego adresu IP eliminując tym bugi w pewnych
aplikacjach, np. z rodziny GNOME. W przypadku, gdy się posiada stały adres IP, to w miejsce
`127.0.1.1` wpisujemy ten adres.

Druga sprawa dotyczy tego co znajduje się po adresie IP. Z reguły jest tam jedno pole wskazujące na
hostname. U mnie jest jeszcze ustawiona nazwa domeny. Do danego hosta można się odwoływać przez
adres IP, hostname albo przez hostname.domainname . Oczywiście można zrezygnować ze środkowego pola
i zostawić tylko relacje ip <-> hostname.

#### Plik /etc/resolv.conf

W pliku `/etc/resolv.conf` jest trzymana konfiguracja resolvera. To na podstawie wpisów w tym pliku,
system wie gdzie nas kierować ilekroć próbujemy odwiedzić jakąś domenę. Również i w tym przypadku
mamy możliwość korzystania z mechanizmu oferowanego przez systemd ale nie będziemy go tutaj
opisywać. Skupimy się na standardowym mechanizmie rozwiązywania nazw jaki oferuje nam Debian.

Jedyne czego od nas się oczekuje w przypadku konfiguracji systemowego resolvera, to zdefiniowanie
odpowiednich adresów IP. Tak wygląda mój plik:

    nameserver 208.67.222.222
    nameserver 208.67.220.220

W przypadku konfiguracji DHCP, wpisy w pliku `/etc/resolv.conf` będą automatycznie dostosowywane.
Jeśli korzystamy z innego providera DNS, to możemy poinstruować klienta DHCP, by ignorował wpisy
odpowiedzialne za serwery DNS. W tym celu edytujemy plik `/etc/dhcp/dhclient.conf` i odpowiednio
dostosowujemy zapytanie DHCP. Tak wygląda to standardowe:

    request subnet-mask, broadcast-address, time-offset, routers,
            domain-name, domain-name-servers, domain-search, host-name,
            dhcp6.name-servers, dhcp6.domain-search, dhcp6.fqdn, dhcp6.sntp-servers,
            netbios-name-servers, netbios-scope, interface-mtu,
            rfc3442-classless-static-routes, ntp-servers;

Jeśli nie chcemy korzystać z serwerów DNS od ISP, to usuwamy pozycję `domain-name-servers` .
Powyższe zapytanie można sobie oczywiście dostosować wedle uznania.

#### Cache i szyfrowanie zapytań DNS

Istnieje także opcja zaszyfrowania ruchu DNS. Niemniej jednak, ten temat wykracza poza ramy tego
artykułu. Jeśli kogoś ciekawi ta opcja, to zachęcam do przeczytania wpisu na temat [szyfrowania
zapytań DNS przy użyciu dnscrypt-proxy][12]. Zachęcam także do rzucenia okiem na wpis traktujący o
[cache zapytań DNS][13] jak i również polecam artykuł o [konfiguracji cache DNS w przeglądarce
Firefox][14].

#### Sieć bezprzewodowa (WiFi)

Jeśli interesuje nas łączność bezprzewodowa, to jak najbardziej Debian jest nam ją w stanie
zapewnić. Jedyne czego potrzebujemy to odpowiedniego firmware dla naszej karty WiFi no i oczywiście
stosownej konfiguracji interfejsu. Dodatkowo będziemy musieli się w takiej bezprzewodowej sieci
uwierzytelnić i do tego celu będziemy potrzebować pakiet `wpasupplicant` . Nie będę tutaj opisywał
[jak skonfigurować sobie sieć WiFi na Debianie przy pomocy wpa_supplicant][15], bo to zostało
zrobione w osobnym artykule.

### Synchronizacja czasu

We wcześniejszych swoich wydaniach, Debian do synchronizacji czasu wykorzystywał demona NTP, który
był dostarczany wraz z pakietem `ntp` . Po migracji na systemd, nie ma zbytnio potrzeby instalowania
tego pakietu, a samą synchronizację możemy włączyć w opcjach systemd. Ważne jest jednak, by ten krok
przeprowadzić już po całym procesie instalacji systemu.

    # timedatectl set-ntp 1

    # timedatectl status | grep -i ntp
    NTP synchronized: yes

### Instalacja kernela

Przyszła pora na zainstalowanie kernela. To jakie pakiety zostaną tutaj zainstalowane zależy od
posiadanego sprzętu. W dużej mierze będą decydować sterowniki/firmware do tych wszystkich urządzeń,
z których składa się nasz komputer. Poniżej są te pakiety, które są wymagane przez podzespoły mojej
maszyny:

    # aptitude install firmware-linux-free firmware-realtek firmware-brcm80211 firmware-atheros \
    intel-gpu-tools intel-microcode iucode-tool \
    v86d

Do tych powyższych należy także dodać pakiety z kernelem oraz szeregiem jego modułów. Podobnie jak w
przypadku firmware, moduły również mogą być inne w sporej części przypadków. Poniżej są pakiety,
które ja zwykłem instalować u
    siebie:

    # aptitude install initramfs-tools sysdig sysdig-dkms xtables-addons-common xtables-addons-dkms \
    conntrack ipset \
    linux-{image,headers}-amd64

### Bootloader

Dokładny [opis instalacji bootloader'a extlinux w Debianie][16] został już opisany w osobnym wpisie.
Zachęcam zatem do przeczytania tego artykułu, bo bez bootloader'a ani rusz.

### Firewall

Przydałoby się jeszcze mieć prosty filtr pakietów, który zrzuci wszystkie próby nowych połączeń
sieciowych pod adresem naszej maszyny. [Projekt takiego firewall'a w oparciu o usługę systemd][17]
został już opisany w osobnym artykule i również zalecam zapoznanie się z nim.

Jeśli nie korzystamy z systemd albo też nie chcemy pisać plików usług pod zaporę systemową, to
zawsze można też skorzystać z pakietu `iptables-persistent` , którego celem jest ładowanie reguł dla
iptables podczas fazy boot.

### Hasła, użytkownicy oraz grupy

Mamy już wgrany i wstępnie skonfigurowany system podstawowy oraz bootloader. Mamy także
zainstalowanego kernela. Możemy więc przejść już do ostatniej fazy przeprowadzonej w `chroot` z
poziomu systemu live. Pozostało nam skonfigurowanie użytkowników.

Na sam początek ustawmy hasło użytkownikowi root. Robimy to przez wydanie w terminalu polecenia
`passwd` bez argumentów:

    # passwd

Dodamy jeszcze zwykłego użytkownika. Konfigurujemy mu także grupy oraz ustawiamy mu hasło:

    # groupadd -g 1000 morfik
    # useradd -m -g morfik -G users,sudo,cdrom,floppy,dip,plugdev -s /bin/bash morfik
    # passwd morfik

Parametr `-g` przy `groupadd` ustawia id grupy. Domyślnie są one numerowane od 1000 w górę.
Informacje na temat grup w systemie są przechowywane w pliku `/etc/group` , użytkownicy zaś w
`/etc/passwd` , a hasła w `/etc/shadow` .

Poniżej jest także wyjaśnienie opcji użytych w `useradd` :

  - `-m` tworzy katalog domowy, domyślnie w `/home/` .
  - `-g` określa grupę główną.
  - `-G` definiuje grupy dodatkowe.
  - `-s` ustawia shell.
  - `morfik` nadaje nazwę dla konta.

### Tasksel oraz priorytety pakietów

I to w zasadzie by było tyle jeśli chodzi o bardzo podstawowy system operacyjny. Gdybyśmy
zresetowali teraz maszynę, odpali się, a to najważniejsze. Nasz system za dużo jeszcze nie potrafi.
Możemy za to rozszerzyć jego funkcjonalność przez instalację dodatkowych pakietów. Narzędzie
`tasksel` potrafi instalować pakiety o określonych priorytetach. Priorytety do wyboru jakie mamy to:
`required`, `important` (te dwa zostały zainstalowane przez `debootstrap` ), `standard`, `optional`
i `extra`. Instalator Debiana zwykle dodaje jeszcze paczki z priorytetem `standard`. Możemy je
doinstalować za pomocą:

    # tasksel install standard

Być może zdarzy się tak, że to powyższe polecenie zwróci błąd: `tasksel: apt-get failed (100)` . Nie
wiem dlaczego tak się dzieje ale zawsze możemy te pakiety zainstalować za pomocą `aptitude` w
poniższy sposób:

    # aptitude install ~pstandard ~prequired ~pimportant

## Kończenie instalacji

Możemy już wyjść z chroot'a i zresetować maszynę. System, który wgraliśmy powinien się bez problemu
odpalić. Jest to czysty tryb tekstowy. Czeka nas jeszcze instalacja środowiska graficznego lub też
[Xserver'a w połączeniu z prostym menadżerem okien][18], np. [openbox][19].


[1]: https://www.debian.org/CD/live/index.pl.html
[2]: https://www.ubuntu.com/download/desktop/try-ubuntu-before-you-install
[3]: https://pl.wikipedia.org/wiki/Sudo
[4]: /post/jak-przygotowac-dysk-pod-instalacje-debian-linux-z-efi-uefi/
[5]: /post/przygotowanie-srodowiska-chroot-do-pracy/
[6]: https://github.com/morfikov/files/tree/master/configs/etc/apt
[7]: /post/konfiguracja-apt-i-aptitude-w-pliku-apt-conf/
[8]: /post/jezyk-polski-w-srodowisku-graficznym/
[9]: /post/klawiatura-i-jej-konfiguracja-pod-debianem/
[10]: /post/polskie-znaki-pod-tty/
[11]: /post/zmienna-tz-w-srodowisku-linuxowym/
[12]: /post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/
[13]: /post/cache-dns-buforowania-zapytan/
[14]: /post/konfiguracja-cache-dns-w-firefoxie/
[15]: /post/konfiguracja-polaczenia-wifi-pod-debianem/
[16]: /post/instalacja-i-konfiguracja-bootloadera-extlinux/
[17]: /post/firewall-na-linuxowe-maszyny-klienckie/
[18]: /post/konfiguracja-xservera-na-debianie-xorg/
[19]: /post/menadzer-okien-openbox/
[20]: https://en.wikipedia.org/wiki/Argon2
