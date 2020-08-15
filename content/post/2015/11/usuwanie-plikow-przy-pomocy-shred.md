---
author: Morfik
categories:
- Linux
date: "2015-11-24T15:11:32Z"
date_gmt: 2015-11-24 14:11:32 +0100
published: true
status: publish
tags:
- system-plików
- pliki
- foldery
- prywatność
title: Całkowite usuwanie plików przy pomocy shred
---

W przypadku, gdy musimy pozbyć się jakiegoś pliku, który znajduje się na dysku, to nie jest zalecane
korzystanie z narzędzia `rm` . Usuwa ono jedynie odnośnik do pliku, który go identyfikuje w
strukturze systemu plików, tzw. [i-węzeł](https://pl.wikipedia.org/wiki/I-w%C4%99ze%C5%82) (i-node).
To co uzyskujemy za pomocą takich narzędzi jak `rm` , to jedynie oznaczenie pewnych bloków (tych od
pliku) jako wolne, w których system operacyjny będzie w stanie dokonać zapisu danych późniejszym
czasie. Podczas tej operacji nie są usuwane żadne informacje z dysku, a mając na uwadze ten fakt,
możemy bez problemu tak "usunięty" plik odzyskać. By mieć pewność, że plik zostanie trwale
zniszczony, trzeba go ponownie napisać, np. przy pomocy
[shred](http://manpages.ubuntu.com/manpages/wily/en/man1/shred.1.html), który standardowo jest
dostępny w każdej dystrybucji linux'a i to jemu będzie poświęcony ten wpis.

<!--more-->
## Usuwanie plików, a dziennik systemu plików

Gdy nasz linux działa na systemie plików z rodziny `ext*` (obecnie `ext4` ), to wszystkie narzędzia
mające na celu przepisanie odpowiednich bloków mogą nie być skuteczne w zacieraniu śladów po
usuniętych plikach. Zatem trzeba się liczyć z faktem, że po nadpisaniu przez `shred` plik na dysku,
informacje o tej operacji będą znajdować się w dzienniku systemu plików.

W zależności od konfiguracji systemu plików w `/etc/fstab` , dziennik może zawierać jedynie metadane
dokonanych zmian ale może także zawierać również i realne dane. W pierwszym przypadku może i nie
damy rady odzyskać pliku jako takiego ale możemy pozyskać informacje na temat jego wielkości oraz
nazwy. W drugim przypadku zaś, kopia pliku znajduje się w dzienniku i można ją bez problemu wydobyć.

Jednym z wyjść byłoby zatem dostosowanie konfiguracji w pliku `fstab` . Zgodnie z oficjalną
[dokumentacją systemu plików ext4](https://www.kernel.org/doc/Documentation/filesystems/ext4.txt),
możemy określić kilka trybów: `data=writeback` , `data=ordered` oraz `data=journal` . Te dwa
pierwsze logują w dzienniku jedynie metadane, natomiast ten trzeci również faktyczne dane. Domyślnym
trybem jest `data=ordered` , zatem nie powinniśmy tutaj nic zmieniać.

Tak czy inaczej w dzienniku zostaną zalogowane metadane plików i by tego uniknąć trzeba by
zrezygnować z opcji systemu plików odpowiedzialnej za księgowanie. Trzeba tylko pamiętać o tym, że
to może prowadzić do utraty danych w przypadku awarii zasilania czy zwykłego zawieszenia się
systemu. Można by oczywiście tymczasowo usunąć dziennik, skasować wszystkie niewygodne pliki, po
czym ten dziennik utworzyć na nowo. By usunąć dziennik, trzeba pierw odmontować partycję, po czym
wydać to poniższe polecenie:

    # tune2fs -O ^has_journal /dev/sda6

Teraz można zamontować partycję i usunąć wszystkie zbędne pliki. Gdy skończymy, to dziennik tworzymy
w poniższy sposób:

    # tune2fs -j /dev/sda6

## Usuwanie plików

Usuwanie plików przy pomocy `shred` trwa nieco dłużej niż ma to miejsce w przypadku `rm` . Trzeba
przecież zapisać poszczególne bloki, a im większy jest plik, tym ta operacja zajmie odpowiednio
więcej czasu. Pliki usuwamy zaś w poniższy sposób:

    $ shred -f -v -u -z -n 3 plik

Poniżej wyjaśnienie opcji:

  - `-f` -- tryb siłowy, który ma na celu przepisanie uprawnień by umożliwić zapis pliku.
  - `-v` -- tryb rozmowy pokazujący większą ilość informacji.
  - `-u` -- przycina i usuwa plik pod nadpisaniu go.
  - `-z` -- zerowanie pliku po ostatnim nadpisaniu pliku.
  - `-n` -- ilość przepisań, domyślnie 3.

To w zasadzie cała procedura bezpiecznego usuwania plików z dysku. Jeśli chcemy usunąć wszelkie dane
z konkretnego nośnika, to dobrze jest zainteresować się tematem
[zerowania](https://pl.wikipedia.org/wiki/Kasowanie_danych) czy też [ATA Secure
Erase](https://ata.wiki.kernel.org/index.php/ATA_Secure_Erase).
