---
author: Morfik
categories:
- Linux
date: "2016-06-09T18:05:00Z"
date_gmt: 2016-06-09 16:05:00 +0200
published: true
status: publish
tags:
- sieć
- dhcp
title: Konfiguracja interfejsów sieciowych w dhclient
---

Konfiguracja maszyn w sieci za sprawą protokołu DHCP znacznie ułatwia życie administratorom. Cały
ten proces jest nie tylko szybszy ale też eliminuje szereg błędów, które mogą pojawić się za sprawą
czynnika ludzkiego. W przypadku, gdy nasza maszyna dysponuje kilkoma interfejsami sieciowymi, to
każdy z nich możemy skonfigurować nieco inaczej. Oczywiście, nie chodzi o samą konfigurację
adresacji ale o szereg parametrów, które klient przesyła do serwera DHCP. To dzięki nim host min.
wie jak ustawić adresację na interfejsie i pod jaki adres słać zapytania DNS. Każdy interfejs może w
ten sposób posiadać własne opcje, które klient DHCP będzie przesyłał do serwera. W tym artykule
postaramy się konfigurować niezależnie dwa interfejsy sieciowe przy pomocy `dhclient` , czyli
domyślnego klienta DHCP w linux'ie.

<!--more-->
## Standardowa konfiguracja dhclient'a

W domyślnej konfiguracji debiana, `dhclient` traktuje wszystkie interfejsy tak samo. W taki sposób
jeśli podniesiemy interfejs `eth0` , to zostanie mu przypisana adresacja zgodnie z tym o co zapytał
ten klient DHCP. Podobnie sprawa ma się w przypadku interfejsu `wlan0` . Jak mogliśmy się przekonać
we wpisie dotyczącym [ukrywania nazwy hosta (hostname) w protokole
DHCP](/post/jak-ukryc-hostname-w-protokole-dhcp/), w niektórych sieciach nie zawsze
chcemy udostępniać pewne informacje. Mogą przecie one posłużyć do zidentyfikowania nas i oznaczenia
naszego ruchu w bliżej nieokreślonych celach. Z kolei konfiguracja globalnych opcji wpłynie na
wszystkie interfejsy dokładnie w taki sam sposób, co nie zawsze jest pożądane.

Konfiguracja dla `dhclient` jest trzymana w pliku `/etc/dhcp/dhclient.conf` . Zajrzyjmy do niego.
Standardowo mamy tam te poniższe linijki:

    option rfc3442-classless-static-routes code 121 = array of unsigned integer 8;

    send host-name = gethostname();
    request subnet-mask, broadcast-address, time-offset, routers,
        domain-name, domain-name-servers, domain-search, host-name,
        dhcp6.name-servers, dhcp6.domain-search, dhcp6.fqdn, dhcp6.sntp-servers,
        netbios-name-servers, netbios-scope, interface-mtu,
        rfc3442-classless-static-routes, ntp-servers;

Pierwsza z tych powyższych linijek definiuje nową opcję `rfc3442-classless-static-routes` , która
zgodnie z [RFC 3442](https://tools.ietf.org/html/rfc3442) umożliwia określanie statycznych tras w
notacji CIDR (bezklasowe) i przesyłanie ich w komunikatach protokołu DHCP. Jest to nowsza wersja
opcji o numerze 33.

Pozostałe linijki określają parametry globalne, które będą przesyłane w zapytaniu do serwera DHCP (
`send` ) i żądane ( `request` ) od niego w celu skonfigurowania adresacji hosta. Poszczególne opcje
są oddzielone od siebie przy pomocy `,` . Natomiast każda dyrektywa kończy się znakiem `;` . Powyżej
widzimy, że host przesyła do serwera DHCP nazwę hosta uzyskując ją via `gethostname()` i żąda od
tego serwera przesłania szeregu parametrów. Oczywiście żądanie nie oznacza, że parametr jest
wymagany. Jeśli serwer nie będzie w stanie zwrócić wartości części z powyżej ustawionych opcji, to
po prostu tego nie zrobi i wyśle to co ma. Oczywiście, nic nie stoi na przeszkodzie, by wymusić
pobranie danej opcji stosując dyrektywę `require` . Bez uzyskania opcji w tej dyrektywie, oferta
konfiguracji nie zostanie zaakceptowana przez klienta DHCP. Jeśli nie zamierzamy przesyłać żadnych
opcji w dyrektywie `request` , to bezpośrednio za jej nazwą wpisujemy `;` .

## Konfiguracja określonego interfejsu w dhclient

Mając kilka interfejsów możemy nadpisać wszystkie dyrektywy globalne i skonfigurować sobie każdy z
tych interfejsów indywidualnie. W ten sposób każda dyrektywa może posiadać inne opcje, w zależności
od interfejsu przez który zapytanie DHCP zostanie przesłane do serwera. By tego typu rozwiązanie
wdrożyć musimy utworzyć osobny blok `interface` i w nim zawrzeć całą konfigurację. Nie musimy
kasować globalnych opcji, bo one nie będą brane w ogóle pod uwagę. Poniżej jest przykład:

    interface "eth0" {
        supersede domain-name-servers 127.0.0.1;
        request subnet-mask, broadcast-address, time-offset, routers,
            domain-name,
            dhcp6.fqdn, dhcp6.sntp-servers,
            netbios-name-servers, netbios-scope, interface-mtu,
            rfc3442-classless-static-routes, ntp-servers;
    }

W powyższej konfiguracji przepisaliśmy sobie adres resolver'a DNS. Wszystkie dyrektywy, które nie
zostały uwzględnione w bloku `interface "eth0" {}` , a widnieją w sekcji globalnej, będą również
uwzględniane przy konfiguracji interfejsu. Dla przykładu, w dalszym ciągu zostanie wysłana nazwa
hosta do serwera DHCP. Opcji, które możemy ustawić w pliku `/etc/dhcp/dhclient.conf` jest sporo.
Dlatego też zachęcam do zapoznania się z [man
dhcp-options](http://manpages.ubuntu.com/manpages/xenial/en/man5/dhcp-options.5.html) oraz [man
dhclient.conf](http://manpages.ubuntu.com/manpages/xenial/en/man5/dhclient.conf.5.html).
