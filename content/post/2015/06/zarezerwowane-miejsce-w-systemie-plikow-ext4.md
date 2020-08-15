---
author: Morfik
categories:
- Linux
date: "2015-06-18T17:29:16Z"
date_gmt: 2015-06-18 15:29:16 +0200
published: true
status: publish
tags:
- system-plików
- hdd
- ssd
- ext4
title: Zarezerwowane miejsce w systemie plików ext4
---

Zwykle nie zwracamy uwagi na to jak formatujemy partycje w systemie linux i akceptujemy domyślne
ustawienia jakie przyjęli sobie deweloperzy danej dystrybucji. Nie ma tutaj znaczenia czy
instalujemy świeży system za pośrednictwem instalatora i przy jego pomocy kroimy dysk, czy też
tworzymy partycje indywidualnie już z poziomu jakiegoś zainstalowanego systemu, bądź też płytki czy
pendrive live. Domyślne ustawienia mają spełniać oczekiwania jak największej liczby odbiorców i nie
zawsze nam one odpowiadają. W przypadku formatowania dysku, problematyczne może być rezerwowanie
miejsca dla procesów użytkownika root.

<!--more-->
## Domyślne parametry

Zarówno partycjonowanie dysku w instalatorze jak i jego ręczny odpowiednik, w skład którego wchodzi
wykorzystanie min. narzędzi `fdisk` / `gdisk` i `mkfs.*` , sprowadza się do poinstruowania systemu
odnośnie konfiguracji nowo utworzonego systemu plików. Załóżmy, że chcemy sformatować jakąś
partycję. Co oznacza słowo **format** w tym kontekście? Chodzi generalnie o utworzenie na nowo
systemu plików. Nie musimy nawet usuwać i tworzyć partycji. Wystarczy ograniczyć się do skorzystania
z narzędzia `mkfs.ext4` . W jego manualu możemy przeczytać informację na temat parametru `-m` :

> `-m` reserved-blocks-percentage
> Specify the percentage of the filesystem blocks reserved for the super-user. This avoids
> fragmentation, and allows root-owned daemons, such as syslogd(8), to continue to function
> correctly after non-privileged processes are prevented from writing to the filesystem. The default
> percentage is 5%.
>
> [man mkfs.ext4](http://manpages.ubuntu.com/manpages/xenial/en/man8/mkfs.ext4.8.html)

Jak widzimy, ten powyższy parametr przybiera domyślną wartość `5%`. Zatem jeśli tworzymy system
plików na partycji 1TiB, to 50GiB będzie zarezerwowane dla procesów działających z prawami root.
Chodzi o to, że gdy partycja zostanie z jakiegoś powodu wypełniona po brzegi, te kluczowe procesy
systemowe mogą przestać działać, co może zaowocować powieszeniem się systemu operacyjnego no i
oczywiście brakiem możliwości uruchomienia go.

## Jak dostosować zarezerwowane miejsce

Jeśli nasz system jest rozrzucony na kilku partycjach, np. logi trzymamy w `/var/log/` , pliki
użytkowników w katalogu `/home/` , to możemy odpowiednio skonfigurować sobie poszczególne partycje
tak by tylko określone z nich miały zarezerwowaną pewną cześć systemu plików. W przypadku
pozostałych, to zarezerwowane miejsce zwyczajnie idzie na zmarnowanie.

Jeśli proces formatowania mamy już za sobą i nie przywiązywaliśmy uwagi do poszczególnych
parametrów, to prawdopodobnie możemy się znaleźć w takiej sytuacji, gdzie mamy już na dysku trochę
danych i nie uśmiecha nam się ponowne tworzenie systemu plików i to tylko po to by odzyskać marne 5%
wolnego miejsca. Na szczęście nie musimy formatować partycji raz jeszcze by odpowiednio dostosować
sobie ten parametr. W pakiecie `e2fsprogs` jest dostępne narzędzie `tune2fs` , które pozwala nam
przeprowadzić tego typu operacje bez obaw o utratę danych.

Jeśli nie mamy pojęcia ile procent nasze partycje mają zarezerwowanego miejsca, możemy to sprawdzić
wydając poniższe polecenie:

    # tune2fs -l /dev/sda2 | grep "Reserved block count"
    Reserved block count:     13107

Liczba `13107` to ilość zarezerwowanych bloków, co przekłada się na nieco ponad 51MiB (13107\*4096).
Sama partycja ma niecały 1GiB, także 5% próg obowiązuje. Jeśli nie potrzebujemy tych zarezerwowanych
bloków, to wydajemy to poniższe polecenie:

    # tune2fs -m 0 /dev/sda2
    tune2fs 1.42.13 (17-May-2015)
    Setting reserved blocks percentage to 0% (0 blocks)

Trzeba pamiętać, by nie dokonywać zmian parametrów przy podmontowanym systemie plików. Sam parametr
`-m` możemy także wykorzystać bezpośrednio przy tworzeniu nowego systemu plików via `mkfs.ext4` .
