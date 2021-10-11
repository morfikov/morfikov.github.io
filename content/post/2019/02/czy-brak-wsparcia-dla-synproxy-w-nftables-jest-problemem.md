---
author: Morfik
categories:
- Linux
date: "2019-02-16T10:20:52Z"
published: true
status: publish
tags:
- kernel
- nftables
- iptables
- synproxy
- tcp
GHissueID: 307
title: Czy brak wsparcia dla SYNPROXY w nftables jest problemem
---

Przenosząc swoje reguły z `iptables` na `nftables` zauważyłem, że jedna z nich (gdyby tylko jedna)
nie została przetłumaczona przez ten dedykowany translator do reguł. Chodzi
o [mechanizm SYNPROXY][1], który jest zwykle wykorzystywany do ograniczenia skali ataków DDOS z
wykorzystaniem pakietów SYN. Co by nie mówić, to ochrona jaką daje SYNPROXY jest jak najbardziej
pożądana z perspektywy serwerów. Dlaczego zatem, gdy się zajrzy na stronę
[wspieranych rzeczy w nftables][2], to przy SYNPROXY widnieje bliżej nieokreślone sformułowanie
`consider native interface` ? Po rozmowach z deweloperami udało się ustalić, że ten zapis oznacza
brak wsparcia dla SYNPROXY w `nftables` . Jeśli zatem ktoś wykorzystuje ten mechanizm mając dodane
stosowne reguły w `iptables` , to czy powinien się on obawiać przejścia na `nftables` ?

<!--more-->
## Kernel 4.4 i tcp/dccp lockless listener

Do 2016 roku, maszyna z linux'em na pokładzie mogła mieć spore problemy z zagrożeniami jakie niósł
ze sobą atak SYN flood (i jemu podobne). Do tamtej pory przykre konsekwencje tego typu ataków DDOS
był w stanie złagodzić SYNPROXY. Odbywało się to na zasadzie niedopuszczania nowych połączeń (zanim
zostały ustanowione) do mechanizmu śledzenia połączeń (conntrack). W ten sposób cenne zasoby
maszyny nie były przeznaczane na obsługę połączeń, które mogły nawet nigdy nie zostać nawiązane.

Okazuje się jednak, że korzystanie z mechanizmu SYNPROXY w obecnych czasach jest pozbawione sensu.
Chodzi generalnie o fakt, że kernel linux'a od jakiegoś już czasu, a konkretnie od końcówki 2015
roku zyskał nowy mechanizm ochronny zwany [tcp/dccp lockless listener][3]. Dzięki niemu udało się
podbić ilość przetwarzanych pakietów SYN w ciągu sekundy o 2-3 rzędów wielkości w stosunku do tego,
co było do momentu wprowadzenia tego lockless listener'a.

Jeśli zatem nasz linux'owy serwer posiada kernela z numerkiem co najmniej 4.4, który został
wypuszczony w styczniu 2016 roku, to możemy sobie całkowicie odpuścić stosowanie SYNPROXY, bo
żadnej dodatkowej ochrony ten mechanizm nam nie zapewni.

Mając na uwadze powyższe informacje, jeśli do tej pory korzystaliśmy z `iptables` i czynnikiem
blokującym nas przed migracją na `nftables` był właśnie brak wsparcia SYNPROXY w tej nowszej wersji
framework'a, to teraz już nic nie stoi na przeszkodzie, by się na `nftables` przenieść. Warto też
zaznaczyć w tym miejscu, że korzystanie z SYNPROXY w przypadku `iptables` również nie ma większego
sensu i jeśli tylko dysponujemy kernelem 4.4+ , to z powodzeniem możemy pozbyć się tego całego
SYNPROXY.

## Wsparcie dla SYNPROXY w nftables v0.9.2

[Od wersji 0.9.2, nftables posiada już wsparcie dla SYNPROXY][4] i jeśli zamierzamy z niego
korzystać, to nic nie stoi na przeszkodzie by to uczynić. Trzeba jednak dodać stosowne reguły.
Poniżej jest zamieszczony przykład wyciągnięty z commit'a git:

    table ip x {
        chain y {
            type filter hook prerouting priority raw; policy accept;
            tcp dport 8888 tcp flags syn notrack
        }

        chain z {
            type filter hook input priority filter; policy accept;
            tcp dport 8888 ct state invalid,untracked synproxy mss 1460 wscale 7 timestamp sack-perm
            ct state invalid drop
        }
    }



[1]: /post/unikanie-atakow-ddos-z-synproxy/
[2]: https://wiki.nftables.org/wiki-nftables/index.php/Supported_features_compared_to_xtables
[3]: https://lwn.net/Articles/659199/
[4]: https://git.netfilter.org/nftables/commit/?id=1188a69604c3df2a63daca9e735fdb535e8f6b63
