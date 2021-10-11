---
author: Morfik
categories:
- Linux
date: "2015-10-29T01:57:30Z"
date_gmt: 2015-10-28 23:57:30 +0100
published: true
status: publish
tags:
- sysctl
- kernel
GHissueID: 188
title: Aktywacja i konfiguracja klawisza SysRq
---

SysRq (System Request) to klawisz na klawiaturze, po którego przyciśnięciu można wysłać
niskopoziomowe zapytana bezpośrednio do kernela linux'a. Te komendy działają nawet w przypadku
pozornego braku kontaktu z systemem operacyjnym, tj. zacięcia dźwięku, nieruchomy kursor myszy, a
nawet w przypadku braku możliwości wpisywania znaków z klawiatury. Zwykle po opisanych wyżej
symptomach, człowiek jest skłonny przycisnąć przycisk reset na obudowie swojego komputera, no bo jak
inaczej odwiesić taki system? Problem z twardym resetem (za pomocą przycisku) jest taki, że
praktycznie zawsze po nim występuje uszkodzenie struktury systemu plików na dysku, a czasami
uszkodzeniu ulega cała partycja. To niesie ze sobą ryzyko utraty danych. Dlatego też powinniśmy
zaprzestać resetowania komputerów przy pomocy przycisków i zacząć korzystać z klawisza SysRq .

<!--more-->
## Gdzie jest klawisz SysRq i jak go używać

Jeśli popatrzymy na swoją klawiaturę, prawdopodobnie nie dostrzeżemy na niej klawisza z napisem
SysRq . Przynajmniej na mojej klawiaturze ten klawisz jest inaczej podpisany. Mianowicie jest to
Print Screen . By teraz zacząć wydawać kernelowi polecenia, trzeba wcisnąć dodatkowy klawisz, tj.
L-Alt . Mając wciśnięte te dwa klawisze jednocześnie, można w końcu przycisnąć jeszcze jeden i ta
kombinacja nakaże kernelowi by wykonał jakąś określoną operację.

Przy pomocy L-Alt-SysRq-S możemy zażądać wykonania polecenia `sync` , który zapisze wszystkie [dirty
pages](/post/ram-cache-i-dirty-pages/), które rezydują w pamięci operacyjnej RAM,
na dysk. Przy pomocy L-Alt-SysRq-U możemy przemontować wszystkie partycje w tryb tylko do odczytu.
Te dwa polecenia dbają o zabezpieczenie systemu plików. Inna użyteczna kombinacja to L-Alt-SysRq-B ,
który resetuje system. W taki oto sposób możemy bezpiecznie zrestartować komputer. Oczywiście takich
a poleceń SysRq jest o wiele więcej ale nie będę ich zamieszczał w tym wpisie, bo jest on o czymś
zupełnie innym. Te powyższe informacje, to tylko przykład do czego można wykorzystać klawisz SysRq .
Przejdźmy zatem do sedna sprawy.

## Konfiguracja klawisza SysRq

By skonfigurować klawisz SysRq musimy ustawić odpowiednio parametr kernela `kernel.sysrq` , bo
przyjmuje on kilka wartości. Mianowicie `0` i `1` odpowiednio wyłączają i włączają możliwość
korzystania z klawisza SysRq . Trzeba jednak zaznaczyć, że wartość `1` włącza pełną funkcjonalność
tego klawisza, tj. wszystkie akcje będzie można przeprowadzić po jego przyciśnięciu. W pewnych
okolicznościach może to zagrażać bezpieczeństwu systemu. Dlatego też zamiast ustawiać `1` ,
powinniśmy się zastanowić, których z poniższych akcji potrzebujemy, a które są nam zupełnie zbędne.
Każda z nich ma przypisaną wartość i gdy potrzebujemy kilku, to wartość, którą należy wpisać w
parametrze `kernel.sysrq` trzeba obliczyć dodając wartości poszczególnych akcji do siebie. Poniżej
lista akcji:

  - `2` -- włącza kontrolę poziomu logów konsoli `*`
  - `4` -- włącza kontrolę klawiatury (SAK, unraw) `*`
  - `8` -- włącza zrzuty procesów w celu debugowania, itd.
  - `16` -- włącza polecenie sync `*`
  - `32` -- włącza ponowne montowanie tylko do odczytu `*`
  - `64` -- włącza sygnały procesów (term, kill, oom-kill)
  - `128` -- pozwala na restartowanie/wyłączanie `*`
  - `256` -- pozwala ustawiać nice wszystkich zadań czasu rzeczywistego `*`

Przy niektórych pozycjach pojawił się znak `*` . Informuje on, że dana akcja jest włączona przy
domyślnej konfiguracji kernela w debianie. Dodając wartości tych opcji do siebie, uzyskujemy wynik
2+4+16+32+128+256=438 i to tę wartość powinniśmy uwzględnić w parametrze `kernel.sysrq` , który
należy dopisać do pliku `/etc/sysctl.conf` , przykładowo:

    kernel.sysrq = 438
