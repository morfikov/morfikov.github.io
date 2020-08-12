---
author: Morfik
categories:
- Linux
date: "2015-12-02T17:06:40Z"
date_gmt: 2015-12-02 16:06:40 +0100
published: true
status: publish
tags:
- pendrive
- udev
- usb
title: Jak zwiększyć prędkość zapisu w urządzeniach USB
---

Przeglądając sobie [FAQ dotyczący urządzeń USB](http://www.linux-usb.org/FAQ.html) natknąłem się na
punkt, który opisywał parametr `max_sectors` . Niby nic wielkiego, w linux'ie jest przecie pełno
przeróżnych opcji, przy pomocy których jesteśmy w stanie zmienić szereg aspektów pracy naszego
systemu operacyjnego. Rzecz w tym, że parametr `max_sectors` potrafi nawet dość znacznie poprawić
wydajność urządzeń USB, w tym tych wszystkich pendrive'ach, w których prędkość zapisu pozostawia
wiele do życzenia. W tym wpisie postaramy się nieco dostosować ten parametr, tak by przyśpieszyć
transfer kopiowanych plików.

<!--more-->
## Dlaczego max\_sectors jest w stanie zwiększyć prędkość zapisu

Każde urządzenie, które jest w stanie przechowywać jakieś informacje, zapisuje lub odczytuje je w
określonych porcjach. W przypadku pendrive mamy do czynienia z sektorami 512-bajtowymi. Gdyby system
odczytywał porcje 512-bajtowe, transfer by był koszmarnie wolny. Dlatego system stara się czytać
określoną ilość takich sektorów. Im jest ona większa, tym więcej danych można przesłać w jednostce
czasu. Parametr `max_sectors` domyślnie przyjmuje wartość `240` . Zatem w jednym żądaniu, system
jest w stanie przesłać 120KiB danych.

W przypadku nowych urządzeń, w szczególności tych opartych o interfejsy
[USB3](https://pl.wikipedia.org/wiki/Universal_Serial_Bus), ta wartość może okazać się zbyt niska,
przez co osiągnięcie pełnej prędkości zapisu lub odczytu może nie być możliwe. Dlatego też jeśli
naszym zdaniem prędkość zapisu posiadanego przez nas urządzenia USB odbiega od jego parametrów
technicznych, to niekoniecznie powinniśmy od razu obwiniać za to samego producenta sprzętu. Być może
dostosowanie parametru `max_sectors` załatwi sprawę.

## Dostosowanie parametru max\_sectors

Nie wszystkie urządzenia posiadają parametr `max_sectors` . Natomiast wartość tych, które go mają,
może się znacznie różnić. Przede wszystkim, sprawdźmy czy nasz pendrive ma ten parametr. Wsadzamy
nośnik do portu USB i sprawdzamy co zostanie zwrócone po wydaniu tego poniższego polecenia w
terminalu (pamiętajmy o dostosowaniu `sdb`):

    # cat /sys/block/sdb/device/max_sectors
    240

W tym przypadku mamy standardową wartość, czyli `240` sektorów. Czy to dużo czy mało, to trzeba
ustalić przeprowadzając kilka testów. Same testy polegają na mierzeniu czasu zapisu (ewentualnie
odczytu) danych na pendrive przy pomocy `dd` . Poniżej przykładowa
    linijka:

    $ time sh -c "dd if=/dev/zero of=/media/morfik/sdb1-usb-Kingston_DT_101_/01 bs=1M count=1024 && sync"

To powyższe polecenie sprawdzi czas zapisu ciągu zer o długości 1GiB. Tak uzyskamy punkt odniesienia
w stosunku do wartości domyślnych parametru `max_sectors` . Mając prędkość zapisu oraz czas, możemy
zmienić ten parametr przy pomocy tego poniższego polecenia:

    # echo 2048 > /sys/block/sdb/device/max_sectors

Ważne jest by wartość była wielokrotnością 8 lub 16, odpowiednio dla systemów 32 i 64-bitowych. Nie
ma też górnego limitu i możemy tutaj ustawić sobie dowolną wartość. Z tym, że nie ma sensu ustawiać
więcej niż `2048` . Tak czy inaczej poniżej jest test, który porównał wartości 64, 240, 2048 i
32678:

```
 5.1 MB/s       0.00s user 2.51s system 1% cpu 3:32.19 total
 8.4 MB/s       0.00s user 2.38s system 1% cpu 2:11.17 total
10.2 MB/s       0.00s user 2.49s system 2% cpu 1:48.32 total
 9.6 MB/s       0.00s user 2.34s system 2% cpu 1:54.42 total
```

Prędkość zapisu dla tego pendrive w standardowych warunkach waha się w granicach 8M/s. Po
przestawieniu `max_sectors` na 2048, prędkość zapisu skoczyła do ponad 10M/s. Jest zatem dość spora
różnica, bo prawie 2M/s, a to piechotą nie chodzi.

## Reguła dla udev'a

Powyższe ustawienia są jedynie tymczasowe i znikną po tym jak odłączymy pendrive od portu USB. By
parametr `max_sectors` domyślnie przybrał wartość 2048, musimy pokusić się o napisanie [reguły dla
udev'a]({{< baseurl >}}/post/udev-czyli-jak-pisac-reguly-dla-urzadzen/). Tworzymy wobec tego plik
`90-pendrive-max-sectors.rules` w katalogu `/etc/udev/rules.d/` i dodajemy w nim tę poniższą treść:

    KERNEL=="sd?", SUBSYSTEMS=="usb", DRIVERS=="usb-storage" \
         RUN+="/bin/sh -c 'echo 2048 > /sys/block/%k/device/max_sectors'"

Trzeba jednak mieć na względzie, że ta reguła przestawi parametr `max_sectors` wszystkim urządzeniom
USB, co nie zawsze jest pożądane. Możemy ograniczyć jej zakres przez przepisanie jedynie wartości
domyślnej, która wynosi `240` . W ten sposób tylko pewne określone urządzenia będą miały ustawiane
tę większą wartość, co powinno uchronić nas przed negatywnymi skutkami w pewnych bardzo
specyficznych sytuacjach. Dlatego lepszym wyborem jest dodanie tej reguły:

    KERNEL=="sd?", SUBSYSTEMS=="scsi", DRIVERS=="sd", ATTRS{max_sectors}=="240" \
          RUN+="/bin/sh -c 'echo 2048 > /sys/block/%k/device/max_sectors'"

Zapisujemy plik i przeładowujemy bazę udeva przy pomocy:

    # udevadm config --reload

Odłączamy jeszcze urządzenie od komputera i wsadzamy je ponownie, by sprawdzić czy faktycznie
parametr `max_sectors` został przepisany z powodzeniem.
