---
author: Morfik
categories:
- OpenWRT
date: "2016-05-02T17:25:31Z"
date_gmt: 2016-05-02 15:25:31 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
- mac
title: Jak sklonować adres MAC w OpenWRT
---

Każde urządzenie sieciowe ma inny [adres MAC](https://pl.wikipedia.org/wiki/Adres_MAC). Jest to
numer, który identyfikuje je w strukturze sieci. Gdy zachodzi potrzeba rozbudowania sieci domowej,
możemy napotkać problemy z naszym obecnym ISP. Załóżmy, że posiadaliśmy do tej pory jeden komputer,
który był wpięty bezpośrednio do łącza ISP. Jeśli dokupiliśmy router i podłączymy go w miejsce
komputera, to urządzenie, które widzi nasz provider, ulega zmianie. Konkretnie, to nie ma znaczenia
samo urządzenie. Liczy się jedynie adres MAC. Providerzy internetowi mają powiązane adresy MAC z
adresami IP i nowo wpięty router nie otrzyma adresu IP, bo ma nieautoryzowany MAC. W takiej sytuacji
zwykle wystarczy telefon do ISP z prośbą przepisanie adresu. Niemniej jednak, czasami ISP każą sobie
dodatkowo płacić za tę czynność. Jeśli jesteśmy postawieni w takiej sytuacji, to możemy sklonować
sobie adres MAC tej maszyny, którą wcześniej widział nasz provider internetowy. Z jego perspektywy
nic się nie zmieni, a my będziemy mogli sobie rozdzielić sygnał na tyle komputerów, ile tylko
chcemy. W tym wpisie zobaczymy jak pod OpenWRT przeprowadzić klonowanie adresu MAC.

<!--more-->
## Jak sklonować adres MAC

Procedura klonowania adresu MAC w routerach jest podobna do [generowania losowego
adresu]({{< baseurl >}}/post/jak-przypisac-losowy-adres-mac-interfejsu/). W obu przypadkach musimy
przepisać adres interfejsowi karty sieciowej. Niemniej jednak, w przypadku ISP, ten posiada adres
MAC naszej starej maszyny. Dlatego nie możemy użyć dowolnego adresu MAC i musimy sklonować ten
obecny w bazie danych providera. Zatem jeśli do sieci był podpięty zwykły komputer, to jego adres
MAC musimy odczytać w opcjach sieciowych. Na linux'ie służą do tego polecenia `ifconfig` lub `ip` .
Poniżej przykład:

![]({{< baseurl >}}/img/2016/05/1.sklonowac-adres-mac-komputer-openwrt.png)

Widoczne wyżej `link/ether 3c:4a:92:00:4c:5b` , to właśnie adres MAC tej karty sieciowej. To ten
numer musimy uwzględnić w konfiguracji OpenWRT. Logujemy się zatem na router i przechodzimy do
edycji pliku `/etc/config/network` . Tam odszukujemy sekcję `wan` i dopisujemy tę poniższą linijkę:

    config interface 'wan'
    ...
        option macaddr '3c:4a:92:00:4c:5b'

Zapisujemy plik i restartujemy router. Adres MAC powinien zostać sklonowany, co możemy odczytać w
konfiguracji routera wydając polecenie `ifconfig` wskazując w argumencie interfejs WAN:

![]({{< baseurl >}}/img/2016/05/2.sklonowac-adres-mac-komputer-openwrt.png)
