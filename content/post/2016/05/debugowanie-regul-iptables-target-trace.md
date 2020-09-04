---
author: Morfik
categories:
- Linux
date: "2016-05-11T13:20:25Z"
date_gmt: 2016-05-11 11:20:25 +0200
published: true
status: publish
tags:
- iptables
title: Debugowanie reguł iptables via target TRACE
---

Linux'owy firewall w postaci `iptables` może czasami mieć zdefiniowanych sporo reguł. W takim
gąszczu jest czasem ciężko się odnaleźć. Bywa zwykle tak, że dodając kolejną regułę chcemy
przyblokować szereg pakietów, a mimo to przechodzą one przez zaporę jak gdyby nigdy nic. W takim
przypadku zaczyna się mozolne przeszukiwanie reguł w `iptables` i próba ustalenia, którą z nich
zaakceptowała interesujący nas pakiet. Nie musimy jednak głowić się aż tak, by w miarę szybko dociec
jak te reguły przez filtr przechodzą i jak są przetwarzane. Możemy skorzystać z mechanizmu śledzenia
reguł, który jest wbudowany bezpośrednio w `iptables` . Mowa oczywiście o target `TRACE` . W tym
artykule zobaczymy jak przy pomocy `TRACE` ustalić, przez które reguły i łańcuchy przechodzi pakiet.

<!--more-->
## Kilka słów o target TRACE

Przede wszystkim, target `TRACE` może być wykorzystany tylko w tablicy `raw` . Chodzi o to, że
zarówno w kierunku do maszyny jak i od niej, pakiety przechodzą pierw przez tę tablicę i śledzenie
reguły musi się zacząć zaraz na samym początku. W przeciwnym razie ten proces nie miałby sensu.
Każdy pakiet, który spełnia kryteria określone w takiej regule, zostanie zalogowany. Informacje w
logu będą w formie `TRACE: tablename:chainname:type:rulenum` . Każdy wpis ma prefiks `TRACE:` , po
którym mamy nazwę tablicy ( `tablename` ) i łańcucha w niej ( `chainname` ). Z kolei `type` może
przyjąć wartość `rule` dla zwykłych reguł, `return` dla łańcuchów użytkownika oraz `policy` dla
wbudowanych łańcuchów. Natomiast jeśli chodzi o `rulenum` , to jest to numer reguły w danym
łańcuchu. Więcej informacji na temat target `TRACE` można znaleźć w [man
iptables-extensions](http://manpages.ubuntu.com/manpages/xenial/en/man8/iptables-extensions.8.html).

## Śledzenie reguł z target TRACE

Wszystkie te powyższe informacje będą nieco klarowniejsze, gdy rozpatrzymy sobie jakiś przykład.
Jeśli mamy problemy z ogarnięciem przepływu reguł przez pusty filtr, to dobrze jest się zapoznać z
tym poniższym diagramem:

![]({{< baseurl >}}/img/2016/05/1.iptables-przeplyw-pakietow-netfilter.png#huge)

Może nie wydaje się on zbyt skomplikowany ale nie uwzględnia on żadnych łańcuchów użytkownika, no i
nie ma też w nim reguł, które mogą blokować ruch. Tak czy inaczej, jeśli już [zbudowaliśmy swój
firewall]({{< baseurl >}}/post/firewall-na-linuxowe-maszyny-klienckie/) i przy jego rozbudowie coś
nam nie gra, to najwyższy czas skorzystać z target `TRACE` . Jakiś czas temu, gdy bawiłem się
[kontrolą ruchu sieciowego na
linux'ie]({{< baseurl >}}/post/ksztaltowanie-ruchu-sieciowego-traffic-control/), część z pakietów
podczas testów nie trafiała do tych kolejek, do których powinna. Problemem było przepisywanie się
znaczników, które były nakładane w kolejnych regułach mających target `MARK` . Minęło wtedy trochę
czasu zanim ustaliłem przyczynę ale na pewno poszłoby mi o wiele szybciej, gdybym wiedział o target
`TRACE` . Poniższy przykład będzie raczej banalny w porównaniu do kształtowania ruchu ale zawsze
liczy się zasada działania, którą można odnieść do każdej sytuacji. Dodamy sobie regułę, która
powinna blokować ruch przychodzący na port 21 (FTP):

    # iptables -t filter -I tcp 5 -p tcp -m tcp --dport 21 -j DROP

Po dodaniu tej powyższej reguły, z "jakichś bliżej nieznanych" nam przyczyn, użytkownicy nie mogą
się podłączyć do FTP'a. By zobaczyć jak działa śledzenie reguł w `iptables` dodajemy te dwie
poniższe reguły do filtra:

    # iptables -t raw -A OUTPUT -p tcp -m tcp --sport 21 -j TRACE
    # iptables -t raw -A PREROUTING -p tcp -m tcp --dport 21 -j TRACE

Mają one za zadanie śledzić pakiety, które trafiają na port 21. Zarówno te przychodzące na maszynę,
jak i te z niej wychodzące. Co się stanie jeśli teraz jakiś użytkownik spróbuje się podłączyć do
FTP'a? W logu powinniśmy zanotować sporo komunikatów. Wyglądają one mniej więcej tak jak te na tej
poniższej fotce:

![]({{< baseurl >}}/img/2016/05/2.target-trace-iptables-log-pakiet.png#huge)

Powyżej mamy 8 komunikatów. Z nich możemy wyczytać, że pakiet pochodzi z adresu 192.168.10.10 i miał
dotrzeć do hosta o adresie 192.168.1.150. Był także przeznaczony na port 21/tcp. Pakiet miał
ustawioną flagę `SYN` , zatem był to pakiet inicjujący połączenie. Pierwszy komunikat mówi nam, że
ten pakiet przeszedł przez tablicę `raw` i jej łańcuch `PREROUTING` . Z kolei `policy` oznacza, że
została zastosowana domyślna polityka wbudowanego łańcucha (cyfra nas nie interesuje). Wiemy, zatem,
że pakiet przeszedł przez tablicę `raw` i żadna reguła nic z nim nie zrobiła.

Następnie mamy trzy komunikaty dotyczące tablicy `mangle` . Widzimy, że dwa z nich mają `rule` .
Cyfry w tym przypadku oznaczają, że pakiet został dopasowany do reguły 1 i 4 w łańcuchu `PREROUTING`
w tej tablicy. Z kolei w trzecim komunikacie znów mamy `policy` co oznacza, że pakiet został
potraktowany zgodnie z polityką tego wbudowanego łańcucha. Cyfra 5 w tym przypadku nas nie
interesuje. Czyli krótko mówiąc, pakiet przeszedł przez dwie reguły w tablicy `mangle` , które z nim
coś zrobiły i został przesłany dalej.

Jako, że ten pakiet ma ustawioną flagę `SYN` , to jest to nowe połączenie, a te przechodzą tablicę
`nat` . W logu mamy jeden komunikat dotyczący tej tablicy, który wskazuje na `policy` . Zatem pakiet
został potraktowany zgodnie z polityką łańcucha `PREROUTING` w tablicy `nat` i został przesłany
dalej.

Kolejny komunikat w logu informuje nas, że pakiet trafia ponownie do tablicy `mangle` , tym razem do
łańcucha `INPUT` . Przechodzi przez niego, jako, że mamy tutaj ponownie `policy` .

Ostatnie dwa komunikaty dotyczą tablicy `filter` . Pakiet po trafieniu do tej tablicy został
dopasowany do 8 reguły w łańcuchu `INPUT`. W tym przypadku było to przekierowanie do łańcucha
`tcp` . Następnie widzimy, że pakiet w tym drugim łańcuchu został dopasowany do 5 reguły. I na tym
kończy się droga pakietu.

Mając informację na temat tego, gdzie zakończyła się droga pakietu, możemy odszukać tę regułę, bo
przecie znamy tablicę, łańcuch oraz numer reguły. Możemy to zrobić przy pomocy polecenia `iptables
-t filter -nvL tcp --line-numbers` :

![]({{< baseurl >}}/img/2016/05/3.target-trace-iptables-regula.png#huge)

Wyżej widzimy czarno na białym (właściwie to pomarańczowo na czarnym), że reguła numer 5 wskazuje na
tę, którą dodaliśmy na początku artykułu. By umożliwić połączenie z FTP, trzeba ją usunąć lub
zmienić.
