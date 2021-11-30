---
author: Morfik
categories:
- OpenWRT
date:    2021-11-30 20:25:00 +0100
lastmod: 2021-11-30 20:25:00 +0100
published: true
status: publish
tags:
- router
- modem
- lte
- huawei
- e3372
- usb
GHissueID: 583
title: Automatyczny restart połączenia LTE na routerze WiFi z OpenWRT
---

Od już dłuższego czasu (będzie parę lat) korzystam z internetu LTE zamiast tradycyjnego połączenia
przewodowego. Głównie ze względu na fakt, że w mojej okolicy nie ma praktycznie żadnych szanujących
się ISP, z którymi warto by wejść w interakcję i podpisać z nimi jakąś sensową umowę. Poza tym, dla
osób mojego pokroju, które cenią sobie mobilność, internet stacjonarny i tak jest mało praktyczny.
Dlatego w moim domowym routerze mam wgrany firmware OpenWRT umożliwiający zainstalowanie na tym
urządzeniu odpowiedniego oprogramowania obsługującego modemy LTE podłączane przez port USB.
Połączenie sieciowe ze światem zwykle działa prawidłowo ale z jakiegoś powodu jest ono zrywane.
Zwykle taka sytuacja ma miejsce w środku nocy (czasami parokrotnie), zwłaszcza gdy przekroczę limit
danych i do końca okresu rozliczeniowego muszę przemęczyć się z lejkiem 1 mbit/s. Gdy ten lejek
jest aplikowany, to zwykle odpalam sobie torrent'a, tak by pobrać najnowsze obrazy ISO tej czy
innej dystrybucji linux'a. Niemniej jednak, jak mi net rozłączą, to nie załączy się on ponownie sam
z siebie. Modem Huawei E3372s-153 w wersji NON-HiLink zdaje się pracować poprawnie, bo świeci się
na nim dioda sugerująca, że połączenie z internetem jest nawiązane. Dlatego też postanowiłem w
końcu ten problem rozwiązać raz na zawsze i mieć przy tym nieco spokojniejszy sen.

<!--more-->
## Problemy z modemem LTE przy restarcie interfejsu WAN

[Na necie można spotkać wiele rozwiązań][1], które umożliwiają monitorowanie połączenia sieciowego
pod kątem jego aktywności czy sprawności. Zwykle jednak te sposoby radzenia sobie z zanikiem
internetu dotyczą przewodowych połączeń, gdzie zresetowaniu podlega interfejs WAN w przypadku
wykrycia utraty odpowiedzi na zapytania `ping` . Nie byłoby w zasadzie żadnego problemu, gdyby to
rozwiązanie przedstawione w powyżej podlinkowanym artykule działało ale rzecz w tym, że nie działa.

Ja u siebie na OpenWRT mam skonfigurowany interfejs LTE obok interfejsu WAN i te dwa interfejsy
mogą działać niezależnie, w zależności od tego czy chcę wykorzystywać router do połączenia
przewodowego lub LTE. Generalnie to ten interfejs jest konfigurowany w poniższy sposób:

    uci set network.lte=interface
    uci set network.lte.proto='ncm'
    uci set network.lte.device='/dev/cdc-wdm0'
    uci set network.lte.service='preferlte'
    uci set network.lte.pdptype='IP'
    uci set network.lte.apn='internet'
    uci set network.lte.pincode='internet'
    uci set network.lte.ipv6='auto'
    uci set network.lte.peerdns='0'
    uci add_list network.lte.dns='127.0.0.1'

    uci del firewall.@zone[1].network
    uci set firewall.@zone[1].network='wan wan6 lte'

    uci commit

Niestety w przypadku mojego modemu Huawei E3372s-153 w wersji NON-HiLink przy próbie zresetowania
połączenia za sprawą polecenia `ifup lte` , w logu systemowym routera można zaobserwować poniższe
komunikaty:

    ...
    netifd: lte_4 (10284): udhcpc: received SIGTERM
    netifd: lte_4 (10284): udhcpc: unicasting a release of 10.73.3.94 to 10.73.3.93
    netifd: lte_4 (10284): udhcpc: sending release
    netifd: lte_4 (10284): udhcpc: entering released state
    netifd: lte (11278): Stopping network lte
    netifd: lte_4 (10284): Command failed: Permission denied
    netifd: Interface 'lte_4' is now down
    netifd: Network alias '' link is down
    netifd: Interface 'lte_4' has link connectivity loss
    netifd: Interface 'lte_4' is disabled

Czyli interfejs został rozłączony poprawnie ale dalej w logu mamy coś takiego:

    netifd: lte (11278): sending -> AT^NDISDUP=1,0
    netifd: lte (11278): Command failed: Permission denied
    netifd: Interface 'lte' is now down
    netifd: Interface 'lte' is setting up now
    netifd: lte (11340): Failed to get modem information
    netifd: lte (11449): Stopping network lte
    netifd: lte (11449): Failed to get modem information
    netifd: Interface 'lte' is now down
    netifd: Interface 'lte' is setting up now
    netifd: lte (11508): Failed to get modem information
    netifd: lte (11585): Stopping network lte
    netifd: lte (11585): Failed to get modem information
    ...

I tak w nieskończoność, czego efektem jest brak możliwości skonfigurowania modemu LTE do pracy, co
oczywiście przekłada się na brak internetu.

Jedyną opcją by wybrnąć z tej sytuacji jest fizyczne odłączenie modemu LTE od portu USB routera i
podłączenie go ponownie. Można też naturalnie zrestartować router ale czas jego gotowości do pracy
zawsze będzie sporo dłuższy niż samo skonfigurowanie modemu LTE po podpięciu go do portu USB.

## Restart modemu LTE bez wyciągania go z portu USB

Pojawiło się zatem zapytanie na temat tego w jaki sposób rozłączyć modem LTE bez wyciągania go z
portu USB? Okazało się, że nie jest to jakoś specjalnie trudne i można do tego celu skorzystać z
[mechanizmu autoryzacji urządzeń USB][3], który jest wbudowany bezpośrednio w kernel linux'a. To
jest dokładnie ten sam mechanizm, który jest [wykorzystywany np. przez usbguard][2].

### Skrypt watchdog'a

Przerobiłem zatem nieco skrypt watchdog'a, który był podany na stronie `eko.one.pl` do poniższej
postaci:

    #!/bin/sh

    if ! ping -q -c 1 -W 15 8.8.8.8 > /dev/null
    then
        logger "*************************************************"
        logger "**** Brak internetu. Restartuje modem LTE... ****"
        logger "*************************************************"

        echo 0 > /sys/bus/usb/devices/usb3/authorized
        sleep 5
        echo 1 > /sys/bus/usb/devices/usb3/authorized
    fi

W poleceniu `ping` zostały użyte poniższe flagi:

 - `-q`      -- odpowiada za ciche wyjście, tj. nic poza podsumowaniem nie zostanie wydrukowane w
                terminalu.
 - `-c 1`    -- odpowiada za przesłanie do serwera tylko jednego pakietu.
 - `-W 15`   -- odpowiada za czas oczekiwania na odpowiedź od serwera, który w tym przypadku został
                ustawiony na 15 sekund.
 - `8.8.8.8` -- to adres IP, na który mają zostać posłane zapytania. Można także skorzystać z
                domeny.

Zatem jeśli to powyższe polecenie `ping` nie otrzyma odpowiedzi od serwera w ciągu 15 sekund, to
zostaną zainicjowane instrukcje zawarte poniżej w skrypcie watchdog'a.

W skrypcie mamy w zasadzie przesłanie wartości `0` do pliku `authorized` hub'a `usb3` , bo to tutaj
został wpięty modem LTE. Można by odszukać ścieżkę do samego modemu LTE ale ta może ulec zmianie po
rozłączeniu modemu, co skomplikowałoby nam nieco życie. Z racji, że odłączymy cały hub, to
wszystkie urządzenia podłączone do `usb3` zostaną zresetowane.

Warto też zaznaczyć, że mój router ma 2 fizyczne porty USB ale hub'ów dostępnych w systemie jest 4,
z czego 2 wewnętrzne. Akurat w przypadku mojego routera jest tak, że każdy port USB jest wpięty do
innego hub'a ale [nie jest to regułą][4]. Jeśli mamy problem z ustaleniem, który hub jest od
którego portu USB, to trzeba metodą prób i błędów przesłać wartość `0` do każdego z nich i w końcu
ten prawidłowy ujawni się rozłączając modem LTE.

W skrypcie watchdog'a mamy jeszcze przesłanie wartości `1` do pliku `authorized` , co efektywnie
włączy modem mniej więcej w taki sposób gdybyśmy go fizycznie podłączyli do portu USB.

Przesłanie wartości `0` i `1` trzeba też nieco opóźnić, bo inaczej mogą pojawić się problemy z
ponownym włączeniem modemu LTE.

### Praca dla cron

By całość nam jeszcze zadziałała i modem się faktycznie zresetował, gdy połączenie z internetem
zaniknie, musimy stworzyć zadanie dla cron'a, które będzie cyklicznie wykonywane. Tworzymy zatem
plik `/etc/crontabs/root` (jeśli nie istnieje) i dodajemy do niego poniższą zawartość:

    */2 * * * * /etc/skrypty/watchdog

Ta powyższa linijka ma za zadanie wykonywać skrypt watchdog'a co dwie minuty. W przypadku braku
internetu, modem LTE powinien zostać zresetowany, a połączenie z siecią odzyskane.

## Test internetowego watchdog'a

Gdy już wszystko mamy na swoim miejscu, symulujemy brak internetu, np. przez wysłanie zapytania
`ping` pod adres IP, który wiemy, że nie odpowie, np. jakiś lokalny (do ustawienia w skrypcie
watchdog'a). Odpalamy terminal i wpisujemy `logread -f` , po czym obserwujemy co tam system nam
wypisze.

Najpierw pojawia się poniższy komunikat:

    root: *************************************************
    root: **** Brak internetu. Restartuje modem LTE... ****
    root: *************************************************

Zatem wiemy, że skrypt został wykonany. Później mamy zaś takie oto wiadomości:

    kernel: [90294.008064] usb 3-1: USB disconnect, device number 12
    kernel: [90294.008506] option1 ttyUSB0: GSM modem (1-port) converter now disconnected from ttyUSB0
    kernel: [90294.012306] option 3-1:1.0: device disconnected
    kernel: [90294.020559] option1 ttyUSB1: GSM modem (1-port) converter now disconnected from ttyUSB1
    kernel: [90294.024582] option 3-1:1.1: device disconnected
    kernel: [90294.033280] huawei_cdc_ncm 3-1:1.2 wwan0: unregister 'huawei_cdc_ncm' usb-xhci-hcd.1.auto-1, Huawei CDC NCM device
    netifd: Network device 'wwan0' link is down
    netifd: Network alias 'wwan0' link is down
    netifd: Interface 'lte_4' has link connectivity loss
    netifd: lte_4 (26465): udhcpc: SIOCGIFINDEX: No such device
    netifd: lte_4 (26465): udhcpc: received SIGTERM
    netifd: lte_4 (26465): udhcpc: unicasting a release of 10.70.8.81 to 10.70.8.82
    netifd: lte_4 (26465): udhcpc: sending release
    netifd: lte_4 (26465): udhcpc: can't bind to interface wwan0: No such device
    netifd: lte_4 (26465): udhcpc: bindtodevice: No such device
    netifd: lte_4 (26465): udhcpc: entering released state
    netifd: lte_4 (26465): Command failed: Permission denied
    netifd: Interface 'lte_4' is now down
    netifd: Interface 'lte_4' is disabled
    netifd: lte (27037): Control device not valid
    netifd: Interface 'lte' is now down

Widać wyraźnie, że urządzenie USB zostało rozłączone i to mniej więcej w taki sam sposób, gdybyśmy
je fizycznie wyciągnęli z portu USB. Znika nam też z systemu interfejs LTE ( `wwan0` ) i nie będzie
go nawet na liście w wyjściu polecenia `ip` .

Po chwili pojawiają się w logu poniższe komunikaty:

    kernel: [90299.492540] hub 3-0:1.0: USB hub found
    kernel: [90299.492644] hub 3-0:1.0: 1 port detected
    kernel: [90299.495628] usb usb3: authorized to connect
    kernel: [90299.675996] usb 3-1: new high-speed USB device number 13 using xhci-hcd
    kernel: [90299.961386] option 3-1:1.0: GSM modem (1-port) converter detected
    kernel: [90299.961563] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB0
    kernel: [90299.966896] option 3-1:1.1: GSM modem (1-port) converter detected
    kernel: [90299.973418] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB1
    kernel: [90300.151756] huawei_cdc_ncm 3-1:1.2: MAC-Address: 00:1e:10:1f:00:00
    kernel: [90300.151784] huawei_cdc_ncm 3-1:1.2: setting rx_max = 16384
    kernel: [90300.162721] huawei_cdc_ncm 3-1:1.2: NDP will be placed at end of frame for this device.
    kernel: [90300.162874] huawei_cdc_ncm 3-1:1.2: cdc-wdm0: USB WDM device
    kernel: [90300.170843] huawei_cdc_ncm 3-1:1.2 wwan0: register 'huawei_cdc_ncm' at usb-xhci-hcd.1.auto-1, Huawei CDC NCM device, 00:1e:10:1f:00:00
    kernel: [90300.176571] usb-storage 3-1:1.3: USB Mass Storage device detected
    kernel: [90300.188928] scsi host0: usb-storage 3-1:1.3
    kernel: [90301.222184] scsi 0:0:0:0: Direct-Access     HUAWEI   TF CARD Storage  2.31 PQ: 0 ANSI: 2
    kernel: [90301.224138] sd 0:0:0:0: Power-on or device reset occurred
    kernel: [90301.232048] sd 0:0:0:0: [sda] Attached SCSI removable disk

Zatem modem LTE został ponownie wykryty. Pojawił się także fizyczny interfejs sieciowy `wwan0` .

Po chwili modem zostaje przygotowany do pracy:

    netifd: Interface 'lte' is setting up now
    netifd: lte (28012): sending -> AT
    netifd: lte (28012): sending -> ATZ
    netifd: lte (28012): sending -> ATQ0
    netifd: lte (28012): sending -> ATV1
    netifd: lte (28012): sending -> ATE1
    netifd: lte (28012): sending -> ATS0=0
    netifd: lte (28012): sending -> AT+CGDCONT=1,"IP","internet"
    netifd: lte (28012): SIM ready
    netifd: lte (28012): PIN set successfully
    netifd: lte (28012): Configuring modem
    netifd: lte (28012): Starting network lte
    netifd: lte (28012): Connecting modem
    netifd: lte (28012): sending -> AT^NDISDUP=1,1,"internet"
    netifd: lte (28012): Setting up wwan0
    netifd: Interface 'lte' is now up

Następnie interfejs LTE, pod który podlega fizyczny interfejs `wwan0` , zostaje skonfigurowany:

    netifd: Network device 'wwan0' link is up
    netifd: Network alias 'wwan0' link is up
    netifd: Interface 'lte_4' is enabled
    netifd: Interface 'lte_4' has link connectivity
    netifd: Interface 'lte_4' is setting up now
    netifd: lte_4 (28300): udhcpc: started, v1.33.1
    firewall: Reloading firewall due to ifup of lte (wwan0)
    netifd: lte_4 (28300): udhcpc: sending discover
    netifd: lte_4 (28300): udhcpc: sending select for 10.70.8.81
    netifd: lte_4 (28300): udhcpc: lease of 10.70.8.81 obtained, lease time 518400
    netifd: Interface 'lte_4' is now up
    firewall: Reloading firewall due to ifup of lte_4 (wwan0)

I jak widzimy adresacja IP na interfejsie LTE ( `wwan0` ) zostaje uzyskana od serwera DHCP mojego
ISP, czego efektem jest przywrócenie połączenia sieciowego do pełnej sprawności.

## Podsumowanie

W taki oto dość prosty sposób, kawałek skryptu oraz odpowiednie wykorzystanie mechanizmu
autoryzacji podłączanych do portów USB urządzeń umożliwiły nam zaimplementowanie watchdog'a na
routerze z wgranym firmware OpenWRT. Ilekroć tylko połączenie LTE nam zaniknie z jakiegoś powodu,
nasz watchdog zresetuje modem LTE i przywróci je do pełnej sprawności. Nie musimy już się obawiać,
że w środku nocy internet nam po cichu zdechnie, a obrazy ISO różnych dystrybucji linux'a nie
pobiorą się nam przez torrent'a. No i oczywiście też nasz komputer nie będzie pracował całą noc na
marne.


[1]: https://eko.one.pl/?p=openwrt-3g#automatycznyrestartpoczenia
[2]: /post/jak-przy-pomocy-usbguard-zabezpieczyc-porty-usb-przed-zlosliwymi-urzadzeniami/
[3]: https://www.kernel.org/doc/Documentation/usb/authorization.txt
[4]: /post/struktura-plikow-urzadzen-usb-w-katalogu-sys/
