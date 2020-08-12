---
author: Morfik
categories:
- Linux
date: "2015-10-15T17:02:25Z"
date_gmt: 2015-10-15 15:02:25 +0200
published: true
status: publish
tags:
- zmienne-środowiskowe
title: Zmienna TZ w środowisku linux'owym
---

Ten wpis będzie poświęcony poprawie wydajności naszych systemów linux'owych. Okazuje się bowiem, że
w pewnych aspektach ich pracy [nie wszystko działa jak
należy](http://www.brendangregg.com/blog/2014-05-11/strace-wow-much-syscall.html). Rozchodzi się o
zmienną środowiskową `TZ` , w oparciu o którą to zwracane są, np. czasy utworzenia czy modyfikacji
plików na dysku. Ten przypadek jest o tyle dziwny, że każdy z nas ma już na swojej maszynie
skonfigurowany czas systemowy, za który odpowiada pakiet `tzdata` . Niby wyraźnie określiliśmy
strefę czasową, w tym przypadku jest to `Europe/Warsaw` ale widać system operacyjny posiłkuje się
odwołaniami do pliku `/etc/localtime` .

<!--more-->
## Plik /etc/localtime , a zmienna TZ

Cóż takiego znajduje się w tym pliku `/etc/localtime` , że system nie może bez niego żyć i ciągle go
odpytuje? Zgodnie z [manualem
localtime(5)](https://www.freedesktop.org/software/systemd/man/localtime.html) są tam trzymane
globalne ustawienia systemowe dotyczące strefy czasowej, a właściwie jest to zwykłe dowiązanie
symboliczne do jakiegoś pliku w katalogu `/usr/share/zoneinfo/` , który wskazuje na odpowiednią
strefę czasową, z której mają korzystać rozmaite aplikacje zwracające nam pewne dane, tak jak to
robi, np. `ls` przy listowaniu plików w danym katalogu.

Problem w tym, że system się ciągle odwołuje do tego pliku spowalniając tym samym wykonywanie
pewnych operacji. Możemy się o tym przekonać przeprowadzając pewne krótkie doświadczenie polegające
na wpisaniu tych dwóch poniższych linijek do terminala i porównaniu czasu, jaki zostanie zwrócony:

    $ time ls -l `perl -e 'print "/etc " x 1000'` >/dev/null
    $ time TZ=Europe/Warsaw ls -l `perl -e 'print "/etc " x 1000'` >/dev/null

Te dwie linijki nie robią nic innego jak tylko listują w kółko zawartość katalogu `/etc/` . Po
zakończeniu tego eksperymentu, czasy wykonania tych poleceń wskazywały odpowiednio:

    1.11s user 1.34s system 99% cpu 2.461 total
    0.87s user 1.15s system 99% cpu 2.031 total

Jak widać, gdy w grę wchodzi zmienna `TZ` , możemy odczuć drastyczny wzrost wydajności rzędu ponad
20%. Dla przeciętnego użytkownika raczej efekt będzie mało odczuwalny ale zawsze to jest mniej
odwołań do dysku, który przecie jest najwęższym gardłem w procesie przesyłu informacji w obrębie
danej maszyny.

## Ustawienie zmiennej TZ

W powyższym doświadczeniu, zmienna `TZ` została ustawiona jedynie dla pojedynczego polecenia.
Przydałoby się ustawić ją globalnie dla całego systemu, tak by procesy nie musiały się ciągle
odwoływać do pliku `/etc/localtime` . Możemy to zrobić na kilka sposobów.

### Plik .bashrc/.zshrc

Pierwszym wyjściem wyeksportowanie zmiennej `TZ` za pomocą pliku konfiguracyjnego shell'a, z którego
korzystamy. Robimy to przez dopisanie poniższej linijki, np. do pliku `~/.bashrc` :

    export TZ="Europe/Warsaw"

Trzeba jednak pamiętać, że w takim przypadku zmiany nie będą dotyczyć innych shell'i i na dobrą
sprawę trzeba by tę powyższą linijkę dopisać do szeregu plików.

### Zmienna TZ w środowisku graficznym

Środowiska graficzne czy osamotnione menadżery okien, np. Openbox, rządzą się innymi prawami i mają
oddzielne pliku konfiguracyjne, w których to możemy określić odpowiednie zmienne. W przypadku
Openbox'a jest to plik `~/.config/openbox/environment` , gdzie również trzeba by wyeksportować
zmienną `TZ` i to dokładnie w taki sam sposób co w przypadku konfiguracji shell'a.

### Plik /etc/environment

Istnieje także plik `/etc/environment` , w którym możemy wyeksportować zmienne globalne, które będą
uwzględniane przez wszystkie shell'e, każdego użytkownika w systemie oraz przez środowiska
graficzne. [Nie jest to jednak skrypt](https://help.ubuntu.com/community/EnvironmentVariables) i w
nim zmienne definiujemy nieco inaczej, mianowicie w poniższy sposób:

    TZ="Europe/Warsaw"
