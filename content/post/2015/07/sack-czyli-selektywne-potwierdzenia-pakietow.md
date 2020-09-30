---
author: Morfik
categories:
- Linux
date: "2015-07-01T06:04:18Z"
date_gmt: 2015-07-01 04:04:18 +0200
published: true
status: publish
tags:
- tcp
- sysctl
- sieć
title: SACK, czyli selektywne potwierdzenia pakietów
---

Protokół TCP jest tak zbudowany by zapewnić rzetelny transfer danych między dwoma komunikującymi się
punktami. Z początku jednak, ta cecha tego protokołu powodowała marnowanie dość sporych ilości
zasobów jeśli chodzi o przepustowość łącza. Stało się to widoczne przy większych prędkościach
połączeń, gdzie [skalowany był bufor](/post/bufor-polaczen-w-protokole-tcp/)
(okno) TCP, co umożliwiło przesyłanie szeregu segmentów bez potrzeby czekania na ich potwierdzenie
przez odbiorcę. To zwiększyło, co prawda, transfer danych ale pojawił się problem z zagubionymi
pakietami.

<!--more-->
## Zwykły ACK

Gdy pakiet nie dociera do odbiorcy, to po pewnym czasie zostaje retransmitowany. Co się jednak
dzieje w przypadku wysłania na drugą stronę 100 segmentów? Jeśli zagubieniu ulegnie, dajmy na to,
trzeci z nich, to trzeba retransmitować 3,4-100. Mimo, że pakiety 4-100 trafiły do odbiorcy. Mało
tego, w sytuacji zgubienia się pakietu numer 3 i po dotarciu wszystkich następnych, odbiorca musi
wysłać zduplikowane pakiety `ACK` sygnalizując problem z jednym z wcześniejszych pakietów. Tych
duplikatów byłoby 98. Ten proces został zobrazowany poniżej
([źródło](http://www.tcpipguide.com/free/t_TCPNonContiguousAcknowledgmentHandlingandSelective.htm)):

![](/img/2015/06/1.retransmisja-pakietow-ack.png#big)

Niezła rozrzutność i tu właśnie w grę wchodzi mechanizm `SACK`, czyli selektywny ACK.

## SACK

Jak zatem wyglądałaby powyżej opisana sytuacja po zaimplementowaniu mechanizmu SACK? Popatrzmy na
kolejną fotkę
([źródło](http://www.tcpipguide.com/free/t_TCPNonContiguousAcknowledgmentHandlingandSelective.htm)):

![](/img/2015/06/2.retransmisja-pakietow-sack.png.png#big)

Odbiorca otrzymał pakiet 1, 2 oraz 4. Pakiet 3 uległ zagubieniu. Odbiorca potwierdza wszystkie
pakiety, które otrzymał. W buforze nadawczym, ciągle są dostępne te segmenty, które nie zostały
potwierdzone w sposób ciągły, czyli te które nie dotarły w ogóle lub dotarły poza kolejnością.
Natomiast odbiorca ma u siebie w buforze, te pakiety, które otrzymał poza kolejnością. Po chwili
serwer retransmituje zaginione pakiety. Widzi jednak, że część z nich jest oznaczona bitem SACK i te
zostawia w spokoju. Przesłaniu ulegają jedynie te, które nie mają ustawionego tego bitu. Serwer
oczyszcza flagę flagę SACK z pakietu znajdującego się w jego buforze. Po chwili zagubiony pakiet
trafia już do odbiorcy. Ten zaś wysyła pakiet potwierdzający ciągłość odebranych danych, tj. w
punkcie gdzie powinna rozpocząć się transmisja nowych danych i oba bufory zostają opróżnione.

Dzięki takiemu rozwiązaniu możemy znacznie zminimalizować ilość retransmitowanych danych, dlatego
też dobrze jest włączyć taką opcję w kernelu linuxa. Możemy to zrobić przez dopisanie do pliku
`/etc/sysctl.conf` tych poniższych linijek:

    net.ipv4.tcp_sack = 1
    net.ipv4.tcp_dsack = 1
    net.ipv4.tcp_fack = 1

W pewnych warunkach jednak, mogą pojawić się też problemy z wydajnością serwera. Atakujący może
zmusić maszynę do trzymania sporej kolejki na retransmisje pakietów przez długi czas, co może zjeść
procesor, pamięć RAM oraz więcej łącza niż w rzeczywistości powinno. Jeśli serwer jest solidny i nie
przesyła dużych plików, tego typu sytuacja mu nie zagraża. W przypadku gdy serwer ma na celu dbanie
o jak najniższe opóźnienia, SACK jest zbędny i może stanowić zagrożenie dla bezpieczeństwa maszyny.
W przypadku słabych i wolnych łącz, SACK może powodować problemy przez nasycenie łącza pakietami ACK
i powinien zostać wyłączony.
