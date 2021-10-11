---
author: Morfik
categories:
- Linux
date: "2016-02-04T23:36:24Z"
date_gmt: 2016-02-04 22:36:24 +0100
published: true
status: publish
tags:
- systemd
- hibernacja
GHissueID: 516
title: Mechanizm inhibitor lock w systemd
---

Hibernacja w przypadku komputerów, zwłaszcza laptopów, to bardzo użyteczny wynalazek. Na linux'ach
wymaga ona czasem lekkiej konfiguracji ale generalnie można powiedzieć, że działa OOTB. Po migracji
szeregu dystrybucji na systemd, proces hibernacji zdaje się przebiegać nieco inaczej niż to miało
miejsce w przeszłości. Bardzo często możemy się spotkać z sytuacjami, gdzie przy próbie
zahibernowania czy wyłączenia systemu, ten zwyczajnie ignoruje nasze żądanie i pracuje dalej jak
gdyby nigdy nic. W tym przypadku nie mamy do czynienia z bug'iem ale ficzerem. Okazuje się bowiem,
że systemd dysponuje mechanizmem zwanym [inhibitor
lock](https://www.freedesktop.org/wiki/Software/systemd/inhibit/), który ma na celu powstrzymanie
systemu od dokonania hibernacji, uśpienia lub wyłączenia w pewnych określonych sytuacjach. W tym
wpisie przyjrzymy się nieco bliżej temu mechanizmowi.

<!--more-->
## Po co nam ten cały inhibitor lock

Z początku można mieć wrażenie, że inhibitor lock, to zbędna i nikomu niepotrzebna funkcjonalność.
Można założyć, że skoro dany użytkownik chce zahibernować swój system, to nie powinien być przed tym
powstrzymywany, zwłaszcza przez system. Jakie są więc sytuacje, w których ten mechanizm powstrzyma
nas przed dokonaniem hibernacji? Weźmy na przykład płytki CD/DVD, a konkretnie proces ich zapisu
przez nagrywarkę. Czy w takim przypadku ktoś chciałby dokonać hibernacji systemu? Świadomie, raczej
może niekoniecznie. A co w przypadku, gdy użytkownik nie zwrócił uwagi na stan rozładowania
akumulatora i puścił proces nagrywania? Jeśli stopień rozładowania baterii spadnie poniżej pewnego
określonego poziomu, system sam dokona hibernacji lub będzie próbował się wyłączyć. Efektem takiego
działania będzie uszkodzony nośnik CD/DVD. Inhibitor lock w stytemd uchroni nas przed takim stanem
rzeczy. Oczywiście w dalszym ciągu grozi nam wyłączenie się maszyny za sprawą braku energii.
Niemniej jednak, system zwykle nie potrafi podać dokładnego stanu baterii. Weźmy jako przykład
mojego laptopa. Debian, który jest na nim zainstalowany, czasem myśli, że stan rozładowania
akumulatora jest bliski 0%. No i faktycznie, wszystkie czynniki na to wskazują. Problem w tym, że
gdy wybija 0%, to ten laptop jest w stanie jeszcze pracować dobre 30 minut. Zatem nie zawsze można
też dokładnie oszacować ile czasu faktycznie nam zostało i czy ta czynność, którą zamierzamy
przeprowadzić zdąży się dokonać nim minie czas.

Innymi przykładami mogą być aktualizacje systemu, gdzie przecie też nie chcielibyśmy zwykle
przerywać procesu instalowania poprawek. Dalej mamy różnego rodzaju pakiety biurowe, które podczas
zamykania systemu chciałyby automatycznie zapisać wszystkie otwarte dokumenty, by zmiany w nich
wprowadzone nie przepadły. Ten inhibitor lock jest też przydatny w przypadku przeglądarek
internetowych, gdzie podczas hibernacji, system nie zapełni przestrzeni wymiany SWAP cache'm stron,
które ta przeglądarka zassała przy przeglądaniu internetu. W tym przypadku ten mechanizm pozwoli,
np. Firefox'owi, na opróżnienie buforów, co sprawi, że hibernacja dokona się szybciej. No i na samym
końcu mamy przykład poruszający kwestię bezpieczeństwa systemu, czyli blokadę ekranu. W jej
przypadku system dokona hibernacji dopiero po zablokowaniu ekranu. W takim przypadku, po
odhibernowaniu systemu, będziemy musieli wprowadzić hasło w celu zalogowania się w systemie.

Także widzimy, że mamy bardzo wiele aspektów codziennej pracy na komputerze, w których ten inhibitor
lock może znaleźć zastosowanie. Dlatego też jeśli nasz system ma problemy z hibernacją czy
wyłączeniem się, np. po przyciśnięciu przycisku poweroff, to warto rzucić okiem na ten mechanizm
blokady i sprawdzić czy, aby jakaś aplikacja nie powstrzymuje go przed dokonaniem tej akcji.

## Konfiguracja inhibitora

Mamy dwa tryby pracy inhibitora. Pierwszy z nich to `block` , który blokuje akcję hibernacji czy
wyłączenia systemu. Jeśli to on zostanie aktywowany, to akcja zwyczajnie się nie powiedzie. Drugim
trybem zaś jest `delay` , który jedynie opóźnia cały proces przełączenia stanów. Opóźnienie takie
zwykle trwa do momentu zdjęcia blokady przez aplikację, która ją nałożyła. Czas opóźnienia może
także zostać dostosowany w pliku `/etc/systemd/logind.conf` za pomocą parametru
`InhibitDelayMaxSec` . Mamy też kilka typów inhibitorów:

  - `sleep` -- zapobiega hibernacji lub uśpieniu systemu przez nieuprzywilejowanych użytkowników.
  - `shutdown` -- zapobiega wyłączeniu lub zresetowaniu systemu przez nieuprzywilejowanych
    użytkowników.
  - `idle` -- zapobiega automatycznemu przejściu w stan bezczynności (tryb IDLE). Zwykle aktywowany
    automatycznie przez sam system, co skutkuje wyłączeniem lub uśpieniem maszyny.
  - `handle-power-key` , `handle-suspend-key` , `handle-hibernate-key` , `handle-lid-switch` --
    sprawiają, że `logind` nie będzie zarządzał przyciskami zasilania, hibernacji, uśpienia oraz
    przełączaniem stanu pokrywy laptopa.

Blokada akcji wywoływanych przez wciśnięcie fizycznych przycisków na obudowie/klawiaturze lub za
sprawą pokrywy laptopa może być respektowana albo też ignorowana przez systemd. Zależy to od
ustawień określonych w pliku `/etc/systemd/logind.conf` . Mamy tam te cztery poniższe wpisy:

    #PowerKeyIgnoreInhibited=no
    #SuspendKeyIgnoreInhibited=no
    #HibernateKeyIgnoreInhibited=no
    #LidSwitchIgnoreInhibited=yes

W przypadku ustawienia wartości `yes` , akcja przypisana do tych przycisków zawsze zostanie
wykonania i bez znaczenia jest tutaj fakt, że dana aplikacja założyła blokadę. Zatem jeśli chcemy
się pozbyć mechanizmu inhibitor lock, to tutaj możemy przestawić wartości tych powyższych opcji na
`yes` oraz wydać w terminalu to poniższe polecenie:

    # systemctl restart systemd-logind.service

## Jak sprawdzić, który inhibitor lock jest aktywny

W przypadku, gdy mechanizm inhibitor lock przypadł nam do gustu ale w pewnych sytuacjach doprowadza
nas do szału, to zawsze możemy pokusić się o ustalenie procesów, które powstrzymują nasz system od
hibernacji czy też wyłączenia się. Służy do tego narzędzie `systemd-inhibit` , które jest nam w
stanie zwrócić listę inhibitorów zarówno w trybie `block` jak i `delay` , przykładowo:

    # systemd-inhibit --list --mode=block
    0 inhibitors listed.
    
    # systemd-inhibit --list --mode=delay
         Who: Screen Locker (UID 1000/morfik, PID 2143/light-locker)
        What: sleep
         Why: Lock the screen on suspend/resume
        Mode: delay
    
    1 inhibitors listed.

W tym przypadku mamy tylko jeden inhibitor i do tego w trybie `delay` . Każda taka pozycja na liście
składa się z 4 linijek:

  - `Who` -- krótki opis programu, który założył tę blokadę.
  - `What` -- oddzielona przecinkami lista operacji, które mają być opóźnione lub blokowane przez
    ten inhibitor.
  - `Why` -- powód, dla którego ten inhibitor jest aktywny.
  - `Mode` -- to tryb pracy inhibitora.
