---
author: Morfik
categories:
- Linux
date: "2016-01-01T17:18:03Z"
date_gmt: 2016-01-01 16:18:03 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- luks
- szyfrowanie
- systemd
title: Odszyfrowanie kontenerów LUKS w systemd
---

Osoby, które wykorzystują pełne szyfrowanie dysku, wiedzą, że nie zawsze da radę tak skonfigurować
swój system, by upchnąć go na jednej partycji. Nawet jeśli korzystamy z LVM wewnątrz kontenera LUKS,
to i tak zwykle możemy posiadać inne zaszyfrowane partycje, które są odrębną całością i montowane
osobno przy starcie systemu. W takim przypadku zwykle jesteśmy zmuszeni do podawania hasła do
każdego z tych dysków z osobna, co zajmuje czas. Ten problem opisałem po części przy okazji
[implementacji kontenera LUKS na potrzeby
Dropbox'a]({{< baseurl >}}/post/dropbox-i-kontener-luks/), jak i we wpisie poświęconym [przejściu
z kontenerów TrueCrypt'a na te linux'owe, które są wspierane natywnie przez sam
kernel]({{< baseurl >}}/post/przejscie-z-truecrypt-na-luks/). Niemniej jednak, tamto rozwiązanie
było oparte głównie o starszy init (sysvinit), co wymagało dodatkowej konfiguracji, tak by system
otworzył się po podaniu tylko jednego hasła. W tym wpisie postaramy się wdrożyć mechanizm, który
jest oferowany przez systemd.

<!--more-->
## Systemd i keyring kernela

Systemd od jakiegoś czasu jest w stanie wykorzystywać [keyring
kernela](https://www.kernel.org/doc/Documentation/security/keys.txt), co przekłada się na
przechowywanie w nim pewnych haseł. Takie hasła mogą być wykorzystywane, np. [przy zaszyfrowanym
systemie plików oferowanym przez
ecryptfs]({{< baseurl >}}/post/ecryptfs-jako-alternatywa-dla-encfs/). Mogą one także zostać
wykorzystane we wczesnej fazie startu systemu. Każdy użytkownik ma swój własny keyring i nie może
podglądać haseł innych użytkowników. Same hasła zaś mogą mieć ustawiony termin ważności. W takiej
sytuacji, gdy otworzymy systemowy kontener podczas fazy boot, wpisana fraza (hasło) zostanie dodana
do keyring'a na pewien określony czas. Jeśli mamy kilka dodatkowych kontenerów i każdy z nich ma
ustawione to samo hasło, to zostaną one również odszyfrowane właśnie przy pomocy tego hasła, który
został dodany do keyring'a kernela. Po odszyfrowaniu kontenerów, to hasło jest już zwyczajnie zbędne
i może zostać usunięte z keyring'a.

## Skrypty od cryptsetup

We wstępie wspomniałem, że wykorzystując stary mechanizm potrzebne nam są skrypty i odpowiednia
konfiguracja. W debianie skrypty, o których mowa, są dostarczane wraz z pakietem `cryptsetup` .
Natomiast jeśli chodzi o konfigurację, to interesuje nas głównie plik `/etc/crypttab` , który
powinien wyglądać mniej więcej tak:

    # cat /etc/crypttab
    # target name source device                                                         key file      options
    sda1_crypt      UUID=1820485b-6587-47ab-baa4-761ecc104c25       none            luks
    kabi            UUID=f3c10054-0583-4e10-937b-dcdc9a05a25c       sda1_crypt      luks,keyscript=/lib/cryptsetup/scripts/decrypt_derived,noauto,nofail
    grafi           UUID=d314ed20-ffaf-4a18-98a7-91538e79d981       sda1_crypt      luks,keyscript=/lib/cryptsetup/scripts/decrypt_derived,noauto,nofail

Niestety, [systemd nie potrafi jeszcze zrozumieć parametru
keyscript](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=618862). Dlatego też ta powyższa
konfiguracja nie zadziała z tym init'em. Na szczęście systemd ma własny mechanizm, który eliminuje
potrzebę precyzowania opcji `keyscript` .

## Usługi dla systemd

Niewiele osób wie, że systemd nie korzysta bezpośrednio z plików `/etc/crypttab` i `/etc/fstab` . Ma
on zaś kilka generatorów, które w oparciu o te powyższe pliki generują poszczególne usługi. Możemy
się o tym przekonać zaglądając do katalogu `/run/systemd/generator/` . Powinniśmy tam ujrzeć szereg
plików mających końcówki `.mount` . Nazwy tych plików powinny być nam raczej znajome, a ich liczba
zależy od ilości wpisów zdefiniowanych w pliku `fstab` .

Podobnie sprawa ma się w przypadku kontenerów LUKS, z tym, że interesujące nas pliki zaczynają się
od `systemd-cryptsetup@` . Zawierają one konfigurację zdefiniowaną w pliku `crypttab` i ich liczba
również zależy od ilości zdefiniowanych tam wpisów.

Generalnie rzecz biorąc, w większości przypadków, proces odszyfrowania i montowania zasobów powinien
przebiegać automatycznie. Unikajmy jednak dublowania konfiguracji i jeśli definiujemy jakieś wpisy w
plikach `fstab` i `crypttab` , to nie piszmy osobnych usług pod te zasoby.

Poniżej są przykładowe pliki `systemd-cryptsetup@` i `.mount` na wypadek, gdyby ktoś chciał sobie
takie usługi napisać:

Plik usługi otwierającej zaszyfrowany kontener LUKS,
`/etc/systemd/system/systemd-cryptsetup@kabi.service` :

    [Unit]
    Description=Cryptography Setup for %I
    Documentation=man:man:systemd-cryptsetup@.service(8)
    IgnoreOnIsolate=true
    DefaultDependencies=no
    After=dev-disk-by\x2duuid-f3c10054\x2d0583\x2d4e10\x2d937b\x2ddcdc9a05a25c.device
    After=cryptsetup-pre.target
    After=systemd-fsck-root.service
    Before=systemd-fsck@dev-mapper-kabi.service media-Kabi.mount
    Before=cryptsetup.target
    Before=umount.target shutdown.target
    Conflicts=umount.target shutdown.target
    BindsTo=dev-mapper-%i.device
    BindsTo=dev-disk-by\x2duuid-f3c10054\x2d0583\x2d4e10\x2d937b\x2ddcdc9a05a25c.device

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    TimeoutSec=30
    ExecStart=/lib/systemd/systemd-cryptsetup attach 'kabi' '/dev/disk/by-uuid/f3c10054-0583-4e10-937b-dcdc9a05a25c' 'none' 'luks'
    ExecStop=/lib/systemd/systemd-cryptsetup detach 'kabi'

    [Install]
    WantedBy=cryptsetup.target

W sekcji `[Unit]` , na pozycjach `After` oraz `BindsTo` mamy określone linki do urządzeń w katalogu
`/dev/disk/by-uuid/` . Natomiast `\x2d` oznacza `-` . Dlatego te ścieżki tak dziwacznie wyglądają. Z
kolei sekcja `[Service]` wygląda bardzo znajomo, zwłaszcza pozycja `ExecStart` . Jest to dokładnie
ta sama linka, która widnieje w pliku `/etc/crypttab` , z tym, że bez zbędnych opcji.

Plik usługi montującej odszyfrowany zasób, `/etc/systemd/system/media-Kabi.mount`:

    [Unit]
    Documentation=man:fstab(5)
    Requires=systemd-fsck@dev-mapper-kabi.service
    Requires=systemd-cryptsetup@kabi.service
    After=systemd-fsck@dev-mapper-kabi.service
    After=systemd-cryptsetup@kabi.service
    Before=cloud_storage-mega.encrypted.service
    Before=qbittorrent-nox.service
    Before=local-fs.target

    [Mount]
    What=/dev/disk/by-uuid/b47e6dcd-924e-40fa-a8b1-7593419f86d7
    Where=/media/Kabi
    Type=ext4
    Options=defaults,errors=remount-ro,commit=10

Tutaj podobnie, sekcja `[Mount]` jest żywcem wyjęta z pliku `/etc/fstab` .

W obu powyższych przypadkach możemy manipulować kolejnością odpalania usług. W taki sposób mamy
pewność, że określone zasoby są zawsze zamontowane i gotowe do użycia, gdy usługa ich wymagająca
zostanie odpalona. W przeciwnym razie, ta usługa zwyczajnie się nam nie uruchomi, ewentualnie
pociągnie za sobą start tych powyższych usług.

## Jak długo kernel przechowuje hasła do kontenerów LUKS?

To jak długo hasło do kontenera LUKS jest przechowywane w keyring'u kernela zależy od konfiguracji
systemd. Domyślnie jest to `60s` . Niemniej jednak, każdorazowe skorzystanie z takiego klucza, tj.
otwieranie kolejnego kontenera, odświeża ten czas i te 60 sekund należy liczyć od ostatniego użycia
klucza. Zatem jest wielce prawdopodobne, że ten klucz straci ważność chwile po starcie systemu i nie
da rady odczytać hasła:

    # keyctl list @u
    1 key in keyring:
    402183021: --alswrv     0     0 user: cryptsetup

    # keyctl list @u
    1 key in keyring:
    402183021: key inaccessible (Key has expired)

    # keyctl print 402183021
    keyctl_read_alloc: Key has expired

## Bardzo wolne odszyfrowanie kontenerów LUKS

Do tej pory korzystałem ze skryptów dostarczanych w pakiecie `cryptsetup` . Przy przejściu na
mechanizm oferowany przez kernel linux'a oraz systemd, moje kontenery LUKS otwierały się
dramatycznie wolno. Był to rząd wielkości parunastu sekund, co efektywnie spowalniało start systemu.
Początkowo myślałem, że może to są jakieś błędy w samym systemd ale ostatecznie udało mi się ten
problem rozwiązać. Okazało się bowiem, że znaczenie ma to, który keyslot w nagłówku kontenera
zawiera hasło mające być wykorzystane do automatycznego odszyfrowania kontenera, a tych slotów jest
8 (do podejrzenia w `cryptsetup luksDump` ).

W przypadku, gdy wykorzystujemy tylko jeden keyslot, to nie musimy się o nic martwić. Problem
pojawia się, gdy tych haseł mamy kilka. W takim przypadku musimy wskazać, z którego keyslot'a
zamierzamy korzystać przy pomocy opcji `key-slot` w pliku usługi. Jeśli tego nie zrobimy, to każdy
kolejny keyslot będzie sprawdzamy pod kątem wpisanego hasła. Poniżej mamy porównanie dwóch zasobów,
z których jeden ma hasło na pozycji 0, natomiast drugi na pozycji 2:

    # systemd-analyze blame
          10.234s systemd-cryptsetup@kabi.service
          2.693s systemd-cryptsetup@grafi.service

Zatem różnica jest znaczna. Im więcej mamy haseł i im dalej się znajduje to, które ma zostać użyte w
procesie odszyfrowania, tym wolniej będzie przebiegał proces automatycznego otwierania takiego
kontenera LUKS podczas startu systemu. Zatem jeśli mamy zajętych kilka keyslot'ów, przeróbmy linijkę
z `ExecStart` do poniższej
    postaci:

    ExecStart=/lib/systemd/systemd-cryptsetup attach 'kabi' '/dev/disk/by-uuid/f3c10054-0583-4e10-937b-dcdc9a05a25c' 'none' 'luks,key-slot=3'
