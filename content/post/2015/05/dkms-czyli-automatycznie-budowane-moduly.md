---
author: Morfik
categories:
- Linux
date: "2015-05-30T21:05:27Z"
lastmod: 2020-03-14 01:21:00 +0100
published: true
status: publish
tags:
- debian
- dkms
- kernel
- moduły-kernela
GHissueID: 224
title: DKMS, czyli automatycznie budowane moduły
---

Jeśli zamierzamy kupić sprzęt, który dopiero co trafił na półki w sklepach, to prawdopodobnie zaraz
po podłączeniu go do naszego komputera okaże się, że to urządzenie nie jest nawet wykrywane przez
system operacyjny. W przypadku gdy jego producent zapewnia w miarę przyzwoity support, to być może
problemy, których doświadczamy, zostaną rozwiązane wraz z instalacją najnowszego kernela. Co jednak
w przypadku gdy nawet po aktualizacji kernela nie jesteśmy w stanie odpalić, np. nowo zakupionej
karty WiFi? Jako, że te wszystkie sprzęty działają w oparciu określone moduły, wystarczy taki moduł
pozyskać, skompilować i załadować w systemie. Problem w tym, że z każdą nową wersją jądra
operacyjnego, która trafi do repo debiana, będziemy musieli ręcznie budować moduł na nowo i właśnie
w tym artykule opiszę jak nauczyć system, by sam przeprowadzał tę mozolną czynność bez naszego
udziału.

<!--more-->
## Przykład zastosowania DKMS

Mechanizm DKMS domyślnie buduje moduły tylko dla aktualnie używanego kernela i każdego innego
posiadającego wyższą wersję. By zbudować moduł dla kernela z niższym numerkiem, trzeba posłużyć się
przełącznikiem `-k` .

Przede wszystkim, musimy znaleźć na necie odpowiednie sterowniki. Po ich pobraniu, pliki wrzucamy do
`/usr/src/` i zmieniamy nazwę katalogu na `nazwa_modulu-wersja`, przykładowo
`rtl8812au-git-dkms-r31.83f539d` . Następnie wewnątrz tego folderu tworzymy plik `dkms.conf` o
poniższej treści:

    MAKE="'make' all"
    CLEAN="make clean"
    BUILT_MODULE_NAME=8812au
    PACKAGE_NAME=rtl8812au
    PACKAGE_VERSION=1.0
    REMAKE_INITRD=no
    DEST_MODULE_LOCATION=/kernel/drivers/net/wireless
    AUTOINSTALL="yes"

Pod żadnym pozorem nie usuwajmy źródeł modułów z katalogu `/usr/src/` . Jeśli to zrobimy, taki moduł
się nie zbuduje przy instalacji nowego kernela, bo zwyczajnie nie będzie miał jak.

Moduł możemy dodać ( `add` ) , zbudować ( `build` ) lub beż dodać, zbudować i zainstalować (
`install` ), z tym, że trzeba mieć na uwadze iż budowa modułu może trochę zająć:

    # dkms install -m rtl8812au-git-dkms -v r31.83f539d

Parametry `-m` oraz `-v` mają znaczenie, tj. system po nich rozpozna odpowiednią nazwę katalogu.
Jeśli któryś z nich podamy nieprawidłowo, zostanie wyrzucony błąd:

    # dkms install -m rtl8812au-git-dkms -v r31.83f539d-test
    Error! Could not find module source directory.
    Directory: /usr/src/rtl8812au-git-dkms-r31.83f539d-test does not exist.

Jeśli mamy w systemie więcej niż jedno jądro operacyjne, to przydałoby się ten moduł wygenerować dla
nich wszystkich. Robimy to przez zdefiniowanie parametru `-k` , przykładowo:

    # dkms install -m rtl8812au-git-dkms -v r31.83f539d -k 3.16.0-4-amd64

W przypadku gdy kerneli mamy więcej, parametr `-k` możemy podać kilka razy.

W tej chwili powinniśmy mieć już zbudowane moduły dla wszystkich kerneli. Możemy to sprawdzić przez
`status` :

    # dkms status
    rtl8812au-git-dkms, r31.83f539d, 3.16.0-4-amd64, x86_64: installed
    rtl8812au-git-dkms, r31.83f539d, 4.0.0-1-amd64, x86_64: installed

Od tej pory za każdym razem gdy pojawi się nowy kernel w repozytorium, ten moduł zostanie
automatycznie zbudowany.

By usunąć moduły, posługujemy się `remove` , z tym, że nazwę modułu i jego wersję oddzielamy przy
pomocy `/` i, podobnie jak w przypadku instalacji, możemy usunąć moduł z konkretnego kernela (opcja
`-k` ), bądź też ze wszystkich naraz (parametr `--all` ) :

    # dkms remove rtl8812au-git-dkms/r31.83f539d --all
