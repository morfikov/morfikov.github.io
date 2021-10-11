---
author: Morfik
categories:
- Linux
date: "2015-06-18T22:11:45Z"
date_gmt: 2015-06-18 20:11:45 +0200
published: true
status: publish
tags:
- hdd
- ssd
- swap
GHissueID: 140
title: Przestrzeń wymiany SWAP jako plik
---

Każdy system operacyjny musi być przygotowany na ewentualność wyczerpania się pamięci operacyjnej
RAM. W przypadku gdyby nie był, groziłoby mu powieszenie się. Są różne metody ochrony maszyn z
linuxami na pokładzie przed tego typu sytuacją. Jedne z nich zakładają wykorzystanie wbudowanych w
kernel mechanizmów takich jak choćby [oom-killer](https://pl.wikipedia.org/wiki/Brak_pami%C4%99ci),
który ma za zadanie zabijać te najbardziej żarłoczne procesy. Są również bardziej łagodne sposoby na
uchronienie komputera przed zbyt szybkim wyczerpaniem się pamięci i w tym wpisie omówimy sobie
przestrzeń wymiany SWAP.

<!--more-->
## SWAP -- plik czy partycja?

Przestrzeń wymiany SWAP w linuxie można określić na dwa sposoby. Pierwszym z nich jest wydzielenie
całej partycji, drugim zaś jest zwykły plik, do którego będą zapisywane dane niemieszczące się w
pamięci RAM. W przypadku gdy posiadamy dużą ilość pamięci operacyjnej, raczej nie musimy zawracać
sobie głowy przestrzenią wymiany. No chyba, że zamierzamy hibernować system, wtedy nie ma innego
wyjścia. Natomiast jeśli chodzi o samo działanie systemu, to te z większą ilością pamięci nie
potrzebują przestrzeni wymiany.

Czasem jednak zdarza się tak, że raz na jakiś czas potrzebujemy zabezpieczenia w formie SWAP i co w
przypadku gdy przeszliśmy etap partycjonowania i nie uwzględniliśmy tam przestrzeni wymiany? Możemy
przejść cały proces jeszcze raz, możemy też wydzielić trochę miejsca poprzez skurczenie którejś z
istniejących już partycji. Możemy także utworzyć przestrzeń wymiany jako zwykły plik.

Posiadanie przestrzeni SWAP w formie partycji jak i zwykłego pliku ma swoje zalety i wady. Jeśli
zdecydowaliśmy się na partycję SWAP, to będzie nam ona szybciej działać. Jeśli wolimy plik, to nie
musimy się bawić partycjami ale za to dostęp do danych będzie wolniejszy no i istnieje możliwość
fragmentacji. Tak czy inaczej, tutaj skupimy się jak ze zwykłego pliku zrobić przestrzeń wymiany, bo
to może sprawiać jakieś problemy.

## Procedura przygotowania pliku

Procedura jest prosta. Musimy stworzyć plik o określonym rozmiarze, a następnie wskazać go
systemowi, w taki sposób, by go używał jako przestrzeni wymiany. Do utworzenia pliku posłużymy się
programem `dd` . W zależności od tego ile przestrzeni potrzebujemy, plik może przyjąć większy lub
mniejszy rozmiar. W przykładzie poniżej, ma on 512MiB:

    # dd if=/dev/zero of=/swap.512 bs=1M count=512

Mając przygotowany plik, musimy go odpowiednio sformatować. Do tego celu służy narzędzie `mkswap` i
jest ono takie samo co w przypadku tworzenia zwykłej partycji:

    # mkswap /swap.512

Teraz już tylko pozostało nam uaktywnienie tej przestrzeni wymiany:

    # swapon /swap.512

Oczywiście takie manualne operowanie na tym pliku nie jest wygodne. Możemy zautomatyzować cały
proces definiując odpowiedni wpis w pliku `/etc/fstab` :

    /swap.512  swap swap defaults,pri=10 0 0

Od tego momentu, ten plik będzie automatycznie montowany w systemie jako przestrzeń wymiany.
