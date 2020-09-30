---
author: Morfik
categories:
- Linux
date: "2015-11-05T17:17:16Z"
date_gmt: 2015-11-05 16:17:16 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
title: Moduł xt_recent i limitowanie połączeń w iptables
---

Kluczową rolę w filtrze iptables pełnią stany połączeń. Zwykle mamy do czynienia z trzema z nich:
NEW, RELATED i ESTABLISHED. Wszystko co nie pasuje do tych stanów, jest traktowane jako INVALID i
tak dla przykładu trafiają tam pakiety mające niemożliwe kombinacje flag, przynajmniej jeśli chodzi
o punkt widzenia poprawnej komunikacji sieciowej (ustawione flagi `SYN` i `FIN` jednocześnie).
Jednak istnieje szereg pakietów, które mogą potencjalnie zagrażać bezpieczeństwu maszyny i nie są
one uwzględnione w stanie INVALID. Takie pakiety są używane do skanowania portów w celu wykrycia
usług znajdujących się na serwerze. Krótko mówiąc, stan INVALID nie złapie skanów `UDP`, `ACK` oraz
`SYN` . Czy jesteśmy faktycznie bezbronni i nic nie możemy zrobić? Na szczęście iptables ma do
dyspozycji [moduł xt\_recent](http://ipset.netfilter.org/iptables-extensions.man.html), który jest w
stanie zablokować wszystkie te powyżej wymienione formy ataków.

<!--more-->
## Bazowy skrypt iptables

By ułatwić sobie nieco pracę, musimy napisać prosty [skrypt
firewall'a](/post/firewall-na-linuxowe-maszyny-klienckie/). Ten, który został
przedstawiony w przytoczonym linku nadaje się idealnie, bo zawiera rozdział pakietów na protokoły
TCP i UDP oraz domyślnie blokuje połączenia przychodzące.

Generalnie rzecz biorąc, mając zaimplementowany ten powyższy skrypt, nikt nie będzie w stanie
nawiązać żadnego połączenia z serwerem. Problem ze skanami portów jest taki, że ci co je skanują
wcale nie chcą się podłączać. Jedyne o co im chodzi w tego typu atakach, to o odpowiedź serwera, a
ta może być różna w zależności od tego czy na danym porcie nasłuchuje jakaś usługa. Zatem nawet nie
uzyskując połączenia jesteśmy w stanie powiedzieć, czy port, który skanujemy, jest otwarty,
zamknięty, czy odfiltrowany na zaporze. Cały proces skanowania jest wręcz automatyczny i mamy na
rynku całą masę skanerów sieciowych, które potrafią przeanalizować zwracane przez serwer komunikaty.
W tym przypadku zajmiemy się tylko trzema formami skanów, które `nmap` może przeprowadzić gdy mu się
poda te parametry: `-sU` (UDP), `-sA` (ACK), `-sS` (SYN).

Weźmy przykładowo skan `SYN` i zobaczmy jak jakaś maszyna w sieci na niego zareaguje. Przeskanujemy
dany port cztery razy. Pierwszy skan będzie miał miejsce gdy na tym porcie nie nasłuchuje żadna
usługa i nie mamy żadnych reguł wpisanych do iptables. Drugi skan będzie dokonany gdy usługa
zacznie nasłuchiwać na tym porcie, również bez żadnych reguł na zaporze. Trzeci skan przeprowadzimy
po zaaplikowaniu reguł z naszego skryptu firewall'a. Natomiast ostatni skan będzie dokonany w
oparciu o ustawienie polityki `-j DROP` dla tego portu. Poniżej są wyniki wszystkich skanów:

    # nmap -sS 192.168.10.10 -p 22

    PORT   STATE  SERVICE
    22/tcp closed ssh
    22/tcp open  ssh
    22/tcp closed ssh
    22/tcp filtered ssh

Stan portu `open` wskazuje, że na danym porcie nasłuchuje usługa oraz, że jesteśmy w stanie nawiązać
połączenie. Z kolei `filtered` wyraźnie informuje nas, że z serwerem nie ma żadnej komunikacji na
tym porcie. Wiemy też przy okazji, że jest tam jakiś firewall. Z kolei stan `closed` oznacza, że
usługa nie nasłuchuje w danym momencie. Jak widzimy wyżej, stan `closed` pojawił się dwa razy. Raz
gdy demon SSH nie nasłuchiwał, drugi raz zaś ,gdy zaaplikowaliśmy reguły ze skryptu iptables. Mając
na uwadze powyższe informacje, możemy przeskanować wszystkie 65536 portów i ustalić, które z nich są
otwarte. Nie trwałoby to więcej niż godzinę.

## Moduł xt\_recent

Nasuwa się pytanie, w jaki sposób zatem się bronić przed tego typu skanami portów? Musimy
zaprojektować mechanizm, który by opóźnił cały proces. Nie możemy co prawda w żaden sposób
powstrzymać atakującego przed dokonaniem skanu otwartego portu ale w przypadku gdyby dokonał skanu
portu, który nie jest otwarty, można by czasowo dopisać jego adres IP do listy zbanowanych. Do tego
celu można wykorzystać moduł `xt_recent` jaki oferuje nam iptables.

Przede wszystkim, musimy przerobić nieco skrypt firewalla. Chodzi generalnie o regułki co mają
ustawiony target `-j REJECT` , a dokładnie, o te poniższe:

    iptables -t filter -A INPUT -p udp -j REJECT --reject-with icmp-port-unreachable
    iptables -t filter -A INPUT -p tcp -j REJECT --reject-with tcp-reset

Musimy je zastąpić tymi
    dwiema:

    iptables -t filter -A INPUT -p udp -m recent --set --name udp-portscan -j REJECT --reject-with icmp-port-unreachable
    iptables -t filter -A INPUT -p tcp -m recent --set --name tcp-portscan -j REJECT --reject-with tcp-reset

Za sprawą tych powyższych reguł, moduł `xt_recent` tworzy dwie listy (po jednej dla protokołu UDP
oraz TCP) z adresami IP, z których nadeszło zapytanie na zamknięte porty. To tylko część całego
mechanizmu. Albowiem potrzebujemy jeszcze dwóch poniższych
    regułek:

    iptables -t filter -A udp -p udp -m recent --update --seconds 600 --name udp-portscan -j REJECT --reject-with icmp-port-unreachable
    iptables -t filter -A tcp -p tcp -m recent --update --seconds 600 --name tcp-portscan -j REJECT --reject-with tcp-reset

Sprawią one, że przez następne 600 sekund (10 minut), serwer w zależności od protokołu będzie
zwracał komunikat `tcp-reset` lub `icmp-port-unreachable` hostom, które zostały dodane na listę za
sprawą dwóch poprzednich reguł. Nie będzie miało przy tym znaczenia czy port, do którego trafił
pakiet jest otwarty czy zamknięty. Dodatkowo, każdy nowy pakiet od takiego delikwenta będzie
skutkował odświeżeniem czasu, przez jaki serwer będzie odpowiadał powyższymi komunikatami, co czyni
skanowanie portów wielce niepraktycznym. Nie musimy ustawiać też ogromnej wartości jeśli chodzi o
czas. Możemy zwyczajnie pozostać przy 10 minutach, a skanowanie wszystkich portów zajęłoby prawie 2
lata. Oczywiście zakładając, że kolejny port byłby skanowany dokładnie co 10 minut i atakujący nigdy
by nie dostał bana.

## Ataki brute force

Administratorzy sieciowi rzadko kiedy dozbrajają otwarty port pod kątem ilości możliwych zapytań w
pewnym skończonym interwale czasu. Jeśli korzystamy z [port
knocking'u](/post/port-knocking-i-single-packet-authorization/) , to nie mamy
zbytnio się czym martwić. Większość osób jednak tego nie robi i co by się stało w przypadku, gdy
ktoś namierzy port, na którym nasłuchuje demon SSH? Jeśli nie jest on chroniony w jakiś dodatkowy
sposób, np. za pomocą [plików hosts.allow i
hosts.deny](/post/pliki-hosts-allow-i-hosts-deny/), to nasz serwer stoi otworem
przed atakującym i tylko kwestią czasu jest, kiedy nasz serwer padnie ofiarą ataku. Oczywiście,
osoba próbująca dostać się do serwera niekoniecznie musi być kimś kogo w ogóle nie znamy. Może się
zdarzyć tak, że nasz były dobry znajomy chce się odgryźć za coś, cośmy mu uczynili kiedyś tam w
przeszłości. Przydałoby się zatem jakoś zablokować ataki brute force na hasło do konta shell'owego.
Moduł `xt_recent` jest w stanie nam pomóc i z tym zdaniem.

Musimy utworzyć nowy łańcuch i przekierować do niego pakiety, które mają trafić do chronionej
usługi. Załóżmy, że chcemy zabezpieczyć demona SSH, który nasłuchuje na porcie 22. Do skryptu
iptables dodajemy te poniższe linijki:

    iptables -t filter -N ssh

    iptables -t filter -A tcp -p tcp --dport 22 -j ssh -m comment --comment "ssh"

    iptables -t filter -A ssh -m recent --name ssh-bruteforce --rttl --rcheck --hitcount 3 --seconds 20 -j DROP
    iptables -t filter -A ssh -m recent --name ssh-bruteforce --rttl --rcheck --hitcount 10 --seconds 1800 -j DROP
    iptables -t filter -A ssh -m recent --name ssh-bruteforce --set -j ACCEPT

Ten powyższy mechanizm ma na celu banowanie hostów, które w czasie 20 sekund wysłały więcej niż 3
próby logowania na shell'a. W ten sposób każda następna próba zostanie zablokowana. Ten limit
zostanie także nieco rozciągnięty po upływie tych 20 sekund i będzie wynosił 10 prób na 30 minut.
Trzeba wziąć jednak pod uwagę fakt, że powyższe ustawienia mogą umożliwić atak DOS w przypadku gdyby
ktoś zespoofował adres IP.

## Usuwanie wpisów z list modułu xt\_recent

Wszystkie listy, które są tworzone za sprawą modułu `xt_recent` , są przechowywane w katalogu
`/proc/net/xt_recent/` . Jeśli na jedną z nich przez przypadek trafi jakiś adres IP, to bez problemu
możemy go z takiej listy usunąć. Poniżej jest przykład jak to zrobić:

    # echo -192.168.10.100 > /proc/net/xt_recent/ssh-bruteforce
