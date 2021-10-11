---
author: Morfik
categories:
- Linux
date: "2016-08-03T13:41:52Z"
date_gmt: 2016-08-03 11:41:52 +0200
published: true
status: publish
tags:
- ssh
- sysctl
- terminal
GHissueID: 570
title: 'packet_write_wait: Connection to IP port 22: Broken pipe'
---

Operowanie na VPS nie jest jakoś specjalnie trudne, zwłaszcza w przypadku, gdy mamy dostęp root i
możemy logować się na serwer z wykorzystaniem protokołu SSH. Dalej to już zwykła linux'owa
mechanika, która może być nieco inna, w zależności od tego, jaki dokładnie system operacyjny na tym
VPS stoi. Czasami jednak, w pewnym momencie podczas połączenia możemy zostać rozłączeni z
niewiadomych nam przyczyn. Niemniej jednak, zawsze, gdy ten problem występuje, w terminalu można
zobaczyć komunikat: `packet_write_wait: Connection to 1.2.3.4 port 22: Broken pipe` . Przydałoby się
zatem coś na ten stan rzeczy poradzić.

<!--more-->
## Skąd się bierze komunikat "Broken pipe"

Przede wszystkim, połączenia SSH mogą przechodzić w stan IDLE, czyli być bezczynne. W takiej
sytuacji, użytkownik zalogowany po SSH nie przesyła żadnych poleceń do serwera. Podobnie w drugą
stronę, czyli serwer nie przesyła nam żadnych komunikatów, które mają zostać wyświetlone w
terminalu. Innymi słowy, nie przesyłamy żadnych informacji ale połączenie jest w dalszym ciągu
otwarte.

Połączenia w stanie IDLE mogą zostać uszkodzone tworząc tym samym połączenia zombie i by jakoś sobie
z nimi radzić stworzono mechanizm zwany `keepalive` , który polega na wysyłaniu pustych segmentów
(niezawierających żadnych danych) między dwoma punktami komunikacji. Jeśli połączenie działa, druga
strona odpowie pakietem `ACK` . Z kolei jeśli połączenie jest martwe, to zostanie zresetowane
pakietem `RST` , przynajmniej tak to wygląda w największym skrócie. Niemniej jednak, połączenie nie
musi być wcale martwe. W sieci ciągle giną pakiety i to z różnych powodów. Jeśli taki pakiet
`keepalive` nam zaginie, np. z winy słabej jakości łącza (częste w przypadku LTE i WiFi), to serwer
błędnie uzna, że połączenie powinno zostać zakończone. My zaś dostaniemy w terminalu ten cały
komunikat `packet_write_wait: Connection to 1.2.3.4 port 22: Broken pipe` .

## Jak wyeliminować "Broken pipe"

Najlepszym sposobem na wyeliminowanie komunikatu `packet_write_wait: Connection to 1.2.3.4 port 22:
Broken pipe` jest poprawa jakości połączenia. Jeśli jednak korzystamy z technologi bezprzewodowych i
do tego jeszcze mamy niezbyt przyzwoitą lokalizację, to musimy poinstruować system, by zmienił
zachowanie w kwestii pakietów `keepalive` .

[Kernel linux'a dysponuje kilkoma
parametrami](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt), które możemy sobie
dostosować, by zwiększyć częstotliwość przesyłania pakietów `keepalive` i tolerancję na ich
ewentualne zagubienie. Są to te poniższe opcje, które wystarczy dodać do pliku `/etc/sysctl.conf` :

    net.ipv4.tcp_keepalive_time = 300
    net.ipv4.tcp_keepalive_intvl = 60
    net.ipv4.tcp_keepalive_probes = 3

Dwa pierwsze parametry są wyrażone w sekundach. Ostatni zaś to zwykła liczba. Powyższa konfiguracja
wyśle pierwszy pakiet `keepalive` po 5 minutach (300s). Jeśli zostanie na niego udzielona odpowiedź
`ACK` , kolejny pakiet `keepalive` zostanie wysłany po kolejnych 5 minutach, itd. Natomiast w
przypadku nieudzielenia odpowiedzi `ACK` , serwer zwiększy częstotliwość generowania pakietów
`keepalive` z 300 sekund do 60 sekund. Jeśli na kolejne pakiet `keepalive` klient nie będzie zwracał
pakietów `ACK` , to po 3 próbach pod rząd, takie połączenie zostanie oznaczone jako uszkodzone i
zresetowane.

## Ustawienie parametru ServerAliveInterval w ~/.ssh/config

W przypadku, gdy nie mamy możliwości edycji pliku `/etc/sysctl.conf` , np. brak dostępu do konta
administratora, to możemy skonfigurować sobie klienta SSH i ustawić w nim parametr
`ServerAliveInterval` . W tym celu dodajemy do pliku `~/.ssh/config` tę poniższą zwrotkę:

    Host *
        ServerAliveInterval 30

Wartość `30` w parametrze `ServerAliveInterval` trzeba sobie dostosować w zależności od jakości
łącza. Domyślnie ta wartość jest ustawiona na `0` , czyli żadne pakiety `keepalive` nie są
wysyłane, przez co polegamy jedynie na mechanizmach protokołu TCP. W przypadku ustawienia wartości
`30`, klient będzie przesyłał do serwera pakiety `keepalive` co 30 sekund od momentu, gdy połączenie
SSH przejdzie w stan IDLE.

Różnica między tymi pakietami `keepalive` , a tymi z protokołu TCP, polega na tym, że pakiety
generowane przez SSH są wrzucane w szyfrowany kanał SSL/TLS. Są one zatem traktowane jak zwykłe dane
w protokole TCP, w stosunku do pustego pakietu z ustawioną flagą `ACK` .
