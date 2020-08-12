---
author: Morfik
categories:
- Linux
date: "2015-09-06T11:08:42Z"
date_gmt: 2015-09-06 09:08:42 +0200
published: true
status: publish
tags:
- moduły-kernela
- udev
- sysctl
- systemd
title: Aplikowanie zmiennych sysctl przy pomocy udev'a
---

Kernele linux'owe mają dość sporo opcji, które możemy zmienić przy pomocy pliku `/etc/sysctl.conf` .
Niby nic nadzwyczajnego ale co w przypadku tych zmiennych, które muszą być ustawione, z tym, że
moduł, który stworzy odpowiednie ścieżki w katalogu `/proc/sys/` , nie został załadowany z jakichś
względów przy starcie systemu? Zmienne te nie zostaną ustawione, a w logu pojawi się komunikat
informujący nas o nieodnalezieniu określonego pliku. Okazuje się, że jesteśmy w stanie aplikować
określone ustawienia sysctl w momencie ładowania określonych modułów i temu mechanizmowi się
przyjrzymy bliżej w tym wpisie.

<!--more-->
## Komunikaty zwracane przez systemd-sysctl

Przeglądając log systemowy, doszukałem się kilku komunikatów, których treść jest podobna do tej
poniższej linijki::

    # journalctl --no-tail -b -u systemd-sysctl
    ...
    systemd-sysctl[430]: Couldn't write '30' to 'net/netfilter/nf_conntrack_icmpv6_timeout', ignoring: No such file or directory
    ...

Problem udało się rozwiązać i winne były dwa moduły, konkretnie `nf_conntrack_ipv4` oraz
`nf_conntrack_ipv6` , które nie ładowały się przy starcie systemu -- no bo nie było ich wpisanych do
pliku `/etc/modules` . Te dwa moduły były ładowane w późniejszej fazie startu systemu ale
ustawienia, które powinny zostać zaaplikowane, zostały zwyczajnie zignorowane.

Możemy naturalnie wpisać wszystkie niezbędne moduły (te, których wartości są określone w pliku
`sysctl.conf`) do pliku `/etc/modules` . Chodzi o to, że w systemd mamy dwie usługi odpowiedzialne
za ustawianie zmiennych kernela: `systemd-modules-load.service` , która ładuje moduły oraz
`systemd-sysctl.service` , która nakłada ustawienia z pliku `sysctl.conf` . Druga z nich ma wyraźnie
określoną zależność `After=systemd-modules-load.service` , dlatego też wszystkie moduły muszą być
pierw załadowane, by te zmienne mogły zostać zaaplikowane. Nie zawsze jest to jednak idealne
rozwiązanie.

## Reguły udev'a dla określonych modułów

Co w przypadku gdy nie chcemy ładować wszystkich modułów przy starcie systemu? Przecie część z nich
możemy ładować w trakcie jego pracy, oczywiście, jeśli zachodzi taka potrzeba. W takim przypadku
musimy skorzystać z udev'a i napisać regułkę, która w chwili ładowania takiego modułu ustawi
określone dla niego opcje. [W manualu
sysctl.d](https://www.freedesktop.org/software/systemd/man/sysctl.d.html) jest przykład takiej
reguły i ja na jej podstawie stworzyłem plik `/etc/udev/rules.d/99-sysctl.rules` o poniższej
treści:

    ACTION=="add", SUBSYSTEM=="module", KERNEL=="nf_conntrack", \
          RUN+="/lib/systemd/systemd-sysctl --prefix=/net/netfilter"
    
    ACTION=="add", SUBSYSTEM=="module", KERNEL=="nf_conntrack_ipv4", \
          RUN+="/lib/systemd/systemd-sysctl --prefix=/net/netfilter"
    
    ACTION=="add", SUBSYSTEM=="module", KERNEL=="nf_conntrack_ipv6", \
          RUN+="/lib/systemd/systemd-sysctl --prefix=/net/netfilter"

Opcja `ACTION` oraz `SUBSYSTEM` zawsze pozostaje bez zmian. Z kolei na pozycji `KERNEL` wpisujemy
nazwę modułu, tego, który zwykle ładowany jest przy pomocy `modprobe` lub też może zostać
wyświetlony via `lsmod` . W `RUN` następuje wywołanie odpowiedniego narzędzia dostarczanego przez
systemd z opcją `--prefix`, która odpowiada za lokalizację plików modułu w katalogu `/proc/sys/` . W
tym przypadku, konfiguracja powyższych modułów znajduje się pod `/proc/sys/net/netfilter/` , zatem
wszystkie wpisy w `/etc/sysctl.conf` odwołujące się do `net.netfilter` będą aplikowane przy
ładowaniu tych modułów.
