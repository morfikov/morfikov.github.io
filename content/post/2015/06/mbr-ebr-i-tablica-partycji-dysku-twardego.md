---
author: Morfik
categories:
- Linux
date: "2015-06-12T18:44:53Z"
date_gmt: 2015-06-12 16:44:53 +0200
published: true
status: publish
tags:
- mbr
- partycja
- ebr
- ssd
- hdd
title: MBR, EBR i tablica partycji dysku twardego
---

[MBR](https://en.wikipedia.org/wiki/Master_boot_record) to pierwszy sektor dysku twardego, który
może ma jedynie 512 bajtów ale jest to bardzo krytyczny obszar nośnika, po którego uszkodzeniu czy
nadpisaniu tracimy dostęp do wszystkich partycji znajdujących się na dysku. Ci bardziej przezorni
użytkownicy robią sobie backup tego kluczowego punktu, tak by w przypadku problemów mogli go sobie
przywrócić. Nie zawsze jednak potrzebujemy przywracać cały sektor MBR, w większości przypadków
będziemy potrzebować jedynie kodu bootloadera lub samej tablicy partycji.

<!--more-->
## Struktura MBR i EBR

Jak już nadmieniłem we wstępie, MBR ma 512 bajtów długości, z czego pierwsze 446 bajtów zajmuje
program rozruchowy (bootloader). Druga część MBR, to tablica partycji i zwykle zawiera 1-4 pozycje
opisujące poszczególne partycje podstawowe. Każdy wpis ma 16 bajtów, razem 64 bajty. Na końcu MBR
znajdują się 2 bajty sygnatury rozruchu. W sumie daje to 512 bajtów. Poniżej milusia fotka
([źródło](http://www.invoke-ir.com/2015/05/ontheforensictrail-part2.html)) przestawiająca
strukturę MBR, na wypadek gdyby ktoś miał problemy z wyobraźnią:

![]({{< baseurl >}}/img/2015/06/1.struktura-mbr.png#huge)

Wraz ze wzrostem pojemności dysków i zapotrzebowaniem na kolejne partycje, pojawił się problem
ograniczenia ich liczby jedynie do 4. Jako tymczasowe rozwiązanie zaproponowano
[EBR](https://en.wikipedia.org/wiki/Extended_boot_record) , który znalazł wykorzystanie w przypadku
partycji rozszerzonej, na której można ulokować dowolną liczbę dysków logicznych. EBR również
posiada 512 bajtów i praktycznie niczym nie różni się od MBR, za wyjątkiem sposobu działania. Rzućmy
zatem okiem na poniższą fotkę ([źródło](http://thestarman.pcministry.com/asm/mbr/PartTables2.htm)):

![]({{< baseurl >}}/img/2015/06/2.struktura-mbr-ebr.png#big)

Wyżej mamy rozrysowaną dokładnie partycję rozszerzoną. Jej wpis w MBR niczym nie różni się od
pozostałych partycji podstawowych. Generalnie rzecz biorąc, to z punktu widzenia maszyny, ten
powyższy dysk ma tylko 4 partycje. Mamy tam jednak 3 sektory EBR. Lokalizacja każdego z nich
znajduje się zawsze w pierwszym sektorze logicznego dysku. Wyżej wspomniałem, że EBR niczym nie
różni się od MBR, przynajmniej jeśli chodzi o strukturę. Gdybyśmy zajrzeli do środka EBR, to się
okazuje, że miejsce na kod bootloadera (pierwsze 446 bajtów) jest niewykorzystywane, a ponadto,
każdy EBR ma zaprogramowane linkowanie do następnego EBR -- taki swojego rodzaju łańcuch. I w
powyższym przykładzie pozycja nr 8 linkuje do 11, a ta z kolei do 14. Jak one to robią? Za pomocą
tablicy partycji, a konkretnie dwóch jej pozycji. Pierwsza z nich określa początkowy sektor i ich
ilość (tak samo jak w przypadku tych podstawowych partycji), natomiast druga wskazuje położenie
następnego sektora EBR, chyba, że jest to ostatni dysk logiczny, wtedy będzie zajmować 0 bajtów. A
co linkuje do pierwszego dysku logicznego? Nic. On jest oznaczany przez partycję rozszerzoną, choć z
technicznego punktu widzenia jest to partycja podstawowa.

Rzućmy okiem jeszcze na to co zwykle nam zwraca `fdisk` :

    Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048    45641727    22819840   83  Linux
    /dev/sdb2        45641728    80166911    17262592   83  Linux
    /dev/sdb3        80166912   104224767    12028928   83  Linux
    /dev/sdb4       104224768   156299263    26037248    5  Extended
    /dev/sdb5       104226816   119328767     7550976   83  Linux
    /dev/sdb6       119330816   128720895     4695040   83  Linux
    /dev/sdb7       128722944   156299263    13788160   83  Linux

W przypadku pierwszego dysku logicznego `sdb5` , sektor EBR jest na pozycji parametru "start"
partycji rozszerzonej `sdb4`. Natomiast w przypadku kolejnych dysków logicznych, sektor EBR znajduje
się zaraz za końcem poprzedniej partycji. Między tymi dyskami logicznymi jest trochę wolnego
miejsca. W przypadku równania do 1MiB, jest to 2048 sektorów, w tym 1 sektor na EBR. Jeśli byśmy
wykonali kopię tej pozycji co jest w parametrze "start", skopiujemy pierwszy sektor systemu plików,
a nie EBR. Zatem by mieć jasność, pierwszy sektor EBR jest na pozycji `104224768` , drugi na
`119328768` i trzeci na `128720896` .

## Backup MBR oraz tablicy partycji

Większość problemów z dyskiem czy pendrive dotyczy sfery programowej. Zatem nośnikowi niby nic nie
dolega z fizycznego punktu widzenia, np. nie ma na nim uszkodzonych sektorów, ale z jakichś powodów
nie możemy uzyskać dostępu do partycji na nim się znajdujących. Jeśli do tej pory nasz twardziel
działa bez zarzutu, to przydałoby się zrobić backup całego sektora MBR oraz, w przypadku posiadania
partycji rozszerzonej, kopi wpisów całej tablicy partycji. Wszystko sprowadza się do wydania tych
dwóch poniższych poleceń:

    # dd if=/dev/sdb of=./mbr bs=512 count=1
    # sfdisk -d /dev/sdb > sdb_table

Trzeba jednak pamiętać, że zapisany sektor MBR nie posiada żadnych informacji na temat wszystkich
sektorów EBR, a jedynie wskazuje na pierwszy z nich. Także jeśli z jakiegoś powodu nie będzie można
odnaleźć kolejnego sektora EBR, część partycji może zaginąć.

## Przywracanie MBR

Jak widzieliśmy wyżej na schemacie MBR, ten sektor składa się z dwóch części. Jeśli jakiś system
operacyjny nam nadpisał kod bootloadera, to możemy tylko tę część MBR przywrócić.

    # dd if=mbr of=/dev/sdb bs=446 count=1

W przypadku gdybyśmy chcieli przywrócić cały MBR, to wpisujemy poniższe polecenie:

    # dd if=mbr of=/dev/sdb bs=512 count=1

## Przywracanie tablicy partycji

Jeśli chodzi o samą tablicę partycji, to mamy dwa wyjścia. Pierwsze z nich zakłada wycięcie
określonego obszaru z wcześniej utworzonego backupu sektora MBR i wklejenie go w odpowiednie
miejsce na dysku przy pomocy:

    # dd if=./mbr of=/dev/sdb bs=1 count=64 skip=446 seek=446

Drugie rozwiązanie polega na zaprzęgnięciu do pracy narzędzia `sfdisk` :

    # sfdisk /dev/sdb < sdb_table

W taki oto sposób możemy zabezpieczyć się na wypadek programowego uszkodzenia dysku.
