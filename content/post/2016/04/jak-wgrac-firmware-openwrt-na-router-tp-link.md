---
author: Morfik
categories:
- OpenWRT
date: "2016-04-22T12:34:00Z"
date_gmt: 2016-04-22 10:34:00 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- tp-link
GHissueID: 407
title: Jak wgrać firmware OpenWRT na router TP-LINK
---

[OpenWRT](https://openwrt.org/) to alternatywny firmware dla routerów oparty o linux'a. Może on
okazać się dość złożonym systemem i to pomimo faktu, że zajmuje raczej niewiele miejsca, bo rozmiary
obrazów nie przekraczają 8-16 MiB, oczywiście wliczając w to wolne miejsce. Niemniej jednak, by
komfortowo operować na dostarczonym oprogramowaniu, trzeba dysponować określoną wiedzą. OpenWRT w
dużej mierze przypomina linux'a i szereg poleceń jest w tych dwóch systemach taki sam. Jedynie co,
to z powodu dość ograniczonego miejsca, większość programów została okrojona dość solidnie,
zostawiając przy tym tylko niezbędną funkcjonalność. Mamy, co prawda, szereg nakładek z interfejsem
webowym (min. [Gargoyle](https://www.gargoyle-router.com/) i
[LuCi](https://github.com/openwrt/luci/wiki)), które upraszczają operowanie na tym firmware ale my w
tym artykule położymy nacisk na przejście na czyste OpenWRT, z którym będziemy się komunikować za
pomocą poleceń tekstowych wydawanych w terminalu.

<!--more-->
## Gargoyle, LuCi czy czyste OpenWRT?

Na sieci można się spotkać z szeregiem projektów dotyczących OpenWRT. Pomijając samo OpenWRT,
natknąć się możemy na Gargoyle i LuCi. Różnica tych dwóch w stosunku do czystego OpenWRT jest
taka, że te obrazy zawierają w sobie interfejs graficzny, za pomocą którego można skonfigurować
router. W przypadku, gdy nie jesteśmy zbytnio obeznani w tematyce linux'a i całego jego
opensource'owego oprogramowania i do tego jest to nasze pierwsze spotkanie z OpenWRT, to radziłbym
pobrać obrazy mające na pokładzie [Gargoyle](http://eko.one.pl/?p=openwrt-gargoylepl) lub
[LuCi](http://eko.one.pl/?p=openwrt-luci)). Niemniej jednak, w dalszym ciągu będziemy mieć dostęp do
trybu tekstowego przez protokół SSH i po części będziemy mogli uczyć się konfiguracji tego firmware
z poziomu czystej konsoli. Trzeba jednak pamiętać, że takie nakładki graficzne mają własne
ustawienia, które zwykle przepiszą naszą konfigurację zaaplikowaną via terminal. Dlatego też nie
powinniśmy mieszać tych dwóch kwestii i albo zarządzać routerem przez interfejs webowy, albo przez
terminal. Tak czy inaczej, jeśli będziemy chcieli poznać jak działa OpenWRT, to prędzej czy później
ten dostarczany interfejs webowy nas zacznie coraz bardziej ograniczać. Trzeba też wziąć pod uwagę
fakt, że nie wszystkie zadania da radę zrealizować za pomocą tego GUI.

Ci użytkownicy, którzy mieli już styczność z linux'em mogą pokusić się o instalację [czystego
OpenWRT](http://eko.one.pl/?p=openwrt-chaos-calmer). Dobrze jest także opanować obsługę edytora
[vi](https://pl.wikipedia.org/wiki/Vi_%28program%29) (lub [vim](https://pl.wikipedia.org/wiki/Vim)),
bo to za jego pomocą będziemy edytować wszystkie pliki tekstowe po zalogowaniu się na router. Ta
aplikacja nie jest znowu aż tak skomplikowana jak się może wydawać z początku. Wystarczy zacząć
używać `vi` , a parę dni później człowiek sam się nauczy z niego korzystać. Dobrze jest także
poznać filtr pakietów `iptables` oraz zaznajomić się z jego modułami dostępnymi w OpenWRT. Nie ze
wszystkich modułów będziemy korzystać, przynajmniej nie od razu, ale przydałoby się prześledzić je i
wiedzieć miej więcej do czego można owe moduły zastosować. Więcej informacji na temat iptables można
znaleźć [tutaj](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html),
[tutaj](https://pl.wikibooks.org/wiki/Sieci_w_Linuksie/Netfilter/iptables) oraz na [stronie
projektu](http://www.netfilter.org/). Jeśli zdecydujemy się na firmware OpenWRT bez nakładek
graficznych, to będziemy musieli przywyknąć do korzystania z terminala, gdzie polecenia będziemy
wprowadzać za pomocą klawiatury. Zawsze też będziemy mieli możliwość doinstalowania interfejsu LuCi
wgrywając pakiety mające w nazwie `luci-*` .

## Zanim zmienimy firmware

Przed przystąpieniem do jakichkolwiek prac, musimy się upewnić, że router, który posiadamy, jest
obsługiwany przez firmware OpenWRT. Lista dostępnych urządzeń znajduje się [pod tym
linkiem](https://wiki.openwrt.org/toh/start). W tym przypadku mamy do czynienia z routerem [TP-LINK
WDR3600 v1.5](http://www.tp-link.com.pl/products/details/TL-WDR3600.html). Trzeba dobrze czytać
adnotacje, bo nie zawsze dany router jest obsługiwany w pełni przez ten firmware. W części
przypadków może się zdarzyć tak, że wersja routera może być nieodpowiednia lub wspierana dopiero od
określonego wydania OpenWRT. Tutaj sprawa wygląda następująco:

![openwrt-router-wsparcie-firmware](/img/2016/04/1.openwrt-router-wsparcie-firmware.png#huge)

Widzimy, że wersja 1.5 routera WDR3600 jest wspierana od Chaos Calmer 15.05, choć niżej możemy
wyczytać, że sporadycznie mogą się pojawić jakieś drobne problemy. Generalnie nic, czym powinniśmy
się przejmować przy zmianie firmware. To urządzenie będzie działać pod OpenWRT.

## Zmiana firmware na OpenWRT

Na dobrą sprawę nie ma znaczenia, z których obrazów chcemy korzystać. Może to być Gargoyle, LuCi lub
czyste OpenWRT (linki do wszystkich tych obrazów są wyżej). Dla przykładu, wgramy czysty OpenWRT bez
interfejsu webowego. [Pod tym linkiem](http://dl.eko.one.pl/chaos_calmer/ar71xx/) mamy dostępne dwa
obrazy dla routera WDR3600. Jeden z nich ma w nazwie `factory` , drugi zaś ma `sysupgrade` . Przy
przechodzeniu z firmware producenta na alternatywny korzystamy z obrazu `factory` . Pobieramy zatem
plik `openwrt-15.05-ar71xx-generic-tl-wdr3600-v1-squashfs-factory.bin` na dysk.

Zmiana firmware odbywa się na podobnej zasadzie co aktualizacja oprogramowania producenta routera.
Musimy zalogować się do panelu administracyjnego TP-LINK'a. Taki upgrade nie powinien być
przeprowadzany za pośrednictwem sieci WiFi. Dlatego też jeśli jesteśmy połączeni bezprzewodowo,
podepnijmy się do routera za pomocą przewodu. Następnie odpalamy przeglądarkę internetową i w polu
adresu wpisujemy `http://192.168.0.1` . logujemy się do panelu. Standardowy login to `admin` , hasło
również `admin` :

![tp-link-panel-admin-router](/img/2016/04/2.tp-link-panel-admin-router.png#big)

Z menu po lewej stronie wybieramy kolejno System Tools -> Firmware Upgrade i wskazujemy pobrany
plik:

![aktualizacja-firmware-openwrt-panel-admin-tp-link](/img/2016/04/3.aktualizacja-firmware-openwrt-panel-admin-tp-link.png#huge)

Klikamy przycisk Upgrade i po chwili rozpocznie się flash'owanie routera nowym firmware:

![aktualizacja-firmware-openwrt-panel-admin-tp-link](/img/2016/04/4.aktualizacja-firmware-openwrt-panel-admin-tp-link.png#huge)

Po zakończeniu, router powinien się automatycznie zresetować:

![aktualizacja-firmware-openwrt-panel-admin-tp-link](/img/2016/04/5.aktualizacja-firmware-openwrt-panel-admin-tp-link.png#huge)

Wyżej mamy komunikat o odświeżeniu strony w przeglądarce. Jeśli to zrobimy, to nic nam się nie
załaduje. Nawet w przypadku jeśli wgraliśmy firmware z panelem webowym. Domyślny adres dla OpenWRT
to `192.168.1.1` i to pod niego trzeba wysłać żądanie. Dlatego też jeśli wgraliśmy Gargoyle lub
LuCi, to w pasku adresu musimy przepisać URL. Jeśli korzystaliśmy z czystego OpenWRT, to musimy
odpalić terminal i zalogować się na router w trybie tekstowym za pomocą protokołu telnet. Pamiętajmy
też, że skoro zmianie uległa adresacja sieci (z 192.168.0/24 na 192.168.1.0/24), to musimy
zresetować połączenie sieciowe na komputerze, by uzyskać odpowiedni adres IP i być w stanie
połączyć się z routerem. Poniżej przykład logowania się do routera via `telnet` :

![openwrt-logowanie-terminal-telnet](/img/2016/04/6.openwrt-logowanie-terminal-telnet.png#big)

Do pełni szczęścia musimy jeszcze ustawić hasło dla konta administratora root przez wpisanie w
terminalu `passwd` . Zaowocuje to wyłączeniem protokołu telnet i włączeniem protokołu SSH. To w
zasadzie wszystko jeśli chodzi o przejście z oryginalnego firmware producenta routera na firmware
OpenWRT. Czeka nas jeszcze konfiguracja routera, niemniej jednak, te domyślne ustawienia umożliwią
nam połączenie z internetem ale tylko przewodowo.
