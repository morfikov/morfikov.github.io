---
author: Morfik
categories:
- Linux
date: "2015-11-19T16:16:51Z"
date_gmt: 2015-11-19 15:16:51 +0100
published: true
status: publish
tags:
- statystyki
- sieć
GHissueID: 247
title: Statystyki transferu danych w sieci (vnstat)
---

W prehistorycznych czasach, internet był bardzo limitowany. Nie chodzi tutaj o prędkość, która
obecnie sięga 100+ mbit/s, a o transfer danych. Świat poszedł już trochę do przodu od tamtego czasu
i chyba żaden ISP, który obecnie dostarcza internet stacjonarny, nie narzuca swoim klientom ile
danych mogą pobrać i/lub wysłać w konkretnym miesiącu. Problem pojawia się w przypadku internetu
mobilnego, który w niedługim czasie prawdopodobnie zapanuje nad światem. Mowa oczywiście o
[LTE](https://pl.wikipedia.org/wiki/Long_Term_Evolution), czyli szerokopasmowym internecie
bezprzewodowym. Chodzi generalnie o to, że spora cześć providerów (jak nie wszyscy) limitują
transfer danych w tej usłudze. Jest to około 100GB na miesiąc. Może to wydawać się dużo ale trzeba
mieć na względzie, że dotyczy to zarówno download'u jak i upload'u. No i oczywiście, dziś wszystko
mamy w HD i rzadko kto korzysta z internetu sam. Nawet przeciętna strona www waży już kilka MiB.
Przydałoby się zatem wiedzieć ile danych transmitujemy przez sieć każdego dnia, tak by czasem nie
doświadczyć problemów związanych z przekroczeniem transferu. W tym wpisie postaramy się pozyskać te
informacje i wygenerujemy sobie przyzwoite statystyki transferu przy pomocy narzędzia `vnstat` .

<!--more-->
## Konfigurowanie vnstat

Jednym z bardziej popularnych narzędzi, które są w stanie dostarczyć statystyki transferu jest
[vnstat](http://humdi.net/vnstat/) . W debianie dostępny jest on w pakiecie o tej samej nazwie i
instalacja nie powinna sprawić żadnego problemu. Przejdźmy zatem od razu do samej konfiguracji,
która znajduje się w pliku `/etc/vnstat.conf` . Standardowa konfiguracja jest zadowalająca i tak na
dobrą sprawę, to możemy sobie dostosować jedynie pozycję `Interface` , dzięki której nie będzie
potrzeby ciągłego określania interfejsu przy zwracaniu statystyk.

Po dostosowaniu pliku konfiguracyjnego, musimy utworzyć odpowiednie pliki baz danych dla
poszczególnych interfejsów, które chcemy poddać monitorowaniu:

    # vnstat -u -i eth0
    # vnstat -u -i wlan0
    # vnstat -u -i bond0

## Statystyki transferu w vnstat

Po stworzeniu baz, `vnstat` zacznie zbierać statystyki. Niemniej jednak, musi upłynąć trochę czasu
zanim będzie możliwość zwrócenia jakichkolwiek wyników. Dlatego też należy się uzbroić w cierpliwość
i dopiero za jakiś czas sprawdzić czy zbieranie informacji na temat transferu działa w należytym
porządku. Po tym okresie czasu, statystyki możemy odczytać podając `vnstat` odpowiednie flagi: `-h`
dla poszczególnych godzin, `-d` dla statystyk dziennych oraz `-m` dla statystyk miesięcznych.
Poniżej przykłady.

Statystyki transferu z ostatnich 24 godzin:

![](/img/2015/11/1.vnstat-statystyki-transferu-dzien.png#huge)

Dzienne statystyki transferu (ostatnie 30 dni):

![](/img/2015/11/2.vnstat-dzienne-statystyki-transferu.png#huge)

Miesięczne statystyki transferu (ostatnie 12 miesięcy):

![](/img/2015/11/3.vnstat-miesieczne-statystyki-transferu.png#huge)

Z tych powyższych danych jasno wynika, że zużywam jakieś 200GiB transferu miesięcznie. Trochę to za
dużo by się przesiąść na LTE, choć pewnie dałoby radę zaprojektować jakich mechanizm, by te większe
pliki pobierał w godzinach nocnych, gdzie transfer nie jest naliczany. Tak czy inaczej, `vnstat`
jest nam w stanie zapewnić dokładne statystyki transferu danych, na podstawie których możemy
zdecydować czy dany ISP nam odpowiada.
