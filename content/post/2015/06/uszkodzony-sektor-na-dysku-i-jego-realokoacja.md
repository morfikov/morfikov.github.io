---
author: Morfik
categories:
- Linux
date: "2015-06-15T19:18:01Z"
date_gmt: 2015-06-15 17:18:01 +0200
published: true
status: publish
tags:
- smart
- hdd
- ssd
title: Uszkodzony sektor na dysku i jego realokoacja
---

Uszkodzone sektory w przypadku dysków HDD, to jak sama nazwa wskazuje, sektory, które z jakichś
przyczyn nie działają tak jak powinny. Doprowadza to z reguły do niestabilności systemu operacyjnego
objawiającej się jego zawieszaniem w momencie próby odczytu danych z takiego padniętego sektora.
Przyczyny mogą być różne. Zwykle jest to jednak fizyczne uszkodzenie powierzchni nośnika, np w
wyniku wstrząsu, czy też zwyczajne zmęczenie materiału. Jest wielce prawdopodobne, że nie jesteśmy w
stanie nic poradzić w tego typu sytuacji, a ci bardziej zaawansowani użytkownicy zalecają jak
najszybszą wymianę dysku, bo pierwszy bad sektor oznacza, że niedługo będzie ich więcej. Czasem
jednak błędy odczytu mogą być programowe, tj. fizycznie każdy sektor jest w porządku ale z jakiegoś
powodu system nie potrafi odczytać z nich danych. Przy odrobinie szczęścia jesteśmy w stanie
odblokować taki sektor.

<!--more-->
## Pierwszy uszkodzony sektor

Po 18.798 godzinach pracy, mój główny dysk złapał prawdopodobnie bad sektor. W logu
[S.M.A.R.T](https://pl.wikipedia.org/wiki/S.M.A.R.T._%28informatyka%29) można przeczytać info o
takim błędzie:

    Error 25 occurred at disk power-on lifetime: 18798 hours (783 days + 6 hours)
      When the command that caused the error occurred, the device was active or idle.

      After command completion occurred, registers were:
      ER ST SC SN CL CH DH
      -- -- -- -- -- -- --
      40 51 08 00 40 37 e6  Error: UNC 8 sectors at LBA = 0x06374000 = 104284160

      Commands leading to the command that caused the error were:
      CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
      -- -- -- -- -- -- -- --  ----------------  --------------------
      c8 00 08 00 40 37 e6 08      08:54:35.771  READ DMA
      ec 00 00 00 00 00 a0 08      08:54:35.763  IDENTIFY DEVICE
      ef 03 46 00 00 00 a0 08      08:54:35.763  SET FEATURES [Set transfer mode]

Problemy z sektorami można zauważyć również w `/var/log/messages` . Poniżej linijka, która pokaże,
czy są jakieś wzmianki o nich:

    # grep LBA /var/log/messages* | awk '{print $9}' | sort | uniq
    156299375
    2930275055
    976817134

No tak 3 kolejne i żaden z nich nie odpowiada temu, co siedzi w raporcie S.M.A.R.T . To
niekoniecznie muszą być bad sektory ale każdy wymieniony sektor trzeba sprawdzić pod kątem odczytu,
być może były tam jakieś problemy wcześniej i temu system to zanotował ale niekoniecznie oznacza to,
że problemy dalej występują. Najlepiej po prostu odpytać te sektory i jeśli dadzą się czytać,
wszystko jest w porządku i można o nich zapomnieć. No to do dzieła:

    # hdparm --read-sector 156299375 /dev/sda

    /dev/sda:
    reading sector 156299375: succeeded

    # hdparm --read-sector 2930275055 /dev/sda

    /dev/sda:
    reading sector 2930275055: succeeded

    # hdparm --read-sector 976817134 /dev/sda

    /dev/sda:
    reading sector 976817134: succeeded

Oczywiście, tych sektorów w logu może być więcej, ja mam w miarę świeży system i logów jako takich
za dużo w nim jeszcze się nie zdążyło nazbierać. W każdym razie, wszystkie sektory z logu zostały
odczytane pomyślnie.

Jeśli pojawiają się błędy w liczbie sektorów `8` (tak jak w tym przypadku), oznacza to
prawdopodobnie, że dysk używa technologii
[advanced](https://www.ibm.com/developerworks/linux/library/l-4kb-sector-disks/index.html)
[format](https://en.wikipedia.org/wiki/Advanced_Format), czyli ma sektory nie 512 bajtów, a
8\*512=4096 bajtów. Czasami dysk może zwracać wartość 512 bajtów w każdym możliwym miejscu, pomimo
faktu, że ma sektory 4096 bajtowe.

W syslogu, przy próbie odczytu uszkodzonego sektora, można zobaczyć taki komunikat:

    Nov 21 12:57:29 morfikownia kernel: [ 5503.342060] ata1.01: exception Emask 0x0 SAct 0x0 SErr 0x0 action 0x0
    Nov 21 12:57:29 morfikownia kernel: [ 5503.342069] ata1.01: failed command: READ SECTOR(S)
    Nov 21 12:57:29 morfikownia kernel: [ 5503.342077] ata1.01: cmd 20/00:01:00:40:37/00:00:00:00:00/f6 tag 0 pio 512 in
    Nov 21 12:57:29 morfikownia kernel: [ 5503.342077]          res 51/40:01:00:40:37/40:00:06:00:00/f6 Emask 0x9 (media error)
    Nov 21 12:57:29 morfikownia kernel: [ 5503.342082] ata1.01: status: { DRDY ERR }
    Nov 21 12:57:29 morfikownia kernel: [ 5503.342085] ata1.01: error: { UNC }
    Nov 21 12:57:29 morfikownia kernel: [ 5503.379301] ata1.01: configured for UDMA/133
    Nov 21 12:57:29 morfikownia kernel: [ 5503.379331] ata1: EH complete

Mi on nic nie mówi, no może poza tym, że jest błąd odczytu sektora. Co ciekawe dysk sam nie
realokował go. W tabelce S.M.A.R.T widnieje ciągle jako uszkodzony:

    ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
      5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       0
    196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       0
    197 Current_Pending_Sector  0x0032   200   200   000    Old_age   Always       -       1
    198 Offline_Uncorrectable   0x0030   200   200   000    Old_age   Offline      -       2

Uszkodzony sektor nie jest oznaczany jako nie do użytku automatycznie dlatego, że w tym sektorze
rezyduje jakiś plik, lub część jakiegoś pliku. Sektor może być realokowany tylko przy zapisie danych
na dysk, a nie przy ich odczycie. Przy odczycie dostaje się tylko błęda. W tym wypadku zapis
wadliwego sektora nie może się odbyć, bo miejsce jest zajęte przez plik, dlatego też raport
S.M.A.R.T pokazuje błąd odczytu bez przenoszenia sektora, co wskazuje na uszkodzony plik, który
trzeba zlokalizować. Ok, może nie trzeba, wystarczy pewnie nadpisać ponownie ten sektor siłowo i ma
się spokój, przynajmniej do czasu gdy zajdzie potrzeba skorzystania z tego pliku. Ja postaram się
doszukać nieco więcej informacji na temat tego zdarzenia. Tylko jak ustalić jaki to plik i gdzie te
sektory dokładnie się znajdują i co oznacza `LBA=104284160` w raporcie S.M.A.R.T?

## Plik rezydujący na uszkodzonym sektorze

Zgodnie z tym co piszą na wiki: "LBA (ang. Logical Block Addressing) - metoda obsługi dysku twardego
przez system operacyjny. I aby go wyliczyć, trzeba skorzystać z poniższego wzoru":

    LBA = ( numer_cylindra * liczba_glowic_na_cylinder + numer_glowicy ) * liczba_sektorow_na_sciezke + numer_sektora -1

Hmmm, taa, a nie da się jakoś po ludzku? Da się. Przede wszystkim musimy ustalić, na której partycji
występuje bad sektor. To jest akurat stosunkowo proste. Można skorzystać z narzędzia `fdisk` i
popatrzyć po początkach i końcach partycji i jeśli bad sektor się zawiera w którymś z przedziałów,
oznacza to, że partycja zawierająca uszkodzony sektor została zlokalizowana. Tak wygląda struktura
mojego dysku:

    root:~# fdisk -lu /dev/sda

    Disk /dev/sda: 1500.3 GB, 1500301910016 bytes
    255 heads, 63 sectors/track, 182401 cylinders, total 2930277168 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000a8aae

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     1953791      975872   83  Linux
    /dev/sda2         1953792    99610623    48828416   83  Linux
    /dev/sda3        99610624  1466798079   683593728   83  Linux
    /dev/sda4      1466800126  2930276351   731738113    5  Extended
    /dev/sda5      1857425408  2482423807   312499200   83  Linux
    /dev/sda6      2482425856  2930276351   223925248   83  Linux
    /dev/sda7      1466802176  1646376959    89787392   83  Linux
    /dev/sda8      1646379008  1857423359   105522176   83  Linux

    Partition table entries are not in disk order

Liczba `104284160` (ta z błędu S.M.A.R.T) zawiera się w przedziale `99610624-1466798079` , także bad
sektor znajduje się na 3 partycji. W przypadku posiadania kilku padniętych sektorów, poniższe kroki
trzeba zastosować do każdego z nich. W każdym razie, mając już trefną partycję, nasuwa się pytanie o
dokładne miejsce, którego system nie jest w stanie odczytać. Interesuje nas początek partycji i
lokalizacja sektora. Są to dwie zmienne, które będą nam potrzebne do późniejszego równania, oznaczmy
je sobie `L=104284160` oraz `S=99610624` . Musimy pozyskać jeszcze jedną zmienną -- rozmiar bloku, a
ten z kolei możemy wyciągnąć przez `tune2fs` :

    # tune2fs -l /dev/mapper/crypt_filmy | grep Block
    Block count:              170897920
    Block size:               4096
    Blocks per group:         32768

Oczywiście ja używam zaszyfrowanych partycji temu odwołuję się do urządzeń w `/dev/mapper/` .
Standardowo jednak są to partycje typu `/dev/sda2` , etc. Czyli rozmiar bloku systemu plików to
`4096`. Jest to nasza zmienna `B` . Teraz podstawiamy to wszystko do wzoru:

    b = (int)((L-S)*512/B)
    b = (int)((104284160-99610624)*512/4096
    b = 584192

`(int)` bierze z wyniku liczbę całkowitą. W ten sposób uzyskaliśmy numer wadliwego bloku widzianego
przez system plików na danej partycji. Musimy zbadać ten blok przy pomocy `debugfs` :

    # debugfs
    debugfs 1.42.8 (20-Jun-2013)
    debugfs:  open /dev/mapper/crypt_filmy
    debugfs:  testb 584192
    Block 584192 marked in use
    debugfs:  icheck 584192
    Block Inode number
    584192      37486656
    debugfs:  ncheck 37486656
    Inode Pathname
    37486656    /gry/World of Warcraft 3.3.5a (no install)/Data/patch.MPQ

Polecenie `open` przechodzi do analizy systemu plików na wskazanej partycji. Następnie przy pomocy
`testb` sprawdzamy czy dany blok jest w użyciu czy też nie. Jeśli jest, znaczy, że znajduje się tam
plik. Sprawdzamy zatem przy pomocy `icheck`, który inode używa tego bloku, a następnie przy pomocy
`ncheck` sprawdzamy jaka ścieżka pliku na dysku jest mu przypisana. Po nitce do kłębka i tak
znaleźliśmy plik, którego dane znajdują się na uszkodzonym sektorze. No taa, też nie miało gdzie
tego sektora wywalić...

Co oznacza, że plik znajduje się w obszarze bad sektora? Najlepiej odpowiedzieć na to pytanie
poprzez próbę skopiowania tego pliku:

    # ls -al  "/media/Filmy/gry/World of Warcraft 3.3.5a (no install)/Data/patch.MPQ"*
    -rwxrwx--- 1 morfik morfik 3.8G Jul  8  2012 /media/Filmy/gry/World of Warcraft 3.3.5a (no install)/Data/patch.MPQ*
    -rwxrwx--- 1 morfik morfik 1.2G Jul  8  2012 /media/Filmy/gry/World of Warcraft 3.3.5a (no install)/Data/patch.MPQ_back*

Jak widać skopiowało się tylko 1.2GiB danych z faktycznych 3.8GiB . Zaglądając w tym czasie do logu
S.M.A.R.T, można odnotować kilka kolejnych błędów. Jeśli tak się stało, znaczy, że znaleźliśmy
winowajcę. Oczywiście nie da się już tego pliku skopiować, dysk przywiesza system przy próbie
odczytu wadliwego sektora, jedyne co można zrobić to usunąć plik z systemu, a właściwie zwolnić
inode oraz ręcznie realokować ten sektor przy pomocy `dd` lub `hdparm` . Ja skorzystałem z `dd` :

    # dd if=/dev/zero of=/dev/mapper/crypt_filmy bs=4096 count=1 seek=584192
    # sync

Teoretycznie w ten sposób wyzerowany (nadpisany) sektor zostanie realokowany i już nigdy więcej dane
do wadliwego bloku nie trafią.

Przy zapisie danych bezpośrednio na partycję, ogromne znaczenie ma parametr `bs`, a on z kolei
zależy od tego czy dysk ma 512 czy 4096 bajtowe sektory. W przypadku takim jak mój, gdzie ciężko
było to oszacować, można spróbować techniki na jeża, czyli ustawić `bs=512` . Jeśli sektor nie
zostanie realokowany, prawdopodobnie mamy do czynienia z sektorami 4096 bajtowymi, bo by sektor
został przeniesiony, trzeba zapisać pozostałe 7 sektorów, co wypełni 4096 bajtów. Można to albo
ustawić przez `bs=512 count=8` albo `bs=4096 count=1` . Należy pamiętać, że w obu przypadkach
zostanie nadpisanych 4096 bajtów i jeśli mamy 512 bajtowy sektor, to możemy mieć problemy. Ponadto,
parametr `seek` powinien też być wielokrotnością `8` przy 4096 bajtowych sektorach.

S.M.A.R.T powinien automatycznie zaktualizować odpowiednie atrybuty. Niektóre dyski jednak wymagają
by dokonać tego ręcznie przez wykonanie testu `offline`:

    # smartctl -t offline /dev/sda

Będzie on trwał ładnych parę godzin ale można używać PC jak gdyby nigdy nic, z tym, że im bardziej
będzie dysk obciążony, tym wolniej zostanie przeskanowany.

## Problemy z realokowaniem bad sektora

U mnie ten powyższy schemat nie zdał egzaminu, być może dlatego, że próbowałem przez `/dev/mapper/`
. W każdym razie, próba nadpisania tego sektora wyrzucała błąd zapisu. Za bardzo nie wiedziałem w
czym tkwi problem z realokowaniem tego bad bloka, dlatego też posłużyłem się `hdparm` . Chciałem
sprawdzić czy jemu się uda realokować padnięty sektor. Tutaj uwaga na numerki -- podajemy liczbę
wyrzuconą przez raport S.M.A.R.T:

    # hdparm --yes-i-know-what-i-am-doing --write-sector 104284160 /dev/sda

    /dev/sda:
    re-writing sector 104284160: succeeded

W końcu jakiś postęp. Sprawdźmy zatem co S.M.A.R.T nam powie:

    # smartctl -A /dev/sda | egrep "Reallocated|Pending|Uncorrectable"
      5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       0
    196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       0
    197 Current_Pending_Sector  0x0032   200   200   000    Old_age   Always       -       0
    198 Offline_Uncorrectable   0x0030   200   200   000    Old_age   Offline      -       2

A to dopiero dziwne, wartość atrybutu `197` została wyzerowana. Poprzednio była tam wartość `1` .
Atrybut `5` tez ma zero. Szukając info na ten temat, okazało się, że nawet takie problemy z sektorem
jak ja miałem, niekoniecznie oznaczają, że sektor będzie skazany na zapomnienie i odesłany w nicość.
Gdy tego typu sytuacja się przytrafia, oznacza to nic innego jak przywrócenie sektora do życia,
czyli udało się go odblokować.

W przypadku gdyby się nie udało, raport smart by wyglądał jak poniżej:

      5 Reallocated_Sector_Ct   0x0033   200   200   140    Pre-fail  Always       -       1
    196 Reallocated_Event_Count 0x0032   200   200   000    Old_age   Always       -       1
    197 Current_Pending_Sector  0x0032   200   200   000    Old_age   Always       -       0
    198 Offline_Uncorrectable   0x0030   200   200   000    Old_age   Offline      -       2

Zostanie wyzerowany atrybut `197`, a do atrybutów `5` i `196` zostanie dopisane `+1`. Natomiast
atrybut `198` nie zostanie zmieniony.

Po zakończeniu prac, przeprowadzić musimy jeszcze test skanowania całej powierzchni dysku:

    # smartctl -t offline /dev/sda

Powinno to wyzerować atrybut `198` . U mnie, po przejściu testu, ten atrybut nadal ma wartość `2` i
nie mam zielonego pojęcia jak się go pozbyć. Przejście testu powinno też zapalić zieloną lampkę i
oznaczyć urządzenie jako sprawne.

Poniżej info o testach mojego dysku:

    # smartctl -l selftest /dev/sda
    smartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.12-0.slh.3-aptosid-amd64] (local build)
    Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org

    === START OF READ SMART DATA SECTION ===
    SMART Self-test log structure revision number 1
    Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
    # 1  Short offline       Completed without error       00%     18858         -
    # 2  Extended offline    Completed without error       00%     18858         -
    # 3  Conveyance offline  Completed without error       00%     18854         -
    # 4  Short offline       Completed without error       00%     18853         -
    # 5  Short offline       Completed: read failure       90%     18852         104284160
    # 6  Short offline       Completed: read failure       90%     18845         104284160
    # 7  Short offline       Completed: read failure       90%     18839         104284160
    # 8  Short offline       Completed: read failure       90%     18837         104284160
    # 9  Short offline       Completed: read failure       90%     18836         104284160
    #10  Short offline       Completed: read failure       90%     18836         104284160
    #11  Short offline       Completed: read failure       90%     18836         104284160
    #12  Short offline       Completed without error       00%     18673         -
    #13  Short offline       Completed without error       00%     18498         -
    #14  Short offline       Completed without error       00%        85         -
    7 of 7 failed self-tests are outdated by newer successful extended offline self-test # 2

Oraz log chwilę po ukończeniu
    skanowania:

    Nov 22 01:05:51 morfikownia smartd[3463]: Device: /dev/sda [SAT], 2 Offline uncorrectable sectors
    Nov 22 01:05:51 morfikownia smartd[3463]: Device: /dev/sda [SAT], previous self-test completed without error
    Nov 22 01:05:51 morfikownia smartd[3463]: Device: /dev/sda [SAT], Self-Test Log error count decreased from 7 to 0
    Nov 22 01:05:51 morfikownia smartd[3463]: Device: /dev/sda [SAT], Self-Test Log does no longer report errors, warning condition reset after 1 email

Co prawda u mnie udało się wyleczyć dysk z bad sektora ale pierwszy padnięty sektor nie oznacza, że
trzeba od razu lecieć do sklepu z zamiarem kupna nowego urządzenia. Trzeba tylko monitorować czy aby
liczba uszkodzonych sektorów się czasem nie zwiększa. Oczywiście każdy bad sektor oznacza utratę
danych, w tym przypadku dość kosztowną, bo prawie 4GiB. Choć w sumie jakby to był film, to pewnie
dałoby radę wyciąć kawałek od początku filmu do sektora i od sektora do końca filmu, a potem oba
kawałki złączyć i może nawet by nie było zauważalnej różnicy.
