---
author: Morfik
categories:
- Linux
date: "2016-06-16T21:35:20Z"
date_gmt: 2016-06-16 19:35:20 +0200
published: true
status: publish
tags:
- procesy
title: Narzędzia nice, renice, ionice, taskset i trickle
---

W każdym systemie operacyjnym cała masa procesów rywalizuje ze sobą o zasoby, w skład których
wchodzą min. pamięć operacyjna RAM, procesor i operacje I/O dysku twardego. Czasami zdarza się tak,
że niektóre aplikacje są w stanie zdusić inne programy, bo mają zbyt duże wymagania co do zasobów
systemowych. W takich przypadkach administrator systemu powinien zatroszczyć się o przydział zasobów
konkretnym procesom. W linux'ie do tego typu prac przeznaczony jest [mechanizm
cgroups]({{< baseurl >}}/post/ograniczanie-zasobow-procesom-przez-cgroups/) obecny w kernelu.
Niemniej jednak, jeśli cgroups przerasta nasze umiejętności albo też z jakiegoś powodu nie możemy z
niego korzystać, to istnieje inne rozwiązanie, które może nam pomóc ograniczyć zasoby przydzielane
procesom przez nasz system. Chodzi generalnie o narzędzia `nice`/`renice` (procesor) , `ionice`
(dysku twardy) , `taskset` (przypisanie procesu do konkretnego procesora) oraz `trickle` (sieć). W
tym wpisie zobaczymy jak przy pomocy tych powyższych narzędzi limitować procesy systemowe.

<!--more-->
## Przydział procesora i dysku twardego (nice, renice, ionice)

Z reguły `nice` i `ionice` są wykorzystywane razem. Oczywiście, nic nie stoi na przeszkodzie, by
korzystać z tych dwóch narzędzi osobno. Niemniej jednak, gdy w grę wchodzi ograniczanie zasobów
procesom, to zwykle chcemy, by jeden z procesów miał niższy lub wyższy priorytet. A to z reguły
prowadzi do ograniczenia przydziałów dwóch kluczowych urządzeń komputera, czyli procesora i dysku
twardego. W przypadku ograniczenia tylko jednego z tych zasobów, proces w dalszym ciągu byłby w
stanie korzystać z tego drugiego zasobu bez ograniczeń, co może czasami nie być pożądane. W
przypadku korzystania zarówno z `nice` jak i `ionice` możemy tak skonfigurować proces, że jego
operacje nie będą dla nas praktycznie w żaden sposób odczuwalne.

By korzystać z w/w narzędzi, musimy posiadać w swoim systemie odpowiednie pakiety. W przypadku
debiana są to `coreutils` oraz `util-linux` . Raczej nie powinno być problemów z ich
zainstalowaniem, zatem przejdźmy od razu do rzeczy, czyli ograniczenia zasobów procesora i dysku
twardego.

Jeśli chcemy ograniczyć zasoby jakiemuś procesowi, przy jego wywoływaniu musimy dodać `nice` i/lub
`ionice` . Każde z tych dwóch narzędzi może przyjąć też kilka opcji, przy pomocy których nadamy
priorytet procesowi. W przypadku określenia zarówno `nice` jak i `ionice` stworzymy drzewo procesów,
na końcu którego będzie nasza główna aplikacja. Całość działa na zasadzie dziedziczenia parametrów
procesów. Przykładowo, jeśli proces z `nice -n 19` wywoła inny proces, to ten drugi proces również
dostanie z automatu `nice -n 19` . Jest to dokładnie zobrazowane w `htop` na poniższym obrazu:

![]({{< baseurl >}}/img/2016/06/1.procesy-nice-htop-linux.png#huge)

Wyżej widzimy, że jednemu z paneli `tint2` został przypisany `nice -n -2` . Na tym panelu jest
szereg aktywatorów aplikacji. Każdy uruchomiony proces z tego panelu będzie procesem potomnym i
będzie dziedziczył parametry przydziału zasobów, tak jak to jest pokazane wyżej. Poniżej zaś jest
przykładowe polecenie wywołania skryptu od backup'u:

    # nice -n 19 ionice -c 3 backup.sh

Priorytety dla `nice` są z zakresu `-20` do `19` . Zwykły użytkownik może nadawać tylko priorytety
od `0` do `19` . Im wyższy numer tym niższy priorytet, czyli procesy z numerem `-20` mają najwyższy
priorytet, a te z numerem `19` najniższy.

W przypadku `ionice` , przy pomocy `-c 3` ustalamy zapis/odczyt dysku na tryb `IDLE` , czyli gdy
inne procesy aktualnie nie wykorzystują dysku. Możliwe jest także sprecyzowanie mniej agresywnego
rozwiązania, czyli `-c 2 -n 7` . Przy czym, opcja `-n` w tym przypadku jest z zakresu `0-7` i
również im mniejsza wartość, tym wyższy priorytet, a im wyższy priorytet, tym lepszy dostęp do
dysku.

## Zmiana priorytetu procesu w locie

Warto też wiedzieć, że priorytet procesów w dostępie do procesora i dysku twardego można zmienić w
czasie pracy takiego procesu. W przypadku zasobów procesora zamiast `nice` korzystamy z `renice` . Z
kolei jeśli chodzi o dysk twardy, to procedura jest taka sama jak wyżej, czyli korzystamy z `ionice`
. W obu przypadkach musimy korzystać z przełącznika `-p` , za pomocą którego wskazujemy numer
procesu, którego priorytet chcemy zmienić. Poniżej są przykłady zmiany priorytetu procesu:

    # renice -n 10 -p 1110
    # ionice -c 2 -n 5 -p 1110

Z tymi powyższymi poleceniami jest tylko jeden problem. Co w przypadku drzewa procesów? Zmiana
priorytetu jednemu procesowi nie stanowi problemu ale raczej nikomu się nie będzie chciało zmieniać
szeregu procesów potomnych. Istnieje jednak sposób, który automatycznie dostosuje wszystkie te
procesy potomne za nas. Musimy tylko poznać numer ID sesji procesów. To ID możemy poznać, np. przy
pomocy `ps` lub `htop` (domyślnie nie jest pokazywane ID sesji i trzeba zaznaczyć odpowiednie opcje
w konfiguracji). Poniżej jest przykład `ps` :

    # ps -eo "user,group,sess,pid,args" | grep fire
    morfik   morfik    44835  44835 /bin/sh -c /opt/firefox/firefox --new-instance --ProfileManager
    morfik   morfik    44835  44836 /opt/firefox/firefox --new-instance --ProfileManager

Wartość `44835` to numer sesji. ID poszczególnych procesów przypisanych do tej sesji możemy ustalić
w poniższy sposób:

    # ps -eo sess:1=,pid:1=,nice:1= | grep 44835
    44835 44835 0
    44835 44836 0

Lub też przy pomocy `pgrep` . Poniżej przykład:

    # pgrep -s 44835
    44835
    44836

By zmienić `nice` wszystkim procesom podpiętym do jednej sesji, wpisujemy w terminal to poniższe
polecenie:

    # renice -n 19 $(pgrep -s 44835)
    44835 (process ID) old priority 0, new priority 19
    44836 (process ID) old priority 0, new priority 19

Jak widać priorytety uległy zmianie. Podobnie postępujemy w przypadku `ionice` :

    # ionice -c 3 -p $(pgrep -s 44835)

## Jak przydzielić procesowi określony procesor/rdzeń (taskset)

Jeśli interesuje nas opcja, gdzie danym procesem zajmuje się określony rdzeń procesora, to możemy
skorzystać z narzędzia `taskset` (zawarte w pakiecie `util-linux` ). W ten sposób na maszynie
wielordzeniowej, konkretny proces będzie wykorzystywał tylko te rdzenie, do których zostanie
przypisany. Wystarczy tylko, że określimy numer/numery rdzeni (numerowane od 0). I tak dla przykładu
jeśli chcemy, by proces firefox'a działał na pierwszym, drugim, piątym i szóstym rdzeniu, to w
terminalu wpisujemy poniższe polecenie:

    # taskset -c 0,1,4,5 firefox

W przypadku, gdy chcemy zmienić przydział rdzeni procesora dla uruchomionego już procesu, musimy
określić ID procesu. Poniżej przykład:

    # taskset -a -p -c 1 82457
    pid 15538's current affinity list: 0,1
    pid 15538's new affinity list: 0

Za sprawą parametru `-a` wszystkie wątki powiązane z numerem PID procesu zostaną automatycznie
przyporządkowane konkretnym rdzeniom.

## Zmiana priorytetu procesów do zasobów sieciowych (trickle)

Narzędzie `trickle` (dostępne w pakiecie o tej samej nazwie) zajmuje się ograniczaniem pasma
sieciowego dla danego procesu. Jednak trzeba uważać tutaj, by nie ustawić zbyt niskiej wartości.
Jeśli taka sytuacja nam się przytrafi, to wtedy odpalany program zacznie sypać segfault'ami.
Poniżej jest przykład nadania ograniczeń sieciowych przykładowemu procesowi:

    # trickle -s -d 500 -u 100 qbittorrent

Za pomocą przełącznika `-d` określamy prędkość pobierania danych z internetu. Z kolei przy pomocy
opcji `-u` definiujemy ograniczeniu upload'u. Obie wartości są w KiB/s
