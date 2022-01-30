---
author: Morfik
categories:
- OpenWRT
date:    2016-05-02 15:25:31 +0200
lastmod: 2022-01-30 19:50:00 +0100
published: true
status: publish
tags:
- chaos-calmer
- sieć
- router
- mac
- prywatność
GHissueID: 472
title: Jak sklonować adres MAC w OpenWRT
---

Każde urządzenie sieciowe ma przypisany jakiś [adres MAC][1]. Jest to numer, który identyfikuje je
w strukturze sieci. Gdy zachodzi potrzeba rozbudowania sieci domowej, możemy napotkać problemy z
naszym obecnym ISP. Załóżmy, że posiadaliśmy do tej pory jeden komputer, który był wpięty
bezpośrednio do łącza ISP. Jeśli dokupiliśmy router i podłączymy go w miejsce komputera, to
urządzenie, które widzi nasz provider, ulega zmianie. W takim przypadku nie ma znaczenia samo
urządzenie, a liczy się jedynie zmiana adresu MAC. Providerzy internetowi mają powiązane adresy MAC
z adresami IP i nowo wpięty router nie otrzyma adresu IP, bo ma nieautoryzowany MAC. W takiej
sytuacji zwykle wystarczy telefon do ISP z prośbą o aktualizację tego adresu. Niemniej jednak,
czasami ISP każą sobie dodatkowo płacić za tę czynność. Jeśli jesteśmy postawieni w takiej sytuacji,
to możemy sklonować sobie adres MAC tej maszyny, którą wcześniej widział nasz ISP. Z jego
perspektywy nic się nie zmieni, a my będziemy mogli sobie rozdzielić sygnał na tyle komputerów, ile
tylko chcemy, nie ponosząc przy tym dodatkowych opłat. W niniejszym artykule zostanie opisany
sposób na przeprowadzenie klonowania adresu MAC na routerze z wgranym firmware OpenWRT.

<!--more-->
## Klonowanie adresu MAC (plik /etc/config/network)

Procedura klonowania adresu MAC w routerach jest podobna do [generowania losowego adresu na
interfejsie sieciowym][2] w pierwszej lepszej dystrybucji linux'a. W obu przypadkach musimy
przepisać adres interfejsowi karty sieciowej. Niemniej jednak, trzeba pamiętać, że ISP posiada adres
MAC naszej starej maszyny. Dlatego nie możemy użyć dowolnego adresu MAC i musimy sklonować ten
obecny w bazie danych naszego provider'a. Zatem jeśli do sieci był podpięty zwykły komputer, to
jego adres MAC musimy odczytać w ustawieniach sieciowych. Na linux'ie służą do tego polecenia
`ifconfig` lub `ip` . Poniżej przykład:

![sklonowac-adres-mac-komputer-openwrt](/img/2016/05/1.sklonowac-adres-mac-komputer-openwrt.png#huge)

Widoczne wyżej `link/ether 3c:4a:92:00:4c:5b` , to właśnie adres MAC karty sieciowej w komputerze,
który został podłączony bezpośrednio do łącza internetowego. To ten numerek musimy uwzględnić w
konfiguracji OpenWRT. Logujemy się zatem na router via SSH i przechodzimy do edycji pliku
`/etc/config/network` . Tam odszukujemy sekcję `wan` i dopisujemy tę poniższą linijkę:

    config interface 'wan'
    ...
        option macaddr '3c:4a:92:00:4c:5b'

Zapisujemy plik i restartujemy router. Adres MAC powinien zostać sklonowany, co możemy odczytać w
konfiguracji routera wydając polecenie `ifconfig` lub `ip` wskazując w argumencie interfejs WAN:

![sklonowac-adres-mac-komputer-openwrt](/img/2016/05/2.sklonowac-adres-mac-komputer-openwrt.png#big)


[1]: https://pl.wikipedia.org/wiki/Adres_MAC
[2]: /post/jak-przypisac-losowy-adres-mac-interfejsu/
