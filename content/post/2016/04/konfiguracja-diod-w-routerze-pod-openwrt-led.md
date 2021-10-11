---
author: Morfik
categories:
- OpenWRT
date: "2016-04-30T17:11:47Z"
date_gmt: 2016-04-30 15:11:47 +0200
published: true
status: publish
tags:
- iptables
- chaos-calmer
- router
GHissueID: 402
title: Konfiguracja diod w routerze pod OpenWRT (LED)
---

Praktycznie każdy router posiada szereg diod LED, które wizualizują stan pracy takiego urządzenia. W
taki sposób jesteśmy w stanie stwierdzić czy sieć WiFi jest aktualnie włączona albo czy odbywa się
wymiana danych za jej pomocą. Podobnie możemy ocenić aktywność mechanizmu WPS oraz czy połączenie
przewodowe zostało ustanowione. Routery TP-LINK'a mają także w standardzie diodę `system` , która
informuje nas czy router działa prawidłowo i nie uległ powieszeniu. W OpenWRT wszystkie te wyżej
opisane właściwości można skonfigurować, tak by dioda LED reagowała w określony sposób na pewne
zaistniałe zdarzenie. W tym wpisie przyjrzymy się bliżej konfiguracji diod routera.

<!--more-->
## Wsparcie dla diod LED w OpenWRT

Ilość jak i rodzaj diod LED zaimplementowanych na routerach jest różna i zależy głównie od samego
modelu, który przewinie nam się przez nasze łapki. Dlatego też wszystkie poniższe informacje dotyczą
routera [TP-LINK TL-WR1043N/ND v2](http://www.tp-link.com.pl/products/details/TL-WR1043ND.html) , by
mieć jakiś punkt odniesienia. Niemniej jednak, szereg cech będzie wspólny dla większości routerów
pracujących pod kontrolą OpenWRT. To jakie diody zostały wykryte przez system możemy ustalić
zaglądając do katalogu `/sys/class/leds/` . Dla tego modelu routera, ten katalog prezentuje się
następująco:

![](/img/2016/04/1.dioda-led-openwrt-router.png#huge)

W przypadku tego routera wszystkie diody zostały wykryte poprawnie. Niemniej jednak, OpenWRT nie
zawsze zapewnia wsparcie dla wszystkich diod. Część z nich może w ogóle nie działać. Choć tego typu
sytuacje zdarzają się bardzo rzadko. Zwykle też wystarczy zgłosić tego typu informację [do
developerów OpenWRT](https://dev.openwrt.org/) i być może w niedługim czasie ten problem zostanie
wyeliminowany.

Jak widzimy wyżej, OpenWRT wykrył w tym routerze 9 diod. Pozycje `tp-link:green:lan1` ,
`tp-link:green:lan2` , `tp-link:green:lan3` `tp-link:green:lan4` oraz `tp-link:green:wan`
reprezentują 5 portów switch'a (4 porty LAN i 1 WAN). Mamy także `tp-link:green:wps` odpowiedzialny
za diodę WPS. Nie zabrakło też wpisu `tp-link:green:wlan` odpowiedzialnego za WiFi. Ten router
dysponuje także portem USB, zatem ma też `tp-link:green:usb` . No i oczywiście nie mogło zabraknąć
pozycji `tp-link:green:system` odpowiedzialnej za diodę `system` .

Powyższe nazwy dowiązań symbolicznych nie są przypadkowe i wynikają z następującego wzoru. Najpierw
mamy nazwę producenta urządzenia ( `tp-link` ) . Następnie mamy kolor diody ( `green` ). Kolejny
człon nazwy to etykieta takiej diody ( `wlan` ). W tym przypadku wszystkie diody LED są zielone.
Dlatego też nie da się tutaj zrobić wymyślnych kombinacji, które by reagowały na pewne określone
zdarzenia. Gdyby były dostępne tutaj inne kolory, można by przypisać, np. portowi WAN, reagowanie na
pakiet inicjujący połączenie SSH. W taki sposób standardowo świeciłaby się dioda zielona ale przy
próbie nawiązania połączenia SSH zapalałaby się dioda, dajmy na to, czerwona.

Dioda LED w routerze jest zapalana i gaszona w oparciu o pewien określony moduł sterujący.
Informację na temat aktywnego modułu możemy uzyskać w poniższy sposób:

    # cat /sys/class/leds/tp-link\:green\:system/trigger
    [none] switch0 timer default-on netdev usbdev phy0rx phy0tx phy0assoc phy0radio phy0tpt

W przypadku powyższej diody, nie jest wybrany żaden moduł. Nawiasy `[ ]` oznaczają aktywną pozycję.
Mamy tam tylko kilka modułów ale nie są to wszystkie możliwe. Możemy doinstalować szereg pakietów,
które udostępniają własne moduły. [Pełną listę
modułów](https://wiki.openwrt.org/doc/uci/system#leds) możemy uzyskać wpisując w terminalu to
poniższe polecenie:

    # opkg list | grep ledtrig

## Sterowanie zachowaniem diody LED

Każda dioda LED ma osobny moduł sterujący, który możemy dowolnie zaprogramować. By dioda LED
korzystała z pewnego określonego modułu, musimy ją skonfigurować za pomocą pliku
`/etc/config/system` . Poniżej jest standardowa konfiguracja diod dla tego routera:

    ...
    config led 'led_usb'
        option name 'USB'
        option sysfs 'tp-link:green:usb'
        option trigger 'usbdev'
        option dev '1-1'
        option interval '50'

    config led 'led_wlan'
        option name 'WLAN'
        option sysfs 'tp-link:green:wlan'
        option trigger 'phy0tpt'

Każda dioda LED ma swój własny blok `config led` . Dioda, której on dotyczy jest określona przez
plik w katalogu `/sys/class/leds/` . Dla przykładu, mając tam plik `tp-link:green:wlan` , tworzymy
nazwę `led_wlan` . Następnie mamy szereg opcji, z tym, że zależą one od wykorzystywanego modułu.
Niemniej jednak, każdy blok musi posiadać minimum trzy opcje. W `name` określamy nazwę, może być
dowolna. W `sysfs` podajemy nazwę pliku odnoszącego się do diody LED. Zaś w `trigger` wybieramy
moduł. Dalej już są opcje charakterystyczne dla określonego modułu.

## Konfiguracja diody system

W routerze TL-WR1043ND nie ma zbytniego pola manewru jeśli chodzi o diody LED. Niemniej jednak, jest
jedna dioda, która w sumie nic nie robi, tylko ciągle się świeci. Chodzi o diodę `system` . Można ją
jednak zaprogramować tak, by migała w tempie zależnym od obciążenia routera (do wglądu w
`/proc/loadavg` ). Jeśli router jest w spoczynku, to ma niski `loadavg` . Natomiast jeśli pracuje
pod solidnym obciążeniem, to `loadavg` mu wzrasta. Dzięki takiej zależności, im bardziej obciążony
jest router, tym szybciej miga dioda na jego obudowie. W ten sposób jesteśmy w stanie ustalić czy
coś niepokojącego się dzieje na routerze. By wdrożyć u siebie taką funkcjonalność, musimy
doinstalować poniższy pakiet:

    # opkg update
    # opkg install kmod-ledtrig-heartbeat

Następnie edytujemy plik `/etc/config/system` i dopisujemy do niego tę poniższą sekcję:

    config led 'led_system'
        option name 'SYSTEM'
        option sysfs 'tp-link:green:system'
        option trigger 'heartbeat'

Opcja `trigger` musi zostać ustawiona na `heartbeat` . Teraz wystarczy zresetować już router. Po
jego załadowaniu, dioda powinna dość szybko migać ale po chwili zwykle się uspokaja.

## Konfiguracja diody w oparciu o pakiety sieciowe

Wyżej wspomniałem o konfiguracji diody w taki sposób, by migała przy próbie połączenia się do
routera przez SSH na porcie WAN. Może i nie ten router nie ma kolorowych diod ale jego dioda USB
jest na przednim panelu, w przeciwieństwie do innych modelów. W dużej mierze ta dioda jest
niewykorzystywana i możemy zmienić jej przeznaczenie, tak by była sterowana w oparciu o pakiety
sieciowe. Jeśli interesuje nas ta kwestia, to musimy doinstalować poniższy pakiet:

    # opkg update
    # opkg install iptables-mod-led

By skonfigurować ten cały mechanizm powiadamiania o pakietach sieciowych za pomocą diody LED,
potrzebne nam są dwie rzeczy. Pierwszą z nich jest odpowiednia reguła firewall'a, którą trzeba
umieścić w pliku `/etc/firewall.user` . Ta poniższa reguła sprawi, że za każdym razem, gdy nastąpi
nowe połączenia na port SSH z interfejsu WAN, dioda zapali się na 1 sekundę:

    iptables -A input_wan_rule -p tcp --dport 22 -j LED --led-trigger-id ssh --led-delay 1000

Po dodaniu tej regułki, restartujemy firewall:

    # /etc/init.d/firewall restart

Druga potrzebna nam rzecz, to konfiguracja w pliku `/etc/config/system` . Musimy tam określić
odpowiedni trigger w oparciu o tę powyższą regułę. Format trigger'a jest następujący:
`netfilter-ssh` , czyli do `netfilter-` dodajemy `ssh` , który został określony w parametrze
`--led-trigger-id` w powyższej regule firewall'a. Niemniej jednak, z jakiegoś powodu nie da rady
ustawić tego trigger'a. W przypadku, gdy byśmy przepisali sekcję dotyczącą diody USB do tej
poniższej postaci:

    config led 'led_usb'
        option name 'USB'
        option sysfs 'tp-link:green:usb'
        option trigger 'netfilter-ssh'
        option dev '1-1'

To system podczas startu zaloguje taki oto komunikat:

    syslog: Skipping trigger 'netfilter-ssh' for led 'USB' due to missing kernel module

Nie wiem czemu tak się dzieje, bo zresetowanie skryptu `/etc/init.d/led` po uruchomieniu routera już
tego komunikatu nie generuje. No i też trigger jest ustawiany poprawnie. Można to obejść przez
wykomentowanie całej powyższej zawartości z pliku `/etc/config/system` oraz przez dodanie do pliku
`/etc/rc.local` tej poniższej linijki:

    echo "netfilter-ssh" > /sys/class/leds/tp-link\:green\:usb/trigger
