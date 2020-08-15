---
author: Morfik
categories:
- Linux
date: "2015-10-06T21:57:06Z"
date_gmt: 2015-10-06 19:57:06 +0200
published: true
status: publish
tags:
- udev
- hdd
- ssd
title: Parametr multcount w dyskach twardych
---

[W manualu hdparm](http://manpages.ubuntu.com/manpages/xenial/pl/man8/hdparm.8.html) możemy
przeczytać o opcji `-m` , która szerzej jest znana jako **multcount**. Obecnie praktycznie każdy
dysk w większym lub mniejszym stopniu ma zaimplementowaną jej obsługę, tj. wartość tego parametru
różni się i zwykle im większą, tym lepiej dysk powinien się sprawować. Nie wszystkie dyski mają tę
opcję włączoną standardowo. Na przykład te z rodziny WDC, jak czytamy w dokumentacji, są znane z
tego, że działają wolniej po jej ustawieniu. Jako, że mam dysk firmy Western Digital, to
postanowiłem sprawdzić jak, o ile w ogóle, zmieni się wydajność takiego urządzenia po przestawieniu
tego parametru.

<!--more-->
## Co to jest multcount

Zgodnie z opisem znalezionym w manualu, **multcount** to metoda operowania na dysku twardym, która
umożliwia jednoczesny zapis/odczyt wielu sektorów w obrębie jednego
[](http://www.isep.pw.edu.pl/~slawekn/info1/lekcja1/segment8.htm)przerwania wejścia/wyjścia (I/O).
Dyski nie posiadające tego typu opcji, lub też te, które mają ją wyłączoną, mają możliwość
odczytu/zapisu tylko jednego sektora na przerwanie wejścia/wyjścia. W sporej części przypadków,
ustawienie tej opcji powoduje zredukowanie overhead'u dla systemu, który niosą ze sobą operacje
dysku twardego. Czasem także można zaobserwować zwiększenie transferu danych.

Trzeba jednak mieć na uwadze, że niewłaściwe operowanie tym parametrem może doprowadzić do masowej
utraty danych przez uszkodzenie systemu plików na dysku.

## Sprawdzenie dostępności multcount

Zanim zaczniemy się bawić parametrem **multcount**, sprawdźmy w ogóle czy nasz dysk ma
zaimplementowaną jego obsługę. Do tego celu posłuży nam narzędzie `hdparm` dostępne w pakiecie pod
tą samą nazwą. Po jego zainstalowaniu, wklepujemy w terminal poniższe polecenie:

    # hdparm -i /dev/sda | grep -i mult
     BuffType=unknown, BuffSize=16384kB, MaxMultSect=16, MultSect=0

Parametr `MaxMultSect` określa maksymalną ilość sektorów, które dysk będzie w stanie
zapisać/odczytać podczas jednego przerwania wejścia/wyjścia. Z kolei zaś `MultSect` wskazuje
aktualnie ustawioną wartość. W tym powyższym przypadku opcja **multcount** jest wyłączona dla tego
dysku ale nic nie stoi na przeszkodzie by ją włączyć.

## Włączenie multcount

Przy pomocy `hdparm` możemy przestawić wartość `MultSect` , tak by odpowiadała maksymalnej wartości
obsługiwanej przez dysk. W tym przypadku jest to `16` . By tego dokonać, wydajemy poniższe
polecenie:

    # hdparm --yes-i-know-what-i-am-doing -m 16 /dev/sda

Jako, że ta powyższa akcja może uszkodzić dane na dysku, trzeba podać dodatkowy parametr
`--yes-i-know-what-i-am-doing` . Niemniej jednak, utrata danych w przypadku ustawienia parametru
**multcount** należy chyba do niezmiernie żadnych, bo jeszcze nie natrafiłem na taką sytuację.

Po wydaniu powyższego polecenia, w logu systemowym powinniśmy zanotować komunikat podobny do tego
poniżej:

    kernel: ata1.00: configured for UDMA/100
    kernel: ata1: EH complete

Ostatnim krokiem jest zweryfikowanie poprawności wprowadzonych zmian i również robimy to za pomocą
`hdparm` :

    # hdparm -m /dev/sda

    /dev/sda:
     multcount     = 16 (on)

Jak widzimy, parametr `multcount` został przepisany z wartości `0` do `16` .

## Debianowy hdparm

W chyba większości dystrybucji linux'a, `hdparm` występuje w postaci zwykłego pliku wykonywalnego
bez dodatkowych skryptów init i plików konfiguracyjnych. W debianie zaś, mamy dość długi skrypt init
i do tego dwa pliki `/etc/hdparm.conf` oraz `/etc/default/hdparm` . Generalnie rzecz biorąc, ten
mechanizm działa. Problematyczne jednak jest ustawianie opcji, które wymagają podania
`--yes-i-know-what-i-am-doing` , a tak jest w tym przypadku. Wobec takiego stanu rzeczy, ta opcja (i
inne jej podobne) nie zostaje ustawiona poprawnie. W logu natomiast możemy odnotować poniższy
komunikat:

```
 hdparm[8567]: Setting parameters of disc:
hdparm[8567]: /dev/sda:
hdparm[8567]: setting multcount to 16
systemd[1]: Started LSB: Tune IDE hard disks.
hdparm[8567]: /dev/sda failed!
```

Zalecaną metodą na aplikowanie ustawień przy pomocy `hdparm` jest [napisanie prostych reguł dla
udev'a](https://lists.freedesktop.org/archives/systemd-devel/2012-June/005600.html) , przy
jednoczesnej rezygnacji ze wszelkich mechanizmów specyficznych dla danych dystrybucji. W debianie,
trzeba będzie dodatkowo wyłączyć skrypt init od `hdparm` z autostartu systemu.

## Reguła dla udev'a

Problem ze zmianą ustawień za sprawą `hdparm` jest taki, że obowiązują one jedynie do restartu
maszyny. Można pisać usługi dla systemd, można robić skrypty i pliki konfiguracyjne, tak jak to jest
zrobione w przypadku debiana ale można także pójść po najmniejszej linii oporu i zaprzęgnąć do tego
celu udev'a.

Trzeba pamiętać, że określanie dysków po nazwach typu `sda` czy `sdb` może zakończyć się tragicznie
gdy mamy kilka dysków w systemie i z jakiegoś powodu zamienią się one tymi ostatnimi literkami.
Dlatego też lepiej wykorzystać linki znajdujące się w katalogu `/dev/disk/by-id/` . Sprawi to, że
będziemy mieć do czynienia z numerami seryjnymi urządzeń, co czyni je unikalnymi wartościami, w
oparciu o które możemy pisać reguły udev'a i mieć przy tym pewność, że zostaną one zaaplikowane
jedynie do konkretnego dysku twardego.

Sama reguła dla udev'a nie jest jakoś skomplikowana ale trzeba mieć na uwadze, że linki do urządzeń,
o których mowa powyżej, są tworzone za pomocą innej reguły udev'a, a konkretnie jest to
`60-persistent-storage.rules` . Dodatkowo w debianie jest dostępny również plik
`/lib/udev/rules.d/85-hdparm.rules` i dobrze jest by nasza reguła miała numer większy od lub równy
`85` . Stwórzmy zatem plik w katalogu `/etc/udev/rules.d/` i nazwijmy go `90-hdparm.rules` . Teraz
już tylko pozostało wywołanie `hdparm` z określonymi opcjami w przypadku wykrycia przez system
odpowiedniego dysku twardego. Reguła wygląda zatem mniej więcej tak:

    ACTION=="add", SUBSYSTEM=="block", \
          ENV{ID_SERIAL}=="WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928", \
          RUN+="/sbin/hdparm --yes-i-know-what-i-am-doing -m 16 /dev/disk/by-id/ata-WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928"

Jak widać, dopasowujemy urządzenie po numerze seryjnym, ten zaś możemy uzyskać wydając poniższe
polecenie:

    # udevadm info --name /dev/sda | grep -i serial
    E: ID_SERIAL=WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928
    E: ID_SERIAL_SHORT=WD-WX41A90E5928

Jeśli za jakichś powodów opcje użyte w regule są niejasne, odsyłam do przeczytania artykułu [na
temat pisania reguł dla udev'a]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/),
gdzie zostały one wyjaśnione.

By przekonać się czy reguła została poprawnie napisana, przeprowadzamy jej test:

    # udevadm test /sys/block/sda/
    ...
    run: '/lib/udev/hdparm'
    run: '/sbin/hdparm --yes-i-know-what-i-am-doing -m 16 /dev/disk/by-id/ata-WDC_WD2500BEKT-60A25T1_WD-WX41A90E5928'

Jedna z linijek z `run` sugeruje wywołanie pożądanego przez nas polecenia i nie ma przy tym
zwróconego żadnego błędu. Zatem reguła będzie aplikowana podczas startu komputera.
