---
author: Morfik
categories:
- DD-WRT
date: "2016-09-07T18:08:40Z"
date_gmt: 2016-09-07 16:08:40 +0200
published: true
status: publish
tags:
- router
- tp-link
- dd-wrt
GHissueID: 380
title: 'DD-WRT: Jak wgrać firmware na router TP-LINK'
---

[OpenWRT](https://openwrt.org/)/[LEDE](https://lede-project.org/) nie jest jedynym alternatywnym
firmware, który możemy wgrać na router TP-LINK'a. Jest także [projekt
DD-WRT](https://www.dd-wrt.com/site/), który oferuje oprogramowanie dla większości modeli routerów
tego producenta. Tak się składa, że mam router TL-WDR3600 i posiada on pełne wsparcie w DD-WRT.
Postanowiłem zatem zainstalować to oprogramowanie na tym urządzeniu i opisać jak wygląda cały proces
instalacyjny.

<!--more-->
## Pozyskiwanie firmware DD-WRT

Firmware możemy pozyskać ze strony samego projektu DD-WRT. Musimy tylko ustalić, czy nasz router
jest wspierany. Możemy to zrobić przechodząc [pod ten
adres](https://www.dd-wrt.com/site/support/router-database) i wpisując w formularzu nazwę
posiadanego przez nas routera. Możemy także przejść na [wiki
DD-WRT](https://www.dd-wrt.com/wiki/index.php/Supported_Devices#TP-Link) i tam poszukać naszego
urządzenia. Jeśli znajduje się ono na liście wspieranych modeli i nie ma co do naszego router
[żadnych zastrzeżeń](https://www.dd-wrt.com/wiki/index.php/Hardware-specific), to możemy czytać
dalej ten artykuł bez większych obaw.

Jeśli znajdujemy się w sytuacji przed zakupem routera ale już wiemy, że będziemy na nim instalować
firmware DD-WRT, to koniecznie przestudiujmy sobie te listy wspieranych urządzeń i w oparciu o nie
dokonajmy zakupu odpowiedniego routera, który będzie nam działał bez problemu pod tym
oprogramowaniem.

Jako, że w tym przypadku dysponuję routerem
[TL-WDR3600](http://www.tp-link.com/en/products/details/TL-WDR3600.html), to szukamy właśnie za nim:

![](/img/2016/09/1.dd-wrt-tl-wdr3600-tp-link-firmware-szukanie.png#huge)

Jak widzimy, router jest w pełni wspierany przez DD-WRT (koniecznie zwróćmy uwagę na wersję
sprzętu). W przypadku wszystkich wspieranych routerów TP-LINK, firmware DD-WRT można wykorzystywać
bez licencji i nie potrzebujemy kupować żadnego klucza. Klikamy zatem na powyższej pozycji i po
chwili powinna nam się załadować strona, z której możemy pobrać stosowny plik w celu wgrania go na
router:

![](/img/2016/09/2.dd-wrt-tl-wdr3600-tp-link-firmware-pobieranie.png#huge)

Wgranie alternatywnego firmware na router jest stosunkowo proste ale jeśli zrobimy to niepoprawnie,
np. wgramy nie ten plik co trzeba, to router może nam już się nie odpalić. Jako, że flash'ujemy
router po raz pierwszy tym oprogramowaniem, to musimy pobrać plik `factory-to-ddwrt.bin` .

Trzeba wyraźnie zaznaczyć, że te powyższe obrazy zawierają stabilną wersję DD-WRT. Jak możemy
wywnioskować po dacie, są one nieco leciwe. Niemniej jednak, jeśli nie chcemy mieć żadnych problemów
z użytkowaniem routera, to powinniśmy się zdecydować na tę wersję stabilną. W przypadku, gdy chcemy
nowszą wersję firmware, to trzeba skorzystać z [obrazów w wersji
beta](https://www.dd-wrt.com/site/support/other-downloads).

## Różnice w obrazach DD-WRT

Jako, że na rynku jest cała masa urządzeń, które są w stanie pracować pod firmware DD-WRT, to
oprogramowanie zawarte w obrazach może się różnić dość znacznie. Nie da rady przecież zmieścić tych
samych aplikacji na flash'u 8 MiB i 4 MiB. Router TL-WDR3600 dysponuje flash'em 8 MiB, zatem więcej
ciekawych rzeczy DD-WRT może nam zaoferować.

Na wiki DD-WRT znajduje się ciekawa rozpiska na temat[ficzerów, które posiadają różne wersje
obrazów](https://www.dd-wrt.com/wiki/index.php/What_is_DD-WRT%3F#File_Versions). Mamy tam
informację, że oprogramowanie przeznaczone na routery posiadające podzespoły Qualcomm Atheros jest
zbliżone pod względem funkcjonalności do tego obsługującego urządzenia mające czipy Broadcom. Zatem
interesuje nas opis `Broadcom K2.6 BIG` , bo mamy flash 8 MiB, no i oczywiście ten router posiada
podzespoły Qualcomm Atheros'a.

## Proces flash'owania

Mając już plik obrazu na dysku, łączymy się do routera za pomocą przewodu i logujemy się do panelu
webowego podając login `admin` oraz hasło również `admin` :

![](/img/2016/09/3.dd-wrt-tl-wdr3600-tp-link-firmware-logowanie.png#huge)

W panelu przechodzimy do System Tools => Firmware Upgrade i wskazujemy plik, który wcześniej
pobraliśmy:

![](/img/2016/09/4.dd-wrt-tl-wdr3600-tp-link-firmware-flash.png#huge)

Po kliknięciu przycisku `Upgrade` powinien rozpocząć się proces flash'owania routera:

![](/img/2016/09/5.dd-wrt-tl-wdr3600-tp-link-firmware-flash.png#huge)

Po chwili proces powinien się zakończyć powodzeniem, a router powinien się uruchomić ponownie:

![](/img/2016/09/6.dd-wrt-tl-wdr3600-tp-link-firmware-flash.png#huge)

## Panel administracyjny DD-WRT

Dostęp do routera możemy uzyskać z poziomu graficznego panelu webowego, który znajduje się zwykle
pod adresem `http://192.168.1.1/` . Po odwiedzeniu tego adresu będziemy musieli ustawić nazwę
użytkownika i hasło do konta:

![](/img/2016/09/7.dd-wrt-tl-wdr3600-tp-link-firmware-logowanie.png#huge)

Po zmianie danych logowania, będziemy w stanie zalogować się do panelu administracyjnego skąd
będziemy mogli zarządzać routerem:

![](/img/2016/09/8.dd-wrt-tl-wdr3600-tp-link-firmware-panel-admina.png#huge)
