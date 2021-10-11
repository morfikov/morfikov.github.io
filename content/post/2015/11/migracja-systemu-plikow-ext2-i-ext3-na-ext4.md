---
author: Morfik
categories:
- Linux
date: "2015-11-12T14:16:49Z"
date_gmt: 2015-11-12 13:16:49 +0100
published: true
status: publish
tags:
- system-plików
- ext4
GHissueID: 241
title: Migracja systemu plików ext2 i ext3 na ext4
---

Dyski twarde są w stanie pomieścić setki gigabajtów danych. Ilość informacji, które jesteśmy w
stanie przechować na pojedynczym nośniku, rośnie w zastraszającym tempie. Rozwój technologii nie
jest jedynym polem gdzie prowadzone są prace nad nowymi rozwiązaniami poprawiającymi szereg aspektów
pracy tych urządzeń. Innym polem jest sfera programowa, która w przypadków dysków twardych w dużej
mierze dotyczy systemu plików. Albowiem każda powierzchnia, na której mają być przechowywane dane,
potrzebuje odpowiedniej struktury, którą również można usprawnić. Wobec czego, ten domyślny system
plików w linux'ie, tj. `ext` , przeszedł szereg modyfikacji i pojawiły się wersje `ext2` , `ext3` i
`ext4` . Jeśli jakaś partycja dysku twardego zawiera starszą wersję systemu plików, powinniśmy
dokonać migracji na jego nowszy odpowiednik. W przypadku migracji z `ntfs` na `ext4` (czy też
odwrotnie), nieunikniona jest utrata danych. Czy w przypadku migracji z systemu plików `ext2` i
`ext3` na `ext4` również musimy zgrywać wszystkie dane na osobny nośnik by przeformatować
odpowiednio taki dysk czy partycję? Okazuje się że nie musimy i możemy dokonać takiej migracji bez
obaw o utratę danych i w tym wpisie postaramy się ten zabieg przeprowadzić.

<!--more-->
## Ustalanie systemu plików partycji

Zacznijmy zatem od ustalenia systemu plików na przykładowej partycji. Jest wiele narzędzi i metod,
które mogą nam w tym pomoc. Jeśli dany system plików jest zamontowany w systemie, to możemy
skorzystać z polecenia `mount` czy też `df` . W przypadku gdy danej partycji nie możemy z jakiegoś
powodu zamontować, to zawsze możemy posłużyć się `lsblk` , przykładowo:

    # lsblk /dev/sdb
    NAME    SIZE FSTYPE TYPE LABEL MOUNTPOINT UUID
    sdb    14.5G        disk
    ├─sdb1    5G vfat   part                  7C40-491B
    ├─sdb2    3G ext3   part                  24a8233f-1ee9-4e64-9def-5eb7ddec57a6
    └─sdb3  6.5G ext2   part                  bb3c781e-8e86-4453-bc2f-06ee09fb6062

W powyższym przypadku mamy partycje z systemem plików `ext2` i `ext3` . Na obu z nich są wgrane
różne pliki i przydałoby się dokonać migracji ich systemu plików.

## Migracja z ext2 i ext3 na ext4

Migracja ze starszych wersji, w przypadku systemu plików z rodziny `ext` , polega głównie na dodaniu
pewnych opcji, które są charakterystyczne dla danej wersji. Główną różnicą między `ext2` i `ext3`
jest dziennik, za który odpowiada opcja `has_journal` . Z kolei różnica między `ext3` i `ext4`
dotyczy tych trzech opcji: `extents` , `uninit_bg` oraz `dir_index` . Zatem by zmigrować `ext2` na
`ext4` , musimy uwzględnić wszystkie cztery opcje. W tym drugim przypadku, tylko te trzy ostatnie.

Odmontowujemy zatem system plików, który chcemy zmigrować. Następnie korzystamy z narzędzia
`tune2fs` (dostępne w pakiecie `e2fsprogs` ) i przy pomocy parametru `-O` (duże o) dodajemy
odpowiednie opcje.

Dla systemu plików `ext2` polecenie wygląda tak jak poniżej:

    # tune2fs -O extents,uninit_bg,dir_index,has_journal /dev/sdb3

Dla systemu plików `ext3` zaś tak:

    # tune2fs -O extents,uninit_bg,dir_index /dev/sdb2

Po tych powyższych operacjach, system plików powinien ulec zmianie na `ext4` w przypadku obu
partycji. Sprawdźmy przy pomocy `lsblk` czy tak się w istocie stało:

    # lsblk /dev/sdb
    NAME    SIZE FSTYPE TYPE LABEL MOUNTPOINT UUID
    sdb    14.5G        disk
    ├─sdb1    5G vfat   part                  7C40-491B
    ├─sdb2    3G ext4   part                  24a8233f-1ee9-4e64-9def-5eb7ddec57a6
    └─sdb3  6.5G ext4   part                  bb3c781e-8e86-4453-bc2f-06ee09fb6062

Należy odnotować fakt, że numery UUID nie uległy zmianie. Mogłoby się zatem wydawać, że nie musimy
przeprowadzać żadnych dodatkowych czynności konfiguracyjnych, jak np. dostosowanie pliku
`/etc/fstab` . Nie jest to prawdą i plik `fstab` musimy dostosować tak czy inaczej, a to z tego
względu, że jest w nim określony typ systemu plików, który wskazuje na `ext2` i `ext3` . Powinniśmy
te pozycje przepisać na `ext4` . Należy także przeskanować system plików na tych partycjach w
poszukiwaniu ewentualnych błędów przy pomocy `fsck.ext4` .
