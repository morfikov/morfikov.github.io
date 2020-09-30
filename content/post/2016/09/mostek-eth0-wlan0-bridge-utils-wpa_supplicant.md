---
author: Morfik
categories:
- Linux
date: "2016-09-18T13:01:58Z"
date_gmt: 2016-09-18 11:01:58 +0200
published: true
status: publish
tags:
- wifi
- sieć
- bridge
title: Mostek eth0 + wlan0 z bridge-utils i wpa_supplicant
---

Posiadając w komputerze kilka interfejsów sieciowych, prędzej czy później dostrzeżemy wady jakie
niesie ze sobą konfiguracja wszystkich posiadanych kart sieciowych. Skonfigurowanie szeregu
interfejsów przewodowych nie stanowi raczej większego wyzwania. Można je spiąć w jeden za pomocą
bonding'u czy też konfigurując wirtualny interfejs mostka (bridge). A co w przypadku bezprzewodowych
interfejsów? Tu również możemy [skonfigurować interfejs
bond0](/post/konfiguracja-interfejsow-bond-bonding/) lub też podpiąć interfejs
`wlan0` pod mostek. Jako, że bonding już opisywałem, to w tym artykule zajmiemy się mostkowaniem
interfejsu przewodowego i bezprzewodowego, które zwykle dostępne są w naszych laptopach. Ten proces
zostanie opisany w oparciu o dystrybucję linux'a Debian i skontrastujemy go sobie z w/w bonding'iem.

<!--more-->
## Konfiguracja mostka (bridge)

Wirtualny interfejs mostka jest w stanie spiąć szereg interfejsów fizycznych, które mamy dostępne w
komputerze. Takie rozwiązanie ma tę zaletę, że mamy w systemie do skonfigurowania tylko jeden
interfejs sieciowy. Mając jeden interfejs, mamy jeden adres IP, a to z kolei upraszcza nieco naszą
pracę w sieci.

Do skonfigurowania mostka w Debianie, potrzebne są nam odpowiednie narzędzia. Te z kolei znajdują
się w pakiecie `bridge-utils` . Nie powinno być raczej problemów z instalacją tego pakietu, zatem
przejdźmy od razu do konfiguracji mostu w pliku `/etc/network/interfaces` . Edytujemy ten plik i
wrzucamy do niego poniższą zawartość:

    iface wlan0 inet manual
    iface eth0 inet manual

    auto br-lan
    iface br-lan inet dhcp
        bridge_ports eth0 wlan0
    #   bridge_maxwait 10
    #   bridge_waitport 10
        wpa-iface wlan0
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
        wpa-driver nl80211
        wpa-bridge br-lan

Interfejsy, które zamierzamy podpiąć pod mostek, wyjmujemy spod działania różnych narzędzi
konfiguracyjnych, np. Network Manager. Dlatego też wyżej mamy dwie linijki z `iface` , które mają
ustawiony tryb `manual` . Ta duża zwrotka ma na celu skonfigurowanie interfejsu mostka.

Jako, że chcemy, by interfejs `br-lan` był podnoszony na starcie systemu, to dodajemy dyrektywę
`auto` . Chcemy także, aby ten interfejs pobierał konfigurację sieciową przez protokół DHCP. Dlatego
w `iface br-lan` określiliśmy sobie `dhcp` . Niemniej jednak, możemy także statycznie skonfigurować
sobie ten interfejs. My jednak pozostaniemy przy dynamicznej konfiguracji.

Opcja `bridge_ports` określa interfejsy sieciowe, które mają być zarządzane przez mostek. Od tej
pory, to mostek będzie zarządzał statusem tych interfejsów w zależności od tego czy podepniemy
przewód do portu ethernet, czy też podłączymy się do sieci WiFi.

Opcje `bridge_waitport` oraz `bridge_maxwait` mają za zadanie opóźnić nieco konfigurację mostka.
Chodzi o to, że niektóre interfejsy wymagają czasu by stały się dostępne w systemie, a bez nich
mostek nam się źle skonfiguruje. Czas jest w sekundach. Jeśli skrypty konfigurujące mostek mają
problemy z dostępnością interfejsów, które zamierzamy do niego podpiąć, to powinniśmy dostosować
sobie odpowiednio czas w tych opcjach.

Opcje z `wpa-*` odnoszą się do konfiguracji interfejsu bezprzewodowego. W `wpa-iface` określamy
nazwę interfejsu bezprzewodowego, który podpinamy pod mostek. Z kolei w `wpa-conf` wskazujemy plik
z konfiguracją sieci WiFi. W przypadku większości nowych kart WiFi w `wpa-driver` ustawiamy
`nl80211` , w przypadku pozostałych `wext` .

Kluczowa sprawa to poprawne wskazanie interfejsu mostka w `wpa-bridge` . Bez tej opcji AP będą
blokowały połączenie za sprawą różnych adresów MAC. Standardowo system uwierzytelni się w sieci
bezprzewodowej za pomocą karty WiFi, a ta ma inny MAC niż ten zdefiniowany w interfejsie mostka.
Później jak system będzie chciał przesłać jakiś pakiet, to zrobi to podając adres MAC mostka i ten
pakiet zostanie zablokowany przez AP. Definiując ten parametr nie musimy sobie zawracać głowy
dodatkowymi operacjami na filtrze pakietów `iptables` .

Więcej informacji na temat parametrów mostka można znaleźć w [man
bridge-utils-interfaces](http://manpages.ubuntu.com/manpages/xenial/man5/bridge-utils-interfaces.5.html).

## Tryb WDS na potrzeby interfejsu WiFi

Mostki zostały stworzone głównie z zamiarem obsługi interfejsów przewodowych lub bezprzewodowych w
trybie AP. W naszym przypadku, gdy zamierzamy wykorzystać taki mostek w połączeniu z interfejsem
WiFi w trybie klienta (STA), to możemy napotkać całą masę problemów.

Standardowo takiego bezprzewodowego interfejsu w trybie klienta nie da rady podpiąć pod mostek.
Trzeba na karcie WiFi aktywować `4addr` odpowiadający za tryb WDS. Możemy to zrobić wykorzystując
`iw` (dostępne w pakiecie o tej samej nazwie). Następnie w pliku `/etc/network/interfaces` w bloku
konfigurującym interfejs `br-lan` dodajemy poniższe linijki:

    iface br-lan inet dhcp
    ...
        pre-up iw dev wlan0 set 4addr on
        post-down iw dev wlan0 set 4addr off

Zatem włączanie trybu WDS na karcie sieciowej laptopa mamy z głowy. Teraz interfejs `wlan0`
powinniśmy być w stanie dodać do naszego mostka. Niemniej jednak, to nie koniec naszej pracy. Na
każdym routerze, z którym zamieramy się łączyć, tryb WDS musi być również aktywowany. W przeciwnym
razie nie damy rady się podłączyć po WiFi.

W przypadku domowych routerów bezprzewodowych, zwłaszcza tych z firmware OpenWRT/LEDE, tryb WDS
możemy ustawić w pliku `/etc/config/wireless` dodając do konfiguracji sieci ten poniższy parametr:

    config wifi-iface
    ...
            option wds '1'

## Pętla za sprawą broadcast'u i protokół STP

Zgodnie z [informacjami zawartymi
tutaj](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/5234-5.html),
przy konfiguracji mostu może nastąpić pętla, która jest w stanie ubić naszą sieć. Generalnie rzecz
biorąc, mamy dwa połączenia i musimy je skonfigurować tak, by tylko jedno z nich było aktywne w
danym czasie. W przeciwnym razie pakiety broadcast, np. podczas fazy pobierania adresacji za sprawą
protokołu DHCP, będą krążyć między naszym mostkiem a switch'em routera, co ostatecznie doprowadzi
naszą sieć do zawału. Taka pętla objawia się w poniższy sposób:

    kernel: br-lan: received packet on wlan0 with own address as source address
    kernel: br-lan: received packet on eth0 with own address as source address
    kernel: br-lan: received packet on wlan0 with own address as source address
    kernel: net_ratelimit: 12758 callbacks suppressed

Takich komunikatów jest cała masa. Dodatkowo serwer DHCP na routerze WiFi dostaje ogromną ilość
zapytań o adresację IP i właśnie te pakiety w końcu wykończą naszą sieć.

By rozwiązać ten problem, możemy korzystać tylko z jednego interfejsu sieciowego, np. `wlan0` lub
`eth0` w danej chwili. Jeśli mamy zamiar podłączyć przewód do portu ethernet, to musimy pierw
wyłączyć WiFi i podobnie w drugą stronę. Z tym, że takie rozwiązanie nie jest zbyt wygodne i
musimy poszukać sobie innego sposobu.

Możemy skorzystać z protokołu STP (Spanning Tree Protocol). Ma on za zadanie tak skonfigurować
mostki i switch'e, by pętle w sieci nie występowały. Za każdym razem, gdy interfejs będzie
podnoszony, STP sprawdzi konfigurację sieci i ustali, jak ma przebiegać trasa pakietu. W przypadku
włączenia zarówno interfejsu `wlan0` i `eth0` , jeden z nich zostanie skonfigurowany jako backup na
wypadek utraty połączenia przez drugi interfejs. By skonfigurować STP na interfejsie `br-lan`
dodajemy do konfiguracji mostu te poniższe wpisy:

    iface br-lan inet dhcp
        ...
        bridge_stp on
        bridge_portprio eth0 10
        bridge_portprio wlan0 20
        bridge_fd 3
        bridge_maxage 6

Opcja `bridge_stp` aktywuje protokół STP, natomiast `bridge_portprio` ustawia priorytet interfejsom
podpiętym do mostka. Im mniejsza wartość, tym wyższy priorytet. W takiej konfiguracji jak powyżej,
to przez interfejs `eth0` będą przepuszczane pakiety w przypadku, gdy oba interfejsy będą aktywne.
Gdy odłączymy przewód od portu ethernet, po chwili nastąpi rekonfiguracja sieci i zostaniemy
przełączeni na interfejs `wlan0` . Po podłączeniu przewodu, zostaniemy przepięci z powrotem na
interfejs `eth0` .

Takie przełączanie trwa chwilę. By być szczerym, to trwa ono dość długo, bo prawie jedną minutę.
Musimy ten czas nieco skrócić i do tego celu służą opcje `bridge_fd` oraz `bridge_maxage` . Każda z
nich ma minimalną wartość jaką możemy ustawić. Te powyższe wartości są minimalne, a czas jest
liczony w sekundach. Trzeba jednak pamiętać, że im większa jest sieć, tym te wartości powinny być
większe. Na nasze standardy wystarczą te minimalne.

Przy takim ustawieniu jak wyżej, przejście z `eth0` na `wlan0` zajmie około 9 sekund. Natomiast z
`wlan0` na `eth0` około 2 sekund.

## Mostek i roaming WiFi

Przy takim rozwiązaniu jak to opisane powyżej będą występować częściowe problemy z roamingiem w
sieci WiFi. Chodzi o to, że demon ramingowy nie może zostać w takiej sytuacji uruchomiony i przy
zmianie sieci nie zostanie pobrana ponownie adresacja IP. W efekcie może i przełączenie do nowego AP
nastąpi ale system będzie korzystał ze starego adresu IP, bo parametr `id_str` , który ustawiany
jest w pliku `/etc/wpa_supplicant/wpa_supplicant.conf` nie ma żadnej mocy i w efekcie system nie
przełącza się.

Jeśli chodzi zaś o [roaming w tej samej sieci
WiFi](/post/jak-skonfigurowac-roaming-wifi-wpa_supplicant-linux/), to działa on bez
zarzutu, bo tutaj nie trzeba zmieniać adresacji IP.

## Testy mostka

Poniżej jest test konfiguracji mostka.

![](/img/2016/09/1.mostek-bridge-linux-debian-test.png#huge)

Jak widzimy, mamy tutaj aktywny zarówno interfejs `wlan0` jak i `eth0` . Interfejs `wlan0` został
przełączony w stan blokady i wykorzystywany jest interfejs `eth0` , przez który została otrzymana
konfiguracja dla `br-lan` . Sprawdźmy zatem co się stanie po odłączeniu i podłączeniu przewodu do
portu ethernet:

![](/img/2016/09/2.mostek-bridge-linux-debian-test.png#huge)

Po odłączeniu przewodu mamy tylko jeden interfejs aktywny i to przez niego zostają przepuszczone
pakiety. W przypadku podpięciu przewodu do portu ethernet, pojawia się drugi interfejs aktywny i
system decyduje się wyłączyć interfejs bezprzewodowy by uniknąć pętli.

## Bridging czy bonding

Jeśli miałbym być szczery, to dla mnie za dużo kombinowania z tym całym mostkiem w przypadku
bezprzewodowych interfejsów WiFi. Gdybym miał wybierać między bridging a bonding, to jednak wolę
bonding. Przynajmniej w jego przypadku nie ma problemów z przełączaniem się między WiFi i wire, no i
też nie trzeba włączać żadnego trybu WDS, zwłaszcza na routerach.
