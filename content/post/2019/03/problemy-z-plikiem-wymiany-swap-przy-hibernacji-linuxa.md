---
author: Morfik
categories:
- Linux
date: "2019-03-23T19:30:22Z"
published: true
status: publish
tags:
- debian
- hibernacja
GHissueID: 295
title: Problemy z plikiem wymiany SWAP przy hibernacji linux'a
---

Po wyjaśnieniu paru rzeczy w kwestii przestrzeni wymiany, a
konkretnie [co jest lepsze: plik SWAP czy osobna partycja]({{ site.baseurl  }}/post/czy-w-linux-plik-swap-jest-lepszy-niz-partycja-wymiany/),
postanowiłem nieco zmienić układ partycji na dysku mojego laptopa (LUKS+LVM) i zaimplementować
sobie przestrzeń wymiany w postaci pliku SWAP. Standardowo jednak plik SWAP nie działa w przypadku
hibernacji linux'a i kernel ma problemy z wybudzeniem maszyny z głębokiego snu. Jeśli zamierzamy
korzystać z hibernacji, to trzeba inaczej skonfigurować system, by nauczyć go korzystać z
przestrzeni wymiany w postaci pliku. W tym artykule rzucimy sobie okiem na ten niezbyt
skomplikowany proces.

<!--more-->
## Przygotowanie partycji / pod plik SWAP

W zależności od tego czy startujemy ze świeżym systemem plików czy mamy aktualnie już wgrany
system operacyjny, trzeba nieco inaczej podejść do zadania tworzenia przestrzeni wymiany. Może i
fragmentacja pliku SWAP aż tak nie wpłynie na wydajność systemu ale części tego dużego pliku lepiej
jest mieć jak najbliżej siebie.

Plik SWAP będziemy tworzyć na partycji `/` i najlepiej jest zacząć od sprawdzenia jak prezentuje
się aktualnie fragmentacja wolnej przestrzeni tego systemu plików w `e2freefrag` . U mnie wygląda
ona następująco:

    # e2freefrag /dev/mapper/wd_black_label-root
    Device: /dev/mapper/wd_black_label-root
    Blocksize: 4096 bytes
    Total blocks: 7864320
    Free blocks: 4175483 (53.1%)

    Min. free extent: 4 KB
    Max. free extent: 85632 KB
    Avg. free extent: 596 KB
    Num. free extent: 27934

    HISTOGRAM OF FREE EXTENT SIZES:
    Extent Size Range :  Free extents   Free Blocks  Percent
       4K...    8K-  :          5767          5767    0.14%
       8K...   16K-  :          3749          8890    0.21%
      16K...   32K-  :          2560         13146    0.31%
      32K...   64K-  :          4138         49365    1.18%
      64K...  128K-  :          3109         73707    1.77%
     128K...  256K-  :          1435         66024    1.58%
     256K...  512K-  :          1446        131012    3.14%
     512K... 1024K-  :          1492        269814    6.46%
       1M...    2M-  :          1934        703334   16.84%
       2M...    4M-  :          1409        968281   23.19%
       4M...    8M-  :           617        843599   20.20%
       8M...   16M-  :           200        524080   12.55%
      16M...   32M-  :            61        321823    7.71%
      32M...   64M-  :            15        158138    3.79%
      64M...  128M-  :             2         38503    0.92%

W tym przypadku utworzenie pliku o rozmiarze 4 GiB skutkowało jego dość mocną fragmentacją, bo
zostało utworzonych ponad 800 kawałków rozrzuconych na obszarze 30 GiB. No jednak to trochę dużo.
Nie było innej opcji jak przeniesienie zawartość partycji `/` na inny nośnik i dopiero po takim
opróżnieniu systemu plików można było stworzyć w nim plik SWAP, którego struktura była w miarę
ciągła. Pliki systemu oczywiście trzeba było przerzucić z powrotem.

Jeśli na partycji `/` mamy kilka większych wolnych zakresów bloków, to możemy spróbować utworzyć
na niej plik wymiany bez zawracania sobie głowy fragmentacją. Nie ma co przesadzać też z wielkością
pliku SWAP. Tak 4-6 GiB powinno nam w zupełności wystarczyć. Lepiej jest jednak też za w czasu
przemyśleć rozmiar tego pliku, tak by nie zrobić go za małego. Oczywiście w późniejszym czasie
będzie można ten plik powiększyć lub też dorobić drugi.

## Tworzenie pliku wymiany

Plik SWAP możemy utworzyć przy pomocy `dd` lub też zaprzęgnąć do pracy `fallocate` . To drugie
rozwiązanie praktycznie od razu nam stworzy nawet i bardzo dużych rozmiarów pliki, co może nam
zaoszczędzić trochę czasu w przypadku, gdy go nie mamy za dużo. Poniżej przykłady zastosowania obu
narzędzi:

    # fallocate -l 4g /swapfile

    # dd if=/dev/zero of=/swapfile bs=16M count=256

Czasami tworząc duże pliki, i to nawet w zupełnie pustym systemie plików, może dojść do ich
większej lub mniejsze fragmentacji. Poniżej przykład:

    # filefrag /swapfile
    Filesystem type is: ef53
    File size of /swapfile is 4294967296 (1048576 blocks of 4096 bytes)
    /swapfile: 72 extents found

No niby pusty system plików, a i tak plik o rozmiarze 4G został pocięty na 72 kawałki. Jak ten ext4
tworzy te pliki, to chyba tylko on wie. Można oczywiście ten plik zdefragmentować chwilę po jego
utworzeniu i odzyskać sporo zakresów:

    # e4defrag -v /swapfile

W takim pliku musimy utworzyć przestrzeń wymiany za pomocą `mkswap` :

    # chmod 600 /swapfile
    # mkswap /swapfile
    Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
    no label, UUID=d40e356e-dec4-4f23-97f9-40963128fe20

Ten `UUID` widoczny wyżej do niczego nam się nie przyda i można na niego w ogóle nie zwracać uwagi.

Dodajmy jeszcze stosowny wpis do pliku `/etc/fstab` , komentując przy tym linijkę zawierającą
konfigurację dla partycji wymiany:

    #UUID=7770d2c3-0542-43b9-9f3e-dcadc6f25004	swap		swap	defaults,pri=10 0 0
    /swapfile									swap		swap	defaults 0 0

Jeśli wszystko poszło dobrze, to powinniśmy być w stanie aktywować przestrzeń wymiany przy pomocy
`swapon` :

    # swapon -a

    # swapon --show
    NAME      TYPE SIZE  USED PRIO
    /swapfile file   4G 14.3M   -2

Działający plik SWAP to nie koniec zabawy. Musimy jeszcze nauczyć system jak odczytać ten plik
podczas wybudzania maszyny ze stanu hibernacji.

## Dlaczego linux ma problemy z odhibernowaniem się z pliku SWAP

By linux był w stanie wybudzić system ze stanu
hibernacji [musi znaleźć i załadować obraz pamięci](https://www.kernel.org/doc/Documentation/power/swsusp-and-swap-files.txt),
który został stworzony przed wyłączeniem komputera. Standardowo taki obraz jest umieszczany na
partycji wymiany. Położenie obrazu pamięci było zawsze znane, bo znajdował się on zaraz na początku
partycji. W takiej sytuacji wystarczyło podać urządzenie/partycje, z której kernel linux'a ma
odczytać obraz pamięci i to wszystko czego od nas wymagano przy konfiguracji systemu pod kątem
hibernacji.

W przypadku pliku SWAP sprawa wygląda nieco bardziej skomplikowanie, bo plik umieszczony w systemie
plików może znajdować się praktycznie w dowolnym miejscu partycji. Linux zatem nie wie, w którym
miejscu ten plik SWAP figuruje, a jeśli nie zna jego lokalizacji, to logiczne jest, że ma on
problemy z załadowaniem obrazu pamięci. Mimo wszystko zahibernować system się da bez problemu ale
podczas startu komputera, kernel uzna, że obrazu pamięci nie ma i zainicjuje standardową procedurę
startu.

By poradzić sobie z tym problemem, musimy odszukać offset pliku SWAP i podać go kernelowi.

### Jak ustalić offset pliku SWAP na potrzeby hibernacji

Są w zasadzie dwa sposoby na skonfigurowanie hibernacji w pliku SWAP. Pierwszy z nich zakłada
przeprowadzenie wszystkich operacji ręcznie. W drugim zaś można zaprzęgnąć do pracy narzędzia z
pakietu `uswsusp` . My tutaj skupimy się na tym pierwszym rozwiązaniu.

Offset pliku SWAP, którego poszukujemy, musi być podany względem systemu plików, w obrębie którego
ten plik się znajduje. Możemy skorzystać z kilku narzędzi, np. `debugfs` , czy też widoczny wyżej
`filefrag` . Rzućmy okiem na wyjście `filefrag`'a :

    # filefrag -ve /swapfile
    Filesystem type is: ef53
    File size of /swapfile is 4294967296 (1048576 blocks of 4096 bytes)
     ext:     logical_offset:        physical_offset: length:   expected: flags:
       0:        0..   32767:   58396672..  58429439:  32768:
    ...

Interesuje nas tutaj jedynie zakres `0` . W kolumnie `physical_offset` mamy zaś numerek wskazujący
na `58396672` i to jest nasz offset, który musimy podać kernelowi.

### Plik /etc/initramfs-tools/conf.d/resume

W dystrybucji Debian, do konfiguracji hibernacji używa się pliku
`/etc/initramfs-tools/conf.d/resume` . Nie możemy jednak podać w nim wyżej odnalezionego offsetu.
Przynajmniej mi się tego nie udało zrobić. W tym pliku znajduje się zmienna `$RESUME` , która
przyjmuje wartość ścieżki do partycji wymiany. My z tej partycji wymiany korzystać nie będziemy,
zatem przydałoby się wykomentować ten wpis:

    #RESUME=/dev/mapper/wd_black_label-swap

### Parametry kernela resume= oraz resume_offset=

Ostatnim krokiem konfiguracji hibernacji linux'a jest dodanie dwóch parametrów, tj. `resume=` oraz
`resume_offset=` bezpośrednio w linijce kernela w konfiguracji bootloader'a, przykładowo

    APPEND root=/dev/mapper/wd_black_label-root ... resume=/dev/mapper/wd_black_label-root resume_offset=58396672 ...

Parametr `resume=` może zawierać ścieżkę do urządzenia lub UUID partycji, na której znajduje się
plik wymiany. Natomiast parametr `resume_offset=` określa offset pliku SWAP uzyskany wcześniej.

### Aktualizacja obrazu initramfs/initrd

Po dokonaniu wszystkich tych wyżej wymienionych zmian musimy na nowo wygenerować obraz
initramfs/initrd :

    # update-initramfs -u -k all

No i to w zasadzie wszystkie kroki, które trzeba poczynić, by skonfigurować w systemie plik SWAP na
 potrzeby odhibernowania linux'a.

## Zastosowanie uswsusp

Dla mnie ten wyżej opisany sposób konfiguracji pliku SWAP na potrzeby hibernacji jest bardzo
przejrzysty i prosty do implementacji. Niemniej jednak, można też skorzystać z narzędzi zawartych w
pakiecie `uswsusp` . Opis jak tego dokonać można znaleźć
np. [tutaj](https://askubuntu.com/questions/6769/hibernate-and-resume-from-a-swap-file) ale patrząc
po jego stopniu skomplikowania i krokach, które trzeba poczynić, wydaje mi się, że ten manualny
sposób przedstawiony w niniejszym artykule jest nieco lepszym rozwiązaniem, no i nie potrzebne nam
jest w zasadzie żadne dodatkowe oprogramowanie.

## Pełne szyfrowanie dysku, a SWAP w pliku

Wydawać się może, że w przypadku wykorzystywania pełnego szyfrowania dysku, taka opcja trzymania
przestrzeni wymiany w pliku dyskwalifikuje nas i możemy o niej zapomnieć. Nic bardziej mylnego, bo
plik SWAP działa dobrze nawet na LVM osadzonym w kontenerze LUKS i system się bez problemu
hibernuje i wybudza, tak jak to ma miejsce w przypadku zwykłej partycji wymiany.

Co ciekawe, jeśli ktoś nie korzysta z pełnego szyfrowania dysku, to mógłby się bardziej
zainteresować tym pakietem `uswsusp` , bo jest on w stanie zaszyfrować samą przestrzeń wymiany i
później przy podnoszeniu systemu trzeba będzie podać hasło, by snapshot pamięci RAM z pliku SWAP
wydobyć. Oczywiście to tylko tak na marginesie, bo ja jednak wolę FDE i mieć wszystko pod hasłem.
