---
author: Morfik
categories:
- Linux
date: "2015-06-18T18:32:32Z"
date_gmt: 2015-06-18 16:32:32 +0200
published: true
status: publish
tags:
- system-plików
- hdd/ssd
- ext4
title: Etykieta systemu plików i jej dostosowanie
---

W poprzednim wpisie dostosowywaliśmy [zarezerwowane
miejsce]({{< baseurl >}}/post/zarezerwowane-miejsce-w-systemie-plikow-ext4/) na określonych
partycjach dla systemowych procesów. Okazuje się także, że zmiana etykiety systemu plików może
przysporzyć wiele problemów początkującym użytkownikom linuxa. Choć jeśli chodzi akurat o nadawanie
czy zmianę etykiet, to tutaj już mamy możliwość przeprowadzenia tej operacji z poziomu narzędzi GUI,
takich jak `gparted` , z tym, że niektóre jego komunikaty mogą nieco odstraszać.

<!--more-->
## Etykieta w gparted

Etykieta to taka ludzka forma nazywania systemu plików. Nie musimy się do niego dowoływać za pomocą
urządzeń typu `/dev/sda1` , czy po [numerkach
UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), które nie są zbytnio przyjazne
człowiekowi. Zamiast nich możemy podać krótką nazwę, [nie dłuższą niż 16
znaków](https://wiki.archlinux.org/index.php/Ext3#Assigning_a_label), i to jej użyć we wszystkich
operacjach, np. w pliku `/etc/fstab` . Dodatkowo system potrafi montować partycje automatycznie w
oparciu właśnie o etykietę. Także umiejętność jej dostosowania jest wielce przydatna.

Na początek spróbujemy zmienić etykietę przy pomocy `gparted` . Odpalamy zatem tę aplikację i
wybieramy pożądany dysk. Następnie klikamy prawym przyciskiem myszy na danej partycji i powinno nam
się ukazać poniższe menu:

![]({{< baseurl >}}/img/2015/06/1.etykieta-zmiana-gparted.png)

Wybieramy `Label` (etykieta) i podajemy nazwę. Przy dokonywaniu zmian, zostanie nam wyrzucony ten
poniższy komunikat:

![]({{< baseurl >}}/img/2015/06/2.etykieta-zmiana-blad.png)

Informuje on nas, że dokonywanie zmian w partycjach zwykle kończy się utratą danych. Nie wiem czemu
ten komunikat jest wyświetlany w przypadku operacji, które do niej nie prowadzą, a taką przecie jest
zmiana etykiety. Zatem możemy zwyczajnie ten monit zignorować i po chwili etykieta systemu plików
powinna zostać zmieniona.

## Zmiana etykiety via e2label

W paczce `e2fsprogs` mamy dostępne narzędzie `e2label` . Odpowiada ono dokładnie za to samo zadanie,
które zostało opisane wyżej, z tym, że nie potrzebujemy do niego sesji graficznej i całą procedurę
zmiany etykiety możemy przeprowadzić z poziomu konsoli.

Na początek sprawdźmy jaką etykietę ma system plików, który chcemy poddać edycji. Robimy to przez
podanie w argumencie ścieżki do urządzenia (partycji) w katalogu `/dev/` :

    # e2label /dev/sdb1
    partycja1

Zmieńmy teraz tę nazwę na jakąś inną:

    # e2label /dev/sdb1 bad
    # e2label /dev/sdb1
    bad

Jak widać, etykieta została zmieniona i to bez zbędnych ostrzeżeń.

## Zmiana etykiety przy pomocy tune2fs

Istnieje także możliwość zmiany etykiety systemu plików za pomocą narzędzia `tune2fs` , tak
przynajmniej można wywnioskować z manuala:

> It is also possible to set the filesystem label using the -L option of tune2fs(8).
>
> [man e2label](http://manpages.ubuntu.com/manpages/xenial/en/man8/e2label.8.html)

Zanim jednak skorzystamy z sugerowanego parametru `-L` , sprawdźmy czy jesteśmy w stanie odczytać
etykietę przy pomocy samego `tune2fs` :

    # tune2fs -l /dev/sdb1 | grep "Filesystem volume name:"
    Filesystem volume name:  bad

By teraz zmienić tę nazwę, wpisujemy w terminal poniższe polecenie:

    # tune2fs /dev/sdb1 -L good
    tune2fs 1.42.13 (17-May-2015)

    # tune2fs -l /dev/sdb1 | grep "Filesystem volume name:"
    Filesystem volume name:  good

Jak widać, zmiana przebiegła również bez większych problemów.

## Ustawianie etykiety przy tworzeniu systemu plików

Podobnie jak w przypadku narzędzia `tune2fs` , istnieje możliwość sprecyzowania dodatkowego
parametru przy tworzeniu systemu plików i jak możemy wyczytać w manualu, również jest to opcja `-L`
:

> `-L` new-volume-label
> Set the volume label for the filesystem to new-volume-label. The maximum length of the volume
> label is 16 bytes.
>
> [man mkfs.ext4](http://manpages.ubuntu.com/manpages/xenial/en/man8/mkfs.ext4.8.html)

Zatem nadanie etykiety przy tworzeniu systemu plików sprowadza się do wydania poniższego polecenia:

    # mkfs.ext4 -L bad /dev/sdb1

Trzeba jednak pamiętać, że w przypadku gdybyśmy podali etykietę dłuższą niż 16 znaków, zostanie ona
przycięta.
