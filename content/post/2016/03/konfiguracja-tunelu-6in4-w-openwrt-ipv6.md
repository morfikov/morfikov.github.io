---
author: Morfik
categories:
- OpenWRT
date: "2016-03-01T15:28:22Z"
date_gmt: 2016-03-01 14:28:22 +0100
published: true
status: publish
tags:
- chaos-calmer
- sieć
- ipv6
- router
- 6in4
title: Konfiguracja tunelu 6in4 w OpenWRT (IPv6)
---

Ludzie z IETF prawie 20 lat temu opracowali protokół IPv6. Niemniej jednak, w dalszym ciągu ogromna
część providerów internetowych nie ma zamiaru zaimplementować u siebie jego obsługi. [Ilość
użytkowników, którzy mają natywne wsparcie dla protokołu
IPv6](http://www.google.com/intl/pl/ipv6/statistics.html) oscyluje w granicach 10% . Jeśli jednak
mamy do dyspozycji router z firmware OpenWRT na pokładzie, to możemy pokusić się o skonfigurowanie
tunelu 6in4. Jedynym warunkiem jest posiadanie zewnętrznego adresu IP. Tunel 6in4 jest bardzo
podobny do tego [6to4, który był opisywany na przykładzie
debiana]({{< baseurl >}}/post/implementacja-protokolu-ipv6-za-pomoca-tunelu-6to4/). Tutaj jednak
ten tunel zostanie ustawiony na routerze i w ten sposób cała wewnętrzna sieć będzie miała
przydzieloną określoną przestrzeń adresową z puli IPv6.

<!--more-->
## OpenWRT Chaos Calmer i potrzebne oprogramowanie

Wszystkie poniższe kroki będą przeprowadzane na routerze [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/cat-9_Archer-C7.html), gdzie wgrany jest [firmware
OpenWRT Chaos Calmer (15.05)](http://eko.one.pl/). Z tego co zdążyłem ustalić, we wcześniejszych
wersjach (BB, AA) cześć rzeczy nie jest wspieranych lub też szereg rozwiązań jest już
przestarzałych. Dlatego też jest niemal pewne, że niżej opisane rzeczy nie będą się stosować do
tych starszych wersji OpenWRT i by wdrożyć u siebie tunel 6in4, to będziemy musieli zaktualizować
firmware routera.

Logujemy się zatem na router i instalujemy potrzebne oprogramowanie. Modułów kernela, serwera DHCPv6
jak i samego filtra pakietów nie musimy instalować w OpenWRT Chaos Calmer, bo te pakiety są już
domyślnie wgrane. Interesuje nas głownie pakiet `6in4` oraz `iputils-ping6` , który umożliwi nam
przeprowadzenie testu połączenia:

    # opkg update
    # opkg install kmod-ipv6 kmod-ip6tables ip6tables odhcpd 6in4 iputils-ping6

## Jak zarejestrować tunel 6in4

Mając zainstalowane potrzebne oprogramowanie na routerze, odpalamy przeglądarkę i przechodzimy na
stronę [tunnelbroker.net](https://tunnelbroker.net/). To tutaj zarejestrujemy za free tunel, którego
konfigurację będziemy mogli wdrożyć na routerze. By móc korzystać z usług tego serwisu, musimy się
pierw zarejestrować. Po zarejestrowaniu, tworzymy tunel wybierając z menu po lewej stronie `Create
Regular Tunnel` .

![]({{< baseurl >}}/img/2016/03/1.rejestracja-tunelu-6in4-hurricane-electric.png#small)

Przy tworzeniu tunelu 6in4, trzeba określić jeden z jego z końców. Po to nam jest właśnie potrzebny
zewnętrzny adres IP w protokole IPv4. Nasz aktualny adres powinien zostać automatycznie wykryty i
wystarczy go przekopiować w pole `IPv4 Endpoint` . Ważne jest by podczas tego kroku router był w
stanie odpowiadać na zapytania `ping` od strony WAN. W przeciwnym razie dostaniemy taki oto
komunikat i nie będziemy mogli kontynuować:

![]({{< baseurl >}}/img/2016/03/2.problem-sprawdzanie-tunel-6in4.png#big)

Jeśli nasz router odpowiada na żądanie `ping` z tego serwisu, to powinniśmy zobaczyć wiadomość o
tym, że dany adres może być potencjalną końcówką dla tunelu 6in4:

![]({{< baseurl >}}/img/2016/03/3.pomyslna-weryfikacja-tunel-6in4.png#big)

Standardowo możemy zarejestrować 5 tunelów ale tylko jeden tunel może być przypisany do konkretnego
zewnętrznego adresu IP w protokole IPv4. Wybieramy także serwer, do którego mamy najbliżej. W tym
przypadku jest to Europa > Warszawa. Po chwili zostanie nam zwrócona strona z informacjami o nowo
utworzonym tunelu.

## Konfiguracja tunelu 6in4 w OpenWRT

Mając tunel, musimy uzyskać jego konfigurację, którą później wpiszemy w OpenWRT. Przechodzimy zatem
na zakładkę `Example Configurations` i wybieramy z listy OpenWRT. Nie ma, co prawda, Chaos Calmer
ale ta konfiguracja podana dla Barrier Breaker również działa prawidłowo.

![]({{< baseurl >}}/img/2016/03/4.konfiguracja-tunelu-6in4-openwrt.png#big)

W tej konfiguracji jest wykorzystywane narzędzie `uci` ale nic nie stoi na przeszkodzie, by
wszystkie wpisy ręcznie sobie dodać przez edycję pliku `/etc/config/network` . Logujemy się zatem na
router i we wspomnianym pliku odszukujemy sekcję `wan6` . Przerabiamy ją do poniższej postaci:

    config interface 'wan6'
          option ifname    'eth0'
          option proto     '6in4'
          option peeraddr  '216.66.80.162'
          option ip6addr   '2001:470:11:22::2/64'
          option ip6prefix '2001:470:11:22::/64'
          option tunnelid  '12345'
          option username  'login'
          option password  'pass'

Wyżej w pliku `/etc/config/network` mamy opcję `ula_prefix` . Może ona powodować problemy z
przydzielaniem adresów IPv6 w sieci lokalnej. W taki sposób zamiast adresów oferowanych przez
Hurricane Electric, OpenWRT będzie przydzielał adresy [ULA (Unique Local
Address)](https://en.wikipedia.org/wiki/Unique_local_address) i hosty lokalne nie będą miały dostępu
do tunelu 6in4. Dobrze jest zatem wykomentować ten poniższy blok:

    #config globals 'globals'
    #       option ula_prefix 'fd42:156b:900b::/48'

Zapisujemy konfigurację i resetujemy router. Po podniesieniu się routera, w statystykach interfejsów
powinien się pojawić interfejs `6in4-wan6` posiadający adres IPv6: `2001:...::2/64` . Dodatkowo, na
interfejsie `br-lan` również powinien się pojawić adres IPv6: `2001:...::1/64` . Można to sprawdzić
przy pomocy narzędzi takich jak `ifconfig` lub `ip` .

## Test połączenia IPv6 przez tunel 6in4 na routerze

Pozostało nam jeszcze przetestowanie konfiguracji samego tunelu. Będąc na routerze, ping'ujemy adres
`ipv6.google.com` :

    # ping -6 ipv6.google.com
    PING ipv6.google.com (2a00:1450:4008:800::200e): 56 data bytes
    64 bytes from 2a00:1450:4008:800::200e: seq=0 ttl=54 time=36.147 ms
    64 bytes from 2a00:1450:4008:800::200e: seq=1 ttl=54 time=32.065 ms
    64 bytes from 2a00:1450:4008:800::200e: seq=2 ttl=54 time=33.969 ms
    ^C
    --- ipv6.google.com ping statistics ---
    3 packets transmitted, 3 packets received, 0% packet loss
    round-trip min/avg/max = 32.065/34.060/36.147 ms

Pakiety docierają do celu i wracają do nas bez większego problemu. Zatem tunel działa i realizuje
połączenie. Niemniej jednak, póki co tylko router ma dostęp do sieci IPv6. Musimy jeszcze
skonfigurować hosty w sieci lokalnej, tak by im też został przypisany odpowiedni adres IPv6.

## Konfiguracja IPv6 na hoście lokalnym

Konfiguracja adresacji w protokole IPv6 odbywa się automatycznie. W zależności od posiadanego
systemu operacyjnego, może być wymagane dodatkowe dostosowanie pewnych parametrów. Niemniej jednak,
zarówno windows jak i linux, bez problemu powinny podłapać konfigurację przesyłaną przez router w
[wiadomościach Router
Advertisements](http://www.tcpipguide.com/free/t_ICMPv6RouterAdvertisementandRouterSolicitationMess.htm)
i automatycznie się skonfigurować bez naszej ingerencji. Restartujemy zatem połączenie sieciowe na
swoim laptopie i sprawdzamy, czy interfejsowi sieciowemu został przydzielony adres IPv6 z puli,
którą otrzymał router za sprawą tunelu 6in4.

Na dobrą sprawę powinniśmy mieć jeden adres zaczynający się od `fe80:` ([link-local
address](https://en.wikipedia.org/wiki/Link-local_address)) i, w zależności od konfiguracji, jeden
lub dwa adresy zaczynające się od `2001:` . Czemu jeden albo dwa? Standardowo, konfiguracja
adresacji w protokole IPv6 jest dokonywana via mechanizm [stateless address autoconfiguration
(SLAAC).](https://en.wikipedia.org/wiki/IPv6#Stateless_address_autoconfiguration_.28SLAAC.29)
Dodatkowo, jeśli posiadamy w sieci serwer DHCPv6, a posiadamy, to za jego sprawą może zostać
przypisany drugi adres IPv6, z tym, że jego końcówka będzie taka sama jak tego adresu z puli IPv4.

Tak czy inaczej, musimy sprawdzić czy połączenie na lokalnych hostach działa. Odpalamy przeglądarkę
i przechodzimy na adres `http://ipv6test.google.com/` . Jeśli wszystko działa jak należy, powinniśmy
zobaczyć ten poniższy komunikat:

![]({{< baseurl >}}/img/2016/03/5.test-tunel-6in4-google-ivp6.png#medium)

Możemy także sprawdzić jakość tego połączenia przechodząc pod adres `http://test-ipv6.com/` . W tym
przypadku wygląda to tak:

![]({{< baseurl >}}/img/2016/03/6.test-tunel-6in4-jakosc-ipv6.png#big)
