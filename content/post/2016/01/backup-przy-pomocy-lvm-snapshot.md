---
author: Morfik
categories:
- Linux
date: "2016-01-15T14:09:15Z"
lastmod: 2020-08-16 17:11:00 +0200
published: true
status: publish
tags:
- lvm
- hdd
- ssd
- pliki
- backup
GHissueID: 521
title: Backup systemu przy pomocy LVM snapshot
---

W dzisiejszych czasach systemy operacyjne są bardziej odporne na błędy niż to miało miejsce kilka
czy kilkanaście lat temu. Bardzo ciężko jest się zatem odnaleźć w sytuacji, gdzie nasz linux odmawia
współpracy i nie chce się w ogóle uruchomić. Niemniej jednak, jeśli chodzi o samą kwestię
naprawiania szkód po ewentualnej awarii systemu, to, jakby nie patrzeć, zajmuje ona nasz cenny czas.
Oczywiście takie błędy sprawiają, że mamy szansę nieco zgłębić strukturę używanego systemu
operacyjnego ale też pojawiają się one w najmniej oczekiwanym momencie. W takiej sytuacji nie ma
mowy byśmy siedzieli paręnaście minut i zastanawiali się nad tym dlaczego coś nie działa jak należy.
Jest kilka mechanizmów bezpieczeństwa, które mogą nam nieco czasu zaoszczędzić. W tym wpisie omówimy
sobie zagadnienia związane z [LVM snapshot][1], czyli migawką systemu, którą możemy wykonać
praktycznie natychmiast i w razie problemów przywrócić system do stanu sprzed wprowadzenia w nim
zmian.

<!--more-->
## LVM i jego snapshot'y

[LVM][2] to mechanizm, który umożliwia przyzwoite zarządzanie partycjami przez tworzenie dysków
wirtualnych. W taki sposób można obejść szereg ograniczeń wynikających ze stosowania, np. tablicy
partycji MS-DOS, czy też jesteśmy w stanie [dowolnie zmieniać rozmiar poszczególnych voluminów
LVM][3] nie wyłączając przy tym systemu.

Jeśli chodzi zaś o same snapshot'y, to umożliwiają one natychmiastowe dokonanie kopi pracującego
systemu. Nie ma przy tym znaczenia, czy pliki są otwarte i zmieniane. Generalnie rzecz biorąc, wraz
ze stworzeniem snapshot'a, tworzona jest także tablica, w której to są przechowywane informacje na
temat zmienianych bloków w oryginalnym voluminie. Dlatego taki snapshot systemu jest wykonywany
praktycznie od razu. W późniejszym czasie, np. podczas aktualizacji systemu, pewne bloki na dysku
systemowym ulegają zmianie ale zanim ulegną one przekształceniu, te oryginalne bloki są kopiowane i
umieszczane w voluminie snapshot'a. Jeśli coś pójdzie nie tak, to system w oparciu o te dane wie, co
i gdzie ma przywrócić.

By wdrożyć u siebie mechanizm snapshot'ów, musimy korzystać z LVM. Niekoniecznie musimy robić
migawkę dysku systemowego. Może to być dowolny volumin ale musi on być w obszarze dysku objętym
LVM. Także nie damy rady zrobić snapshot'a zewnętrznego dysku, na którym trzymamy sobie zdjęcia czy
jakieś ważne dokumenty. No chyba, że tam również zaimplementujemy LVM.

## Jak stworzyć snapshot LVM

Zakładam, że mamy już na dysku strukturę LVM i wiemy jak na niej operować. Jeśli jednak nie
stworzyliśmy jej jeszcze, to przykład jej wdrożenia możemy znaleźć, np. we wpisie poświęconym
[instalacji debiana przy pomocy debootstrap][4]. Poniżej jest
przykład takiej struktury:

    # pvscan
      PV /dev/sda1   VG lvm             lvm2 [40.00 GiB / 18.00 GiB free]
      Total: 1 [40.00 GiB] / in use: 1 [40.00 GiB] / in no VG: 0 [0   ]
    root@morfikownia:~# lvscan
      ACTIVE            '/dev/lvm/root' [18.00 GiB] inherit
      ACTIVE            '/dev/lvm/swap' [4.00 GiB] inherit

Partycja `/dev/sda1` ma 40 GiB i została przeznaczona pod fizyczny volumin LVM. Mamy tutaj utworzone
dwa dyski logiczne, które nie mają ciągłej struktury ( `inherit` ). Zajmują one łącznie 22 GiB,
zatem mamy 18 GiB wolnego miejsca. Jest to dokładnie tyle co rozmiar dysku `root` i to tę przestrzeń
wykorzystamy pod snapshot LVM. Oczywiście sam snapshot nie musi być rozmiaru backup'owanego dysku.
Ważne jest to, ile danych na źródłowej partycji będzie podlegało zmianie. Jeśli wyrobimy się w 1
GiB, to nic nie stoi na przeszkodzie, by snapshot miał rozmiar 1 GiB. Nie ma też większego sensu
wychodzenie rozmiarem poza wielkość dysku, którego snapshot robimy, no bo przecież więcej danych niż
te 18 GiB nie będziemy w stanie zmienić.

Mając już potrzebną ilość miejsca w LVM, snapshot tworzymy w poniższy sposób:

    # lvcreate -s -n snap -L 18G lvm/root
      Logical volume "snap" created.

Kluczowe jest tutaj skorzystanie z parametru `-s` oraz wskazanie voluminu, którego snapshot robimy.
W tym przypadku jest to `lvm/root` , tj. grupa `lvm` , volumin `root` . Sprawdźmy jak wygląda teraz
struktura:

    # lvscan
      ACTIVE   Original '/dev/lvm/root' [18.00 GiB] inherit
      ACTIVE            '/dev/lvm/swap' [4.00 GiB] inherit
      ACTIVE   Snapshot '/dev/lvm/snap' [18.00 GiB] inherit

Mamy trzy voluminy, przy dwóch zaś stosowne oznaczenia: `Original` i `Snapshot` . By określić w
jakim stopniu dysk `root` uległ przekształceniu w wyniku dokonywanych na nim operacji, wydajemy to
poniższe polecenie:

    # lvs
      LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root lvm  owi-aos--- 18.00g
      snap lvm  swi-aos--- 18.00g      root   0.05
      swap lvm  -wi-a-----  4.00g

Wyżej mamy szereg atrybutów. Przydałoby się więc wiedzieć co one oznaczają. Po pełną listę możliwych
opcji odsyłam do [man lvs][5]. Niżej mamy jedynie te, które pojawiły się w wyjściu `lvs` :

  - `o`rigin przy voluminie `root` wskazuje, że ten dysk jest snapshot'owany.
  - `s`napshot przy voluminie `snap` wskazuje, gdzie są przechowywane zmieniane dane dysku `root` .
  - `w`rite określa prawa dostępu. W tym przypadku jest to odczyt i zapis.
  - `i`nherited określa strukturę voluminu (nieciągłą).
  - `a`ctive mówi nam, że zasób jest aktywny.
  - `o`pen odpowiada za informację o otwarciu danego danego dysku.
  - `s`napshot wskazuje, że w przypadku voluminów `root` i `snap` wykorzystywany jest sterownik
    device-mapper oraz, że wykorzystywany jest mechanizm snapshot'ów.

Widzimy także, że `Data%` wskazuje na `0.05` , zatem tylko 0.05% z 18 GiB na dysku `root` uległo
zmianie. Co ciekawe, ten snapshot jesteśmy w stanie zamontować jak każdy inny zasób i w ten sposób
podejrzeć oryginalne pliki.

    # mount /dev/lvm/snap /mnt

    # df -Th
    Filesystem           Type      Size  Used Avail Use% Mounted on
    ...
    /dev/dm-1            ext4       18G  4.6G   13G  27% /
    ...
    /dev/mapper/lvm-snap ext4       18G  4.6G   13G  27% /mnt

Poniżej jest zaś przykład tego co się dzieje ze snapshot'em w przypadku, gdy zmieniamy szereg plików
na dysku `root` :

    # lvs
      LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root lvm  owi-aos--- 18.00g
      snap lvm  swi-a-s--- 18.00g      root   0.06
      swap lvm  -wi-a-----  4.00g
    # lvs
      LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root lvm  owi-aos--- 18.00g
      snap lvm  swi-a-s--- 18.00g      root   3.12
      swap lvm  -wi-a-----  4.00g
    # lvs
      LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
      root lvm  owi-aos--- 18.00g
      snap lvm  swi-a-s--- 18.00g      root   9.52
      swap lvm  -wi-a-----  4.00g

Jak widać, snapshot się rozrasta i obecnie zajmuje 9.52%

## Przywracanie snapshot'a

Tak stworzony snapshot nie nadaje się raczej do ciągłej pracy, przynajmniej w przypadku domowego
systemu operacyjnego. Dużo lepiej jest taki snapshot sobie stworzyć tuż przed aktualizacją systemu.
W przypadku, gdy aktualizacja się powiedzie, to tak stworzona migawka jest nam zbędna i możemy ją
zwyczajnie usunąć, np. przy pomocy tego poniższego polecenia:

    # lvremove lvm/snap

Jeśli jednak znaleźliśmy się w sytuacji, gdzie nam szereg rzeczy nagle przestał działać, to lepszym
wyjściem będzie przywrócenie systemu do stanu sprzed aktualizacji, poniżej przykład:

    # lvconvert --merge -v lvm/snap
        Using logical volume(s) on command line.
        Archiving volume group "lvm" metadata (seqno 19).
        Device dm-1 (252:1) appears to be mounted on /.
      Logical volume lvm/root contains a filesystem in use.
      Can't merge over open origin volume.
        Creating volume group backup "/etc/lvm/backup/lvm" (seqno 20).
      Merging of snapshot lvm/snap will occur on next activation of lvm/root.

W przypadku przywracania snapshot'a, zasób musi być odmontowany. Wyżej jednak próbujemy przywrócić
stan partycji systemowej, a tej nie da się odmontować. Dlatego też odtworzenie stanu tego voluminu
może mieć miejsce dopiero po ponownym uruchomieniu komputera. Po całym procesie, snapshot zostanie
usunięty zwalniając tym samy zajmowane miejsce:

    # pvscan
      PV /dev/sda1   VG lvm             lvm2 [40.00 GiB / 18.00 GiB free]
      Total: 1 [40.00 GiB] / in use: 1 [40.00 GiB] / in no VG: 0 [0   ]

Warto tutaj dodać, że ten proces odtwarzania stanu voluminu nie jest natychmiastowy. W zależności
od ilości danych, które zostały załadowane do snapshot'a, przywrócenie oryginalnego voluminu do
stanu sprzed utworzenia jego migawki może zając nawet kilka godzin. Oczywiście z systemu będzie
można korzystać w tym czasie ale będzie on nam trochę mulił albo nawet i bardzo.

## Bardzo słaba wydajność snapshot'ów

O ile snapshot'y LVM bardzo się przydają, to niestety ich problemem jest [bardzo słaba wydajność
przy operacjach zapisu][6], zwłaszcza na dyskach HDD (talerzowych). Spowolnienie pracy części
systemu w takim przypadku może sięgnąć wartości rzędu 10-20 razy, a w pewnych sytuacjach nawet i
więcej. Chodzi generalnie o to, że by zachować spójność danych snapshot'a, jego aktualizacje COW
(copy-on-write) muszą być zapisane na dysk zanim zapis do oryginalnego voluminu się dokona.W
przeciwnym wypadku, snapshot mógłby nie zawierać kopi oryginalnych danych, gdyby nastąpiła nagła
utrata zasilania. Większość dysków nie posiada wsparcia dla barier zapisu ([write barriers][7]), co
oznacza, że jedynym sposobem na zachowanie spójności danych jest synchronizowanie buforów dysku
snapshot'a przed zapisem oryginalnego voluminu. Te operacje `sync` są bardzo powolne na dyskach
talerzowych. W ten sposób, jeśli mamy utworzony taki snapshot, to asynchroniczne operacje zapisu
przechodzą w synchroniczny zapis (każdy asynchroniczny zapis może potencjalnie implikować dodatkowy
zapis synchroniczny). Idąc dalej, jeśli snapshot jest przechowywany na tym samym fizycznym nośniku,
co oryginalny volumin, to taka sytuacja dodatkowo powoduje całą masę operacji przeszukiwania
nośnika, co jeszcze bardziej spowalnia działanie snapshot'a. Jedyny plus jest taki, że to
spowolnienie ma miejsce jedynie przy zapisie części oryginalnego voluminu, które nie zostały
jeszcze skopiowane do snapshot'a. Zatem jeśli będziemy przepisywać te same bity, to po pierwszym
wolnym zapisie wydajność wróci do normy, przynajmniej dla tych danych. Dlatego też jeśli zamierzamy
korzystać ze snapshot'ów LVM, to pomyślmy o dysku SSD.


[1]: http://www.tldp.org/HOWTO/html_single/LVM-HOWTO/#snapshotintro
[2]: https://pl.wikipedia.org/wiki/LVM
[3]: /post/zmiana-rozmiaru-lvm/
[4]: /post/instalacja-debiana-z-wykorzystaniem-debootstrap/
[5]: http://manpages.ubuntu.com/manpages/wily/en/man8/lvs.8.html
[6]: https://www.nikhef.nl/~dennisvd/lvmcrap.html
[7]: https://en.wikipedia.org/wiki/Write_barrier
