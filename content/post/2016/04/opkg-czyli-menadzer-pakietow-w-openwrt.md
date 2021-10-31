---
author: Morfik
categories:
- OpenWRT
date: "2016-04-24T17:49:09Z"
date_gmt: 2016-04-24 15:49:09 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
GHissueID: 421
title: OPKG, czyli menadżer pakietów w OpenWRT
---

W OpenWRT do zarządzania pakietami wykorzystywany jest menadżer pakietów
[opkg](https://wiki.openwrt.org/doc/techref/opkg). To przy jego pomocy instalujemy i usuwamy
pakiety. Niemniej jednak, to narzędzie nie ogranicza się jedynie do tych dwóch powyższych czynności.
Przy pomocy `opkg` jesteśmy w stanie przeprowadzić szereg innych operacji dotyczących zarządzania
pakietami w systemie operacyjnym naszego domowego routera. W tym wpisie prześledzimy sobie
poszczególne opcje jakie ten menadżer pakietów nam oferuje.

<!--more-->
## Dlaczego "opkg update" jest taki ważny

Wszystkie pakiety, które chcemy zainstalować w OpenWRT, są przechowywane w repozytoriach. Listy z
pakietami dostępnymi w repozytoriach można pobrać za pomocą polecenia `opkg update` . Cały proces
wygląda mniej więcej tak jak pokazano na obrazku poniżej:

![opkg-update-lista-repozytoria-openwrt](/img/2016/04/2.opkg-update-lista-repozytoria-openwrt.png#huge)

Użytkownikom windowsa ten mechanizm może wydawać się początkowo obcy i niezrozumiały. Generalnie
rzecz ujmując, na linux'ach nie zwykło się pobierać programów z pierwszego linku, który zwróci nam
google. Dlatego są te repozytoria. Z jednej strony poprawiają one bezpieczeństwo, a z drugiej zaś
wiemy, że dana aplikacja będzie nam działać bez problemu, bo została sprawdzona. Wszystkie pakiety,
które nie spełniają wymogów stawianych przez developerów OpenWRT, są zwyczajnie usuwane z
repozytoriów. W dalszym ciągu, takie usunięte pakiety możemy sobie zainstalować ale wtedy jeśli
router nie będzie chciał się nam uruchomić lub jakaś usługa nie będzie działać jak należy, to
pretensje możemy mieć tylko do siebie.

Różnica w stosunku do dystrybucji linux'a w kwestii menadżera pakietów jest taka, że z reguły
desktop'y mają o wiele większe dyski twarde w porównaniu do flash'a przeciętnego routera domowego.
Dlatego też z tego powodu, wszystkie listy pakietów w OpenWRT są pobierane do pamięci operacyjnej,
której routery mają zwykle więcej niż dostępnej przestrzeni na flash'u. Dane w pamięci RAM nie
przetrwają restartu routera, dlatego za każdym razem musimy wydawać polecenie `opkg update` , by te
listy były obecne w stosownym miejscu w pamięci. Ponadto, biorąc pod uwagę fakt, że aplikacje są
rozwijane oraz, że OpenWRT posiada ich całą masę, wersje pakietów ulegają zmianie. Wraz z nimi
zmieniają się także zależności, które muszą być zainstalowane, by dana usługa mogła działać bez
przeszkód. Pobranie nowych list aktualizuje te wszystkie wpisy. Trzeba się z tym liczyć w przypadku,
gdy chcemy zainstalować jakiś pakiet na starszej wersji firmware. Może to nie być możliwe za sprawą
konfliktu zależności w pakietach, które są obecne w repozytoriach. Dlatego też zawsze starajmy się
posiadać w miarę nową wersję OpenWRT, by ograniczyć tego typu problemy do minimum.

## Konfiguracja opkg

Plik konfiguracyjny dla menadżera pakietów `opkg` jest zlokalizowany w `/etc/opkg.conf` . We
wcześniejszych wersjach OpenWRT znajdowały się tam także linki do repozytoriów. Obecnie w wydaniu
Chaos Calmer, te linki zostały przeniesione do osobnego pliku i teraz znajdują się pod
`/etc/opkg/distfeeds.conf` . Poniżej jest przykład konfiguracji:

    dest root /
    dest ram /tmp
    lists_dir ext /var/opkg-lists
    option overlay_root /overlay
    option check_signature 1

Linijki zaczynające się od `dest` określają miejsce docelowe dla instalowanych pakietów. Wyżej mamy
zdefiniowane dwie lokalizacje: `/` oraz `/tmp/` . Nazwy `root` i `ram` mogą być dowolne i robią za
opisy. Te nazwy są wykorzystywane przy instalacji pakietów, co bardzo przydaje się, gdy dysponujemy
routerem mającym flash o niewielkich rozmiarach. Opcja `lists_dir` wskazuje lokalizację pobieranych
list pakietów. Opcja `overlay_root` odpowiada za system plików, który ma być sprawdzony pod kątem
wolnego miejsca przy instalowaniu pakietów. Natomiast `check_signature` odpowiada za weryfikację
sygnatur repozytoriów. W przypadku dokonania zmian w powyższej konfiguracji, dobrze jest wyczyścić
katalog `/var/opkg-lists/` przy pomocy poniższego polecenia:

    # rm /var/opkg-lists/*

## Szukanie pakietów

W przypadku, gdy nie pamiętamy dokładnej nazwy pakietu, który chcemy zainstalować lub odinstalować,
możemy przeszukać listy pakietów. W `opkg` mamy do tego celu oddelegowane dwie opcje: `list` oraz
`list-installed` . Pierwsza z nich zwraca wszystkie dostępne pakiety, druga zaś jedynie te, które są
zainstalowane w systemie. Obie te listy mogą zostać przefiltrowane za pomocą narzędzia `grep` .
Przeszukiwanie list odbywa się w oparciu o nazwę jak i opis danego pakietu. Poniżej przykłady:

    # opkg list | grep dhcp
    # opkg list-installed | grep dns

Wygląda to mniej więcej tak jak na poniższej fotce:

![opkg-list-listinstalled-openwrt](/img/2016/04/3.opkg-list-listinstalled-openwrt.png#huge)

## Instalowanie pakietów

Pakiety możemy instalować za pomocą opcji `install` na trzy sposoby. Najpopularniejszym z nich jest
instalowanie pakietu z repozytorium. W tym przypadku podajemy jedynie nazwę pakietu. Może się jednak
zdarzyć tak, że pakiet nie jest dostępny w repozytoriach OpenWRT. Nie przeszkadza to jednak w jego
instalacji. W dalszym ciągu możemy go zainstalować ręcznie, z tym, że trzeba mieć na uwadze
zależności, których ten pakiet wymaga do poprawnego działania. Sam pakiet instalujemy zaś podając
bezpośredni link do pliku `.ipk` , bądź też przez wskazanie ścieżki do tego pliku w przypadku, gdy
pobraliśmy go już na dysk. Poniżej przykłady:

    # opkg install ipset
    # opkg install http://dl.eko.one.pl/chaos_calmer/ar71xx/packages/ipset_6.24-1_ar71xx.ipk
    # opkg install /tmp/ipset_6.24-1_ar71xx.ipk

Pamiętajmy też, że nie należy mieszać modułów kernela. Dla przykładu, kernel pochodzi z repozytorium
OpenWRT. Część modułów może nie być udostępniona z różnych powodów. Możemy za to dysponować
zewnętrznym repozytorium, w którym będziemy mieć pożądany przez nas moduł. Nawet jeśli wersja tego
modułu pasuje do wersji kernela, to nie powinniśmy instalować tego modułu.

## Odinstalowanie pakietów

Pakiety mogą również zostać odinstalowane, np. w celu zwolnienia miejsca. Trzeba jednak pamiętać
przy tym, że pakiety, które są dostarczane wraz z obrazem firmware nie mogą zostać fizycznie z niego
usunięte. Mało tego, informacja o usunięciu tych pakietów zajmuje miejsce i paradoksalnie nie dość,
że nie zwolnimy ani jednego bajta, to jeszcze tylko zmarnujemy trochę dostępnej przestrzeni.

Wszystkie inne pakiety, tj. te, które instalowaliśmy po flash'owaniu routera, możemy bez problemu
odinstalować przy pomocy opcji `remove` . Problem w tym, że praktycznie każdy pakiet zawiera listę
zależności, które są instalowane wraz z nim. Później, gdy taki pakiet chcemy usunąć, to w systemie
zostaną nam śmieci w postaci nieużywanych pakietów. To z kolei zajmuje miejsce. Na szczęście `opkg`
jest w stanie sobie z tego typu problemem poradzić. Mamy do tego celu dwa parametry: `--autoremove`
oraz `--force-removal-of-dependent-packages` . Pierwsza z nich usunie zależności, które zostały
zainstalowane wraz z pakietem. Niemniej jednak, cześć z tych zależności może być wykorzystywana
przez inne pakiety. W efekcie część z nich nie zostanie usunięta, co jest logiczne. Druga z
powyższych opcji usunie wszystkie zależności bez znaczenia czy inne pakiety z nich korzystają.
Poniżej przykłady:

    # opkg remove ipset
    # opkg remove --autoremove ipset
    # opkg remove --force-removal-of-dependent-packages ipset

## Aktualizacja pakietów

Aktualizacja oprogramowania, które operuje na routerze, to bardzo ważna rzecz. Tak ważna, że `opkg`
posiada możliwość aktualizowania pakietów przy pomocy opcji `upgrade` . Problem w tym, że flash
routera jest nie za duży i aktualizacja pakietów nie zawsze jest możliwa. Trzeba także pamiętać o
tym, że pakiety dostarczane standardowo z firmware po aktualizacji będą zjadać dwa razy tyle
miejsca. A to z tego względu, że starych pakietów obecnych w obrazie firmware nie można usunąć.
Aktualizacja pakietów ma tylko sens w przypadku instalowanych rzeczy po flash'owaniu routera. Listę
pakietów, które wymagają aktualizacji, możemy uzyskać za pomocą opcji `list-upgradable` . Poniżej
przykłady:

    # opkg list-upgradable
    # opkg upgrade ipset

Niektóre pakiety, w szczególności moduły kernela, nie powinny być nigdy aktualizowane. Taka akcja
może sprawić, że router będzie działał bardzo niestabilnie. Bardziej jednak prawdopodobnego jest,
że go w ten sposób powiesimy i trzeba będzie go ratować przy pomocy [trybu
failsafe](/post/tryb-ratunkowy-failsafe-w-openwrt/).

## Informacje o pakiecie

Menadżer pakietów `opkg` posiada pewne informacje, które mogą okazać się użyteczne dla nas. Możemy
je wydobyć za pomocą opcji `info` . Poniżej informacje na temat przykładowego pakietu:

![opkg-info-openwrt](/img/2016/04/4.opkg-info-openwrt.png#big)

Mamy tutaj nazwę pakietu, jego wersję oraz zależności, które muszą być spełnione, by ten pakiet mógł
zostać zainstalowany w systemie. Dalej widzimy, że pakiet został zainstalowany przez użytkownika.
Mamy także informacje o sekcji, do której pakiet przynależy oraz architekturę, dla której został
zbudowany. Jest także suma kontrolna pakietu i rozmiar w bajtach. Przy czym, jeśli chodzi o rozmiar,
to trzeba bać poprawkę, gdyż system plików routera jest skompresowany. W efekcie ten pakiet po
zainstalowaniu zwykle będzie zajmował mniej miejsca niż jest uwzględnione w opisie pakietu. Dalej
mamy pełną nazwę pliku `.ipk` . Są też wyszczególnione pliki konfiguracyjne wraz z sumami
kontrolnymi (o nich niżej). Mamy też informacje o źródłach, z których pakiet został zbudowany. Na
końcu zaś mamy opis pakietu i datę zainstalowania w formie unix'owego timestamp'a. W tym przypadku,
`1460787028` wskazuje na [2016-04-16 08:10:28 CEST GMT+2:00 DST](https://www.epochconverter.com/).

Na fotce widzimy, że ten pakiet (jak i cała masa innych) dostarcza pliki konfiguracyjne. Są one
zwykle wgrywane do katalogu `/etc/` . Wszystkie te pliki są monitorowane pod kątem zmian. Dlatego
też `opkg` ma ich sumy kontrolne. Jeśli jakiś plik zmienimy w drodze konfiguracji routera, to suma
kontrolna takiego pliku również ulegnie zmianie. W oparciu o ten fakt, menadżer pakietów jest nam w
stanie zwrócić listę zmienionych plików konfiguracyjnych za pomocą opcji `list-changed-conffiles` .
Poniżej przykład:

    # opkg list-changed-conffiles

Każdy pakiet składa się zwykle też z szeregu innych plików, które na powyższym listingu nie
figurują. Niemniej jednak, `opkg` posiada informacje na temat plików zawartych w poszczególnych
pakietach. Dokładną listę tych plików możemy uzyskać za pomocą opcji `files` . Poniżej przykład:

    # opkg files dnsmasq

Oczywiście na powyższych listach nie ma plików, które sami stworzyliśmy.

Jeśli chcemy ustalić do jakiego pakietu należy konkretny plik, to przy pomocy opcji `search` możemy
taki pakiet odszukać. Poniżej przykład:

    # opkg search /etc/dnsmasq.conf

## Niezbyt czytelne wyniki zwracane w opkg

Przeszukiwanie pakietów za pomocą `opkg list` nie należy do przyjemnych rzeczy. Głównie chodzi tutaj
o fakt, że lista pakietów, która jest nam zwracana po wydaniu powyższego polecenia, nie jest zbytnio
czytelna. Wystarczy popatrzeć na poniższą fotkę:

![opkg-list-openwrt](/img/2016/04/6.opkg-list-openwrt.png#huge)

Na samym początku mamy nazwę pakietu, później jest jego wersja, a na końcu opis. Gdyby te wpisy były
zwracane przez `opkg` na zasadzie jeden na linijkę, to nie byłoby jeszcze tak źle. Niemniej jednak,
opisy mogą być bardzo długie i odnalezienie nazwy pakietu w gąszczu tekstu nie jest zbytnio łatwe i
szybkie. By poprawić nieco przejrzystość komunikatów zwracanych przez `opkg` , tak by, np. nazwy
pakietów odróżniały się nieco od ich opisów, możemy dodać poniższy blok kodu do pliku
`/etc/profile` :

	# Written by Gordio <me@gordio.pp.ua>
	# Licence: MIT
	opkg () {
		BOLD=$(echo -e '\033[1m');
		NORM=$(echo -e '\033[0m');
		COL="no";
		for arg in $*; do
			if [ $arg == "whatdepends" -o $arg == "list" \
			-o $arg == "list-installed" -o $arg == "list-upgradable" \
			-o $arg == "list-changed-conffiles" -o $arg == "status" \
			-o $arg == "info" -o $arg == "find" ]; then
				COL="yes";
				break;
			fi
		done
		if [ $COL == "yes" ]; then
			# (|\t*) added for 'whatdepends'
			/bin/opkg $* | sed -re "s/^(|\t*)[a-z0-9-]*/$BOLD&$NORM/g";
		else
			/bin/opkg $*;
		fi
	}
	# vim: set fdm=marker ts=4 sw=4 tw=80 fo-=t ff=unix:

By zmiany zaczęły obowiązywać, musimy się zalogować ponownie na router albo wpisać w terminalu to
poniższe polecenie:

    # source /etc/profile

Wyniki zwracane przez `opkg` powinny teraz być bardziej czytelne:

![opkg-list-openwrt-fix](/img/2016/04/7.opkg-list-openwrt-fix.png#huge)
