---
author: Morfik
categories:
- Linux
date: "2015-06-19T20:11:42Z"
date_gmt: 2015-06-19 18:11:42 +0200
published: true
status: publish
tags:
- pendrive
- cd
- dvd
- hdd
- ssd
title: Udevil i montowanie zasobów bez uprawnień root
---

Linux jest bezpiecznym środowiskiem operacyjnym ale to nie ze względu na to, że jego kod jest jakoś
mniej podatny na błędy czy coś w tym stylu, tylko przez restrykcyjną politykę dostępu do szeregu
miejsc w systemie. Jednym z nich są wszelkie urządzenia, w skład których wchodzą również i dyski
twarde, pendrive czy napędy cd/dvd. Oczywiście tych urządzeń może być o wile więcej ale my w tym
wpisie omówimy sobie dostęp tylko do tych trzech wymienionych wyżej. Standardowo zwykły użytkownik w
systemie nie ma możliwości przeprowadzenia szeregu czynności pod kątem większości z tych urządzeń i
wliczyć w to można, np. montowanie zasobów. Tego typu operacje może przeprowadzać jedynie
administrator. Ma to na celu ochronę bezpieczeństwa systemu. Jeśli chodzi o nośniki wymienne takie
jak płytki cd/dvd czy pendrive, to na nich może znajdować się podejrzane oprogramowanie i po
zamontowaniu takiego nośnika w systemie, wirusy, trojany czy keyloggery mogą się wgrać do systemu
zagrażając tym samym prywatności wszystkich użytkowników.

<!--more-->
## Montowanie z uprawnieniami czy bez uprawnień

[Automatyczne montowanie
zasobów](/post/autostart-i-automatyczne-montowanie-nosnikow/) zostało już
poruszone wcześniej. Tutaj zajmiemy się tylko i wyłącznie możliwością montowania urządzeń jako
zwykły użytkownik, a to czy one będą montowane automatycznie, zależy już od każdego z nas.

Przede wszystkim, trzeba mieć na uwadze, że może i podpięte do komputera urządzenie nie zostanie
zamontowane ani żadne programy na takim nośniku nie zostaną uruchomione ale też nie zawsze chronimy
dostęp do swojego komputera tak jak powinniśmy. Ile razy w ciągu tylko tego pojedynczego dnia
zostawiliśmy swój komputer bez nadzoru? W przypadku gdy tracimy z maszyną fizyczny kontakt i nie
blokujemy sesji/ekranu, ktoś może uzyskać dostęp do naszego komputera, podpiąć pendrive czy wsadzić
płytkę cd/dvd do napędu i zamontować trefny zasób.

W przypadku gdy nie czujemy się na siłach bronić nieustannie naszego systemu, to nie powinniśmy
zezwalać na montowanie zasobów bez uprawnień zwykłym użytkownikom. W takim przypadku tylko root
powinien mieć możliwość interakcji z nowymi zasobami. Nawet jeśli ktoś niepowołany uzyskałby dostęp
do komputera, to nie będzie w stanie wgrać/zgrać żadnych danych, przynajmniej za pomocą lokalnych
urządzeń. Inną kwestią jest naturalnie dostęp do internetu i możliwość pobrania z niego określonych
plików, czy też przesłania pewnych danych na jakiś serwer. W każdym razie, zabezpieczenia są głównie
po to, by utrudnić potencjalnemu atakującemu ukrycie faktu, że dokonał on ataku i lepiej nie
ułatwiać mu zadania.

Jeśli jednak potrafimy zadbać o fizyczny dostęp do swojej maszyny, to możemy lekko poluzować swoje
zabezpieczenia odnośnie dostępu do nośników wymiennych i umożliwić montowanie ich konkretnym
użytkownikom. Są różne mechanizmy, które mogą nam w tym pomóc i chyba tym najbardziej popularnym
jest [udisks2](http://storaged.org/doc/udisks2-api/latest/) . Choć z tego co można wyczytać [choćby
tutaj](https://igurublog.wordpress.com/2012/03/11/udisks2-another-loss-for-linux/) , to narzędzie
jest raczej sprzęgnięte ze środowiskiem GNOME, a spora część użytkowników woli jakieś lżejsze
alternatywy i taką jest [udevil](https://ignorantguru.github.io/udevil/) . Pakiet jest dostępny w
debianie, także nie powinno być problemów z jego zainstalowaniem.

## Udevil

Narzędzie `udevil` posiada pliki konfiguracyjne w katalogu `/etc/udevil/` i domyślnie jest tam jeden
plik: `udevil.conf` . Jest to konfiguracja systemowa określająca co i gdzie może być montowane. W
powyższym katalogu można także utworzyć plik z konfiguracją użytkownika, dzięki czemu każdy
użytkownik w systemie może mieć inne uprawnienia do określonych zasobów. Ten plik musi posiadać
nazwę podobną do tej: `udevil-user-morfik.conf` . Można zrobić kopię konfiguracji systemowej i
pozmieniać tylko odpowiednie linijki.

Ja na swoje potrzeby zmieniłem te cztery poniższe wpisy:

    allowed_types = $KNOWN_FILESYSTEMS, file, nfs, curlftpfs, sshfs
    allowed_media_dirs = /media/$USER, /media, /run/media/$USER, /mnt
    allowed_internal_devices = /dev/mapper/pen*
    allowed_networks = 127.0.0.1, 192.168.1.*, 192.168.2.*

Opcja `allowed_types` określa jakie typy systemów plików może montować określony użytkownik.
Standardowo są tam obecne tylko dwie pozycje: `$KNOWN_FILESYSTEMS` oraz `file` . Pierwsza z nich
odpowiada za możliwość montowania typów, które widnieją w `/proc/filesystems` , natomiast druga z
nich za różnego rodzaju obrazy `.iso` czy `.img` . Natomiast pozycje `nfs`, `curlftpfs`, `sshfs`
odpowiadają za możliwość montowania zasobów sieciowych. Dalej w pliku mamy `allowed_media_dirs` i są
to katalogi, w których użytkownik może montować wyżej wymienione zasoby. Następnie mamy
`allowed_internal_devices` i ta opcja dotyczy montowania zasobów, które nie mają ustawionej flagi
`removable` , czyli w dużej mierze dyski twarde. Co ciekawe, w przypadku pendrive, jeśli któraś z
jego partycji jest zaszyfrowana, to wtedy urządzenia pojawiają się w katalogu `/dev/mapper/` i pliki
w tym folderze są traktowane jako urządzenia wewnętrzne. By użytkownik mógł je montować bez
problemu, trzeba je w konfiguracji wyraźnie określić. Ostatnia opcja `allowed_networks` dotyczy
zawężenia kręgu adresów sieciowych, na podstawie których zasoby sieciowe mogą zostać zamontowane.
W powyższym przykładzie mam dodane zasoby lokalnej sieci. Wszystkie pozostałe w dalszym ciągu
wymagają praw administratora.

Generalnie rzecz biorąc, to opcje w pliku są bardzo dobrze i szczegółowo opisane i raczej nie
powinno być problemów z określeniem pożądanej polityki montowania urządzeń przez zwykłych
użytkowników.

Załóżmy, że chcemy zamontować jedną z partycji pendrive ale zwykły użytkownik nie ma do tego
wystarczających uprawnień. Możemy się o tym przekonać gdy spróbujemy skorzystać z `mount` :

    $ mount /dev/sdb1 /mnt/pen
    mount: only root can do that

Sprawdźmy zatem jak wygląda w praktyce posługiwanie się narzędziem `udevil` :

    $ udevil mount /dev/sdb1 /mnt/pen
    Mounted /dev/sdb1 at /mnt/pen

Warto dodać w tym miejscu, że nie musimy uprzednio tworzyć punktów montowań. Jeśli nie istnieją, to
zostaną automatycznie stworzone.

By z kolei odmontować system plików partycji, wydajemy poniższe polecenie:

    $ udevil umount /mnt/pen

Podobnie jak w przypadku tworzenia katalogu pod punkt montowania, po odmontowaniu zasobu zostanie on
automatycznie usunięty.

Przetestujmy czy zabezpieczenia odnośnie pozostałych nośników działają, tj. czy jesteśmy w stanie
zamontować jakieś urządzenia wewnętrzne:

    $ udevil mount /dev/mapper/grafi /mnt/grafi
    udevil: denied 80: device /dev/mapper/grafi is not an allowed device

Zatem nastąpiła odmowa dostępu, bo to urządzenie nie jest na liście dozwolonych.
