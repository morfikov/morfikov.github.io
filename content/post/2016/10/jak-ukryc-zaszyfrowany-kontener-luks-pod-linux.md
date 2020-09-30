---
author: Morfik
categories:
- Linux
date: "2016-10-25T19:30:23Z"
date_gmt: 2016-10-25 17:30:23 +0200
published: true
status: publish
tags:
- pendrive
- luks
- hdd
- ssd
- szyfrowanie
title: Jak ukryć zaszyfrowany kontener LUKS pod linux
---

Gdy w grę wchodzi poufność informacji, to przeciętny użytkownik komputera od razu zaczyna rozważać
szyfrowanie danych. Są różne narzędzia, które te kwestię realizują w mniejszym lub większym stopniu.
Kiedyś wszyscy korzystali z [TrueCrypt](http://truecrypt.sourceforge.net/) ale po jego dziwnych
przygodach ludzie stopniowo zaczęli od tego oprogramowania odchodzić. W jego miejscu zaczęły
pojawiać się różne forki, np. [VeraCrypt](https://veracrypt.codeplex.com/). Abstrahując od tych ww.
narzędzi, w każdym linux'ie mamy również dostępny [mechanizm szyfrujący na bazie
LUKS](https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions) i jego gołą wersję
wykorzystującą dm-crypt. Przy pomocy każdego z tych narzędzi jesteśmy w stanie zaszyfrować dysk
komputera, pendrive, czy nawet kartę SD, w taki sposób, by nikt inny nie uzyskał dostępu do danych
zgromadzonych na tych nośnikach informacji. Problem w tym, że w dalszym ciągu ktoś może nas
torturować, by wydobyć od nas hasło czy keyfile i uzyskać dostęp do tych zaszyfrowanych danych bez
większego trudu. Dlatego też pojawiło się coś nazwanego [Plausible
Deniability](https://en.wikipedia.org/wiki/Plausible_deniability), gdzie wykorzystywane są tak
naprawdę dwa nośniki z czego jeden robi za przykrywkę, a na drugim mamy zgromadzone poufne pliki. W
ten sposób agresorowi podajemy hasło do trefnego kontenera i wszyscy są zadowoleni. Czy aby na
pewno?

<!--more-->
## Sztuka ukrycia zaszyfrowanego kontenera

Generalnie rzecz biorąc, ukrycie zaszyfrowanego kontenera zostało rozpowszechnione za sprawą
TrueCrypt, gdzie cały proces był automatyczny i banalnie prosty. Gdy chodzi o inne narzędzia, to nie
wszystkie one posiadają taki ficzer. W przypadku LUKS wszyscy są niemal zgodni, że nagłówek
zaszyfrowanej partycji czy dysku nie nadaje się wręcz do operacji mających na celu ukrycie jakiegoś
voluminu, tak jak to robi wspomniany TrueCrypt.

Jeśli spróbowalibyśmy zaszyć taki ukryty kontener LUKS w innym zaszyfrowanym kontenerze czy nawet na
gołej partycji, to jest tylko kwestią czasu, gdy ten nagłówek zostanie odnaleziony. W jaki sposób?
Skoro system operacyjny jest w stanie rozpoznać zaszyfrowany nagłówek LUKS, to raczej nie jest on
jakoś specjalnie ukryty. Jest więc niemal pewne, że są już narzędzia, które przeszukają binarnie
dysk czy pendrive w poszukiwaniu pewnych sekwencji bitów charakterystycznych dla tego nagłówka. Po
jego znalezieniu raczej będziemy mieć problem z wytłumaczeniem, co on tam robi. Zwłaszcza w
przypadku, gdy coś się nie zgadza w ilości zajmowanego miejsca.

Swego czasu, gdy korzystałem z TrueCrypt i testowałem te ukryte kontenery, to byłem w stanie zapisać
tylko pewną określoną ilość miejsca na danym nośniku. Przykładowo, dysk miał 100 GiB. Podzieliłem go
w równych proporcjach, tak by 50 GiB przypadło przykrywce, a drugie 50 GiB było ukryte. Każdy z tych
kontenerów miał inny klucz szyfrujący oraz inne hasło służące do odszyfrowania tego klucza. Jeśliby
ktoś na nas naciskał, to możemy mu podać hasło od przykrywki. Tam z kolei była informacja o
maksymalnym rozmiarze partycji, gdzie widać było 100 GiB.

Niemniej jednak, próbując zapisać te 100 GiB, w pewnym momencie zapis był przerywany. Działo się to
po zapisie 50 GiB, no bo przecie na pozostałych 50 GiB znajduje się drugi volumin. Wiedząc to,
automatycznie też wiemy, że istnieje także drugi dysk i możemy dalej torturować człowieka, by
wyjawił również i drugie hasło.

Podobnie sprawa ma się w przypadku LUKS. Takie schowane kontenery nie dają nam praktycznie żadnej
ochrony, tylko mogą nam przysporzyć jeszcze więcej problemów. Czy nie ma zatem żadnego sposobu, by
jakoś ukryć te zasoby? Ależ są, tylko trzeba nieco się wysilić i nie korzystać z rozwiązań OOTB,
które działają po przyciśnięciu jednego przycisku.

## Jak ukryć pliki wewnątrz kontenera LUKS

Ukrycie danych na zasadzie podzielenia nośnika na część widoczną i ukrytą jest bardzo ciekawym
pomysłem. Niemniej jednak, jego implementacja zwykle nie jest najlepsza. Chodzi o to, że do tej
pory chowanie zasobów odbywa się horyzontalnie, przez co łatwo jest te dane odnaleźć. Może zamiast
bawić się w tryb horyzontalny powinniśmy skupić się bardziej na trybie wertykalnym? Problem z tym
drugim rozwiązaniem jest taki, że istnieje bardzo wysokie ryzyko utraty danych ale tylko w przypadku
nieumiejętnego obchodzenia się z takim nośnikiem.

Pliki przechowywane na dyskach twardych, pendrive, karach SD (czy na czymkolwiek co może nam
posłużyć za jakiś nośnik informacji) wymagają systemu plików. Standardowo na dysku oznaczamy
partycje i formatujemy je z wykorzystaniem takiego systemu plików. Każdy wie, że ponowne
sformatowanie tej samej partycji wiążę się z utratą danych. Ale czy aby na pewno musimy formatować
całą partycję? A co jeśli sformatujemy jedynie jej kawałek? Czy to w ogóle możliwe, no bo formatować
można tylko całą partycję, a nie jej kawałek? Przecie nie ma takich opcji w `fdisk` , `gdisk` ,
`parted` , `gparted` czy `mkfs` . No właśnie, a kto powiedział, że zamierzamy zdefiniować partycję w
taki sposób jak to rozumieją te powyższe narzędzia?

Patrycją może być dowolny kawałek przestrzeni na nośniku informacyjnym. Ta przestrzeń musi być na
tyle duża by zmieściła się w niej struktura opisująca wykorzystywany system plików na tej partycji.
Taka przestrzeń może też znajdować się w dowolnym miejscu tego nośnika. Rozmiary partycji mogą mieć
kilka MiB albo i wiele GiB, zatem daje nam to niesamowitą dowolność w tworzeniu partycji.

Myśląc nad rozwiązaniem ukrytego kontenera wpadłem na pomysł stworzenia dwóch niezależnych od siebie
zaszyfrowanych kontenerów LUKS, z których jeden jest zaszyty wewnątrz drugiego. W ten sposób,
standardowo pod odszyfrowaniu takiego zagnieżdżonego kontenera, dana osoba będzie mieć dostęp, tylko
do pierwszego systemu plików, który z kolei rozciągałby się na całą partycję. Nawet jeśli ta osoba
podejrzewałaby możliwość ukrycia czegoś wewnątrz zaszyfrowanego kontenera, to nie ma jak tego
sprawdzić. Mogłaby, gdyby ten ukryty kontener zawierał nagłówek.

Ten mechanizm jest o tyle ciekawy, że mając do dyspozycji 100 GiB, możemy ukryć tak 99 GiB
zasłaniając je 1 GiB. Jak niby można zasłonić w ten sposób tak dużą część nośnika? Powiem więcej,
nie tyle można zasłonić te 99 GiB ale także nikt nie będzie w stanie ustalić, że oszukujemy.

## Ukryty kontener LUKS

Jak przygotować nośnik zawierający nałożone na siebie dwa kontenery LUKS? Przede wszystkim, musimy
podzielić dysk/partycję/pendrive/kartę SD na dwa obszary. Rozmiary mogą być dowolne. Trzeba jednak
pamiętać, że im więcej przestrzeni zostawimy na widoczne pliki, tym większa szansa powodzenia całego
przedsięwzięcia.

Pamiętajmy tylko, że ta zabawa stwarza bardzo poważne zagrożenie dla danych ukrytych i bez
właściwego obchodzenia się z tym nośnikiem, dane na nim zgromadzone mogą ulec skasowaniu. Lepiej
zastanówmy się czy chcemy się w to bawić. Dla przykładu, jeśli ktoś zacznie wgrywać dane na widoczny
zasób, to w pewnym momencie zacznie zapisywać dane na ukrytym systemie plików niszcząc go przy tym
kompletnie. W efekcie nasz przykładowy kontener 100 GiB zostałby zapisany w pełni jak gdyby nigdy
nic, no tyle, że przy okazji została by przepisana nasza tajna konwersacja ze Snowden'em.

W tym doświadczeniu wykorzystamy sobie kartę SD 2 GiB. Nie jest ona duża ale to nie ma większego
znaczenia. Zasada jest zawsze taka sama. Ważne jest, by oba zasoby były jednolite, tj. miały ten sam
system plików, katalog w którym zamierzamy je montować, a nawet nazwę odszyfrowanego zasobu. Ma to
na celu ukrycie faktu posiadania ukrytego kontenera, np. w logach aplikacji, które mają zamiar
korzystać z takiego zasobu. Niemniej jednak, nie zawsze wszystkie informacje da radę zataić i
niektóre mogą powędrować do logu systemowego, np. różnica w pojemności voluminów.

### Czyszczenie nośnika danymi losowymi

Trzeba także pamiętać, by zapisać cały nośnik/partycję losowymi danymi przed próbą ukrycia w nim
jakiegoś zasobu. Jeśli tego nie zrobimy, to analiza takiego dysku wykaże, w którym miejscu coś
próbujemy ukryć.

    # dd if=/dev/urandom of=/dev/sdb bs=2M

### Tworzenie szyfrowanego kontenera LUKS

Teraz w przy pomocy `gparted` stwórzmy na nim jednolitą partycję `sdb1` :

![](/img/2016/10/1.gparted-linux-tworzenie-partycji-pod-luks.png#huge)

Tutaj wykorzystujemy system plików EXT4. Poniżej widzimy, że struktura opisowa tej partycji zajmuje
około 70 MiB. To jest minimum, poniżej którego nie możemy zejść.

![](/img/2016/10/2.gparted-linux-tworzenie-partycji-pod-luks.png#huge)

Do tego trzeba będzie przeznaczyć jeszcze jakieś miejsce na widoczne pliki. Rozmiar tej karty SD
widziany w systemie to 1,84 GiB, a nasza przykrywka będzie miała 256 MiB. Pozostała część obszaru
dysku będzie przykryta. Stwórzmy zatem na całej dostępnej przestrzeni zaszyfrowany kontener. Hasło
nie musi być długie i skomplikowane, w końcu będziemy musieli je podać w razie torturowania nas i
lepiej go nie zapomnieć w takiej sytuacji.

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase --verbose luksFormat /dev/sdb1

Otwórzmy teraz ten kontener:

    # cryptsetup luksOpen /dev/sdb1 sdb1

Póki co, nie mamy jeszcze nic w kontenerze. To jest tylko wirtualny obszar, który zamierzamy sobie
podzielić. Wewnątrz tego kontenera musimy teraz stworzyć przykrywkowy system plików. Pamiętajmy
jednak, że z jednej strony on musi wypełniać całą dostępną przestrzeń zaszyfrowanego kontenera ale
pliki muszą zostać zapisane na początku systemu plików (<256 MiB). By tego typu sztuczkę
zastosować, musimy nieco inaczej podejść do kwestii tworzenia systemu plików.

Na samym początku stworzymy system plików dla całego kontenera. Później go skurczymy do 256 MiB i
zapiszemy w nim jakieś mało znaczące dla nas pliki. Następnie ten system plików zostanie rozszerzony
znów tak, by wypełnił cały kontener. Kolejnym krokiem będzie stworzenie ukrytego kontenera. Będzie
on się znajdował gdzieś na granicy pierwszych 320 MiB. My będziemy wykorzystywać jedynie ukryty
system plików, natomiast każdy inny niewtajemniczony osobnik będzie próbował zamontować przykrywkowy
system plików po odszyfrowaniu kontenera. Mniej więcej tak jak to zrobi standardowo każdy system.
Taka osoba zobaczy dane, które powinna i nie będzie miała żadnej możliwości zweryfikowania czy coś z
tym system plików jest nie tak.

Test zapisu również nic nie wykaże. Natomiast przy próbie zapisu nastąpi ciche zniszczenie ukrytego
kontenera i jego systemu plików. Może i stracimy poufne dla nas informacje ale nikt się o tym nawet
nie dowie. Zatem do dzieła, tworzymy system plików w kontenerze głównym:

    # mkfs.ext4 -m 0 -L sd-ext4 /dev/mapper/sdb1

Tak stworzony system plików zawiera journal, czyli dziennik, w którym są zapisywane informacje na
temat operacji na plikach. Można go usunąć lub zostawić. Jeśli zostawiamy, to trzeba się liczyć, że
on trochę zajmuje. W przypadku małych systemów plików (tak jak tutaj), jest to 4 MiB. W przypadku
większych zasobów może być nawet 128 MiB i więcej. Ten dziennik jednak nie wpływa w żadnym stopniu
na ukryty system plików. Mówi nam jednak nieco o wykorzystaniu przykrywkowego systemu plików.

### Umieszczanie plików w określonym obszarze kontenera

Mając stworzony system plików, musimy go teraz skurczyć do rozmiarów 256 MiB. Możemy to zrobić przy
pomocy `resize2fs` .

    # resize2fs -p /dev/mapper/sdb1 256M
    resize2fs 1.43.3 (04-Sep-2016)
    Resizing the filesystem on /dev/mapper/sdb1 to 65536 (4k) blocks.
    Begin pass 2 (max = 8192)
    Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Begin pass 3 (max = 15)
    Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    The filesystem on /dev/mapper/sdb1 is now 65536 (4k) blocks long.

Montujemy teraz ten system plików:

    # mount /dev/mapper/sdb1 /mnt

    # df -h | grep -i mnt
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    /dev/mapper/sdb1               ext4      221M  1.9M  214M   1% /mnt

Jak widzimy nasz kontener ma do dyspozycji 214 MiB. Zapiszmy w nim trochę mało poufnych dla nas
danych, które mają być "słabo szyfrowane" (jak z tej reklamy dysków twardych pewnej firmy).

    # df -h | grep mnt
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    /dev/mapper/sdb1               ext4      221M  103M  113M  48% /mnt

Dane są na swoim miejscu. Teraz odmontujmy ten system plików i powiększmy go tak, by wypełnił cały
kontener. Pliki zostaną na swoim miejscu.

    # umount /mnt
    # e2fsck -f /dev/mapper/sdb1
    # resize2fs -p /dev/mapper/sdb1
    resize2fs 1.43.3 (04-Sep-2016)
    Resizing the filesystem on /dev/mapper/sdb1 to 481024 (4k) blocks.
    The filesystem on /dev/mapper/sdb1 is now 481024 (4k) blocks long.

### Tworzenie ukrytego kontenera LUKS

Mając przygotowany przykrywkowy system plików, możemy przejść do tworzenia ukrytego kontenera LUKS.
Pamiętajmy, że dane mamy wgrane gdzieś w obrębie pierwszych 256 MiB. Zatem zaczniemy tworzyć nowy
system plików gdzieś koło 320 MiB. Na sam początek zamknijmy główny kontener jeśli jeszcze jest on
otwarty:

    # cryptsetup luksClose sdb1

I teraz cała sztuka polega na tym, by ustalić odpowiedni offset dla ukrytego kontenera. Ten z kolei
podajemy w blokach 512 bajtowych. Zatem będzie to 320x1024x1024/512=655360. I to tę wartość trzeba
podać w `--align-payload` narzędziu `cryptsetup` .

Dodatkowo w celu zwiększenia poufności nowego kontenera możemy wykroić nagłówek i umieścić go na
osobnym nośniku przy pomocy opcji `--header` . Jest to o tyle użyteczna opcja, że nie pozostawia
śladu obecności ukrytego kontenera. Każdy, kto by analizował strukturę takiego nośnika, zobaczyłby
jedynie losowe dane, co jeszcze bardziej by go upewniło, że nic nie skrywamy na naszym zaszyfrowanym
dysku czy pendrive.

    # dd if=/dev/zero of=/home/morfik/Desktop/header-sd-ext4 bs=1M count=2

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase --verbose luksFormat /dev/sdb1 --align-payload 655360 --header /home/morfik/Desktop/header-sd-ext4

Otwieramy nasz ukryty kontener i tworzymy na nim system plików:

    # cryptsetup luksOpen --header /home/morfik/Desktop/header-sd-ext4 /dev/sdb1 sdb1
    # mkfs.ext4 -m 0 -L sd-ext4 /dev/mapper/sdb1

Montujemy teraz ten ukryty system plików:

    # mount /dev/mapper/sdb1 /mnt
    # df -h | grep mnt
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    /dev/mapper/sdb1               ext4      1.5G  4.6M  1.5G   1% /mnt

Jak widać, mamy do dyspozycji 1,5 GiB z całkowitych 1,9 GiB. Reszta robi za przykrywkę. Odmontujmy
zatem nasz ukryty volumin i sprawdźmy czy nic nie dolega przykrywce:

    # umount /mnt
    # cryptsetup luksClose sdb1
    # cryptsetup luksOpen /dev/sdb1 sdb1
    # mount -r /dev/mapper/sdb1 /mnt

    # df -h | grep mnt
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    /dev/mapper/sdb1               ext4      1.8G  106M  1.7G   6% /mnt

A tu już mamy widoczny zaszyfrowany kontener z paroma plikami wgranymi wcześniej no i oczywiście
jego system plików rozciąga się na całym nośniku bez jakiegokolwiek znaku, że na granicy 320 MiB
znajduje się ukryty kontener.

Ważna uwaga tutaj, jeśli zamierzamy wchodzić w interakcję z przykrywkowym systemem plików, to zawsze
montujmy go w trybie do odczytu. Ewentualna inspekcja tego nośnika nic nie wykaże, bo nagłówek jest
na osobnym nośniku. A co z próbą zapisu danych w celu ustalenia faktycznej pojemności systemu
plików? Cóż, sprawdźmy. Zapiszmy ten system plików danymi z `/dev/zero` :

    # dd if=/dev/zero of=/mnt/zero.img bs=2M
    dd: error writing '/mnt/zero.img': No space left on device
    848+0 records in
    847+0 records out
    1777975296 bytes (1.8 GB, 1.7 GiB) copied, 311.186 s, 5.7 MB/s

    # df -h
    Filesystem                     Type      Size  Used Avail Use% Mounted on
    /dev/mapper/sdb1               ext4      1.8G  1.8G     0 100% /mnt

Oczywiście dane wgrane na samym początku do tego przykrywkowego systemu plików pozostały nietknięte
ale nasz ukryty kontener zwyczajnie wyparował. Teraz już nikt nie wyciągnie z niego żadnych danych,
nawet i my. Możemy się o tym przekonać na własne oczy:

    # umount /mnt
    # cryptsetup luksClose sdb1
    # cryptsetup luksOpen --header /home/morfik/Desktop/header-sd-ext4 /dev/sdb1 sdb1
    # mount /dev/mapper/sdb1 /mnt
    mount: /dev/mapper/sdb1 is write-protected, mounting read-only
    mount: wrong fs type, bad option, bad superblock on /dev/mapper/sdb1,
           missing codepage or helper program, or other error

           In some cases useful info is found in syslog - try
           dmesg | tail or so.

Może i system poprawnie przetworzył nagłówek LUKS, który znajduje się u mnie na dysku, i stosownie
oznaczy ukryty obszar karty SD ale próba zamontowania tego voluminu zwraca błąd. Pozostała nam zatem
jedynie zaszyfrowana przykrywka, która nie wzbudzała i nadal nie wzbudza żadnych podejrzeń.

## Ukryty kontener przy wykorzystaniu dm-crypt

LUKS ma ten problem, że na partycji zostawia ślad w postaci nagłówka. Ten nagłówek jest zwykle
niezbędny jeśli potrzebujemy menadżera kluczy, który jest oferowany prze LUKS. Zwykle jednak
korzystamy tylko z jednego hasła i cała funkcjonalność LUKS jest dla nas zwyczajnie zbędna. Dlatego
też zamiast tworzyć ukryty kontener LUKS, możemy wykorzystać do tego celu `dm-crypt` . Nie zostawia
on żadnych śladów na partycji i idealnie nadaje się do zastosowania w naszym przypadku. Do
zmapowania interesującego nas obszaru dysku wykorzystamy poniższe polecenie:

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --verify-passphrase --verbose open --type=plain /dev/sdb1 sdb1 --offset=655360

Tę linijkę trzeba będzie wpisywać za każdym razem odszyfrowując ukryty kontener, jako, że na
partycji nie ma nagłówka, który by przechowywał choćby informacje na temat rodzaju i długości
klucza. Może nie jest to wygodne ale za to nie musimy ze sobą targać nagłówka LUKS na osobnym
nośniku.

W tak zmapowanym obszarze, musimy jeszcze utworzyć system plików:

    # mkfs.ext4 -m 0 -L sd-ext4 /dev/mapper/sdb1

No i teraz już możemy zamontować nasz ukryty kontener w systemie:

    # mount /dev/mapper/sdb1 /mnt

Taki kontener zaś zamykamy standardowo:

    # umount /mnt
    # cryptsetup luksClose sdb1
