---
author: Morfik
categories:
- Linux
date: "2015-06-22T18:58:18Z"
date_gmt: 2015-06-22 16:58:18 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- pliki
- foldery
- pam
GHissueID: 107
title: ECryptfs jako alternatywa dla encfs
---

Dnia 2014-10-07 w Debianie była aktualizacja pakietu `encfs` , która to zawierała informację na
temat [audytu bezpieczeństwa][1] jaki się dokonał parę miesięcy wstecz. Wyniki niezbyt dobrze
wypadły, a nawet można powiedzieć, że wręcz katastrofalnie. Generalnie cały test został skwitowany
słowami, że jeśli szyfrujemy pliki przy pomocy tego narzędzia, to jesteśmy relatywnie bezpieczni,
nawet w przypadku jeśli ktoś te pliki przechwyci. Natomiast jeśli zaczniemy zmieniać/dodawać pliki
i ten ktoś ponownie przechwyci nasz zaszyfrowany katalog, wtedy może on bez problemu odszyfrować
całą jego zawartość. Jeśli wykorzystujemy `encfs` lokalnie, to być może nam nic nie grozi, nawet w
przypadku raidu NSA na naszą chałupę. Problem w tym, że ogromna rzesza ludzi wykorzystuje `encfs`
do zaszyfrowania plików w chmurze, np. na Dropbox'ie czy MEGA, a jak wiadomo, przy każdej
synchronizacji, zmiany w danym katalogu są przesyłane do chmury i wszelkie zabezpieczenie jakie
daje `encfs` diabli biorą. Może najwyższy czas zainteresować się `ecryptfs`?

<!--more-->
## Czym jest eCryptfs

Ja używałem `encfs` do szyfrowania katalogów w serwisach takich jak Dropbox, Spideroak oraz na MEGA
i niestety zmuszony byłem usunąć całą ich zawartość. Zrobiłem to od razu, tuż po przeczytaniu
podlinkowanego we wstępie artykułu i to nawet bez zastanawiania się. Jedyne co, to została mi
odszyfrowana kopia lokalna katalogu. Niemniej jednak, nie uśmiecha mi się pakować tych plików
bezpośrednio do kontenera LUKS czy TrueCrypt i chciałbym by opcja szyfrowania pojedynczych plików
została. Szukając info na sieci na temat tego czym by tutaj zastąpić `encfs` , doszukałem się
[eCryptfs][2].

Cóż to takiego ten cały `eCryptfs` ? Zgodnie z tym co piszą na stronie projektu, jest to system
plików, który przechowuje metadane kryptograficzne w nagłówkach zaszyfrowanych plików. `eCryptfs`
jest częścią kernela i nie potrzeba instalować żadnych dodatkowych rzeczy by działał, w
przeciwieństwie do `encfs` . No chyba, że ktoś potrzebuje pewnych narzędzi zawartych w pakiecie
`ecryptfs-utils` ale można się też obejść i bez nich. Po zaszyfrowaniu plików, można je przesłać na
innego linux'a i tam dokonać deszyfracji przy pomocy klucza, który jest przechowywany w keyring'u
kernela. Do obsługi tego keyring'a potrzebne jest [narzędzie][3] [keyctl][4] zawarte w pakiecie
`keyutils` .

Generalnie `eCryptfs` zwykło używać się do zaszyfrowania katalogu domowego użytkownika i przy tej
operacji przydają się te [narzędzia][5] dostarczane z pakietem `ecryptfs-utils` , tylko są też
pewne ograniczenia wynikające z ich używania. Przede wszystkim, maksymalna ilość obsługiwanych
katalogów per user to jeden. Akurat na katalog `/home/$USER/` . Dodatkowo, nie mamy możliwości
określenia długości klucza. Jest wkodowany na sztywno i jest to `AES 128` -- trochę słabo. Tak czy
inaczej wszystko to, co można zrobić przy pomocy natywnych narzędzi w stosunku do katalogu domowego,
można też uczynić ręcznie w stosunku do każdego innego katalogu i tu mamy już spore pole manewru.

`eCryptfs` może być montowany przez plik `/etc/fstab` . W przypadku `encfs` nie szło w prosty sposób
tego zrobić, choć nie powinno się używać pliku `fstab` do obsługi montowania zasobów należących do
użytkownika. Niemniej jednak, jeśli jesteśmy jedynym użytkownikiem naszego PC i przy tym, posiadamy
zaszyfrowany system, możemy sobie dość znacząco ułatwić życie w przypadku korzystania z szyfrowanych
katalogów trzymanych w chmurze.

## Szyfrowanie katalogu /home/$USER/ przy pomocy eCryptfs

Szyfrowanie katalogu domowego z wykorzystaniem ubuntowych narzędzi sprowadza się do zalogowania się
na konto root'a i wydania jednego polecenia. Ja na potrzeby testów, stworzyłem dodatkowego
użytkownika i to jego katalog domowy zaszyfrowałem:

    # adduser morfikanin

    # ecryptfs-migrate-home -u morfikanin
    INFO:  Checking disk space, this may take a few moments.  Please be patient.
    INFO:  Checking for open files in /home/morfikanin
    Enter your login passphrase [morfikanin]:

    ************************************************************************
    YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
      ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
    THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
    ************************************************************************


    Done configuring.

    chown: cannot access '/dev/shm/.ecryptfs-morfikanin': No such file or directory
    INFO:  Encrypted home has been set up, encrypting files now...this may take a while.
    sending incremental file list
    ./
    .bash_logout
                220 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=2/4)
    .bashrc
              3,515 100%    3.35MB/s    0:00:00 (xfr#2, to-chk=1/4)
    .profile
                675 100%  329.59kB/s    0:00:00 (xfr#3, to-chk=0/4)

    ========================================================================
    Some Important Notes!

     1. The file encryption appears to have completed successfully, however,
        morfikanin MUST LOGIN IMMEDIATELY, _BEFORE_THE_NEXT_REBOOT_,
        TO COMPLETE THE MIGRATION!!!

     2. If morfikanin can log in and read and write their files, then the migration is complete,
        and you should remove /home/morfikanin.vvDPp6xm.
        Otherwise, restore /home/morfikanin.vvDPp6xm back to /home/morfikanin.

     3. morfikanin should also run 'ecryptfs-unwrap-passphrase' and record
        their randomly generated mount passphrase as soon as possible.

     4. To ensure the integrity of all encrypted data on this system, you
        should also encrypt swap space with 'ecryptfs-setup-swap'.
    ========================================================================

Jeśli dokonujemy szyfrowania katalogu domowego użytkownika, jego pliki zostaną pierw przeniesione do
`/home/$USER.vvDPp6xm/` , a następnie skopiowane do podmontowanego folderu i tym samym zostaną
zaszyfrowane. Także jeśli mamy sporo plików w katalogu `/home/$USER/` , to ten proces może trochę
zająć. W tym przypadku były trzy małe pliki tekstowe.

Zgodnie z powyższymi instrukcjami, logujemy się teraz na konto użytkownika, którego katalog domowy
zaszyfrowaliśmy i sprawdzamy czy możemy odczytywać/zapisywać jego pliki:

    $ ls -al /home/
    total 40
    drwxr-xr-x  7 root       root        4096 Oct 11 06:34 .
    drwxr-xr-x 24 root       root        4096 Oct  9 10:37 ..
    drwxr-xr-x  3 root       root        4096 Oct 11 06:34 .ecryptfs
    drwx------  2 root       root       16384 Jun 27  2013 lost+found
    drwxr-xr-x 80 morfik     morfik      4096 Oct 11 00:11 morfik
    drwx------  3 morfikanin morfikanin  4096 Oct 11 06:50 morfikanin
    drwx------  2 morfikanin morfikanin  4096 Oct 11 06:34 morfikanin.vvDPp6xm

    $ ls -al /home/morfikanin/
    total 80
    drwx------ 3 morfikanin morfikanin 4096 Oct 11 06:50 .
    drwxr-xr-x 7 root       root       4096 Oct 11 06:34 ..
    lrwxrwxrwx 1 morfikanin morfikanin   35 Oct 11 06:34 .Private -> /home/.ecryptfs/morfikanin/.Private
    -rw-r--r-- 1 morfikanin morfikanin  220 Oct 11 06:34 .bash_logout
    -rw-r--r-- 1 morfikanin morfikanin 3515 Oct 11 06:34 .bashrc
    lrwxrwxrwx 1 morfikanin morfikanin   36 Oct 11 06:34 .ecryptfs -> /home/.ecryptfs/morfikanin/.ecryptfs
    -rw-r--r-- 1 morfikanin morfikanin  675 Oct 11 06:34 .profile

    $ touch test

    $ echo test > test

    $ cat test
    test

Wygląda w porządku. W katalogu `.Private` są trzymane zaszyfrowane pliki. Katalog `/home/$USER/` , w
którym się znajdujemy, jest punktem montowania.

Musimy także odczytać zaszyfrowane hasło przy pomocy `ecryptfs-unwrap-passphrase` i gdzieś je sobie
zapisać. Jeśli to hasło przepadnie, nie uda nam się już odzyskać plików z tego katalogu.

    $ ecryptfs-unwrap-passphrase
    Passphrase:
    4ce20ef0799aba39d28d511a452a0f90

Po tym jak się wylogujemy z konta, podgląd katalogu domowego będzie się prezentował następująco:

    # ls -al /home/morfikanin
    total 8.0K
    dr-x------ 2 morfikanin morfikanin 4.0K Oct 11 06:34 ./
    drwxr-xr-x 7 root       root       4.0K Oct 11 06:34 ../
    lrwxrwxrwx 1 morfikanin morfikanin   35 Oct 11 06:34 .Private -> /home/.ecryptfs/morfikanin/.Private/
    lrwxrwxrwx 1 morfikanin morfikanin   36 Oct 11 06:34 .ecryptfs -> /home/.ecryptfs/morfikanin/.ecryptfs/
    lrwxrwxrwx 1 morfikanin morfikanin   56 Oct 11 06:34 Access-Your-Private-Data.desktop -> /usr/share/ecryptfs-utils/ecryptfs-mount-private.desktop
    lrwxrwxrwx 1 morfikanin morfikanin   52 Oct 11 06:34 README.txt -> /usr/share/ecryptfs-utils/ecryptfs-mount-private.txt

Krótko pisząc, katalog domowy został odmontowany po wylogowaniu się użytkownika i nikt nie ma
dostępu do jego plików, przynajmniej w formie odszyfrowanej.

Automontowanie zasobu odbywa się za pośrednictwem modułu PAM. Jeśli nie wiemy czy mamy aktywowany
ten moduł, możemy to sprawdzić wpisując w terminalu `pam-auth-update` jako root:

![](/img/2015/06/1.ecryptfs-pam-konfiguracja.png#huge)

Po udanej akcji zaszyfrowania katalogu domowego, kasujemy wcześniej utworzony automatycznie katalog
z backup'em `/home/$USER.vvDPp6xm/` :

    # rm -R /home/morfikanin.vvDPp6xm/

## Katalog .ecryptfs

Każdy katalog `/home/$USER/` ma podkatalog `.ecryptfs`, w którym to znajdują się informacje
niezbędne do operowania na zaszyfrowanym katalogu domowym użytkownika. Mamy tam poniższe pliki:

    # ls -al /home/.ecryptfs/morfikanin/.ecryptfs/
    total 20K
    drwx------ 2 morfikanin morfikanin 4.0K Oct 11 06:34 ./
    drwxr-xr-x 4 morfikanin morfikanin 4.0K Oct 11 06:34 ../
    -rw------- 1 morfikanin morfikanin   17 Oct 11 06:34 Private.mnt
    -rw------- 1 morfikanin morfikanin   34 Oct 11 06:34 Private.sig
    -rw-r--r-- 1 morfikanin morfikanin    0 Oct 11 06:34 auto-mount
    -rw-r--r-- 1 morfikanin morfikanin    0 Oct 11 06:34 auto-umount
    -r-------- 1 morfikanin morfikanin   48 Oct 11 06:34 wrapped-passphrase

Pliki `auto-mount` oraz `auto-umount` to puste pliki i moduł PAM sprawdza ich obecność i jeśli
istnieją, to automatycznie montuje/demontuje zasób, gdy użytkownik się loguje/wylogowuje do/z
systemu. Sam zaszyfrowany katalog jest montowany w miejscu określonym w pliku `Private.mnt` . Z
kolei w pliku `Private.sig` są przechowywane sygnatury haseł, którymi to są szyfrowane pliki i ich
nazwy. W pliku `wrapped-passphrase` znajduje się losowo wygenerowane i zaszyfrowane hasło służące do
automatycznego montowania katalogu domowego wraz z logowaniem się użytkownika do systemu. By
odszyfrować to hasło, trzeba podać inne hasło, w tym przypadku jest to hasło do konta użytkownika w
systemie -- to tego hasła używa moduł PAM.

Domyślnie, zarówno pliki w katalogu jak i ich nazwy są szyfrowane niezależnie dwoma różnymi
kluczami. Jeśli podejrzymy keyring w krenelu dla tego użytkownika, ujrzymy tam dwa wpisy:

    $ keyctl list @u
    2 keys in keyring:
    167322969: --alswrv  1001  1001 user: d2a0514b5e5288d3
    645190684: --alswrv  1001  1001 user: 5edd65cdfbbb0a0f

Hash po prawej stronie to sygnatura hasła, na podstawie której kernel identyfikuje konkretne klucze
w keyring'u. Są one także zapisane w pliku `Private.sig` :

    # cat /home/.ecryptfs/morfikanin/.ecryptfs/Private.sig
    5edd65cdfbbb0a0f
    d2a0514b5e5288d3

## Mechanizm działania eCryptfs

To jak ten mechanizm działa dokładnie, zostało wyjaśnione na [blogu jednego z
twórców][6] eCryptfs. W skrócie, gdy logujemy się do systemu, moduł `pam_ecryptfs` przy pomocy
hasła do naszego konta deszyfruje symetrycznie plik `wrapped-passphrase` zlokalizowany w katalogu
`~/.ecryptfs/` . W tym pliku domyślnie znajduje się 128-bitowy losowo wygenerowany klucz. Po
odszyfrowaniu, jest on solony i hash'owany 65536 razy przy wykorzystaniu hash'a `sha512`. Następnie
taki klucz jest ładowany do keyring'a kernela i to przy jego pomocy są szyfrowane nagłówki plików,
w których to znajdują się unikatowe klucze szyfrujące same pliki. Dzięki takiemu rozwiązaniu, mając
dwa pliki o takiej samej odszyfrowanej zawartości, po zaszyfrowaniu będą miały kompletnie inną
formę. Wszystkie nowsze wersje kernela szyfrują także nazwy plików i robią to osobnym kluczem. W
tym przypadku, hasło określone w pliku `wrapped-passphrase` jest wczytywane drugi raz i solone inną
wartością niż miało to miejsce wcześniej, po czym jest ono również hash'owane 65536 razy przy
użyciu `sha512` , a wynikowy klucz jest dodawany do keyring'a kernela. Dlatego też, na innej
maszynie, przy pomocy znanego tylko nam hasła, możemy odszyfrować konkretny plik.

Powyżej, w keyring'u kernela, mamy dwie sygnatury. Sprawdźmy zatem czy faktycznie pochodzą one od
zaszyfrowanego hasła służącego do montowania katalogu domowego. By je porównać, trzeba wydobyć
zaszyfrowane hasło z pliku i dodać je do keyring'a:

    $ ecryptfs-unwrap-passphrase
    Passphrase:
    4ce20ef0799aba39d28d511a452a0f90

    $ ecryptfs-add-passphrase --fnek
    Passphrase:
    Inserted auth tok with sig [5edd65cdfbbb0a0f] into the user session keyring
    Inserted auth tok with sig [d2a0514b5e5288d3] into the user session keyring

Jak widać powyżej, obie sygnatury się zgadzają.

## Odzyskiwanie/przenoszenie zaszyfrowanego katalogu

Jeśli znaleźlibyśmy się w sytuacji krytycznej, tj. stracilibyśmy pliki w katalogu `.ecryptfs`, lub
zwyczajnie chcielibyśmy odtworzyć zaszyfrowany katalog na innej maszynie ale mamy do dyspozycji
tylko zaszyfrowane pliki oraz odszyfrowane hasło, możemy przeprowadzić poniższe kroki by uzyskać
dostęp do zaszyfrowanego katalogu. Na potrzeby testu wylogowałem się z konta użytkownika, którego
katalog domowy poddałem szyfrowaniu. Następnie nadpisałem kompletnie pliki w katalogu
`/home/morfikanin/.ecryptfs/`:

    # cd /home/
    # > ./morfikanin/.ecryptfs/wrapped-passphrase
    # > ./morfikanin/.ecryptfs/Private.mnt
    # > ./morfikanin/.ecryptfs/Private.sig

    # ls -al ./morfikanin/.ecryptfs/
    total 8.0K
    drwx------ 2 morfikanin morfikanin 4.0K Oct 11 08:12 ./
    drwxr-xr-x 4 morfikanin morfikanin 4.0K Oct 11 06:34 ../
    -rw------- 1 morfikanin morfikanin    0 Oct 11 08:17 Private.mnt
    -rw------- 1 morfikanin morfikanin    0 Oct 11 08:17 Private.sig
    -rw-r--r-- 1 morfikanin morfikanin    0 Oct 11 06:34 auto-mount
    -rw-r--r-- 1 morfikanin morfikanin    0 Oct 11 06:34 auto-umount
    -rw-r--r-- 1 morfikanin morfikanin    0 Oct 11 08:17 wrapped-passphrase

W tej chwili, bez posiadania odszyfrowanej formy hasła nie jesteśmy w stanie odzyskać zaszyfrowanych
plików. Musimy zatem zaszyfrować hasło potrzebne do zamontowania katalogu `/home/$USER/` przy pomocy
`ecryptfs-wrap-passphrase` podając w `Passphrase to wrap` odszyfrowaną formę hasła, a w `Wrapping
passphrase` hasło do logowania do systemu:

    # ecryptfs-wrap-passphrase ./morfikanin/.ecryptfs/wrapped-passphrase
    Passphrase to wrap:
    Wrapping passphrase:

Sprawdzamy czy jesteśmy w stanie odszyfrować hasło, które znalazło się w pliku:

    # ecryptfs-unwrap-passphrase ./morfikanin/.ecryptfs/wrapped-passphrase
    Passphrase:
    4ce20ef0799aba39d28d511a452a0f90

Zatem ten krok mamy z głowy. Definiujemy także gdzie zamontować zaszyfrowany zasób:

    # echo "/home/morfikanin/" > ./morfikanin/.ecryptfs/Private.mnt

Potrzebne nam są jeszcze sygnatury kluczy szyfrujących. W `Passphrase:` podajemy odszyfrowane hasło
do montowania:

    # ecryptfs-add-passphrase --fnek
    Passphrase:
    Inserted auth tok with sig [5edd65cdfbbb0a0f] into the user session keyring
    Inserted auth tok with sig [d2a0514b5e5288d3] into the user session keyring

Uzupełniamy sygnatury:

    # echo 5edd65cdfbbb0a0f > ./morfikanin/.ecryptfs/Private.sig
    # echo d2a0514b5e5288d3 >> ./morfikanin/.ecryptfs/Private.sig

Teraz już tylko zostało nam przetestować czy po zalogowaniu się, katalog `/home/$USER/` zostanie
automatycznie odszyfrowany i zamontowany. W moim przypadku, wszystko przebiegło bez większych
problemów.

## Szyfrowanie folderu ~/Private/

Jeśli chcemy zaszyfrować [inny folder niż domowy][7], możemy to zrobić przy pomocy narzędzia
`ecryptfs-setup-private` . Działać to będzie na tej samej zasadzie co szyfrowanie całego folderu
domowego, z tym, że zostanie ograniczone tylko do folderu `~/Private/` . Korzystając z
`ecryptfs-setup-private` nie mamy możliwości ani dostosowania długości klucza, który wynosi 128
bitów, ani też określenia innego katalogu niż tego domyślny. Poniżej jest przedstawione
wykorzystanie ww. narzędzia:

    morfik:~$ ecryptfs-setup-private
    Enter your login passphrase [morfik]:
    Enter your mount passphrase [leave blank to generate one]:

    ************************************************************************
    YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
      ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
    THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
    ************************************************************************


    Done configuring.

    Testing mount/write/umount/read...
    Inserted auth tok with sig [b91ce3811ab95844] into the user session keyring
    Inserted auth tok with sig [c1884ffbc9e246b7] into the user session keyring
    Inserted auth tok with sig [b91ce3811ab95844] into the user session keyring
    Inserted auth tok with sig [c1884ffbc9e246b7] into the user session keyring
    Testing succeeded.

    Logout, and log back in to begin using your encrypted directory.

I to w zasadzie tyle. By móc korzystać z tego folderu, musimy się ponownie zalogować w systemie.
Dobrze jest też sprawdzić po zalogowaniu czy folder jest poprawnie montowany przez wydanie polecenia
`mount` :

    $ mount
    ...
    /home/morfik/.Private on /home/morfik/Private type ecryptfs (rw,nosuid,nodev,relatime,ecryptfs_fnek_sig=c1884ffbc9e246b7,ecryptfs_sig=b91ce3811ab95844,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_unlink_sigs)

Po wylogowaniu się, folder powinien zostać automatycznie zdemontowany.

Istnieje też, co prawda, opcja sprecyzowania, który folder chcemy zaszyfrować i gdzie go zamontować
ale w takim przypadku, PAM nie jest w stanie montować/demontować takiego katalogu, dlatego też
darujemy sobie tę konfigurację.

## Szyfrowanie dowolnego folderu

Ja zbytnio nie potrzebuję ani szyfrować katalogu domowego, ani katalogu `~/Private/` , bo mam
zaimplementowane pełne szyfrowanie dysku na bazie LUKS. Poza tym, problem z powyższym rozwiązaniem
jest oczywisty jeśli chodzi o przechowywanie tak zaszyfrowanych plików w różnego rodzaju chmurach.
Mianowicie, jak udostępnić taki folder, np. na Dropbox? Dropbox wymaga od nas by udostępniane
pliki umieszczać w określonym przez niego katalogu. Z kolei z MEGA jest trochę lepiej, bo możemy
precyzować foldery niezależnie i możemy wybrać zaszyfrowany folder `~/.Private/` i go przesłać do
chmury. A co w przypadku, gdy nam nie odpowiada położenie katalogu `~/.Private/` albo też co w
przypadku gdybyśmy chcieli mieć wiele zaszyfrowanych folderów? Tego typu konfiguracji nie sprostają
ubuntowe narzędzia. Jest to, co prawda, wykonalne ale trzeba ręcznie stworzyć cały setup.

Na sam początek potrzebujemy kilka katalogów -- po dwa na każdy zaszyfrowany zasób. Dodatkowo,
nadajemy im odpowiednie prawa dostępu:

    # cd /media/Server/test/
    # mkdir .1 1 .2 2 .3 3
    # chmod 500 1 2 3
    # chmod 700 .1 .2 .3

    # ls -al
    total 32K
    drwxr-xr-x  8 morfik morfik 4.0K Oct 11 20:07 ./
    drwxr-xr-x 18 morfik morfik 4.0K Oct 11 19:35 ../
    drwx------  2 morfik morfik 4.0K Oct 11 19:36 .1/
    drwx------  2 morfik morfik 4.0K Oct 11 19:36 .2/
    drwx------  2 morfik morfik 4.0K Oct 11 20:07 .3/
    dr-x------  2 morfik morfik 4.0K Oct 11 20:07 1/
    dr-x------  2 morfik morfik 4.0K Oct 11 20:07 2/
    dr-x------  2 morfik morfik 4.0K Oct 11 20:07 3/

To jaką konfigurację sobie obierzemy, zależy tylko od nas. Możemy mieć wszystkie katalogi na jedno
hasło, lub każdy na inne. Niemniej jednak, w przypadku posiadania wielu katalogów, hasła do nich
będziemy musieli przechowywać w postaci czystego tekstu gdzieś na dysku, dlatego, też jeśli mamy
niezaszyfrowany system, to hasło może zostać przechwycone, z czym należy się jak najbardziej liczyć.

Na necie doszukałem się informacji na temat pliku `~/.ecryptfsrc` i próbowałem przy jego pomocy
zaszyfrować katalogi. Niby się nawet udało to zrobić ale tylko z konta użytkownika root. Nie szło za
to przenieść tej konfiguracji na zwykłego user'a. Sam plik ma poniższą postać:

    key=passphrase
    passphrase_passwd=4490af7434bebacd
    #passphrase_passwd_file=/mnt/usb/passwd_file.txt
    #ecryptfs_unlink_sigs
    ecryptfs_xattr
    ecryptfs_key_bytes=32
    ecryptfs_cipher=aes
    ecryptfs_passthrough=n
    ecryptfs_sig=eb8898761ad51521
    ecryptfs_fnek_sig=eb8898761ad51521

Dodatkowo są jakieś problemy z sygnaturami i nie idzie ustawić w tym pliku dwóch różnych, co
powoduje, że nagłówki plików i nazwy plików są szyfrowane tym samym kluczem. Poza tym, natknąłem się
na szereg błędów przy montowaniu, także raczej odradzam korzystanie z tego pliku.

Na szczęście istnieje inne rozwiązanie, przy pomocy którego udało mi się osiągnąć pożądaną przeze
mnie konfigurację. Zakłada ona wykorzystanie autostartu środowiska graficznego. Schemat działania
jest prosty. Wczytujemy hasło i na jego podstawie są generowane dwie sygnatury, które są umieszczane
w keyring'u kernela. Wszystkie opcje szyfrowania umieszczamy w pliku `/etc/fstab` . Dodatkowo, w
opcjach dodajemy `users` tak, by montowanie tych zasobów mogło odbywać się także z konta zwykłego
użytkownika ale tylko tego będącego członkiem grupy `users` .

Na początek zajmijmy się hasłem. By wygenerować jakiś rozsądny losowy ciąg znaków (32 bajty),
wpisujemy w terminal poniższe polecenie:

    $ od -x -N 32 --width=32 /dev/urandom | head -n 1 | sed "s/^0000000//" | sed "s/\s*//g"
    dde1109bb38ede1b9d7309929a956365b132d454d8ea77636e0b02a792e9e82b

Teraz tym hasłem musimy nakarmić `ecryptfs-add-passphrase` :

    $ ecryptfs-add-passphrase --fnek
    Passphrase:
    Inserted auth tok with sig [49d33e33ef5a3ecb] into the user session keyring
    Inserted auth tok with sig [80d9a32888b65b44] into the user session keyring

Uzyskane w ten sposób sygnatury wpisujemy do pliku `/etc/fstab` wraz z dodatkową
    konfiguracją:

    /media/Server/test/.1 /media/Server/test/1 ecryptfs    defaults,user,noauto,nofail,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_cipher=aes,ecryptfs_key_bytes=32 0 0
    /media/Server/test/.2 /media/Server/test/2 ecryptfs   defaults,user,noauto,nofail,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_cipher=aes,ecryptfs_key_bytes=32 0 0
    /media/Server/test/.3 /media/Server/test/3 ecryptfs  defaults,user,noauto,nofail,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_cipher=aes,ecryptfs_key_bytes=32 0 0

By przetestować montowanie zasobów, wydajemy poniższe polecenia:

    $ keyctl list @u
    2 keys in keyring:
    769826815: --alswrv  1000  1000 user: 49d33e33ef5a3ecb
    185921157: --alswrv  1000  1000 user: 80d9a32888b65b44

    morfik:~$ mount -i /media/Server/test/1
    morfik:~$ mount -i /media/Server/test/2
    morfik:~$ mount -i /media/Server/test/3

    $ mount
    ...
    /media/Server/test/.1 on /media/Server/test/1 type ecryptfs (rw,nosuid,nodev,noexec,relatime,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_cipher=aes,ecryptfs_key_bytes=32,_netdev,user=morfik)
    /media/Server/test/.2 on /media/Server/test/2 type ecryptfs (rw,nosuid,nodev,noexec,relatime,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_cipher=aes,ecryptfs_key_bytes=32,_netdev,user=morfik)
    /media/Server/test/.3 on /media/Server/test/3 type ecryptfs (rw,nosuid,nodev,noexec,relatime,ecryptfs_fnek_sig=80d9a32888b65b44,ecryptfs_sig=49d33e33ef5a3ecb,ecryptfs_cipher=aes,ecryptfs_key_bytes=32,_netdev,user=morfik)

By zautomatyzować proces montowania zasobów na starcie systemu, do autostartu Openbox'a (lub
innego środowiska graficznego) dodajemy poniższą zwrotkę:

    if [ -z "$(mount | grep ecryptfs)" ] ; then
          printf "%s" "dde1109bb38ede1b9d7309929a956365b132d454d8ea77636e0b02a792e9e82b" | ecryptfs-add-passphrase --fnek -
          mount -i /media/Server/test/1
          mount -i /media/Server/test/2
          mount -i /media/Server/test/3
    fi

Za każdym razem gdy będziemy wywoływać graficzną sesję logowania, system sprawdzi czy jakieś
zaszyfrowane katalogi są zamontowane i jeśli nie są, załaduje sygnatury do keyring'a kernela, po
czym podmontuje zasoby.

Powyższą zwrotkę możemy także dodać do pliku `~/.profile` lub `~/.bashrc` jeśli interesuje nas
dostęp do zaszyfrowanych katalogów wyłącznie spod TTY.

## Odszyfrowywanie pojedynczych plików

Jeśli bylibyśmy zmuszeni z jakiegoś powodu pobrać surową kopię pliku z Dropboxa, możemy tak uzyskany
plik bez większego problemu odszyfrować. Wystarczy stworzyć dwa katalogi i w jednym z nich umieścić
zaszyfrowany plik. Następnie znając hasło, możemy określić
    sygnatury:

    # printf "%s" "dde1109bb38ede1b9d7309929a956365b132d454d8ea77636e0b02a792e9e82b" | ecryptfs-add-passphrase --fnek -
    Inserted auth tok with sig [49d33e33ef5a3ecb] into the user session keyring
    Inserted auth tok with sig [80d9a32888b65b44] into the user session keyring

Mając sygnatury, możemy przejść do katalogu, w którym stworzyliśmy dwa testowe podkatalogi i z konta
root dokonać zamontowania zasobu:

    # mount -t ecryptfs .1 1
    Select key type to use for newly created files:
     1) tspi
     2) passphrase
    Selection: 2
    Passphrase:
    Select cipher:
     1) aes: blocksize = 16; min keysize = 16; max keysize = 32
     2) blowfish: blocksize = 8; min keysize = 16; max keysize = 56
     3) des3_ede: blocksize = 8; min keysize = 24; max keysize = 24
     4) twofish: blocksize = 16; min keysize = 16; max keysize = 32
     5) cast6: blocksize = 16; min keysize = 16; max keysize = 32
     6) cast5: blocksize = 8; min keysize = 5; max keysize = 16
    Selection [aes]: 1
    Select key bytes:
     1) 16
     2) 32
     3) 24
    Selection [16]: 2
    Enable plaintext passthrough (y/n) [n]:
    Enable filename encryption (y/n) [n]: y
    Filename Encryption Key (FNEK) Signature [49d33e33ef5a3ecb]: 80d9a32888b65b44
    Attempting to mount with the following options:
      ecryptfs_unlink_sigs
      ecryptfs_fnek_sig=80d9a32888b65b44
      ecryptfs_key_bytes=32
      ecryptfs_cipher=aes
      ecryptfs_sig=49d33e33ef5a3ecb
    WARNING: Based on the contents of [/root/.ecryptfs/sig-cache.txt],
    it looks like you have never mounted with this key
    before. This could mean that you have typed your
    passphrase wrong.

    Would you like to proceed with the mount (yes/no)? : yes
    Would you like to append sig [49d33e33ef5a3ecb] to
    [/root/.ecryptfs/sig-cache.txt]
    in order to avoid this warning in the future (yes/no)? : no
    Not adding sig to user sig cache file; continuing with mount.
    Mounted eCryptfs

Sprawdzamy, czy plik został pomyślnie odszyfrowany:

    # ls -al .1/
    total 19M
    drwxr-xr-x 2 morfik morfik 4.0K Oct 12 02:03 ./
    drwxr-xr-x 4 morfik morfik 4.0K Oct 12 02:03 ../
    -rw-r--r-- 1 morfik morfik  19M Oct 11 00:20 ECRYPTFS_FNEK_ENCRYPTED.FXbOcDQ6N4OQfkZidLU.rJ-fktfdjINirZfMNdk7XKECoP8wqi2eqiHmZJmgAms1xrIVSmwYtw5.l2s-

    # ls -al 1/
    total 19M
    drwxr-xr-x 2 morfik morfik 4.0K Oct 12 02:03 ./
    drwxr-xr-x 4 morfik morfik 4.0K Oct 12 02:03 ../
    -rw-r--r-- 1 morfik morfik  19M Oct 11 00:20 2014_10_11_00_13_29.tar.gz

Trzeba tylko pamiętać, by po skończonej robocie, zdemontować zaszyfrowany katalog.

## Dodatkowe zabezpieczenia

Jako, że opcje szyfrowania mamy zdefiniowane w pliku `/etc/fstab` oraz, że hasło w postaci zwykłego
tekstu widnieje w pliku autostartu środowiska graficznego, trzeba odpowiednio zabezpieczyć te pliki.
W przypadku `/etc/fstab` , ustawiamy prawa dostępu na `600`, podobnie postępujemy z plikiem
openboxa:

    # chmod 600 ~/.config/openbox/autostart
    # chmod 600 /etc/fstab

Jeśli nie korzystamy z pełnego szyfrowania dysku, a jedynie szyfrujemy katalog domowy, musimy także
pomyśleć o zaszyfrowaniu przestrzeni wymiany, jeśli takową posiadamy w systemie, oraz o
zabezpieczeniu katalogu `/tmp/` . Ubuntowe narzędzia od `eCryptfs` dostarczają oprogramowanie, które
potrafi zaszyfrować SWAP ale z tego co czytałem, to działa ono jedynie pod ubuntu.

Doczytałem się także informacji, że `eCryptfs` może obsługiwać certyfikaty zamiast haseł, co potrafi
dodatkowo zwiększyć bezpieczeństwo zaszyfrowanych danych. Tylko, że ten ficzer zwyczajnie nie
działa:

    # ecryptfs-manager

    eCryptfs key management menu
    -------------------------------
            1. Add passphrase key to keyring
            2. Add public key to keyring
            3. Generate new public/private keypair
            4. Exit

    Make selection: 3
    Select key type to use for newly created files:
    Selection:
    Select key type to use for newly created files:
    Selection: 1
    Select key type to use for newly created files:
    Selection: 2
    Select key type to use for newly created files:
    Selection: 4
    Select key type to use for newly created files:
    Selection: ^C

Jak widać powyżej, nie ma żadnego typu klucza. Dodatkowo, w syslog'u można zaobserwować poniższy
błąd:

    ecryptfs-manager: Error initializing key module [/usr/lib/x86_64-linux-gnu/ecryptfs/libecryptfs_key_mod_gpg.so]; rc = [-22]
    ecryptfs-manager: Key module [tspi] does not have a key generation subgraph transition node
    ecryptfs-manager: Key module [passphrase] does not have a key generation subgraph transition node

Pozostaje mieć nadzieję, że wszystkie błędy, na które natrafiłem przy przejściu z `encfs` na
`eCryptfs` zostaną wyeliminowane.

## Prace nad encfs

Doszukałem się informacji, że prace nad `encfs` ruszyły oraz, że kod projektu został przeniesiony na
[github'a][8] i jak pisze twórca, ten projekt teraz zależy głównie od społeczności. Przez trzy i
pół roku aplikacja przebywała w stanie uśpienia ale 10 marca 2015 roku doczekała się wersji 1.8.


[1]: https://defuse.ca/audits/encfs.htm
[2]: https://ecryptfs.org/
[3]: https://www.ibm.com/developerworks/library/l-key-retention/
[4]: https://lwn.net/Articles/210502/
[5]: https://ecryptfs.org/documentation.html
[6]: https://blog.dustinkirkland.com/2009/02/how-encrypted-home-ecryptfs-works.html
[7]: https://help.ubuntu.com/community/EncryptedPrivateDirectory
[8]: https://github.com/vgough/encfs
