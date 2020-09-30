---
author: Morfik
categories:
- Linux
date: "2015-06-28T17:33:54Z"
date_gmt: 2015-06-28 15:33:54 +0200
published: true
status: publish
tags:
- ciasteczka
- tcp
- sysctl
title: Znacznik czasu (timestamp) w protokole TCP
---

O znacznikach czasu (timestamp) wspominałem już raz w ramach omawiania mechanizmu jakim jest [bufor
połączenia][1], a konkretnie rozchodziło się o skalowanie okien TCP. Generalnie rzecz biorąc, przy
wyższych prędkościach, rzędu 1 gbit/s, nie ma innej opcji jak skorzystanie z opcji znaczników czasu,
które są niejako rozszerzeniem czegoś co widnieje pod nazwą [numery sekwencyjne][2] . Z jednej
strony może i mamy możliwość implementacji łącz o większej przepustowości ale z drugiej te znaczniki
czasu w pakietach TCP [mogą zagrozić bezpieczeństwu][3] stacji roboczej.

<!--more-->
## Timestamp, a bezpieczeństwo systemu

Jak możemy przeczytać w podlinkowanym we wstępie artykule, jeśli jakaś maszyna korzysta z opcji
timestamp w nagłówku TCP, to jesteśmy w stanie obliczyć czas jej pracy (uptime). A jak wiadomo,
aktualizacja oprogramowania ma to do siebie, że by poprawki weszły w życie trzeba zresetować kilka
usług. Część z nich może wymagać ponownego uruchomienia maszyny i tu właśnie jest pewien problem.
Znając czas pracy komputera, możemy ocenić czy określone poprawki zostały nałożone na system i w
przypadku gdy czas pracy wychodzi poza datę wydania łaty, maszyna może być podatna na atak.

Nie chce za bardzo wchodzić w cały ten proces, a jedynie zastanowić się co ta opcja timestamp w
nagłówku TCP niesie dla zwykłego człowieka korzystającego z sieci. Jak czytamy dalej w linku,
nasza maszyna może zostać oznaczona sprzętowym ciasteczkiem już na poziomie warstwy transportowej,
dzięki czemu wszystkie boty będą mogły nas w łatwy sposób zidentyfikować.

To czy nasz system ma włączoną opcję znaczników czasowych możemy zweryfikować przez sprawdzenie
poniższego parametru:

    # sysctl -a | grep timestamp
    net.ipv4.tcp_timestamps = 1
    net.netfilter.nf_conntrack_timestamp = 1

Poniżej jest zaś przykładowy wynik skanowania przy pomocy `nmap` :

    # nmap -PN -O -v 192.168.10.100
    ...
    Uptime guess: 197.261 days (since Sat Dec 13 09:51:44 2014)
    ...

W moim przypadku wynik nie jest trafiony. Zgodnie z tym co pokazuje aktualnie `uptime` , to mam `1
day, 4:58,` . Także niezła rozbieżność. Pamiętam jak kiedyś poruszałem tę kwestię na [forum
dug'a][4]. Widać coś namieszali i nie idzie już tak łatwo wyciągnąć uptime.

Jeśli chodzi z jakichś powodów skanowanie wykrywa nasz prawdziwy uptime, to zawsze można się
posłużyć `iptables` i zablokować określone pakiety protokołu ICMP. Jeśli przejrzymy [tabelkę
typów komunikatów][5], to widzimy tam numerki `13` i `14` . Zatem komunikaty z żądaniem i
odpowiedzią timestamp można zablokować niczym zwykły ping przy poniższych regułek:

    iptables -A icmp_in -p icmp -m icmp --icmp-type 13 -m comment --comment timestamp-request -j DROP
    iptables -A icmp_out -p icmp -m icmp --icmp-type 14 -m comment --comment timestamp-reply -j DROP

I już żaden skaner nie wyciągnie od nas informacji o uptime naszego serwera.


[1]: /post/bufor-polaczen-w-protokole-tcp/
[2]: /post/numery-sekwencyjne-w-strumieniu-tcp/
[3]: https://nfsec.pl/security/2306
[4]: https://forum.dug.net.pl/viewtopic.php?pid=265657
[5]: https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xml
