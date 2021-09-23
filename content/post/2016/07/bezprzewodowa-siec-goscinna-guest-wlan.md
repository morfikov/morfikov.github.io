---
author: Morfik
categories:
- OpenWRT
date: "2016-07-06T21:00:34Z"
date_gmt: 2016-07-06 19:00:34 +0200
published: true
status: publish
tags:
- wifi
- chaos-calmer
- sieć
- router
title: Bezprzewodowa sieć gościnna (guest WLAN)
---

Routery WiFi zwykle oferują jedną sieć bezprzewodową, do której użytkownicy mogą się łączyć po
podaniu nazwy ESSID oraz hasła. W przypadku znanych nam osób chcących korzystać z udostępnianego
przez nas AP, taka sieć powinna nam w pełni wystarczyć. Problem jednak zaczyna się w przypadku tych
użytkowników, którym chcemy zezwolić na dostęp do naszej sieci WiFi ale nie darzymy ich zbytnio
wysokim kredytem zaufania. By ten problem rozwiązać, w OpenWRT możemy skonfigurować [mechanizm zwany
"guest WLAN"](https://wiki.openwrt.org/doc/recipes/guest-wlan), czyli bezprzewodowa sieć gościnna. W
tym artykule zobaczymy jak odseparować od siebie hosty w sieci LAN od tych w sieci gościnnej.

<!--more-->
## Co nam daje sieć gościnna

Bezprzewodowa sieć gościnna, którą zamierzamy sobie zaimplementować na routerze, to nic innego jak
dodatkowa sieć WiFi, która ma nieco inne parametry techniczne. Z reguły taka sieć nie posiada
żadnych zabezpieczeń i jest dostępna dla każdego użytkownika znajdującego się w zasięgu routera.
Oczywiście nic nie stoi na przeszkodzie, by założyć tej sieci hasło i korzystać z algorytmu
WPA2-PSK. Niemniej jednak, sieć gościnna powinna być prosta, a łączenie do niej możliwie szybkie i
bezproblemowe.

Dodatkowo, jako że sieć gościnna będzie niezabezpieczona, to musimy zatroszczyć się o hosty w sieci
LAN. Te dwie grupy użytkowników muszą zostać od siebie odseparowane ze względów bezpieczeństwa.
Podobnie sprawa ma się w przypadku dostępu do routera przez użytkowników sieci gościnnej. Raczej nie
chcielibyśmy, by któryś z nich mógł nawiązywać jakiekolwiek połączenia z tym urządzeniem i mieszać w
ustawieniach systemowych.

Trzeba także pamiętać o odizolowaniu od siebie wszystkich klientów sieci gościnnej. W ten sposób
użytkownicy w tej sieci nie będą mogli nawiązywać ze sobą połączeń, ani nawet się dostrzec. Ten
mechanizm może być także stosowany w tradycyjnej sieci WiFi ale w warunkach domowych jego
implementacja jest z reguły zbędna. Natomiast w przypadku sieci gościnnej, izolowanie poszczególnych
klientów ma jak najbardziej sens.

## Jak stworzyć sieć gościnną w OpenWRT

Tworzenie sieci gościnnej w OpenWRT nie jest zbytnio skomplikowane ale trzeba poddać edycji szereg
plików konfiguracyjnych. Logujemy się zatem na router i otwieramy plik `/etc/config/network` . W nim
musimy zdefiniować nową sieć i określić jej odpowiednią adresację:

    config 'interface' 'guest'
        option 'proto' 'static'
        option 'ipaddr' '10.0.0.1'
        option 'netmask' '255.255.255.0'

W tym przypadku hosty w sieci LAN mają adresy 192.168.1.0/24, a te z guest WLAN będą miały
10.0.0.0/24. Oczywiście można sobie dobrać dowolną adresację ale trzeba pamiętać, by to były dwie
różne sieci.

Następnie w pliku `/etc/config/wireless` musimy określić parametry sieci WiFi oraz podpiąć pod nią
tę wyżej zdefiniowaną adresację. Najprościej jest skopiować konfigurację naszego domowego AP, która
figuruje w bloku `config wifi-iface` i dostosować odpowiednie wartości:

    config wifi-iface 'guest'
        option device       'radio0'
        option network      'guest'
        option mode         'ap'
        option ssid         'Guest-WLAN'
        option encryption   'none'
    #   option encryption   'psk2+aes'
    #   option key          ''
        option isolate      '1'
        option disabled     '0'

W powyższym bloku kluczowe jest określenie prawidłowej nazwy w opcji `network` . Musi ona pasować do
tej, którą zdefiniowaliśmy w pliku `/etc/config/network` . Nie stosujemy też żadnego szyfrowania,
przez co nasz AP będzie otwarty dla każdego użytkownika, który będzie chciał się podłączyć do sieci.
Jeśli jednak chcemy założyć hasło, to w `encryption` podajemy `psk2+aes` oraz w `key` umieszczamy
hasło do sieci WiFi. Z kolei, by odizolować od siebie wszystkie hosty w sieci gościnnej, musimy
uwzględnić także opcjię `isolate` .

Sieć gościnna ma skonfigurowaną już adresację ale hosty w sieci muszą jakoś otrzymać adres IP i
pozostałe parametry połączenia. Musimy zatem skonfigurować też serwer DHCP, który będzie obsługiwał
klientów w sieci gościnnej i przydzielał im odpowiednią adresację. W tym celu edytujemy plik
`/etc/config/dhcp` i dodajemy w nim ten poniższy blok kodu:

    config dhcp 'guest'
        option interface 'guest'
        option start '50'
        option limit '200'
        option leasetime '2h'
        option dhcpv6 'server'
        option ra 'server'

Ostatnia rzecz, którą nam pozostała, to skonfigurowanie firewall'a. Pamiętajmy, że sieć gościnna ma
mieć odmówiony dostęp do samego routera za wyjątkiem serwera DNS i DHCP. Edytujemy plik `
/etc/config/firewall` i dodajemy do niego poniższy kod:

    config 'zone'
        option 'name' 'guest'
        option 'network' 'guest'
        option 'input' 'REJECT'
        option 'forward' 'REJECT'
        option 'output' 'ACCEPT'

    config 'forwarding'
        option 'src' 'guest'
        option 'dest' 'wan'

    config 'rule'
        option 'name' 'Allow DNS Queries'
        option 'src' 'guest'
        option 'dest_port' '53'
        option 'proto' 'tcpudp'
        option 'target' 'ACCEPT'

    config 'rule'
        option 'name' 'Allow DHCP request'
        option 'src' 'guest'
        option 'src_port' '67-68'
        option 'dest_port' '67-68'
        option 'proto' 'udp'
        option 'target' 'ACCEPT'

Najpierw definiujemy strefę `guest` na zaporze i konfigurujemy wstępnie przepływ pakietów. Następnie
zezwalamy na forwarding pakietów z sieci gościnnej do internetu. Dalej mamy dwie reguły zezwalające
na komunikację na portach protokołów DNS i DHCP.

## Testy sieci gościnnej

To w zasadzie cała konfiguracja, którą musieliśmy uwzględnić przy tworzeniu sieci gościnnej na
routerze z OpenWRT. By przetestować te ustawienia, musimy zresetować router i spróbować się
zalogować na niego podając dwa różne ESSID: jeden od zwykłej sieci WiFi, drugi od sieci gościnnej.
W przypadku naszego zwykłego AP, powinniśmy zostać zalogowani na router:

![](/img/2016/07/1.openwr-guest-network-wlan-siec-goscinna.png#big)

Jeśli teraz spróbujemy podłączyć się do routera z sieci gościnnej, to ten krok nie uda nam się.
Niemniej jednak, wciąż mamy połączenie z internetem:

![](/img/2016/07/2.openwr-guest-network-wlan-siec-goscinna.png#big)

Warto wspomnieć, że sieć gościnna może dość znacznie obciążać nam łącze, jako że sporo osób może z
niej korzystać. By się uporać z tego typu problemem, trzeba będzie pomyśleć o zaimplementowaniu
jakiegoś [mechanizmu QoS](/post/quality-service-qos-w-openwrt/). Niekoniecznie
trzeba ręcznie wyrzeźbić sobie cały setup. Możemy zaprzęgnąć do pracy [skrypty z pakietu
qos-scripts](/post/ksztaltowanie-ruchu-qos-scripts-openwrt/).
