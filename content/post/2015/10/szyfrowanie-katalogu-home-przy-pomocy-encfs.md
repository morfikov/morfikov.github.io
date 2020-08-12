---
author: Morfik
categories:
- Linux
date: "2015-10-19T21:54:02Z"
date_gmt: 2015-10-19 19:54:02 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- moduły-pam
title: Szyfrowanie katalogu /home/ przy pomocy encfs
---

Wielu ludzi uważa, że szyfrowanie całego dysku jest zbędne i pozbawione większego sensu, no bo
przecie "system nie zawiera żadnych wrażliwych danych, które by wymagały szyfrowania". Nie będę się
tutaj spierał co do tego punktu widzenia, bo raczej wszyscy znają moje zdanie na temat "danych
wymagających szyfrowania" i skupię się tu raczej na tym jak troszeczkę podratować niepełne
szyfrowanie, które ludzie, nie wiedząc czemu, są bardziej skłonni stosować, niż cały ten full disk
encryption.

Narzędzie `encfs` nie przeszło pomyślnie [audytu bezpieczeństwa](https://defuse.ca/audits/encfs.htm)
, a to z takiego powodu, że projekt nie był rozwijany przez szereg lat. [Obecnie jest on w rekach
społeczności](https://github.com/vgough/encfs) i to od niej będzie zależeć czy te wykryte błędy
zostaną poprawione.

<!--more-->
## Katalog domowy

Zwykle ludzie próbują szyfrować pojedyncze foldery czy pliki, które są dla nich dość istotne. Takim
folderem na pewno jest katalog `/home/$USER` ale wielu ludzi nie ma pojęcia jak się zabrać do jego
zaszyfrowania, poza tym samo zaszyfrowanie to nie wszystko. Trzeba ten folder jeszcze otworzyć przy
logowaniu się do systemu. Ponadto, potrzebne są również dodatkowe zabezpieczenia, które uchronią nas
przed wyciekiem wrażliwych informacji.

Większość dystrybucji linux'owych ma opcję zaszyfrowania katalogu `/home/$USER` w instalatorze i z
tego co zauważyłem, to ta opcja ma zastosowanie jedynie dla głównego użytkownika systemu, czyli tego
tworzonego w instalatorze. Pozostali użytkownicy już nie mają zaszyfrowanych swoich katalogów
domowych. Jeśli posiadamy kilku użytkowników w systemie i chcemy dać każdemu z nich możliwość
zaszyfrowania swojego katalogu w `/home/` , musimy nauczyć się jak to robić ręcznie i zrezygnować z
dobrodziejstw oferowanych przez graficzne instalatory. Do tego celu będzie nam potrzebnych kilka
pakietów: `encfs` , `libpam-encfs` , `libpam-mount` oraz `fuse` .

### Moduł fuse

Ważne jest by załadować moduł `fuse` na starcie systemu oraz, by dodać użytkownika do grupy `fuse` .
Moduł dodajemy do pliku `/etc/modules` , a użytkownika za pomocą poniższego polecenia:

    # adduser morfik fuse

Zmiany wejdą w życie po ponownym zalogowaniu się do systemu.

### Pliki konfiguracyjne

Przechodzimy teraz do edycji kilku plików konfiguracyjnych. Z racji tego, że kolejność linijek w
tych plikach jest ważna, powklejam tutaj całe pliki, usuwając tylko komentarze, tak by można było
się łatwiej zorientować co dodać do swoich plików konfiguracyjnych.

Plik `/etc/security/pam_encfs.conf` :

    drop_permissions
    encfs_default
    fuse_default nonempty
    -    /home/.encfs    -    -v    allow_root

Plik `/etc/security/pam_env.conf` :

    ICEAUTHORITY DEFAULT=/tmp/.ICEauthority_@{PAM_USER}

Plik `/etc/fuse.conf` :

    user_allow_other

Plik `/etc/pam.d/common-auth` :

    auth    sufficient      pam_encfs.so
    auth    [success=1 default=ignore]      pam_unix.so use_first_pass nullok_secure
    auth  requisite               pam_deny.so
    auth  required                pam_permit.so
    auth  optional    pam_mount.so

### Backup katalogu /home/

Robimy teraz backup katalogu `/home/` :

    # mkdir /home.original/
    # cp -a /home/ /home.original/

Edytujemy także plik `/etc/passwd` zmieniając w nim katalog użytkownika:

    morfik:x:1000:1000:Morfik,,,:/home.original/morfik:/bin/bash

I relogujemy się. W tej chwili ustawienia konta są czytane z nowej lokalizacji i możemy opróżnić
stary katalog `/home/` . Gdy jest on już pusty, tworzymy w nim kilka folderów:

    # mkdir -p /home/.encfs/morfik /home/morfik
    # chown morfik:morfik /home/.encfs/morfik/ /home/morfik/

Powyższe polecenia trzeba powtórzyć dla każdego użytkownika, który ma mieć zaszyfrowany swój katalog
`/home/` .

### Zastosowanie encfs

Następnie musimy nauczyć system co i jak ma szyfrować:

    $ encfs -v /home/.encfs/morfik/ /home/morfik/
    16:23:46 (main.cpp:523) Root directory: /home/.encfs/morfik/
    16:23:46 (main.cpp:524) Fuse arguments: (daemon) (threaded) (keyCheck) encfs /home/morfik/ -s -o use_ino -o default_permissions
    Creating new encrypted volume.
    Please choose from one of the following options:
     enter "x" for expert configuration mode,
     enter "p" for pre-configured paranoia mode,
     anything else, or an empty line will select standard mode.
    ?> x

    Manual configuration mode selected.
    The following cipher algorithms are available:
    1. AES : 16 byte block cipher
     -- Supports key lengths of 128 to 256 bits
     -- Supports block sizes of 64 to 4096 bytes
    2. Blowfish : 8 byte block cipher
     -- Supports key lengths of 128 to 256 bits
     -- Supports block sizes of 64 to 4096 bytes

    Enter the number corresponding to your choice: 1

    Selected algorithm "AES"

    Please select a key size in bits.  The cipher you have chosen
    supports sizes from 128 to 256 bits in increments of 64 bits.
    For example:
    128, 192, 256
    Selected key size: 256

    Using key size of 256 bits

    Select a block size in bytes.  The cipher you have chosen
    supports sizes from 64 to 4096 bytes in increments of 16.
    Or just hit enter for the default (1024 bytes)

    filesystem block size: 4096

    Using filesystem block size of 4096 bytes

    The following filename encoding algorithms are available:
    1. Block : Block encoding, hides file name size somewhat
    2. Null : No encryption of filenames
    3. Stream : Stream encoding, keeps filenames as short as possible

    Enter the number corresponding to your choice: 1

    Selected algorithm "Block""

    Enable filename initialization vector chaining?
    This makes filename encoding dependent on the complete path,
    rather then encoding each path element individually.
    The default here is Yes.
    Any response that does not begin with 'n' will mean Yes:

    Enable per-file initialization vectors?
    This adds about 8 bytes per file to the storage requirements.
    It should not affect performance except possibly with applications
    which rely on block-aligned file io for performance.
    The default here is Yes.
    Any response that does not begin with 'n' will mean Yes:

    Enable filename to IV header chaining?
    This makes file data encoding dependent on the complete file path.
    If a file is renamed, it will not decode sucessfully unless it
    was renamed by encfs with the proper key.
    If this option is enabled, then hard links will not be supported
    in the filesystem.
    The default here is No.
    Any response that does not begin with 'y' will mean No:

    Enable block authentication code headers
    on every block in a file?  This adds about 12 bytes per block
    to the storage requirements for a file, and significantly affects
    performance but it also means [almost] any modifications or errors
    within a block will be caught and will cause a read error.
    The default here is No.
    Any response that does not begin with 'y' will mean No:

    Add random bytes to each block header?
    This adds a performance penalty, but ensures that blocks
    have different authentication codes.  Note that you can
    have the same benefits by enabling per-file initialization
    vectors, which does not come with as great of performance
    penalty.
    Select a number of bytes, from 0 (no random bytes) to 8: 0

    Enable file-hole pass-through?
    This avoids writing encrypted blocks when file holes are created.
    The default here is Yes.
    Any response that does not begin with 'n' will mean Yes:

    16:28:41 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    16:28:41 (FileUtils.cpp:1123) Using cipher AES, key size 256, block size 4096

    Configuration finished.  The filesystem to be created has
    the following properties:
    16:28:41 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    16:28:41 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 24, ivlength 16
    Filesystem cipher: "ssl/aes", version 3:0:2
    16:28:41 (Interface.cpp:165) checking if nameio/block(3:0:1) implements nameio/block(3:0:1)
    Filename encoding: "nameio/block", version 3:0:1
    16:28:41 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    16:28:41 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    Key Size: 256 bits
    Block Size: 4096 bytes
    Each file contains 8 byte header with unique IV data.
    Filenames encoded using IV chaining mode.
    File holes passed through to ciphertext.

    Now you will need to enter a password for your filesystem.
    You will need to remember this password, as there is absolutely
    no recovery mechanism.  However, the password can be changed
    later using encfsctl.

    16:28:41 (openssl.cpp:48) Allocating 41 locks for OpenSSL
    16:28:41 (FileUtils.cpp:1180) useStdin: 0
    New Encfs Password:
    Verify Encfs Password:
    16:28:53 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    16:28:53 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    16:28:53 (Interface.cpp:165) checking if nameio/block(3:0:1) implements nameio/block(3:0:1)

Oczywiście można wybrać domyślne opcje. Jedyne co nas tutaj interesuje, to hasło, które musi pasować
do tego używanego do logowania się w systemie. W przeciwnym wypadku trzeba będzie podawać dwa hasła,
a nam zależy na tym by odblokowanie katalogu domowego nie rzucało się za bardzo w oczy.

Sprawdzamy czy system zamontował zaszyfrowany katalog:

    $ mount | grep encfs
    encfs on /home/morfik type fuse.encfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000,default_permissions,allow_root)

### Odtwarzanie backup'u

Przenosimy/kopiujemy teraz zawartość uprzednio zrobionego backup'u do odpowiednich katalogów. W moim
przypadku to wygląda tak:

    $ mv /home.original/morfik/* /home/morfik/
    $ mv /home.original/morfik/.* /home/morfik/

Pamiętajmy również o zmianie katalogu domowego w pliku `/etc/passwd` .

Po zakończeniu procesu kopiowania możemy zresetować maszynę. Jeśli chcemy sprawdzić czy faktycznie
system montuje katalog użytkownika, możemy zalogować się na TTY. Powinna zostać wyrzucona informacja
o montowaniu zasobu.

Jeśli z jakiegoś powodu chcielibyśmy odmontować katalog `/home/$USER` , robimy to w poniższy sposób:

    $ fusermount -u /home/morfik

### Zmiana hasła do katalogu domowego

Przydatną rzeczą też jest umiejętność zmiany hasła do zaszyfrowanego katalogu. Trzeba tylko pamiętać
by zmienić również hasło do konta:

    $ passwd

    $ encfsctl passwd /home/.encfs/morfik/

### Wrażliwy plik .encfs6.xml

I została jeszcze ostatnia kwestia tycząca się zaszyfrowanego katalogu z wykorzystaniem `encfs` ,
mianowicie trzeba zadbać o plik `.encfs6.xml` , który w tym przypadku znajduje się w katalogu
`/home/.encfs/morfik/` . Jeśli stracimy ten plik, nie uzyskamy już dostępu do danych. Dlatego trzeba
go skopiować i trzymać gdzieś w ukryciu.

## Przestrzeń wymiany SWAP

Samo zaszyfrowanie katalogu `/home/` , to nie wszystko. Wrażliwe dane mogą być złapane przez SWAP i
zapisane na dysku w formie niezaszyfrowanej. W przypadku gdy z jakiegoś powodu posiadamy SWAP, np.
korzystamy z hibernacji, trzeba go zaszyfrować. Nie będę tutaj opisywał całego procesu, bo zostało
to zrobione to [we wpisie dotyczącym szyfrowania
SWAP]({{< baseurl >}}/post/zaszyfrowana-przestrzen-wymiany-swap/).

## Katalog /tmp/

Ostatnia kwestia, która mi przychodzi na myśl, jeśli już mowa o zabezpieczeniach, to katalog `/tmp/`
. Jak nie patrzeć on zostaje odszyfrowany, a tam przecie trafiają pliki użytkownika. Można go
oczywiście zaszyfrować, a cały proces tworzenia zaszyfrowanego kontenera pod katalog `/tmp/` jest
dokładnie taki sam jak w przypadku przestrzeni wymiany. Jedynie co, to zamiast `mkswap`, trzeba użyć
`mkfs` z odpowiednim systemem plików.

Jak widać, w przypadku `encfs` trochę śmieszne się wydaje nieszyfrowanie pozostałej części systemu.
Zaszyfrowanie przestrzeni SWAP oraz katalogu `/tmp/` powinno zatrzymać wyciek wrażliwych informacji.
Nie dają jednak żadnej gwarancji nienaruszalności plików systemowych, o czym trzeba pamiętać.
