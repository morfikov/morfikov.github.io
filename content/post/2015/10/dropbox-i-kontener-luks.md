---
author: Morfik
categories:
- Linux
date: "2015-10-21T20:34:42Z"
date_gmt: 2015-10-21 18:34:42 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- online-storage
- luks
title: Dropbox i kontener LUKS
---

Ogarnęliśmy już szyfrowanie plików na Dropbox przy pomocy [encfs][1] oraz [kontenerów
TrueCrypt][2]. Każda z w/w operacji drastycznie poprawiła prywatność naszych plików, które
przechowujemy w chmurze. Poniższy wpis będzie w podobnym klimacie, tj. spróbujemy umieścić na
Dropbox [kontener LUKS][3], co niesie ze sobą sporo udogodnień i czyni korzystanie z
zaszyfrowanego Dropbox'a praktycznie transparentnym.

<!--more-->
## Tworzenie kontenera pod dane na Dropbox

Przede wszystkim, musimy utworzyć plik składający się z samych zer. Tak stworzony plik umieszczamy
na Dropbox. Sam plik może być dowolnego rozmiaru i nie ma tutaj znaczenia czy będzie to 100 MiB
czy 10GiB. Liczy się tylko to ile miejsca chcemy przeznaczyć na Dropbox na szyfrowane pliki.
Najciekawsze w tym wszystkim jest to, że plik składający się z samych zer zostanie przed wysłaniem
do chmury skompresowany, a kompresja samych zer daje wynik porównywalny z zerem. I tak, np. 10 GiB
plik będzie zajmować parę KiB, co oczywiście cieszy, bo nie będzie trzeba wysyłać do chmury całych
10GiB danych.

Na sam początek zatrzymujemy synchronizację plików w Dropbox i przechodzimy do katalogu gdzie
mamy swoją lokalną kopię plików. W nim tworzymy plik kontenera LUKS:

    $ cd /media/Server/Dropbox/
    $ dropbox status
    Syncing paused

    $ dd if=/dev/zero of=./luks_dropbox bs=2M count=500

Puszczamy synchronizację i po chwili kontener powinien zostać przesłany do chmury. Stan
synchronizacji możemy sprawdzić w poniższy sposób:

    $ dropbox filestatus

    luks_dropbox:        up to date

Ponownie wstrzymujemy synchronizację plików, po czym logujemy się na konto root w celu przygotowania
urządzenia `loop`, które zamontuje nam w systemie plik kontenera:

    # losetup /dev/loop0 /media/Server/Dropbox/luks_dropbox
    # losetup -a
    /dev/loop0: [0807]:2883594 (/media/Server/Dropbox/luks_dropbox)

Teraz trzeba zaszyfrować plik kontenera przy pomocy `cryptsetup` . Pamiętajmy by wskazać urządzenie
`/dev/loop0` , a nie faktyczny plik na dysku:

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase --verbose luksFormat /dev/loop0

Otwieramy kontener i tworzymy w nim system plików:

    # cryptsetup luksOpen /dev/loop0 crypt_dropbox

    # mkfs.ext4 -m 0 -L dropbox /dev/mapper/crypt_dropbox

W tej chwili mamy przygotowany kontener na dane. Zamykamy go i synchronizujemy Dropbox'a:

    # cryptsetup luksClose crypt_dropbox

    $ dropbox status
    Updating (1 file, 9 mins left)
    Uploading "luks_dropbox" (70.0 kB/sec, 9 mins left)

## Konfiguracja kontenera LUKS

Powyższy setup powinien działać prawidłowo. Musimy jednak jeszcze nauczyć system jak automatycznie
montować ten kontener. Ja już mam w swoim systemie kilka zaszyfrowanych dysków, dlatego też u mnie
nie będzie potrzebne dodatkowe hasło, które trzeba by podawać przy starcie systemu. Hasła co prawda
są inne, i w przypadku systemu, i w przypadku kontenera dropbox ale przy pomocy ciekawej sztuczki
można uzależnić kontener Dropbox'a od klucza jakiejś zaszyfrowanej już partycji. Można też
wykasować hasło do kontenera ale wtedy uzależnimy kontener zupełnie od nagłówków tej partycji.
Jeśli te nagłówki ulegną uszkodzeniu albo przez przypadek stworzymy nowy kontener pod nowy system,
stracimy dostęp do danych na Dropbox.

Przede wszystkim narzędzie `losetup` musi widzieć nasz kontener:

    #  losetup -a
    /dev/loop0: [0807]:2883594 (/media/Server/Dropbox/luks_dropbox)

### Plik /etc/crypttab

Edytujemy teraz plik `/etc/crypttab` , z tym, że konfiguracja w nim zawarta będzie się różnić w
zależności od tego czy mamy już w systemie jakieś zaszyfrowane dyski, czy tez i nie. Jeśli nie
mamy, to trzeba będzie niestety podawać na starcie systemu hasło. Można też sprecyzować keyfile,
choć na niezaszyfrowanym systemie nie polecam tego rozwiązania. Keyfile jest dopuszczalną formą
uzyskania dostępu do kontenera, tylko gdy rezyduje na zaszyfrowanej przestrzeni, w chronionej przez
roota lokalizacji (prawa 400). Także jeśli nie mamy zaszyfrowanego systemu, w pliku `/etc/crypttab`
dodalibyśmy coś na wzór tej linijki poniżej:

    crypt_dropbox             /media/Server/Dropbox/luks_dropbox   none      luks,noauto

Ja nie będę się zajmował powyższym rozwiązaniem, bo jest ono wielce niepraktyczne. Dlatego ja
wykorzystuje pełne szyfrowanie dysku i jak zobaczymy poniżej, to powoli zaczyna ułatwiać życie.

Tak powinien mniej więcej wyglądać plik `/etc/crypttab` przy posiadaniu zaszyfrowanego dysku
systemowego:

    sda2_crypt         UUID=727fa348-8804-4773-ae3d-f3e176d12dac   none      luks
    crypt_dropbox     /media/Server/Dropbox/luks_dropbox              sda2_crypt  luks,keyscript=/lib/cryptsetup/scripts/decrypt_derived,noauto

Teraz jeszcze trzeba wyciągnąć 128 znakowy ciąg z tabeli `dmsetup` :

    # dmsetup table --showkeys

Odczytujemy ten długi ciąg znaków przy pozycji `sda2_crypt` i dodajemy go do nagłówka kontenera LUKS
jako hasło:

    # cryptsetup luksAddKey /media/Server/Dropbox/luks_dropbox

Sprawdzamy czy są zajęte dwa sloty:

    # cryptsetup luksDump /media/Server/Dropbox/luks_dropbox
    ...
    Key Slot 0: ENABLED
    ...
    Key Slot 1: ENABLED
    ...

Zamykany kontener:

    # cryptsetup luksClose crypt_dropbox

    # losetup -D

### Plik /etc/fstab

Otwórzmy raz jeszcze kontener z plikami Dropbox'a, bo jakby nie patrzeć, to nie jest koniec naszej
pracy. Musimy min. dodać odpowiedni wpis w `/etc/fstab` . By to zrobić trzeba uzyskać UUID systemu
plików w kontenerze:

    # tune2fs -l /dev/mapper/crypt_dropbox | grep UUID
    Filesystem UUID:          0d959e74-ec19-43bf-b779-60134c676aef

Edytujemy plik `/etc/fstab` i dodajemy tam poniższy wpis:

    # dropbox
    UUID=0d959e74-ec19-43bf-b779-60134c676aef /media/Dropbox    ext4  defaults,noauto,user,nofail,noatime,commit=20   0 2

Tworzymy także punkt montowania w katalogu `/media/` :

    # mkdir /media/Dropbox

### Generowanie nowego initramfs

Po dostosowaniu plików `/etc/crypttab` i `/etc/fstab` musimy wygenerować nowy initramfs przy pomocy
poniższego polecenia:

    # update-initramfs -u -k all

W tej chwili system będzie w stanie wykorzystać hexalną wartość klucza partycji systemowej do
odblokowania kontenera przeznaczonego pod pliki Dropbox'a. Możemy sprawdzić czy tak faktycznie się
stanie. By otworzyć kontener wydajemy poniższe polecenie:

    # cryptdisks_start crypt_dropbox

By zamknąć:

    # cryptdisks_stop crypt_dropbox

Skrypty `cryptdisks_start` oraz `cryptdisks_stop` działają w oparciu o plik `/etc/crypttab` i by z
nich skorzystać, trzeba mieć w tym pliku zdefiniowane kontenery.

### Problemy z odblokowaniem kontenera LUKS

Przy otwieraniu kontenera możemy napotkać na pewien problem. Mianowicie, ten kontener znajduje się
na partycji, a jej system plików z kolei trzeba pierw zamontować by uzyskać dostęp do znajdującego
się na nim kontenera. To powoduje, że nie można ustawić automatycznego otwarcia kontenera przy
starcie systemu w `/etc/crypttab` . Jeśli się przyjrzymy bliżej temu plikowi, to dostrzeżemy tam
opcję `noauto`. Jeśli nie można otworzyć kontenera na starcie, nie można też zamontować
automatycznie jego systemu plików. A to z kolei obeszliśmy przez opcję `noauto` w pliku
`/etc/fstab` .

Problem jest jeszcze taki, że mimo iż kontener nie jest otwierany, a jego system plików nie jest
montowany to i tak podczas startu systemu jest wyrzucany błąd o braku urządzenia z UUID naszego
kontenera. Opcja `nofail` w `/etc/fstab` rozwiązuje ten problem i tłumi ten komunikat.

## Automatyczne montowanie systemu plików kontenera

Obecnie, za sprawą opcji, które ustawiliśmy, ani kontener nie jest otwierany na starcie, ani system
plików nie jest montowany. To chyba nie tak miało wyglądać, prawda? Oczywiście, że nie. Musimy
stworzyć dwa pliki w katalogu `/etc/systemd/system/` , które będą wchodzić w skład potrzebnej nam
usługi.

Plik `systemd-cryptsetup@dropbox.service` :

    [Unit]
    Description=Cryptography Setup for %I
    Documentation=man:cryptdisks_start man:cryptdisks_stop man:systemd-cryptsetup@.service(8)
    IgnoreOnIsolate=true
    DefaultDependencies=no
    Requires=media-Kabi.mount
    After=cryptsetup-pre.target
    After=systemd-fsck-root.service
    After=media-Kabi.mount
    Before=systemd-fsck@dev-mapper-crypt_dropbox.service
    Before=cryptsetup.target
    Before=media-Dropbox.mount
    Before=umount.target shutdown.target
    Conflicts=umount.target shutdown.target
    ConditionPathExists=/media/Kabi/luks_dropbox

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    TimeoutSec=30
    ExecStart=/lib/systemd/systemd-cryptsetup attach 'crypt_dropbox' '/media/Kabi/luks_dropbox' 'none' 'luks'
    ExecStop=/lib/systemd/systemd-cryptsetup detach 'crypt_dropbox'

    [Install]
    WantedBy=cryptsetup.target

Plik `media-Dropbox.mount` :

    [Unit]
    Documentation=man:fstab(5)
    Requires=systemd-fsck@dev-mapper-crypt_dropbox.service
    Requires=systemd-cryptsetup@crypt_dropbox.service
    Requires=media-Kabi.mount
    After=systemd-fsck@dev-mapper-crypt_dropbox.service
    After=systemd-cryptsetup@crypt_dropbox.service
    After=media-Kabi.mount
    Before=local-fs.target

    [Mount]
    What=/dev/mapper/crypt_dropbox
    Where=/media/Dropbox
    Type=ext4
    Options=defaults,errors=remount-ro,commit=20,loop=/dev/loop5

Dodajemy jeszcze `systemd-cryptsetup@dropbox.service` do autostartu systemu:

    # systemctl enable systemd-cryptsetup@dropbox.service

I to by było w zasadzie wszystko. Po starcie systemu, kontener zostanie otworzony a jego system
plików zamontowany. Przy czym, taki otwarty kontener jest przez dropboxa traktowany jako plik w
fazie edycji.

## Ochrona kontenera z plikami Dropbox'a

Jeszcze kilka słów na temat tego w jaki sposób dostęp do kontenera dropbox powinien być chroniony.
Słabe hasło najlepiej jest zastąpić przez keyfile. Oczywiście sposób uzyskiwania dostępu do
kontenera zostaje taki sam ale na wypadek problemów z zależną partycją musimy mieć jakieś
zabezpieczenie, by dane z Dropbox'a nie przepadły. Najlepiej stworzyć keyfile w oparciu o plik
binarny, który jest trzymany "w głębokim ukryciu". Głębokie ukrycie będzie polegało na wybraniu
jakiegoś pliku, który rezyduje na naszym dysku ale jest już używany w jakiś sposób, np. plik mp3 czy
avi.

Teraz przy pomocy `--new-keyfile-size=` oraz `--new-keyfile-offset=` możemy wskazać, od której
części pliku ma się zaczynać nasz keyfile oraz jak długi ma on być. Przykład wykorzystania takiego
mechanizmu:

    cryptsetup luksAddKey --new-keyfile-size=2048 --new-keyfile-offset=4096 '/media/Server/Dropbox/luks_dropbox' '/media/mp3/mp3.mp3'

Jeśli spróbowalibyśmy otworzyć ten kontener przy pomocy powyższego pliku, nie uda nam się ta
operacja:

    # cryptsetup luksOpen  '/media/Server/Dropbox/luks_dropbox' crypt_dropbox --key-file='/media/mp3/mp3.mp3'
    No key available with this passphrase.

Podobnie sprawa ma się w przypadku podania niewłaściwych parametrów, tj. rozmiaru i offsetu pliku
klucza:

    # cryptsetup luksOpen '/media/Server/Dropbox/luks_dropbox' crypt_dropbox --key-file='/media/mp3/mp3.mp3' --keyfile-offset=4095 --keyfile-size=2048
    No key available with this passphrase.

    # cryptsetup luksOpen '/media/Server/Dropbox/luks_dropbox' crypt_dropbox --key-file='/media/mp3/mp3.mp3' --keyfile-offset=4096 --keyfile-size=2047
    No key available with this passphrase.

Powyżej został pokazany prosty przykład, jednak wartościami `offset` i `size` można dowolnie
manipulować i ustawić można sobie dowolne wartości i nawet jeśli już ktoś wejdzie w posiadanie
naszego pliku, to będzie musiał znać 2 parametry by odblokować kontener. Możliwe jest, co prawda,
zdefiniowanie tego keyfile w `/etc/crypttab` i użycie go do odblokowania kontenera ale ja bym tego
nie robił. Można tym zdradzić położenie keyfile no i oczywiście wartości `offset` i `size` . Lepiej
zostawić ten keyfile jako backup.

W przypadku zrobienia backupu w postaci keyfile, trzeba pamiętać też o wyczyszczeniu odpowiednich
slotów w nagłówku kontenera LUKS. Służy do tego `cryptsetup luksKillSlot`:

    # cryptsetup luksKillSlot '/media/Server/Dropbox/luks_dropbox' 0
    Enter any remaining passphrase:


[1]: /post/implementacja-encfs-na-dropboxie/
[2]: /post/kontener-truecrypt-trzymany-na-dropboxie/
[3]: https://pl.wikipedia.org/wiki/Linux_Unified_Key_Setup
