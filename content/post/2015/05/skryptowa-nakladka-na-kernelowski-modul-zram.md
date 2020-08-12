---
author: Morfik
categories:
- Linux
date: "2015-05-17T21:01:24Z"
lastmod: 2015-05-17 21:01:24 +0200
published: true
status: publish
tags:
- moduły-kernela
- systemd
title: Skryptowa nakładka na kernelowski moduł ZRAM
---

ZRAM to moduł kernela, który tworzy wirtualne urządzenia w pamięci operacyjnej komputera, a te z
kolei można wykorzystać pod system plików lub też i pod przestrzeń wymiany SWAP. Dane w takim
urządzeniu są kompresowane, dzięki czemu mamy do dyspozycji więcej miejsca w pamięci. Jeśli mamy
niepierwszej jakości dysk twardy, lub też wykorzystujemy szyfrowanie i nie mamy przy tym procka ze
wsparciem dla AES, to operacje zapisu/odczytu mogą zająć wieki. Możemy w dość znacznym stopniu
odciążyć dysk przenosząc pliki do pamięci RAM, który jest o wiele szybszy, no i dane w nim nie
podlegają szyfrowaniu. Jeśli mamy dużo pamięci operacyjnej, to ZRAM raczej będzie zbędny i tylko
podniesie nam rachunek za prąd. Natomiast jeśli nasz komputer nie szasta zbytnio pamięcią, możemy
rozszerzyć ją 2-3 krotnie.

<!--more-->
## Obsługa modułu ZRAM w kernelu

By móc skorzystać z modułu ZRAM, musimy mieć go wkompilowanego w kernel. Ważne jest, by w kernelu
było także wsparcie dla kompresji `lz4` :

    $ egrep -i zram /boot/config-4.7.0-1-amd64
    CONFIG_ZRAM=m
    CONFIG_ZRAM_LZ4_COMPRESS=y

## Projekt zram-init

Na [githubie](https://github.com/vaeth/zram-init/) znalazłem projekt co się zwie `zram-init` . Jest
to skrypt, zajmujący się tworzeniem odpowiednich urządzeń i całość działa OOTB, przynajmniej, gdy
kernel ma ustawiony `LZ4_COMPRESS` . W przeciwnym razie, algorytm kompresji trzeba będzie sobie
zmienić przy tworzeniu urządzeń.

W paczce znajdziemy kilka innych plików, w tym też konfigurację dla modułu ZRAM. Chodzi o ilość
urządzeń, które system będzie tworzył na starcie. Domyślnie są 3: jeden dla SWAP, jeden dla `/tmp/`
i jeden dla `/var/tmp/` . Jeśli nie odpowiada nam taka konfiguracja, to zawsze możemy zmienić ilość
tych urządzeń w pliku `/etc/modprobe.d/zram.conf` . Jeśli chodzi zaś o sam skrypt `zram-init` , to
wgrywamy go do `/usr/sbin/` .

Ja używam systemd, zatem potrzebne mi są również pliki `.service` , które są także dostarczone w
paczce. W przypadku, gdy nasz kernel nie wspiera algorytmu `lz4` , trzeba przepisać sobie linijkę
wywołującą tworzenie urządzeń, tak by ustawić algorytm na `lzo` :

    ExecStart=/usr/sbin/zram-init -s2 -alzo -Lzram_swap 2048

W przypadku, gdy nie odpowiada nam rozmiar urządzenia (tutaj 2 GiB), to również i tę kwestię możemy
sobie dostosować zmieniając ostatnią wartość w powyższej linijce.

Pliki po edycji wrzucamy do `/etc/systemd/system/` , po czym włączamy wszystkie 3 usługi:

    # systemctl enable zram_swap.service
    # systemctl enable zram_tmp.service
    # systemctl enable zram_var_tmp.service
    # systemctl enable zram_btrfs.service

Jako, że wyżej włączyliśmy usługi, to przyszła pora na ich wystartowanie, z tym, że akurat tę
czynność system musi wykonać po zresetowaniu maszyny. Tylko w takim przypadku urządzenia ZRAM
zostaną odpowiednio utworzone, sformatowane i zamontowane.

## Partycja /tmp/

W przypadku korzystania ze `zram_tmp.service` , trzeba zamaskować punkt montowania `tmp.mount` . W
przeciwnym razie mogą wystąpić problemy podczas startu systemu. Maskowanie przeprowadzamy w poniższy
sposób:

    # systemctl mask tmp.mount

Dodatkowo, jeśli mamy jakieś wpisy w pliku `/etc/fstab` dotyczące katalogu `/tmp/` , to trzeba je
wykomentować.
