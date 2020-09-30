---
author: Morfik
categories:
- OpenWRT
date: "2016-05-10T22:17:38Z"
date_gmt: 2016-05-10 20:17:38 +0200
published: true
status: publish
tags:
- iptables
- ipset
- chaos-calmer
- router
- dnsmasq
title: Jak zablokować Facebook i YouTube w OpenWRT
---

Serwisy społecznościowe takie jak Facebook, Twitter czy YouTube coraz bardziej dają się we znaki
przedsiębiorcom, który muszą cały czas pilnować, by ich pracownicy nie siedzieli ciągle w
internecie, przynajmniej w czasie pracy. Problem nagminnego przebywania w tych w/w portalach można
bardzo łatwo rozwiązać przez... porozmawianie z pracownikami. No może nie zawsze ale co nam szkodzi
spróbować? W przypadku, gdy upomnienia nie są w stanie zmusić ludzi w naszej firmie do pracy, a nie
możemy przy tym ich zwolnić, to możemy pójść o krok dalej i spróbować im założyć blokadę na te
powyższe serwisy. Oczywiście blokada szeregu adresów IP nie wchodzi w rachubę. Korporacje typu
Facebook czy Google mają wiele adresów IP na których świadczą swoje usługi. Nie wszystkie z nich są
uwzględniane na różnego rodzaju listach. Niemniej jednak, tak na dobrą sprawę to nie musimy nawet
znać tych adresów. Jedyne czego nam potrzeba to nazwa domeny oraz kilka pakietów standardowo
dostępnych w repozytorium OpenWRT. Mowa o `iptables` , `dnsmasq` oraz `ipset` . W OpenWRT, przy
pomocy tych narzędzi możemy zaprojektować filtr, który może zablokować ludziom z naszej sieci dostęp
do praktycznie każdego serwisu www. W tym artykule zobaczymy jak taki filtr skonstruować.

<!--more-->
## Filtr w oparciu o iptables, dnsmasq i ipset

Przede wszystkim, trzeba powiedzieć, że w OpenWRT mamy sporo okrojonych pakietów. Do nich zalicza
się `dnsmasq` . W oparciu o tę standardową wersję nie damy rady stworzyć mechanizmu, który rozwiąże
nazwy domen na adresy IP i doda je do seta w `ipset` . Musimy ten pakiet usunąć i wgrać na jego
miejsce `dnsmasq-full` . Instalujemy także pakiet `ipset` :

    # opkg update
    # opkg remove dnsmasq
    # opkg install dnsmasq-full ipset

Mając zainstalowane stosowne pakiety, przejdźmy do projektowania filtru blokującego serwisy
Facebook, Twitter i Youtube. Naturalnie może do być dowolny serwis www czy nawet usługa ale w tym
wpisie ograniczymy się do zablokowania komunikacji tylko z tymi trzema portalami.

Przede wszystkim opiszmy sobie za co będą odpowiadać poszczególne narzędzia i jak ten filtr będzie
realizował swoje zadanie. Gdy użytkownik z naszej sieci LAN w przeglądarce wpisze nazwę domeny
powyższych serwisów, zapytanie DNS zostaje przesłane do serwera DNS w routerze. Realizacją zadania
polegającego na rozwiązaniu nazwy na adres IP i zwróceniu go do hosta lokalnego zajmuje się
`dnsmasq` . Standardowo robi on jedynie za cache DNS i wszystkie zapytania, które nie są w cache,
przesyłane są do upstream'owych serwerów DNS. W przypadku naszego filtra, domeny `facebook.com` ,
`youtube.com` oraz `twitter.com` zostaną rozwiązane jak gdyby nigdy nic. W ten sposób zostaną
uzyskane adresy IP tych stron, które `dnsmasq` będzie dodawał do budowanej listy adresów. Takie
listy są tworzone i zarządzane przez `ipset` . Jeśli mamy już listę i na niej kilka adresów, to
`iptables` może na niej operować i w oparciu o nią może blokować lub też przepuszczać ruch.

## Tworzenie listy ipset dla serwisu Facebook, Twitter i YouTube

Wiemy zatem jak ma działać nasz filtr. Musimy teraz skonfigurować poszczególne jego elementy.
Zaczniemy od konfiguracji list adresowych. Możemy to zrobić przez plik `/etc/config/firewall` .
[Mechanizm firewall'a w
OpenWRT](/post/filtr-pakietow-sieciowych-w-openwrt-firewall/) jest w stanie
stworzyć [odpowiednie sety z adresami
IP](https://lists.openwrt.org/pipermail/openwrt-devel/2013-May/019937.html). Wszystko czego nam
trzeba, aby skonfigurować listy `ipset` , to dodanie w tym pliku tego poniższego bloku:

    config ipset
        option enabled          '1'
        option name             'socjal'
        option family           'ipv4'
        option storage          'hash'
        option match            'dest_ip'
        option maxelem          '256'
        option timeout          '7200'

Warto zaznaczyć, że jest to lista adresów IPv4, także jeśli nasz ISP zapewnia wsparcie dla protokołu
IPv6, musimy utworzyć drugi set i odpowiednio dostosować sobie opcje `name` i `family` . Listy nie
muszą być duże. 256 elementów w zupełności nam wystarczy. Im większą wartość określimy w opcji
`maxelem` , tym więcej pamięci taki set będzie utylizował po utworzeniu, nawet jeśli nie zawiera
żadnego adresu IP. Jako, że chcemy zablokować połączenie na zdalny adres to w opcji `match`
określamy `dest_ip` . Z kolei opcja `timeout` precyzuje ile czasu każdy z wpisów ma się znajdować
na liście. Jest to kluczowa opcja, która ma na celu dbanie o czystość listy i usuwanie z niej
niepotrzebnych wpisów. Adresy mogą ulec zmianie z różnych powodów. Może na portalu Facebook zadziała
jakiś load balancer lub też zostanie aktywowany jakiś failover na Youtube. W takich przypadkach
zostaniemy skierowani pod inny adres IP. Nie ma sensu marnować pamięci RAM na te nieaktualne w danej
chwili wpisy, bo i tak z nich nie skorzystamy. W przypadku, gdy nikt nie będzie próbował połączyć
się z Facebook'em przez 2 godziny, to wpisy zostaną usunięte. Niemniej jednak, w dalszym ciągu
filtr działa, bo przed połączeniem znów trzeba wykonać zapytanie DNS, a ono z kolei doda aktualny
adres serwisu Facebook na listę. Upewnijmy się także, że opcja `enabled` jest włączona.

## Reguła dla iptables

Mamy zatem skonfigurowany set. Potrzebujemy jednak reguły dla `iptables` , która zablokuje ruch w
oparciu o adresy znajdujące się na tej liście. Musimy dodać kolejny blok do pliku
`/etc/config/firewall` . Wygląda on mniej więcej tak jak ten poniżej:

    config rule
        option name 'socjal'
        option src 'lan'
        option dest 'wan'
        option proto 'all'
        option ipset 'socjal'
        option family 'ipv4'
        option target 'REJECT'

Ta reguła ma za zadanie dopasować ruch w oparciu o set `socjal` . Zostanie ona dodana do łańcucha
`zone_lan_forward` , tak jak to jest zobrazowane na poniższej fotce:

![](/img/2016/05/1.iptables-blokowanie-facebook-youtube-twitter-openwrt-regula.png#huge)

## Konfiguracja dnsmasq

Pozostało nam jeszcze skonfigurowanie `dnsmasq` . W pliku `/etc/dnsmasq.conf` musimy dodać trzy
parametru odpowiedzialne za konfigurację cache DNS. Musimy ustawić jego rozmiar oraz czas ważności
wpisów. Dopisujemy zatem do tego pliku poniższe parametry:

    cache-size=10000
    min-cache-ttl=3600
    max-cache-ttl=7200

Teraz przechodzimy do edycji pliku `/etc/config/dhcp` i określamy w nim domeny, których adresy IP
mają wędrować do listy `ipset` :

    config dnsmasq
        ...
        list ipset '/fb.com/socjal'
        list ipset '/facebook.com/socjal'
        list ipset '/twitter.com/socjal'
        list ipset '/youtube.com/socjal'

Zapisujemy plik i restartujemy router. Po chwili urządzenie powinno być gotowe do pracy, a filtr
aktywny.

## Test filtra domen

Przechodzimy na komputer i w pasku adresu przeglądarki wpisujemy adres jednej z dodanych wyżej
domen. Niech to będzie `facebook.com` . Strona nie powinna się nam załadować. Natomiast na liście
adresów powinno pojawić się kilka wpisów. Sprawdzamy to poleceniem `ipset list socjal` :

![](/img/2016/05/2.ipset-lista-blokowanie-facebook-youtube-twitter-openwrt.png#big)

Powyżej widzimy, że trochę tych adresów się uzbierało. Mamy też statystyki dotyczące wykorzystania
pamięci operacyjnej przez ten set. Przy każdym wpisie zaś widnieje `timeout` wraz z pewną wartością.
Przy dodawaniu wpisu jest ona ustawiana na 7200. Co jedną sekundę ta wartość zmniejsza się o 1. Gdy
licznik wybije 0, wpis jest usuwany.

## Problemy związane z filtrem

Zaprojektowany wyżej filtr nie jest doskonały. Osoby, które orientują się nieco bardziej w tematyce
IT, wiedzą, że kluczem do obejścia tego mechanizmu jest serwer DNS. Można bowiem nie korzystać z
adresu DNS routera otrzymanego w lease DHCP. Zamiast niego, można korzystać bezpośrednio z
upstream'owego serwera DNS i to do niego przesyłać zapytania o domeny. W takim przypadku, domena nie
zostanie rozwiązana na routerze, a jej adres nie zostanie dodany do seta. W efekcie set będzie pusty
i `iptables` nie będzie w stanie dopasować pakietu. Niemniej jednak, można wymusić przekierowanie
zapytań DNS również przy pomocy `iptables` . W ostateczności można też zablokować komunikację z
adresami IP providerów DNS, np. góglowski 8.8.8.8 i 8.8.4.4 . Można te adresy zebrać i zrobić z nich
statyczną listę i ją przekazać do `iptables` .
