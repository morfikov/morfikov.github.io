---
author: Morfik
categories:
- OpenWRT
date: "2016-04-23T18:50:55Z"
date_gmt: 2016-04-23 16:50:55 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
title: Tryb ratunkowy (failsafe) w OpenWRT
---

Obrazy OpenWRT mają wbudowany [tryb failsafe](https://wiki.openwrt.org/doc/howto/generic.failsafe)
obchodzący całą konfigurację, która została zapisana na [partycji
JFFS2](https://pl.wikipedia.org/wiki/JFFS2). To jest ta części pamięci flash routera, w której
możemy dokonywać własnych zmian w konfiguracji tego firmware. Jak to zwykle bywa, czasem coś
uszkodzimy przez przypadek i masz router nie chce zbytnio działać tak jak byśmy od niego oczekiwali.
Tryb `failsafe` nie jest tym samym co `firstboot`, który resetuje ustawienia do fabrycznych. W
przypadku `failsafe` , wszelkie zmiany jakich dokonaliśmy w procesie konfiguracji zostają i nic nie
tracimy. Jedynie co, to nie jest montowana partycja, na której ten zmiany figurują. Dodatkowo, ilość
odpalonych usług jest ograniczona do minimum. Nie mamy, np. dostępu do internetu, nie działa WiFi,
resolver DNS i nie mamy możliwości uzyskania adresu za pomocą protokołu DHCP. W tym wpisie
przyjrzymy się nieco bliżej temu mechanizmowi.

<!--more-->
## Zanim wejdziemy w tryb failsafe

We wstępie wspomniałem o usługach, które w trybie `failsafe` nie będą uruchamiane. Mamy tam
wyszczególniony serwer DHCP, który przydziela konfigurację hostom w sieci. Bez niego nie uzyskamy
adresu IP, co uniemożliwi nam połączenie. Podobnie sprawa ma się w przypadku połączenia
bezprzewodowego. Jeśli jesteśmy zmuszeni do korzystania z trybu `failsafe` , to róbmy to przewodowo
i przydzielmy danej maszynie statyczny adres IP, tak by połączenie z routerem w trybie ratunkowym
zostało nawiązane bez problemów. Router po wystartowaniu w tym trybie jest dostępny pod adresem
192.168.1.1/24 i możemy się do niego zalogować za pomocą protokołu telnet, tak jak to zwykle robimy
po wgraniu nowszej wersji firmware.

## Aktywacja trybu failsafe

Jeśli problem jest czysto programowy i dotyczy błędu w plikach konfiguracyjnych lub powstał on za
sprawą niezbyt dopracowanych usług to wystarczy zamontować partycję, na której wprowadzane są
zmiany. To jest cała istota OpenWRT, czyli nie wprowadzamy zmian bezpośrednio w firmware, a jedynie
są one aplikowane na bazową konfigurację. W przypadku problemów, można tą nakładkę ściągnąć i
wyeliminować błędy, które za jej sprawą się pojawiły.

Po ustawieniu statycznej konfiguracji na komputerze, resetujemy (lub wyłączamy/włączamy) router i
przyciskamy dość energicznie jeden z jego działających przycisków, inny niż przycisk zasilania.
Dokładny moment, w którym należałoby wcisnąć przycisk można ustalić podsłuchując pakiety w sieci. W
odpowiednim momencie, router wysyła pakiet na adres 192.168.1.255 (broadcast sieci) na port
4919/UDP. Można ustawić odpowiednią regułkę na firewall'u i zalogować to zdarzenie. Na linux'ach w
logu wtedy wygląda to mniej więcej tak:

![]({{< baseurl >}}/img/2016/04/1.tryb-failsafe-iptables-log-router.png#huge)

Po pojawieniu się powyższego komunikatu, trzeba wcisnąć przycisk na routerze i to automatycznie
zainicjuje tryb `failsafe` . Jeśli nie mamy możliwości obserwowania logów, to pozostaje nam
energiczne wciskanie przycisku. Tryb `failsafe` można poznać po częstotliwości migania diody
`system` na routerze. Po przyciśnięciu przycisku w odpowiednim momencie, ta dioda zacznie migać
bardzo szybko.

Pozostaje nam już tylko zalogować się na ruter za pomocą protokołu telnet. Po pomyślny dostaniu się
do systemu, powinien nas przywitać komunikat podobny do tego poniżej:

![]({{< baseurl >}}/img/2016/04/2.tryb-failsafe-logowanie-telnet-openwrt.png#big)

Proces naprawczy rozpoczynamy zawsze od zamontowania partycji, która przechowuje wprowadzane przez
nas zmiany. Robimy to przez wpisanie w terminalu poniższego polecenia:

    # mount_root

Na wypadek, gdyby to powyższe polecenie nie zadziałało, to możemy spróbować tę partycję zamontować
ręcznie przy pomocy `mount` :

    # mount -t jffs2 /dev/mtdblock3 /overlay

## Zmiana zapomnianego hasła do konta root

Różne mogą być przyczyny, które zagnają nas w tryb `failsafe` . Jedną z nich może być zapomniane
hasło do konta administratora root. Bez niego nie zalogujemy się na router. Niemniej jednak w
trybie ratunkowym już jesteśmy zalogowani na to konto i po zamontowaniu partycji możemy wydać
polecenie `passwd` i to hasło ustawić ponownie.

    # passwd
    Changing password for root
    New password:
    Retype password:
    Password for root changed by root

## Edycja ustawień

Jako, że po zamontowaniu partycji, na której są przechowywane wszystkie zmiany jakie wprowadziliśmy
w konfiguracji OpenWRT, to możemy zwyczajnie tę konfigurację teraz przejrzeć. W oparciu o to, co
robiliśmy ostatnio na routerze, możemy postarać się o usunięcie pewnych pakietów z systemu za pomocą
menadżera pakietów `opkg` . Jeśli winna jest literówka, która zaplątała się w jednym z plików
konfiguracyjnych, to bez problemu możemy poprawić ten błąd za pomocą edytora `vi` .

## Kończenie pracy w trybie failsafe

Jeśli ustaliliśmy przyczynę problemów routera i wyeliminowaliśmy ją z powodzeniem, to jedyna
czynność jaka nam teraz pozostała, to zresetowanie routera przy pomocy tego poniższego polecenia:

    # reboot -f

Jeśli nie mamy pojęcia co się popsuło, to możemy [zresetować router do ustawień fabrycznych, tzw.
firstboot]({{< baseurl >}}/post/reset-ustawien-w-openwrt-firstboot/), przez wpisanie w terminalu
`firstboot` i zresetowanie routera.
