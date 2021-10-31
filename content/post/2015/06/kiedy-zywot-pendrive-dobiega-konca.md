---
author: Morfik
categories:
- Hardware
date: "2015-06-09T20:47:42Z"
date_gmt: 2015-06-09 18:47:42 +0200
published: true
status: publish
tags:
- pendrive
- flash
- karta-sd
GHissueID: 95
title: Kiedy żywot pendrive dobiega końca...
---

Pendrive to jedno z cenniejszych urządzeń w obecnych czasach. Jest mały, poręczny i potrafi
przechować parę set GiB danych, oraz jako że nie ma części ruchomych, można nie obchodzić się z nim
jak z jajkiem i to bez ryzyka utraty danych. Często wykorzystuje się go jako środek transportu na
krótkie odległości między kilkoma maszynami, gdzie z pewnych powodów nie można pociągnąć kabla
sieciowego, nie wspominając przy tym o wifi. Systemy live wręcz nie mogą się bez nich obejść.
Jesteśmy w sumie uzależnieni od tych małych urządzeń tak bardzo, że dostrzegamy to dopiero w chwili
gdy, któryś pendrive zaczyna odmawiać posłuszeństwa.

<!--more-->
## Input/output error

Dziś z rana postanowiłem zaktualizować sobie obraz live, który zwykle po wygenerowaniu wgrywam na
pendrive. Nie był to zwykły ranek, bo błąd widoczny wyżej pojawił mi się przy kopiowaniu samego
obrazu via `dd` :

    # dd if=/media/live/debian.iso of=/dev/sdc
    dd: writing to '/dev/sdc': Input/output error
    99105+0 records in
    99104+0 records out
    50741248 bytes (51 MB) copied, 21.6581 s, 2.3 MB/s

Jak widać wyżej, po skopiowaniu jedynie 50MiB, proces został przerwany w wyniku błędu. Dotyczy to
tylko i wyłącznie wgrywania obrazów bit po bicie. W każdym innym przypadku, nie powinno być
problemów. Samo tworzenie partycji na tym pendrive nie zwraca żadnych błędów:

![linux-gparted-pendrive](/img/2015/06/1.linux-gparted-pendrive.png#huge)

Zatem co się z nim dzieje?

## Skanowanie pendrive przy pomocy f3

By ustalić przyczynę takiego stanu rzeczy, musimy sięgnąć do narzędzi diagnostycznych. W przypadku
pamięci flash jest to min. `f3` , dostępny w pakiecie pod tą samą nazwą. Testuje on faktyczną
pojemność pamięci flash ogółem, nie tylko pendrive ale również i karty sd w celu ustalenia ich
[faktycznej pojemności](/post/rzeczywista-pojemnosc-pendrive-i-kart-sd/). Ten
pendrive nie jest
[podróbką](http://www.ebay.com/gds/All-About-Fake-Flash-Drives-2013-/10000000177553258/g.html) ale
ma już swoje lata i był dość ostro wykorzystywany przez ten czas. Zatem podejrzewam, że już pada ze
starości.

By zweryfikować czy aby moje przypuszczenia są prawdziwe, przeskanowałem tego pendrive i co się
okazało? Poniżej jest zobrazowany zapis danych:

    # f3write /media/morfik/224e0447-1b26-4c3e-a691-5bf1db650d21
    Free space: 14.06 GB
    Creating file 1.h2w ... OK!
    Creating file 2.h2w ... OK!
    Creating file 3.h2w ... OK!
    Creating file 4.h2w ... OK!
    Creating file 5.h2w ... OK!
    Creating file 6.h2w ... OK!
    Creating file 7.h2w ... OK!
    Creating file 8.h2w ... OK!
    Creating file 9.h2w ... OK!
    Creating file 10.h2w ... OK!
    Creating file 11.h2w ... OK!
    Creating file 12.h2w ... OK!
    Creating file 13.h2w ... OK!
    Creating file 14.h2w ... OK!
    Creating file 15.h2w ... OK!
    Free space: 16.02 MB
    Average writing speed: 7.43 MB/s

Także zapis jest w miarę w porządku. Natomiast jeśli chodzi o odczyt tych plików, to już tak różowo
nie jest:

    # f3read /media/morfik/224e0447-1b26-4c3e-a691-5bf1db650d21
                      SECTORS      ok/corrupted/changed/overwritten
    Validating file 1.h2w ... 2097112/       40/      0/      0
    Validating file 2.h2w ... 2097120/       32/      0/      0
    Validating file 3.h2w ... 2097098/       54/      0/      0
    Validating file 4.h2w ... 2097148/        4/      0/      0
    Validating file 5.h2w ... 2097114/       38/      0/      0
    Validating file 6.h2w ... 2097152/        0/      0/      0
    Validating file 7.h2w ... 2097152/        0/      0/      0
    Validating file 8.h2w ... 2097152/        0/      0/      0
    Validating file 9.h2w ... 2097152/        0/      0/      0
    Validating file 10.h2w ... 2097152/        0/      0/      0
    Validating file 11.h2w ... 2097152/        0/      0/      0
    Validating file 12.h2w ... 2097152/        0/      0/      0
    Validating file 13.h2w ... 2097152/        0/      0/      0
    Validating file 14.h2w ... 2097152/        0/      0/      0
    Validating file 15.h2w ...   90664/        0/      0/      0

      Data OK: 14.05 GB (29450624 sectors)
    Data LOST: 84.00 KB (168 sectors)
                 Corrupted: 84.00 KB (168 sectors)
          Slightly changed: 0.00 Byte (0 sectors)
               Overwritten: 0.00 Byte (0 sectors)
    Average reading speed: 18.77 MB/s

Jak widzimy `168` sektorów na przestrzeni pierwszych 5GiB jest padniętych. No może nie do końca
padniętych ale są problemy z ich czytaniem. Jakby nie patrzeć, to głównie ten obszar tego pendrive
był wykorzystywany -- 3GiB na obrazy iso oraz 2GiB na szyfrowany persistence. Jako, że reszta
urządzenia jest zdrowa, można z niego jeszcze korzystać ale nie radziłbym wgrywać danych na ten
obszar początkowy.

## Analiza przy pomocy whdd

Innym narzędziem, które nam może pomóc w analizie struktury pendrive, to [whdd](http://whdd.org/) .
Skanuje ono i pokazuje czas dostępu do sektorów na całej powierzchni nośnika. W przypadku mojego
pendrive jest trochę problemów, poniżej focia z początku skanowania:

![linux-pendrive-whdd-scan](/img/2015/06/3.linux-pendrive-whdd-scan.png#huge)

Wyraźnie coś jest nie tak z tym niejednorodnym obszarem. Są, co prawda, tylko 4 błędy ale problemy z
odczytem/zapisem pozostałych sektorów mogą się również pojawić, dlatego `f3` zgłosił 168 sektorów.
Gdybym przeskanował nim jeszcze raz tego pendrive, prawdopodobnie ta liczba by uległa zmianie.

## Okrajanie pendrive

Możemy pokusić się o wykrojenie nieuszkodzonej przestrzeni, choć nie wiem czy jest jakiś sens
próbować odzyskać coś z pierwszych 5GiB. Nawet jeśli większość z tych komórek nie jest padniętych w
tej określonej chwili, to na pewno są w znacznym stopniu nadwyrężone i mogą paść lada moment.
Dlatego też ja bym jednak spisał na straty te dwie partycje, które miałem na tym pendrive i oznaczył
do użytku jedynie pozostałą przestrzeń.

Możemy także utworzyć partycję na tym niezbyt bezpiecznym obszarze urządzenia i spróbować
przeskanować go przy pomocy `fsck` z opcją `-c` , za sprawą to której zostanie zaprzęgnięty program
`badblocks` i wszystkie padnięte sektory zostaną oznaczone i dodane do i-węzła bad bloków, co
zapobiegnie ich alokacji:

![linux-badblocks-pendrive](/img/2015/06/2.linux-badblocks-pendrive.png#huge)

U mnie jednak ilość padniętych bloków ciągle zmieniała się z każdym kolejnym skanem i raczej wątpię
by w ten sposób szło coś ugrać.
