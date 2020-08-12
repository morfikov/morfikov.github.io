---
author: Morfik
categories:
- Linux
date: "2015-06-17T20:59:18Z"
date_gmt: 2015-06-17 18:59:18 +0200
published: true
status: publish
tags:
- system-plików
- hdd/ssd
- ext4
title: Sprawdzanie błędów systemu plików ext4
---

Systemy plików stosuje się dla różnych nośników danych, takich jak dyski twarde, czy pendrive albo
nawet płyty cd/dvd. Z formalnego punktu widzenia, system plików jest to metoda przechowywania danych
i uzyskiwania do nich dostępu. Bez tego mechanizmu, informacje umieszczone na nośniku przypominały
by jedynie ciąg bitów i nie wiedzielibyśmy gdzie zaczyna się jakiś plik i gdzie się on kończy.
Czasami jednak zdarzają się błędy w systemie plików, które mogą doprowadzić do poważnych awarii
systemu operacyjnego. Dlatego też linux co kilkanaście lub kilkadziesiąt uruchomień sprawdza stan
systemu plików na każdej partycji i naprawia ewentualne błędy. W przypadku gdyby nie były one
naprawiane, mogą pojawić się nowe błędy doprowadzając tym samym do całkowitej zapaści systemu.

<!--more-->
## Rutynowe sprawdzanie plików

Jako, że sprawdzanie systemu plików to podstawowa czynność, którą administratorzy powinni
przeprowadzać w miarę regularnie, mamy do dyspozycji kilka narzędzi w zależności od tego z jakim
systemem plików przyjdzie nam pracować. W tym artykule skupię się głównie na domyślnym systemie
plików stosowanym w linuxach, tj. `ext4` . Jeśli chodzi o ten system plików, to narzędzia
przeznaczone do jego obsługi znajdują się w pakiecie [e2fsprogs](http://e2fsprogs.sourceforge.net) ,
który to umożliwia tworzenie, sprawdzanie i utrzymywanie systemu plików ext2/ext3/ext4 . Paczka
zawiera także program `badblocks` , który jest w stanie przeskanować nośnik w poszukiwaniu
uszkodzonych sektorów. Nas jednak najbardziej będzie interesować `e2fsck` , przy pomocy, którego
będziemy sprawdzać dysk w poszukiwaniu błędów.

Do systemu plików możemy odwoływać się na kilka sposobów. Zwykle będziemy to robić przez urządzenia
w katalogu `/dev/` lub `/dev/disk/by-*` lub też przez punkty montowania, np. `/home/` . Warto
wiedzieć też, że w celu skrócenia łącznego czasu potrzebnego do sprawdzenia wszystkich systemów
plików, program `fsck` będzie usiłował sprawdzać równolegle systemy plików umieszczone na fizycznie
różnych dyskach.

Skanowanie systemu plików możemy zainicjować ręcznie lub też może się ono odbywać automatycznie, np.
co kilka montowań partycji, przy starcie systemu operacyjnego. Możemy też je całkowicie wyłączyć.

Za pomocą narzędzia `dumpe2fs` możemy zobaczyć szczegółowe statystyki systemu plików dla konkretnej
partycji. Nas głównie interesują dwie pozycje:

    # dumpe2fs /dev/sda2 | grep -i "mount count"
    dumpe2fs 1.42.13 (17-May-2015)
    Mount count:              528
    Maximum mount count:      -1

W tym przypadku, ostatnia linijka informuje nas, że skanowanie tej partycji w poszukiwaniu błędów
zostało z jakiegoś powodu wyłączone, co możemy wnioskować także po liczbie w `Mount count` , która
oznacza liczbę montowań, czyli ile razy zamontowaliśmy już tą partycję. Przydałoby się włączyć
skanowanie dla tego systemu plików. By określić co ile montowań systemu plików ma być dokonywane
sprawdzanie integralności danych, posłużymy się programem `tune2fs` z opcją `-c` , w argumencie
podając liczbę, która oznacza ilość montowań. W poniższym przykładzie, licznik został ustawiony na
30:

    # tune2fs -c 30 /dev/sda2

Jeśli jeszcze raz zajrzymy w statystyki systemu plików, dostrzeżemy, że pozycja z `Maximum mount
count` uległa przepisaniu z `-1` na `30` i jest to graniczna wartość, po osiągnięciu której nastąpi
skanowanie systemu plików. W tym przypadku po 30 montowaniach nastąpi zresetowanie licznika w `Mount
count` i zostanie zainicjowany program `fsck` dla tej partycji, czyli w tym przypadku, po
zresetowaniu maszyny.

Gdybyśmy zamiast `30` wstawili `0` , skanowanie zostanie trwale wyłączone. Nie jest to jednak
zalecane, gdyż może się to przyczynić do utraty danych. Jeżeli jednak nie odpowiada nam, że w
najmniej oczekiwanym momencie licznik montowań akurat wybił 30 i zostaje automatycznie uruchomione
skanowanie systemu plików, możemy ustawić parametr na 0 i ręcznie dokonywać sprawdzania systemu
plików co określony przedział czasu.

## Sterowanie fsck przy pomocy fstab

Sprawdzanie systemu plików, możemy wyłączyć też za pomocą pliku `/etc/fstab` . Poniżej znajduje się
przykładowy wpis z partycją:

    # file system   mount point   type   options                      dump  pass
    /dev/sda2      /boot          ext4   defaults,errors=remount-ro   0     2

Ostatni parametr `pass` określa kolejność sprawdzania systemu plików. Może przyjąć trzy wartości:
`0` , `1` oraz `2` . Z reguły `1` jest ustawiany dla głównego systemu plików, a `2` dla każdego z
pozostałych. Wartość `0` oznacza, że partycja nie będzie sprawdzana.

## Wymuszenie skanowania

Jeśli chodzi zaś o ręczne skanowanie systemu plików, to możemy go dokonać na kilka sposobów.
Pierwszym z nich jest zresetowanie lub zamknięcie systemu z opcją `-F`:

    # shutdown -Fr now
    # shutdown -Fh now

Parametr `-r` oznacza reboot (zresetuj), zaś parametr `-h` halt (wstrzymaj). Po ponownym
uruchomieniu komputera nastąpi sprawdzenia systemu plików.

W przypadku gdy chcemy zaplanować skanowanie systemu plików przy następnym resecie komputera ale nie
uśmiecha nam się resetować go w tym konkretnym momencie, możemy utworzyć plik `forcefsck` w głównym
katalogu `/` :

    # touch /forcefsck

Plik `forcefsck` jest kasowany przy starcie systemu operacyjnego, chwilę po zakończeniu sprawdzania
integralności plików. Możemy także dodać atrybut `+i` dla tego pliku:

    # chattr +i /forcefsck

Dzięki temu atrybutowi, nawet użytkownik root nie będzie w stanie tego pliku skasować i system
będzie sprawdzał wszystkie wpisy zdefiniowane w `/etc/fstab` przy każdym uruchomieniu komputera.

## Skanowanie zaszyfrowanych partycji

W przypadku skanowania zaszyfrowanych partycji musimy pierw odszyfrować ową partycję, a dopiero
później zeskanować jej system plików. Problematyczne może być znalezienie dysku (lub partycji),
który chcemy poddać skanowaniu. Zaszyfrowane partycje pod linuxem wykrywane są jako `/dev/dm-*` , z
kolei w katalogu `/dev/mapper/` możemy odnaleźć linki do tych powyżej i one powinny mieć już
bardziej ludzkie nazwy. Gdy ustalimy już co chcemy skanować, robimy to poniższym poleceniem:

    # fsck.ext4 -nvf /dev/mapper/debian_laptop-root

Użyte parametry odpowiadają za:

  - `-n` -- nie dokonuje zmian w systemie plików (tryb tylko do odczytu)
  - `-f` -- wymusza sprawdzenie nawet czystego systemu plików, czyli takiego, który pozornie nie
    zawiera błędów
  - `-v` -- pokazuje większą ilości informacji

W przypadku dokonywania skanu w trybie tylko do odczytu, partycja może być podmontowana, w
przeciwnym wypadku musi zostać odmontowana, gdyż naprawa podmontowanego systemu plików grozi utratą
danych.
