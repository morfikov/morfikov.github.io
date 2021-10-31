---
author: Morfik
categories:
- Linux
date: "2016-02-04T16:57:37Z"
date_gmt: 2016-02-04 15:57:37 +0100
published: true
status: publish
tags:
- sieć
- ipv6
- 6to4
- sysctl
GHissueID: 518
title: Implementacja protokołu IPv6 za pomocą tunelu 6to4
---

Ogromna cześć lokalnych ISP zdaje się nie nadążać za ciągle zmieniającą się rzeczywistością. Problem
dotyczy implementacji protokołu IPv6, który jest już z nami od bardzo wielu lat. W przypadku mojego
obecnego ISP raczej nie mam co liczyć na to, by w bliżej nieokreślonej przyszłości dodał on obsługę
tego protokołu. Istnieje jednak mechanizm zwany [tunelowaniem pakietów protokołu IPv6 wewnątrz
pakietów protokołu IPv4](https://pl.wikipedia.org/wiki/6to4) (w skrócie 6to4), którym warto się
zainteresować. W ogromnym skrócie, część puli adresowej IPv6 jest zarezerwowana i zmapowana na
adresy protokołu IPv4. Dzięki takiemu podejściu, każdy kto posiada stały zewnętrzny adres IPv4 ma
również adres w puli IPv6. Mając zatem zarezerwowany adres, możemy pokusić się o utworzenie tunelu
6to4, co aktywuje w naszej infrastrukturze ten nowszy protokół obchodząc jednocześnie ograniczenia
ISP. Trzeba jednak mieć na względzie, że nie jest to natywne wsparcie dla protokołu IPv6 i jest
niemal pewne, że wystąpią mniejsze lub większe problemy z wydajnością.

<!--more-->
## Czy mój linux obsługuje IPv6

Obsługa protokółu IPv6 jest standardowo włączona w kernelu linux'a i raczej nie musimy przeprowadzać
żadnych dodatkowych czynności konfiguracyjnych, by ten protokół aktywować. Jeśli jednak nie jesteśmy
pewni czy mamy poprawnie skonfigurowany system, to warto rzucić okiem na opcje zwracane w `sysctl` .
Interesują nas głównie te poniższe pozycje:

    net.ipv6.conf.all.disable_ipv6 = 0
    net.ipv6.conf.default.disable_ipv6 = 0
    net.ipv6.conf.all.forwarding = 0

Warto też sprawdzić konfigurację bootloader'a pod kątem parametru `ipv6.disable=1` lub
`ipv6.disable_ipv6=1` . Pierwszy z tych parametrów wyłącza całkowicie stos IPv6, drugi jedynie
adresację.

Przeglądarki internetowe również mogą być skonfigurowane w taki sposób, by np. nie dokonywać zapytań
DNS o rekordy AAAA. W Firefox'ie warto zajrzeć do `about:config` i upewnić się, czy opcja
`network.dns.disableIPv6` nie jest przypadkiem ustawiona na `true` . A skoro już mowa o zapytaniach
DNS, to trzeba się upewnić, że resolver DNS, z którego korzystamy, jest w stanie rozwiązywać nazwy
na adresy IPv6. W sporej części globalnych providerów DNS, takich jak google czy OpenDNS, nie musimy
się o nic martwić. Problemem mogą być serwery DNS lokalnych ISP i jeśli znajdujemy się w takiej
sytuacji, to musimy zmienić konfigurację resolver'a w pliku `/etc/resolv.conf` .

Te powyższe kroki są konieczne, by zaimplementować tunel 6to4, niemniej jednak, same z siebie nie
włączą nam obsługi protokołu IPv6. Możemy się o tym przekonać [testując swoje
połączenie](http://test-ipv6.com/). Jeśli nasz wynik jest podobny do tego poniżej, to nic nie
szkodzi:

![brak-polaczenia-ipv6-test](/img/2016/02/1.brak-polaczenia-ipv6-test.png#huge)

## Tunel 6to4

By zaimplementować sobie protokół IPv6 musimy skonfigurować tunel 6to4. Nie będziemy tutaj korzystać
z graficznych automatów typu network-manager. Interesuje nas głównie plik
`/etc/network/interfaces` , do którego musimy dodać konfigurację dla interfejsu `6to4` . Oczywiście
[możemy taki tunel
utworzyć ręcznie](http://tldp.org/HOWTO/html_single/Linux+IPv6-HOWTO/#configuring-ipv6to4-tunnels)
za pomocą narzędzi `ip` czy `ifconfig` ale my posłużymy się tym powyższym plikiem. Poniżej jest
przykładowa konfiguracja:

    auto lo
    iface lo inet loopback

    allow-hotplug enp2s0
    iface enp2s0 inet dhcp

    auto 6to4
    iface 6to4 inet6 6to4
          local 82.120.56.22

Nas głównie interesuje ostatnia zwrotka, gdzie mamy podany adres IP. Ten adres musi wskazywać na
nasz zewnętrzny stały adres IPv4. Standardowo też w systemie nie mamy interfejsu `6to4` . Zostanie
on stworzony przy podnoszeniu tego interfejsu via `ifup` (zostanie załadowany moduł `sit` ).
Zapisujemy zatem zmiany i w terminalu wpisujemy to poniższe polecenie:

    # ifup 6to4

Po chwili, narzędzia `ip` oraz `ifconfig` powinny widzieć nowy interfejs. Powinien on także posiadać
stosowną konfigurację i zawierać dwa adresy, przykładowo:

    # ip addr show 6to4
    13: 6to4@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default
        link/sit 82.120.56.22 brd 0.0.0.0
        inet6 2002:5278:3816::1/16 scope global
           valid_lft forever preferred_lft forever
        inet6 ::82.120.56.22/96 scope global
           valid_lft forever preferred_lft forever

Wyżej widzimy adres `2002:5278:3816::1/16` . Nie jest on przypadkowy i jeśli odpowiednio przeliczymy
go na system dziesiętny, to powinniśmy otrzymać taki sam adres co w przypadku IPv4. Interesuje nas
tylko `5278:3816` , gdzie mamy 4 liczby w zapisie HEX: `52` , `78` , `38` oraz `16` . Odpowiadają
one poszczególnym oktetom adresu IPv4.

By zobaczyć tablicę routing'u, wpisujemy w terminalu to poniższe polecenie:

    # ip -6 route show
    ::/96 dev 6to4  proto kernel  metric 256  pref medium
    2002::/16 dev 6to4  proto kernel  metric 256  pref medium
    2000::/3 via ::192.88.99.1 dev 6to4  metric 1024  pref medium

Ten adres `192.88.99.1` widoczny wyżej, to adres routera, który robi za bramkę między sieciami IPv4
i IPv6. Nie jest on przypadkowy i został wyraźnie określony w
[rfc3068](https://tools.ietf.org/html/rfc3068).

## Test połączenia IPv6

Na dobrą sprawę, to jest cała konfiguracja tunelu 6to4 w debianie. Sprawdźmy zatem czy tunel
realizuje swoje zadanie:

![test-polaczenia-ipv6-tunel-6to4](/img/2016/02/2.test-polaczenia-ipv6-tunel-6to4.png#huge)

Wygląda, że działa. W prawdzie ocena połączenia jest 7/10 ale lepsze to niż 0/10. Możemy także
sprawdzić dodatkowe informacje dotyczące czasów:

![wydajnosc-polaczenia-ipv6-tunel-6to4](/img/2016/02/3.wydajnosc-polaczenia-ipv6-tunel-6to4.png#big)

## Oprogramowanie, a protokół IPv6

W przypadku wszystkich tych linux'owych narzędzi sieciowych trzeba brać poprawkę jeśli zamierzamy
korzystać z protokołu IPv6. W taki sposób, zamiast polecenia `ping` korzystamy z `ping6` , zamiast
`traceroute` dajemy `traceroute6` .

Podobnie sprawa ma się w przypadku `ssh` , gdzie połączenie odbywa się w poniższy sposób:

    $ ssh -6 2002:5278:3816::1

Możemy naturalnie podać domenę ale jeśli chcemy zmusić `ssh` do połączenia nas via IPv6, to musimy w
pliku `~/.ssh/config` dodać poniższy blok:

    Host domena.com 2002:5278:3816::1
          AddressFamily inet6
          User morfik
          IdentitiesOnly yes
          IdentityFile ~/.ssh/key_rsa
          CheckHostIP yes
          Port 6666

Pamiętajmy też o odpowiedniej konfiguracji zapory. W przypadku protokołu IPv6 nie korzystamy z
`iptables` tylko z `ip6tables` .
