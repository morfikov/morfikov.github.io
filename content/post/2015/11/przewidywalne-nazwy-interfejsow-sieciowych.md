---
author: Morfik
categories:
- Linux
date:    2015-11-22 21:44:11 +0100
lastmod: 2022-01-02 15:00:00 +0100
published: true
status: publish
tags:
- udev
- systemd
- sieć
GHissueID: 263
title: Przewidywalne nazwy interfejsów sieciowych
---

Podczas jednej z aktualizacji systemu został mi zwrócony pewien komunikat. Oświadczał on bowiem, że
od jakiegoś czasu nazewnictwo interfejsów sieciowych w systemie uległo zmianie, oraz, że w wersji 10
debiana, ten obecny system nazw nie będzie już wspierany. Rozchodzi się o coś co nazywa się
[Predictable Network Interface Names][1], czyli przewidywalne nazwy interfejsów sieciowych. Jako,
że aktualne wydanie stabilnego debiana ma numerek 8 i w niedalekiej przyszłości zostanie wydana 9,
to przydałoby się już zacząć migrować na ten nowy system nazw. W tym wpisie dokonamy takiej
migracji i zobaczymy jakie zmiany musimy poczynić, by nie doświadczyć problemów związanych z tą
migracją nazw.

<!--more-->
## Nazwy interfejsów sieciowych i problemy z nimi związane

Wszystkie narzędzia sieciowe, począwszy od `ifconfig` czy `ip` skończywszy na zaporach sieciowych,
np. `iptables` , operują na interfejsach sieciowych. Zwykle mamy `eth*` dla połączeń przewodowych i
`wlan*` dla bezprzewodowych. Są oczywiście i inne ale problem dotyczy głównie tych wymienionych
wyżej.

Co się dzieje w przypadku gdy podłączamy do komputera nową kartę sieciową? Po jej wykryciu system
nada jej odpowiedni numerek interfejsu. Jaki numerek zapytacie? No i to jest właśnie dobre pytanie i
nikt na dobrą sprawę nie zna na nie odpowiedzi. Może się zatem zdarzyć tak, że mając dwie karty
sieciowe, ich interfejsy się zamienią. By przeciwdziałać takim niepożądanym zmianom nazw, został
obmyślony mechanizm przepisywania ich przy pomocy `udev` . Jeśli zajrzymy do katalogu
`/etc/udev/rules.d/` , to mamy w nim min. plik `70-persistent-net.rules` . W nim jest kilka regułek,
które zależą od tego jaki sprzęt podpinaliśmy do naszego komputera. Poniżej przykład takiej
    reguły:

    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:e0:4c:75:13:09", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

W skrócie, karcie sieciowej mającej adres MAC `00:e0:4c:75:13:09` zostanie przypisany interfejs
`eth0` . Jeśli byśmy podłączyli drugą kartę sieciową, to będzie ona miła inny numerek niż ma
`eth0` , prawdopodobnie będzie to `eth1` , itd. W taki sposób, mamy mniej więcej pewność, że każda
karta ma stały interfejs.

Co zatem jest złego w tym sposobie przepisywania nazw, przecież mamy przewidywalne nazwy
interfejsów? Co jest nie tak z tym mechanizmem, że trzeba go zastąpić nowym? Odpowiedź możemy
znaleźć czytając plik `/usr/share/doc/udev/README.Debian.gz` , w którym mamy informację, że w
pewnych sytuacjach mogą wystąpić kolizje nazw interfejsów, w wyniku których mogą się pojawić nazwy
takie jak `rename1` . By temu przeciwdziałać, trzeba zapisywać pewne informacje w katalogu `/etc/` ,
co nie jest możliwe gdy partycja `/` jest w trybie tylko do odczytu. Poza tym ten mechanizm nie
działa w ogóle w środowiskach wirtualnych. W pliku `README.Debian.gz` mamy także instrukcję
dotyczącą przeprowadzenia migracji nazw interfejsów sieciowych, zatem do dzieła.

## Migracja na nowy system nazw

Może i zostało nam jeszcze trochę czasu do momentu, gdy debian przestanie wspierać stare nazwy
interfejsów sieciowych ale po co czekać do samego końca? Przeprowadzając migrację nazw w tym
momencie oszczędzimy sobie sporo nerw związanych z robieniem wszystkiego na ostatnią chwilę.

Na początek musimy ustalić jakie interfejsy mamy w swoim systemie. Możemy tego dokonać na kilka
sposobów. Najprościej jest zwyczajnie zajrzeć do pliku `/etc/udev/rules.d/70-persistent-net.rules` ,
no bo przecie są tam reguły dotyczące wszystkich kart sieciowych, które kiedykolwiek podpięliśmy do
naszej maszyny. Możemy także zajrzeć do katalogu `/sys/class/net/` lub zwyczajnie podejrzeć aktualne
interfejsy via `ip link` czy `ifconfig` .

Teraz musimy ustalić w jakich plikach konfiguracyjnych w katalogu `/etc/` mamy odwołania do tych
poszczególnych interfejsów. Na pewno jest to konfiguracja sieciowa ale oprócz tego, mogą to być
także skrypty startowe, czy też reguły iptables. Nie bez znaczenia może być także zwykła
konfiguracja poszczególnych aplikacji. By ustalić, które pliki wymagają zmian, logujemy się na konto
root i wydajemy w terminalu to poniższe polecenie dostosowując je pod kątem nazwy interfejsu:

    # grep -r eth0 /etc/

Wszystkie te pliki, które zostaną wypisane, trzeba będzie odpowiednio przerobić. Oczywiście
zwracajmy uwagę na komentarze. Jest tylko jeden problem, mianowicie nie wiemy jakie będą nowe nazwy
interfejsów. By to ustalić musimy usunąć plik z regułami udev'a (
`/etc/udev/rules.d/70-persistent-net.rules` ). Dodatkowo na wszystkich maszynach wirtualnych musimy
także usunąć plik `/etc/udev/rules.d/80-net-setup-link.rules` . Regenerujemy teraz initramfs przy
pomocy tego poniższego polecenia:

    # update-initramfs -u -k all

Po czym uruchamiamy ponowne komputer. Po załadowaniu się systemu, dostosowujemy całą konfigurację w
oparciu o nowe nazwy interfejsów sieciowych. W moim przypadku pojawiły się `enp2s0` (przewodowy) i
`wlp3s0b1` (bezprzewodowy). Jeśli mamy obecny w systemie pakiet `ifrename` , to możemy go bez
problemu odinstalować.

## Nowe nazwy interfejsów

W stosunku do starych nazw, tj. `eth0` czy `wlan1` , nazwy takie jak `wlp3s0b1` mogą nieco
przytłoczyć. Wyjaśnienie struktury tych nazw znajduje się [tutaj][2]. Jeśli komuś się nie chce
zaglądać do przytoczonego linka, to warto nadmienić, że interfejsy przewodowe zawsze zaczynają się
od `en*` , a bezprzewodowe od `wl*`. Pozostała część nazwy będzie stała dla konkretnej maszyny.

Warto też dodać, że jest kilka schematów nadawania nazw interfejsom sieciowym. Możemy wyróżnić nazwy
w oparciu o fizyczne położenie ( `path` ). Są też nazwy interfejsów w oparciu o adres MAC ( `mac` )
czy też nazwy zwracane przez sterownik karty ( `onboard` ). [Naturalnie jest jeszcze więcej polityk
nadawania nazw][3] ale zwykle z tymi powyższymi typami nazw będziemy mieli największą styczność.

Wszystkie aktualnie rozpoznane interfejsy w systemie są do wglądu w katalogu `/sys/class/net/` .
Weźmy sobie przykładowo interfejs `wlan1` . Jego alternatywne nazwy są przechowywane w zmiennych
udev'a i w zależności od wybranej polityki są one aplikowane. Poniżej znajdują się przykładowe nazwy
dla tego interfejsu:

    $  udevadm info /sys/class/net/wlan1
    ...
    E: ID_NET_NAME=wlan1
    E: ID_NET_NAME_MAC=wlxe8de271d4ca5
    E: ID_NET_NAME_PATH=wlp0s29u1u3

Nazwy oparte o MAC i PATH nie są zbytnio przyjazne użytkownikowi ale to nie stanowi praktycznie
żadnego problemu, bo nikt nas nie zmusza do korzystania z nich. Nazwę praktycznie każdego
interfejsu jesteśmy w stanie bez problemu przepisać na dowolną. By tego dokonać, musimy stworzyć
sobie plik `/etc/systemd/network/05-wlan1.link` i dodać w nim tę poniższą treść:

    [Match]
    MACAddress=e8:de:27:1d:4c:a5

    [Link]
    Description=Wireless interface configuration (wlan1)
    Name=wlan1

W tym przypadku mamy do czynienia z kartą USB, dlatego lepiej nie korzystać ze ścieżki, bo ta może
ulec zmianie. Można natomiast wykorzystać adres MAC tego urządzenia, bo ten zwykle jest
niepowtarzalny i stały dla konkretnej karty.

## Parametry net.ifnames= oraz biosdevname=

W nowszych wydaniach dystrybucji Debian lepszym rozwiązaniem zdaje się być skorzystanie z parametru
`net.ifnames=` lub `biosdevname=` . Jaka jest różnica między tymi dwiema opcjami?

Parametr `biosdevname=` jest przeznaczony do wykorzystania wszędzie tam gdzie jest
zainstalowane [stosowne oprogramowanie od deweloperów DELL'a][4] mające na celu wspomagać UDEV'a
przy przepisywaniu nazw interfejsów sieciowych, tak by były one konsekwentne i niezmienne w czasie.

Niemniej jednak, w Debianie nie ma tego oprogramowania i na dobrą sprawę nie jest ono potrzebne,
przynajmniej tam, [gdzie mamy do czynienia z systemd][5]. Systemd posiada własny mechanizm
przepisywania nazw interfejsów sieciowych i do jego kontroli służy właśnie ten drugi z powyżej
wymienionych parametrów, tj. `net.ifnames=` . Domyślnie mechanizm przepisywania tych nazw jest
włączony i dlatego na świeżo postawionym systemie nie uświadczymy interfejsów typu `wlan0` czy
`eth0` .

Jeśli nie podobają nam się nowe nazwy interfejsów sieciowych albo też zwyczajnie ich nie
potrzebujemy, bo nasz system posiada tylko jeden interfejs przewodowy i bezprzewodowy, to dla
uproszenia możemy ten mechanizm przepisywania nazw wyłączyć dopisując w konfiguracji bootloader'a
(np. GRUB) `net.ifnames=0` (ten drugi parametr można pominąć).


[1]: https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/
[2]: https://github.com/systemd/systemd/blob/master/src/udev/udev-builtin-net_id.c#L20
[3]: https://www.freedesktop.org/software/systemd/man/systemd.link.html
[4]: https://linux.dell.com/files/biosdevname/
[5]: https://man7.org/linux/man-pages/man8/systemd-udevd.service.8.html
