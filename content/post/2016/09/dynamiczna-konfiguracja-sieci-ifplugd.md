---
author: Morfik
categories:
- Linux
date: "2016-09-01T12:24:40Z"
date_gmt: 2016-09-01 10:24:40 +0200
published: true
status: publish
tags:
- debian
- openbox
- sieć
title: Dynamiczna konfiguracja sieci w oparciu o ifplugd
---

Sporo użytkowników różnego rodzaju linux'ów, zwłaszcza dystrybucji Debian, niezbyt chwali sobie
automaty konfigurujące połączenie sieciowe typu [Network
Manager](https://wiki.gnome.org/Projects/NetworkManager). W sumie nigdy się jemu bliżej nie
przyglądałem ale na necie nie cieszy się on najlepszą opinią. Niemniej jednak, Network Manager
potrafi automatyzować pewne aspekty pracy w sieci. Weźmy przykład korzystania z dwóch różnych pod
względem parametrów sieci przewodowych. Jak się zachowa nasz OS w chwili przełączania się między
tymi sieciami w przypadku, gdy nie będziemy mieli zainstalowanego jakiegoś automatu dynamicznie
konfigurującego połączenie? W przypadku jednej sieci, połączenie będzie nam działać, w przypadku
drugiej zaś napotkamy problemy. W lekkich środowiskach opartych o menadżery okien, np. Openbox, nie
musimy instalować Network Manager'a, by ogarnąć tę kwestię konfiguracyjną. Możemy posiłkować się
demonem `ifplugd` i to tym narzędziu będzie ten wpis.

<!--more-->
## Po co nam ifplugd

[Demon ifplugd](http://0pointer.de/lennart/projects/ifplugd/) jest przeznaczony głównie dla bardzo
okrojonych linux'ów pozbawionych środowisk graficznych (GNOME, KDE). Generalnie, to znajduje on
zastosowanie wszędzie tam, gdzie ręcznie konfigurujemy połączenie sieciowe, zwykle za pomocą pliku
`/etc/network/interfaces` . W tym pliku znajdują się zwrotki konfigurujące poszczególne interfejsy,
np. `eth0` . Poniżej jest przykład takiej konfiguracji:

    #allow-hotplug eth0
    auto eth0
    iface eth0 inet dhcp

W przypadku korzystania z `allow-hotplug` , interfejs `eth0` zostanie podniesiony za każdym razem,
gdy nastąpi zdarzenie hotplug. Z kolei zaś jeśli używamy `auto` , to system podniesie ten interfejs
tylko podczas fazy boot. Linijka z `iface eth0` określa jak ten interfejs ma zostać skonfigurowany.
Tutaj wykorzystywany jest protokoł DHCP do pobrania adresacji IPv4, czyli standardowe podejście.

Problem w tym, że zdarzenia hotplug pojawiają się w momencie podłączania/odłączania urządzenia od
komputera. Ta karta sieciowa, której interfejs jest skonfigurowany powyżej, siedzi wewnątrz PC i nie
podłączamy jej za każdym razem, gdy chcemy nawiązać połączenie sieciowe. Karta zostanie wykrywa
tylko raz podczas fazy boot i skonfigurowana w tym konkretnym momencie. Jeśli zatem spróbujemy
podłączyć się do innej sieci, już po zalogowaniu się do systemu, to nasz linux będzie korzystał ze
starych ustawień sieci, czego efektem będzie brak połączenia. Poniżej jest zobrazowany moment
konfiguracji połączenia i przepięcia do drugiej sieci:

![]({{< baseurl >}}/img/2016/08/1.konfiguracja-sieci-ifplugd-eth0-log.png)

Ostatnie dwa komunikaty, tj. `eth0: link down` oraz `eth0: link up` informują nas, że połączenie
zostało zerwane w tym momencie. Może i zostało ono ponownie ustanowione, tylko co z tego, jak nie
została pobrana nowa konfiguracja sieci? By to połączenie znów nam działało, musimy ręcznie położyć
i podnieść interfejs `eth0` za pomocą poniższych poleceń:

    # ifdown eth0
    # ifup eth0

No i tu właśnie do gry wchodzi demon `ifplugd` , którzy jest w stanie zautomatyzować ten powyższy
proces.

## Jak działa ifplugd

Demon `ifplugd` działa sobie w tle i monitoruje poszczególne interfejsy pod kątem ich aktywności. W
przypadku podpięcia przewodu do portu przyporządkowanego interfejsowi `eth0` , demon `ifplugd`
wykryje to zdarzenie i zainicjuje akcję konfiguracji tego interfejsu sieciowego. Podobnie sprawa ma
się w przypadku odłączenia przewodu. Jeśli chodzi zaś o interfejsy WiFi, to `ifplugd` nie działa z
nimi i trzeba o tym pamiętać.

Zainstalujmy sobie zatem potrzebne narzędzia i skonfigurujmy demona `ifplugd` . W dystrybucji
Debian, ten demon znajduje się w pakiecie `ifplugd` i nie powinno być problemów z jego instalacją w
systemie. Po tym jak proces instalacyjny dobiegnie końca, w terminalu jako root wpisujemy poniższe
polecenie:

    # dpkg-reconfigure ifplugd

Naszym oczom pokaże się szereg okienek, które pomogą nam skonfigurować demona `ifplugd` .

Poniżej określamy wszystkie interfejsy przewodowe, które mają być ogarniane przez `ifplugd` (te bez
hotplug'u) :

![]({{< baseurl >}}/img/2016/08/2.konfiguracja-sieci-ifplugd-eth0.png)

W następnym okienku określamy interfejsy z hotplug'iem:

![]({{< baseurl >}}/img/2016/08/3.konfiguracja-sieci-ifplugd-eth0.png)

Dalej mamy instrukcję dotyczącą opcji demona (można przewijać strzałkami). Domyślnie są
wykorzystywane te poniższe flagi:

![]({{< baseurl >}}/img/2016/08/4.konfiguracja-sieci-ifplugd-eth0.png)

Z tych powyższych dostosowujemy jedynie `-u` i `-d` odpowiadające za czas rekonfiguracji interfejsu
sieciowego. W tym przypadku ustawiliśmy czas 2 sekundy. Po odłączeniu przewodu, interfejs zostanie
położony po około dwóch sekundach. Podobnie sprawa wygląda po podłączeniu przewodu do karty
sieciowej, z tym, że tutaj interfejs zostanie podniesiony po dwóch sekundach.

Ostatnie okienko zaś dotyczy hibernacji/wstrzymania, tj. co zrobić w przypadku, gdy przełączymy nasz
system w taki stan. Dobrze jest tutaj ustawić `stop` , bo to zresetuje demona po wybudzeniu, co
przydaje się przy korzystaniu z narzędzi takich jak `guessnet` lub `whereami` , które są w stanie
wykryć zmiany w środowisku sieciowym, np. podczas hibernacji laptop z roboty przynieśliśmy do domu i
tutaj go podłączamy do sieci.

![]({{< baseurl >}}/img/2016/08/5.konfiguracja-sieci-ifplugd-eth0.png)

Wszystkie te powyżej określone ustawienia są zapisywane w pliku `/etc/default/ifplugd` (dawniej w
`/etc/ifplugd/ifplugd.conf` ). Demona możemy naturalnie konfigurować bezpośrednio przez ten plik,
tylko pamiętajmy, by zresetować demona po dokonaniu zmian.

## Test połączenia

Demon powinien już nasłuchiwać i przydałoby się przetestować jak działa nasze połączenie sieciowe.
Podłączamy zatem przewód do karty sieciowej i patrzymy w log:

![]({{< baseurl >}}/img/2016/08/6.konfiguracja-sieci-ifplugd-eth0-log.png)

Jak widzimy, demon `ifplugd` podniósł interfejs sieciowy `eth0` i uzyskaliśmy adresację sieci. No to
teraz wyciągnijmy wtykę z portu i zobaczmy co się stanie:

![]({{< baseurl >}}/img/2016/08/7.konfiguracja-sieci-ifplugd-eth0-log.png)

Interfejs został położony zgodnie z planem. No to jeszcze sprawdźmy czy przy podłączeniu do drugiej
sieci, adresacja zostanie zmieniona również bez większego problemu:

![]({{< baseurl >}}/img/2016/08/8.konfiguracja-sieci-ifplugd-eth0-log.png)

I o takie zachowanie nam chodziło.
