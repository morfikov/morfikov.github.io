---
author: Morfik
categories:
- Linux
date: "2015-10-19T23:11:22Z"
date_gmt: 2015-10-19 21:11:22 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- moduły-pam
- online-storage
GHissueID: 169
title: Implementacja encfs na dropbox'ie
---

Jako, że ostatnio [zaszyfrowaliśmy katalog domowy przy pomocy
encfs](/post/szyfrowanie-katalogu-home-przy-pomocy-encfs/) , to nie sposób sobie
nie zadać pytania czy tego typu mechanizm może działać w oparciu o serwisy online takie jak, np.
[dropbox](https://www.dropbox.com/). Chodzi o to, że dropbox umożliwia synchronizację plików w
czasie rzeczywistym, co może być problematyczne, gdy w grę wchodzi szyfrowanie danych. Wszelkie inne
rozwiązania na bazie [LUKS](https://pl.wikipedia.org/wiki/Linux_Unified_Key_Setup) są mało
praktyczne w tym przypadku. Natomiast synchronizowanie pojedynczych plików zaszyfrowanych przy
pomocy `encfs` zapowiada się obiecująco.

<!--more-->
## Szyfrowany katalog na potrzeby dropbox'a

Jako, że moduł PAM `libpam-encfs` jest w stanie automatycznie odszyfrować nam katalog domowy, gdy
tylko logujemy się do systemu, moglibyśmy stworzyć zewnętrzny katalog i przeznaczyć go pod zasoby
trzymane na dropbox'ie. Zatem do dzieła, tworzymy dwa katalogi i szyfrujemy jeden z nich:

    $ mkdir -p /media/Server/Dropbox.encfs/Dropbox/encrypted /media/Server/Dropbox

    $ encfs -v /media/Server/Dropbox.encfs/Dropbox/encrypted /media/Server/Dropbox
    14:04:58 (main.cpp:523) Root directory: /media/Server/Dropbox.encfs/Dropbox/encrypted/
    14:04:58 (main.cpp:524) Fuse arguments: (daemon) (threaded) (keyCheck) encfs /media/Server/Dropbox -s -o use_ino -o default_permissions
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

    14:05:21 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    14:05:21 (FileUtils.cpp:1123) Using cipher AES, key size 256, block size 4096

    Configuration finished.  The filesystem to be created has
    the following properties:
    14:05:21 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    14:05:21 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 24, ivlength 16
    Filesystem cipher: "ssl/aes", version 3:0:2
    14:05:21 (Interface.cpp:165) checking if nameio/block(3:0:1) implements nameio/block(3:0:1)
    Filename encoding: "nameio/block", version 3:0:1
    14:05:21 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    14:05:21 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    Key Size: 256 bits
    Block Size: 4096 bytes
    Each file contains 8 byte header with unique IV data.
    Filenames encoded using IV chaining mode.
    File holes passed through to ciphertext.

    Now you will need to enter a password for your filesystem.
    You will need to remember this password, as there is absolutely
    no recovery mechanism.  However, the password can be changed
    later using encfsctl.

    14:05:21 (openssl.cpp:48) Allocating 41 locks for OpenSSL
    14:05:21 (FileUtils.cpp:1180) useStdin: 0
    New Encfs Password:
    Verify Encfs Password:
    14:05:38 (Interface.cpp:165) checking if ssl/aes(3:0:2) implements ssl/aes(3:0:2)
    14:05:38 (SSL_Cipher.cpp:370) allocated cipher ssl/aes, keySize 32, ivlength 16
    14:05:39 (Interface.cpp:165) checking if nameio/block(3:0:1) implements nameio/block(3:0:1)

Hasło musi być takie samo jak to, którego używamy do logowania się w systemie, bo raczej nie chcemy
wprowadzać dwóch różnych haseł podczas fazy logowania.

### Problem z modułem PAM

Wychodzi na to, że w pliku `/etc/security/pam_encfs.conf` można sprecyzować tylko jedną regułkę per
user. Zatem jeśli już mamy tam wpisaną jedną, np. mając szyfrowany katalog domowy, to kolejnej nie
damy rady dopisać w tym pliku. Jeśli to zrobimy, to zostanie zwyczajnie zignorowana. Co zatem nam
pozostaje? Na szczęście możemy sprecyzować tyle punktów montowania ile chcemy, tylko trzeba to
zrobić w innym pliku, mianowicie w `/etc/security/pam_mount.conf.xml` . Edytujemy go zatem i
dodajemy w nim coś na wzór poniższych linijek:

    <!-- Volume definitions -->
    <volume user="morfik" fstype="fuse" path="encfs#/media/Server/Dropbox.encfs/Dropbox/encrypted" mountpoint="/media/Server/Dropbox" options="nonempty" />

## Testowanie konfiguracji

Uruchamiamy maszynę ponownie i sprawdzamy czy wszystkie katalogi obrabiane przez `encfs` zostały
zamontowane po zalogowaniu się do systemu:

    $ mount | grep encfs
    encfs on /home/morfik type fuse.encfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000,default_permissions,allow_root)
    encfs on /media/Server/Dropbox type fuse.encfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000,default_permissions)

Zatem wszystko działa jak należy. Wszystkie pliki, które zostaną umieszczone w katalogu
`/media/Server/Dropbox` zostaną zaszyfrowane, a ich zaszyfrowana postać będzie zlokalizowana w
`/media/Server/Dropbox.encfs/Dropbox/encrypted/` . Zatem jeśli w ustawieniach dropbox'a mamy
ustawiony katalog `/media/Server/Dropbox.encfs/Dropbox/` , co katalog `encrypted/` z powodzeniem
będzie synchronizowany w locie.
