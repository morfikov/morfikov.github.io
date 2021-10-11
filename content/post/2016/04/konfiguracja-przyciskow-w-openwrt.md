---
author: Morfik
categories:
- OpenWRT
date: "2016-04-30T20:00:19Z"
date_gmt: 2016-04-30 18:00:19 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
GHissueID: 433
title: Konfiguracja przycisków w OpenWRT
---

Każdy router w standardzie ma na swojej obudowie kilka przycisków. Zwykle są to przyciski zasilania,
restartu i przełącznik sieci bezprzewodowej. Różnie są one oznaczane i można się spotkać z QSS, WPS,
reset czy WiFi. O ile z przyciskiem zasilania nic więcej się nie da zrobić, bo po naciśnięciu go
router jest odcinany od źródła zasilania ale w przypadku pozostałych przycisków mamy zwykle pełną
swobodę w ich konfiguracji i można je sobie odpowiednio zaprogramować. Oczywiście trzeba widzieć jak
tego dokonać i dlatego właśnie powstał ten artykuł.

<!--more-->
## Wsparcie dla przycisków w OpenWRT

Przyciski na routerach dzielimy z grubsza na trzy rodzaje. Mamy włącznik/wyłącznik, zwykle tego typu
jest przycisk zasilania. Możemy też mieć przełącznik pozycyjny i ten często jest stosowany w
przypadku WiFi. Mamy też zwykły przycisk, który się wciska, a po puszczeniu go wraca on do pozycji
wyjściowej. Biorąc pod uwagę powyższe informacje, z reguły tylko ten ostatni typ przycisku jest
wspierany przez OpenWRT. Choć ostatnio przełączniki też działają. Jeśli posiadamy taki przełącznik,
to mamy spore prawdopodobieństwo, że mogą być z nim problemy pod tym firmware. Oczywiście nie należy
mylić braku wsparcia obsługi danego przycisku z brakiem wykonania się przypisanej mu akcji. W
przypadku, gdy mamy do czynienia z problemem czysto programowym, to raczej powinniśmy być w stanie
wyeliminować go bez trudu. Dla przykładu, router [TP-LINK TL-WR1043N/ND
v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) ma dwa przyciski, oba wciskane i
oba z nich działają bez zarzutu. Z kolei mam też inny router, tj. [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html), i w przeszłości oba jego przyciski
nie działały. Obecnie nie ma z nimi żadnego problemu.

W przypadku, gdy nie możemy w prosty sposób ustalić czy przycisk jest aktywny i działa, to możemy
zobrazować sobie akcje wciskania przycisków. Chodzi generalnie o to, że każde naciśnięcie przycisku
(i jego puszczenie) jest wykrywane przez router. Jeśli przycisk działa, takie zdarzenie wciśnięcia
powinno zostać zarejestrowane w logu systemowym. By tego typu informacje uzyskać od routera, musimy
do katalogu `/etc/hotplug.d/button/` wgrać plik `/etc/hotplug.d/button/buttons` o poniższej treści
(należy utworzyć zarówno katalog jak i plik):

    #!/bin/sh
    logger $BUTTON
    logger $ACTION

Jako, że utworzony przez nas plik jest skryptem shell'owym, to musimy nadać mu odpowiednie
uprawnienia ( `chmod +x` ). Teraz możemy bez problemu zweryfikować czy przyciski reagują na
naciśnięcie czy też są martwe i nic więcej z nimi się nie da zrobić. Poniżej przykład z routera
TL-WR1043ND:

![](/img/2016/04/1.test-dzialanie-przyciskow-openwrt-router.png#huge)

Oczywiście w sytuacji, gdy router fizycznie reaguje na naciśnięcie przycisku, tak jak w tym
przypadku została wyłączona sieć WiFi, to naturalnie bez problemu jesteśmy w stanie wywnioskować, że
przycisk działa. Poza tym, mamy w logu też stosowną informacje. Ten powyższy skrypt dodał do tego
logu pierwsze cztery linijki, które widzimy na powyższej fotce. Dlatego też, jeśli przycisk pozornie
nie reaguje na przyciśnięcie ale w logu pojawia się nazwa tego przycisku + `pressed` (przyciśnięty)
i `released` (puszczony), to oznacza to nic innego jak tylko tyle, że OpenWRT potrafi na nim
operować oraz, że mamy do czynienia jedynie z błędem programistycznym. A ten z kolei możemy sami
wyeliminować.

## Konfiguracja przycisków routera

Pora przejść do konfiguracji przycisków. Głównie interesować nas będzie katalog `/etc/rc.button/` .
To w nim jest trzymana cała konfiguracja przycisków. Mamy tam kilka plików o nazwach, które zwykle
powinny wskazywać na tę nazwę, którą zwrócił nam w logu powyższy skrypt. I jeśli popatrzymy uważnie,
to w przypadku tego modelu routera mamy tam plik `rfkill` . Wszystkie pliki w katalogu
`/etc/rc.button/` to zwyczajnie skrypty shell'owe i przy naciśnięciu przycisku jest po prostu
wywoływany konkretny skrypt.

Najczęstszą przyczyną niedziałania wykrywanych przez OpenWRT przycisków jest nieprawidłowa nazwa
pliku ze skryptem. W przypadku tego Archer'a C7, po przyciśnięciu przycisku była zwracana nazwa
`wps` . Niemniej jednak, w katalogu `/etc/rc.button/` nie było takiego pliku. Rozwiązaniem było
utworzenie go i dodanie w nim stosownych akcji. Bywa też tak, że inny skrypt ma już w sobie
zdefiniowane procedury, np. restartu urządzenia. W taki sposób można jedynie utworzyć link
wskazujący na ten skrypt.

Jako, że mamy do czynienia ze zwykłymi skryptami, to możemy każdy przycisk sobie dowolnie
zaprogramować. Mało tego, możemy do takiego przycisku przypisać więcej niż jedną akcję. Te z kolei
mogą być aktywowane w zależności od stanu przycisku (przyciśnięty lub puszczony) oraz na podstawie
czasu trzymania takiego przycisku w pozycji przyciśniętej. Wszystkie te informacje są zwracane przez
`logread` . Poniżej przykład trzymania wciśniętego przycisku `reset` przez 3 sekundy:

![](/img/2016/04/2.test-przyciskow-czas-openwrt-router.png#huge)

Oczywiście urządzenie nie zresetowało się, bo akurat ten przycisk ma przypisane dwie akcje, z
których żadna nie spełnia warunku przyciśnięcia przez 3 sekundy.

## Określanie zachowania przycisków

Omówmy zatem sobie przykładowy plik z katalogu `/etc/rc.button/` . Niech będzie ten powyżej
wykorzystany `reset` . Zajrzymy do niego. Mamy tutaj mniej więcej taki kod:

![](/img/2016/04/3.akcje-przypisane-do-przyciskow-openwrt-router.png#medium)

Na samym początku mamy warunek `[ "${ACTION}" = "released" ]` . Czyli zawartość tego skryptu
zostanie przetworzona po puszczeniu przycisku. [Czas trzymania przycisku jest
zapisywany](https://wiki.openwrt.org/doc/howto/hardware.button) w zmiennej `$SEEN` . W oparciu o tę
zmienną mamy dwa warunki: `[ "$SEEN" -lt 1 ]` oraz `[ "$SEEN" -gt 5 ]` . Jest to zwykłe porównanie
wartości zapisanej w zmiennej z tą, którą sobie ustawimy. W ten sposób pierwszy warunek dotyczy
sytuacji, gdy przycisk był trzymany przez mniej niż 1 sekundę. Natomiast drugi warunek jest
spełniony, gdy przycisk był trzymany przez ponad 5 sekund. W każdym warunku mamy szereg poleceń,
które zostaną wykonane w przypadku, gdy warunek będzie spełniony.

Wracając do powyższego przykładu, już raczej wiadomo dlaczego router ani się nie zresetował, ani nie
wykonał żadnej akcji. Niemniej jednak, przycisk `reset` ma przypisane dwie akcje. Pierwsza to
resetowanie urządzenia, druga zaś to [przywracanie ustawień fabrycznych (factory reset,
firstboot)](/post/reset-ustawien-w-openwrt-firstboot/). Te standardowe akcje możemy
usunąć lub zmienić wedle uznania. Możemy też dorobić własne akcje i przypisać im czas, przez który
przycisk musi być wciśnięty.
