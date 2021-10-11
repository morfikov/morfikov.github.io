---
author: Morfik
categories:
- Linux
date: "2015-10-20T09:58:18Z"
date_gmt: 2015-10-20 07:58:18 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- luks
- szyfrowanie
GHissueID: 164
title: Przejście z Truecrypt na LUKS
---

Jakiś czas temu, można było usłyszeć, że TrueCrypt nie dba wcale o bezpieczeństwo danych
zaszyfrowanych za jego pomocą. [Audyt bezpieczeństwa](http://istruecryptauditedyet.com/) jednak nie
wykazał większych podatności w tym oprogramowaniu. [Analiza
binarek](https://madiba.encs.concordia.ca/~x_decarn/truecrypt-binaries-analysis/) dostępnych na
stronie TrueCrypt'a pod kątem [Reproducible Builds](https://wiki.debian.org/ReproducibleBuilds)
również nie wykazała większych odchyłów w stosunku do binarek generowanych prosto z kodu
źródłowego. Problematyczne może być jednak to, że tak naprawdę nie wiadomo kto stoi za tym całym
projektem, przynajmniej gdy był jeszcze rozwijany. Cała sytuacja zamknięcia TrueCrypt'a z sieci była
też co najmniej dziwna. W obliczu takich niewiadomych, powinniśmy rozważyć przejście na natywne
rozwiązania linux'owe, które są jawnie rozwijane, wiadomo kto za nimi stoi i, co najważniejsze, mają
wsparcie w samym kernelu.

<!--more-->
## Problemy wynikające ze stosowania TrueCrypt

TrueCrypt standardowo w obsłudze jest niezbyt wygodny. Co prawda można go dostosować do swoich
potrzeb, tak by montować wszystko za pomocą jednego klika ale tracimy w ten sposób kontrolę nad tym
co i jak montujemy. Przykładowo, nie można sprecyzować opcji montowania systemu plików zawartego w
otwieranych kontenerach.

W przypadku [LUKS'a](https://pl.wikipedia.org/wiki/Linux_Unified_Key_Setup) zaś, wszelkie dane
potrzebne do poprawnej obsługi systemu plików voluminu są definiowane w `/etc/fstab` , czyli tam
gdzie opcje pozostałych partycji. Mamy także zapewniony support przy sprawdzaniu systemu plików w
poszukiwaniu błędów co określoną liczbę montowań i to zaraz na starcie systemu, tuż po otworzeniu
kontenerów. Nie trzeba tego robić ręcznie. Dodatkowo, przy korzystaniu z TrueCrypt'a pod linux'em,
zakładając, że mamy zaszyfrowany system, trzeba wpisywać minimum dwa hasła, nawet w przypadku gdy są
one takie same. Po przejściu z TrueCrypt'a na LUKS, będziemy mogli odszyfrować system przy pomocy
tylko jednego hasła. Poza tym, pozbywamy się także zbędnego pakietu, którego zresztą i tak nie ma w
repozytoriach debiana.

Użytkownicy windowsa do odczytu kontenerów LUKS mogą użyć
[FreeOTFE](https://en.wikipedia.org/wiki/FreeOTFE). Niemniej jednak, mając system plików ext4 w
kontenerach TrueCrypt'a, nigdy nie udało mi się zmusić windowsa do pracy na tych voluminach. Pomijam
już zapis ext4 pod win ale nigdy nie udało mi się odczytać danych z systemu plików ext4 zawartych w
takim kontenerze. Oczywiście, zaszyfrowany kontener mający systemem plików ntfs można montować
zarówno pod windowsem jak i linux'em.

## Kontener LUKS

Wbrew pozorom samo przejście z TrueCrypt na LUKS nie jest znowu takie trudne jak mogłoby się
początkowo wydawać. Jeśli jednak zdecydujemy się pozbyć TrueCrypt'a, dane z jego kontenera trzeba
uprzednio gdzieś zgrać. A to z tego względu, że trzeba będzie utworzyć na nowo system plików. Gdy
już zgramy wszystkie pliki, możemy przystąpić do tworzenia kontenera
    LUKS:

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase luksFormat /dev/sda8
    # cryptsetup luksOpen /dev/sda8 crypt_leon
    # mkfs.ext4 -m 0 -L Leon /dev/mapper/crypt_leon
    # mount /dev/mapper/crypt_leon /media/Leon/

Po powyższej operacji jesteśmy w stanie wrzucać już pliki do zaszyfrowanej przestrzeni dysku ale to
nie jest koniec naszej pracy. Musimy jeszcze nauczyć system jak otwierać i montować taki kontener. W
zależności od tego co chcemy osiągnąć, mamy z grubsza dwie opcje do wyboru.

Pierwsza z opcji zakłada, że wszystkie kontenery będą otwierane automatycznie na starcie systemu za
pomocą tego samego hasła wpisanego tylko jeden raz. Bez znaczenia jest przy tym liczba
zaszyfrowanych dysków. Zaś druga opcja zaś opiera się na rozdzieleniu systemowych voluminów od tych
niesystemowych. Przy czym, w tym przypadku także będziemy mieli możliwość automatycznego montowania
wszystkich partycji przy starcie systemu, z tą różnicą, że te dodatkowe voluminy będą posiadać inne
hasło w stosunku do tych systemowych, czyli trzeba będzie wpisać dwa różne hasła. Naturalnie, będzie
można zrezygnować z funkcji automatycznego montowania w każdej chwili przez dopisanie parametru
`noauto` do pliku `/etc/crypttab` .

### Pliki /etc/crypttab oraz /etc/fstab

By móc zamontować automatycznie zaszyfrowane voluminy, trzeba pierw je skonfigurować przez pliki
`/etc/crypttab` oraz `/etc/fstab` . Nie będę tutaj rozdrabiał się nad opisywaniem zawartości
drugiego z tych plików, bo powinniśmy umieć na nim operować w przypadku, gdy zabieramy się za
szyfrowanie dysków. Jeśli jednak mamy problemy z ogarnięciem pliku `/etc/fstab` , to musimy zapoznać
się z [man fstab(5)](http://manpages.ubuntu.com/manpages/xenial/en/man5/fstab.5.html).

Po edycji pliku `/etc/crypttab` zawsze trzeba wygenerować nowy initramfs przy pomocy
`update-initramfs -u -k all` . Jeśli tego nie zrobimy, system będzie próbował odszyfrować kontenery
w oparciu o informacje, które były zawarte w tym pliku przed edycją.

Tutaj skupimy się głównie na pliku `/etc/crypttab` , który, w moim przypadku, wygląda mniej więcej
tak:

    # <target name>    <source device>         <key file>  <options>
    sda2_crypt        UUID=727fa348-8804-4773-ae3d-f3e176d12dac   none      luks
    crypt_leon        UUID=dc7f4586-a33d-4707-98e9-8b55c559b0d2 sda2_crypt  luks,keyscript=/lib/cryptsetup/scripts/decrypt_derived

Jak widać mamy w nim zdefiniowane dwa kontenery. Pierwszy z nich jest od partycji systemowych (LVM),
drugi zaś od nowego voluminum. Składnia pliku `/etc/crypttab` jest prosta: najpierw podawana jest
nazwa pod jaką zostanie otwarty kontener ( `crypt_leon` ), następnie jest definiowana partycja po
UUID. Trzecie pole z kolei definiuje w jaki sposób kontener zostanie otwarty. Może to być hasło albo
keyfile. Ostatnie pole to opcje.

W przypadku pierwszego kontenera, na trzeciej pozycji widzimy `none` . Oznacza to, że trzeba będzie
podać hasło na starcie systemu przy jego otwieraniu. Natomiast w przypadku drugiego kontenera,
`sda2_crypt` wskazuje volumin, którego hasło (choć to nie do końca jest to samo hasło) zostanie
wykorzystane przy otwieraniu tego zasobu. Czyli, by otworzyć `crypt_leon` potrzebne jest wpisanie
hasła do `sda2_crypt`. Po jego wpisaniu, `crypt_leon` zostanie automatycznie odblokowany. Nie jest o
jednak takie proste, bo wymagany jest skrypt `/lib/cryptsetup/scripts/decrypt_derived` , który
został dopisany w opcjach drugiego kontenera.

#### Parametr keyscript i skrypt decrypt\_derived

By móc użyć skryptu `/lib/cryptsetup/scripts/decrypt_derived` do automatycznego montowania, trzeba
pierw wygenerować keyfile, którego to zawartość zostanie na stałe dodana do nagłówka kontenera,
który ma być otwierany na starcie systemu. Można to zrobić ręcznie albo też posłużyć się w tym celu
powyższym skryptem:

    # /lib/cryptsetup/scripts/decrypt_derived sda2_crypt > klucz

Plik będzie zawierał reprezentację hexadecymalną klucza binarnego użytego do
szyfrowania/deszyfrowania danych na wskazanej partycji. Ten parametr, jak i szereg innych, można
wyciągnąć z:

    # dmsetup table --showkeys

Ciąg składa się ze 128 znaków i można go ręcznie wpisać do drugiego slota w nagłówku zaszyfrowanej
partycji albo też można w tym celu wskazać wygenerowany plik. W każdym razie oba rozwiązania mają
dokładnie taki sam cel, czyli utworzenie nowego keyslota ze 128 znakowym hasłem. Dla każdej
zaszyfrowanej partycji taki ciąg będzie inny, dlatego też nadaje się on na keyfile, do którego
dostęp można uzyskać tylko po podaniu odpowiedniego hasła. Bez hasła, klucz nie zostanie
odszyfrowany i nie będzie ciągu, który by można wydobyć z `dmsetup`. Keyfile do nagłówka drugiego
kontenera dodajemy w poniższy sposób:

    # cryptsetup luksAddKey /dev/sda8 klucz

Sprawdzamy czy mamy zajęte dwa sloty:

    # cryptsetup luksDump /dev/sda8

Jeśli tak, znaczy, że wszystko jest w porządku. Taka mała uwaga jeszcze. Jako, że w nagłówku mamy
zajęte dwa sloty, uniezależnia nas to od nagłówka partycji `sda2`, czyli możemy montować sobie dysk
na innych maszynach bez większego problemu. Można oczywiście skasować pierwszy slot z rzeczywistym
hasłem ale wtedy jeśli coś się stanie z nagłówkiem partycji systemowej, dane na `sda8` zostaną
bezpowrotnie utracone.

Cała ta zabawa z tym skryptem `decrypt_derived` przypomina mi trochę odszyfrowanie dysku przy pomocy
binarnego keyfile, który jest fizycznie zlokalizowany na innym zaszyfrowanym dysku, np w katalogu
`/root/` . Wtedy po odszyfrowaniu systemu uzyskuje się dostęp do keyfile i można zamontować kontener
przy jego pomocy. Można sprecyzować ścieżkę do binarnego keyfile w `/etc/crypttab` i na dobrą sprawę
nie ma większej różnicy między jednym i drugim rozwiązaniem. No chyba, że się nada złe prawa do
pliku, wtedy będzie można odczytać keyfile na działającej maszynie, zaś dostęp do tabeli z kluczami
w `dmsetup` wymaga uprawnień administratora systemu.

### Test kontenera LUKS

Nie musimy restartować maszyny by sprawdzić czy powyższe kroki zostały przeprowadzone prawidłowo.
Możemy zwyczajnie wpisać w terminalu to poniższe polecenie:

    # /lib/cryptsetup/scripts/decrypt_derived sda2_crypt | cryptsetup luksOpen /dev/sda8 crypt_leon

Zakładając, że pliki `/etc/crypttab` oraz `/etc/fstab` mają odpowiednie linijki oraz, że w wyniku
powyższego polecania kontener został otworzony, można być pewnym, że zostanie on otworzony również
przy starcie systemu.

### Problematyczna identyfikacja kontenerów

Trzeba także odróżnić od siebie dwa numery UUID. Odszyfrowany kontener widoczny jest w systemie w
poniższy sposób:

    # lsblk -o name,size,type,mountpoint,uuid
    NAME                             SIZE TYPE  MOUNTPOINT    UUID
    ...
    └─sda8                         100.6G part                dc7f4586-a33d-4707-98e9-8b55c559b0d2
      └─crypt_leon (dm-5)          100.6G crypt /media/Leon   5e3242e1-ec7a-4806-92f7-88a126feea94

Jak można zauważyć `sda8` oraz `crypt_leon` mają dwa różne UUID. Którego zatem mamy użyć w przypadku
pisania regułek w plikach `/etc/fstab` oraz `/etc/crypttab` ? UUID od `crypt_leon` trzeba wpisać do
`/etc/fstab` . Natomiast w `/etc/crypttab` trzeba podać UUID partycji `sda8` . Jeśli te dwa numerki
zamienimy w w/w plikach, system nie będzie w stanie odszyfrować kontenera i zamontować jego systemu
plików.

### Backup nagłów kontenera LUKS

Dobrze jest też zrobić sobie backup nagłówków zaszyfrowanych partycji i trzymać te pliki w
bezpiecznym miejscu. Backup robimy w poniższy sposób:

    # cryptsetup luksHeaderBackup /dev/sda8 --header-backup-file sda8_header_backup
