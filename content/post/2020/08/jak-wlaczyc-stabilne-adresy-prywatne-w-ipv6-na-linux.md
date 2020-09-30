---
author: Morfik
categories:
- Linux
date:    2020-08-19 19:31:00 +0200
lastmod: 2020-08-19 19:31:00 +0200
published: true
status: publish
tags:
- debian
- sysctl
- ipv6
- sieć
- slaac
- prywatność
title: Jak włączyć stabilne adresy prywatne w IPv6 na linux
---

Jakiś już czas temu opisywałem [jak włączyć rozszerzenia prywatności IPv6 na Debianie][3] (IPv6
Privacy Extensions) w przypadku korzystania z mechanizmu automatycznej konfiguracji adresacji
hostów SLAAC (StateLess Address AutoConfiguration). Miało to za zadanie poprawić nieco prywatność
osób podłączonych do internetu za sprawą protokołu IPv6, bo generowane adresy IP standardowo
zawierają adresy MAC kart sieciowych ([identyfikator EUI64][9]). Parę dni temu dowiedziałem się, że
w linux można również aktywować inny mechanizm zwany stabilnymi adresami prywatnymi
([stable-privacy addresses][5]), które to wykorzystują inny system przy generowaniu identyfikatorów
interfejsów sieciowych. Ten mechanizm sprawia, że część adresu IPv6 odpowiedzialna za identyfikację
hosta ma losowe, choć stabilne wartości, które nie mają nic wspólnego z adresem MAC karty sieciowej
naszego komputera. W ten sposób możemy ukrócić śledzenie nas w sieci na podstawie adresu IPv6.
Poniższy artykuł ma za zadanie pomóc skonfigurować nam te stabilne adresy prywatne na linux oraz
pokazać w jaki sposób są one w stanie pomóc naszej prywatności.

<!--more-->
## Czym są stabilne adresy prywatne

Zadaniem stabilnych adresów prywatnych jest zapobieganie śledzeniu hostów (i ich użytkowników) w
sieci na podstawie adresu IPv6. Gdy podłączymy się do innej sieci (w domu, w biurze, czy też w
dowolnym innym miejscu), to zmianie ulega prefiks tej sieci wykorzystywany przy autokonfiguracji
SLAAC. Niemniej jednak, część adresu IPv6 odpowiedzialna za identyfikację hosta pozostaje taka sama
w każdej sieci, do której się podłączymy. W ten sposób serwisy WWW mogą nas bez problemu śledzić,
bo są one w stanie zidentyfikować konkretnego hosta podpiętego do sieci IPv6 po jego
identyfikatorze EUI64, który ma wbudowany adres sprzętowy MAC karty sieciowej. Jeśli teraz
zaimplementujemy sobie stabilne adresy prywatne, to zmianie ulegnie identyfikator interfejsu (IID,
Interface Identifier) za sprawą choćby zmiany identyfikatora sieci (np. ESSID w sieciach WiFi). W
ten sposób wynikowy identyfikator interfejsu będzie inny dla każdej z odwiedzanych przez nas sieci.
Inny identyfikator interfejsu oznacza również inny adres IPv6, bo część hosta za każdym razem
będzie ulegać zmianie, tj. dla każdej nowej sieci będzie ten identyfikator inny. Przynajmniej taka
jest teoria działania tego mechanizmu, bo jak możemy wyczytać w RFC7217, ten ficzer ze zmianą
adresu IPv6 w zależności od sieci, do której się podłączymy, jest jedynie opcjonalny:

> Network_ID:
>   Some network-specific data that identifies the subnet to which
>   this interface is attached -- for example, the IEEE 802.11
>   Service Set Identifier (SSID) corresponding to the network to
>   which this interface is associated.  Additionally, Simple DNA
>   [RFC6059] describes ideas that could be leveraged to generate
>   a Network_ID parameter.  **This parameter is OPTIONAL**.
>
> -- [https://tools.ietf.org/html/rfc7217#section-5][8]

Jak można się domyślać, póki co ta opcja nie jest zaimplementowana w kernelu linux'a (testowane na
5.8.1). Niemniej jednak, stabilne adresy prywatne są nas w stanie również uchronić przed różnymi
[technikami skanowania adresów IP][1] wykorzystującymi przewidywalne identyfikatory EUI64. Chodzi
generalnie o to, że identyfikatory OUI ([Organizationally Unique Identifier][2]) adresów MAC kart
sieciowych mają może i te 24 bity ale każdy producent sprzętu ma przypisany konkretny OUI i w ten
sposób te 24 bity są powszechnie znane. By utworzyć identyfikator EUI64, potrzebne są jeszcze dwie
stałe wartości `0xff` i `0xfe` oraz losowe 24 bity, które dopełniają ten 64 bitowy (24+16+24)
identyfikator interfejsu. Takie rozwiązanie ułatwia dość znacznie atakującemu wyszukiwanie
indywidualnych adresów IP przy założeniu, że ofiara ma sprzęt jednego z tych popularniejszych
producentów. W ten sposób spora część ochrony przeciw skanowaniu adresów wynikająca z ogromnej puli
adresów IP, jaką daje IPv6 w stosunku do IPv4, jest tracona bezpowrotnie.

Adresy prywatne dodatkowo chronią nas przed wyciekiem informacji za sprawą osadzania adresów
sprzętowych MAC w identyfikatorze interfejsu, co mógłby zostać wykorzystane do przeprowadzenia
ataków na określone urządzenia. Warto też dodać, że różne identyfikatory interfejsów dla każdego
skonfigurowanego adresu IPv6 sprawiają, że pozyskanie informacji na temat IID dla jednego
stabilnego adresu nie wpływa w negatywny sposób na bezpieczeństwo czy prywatność innych stabilnych
adresów skonfigurowanych dla innych prefiksów.

Kolejna sprawa to fakt, że stabilne adresy prywatne mogą być niezależne od sprzętu (NIC).
Wygenerowane adresy IPv6 nie ulegną więc zmianie, np. po wymianie karty sieciowej w komputerze. W
ten sposób nie będziemy musieli zmieniać konfiguracji swojego linux'a czy też domowej sieci. Warto
tu jednak zaznaczyć, że dwa różne systemy operacyjne działające na tym samym hoście w konfiguracji
multi/dual boot mogą mieć przydzielane różne adresy IPv6.

Stabilne adresy prywatne są podobne do rozszerzeń prywatności IPv6 (IPv6 Privacy Extensions) ale to
są dwa różne mechanizmy poprawiające prywatność przy dostępie do sieci IPv6. Żadne z nich nie
zastępuje drugiego i nie czyni go przestarzałym. Oba te mechanizmy mogą i powinny współistnieć
jednocześnie.

## Konfiguracja stabilnych adresów prywatnych na Debianie

Implementacja protokołu IPv6 na świecie póki co dość mocno kuleje i przez ostatnie 1,5 roku ilość
maszyn operujących na tym protokole zwiększyła się zaledwie o jakieś 5-8% i [obecnie wynosi
~30%][4]. W Polsce, IPv6 to najwyraźniej jakaś abstrakcja, bo zaledwie 12% maszyn jest podłączonych
do sieci po IPv6. W ten sposób dość ciężko jest uzyskać dostęp do tej sieci, no i ja go aktualnie
nie posiadam. Niemniej jednak, wsparcie dla IPv6 u naszego ISP nie jest konieczne by zrozumieć jak
działają stabilne adresy prywatne i jak je sobie skonfigurować, tak by móc zrobić z nich użytek w
przyszłości, gdy IPv6 stanie się dla nas dostępne.

Trochę szkoda, że póki co nie da rady zabezpieczyć prywatności użytkowników przy roamingu między
różnymi sieciami, głównie WiFi, ale na szczęście nie jest tak źle jak mogłoby się na pierwszy rzut
oka wydawać. Przy odrobinie determinacji możemy w mniejszym lub większym stopniu to zadanie
zrealizować. Niemniej jednak, trzeba pierw w systemie skonfigurować sobie stabilne adresy prywatne,
bo domyślnie ten ficzer jest wyłączony. Zatem do dzieła.

Tak wygląda przykład interfejsu sieciowego bez skonfigurowanych stabilnych adresów prywatnych:

    # ip addr show dev bond0
    2: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 00:21:cc:c3:05:b0 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic bond0
           valid_lft 86309sec preferred_lft 86309sec
        inet6 fd9d:40f3:50f5:0:221:ccff:fec3:5b0/64 scope global dynamic mngtmpaddr
           valid_lft forever preferred_lft forever
        inet6 fe80::221:ccff:fec3:5b0/64 scope link
           valid_lft forever preferred_lft forever

Jak widać, mój ISP nie wie co to IPv6 i udostępnia mi jedynie połączenie IPv4. Te widoczne wyżej
linijki z `inet6` to są lokalne adresy IPv6 uzyskiwane nawet jeśli host nie ma możliwości globalnej
komunikacji z wykorzystaniem protokołu IPv6 (wystarczy, że domowy router posiada wsparcie dla IPv6).
W ten sposób mamy `fe80::/64` , który wskazuje na adres linku lokalnego ([link-local address][6])
oraz `fd00::/8` wskazujący na unikalny adres lokalny ([ULA, Unique Local Address][7]). Może i nie
mamy tutaj globalnego adresu IPv6 ale te dwa lokalne adresy IPv6 nam w zupełności wystarczą by
zademonstrować działanie stabilnych adresów prywatnych.

Jak można było zauważyć wyżej w wyjściu `ip` , adres MAC karty sieciowej od tego interfejsu to
`00:21:cc:c3:05:b0` . Z niego został zbudowany identyfikator EUI64 `0221:ccff:fec3:05b0` . Bez
większego problemu można ten identyfikator zamienić na MAC i w ten sposób poznać adres sprzętowy
karty sieciowej komputera. Ten identyfikator EUI64 byłby obecny też w globalnym adresie IPv6,
gdybyśmy go tylko posiadali.

### Parametry addr_gen_mode oraz stable_secret w sysctl

Generowanie stabilnych adresów prywatnych na linux można skonfigurować w pliku `/etc/sysctl.conf` .
Ogólnie rzecz ujmując to interesują nas dwa parametry, tj. `addr_gen_mode` oraz `stable_secret` . W
różnych linux'ach te parametry przyjmują inne wartości. W Debianie, parametr `addr_gen_mode` ma
standardowo wartość `0` , natomiast próżno szukać w konfiguracji `sysctl` parametru `stable_secret`
(o tym za chwilę).

Parametr `addr_gen_mode` określa w jaki sposób są generowane adresy typu `link-local` oraz te
automatycznie konfigurowane (SLAAC). W przypadku ustawienia tutaj wartości `0` , adresy IPv6 będą
generowane w oparciu o identyfikator EUI64 i to jest to domyślne zachowanie, które zagraża nieco
naszej prywatności.

#### Ustawienie addr_gen_mode na 1

Gdy ustawimy wartość `addr_gen_mode` na `1` , to system zaprzestanie generowania adresów typu
`link-local` .  Brak adresu `link-local` sprawia, że protokoły takie jak NDP ([Neighbor Discovery
Protocol][10]) czy [DHCPv6][11] przestaną nam działać. W ten sposób nie damy rady już automatycznie
skonfigurować globalnego adresu IPv6 na interfejsie sieciowym (potrzebna będzie manualna
konfiguracja adresacji).

    net.ipv6.conf.bond0.addr_gen_mode = 1

Poniżej jest przykład tego samego interfejsu, który mieliśmy wcześniej ale z ustawionym parametrem
`addr_gen_mode` na `1` :

    # ip addr show dev bond0
    2: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 00:21:cc:c3:05:b0 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic bond0
           valid_lft 86321sec preferred_lft 86321sec

No jak widać, w tym przypadku został nam jedynie adres IPv4.

#### Ustawienie addr_gen_mode na 2/3 i stable_secret

W przypadku ustawienia wartości `addr_gen_mode` na `2` , stabilne adresy prywatne generowane będą
wykorzystując wartość określoną w parametrze `stable_secret` (zgodnie z RFC7217). W przypadku
nieustawienia `stable_secret` , w zasadzie zachowanie tego mechanizmu będzie takie samo jak przy
wartości `1` , tj. brak adresów lokalnych. Trzeba zatem się upewnić, że `stable_secret` został
wygenerowany. Standardowo w Debianie nie ma w `sysctl` takiego parametru jak `stable_secret` .
Pojawi się on dopiero wtedy, gdy system go wygeneruje, a wygeneruje go w momencie, gdy ustawi się w
`addr_gen_mode` wartość `3` lub też ręcznie ustawimy wartość sekretu dla danego interfejsu,
przykładowo:

    #net.ipv6.conf.all.stable_secret = e158:669a:0b27:307a:931d:aef0:bb70:b9aa
    net.ipv6.conf.default.stable_secret = e158:669a:0b27:307a:931d:aef0:bb70:b9aa
    net.ipv6.conf.bond0.stable_secret = e158:669a:0b27:307a:931d:aef0:bb70:b9aa

Teoretycznie parametr `stable_secret` ma również odpowiednik `all` i `default` ale do `all` nie da
rady nic zapisać. Jeśli zaś chcemy ustawić sekret dla każdego nowego interfejsu w systemie, to
korzystamy z `default` , a jeśli tylko dla określonego interfejsu, to podajemy jego nazwę.

Z chwilą wygenerowania sekretu, kernel nie będzie już go automatycznie generował ponownie,
przynajmniej podczas pracy systemu. By ten sekret został ponownie wygenerowany, trzeba będzie
zrestartować maszynę. Jeśli zaś chcemy ten sekret zmienić podczas pracy systemu, to przy pomocy
`sysctl` trzeba przepisać wartość parametru `stable_secret` ręcznie.

Gdy mamy wygenerowany sekret, to nie ma znaczenia już czy wartość parametru `addr_gen_mode` jest
ustawiona na `2` lub `3` , bo ten mechanizm generowania stabilnych adresów prywatnych będzie
zachowywał się od tego momentu dokładnie tak samo:

    net.ipv6.conf.all.addr_gen_mode = 2
    net.ipv6.conf.default.addr_gen_mode = 2

Te powyższe parametry zmieniają politykę generowania adresów IPv6 dla każdego interfejsu w systemie.
Jeśli chcemy jedynie zmienić politykę generowania adresów dla konkretnego interfejsu, to w miejscu
`all` albo `default` trzeba podać nazwę interfejsu sieciowego, np. `bond0` :

    net.ipv6.conf.bond0.addr_gen_mode = 2

Jeśli podejrzymy teraz jeszcze raz nasz interfejs sieciowy, to zobaczymy coś takiego:

    # ip addr show dev bond0
    2: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 00:21:cc:c3:05:b0 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic bond0
           valid_lft 86395sec preferred_lft 86395sec
        inet6 fd9d:40f3:50f5:0:6e6f:2f3b:5a2:13cc/64 scope global dynamic mngtmpaddr stable-privacy
           valid_lft forever preferred_lft forever
        inet6 fe80::8c2a:9317:9f2e:76/64 scope link stable-privacy
           valid_lft forever preferred_lft forever

Widzimy tutaj, że przy pozycjach z adresami IPv6 ( `inet6` ) pojawiły się tagi `stable-privacy` .
Oznaczają one, że mamy do czynienia ze stabilnymi adresami prywatnymi, co możemy też poznać po
innym identyfikatorze dla każdego z tych dwóch adresów IPv6:
`8c2a:9317:9f2e:0076` / `6e6f:2f3b:05a2:13cc` vs. standardowy identyfikator EUI64
`0221:ccff:fec3:05b0` wygenerowany na podstawie adresu MAC karty sieciowej. Gdyby linux posiadał
wsparcie dla roamingu między sieciami, to dodatkowo dla każdej takiej sieci te adresy IPv6 byłyby
inne.

### Przeładowanie konfiguracji sysctl i restart sieci

Po dodaniu opisanych wyżej parametrów do pliku `/etc/sysctl.conf` i ustawieniu im odpowiednich
wartości, musimy jeszcze przeładować konfigurację `sysctl` , tj. wydać w terminalu to poniższe
polecenie, by zmiany weszły w życie:

    # sysctl -p

Oczywiście by odczuć jakąkolwiek różnicę w zachowaniu systemu, trzeba położyć interfejs i go
podnieść ponownie. Krótko mówiąc, trzeba zrestartować połączenie sieciowe.

    # ifdown bond0
    # ifup bond0

## Zabezpieczenie pliku /etc/sysctl.conf

Trzeba także mieć na uwadze, że domyślnie każdy użytkownik w systemie jest w stanie odczytać plik
`/etc/sysctl.conf` , w którym ten sekret będzie przechowywany. Dlatego też praktycznie każdy proces
dowolnego użytkownika w systemie może poznać wartość parametru `stable_secret`, co czyni cały
mechanizm ździebko bez sensu. Dlatego też powinniśmy zadbać o odpowiednie prawa dostępu do pliku
`/etc/sysctl.conf` :

	# chmod 600 /etc/sysctl.conf
    # ls -al /etc/sysctl.conf
    -rw------- 1 root root 216915 2020-08-17 20:58:25 /etc/sysctl.conf

## Poprawa prywatności przy roamingu WiFi

Stabilne adresy prywatne są ciekawym mechanizmem poprawiającym naszą prywatność w sieciach IPv6.
Szkoda tylko, że póki co w linux nie ma zaimplementowanego wsparcia dla roamingu sieci WiFi.
Niemniej jednak, jeśli konfigurujemy osobisty laptop, którego zabieramy w podróż po świecie, to
możemy spokojnie w `addr_gen_mode` ustawić `3` i zrezygnować z ustawiania sekretu w
`stable_secret` . W taki sposób, za każdym razem jak tylko uruchomimy komputer ponownie, to
wszystkie adresy IPv6 na interfejsach sieciowych ulegną zmianie i każdy, kto nas śledził w sieci,
straci zwyczajnie trop. Trzeba jednak pamiętać, że wygenerowany sekret będzie stały do momentu
restartu maszyny. Dlatego też dobrze jest co jakiś czas ten sekret przepisać ręcznie i zrestartować
połączenie sieciowe. W przypadku każdego innego wykorzystania komputera, możemy ustawić
`addr_gen_mode` na `2` oraz określić stały sekret. Raz ustawiony sekret nie powinien być zmieniany.

Jeśli jednak chcielibyśmy trochę zautomatyzować cały ten proces poprawy prywatności w sieciach WiFi,
to możemy napisać prosty skrypt dla `ifupdown` , który będzie wywoływany chwilę przed podniesieniem
interfejsu. Zadaniem tego skryptu będzie wygenerowanie nowego sekretu i uzupełnienie parametru
`stable_secret` . By taką automatyzację wdrożyć, tworzymy plik
`/etc/network/if-pre-up.d/random-secret` :

	# touch /etc/network/if-pre-up.d/random-secret
	# chmod +x /etc/network/if-pre-up.d/random-secret

i dodajemy do niego poniższą zawartość:

	#!/bin/sh

	RUN="yes"

	if [ "$RUN" = "yes" ]; then

		if [ "$IFACE" = "wwan0" ] ||
		   [ "$IFACE" = "bond0" ] ||
		   [ "$IFACE" = "usb0" ]  ||
		   [ "$IFACE" = "eth0" ]  ||
		   [ "$IFACE" = "wlan0" ]; then

				secret=$(head -c16 </dev/urandom | \
						 xxd -p | \
						 fold -w4 | \
						 paste -sd':')
				sysctl -q -w net.ipv6.conf.$IFACE.stable_secret=$secret
		fi
	fi

W przypadku, gdy zajdzie potrzeba wyłączenia tego skryptu, to zawsze możemy przestawić `RUN="yes"`
na `RUN="no"` . Dodatkowo, możemy sobie określić, które z interfejsów w systemie będą mieć
generowane stabilne adresy prywatne przez uzupełnienie warunku `[ "$IFACE" = "wwan0" ]` .
Można naturalnie podać więcej interfejsów niż jeden.

I jeśli zaś chodzi o generowanie sekretu, to `head -c16 </dev/urandom` czyta 16 losowych bajtów
(128 bitów) z urządzenia `/dev/urandom` . Następnie `xxd -p` przerabia te bity na zapis HEX i w
taki sposób otrzymujemy ciąg typu `06294cadd1ead395f7cb634bd4da270a` .  Dalej mamy `fold -w4` ,
który tnie tę jedną długą linijkę na kilka mniejszych, z których każda będzie miała 4 znaki,
tj. `0629`, `4cad`, etc. Później `paste -sd':'` ma za zadanie te kolejne wiersze połączyć ze sobą
używając `:` . I w taki oto sposób otrzymujemy sekret w postaci
`0629:4cad:d1ea:d395:f7cb:634b:d4da:270a` , który jest następnie podawany w `sysctl` . Generowane w
ten sposób sekrety będą unikalne dla każdego z interfejsów sieciowych, nawet w przypadku, gdy
jednocześnie konfigurujemy wiele z nich, np. za sprawą `systemctl restart networking` .

Sprawdźmy jak wygląda teraz zachowanie naszego linux'a przy restarcie połączenia sieciowego:

	# ip addr show bond0
	2: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
		link/ether 00:21:cc:c3:05:b0 brd ff:ff:ff:ff:ff:ff
		inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic bond0
		   valid_lft 43191sec preferred_lft 43191sec
		inet6 fd9d:40f3:50f5:0:9715:f78e:a22a:d5dd/64 scope global dynamic mngtmpaddr stable-privacy
		   valid_lft forever preferred_lft forever
		inet6 fe80::35a7:1164:74c1:87ca/64 scope link stable-privacy
		   valid_lft forever preferred_lft forever

	# ifdown bond0
	# ifup bond0

	# ip addr show bond0
	2: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
		link/ether 00:21:cc:c3:05:b0 brd ff:ff:ff:ff:ff:ff
		inet 192.168.1.150/24 brd 192.168.1.255 scope global dynamic bond0
		   valid_lft 43198sec preferred_lft 43198sec
		inet6 fd9d:40f3:50f5:0:a9e8:9496:ba2e:e035/64 scope global tentative dynamic mngtmpaddr stable-privacy
		   valid_lft forever preferred_lft forever
		inet6 fe80::de92:3cfd:25b8:c944/64 scope link stable-privacy
		   valid_lft forever preferred_lft forever

Jak widać, adresy IPv6 uległy zmianie po restarcie połączenia sieciowego i widać na żywym
przykładzie, że możemy w dość prosty sposób pozbyć się identyfikatora sprzętowego adresu MAC z
części identyfikującej hosta w adresie IPv6 i tym samym utrudnić dość mocno śledzenie nas w sieci.


[1]: https://tools.ietf.org/html/rfc7707#section-4.1.1.1
[2]: https://en.wikipedia.org/wiki/Organizationally_unique_identifier
[3]: /post/jak-wlaczyc-ipv6-privacy-extensions-w-debianie-slaac/
[4]: https://www.google.com/intl/en/ipv6/statistics.html
[5]: https://tools.ietf.org/html/rfc7217
[6]: https://en.wikipedia.org/wiki/IPv6_address#Local_addresses
[7]: https://en.wikipedia.org/wiki/Unique_local_address
[8]: https://tools.ietf.org/html/rfc7217#section-5
[9]: https://en.wikipedia.org/wiki/MAC_address
[10]: https://en.wikipedia.org/wiki/Neighbor_Discovery_Protocol
[11]: https://en.wikipedia.org/wiki/DHCPv6
