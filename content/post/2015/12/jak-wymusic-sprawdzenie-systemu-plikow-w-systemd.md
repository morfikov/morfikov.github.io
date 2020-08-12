---
author: Morfik
categories:
- Linux
date: "2015-12-04T20:11:46Z"
date_gmt: 2015-12-04 19:11:46 +0100
published: true
status: publish
tags:
- system-plików
- systemd
title: Jak wymusić sprawdzenie systemu plików w systemd
---

Jakiś czas temu opisywałem jak w systemach linux'owych przeprowadzić [sprawdzenie systemu plików pod
kątem ewentualnych błędów]({{< baseurl >}}/post/sprawdzanie-bledow-systemu-plikow-ext4/). Był tam
poświęcony kawałek na temat ręcznego wymuszenia takiego skanowania. Ten sposób, który został opisany
w tamtym wpisie działa wyśmienicie w przypadku sysvinit. Natomiast przy systemd mogą pojawić się
pewne problemy, w efekcie czego nie będziemy w stanie wymusić skanowania pewnych partycji.
Generalnie to rozchodzi się o tę główną, na której znajduje się system plików `/` . Postanowiłem się
przyjrzeć nieco temu mechanizmowi i sprawdzić czy faktycznie nic nie da się zrobić i czy musimy
czekać pełną ilość cykli startu systemu, by ten system plików został przez niego przeskanowany
automatycznie.

<!--more-->
## Kiedy kernel stara się wymusić sprawdzanie systemu plików

Kernel linux'a jest w stanie oszacować integralność danych w określonym systemie plików na podstawie
informacji zawartych w superbloku. Te informacje możemy uzyskać przy pomocy `tune2fs -l` , bądź też
i `dumpe2fs` . Nas tutaj interesują generalnie dwa parametry. Jeden z nich odpowiada za limit cykli
montowań systemu plików, a drugi zaś zwraca informację na temat tego ile razy już ten system plików
został zamontowany, poniżej przykład:

    # tune2fs -l /dev/sda2 | grep -i "mount count"
    Mount count:              3
    Maximum mount count:      30

Gdy te dwie wartości zrównają się, kernel zainicjuje sprawdzanie systemu plików. Co jednak w
przypadku, gdy chcielibyśmy przyśpieszyć nieco ten proces? Nie pomoże nam utworzenie pliku
`forcefsck` w katalogu `/` , ani też zresetowanie maszyny przy pomocy `shutdown -rF now` . Możemy
jednak skorzystać z kilku rozwiązań, które są w stanie wymusić skanowanie systemu plików praktycznie
niezależnie od posiadanego init'u.

## Obniżenie wartości parametru "Maximum mount count"

Jednym z tych bardziej niezalecanych rozwiązań jest obniżenie wartości parametru `Maximum mount
count` , którą widzieliśmy wyżej w informacjach zawartych w superbloku. Im ta wartość będzie
mniejsza, tym częściej system plików będzie skanowany. W przypadku obniżenia jej do `1` , kernel
będzie skanował system plików przy każdym uruchomieniu systemu. By obniżyć tę wartość, posłużymy
się narzędziem `tune2fs` , gdzie w opcji `-c` określamy maksymalną ilość montowań, przykładowo:

    # tune2fs -c 5 /dev/sda2

Oczywiście to nie jest dokładnie to o co nam chodzi, czyli o wymuszenie sprawdzania systemu plików
ale tak czy inaczej, ten parametr dobrze jest sobie dostosować w zależności od częstości ponownego
uruchamiania komputera.

## Zwiększenie wartości parametru "Mount count"

Jeśli chodzi zaś o parametr `Mount count` , to przy jego pomocy jesteśmy w stanie oszukać kernel.
Wystarczy ustawić tutaj większą wartość niż jest określona w `Maximum mount count` . Wtedy podczas
startu systemu, ten zobaczy, że licznik wybił już określoną ilość cykli i wymusi skanowanie systemu
plików. Do przestawienia wartości tego parametru również wykorzystamy `tune2fs` , z tym, że tym
razem skorzystamy z przełącznika `-C` , przykładowo:

    # tune2fs -C 40 /dev/sda2

Po resetowaniu maszyny, ten współczynnik zostanie wyzerowany.

## Oznaczenie systemu plików jako "dirty"

Generalnie rzecz biorąc, system plików jest skanowany w trzech przypadkach. Gdy licznik dobije do
wartości granicznej (opisane powyżej), gdy użytkownik wymusi skanowanie (to odpada przy systemd),
oraz gdy system plików zostanie niepoprawnie odmontowany. W tym ostatnim przypadku jest mu ustawiana
flaga `dirty` . Gdy kernel będzie próbował zamontować taki system plików, to automatycznie powinien
go przeskanować. Jakby nie patrzeć, w taki sposób oznaczony system plików prawie na pewno zawiera
błędy. Jak zatem oznaczyć dany system plików jako `dirty` i wymusić jego skanowanie?

W przypadku systemu plików `ext4` możemy posłużyć się narzędziem `debugfs` . Logujemy się zatem na
użytkownika root i wpisujemy w terminalu te poniższe polecenia:

    # debugfs
    debugfs 1.42.13 (17-May-2015)
    debugfs:  open -w /dev/sda2
    debugfs:  set_super_value  2
    debugfs:  close

Przy pomocy `set_super_value` zmieniliśmy stan ( `state` ) systemu plików. Do wyboru mamy trzy
wartości: `0` (not clean), `1` (clean) oraz `2` (not clean with errors). Po zmianie wartości,
możemy sprawdzić czy została ona poprawnie ustawiona przy pomocy tego poniższego polecenia:

    # tune2fs -l /dev/sda2 | grep state
    Filesystem state:         not clean with errors

Pamiętajmy by zmiany stanu systemu plików dokonywać jedynie gdy ten nie jest zamontowany. Jeśli
chcemy w ten sposób wymusić sprawdzanie systemu plików podczas fazy boot, to wystarczy przestawić
stan na `0` lub `2` . Poniżej zaś jest log, w którym możemy zaobserwować, że system wymusił
sprawdzenie w obu tych przypadkach:

    systemd-fsck[1308]: kabi was not cleanly unmounted, check forced.
    systemd-fsck[1320]: grafi contains a file system with errors, check forced.

## Parametry kernela fsck.mode oraz fsck.repair

Przeglądając jeszcze manual dotyczący
[systemd-fsck@.service](https://www.freedesktop.org/software/systemd/man/systemd-fsck@.service.html)
natrafiłem na dwa ciekawe parametry: `fsck.mode` oraz `fsck.repair` . Z opisu wychodzi na to, że
przy pomocy `fsck.mode` jesteśmy w stanie manipulować zachowaniem mechanizmu sprawdzania systemu
plików i możemy wymusić ( `force` ), pominąć ( `skip` ) lub zostawić domyślne ustawienia ( `auto` ).
Z kolei przy pomocy parametru `fsck.repair` możemy określić czy i jakie błędy kernel ma naprawiać.
Mamy tutaj do wyboru `preen` (naprawia łatwe błędy), `yes` (naprawia wszystkie błędy) oraz `no` (nie
naprawia błędów w ogóle). Oba te parametry dodajemy oczywiście w konfiguracji bootloader'a.

Problem z tymi parametrami jest taki, że nie da rady za ich pomocą wymusić sprawdzenia głównego
systemu plików ( `/` ) , bo ten zwraca coś w stylu:

    # systemctl status systemd-fsck-root.service
    ...
    Condition: start condition failed at Fri 2015-12-04 19:33:20 CET; 13min ago
               ConditionPathExists=!/run/initramfs/fsck-root was not met
    ...

Zdaje się, że [nie tylko ja mam taki
problem](https://lists.debian.org/debian-user/2015/04/msg01423.html).
