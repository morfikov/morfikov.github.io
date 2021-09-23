---
author: Morfik
categories:
- OpenWRT
date: "2016-04-27T20:47:12Z"
date_gmt: 2016-04-27 18:47:12 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- init
title: Skrypty startowe init w OpenWRT
---

Znaczna część usług, które są dostępne w OpenWRT, jest odpalana [w fazie startowej
routera](https://wiki.openwrt.org/doc/techref/process.boot#init). Te zadnia są realizowane przez
skrypty startowe, które są dołączane w konkretnych pakietach. Takie skrypty są wywoływane w
odpowiedniej kolejności. Każdy z nich można dodać lub usunąć z autostartu. W przypadku, gdy jakaś
usługa nie dostarcza swojego skryptu startowego, możemy pokusić się o napisanie jej takowego.
Oczywiście, nic też nie stoi na przeszkodzie, by utworzyć własne skrypty startowe, które
niekoniecznie odnoszą się do określonych usług. Mogą one np. realizować pewne określone zadanie. W
tym wpisie przybliżymy sobie trochę budowę skryptów startowych w OpenWRT tak, by być w stanie je
tworzyć i edytować jeśli zajdzie taka potrzeba.

<!--more-->
## Gdzie zlokalizowane są skrypty startowe

Wszystkie skrypty startowe dostarczane w pakietach są po instalacji umieszczane w katalogu
`/etc/init.d/` . Każdy skrypt posiada szereg opcji, z którymi może zostać wywołany. Część z tych
opcji jest wspólna dla wszystkich skryptów startowych. Niemniej jednak, nic nie stoi na
przeszkodzie, aby stworzyć własne opcje i przypisać im określone akcje. By się dowiedzieć jakimi
opcjami dysponuje jakiś skrypt startowy, najlepiej jest go po prostu podejrzeć lub też wywołać go
bez żadnej opcji. Poniżej przykład:

![](/img/2016/04/2.skrypty-startowe-openwrt.png#huge)

Przy pomocy opcji `start` lub `stop` jesteśmy w stanie uruchomić usługę lub ją zatrzymać. Możemy
także taką usługę zrestartować przy pomocy `restart` . Choć można ten sam stan rzeczy otrzymać
wywołując kolejno `stop` i `start` . Nie każda usługa posiada opcję `reload` , która to wczytuje
zmiany dokonane w plikach konfiguracyjnych, na których operuje skrypt. Natomiast wszystkie z usług
posiadają opcje `enable` i `disable` . Odpowiadają one za włączenie i wyłączenie danego skryptu,
czyli decydują o tym, czy jakaś usługa będzie aktywowana automatycznie podczas startu systemu
routera czy też nie.

## Dodawanie usług do autostartu systemu

Każdy z plików zlokalizowanych w `/etc/init.d/` posiada nagłówek. W nim zawarte są min informacje na
temat pozycji skryptu w kolejce podczas fazy boot. Odpowiada za to zmienna `START` . Z kolei zmienna
`STOP` jest opcjonalna i dotyczy kolejności wyłączania usług. Poniżej jest przykład:

![](/img/2016/04/3.skrypty-startowe-openwrt-naglowek.png#medium)

Te dwie powyższe zmienne są brane pod uwagę przy wywoływaniu skryptu z opcjami `enable` i
`disable` . Po ich wydaniu, w katalogu `/etc/rc.d/` zostaną utworzone dowiązania do skryptu,
przykładowo:

![](/img/2016/04/1.skrypty-startowe-kolejnosc-startu-openwrt-boot.png#huge)

Nazwa linku jest taka sama jak nazwa skryptu, z tą różnicą, że mamy tam dodatkowo literki `S` lub
`K` i jakiś numerek. Skrypty, które mają linki zaczynające się od `S` będą ładowane przy starcie
systemu. Natomiast te, które mają dowiązania zaczynające się od `K` , będą wykonywane przy restarcie
czy wyłączeniu (via `poweroff`) routera. Z kolei liczba, np. `50` , oznacza wartość, która widnieje
w zmiennej `START`/`STOP` w skrypcie startowym. Każdy skrypt ma jakiś numer. Te, które mają mniejsze
numerki są wykonywane przed tymi co mają większe. W przypadku, gdy dwa skrypty mają taki sam numer,
kolejność ich startu jest ustalana w oparciu o porządek alfabetyczny.

## Budowa skryptów startowych

Wszystkie skrypty startowe obecne w katalogu `/etc/init.d/` muszą posiadać atrybut wykonywalności (
`chmod +x` ), jako że są to zwykle skrypty shell'owe. Każdy z tych plików musi także rozpoczynać
się od poniższej linijki:

    #!/bin/sh /etc/rc.common

Jest to wrapper zapewniający główną i domyślną funkcjonalność dla skryptu. Sprawdza on także skrypt
startowy pod kątem ewentualnych błędów przed wykonaniem go. Dodatkowo, każdy skrypt musi posiadać
przynajmniej dwa standardowe bloki kodu. Po jednym dla akcji `start` i `stop` :

    start() {
    ...
    }

    stop() {
    ...
    }

To w nich umieszczamy instrukcje, na podstawie, których system wie co ma robić przy przetwarzaniu
konkretnego skryptu. W przypadku, gdy w skrypcie ma być zawarta inna procedura startu w stosunku do
normalnego wywołania skryptu (w tym również w przypadku restartu usługi), to musimy także zawrzeć
poniższy blok kodu:

    boot() {
    ...
    }

W skryptach startowych mogą być także zdefiniowane własne polecenia. Wystarczy w nagłówku pliku
zawszeć poniższą linijkę:

    EXTRA_COMMANDS="create delete"

Powyżej mamy zdefiniowane dwa nowe polecenia: `create` oraz `delete` . By je wykorzystać w pliku,
tworzymy dla nich osobne sekcje podobne do tych powyżej:

    create() {
    ...
    }

    delete() {
    ...
    }
